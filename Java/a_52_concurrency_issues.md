# 목차

1. [문제 상황](#1-문제-상황) <br/>
2. [원인 분석](#2-원인-분석) <br/>
3. [문제 해결 과정](#3-문제-해결-과정) <br/>
    1. [Syncronized (암시적 Lock)](#1-syncronized-암시적-lock) <br/>
        1. [하지만 이 방법은 사용하지 않기로 판단했다](#하지만-이-방법은-사용하지-않기로-판단했다) <br/>
    2. [낙관적 잠금 (Optimistic Lock)](#2-낙관적-잠금-optimistic-lock) <br/>
        1. [하지만 이 방법도 사용하지 않기로 판단했다](#하지만-이-방법도-사용하지-않기로-판단했다) <br/>
    3. [비관적 잠금 (Pessimistic Lock)](#3-비관적-잠금-pessimistic-lock) <br/>
        1. [비관적 잠금을 사용하기로 판단한 이유](#비관적-잠금을-사용하기로-판단한-이유) <br/>

<br/>

# [Mysql, JPA] 동시성 이슈 해결 (Syncronized, 낙관적 락, 비관적 락)

## 1. 문제 상황

**[저저번 글](https://github.com/tjdtls690/TIL/blob/main/Java/a_50_select_perform_improve.md)** 에서 **반정규화를 통한 조회 성능 개선**을 진행한 적이 있다. 간단히 개선 과정을 설명하자면, 각 Post의 Vote개수를 기준으로 정렬하는 작업이, 원래는 조회(인기순 목록 조회)하는 시점에 실제 Vote 테이블에서 각 Post의 Vote 개수를 구하고 그걸로 정렬하는 플로우였는데, 성능이 너무 안좋아서 Post 테이블에 vote_count 컬럼을 추가해서 insert(투표)하는 시점에 +, - 하는 형식으로 변경하여 성능을 개선하였다.

문제는 이런 식으로 성능을 개선하고 나니, 동시성 문제가 터질 수 있게 되었다는 것이다.

<br/>

## 2. 원인 분석

반정규화를 통한 성능 개선을 진행할 때, **투표하는 기능** 외에 **투표를 변경하는 기능**도 같이 영향을 받게 되었다. 이유는 조회 성능을 개선하기 위해, Post 뿐만 아니라 Post의 선택지에 해당하는 PostOption에도 vote_count 컬럼을 추가해주었기 때문이다.

그래서 **투표하는 기능**과 **투표를 변경하는 기능** 모두 동시성 이슈 발생 가능성이 생기게 되었는데, 이번엔 **투표 변경 기능**을 토대로 동시성 이슈를 해결해나갈 것이다.

먼저 현재의 상황을 ERD로 먼저 살펴보자. Post <-> PostOption <-> Vote 형식으로 참조하고 있다. 또한 Post와 PostOption 모두 vote_count 컬럼을 갖고 있다.

<img src="https://tjdtls690.github.io/assets/img/blog/동시성_이슈1.PNG">

<br/>

투표 변경 기능은 특정 Post의 투표 선택지에 해당하는 2개의 PostOption의 투표 개수(vout_count)를 조정하는 기능이다. 즉, Post_Option 테이블의 레코드인 A, B가 있을 때, A에 투표했던 것을 B에 투표하는 것으로 변경하는 기능이다.

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
        // 투표를 바꿀 게시글을 꺼낸다.
        final Post post = postRepository.findById(postId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        // 원래 투표했던 선택지를 꺼낸다
        final PostOption originPostOption = postOptionRepository.findById(originPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
        // 새롭게 투표할 선택지를 꺼낸다.
        final PostOption newPostOption = postOptionRepository.findById(newPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        // 원래 투표를 꺼낸다.
        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        // 원래 투표를 삭제한다.
        voteRepository.delete(originVote);
        // 새로운 투표를 생성해서 저장한다.
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        // 원래 선택지의 vote_count를 1 감소시킨다.
        originPostOption.decreaseVoteCount();
        // 새로운 선택지의 vout_count를 1 증가시킨다.
        newPostOption.increaseVoteCount();
    }
    
    // ...생략
}
```

<br/>

이 상태에서, 만약 post_option 레코드에서 **A의 vote_count가 2**, **B의 vote_count가 6**이라 가정하고 2가지의 케이스를 예시로 들어 동시성에 대해 설명해보려 한다. 

- **동시에 유저 1, 2 모두 A -> B**로 투표를 옮길 경우

원래 의도한 값인 **0, 8**이 아닌 **1, 7**이 나올 수도 있다. 밑의 흐름을 자세히 살펴보자.

|                         **Thread_1**                         |    post_option (A)<br />{id : 1, vote_count : 2}     |    post_option (B)<br />{id : 2, vote_count : 6}     |                           Thread_2                           |
| :----------------------------------------------------------: | :--------------------------------------------------: | :--------------------------------------------------: | :----------------------------------------------------------: |
| select * <br />from post_option <br />where id = 1;<br /><br /> **조회 결과 : {id : 1, vote_count : 2}** |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               | select * <br />from post_option <br />where id = 1;<br /><br /> **조회 결과 : {id : 1, vote_count : 2}** |
| select * <br />from post_option <br />where id = 2;<br /><br />**조회 결과 : {id : 2, vote_count : 6}** |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               | select * <br />from post_option <br />where id = 2;<br /><br />**조회 결과 : {id : 2, vote_count : 6}** |
| update set vote_count = 1<br />from post_option<br />where id = 1; | **{id : 1, vote_count : 1}**<br /><br />컬럼 값 갱신 |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 1}               |               {id : 2, vote_count : 6}               | update set vote_count = 1<br />from post_option<br />where id = 1; |
| update set vote_count = 7<br />from post_option<br />where id = 2; |               {id : 1, vote_count : 1}               | **{id : 2, vote_count : 7}**<br /><br />컬럼 값 갱신 |                                                              |
|                                                              |               {id : 1, vote_count : 1}               |               {id : 2, vote_count : 7}               | update set vote_count = 7<br />from post_option<br />where id = 2; |

<br/>

- **동시에 유저 1은 A -> B**로, **유저 2는 B -> A**로 투표를 옮길 경우

원래 의도한 값인 **2, 6**이 아닌 **3, 7**이 나와버리는 사태도 발생할 수 있다. 밑의 흐름을 자세히 살펴보자.

|                         **Thread_1**                         |    post_option (A)<br />{id : 1, vote_count : 2}     |    post_option (B)<br />{id : 2, vote_count : 6}     |                           Thread_2                           |
| :----------------------------------------------------------: | :--------------------------------------------------: | :--------------------------------------------------: | :----------------------------------------------------------: |
| select * <br />from post_option <br />where id = 1;<br /><br /> **조회 결과 : {id : 1, vote_count : 2}** |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               | select * <br />from post_option <br />where id = 2;<br /><br /> **조회 결과 : {id : 2, vote_count : 6}** |
| select * <br />from post_option <br />where id = 2;<br /><br />**조회 결과 : {id : 2, vote_count : 6}** |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 2}               |               {id : 2, vote_count : 6}               | select * <br />from post_option <br />where id = 1;<br /><br />**조회 결과 : {id : 1, vote_count : 2}** |
| update set vote_count = 1<br />from post_option<br />where id = 1; | **{id : 1, vote_count : 1}**<br /><br />컬럼 값 갱신 |               {id : 2, vote_count : 6}               |                                                              |
|                                                              |               {id : 1, vote_count : 1}               | **{id : 2, vote_count : 5}**<br /><br />컬럼 값 갱신 | update set vote_count = 5<br />from post_option<br />where id = 2; |
| update set vote_count = 7<br />from post_option<br />where id = 2; |               {id : 1, vote_count : 1}               | **{id : 2, vote_count : 7}**<br /><br />컬럼 값 갱신 |                                                              |
|                                                              | **{id : 1, vote_count : 3}**<br /><br />컬럼 값 갱신 |               {id : 2, vote_count : 7}               | update set vote_count = 3<br />from post_option<br />where id = 1; |

<br/>

**데이터의 정합성**이 전혀 맞지 않는 사태가 발생하게 된다.

<br/>

## 3. 문제 해결 과정

동시성을 해결할 수 있는 방법으론 총 3가지를 고려해보게 되었다. 결론부터 말하자면 **'비관적 락'** 을 사용하기로 판단을 했는데, 그 이유를 지금부터 자세히 설명하겠다.

### 1. Syncronized (암시적 Lock)

Synchronized 키워드는 여러개의 스레드가 한개의 자원을 사용하고자 할 때, **현재 자원을 사용하고 있는 해당 스레드를 제외하고 나머지 스레드들은 자원에 접근 할 수 없도록 막는 개념**입니다.

<br/>

- **Syncronized 메서드를 적용한 경우**

해당 메서드는 동시에 여러 스레드가 사용할 수 없다. 이유는 어차피 **VoteService는 스프링 컨테이너가 싱글톤으로 관리**해주기 때문에, synchronized가 걸린 메서드도 단 한 개밖에 존재하지 않는다. 따라서 한 서버 내에서라면, 모든 스레드를 동기화 할 수 있다.

```java
@Service
@RequiredArgsConstructor
public class VoteService {

    // ...생략
    
    @Transactional
    public synchronized void changeVote( // synchronized 키워드 추가
            final Member member,
            final Long postId,
            final Long originPostOptionId,
            final Long newPostOptionId
    ) {
        final Post post = postRepository.findById(postId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        final PostOption originPostOption = postOptionRepository.findById(originPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
        final PostOption newPostOption = postOptionRepository.findById(newPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        voteRepository.delete(originVote);
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        originPostOption.decreaseVoteCount();
        newPostOption.increaseVoteCount();
    }
    
    // ...생략
}
```

<br/>

- **Syncronized 블록을 적용한 경우**

Syncronized 블록은 어디서부터 어디까지 감쌀지 자신이 알아서 설정해주면 된다. 그래서 Syncronized 메서드보다 더 효율적으로 관리해줄 수 있다. 물론 지금은 메서드 내부 코드 전체를 감쌌고, Syncronized 블록의 키를 this로 잡았기 때문에, 위의 Syncronized 메서드와 동일하게 되었다.

위의 Syncronized 메서드와 동일하다고 한 이유는, Syncronized 블록의 키로 설정된 this, 즉 VoteService는 스프링 컨테이너에 의해 싱글톤으로 관리되어 단 한개만 존재하게 되고, 메서드 내부 전체를 블록으로 감쌌기 때문이다. 따라서 해당 메서드 자체가 한 서버 내에서라면, 모든 스레드를 동기화 할 수 있다는 점이 같다.

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
        synchronized(this) { // synchronized 블록 추가
            final Post post = postRepository.findById(postId)
                	.orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

            final PostOption originPostOption = postOptionRepository.findById(originPostOptionId)
                    .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
            final PostOption newPostOption = postOptionRepository.findById(newPostOptionId)
                    .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

            final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                    .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

            voteRepository.delete(originVote);
            final Vote vote = post.makeVote(member, newPostOption);
            voteRepository.save(vote);
            originPostOption.decreaseVoteCount();
            newPostOption.increaseVoteCount();
        }
    }
    
    // ...생략
}
```

<br/>

#### 하지만 이 방법은 사용하지 않기로 판단했다.

1. **성능 상 너무 비효율적인 방법이다.**

   - 한 유저가 Syncronized 메서드 혹은 블록을 사용중이면, 다른 유저가 접근하려는 **레코드가 다르더라도** 대기해야한다.

     <br/>

2. **여러대의 프로세스가 접근하는 것은 막을 수 없다.**

   - 즉 서버가 여러대일 때, 여러대의 프로세스가 접근하는 것은 막을 수 없다는 것이다.

   - Syncronized 키워드는 자바 코드 레벨에 존재하는 것이기 때문에, 여러 서버에 Syncronized 키워드가 각각 존재하게 될 것이다.

   - 각 Syncronized 키워드는 그 서버 안에서만 여러 스레드가 접근하는 것을 막아줄 수 있는 것이다.

     <br/>

3. **@Transactional과 같이 사용 시, 데이터 정합성을 보장 할 수 없다. (메서드 레벨로 Syncronized를 걸어도 마찬가지)**

   - 동작 순서

     1. 트랜잭션 시작
     2. changeVote 메서드 실행 (Syncronized 적용으로 인해 다른 스레드 접근 불가)
     3. vote_count 값 변경
     4. changeVote 메서드 종료 (다른 스레드 접근 가능)
     5. 트랜잭션 종료 및 커밋

   - 여기서 **changeVote 메서드가 종료되고 트랜잭션이 커밋하기 전 시점**에, 대기하고 있던 다른 스레드가 종료된 changeVote 메서드에 들어와서 작업을 진행해버릴 수도 있기 때문이다.

     <br/>

### 2. 낙관적 잠금 (Optimistic Lock)

낙관적 잠금은 현실적으로 데이터 갱신 시 경합이 별로 발생하지 않을 것이라고 보고 잠금을 거는 방식이다.

조회하는 시점엔 평소처럼 락을 걸지 않고, 조회한 레코드를 update 하는 시점에 버전 값을 비교한다. 만약 update 할 때 넣으려는 Version이 이미 존재한다면, 먼저 Hibernate에서 **'StaleStateException'** 예외를 발생시키고, Spring에서 이 예외를 **'ObjectOptimisticLockingFailureException'** 예외로 감싸서 응답합니다.

사실상 **충돌 감지** 기능에 가깝다고 볼 수 있다.

낙관적 잠금의 동작 방식에 대한 한 예를 들어보겠다. Tread_1과 Tread_2가 동시에 id가 1인 PostOption의 voteCount를 1 증가시키려 하는 상황이다.

|                           Tread_1                            |    post_option<br />{id : 1, vote_count : 2, version : 1}    |                           Tread_2                            |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| select *<br />from post_option<br />where id = 1;<br /><br />**조회 결과 : {id : 1, vote_count : 2, version : 1}** |            {id : 1, vote_count : 2, version : 1}             |                                                              |
|                                                              |            {id : 1, vote_count : 2, version : 1}             | select *<br />from post_option<br />where id = 1;<br /><br />**조회 결과 : {id : 1, vote_count : 2, version : 1}** |
| update set vote_count = 3 and version = 2<br />from post_option<br />where id = 1; | **{id : 1, vote_count : 3, version : 2}**<br /><br />vote_count와 version 갱신 |                                                              |
|                                                              |            {id : 1, vote_count : 3, version : 2}             | update set vote_count = 3 and version = 2<br />from post_option<br />where id = 1;<br /><br />**현재 version이 1에서 이미 바뀌어버린 것을 확인 후,<br />ObjectOptimisticLockingFailureException 예외 발생**<br />**이후 롤백** |

<br/>

- **JPA 낙관적 잠금을 적용한 경우**

```java
@Entity
@Table(uniqueConstraints = {@UniqueConstraint(columnNames = {"post_id", "sequence"})})
public class PostOption extends BaseEntity {

    // ...생략
    
    // 낙관적 락에서 사용될 version 컬럼 추가
    @Version
    private Long version;
    
    // ...생략
}
```

```java
public interface PostOptionRepository extends JpaRepository<PostOption, Long> {
    @Lock(LockModeType.OPTIMISTIC) // 낙관적 락 설정
    @Query("SELECT po FROM PostOption po where po.id = :postOptionId")
    Optional<PostOption> findByIdForUpdate(@Param("postOptionId") final Long postOptionId);
}
```

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

        // 낙관적 잠금을 적용한 findByIdForUpdate() 메서드로 변경
        final PostOption originPostOption = postOptionRepository.findByIdForUpdate(originPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
        // 낙관적 잠금을 적용한 findByIdForUpdate() 메서드로 변경
        final PostOption newPostOption = postOptionRepository.findByIdForUpdate(newPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        voteRepository.delete(originVote);
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        originPostOption.decreaseVoteCount();
        newPostOption.increaseVoteCount();
    }
    
    // ...생략
}
```

<br/>

**Spring의 Transaction 처리**는 기본적으로 **checked exception은 rollback을 진행하지 않고**, **unchecked exception은 rollback을 진행**한다. 물론 checked exception이어도 rollback이 되게끔, 혹은 unchecked exception이어도 rollback이 안되게끔 설정을 해줄 수 있다는 것도 기본 상식으로 알고 있자.

**[자바 공부를 어떻게 하길래, "언체크드 예외 발생시 트랜잭션 롤백?" - 백기선님 youtube 영상](https://www.youtube.com/watch?v=_WkMhytqoCc)** 참고

만약 낙관적 락을 사용하는 중에, 동시성 이슈가 터지면서 ObjectOptimisticLockingFailureException 예외가 발생한다면, **ObjectOptimisticLockingFailureException 는 unchecked exception**이기 때문에 알아서 rollback이 진행될 것이다.

<br/>

#### 하지만 이 방법도 사용하지 않기로 판단했다.

- **동시성 이슈가 많이 발생할 기능이라면 오히려 성능이 떨어질 수 있다.**

  - 애초에 데이터 갱신 시 경합이 별로 발생하지 않을 것이라고 볼 때 사용하는 락이다.

  - 만약 경합이 많이 발생하는 기능이라면, 그만큼 불필요한 rollback이 자주 일어날 것이다. (성능 저하)

<br/>

### 3. 비관적 잠금 (Pessimistic Lock)

비관적 잠금은 현실적으로 데이터 갱신 시 경합이 많이 발생할 것이라고 보고 잠금을 거는 방식이다.

현재 thread가 조회하는 시점부터 락을 걸어버리고, 해당 레코드를 update하고 나올 때까지 해당 레코드에 접근하려는 다른 thread들은 기다려야 한다.

비관적 락에는 **공유 락(Shared Lock), 베타 락(혹은 쓰기 락, Exclusive Lock)** 이 있는데, 이 중 베타 락을 사용하기로 판단했다. 이유는 특정 레코드를 조회를 한 뒤, 그 레코드를 update 할 때까지 다른 스레드가 접근하면 안되기 때문이다.

베타 락을 이용한 비관적 잠금의 동작 방식에 대한 한 예를 들어보겠다. 여기도 낙관적 잠금의 예와 마찬가지로, Tread_1과 Tread_2가 동시에 id가 1인 PostOption의 voteCount를 1 증가시키려 하는 상황이다.

|                           Tread_1                            |        post_option<br />{id : 1, vote_count : 2}        |                           Tread_2                            |
| :----------------------------------------------------------: | :-----------------------------------------------------: | :----------------------------------------------------------: |
| select *<br />from post_option<br />where id = 1;<br /><br />**id가 1인 레코드의 Exclusive Lock 획득<br />조회 결과 : {id : 1, vote_count : 2}** |                {id : 1, vote_count : 2}                 |                                                              |
|                                                              |                {id : 1, vote_count : 2}                 | **id가 1인 레코드에 첫 select 시도**<br /><br />**이미 Thread_1이 id가 1인 레코드의 <br />Exclusive Lock을 갖고 있으므로<br />대기 상태로 돌입** |
| update set vote_count = 3<br />from post_option<br />where id = 1; | **{id : 1, vote_count : 3}**<br /><br />vote_count 갱신 |                                                              |
|                                                              |                {id : 1, vote_count : 3}                 | **Tread_1의 작업이 마무리 되어 대기 끝**<br /><br />select *<br />from post_option<br />where id = 1;<br /><br />**조회 결과 : {id : 1, vote_count : 3}** |
|                                                              | **{id : 1, vote_count : 4}**<br /><br />vote_count 갱신 | update set vote_count = 4<br />from post_option<br />where id = 1; |

<br/>

- **JPA 비관적 잠금의 Exclusive Lock을 적용한 경우**

```java
public interface PostOptionRepository extends JpaRepository<PostOption, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE) // 비관적 잠금의 Exclusive Lock 설정
    @Query("SELECT po FROM PostOption po where po.id = :postOptionId")
    Optional<PostOption> findByIdForUpdate(@Param("postOptionId") final Long postOptionId);
}
```

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

        // 비관적 잠금을 적용한 findByIdForUpdate() 메서드로 변경
        final PostOption originPostOption = postOptionRepository.findByIdForUpdate(originPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));
        // 비관적 잠금을 적용한 findByIdForUpdate() 메서드로 변경
        final PostOption newPostOption = postOptionRepository.findByIdForUpdate(newPostOptionId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        final Vote originVote = voteRepository.findByMemberAndPostOption(member, originPostOption)
                .orElseThrow(() -> new NotFoundException(VoteExceptionType.NOT_FOUND));

        voteRepository.delete(originVote);
        final Vote vote = post.makeVote(member, newPostOption);
        voteRepository.save(vote);
        originPostOption.decreaseVoteCount();
        newPostOption.increaseVoteCount();
    }
    
    // ...생략
}
```

<br/>

#### 비관적 잠금을 사용하기로 판단한 이유

결국 지금까지 고려해본 3개의 동시성 해결 방법 중 비관적 잠금을 선택했다.

물론 비관적 잠금은 처음부터 락을 걸고 시작하기 때문에, 성능이 안좋다고 여길 수도 있다. 하지만 처음부터 락을 확실히 걸어주는 만큼, **동시성 이슈가 많이 발생하는 기능에서 낙관적 잠금처럼 불필요한 롤백이 일어나지 않을 것**이기에 오히려 더 효율적이겠다고 판단하였다.

또한 Mysql의 5.5 버전부터 기본 스토리지 엔진이 된 **InnoDB 엔진**은, 5.5 버전 이전까지 기본 스토리지 엔진이었던 **MyISAM 엔진**에 비해 효율적이다. 이유는 **MyISAM 엔진은 기본 락이 테이블 락**이고 **InnoDB 엔진은 기본 락이 레코드 락**이기 때문이다. 따라서 비관적 잠금이어도 2개의 PostOption 레코드만 락을 거는 것이기에 충분히 감당할만 하다고 판단하였다.

<br/>

동시성 이슈를 잘 해결하고 기분이 좋았는데 새로운 이슈가 터졌다. 바로 **'데드락 이슈'** 이다. 잠금을 걸어주게 되면서 데드락 이슈 발생 가능성도 새롭게 생기게 된 것이다. 데드락 문제 해결에 대한 포스팅은 다음 글에서 진행하도록 하겠다.