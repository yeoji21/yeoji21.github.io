
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

java ServiceLoader
interface만 정의하고 그 것을 구현한 jar 파일을 갈아끼우면서? 그 구현체를 사용할 수 있는 것?

-----------------------------------------------------------

### 람다식(lambda expression)

자바가 1996년에 처음 등장한 이후로 두 번의 큰 변화가 있었는데, 첫 번째는 JDK 1.5부터 추가된 제네릭(generics)의 등장이고, 두 번째는 JDK 1.8부터 추가된 람다식(lambda expression)의 등장이다. 
특히 람다식의 도입으로 인해, 자바는 객체지향 언어인 동시에 함수형 언어가 되었다. 

앞선 [3주차 스터디 - 연산자](https://yeoji21.github.io/posts/javastudy03-02/)에서 람다식을 사용하는 방법에 대해 간단히 언급했었는데, 기억을 되짚어보면서 람다식에 대해서 자세히 알아보자.


#### 람다식이란?
람다식은 간단히 말해서 메소드를 하나의 식(expression)으로 표현한 것이다. 람다식은 함수를 간략하면서도 명확한 식으로 표현할 수 있게 해준다. 메소드를 람다식으로 표현하면 메소드의 이름과 반환값이 없어지므로, 람다식을 익명 함수(anonymous function)이라고도 한다. 

Q. 여기서 말하는 메소드와 함수의 차이가 무엇일까?  
A. 우리가 프로그래밍에서 사용하는 함수라는 용어는 수학에서 따온 것이다. 그러나 객체지향 개념에서는 함수(function)대신 객체의 행위나 동작을 의미하는 메소드(method)라는 용어를 사용한다.  
메소드는 함수와 같은 의미이지만, 특정 클래스에 반드시 속해야 한다는 제약이 있기 때문에 함수와는 다른 용어를 선택해서 사용하게 되었다. 그러나 람다식을 사용하면 메소드가 하나의 독립적인 기능을 하기 때문에 함수라는 용어를 사용하는 것이다.  

### 람다식 사용법
람다식은 **메소드의 이름과 반환타입을 제거**하고 매개변수 선언부와 몸통{} 사이에 `->` 를 추가한다.  

예를 들어 두 값 중 더 큰 값을 반환하는 메소드 max가 있다.
```java
int max(int a, int b) {
    return a > b ? a : b;
}
```

이것을 람다식으로 변환하면 다음과 같다.
```java
(int a, int b) -> { return a > b ? a : b;}
```

위와 같이 반환값이 있는 메소드는 return문 대신 식으로 대체할 수 있다. 식의 연산 결과가 자동으로 반환값이 되는데 이 때 문장이 아니라 식이므로 끝에 `;` 를 붙이지 않아야 한다.
```java
(int a, int b) ->  a > b ? a : b 
```

람다식에 선언된 매개변수의 타입은 추론이 가능한 경우 생략 가능한데, 대부분의 경우에 생략 가능하다. 람다식에 반환 타입이 없는 이유는 항상 추론이 가능하기 때문이다. 
```java
(a, b) -> a > b ? a : b
```

만약 람다식에 선언된 매개변수가 하나뿐이라면 괄호()를 생략할 수 있다. 하지만 매개변수의 타입을 생략하지 않고 명시한다면 괄호를 생략할 수 없다.  
```java
a -> a * a      //ok
int a -> a* a   //error
```

마지막으로, 괄호{} 안의 문장이 하나인 경우, 괄호를 생략할 수 있다. 이 때 문장의 끝에 `;`를 붙이지 않아야 한다. 그러나 괄호 안의 문장이 return 문이라면 괄호를 생략할 수 없으니 주의하자.  

```java
(number, i) -> System.out.println(number + ":" + i)     //ok
() -> return (int)(Math.random() * 2)                   //error
```

### 함수형 인터페이스
자바에서 모든 메소드는 클래스 내에 포함되어야 하는데, 람다식은 어떤 클래스에 포함되는 것일까? 지금까지 람다식이 메소드와 동등한 것처럼 보였지만, 사실 람다식은 익명 클래스의 객체와 동일하다. 

```java
(int a, int b) -> a > b ? a : b
```
<center> <h2>🔃</h2> </center>

```java
new Object(){
    int max(int a, int b){
        return a > b ? a : b;
    }
}
```

그렇다면 람다식으로 정의된 익명 객체의 메소드를 어떻게 호출하는 것인지 차근차근 알아보자.

이미 우리가 알고있는 것처럼 참조변수가 있어야 객체의 메소드를 호출할 수 있을 것이다. 따라서 위의 예시코드에 나온 익명 객체의 주소를 f라는 참조변수에 저장해보자. 

```java
타입 f = (int a, int b) -> a > b ? a : b;
```
그렇다면 이 때 참조 변수 f의 타입은 어떤 것이어야 할까? 참조형이기에 클래스 또는 인터페이스가 가능할 것이다. 또한, 람다식과 동일한 메소드가 정의되어 있는 것이어야 한다. 
쉽게 생각해보면, 이렇게 구성된 클래스나 인터페이스만이 참조변수로 익명 객체(람다식)의 메소드를 호출할 수 있기 때문에 당연한 것이다. 

예를 들어 아래와 같이 max()라는 메소드가 정의된 MyFunction 인터페이스가 정의되어 있다고 가정하자.

```java
interface MyFunction{
    int max(int a, int b);
}
```

이 인터페이스를 구현한 익명 클래스의 객체는 다음과 같이 생성할 수 있다.

```java
MyFunction f = new MyFunction(){
    public int max(int a, int b){
        return a > b ? a : b;
    };
}

int bigger = f.max(5, 3);
```
MyFunction 인터페이스에 정의된 메소드 max()는 람다식 `(int a, int b) -> a > b ? a : b`과 메소드의 선언부가 일치한다. 그래서 위 코드의 익명 객체를 람다식으로 아래와 같이 대체할 수 있다. 

```java
MyFunction f = (int a, int b) -> a > b ? a : b;     //익명 객체를 람다식으로 대체
int bigger = f.max(5, 3);                           // 익명 객체의 메소드를 호출
```

이렇게 MyFunction 인터페이스를 구현한 익명 객체를 람다식으로 대체가 가능한 이유는, 람다식도 실제로는 익명 객체이고, Myfunction 인터페이스를 구현한 익명 객체의 메소드 max()와 람다식의 매개변수의 타입과 개수, 반환타입이 일치하기 때문이다.  

위의 예제로 알아본 것처럼, 하나의 메소드가 선언된 인터페이스를 정의해서 람다식으로 다루는 것은 기존의 자바의 규칙들을 어기지 않으면서도 자연스럽다. 그래서 인터페이스를 통해 람다식을 다루기로 결정되었고, 람다식을 다루기 위한 인터페이스를 함수형 인터페이스(functional interface)라고 부르기로 약속했다.

```java
@FunctionalInterface
interface MyFunction{   //함수형 인터페이스 MyFunction
    int madx(int a, int b);
}
```

단, 함수형 인터페이스에는 인터페이스의 메소드와 람다식이 1:1로 연결되도록 오직 하나의 추상 메소드만 정의되어 있어야 한다는 제약이 있다. 그렇기 때문에 static 메소드와 default 메소드는 몇 개가 있건 상관없다. 
함수형 인터페이스 정의 시, @FunctionalInterface 어노테이션을 사용하면 컴파일러가 함수형 인터페이스를 올바르게 정의하였는지 확인해주므로 꼭 붙이도록 하자

기존에는 아래와 같이 인터페이스의 메소드 하나를 구현하는데도 복잡하고 해야 했는데, 

```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

Collections.sort(list, new Comparator<String>(){
    public int compare(String s1, String s2){
        return s2.compareTo(s1);
    }
});
```
아래와 같이 람다식으로 간단하게 처리할 수 있다.

```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

#### 함수형 인터페이스 타입의 매개변수와 반환타입 
앞에서 선언한 MyFunction 인터페이스가 있을 때, 어떤 메소드의 매개변수가 MyFunction 타입이면, 이 메소드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야한다는 뜻이다. 

```java
@FunctionalInterface
interface MyFunction{
    void myMethod();
}

void aMethod(MyFunction f){
    f.myMethod();
}
...

MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);
```

또는 참조변수 없이 직접 람다식을 매개변수로 지정하는 것도 가능하다.
```java
aMethod(()->System.out.println("myMethod()"));
```

그리고 메소드의 반환타입이 함수형 인터페이스라면, 이 함수형 인터페이스의 추강 메소드와 동등한 람다식을 가리키는 참조변수를 반환하거나 람다식을 직접 반환할 수 있다. 
```java
    MyFunction myMethod(){
        MyFunction f = () -> {};
        return f;                   //return () -> {};로 줄일 수 있음
    }
