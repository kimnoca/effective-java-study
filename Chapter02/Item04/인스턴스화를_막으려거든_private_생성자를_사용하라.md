단순한 정적 메소드와 정적 필드만을 담은 클래스를 만들어야하는 상황이 있다.

```java
public class Utils {

    // 컴파일러가 자동으로 만들어주는 생성자
    public Utils() {

    }

    public static final int MAX_LOOP_COUNT = 100;

    // ... 더 많은 상수나 static Method
}
```

해당 Utils class와 같이 static 변수나 static Method들만을 모은 class들은 인스턴스화를 할 필요가 없다. 하지만 생성자를 명시하지 않으면 컴파일러가 생성자를 만들어준다.

```java
public class Utils {

    // 기본 생성자가 만들어지는 것을 막는다.
    private Utils() {

    }

    public static final int MAX_LOOP_COUNT = 100;

    // ... 더 많은 상수나 static Method
}
```

의도치 않은 인스턴스화를 막기위해서는 **직접 private 생성자를 선언해 인스턴스화를 막을 수 있다**. 이 코드는 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아준다.

또한 이 방식은 상속을 불가능하게 하는 효과도 있다. 모든 생성자는 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언해 하위 클래스가 상위클래스의 생성자에 접근할 수 없어 상속이 불가능하다.
