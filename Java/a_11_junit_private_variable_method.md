# 목차

1. [Private Variable 접근 방법](#1-private-variable-접근-방법) <br/>

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
          
          assertThat(a + b).isEqualTo(5);
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