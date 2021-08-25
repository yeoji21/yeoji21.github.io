
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

----------------------------------

애너테이션이란?
프로그램 소스코드 안에 다른 프로그램을 위한 정보를 미리 약속된 형식으로 포함시킨 것이 바로 애너테이션이다. 
애너테이션은 주석처럼 프로그래밍 언어에 영향을 미치지 않으면서도 다른 프로그램에게 유용한 정보를 제공할 수 있다는 장점이 있다.
> annotation의 뜻은 주석, 주해, 메모이다.

예를 들어, 테스트 코드를 작성할 때 `@Test` 애너테이션을 메소드 앞에 붙이는데 @Test는 이 메소드를 테스트해야한다는 것을 테스트 프로그램에게 알리는 역할을 할 뿐, 
메소드가 포함된 프로그램 자체에는 아무런 영향을 미치지 않는다. 주석처럼 존재하지 않는 것이나 다름없다.

그렇다고 모든 프로그램에서 의미있는 것은 아니다. 해당 프로그램에 미리 정의된 종류와 형식으로 작성해야만 의미가 있다.
@Test는 테스트 프로그램을 제외한 다른 프로그램에게는 아무런 의미가 없는 정보일 것이다. 

애너테이션은 JDK에서 기본적으로 제공하는 것과 다른 프로그램에서 제공하는 것들이 있는데, 어느 것이든 그저 약속된 형식으로 정보를 제공하기만 하면 된다. 
JDK에서 제공하는 표준 애너테이션은 주로 컴파일러를 위한 것으로 컴파일러에게 유용한 정보를 제공한다. 그리고 새로운 애너테이션을 정의할 때 사용하는 메타 애너테이션을 제공한다.
> JDK에서 제공하는 애너테이션은 java.lang.annotation 패키지에 포함되어 있다. 

표준 애너테이션 
자바에서 기본적으로 제공하는 애너테이션은 몇 개 없다. 그나마도 애너테이션을 정의하는데 사용하는 메타 애너테이션(meta annotation)이다. 

|애너테이션|설명|
|--:|--:|
|@Override|컴파일러에게 오버라이딩 메소드임을 알린다|
|@Deprecated|앞으로 사용하지 않을 것을 권장하는 대상에 붙인다|
|@SuppressWarnings|컴파일러의 특정 경고메시지가 나타나지 않게 해준다|
|@SafeVarargs|지네릭스 타입의 가변인자에 사용한다 (JDK 1.7)|
|@FunctionalInterface|함수형 인터페이스임을 알린다(JDK 1.8)|
|@Native|native메소드에 참조되는 상수 앞에 붙인다(JDK 1.8)|
|@Target *|애너테이션이 적용가능한 대상을 지정하는데 사용한다|
|@Documented *|애너테이션 정보가 javadoc으로 작성된 문서에 포함되게 한다|
|@Inherited *|애너테이션이 자손 클래스에 상속되도록 한다|
|@Retention *|애너테이션이 유지되는 범위를 지정하는데 사용한다|
|@Repreatable *|애너테이션을 반복해서 적용할 수 있게 한다(JDK 1.8)|

> *가 붙은 것은 메타 애너테이션

메타 에너테이션
앞서 설명한 것과 같이 메타 애너테이션은 '애너테이션을 위한 애너테이션' 즉, 애너테이션을 정의할 때 애너테이션의 적용대상(target)이나 유지기간(retention) 등을 지정하는데 사용된다.  

@Target
애너테이션이 적용가능한 대상을 지정하는데 사용된다. 
아래는 @SuppressWarnings를 정의한 것인데, 이 애너테이션에 적용할 수 있는 대상을 @Target으로 지정하고 있다. 애너테이션의 인자로 여러 개의 값을 지정할 때는 배열에서처럼 괄호{}를 사용한다.
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

@Target으로 지정할 수 있는 애너테이션 적용대상의 종류는 아래와 같다.

|대상 타입|의미|
|--:|--:|
|ANNOTATION_TYPE|애너테이션|
|CONSTRUCTOR|생성자|
|FIELD|필드(멤버변수, enum 상수)|
|LOCAL_VARIABLE|지역변수|
|METHOD|메소드|
|PACKAGE|패키지|
|PARAMETER|매개변수|
|TYPE|타입(클래스, 인터페이스, enum)|
|TYPE_PARAMETER|타입 매개변수 (JDK 1.8)|
|TYPE_USE|타입이 사용되는 모든 곳(JDK 1.8)|

TYPE은 타입을 선언할 때 애너테이션을 붙일 수 있다는 뜻이고, TYPE_USE는 해당 타입의 변수를 선언할 때 붙일 수 있다는 뜻이다.
표의 값을은 java.lang.annotation.ElementType이라는 열거형에 정의되어 있으며, static import하면 ElementType.TYPE을 TYPE과 같이 간단히 할 수 있다. 
FILED는 기본형에, TYPE_USE는 참조형에 사용된다는 점에 주의하자

