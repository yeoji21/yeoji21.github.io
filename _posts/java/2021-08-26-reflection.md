---
title: "리플렉션 (Reflection)"
author: "yeoji21"
date: 2021-08-27
categories: [Backend, Java]
tags: [java, 자바]
---

자바 스터디 12주차 애너테이션 과제를 진행하면서 리플렉션에 대한 언급이 있었는데 워낙 중요한 개념이기도 하고 언급될 때마다 항상 애매하게 이해하고 넘어갔어서 이번에 리플렉션만 따로  빼서 포스팅하려고 한다.  

## 리플렉션(Reflection)이란?
리플렉션은 **"구체적인 클래스의 타입을 알지 못해도, 클래스의 메소드, 타입, 변수 등에 접근할 수 있도록 해주는 자바 API"** 라고 정의할 수 있다.  

이렇게 딱딱하고 교과서적인 정의를 보고 단번에 이해하기가 쉽지 않은데, 아래 코드를 한 번 보자. 

```java
class Digimon{
    public void attack(){
        System.out.println("attack");
    }

    public static void main(String[] args) {
        Object digimon = new Digimon();
        digimon.attack();
    }
}
```

`Object` 타입으로 변수를 선언하고 `Digimon` 클래스의 인스턴스를 담았다. 여기까지는 문제가 없으나, `attack()` 메소드를 호출하면 Object 타입으로 선언된 digimon 객체는 Object의 메소드나 변수에만 접근 가능하기 때문에 컴파일 에러가 발생한다. 

왜냐면 동적 로딩에 의해 `main()`이 실행되고 객체가 생성되어 메소드가 호출될 때,
그 때가 되어서야 클래스가 로드되기 때문이다

리플렉션의 정의에서 "구체적인 클래스의 타입을 알지 못해도" 라는 말은 즉, **Java는 컴파일 시점에 클래스의 타입이 결정되기 때문에 런타임에서 어떤 클래스의 타입이 사용될지 모른다**는 것을 의미한다. 

따라서 다시 정리하면, 리플렉션은 이런 경우에 **컴파일 시점이 아닌 런타임 시점에서 클래스의 정보를 추출해낼 수 있는 기술**이라고 할 수 있다.

## 리플렉션의 원리
Java에서는 모든 `.class` 파일 하나 당 `java.lang.Class` 객체가 하나씩 생성된다. Class는 모든 .class파일들의 메타 데이터를 가지고 있으며 .class 파일에 같이 저장된다.  

모든 .class파일들은 이 클래스를 최초로 사용하는 시점에 동적으로 `Class Loader`를 통해 JVM에 로드된다. 

최초로 사용하는 시점은 해당 .class 파일에 `static`을 최초로 사용할 때를 의미한다. 생성자도 static 메소드이기 때문에 `new` 키워드와 함께 사용하게 되면 클래스가 로드되는데 이를 **동적 로딩**이라고 한다.  

이렇게 .class파일의 정보와 Class 객체는 JVM에 `Runtime Data Area`의 `Method Area`에 저장된다. 
> Method Area == Class Area == Static Area

리플렉션을 사용하면 런타임 시에 위 영역에 접근하여 클래스의 정보, 접근제어자, 패키지, 어노테이션 등을 알아낼 수 있는 것이다. 

## 리플렉션은 언제 사용해야 하나

작성 시점에는 어떤 클래스를 사용해야 할지 모르지만, 런타임 시점에서 클래스를 가져와서 실행해야 할 경우에 필요하다. 즉, **동적으로 클래스를 사용해야 할 때** 리플렉션을 사용한다. 

이렇게 말하면 **"내가 만든 프로그램인데, 사용할 클래스의 이름과 타입을 모르는게 말이 돼?"** 라고 할 수 있으나, 라이브러리나 프레임워크는 사용하는 사람이 어떤 클래스를 만들지 모른다. 이럴 때 동적으로 해결해주기 위해서 리플렉션을 사용하는 것이다.

이런 상황의 대표적인 예가 바로 `JDBC`이다. 
```java
Class.forName("oracle.jdbc.driver.OracleDriver");
```
자바 가상머신이 동작을 시작하고 코드가 실행되기 전까지는 어떤 JDBC의 드라이버가 사용될지 알 수 없다. 

왜냐하면 JDBC는 JDBC DB 라이브러리 (Oracle, MsSql, MySql) 모두 `static`하게 링크하여 컴파일 하는 방식이 아닌 해당 라이브러리를 설치한 상태에서 **필요할 때 호출할 수 있도록 동적 로딩을 사용**하기 때문이다. 

이 외에도 리플렉션을 사용하는 대표적인 예시로 Intellij의 자동완성 기능, Spring의 Annotations 등이 있다.  

