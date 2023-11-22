# 목차

1. [함수형 인터페이스를 통한 입력 및 출력 처리](#1-함수형-인터페이스를-통한-입력-및-출력-처리) <br/>
2. [근데 정말 이걸로 괜찮은거 맞아??](#2-근데-정말-이걸로-괜찮은거-맞아) <br/>

<br/>

# [Java] Functional interface를 통한 의존성 제거

## 1. 함수형 인터페이스를 통한 입력 및 출력 처리

먼저 기존 Application의 로직 중 일부를 먼저 살펴보자.

```java
	private void giveCardToParticipants(BlackJackGame blackJackGame) {
        List<Player> participants = blackJackGame.getParticipants();
        for (Player participant : participants) {
            giveCardToParticipant(blackJackGame, participant);
        }
    }

	private void giveCardToParticipant(BlackJackGame blackJackGame, Player participant) {
        AddCardCommand command = getCommand(participant);
        if (command.isAddCardCommand()) {
            blackJackGame.giveCard(participant);
            OutputView.printParticipantCardCondition(List.of(participant));
        }
        
        if (command.isNotAddCardCommand() || participant.isBurst()) {
            stopGivingCard(participant, command);
            return;
        }
        giveCardToParticipant(blackJackGame, participant);
    }
    
    ... 생략
```

이처럼 참가자들을 Application으로 꺼내서 for문으로 돌려줘야 했다. 이유는 카드 한 장을 더 받을 지 말 지를, InputView에서 입력받은 커맨드를 통해 결정해야하기 때문이다.

하지만 이 코드의 문제는 결국 Application이 게임의 흐름에 지나치게 관여하게 되거나 로직이 너무 더러워진다는 점이다. 또한 Participant들을 getter로 꺼내야 한다. 리뷰어님도 이러한 점들이 좀 걸렸는지, 이런 피드백을 줬다.

<img src="https://tjdtls690.github.io/assets/img/blog/blackjack_001.PNG" width="850" height="300">

처음엔 이게 무슨소린가 했다. Functional interface에 대한 사용법은 당연히 알고 있었지만, 이걸로 어떻게 입력 로직과 비즈니스 로직을 도메인 안에서 같이 진행하게 할 수 있다는건지 이해가 잘 안갔다.

혼자 고민도 해보고 다른 크루들의 로직들도 살펴보다가, **'킹갓 로지'** 의 코드를 보고 바로 이해할 수 있게 되었다.

일단 해당 피드백을 받고 개선한 코드를 먼저 살펴보자.

<img src="https://tjdtls690.github.io/assets/img/blog/blackjack_002.PNG" width="850" height="300">

입력받는다는 행위 자체를 Functional interface로 넘기면 된다. 그럼 이 행위 자체를 도메인 객체 안에서 언제든지 사용할 수 있다. 도메인 안에서 Player들을 순서대로 돌리면서 써도 되고, 그냥 마음대로 쓸 수 있게 된다.

이러한 부분들은 전부 Functional interface 의 특성들 중 **레이지** 특성의 엄청난 장점이 나타난 부분이라 생각한다. 바로 어떤 객체나 데이터가 생겨나는 것이 아닌, Functional interface 가 실행 될 때마다, 각 행동을 고유의 행동으로 봐줄 수 있기 때문이다.

이 부분에서 지금까지 Functional interface를 과소평가하고 있었다는 것을 뼈저리게 느꼈다.

<br/>

## 2. 근데 정말 이걸로 괜찮은거 맞아??

근데 여기서 의문점이 드는 1가지가 생긴다. **View 관련 로직을 행동(Functional interface)으로 넘겨줬다고 해서, 도메인이 View에 대한 의존성이 전혀 없다고 할 수 있는가??** 결국 저 Functional interface 안에 View 관련 로직이 들어간다면, 그것 또한 도메인이 View에 대한 의존성을 가지게 되는 것은 아닐까??

아직 이 부분에 대해선 리뷰어와 의견을 나눠보진 못했다. 하지만 개인적인 의견을 간단히 정리해 보자면,

1. **의존성은 각 클래스 파일의 위에 import 되어있는 것들을 말하는 것이라 생각한다.**
   - 물론 패키지가 같으면 import 문 없이 바로 쓸 수 있지만, 그렇다 해도 결국 **View 객체가 도메인 안에서 명시적으로 적힌 적은 단 한 번도 없다는 것**이 중요하다고 생각한다.
2. **해당 Functional interface에 View관련 로직만 들어온다는 보장이 있는가??**
   - 어떤 명확하게 명시된 객체나 데이터가 아닌 **'행위'** 그 자체이기 때문에, 어떤 행위가 들어올 지는 아무도 알 수 없다.

이것으로 도메인이 View에 의존하지 않으면서도, 도메인 안에서 입력 및 비즈니스 로직을 동시에 수행할 수 있다고 생각한다. 지금 이 글을 쓰다가 리뷰어님의 피드백이 막 도착했는데, 리뷰어님의 의견도 한 번 물어보자...ㅋ