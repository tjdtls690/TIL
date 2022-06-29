
# 목차

1. [dessert](#1-dessert) <br/>
2. [inequal](#2-inequal) <br/>

<br/>

# 재귀 함수 응용 문제

# 1. dessert

- ## 문제

  - 농부 존은 소들의 저녁식사 줄 세우는 새로운 방법을 개발 했다. N(1~15)마리의 소들을 순서대로 세 워놓은 후, 각 소들 사이에 +, - , . 셋 중 1가지가 써져있는 냅킨을 배치해서 최종 결과가 0 이 되게 해야 하는 것이다. 점(.)이 써져있는 냅킨을 통해 더 큰 수를 만들 수 있게 된다. 아래와 같은 경우를 보자. (ps .이 써져있는 냅킨은 '공백'이라고 생각하면 된다.)

    <code><strong>1-2.3-4.5+6.7</strong></code>

    이와 같은 배치는 1-23-45+67 을 나타낸다. 결과는 0 이다. 10.11은 1011 로 해석된다.

    

- ## 입력

  - 첫 째 줄에는 소들의 수 N이 입력된다.

    

- ## 출력

  - 처음 20줄에 대해 가능한 20가지 답을 출력하는데, 사전 순으로 앞선 것을 출력한다. 순서는 +가 가장 앞서고 -와 . 이 순서대로 뒤따른다. 답이 20개 미만이면 가능한 답을 각 숫자와 문자 사이에 공백을 두고 출력한다. 모두 출력한다. 마지막 줄에는 가능한 답의 총 가지수를 출력한다.

    

- ## 예제 입력

  - <code><strong>7</strong></code>

    

- ## 예제 출력

  - ```java
    1 + 2 - 3 + 4 - 5 - 6 + 7
    1 + 2 - 3 - 4 + 5 + 6 - 7
    1 - 2 + 3 + 4 - 5 + 6 - 7
    1 - 2 - 3 - 4 - 5 + 6 + 7 
    1 - 2 . 3 + 4 + 5 + 6 + 7 
    1 - 2 . 3 - 4 . 5 + 6 . 7
    6
    ```

    

- ## 내 풀이

  - ```java
    import java.io.*;
    import java.util.*;
    
    public class Main{
      int n;
      int[] symbol;
      char[] ch = {'+', '-', '.'};
      int count;
      
      public static void main(String[] arhs) throws IOException{
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        Main m = new Main();
        m.n = Integer.parseInt(br.readLine());
        m.symbol = new int[m.n - 1];
        
        m.recur(0);
          
        // 전부 출력 끝난 후 마지막으로 누적된 count 출력
        System.out.println(m.count);
      }
      
      void recur(int current){
        // 기저조건 : 연산기호 개수가 (숫자개수 - 1)과 같다면 출력 후 연산기호 추가 중지
        if(current == n - 1){
          // evaluate() : 연산 수행 결과를 반환
          int result = evaluate();
          
          // 연산 수행 결과가 0이면 count++ 후 출력
          if(result == 0){
            count++;
              
            // 카운트가 20을 초과하면 카운트만 올리고 출력은 안하기.
            if(count <= 20){
              print();
            }
          }
          return;
        }
        
        // 연산기호의 모든 경우의 수를 재귀호출로 완전탐색 수행
        // 중복 순열
        for(int i = 0; i < 3; i++){
          symbol[current] = i;
          recur(current + 1);
        }
      }
      
      // 연산 수행 결과를 반환
      int evaluate(){
        int myNum = 1;
        int mySym = 0;
        int sum = 0;
        
        // 연산기호를 나타내는 int형 배열을 통해 계산 수행
        for(int i = 0; i < symbol.length; i++){
            
          // . 이 아니면 연산 수행
          if(symbol[i] == 0 || symbol[i] == 1){
            if(mySym == 0) sum += myNum;
            else sum -= myNum;
            
            myNum = i + 2;
            mySym = symbol[i];
              
            // . 이라면 뒤의 숫자를 현재 숫자와 이어붙이기 수행
          }else{
            if(i + 2 >= 10) myNum *= 100;
            else myNum *= 10;
            
            myNum += i + 2;
          }
        }
        
        // 마지막 연산 수행 안된 것 마저 수행
        if(mySym == 0) sum += myNum;
        else sum -= myNum;
        
        // 연산 결과값 반환
        return sum;
      }
      
      // 조건에 맞는 식일 경우 실시간으로 출력
      void print(){
        for(int i = 0; i < symbol.length; i++){
          System.out.print(i + 1 + " ");
          System.out.print(ch[symbol[i]] + " ");
        }
        System.out.println(n);
      }
    }
    ```

    

# 2. inequal

- ## 문제

  - 두 종류의 부등호 기호 ‘<’와 ‘>’가 k 개 나열된 순서열 A가 있다. 우리는 이 부등호 기호 앞뒤에 서로 다른 한 자릿수 숫자를 넣어서 모든 부등호 관계를 만족시키려고 한다. 예를 들어, 제시된 부등호 순서열 A가 다음과 같다고 하자.

    A ⇒ < < < > < < > < >

    부등호 기호 앞뒤에 넣을 수 있는 숫자는 0부터 9까지의 정수이며 선택된 숫자는 모두 달라야 한다. 아래는 부등호 순서열 A를 만족시키는 한 예이다.

    3 < 4 < 5 < 6 > 1 < 2 < 8 > 7 < 9 > 0

    이 상황에서 부등호 기호를 제거한 뒤, 숫자를 모두 붙이면 하나의 수를 만들 수 있는데 이 수를 주어진 부등호 관계를 만족시키는 정수라고 한다. 그런데 주어진 부등호 관계를 만족하는 정수는 하나 이상 존재한다. 예를 들어 3456128790 뿐만 아니라 5689023174도 아래와 같이 부등호 관계 A를 만족시킨다.

    5 < 6 < 8 < 9 > 0 < 2 < 3 > 1 < 7 > 4

    여러분은 제시된 k 개의 부등호 순서를 만족하는 (k+1) 자리의 정수 중에서 최대값과 최소값을 찾아야 한다. 앞서 설명한 대로 각 부등호의 앞뒤에 들어가는 숫자는 { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 } 중에서 선택해야 하며 선택된 숫자는 모두 달라야 한다. 프로그램의 실행시간은 0.5초를 넘을 수 없다.

    

- ## 입력

  - 첫 줄에 부등호 문자의 개수를 나타내는 정수 가 주어진다. 그 다음 줄에는 k 개의 부등호 기호가 하나의 공백을 두고 한 줄에 모두 제시된다. k 의 범위는 2 <= k <= 9이다.

    

- ## 출력

  - 여러분은 제시된 부등호 관계를 만족하는 자리의 최대, 최소 정수를 첫째 줄과 둘째 줄에 각각 출력해야 한다. 단 아래 예(1)과 같이 첫 자리가 0인 경우도 정수에 포함되어야 한다. 모든 입력에 답은 항상 존재하며 출력 정수는 하나의 문자열이 되도록 해야 한다.

    

- ## 예제 입력 (1)

  - <code><strong>2</strong></code>

    <code><strong>< ></strong></code>

    

- ## 예제 출력 (1)

  - <code><strong>897</strong></code>

    <code><strong>021</strong></code>

    

    <br/>

    

- ## 예제 입력 (2)

  - <code><strong>9</strong></code>

    <code><strong>> < < < > > > < <</strong></code>

    

- ## 예제 출력 (2)

  - <code><strong>9567843012</strong></code>

    <code><strong>1023765489</strong></code>

    

- ## 내 풀이

  - ```java
    import java.io.*;
    import java.util.*;
    
    public class Main{
      int n;
      int[] sign;
      int[] result;
      boolean[] resCheck;
      boolean boo;
      
      public static void main(String[] args) throws IOException{
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        Main m = new Main();
        m.n = Integer.parseInt(br.readLine());
        m.sign = new int[m.n];
        m.result = new int[m.n + 1];
        m.resCheck = new boolean[10];
        
        StringTokenizer st = new StringTokenizer(br.readLine());
        
        // 다루기 쉽게 < 는 0, > 는 1 로 치환한다.
        for(int i = 0; i < m.sign.length; i++){
          if(st.nextToken().equals("<")){
            m.sign[i] = 0;
          }else{
            m.sign[i] = 1;
          }
        }
        
        // 조건에 해당되는 수들 중 최대값 출력
        m.recur2(0);
          
        // 일반 순열의 조건을 갖추기 위한 boolean 배열을 다시 초기화
        // 한 번 더 써야하기 때문에.
        Arrays.fill(m.resCheck, false);
          
        // 조건에 해당되는 수들 중 최소값 출력
        m.recur1(0);
      }
      
      // 조건에 해당되는 수들 중 최대값 출력
      void recur1(int current){
        // 조건에 해당되어 한 번 출력 후 재귀를 아예 빠져나오기.
        if(!boo) return;
        
        // 기저조건 : 숫자의 인덱스가 (입력된 부등호의 개수 + 1) 보다 같으면 기저조건에 해당 됨.
        if(current >= n + 1){
          // 나열되어있는 숫자를 차례대로 출력
          for(int i = 0; i < result.length; i++){
            System.out.print(result[i]);
          }
          System.out.println();
            
          // 재귀함수 완전히 빠져나오기
          boo = false;
          return;
        }
        
        // 1 ~ 9 까지의 숫자를 재귀호출을 통해 완전탐색
        // 일반 순열
        for(int i = 0; i < 10; i++){
          // 일반 순열이기에 이미 들어가있는 숫자는 건너뛰기
          if(!resCheck[i]){
            // 인덱스 0 차례가 아닐 때
            if(current != 0){
              // 부등호에 따른 숫자의 조건이 맞지 않는 것들 거르기
              if(sign[current - 1] == 1 && result[current - 1] <= i){
                continue;
              }else if(sign[current - 1] == 0 && result[current - 1] >= i){
                continue;
              }
            }
              
            // 이미 들어간 숫자를 거르기 위해 true 로 체크
            resCheck[i] = true;
            // 숫자 집어넣기
            result[current] = i;
            // 다음 인덱스에 숫자를 넣기 위해 재귀호출 수행
            recur1(current + 1);
            // 다음 인덱스의 재귀함수가 끝났기에 들어갔던 숫자를 다시 빼준다는 의미의 체크
            resCheck[i] = false;
          }
        }
      }
      
      // 조건에 해당되는 수들 중 최소값 출력
      // 위의 메서드와 for문(숫자)이 거꾸로 진행한다는 것, 재귀함수를 완전 빠져나오는 boolean값이 반대라는 것 외에 모두 동일
      void recur2(int current){
        if(boo) return;
        
        if(current >= n + 1){
          for(int i = 0; i < result.length; i++){
            System.out.print(result[i]);
          }
          System.out.println();
          boo = true;
          return;
        }
        
        for(int i = 9; i >= 0; i--){
          if(!resCheck[i]){
            if(current != 0){
              if(sign[current - 1] == 1 && result[current - 1] <= i){
                continue;
              }else if(sign[current -1] == 0 && result[current - 1] >= i){
                continue;
              }
            }
            
            resCheck[i] = true;
            result[current] = i;
            recur2(current + 1);
            resCheck[i] = false;
          }
        }
      }
    }
    ```

    

