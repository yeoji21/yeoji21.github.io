
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

java ServiceLoader
interface만 정의하고 그 것을 구현한 jar 파일을 갈아끼우면서? 그 구현체를 사용할 수 있는 것?

-----------------------------------------------------------

#### 지네릭스(Generics)란?
지네릭스는 다양한 타입의 객체들을 다루는 메소드나 컬렉션 클래스에 컴파일 시의 타입 체크(compile-time type check)를 해주는 기능이다.  

객체의 타입을 컴파일 시에 체크하기 때문에 객체의 타입 안정성을 높이고 형변환의 번거로움을 줄여준다. 타입의 안정성을 높인다는 것은 의도하지 않은 타입의 객체가 저장되는 것을 막고, 잘못된 형변환으로 발생할 수 있는 오류를 줄여준다는 뜻이다.  

예를 들어, ArrayList와 같은 컬렉션 클래스는 다양한 종류의 객체를 담을 수 있긴 하지만, 보통 한 종류의 객체를 담는 경우가 더 많다. 그런데도 꺼낼 때마다 타입체크를 하고 형변환을 하는 것은 너무 불편할 것이다. 게다가 원치않는 종류의 객체가 포함되는 것을 막을 방법이 없다는 것도 문제인데, 이러한 문제들을 지네릭스가 해결해준다. 

#### 지네릭 클래스의 선언
지네릭 타입은 클래스와 메소드에 선언할 수 있는데, 먼저 클래스에 선언하는 지네릭 타입에 대해 알아보자. 
```java
class Box{
    Object item;
    public void setItem(Object item) { this.item = item; }
    public Object getItem() { return item; }
}
```
예를 들어 이러한 Box 클래스가 있을 때, 이 클래스를 지네릭 클래스로 변경하면 다음과 같이 클래스 옆에 <T>를 붙이면 된다. 그리고 Object를 모두 T로 바꾼다. 
```java
class Box<T>{
    T item;
    public void setItem(T item) { this.item = item; }
    public T getItem() { return item; }
}
```

Box<T>에서 T를 타입 변수(type variable)라고 하며, Type의 첫 글자에서 따온 것이다. 타입 변수는 T가 아닌 다른 것을 사용해도 된다. 타입 변수가 여러 개인 경우 Map<K,V>와 같이 콤마를 구분자로 나열하면 된다. 여기서 K와 V는 Key와 Value를 의미하는데, 무조건 T를 사용하기 보다는 이처럼 상황에 맞게 의미있는 문자를 선택해서 사용하는 것이 좋다. 

이들은 기호의 종류만 다를 뿐 임의의 참조형 타입을 의미한다는 것은 모두 같다. 기존에는 다양한 종류의 타입을 다루는 메소드의 매개변수나 리턴타입으로 Object타입의 참조변수를 많이 사용했고, 그로 인해 형변환이 불가피했지만, 지네릭스를 사용하면 Object타입 대신 원하는 타입을 지정하기만 하면 된다.  

이제 지네릭스 클래스가 된 Box 클래스의 객체를 생성할 때는 다음과 같이 참조변수와 생성자에 타입 T대신에 사용될 실제 타입을 지정해주어야 한다.  

```java
Box<String> b = new Box<String>();
b.setItem(new Object());    //error, String 이외의 타입을 지정하면 에러발생
b.setItem("ABC");
```

위 예제에서 T대신 String타입을 지정했으므로 지네릭 클래스 Box<T>는 다음과 같이 정의된 것과 같다. 

```java
class Box<String>{
    String item;
    public void setItem(String item) { this.item = item; }
    public String getItem() { return item; }
}
```

#### 지네릭스의 용어

