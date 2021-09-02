---
title: "[JAVA] 컬렉션 프레임워크 (Collections Framework)"
author: "yeoji21"
date: '2021-09-03 03:55:00 +0900'
categories: [Backend, Java]
tags: [java, 자바]
---

## 목차
- 컬렉션 프레임워크란?
- 컬렉션 프레임워크의 핵심 인터페이스
- ArrayList
- LinkedList
- Stack과 Queue
- HashSet
- TreeSet
- HashMap과 Hashtable
- TreeMap
- 그 외




### **컬렉션 프레임워크란?**

컬렉션 프레임워크란, 데이터 군을 저장하는 클래스들을 표준화한 설계를 뜻한다. 여기서 `컬렉션(Collection)`은 다수의 데이터, 데이터 그룹을 의미하고 `프레임워크(Framework)`는 표준화된 프로그래밍 방식을 말한다.

`JDK 1.2` 이전까지는 `Vector`, `Hashtable`, `Properties`와 같은 컬렉션 클래스들을 각자의 방식으로 처리해야 했으나, `JDK 1.2`부터 컬렉션 프레임워크가 등장하면서 다양한 종류의 컬렉션 클래스가 추가되고 모든 컬렉션 클래스를 표준화된 방식으로 다룰 수 있도록 체계화되었다.  

### **컬렉션 프레임워크의 핵심 인터페이스**

컬렉션 프레임워크에서는 컬렉션 데이터 그룹을 크게 세 가지 타입의 인터페이스로 나누었다. 그리고 인터페이스 `List`와 `Set`의 공통 부분을 뽑아서 새로운 인터페이스인 `Collection`을 추가로 정의했다. `Map` 인터페이스는 이들과는 전혀 다른 형태로 컬렉션을 다루기 때문에 같은 상속 계층도에 포함되지 못했다.

<img src="assets/../../../assets/img/collections/1.jpeg" width=500>

>JDK 1.5부터 Iterable 인터페이스가 추가되고 Collection 인터페이스가 이것을 상속받도록 변경되었으나. 이것은 단지 공통적인 메소드를 뽑아서 중복을 제거하기 위한 것이므로 상속 계층도에서 별 의미가 없다.

|인터페이스|특징|구현 클래스|
|--|--:|--:|
|List|순서가 있는 데이터의 집합, 데이터의 중복 허용 <br> 예) 대기자 명단| ArrayList, LinkedList, Stack, Vactor|
|Set|순서를 유지하지 않고 데이터의 중복이 허용되지 않음 <br> 예) 양의 정수집합, 소수의 집합|HashSet, TreeSet|
|Map|key와 value의 쌍으로 이루어진 데이터의 집합<br>순서유지 x, key는 중복허용 x, value는 중복허용 o<br>예) 우편번호, 전화번호|HashMap,TreeMap,Hashtable,Properties|

컬렉션 프레임워크의 모든 컬렉션 클래스들은 이 셋 중의 하나를 구현하고 있으며, 구현한 인터페이스의 이름이 클래스의 이름에 포함되어 있어서 이름만으로도 클래스의 특징을 쉽게 알 수 있다. 

하지만 `Vector`, `Stack`, `Hashtable`, `Properties`와 같이 컬렉션 프레임워크가 만들어지기 이전부터 존재하던 클래스들은 이러한 명명 규칙을 따르지 않는다. 이런 기존의 컬렉션 클래스들은 호환을 위해 남겨두었지만 가능한 사용하지 않는 것이 좋다.

#### Collection 인터페이스 

|메소드|설명|
|--:|--:|
|boolean **add(Object o)**<br>boolean **addAll(Collection c)**|지정된 객체(o) 또는 Collection의 객체들을 Collection에 추가|
|void **clear()**|Collection의 모든 객체를 삭제|
|boolean **contains(Object o)**<br>boolean **containsAll(Collection c)**|지정된 객체 또는 Collection의 객체들이 Collection에 포함되어 있는지 확인|
|boolean **equals(Object o)**|동일한 Collection인지 비교|
|int **hashCode()**|Collection의 hash code 반환|
|boolean **isEmpty()**|Collection이 비어있는지 확인|
|Iterator **iterator()**|Collection의 iterator를 얻어서 반환|
|boolean **remove(Object o)**|지정된 객체를 삭제|
|boolean **removeAll(Collection c)**|지정된 Collection에 포함된 모든 객체를 삭제|
|boolean **retainAll(Collection c)**|지정된 Collection에 포함된 객체만 남기고 다른 나머지 객체들은 삭제 <br> 이로인해 Collection에 변화가 생기면 true를 반환|
|int **size()**|Collection에 저장된 객체의 개수를 반환|
|Object[] **toArray()**|Collection에 저장된 객체를 객체 배열로 반환|
|Object[] **toArray(Object[] a)**|지정된 배열에 Collection의 객체를 저장해서 반환|

