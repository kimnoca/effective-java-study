---
title: "[Effective Java] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
date: 2024-07-07 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)


> 하나의 클래스가 다른 클래스에 **의존하여 동작**하는 경우 정적 유틸리티와 싱글턴은 유연하지 못하다.   
> 생성자를 통해 넘겨는 주는 방식을 선택하자    
> 즉, Dependency Injection을 사용하라    

## 정적 유틸리티와 싱글턴의 잘못된 사용
---
맞춤법 검사기를 예로 들 수 있다.   
맞춤법 검사기의 경우 **특정 사전을 기준**으로 맞춤법을 검사한다.   
이 경우 **정적 유틸리티로 클래스나 싱글턴으로** 만드는 경우가 있다.

### 유틸리티 클래스와 싱글턴 클래스
---
아래와 같은 코드가 주어졌을 때 우리는 `SpellChecker.isValid(word)`와 같이 사용할 수 있다.   

#### 유틸리티 클래스
```java
public class SpellChecker{
	
	private static final Lexicon dictionary = new Lexicon();

	private SpellChecker(){} // 유틸리티 클래스 이므로 객체 생성 방지

	public static boolean isValid(String word){
		// dictionary 클래스를 이용한 맞춤법 검사 로직
	}
}
```

싱글턴 클래스로 만들었을 때는 
#### 싱글턴 클래스
```java
public class SpellChecker{
	
	private final Lexicon dictionary = new Lexicon();

	private SpellChecker(){} //객체 생성 방지
	public static SpellChecker INSTANCE = new new Spellchecker();

	public static boolean isValid(String word){
		// dictionary 클래스를 이용한 맞춤법 검사 로직
	}
}
```

하지만! 이는 **유연하지 못하다.**

만약, 사용자가 **다른 사전을 기준**으로 맞춤법검사를 하고자 한다면?   
사전을 final로 선언해두었기 때문에 **dictionary를 바꾸는 것은 불가능**하다.
그렇다면 아래와 같이 final을 제거하고 dictionary를 설정하는 메서드를 사용면 안되나?

#### final을 제거한 방식
```java
// dictionary에 final을 제거
public class SpellChecker{
	
	private static Lexicon dictionary = new Lexicon();

	private SpellChecker(){} // 유틸리티 클래스 이므로 객체 생성 방지

	// dictionary를 바꿔줌
	public static void changeDictionary(Lexcion dictionary){
		this.dictionary = dictionary;
	}

	public static boolean isValid(String word){
		// dictionary 클래스를 이용한 맞춤법 검사 로직
	}
}
```

결론은 **안된다.** 이 경우, **멀티스레드 환경**에서 문제가 생긴다.    

하나의 스레드가 SpellChecker를 사용하기 위해 `changeDictionary`를 사용했을 때, 다른 스레드가 `isValid`를 사용하면 어떻게 될까? 의도치 않은 dictionary로 맞춤법 검사를 할 수 있다. 

즉, **동시성 문제**가 생긴다. 이 경우, mutex 설정을 해준다는 등, 다른 조치가 필요하다.    
이는 적합한 사용 방법이 아니다.

## 진짜 해결책 DI (Dependency Injection)
---
외부의 자원을 사용한다면 해당 자원을 **사용자가 주입하여 사용하도록 한다는 것**이다.    
SpellChecker를 사용하고 싶다면 **dictionary에 대한 정보를 주입**하여 사용하도록한다.

이를 의존성 주입(DI, Dependency Injection)이라 한다.

이때, **생성자를 통해서 주입**하도록 한다.

```java
public class SpellChecker{
	
	private fianl Lexicon dictionary;

	//객체를 생성할 때 dictionary를 지정하고 바꾸지 못하도록 한다.
	public SpellChecker(Lexion dictionary){
		this.dictionary = diactionsary;
	}

	public static boolean isValid(String word){
		// dictionary 클래스를 이용한 맞춤법 검사 로직
	}
}
```

위와 같이 객체를 만들면 `SpellChecker`의 맞춤법 검사 로직은 **주입받은 dictionary의 기준에 따라 동작**한다.

따라서, 여러 기준(다양한 사전)을 기준으로 **여러 `SpellChecker`**를 만들 수 있게 된다.

## 활용! 자원 팩토리를 넘겨주는 방법
---
생성자에 객체를 넘겨주는 것이 아닌, **자원 팩토리를 넘겨주는 방식**도 있다.

> **팩토리**란 호출할 때마다 **특정 타입의 인스턴스를 반복해서 만들어주는 객체**를 말한다.

```java
public Mosaic create(Supplier<? extends Tile> tileFactory){
	...
}
```

위 코드는 Tile들을 반환하는 tileFactory를 넘겨준 경우다.

이 경우 create함수를 통해서 tileFactory의 Tile로 Mosaic를 만들어주는 기능을 하게 된다.

## Spring의 Container
---
프로젝트가 커지게 되면 이러한 의존성을 관리하는 것이 매우 어렵게 된다.

따라서 Spring의 경우 Spring Container를 이용해 의존성을 주입해준다.

즉, AutoWired와 같은 어노테이션으로 의존성 주입을 자동으로 이루어지게 할 수 있다.

