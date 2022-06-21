---
title: "[JAVA] Java Refelction 마스터 강의 1장"
author: "yeoji21"
date: 2022-06-19
categories: [Backend, Java]
tags: [java, 자바]
---

## 들어가면서
유데미에 있는 [Java 심화 과정: Java Reflection 마스터](https://www.udemy.com/share/106k34/) 강의 내용을 정리한 글입니다. 자세한 내용은 강의를 통해 확인해주시길 바랍니다. 

# 1. Java Reflection 개요
## 리플렉션이란
- 실행하는 동안 애플리케이션 클래스와 객체 정보를 추출할 수 있는 JVM 기능이다.
- JDK의 리플렉션 API로 구현할 수 있다.
- 리플렉션을 통해 다양한 sw 컴포넌트를 연결하고 소스 코드의 수정 없이 새로운 프로그램 순서를 만들 수 있다.
- 프로그램을 실행하면서 객체와 클래스를 분석하고 이를 입력값으로 사용할 수 있어 훌륭한 라이브러리와 프레임워크를 만들고 뛰어난 sw 설계를 할 수 있다.

## 리플렉션 사용 예시
- 대표적으로 `JUnit`이 리플렉션을 사용한 프레임워크
- @BeforeEach, @Test 등의 어노테이션을 보고 테스트하고자 하는 클래스의 인스턴스를 만들고 클래스의 메소드를 찾아서 실행함

- `Spring`과 `Google Guice` 등 또한 리플렉션을 사용한 프레임워크
- Spring은 리플렉션을 통해 자동으로 의존성을 주입할 수 있음
  
- 라이브러리에서도 리플렉션을 사용해 프로토콜 간 변환을 실행할 수 있음
- `Jackson`같은 라이브러리는 Json이 입력값으로 들어오면 리플렉션을 사용해서 클래스를 확인하고 필드를 전부 분석함
- 그리고 필드명에 따라 json 문자열에서 가져온 값만 입력한다
- 이외에도 리플렉션은 로깅 프레임워크, ORM과 웹 프레임워크 개발자 도구에도 사용

## 리플렉션의 문제점
- 보이지 않는 구조에 접근할 수 있기 때문에 문제가 발생할 수 있음
- 목적에 맞지 않게 사용되면 코드를 변경해야 하고 실행속도가 느려지기도 함
- 큰 힘에는 큰 책임이 따른다

[YouTube - 리플렉션 2강 - 왜 리플렉션을 써야 하나](https://youtu.be/AyQwvxRJ0q0)


# 2. Reflection API 게이트웨이와 와일드카드
## Reflection의 진입점, Class 타입
- 자바에서 대문자로 시작하는 `Class`는 앱 코드를 보여주는 진입점
- `Class` 타입 객체에는 객체의 런타임에 관한 정보가 있거나 특정 클래스가 있음
- 그리고 어떤 메소드와 필드를 포함하는지 
- 어떤 클래스를 extends하고 어떤 인터페이스를 implements하는지 알 수 있음

## Class 객체를 얻는 방법
1번. 객체의 인스턴스에 `getClass()` 메소드 사용 

```java
Map<String, String> map = new HashMap();
Class<?> mapClass = map.getClass(); //return HashMap class
```
   - 위 경우에 Class는 Map이 아닌 HashMap 클래스를 나타냄
   - 왜냐면 객체는 변수의 런타임 타입이기 때문 
   - 또한 원시타입은 객체가 아니기에 위 방법을 사용하지 못함

2번. 타입 이름에 `.class ` 사용
```java
Class<String> stringClass = String.class;
Class booleanType = boolean.class;
``` 
- 이 방법을 통해 원시 타입의 Class를 얻을 수 있음
- 메소드 매개변수나 클래스 멤버가 원시타입일 수 있기때문에 확인할 때 이 내용을 기억해야 함
 

3번. 정적 메소드 `Class.forName()` 사용
```java
Class<?> stringType = Class.forName("java.lang.String");
Class<?> engineType = Class.forName("vehicles.Car$Engine");
```
- class path를 통해 동적으로 클래스를 찾을 수 있음 -> 패키지명 포함
- 내부 클래스의 정보에 접근하려면 `$` 기호를 사용
- 앞의 두 방식과 다르게 **런타임 에러**를 발생시킬 수 있어 가장 위험한 방식
- 하지만 Class 객체를 찾기 위해서 forName 메소드를 써야만 하는 경우도 있음
- 첫 번째로 인스턴스를 확인하거나 만들려는 타입이 사용자 정의 구성 파일(ex. xml)에서 전달될 때 forName()을 써야만 함
- 그렇게 하면 외부 파일을 변경하면 소스 코드를 변경하거나 다시 컴파일하지 않고 동작을 변경할 수 있음 
- 두 번째로, 클래스를 불러와서 사용하는 애플리케이션과 분리해서 별도로 라이브러리를 구축하는 경우에 매우 편리함

## 추가 <?>는 무슨 의미?
- 자바 와일드카드
- 자바에서 Integer는 Number를 extends하고 String은 CharSequence를 implements하지만 **List\<Number\>가 List\<Integer\>의 상위 클래스가 되는 것은 아님**  
- 하지만 와일드카드라고 부르는 `?`는 제네릭 타입 목록이면 전부 해당하는 상위 타입
- 컴파일하는 동안 컴파일러가 제네릭 타입을 정확하게 알 수 없을 때 와일드카드를 유용하게 사용할 수 있음
- 예를들어 forName()으로 Class를 찾는 경우
- 그리고 클래스의 제네릭 매개변수가 제네릭 타입이라면 와일드카드를 사용해야 함

```java
Class<?> carClass = Class.forName("vehicles.Car");
Map<String, String> genericMap = new HashMap();
Class<?> hashMapClass = genericMap.getClass();
```