이 외에도 `JDK 1.8`부터 추가된 `Lambda`와 `Stream`에 관련된 메소드들이 더 있는데, 이 것들은 해당 주제를 다루는 포스트에서 따로 언급하겠다.
> 실제 Java API 문서를 보면 표에서 사용된 Object가 아닌 제네릭 타입 E로 표기되어 있는데, 이해를 돕기 위해 Object로 표기했다.

#### List 인터페이스

`List` 인터페이스는 **중복을 허용**하면서 **저장순서가 유지**되는 컬렉션을 구현하는데 사용된다. 

<img src="assets/../../../assets/img/collections/2.jpeg" width=400>

|메소드|설명|
|--:|--:|
|void **add(int index, Object element)**<br>boolean **addAll(int index, Collection c)**|지정된 위치에 객체 또는 컬렉션에 포함된 객체들을 추가|
|Object **get(int index)**|지정된 위치에 있는 객체를 반환|
|int **indexOf(Object o)**|지정된 객체의 위치를 반환<br>(List의 첫번째 요소부터 순방향으로 탐색)|
|int **lastIndexOf(Object o)**|지정된 객체의 위치를 반환<br>(List의 마지막 요소부터 역방향으로 탐색)|
|ListIterator **listIterator()**<br>ListIterator **listIterator(int index)**|List의 객체에 접근할 수 있는 ListIterator 반환|
|Object **remove(int idex)**|지정된 위치에 있는 객체를 삭제하고 삭제된 객체를 반환|
|Object **set(int index, Object element**)|지정된 위치에 객체를 저장|
|void **sort(Comparator c)**|지정된 비교자로 List를 정렬|
|List **subList(int fromIndex, int toIndex)**|지정된 범위에 있는 객체를 반환|

#### Set 인터페이스
`Set` 인터페이스는 **중복을 허용하지 않고 저장순서가 유지되지 않는** 컬렉션 클래스를 구현하는데 사용된다. 

<img src="atssets/../../../assets/img/collections/3.jpeg" width=300>

#### Map 인터페이스
`Map`인터페이스는 key와 value를 하나의 쌍으로 묶어서 저장하는 컬렉션 클래스를 구현하는 데 사용된다. 이 때 key는 중복될 수 없지만 value는 중복을 허용한다. 만약 기존에 저장된 데이터와 중복된 key로 데이터를 저장하면 기존의 value는 없어지고 덮어써진다. 

<img src="asserts/../../../assets/img/collections/4.jpeg" width=450>

|메소드|설명|
|--:|--:|
|void **clear()**|Map의 모든 객체를 삭제|
|boolean **containsKey(Object key)**|지정된 key와 일치하는 Map의 key가 있는지 확인|
|boolean **containsValue(Object value)**|지정된 value와 일치하는 Map의 value가 있는지 확인|
|Set **entrySet()**|Map의 key-value쌍을 Map.Entry타입의 객체로 저장한 Set으로 반환|
|boolean **equals(Object o)**|동일한 Map인지 비교|
|Object **get(Object key)**|지정한 key에 대응하는 value를 반환|
|int **hashCode()**|해시코드를 반환|
|boolean **isEmpty()**|Map이 비어있는지 확인|
|Set **keySet()**|Map에 저장된 모든 key를 반환|
|Object **put(Object key, Object value)**|Map에 key-value쌍을 저장|
|void **putAll(Map t)**|지정된 Map의 모든 key-value쌍을 추가|
|Object **remove(Object key)**|지정된 key와 일치하는 key-value쌍을 삭제|
|int **size()**|Map에 저장된 key-value쌍의 개수를 반환|
|Collection **values()**|Map에 저장된 모든 value객체를 반환|

여기서 `values()`의 반환타입은 `Collection`이고, `keySet()`의 반환타입은 `Set`인 것에 주목하자. Map 인터페이스에서 값은 중복을 허용하기 때문에 Collection타입으로 반환하고, 키는 중복을 허용하지 않기 때문에 Set타입으로 반환한다. 

