---
title: "[JAVA] Java Refelction 마스터 강의 2장"
author: "yeoji21"
date: 2022-06-20
categories: [Backend, Java]
tags: [java, 자바]
---

## 들어가면서
유데미에 있는 [Java 심화 과정: Java Reflection 마스터](https://www.udemy.com/share/106k34/) 강의 내용을 정리한 글입니다. 자세한 내용은 강의를 통해 확인해주시길 바랍니다. 


# 1. 객체 생성과 생성자

## Constructor 클래스
- 자바 클래스의 생성자는 `Constructor` 클래스의 인스턴스로 나타냄
- 이 객체는 클래스 생성자의 모든 정보를 포함


## Constructor 클래스를 얻는 방법
- 첫 번째로 클래스의 생성자를 얻는 가장 유용한 메소드는 `Class.getDeclaredConstructors()`
  - 생성자의 접근자에 관계없이 클래스에 선언된 모든 생성자를 반환
- 두 번째는 `Class.getConstructors()` 메소드
  - 클래스의 public 생성자만 반환한다. 

- 특정 생성자를 찾을 때, 가져올 생성자의 매개변수 타입을 안다면 위 두 메소드에 매개변수에 타입을 나타내는 타입 목록을 전달할 수도 있다. 
```java
Class.getConstructors(Class<?> ... parameterTypes) or
Class.getDeclaredConstructors(Class<?>... parameterTypes)
-> 반환할 알맞은 생성자가 없다면 NoSuchMethodException
```


## 객체를 생성하는 방법
- 생성자가 주어진 클래스의 새로운 객체를 생성하기 위해 정적 메소드인 `Constructor.newInstance(Object… arguments)` 메소드를 사용
- 인수가 올바른 타입과 올바른 순서로 전달되고 생성자에 접근할 수 있다면, 특정 클래스의 생성자가 호출되고 이에 성공하면 클래스의 객체가 반환됨

## private constructor 호출하기
- private method와 private constructor는 해당 클래스 외부에서는 호출할 수 없음
- 하지만 리플렉션을 사용하면 어떤 접근 제한자에도 모두 접근 가능
- 단, private 생성자 사용 시 `constructor.setAccessible(true)`로 설정
- 이 기능은 특별한 경우를 위해 아껴 두어야 하고, 내부 생성자나 우리가 소유하지 않은 클래스에서 의도적으로 제한된 생성자에는 남용하면 안됨
- 합당한 경우에만 사용해 애플리케이션 설계 수준을 높여야 함

## package-private 클래스를 인스턴스화
- `package-private` 클래스는 클래스 선언에 접근 제한자가 없는 클래스를 말함
- 이는 내부 데이터 모델, 헬퍼 클래스 등 구현 상세 사항을 숨겨 혼란을 줄이는 역할을 함
- 그런데 외부 라이브러리 사용 시, 애플리케이션 내 package-private 클래스에 접근해야 하는 경우가 생김

다음 예에서 json으로 변환하는 `Request` 클래스는 package-private 클래스
```java
class HttpClient{
	private final ObjectMapper objectMapper;
	
	public float getPriceFromService(){
		Request request = buildRequest();
		byte[] requestData = objectMapper.writeValueAsBytes(request);
...
	}
}
```

- `ObjectMapper`는 리플렉션을 통해 package-private 클래스인 Request class의 필드를 분석하고 json으로 변환

## 의존성 주입 구현 예제
리플렉션을 통해 생성자를 획득하고 인스턴스를 생성하는 방식을 활용하여 애플리케이션 시작 시 클래스를 자동 생성하고 의존성 주입을 구현

```java
public static void main(String[] args) {
        Game game = createObjectRecursively(TicTacToeGame.class);
        game.startGame();
    }

    public static <T> T createObjectRecursively(Class<T> clazz) throws InvocationTargetException, InstantiationException, IllegalAccessException {
        Constructor<?> constructor = getFirstConstructor(clazz);
        List<Object> constructorArguments = new ArrayList<>();

        for (Class<?> parameterType : constructor.getParameterTypes()) {
            Object objectType = createObjectRecursively(parameterType);
            constructorArguments.add(objectType);
        }
        constructor.setAccessible(true);
        return (T) constructor.newInstance(constructorArguments.toArray());
    }


    private static Constructor<?> getFirstConstructor(Class<?> clazz) {
        Constructor<?>[] constructors = clazz.getDeclaredConstructors();
        if (constructors.length == 0) throw new IllegalArgumentException("no constructor");
        return constructors[0];
    }
```

위 예제 코드는 `TicTacToeGame` 클래스가 의존하는 클래스를 재귀적으로 생성하여 주입해주는 방식을 보여준다.
