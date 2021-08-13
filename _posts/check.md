```java
Queue<Integer> queue = new LinkedList<>();

//아래 두 개씩 묶어진 메소드의 차이는 null일 때 예외를 던지는지 null을 반환하는지 
//아래에 있는 것들이 예외를 던지는 메소드들

queue.offer(1);
queue.add(1);

queue.poll();
queue.remove();

queue.peek();
queue.element();

//무엇을 쓰든 자신의 요구사항에 맞춰서 사용하면 되는데 중요한 것은 일관성
```
 
```java
public class StatefulTest{

    private int number;

    @Test
    void test1(){
        System.out.println(number);
    }

    @Test
    void test2(){
        System.out.println(number);
    }
}
```
```console
0
0
```
테스트 케이스마다 인스턴스를 새로 만들기 때문에 0 0이 찍힘

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class StatefulTest{

    private int number;

    @Test
    void test1(){
        System.out.println(number);
    }

    @Test
    void test2(){
        System.out.println(number);
    }
}
```
```console
0
1
```
이렇게 테스트의 instance를 per class로 설정하면 값을 공유하게 된다. (기본 값은 per method)

이 방법이 유용할 때는 테스트의 순서까지 지정해서 

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class StatefulTest{

    private int number;

    @Test @Order(2)
    void test1(){
        System.out.println(number);
    }

    @Test @Order(1)
    void test2(){
        System.out.println(number);
    }
}
```