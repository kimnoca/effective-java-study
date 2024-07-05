Java 라이브러리에서는 close 메소드를 통해서 직접 닫아줘야하는 자원들이 많다.

전통적으로 자원이 닫힘을 보장하는 수단으로 `try-finally` 를 사용했다.

```java
public static void file_read_try_catch_finally(String filePath) throws IOException {
    FileInputStream fileInputStream = null;
    BufferedInputStream bufferedInputStream = null;
    try {
        fileInputStream = new FileInputStream(filePath);
        bufferedInputStream = new BufferedInputStream(fileInputStream);
        int data;
        while ((data = bufferedInputStream.read()) != -1) {
            System.out.println((char) data);
        }

    } finally {
        if (fileInputStream != null) {
            fileInputStream.close();
        }
        if (bufferedInputStream != null) {
            bufferedInputStream.close();
        }
    }
}

```

이 방식은 회수를 해야하는 자원이 많아질수록 코드가 지저분해진다.

이 문제를 Java 7에서 부터 지원하는 `try-with-resourcee` 를 통해서 해결 되었다.

### try-with-resources 사용예시

```java
public static void file_read_try_with_resources(String filePath) throws IOException {
    try (FileInputStream fileInputStream = new FileInputStream(filePath);
        BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream)) {
        int data;
        while ((data = bufferedInputStream.read()) != -1) {
            System.out.print((char) data);
        }
    }
}
```

다음 코드는 `file_read_try_catch_finally` 메소드를 `try-with-resources`를 사용해 작성한 예시 이다. `try-with-resources` 를 사용하기 위해서는 해당 자원이 `AutoClosealbe` 인터페이스를 구현해야 한다.

### try-with-resources를 이용한 예외처리

try-catch-finally를 이용하면 에러가 발생해도 에럭 스택 트레이스가 누락되는 경우가 발생할 수 있다. 예를 들어 아래와 같은 간단한 자원을 반납해야하는 클래스가 있다고하자.

```java
public class MyResources implements AutoCloseable {
    @Override
    public void close() throws RuntimeException {
        System.out.println("My Resources Close");
        throw new IllegalStateException();
    }


    public void method() {
        System.out.println("My Resources method");
        throw new IllegalStateException();
    }
}
```

`close` 메소드와 `method` 메소드를 호출시 `IllegalStateException`이 발생하도록 하였다.

**try-catch-finally로 자원을 반납하는 경우**

```java
public class Main {

    public static void main(String[] args) {
        exception_try_finally();
    }

    public static void exception_try_finally() {
        MyResources myResources = null;

        try {
            myResources = new MyResources();
            myResources.method();

        } finally {
            if (myResources != null) {
                myResources.close();
            }
        }
    }
}
```

실행결과

```
Exception in thread "main" java.lang.IllegalStateException
	at Item09.MyResources.close(MyResources.java:7)
	at Item09.Main.exception_try_finally(Main.java:23)
	at Item09.Main.main(Main.java:11)
```

코드를 실행하면 `method` 메소드에서 발생한 에러 트레이스가 정상적으로 찍히지 않는다.

**try-with-resources로 자원을 반납하는 경우**

```java
public class Main {

    public static void main(String[] args) {
        exception_try_with_resources();
    }

    public static void exception_try_with_resources() {
        try (MyResources myResources = new MyResources()) {
            myResources.method();
        }
    }
}
```

실행결과

```
Exception in thread "main" java.lang.IllegalStateException
	at Item09.MyResources.method(MyResources.java:13)
	at Item09.Main.exception_try_with_resources(Main.java:16)
	at Item09.Main.main(Main.java:10)
	Suppressed: java.lang.IllegalStateException
		at Item09.MyResources.close(MyResources.java:7)
		at Item09.Main.exception_try_with_resources(Main.java:15)
		... 1 more
```

`try-with-resources`를 이용한다면 `method`와 `close` 모두 에러 트레이스를 모두 남길 수 있다.
