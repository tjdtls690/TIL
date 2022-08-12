# 목차

1. [joining() 메서드는 무엇일까??](#1-joining-메서드는-무엇일까) <br/>
2. [Collectors.joining() 의 실제 구현 모습](#2-collectorsjoining-의-실제-구현-모습) <br/>
3. [joining() 사용법](#3-joining-사용법) <br/>
    1. [문자열(String)의 문자들을 역정렬해서 반환](#3-1-문자열string의-문자들을-역정렬해서-반환) <br/>
    2. [접두사와 접미사를 붙여서 반환](#3-2-접두사와-접미사를-붙여서-반환) <br/>
    3. [3-3. String 뿐만이 아닌 CharSequence 인터페이스를 구현한 클래스 타입의 배열은 전부 사용 가능](#3-3-string-뿐만이-아닌-charsequence-인터페이스를-구현한-클래스-타입의-배열은-전부-사용-가능---stringbuilder-stringbuffer-string-등등) <br/>
    4. [객체의 toString 을 이용해서 결합](#3-4-객체의-tostring-을-이용해서-결합) <br/>

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

    


## 2. Collectors.joining() 의 실제 구현 모습

- ```java
  public static Collector<CharSequence, ?, String> joining() {
      return new CollectorImpl<CharSequence, StringBuilder, String>(
          StringBuilder::new, StringBuilder::append,
          (r1, r2) -> { r1.append(r2); return r1; },
          StringBuilder::toString, CH_NOID);
  }
  ```

- 위 구현 모습을 최대한 풀어서 이해해 보자면,

  1. StringBuilder 를 생성한다.
  2. StringBuilder 로 모든 요소를 append 한다.
  3. append 한 StringBuilder 를 String 으로 반환한다.



## 3. joining() 사용법

### 3-1. 문자열(String)의 문자들을 역정렬해서 반환

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

    

### 3-2. 접두사와 접미사를 붙여서 반환

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

        

### 3-3. String 뿐만이 아닌 CharSequence 인터페이스를 구현한 클래스 타입의 배열은 전부 사용 가능 - StringBuilder, StringBuffer, String 등등

- 예시

  - ```java
    class Scratch {
        public static void main(String[] args) {
            StringBuilder[] s = {new StringBuilder("aa"), new StringBuilder("cc"), new StringBuilder("bb")};
            String collect = Stream.of(s).collect(Collectors.joining());
            System.out.println(collect);
        }
    }
    
    // 출력 결과
    // aaccbb
    
    class Scratch {
        public static void main(String[] args) {
            StringBuffer[] s = {new StringBuffer("aa"), new StringBuffer("cc"), new StringBuffer("bb")};
            String collect = Stream.of(s).collect(Collectors.joining());
            System.out.println(collect);
        }
    }
    
    // 출력 결과
    // aaccbb
    ```

  - **StringBuilder, StringBuffer 의 배열을 만들고 바로 joining 메서드를 사용해서 이어붙이고, String 으로 반환한다.**

    

- 근데 사실 CharSequence 구현체를 이어붙여서 String으로 반환하는 방법은 **String.join() 메서드를 통해 더 쉽게 이어붙일 수 있다.**

  - 예시

    - ```java
      class Scratch {
          public static void main(String[] args) {
              StringBuilder[] s = {new StringBuilder("aa"), new StringBuilder("cc"), new StringBuilder("bb")};
              String collect = String.join("", s);
              System.out.println(collect);
          }
      }
      
      // 출력 결과
      // aaccbb
      
      class Scratch {
          public static void main(String[] args) {
              StringBuffer[] s = {new StringBuffer("aa"), new StringBuffer("cc"), new StringBuffer("bb")};
              String collect = String.join("", s);
              System.out.println(collect);
          }
      }
      
      // 출력 결과
      // aaccbb
      ```

      

### 3-4. 객체의 toString 을 이용해서 결합

- 예시

  - ```java
    class Scratch implements Comparable<Scratch> {
        String name;
        int age;
        
        public Scratch(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public static void main(String[] args) {
            Scratch jun = new Scratch("jun", 29);
            Scratch sin = new Scratch("sin", 23);
            Scratch moon = new Scratch("moon", 25);
            Scratch[] scArr = {jun, sin, moon};
        
            String collect = Stream.of(scArr)
                    .sorted(Comparator.reverseOrder()) // 나이를 기준으로 내림차순 정렬을 한다.
                    .map(Object::toString) // 객체의 toString 값을 가져온다.
                    .collect(Collectors.joining("\n")); // 개행문자로 joining 한다.
            System.out.println(collect);
        }
        
        @Override
        public int compareTo(Scratch o) {
            return this.age - o.age; // 나이순으로 오름차순 정렬
        }
        
        @Override
        public String toString() {
            return "Scratch{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    
    // 출력 결과
    // Scratch{name='jun', age=29}
    // Scratch{name='moon', age=25}
    // Scratch{name='sin', age=23}
    ```

    