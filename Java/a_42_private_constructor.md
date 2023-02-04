# 목차

1. [인스턴스화를 막는 경우는 어떤 경우인가??](#1-인스턴스화를-막는-경우는-어떤-경우인가) <br/>
2. [첫번째 방법 : 추상 클래스화 (비추천 방법)](#2-첫번째-방법--추상-클래스화-비추천-방법) <br/>
3. [두번째 방법 : 생성자를 private 으로 설정 (권장 방법)](#3-두번째-방법--생성자를-private-으로-설정-권장-방법) <br/>

<br/>

# [자바, Java] 이펙티브 자바(Effective Java) - 아이템 04 인스턴스화를 막으려거든 private 생성자를 사용하라

<br/>

## 1. 인스턴스화를 막는 경우는 어떤 경우인가??

**static 메서드만 존재하는 유틸성 클래스의 경우가 그렇다.** static 메서드는 클래스를 통해 바로 접근이 가능하다. 그래서 static 메서드를 인스턴스 변수를 통해 접근하는 방법은 매우 안좋은 방법이다.

static 메서드에 접근하는 예제를 간단히 만들어보자.

```java
public class StringUtils {
    public static boolean isBlank(String str) {
        return Objects.isNull(str) || str.isBlank();
    }
}
```

```java
public class App {
    public static void main(String[] args) {
        // 클래스를 통해 접근
        String str1 = " ";
        System.out.println("str1 문자열이 비어있는가?? : " + StringUtils.isBlank(str1));
    
        // 인스턴스를 통해 접근
        StringUtils instance = new StringUtils();
        String str2 = "a";
        System.out.println("str2 문자열이 비어있는가?? : " + instance.isBlank(str2));
    }
}

// 출력 결과
// 문자열이 비어있는가?? : true
// 문자열이 비어있는가?? : false
```

<br/>위 예제에서 **인스턴스를 통해 static 메서드를 호출하는 방법의 단점은 2가지가 있다.**

1. **해당 메서드가 static 메서드인지 파악이 잘 안된다.**
2. **클래스로 바로 접근 가능함에도 쓸데없이 객체를 생성하게 된다.**

이러한 단점들로 인해, 보통 유틸성 클래스는 생성자 사용을 막는다. 이제 생성자 사용을 막는 방법에 대해 알아보자.<br/>

## 2. 첫번째 방법 : 추상 클래스화 (비추천 방법)

이 방법은 스프링 API에서 생성자 사용을 막을 때 주로 사용하는 방법이다. 하지만 이 방법은 생성자 사용을 완벽히 막아주지 못한다. 그 이유를 살펴보자.

1. **해당 추상 클래스를 상속받은 하위 클래스를 생성하게 되면, 그 부모인 추상 클래스의 생성자도 같이 동작하게 된다.**

   - 생성자를 통한 객체 생성은 막을 수 있지만, 상속은 할 수 있다.

   - 때문에, 그 하위 클래스를 통해 상위 클래스의 static 메서드를 호출할 수 있게 된다.

     ```java
     // 추상 클래스화
     public abstract class StringUtils {
         public static boolean isBlank(String str) {
             return Objects.isNull(str) || str.isBlank();
         }
     }
     ```

     ```java
     public class StringServeUtils extends StringUtils {
     }
     ```

     ```java
     public class App {
         public static void main(String[] args) {
             // 유틸 클래스를 상속받은 하위 클래스를 통해 static 메서드 호출
             StringUtils instance = new StringServeUtils();
             String str = "a";
             System.out.println("str 문자열이 비어있는가?? : " + instance.isBlank(str));
         }
     }
     
     // 출력 결과
     // str 문자열이 비어있는가?? : true
     ```

2. **다른 이용자가 보기에 상속해서 사용하라는 뜻으로 오해할 수 있다.**

   - 추상 클래스가 있으면 생성자를 막는 용도보다는 상속해서 사용하라는 용도로 생각하기 쉬워진다.<br/>

## 3. 두번째 방법 : 생성자를 private 으로 설정 (권장 방법)

private 생성자를 가지면, **해당 클래스 외의 곳에선 생성자를 통해 객체를 생성할 수 없다. 또한 상속도 막아준다.** 물론 **'리플렉션'과 '직렬화와 역직렬화' 로 새로운 객체 생성이 가능**하지만, 이에 대한 **대처법은 이전글인 '싱글톤 파트 정리 글'** 을 보면 될 것이다.

그리고 해당 클래스 안에서도 인스턴스화 하는 것을 막고 싶다면, **private 생성자 구현부에 에러를 던져주면 될 것**이다. Error 에러를 던져서 이 생성자의 동작을 전혀 의도하지 않는다는 것을 보여주자.

참고로, 대처를 할 수 있는 예외(Exception)와는 다르게, 에러(Error)는 복구할 수 없는 심각한 오류라는 것을 나타낸다.

하지만 이에 대한 단점도 존재한다. 어차피 못 쓰는 생성자인데 작성이 되어있으니 직관적이지 못하다는 것이다. 그래서 이 생성자에대한 주석을 명확히 달아둘 필요가 있다.

이제 여기서 설명한 모든 부분을 이용해서 간단한 private 생성자 예제를 만들어보자.

```java
public class StringUtils {
    /**
     * 인스턴스를 생성할 수 없는 유틸 클래스입니다.
     */
    private StringUtils() {
        throw new AssertionError("해당 객체는 생성이 불가능합니다.");
    }
    
    public static boolean isBlank(String str) {
        return Objects.isNull(str) || str.isBlank();
    }
}
```

<br/>

## Reference

1. [이펙티브 자바 완벽 공략 1부 - 백기선님](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1#)
2. [이펙티브 자바 3/E - 교보문고](https://product.kyobobook.co.kr/detail/S000001033066)