```
Box<T> : 지네릭 클래스, T의 Box 또는 T Box라고 읽는다.
T : 타입 변수 또는 타입 매개변수 (T는 타입 문자)
Box : 원시 타입(raw type)
```
아래와 같이 타입 매개변수에 타입을 지정하는 것을 '지네릭 타입 호출'이라 하고, 지정된 타입 String을 매개변수화된 타입(parameterized type)이라고 한다. 
```java
Box<String> box = new Box<String>();
```
예를 들어, Box<String>과 Box<Integet>는 지네릭 클래스 Box<T>에 서로 다른 타입을 대입하여 호출한 것일 뿐, 이 둘이 별개의 클래스를 의미하는 것은 아니다. 컴파일 후에 Box<String>과 Box<Integet>는 이들의 원시타입인 Box로 바뀐다. 즉, 지네릭 타입이 제거되는데 이 것에 대해서는 잠시 후에 자세히 알아보자. 

#### 지네릭스의 제한
지네릭 클래스에서 static 멤버에 타입 변수 T를 사용할 수 없다. T는 인스턴스 변수로 간주되기 때문이다. 이미 알고 있는 것처럼 static은 인스턴스 변수를 참조할 수 없다. static 멤버는 타입 변수에 지정된 타입의 종류와 관계없이 동일한 것이어야 하기 때문이다. 

그리고 지네릭 타입의 배열을 생성하는 것도 허용되지 않는다. 지네릭 배열의 참조변수를 선언하는 것은 가능하지만, new T[10]과 같이 배열을 생성하는 것은 안된다는 의미이다.
```java
class Box<T>{
    T[] itemArr;        // OK. T 타입의 배열을 위한 참조변수
    ...
    T[] toArray(){
        T[] tempArr = new T[itemArr.length];        //error. 지네릭 배열 생성 불가
        ...
    }
}
```

지네릭 배열을 생성할 수 없는 것은 new 연산자 때문인데, 이 연산자는 컴파일 시점에 타입 T가 뭔지 정확히 알아야 한다. 그런데 위의 코드에 정의된 Box<T>클래스를 컴파일하는 시점에서는 T가 어떤 타입이 될지 알 수 없다. 따라서 instanceof 연산자도 같은 이유로 T를 피연산자로 사용할 수 없다.

꼭 지네릭 배열을 생성해야할 필요가 있을 대는 new 연산자 대신 Reflection을 사용해 newInstance()와 같이 동적으로 객체를 생성하는 메소드를 통해 배열을 생성하거나 Object 배열을 생성해서 복사한 다음 T[]로 형변환하는 방법 등을 사용해야 한다.

#### 제한된 지네릭 클래스
타입 문자로 사용할 타입을 명시하면 한 종류의 타입만 저장할 수 있도록 제한할 수 있지만, 그래도 여전히 모든 종류의 타입을 지정할 수 있다는 것에는 변함이 없다. 예를 들어 아래 예제 같은 경우, 과일상자 클래스에 타입 매개변수로 장난감 클래스를 지정할 수 있다. 
```java
FruitBox<Toy> fruitBox = new FruitBox<>();
fruitBox.add(new Toy());
```

이런 경우, 지네릭 타입에 extends 키워드를 사용하면 특정 타입의 자손들만 대입할 수 있게 제한할 수 있다. 

```java
class FruitBox<T extends Fruit>{
    ArrayList<T> list = new ArrayList<T>();
    ...
}
```
여전히 한 종류의 타입만 담을 수 있지만, Fruit 클래스의 자손들만 담을 수 있다는 제한이 더 추가된 것이다. FruitBox클래스의 add()의 매개변수 타입 T도 Fruit와 그 자손 타입이 될 수 있으므로, 아래와 같이 여러 과일을 담을 수 있는 상자가 가능하게 된다. 
```java
FruitBox<Fruiut> fruitBox = new FruitBox<>();
fruitBox.add(new Apple());
fruitBox.add(new Grape());
```
다형성에서 부모타입의 참조변수로 자식타입의 객체를 가리킬 수 있는 것처럼, 매개변수화된 타입의 자손도 가능한 것이다. 만약 타입 매개변수 T에 Object를 대입하면, 모든 종류의 객체를 저장할 수 있을 것이다. 
>만일 클래스가 아니라 인터페이스를 구현해야 한다는 제약이 필요한 경우에도 implements가 아닌 extends 키워드를 사용해야 한다는 점을 주의하자

