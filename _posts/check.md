
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

java ServiceLoader
interface만 정의하고 그 것을 구현한 jar 파일을 갈아끼우면서? 그 구현체를 사용할 수 있는 것?

-----------------------------------------------------------


#### 문자기반 스트림 - Reader, Writer
지금까지 알아본 스트림은 모두 바이트기반의 스트림이었다. 바이트 기반은 입출력의 단위가 1byte라는 의미이다. 
자바에서 char형의 크기는 2byte이기때문에 바이트기반의 스트림으로 문자를 처리하는 데는 어려움이 있다.
이 점을 보완하기 위해 문자기반의 스트림이 제공되는데 문자데이터를 입출력할 때는 바이트 기반 스트림 대신 문자기반 스트림을 사용하는 것을 권장한다.

```java
InputStream -> Reader
OutputStream -> Writer
```
|바이트기반 스트림|문자기반 스트림|
|--:|--:|
|FileInputStream<br>FileOutputStream|FileReader<br>FileWriter|
|ByteArrayInputStream<br>ByteArrayOutputStream|CharArrayReader<br>CharArrayWriter|
|PipedInputStream<br>PipedOutputStream|PipedReader<br>PipedWriter|
|StringBufferInputStream<br>StringBufferOutputStream|StringReader<br>StringWriter|

문자기반 스트림의 이름은 바이트 기반 스트림의 이름에서 InputStream은 Reader로, OutputStream은 Writer로 바꾸기만 하면 된다. 단, ByteArrayInputStream에 대응하는 문자기반 스트림은 char배열을 사용하는 CharArrayReader이다. 

|Reader|Writer|
|--:|--:|
|int read()|void write(int c)|
|int read(char[] cbuf)|void write(char[] cbuf)|
|abstract int read(char[] cbuf, int off, int len)|abstract void write(char[] cbuf, int off, int len)|
||void write(String str)|
||void write(String str, int off, int len)|

위이 표는 문자기반 스트림의 읽기와 쓰기에 사용되는 메소드이다. 바이트기반 스트림과 비교하면 byte 배열 대신 char 배열을 사용하는 것과 추상 메소드가 달라졌다는 차이점이 있다. 

Reader와 Writer 모두 추상메소드가 아닌 메소드들은 추상메소드를 이용해서 작성되었다.  보조스트림도 문자기반 보조 스트림이 존재하며, 역시 InputStream을 Reader로 바꾸고 OutputStream을 Writer로 바꾸기만 하면 된다. 사용목적과 방식은 바이트기반 보조스트림과 다르지 않다. 



### 문자기반 스트림
문자데이터를 다루는데 사용된다는 것을 제외하고는 바이트기반 스트림과 문자기반 스트림의 사용방법은 거의 같기 때문에 앞서 설명한 바이트기반 스트림에 대한 내용만으로도 쉽게 이해가 가능할 것이다. 

#### Reader와 Writer
문자기반 스트림의 조상은 Reader와 Writer이다. byte배열 대신 char 배열을 사용한다는 것 외에는 InputStream/OutputStream의 메소드와 다르지않다.

Reder의 메소드

|메소드명|설명|
|--:|--:|
|abstract void close()|입력스트림을 닫음으로써 자원을 반환|
|void mark(int readlimit)|현재위치를 표시해놓는다. 후에 reset()에 의해 표시해놓은 위치로<br> 돌아갈 수 있고 readlimit은 되돌아갈 수 있는 byte의 수|
|boolean markSupported()|mark()와 reset()을 지원하는지 알려줌|
|int read()|입력소스로부터 하나의 문자를 읽어온다. char의 범위인 0~65536범위의 정수를 반환하며, 입력스트림의 마지막에 도달하면 -1을 반환한다.|
|int read(char[] c)|배열 c의 크기만큼 읽어서 배열을 채우고 읽어 온 데이터의 수를 반환|
|int read(char[] c,<br>int off, int len)|입력소스로부터 최대 len개의 문자를 읽어서 배열 c의 지정된 위치(off)부터 읽은만큼 저장한다.|
|int read(CharBuffer target)|입력소스로부터 읽어서 문자버퍼에 저장|
|boolean ready()|입력소스로부터 데이터를 읽을 준비가 되어있는지 알려줌|
|void reset()|입력소스에서 위치를 마지막으로 mark()가 호출된 위치로 되돌림 |
|long skip(long n)|현재 위치에서 주어진 길이(n)만큼을 건너뜀|

Writer의 메소드

