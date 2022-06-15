# 목차

1. [Scope](#1-scope) <br/>
2. [call by value](#2-call-by-value) <br/>
3. [call by reference](#3-call-by-reference) <br/>
4. [call by value VS call by reference](#4-call-by-value-와-call-by-reference-의-장단점) <br/>

<br/>

# [Java, C] call by value VS call by reference


<span style = " font-size:1.2em; color: blue; font-weight: bold; ">만약 이전 글을 안읽었다면, 읽고 이번 글을 읽기 바란다.</span><br/>
<span style = " font-size:1.2em; color: blue; font-weight: bold; ">훨씬 더 이해가 잘 될 것이다. (바로 밑의 링크)</span>

**이전 글 :** **[[Java, C] 배열의 본질, 포인터와 배열의 관계](https://bit.ly/3QgG4XR){:target="_blank"}**



> **이전 글에서 포인터와 참조변수의 관계에 대해 이해했다면,**
>
> **이번에 공부해 볼 call by value 와 call by reference 의 차이점과 활용법에 대해서도**
>
> **쉽게 접근이 가능할 것이라 믿는다.**
>
> **call by value 와 call by reference 는 무엇이고, 그 차이점은 무엇일까??**



<span style = " font-size:1.5em; color: red; font-weight: bold; "> 일단 Scope 의 개념부터 알아야 한다.</span>



# 1. Scope

- ### Scope 는 무엇일까??

  - <code><strong>직역하면 '범위' 라는 뜻이다.</strong></code>

- ### Scope 의 개념

  - <code><strong>변수는 선언된 블록 내에서만 접근할 수 있다.</strong></code>
    
    - 즉, 자신이 선언된 블록 내에서만 생존이 가능하다.
    
    - **메서드 (함수) 간** <code><strong>'작업의 완벽한 분담'을 위해</strong></code> scope 개념이 존재하는 것이다.
    
      ```java
      public class Test {
          public static void main(String[] args) {
              int a = 10;
              int b = 20;
      
              System.out.println(a + " " + b);
      
              Test t = new Test();
              t.printNum();
      
      //      System.out.println(c); // 에러
          }
      
          public void printNum(){
              int c = 30;
              
      //      System.out.println(a + " " + b); // 에러
              System.out.println(c);
          }
      }
      
      // 출력 결과
      // 10 20
      // 30
      ```
    
      
    
      - 위의 예제에서, main() 메서드와 printNum() 메서드는 서로 전혀 영향을 끼치지 못한다.
        1. printNum() 메서드에서는 main() 메서드 안에 선언된 변수 a, b에 접근할 수 없다.
    
        2. main() 메서드에서도 printNum() 메서드 안에 선언된 변수 c 에 접근할 수 없다.
    
    
    
    

<span style = " font-size:1.5em; color: red; font-weight: bold; "> 이제 본격적으로 call by value 부터 알아보자.</span>



# 2. call by value

- ### call by value 는 무엇일까??

  - 직역하면 <code><strong>'값에 의한 호출'</strong></code> 이란 뜻이다.

- ### call by value 의 개념

  - **메서드를 호출할 때,** <code><strong>값만을 복사해서 호출하는 것</strong></code>
  - 즉, 변수 안에 들어있는 값만 복사해서 넘겨주는 것이다.
    - 변수 그 자체(주소값) 를 넘겨주는 것이 아니다.
  - <code><strong>호출할 메서드의 매개변수가'기본형 변수'일 때 call by value 현상이 일어난다.</strong></code>

```java
public class Test {
    public static void main(String[] args) {
        Test t = new Test();
        int a = 10;
        int b = 20;
        t.printNum(a, b);

        a++; b++;
        System.out.println(a + " " + b);
    }

    public void printNum(int a, int b){
        a = 5;
        b = 10;
        System.out.println(a + " " + b);
    }
}

// 출력 결과
// 5 10
// 11 21
```



- 위 예제가 **어떤식으로 동작되는지 하나씩 풀어서** 보도록 해보자.

  1. 호출 스택에 **main() 메서드가 들어오고 실행**된다.

  2. main() 메서드에서 Test 인스턴스를 생성 후, 참조변수 t에 받는다.

  3. 변수 a, b 에 각각 **10, 20 을 대입**한다.

  4. 참조변수 t를 통해 **printNum() 메서드를 호출**한다.

     - 호출과 동시에 인자로 각각 **10, 20 이 대입된 변수 a, b 를 전달**한다.

  5. 호출 스택에 main() 메서드보다 위에 printNum() 이 들어오게 된다.

  6. **main() 메서드는** printNum() 메서드가 끝날 때까지 **printNum() 메서드를 호출한 지점에서 대기**한다.

  7. printNum() 안에서 인자로 전달받은 **매개변수 a, b 에 각각 5, 10 을 대입**한다.

  8. printNum() 메서드에서 먼저 **a, b 를 출력**한다.

  9. printNum() 메서드가 종료되고 호출 스택에서 printNum() 메서드가 빠져나온다.

     - 여기서 printNum() 메서드 안의 변수 a, b 도 수명을 다하고 없어지게 된다.

  10. 대기하고 있던 main() 메서드가 **printNum() 메서드를 호출한 지점 이후부터 다시 이어서 진행**한다.

  11. main() 메서드에서 **a, b 를 1씩 증가**시킨다.

  12. **a, b 를 출력**한다.

  13. 호출 스택에서 main() 메서드가 없어지며 프로그램이 종료된다.

<br/>
      

- <span style = " font-size:1.2em; color: blue; font-weight: bold; "> main() 메서드의 a, b 랑 printNum() 메서드의 a, b 는 같은 변수 아니였어?? 왜 완전히 다른 값으로 출력이 될까??</span>

  1. 분명히 main() 메서드의 변수 **a, b 를 printNum() 메서드에 인자로 전달**을 했다.
     - 이러면 main() 의 a, b 랑 printNum() 메서드의 a, b 가 같은 값을 공유하는 같은 변수인게 아니었나..?

  2. printNum() 에서 그 받은 인자 a, b 에 각각 **5, 10 을 새로 대입**했다.
  3. 그런데 출력은 printNum() 메서드에선 **5, 10** 그리고 main() 메서드에선 **11, 21** 이 출력되었다.
  4. <code><strong>만약 main()와 printNum() 메서드의 변수 a, b가 서로 같은 것이라면,</strong></code> main() 메서드의 출력이 6, 11 이 나와야 했을 것이다.
     - printNum() 에서 a, b 에 **5, 10** 을 넣은 시점에 main() 메서드의 a, b 도 **5, 10** 이 됐을 것이다.
     - 그리고, 마지막 출력 전에 a, b 를 1씩 증가를 시켜줬으니 **6, 11** 이 출력이 되어야 맞다.
     - 하지만 실제로는 **5, 10** 그리고 **11, 21** 이라는 **완전히 다른 수가 출력**되었다.

  <br/>

- 위 예제를 통해 알 수 있는 사실은

  - <span style = " font-size:1.3em; color: blue; font-weight: bold; "> main() 메서드의 a, b 와 printNum() 메서드의 a, b 는 완전 별개의 변수라는 것이다.</span>

    - main() 에서 printNum() 메서드에 인자로 전달한 것은, 변수 **a, b 의 주소가 아닌** <code><strong>그 안의 값을 전달해준 것</strong></code> 뿐이다.

    - 한 사람이 전혀 다른 사람에게 **같은 숫자 카드를 복사해서 나눠준 것뿐**, 서로 **아예 다른 사람**이다.

    - <code><strong>이것이 call by value 다.</strong></code>

      

<span style = " font-size:1.5em; color: red; font-weight: bold; "> 이제 call by reference 를 알아보자.</span>



# 3. call by reference

- ### call by reference 는 무엇일까?

  - 직역하면 <code><strong>'참조에 의한 호출'</strong></code> 란 뜻이다.

- ### call by reference 의 개념

  - <code><strong>값을 갖고 있는 '변수의 주소'를 넘기는 것</strong></code>

  - <code><strong>호출할 메서드의 매개변수가'참조 변수'일 때 call by reference 현상이 일어난다.</strong></code>

    - 매개변수로 전달해 준 쪽과 전달 받은 쪽의 객체(참조변수) 둘 다, 완전히 같은 주소를 가리키는 같은 객체(참조변수)가 된다.

    ```java
    public class Test {
        public static void main(String[] args) {
            int[] arr = {10, 50};
            System.out.println("main 1 : " + arr[0] + " " + arr[1]);
    
            Test t = new Test();
            t.swap(arr);
            System.out.println("main 2 : " + arr[0] + " " + arr[1]);
        }
    
        public void swap(int[] arr){
            int tmp = arr[0];
            arr[0] = arr[1];
            arr[1] = tmp;
    
            System.out.println("swap : " + arr[0] + " " + arr[1]);
        }
    }
    
    // 출력 결과
    // main 1 : 10 50
    // swap : 50 10
    // main 2 : 50 10
    ```

    

  - 위 예제가 어떤식으로 동작되는지 하나씩 풀어서 보도록 해보자.

    1. **호출 스택에 main() 메서드**가 들어가면서 프로그램이 실행된다.

    2. 배열 arr 을 **인덱스 0에 10, 인덱스 1에 50** 을 넣으며 생성한다.

    3. arr[0], arr[1] 을 출력한다.

    4. 해당 클래스의 객체를 생성하여 swap() 메서드를 호출한다.

       - 호출하면서, <code><strong>참조변수 arr 의 주소값을 인자로 넘긴다.</strong></code>

    5. 호출 스택에 swap() 메서드가 들어가면서 실행된다.

    6. swap() 메서드 안에서 <code><strong>매개변수로 전달 받은 주소</strong></code>를 통해, arr[0], arr[1] 의 값을 서로 바꾼다.

    7. arr[0], arr[1] 을 출력한다.

    8. 호출 스택에서 swap() 메서드가 사라지면서 swap() 메서드는 종료된다.

    9. main() 메서드에서 arr[0], arr[1] 을 한번 더 출력한다.

    10. 호출 스택에서 main() 메서드가 사라지면서 프로그램이 종료된다.

<br/>

- <span style = " font-size:1.3em; color: blue; font-weight: bold; "> 어떻게 swap() 메서드에서 값을 바꾼 것이 main() 메서드에 영향을 끼칠 수 있는가??</span>

  - main() 메서드에서 swap() 메서드에 **참조변수 arr** 을 넘길 때, <code><strong>arr의 주소값을 넘긴다.</strong></code>

  - 그래서, **main() 메서드의 arr 과 swap() 메서드의 arr 은** <code><strong>같은 주소를 바라보게 된다.</strong></code>

  - <span style = " font-size:1.2em; color: blue; font-weight: bold; "> 그렇기 때문에, main() 메서드의 arr 과 swap() 메서드의 arr 은 완전히 같은 객체가 된다.</span>

    

<span style = " font-size:1.5em; color: red; font-weight: bold; "> 이제 call by value 와 call by reference 의 차이를 알아보자.</span>



# 4. call by value 와 call by reference 의 장단점

- <span style = " font-size:1.5em; color: black; font-weight: bold; "> call by value</span>

  - <span style = " font-size:1.3em; color: black; font-weight: bold; ">장점</span>

    - 서로 관여하지 않는 <code><strong>완벽한 분업</strong></code>을 할 수 있다.
      - 서로 관여하지 않기에, 어떤 기능을 짤 때 <code><strong>다른 기능들을 신경쓰지 않아도 된다.</strong></code>

  - <span style = " font-size:1.3em; color: black; font-weight: bold; ">단점</span>

    - **서로 관여하지 않기 때문에 불편한 경우**가 생긴다.

      - 예를 들어, main() 메서드와 swap() 메서드가 있다고 칠 때, 

      - main() 메서드에서 swap() 메서드를 호출할 때, 두 개의 인자를 넘긴다 해도 

      - swap() 메서드에선 main() 메서드의 <code><strong>두 변수의 값이 서로 바뀌게 할 수 없다.</strong></code>

        

- <span style = " font-size:1.5em; color: black; font-weight: bold; ">call by reference</span>

  - <span style = " font-size:1.3em; color: black; font-weight: bold; ">장점</span>

    - 서로 다른 메서드에서 <code><strong>서로간의 변수를 컨트롤 할 수 있다.</strong></code>
    - 메서드의 **역할만 제대로 정의** 되어있고, **그 정의만 벗어나지 않으면** <code><strong>오히려 더 좋은 코드가 될 수 있다.</strong></code>
      - ex) 'call by reference 파트' 예제에서 **swap()** 과 같은 기능들

  - <span style = " font-size:1.3em; color: black; font-weight: bold; ">단점</span>

    - <code><strong>메서드 (함수) 의 철학을 벗어난다.</strong></code> (= <code><strong>'완벽한 분업'</strong></code>**을 벗어나는 행위**)

      - 서로간의 메서드가 **간섭하는 상황이 오게되면,** <code><strong>유지보수 측면</strong></code>에서도 좋지 않다.

      - **'완벽한 분업' 이 되게끔 디자인** 하는 것이 좋다.
      
<br/>

- <span style = " font-size:1.3em; color: blue; font-weight: bold; "> 가능하면 call by reference 는 쓰지 않는 것이 좋다. (함수의 철학 위배)</span>

  - <span style = " font-size:1.3em; color: blue; font-weight: bold; "> 그러나 필요상황에 맞춰서 잘 쓰면 OK. </span>

<br/>


<span style = " font-size:1.5em; color: red; font-weight: bold; "> 결론</span>

- <span style = " font-size:1.3em; color: blue; font-weight: bold; ">call by value : 값을 복사하여 함수에게 넘기므로, 서로 영향을 주지 않는다. </span>
- <span style = " font-size:1.3em; color: blue; font-weight: bold; "> call by reference : 포인터(주소) 를 넘기는 것이므로 서로 영향을 줄 수 있다. </span>
