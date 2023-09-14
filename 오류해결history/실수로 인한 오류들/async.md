- 오류상황
    - 회원 가입 시 메일발송하는 이벤트리스너 실행
    - 이때, 메일 실패 시 회원가입 롤백
    - 하지만 메일 실패시 회원가입은 안되는데 스웨거 response에는 찍힘.
- 원인
    - 생명주기가 다른 것을 혼용하고 있음
    - async이랑 롤백은 같이 쓰면 안됨
- 생명주기를 어떻게 정하는지 이슈
    1. 데이터 저장은 하고, 이벤트 발행 오류는 무시
        - 메일 전송 실패시 무시한다   
    2. 이벤트 발행이 안되면 롤백한다
        - 이때 롤백 에러 메시지는 500대 에러면 됨
        - 이때 비동기로 하면 안됨. 동기로 해야.
        - 그래서 스웨거에 찍히는 것
        
- 접근방법
     - 유효성 검사로 푸는게 좋음
    - 혹은 메일 서버에 문제가 있어서 전송이 안되는 경우도 있음. 
```java
@Async
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void sendMail(MemberCreatedMessage event) {

    MimeMessage mimeMessage = javaMailSender.createMimeMessage();

    try{
        makeEmailForm(event, mimeMessage);
        javaMailSender.send(mimeMessage);

    } catch (MessagingException e){
        log.error("error");
        rollbackRegistration;
    }

    log.info("Success");

}

private void makeEmailForm(MemberCreatedMessage event, MimeMessage mimeMessage) throws MessagingException {
    //..
}

// 인증 실패 시 회원가입 취소 -> 가입은 시키되 status로 분류로 변경
private void rollbackRegistration(MemberCreatedMessage event) {
    log.error(String.format("%d 님의 메일전송 오류. 롤백합니다.",event.getId()));
    memberRepository.deleteById(event.getId());
    throw BusinessLogicException.sendingEmailException(event.getId());
}
```