# 목차

1. [문제 - 피드백 받기 전의 코드](#1-문제---피드백-받기-전의-코드) <br/>
2. [해결 - 피드백 받은 후의 코드](#2-해결---피드백-받은-후의-코드) <br/>
3. [표현 레벨에서 검증해주는 DTO](#3-표현-레벨에서-검증해주는-dto) <br/>

<br/>

# [자바, Java] View와 Domain 계층의 유효성 검증 분류

<br/>

> **nextstep 에서 이번에 피드백 받은 내용 중, 유효성 검증에 관한 피드백을 받았다.**
>
> **뭔가 간단한 문제인 것 같으면서도 좀 헷갈린 부분이 많은 피드백이었어서 고민하는 시간이 길어졌다.**
>
> **그러한 과정에서 학습하게 된 내용을 간단하게 정리해보려 한다.**

<br/>

## 1. 문제 - 피드백 받기 전의 코드

원래는 입력 값의 유효성 검증을 전부 다 InputView 클래스 즉, View 계층에서 처리를 했다. 그러한 이유 중 하나는 보통 유효성 검증을 Matcher 와 Pattern 을 통해 정규 표현식으로 많은 유효성 검증을 한 번에 하기 때문이다. 정규표현식을 쓰는 이유는 방금 얘기한 것처럼, 수많은 예외를 정규 표현식 하나로 한 번에 처리 할 수 있어서, 유효성 검증에 쓰이는 메서드가 현저하게 줄어들게 된다.



물론 정규 표현식 자체가 가독성이 높지 않다는 단점을 가지고 있다. 하지만 개인적인 생각은 개발자라면 정규표현식은 당연히 알고 있어야 한다고 생각한다. 그리고 정규 표현식으로 인한 가독성 저하의 단점보다, 그로 인해 획기적으로 메서드와 코드의 수가 줄어드는 장점이 더 크다고 생각한다. 그만큼 유효성 검증에 필요한 메서드, 코드의 수가 현격하게 줄어든다.



먼저 피드백 받기 전의 코드를 보자

```java
public class InputView {
    private static final Scanner SCANNER = new Scanner(System.in);
    private static final String PLAYER_NAMES_INPUT_MESSAGE = "\n참여할 사람 이름을 입력하세요. (이름은 쉼표(,)로 구분하세요)";
    private static final String INPUT_EXCEPTION_MESSAGE = "올바른 입력 형식이 아닙니다. 다시 입력해주세요.";
    private static final String PLAYER_NAMES_INPUT_FORM = "[a-zA-Z]{1,5}(,\\s?[a-zA-Z]{1,5})*";
    private static final String SPACE = " ";
    private static final String EMPTY = "";
    private static final String DELIMITER = ",";
    
    public static Players inputPlayerNames() {
        try {
            System.out.println(PLAYER_NAMES_INPUT_MESSAGE);
            return inputPlayerNames(SCANNER.nextLine());
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
            return inputPlayerNames();
        }
    }
    
    public static Players inputPlayerNames(String playerNames) {
        checkBlankException(playerNames);
        checkPlayerNamesInputForm(playerNames);
        return new Players(split(removeSpace(playerNames)));
    }
    
    private static String removeSpace(final String playerNames) {
        return playerNames.replace(SPACE, EMPTY);
    }
    
    private static List<String> split(final String playerNames) {
        return Arrays.stream(playerNames.split(DELIMITER))
                .collect(Collectors.toList());
    }
    
    private static void checkPlayerNamesInputForm(final String playerNames) {
        final Matcher matcher = Pattern.compile(PLAYER_NAMES_INPUT_FORM).matcher(playerNames);
        if (!matcher.matches()) {
            throw new IllegalArgumentException(INPUT_EXCEPTION_MESSAGE);
        }
    }
    
    private static void checkBlankException(final String input) {
        if (StringUtils.isBlank(input)) {
            throw new IllegalArgumentException(INPUT_EXCEPTION_MESSAGE);
        }
    }
}
```

```java
class InputViewTest {
    private static final String INPUT_EXCEPTION_MESSAGE = "올바른 입력 형식이 아닙니다. 다시 입력해주세요.";
    
    @Test
    @DisplayName("플레이어 이름 입력 반환")
    void inputPlayerNames() {
        System.out.println("pobi, jun, honux, jk?? : " + InputView.inputPlayerNames("pobi, jun, honux, jk"));
    }
    
    @DisplayName("플레이어 이름 입력 시, null or \"\" 입력 시 예외 던지기")
    @ParameterizedTest(name = "{displayName} : {0}")
    @NullAndEmptySource
    void inputPlayerNamesNullOrEmptyException(String input) {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> InputView.inputPlayerNames(input))
                .withMessage(INPUT_EXCEPTION_MESSAGE);
    }
    
    @Test
    @DisplayName("플레이어 이름 입력 시, 구분자가 쉼표가 아니면 예외 던지기")
    void inputPlayerNamesDelimiterException() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> InputView.inputPlayerNames("pobi, jun, honux. jk"))
                .withMessage(INPUT_EXCEPTION_MESSAGE);
    }
    
    @Test
    @DisplayName("플레이어 이름 입력 시, 한 플레이어 이름의 글자 5자 초과 시 예외 던지기")
    void inputPlayerNamesLengthException() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> InputView.inputPlayerNames("pobi, tjdtls, honux, jk"))
                .withMessage(INPUT_EXCEPTION_MESSAGE);
    }
    
    @Test
    @DisplayName("플레이어 이름 입력 시, 알파벳이 아닌 경우 예외 던지기")
    void inputPlayerNamesNonAlphabeticException() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> InputView.inputPlayerNames("pobi, jun, ho2ux, jk"))
                .withMessage(INPUT_EXCEPTION_MESSAGE);
    }
}
```

```java
# 기능 구현 목록

### 입력
- [x] 플레이어들의 이름을 입력한다.
  - [x] 입력한 값을 반환한다.
  - [x] 쉼표로 사람 이름을 구분한다.
  - [x] 알파벳이어야 한다.
  - [x] null, "" 입력은 입력할 수 없다.
  - [x] 최대 5글자까지 부여할 수 있다.
    
... 생략
```

<br/>

InputView 와 InputViewTest 를 보면 알겠지만, 기능 구현 목록에 있는 유효성 검증을 전부 다 InputView 에서 처리하고 테스트 하는 모습을 볼 수 있다. 이 상황에서 받은 피드백을 요약하자면 <code><strong>검증은 꼭 한곳에서 이루어져야하는 것은 아니다. 여러가지 제약사항을 뷰와 도메인이 각각 담당하여 적절한 검증을 수행할 수 있다.</strong></code> 라는 내용이었다. 즉 View 또는 Domain 어느 한 곳에서만 유효성 검증을 다 하는 것이 아니라, 이 상태에서 도메인에 관련한 유효성 검증들을 도메인에 따로 또 추가해주는 것이 좋다는 뜻이다.

<br/>

## 2. 해결 - 피드백 받은 후의 코드

결국 지금 문제의 상황에선 도메인에도 따로 도메인 관련 유효성 검증만 추가해주면 좋다는 의미이다. 처음엔 어떻게 View 와 Domain 의 유효성 검증을 분류하는지 정말 고민이 많이 들었다. 고민을 거듭한 결과, <code><strong>도메인 유효성 검증 종류는 요구사항의 내용에서 요구한 제한 사항이다.</strong></code> 라는 결론을 내리게 되었다.



해결 후의 코드를 보자

```java
public class PlayerName {
    private static final int MAX_LENGTH_OF_PLAYER_NAME = 5;
    
    private final String playerName;
    
    public PlayerName(final String playerName) {
        if (playerName.length() > MAX_LENGTH_OF_PLAYER_NAME) {
            throw new PlayerNameLengthExceededException();
        }
        this.playerName = playerName;
    }
    
    ... 생략
}
```

```java
public class PlayerNameTest {
    @Test
    @DisplayName("플레이어 이름 5자 초과시 예외")
    void name_length_exceeded_exception() {
        assertThatIllegalArgumentException()
                .isThrownBy(() -> new PlayerName("abcdef"))
                .withMessage("플레이어 이름은 5자를 초과할 수 없습니다.");
    }
    
    ... 생략
}
```

```java
# 기능 구현 목록

### 입력
- [x] 플레이어들의 이름을 입력한다.
  - [x] 입력한 값을 반환한다.
  - [x] 쉼표로 사람 이름을 구분한다.
  - [x] 알파벳이어야 한다.
  - [x] null, "" 입력은 입력할 수 없다.
  - [x] 최대 5글자까지 부여할 수 있다. (도메인 검증)
```

<br/>

InputView 와 InputViewTest 의 코드는 동일하다. 여기서 도메인 객체인 PlayerName 과 PlayerNameTest 에 도메인 관련 유효성 검증만 추가해 준 것 뿐이다. 요구사항에선 <code><strong>쉼표를 구분자로 사용해야 한다.</strong></code> 와 <code><strong>이름 길이는 5자를 초과할 수 없다.</strong></code> 이렇게 2가지가 있다. 근데 사실 쉼표 구분자 유효성 검증은 도메인 유효성 검증이라기 보단, 사람이 입력할 때의 수많은 변수 중 하나 즉, View 의 유효성 검증에 해당한다고 본다. 그리고 구분자 유효성 검증은 그것만 따로 검증하려 해도 꽤나 많은 메서드가 필요해보인다.



결국 확실한 도메인 관련 유효성 검증은 <code><strong>이름 길이는 5자를 초과할 수 없다.</strong></code> 이다. 사실 InputView 에서 정규표현식을 통해 이미 이름 길이까지 유효성 검증을 하고 있다. 하지만 안전한 설계와 구현을 위해, 피드백 내용처럼 각자의 역할에 맡는 유효성 검증은 전부 추가해 주는 것이 좋아보인다.

<br/>

## 3. 표현 레벨에서 검증해주는 DTO

꼭 Domain 과 View 에서 유효성 검증을 하지 않더라도, 입력 받은 값을 바로 DTO 의 생성자를 통해 유효성 검증을 하면서 DTO 를 반환해주는 형식도 가능하다고 한다. 생각해보면 DTO 의 데이터 전달 경로는 Domain -> View 뿐만이 아닌 View -> Domain 도 될 수 있다는 사실을 간과한 것이다.



DTO 를 통해 유효성 검증을 하고 있는 코드이다.

```java
public class InputView {
    ... 생략

    public static LadderHeightDTO inputLadderHeight() {
        try {
            System.out.println(LADDER_HEIGHT_INPUT_MESSAGE);
            return new LadderHeightDTO(SCANNER.nextLine());
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
            return inputLadderHeight();
        }
    }
    
    ... 생략
}
```

```java
public class LadderHeightDTO {
    private static final String INPUT_EXCEPTION_MESSAGE = "올바른 입력 형식이 아닙니다. 다시 입력해주세요.";
    private static final String LADDER_HEIGHT_INPUT_FORM = "[1-9][0-9]*";
    
    private final int ladderHeight;
    
    public LadderHeightDTO(String ladderHeight) {
        this.ladderHeight = parseLadderHeight(ladderHeight);
    }

    private static int parseLadderHeight(String ladderHeight) {
        checkLadderHeightInputForm(ladderHeight);
        return Integer.parseInt(ladderHeight);
    }

    private static void checkLadderHeightInputForm(String ladderHeight) {
        Matcher matcher = Pattern.compile(LADDER_HEIGHT_INPUT_FORM).matcher(ladderHeight);
        if (!matcher.matches()) {
            throw new IllegalArgumentException(INPUT_EXCEPTION_MESSAGE);
        }
    }
    
    ... 생략
}
```

표현 레벨에서 바로 DTO 의 생성자를 통해 유효성 검증 및 DTO 전달을 수행하고 있다. null 검증은 완전한 View 의 역할이라 생각해서 InputView 로 따로 빼주었다.