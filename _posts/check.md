
메이븐이나 그레들 중 하나는 공부할 것  

자바 디자인 패턴

객체지향의 사실과 오해 정리

java ServiceLoader
interface만 정의하고 그 것을 구현한 jar 파일을 갈아끼우면서? 그 구현체를 사용할 수 있는 것?

-----------------------------------------------------------

### 스트림(stream) 
자바에서 입출력을 수행하려면, 즉 어느 한쪽에서 다른 쪽으로 데이터를 전달하려면 두 대상을 연결하고 데이터를 전송할 수 있는 무언가가 필요한데, 이 것을 스트림이라고 정의한다.
> Collections의 스트림과 이름이 같지만 다른 개념이다.

스트림은 연속적인 데이터의 흐름을 물에 비유해서 붙여진 이름인데, 여러 가지로 유사한 점이 많다. 물이 한쪽 방향으로만 흐르는 것처럼 스트림은 단방향통신만 가능하기때문에 하나의 스트림으로 입력과 출력을 동시에 처리할 수 없다. 
그래서 입력과 출력을 동시에 수행하려면 입력을 위한 입력 스트림과 출력을 위한 출력 스트림 두 가지가 필요하다. 

스트림은 먼저 보낸 데이터를 먼저 받게 되어 있으며 중간에 건너뜀없이 연속적으로 데이터를 주고받는다. queue와 같은 FIFO구조라고 생각하면 된다. 

#### 바이트기반 스트림 - InputStream, OutputStream
스트림은 바이트단위로 데이터를 전송하며 입출력 대상에 따라 다음과 같은 입출력 스트림이 있다. 

|입력 스트림|출력스트림|대상의 종류|
|--:|--:|--:|
|FileInputStream|FileOutputStream|파일|
|ByteArrayInputStream|ByteArrayOutputStream|메모리(byte배열)|
|PipedInputStream|PipedOutputStream|프로세스(프로세스간 통신)|
|AudioInputStream|AudioOutputStream|오디오 장치|

이 중에서 어떠한 대상에 대해서 작업을 할 것인지, 입력을 할 것인지 출력을 할 것인지에 따라 해당 스트림을 선택해서 사용하면 된다. 
이들은 모두 InputStream과 OutputStream의 자손으로, 각각 읽고 쓰는데 필요한 추상 메소드를 자신에 맞게 구현해놓았다. 

자바에서는 java.io 패키지를 통해서 많은 종류의 입출력관련 클래스들을 제공하고 있으며, 입출력을 처리할 수 있는 **표준화된 방법**을 제공함으로써 입출력의 대상이 달라져도 동일한 방법으로 입출력이 가능하기 때문에 프로그래밍을 하기에 편리하다. 

|InputStream|OutputStream|
|--:|--:|
|abstract int read()|abstract void write(int b)|
|int read(byte[] b)|void write(byte[] b)|
|int read(byte[] b, int off, int len)|void write(byte b[], int off, int len)|

이 표에있는 메소드의 사용법만 알고 있어도 데이터를 읽고 쓰는 것은 입출력 대상의 종류에 관계없이 아주 간단해진다.  

InputStream의 read()와 OutputStream의 write(int b)는 입출력의 대상에 따라 읽고 쓰는 방법이 다르기때문에 각 상황에 맞게 구현하라는 의미에서 추상 메소드로 정의되어 있다. 
나머지 메소드들은 추상 메소드가 아니니 바로 사용하면 된다고 생각할 수 있지만, 내부적으로 read()와 write(int b)를 이용해서 구현하고 있으므로 read()와 write(int b)가 구현되어 있지 않으면 이들은 아무런 의미가 없다. 

#### 보조 스트림 
보조 스트림은 실제 데이터를 주고 받는 스트림이 아니기 때문에 데이터를 입출력할 수 있는 기능은 없지만 스트림의 기능을 향상시키거나 새로운 기능을 추가할 수 있다
그래서 보조 스트림만으로는 입출력을 처리할 수 없고, 스트림을 먼저 생성한 다음에 이를 이용해서 보조스트림을 생성해야 한다.

