# 목차

1. [indent depth 란??](#1-indent-depth-란) <br/>
2. [우테코에서는 왜 indent depth 를 줄이는 것을 지향할까??](#2-우테코에서는-왜-indent-depth-를-줄이는-것을-지향할까) <br/>
3. [indent depth 를 줄이면서 메서드 기능이 분리되는 과정](#3-indent-depth-를-줄이면서-메서드-기능이-분리되는-과정) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 2주차 - indent depth를 1로 줄이기 위해 노력하면서 얻은 경험

<br/>

> **우아한 테크 코스 5기의 프리코스 2주차 미션부터**
>
> **프로그래밍 요구 사항에 드디어 객체지향 생활 체조 원칙과 관련한 추가 요구사항들이 나오기 시작했다.**
>
> **'그 중 indent depth 는 2까지 허용한다'라는 요구사항을 적용하면서 얻은 경험에 대해 정리해보고자 한다.**

<br/>

## 1. indent depth 란??

indent depth 란 **'들여쓰기'**를 말한다. 즉, for 문을 한 번쓰면 그 안에서 indent 가 1 증가하고, 그 for 문 안에서 if문을 한 번 더 쓰면 indent 가 1 증가해서 총 indent depth 는 2가 된다. 실제로 프리코스에서 구현된 코드를 가져오면 문제 유출이 될 수 있으니, 즉석으로 indent depth 가 3인 예시를 간단히 구현해보자.

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        int sum = 0;
        
        for (Integer integer : list) {
            if (integer % 3 == 0) { // for문 블록 안의 코드 indent 1 증가
                if (integer % 6 != 0) { // if문 블록 안의 코드 indent 1 증가
                    integer += 2; // if문 블록 안의 코드 indent 1 증가 		=> 총 indent depth 3
                }
                integer += 1;
            }
            sum += integer;
        }
        System.out.println(sum);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 31
```

<br/>

## 2. 우테코에서는 왜 indent depth 를 줄이는 것을 지향할까??

indent 를 줄인다는 것은 **메서드를 분리한다는 말과 일맥상통**하다. 위의 1번 예시에서 보면 알겠지만, **indent depth 가 높다는 것은 그만큼 중첩된 코드가 많고 복잡하다는 뜻과 같다.**

어떤 프로그램을 구현한 후에, 요구사항이 변경되었다고 가정해보자. 어떤 특정 부분의 로직을 수정해야 하는데, 정말 수많은 로직들이 위의 예시처럼 하드 코딩이 난무한다고 생각해보자. 수정해야할 특정 부분을 찾을 자신이 있는가?? 찾았다면, 다른 코드들에게 영향을 최소한으로 주면서 리팩토링 할 자신이 있는가??

유지보수를 굉장히 중요시 여긴다면, 당연히 **이러한 가독성과 유지보수성이 쓰레기인 구현 방식은 '하드 코딩'으로 여기고 지양해야 할 것**이다.

<br/>

## 3. indent depth 를 줄이면서 메서드 기능이 분리되는 과정

먼저 위의 1번 예시에서 중첩 코드를 더 집어 넣어서 구현해보자.

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        int sum = 0;
        
        for (Integer integer : list) {
            if (integer % 3 == 0) { // indent depth 1
                if (integer % 6 != 0) { // indent depth 2
                    integer += 2; // indent depth 3
                }
                integer += 1;
            }
            sum += integer;
        }
        System.out.println(sum);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 31
```

<br/>

이제 저 더러운 indent depth 가 3인 블록 중첩 코드를, 메서드(기능) 분리를 통해 indent depth 1로 줄여볼 것이다. 먼저 맨 안쪽에 있는 indent depth 가 3인 코드부터 처리해 나가보자.

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        int sum = 0;
        
        for (Integer integer : list) {
            if (integer % 3 == 0) { // 3의 배수 필터 => indent depth 1
                integer = plusOne(integer); // indent depth 2
            }
            sum += integer;
        }
        System.out.println(sum);
    }
    
    private Integer plusOne(Integer integer) { // 3의 배수 중 6이 아니면 2, 6이면 1을 더한 값을 반환
        if (integer % 6 != 0) {
            integer += 1;
        }
        
        return integer + 1;
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// 31
```

<br/>

첫번째 if문 블록내에 해당하는 로직들을 plusOne() 이라는 메서드로 분리를 한 모습이다. 그냥 예시를 위해 막 구현한 것이라서, 메서드 명을 뭘로 할 지 몰라 1을 더한다는 의미로 이름을 정했다. 이로써 indent depth 가 1이 줄어들어서 indent depth 가 2로 줄었다.

다시 indent depth 가 2인 로직까지 기능 분리를 하여 indent depth 를 1로 줄여보겠다. 

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        int sum = 0;
        
        for (Integer integer : list) {
            sum = sum(sum, integer); // indent depth 1
        }
    
        System.out.println(sum);
    }
    
    private int sum(int sum, Integer integer) {
        if (integer % 3 == 0) {
            integer = plusOne(integer);
        }
        
        sum += integer;
        return sum;
    }
    
    private int sum(int sum, Integer integer) {
        if (integer % 3 == 0) {
            integer = plusOne(integer);
        }
        
        return sum + integer;
    }
    
    private Integer plusOne(Integer integer) {
        if (integer % 6 != 0) {
            integer += 1;
        }
        
        return integer + 1;
    }
}

// 출력 결과
// 31
```

<br/>

indent depth 가 2인 로직을 기능 분리하여 indent depth 가 1로 줄어든 모습을 볼 수 있다. 여기서 indent depth 가 1인 로직까지 메서드 분리하여 0으로 줄일 수도 있고, 각 if 문의 조건들까지도 메서드 추출을 할 수도 있고, 그 외에도 리팩토링을 할만한 부분은 아직 많이 보입니다. 하지만 예시는 여기까지만 구현하도록 하겠다. 여기까지 봤다면, 어차피 이해가 거의 다 되었으리라 생각하기 때문이다.

기능이 정상적으로 동작 되도록 구현을 했다고 해도, 앞으로의 요구사항은 어떻게 변할 지 모른다. 그러한 변화되는 요구사항에 따라 코드를 수정해나갈 때, 이러한 메서드가 잘 분리된 로직이라면 **특정 기능을 맡은 작은 규모의 메서드만 수정**을 해주면 된다. 이는 **유지보수에 있어서 엄청난 장점**일 것이다.

이러한 기능 분리는 메서드 분리에서 더 발전하면, **클래스 분리**까지도 갈 수 있다. 클래스 분리까지 나아간다면 **진정한 객체지향 설계**가 시작되는 것이라 생각한다.

사실 이렇게 아무런 의미 없이 막 구현 한 후, 메서드 분리를 한 것이라 진정한 가치가 잘 안느껴 질 수도 있다. 하지만 실제 의미있는 프로그램 로직의 기능 분리를 통해 **메서드가 작게 쪼개지고, 그로인해 코드의 가독성과 유지보수성이 증가하는 것**을 직접 체감하게 된다면, indent depth 를 줄이는 작업이 얼마나 귀중한 작업인지 알게 될 것이다.