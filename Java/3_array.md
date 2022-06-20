# 목차

1. [배열은 왜 필요할까??](#1-배열은-왜-필요할까) <br/>
2. [배열의 개념](#2-배열의-개념) <br/>
3. [참조형 배열](#3-참조형-배열) <br/>

<br/>

# [Java, 자바] 배열

* toc
{:toc .large-only}

> **배열은 프로그램을 짤 때도, 알고리즘을 풀 때도 굉장히 자주 사용하는 녀석이다.**
>
> **이 녀석을 얼마나 잘 다루느냐에 따라, 코드의 질이 완전히 달라지는 경우도 많다.**
>
> **한번 깊이 파보자.**

<br/>

# 1. 배열은 왜 필요할까??

- #### 방대한 양의 데이터를 손쉽게 다루기 위해 사용된다.

  - 배열이 없다면, 데이터가 추가될 때마다 변수를 계속해서 늘려줘야 할 것이다.

<br/>

# 2. 배열의 개념

- 배열은 <code><strong>'동일한 타입'</strong></code> 의 여러 데이터를 **하나로 묶어서 다루는 것**이다.

  <br/>

- ## 배열이 메모리 공간에서 저장되는 형태는??

  1. **배열 선언이 이루어 졌을 때 : ex) int[] arr;**

       - 생성된 배열을 다루기 위한 참조변수의 공간이 만들어진다.

         

  2. **배열 생성을 할 때 : ex) = new int[3];**

     - 메모리 공간에서 3개의 공간이 연속적인 저장공간으로 할당되어 생성이 된다.

     - 참조변수 arr 은, arr[0] (배열의 첫번째 인덱스) 의 주소를 가리키게 된다.

       1. arr[0], arr[1], arr[2] 의 메모리 공간을 한 덩어리로 묶어서 할당해준다.
       2. 'arr = arr[0] 의 주소' 대입

       ```java
       int[] arr = new int[]{5, 8, 6, 2, 11, 5, -1, -3};
       System.out.println(arr);
       
       // 출력 결과
       // [I@10f87f48
       
       // '타입@배열의 내부주소(16진수)' 형태의 주소가 찍히는 걸 볼 수 있다.
       ```

       

     - **참고로 char 배열은 바로 출력해도 주소가 아닌, 모든 요소가 붙어서 출력되는 형식이다.**

       - 자바의 String 클래스가 char[] 배열을 업그레이드 시킨 기능이란 것을 인지하자.

       ```java
       char[] ch = {'a', 'b', 'c', 'd', 'e', 'f', 'g'};
       System.out.println(ch);
       
       // 출력 결과
       // abcdefg
       ```

       

- ## 배열의 선언과 생성

  - #### 배열의 선언 2가지 방법

    1. int[] arr;

    2. int arr[];

       

  - #### 배열의 생성 3가지 방법

    1. 타입[] 변수이름 = new 타입 [길이];

       ```java
       int[] arr = new int[15];
       ```

       

    2. 타입[] 변수이름 = {요소1, 요소2, ...};

       ```java
       int[] arr = {1, 3, 5, 7};
       ```

       

    3. 타입[] 변수이름 = new 타입[]{요소1, 요소2, ...};

       ```java
       int[] arr = new int[]{1, 3, 5};
       ```

       

- ## 배열의 각 요소에 접근하는 법

  - #### '배열이름[인덱스]' 형식으로 접근한다.

    - **ex) arr[1] = 2680;**

      - 배열의 2번째 요소에 2680을 넣음. (int형 변수를 이용해서 넣는 것도 가능)

    - **ex) int val = arr[1];**

      - 거꾸로 arr[1] 에 저장되어있던 2680 을 변수 val 에 저장하는 것도 가능하다.

        

- ## java.util.Arrays 클래스의 유용한 메서드

  - **Arrays.fill(int[] arr, int n); => arr 배열에 n이라는 요소로 모든 요소를 초기화**

    ```java
    int[] arr = {1, 3, 5};
    Arrays.fill(arr, 4);
    for(int i : arr){
        System.out.print(i + " ");
    }
    
    // 출력 결과
    // 4 4 4
    ```

    

  - **Arrays.toString(int[] arr); => 배열의 모든 요소를 보여주는 함수**

    ```java
    int[] arr = {1, 3, 5};
    System.out.println(Arrays.toString(arr));
    
    // 출력 결과
    // [1, 3, 5]
    ```

    

  - **Arrays.sort(int[] arr); => 배열의 모든 요소를 정렬해주는 함수**

    ```java
    int[] arr = {5, 8, 6, 2, 11, 5, -1, -3};
    Arrays.sort(arr);
    System.out.println(Arrays.toString(arr));
    
    // 출력 결과
    // [-3, -1, 2, 5, 5, 6, 8, 11]
    ```

    

