# 목차

1. [Collectors.toSet() 는 무엇인가??](#1-collectorstoset-는-무엇인가) <br/>
2. [어떻게 중복 데이터가 제거되는가??](#2-어떻게-중복-데이터가-제거되는가) <br/>
3. [Collectors.toSet() 의 사용법](#3-collectorstoset-의-사용법) <br/>

<br/>

# [자바, Java] Collectors.toSet() - 중복 제거

<br/>

> **프로그래밍 구현 시, 어떤 List 의 요소들 중에서 Stream을 통해 중복을 제거하는 기능을 구현할 때,** 
>
> **distinct() 도 있지만 Collectors.toSet() 을 통해서도 가능하다는 것을 알게되어서 간단히 정리해보고자 한다.**

<br/>

## 1. Collectors.toSet() 는 무엇인가??

- <code><strong>Stream 의 최종연산자인 collect() 를 통해 요소를 수집해서 반환 시, Set 자료구조 형식으로 수집하게끔 결정해주는 역할이다.</strong></code>
  
  - toList(), toMap() 도 있으나, 이건 나중에 필요할 때 써보고 정리해보고자 한다.
  
    

## 2. 어떻게 중복 데이터가 제거되는가??

- 먼저 Collectors.toSet() 의 실제 구현 형태를 살펴보자

  - ```java
    public static <T> Collector<T, ?, Set<T>> toSet() {
        return new CollectorImpl<>((Supplier<Set<T>>) HashSet::new, Set::add,
                                   (left, right) -> { left.addAll(right); return left; },
                                   CH_UNORDERED_ID);
    }
    ```

  - 좀 복잡해서 전부 설명하기는 좀 힘들지만, 결국 <code><strong>HashSet 자료구조 형태로 요소를 수집해서 반환한다.</strong></code>

    - 여기서 중복된 데이터가 자동적으로 걸러지게 된다.
    
      

## 3. Collectors.toSet() 의 사용법

- **개인적으로 프로그래밍을 짤 때, Stream 을 통해 중복을 제거할 용도로 많이 사용중이다.**

- 예제

  - ```java
    import java.util.*;
    import java.util.stream.Collectors;
    
    class Scratch {
        int a;
        int b;
        
        public Scratch(int a, int b) {
            this.a = a;
            this.b = b;
        }
        
        public static void main(String[] args) {
            int[] arr = {10, 5, 3, 2, 4, 1, 2, 4, 6, 7, 3, 3};
            Set<Integer> collect1 = Arrays.stream(arr).boxed().collect(Collectors.toSet());
            System.out.println("int : " + collect1);
            System.out.println();
            
            String[] strArr = {"cc", "bb", "cc", "bb", "cc", "aa"};
            Set<String> collect2 = Arrays.stream(strArr).collect(Collectors.toSet());
            System.out.println("String : " + collect2);
            System.out.println();
            
            Scratch[] scArr = {new Scratch(10, 20), new Scratch(30, 20), new Scratch(10, 20)};
            Set<Scratch> collect3 = Arrays.stream(scArr).collect(Collectors.toSet());
            System.out.println("Scratch size : " + collect3.size());
            System.out.println();
            collect3.forEach(scratch -> System.out.println("Scratch a, b : " + scratch.a + " " + scratch.b));
        }
        
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Scratch scratch = (Scratch) o;
            return a == scratch.a && b == scratch.b;
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(a, b);
        }
    }
    
    // 출력 결과
    // int : [1, 2, 3, 4, 5, 6, 7, 10]
    // 
    // String : [cc, bb, aa]
    // 
    // Scratch size : 2
    // 
    // Scratch a, b : 30 20
    // Scratch a, b : 10 20
    ```

    1. int 타입은 바로 중복된 데이터가 걸러져서 출력되는 모습이다.
    2. String 클래스타입은 이미 equals() 와 hashcode() 가 오버라이딩 되어있기에 중복된 문자열이 걸러지는 모습이다.
    3. Scratch 클래스도 String 과 마찬가지로 equals() 와 hashcode() 를 오버라이딩 해줘서 중복된 인스턴스 데이터가 들어있는 객체를 걸러내는 모습이다.

  