#### Map.Entry 인터페이스
`Map.Entry` 인터페이스는 `Map`인터페이스의 내부 인터페이스다. Map에 저장되는 key-value 쌍을 다루기 위해 내부적으로 `Entry` 인터페이스를 정의해 놓았다. 
> 내부 클래스처럼 인터페이스도 인터페이스 안에 인터페이스를 정의하는 내부 인터페이스를 정의할 수 있다. 

|메소드|설명|
|--:|--:|
|boolean **equals(Object o)**|동일한 Entry인지 비교|
|Object **getKey()**|Entry의 key객체를 반환|
|Object **getValue()**|Entry의 value객체를 반환|
|int **hashCode()**|Entry의 해시코드를 반환|
|Object **setValue(Object value)**|Entry의 value객체를 지정된 객체로 변경|

### **ArrayList**
`ArrayList`는 컬렉션 프레임워크 중에서 가장 많이 사용되는 컬렉션 클래스다. 이름에서 알 수 있듯이 `List` 인터페이스를 구현하기 때문에 데이터의 저장순서가 유지되고 중복을 허용한다는 특징을 갖는다.  

ArrayList는 기존의 `Vector`를 개선한 것으로, 구현원리와 기능적인 측면에서 동일하다고 할 수 있다. 앞서 얘기했듯이 Vector는 legacy code와의 호환성을 위해 계속 남겨두고 있을 뿐, 가능한 ArrayList를 사용해야 한다.

ArrayList는 내부적으로 `Object 배열`을 이용해서 데이터를 순차적으로 저장한다. 
```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, 
    RandomAccess, Cloneable, java.io.Serializable{
        ...
transient Object[] elementData;
        ...
}
```

따라서 `ArrayList`나 `Vector` 같이 배열을 이용한 자료구조는 데이터를 읽어오고 저장하는 데는 효율이 좋지만, 용량을 변경해야 할 때는 새로운 배열을 생성한 후 기존으 배열로부터 새로 생성된 배열로 데이터를 복사해야하기 때문에 상당히 효율이 떨어진다는 단점을 가진다. 

그래서 처음에 인스턴스를 생성할 때, 저장할 데이터의 개수를 잘 고려하여 충분한 용량의 인스턴스를 생성하는 것이 좋다.

|생성자|설명|
|--:|--:|
|ArrayList()|크기가 10인 ArrayList를 생성|
|ArrayList(Collection c)|주어진 Collection이 저장된 ArrayList 생성|
|ArrayList(int initCapacity)|지정된 초기용량을 갖는 ArrayList 생성|

생성할 때 지정한 크기보다 더 많은 객체를 저장하면 자동적으로 크기가 늘어나기는 하지만 이 과정에서 처리 시간이 많이 소요된다. 

<img src="assets/../../../assets/img/collections/5.jpeg" width=700>

`ArrayList`의 `Object remove(int index)`메소드는 지정된 위치에 있는 객체를 삭제하고 삭제한 객체를 반환하는데, 삭제할 객체의 바로 아래에 있는 데이터를 한 칸씩 위로 복사해서 삭제할 객체를 덮어쓰는 방식으로 처리한다. 만일 삭제할 객체가 마지막 데이터라면, 단순히 `null`로 변경해주기만 하면 된다.

### **LinkedList**
배열은 간단하며 사용하기 쉽고, 데이터를 읽어오는데 걸리는 시간(access time)이 가장 빠르다는 장점을 가지고 있지만 다음과 같은 단점도 있다. 

1. 크기를 변경할 수 없다.  
    크기를 변경할 수 없어 새로운 배열을 생성해 기존 데이터를 복사해야 한다. 이를 막기 위해서 충분히 큰 크기의 배열을 생성해야 하므로 메모리가 낭비된다. 
2. 비순차적인 데이터의 추가 또는 삭제에 시간이 많이 걸린다.  
    차례대로 데이터를 추가하고 마지막 데이터를 삭제하는 것은 빠르지만, 배열의 중간에 데이터를 추가하거나 삭제하면 다른 데이터들이 이동해야 한다.

