참고] 스프링 프레임워크 첫걸음 p73

# AOP의 고유용어
### Advice
- 횡단적 관심사의 구현(메서드). 로그 출력 및 트랜잭션 제어 등
- 5가지 종류
    1. @Before
        - 중심적 관심사 실행되기 이전에 횡단적 관심사를 실행
    2. @AfterReturning
        - 중심적 관심사가 '정상적으로 종료된후'에 횡단적 관심사를 실행
    3. @AfterThrowing
        - 중심적 관심사로부터 '예외가 던져진 후'로 횡단적 관심사를 실행
    4. @After
        - 중심적 관심사 실행된 이후에 횡단적 관심사를 실행
    5. @Around
        - 중앙적 관심사 호출 '전후'에 횡단적 관심사를 실행
### Aspect
- 어드바이스를 정리한 클래스
### JoinPoint
- 어떤 걸 잡을지 명시하는 것
- 모든 정보를 꺼내올 수 있음
- 어드바이스를 중심적인 관심사에 적용하는 타이밍.
- 메서드(생성자) 실행 전, 메서드(생성자) 실행 후 등 실행되는 타이밍

### Ponitcut
- 어드바이스를 삽입할 수 있는 위치
- 예를 들어, 메서드 이름이 get으로 시작할 때만 처리하는 조건을 정의
- 포인트컷 지시자 종류
    1. execution 
        - 메서드 실행 조인 포인트를 매칭 한다. 스프링 AOP에서 가장 많이 사용하며, 기능도 복잡하다.
    2. within 
        - 특정 타입 내의 조인 포인트를 매칭한다.
    3. args 
        - 인자가 주어진 타입의 인스턴스인 조인 포인트
    4. this 
        - 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
    5. target 
        - Target 객체(스프링 AOP 프록시가 가리키는 실제 대상)를 대상으로 하는 조인 포인트
    6. @target 
        - 실행 객체의 클래스에 주어진 타입의 어노테이션이 있는 조인 포인트
    7. @within 
        - 주어진 어노테이션이 있는 타입 내 조인 포인트
    8. @annotation 
        - 메서드가 주어진 어노테이션을 가지고 있는 조인 포인트를 매칭
    9. @args 
        - 전달된 실제 인수의 런타임 타입이 주어진 타입의 어노테이션을 갖는 조인 포인트
    10. bean 
        - 스프링 전용 포인트컷 지시자로 빈의 이름으로 포인트컷을 지정한다.
### Interceptor
- 처리의 제어를 인터셉트하기위한 구조 또는 프로그램
- 스프링 프레임워크에서는 인터셉트라는 메커니즘으로 어드바이스를 중심관심사에 추가한 것처럼 보이게 함
### Target
- 어드바이스가 도입되는 대상


# 포인트컷 식
- 직접 어드바이스를 만드는 경우 패키지, 클래스, 메서드 등 어드바이스 삽입 대상을 조건으로 지정할 수 있음
- 지정하는 조건 방법에는 포인트컷 식을 사용
- 포인트컷 표현식은 여러가지가 있지만 이 문서에서는 'execution'지시자를 설명

## execution 지시자의 구문
- execute(반환값 패키지.클래스.메서드(인수))
- 와일드카드를 이용하여 유연하게 적용 범위를 지정 가능

## 와일드카드
1. ```*```
- 임의의 문자열. 패키지를 나타낼 때는 임의의 패키지 한 계층을 나타냄
- 메서드의 인수에서는 한 개의 인수를 나타내 반환값으로도 이용 가능
2. ```..```
- 패키지를 나타내는 경우 0개 이상의 패키지를 나타냄
- 메서드의 인수를 표현하는 경우는 0개 이상의 임의의 인수를 나타냄
3. ```+```
- 클래스명 뒤에 기술해 클래스와그 서브클래스 및 구현 클래스 모두를 나타냄
 
