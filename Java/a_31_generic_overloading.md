# 목차

1. [문제 발생 - 주 생성자와 부 생성자의 제네릭 타입](#1-문제-발생---주-생성자와-부-생성자의-제네릭-타입) <br/>
2. [문제 해결 - 외부에서부터 데이터 가공후 생성자 파라미터로 전달](#2-문제-해결---외부에서부터-데이터-가공후-생성자-파라미터로-전달) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 1주차 - 제네릭 타입과 오버로딩의 연관성

<br/>

> **드디어 떨리는 우테코 5기 프리코스가 시작되었다.**
>
> **이번 5기 테스트 과정은 1차 알고리즘 테스트가 빠지게 되었다.**
>
> **무조건 실력순으로 뽑는 것이 아닌, 신청한 모든 사람들에게 프리코스의 경험을 제공함으로써**
>
> **'소프트웨어의 생태계에 선한 영향력'을 끼치려는 자바지기님의 철학을 볼 수 있는 대목이라 생각한다.**
>
> **이번 1주차의 미션을 객체지향 설계로 구현하면서,** 
>
> **어떤 문제들이 생기고 그 문제들을 해결하는 과정이 있었는데,**
>
> **그 부분들에 대해서 하나씩 정리해 보고자 한다.**

<br/>

## 1. 문제 발생 - 주 생성자와 부 생성자의 제네릭 타입

엘레강트 오브젝트 책에서 생성자 파트에서 이런 내용이 나온다.

> **하나의 클래스는 2\~3개의 메서드와 5\~10개의 생성자를 포함하는 것이 적당하다. 응집도가 높고 견고한 클래스에는 적은 수의 메서드와 상대적으로 더 많은 수의 생성자가 존재한다.**
>
> **생성자가 많을수록 클래스는 더 개선되고, 사용자 입장에서 더 편하게 해당 클래스를 사용할 수 있다. 메서드가 많아지면 클래스의 초점이 흐려지고, 단일 책임 원칙을 위반한다. 생성자가 많아지면 유연성이 향상된다.**
>
> **생성자의 주된 작업은 제공된 인자를 사용해 캡슐화하고 필드를 초기화하는 일이다. 이러한 초기화 로직을 단 하나의 생성자에만 위치시키고, 주생성자라고 부르기를 권장하며, 부 생성자라고 부르는 다른 생성자들이 이 주 생성자를 호출하도록 만들자.**

추가로 내 개인적인 의견을 덧붙이자면, 해당 클래스가 가지고 있는 상태 즉, 인스턴스 변수 타입을 바로 전달 받는 생성자가 주 생성자, 그 외에 주 생성자를 this() 형식으로 호출함으로써 초기화 역할을 하는 나머지 생성자들을 부 생성자로 여긴다. 



이번에 객체지향 설계를 통해 프로그램을 구현하면서 생긴 문제를 예제로 간단히 보자.

```java
public class Users {
    private final List<User> users;
    
    // 부 생성자
    public Users(final List<List<String>> users) { // 에러
        this(initUsers(users));
    }
    
    // 주 생성자
    public Users(final List<User> users) {
        this.users = users;
    }
    
    private static List<User> initUsers(final List<List<String>> users) {
        return users.stream()
                .map(Users::initUser)
                .collect(Collectors.toList());
    }
    
    private static User initUser(final List<String> user) {
        return new User(user.get(1), user.get(0));
    }
}
```

<br/>

처음엔 왜 부 생성자에서 에러가 나는지 몰랐는데, 검색해보다가 답을 찾았다. **'제네릭 타입이 다른 것'은 오버로딩의 조건을 충족시켜주지 못하기 때문이다.** 그렇다면 왜 '제네릭 타입의 다름'은 오버로딩의 조건을 충족시켜주지 못하는 것일까??

<br/>

### Generic Type Erasure

자바 개발자들은 이러한 하위 호환성을 위해서 **Type Erasure 라는 개념**을 도입하였다. 자바 1.5 버전 이하에서는 제네릭이라는 개념이 없었다. 그래서 자바 컴파일러는 1.5 이전과 이후 버전의 상호 호환성을 위해서 제네릭을 지워버린다.

```java
public class Users {
    private final List<User> users;
    
    // 부 생성자
    public Users(final List users) { // 컴파일러에 의해 제네릭이 지워짐
        this(initUsers(users));
    }
    
    // 주 생성자
    public Users(final List users) { // 컴파일러에 의해 제네릭이 지워짐
        this.users = users;
    }
    
    private static List<User> initUsers(final List<List<String>> users) {
        return users.stream()
                .map(Users::initUser)
                .collect(Collectors.toList());
    }
    
    private static User initUser(final List<String> user) {
        return new User(user.get(1), user.get(0));
    }
}
```

<br/>

위 예시와 같이 컴파일이 될 때 제네릭을 지우고 컴파일 된다. 그래서 같은 List 라는 타입으로 취급받고 컴파일 에러가 나면서 오버로딩이 불가능해지는 것이다.

<br/>

## 2. 문제 해결 - 외부에서부터 데이터 가공후 생성자 파라미터로 전달

결국 맘에 안드는 방법이지만 외부에서부터 데이터를 가공해서 전달하는 식으로 해결을 했다.

```java
public class Users {
    private final List<User> users;
    
    public Users(final List<User> users) {
        this.users = users;
    }
}


// Users 클래스의 생성자를 호출하는 외부 클래스
public class Manager {
    private final Users users;
    private final DuplicateAccountEmails duplicateAccountEmails;
    
    public Manager(final List<List<String>> users) {
        this(initUsers(users));
    }
    
    public Manager(final Users users) {
        this.users = users;
        this.duplicateAccountEmails = new DuplicateAccountEmails();
    }
    
    private static Users initUsers(final List<List<String>> users) {
        return new Users(parseUsers(users)); // Users 클래스 생성자 호출하는 부분
    }
    
    private static List<User> parseUsers(final List<List<String>> users) { // 외부 클래스에서부터 데이터를 가공하는 모습
        return users.stream()
                .map(Manager::initUser)
                .collect(Collectors.toList());
    }
    
    private static User initUser(final List<String> user) {
        return new User(user.get(1), user.get(0));
    }
}
```

<br/>

사실 이보다 더 나은 방법을 찾아보고 싶었다. 괜히 외부 클래스의 로직이 많아지는 것 같은 느낌을 받아서 그렇다. 앞으로의 프리코스 기간동안 경험하게 될 많은 문제들에 대해, 더 심도 깊은 고민을 해봐야 할 것 같다.