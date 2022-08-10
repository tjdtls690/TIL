# 목차

1. [joining() 메서드는 무엇일까??](#1-joining-메서드는-무엇일까) <br/>
2. [joining() 사용법](#2-joining-사용법) <br/>
    1. [문자열(String)의 문자들을 역정렬해서 반환](#2-1-문자열string의-문자들을-역정렬해서-반환) <br/>
    2. [접두사와 접미사를 붙여서 반환](#2-2-접두사와-접미사를-붙여서-반환) <br/>

<br/>

# [자바, Java] Stream - Collectors.joining()

<br/>

> **프로그래머스를 통해 알고리즘 문제를 풀었는데, 내가 for문으로 String 을 이어 붙여서 반환한 것과 다르게,**
>
> **다른 분의 풀이 방법에서 Stream 의 Collectors.joining() 로 쉽게 답을 반환하는 모습이 보였다.**
>
> **toList() 같은 컬렉션 반환 방법만 알고있었지, String 배열을 이어붙여서 하나의 String 으로 반환시키는 기능은 몰랐다.**
>
> **이번에 joining 에대해 공부한 만큼 정리해보고자 한다.**

<br/>

## 1. joining() 메서드는 무엇일까??

- <code><strong>문자열 스트림의 모든 요소를 하나의 문자열로 연결해서 반환하는 역할을 가진다.</strong></code>

  - joining() 에 인자를 안주면 그대로 이어붙이고, 인자를 주면 요소들의 사이마다 인자를 넣어서 이어붙인다.

  

## 2. joining() 사용법

### 2-1. 문자열(String)의 문자들을 역정렬해서 반환

- 예시

  - ```java
    class Scratch {
        public static void main(String[] args) {
            String s = "dafaafq";
            String collect = Stream.of(s.split(""))
                    .sorted(Comparator.reverseOrder())
                    .collect(Collectors.joining());
            System.out.println(collect);
        }
    }
    
    // 출력 결과
    // qffdaaa
    ```

  - **문자열의 문자들을 역정렬 시킨 후, 다시 그대로 이어붙여서 반환하는 모습을 볼 수 있다.**

    

### 2-2. 접두사와 접미사를 붙여서 반환

- 예시

  - ```java
    class Scratch {
        public static void main(String[] args) {
            String s = "dafaafq";
            String collect = Stream.of(s.split(""))
                    .sorted(Comparator.reverseOrder())
                    .collect(Collectors.joining(", ", "[", "]"));
            System.out.println(collect);
        }
    }
    
    // 출력 결과
    // [q, f, f, d, a, a, a]
    ```

  - **문자열의 문자들을 역정렬 시킨 후, '쉼표 + 띄어쓰기'를 요소들 사이마다 넣어준 뒤, 접두어에 "[" 를, 접미어에 "]" 를 붙여서 String으로 반환한다.**

    - 참고로 이건 배열출력이 아닌 String 출력임을 주의하자.
      - 일부러 배열출력과 같은 형식의 출력 형식으로 만들어보았다.