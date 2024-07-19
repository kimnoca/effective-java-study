---
title: "[Effective Java] clone의 재정의는 항상 주의해서 진행하라"
date: 2024-07-10 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
  - 이펙티브자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>인터페이스를 만들때 절대 `Cloneable`을 확장해서는 안된다. `Cloneable`은 믹스인 용도로 만들어진 것이다.   
 `final` 클래스라면 성능 최적화 관점에서 검토후 문제가 없을때만 `Cloneable`을 구현하자.   
 객체의 복제 기능은 `Cloneable/clone` 방식보다 **복사 팩터리와 복사 생성자**를 이용하는것이 가장 좋다.   
 단, 배열같은 경우는 `clone`방식을 가장 적합하므로 예외로 친다.

## 얕은 복사 VS 깊은 복사
---
#### 얕은 복사
![](images/2024-07-17-Effective-Java-Item13-1.png)

#### 깊은 복사
![](Pasted%20image%2020240717212529.png)
## Cloneable 인터페이스
---
- 믹스인 인터페이스이다.
- 아무내용이 없다.
- 이 인터페이스를 구현했다는 것은 clone을 사용가능하다는 것이다.

보통 인터페이스라 하면 정의된 기능을 사용한다는 것을 뜻한다.   
하지만 Cloneable의 경우 Object의 clone()동작 방식을 변경한 것이다.

## clone()
---
```java
public class Object {
    // ...
    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
    // ...
}
```

Object의 clone은 기본적으로 protected이다.   
동작방식은 아래와 같다.
1. Cloneable을 구현한 클래스인지 확인한다.
	- 아니라면 CloneNotSupportedException이 터진다.
2. 객체를 복제한 새 객체를 반환한다.

clone을 재정의 할 때는 아래와 같은 규약을 지켜야한다.
    1. x.clone() != x : 새로운 객체여야 한다.
    2. x.clone().getClass() == x.getClass() : 같은 타입(클래스)여야 한다.
    3. x.clone().equals(x) : 필드가 모두 같아야 한다.

~~

## 복사 생성자 & 복사 팩터리
확장하려는 클래스가 Cloneable을 구현한 경우 어쩔 수 없이 clone()을 재정의해줘야 한다.    
하지만 그렇지 않은 상황에서는 복사 생성자 & 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

> 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다.

```java
public class Car {

    // 복사 생성자
    public Car(Car car) {
        ...
        return newCar;
    }

    // 복사 팩터리
    public static Car newInstance(Car car) {
        ...
        return newCar;
    }

}
```

이 방식은 인자를 받기 때문에 구현 클래스가 아니라 인터페이스를 받을 수도 있다.    
따라서 해당 인터페이스를 구현하는 클래스끼리는 다른 구현 클래스로의 복사도 가능해진다   
- (HashSet -> TreeSet 등).   
개인적으로는 정확하게 복사한다는 것을 명시해줄 수 있는 팩터리 방식이 더 좋아보인다.

