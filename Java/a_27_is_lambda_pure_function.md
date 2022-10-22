# 목차

1. [람다는 정말 순수 함수가 맞아??](#1-람다는-정말-순수-함수가-맞아) <br/>
2. [그럼 람다는 순수 함수가 아니니 안좋은건가??](#2-그럼-람다는-순수-함수가-아니니-안좋은건가) <br/>

<br/>

# [자바, Java] 람다 (Lambda) 는 순수 함수가 정말 맞아??

<br/>

> **람다를 공부하면서 아주 많은 곳에서 '람다 == 순수함수' 라고 설명을 한다는 것을 알게 되었다.**
>
> **이에 대한 반론을 제기해보려 한다.**

<br/>

## 1. 람다는 정말 순수 함수가 맞아??

많은 강의와 많은 블로그에서 람다는 **'순수 함수'** 라고 한다. 순수 함수는 **같은 값을 넣었을 때, 같은 값이 안나올 수 있는 여지가 아예 없는 함수**이다. 즉, **어떤 시점에서든 input 이 같다면, output 도 항상 같아야 하는 함수**를 말한다.



하지만 직접 람다를 많이 사용해본 입장에서, <code><strong>"람다는 순수 함수다."</strong></code> 라는 의견에 절대 동의할 수 없다. 왜냐하면 **자바는 람다식 안에서도 순수 함수의 특징들을 깨트릴만한 문법들이 너무 많이 허용되고 있기 때문**이다. 자바에서의 람다는 결국 **'인터페이스를 구현한 특수한 객체'**일 뿐이다. 순수 함수에 반하는 람다의 허용 문법들을 살펴보자.

1. **매개변수로 들어오는 값 외의 상태 값을 가지는 것이 허용된다. 외부든 내부든.**

   - 상태 값을 가질 수 있는 것부터 이미 **순수 함수의 특징을 깨트릴 여지가 생긴 것**이다.

   - ```java
     class Scratch {
         int a = 10;
         
         void run () throws IOException {
             int b = 20;
             Function<Integer, Integer> function = number -> {
                 int c = 30;
                 return number + a + b + c;
             };
             System.out.println(function.apply(10));
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // 70
     ```

     <br/>

2. **상태 값의 변경이 가능하다.**

   - 로컬 상태 값만 effectively final 일 뿐, **클래스 필드나 람다식 내부의 상태 값은 얼마든지 변경이 가능하다.**

     - 밑의 예제에서 같은 값을 넣었는데도 필드 값의 변경에 의해 다른 결과가 나오는 모습을 볼 수 있다.

   - ```java
     class Scratch {
         int a = 10;
         
         void run () throws IOException {
             int b = 20;
             
             Function<Integer, Integer> function = number -> {
                 int c = 30;
                 a++;
     //            b++; // 로컬 변수 b만 변경 불가능
                 c++;
                 return number + a + b + c;
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
     // lambda result : 72
     // a : 11
     // lambda result : 73
     // a : 12
     ```

     <br/>

   - 뿐만 아니라, 로컬 객체라 할지라도 주소값만 바뀌지 않는다면 해당 객체의 내부 값은 얼마든지 변경이 가능하다.

   - ```java
     class Scratch {
         void run () throws IOException {
             List<Integer> list = new ArrayList<>();
             list.add(1);
             System.out.println(list);
             Runnable runnable = () -> {
                 list.add(2);
                 list.set(0, 3);
                 System.out.println(list);
             };
         
             runnable.run();
         }
         
         public static void main(String[] args) throws IOException {
             new Scratch().run();
         }
     }
     
     // 출력 결과
     // [1]
     // [3, 2]
     ```

     <br/>

지금까지의 람다에 대한 개인적인 결론 

1. 람다는 개발자의 역량에 따라 불변성을 보장 해줄 수 있을 뿐, **절대적인 불변성을 보장하지 못한다.**
2. 람다는 순수 함수로 사용 될지 말지 개발자에 의해 선택될 뿐, **'람다 == 순수함수'는 아니다.**
3. 람다는 **익명 클래스보다 더 사용하기 편리한, 특수한 객체일 뿐**이다.

<br/>

## 2. 그럼 람다는 순수 함수가 아니니 안좋은건가??

그건 절대 아니라고 생각한다. 함수형 프로그래밍은 엄연히 장단점이 존재한다. 오히려 자바의 람다는 **개발자의 역량에 따라 함수형 프로그래밍의 장점을 살리면서도, 허용된 문법 안에서 더 유연한 사용이 가능한 굉장히 좋은 시스템**이라고 생각한다. 



- **함수형 프로그래밍의 장점**

  1. **Thread-safe**

     - 어느 시점이던지 같은 값이 들어가면 같은 결과가 나오기 때문에 병렬 처리에 안전하다.

  2. **Side Effect 가 없다.**

     - 그래서 버그나 부작용의 확률이 적어진다.
     - 프로그램의 동작 예측이 수월해진다.

  3. **테스트하기 쉽다.**

     - Input 과 Output 이 명확하기 때문에 단위 테스트가 용이하다.

  4. **재사용성이 좋아진다.**

     - 중복되는 코드의 양을 최대한 줄일 수 있다.

     <br/>

- **함수형 프로그래밍의 단점**

  1. **가독성이 좋지 않을 수 있다.**
     - 오히려 순수 함수의 원칙을 지키려해서 코드가 불필요하게 복잡해지는 경우가 생길 수 있다.
  2. **각 순수 함수를 조합하여 프로그래밍을 하는 것이 난이도가 쉽지 않을 수 있다.**

<br/>

프로그래머의 역량에 따라, 함수형 프로그래밍의 장점을 살리면서 단점을 보완한 자바만의 함수형 프로그래밍을 구현할 수 있다고 생각한다. 이 글을 작성하면서, 람다 안에선 왜 멤버 필드는 변경 가능하고 로컬 변수는 변경이 불가능한지에 대해 깊게 다루어보고 싶어졌다. 다음 글에선 그 부분에 대해 학습하고 정리해보려 한다.