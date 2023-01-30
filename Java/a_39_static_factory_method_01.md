# 목차

1. [정적 팩토리 메서드란??](#1-정적-팩토리-메서드란) <br/>
2. [정적 팩토리 메서드의 장점](#2-정적-팩토리-메서드의-장점) <br/>
3. [정적 팩토리 메서드의 주의사항](#3-정적-팩토리-메서드의-주의사항) <br/>
4. [정적 팩토리 메서드와 팩토리 메서드 패턴의 연관성 (내 개인적인 생각)](#4-정적-팩토리-메서드와-팩토리-메서드-패턴의-연관성-내-개인적인-생각) <br/>
5. [결론 : 생성자보다 정적 팩토리 메서드를 사용해야하는 경우](#5-결론--생성자보다-정적-팩토리-메서드를-사용해야하는-경우) <br/>

<br/>

# [자바, Java] 이펙티브 자바(Effective Java) - 아이템 01 정적 팩토리 메서드

<br/>

> **드디어 기다리던 이펙티브 자바를 공부하기 시작한다.**
>
> **좀 어렵진 않을까 두려우면서도 성장하는 내 모습을 기대하며 설레는 마음도 같이 생긴다.**
>
> **스프링부터 공부할까 생각했지만, 자바가 먼저라는 마인드로 이펙티브 자바를 먼저 공부하게 되었다.**
>
> **오늘은 아이템 1인 정적 팩토리 메서드에 관해 공부한 내용을 간단히 정리해보려 한다.**

<br/>

## 1. 정적 팩토리 메서드란??

말 그대로 <code><strong>'클래스의 인스턴스를 반환하는 단순한 정적 메서드'</strong></code>를 말한다. 예제를 간단히 만들어보면,

```java
public class Car {
    private final CarName carName;
    private final CarPosition carPosition;
    
    // 정적 팩토리 메서드 사용을 유도하기 위해 생성자는 전부 private 처리를 했다.
    private Car(String carName, int carPosition) {
        this(new CarName(carName), new CarPosition(carPosition));
    }
    
    private Car(CarName carName, CarPosition carPosition) {
        this.carName = carName;
        this.carPosition = carPosition;
    }
    
    // 해당 메서드가 정적 팩토리 메서드이다.
    // 여러개의 인자를 받아 인스턴스를 반환하는 형태이기 때문에 of로 네이밍했다.
    public static Car of (String carName, int carPosition) {
        return new Car(carName, carPosition);
    }
    
    @Override
    public String toString() {
        return "Car{" +
                "carName=" + carName +
                ", carPosition=" + carPosition +
                '}';
    }
}
```

```java
public class CarName {
    private final String carName;
    
    public CarName(String carName) {
        this.carName = carName;
    }
    
    @Override
    public String toString() {
        return "CarName{" +
                "carName='" + carName + '\'' +
                '}';
    }
}
```

```java
public class CarPosition {
    private final int carPosition;
    
    public CarPosition(int carPosition) {
        this.carPosition = carPosition;
    }
    
    @Override
    public String toString() {
        return "CarPosition{" +
                "carPosition=" + carPosition +
                '}';
    }
}
```

```java
public class App01 {
    public static void main(String[] args) {
        // 생성자가 전부 private 이기에 에러가 뜬다.
//        Car abel01 = new Car("Abel", 3);
//        Car abel02 = new Car(new CarName("Abel"), new CarPosition(2));
        
        // 정적 팩토리 메서드를 통해 Car 인스턴스를 반환받는 모습이다.
        Car abelCar = Car.of("Abel", 3);
        System.out.println(abelCar);
    }
}

// 출력 결과
// Car{carName=CarName{carName='Abel'}, carPosition=CarPosition{carPosition=3}}
```

<br/>

## 2. 정적 팩토리 메서드의 장점

>  **생성자가 있는데 굳이 왜 정적 팩토리 메서드가 존재하는 것일까??**

그 이유를 5가지의 장점으로 살펴보고자 한다.

### 1) 이름을 가질 수 있다.

**이름을 가질 수 있다는 게 무슨 말일까?? 어떤 이름을 뜻하는 것일까??**

여기서 이름은 메서드의 이름을 말한다. 아시다시피 생성자는 원하는 이름을 마음대로 가질 수 없다. 이유는, 생성자는 클래스 이름과 동일해야 하기 때문이다. 그래서 해당 생성자의 역할이 무엇인지 정확하지 않을 수 있다. 예제를 간단히 만들어서 살펴보자.

```java
public class Car {
    private final CarName carName;
    private final CarPosition carPosition;
    private final CarSpeed carSpeed;
    
    // 생성자 선언부 에러
    public Car(String carName, int carPosition) {
        this(new CarName(carName), new CarPosition(carPosition), new CarSpeed(0));
    }
    
    // 생성자 선언부 에러
    public Car(String carName, int carSpeed) {
        this(new CarName(carName), new CarPosition(0), new CarSpeed(carSpeed));
    }
}
```

```java
public class CarPosition {
    private final int carPosition;
    
    public CarPosition(int carPosition) {
        this.carPosition = carPosition;
    }
}
```

```java
public class CarSpeed {
    private final int carSpeed;
    
    public CarSpeed(int carSpeed) {
        this.carSpeed = carSpeed;
    }
}
```

<br/>

위 예제를 보면 CarPosition과 CarSpeed 두개가 같은 int 타입인 것을 볼 수 있다. 여기서 **'차 이름과 포지션만 입력받아서 초기화하는 생성자'** 혹은 **'차 이름과 스피드만 입력받아서 초기화하는 생성자'** 두개를 전부 만들고 싶다면 어떻게 해야할까??

일단 생성자는 메서드와 마찬가지로 같은 시그니처로 만들 수 없다. 생성자는 이름을 바꿀 수도 없기 때문에 다른 방법을 고민해야한다. 그렇다면 딱 한 가지 방법이 떠오른다. **생성자 인자의 순서를 바꿔서 구현하는 것**이다.

```java
public class Car {
    private final CarName carName;
    private final CarPosition carPosition;
    private final CarSpeed carSpeed;
    
    // 차 이름과 포지션만 입력받아서 초기화하는 생성자
    private Car(String carName, int carPosition) {
        this(new CarName(carName), new CarPosition(carPosition), new CarSpeed(0));
    }
    
    // 차 이름과 스피드만 입력받아서 초기화하는 생성자
    private Car(int carSpeed, String carName) {
        this(new CarName(carName), new CarPosition(0), new CarSpeed(carSpeed));
    }
    
    private Car(CarName carName, CarPosition carPosition, CarSpeed carSpeed) {
        this.carName = carName;
        this.carPosition = carPosition;
        this.carSpeed = carSpeed;
    }
}
```

<br/>

문제는 이 방법이 과연 해결책이 될 수 있냐는 것이다. **'만약 같은 타입의 필드가 더 늘어난다면??', '만약 제 3자의 개발자가 인자의 순서가 다른 생성자를 보고 올바르게 생성자를 사용할 수 있을까??'** 등등 많은 문제점이 야기된다.

이러한 문제를 정적 팩토리 메서드의 이름 짓기를 통해 해결할 수 있다. 예제를 간단히 만들어보자.

```java
package selftest;

public class Car {
    private final CarName carName;
    private final CarPosition carPosition;
    private final CarSpeed carSpeed;
    
    private Car(String carName, int carPosition, int carSpeed) {
        this(new CarName(carName), new CarPosition(carPosition), new CarSpeed(carSpeed));
    }
    
    private Car(CarName carName, CarPosition carPosition, CarSpeed carSpeed) {
        this.carName = carName;
        this.carPosition = carPosition;
        this.carSpeed = carSpeed;
    }
    
    // 차 이름과 포지션만 입력받아서 초기화하는 정적 팩토리 메서드
    public static Car nameAndPosition (String carName, int carPosition) {
        return new Car(carName, carPosition, 0);
    }
    
    // 차 이름과 스피드만 입력받아서 초기화하는 정적 팩토리 메서드
    public static Car nameAndSpeed (String carName, int carSpeed) {
        return new Car(carName, 0, carSpeed);
    }
}
```

<br/>

어차피 예제이기 때문에 메서드의 이름은 막 지었다는 것을 생각하자...ㅋ

위 예제처럼 인자의 타입과 순서가 같다고 해도, **메서드의 이름을 통해 어떤 필드를 초기화하는지 명확히 나타낼 수 있다.** 그렇다면 클라이언트에서 어떤식으로 사용 가능할까???

```java
public class App01 {
    public static void main(String[] args) {
        Car nameAndSpeed = Car.nameAndSpeed("Abel01", 130);
        Car nameAndPosition = Car.nameAndPosition("Abel02", 50);
    
        System.out.println(nameAndSpeed);
        System.out.println(nameAndPosition);
    }
}

// 출력 결과
// Car{carName=CarName{carName='Abel01'}, carPosition=CarPosition{carPosition=0}, carSpeed=CarSpeed{carSpeed=130}}
// Car{carName=CarName{carName='Abel02'}, carPosition=CarPosition{carPosition=50}, carSpeed=CarSpeed{carSpeed=0}}
```

메서드의 이름으로 역할을 추정할 수 있기 때문에, 위 예제처럼 인자의 타입과 순서가 같다고 해도 클라이언트가 원하는대로 초기화된 인스턴스를 반환받을 수 있다.

<br/>

### 2) 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

UniqueTarget 이란 클래스를 만들어서 간단한 예시를 들어보겠다. 밑의 예시를 보면 생성자를 통해 객체를 만드는 경우, 초기화 값이 같더라도 항상 새롭게 객체를 생성하는 모습을 볼 수 있다. 4개 인스턴스의 주소가 전부 다르다.

```java
public class UniqueTarget {
    private final int number;
    
    public UniqueTarget(int number) {
        this.number = number;
    }
}
```

```java
public class App02 {
    public static void main(String[] args) {
        UniqueTarget uniqueTarget01 = new UniqueTarget(3);
        UniqueTarget uniqueTarget02 = new UniqueTarget(3);
        UniqueTarget uniqueTarget03 = new UniqueTarget(3);
        UniqueTarget uniqueTarget04 = new UniqueTarget(3);
    
        System.out.println(uniqueTarget01);
        System.out.println(uniqueTarget02);
        System.out.println(uniqueTarget03);
        System.out.println(uniqueTarget04);
    }
}

// 출력 결과
// selftest.lotto.UniqueTarget@1b2c6ec2
// selftest.lotto.UniqueTarget@4edde6e5
// selftest.lotto.UniqueTarget@70177ecd
// selftest.lotto.UniqueTarget@1e80bfe8
```

<br/>

여기서 **초기화 값이 사용된 적이 있는 값인 경우 원래 사용하던 인스턴스를 반환하게 하려고 한다.** 이 문제는 **생성자만 가지고는 해결이 안되는 부분**이다. **정적 팩토리 메서드를 이용**해서 구현해보자.

```java
public class UniqueTarget {
    // 캐싱 역할을 맡은 맵
    private static final Map<Integer, UniqueTarget> MAP = new HashMap<>();
    
    private final int number;
    
    // 해당 클래스 안에서만 사용하도록 하기 위해 생성자를 private으로 설정
    private UniqueTarget(int number) {
        this.number = number;
    }
    
    // 정적 팩토리 메서드
    // 매개변수의 값이 이미 들어온 적이 있는 값이면 이전에 만들었던 인스턴스를 가져온다.
    // 처음 들어온 값이면 새로 만들어서 반환한다.
    public static UniqueTarget instance(int number) {
        if (!MAP.containsKey(number)) {
            MAP.put(number, new UniqueTarget(number));
        }
        
        return MAP.get(number);
    }
}
```

```java
public class App02 {
    public static void main(String[] args) {
        UniqueTarget uniqueTarget01 = UniqueTarget.instance(3);
        UniqueTarget uniqueTarget02 = UniqueTarget.instance(4);
        UniqueTarget uniqueTarget03 = UniqueTarget.instance(3);
        UniqueTarget uniqueTarget04 = UniqueTarget.instance(4);
    
        System.out.printf("%s : %d\n", uniqueTarget01, uniqueTarget01.number());
        System.out.printf("%s : %d\n", uniqueTarget02, uniqueTarget02.number());
        System.out.printf("%s : %d\n", uniqueTarget03, uniqueTarget03.number());
        System.out.printf("%s : %d\n", uniqueTarget04, uniqueTarget04.number());
    }
}


// 출력 결과
// selftest.lotto.UniqueTarget@65b3120a : 3
// selftest.lotto.UniqueTarget@3f91beef : 4
// selftest.lotto.UniqueTarget@65b3120a : 3
// selftest.lotto.UniqueTarget@3f91beef : 4
```

위 예시에서 초기화 값이 같으면 같은 객체(같은 주소)를 반환하는 모습을 볼 수 있다.

<br/>

### 3) 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

**반환 타입을 부모 클래스 혹은 인터페이스로 설정 가능하다는 의미**이다. 보통은 인터페이스의 의미가 훨씬 더 크다. 점수에 따라 다른 등급을 반환하는 예시로 설명해보겠다. 먼저 생성자만을 통해 반환하는 예시를 간단히 만들어보자.

```java
public class Challenger {
    private final int score;
    
    public Challenger(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class Diamond {
    private final int score;
    
    public Diamond(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class Silver {
    private final int score;
    
    public Silver(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class App03 {
    public static void main(String[] args) {
        Challenger challenger = new Challenger(2400);
        Diamond diamond = new Diamond(1900);
        Silver silver = new Silver(1300);
    
        System.out.println(challenger);
        System.out.println(diamond);
        System.out.println(silver);
    }
}

// 출력 결과
// Challenger : 2400
// Diamond : 1900
// Silver : 1300
```

<br/>

가장 큰 문제점은 **'구현 클래스가 그대로 드러난다는 것'** 이다. 또한 각 구현 클래스를 전부 드러내야하기 때문에 API의 크기도 쓸데없이 커지기도 한다. 이를 방지하기 위해, 반환 타입이 인터페이스인 정적 팩토리 메서드를 이용해보자.

```java
public interface Rate {
    int score();
    
    static Rate createChallenger(int score) {
        return new Challenger(score);
    }
    
    static Rate createDiamond(int score) {
        return new Diamond(score);
    }
    
    static Rate createSilver(int score) {
        return new Silver(score);
    }
    
    default String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score());
    }
}
```

```java
public class AbstractRate implements Rate {
    private final int score;
    
    public AbstractRate(int score) {
        this.score = score;
    }
    
    @Override
    public int score() {
        return score;
    }
}
```

```java
// 각 구현체의 생성자는 default 로 설정
// 다른 패키지에서 사용 불가

public class Challenger extends AbstractRate {
    Challenger(int score) {
        super(score);
    }
}

public class Diamond extends AbstractRate {
    Diamond(int score) {
        super(score);
    }
}

public class Silver extends AbstractRate {
    Silver(int score) {
        super(score);
    }
}
```

```java
public class App03 {
    public static void main(String[] args) {
        Rate challenger = Rate.createChallenger(2400);
        Rate diamond = Rate.createDiamond(1900);
        Rate silver = Rate.createSilver(1300);
    
        System.out.println(challenger.description());
        System.out.println(diamond.description());
        System.out.println(silver.description());
    }
}

// 출력 결과
// Challenger : 2400
// Diamond : 1900
// Silver : 1300
```

<br/>

**구현 클래스가 드러나지 않기 때문에, '캡슐화'의 특성을 잘 살릴 수 있다.** 또한 여기서 중요한 점은 클라이언트쪽에선 구현 클래스들은 드러나지 않아도 되기 때문에, **API를 작게 유지할 수 있다.**

참고할 점 하나는, **'자바8'** 부터는 위 예시처럼 인터페이스에도 **'default 메서드와 static 메서드'** 를 사용할 수 있기 때문에, 인스턴스화 불가 동반 클래스를 굳이 만들지 않아도 된다는 것이다.

만약 자바8 이전 버전이었다면, Rates 라는 동반 클래스를 만들어서 private 생성자 설정으로 인스턴스화를 방지한 다음, 그 클래스 안에서 정적 팩토리 메서드를 구현했을 것이다. 이는 현재 자바 API 중 Collections 클래스가 대표적인 예이다.

물론 **인터페이스에 private 필드를 설정할 수 있는 기능은 아직 없기 때문에** 동반 클래스의 활용이 아예 사라진 것은 아니다.

<br/>

> **이 3번 장점은 4번과 5번 장점으로 연결된다.**

<br/>

### 4) 어떤 매개변수의 값이 오는지에 따라 다른 클래스를 반환할 수 있다.

**생성자는 인터페이스 타입으로 반환할 수 있는 기능이 없다.** 여기서 나타나는 차이점은 **'유연성'** 이다.

3번 예시를 다시 사용해보자. 먼저 생성자만을 통해 반환하는 예시를 보겠다.

```java
public class Challenger {
    private final int score;
    
    public Challenger(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class Diamond {
    private final int score;
    
    public Diamond(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class Silver {
    private final int score;
    
    public Silver(int score) {
        this.score = score;
    }
    
    public String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score);
    }
}
```

```java
public class App03 {
    public static void main(String[] args) {
        Challenger challenger = new Challenger(2400);
        Diamond diamond = new Diamond(1900);
        Silver silver = new Silver(1300);
    
        System.out.println(challenger);
        System.out.println(diamond);
        System.out.println(silver);
    }
}

// 출력 결과
// Challenger : 2400
// Diamond : 1900
// Silver : 1300
```

<br/>

생성자만 사용하는 경우엔 위 예시처럼 따로따로 값을 주면서 인스턴스 생성을 해주어야 한다. 여기서 벌써 문제점이 여러개가 보인다.

1. **등급의 개수가 지속적으로 늘어나는 경우**
   - 등급이 하나씩 늘어날 때마다 등급을 반환받는 클라이언트 코드가 계속 변경된다.
2. **등급의 제도가 전체적으로 개편되는 경우**
   - 이건 뭐... 말할 필요도 없다.
3. **중복 기능이 생긴다.**
   - 예제의 description() 메서드가 중복된다.
4. 등등...

<br/>

이러한 문제점들을 **'정적 팩토리 메서드의 반환 타입을 인터페이스로 설정'함으로써 대처**할 수 있다. 예시를 간단히 만들어보자.

```java
public interface Rate {
    int score();
    
    // 정적 팩토리 메서드
    static Rate from(int score) {
        if (score >= 2400) {
            return new Challenger(score);
        }
    
        if (score >= 1900) {
            return new Diamond(score);
        }
    
        return new Silver(score);
    }
    
    default String description() {
        return String.format("%s : %d", this.getClass().getSimpleName(), score());
    }
}
```

```java
public class AbstractRate implements Rate {
    private final int score;
    
    public AbstractRate(int score) {
        this.score = score;
    }
    
    @Override
    public int score() {
        return score;
    }
}
```

```java
public class Challenger extends AbstractRate {
    Challenger(int score) {
        super(score);
    }
}

public class Diamond extends AbstractRate {
    Diamond(int score) {
        super(score);
    }
}

public class Silver extends AbstractRate {
    Silver(int score) {
        super(score);
    }
}
```

```java
public class App03 {
    public static void main(String[] args) {
        Rate challenger = Rate.from(2400);
        Rate diamond = Rate.from(1900);
        Rate silver = Rate.from(1300);
    
        System.out.println(challenger.description());
        System.out.println(diamond.description());
        System.out.println(silver.description());
    }
}

// 출력 결과
// Challenger : 2400
// Diamond : 1900
// Silver : 1300
```

위 예시에서 클라이언트 코드(App03 클래스 코드)를 보면 알겠지만, <code><strong>Rate.from() 정적 팩토리 메서드 하나만으로 점수에 따라 다른 인스턴스를 반환</strong></code>받는 모습을 볼 수 있다. 이는 생성자만 쓸 때와 비교했을 때, 엄청난 유연성을 보여주는 것이다.

<br/>

### 5) 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이펙티브 자바 책에선 **'서비스 제공자 프레임워크'** 를 예시로 든다. 이 부분이 좀 어려울 수 있는데 예시를 간단히 만들어보자. **Challenger, Diamond, Silver 클래스가 전혀 존재하지 않는다는 가정**하에 만드는 것이다.

```java
public interface Rate {
    ... 생략
    
    static Rate from(int score) {
        // ServiceLoader.load() 메서드는 META-INF에 등록된 Rate 인터페이스를 구현한 구현체들을 가져온다.
        // 이 때 조건은, 다른 프로젝트에서 Rate 인터페이스를 구현한 구현체를 만들고,
        // 그 프로젝트에서 resources 디렉터리 안에 META-INF.services 디렉터리를 만들고,
        // services 디렉터리 안에 Rate 인터페이스의 풀네임을 이름으로 가진 파일을 만들고,
        // 그 파일 안에 Rate 인터페이스를 구현한 구현체의 풀네임을 등록해야만 
        // 해당 구현체를 서비스 제공자 프레임워크를 통해 가져온다.
        // 한 가지 더 추가할 조건은 그 프로젝트를 jar형태로 로컬에 배포한 상태여야 한다.
        
        // META-INF에 등록된 Rate 인터페이스를 구현한 구현체들을 가져온다.
        ServiceLoader<Rate> loader = ServiceLoader.load(Rate.class);
        
        // 가져온 구현체들 중 첫번째 구현체를 가져온다.
        // 가져올 구현체가 전혀 없기 때문에 Optional로 반환받게 된다.
        Optional<Rate> rateOptional = loader.findFirst();
        
        // 구현체가 있으면 그 구현체를 반환하고, 없으면 null을 반환한다.
        return rateOptional.orElse(null);
    }
    
    ... 생략
}
```

<br/>

서비스 제공자 프레임워크의 경우, 위 주석 설명처럼 조건이 꽤 까다롭다. 하지만 이 예시처럼 **구현체가 전혀 존재하지 않아도 인터페이스만으로 정적 팩토리 메서드를 작성할 수 있다는 점**을 보면 될 것 같다. 이 또한 **유연성 측면에서 굉장한 장점**이라 생각한다.

JDBC도 이러한 서비스 제공자 프레임워크의 대표적 예라고 한다. 추후에 좀 더 자세히 살펴봐야겠다.

아래 예시처럼도 생각할 수 있을 것 같다. 아주 간단한 예시지만 구현 클래스가 전혀 없이도 정적 팩토리 메서드를 작성한 모습이다.

```java
public interface Rate {
    ... 생략
        
    static List<Rate> createRates() {
        return new ArrayList<>();
    }
    
    ... 생략
}
```

<br/>

## 3. 정적 팩토리 메서드의 주의사항

굳이 정적 **팩토리 메서드의** **'단점'** 이 아닌 **'주의사항'** 이라고 작성한 이유는, 더 좋은 방향으로 대처할 수 있는 방법이 있기 때문이다.

<br/>

### 1) 정적 팩토리 메서드를 구현한 클래스는 상속이 불가능하다.

이유는 정적 팩토리 메서드를 사용하도록 유도해야하기 때문에 생성자를 private 으로 설정하게 된다. 그럼 그 클래스는 상속을 할 수 없게 된다. 해당 클래스의 생성자를 사용할 수 없으니, 부모 클래스의 생성자를 사용해야하는 자식 클래스 입장에서 보면 말이 안되기 때문이다.

그러나 이는 더 좋은 방향으로 유도될 수 있는 부분이기도 하다. 나중에 공부할 아이템 18인 **'상속보다 컴포지션을 이용하라'** 부분처럼 **컴포지션을 이용하도록 유도되기 때문**이다. 또한 상속을 못한다는 특성은 아이템 17인 **'불변 타입'을 지키도록 유도** 하기도한다.

컴포지션을 이용하여 간단히 예시를 작성해보자.

```java
public class Parent {
    private final int number;
    
    private Parent(int number) {
        this.number = number;
    }
    
    public static Parent of(int number) {
        return new Parent(number);
    }
    
    public String description() {
        return "Parent: " + number;
    }
}
```

```java
public class Child {
    private final Parent parent;
    
    public Child() {
        this.parent = Parent.of(5);
    }
    
    public String description() {
        return parent.description();
    }
}
```

```java
public class App03 {
    public static void main(String[] args) {
        Child child = new Child();
        System.out.println(child.description());
    }
}

// 출력 결과
// Parent: 5
```

<br/>

### 2) 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.

**생성자는 API 문서에서 따로 구분되어 명확히 드러나있다.** 하지만 **정적 팩토리 메서드**는 메서드이기 때문에 **다른 메서드들과 같이 분류되어서 찾기가 어렵다.**

그래서 필요한 것이 **'널리 알려진 정적 팩토리 메서드의 이름 규약'** 이다. 개발자들끼리 암묵적으로 정해놓은 일종의 규약이다.

1. **from** : 하나의 매개변수를 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드
2. **of** : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
3. **valueOf** : from과 Of의 더 자세한 버전
4. **instance** 혹은 **getInstance** : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장 하지 않음
5. **create** 혹은 **newInstance** : instance 혹은 getInstance 와 같으나 매번 새로운 인스턴스를 생성해 반환 함을 보장.
6. **getType** : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 펙토리 메서드를 정의할 때 쓴다. "Type"은 팩토리 메서드가 반환할 객체의 타입이다.
7. **newType** : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 펙토리 메서드를 정의할 때 쓴다. "Type"은 팩토리 메서드가 반환할 객체의 타입이다.
8. **type** : getType과 newType의 간결한 버전이다.

그래서 최종 대처 방법은, 자바독이 이러한 부분들까지도 알아서 처리해주는 그 날이 오기전까진, API문서를 잘 정리하면서 널리 알려진 규약을 따라 이름을 지어야 한다. 제대로 문서화(Javadoc)하는 방법은 추후에 공부해서 글로 정리해볼 예정이다.

<br/>

## 4. 정적 팩토리 메서드와 팩토리 메서드 패턴의 연관성 (내 개인적인 생각)

처음 공부할 때 이 두 가지가 굉장히 헷갈렸다. 이유는 **정적 팩토리 메서드의 장점들 중 4번 장점이 팩토리 메서드 패턴과 일치**한다고 생각하기 때문이다.

1. **팩토리 메서드 패턴** : 팩토리 안에서 조건에 따라 분기해서 객체를 생성하는 패턴
2. **정적 팩토리 메서드** : 객체를 생성하는 팩토리를 각각 구현한 메서드

하지만 자바 API중 하나인 Boolean.valueOf() 메서드를 살펴보자.

```java
@IntrinsicCandidate
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

이건 **정적 팩토리 메서드**이다. 하지만 조건에 따라 분기해서 객체를 반환하는 **팩토리 메서드 패턴**이기도 하다. 그래서 난 개인적으로 정적 팩토리 메서드의 특성들 중 하나가 팩토리 메서드 패턴이라고 생각한다. 즉, 팩토리 메서드 패턴이 정적 팩토리 메서드의 특성에 속한다고 생각한다.

이펙티브 자바의 9페이지에 보면 **'디자인 패턴 중 정적 팩토리 메서드와 일치하는 패턴은 없다'** 라고 적혀있다. 하지만 그 뒤에 이 문장을 덧붙이고 싶다.

> **하지만 정적 팩토리 메서드의 특성에 속하는 디자인 패턴은 존재한다.**

개인적으로 이렇게 이해해야 헷갈리지 않았다. 내 생각이 틀릴 수도 있지만, 지금은 이것이 가장 명확한 문장으로 보인다.

<br/>

## 5. 결론 : 생성자보다 정적 팩토리 메서드를 사용해야하는 경우

### 1) 한 클래스에 같은 시그니처의 생성자가 여러 개 필요한 경우

	- 각 팩토리 메서드의 차이를 잘 나타내는 이름을 지어주자.

### 2) 재활용 가능한 인스턴스인 경우

- 불필요한 객체 생성을 피한다.
- 인스턴스들의 생존 주기를 철저히 통제할 수 있다.
- 플라이웨이트 패턴과 열거 타입(Enum)이 같은 이치이다.

### 3) 인자 값에 따라 다른 값의 반환을 원하는 경우 (유연성)

<br/>

## Reference

1. [이펙티브 자바 완벽 공략 1부 - 백기선님](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1)
2. [정적 팩토리 메서드(Static Factory Method) - cjh8746.log](https://velog.io/@cjh8746/%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9CStatic-Factory-Method)
3. [이펙티브 자바: 아이템1. 생성자 대신 정적 팩토리 메서드를 고려하라 - Philosophia](https://sun-22.tistory.com/84)