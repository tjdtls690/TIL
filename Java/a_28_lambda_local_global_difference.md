# 목차

1. [Effectively Final 이란??](#1-effectively-final-이란) <br/>
2. [람다식에서의 로컬 변수는 (Effectively) Final 이어야 하는 이유](#2-람다식에서의-로컬-변수는-effectively-final-이어야-하는-이유) <br/>
3. [람다식에서의 멤버 변수의 값 변경이 가능한 이유](#3-람다식에서의-멤버-변수의-값-변경이-가능한-이유) <br/>

<br/>

# [자바, Java] 람다 (Lambda) - 로컬 변수와 멤버 변수의 차이점과 이유

<br/>

> **이전 글에서 람다는 순수 함수가 아니라는 의견을 제시해 보았다.**
>
> **이제 람다식에서의 로컬 변수와 멤버 변수의 차이점에 대해 알아보려 한다.**

<br/>

## 1. Effectively Final 이란??

먼저 Effectively Final 이 무엇인지부터 보자. **자바 코드상에서 final이 붙지 않은 변수인데, 단 한번도 해당 변수의 값이 변경되지 않고 프로그램이 종료되었을 때의 그 변수**를 **'사실상 final'** 즉, **Effectively Final** 이라고 한다. **람다식 안에서 사용될 수 있는 로컬 변수는 Effectively Final 변수뿐**이다. 코드상에서 보자면,

```java
class Scratch {
    void run () throws IOException {
        int a = 20;
        System.out.println(a + 10);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 30
```

위 예제의 변수 a는 final 이 안붙어 있지만, 사실상 final 즉, Effectively Final 로 취급된다. 프로그램이 시작하고 끝날 때까지 단 한번도 값이 변한 적이 없기 때문이다. 이러한 특징이 람다식에서 나타나는 경우가 있다.

<br/>

```java
class Scratch {
    void run () throws IOException {
        int a = 20;
        
        Function<Integer, Integer> function = number -> {
            a++; // 이 부분에서 에러
            return number + a;
        };
        
        System.out.println("a : " + a);
        System.out.println("lambda result : " + function.apply(10));
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 실행 결과
// C:\Users\tjdtl\AppData\Roaming\JetBrains\IntelliJIdea2022.2\scratches\scratch.java:22:13
// local variables referenced from a lambda expression must be final or effectively final
```

람다식 안에서 참조하는 로컬 변수의 값은 변경할 수 없다. 에러 내용을 보면 **'람다 식에서 참조하는 지역 변수는 최종적이거나 사실상 최종적이어야 합니다.'** 라고 뜨는 모습을 볼 수 있다. 컴파일을 할 때부터 나타나는 컴파일 에러다.

<br/>

## 2. 람다식에서의 로컬 변수는 (Effectively) Final 이어야 하는 이유

이유는, <code><strong>람다식에서 사용되는 로컬 변수는 해당 변수 그 자체가 아니라 복사본이기 때문이다.</strong></code> 복사본일 수밖에 없는 이유부터 살펴보자.

1. **로컬 변수와 해당 람다식의 생명 주기는 다르다.**

   - 메서드가 종료되어 호출 스택에서 사라지게 될 때, 해당 메서드 안의 로컬 변수들도 생명주기가 같이 끝나게 되어 사라진다.

   - 하지만 람다는 **'고차 함수'** 이기 때문에 **람다식(함수) 자체를 인자로 받고 다시 리턴도 할 수 있기에**, 메서드의 외부에서 생성되어 들어왔을 가능성도 있다.

   - 그런 경우, 해당 메서드와 그 안에 있는 로컬 변수의 생명 주기가 끝난다 해도, 람다식의 생명주기는 여전히 실행되고 있을 수 있다.

   - 결국 람다가 참조한 로컬 변수의 생명 주기가 끝난다 해도, 람다는 해당 로컬 변수의 값을 복사해서라도 갖고 있어야 한다.

     <br/>

2. **로컬 변수를 다루는 Thread 와 람다식을 다루는 Thread 가 서로 다른 Thread 이다.**

   - 이 특징은 **Thread Safe 를 위한 특징**이라고도 볼 수 있다.
   - **로컬 변수는 JVM 메모리 구조 상 Stack 에 저장된다.**
     - Stack 은 하나의 Thread 에 하나씩 배정된다.
     - 즉, 해당 Thread 가 종료되면, 그 안의 로컬 변수는 사라진다는 점을 먼저 인지하자.
   - **람다는 별도의 Thread 에서 실행된다.**
     - **만약 복사가 아니라 직접 스택에 접근하여 참조하는 식이라면, 멀티 쓰레드 환경에서 굉장히 위험해진다.**
       - 람다가 참조한 로컬 변수를 관리하는 Thread 가 종료되었을 때, 
       - 람다를 실행한 Thread에서 이미 생명주기가 끝난 Thread 에 접근하려 할 수 있다.

<br/>

## 3. 람다식에서의 멤버 변수의 값 변경이 가능한 이유

예제처럼 멤버 변수는 람다식 안에서도 변경이 가능하다. 이전 글에서 람다는 순수 함수가 아니라는 의견의 근거로 말한 부분이기도 하다.

```java
class Scratch {
    int a = 10;
    
    void run () throws IOException {
        Function<Integer, Integer> function = number -> {
            a++;
            return number + a;
        };
        
        System.out.println("a : " + a);
        System.out.println("lambda result : " + function.apply(10));
        System.out.println("a : " + a);
        System.out.println("lambda result : " + function.apply(10));
        System.out.println("a : " + a);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// a : 10
// lambda result : 21
// a : 11
// lambda result : 22
// a : 12
```

<br/>

그렇다면 로컬 변수와 다르게 멤버 변수는 왜 람다식에서 Effectively Final 이 아니어도 되는걸까?? 멤버 변수는 로컬 변수와 다르게 **모든 Thread 에서 접근 가능한 'Heap' 영역에 저장이 된다.** 여기서 멤버 변수의 값을 람다 안에서 변경 가능한 이유가 나온다.

1. **람다의 생명 주기가 끝날 때까지 람다가 참조한 멤버 변수의 생존이 보장된다.**
   - GC(Garbage Collector) 는 Heap 영역의 데이터를 처리할 때, 현재 사용되고 있는 데이터가 있다면 바로 회수하지 않는다.
2. **람다가 참조한 멤버 변수의 값이 최신 값임이 보장된다.**

람다가 참조하는 멤버 변수의 변경이 가능한 만큼 멀티 쓰레드 환경에서의 동기화 처리에 신경을 써줄 필요성이 보인다.