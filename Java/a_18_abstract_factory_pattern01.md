# 목차

1. [추상 팩토리 패턴이란??](#1-추상-팩토리-패턴이란) <br/>
2. [추상 팩토리의 활용성](#2-추상-팩토리의-활용성) <br/>
3. [추상 팩토리 구현](#3-추상-팩토리-구현) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 추상 팩토리 패턴 (Abstract Factory Pattern)

<br/>

> 현재 계속해서 백기선님의 강의를 통해 디자인 패턴을 차근차근 배워나가는 중인데,
>
> 사실 싱글톤이 팩토리 메서드보다 먼저였지만, 싱글톤 패턴을 공부한 내용은 다음 글로 작성할 예정이다.
>
> 이번엔 추상 팩토리 패턴에 대해 배웠는데. 
>
> 팩토리 메서드와 구조는 거의 비슷하지만 그 안에서의 차이점을 최대한 인지하고 배워보도록 하자.

## 1. 추상 팩토리 패턴이란??

- <code><strong>서로 관련이 있거나 독립적인 여러 객체를 생성하기 위한 인터페이스이다.</strong></code>

  - 즉, 팩토리 메서드는 하나의 객체 생성을 위해 인터페이스가 존재했다면,

  - 추상 팩토리는 관련있는 여러 객체 군을 모아서 생성하는 인터페이스라고 보면 된다.

    <br/>

## 2. 추상 팩토리의 활용성

- 여러 활용성이 존재하지만 그 중, 가장 인상적인 특징 3가지만 정리해 보고자 한다.

  1. 클라이언트 코드에서 **구체적인 클래스의 의존성을 제거**하고 싶을 때.
     - 팩토리 메서드 패턴과 같은 활용성으로 보인다.
  2. **관련된 제품 객체들이 함께 사용되도록 설계**되었고, 이 부분에 대한 **제약 조건이 외부에도 지켜지도록** 하고 싶을 때.
     - 아마 이 부분이 팩토리 메서드 패턴과 가장 큰 차이점이지 않을까 싶다.
  3. 여러 제품군 중 하나를 선택해서 제품 설정을 하고, **이후에도 이미 구성된 제품군을 다른 제품군으로 쉽게 대체할 수 있다.**
     - 구체 팩토리는 응용프로그램에서 한번만 나타나기 때문에 구체 팩토리를 변경하기가 매우 쉽다.
     - 하나의 팩토리가 여러 관련있는 제품들을 세트로 묶어서 한번에 생산을 해주기 때문이다.
  
  
  <br/>

## 3. 추상 팩토리 구현

- 이전의 팩토리 메서드 패턴에서 만들었던 House 를 만들건데, 팩토리 메서드와 어떤 부분이 다른지 구현한 코드를 보며 살펴보도록 하자.

  - 학습의 의미로 스스로 강사님의 예제와 다르게 구현해봤다.

- **먼저 다이어그램을 통해 관계를 살펴보자.**

  - ![abstract_factory_img01](https://tjdtls690.github.io/assets/img/github_img/abstract_factory_pattern01.PNG)

    <br/>

  - House 를 생성하는 팩토리 영역과 House 를 구성하는 제품들을 세트로 묶어서 생성하는 팩토리 영역이 있다.

    - 세트 아이템들도 전부 각 인터페이스들이 존재해서 '개방 폐쇄의 원칙' 이 지켜지게 된다.

    <br/>

- 예제

  - ```java
    // 제품 생성을 요청하는 고객 입장
    
    // 이 예제는 편의상 new 연산자를 이용해서 구체 팩토리를 사용했지만, 
    // 실제론 문자열 전송을 통해 팩토리 생성을 하게끔 구현해서 캡슐화의 장점을 살릴 수 있다.
    
    // 집을 구성하는 여러 가구들을 세트로 생성하는 팩토리를 이용해서, 집을 생성하는 모습이다.
    
    public class Client {
        public static void main(String[] args) {
            HouseFactory houseFactory = new WhiteHouseFactory(new BlackHouseDetailsFactory());
            House house = houseFactory.create();
            printHouseInform(house);
        }
        
        private static void printHouseInform(House house) {
            System.out.println(house);
        }
    }
    
    // 출력 결과
    // House{name='WhiteHouse', color='white', door=BlackDoor, wall=BlackWall}
    ```

  - ```java
    // HouseFactory 영역
    public interface HouseFactory {
        House create();
    }
    
    
    // 구체적인 생성을 맡은 HouseFactory
    public class WhiteHouseFactory implements HouseFactory {
        private final HouseDetailsFactory houseDetailsFactory;
        
        public WhiteHouseFactory(HouseDetailsFactory houseDetailsFactory) {
            this.houseDetailsFactory = houseDetailsFactory;
        }
        
        @Override
        public House create() {
            return new WhiteHouse(houseDetailsFactory);
        }
    }
    ```

  - ```java
    // House 영역
    public class House {
        private String name;
        private String color;
        private Door door;
        private Wall wall;
        
        public House(String name, String color, HouseDetailsFactory houseDetailsFactory) {
            this.name = name;
            this.color = color;
            this.door = houseDetailsFactory.createDoor();
            this.wall = houseDetailsFactory.createWall();
        }
        
        @Override
        public String toString() {
            return "House{" +
                    "name='" + name + '\'' +
                    ", color='" + color + '\'' +
                    ", door=" + door.getClass().getSimpleName() +
                    ", wall=" + wall.getClass().getSimpleName() +
                    '}';
        }
    }
    
    
    // House 제품
    public class WhiteHouse extends House {
        public WhiteHouse(HouseDetailsFactory houseDetailsFactory) {
            super("WhiteHouse", "white", houseDetailsFactory);
        }
    }
    ```

    

  - ```java
    // 집을 구성하는 세부 요소들의 세트 생성을 맡은 HouseDetailsFactory 영역
    public interface HouseDetailsFactory {
        Door createDoor();
        
        Wall createWall();
    }
    
    
    // HouseDetailsFactory 영역의 구체 팩토리
    public class BlackHouseDetailsFactory implements HouseDetailsFactory {
        @Override
        public Door createDoor() {
            return new BlackDoor();
        }
        
        @Override
        public Wall createWall() {
            return new BlackWall();
        }
    }
    ```

  - ```java
    // Door 영역
    public interface Door {
    }
    
    // Door 제품
    public class BlackDoor implements Door {
    }
    ```

  - ```java
    // Wall 영역
    public interface Wall {
    }
    
    // Wall 제품
    public class BlackWall implements Wall {
    }
    ```

    <br/>

- 여기서 만약 WhiteHouse 의 세부 요소를 Green 색깔의 문과 벽으로 바꿀 때, '개방 폐쇄의 원칙'이 지켜지고 변경이 매우 쉽다.

  - 제품을 확장한 뒤, 팩토리 하나만 바꿔서 넣어주면 끝이다.

  - ```java
    public class Client {
        public static void main(String[] args) {
            // House 의 세부 요소 제품군 팩토리 하나만 Green 색깔로 교체해서 넣었다
            HouseFactory houseFactory = new WhiteHouseFactory(new GreenHouseDetailsFactory());
            House house = houseFactory.create();
            printHouseInform(house);
        }
        
        private static void printHouseInform(House house) {
            System.out.println(house);
        }
    }
    
    // 출력 결과
    // House{name='WhiteHouse', color='white', door=GreenDoor, wall=GreenWall}
    ```

  - ```java
    public class GreenHouseDetailsFactory implements HouseDetailsFactory {
        @Override
        public Door createDoor() {
            return new GreenDoor();
        }
        
        @Override
        public Wall createWall() {
            return new GreenWall();
        }
    }
    ```

  - ```java
    public class GreenHouseDetailsFactory implements HouseDetailsFactory {
        @Override
        public Door createDoor() {
            return new GreenDoor();
        }
        
        @Override
        public Wall createWall() {
            return new GreenWall();
        }
    }
    ```

  - ```java
    public class GreenDoor implements Door {
    }
    
    public class GreenWall implements Wall {
    }
    ```

    <br/>

  - 다이어그램

    - ![abstract_factory_img02](https://tjdtls690.github.io/assets/img/github_img/abstract_factory_pattern02.PNG)


<br/>

## Reference

- [인프런 - 코딩으로 학습하는 GoF의 디자인 패턴 : 백기선님](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4)
- [GOF 의 디자인 패턴 - 에릭 감마(Erich Gamma), 리차드 헬름(Richard Helm), 랄프 존슨(Ralph Johnson), 존 블리시데스(John Vlissides)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791195444953)