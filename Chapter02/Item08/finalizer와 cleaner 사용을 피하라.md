_해당 게시글은 이펙티브 자바 교재와 백기선님의 인프런 강의를 보고 정리한 글입니다._
<br>

item8에서는 객체 소멸자인 finalizer와 cleaner를 사용하면 안되는 이유를 열거하고 이를 대신할 AutoClosable에 대해 설명한다. 


<br>
<br>


### finalizer와 cleaner 쓰면 안되는 이유

<br>

#### 1. finalizer와 cleaner는 즉시 수행된다는 보장이 없다.

<br>
finalizer와 cleaner로 제때 실행되어야 하는 작업은 절대 할 수 없다.

ex) 파일 리소스를 반납하는 상황 . 
시스템에서 동시에 파일 여는 개수 제한되있음. 반납이 제때 되지 않아 새로운 파일을 열지 못하는 상황이 발생할 수 있음

<br>

#### 2. 얼마나 빠르게 실행될지 알 수 없다.

<br>
finalizer와 cleaner가 얼마나 빠르게 수행될지는 전적으로 가비지 컬렉터 알고리즘에 달렸고 이는 가비지 컬렉터 마다 다름.

테스트한 JVM에선 완벽하게 동작하더라도 고객 시스템에선 재앙을 일으킬 수도 있다.

<br>

#### 3. 쓰레드 우선순위가 낮다.

<br>
finalizer 스레드는 다른 스레드보다 우선순위가 낮아 실행 기회를 얻지 못할 수 있다. 따라서, 자원 회수가 지연될 수 있고 이는 OutOfMemoryError로 이어질 수 있다.

cleaner는 일부 개선되었지만 즉각적인 수행 보장은 없다.

<br>

#### 4. finalizer와 cleaner는 실행되지 않을 수도 있다.

<br>

수행 여부조차 보장하지 않는다. 따라서, 상태를 영구적으로 수행하는 작업에서는 절대 의존하면 안됨.

ex) 데이터베이스 같은 공유 자원의 락 해제를 finalizer나 cleaner에 맡기면 분산 시스템 전체가 서서히 멈출 것임.
<br>


#### 5.  finalizer 동작 중에 예외가 발생하면 정리 작업이 처리되지 않을 수도 있다. 
<br>

예외 발생 시 자원 반납 안된 채로 그대로 끝날 수 있음



<br>

#### 6. finalizer와 cleaner는 심각한 성능 문제가 있다.



<br>

#### 7. finalizer는 보안 문제도 있다.

<br>

생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다.
이 finalizer는 정적 필드에 자신의 참조를 할당해 가비지 컬렉터가 객체를 수집하지 못하게 하고, 그 객체의 메서드를 호출해 허용되지 않은 작업을 수행할 수 있다. 

<br>
<br>
<br>

### finalizer 사용 예시

<br>
사용하고자 하는 클래스에 finalize() 메소드를 오버라이딩 하면 됨. 이 메소드는 Object 클래스에 정의된 메소드임. 

```java
public class FinalizerIsBad {

    @Override
    protected void finalize() throws Throwable {
        System.out.print("");
    }
}
```
<br>

while문에서  객체가 무한정 생성되는데 레퍼런스 되지않으므로 GC 대상이 됨. 따라서, finalize 메소드가 호출된다.

Finalizer라는 클래스가 있는데 이 안에 있는 queue가 있고 여기에 finalize 메소드가 들어감.

백만개쯤 생성될 때 이 큐에 얼마나 많은 오브젝트가 쌓여있는지 출력함.

``` java
public class App {

    /**
     * 코드 참고 https://www.baeldung.com/java-finalize
     */
    public static void main(String[] args) throws InterruptedException, ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
        int i = 0;
        while(true) {
            i++;
            new FinalizerIsBad(); 

            if ((i % 1_000_000) == 0) {
                Class<?> finalizerClass = Class.forName("java.lang.ref.Finalizer");
                Field queueStaticField = finalizerClass.getDeclaredField("queue");
                queueStaticField.setAccessible(true);
                ReferenceQueue<Object> referenceQueue = (ReferenceQueue) queueStaticField.get(null);

                Field queueLengthField = ReferenceQueue.class.getDeclaredField("queueLength");
                queueLengthField.setAccessible(true);
                long queueLength = (long) queueLengthField.get(referenceQueue);
                System.out.format("There are %d references in the queue%n", queueLength);
            }
        }
    }
}
```