예를 들어 test.txt라는 파일을 읽기 위해 FileInputStream을 사용할 때, 입력 성능을 향상 시키기 위해 버퍼를 사용하는 보조스트림인 BufferedInputStream을 사용하는 코드는 아래와 같다. 

```java
FileInputStream fis = new FileInputStream("test.txt");
BufferedInputStream bis = new BufferedInputStream(fis);
bis.read();
```

코드 상으로 보면 BufferedInputStream이 입력 기능을 수행하는 것처럼 보이지만, 실제 입력기능은 연결된 FileInputStream이 수행하고, 보조 스트림 BufferedInputStream은 버퍼만 제공한다.
버퍼를 사용한 입출력과 사용하지 않은 입출력 간의 성능차이는 상당하기 때문에 대부분의 경우에 버퍼를 이용한 보조스트림을 사용한다.

|입력|출력|설명|
|--:|--:|--:|
|FilterInputStream|FilterOutputStream|필터를 이용한 입출력 처리|
|BufferedInputStream|BufferedOutputStream|버퍼를 이용한 입출력 성능향상|
|DataInputStream|DataOutputStream|int, float와 같은 기본형 단위(primitive type)로<br> 데이터를 처리하는 기능|
|SequenceInputStream|없음|두 개의 스트림을 하나로 연결|
|LineNumberInputStream|없음|읽어 온 데이터의 라인 번호를 카운트<br>JDK1.1부터 LineNumberReader로 대체|
|ObjectInputStream|ObjectOutputStream|데이터를 객체단위로 읽고 쓰는데 사용<br>주로 파일을 이용한 객체 직렬화와 관련됨|
|없음|PrintStream|버퍼를 이용해 추가적인 print관련 기능<br>(print,printf,println메소드)|
|PushbackInputStream|없음|버퍼를 이용해서 읽어 온 데이터를 다시 되돌리는 기능 <br> (unread,push back to buffer)|

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

### 바이트기반 스트림
#### InputStream과 OutputStream

앞서 언급했다시피 InputStream과 OutputStream은 모든 바이트기반의 스트림의 조상이며 다음과 같은 메소드가 선언되어 있다.

InputStream의 메소드 

|메소드명|설명|
|--:|--:|
|int available()|스트림으로붜 읽어올 수 있는 데이터의 크기를 반환|
|void close()|스트림을 닫음으로써 자원을 반환|
|void mark(int readlimit)|현재위치를 표시해놓는다. 후에 reset()에 의해 표시해놓은 위치로<br> 돌아갈 수 있고 readlimit은 되돌아갈 수 있는 byte의 수|
|boolean markSupported()|mark()와 reset()을 지원하는지 알려줌|
|abstract int read()|1byte를 읽어온다. 읽어올 데이터가 없으면 -1을 반환한다.|
|int read(byte[] b)|배열 b의 크기만큼 읽어서 배열을 채우고 읽어 온 데이터의 수를 반환|
|int read(byte[] b,<br>int off, int len)|최대 len개의 byte를 읽어서 지정된 위치(off)부터 저장한다.|
|void reset()|스트림의 위치를 마지막으로 mark()가 호출된 위치로 되돌린다. |
|long skip(long n)|스트림에서 주어진 길이(n)만큼을 건너뛴다.|


OutputStream의 메소드
|메소드명|설명|
|--:|--:|
|void close()|입력소스를 닫음으로써 자원을 반환|
|void flush()|스트림의 버퍼에 있는 모든 내용을 출력소스에 write|
|abstract void write(int b)|주어진 값을 출력소스에 write|
|void write(byte[] b)|배열 b에 저장된 모든 내용을 출력소스에 write|
|void write(byte[] b,<br>int off,int len)|배열 b에 저장된 내용 중에서 <br>off번째부터 len개만큼 읽어서 출력소스에 write|