만약 Fruit 클래스의 자식이면서 Eatable 인터페이스도 구현해야 한다면 아래와 같이 & 기호로 연결한다.
```java
class FruitBox<T extends Fruit & Eatable>
```

#### 와일드 카드
앞선 예제들과 연결해 매개변수에 과일박스를 대입하면 주스를 만들어서 반환하는 Juicer라는 클래스가 있고, 이 클래스에는 과일을 주스로 만들어 반환하는 makeJuice()라는 static 메소드가 아래와 같이 정의되어 있다고 가정하자. 
```java
class Juicer{
    static Juice makeJuice(FruitBox<Fruit> box){
        String temp="";
        for(Fruit f : box.getList())
            temp += f + " ";
        return new Juice(temp);
    }
}
```
Juicer 클래스는 지네릭 클래스가 아닌데다, 지네릭 클래스라고 해도 static 메소드에는 타입 매개변수 T를 매개변수에 사용할 수 없으므로 아예 지네릭스를 사용하지 않거나, 예제와 같이 타입매개 변수 대신 특정 타입을 지정해주어야 한다.

그런데 이렇게 지네릭 타입을 FruitBox<Fruit>로 고정해놓으면, FruitBox<Apple>타입의 객체는 makeJuice()의 매개변수가 될 수 없다. 
```java
FruitBox<Fruit> fruitBox = FruitBox<>();
FruitBox<Apple> appleBox = FruitBox<>();
...
System.out.println(Juicer.makeJuice(fruitBox));     //ok
System.out.println(Juicer.makeJuice(appleBox));     //error
```

따라서 이 것을 해결하기 위해 아래와 같이 FruitBox<Apple>타입을 매개변수로 갖는 makeJuice()를 추가할 수 밖에 없다.
```java
static Juice makeJuice(FruitBox<Apple> box){
        String temp="";
        for(Fruit f : box.getList())
            temp += f + " ";
        return new Juice(temp);
    }
```

하지만 이렇게 오버로딩하면 컴파일 에러가 발생한다. 지네릭 타입이 다른 것만으로는 오버로딩이 성립하지 않기 때문이다. 지네릭 타입은 컴파일러가 컴파일할 때만 사용하고 제거해버린다. 따라서 위의 두 메소드는 오버로딩 되지않고 메소드 중복정의로 판단된다. 

이럴 때 사용하기 위해 고안된 것이 바로 와일드 카드이다. 와일드 카드는 기호 ?로 표현하는데, 와일드 카드는 어떠한 타입도 될 수 있다. 그렇기에 ? 만으로는 Object타입과 다를게 없으므로 다음과 같이 extends와 super로 상한(upper bound)과 하한(lower bound)을 제한할 수 있다.
```
<? extends T>       와일드 카드의 상한 제한. T와 그 자손들만 가능
<? super T>         와일드 카드의 하한 제한. T와 그 조상들만 가능
<?>                 제한 없음. 모든 타입이 가능
```
> 와일드 카드에는 지네릭 클래스와 달리 &를 사용할 수 없다. 

와일드 카드를 이용해서 makeJuice()의 매개변수 타입을 FruitBox<? extends Fruit>로 변경하면 이 메소드로 FruitBox<Fruit>와 FruitBox<Apple>, FruitBox<Grape> 등도 문제없이 사용이 가능해진다. 

#### 지네릭 메소드
메소드의 선언부에 지네릭 타입이 선언된 메소드를  지네릭 메소드라고 하면, 지네릭 타입의 선언 위치는 반환타입 바호 앞이다. 
```java
static <T> void sort(List<T> list, Comparator<? super T> c)
```

지네릭 클래스에 정의된 타입 매개변수와 지네릭 메소드에 정의된 타입 매개변수는 별개의 것이다. 같은 타입의 문자 T를 사용해도 문자만 같을 뿐 서로 다른 것이므로 주의해야 한다.
> 지네릭 메소드는 지네릭 클래스가 아닌 클래스에도 정의될 수 있다. 

