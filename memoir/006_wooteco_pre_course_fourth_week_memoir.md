# 목차

1. [상태 패턴으로 분리하면서 체감하게 된 이점](#1-상태-패턴으로-분리하면서-체감하게-된-이점) <br/>
2. [enum 클래스로 분리하면서 얻은 이점](#2-enum-클래스로-분리하면서-얻은-이점) <br/>

<br/>

# 우아한 테크 코스 5기 프리코스 - 4주차 회고

<br/>

> **우테코 프리코스 4주의 기간동안 정말 많은 것을 배웠다.**
>
> **물론 1:1 피드백 없이 주마다 공통 피드백만 받으면서 진행했다.**
>
> **하지만 '토론과 피어리뷰'를 통해 '미션 설계 및 구현에 대한 부분'과 '더 클래스를 분리할 수 있는 부분',**
>
> **'막연하게만 생각했던 개념들의 구체화', '도메인을 설계하는 과정' 등등**
>
> **너무나 많은 부분에서 배우고 실력을 더 올릴 수 있었다.**
>
> **이런 귀한 계기를 제공해준 우테코에 정말 감사드립니다.**
>
> **디테일한 부분까지 회고하면 너무 길어질 것 같아서, 가장 인상깊었던 부분 몇가지만 정리해보려 한다.**

<br/>

- **4주차 문제 :** [다리 건너기 게임](https://bit.ly/3XuYYya)

- **내가 제출한 PR :** [다리 건너기 게임 제출 PR](https://bit.ly/3U5NLAR)

  <br/>

## 1. 상태 패턴으로 분리하면서 체감하게 된 이점

이번 4주차에선 3주차와 마찬가지로 클래스 분리에 더 초점을 맞춘 요구사항들이 들어왔다. 그런 만큼 클래스를 분리해보기 위해 많이 살펴보고 시도를 했었고, 그 과정에서 얻은 경험에 대해 정리해보려 한다. 그 중 1가지가 상태 패턴(인터페이스) 추출이다.

처음 4차 미션의 요구사항을 봤을 때, 수많은 if문이 필요할 수 있겠다고 생각했다. '이동하는 상태의 종류'와 '게임 종료 조건'이 여러 가지였기 때문이다. 그래서 최대한 if문을 줄이기 위해, 다리를 건너는 과정을 상태 패턴을 적용하여 설계했다.

<img src="https://tjdtls690.github.io/assets/img/blog/diagram01.PNG" width="700" height="600">

```java
public interface MovingResultState {
    MovingResultState move(final int positionToMove, final String moving);
    
    boolean isMoveFailed();
    
    boolean isGameFinished(final int numberOfMoves);
    
    String moving();
}
```

```java
public abstract class Started implements MovingResultState {
    private final Bridge bridge;
    
    Started(final Bridge bridge) {
        this.bridge = bridge;
    }
    
    public Bridge bridge() {
        return bridge;
    }
}
```

```java
public abstract class MovingComplete extends Started {
    private static final String MOVE_NOT_AVAILABLE = "[ERROR] 이미 다음 위치로 이동이 끝난 상황입니다.";
    
    private final String moving;
    
    MovingComplete(final Bridge bridge, final String moving) {
        super(bridge);
        this.moving = moving;
    }
    
    @Override
    public MovingResultState move(final int positionToMove, final String moving) {
        throw new IllegalStateException(MOVE_NOT_AVAILABLE);
    }
    
    @Override
    public String moving() {
        return moving;
    }
    
    @Override
    public String toString() {
        return " {bridge => " + bridge() + "moving => " + moving + "} ";
    }
}
```

```java
public class Ready extends Started {
    private static final String FAIL_CHECK_NOT_AVAILABLE = "[ERROR] 현재 상태에선 이동 성공 여부를 판단할 수 없습니다.";
    private static final String GAME_END_CHECK_NOT_AVAILABLE = "[ERROR] 현재 상태에선 게임 종료 여부를 판단할 수 없습니다.";
    private static final String MOVING_RETURN_NON_AVAILABLE = "[ERROR] 아직 이동하지 않아서 데이터가 없습니다.";
    
    public Ready(final Bridge bridge) {
        super(bridge);
    }
    
    @Override
    public MovingResultState move(final int positionToMove, final String moving) {
        if (isPartBridgeExist(positionToMove, moving)) {
            return new Success(bridge(), moving);
        }
        
        return new Fail(bridge(), moving);
    }
    
    private boolean isPartBridgeExist(final int positionToMove, final String moving) {
        return bridge().isPartBridgeExist(positionToMove, moving);
    }
    
    @Override
    public boolean isMoveFailed() {
        throw new IllegalStateException(FAIL_CHECK_NOT_AVAILABLE);
    }
    
    @Override
    public boolean isGameFinished(final int numberOfMoves) {
        throw new IllegalStateException(GAME_END_CHECK_NOT_AVAILABLE);
    }
    
    @Override
    public String moving() {
        throw new IllegalStateException(MOVING_RETURN_NON_AVAILABLE);
    }
}
```

```java
public class Success extends MovingComplete {
    public Success(final Bridge bridge, final String moving) {
        super(bridge, moving);
    }
    
    @Override
    public boolean isMoveFailed() {
        return false;
    }
    
    @Override
    public boolean isGameFinished(final int numberOfMoves) {
        return bridge().isSameWithBridgeSize(numberOfMoves);
    }
}
```

```java
public class Fail extends MovingComplete {
    public Fail(final Bridge bridge, final String moving) {
        super(bridge, moving);
    }
    
    @Override
    public boolean isMoveFailed() {
        return true;
    }
    
    @Override
    public boolean isGameFinished(final int numberOfMoves) {
        return false;
    }
}
```

<br/>

사실 이 부분에서 고민이 들었던 점은, **Fail 상태일 땐 Bridge 인스턴스를 쓸 상황이 없는데, 쓸데없이 생성자를 통해 인스턴스를 받아놓기 때문이다.** 그렇다고 **Ready와 Success 에 따로따로 Bridge 를 갖고있게 하자니, 코드 중복이기에 비효율적이라고 생각했다.** 

그런 이유로, 일단 Started 추상 클래스를 통해 전부 Bridge 를 갖고 있는 것으로 마무리를 지었지만 여전히 어떤 방식이 더 효율적일지 헷갈리는 부분인 것 같다.

이제 본론으로 들어가기 위해, 일단 이 상태 인스턴스들을 이용하는 클래스를 살펴보자.

```java
public class MovingResultStates {
    private final LinkedList<MovingResultState> movingResultStates;
    
    public MovingResultStates() {
        movingResultStates = new LinkedList<>();
    }
    
    public void move(final MovingDTO movingDTO, final Bridge bridge) {
        readyState(bridge);
        convertToNextState(movingDTO);
    }
    
    private void readyState(final Bridge bridge) {
        movingResultStates.add(new Ready(bridge));
    }
    
    private void convertToNextState(final MovingDTO movingDTO) {
        movingResultStates.set(statesLastIndex(), nextState(movingDTO));
    }
    
    private MovingResultState nextState(final MovingDTO movingDTO) {
        return lastState().move(statesLastIndex(), movingDTO.getMoving());
    }
    
    private int statesLastIndex() {
        return movingResultStates.size() - 1;
    }
    
    private MovingResultState lastState() {
        return movingResultStates.getLast();
    }
    
    public boolean isMoveFail() {
        return lastState().isMoveFailed();
    }
    
    public void initMovingResultStates() {
        movingResultStates.clear();
    }
    
    public boolean isGameFinished() {
        return !isStatesEmpty() && isAllSucceed();
    }
    
    private boolean isAllSucceed() {
        return lastState().isGameFinished(movingResultStates.size());
    }
    
    private boolean isStatesEmpty() {
        return movingResultStates.isEmpty();
    }
    
    public List<MovingResultState> states() {
        return Collections.unmodifiableList(movingResultStates);
    }
    
    public List<String> movings() {
        return movingResultStates.stream()
                .map(MovingResultState::moving)
                .collect(Collectors.toUnmodifiableList());
    }
    
    @Override
    public String toString() {
        return "MoveResultStates{" +
                "states=" + movingResultStates +
                '}';
    }
}
```

<br/>

이 예제를 보면서 하나씩 정리해보고자 한다. 상태 패턴(인터페이스)을 적용하면서 가장 체감했던 장점 2가지가 있다.

1. **if문 사용이 굉장히 적어짐**

   만약 인터페이스를 추출하지 않았다면 if문을 몇 개나 써야 했을까?? 직접 if문으로 구현해본 적은 없지만, 최소한 3~4개는 더 써야할 것 같다는 생각이 들었다. '이동의 실패 vs 성공', '바로 종료 vs 재시도 선택' 등등 여러 가지를 판단하는 if문이 난무했을 것이다.

   하지만 이 예제에선 if문 사용 없이 이 조건들을 판단하는 로직을 무리없이 구현한 모습을 볼 수 있다. 이유는, 상태 패턴의 각 클래스들이 알아서 판단해서 다음 상태 인스턴스를 넘겨주고, 각 상태 클래스마다 메서드의 동작 방식이 알아서 달라지기 때문이다.

2. **가독성 향상**

   사실 이 부분은 이번에 처음으로 체감한 부분이다. 다리 건너기를 성공(Success)했을 때와 실패(Fail)했을 때의 로직을, 따로따로 응집되어있는 클래스를 통해 살펴볼 수 있게 된다. 국비지원 학원의 프로젝트를 진행하면서 if문을 거리낌 없이 난무했을 때와 비교해보면, 코드를 파악할 수 있는 난이도가 천지차이다.

   그만큼 가독성과 유지보수성이 증가한다는 것을 직접적으로 체감하는 계기가 되었다.

<br/>

## 2. enum 클래스로 분리하면서 얻은 이점

저번 3주차의 공통 피드백에서 이런 피드백이 들어있었다.

>  **연관성이 있는 상수는 static final 대신 enum을 활용한다**

<br/>

사실 이 부분은 내가 지금까지 객체지향 설계를 해보면서 한 번도 시도한 적 없는 부분이다. 문자열들을 상수화를 통해 명확한 네이밍만 잘 해준 것만으로도 충분하다고 여겼기 때문이다.

하지만 이 피드백을 보고, 이런 연관성이 있는 상수를 잘 묶어서 enum 으로 분리해 준다면, 이 또한 위의 상태 패턴 적용과 마찬가지로 복잡한 if문과 같은 로직들을 개선할 수 있겠다는 생각이 들었다.

밑의 예제는 피드백을 적용해보기 위해, 연관성 있는 상수를 enum으로 추출한 모습이다.

```java
public enum MoveResultDisplay {
    SUCCESS("성공", "O"),
    FAIL("실패", "X");
    
    private final String gameResult;
    private final String movingResult;
    
    MoveResultDisplay(final String gameResult, final String movingResult) {
        this.gameResult = gameResult;
        this.movingResult = movingResult;
    }
    
    public static List<MoveResultDisplay> convertToMoveResult(final List<MovingResultState> moveMovingResultStates) {
        return moveMovingResultStates.stream()
                .map(MoveResultDisplay::parseMoveResult)
                .collect(Collectors.toUnmodifiableList());
    }
    
    private static MoveResultDisplay parseMoveResult(final MovingResultState movingResultState) {
        if (isStateFailed(movingResultState)) {
            return FAIL;
        }
        
        return SUCCESS;
    }
    
    private static boolean isStateFailed(final MovingResultState movingResultState) {
        return movingResultState.isMoveFailed();
    }
    
    public String getGameResult() {
        return gameResult;
    }
    
    public String getMovingResult() {
        return movingResult;
    }
}
```

<br/>

이를 통해 가장 득을 많이 본 클래스는 OutputView이다. 먼저 위 예제처럼 enum으로 상수를 추출하지 않았을 때의 코드를 보자.

```java
public class OutputView {
    private static final String SUCCESS_MESSAGE = "성공";
    private static final String FAIL_MESSAGE = "실패";
    private static final String PLACES_TO_GO_DISPLAY = "O";
    private static final String PLACES_NOT_TO_GO_DISPLAY = "X";
    
    ... 생략
    
    private String parseMovingResultDisplay(final MoveResult moveResult) {
        if (moveResult.isSuccess()) {
            return PLACES_TO_GO_DISPLAY;
        }
        
        return PLACES_NOT_TO_GO_DISPLAY;
    }
    
    ... 생략
    
    private String whetherGameSuccess(final GameResultDTO gameResultDTO) {
        if (lastMovingResult(gameResultDTO).isSuccess()) {
            return SUCCESS_MESSAGE;
        }
        
        return FAIL_MESSAGE;
    }
    
    ... 생략
}
```

<br/>

지금 설명하려는 부분과 관련 있는 메서드들만 가져와봤다. 이처럼 이전엔 if문을 통해 모든 것을 판단하면서 출력 문자열을 정해야했다. 하지만 enum 으로 출력 한 뒤의 밑의 예제에선 상수의 개수를 줄이고 if문 로직이 간결하게 개선된 모습을 볼 수 있다.

```java
public class OutputView {
    
    ... 생략
    
    private String currentMovingResult(final GameResultDTO gameResultDTO, final int countOfMoving) {
        return currentMoveResult(gameResultDTO, countOfMoving).getMovingResult();
    }
    
    ... 생략
    
    private String whetherGameSuccess(final GameResultDTO gameResultDTO) {
        return lastMovingResult(gameResultDTO).getGameResult();
    }
    
    ... 생략
}
```

안그래도 상수의 개수가 굉장히 많고 메서드의 수도 많았던 클래스인데, enum으로 분리하는 과정을 통해 이전보다 더 깔끔하게 개선시킬 수 있었다. 그리고 위의 예제에선 안 나타났지만, 잔여 효과로 이 외의 private 메서드들 중에서도 같이 줄어든 메서드들이 꽤 된다.

가시적인 효과를 정리해보자면, 모든 클래스에서 (테스트 코드 포함, 새로 생긴 메서드 개수만큼 빼고) 메서드가 5개, static final이 4개 줄었고, OutputView의 로직이 이전에 비해 if문이 많이 사라져서 굉장히 깔끔해졌다.

<br/>

이제 모든 프리코스 과정이 끝났고 1차 합격자 발표까지 3주를 기다려야 한다. 그 3주동안 더 레벨업을 하기 위해, 지금까지 했던 미션들과 이전 기수들이 진행했던 미션들을 지금까지의 방식이 아닌 **'책임 주도 설계'**를 통해 설계 및 구현을 해볼 생각이다.