## Class 클래스
Java의 모든 클래스와 인터페이스는 컴파일 후 `.class` 파일로 생성된다. `.class`파일에는 클래스의 정보(멤버변수, 메소드, 생성자 등)가 담겨있는데 `Class 클래스`는 이러한 class 파일에서 클래스의 정보를 가져올 수 있다. 

Class 클래스의 객체를 받아오는 방법은 세 가지가 있다.  
1. Object 클래스의 메서드인 getClass()
   ```java
    String str = new String();
    Class class = str.getClass();
   ```

2. 컴파일된 상태로 Class 호출
   ```java
    Class class = String.class;
   ```

3. forName()을 이용한 동적로딩
   ```java
    Class class = Class.forName("java.lang.String");
   ```

세 가지 방법 중 `forName()`을 통해 가져오는 방법이 가장 많이 사용되고, 이 것을 **동적 로딩**이라 한다.  

동적 로딩이라고 부르는 이유는 보통 다른 클래스 파일을 불러올 때 컴파일 시 `static`에 그 클래스 파일이 같이 바인딩되지만, `forName()`으로 class 파일을 불러올 때는 컴파일에 바인딩이 되지않고 런타임 시에 불러오기 때문이다. 

Class 클래스의 주요 메소드

|메소드명|리턴타입|설명|
|--:|--:|--:|
|getFields()|Field[]|접근 제어자가 public인 필드들을 Field 배열로 반환<br> 부모 클래스의 필드들도 함께 반환한다.|
|getConstructors()|	Constructor[]|	접근 제어자가 public인 생성자들을 Constructor 배열로 반환.|
|getMethods()|Method[]|접근 제어자가 public인 메서드들을 Method 배열로 반환. <br> 부모 클래스의 메서드들도 함께 반환한다.|
|getDeclaredFields()|Field[]|접근 제어자에 상관없이 모든 필드들을 Field배열로 반환. <br> 부모 클래스의 필드들은 반환하지 않는다.|
|getDeclaredConstructors()|	Constructor[]|접근 제어자에 상관없이 모든 생성자들을 Constructor배열로 반환.<br> 부모 클래스의 생성자들은 반환하지 않는다.|
|getDeclaredMethod()|Method[]|접근 제어자에 상관없이 모든 메서드들을 Method배열로 반환. <br> 부모 클래스의 메서드들은 반환하지 않는다.|

