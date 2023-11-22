# 목차

1. [Layered Architecture](#layered-architecture) <br/>
2. [Service Layer의 필요성](#service-layer의-필요성) <br/>
3. [Service Layer Test](#service-layer-test) <br/>
4. [여러 Dao(Repository)? 혹은 여러 Service?](#여러-daorepository-혹은-여러-service) <br/>

<br/>

# [Java, Spring] Service Layer는 무엇이고 어떻게 활용해야 하는가?

> **예전 국비 학원에서 메인 프로젝트를 진행했을 때, Service Layer의 역할을 굉장히 모호하게 알고 있었다.**
>
> **Presentation Layer와 Persistance Layer간의 데이터 통로 역할로만 사용하며 그것에 의의를 두었었다.**
>
> **Controller에 모든 비즈니스 로직을 때려 박고,** 
>
> **Service Layer는 Persistence Layer에 데이터를 전달하고 전달 받는 로직만 존재했던 지난날을 반성하며**
>
> **Service Layer의 역할과 활용법에 대해 간단하게나마 정리해보려 한다.**

<br/>

## Layered Architecture

Service Layer에 대해 알려면, 먼저 **Layered Architecture(계층 구조)**를 이해해야 한다. 

Layered Architecture는 코드를 모듈화하고 재사용성을 높여서,  애플리케이션을 쉽게 확장하고 유지보수 하기 위해 사용한다. 즉, 각 계층이 독립적으로 개발되고, 테스트될 수 있으며, 따라서 애플리케이션 전체의 유지보수성이 향상된다.

각 계층은 분명 서로 상호작용이 일어나지만, 다른 계층에서 발생하는 일에 대해선 전혀 몰라도 된다.

![계층 구조](C:\Users\tjdtl\Desktop\계층 구조.PNG)
<img src="https://tjdtls690.github.io/assets/img/blog/layered_architecture.PNG" width="850" height="300">

개인적으로 Layered Architecture에 대해 가장 이해하기 쉬웠던 그림이다.

**Presentation Layer**는 Spring MVC 패턴을 통해 사용자의 요청을 처리하여 응답을 생성하고,

**Persistence Layer(Data Access Layer)**는 데이터의 영속성을 관리하고 저장, 검색, 수정, 삭제 등의 작업을 수행한다.

그럼 **Service Layer**는 무엇을 하는 녀석일까?

<br/>

## Service Layer의 필요성

- **비즈니스 로직의 분리와 중앙화 :** 도메인 객체간의 상호작용을 조정하고, Persistence Layer와 데이터를 주고 받으며, 한 기능 단위의 결과물을 Controller로 전달한다. 즉, Presentation Layer, Persistence Layer, Domain Model과 전부 상호 작용을 하며 핵심 비즈니스 로직을 처리하는 역할을 맡는다.

  비즈니스 로직을 Controller나 Persistence Layer에서 직접 처리하는 경우, 코드가 복잡해지고 유지보수가 어려워진다. 이미 각자의 역할이 있는데 거기에 비즈니스 로직이 또 추가가 되기 때문이다. 그래서 서비스 계층을 도입함으로써 비즈니스 로직을 분리하고 중앙화 하여, 코드의 가독성과 유지보수성이 향상되고 개발 효율성을 높이게 된다.

- **각 계층의 역할 명확화 :** 바로 위의 역할에 연장되는 느낌인데, Controller는 사용자의 요청을 처리하고 응답을 생성하는 역할을, Persistence Layer는 DB와의 상호작용을 담당하는 역할을, Service Layer는 비즈니스 로직과 트랜잭션 관리를 수행하는 역할을 하게 된다. 이렇게 각 계층의 역할이 분명해지면 코드의 가독성과 유지보수성이 향상된다.

- **트랜잭션 관리 :** 각 핵심 기능마다 데이터의 일관성과 무결성을 보장하기 위해 트랜잭션을 관리한다. 하나의 기능을 처리하기 위해 여러 Dao 혹은 Service를 통해 데이터를 다뤄야 할 때, 중간에 어떤 계기로 인해 기능이 중단되어 한 단위 기능의 원자성이 파괴되는 것을 예방하기 위함이다. 또한, 기능 단위로 트랜잭션을 관리할 수 있어서 트랜잭션의 경계를 더 명확히 할 수 있다.

  Controller에서 관리하기엔, 각 트랜잭션의 단위가 너무 커지고 비즈니스 로직 처리와 트랜잭션 처리가 동시에 이루어지지 않아서 유지보수의 어려움이 생길 수 있다. 또한, Transactional은 AOP를 통해 구현되는데, Controller는 보통 인터페이스가 없다. AOP는 Dynamic proxy를 사용하기에 인터페이스가 필요하기에, Controller에 Tracsantional을 적용하기엔 어려움이 따를 수 있다.

  Persistence Layer에서 트랜잭션을 관리하기엔, 이 또한 비즈니스 로직과 데이터의 영속성과 관련된 작업이 혼재될 수 있다. 그리고 트랜잭션 범위의 한계가 너무 좁아져서 각각의 기능 단위를 관리하는 Service Layer에서 트랜잭션 처리를 하기 어렵게 됩니다.

- **코드 재사용성 촉진 :** 서비스 계층은 비즈니스 로직을 캡슐화하므로 애플리케이션의 여러 부분에서 코드 재사용성을 촉진한다. 예를 들어 여러 컨트롤러가 동일한 서비스를 사용하여 공통 작업을 수행하여 코드 중복을 줄일 수 있다.

<br/>

## Service Layer Test

Service Layer의 단위 테스트를 진행할 때, 고민되는 지점이 한 가지 있다. 

> **Mock을 사용해야할까?**

마틴 파울러의 F.I.R.S.T에 따르면 단위 테스트는 

1. **빠르고**
2. **격리되어야 하며**
3. **반복 가능하고**
4. **자체 검증이 가능하며**
5. **제품코드가 작성되기 바로전에 작성되어야 한다.**

따라서 Service 테스트는 **Persistence Layer 객체들을 Mocking하고 상호 작용을 확인만 함으로써** 단위 테스트의 본질을 잊지 말아야 한다고 생각한다. 만약 Mocking을 해주지 않고 테스트를 한다면, 테스트의 성능에 가장 큰 영향을 끼치는 DB와의 상호작용이 일어나게 되어 효율적인 Service Layer의 테스트 방법이 아니라고 생가하기 때문이다.

어차피 모든 Bean을 통해 통합 테스트를 하는 것은 Controller Test에서 이루어지고 있으니, 굳이 Service에서까지 각 계층간의 테스트까지 할 필요가 있을까 싶다.

<br/>

## 여러 Dao(Repository)? 혹은 여러 Service?

하나의 Service 안에서 여러 테이블을 통해 데이터를 다뤄야 할 때, 여러 Dao를 가져야 할까, 여러 Service를 가져야 할까?

먼저 **여러 Dao를 의존하는 경우**를 생각해보자. 만약 가벼운 어플리케이션을 만든다면, 굳이 다른 Service를 통하지 않고 다른 Dao를 의존하여 사용하는 것이 오히려 용이하다고 생각한다. 각각의 기능 단위가 그리 크지 않을 것이기 때문이다.

하지만 어플리케이션의 규모가 커진다면 이는 문제가 될 수 있다. 각각의 기능 단위가 커지고, 그만큼 각 기능마다 다루는 Dao가 많아질 수 있다. 따라서 단일 Service의 책임이 너무 무거워지고, 많은 중복코드가 발생할 수 있으며, 해당 Persistence Layer 객체를 관리하는 Service가 다른 Persistence Layer 객체도 관리하게 되는 경우가 많아져서 각각의 Service의 역할도 모호해질 수 있다고 생각한다.

그러나 **여러 Service를 의존하는 경우**에도 문제가 있을 수 있는데, 가장 주의해야할 부분은 **순환 참조 발생**이라고 생각한다. 각 Service가 서로 의존하다가 생길 수 있는 문제이다. 내 개인적인 생각은 이는 각 Service간의 계층도 나누어주거나 설계를 보완함으로써 충분히 극복할 수 있는 문제라고 생각한다.

혹은 여러 Service의 의존이 아닌 하나의 Controller가 여러 Service를 의존함으로써 해결하는 방법을 주장하는 개발자도 꽤 되어보인다. 하지만 개인적인 생각은 비즈니스 로직을 가짐으로써 트랜잭션 관리를 해주는 Service Layer를 여러개를 사용하게 된다면, 기능의 원자성 보호가 어려워지거나 각 계층의 역할 구분이 모호해질 수 있다고 생각한다.

아니면 Dao와 Service를 적절히 조합하여 의존하는 것은 어떨까? 좀 더 많은 구현을 통해 경험을 해봐야겠다.

<br/>

Service Layer에서 할 수 있는 적절한 유효성 검증은 어떤 것이 있는지도 다루려다가, 이 부분은 다른 계층과 함께 따로 글을 작성하는 것이 좋을 것이란 생각이 들어서 여기서 마친다.