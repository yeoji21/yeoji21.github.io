```java
public class HelloWorld{

    static class Person{
        public void print(){
            System.out.println("Peson");
        }
    }

    static class Student extends Person{
        public void print(){
            System.out.println("Student");
        }
    }

     public static void main(String []args){
        Person a = new Student();
        a.print();
     }
}
```
이렇게 업 캐스팅하고 오버라이딩된 메소드를 호출하면 자식 클래스의 메소드로 호출됨 => 다형성
Dynamic method dispatch는 위와 같은 상황에서 런타임 시에 어떤 메소드가 호출될지 결정하는 것

dispatch는 static dispatch와 dynamic dispatch가 있는데 static은 구현클래스를 이용해 컴파일타임에서부터 어떤 메서드가 호출될지 정해져있는것이고, dynamic은 인터페이스를 이용해 참조함으로서 호출되는 메서드가 동적으로 정해지는걸 말한다.

출처: https://multifrontgarden.tistory.com/133 [우리집앞마당]   
위 링크 참조할것
