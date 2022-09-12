# 목차

1. [전략 패턴이란??](#1-전략-패턴이란) <br/>
2. [전략 패턴을 쓰는 이유](#2-전략-패턴을-쓰는-이유) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 전략 패턴 (Strategy Pattern)

<br/>

> **nextstep 에서 배웠을 때도, 알고리즘 공부할 때 sort 메서드 쓸때도**
>
> **어떤 함수형 인터페이스를 매개변수로 전달해서 동작 방식을 변경하는 기법을 자주 보았다.**
>
> **이러한 것들을 검색을 통해 전략 패턴이라것을 알게 되었고,**
>
> **이번에 그 패턴에 대해 공부한 내용을 간단히 정리해 보고자 한다.**

<br/>

## 1. 전략 패턴이란??

- **여러 유사한 알고리즘을 캡슐화해서 객체의 행위를 동적으로 변경 가능하게 만드는 패턴이다.**
  - 상태 패턴과 굉장히 유사해 보이지만, 그 차이는 나중에 정리해보고자 한다.

<br/>

## 2. 전략 패턴을 쓰는 이유

1. **가독성, 유지보수성이 좋아진다.**

   - **만약 전략 패턴을 쓰지 않는다면,**

     1. 커피를 만드는 기계를 설계한다.
     2. 종류는 아메리카노, 카페라뗴, 카페모카 3가지이다.
     3. 여기서 온도 3가지 (따뜻한, 차가운, 미지근한) 을 추가한다.
        - 따뜻한 아메리카노, 아이스 아메리카노, 미지근한 아메리카노, 따뜻한 카페라떼......
     4. 여기서 샷 추가 3가지(1샷, 2샷, 3샷) 를 추가한다.
        - 따뜻한 아메리카노 1샷, 따뜻한 아메리카노 2샷, 따뜻한 아메리카노 3샷, 아이스 아메리카노 2샷 ......
     5. 종류(객체)가 기하급수적으로 늘어난다.
        - 3 x 3 x 3 = 27
        - 이 정도의 종류만 추가해도 **클래스가 27개**로 늘어나버린다.
     6. 어떤 종류의 커피가 필요할 때, 27개의 객체중 하나를 찾아서 써야한다.

   - **만약 전략 패턴을 적용한다면,**

     - 종류, 온도, 샷 추가를 필요한 커피가 생길 때마다 그에 맞는 객체를 하나씩 꺼내서 조합한다.

     - 필요할 때마다 조합이 가능하니, 각 종류를 담당할 **인터페이스 3개, 클래스는 총 9개가 필요**하다.

     - 객체의 숫자와 복잡성이 훨씬 줄어들고, 운영 방법도 훨씬 유연해진다.

       <br/>

2. **OCP (개방폐쇄의 원칙)** : 변경엔 닫혀있고 확장엔 열려있는 객체지향 원칙이 실현된다.

   - 다른 행동 및 전략이 추가된다고 해도 기존의 코드는 변경하지 않고 확장이 가능하다.

     <br/>

3. **상속 대신 위임을 사용할 수 있다.**

   - **상속의 단점**
     1. 상속은 단 한 개의 클래스만 상속할 수 있기에, 나중에 진짜 상속해야할 클래스가 생길때 상속을 못한다.
     2. 상위 클래스가 바뀌면 하위 클래스들도 모조리 영향을 받기에, 종속성 측면에서의 단점이 있다.
   - 위임의 장점
     1. 해당 Context 클래스가 변경되더라도, 전략 영역은 전혀 영향을 받지 않는다.
        - 밑의 예제 참고

<br/>