|메소드명|설명|
|--:|--:|
|Writer append(char c)|지정된 문자를 출력소스에 출력|
|Writer append(CharSequence c)|지정된 문자열을 출력소스에 출력|
|Writer append(CharSequence c, <br>int start, int end)|지정된 문자열의 일부를 출력소스에 출력<br>(CharBuffer, String, StringBuffer가 구현)
|void close()|출력스트림을 닫음으로써 자원을 반환|
|void flush()|스트림의 버퍼에 있는 모든 내용을 출력소스에 write|
|void write(int b)|주어진 값을 출력소스에 write|
|void write(char[] c)|배열 b에 저장된 모든 내용을 출력소스에 write|
|void write(char[] c,<br>int off,int len)|배열 c에 저장된 내용 중에서 <br>off번째부터 len개만큼 읽어서 출력소스에 write|
|void write(String str)|주어진 문자열을 출력소스에 write|
|void write(STring str, int off, int len)|주어진 문자열의 off부터 len개만큼의 문자열을 write|

문자기반 스트림이라는 것이 단순히 2byte로 스트림을 처리하는 것만을 의미하지는 않느다. 문자 데이터를 다루는데 필요한 또 하나의 정보는 인코딩(encoding)이다. 
문자기반 스트림은 여러 종류의 인코딩과 자바에서 사용하는 유니코드(UTF-16)간의 변환을 자동적으로 처리해준다. Reader는 특정 인코딩을 읽어서 유니코드로 변환하고 Writer는 유니코드를 특정 인코딩으로 변환하여 저장한다.


#### FileReader와 FileWriter
FileReader와 FileWriter는 파일로부터 텍스트 데이터를 읽고 파일에 쓰는데 사용된다. 사용하는 방법은 FileInputStream/FileOutputStream과 다르지않다. 
```java
FileReader fr = new FileReader("text.txt");
int data = 0;
while ((data = fr.read()) != -1) {
    System.out.print((char) data);
}
System.out.println();
fr.close();
```

#### PipedReader와 PipedWriter
PipedReader와 PipedWriter는 쓰레드 간에 데이터를 주고받을 때 사용된다. 다른 스트림과는 달리 입력과 출력스트림을 하나의 스트림으로 연결해서 데이터를 주고받는다는 특징이 있다.

스트림을 생성한 다음에 어느 한쪽 쓰레드에서 connect()를 호출해서 입력스트림과 출력스트림을 연결한다. 입출력을 마친 후에는 어느 한쪽 스트림만 닫아도 나머지 스트림은 자동으로 닫힌다. 이 점을 제외하고는 일반 입출력 방법과 다르지 않다.

#### StringReader와 StringWriter
StringReader와 StringWriter는 CharArrayReader와 CharArrayWriter처럼 입출력의 대상이 메모리인 스트림이다. StringWriter에 출력되는 데이터는 내부의 StringBuffer에 저장된다.

### 문자기반의 보조스트림
#### BufferedReader와 BufferedWriter
BufferedReader와 BufferedWriter느 버퍼를 이용해 입출력 효율을 높인다. BufferedReader의 readLine()을 사용하면 데이터를 라인단위로 읽을 수 있고, BufferedWriter는 newLine()이라는 줄바꿈 메소드를 가지고 있다.  

```java
FileReader fr = new FileReader("IOTest.java");
BufferedReader br = new BufferedReader(fr);
String line = "";
for (int i = 1; (line = br.readLine()) != null; i++) {
    if(line.indexOf(";")!=-1)
        System.out.println(i + ":" + line);
}
br.close();
```
BufferedReader의 readLine()을 이용해서 파일을 라인단위로 읽은 다음 indexOf()를 이용해서 ';'를 포함하고 있는지 확인하여 출력하는 예제이다. 이렇게 파일에서 특정 문자 또는 문자열을 포함한 라인을 쉽게 찾을 수 있다는 것을 보여준다.

#### InputStreamReader와 OutputStreamWriter
InputStreamReader와 OutputStreamWriter는 바이트기반 스트림을 문자기반 스트림으로 연결시켜주는 역할을 한다. 그리고 바이트기반 스트림의 데이터를 지정된 인코딩의 문자데이터로 변환하는 작업을 수행한다.

|생성자|설명|
|--:|--:|
|InputStreamReader(InputStream in)|	OS에서 사용하는 기본 인코딩의 문자로 변환하는 InputStreamReader를 생성한다.|
|InputStreamReader(InputStream in, String encoding)	|지정된 인코딩을 사용하는 InputStreamReader를 생성한다.|
|String getEncoding()|InputStreamReader의 인코딩을 반환한다.|

