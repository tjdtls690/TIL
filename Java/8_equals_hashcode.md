# 목차

1. [equals 와 hashcode 의 역할](#1-equals-와-hashcode-의-역할) <br/>
2. [hashcode 의 존재 이유는 무엇인가??](#2-hashcode-의-존재-이유는-무엇인가) <br/>
3. [equals 와 hashcode 는 왜 오버라이딩을 같이 해야하는가??](#3-hash-자료구조에서-equals-와-hashcode-의-동작-원리) <br/>
    1. [해쉬 충돌](#3-1-해쉬-충돌)<br/>
4. [equals 와 hashcode 는 왜 오버라이딩을 같이 해야하는가??](#4-equals-와-hashcode-는-왜-오버라이딩을-같이-해야하는가) <br/>

<br/>

# [자바, Java] equals() 와 hashcode() 를 같이 오버라이딩 하는 이유

<br/>

> **인텔리제이로 프로그램을 짤 때마다 항상 궁금했던 것이,** 
>
> **인텔리제이 단축키로 equals() 메서드를 오버라이딩 하려고 하면**  
>
> **꼭 hashcode() 메서드까지 같이 오버라이딩을 하게 한다.**
>
> **equals 만 써도 전부 비교 가능하다고 생각하는데, 왜 꼭 둘 다 오버라이딩 하게끔 설정되어 있는걸까??**

<br/>

## 1. equals 와 hashcode 의 역할

- 이 두 개의 메서드는 모든 클래스의 조상인 **'Object 클래스'의 메서드**로써, **해당 인스턴스의 메모리 주소값을 이용**한다.

  1. hashcode() 메서드는 그 주소값을 Hash Function 을 통해 Integer 타입의 해쉬값으로 변환하여 비교한다.
  2. equals() 메서드는 그 주소값으로 비교를 한다.

- 이 말은 즉, <code><strong>모든 객체는 생성된 직후엔, 서로 전부 다른 객체로 인식한다는 것이다.</strong></code>

  

## 2. hashcode 의 존재 이유는 무엇인가??

- **'equals 메서드만 있어도 모든 객체를 비교할 수 있는데, 왜 굳이 hashcode 메서드 같은걸 만들어서 더 헷갈리게 할까??'** 라는 의문을 계속 갖고 있었다.

  - 그래서 이번에 찾아보게 되었고 그 답을 찾았다. 
  - 그 이유는 어찌보면 당연한 거엿는데, <code><strong>객체를 비교할 때 드는 비용을 낮추기 위함이다.</strong></code>

- Hashcode 를 사용하는 **HashSet, HashMap, Hashtable 등등**에서 **'key값(Object)을 통해 value값(Integer) 을 꺼내는 기능'**, **'동일한 객체는 중복해서 추가할 수 없게끔 하는 기능', '객체를 매핑하여 동일한 객체를 찾는 기능'** 등등의 경우에 서로의 객체가 같은지 다른지 비교를 해나가야한다.

  - 여기서 equals 가 오버라이딩 되어있는 가정 하에 equals 메서드로 비교하게 되면, 

  - 모든 객체들을 하나하나 해당 객체의 인스턴스 변수의 값들로 비교를 하게 된다.

  - 그러면 <code><strong>단순히 Integer 값만 비교하면 되는 hashcode 메서드보다 더 많은 비용 즉, 더 많은 시간이 들어가기 때문</strong></code>이다.

    - 즉, <code><strong>Hash 자료구조의 좋은 성능을 위해 존재하는 것이다.</strong></code>

    


## 3. Hash 자료구조에서 equals() 와 hashcode() 의 동작 원리

- **Hash 자료구조에서 에서 비교하는 순서**

  - <code><strong>Hashcode 가 같은가??</strong></code>

    1. HashCode 가 같은 경우

       - <code><strong>equals 가 같은가??</strong></code>
         1. **equals 가 같은 경우**
            - 결과 => true
         2. **equals 가 다른 경우**
            - 결과 => false

    2. **Hashcode 가 다른 경우**

       - 결과 => false

         

- <code><strong>만약, hashcode가 다르면 equals 로 비교조차 하지 않고 다른 객체임을 인식하게 된다.</strong></code>

- 그렇다면 여기서 또 드는 의문이 <code><strong>'더 성능 좋은 hashcode 로 전부 비교하게 하면 되지 않을까??'</strong></code>

  - 동작 원리가 이러는 이유는 '해쉬충돌'이 무엇인지 알게 되면 좀 더 이해하기 쉬울 것이다.

  

- ### 3-1. 해쉬 충돌

  - <code><strong>hash 자료구조에서 hashcode 의 범위가 한정적이기에 equals 가 달라도 hashcode() 메서드의 결과는 같은 경우가 있다. (해쉬충돌)</strong></code>
    - 여기서 <code><strong>'체이닝 기법'</strong></code>이 나오는데 구조를 보자면, 
         - 해쉬값이 버킷(배열의 형태)에 쭉 저장되고,
         - 하나의 해쉬 값(하나의 버킷) 안에 LinkedList 형식으로, 해쉬코드가 같은 객체들이 저장되어있다.
         - 다시 말해서, 같은 해쉬 값 즉, 같은 버킷(배열의 한 요소) 안에 하나의 LinkedList 가 존재하게 되고, 
         - 그 하나의 LinkedList  안에있는 객체들끼리 비교할 때 사용하는 것이 equals() 메서드이다.
  - 쉽게 생각해서 <code><strong>2차원 배열의 모습과 동일하다고 보면 된다고 생각한다.</strong></code>
    - **각 hashcode (배열의 요소) 들이 하나의 LinkedList 참조하는 모습이니 말이다.**



## 4. equals 와 hashcode 는 왜 오버라이딩을 같이 해야하는가??

- 일단 equals 를 오버라이딩 하게 되면, 해당 객체 안의 인스턴스 변수 값들로 비교를 하게 된다.
  - 그래서 hashcode 도 해당 객체 안의 인스턴스 변수들을 통해 해쉬값을 변환하도록 오버라이딩 해야한다.
  
- 더 구체적으로 들어가보자.

  - 각 상황마다 HashSet, HashMap, ArrayList 를 통해 예시 코드를 작성해보았다.

    1. <code><strong>둘 다 오버라이딩 안 할 경우</strong></code>

       - 애초에 이 두 메서드는 해당 객체의 주소값을 이용해서 즉, 

         - hashcode() 는 해당 객체의 주소값을 이용해서 Integer 타입인 해쉬값을 만들고 
         - equals() 는  그 주소값 자체로 비교하기 때문에, 
         - **객체 안의 인스턴스 변수들의 값이 같다고 해도 다른 객체로 인식하게 된다.**

       - 이 경우엔 애초에 서로 비교할 일이 없거나 hash 함수를 쓸 일이 없기 때문에 오버라이딩을 안했을 가능성이 높다.

         - 그래서 크게 문제될 건 없다고 본다.
         - 물론 아직 실무를 안뛰어봤기에, 실제로는 객체마다 무조건 오버라이딩을 다 해놔야 하는건지는 잘 모르겠다.

       - ```java
         import java.util.*;
         
         public class Test {
             private final int a;
             private final int b;
             
             public Test(int a, int b) {
                 this.a = a;
                 this.b = b;
             }
             
             public static void main(String[] args) {
                 HashSet<Test> hashSet = new HashSet<>();
                 HashMap<Test, Integer> hashMap = new HashMap<>();
                 List<Test> list = new ArrayList<>();
                 
                 hashSet.add(new Test(10, 20));
                 hashSet.add(new Test(10, 20)); // 동등한 객체 추가
                 hashSet.add(new Test(10, 30));
                 
                 hashMap.put(new Test(10, 20), 1);
                 hashMap.put(new Test(10, 20), 2);
                 hashMap.put(new Test(10, 30), 3);
                 
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 30));
                 
                 System.out.println("hashSet 크기 : " + hashSet.size());
                 
                 for (Test next : hashSet) {
                     System.out.printf("hashSet 데이터 꺼내기 : %d %d\n", next.a, next.b);
                 }
                 
                 System.out.println("\nhashMap 크기 : " + hashMap.size());
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 20)));
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 30)));
                 System.out.println("\nlist 크기 : " + list.size());
             
                 System.out.println("\nequals 기능 확인 : " + new Test(10, 20).equals(new Test(10, 20)));
                 System.out.println("\nhashcode 기능 확인 : " + (new Test(10, 20).hashCode() == new Test(10, 20).hashCode()));
             }
         }
         
         
         // 출력 결과
         // hashSet 크기 : 3
         // hashSet 데이터 꺼내기 : 10 20
         // hashSet 데이터 꺼내기 : 10 20
         // hashSet 데이터 꺼내기 : 10 30
         // 
         // hashMap 크기 : 3
         // hashMap 데이터 꺼내기 : null
         // hashMap 데이터 꺼내기 : null
         // 
         // list 크기 : 3
         // 
         // equals 기능 확인 : false
         // 
         // hashcode 기능 확인 : false
         ```

         - 위 예시를 보면

           1. HashSet 은 동일한 객체여도 그대로 중복해서 추가가 되는 모습이다.

           2. HashMap 은 동일한 객체여도 그대로 중복해서 추가가 되는 것도 문제지만, 키 값인 동일 객체를 아예 찾지를 못한다.

           3. ArrayList 는 중복이어도 별 상관 없다.

           4. equals() 와 hashcode() 도 동일한 객체를 비교하는데도 전부 false 가 나오는 모습이다.

              

    2. <code><strong>equals 만 오버라이딩 할 경우</strong></code>

       - **hash 자료구조를 사용할 경우, equals() 메서드로 비교도 하기 전에 hashcode() 메서드에서 먼저 다른 객체로 인식하게 된다.**

       - 물론, hash 자료구조를 사용하지 않고, equals 를 통해서만 비교를 하는 상황이라면 상관이 없다.

         - 그러나 Hashtable, HashMap 과 같은 Hash 자료구조를 사용하지 않으면 상관 없어도, 
         - 후에 요구사항이 변경되거나 본인과 협업중인 다른 개발자가 Hash 자료구조를 사용하게 될 경우,
         - null 에러가 뜰 위험이 굉장히 커지므로 
         - 당장은 오버라이딩 안해도 괜찮을 수 있어도, 같이 오버라이딩 해놓자.

       - ```java
         import java.util.*;
         
         public class Test {
             private final int a;
             private final int b;
             
             public Test(int a, int b) {
                 this.a = a;
                 this.b = b;
             }
             
             public static void main(String[] args) {
                 HashSet<Test> hashSet = new HashSet<>();
                 HashMap<Test, Integer> hashMap = new HashMap<>();
                 List<Test> list = new ArrayList<>();
                 
                 hashSet.add(new Test(10, 20));
                 hashSet.add(new Test(10, 20)); // 동등한 객체 추가
                 hashSet.add(new Test(10, 30));
                 
                 hashMap.put(new Test(10, 20), 1);
                 hashMap.put(new Test(10, 20), 2);
                 hashMap.put(new Test(10, 30), 3);
                 
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 30));
                 
                 System.out.println("hashSet 크기 : " + hashSet.size());
                 
                 for (Test next : hashSet) {
                     System.out.printf("hashSet 데이터 꺼내기 : %d %d\n", next.a, next.b);
                 }
                 
                 System.out.println("\nhashMap 크기 : " + hashMap.size());
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 20)));
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 30)));
                 System.out.println("\nlist 크기 : " + list.size());
             
                 System.out.println("\nequals 기능 확인 : " + new Test(10, 20).equals(new Test(10, 20)));
                 System.out.println("\nhashcode 기능 확인 : " + (new Test(10, 20).hashCode() == new Test(10, 20).hashCode()));
             }
             
             @Override
             public boolean equals(Object o) {
                 if (this == o) return true;
                 if (o == null || getClass() != o.getClass()) return false;
                 Test test = (Test) o;
                 return a == test.a && b == test.b;
             }
         }
         
         // 출력 결과
         // hashSet 크기 : 3
         // hashSet 데이터 꺼내기 : 10 20
         // hashSet 데이터 꺼내기 : 10 20
         // hashSet 데이터 꺼내기 : 10 30
         // 
         // hashMap 크기 : 3
         // hashMap 데이터 꺼내기 : null
         // hashMap 데이터 꺼내기 : null
         // 
         // list 크기 : 3
         // 
         // equals 기능 확인 : true
         // 
         // hashcode 기능 확인 : false
         ```

         - 위 예시를 보면

           1. equals() 메서드의 기능만 제대로 동작한다.

           2. 나머지 결과는 전부 1번 예시의 결과와 동일하다.

              

    3. <code><strong>hashcode 만 오버라이딩 할 경우</strong></code>

       - 2번과 마찬가지로 Hash 자료구조를 쓸 경우, 

       - hashcode 까지는 잘 찾아가지만, 

       - 해당 hashcode의 버킷(LinkedList) 안에서 한 번더 equals로 비교하게 될 때, 

       - **동일한 객체를 찾을 수 없기 때문에 null 이 뜨게 된다.**

         - 그래서 2번과 동일하게, 중복 추가가 되는 것도 막을 수 없다.

       - ```java
         import java.util.*;
         
         public class Test {
             private final int a;
             private final int b;
             
             public Test(int a, int b) {
                 this.a = a;
                 this.b = b;
             }
             
             public static void main(String[] args) {
                 HashSet<Test> hashSet = new HashSet<>();
                 HashMap<Test, Integer> hashMap = new HashMap<>();
                 List<Test> list = new ArrayList<>();
                 
                 hashSet.add(new Test(10, 20));
                 hashSet.add(new Test(10, 20)); // 동등한 객체 추가
                 hashSet.add(new Test(10, 30));
                 
                 hashMap.put(new Test(10, 20), 1);
                 hashMap.put(new Test(10, 20), 2);
                 hashMap.put(new Test(10, 30), 3);
                 
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 30));
                 
                 System.out.println("hashSet 크기 : " + hashSet.size());
                 
                 for (Test next : hashSet) {
                     System.out.printf("hashSet 데이터 꺼내기 : %d %d\n", next.a, next.b);
                 }
                 
                 System.out.println("\nhashMap 크기 : " + hashMap.size());
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 20)));
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 30)));
                 System.out.println("\nlist 크기 : " + list.size());
             
                 System.out.println("\nequals 기능 확인 : " + new Test(10, 20).equals(new Test(10, 20)));
                 System.out.println("\nhashcode 기능 확인 : " + (new Test(10, 20).hashCode() == new Test(10, 20).hashCode()));
             }
             
             @Override
             public int hashCode() {
                 return Objects.hash(a, b);
             }
         }
         
         // 출력 결과
         // hashSet 크기 : 3
         // hashSet 데이터 꺼내기 : 10 30
         // hashSet 데이터 꺼내기 : 10 20
         // hashSet 데이터 꺼내기 : 10 20
         // 
         // hashMap 크기 : 3
         // hashMap 데이터 꺼내기 : null
         // hashMap 데이터 꺼내기 : null
         // 
         // list 크기 : 3
         // 
         // equals 기능 확인 : false
         // 
         // hashcode 기능 확인 : true
         ```

         - 위 예시를 보면

           1. hashcode() 메서드만 제대로 작동하는 모습이다.

           2. 나머지 결과는 1번 예시의 결과와 전부 동일하다.

              

    4. <code><strong>둘 다 오버라이딩 할 경우</strong></code>

       - Hash 자료구조를 쓰더라도,

       - equals, hashcode 둘 다 객체 안의 인스턴스 변수를 이용해서  비교를 하게 되기 때문에,

         - (hashcode() 는 인스턴스 변수들로 해쉬값을 만들어서 반환하게 된다.)

       - **동일한 객체, 다른 객체를 정확히 구별해 낼 수 있게 된다.**

       - ```java
         import java.util.*;
         
         public class Test {
             private final int a;
             private final int b;
             
             public Test(int a, int b) {
                 this.a = a;
                 this.b = b;
             }
             
             public static void main(String[] args) {
                 HashSet<Test> hashSet = new HashSet<>();
                 HashMap<Test, Integer> hashMap = new HashMap<>();
                 List<Test> list = new ArrayList<>();
                 
                 hashSet.add(new Test(10, 20));
                 hashSet.add(new Test(10, 20)); // 동등한 객체 추가
                 hashSet.add(new Test(10, 30));
                 
                 hashMap.put(new Test(10, 20), 1);
                 hashMap.put(new Test(10, 20), 2);
                 hashMap.put(new Test(10, 30), 3);
                 
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 20));
                 list.add(new Test(10, 30));
                 
                 System.out.println("hashSet 크기 : " + hashSet.size());
                 
                 for (Test next : hashSet) {
                     System.out.printf("hashSet 데이터 꺼내기 : %d %d\n", next.a, next.b);
                 }
                 
                 System.out.println("\nhashMap 크기 : " + hashMap.size());
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 20)));
                 System.out.println("hashMap 데이터 꺼내기 : " + hashMap.get(new Test(10, 30)));
                 System.out.println("\nlist 크기 : " + list.size());
             
                 System.out.println("\nequals 기능 확인 : " + new Test(10, 20).equals(new Test(10, 20)));
                 System.out.println("\nhashcode 기능 확인 : " + (new Test(10, 20).hashCode() == new Test(10, 20).hashCode()));
             }
             
             @Override
             public boolean equals(Object o) {
                 if (this == o) return true;
                 if (o == null || getClass() != o.getClass()) return false;
                 Test test = (Test) o;
                 return a == test.a && b == test.b;
             }
             
             @Override
             public int hashCode() {
                 return Objects.hash(a, b);
             }
         }
         
         // 출력 결과
         // hashSet 크기 : 2
         // hashSet 데이터 꺼내기 : 10 30
         // hashSet 데이터 꺼내기 : 10 20
         // 
         // hashMap 크기 : 2
         // hashMap 데이터 꺼내기 : 2
         // hashMap 데이터 꺼내기 : 3
         // 
         // list 크기 : 3
         // 
         // equals 기능 확인 : true
         // 
         // hashcode 기능 확인 : true
         ```

         - 위 예시를 보면

           1. hashSet 은 중복추가를 제대로 막아주고 있는 모습이다.

           2. hashMap 은 중복추가도 제대로 막아줄 뿐더러, 동일한 객체를 찾아내어 데이터도 꺼내는 모습이다.

           3. ArrayList 는 별 상관이 없다.

           4. equals() 와 hashcode() 둘 다 제대로 기능을 해주고 있는 모습이다.

              

         - <code><strong>이처럼 equals() 와 hashcode() 둘 다 오버라이딩 해줘야 전부 제대로 동작할 수 있다는 것을 알 수 있다.</strong></code>

       
