# JUnit5

- JAVA 8 버전부터 사용 가능
- 테스트 주도 개발(TDD, Test Driven Development)을 위함
    - 요구사항을 검증하는 테스트 케이스 작성
    - 테스트 통과를 위한 코드 작성
    - 이후 리팩토링하여 실제 개발 코드에 적용
- 참고 사이트 https://velog.io/@ynjch97/JUnit5-JUnit5-%EA%B5%AC%EC%84%B1-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-Assertions-%EC%A0%95%EB%A6%AC

# 1-1. JUnit5 구성

- JUnit5는 JUnit Platform, JUnit Jupiter, JUnit Vintage가 결합한 형태
- JUnit Platform
    - JVM에서 테스트 프레임워크를 실행하는 런처 제공
    - TestEngine API를 제공
- JUnit Jupiter
    - TestEngine API 구현체
    - JUnit5에서 제공
- JUnit Vintage
    - TestEngine API 구현체
    - JUnit3, 4에서 제공
    - 하위 호환성을 위해 JUnit3, 4 기반 플랫폼에 TestEngine 제공

# 1-2. JUnit5 어노테이션

- @Test : 테스트 Method임을 선언
- @ParameterizedTest : 매개변수를 받는 테스트 작성
- @RepeatedTest : 반복되는 테스트 작성
- @TestFactory : 동적으로 테스트를 사용함
    - @Test : 정적 테스트
- @TestInstance : 테스트 클래스의 생명주기 설정
- @TestTemplate : 공급자에 의해 여러 번 호출될 수 있도록 설계된 테스트 케이스 템플릿임을 나타냄
- @TestMethodOrder : 테스트 메소드 실행 순서 구성에 사용
- @DisplayName : 테스트 클래스 or 메소드의 사용자 정의 이름 선언
- @DisplayNameGeneration : 이름 생성기 선언
    - ex) '_'를 공백 문자로 치환해주는 생성기
    - new_test -> new test
- @BeforeEach : 모든 테스트 실행 전에 실행할 테스트
    - JUnit4에서는 @Before
- @AfterEach : 모든 테스트 실행 후에 실행할 테스트
    - JUnit4에서는 @After
- @BeforeAll : 현재 클래스 실행 전 제일 먼저 실행할 테스트
    - static으로 선언
    - JUnit4에서는 @BeforeClass
- @AfterAll : 현재 클래스 종료 직후 실행할 테스트
    - static으로 선언
    - JUnit4에서는 @AfterClass
- @Nested : (정적이 아닌) 중첩 테스트 클래스
- @Tag : 클래스 or 메소드 레벨에서 태그 선언
- @Disabled : 이 클래스나 테스트를 사용하지 않음
    - JUnit4에서는 @Ignore
- @Timeout : 테스트 실행 시간 선언, 초과되면 실패하도록 설정
- @ExtendWith : 확장을 선언적으로 등록할 때 사용
- @RegisterExtension : 필드를 통해 프로그래밍 방식으로 확장을 등록할 때 사용
- @TempDir : 필드 주입 or 매개변수 주입을 통해 임시 디렉토리 제공

# 1-3. JUnit4 Assertions