|생성자|설명|
|--:|--:|
|OutputStreamWriter(OutputStream out)|OS에서 사용하는 기본 인코딩의 문자로 변환하는 OutputStreamWriter를 생성한다.|
|OutputStreamWriter(OutputStream out, String encoding)|지정된 인코딩을 사용하는 OutputStreamWriter를 생성한다.|
|String getEncoding()|OutputStreamWriter의 인코딩을 알려준다.|

만약 인코딩을 지정해주지 않는다면 OS에서 사용하는 디폴트 인코딩을 사용해서 파일을 보여주기 때문에 원래 작성된 데로 볼 수 없는 경우가 있다. 
```java
InputStreamReader isr = new InputStreamReader(System.in);
BufferedReader br = new BufferedReader(isr);
String line = "";
System.out.println("사용중인 OS의 인코딩 : " + isr.getEncoding());
do {
    System.out.println("문장을 입력하세요. (마치려면 q)");
    line = br.readLine();
    System.out.println("입력한 문장 : " + line);
} while (!line.equalsIgnoreCase("q"));
System.out.println("시스템을 종료합니다.");
```

BufferedReader의 readLine()을 이용해서 사용자의 입력을 라인단위로 받으면 편리하다. 그래서 BufferedReader와 InputStream인 System.in을 연결하기 위해 InputStreamReder를 사용하였다. JDK 1.5부터는 Scanner가 추가되어 이렇게 사용하지 않아도 간단하게 사용 가능하다.

### 표준입출력과 File
#### 표준 입출력 - System.in, System.out, System.err
표준입출력은 콘솔을 통한 데이터의 입력과 콘솔로의 데이터 출력을 의미한다. 자바에서는 표준 입출력(standart I/O)을 위해 세 가지 입출력 스트림을 제공하는데, 이 들은 자바 애플리케이션의 실행과 동시에 사용할 수 있게 자동적으로 생성되기 때문에 개발자가 별도로 스트림을 생성하는 코드를 작성하지 않고도 사용이 가능하다.
```java
System.in //콘솔로부터 데이터를 입력받는데 사용
System.out //콘솔로 데이터를 출력하는데 사용
System.err //콘솔로 데이터를 출력하는데 사용
```
여기서 in, out, err은 System 클래스에 선언된 클래스변수(static변수)이다. 
아래는 실제 System 클래스의 일부분이다.
```java
public static final InputStream in = null;
...
public static final PrintStream out = null;
...
public static final PrintStream err = null;
```

선언부분만 보면 in, out, err은 InputStream, PrintStream이지만 실제로는 버퍼를 사용하는 BufferedInputStream과 BufferedOutputStream의 인스턴스를 사용한다.

그래서 콘솔 입력은 버퍼를 가지고있기 때문에 입력 중에 backspace를 통해서 내용을 지우거나 수정할 수 있는 것이다. 따라서 Enter키나 입력의 끝을 알리는 '^z'를 누르기 전까지는 아직 데이터가 입력중인 것으로 간주되어 커서가 립력을 계속 기다리는 상태(blocking)에 머무르게 된다.  

콘솔에 데이터를 입력하고 Enter키를 누르면 입력대기상태에서 벗어나 입력된 데이터를 읽기 시작한다. 이 것으로 알 수 있듯이 Enter키를 누르는 것은 두 개의 특수문자 '\r'과 '\n'이 입력된 것으로 간주된다. '\r'은 캐리지리턴(carriage return), 즉 커서를 현재 라인의 첫 번째 column으로 이동시키고, '\n'은 커서를 다음 줄로 이동시키는 줄바꿈(new line)을 한다.

그래서 Enter키를 누르면, 캐리지리턴과 줄바꿈이 수행되어 다음 줄의 첫 번째 column으로 커서가 이동하는 것이다. 여기서 한 가지 문제는 Enter키도 사용자 입력으로 간주되어 매 입력마다 '\r'과 '\n'이 붙기 떄문에 이 것을 제거해주어야 하는 불편함이 있다. 

이러한 불편함을 제거하려면 이 전에 살펴본 것과 같이 System.in에 BufferedReader를 이용해서 readLine()을 통해 라인단위로 데이터를 입력받으면 된다. Java에서는 콘솔을 통한 입력에 대한 지원이 미약했지만 나중에 Scanner와 Console같은 클래스가 추가되면서 많이 보완되었다.

