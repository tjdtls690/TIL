# 목차

1. [재귀함수란??](#1-재귀함수란) <br/>
2. [두가지 계산 방법](#2-두가지-계산-방법) <br/>
3. [귀납적 계산법 내의 가정 관계](#3-귀납적-계산법-내의-가정-관계) <br/>
4. [수학적 귀납법 증명](#4-수학적-귀납법-증명) <br/>
5. [재귀함수 구현 3가지 절차](#5-재귀함수-구현-3가지-절차) <br/>
6. [재귀함수 사용 시 중요하다 느낀 2가지](#6-재귀함수-사용-시-중요하다-느낀-2가지) <br/>
7. [재귀함수 필요 여부를 구별하기 좋은 방법](#7-재귀함수-필요-여부를-구별하기-좋은-방법) <br/>

<br/>


# 재귀함수

# 1. 재귀함수란??

- <code><strong>자기 자신을 부르는 함수</strong></code>이다.

  - 즉, 귀납적 계산 방법이다.
  
  

# 2. 두가지 계산 방법

- ## 순차적 계산법

  - 말 그대로 순차적으로 계산하는 방법이다.
  
    1. A를 계산한다.
    2. A의 결과를 이용해서 B를 계산한다.
    3. B의 결과를 이용해서 C를 계산함으로써 원하는 결과를 얻는다.
  
    ```java
    public class Main {
        public static void main(String[] args) {
            int a = 10;
            int b = 15;
            // A를 계산한다.
            int sum = add(a, b); 
    
            int c = 20;
            // A의 결과를 이용해서 B를 계산한다.
            sum = subtract(sum, c); 
    
            int d = 100;
            // B의 결과를 이용해서 C를 계산함으로써 원하는 결과를 얻는다.
            System.out.println(multiply(sum, d)); 
        }
    
        public static int add(int a, int b){
            return a + b;
        }
    
        public static int subtract(int sum, int c){
            return sum - c;
        }
    
        public static int multiply(int sum, int d){
            return sum * d;
        }
    }
    
    // 출력 결과
    // 500
    ```
  
    
  
  - <code><strong>지금까지 우리가 해온 계산법은 전부 순차적 계산법이라고 보면 된다.</strong></code>



- ## 귀납적 계산법

  - 구하려는 값을 f(x) 라고 할 때, f(x) 를 구하기 위해 또 f(x) 를 활용하는 방법을 말한다.

    - <code><strong>f(n) = n * f(n-1)</strong></code> 은 <code><strong>n! = n * (n - 1)!</strong></code> 로 나타낼 수 있다.

      - ```java
        public class Main {
            public static void main(String[] args) {
                System.out.println(factorial(5));
            }
        
            public static int factorial(int n){
                // 기저조건 => 0! 은 1인 것으로 약속한다.
                if(n == 0){
                    return 1;
                }
        
                // factorial(n) 을 구하기 위해 자기자신인 factorial(n - 1) 을 이용한다.
                return n * factorial(n - 1);
            }
        }
        
        // 출력 결과
        // 120
        ```

        

# 3. 귀납적 계산법 내의 가정 관계

- 위의 귀납적 계산법 예제로 재귀함수가 어떻게 도는지 깊이 이해해보자.

  1. <code><strong>무수한 가정이 세워지면서 기저조건을 향해 계속 돌게 된다.</strong></code>

     - factorial(5) 가 factorial(4) 에게 "너가 factorial(4) 의 답을 반환한다는 것을 보장만 해줘! 그럼 내가 5만 한번 더 곱해서 반환할게!"
     - factorial(3) 도, factorial(2) 도 마찬가지로 자기가 호출한 factorial(n - 1) 이 정확한 값을 반환할거라 가정합니다.
     - <code><strong>그럼 대체 언제까지 가정만 세울거야?? 무한루프 돌거야??</strong></code>

  2. <code><strong>기저조건을 만나는 순간 지금까지 세워놓은 가정이 실질적으로 성립하게 된다.</strong></code>

     - 끝까지 내려가면, 결국 factorial(1) 이 factorial(0) 에게 1을 반환받는다는 것을 보장받게 된다.

     - 그러므로 factorial(1) 이 정확한 값을 반환한다는 가정이 실제로 성립하게 된다.

     - 실제로 factorial(1) 이 정확한 값을 반환한다는 가정이 성립하니 factorial(2) 도 정확한 값을 반환하게 된다.

     - 이렇게 연쇄적으로 factorial(2), factorial(3), factorial(4), factorial(5) 까지 전부 제대로 값을 반환하게 된다.

       

- 즉, <code><strong>수많은 가정을 하다가, 맨 끝(기저조건)에는 정확한 값이 있기 때문에 가능하다.</strong></code>

  - 메서드 안에서 어떤 메서드가 호출되면, 호출스택에 새로 호출된 메서드가 맨 위로 올라오게 되고, 

  - 그 밑에 메서드들은 새로 호출된 메서드가 종료될 때까지 기다리게 된다는 것을 인지해야 한다.

    

# 4. 수학적 귀납법 증명

- <code><strong>명제 P(n)이 모든 자연수 n에 대하여 성립함을 보이자</strong></code>

- 증명 순서

  1. P(1) 이 참임을 보인다.

  2. P(k) 가 성립한다고 가정한 후, P(k + 1) 이 성립함을 보인다.

  3. 따라서 모든 자연수 n 에 대하여 P(n) 이 성립한다.

     

- **등차수열의 합 공식으로 증명해보자**

  1. 1 + 2 + ... + n = n(n + 1) / 2

  2. 1 + 2 + ... + k = k(k + 1) / 2

     - **여기서 k + 1 까지 계산해도 성립하는가??**

  3. 1 + 2 + ... + k + (k + 1) = k(k + 1) / 2 + (k + 1)

     - <code><strong>이 식을 위의 2번 식과 같은 모양으로 풀어나가보자.</strong></code>
     - **여기서 우변의 분모를 2로 통일하기 위해 (k + 1) 을 2 로 나누면,**

  4. 1 + 2 + ... + k + (k + 1) = k제곱 + k + (2k + 2) / 2

     - **정리하면**

  5. 1 + 2 + ... + k + (k + 1) = k제곱 + 3k + 2 / 2

     - **우변의 k제곱 + 3k + 2 를 인수분해 하면**

  6. 1 + 2 + ... + k + (k + 1) = (k + 1)(k + 2) / 2

     - <code><strong>드디어 2번 식과 같은 모양이 만들어졌다.</strong></code>

       

- 위의 증명으로 **P(k) 가 성립할 때, P(k + 1) 이 성립**한다는 것이 증명 되었다.

  - 즉, 계속 자연수를 1씩 타고 올라가면서 결국, <code><strong>모든 자연수가 성립</strong></code>한다는 것을 알 수 있다.

    

# 5. 재귀함수 구현 3가지 절차

1. **함수의 역할을 말로 정확하게 정의해야 한다.**

   - ex) 이 메서드는 매개변수 n의 팩토리얼 즉, n! 을 구하는 메서드다.

2. **기저조건에서 함수가 제대로 동작해야 한다.**

   - ex) 팩토리얼을 구현할 때, 매개변수 n이 0일 때, 1을 제대로 반환해줘야 한다.

3. **함수가 제대로 동작한다고 가정하고 함수를 완성한다.**

   

# 6. 재귀함수 사용 시, 중요하다 느낀 2가지

1. **반드시 방향은 기저조건을 향해야 한다.**

   - 만약 방향이 기저조건을 향해 가지 않는다면,
   - 걸리는 조건식이 없기에 무한루프를 돌며 계속 호출스택에 쌓이다가 스택오버플로우가 뜬다.

2. **반드시 기저조건은 정확한 값을 내놓아야 한다.**

   - 기저조건의 조건이나 반환값이 틀리게 되면, 모든 가정이 무너지면서 오답을 반환받게 된다.

     

# 7. 재귀함수 필요 여부를 구별하기 좋은 방법

- #### 자기보다 작은 또는 큰 자신을 이용해서 자신을 구할 수 있는 상황인지를 분석해보는 방법

  - 팩토리얼 문제

    ```java
    public class Main {
        public static void main(String[] args) {
            System.out.println(factorial(5));
        }
    
        public static int factorial(int n){
            if(n == 0){
                return 1;
            }
    
            // factorial(n) 을 구하기 위해 자기보다 작은 자기자신인 factorial(n - 1) 을 이용한다.
            return n * factorial(n - 1);
        }
    }
    
    // 출력 결과
    // 120
    ```

    

  - N to M 문제

    ```java
    // N 부터 M 까지 1씩 증가하면서 자연수를 전부 더한 값을 출력하는 문제
    // 3 + 4 + 5 + 6 + 7 + 8 => 33
    public class Main {
        public static void main(String[] args) {
            int n = 3;
            int m = 8;
            System.out.println(NtoM(n, m));
        }
    
        static int NtoM(int n, int m){
            if(n == m){
                return m;
            }
            // NtoM(n, m) 을 구하기 위해 자신보다 큰 자신인 NtoM(n + 1, m) 을 이용한다.
            return n + NtoM(n + 1, m);
        }
    }
    
    // 출력 결과
    // 33
    ```

    

