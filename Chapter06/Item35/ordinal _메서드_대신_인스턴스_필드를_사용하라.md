대부분의 열거 타입 상수는 자연스럽게 하나의 정수값에 대응된다.
해당 정수값을 return 하는`ordinal` 이라는 메소드를 지원한다.

아래 코드는 연주자가 1명인 Solo 부터 10명인 dectet까지 정의한 열거 타입이다.

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, DOUBLE_QUARTET,
    NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```

이 코드는 유지보수 하기 나쁜 코드이다. 만약 상수 선언 순서가 바뀌거나. 삭제가 된다면 `numberOfMusicians` 메서드가 오동작한다.
만약 12명의 연주자가 있는 상수를 추가해 줘야한다면 11명의 연주자가 있는 상수가 없기 때문에 더미 상수를 같이 추가해야한다.

이런식으로 `ordinal` 메소드를 사용하면 실용성이 떨어진다.

열거 타입 상수에 연걸된 값은 절대 `ordinal` 메소드로 얻지 말, 인스턴스 필드에 저장하자.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

ordinal 메서드는, EnumSet과 EnumMap과 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다. 따라서, 이런 용도가 아니라면 열거 타입 상수에 연결된 값은 ordinal 메서드를 절대 사용하지 말자.