#### 표준입출력의 대상 변경 - setOut(), setErr(), setIn()
초기에는 표준입출력의 대상이 콘솔화면이지만, setOut(), setErr(), setIn()를 사용하면 입출력을 콘솔 이외의 다른 대상으로 변경할 수 있다.
```java
FileOutputStream fos = new FileOutputStream("test.txt");
PrintStream ps = new PrintStream(fos);
System.setOut(ps);

System.out.println("Hello by System.out");
System.err.println("Hello by System.err");
```
```console
Hello by System.err
```
System.out의 출력소스를 test.txt 파일로 변경하였기 때문에 System.out을 이용한 출력은 모두 test.txt파일에 저장된다. 이 방법 외에도 커맨드라인에서 표준입출력의 대상을 간단히 바꿀 수 있는 방법이 있다.

```console
yeojiwon@yeojiwon-ui-MacBookAir week13 % java IOTest
out : Hello World!
err : Hello World!
```
IOTest의 System.out 출력을 콘솔이 아닌 output.txt로 지정하려면 아래와 같이 한다. 만약 기존에 output.txt 파일이 있었다면 기존의 내용은 삭제된다.
```console
yeojiwon@yeojiwon-ui-MacBookAir week13 % java IOTest > output.txt
err : Hello World!
yeojiwon@yeojiwon-ui-MacBookAir week13 % type output.txt
out : Hello World!
``` 

만약 기존의 내용을 지우지 않고 기존 내용의 마지막에 새로운 내용을 추가하고 싶다면 >>를 사용한다.
```console
yeojiwon@yeojiwon-ui-MacBookAir week13 % java IOTest >> output.txt
err : Hello World!
yeojiwon@yeojiwon-ui-MacBookAir week13 % type output.txt
out : Hello World!
out : Hello World!
``` 

#### RandomAccessFile
자바에서는 입력과 출력이 각각 분리되어 별도로 작업을 하도록 설계되어 있는데, RandomAccessFile만은 하나의 클래스로 파일에 대한 입력과 출력을 모두 할 수 있도록 되어있다.
InputStream이나 OutputStream이 아닌 DataInput과 DataOutput을 구현하기 때문에 기본 자료형 단위로 데이터를 읽고 쓸 수 있다. 
그래도 역시 RandomAccessFile클래스의 가장 큰 장점은 파일의 어느 위치에서나 읽기/쓰기가 가능하다는 것이다. 다른 입출력 클래스들은 입력소스에 순차적으로 읽기/쓰기를 하기 때문에 읽기와 쓰기가 제한적인데 반해서 RandomAccessFile클래스는 파일에 읽고 쓰는 위치에 제한이 없다.

이것을 가능하게 하기 위해서 내부적으로 파일 포인터를 사용하는데, 입출려 시에 작업이 수행되는 곳이 바로 파일 포인터가 위치한 곳이 된다. 만약 파일의 임의의 위치에 있는 내용에 대해 작업하고자 한다면 seek(long pos)나 skipBytes(int n)를 사용하면 된다.

#### File
파일은 기본적이면서도 가장 많이 사용되는 입출력 대상이기 때문에 중요하다. 자바에서는 File 클래스를 통해서 파일과 디렉토리를 다룰 수 있다. 그래서 File인스턴스는 파일일 수도 있고, 디렉토리일 수도 있다. 

### 직렬화 (Serialization)

#### 직렬화란?
직렬화란 객체를 데이터 스트림으로 만드는 것을 뜻한다. 다시 얘기하면 객체에 저장된 데이터를 스트림에 쓰기(write)위해 연속적인(serial) 데이터로 변환하는 것을 말한다.

반대로 스트림으로부터 데이터를 읽어서 객체를 만드는 것을 역직렬화(deserialization)라고 한다.

직렬화라는 용어때문에 어렵게 느껴질 수 있으나 사실 객체를 저장하거나 전송하려면 이렇게 할 수 밖에 없다. 여기서 객체란 무엇이며, 객체를 저장한다는 것은 무엇을 의미하는가에 대해서 다시 한 번 정리해보자

객체는 클래스에 정의된 인스턴스 변수의 집합이다. 객체에는 클래스 변수나 메소드가 포함되지 않고 오직 인스턴스 변수들로만 구성되어 있다. 인스턴스 변수는 인스턴스마다 다른 값을 가질 수 있어야하기 때문에 별도의 메모리공간이 필요하지만 메소드는 변하는 것이 아니라서 메모리를 낭비해가면서 인스턴스마다 같은 내용을 포함시킬 필요가 없기 때문이다. 

