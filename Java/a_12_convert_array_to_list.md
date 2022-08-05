# 목차

1. [반복문](#1-반복문) <br/>
2. [Arrays.asList()](#2-arraysaslist) <br/>
3. [Stream 의 최종연산자 - collect()](#3-stream-의-최종연산자---collect) <br/>

<br/>

# [자바, Java] int[] 배열을 List 로 변환시키는 방법

<br/>

> **1달 전쯤부턴 자바의 API 를 최대한 많이 활용하려고 노력한다.**
>
> **그래서 코드를 짤 때, 어떤 기능을 만들 때마다 내가 직접 알고리즘을 구현하는 것이 아닌** 
>
> **해당 기능의 API 존재 여부부터 알아본다.**
>
> **이번에도 int[] 배열을 List 로 변환시켜야 할일이 있어서 API 를 찾아보는 중에,**
>
> **Arrays.asList() 메서드를 알게 되었다.**
>
> **근데 int[] 배열을 asList() 메서드의 인자로 바로 넣어서 만드려니, 자꾸 요소타입이 int[] 로 잡히면서 에러가 뜬다.**
>
> **이유를 찾으려 공부하는 중에, 이유도 찾고 다른 더 좋은 방법들도 알게되어 정리해보고자 한다.**

<br/>

## 1. 반복문

- 가장 직관적이고 고전적이고 정석적인 방법이라 생각한다.

  - API 를 통해 간단히 구현하는 것보다 더 많은 코드를 쳐야하는 단점이 있다.
  - 하지만 속도면에선 가장 빠를 수 있다고 생각한다.
    - 왜냐면 최근에 알고리즘을 꾸준히 공부하는 중에, stream 을 사용하는 것보다 직접 알고리즘을 짜서 코딩하는 것이 더 빠른경우가 많았다.
    - 물론 stream 이 구현하기엔 훨씬 편하긴 하다.

- 예제

  - ```java
    class Scratch {
        public static void main(String[] args) {
            int[] tmp = {2, 6, 4, 8, 10};
            List<Integer> list = new ArrayList<>();
            
            for (int i : tmp) {
                list.add(i);
            }
            System.out.println(list);
        }
    }
    
    // 출력 결과
    // [2, 6, 4, 8, 10]
    ```

    

## 2. Arrays.asList()

- 이 방법은 주의할 점이, <code><strong>Arrays.asList()의 인자에는 int[] 가 아닌 Integer[] 로 변환 후 넣어줘야 한다.</strong></code>

  - 그래서 int[] 에서 Integer[] 로 처음에 변환하는 과정이 한 번 더 필요하다.
  - 변환하지 않는다면 List<int[]> 타입으로 반환을 받게 된다.

- **개인적으로 이유가 무엇일까 생각해봤다.**

  - 사실 List 의 요소에 기본타입은 들어가지 못한다는 것을 알고있다면, 당연한 부분이기도 하다.
    - 그래서 객체만이 List 의 요소타입이 될 수 있다는 사실을 알고있다면,
    - int 타입으로 리스트를 반환받으려할 때 왜 자꾸 int[] 로 객체화해서 지정되는지 이해가 되는 것 같다.
    - 기본타입 int 와는 다르게 int[] 배열은 주소값을 가진 하나의 객체와 같으니 말이다.

- 예제

  - ```java
    class Scratch {
        public static void main(String[] args) {
            int[] num = {1, 1, 3, 3, 5, 6, 6};
            List<Integer> integers = Arrays.asList(Arrays.stream(num).boxed().toArray(Integer[]::new));
            System.out.println(integers);
        }
    }
    
    // 출력 결과
    // [1, 1, 3, 3, 5, 6, 6]
    
    
    // 위의 코드를 좀 더 이해하기 쉽게 풀어서 써보자면
    
    class Scratch {
        public static void main(String[] args) {
            int[] num = {1, 1, 3, 3, 5, 6, 6};
            Integer[] integers1 = Arrays.stream(num).boxed().toArray(Integer[]::new);
            List<Integer> integers2 = Arrays.asList(integers1);
            System.out.println(integers2);
        }
    }
    ```

    

## 3. Stream 의 최종연산자 - collect()

- 사실 속도는 잘 모르겠지만 이 방법이 가장 깔끔한 방법이라고 생각한다.

  - 최종 연산자 collect() 메서드에 Collectors.toList() 를 인자로 전달하여 List 를 반환받는 방법이다.

- 예제

  - ```java
    class Scratch {
        public static void main(String[] args) {
            int[] num = {1, 1, 3, 3, 5, 6, 6};
            List<Integer> integers = Arrays.stream(num).boxed().collect(Collectors.toList());
            System.out.println(integers);
        }
    }
    
    // 출력 결과
    // [1, 1, 3, 3, 5, 6, 6]
    ```

    