# 목차

1. [리플렉션이란??](#1-리플렉션이란) <br/>
2. [리플렉션의 장단점](#2-리플렉션의-장단점) <br/>
3. [리플렉션을 통해 클래스 멤버에 접근하기](#3-리플렉션을-통해-클래스-멤버에-접근하기) <br/>
4. [리플렉션을 통해 여러가지 제약을 거는 테스트 코드 만들어보기](#4-리플렉션을-통해-여러가지-제약을-거는-테스트-코드-만들어보기) <br/>
5. [간단한 DI 만들어보기](#5-간단한-di-만들어보기) <br/>

<br/>

# [자바, Java] 리플렉션 (Reflection) - 리플렉션의 개념 및 사용법

<br/>

> **국비학원에서 리플렉션이란 말을 스치듯 들은 적이 있어서 리플렉션의 존재는 알고 있었다.**
>
> **하지만 어차피 잘 사용하지 않을 것 같고, 막연하게 너무 고급스럽고 깊고 어려운 개념으로 여기고 있었다.**
>
> **그런데 최근에 스프링의 어노테이션, AOP, DI 등등의 많은 기능들이 리플렉션을 통한다는 사실을 알게 됐다.**
>
> **갑자기 너무 궁금해졌다.**
>
> **어떻게 @Autowired 와 같은 어노테이션은 자동으로 인스턴스 객체를 주입할 수 있는 것인가??**
>
> **리플렉션의 개념이 휘발성으로 사라질지 모를 어려운 지식들이다.**
>
> **그래도 어떤 원리로 되는지 아는 거랑 모르는 것에는 많은 차이가 있을 것으로 믿고 있다.**

<br/>

## 1. 리플렉션이란??

리플렉션은 힙 영역에 로드 된 Class 타입의 객체를 통해, 접근 제어자 상관없이 원하는 클래스의 정보에 접근해서 조작할 수 있도록 지원하는 API이다.

- 조작할 수 있는 기능들을 나열해보면,

  - **필드 (목록) 가져오기**

  - **메소드 (목록) 가져오기**

  - **상위 클래스 가져오기**
  - **인터페이스 (목록) 가져오기** 
  - **애노테이션 가져오기**
  - **생성자 가져오기**
  - **생성자를 통해 인스턴스 객체 생성하기**
  - 등등...

<br/>

## 2. 리플렉션의 장단점

- **장점**

  1. **유연성과 확장성**

     - 구체적인 클래스를 알지 못해도 동적으로 클래스를 만들어서 의존 관계를 맺어줄 수 있다.
     - 개발 규모가 큰 스프링인 경우, 리플렉션을 이용한 Dynamic proxy를 통해서 @AutoWired, @Service, @Controller, @Repository 와 같은 DI 어노테이션을 활용한다.

  2. **접근 제한 상관없이 테스트 가능**

     - 밑에서 예제로 보겠지만, 접근 제어자가 private 이더라도 얼마든지 접근해서 조작할 수 있다.
     - 즉, private 메서드도 테스트가 가능하다는 소리이다.
       - 하지만 이 부분은 정말로 장점인 것인지 생각해볼 문제로 보인다.

     <br/>

- **단점**
  
  1. **캡슐화 저해**
     - 어찌보면 당연한 문제점이다.
     - private 한 데이터에도 접근이 가능한데, 캡슐화가 깨질 위험성이 존재하는 것은 당연하다.
  2. **성능 이슈**
     - 단순 접근보다 리플렉션을 통한 접근이 대부분 느리다.
     - 그래서 자주 호출되는 성능에 민감한 코드에는 웬만하면 적용하지 말아야 한다.
  3. **디버깅의 어려움**
     - private 멤버 테스트를 할 수 있기에 디버깅이 좋아질 수 있다고 생각할 수 있다.
     - 하지만 컴파일 단계가 아닌 런타임 단계에서 에러가 발생함으로써 생기는 디버깅의 어려움이 더 크게 느껴질 것이다.

<br/>

## 3. 리플렉션을 통해 클래스 멤버에 접근하기

먼저 리플렉션을 사용하려면, 모든 리플렉션의 시작인 **'Class\<T\>'** 타입을 가져와야 한다. 클래스 타입을 가져올 수 있는 방법은 총 3가지이다.

reflection 패키지 안에 있는 Car 클래스를 가져오는 예제를 살펴보자.

```java
// 가져올 클래스

package reflection;

public class Car {
    ... 생략
}
```

<br/>

1. **Class.forName(“FQCN”)**
   
   - FQCN (Fully Qualified Class Name)
     - **object, 함수, 변수의 계층적 구조를 명시적으로 모두 표현하는 것을 말한다**. 
     - Java의 경우 클래스가 포함된 패키지를 명시한다.
   
   - 예시
   
     - ```java
       package reflection;
       
       public class RefectionEx {
           public static void main(String[] args) throws ClassNotFoundException {
               Class<?> carClass = Class.forName("reflection.Car");
               System.out.println("FQCN : " + carClass.getName());
           }
       }
       
       // 출력 결과
       // FQCN : reflection.Car
       ```
   
     - Car 클래스를 제대로 가져와서 클래스의 풀 네임을 출력하는 모습을 볼 수 있다.
   
     <br/>
   
2. **클래스 타입.class**

   - 예시

     - ```java
       package reflection;
       
       public class RefectionEx {
           public static void main(String[] args) {
               Class<?> carClass = Car.class;
               System.out.println("FQCN : " + carClass.getName());
           }
       }
       
       // 출력 결과
       // FQCN : reflection.Car
       ```

     - 1번 방법보다 훨씬 간단히 가져오는 모습이다.

       <br/>

3. **인스턴스 변수.getClass()**

   - 예시

     - ```java
       package reflection;
       
       public class RefectionEx {
           public static void main(String[] args) {
               Car car = new Car();
               Class<?> carClass = car.getClass();
               System.out.println("FQCN : " + carClass.getName());
           }
       }
       
       // 출력 결과
       // FQCN : reflection.Car
       ```

     - 인스턴스 변수의 메서드인 getClass() 메서드를 이용해서 가져올 수 있다.

     - 참고 : getClass() 메서드는 최상위 클래스인 Object 클래스의 메서드이다.


이제 가져온 Class\<T\> 타입을 통해 필드와 메소드, 생성자를 어떻게 가져오는지, 가져온 후 어떤 조작을 할 수 있는지 몇 개만 간단히 정리해보자.

<br/>

### 1) 필드

1. **원하는 필드를 가져오는 법**
   
   1. **public 필드 가져오기**
   
      - ```java
        public class Car {
            public final CarName carName;
            private final CarPosition carPosition;
            
            ... 생략
        }
        ```
   
        - 현재 Car 클래스엔 public, private 필드가 하나씩 존재한다.
        
      - ```java
        public class RefectionEx {
            public static void main(String[] args) {
                Class<Car> carClass = Car.class;
                Arrays.stream(carClass.getFields())
                        .map(Field::getName)
                        .map(fieldName -> "field name : " + fieldName)
                        .forEach(System.out::println);
            }
        }
        
        // 출력 결과
        // field name : carName
        ```
   
        - 보시다시피 getFields() 메서드는 public 필드만 가져온다.


        <br/>

   2. **private 필드까지 전부 가져오기**

      - ```java
        public class RefectionEx {
            public static void main(String[] args) {
                Class<Car> carClass = Car.class;
                Arrays.stream(carClass.getDeclaredFields())
                        .map(Field::getName)
                        .map(fieldName -> "field name : " + fieldName)
                        .forEach(System.out::println);
            }
        }
        
        // 출력 결과
        // field name : carName
        // field name : carPosition
        ```

        - getDeclaredFields() 메서드를 사용하면 private 필드까지 가져올 수 있다.
        
        
        <br/>
        

   3. **원하는 필드 하나만 가져오기**

     - ```java
       public class RefectionEx {
           public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
               Car car = new Car();
               Class<? extends Car> carClass = car.getClass();
               Field carClassField = carClass.getDeclaredField("carPosition");
           
               System.out.println("field name : " + carClassField.getName());
           }
       }


​       
       // 출력 결과
       // 
       ```
    
       - getDeclaredField("필드 이름") 을 통해 원하는 필드 하나만 가져올 수 있다.
    
       <br/>

2. **해당 필드의 값을 가져오기**

   - 먼저 Car의 필드 초기화 값을 보자면,

     - ```java
       public class Car {
           public final CarName carName;
           private final CarPosition carPosition;
           
           public Car() {
               this("car", 100); // Car의 초기화 값
           }
           
           public Car(String carName, int carPosition) {
               this(new CarName(carName), new CarPosition(carPosition));
           }
           
           public Car(CarName carName, CarPosition carPosition) {
               this.carName = carName;
               this.carPosition = carPosition;
           }
           
           ... 생략
       }
       ```

       - carName 는 'car', carPosition 은 '100' 이다.

       <br/>

   - 이제 필드의 값들을 가져와보자.

     - ```java
       public class RefectionEx {
           public static void main(String[] args) {
               Car car = new Car();
               Class<? extends Car> carClass = car.getClass();
               
               Arrays.stream(carClass.getDeclaredFields())
                       .forEach(field -> {
                           // 접근 제어자를 분별하기 위해 modifiers를 가져온다.
                           int modifiers = field.getModifiers();
                           
                           // 1. private 인지 확인 후, 맞으면 setAccessible 를 true로 설정해준다.
                           if (Modifier.isPrivate(modifiers)) {
                               field.setAccessible(true);
                           }
                           
                           // Field 클래스의 get() 메서드를 통해 해당 필드의 값을 가져온다.
                           // 이 때, get() 메서드의 파라미터에는 어떤 인스턴스 객체의 필드 값을 가져올지를 정해준다.
                           // 지금은 car에 대입된 인스턴스 객체의 필드 값을 가져오겠다는 의미이다.
                           try {
                               System.out.println(field.get(car));
                           } catch (IllegalAccessException e) {
                               throw new RuntimeException(e);
                           }
                       });
           }
       }
       
       // 출력 결과
       // CarName{name='car'}
       // CarPosition{carPosition=100}
       ```

       <br/>

3. **해당 필드의 값을 변환하기**

   - ```java
     public class RefectionEx {
         public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
             Car car = new Car();
             Class<? extends Car> carClass = car.getClass();
             Field carClassField = carClass.getDeclaredField("carPosition");
             carClassField.setAccessible(true);
             
             System.out.println("변환 전 : " + carClassField.get(car));
             
             carClassField.set(car, new CarPosition(300));
             
             System.out.println("변환 후 : " + carClassField.get(car));
         }
     }
     
     // 출력 결과
     // 변환 전 : CarPosition{carPosition=100}
     // 변환 후 : CarPosition{carPosition=300}
     ```

     - Field 클래스의 **'set(Object obj, Object value)'** 메서드를 통해 필드의 값을 변경합니다.
       - obj : 필드의 값을 바꿀 인스턴스 객체
       - value : 변환 후의 값


<br/>

### 2) 메서드

일단 가져올 Car 클래스의 메서드들부터 보자

```java
package reflection;

public class Car {
    ... 생략
    
    public static void aaa() {
        System.out.println("aaa method");
    }
    
    private static void bbb() {
        System.out.println("bbb method");
    }
    
    public int ccc(int a, int b) {
        return a + b;
    }
    
    private int ddd() {
        return 100;
    }
    
    ... 생략
}
```

<br/>

1. **원하는 메서드를 가져오기**
   
   1. **public 메서드 가져오기**
   
      - ```java
        public class RefectionEx {
            public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
                Car car = new Car();
                Class<? extends Car> carClass = car.getClass();
                Arrays.stream(carClass.getMethods())
                        .forEach(method -> {
                            System.out.println(method.getName());
                            System.out.println(method);
                            System.out.println("================================================");
                        });
            }
        }
        
        // 출력 결과
        // getCarName
        // public reflection.CarName reflection.Car.getCarName()
        // ================================================
        // ccc
        // public int reflection.Car.ccc(int,int)
        // ================================================
        // aaa
        // public static void reflection.Car.aaa()
        // ================================================
        // getCarPosition
        // public reflection.CarPosition reflection.Car.getCarPosition()
        // ================================================
        // wait
        // public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
        // ================================================
        // wait
        // public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
        // ================================================
        // wait
        // public final void java.lang.Object.wait() throws java.lang.InterruptedException
        // ================================================
        // equals
        // public boolean java.lang.Object.equals(java.lang.Object)
        // ================================================
        // toString
        // public java.lang.String java.lang.Object.toString()
        // ================================================
        // hashCode
        // public native int java.lang.Object.hashCode()
        // ================================================
        // getClass
        // public final native java.lang.Class java.lang.Object.getClass()
        // ================================================
        // notify
        // public final native void java.lang.Object.notify()
        // ================================================
        // notifyAll
        // public final native void java.lang.Object.notifyAll()
        // ================================================
        ```
   
        - Class 클래스의 getMethods() 메서드를 통해 public 메서드의 배열을 가져온다.
   
        - 예시를 보면 알겠지만, Car 클래스의 public 메서드들과 Car 의 부모클래스인 Object 클래스의 public 메서드까지 가져오는 모습이다.
   
          <br/>
   
   2. **private 메서드까지 전부 가져오기**
   
      - ```java
        public class RefectionEx {
            public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
                Car car = new Car();
                Class<? extends Car> carClass = car.getClass();
                Arrays.stream(carClass.getDeclaredMethods())
                        .forEach(method -> {
                            System.out.println(method.getName());
                            System.out.println(method);
                            System.out.println("================================================");
                        });
            }
        }
        
        // 출력 결과
        // getCarPosition
        // public reflection.CarPosition reflection.Car.getCarPosition()
        // ================================================
        // aaa
        // public static void reflection.Car.aaa()
        // ================================================
        // ccc
        // public int reflection.Car.ccc(int,int)
        // ================================================
        // ddd
        // private int reflection.Car.ddd()
        // ================================================
        // getCarName
        // public reflection.CarName reflection.Car.getCarName()
        // ================================================
        // bbb
        // private static void reflection.Car.bbb()
        // ================================================
        ```
   
        - Class 클래스의 getDeclaredMethods() 메서드를 통해 Car 클래스의 public, private 메서드를 전부 가져온다.
   
        - 여기서 getMethods() 메서드와 차이점은 부모 클래스(Object)는 제외하고 해당 클래스의 메서드들만 가져온다는 것이다.
   
          <br/>
   
2. **해당 메소드를 호출하기**

   1. **인자 없는 private void 메서드 호출**

      - ```java
        public class RefectionEx {
            public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
                Car car = new Car();
                Class<? extends Car> carClass = car.getClass();
                Method bbbMethod = carClass.getDeclaredMethod("bbb");
                bbbMethod.setAccessible(true);
                
                bbbMethod.invoke(car);
            }
        }
        
        // 출력 결과
        // bbb method
        ```

        - 인자가 없는 메서드를 호출 시, getDeclaredMethod() 메서드엔 인자의 타입을 적어줄 필요 없이 메서드 이름만 전달해주면 된다.

          <br/>

   2. **인자와 반환 값이 존재하는 private int 메서드 호출**

      - ```java
        public class RefectionEx {
            public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
                Car car = new Car();
                Class<? extends Car> carClass = car.getClass();
                
                // 메서드 이름과 메서드 인자들의 타입을 Class 타입으로 전달한다.
                Method cccMethod = carClass.getDeclaredMethod("ccc", int.class, int.class);
                cccMethod.setAccessible(true);
            
                // invoke() 메서드 호출 시, 메서드를 실행할 인스턴스 객체와 인자의 값들을 전달한다.
                System.out.println(cccMethod.invoke(car, 10, 20));
            }
        }
        
        // 출력 결과
        // 30
        ```


<br/>

### 3) 생성자

생성자를 가져올 Car 클래스를 살펴보자

```java
package reflection;

public class Car {
    public final CarName carName;
    private final CarPosition carPosition;
    
    public Car() {
        this("car", 100);
    }
    
    private Car(String carName, int carPosition) {
        this(new CarName(carName), new CarPosition(carPosition));
    }
    
    public Car(CarName carName, CarPosition carPosition) {
        this.carName = carName;
        this.carPosition = carPosition;
    }
    
    ... 생략
    
    @Override
    public String toString() {
        return "Car{" +
                "carName=" + carName +
                ", carPosition=" + carPosition +
                '}';
    }
}
```

<br/>

1. **원하는 생성자를 가져오기**
   
   1. **public 생성자**
   
      1. **public 생성자 전부 가져오기**
   
         - ```java
           public class RefectionEx {
               public static void main(String[] args) {
                   Car car = new Car();
                   Class<? extends Car> carClass = car.getClass();
                   Arrays.stream(carClass.getConstructors())
                           .forEach(System.out::println);
               }
           }
           
           // 출력 결과
           // public reflection.Car(reflection.CarName,reflection.CarPosition)
           // public reflection.Car()
           ```
   
           - Class 클래스의 getConstructors() 메서드를 이용해서 public 생성자들을 가져온다.
      2. **public 생성자 하나만 가져오기**
   
         - ```java
           public class RefectionEx {
               public static void main(String[] args) {
                   Class<? extends Car> carClass = Car.class;
                   Constructor<? extends Car> constructor = carClass.getConstructor(CarName.class, CarPosition.class);
               
                   System.out.println(constructor);
               }
           }
           
           // 출력 결과
           // public reflection.Car(reflection.CarName,reflection.CarPosition)
           ```
   
           <br/>
   
   2. **private 생성자**
   
      1. **public, private 생성자 전부 가져오기**
   
         - ```java
           public class RefectionEx {
               public static void main(String[] args) {
                   Class<? extends Car> carClass = Car.class;
                   Arrays.stream(carClass.getDeclaredConstructors())
                           .forEach(System.out::println);
               }
           }
           
           // 출력 결과
           // public reflection.Car(reflection.CarName,reflection.CarPosition)
           // private reflection.Car(java.lang.String,int)
           // public reflection.Car()
           ```
   
           - Class 클래스의 getDeclaredConstructors() 메서드를 이용해서 public, private 생성자들을 전부 가져온다.
   
      2. **원하는 private 생성자 하나만 가져오기**
   
         - ```java
           public class RefectionEx {
               public static void main(String[] args) {
                   Class<? extends Car> carClass = Car.class;
                   Constructor<? extends Car> constructor = carClass.getDeclaredConstructor(String.class, int.class);
               
                   System.out.println(constructor);
               }
           }
           
           // 출력 결과
           // private reflection.Car(java.lang.String,int)
           ```
   
           - Class 클래스의 getDeclaredConstructor() 메서드를 이용해서 원하는 private 생성자를 가져온다.
   
           - 이 때, 인자는 생성자 인자들의 타입을 전달한다.
   
             <br/>
   
2. **해당 생성자를 통해 인스턴스 객체 생성하기**

   - ```java
     public class RefectionEx {
         public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
             Class<? extends Car> carClass = Car.class;
             Constructor<? extends Car> constructor = carClass.getDeclaredConstructor(String.class, int.class);
             constructor.setAccessible(true);
             Car car = constructor.newInstance("jun", 5000);
         
             System.out.println(car);
             System.out.println(car.getCarName());
             System.out.println(car.getCarPosition());
         }
     }
     
     // 출력 결과
     // Car{carName=CarName{name='jun'}, carPosition=CarPosition{carPosition=5000}}
     // CarName{name='jun'}
     // CarPosition{carPosition=5000}
     ```

     - getDeclaredConstructor() 메서드의 인자엔 생성자 인자들의 타입을, newInstance() 메서드의 인자엔 초기화 값을 전달한다.


<br/>

## 4. 리플렉션을 통해 여러가지 제약을 거는 테스트 코드 만들어보기

일단 테스트 리플렉션을 통해 할 Car 클래스를 살펴보자.

```java
package reflection;

public class Car {
    public final CarName carName;
    private final CarPosition carPosition;
    private CarPosition cc;
    
    public Car() {
        this("car", 100);
    }
    
    private Car(String carName, int carPosition) {
        this(new CarName(carName), new CarPosition(carPosition));
    }
    
    public Car(CarName carName, CarPosition carPosition) {
        this.carName = carName;
        this.carPosition = carPosition;
    }
    
    public static void aaa() {
        System.out.println("aaa method");
    }
    
    private static void bbb() {
        System.out.println("bbb method");
    }
    
    private int ccc(int a, int b) {
        return a + b;
    }
    
    private int ddd() {
        return 100;
    }
    
    public CarName getCarName() {
        return carName;
    }
    
    public CarPosition getCarPosition() {
        return carPosition;
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



- 각 클래스마다의 임의 규정 사항

  1. **인스턴스 변수 3개 이하로 제한**

     - ```java
       class CarTest {
           @Test
           void reflectionTest() {
               Class<Car> carClass = Car.class;
               long countOfInstanceField = Arrays.stream(carClass.getDeclaredFields())
                       .map(Field::getModifiers) // 각 필드의 modifiers 를 가져오고
                       .filter(Predicate.not(Modifier::isStatic)) // 그 modifiers 를 가지고 static이 아닌지 판단
                       .count(); // static 이 아닌 필드들의 개수 구하기
           
               System.out.println("countOfInstanceField : " + countOfInstanceField);
               assertThat(countOfInstanceField).isLessThanOrEqualTo(3);
           }
       }
       
       // 출력 결과
       // countOfInstanceField : 3
       
       // 테스트 성공
       ```
  
       <br/>
  
  1. **메서드 파라미터 3개 제한**
  
     - ```java
       class CarTest {
           @Test
           void reflectionTest() {
               Class<Car> carClass = Car.class;
               boolean anyMatch = Arrays.stream(carClass.getDeclaredMethods())
                       .anyMatch(method -> {
                           int parameterCount = method.getParameterCount();
                           System.out.printf("%s() 메서드의 인자 개수 : %d\n", method.getName(), parameterCount);
                           return parameterCount > 3;
                       });
               
               assertThat(anyMatch).isFalse();
           }
       }
       
       // 출력 결과
       // toString() 메서드의 인자 개수 : 0
       // bbb() 메서드의 인자 개수 : 0
       // getCarPosition() 메서드의 인자 개수 : 0
       // aaa() 메서드의 인자 개수 : 0
       // ccc() 메서드의 인자 개수 : 2
       // getCarName() 메서드의 인자 개수 : 0
       // ddd() 메서드의 인자 개수 : 0
       
       // 테스트 성공
       ```
  
       <br/>
  
  1. **static 메서드 2개 제한**
  
     - ```java
       class CarTest {
           @Test
           void reflectionTest() {
               Class<Car> carClass = Car.class;
               long countOfStaticMethod = Arrays.stream(carClass.getDeclaredMethods())
                       .map(Method::getModifiers)
                       .filter(Modifier::isStatic)
                       .count();
           
               System.out.println("countOfStaticMethod : " + countOfStaticMethod);
               assertThat(countOfStaticMethod).isLessThanOrEqualTo(2);
           }
       }
       
       // 출력 결과
       // countOfStaticMethod : 2
       
       // 테스트 성공
       ```
  
       <br/>
  
  1. **원하는 어노테이션을 사용한 메서드의 갯수 지정하기**
  
     - 그 전에 테스트 용으로 간단히 어노테이션 2개를 만든 후, Car 메서드들에 랜덤으로 붙여보자.
  
       - ```java
         @Retention(RetentionPolicy.RUNTIME)
         public @interface FirstCustom {}
         
         @Retention(RetentionPolicy.RUNTIME)
         public @interface SecondCustom {}
         ```
  
         
  
       - ```java
         package reflection;
         
         public class Car {
             ... 생략
             
             @FirstCustom
             public static void aaa() {
                 System.out.println("aaa method");
             }
             
             @SecondCustom
             private static void bbb() {
                 System.out.println("bbb method");
             }
             
             @FirstCustom
             @SecondCustom
             private int ccc(int a, int b) {
                 return a + b;
             }
             
             @SecondCustom
             private int ddd() {
                 return 100;
             }
             
             public CarName getCarName() {
                 return carName;
             }
             
             @SecondCustom
             public CarPosition getCarPosition() {
                 return carPosition;
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
  
         <br/>
  
     - 이제 각 어노테이션의 개수가 맞는지 테스트해보자
  
       - ```java
         class CarTest {
             @Test
             void reflectionTest01() {
                 Class<Car> carClass = Car.class;
                 long countOfFirstAnnotationMethod = Arrays.stream(carClass.getDeclaredMethods())
                         .map(method -> method.getAnnotation(FirstCustom.class))
                         .filter(Objects::nonNull)
                         .count();
             
                 System.out.println("countOfFirstAnnotationMethod : " + countOfFirstAnnotationMethod);
                 assertThat(countOfFirstAnnotationMethod).isEqualTo(2);
             }
             
             // 출력 결과
             // countOfFirstAnnotationMethod : 2
             
             // 테스트 성공
             
             @Test
             void reflectionTest02() {
                 Class<Car> carClass = Car.class;
                 long countOfSecondAnnotationMethod = Arrays.stream(carClass.getDeclaredMethods())
                         .map(method -> method.getAnnotation(SecondCustom.class))
                         .filter(Objects::nonNull)
                         .count();
                 
                 System.out.println("countOfSecondAnnotationMethod : " + countOfSecondAnnotationMethod);
                 assertThat(countOfSecondAnnotationMethod).isEqualTo(4);
             }
             
             // 출력 결과
             // countOfSecondAnnotationMethod : 4
             
             // 테스트 성공
         }
         ```
  
         
  

<br/>

## 5. 간단한 DI 만들어보기

직접 만든 @Inject 어노테이션을 붙인 필드는 인스턴스 객체를 자동으로 주입받도록 만들어보자.

먼저 테스트 할 Inject 어노테이션과 Car 클래스 그리고 테스트 코드부터 살펴보자.

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Inject {
}
```



```java
public class Car {
    public CarName carName;
    @Inject
    private CarPosition carPosition;
    
    public CarName getCarName() {
        return carName;
    }
    
    public CarPosition getCarPosition() {
        return carPosition;
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
class CarTest {
    @Test
    void reflectionTest() {
        Car car = RefectionEx.getObject(Car.class);
        System.out.println(car);
        
        assertAll(
                () -> assertThat(car).isNotNull(),
                () -> assertThat(car.getCarName()).isNull(),
                () -> assertThat(car.getCarPosition()).isNotNull()
        );
    }
}
```

<br/>

위의 테스트 코드가 통과 되려면 getObject() 메서드로 인스턴스를 생성 시, 해당 인스턴스와 그 인스턴스의 필드 중 @Inject 어노테이션이 붙은 것까지 인스턴스 주입이 되어야 한다. 이제 간단한 객체 주입 코드를 만들어보자.

```java
public class RefectionEx {
    public static <T> T getObject(Class<T> clazz) {
        // 해당 클래스 타입의 인스턴스 생성 및 대입
        T instance = createInstance(clazz);
        
        // 해당 인스턴스가 가지고 있는 필드를 하나씩 꺼낸다.
        Arrays.stream(instance.getClass().getDeclaredFields())
                .forEach(field -> {
                    
                    // 해당 필드에 @Inject 어노테이션이 붙어있는지 확인한다.
                    if (!Objects.isNull(field.getAnnotation(Inject.class))) {
                        try {
                            // private 인 경우를 대비해서 true 로 설정
                            field.setAccessible(true);
                            
                            // 위에서 만든 인스턴스 객체의 해당 필드에 필드 인스턴스를 생성해서 대입(set)한다.
                            field.set(instance, createInstance(field.getType()));
                        } catch (IllegalAccessException e) {
                            throw new RuntimeException(e);
                        }
                    }
                });
        
        // 인스턴스를 반환한다.
        return instance;
    }
    
    // 클래스 타입을 받고 해당 클래스 타입의 인스턴스 객체를 생성해서 반환
    private static <T> T createInstance(Class<T> clazz) {
        try {
            return clazz.getConstructor().newInstance();
        } catch (NoSuchMethodException | InvocationTargetException | InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }
}
```

<br/>

5번 예제는 인프런의 백기선님의 강의를 들은 후 굉장히 신기해서, 복습 차원에서 혼자서 다시 만들어 본 DI 프레임워크이다. 이 외의 더 자세한 내용을 원하면 백기선님의 강의를 듣기를 바란다.