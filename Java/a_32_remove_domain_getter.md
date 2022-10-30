# 목차

1. [문제 발생 - 나도 모르게 사용한 도메인 간의 getter 메서드](#1-문제-발생---나도-모르게-사용한-도메인-간의-getter-메서드) <br/>
2. [문제 해결 - 객체에 메시지를 보내자](#2-문제-해결---객체에-메시지를-보내자) <br/>
3. [그래도 아직 해결하지 못한 로직... - 역량을 더 끌어 올려야 겠다고 통감한 부분](#3-그래도-아직-해결하지-못한-로직---역량을-더-끌어-올려야-겠다고-통감한-부분) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 1주차 - 도메인 간의 getter사용을 없애자

<br/>

> **저번 10월 9일에 '계층을 넘어갈 땐, getter와 setter 대신 DTO' 라는 글을 작성한 적이 있다.**
>
> **이번엔 계층간이 아닌 도메인간의 getter 사용에 대한 개인적인 견해와,**
>
> **모든 코드를 최종 검토하면서 getter 를 줄이려고 노력하는 과정에서 배운 부분들을 적어보려 한다.**

<br/>

## 1. 문제 발생 - 나도 모르게 사용한 도메인 간의 getter 메서드

1주차엔 총 7문제가 나왔고 알고리즘 문제 형식이지만, 일반 코딩테스트처럼 무조건 **빨리 푸는 것에만 신경을 써야하는 문제가 아니라고 느꼈다.** 분명 알고리즘 문제이지만 기간은 1주일이나 주어졌다. 그래서 최대한 객체지향 설계를 추구하려고 노력하면서 알고리즘 문제를 풀었다.



3문제 정도의 비즈니스 로직에서 도메인간의 getter 메서드 사용이 발견되었지만, 그 중 한 문제를 예로 들어 설명해보려 한다. 어떤 한 문제의 비즈니스 로직 중 일부분의 예시이다.

```java
public class DuplicateAccountEmails {
    private final Set<UserEmail> duplicateAccountEmails;
    
    public DuplicateAccountEmails() {
        duplicateAccountEmails = new HashSet<>();
    }
    
    ... 생략
        
    // 주목할 메서드
    private void saveDuplicateTwoLetterNameAccountEmail(final Map<UserName, UserEmail> checkDuplicateAccounts, final User twoLetterNameAccount) {
        final UserName twoLetterName = twoLetterNameAccount.getUserName(); // getter 사용
        
        if (checkDuplicateAccounts.containsKey(twoLetterName)) {
            duplicateAccountEmails.add(checkDuplicateAccounts.get(twoLetterName));
            duplicateAccountEmails.add(twoLetterNameAccount.getUserEmail()); // getter 사용
        }
    }
    
    private Map<UserName, UserEmail> parseCheckDuplicateAccounts(final List<User> twoLetterNameUsers) {
        return twoLetterNameUsers.stream() // getter 사용
                .collect(Collectors.toMap(User::getUserName, User::getUserEmail, (firstUserEmail, secondUserEmail) -> firstUserEmail));
    }
    
    ... 생략
}
```

```java
public class User {
    private final UserName userName;
    private final UserEmail userEmail;
    
    ... 생략
    
    public UserName getUserName() { // getter
        return userName;
    }
    
    public UserEmail getUserEmail() { // getter
        return userEmail;
    }
}
```

<br/>

예시를 보면 알겠지만 User 클래스의 getter 메서드들을 통해 데이터를 꺼낸 뒤, duplicateAccountEmails 라는 Set 자료구조에 데이터를 추가하는 모습이다. 분명 설계 및 구현할 때부터 최대한 getter 사용을 지양했다고 생각했다. 나도 모르게 무의식적으로 쓴 모습을 보니 아직 갈길이 멀었다는 생각이 든다.



먼저 내가 왜 무의식적으로 저 getter 메서드들을 사용했는지 그 이유부터 찾으려고 계속 고민을 해보았고, 어느정도 결론이 나왔다. **getter 메서드를 쓰면, 더 적고 간결한 코드로 기능 구현을 할 수 있다고 생각했기 때문**이다. 



실제로 지금 로직을 보면 getter 로 빼낸 데이터들을, duplicateAccountEmails 라는 Set 자료구조에 보기 쉽게 추가해주는 모습이다. 코드도 User 클래스에선 getter 외엔 별다른 코드가 없어도 된다. **자칫하면 이게 더 간결하고 더 가독성이 뛰어나 보여서 좋아보일 수 있다.** 하지만 난 알고있다. 유지보수 측면에서 이러한 로직은 굉장히 안좋은 로직이라는 것을.



객체는 자신만이 가지고 있는 상태 즉, 인스턴스 변수가 존재하고, 각 객체는 자신의 인스턴스 변수에 관련된 로직(역할)들을 구현하는 것이 좋다. 그 상태에서 getter 가 아닌, 해당 역할을 맡고있는 객체에 메시지를 보내는 형태로 로직을 구현해야 한다. 그래야 **각 도메인들이 상호간의 의존성을 최소한인 상태로 유지하게 되고, 기능을 개선하거나 확장하는 작업 즉, 유지보수가 수월해진다.**



이제 저 더러운 getter 사용 로직들을 빨리 리팩토링 해보자.

<br/>

## 2. 문제 해결 - 객체에 메시지를 보내자

먼저 getter 메서드 사용 로직을 리팩토링 한 코드를 예시로 보자면,

```java
public class DuplicateAccountEmails {
    private final Set<UserEmail> duplicateAccountEmails;
    
    public DuplicateAccountEmails() {
        duplicateAccountEmails = new HashSet<>();
    }
    
    ... 생략
    
    private void saveDuplicateTwoLetterNameAccountEmail(final Map<UserName, UserEmail> checkDuplicateAccounts, final User twoLetterNameAccount) {
        if (twoLetterNameAccount.isInADuplicateAccountEmail(checkDuplicateAccounts)) {
            // User 객체에 메시지를 보내서 결과 값 반환 받기
            final UserEmail duplicateAccountEmail = twoLetterNameAccount.findDuplicateAccountEmail(checkDuplicateAccounts);
            // 결과 값을 Set 자료구조인 duplicateAccountEmails 에 추가하기
            duplicateAccountEmails.add(duplicateAccountEmail); 
            // duplicateAccountEmails 자체를 User 객체에게 파라미터로 전달하면서 UserName 추가해주기
            twoLetterNameAccount.addDuplicateAccountEmail(duplicateAccountEmails);
        }
    }
    
    private Map<UserName, UserEmail> parseCheckDuplicateAccounts(final List<User> twoLetterNameUsers) {
        return twoLetterNameUsers.stream()
                .collect(Collectors.toMap(User::getUserName, User::getUserEmail, (firstUserEmail, secondUserEmail) -> firstUserEmail));
    }
    
    ... 생략
}
```

```java
public class User {
    private final UserName userName;
    private final UserEmail userEmail;
    
    ... 생략
    
    // 추가된 메서드
    public boolean isInADuplicateAccountEmail(final Map<UserName, UserEmail> checkDuplicateAccounts) {
        return checkDuplicateAccounts.containsKey(userName);
    }
    
    // 추가된 메서드
    public UserEmail findDuplicateAccountEmail(final Map<UserName, UserEmail> checkDuplicateAccounts) {
        return checkDuplicateAccounts.get(userName);
    }
    
    // 추가된 메서드
    public void addDuplicateAccountEmail(final Set<UserEmail> duplicateAccountEmails) {
        duplicateAccountEmails.add(userEmail);
    }
    
    public UserName getUserName() {
        return userName;
    }
    
    public UserEmail getUserEmail() {
        return userEmail;
    }
}

```

<br/>

위 예시에서 리팩토링 한 방법은 2가지이다.

1. **Set 에 추가할 데이터를 User 객체에 메시지를 보내서 받아온다.**
2. **Set 을 User 객체에 파라미터로 전달하면서 User 객체 안에서 Set 에 데이터를 추가한다.**

이렇게 리팩토링을 하면서 느낀 점은, 계층간의 데이터 전달이 아닌 **'도메인간의 데이터 전달은 개발자의 역량에 따라 getter 를 전혀 쓰지 않고 설계가 가능하다'**라는 것이다.

<br/>

## 3. 그래도 아직 해결하지 못한 로직... - 역량을 더 끌어 올려야 겠다고 통감한 부분

DuplicateAccountEmails 클래스의 saveDuplicateTwoLetterNameAccountEmail 메서드는 이제 해결 되었다. 그러나 아직 **parseCheckDuplicateAccounts 메서드**의 getter 메서드를 해결하지 못했다. 

```java
private Map<UserName, UserEmail> parseCheckDuplicateAccounts(final List<User> twoLetterNameUsers) {
    return twoLetterNameUsers.stream()
        .collect(Collectors.toMap(User::getUserName, User::getUserEmail, (firstUserEmail, secondUserEmail) -> firstUserEmail));
}
```

<br/>

사실 이 로직의 User::getUserName, User::getUserEmail 같은 getter 메서드들을 어떻게 리팩토링을 할까 생각하다가 결론을 내지 못했다. 밑의 코드 예시에 적어놓은 Collectors.toMap 메서드의 기능을 User 클래스에 똑같이 추가해볼까 하는 생각까지 해보았다. 하지만 그렇다면 Stream 을 쓰는 의미가 전혀 없을 뿐더러, 오히려 더 로직이 복잡해 질수 있겠다고 느꼈다..

```java
public static <T, K, U> Collector<T, ?, Map<K, U>> toMap(
    Function<? super T, ? extends K> keyMapper,
    Function<? super T, ? extends U> valueMapper,
    BinaryOperator<U> mergeFunction) {
    return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
}
```

<br/>

여기서 문득 이러한 **'부분이 Stream 의 단점이 아닐까??'** 하는 생각이 들었다. 물론 데이터를 선언적으로 처리하여 가독성이 높고, 까다로운 구현들을 몇개의 체인 메서드만으로 쉽고 한 번에 처리할 수 있다는 엄청난 장점들이 많은 것은 사실이다. 하지만 유지보수 측면에서 봤을 땐, 여러 기능들을 한 번에 처리하려는 습성이 있는 Stream 은 그리 좋은 방법이 아닐 수도 있겠다는 생각이 든다.



지금과 같은 상황만 봐도 Stream 으로 처리하려는 방향성을 포기하지 않는다면 지금 이 로직에서 getter 를 리팩토링 할 방법이 전혀 생각이 나지 않는다. 물론 내가 역량이 많이 낮아서 그런 것일 수도 있겠지만 말이다.



지금까지 객체지향 설계와 TDD 를 공부하면서 늘 느끼는 거지만, **개발에 정답은 없고 배움의 끝은 없다는 것을 뼈저리게 느끼게 된다.**