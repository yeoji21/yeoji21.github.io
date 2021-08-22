
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

----------------------------------

```java
public class ThreadTest {
    public static void main(String[] args) {
        ThreadEx1 t1 = new ThreadEx1("Thread By Thread");
        ThreadEx2 r = new ThreadEx2();
        Thread t2 = new Thread(r, "Thread By Runnable");

        t1.start();
        t2.start();
    }
}

@NoArgsConstructor
class ThreadEx1 extends Thread {
    public ThreadEx1(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName());
        }
    }
}

class ThreadEx2 implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName());
        }
    }
}
```

종료된 스레드는 다시 실행할 수 없기때문에 하나의 쓰레드에 대해 start()가 한 번만 호출될 수 있다.
그래서 만일 쓰레드의 작업을 한 번 더 수행해야 한다면 새로운 쓰레드를 생성해야 한다.  
그렇지않고 한 스레드에 대해 start()를 두 번 이상 실행하면 IllegalThreadStateException이 발생한다.


내가 만든 쓰레드에는 분명히 run()을 오버라이딩했는데 main메소드에서는 start()를 호출하는 이유가 무엇일까? 
main메소드에서 run()을 호출하는 것은 스레드를 생성하는 것이 아니라 단순히 클래스에 선언된 메소드를 호출하는 것일뿐이다.
반면에 start()는 새로운 쓰레드가 작업을 실행하는데 필요한 호출스택(call stack)을 생성한 다음에 run()을 호출해서 생성된 호출 스택에 run()이 첫번쨰로 올라가게 한다. 

모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출스택을 필요로 한다. 
호출스택에서는 가장 위에 있는 메소드가 현재 실행중인 메소드이고 나머지 메소드들은 대기상태이다. 그러나 스레드가 둘 이상일때는 맨 위에 있는 메소드일지라도 대기상태에 있을 수 있다. 
스케줄러는 실행대기 중인 쓰레드들의 우선순위를 고려하여 실행순서와 실행시간을 결정한다. 자바가 OS 독립적이라고 하지만 실제로 OS 종속적인 부분이 몇 가지 있는데 쓰레드도 그 중 하나이다.
이 때 주어진 시간동안 작업을 마치지 못한 쓰레드는 다시 대기상태로 가고,
작업을 마친 쓰레드, 즉 run()이 수행 종료된 쓰레드는 호출스택이 모두 비워지면서 쓰레드가 사용하던 호출스택은 사라진다. 

메인 메소드의 작업을 수행하는 것도 쓰레드이며 이것을 main 쓰레드라고 한다. 만약 main 메소드가 작업을 마쳤다하더라도 다른 쓰레드가 아직 작업을 마치지 않은 상태라면 프로그램이 종료되지 않는다. 
실행 중인 사용자 쓰레드가 하나도 없을 때 프로그램은 종료된다. 


싱글 스레드와 멀티 스레드
context switching시간이 있기 때문에 멀티 스레드를 쓴다고해서 항상 싱글 스레드보다 더 빠른건 아니다. 
하지만 두 쓰레드가 서로 다른 자원을 사용하는 작업의 경우에는 멀티쓰레드 프로세스가 더 효율적이다. 예를 들면 사용자로부터 데이터를 입력받는 작업, 네트워크로 파일을 주고받는 작업, 프린터로 파일을 출력하는 작업 등.. 입력을 기다리는 시간에 다른 쓰레드가 작업을 처리할 수 있기때문에 보다 효율적인 CPU의 사용이 가능하다. 

쓰레드의 우선순위 
쓰레드는 우선순위(priority)라는 멤버변수를 가진다. 이 우선순위 값에 따라 쓰레드가 얻는 실행시간이 달라진다. 따라서 스레드의 중요도에 따라 우선순위를 서로 다르게 지정하여 특정 쓰레드가 더 많은 작업시간을 가지도록 할 수 있다. 
```java
void setPriority(int newPriority)
int getPriority()
```
쓰레드가 가질 수 잇는 우선순위의 범위는 1~10까지이며 숫자가 높을 수록 우선순위가 높다. 한가지 알아두어야 할 점은 쓰레드의 우선순위는 쓰레드를 생성한 쓰레드로부터 상속받는다는 것이다.
main 메소드를 수행하는 쓰레드는 우선순위가 5이므로 main 메소드 내에서 생성하는 쓰레드의 우선순위는 자동적으로 5가 된다. 
그러나 멀티코어 환경에서는 쓰레드의 우선순위에 따른 차이가 전혀 없다. 따라서 차라리 쓰레드에 우선순위를 부여하는 작업 대신 작업에 우선순위를 두어 PriorityQueue에 저장해놓고, 우선순위가 높은 작업이 먼저 처리되도록 하는 것이 나을 수 있다. 