이러한 배열의 단점을 보완하기 위해서 `LinkedList`라는 자료구조가 고안되었다. LinkedList는 불연속적으로 존재하는 데이터를 서로 연결(link)한 형태로 구성되어 있다. 
>실제 LinkedList 클래스는 접근성을 높이기 위해 이름과 달리 linked list가 아닌 double linked list로 구현되어 있다. 

하지만 `LinkedList`는 원하는 값을 얻으려면 처음부터 n번째 데이터까지 차례대로 따라가야 한다는 단점이 있다. 그래서 저장해야하는 데이터의 개수가 많아질수록 데이터를 읽어오는 시간이 길어진다.

따라서 데이터의 개수가 변하지 않거나 순차적으로 추가/삭제하는 경우에는 `ArrayList`가 `LinkedList`보다 좋은 선택이고, 데이터 개수의 변경이 잦거나 중간 데이터를 추가/삭제하는 경우에는 `LinkedList`가 `ArrayList`보다 유리하다.

### **Stack과 Queue**

**stack 클래스의 메소드**

|메소드|설명|
|--:|--:|
|boolean **empty()**|Stack이 비었는지|
|Object **peek()**|Stack의 맨 위에 저장된 객체를 반환 (꺼내지는 않음)|
|Object **pop()**|Stack의 맨 위에 저장된 객체를 꺼냄|
|Object **push(Object item)**|Stack에 객체를 저장|
|int **search(Object o)**|주어진 객체를 찾아서 위치를 반환, 없으면 -1 <br> (배열과 달리 위치는 0이아닌 1부터 시작)|

<br>

**Queue 인터페이스의 메소드**

|메소드|설명|
|--:|--:|
|boolean **add(Object o)**|지정된 객체를 Queue에 추가<br>성공하면 true, 저장돈간이 부족하면 IllegalStateException 발생|
|Object **remove()**|Queue에서 객체를 꺼내 반환 <br> 비어있으면 NoSuchElementException 발생|
|Object **element()**|삭제없이 요소를 읽어옴 <br> 비어있으면 NoSuchElementException 발생|
|boolean **offer(Object o)**|Queue에 객체를 저장 <br> 성공하면 true, 실패하면 false 리턴|
|Object **poll()**|Queue에서 객체를 꺼내서 반환. 비어있으면 null|
|Object **peek()**|삭제없이 요소를 읽어옴. 비어있으면 null|


```java
public static void main(String[] args) {
    Stack<Integer> s = new Stack<>();
    Queue<Integer> q = new LinkedList<>();

    s.push(0);
    s.push(1);
    s.push(2);

    q.offer(0);
    q.offer(1);
    q.offer(2);

    System.out.println("---stack---");
    while (!s.isEmpty()) {
        System.out.println(s.pop());
    }

    System.out.println("---queue---");
    while (!q.isEmpty()) {
        System.out.println(q.poll());
    }
}
```

`Stack`과 `Queue`에 데이터를 저장하고 출력하는 기본적인 예제다. 여기서 특이점을 발견했는가? 

Java에서 스택은 `Stack클래스`로 구현해 제공하고 있지만, 큐는 `Queue인터페이스`로만 정의해 놓았을 뿐 별도의 클래스를 제공하지 않고 있다. 대신 Queue인터페이스를 구현한 클래스들이 있어서 이 중에 하나를 선택해서 사용하면 된다.  

큐는 데이터를 꺼낼 때 항상 첫 번째에 저장된 데이터를 삭제하므로, `ArrayList`와 같이 배열기반의 컬렉션 클래스를 사용하면 데이터를 꺼낼 때마다 빈 공간을 채우기 위해 데이터의 복사가 발생하므로 비효율적이다. 그래서 큐는 데이터의 추가/삭제가 쉬운 `LinkedList`로 구현하는 것이 적합하다.

### **HashSet**

`HashSet`은 `Set`인터페이스를 구현한 가장 대표적인 컬렉션이다. `HashSet`에 새로운 요소를 추가할 때는 `add`메소드나 `addAll`메소드를 사용하는데, 이미 `HashSet`에 저장되어 있는 요소와 중복된 요소를 추가하고자 한다면 이 메소드들은 `false`를 반환함으로써 중복된 요소이기때문에 추가에 실패했다는 것을 알린다.

`List`인터페이스를 구현한 컬렉션과 달리 `HashSet`은 저장순서를 유지하지 않으므로 저장순서를 유지하고자 한다면 `LinkedHashSet`을 사용해야 한다. 