그리고 위의 sort()의 정의를 보면 static 메소드인 것을 알 수 있는데, 앞서 설명한 것처럼 static 멤버에는 타입 매개변수를 사용할 수 어뵤지만, 메소드에 지네릭 타입을 선언하고 사 용하는 것은 가능하다. 메소드에 선언된 지네릭 타입은 지역변수를 선언한 것과 같다고 생각하면 이해하기 쉬운데, 이 타입의 매개변수는 메소드 내에서만 지역적으로 사용될 것이므로 메소드가 static이건 아니건 상관이 없는 것이다.  

앞서 나왔던 makeJuice()를 지네릭 메소드로 바꾸면 다음과 같다. 
```java
static Juice makeJuice(FruitBox<? extends Fruit> box){
    String temp="";
    for(Fruit f : box.getList())
        temp += f + " ";
    return new Juice(temp);
}
```
⬇️
```java
static <T extends Fruit> Juice makeJuice(FruitBox<T> box){
    String temp="";
    for(Fruit f : box.getList())
        temp += f + " ";
    return new Juice(temp);
}
```
이제 이 메소드를 호출할 때는 아래와 같이 타입 변수에 타입을 대입해야 한다.
```java
FruitBox<Fruit> fruitBox = FruitBox<>();
FruitBox<Apple> appleBox = FruitBox<>();
...
System.out.println(Juicer.<Fruit>makeJuice(fruitBox));
System.out.println(Juicer.<Apple>makeJuice(appleBox)); 
```
하지만 대부분의 경우 컴파일러가 타입을 추정할 수 있기 때문에 생략해도 된다. 위의 코드에서도 fruitBox와 appleBox의 선언부를 통해 대입된 타입을 컴파일러가 추정할 수 있다.
```java
System.out.println(Juicer.makeJuice(fruitBox));
System.out.println(Juicer.makeJuice(appleBox)); 
```

#### 지네릭 타입의 형변환
지네릭 타입과 원시타입(raw type)간의 형변환이 가능할까?
```java
Box box = null;
Box<Object> objBox = null;

box = (Box)objBox;          //ok. 지네릭타입 -> 원시타입 경고 발생
objBox = (Box<Object>)box;  //ok. 원시타입 -> 지네릭타입 경고 발생
```
이렇게 지네릭 타입과 넌지네릭 타입간의 형변환은 경고가 발생하긴 하지만 항상 가능하다. 그렇다면 대입된 타입이 다른 지네릭 타입 간에는 형변환이 가능할까?

```java
Box<Object> objBox = null;
Box<String> strBox = null;

objBox = (Box<Object>)strBox;   //error
strBox = (Box<String>)objBox;   //error
```
이건 불가능하다. 대입된 타입이 Object라도 불가능한건 마찬가지이다. 이 사실은 이미 배웠는데, 아래의 문장이 안된다는 것이 바로 Box<String>이 Box<Object>로 형변환될 수 없다는 사실을 간접적으로 알려주는 것이기 때문이다.
```java
Box<Object> objBox = new Box<String>();     //error
```
그렇다면 아래의 문장은 어떨까?

```java
Box<? extends Object> objBox = new Box<String>();
```
이건 형변환이 된다. 그래서 전에 배운 makeJuice()의 매개변수에 다형성이 적용될 수 있었던 것이다. 

```java
static Juice makeJuice(FruitBox<? extends Fruit> box)
...
```

이번엔 좀 더 실질적인 예제를 살펴보자. 다음은 java.util.Optional 클래스의 일부분이다. 
```java
public final class Optional<T>{
    private static final Optional<?> EMPTY = new Optional<>();
    private final T value;
    ...
    public static<T> Optional<T> empty(){
        Optional<T> t = (Optional<T>)EMPTY;
        return t;
    }
}
```

