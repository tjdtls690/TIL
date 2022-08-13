# 목차

1. [lombok 이란??](#1-lombok-이란) <br/>
2. [내 문제가 일어났던 이유](#2-내-문제가-일어났던-이유) <br/>
3. [어노테이션 프로세서란??](#3-어노테이션-프로세서란) <br/>
4. [어노테이션 프로세서 문제 해결 방법](#4-어노테이션-프로세서-문제-해결-방법) <br/>

<br/>

# [자바, Java]  Lombok - cannot find symbol 에러 해결 방법

<br/>

> **프로그래밍을 하는 도중, lombok 의 @EqualsAndHashCode 어노테이션을 사용했다.**
>
> **문제는 JUnit 테스트코드에서 계속 cannot find symbol 에러가 난다...**
>
> **클래스에 직접 equals, hashcode 메서드를 오버라이딩 하면 isEqualTo() 테스트가 통과된다.**
>
> **근데 lombok 을 통해 @EqualsAndHashCode 어노테이션을 써서 하면 isEqualTo() 테스트 통과가 안된다.**
>
> **이게 무슨일일까?? 하고 찾아봤는데 결국 해결을 했다.**
>
> **그 과정에서 공부한 내용을 정리해보고자 한다.**
>
> **참고로 기준 IDE는 IntelliJ 이다.**

<br/>

## 1. lombok 이란??

- **Lombok 이란 Java 라이브러리로 반복되는 getter, setter, toString .. 등의 반복 메서드 작성 코드를 줄여주는 <code><strong>코드 다이어트 라이브러리</strong></code> 이다.**
  - 어노테이션을 써서 많이 사용하는 패턴을 직접 작성하지 않아도 되게끔 해준다.



## 2. 내 문제가 일어났던 이유

- <code><strong>롬복 라이브러리는 APT(Annotation Processing Tool)를 통해 어노테이션 프로세서(Annotation Processor)로 컴파일 단계에서 수행하게 된다.</strong></code>

  - 이 사실을 인지한 상태에서 살펴보자.

- 일단 내 원래 gradle 코드를 보자.

  - ```groovy
    dependencies {
        testImplementation "org.junit.jupiter:junit-jupiter:5.7.2"
        testImplementation "org.assertj:assertj-core:3.19.0"
    
        implementation 'org.projectlombok:lombok:1.18.24'
    }
    ```

  - 얼핏 보면 라이브러리 추가도 잘 해줬는데 왜 안되지?? 라는 생각이 든다.

- 롬복을 처음 써보고 이번에 알게된 사실인데, <code><strong>롬복은 라이브러리만 추가해서는 안된다.</strong></code>

  - <code><strong>어노테이션 프로세서(Annotation Processor) 빌드까지 추가해줘야 한다.</strong></code>

    

## 3. 어노테이션 프로세서란??

- **컴파일 시점에 끼어들어 특정한 애노테이션이 붙어있는 소스코드를 참조해서 새로운 소스코드를 만들어 낼 수 있는 기능이다.**
  - 컴파일 단계에서 실행되기 때문에, 빌드 단계에서 에러를 출력하게 할 수 있고, 소스코드 및 바이트 코드를 생성할 수도 있다.

  - 결국 롬복을 사용하기 위해선 설정해줘야 하는 요소이다.

    

## 4. 어노테이션 프로세서 문제 해결 방법

- 1단계 : **롬복 라이브러리 추가**
- 2단계 : **Annotation Processor 설정**
  1. 인텔리제이의 빌드, 테스트 실행 설정이 <code><strong>'gradle'</strong></code> 로 되어있는 경우 (1가지 방법)
     - gradle 빌드 코드에 annotationProcessor 'org.projectlombok:lombok:1.18.24' 추가
  2. 인텔리제이의 빌드, 테스트 실행 설정이 <code><strong>'IntelliJ IDEA'</strong></code> 로 되어있는 경우 (2가지 방법)
     - 1번째 방법 : annotationProcessor 'org.projectlombok:lombok:1.18.24' - 위와 동일
     - 2번째 방법 : 설정 -> 빌드, 실행, 배포 -> 컴파일러 -> 어노테이션 프로세서 -> 어노테이션 처리 활성화 체크



<span style = " font-size:1.5em; color: red; font-weight: bold; ">결론은 annotationProcessor 'org.projectlombok:lombok:1.18.24' 를 빌드에 추가해주는 것이 가장 좋겠다고 정해졌다.</span>
