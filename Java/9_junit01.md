# 목차

1. [Junit 이란 무엇인가??](#1-junit-이란-무엇인가) <br/>
2. [단위 테스트란??](#2-단위-테스트란) <br/>
3. [JUnit 5 의 메서드들](#3-junit-5-의-메서드들) <br/>
4. [JUnit 5 의 어노테이션들](#4-junit-5-의-어노테이션들) <br/>

<br/>

# [자바, Java] JUnit 5 의 개념 및 기초 사용법

<br/>

> **어떠한 규칙과 목적이 정해져있는 프로그램을 구현하는 것을 통해 TDD 방법론을 적용하는 연습을 진행하게 되었다.**
>
> **여기서 처음 접해보는 것이 있었는데 그게 바로 Junit 테스트 코드 구현이다.**
>
> **국비지원 학원을 수료할 때까지 테스트 코드란 것이 있는지 몰랐고, 애초에 Junit 이 무엇인지조차 몰랐다.**
>
> **처음엔 assertThat() 클래스 메서드도 import 되지가 않아서 헤맸던 기억이 있다.**
>
> **내가 아는 것까지 간단히 정리해보고자 한다.**

<br/>

## 1. JUnit 이란 무엇인가??

- <code><strong>자바 프로그래밍 언어용 유닛 테스트 프레임워크이다.</strong></code>

  - 쉽게 말해서, 자바 애플리케이션에 대한 단위 테스트를 쉽게 해주는 테스트용 프레임워크이다.
  
- 단위 테스트의 대표적 프레임워크이다.

  

## 2. 단위 테스트란??

- <code><strong>특정 소스코드의 모듈이 의도한 대로 작동하는지 검증하는 테스트이다.</strong></code>

- 번거롭게 사람이 직접 System.out.println() 으로 테스트와 디버깅 해야 할 과정을 훨씬 쉽게 해준다.

  

## 3. JUnit 5 의 메서드들

1. <code><strong>assertEquals(A, B) </strong></code>

   - **A 와 B 가 같은 값인지 확인한다.**

   - 예제

     - ```java
       public class Calculate {
           public int add(int a, int b) {
               return a + b;
           }
       }
       
       
       public class CalculateTest {
           @Test
           void add() {
               Calculate calculate = new Calculate();
               int sum = calculate.add(3, 4);
               assertEquals(sum, 7);
           }
       }
       
       // 테스트 성공
       
       // =============================================================== //
       
       public class CalculateTest {
           @Test
           void add() {
               Calculate calculate = new Calculate();
               int sum = calculate.add(3, 4);
               assertEquals(sum, 8);
           }
       }
       
       // 테스트 실패
       ```
       
       

2. <code><strong>assertEquals(A, B, C) </strong></code>

  - **A 와 B 의 차이가 C 만큼의 오차범위 안인지 확인한다.**

  - 예제

    - ```java
      public class Calculate {
          public int add(int a, int b) {
              return a + b;
          }
      }
      
      
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              int sum = calculate.add(3, 4);
              assertEquals(sum, 7, 3);
              assertEquals(sum, 4, 3);
              assertEquals(sum, 10, 3);
          }
      }
      
      // 테스트 성공
      
      // ================================================== //
      
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              int sum = calculate.add(3, 4);
              assertEquals(sum, 3, 3);
              assertEquals(sum, 11, 3);
          }
      }
      
      // 테스트 실패
      ```

      

3. <code><strong>assertArrayEquals(A, B) </strong></code>

  - **배열 A 와 B 가 일치하는지 확인한다.**

  - 예제

    - ```java
      public class CalculateTest {
          @Test
          void add() {
              int[] a = {1, 2, 3};
              int[] b = {1, 2, 3};
              
              assertArrayEquals(a, b);
          }
      }
      
      // 테스트 성공
      
      // ===================================== //
      
      public class CalculateTest {
          @Test
          void add() {
              int[] a = {1, 2, 3};
              int[] b = {1, 2, 3, 4};
              
              assertArrayEquals(a, b);
          }
      }
      
      // 테스트 실패
      ```

      

4. <code><strong>assertSame(A, B) </strong></code>

  - **객체 A 와 B 의 주소값을 비교해서 같은지 확인한다.**

  - 예제

    - ```java
      public class CalculateTest {
          @Test
          void add() {
              ValueTest valueTest1 = new ValueTest(1, 2);
              ValueTest valueTest2 = valueTest1;
              
              assertSame(valueTest1, valueTest2);
          }
      }
      
      // 테스트 성공
      
      // ======================================================= //
      
      public class CalculateTest {
          @Test
          void add() {
              ValueTest valueTest1 = new ValueTest(1, 2);
              ValueTest valueTest2 = new ValueTest(1, 2);
              
              assertSame(valueTest1, valueTest2);
          }
      }
      
      // 테스트 실패
      ```

      

5. <code><strong>assertTrue(A) </strong></code>

  - **조건 A 가 참인지 확인한다.**

  - 예제

    - ```java
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              assertTrue(calculate.add(3, 4) == 7);
          }
      }
      
      // 테스트 성공
      
      // ======================================================= //
      
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              assertTrue(calculate.add(3, 4) == 8);
          }
      }
      
      // 테스트 실패
      ```

      

6. <code><strong>assertNull(A) </strong></code>

  - **객체 A 가 null 인지 확인한다.**

  - 예제

    - ```java
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = null;
              assertNull(calculate);
          }
      }
      
      // 테스트 성공
      
      // ======================================================= //
      
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              assertNull(calculate);
          }
      }
      
      // 테스트 실패
      ```

      

7. <code><strong>assertNotNull(A)</strong></code>

  - **객체 A 가 null 이 아닌지 확인한다.**

  - 예제

    - ```java
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = new Calculate();
              assertNotNull(calculate);
          }
      }
      
      // 테스트 성공
      
      // ======================================================= //
      
      public class CalculateTest {
          @Test
          void add() {
              Calculate calculate = null;
              assertNotNull(calculate);
          }
      }
      
      // 테스트 실패
      ```

      

## 4. JUnit 5 의 어노테이션들

1. <code><strong>@Test</strong></code>

   - **해당 어노테이션이 달린 메서드가 테스트 메서드라는 신호를 보내는 데 사용됩니다.**

     - 해당 메서드는 private 이거나 static 이어서는 안 되며 값을 반환해서는 안 됩니다. (void 만 허용)

     - 위의 예시에서 사용된 모습을 볼 수 있다.

       

2. <code><strong>@DisplayName(String displayName)</strong></code>

   - **해당 어노테이션의 인자인 문자열(displayName)이 테스트 실행 시 메서드의 역할을 나타낸다.**

   

3. <code><strong>@ParameterizedTest</strong></code>

   - **여러가지 값에 따른 테스트 코드를 각각 작성하는 것이 아닌, 인자로 Source들을 주입시켜 여러 테스트를 하나의 포맷에서 진행하는 것이다.**

   - @ValueSource , @CsvSource 등의 Source 어노테이션들이 같이 쓰이게 된다.

     

4. <code><strong>@ValueSource()</strong></code>

   - **여러가지 값의 테스트를 순차적으로 진행한다.**

   - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

   - 예제

     - ```java
       public class CalculateTest {
           @ParameterizedTest
           @ValueSource(ints = {5, 6, 7})
           void add(int num) {
               assertEquals(num, 7);
           }
       }
       
       // 테스트 횟수 3번
       // 1번째 실패
       // 2번째 실패
       // 3번째 성공
       ```

       

5. <code><strong>@CsvSource()</strong></code>

   - <code><strong>하나 이상의 인자값을 전달하고, 여러가지 값으로 테스트를 진행한다.</strong></code>

   - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

   - 예제

     - ```java
       public class CalculateTest {
           @ParameterizedTest
           @CsvSource(value = {"3, 4, 7", "4, 5, 10"})
           void add(int num1, int num2, int num3) {
               assertEquals(num1 + num2, num3);
           }
       }
       
       // 테스트 횟수 2번
       // 1번째 성공
       // 2번째 실패
       ```

       

6. <code><strong>@MethodSource()</strong></code>

   - **보다 복잡한 객체를 인자로 전달해서 테스트 할 때 사용 된다.**

   - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

   - **Arguments.of() 의 인자들이 테스트 메서드의 인자들에 순서대로 매핑되어 들어가는 것이다.**

   - 예제

     - ```java
       public class CalculateTest {
           @ParameterizedTest
           @MethodSource("addDocuments")
           void add(int[] numArr1, int[] numArr2, boolean check) {
               assertEquals(numArr1[0] == numArr2[0], check);
           }
           
           public static Stream<Arguments> addDocuments() {
               return Stream.of(
                   Arguments.of(new int[]{1, 2, 3}, new int[]{1, 2, 3}, true),
                   Arguments.of(new int[]{2, 2, 3}, new int[]{1, 2, 3}, true),
                   Arguments.of(new int[]{3, 2, 3}, new int[]{1, 2, 3}, false)
               );
           }
       }
       
       // 테스트 횟수 3번
       // 1번째 성공
       // 2번째 실패
       // 3번째 성공
       ```

       

7. <code><strong>@RepeatedTest()</strong></code>

   - **지정한 횟수 만큼 반복 테스트한다.**

   - 예제

     - ```java
       public class Calculate {
           public int getRanNum() {
               return (int) (Math.random() * 10); // 0 ~ 9
           }
       }
       
       
       public class CalculateTest {
           @RepeatedTest(20)
           void add() {
               Calculate calculate = new Calculate();
               assertTrue(calculate.getRanNum() < 10);
           }
       }
       
       // 테스트 횟수 20번
       // 20번 모두 성공
       ```

       

8. <code><strong>@NullSource</strong></code>

   - **인자로 null 을 한 번 전달해준다.**

   - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

   - 예제

     - ```java
       public class CalculateTest {
           @ParameterizedTest
           @NullSource
           void add(String s) {
               assertNull(s);
           }
       }
       
       // 테스트 성공
       
       // ============================= //
       
       public class CalculateTest {
           @ParameterizedTest
           @NullSource
           void add(String s) {
               assertNotNull(s);
           }
       }
       
       // 테스트 실패
       ```

       

9. <code><strong>@EmptySource</strong></code>

   - **인자로 빈("")  데이터를 전달한다.**

   - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

   - 예제

     - ```java
       public class CalculateTest {
           @ParameterizedTest
           @EmptySource
           void add(String s) {
               assertEquals(s, "");
           }
       }
       
       // 테스트 성공
       
       // ============================ //
       
       public class CalculateTest {
           @ParameterizedTest
           @EmptySource
           void add(String s) {
               assertEquals(s, "s");
           }
       }
       
       // 테스트 실패
       ```

       

10. <code><strong>@NullAndEmptySource</strong></code>

    - **@NullSource 와 @EmptySource 를 합친 것으로 null 과 빈("") 데이터를 순차적으로 인자에 전달한다.**

    - @ParameterizedTest 어노테이션과 같이 쓰이는 Source 어노테이션이다.

    - 예제

      - ```java
        public class CalculateTest {
            @ParameterizedTest
            @NullAndEmptySource
            void add(String s) {
                assertTrue(StringUtils.isBlank(s)); // isBlank() 메서드는 null 과 빈 문자열을 전부 true 로 반환한다.
            }
        }
        
        // 테스트 횟수 2번
        // 2번 모두 성공
        ```

        

11. <code><strong>@BeforeEach</strong></code>

    - **각 테스트 코드가 실행되기 전에 먼저 한 번씩 실행되는 메서드를 뜻한다.**

      

12. <code><strong>@AfterEach</strong></code>

    - **각 테스트 코드가 실행된 후에 한 번씩 실행되는 메서드를 뜻한다.**

      

13. <code><strong>@BeforeAll</strong></codE>

    - **모든 테스트 코드가 실행되기 전에 한 번 실행되는 메서드를 뜻한다.**

      

14. <code><strong>@AfterAll</strong></code>

    - **모든 테스트 코드가 실행된 후에 한 번 실행되는 메서드를 뜻한다.**