```

람다식을 참조변수로 다룰 수 있다는 것은 메소드를 통해 람다식을 주고받을 수 있다는 것을 의미한다. 즉, 변수처럼 메소드를 주고받는 것이 가능해진 것이다. 사실상 내부적으로는 메소드가 아니라 객체를 주고받는 것이라 근본적으로 달라진 것은 아무것도 없지만, 람다식 덕분에 예전보다 코드가 간결하고 이해하기 쉬워졌다.

```java
@FunctionalInterface
interface MyFunction{
    void run();
}

public class LambdaClass {

    static void execute(MyFunction f) {
        f.run();
    }

    static MyFunction getMyFunction() {
        return () -> System.out.println("f3.run()");
    }

    public static void main(String[] args) {
        MyFunction f1 = () -> System.out.println("f1.run()");
        MyFunction f2 = new MyFunction() {
            @Override
            public void run() {
                System.out.println("f2.run()");
            }
        };
        MyFunction f3 = getMyFunction();

        f1.run();
        f2.run();
        f3.run();

        execute(f1);
        execute(() -> System.out.println("run()"));
    }
}
```
```console
f1.run()
f2.run()
f3.run()
f1.run()
run()
```

#### 람다식의 타입과 형변환
함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다. 람다식은 익명 객체이고 익명 객체는 타입이 없다. 
정확히 말하면 타입은 있지만 컴파일러가 임의로 이름을 정하기 때문에 알 수 없다. 그래서 대입 연산자의 양변의 타입을 일치시키기 위해서는 아래와 같이 형변환이 필요하다.
```java
MyFunction f = (MyFunction) (()->{});       //양변의 타입이 다르므로 형변환이 필요
```

람다식은 MyFunction 인터페이스를 직접 구현하지 않았지만, 이 인터페이스를 구현한 클래스의 객체와 완전히 동일하기 때문에 위와 같은 형변환을 허용한다. 그리고 이 형변환은 생략 가능하다.  

또한, 람다식은 이름이 없을 뿐 분명히 객체인데도, 아래와 같이 Object 타입으로 형변환할 수 없다. 람다식은 오직 함수형 인터페이스로만 형변환이 가능하다. 
```java
Object obj = (Object)(()->{});              //error. 함수형 인터페이스로만 형변환 가능
```
>굳이 Object 타입으로 형변환하려면, 먼저 함수형 인터페이스로 변환한 뒤에 형변환 해야한다. 

예제를 통해 확인해보자.

```java
@FunctionalInterface
interface MyFunction{
    void myMethod();
}

