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