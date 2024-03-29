# 목차

1. [문제 발생](#1-문제-발생) <br/>
2. [문제 해결](#2-문제-해결) <br/>
    1. [의존성 역전](#1-의존성-역전) <br/>
    2. [외부 클래스 이용](#2-외부-클래스-이용) <br/>

<br/>

# [Java] 순환 참조를 방지하기 위한 의존성 역전 및 의존 관계 연결

## 1. 문제 발생

<img src="https://tjdtls690.github.io/assets/img/blog/chess_001.PNG">

내가 설계했던 구조는, Piece를 move하라고 명령할 때, 현재 해당 Piece가 자신이 갈 수 있는 좌표를 전부 알아내는 것이다. 그러려면 Piece가 ChessBoard를 파라미터로 받고, 그 ChessBoard를 이용해서 자신이 갈 수 있는 좌표를 구해야 한다.

문제는 이러한 구조는 위의 이미지처럼 ChessBoard와 Piece가 순환 참조 형태를 띄게 된다는 것이다. ChessBoard는 처음부터 좌표들과 Piece들을 갖고있고, Piece는 파라미터로 ChessBoard를 받음으로써 서로가 서로에 대한 의존성을 가지게 된다.

그렇다면 **순환 참조는 어떤 이유때문에 회피하라는 것**인가??

1. **메모리 누수** : 객체들이 서로를 계속해서 참조하게 되면, 메모리에서 해제되지 않는 객체들이 계속해서 쌓이면서 메모리 누수가 발생할 수 있다.
2. **예측 불가능한 동작** : 순환 참조는 어떤 객체가 먼저 생성될지, 또 어떤 객체가 어떤 객체를 참조할지에 대한 예측이 어렵다. 이로 인해 예기치 않은 동작이 발생할 수 있다.
3. **가비지 컬렉션 오버헤드** : 순환 참조를 가진 객체들은 가비지 컬렉션의 오버헤드를 증가시킨다. 가비지 컬렉터는 순환 참조를 가진 객체들을 찾아내고 처리하는데 추가적인 자원이 소모된다.
4. **코드 복잡도 증가** : 순환 참조를 가진 객체들은 객체 간의 의존성이 복잡해지며 코드의 가독성과 유지보수성을 저하시킨다. 순환 참조 객체들 중 어떤 한 객체에서 디버깅을 했을 때, 순환 참조가 되고있는 객체들이 연쇄적으로 영향을 받을 수 있는 위험이 존재한다.

<br/>

## 2. 문제 해결

### 1) 의존성 역전

<img src="https://tjdtls690.github.io/assets/img/blog/chess_003.PNG">

그래서 찾은 해결법이 의존성 역전이다. ChessBoard 인터페이스를 만들고, ChessBoard 구현체와 Piece가 ChessBoard 인터페이스를 바라보게끔 개선했다. 

그런데 기능을 더 추가하다보니 ConcreteBoard 클래스에서 Piece를 파라미터로 받을 일이 생기면서, ChessBoard 인터페이스도 Piece를 의존하게 되었다. 해당 Piece와 경로에 있는 기물들을 비교하면서 같은 팀인지 확인을 해야하기 때문이다.

<br/>

### 2) 외부 클래스 이용

<img src="https://tjdtls690.github.io/assets/img/blog/chess_002.PNG">

그래서 마지막으로 찾은 방법이 ChessBoard를 의존하게 만드는 Piece에 있는 로직을 외부 클래스로 빼는 것이다. 결과는 Piece가 ChessBoard를 의존하는 것을 제거하고, ChessBoard와 Piece의 의존 관계를 외부(RouteSearchStrategy)에서 연결하게 되었다. 이로써 순환 참조를 제거할 수 있게 되었다.

<br/>

사실 외부에서 의존 관계를 연결해주는 역할은 Factory Bean의 역할과 유사하다고 생각한다. 예를 들어,

<img src="https://tjdtls690.github.io/assets/img/blog/chess_004.PNG">

밑의 구현체들을 모은 패키지는 인터페이스를 구현한 것이기에 당연히 Interface 패키지에 대한 의존성이 생긴다. 문제는 인터페이스 패키지에서도 서로 다른 인터페이스의 구현체들에 대한 의존성을 가질 수 있게 되는 것이다. 이때, FactoryBean의 역할은,

<img src="https://tjdtls690.github.io/assets/img/blog/chess_005.PNG">

이렇게 FactoryBean에서 두 패키지에 대한 의존성을 연결시켜주는 역할을 맡는다. 그러므로 순환 참조를 제거해줄 수 있다. 아직 구체적으로 전부 파악한 것은 아니지만, 형태로나마 어떤 형식으로 순환 참조 제거가 되는 지 조금은 이해가 되는 것 같다.