프로그램이 종료될 때, 사용하고 닫지 않은 스트림을 JVM이 자동적으로 닫아 주기는 하지만, 스트림을 사용해서 모든 작업을 마치도 난 후에는 close()를 호출해서 반드시 닫아주어야 한다. 
그러나 ByteArrayInputStream과 같이 메모리를 사용하는 스트림과 System.in, Sytem.out과 같은 표준 입출력 스트림은 닫아주지 않아도 된다.

#### ByteArrayInputStream과 ByteArrayOutputStream
ByteArrayInputStream과 ByteArrayOutputStream은 메모리, 즉 바이트배열에 데이터를 입출력하는데 사용되는 스트림이다. 주로 다른 곳에 입출력하기 전에 데이터를 임시로 바이트배열에 담아서 변환 등의 작업을 하는데 사용된다.

```java
byte[] inSource = {0,1,2,3,4,5};
byte[] outSource = null;

ByteArrayInputStream input = new ByteArrayInputStream(inSource);
ByteArrayOutputStream output = new ByteArrayOutputStream();

int data = 0;
while((data = input.read()) != -1){
    output.write(data);
}

outSource = output.toByteArray();
System.out.println("Input Source = " + Arrays.toString(inSource));
System.out.println("Output Source = " + Arrays.toString(outSource));
```
```console
Input Source = [0, 1, 2, 3, 4, 5]
Output Source = [0, 1, 2, 3, 4, 5]
```

바이트배열은 사용하는자원이 메모리 밖에 없으므로 가비지 컬렉터에 의해 자동적으로 자원을 반환하기 때문에 close()를 이용해서 스트림을 닫지 않아도 된다. read()와 write(int b)를 사용하기 때문에 한 번에 1byte만 읽고 쓰므로 작업 효율은 떨어진다. 
아래 예제는 배열을 사용해서 입출력이 보다 효율적으로 이루어지도록 했다.

```java
byte[] inSource = {0,1,2,3,4,5,6,7,8,9};
byte[] outSource = null;
byte[] temp = new byte[4];

ByteArrayInputStream input = new ByteArrayInputStream(inSource);
ByteArrayOutputStream output = new ByteArrayOutputStream();

System.out.println("Input Source = " + Arrays.toString(inSource));

try{
    while(input.available() > 0){
        input.read(temp);
        output.write(temp);
        outSource = output.toByteArray();
        System.out.println("        temp = " + Arrays.toString(temp));
        System.out.println("Output Source = " + Arrays.toString(outSource));
    }
}catch (IOException e){}
```
```console
Input Source = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
        temp = [0, 1, 2, 3]
Output Source = [0, 1, 2, 3]
        temp = [4, 5, 6, 7]
Output Source = [0, 1, 2, 3, 4, 5, 6, 7]
        temp = [8, 9, 6, 7]
Output Source = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 6, 7]
```

read()나 write()에서 IOException을 발생시킬 수 있기 때문에 try-catch문으로 감싸주었다. available()은 blocking없이 읽어올 수 있는 바이트의 수를 반환한다.
>blocking이란, 데이터를 읽어 올 때 데이터를 기다리기 위해 멈춰있는 것을 말한다. 예를 들어 사용자가 데이터를 입력하기 전까지 기다리고 있을 때 블락킹 상태에 있다고 한다. 

그런데 우리가 예상한 것과 다른 결과가 나왔다. Input Source와 동일하게 0~9까지 숫자가 Output Source에 복사되었을 것이라고 생각했는데 그 뒤에 6,7이 중복해서 나타난다. 

그 이유는 마지막에 읽은 배열의 9번째와 10번째 요소값인 8과 9만 출력해야하는데 temp에 남아있던 6,7까지 출력했기 때문이다. 

