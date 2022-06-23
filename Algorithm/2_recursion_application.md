
# 목차

1. [재귀호출을 활용한 완전탐색](#1-재귀호출을-활용한-완전탐색) <br/>
2. [재귀 호출없이 풀기 어려운 유형](#2-재귀-호출없이-풀기-어려운-유형) <br/>

<br/>

# 재귀함수 실전 응용

<br/>

# 1. 재귀호출을 활용한 완전탐색

- ## 재귀호출의 문제 유형 3가지

  1. ### 중복 순열

     - 중복된 수가 답이 될 수 있는 것
     - 테스트 케이스의 정답 예 : 1110, 1233

  2. ### 일반 순열

     - 중복된 수가 답이 될 수 없는 것
     - 테스트 케이스의 정답 예 : 1234, 3241

  3. ### 조합

     - 순서가 달라도 구성이 같은 것은 답이 될 수 없는 것

     - 테스트 케이스의 정답 예 : 1234, 3215

       

# 2. 재귀 호출없이 풀기 어려운 유형

- ## n중 for 문을 써야할 때

  - 문제

    - n개의 카드가 있고, 각각은 1부터 n까지의 번호를 갖는다.

      이 때, n개의 카드를 한 줄로 세울 수 있는 모든 경우를 출력하시오.

      (단, 각 카드의 숫자는 겹치면 안된다.)

      (n >= 1 && 9 >= n)

      

  - **n == 3 일 때**

    ```java
    public class Application {
        public static void main(String[] args) {
            int n = 3;
            for(int i = 1; i <= n; i++){
                for(int j = 1; j <= n; j++){
                    for(int k = 1; k <= n; k++){
                        if(i != j && j != k && i != k)
                            System.out.println("" + i + j + k);
                    }
                }
            }
        }
    }
    
    // 출력 결과
    // 123
    // 132
    // 213
    // 231
    // 312
    // 321
    ```

    

  - **n 이 10, 100 또는 1000 이라면 이런식으로 풀 수 없다.**

    - 일일이 중첩 for문을 써주는 것은 불가능하다.

    - 이런 것을 재귀함수로 해결 가능하다.

      

  - **재귀 함수로 들어가기 전에 꼭 인지하고 있어야 할 2가지**

    1. **스코프 개념**

       - **[[Java, C] call by value   VS   call by reference](https://bit.ly/3xUzAHl)**
       - 위 게시글의 1번 스코프 개념글 가볍게 참고

    2. **호출 스택 개념**

       - 호출된 메서드들을 메서드가 종료될 때까지 담고있는 공간이다.

       - 스택은 FILO (First In, Last Out) 개념의 자료구조이다.

         

  - ### n중 for문을 재귀 호출로 풀어보기 (일반 순열)

    ```java
    public class Application {
        int n;
        int[] arr;
        boolean[] check;
    
        public static void main(String[] args) throws IOException {
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            Application m = new Application();
            m.n = Integer.parseInt(br.readLine());
            m.arr = new int[m.n];
            m.check = new boolean[m.n + 1];
            m.recur(0);
        }
    
        // x : 현재 몇 번째 for문인지를 나타낸다.
        void recur(int x){
            if(x == n) {
                for(int i : arr) System.out.print(i);
                System.out.println();
                return;
            }else{
    
                // i : x번째에 어떤 숫자가 들어갈지를 정한다.
                for(int i = 1; i <= n; i++){
                    // check 배열은 번호가 중복이 안되게 체크해준다.
                    if(!check[i]){
                        check[i] = true;
                        arr[x] = i;
                        // (스택의 개념)
                        // recur(x + 1); 가 끝날때까지 recur(x); 는 여기서 계속 기다린다.
                        recur(x + 1); 
                        
                        
                        check[i] = false;
                    }
                }
            }
        }
    }
    
    
    // 입력 : 3
    
    // 출력 결과
    // 123
    // 132
    // 213
    // 231
    // 312
    // 321
    ```

    

    <code><strong>여기서 한 발자국만 더 응용해보자</strong></code>

    

  - ### 순열 구하기 (일반 순열)

    - 문제

      - n개의 알파벳 중에서 r개를 나열할 수 있는 경우를 모두 출력하시오.

        각 알파벳은 한번씩밖에 못쓴다. (즉, 알파벳 중복 불가능)

        ```java
        package study;
        
        import com.sun.tools.javac.Main;
        
        import java.io.BufferedReader;
        import java.io.IOException;
        import java.io.InputStreamReader;
        import java.util.StringTokenizer;
        
        public class Application {
            int n;
            int r;
            boolean[] check;
            char[] ch;
        
            public static void main(String[] args) throws IOException{
                BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
                Application m = new Application();
                StringTokenizer st = new StringTokenizer(br.readLine());
                m.n = Integer.parseInt(st.nextToken());
                m.r = Integer.parseInt(st.nextToken());
                m.check = new boolean[m.n]; // 사용 가능한 알파벳의 갯수만큼 체크해줘야 한다.
                m.ch = new char[m.r]; // 들어갈 수 있는 알파벳의 갯수만큼 넣어줘야 한다.
                m.recur(0);
            }
        
            // n : 몇개의 알파벳을 쓸 수 있는지 알려주는 변수
            // r : 중첩 for문의 갯수가 몇개인지 알려주는 변수
            // x : 현재 몇 번째 for문인지 알려주는 변수
        
        
            void recur(int x){
                if(x == r){
                    System.out.println(ch);
                    return;
                }
        
                // i : x번째에 어떤 알파벳이 들어갈지를 정한다.
                for(int i = 0; i < n; i++){
                    if(!check[i]){
                        ch[x] = (char)('a' + i);
                        check[i] = true;
                        recur(x + 1);
                        check[i] = false;
                    }
                }
            }
        }
        
        // 입력 : 3 2
        
        // 출력 결과
        // ab
        // ac
        // ba
        // bc
        // ca
        // cb
        ```

        

    