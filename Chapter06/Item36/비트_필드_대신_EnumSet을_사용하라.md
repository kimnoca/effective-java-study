>비트 필드를 사용할 이유는 없다. EnumSet을 사용하자.   
>EnumSet은 대부분의 오류를 해결해주고 높은 성능을 보여준다.    
>유일한 단점은 불변 EnumSet은 만들 수 없다는 것이다.


## 비트 필드
---
사실 현재 자바에서는 직접적으로 메모리를 관할 일이 적다.   
하지만 옛날처럼 메모리가 부족한 시절에는 비트 하나하나가 소중했다.    
메모리를 아끼기 위해 비트 필드와 같은 기술(?)을 사용하였다.

아래와 같이 텍스트에 여러가지 옵션을 줄 수 있는 상황을 생각해보자

```java
public class Text {
    public boolean BOLD;
    public boolean ITALIC;
    public boolean UNDERLINE;
    public boolean STRIKETHROUGH;
}
```

옵션을 주거나 안주거나, 0과 1로 표현할 수 있다. 즉, **1bit면 충분**하다.     
하지만, 타입을 **boolean으로 주었기 때문에 1byte**가 사용된다.

컴퓨터가 다룰 수 있는 최소 단위가 byte이기 때문에 true, false 만 판단하더라도 1byte가 필요하다.

이러한 문제를 해결하기 위해 정수 열거 패턴이 나왔다.

```java
public class Text {
    public static final int BOLD = 1 << 0;           // 1
    public static final int ITALIC = 1 << 1;         // 2
    public static final int UNDERLINE = 1 << 2;      // 4
    public static final int STRIKETHROUGH = 1 << 3;  // 8

    public void applyStyles(int styles) {
        // ...
    }
}
```

비트별로 옵션을 구분하는 것이다. (BOLD는 첫번째 비트, ITALIC은 두번째 비트)   
styles는 이 비트를 조합한 int값이 매개변수로 들어가게 된다.

```java
text.applyStyles(BOLD | UNDERLINE); // BOLD | UNDERLINE은 3
text.applyStyles(3);
```

위와 같이 OR 연산을 통해 여러 상수를 하나의 집합으로 모을 수 있다.   
이렇게 만들어진 집합을 비트 필드라 한다.

int는 4byte이고 MSB를 제외하고 31bit를 옵션으로 사용할 수 있게 된다.    
즉, 4byte로 4개의 옵션만 사용하던 것에서 **4byte로 31**개의 옵션을 사용할 수 있게 된 것이다.

저자는 이 방식을 구닥다리 방식이라고 하는데 몇가지 문제가 있기 때문이다.

#### 문제점
**1. 비트 필드 값이 그대로 출력되면 해석하기 어렵다.**    
위의 코드에서 볼 수 있듯이 BOLD와 UNDERLINE을 적용한 스타일은 3이다.

숫자 3을 보고서 BOLD와 UNDERLINE을 생각해낼 사람은 없을 것이다.

**2. 필요한 옵션이 늘어나면 타입을 바꿔야 할 수도 있다.**    
만약 int(4byte)로 31개의 옵션을 나타냈다고 가정해보자.

만약 옵션이 늘어 50개의 옵션을 나타내야 한다면?

**int를 long으로** 바꿔줘야한다. 이는 API의 수정이 일어나는 것이다.    
(기존 코드를 갈아 치워야한다.)

따라서, 저자는 **EnumSet을 활용하라고 한다.**
## EnumSet
---
이러한 문제들을 모두 해결해주는 것이 EnumSet이다.

```java
public class Text {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHOUGH
    }

    public void applyStyles(Set<Style> styles) {
        this.styles.addAll(styles);
    }
    
    private Set<Style> styles = new EnumSet.noneOf(Style.class);
}
```

클래스 내에 enum을 선언해두고 사용한다.   
applyStyles 메서드는 Set을 매개변수로 받고 styles에 적용한다.

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

EnumSet을 사용하면 다양한 장점이 있다.
- EnumSet 클래스는 열거 타입 상수 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현한다. (어떤 Set을 넣든 상관없지만 EnumSet을 쓰면 더 좋다.)
	- EnumSet **내부는 비트 벡터**로 구현되어 있다.
	- 대부분의 경우, long 변수 하나로 표현하여 **비트 필드에 대등한 성능**을 보여주기 때문이다.
- removeAll과 retainAll과 같은 **대량 작업은 산술 연산**을 통해 효율적으로 처리한다.
- EnumSet이 **난해한 작업을 다 처리**해주고 흔히 겪는 **오류들을 해결**해준다.
- 타입이 안전하다.

유일한 단점은 **불변 EnumSet을 만들 수 없다**는 것이다.

## 결론
---
비트 필드를 사용할 이유는 없다. EnumSet을 사용하자.

EnumSet은 대부분의 오류를 해결해주고 높은 성능을 보여준다.

유일한 단점은 불변 EnumSet은 만들 수 없다는 것이다.