- ## 배열을 복사하는 법

  - 배열 복사 기능은 보통 배열의 길이를 늘리고 싶을 때 사용한다.

    1. #### 반복문을 이용한 복사 (가장 일반적인 복사)

       ```java
       import java.util.Arrays;
       
       public class ArraysPractice {
           public static void main(String[] args) {
               // 1단계 : 길이를 늘리기 위해 두배정도의 길이를 가진 배열 생성
               int[] before = {1, 2, 3};
               int[] copy = new int[before.length * 2];
       
               // 2단계 : 반복문을 통해 데이터를 복사한다.
               for(int i = 0; i < before.length; i++){
                   copy[i] = before[i];
               }
               System.out.println("before : " + Arrays.toString(before));
               System.out.println("copy : " + Arrays.toString(copy));
       
               // 3단계 : 복사받은 새로운 배열의 주소를 before 참조변수가 가리킨다.
               // 이렇게 before 는 같은 데이터를 가지면서 길이가 2배로 늘어나게 된다.
               before = copy;
               System.out.println("before : " + Arrays.toString(before));
       
               // 결과적으로 before 와 copy 는 같은 배열이 된다.
           }
       }
       
       // 출력 결과
       // before : [1, 2, 3]
       // copy : [1, 2, 3, 0, 0, 0]
       // before : [1, 2, 3, 0, 0, 0]
       ```

       

    2. #### System.arraycopy() 메서드를 이용한 복사

       - 위 예제와 차이점은 for문이 System.arraycopy() 로 바뀐 것 뿐이다.

       ```java
       import java.util.Arrays;
       
       public class ArraysPractice {
           public static void main(String[] args) {
               // 1단계 : 길이를 늘리기 위해 두배정도의 길이를 가진 배열 생성
               int[] before = {1, 2, 3};
               int[] copy = new int[before.length * 2];
       
               // 2단계 : 반복문을 통해 데이터를 복사한다.
               System.arraycopy(before, 0, copy, 0, before.length);
               System.out.println("before : " + Arrays.toString(before));
               System.out.println("copy : " + Arrays.toString(copy));
       
               // 3단계 : 복사받은 새로운 배열의 주소를 before 참조변수가 가리킨다.
               before = copy;
               System.out.println("before : " + Arrays.toString(before));
       
               // 결과적으로 before 와 copy 는 같은 배열이 된다.
           }
       }
       
       // 출력 결과
       // before : [1, 2, 3]
       // copy : [1, 2, 3, 0, 0, 0]
       // before : [1, 2, 3, 0, 0, 0]
       ```

       - ##### System.arraycopy() 5개 인자의 역할

         - 첫번째 인자 : 기존의 배열

         - 두번째 인자 : 기존 배열의 복사를 시작할 인덱스 값

         - 세번째 인자 : 새로운 배열

         - 네번째 인자 : 새로운 배열의 복사를 받기 시작할 인덱스 값

         - 다섯번째 인자 : 기존 배열의 시작 인덱스부터 복사를 받을 길이

           


# 3. 참조형 배열

- int[], boolean[], char[] 같은 기본형 타입 배열을 제외한 <code><strong>모든 (래퍼) 클래스 타입 배열</strong></code>을 말한다.

- ## 초기화 값

  - **기본형 타입 배열의 초기화 값은 저마다 다르다.**

    - boolean : false
    - char : '\u0000'
    - byte, short, int : 0
    - long : 0L
    - float : 0.0f
    - double : 0.0d 또는 0.0

  - **그러나 클래스 타입 배열의 초기화 값은 모두 null 이다.**

    ```java
    import java.util.Arrays;
    
    public class ArraysPractice {
        public static void main(String[] args) {
            int[] arr0 = new int[5];
            System.out.println(Arrays.toString(arr0));
    
            Integer[] arr1 = new Integer[5];
            System.out.println(Arrays.toString(arr1));
    
            ArraysPractice[] arr2 = new ArraysPractice[5];
            System.out.println(Arrays.toString(arr2));
        }
    }
    
    // 출력 결과
    // [0, 0, 0, 0, 0]
    // [null, null, null, null, null]
    // [null, null, null, null, null]
    ```

    