데몬 스레드
데몬 스레드는 일반 쓰레드의 작업을 돕는 보조적인 역할을 수행하는 쓰레드이다. 일반 쓰레드가 모두 종료되면 데몬 쓰레드는 강제적으로 종료된다. 이 점을 제외하고는 일반 쓰레드와 데몬 쓰레드는 다르지않다.
데몬 쓰레드의 예로는 가비지 컬렉터, 워드 프로세서의 자동저장, 화면자동갱신 등이 있다. 
데몬 쓰레드는 무한루프와 조건문을 이용해서 실행 후 대기하고 있다가 특정 조건이 만족되면 작업을 수행하고 다시 대기하도록 작성한다.

데몬 쓰레드는 일반 쓰레드의 작성방법과 실행방법이 같으며 다만 쓰레드를 생성한 다음 실행하기 전에 setDaemon(true)를 호출하기만 하면 된다. 그리고 데몬 쓰레드가 생성한 쓰레드는 자동으로 데몬 쓰레드가 된다는 점도 알아두자 
```java
boolean isDaemon()
void setDaemon(boolean on)
```

```java
class Thread10 implements Runnable {
    static boolean autoSave = false;

    public static void main(String[] args) {
        Thread t = new Thread(new Thread10());
        t.setDaemon(true);
        t.start();

        for (int i = 0; i < 10; i++) {
            try{
                Thread.sleep(1000);
            } catch (InterruptedException e) { }
            System.out.println(i);
            if (i == 5) {
                autoSave = true;
            }
        }
        System.out.println("프로그램을 종료합니다.");
    }

    @Override
    public void run() {
        while (true) {
            try{
                Thread.sleep(3 * 1000);
            } catch (InterruptedException e) { }
            if (autoSave) {
                autoSave();
            }
        }
    }

    public void autoSave() {
        System.out.println("작업파일이 자동저장되었습니다.");
    }
}
```

3초마다 autoSave 변수의 값을 확인하는 데몬 쓰레드이다. 만일 이 쓰레드를 데몬 쓰레드로 설정하지 않았다면, 이 프로그램은 강제종료하지 않는 한 영원히 종료되지 않을 것이다. 
setDaemon()을 start()를 호출하기 전에 실행해야 한다. 그렇지 않으면 IllegalThreadStateException이 발생한다. 

쓰레드의 실행제어
resume(), stop(), suspend()는 쓰레드를 교착상태(dead-lock)로 만들기 쉽기 때문에 deprecated 되었다. 

쓰레드의 상태
NEW - 쓰레드가 생성되고 아직 start()가 호출되지 않은 상태
RUNNABLE - 실행 중 또는 실행 가능한 상태 
BLOCKED - 동기화 블럭에 의해서 일시정지된 상태(lock이 풀릴 때까지 기다리는 상태)
WAITING, TIMED_WAITING - 쓰레드의 작업이 종료되지는 않았지만 실행가능하지 않은(unrunnable) 일시정지 상태, TIMED_WATING은 일시정지 시간이 지정된 경우를 의미한다. 
TERMINATED - 쓰레드의 작업이 종료된 상태
> 참고로 Java 5부터 getState() 메소드를 통해 Thread의 상태를 확인할 수 있게 되었다. 

sleep() - 일정시간동안 쓰레드를 멈추게한다.
sleep()은 항상 현재 실행중인 쓰레드에 대해 동작하기 때문에 th1.sleep(2000)과 같이 호출했어도 실제로 영향을 받는 것은 main 메소드를 실행하는 main 쓰레드이다. 
그래서 sleep()은 static으로 선언되어 있으며 참조변수를 이용해서 호출하기 보다는 Thread.sleep(2000)과 같이 해야 한다.

interrupt(), interrupted() - 쓰레드의 작업을 취소한다.
interrupt()는 쓰레드에게 작업을 멈추라고 요청한다. 단지 멈추라고 요청하는 것일 뿐 쓰레드를 강제로 종료시키지는 못한다. interrupt()는 그저 쓰레드의 인스턴스 변수인 interrupted변수를 바꾼다.  

