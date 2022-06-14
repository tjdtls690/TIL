# 목차

1. scope
2. call by value
3. call by reference
4. call by value VS call by reference

<br/><br/>

# 1. Scope

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
    
    
    
    
<br/>
<br/>



# 2. call by value

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



<br/>

  - <span style = " font-size:1.3em; color: orange; font-weight: bold; "> main() 메서드의 a, b 와 printNum() 메서드의 a, b 는 완전 별개의 변수다.</span>

    - main() 에서 printNum() 메서드에 인자로 전달한 것은, 변수 **a, b 의 주소가 아닌** <code><strong>그 안의 값을 전달해준 것</strong></code> 뿐이다.

    - 한 사람이 전혀 다른 사람에게 **같은 숫자 카드를 복사해서 나눠준 것뿐**, 서로 **아예 다른 사람**이다.

    - <code><strong>이것이 call by value 다.</strong></code>

      

<br/>
<br/>



# 3. call by reference

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

    

  

<br/>

- <span style = " font-size:1.3em; font-weight: bold; "> 어떻게 swap() 메서드에서 값을 바꾼 것이 main() 메서드에 영향을 끼칠 수 있는가??</span>

  - main() 메서드에서 swap() 메서드에 **참조변수 arr** 을 넘길 때, <code><strong>arr의 주소값을 넘긴다.</strong></code>

  - 그래서, **main() 메서드의 arr 과 swap() 메서드의 arr 은** <code><strong>같은 주소를 바라보게 된다.</strong></code>

  - <span style = " font-size:1.2em; color: orange; font-weight: bold; "> 그렇기 때문에, main() 메서드의 arr 과 swap() 메서드의 arr 은 완전히 같은 객체가 된다.</span>

    

<br/><br/>



# 4. call by value 와 call by reference 의 장단점

- <span style = " font-size:1.5em; font-weight: bold; "> call by value</span>

  - <span style = " font-size:1.3em; font-weight: bold; ">장점</span>

    - 서로 관여하지 않는 <code><strong>완벽한 분업</strong></code>을 할 수 있다.
      - 서로 관여하지 않기에, 어떤 기능을 짤 때 <code><strong>다른 기능들을 신경쓰지 않아도 된다.</strong></code>

  - <span style = " font-size:1.3em; font-weight: bold; ">단점</span>

    - **서로 관여하지 않기 때문에 불편한 경우**가 생긴다.

      - 예를 들어, main() 메서드와 swap() 메서드가 있다고 칠 때, 

      - main() 메서드에서 swap() 메서드를 호출할 때, 두 개의 인자를 넘긴다 해도 

      - swap() 메서드에선 main() 메서드의 <code><strong>두 변수의 값이 서로 바뀌게 할 수 없다.</strong></code>

        

- <span style = " font-size:1.5em; font-weight: bold; ">call by reference</span>

  - <span style = " font-size:1.3em; font-weight: bold; ">장점</span>

    - 서로 다른 메서드에서 <code><strong>서로간의 변수를 컨트롤 할 수 있다.</strong></code>
    - 메서드의 **역할만 제대로 정의** 되어있고, **그 정의만 벗어나지 않으면** <code><strong>오히려 더 좋은 코드가 될 수 있다.</strong></code>
      - ex) 'call by reference 파트' 예제에서 **swap()** 과 같은 기능들

  - <span style = " font-size:1.3em; font-weight: bold; ">단점</span>

    - <code><strong>메서드 (함수) 의 철학을 벗어난다.</strong></code> (= <code><strong>'완벽한 분업'</strong></code>**을 벗어나는 행위**)

      - 서로간의 메서드가 **간섭하는 상황이 오게되면,** <code><strong>유지보수 측면</strong></code>에서도 좋지 않다.

      - **'완벽한 분업' 이 되게끔 디자인** 하는 것이 좋다.
      

<br/>

- <span style = " font-size:1.3em; color: orange; font-weight: bold; "> 가능하면 call by reference 는 쓰지 않는 것이 좋다. (함수의 철학 위배)</span>

  - <span style = " font-size:1.3em; color: orange; font-weight: bold; "> 그러나 필요상황에 맞춰서 잘 쓰면 OK. </span>

<br/>


<span style = " font-size:1.5em; color: white; font-weight: bold; "> 결론</span>

- <span style = " font-size:1.3em; color: orange; font-weight: bold; ">call by value : 값을 복사하여 함수에게 넘기므로, 서로 영향을 주지 않는다. </span>
- <span style = " font-size:1.3em; color: orange; font-weight: bold; "> call by reference : 포인터(주소) 를 넘기는 것이므로, 서로 영향을 줄 수 있다. </span>