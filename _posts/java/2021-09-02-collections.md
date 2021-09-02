---
title: "컬렉션 프레임워크 (Collections Framework)"
author: "yeoji21"
date: '2021-09-02 03:35:00 +0900'
categories: [Backend, Java]
tags: [java, 자바]
---

## 목차
- 컬렉션 프레임워크란?
- 컬렉션 프레임워크의 핵심 인터페이스
- ArrayList

### **컬렉션 프레임워크란?**

컬렉션 프레임워크란, 데이터 군을 저장하는 클래스들을 표준화한 설계를 뜻한다. 여기서 컬렉션(Collection)은 다수의 데이터, 데이터 그룹을 의미하고 프레임워크(Framework)는 표준화된 프로그래밍 방식을 말한다.

JDK 1.2 이전까지는 `Vector`, `Hashtable`, `Properties`와 같은 커렉션 클래스들을 각자의 방식으로 처리해야 했으나, JDK 1.2부터 컬렉션 프레임워크가 등장하면서 다양한 종류의 컬렉션 클래스가 추가되고 모든 컬렉션 클래스를 표준화된 방식으로 다룰 수 있도록 체계화되었다.  

### **컬렉션 프레임워크의 핵심 인터페이스**

컬렉션 프레임워크에서는 컬렉션 데이터 그룹을 크게 세 가지 타입의 인터페이스로 나누었다. 그리고 인터페이스 `List`와 `Set`의 공통 부분을 뽑아서 새로운 인터페이스인 `Collection`을 추가로 정의했다. `Map` 인터페이스는 이들과는 전혀 다른 형태로 컬렉션을 다루기 때문에 같은 상속 계층도에 포함되지 못했다.

<img src="assets/../../../assets/img/collections/1.jpeg" width=500>

>JDK 1.5부터 Iterable 인터페이스가 추가되고 Collection 인터페이스가 이것을 상속받도록 변경되었으나. 이것은 단지 공통적인 메소드를 뽑아서 중복을 제거하기 위한 것이므로 상속 계층도에서 별 의미가 없다.

|인터페이스|특징|구현 클래스|
|--|--:|--:|
|List|순서가 있는 데이터의 집합, 데이터의 중복 허용 <br> 예) 대기자 명단| ArrayList, LinkedList, Stack, Vactor|
|Set|순서를 유지하지 않고 데이터의 중복이 허용되지 않음 <br> 예)양의 정수집합, 소수의 집합|HashSet, TreeSet|
|Map|key와 value의 쌍으로 이루어진 데이터의 집합<br>순서 x, key는 중복허용 x, value는 중복허용<br>예)우편번호, 전화번호|HashMap,TreeMap,Hashtable,Properties|

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