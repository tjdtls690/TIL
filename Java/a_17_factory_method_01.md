# 목차

1. [팩토리 메서드 패턴이란??](#1-팩토리-메서드-패턴이란) <br/>
2. [생성자가 있는데 왜 굳이 팩토리 메서드 패턴을 쓸까??](#2-생성자가-있는데-왜-굳이-팩토리-메서드-패턴을-쓸까) <br/>
3. [팩토리 메서드 구현방법 2가지](#3-팩토리-메서드-구현방법-2가지) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 팩토리 메서드 (Factory Method)

<br/>

> **난 디자인 패턴이라는 개념이 굉장히 생소했다.**
>
> **'코딩을 하는데 무슨 디자인까지 필요한가...?' 라는 어처구니 없는 생각을 가지고 있었다.**
>
> **그럴만 한게, 국비지원 학원같은 곳은 절대 그런 깊은 개념은 알려주지 않는다.**
>
> **그 학원들을 까는건 아니다.** 
>
> **6개월이란 시간제한이 있기에 어차피 그러한 개념들까지 가르치는 것은 불가능하다.**
>
> **물론, 그렇기 때문에 국비학원 출신의 개발자들이 질이 안좋은 코딩을 할 수밖에 없지만 말이다.**
>
> **디자인 패턴에 대해 공부를 하기 시작했는데 가장 익숙한 패턴들부터 공부해보려고 한다.**
>
> **오늘은 팩토리 메서드 패턴에 대해 공부했는데 간단히 정리해보자.**

<br/>

## 1. 팩토리 메서드 패턴이란??

- **객체 생성의 역할을 하는 클래스 메서드**이다.

  - 즉, 객체를 생성하는 역할을 분리하겠다는 취지가 담겨있다.

- 자바에서 실제로 구현되어있는 코드를 보자 - **Calendar 클래스**

  - ```java
    public abstract class Calendar implements Serializable, Cloneable, Comparable<Calendar> {
        
        ...
        
    	private static Calendar createCalendar(TimeZone zone,
                                               Locale aLocale)
        {
            CalendarProvider provider =
                LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
                                     .getCalendarProvider();
            if (provider != null) {
                try {
                    return provider.getInstance(zone, aLocale);
                } catch (IllegalArgumentException iae) {
                    
                }
            }
    
            Calendar cal = null;
    
            // 이 부분이 팩토리 메서드의 핵심 부분이라 볼 수 있다.
            if (aLocale.hasExtensions()) {
                String caltype = aLocale.getUnicodeLocaleType("ca");
                if (caltype != null) {
                    switch (caltype) {
                    case "buddhist":
                    cal = new BuddhistCalendar(zone, aLocale);
                        break;
                    case "japanese":
                        cal = new JapaneseImperialCalendar(zone, aLocale);
                        break;
                    case "gregory":
                        cal = new GregorianCalendar(zone, aLocale);
                        break;
                    }
                }
            }
            if (cal == null) {
                if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
                    cal = new BuddhistCalendar(zone, aLocale);
                } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                           && aLocale.getCountry() == "JP") {
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                } else {
                    cal = new GregorianCalendar(zone, aLocale);
                }
            }
            return cal;
        }
        
        ...
        
    }
    ```

    - 이처럼 직접적으로 생성자를 통해 객체를 생성하는 것이 아닌 메서드를 통해서 객체를 생성하는 것을 **'팩토리 메서드 패턴'**이라고 한다.

<br/>

## 2. 생성자가 있는데 왜 굳이 팩토리 메서드 패턴을 쓸까??

- 팩토리 메서드의 많은 장점들이 있겠지만, 내가 프로그래밍을 하면서 직접적으로 느낀점을 정리해보려고 한다.
  1. **OCP (개방폐쇄의 원칙)** : 변경엔 닫혀있고 확장엔 열려있는 객체지향 원칙이 실현된다.
     - 직접적으로 생성자를 통해 객체 생성을 하게 되면, 새로운 제품이 추가 될 때마다 객체 생성을 해줘야 하기에 코드 변경이 불가피하다.
     - 하지만 팩토리 메서드 패턴을 구현하게 되면, 자연스럽게 인터페이스를 쓰게 된다.
     - 다른 제품이 추가될 때, 팩토리 메서드에 객체 생성 조건 한 줄 추가되는 것 외의 다른 부분에서 변경이 전혀 없다. 
       - 유지보수성이 뛰어나다.
     - 그리고 인터페이스를 통해 새로운 제품을 확장만 하면 된다.
  2. **SRP (단일책임의 원칙)** : 객체를 생성하는 책임을 하나의 클래스에 부여함으로써 가독성, 유지보수성이 좋아진다.
     - 책임을 최대한 나눌수록 디버깅하기 쉬워진다.
  3. **객체 생성을 캡슐화 할 수 있다.**
     - 어떤 객체가 생성되는지, 객체가 어떻게 생성되는지 등등을 은닉할 수 있다.
  4. **이름을 부여할 수 있다.**
     - 팩토리 메서드의 이름을 통해 어떤 종류의 객체 생성 메서드인지 알고 사용할 수 있다.

<br/>

## 3. 팩토리 메서드 구현방법 2가지

1. **Factory 클래스와 Product 클래스 둘 다 확장을 하는 방식의 팩토리 패턴**
   - Product 생성의 책임을 갖는 Factory 영역과 제품 그 자체인 Product 영역, 두 영역 전부 인터페이스를 두고 제품을 확장해나가는 형식이다.
   
     - 그리고 고객 입장에서 각 제품의 Factory 를 통해서 제품을 제공받는다.
   
   - 다이어그램으로 구조를 살펴보면 좀 더 이해가 갈 것이다.
   
     - ![notion_img01](https://tjdtls690.github.io/assets/img/github_img/factory_method01.PNG)
   
       <br/>
   
   - 예제
   
     - ```java
       // 고객 입장에서 제품을 주문하고, 완성된 제품을 받는 곳.
       public class Client {
           public static void main(String[] args) {
               printHouseInform(new WhiteHouseFactory());
           }
           
           private static void printHouseInform(HouseFactory houseFactory) {
               System.out.println(houseFactory.create());
           }
       }
       
       // 출력 결과
       // House{name='WhiteHouse', color='white'}
       ```
   
     - ```java
       // Factory 영역의 인터페이스
       public interface HouseFactory {
           House create();
       }
       
       // Product 영역의 인터페이스 역할을 맡은 부모 클래스
       public class House {
           private String name;
           private String color;
           
           public House(String name, String color) {
               this.name = name;
               this.color = color;
           }
           
           @Override
           public String toString() {
               return "House{" +
                       "name='" + name + '\'' +
                       ", color='" + color + '\'' +
                       '}';
           }
       }
       ```
   
     - ```java
       // 화이트 색상의 집을 만드는 Factory
       public class WhiteHouseFactory implements HouseFactory {
           @Override
           public House create() {
               return new WhiteHouse();
           }
       }
       
       // 화이트 색상의 집 (Product)
       public class WhiteHouse extends House {
           public WhiteHouse() {
               super("WhiteHouse", "white");
           }
       }
       ```
   
       <br/>
   
     - 팩토리 메서드의 가치는, 현 상태에서 블랙 색상의 집을 추가제작 (확장) 하려고 할 때, Client 의 주문부분만 제외하고 변경하는 코드가 일절 없다.
   
       - 개방폐쇄의 원칙이 지켜지는 모습이다.
   
       - ```java
         public class Client {
             public static void main(String[] args) {
                 printHouseInform(new WhiteHouseFactory());
                 printHouseInform(new BlackHouseFactory()); // 새로운 제품을 제작 요청하는 한 줄 추가
             }
             
             private static void printHouseInform(HouseFactory houseFactory) {
                 System.out.println(houseFactory.create());
             }
         }
         
         // 출력 결과
         // House{name='WhiteHouse', color='white'}
         // House{name='BlakHouse', color='black'}
         ```
   
       - ```java
         // 블랙 색상의 집을 만드는 Factory
         public class BlackHouseFactory implements HouseFactory {
             @Override
             public House create() {
                 return new BlackHouse();
             }
         }
         
         // 블랙 색상의 집 (Product)
         public class BlackHouse extends House{
             public BlackHouse() {
                 super("BlakHouse", "black");
             }
         }
         ```
   
       - 기존 코드를 변경하지 않고, 확장함으로써 새로운 제품이 추가된 모습을 볼 수 있다.
   
     
   
   <br/>
   
2. **사용자로부터 입력받은 문자열 즉, 매개변수의 값에 따라 각기 다른 객체를 생성하는 팩토리 패턴**
   - 심플 팩토리 메서드 패턴 (Simple Factory Method Pattern) 라고도 부른다.
   
     - 1번 패턴과는 다르게, 팩토리 영역은 확장하지 않는 더 단순화된 패턴이다.
   
   - 다이어그램
   
     - ![notion_img01](https://tjdtls690.github.io/assets/img/github_img/factory_method02.PNG)
     - 1번 예제와 다르게 Factory 영역을 팩토리 메서드 하나로 관리하는 모습을 볼 수 있다.
   
     <br/>
   
   - 예제
   
     - ```java
       // 고객 입장에서 제품을 주문하고, 완성된 제품을 받는 곳.
       public class Client {
           public static void main(String[] args) {
               printHouseInform(HouseFactory.from("WhiteHouse"));
           }
           
           private static void printHouseInform(House house) {
               System.out.println(house);
           }
       }
       
       // 출력 결과
       // House{name='WhiteHouse', color='white'}
       ```
   
     - ```java
       // 팩토리 메서드
       public class HouseFactory {
           public static House from(String houseName) throws IllegalArgumentException{
               if (houseName.equals("WhiteHouse")) {
                   return new WhiteHouse();
               }
               
               throw new IllegalArgumentException("해당 제품이 존재하지 않습니다.");
           }
       }
       ```
   
     - ```java
       // Product 영역의 인터페이스 역할을 맡은 부모 클래스
       public class House {
           private String name;
           private String color;
           
           public House(String name, String color) {
               this.name = name;
               this.color = color;
           }
           
           @Override
           public String toString() {
               return "House{" +
                       "name='" + name + '\'' +
                       ", color='" + color + '\'' +
                       '}';
           }
       }
       
       // 화이트 색상의 집 (Product)
       public class WhiteHouse extends House {
           public WhiteHouse() {
               super("WhiteHouse", "white");
           }
       }
       ```
   
       <br/>
   
     - 여기서 추가 제품을 확장하고 싶을 땐, 팩토리 메서드의 조건 하나와 클라이언트의 주문부분 한 줄이 추가되는 것 외엔 변경되는 코드는 없다.
   
       - ```java
         public class Client {
             public static void main(String[] args) {
                 printHouseInform(HouseFactory.from("WhiteHouse"));
                 printHouseInform(HouseFactory.from("BlackHouse")); // 새로운 제품을 주문하기 위해 한 줄 추가
             }
             
             private static void printHouseInform(House house) {
                 System.out.println(house);
             }
         }
         
         // 출력 결과
         // House{name='WhiteHouse', color='white'}
         // House{name='BlackHouse', color='black'}
         ```
   
       - ```java
         // 팩토리 메서드
         public class HouseFactory {
             public static House from(String houseName) throws IllegalArgumentException{
                 if (houseName.equals("WhiteHouse")) {
                     return new WhiteHouse();
                 }
                 
                 if (houseName.equals("BlackHouse")) { // 새로운 제품 생성을 위한 조건식 하나 추가
                     return new BlackHouse();
                 }
                 
                 throw new IllegalArgumentException("해당 제품이 존재하지 않습니다.");
             }
         }
         
         
         // 블랙 색상의 집 (Product)
         public class BlackHouse extends House {
             public BlackHouse() {
                 super("BlackHouse", "black");
             }
         }
         ```
   
         <br/>
   
       - 새로운 제품이 추가될 때마다, 팩토리 메서드만 신경써주고 그 외엔 확장만 해주면 되기에 유지보수에 굉장히 용이하다.