|생성자|설명|
|--:|--:|
|HashSet()|HashSet 객체를 생성|
|HashSet(Collection c)|주어진 컬렉션을 포한하는 HashSet 객체 생성|
|HashSet(int initialCapacity)|주어진 값을 초기용량으로 하는 HashSet 객체 생성|
|HashSet(int initialCapacity, float loadFactor)|초기용량과 loadFactor를 지정하는 생성자|

> load factor는 컬렉션 클래스의 저장공간이 가득 차기 전에 미리 용량을 확보하기 위한 것으로 이 값을 0.8로 지정하면, 저장공간의 80%가 채워졌을 때 용량이 두 배로 늘어난다. 기본값은 0.75이다.  

```java
class HashSetEx{
    public static void main(String[] args) {
        Object[] objArr = {"1",new Integer(1), "2","2","2","3","4","4","4"};
        Set set = new HashSet();

        for (int i = 0; i < objArr.length; i++) {
            set.add(objArr[i]);
        }
        System.out.println(set);
    }
}
```
```console
[1, 1, 2, 3, 4]
```

`HashSet`을 사용한 기본적인 예제이다. 그런데 예상과 달리, 결과를 보면 1이 두 번 출력되는 것을 알 수 있다. 두개의 1 중에서 하나는 `String인스턴스`이고 다른 하나는 `Integer인스턴스`로 서로 다른 객체이므로 중복으로 간주되지 않는 것이다.

또 다른 예제를 살펴보자
```java
class Digimon{
    String name;
    int age;

    public Digimon(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + ":" + age;
    }

    public static void main(String[] args) {
        Set<Digimon> set = new HashSet<>();
        set.add(new Digimon("agumon", 10));
        set.add(new Digimon("agumon", 10));

        System.out.println(set);
    }
}
```
```console
[agumon:10, agumon:10]
```

`Digimon` 클래스는 `name`과 `age`를 멤버변수로 갖는다. 이름과 나이가 같으면 같은 객체로 인식하도록 하려는 의도로 작성하였지만, 결과를 보면 name과 age가 같음에도 불구하고 서로 다른 것으로 인식하여 `agumon:10`가 두 번 출력되었다. 

원래 클래스 작성 의도대로 이 두 인스턴스를 같은 것으로 인식하게 하려면 어떻게 해야 할까? 

`HashSet`의 `add()`는 새로운 요소를 추가하기 전에 기존에 저장된 요소와 같은 것인지 판별하기 위해 추가하려는 요소의 `equals()`와 `hashCode()`를 호출하기 때문에 `equals()`와 `hashCode()`를 목적에 맞게 오버라이딩해야 한다.

```java
@Override
public int hashCode() {
    return (name+age).hashCode();
}

@Override
public boolean equals(Object obj) {
    if (obj instanceof Digimon) {
        Digimon temp = (Digimon)obj;
        return name.equals(temp.name) && age == temp.age;
    }
    return false;
}
```

두 인스턴스의 `name`과 `age`가 서로 같으면 `true`를 반환하도록 `equals()`를 오버라이딩했다. 그리고 `hashCode()`는 `String`클래스의 `hashCode()`를 이용해서 구현했다. 

위의 `hashCode()`를 `JDK 1.8`부터 추가된 `java.util.Objects`클래스의 `hash()`를 이용해서 작성하면 아래와 같다. 
```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

### **TreeSet**
`TreeSet`은 `이진 검색 트리(binary search tree)`의 형태로 데이터를 저장하는 컬렉션 클래스다. 이진 검색 트리는 정렬, 검색, 범위검색(range search)에 높은 성능을 보이는 자료구조이며, `TreeSet`은 이진 검색 트리의 성능을 향상시킨 `레드-블랙 트리`로 구현되어 있다.

그리고 `Set`인터페이스를 구현했으므로 중복된 데이터의 저장을 허용하지 않으며, 정렬된 위치에 저장하므로 저장순서를 유지하지도 않는다. 

이진 검색 트리는 부모노드의 왼쪽에는 부모노드보다 작은 값의 자식노드를, 오른쪽에는 부모노드보다 큰 값의 자식노드를 저장하기 때문에 `TreeSet`에 저장되는 객체가 `Comparable`을 구현하거나, `TreeSet`에게 `Comparator`를 제공해서 두 객체를 비교할 방법을 알려줘야 한다.

```java
class TreeSetLotto{
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();