```java
@Target({FIELD, TYPE, TYPE_USE})
@interface MyAnnotation{ }

@MyAnnotation       //TYPE
public class AnnotationTest {
    @MyAnnotation   //FIELD
    int a;
    
    @MyAnnotation   //TYPE_USE
    AnnotationTest at;
}
```

@Retention
애너테이션이 유지되는 기간을 지정하는데 사용된다. 애너테이션의 유지 정책(retention policy)의 종류는 다음과 같다.
SOURCE - 소스파일에만 존재. 클래스파일에는 존재하지 않음
CLASS - 클래스 파일에 존재. 실행시에 사용불가. 기본값
RUNTIME - 클래스 파일에 존재. 실행시에 사용가능

@Override나 OSuppressWarnings처럼 컴파일러가 사용하는 애너테이션은 유지 정책이 SOURCE이다. 컴파일러를 직접 작성할 것이 아니면 이 유지 정책은 필요없다.
유지정책을 RUNTIME으로 하면, 실행 시에 리플렉션(reflection)을 통해 클래스 파일에 저장된 애너테이션의 정보를 읽어서 처리할 수 있다.
CLASS는 컴파일러가 애너테이션의 정보를 클래스 파일에 저장할 수 있게는 하지만, 클래스 파일이 JVM에 로딩될 때는 애너테이션의 정보가 무시되어 실행 시에 애너테이션에 대한 정보를 얻을 수 없다.
따라서 CLASS가 기본값임에도 불구하고 잘 쓰이지 않는다.
> 지역변수에 붙은 애너테이션은 컴파일러만 인식할 수 있으므로, 유지정책이 RUNTIME인 애너테이션을 지역변수에 붙여도 실행 실에는 인식되지 않는다.

@Documented
애너테이션에 대한 정보가 javadoc으로 작성한 문서에 포함되도록 한다.

@Inherited
애너테이션이 자손 클래스에 상속되도록 한다. @Inherited가 붙은 애너테이션을 조상 클래스에 붙이면, 자손 클래스도 이 애너테이션이 붙은 것과 같은 효과를 받는다.

@Repeatable
@Repeatable이 붙은 애너테이션은 여러 번 붙일 수 있다.

```java
@Repeatable(ToDos.class)
@interface ToDo{
    String value();
}

@ToDo("delete test codes")
@ToDo("override inherited methods")
public class AnnotationTest {

}
```
다만 일반적인 애너테이션과 달리 같은 이름의 애너테이션이 여러 개가 하나의 대상에 적용될 수 있기때문에 이 애너테이션들을 하나로 묶어서 다룰 수 있는 애너테이션도 추가로 정의해야 한다.
```java
@interface ToDos{
    ToDo[] value();
}

@Repeatable(ToDos.class)
@interface ToDo{
    String value();
}
```

@Native
네이티브 메소드에 의해 참조되는 상수 필드에 붙이는 애너테이션이다. 아래는 java.lang.Long 클래스에 정의된 상수이다.
```java
@Native public static final long MIN_VALUE = 0x8000000000000000L;
@Native public static final long MAX_VALUE = 0x7fffffffffffffffL;
```

네이티브 메소드는 JVM이 설치된 OS의 메소드를 말한다. 네이티브 메소드는 보통 C언어로 작성되어 있는데, 자바에서는 메소드의 선언부만 정의하고 구현은 하지 않는다.
그래서 추상 메소드처럼 선언부만 있고 몸통이 없다.

Object 클래스의 메소드들은 대부분 네이티브 메소드이다. 네이티브 메소드는 자바로 정의되어 있기 때문에 호출하는 방법은 자바의 일반 메소드와 같지만 실제로 호출되는 것은 OS의 메소드이다.
JNI(Java Native Interface)가 자바에 정의된 네이티브 메소드와 OS의 메소드를 연결해주는 작업을 한다. 

애너테이션 타입 정의하기
@기호를 붙이는 것을 제외하면 인터페이스를 정의하는 것과 동일하다
```java
@interface AnnotationName{
    Type name();    //애너테이션의 요소를 선언한다.
    ...
}
```
엄밀히 말해서 @Override는 애너테이션이고 Override는 애너테이션의 타입이다./

애너테이션의 요소
애너테이션 내에 선언된 메소드를 애너테이션의 요소(element)라고 하며, 아래에 선언된 TestInfo 애너테이션은 다섯 개의 요소를 갖는다. 

```java
@interface TestInfo{
    int count();
    String testedBy();
    String[] testTools();
    TestType testType();
    DateTime testDate();
}

enum TestType{ FIRST, FINAL}

@interface DateTime{
    String yymmdd();
    String hhmmss();
}
```

