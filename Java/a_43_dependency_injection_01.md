# 목차

1. [인스턴스화를 막는 경우는 어떤 경우인가??](#1-인스턴스화를-막는-경우는-어떤-경우인가) <br/>
2. [첫번째 방법 : 추상 클래스화 (비추천 방법)](#2-첫번째-방법--추상-클래스화-비추천-방법) <br/>
3. [두번째 방법 : 생성자를 private 으로 설정 (권장 방법)](#3-두번째-방법--생성자를-private-으로-설정-권장-방법) <br/>

<br/>

# [자바, Java] 이펙티브 자바(Effective Java) - 아이템 05 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

<br/>

## 1. 의존 객체 주입(Dependency Injection)란??

**Dependency Injection**의 줄임말로 **DI** 라고도 한다. **해당 객체가 의존하고 있는 객체를 외부에서 생성하여 생성자를 통해 주입하는 것을 말한다.** 다시 말해서, 클래스 안에서 직접 명시하여 객체를 생성하는 것이 아닌, 외부에서 생성자를 통해 객체를 주입받도록 구현하는 것이다.

그 형태를 간단하게 예제로 만들어보자.

```java
public interface Engine {
    void display();
}
```

```java
public class EightCylinderEngine implements Engine {
    @Override
    public void display() {
        System.out.println("가벼운 객체 : 8기통 엔진");
    }
}
```

```java
public class SixCylinderEngine implements Engine {
    @Override
    public void display() {
        System.out.println("무거운 객체 : 6기통 엔진");
    }
}
```

```java
public class Car {
    private final Engine engine;
    
    // 외부에서 객체를 주입
    public Car(Engine engine) {
        this.engine = engine;
    }
    
    public void displayEngine () {
        startEngineDemonstration();
        engine.display();
        endEngineDemonstration();
    }
    
    private void startEngineDemonstration() {
        System.out.println("엔진 시연회 준비");
    }
    
    private void endEngineDemonstration() {
        System.out.println("엔진 시연회 종료\n");
    }
}
```

```java
public class App {
    public static void main(String[] args) {
        Car car1 = new Car(new EightCylinderEngine());
        car1.displayEngine();
    
        Car car2 = new Car(new SixCylinderEngine());
        car2.displayEngine();
    }
}

// 출력 결과
// 엔진 시연회 준비
// 가벼운 객체 : 8기통 엔진
// 엔진 시연회 종료
// 
// 엔진 시연회 준비
// 무거운 객체 : 6기통 엔진
// 엔진 시연회 종료
```

<br/>

## 2. 언제 DI를 사용하는 것이 좋고, 이유는 무엇일까??

DI가 더 유용할 때가 있으니 사용할 것이다.

1. **사용하는 인스턴스에 따라 동작이 달라지는 클래스**
2. **사용하는 인스턴스 외에, 해당 클래스가 자체적으로 가지고 있는 코드도 같이 존재하는 클래스**

<br/>이제 이 2가지의 상황에서 DI를 썼을 때의 장점을 살펴보자. 해당 클래스 안에서 **'직접 객체를 명시하는 것'** 과 **'DI'** 를 비교하면서 장점을 소개할 것이다.

1. **테스트의 용이성**

   - 먼저 직접 객체를 명시하는 예제를 간단히 만들어보자.

     ```java
     public class DirectCar {
         private final SixCylinderEngine engine;
         
         public DirectCar() {
             this.engine = new SixCylinderEngine();
         }
         
         public void displayEngine() {
             startEngineDemonstration();
             engine.display();
             endEngineDemonstration();
         }
         
         private void startEngineDemonstration() {
             System.out.println("엔진 시연회 준비");
         }
         
         private void endEngineDemonstration() {
             System.out.println("엔진 시연회 종료\n");
         }
     }
     ```

     ```java
     class DirectCarTest {
         
         @Test
         void demoStartAndEnd() {
             DirectCar directCar = new DirectCar();
         
             assertThatNoException()
                     .isThrownBy(directCar::displayEngine);
         }
     }
     
     // 테스트 결과
     // 성공
     
     // 출력 결과
     // 엔진 시연회 준비
     // 무거운 객체 : 6기통 엔진
     // 엔진 시연회 종료
     ```

     <br/>이렇게 엔진 시연회 준비와 종료 과정만 테스트를 하고 싶을때도, 무거운 객체라고 가정한 6기통 엔진 객체가 계속 관여를 할 수밖에 없게 된다. 이는 비효율적인 테스트가 된다.

     이제 **DI** 를 이용한 예제를 간단히 만들어보자.

     ```java
     class DiCarTest {
         @Test
         void demoStartAndEnd() {
             // SixCylinderEngine 보다 가벼운 EightCylinderEngine 객체를 의도적으로 주입
             DiCar diCar = new DiCar(new EightCylinderEngine());
             assertThatNoException()
                     .isThrownBy(diCar::displayEngine);
         }
     }
     
     // 테스트 결과
     // 성공
     
     // 출력 결과
     // 엔진 시연회 준비
     // 가벼운 객체 : 8기통 엔진
     // 엔진 시연회 종료
     ```

     <br/>이처럼 **가벼운 객체를 선택적으로 넣어줌으로써** 준비와 종료 과정을 **효율적으로 테스트** 해줄 수 있다. 또는 아예 테스트 디렉토리 하위에 Mock 객체를 만들어서 테스트 할 때 주입할 수도 있다.<br/>

2. **유연성, 재사용성 (예제는 위의 예제들 참고)**

   - 직접 객체를 명시한 경우, **이미 클래스 내부에서 직접 명시된 인스턴스만 쓸 수밖에 없기 때문에, 상황마다 다른 인스턴스를 쓰고싶어도 쓸 수 없다.**
     - 다른 인스턴스를 넣어주기 위해 이름만 다른 클래스를 하나 더 만드는 너무 비효율적인 짓을 해야한다.
   - 하지만 **DI** 를 한 경우, **클래스는 그대로 두고 생성자에 의존 객체를 다른 객체로 주입만 해주면 된다.**
   - 또한, **DI** 를 한 경우, **생성자에 일반 클래스가 아닌 팩토리를 넘겨주는 방식으로도 응용이 가능하다.**
     - 그러면 복잡한 과정을 거쳐야만 만들 수 있는 인스턴스도 넘기기 용이해진다. 이 형식은 [디자인 패턴 - 팩토리 메서드 (Factory Method)](https://bit.ly/3cyR5oC) 에서 확인할 수 있다.
   - 그리고 **DI** 를 한 경우, **팩토리를 Supplier<> 타입으로 받게 구현해줄 수도 있다.** 생성자를 메서드 참조로 바로 넘겨주거나, 팩토리 메서드를 메서드 참조로 바로 넘겨주는 형식 말이다.
     - 이를 통해 객체를 필요할 때에만 생성을 하도록 하는 **'레이지'** 특성을 취할 수 있다. 또한 Supplier<> 인터페이스 함수의 제네릭 타입을 이용해서, **팩토리 타입 매개변수의 제한 및 개방을 의도대로 조율할 수 있다.**
   - 단, **클래스가 사용하는 인스턴스가 바뀔 일이 없으면서 그 인스턴스를 통해 사용되는 코드 외에 자체적으로 가지고있는 코드가 없다면 직접 명시해서 사용하는 것이 더 깔끔할 것이다.**<br/>

이러한 형태는 일반적인 형태로 더 확장 및 발전하면서, **스프링에서는 스프링 컨테이너가 Bean 끼리의 DI 를 알아서 해준다.**





## Reference

1. [이펙티브 자바 완벽 공략 1부 - 백기선님](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1)
2. [이펙티브 자바 3/E - 교보문고](https://product.kyobobook.co.kr/detail/S000001033066)
3. [의존관계 주입(Dependency Injection) 쉽게 이해하기 - 3기 완태님](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)