_해당 게시글은 이펙티브 자바 교재와 백기선님의 인프런 강의를 보고 정리한 글입니다._
<br>

item3에서는 싱글턴을 생성하는 3가지 방식에 대해 소개하고 각각의 장단점에 대해 소개한다.


<br>

### 싱글턴이란

> 인스턴스를 오직 하나만 생성할 수 있는 클래스

<br>

### 1. public static final 키워드 사용

``` java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

}
```
INSTANCE는 static이므로 클래스가 로딩될 시점에 Elvis 객체가 생성됨. private 생성자도 이때 한 번 호출되고 이후에 호출 안됨. 따라서, 이때 만들어진 인스턴스가 전체 시스템에 하나뿐임이 보장됨.


#### 장점
1. 간결함
2. 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
	static final로 만들면 자바 doc 생성 시 해당 필드가 싱글톤object임이 명시적으로 기재됨

#### 단점
1. 싱글턴을 사용하는 클라이언트를 테스트하기 어렵다.
2. 리플렉션 API 사용 시 private 생성자를 호출 가능
3. 역직렬화 시 새로운 인스턴스 생성 가능

<br>

#### 단점1. 싱글턴을 사용하는 클라이언트를 테스트하기 어렵다.


Elvis 클래스

```java
public class Elvis{

    /**
     * 싱글톤 오브젝트
     */
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {}

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public void sing() {
        System.out.println("I'll have a blue~ Christmas without you~");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }

}
```



<br> 이를 사용하는 Concert 클래스(클라이언트)

``` java
public class Concert {

    private boolean lightsOn;

    private boolean mainStateOpen;

    private Elvis elvis;

    public Concert(Elvis elvis) {
        this.elvis = elvis;
    }

    public void perform() {
        mainStateOpen = true;
        lightsOn = true;
        elvis.sing();
    }

    public boolean isLightsOn() {
        return lightsOn;
    }

    public boolean isMainStateOpen() {
        return mainStateOpen;
    }
} 
```

<br>
Concert 클래스 perform() 메소드 테스트

``` java
class ConcertTest {

    @Test
    void perform() {
        Concert concert = new Concert(Elvis.INSTANCE);
        concert.perform();

        assertTrue(concert.isLightsOn());
        assertTrue(concert.isMainStateOpen());
    }

}
```

테스트 할 때마다 Elvis 싱글톤 객체를 호출하는 것은 비효율적임. 오퍼레이션 자체가 외부 네트워크를 거쳐 다른 외부 api를 호출하는 경우거나 연산이 오래 걸리는 작업일 수도 있음.(비싼 오퍼레이션). 테스트 할 때마다 이런 오퍼레이션을 호출하는 건 비효율적이다.

<br>

#### 해결1. 인터페이스를 구현한 가짜(mock) Elvis를 생성해서 사용하자. 


<br>인터페이스
``` java
public interface IElvis {

    void leaveTheBuilding();

    void sing();
}
```

이를 구현한 mock Elvis
``` java
public class MockElvis implements IElvis {
    @Override
    public void leaveTheBuilding() {

    }

    @Override
    public void sing() {
        System.out.println("You ain't nothin' but a hound dog.");
    }
}
```


수정된 테스트 코드
``` java
class ConcertTest {

    @Test
    void perform() {
        Concert concert = new Concert(new MockElvis());
        concert.perform();

        assertTrue(concert.isLightsOn());
        assertTrue(concert.isMainStateOpen());
    }

}
```

인터페이스를 사용하므로 Elvis와 Concert 코드도 인터페이스 기반 코드로 바꿔야함

<br>

#### 단점2. 리플렉션 API 사용 시 private 생성자를 호출 가능

리플렉션 코드
``` java
public class ElvisReflection {

    public static void main(String[] args) {
        try {
            Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor(); //클래스 정보를 통해 생성자에 접근
            defaultConstructor.setAccessible(true); //true 줘야 private 생성자에 접근 가능
            Elvis elvis1 = defaultConstructor.newInstance();
            Elvis elvis2 = defaultConstructor.newInstance();
            Elvis.INSTANCE.sing();
            System.out.println(elvis1==elvis2); //각기 다른 인스턴스
        	System.out.println(elvis1==Elvis.INSTANCE); //싱글턴 인스턴스와도 다름
        } catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

}
```
<br>리플렉션 API를 사용하면 private 생성자에 접근 가능하고 이를 통해 새로운 인스턴스를 생성할 수 있다. 

<br>

#### 해결2. private 생성자에 카운팅을 하거나 flag를 두자. 
아래는 flag를 사용하여 해결하는 방식

```java
public class Elvis implements IElvis{

    public static final Elvis INSTANCE = new Elvis();
    private static boolean created;

    private Elvis() {
        if (created) {
            throw new UnsupportedOperationException("can't be created by constructor.");
        }

        created = true;
    }
    
    //아래 코드 생략
}
```


<br>

#### 단점3. 역직렬화 시 새로운 인스턴스 생성 가능 
> 직렬화, 역직렬화란?
https://velog.io/@sw_smj/Java-%EC%A7%81%EB%A0%AC%ED%99%94%EC%99%80-%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94