public class LambdaClass {
    public static void main(String[] args) {
        MyFunction f = ()->{ };
        Object obj = (MyFunction)(()->{ });
        String str = ((Object) ((MyFunction) (() -> { }))).toString();

        System.out.println(f);
        System.out.println(obj);
        System.out.println(str);

//        System.out.println(()->{});
        System.out.println((MyFunction)(()->{}));
//        System.out.println((MyFunction)(()->{}).toString());
        System.out.println(((Object) ((MyFunction) (() -> { }))).toString());
    }
}
```
```console
com.example.javastudy.week15.LambdaClass$$Lambda$15/0x0000000800067c40@2acf57e3
com.example.javastudy.week15.LambdaClass$$Lambda$16/0x0000000800066040@506e6d5e
com.example.javastudy.week15.LambdaClass$$Lambda$17/0x0000000800066440@2812cbfa
com.example.javastudy.week15.LambdaClass$$Lambda$18/0x0000000800065840@3796751b
com.example.javastudy.week15.LambdaClass$$Lambda$19/0x0000000800065c40@4411d970
```
실행 결과를 보면, 컴파일러가 람다식의 타입을 어떤 형식으로 만들어내는지 알 수 있다. 
일반적인 익명 객체라면, 객체의 타입이 `외부클래스이름$번호`와 같은 형식으로 타입이 결정되었을 텐데, 람다식의 타입은 `외부클래스이름$$Lambda번호`와 같은 형식으로 되어있다. 

#### 외부 변수를 참조하는 람다식
