# 목차

1. [Private Variable 접근 방법](#1-private-variable-접근-방법) <br/>
2. [Private Method 접근 방법](#2-private-method-접근-방법) <br/>

<br/>

# [자바, Java] JUnit - private 변수, 메서드 테스트하기

<br/>

> **자바에서 테스트 코드 작성 시, private 변수 또는 메서드를 테스트해야 하는 경우가 생길 수 있다.**
>
> **물론 최대한 private 멤버를 끌어들이지 않도록 애초에 설계를 잘 하는 것이 우선이겠지만, 알아두도록 하자.**

<br/>

## 1. Private Variable 접근 방법

- ```java
  public class Calculate {
      private final int a;
      private final int b;
      
      public Calculate(int a, int b) {
          this.a = a;
          this.b = b;
      }
  }
  
  // ==================================== //
  
  class CalculateTest {
      @Test
      void private_num_test() {
          Calculate calculate = new Calculate(2, 3); // a = 2, b = 3
          int a = 0;
          int b = 0;
          try {
              Field field = calculate.getClass().getDeclaredField("a"); // Field를 생성하고 내가 얻고자하는 변수명을 입력해준다.
              field.setAccessible(true); // 해당 변수를 field에 담았으니 접근 가능하게 만든다.
              a = (int) field.get(calculate); // field.get(생성한 클래스명)을 통해 얻고자했던 private value를 얻어주면 된다.
      
              Field field2 = calculate.getClass().getDeclaredField("b");
              field2.setAccessible(true);
              b = (int) field2.get(calculate);
          } catch (NoSuchFieldException | IllegalAccessException e) {
              throw new RuntimeException(e);
          }
          
          assertThat(a + b).isEqualTo(5); // 테스트 성공
      }
  }
  
  // 테스트 성공
  ```

  

- 여기서 private 변수를 여러개 뺄 때 코드가 지저분해진다.

  - 그래서 내가 개인적으로 쓰는 방법이 있다. (더 좋은 방식이 있다면 알려주십쇼.)

  - ```java
    public class Calculate {
        private final int a;
        private final int b;
        
        public Calculate(int a, int b) {
            this.a = a;
            this.b = b;
        }
    }
    
    // ==================================== //
    
    class CalculateTest {
        @Test
        void private_num_test() {
            Calculate calculate = new Calculate(2, 3);
            int a = (int) getPrivateVariable(calculate, "a");
            int b = (int) getPrivateVariable(calculate, "b");
            
            assertThat(a + b).isEqualTo(5);
        }
        
        private Object getPrivateVariable(Object object, String variableName) {
            try {
                Field field = object.getClass().getDeclaredField(variableName);
                field.setAccessible(true);
                return field.get(object);
            } catch (NoSuchFieldException | IllegalAccessException e) {
                throw new RuntimeException(e);
            }
        }
    }
    
    // 테스트 성공
    ```

    - 이런식으로 형변환을 해서 반환 받게끔 메서드를 구현해서 사용한다.

      - 이러면 몇개를 빼든, 여러 타입을 빼든 이 메서드 하나로 전부 커버가 가능하다.

        

## 2. Private Method 접근 방법

- ```java
  public class Calculate {
      private int add(int a, int b) {
          return a + b;
      }
  }
  
  // ========================================= //
  
  class CalculateTest {
      @Test
      @DisplayName("더하기")
      void add() {
          Calculate calculate = new Calculate();
          int sum = 0;
          try {
              // getDeclaredMethod 첫번째 인자에 메서드 이름, 두번째 인자부턴 메서드의 매개변수 타입을 순서대로 지정해준다.
              Method method = calculate.getClass().getDeclaredMethod("add", int.class, int.class);
              method.setAccessible(true); // 해당 메서드에 접근 가능하도록 설정
              sum = (int) method.invoke(calculate, 3,4); // 해당 메서드를 인자 전달과 함께 호출하고, 값을 반환받는다.
          } catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException e) {
              throw new RuntimeException(e);
          }
          
          assertThat(sum).isEqualTo(7); // 테스트 성공
      }
  }
  
  // 테스트 성공
  ```

  - 여기도 마찬가지로 private 메서드를 여러개 호출할 때 코드가 지저분해진다.

    - 그래서 내가 개인적으로 쓰는 방법이 있다.

    - ```java
      public class Calculate {
          private int add(int a, int b) {
              return a + b;
          }
      }
      
      // ===================================== //
      
      class CalculateTest {
          @Test
          @DisplayName("더하기")
          void add() {
              Calculate calculate = new Calculate();
              int sum = (int) getPrivateMethodResult(calculate, "add", 3, 4);
              assertThat(sum).isEqualTo(7);
          }
          
          private Object getPrivateMethodResult(Object object, String methodName, int n1, int n2) {
              try {
                  Method method = object.getClass().getDeclaredMethod(methodName, int.class, int.class);
                  method.setAccessible(true);
                  return method.invoke(object, n1, n2);
              } catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException e) {
                  throw new RuntimeException(e);
              }
          }
      }
      
      // 테스트 성공
      ```

      - 이런식으로 같은 private method 나 같은 매개변수 타입과 순서를 가진 메서드를 추가로 호출할 때, 형변환 해서 갖다 쓸 수 있도록 구현한다.

