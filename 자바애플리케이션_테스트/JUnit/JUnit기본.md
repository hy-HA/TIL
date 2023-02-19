1. JUnit 구조
2. JUnit 시작하기
3. 테스트 이름 표시하기 
4. Assertion
5. 테스트 인스턴스
6. 테스트 순서__@TestMethodOrder
7. JUnit 설정__junit-platform.properties
8. 확장 모델
9. JUnit 4 마이그레이션

# 1. JUnit 구조
[참고](https://junit.org/junit5/docs/current/user-guide/)
1. Platform:  테스트를 실행해주는 런처 제공하고 TestEngine API를 제공하는 모듈.
2. Jupiter: TestEngine API 구현체로 JUnit 5를 제공.
3. Vintage: JUnit 4와 3을 지원하는 TestEngine 구현체.

# 2. JUnit 시작하기

- 스프링 부트 프로젝트 만들기 
    - 2.2+ 버전의 스프링 부트 프로젝트를 만든다면 기본으로 JUnit 5 의존성 추가 됨.

## 기본 애노테이션
### 1. @Test

### 2. @BeforeAll / @AfterAll
1. 모든 @Test를 실행하기 전(후)에 1번만 호출이 됨
2. 이 애너테이션을 사용할 때는 반드시 static 메서드를 사용해야. 
    - private(x) default는 됨.
    - return 타입이 있으면 안됨
    - static void로 사용한다고 기억하면 좋음.

### 3. @BeforeEach / @AfterEach
1. 각각 테스트를 실행하기 전(후)로 호출이 됨

### 4. @Disabled
- 테스트하고싶지 않은 메서드에 이 애너테이션 사용.
- 전체 테스트를 실행해도 이 애너테이션이 붙은 메서드는 실행되지 않음


# 3. 테스트 이름 표시하기 

[그 외 애너테이션 참고](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names)

- JUnit 테스트시 기본적으로 테스트 이름은 메서드 이름이 표기됨
- 아래 에너테이션을 사용하여 원하는 문자열로 보이도록 설정함

### @DisplayNameGeneration
- Method와 Class 레퍼런스를 사용해서 테스트 이름을 표기하는 방법 설정.
- 기본 구현체로 ReplaceUnderscores 제공

### @DisplayName
- 테스트 이름을 보다 쉽게 표현할 수 있는 방법을 제공하는 애노테이션.
- @DisplayNameGeneration 보다 우선 순위가 높다.

```
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Test
    @DisplayName("스터디 만들기 ╯°□°）╯")
    void create_new_study() {
        Study study = new Study();
        assertNotNull(study);
        System.out.println("create");
    }
}
```

### 테스트 단축키
- 컨트롤+shift+R
- 커서를 특정 메서드위에 올리고 단축키 누르면 해당 메서드만 테스트 됨

<br>


# 4. Assertion
- org.junit.jupiter.api.Assertions.*

### 1. assertEqulas(expected, actual)
- 실제 값이 기대한 값과 같은지 확인
    - 첫번째 파라미터 : 기대한 값
    - 두번째 파라미터 : 실제 값

```
Study study = new Study();
assertEquals(StudyStatus.DRAFT, study.getStatus(),"스터디를 처음 만들면 상태값이 DRAFT여야 한다")
```
- 메세지를 람다식으로 사용하기
    - 에러 발생할 때에만 해당 메세지 출력하도록
```
Study study = new Study();
assertEquals(StudyStatus.DRAFT, study.getStatus(),
            () -> "스터디를 처음 만들면" + StudyStatus.DRAFT + "상태다")
```

### 2. assertNotNull(actual)
- 값이 null이 아닌지 확인

```
Study study = new Study();
assertNotNull(study);
```

### 3. assertTrue(boolean)
- 다음 조건이 참(true)인지 확인
```
assertTrue(2<1);
```

### 4. assertAll(executables...)
- 모든 확인 구문 확인
    - 원래는 최상단의 테스트가 에러나면 뒤 테스트는 실행하지 않음
    - 최상단의 에러 관계 없이 모든 테스트를 하고 싶을때 assertAll로 감싸기
- executable 타입으로 묶은 것
    - executable 3개를 파라미터로 전달한 것
- assertAll은 executable 타입을 받음
```
assertAll(
    () -> assertNotNull(study),
    () -> assertEquals(StudyStatus.DRAFT, study.getStatus(),
        ()->"스터디를 처음 만들면" + StudyStatus.DRAFT + "상태다"), 
    () -> assertTrue(study.getLimit() > 0, "스터디 최대 참석 가능인원은 0보다 크다")
);
```

### 5. assertThrows(expectedType, executable)
- 예외 발생 확인
    - 첫번째 파라미터 : 어떤 종류의 에러가 발생할지
    - 두번째 파라미터 : 어떤 코드를 실행할 때
```
assertThrows(IllegalArgumentException.class, () -> new Study(-10));
```
```
IllegalArgumentException exception = 
    assertThrows(IllegalArgumentException.class, () -> new Study(-10));
String message = exception.getMessage();
assertEquals("limit은 0보다 커야한다",exeption.getMessage());

-------------
Study.class>>

public Study(int limit) {
    if(limit < 0) {
        throw new IllegalArgumentException("limit은 0보다 커야한다")
    }
    this.limit = limit;
}
```
### 6. assertTimeout(duration, executable)
- 특정 시간 안에 실행이 완료되는지 확인
    - 첫번째 파라미터 : 얼마만에 끝내야 할지
    - 두번째 파라미터 : 어떤 것을 할 때
- 특정 시간 안에 실행이 안되면 에러 뜸
```
assertTimeout(Duration.ofMillis(100), () -> {
    new Study(10);
    Thread.sleep(300);
});
```


## 스트링부트스타터가 제공하는 assertj,hamcrest 사용하기

### 1. assertThat
```
Study actual = new Study(10);
assertThat(actual.getLimit()).isGreaterThan(0);
```

# 5. 테스트 인스턴스__@TestInstance(TestInstance.Lifecycle.PER_CLASS)


1. JUnit은 테스트 메소드 마다 테스트 인스턴스를 새로 만든다.이것이 기본 전략.
    - JUnit으로 StudyTest클래스를 테스트 실행할때, 클래스의 인스턴스를 만든다. 
    - 클래스 내부의 메서드들을 테스트 실행할 때마다 클래스의 인스턴스를 만든다.
2. 테스트 메소드를 독립적으로 실행하여 예상치 못한 부작용을 방지하기 위함이다.
    - 테스트는 메소드 작성 순서대로 실행되지 않음.
        - 기본적으로 코드 작성된 순서대로 실행되지만 매번 그럴 것이라 생각하면안됨
        - 순서에 의존하면 안됨
        - JUnit 내부 구현 로직에 따라 언제든 순서는 바뀔 수 있음(?)
3. 이 전략을 JUnit 5에서 변경할 수 있다.

```
class StudyTest {

    int value = 1;

    @FastTest
    @DisplayName("스터디 만들기 fast")
    void create_new_study() {
        System.out.println(this);
        System.out.println(value++);
        Study actual = new Study(1);
        assertThat(actual.getLimit()).isGreaterThan(0);
    }

    @SlowTest
    @DisplayName("스터디 만들기 slow")
    void create_new_study_again() {
        System.out.println(this);
        System.out.println("create1 " + value++ );
    }
}

>> value는 모든 메서드에서 1이 출력됨
>> this의 값(클래스의 인스턴스)은 모든 메서드마다 다름(@6a2b9Be. @3h8f9Ej,,,)
```

### @TestInstance(Lifecycle.PER_CLASS)
- 테스트 클래스당 인스턴스를 하나만 만들어 사용한다.
```
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class StudyTest {
```
- @BeforeAll과 @AfterAll을 꼭 static으로 선언하지 않아도됨
    - 인스턴스 메소드 또는 인터페이스에 정의한 default 메소드로 정의할 수도 있다. 
    - 참고(java/2.객체지향프로그래밍/1/10)

경우에 따라, 테스트 간에 공유하는 모든 상태를 @BeforeEach 또는 @AfterEach에서 초기화 할 필요가 있다.

# 6. 테스트 순서__@TestMethodOrder

### 테스트 실행 순서의 불명확성
- 실행할 테스트 메소드 특정한 순서에 의해 실행되지만 
- 어떻게 그 순서를 정하는지는 의도적으로 분명히 하지 않는다. 
    - 테스트 인스턴스를 테스트 마다 새로 만드는 것과 같은 이유

### 테스트 실행 순서를 정하고 싶을 때
- 경우에 따라, 특정 순서대로 테스트를 실행하고 싶을 때도 있다. 
- 그 경우에는 테스트 메소드를 원하는 순서에 따라 실행하도록 
- @TestInstance(Lifecycle.PER_CLASS)와 함께 
    - 테스트 클래스 인스턴스를 1개로 실행
- @TestMethodOrder를 사용할 수 있다.

### @TestMethodOrder
- MethodOrderer 구현체를 설정한다.
- 기본 구현체
    1. Alphanumeric
    2. OrderAnnoation
        - springframework가 아닌 Junit이 제공하는 인터페이스로 사용할 것!
        - @Order(1), @Order(2),,, 
    3. Random

```
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnoation.class)
public class StudyCreateUsecaseTest {

    private Study study;

    @Order(1)
    @Test
    @DisplayName("스터디 만들기")
    public void create_study() {
        study = new Study(10, "자바");
        assertEquals(StudyStatus.DRAFT, study.getStatus());
    }

    @Order(2)
    @Test
    @DisplayName("스터디 공개")
    public void publish_study() {
        study.publish();
        assertEquals(StudyStatus.OPENED, study.getStatus());
        assertNotNull(study.getOpenedDateTime());
    }

}


```

<br>

# 7. JUnit 설정__junit-platform.properties

## junit-platform.properties
- JUnit5는 JUnit 설정 파일을 제공함.
    - 아래 기능들을 설정할 수 있음
- 클래스패스 루트 (src/test/resources/)에 넣어두면 적용된다.
- 인텔리J에서 해당 폴더를 테스트 리소스 폴더로 인식하도록 해야 함
    - 테스트 실행 시 해당 경로를 테스트에 클래스패스로 사용하도록 설정
        - File/Project Structure/Modules에서 resources디렉토리를 Test Resources디렉토리로 설정(Test Resources 버튼 클릭 후 ok버튼)
        - 설정 후 resources폴더의 아이콘이 바뀐 것 확인
### 1. 테스트 인스턴스 라이프사이클 설정
```
junit-platform.properties>>

junit.jupiter.testinstance.lifecycle.default = per_class
```
### 2. 확장팩 자동 감지 기능
```
junit-platform.properties>>

junit.jupiter.extensions.autodetection.enabled = true
// 기본값은 false
```
### 3. @Disabled 무시하고 실행하기
```
junit-platform.properties>>

junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition
//@Disabled 붙어있어도 실행하게 됨
```
### 4. 테스트 이름 표기 전략 설정
```
junit-platform.properties>>

junit.jupiter.displayname.generator.default = \
    org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores

// \ : 줄바꿈
```

# 8. 확장 모델
[참고](https://junit.org/junit5/docs/current/user-guide/#extensions)
- JUnit 4의 확장 모델은 @RunWith(Runner), TestRule, MethodRule.
- JUnit 5의 확장 모델은 단 하나, Extension.
    - JUnit 4에서 쓰던 모델 5에서 못 씀
## 확장팩 등록 방법

1. 선언적인 등록 @ExtendWith
2. 프로그래밍 등록 @RegisterExtension
3. 자동 등록 자바 ServiceLoader 이용


## 확장 모델 사용하기
### 1. 1초 이상 시간이 걸리는 테스트 메소드 확인하는 Extension 생성 
- 아래 문장 출력하도록
    - "Please consider mark method [%s] with @SlowTest.\n", testMethodName"
```
public class FindSlowTestExtension 
    implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private long THRESHOLD = 1000L;


    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        ExtensionContext.Store store = getStore(context);
        store.put("START_TIME", System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method requiredTestMethod = context.getRequiredTestMethod();
        SlowTest annotation = requiredTestMethod.getAnnotation(SlowTest.class);
        String testMethodName = requiredTestMethod.getName();
        ExtensionContext.Store store = getStore(context);
        long start_time = store.remove("START_TIME", long.class);
        long duration = System.currentTimeMillis() - start_time;
        if (duration > THRESHOLD && annotation == null) {
            System.out.printf("Please consider mark method [%s] with @SlowTest.\n", testMethodName);
        }

    }

    private ExtensionContext.Store getStore(ExtensionContext context) {
        String testClassName = context.getRequiredTestClass().getName();
        String testMethodName = context.getRequiredTestMethod().getName();
        return context.getStore(ExtensionContext.Namespace.create(testClassName, testMethodName));
    }
}
```

### 2. 테스트클래스에 선언하기__@ExtendWith

```
@ExtendWith(FindSlowTestExtension.class)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class StudyTest {

}
```

### 3. 테스트클래스에 선언하기__@RegisterExtension
- FindSlowTestExtension.class 내부의 THRESHOLD를 테스트 별로 다르게 설정할 수 있음

``` 
FindSlowTestExtension.class>>

public class FindSlowTestExtension 
    implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private long THRESHOLD;

    public FindSlowTestExtension(long THRESHOLD) {
            this.THRESHOLD = THRESHOLD;
    }
}
```

```
StudyTest.class>>

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class StudyTest {

    int value = 1;

    @RegisterExtension // THRESHOLD 시간 설정 가능
    static FindSlowTestExtension findSlowTestExtension =
            new FindSlowTestExtension(1000L);

    @Order(2)
    @FastTest
    @DisplayName("스터디 만들기 fast")
    void create_new_study() {
        ~~~~
    }
}
```

# 9. JUnit 4 마이그레이션
- junit-vintage-engine을 의존성으로 추가하면, 
    - JUnit 5의 junit-platform으로 JUnit 3과 4로 작성된 테스트를 실행할 수 있다.

- 완벽하진 않음
    - @Rule은 기본적으로 지원하지 않지만, 
        - junit-jupiter-migrationsupport 모듈이 제공하는 @EnableRuleMigrationSupport를 사용하면 다음 타입의 Rule을 지원한다.
            1. ExternalResource
            2. Verifier
            3. ExpectedException

- JUnit 4 vs JUnit 5

```
JUnit 4                     vs     JUnit 5

@Category(Class)            >>   @Tag(String)
@RunWith, @Rule, @ClassRule >>   @ExtendWith, @RegisterExtension
@Ignore                     >>   @Disabled
@Before, @After,            >>   @BeforeEach, @AfterEach, 
@BeforeClass, @AfterClass   >>   @BeforeAll, @AfterAll
```

@RunWith
- @RunWith를 메타 애너테이션으로 사용할 수 없었음
- 5부터는 @SpringBootTest에서 제공하는 @ExtendWith에서 메타 애너테이션으로 사용가능해짐





