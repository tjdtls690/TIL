# 목차

1. [AssertJ 란 무엇인가??](#1-assertj-란-무엇인가) <br/>
2. [AssertJ 의 메서드들](#2-assertj-의-메서드들) <br/>

<br/>

# [자바, Java] AssertJ 의 개념 및 기초 사용법

<br/>

> 맨 처음엔 assertThat() 메서드를 JUnit API 알고 사용했었는데,
>
> 알고보니 AssertJ API였다.
>
> AssertJ API 의 정체는 무엇인가??

<br/>

## 1. AssertJ 란 무엇인가??

- **테스트코드를 작성 시, JUnit 보다 더욱 테스트 코드의 가독성과 편의성을 높여 주는 라이브러리이다.**

- <code><strong>메소드 체이닝</strong></code>을 지원하기 때문에 좀 더 깔끔하고 읽기 쉬운 테스트 코드를 작성할 수 있습니다.

  - 형식 : assertThat(테스트 타겟).메소드1().메소드2().메소드3()'

  - **assertThat() 으로 시작한다.**

    

## 2. AssertJ 의 메서드들

- 문자열 테스트 예제

  - ```java
    public class CalculateTest {
        @Test
        void add() {
            String s = assertThat("Java Test Success.")   // 주어진 "Java Test Success." 라는 문자열은
                    .isNotEmpty()   // 비어있지 않고
                    .isNotNull() // null 이 아니며
                    
                    .startsWith("Jav") // "Jav"로 시작하고
                    .endsWith("s.") // "s."로 끝나며
                    
                    .doesNotContain("aaa")  // "aaa"는 포함하지 않으며
                    .contains("Java")   // "Java"를 포함하고
                    .contains("Success")  // "Success"도 포함하며
                    
    	            // "Java Test Success." 와 equals() 메서드로 비교시 true 이고 (데이터 비교)
                    .isEqualTo(new String("Java Test Success."))
                    .isNotEqualTo("ggg")  // "ggg" 와 equals() 메서드로 비교시 false 이며 (데이터 비교)
                    
                    .isSameAs("Java Test Success.") // "Java Test Success." == "Java Test Success." 가 true 이고 (주소값 비교)
    	            // "Java Test Success." == new String("Java Test Success.") 가 false 이며 (주소값 비교)
                    .isNotSameAs(new String("Java Test Success.")) 
                
                    .isInstanceOf(String.class) // String 클래스이고
                    .isInstanceOf(CharSequence.class) // String 이 구현한 CharSequence 인터페이스이기도 하고
                    .isNotInstanceOf(Character.class) // Character 클래스는 아니며
                    
                    .isInstanceOfAny(String.class, Character.class) // String 클래스이거나 또는 Character 클래스이고
                    .isNotInstanceOfAny(Character.class, Integer.class) // Character 클래스가 아니고 Integer 클래스가 아니며
                    
                    .isExactlyInstanceOf(String.class) // '정확하게는' String 클래스이고
                    .isNotExactlyInstanceOf(CharSequence.class) // '정확하게' CharSequence 인터페이스는 아니며
                    
                    .toString(); // getClass().getName() + '@' + Integer.toHexString(hashCode()) 를 반환한다.
            
            System.out.println(s);
        }
    }
    
    // 테스트 성공
    
    // 출력 결과
    // org.assertj.core.api.StringAssert@1
    ```

  

- 숫자 테스트 예제

  - ```java
    public class CalculateTest {
        @Test
        void add() {
            String s = assertThat(8.67)   // 주어진 8.67 이라는 숫자는
                    .isBetween(8.67, 9d) // 8.67 이상 9 이하이고
                    .isStrictlyBetween(8.66, 9d) // 8.66 초과 9 미만이며
                
                    .isCloseTo(8, offset(0.67d)) // 8과의 오차범위가 0.67 이내이고
                    .isCloseTo(8, withPercentage(10)) // 8과의 오차범위가 10퍼센트 이내이며
                
                    .isNotCloseTo(8, offset(0.66d)) // 8과의 오차범위가 0.66 초과이고
                    .isNotCloseTo(8, withPercentage(1)) // 8과의 오차범위가 1퍼센트 초과이며
                
                    .isGreaterThan(8)   // 8보다 크고
                    .isLessThan(9)  // 9보다 작으며
                
                    .isPositive()   // 양수이고
                    .isNotNegative() // 음수가 아니고
                    .isNotZero() // 0이 아니며
                
                    .isFinite() // 유한한 숫자이고
                    .isNotNaN() // NAN 이 아니며
                
                    .isEqualTo(8, offset(1d))  // 8과의 차이가 오차범위 1 이내이고
                    .isEqualTo(8.6, offset(0.1d))  // 8.6과의 차이가 오차범위 0.1 이내이며
                    .isEqualTo(8.67)   // 오프셋 없이는 8.67와 같습니다
                
                    .toString(); // getClass().getName() + '@' + Integer.toHexString(hashCode()) 를 반환한다
        
            System.out.println(s);
        }
    }
    
    // 테스트 성공
    
    // 출력 결과
    // org.assertj.core.api.DoubleAssert@1
    ```