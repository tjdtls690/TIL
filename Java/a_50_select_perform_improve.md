# 목차

1. [문제 발생](#1-문제-발생) <br/>
2. [원인 분석](#2-원인-분석) <br/>
3. [문제 해결](#3-문제-해결) <br/>

<br/>

# [Spring, MySQL] 비즈니스 로직의 흐름을 변경하여 조회 성능 개선하기

## 1. 문제 발생

진행했던 프로젝트에서 우리는 대용량 데이터 테스트를 위해 개발 서버에 **2000만 건 이상의 데이터**를 넣었다. 

그리고 **'인기순 전체 게시글 목록 조회'** 라는 중요한 기능이 있다. 전체의 Post를 Vote(투표)가 많은 순으로 정렬해서, Spring Data JPA의 Pagination을 활용하여 10개씩 가져오는 기능이다.

문제는 개발서버에서 QA를 할 때, 이 조회 기능을 한 번씩 실행할 때마다 **8초 이상**이 걸린다는 것이다. 우리 팀은 운영서버로 배포하기 전에 항상 개발서버에서 QA 타임을 가지는데, 속도가 너무 느려서 시간이 오래 걸리는 것이다.

물론 실제 운영 서버엔 회원수 200명, 게시글 수 700건, 투표 수 7000건 정도라서 좀 이른 감이 있지만, 이 기회에 개발 서버를 통해 대용량 데이터일 때의 성능 개선을 시도해보고자 하였다.

<br/>

## 2. 원인 분석

먼저 엔티티 연관관계를 간단히 살펴보면,

- **Post** -> @OneToMany -> **PostOption** -> @OneToMany -> **Vote**
- **Post** <- @ManyToOne <- **PostOption** <- @ManyToOne <- **Vote**

이처럼 Post부터 Vote까지 전부 양방향 매핑관계를 맺어주고 있는 상태이다. 이 관계를 참고하면서 **'인기순 전체 게시글 목록 조회'** 기능의 실제 코드를 보자. 먼저 Post의 Service 클래스이다.

```java
@RequiredArgsConstructor
@Service
public class PostGuestService {

    private static final int BASIC_PAGE_SIZE = 10;

    private final PostRepository postRepository;
    private final PostCategoryRepository postCategoryRepository;

    @Transactional(readOnly = true)
    public List<PostResponse> getPosts(
            final int page,
            final PostClosingType postClosingType,
            final PostSortType postSortType,
            final Long categoryId
    ) {
        final Pageable pageable = PageRequest.of(page, BASIC_PAGE_SIZE);
        final List<Post> posts =
                postRepository.findPostsWithFilteringAndPaging(postClosingType, postSortType, categoryId, pageable);
        return convertToResponses(posts);
    }
    
    ... 생략
}
```

<br/>

**getPosts() 메서드의 파라미터 설명**

1. page : 이동할 페이지 번호
2. postClosingType : 게시글 투표 마감 여부 (전체, 마감, 마감안됨)
3. postSortType : 전체 게시글 정렬 기준 (최신순, 인기순)
4. categoryId : 카테고리 ID (IT, 고민, 운동, 연애 등등...)

이제 Querydsl이 있는 findPostsWithFilteringAndPaging() 메서드 내부 코드를 살펴보자.

```java
@RequiredArgsConstructor
@Repository
public class PostCustomRepositoryImpl implements PostCustomRepository {

    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public List<Post> findPostsWithFilteringAndPaging(
            final PostClosingType postClosingType,
            final PostSortType postSortType,
            final Long categoryId,
            final Pageable pageable
    ) {
        return jpaQueryFactory
                .selectDistinct(post)
                .from(post)
                .join(post.writer).fetchJoin()
                .leftJoin(post.postCategories, postCategory)
                .where(
                        categoryIdEq(categoryId),
                        deadlineEq(postClosingType),
                        post.isHidden.eq(false)
                )
                .orderBy(orderBy(postSortType))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();
    }

    private BooleanExpression categoryIdEq(final Long categoryId) {
        return categoryId == null ? null : postCategory.category.id.eq(categoryId);
    }
    
    private BooleanExpression deadlineEq(final PostClosingType postClosingType) {
        final LocalDateTime now = LocalDateTime.now();
        return switch (postClosingType) {
            case PROGRESS -> post.postDeadline.deadline.after(now);
            case CLOSED -> post.postDeadline.deadline.before(now);
            default -> null;
        };
    }

    private OrderSpecifier orderBy(final PostSortType postSortType) {
        return switch (postSortType) {
            case LATEST -> post.createdAt.desc();
            case HOT -> new OrderSpecifier<>(
                    Order.DESC,
                    JPAExpressions.select(vote.id.count())
                            .from(vote)
                            .where(vote.postOption.id.in(
                                    JPAExpressions.select(postOption.id)
                                            .from(postOption)
                                            .where(postOption.post.id.eq(post.id))
                            ))
            );
            default -> OrderByNull.DEFAULT;
        };
    }
}
```

<br/>

각 Post의 Vote 개수로 정렬하기 위해 orderBy() 메서드의 HOT case(인기순) 코드를 보면 압권이다. 2000만 건 데이터가 있는 vote 테이블을 계속 스캔해야한다.

이제 이 기능을 Swagger로 테스트해보자. orderBy() 메서드를 보면 인기순(PostSortType == HOT)이 훨씬 더 성능이 느릴 게 뻔하니 인기순으로 테스트 해보겠다.

**현재 개발서버 각 테이블 데이터 개수**

1. post : **10만 건**
2. post_option : **20만 건**
3. vote : **2000만 건**

**테스트 시, 전달한 인자 값**

1. page : **3000**
2. postClosingType : **PROGRESS** (마감 안된 것)
3. postSortType : **HOT** (인기순)
4. categoryId : **1** (신경 안써도 되는 부분)

성능 결과부터 보자.

```sql
[Query Count] = [13]
[Total Time] = [7908ms]
```

쿼리는 총 13개가 발생했고, 시간은 총 8초가 걸렸다. 왜 이렇게 속도가 느린지 분석하기 위해, 발생한 13개의 쿼리 중 성능에 99%의 영향을 끼치고 있는 주요 쿼리 1개를 살펴보자.

```sql
== 발생한 주요 쿼리 ==

	select
        distinct p1_0.id,
        (select
            count(*)
        from
            comment c
        where
            c.post_id = p1_0.id),
        p1_0.created_at,
        p1_0.is_hidden,
        p1_0.content,
        p1_0.title,
        p1_0.deadline,
        p1_0.updated_at,
        p1_0.member_id,
        w1_0.id,
        w1_0.birth_year,
        w1_0.created_at,
        w1_0.gender,
        w1_0.nickname,
        w1_0.social_id,
        w1_0.social_type,
        w1_0.updated_at
    from
        post p1_0
    join
        member w1_0
            on w1_0.id=p1_0.member_id
    left join
        post_category p2_0
            on p1_0.id=p2_0.post_id
    where
        p2_0.category_id=1
		and p1_0.deadline>'2023-09-28T20:59:49.237815'
        and p1_0.is_hidden=false
    order by
        (select
            count(v1_0.id)
        from
            vote v1_0
        where
            v1_0.post_option_id in
         (select
                p4_0.id
            from
                post_option p4_0
            where
                p4_0.post_id=p1_0.id)
        ) desc limit 12340,10;
```

<br/>

현재 vote 테이블엔 2000만건의 데이터가 들어가있는데, 정렬의 기준이 될 vote의 개수를 구하기 위해 order by에서 중첩 서브쿼리로 vote 개수를 구하는 모습이다.

이 쿼리의 실행계획은 너무 커서 여기에 첨부하진 않겠지만, 데이터가 많은 테이블에서 딱히 Using temporary, Using filesort가 일어나는 것도 아니다. 그럼에도 불구하고, 대용량 데이터를 스캔해서 각 Post의 vote 개수를 구하고, 그걸로 vote 건너건너의 Post를 정렬하려니 오래 걸릴 수밖에 없다.

<br/>

## 3. 문제 해결

따라서 생각해낸 해결책은, **'인기순으로 조회할 때, vote 테이블을 스캔하게 하지 않는 것'** 이다.

**해결 과정**

1. Post 테이블에 vote_count 컬럼을 새로 추가
2. 투표를 할 때마다 Post 테이블의 vote_count 컬럼을 1씩 증가
3. 조회할 때, Post 테이블의 vote_count로 정렬

이러면 조회할 땐 Vote 테이블을 스캔하지 않아도 되어 성능이 훨씬 더 올라가게 된다. 이제 실제로 개선한 코드와 성능 결과를 살펴보자.

먼저 VoteService에서 새롭게 추가된, '투표를 할 때마다 Post의 voteCount가 1 증가'하는 로직이다.

```java
@Service
@Transactional
@RequiredArgsConstructor
public class VoteService {

    private final VoteRepository voteRepository;
    private final PostRepository postRepository;
    private final PostOptionRepository postOptionRepository;
    private final MemberMetricRepository memberMetricRepository;

    public void vote(
            final Member member,
            final Long postId,
            final Long postOptionId
    ) {
        final Post post = postRepository.findByIdForUpdate(postId)
                .orElseThrow(() -> new NotFoundException(PostExceptionType.NOT_FOUND));

        validateAlreadyVoted(member, post);

        final PostOption postOption = postOptionRepository.findByIdForUpdate(postOptionId)
                .orElseThrow(() -> new NotFoundException(PostOptionExceptionType.NOT_FOUND));
        final MemberMetric memberMetric = memberMetricRepository.findByMemberIdForUpdate(member.getId())
                .orElseThrow(() -> new NotFoundException(MemberExceptionType.NOT_FOUND_METRIC));
        final int voteCount = voteRepository.countByMember(member);

        final Vote vote = post.makeVote(member, postOption);
        voteRepository.save(vote);
        memberMetric.updateVoteCount(voteCount + 1);
        post.increaseVoteCount(); // 여기서 Post의 vote_count 컬럼 값이 1 증가한다.
        postOption.increaseVoteCount();
    }
    
    ...생략
}
```

<br/>

Querydsl 코드에서 변경된 부분을 살펴보자.

```java
@RequiredArgsConstructor
@Repository
public class PostCustomRepositoryImpl implements PostCustomRepository {
    ...생략
        
    private OrderSpecifier orderBy(final PostSortType postSortType) {
        return switch (postSortType) {
            case LATEST -> post.id.desc();
            case HOT -> post.voteCount.desc();
            default -> OrderByNull.DEFAULT;
        };
    }
    
    ...생략
}
```

<br/>

위에서 개선하기 전의 orderBy() 메서드와 비교해보면, HOT case의 코드가 post의 voteCount로 정렬하는 것으로 아주 간결하게 바뀐 모습을 볼 수 있다. 그리고 Post의 vote_count 컬럼에 인덱스를 걸어준다.

이제 swagger로 다시 **'인기순 전체 게시글 목록 조회'** 기능을 테스트 해보자.

```sql
발생한 주요 쿼리

select
        distinct p1_0.id,
        p1_0.created_at,
        p1_0.is_hidden,
        p1_0.content,
        p1_0.title,
        p1_0.deadline,
        p1_0.updated_at,
        p1_0.vote_count,
        p1_0.member_id,
        w1_0.id,
        w1_0.alarm_checked_at,
        w1_0.birth_year,
        w1_0.created_at,
        w1_0.gender,
        w1_0.nickname,
        w1_0.roles,
        w1_0.social_id,
        w1_0.social_type,
        w1_0.updated_at 
    from
        post p1_0 
    join
        member w1_0 
            on w1_0.id=p1_0.member_id 
    left join
        post_category p2_0 
            on p1_0.id=p2_0.post_id 
    where
        p2_0.category_id=1
		and p1_0.deadline>'2023-09-28T21:40:45.365115'
        and p1_0.is_hidden=false
    order by
        p1_0.vote_count desc -- 개선하기 전에 엄청나게 중첩된 서브쿼리가 발생한 것에 비해서 훨씬 개선된 모습이다.
	limit 12340,10;
```

```sql
성능 결과

[Total Time] = [270ms]
```

**<u>8초 -> 0.27초</u>** 즉, **<u>29~30배</u>** 정도 성능이 개선된 모습을 볼 수 있다. 굉장히 성공적인 성능 개선이라고 생각한다.

<br/>

물론 이러면 조회 성능이 올라가는 대신 투표하는 insert 기능의 성능이 저하되는 것 아니냐 생각할 수도 있다. 하지만 insert의 성능 변화는 거의 없다고 보면 된다. Post의 voteCount를 1 증가시키는 작업만 추가된 것 뿐이기 때문이다.

실제로 insert 성능(투표 기능)을 swagger로 테스트 해본 결과, 차이가 아예 없었다.

이렇게 조회 성능을 향상시키는 작업을 하면서 항상 느끼는 점은, insert와 select 성능의 trade off는 **아주 조금의 손해를 보면서 아주 많은 이익을 볼 수 있는 가성비 작업**이라는 것이다.

