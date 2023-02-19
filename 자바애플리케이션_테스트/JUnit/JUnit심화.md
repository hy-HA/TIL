1. 조건에 따라 테스트 실행하기
2. 태깅과 필터링
3. 커스텀 태그
4. 테스트 반복하기 1부
5. 테스트 반복하기 2부


# 1. 조건에 따라 테스트 실행하기
- 특정한 조건을 만족하는 경우에 테스트를 실행하는 방법.

## org.junit.jupiter.api.Assumptions.*

### assumeTrue(조건)
- 로컬인 환경에만 아래 테스트를 수행한다
```
String test_env = System.getenv("TEST_ENV")
System.out.pringln(test_env);
assumeTrue("LOCAL".equalsIgnoreCase(System.getenv("TEST_ENV")));

Study. actual = new Study(10);
assertThat(actual.getLimit()).isGreaterThan(0);

>> 환경변수에 로컬로 설정되지 않은 경우 테스트 수행 안됨
```
### assumingThat(조건, 테스트)
```
String test_env = System.getenv("TEST_ENV")

assumingThat("LOCAL".equalsIgnoreCase(test_env), () -> {
    System.out.pringln("local");
    Study. actual = new Study(100);
    assertThat(actual.getLimit()).isGreaterThan(0);
});

assumingThat("heeha".equalsIgnoreCase(test_env), () -> {
    System.out.pringln("heeha");
    Study. actual = new Study(10);
    assertThat(actual.getLimit()).isGreaterThan(0);
});


>> 환경변수가 로컬, heeha로 설정된 경우 각각 테스트
```


## @Enabled___ 와 @Disabled___

### 1. @EnabledOnOS  @DisabledOnOS
- 특정 OS에서 작동, 작동되지 않도록 설정
```
@Test
@DisplayName("스터디 만들기")
@EnabledOnOS({OS.MAC, OS.LINUX})
void create_new_study(){
    String test_env = System.getenv("TEST_ENV")
    System.out.pringln("local");
    Study. actual = new Study(100);
    assertThat(actual.getLimit()).isGreaterThan(0);
}

@Test
@DisplayName("스터디 만들기2")
@DisabledOnOS(OS.MAC)
void create_new_study_again(){ System.out.pringln("create1"); }
```

### 2. @EnabledOnJre  @DisabledOnJre
- 특정 자바버전에서 작동, 작동되지 않도록 설정
```
@Test
@DisplayName("스터디 만들기")
@EnabledOnJre({JRE.JAVA_8, JRE.JAVA_9, JRE.JAVA_10, JRE.JAVA_11})
void create_new_study(){
    String test_env = System.getenv("TEST_ENV")
    System.out.pringln("local");
    Study. actual = new Study(100);
    assertThat(actual.getLimit()).isGreaterThan(0);
}

@Test
@DisplayName("스터디 만들기2")
@DisabledOnJre(JRE.OTHER)
void create_new_study_again(){ System.out.pringln("create1"); }
```

### 그 외
- IfSystemProperty
- IfEnvironmentVariable
- If