- 참고 사이트 [https://www.baeldung.com/junit-assertions](https://www.baeldung.com/junit-assertions)

## 1-3-1. assertEquals

- 두 값을 비교하여 일치 여부 판단
- `assertEquals("불일치", expected, actual);` 실패 시 반환할 메세지 설정 가능

```
@Test
public void test() {
    String expected = "eunjy";
    String actual = "eunjy";

    assertEquals(expected, actual);
}
```

## 1-3-2. assertArrayEquals

- 두 배열을 비교하여 일치 여부 판단
- 두 배열이 모두 null이어도 동일한 것으로 간주함

```
@Test
public void test() {
    char[] expected = {'J','u','n','i','t'};
    char[] actual = "Junit".toCharArray();

    assertArrayEquals(expected, actual);
}
```

## 1-3-3. assertNotNull & assertNull

- 객체의 null 여부 확인

```
@Test
public void test() {
    Object car = null;

    assertNull("The car should be null", car);
}
```

## 1-3-4. assertNotSame & assertSame

- 두 변수가 동일한 객체를 참조하는지 확인

```
@Test
public void test() {
    Object cat = new Object();
    Object dog = new Object();

    assertNotSame(cat, dog);
}
```

## 1-3-5. assertTrue & assertFalse

- 특정 조건이 true인지 false인지 판단

```
@Test
public void test() {
    assertTrue("5 is greater then 4", 5 > 4);
    assertFalse("5 is not greater then 6", 5 > 6);
}
```

## 1-3-6. fail

- AssertionFailedError를 발생시키는 테스트에 실패
- 실제 예외가 발생했는지 확인하거나, 개발 중에 테스트를 실패하게 만들고 싶을 때 사용

```
@Test
public void test() {
    try {
        methodThatShouldThrowException();
        fail("Exception not thrown");
    } catch (UnsupportedOperationException e) {
        assertEquals("Operation Not Supported", e.getMessage());
    }
}
```

## 1-3-7. assertThat

- 첫번째 파라미터에 비교대상 값, 두번째 파라미터에 비교 로직이 담긴 Matcher를 받음

```
@Test
public void test() {
	assertThat("Sample string.", is(not(startsWith("Test"))));
}
```

# 1-4. JUnit5 Assertions

## 1-4-1. assertArrayEquals

- 예상 배열과 실제 배열이 동일한지 확인
- 배열이 같지 않으면 마지막 인자로 들어간 메세지가 출력됨

```
@Test
public void test() {
    char[] expected = { 'J', 'u', 'p', 'i', 't', 'e', 'r' };
    char[] actual = "Jupiter".toCharArray();

    assertArrayEquals(expected, actual, "Arrays should be equal");
}
```

## 1-4-2. assertEquals

- 두 값을 비교하여 일치 여부 판단

```
@Test
public void test() {
    float square = 2 * 2;
    float rectangle = 2 * 2;

    assertEquals(square, rectangle);
}
```

## 1-4-3. assertTrue & assertFalse

- JUnit4 버전과 동일하며 BooleanSupplier도 사용 가능

```
@Test
public void test() {
    BooleanSupplier condition = () -> 5 > 6;

    assertFalse(condition, "5 is not greater then 6");
}
```

## 1-4-4. assertNull and assertNotNull

- JUnit4 버전과 동일
- 객체의 null 여부 확인

```
@Test
public void test() {
    Object cat = null;

    assertNull(cat, () -> "The cat should be null");
}
```

## 1-4-5. assertSame and assertNotSame

- JUnit4 버전과 동일
- 예상되는 값과 실제 값이 동일한 객체를 참조하는지 확인

```
@Test
public void test() {
    String language = "Java";
    Optional<String> optional = Optional.of(language);

    assertSame(language, optional.get());
}
```

## 1-4-6. fail

- 제공된 실패 메시지와 기본 원인으로 테스트에 실패
- 개발이 완료되지 않은 테스트를 표시하는 데 유용

```
@Test
public void test() {
    // Test not completed
    fail("FAIL - test not completed");
}
```

## 1-4-7. assertAll

- 모든 Assertion이 실행되고 실패가 함께 보고되는 그룹화된 Assertion
- MultipleFailureError에 대한 메시지 문자열에 포함될 제목과 실행 가능한 스트림을 허용
- 실행 파일 중 하나에서 OutOfMemoryError가 발생한 경우에만 중단됨
- 메소드 내에서 인자로 람다식을 사용
    - 여러 개의 람다식이 동시에 실행됨

```
@Test
public void test() {
    assertAll(
      "heading",
      () -> assertEquals(4, 2 * 2, "4 is 2 times 2"),
      () -> assertEquals("java", "JAVA".toLowerCase()),
      () -> assertEquals(null, null, "null is equal to null")
    );
}
```

## 1-4-8. assertIterableEquals

- 예상 반복 가능 항목과 실제 반복 가능 항목이 동일한지 확인
- 두 Iterable은 동일한 순서로 동일한 요소를 반환해야 함
- 두 Iterable이 동일한 유형일 필요는 없음
- 아래에서 서로 다른 유형의 두 목록(LinkedList 및 ArrayList)이 동일한지 확인

```
@Test
public void test() {
    Iterable<String> al = new ArrayList<>(asList("Java", "Junit", "Test"));
    Iterable<String> ll = new LinkedList<>(asList("Java", "Junit", "Test"));

    assertIterableEquals(al, ll);
}
```

## 1-4-9. assertLinesMatch

- 예상 목록이 실제 목록과 일치하는지 확인
- assertEquals, assertIterableEquals와 다름
    - 예상 줄이 실제 줄과 같은지 확인
    - 같으면 다음 쌍으로 이동
    - String.matches() 메서드로 검사
    - fast-forward marker 확인
- 아래에서 두 목록에 일치하는 행이 있는지 검사

```
@Test
public void test() {
    List<String> expected = asList("Java", "\\d+", "JUnit");
    List<String> actual = asList("Java", "11", "JUnit");

    assertLinesMatch(expected, actual);
}
```

## 1-4-10. assertNotEquals

- 예상 값과 실제 값이 다름을 확인

```
@Test
public void test() {
    Integer value = 5; // result of an algorithm

    assertNotEquals(0, value, "The result cannot be 0");
}
```

## 1-4-11. assertThrows

- 특정 예외가 발생하였는지 확인
- 첫 번째 인자는 확인할 예외 클래스
- 두 번째 인자는 테스트하려는 코드

```
@Test
void test() {
    Throwable exception = assertThrows(
      IllegalArgumentException.class,
      () -> {
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

## 1-4-12. assertTimeout & assertTimeoutPreemptively

- 특정 시간 안에 실행이 끝나는지 확인
- 시간 내 실행이 끝나는지 여부 확인 시 : assertTimeout
- 지정한 시간 내 끝나지 않으면 바로 종료 : assertTimeoutPreemptively

```
@Test
public void test() {
    assertTimeout(
      ofSeconds(2),
      () -> {
        // code that requires less then 2 minutes to execute
        Thread.sleep(1000);
      }
    );
}
```