        for (int i = 0; set.size() < 6; i++) {
            set.add((int) (Math.random() * 45) + 1);
        }

        System.out.println(set);
    }
}
```
```console
[9, 13, 15, 33, 36, 41]
```

`TreeSet`은 이진 검색 트리로 저장되기 때문에, 저장 시에 이미 정렬되어 읽어올 때 따로 정렬할 필요가 없다. 

### **HashMap과 Hashtable**
`HashMap`과 `Hashtable`은 앞서 살펴 본 Vector와 ArrayList의 관계와 같아서 가능한 `HashMap`을 사용할 것을 권장한다. 

`HashMap`은 `Map`을 구현했으므로 이전에 언급한 `Map`의 특징을 가지면서 `해싱(hashing)`을 사용하기 때문에 많은 양의 데이터를 검색하는데 있어서 뛰어난 성능을 보인다.

다음은 `HashMap` 클래스 중 일부분이다.
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, 
                                                        Cloneable, Serializable {
    transient Node<K,V>[] table;
    ...
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        ...
    }
    ...
}
```

`HashMap`은 `Node`라는 내부 클래스를 정의하고, 다시 `Node` 타입의 배열을 선언하고 있다. `key`와 `value`는 서로 별개의 값이 아니기때문에 각각의 배열로 선언하기 보다는 하나의 클래스로 정의해서 배열로 다루는 것이 데이터의 무결성적인 측면에서 더 바람직하기 때문이다.  

|생성자|설명|
|--:|--:|
|HashMap()|HashMap 객체 생성|
|HashMap(int initialCapacity)|주어진 값을 초기용량으로 하는 HashMap 객체 생성|
|HashMap(int initialCapacity, float loadFactor)|초기용량과 loadFactor를 지정하는 생성자|
|HashMap(Map m)|지정된 Map의 모든 요소를 포함하는 HashMap 객체 생성|

```java
class HashMapEx{
    public static void main(String[] args) {
        String[] data = {"A","K","A","K","D","K","A","K","A","K","K","K","Z","D"};
        HashMap<String, Integer> map = new HashMap<>();

        for (int i = 0; i < data.length; i++) {
            if (map.containsKey(data[i])) {
                Integer value = map.get(data[i]);
                map.put(data[i], value + 1);
            }
            else{
                map.put(data[i], 1);
            }
        }

        Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();

        while (it.hasNext()) {
            Map.Entry<String, Integer> entry = it.next();
            int value = entry.getValue();       //auto boxing
            System.out.println(entry.getKey() + ":" + 
                                        printBar('*',value) + " "+ value);
        }
    }

    private static String printBar(char s, int value) {
        char[] bar = new char[value];
        for (int i = 0; i < bar.length; i++) {
            bar[i] = s;
        }
        return new String(bar);
    }
}
```
```console
A:**** 4
D:** 2
Z:* 1
K:******* 7
```
한정된 범위 내에 있는 순차적인 값들의 빈도수는 배열을 이용하지만, 이처럼 한정되지 않은 범위의 비순차적인 값들의 빈도수는 `HashMap`을 이용해서 구할 수 있다. 
>결과를 통해 HashMap과 같이 해싱을 구현한 컬렉션 클래스들은 저장순서를 유지하지 않는다는 사실을 다시 한 번 확인하자. 

### **TreeMap**
`TreeMap`은 이름에서 알 수 있듯이 `이진 검색 트리`의 형태로 키와 값의 쌍으로 이루어진 데이터를 저장한다. 그래서 검색과 정렬에 적합한 컬렉션 클래스다. 

앞에서 다 설명한 내용들의 조합이기에 특별한 점은 없다. `HashMap`과 `TreeMap`의 검색 성능은 대부분의 경우에서 `HashMap`이 더 뛰어나지만, 범위검색이나 정렬이 필요한 경우에는 `TreeMap`을 사용하도록 하자.

