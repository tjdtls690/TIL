# 목차

1. [문제 인식 - 비관적 락을 걸면 다른 유저의 조회 기능은...?](#1-문제-인식---비관적-락을-걸면-다른-유저의-조회-기능은) <br/>
2. [문제 해결 - 낙관적 락](#2-문제-해결---낙관적-락) <br/>

<br/>

# [MySQL, JPA] 페이징 성능 개선 여정 3편 - 동시성 처리 방식 개선

## 1. 문제 인식 - 비관적 락을 걸면 다른 유저의 조회 기능은...?

1편에서 성능을 개선하기 위해 반정규화 방식을 사용했다. Post 테이블에 vote_count 컬럼을 추가함으로써 게시글을 조회할 때 Vote 테이블을 건드리지 않아도 되게끔 개선했다.

**1편 링크 : [[MySQL] 페이징 성능 개선 여정 1편 - 잘못된 성능 개선 바로잡기](https://github.com/tjdtls690/TIL/blob/main/db/001_performance_improvement_journey_001.md)**

하지만 '투표'와 '투표 변경' 기능이 동작할 때 vote_count의 값이 실시간으로 변경이 되어야 하는데, 여기서 동시성 이슈가 터지게 된다. 이 문제를 비관적 락으로 해결한 과정을 적은 글이 밑의 링크다.

**링크 : [[Mysql, JPA] 동시성 이슈 해결 (Syncronized, 낙관적 락, 비관적 락)](https://github.com/tjdtls690/TIL/blob/main/Java/a_52_concurrency_issues.md)**

대부분 잘 알다시피 비관적 락은 실제로 데이터베이스의 락을 사용하여 동시성을 제어하는 방법이다. 그리고 보통 락은 **'베타 락'** 을 사용하게 된다. 동시성 이슈 해결 글을 보면 알겠지만, 비관적 락을 사용한 이유는 '투표'와 '투표 변경' 기능은 굉장히 많이 사용될 것 같아서 충돌이 자주 일어날 것으로 예상했기 때문이다.

그러나 여기서 간과한 점이 하나 있다. 사실 전체 기능에서 사용 빈도수를 측정해보면 보통 '조회' 유형의 기능이 가장 많이 동작될 것이다. 그런데 투표 기능이 동작할 때마다 해당 레코드에 '베타 락'이 걸리게 되면 어떻게 될까? '투표'와 '투표 변경' 기능에서 사용되는 레코드를 조회하려는 기능들이 그 순간만큼은 모두 타격을 입게 된다.

즉, 상대적으로 적게 동작되는 기능에 의해 더 많이 동작되는 기능이 타격을 받는 굉장히 비효율적인 모습이 되어버린다. 전체적인 기능을 생각하지 않고 성능을 개선하려는 기능에만 초점이 맞춰져서 시야가 좁아진 탓이다.

## 2. 문제 해결 - 낙관적 락

이번 프로젝트에선 다른 유저의 조회기능에 영향을 주지 않도록 동시성 이슈를 처리해보려 한다.

그럼 '조회는 같이 가능하면서도 충돌이 일어날 때만 처리해주는 방식'이 뭐가 있을까 생각해보면 '낙관적 락'이 떠오른다. 낙관적 락은 경합이 별로 발생하지 않을 것이라고 보고 잠금을 거는 방식이다. 따라서 다수의 유저가 같은 레코드를 동시에 조회하는 것은 가능하면서도, 만약 데이터를 수정할 때 'Race Condition' 이슈가 발생한다면 그 때에만 충돌을 잡아준다.

비관적 락과 낙관적 락에 대한 자세한 설명은 '동시성 이슈 해결' 글에 적어놨으니, 이 글에선 '낙관적 락'을 실제로 적용하는 과정에 대해서만 적어보려 한다.

먼저 낙관적 락을 건 다음, 충돌이 일어날 시 재시도를 최대 3번은 할 수 있도록 구현했다.

**낙관적 락)**

```java
public class Card extends BaseEntity {

    // ... 생략

    @Version
    @ColumnDefault("0")
    private Long version;

    // ... 생략

}
```

Card 엔티티에서 @Version 컬럼을 추가하여 낙관적 락을 적용한 모습이다.

<br/>

**재시도 처리 로직)**

```java
public class CardService {
	@Retryable(
            retryFor = ObjectOptimisticLockingFailureException.class,
            backoff = @Backoff(delay = 100)
    )
    public WriteCommentOnCardResponse writeCommentOnCard(final CacheMember member, final Long cardId) {
        commentRepository.findByMemberIdAndCardId(member.getId(), cardId)
                .ifPresent(comment -> {throw new BadRequestException(CardExceptionType.MY_COMMENT_ALREADY_EXIST);});
        final Card card = cardRepository.findById(cardId)
                .orElseThrow(() -> new NotFoundException(CardExceptionType.NOT_FOUND_CARD));

        card.increaseCommentCountByOne();
        todayCardIncreaseCommentCount(card);
        saveComment(member, card);

        return WriteCommentOnCardResponse.from(WRITE_COMMENT_ON_CARD_SUCCESS);
    }

    // ... 생략

    @Recover
    public WriteCommentOnCardResponse recoverWriteCommentOnCard(
            final OptimisticLockingFailureException e,
            final CacheMember member,
            final Long cardId
    ) {
        throw new RetryLimitExceededException(CardExceptionType.WRITE_COMMENT_ON_CARD_FAILURE);
    }
}
```

<br/>

@Retryable은 이 의존성을 추가해줘야 사용할 수 있다.

```groovy
implementation 'org.springframework.retry:spring-retry'
```

지금은 retryFor, backoff 속성만 사용하고 있기 때문에 maxAttempts 속성은 기본 값이 '3'이 된다. 즉, 3번까지만 재시도를 하고 그래도 충돌이 계속 일어난다면 @Recover로 넘어간다는 의미다.

여기서 나온 @Retryable의 각 속성을 살펴보면,

- **retryFor :** 재시도 될 Exception 타입 정의
- **backoff - delay :** 각 재시도 전의 대기 시간
- **maxAttempts :** 최대 시도 횟수

이렇게 코드로 직접 구현하지 않고도 손쉽게 '낙관적 락'을 걸고 '재시도 로직'을 구현할 수 있다. 이런걸 보면 스프링의 꽃은 'Reflection'과 'AOP'란 생각이 든다. 덕분에 너무나 편리한 라이브러리들이 만들어질 수 있으니 말이다.

<br/>

**재시도 로직 테스트 코드)**

```java
	@Test
    @DisplayName("여러 사용자가 동시에 같은 카드의 댓글을 작성할 경우, 동기화 처리가 정상 동작된다.")
    void handleConcurrentCommentsOnCardSynchronously() {
        // given
        // data-test.sql
        final Member memberA = memberService.findById(1L);
        final Member memberB = memberService.findById(2L);
        long cardIdA = 1L;
        long cardIdB = 2L;

        CompletableFuture<Void> futureA =
                CompletableFuture.runAsync(() -> cardService.writeCommentOnCard(CacheMember.from(memberA), cardIdA));
        CompletableFuture<Void> futureB =
                CompletableFuture.runAsync(() -> cardService.writeCommentOnCard(CacheMember.from(memberB), cardIdB));

        CompletableFuture<Void> combinedFuture = CompletableFuture.allOf(futureA, futureB);

        // when, then
        assertThatNoException().isThrownBy(combinedFuture::get);
    }
```

이처럼 2개의 쓰레드를 이용하여 동시성 테스트를 진행했는데 재시도가 잘 동작하는 모습을 볼 수 있었다.