```java
void interrupt() - 쓰레드의 interrupted 상태를 false에서 true로 변환 
boolean isInterrupted() - 쓰레드의 interrupted상태를 반환 
static boolean interrupted() - 현재 쓰레드의 interrupted 상태를 반환 후 false로 변경 
```
쓰레드가 sleep(), wait(), join()에 의해 '일시정지 상태(WAITING)'에 있을 때, 해당 쓰레드에 대해 interrupt()를 호출하면 InterruptedException이 발생하고 쓰레드는 '실행대기 상테(RUNNABLE)'로 바뀐다. 즉, 멈춰있던 쓰레드를 깨워서 실행가능한 상태로 만드는 것이다. 

```java
while (!stopped) {
    if (!suspend) {
        System.out.println(name);
        try{
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            System.out.println(name + " - interrupted");
        }
    }
    else{
        Thread.yield();
    }
}
```
```java
public void suspend(){
    suspend = true;
    th.interrupt();
    System.out.println(th.getName() + " - interrupted() by suspend()");
}

public void stop(){
    stopped = true;
    th.interrupt();
    System.out.println(th.getName() + " - interrupted() by stop()");
}
```

join() - 다른 쓰레드의 작업을 기다린다. 
쓰레드 자신이 하던 작업을 잠시 멈추고 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 할 때 join()을 사용한다.
시간을 지정하지 않으면 해당 쓰레드가 작업을 모두 마칠 때까지 기다린다. 작업 중에 다른 쓰레드의 작업이 먼저 수행되어야 할 필요가 있을 떄 join()을 사용한다. 

```java
try{
    th1.join();     //현재 실행중인 쓰레드가 쓰레드 th1의 작업이 끝날 때까지 기다린다.
}catch(InterruptedException e){}
```

쓰레드의 동기화
멀티쓰레드 프로세스의 경우, 여러 쓰레드가 같은 프로세스 내의 자원을 공유해서 작업하기 때문에 서로의 작업에 영향을 주게된다. 
이러한 일이 발생하는 것을 방지하기 위해서 한 쓰레드가 특정 작업을 끝마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 하는 것이 필요하다. 
그래서 도입된 개념이 바로 임계 영역(critical section)과 잠금(lock)이다. 

공유 데이터를 사용하는 코드 영역을 임계 영역으로 지정해놓고, 공유 데이터(객체)가 가지고 있는 lock을 획득한 단 하나의 쓰레드만 이 영역 내의 코드를 수행할 수 있게 한다.
그리고 해당 쓰레드는 임계 영역 내의 코드를 수행한 뒤 lock을 반납해야 한다. 

이처럼 한 쓰레드가 진행중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 '쓰레드의 동기화(synchronization)'라고 한다. 
자바에서는 synchronized 블럭을 이용해서 쓰레드의 동기화를 지원했지만, Java 5부터는 java.util.concurrent.locks와 java.util.concurrent.atomic 패키지를 통해서 다양한
방식으로 동기화를 구현할 수 있도록 지원하고 있다. 

1. synchronized를 이용한 동기화
이 키워드는 임계 영역을 설정하는데 사용된다.
```java
// 1. 메소드 전체를 임계 영역으로 지정
public synchronized void calcSum(){
    ...
} 
// 2.특정한 영역을 임계 영역으로 지정
synchronized(객체의 참조변수){
    ...
}
```
두 번째 방법은 메소드 내의 코드 일부를 블럭으로 감싸고 앞에 synchronized(객체의 참조변수)를 붙이는 것인데, 이 때 참조변수는 lock을 걸고자 하는 객체를 참조하는 것이어야 한다.
이 블럭의 영역 안으로 들어가면서부터 쓰레드는 지정된 객체의 lock을 얻게 되고, 이 블럭을 벗어나면 lock을 반납한다.  
위의 두 방법 모두 lock의 획득과 반납이 모두 자동으로 이루어지므로 우리가 해야할 일은 그저 임계 영역을 설정하는 것 뿐이다. 

모든 객체는 lock을 하나씩 가지고 있으며, 해당 객체의 lock을 가지고 있는 쓰레드만 임계 영역의 코드를 수행할 수 있다. 그리고 다른 쓰레드들은 lock을 얻을 때까지 기다리게 된다.
따라서 임계 영역은 멀티쓰레드 프로그램의 성능을 좌우하기 때문에 가능하면 메소드 전체에 lock을 거는 것보다 synchronized 블럭으로 임계 영역을 최소화해서 효율적인 프로그램이 되도록 노력해야 한다. 

