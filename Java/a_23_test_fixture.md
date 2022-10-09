# 목차

1. [Test Fixture 는 무엇인가??](#1-test-fixture-는-무엇인가) <br/>
2. [Test 코드에 공을 들일 필요가 있을까??](#2-test-코드에-공을-들일-필요가-있을까) <br/>
3. [내가 처음에 구현했던 Test Fixture](#3-내가-처음에-구현했던-test-fixture) <br/>
4. [Test Fixture 란 존재를 알고난 후, 내 테스트 코드의 발전 과정](#4-test-fixture-란-존재를-알고난-후-내-테스트-코드의-발전-과정) <br/>
    1. [모든 Test Fixture 를 각 테스트 클래스에 상수로 선언한 형태](#모든-test-fixture-를-각-테스트-클래스에-상수로-선언한-형태) <br/>
    2. [Test Fixture 가 많은 테스트의 경우 Test Fixture 패키지로 따로 추출한다.](#test-fixture-가-많은-테스트의-경우-test-fixture-패키지로-따로-추출한다) <br/>
5. [마무리 하며...](#5-마무리-하며) <br/>

<br/>

# [자바, Java] JUnit 테스트 - Test Fixture

<br/>

> **지난주에 nextstep 1단계 자동차 게임 과정을 전부 끝마친 뒤, 자바지기님의 코드를 분석한 적이 있었다.**
>
> **근데 테스트 코드에서 각 테스트 클래스마다 상수 형식으로 선언해놓은 모습을 볼 수 있었다.**
>
> **'어라?? 이 방법 괜찮은 것 같은데??' 라는 생각이 들어서, 실제 구현하는 중인 2단계, 3단계에 적용을 해봤다.**
>
> **그리고 저절로 모든 테스트 클래스들이 그 상수화된 테스트 객체들을 상호 간에 바로바로 사용하면서**
>
> **테스트 코드가 굉장히 깔끔하게 구현되는 신기한 경험을 하게 되었다.**
>
> **이러한 구조들을 검색을 통해 Test Fixture 의 일부임을 알 수 있었다.**
>
> **물론 'java-test-fixtures' 플러그 인을 사용하는 것이 보통이지만,** 
>
> **플러그인을 사용하지 않고 Test Fixture 를 만들면서 프로그램 구현을 했다.**
>
> **그래서 이번 글에선 공부하면서 알게된 Test Fixture 에 대해 정리해보고자 한다.**

<br/>

## 1. Test Fixture 는 무엇인가??

- **여러 테스트에서 같은 Data 를 사용해야 할 때, 코드의 중복을 없애주는 역할을 하는 테스트 전용 데이터이다.**
  - 예를 들어, CarTest 와 CarsTest 둘 다 동일하게 사용하는 데이터가 있다면
  - 그 객체를 Test Fixture 로 생성을 해두고, 그 객체를 두 곳에서 가져다 쓰는 형식이다.

<br/>

## 2. Test 코드에 공을 들일 필요가 있을까??

- **테스트 코드는 그 자체로 프로덕션 코드와 동일하게 높은 품질을 유지해야 한다.**
- **만약 테스트 코드의 품질이 낮다면,**
  - 여기저기의 테스트가 깨질 것이고,
  - 가독성이 떨어져서 어떤 테스트가 이루어지는지 파악이 어려울 것이고,
  - 테스트 코드의 유지보수는 점점 더 어려워지며,
  - 프로덕션 코드와 테스트 코드에 들어가는 자원이 더욱 많아질 것이다.
  - **그렇다고 테스트를 포기한다??, 지옥의 문이 열릴 것이다.**
    - **인간은 신이 아니다.**

<br/>

## 3. 내가 처음에 구현했던 Test Fixture

- 내가 setUp() 메서드를 통해 하는 짓이, Test Fixture 를 설정하는 일이라는 것조차 모르던 때의 코드이다.

- ```java
  class LottoTicketsTest {
      private List<LottoTicket> ascendingLottoTickets;
      private List<LottoNumber> ascendingLottoNumbers;
      private LottoTickets lottoTickets;
  
      @BeforeEach
      void setUp() {
          ascendingLottoNumbers = IntStream.rangeClosed(1, 6)
                  .mapToObj(LottoNumber::new)
                  .collect(Collectors.toList());
          ascendingLottoTickets = IntStream.range(0, 2)
                  .mapToObj(ticketCount -> new LottoTicket(ascendingLottoNumbers))
                  .collect(Collectors.toList());
          lottoTickets = new LottoTickets(ascendingLottoTickets);
      }
      
      @Test
      @DisplayName("여러장의 로또 생성")
      void create() {
          assertThat(lottoTickets).isEqualTo(new LottoTickets(ascendingLottoTickets));
      }
  
      @Test
      @DisplayName("일치 번호 개수 리스트 반환")
      void numberOfMatches() {
          List<LottoNumber> winningLottoNumbers = Arrays.asList(new LottoNumber(1), new LottoNumber(2), new LottoNumber(3), new LottoNumber(4), new LottoNumber(5), new LottoNumber(45));
          List<LottoRank> lottoRanks = lottoTickets.lottoRanks(winningLottoNumbers);
          assertThat(lottoRanks).isEqualTo(Arrays.asList(LottoRank.SECOND, LottoRank.SECOND));
      }
  }
  ```

- ```java
  class LottoNumberTest {
      @Test
      @DisplayName("로또 번호 생성")
      void create() {
          LottoNumber lottoNumber = new LottoNumber(8);
          assertThat(lottoNumber).isEqualTo(new LottoNumber(8));
      }
  }
  ```

  

- **setUp() 메서드 부분을 보면 엄청나다고 볼 수 있다.**

- Test Fixture 를 설정 해주는 것만으로도 저정도다.

  - 벌써부터 테스트가 하기 싫어진다.

- 문제는 저런 Test Fixture 를 설정하는 클래스가 한 두개가 아니다.

  - 내가 내 예전 코드를 지금 다시보는데도 얼마나 짜증났을지 감도 안잡힌다.

<br/>

## 4. Test Fixture 란 존재를 알고난 후, 내 테스트 코드의 발전 과정

1. ### 모든 Test Fixture 를 각 테스트 클래스에 상수로 선언한 형태

   - Test Fixture 를 설정하는 것, 더 나아가 Test 코드도 프로덕션 코드 못지 않게 높은 품질을 유지해야 한다는 사실을 깨달은 이후이다.

   - 처음 자바지기님이 구현한 Test Fixture 의 형태 즉, 각 테스트 클래스의 상수로 생성을 시작했다.

     - ```java
       public class LottoTicketsTest {
           public static final LottoTickets LOTTO_TICKETS = new LottoTickets(Arrays.asList(LottoTicketTest.LOTTO_TICKET, LottoTicketTest.LOTTO_TICKET));
       
           @Test
           @DisplayName("여러장의 로또 생성")
           void create() {
               assertThat(LOTTO_TICKETS).isNotNull();
           }
       
           @Test
           @DisplayName("일치 번호 개수 리스트 반환")
           void numberOfMatches() {
               List<LottoNumber> winningLottoNumbers = Arrays.asList(LottoNumberTest.ONE, LottoNumberTest.TWO, LottoNumberTest.THREE, LottoNumberTest.FOUR, LottoNumberTest.FIVE, new LottoNumber(45));
               List<LottoRank> lottoRanks = LOTTO_TICKETS.lottoRanks(winningLottoNumbers);
               assertThat(lottoRanks).isEqualTo(Arrays.asList(LottoRank.SECOND, LottoRank.SECOND));
           }
       }
       
       class LottoNumberTest {
           public static final LottoNumber ONE = new LottoNumber(1);
           public static final LottoNumber TWO = new LottoNumber(2);
           public static final LottoNumber THREE = new LottoNumber(3);
           public static final LottoNumber FOUR = new LottoNumber(4);
           public static final LottoNumber FIVE = new LottoNumber(5);
           public static final LottoNumber SIX = new LottoNumber(6);
           
           @Test
           @DisplayName("로또 번호 생성")
           void create() {
               assertThat(ONE).isNotNull();
           }
       }
       ```

     - **3번의 예제와 가독성 측면에서 상대조차 되지 않는다.**

       - 그냥 아예 setUp() 메서드가 사라졌다.

       - **여러 테스트 클래스들이 상호 간의 Test Fixture 를 공유하면서 코드가 굉장히 깔끔해진 모습을 볼 수 있다.**

       - 여기선 Test Fixture 를 설정 하는 부분이 두드러지게 차이가 나지만, 

         - 실제로 써보면 테스트를 하는 메서드들도 전부 가독성이 눈에 띄게 좋아진다.

           <br/>

2. ### Test Fixture 가 많은 테스트의 경우 Test Fixture 패키지로 따로 추출한다.

   - 사실 이 방법은 위에서 말한 'java-test-fixtures 플러그 인' 을 사용한 방법을, 플러그 인을 사용하지 않고 유사하게 따라한 모양이다.

   - ```java
     public class LottoTicketsTest {
         public static final LottoTickets LOTTO_TICKETS = new LottoTickets(Arrays.asList(LottoTicketTest.LOTTO_TICKET, LottoTicketTest.LOTTO_TICKET));
         
         @Test
         @DisplayName("여러장의 로또 생성")
         void create() {
             assertThat(LOTTO_TICKETS).isNotNull();
         }
     
         @Test
         @DisplayName("일치 번호 개수 리스트 반환")
         void numberOfMatches() {
             List<LottoNumber> winningLottoNumbers = Arrays.asList(LottoNumberFixture.ONE, LottoNumberFixture.TWO, LottoNumberFixture.THREE, LottoNumberFixture.FOUR, LottoNumberFixture.FIVE, LottoNumberFixture.THIRTY);
             List<LottoRank> lottoRanks = LottoTicketsTest.LOTTO_TICKETS.parseLottoRanks(new WinningLottoNumbers(winningLottoNumbers, LottoNumberFixture.SIX));
             assertThat(lottoRanks).isEqualTo(Arrays.asList(LottoRank.SECOND, LottoRank.SECOND));
         }
     }
     
     public class LottoNumberTest {
         @Test
         @DisplayName("로또 번호 생성")
         void create() {
             assertThat(LottoNumberFixture.ONE).isNotNull();
         }
     }
     
     public class LottoNumberFixture {
         public static final LottoNumber ONE = new LottoNumber(1);
         public static final LottoNumber TWO = new LottoNumber(2);
         public static final LottoNumber THREE = new LottoNumber(3);
         public static final LottoNumber FOUR = new LottoNumber(4);
         public static final LottoNumber FIVE = new LottoNumber(5);
         public static final LottoNumber SIX = new LottoNumber(6);
         public static final LottoNumber THIRTY = new LottoNumber(30);
     }
     ```

     - LottoNumberTest 의 Test Fixture 만 LottoNumberFixture 로 추출한 모습이다.

       - 패키지도 아예 Fixture 전용으로 따로 만들었다.

       <br/>

## 5. 마무리 하며...

사실 Test Fixture 플러그 인까지 사용하면서 테스트 구현을 해보고 싶었는데, nextstep 에서 리뷰를 받을 때 뭔가 최대한 외부 API 를 쓰지 않고 순수 객체지향 설계만으로 해결해보자는 오기가 있었던 것 같다. 덕분에 OOP 에 대한 실력이 처음에 비하면 굉장히 올랐다는 것이 체감이 된다. 역시 강사보다 수강생이 고생해야 수강생의 실력이 늘 수 있다는 nextstep, 특히 자바지기님의 철학이 너무 맘에 드는 시간이었다.