실제 출력을 보면 10만개 정도 쌓이는 것을 확인할 수 있는데

이는 객체를 생성하느라 바빠서 Finalizer의 queue에 들어있는 레퍼런스를 정리하지 못한 것임.(큐를 처리하는 쓰레드의 우선순위가 더 낮기 때문) → 쓰지말장

그리고, finalize() 안에서 다른 오브젝트나 자기 자신을 참조한다거나 만들어서 쓴다거나 하면 오히려 오브젝트가 늘어나게 됨. 조심


<br>
<br>
<br>


### cleaner 사용 예시

<br>


BigObject 클래스에서 resource는 반납해야하는 자원이라 하자. 

Runnable 구현에서 finalizer와 동일한 작업을 수행함. 즉, 여기서 리소스 정리 작업 진행.  

이때, 이 클래스에서 절대 BigObject 에 대한 레퍼런스가 있으면 안됨. 작업하다가 다시 객체가 생겨날 수 있기 때문. 여기서는 BigObject가 아닌 객체가 들고 있는 리소스를 참조함.

``` java
public class BigObject {

    private List<Object> resource; //반납해야하는 자원

    public BigObject(List<Object> resource) {
        this.resource = resource;
    }

    public static class ResourceCleaner implements Runnable {

        private List<Object> resourceToClean;

        public ResourceCleaner(List<Object> resourceToClean) {
            this.resourceToClean = resourceToClean;
        }

        @Override
        public void run() {
            resourceToClean = null;
            System.out.println("cleaned up.");
        }
    }
}

```
<br>

사용 예제는 다음과 같다.

Cleaner 객체를 생성하고 GC하고자 하는 객체를 클리너에 등록함. 그러면, ResourceCleaner에서 정의한 태스크를 사용해서 GC 작업을 함.



``` java
public class CleanerIsNotGood {

    public static void main(String[] args) throws InterruptedException {
        Cleaner cleaner = Cleaner.create();

        List<Object> resourceToCleanUp = new ArrayList<>();
        BigObject bigObject = new BigObject(resourceToCleanUp);
        cleaner.register(bigObject, new BigObject.ResourceCleaner(resourceToCleanUp));

        bigObject = null;
        System.gc();
        Thread.sleep(3000L);
    }

}
```


<br>
<br>
<br>

### 권장 방법ㅣAutoCloseable 구현, try-with-resource 사용


<br>

AutoClosable 인터페이스를 구현하고 close 메소드에서 정리를 한다.

``` java
public class AutoClosableIsGood implements AutoCloseable {

    private BufferedInputStream inputStream;

    @Override
    public void close() {
        try {
            inputStream.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
<br>

클라이언트 코드에선  try-with-resource 구문을 통해 자원을 종료시킨다. 왜 이렇게 해야하는지는 아이템9에서

``` java
public class App {

    public static void main(String[] args) {
        try(AutoClosableIsGood good = new AutoClosableIsGood("")) {
            //TODO자원 반납 처리가 됨.
}
    }
}
```



<br>
<br>
<br>


### 그렇다면 cleaner는 언제 쓰는게 좋냐

<br>

cleaner는 안전망!
<br>
1. AutoClosable을 구현했는데 클라이언트 코드에서 try-with-resource를 안쓸 수 있다. 안전망을 두자! 
    
GC할 때 자원을 반납할 수 있는 기회를 가질 수 있도록(물론, 호출되리라는 보장은 없다.)
    
<br>
2. 네이티브 피어 자원 반납

마찬가지로 안전망. 하지만, AutoClosable 방식을 사용하는게 좋다.



<br>
<br>
<br>

### 안전망 예제
<br>

Room 클래스는 AutoCloseable도 구현하고 내부에 Cleaner도 존재

``` java
public class Room implements AutoCloseable {
    private static final Cleanercleaner= Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable =cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

<br>

 try-with-resource 구문을 사용하므로 AutoClosable 방식으로 자원이 정리된다.
 
 ``` java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```


<br>
try-with-resource 구문 없이 Room 객체를 생성하고 가비지 컬렉션을  호출하므로 안전망인 cleaner가 호출되어 정리 작업이 진행됨.

``` java
public class Teenager {

    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");

        // 다음 줄의 주석을 해제한 후 동작을 다시 확인해보자.
        // 단, 가비지 컬렉러를 강제로 호출하는 이런 방식에 의존해서는 절대 안 된다!
        System.gc();
    }
}
```
