# 목차

1. [싱글톤에 대한 정리 글 링크](#1-싱글톤에-대한-정리-글-링크) <br/>
2. [각 싱글톤 형태의 장단점 및 권장 형태](#2-각-싱글톤-형태의-장단점-및-권장-형태) <br/>
    1. [private 생성자와 public static final 필드 이용](#1-private-생성자와-public-static-final-필드-이용) <br/>
    2. [private 생성자와 정적 팩토리 메서드 이용](#2-private-생성자와-정적-팩토리-메서드-이용) <br/>
    3. [열거 타입(Enum)을 이용 (이펙티브 자바 권장 방법)](#3-열거-타입enum을-이용-이펙티브-자바-권장-방법) <br/>

<br/>

# [자바, Java] 이펙티브 자바(Effective Java) - 아이템 03 private 생성자나 열거 타입으로 싱글톤임을 보증하라

<br/>

## 1. 싱글톤에 대한 정리 글 링크

싱글톤 패턴에 대해 알고 싶다면, 밑의 링크를 참고하길 바란다.

**[[자바, Java] 디자인 패턴 - 싱글톤 패턴 (Singleton pattern)](https://bit.ly/3KS2lJf)**

<br/>

## 2. 각 싱글톤 형태의 장단점 및 권장 형태

사실 위의 싱글톤 패턴 정리 글과 겹치는 부분이 꽤 있다. 하지만 이펙티브 자바를 통해 더 깊고 자세하게 알게된 내용이 꽤 된다. 이제 각 형태와 그 장단점들을 살펴보자.

<br/>

### 1) private 생성자와 public static final 필드 이용

먼저 이에 대한 예제를 간단히 만들어보자.

```java
public class UniqueSuperCar {
    public static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
    
    private UniqueSuperCar() { }
    
    public void run() {
        System.out.println("run~~~");
    }
}
```

```java
public class App {
    public static void main(String[] args) {
        UniqueSuperCar uniqueSuperCar = UniqueSuperCar.UNIQUE_SUPER_CAR;
        uniqueSuperCar.run();
    }
}

// 출력 결과
// run~~~
```

<br/>

위 예제처럼 생성자를 private 으로 만들어서 새로운 객체 생성을 방지하고, public 상수를 통해서만 이미 만들어진 객체를 가져오도록 설계한 것이다. 이런 형태에 대한 장단점, 그리고 단점을 극복하는 방법들에 대해서도 살펴보자.

- **장점**
  1. **간결하다.**
     - 코드가 간결하기 때문에 가독성이 좋다.
  2. **싱글톤임을 API에 명확하게 드러낼 수 있다.**
     - 주석 처리만 잘 해놓는다면, Javadoc에서 public static final 필드가 싱글톤 인스턴스임을 명확히 나타낼 수 있다.<br/>

- **단점**

  1. **인터페이스를 구현하지 않은 싱글톤을 사용하는 클라이언트 코드는 테스트하기가 어려워진다. (싱글톤의 공통 단점)**

     - 이 단점은 어떤 싱글톤 형태이든, 싱글톤이라면 가지고 있는 **공통 단점**이다.

     - 테스트 할 클라이언트 메서드가 해당 싱글톤을 사용하는 중이라면, **테스트를 할 때마다 해당 싱글톤을 같이 사용**하게 된다. 테스트의 주 목적이 클라이언트 메서드 안에서 싱글톤 외의 코드들을 테스트하는 것일 때, **싱글톤 객체가 굉장히 무겁거나 오래 걸린다면, 클라이언트 메서드는 테스트 할 때마다 쓸데없이 그 영향을 받게 될 것이다.**

     - 그래도 이에 대한 **대책은 존재**한다. 싱글톤 클래스가 **인터페이스를 구현하는 것으로 구조를 바꾸고, 테스트 할 때만 대역을 해줄 가짜 객체를 만들어서 테스트 하는 것**이다. 

       - 먼저 싱글톤 클래스를 사용하는 클라이언트 메서드를 테스트하는 예제를 만들어보자.

       - ```java
         public class RacingGame {
             private final UniqueSuperCar uniqueSuperCar;
             
             public RacingGame(UniqueSuperCar uniqueSuperCar) {
                 this.uniqueSuperCar = uniqueSuperCar;
             }
             
             public void runGame() {
                 readyToRace();
                 uniqueSuperCar.run();
                 carMaintenance();
             }
             
             private void readyToRace() {
                 System.out.println("클라이언트 : ready~~");
             }
             
             private void carMaintenance() {
                 System.out.println("클라이언트 : under maintenance~~");
             }
         }
         ```

         ```java
         class RacingGameTest {
             
             @Test
             void runGame() {
                 RacingGame racingGame = new RacingGame(UniqueSuperCar.UNIQUE_SUPER_CAR);
             
                 assertThatNoException()
                         .isThrownBy(racingGame::runGame);
             }
         }
         
         // 테스트 결과
         // 성공
         
         // 출력 결과
         // 클라이언트 : ready~~
         // 싱글톤 : run~~~
         // 클라이언트 : under maintenance~~
         ```

         <br/>

         이 예제에선 runGame() 메서드의 클라이언트 코드만 테스트해보고 싶어도, 중간에 같이 껴있는 싱글톤 메서드도 같이 호출될 수밖에 없다. 저 싱글톤과 싱글톤 메서드가 굉장히 무거운 코드라 가정했을 때, 상대적으로 가벼운 클라이언트 코드만 테스트하고 싶어도 그러지 못한다.

         이제 인터페이스를 만들고 가짜 객체를 사용한 테스트 예제를 간단히 만들어보자.

         ```java
         public interface RacingCar {
             void run();
         }
         ```

         ```java
         // 만든 RacingCar 인터페이스를 구현
         public class UniqueSuperCar implements  RacingCar {
             public static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
             
             private UniqueSuperCar() { }
             
             @Override
             public void run() {
                 System.out.println("싱글톤 : run~~~");
             }
         }
         ```

         ```java
         public class RacingGame {
             // 클라이언트에선 인터페이스 타입으로 받기
             private final RacingCar racingCar;
             
             public RacingGame(RacingCar racingCar) {
                 this.racingCar = racingCar;
             }
             
             public void runGame() {
                 readyToRace();
                 racingCar.run();
                 carMaintenance();
             }
             
             private void readyToRace() {
                 System.out.println("클라이언트 : ready~~");
             }
             
             private void carMaintenance() {
                 System.out.println("클라이언트 : under maintenance~~");
             }
         }
         ```

         ```java
         // 테스트 디렉터리 아래에 인터페이스를 구현한 목 클래스 만들기
         public class MockRacingCar implements RacingCar {
             @Override
             public void run() {
                 System.out.println("가상 객체 : test run~~");
             }
         }
         ```

         ```java
         class RacingGameTest {
             
             @Test
             void runGame() {
                 // 실제 객체 대신 방금 만든 가짜 객체 주입
                 RacingGame racingGame = new RacingGame(new MockRacingCar());
             
                 assertThatNoException()
                         .isThrownBy(racingGame::runGame);
             }
         }
         
         // 테스트 결과
         // 성공
         
         // 출력 결과
         // 클라이언트 : ready~~
         // 가상 객체 : test run~~
         // 클라이언트 : under maintenance~~
         ```

         이런식으로 무거운 싱글톤 객체는 가벼운 가짜 객체로 바꿔주면서 테스트를 진행할 수 있다.<br/>

  2. **리플렉션 API를 사용해서 private 생성자를 얼마든지 이용할 수 있다. (1,2번 방법 공통 단점)**

     - 즉, private 생성자를 이용해서 인스턴스 객체를 새로 생성할 수 있는 것이다.

     - **지난번에 작성한 리플렉션 정리 글 링크 : [[자바, Java] 리플렉션 (Reflection) - 리플렉션의 개념 및 사용법](https://bit.ly/3kPN1UV)**

     - 리플렉션으로 private 생성자를 꺼내서 객체를 새로 생성하는 예제를 간단히 만들어보자.

       ```java
       public class App {
           public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
               Class<UniqueSuperCar> uniqueSuperCarClass = UniqueSuperCar.class;
               Constructor<UniqueSuperCar> constructor = uniqueSuperCarClass.getDeclaredConstructor();
               constructor.setAccessible(true);
           
               UniqueSuperCar uniqueSuperCar1 = constructor.newInstance();
               uniqueSuperCar1.run();
           
               UniqueSuperCar uniqueSuperCar2 = UniqueSuperCar.UNIQUE_SUPER_CAR;
               uniqueSuperCar2.run();
           
               System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
           }
       }
       
       // 출력 결과
       // 싱글톤 : run~~~
       // 싱글톤 : run~~~
       // 같은 객체인가?? : false
       ```

       <br/>

       당당하게 private 생성자를 이용해서 객체를 새로 생성하는 모습이다. 이에 대한 대처법도 존재하는데, 그 예제를 간단히 만들어보자.

       ```java
       public class UniqueSuperCar implements  RacingCar {
           public static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
           private static boolean createCheck;
           
           private UniqueSuperCar() {
               if (createCheck) {
                   throw new UnsupportedOperationException("더이상 새로운 객체를 생성할 수 없습니다.");
               }
               
               createCheck = true;
           }
           
           @Override
           public void run() {
               System.out.println("싱글톤 : run~~~");
           }
       }
       ```

       ```java
       public class App {
           public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
               Class<UniqueSuperCar> uniqueSuperCarClass = UniqueSuperCar.class;
               Constructor<UniqueSuperCar> constructor = uniqueSuperCarClass.getDeclaredConstructor();
               constructor.setAccessible(true);
           
               // 리플렉션을 통해 객체를 생성하려는 13번째 줄에서 예외가 발생한다.
               UniqueSuperCar uniqueSuperCar1 = constructor.newInstance(); // 예외 발생 줄
               uniqueSuperCar1.run();
           
               UniqueSuperCar uniqueSuperCar2 = UniqueSuperCar.UNIQUE_SUPER_CAR;
               uniqueSuperCar2.run();
           
               System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
           }
       }
       
       // 실행 결과
       // 13번째 줄에서 예외 발생
       ```

       <br/>이런 식으로 생성자가 2번 이상 호출되는 경우엔 예외를 던지는 식으로 방어할 수 있다. 하지만 이 대안을 사용하게 된다면, **public static final 필드를 사용하는 싱글톤의 장점인 '간결함' 은 사라질 수 있다.**<br/>

  3. **역직렬화 할 때 새로운 인스턴스가 생길 수 있다. (1,2번 방법 공통 단점)**

     - 먼저 직렬화와 역직렬화를 통해 새로운 객체를 생성하는 예제를 간단히 만들어보자.

       ```java
       public class App {
           public static void main(String[] args) {
               UniqueSuperCar uniqueSuperCar1 = UniqueSuperCar.UNIQUE_SUPER_CAR;
               
               // 직렬화 과정
               try (ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("test.txt"))) {
                   os.writeObject(uniqueSuperCar1);
                   uniqueSuperCar1.run();
               } catch (IOException e) {
                   throw new RuntimeException(e);
               }
               
               // 역직렬화 과정
               try (ObjectInputStream oi = new ObjectInputStream(new FileInputStream("test.txt"))) {
                   UniqueSuperCar uniqueSuperCar2 = (UniqueSuperCar) oi.readObject();
                   uniqueSuperCar2.run();
           
                   System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
               } catch (IOException | ClassNotFoundException e) {
                   throw new RuntimeException(e);
               }
           }
       }
       
       // 출력 결과
       // 싱글톤 : run~~~
       // 싱글톤 : run~~~
       // 같은 객체인가?? : false
       ```

       <br/>역시나 역직렬화 과정에서 생성자가 private임에도 불구하고 새로 객체를 생성하는 모습이다. 

       하지만 **이 또한 대안이 존재**한다. **역직렬화를 할 때 사용되는 readResolve() 메서드를 싱글톤 인스턴스를 반환하게끔 구현해서 싱글톤 클래스에 제공해주면 된다.** 간단히 예제를 만들어보자.

       ```java
       public class UniqueSuperCar implements  RacingCar, Serializable {
           public static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
           private static boolean createCheck;
           
           private UniqueSuperCar() {
               if (createCheck) {
                   throw new UnsupportedOperationException("더이상 새로운 객체를 생성할 수 없습니다.");
               }
               
               createCheck = true;
           }
           
           @Override
           public void run() {
               System.out.println("싱글톤 : run~~~");
           }
           
           // 역직렬화를 할 때 readResolve() 메서드 사용
           private Object readResolve() {
               return UNIQUE_SUPER_CAR;
           }
       }
       ```

       ```java
       public class App {
           public static void main(String[] args) {
               UniqueSuperCar uniqueSuperCar1 = UniqueSuperCar.UNIQUE_SUPER_CAR;
               
               try (ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("test.txt"))) {
                   os.writeObject(uniqueSuperCar1);
                   uniqueSuperCar1.run();
               } catch (IOException e) {
                   throw new RuntimeException(e);
               }
               
               try (ObjectInputStream oi = new ObjectInputStream(new FileInputStream("test.txt"))) {
                   UniqueSuperCar uniqueSuperCar2 = (UniqueSuperCar) oi.readObject();
                   uniqueSuperCar2.run();
           
                   System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
               } catch (IOException | ClassNotFoundException e) {
                   throw new RuntimeException(e);
               }
           }
       }
       
       // 출력 결과
       // 싱글톤 : run~~~
       // 싱글톤 : run~~~
       // 같은 객체인가?? : true
       ```

       <br/> 여기서 가장 신기했던 부분은 Object 가 반환타입인 readResolve() 는 **오버라이드 개념은 아니지만, 역직렬화가 될 때 사용**된다고 한다.

       궁금해서 개인적인 추측해봤는데, 그렇다면 역직렬화 API 안에서 리플렉션으로 readResolve() 메서드가 존재하는지 보고, 존재한다면 해당 메서드가 반환하는 인스턴스를 사용하는 형식이 아닐까?? 생각했었다. 그게 아니라면 오버라이드가 아니고서는 해당 메서드를 콕 집어서 사용할 방법이 없기 때문이다.

       그래서 직접 ObjectInputStream 객체 안을 뒤져보았다. 그리고 역시나 **리플렉션으로 readResolve() 메서드를 가져오는 모습**을 볼 수 있었다.

       ```java
       public class ObjectInputStream
           extends InputStream implements ObjectInput, ObjectStreamConstants {
           
           ... 생략
               
           // **** 1번 ****
           public final Object readObject() throws IOException, ClassNotFoundException {
               // **** 2번 ****
               return readObject(Object.class);
           }
           
           // **** 2번 ****
           private final Object readObject(Class<?> type) throws IOException, ClassNotFoundException {
               ... 생략
               
               try {
                   // **** 3번 ****
                   Object obj = readObject0(type, false);
                   
                   ... 생략
               } finally {
                   passHandle = outerHandle;
                   if (closed && depth == 0) {
                       clear();
                   }
               }
           }
           
           // **** 3번 ****
           private Object readObject0(Class<?> type, boolean unshared) throws IOException {
               
               ... 생략
               
               try {
                   switch (tc) {
                           
                       ... 생략
       
                       case TC_OBJECT:
                           if (type == String.class) {
                               throw new ClassCastException("Cannot cast an object to java.lang.String");
                           }
                           
                           // **** 4번 메서드 => readOrdinaryObject() ****
                           return checkResolve(readOrdinaryObject(unshared));
       
                       ... 생략
                   }
               } finally {
                   ... 생략
               }
           }
           
           // **** 4번 메서드 ****
           private Object readOrdinaryObject(boolean unshared) throws IOException {
               
               ... 생략
       
               if (obj != null &&
                   handles.lookupException(passHandle) == null &&
                   desc.hasReadResolveMethod()) {
                   
                   // **** 5번 메서드 ****
                   // 여기서 desc 변수는 ObjectStreamClass 클래스 타입이다.
                   // ObjectStreamClass 클래스의 invokeReadResolve(Object obj) 메서드로 들어가보면,
                   Object rep = desc.invokeReadResolve(obj);
                   
                   ... 생략
               }
       
               return obj;
           }
           
           ... 생략
       }
       ```

       ```java
       public class ObjectStreamClass implements Serializable {
           private Method readResolveMethod;
           
           ... 생략
           
           // **** 5번 메서드 ****
           Object invokeReadResolve(Object obj) throws IOException, UnsupportedOperationException {
               requireInitialized();
               if (readResolveMethod != null) {
                   try {
                       // 여기서 readResolveMethod 라는 변수의 타입을 보니
                       // 위에 필드를 보면 알겠지만 Method 타입이다.
                       // 즉 정확한 구조 파악은 안되었지만,
                       // 최소한 리플렉션을 이용해서 readResolve() 메서드가 있는지 확인하면
                       // readresolve() 메서드를 통해 반환받는 사실을 알 수 있다.
                       return readResolveMethod.invoke(obj, (Object[]) null);
                   } catch (InvocationTargetException ex) {
                       ... 생략
                   } catch (IllegalAccessException ex) {
                       ... 생략
                   }
               } else {
                   ... 생략
               }
           }
           
           ... 생략
       }
       ```

       <br/>즉, ObjectInputStream 객체의 readObject() 메서드는 리플렉션으로 해당 클래스 안에서 readResolve() 메서드를 찾아서 있으면 readResolve() 메서드가 반환한 인스턴스를, 없으면 새로운 인스턴스를 반환하는 것으로 보인다. API의 내부 구조를 완벽히 파악한 것이 아니라서, 내용이 틀린 부분이 있을 수 있으니, 만약 있다면 피드백 바란다.<br/>

- **결론**

  1. **인터페이스를 만든다.** 
     - 테스트하기 용이하도록 하기 위함.
  2. **생성자를 private으로 설정 후 생성자에 유효성 검증 코드를 넣는다.**
     - 리플렉션 방지하기 위함.
  3. **readResolve() 메서드를 싱글턴 객체를 반환하게끔 구현하는 것이다.**
     - 역직렬화로 인한 새로운 객체 생성 방지하기 위함.
  4. **단, 위 1,2,3번을 전부 조치한다고 가정하면, '간결함' 이라는 장점은 무조건 사라지게 될 것이다.**

<br/>

### 2) private 생성자와 정적 팩토리 메서드 이용

예제부터 간단히 만들어보자.

```java
public class UniqueSuperCar {
    private static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
    
    private UniqueSuperCar() {
    }
    
    // 정적 팩토리 메서드
    public static UniqueSuperCar getInstance() {
        return UNIQUE_SUPER_CAR;
    }
}
```

```java
public class App {
    public static void main(String[] args) {
        UniqueSuperCar uniqueSuperCar1 = UniqueSuperCar.getInstance();
        uniqueSuperCar1.run();
    
        UniqueSuperCar uniqueSuperCar2 = UniqueSuperCar.getInstance();
        uniqueSuperCar2.run();
    
        System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
    }
}

// 출력 결과
// 싱글톤 : run~~~
// 싱글톤 : run~~~
// 같은 객체인가?? : true
```

<br/>일단 단점은 위 1번 형태의 단점들과 완전 동일하다. 그렇다면 위 1번 형태에 비해 어떤 장점들이 있는지만 알아보자.<br/>

- **장점**

  1. **API(클라이언트 코드)를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.**

     - 팩토리 메서드를 이용해서 객체를 반환받는 클라이언트의 코드는 변경하지 않고, 팩토리 메서드의 구현부만 변경해서 동작을 바꿀 수 있는 것을 의미한다. 동작을 바꾼 예제를 간단히 만들어보자.

       ```java
       public class UniqueSuperCar {
           private static final UniqueSuperCar UNIQUE_SUPER_CAR = new UniqueSuperCar();
           
           private UniqueSuperCar() {
           }
           
           // 정적 팩토리 메서드가 새로운 객체를 반환하도록 변경
           public static UniqueSuperCar getInstance() {
               return new UniqueSuperCar();
           }
           
           public void run() {
               System.out.println("싱글톤 : run~~~");
           }
       }
       ```

       ```java
       // 해당 클라이언트 코드는 변경이 없는 상태에서 새로운 객체를 반환받게 된다.
       public class App {
           public static void main(String[] args) {
               UniqueSuperCar uniqueSuperCar1 = UniqueSuperCar.getInstance();
               uniqueSuperCar1.run();
           
               UniqueSuperCar uniqueSuperCar2 = UniqueSuperCar.getInstance();
               uniqueSuperCar2.run();
           
               System.out.println("같은 객체인가?? : " + (uniqueSuperCar1 == uniqueSuperCar2));
           }
       }
       
       // 출력 결과
       // 싱글톤 : run~~~
       // 싱글톤 : run~~~
       // 같은 객체인가?? : false
       ```

       <br/>

  2. **정적 팩토리를 제네릭 싱글턴 팩터리로 만들 수 있다.**

     - 클래스를 제네릭 클래스로, 그리고 정적 팩토리를 제네릭 정적 팩토리로 구현했을 때, 제네릭 정적 팩토리는 대입 받을 인스턴스 변수의 제네릭 타입이 어떤지에 따라 알아서 맞춰서 반환해준다. 좀 어려울 수 있으니 예제를 간단히 만들어보자.

       ```java
       // 제네릭 클래스
       public class UniqueSuperCar<T> {
           // 제네릭 타입을 모든 타입으로 강제 캐스팅이 가능하게끔 Object로 설정한다.
           private static final UniqueSuperCar<Object> UNIQUE_SUPER_CAR = new UniqueSuperCar<>();
           
           private UniqueSuperCar() {
           }
           
           // 제네릭 메서드
           @SuppressWarnings("unchecked")
           public static <E> UniqueSuperCar<E> getInstance() {
               // 여기서 강제 캐스팅을 해줘서 반환하는 형식으로 구현한다.
               return (UniqueSuperCar<E>) UNIQUE_SUPER_CAR;
           }
           
           public void print(T t) {
               System.out.println(t);
           }
           
           public void run() {
               System.out.println("싱글톤 : run~~~");
           }
       }
       ```

       ```java
       public class App {
           public static void main(String[] args) {
               // 오른쪽의 <String> 생략 가능
               UniqueSuperCar<String> uniqueSuperCar1 = UniqueSuperCar.<String>getInstance();
               uniqueSuperCar1.run();
               uniqueSuperCar1.print("aa");
           
               // 인스턴스를 반환받는 변수의 제네릭 타입을 보고 컴파일러는 어떤 제네릭 타입인지 유추할 수 있다.
               // 그래서 위의 getInstance() 메서드 옆에 있는 제네릭 표기를 생략할 수 있다.
               // 밑의 getInstance() 왼쪽 옆엔 <Integer> 가 생략되어 있는 것이다.
               UniqueSuperCar<Integer> uniqueSuperCar2 = UniqueSuperCar.getInstance();
               uniqueSuperCar2.run();
               uniqueSuperCar2.print(10);
           }
       }
       
       // 출력 결과
       // 싱글톤 : run~~~
       // aa
       // 싱글톤 : run~~~
       // 10
       ```

       <br/>위 예제처럼 굳이 getInstance() 메서드의 왼쪽에 제네릭 타입을 지정해주지 않아도, **인스턴스를 반환받을 변수의 제네릭 타입에 따라 제네릭 타입을 맞춰서 반환해준다.** 즉, 제네릭 정적 팩토리 메서드 하나만으로도 충분하다는 것이다. 그렇기 때문에 제네릭 정적 팩토리 메서드의 활용도는 괜찮다고 할 수 있다.<br/>

  3. **정적 팩토리의 메서드 참조를 Supplier 함수로 사용할 수 있다.**

     - 인자가 없고 반환값만 존재하는 함수형 인터페이스를 통해 사용 가능하다는 소리다. Supplier 함수로 사용하고 싶을 땐 장점이 될 수 있겠다. 간단하게 예제로 만들어보자.

       ```java
       public class App {
           public static void main(String[] args) {
               // 정적 팩토리 메서드는 이런식으로 Supplier 함수에 메서드 참조로 넣어서 이용할 수도 있다.
               UniqueSuperCar<String> uniqueSuperCar1 = printAll(UniqueSuperCar::getInstance);
               uniqueSuperCar1.print("aa");
           
               UniqueSuperCar<Integer> uniqueSuperCar2 = printAll(UniqueSuperCar::getInstance);
               uniqueSuperCar2.print(10);
           }
           
           public static <T> UniqueSuperCar<T> printAll(Supplier<UniqueSuperCar<T>> supplier) {
               UniqueSuperCar<T> uniqueSuperCar = supplier.get();
               supplier.get().run();
               return uniqueSuperCar;
           }
       }
       
       // 싱글톤 : run~~~
       // aa
       // 싱글톤 : run~~~
       // 10
       ```

  <br/>

- **하지만 이러한 장점들이 필요없는 상황이라면 오히려 final 필드를 이용한 1번째 방식이 더 좋다.**<br/>

### 3) 열거 타입(Enum)을 이용 (이펙티브 자바 권장 방법)

겉모습만 보면 뭔가 부족해보일 수 있지만, 난 개인적으로 이 방법이 가장 럭셔리한 싱글톤 방법이라고 생각한다. 이제 예제를 만들어 볼건데, 열거 상수 하나만 넣고 끝나도 되지만 심심하니까 좀 더 여러가지를 구현해 볼 것이다.

```java
public interface RacingCar<T> {
    default void run() {
        System.out.println("열거 타입 싱글톤 : run~~~");
    }
    
    void print(T t);
}
```

```java
public enum LuxurySuperCar implements RacingCar {
    LAMBORGHINI;
    
    private static int serialNumber = 0;
    
    @Override
    public void print(Object o) {
        System.out.printf("%d번째 실행 : %s%n", ++serialNumber, o);
    }
}
```

```java
public class App {
    public static void main(String[] args) {
        // 제네릭 타입 String 으로 설정
        RacingCar<String> uniqueSuperCar1 = LuxurySuperCar.LAMBORGHINI;
        uniqueSuperCar1.run();
        uniqueSuperCar1.print("aa");
        // uniqueSuperCar1.print(11) => 컴파일 에러 발생
        // 인터페이스의 제네릭 타입을 String 으로 지정해줬기 때문에 String 만 가능하다.
    
        // 제네릭 타입 Integer 으로 설정
        RacingCar<Integer> uniqueSuperCar2 = LuxurySuperCar.LAMBORGHINI;
        uniqueSuperCar2.run();
        uniqueSuperCar2.print(11);
        // uniqueSuperCar2.print("aa"); => 컴파일 에러 발생
        // 인터페이스의 제네릭 타입을 Integer 으로 지정해줬기 때문에 Integer 만 가능하다.
        
        System.out.println("같은 객체인가?? : " + (uniqueSuperCar1.equals(uniqueSuperCar2)));
    }
}

// 출력 결과
// 열거 타입 싱글톤 : run~~~
// 1번째 실행 : aa
// 열거 타입 싱글톤 : run~~~
// 2번째 실행 : 11
// 같은 객체인가?? : true
```

<br/>사실 열거 상수 LAMBORGHINI 만 LuxurySuperCar enum클래스 안에다 넣어놓고 끝내도 되긴 하는데, 뭔가 좀 심심해서 Enum 클래스를 제네릭 타입으로 받는 것으로 예제를 만들어봤다. 이건 내가 배우고있는 강의 내용에도 비슷한 형태조차 없는 형태다.

**Enum 클래스는 제네릭 타입이 허용되지 않기 때문에, 인터페이스를 구현해서 그 인터페이스에 제네릭 타입을 부여하는 형식으로 응용해서 구현했다.**

참고로 Enum 클래스에 있는 오버라이딩한 print() 메서드의 인자타입이 Object 라고 해서 모든 타입을 다 받을 수 있을거란 생각은 큰 오산이다. RacingCar 인터페이스의 print() 메서드는 제네릭 타입을 인자로 받고있기 때문에, **Enum 클래스에 있는 print() 메서드의 인자 타입이 Object 라고 해도, 인스턴스 변수 타입인 인터페이스의 제네릭 타입을 따라가게 된다.**

그리고 맨 마지막에 같은 객체인지 확인하는 코드에서 **equals()** 메서드를 쓴 이유는, 제네릭 타입이 다르면 **'=='** 로 비교하는 것을 컴파일 단계부터 쓰지 못하도록 막아버린다. 

근데 또 신기한건 equals() 메서드로 비교하면 또 같은 객체라고 나온다는 것이다. 인터페이스의 제네릭 타입이 다를 뿐, 결국 그 속은 같은 Enum 인스턴스이기 때문이다.

이제 enum 클래스로 싱글톤을 구현할 시의 장점을 살펴보자.<br/>

- **장점**

  1. **모든 싱글톤 형태 중 가장 간결한 형태다.**

     - 위의 예시는 내가 뻘짓하면서 좀 복잡하게 만들었지만, 사실 상수 하나만 넣어놓으면 끝난다..ㅋ

     - 이제 진짜 간단하게 예제를 만들어보자.

       ```java
       public enum LuxurySuperCar {
           LAMBORGHINI;
           
           public void run() {
               System.out.println("열거 타입 싱글톤 : run~~~");
           }
       }
       ```

       ```java
       package selftest.singleton;
       
       public class App {
           public static void main(String[] args) {
               LuxurySuperCar lamborghini1 = LuxurySuperCar.LAMBORGHINI;
               lamborghini1.run();
           
               LuxurySuperCar lamborghini2 = LuxurySuperCar.LAMBORGHINI;
               lamborghini2.run();
           
               System.out.println("같은 객체인가?? : " + (lamborghini1 == lamborghini2));
           }
       }
       ```

       진짜 저렇게 상수 하나 넣어주면 끝난다. 근데 모양은 저래도 엄청난 장점들을 가지고있는 녀석이다.<br/>

  2. **리플렉션에 안전하다.**

     - 아무런 조치를 취해주지 않아도 된다.
     - 지정해준 열거 상수들만 인스턴스로 사용할 수 있다는 enum 클래스의 특징을 알고 있다면, 리플렉션은 통하지 않는다는 것도 당연히 알 것이다.<br/>

  3. **역직렬화에 안전하다.**

     - 이것 또한 아무런 조치를 취해주지 않아도 된다.
     - 직렬화 후 역직렬화로 가져와도 새로운 객체가 아닌 직렬화 했던 객체가 그대로 다시 나오게 된다.
     - 결국 어떤 방법을 써도 지정해준 열거 상수들 외엔 새로운 객체는 절대 생성되지 않는다.<br/>

  4. **인터페이스 구현을 통해 원활한 테스트도 가능하다.**

     - 아까 맨 위에 제일 먼저 enum 싱글톤 예제로 작성한 형태가 바로 인터페이스를 구현한 형태다. Enum 클래스는 상속은 받을 수 없지만, 인터페이스 구현은 가능하다. 
     - 먼저 Enum 클래스가 상속받을 수 없는 이유를 상식적으로 따져보자면,
       - **Enum은 결국 열거된 객체들 하나하나가 해당 Enum 클래스를 상속받고 있는 형식이다.** 여기서 **'자바는 상속을 하나의 클래스만 받을 수 있다'** 는 사실을 알고 있다면, 그 이유를 어렵지 않게 추론할 수 있을 것이다. 하지만 자바에서 인터페이스는 여러개의 인터페이스를 구현하는 것이 가능하다.<br/>

## 결론 : 싱글톤은 열거 타입으로 구현하자<br/>

## Reference

1. [이펙티브 자바 1부 - 백기선님](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1#)
2. [자바 제네릭(Generics) 기초 (테코블) - 2기_둔덩](https://tecoble.techcourse.co.kr/post/2020-11-09-generics-basic/)
3. [The Basics of Java Generics](https://www.baeldung.com/java-generics)