# Junit
참고 ] https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html

참고 ] https://velog.io/@ynjch97/JUnit5-JUnit5-%EA%B5%AC%EC%84%B1-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-Assertions-%EC%A0%95%EB%A6%AC

- 자바 개발자가 가장 많이 사용하는 테스팅 기반 프레임워크

# JUnit5 어노테이션
```
@Test : 테스트 Method임을 선언
@BeforeEach : 모든 테스트 실행 전에 실행할 테스트
              JUnit4에서는 @Before
@AfterEach : 모든 테스트 실행 후에 실행할 테스트
              JUnit4에서는 @After
@ParameterizedTest : 매개변수를 받는 테스트 작성
@RepeatedTest : 반복되는 테스트 작성
@TestFactory : 동적으로 테스트를 사용함
@TestInstance : 테스트 클래스의 생명주기 설정
@TestTemplate : 공급자에 의해 여러 번 호출될 수 있도록 설계된 테스트 케이스 템플릿임을 나타냄
@TestMethodOrder : 테스트 메소드 실행 순서 구성에 사용
@DisplayName : 테스트 클래스 or 메소드의 사용자 정의 이름 선언
@DisplayNameGeneration : 이름 생성기 선언
ex) '_'를 공백 문자로 치환해주는 생성기
new_test -> new test
@BeforeAll : 현재 클래스 실행 전 제일 먼저 실행할 테스트
static으로 선언
JUnit4에서는 @BeforeClass
@AfterAll : 현재 클래스 종료 직후 실행할 테스트
static으로 선언
JUnit4에서는 @AfterClass
@Nested : (정적이 아닌) 중첩 테스트 클래스
@Tag : 클래스 or 메소드 레벨에서 태그 선언
@Disabled : 이 클래스나 테스트를 사용하지 않음
JUnit4에서는 @Ignore
@Timeout : 테스트 실행 시간 선언, 초과되면 실패하도록 설정
@ExtendWith : 확장을 선언적으로 등록할 때 사용
@RegisterExtension : 필드를 통해 프로그래밍 방식으로 확장을 등록할 때 사용
@TempDir : 필드 주입 or 매개변수 주입을 통해 임시 디렉토리 제공
```

# JUnit 4 Assertions
- org.junit.jupiter.api.Assertions 클래스는 값 검증을 위한 assert로 시작하는 static 메서드를 제공

## assertEquals
  - 두 값을 비교하여 일치 여부 판단
  - assertEquals("불일치", expected, actual); 실패 시 반환할 메세지 설정 가능
    ```
    @Test
    public void test() {
        String expected = "eunjy";
        String actual = "eunjy";

        assertEquals(expected, actual);
    }
    ```

## assertThat
  - 첫번째 파라미터에 비교대상 값, 두번째 파라미터에 비교 로직이 담긴 Matcher를 받음
    ```
    @Test
    void 회원가입() {

    //given
    Member member = new Member();
    member.setName("hello");

    //when
    Long ​saveId = memberService.join(member);

    //then
    Member findMember = memberService.findOne(saveId).get();
    assertThat(member.getName()).isEqualTo(findMember.getName());
    ```
    ```
    @Test
    public void test() {
      assertThat("Sample string.", is(not(startsWith("Test"))));
    }
    ```
# JUnit5 Assertions

## assertThrows
  - 특정 예외가 발생하였는지 확인
  - 첫 번째 인자는 확인할 예외 클래스
  - 두 번째 인자는 테스트하려는 코드
    ```
    memberService.join(member1);
    IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
    assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.")
    ```
    ```
    @Test
    void test() {
        Throwable exception = assertThrows(
          IllegalArgumentException.class, () -> 
          {
              throw new IllegalArgumentException("Exception message");
          }
        );
        assertEquals("Exception message", exception.getMessage());
    }
    ```
    ```
    @Test
        void test() {
            Exception exception = assertThrows(ArithmeticException.class, () ->
                calculator.divide(1, 0));
            assertEquals("/ by zero", exception.getMessage());
        }
    ```

## Java RuntimeException
- Java 프로그래밍을 하면서 예외처리를 발생시켜야하는 경우, RuntimeException을 상속하는 예외를 사용하면 됨. 그리고 RuntimeException을 상속하는 예외를 새롭게 만드는 것보다는 JDK에서 제공하는 표준 RuntimeExcepton 상속 예외들을 사용하는 것이 바람직
- JDK 12 기준으로 RuntimeException을 직접 상속하는 예외는 총 58개입니다. (참고 - OpenJdk 12 RuntimeException)
```
참고로 전체 계층 구조를 보려면 Intellij의 Hierarchy 기능을 이용
  - RuntimeException의 전체 계층 구조 확인
        - shift 연속 두번 누른 후 RuntimeException 입력 후 선택 : RuntimeException class가 열림
        - Ctrl + H
```
1. NullPointException
- null을 허용하지 않는 메소드에 null이 인자로 넘어왔을 때 사용
2. IllegalStateException
- 호출받은 객체가 요청을 수행하기에 적합하지 않은 상태