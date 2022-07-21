# 목차

1. [StringUtils API가 만들어진 이유](#1-stringutils-api가-만들어진-이유) <br/>
2. [StringUtils dependency 추가](#2-stringutils-dependency-추가) <br/>
3. [StringUtils 의 메서드들](#3-stringutils-의-메서드들) <br/>
    1. [isEmpty(final CharSequence cs)](#1-isemptyfinal-charsequence-cs) <br/>
    2. [isBlank(final CharSequence cs)](#2-isblankfinal-charsequence-cs) <br/>
    3. [trim(final String str)](#3-trimfinal-string-str) <br/>
    4. [strip(final String str)](#4-stripfinal-string-str) <br/>
    4. [equals(final CharSequence cs1, final CharSequence cs2)](#5-equalsfinal-charsequence-cs1-final-charsequence-cs2) <br/>
    4. [compare(final String str1, final String str2)](#6-comparefinal-string-str1-final-string-str2) <br/>

<br/>

# [자바, Java] StringUtils API (1)

<br/>

> **문자열 계산기 프로그램을 만드는 중에,** 
>
> **입력받은 문자열 안에 여러개의 특정 문자열이 포함되어있는지 확인해야 하는 상황이 있었다.**
>
> **근데 String 클래스의 contains() 메서드는 하나의 문자열만 확인할 수 있어서**
>
> **여러개가 포함되어있는지 확인하려면 조건식을 여러개 사용해야하는 상황이었다.**
>
> **그렇다면 이걸 한번에 처리할 수 있는 메서드는 없을까?? 찾는 도중에**
>
> **StringUtils API 를 찾게 되었고, 이 클래스를 공부한 것들을 간단히 정리해보려 한다.**



# 1. StringUtils API가 만들어진 이유

- **우선, 직접 StringUtils 클래스에 들어가서 어떤 메서드들이 있는지 찾아보았다.**

  - 근데 클래스에 들어가보니 맨 첫 줄 주석에 <code><strong>Operations on String that are null safe.</strong></code> 라 적혀있다.

  - 즉, String 클래스와 같이 문자열에 관한 작업이지만, NULL 에게서 안전하게 작업할 수 있도록 업그레이드 시킨것으로 보인다.

  - 쉽게 말해서 String 클래스의 더 안전한 업그레이드 판(?) 이라 보면 될 듯 싶다.

    

# 2. StringUtils dependency 추가

- 기본 라이브러리가 아닌 'org.apache.commons' 에 들어있기에 dependency 로 추가해줘야 한다.

- ```groovy
  dependencies {
      (...)
      implementation 'org.apache.commons:commons-lang3:3.12.0'
  }
  ```

  

# 3. StringUtils 의 메서드들

- **더 많은 메서드들이 있었지만 쓸만한 메서드들로 추려보았다.**

  - **IsEmpty/IsBlank** - 문자열에 텍스트가 포함되어 있는지 확인

  - **Trim/Strip** - 선행 및 후행 공백 제거

  - **Equals/Compare** - null 안전 방식으로 두 문자열을 비교합니다.

  - **IndexOfAny/LastIndexOfAny/IndexOfAnyBut/LastIndexOfAnyBut** - 문자열 집합의 인덱스

  - **ContainsOnly/ContainsNone/ContainsAny** - 문자열에 해당 문자열만 있는가?  /  문자열에 해당 문자열이 없는가?  /  해당 문자열이 하나라도 포함되어 있는지 확인합니다.

  - **Remove/Delete** - 문자열의 일부를 제거합니다.

  - **Replace/Overlay** - 문자열을 검색하고 한 문자열을 다른 문자열로 바꿉니다.

  - **UpperCase/LowerCase/SwapCase/Capitalize/Uncapitalize** - 문자열의 대소문자를 변경합니다.

  - **Reverse/ReverseDelimited** - 문자열 반전

    

- 여기서 특정 여러개의 문자열을 전부 포함하는지 알 수 있는 메서드는 없는 것 같아서 아쉬웠다.

  - 여러 문자열들 중 하나만이라도 포함하면 true 를 반환해주는 기능은 있었다.

- 하지만, 그 외의 쓸만한 메서드들이 정말 많았다. 

- 그래서 클래스 안의 설명은 위와 같이 써져있었지만, **더 자세히 알아보기 위해 한 번씩 전부 사용해 보기로 했다.**

  

- ## 1. isEmpty(final CharSequence cs)

  - <code><strong>해당 문자열이 비어있는지 확인하는 메서드</strong></code>

  - 메서드 실제 구현 모습

    - ```java
      public static boolean isEmpty(final CharSequence cs) {
          return cs == null || cs.length() == 0;
      }
      
      // 즉, true 가 나오는 조건은 두가지
      // 1. StringUtils.isEmpty(null)      = true
      // 2. StringUtils.isEmpty("")        = true
      
      // 단 한개의 문자 또는 공백이라도 들어가 있으면 false 가 뜬다.
      ```

    - ```java
      import org.apache.commons.lang3.StringUtils;
      
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.isEmpty(null));
              System.out.println(StringUtils.isEmpty(""));
              System.out.println(StringUtils.isEmpty(" "));
              System.out.println(StringUtils.isEmpty("b"));
          }
      }
      
      // 출력 결과
      // true
      // true
      // false
      // false
      ```

      

  - 여기서 인자로 <code><strong>CharSequence</strong></code> 가 나왔는데 이건 뭘까??

    - 이름만 봐서는 **'character 형으로 이루어진 배열 == String'** 이렇게 이해가 되었다.

      - 왜냐면 String 클래스도 본질은 character 형 배열인 것을 알고있었기 때문이다.

      - 혹시나 싶어서 여러가지 시도를 해봤다.

      - ```java
        class Scratch {
            public static void main(String[] args) {
                CharSequence c = "aabbccdd";
                String s = "하나둘셋넷";
                System.out.println(c + s);
                System.out.println(c.subSequence(0, 4));
            }
        }
        
        // aabbccdd하나둘셋넷
        // aabb
        ```

      - String 타입과 거의 같은 취급을 받고있는 것으로 보인다.

        - substring() 메서드와 같은 역할인 subSequence() 메서드도 존재한다.
        - 다른 비슷한 메서드들도 많은 것으로 보였다.

        

  - ### 1-1. CharSequence 

    - 결국 못참고 CharSequence 클래스도 뒤져보기로 결심했다.

      - 제일 첫 문장에 <code><strong>'CharSequence 는 읽을 수 있는 char 값 시퀀스입니다.'</strong></code> 라고 적혀있다.

        - 아까 내가 예상한 character 형 배열이 맞는 생각인 것 같다.

      - 그럼, String 클래스와 CharSequence 의 차이점은 무엇일까??

        1. String 은 클래스, CharSequence 는 인터페이스이다.
           - String 클래스는 CharSequence 인터페이스를 구현한 객체이다.
           - 이 외에도 String, SpannableStringBuilder, StringBuilder, StringBuffer 등이 CharSequence 를 구현했다.
        2. String 은 '변경할 수 없는 문자열', CharSequence 는 '스타일 문자 또는 연속된 문자' 라고도 부른다
           - 쉽게 말해서 String 은, 마크업 문자를 입출력할 때 문제가 발생하고
           - CharSequence 는 변형과 가공을 할 수 있기에, String과 같은 유니코드라도 마크업 문자를 사용할 수 있다.

      - String 과 CharSequence 의 차이점을 아주아주 간단히만 살펴봤는데 이따가 더 깊게 파봐야 겠다.

        

      - 무튼 지금까지 알아본 바로는 <code><strong>String 은 CharSequence 의 구현체이기에 String 도 인자로 가능하다는 것을 알 수 있다.</strong></code>

        

- ## 2. isBlank(final CharSequence cs)

  - <code><strong>해당 문자열이 비어있는지 확인하는 메서드</strong></code>

  - 메서드 실제 구현 모습

  - ```java
    public static boolean isBlank(final CharSequence cs) {
        final int strLen = length(cs);
        if (strLen == 0) {
            return true;
        }
        for (int i = 0; i < strLen; i++) {
            if (!Character.isWhitespace(cs.charAt(i))) {
                return false;
            }
        }
        return true;
    }
    ```

  - isEmpty() 와 다른 점은 공백도 true 를 반환한다는 것이다.

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.isBlank(null));
              System.out.println(StringUtils.isBlank(""));
              System.out.println(StringUtils.isBlank(" "));
              System.out.println(StringUtils.isBlank("b"));
          }
      }
      
      // 출력 결과
      // true
      // true
      // true
      // false
      ```

      

- ## 3. trim(final String str)

  - <code><strong>해당 문자열의 선행과 후행 공백 제거</strong></code>

  - 메서드 실제 구현 모습

    - ```java
      public static String trim(final String str) {
          return str == null ? null : str.trim();
      }
      ```

    - String 의 trim() 메서드와 기능이 같은 것을 볼 수 있다.

      - ```java
        class Scratch {
            public static void main(String[] args) {
                System.out.println(StringUtils.trim("  aa bb cc  "));
            }
        }
        
        // 출력 결과
        // aa bb cc
        ```

      - 문자열 사이에 있는 공백 말고 가장 바깥쪽의 공백들만 제거된 모습을 볼 수 있다.

        

- ## 4. strip(final String str)

  - <code><strong>해당 문자열의 선행과 후행 공백 제거</strong></code>

  - 메서드 실제 구현 모습

    - ```java
      public static String strip(final String str) {
          return strip(str, null);
      }
      ```

    - 예제

      - ```java
        class Scratch {
            public static void main(String[] args) {
                System.out.println(StringUtils.strip("  aa bb cc  "));
            }
        }
        
        // 출력 결과
        // aa bb cc
        ```

        

  - <code><strong>trim() 과 strip() 의 차이점</strong></code>

    - trim() : '\u0020' 이하의 공백들만 제거합니다.

    - strip() : 유니코드의 공백들을 모두 제거합니다.

    - 유니코드엔 스페이스('\u0020'), 탭('\u0009) 을 제외하고 더 많은 공백이 존재합니다.

      

- ## 5. equals(final CharSequence cs1, final CharSequence cs2)

  - <code><strong>두 문자열이 같은지 확인</strong></code>

  - 메서드 실제 구현 모습

    - ```java
      public static boolean equals(final CharSequence cs1, final CharSequence cs2) {
          if (cs1 == cs2) {
              return true;
          }
          if (cs1 == null || cs2 == null) {
              return false;
          }
          if (cs1.length() != cs2.length()) {
              return false;
          }
          if (cs1 instanceof String && cs2 instanceof String) {
              return cs1.equals(cs2);
          }
          // Step-wise comparison
          final int length = cs1.length();
          for (int i = 0; i < length; i++) {
              if (cs1.charAt(i) != cs2.charAt(i)) {
                  return false;
              }
          }
          return true;
      }
      ```

    - 예제

      - ```java
        class Scratch {
            public static void main(String[] args) {
                System.out.println(StringUtils.equals(null, null));
                System.out.println(StringUtils.equals(null, "abc"));
                System.out.println(StringUtils.equals("abc", null));
                System.out.println(StringUtils.equals("abc", "abc"));
                System.out.println(StringUtils.equals("abc", "ABC"));
            }
        }
        
        // 출력 결과
        // true
        // false
        // false
        // true
        // false
        ```

        

- ## 6. compare(final String str1, final String str2)

  - <code><strong>두 문자열을 사전식으로 비교해서 음수, 0, 양수 를 반환</strong></code>

  - 메서드 실제 구현 모습

    - ```java
      public static int compare(final String str1, final String str2) {
          return compare(str1, str2, true);
      }
      ```

      - str1 이 str2 (또는 둘 다 null )인 경우=> int = 0 
      - str1 이 str2 보다 작은 경우 => int < 0 
      - str1 이 str2 보다 큰 경우 => int > 0 

    - 예제

      - ```java
        class Scratch {
            public static void main(String[] args) {
                System.out.println(StringUtils.compare(null, null));
                System.out.println(StringUtils.compare(null , "a"));
                System.out.println(StringUtils.compare("a", null));
                System.out.println(StringUtils.compare("abc", "abc"));
                System.out.println(StringUtils.compare("a", "b"));
                System.out.println(StringUtils.compare("b", "a"));
                System.out.println(StringUtils.compare("a", "B"));
                System.out.println(StringUtils.compare("ab", "abc"));
            }
        }
        
        // 출력 결과
        // 0
        // -1
        // 1
        // 0
        // -1
        // 1
        // 31
        // -1
        ```

# 나머지 메서드는 (2) 편에서...