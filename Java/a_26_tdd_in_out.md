# 목차

1. [TDD 의 사이클](#1-tdd-의-사이클) <br/>
2. [내가 TDD 를 학습하면서 느꼈던 장점](#2-내가-tdd-를-학습하면서-느꼈던-장점) <br/>
3. [TDD 가 어디서 어떻게 시작할 지 어렵거나 막막한 이유](#3-tdd-가-어디서-어떻게-시작할-지-어렵거나-막막한-이유) <br/>
4. [해결책](#4-해결책) <br/>
5. [내가 해결책들을 실제로 이용한 방식](#5-내가-해결책들을-실제로-이용한-방식) <br/>

<br/>

# [자바, Java] TDD - Out-In 보단 In-Out 방식

<br/>

> **사실 nextstep 미션들을 진행하면서 느꼈던 부분은,**
>
> **그냥 단위 테스트를 추가해 주는것과 TDD 는 난이도가 완전 다르다는 것이다.**
>
> **맨 처음 미션의 요구사항을 분석하고 테스트 코드를 짜려고 해도,**
>
> **TDD 를 어디서 어떻게 시작해야 할 지 막막해 지는 경우가 굉장히 많다.**
>
> **이러한 부분들을 공통 피드백을 통해 학습하고 해결해 나간 과정을 간단히 정리해보려 한다.**

<br/>

## 1. TDD 의 사이클

TDD 가 무엇인지에 대한 내용까지 정리하려면 너무 많은 시간과 글이 들어가기 때문에 간단히 사이클에 대해서만 정리해보려 한다.

1. **구현하려는 메서드에 대해 실패하는 테스트 코드를 먼저 작성한다.**
2. **실패한 테스트 코드가 성공하기 위해 프로덕션 코드를 구현한다.**
3. **오로지 테스트 성공만을 위해 막 구현한 프로덕션 코드를 리팩토링 한다.**

<br/>

## 2. 내가 TDD 를 학습하면서 느꼈던 장점

1. **버그 발생의 횟수가 현저히 줄거나 혹은 아예 없는 경우도 있다.**
   - 미션 규모가 큰게 아니여서 아예 없는 경우도 나오긴 하더라.
   - 메서드 하나하나 테스트를 통과시키면서 진행을 하니, 버그가 줄어드는 것은 당연하다.
2. **발생한 버그를 잡기가 매우 수월해진다.**
   - 사실 테스트 코드만 꼼꼼히 잘 작성해 놓으면, 
   - 프로그램을 직접 실행하면서가 아니라 전체의 테스트 코드를 실행하면서 버그가 발생한 곳을 바로 잡아낼 수 있다.
   - 실제로 프로그램 구현 완료 될 때까지 테스트 코드만 한 번씩 돌려주면서 버그 발생 체크만 해줘도, 구현 완료된 후엔 버그가 없는 경우가 많다.
3. **리팩토링이 굉장히 수월해진다.**
   - 하드코딩이 점철된 코드는 코드를 조금만 건드려도 여기 저기서 버그가 빵빵 터져서 리팩토링이 두려워진다.
   - TDD 방식이 리팩토링하기 수월한 이유
     1. **TDD 방식으로 개발을 하면, 애초부터 메서드의 기능분리가 잘 되어진다.**
        - 메서드 단위로 하나씩 구현을 해나가기 때문에, 역할 분담이 잘 되어진다.
     2. **테스트 코드가 뒷받침 되기 때문에 리팩토링이 전혀 두렵지 않다.**
        - 테스트 코드 통과를 유지하면서 리팩토링을 해나가면 안전한 리팩토링이 이루어진다.
        - 리팩토링을 하면서 버그가 발생하는지 안하는지 바로바로 체크하면서 진행이 가능하다.
   - **TDD 는 테스트 코드라는 나의 동반자를 내 옆에 두고 의지할 수 있게 한다.**

<br/>

## 3. TDD 가 어디서 어떻게 시작할 지 어렵거나 막막한 이유

처음 TDD 란 개념을 nextstep 에서 배웠을 때, 굉장히 막막했다. 지금까지 TDD 가 어렵게 느껴진 이유를 정리해보자면,

1. **어떤 메서드부터 TDD 를 시작하느냐에 따라 난이도가 천차만별이다.**
   - TDD 사이클 대로 테스트 코드부터 작성 했는데, 테스트 하려는 해당 메서드의 기능이 너무 거대한 기능이어서, 
   - 해당 테스트 하나만 통과시키는 데에 굉장히 많은 프로덕션 코드를 구현해야하는 상황이 오는 경우가 있다. 
2. **랜덤 요소와 같은 테스트 하기 힘든 코드의 테스트가 힘들다.**
   - 1 ~ 9 까지의 숫자 중 랜덤으로 반환하는 메서드는 대체 어떻게 테스트 해야 할까??

<br/>

## 4. 해결책

1. **설계할 때부터 기능을 최대한 작은 단위로 쪼갠다.**
   - 보통 프로그램을 구현할 때 In-Out 이 아닌 Out-In 방식으로 구현한다.
   - 왜냐하면 그 방식이 더 쉽기 때문이다.
   - 그러나 TDD 의 관점에선 굉장히 어려워지는 방식이다.
   - TDD 는 In-Out 방식처럼 아주아주 작은 기능부터 구현을 해나가야 수월해 진다.
   - 그래서 **구현 이전에 설계부터 잘 해서 아주 작은 기능 단위부터 추출해서 TDD 를 시작해야 한다.**
2. **테스트 하기 힘든 요소와 가능한 요소를 분리해서 테스트할 줄 알아야 한다.**
   - 나 같은 경우는 보통 전략 패턴(인터페이스)를 사용해서 테스트 하기 힘든 코드 안에서 테스트 가능한 코드를 추출한다.

<br/>

## 5. 내가 해결책들을 실제로 이용한 방식

지금부터 이 글을 작성하면서 즉석으로 프로그램을 구현한 코드를 예제로 삼아보겠다.

1. **설계할 때부터 기능을 최대한 작은 단위로 쪼갠 뒤, In-Out 방식으로 해결한다.**

   - 일단 예제용으로 구현할 아주 간단한 프로그램은 '두 숫자의 사칙연산을 수행하는 프로그램'이다.

   - 여기서 TDD 로 구현할 첫번째 메서드로 만약 '두 숫자의 사칙연산을 전부 수행하는 기능'을 선택한다면 굉장히 골치 아파진다.

     - ```java
       public class CalculatorTest {
           @ParameterizedTest
           @DisplayName("두 숫자의 사칙연산 기능")
           @CsvSource(value = {"3 + 2, 5", "3 - 2, 1", "3 * 2, 6", "3 / 2, 1"})
           void calculate(String formula, int result) {
               Calculator calculator = new Calculator();
               assertThat(calculator.calculate(formula)).isEqualTo(result);
           }
       }
       ```

     - 첫번째로 구현할 메서드부터 모든 사칙연산이 가능한 기능을 구현해야한다.

     - 이런식으로 TDD 를 하게되면 막막하고 어려워지기 때문에 TDD를 포기해버리게 된다.

   - **이제 TDD를 하기에 앞서 기능을 최대한 작은 단위로 쪼개본다.**

     - 두 숫자의 모든 사칙연산을 구현해야 한다.

     - 그럼 각 사칙연산 기능부터 하나씩 구현하는 것이 좋겠네.

     - 더하기 기능부터 만들어보자.

     - ```java
       public class AddTest {
           @Test
           @DisplayName("더하기")
           void add() {
               Add add = new Add();
               assertThat(add.add(Arrays.asList("4", "2"))).isEqualTo(6);
           }
       }
       
       public class Add {
           public int add(final List<String> numbers) {
               return Integer.parseInt(numbers.get(0)) + Integer.parseInt(numbers.get(1));
           }
       }
       ```

       <br/>

     - 빼기, 곱하기, 나누기도 차례대로 TDD 방식으로 구현해보자.

     - ```java
       public class SubtractTest {
           @Test
           @DisplayName("빼기")
           void subtract() {
               Subtract subtract = new Subtract();
               assertThat(subtract.subtract(Arrays.asList("4", "2"))).isEqualTo(2);
           }
       }
       
       public class Subtract {
           public int subtract(final List<String> numbers) {
               return Integer.parseInt(numbers.get(0)) - Integer.parseInt(numbers.get(1));
           }
       }
       ```

     - ```java
       public class MultiplyTest {
           @Test
           @DisplayName("곱하기")
           void multiply() {
               Multiply multiply = new Multiply();
               assertThat(multiply.multiply(Arrays.asList("4", "2"))).isEqualTo(8);
           }
       }
       
       public class Multiply {
           public int multiply(final List<String> numbers) {
               return Integer.parseInt(numbers.get(0)) * Integer.parseInt(numbers.get(1));
           }
       }
       ```

     - ```java
       public class DivideTest {
           @Test
           @DisplayName("나누기")
           void divide() {
               Divide divide = new Divide();
               assertThat(divide.divide(Arrays.asList("4", "2"))).isEqualTo(2);
           }
       }
       
       public class Divide {
           public int divide(final List<String> numbers) {
               return Integer.parseInt(numbers.get(0)) / Integer.parseInt(numbers.get(1));
           }
       }
       ```

       <br/>

     - 기능을 최대한 작은 단위로 쪼개서 TDD 로 구현해 나가기 시작하니 굉장히 수월하다.

     - 이제 지금까지 구현한 기능들을 이용해서 사칙연산을 모두 수행 가능한 기능을 TDD 로 구현해보겠다.

       - 먼저 사칙연산 기능들을 통합 관리하기 위해 인터페이스를 이용한다.

       - ```java
         @FunctionalInterface
         public interface CalculateStrategy {
             int calculate(final List<String> numbers);
         }
         
         public class Add implements CalculateStrategy {
             @Override
             public int calculate(final List<String> numbers) {
                 return Integer.parseInt(numbers.get(0)) + Integer.parseInt(numbers.get(1));
             }
         }
         
         public class Subtract implements CalculateStrategy {
             @Override
             public int calculate(final List<String> numbers) {
                 return Integer.parseInt(numbers.get(0)) - Integer.parseInt(numbers.get(1));
             }
         }
         
         public class Multiply implements CalculateStrategy {
             @Override
             public int calculate(final List<String> numbers) {
                 return Integer.parseInt(numbers.get(0)) * Integer.parseInt(numbers.get(1));
             }
         }
         
         public class Divide implements CalculateStrategy {
             @Override
             public int calculate(final List<String> numbers) {
                 return Integer.parseInt(numbers.get(0)) / Integer.parseInt(numbers.get(1));
             }
         }
         ```

         <br/>

       - 당연히 테스트 코드의 메서드들도 전부 calculate() 로 변경해준다.

       - 그 후에 아까 TDD 를 실패했던 사칙연산 계산 기능 메서드를 구현한다.

       - ```java
         public class CalculatorTest {
             @ParameterizedTest
             @DisplayName("두 숫자의 사칙연산 기능")
             @CsvSource(value = {"3 + 2, 5", "3 - 2, 1", "3 * 2, 6", "3 / 2, 1"})
             void calculate(String formula, int result) {
                 Calculator calculator = new Calculator();
                 assertThat(calculator.calculate(formula)).isEqualTo(result);
             }
         }
         
         public class Calculator {
             public static final Map<String, CalculateStrategy> CALCULATOR = new HashMap<>();
             
             static {
                 CALCULATOR.put("+", new Add());
                 CALCULATOR.put("-", new Subtract());
                 CALCULATOR.put("*", new Multiply());
                 CALCULATOR.put("/", new Divide());
             }
             
             public int calculate(final String formula) {
                 final String[] split = formula.split(" ");
                 return CALCULATOR.get(split[1]).calculate(Arrays.asList(split[0], split[2]));
             }
         }
         ```

       - 기능을 최대한 작은 단위로 쪼갠 뒤, In-Out 방식으로 TDD 로 구현하니 굉장히 수월하게 구현이 된 모습을 볼 수 있다.

         <br/>

2. **테스트하기 힘든 코드와 가능한 코드로 분리 설계 후 TDD 진행**

   - **단위 테스트가 힘든 요구사항**

     - 0 ~ 9 까지의 숫자 중 무작위로 뽑아서 4를 초과하면 자동차는 1칸 이동한다.

     - 랜덤의 요소라서 테스트를 어떻게 진행해야 할 지 감이 안잡힌다.

       - 참고로 테스트는 어떨 땐 실패, 어떨 땐 성공이라면 그 테스트는 실패한 테스트이다.
       - 항상 성공을 유지해야 한다.

     - ```java
       public class CarTest {
           @Test
           @DisplayName("무브")
           void move() {
               Car car = new Car(1);
               assertThat(car.move()).isEqualTo(2);
           }
       }
       
       public class Car {
           private final int position;
           
           public Car(final int position) {
               this.position = position;
           }
           
           public int move() {
               if (getRandomNum() > 4) {
                   return position + 1;
               }
               
               return position;
           }
           
           private int getRandomNum() {
               return new Random().nextInt(10);
           }
       }
       ```

       - 리팩토링 할 부분이 많지만, 예제이기 때문에 이 상태로 진행하겠다.

       - **여기서 TDD 가 더이상 진행이 안될 것이다.**

         - **car 가 무브를 할 지, 안할 지 보장이 안되기 때문이다.**

           <br/>

       - **전략 패턴(인터페이스) 를 이용해서 테스트 가능한 부분을 추출해보자.**

         - **if 문의 조건문에 있는 랜덤 특성을 인터페이스로 추출할 것이다.**

         - ```java
           @FunctionalInterface
           public interface MoveStrategy {
               boolean move();
           }
           
           public class RandomMoveStrategy implements MoveStrategy {
               @Override
               public boolean move() {
                   return getRandomNum() > 4;
               }
               
               private int getRandomNum() {
                   return new Random().nextInt(10);
               }
           }
           
           public class Car {
               private final int position;
               
               public Car(final int position) {
                   this.position = position;
               }
               
               public int move(MoveStrategy moveStrategy) {
                   if (moveStrategy.move()) {
                       return position + 1;
                   }
                   
                   return position;
               }
           }
           
           
           public class CarTest {
               @Test
               @DisplayName("무브")
               void move() {
                   Car car = new Car(1);
                   assertThat(car.move(() -> true)).isEqualTo(2);
               }
           }
           
           // 테스트 성공
           ```

         - 실제로 구현 될 프로그램 모습은, Car 의 상위 객체에서 RandomMoveStrategy 클래스를 매개변수로 전달하는 모습일 것이다.

         - 테스트 할 때는 **람다식을 이용해서 이동 혹은 정지를 강제**할 수 있어서, 완벽한 테스트가 가능해진다.

     

   