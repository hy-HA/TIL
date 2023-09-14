- 참고 
    - 자바 > 객체지향프로그래밍 > 일급컬렉션, 콜바이밸류&콜바이레퍼런스& [옵저버 패턴](https://yuddomack.tistory.com/entry/관찰자observer-패턴과-발행구독publishsubscribe과-프론트엔드)
    - [리액티브 프로그래밍](https://velog.io/@korea3611/리액티브-프로그래밍에-대하여)

# Event
## 이벤트 객체
- 발행되는 이벤트 객체
```java
@Getter
@ToString
public class MemberCreatedMessage {

    private Long id;
    private String username;
    private String to;
    private String subject;
    private String message;
    private String token;
    private String state;
    private String email;

    public MemberCreatedMessage(Member member){
        this.id = member.getId();
        this.to = member.getEmail();
        this.subject = "축하합니다. 가입이 승인되었습니다";
        this.message = String.format("%s님의 가입을 축하합니다", member.getUsername());
        this.username = member.getUsername();
        this.token = member.getToken();
        this.state = member.getState();
        this.email = member.getEmail();
    }
}
```
## 이벤트 리스너
- ApplicationEventPublisher가 발행한 이벤트를 감지하는 객체
- 리스너는 빈으로 생성되어야
- 발행되는 이벤트 객체를 활용해 비즈니스 로직 실행
- 이벤트를 언제 실행할지 결정 가능
    - @TransactionalEventListener로 옵션 설정
        - 커밋 후 이벤트 발행하는 옵션
            - (phase = TransactionPhase.AFTER_COMMIT)
```java
@Component
@Slf4j
@RequiredArgsConstructor
public class MemberEventListener {

    private final JavaMailSender javaMailSender;
    private final TemplateEngine templateEngine;
    private final AppProperties appProperties;
    private final MemberRepository memberRepository;

    @EventListener
    public void info(MemberCreatedMessage event){
        System.out.println(String.format("%s님이 생성되었습니다", event.getUsername()));
    }

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void sendMail(MemberCreatedMessage event) {

        MimeMessage mimeMessage = javaMailSender.createMimeMessage();

        try{
            makeEmailForm(event, mimeMessage);
            javaMailSender.send(mimeMessage);

        } catch (MessagingException e){
            log.error("error");
            rollbackRegistration(event);
        }
        log.info("Success");

    }
}
```
## 이벤트 발행
- ApplicationEventPublisher객체 사용
- ApplicationEventPublisher가 이벤트를 발행하면, 이벤트리스너가 생성됨.
- 리스너가 생성되면 메서드로 이동이 아니라 빈 아이콘으로 이동함
```java
public MemberResponse createMember(MemberRequest.CreateMemberRequest request) {

    ...//회원가입 진행

    Member persistMember = memberRepository.save(member);

    applicationEventPublisher.publishEvent(new MemberCreatedMessage(persistMember));

    return memberMapper.toMemberResponse(persistMember);

}
```
# HistoryResponse로 도메인 생성/수정/삭제 로그 조회하기
