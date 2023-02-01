# 목차

1. [매개변수가 많을 때 생성자나 정적팩토리를 사용하면 무슨 일이 생길까??](#1-매개변수가-많을-때-생성자나-정적팩토리를-사용하면-무슨-일이-생길까) <br/>
2. [생성자 체이닝의 단점을 보완한 자바빈즈 패턴은 좋을까??](#2-생성자-체이닝의-단점을-보완한-자바빈즈-패턴은-좋을까) <br/>
3. [빌더 패턴 - 이펙티브 자바의 권장 패턴](#3-빌더-패턴---이펙티브-자바의-권장-패턴) <br/>
4. [계층형 빌더 패턴 (빌더 패턴의 최종 진화 형태(?))](#4-계층형-빌더-패턴-빌더-패턴의-최종-진화-형태) <br/>
5. [결론 : 매개변수가 4개 이상인 경우엔 생성자나 정적팩토리 대신 빌더 패턴을 사용하자](#결론--매개변수가-4개-이상인-경우엔-생성자나-정적팩토리-대신-빌더-패턴을-사용하자) <br/>

<br/>

# [자바, Java] 이펙티브 자바(Effective Java) - 아이템 02 생성자에 매개변수가 많다면 빌더를 고려하라

<br/>

> **이펙티브 자바를 공부하다보면 자연스럽게 디자인 패턴도 병행해서 공부를 하게 된다.**
>
> **디자인 패턴만 공부하는 것보다, 왜 이 패턴이 필요한지에 대해 더 명확히 알게되어 좋은 것 같다.**

<br/>

## 1. 매개변수가 많을 때 생성자나 정적팩토리를 사용하면 무슨 일이 생길까??

정적 팩토리와 생성자는 선택적 매개변수가 많을 때 대처하기가 쉽지 않다. 이런 경우에 생성자를 이용한 예시를 간단히 만들어보자.

<br/>

```java
public class Character {
    private final String name;
    private final int faceSize;
    private final int neckSize;
    private final int bodySize;
    private final int armSize;
    private final int legSize;
    
    public Character(String name) {
        this(name, 0);
    }
    
    public Character(String name, int faceSize) {
        this(name, faceSize, 0);
    }
    
    public Character(String name, int faceSize, int neckSize) {
        this(name, faceSize, neckSize, 0);
    }
    
    public Character(String name, int faceSize, int neckSize, int bodySize) {
        this(name, faceSize, neckSize, bodySize, 0);
    }
    
    public Character(String name, int faceSize, int neckSize, int bodySize, int armSize) {
        this(name, faceSize, neckSize, bodySize, armSize, 0);
    }
    
    public Character(String name, int faceSize, int neckSize, int bodySize, int armSize, int legSize) {
        this.name = name;
        this.faceSize = faceSize;
        this.neckSize = neckSize;
        this.bodySize = bodySize;
        this.armSize = armSize;
        this.legSize = legSize;
    }
}
```

```java
public class App04 {
    public static void main(String[] args) {
        // 중간의 매개변수 2개는 설정할 필요가 없어서 0으로 설정함
        Character abel = new Character("Abel", 10, 0, 0, 30, 30);
        System.out.println(abel);
    }
}

// 출력 결과
// Character{name='Abel', faceSize=10, neckSize=0, bodySize=0, armSize=30, legSize=30}
```

이처럼 필드가 많은 경우 보통 **this()** 를 이용한 **'점층적 생성자 패턴 또는 생성자 체이닝'** 을 이용한다. 하지만 이 방법엔 치명적인 단점들이 존재한다.

### 1) 단점

1. **매개변수가 늘어날수록 생성자가 많아진다.**
2. **매개변수가 늘어나면 클라이언트 코드를 작성하거나 읽기 어렵다.**
   - 몇번째 인자가 어떤 값을 의미하는지 모른다.
   - 인자가 몇 개인지도 주의깊게 세어봐야한다.
   - 물론 인텔리제이는 인자값을 작성 시 인자의 타입과 이름을 알려주는 기능(Ctrl + P)이 있지만, 이것이 해결책이 될 순 없다.
   - 모두가 인텔리제이를 사용하는 것도 아니고, 인텔리제이에서 이 기능이 나온것도 얼마 안됐다.
3. **같은 시그니처의 생성자는 만들지 못하기에 완벽하게 원하는 값을 초기화하도록 제어하지도 못한다.**
   - 뒤쪽에 있는 매개변수에 값을 전달하기 위해, 중간 순위의 필요없는 매개변수들까지도 설정을 해주는 상황이 발생한다.
   - 위 예시에서 클라이언트 코드에 나타난 현상이 바로 이 현상이다.

정적 팩토리 메서드 또한 매개변수가 늘어날수록 답이 없어지는 것은 매한가지다. 정적 팩토리 메서드의 개수가 기하급수적으로 늘어날 것이 뻔하기 때문이다.

<br/>

## 2. 생성자 체이닝의 단점을 보완한 자바빈즈 패턴은 좋을까??

### 1) 자바 빈이란??

**JSP의 표준 액션 태그로 접근할 수 있는 자바 클래스**로서, 값을 가지는 속성(멤버변수)과 값을 설정하는 메소드(setter) 값을 추출하는 메소드(getter)로 이루어져 있다.<br/>

### 2) 자바빈즈 패턴이란??

**매개 변수가 없는 생성자 (기본 생성자)**로 객체를 만든 후 **setter 메서드**를 호출하여 원하는 매개변수의 값을 설정하는 방식이다.<br/>

자바 빈에 대해선 추후에 따로 더 학습하고 글로 정리해보려 한다. 일단 자바빈즈 패턴 예시를 간단히 만들어보자.

```java
public class Character {
    private String name;
    private int faceSize;
    private int neckSize;
    private int bodySize;
    private int armSize;
    private int legSize;
    
    public void setName(String name) {
        this.name = name;
    }
    
    public void setFaceSize(int faceSize) {
        this.faceSize = faceSize;
    }
    
    public void setNeckSize(int neckSize) {
        this.neckSize = neckSize;
    }
    
    public void setBodySize(int bodySize) {
        this.bodySize = bodySize;
    }
    
    public void setArmSize(int armSize) {
        this.armSize = armSize;
    }
    
    public void setLegSize(int legSize) {
        this.legSize = legSize;
    }
    
    @Override
    public String toString() {
        return "Character{" +
                "name='" + name + '\'' +
                ", faceSize=" + faceSize +
                ", neckSize=" + neckSize +
                ", bodySize=" + bodySize +
                ", armSize=" + armSize +
                ", legSize=" + legSize +
                '}';
    }
}
```

```java
public class App04 {
    public static void main(String[] args) {
        Character abel = new Character();
        abel.setName("Abel");
        abel.setFaceSize(10);
        abel.setArmSize(30);
        abel.setLegSize(30);
        
        System.out.println(abel);
    }
}

// 출력 결과
// Character{name='Abel', faceSize=10, neckSize=0, bodySize=0, armSize=30, legSize=30}
```

그렇다면 **자바빈즈 패턴의 장단점**은 무엇일까??

### 3) 장점

1. **인스턴스 생성이 굉장히 간단해진다.**
   - 생성자 체이닝에 비하여 객체 생성이 훨씬 간단해졌다. 기본 생성자만으로 객체 생성을 할 수 있기 때문이다.
2. **필요한 매개변수만 셋팅할 수 있다.**
   - 뒤쪽 순위의 매개변수를 설정하려면 필요없는 중간 순위의 매개변수까지 어쩔 수 없이 설정해줘야 하는 생성자 체이닝에 비해 효율적이다.
3. **가독성이 좋다.**
   - 메서드 이름을 통해 어떤 매개변수를 설정하는 것인지 명확히 파악할 수 있다.

<br/>

### 4) 단점

1. **필수적인 매개변수를 지정할 수 없다.**

   - 그래서 필수로 설정해줘야 하는 필드가 초기화되지 않고 객체가 생성될 여지가 높다.

2. 완전한 객체를 만들려면 여러개의 메서드를 호출해야하기 때문에, **객체의 일관성이 무너질 수 있다.**

   - 일관성이 무너질 수 있는 이유를 더 자세히 살펴보자면,
     1. **실수로라도 어떤 한 setter 메서드를 빼먹어서 객체가 불완전한 상태로 사용될 여지가 있다.**
     2. **세팅이 다 되기도 전에 이 객체가 사용될 여지가 있다.**
     3. **다른 개발자 입장에서 해당 객체가 어느정도까지 셋팅을 해줘야 하는지 파악하기 어렵다.**

3. **클래스를 불변으로 만들 수 없다.**

   - 인스턴스를 생성한 후에도 setter 메서드는 여전히 사용 가능하기 때문이다. 그래서 보통 객체지향 설계를 할 땐, getter 와 setter 사용을 최대한 지양한다.

   - 물론 **'객체 프리징 기술'** 로 대처할 수 있긴 하지만, **현업에선 별로 사용하지 않는 기술**이라고 한다.

     - 자바스크립트엔 있는 기술이지만, 자바엔 없기 때문에 제각각이 직접 기술을 만들어 사용해야한다. 원래 변경 가능한 가변 객체를 중간에 변경 불가능한 객체로 만든다면, 제 3자 입장에서 코드의 흐름을 파악하는 데에 너무나 큰 불편함이 생길 것이다.

     - 때문에 개념적으로만 알고 넘어가면 좋을 듯 하다.

     - 그래도 간단한 프리징 기술 예제를 만들어 보긴 해보자.

       ```java
       package selftest.builder.constructor;
       
       public class Character {
           private String name;
           private int faceSize;
           private int neckSize;
           private int bodySize;
           private int armSize;
           private int legSize;
           private boolean freeze;
           
           public void setName(String name) {
               validateSetter();
               this.name = name;
           }
           
           public void setFaceSize(int faceSize) {
               validateSetter();
               this.faceSize = faceSize;
           }
           
           public void setNeckSize(int neckSize) {
               validateSetter();
               this.neckSize = neckSize;
           }
           
           public void setBodySize(int bodySize) {
               validateSetter();
               this.bodySize = bodySize;
           }
           
           public void setArmSize(int armSize) {
               validateSetter();
               this.armSize = armSize;
           }
           
           public void setLegSize(int legSize) {
               validateSetter();
               this.legSize = legSize;
           }
           
           private void validateSetter() {
               if (freeze) {
                   throw new IllegalStateException("더이상 필드를 설정할 수 없습니다.");
               }
           }
           
           public void freeze() {
               freeze = true;
           }
       }
       ```

       이런식으로 freeze() 메서드를 한 번이라도 호출 시, 더이상 필드를 설정할 수 없게 만들 수 있다.

<br/>

## 3. 빌더 패턴 - 이펙티브 자바의 권장 패턴

**'생성자 체이닝'** 과 **'자바빈즈 패턴'** 모두 **치명적인 단점들**을 가지고 있다. 그래서 **이펙티브 자바에서 최종적으로 권장하는 패턴**이 바로 이 **'빌더 패턴'** 이다.

먼저 빌더 패턴 사용 예시부터 간단히 만들어보자.

```java
public class Character {
    private final String name;
    private final int faceSize;
    private final int neckSize;
    private final int bodySize;
    private final int armSize;
    private final int legSize;
    
    public static class CharacterBuilder {
        private String name;
        private int faceSize;
        private int neckSize;
        private int bodySize;
        private int armSize;
        private int legSize;
    
        public CharacterBuilder(String name, int faceSize) {
            this.name = name;
            this.faceSize = faceSize;
        }
        
        public CharacterBuilder neckSize(int neckSize) {
            this.neckSize = neckSize;
            return this;
        }
    
        public CharacterBuilder bodySize(int bodySize) {
            this.bodySize = bodySize;
            return this;
        }
    
        public CharacterBuilder armSize(int armSize) {
            this.armSize = armSize;
            return this;
        }
    
        public CharacterBuilder legSize(int legSize) {
            this.legSize = legSize;
            return this;
        }
        
        public Character build() {
            return new Character(this);
        }
    }
    
    private Character(CharacterBuilder builder) {
        name = builder.name;
        faceSize = builder.faceSize;
        neckSize = builder.neckSize;
        bodySize = builder.bodySize;
        armSize = builder.armSize;
        legSize = builder.legSize;
    }
    
    @Override
    public String toString() {
        return "Character{" +
                "name='" + name + '\'' +
                ", faceSize=" + faceSize +
                ", neckSize=" + neckSize +
                ", bodySize=" + bodySize +
                ", armSize=" + armSize +
                ", legSize=" + legSize +
                '}';
    }
}
```

```java
public class App04 {
    public static void main(String[] args) {
        Character abel = new Character.CharacterBuilder("Abel", 10)
                .armSize(30)
                .legSize(30)
                .build();
    
        System.out.println(abel);
    }
}

// 출력 결과
// Character{name='Abel', faceSize=10, neckSize=0, bodySize=0, armSize=30, legSize=30}
```

<br/>

이제 이 빌더 패턴의 장단점을 살펴보자.

### 1) 장점

1. **점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하다.**
   - 생성자의 매개변수를 통해 **필수적인 필드가 무엇인지 파악하기 쉽다.** 그리고 부가적인 필드들은 메서드들의 이름을 보고 **어떤 필드 설정 메서드인지 파악하기 쉽다.**
   - 다른 개발자 입장에서 셋팅 방법을 파악하는데에 점층적 생성자보다 훨씬 용이하다.
2. **자바빈즈보다 훨씬 안전하다.**
   - 메서드 체이닝 끝에 최종적으로 **build() 메서드** 호출을 통해 반환받는 객체는 **모든 필드가 final인 '불변 객체'** 로 반환받을 수 있다. build() 메서드가 호출될 때 최초로 원하는 객체 생성이 이루어지기 때문에 가능한 것이다.
3. **매개변수의 값들을 원하는대로 유기적으로 필드에 설정해줄 수 있다.**
   1. **가변인수 매개변수를 여러개 사용할 수 있다.**
      - 빌더 패턴은 셋팅용 메서드가 여러개로 나뉘어있기 때문에 가변인수의 사용을 늘릴 수 있다.
   2. **메서드 여러개로부터 받은 값들을 한 필드로 모을 수도 있다.**
      - 최종적으로 반환받을 객체의 필드가 컬렉션 종류의 필드 하나뿐일 때, 메서드 체이닝이 하나씩 동작될 때마다 매개변수의 값을 하나씩 해당 필드에 추가해주는 식이 그 예일 것이다.

<br/>

### 2) 단점

1. **성능에 민감한 상황에선 문제가 될 수 있다.**

   - 빌더 객체의 생성 비용이 많이 큰 편은 아니지만, 그 수가 늘어난다면 얘기가 달라질 것이다.\

2. **매개변수가 최소한 4개 이상은 되어야 수지타산이 맞다.**

   - 장황한 코드의 빌더 패턴을 구현하기 위해, 명분이 필요하지 않겠는가..?ㅋㅋ
   - 하지만 보통 API는 시간이 지날수록 매개변수의 개수가 많아지는 편이기 때문에, 처음부터 빌더로 시작하는 것이 나을 수도 있다.

3. **패턴을 구현하기 위한 코드는 이해하기 어려운 편에 속한다.**

   - 필드가 중복이 되는 부분이 생기고, 빌드를 해주는 클래스도 하나 더 필요하다. 그리고 구조 자체가 패턴들 중 꽤 복잡한 편에 속한다.

   - 하지만 이에 대한 대처법도 존재하는데, **'롬복에서 제공하는 @Builder 어노테이션'** 을 사용하면 빌더를 알아서 만들어준다...ㄷㄷ

   - 간단히 예시를 만들어보자.

     ```java
     import lombok.Builder;
     
     // 롬복에서 제공하는 빌더 어노테이션
     @Builder
     public class Character01 {
         private final String name;
         private final int faceSize;
         private final int neckSize;
         private final int bodySize;
         private final int armSize;
         private final int legSize;
         
         @Override
         public String toString() {
             return "Character01{" +
                     "name='" + name + '\'' +
                     ", faceSize=" + faceSize +
                     ", neckSize=" + neckSize +
                     ", bodySize=" + bodySize +
                     ", armSize=" + armSize +
                     ", legSize=" + legSize +
                     '}';
         }
     }
     ```

     ```java
     public class App04 {
         public static void main(String[] args) {
             Character01 abel = new Character01.Character01Builder()
                     .name("Abel")
                     .faceSize(10)
                     .armSize(30)
                     .legSize(30)
                     .build();
         
             System.out.println(abel);
         }
     }
     
     // 출력 결과
     // Character01{name='Abel', faceSize=10, neckSize=0, bodySize=0, armSize=30, legSize=30}
     ```

     <br/>

   - 빌더 어노테이션을 **@Builder(builderClassName = "Builder")** 형식으로 작성해서 자동으로 만들 빌더의 클래스 이름을 지정해줄 수도 있다.

     - 기본 값은 '생성할 클래스 이름 + Builder' 이다.

       <br/>

   - 물론 롬복 빌더도 꽤 치명적인 단점이 존재한다. 롬복이 만들어준 컴파일된 코드를 살펴보자.

     ```java
     public class Character01 {
         private final String name;
         private final int faceSize;
         private final int neckSize;
         private final int bodySize;
         private final int armSize;
         private final int legSize;
     
         public String toString() {
             return "Character01{name='" + this.name + "', faceSize=" + this.faceSize + ", neckSize=" + this.neckSize + ", bodySize=" + this.bodySize + ", armSize=" + this.armSize + ", legSize=" + this.legSize + "}";
         }
     
         Character01(final String name, final int faceSize, final int neckSize, final int bodySize, final int armSize, final int legSize) {
             this.name = name;
             this.faceSize = faceSize;
             this.neckSize = neckSize;
             this.bodySize = bodySize;
             this.armSize = armSize;
             this.legSize = legSize;
         }
     
         public static Character01Builder builder() {
             return new Character01Builder();
         }
     
         public static class Character01Builder {
             private String name;
             private int faceSize;
             private int neckSize;
             private int bodySize;
             private int armSize;
             private int legSize;
     
             Character01Builder() {
             }
     
             public Character01Builder name(final String name) {
                 this.name = name;
                 return this;
             }
     
             public Character01Builder faceSize(final int faceSize) {
                 this.faceSize = faceSize;
                 return this;
             }
     
             public Character01Builder neckSize(final int neckSize) {
                 this.neckSize = neckSize;
                 return this;
             }
     
             public Character01Builder bodySize(final int bodySize) {
                 this.bodySize = bodySize;
                 return this;
             }
     
             public Character01Builder armSize(final int armSize) {
                 this.armSize = armSize;
                 return this;
             }
     
             public Character01Builder legSize(final int legSize) {
                 this.legSize = legSize;
                 return this;
             }
     
             public Character01 build() {
                 return new Character01(this.name, this.faceSize, this.neckSize, this.bodySize, this.armSize, this.legSize);
             }
     
             public String toString() {
                 return "Character01.Character01Builder(name=" + this.name + ", faceSize=" + this.faceSize + ", neckSize=" + this.neckSize + ", bodySize=" + this.bodySize + ", armSize=" + this.armSize + ", legSize=" + this.legSize + ")";
             }
         }
     }
     ```

     <br/>

   - 위 예시를 보면서 롬복 빌더의 단점을 살펴보자.

     1. **무조건 모든 필드의 값을 받는 생성자 하나를 만든다.**
        - 빌더가 아닌 방법으로 객체를 생성할 수 있게 되어버린다.
        - 물론 이에 대한 대처법은 존재한다. 롬복에서 제공하는 **@AllArgsConstructor(access = AccessLevel.PRIVATE)** 어노테이션을 설정해주면 빌더 클래스의 생성자를 사용하지 못하게 막을 수 있다.
     2. **필수 값을 지정해줄 수 없다.**
        - 1번 단점은 극복할 방법이 있지만 이건 극복할 방법이 존재하지 않는다.
        - **결국 필수적인 값이 존재하는 빌더를 만들고 싶다면, 스스로 빌더 패턴을 구현해야만 한다.**

<br/>

## 4. 계층형 빌더 패턴 (빌더 패턴의 최종 진화 형태(?))

계층 구조에서 사용하는 빌더 패턴이다. 일단 예시를 간단히 만들어보자. 예시를 통해 설명할 부분들은 예시 안에서 주석 형식으로 적어보겠다.

```java
public abstract class Car {
    public enum Option {WHEEL, NAVIGATION, AROUND_VIEW, AUTONOMOUS_DRIVING, SUNROOF}
    final Set<Option> options;
    
    // Car.CarBuilder 클래스는 '재귀적 타입 한정 제네릭 타입' 이다.
    // 자신 또는 자신의 자식 클래스들만 제네릭 타입으로 받기 때문이다.
    public abstract static class CarBuilder<T extends CarBuilder<T>> {
        private final EnumSet<Option> options = EnumSet.noneOf(Option.class);
        
        // 여기서 addOption() 메서드의 리턴 값이 this가 아닌 self() 메서드이다.
        // 이 self() 메서드는 self 타입이 없는 자바를 위한 우회 방법으로
        // '시뮬레이트한 셀프 타입 관용구' 라고도 한다.
        public T addOption(Option option) {
            options.add(Objects.requireNonNull(option));
            return self();
        }
        
        protected abstract T self();
        
        abstract Car build();
    }
    
    Car(CarBuilder<?> builder) {
        this.options = builder.options.clone();
    }
}
```

```java
public class Sedan extends Car {
    public enum SedanColor {WHITE, RED, GREEN}
    private final SedanColor sedanColor;
    
    public static class SedanBuilder extends Car.CarBuilder<SedanBuilder> {
        private final SedanColor sedanColor;
        
        public SedanBuilder(SedanColor sedanColor) {
            this.sedanColor = sedanColor;
        }
    
        @Override
        protected SedanBuilder self() {
            return this;
        }
    
        // 하위 빌더 클래스의 build() 메서드는 상위 클래스가 아닌 하위 클래스를 반환하는 모습이다.
        // 이처럼 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌,
        // 그 하위 타입을 반환하는 기능을 '공변 반환 타이핑' 이라 한다.
        @Override
        Sedan build() {
            return new Sedan(this);
        }
    }
    
    private Sedan(SedanBuilder builder) {
        super(builder);
        this.sedanColor = builder.sedanColor;
    }
    
    @Override
    public String toString() {
        return "Sedan{" +
                "sedanColor=" + sedanColor +
                ", options=" + options +
                '}';
    }
}
```

```java
public class SuperCar extends Car {
    private final boolean turboEngine;
    
    public static class SuperCarBuilder extends CarBuilder<SuperCarBuilder> {
        private boolean turboEngine;
        
        public SuperCarBuilder turboEngine(boolean turboEngine) {
            this.turboEngine = turboEngine;
            return this;
        }
    
        @Override
        protected SuperCarBuilder self() {
            return this;
        }
    
        @Override
        SuperCar build() {
            return new SuperCar(this);
        }
    }
    
    private SuperCar(SuperCarBuilder builder) {
        super(builder);
        turboEngine = builder.turboEngine;
    }
    
    @Override
    public String toString() {
        return "SuperCar{" +
                "turboEngine=" + turboEngine +
                ", options=" + options +
                '}';
    }
}
```

```java
public class App05 {
    public static void main(String[] args) {
        Sedan sedan = new Sedan.SedanBuilder(Sedan.SedanColor.WHITE)
                .addOption(Car.Option.WHEEL)
                .addOption(Car.Option.AUTONOMOUS_DRIVING)
                .build();
        
        SuperCar superCar = new SuperCar.SuperCarBuilder()
                .addOption(Car.Option.SUNROOF)
                .turboEngine(true)
                .addOption(Car.Option.NAVIGATION)
                .build();
    
        System.out.println(sedan);
        System.out.println(superCar);
    }
}

// 출력 결과
// Sedan{sedanColor=WHITE, options=[WHEEL, AUTONOMOUS_DRIVING]}
// SuperCar{turboEngine=true, options=[NAVIGATION, SUNROOF]}
```

<br/>

Car 추상클래스의 addOption() 메서드의 리턴 값이 **this가 아닌 self() 메서드여야 하는 이유**를 살펴보자면,

1. **하위(구체) 클래스의 Builder 를 반환하기 때문에 강제 캐스팅을 안해도 된다.**
   - 즉, 클라이언트가 형변환에 신경쓰지 않아도 된다.
2. **각각의 하위 클래스의 빌더들이 자기만 필요한 메서드를 메서드 체이닝 중간에서 사용할 수 있게 된다.**

<br/>

self() 가 아닌 this 인 경우도 예시로 만들어보자.

```java
public abstract class Car {
    public enum Option {WHEEL, NAVIGATION, AROUND_VIEW, AUTONOMOUS_DRIVING, SUNROOF}
    final Set<Option> options;
    
    public abstract static class CarBuilder<T extends CarBuilder<T>> {
        private final EnumSet<Option> options = EnumSet.noneOf(Option.class);
        
        // 반환 타입을 T -> CarBuilder<T> 로 바꾸고,
        // 반환 값을 self() -> this 로 바꿈
        public CarBuilder<T> addOption(Option option) {
            options.add(Objects.requireNonNull(option));
            return this;
        }
        
        protected abstract T self();
        
        abstract Car build();
    }
    
    Car(CarBuilder<?> builder) {
        this.options = builder.options.clone();
    }
}
```

```java
public class App05 {
    public static void main(String[] args) {
        // 강제 캐스팅 생김
        Sedan sedan = (Sedan) new Sedan.SedanBuilder(Sedan.SedanColor.WHITE)
                .addOption(Car.Option.WHEEL)
                .addOption(Car.Option.AUTONOMOUS_DRIVING)
                .build();
        
        // 강제 캐스팅 생김
        SuperCar superCar = (SuperCar) new SuperCar.SuperCarBuilder()
                .addOption(Car.Option.SUNROOF)
//                .turboEngine(true) => turboEngine() 메서드는 메서드 체이닝 중간에서 사용 불가능
                .addOption(Car.Option.NAVIGATION)
                .build();
    
        System.out.println(sedan);
        System.out.println(superCar);
    }
}

// 출력 결과
// Sedan{sedanColor=WHITE, options=[WHEEL, AUTONOMOUS_DRIVING]}
// SuperCar{turboEngine=false, options=[NAVIGATION, SUNROOF]}
```

<br/>

위 예시처럼 this를 리턴하는 것으로 바꾸면 2가지 문제가 터진다. 

1. 더이상 **addOption() 메서드가 하위 빌더를 반환하는 것이 아닌 상위 빌더를 반환하게 된다.** 
   - 그래서 **각 하위 빌더만이 가지고 있는 메서드**는 **메서드 체이닝의 중간에서 사용이 불가능**해진다.

2. 메서드의 마지막 메서드인 **build() 메서드도 상위 빌더가 반환하는 상위 클래스를 반환**한다.
   - 그래서 하위 클래스 타입으로 받으려면 **강제 캐스팅**을 해줘야 한다. 물론 이 부분은 상위 클래스인 Car 타입으로 받아준다면 해결된다. 하지만 위 1번의 문제는 여전히 해결되지 않는다.

<br/>

- ## 결론 : 매개변수가 4개 이상인 경우엔 생성자나 정적팩토리 대신 빌더 패턴을 사용하자.

<br/>

## Reference

1. [이펙티브 자바 완벽 공략 1부 - 백기선님](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1#)
2. [(Java) 점층적 생성자 패턴 & 자바 빈즈 패턴](https://wildeveloperetrain.tistory.com/29)
3. [자바 빈즈란?](https://show400035.tistory.com/m/83)