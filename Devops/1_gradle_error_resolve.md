# 목차

1. [에러가 났던 코드](#1-에러가-났던-코드) <br/>
2. [에러가 난 이유](#2-에러가-난-이유) <br/>
3. [에러 해결 방법](#3-에러-해결-방법) <br/>

<br/>

# gradle 빌드 추가 시 - Could not find method compile 해결 방법

<br/>

> **TDD 설계를 연습하기 위해 프로그래밍을 하는 도중 StringUtils 라이브러리를 사용해야할 일이 생겨서 의존성 추가를 하려고 했다.**
>
> **근데 Could not find method compile() 에러가 뜨면서 빌드가 진행되지를 않는다.**
>
> **실은 국비지원 학원에선 Maven 으로만 프로젝트를 진행했었기 때문에 Gradle 에 대해 정확하게 모르고 있는 것이 문제였다.**
>
> **이 문제를 해결하면서 공부한 Gradle 관련 내용을 간단하게 정리해보고자 한다.**

<br/>

## 1. 에러가 났던 코드

- ```groovy
  compile "org.apache.commons:commons-lang3:3.12.0"
  ```

- 분명 구글 검색을 해보고 적었기에 당연히 될 줄 알았는데 Could not find method compile() 에러가 뜨면서 안된다...



## 2. 에러가 난 이유

- <code><strong>compile 은 Gradle 4.10 (2018.8.27) 이래로 deprecate 되었고 Gradle 7.0 (2021.4.9) 부터 완전 삭제되었다.</strong></code>

  - 더 정확히 말하자면 <code><strong>compile, runtime, testCompile, testRuntime</strong></code> 가 삭제되었다.
  - 참고로 필자는 Gradle 7.0.2 를 사용하고 있었다.

- 그럼 어떤 걸로 바뀐것인가??

  - **compile -> implementation**
  - **runtime -> runtimeOnly**
  - **testCompile -> testImplementation**
  - **testRuntime -> testRuntimeOnly**

  

## 3. 에러 해결 방법

- ```groovy
  implementation "org.apache.commons:commons-lang3:3.12.0"
  ```

- **너무나 간단한 문제였어서 공부가 중요하다는 것을 더욱 깨닫는 계기가 되었다..**

