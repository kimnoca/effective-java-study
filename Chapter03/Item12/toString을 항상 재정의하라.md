_해당 게시글은 이펙티브 자바 교재와 백기선님의 인프런 강의를 보고 정리한 글입니다.
_
<br>
### toString을 재정의해야 하는 이유

<br>

#### Object의 toString 메소드

Object 클래스의 toString은 클래스이름@16진수로 표시한 해시 코드로 나타냄

``` java
public final class PhoneNumber {
    private final int areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode; //지역 코드
        this.prefix   = prefix; //프리픽스
        this.lineNum  = lineNum; //가입자 번호
    }
    
    public static void main(String[] args) {
        PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
        System.out.println("제니의 번호: " + jenny); 
    }
}
```


println에 객체를 넘기면 toString 메서드가 자동으로 호출된다.

이때, Object의 toString을 재정의하지 않았을 때 jenny를 println에 넘긴 결과, PhoneNumber@3f91beef를 반환함.
출력 결과: 제니의 번호: PhoneNumber@3f91beef


하지만, 제니의 번호: 707-867-5309 이런식으로 객체의 정보를 직접 알려주는 형태가 훨씬 유익할 것이다.

toString의 규약에도 '모든 하위 클래스에서 이 메서드를 재정의하라'고 있음.

<br>


#### toString 메서드는 자동 호출됨

println, printf, 문자열 연결 연산자（+）, assert 구문, 디버거가 객체를 출력할 때 자동으로 toString 메서드가 호출됨. 즉, 우리가 직접 호출하지 않아도 다른 어딘가에서 쓰인다. 따라서, 제대로 재정의해서 쓸모있는 메시지를 남길 수 있도록 하자.

<br>

#### 컬렉션 인스턴스에서도 유용하게 쓰임

예를 들어, map 객체를 출력했을 때 {Jenny = PhoneNumber@addbb}보다 {Jenny = 012-1234-5678}라는 메시지가 훨씬 더 가독성이 좋다.

<br>
<br>


### toString을 재정의하자!

<br>

#### 1. 간결하고 사람이 읽기 쉬운 형태의 유익한 정보를 반환하자.

toString 메서드에 인스턴스 정보를 포함하여 재정의한 결과
``` java
public final class PhoneNumber {
    private final int areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix   = prefix;
        this.lineNum  = lineNum;
    }
    
    @Override 
    public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }

    public static void main(String[] args) {
        PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
        System.out.println("제니의 번호: " + jenny);
    }
}
```

제니의 번호: 707-867-5309



<br>


#### 2. 객체가 가진 주요 정보를 모두 반환하자.

객체를 완벽히 설명하는 문자열이 제일 이상적임.
객체가 거대하거나 (객체에 담긴 정보가 많거나) 상태를 문자열로 표현하기 적합하지 않으면 요약 정보를 담자. 

ex) 맨해튼 거주자 전화번호부(총 14236264개)



_백기선님은 다른 의견을 가지고 계신데, 절대 밖으로 노출되면 안되는 데이터들(예를 들어, 주문 내역, 로그 정보 등)은 toString으로 노출하면 안된다 라고 하심._

<br>

#### 3. 반환값의 포맷을 문서화할지 정하자.
포맷을 명시하면 객체를 표준화하고 명확하게 드러낼 수 있다. 단, 포맷을 명시하려면 다음과 같이 아주 정확하게 해야한다.

``` java
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
```

포맷을 명시하면 명시한 포맷에 맞는 문자열과 상호 전환할 수 있는 정적 팩터리나 생성자도 함께 제공하면 좋다.

``` java
package hello.hello_spring.controller;

public final class PhoneNumber {
    private final int areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix   = prefix;
        this.lineNum  = lineNum;
    }

    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
      @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }

    public static PhoneNumber of(String phoneNumberString) {
        String[] split = phoneNumberString.split("-");
        PhoneNumber phoneNumber = new PhoneNumber(
                Integer.parsInt(split[0]),
                Integer.parsInt(split[1]),
                Integer.parsInt(split[2]));
        return phoneNumber;
    }

    public static void main(String[] args) {
        PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
        System.out.println("제니의 번호: " + jenny);
        PhoneNumber phoneNumber = PhoneNumber.of("707-867-5309");
    }
}
```


단점도 존재한다. 포맷을 한번 명시하면 평생 그 포맷에 얽매이고 수정이 어려울 수 있다. 
반대로 포맷을 명시하지 않으면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻을 수 있다.

<br>

#### 4. toString에 포함된 정보를 얻어올 수 있는 API를 제공하자.

toStirng으로 노출하는 데이터(외부에 공개해도 되는 데이터)는 각각의 데이터를 전달받을 수 있도록 하자. 


``` java
    public int getAreaCode() {
        return areaCode;
    }

    public int getPrefix() {
        return prefix;
    }

    public int getLineNum() {
        return lineNum;
    }
```
프로그래머가 직접 toString 반환값을 파싱하지 않도록 각각의 데이터를 얻을 수 있도록 하자. 가령, getter로

<br>

#### 5. 정적 유틸리티 클래스, 열거형은 재정의 안해도됨.

정적 유틸리티 클래스(static 변수, 메소드로 구성된 클래스)는 toString가 필요 없음. 열거형은 자바가 이미 완벽한 toString을 제공함.


<br>

#### 6. 자동생성

우리가 원하는 포맷이 있다면 인텔리제이 자동생성이나 롬복의 @toStiring으로 자동 생성하는 건 적절하지 않을 수도.
