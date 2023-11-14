# 목차

1. [문제 상황](#1-문제-상황) <br/>
2. [원인 분석](#2-원인-분석) <br/>
3. [문제 해결](#3-문제-해결) <br/>
    1. [잠금 관리용 테이블을 새로 만든다. - 채택하지 않음](#1-잠금-관리용-테이블을-새로-만든다---채택하지-않음) <br/>
    2. [2개 PostOption(선택지)의 조회 순서를 획일화시킨다.](#2-2개-postoption선택지의-조회-순서를-획일화시킨다) <br/>

<br/>

# [Mysql, Java] 데드락 이슈 해결

## 1. 문제 상황

**[저번 글](https://bit.ly/40qtVFj)** 에서 **동시성 이슈**를 해결하기 위해 **비관적 락의 베타 락**을 썼었다. 동시에 같은 레코드에 접근할 땐 단 한 명의 유저만 접근할 수 있어야 하기 때문이다. 하지만 이 과정에서 데드락 이슈의 가능성이 생겼다는 것을 알게 되었다.

<br/>

## 2. 원인 분석

왜 베타 락을 통해 동시성 이슈를 해결하니 데드락 이슈가 발생하게 되었을까?

먼저 동시성 이슈를 해결한 뒤의 **'투표 선택지 변경 기능'** 의 로직을 살펴보자. 2개의 PostOption을 **각각 락을 걸면서** 조회하고, 각 투표 개수의 수정이 끝난 뒤에 락이 반환되는 모습이다.

```java
@Service
@RequiredArgsConstructor
public class VoteService {

    // ...생략
    
    @Transactional
    public void changeVote(
            final Member member,
            final Long postId,
            final Long originPostOptionId,
            final Long newPostOptionId
    ) {
        final Post post = postRepository.findById(postId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        // Exclusive Lock을 적용한 findByIdForUpdate() 메서드
        final PostOption originPostOption = postOptionRepository.findByIdForUpdate(originPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
        // Exclusive Lock을 적용한 findByIdForUpdate() 메서드
        final PostOption newPostOption = postOptionRepository.findByIdForUpdate(newPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        voteRepository.delete(originVote);
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        // 기존 선택지의 투표 개수 감소
        originPostOption.decreaseVoteCount();
        // 새로운 선택지의 투표 개수 증가
        newPostOption.increaseVoteCount();
    }
    
    // ...생략
}
```

<br/>

이러한 상태에서, 데드락 이슈는 어느 상황에 발생할 수 있을까?

A와 B 유저가 있고, 1번과 2번 선택지가 있다고 가정합니다. 만약 동시에 A유저가 1번 -> 2번 선택지로, B유저가 2번 -> 1번 선택지로 투표를 바꿀 때. 어떤 상황이 펼쳐질까?

|                            A유저                             |       1번 PostOption<br />{id : 1, vote_count : 2}        |       2번 PostOption<br />{id : 2, vote_count : 6}        |                            B유저                             |
| :----------------------------------------------------------: | :-------------------------------------------------------: | :-------------------------------------------------------: | :----------------------------------------------------------: |
| select * <br />from post_option <br />where id = 1;<br /><br />**id가 1인 레코드의 Exclusive Lock 획득 조회 결과 : {id : 1, vote_count : 2}** | {id : 1, vote_count : 2}<br /><br />**Lock 주인 : A유저** |                 {id : 2, vote_count : 6}                  |                                                              |
|                                                              | {id : 1, vote_count : 2}<br /><br />**Lock 주인 : A유저** | {id : 2, vote_count : 6}<br /><br />**Lock 주인 : B유저** | select * <br />from post_option <br />where id = 2;<br /><br />**id가 2인 레코드의 Exclusive Lock 획득 조회 결과 : {id : 2, vote_count : 6}** |
| select * <br />from post_option <br />where id = 2;<br /><br />id가 2인 PostOption을 조회하려 하지만 **B유저가 락을 갖고 있어서 대기 상태로 돌입** | {id : 1, vote_count : 2}<br /><br />**Lock 주인 : A유저** | {id : 2, vote_count : 6}<br /><br />**Lock 주인 : B유저** |                                                              |
|                                                              | {id : 1, vote_count : 2}<br /><br />**Lock 주인 : A유저** | {id : 2, vote_count : 6}<br /><br />**Lock 주인 : B유저** | select * <br />from post_option <br />where id = 1;<br /><br />id가 1인 PostOption을 조회하려 하지만 **A유저가 락을 갖고 있어서 대기 상태로 돌입** |

<br/>

결국 **A유저와 B유저 둘 다 대기 상태로 돌입**하게 되면서 **데드락 이슈**가 발생하게 된다.

<br/>

## 3. 문제 해결

이 데드락 이슈를 해결할 방법을 고민해보았다.

#### 1) 잠금 관리용 테이블을 새로 만든다. - 채택하지 않음

한 번에 2개의 레코드의 Lock을 걸 수 있는 방법이 있는지 고민하다 고안해낸 것인데, **‘외래키는 부모테이블이나 자식 테이블에 데이터가 있는지 체크하는 작업이 필요하므로 잠금이 여러 테이블로 전파된다’** 는 특성을 이용한 방법이다.

잠금용 테이블의 컬럼 구성은 PostOption의 id 2개이다. 그래서 Post **'투표 선택지 변경 기능'** 의 흐름을 가정해보면, 

1. 해당 테이블에 기존 PostOption과 새로운 PostOption의 id를 저장한다.
2. 저장한 해당 레코드의 Exclusive Lock을 가져온다.
   - 새로 만든 테이블에서 하나의 레코드에 락이 걸리면, PostOption 테이블에서 2개의 레코드에 락이 걸리게 된다.
3. 투표를 기존 PostOption에서 새로운 PostOption으로 변경한다.
   - 기존 선택지의 투표 개수 감소, 새로운 선택지의 투표 개수 증가
4. 잠금용 테이블에 저장한 레코드를 삭제한다.

<br/>

하지만 이 방법은 채택하지 않았다. 이유는 테이블 연관관계와 도메인의 복잡성을 불필요하게 증가시킨다고 판단했기 때문이다.

<br/>

#### 2) 2개 PostOption(선택지)의 조회 순서를 획일화시킨다.

데드락 이슈가 발생하는 상황을 생각해보면, A유저가 1번 -> 2번, B유저가 2번 -> 1번 으로 동시에 바꿀 때이다. 그렇다면 이 조회하는 순서를 획일화 시킨다면 어떨까?

즉, 무조건 PostOption의 id가 낮은순으로 조회하도록 구현하는 것이다. 그렇다면 위와 같은 상황에서 A유저도 1번 -> 2번, B유저도 1번 -> 2번 순으로 PostOption을 조회하기 때문에 데드락 이슈가 발생하지 않게 된다.

물론 기존의 PostOption의 voteCount는 감소, 새로운 PostOption의 voteCount는 증가를 해야하기 때문에 동작 방식이 달라져버릴 수 있다. 하지만 이는 비즈니스 로직 상에서 해결해줄 수 있다. 현재는 기존 PostOption과 새로운 PostOption을 분별해내는 메서드를 따로 만들어 해결한 상태다.

이제 조회 순서를 획일화 시킨 방법을 적용하여 개선한 코드를 살펴보자.

```java
@Service
@RequiredArgsConstructor
public class VoteService {

    // ...생략
    
    @Transactional
	public void changeVote(
            final Member member,
            final Long postId,
            final Long originPostOptionId,
            final Long newPostOptionId
    ) {
        final Post post = postRepository.findById(postId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        // 파라미터로 전달받은 PostOption의 id가 낮은 순으로 정렬하여 PostOption을 차례대로 조회한다.
        // 물론 Exclusive Lock을 획득하는 상황이다.
        final List<PostOption> postOptions = Stream.of(originPostOptionId, newPostOptionId)
                .sorted()
                .map(postOptionRepository::findByIdForUpdate)
                .map(postOption -> postOption.orElseThrow(
                        () -> new NotFoundException(PostOptionExceptionType.NOT_FOUND)))
                .toList();

        // 기존 PostOption을 분별해내어 가져온다.
        final PostOption originPostOption = getPostOptionById(postOptions, originPostOptionId);
        // 새로운 PostOption을 분별해내어 가져온다.
        final PostOption newPostOption = getPostOptionById(postOptions, newPostOptionId);

        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        voteRepository.delete(originVote);
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        // 기존 선택지의 투표 개수 감소
        originPostOption.decreaseVoteCount();
        // 새로운 선택지의 투표 개수 증가
        newPostOption.increaseVoteCount();
    }

    // 기존과 새로운 PostOption을 분별해내는 메서드
    private PostOption getPostOptionById(final List<PostOption> postOptions, final Long id) {
        return postOptions.stream()
                .filter(postOption -> Objects.equals(postOption.getId(), id))
                .findAny()
                .orElseThrow(() -> new NotFoundException(PostOptionExceptionType.NOT_FOUND));
    }
}
```

<br/>

데드락 이슈를 해결했다 하더라도 혹시 모르니, 최악의 상황을 한 번 가정해보자.

1. A유저 : 1번 -> 2번
2. B유저 : 2번 -> 3번
3. C유저 : 3번 -> 4번
4. D유저 : 4번 -> 5번

위와 같이 동시에 4명의 유저가 투표 선택지를 변경한다고 가정했을 때, A유저부터 D유저까지 차례대로 대기상태가 걸리게 된다. 하지만 결국 D유저의 작업이 끝나게 되고, 다시 거꾸로 차례대로 작업이 끝날 수 있게 된다.

따라서 어떠한 상황이 온다 하더라도 데드락 이슈가 발생할 일은 생기지 않는다.