```java
class TreeMapEx{
    public static void main(String[] args) {
        String[] data = {"A","K","A","K","D","K","A","K","A","K","K","K","Z","D"};

        TreeMap<String, Integer> map = new TreeMap<>();

        for (int i = 0; i < data.length; i++) {
            if (map.containsKey(data[i])) {
                Integer value = map.get(data[i]);
                map.put(data[i], value + 1);
            }
            else{
                map.put(data[i], 1);
            }
        }

        Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
        System.out.println("---기본정렬---");
        while (it.hasNext()) {
            Map.Entry<String, Integer> entry = it.next();
            int value = entry.getValue();
            System.out.println(entry.getKey() + ":" + 
                                        printBar('*',value) + " "+ value);
        }
        System.out.println();

        //map을 ArrayList로 변환한 후 Collections.sort()로 정렬
        Set<Map.Entry<String, Integer>> set = map.entrySet();
        List<Map.Entry<String, Integer>> list = new ArrayList<>(set);

        Collections.sort(list, new ValueComparator());

        it = list.iterator();
        System.out.println("---값의 크기가 큰 순으로 정렬---");
        while (it.hasNext()) {
            Map.Entry<String, Integer> entry = it.next();
            int value = entry.getValue();
            System.out.println(entry.getKey() + ":" + 
                                        printBar('*',value) + " "+ value);
        }
    }

    static class ValueComparator implements Comparator {
        @Override
        public int compare(Object o1, Object o2) {
            if (o1 instanceof Map.Entry && o2 instanceof Map.Entry) {
                Map.Entry e1 = (Map.Entry) o1;
                Map.Entry e2 = (Map.Entry) o2;

                int v1 = (Integer)e1.getValue();
                int v2 = (Integer)e2.getValue();

                return v2-v1;
            }
            return -1;
        }
    }


    private static String printBar(char s, int value) {
        char[] bar = new char[value];
        for (int i = 0; i < bar.length; i++) {
            bar[i] = s;
        }
        return new String(bar);
    }
}
```
```console
---기본정렬---
A:**** 4
D:** 2
K:******* 7
Z:* 1

---값의 크기가 큰 순으로 정렬---
K:******* 7
A:**** 4
D:** 2
Z:* 1
```
이 예제는 직전의 `HashMap`예제를 `TreeMap`으로 변경한 것인데, `TreeMap`을 사용했기 때문에 `HashMap`의 결과와 달리 기본적으로 `key`가 오름차순으로 정렬되어 있는 것을 알 수 있다. 

`key`가 `String`인스턴스이기 때문에 `String` 클래스에 정의된 정렬 기준에 의해서 정렬된 것이고, `Comparator`를 구현한 클래스로 `value`를 내림차순으로 정렬하는 방법도 알 수 있다.

### **그 외** 

#### **Arrays**
`Arrays`클래스에는 배열을 다루는데 유용한 메소드들이 정의되어 있다. 

- **배열의 복사 - copyOf(), copyOfRange()**  
`copyOf()`는 배열 전체를, `copyOfRange()`는 배열의 일부를 복사해서 새로운 배열을 만들어 반환한다. `copyOfRange()`의 지정된 범위의 끝은 포함되지 않는다. 

- **배열 채우기 - fill(), setAll()**  
`fill()`은 배열의 모든 요소를 지정된 값으로 채운다. `setAll()`은 배열을 채우는데 사용할 함수형 인터페이스를 매개변수로 받는다. 따라서 이 메소드를 호출할 때는 함수형 인터페이스를 구현한 객체를 매개변수로 전달하던가 람다식을 지정해야 한다.  

    ```java
    int[] arr = new int[5];
    Arrays.fill(arr, 9);
    Arrays.setAll(arr, (i) -> (int) (Math.random() * 5) + 1);
    Arrays.stream(arr).forEach(System.out::print);
    ```

- **배열의 정렬과 탐색 - sort(), binarySearch()**  
    `sort()`는 배열을 정렬할 때, `binarySearch()`는 배열에 저장된 요소를 검색할 때 사용한다. `binarySearch()`는 지정된 값이 저장된 위치(index)를 반환하는데, 반드시 배열이 정렬된 상태여야만 올바른 결과를 얻는다. 그리고 만일 검색한 값과 일치하는 요소들이 여러 개 있다면, 이 중에서 어떤 것의 위치가 반환될지는 알 수 없다. 

- **배열의 비교와 출력 - equals(), toString()**  
    `toString()`로 배열의 모든 요소를 문자열로 편하게 출력할 수 있다. 다만 `toString()`은 1차원 배열에서만 사용할 수 있으므로 다차원 배열에는 `deepToString()`을 사용해야 한다. `deepToString()`은 배열의 모든 요소를 재귀적으로 접근해서 문자열을 구성하므로 2차원뿐만 아니라 3차원 이상의 배열에도 동작한다.

    `equals()`는 두 배열에 저장된 모든 요소를 비교해서 같으면 `true`를 반환한다. `equals()`도 일차원 배열에만 가능하므로, 다차원 배열의 비교에는 `deepEquals()`를 사용하자.