직렬화, 역직렬화 코드
``` java
public class EnumElvisSerialization {

    public static void main(String[] args) {
    //직렬화
        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
            out.writeObject(Elvis.INSTANCE);
        } catch (IOException e) {
            e.printStackTrace();
        }
   //역직렬화
        try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
            Elvis elvis = (Elvis) in.readObject();
            System.out.println(elvis == Elvis.INSTANCE); //다름
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
읽어올 때 새로운 인스턴스가 생성됨. 

<br>

#### 해결3. Elvis에 다음 readReslove() 메소드를 둔다. 

``` java
private Object readResolve() {
        return INSTANCE;
    }
```

역직렬화 시 이 메소드가 호출되어(오버라이딩 비스무리하게 호출됨) 싱글톤 객체를 반환함.

<br>
<br>

### 2. static 팩토리 메소드 방식


``` java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

}
```



1번 방식과 유사해보이지만 다르다. 우선, 1에서는 static 필드를 public으로 선언해서 싱글턴 객체를 필드로 직접 접근했지만 이 방식은 필드를 private으로 선언했다. 그리고, static 팩토리 메소드를 public으로 두고 이 메소드에서 인스턴스를 제공한다. 
<br>즉, 1번 방식은 필드로 싱글턴 객체에 직접 접근을 하고 2번 방식은 static 팩토리 메소드를 통해 싱글턴 객체에 접근한다.

<br>

#### 장점
1. 클라이언트 코드를 바꾸지 않고 싱글턴이 아니게 변경할 수 있다.
2. 제네릭 클래스에도 적용 가능 
3. 메소드 참조를 공급자로 사용 가능

<br>

#### 장점1. 클라이언트 코드를 바꾸지 않고 싱글턴이 아니게 변경할 수 있다.


static 팩토리 메소드에서 새로운 인스턴스를 생성하는 코드로 바꾸면(new 키워드) 클라이언트에서는 코드 변경 없이 싱글톤 객체가 아닌 새로운 객체를 받을 수 있다.

``` java
public class Elvis{
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return new Elvis(); }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();

        System.out.println(Elvis.getInstance());
        System.out.println(Elvis.getInstance());
    }

}
```

<br>

#### 장점2. 제네릭 클래스에도 적용 가능

인스턴스는 동일하고 타입만 다르게 사용 가능

```java
public class MetaElvis<T> {

    private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();

    private MetaElvis() { }

    @SuppressWarnings("unchecked")
    public static <E> MetaElvis<E> getInstance() { return (MetaElvis<E>) INSTANCE; }

    public void say(T t) {
        System.out.println(t);
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public static void main(String[] args) {
        MetaElvis<String> elvis1 = MetaElvis.getInstance();
        MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
        System.out.println(elvis1);
        System.out.println(elvis2);
        elvis1.say("hello");
        elvis2.say(100);
    }

}
```

<br>

#### 장점3. 메소드 참조를 공급자로 사용 가능

메소드 참조 - Supplier<T>
 
  
 ``` java
public interface Supplier<T>{
	T get();
}
```
  Supplier<T>의 get() 메소드처럼 생긴(매개변수x, 반환값 존재) 어떤 메소드든 이 Supplier<T>인터페이스를 구현하지 않고 사용 가능.
  
  
<br>

  
  ``` java
  public class Elvis implements Singer {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { 
	    return INSTANCE; 
    }
}
```
 getInstance()가 바로 이 get() 메소드에 준하는 메소드라 볼 수 있다.
  
  <br>


  ``` java
public class Concert {

    public void start(Supplier<Singer> singerSupplier) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }

    public static void main(String[] args) {
        Concert concert = new Concert();
        concert.start(Elvis::getInstance);
    }
}
  ```
  
start()메소드에서 Supplier<T> 타입의 매개변수인 singerSupplier 선언함. 이때, main함수에서 start() 메소드를 호출하는데 이 Supplier<T>의 get()에 준하는 Elvis 클래스의 getInstance() 메소드를 인자로 주면 singerSupplier.get() 호출 시 getInstance()의 반환값 즉, 싱글턴 객체가 리턴됨.

<br>

#### 단점
은 1번 방식과 동일하고 해결 방법도 동일하다.

<br>
<br>
  

### 3. Enum 방식

``` java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```

따로 private 생성자를 두거나 팩토리 메소드를 선언한다거나 필드를 두지 않고 이렇게만 선언하면 된다. 
이러한 방식이 권장되는데 이유는 리플렉션과 역직렬화에 안전하기 때문이다. <br>
  
  #### 어떻게 안전?
  
  
1. 리플렉션
Elvis.class.getDeclaredConstructor() 호출 시 에러 발생
enum Elvis의 컴파일된 자바 바이트 코드를 보면 private 생성자가 존재하긴 함.
하지만, 열거형 자체가 new로 생성할 수 없음. 열거형 선언 시에 선언한 것들만 사용 가능. 내부적으로 생성자를 호출할 수 없게 막아뒀다.
<br>따라서 리플렉션을 방지하기 위한 별도의 수단을 쓰지 않아도 된다.


2. 역직렬화
별도의 수단 쓰지 않아도 역직렬화 시 새로운 객체가 아닌 원래 객체를 얻을 수 있다.

3. 클라이언트 코드 테스트
enum도 인터페이스 구현 가능함. 필요 시 인터페이스 구현하고 테스트 코드에서 목킹으로 대체해서 사용하면 된다.
