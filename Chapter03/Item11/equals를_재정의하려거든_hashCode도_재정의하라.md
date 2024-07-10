---
title: "[Effective Java] equals를 재정의하려거든 hashCode도 재정의하라"
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

> equals를 재정의 할 때는 hashCode도 재정의 해야한다.   
> 서로 다른 인스턴스이면 해시값도 다르게 나오는게 성능상 좋다.   
> 같은 인스턴스(equals)라면 무조건 같은 해시값이 나와야한다.

우선 해당 글에서 같은 객체라는 표현은 논리적 동등성을 따졌을 때 같다는 의미이다.   
즉, equals 메서드를 사용했을 때 true를 반환한다는 의미이다.

## hashCode 재정의 해야하는 이유?
---
equals를 재정의하지 않았다면 굳이 재정의 할 필요는 없다.

하지만, 재정의 했다면 꼭 hashCode도 재정의 해줘야한다.

그 이유는 HashMap, HashSet과 같이 **hash를 이용하는 컬렉션을 사용할 때 문제가 생기기 때문**이다.

무슨 문제가 생기는지는 Object 명세에서 알 수 있다.

> equals 비교에 사용되는 **정보가 변경되지 않았다면** hashCode 의 값은 **항상 같은 값을 반환**한다.   
> equals 가 두 객체를 **같다고 판단**했으면 두 객체의 hashCode 는 **같은 값을 반환**한다.   
> equals 가 두 객체를 **다르다고 판단**했더라도 두 객체의 hashCode 가 **다를 필요는 없다.**

**이 중 두번째 조항때문에 문제가 생긴다.**

우리는 분명 equals를 통해서 같은 객체라고 판단했는데 hashCode가 다르다??

이건 논리적으로 맞지 않는 것이다. 따라서 아래와 같은 문제가 생긴다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309));
```

위 코드에서 우리는 PhoneNumber 객체를 넘겨줬다.  
그리고 해당 PhoneNumber를 조회하면 제니가 나오길 바랬다.   
하지만, 결과는 **null이 반환**된다. 그 이유는 **Object.hashCode를 사용해 비교**했기 때문이다.

**hashCode가 달라서 그냥 다른 객체구나 하고 넘어간 것이다.**

## 주의할 점
---
우선 Object 명세에서 알 수 있는 사실이 하나 있다.

> **hashCode가 같은 객체끼리만 동치일 가능성이 있다.**

즉, hashCode가 다르면 무조건 다른 객체이고 hashCode가 같으면 같은 객체일 수 도 있다.

그럼 일단 같은 객체끼리는 hashCode를 맞춰 줘야한다.

#### 그럼 그냥 같은 값으로 해야지
그냥 다 32로 반환하면 되는거 아닌가? 어차피 같은 객체끼리는 hashCode가 같을테니까?

이렇게 생각하는 것은 매우 위험한 생각이다. 이 경우 모든 객체를 equals로 비교해봐야한다.

절대로 이렇게 사용해선 안된다.

최대한 다른 객체끼리 hashCode가 겹치지 않도록 해야한다.

## 잘 정의하는 방법
---
**모든 핵심 필드에 대한 hashCode**를 구해 아래와 같이 적용한다.

> 핵심 필드 : equals에 사용한 모든 필드

**주의할 점은 핵심 필드가 아닌 필드는 hashCode의 과정에 들어가면 안된다.**

1. int result = c
    - c 는 아래의 2단계때 구함
2. c 의 계산 방식
    1. 기본 타입 필드(String, Integer 등)라면 Type.hashCode(필드)
	    - *Integer.hashCode(intVariable)*
    2. 참조 타입 필드(기본 타입 이외의 타입)라면 재귀적으로 호출, 계산이 복잡해진다면 표준형을 만들어 hashCode 호출
        - ex) getLottoMachine().getLottoTicket().getLotto().getLottoNumber()
    3. 배열 필드라면 핵심 원소에 대해 위의 1, 2 방식으로 해시코드 계산하여 비교, 모든 원소가 핵심 원소라면 Arrays.hashCode 이용
3. result = 31 * result + c
    - 31은 뭐야?

31인 이유는 명확하진 않지만 Collision을 줄여준다는 말이 있다.   
또한, 곱셈을 시프트 연산과 뺄셈 연산으로 최적화 할 수 잇다고 한다. (31 * i = (i << 5) - i)

#### 불변 객체에 대한 성능 향상 방법
객체가 불변이라면 hashCode는 변하지 않는다.  
이 경우 hashCode가 처음 호출 될 때 저장해두고 재호출 시 재활용 할 수 있다.

```java
private int hashCode;

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0) {
	    // 해시코드 계산 ...
	    hashCode = result;
    }
    return result;
}
```


## 그 외 주의할 점
---
첫번째로 성능을 위해서 핵심 필드를 생략하는 행위는 해서는 안된다.   
해당 필드가 hashCode를 퍼트려 주는 효과도 있다.

> 자바2 에선 16글자만 뽑아내 hashCode를 만들었다.   
> URL의 경우 겹치는 부분이 많은데 이 때문에 Collision이 많이 일어나는 문제가 생겼다.

두번재로 hashcode 값의 생성 규칙을 API 사용자에게 자세히 공표해서는 안된다.   
그래야 클라이언트가 이 값에 의지하여 코딩하는 일이 생기지 않는다.
