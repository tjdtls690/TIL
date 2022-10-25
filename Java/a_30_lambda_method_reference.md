# 목차

1. [메서드 레퍼런스(참조)란??](#1-메서드-레퍼런스참조란) <br/>
2. [메서드 레퍼런스의 장점](#2-메서드-레퍼런스의-장점) <br/>
3. [메서드 레퍼런스의 형태](#3-메서드-레퍼런스의-형태) <br/>
4. [JDK 라이브러리를 메서드 레퍼런스 형태로 사용한 예제](#4-jdk-라이브러리를-메서드-레퍼런스-형태로-사용한-예제) <br/>

<br/>

# [자바, Java] 람다 (Lambda) - 메서드 레퍼런스

<br/>

> **함수형 인터페이스와 람다를 처음 배울 때, 가장 이해하기 힘들었던 부분이 메서드 레퍼런스였다.**
>
> **사실 그땐 람다를 100% 이해하지도 못한 상황이었어서 당연한 현상이긴 하지만,**
>
> **이번 기회에 더 확실하게 정립하기 위해 다시 한번 정리해보려 한다.**

<br/>

## 1. 메서드 레퍼런스(참조)란??

**람다 표현식에서 메서드 단 하나만 호출하는 형태일 때, 해당 기능을 갖고있는 메서드가 이미 존재하는 경우 그 메서드를 참조해서 그 메서드를 호출하는 형태를 의미한다.** 어렵게 생각할 필요 없이, 갖다 쓰려는 메서드를 참조한다고 생각하면 된다.

<br/>

## 2. 메서드 레퍼런스의 장점

가장 큰 장점은 별거없다. 단지 **일반적인 매개변수 있는 람다 표현식보다 훨씬 더 간결하게 작성 가능하기 때문**이다. 예제를 보면 알 수 있다.

```java
class Scratch {
    void run () throws IOException {
        Function<Integer, String> function1 = this::parseString; // 메서드 레퍼런스
        Function<Integer, String> function2 = (num) -> this.parseString(num); // 일반적인 람다 표현식
    
        System.out.println(function1.apply(10));
        System.out.println(function2.apply(10));
    }
    
    String parseString(int num) {
        return String.valueOf(num);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 10
// 10
```

<br/>

## 3. 메서드 레퍼런스의 형태

메서드 레퍼런스는 크게 **4가지의 형태**가 있다. 밑의 예제들을 통해 일반적인 람다 표현식과 비교하면서 메서드 레퍼런스를 분석해보자.

1. **타입::스태틱 메서드**

   - **모든 파라미터가 스태틱 메서드의 파라미터로 전달된다.**

   - ```java
     class Scratch {
         void run () throws IOException {
             // function1 와 function2 는 같은 의미의 코드
             Function<Integer, String> function1 = (num) -> Scratch.parseString(num); // 일반적인 람다 표현식
             Function<Integer, String> function2 = Scratch::parseString; // 메서드 레퍼런스
         
             System.out.println(function1.apply(10));
             System.out.println(function2.apply(10));
         }
         
         static String parseString(int num) {
             return String.valueOf(num);
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 10
     // 10
     ```

     <br/>

2. **타입::인스턴스 메서드**

   - **첫번째 파라미터가 메서드 호출자가 되고, 그 뒤의 파라미터들이 메서드의 파라미터로 전달되는 형태이다.**

   - 1번과 같은 형태여서 헷갈릴 수 있다.

   - ```java
     class Scratch {
         void run () throws IOException {
             // biFunction1 와 biFunction2 는 같은 의미의 코드
             BiFunction<Scratch, Integer, String> biFunction1 = (scratch, number) -> scratch.parseString(number); // 일반적인 람다 표현식
             BiFunction<Scratch, Integer, String> biFunction2 = Scratch::parseString; // 메서드 레퍼런스
         
             System.out.println(biFunction1.apply(this, 10));
             System.out.println(biFunction2.apply(this, 10));
         }
         
         String parseString(int num) {
             return String.valueOf(num);
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 10
     // 10
     ```

     <br/>

3. **객체 레퍼런스::인스턴스 메서드**

   - **특정 객체의 인스턴스 메서드를 참조하는 형태이다.**

   - 외부의 특정 객체 참조 변수를 통해 메서드를 호출한다.

   - ```java
     class Scratch {
         void run () throws IOException {
             // function1 와 function2 는 같은 의미의 코드      
             final Scratch scratch = new Scratch();
             Function<Integer, String> function1 = (number) -> scratch.parseString(number); // 일반적인 람다 표현식
             Function<Integer, String> function2 = scratch::parseString; // 메서드 레퍼런스
         
             System.out.println(function1.apply(10));
             System.out.println(function2.apply(10));
         }
         
         String parseString(int num) {
             return String.valueOf(num);
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 10
     // 10
     ```

   - ```java
     class Scratch {
         void run () throws IOException {
             // function1 와 function2 는 같은 의미의 코드
             Function<Integer, String> function1 = (number) -> this.parseString(number); // 일반적인 람다 표현식
             Function<Integer, String> function2 = this::parseString; // 메서드 레퍼런스
         
             System.out.println(function1.apply(10));
             System.out.println(function2.apply(10));
         }
         
         String parseString(int num) {
             return String.valueOf(num);
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 10
     // 10
     ```

     <br/>

4. **타입::new**

   - **생성자를 참조하는 형태이다.**

   - 예제를 보면 람다로 들어오는 매개변수의 타입이나 개수가 다를 때, 그에 맞춰서 **다른 생성자가 실행**되는 모습을 볼 수 있다.

   - ```java
     class Scratch {
         private final int number;
         
         public Scratch() {
             number = 100;
         }
         
         public Scratch(final int number) {
             this.number = number;
         }
         
         public Scratch(String number) {
             this.number = Integer.parseInt(number) + 500;
         }
         
         void run () throws IOException {
             // supplier1 와 supplier2 는 같은 의미의 코드
             Supplier<Scratch> supplier1 = () -> new Scratch(); // 일반적인 람다 표현식
             Supplier<Scratch> supplier2 = Scratch::new; // 메서드 레퍼런스
             
             // function1 와 function2 는 같은 의미의 코드
             Function<Integer, Scratch> function1 = (number) -> new Scratch(number); // 일반적인 람다 표현식
             Function<Integer, Scratch> function2 = Scratch::new; // 메서드 레퍼런스
         
             // function3 와 function4 는 같은 의미의 코드
             Function<String, Scratch> function3 = (stringNumber) -> new Scratch(stringNumber); // 일반적인 람다 표현식
             Function<String, Scratch> function4 = Scratch::new; // 메서드 레퍼런스
         
             System.out.println(supplier1.get().getNumber());
             System.out.println(supplier2.get().getNumber());
             System.out.println();
         
             System.out.println(function1.apply(200).getNumber());
             System.out.println(function2.apply(200).getNumber());
             System.out.println();
         
             System.out.println(function3.apply("200").getNumber());
             System.out.println(function4.apply("200").getNumber());
         }
         
         public int getNumber() {
             return number;
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 100
     // 100
     // 
     // 200
     // 200
     // 
     // 700
     // 700
     ```

     <br/>

## 4. JDK 라이브러리를 메서드 레퍼런스 형태로 사용한 예제

가장 헷갈릴 법한 형태인 **'타입::인스턴스 메서드'** 를 예로 들어보겠다. 총 3가지 형태로 예를 들어서 문자열 리스트를 정렬시켜 볼 것이다.

1. **메서드 레퍼런스**
2. **JDK 에서 제공하는 '두 문자열을 비교하는 라이브러리'**
3. **JDK 에서 제공하는 '정렬 전략 패턴'**

```java
class Scratch {
    void run () throws IOException {
        final List<String> strings1 = Arrays.asList("bbb", "ccc", "aaa", "eee", "ddd");
        final List<String> strings2 = Arrays.asList("bbb", "ccc", "aaa", "eee", "ddd");
        final List<String> strings3 = Arrays.asList("bbb", "ccc", "aaa", "eee", "ddd");
        
        // 3개의 줄 모두 같은 의미의 코드
        // 1)
        strings1.sort(String::compareTo); // String 의 compareTo 메서드를 메서드 레퍼런스 형식으로 호출
        
        // 2)
        strings2.sort((str1, str2) -> str1.compareTo(str2)); // String 의 compareTo 메서드를 일반 람다 표현식으로 호출
        
        // 3)
        strings3.sort(Comparator.naturalOrder()); // 자바에서 정렬 패턴 형식을 지정하라고 제공해준 정렬 전략 패턴 리턴 메서드
        									  // 해당 Comparator.naturalOrder() 메서드는 오름차순 정렬 패턴으로 설정해준다.
        
        System.out.println(strings1);
        System.out.println(strings2);
        System.out.println(strings3);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// [aaa, bbb, ccc, ddd, eee]
// [aaa, bbb, ccc, ddd, eee]
// [aaa, bbb, ccc, ddd, eee]
```

<br/>

사실 처음에는 굳이 이걸 왜 사용하나 싶을 정도로 더 헷갈리고 사용하기 어려울 지 모르지만, 자꾸 쓰다보면 오히려 더 익숙해지고 편해져서 알아서 잘 사용하게 될 것이다. 나도 지금은 일반적인 람다 표현식보다 메서드 레퍼런스가 훨씬 간결해서 사용하기 편하다.