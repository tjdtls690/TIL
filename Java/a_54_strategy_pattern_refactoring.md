# 목차

1. [문제 인식](#1-문제-인식) <br/>
2. [원인 분석](#2-원인-분석) <br/>
3. [문제 해결](#3-문제-해결) <br/>

<br/>

# [Java, JPA] 코드 리팩토링 (if문 처리 - 전략패턴)

## 1. 문제 인식

신고기능은 총 3가지가 있다.

1. 게시글 신고
2. 게시글의 댓글 신고
3. 유저의 닉네임 신고

그런데 이 기능들이 동작하는 로직을 보니 if문이 꽤나 많은 모습을 볼 수 있었다. 이 부분을 어떻게 처리할 수 없을까?

<br/>

## 2. 원인 분석

먼저 문제가 되는 로직을 살펴보자. 위의 **3가지 신고 기능을 전부 하나의 메서드에서 처리하려다보니** 불가피하게 if문을 통해 구현된 모습을 볼 수 있다.

```java
@RequiredArgsConstructor
@Service
public class ReportService {

    // ...생략

    @Transactional
    public void report(final Member reporter, final ReportRequest request) {
        if (request.type() == ReportType.POST) { // 게시글 신고인 경우
            reportPost(reporter, request);
        }
        if (request.type() == ReportType.COMMENT) { // 댓글 신고인 경우
            reportComment(reporter, request);
        }
        if (request.type() == ReportType.NICKNAME) { // 닉네임 신고인 경우
            reportNickname(reporter, request);
        }
    }

    private void reportPost(
            final Member reporter,
            final ReportRequest request
    ) {
        final Post reportedPost = postRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(PostExceptionType.POST_NOT_FOUND));
        validatePost(reporter, reportedPost, request);

        saveReport(reporter, request);
        blindPost(request, reportedPost);
    }
    
    // ...생략
    
    private void reportComment(
            final Member reporter,
            final ReportRequest request
    ) {
        final Comment reportedComment = commentRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(CommentExceptionType.COMMENT_NOT_FOUND));
        validateComment(reporter, request, reportedComment);

        saveReport(reporter, request);
        blindComment(request, reportedComment);
    }
    
    // ...생략
    
    private void reportNickname(
            final Member reporter,
            final ReportRequest request
    ) {
        final Member reportedMember = memberRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(MemberExceptionType.NONEXISTENT_MEMBER));
        validateNickname(reporter, request);

        saveReport(reporter, request);
        changeNicknameByReport(reportedMember, request.type());
    }
    
    // ...생략
}
```

<br/>

지금 생략한 로직까지 전부 꺼내서 본다면, ReportService의 크기가 너무 비대하다는 것을 느끼게 된다. 하나의 기능에 3가지 동작이 들어가있고, 각 동작에서 분리되는 메서드 개수가 만만치 않기 때문이다.

때문에 ReportService의 응집도가 떨어지고, 동시에 가독성도 같이 떨어져서 코드를 파악하기가 쉽지 않다. 각 동작의 주 메서드가 public이 아니고 private이기 때문에 다른 메서드들과 더 헷갈리는 점도 존재한다.

<br/>

## 3. 문제 해결

리팩토링을 어떤 식으로 할 지 고민을 시작할 때 가장 먼저 생각난 방법이 전략 패턴이었기 때문에 이 방법을 택했다. 각 동작에 해당하는 로직끼리 묶어서 따로 분리하기 때문에 OOP의 여러 장점들을 챙길 수 있을 것이라 생각했기 때문이다. 자세한 장점들은 밑에서 코드를 보면서 알아보자.

먼저 리팩토링 한 뒤의 로직을 살펴보자.

일단 위의 코드를 보면 알겠지만, 3개의 메서드에서 공통되는 메서드가 saveReport() 메서드가 있다. 이 부분까지 생각해주면서 전략의 기반이 되는 인터페이스를 만들어보자.

```java
@FunctionalInterface
public interface ReportStrategy {

    void report(final Member reporter, final ReportRequest request);

    default void saveReport(
            final Member reporter,
            final ReportRequest request,
            final ReportRepository reportRepository
    ) {
        // ...생략
    }

}
```

<br/>

추상 메서드는 report 메서드 하나밖에 없기에, @FunctionalInterface를 통해 함수형 인터페이스라는 것을 검증 및 명시해주었다. 

람다 형식으로 해당 인터페이스를 사용하지 않게 되더라도, 추상 메서드가 하나만 존재한다면 @FunctionalInterface는 꼭 붙여주었다. 테스트든 어디서든, 언제 어느때에 람다형식으로 해당 인터페이스를 사용하게 될 지 알 수 없어서이다.

또한 saveReport default 메서드는 해당 인터페이스가 구현할 전략 클래스들이 바로바로 가져다 사용할 수 있다. 3가지 동작에서 공통으로 쓰이는 메서드이기에 인터페이스로 따로 빼주었다.

이제 해당 인터페이스를 구현한 각 전략(동작)들을 분리해보자.

```java
@RequiredArgsConstructor
@Component
public class ReportPostStrategy implements ReportStrategy {

    private final PostRepository postRepository;
    private final ReportRepository reportRepository;

    @Override
    public void report(final Member reporter, final ReportRequest request) {
        final Post reportedPost = postRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(PostExceptionType.POST_NOT_FOUND));
        validatePost(reporter, reportedPost, request);

        saveReport(reporter, request, reportRepository); // 인터페이스의 default 메서드
        blindPost(request, reportedPost);
    }

    // ...생략

}

```

```java
@RequiredArgsConstructor
@Component
public class ReportCommentStrategy implements ReportStrategy {

    private final CommentRepository commentRepository;
    private final ReportRepository reportRepository;

    @Override
    public void report(final Member reporter, final ReportRequest request) {
        final Comment reportedComment = commentRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(CommentExceptionType.COMMENT_NOT_FOUND));
        validateComment(reporter, request, reportedComment);

        saveReport(reporter, request, reportRepository); // 인터페이스의 default 메서드
        blindComment(request, reportedComment);
    }

    // ...생략

}
```

```java
@RequiredArgsConstructor
@Component
public class ReportNicknameStrategy implements ReportStrategy {

    private final MemberRepository memberRepository;
    private final ReportRepository reportRepository;

    @Override
    public void report(final Member reporter, final ReportRequest request) {
        final Member reportedMember = memberRepository.findById(request.id())
                .orElseThrow(() -> new NotFoundException(MemberExceptionType.NONEXISTENT_MEMBER));
        validateNickname(reporter, request);

        saveReport(reporter, request, reportRepository); // 인터페이스의 default 메서드
        changeNicknameByReport(reportedMember, request);
    }

    // ...생략

}
```

<br/>

또한 이 전략들을 따로 관리하면서 반환 값을 전달해주는 클래스를 만들었다. 그래야 여러 기능이 존재할 수 있는 ReportService의 로직이 훨씬 더 깔끔해질 것이라 생각했기 때문이다.

```java
@Component
public class ReportActionProvider {

    private final Map<ReportType, ReportStrategy> reportActions;

    public ReportActionProvider(
            final ReportPostStrategy reportPostStrategy,
            final ReportCommentStrategy reportCommentStrategy,
            final ReportNicknameStrategy reportNicknameStrategy
    ) {
        this.reportActions = new EnumMap<>(ReportType.class);
        this.reportActions.put(ReportType.POST, reportPostStrategy);
        this.reportActions.put(ReportType.COMMENT, reportCommentStrategy);
        this.reportActions.put(ReportType.NICKNAME, reportNicknameStrategy);
    }

    public ReportStrategy getStrategy(ReportType type) {
        return reportActions.get(type);
    }

}
```

<br/>

ReportActionProvider를 빈 객체로 관리하는 이유는, 해당 클래스는 단 한 개만 존재해도 상관이 없고, Repository들을 쉽게 관리하기 위함이다.

이제 ReportService의 로직을 살펴보자.

```java
@RequiredArgsConstructor
@Service
public class ReportService {

    private final ReportActionProvider reportActionProvider;

    @Transactional
    public void report(final Member reporter, final ReportRequest request) {
        final ReportStrategy reportStrategy = reportActionProvider.getStrategy(request.type());
        reportStrategy.report(reporter, request);
    }

}
```

<br/>

로직을 리팩토링 한 뒤의 패키지 구조를 살펴보면,

<img src="https://tjdtls690.github.io/assets/img/blog/전략패턴_리팩토링1.PNG">

<br/>

이렇게 구현해놓고 보니, 전략패턴의 장점들이 눈에 보이기 시작한다.

첫번째로, **단일 책임의 원칙을 지킬 수 있다.**

이는 높은 응집도와 많은 연관성을 가지고 있는데, 각 클래스가 하나의 동작과 관련된 로직만을 가질 수 있게 되기 때문이다. 때문에 유지보수성을 높이고, 이는 가독성까지도 챙길 수 있는 효과를 가져온다. 

각 클래스를 살펴볼 때마다 어떤 동작에 관한 로직인지 빠르게 파악할 수 있고, ReportService에서 기능이 더 추가된다 하더라도 비약적으로 비대해지지 않을 수 있기 때문이다.

<br/>

두번째로, **OCP (개방폐쇄의 원칙) , 즉 변경엔 닫혀있고 확장엔 열려있는 객체지향 원칙이 실현된다.**

이는 낮은 결합도와 많은 연관성을 가지고 있다. enum 타입인 ReportType의 종류와 신고 동작이 더 추가가 될 때, ReportService에서 if문과 메서드가 더 추가가 되는 것이 아니라 전략(클래스) 하나만 더 추가해주면 끝나기 때문이다.

또한 그로 인해 ReportService의 로직에서 기능(메서드)들이 더 추가된다고 하더라도 깔끔하게 추가해나갈 수 있다.

역량이 아직 부족해서인지, 이 방법보다 더 깔끔하게 리팩토링을 할 수 있는 방법을 찾진 못했다.