# 사용법
## 라이브러리 추가
- 스프링 웹이 설치되어있으면 자동으로 추가됨
## @EnableAspectAutoProxy
- @SpringBootApplication 또는 @Configuration을 사용한 클래스에 @EnableAspectAutoProxy를 추가하면 Spring AOP를 사용이 가능
```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class TestApplication {

	public static void main(String[] args) {
		SpringApplication.run(TestApplication.class, args);
	}
}

```

## **클래스에 @Aspect 빈 등록**
- 스프링 빈에 등록된 클래스에서 @Aspect를 추가하면 해당 빈이 Aspect로 작동
- 클래스에 @Order를 명시하여 @Aspect로 등록된 빈 간의 작동 순서를 정할 수 있음
    - 순서는 int 타입의 정수로 순서를 정할 수 있는데 값이 낮을수록 우선순위가 높음.
    - 순서의 기본값은 가장 낮은 우선순위를 가지는 Ordered.LOWEST_PRECEDENCE
- @Aspect가 등록된 빈에는 어드바이스(Advice)라 불리는 메서드를 작성 가능
- 대상 스프링 빈의 method의 시점과 방법에 따라 @Before, @After, @AfterReturning, @AfterThrowing, @Around 등을 명시

## **어느 시점을 잡을지 타겟팅하기**
### @Around && @Pointcut
- @within
    - 클래스
- @@annotation
    - 메소드
```java
@Around(value = "restController() && getMapping()")
public Object endpoint(ProceedingJoinPoint joinPoint) { ... }

@Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
protected void getMapping() {}

@Pointcut("@within(org.springframework.web.bind.annotation.RestController)")
protected void restController() {}
```

## **실행 시점 정하기**
### @Before
- @Aspect 클래스의 method 레벨에 @Before 어드바이스를 명시하면 대상 method 실행 전에 원하는 작업을 설정
- 끼어들기만 할 뿐 대상 메써드의 제어나 가공은 불가능
```java
@Before("execution(* com.example.*.*(..))")
public void somethingBefore(JoinPoint joinPoint) {
}
```
- 어드바이스 parameter의 내용은 PointCut이라는 표현식
- 끼어들 메써드의 범위를 지정
- 전달 받는 JoinPoint 오브젝트는 method의 정보를 담고 있음
### @AfterReturning
- @AfterReturning을 설정하면 해당 method의 실행이 종료되어 값을 리턴할 때
```
@AfterReturning(pointcut = "execution(* com.example.*.*(..))", returning = "result")
public void doSomethingAfterReturning(JoinPoint joinPoint, Object result) {
	// 리턴 시 실행
}
```

# JoinPoint 객체
- JoinPoint객체는 각 컨트롤러에 정의된 method들의 args(매개변수)정보를 담고 있는 객체
### 사용법
1. 포인트 컷에 해당하는 정보가 JoinPoint에 있음
2. JoinPoint를 호출하는 메소드를 통해 result에 결과를 담음.

# 포인트컷
- 관심 조인 포인트를 결정
- 어드바이스가 실행되는 시기를 제어할 수 있다.
### 포인트컷 지시자


# @Around
- @Around 어드바이스는 @Before @After 기능을 모두 포괄
- 대상 method를 실행 앞/뒤 시점에 원하는 작업 설정 가능
```java
// 특정 어노테이션이 명시된 모든 메써드의 실행 전후로 끼어들 수 있다.
@Around("@annotation(sampleAnnotation)")
public Object somethingAround(final ProceedingJoinPoint joinPoint, final SomeAnnotation someAnnotation) {

    // 실행 전
    Object result = joinPoint.proceed();
    // 실행 후

    return result;
}
// 특정 어노테이션이 설정된 모든 method 실행 앞/뒤
@Around("@annotation(scheduled)")
public Object process(final ProceedingJoinPoint joinPoint, final Scheduled scheduled) throws Throwable {
}

// 클래스 레벨에 특정 어노테이션이 명시된 모든 method
@Around("@within(org.springframework.web.bind.annotation.RestController) || @within(org.springframework.stereotype.Service) || @within(org.springframework.stereotype.Repository)")
public Object process(final ProceedingJoinPoint joinPoint) throws Throwable {
}
```