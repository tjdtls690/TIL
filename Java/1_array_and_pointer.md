# 목차

1. [포인터란 무엇인가?](#1-포인터란-무엇인가) <br/>
2. [void mallocint(x) 함수는 무엇인가](#2-void-mallocint-x-함수는-무엇인가) <br/>
3. [포인터와 배열의 관계](#3-포인터와-배열의-관계) <br/>

<br/>

# 배열의 본질, 포인터와 배열의 관계

> **나의 주 언어는 자바지만, 배열의 본질을 공부하는 데엔  이 정도로 좋은 접근법은 없다고 생각한다.**
>
> **그래서 C언어의 포인터를 통해 배열의 본질을 알아보는 시간을 가졌다.**
>
> **주 언어가 자바이다보니 조금씩 정확하지 않은 부분이 있을 수 있다.** 
>
> **그러한 부분들은 피드백 부탁한다.**



<span style = " font-size:1.5em; color: red; font-weight: bold; "> 일단 포인터가 무엇인지부터 알아야한다.</span>



# 1. 포인터란 무엇인가?

- C언어엔 <code><strong>'일반 변수'</strong></code>와 <code><strong>'포인터 변수'</strong></code>가 존재한다.

  - <code><strong>일반 변수 :</strong></code> 데이터를 담는 그릇
  - <code><strong>포인터 변수 : </strong></code> 특정 변수의 위치(주소)를 담는 그릇 (특정 변수를 가리키는 변수)

- 포인터 변수를 선언할 땐, 자료형 뒤에 * (애스터리스크) 가 붙는다.

- 주의사항 : 애스터리스크가 **자료형 뒤에 붙을 때**와 **변수명 앞에 붙을 때**의 의미가 달라진다.

  - <code><strong>int* aPointer :</strong></code> 특정 int형 변수를 가키리는 포인터변수 aPointer (**특정 int형 변수의 '주소값'**을 가지는 포인터변수 aPointer)
  - <code><strong>*aPointer :</strong></code> **aPointer가 가리키는 변수의 값**을 가져온다.
    - <code><strong>aPointer :</strong></code> 포인터 변수는 *을 붙이지 않으면 자신이 가리키는 변수의 값이 아닌, **자신이 가리키는 변수의 위치(주소)값**을 꺼내게 된다.
  - <code><strong>&a :</strong></code> 일반변수 앞에 &(앰퍼센트) 를 붙여서, a가 가지고있는 값이 아닌 a의 위치 즉, **a의 주소값**을 꺼내게 된다.

  ```c
  int a = 10; // 일반 변수 a (5라는 데이터가 들어있다.)
  int* aPointer = &a; // 포인터 변수 aPointer (일반변수 a의 주소(위치)값이 들어있다.)
  printf("%d", *aPointer); // *포인터변수명 : 포인터 변수가 가리키고있는 일반변수 a의 값을 가져온다.
  
  // %d : 10진수의 정수형 출력
  // %p : 포인터의 주소 출력
  
  // 출력 결과
  // 10
  ```

  

- 위의 예제가 메모리상에서 어떤식으로 작동되는지 하나씩 풀어보자.
  1. 메모리(RAM)엔 각각 **1byte (8bit)** 의 데이터가 들어갈 수 있는 방들이 존재한다.
  2. 그리고 각 방의 위치를 가리키는 **주소값들이 존재**한다.
  3. 일반변수 a는 자료형이 **int** 이므로, 메모리에서 **연속적인 4개의 방(4byte)** 을 차지하게 된다.
  4. 그 4개의 방을 전부 활용해서 이진수의 형태로 십진수 10을 저장하게 된다.
     - **1번째 방 : 0 0 0 0 1 0 1 0    ->    주소값 : 0x7fffc8e3d660**
     - **2번째 방 : 0 0 0 0 0 0 0 0    ->    주소값 : 0x7fffc8e3d661**
     - **3번째 방 : 0 0 0 0 0 0 0 0    ->    주소값 : 0x7fffc8e3d662**
     - **4번째 방 : 0 0 0 0 0 0 0 0    ->    주소값 : 0x7fffc8e3d663**
  5. 포인터변수 aPointer는 주소값을 저장해야하기에, 주소값의 크기인 8개의 방(8byte) 을 차지하게 된다.
     - 지금은 보통 64bit 운영체제를 사용하기에 주소값의 크기는 8byte가 된다.
     - 하지만, 64bit 를 다 쓸 필요가 없기에, 2개의 방은 전부 0으로 차고, 48bit(6개의 방) 만 사용한다고 한다.
       - 그런식으로 8개의 방을 활용해서 일반변수 a의 첫번째 방의 주소값인 7fffc8e3d660 (16진수) 를 이진수의 형태로 저장한다.
         - ​&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(2진수)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (16진수 : a의 첫번째 주소값)
         - **1번째 방 : 0 1 1 0 0 0 0 0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6 0**
         - **2번째 방 : 1 1 0 1 0 1 1 0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;d 6**
         - **3번째 방 : 1 1 1 0 0 0 1 1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e 3**
         - **4번째 방 : 1 1 0 0 1 0 0 0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;c 8**
         - **5번째 방 : 1 1 1 1 1 1 1 1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;f f**
         - **6번째 방 : 0 1 1 1 1 1 1 1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7 f**
         - **7번째 방 : 0 0 0 0 0 0 0 0**
         - **8번째 방 : 0 0 0 0 0 0 0 0**
  6. 5번처럼, 포인터변수 aPointer 엔 일반변수 **a의 제일 첫번째 주소값**인 **0x7fffc8e3d660** 을 가진다.
  7. 포인터변수 aPointer 앞에 *(애스터리스크) 를 붙임으로써, aPointer 가 가지고있는 주소**(변수 a의 위치) 에 있는 값**을 가져온다.
  8. 일반변수 a의 값인 10을 출력한다.



<span style = " font-size:1.5em; color: red; font-weight: bold; "> 이제 포인터변수에게 메모리의 공간을 수동으로 할당해주는 malloc() 함수를 알아야한다.</span>



# 2. void* malloc(int x) 함수는 무엇인가?

- **x byte 의 공간을 할당하여 1번째 방의 주소를 반환하는 함수**

  - 어차피 C언어에 관한 것이기에 어떤 기능을 가지는 함수인지만 알고 넘어가자.

  ```c
  // 8byte 만큼 메모리를 할당해서 8개의 방중 첫번째 방의 주소값을 포인터변수 x에 반환한다.
  int* x = (int*)malloc(8);
  ```

  

- 여기서 드는 의문점 : **int 자료형은 4byte 크기**인데 8byte를 할당해서 주면 **남은 4byte는 낭비**되는 것이 아닌가??

  - 결론부터 말하면, <code><strong>8byte 전부 활용 가능</strong></code>하다.

  ```c
  int* x = (int*)malloc(8);
  
  // x가 가리키는 주소의 1번째 방에 10을 넣는다.
  // 1 ~ 4번째 방까지 활용해서 십진수 10을 이진수의 형태로 저장한다.
  *x = 10;
  
  // x가 가리키는 주소의 5번째 방에 20을 넣는다.
  // 5 ~ 8번째 방까지 활용해서 십진수 20을 이진수의 형태로 저장한다.
  *(x + 1) = 20;
  ```

  

  - 위의 예제가 메모리상에서 어떤식으로 작동되는지 하나씩 풀어보자.

    1. 메모리에서 **8개의 방(8byte) 만큼 할당**을 해서 **그 중 1번째 방의 주소값**을 포인터변수 x에 반환한다.

    2. 1 ~ 4번째 방을 활용해서 10을 저장한다.

    3. 5 ~ 8번째 방을 활용해서 20을 저장한다.

       - 여기서 주의할 점은, *(x + 1) 이 2번째 방을 가리키는 것이 아닌, **5번째 방**을 가리킨다는 것이다.

       - 이게 가능한 이유는, 저장할 데이터의 자료형이 **int형(4byte) 이라는 것을 컴퓨터가 인지**하고 있기 때문이다.

         - 그렇기에, 알아서 4개의 방씩 건너뛰는 것이다.

         

  - 여기서 **sizeof() 함수**를 활용해보면

    ```c
    // sizeof(int) : int의 크기 즉, 4를 반환한다. (기준 단위 byte)
    // sizeof(int) * 5 : int 자료형의 크기 5개를 뜻한다.
    // sizeof(int) * 5   =>   4 * 5   =>   20 byte
    // 포인터 변수 x에 20byte 의 메모리공간을 할당해서 20개의 방 중 1번째 방의 주소를 반환한다.
    int* x = (int*)malloc(sizeof(int) * 5);
    ```

    - 위의 예제는 결과적으로, 포인터변수 **x에 int 타입 5개의 변수를 저장할 수 있는 메모리 공간을 할당해주는 것**이다.

      

  - 여기까지 이해했다면, 다음의 예제도 이해하기 쉬울 것이다.

    ```c
    // 메모리 공간에서 int 자료형의 크기 5개를 할당한다.
    int* x = (int*)malloc(sizeof(int) * 5);
    
    // int형 데이터 5개를 순차적으로 대입한다.
    *x = 10;
    *(x + 1) = 20;
    *(x + 2) = 30;
    *(x + 3) = 40;
    *(x + 4) = 50;
    
    // %d : 10진수의 정수형 출력
    // %p : 포인터의 주소 출력
    
    // 대입한 int형 데이터 5개를 순차적으로 출력한다.
    printf("%d\n", *x);
    printf("%d\n", *(x + 1));
    printf("%d\n", *(x + 2));
    printf("%d\n", *(x + 3));
    printf("%d\n\n", *(x + 4));
    
    // 대입한 int형 데이터 5개의 각각의 1번째방 주소를 출력한다.
    // 메모리 공간에서 연속적인 공간인 것과, 
    // 저장된 데이터들이 int형임을 잘 인지하고 있다면,
    // 주소가 4씩 건너뛸 것을 충분히 예상할 수 있을 것이다.
    printf("%p\n", x);
    printf("%p\n", x + 1);
    printf("%p\n", x + 2);
    printf("%p\n", x + 3);
    printf("%p\n", x + 4);
    
    // 출력 결과
    // 10
    // 20
    // 30
    // 40
    // 50
    //
    // 0xe15010
    // 0xe15014
    // 0xe15018
    // 0xe1501c
    // 0xe15020
    ```



<span style = " font-size:1.5em; color: red; font-weight: bold; "> 근데 이거 어디서 많이 본 모양새인데?? 설마 배열??</span>



# 3. 포인터와 배열의 관계

- 자바를 배운 상태라면, **배열은** '기본형 변수'가 아닌, <code><strong>'참조 변수'</strong></code>에 대입하는 것을 알고 있을 것이다.

  ```java
  // 메모리 공간에서 int 크기 * 5 즉, 4 * 5 => 20 byte를 할당하고,
  // 그 중, 1번째 방의 주소값을 참조변수 arr 에 반환한다.
  int[] arr = new int[5];
  ```

  

- C언어도 마찬가지다.

  ```c
  // 메모리 공간에서 int 크기 * 5 즉, 4 * 5 => 20 byte를 할당하고,
  // 그 중, 1번째 방의 주소값을 참조변수 arr 에 반환한다.
  int arr[5];
  ```

  

- 여기서 소름돋는 사실이 하나 있다.

  ```c
  // 1번
  
  int* x = (int*)malloc(sizeof(int) * 5);
  
  *x = 10;
  *(x + 1) = 20;
  *(x + 2) = 30;
  *(x + 3) = 40;
  *(x + 4) = 50;
  
  printf("%d\n", *x);
  printf("%d\n", *(x + 1));
  printf("%d\n", *(x + 2));
  printf("%d\n", *(x + 3));
  printf("%d\n\n", *(x + 4));
  printf("%p\n", x);
  printf("%p\n", x + 1);
  printf("%p\n", x + 2);
  printf("%p\n", x + 3);
  printf("%p\n", x + 4);
  
  
  
  // 2번
  
  int x[5];
  
  x[0] = 10;
  x[1] = 20;
  x[2] = 30;
  x[3] = 40;
  x[4] = 50;
  
  printf("%d\n", x[0]);
  printf("%d\n", x[1]);
  printf("%d\n", x[2]);
  printf("%d\n", x[3]);
  printf("%d\n\n", x[4]);
  printf("%p\n", &x[0]);
  printf("%p\n", &x[1]);
  printf("%p\n", &x[2]);
  printf("%p\n", &x[3]);
  printf("%p\n", &x[4]);
  ```

  - 위 예제의 1번과 2번 예제가 <code><strong>완전히 똑같은 의미의 코드</strong></code>라는 것이다.

    

<span style = " font-size:1.5em; color: red; font-weight: bold; "> 내 수준이 아직 낮아서 나만 소름돋는 것일 수 있겠다는 생각도 든다...</span>



- 위 예제의 1번과 2번 예제를 **섞어서 쓰는 것도 가능**하다.

  - 데이터 값과 주소값이 모두 동일하게 출력되는 것을 확인할 수 있다.

  ```c
  // 배열 선언
  int x[5];
  
  // 배열의 각 요소에 데이터 대입
  *x = 10;
  *(x + 1) = 20;
  *(x + 2) = 30;
  *(x + 3) = 40;
  *(x + 4) = 50;
  
  // 데이터와 주소값 출력
  printf("%d ", *x);
  printf("%d ", *(x + 1));
  printf("%d ", *(x + 2));
  printf("%d ", *(x + 3));
  printf("%d\n\n", *(x + 4));
  
  printf("%p\n", x);
  printf("%p\n", x + 1);
  printf("%p\n", x + 2);
  printf("%p\n", x + 3);
  printf("%p\n\n", x + 4);
  
  printf("%d ", x[0]);
  printf("%d ", x[1]);
  printf("%d ", x[2]);
  printf("%d ", x[3]);
  printf("%d\n\n", x[4]);
  
  printf("%p\n", &x[0]);
  printf("%p\n", &x[1]);
  printf("%p\n", &x[2]);
  printf("%p\n", &x[3]);
  printf("%p\n", &x[4]);
  
  // 출력 결과
  // 10 20 30 40 50
  //
  // 0x7ffd6ea38910
  // 0x7ffd6ea38914
  // 0x7ffd6ea38918
  // 0x7ffd6ea3891c
  // 0x7ffd6ea38920
  // 
  // 10 20 30 40 50
  // 
  // 0x7ffd6ea38910
  // 0x7ffd6ea38914
  // 0x7ffd6ea38918
  // 0x7ffd6ea3891c
  // 0x7ffd6ea38920
  ```



<span style = " font-size:1.5em; color: red; font-weight: bold; ">이번에 공부하면서 자바에 빗대어 생각해볼 때, 사실 배열뿐만이 아닌 모든 참조변수는 결국 C언어의 포인터와 일맥상통하다고 느꼈다.</span>

<span style = " font-size:1.5em; color: red; font-weight: bold; ">여기서 한 발자국만 더 나아가보자. (밑의 링크)</span>


**다음 글 :** **[[Java, C] call by value   VS   call by reference](https://bit.ly/3O9YyYm){:target="_blank"}**