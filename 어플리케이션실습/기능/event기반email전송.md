[개념 참고](https://velog.io/@bagt/Spring-Event-%EA%B8%B0%EB%B0%98-email-%EC%A0%84%EC%86%A1%EA%B3%BC-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B4%80%EB%A6%AC)
[세부코드 참고](https://velog.io/@tjddus0302/Spring-Boot-%EB%A9%94%EC%9D%BC-%EB%B0%9C%EC%86%A1-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-Gmail)

# 목표
1. Spring Event를 통해 비즈니스 로직 간 강결합을 느슨한 결합으로 개선
2. 이후 기능 추가로 다른 비즈니스 로직이 개입되더라도 결합(연관관계)이 추가되는 것이 아닌, listener 추가만으로 해결하여 간결한 비즈니스 로직을 유지
3. 이메일 전송 처리시간으로 인한 사용자의 응답 지연을 @Async(비동기 처리)를 통해 해결
4. @TransactionalEventListener를 통해 기존의 로직과 event 로직 간의 트랜잭션 및 rollback 문제를 해결

# 사용법

## 구글 계정 설정
1. 구글 계정 관리에 들어가서 보안 탭을 클릭한다.
2. 2단계 인증을 설정하지 않았다면 사용하는 것으로 설정을 변경한다.
3. 앱 비밀번호를 설정한다.
    - 앱 선택 목록에서 메일을 선택한다.
    - 기기 선택 목록에서 기타를 선택한 후 기기 이름을 입력한다(원하는대로 작성).
    - 생성 버튼을 눌러 앱 비밀번호를 생성한다.
    - 16자리의 앱 비밀번호를 확인합니다. 앱 비밀번호는 노출되지 않도록 주의하고, 개인 공간에 복사해둔다.
    - 확인버튼을 눌러 완료한다.

## 구글 메일 설정
1. Gmail 설정에 들어가서 전달 및 POP/IMAP 탭으로 들어간다.
2. 모든 메일에 POP를 활성화 하기를 선택한다.
3. IMAP 사용을 선택한다.
4. 변경사항을 저장한다.

## 의존성 & yml 파일 설정
- build.gradle
    - Spring Boot Starter Mail: 메일서버와 연결해서 메일을 발송하는데 필요한 라이브러리
```
implementation 'org.springframework.boot:spring-boot-starter-mail'
```
- application.yml
```
spring:
  mail:
    host: smtp.gmail.com # 1
    port: 587 # 2
    username: ${mail.username} # 3
    password: ${mail.password} # 4
    properties:
      mail:
        smtp:
          auth: true # 5
          timeout: 5000 # 6
          starttls:
            enable: true # 7
```
- application.properties
```
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=Gmail아이디@gmail.com
spring.mail.password=Gmail비밀번호
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

spring.mail.transport.protocol=smtp
spring.mail.debug=true
spring.mail.default.encoding=UTF-8
```
- 인텔리제이 환경변수 설정
1.  Edit Configurations에 들어간다.
2. Environment variables 설정한다
```
mail.username=[@앞까지 기재];mail.password=[16자리 비번입력]
```

# 코드 설정

## 도메인 
- Member클래스
    ```java
    private String email;
    private String token;
    private String state;

    @Builder
    public Member(String username, String email, String pw, int age, String interest, board.member.type.Address address){
        this.username = username;
        this.email = email;
        this.pw = pw;
        this.interest = interest;
        this.address = address;
        this.age = age;
        this.token = "";
        this.state = "NOT_APPROVED";
    }

    public void updateToken(){
        this.token = UUID.randomUUID().toString();
    }

    public void updateStatus(String state){
        this.state = state;
    }
    ```
- DTO클래스
    ```java
    MemberRequest>>

    @Schema(description = "메일주소", required = true, example = "wikihowto99@gmail.com")
    private String email;

    -----------------
    MemberREsponse>>

    @Schema(description = "메일주소", required = true, example = "wikihowto99@gmail.com")
    private String email;
    ```
## 리스너가 감지할 이벤트 클래스
- MemebrCreatedMessage 클래스
    - Spring 4.2 미만에서는 이벤트를 만들기 위해 ApplicationEvent를 상속받아야 한다.
    - Spring 4.2부터는 이벤트가 ApplicationEvent를 상속받을 필요가 없다.
        - 이벤트클래스는 spring 패키지를 전혀 사용하지 않는 POJO(Plain Old Java Object)가 된다.
    - Spring에서 이벤트는 빈으로 등록하지 않는다.
    ```java
    @Getter
    @ToString
    public class MemberCreatedMessage {

        private String username;
        private String to; //수신자
        private String subject; //메일제목
        private String message; //본문내용

        public MemberCreatedMessage(Member member){
            this.to = member.getEmail();
            this.subject = "축하합니다. 가입이 승인되었습니다";
            this.message = String.format("%s님의 가입을 축하합니다", member.getUsername());
            this.username = member.getUsername();
        }
    }
    ```
## 리스너 클래스
- 이벤트 발생시 코드를 실행할 리스너
    - @EventListener 애노테이션을 붙여준다.
    ```java
    @Component
    @RequiredArgsConstructor
    public class MemberEventListener {

        private final JavaMailSender javaMailSender;

        @Async
        @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
        public void sendMail(MemberCreatedMessage event) {

            MimeMessage mimeMessage = javaMailSender.createMimeMessage();

            try{
                makeEmailForm(event, mimeMessage);
                javaMailSender.send(mimeMessage);

            } catch (MessagingException e){
                log.error("error");
            }

            log.info("Success");

        }

        private void makeEmailForm(MemberCreatedMessage event, MimeMessage mimeMessage) throws MessagingException {

            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, false, "UTF-8");
            // 메일 수신자
            mimeMessageHelper.setTo(event.getTo()); 
            // 메일 제목
            mimeMessageHelper.setSubject(event.getSubject()); 

            // 본문 내용 context에 담기
            String token = event.getToken();
            Context context = new Context();
            context.setVariable("link", "/v1/member/check-email-token?token=" + token +
                    "&id=" + event.getId());
            context.setVariable("userName", event.getUsername());
            context.setVariable("linkName", "이메일 인증하기");
            context.setVariable("message", " 챌린저 서비스를 사용하려면 링크를 클릭하세요.");
            context.setVariable("host", appProperties.getHost());
            // 템플릿엔진에 context 연결
            String message = templateEngine.process("mail/simple-link", context);
            
            // 메일 본문 내용, HTML 여부
            mimeMessageHelper.setText(message, true); 
        }
    ```
### 서비스 클래스
- 회원 생성 시 이벤트 리스너가 감지하여 메일 발송
- ApplicationEventPublisher
    - Spring의 ApplicationContext가 상속하는 인터페이스 중 하나이다.
    - 옵저버 패턴의 구현체로 이벤트 프로그래밍에 필요한 기능을 제공해준다.
```java
@Service
@RequiredArgsConstructor
public class MemberService {

    private final ApplicationEventPublisher applicationEventPublisher;

    @Transactional
    public MemberResponse createMember(MemberRequest request) {

        Member member = memberMapper.toMember(request);
        member.updateToken();
        Member persistMember = memberRepository.save(member);

        applicationEventPublisher.publishEvent(new MemberCreatedMessage(persistMember));

        return memberMapper.toMemberResponse(persistMember);

    }
}
```
## 인증메일 발송
- web컨트롤러
    ```java
    @Controller
    @RequiredArgsConstructor
    public class MemberWebController {

        private final MemberService memberService;

        @GetMapping("/v1/member/check-email-token")
        public String checkEmailToken(String token, Long id, Model model){
        return memberService.checkEmailToken(token,id,model);
        }

    }
    ```
- 서비스
    ```java
    @Transactional
    public String checkEmailToken(String token, Long id, Model model) {
        Member member = memberRepository.findById(id).orElseThrow(()-> DomainException.notFindRow(id));

        String view = "mail/checked-email";

        if(!member.getToken().equals(token)){
            model.addAttribute("error", "wrong.email");
            return view;
        }

        //member status 변경
        member.updateStatus("valid");
        model.addAttribute("userName", member.getUsername());
        return view;
    }
    ```
# 객체 설명
## ApplicationEventPublisher
- 생성해놓은 event를 발행하는 trigger 역할
- 발행한 event는 이후 알맞는 event listener에 의해 처리된다.
- 장점
    - 만일 이벤트 없이 여러 service를 호출해야 한다고 생각해보면, service의 개수만큼 의존성이 추가되어 강결합이 발생했을 것이다.
    - 하지만 이벤트 기반으로 처리하면 아래와 같이 ApplicationEventPublisher 하나만으로 처리할 수 있기 때문에, 느슨한 결합이 가능하다.

## ApplicationEvent
- ApplicationEvent를 구현하여 event를 생성하며, event는 ApplicationEventPublisher에 의해 발행되고, EventListener에 의해 처리된다.
- 쉽게 말하면, event 처리에 필요한 정보라고 보면 될 것 같다.

## EventListener
- EventListener는 publisher에 의해 실행된 event를 처리하는 클래스이며, @EventListener 애노테이션을 사용하여 구현할 수 있다.

## 비동기 처리 - @Async
- 이메일 전송은 꽤 시간이 걸리는 동작이며, 이는 회원가입 로직에도 영향을 미친다. 따라서 @Async로 별도의 스레드로 실행하여 비동기 처리하도록 할 수 있다.
- @Async를 사용하기 위해서는 해당 클래스에 @Component, main 클래스에 @EnableAsync가 추가되어야 한다
```java
@EnableAsync
@SpringBootApplication
public class FlyAwayApplication {}
```
```java
@Component
@Slf4j
@RequiredArgsConstructor
public class MemberRegistrationEventListener {

    private final EmailSender emailSender;
    private final MemberRegistrationMessage memberRegistrationMessage;

    @Async
    @EventListener
    public void listen(MemberRegistrationEvent event) {
        sendRegistrationEmail(event);
    }
}
```

# 트랜잭션
- event 기반 로직을 구현할 때에는 항상 트랜잭션 처리를 고려해야 한다.
- 현재 상황에서는 다음 두 가지 경우를 고려할 수 있다.
## 이메일 전송 실패
- 요구사항에 따라 다르겠지만, 이 경우엔 회원 가입이 rollback 되어야 할 수도 있다.
- 이메일 전송에 실패할 경우에 대비하여, try-catch로 이메일 전송 실패 시 member를 삭제 처리하였다.
```java
/**
    * 이메일 전송
*/
public void sendRegistrationEmail(MemberRegistrationEvent event) {
    try {
        String[] to = new String[]{event.getEmail()};
        String message = memberRegistrationMessage.createMessage(event.getName());
        String subject = memberRegistrationMessage.createSubject(event.getName());
        emailSender.sendEmail(to, subject, message);
    } catch (Exception e) {
        e.printStackTrace();
        rollbackRegistration(event);
    }
}

/**
* rollback
*/
public void rollbackRegistration(MemberRegistrationEvent event) {
    log.error("###MailSendException : rollback memberId - {}", event.getId());
    memberRepository.deleteById(event.getId());
    throw new BusinessLogicException(EMAIL_SEND_FAILED);
}
```
## 2. 회원 가입 트랜잭션 실패
- 이것도 정하기 나름이겠지만, 이메일이 전송되지 않도록 구현하려고 한다.
- event 처리에는 성공했지만 기존의 로직이 rollback되는 상황이 발생하는 경우는 어떻게 막을 수 있을까?
- 이는 @TransactionalEventListener를 사용하면 간편하게 처리할 수 있으며, 아래와 같이 @EventListener를 가지고 있는 것을 볼 수 있다.
- @TrransactionalEventListener를 사용하면 이벤트 publish 후 바로 실행되는 것이 아닌, 해당 트랜잭션이 commit되고 난 후에 실행되도록 설정할 수 있다.
```
@EventListener
public @interface TransactionalEventListener{}
```
### @TransactionalEventListener
- phase = TransactionPhase.AFTER_COMMIT 
    - 해당 트랜잭션 commit 후 실행되도록 설정
```java
@Async
@TransactionalEventListener(
    classes = MemberRegistrationEvent.class,
    phase = TransactionPhase.AFTER_COMMIT
)
public void listen(MemberRegistrationEvent event) {
    sendRegistrationEmail(event);
}
```

