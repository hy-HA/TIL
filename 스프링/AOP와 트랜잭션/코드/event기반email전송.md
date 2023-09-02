[참고](https://velog.io/@bagt/Spring-Event-%EA%B8%B0%EB%B0%98-email-%EC%A0%84%EC%86%A1%EA%B3%BC-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B4%80%EB%A6%AC)

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

## 메일 필드 추가
- Member클래스
```java
private String email;
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
```java
@Getter
@ToString
public class MemberCreatedMessage {

    private String username;
    private String to;
    private String subject;
    private String message;

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
```java
@Component
@RequiredArgsConstructor
public class MemberEventListener {

    private final JavaMailSender javaMailSender;

    @EventListener
    public void sendMail(MemberCreatedMessage event) {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        try {
            MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, false, "UTF-8");
            mimeMessageHelper.setTo(event.getTo()); // 메일 수신자
            mimeMessageHelper.setSubject(event.getSubject()); // 메일 제목
            mimeMessageHelper.setText(event.getMessage(), true); // 메일 본문 내용, HTML 여부
            javaMailSender.send(mimeMessage);

            log.info("Success");


        } catch (MessagingException e) {
            log.info("fail");
            throw new RuntimeException(e);
        }
    }
}

```