애너테이션의 요소는 반환값이 있고 매개변수는 없는 추상 메소드의 형태를 가지며, 상속을 통해 구현하지 않아도 된다.
다만, 애너테이션을 적용할 때 이 요소들의 값을 빠짐없이 지정해주어야 한다. 요소 이름도 같이 적어주므로 순서는 상관없다.
```java
@TestInfo(count = 3, testedBy = "yeoji",
testTools = {"JUnit", "AutoTester"},    //요소의 타입이 배열인 경우 괄호{}를 통해 여러 개의 값을 지정함
testType = TestType.FINAL, testDate = @DateTime(yymmdd = "210825",hhmmss = "201921"))
public class AnnotationTest {
    ...
}
```

애너테이션의 각 요소는 기본값을 가질 수 있다. 기본값이 있는 요소는 애너테이션을 적용할 때 값을 지정하지 않으면 기본값이 사용된다.
기본값을 지정할 요소의 타입이 배열인 경우 괄호{}를 사용해 여러 개의 값을 지정할 수 있다. 
```java
@interface TestInfo{
    int count() default 1;
    String[] names() default {"yeo","ji"};
}
```

애너테이션 요소가 오직 하나뿐이고 이름이 value인 경우, 애너테이션을 적용할 때 요소의 이름을 생략하고 값만 적어도 된다. 
```java
@interface TestInfo{
    String value();
}
@TestInfo("test")
public class AnnotationTest {
    ...
}
```

요소의 타입이 배열일 때도 요소의 이름이 value이면, 애너테이션 사용 시 요소의 이름을 생략할 수 있다. 예를 들어, @SuppressWarnings의 경우, 요소의 타입이 String 배열이고 이름이 value이기때문에 적용 시에 요소의 이름을 생략할 수 있는 것이다.
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
```java
@SuppressWarnings({"deprecation","unchecked"})
public class AnnotationTest {
    ...
}
```


java.lang.annotation.Annotation
모든 애너테이션의 조상은 Annotation 인터페이스이다. 
```java
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```
모든 애너테이션의 조상인 Annotation 인터페이스가 위와 같이 정의되어 있기 때문에, 모든 애너테이션 객체에 대해 equals(), hashCode(), toString()과 같은 메소드를 호출하는 것이 가능하다.
```java
Class<TestInfo> tic = TestInfo.class;
    Annotation[] annotations = TestInfo.class.getAnnotations();
    for (Annotation a : annotations) {
        System.out.println(a.toString());
        System.out.println(a.hashCode());
        System.out.println(a.equals(a));
        System.out.println(a.annotationType());
    }
```

마커 애너테이션 Marker Annotation
Serialization이나 Cloneable 인터페이스처럼, 요소가 하나도 정의되지 않은 애너테이션을 마커 애너테이션이라고 한다.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {}
```

애너테이션 요소의 규칙
- 요소의 타입은 기본형, String, enum, 애너테이션, Class만 허용된다.
- ()안에 매개변수를 선언할 수 없다.
- 예외를 선언할 수 없다.
- 요소를 타입 매개변수(<T>)로 정의할 수 없다.

-----------------------------------------------------------

애너테이션은 주석이기때문에 실행되지 않는 코드라고 생각안하는게 좋아요
마치 애너테이션이 기능을 가지고있다고 생각할 수 있는데, 애너테이션은 그저 표식, 주석, 마킹의 개념이다. 
그래서 런타임중에 알아내야 하는 값은 어노테이션에 넣을 수 없다. 컴파일러 수준에서 해석이되거나 정적인 값이어야 한다.

SOURCE는 말 그대로 소스에만 남아 있고 컴파일한 바이트코드에는 남지 않는다. 따라서 정말 주석과 같은 용도로 사용하거나 컴파일 타임에만 쓰이는 에너테이션이라는 의미이다.
예를 들어 @Override는 컴파일 타임에 오버라이딩 메소드가 맞는지 검사하는 에너테이션이므로 SOURCE로 정의되어있다.  

CLASS는 바이트코드까지 에너테이션을 남겨놓겠다는 의미이다. 하지만 런타임시에 클래스 로더가 클래스의 정보를 읽어서 메모리에 로드하는 과정에서 CLASS 에너테이션의 정보는 누락시킨다. 따라서 메모리에 적재되었을 때에도 에너테이션을 유지하려면 RUNTIME으로 정의해야 한다. 
RUNTIME으로 만들면 리플렉션이 가능해진다.
리플렉션에 대해 더 공부해보자 

getDeclaredFields() vs getFields()
getDeclaredFields()는 해당 클래스에 선언되어 있는 것 모두 가져오고(private 포함), getFields()는 밖에서 볼 수 있는 것들만 가져온다. 

@Target 안쓰면 해당 에너테이션은 아무데나 붙일 수 있음

javadoc과 에너테이션 프로세서 알아보기

java ServiceLoader알아보기
interface만 정의하고 그 것을 구현한 jar 파일을 갈아끼우면서? 그 구현체를 사용할 수 있는 것?

