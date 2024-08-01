## 정수 열거 패턴

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

Java에서 열거 타입을 지원하기 전에는 다음 코드처럼 정수 상수를 한 묶음 선언해서 사용하곤 했다.

### 정수 열거 패턴의 단점

-   오렌지에서 쓰여야할 상수가 사과에서 쓰여도 컴파일러는 아무런 경고 메시지를 출력하지 않는다. (타입 안전을 보장할 방법이 없다.)

-   정수 열거 타입을 위한 별도의 namespace를 지원하지 않기 때문에 어쩔 수 없이 접두어(`APPLE_`, `ORANGE_`)를 사용해 이름 충돌을 방지한다.

-   정수 열거 패턴을 사용한 프로그램을 깨지기 쉽다. 평범한 상수를 나열한것 뿐만 아니라 컴파일 하면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.

-   정수 상수는 문자열로 출력하기가 다소 까다롭다. 그 값을 출력하면 단지 숫자로만 보여서 썩 도우미이 되지 않는다. 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다. 심지어 그 안에 상수가 몇 개인지도 알 수 없다.

정수 대신 문자열 상수를 사용하는 변형 패턴도 존재하지만 이 변형으 더 나쁜 방법이다.

### 열거 타입

```java
public enum Apple {
    FUJI, PIPPIN, GRANNY_SMITH
}

public enum Orange {
    NAVEL, TEMPLE, BLOOD
}

```

위 정수 열거 패턴 기법을 간단한 열거 타입으로 바꾼 코드이다.

### 열거 타입

Java의 열거 타입은 완전한 형태의 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.
따라서 클라이언트가 인트선스를 직접 생성하거나 확장할 수 없으니, 인스턴스들은 딱 하나씩만 존재한다. 열거타입은 싱글턴을 일반화한 형태인 것이다.

열거 타입은 타입 안정성을 제공한다. Apple 열거 타입을 매개변수로 받는 메서드를 선언하고 Orange 타입등의 다른 타입을 넘기려하면 컴파일 오류가 난다.

열거 타입에는 각자의 이름공간이 있어 이름이 같은 상수도 공존가능하다.
열거타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다.(클라이언트에 각인되지 않기 때문에)

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

```java
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("%s에서의 무게는 %f이다.%n",
                           p, p.surfaceWeight(mass));
   }
}
```

대상 객체의 지구에서의 질량을 입력 받아 모든 Plaent에 포함된 행성들에서의 무게를 출력하는 방법은 위와 같이 간단한다.

열거 타입에서 하나의 상수를 제거해도 참조하는 클라이언트에는 아무 영향이 없다. 그저 출력하는 줄 수가 하나 줄어들 뿐이다.

```java
public enum Operation {
    PULS, MINUS, TIMES;

    public double calculate(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x / y;
        }
        throw new AssertionError("알 수 없는 연산 :" + this);
    }
}
```

`AssertionError`에 실제로는 도달할 일이 없지만 컴파일을 위해서라면 생략할 수 없다. 만약 새로운 상수가 추가된다면 case문도 추가를 해줘야한다. 만약 깜빡하게 되면 새로 추가한 상수의 연산을 할때 알 수 없는 연산이라는 런타임 오류가 발생한다.

이러한 방법을 개선하기 위해서 calculate라는 추상 메서드를 선언하고 각 상수 별로 추상 메서드르르 자신에 맞게 재정의 하는 방법을 사용한다.

```java
public enum Operation {
    PLUS {
        @Override
        public double calculate(double x, double y) {
            return x + y;
        }
    }, MINUS {
        @Override
        public double calculate(double x, double y) {
            return x - y;
        }
    }, TIMES {
        @Override
        public double calculate(double x, double y) {
            return x * y;
        }
    };

    public abstract double calculate(double x, double y);
}
```

새로 상수를 추가 하더라도 calculate라는 함수를 재정의 하지 않ㅇ면 컴파일 오류로 알려준다.

또한 각 연산을 뜻하는 기호를 반환하도록 상수별 데이터와 결합할 수도 있다.

### 값에 따라 분기하여 코드를 공유하는 열거 타입

```java
enum PayrolLDay {
    MONDAY, TUESDAY, WEDSENDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY:

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        // 기본수당
        int basePay = minuitesWorked * payRate;

        // 잔업수당
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay /2;
                break;
            default:
                overtimePay = minuutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate /  2;
        }
        return basePay + overtimePay;
    }
}
```

위 코드는 간결해 보이지만 관리 측면에서 좋은코드는 아니다. 만약 공휴일이나 휴가와 같은 새로운 값을 상수값으로 추가 하려면 그 값을 처리하는 case 문에 넣어줘야한다.

이러한 문제를 해결하기 위한 가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다.
잔업수당 계산을 private 중첩 열거 타입(PayType)으로 옮기고 PayrtrollDay 열거타입의 생성자에서 이 중 적당한 것을 선택하는 것이다. 그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다.

```java
enum PayrollDay {

    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY), SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

### 열거 타입을 언제 쓰면 좋을까

-   필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
