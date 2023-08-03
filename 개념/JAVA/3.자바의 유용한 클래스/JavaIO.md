# Java IO

## IO란?

- 입력(Input)과 출력(Output)을 줄여 부르는 말
- 두 대상 간의 데이터를 주고 받는 것
- 예) 키보드를 통한 사용자 입력이나 화면에 결과를 출력하는 것 or 하드디스크의 파일을 읽거나 저장하는 것

### 어플리케이션이 HW에 접근하기까지

![](https://hackmd.io/_uploads/rJqkE8d_2.png)

## Java I/O란?

- Java 프로그램이 데이터를 읽고 쓸 수 있게 해주는 기능.
    - 여기서 '데이터'는 사용자로부터의 입력, 파일, 네트워크 연결 등 다양한 형태를 가질 수 있다.
- Java에서 데이터 입출력을 위한 기능을 제공하는 패키지
- Java는 I/O 작업을 빠르게 하기 위해 스트림 개념을 사용한다.
    - `java.io` 패키지에는 입력 및 출력 작업에 필요한 모든 클래스가 포함되어 있다.
    - 예시: 파일에서 데이터를 읽어오는 작업은 `FileInputStream` 클래스를 사용하고, 콘솔로 데이터를 출력하는 작업은 `System.out.println()` 메서드를 사용한다.
- Java I/O API를 통해 Java에서 파일 처리를 수행할 수 있다

## Java I/O의 구성
- 자바 I/O는 크게 스트림(Stream)과 채널(Channel), 그리고 버퍼(Buffer) 등 세 가지 주요 요소로 구성되어 있다.
- '스트림'은 일련의 데이터를 표현하는 개념으로, 데이터가 프로그램에 들어오는 '입력 스트림'과 프로그램에서 나가는 '출력 스트림'으로 나뉜다.
- '채널'은 스트림과 비슷하지만, 읽기와 쓰기를 동시에 수행할 수 있으며, 비동기 I/O 작업을 지원한다.
- '버퍼'는 데이터를 임시적으로 저장하는 메모리 영역으로, 채널을 통해 데이터를 읽고 쓸 때 사용된다.

## Java I/O의 클래스 계층구조
- 자바 I/O의 기능은 java.io 패키지와 java.nio 패키지에 포함된 다양한 클래스와 인터페이스를 통해 제공된다.
- 이들 클래스는 크게 '스트림 클래스', '리더/라이터 클래스', '채널 클래스', '버퍼 클래스' 등으로 나뉘며, 각각 다른 종류의 데이터와 상황에 따라 사용된다.



## 스트림(stream)

- 데이터를 운반(입출력)하는데 사용되는 연결 통로
- 연속적인 데이터의 흐름을 물(stream)에 비유해서 붙여진 이름
- 하나의 스트림으로 입출력을 동시에 수행할 수 없다.(단방향 통신)
- 입출력을 동시에 수행하려면, 2개의 스트림이 필요하다


![](https://hackmd.io/_uploads/ByLIbNsI3.png)
> 스트림을 통해 데이터를 주고받는 예시

<details>
<summary>Java의 기본 Stream</summary>
<div markdown="1">

- Java에서는 3개의 스트림이 자동으로 생성되며, 이 스트림들은 콘솔에 연결된다.
  - `System.out`: 표준 출력 스트림
  - `System.in`: 표준 입력 스트림
  - `System.err`: 표준 오류 스트림

**`System.out`, `System.err` 예시 코드**
``` java
System.out.println( "간단한 메시지" );  
System.err.println( "오류 메시지" );
```

**`System.in` 예시 코드**
``` java
int i=System.in.read(); //첫 번째 문자의 ASCII 코드를 반환한다.  
System.out.println(( char )i); //문자를 출력한다.  
```

</div>
</details>

---

### 버퍼(Buffer)

![](https://hackmd.io/_uploads/HyD9_Xawn.png)

- 데이터를 운반하는 데 사용되는 컨테이너.
- 버퍼는 데이터를 담을 수 있는 고정된 크기의 공간이며, 데이터의 읽기, 쓰기 작업을 버퍼를 통해 진행한다.
- Stream은 버퍼를 사용하지 않는다.

---


## Java의 I/O 입출력 클래스

- Java의 I/O는 `java.io` 패키지에 클래스가 정의되어 있는 경우가 대부분이다.
- Java의 초기에는 입출력 클래스도 단순했고 단순히 바이트 단위의 입출력만 지원했지만, 현재는 계속적으로 확장되며 문자 단위의 입출력 뿐만 아니라 다양한 기능을 지원하는 클래스들이 생겨났다.

### I/O 클래스의 이름과 의미

- Stream : 바이트 단위로 입출력 수행
- Reader / Writer : 문자 단위로 입출력 수행
- File : 하드디스크의 파일을 사용
- Data : 자바의 원시 자료형을 출력하기 위한 클래스
- Buffered : 시스템의 버퍼를 사용

> **1차 스트림(Primary Stream)**
> - 입/출력 통로를 직접 만드는 클래스
> - 1차 스트림은 데이터 소스나 데이터 목적지에 직접 연결되는 스트림을 의미한다
> - 예시로는 `FileInputStream`, `FileOutputStream`, `FileReader`, `FileWriter` 등이 있다

> **2차 스트림(Secondary Stream)**
> - 1차 스트림을 통해 생성되며, 기존의 통로를 이용하여 새로운 기능을 더하는 클래스
> - 버퍼링, 문자변환, 객체직렬화 등의 보조 스트림을 추가하는 것이 이에 해당한다.
> - 예시로는 `BufferedInputStream`, `ObjectOutputStream`, `InputStreamReader` 등이 있습니다.



### InputStream / OutputStream (바이트 입출력)

![바이트 입출력](https://raw.githubusercontent.com/KangMoo/TIL/master/Java/image/io_class.jpeg)
> `InputStream`, `OutputStream`의 클래스 계층도

- InputStream

  | 클래스                  | 설명                                              | Stream     |
  | ----------------------- | ------------------------------------------------- | ---------- |
  | InputStream             | 바이트 입력 스트림을 위한 추상 클래스             | 2차 스트림 |
  | FileInputStream         | 파일에서 바이트를 읽어들여 바이트 스트림으로 변환 | 1차 스트림 |
  | PipedInputStream        | PipedOutputStream에서 읽어들임                    | 1차 스트림 |
  | FilterInputStream       | 필터 적용 바이트 입력을 위한 추상 클래스          | 2차 스트림 |
  | LineNumberInputStream   | 바이트 입력시 라인 번호를 유지 **(비추천)**       | 2차 스트림 |
  | DataInputStream         | 기본 자료형 데이터를 바이트로 출력                | 2차 스트림 |
  | BufferedInputStream     | 바이트 버퍼 입력                                  | 2차 스트림 |
  | PushbackInputStream     | 읽어들인 바이트를 되돌림 (pushback)               | 2차 스트림 |
  | ByteArrayInputStream    | 바이트 배열에서 읽어들임                          | 1차 스트림 |
  | SequenceInputStream     | 서로 다른 InputStream을 입력받은 순서대로 이어줌  | 2차 스트림 |
  | StringBufferInputStream | 문자열에서 읽어들임 **(비추천)**                  | 1차 스트림 |
  | ObjectInputStream       | 객체로 직렬화된 데이터를 역직렬화하여 읽음        | 2차 스트림 |

- OutputStream

  | 클래스                | 설명                                     | Stream     |
  | --------------------- | ---------------------------------------- | ---------- |
  | OutputStream          | 바이트 출력 스트림을 위한 추상 클래스    | 2차 스트림 |
  | FileOutputStream      | 바이트 스트림을 바이트 파일로 변환       | 1차 스트림 |
  | PipedOutputStream     | PipedOutputStream에 출력                 | 1차 스트림 |
  | FilterOutputStream    | 필터 적용 바이트 출력을 위한 추상 클래스 | 2차 스트림 |
  | DataOutputStream      | 바이트를 기본 자료형으로 출력            | 2차 스트림 |
  | BufferedOutputStream  | 바이트 스트림에 버퍼 출력                | 2차 스트림 |
  | PrintStream           | Stream 값과 객체를 브린트                | 2차 스트림 |
  | ByteArrayOutputStream | 바이트 스트림에 바이트 배열 출력         | 1차 스트림 |
  | ObjectputStream       | 데이터를 객체로 직렬화하여 출력          | 2차 스트림 |



## Reader / Writer (문자 입출력)

![Reader/Writer](https://raw.githubusercontent.com/KangMoo/TIL/master/Java/image/reader_writer_class.jpeg)
> `Reader`,`Writer`의 클래스 계층도

> 문자 입출력에는 문자 Encoding이 관여된다

- Reader

  | 클래스            | 설명                                            | Stream     |
  | ----------------- | ----------------------------------------------- | ---------- |
  | Reader            | 문자 입력 스트림을 위한 추상 클래스           | 2차 스트림 |
  | BufferedReader    | 문자 버퍼 입력, 라인 해석                       | 2차 스트림 |
  | LineNumberReader  | 문자 입력 시 라인 번호를 유지                   | 2차 스트림 |
  | CharArrayReader   | 문자 배열에서 읽어들임                          | 1차 스트림 |
  | InputStreamReader | 바이트 스트림을 문자 스트림으로 변환            | 2차 스트림 |
  | FileReader        | 파일에서 바이트를 읽어들여 문자 스트림으로 변환 | 1차 스트림 |
  | FilterReader      | 필터 적용 문자 이력을 위한 추상 클래스          | 2차 스트림 |
  | PushbackReader    | 읽어들인 문자를 되돌림                          | 2차 스트림 |
  | PipedReader       | PipedWriter에서 읽어들임                        | 1차 스트림 |
  | StringReader      | 문자열에서 읽어들임                             | 1차 스트림 |

- Writer

  | 클래스             | 설명                                   | Stream     |
  | ------------------ | -------------------------------------- | ---------- |
  | Writer             | 문자 출력 스트림을 위한 추상 클래스    | 2차 스트림 |
  | BufferdWriter      | 문자 스트림에 버퍼 출력, 줄바꿈 사용   | 1차 스트림 |
  | CharArrayWriter    | 문자 스트림에 문자배열 출력            | 1차 스트림 |
  | OutputStreamWriter | 문자 스트림을 바이트 스트림으로 변환   | 2차 스트림 |
  | FileWriter         | 문자 스트림을 바이트 파일로 변환       | 1차 스트림 |
  | FilterWriter       | 필터 적용 문자 출력을 위한 추상 클래스 | 2차 스트림 |
  | PipedWriter        | PipedReader에 출력                     | 1차 스트림 |
  | StringWriter       | 문자열 출력                            | 1차 스트림 |
  | PrintWriter        | Writer값과 객체를 프린트               | 2차 스트림 |



## 그 외의 `java.io`클래스

| 클래스                 | 설명                                                         |
| ---------------------- | ------------------------------------------------------------ |
| console                | 명령행에서 쉽게 입력받고 정형화된 출력을 명령행에 쉽게 출력 가능 |
| File                   | 파일 객체를 생성                                             |
| FileDescriptor         | 물리적 파일에 대한 현재의 연결을 나타내기 위한 클래스        |
| FilePermission         | 파일 및 디렉토리에 엑세스 권한을 관리하는 클래스             |
| RandomAccessFile       | 랜덤 액세스 파일로부터 읽기와 쓰기가 동시에 이뤄질 수 있다   |
| SerializablePermission | 직렬화 가능 액세스 권한을 위한 클래스                        |
| StreamTokenizer        | 입력 스트림에서 토큰을 추출하며, 한 번에 하나의 토큰을 읽어들인다 |



## Java I/O를 이용 패키지를한 코드 샘플

1. **`FileOutputStream`을 이용하여 파일 쓰기**

    ```java
    import java.io.FileOutputStream;
    import java.io.IOException;

    public class WriteFileExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 열어서 FileOutputStream 객체 fos를 생성
                FileOutputStream fos = new FileOutputStream("example.txt");

                // 파일에 쓸 문자열을 정의
                String str = "Hello, Java!";

                // 문자열을 바이트 배열로 변환
                byte[] bytes = str.getBytes();

                // 변환한 바이트 배열을 파일에 씀
                fos.write(bytes);

                // 파일을 닫음. 이는 시스템 리소스를 반환하기 위해 필요함
                fos.close();
            } catch (IOException e) {
                // 파일 입출력 중 오류가 발생하면 오류 메시지를 출력
                e.printStackTrace();
            }
        }
    }

    ```

2. **`FileInputStream`을 이용하여 파일 읽기**

    ```java
    import java.io.FileInputStream;
    import java.io.IOException;

    public class ReadFileExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 열어서 FileInputStream 객체 fis를 생성
                FileInputStream fis = new FileInputStream("example.txt");

                int i;
                // fis.read()는 한 바이트를 읽어서 반환하고, 파일의 끝에 도달하면 -1을 반환
                // 따라서 파일의 끝에 도달할 때까지 루프를 돌며 파일에서 한 바이트씩 읽음
                while ((i = fis.read()) != -1) {
                    // 읽은 바이트를 문자로 변환하여 출력
                    System.out.print((char) i);
                }

                // 파일을 닫음. 이는 시스템 리소스를 반환하기 위해 필요함
                fis.close();
            } catch (IOException e) {
                // 파일 입출력 중 오류가 발생하면 오류 메시지를 출력
                e.printStackTrace();
            }
        }
    }
    ```


3. **`FileWriter`와 `BufferedWriter`를 이용하여 텍스트 파일 쓰기**

    ```java
    import java.io.BufferedWriter;
    import java.io.FileWriter;
    import java.io.IOException;

    public class WriteTextFileExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일에 쓰기 위한 FileWriter 객체 writer를 생성
                FileWriter writer = new FileWriter("example.txt");

                // FileWriter에 BufferedWriter를 연결하여 버퍼링 기능을 추가
                // 버퍼링을 통해 파일 쓰기 성능을 향상시킴
                BufferedWriter bufferedWriter = new BufferedWriter(writer);

                // 버퍼에 "Hello, Java!" 문자열을 쓴다.
                bufferedWriter.write("Hello, Java!");

                // 버퍼에 새 줄 문자를 추가한다.
                bufferedWriter.newLine();

                // 버퍼에 "This is an example text file." 문자열을 쓴다.
                bufferedWriter.write("This is an example text file.");

                // 버퍼를 닫고, 이 때 버퍼에 남아있는 모든 데이터를 파일에 쓴 후 시스템 리소스를 반환한다.
                bufferedWriter.close();
            } catch (IOException e) {
                // 파일 입출력 중 오류가 발생하면 오류 메시지를 출력한다.
                e.printStackTrace();
            }
        }
    }

    ```

4. **`FileReader`와 `BufferedReader`를 이용하여 텍스트 파일 읽기**

    ```java
    import java.io.BufferedReader;
    import java.io.FileReader;
    import java.io.IOException;

    public class ReadTextFileExample {

        public static void main(String[] args) {
            try {
                // "example.txt" 파일을 읽기 위한 FileReader 객체 reader를 생성
                FileReader reader = new FileReader("example.txt");

                // FileReader에 BufferedReader를 연결하여 버퍼링 기능을 추가
                // 버퍼링을 통해 파일 읽기 성능을 향상시킴
                BufferedReader bufferedReader = new BufferedReader(reader);

                String line;
                // bufferedReader.readLine()은 한 줄을 읽고 반환하며, 
                // 파일의 끝에 도달하면 null을 반환
                // 따라서 파일의 끝에 도달할 때까지 루프를 돌며 한 줄씩 읽음
                while ((line = bufferedReader.readLine()) != null) {
                    // 읽은 한 줄을 출력
                    System.out.println(line);
                }

                // 파일을 닫음. 이는 시스템 리소스를 반환하기 위해 필요함
                reader.close();
            } catch (IOException e) {
                // 파일 입출력 중 오류가 발생하면 오류 메시지를 출력
                e.printStackTrace();
            }
        }
    }
    ```



> 각각의 예시에서, 항상 입출력을 완료한 후에 스트림을 닫는 것(`close()`)을 확인할 수 있다. 이는 리소스를 올바르게 해제하고, 데이터의 손실을 방지하기 위한 중요한 습관이다.