클래스에 정의된 인스턴스 변수가 단순히 기본형일 때는 인스턴스 변수의 값을 저장하는 일이 간단하지만, 인스턴스 변수의 타입이 참조형일 때는 그리 간단하지 않다. 하지만 우리에게는 객체를 직렬화/역직렬화할 수 있는 ObjectInputStream과 ObjectOutputStream이 있기때문에 걱정할 필요가 없다 

#### ObjectInputStream과 ObjectOutputStream
직렬화에는 ObjectInputStream를 사용 하고 역직렬화에는 ObjectOutputStream를 사용한다. ObjectInputStream과 ObjectOutputStream는 각각 InputStream과 OutputStream을 직접 상속받지만 보조스트림이다. 따라서 객체를 생성할 때 입출력할 스트림을 지정해주어야 한다. 

만일 파일에 객체를 저장하고 싶다면 다음과 같이 하면 된다.
```java
FileOutputStream fos = new FileOutputStream("objectfile.txt");
ObjectOutputStream out = new ObjectOutputStream(fos);
out.writeObject(new UserInfo());
```
ObjectOutputStream의 writeObject(Object obj)를 사용해서 객체를 출력하면, 객체가 파일에 직렬화되어 저장된다. 
역직렬화 역시 간단한데, 입력스트림을 사용하고 readObject()를 통해 저장된 데이터를 읽기만 하면 객체로 역직렬화된다. 다만 readObject()의 반환타입이 Object이기때문에 객체의 원래타입으로 형변환 해주어야 한다.

#### 직렬화가 가능한 클래스 만들기 - Serializable, transient
직렬화가 가능한 클래스로 만들기 위해서는 java.io.Serializable 인터페이스를 구현하도록 하면 된다. Serializable은 빈 인터페이스이지만 직렬화를 고여하여 작성한 클래스인지를 판단하는 기준이 된다.

또한, Serializable를 구현한 클래스를 상속하는 자식 클래스는 Serializable를 구현하지 않아도 직렬화가 가능하다. 그러나 다음과 같이 부모 클래스가 Serializable를 구현하지 않았다면 자식 클래스를 직렬화할 때 부모 클래스에 정의된 인스턴스 변수 name과 strength는 직렬화 대상에서 제외된다.

```java
class Monster{
    String name;
    int strength;
}
class Digimon extends Monster implements Serializable{
    String skill;
}
```
이런 경우 조상 클래스에 정의된 인스턴스 변수까지 직렬화 대상에 포함시키기 위해서는 부모 클래스가 Serializable를 구현하도록 하던가 Digimon 클래스에서 부모 클래스의 인스턴스 변수들이 직렬화되도록 처리하는 코드를 직접 추가해야 한다.

그리고 만약 Serializable를 구현한 클래스에서 인스턴스 변수로 직렬화할 수 없는 클래스의 객체를 참조하고 있으면 java.io.NotSerializableException이 발생한다. 
```java
class Digimon extends Monster implements Serializable{
    String skill;
    Object obj = new Object();
}
```

하지만 아래의 경우에는 직렬화가 가능하다. 인스턴스 변수 obj의 타입이 직렬화가 되지않는 Object이긴 하지만 실제로 저장된 객체는 직렬화가 가능한 String 인스턴스이기 때문에 직렬화가 가능하다. 인스턴스 변수의 타입이 아닌 실제로 연결된 객체의 종류에 의해서 결정된다는 것을 기억하자.
```java
class Digimon extends Monster implements Serializable{
    String skill;
    Object obj = new String("abc");
}
```

직렬화하고자 하는 객체의 클래스에 직렬화가 되지 않는 객체에 대한 참조를 포함하고 있다면 제어자 transient를 붙여서 직렬화 대상에서 제외되도록 할 수 있다. 또는 보안상 직렬화되면 안되는 인스턴스 변수에 대해서도 transient를 사용할 수 있다. 
실제로 transient가 붙은 인스턴스 변수의 값은 그 타입의 기본값으로 직렬화된다.

```java
class Digimon extends Monster implements Serializable{
    String skill;
    transient String password;
    transient Object obj = new Object();
}
```

------------------------------------------------------------------------

버퍼를 사용하면 OS레벨의 System call 호출 횟수가 줄어들기 때문에 속도가 빨라지는 것

direct buffer vs none direct buffer

데코레이터 패턴?

시리얼라이즈에 serialVersionUID 언급하세요