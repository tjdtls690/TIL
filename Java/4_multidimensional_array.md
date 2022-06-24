# 목차

1. [2차원 배열 선언 및 접근 방법](#1-2차원-배열-선언-및-접근-방법) <br/>
2. [2차원 배열을 초기화 하는 방법](#2-2차원-배열을-초기화-하는-방법) <br/>
3. [특정 데이터로 전부 초기화 하는 방법 2가지](#3-특정-데이터로-전부-초기화-하는-방법-2가지) <br/>
4. [가변 배열](#4-가변-배열) <br/>

<br/>

# [Java, 자바] 다차원 배열

<br/>

# 1. 2차원 배열 선언 및 접근 방법

- 2차원 배열 선언 방법 3가지

  - ```java
    1. 타입 변수이름[][] => int arr1[][];
    2. 타입[] 변수이름[] => int[] arr2[];
    3. 타입[][] 변수이름; => int[][] arr;
    ```

    

- 2차원 배열의 형태는 테이블 형태와 많이 닮아있다.

  - 물론 메모리상에선 쭉 이어져 붙어있는 것과 같다.

  ```java
  int[][] arr = new int[100][200]; 라고 했을 때
  100행 200열의 테이블이 만들어진 것이라 보면 된다.
  
  - 여기서 1차원 올려서 3차원 배열로 가게되면 입체적인 사각형 형태의 계층구조의 테이터 형태를 만들 수 있겠다는 생각이 든다.

  - 회사마다 상황이 좀 다를 순 있겠지만, 전해 듣기론 실무에서 보통 3차원 배열은 쓸 일이 별로 없다고 한다. (주로 2차원 배열까지 많이 쓴다.)
  ```

  

- 2차원 배열의 접근방법 (메모리 참조 형태)

  - ```java
    int[][] arr = new int[5][4];
    
    // 위와 같이 2차원 배열이 만들어 졌을 때 메모리 상에서의 관계
    // 첫 배열을 담당하는 5개의 행이 각각의 열에 있는 4열짜리 배열의 첫 방의 주소를 참조한다.
      arr[]		=> 					arr[][]
    [arr[0]]		=> [arr[0][0]] [arr[0][1]] [arr[0][2]] [arr[0][3]]
    [arr[1]]		=> [arr[1][0]] [arr[1][1]] [arr[1][2]] [arr[1][3]]
    [arr[2]]		=> [arr[2][0]] [arr[2][1]] [arr[2][2]] [arr[2][3]]
    [arr[3]]		=> [arr[3][0]] [arr[3][1]] [arr[3][2]] [arr[3][3]]
    [arr[4]]		=> [arr[4][0]] [arr[4][1]] [arr[4][2]] [arr[4][3]]
    
    // 이런 경우
    // arr.length -> 5
    // arr[0].length => 4
    ```

  - ```java
    public class Application {
        public static void main(String[] args) {
            int[][] arr = new int[5][4];
            System.out.println(arr.length);
            System.out.println(arr[0].length);
        }
    }
    
    // 출력 결과
    // 5
    // 4
    ```

    

    

    

# 2. 2차원 배열을 초기화 하는 방법

- ```java
  1. int[][] arr = new int[][]{{3, 3, 9}, {4, 4, 16}, {10, 20, 300}}; 
  
  // new int[][] 생략 가능
  // (물론 선언과 생성을 동시에 할 때만)
  2. int[][] arr = {{3, 3, 9}, {4, 4, 16}, {10, 20, 300}}; 
  ```

  

- 보통 이해하기 쉽게 2차원 배열을 초기화 할 땐 이런식으로 초기화 한다.

  - ```java
    int[][] arr = {
    	{3, 3, 9},
    	{4, 4, 16},
        {10, 20, 300}
    };
    ```

    

# 3. 특정 데이터로 전부 초기화 하는 방법 2가지

- **2중 for 문**

  - ```java
    public class Application {
        public static void main(String[] args) {
            int[][] arr = new int[5][4];
            
            for (int i = 0; i < arr.length; i++) {
                for (int j = 0; j < arr[i].length; j++) {
                    arr[i][j] = 35;
                }
            }
    
            for (int[] ints : arr) {
                for (int anInt : ints) {
                    System.out.print(anInt + " ");
                }
                System.out.println();
            }
        }
    }
    
    // 출력 결과
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    ```

  

- **Arrays.fill()** 

  - ```java
    public class Application {
        public static void main(String[] args) {
            int[][] arr = new int[5][4];
            
            for (int[] value : arr) {
                Arrays.fill(value, 35);
            }
    
            for (int[] ints : arr) {
                for (int anInt : ints) {
                    System.out.print(anInt + " ");
                }
                System.out.println();
            }
        }
    }
    
    // 출력 결과
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    // 35 35 35 35 
    ```

    

# 4. 가변 배열

- **2차원 배열에선 각 행마다 열의 갯수를 다르게 생성할 수 있다.**

  - ```java
    public class Application {
        public static void main(String[] args) {
            int[][] arr = new int[5][];
            arr[0] = new int[3];
            arr[1] = new int[1];
            arr[2] = new int[2];
            arr[3] = new int[1];
            arr[4] = new int[4];
            for (int[] ints : arr) {
                System.out.println(ints.length);
            }
        }
    }
    
    // 출력 결과
    // 3
    // 1
    // 2
    // 1
    // 4
    ```

    

- **위 예제의 2차원 가변배열의 메모리상 형태를 보면**

  - ```java
    // arr[]		=> 					arr[][]
    // [arr[0]]		=> [arr[0][0]] [arr[0][1]] [arr[0][2]]
    // [arr[1]]		=> [arr[1][0]]
    // [arr[2]]		=> [arr[2][0]] [arr[2][1]]
    // [arr[3]]		=> [arr[3][0]]
    // [arr[4]]		=> [arr[4][0]] [arr[4][1]] [arr[4][2]] [arr[4][3]]
    ```

  

- **참고로 가변배열도 이해하기 쉬운 형태로 초기화가 가능하다.**

  - ```java
    public class Application {
        public static void main(String[] args) {
            int[][] arr = {
                    {3, 3, 9},
                    {20},
                    {9, 81},
                    {21},
                    {15, 20, 25, 30}
            };
    
            for (int[] ints : arr) {
                for (int anInt : ints) {
                    System.out.print(anInt + " ");
                }
                System.out.println();
            }
        }
    }
    
    // 출력 결과
    // 3 3 9 
    // 20 
    // 9 81 
    // 21 
    // 15 20 25 30 
    ```
