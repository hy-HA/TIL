1. Mockito 소개
2. Mockito 시작하기
3. Mock 객체 만들기
4. Mock 객체 Stubbing



# 1. Mockito 소개
[참고](https://www.jetbrains.com/lp/devecosystem-2019/java/)
[참고-단위테스트](https://martinfowler.com/bliki/UnitTest.html)

### Mock 
- 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.

### Mockito
- Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.

- 테스트를 작성하는 자바 개발자 50%+ 사용하는 Mock 프레임워크.

# 2. Mockito 시작하기
- 스프링 부트 2.2+ 프로젝트 생성시 spring-boot-starter-test에서 자동으로 Mockito 추가해 줌.
- 다음 세 가지만 알면 Mock을 활용한 테스트를 쉽게 작성할 수 있다.
    1. Mock을 만드는 방법
    2. Mock이 어떻게 동작해야 하는지 관리하는 방법
    3. Mock의 행동을 검증하는 방법

[참고-Mockito 레퍼런스](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)

<br>


# 3. Mock 객체 만들기
1. StudyService의 인스턴스 만들기 : createStudyService

### 1. Mockito.mock() 메소드로 객체 만들기
- 인터페이스만 있고 구현체는 없는 MemberService와 StudyRepository의 구현체를 직접 Mock이 만들어줌
```
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudyService() {
        MemberService memberService = mock(MemberService.class);
        StudyRepository studyRepository = mock(StudyRepository.class);

        StudyService studyService = new StudyService(memberService, studyRepository);
        assertNotNull(studyService);
    }

}
```

### 2. @Mock 애노테이션으로 객체 만들기
1. @ExtendWith(MockitoExtension.class)
    - JUnit 5 extension으로 MockitoExtension을 선언
        - @Mock애너테이션을 사용하여 Mock객체를 만들 수 있음

2. 필드(or 메소드 매개변수)로 선언

- Mock객체를 여러 테스트에 걸쳐 사용하는 경우 아래처럼  등록하는게 편함
```
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Mock
    MemberService memberService

    @Mock
    StudyRepository studyRepository

    @Test
    void createStudyService() {

        StudyService studyService = new StudyService(memberService, studyRepository);
        assertNotNull(studyService);
    }

}
```
- mock객체를 해당 메서드 안에서만 사용한다면, 애너테이션으로 만들어서 사용 가능
    - 메서드의 파라미터로 mock애너테이션을 사용해도 객체를 만들어줌
```
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudyService(@Mock MemberService memberService,
                            @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);
        assertNotNull(studyService);
    }

}
```

# 4. Mock 객체 Stubbing

### 모든 Mock 객체의 행동
- Null을 리턴한다. (Optional 타입은 Optional.empty 리턴)
- Primitive 타입은 기본 Primitive 값.
- 콜렉션은 비어있는 콜렉션.
- Void 메소드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.

### Mock 객체를 조작해서
- 특정한 매개변수를 받은 경우 특정한 값을 리턴하거나 예외를 던지도록 만들 수 있다.
    - [How about some stubbing?](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#2)
    - [Argument matchers](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#3)
- Void 메소드 특정 매개변수를 받거나 호출된 경우 예외를 발생 시킬 수 있다.
    - [Subbing void methods with exceptions](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#5)
- 메소드가 동일한 매개변수로 여러번 호출될 때 각기 다르게 행동하도록 조작할 수도 있다.
    - [Stubbing consecutive calls](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#10)

```
public interface MemberService {

    Optional<Member> findById(Long memberId);

    void validate(Long memberId);
}
```
```
public class StudyService {

    private final MemberService memberService;

    private final StudyRepository repository;

    public StudyService(MemberService memberService, StudyRepository repository) {
        this.memberService = memberService;
        this.repository = repository;
    }

    public Study createNewStudy(Long memberId, Study study) {
        Member member = memberService.findById(memberId);
        if (member == null) {
            throw new IllegalArgumentException("Member doesn't exist for id: '" + memberId + "'");
        }
        study.setOwner(member);
        return repository.save(study);
    }

}
```
- when()이 호출되면 thenReturn()을 리턴하라
- when()이 호출되면 thenThrow()예외를 던져라
    - Argument matchers를 any로 설정하면 '아무거나 받아도'라는 뜻
```
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createNewStudy(@Mock MemberService memberService,
                            @Mock StudyRepository studyRepository) {
        
        //given
        StudyService studyService = new StudyService(memberService, studyRepository);
        assertNotNull(studyService);

        Member member = new Member();
        member.setId(1L);
        member.setEmail("heeha@email.com");

        Study study = new Study(10, "java");

//        studyService.createNewStudy(1L, study); //이때 사용되는 memberService는 mock객체

        (1)첫번째 케이스
        //when - 파라미터로 아무거나 받아도 아래 결과를 리턴/오류를 던져라
        when(memberService.findById(any())).thenReturn(Optional.of(member))

        //then - 1로 호출하던, 2로 호출하던 호출된 객체의 이메일 값은 member의 이메일 값을 리턴함
        assertEquals("heeha@email.com", memberService.findById(1L).get().getEmail());
        assertEquals("heeha@email.com", memberService.findById(2L)).get().getEmail());

        (2)두번째 케이스--when&then(리턴타입이 있는 경우)
        //1이라는 값으로 findById가 호출이 되면 예외를 던지겠다(리턴타입이 있는 경우)
        when(memberService.findById(1L)).thenThrow(new RuntimeException());

        (3)세번째 케이스--doThrow(void의 경우)
        //when - 멤버서비스가 validate가 1이라는 값으로 호출됐을때 예외를 던져라
        doThrow(new IllegalArgumentException()).when(memberService).validate(1L);

        //then 
        assertThrows(IllegalArgumentException.class, () -> {
            memberService.validate(1L);

        memberService.validate(2L) //이 경우는 예외가 나지 않음
        
        (4)
        //when - 파라미터로 아무거나 받아도 아래 결과를 리턴/오류를 던져라
        when(memberService.findById(any()))
                .thenReturn(Optional.of(member))   //첫번째 호출시
                .thenThrow(new RuntimeException()) //두번째 호출시
                .thenReturn(Optional.empty());     //세번째 호출시

        //then - 첫번째 호출시, 두번째 호출시, 세번째 호출시 각각의 위의 결과가 나오도록

            //첫번째 호출
        Optional<Member> byId = memberService.findById(1L);
        assertEquals("keesun@email.com", byId.get().getEmail());
        
            //두번째 호출
        assertThrows(RuntimeException.class, () -> {
            memberService.findById(2L);
        });

            //세번째 호출
        assertEquals(Optional.empty(), memberService.findById(3L));


        });
    }

}
```