`Class` 클래스의 다른 메소드들과 `Method` 클래스와 `Field` 클래스의 메소드들이 궁금하면 [길은 가면, 뒤에 있다.](https://12bme.tistory.com/129)   
이 블로그를 참조하길 추천한다.

## 리플렉션 사용 예시

예제에서 공통적으로 사용할 Monster 클래스와 Digimon 클래스는 아래와 같다.
```java
class Monster{
    public String name;

    public Monster() {
    }

    public Monster(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

```java
class Digimon extends Monster{
    public int strength = 100;
    public String partner;

    public Digimon() {
    }

    public Digimon(String partner) {
        this.partner = partner;
    }

    private Digimon(int strength) {
        this.strength = strength;
    }

    public void attack(){
        System.out.println("attack!!");
    }

    public String getPartner() {
        return partner;
    }
    private void secret(){
        System.out.println("this is private method");
    }
}
```

### Constructor 찾기

`getDeclaredConstructors()`는 해당 클래스의 `private`, `public` 등의 모든 생성자를 리턴한다. (부모 클래스의 생성자는 x)

`getConstructors()`는 해당 클래스의 `public` 생성자만 리턴한다.
```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");

Constructor[] declaredConstructors = digimon.getDeclaredConstructors();
System.out.println("getDeclaredConstructors => ");
for (Constructor declaredConstructor : declaredConstructors) {
    System.out.println(declaredConstructor);
}

System.out.println("getConstructors => ");
Constructor[] constructors = digimon.getConstructors();
for (Constructor constructor : constructors) {
    System.out.println(constructor);
}
```
```console
getDeclaredConstructors => 
private com.example.javastudy.week12.Digimon(int)
public com.example.javastudy.week12.Digimon(java.lang.String)
public com.example.javastudy.week12.Digimon()

getConstructors => 
public com.example.javastudy.week12.Digimon(java.lang.String)
public com.example.javastudy.week12.Digimon()
```

다음 예제에서는 `getConstructor()`의 인자로 특정 생성자의 인자를 지정해 하나의 생성자를 가져온다. 여기서는 Digimon 클래스의 partner를 인자로 하는 생성자를 받아와서 `newInstance()`를 사용해 객체 생성 후 `attack()` 메소드까지 실행하였다.

다만 여기서 newInstance()를 사용하면 Object 클래스의 객체로 받아오기 때문에 명시적으로 **타입 캐스팅**을 해주어야 한다. 

```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");

Constructor constructor = digimon.getConstructor(String.class);
Digimon agumon = (Digimon)constructor.newInstance("taeil");
agumon.attack();
```
```console
attack!!
```
메소드 호출까지 잘 된것을 알 수 있다. 

### Method 찾기
모든 해당 클래스에 정의된 메소드를 찾으려면 `getDeclaredMethods()`를 사용한다. Class 클래스의 메소드들 중 `Declared`가 들어간 메소드들은 공통적으로 부모 클래스의 정보는 가져오지 않는다. 

`getMethods()`는 해당 클래스에서 `public`으로 선언된 메소드들만 가져오지만, 부모 클래스로부터 상속받은 메소드들도 모두 찾아준다. 

```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");

Method[] declaredMethods = digimon.getDeclaredMethods();
System.out.println("getDeclaredMethods =>");
for (Method declaredMethod : declaredMethods) {
    System.out.println(declaredMethod);
}

Method[] methods = digimon.getMethods();
System.out.println("getMethods =>");
for (Method method : methods) {
    System.out.println(method);
}
```
```console
getDeclaredMethods =>
public void com.example.javastudy.week12.Digimon.attack()
public java.lang.String com.example.javastudy.week12.Digimon.getPartner()
private void com.example.javastudy.week12.Digimon.secret()

getMethods =>
public void com.example.javastudy.week12.Digimon.attack()
public java.lang.String com.example.javastudy.week12.Digimon.getPartner()
public java.lang.String com.example.javastudy.week12.Monster.getName()
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public java.lang.String java.lang.Object.toString()
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
```

생성자를 호출할 때와 마찬가지로 메소드명과 인자를 넘겨서 특성 메소드를 받아올 수 있다.
```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");
Object instance = digimon.newInstance();
Method attack = digimon.getMethod("attack", null);
Method getName = digimon.getMethod("getName", null);
Method secret = digimon.getDeclaredMethod("secret", String.class);
```

### Method 호출
클래스로부터 메소드 정보를 가져온 뒤, 객체의 메소드를 호출할 수 있다.  

메소드 객체를 생성한 뒤, `Method.invoke()` 메소드를 호출하는데 첫 번째 인자는 호출하려는 객체, 두 번째 인자는 전달할 파라미터를 넘겨준다.  
```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");
Object instance = digimon.newInstance();

Method attack = digimon.getMethod("attack", null);
attack.invoke(instance);

Method secret = digimon.getDeclaredMethod("secret", String.class);
secret.setAccessible(true);
secret.invoke(instance, "hi this is private method");
```
`attack()`메소드는 인자가 없기때문에 `invoke()`의 인자로 객체의 인스턴스만 넘겨주었다. 

`secret()`메소드는 `private` 메소드이므로 `getDeclaredMethod()`를 통해 받아왔고, private 메소드에 접근하기 위해 `setAccesible()`의 인자로 true를 전달한 뒤 invoke() 하였다.

```console
attack!!
hi this is private method
```


### Field 변경
클래스로부터 멤버 변수 정보를 가져와 객체의 변수 값을 변경할 수도 있다. 
```java
Class digimon = Class.forName("com.example.javastudy.week12.Digimon");
Object instance = digimon.newInstance();
Field name = digimon.getField("name");
name.set(instance, "agumon");

Field partner = digimon.getField("partner");
partner.set(instance, "taeil");

System.out.println(name.get(instance));
System.out.println(partner.get(instance));
```
```console
agumon
taeil
```
반복된 예제들로 감을 잡았겠지만, 만약 `private` 멤버 변수의 값을 변경하고자 한다면 `getDeclaredField()`로 멤버 변수의 정보를 받아온 뒤, `Method.setAccesible(true)`를 실행한 뒤 `set()`을 호출하면 된다.


## 참조
<https://transferhwang.tistory.com/150>  
<https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=zzang9ha&logNo=222090490272>  
<https://ozofweird.tistory.com/entry/Java-Class-%ED%81%B4%EB%9E%98%EC%8A%A4>  
<https://velog.io/@foeverna/Java-%EC%9E%90%EB%B0%94-%EA%B8%B0%EB%B3%B8-%ED%81%B4%EB%9E%98%EC%8A%A4-Class-%ED%81%B4%EB%9E%98%EC%8A%A4>  
<https://ckdgus.tistory.com/89>  
<https://parkadd.tistory.com/54>  
<https://dublin-java.tistory.com/53>  
<https://joont.tistory.com/167>  