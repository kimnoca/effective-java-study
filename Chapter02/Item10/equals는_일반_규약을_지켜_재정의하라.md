---
title: "[Effective Java] equals는 일반 규약을 지켜 재정의하라"
date: 2024-07-07 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>꼭 필요한 경우가 아니라면 **재정의하지 말자.**   
  그래도 필요하다면 핵심필드를 빠짐 없이 비교하며 **다섯 가지 규약**을 지키자.   
  어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 **다를 가능성이 더 크거나** 비교하는 **비용이 더 싼 필드를 먼저** 비교하자.   
  객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.   
  핵심 필드로부터 파생되는 필드가 있다면 굳이 둘다 비교할 필요는 없다. 편한 쪽을 선택하자.   
  [equals를 재정의할 땐 hashCode도 반드시 재정의하자(아이템 11)]   
  equals의 매개변수 입력을 Object가 아닌 타입으로는 선언하지 말자.   
  [@Override 애너테이션을 일관되게 사용하라(아이템40)]   

## 재정의하지 않는 경우
---
#### 1. 각 인스턴스가 본질적으로 고유하다
Integer와 String처럼 값을 표현하는 것이 아닌 Thread처럼 **동작하는 개체를 표현**하는 클래스가 해당된다.

값을 표현하는 클래스더라도 값이 같은 인스턴스가 둘 이상 만들어지지 않는 것을 보장한다면 이도 해당된다.    
이에는 **enum이나 인스턴스 통제 클래스** 등이 해당된다.   

#### 2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.
정규식에 equals를 적용할 때, 두 정규식이 동일한 정규식인지 검사할 수도 있다.    
하지만, **설계자**가 이러한 논리적 동치성을 검사하길 원하지 않는다면 **Object의 equals로도 충분**하다.

아래의 코드에서 볼 수 있듯 Pattern 클래스는 Object의 equals를 사용하였다.    
그 결과, 같은 정규식을 가지고 있지만 **p1.equals(p2)의 결과는 false**가 나오게 된다.

```java
public void patternEqualsTest(){
	final Pattern p1 = Pattern.compile("0*")
	final Pattern p2 = Pattern.complie("0*")

	System.out.println(p1.equals(p2));  // false
}
```

#### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
**LinkedList는 List의 equals를 상속**받아 사용하고 **List는 AbstractList의 equals를 상속**받아 쓴 경우다.

따라서 아래와 같은 코드를 실행시 true가 출력된다.

```java
public void listEqualsTest(){
	LinkedList<String> lList = new LinkedList<String>();
    ArrayList<String> aList = new ArrayList<String>();
    
    lList.add("data1");
    aList.add("data1");

	System.out.println(lList.equals(aList)); // true;
    }
}
```

#### 4. 클래스가 private이거나 package-private이고 equals를 호출할 일이 없다.
당연히 호출 될 일이 없다면 재정의 할 이유가 없다.    
하지만, 극단적으로 막고싶다면 아래와 같이 Exception을 터트리는 방법도 있다.

```java
public boolean equals(Object o){
    throw new AssertionError(); // 호출 시 Exception
}
```

## equals의 일반규약
---
아래의 일반 규약을 **철저히 지켜 작성해야 문제가 생기지 않는다.**   

> 존 던(John Donne)씨는 '**세상에 홀로 존재하는 클래스는 없다.**'라고 한 만큼 혼자 쓰이는 클래스는 드물다.   
> 즉, 각 클래스들이 **equals 규약을 지킨다고 가정하고 동작**한다.

이산수학에서 본 듯한 개념이들이다.

1. **반사성**
	- `x.equals(x) == true`
2. **대칭성**
	- `x.equals(y) == true`라면 `y.equals(x) == true`
3. **추이성**
	- `x.equals(y) == true`이고 `y.equals(z) == true`라면 `z.equals(x) == true`
4. **일관성**
	- `x.equals(y)`를 여러번 호출해도 결과는 변하지 않음
5. **null-아님**
	- `x.equals(null) == false`

### 1. 반사성
---
null이 아닌 모든 참조 값 x 에 대해 `x.equals(x)`를 만족해야한다.

즉, **자기 자신**에게 equals를 사용하면 **항상 true를 반환**해야한다.

이러한 점이 안지켜지면, collection에 contains가 정상적으로 동작하지 않게 된다.

### 2. 대칭성
---
null이 아닌 모든 참조값 x, y에 대해`x.equals(y) == true`라면 `y.equals(x) == true`가 만족한다.

아래의 코드는 대칭성이 지켜지지 않은 경우다.

CaseInsensitiveString클래스는 비교대상이 CaseInsensitiveString인지 일반 String인지 **구분하여 동작**한다.   
하지만 String 클래스의 equals는 CaseInsensitiveString과의 비교에서 **false를 반환**한다.    
(이는 구현하기 나름이라 런타임 Exception을 터트릴 수도 있다.)

