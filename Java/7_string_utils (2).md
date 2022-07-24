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
16. [replace(final String text, final String searchString, final String replacement)](#16-replacefinal-string-text-final-string-searchstring-final-string-replacement) <br/>
17. [overlay(final String str, String overlay, int start, int end)](#17-overlayfinal-string-str-string-overlay-int-start-int-end) <br/>
18. [upperCase(final String str)](#18-uppercasefinal-string-str) <br/>
19. [lowerCase(final String str)](#19-lowercasefinal-string-str) <br/>
20. [20. swapCase(final String str)](#20-swapcasefinal-string-str) <br/>
21. [capitalize(final String str)](#21-capitalizefinal-string-str) <br/>
22. [uncapitalize(final String str)](#22-uncapitalizefinal-string-str) <br/>
23. [reverse(final String str)](#23-reversefinal-string-str) <br/>
24. [reverseDelimited(final String str, final char separatorChar)](#24-reversedelimitedfinal-string-str-final-char-separatorchar) <br/>

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

    

- ## 16.  replace(final String text, final String searchString, final String replacement)

  - <code><strong>text 문자열에서 searchString 을 replacement 로 대체한다.</strong></code>

    - 이 메서드에 전달된 null 참조는 작동하지 않는다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.replace(null, " ", ""));
              System.out.println(StringUtils.replace("", "", "a"));
              System.out.println(StringUtils.replace("any", null, "z"));
              System.out.println(StringUtils.replace("any", "a", null));
              System.out.println(StringUtils.replace("any", "", "z"));
              System.out.println(StringUtils.replace("aba", "a", null));
              System.out.println(StringUtils.replace("aba", "a", ""));
              System.out.println(StringUtils.replace("aba", "a", "z"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // any
      // any
      // any
      // aba
      // b
      // zbz
      ```

      

- ## 17. overlay(final String str, String overlay, int start, int end)

  - <code><strong>str 문자열에서 start 부터 end 까지 overlay 로 변경한다.</strong></code>

    - 문자열의 일부를 다른 문자열로 오버레이한다.
      - null 문자열 입력은 null 을 반환한다.
      - 음수 인덱스는 0으로 처리된다.
      - 문자열 길이보다 큰 인덱스는 문자열 길이로 처리된다. 
      - 시작 인덱스는 항상 두 인덱스 중 작은 값이다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.overlay(null, "a", 0, 0));
              System.out.println(StringUtils.overlay("", "abc", 0, 0));
              System.out.println(StringUtils.overlay("abcdef", null, 2, 4));
              System.out.println(StringUtils.overlay("abcdef", "", 2, 4));
              System.out.println(StringUtils.overlay("abcdef", "", 4, 2));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", 2, 4));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", 4, 2));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", -1, 4));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", 2, 8));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", -2, -3));
              System.out.println(StringUtils.overlay("abcdef", "zzzz", 8, 10));
          }
      }
      
      // 출력 결과
      // null
      // abc
      // abef
      // abef
      // abef
      // abzzzzef
      // abzzzzef
      // zzzzef
      // abzzzz
      // zzzzabcdef
      // abcdefzzzz
      ```

      

- ## 18. upperCase(final String str)

  - <code><strong>문자열을 대문자로 변환한다.</strong></code>

    - null 입력 문자열은 null 을 반환한다.

  - 실제 메서드 구현 모습

    - ```java
      public static String upperCase(final String str) {
          if (str == null) {
              return null;
          }
          return str.toUpperCase();
      }
      ```

    - null 에 대한 대처만 추가하고 String 의 toUpperCase 메서드를 쓰는 모습이다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.upperCase(null));
              System.out.println(StringUtils.upperCase(""));
              System.out.println(StringUtils.upperCase("aBc"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // ABC
      ```

      

- ## 19. lowerCase(final String str)

  - <code><strong>문자열을 소문자로 변환한다.</strong></code>

    - null 입력 문자열은 null 을 반환한다.

  - 실제 메서드 구현 모습

    - ```java
      public static String lowerCase(final String str) {
          if (str == null) {
              return null;
          }
          return str.toLowerCase();
      }
      ```

    - 이것도 null 에 대한 대처만 추가하고 String 의 toLowerCase 메서드를 쓰는 모습이다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.lowerCase(null));
              System.out.println(StringUtils.lowerCase(""));
              System.out.println(StringUtils.lowerCase("aBc"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // abc
      ```

      

- ## 20. swapCase(final String str)

  - <code><strong>대문자와 제목을 소문자로, 소문자를 대문자로 바꾸는 문자열의 대소문자를 바꾼다.</strong></code>

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.swapCase(null));
              System.out.println(StringUtils.swapCase(""));
              System.out.println(StringUtils.swapCase("The dog has a BONE"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // tHE DOG HAS A bone
      ```

      

- ## 21. capitalize(final String str)

  - <code><strong>문자열의 가장 첫번째 문자를 대문자로 바꿔준다.</strong></code>

    - null 입력 문자열은 null 을 반환한다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.capitalize(null));
              System.out.println(StringUtils.capitalize(""));
              System.out.println(StringUtils.capitalize("cat"));
              System.out.println(StringUtils.capitalize("cAt"));
              System.out.println(StringUtils.capitalize("'cat'"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // Cat
      // CAt
      // 'cat'
      ```

      

- ## 22. uncapitalize(final String str)

  - <code><strong>문자열의 가장 첫번째 문자를 소문자로 바꿔준다.</strong></code>

    - null 입력 문자열은 null 을 반환한다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.uncapitalize(null));
              System.out.println(StringUtils.uncapitalize(""));
              System.out.println(StringUtils.uncapitalize("cat"));
              System.out.println(StringUtils.uncapitalize("Cat"));
              System.out.println(StringUtils.uncapitalize("CAT"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // cat
      // cat
      // cAT
      ```

      

- ## 23. reverse(final String str)

  - <code><strong>문자열을 뒤집는다.</strong></code>

    - null 문자열은 null 을 반환한다.

  - 실제 메서드 구현 모습

    - ```java
      public static String reverse(final String str) {
          if (str == null) {
              return null;
          }
          return new StringBuilder(str).reverse().toString();
      }
      ```

    - null 만 대처한 뒤 StringBuilder 의 reverse() 메서드를 쓰는 모습이다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.reverse(null));
              System.out.println(StringUtils.reverse(""));
              System.out.println(StringUtils.reverse("bat"));
          }
      }
      
      // 출력 결과
      // null
      // 
      // tab
      ```

      

- ## 24. reverseDelimited(final String str, final char separatorChar)

  - <code><strong>특정 문자로 구분된 문자열을 뒤집는다.</strong></code>

    - 구분 기호 사이의 문자열은 반전되지 않는다.

  - 실제 메서드 구현 모습

    - ```java
      public static String reverseDelimited(final String str, final char separatorChar) {
          if (str == null) {
              return null;
          }
          // could implement manually, but simple way is to reuse other,
          // probably slower, methods.
          final String[] strs = split(str, separatorChar);
          ArrayUtils.reverse(strs);
          return join(strs, separatorChar);
      }
      ```

    - null 에 대한 대처를 한 후, split 으로 나눠서 배열 순서를 뒤집는 모습이 보인다.

  - 예제

    - ```java
      class Scratch {
          public static void main(String[] args) {
              System.out.println(StringUtils.reverseDelimited(null, 'x'));
              System.out.println(StringUtils.reverseDelimited("", '.'));
              System.out.println(StringUtils.reverseDelimited("a.b.c", 'b'));
              System.out.println(StringUtils.reverseDelimited("a.b.c", 'x'));
              System.out.println(StringUtils.reverseDelimited("a.b.c", '.'));
          }
      }
      
      // 출력 결과
      // null
      // 
      // .cba.
      // a.b.c
      // c.b.a
      ```

      

      <br/>

      

- 사실 이 외에도 더 많은 String 클래스와 같은 메서드들이 존재하지만 이 정도만 해도 굉장히 유용하다 생각한다.

- null 에 대한 안정성이 생기는 것이니 앞으로 String 메서드보다 StringUtils 메서드를 더 활용해보는 것이 좋겠다.