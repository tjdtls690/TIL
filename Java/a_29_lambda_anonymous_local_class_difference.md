# 목차

1. [쉐도잉이란 무엇인가??](#1-쉐도잉이란-무엇인가) <br/>
2. [로컬 클래스와 익명 클래스의 쉐도잉](#2-로컬-클래스와-익명-클래스의-쉐도잉) <br/>
3. [람다의 쉐도잉](#3-람다의-쉐도잉) <br/>
4. [람다 <-> 로컬 클래스, 익명 클래스의 쉐도잉 차이점이 생긴 이유](#4-람다---로컬-클래스-익명-클래스의-쉐도잉-차이점이-생긴-이유) <br/>

<br/>

# [자바, Java] 람다 (Lambda) - 쉐도잉 (Shadowing)

<br/>

> **'람다' 와 '로컬 클래스, 익명 클래스' 의 차이점에 대해 학습한 내용을 정리해보려 한다.**
>
> **처음 람다를 공부할 때, 익명 클래스와 똑같이 생각하면 되겠다는 생각을 갖고 있었는데,**
>
> **프로그램 학습 구현을 하면서 람다와 익명 클래스의 쉐도잉 차이점이 있다는 것을 알게 되었고, 그 부분에 대해서 정리해보려 한다.**

<br/>

## 1. 쉐도잉이란 무엇인가??

**어떤 블록(구역) 안에 또 다른 블록이 있을 때, 밖의 블록 범위 중 일부분을 안에 있는 블록이 차지하는 것을 말한다.** 밖의 블록에서 선언된 변수와 안의 블록에서 선언된 변수의 이름이 같을 시, 안의 블록 안에선 안의 블록에서 선언된 변수가 우선권을 갖는 것을 의미한다. **즉, 다른 Scope 으로 설정된다는 것이다.** 이 예제에서 **멤버 필드인 a보다 로컬 변수 a가 더 우선시되어 20이 출력되는 모습**을 볼 수 있다. 이것이 **'쉐도잉'**이다.

```java
class Scratch {
    int a = 10;
    
    void run () throws IOException {
        int a = 20;
        System.out.println(a);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 20
```

<br/>

## 2. 로컬 클래스와 익명 클래스의 쉐도잉

**로컬 클래스와 익명 클래스는 쉐도잉이 일어난다.** 블록 밖의 변수 이름과 블록 안의 변수 이름이 같을 시, 블록 안의 변수가 더 우선시 됨을 예제를 통해 볼 수 있다.

```java
class Scratch {
    void run () throws IOException {
        int a = 10;
        
        final class LocalClass {
            void printA() {
                int a = 20;
                System.out.println(a);
            }
        }
    
        final Runnable runnable = new Runnable() {
            @Override
            public void run() {
                int a = 30;
                System.out.println(a);
            }
        };
    
        LocalClass localClass = new LocalClass();
        localClass.printA();
        
        runnable.run();
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 20
// 30
```

<br/>

## 3. 람다의 쉐도잉

**람다는 쉐도잉이 일어나지 않는다.** **즉, 밖의 블록과 람다의 블록이 같은 Scope 로 취급받는다는 것**을 알 수 있다. 이 예제처럼 블록 밖의 변수 이름과 블록 안의 변수 이름이 같을 시, 이미 run() 메서드에 같은 이름의 변수가 선언되어있다면서 에러가 나게 된다.

```java
class Scratch {
    void run () throws IOException {
        int a = 10;
    
        final Consumer<Integer> consumer = (a) -> System.out.println(a + 10); // 컴파일 에러
    
        consumer.accept(20);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// ... 생략
// variable a is already defined in method run()
```

```java
class Scratch {
    void run () throws IOException {
        int a = 10;
    
        final Consumer<Integer> consumer = (num) -> {
            int a = 20; // 컴파일 에러
            System.out.println(num + a);
        };
    
        consumer.accept(1);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// ... 생략
// variable a is already defined in method run()
```

<br/>

## 4. 람다 <-> 로컬 클래스, 익명 클래스의 쉐도잉 차이점이 생긴 이유

1. 로컬 클래스와 익명 클래스는 메서드 안에서 생성될 시 별도의 클래스 파일 즉, 별도의 Scope 가 생기기 때문이다.
2. 람다는 별도의 클래스로 생성되지 않기 때문에 같은 Scope 로 취급 받는다.

이러한 부분 때문에 눈여겨 볼만한 특징이 한 가지 있다. 바로 **'this' 의 차이점**이다. 익명 클래스, 로컬 클래스의 this 와 람다에서의 this 는 다른 객체를 가리키게 된다. 예제를 보면, **익명 및 로컬 클래스의 this 는 현재 클래스와 다른 주소 값을 가리키고, 람다는 현재 클래스와 같은 주소 값을 가리키는 모습을 볼 수 있다.**

```java
class Scratch {
    void run () throws IOException {
        System.out.println(this + " : Current Class");
    
        final Runnable runnable1 = () -> System.out.println(this + " : Lambda");
        
        final class LocalClass {
            void printA() {
                System.out.println(this + " : Local Class");
            }
        }
    
        final Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println(this + " : Anonymous Class");
            }
        };
        
        runnable1.run();
        new LocalClass().printA();
        runnable2.run();
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// Scratch@135fbaa4 : Current Class
// Scratch@135fbaa4 : Lambda
// Scratch$1LocalClass@3b9a45b3 : Local Class
// Scratch$1@7699a589 : Anonymous Class
```

<br/>

**로컬 및 익명 클래스는 별도의 객체를 생성해서 별도의 Scope 로 인정받고, 람다는 같은 Scope 로 취급받는다는 사실을 알고 있다면, 이는 당연한 현상인 것이다.**