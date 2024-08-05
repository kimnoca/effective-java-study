_해당 게시글은 이펙티브 자바 교재를 보고 정리한 글입니다._

<br>


### 매개변수화 타입은 불공변이다.
매개변수화 타입은 불공변(invariant)이다. 즉, 서로 다른 타입 Typel과 Type2가 있을 때 List<Typel\>은 List<Type2\>의 하위 타입도 상위 타입도 아니다.

예를 들어, List<String\>은 List<Object\>의 하위 타입이 아니다. 왜냐하면, List<Object\>에는 어떤 객체든 넣을 수 있지만 List<String\>에는 문자열만 넣을 수 있다. 즉, List<String\>은 List<Object\>가 하는 일을 제대로 수행하지 못하므로 하위타입이 아니다.(리스코프 치환 원칙 위배)

> 리스코프 치환 원칙(LSP)이란?
객체지향 설계의 5가지 원칙인 SOLID 중 하나로,
부모 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스를 바꿀 수 있어야한다는 것.

그러나, 때론 이러한 불공변 방식보다 유연한 방식이 필요할 때가 있다. 이때, 한정적 와일드 카드가 유용하게 사용될 수 있다.

<br>
<br>

### 한정적 와일드카드를 사용해보자.
컬렉션 프레임워크 중 하나인 Stack<E\> 클래스에 **pushAll()**이라는 메서드를 추가한다고 하자. 이때, 이 메서드는 일련의 원소를 스택에 넣는 기능을 한다.


#### 와일드카드 타입 사용 전
``` java
public void pushAll(Iterable<E> src) {
	for (E e : src)
		push(e);
}
```
이때, 다음 3번째 줄 코드를 실행하면 오류가 뜬다.

``` java
Stack<Number> numberstack = new Stack<>();
Iterable<Integer> integers =...;
numberstack.pushAll(integers); //오류 발생

```
Iterable<Number\> 타입의 매개변수에 Iterable<Integer\>형을 전달하려고 해서 오류가 발생하는 것이다.(Iterable src의 원소 타입이 Stack의 원소 타입과 일치하면 잘 동작함.)


<br>

#### 와일드카드 타입 사용 후

매개변수를 다음과 같이 선언하면 Iterable src의 원소 타입이 Stack의 원소 타입의 하위 타입인 경우(같을 때도 포함)에도 잘 동작한다.

``` java
public void pushAll(Iterable<? extends E> src) {
	for (E e : src)
		push(e);
}
```
이떄, src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 **생산자**(데이터를 생성하거나 제공하는 역할. 데이터의 추가, 삽입 등)라고 볼 수 있다.

<br>

-----



이제, Stack<E\> 클래스에 **popAll()**이라는 메서드를 추가한다고 하자. 이때, 이 메서드는 Stack 안의 모든 원소를 주어진 컬렉션으로 옮겨 담는다


#### 와일드카드 타입 사용 전

``` java
public void popAll(Collection<E> dst) {
	while (!isEmpty())
		dst.add(pop());
}	
```
마찬가지로, 다음 3번째 줄 코드 실행 시 오류가 발생한다.

``` java
Stack<Number> numberstack = new Stack<>();
Collection<Object> objects = ...;
numberstack.popAll(objects); //오류 발생
```
Collection<Number\> 타입의 매개변수에 Colletion<Object\>형을 전달하려고 해서 오류가 발생하는 것이다.

<br>

#### 와일드카드 타입 사용 후

매개변수를 다음과 같이 선언하면 Collection dst의 원소 타입이 Stack의 원소 타입의 상위 타입인 경우(같을 때도 포함)에도 잘 동작한다.

``` java
public void popAll(Collection<? super E> dst) {
	while (!isEmpty())
		dst.add(pop());
}
```
이때, dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 **소비자**(데이터를 읽거나 사용하는 역할. 데이터 처리, 출력 등)라고 볼 수 있다.

