# 목차

1. [sorted() 란 무엇인가??](#1-sorted-란-무엇인가) <br/>
2. [sorted() 의 사용법](#2-sorted-의-사용법) <br/>
    1. [객체 비교](#2-1-객체-비교) <br/>
    2. [커스텀(자신이 정한) 정렬 키로 비교하기](#2-2-커스텀자신이-정한-정렬-키로-비교하기) <br/>

<br/>

# [자바, Java] Stream 의 중간연산자 sorted()

<br/>

> 알고리즘과 자료구조를 공부할 때, 정렬의 기능을 많이 써야하는 알고리즘 문제들 특성상 
>
> Stream 의 sorted() 를 굉장히 자주 사용하게 된다.
>
> 알고리즘 문제를 풀면서 sorted() 에 대해 알아보면서, 그리고 직접 써보면서 알아낸 것들을 정리해보고자 한다.

<br/>

## 1. sorted() 란 무엇인가??

- <code><strong>Stream 을 정렬할 때 사용하는 Stream 의 중간연산자이며, 오름차순이 아닌 다른 방식으로 정렬할 땐 Comparator 또는 Comparable 을 사용해서 정렬 형식을 지정해준다.</strong></code>
  - 속도면에서는 본인이 직접 반복문을 통해서 정렬작업을 하는 것보다 느릴 수 있다. 
    - 하지만 속도의 성능이 제일 중요한 상황이 아닌 이상, 하드웨어가 많이 발전하고 유지보수의 중요성이 높아지고 있는 현재엔 유용한 기능이라고 생각한다.

    - 왜냐면 Stream 의 가장 큰 장점은 가독성 즉, 유지보수성이라고 생각하기 때문이다.

      

## 2. sorted() 의 사용법

- 이제 지금까지의 공부와 경험에 기반해서 몇 가지 사용법을 정리해볼 것이다.


### 2-1. 객체 비교

- 비교 대상이 되는 객체에 Comparable 인터페이스를 구현해서 직접 compareTo() 를 오버라이딩 해놓으면, **sorted() 메서드가 알아서 compareTo() 메서드를 사용해서 비교한다.**

  - 이것은 비단 stream 의 sorted() 뿐만이 아닌 **Arrays.sort(), Collections.sort()** 에도 해당되는 사항이다.

- 예제

  - 이 예제는 객체의 name 문자열에서 <code><strong>2번째 문자</strong></code>를 기준으로 비교해서 객체를 오름차순 정렬을 하는 것이다.

  - ```java
    class Scratch {
        public static void main(String[] args) {
            Scratch scratch = new Scratch();
            // String[] 배열, 비교 기준이 될 2번째 문자를 지정하기 위한 인덱스 1을 인자로 전달
            String[] strArr = scratch.getSortedStrArr(new String[]{"aba", "bca", "cac"}, 1); 
            System.out.println(Arrays.toString(strArr));
        }
        
        public String[] getSortedStrArr(String[] strings, int n) {
            return Arrays.stream(strings).sorted() // 먼저 문자열을 오름차순으로 정렬
                // 각 문자열들을 문자열과 지정된 n번째 문자를 인자로 전달해서 Str 클래스로 치환한다. 
                    .map(str -> new Str(str, str.charAt(n))) 
                    .sorted() // Str 클래스에 정의된 compareTo 를 통해 Str 클래스들을 정렬한다.
                    .map(Str::getName).toArray(String[]::new); // Str 클래스들을 getName() 메서드를 사용해서 다시 문자열들로 치환한다.
        }
    }
    
    class Str implements Comparable<Str> {
        String name;
        char ch;
        
        public Str(String name, char ch) {
            this.name = name;
            this.ch = ch;
        }
        
        String getName() {
            return name;
        }
        
        @Override
        public int compareTo(Str o) {
            return this.ch - o.ch; // 문자열 name 의 n번째 문자를 기준으로 오름차순 정렬
        }
    }
    
    // 출력 결과
    // [cac, aba, bca]
    ```

  - 출력 결과를 보면 **문자열의 2번째 문자**들을 기준으로 오름차순 정렬이 되어있는 모습을 볼 수 있다.

    

### 2-2. 커스텀(자신이 정한) 정렬 키로 비교하기

- 자신이 비교하고 싶은 값(정렬 키)을 마음대로 정하고, 그 해당 키로 비교해서 정렬하는 기능이다.

  - Comparator.comparing() 메서드는 함수 내부에서 반환받는 값이, 비교할 때의 기준값이 된다.
  - 이 예시는 각 문자열의 두번째 문자로 비교해서 정렬하는 코드다.

- 예시

  - ```java
    class Scratch {
        public static void main(String[] args) {
            Scratch scratch = new Scratch();
            String[] st = {"aba", "bca", "cac"};
            String[] strings = Arrays.stream(st)
                    .sorted() // 오름차순 정렬
                    .sorted(Comparator.comparing(s -> s.charAt(1))) // 2번째 문자를 기준으로 비교해서 오름차순 정렬
                    .toArray(String[]::new); // String[] 배열로 모아서 반환
            System.out.println(Arrays.toString(strings));
        }
    }
    
    // 출력 결과
    // [cac, aba, bca]
    ```

    