```java
public void withdraw(int money) {
    if (balance >= money) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        balance -= money;
    }
}
```

wait()와 notify()
synchronized로 동기화해서 공유 데리터를 보호하는 것까지는 좋은데, 특정 쓰레드가 객체의 lock을 가진 상태로 오랜 시간을 보내지 않도록 하는 것도 중요하다. 
이 것을 위해 고안된 것이 바로 wait()와 notify()이다.

동기화된 임계영역의 코드를 수행하다가 작업을 더 이상 진행할 상황이 아니면, 일단 wait()를 호출하여 쓰레드가 lock을 반납하고 기다리게 한다. 이렇게 하면 다른 쓰레드가 lock을 얻어 해당 객체에
대한 작업을 수행할 수 있게 된다. 나중에 작업을 진행할 수 있는 상황이 되면 notify()를 호출해서 작업을 중단했던 쓰레드가 다시 lock을 얻어 작업을 진행할 수 있게 한다.
wait()와 notify()는 특정 객체에 대한 것이므로 Object 클래스에 정의되어있다.  

wait(), notify(), notifyAll()은 동기화 블록 내에서만 사용할 수 있는 메소드들이다. 
wait()이 호출되면 시행중이던 쓰레드는 해당 객체의 대기실(waiting pool)에서 통지를 기다린다. notify()가 호출되면 해당 객체의 대기실에 있던 모든 쓰레드 중에서 임의의 쓰레드만 통지를 받는다.
notifyAll()은 기다리고 있는 모든 쓰레드에게 통보를 하지만 그래도 lock을 얻을 수 있는 쓰레드는 단 하나뿐이다. 
그리고 waiting pool은 객체마다 존재하는 것이므로  notifyAll()이 호출된다고 해서 모든 객체의 waiting pool에 있는 쓰레드가 깨워지는 것은 아니다. 

Lock과 Condition을 이용한 동기화 
java.util.concurrent.locks 패키지가 제공하는 lock 클래스들을 이용하는 방법도 있다. 
synchronized 블럭으로 동기화를 하면 자동적으로 lock이 잠기고 풀리기때문에 편리하다. 심지어 synchronized 블럭 내에서 예외가 발생해도 lock은 자동적으로 풀린다. 
그러나 때로는 같은 메소드 내에서만 lock을 걸 수 있다는 제약이 불편하기도한데, 이럴 때 lock 클래스를 이용한다. lock 클래스의 종류는 다음과 같이 3가지가 있다.

```java
ReentrantLock //재진입 가능한 lock, 가장 일반적인 배타 lock
ReentrantReadWriteLock //읽기에는 공유적이고 쓰기에는 배타적인 lock
StampedLock //ReentrantReadWriteLock에 낙관적인 lock의 기능을 추가
```

ReentrantLock은 자동적으로 lock을 관리해주지 않기 때문에 직접 메소드를 호출하면서 lock을 얻고, 반납해야 한다.
```java
void lock()
void unlock()
boolean isLocked()
```

앞서 wait() & notify() 예제에서 요리사 쓰레드와 손님 쓰레드를 구분해서 통지하지 못한다는 단점이 있었다. Condition은 이러한 문제점을 해결하기 위한 것이다. 
손님 쓰레드를 위한 Condition과 요리사 쓰레드를 위한 Condition을 만들어서 각각의 waiting pool에서 따로 기다리도록 하는 것이다. 
Condition은 이미 생성된 lock으로부터 newCondition() 메소드를 호출해서 생성한다,
```java
private ReentrantLock lock = new ReentrantLock();
private Condition forCook = lock.newCondition();
private Condition forCust = lock.newCondition();
```
그 다음엔 wait()와 notify() 대신 Condition의 await()와 signal() 메소드를 사용하면 된다.

이렇게하면 요리사 쓰레드가 통지를 받아야하는 상황에서 손님 쓰레드가 통지를 받는 경우가 없어지기 때문에 기아 현상이나 경쟁 상태가 개선된다. 
그러나 여전히 특정 쓰레드를 선택할 수는 없기 때문에 같은 종류의 쓰레드간의 기아현상이나 경쟁상태가 발생할 가능성은 남아있다. 







