---
title: "[JAVA] Java Refelction 마스터 강의 3장"
author: "yeoji21"
date: 2022-06-21
categories: [Backend, Java]
tags: [java, 자바]
---

## 들어가면서
유데미의 [Java 심화 과정: Java Reflection 마스터](https://www.udemy.com/share/106k34/) 강의 내용을 정리한 글입니다. 자세한 내용은 강의를 통해 확인해주시길 바랍니다. 

# 필드 검사와 배열 검사
## 필드 개요
- 자바 클래스에서 필드는 클래스나 인터페이스 안에서 선언된 변수를 의미
- 클래스 인스턴스에 따라 메모리와 값은 다름
- Reflection에서 `Field` 클래스는 각 필드를 의미하며, 각 필드 객체에는 필드 이름이 있고 필드 타입과 다양한 특성이 있다. 

## Field 클래스를 얻는 방법
- `Class.getDeclaredFields()`
	- 클래스에 선언된 필드 배열을 반환하는 메소드
	- 접근 제어자에 상관없이 모든 필드를 포함
	- 단, 상속된 필드는 제외됨
- `Class.getFields()`
	- 모든 public 필드를 얻을 수 있는 메소드
	- 상속받은 필드까지 모든 public 필드가 포함됨
- 필드 이름을 안다면, 필드 이름을 전당해서 바로 필드 객체를 만들 수 있음
	- Class.getDeclaredField(fieldName)
	- Class.getField(fieldName)
	- 주어진 이름에 해당하는 필드를 찾을 수 없다면 `NoSuchFieldException` 발생


## Synthetic class 필드
- 클래스에 명시하여 선언한 필드 외에도 java 컴파일러가 내부 사용을 위해 인위적으로 필드를 생성함
- 이는 리플렉션을 사용해서 찾지 않는 한 보이지 않는 필드
- 이러한 필드를 `synthetic field`라 함
- synthetic field는 컴파일러가 정한 특정한 이름을 갖고, 그 이름과 내용을 사용자가 의도적으로 변경하지 않음
- 주어진 필드가 synthetic field 인지는 `Field.isSynthetic()` 메소드를 통해 확인할 수 있음

```java
public static void main(String[] args) throws IllegalAccessException {
    printDeclaredFieldsInfo(Movie.MovieStats.class);
}

public static void printDeclaredFieldsInfo(Class<?> clazz) throws IllegalAccessException {
    for (Field field : clazz.getDeclaredFields()) {
        System.out.println(String.format("Field name : %s type : %s",
                field.getName(),
                field.getType().getName()));
        System.out.println(String.format("Is synthetic field : %s", field.isSynthetic()));
        System.out.println();
    }
}

public static class Movie extends Product {
    public static final double MINIMUM_PRICE = 10.99;

    private boolean isReleased;
    private Category category;
    private double actualPrice;

    public Movie(String name, int year, double price, boolean isReleased, Category category) {
        super(name, year);
        this.isReleased = isReleased;
        this.category = category;
        this.actualPrice = Math.max(price, MINIMUM_PRICE);
    }

    public class MovieStats {
        private double timesWatched;

        public MovieStats(double timesWatched) {
            this.timesWatched = timesWatched;
        }

        public double getRevenue() {
            return timesWatched * actualPrice;
        }
    }
}
```

위 예제에서 `Movie` 클래스의 내부 클래스인 `MovieStats` 클래스를 살펴보면 **내부 클래스**기때문에 Movie 클래스의 필드인 actualPrice에 접근할 수 있는 것을 알 수 있다.  
이 점을 기억하고 MovieStats의 Field를 확인해보면 결과는 다음과 같다. 

```
Field name : timesWatched type : double
Is synthetic field : false

Field name : this$0 type : exercises.ch3.Main$Movie
Is synthetic field : true
```

Movie 클래스와 `this`라는 필드명으로 연결되어 있는 것을 볼 수 있으며, 이는 synthetic field이다. 

이를 통해 유추하면 컴파일러가 Movie 클래스의 인스턴스와 MovieStats 클래스의 인스턴스를 연결하는 것을 알 수 있다.   
synthetic field의 변수를 수정하면 예상치 않은 오류가 발생할 수 있으니 사용자가 의도적으로 수정하지 않는 것이 좋다.

또 다른 예로 `enum class`인 Category의 Field를 분석하면 다음과 같다
```java
public enum Category {
    ADVENTURE,
    ACTION,
    COMEDY
}
```

```
Field name : ADVENTURE type : exercises.ch3.Main$Category
Is synthetic field : false

Field name : ACTION type : exercises.ch3.Main$Category
Is synthetic field : false

Field name : COMEDY type : exercises.ch3.Main$Category
Is synthetic field : false

Field name : $VALUES type : [Lexercises.ch3.Main$Category;
Is synthetic field : true
```

enum type의 각 값에 대응하는 필드를 확인할 수 있고,  `$VALUES`라는 synthetic 필드도 확인할 수 있다. 
이는 컴파일러가 생성한 필드로, enum이 가지는 모든 필드를 나타낸 배열을 포함한다


## Field 클래스로부터 값을 얻는 방법
- `Field.get(instance)`
- `get()` 메소드의 인자로 인스턴스를 전달하면 해당 인스턴스가 가지는 필드값을 리턴
- 클래스 static 필드값은 인자로 null을 전달해도 값을 출력할 수 있으나 이외의 필드는 인스턴스를 전달하지 않으면 NPE 발생

## 배열
- 배열은 자바의 매우 중요한 구조체로 Reflection API에서 특별한 기능이 있음

 - 자바에서 배열은 특별한 조항이기에 `Class<?> API`를 사용해 모든 차원의 배열에 관한 기본 정보를 얻을 수 있음
 - `Class.isArray()` 메소드를 통해 해당 객체가 배열임을 확인하고, 
 - `Class.getComponentType()` 메소드로 배열 요소의 타입을 알 수 있다

- `Array` 클래스는 배열 객체에서 데이터를 얻을 수 있도록 도와주는 정적 메소드만을 가지는 헬퍼 클래스
- Array 클래스의 정적 메소드로는 `Array.getLength(Object arrayObject)`, `Array.get(Object arrayObject, int index)` 등이 있음


## 정리
- 리플렉션으로 클래스나 필드 객체의 정보를 실행 시 동적으로 가져올 수 있다
- 이 기능으로 객체 타입을 미리 알 수 없거나 프로그램이 실행되기 전까지 존재하지 않는 제네릭 알고리즘과 라이브러리를 작성할 수 있다. 