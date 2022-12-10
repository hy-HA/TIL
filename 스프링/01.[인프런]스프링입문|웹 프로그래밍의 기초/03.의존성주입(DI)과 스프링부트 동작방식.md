# 의존성 주입과 스프링부트 동작방식
참고] 인프런 김영한 스프링 강의 + 스프링 프레임워크 첫걸음

1. 스프링부트 동작방식
2. 의존성 주입
3. 다섯가지 규칙
4. 컴포넌트 스캔 : 스프링 빈과 의존관계
5. DI의 3가지 방법
6. 컴포넌트 스캔과 설정을 통한 스프링 빈 등록 선택기준

# 1. 스프링부트 동작방식

1. @SpringbootApplication
    1. Application의 main 메서드에 붙임
        - 해당 패키지와 하위 패키지를 컴포넌트 스캐닝 > 스프링 빈으로 자동등록
    2. 컴포넌트 스캐닝 대상 어노테이션
        - @Service, @Controller, @Repository, @Component
2. @PersistenceContext
    1. 스프링이 생성한 JPA EntityManager를 주입해줌
        ```
        @PersistenceContext
        private EntityManager em;
        ```
    2. EntityManager팩토리를 주입받고 싶다면 @PersistenceUnit 사용
        ```
        @PersistenceUnit
        private EntityManagerFactory emf;
        ```

# 2. 의존성 주입 
- 의존하는 부분을 외부에서 주입하는 것. 
    - 의존하는 부분 
        - '사용하는 객체' 클래스에 '사용되는 객체' 클래스가 작성된 상태
    - 외부로부터 주입 
        - '사용하는 객체' 클래스의 밖에서 '사용되는 객체' 인스턴스를 주입
- 인스턴스 생성 작업을 프레임워크에 맡김(new키워드를 사용하지 않음)
    - 그 역할을 하는 것이 DI컨테이너

        ```
        [인터페이스]
        메서드의 구체적인 처리내용을 작성하지 않는 추상 메서드 작성.

        1. 구현 키워드
        - 인터페이스를 구현할 때는 implements 키워드 사용
        - 인터페이스가 인터페이스를 구현할 때는 extends 키워드 사용

        2. 인터페이스로 정의되는 추상메서드는 모두 구현해야 함
        - 구현하지 않으면 컴파일 에러가 발생

        3. 구현할 때 public을 선언해야
        - 인터페이스의 추상메서드는 암묵적으로 public abstract 한정자가 붙음

        4. @Override
        - 슈퍼 클래스나 인터페이스의 메서드를 상속 혹은 구현하는 클래스에서 재정의하는 것
        - 메서드에 부여하는 것으로 "이것은 재정의(Override)된 메서드입니다. 만약 재정의되어 있지 않으면 에러가 발생합니다"라는 것을 알려줌
        ```

        ```
        [예시1 - 생성자를 통해 구현클래스 주입]
        [예시2 - 인터페이스를 통해 구현클래스 주입]
        ```

# 3. 다섯가지 규칙
1. 인터페이스를 이용하여 의존성을 만든다.
2. 인스턴스를 명시적으로 생성하지 않는다
3. 어노테이션을 클래스에 부여한다
    - '사용하는 객체' 클래스 
        - @SpringBootApplication
            - spring Initioalizer에서 스프링부트 프로젝트를 생성하면 기본적으로 사용자가 만든 프로젝트 이름 + Application 클래스가 생성됨
            - 이 클래스에는 @SpringBootApplication 주석이 부여됨
            - 메인 메서드를 포함한 클래스에 @SpringBootApplication 어노테이션을 부여하면 스프링부트 애플리케이션이라고 인식됨. 
    - '사용되는 객체' 클래스
        - @Component
            - 인터페이스를 구현한 클래스에 @Component를 부여
4. 스프링 프레임워크에서 인스턴스를 생성한다
5. 인스턴스를 사용하고 싶은 곳에 어노테이션을 부여한다. 
    - '사용하는 객체' 클래스의 필드
        - @Autowired
            - 스프링 프레임워크에 의해 생성된 인스턴스를 이용하고 싶은 곳에 참조를 받는 필드를 선언하고 @Autowired 어노테이션 부여

# 4. 컴포넌트 스캔 : 스프링 빈과 의존관계
- @Service 
    - 인스턴스 생성 지시, 트랜잭션 경계가 되는 도메인(서비스) 기능에 부여
    - 도메인 레이어의 업무 처리에 부여
- @Controller 
    - 인스턴스 생성 지시, 스프링MVC를 이용할 때 컨트롤러에 부여
    - 애플리케이션 레이어의 컨트롤러에 부여
- @Repository
    - 인스턴스 생성 지시, 데이터베이스 액세스(리포지토리) 기능에 부여
    - 인프라 레이어의 테이터베이스 액세스 처리에 부여
- @Component
    - 위 용도 이외의 클래스에 부여
- @autowired 
    - 스프링프레임워크에 의해 생성된 인스턴스를 이용하는 클래스에 참조받는 필드를 선언하고 필드에 @autowired 부여


