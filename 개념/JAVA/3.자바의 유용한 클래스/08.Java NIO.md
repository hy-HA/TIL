# Java NIO

Java NIO는 Java I/O의 제한 사항을 극복하기 위해 설계되었으며, Non-blocking I/O, 버퍼 지향성, 셀렉터를 이용한 다중 채널 관리 등과 같은 고급 기능을 제공한다. 이러한 기능들은 고성능 네트워킹 애플리케이션을 개발하는데 유용하게 활용될 수 있다.

## Java NIO란?

- NIO (New Input/Output)는 Java 1.4버전에서 도입된 입력/출력 방식으로, 표준 I/O와는 다르게 비동기식 및 블록 모드를 지원한다.
- Java NIO는 Java I/O가 제공하는 블로킹 방식의 API를 보완하고, 기존 Java I/O의 효율적이지 않은 부분을 개선한 기능을 제공한다.
- Java NIO는 빠르고 비동기적인 I/O 처리를 가능하게 하여, 대량의 데이터를 처리하는 데 있어서 효율성을 극대화한다.


## Java IO가 느린 이유

![](https://hackmd.io/_uploads/H1BfchzYn.png)

- Java IO를 통해 File Read 과정
    1. 프로세스(JVM)가 파일을 읽기 위해 커널에 명령을 전달
    2. 커널은 시스템 콜(read())을 사용
    3. 디스크 컨트롤러가 물리적 디스크로부터 파일을 읽어옴
    4. DMA를 이용하여 커널 버퍼로 복사
    5. 프로세스(JVM) 내부 버퍼로 복사 (복사 과정이 중복되어 발생)
- 모든 IO는 반드시 커널 영역을 거치게 되는데, 커널 내부 버퍼에 직접 접근하지 않는 이상 CPU 연산, 불필요한 GC, 스레드 블록킹이 발생하기 때문에 IO가 느려지게 된다


## NIO의 주요 구성요소

### 채널(Channel)

![](https://hackmd.io/_uploads/Hy2E2m6Pn.png)

- Java NIO에서 데이터의 입출력을 담당하는 역할은 채널이다.
- 채널은 기본적으로 Stream과 유사하지만, 차이점이 있다면 Stream은 단방향이지만 채널은 양방향으로 데이터 전송이 가능하다는 점이다.
- 채널은 블로킹 방식뿐 아니라 논블로킹 방식도 지원한다.
- 예시: `FileChannel`, `DatagramChannel`, `SocketChannel`, `ServerSocketChannel` 등

### 버퍼(Buffer)

- 데이터를 운반하는 데 사용되는 컨테이너.
- 버퍼는 데이터를 담을 수 있는 고정된 크기의 공간이며, 데이터의 읽기, 쓰기 작업을 버퍼를 통해 진행한다.
- 채널을 통해 데이터를 읽거나 쓸 때, 데이터는 버퍼를 경유하게 된다.
- 버퍼의 종류로는 `ByteBuffer`, `CharBuffer`, `DoubleBuffer`, `FloatBuffer`, `IntBuffer`, `LongBuffer`, `ShortBuffer` 등이 있다.

### 셀렉터(Selector)

![](https://hackmd.io/_uploads/BJg1UHaPh.png)

- Java NIO의 핵심 컴포넌트 중 하나로, 여러 개의 채널을 다루는데 사용된다.
- 셀렉터를 이용하면 한 개의 스레드만으로 여러 채널을 동시에 처리할 수 있게 되므로, 멀티 스레드 환경에서의 컨텍스트 스위칭 오버헤드를 줄일 수 있다.
- 셀렉터는 등록된 채널 중 I/O 이벤트가 발생한 채널을 찾아내는 역할을 한다.


## NIO의 주요 클래스

| 클래스 명 | 설명 |
| --------- | ---- |
| `java.nio.Buffer` | 버퍼는 기본 데이터 유형을 저장하며, 모든 데이터는 버퍼를 통해 처리된다. `ByteBuffer`, `CharBuffer`, `DoubleBuffer`, `FloatBuffer`, `IntBuffer`, `LongBuffer`, `ShortBuffer`와 같은 다양한 버퍼 클래스가 있다. |
| `java.nio.Channel` | 채널은 버퍼로 데이터를 전송하는 미디어를 제공한다. `FileChannel`, `DatagramChannel`, `SocketChannel`, `ServerSocketChannel` 등의 클래스가 있으며, 각각은 파일, UDP, TCP 클라이언트, TCP 서버에 사용된다. |
| `java.nio.Selector` | Selector는 비동기 작업을 위해 여러 채널을 관리하는데 사용된다. |
| `java.nio.charset.Charset` | Charset 클래스는 바이트와 문자열 간의 변환을 처리한다. |
| `java.nio.file.Path` | Path 클래스는 파일 및 디렉토리 경로에 대한 인터페이스를 제공한다. |
| `java.nio.file.Files` | Files 클래스는 파일 및 디렉토리를 생성, 삭제, 읽기, 쓰기 등을 수행하는 메소드를 제공한다. |
| `java.nio.file.FileSystem` | FileSystem 클래스는 파일 시스템을 대표하며, FileSystems 클래스를 통해 접근할 수 있다. |

## Java NIO를 이용한 코드 샘플

1. **`FileChannel`을 이용하여 파일 쓰기**

    ```java
    import java.io.RandomAccessFile;
    import java.nio.ByteBuffer;
    import java.nio.channels.FileChannel;

    public class NIOFileWriteExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 읽고 쓸 수 있는 상태로 열고, 이를 RandomAccessFile 객체에 할당한다.
                RandomAccessFile file = new RandomAccessFile("example.txt", "rw");

                // RandomAccessFile에서 FileChannel을 가져온다.
                // FileChannel을 통해 파일에 대한 입출력 작업을 수행한다.
                FileChannel fileChannel = file.getChannel();

                // 파일에 쓸 문자열을 정의한다.
                String text = "Hello, Java NIO!";

                // 문자열을 바이트 배열로 변환한다.
                byte[] byteArray = text.getBytes();

                // 변환된 바이트 배열을 담는 ByteBuffer를 생성한다.
                ByteBuffer buffer = ByteBuffer.wrap(byteArray);

                // fileChannel.write(buffer)를 통해 buffer에 있는 내용을 파일에 쓴다.
                fileChannel.write(buffer);

                // 파일을 닫는다. 이는 시스템 리소스를 반환하기 위해 필요하다.
                file.close();
            } catch (IOException e) {
                // IOException이 발생한 경우 스택 추적을 출력한다.
                e.printStackTrace();
            }
        }
    }

    ```

2. **`FileChannel`을 이용하여 파일 읽기**

    ```java
    import java.io.RandomAccessFile;
    import java.nio.ByteBuffer;
    import java.nio.channels.FileChannel;

    public class NIOFileReadExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 읽기 모드로 열고, 이를 RandomAccessFile 객체에 할당한다.
                RandomAccessFile file = new RandomAccessFile("example.txt", "r");

                // RandomAccessFile에서 FileChannel을 가져온다.
                // FileChannel을 통해 파일에 대한 입출력 작업을 수행한다.
                FileChannel fileChannel = file.getChannel();

                // 읽어온 데이터를 저장할 ByteBuffer를 생성한다.
                // 여기서는 최대 1024바이트의 데이터를 저장할 수 있는 버퍼를 생성한다.
                ByteBuffer buffer = ByteBuffer.allocate(1024);

                // 파일에서 데이터를 읽어 buffer에 저장한다.
                // fileChannel.read(buffer)는 읽은 바이트 수를 반환하며, 파일의 끝에 도달하면 -1을 반환한다.
                while(fileChannel.read(buffer) > 0) {
                    // buffer를 flip()하여 읽기 모드로 전환한다.
                    buffer.flip();

                    // 버퍼에 남아있는 데이터가 있는 동안 루프를 돈다.
                    while(buffer.hasRemaining()) {
                        // 버퍼에서 한 바이트를 가져와 문자로 변환하고 출력한다.
                        System.out.print((char) buffer.get());
                    }

                    // 버퍼를 clear()하여 쓰기 모드로 전환하고 모든 데이터를 지운다.
                    buffer.clear();
                }

                // 파일을 닫는다. 이는 시스템 리소스를 반환하기 위해 필요하다.
                file.close();
            } catch (IOException e) {
                // IOException이 발생한 경우 스택 추적을 출력한다.
                e.printStackTrace();
            }
        }
    }

    ```

3. **`AsynchronousFileChannel`을 이용한 파일 쓰기**

    ``` java
    import java.nio.ByteBuffer;
    import java.nio.channels.AsynchronousFileChannel;
    import java.nio.file.Paths;
    import java.nio.file.StandardOpenOption;
    import java.util.concurrent.Future;

    public class AsynchronousFileWriteExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 비동기 쓰기 모드로 열고, 이를 AsynchronousFileChannel 객체에 할당한다.
                AsynchronousFileChannel asyncFileChannel = AsynchronousFileChannel.open(
                        Paths.get("example.txt"),
                        StandardOpenOption.WRITE,
                        StandardOpenOption.CREATE);

                // 파일에 쓸 문자열을 정의하고, 이를 바이트 배열로 변환한다.
                String text = "Hello, Java Asynchronous IO!";
                byte[] byteText = text.getBytes();

                // 변환된 바이트 배열을 담는 ByteBuffer를 생성한다.
                ByteBuffer buffer = ByteBuffer.wrap(byteText);

                // 비동기로 buffer에 있는 내용을 파일에 쓴다.
                // 이 메소드는 즉시 Future 객체를 반환하며, 이 객체를 통해 비동기 작업의 상태와 결과를 얻을 수 있다.
                Future<Integer> result = asyncFileChannel.write(buffer, 0);

                // 비동기 작업이 완료될 때까지 기다린다.
                // 작업이 완료될 동안 메인 스레드는 다른 작업을 계속 수행할 수 있다.
                while (!result.isDone()) {
                    // 비동기 작업이 완료될 때까지 대기...
                }

                // 비동기 작업이 완료되면 작성된 바이트 수를 출력한다.
                System.out.println("Written: " + result.get() + " bytes to the file");

                // 파일을 닫는다. 이는 시스템 리소스를 반환하기 위해 필요하다.
                asyncFileChannel.close();

            } catch (Exception e) {
                // 예외가 발생하면 스택 추적을 출력한다.
                e.printStackTrace();
            }
        }
    }
    ```

4. **`AsynchronousFileChannel`을 이용한 파일 읽기**

    ``` java
    import java.nio.ByteBuffer;
    import java.nio.channels.AsynchronousFileChannel;
    import java.nio.file.Paths;
    import java.nio.file.StandardOpenOption;
    import java.util.concurrent.Future;

    public class AsynchronousFileReadExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 비동기 읽기 모드로 열고, 이를 AsynchronousFileChannel 객체에 할당한다.
                AsynchronousFileChannel asyncFileChannel = AsynchronousFileChannel.open(
                        Paths.get("example.txt"),
                        StandardOpenOption.READ);

                // 읽어온 데이터를 저장할 ByteBuffer를 생성한다. 여기서는 최대 1024바이트의 데이터를 저장할 수 있는 버퍼를 생성한다.
                ByteBuffer buffer = ByteBuffer.allocate(1024);

                // 비동기로 파일에서 buffer에 데이터를 읽어온다.
                // 이 메소드는 즉시 Future 객체를 반환하며, 이 객체를 통해 비동기 작업의 상태와 결과를 얻을 수 있다.
                Future<Integer> result = asyncFileChannel.read(buffer, 0);

                // 비동기 작업이 완료될 때까지 기다린다.
                // 작업이 완료될 동안 메인 스레드는 다른 작업을 계속 수행할 수 있다.
                while (!result.isDone()) {
                    // 비동기 작업이 완료될 때까지 대기...
                }

                // 비동기 작업이 완료되면 읽은 바이트 수를 출력한다.
                System.out.println("Read: " + result.get() + " bytes from the file");

                buffer.flip();
                while(buffer.hasRemaining()) {
                    System.out.print((char) buffer.get());
                }

                // 파일을 닫는다. 이는 시스템 리소스를 반환하기 위해 필요하다.
                asyncFileChannel.close();

            } catch (Exception e) {
                // 예외가 발생하면 스택 추적을 출력한다.
                e.printStackTrace();
            }
        }
    }

    ```

---

# NIO와 IO의 차이점

|java.io|java.nio|
|---|---|
|스트림|채널|
|필터 스트림을 이용한 버퍼 사용|버퍼 사용을 기본으로 함|
|동기 방식|동기/비동기 둘 다 지원|
|Blocking 방식|Blocking/Non-Blocking 둘 다 지원|

**입출력 방식**
- I/O Stream은 단방향 통신만 가능한 반면, NIO Channel은 양방향 통신이 가능하다.

**블록킹 방식**
- 기존의 I/O는 Blocking I/O를 지원하는 반면에, NIO는 Blocking & Non-blocking I/O 모두 지원한다.

**버퍼 사용 여부**
  - Java I/O에서는 버퍼가 애플리케이션 데이터를 운반하는 데 필수적인 요소가 아니지만, NIO에서는 모든 데이터가 버퍼를 통해 이동한다.
  - 이에 따라 NIO에서는 데이터를 처리할 때 버퍼의 용량과 위치 등을 더욱 세밀하게 제어할 수 있다.

**Selector**
  - NIO에서는 Selector라는 개념을 도입했다. Selector를 사용하면 하나의 스레드에서 여러 Channel을 효율적으로 처리할 수 있다.
  - 이를 통해 멀티스레드 환경에서 발생할 수 있는 동시성 문제를 효과적으로 제어하고, 자원을 효율적으로 사용할 수 있다. 이러한 기능은 기존의 I/O에서는 제공되지 않았다.

## IO / NIO 선택

- **단순성과 코드 가독성**
  - `java.io`는 `java.nio`보다 코드가 더 단순하고 직관적이다. 스트림 기반의 모델이 읽기와 쓰기를 위한 API를 더욱 단순화시킨다. 따라서, 간단한 I/O 작업이 필요하거나 특별한 성능 요구사항이 없는 경우 `java.io`를 사용하는 것이 좋다.
- **비동기 I/O 작업**
  - `java.nio`는 비동기 I/O 작업을 지원하므로, 단일 스레드에서 여러 채널의 I/O 작업을 동시에 처리해야 하는 상황에 적합하다. 따라서, 성능이 중요하거나 스케일링이 필요한 네트워크 서버와 같은 고성능 애플리케이션을 개발하는 경우 `java.nio`를 사용하는 것이 좋다.
- **대용량 데이터 처리**
  - `java.nio`의 버퍼와 채널은 대용량 데이터를 효율적으로 처리하는 데 유용하다. 디스크 I/O 작업을 최소화하고 시스템 리소스를 절약하려는 경우 `java.nio`를 사용하는 것이 좋다.
- **선택적 블로킹 / 논블로킹 I/O**
  - 일부 애플리케이션은 블로킹 I/O와 논블로킹 I/O 중 어느 것이든 선택적으로 사용할 수 있는 유연성이 필요할 수 있다. 이런 경우에는 `java.nio`가 더 적합하다.