static 상수 EMPTY에 비어있는 Optional객체를 생성해서 저장했다가 empty()를 호출하면 EMPTY를 형변환해서 반환한다. 먼저 상수를 선언하는 문장을 단계별로 분석해보면 다음과 같다. 
```java
    Optional<?> EMPTY = new Optional<>();
->  Optional<? extends Object> EMPTY = new Optional<>();
->  Optional<? extends Object> EMPTY = new Optional<Object>();
```
<?>는 <? extends Object>를 축약한 것이며, <> 안에 생략된 타입인 ?가 아니라 Object이다. 
위의 문장에서 EMPTY의 타입을 Optional<Object>가 아닌 Optional<?>로 한 이유는 Optional<T>로 형변환이 가능하기 때문이다. 
empty()의 반환타입이 Optional<T>이므로 EMPTY를 Optional<T>로 형변환해야 하는데, 위의 코드에서 알 수 있는 것처럼 Optional<Object>는 Optional<T>로 형변환이 불가능하다.

정리해보자면, Optional<Object>를 Optional<String>으로 직접 형변환하는 것은 불가능하지만, 와일드 카드가 포함된 지네릭 타입으로 형변환하면 가능하다. 대신 확인되지 않은 타입으로의 형변환이라는 경고가 발생한다. 


#### 지네릭 타입의 제거 Erasure
컴파일러는 지네릭 타입을 이용하여 소스파일을 체크하고, 필요한 곳에 형변환을 넣어준다. 그리고 지네릭 타입을 제거한다. 따라서 컴파일된 파일(.class)에는 지네릭 타입에 대한 정보가 없다. 

이렇게 하는 이유는 지네릭이 도입되기 이전의 소스 코드와의 호환성을 유지하기 위해서이다. JDK 1.5부터 지네릭스가 도입되었지만, 아직도 원시타입을 사용해서 코드를 작성하는 것을 허용한다. 

지네릭타입의 기본적인 제거 과정에 대해서 알아보자.

1. 지네릭 타입의 경계(bound)를 제거한다.   
    지네릭 타입이 <T extends Fruit>라면 T는 Fruit로 치환된다. <T>인 경우에 T는 Object로 치환된다. 그리고 클래스 옆의 선언은 제거된다. 
2. 지네릭 타입을 제거한 후에 타입이 일치하지 않으면 형변환을 추가한다.  
    List의 get()은 Object타입을 반환하므로 형변환이 필요하다.


-----------------------------------------------------------
Apple class
```java
@Getter
public class Apple {
    private Integer id;

    public static Apple of(Integer id) {
        Apple apple = new Apple();
        apple.id = id;
        return apple;
    }
}
```

Banana class
```java
@Getter
public class Banana {
    private Integer id;

    public static Banana of(Integer id) {
        Banana banana = new Banana();
        banana.id = id;
        return banana;
    }
}
```

AppleDao
```java
public class AppleDao {
    private Map<Integer, Apple> source = new HashMap<>();

    public void save(Apple apple) {
        source.put(apple.getId(), apple);
    }

    public void delete(Apple apple) {
        source.remove(apple.getId());
    }

    public void delete(Integer id) {
        source.remove(id);
    }
    
    public List<Apple> findAll(){
        return new ArrayList<>(source.values());
    }

    public Apple findById(Integer id) {
        return source.get(id);
    }
}
```

BananaDao
```java
public class BananaDao {
    private Map<Integer, Banana> source = new HashMap<>();

    public void save(Banana banana) {
        source.put(banana.getId(), banana);
    }

    public void delete(Banana banana) {
        source.remove(banana.getId());
    }

    public void delete(Integer id) {
        source.remove(id);
    }
    
    public List<Banana> findAll(){
        return new ArrayList<>(source.values());
    }

    public Banana findById(Integer id) {
        return source.get(id);
    }
}
```

```java
public class Store {
    public static void main(String[] args) {
        AppleDao appleDao = new AppleDao();
        appleDao.save(Apple.of(1));
        appleDao.save(Apple.of(2));

        List<Apple> all = appleDao.findAll();
        all.forEach(System.out::println);
    }
}
```
```console
com.example.javastudy.week14.Apple@2d6e8792
com.example.javastudy.week14.Apple@2812cbfa
```

⬇️

```java
public interface Entity<K> {
    K getId();
}
```