보다 나은 성능을 위해서 temp에 담긴 내용을 지우고 쓰는 것이 아니라 기존의 내용 위에 덮어쓴다. 그래서 temp의 [4,5,6,7] 위에 8,9를 읽으니 [8,9,6,7]이 된 것이다.

원하는 코드를 얻기 위해서는 아래의 코드와 같이 수정해야 한다. 
```java
while(input.available() > 0){
    int len = input.read(temp);
    output.write(temp,0,len);
}
```
```console
Input Source = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
        temp = [0, 1, 2, 3]
Output Source = [0, 1, 2, 3]
        temp = [4, 5, 6, 7]
Output Source = [0, 1, 2, 3, 4, 5, 6, 7]
        temp = [8, 9, 6, 7]
Output Source = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
코드를 수정하여 배열의 내용 전체가 아닌, 읽은 만큼만 출력하도록 하니 우리가 원하는 결과를 얻은 것을 확인할 수 있다.

#### FileInputStream과 FileOutputStream
FileInputStream과 FileOutputStream은 파일에 입출력을 하기 위한 스트림이다. 실제로 가장 많이 사용되는 스트림 중에 하나이다. 

|생성자|설명|
|--:|--:|
|FileInputStream(String name)|지정된 파일이름을 가진 파일과 연결된 FileInputStream 생성|
|FileInputStream(File file)|File 인스턴스로 파일과 연결된 FileInputStream 생성|
|FileInputStream(FileDescriptor fdObj)|파일 디스크립터로 FileInputStream 생성|
|FileOutputStream(String name)|지정된 파일이름을 가진 파일과 연결된 FileOutputStream 생성|
|FileOutputStream(String name, boolean append)|지정된 파일이름을 가진 파일과 연결된 FileOutputStream 생성하는데 <br> append가 true이면 출력 시 기존의 파일 내용의 마지막에 덧붙이고 false면 기존의 파일 내용을 덮어쓴다.|
|FileOutputStream(File file)|File 인스턴스로 파일과 연결된 FileOutputStream 생성|
|FileOutputStream(File file, boolean append)|파일명대신 File 인스턴스로 전달하는 점을 제외하고 <br>FileOutputStream(String name, boolean append)과 같다.|
|FileOutputStream(FileDescriptor fdObj)|파일디스크립터로 FileOutputStream을 생성한다.|

```java
public class IOTest {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream(args[0]);
        int data = 0;

        while ((data = fis.read()) != -1) {
            char c = (char)data;
            System.out.print(c);
        }
    }
}
```
```console
yeojiwon@yeojiwon-ui-MacBookAir week13 % java IOTest.java IOTest.java
package com.example.javastudy.week13;

import java.io.*;
import java.util.Arrays;

public class IOTest {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream(args[0]);
        int data = 0;

