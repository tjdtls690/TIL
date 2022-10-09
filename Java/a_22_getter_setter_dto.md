# 목차

1. [객체지향 설계에서 getter와 setter 사용을 왜 지양해야 하는가??](#1-객체지향-설계에서-getter와-setter-사용을-왜-지양해야-하는가) <br/>
2. [그래도 getter 는 써야할 때가 있다.](#2-그래도-getter-는-써야할-때가-있다) <br/>
3. [계층간 데이터 전달은 왜 굳이 DTO 를 사용해야 하는가??](#3-계층간-데이터-전달은-왜-굳이-dto-를-사용해야-하는가) <br/>
4. [DTO 는 무엇인가??](#4-dto-는-무엇인가) <br/>
5. [단, 도메인은 DTO 를 알면 안된다.](#5-단-도메인은-dto-를-알면-안된다) <br/>
6. [DTO 실사용 예제](#6-dto-실사용-예제) <br/>

<br/>

# [자바, Java] 객체지향 설계 - 계층을 넘어갈 땐, getter와 setter 대신 DTO

<br/>

> **난 국비학원에서 처음 코딩을 접하였고,**
>
> **많은 사람들이 알다시피 국비학원은 객체지향 설계같은거 신경조차 쓰지 않는다.** 
>
> **분명 Java 라는 객체지향 언어로 프로그래밍을 하는데, 코드를 짜는 것을 보면 절차지향 언어와 다를바가 없다.**
>
> **심지어 절차지향 언어보다 더 하드코딩이 되는 경우도 다반사다.**
>
> **현재 nextstep에서 미션을 진행하고 있는데, 여기서 공부한 내용들을 점진적으로 글로 작성해보려 한다.**
>
> **피드백 받은 내용과 공부하면서 스스로 깨달은 것들에 대해서 같이 정리해보고자 한다.**

<br/>

## 1. 객체지향 설계에서 getter와 setter 사용을 왜 지양해야 하는가??

1. **캡슐화가 깨져버린다.**
   - 물론 getter 로 꺼낸다고 해서 무조건 객체의 변경까지 가능한 것은 아니다
   - 그러나 그 꺼낸 값이 원시값이 아닌 객체라면 위험해진다.
   - getter 의 반환값으로 도메인 관련 객체가 나오게 된다면, setter 또는 다른 메서드로 그 객체의 상태를 변경시켜버릴 수 있다.
2. **getter 메서드의 사용을 남발하는 안좋은 습관을 만들 이유가 없다.**
   - getter 와 setter 를 사용을 할 수록, 하드코딩이 된다.
   - 해당 객체에 메시지 하나만 보내면 끝날 일을, 왜 굳이 꺼내서(getter) 변경 시킨다음 다시 집어넣으려고(setter) 하는가??
   - 전혀 객체지향적이지 못하다는 것은 코드를 보면 바로 느껴지게 되어있다.

<br/>

## 2. 그래도 getter 는 써야할 때가 있다.

- **setter 는 무슨일이 있어도 사용을 금기시한다.**
  - 캡슐화는 개나주고 객체를 마음대로 변경시켜버리는 무시무시한 메서드이다.
- **그래도 getter 는 쓸 일이 있다. 이유는??**
  - service, controller, view 계층으로 데이터를 보내야 할 때, 어차피 한 번은 써야한다.
  - 안 그러면 데이터를 꺼내서 전달해 줄 방법이 없다.
- **그렇다면 getter 는 언제 쓸까??**
  1. **다른 계층으로 데이터를 전달하고자 할 때, DTO 를 통해 데이터를 전달하는 용도로 사용한다.**
     1. controller 나 service 계층에서, DTO 생성자 매개변수로 도메인 객체를 바로 넣어주면서 DTO를 생성한다.
     2. DTO 안에서 생성자가 도메인 객체의 getter 메서드를 이용해서 데이터를 꺼내고, 그 데이터를 DTO 에 저장한다.
     3. 해당 DTO 를 다른 계층으로 전달한다.
     4. 전달 받은 계층에서 DTO 의 getter 를 통해 데이터를 꺼낸다.
  2. **도메인간 데이터 전달 목적으로 사용한다.**
     - 도메인간 데이터 이동은 어차피 다른 계층을 넘나드는 것이 아니기 때문에 getter 를 통해 전달 받아도 크게 상관없다.
     - 단, 남발하거나 getter 를 통해 객체가 마음대로 변형되는 일은 없어야 할 것이다.

<br/>

## 3. 계층간 데이터 전달은 왜 굳이 DTO 를 사용해야 하는가??

- **캡슐화를 지키기 위함이다.**
  - 다른 계층인 Controller, Service, View 계층 같은 곳에서 도메인 객체를 알고 있다면, **의도치 않은 혹은 의도한 도메인 객체의 변형이 일어날 수 있다.**
- 그래서 객체의 데이터만을 전달만을 목적으로 갖는 DTO 를 통해 전달한다.
  - 그러면 **도메인 객체의 멤버 정보들은 전혀 모르게 되면서, 데이터를 전달할 수 있게 된다.**
- 물론 DTO 에서도 데이터 저장은, setter 가 아닌 생성자를 통해서 해야 한다.
  - **setter 는 모든 악의 근원이다...ㅋ**

<br/>

## 4. DTO 는 무엇인가??

- **Data Transfer Object** : 계층 간 데이터 교환을 하기 위해 사용하는 객체로, DTO는 로직을 가지지 않는 순수한 데이터 객체(getter & setter 만 가진 클래스)다.

  - 현재 nextstep 을 통해 자바 프로그래밍을 학습할 땐 setter 를 넣어주지 않지만, 

  - 스프링에선 자동으로 getter 와 setter 를 통해 DB와 다른 계층 간 데이터 전달이 이루어지는 것으로 알고 있다.

    <br/>

## 5. 단, 도메인은 DTO 를 알면 안된다.

- 내가 nextstep 을 통해 가장 많은 피드백을 받은 부분이 바로 이 부분이었다.

- 내가 처음에 DTO 를 썼던 방식을 간단한 예제로 보여주겠다.

  - ```java
    public class Car {
        String carName;
        
        public Car(String carName) {
            this.carName = carName;
        }
        
        public CarDTO getCarInformation() {
            return new CarDTO(carName);
        }
    }
    ```

  - 위 예제는 핵심 도메인 객체 중 하나인 Car 객체라 가정해보자.

  - 다른 계층에서 Car 의 information 메서드를 통해 DTO 를 반환받는 형태임을 알 수 있다.

- 이와 같은 상황에선 어떤 부분이 문제가 될까??

  1. **다른 계층에서 도메인 객체를 모르도록 최대한으로 조치를 취하지 못한다.**
  2. **도메인과 DTO 의 의존성이 생겨버린다.**
     - 나중에 요구사항의 변경 또는 다양한 형태의 여러 API 에서 도메인을 받아올 때, 상호간의 유지보수가 굉장히 힘들어지게 된다.

- 그래서 피드백을 받은 후, 현재 내가 사용하는 DTO 의 용법을 밑의 예제에서 살펴보겠다.

<br/>

## 6. DTO 실사용 예제

- ```java
  public class PlayerDTO {
      private final String playerName;
      private final List<Frame> frames;
      
      
      public PlayerDTO(final Player player) {
          final PlayerName playerName = player.getPlayerName();
          this.playerName = playerName.getPlayerName();
          
          final Frames frames = player.getFrames();
          this.frames = frames.getFrames();
      }
      
      public String getPlayerName() {
          return playerName;
      }
      
      public List<Frame> getFrames() {
          return Collections.unmodifiableList(frames);
      }
  }
  ```

  - 현재 내가 nextstep 의 사다리 게임 구현한 부분 중 하나의 DTO 를 가져온 것이다.

    - 아까 말했듯, **setter 가 아닌 생성자를 통해 DTO 에 저장하는 모습을 볼 수 있다.**

      <br/>

- ```java
  public class Bowling {
      public void run() {
          ... 생략
              
          ResultView.printPlayerFramesDisplay(new PlayerDTO(player));
          
          ... 생략
      }
  }
  ```

  - **컨트롤러에서 도메인의 정보를 DTO 생성자를 통해 View 계층으로 전달하는 모습이다.**

    <br/>

- ```java
  public class ResultView {
      ... 생략
      
      public static void printPlayerFramesDisplay(final PlayerDTO playerDTO) {
          System.out.println(BOARD_BASE_DISPLAY);
          System.out.println(getPlayerResultDisplayFormat(playerDTO));
      }
      
      private static String getPlayerResultDisplayFormat(final PlayerDTO playerDTO) {
          return IntStream.range(0, 10)
                  .mapToObj(index -> getFrameDisplayFormat(playerDTO.getFrames(), index))
                  .collect(Collectors.joining(DELIMITER, DELIMITER + parseFrameDisplayPrintFormat(playerDTO.getPlayerName()) + DELIMITER, DELIMITER));
      }
      
      ... 생략
  }
  ```

  - ResultView (view 계층) 에서 DTO 의 getter 를 통해 데이터를 전달받고, 사용하는 모습을 볼 수 있다.

    <br/>

- **객체지향 설계, TDD 학습은 언제나 옳다.**