# 5. DI의 3가지 방법

## 1. 필드주입
- 스프링 뜰 때만 넣어주고, 중간에 바꿔치기 할 수 있는 방법이 없음. 

## 2. setter 주입
- 생성 후 세터는 나중에 호출이 되서 멤버 서비스가 들어옴. 
- 누군가가 멤버 컨트롤러를 호출 했을 때 setter가 public으로 열려있어야 함.  
- public에 노출이 됨. 
- 중간에 잘못바꾸면 문제가 생김. 

## 3. 생성자 주입
- 위의 2가지 방식은 각각의 단점으로 생성자 주입을 선호. 
- 처음 어플리케이션 조립될 때 (스프링 컨테이너에 올라가고 세팅이 되는) 생성자가 들어오면 끝남.
- 조립시점에 생성자로 조립해놓고 끝을 내는게 좋음. (변경을 못하도록 막기)
- 의존관계가 실행중에 동적으로 변하는 경우는 없으므로 생성자 주입을 권함. 


# 6. 컴포넌트 스캔과 설정을 통한 스프링 빈 등록 선택기준
1) 컴포넌트 스캔 사용기준
- 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용.

    1-1. 컴포넌트 스캔의 주의할 점

    - @Autowired를 통한 DI는 'helloController','MemberService'등과 같이 스프링이 관리하는 객체에서만 동작. 스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않음. 

2) 자바 코드로 직접 스프링 빈 등록
- 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다. 

# 7. Dependecy Injection 예시
- MemberService의 리포지토리와 MemberServiceTest의 리포지토리가 다르게 설정되어있음. 이걸 같은 리포지토리를 사용하게끔 설정. 
- MemberService입장에서 리포지토리를 new로 생성하는것이 아닌 생성된 것을 외부에서 주입받는 것

```
class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }
```
```
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```


# 8. IoC컨테이너,DI컨테이너
- 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 IoC컨테이너(DI컨테이너)
- JUnit도 IoC
- IoC (제어의 역전)


## @Configuration | @Bean
## ApplicationContext
- 기존에는 개발자가 appConfig를 사용해서 직접 객체 생성했지만 이제는 스프링 컨테이너를 통해 사용
- 인터페이스, 스프링 컨테이너라 함
- 스프링 컨테이너는 xml를 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다

## AnnotationConfigApplicationContext
- ApplicationContext 인터페이스의 구현체
- 빈을 컨테이너에 등록해서 관리해줌


# 9. 스프링 컨테이너와 스프링 빈

## 1 스프링 컨테이너와 스프링 빈
1. 스프링 컨테이너를 생성하고 설정정보를 참고해서 스프링 빈을 등록, 의존관계를 설정함.
    - 스프링은 빈을 생성하고 의존관계를 주입하는 단계가 나누어져 있음
    - 그런데 생성자를 호출하면서 의존관계 주입도 한번에 처리됨
    - @Component 단계
2. 스프링 컨테이너에서 데이터를 조회
    - ac.getBeanDefinitionNames() : 스프링에 등록된 모든 빈 이름 조회
    - ac.getBean() : 빈 이름으로 빈 객체(인스턴스)를 조회
    - @Autowired 단계
    
<br>    
<br>

## 2 스프링 빈 조회 
1. 기본
- 스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법
    1. ac.getBean(빈이름, 타입)
    2. ac.getBean(타입)
2. 동일한 타입이 둘 이상일 경우
    - 이름을 붙여준다

3. 상속관계 (중요)
- 부모 타입으로 조회하면 자식타입도 함께 조회한다
- 그래서 모든 자바 객체의 최고 부모인 Object 타입으로 조회하면 모든 스프링 빈을 조회한다.

<br>
<br>

## 3. BeanFactory와 ApplicationContext
- BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다. 

### 1. BeanFactory 
- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다. 
- getBean() 을 제공한다.
- 지금까지 우리가 사용했던 대부분의 기능은 BeanFactory가 제공하는 기능이다. 

### 2. ApplicationContext 
- BeanFactory 기능을 모두 상속받아서 제공한다.
- 빈을 관리하고 검색하는 기능을 BeanFactory가 제공해주는데, 그러면 둘의 차이가 뭘까? 
- 애플리케이션을 개발할 때는 빈은 관리하고 조회하는 기능은 물론이고, 수 많은 부가기능이 필요하다. 
    1. 메시지소스를 활용한 국제화 기능 
    - 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력 
    2. 환경변수 
    - 로컬, 개발, 운영등을 구분해서 처리 
    3. 애플리케이션 이벤트 
    - 이벤트를 발행하고 구독하는 모델을 편리하게 지원 
    4. 편리한 리소스 조회 
    - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회 

<br>
<br>

## 4. 스프링 빈 설정 메타 정보 - BeanDefinition
- BeanDefinitio을 빈 설정 메타정보라 한다
    - @Bean, <bean> 당 각각 하나씩 메타 정보가 생성됨
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다