- ## 기본형 타입과 클래스 타입 배열의 요소

  - **기본형 타입** 배열은 <code><strong>데이터 그 자체</strong></code>를 저장한다.

  - **클래스형 타입** 배열은 해당 클래스가 저장된 <code><strong>메모리 주소값</strong></code>을 저장한다.

    ```java
    import java.util.Arrays;
    
    public class ArraysPractice {
        public static void main(String[] args) {
            int[] arr1 = new int[2];
            ArraysPractice[] arr2 = new ArraysPractice[2];
    
            arr1[0] = 3;
            arr1[1] = 5;
    
            arr2[0] = new ArraysPractice();
            arr2[1] = new ArraysPractice();
            
            System.out.println(Arrays.toString(arr1));
            System.out.println(Arrays.toString(arr2));
        }
    }
    
    // 출력 결과
    // [3, 5]
    // [study.chap05.ArraysPractice@4eec7777, study.chap05.ArraysPractice@3b07d329]
    ```

    

    

- ## String 클래스 배열

  - 클래스 타입 배열 중 **String 클래스 배열**은 <code><strong>조금 다른 방식으로 초기화</strong></code>를 할 수 있다.

  - #### 보통 클래스 타입 배열에 값을 넣는 법

    ```java
    import java.util.Arrays;
    
    public class ArraysPractice {
        public static void main(String[] args) {
            ArraysPractice[] arr1 = new ArraysPractice[5];
            arr1[0] = new ArraysPractice();
            System.out.println(Arrays.toString(arr1));
        }
    }
    
    // 출력 결과
    // [study.chap05.ArraysPractice@4eec7777, null, null, null, null]
    // study.chap05.ArraysPractice@3b07d329
    
    // 출력 결과를 보면 클래스의 toString() 이 나온 것을 볼 수 있다.
    // 그러므로, 클래스가 요소에 잘 저장이 되었단 뜻이다.
    ```

    

  - #### String 클래스 타입 배열에 값을 넣는 법

    ```java
    import java.util.Arrays;
    
    public class ArraysPractice {
        public static void main(String[] args) {
            String[] arr1 = new String[5];
            arr1[0] = new String("!!!!!!!!");
            arr1[1] = "????????";
            System.out.println(Arrays.toString(arr1));
        }
    }
    
    // 출력 결과
    // [!!!!!!!!, ????????, null, null, null]
    
    // 값을 넣는 방법이 꼭 new 연산자를 쓰지 않더라도, 
    // 쌍따옴표("")를 이용해서 바로 값을 넣는 것이 가능하다.
    ```

    - 이런 방식의 초기화가 가능한 이유는 **String 클래스의 본질**은 결국 <code><strong>char[] 배열</strong></code>이기 때문이라 생각한다.

      ```java
      public final class String
          implements java.io.Serializable, Comparable<String>, CharSequence,
                     Constable, ConstantDesc {
                         
      	public String(char value[]) {
              this(value, 0, value.length, null);
          }
                         
      	String(char[] value, int off, int len, Void sig) {
              if (len == 0) {
                  this.value = "".value;
                  this.coder = "".coder;
                  return;
              }
              if (COMPACT_STRINGS) {
                  byte[] val = StringUTF16.compress(value, off, len);
                  if (val != null) {
                      this.value = val;
                      this.coder = LATIN1;
                      return;
                  }
              }
              this.coder = UTF16;
              this.value = StringUTF16.toBytes(value, off, len);
          }
                         
      	public String(char value[], int offset, int count) {
              this(value, offset, count, rangeCheck(value, offset, count));
          }
      
          private static Void rangeCheck(char[] value, int offset, int count) {
              checkBoundsOffCount(offset, count, value.length);
              return null;
          }
                         
          // 실제 String 클래스의 수많은 메서드들 중 몇개만 갖고 온 것이다.
      	// String 생성자들을 보면 결국 char[] 배열을 통해 처리하는 모습을 볼 수 있다.
                         
      	// 위에서도 언급했지만, char[] 배열은 출력문에 참조변수를 그냥 써도 
      	// 문자들이 알아서 붙어서 출력 된다.
      }
      ```

  - #### String 객체는 불변 객체이다.

    ```java
    public class ArraysPractice {
        public static void main(String[] args) {
            String str1 = "aaa";
            System.out.println(str1);
            str1 += "bbb";
            System.out.println(str1);
        }
    }
    
    // 출력 결과
    // aaa
    // aaabbb
    
    // 이번 예제에선 str 의 내용이 변경되는 것처럼 보이지만, 
    // 사실은 String 객체를 지우고 다시 새로 만든 것이다.
    ```

    - 생성을 다시 하는 것이 아닌 변경을 하려면 <code><strong>StringBuffer, StringBuilder</strong></code> 클래스를 사용하면 된다.