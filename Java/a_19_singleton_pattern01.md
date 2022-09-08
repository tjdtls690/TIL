# 목차

1. [싱글톤 패턴이란??](#1-싱글톤-패턴이란) <br/>
2. [싱글톤을 쓰는 이유](#2-싱글톤을-쓰는-이유) <br/>
3. [싱글톤이 안티패턴이 될 수 있는 이유](#3-싱글톤이-안티패턴이-될-수-있는-이유) <br/>
4. [싱글톤의 5가지 구현 방법](#4-싱글톤의-5가지-구현-방법) <br/>
5. [싱글톤을 깨뜨릴 수 있는 2가지 방법](#5-싱글톤을-깨뜨릴-수-있는-2가지-방법) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 싱글톤 패턴 (Singleton pattern)

<br/>

> **익숙한 패턴인 만큼, 디자인 패턴을 공부하기 시작할 때 처음 배운 패턴이다.**
>
> **국비학원에서도 스프링 과정에서 싱글톤 패턴을 간단하게 배우기 때문에, 어떤 패턴인지 대강 알고는 있었다.**
>
> **그러나 공부하고 나니 싱글톤에도 굉장히 여러 방법이 존재하고 각각의 장단점이 존재한다는 사실을 알게 됐다.**
>
> **모든 싱글톤 패턴과 그 중에서도 어떤 것들이 안전하고 보편적으로 사용되는 방법들인지 정리해보고자 한다.**

<br/>

## 1. 싱글톤 패턴이란??

- 오직 한 개의 클래스 인스턴스만을 갖도록 보장한다.

  - 전역적인 접근이 가능하다는 특징도 갖고 있지만, 이것은 단점이 될 수도 있기에 안티 패턴으로써 따로 정리해보고자 한다.

    <br/>

## 2. 싱글톤을 쓰는 이유

1. ### 해당 클래스의 인스턴스가 반드시 하나여야 함을 보장해야되는 상황

   - 보통 환경설정이 2개, 3개 이상이 아닌 1개만 있어야 하는 상황과 비슷한 경우이다.

     <br/>

2. ### 메모리 낭비 방지

   - new 연산자를 통해 생성을 하게 되면, 새로운 객체를 계속 생성하게 된다. 그럼 새로 객체를 생성한 만큼 메모리를 소비하게 된다.

     <br/>

3. ### 데이터 공유가 용이

   - 싱글톤 패턴을 구현할 때, static 메서드로 구현을 하기때문에, 전역 접근이 가능하다.

   - 그러나 이 부분은 안티 패턴의 이유가 될 수도 있는 부분이라 더 정리해보려고 한다.

     <br/>

## 3. 싱글톤이 안티패턴이 될 수 있는 이유

1. ### 전역 상태는 객체지향 프로그래밍에선 권장되지 않는 방법이다.

   - 아무 객체에서나 접근이 가능해지기 때문이다.

     <br/>

2. ### 리플렉션, 직렬화와 역직렬화 이 2가지 방법을 통해 싱글톤을 깨트릴 수 있다.

   - 물론 이러한 방법들도 방지할 수 있는 방법이 있긴 하지만 완벽하진 못하다.

     <br/>

3. ### 생성자의 접근제어자가 private 이기 때문에 상속이 불가능하기에 객체지향적이지 못하다.

   - 상속이 불가능한 또다른 이유도 있는데, static 메서드, static 필드를 사용해야 한다는 것이다.

     <br/>

4. ### 테스트하기가 힘들다

    - 객체 생성 방식 자체를 제한하고 있기 때문이다.

  <br/>

## 4. 싱글톤의 5가지 구현 방법

- ### 싱글톤이 아닌 예제

  - ```java
    public class NotSingleton {
    
    }
    
    public class Main {
        public static void main(String[] args) {
            NotSingleton notSingleton1 = new NotSingleton();
            NotSingleton notSingleton2 = new NotSingleton();
            System.out.println(notSingleton1 == notSingleton2);
        }
    }
    
    // 출력 결과
    // false
    ```

    - 아무런 조치를 취하지 않았고, **new 연산자를 통해 객체를 새로 생성**했기 때문에 당연히 객체가 다르다.
      - 그래서 출력 결과도 당연히 **false**.

    <br/>

- ### 첫번째 예제

  - ```java
    public class Singleton01 {
        private static Singleton01 instance;
        
        private Singleton01() { }
        
        public static Singleton01 getInstance() {
            if (instance == null) {
                instance = new Singleton01();
            }
            return instance;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Singleton01 singleton01_1 = Singleton01.getInstance();
            Singleton01 singleton01_2 = Singleton01.getInstance();
            System.out.println(singleton01_1 == singleton01_2);
        }
    }
    
    // 출력 결과
    // true
    ```

    - 클래스 메서드를 통해 인스턴스가 이미 생성되어있다면 이미 생성되어있는 것을 그대로 가져오게끔 구현했다.

    - 여기서 단점이 될 수 있는 부분은 **'멀티 쓰레드 환경에서 싱글톤이 깨질 수 있는'** 부분이다.

      - **두 유저가 시간이 거의 차이나지 않게 if 문 안으로 들어가버리면**, 인스턴스가 2개이상 생겨버리는 참사가 생길 수 있다.

      - 이걸 보완한 형태가 **두번째 예제**이다.

        <br/>

- ### 두번째 예제

  - ```java
    public class Singleton02 {
        private static Singleton02 instance;
        
        private Singleton02() { }
        
        public static synchronized Singleton02 getInstance() {
            if (instance == null) {
                instance = new Singleton02();
            }
            return instance;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Singleton02 singleton02_1 = Singleton02.getInstance();
            Singleton02 singleton02_2 = Singleton02.getInstance();
            System.out.println(singleton02_1 == singleton02_2);
        }
    }
    
    // 출력 결과
    // true
    ```

    - synchronized 동기화 처리에 의해 **한 명이 메서드 안으로 들어가면**, 그 한 명이 다시 나올 때까지 **다른 사람은 메서드에 들어가지 못한다**.

    - 그러나 이 역시 멀티 쓰레드 환경에서 **'병목 현상' 을 일으키는 단점**이 있다.

      - synchronized 로 메서드 전체를 잠궈버리면, 멀티 쓰레드의 **'동시에 여러 작업을 처리' 할 수 있는 장점이 사라진다.**
      - 한 명이 메서드를 사용하는 시간동안 다른 누구도 메서드를 들어가지 못하고 전부 기다리고 있어야 하니 성능저하는 당연하다.

    - 그래서 항상 **synchronized 통해 락을 거는 범위는 최소화** 해야한다.

    - 이러한 단점을 보완한 형태가 **세번째 예제**이다.

      <br/>

- ### 세번째 예제

  - ```java
    public class Singleton03 {
        private static volatile Singleton03 instance;
        
        private Singleton03() { }
        
        public static Singleton03 getInstance() {
            if (instance == null) {
                synchronized (Singleton03.class) {
                    if (instance == null) {
                        instance = new Singleton03();
                    }
                }
            }
            
            return instance;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Singleton03 singleton03_1 = Singleton03.getInstance();
            Singleton03 singleton03_2 = Singleton03.getInstance();
            System.out.println(singleton03_1 == singleton03_2);
        }
    }
    
    // 출력 결과
    // true
    ```

    - **double checked locking** 을 이용해서 더 효율적으로 동기화 처리를 한 형태이다.

    - 첫번재 if 문을 다수의 사람이 같이 뚫었다고 해도, 그 안의 동기화 블럭에 의해 걸러지게 된다.

      - 그리고 처음 입장한 쓰레드가 객체를 생성한 후엔, 처음 if 문을 같이 뚫었던 쓰레드들이 동기화 안쪽 if문에서 걸러지고,
      - 객체가 한 번 생성된 후에는 동기화 블럭을 거치지 않고도 바깥의 if문을 통해 바로바로 걸러지게 되어 <code><strong>성능의 저하를 최소화</strong></code> 할 수 있다.

    - 그러나 이것도 단점은 존재한다.

      1. **구현하기 위한 코드가 복잡한 편이다.**
      2. **변수에 volatile 키워드를 붙여줘야 한다.**
         - 이 부분 때문인지는 몰라도 **이 방법은 자바 1.5 이상부터만 정상적인 기능이 가능한 코드**이다.
         - volatile 키워드에 대해선 따로 공부하고 정리해볼 생각이다.

    - 만약 생성할 객체가 가볍고 미리 생성해놔도 상관 없다면 네번째 예제가 더 좋을 수 있다.

      <br/>

- ### 네번째 예제

  - ```java
    public class Singleton04 {
        private static final Singleton04 INSTANCE = new Singleton04();
        
        private Singleton04() { }
        
        public static Singleton04 getInstance() {
            return INSTANCE;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Singleton04 singleton04_1 = Singleton04.getInstance();
            Singleton04 singleton04_2 = Singleton04.getInstance();
            System.out.println(singleton04_1 == singleton04_2);
        }
    }
    
    // 출력 결과
    // true
    ```

    - **'이른 초기화'** 형태인 싱글톤이다.

      - 첫번째 ~ 세번째 예제까지는 모두 사용할 때 처음 객체를 생성하는 **'지연 초기화'** 의 형태였다.

    - **멀티 쓰레드 환경에서도 안전**하다.

    - 단점

      1. **객체가 무거운 경우 문제가 될 수 있다.**
         - 이른 초기화로 인해서 로딩할 때 바로 객체가 생성 된다.
         - 안그래도 오래걸리는 로딩 시간이 더욱 오래걸리게 만드는 결과를 가져오게 된다.

    - 자바 1.4 이하 버전을 쓰고있거나 '지연 초기화' 를 원하는 경우 세번째, 네번째 예제는 적절하지 못하다.

      - 그래서 가장 선호하는 싱글톤 형태 중 하나가 다섯번째 예제이다.

      <br/>

- ### 다섯번째 예제

  - ```java
    public class Singleton05 {
        private static class Singleton05HolderClass {
            private static final Singleton05 INSTANCE = new Singleton05();
        }
        
        public static Singleton05 getInstance() {
            return Singleton05HolderClass.INSTANCE;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Singleton05 singleton05_1 = Singleton05.getInstance();
            Singleton05 singleton05_2 = Singleton05.getInstance();
            System.out.println(singleton05_1 == singleton05_2);
        }
    }
    
    // 출력 결과
    // true
    ```

    - '지연 초기화' 를 지키면서 멀티 쓰레드 환경에서 안전한 싱글톤으로 적합한 형태이다.

      <br/>

## 5. 싱글톤을 깨뜨릴 수 있는 2가지 방법

- **다섯번째 예제를 쓴다고 해도 완벽하게 안전한 것이 아니다.**

- <code><strong>싱글톤을 깨뜨리려면 깨뜨릴 수 있다. (2가지 방법)</strong></code>

  1. **리플렉션을 이용해서.**
  2. **역직렬화와 직렬화를 이용해서.**

- 그리고 다섯번째 예제는 **역직렬화와 직렬화를 이용해서 싱글톤을 깨뜨리는 것은 방지**할 수 있다.

  - 역직렬화, 직렬화를 할 때 사용하는 메서드를 재정의 하는식으로 방지 가능하다.
  - 다만 **리플렉션을 이용한 방법은 방지하지 못한다.**

- 두 가지 방법을 모두다 방지하고 싶으면 **enum 열거형 클래스를 이용한 싱글톤 구현**으로 가능하다.

  - 하지만 이 또한 **상속을 하지 못한다는 점에서 단점**이 존재한다.

- 이러한 부분들의 더 상세한 내용들은 쓰지 않겠다.

  - 궁금하면 백기선님 강의로 ㅋㅋ...

    <br/>

## Reference

- [인프런 - 코딩으로 학습하는 GoF의 디자인 패턴 : 백기선님](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4)
- [GOF 의 디자인 패턴 - 에릭 감마(Erich Gamma), 리차드 헬름(Richard Helm), 랄프 존슨(Ralph Johnson), 존 블리시데스(John Vlissides)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791195444953)