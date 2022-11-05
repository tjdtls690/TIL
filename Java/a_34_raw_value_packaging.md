# 목차

1. [문제 발생 - 모든 원시 값과 문자열을 포장하는 것의 이점을 정확히 인지하지 못한다.](#1-문제-발생---모든-원시-값과-문자열을-포장하는-것의-이점을-정확히-인지하지-못한다) <br/>
2. [문제 해결 - 이점 2가지](#2-문제-해결---이점-2가지) <br/>
    1. [단일 책임의 원칙 - 가독성과 유지보수성이 증가](#1-단일-책임의-원칙---가독성과-유지보수성이-증가) <br/>
    2. [테스트 코드의 유지보수성 및 효율성 증가](#2-테스트-코드의-유지보수성-및-효율성-증가) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 2주차 - 모든 원시값과 문자열을 포장하는 것의 이점을 인지하자

<br/>

> **이번 프리코스 2주차에서 모든 원시 값과 문자열을 포장하면서**
>
> **직접 고민하고 느끼면서 인지하게된 이점들에 대해 정리해 보고자 한다.**

<br/>

## 1. 문제 발생 - 모든 원시 값과 문자열을 포장하는 것의 이점을 정확히 인지하지 못한다.

사실 지금까지 객체지향 설계를 하면서 **모든 원시 값과 문자열을 포장**했었다. 하지만 그 이유가 **객체지향 생활 체조 원칙에 있으니까 무지성으로 하는 느낌**이 굉장히 강했다. 이렇게 포장을 했을 때의 이점을 정확하게 인지하지 못하고 있었다.



원시 값과 문자열을 포장해서 역할을 위임해주는 것뿐인데 대체 무슨 이점이 있는 것일까 항상 궁금했고 고민도 해봤지만, 경험이 부족해서인지 역량 자체가 부족해서인지 제대로 파악이 되질 않았다.



그러나 이번 프리코스 2주차를 설계하고 구현하면서 이 부분에 대한 이점을 확실하게 느낄 수 있었다. 총 2가지의 이점에 대해 정리를 해보려고 한다. 



우테코 프리코스의 문제 유출을 피하기 위해, 실제 프리코스에서 구현한 코드가 아닌 이 글을 작성하면서 즉흥적으로 구현한 예시를 사용할 것이다.

<br/>

## 2. 문제 해결 - 이점 2가지

### 1. 단일 책임의 원칙 - 가독성과 유지보수성이 증가

보통 포장은 원시 타입과 문자열 인스턴스 변수가 2개 이상인 클래스에서 하게 된다. 포장하기 전의 예시를 간단히 구현해보자.

```java
public class Car {
    private final String name;
    private final int speed;
    
    public Car(final String name, final int speed) {
        this.name = name;
        this.speed = speed;
    }
}
```

<br/>

만약 이 상황에서 speed 와 name 두 변수 모두 유효성 검증을 해줘야 하는 상황이라고 해보자.

```java
public class Car {
    private static final int MIN_SPEED = 0;
    private static final int MAX_SPEED = 350;
    private static final int MAX_LENGTH_OF_NAME = 3;
    private static final String CAR_NAME_FORM = "[a-zA-Z]+";
    
    
    private final String name;
    private int speed;
    
    public Car(final String name, final int speed) {
        isValid(name); // name 에 대한 유효성 검증
        isValid(speed); // speed 에 대한 유효성 검증
        this.name = name;
        this.speed = speed;
    }
    
    private void isValid(final String name) { // name 에 대한 유효성 검증
        validateNameLength(name);
        validateNameForm(name);
    }
    
    private void validateNameLength(final String name) {
        if (name.length() > MAX_LENGTH_OF_NAME) {
            throw new IllegalArgumentException("Car name must be less than 3 characters");
        }
    }
    
    private void validateNameForm(final String name) {
        final Matcher matcher = Pattern.compile(CAR_NAME_FORM).matcher(name);
        if (!matcher.matches()) {
            throw new IllegalArgumentException("Car names must be english");
        }
    }
    
    private void isValid(final int speed) { // speed 에 대한 유효성 검증
        validateOverSpeed(speed);
        validateNegative(speed);
    }
    
    private void validateOverSpeed(final int speed) {
        if (speed > MAX_SPEED) {
            throw new IllegalArgumentException("Speed must be less than " + MAX_SPEED);
        }
    }
    
    private void validateNegative(final int speed) {
        if (speed < MIN_SPEED) {
            throw new IllegalArgumentException("Speed must be positive");
        }
    }
}
```

<br/>

두 개의 상태에 대해 모두 유효성 검증을 해주는 모습이다. 두 개의 상태에 대한 책임을 모두 가지다 보니, 자연스럽게 코드는 길어지고 복잡해진다. 상수들도 두 변수에 관련된 것들을 모두 가지게 된다. 결국 가독성과 유지보수성 모두 떨어지는 사태가 발생하게 된다. 



유효성 검증만 해도 이런데, 두 개의 상태에 대한 로직(기능)들이 추가되기 시작한다면 진정한 **'헬 게이트'**가 열릴 것이다.



**하나의 객체에 이렇게 많은 책임을 부여하는 것은 바람직한 설계가 아니라고 생각한다.** 가독성 측면에서든, 유지보수 측면에서든 말이다. 이제 원시 타입과 문자열 변수 모두 포장을 해보자.

```java
public class Car {
    private final CarName name;
    private final CarSpeed carSpeed;
    
    public Car(final String name, final int speed) {
        this(new CarName(name), new CarSpeed(speed));
    }
    
    public Car(final CarName name, final CarSpeed carSpeed) {
        this.name = name;
        this.carSpeed = carSpeed;
    }
}
```

```java
public class CarSpeed {
    private static final int MIN_SPEED = 0;
    private static final int MAX_SPEED = 350;
    
    private final int speed;
    
    public CarSpeed(final int speed) {
        isValid(speed); // speed 에 대한 유효성 검증
        this.speed = speed;
    }
    
    private void isValid(final int speed) {
        validateOverSpeed(speed);
        validateNegative(speed);
    }
    
    private void validateOverSpeed(final int speed) {
        if (speed > MAX_SPEED) {
            throw new IllegalArgumentException("Speed must be less than " + MAX_SPEED);
        }
    }
    
    private void validateNegative(final int speed) {
        if (speed < MIN_SPEED) {
            throw new IllegalArgumentException("Speed must be positive");
        }
    }
}
```

```java
public class CarName {
    private static final int MAX_LENGTH_OF_NAME = 3;
    private static final String CAR_NAME_FORM = "[a-zA-Z]+";
    
    private final String name;
    
    public CarName(final String name) {
        isValid(name); // name 에 대한 유효성 검증
        this.name = name;
    }
    
    private void isValid(final String name) {
        validateNameLength(name);
        validateNameForm(name);
    }
    
    private void validateNameLength(final String name) {
        if (name.length() > MAX_LENGTH_OF_NAME) {
            throw new IllegalArgumentException("Car name must be less than 3 characters");
        }
    }
    
    private void validateNameForm(final String name) {
        final Matcher matcher = Pattern.compile(CAR_NAME_FORM).matcher(name);
        if (!matcher.matches()) {
            throw new IllegalArgumentException("Car names must be english");
        }
    }
}
```

<br/>

하나의 객체가 하나의 상태에 대한 책임만 가지게 되면서 응집도가 높아진다. 그로인해 가독성과 유지보수성이 향상되는 모습을 볼 수 있다.

<br/>

## 2. 테스트 코드의 유지보수성 및 효율성 증가

먼저 포장하기 전의 프로덕션과 테스트예시를 다시 구현해보자.

```java
public class Car {
    private static final int MIN_SPEED = 0;
    private static final int MAX_SPEED = 350;
    private static final int MAX_LENGTH_OF_NAME = 3;
    private static final String CAR_NAME_FORM = "[a-zA-Z]+";
    
    
    private final String name;
    private int speed;
    
    public Car(final String name, final int speed) {
        isValid(name); // name 에 대한 유효성 검증
        isValid(speed); // speed 에 대한 유효성 검증
        this.name = name;
        this.speed = speed;
    }
    
    private void isValid(final String name) { // name 에 대한 유효성 검증
        validateNameLength(name);
        validateNameForm(name);
    }
    
    private void validateNameLength(final String name) {
        if (name.length() > MAX_LENGTH_OF_NAME) {
            throw new IllegalArgumentException("Car name must be less than 3 characters");
        }
    }
    
    private void validateNameForm(final String name) {
        final Matcher matcher = Pattern.compile(CAR_NAME_FORM).matcher(name);
        if (!matcher.matches()) {
            throw new IllegalArgumentException("Car names must be english");
        }
    }
    
    private void isValid(final int speed) { // speed 에 대한 유효성 검증
        validateOverSpeed(speed);
        validateNegative(speed);
    }
    
    private void validateOverSpeed(final int speed) {
        if (speed > MAX_SPEED) {
            throw new IllegalArgumentException("Speed must be less than " + MAX_SPEED);
        }
    }
    
    private void validateNegative(final int speed) {
        if (speed < MIN_SPEED) {
            throw new IllegalArgumentException("Speed must be positive");
        }
    }
}
```

```java
class CarTest {
    @Test
    @DisplayName("예외 처리 : 차 이름 길이 초과")
    void name_length_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new Car("abcd", 0)) // name 관련 테스트인데도 speed까지 설정해줘야함
                .withMessage("Car name must be less than 3 characters");
    }
    
    @Test
    @DisplayName("예외 처리 : 영어가 아닌 경우")
    void not_english_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new Car("가나다", 0))
                .withMessage("Car names must be english");
    }
    
    @Test
    @DisplayName("예외 처리 : 최대 스피드 초과한 경우")
    void over_speed_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new Car("abc", 450)) // speed 관련 테스트인데도 name까지 설정해줘야함
                .withMessage("Speed must be less than 350");
    }
    
    @Test
    @DisplayName("예외 처리 : 스피드가 음수인 경우")
    void negative_speed_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new Car("abc", -1))
                .withMessage("Speed must be positive");
    }
}
```

<br/>

위의 테스트 코드 예시를 보면, CarTest 에 두 변수와 관련된 모든 테스트 코드가 작성된다. 그로인해 테스트 코드마저 너무 비대해지고 복잡해지게 된다. 



더 중요한 것은, **name만 테스트 할 때도 speed 값 설정이 필요하고, speed만 테스트 할 때도  name 값 설정이 필요**하다. 이는 굉장히 비효율적인 테스트 코드로 보인다. 그리고 유효성 검증 뿐만이 아닌, 추가되는 기능들에 대한 테스트도 해야한다면 더 힘들어질 것이다. 



포장을 해서 책임을 각 객체에 위임한 뒤의 테스트를 구현해보자.

```java
public class Car {
    private final CarName name;
    private final CarSpeed carSpeed;
    
    public Car(final String name, final int speed) {
        this(new CarName(name), new CarSpeed(speed));
    }
    
    public Car(final CarName name, final CarSpeed carSpeed) {
        this.name = name;
        this.carSpeed = carSpeed;
    }
}
```

```java
public class CarSpeed {
    private static final int MIN_SPEED = 0;
    private static final int MAX_SPEED = 350;
    
    private final int speed;
    
    public CarSpeed(final int speed) {
        isValid(speed); // speed 에 대한 유효성 검증
        this.speed = speed;
    }
    
    private void isValid(final int speed) {
        validateOverSpeed(speed);
        validateNegative(speed);
    }
    
    private void validateOverSpeed(final int speed) {
        if (speed > MAX_SPEED) {
            throw new IllegalArgumentException("Speed must be less than " + MAX_SPEED);
        }
    }
    
    private void validateNegative(final int speed) {
        if (speed < MIN_SPEED) {
            throw new IllegalArgumentException("Speed must be positive");
        }
    }
}
```

```java
public class CarName {
    private static final int MAX_LENGTH_OF_NAME = 3;
    private static final String CAR_NAME_FORM = "[a-zA-Z]+";
    
    private final String name;
    
    public CarName(final String name) {
        isValid(name); // name 에 대한 유효성 검증
        this.name = name;
    }
    
    private void isValid(final String name) {
        validateNameLength(name);
        validateNameForm(name);
    }
    
    private void validateNameLength(final String name) {
        if (name.length() > MAX_LENGTH_OF_NAME) {
            throw new IllegalArgumentException("Car name must be less than 3 characters");
        }
    }
    
    private void validateNameForm(final String name) {
        final Matcher matcher = Pattern.compile(CAR_NAME_FORM).matcher(name);
        if (!matcher.matches()) {
            throw new IllegalArgumentException("Car names must be english");
        }
    }
}
```

```java
class CarNameTest {
    @Test
    @DisplayName("예외 처리 : 차 이름 길이 초과")
    void name_length_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new CarName("abcd")) // name 관련 테스트는 name 설정만
                .withMessage("Car name must be less than 3 characters");
    }
    
    @Test
    @DisplayName("예외 처리 : 영어가 아닌 경우")
    void not_english_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new CarName("가나다"))
                .withMessage("Car names must be english");
    }
}
```

```java
class CarSpeedTest {
    @Test
    @DisplayName("예외 처리 : 최대 스피드 초과한 경우")
    void over_speed_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new CarSpeed(450)) // speed 관련 테스트는 speed 설정만
                .withMessage("Speed must be less than 350");
    }
    
    @Test
    @DisplayName("예외 처리 : 스피드가 음수인 경우")
    void negative_speed_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new CarSpeed(-1))
                .withMessage("Speed must be positive");
    }
}
```

<br/>

CarTest 가 완전히 삭제되고, 각 객체가 자신이 맡은 상태와 관련된 유효성 검증 테스트를 하는 모습을 볼 수 있다. 테스트 코드의 가독성과 유지보수성이 높아진다. 그로인해 테스트 코드의 높은 품질을 유지할 수 있게 된다.



또한 더 중요한 것은, **name관련 테스트를 할 때 쓸데없이 speed까지 설정해줘야 하는 비효율적인 테스트를 피할 수 있다는 것**이다. 이 효율성은 클래스와 변수가 더 많아질 수록 훨씬 더 큰 힘을 발휘하게 될 것이다.



물론 이렇게 내가 직접적으로 느낀 2가지 뿐만 아니라 이보다 더 많은 장점들이 있을 것이다. 인터넷에서 원시 값을 포장하는 것에 대한 이점을 검색해보면, '자료형에 구애받지 않을 수 있다', '비즈니스 로직의 중복을 피할 수 있다' 등등 더 많은 이점들을 볼 수 있다. 이 부분들도 같이 학습해보면 많은 도움이 될 것이다.