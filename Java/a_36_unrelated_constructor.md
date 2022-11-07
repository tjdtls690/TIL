# 목차

1. [이 주제를 가져온 이유](#1-이-주제를-가져온-이유) <br/>
2. [내가 이 주제에 대해 긍정적인 견해를 가지게 된 이유](#2-내가-이-주제에-대해-긍정적인-견해를-가지게-된-이유) <br/>
3. [테스트만을 위한 생성자 구현의 장점](#3-테스트만을-위한-생성자-구현의-장점) <br/>
    1. [기본 생성자만 있는 경우](#1-기본-생성자만-있는-경우) <br/>
    2. [테스트만을 위한 생성자까지 구현한 경우](#2-테스트만을-위한-생성자까지-구현한-경우) <br/>
4. [테스트만을 위한 생성자 구현의 장점](#4-부정적인-의견) <br/>
    1. [비즈니스 로직의 이해를 방해할 수 있다](#1-비즈니스-로직의-이해를-방해할-수-있다) <br/>
    2. [비즈니스 로직에서 사용하면 안될 때, 제 3자가 사용할 수 있는 위험이 존재한다](#2-비즈니스-로직에서-사용하면-안될-때-제-3자가-사용할-수-있는-위험이-존재한다) <br/>
5. [마무리 하며...](#5-마무리-하며) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 2주차 - 테스트만을 위한 생성자 구현에 대한 견해

<br/>

## 1. 이 주제를 가져온 이유

난 테**스트만을 위한 생성자 구현을 굉장히 지향하는 편**이다. 그러나 내가 기존에 알던 것과는 달리, **이 주제에 대한 부정적인 평가도 보였다.** 그러한 반대 입장을 보고 다른 사람들은 어떻게 생각할 지 궁금했다.

이번 우테코 5기 프리코스부터는 우테코 프리코스에 참가하는 모든 사람들이 토론하고 질의응답을 할 수 있는 **커뮤니티(GitHub Discussioin)** 가 제공된다. 그 안에서도 여러가지 탭 목록이 있지만, 그 중 **'아고라'**라는 토론을 할 수 있는 게시판에 이 주제에 대한 **토론을 여는 글을 작성**했었다.

지금까지의 내 견해와 부정적인 의견, 그리고 아고라에서 토론을 나눈 내용까지 합해서 이 주제에 대해 정리를 해보고자 한다.

<br/>

## 2. 내가 이 주제에 대해 긍정적인 견해를 가지게 된 이유

'엘레강트 오브젝트'라는 서적의 생성자 파트를 보면,

> **하나의 클래스는 2~3개의 메서드와 5~10개의 생성자를 포함하는 것이 적당하다.**
> **응집도가 높고 견고한 클래스에는 적은 수의 메서드와 상대적으로 더 많은 수의 생성자가 존재한다.**
> **생성자가 많아지면 유연성이 향상된다.**

이처럼 생성자가 많아질수록, 클래스의 사용성이 좋아지는 것을 말하고 있다. 난 메서드는 최대한 줄이고 생성자는 늘려도 된다는 이 의견에 전적으로 동의한다. 

생성자는 setter 메서드처럼 객체의 값을 함부로 바꾸는 것이 아닌, 해당 객체의 초기화만을 책임지기 때문에, **생성자의 개수를 늘린다고 해도 Thread-safe 하다고 생각**한다. 또한 생성자가 많아질수록 **클래스 사용의 유연성이 증가**하여 사용자 입장에서도 굉장히 편리하게 사용할 수 있다는 것도 좋은점으로 본다.

실제로 난 테스트 코드를 작성할 때 이러한 유연성의 효과를 굉장히 많이 받는다는 것을 체감하고 있다. 그 부분에 대해 먼저 정리해보자.

<br/>

## 3. 테스트만을 위한 생성자 구현의 장점

이 주제의 문장만 봐도 알 수 있듯이, **'비즈니스 로직과는 상관 없이 오직 수월한 테스트만을 위한 생성자를 구현하는 것'**은 어떤 장점이 있을 까??

먼저 이 주제에 대한 간단한 예시를 구현해보겠다. (실제로 내가 프리코스 내에서 구현한 로직은 문제 유출의 위험이 있을 수 있기에 가져오지 않았다.)

<br/>

### 1. 기본 생성자만 있는 경우

일단 기본 생성자만 있을때의 비즈니스 로직과 테스트 구현을 한 모습이다.

```java
public class Speaker {
    private int volume;
    
    public void increaseVolume() {
        volume++;
    }
    
    public static Speaker findBiggestSpeaker(final List<Speaker> speakers) {
        int maxVolume = findMaxVolume(speakers);
        
        return speakers.stream()
                .filter(speaker -> speaker.volume == maxVolume)
                .findAny()
                .orElse(null);
                
    }
    
    private static int findMaxVolume(final List<Speaker> speakers) {
        return speakers.stream()
                .mapToInt(Speaker::volume)
                .max()
                .orElse(0);
    }
    
    private int volume() {
        return volume;
    }
}
```

```java
class SpeakerTest {
    @Test
    void isBiggerVolume() {
        Speaker speaker1 = new Speaker();
        speaker1.increaseVolume();
        speaker1.increaseVolume();
        
        Speaker speaker2 = new Speaker();
        speaker2.increaseVolume();
    
        Speaker speaker3 = new Speaker();
        speaker3.increaseVolume();
        speaker3.increaseVolume();
        speaker3.increaseVolume();
        
        assertThat(Speaker.findBiggestSpeaker(List.of(speaker1, speaker2, speaker3))).isEqualTo(speaker3);
    }
}
```

<br/>

SpeakerTest 에선 여러 스피커들 중에 볼륨이 가장 큰 스피커를 찾아내는 테스트를 하고 있다. 그런데 여기서 한 가지 위화감이 드는 부분이 있다. 테스트를 위해 각 스피커의 볼륨을 설정을 해줘야 한다. 그런데 예시에선 각 스피커의 볼륨을 increaseVolume() 메서드를 통해 일일이 볼륨을 올려주고 테스트를 진행하고 있다.

과연 이러한 테스트가 최선인 것일까?? 만약 테스트 해볼 스피커의 개수가 4개라면?? 아니면 5개?? 그 이상이라면?? 점점 더 테스트하기 곤란해지고, 테스트를 하기 귀찮아 질 것이다.

<br/>

### 2. 테스트만을 위한 생성자까지 구현한 경우

이제 생성자를 하나 더 추가해서 좀 더 효율적인 테스트를 진행해 보겠다.

```java
public class Speaker {
    private int volume;
    
    public Speaker() {
        this(0);
    }
    
    public Speaker(final int volume) { // 볼륨 값을 받아서 초기화하는 생성자 추가
        this.volume = volume;
    }
    
    public void increaseVolume() {
        volume++;
    }
    
    public static Speaker findBiggestSpeaker(final List<Speaker> speakers) {
        int maxVolume = findMaxVolume(speakers);
        
        return speakers.stream()
                .filter(speaker -> speaker.volume == maxVolume)
                .findAny()
                .orElse(null);
                
    }
    
    private static int findMaxVolume(final List<Speaker> speakers) {
        return speakers.stream()
                .mapToInt(Speaker::volume)
                .max()
                .orElse(0);
    }
    
    private int volume() {
        return volume;
    }
}
```

```java
class SpeakerTest {
    
    @Test
    void isBiggerVolume() {
        Speaker speaker1 = new Speaker(2);
        Speaker speaker2 = new Speaker(1);
        Speaker speaker3 = new Speaker(3);
        
        assertThat(Speaker.findBiggestSpeaker(List.of(speaker1, speaker2, speaker3))).isEqualTo(speaker3);
    }
}
```

<br/>

볼륨 값을 받아서 초기화하는 생성자를 하나 더 추가함으로써, 생성자를 통해 바로 볼륨을 설정해주고 테스트를 진행할 수 있게 되었다. 일일이 increaseVolume() 메서드를 통해 볼륨을 설정해서 테스트 할 필요가 없어졌다는 것이다.

만약 비교해볼 스피커가 3개가 아니라 그 이상으로 많아진다면 그 테스트 효율성의 차이는 훨씬 더 커질 것이다.

<br/>

## 4. 부정적인 의견

사실 이 주제를 다루는 글을 지금 작성하는 이유는 부정적인 의견을 보고 이러한 입장에 대해서도 한 번 고민해보기 위해서이다. 대표적인 부정적인 의견들은 어떤 것들이 있을까??

<br/>

### 1. 비즈니스 로직의 이해를 방해할 수 있다

볼륨 값을 받아서 초기화하는 생성자는 테스트만을 위해 구현된 생성자다. 그래서 당연히 비즈니스 로직과는 전혀 상관없는 생성자가 되어버린다. 만약 제 3자가 비즈니스 로직을 보았을 때, **'볼륨 값을 마음대로 초기화 할 수 있는 생성자는 어디서 쓰이고 왜 필요한 것이지??'** 라는 의문점이 들게 된다.

이러한 의문점이 드는 것 자체가 비즈니스 로직의 이해를 방해하는 것이라 볼 수 있다.

<br/>

### 2. 비즈니스 로직에서 사용하면 안될 때, 제 3자가 사용할 수 있는 위험이 존재한다

예를 들어, **'Speaker의 volume은 항상 초기 값이 0이다'** 라는 요구사항이 있다고 가정해보자. 그런데 볼륨 초기 값이 0이 아닌 Speaker 를 제 3자가 만들 수 있는 상황이라면, 이는 요구사항의 의도와는 다른 상황 즉, 버그를 만들 수 있는 것이다.

그래서 이 의견을 내신 분은 대안 책 중 하나로, **비즈니스 로직이 아니라 테스트 파일 내에서 '볼륨을 한번에 조절할 수 있는 메서드'를 만드는 방법**을 제시한 적이 있다.

물론 이 제안은 테스트 코드의 가독성을 떨어뜨릴 수 있다는 부작용이 존재할 수 있다고 생각하지만, 굉장히 일리가 있는 제안이라고 생각한다. 왜냐하면 이는 비즈니스 로직과 테스트 로직 중 어느 쪽에 더 중요성의 무게를 두느냐에 따라 차이가 있을 수 있다고 생각하기 때문이다.

물론 난 테스트 코드의 품질도 프로덕션 코드 만큼이나 높은 품질을 유지해야 한다고 생각한다. 그러나 이러한 대안 책이 생각을 확장하는 계기가 되었다.

<br/>

## 5. 마무리 하며...

이번 우테코 5기 프리코스를 진행하면서 한 주 한 주 지날 때마다, 객체지향 생활 체조 원칙과 우테코에서 제공한 요구사항들을 지키며 미션을 구현 하는 것, 커뮤니티에서 여러가지 주제를 놓고 토론 및 리뷰를 하는 것 등등을 통해 많은 것을 배우고 실력을 쌓아가는 굉장히 유용한 시간이라는 것을 체감하게 된다.

앞으로의 결과가 어떻게 나올 진 모르겠지만, 프리코스를 진행하는 와중에도 이미 프리코스에 대한 깊은 의미와 철학을 느끼게 된다.