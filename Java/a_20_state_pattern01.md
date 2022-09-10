# 목차

1. [상태 패턴이란??](#1-상태-패턴이란) <br/>
2. [상태 패턴을 쓰는 이유](#2-상태-패턴을-쓰는-이유) <br/>
2. [상태 패턴 학습 구현](#3-상태-패턴-학습-구현) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 상태 패턴 (State Pattern)

<br/>

> **nextstep 의 자바 플레이그라운드 과정을 진행하는 중에 마지막 단계의 미션인 '블랙잭 TDD 구현'을 하게 되었다.**
>
> **이전 단계의 미션까지는 어떻게 어떻게 여러 브랜치를 만들고 시도하면서 TDD 로 구현을 해내었다.**
>
> **그런데 블랙잭은 해당 객체가 어떤 상태인지에 따라 행동 방식이 달라지기 때문에 구현하는데에 꽤 어려움을 느꼈다.**
>
> **몇 번 계속 시도하다가 안되겠다 싶어서 피드백 페이지를 들어갔다.**
>
> **그 페이지에 삽입된 클래스 다이어그램에는 맨 위에 State 인터페이스를 두고 상태 객체들을 관리하는 형식의 피드백 내용이 들어있었다.**
>
> **처음엔 이게 대체 뭐지.. State 인터페이스는 또 뭐야.. 먹는건가.. 하면서 전혀 이해가 가질 않았다.**
>
> **그러다가 블랙잭 미션과 State 에 관한 검색을 통해 이러한 구조가 상태패턴 이란 것을 알게 되었다.**
>
> **그래서 추상 팩토리 패턴에서 바로 상태 패턴으로 넘어온 것이다.** 
>
> **이번에 공부하면서 알게된 점들을 간단히 정리할 생각이다.**

<br/>

## 1. 상태 패턴이란??

- **객체 내부 상태의 변경에 따라 행동이 변화하는 패턴을 말한다.**

  - if ~ else 문같은 조건문을 쓰는 것이 아닌, 상태 자체들도 객체로 관리하는 패턴이다.

    <br/>

## 2. 상태 패턴을 쓰는 이유

1. **가독성과 유지 보수성이 좋아진다.**

   - 상태 패턴을 쓰지 않으면, if ~ else 문과 같이 조건문과 분기점이 많이 생기게 된다.
     - 가독성과 유지보수성이 현저히 떨어지게 된다.
   - 그 각각의 조건들을 전부 클래스로 빼서 그 상태 객체들을 하나의 인터페이스로 관리한다.
     - 객체는 많아지지만, 훨씬 더 가독성이 좋아진다.

2. **OCP (개방폐쇄의 원칙)** : 변경엔 닫혀있고 확장엔 열려있는 객체지향 원칙이 실현된다.

   - 다른 상태가 추가가 된다고 해도, 기존의 코드는 변경하지 않고 확장이 가능해진다.

     <br/>

## 3. 상태 패턴 학습 구현

- 먼저 어떤 프로그램을 상태 패턴을 통해 구현할 것인지 설계해보았다.

  1. 호텔이 있다.
  2. 호텔을 이용하려면 먼저 고객이 호텔로부터 티켓을 받아야 한다.
  3. 호텔의 상태는 총 3가지가 존재한다.
     1. **Draft : 고객이 아직 없어서 운영을 안하는 상태.**
        - 티켓이 있든 없든 고객은 받을 수 있는 상태
        - 리뷰 작성이 불가능한 상태
     2. **Published : 고객이 한 명이 들어와서 운영을 막 시작한 상태**
        - 티켓이 있든 없든 고객은 받을 수 있는 상태
        - 호텔을 이용하지 않더라도 모두 리뷰 작성이 가능한 상태
     3. **Private : 호텔을 이용하는 고객이 2명 이상이 되어 보안을 지키기 시작한 상태**
        - 티켓을 받은 사람만 호텔을 이용할 수 있는 상태
        - 호텔을 이용하고 있는 사람만 리뷰 작성이 가능한 상태

- 예제

  - ```java
    // 호텔
    public class Hotel {
        private State state;
        private final Set<Client> clients;
        private final Set<String> reviews;
        
        public Hotel() {
            // 호텔은 고객이 한 명도 없는 Draft 상태에서 시작
            this.state = new Draft(this);
            this.clients = new HashSet<>();
            this.reviews = new HashSet<>();
        }
        
        public void addClient(Client client) {
            this.state.addClient(client);
        }
        
        public void addReview(Client client, String review) {
            this.state.addReview(client, review);
        }
        
        public void changeState(State state) {
            this.state = state;
        }
        
        public Set<Client> getClients() {
            return clients;
        }
        
        public Set<String> getReviews() {
            return reviews;
        }
        
        public State getState() {
            return state;
        }
    }
    ```
    
  - ```java
    // 고객
    public class Client {
        private final String name;
        private final Set<Hotel> hotels;
        
        public Client(String name) {
            this.name = name;
            this.hotels = new HashSet<>();
        }
        
        public void addPrivate(Hotel hotel) {
            this.hotels.add(hotel);
        }
        
        
        public boolean isAvailable(Hotel hotel) {
            return hotels.contains(hotel);
        }
        
        @Override
        public String toString() {
            return "Client{" +
                    "name='" + name +
                    '}';
        }
    }
    ```
  
  - ```java
    // 모든 상태 객체를 관리하는 인터페이스
    public interface State {
        void addClient(Client client);
        void addReview(Client client, String review);
    }
    ```
  
  - ```java
    // Draft 상태
    public class Draft implements State {
        private final Hotel hotel;
        
        public Draft(Hotel hotel) {
            this.hotel = hotel;
        }
        
        @Override
        public void addClient(Client client) {
            hotel.getClients().add(client);
            if (hotel.getClients().size() > 0) {
                hotel.changeState(new Published(hotel));
            }
        }
        
        @Override
        public void addReview(Client client, String review) {
            throw new UnsupportedOperationException("해당 호텔은 고객이 0명이기 때문에 리뷰를 하실 수 없습니다.");
        }
    }
    
    
    // Published 상태
    public class Published implements State {
        private final Hotel hotel;
        
        public Published(Hotel hotel) {
            this.hotel = hotel;
        }
        
        @Override
        public void addClient(Client client) {
            hotel.getClients().add(client);
            if (hotel.getClients().size() > 1) {
                hotel.changeState(new Private(hotel));
            }
        }
        
        @Override
        public void addReview(Client client, String review) {
            hotel.getReviews().add(review);
        }
    }
    
    
    
    
    // Private 상태
    public class Private implements State {
        private final Hotel hotel;
        
        public Private(Hotel hotel) {
            this.hotel = hotel;
        }
        
        @Override
        public void addClient(Client client) {
            if (client.isAvailable(hotel)) {
                hotel.getClients().add(client);
                return;
            }
            
            throw new UnsupportedOperationException("호텔을 이용하실 수 없습니다.");
        }
        
        @Override
        public void addReview(Client client, String review) {
            if (hotel.getClients().contains(client)) {
                hotel.getReviews().add(review);
                return;
            }
            
            throw new UnsupportedOperationException("리뷰를 하실 수 없습니다.");
        }
    }
    ```
  
  - ```java
    // 메인 메서드
    public class Main {
        public static void main(String[] args) {
            Hotel hotel = new Hotel();
            Client jun = new Client("jun");
            Client young = new Client("young");
            Client sin = new Client("sin");
            
            // Draft 상태에선 리뷰 작성 불가능
    //      hotel.addReview(sin, "hello");
            
            hotel.addClient(jun); // 고객이 한 명 늘어서 Published 상태로 변경
            hotel.addReview(sin, "hello"); // Published 상태에선 호텔을 이용하지 않는 고객도 리뷰 작성 가능
            
            hotel.addClient(young); // 고객이 두 명 이상이 되어서 Private 상태로 변경
            hotel.addReview(young, "nice!!"); // Private 상태에서 호텔을 이용하는 고객은 리뷰 작성 가능
    
            sin.addPrivate(hotel); // Private 상태이기 때문에 티켓 발급부터 받아야 한다.
            hotel.addClient(sin); // Private 상태에서 티켓을 발급 받은 고객은 호텔 이용 가능
            hotel.addReview(sin, "hi~"); // Private 상태에서 호텔을 이용중인 고객은 리뷰 작성 가능
            
        
            System.out.println(hotel.getClients()); // 현재 호텔을 이용중인 모든 고객 출력
            System.out.println(hotel.getReviews()); // 현재 작성된 모든 호텔의 리뷰 출력
            System.out.println(hotel.getState().getClass().getSimpleName()); // 현재 호텔의 상태 출력
        }
    }
    
    // 출력 결과
    // [Client{name='young}, Client{name='sin}, Client{name='jun}]
    // [nice!!, hello, hi~]
    // Private
    ```
    
    