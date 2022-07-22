# 목차

7. [indexOfAny(final CharSequence cs, final char... searchChars)](#7-indexofanyfinal-charsequence-cs-final-char-searchchars) <br/>
8. [lastIndexOfAny(final CharSequence str, final CharSequence... searchStrs)](#8-lastindexofanyfinal-charsequence-str-final-charsequence-searchstrs) <br/>
9. [indexOfAnyBut(final CharSequence seq, final CharSequence searchChars)](#9-indexofanybutfinal-charsequence-seq-final-charsequence-searchchars) <br/>
10. [lastIndexOfAnyBut](#10-lastindexofanybut) <br/>
11. [containsOnly(final CharSequence cs, final String validChars)](#11-containsonlyfinal-charsequence-cs-final-string-validchars) <br/>
12. [containsNone(final CharSequence cs, final String invalidChars)](#12-containsnonefinal-charsequence-cs-final-string-invalidchars) <br/>
13. [containsAny(final CharSequence cs, final CharSequence searchChars)](#13-containsanyfinal-charsequence-cs-final-charsequence-searchchars) <br/>
14. [remove(final String str, final String remove)](#14-removefinal-string-str-final-string-remove) <br/>
15. [delete](#15-delete) <br/>

<br/>

# [자바, Java] StringUtils API (2)

<br/>

- ## 7. indexOfAny(final CharSequence cs, final char... searchChars)

  - <code><strong>CharSequence 를 검색하여 주어진 문자 집합에서 문자의 첫 번째 인덱스를 찾는다.</strong></code>

  - 메서드 실제 구현 모습은 복잡하기에 안적는게 좋겠다..

    - 예제

      - ```java
        class Scratch {
            public static void main(String[] args) {
                System.out.println(StringUtils.indexOfAny(null, 'a'));
                System.out.println(StringUtils.indexOfAny("", 'a'));
                // System.out.println(StringUtils.indexOfAny("abcd", null)); // 에러
                System.out.println(StringUtils.indexOfAny("zzabyycdxx", 'z', 'a'));
                System.out.println(StringUtils.indexOfAny("zzabyycdxx", 'c', 'y'));
                System.out.println(StringUtils.indexOfAny("aba", 'z'));
            }
        }
        
        // 출력 결과
        // -1
        // -1
        // 0
        // 4
        // -1
        ```

        

- ## 8. lastIndexOfAny(final CharSequence str, final CharSequence... searchStrs)

  - <code><strong>CharSequence 를 검색하여 주어진 문자 집합에서 문자의 마지막 인덱스를 찾는다.</strong></code>

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.lastIndexOfAny(null, "a"));
              System.out.println(StringUtils.lastIndexOfAny("a", null));
              System.out.println(StringUtils.lastIndexOfAny("zzabyycdxx", "ab", "cd"));
              System.out.println(StringUtils.lastIndexOfAny("zzabyycdxx", "cd", "ab"));
              System.out.println(StringUtils.lastIndexOfAny("zzabyycdxx", "mn", "op"));
              System.out.println(StringUtils.lastIndexOfAny("zzabyycdxx", "mn", "op"));
              System.out.println(StringUtils.lastIndexOfAny("zzabyycdxx", "mn", ""));
          }
      }
      
      // 출력 결과
      // -1
      // -1
      // 6
      // 6
      // -1
      // -1
      // 10
      ```

      

- ## 9. indexOfAnyBut(final CharSequence seq, final CharSequence searchChars)

  - <code><strong>CharSequence 를 검색하여 주어진 문자 집합에 없는 문자의 첫 번째 인덱스를 찾는다.</strong></code>

    - null CharSequence는 -1 을 반환한다.
    - null 또는 빈 검색 문자열은 -1 을 반환한다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.indexOfAnyBut(null, "a"));
              System.out.println(StringUtils.indexOfAnyBut("", "a"));
              // System.out.println(StringUtils.indexOfAnyBut("a", null)); // 에러
              System.out.println(StringUtils.indexOfAnyBut("a", ""));
              System.out.println(StringUtils.indexOfAnyBut("zzabyycdxx", "za"));
              System.out.println(StringUtils.indexOfAnyBut("zzabyycdxx", ""));
              System.out.println(StringUtils.indexOfAnyBut("aba", "ab"));
          }
      }
      
      // 출력 결과
      // -1
      // -1
      // -1
      // 3
      // -1
      // -1
      ```

      

- ## 10. lastIndexOfAnyBut

  - 주석 설명엔 있다고 써있지만, 실제로 클래스를 뒤져보면 존재하지 않는 메서드이다.

    - 왜 그런지는 좀 더 찾아봐야 할 듯 싶다.

    - 버전이 올라가면서 deprecated 된건가..?

      

- ## 11. containsOnly(final CharSequence cs, final String validChars)

  - <code><strong>CharSequence 에 특정 문자만 포함되어 있는지 확인한다.</strong></code>

    - null CharSequence는 false 를 반환한다.
    - 빈 문자열 (length()=0) 은 항상 true 를 반환한다.

  - 메서드 실제 구현 모습

    - ```java
      public static boolean containsOnly(final CharSequence cs, final char... valid) {
          if (valid == null || cs == null) {
              return false;
          }
          if (cs.length() == 0) {
              return true;
          }
          if (valid.length == 0) {
              return false;
          }
          return indexOfAnyBut(cs, valid) == INDEX_NOT_FOUND;
      }
      
      // 두번째 인자값이 다르지만, 어차피 String 을 인자값으로 쓰는 메서드도 이 메서드를 쓰기에 상관 없다.
      ```

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.containsOnly(null, "a"));
              // System.out.println(StringUtils.containsOnly("a", null)); // 에러
              System.out.println(StringUtils.containsOnly("", "a"));
              System.out.println(StringUtils.containsOnly("ab", ""));
              System.out.println(StringUtils.containsOnly("abab", "abc"));
              System.out.println(StringUtils.containsOnly("ab1", "abc"));
              System.out.println(StringUtils.containsOnly("abz", "abc"));
          }
      }
      
      // 출력 결과
      // false
      // true
      // false
      // true
      // false
      // false
      ```

      

- ## 12. containsNone(final CharSequence cs, final String invalidChars)

  - <code><strong>CharSequence에 특정 문자가 포함되어 있지 않은지 확인한다.</strong></code>

    - null CharSequence 는 true 를 반환한다
    - 빈 문자열("")은 항상 true를 반환한다

  - 이 메서드도 꽤 복잡하므로 실제 메서드는 생략...

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.containsNone(null, "a"));
              // System.out.println(StringUtils.containsNone("a", null)); // 에러
              System.out.println(StringUtils.containsNone("", "a"));
              System.out.println(StringUtils.containsNone("ab", ""));
              System.out.println(StringUtils.containsNone("abab", "xyz"));
              System.out.println(StringUtils.containsNone("ab1", "xyz"));
              System.out.println(StringUtils.containsNone("abz", "xyz"));
          }
      }
      
      // 출력 결과
      // true
      // true
      // true
      // true
      // true
      // false
      ```

      

- ## 13. containsAny(final CharSequence cs, final CharSequence searchChars)

  - <code><strong>CharSequence에 지정된 문자 집합에 한 문자라도 포함되어 있는지 확인한다.</strong></code>

    - null CharSequence는 false 를 반환한다.

  - 이 메서드도 꽤 복잡하므로 실제 메서드는 생략...

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.containsAny(null, "a"));
              System.out.println(StringUtils.containsAny("", "a"));
              // System.out.println(StringUtils.containsAny("a", null)); // 에러
              System.out.println(StringUtils.containsAny("a", ""));
              System.out.println(StringUtils.containsAny("zzabyycdxx", "za"));
              System.out.println(StringUtils.containsAny("zzabyycdxx", "by"));
              System.out.println(StringUtils.containsAny("zzabyycdxx", "zy"));
              System.out.println(StringUtils.containsAny("zzabyycdxx", "\tx"));
              System.out.println(StringUtils.containsAny("zzabyycdxx", "$.#yF"));
              System.out.println(StringUtils.containsAny("aba", "z"));
          }
      }
      
      // 출력 결과
      // false
      // false
      // false
      // true
      // true
      // true
      // true
      // true
      // false
      ```

      

- ## 14. remove(final String str, final String remove)

  - <code><strong>문자열에서 2번째 인자의 문자들은 전부 삭제한다.</strong></code>

    - null 소스 문자열은 null 을 반환합니다. 
    - 빈("") 소스 문자열은 빈 문자열을 반환합니다. 
    - null 제거 문자열은 소스 문자열을 반환합니다. 
    - 빈("") 제거 문자열은 소스 문자열을 반환합니다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.remove(null, "a"));
              System.out.println(StringUtils.remove("", "a"));
              System.out.println(StringUtils.remove("a", null));
              System.out.println(StringUtils.remove("a", ""));
              System.out.println(StringUtils.remove("queued", "ue"));
              System.out.println(StringUtils.remove("queued", "zz"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // a
      // a
      // qd
      // queued
      ```

      

- ## 15. delete

  - 이것도 lastIndexOfAnyBut 와 마찬가지로 존재하지 않는 메서드이다.

    

- ## 16.  여기서부터 이어서 작성