- **배열을 List로 반환 - asList(Object...a)**  
    `asList()`는 배열을 `List`에 담아서 반환한다. 매개변수의 타입이 가변인수라서 배열의 생성 없이 저장할 요소들을 나열하는 것도 가능하다.

    ```java
    List<Integer> list = Arrays.asList(new Integer[]{1, 2, 3, 4, 5});
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    list.add(6);        //UnsupportedOperationException 발생
    ```
    한가지 주의할 점은 `asList()`가 반환한 `List`의 크기를 변경할 수 없다는 것이다. 즉 추가 또는 삭제가 불가능하고 내용변경은 가능하다. 만약 크기를 변경할 수 있는 `List`가 필요하다면 다음과 같이 하면 된다. 
    ```java
    List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
    ``` 

#### **Comparator와 Comparable**
`Comparator`와 `Comparable`은 둘 다 인터페이스로, 컬렉션을 정렬하는데 필요한 메소드를 정의하고 있다. 
```java
public interface Comparable<T> {
    public int compareTo(T o);
}

@FunctionalInterface
public interface Comparator<T> {
    ...
    int compare(T o1, T o2);
    ...
}
```

`compare()`와 `compareTo()`는 선언형태와 이름이 약간 다를 뿐 두 객체를 비교한다는 같은 기능을 목적을 가진다. 두 메소드에서 비교하는 두 객체가 같으면 0, 비교하는 값보다 작으면 음수, 크면 양수를 반환하도록 구현해야 한다. 

```java
public final class Integer extends Number implements Comparable<Integer> {
    ...
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }

    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
    ...
}
```
위 코드는 실제 `Integer`클래스의 일부이다. `Comparable`의 `compareTo()`를 구현한 것을 알 수 있는데, 두 `Integer` 객체의 값을 비교해 같으면 0, 크면 1, 작으면 -1을 반환한다. 

`Comparable`을 구현한 클래스들이 기본적으로 오름차순으로 정렬되어 있지만, 내림차순으로 정렬하거나 혹은 어떤 다른 기준에 의해서 정렬되도록 하고 싶을 때 `Comparator`를 구현해서 정렬 기준을 제공할 수 있다. 
> Comparator는 익명 클래스로 사용되는 경우가 많다.

```java
public static void main(String[] args) {
    String[] arr = {"cat", "Dog", "lion", "tiger"};

    Arrays.sort(arr);
    System.out.println(Arrays.toString(arr));

    Arrays.sort(arr, String.CASE_INSENSITIVE_ORDER);
    System.out.println(Arrays.toString(arr));

    Arrays.sort(arr, new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            if (o1 instanceof Comparable && o2 instanceof Comparable) {
                return o1.compareTo(o2) * -1;
            }
            return -1;
        }
    });
    System.out.println(Arrays.toString(arr));
}
```
```console
[Dog, cat, lion, tiger]
[cat, Dog, lion, tiger]
[tiger, lion, cat, Dog]
```

`Arrays.sort()`는 배열을 정렬할 때, `Comparator`를 지정해주지 않으면 저장하는 객체에 구현된 내용에 따라 정렬된다. (주로 Comprable의 compareTo())

`String`의 `Comparable`구현은 문자열이 사전 순으로 정렬되도록 작성되어 있다. 문자열의 오름차순 정렬은 공백, 숫자, 대문자, 소문자의 순으로 정렬되는 것을 의미하고, 정확히 얘기하면 유니코드의 순서가 작은 값부터 큰 값으로 정렬되는 것이다.

```java
public static final Comparator<String> CASE_INSENSITIVE_ORDER
                                         = new CaseInsensitiveComparator();
```
이 `Comparator`를 이용하면 문자열을 대소문자 구분없이 정렬할 수 있다.  

`String`의 기본정렬을 반대로 하는 것, 즉 문자열을 내림차순으로 구현하는 것은 아주 간단한데, `String`에 구현된 `compareTo()`의 결과에 -1을 곱하기만 하면 된다. 또는 비교하는 객체의 위치를 바꿔서 `o2.compareTo(o1)`과 같이 해도 된다. 

## 참조
[자바의 정석](http://www.yes24.com/Product/Goods/24259565)  