# 목차

1. [split(".")](#1-split---점) <br/>
2. [split(" ")](#2-split----공백) <br/>
3. [구분자를 여러개 지정하기](#3-구분자를-여러개-지정하기) <br/>

<br/>

# [자바, Java] String의 split() 메서드

<br/>

> **프로그램을 만드는 중에, split() 메서드를 이용해서 입력받은 문자열을 분리해야 할 상황이 있었다.**
>
> **여기서 문자열 입력 시에 구분자도 입력받아서 구분자를 마음대로 지정할 수 있도록 구현 했다.**
>
> **근데 이상하게 점은 구분자로 지정이 되질 않는다.**
>
> **그래서 split 메서드에 대해 의도치 않게 공부를 하게 되었는데,** 
>
> **그 과정에서 배운 것들을 간단히 정리 해보고자 한다.**



# 1. split(".") - 점

- 점을 구분자로 해서 나누는 2가지 방법

  1. split("[.]")

  2. split("\\\\.")

     ```java
     class Scratch {
         public static void main(String[] args) {
             String str = "aa.bb.cc";
             String[] split = str.split(".");
             for (String s : split) {
                 System.out.println(s);
             }
         }
     }
     
     // 출력 결과
     // []
     
     // 아무것도 출력되지 않음
     ```

     ```java
     import java.util.Arrays;
     
     class Scratch {
         public static void main(String[] args) {
             String str = "aa.bb.cc";
             String[] split = str.split("\\.");
             System.out.println(Arrays.toString(split));
             for (String s : split) {
                 System.out.println(s);
             }
         }
     }
     
     // 출력 결과
     // [aa, bb, cc]
     // aa
     // bb
     // cc
     ```

     ```java
     import java.util.Arrays;
     
     class Scratch {
         public static void main(String[] args) {
             String str = "aa.bb.cc";
             String[] split = str.split("[.]");
             System.out.println(Arrays.toString(split));
             for (String s : split) {
                 System.out.println(s);
             }
         }
     }
     
     // 출력 결과
     // [aa, bb, cc]
     // aa
     // bb
     // cc
     ```

     

# 2. split(" ") - 공백

- 공백은 점과 달리 잘 나눠졌지만, 잘 안나눠지는 경우도 있다고 한다.

- 공백을 구분자로 해서 나누는 방법 2가지

  1. split("\\\\s")

  2. split(" ")

     ```java
     import java.util.Arrays;
     
     class Scratch {
         public static void main(String[] args) {
             String str = "aa bb cc";
             String[] split = str.split("\\s");
             System.out.println(Arrays.toString(split));
             for (String s : split) {
                 System.out.println(s);
             }
         }
     }
     
     // 출력 결과
     // [aa, bb, cc]
     // aa
     // bb
     // cc
     ```

     

     ```java
     import java.util.Arrays;
     
     class Scratch {
         public static void main(String[] args) {
             String str = "aa bb cc";
             String[] split = str.split(" ");
             System.out.println(Arrays.toString(split));
             for (String s : split) {
                 System.out.println(s);
             }
         }
     }
     
     // 출력 결과
     // [aa, bb, cc]
     // aa
     // bb
     // cc
     ```

     

# 3. 구분자를 여러개 지정하기

- 점을 구분자로 사용하기 위해 썼던 "[]" 대괄호를 사용하면 된다.

  ```java
  import java.util.Arrays;
  
  class Scratch {
      public static void main(String[] args) {
          String str = "aa,bb;cc;,dd";
          String[] split = str.split(";,");
          System.out.println(Arrays.toString(split));
          for (String s : split) {
              System.out.println(s);
          }
      }
  }
  
  // 출력 결과
  // [aa,bb;cc, dd]
  // aa,bb;cc
  // dd
  
  // ;, 가 함께 붙어있는 것만 구분자로 취급하는 모습을 볼 수 있다.
  ```

  

  ```java
  import java.util.Arrays;
  
  class Scratch {
      public static void main(String[] args) {
          String str = "aa,bb;cc;,dd";
          String[] split = str.split("[;,]");
          System.out.println(Arrays.toString(split));
          for (String s : split) {
              System.out.println(s);
          }
      }
  }
  
  // 출력 결과
  // [aa, bb, cc, , dd]
  // aa
  // bb
  // cc
  // 
  // dd
  
  // 제대로 구분되는 모습을 볼 수 있다.
  ```

  