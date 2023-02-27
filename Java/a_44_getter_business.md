# 목차

1. [로직이 getter 메서드의 로직과 같다면 getter로 봐야하는가??](#로직이-getter-메서드의-로직과-같다면-getter로-봐야하는가) <br/>
2. [그렇다면 해당 메서드는 단위 테스트를 추가할 만한 가치가 있는가??](#그렇다면-해당-메서드는-단위-테스트를-추가할-만한-가치가-있는가) <br/>

<br/>

# [우아한 테크 코스 5기] 레벨1 - 로직은 getter, 하지만 쓰임새는 비즈니스

## 로직이 getter 메서드의 로직과 같다면 getter로 봐야하는가??

사다리의 구성 요소를 만드는 과정에서 가장 작은 단위인 Bar 객체가 탄생했다. 이 클래스는 해당 Bar가 있는지 없는지만 체크해주면 역할은 끝난다.

```java
public enum Bar {
    TRUE(true),
    FALSE(false);
    
    private final boolean isExistBar;
    
    Bar(boolean isExistBar) {
        this.isExistBar = isExistBar;
    }
    
    public static Bar valueOfBar(boolean otherBar) {
        return Arrays.stream(values())
                .filter(bar -> bar.isSame(otherBar))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 Bar입니다."));
    }
    
    private boolean isSame(boolean otherBar) {
        return this.isExistBar == otherBar;
    }
    
    public boolean isExistBar(){
        return this.isExistBar;
    }
}
```

여기서 한 가지 고민되는 지점이 있다. **isExistBar() 메서드는 getter 로 보아야 하는가??** 사실 로직만 보면 getter 의 로직과 동일하다. 하지만 그 의도는 해당 Bar가 존재하는지 존재하지 않는지 확인하는 비즈니스 로직에 가깝다.

이렇게 메서드명을 부여했을 때의 장점은 분명히 존재한다고 생각한다.

1. **Bar라는 클래스 안에 어떤 값이 필드로 들어있는지 숨길 수 있다.**
2. **어떤 과정을 통해서 Bar가 들어있는지 없는지를 확인하는지 숨길 수 있다.**

만약 getter 로 메서드명을 작성했다면, **해당 객체의 필드가 무엇인지가 드러나게 된다**고 느꼈다. getter 자체가 해당 클래스의 필드를 그대로 가져오겠다는 것을 나타낸다고 생각하기 때문이다. 즉, 캡슐화를 저해할 수 있다고 느낀다.

<br/>

## 그렇다면 해당 메서드는 단위 테스트를 추가할 만한 가치가 있는가??

결국 정답은 없지만 리뷰어분과 토론하면서 몇 가지 기준을 세울 수 있었던 것 같다. 

1. **해당 로직이 얼마나 중요한 비즈니스 로직인지...** 
2. **후에 어떤 특정한 로직이 포함될 수 있는 로직인지...**

에 따라 적절히 테스트를 추가해 줄 지 말 지를 결정하면 된다고 정리했다. 

해당 메서드(isExistBar) 또한 마찬가지이다. 리뷰어분과의 토론 과정에서 **enum의 필드는 고정되어 있다보니 고정된 값이 실제로 일치하는지에 대한 테스트가 중요하다 생각하여 추가하는 경우도 존재할 수 있겠다** 는 것으로 귀결되었다.

이 의견에 굉장히 동의한다. 결국 **해당 메서드의 의도는 비즈니스 로직**이고, **enum의 필드처럼 고정된 값은 한 번 테스트를 성공했을 때의 효율이 굉장히 높다고 생각**되어 테스트를 추가할 만한 가치가 있다고 생각하기 때문이다.