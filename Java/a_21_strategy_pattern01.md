# 목차

1. [전략 패턴이란??](#1-전략-패턴이란) <br/>
2. [전략 패턴을 쓰는 이유](#2-전략-패턴을-쓰는-이유) <br/>
3. [전략 패턴 학습 구현](#3-전략-패턴-학습-구현) <br/>
4. [자바 JDK에서 사용되는 전략 패턴](#4-자바-jdk에서-사용되는-전략-패턴) <br/>

<br/>

# [자바, Java] 디자인 패턴 - 전략 패턴 (Strategy Pattern)

<br/>

> **nextstep 에서 배웠을 때도, 알고리즘 공부할 때 sort 메서드 쓸때도**
>
> **어떤 함수형 인터페이스를 매개변수로 전달해서 동작 방식을 변경하는 기법을 자주 보았다.**
>
> **이러한 것들을 검색을 통해 전략 패턴이라것을 알게 되었고,**
>
> **이번에 그 패턴에 대해 공부한 내용을 간단히 정리해 보고자 한다.**

<br/>

## 1. 전략 패턴이란??

- **여러 유사한 알고리즘을 캡슐화해서 객체의 행위를 동적으로 변경 가능하게 만드는 패턴이다.**
  - 상태 패턴과 굉장히 유사해 보이지만, 그 차이는 나중에 정리해보고자 한다.

<br/>

## 2. 전략 패턴을 쓰는 이유

1. **가독성, 유지보수성이 좋아진다.**

   - **만약 전략 패턴을 쓰지 않는다면,**

     1. 어떤 한 로봇이 서 있다.
        - 서있는 로봇 객체를 만든다.
     2. 로봇이 뛴다.
        - 뛰는 로봇 객체를 만든다.
     3. 여기서 로봇에 감정을 추가하게 된다.
     4. 뛰는 로봇이 행복해졌다.
        - 행복한 뛰는 로봇 객체를 만든다.
     5. 다시 로봇이 걸으면서 감정이 상했다.
        - 화난 걷는 로봇 객체를 만든다.
     6. 객체가 기하 급수적으로 늘어난다...

   - **만약 전략 패턴을 적용한다면,**

     1. 어떤 한 로봇이 서 있다.

        - 행동 인터페이스를 생성해서 행동 영역을 묶고, '서있다' 객체를 만든다.
        - 로봇 객체에 '서있다' 객체를 넣어서 행동방식을 바꾼다.
     
     2. 로봇이 뛴다.
     
        - '뛴다' 객체를 만든다.
        - 로봇 객체에 '뛴다' 객체를 넣어서 행동방식을 바꾼다.
     
     3. 여기서 로봇에 감정을 추가하게 된다.
     
     4. 뛰는 로봇이 행복해졌다.
     
        - 감정 인터페이스를 생성해서 감정 영역을 묶고, '행복한' 객체를 만든다.
        - 로봇 객체에 '행복한' 객체를 넣어서 감정을 바꾼다.
     
     5. 다시 로봇이 걸으면서 감정이 상했다.
     
        - 감정 인터페이스를 생성해서 감정 영역을 묶고, '화난' 객체를 만든다.
        - 로봇 객체에 '화난' 객체를 넣어서 감정을 바꾼다.
     
     6. 각자에 해당하는 감정 및 행동 객체를 만들어서 로봇의 변화에 따라 교체만 해주면 된다.
     
        - **전략 패턴을 사용하지 않을 때보다 객체의 수가 훨씬 줄어들고 유지보수성이 좋아진다.**
     
        <br/>

2. **OCP (개방폐쇄의 원칙)** : 변경엔 닫혀있고 확장엔 열려있는 객체지향 원칙이 실현된다.

   - 다른 행동 및 전략이 추가된다고 해도 기존의 코드는 변경하지 않고 확장이 가능하다.

     <br/>

3. **상속 대신 위임을 사용할 수 있다.**

   - **상속의 단점**
     1. 상속은 단 한 개의 클래스만 상속할 수 있기에, 나중에 진짜 상속해야할 클래스가 생길때 상속을 못한다.
     2. 상위 클래스가 바뀌면 하위 클래스들도 모조리 영향을 받기에, 종속성 측면에서의 단점이 있다.
   - **위임의 장점**
     1. 해당 Context 클래스가 변경되더라도, 전략 영역은 전혀 영향을 받지 않는다.
        - 밑의 예제 참고

<br/>

## 3. 전략 패턴 학습 구현

- **예제 - 로봇 설계**

  - ```java
    // 로봇
    public class Robot {
        private MoveStrategy moveStrategy;
        private EmotionStrategy emotionStrategy;
        
        public Robot(MoveStrategy moveStrategy, EmotionStrategy emotionStrategy) {
            this.moveStrategy = moveStrategy;
            this.emotionStrategy = emotionStrategy;
        }
        
        public void robotExplain() {
            moveStrategy.move();
            emotionStrategy.emotion();
            System.out.println("로봇");
        }
        
        public void changeMoveStrategy(MoveStrategy moveStrategy) {
            this.moveStrategy = moveStrategy;
        }
        
        public void changeEmotionStrategy(EmotionStrategy emotionStrategy) {
            this.emotionStrategy = emotionStrategy;
        }
    }
    ```

  - ```java
    // 행동 영역 설정
    public interface MoveStrategy {
        void move();
    }
    
    // 감정 영역 설정
    public interface EmotionStrategy {
        void emotion();
    }
    ```

  - ```java
    // 서있는 행동 객체
    public class Stand implements MoveStrategy {
        @Override
        public void move() {
            System.out.print("서있는 ");
        }
    }
    
    // 걷고있는 행동 객체
    public class Walking implements MoveStrategy {
        @Override
        public void move() {
            System.out.print("걷고있는 ");
        }
    }
    
    // 달리는 행동 객체
    public class Running implements MoveStrategy {
        @Override
        public void move() {
            System.out.print("달리는 ");
        }
    }
    ```

  - ```java
    // 화난 감정 객체
    public class Angry implements EmotionStrategy {
        
        @Override
        public void emotion() {
            System.out.print("화난 ");
        }
    }
    
    // 행복한 감정 객체
    public class Happy implements EmotionStrategy {
        @Override
        public void emotion() {
            System.out.print("행복한 ");
        }
    }
    ```

  - ```java
    // 메인 메서드
    public class Main {
        public static void main(String[] args) {
            Robot robot = new Robot(new Walking(), new Angry()); // 걷고 있는, 화난 객체 설정
            robot.robotExplain(); // 출력
            robot.changeMoveStrategy(new Running()); // 걷고 있는 -> 달리는 교체
            robot.robotExplain(); // 출력
            robot.changeEmotionStrategy(new Happy()); // 화난 -> 행복한 교체
            robot.changeMoveStrategy(new Stand()); // 달리는 -> 서있는 교체
            robot.robotExplain(); // 출력
        }
    }
    
    
    // 출력 결과
    
    // 걷고있는 화난 로봇
    // 달리는 화난 로봇
    // 서있는 행복한 로봇
    ```

    <br/>

## 4. 자바 JDK에서 사용되는 전략 패턴

- 정말 대표적으로 많이 사용되는 전략 패턴 예제를 하나 꼽자면, sort() 정렬 메서드에 사용되는 Comparator 인터페이스이다.

  - ```java
    class Scratch {
        void run () throws IOException {
            Integer[] arr = {4, 7, 100, 2, 6, 33523, 6345, 423, 342, 324, 652, 4};
            System.out.println(Arrays.toString(arr) + "\n");
            
            Arrays.sort(arr); // 오름차순 정렬
            System.out.println(Arrays.toString(arr) + "\n");
            
            Arrays.sort(arr, Comparator.reverseOrder()); // reverseOrder() : 내림차순 정렬 알고리즘 부여
            System.out.println(Arrays.toString(arr) + "\n");
            
            Arrays.sort(arr, Comparator.naturalOrder()); // naturalOrder() : 오름차순 정렬 알고리즘 부여
            System.out.println(Arrays.toString(arr) + "\n");
        }
        
        public static void main(String[] args) throws IOException {
            new Scratch().run();
        }
    }
    
    // 출력 결과
    // [4, 7, 100, 2, 6, 33523, 6345, 423, 342, 324, 652, 4]
    //
    // [2, 4, 4, 6, 7, 100, 324, 342, 423, 652, 6345, 33523]
    // 
    // [33523, 6345, 652, 423, 342, 324, 100, 7, 6, 4, 4, 2]
    // 
    // [2, 4, 4, 6, 7, 100, 324, 342, 423, 652, 6345, 33523]
    ```

  - **전략 객체를 교체함으로써 알고리즘과 행동 방식이 변경되는 모습을 볼 수 있다.**