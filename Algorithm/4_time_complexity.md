
# 목차

1. [시간 복잡도란??](#1-시간-복잡도란) <br/>
2. [왜 알아야 하는가??](#2-왜-알아야-하는가) <br/>
3. [시간 복잡도 계산법 Big-O](#3-시간-복잡도-계산법-big-o) <br/>

<br/>

# [알고리즘, Algorithm] 시간 복잡도 (Time Complexity)

<br/>

> **시간 복잡도는 내가 처음 알고리즘을 배울 때 접했던 개념이다.**
>
> **초중반에 난이도가 그리 어렵지 않은 알고리즘 문제들은 효율성을 굳이 따지지 않아도 되기에 크게 신경을 안썼다.**
>
> **근데 요즘 난이도를 점점 더 높여서 알고리즘 문제를 풀다보니, 효율성 테스트까지 통과를 해야하는 상황에 많이 직면하게 되었다.**
>
> **그래서 내가 짠 코드가 얼마나 빨리 돌아가는지까지 신경을 쓰게 되었다.**
>
> **이 녀석의 중요성을 다시 한 번 체감하게 되어 정리해보고자 한다.**

<br/>

## 1. 시간 복잡도란??

- **컴퓨터 프로그램의 입력값과 연산 수행 시간의 상관관계를 나타내는 척도이다.**

- 프로그램이 <code><strong>대략적으로</strong></code> 몇 개의 명령을 수행하는지 나타내는 것이라고도 보면 된다.

  - 왜 대략적이라고 하는지는 밑에서 Big-O 표기법을 정리할 때 알 수 있다.

  - <code><strong>시간복잡도 계산시, 반복문이 가장 큰 영향을 끼친다.</strong></code>
  
    

## 2. 왜 알아야 하는가??

1. **문제를 효율적으로 해결할 수 있기 때문이다.**

   	- 똑같은 문제를 해결해도 빠르게 해결하는 것이 중요하다.
   	- 즉, <code><strong>성능의 문제</strong></code>와 관련된 중요한 부분이다.

2. **내 프로그램은 얼마나 빠른지 유추할 수 있기 때문이다.**

   - 내 프로그램을 굳이 돌려보지 않고 어느정도로 빠를지 예측하고 싶을 때 유용하다.

      

## 3. 시간 복잡도 계산법 Big-O

1. ### O(상수) == O(1)

   - ```java
     class Scratch {
         public static void main(String[] args) {
             int a;
             int b = 3;
             int c = 2;
             
             a = b + c;
         
             System.out.println(a);
         }
     }
     ```

     - 이 예제는 **대략 5번의 연산을 수행**한다고 판단하고 **O(5)** 라고 부른다.

     - 그러나 컴퓨터의 연산은 굉장히 빠르기 때문에 O(5) 나 O(1) 이나 차이가 없다. 

     - 즉, <code><strong>O(상수) 는 전부 O(1) 로 봐야한다는 것이다.</strong></code>

       

2. ### O(n + 상수) == O(n)

   - ```java
     class Scratch {
         public static void main(String[] args) throws IOException {
             BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
             int n = Integer.parseInt(br.readLine());
             int sum = 0;
             
             for (int i = 0; i < n; i++) {
                 sum += i;
             }
             
             System.out.println(sum);
         }
     }
     ```

     - 이 예제는 **대략 n + 4 번 연산을 수행**한다고 판단했을 때, **O(n + 4)** 라 부른다.

     - 그러나 n + 4 에서 어떤 것이 연산 횟수에 가장 영향을 끼치는지 판단할 때, **n 이 가장 영향이 크다고 보게 된다.**

       - 이유는 n 은 사용자로부터 어떤 숫자를 입력받을지 전혀 모르는 상황이다.
       - 결국 n 에 굉장히 큰 수가 올 가능성도 있다는 것이다.

     - 즉, <code><strong>이 예제는 상수를 제외하고 O(n) 으로 봐야한다는 것이다.</strong></code>

       

3. ### O(n제곱 + n) == O(n제곱)

   - ```java
     class Scratch {
         public static void main(String[] args) throws IOException {
             BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
             int n = Integer.parseInt(br.readLine());
             int sum = 0;
             
             for (int i = 0; i < n; i++) {
                 for (int j = 0; j < n; j++) {
                     sum += i * j;
                 }
             }
             
             for (int i = 0; i < n; i++) {
                 sum += i;
             }
             
             System.out.println(sum);
         }
     }
     ```

     - 이 예제는 **대략 n제곱 + n 번 연산을 수행**한다고 판단했을 때, **O(n제곱 + n)** 라 부른다.

     - 그러나 n제곱 + n 에서 어떤 것이 연산 횟수에 가장 영향을 끼치는지 판단할 때, **n제곱 이 가장 영향이 크다고 보게 된다.**

       - 이유는 n제곱에 비하면 n 은 굉장히 작은 숫자에 불과하기 때문이다.

     - 즉, <code><strong>이 예제는 n을 제외하고 O(n제곱) 으로 봐야한다는 것이다.</strong></code>

       

4. ### O(n(n + 1) / 2) == O(n제곱)