---
title: "[Effective Java] Comparable을 구현할지 고려하라"
date: 2024-07-17 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
  - 이펙티브자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>순서를 고려해야하는 값 클래스를 작성한다면 꼭 `Comparable` 인터페이스를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.   
>`compareTo` 메서드에서 필드의 값을 비교할 때 `<`와`>` 연산자는 지양해야 한다.   
>그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator`가 제공하는 비교자 생성 메서드를 이용하자.

## Comparable 인터페이스
---
`compareTo()`가 정의된 인터페이스이다.

`compareTo()`는 순서비교를 위한 메서드이다.

해당 인터페이스를 구현하면 TreeSet이나 Collections와 같은 순서 이용한 클래스와 메서드를 이용할 수 있다.


## compareTo()
---
> compareTo는 해당 객체와 전달된 객체의 순서를 비교한다.

compareTo는 Object의 equal와 유사하지만 차이점이 존재한다.   
그것은 동치성뿐만 아니라 순서까지 비교할 수 있다는 것이다.

`compareTo()`를 구현할 때는 아래와 같은 일반 규약을 지키는 것이 좋다.   
>아래 설명에서 `sgn`은 부호함수를 뜻하며, 표현식의 값이 `음수`, `0`, `양수`일 때 `-1`, `0`, `1`을 반환하도록 정의했다.

#### 대칭성
두 객체참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다.  

> Comparable을 구현한 클래스는 모든 x, y에 대하여 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.   
> 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.

#### 추이성
첫 번째가 두번째보다 크고 두 번째가 세번째보다 크면 첫번째는 세번째보다 커야한다.  
> Comparable을 구현한 클래스는 추이성을 보장해야 한다. 
> 즉, (x.compareTo(y)>0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.

#### 반사성
크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.  
> Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.

#### equals
`compareTo` 메서드로 수행한 동치성테스트 결과가 `equals`와 같아야한다.  
>(x.compareTo(y) == 0) == (x.equals(y))여야 한다. 
>Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. (주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.)

## compareTo() 작성 요령
---
- `Comparable`은 타입을 인수로 받는 **제네릭 인터페이스**이므로 `compareTo`의 인수타입은 컴파일 시에 정해지기 때문에 입력 **인수 확인이나 형변환을 할 필요가 없다.**
- **null을 인수**로 넣으면 `NullPointerException`을 던져야한다.
- `compareTo`는 **동치가 아닌 순서를 비교**한다.
- 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다.
- `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 `Comparator`을 대신 사용한다.

위의 설명은 어쩌면 당연한 것일수도 있다.   
중요한 것은 주의할 점이다.

#### `compareTo()`에서 관계연산자 (<, >)
이는 오류를 유발할 수 있기 때문에 박싱된 클래스의 compare을 사용하는 것이 좋다.   
ex) `Integer.compare()`

#### 핵심 필드가 여러개일 때
가장 핵심적인 필드부터 비교하는 것이 좋다.

핵심적인 필드가 다를 확률이 높기 때문에 성능상 장점을 가져올 수 있다.

#### 추이성을 위반 - 해시코드 값의 차를 기준으로 하는 비교자
```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
};
```

이 방식은 정수 오버플로를 일으키거나 부동소수점 계산방식에 따른 오류를 발생시킬 수 있어 사용하면 안된다.   
따라서 아래의 두 방식을 대신 사용하도록 하자.

#### 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
};
```

#### 정적 compare 메서드를 활용한 비교자
```java
static Comparator<Object> hashCodeOrder = 
Comparator.comparingInt(o -> o.hashCode());
```



