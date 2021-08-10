java13부터 switch가 바뀐게 아니고 스위치문은 그대로 있고 13부터 스위치 오퍼레이터가 생긴거


```java
int i = 0;
int j = 0;

if(i++ == 0 || j++ == 0){
    System.out.println("Hello");
}
System.out.println(i);
System.out.println(j);

```

```console
Hello
1
0
//한 쪽을 검사해 조건이 만족되면 다른 쪽은 검사하지 않음
```

```java
int i = 0;
int j = 0;

if(i++ == 0 | j++ == 0){
    System.out.println("Hello");
}
System.out.println(i);
System.out.println(j);

```
```console
Hello
1
1
//양 쪽 조건을 모두 검사함
```

문제 : numbers라는 int형 배열이 있다. 이 배열에 요소들은 한 숫자를 제외하고는 모두 두 번씩 들어있다. 단 한 번만 등장하는 숫자를 찾는 코드를 작성하라.
```java
// XOR을 활용하는 방식
// XOR은 같으면 0 다르면 1
private int solution(int[] numbers){
    int result = 0;
    for(int number : numbers){
        result ^= number;
    }
    return result;
}
```