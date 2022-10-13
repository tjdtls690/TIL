# 목차

1. [toString 의 용도는 무엇일까??](#1-tostring-의-용도는-무엇일까) <br/>
2. [왜 View 출력 로직에 도움을 주는 용도는 안될까??](#2-왜-view-출력-로직에-도움을-주는-용도는-안될까) <br/>
3. [피드백 받기 전과 후의 코드 비교](#3-피드백-받기-전과-후의-코드-비교) <br/>

<br/>

# [자바, Java] toString 의 용도 - 단순 로그?? or View 로직도 가능??

<br/>

> **nextstep 미션을 진행하면서 프로그램을 구현하는 중에,**
>
> **객체에서 toString 을 통해 View 에 출력해야 할 데이터를 반환해주면 코드가 훨씬 깔끔해진다는 생각을 했다.**
>
> **실제로도 피드백을 받기 전, 그렇게 구현했을 때 코드가 굉장히 깔끔했다.**
>
> **도메인 로직에서도, View 로직에서도 출력을 위한 코드가 굉장히 짧아졌다.**
>
> **하지만 곧 피드백을 통해 도메인 객체의 toString 을 View 로직에 관련한 메서드로 사용할 수 없게 되었다.**
>
> **이번 피드백을 통해 공부한 내용을 간단히 정리해 보고자 한다.**

<br/>

## 1. toString 의 용도는 무엇일까??

**오로지 디버깅을 위해 사용되는 재정의 메서드**이다. 다들 알다시피 재정의를 하지 않은 채 바로 출력을 해보면,

```java
class Scratch {
    void run () throws IOException {
        System.out.println(this);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// Scratch@59a6e353
```

<br/>

위의 예시와 같이 해당 객체의 주소값이 출력된다. 이는 개발을 하는 **개발자에게 전혀 쓸모없는 정보**이다. toString() 은 오라클에서 재정의 해서 디버깅할 때 쓰라고 제공해준 메서드라고 보면 된다. 쓸만한 정보 즉, 객체의 상태 값들을 제대로 출력해서 디버깅에 도움을 주는 메서드이다.

```java
class Scratch {
    int a = 2;
    
    void run () throws IOException {
        System.out.println(this);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
    
    @Override
    public String toString() {
        return "Scratch{" +
                "a=" + a +
                '}';
    }
}

// 출력 결과
// Scratch{a=2}
```

<br/>

## 2. 왜 View 출력 로직에 도움을 주는 용도는 안될까??

정말 **'toString() 메서드는 디버깅을 위해 제공된 메서드이니 다른 용도로 사용하면 안된다.'** 이러한 이유로만 알고 있으면 되는걸까?? 진짜 코드가 너무 깔끔해진다는 장점을 포기할 만큼, 확실한 납득을 하기위해 좀 더 명확한 이유가 필요했다. View 로직이 여기에 있으면 안된다는 피드백을 보고 처음에 들었던 의문은, '어차피 System.out.println() 을 통해 출력해주는건 View 레벨에서 해주고 있으니, toString 을 통해 자기 자신을 표현한 값을 반환만 해주는건 View 레벨의 로직과 직접적인 연관은 없지 않나??' 라는 의문이었다.



그래서 리뷰어님과의 대화 및 직접 검색을 통해 공부를 했다. 결론은 **직접적이든, 간접적이든 아주 조금이라도 서로 다른 계층간에 연관성이 있는 것은 좋지 않는다는 것**이다. 만약 요구사항이 '출력을 이런 식이 아니라 저런 식으로 나오게끔 변경해주세요.' 라고 변경된다면, **결국 손을 대야할 클래스가 View 레벨뿐만이 아닌 도메인 객체도 포함이 되어버린다.**



지금 돌이켜 생각해보면, **요구사항의 변경은 어디가 어떻게 변경이 되는지 그 누구도 예측할 수 없다는 사실을 너무 과소평가**했던 부분이 있는 것 같다. 물론 지금의 nextstep 의 미션과 같이 규모가 작은 프로그래밍을 할 땐 느껴지지 않을지 몰라도, 나중에 실무에서의 환경을 생각해 봤을 땐 지금부터 확실한 분리를 습관화해야겠다는 경각심이 들었다.

<br/>

## 3. 피드백 받기 전과 후의 코드 비교

아래 예시를 보자.

```java
public class LottoNumber implements Comparable<LottoNumber> {
    private final int lottoNumber;
    
    ... 생략

    @Override
    public String toString() {
        return String.valueOf(lottoNumber);
    }
}



public class LottoTicket {
    private final List<LottoNumber> lottoTicket;
    
    ... 생략
    
    @Override
    public String toString() {
        return lottoTicket.stream()
                .map(LottoNumber::toString)
                .collect(Collectors.joining(", ", "[", "]"));
    }
}



public class LottoTickets {
    private final List<LottoTicket> issueLottoTickets;
    
    ... 생략
    
    @Override
    public String toString() {
        return issueLottoTickets.stream()
                .map(LottoTicket::toString)
                .collect(Collectors.joining("\n"));
    }
}


public class ResultView {
    ... 생략
    
    public static void purchasedLottoNumbersPrint(LottoTickets lottoTickets, PaymentPrice paymentPrice) {
        System.out.printf(PURCHASED_LOTTO_NUMBER_PRINT_FORM, paymentPrice.numberOfTickets());
        System.out.println(lottoTickets);
        System.out.println();
    }
}
```

이 예시에선 toString() 메서드를 통해 LottoNumber -> LottoTicket -> LottoTickets 방향으로 가공 과정을 거치고, ResultView 에서 객체의 toString() 를 호출함으로써 출력을 완성한 모습이다. 굉장히 깔끔하다. 이렇게 구현하고 나 자신에게 굉장히 뿌듯해 했던 기억이 있다...ㅋ 하지만 곧 피드백을 받고 도메인의 toString() 메서드가 하던 짓들을 전부 ResultView 로 추출하는 작업을 하게 되었다.

<br/>

```java
public class ResultView {
    public static void purchasedLottoNumbersPrint(LottoTickets lottoTickets, PaymentPrice paymentPrice) {
        System.out.printf(PURCHASED_LOTTO_NUMBER_PRINT_FORM, paymentPrice.numberOfTickets());
        lottoTickets.getLottoTickets().stream()
                .map(ResultView::lottoTicketPrintFormat)
                .forEach(System.out::println);
        System.out.println();
    }

    private static String lottoTicketPrintFormat(LottoTicket lottoTicket) {
        return lottoTicket.getLottoTicket().stream()
                .map(LottoNumber::getLottoNumber)
                .map(String::valueOf)
                .collect(Collectors.joining(", ", "[", "]"));
    }
}
```

도메인 객체의 toString 을 전부 지우고 View 레벨의 출력 로직이 더 확대된 모습이다. Stream 을 써서 그런진 모르겠지만, 이것도 꽤 깔끔해 보인다. 물론 Stream 이 아닌 for문을 통해 여러 메서드로 더 나누면서 구현을 했다면, 훨씬 더 코드량이 많아졌을지도 모른다. 하지만 각 계층과 각 객체가 맡은 책임은 지키는 것이, 유지보수 측면에서 유리하다는 것을 잊지 말자.