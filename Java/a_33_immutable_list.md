# 목차

1. [문제 발생 - List 자료구조로 add 하는 코드에서 예외가 발생](#1-문제-발생---list-자료구조로-add-하는-코드에서-예외가-발생) <br/>
2. [불변 리스트를 반환하는 메서드들](#2-불변-리스트를-반환하는-메서드들) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 1주차 - List로 add() 했을 때 UnsupportedOperationException 발생하는 이유

<br/>

> **문제 중에서 입력 값이 List 타입으로 들어오는 문제가 있다.**
>
> **그 들어온 값들을 이용해서 도메인에서 데이터를 가공하려고 하는데,**
>
> **뭔 일인지 UnsupportedOperationException 예외가 발생했다.**
>
> **그 원인과 해결법을 찾아보자.**

<br/>

## 1. 문제 발생 - List 자료구조로 add 하는 코드에서 예외가 발생

이 문제는 이번에 프리코스에서 구현한 로직으로 예시를 들면, 이번 프리코스의 유출이 좀 많이 될 수도 있기 때문에 그냥 직접 간단히 작성한 코드들로 예시를 들어보겠다.

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4);
        list.add(5); // 에러
        System.out.println(list);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// Exception in thread "main" java.lang.UnsupportedOperationException
// 	at java.base/java.util.ImmutableCollections.uoe(ImmutableCollections.java:71)
//	at java.base/java.util.ImmutableCollections$AbstractImmutableCollection.add(ImmutableCollections.java:75)
//	at Scratch.run(scratch.java:23)
//	at Scratch.main(scratch.java:28)
```

<br/>

지금 이 예시에서 발생한 예외와 같은 예외이다. 이러한 예외가 발생한 이유는, List.of() 메서드는 변경할 수 없는 즉, 불변의 리스트를 반환하기 때문이다. 그래서 add() 메서드를 통해 해당 리스트에 요소를 추가하려고 할 때 예외가 발생하는 것이다.



사실 난 원래 List.of() 가 불변 컬렉션을 반환하는 것을 알고 있었다. 그리고 이 예시에선 왜 에러가 나는지 쉽게 파악할 수 있다. 하지만 프리코스 문제를 풀 당시엔, **'불변 리스트를 입력 값으로 전달하는 테스트 코드'**와 **'도메인 객체'**가 분리가 되어있다. 그래서 도메인으로 들어오는 리스트가 불변 리스트인걸 까먹고 한참 해맨 기억이 있다.



이참에 불변 리스트를 반환하는 메서드들과 그 메서드들의 로직들을 간단히 살펴 보면서, 왜 불변인지 알아보고자 한다.

<br/>

## 2. 불변 리스트를 반환하는 메서드들