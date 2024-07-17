_해당 게시글은 이펙티브 자바 교재와 백기선님의 인프런 강의를 보고 정리한 글입니다._

<br>

### public 필드 사용 시 문제점 3가지

아래와 같이 인스턴스 필드들**만**(메소드 없이) 모아놓은 클래스를 작성하려 할 때가 있다.
이때 필드를 public으로 선언하면 클라이언트에서 직접 접근 가능하나 캡슐화의 이점을 제공하지 못함.


``` java
class Point {
    public double x;
    public double y;

}
```



#### 문제1. API의 필드를 수정하지 않고는 내부 표현을 바꿀 수 없다.

메서드 없이 public 필드로만 구성되어있기 때문에 내부 표현을 변경하기 위해서는 API의 필드를 변경해야 한다.
https://github.com/Hyeon9mak/Hyeon9mak.github.io/issues/110#issuecomment-1405942156 댓글에 예시 참고. (_클래스 내부에 데이터를 저장하는 방식을 바꾸고 싶으면(필드명 변경) 필드 자체를 바꿔야함. 이는 그 필드를 사용하는 모든 곳에 영향을 줌. _)

#### 문제2. 불변식을 보장할 수 없다.
클라이언트가 public 필드에 직접 접근해 데이터를 변경할 수 있다.

#### 문제3. 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.
단순히 필드에 직접 접근하는 식의 1차원적인 접근만 가능하고, 추가 로직(예를 들어, 연산 로직, 값을 검증하는 로직 등)을 삽입할 수 없다.

<br>


<br>

### 해결1. private 필드 사용, getter 메서드 추가
필드를 모두 private으로 바꾸고 getter 메서드를 추가한다.

``` java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    public double getX() {return x; }
    public double getY() {return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```
이렇게 하면 클래스 내부 표현 방식을 언제든 바꿀 수 있게 된다.
getter 메서드를 사용하면 내부 표현을 변경하더라도 접근자 메서드만 수정하면 되어, 코드 전체에 광범위한 변경을 피할 수 있다.

<br>
<br>

### 해결2. package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 없다.

1. private 중첩 클래스
``` java
public class TopPoint {

    private static class Point {
        public double x;
        public double y;
    }
    
    public Point getPoint() {
        Point point = new Point();   
        point.x=1.1;
        point.y=2.1
        return point;
    }
}
```

private 중첩 클래스인 Point 클래스의 필드가 public으로 선언되어 있다. 따라서, 탑 클래스인 TopPoint 클래스에서만 Point 클래스의 필드에 접근 가능하기 때문에 위의 3가지 문제점이 나타나지 않는다.




<br>

2. package-private 클래스

package-private 클래스 역시 해당 클래스가 포함되는 패키지 내에서만 조작이 가능하고 패키지 외부에서는 접근이 불가능하기 때문에 마찬가지로 위의 3가지 문제점이 나타나지 않는다.
