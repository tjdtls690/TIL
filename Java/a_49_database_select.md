# 목차

1. [문제](#1-문제) <br/>
2. [해결책](#2-해결책) <br/>
   1. [JdbcTemplate](#1-jdbctemplate) <br/>
   2. [NamedParameterJdbcTemplate](#2-namedparameterjdbctemplate) <br/>

<br/>

# [우아한테크코스 5기] 레벨2 - 여러 조회를 한 번에 진행하기

## 1. 문제

리뷰어와 피드백을 주고 받으면서 깨달았던 점은, 지금까지 여러 ID를 통해 데이터를 조회할 때 너무나 자연스럽게 Stream으로 돌려주면서 하나하나 조회를 해주고 있었다. 당연히 드는 비용의 효율은 낮을 수밖에 없었다.

```java
public CartItemEntity findById(Long id) {
        final String sql = "SELECT * FROM cart_item WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, rowMapper, id);
}
```

```java
 private Set<CartItem> getCartItems(final OrderRequest orderRequest) {
        return orderRequest.getCartItemIds().stream()
                .map(CartItemOrderRequest::getCartItemId)
                .map(cartItemRepository::findById)
                .collect(Collectors.toUnmodifiableSet());
 }
```

이러한 문제점을 해결해주려면 어떤 방법들이 있을까?

<br/>

## 2. 해결책

### 1) JdbcTemplate

```java
public List<CartItemEntity> findByIds(final List<Long> cartItemIds) {
        final String inClause = String.join(",", Collections.nCopies(cartItemIds.size(), "?"));
        String sql = "SELECT * FROM cart_item WHERE id IN (" + inClause + ")";
        return jdbcTemplate.query(sql, rowMapper, cartItemIds.toArray());
}
```

join 메서드를 사용해서 "1,2,3..." 형식으로 id들을 묶고 직접 sql 구문에 삽입하는 방법이 있다.

<br/>

### 2) NamedParameterJdbcTemplate

```java
public List<CartItemEntity> findByIds(final List<Long> cartItemIds) {
    String sql = "SELECT * FROM cart_item WHERE id IN (:ids)";

    MapSqlParameterSource parameters = new MapSqlParameterSource();
    parameters.addValue("ids", cartItemIds);

    return namedParameterJdbcTemplate.query(sql, parameters, rowMapper);
}
```

이처럼 NamedParameterJdbcTemplate을 사용하면 문자열의 join 과정이 필요없이 해결해줄 수 있다.