```java
@Getter
public class Apple implements Entity<Integer>{
    private Integer id;

    public static Apple of(Integer id) {
        Apple apple = new Apple();
        apple.id = id;
        return apple;
    }
}
```

```java
@Getter
public class Banana implements Entity<Integer>{
    private Integer id;

    public static Banana of(Integer id) {
        Banana banana = new Banana();
        banana.id = id;
        return banana;
    }
}
```

```java
public class GenericDao<K,E extends Entity<K>> {
    private Map<K, E> source = new HashMap<>();

    public void save(E e) {
        source.put(e.getId(), e);
    }

    public void delete(E e) {
        source.remove(e.getId());
    }

    public void delete(K id) {
        source.remove(id);
    }

    public List<E> findAll(){
        return new ArrayList<>(source.values());
    }

    public E findById(K id) {
        return source.get(id);
    }
}
```

```java
public class AppleDao extends GenericDao<Integer, Apple> {}
```

```java
public class BananaDao extends GenericDao<Integer, Banana>{}
```

```java
public static void main(String[] args) {
    AppleDao appleDao = new AppleDao();
    appleDao.save(Apple.of(1));
    appleDao.save(Apple.of(2));

    List<Apple> all = appleDao.findAll();
    all.forEach(System.out::println);

    BananaDao bananaDao = new BananaDao();
    bananaDao.save(Banana.of(3));
    bananaDao.save(Banana.of(4));

    List<Banana> bananas = bananaDao.findAll();
    for (Banana banana : bananas) {
        System.out.println(banana);
    }
}
```

```console
com.example.javastudy.week14.Apple@2acf57e3
com.example.javastudy.week14.Apple@506e6d5e
com.example.javastudy.week14.Banana@67b64c45
com.example.javastudy.week14.Banana@4411d970
```

아예 AppleDao와 BananaDao 클래스를 없애도 된다.

```java
public static void main(String[] args) {
    GenericDao<Integer, Apple> appleDao = new GenericDao<>();
    appleDao.save(Apple.of(1));
    appleDao.save(Apple.of(2));

    List<Apple> all = appleDao.findAll();
    all.forEach(System.out::println);
}
```
마치 List를 생성할 때 새로 클래스를 만들지 않고 List<Apple>로 사용하는 것과 같은 이치이다.
그럼에도 불구하고 따로 DAO 클래스를 만드는 이유는 해당 클래스에 특화된 메소드를 추가할 수도 있기 때문이다. 


질문! 
런타임 중에 제네릭의 클래스를 알아낼 수 있을까?
답은 있다. 

```java
private Class<E> entityClass;

public GenericDao(Class<E> entityClass) {
    this.entityClass = entityClass;
}
```
이렇게 생성자로 클래스를 넘겨서 유지하게 하는 방법이 있으나 이 방법은 매번 생성자에 클래스를 넘겨줘야 하므로 비효율적이다. 
이렇게 하지않고도 방법이 있는데, 
지네릭이 컴파일 타임에 지워진다고 해도, 메타데이터에 남아있기 때문에 알아낼 수 있다.

```java
private Class<E> entityClass;
    
public GenericDao() {
    this.entityClass = (Class<E>)((ParameterizedType)this.getClass().getGenericSuperclass()).getActualTypeArguments()[1];
}
```

```java
BananaDao bananaDao = new BananaDao();
System.out.println(bananaDao.getEntityClass());
```
```console
class com.example.javastudy.week14.Banana
```

```java
GenericDao<Integer, Apple> appleDao = new GenericDao<>();
System.out.println(appleDao.getEntityClass());
```
```console
Exception in thread "main" java.lang.ClassCastException: class java.lang.Class cannot be cast to class java.lang.reflect.ParameterizedType (java.lang.Class and java.lang.reflect.ParameterizedType are in module java.base of loader 'bootstrap')
	at com.example.javastudy.week14.GenericDao.<init>(GenericDao.java:15)
	at com.example.javastudy.week14.Store.main(Store.java:7)
```
대신 이렇게 하면 에러 발생


```java

```