<br>

----

#### 결론: 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용해라.
 ~~텍스트단, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입으 쓰지 말아야한다. 이때는 타입을 정확하게 지정해야한다.

``` java
<T> void copy(List<T> src, List<T> dst) {
    for (T item : src) {
        dst.add(item);
    }
}
```

이 메서드는 src 리스트에서 원소를 읽어서 dst 리스트에 원소를 추가한다. 이 경우 src에서 데이터를 쓰고(생산), dst에 데이터를 읽는(소비)  역할을 동시에 한다. 따라서, 와일드카드 타입이 아닌 정확한 타입 T를 사용해야 한다. 와일드카드를 사용할 경우 타입이 불명확해져서, 두 리스트 간의 호환성을 보장할 수 없게 된다.~~


<br>
<br>

### 펙스(PECS) 공식
다음 공식을 통해 어떤 와일드카드 타입을 써야 하는지 기억하자.
> producer-extends, consumer-super

 매개변수화 타입 t가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<?
super T>`를 사용하라.

이전 item에서 등장한 메서드와 생성자 선언을 다시 수정해보자.

#### ex1) Chooser 생성자(item28)
다음 chocies 컬렉션은 T 타입의 값을 생산하는 역할을 한다.

``` java
public Chooser(Collection<T> choices)
```
PECS 공식을 적용하면 다음과 같다.
``` java
public Chooser(Collection<? extends T> choices) {
```
PECS 공식을 적용하기 전에는 Chooser<Number\>의 생성자의 매개변수로 List<Integer\>를 못넘기지만, 공식 적용 후에는 넘길 수 있다.

<br>


#### ex2) union 메서드(코드30-2)
다음 union 메서드의 s1, s2는 모두 E의 생성자이다.
``` java
public static <E> Set<E> union(Set<E> s1, Set<E> s2);
```
공식을 적용하면 다음과 같다.
``` java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);
```
다음과 같은 코드를 실행시킬 수 있다.
```java
Set<Integer> integers = Set.of(1, 2, 3);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles)
```


<br>
<br>

### 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
  
 타입 매개변수와 와일드카드 모두 사용 가능한 경우가 있다.
 
 예를 들어, swap 연산을 두 방식으로 모두 정의하면 다음과 같다.
 
 ``` java
public static <E> void swap(List<E> list, int i, int j); //타입 매개변수 사용
public static void swap(List<?> list, int i, int j); //와일드카드 사용
```
public API라면 와일드카드를 사용하는 두 번째 방식이 낫다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄 것이기 때문이다.

<br>


> 기본 규칙은 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체한다.
>> 이때, 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 
한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

<br>

그러나, 두 번째 방식으로 swap을 다음과 같이 구현하면 문제가 발생한다.
``` java
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i))); 
}
```

list.get(i)으로 꺼낸 원소를 다시 list의 j번째 인덱스에 넣을 수 없다.
- List<?>는 어떤 타입의 리스트인지 구체적으로 알 수 없습니다. 따라서, 리스트의 요소를 수정하거나 추가할 때 컴파일러는 타입의 안전성을 보장할 수 없습니다.

- list.get(i) 메서드는 Object 타입을 반환합니다. 하지만 list.set(i, value) 메서드는 T 타입의 값을 요구합니다. 여기서 T는 List<?>가 가리키는 실제 타입을 의미합니다. 컴파일러는 T가 무엇인지 알 수 없기 때문에, Object 타입을 T 타입으로 변환할 수 없어서 오류가 발생합니다.

<br>

이를 해결하기 위해 와일드카드 타입의 실제 타입을 알려주는 private 메서드를 선언하면 된다.(도우미 메서드)


``` java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(if list.set(j, list.get(i)));
}
```

swapHelper 메서드는 리스트가 List<E\>임을 알고 있다. 즉, 이 리스트에서 꺼낸 값의 타입은 항상 e이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다.