        while ((data = fis.read()) != -1) {
            char c = (char)data;
            System.out.print(c);
        }
    }
}
```
 
커맨드라인으로부터 입력받은 파일의 내용을 읽어서 그대로 화면에 출력하는 간단한 예제이다. read()의 반환값이 int형(4byte)이지만, -1을 제외하고는 0~255(1byte)의 정수값이기때문에 char형(2byte)으로 변환한다 해도 손실되는 값은 없다.

read()가 한 번에 1byte씩 데이터를 읽긴 하지만, 데이터의 범위가 0~255사이의 정수값이고, -1값도 필요하기 때문에 다소 크긴 하지만 정수형 중에서는 연산이 가장 효율적이고 빠른 int형 값을 반환하도록 한 것이다.

만약 텍스트파일을 다루는 경우에는 FileInputStream/FileOutputStream보다 문자기반의 스트림인 FileReader/FileWriter를 사용하는 것이 좋다.

### 바이트기반의 보조스트림
#### FilterInputStream과 FilterOutputStream
FilterInputStream과 FilterOutputStream은 InputStream/OutputStream의 자손이면서 모든 보조스트림의 조상이다. 보조스트림은 자체적으로 입출력을 할 수 없기때문에 기반스트림을 필요로한다. 다음은 FilterInputStream과 FilterOutputStream의 생성자이다. 

```java
protected FilterInputStream(InputStream in)
public FilterOutputStream(OutputStream out)
```

FilterInputStream과 FilterOutputStream의 모든 메소드는 단순히 기반스트림의 메소드를 그대로 호출할 뿐 이 자체로는 아무런 일도 하지 않는다. 
또한 FilterInputStream의 생성자는 protected이므로 바로 인스턴스를 생성해서 사용할 수 없고 상속을 통해서 오버라이딩되어야 한다. FilterInputStream과 FilterOutputStream을 상속받아서 기반스트림에 보조 기능을 추가한 보조스트림 클래스는 다음과 같다.  

FilterInputStream의 자손 - BufferedInputStream, DataInputStream, PushbackInputStream 등

FilterOutputStream의 자손 - BufferedOutputStream, DataOutputStream, PrintStream 등

#### BufferedInputStream과 BufferedOutputStream
BufferedInputStream과 BufferedOutputStream은 스트림의 입출력 효율을 높이기 위해 버퍼를 사용하는 보조스트림이다. 한 바이트씩 입출력하는 것보다 버퍼(바이트 배열)를 이용해서 한 번에 여러 바이트를 입출력하는 것이 빠르기때문에 대부분의 입출력작업에 사용된다.

|생성자|설명|
|--:|--:|
|BufferedInputStream(InputStream in, int size)| 주어진 InputStream인스턴스를 입력소스로 하며, 지정된 크기(byte 단위)의 버퍼를 갖는 BufferedInputStream인스턴스를 생성한다.|
|BufferedInputStream(InputStream in)|주어진 InputStream인스턴스를 입력소스로 하며, 기본적으로 8192byte 크기의 버퍼를 갖는다.|

프로그램에서 입력소스로부터 데이터를 읽기 위해 처음으로 read메소드를 호출하면, BufferedInputStream은 입력소스로부터 버퍼의 크기만큼 데이터를 읽어서 자신의 내부 버퍼에 저장한다. 그 후 프로그램에서는 BufferedInputStream의 버퍼에 저장된 데이터를 읽는다. 

외부의 입력소스로부터 읽는 것보다 내부의 버퍼로부터 읽는 것이 훨씬 빠르기때문에 그만큼 작업의 효율이 높아진다.

프로그램에서 버퍼에 저장된 모든 데이터를 다 읽고 그 다음 데이터를 읽기위해 read 메소드가 호출되면 BufferedInputStream은 입력소스로부터 다시 버퍼크기만큼의 데이터를 읽어다 버퍼에 저장해 놓는다.

|메소드/생성자|설명|
|--:|--:|
|BufferedOutputStream(OutputStream out,int size)|주어진 OutputStream인스턴스를 출력소스로하고 지정된 크기(byte 단위)의 버퍼를 갖는 BufferedOutputStream인스턴스를 생성한다.|
|BufferedOutputStream(OutputStream out)|주어진 OutputStream인스턴스를  출력소스로 하며, 기본적으로 8192byte 크기의 버퍼를 갖는다.|
|flush()|버퍼의 모든 내용을 출력소스에 출력한 다음 버퍼를 비운다.|
|close()|flush()를 호출해서 버퍼를 비운 뒤 BufferedOutputStream가 사용하던 모든 자워을 반환한다.|

BufferedOutputStream 역시 버퍼를 이용해서 출력소스와 작업을 하는데, 입력 소스로부터 데이터를 읽을 떄와는 반대로 프로그램에서 write메소드를 이용한 출력이 BufferedOutputStream의 버퍼에 저장된다. 버퍼가 가득 차면, 그 때 버퍼의 모든 내용을 출력소스에 출력한다. 그리고 버퍼를 비우고 다시 프로그램으로부터 출력을 저장할 준비를 한다.

버퍼가 가득 찼을 때만 출력소스에 출력하기때문에 마지막 출력부분이 출력소스에 쓰이지 못하고 BufferedOutputStream의 버퍼에 남아있는 채로 프로그램이 종료될 수 있다는 점을 주의해야한다.
그래서 모든 출력 작업을 마치면 BufferedOutputStream에 flush()나 close()를 호출해서 마지막에 버퍼에 남은 내용이 출력소스에 출력되도록 해야한다.
>BufferedOutputStream의 close()는 내부적으로 flush()를 호출한 뒤 BufferedOutputStream의 인스턴스의 참조변수에 null을 지정함으로써 사용하던 자원들이 반환되게 한다.

그리고 보조스트림의 close()는 내부적으로 기반스트림의 close()를 호출하기때문에 보조스트림의 close()만 호출하면 된다. 


#### DataInputStream과 DataOutputStream
DataInputStream과 DataOutputStream도 각각 FilterInputStream/FilterOutputStream의 자손이며 DataInputStream은 DataInput 인터페이스를, DataOutputStream은 DataOutput 인터페이스를 구현하기때문에 데이터를 읽고 쓰는데 있어서 byte 단위가 아닌 8가지 기본 자료형의 단위로 읽고 쓸 수 있다는 장점이 있다.

#### SequenceInputStream
SequenceInputStream은 여러 개의 입력스트림을 연속적으로 연결해서 하나의 스트림으로부터 데이터를 읽는 것과 같이 처리할 수 있도록 도와준다. SequenceInputStream의 생성자를 제외하고 나머지 작업은 다른 입력스트림과 다르지않다. 큰 파일을 여러 개의 작은 파일로 나눴다가 하나의 파일로 합치는 것과 같은 작업을 수행할 떄 사용하면 좋을 것이다.
>SequenceInputStream은 다른 보조스트림들과 달리 FilterInputStream이 아닌 InputStream을 바로 상속받아서 구현하였다.

|생성자|설명|
|--:|--:|
|SequenceInputStream(Enumeration e)|Enumeration에 저장된 순서대로 입력 스트림을 하나의 스트림으로 연결한다.|
|SequenceInputStream(InputStream s1, InputStream s2)|두 개의 입력 스트림을 하나로 연결한다.|

#### PrintStream
PrintStream은 데이터를 기반스트림에 다양한 형태로 출력할 수 있는 print, println, printf와 같은 메소드를 오버로딩하여 제공한다.

PrintStream은 데이터를 적절한 문자로 출력하는 것이기 때문에 문자기반 스트림의 역할을 수행한다. 그래서 JDK 1.1부터 PrintStream보다 향상된 기능의 문자기반 스트림인 PrintWriter가 추가되었으나 그 동안 애무 빈번히 사용되던 System.out이 PrintStream이다보니 둘 다 사용할 수 밖에 없게 되었다. 

따라서 PrintStream과 PrintWriter는 거의 같은 기능을 가지고 있지만 PrintWriter가 PrintStream에 비해 다양한 언어의 문자를 처리하는데 적합하기 때문에 가능하면 PrintWriter를 사용하는 것이 좋다.
> System.out, System.err이 PrintStream을 사용한다.

print()나 println()을 이용해서 출력하는 중에 PrintStream의 기반스트림에서 IOException이 발생하면 checkError()를 통해서 인지할 수 있다. println()이나 print()는 예외를 던지지 않고 내부에서 처리하도록 정의하였는데, 그 이유는 println()처럼 매우 자주 사용되는 메소드에 예외를 던지도록 정의하면 println()을 사용하는 모든 곳에 try-catch문을 사용해야 하기 때문이다.

printf()는 JDK1.5부터 추가된 것으로, C언어와 같이 편리하고 형식화된 출력을 지원하게 되었다. 

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