해결: 주석과 같이 **타입  체크 후 비교연산**

```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);  // null 체크 후 Exception
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
    
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
    
//  정상 코드
//	@Override
//	public boolean equals(Object o){
//		return o instance of CaseInsensitiveString &&
//			((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
//	}
}

void symmetryTest() {
    CaseInsensitiveString str1 = new CaseInsensitiveString("Test");
    String str2 = "test";
    System.out.println(str1.equals(str2)); // true
    System.out.println(str2.equals(str1)); // false
}
```


### 3. 추이성
---
null 아닌 모든 참조값 x, y, 에 대해    
`x.equals(y) == true`이고 `y.equals(z) == true`라면 `z.equals(x) == true`가 만족한다.

아래의 코드는 **대칭성은 지켰지만 추이성을 지키지 못한** 경우이다.

```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}

public class ColorPoint extends Point {

    private final Color color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```

아래와 같이 **ColorPoint끼리 비교**하였을 때 false를 반환하며 추이성이 깨지게 된다.

```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```

또한, 이러한 코드는 **무한재귀**에 빠질 수 있다.

#### 3-1 해결 실패 - 무한재귀
```java
public class SmellPoint extends Point {

    private final Smell smell;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof SmellPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.smell == ((SmellPoint) o).smell;
    }
}

void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new SmellPoint(2, 3, Smell.SWEET);

    System.out.println(cp.equals(sp));
}
```

**Point를 상속**받은 SmellPoint가 있다고 했을 때 무한 재귀가 발생한다.

어? 같은 클래스는 아닌데 상속받은건 같네? **너걸로 돌려봐 반복**, StackOverflowError가 발생한다.

#### 3-2 해결실패 - 리스코프 치환 원칙 위배
그렇다면 instanceof 검사를 getClass 검사로 바꾸면 어떻게 될까?

```java
@Override
public boolean equals(Object o) {
    // getClass
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
}
```

같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 동작은 하지만 리스코프 치환원칙을 위배한다.
(어떤 타입에서 중요한 속성이라면 그 **하위 타입에서도 중요하다**.)

> 이는 객체지향 언어에서 동치 관계의 근본적인 문제이다.   
> 따라서, 객체지향의 이점을 포기하지 않는 한 해결이 불가능하다.

**그렇다면 하위 클래스에 값을 추가하려면 어떻게 해야할까?**
#### 3-3 상속대신 컴포지션을 사용하라
[상속대신 컴포지션을 사용하라]

```java
public class ColorPoint2 {

    private Point point;
    private Color color;

    public ColorPoint2(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return this.point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return this.point.equals(cp) && this.color.equals(cp.color);
    }
}
```

#### 3-4 추상클래스
> 추상 클래스의 하위 클래스라면 equals 규약을 지키면서도 값을 추가할 수 있다.

즉, 상위 클래스를 직접 인스턴스로 만드는게 불가능하다면 문제가 생기지 않는다.    
[태그 달린 클래스보다는 클래스 계층구조를 활용하라(아이템23)]

### 4. 일관성
---
null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`를 반복해서 호출하면 항상 같은 값을 반환한다.

이런 경우가 있겠나 싶겠지만 **신뢰할 수 없는 값이 equals에 사용되면** 이런 일이 발생한다.

예로 java.net.URL 클래스가 있다.

```java
@Test
void consistencyTest() throws MalformedURLException {
    URL url1 = new URL("www.xxx.com");
    URL url2 = new URL("www.xxx.com");

    System.out.println(url1.equals(url2)); // 항상 같지 않다.
}
```

같은 도메인 주소라도 **host의 ip주소를 활용**하여 비교하기 때문에 매번 결과가 달라질 수 있다.

따라서, 이런 문제를 피하려면 항시 메모리에 존재하는 결정적(deterministic) 계산만 사용해야한다.


## 5. null-아님
---
null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 false

여기서의 핵심은 true를 반환하지 않는다는 것이 아니라 **NullPointerException을 터트리지 않는다는 것**이다.

사실 이문제는 instanceof 연산을 통해 자연스럽게 처리된다.

```java
@Override 
public boolean equals(Object o) {
    // 우리가 흔하게 인텔리제이를 통해서 생성하는 equals는 다음과 같다.
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    
    // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.
    // o가 null이면 자연스럽게 false가 반환됨 
    if (!(o instanceof Point)) {
        return false;
    }
}
```

## equals의 좋은 재정의
---
```java
@Override
public boolean equals(Object o) {
    // 1. == 연산자를 사용해 반사성 체크
    // 성능상 이점
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사
    
    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x == p.x && this.y == p.y;
    
    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);
    
    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```
