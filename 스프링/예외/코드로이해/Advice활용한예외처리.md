# 예외 처리과정
1. 에러를 표현한다
    - throw new RuntimeException("에러를 발생시킨다.");
2. 에러를 잡는다 
    - stackTrace가 없어짐
    ```
    catch (RuntimeException e) {
        //e.printStackTrace();
        System.out.println("발생한 Exception을 잡는다 (캐치)");
        System.out.println(e.getMessage());
    }
    ```
# 1. 에러 반환하는 객체
- dto/ErrorResponse.java

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ErrorResponse {
    private boolean result;
    private String title;
    private String message;

    public static ErrorResponse of(String title, String message) {
        return new ErrorResponse(false, title, message);
    }
}
```

# 2. 에러를 표현하는 커스텀 
1. RuntimeException 상속
2. 
- 예시 : TypeException
```
public class TypeException extends RuntimeException {

    private TypeException(String message) {
        super(message);
    }

    public static TypeException of(String typeName, String name) {
        return new TypeException(String.format("[%s] 해당 이름을 가진 [%s]이/가 존재하지 않습니다.", typeName, name));
    }
}
```
- 예시 : DomainException
```
public class DomainException extends RuntimeException {

    private DomainException(String message) {
        super(message);
    }

    public static DomainException notFoundRow(Long id) {
        return new DomainException(String.format("%s 해당 Row가 존재하지 않습니다.", id));
    }

    public static DomainException notFoundRow(String str) {
        return new DomainException(String.format("%s 해당 Row가 존재하지 않습니다.", str));
    }
}

```
# 3. 서비스에서 에러 잡기
- 예시 1 : StudentService에서 학생 조회
    - Optional의 orElseThrow 메소드가 사용됨(Optional문법)
    ```java
    StudentService.java>>

    @Transactional(readOnly = true)
    public StudentResponse getStudent(Long id) {
        Student student = studentRepository.findById(id)
                .orElseThrow(() -> DomainException.notFoundRow(id));

        return new StudentResponse(student);
    }
    ```
    - 테스트
    ```
    TestService.java>>

    @Transactional
    public TestResponse createTest(Long studentId, Long subjectId, int score) {
        
        Student student = studentRepository.findById(studentId)
                .orElseThrow(() -> DomainException.notFoundRow(studentId));
        Subject subject = subjectRepository.findById(subjectId)
                .orElseThrow(() -> DomainException.notFoundRow(subjectId));
    ```
- 예시 2 : Subject 빌드
    - enum 클래스에 정의되지 않은 상수 사용
        - 참고 : enum타입 필드를 가진 객체를 생성할 때 enum클래스.valueOf()메소드 사용
    ```
    @Builder
    public Subject(String subjectName){
        
        try {
            this.subjectType = SubjectType.valueOf(subjectName);
        } catch (IllegalArgumentException e){
            throw TypeException.of("과목",subjectName);
        }

        this.isDeleted = false;
    }
    ```
# 4. 범용 컨트롤러에서 에러 잡기
## 예외처리 핸들러 메소드
1. 예외발생시 오류 페이지로 내부이동시키거나
2. 오류 메세지(JSON 텍스트 데이터)를 응답으로 제공
    ```java
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public ResponseEntity<ErrorResponse> exception(Exception e){
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ErrorResponse.of("관리자에게 문의주세요.", e.getMessage()));
    }
    ```
**@ControllerAdvice**
- 모든 @Controller 즉, 전역에서 발생할 수 있는 예외를 잡아 처리해주는 annotation

**@ExceptionHandler**
- @Controller, @RestController가 적용된 Bean내에서 발생하는 예외를 잡아서 해당 어노테이션이 사용된 메서드에서 처리해주는 기능을 한다.
- @ExceptionHandler라는 어노테이션을 쓰고 인자로 캐치하고 싶은 예외클래스를 등록해주면 끝난다.



**@ResponseStatus**

```
범용
    advice : 어플리케이션 실행시 발생하는 익셉션을 후순위로 잡아줌
        ExceptionHandler(value속성)
특화
    TypeException
        String.format
```



## 1. 커스텀 에러 사용

```java
CommonAdviceController.java>>

@ControllerAdvice
public class CommonAdviceController {

    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public ResponseEntity<ErrorResponse> exception(Exception e){
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ErrorResponse.of("관리자에게 문의주세요.", e.getMessage()));
    }

    @ResponseBody
    @ExceptionHandler(value = TypeException.class)
    public ResponseEntity<ErrorResponse> typeException(TypeException e){
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ErrorResponse.of("존재하지 않는 과목입니다.", e.getMessage()));
    }
}
```
## 2. 스프링 에러 사용
```
CommonAdviceController.java>>

@ResponseBody
@ResponseStatus(value = HttpStatus.BAD_REQUEST)
@ExceptionHandler(value = MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> nullException(MethodArgumentNotValidException e){

    String errorMessage = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining());

    /*
    e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .filter(x -> x.equals(":Dd")).findFirst().orElseThrow();

    BindingResult bindingResult = e.getBindingResult();
    List<FieldError> fieldErrors = bindingResult.getFieldErrors();

    String errorMessage = "";
    for (FieldError fieldError : fieldErrors) {
        errorMessage += fieldError.getDefaultMessage();
    }
    */

    return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(ErrorResponse.of("잘못된 요청 파라미터입니다.", errorMessage));
}
```



# null 에러
## 1. requestDTO에서 null어노테이션 사용
```
@Data
@AllArgsConstructor
@NoArgsConstructor
@Schema(description = "학생정보")
public class StudentForm {

    @NotNull(message = "이름은 필수값입니다!")
    @Schema(description  = "이름")
    private String name;

    @Schema(description = "나이")
    private Integer age;
}
```

## 2. 스프링에서 제공하는 MethodArgumentNotValidException 활용하여 에러 잡기
```
@ResponseBody
@ResponseStatus(value = HttpStatus.BAD_REQUEST)
@ExceptionHandler(value = MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> nullException(MethodArgumentNotValidException e){
    String errorMessage = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining());

    /*e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .filter(x -> x.equals(":Dd")).findFirst().orElseThrow();

    BindingResult bindingResult = e.getBindingResult();
    List<FieldError> fieldErrors = bindingResult.getFieldErrors();

    String errorMessage = "";
    for (FieldError fieldError : fieldErrors) {
        errorMessage += fieldError.getDefaultMessage();
    }*/

    return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(ErrorResponse.of("잘못된 요청 파라미터입니다.", errorMessage));
}
```
## 3. 서비스에서 null로 들어오면 에러 던지기
```
@Transactional
public SubjectResponse createSubject(SubjectForm request){

    //null 검증
    if (request.getSubject() == null) {
        //throw NullException.notNull(request);
        throw new NullException(request.getSubject()); //접근제어자
    }

    Subject subject = Subject.builder()
            .subject(request.getSubject())
            .build();
    Subject savedSubject = subjectRepository.save(subject);

    return new SubjectResponse(savedSubject);
}
```

# TEST
```
@DisplayName("exception 잡지 않는 경우")
@Test
public void exceptionTest0() {
    throw new RuntimeException("에러를 발생시킨다.");
}


@DisplayName("exception 잡지 않는 경우")
@Test
public void exceptionTest1() {
    init();
}

@DisplayName("exception 잡는 경우")
@Test
public void exceptionTest2() {
    try {
        init();
    } catch (RuntimeException e) {
        //e.printStackTrace();
        System.out.println("발생한 Exception을 잡는다 (캐치)");
        System.out.println(e.getMessage());
    }
}

private static void init() throws RuntimeException{
    throw new RuntimeException("에러를 발생시킨다.");
}
```

## 참고 - 404에러
- 404오류는 다른 오류와 달리 기본적으로 @Controller를 탈 필요가 없기 때문에 Exception으로 처리되지 않는 특성이 있다
- 따라서 404를 프로그래밍적으로 처리하고 싶다면 404발생 시 예외를 발생시키도록 설정해야.
    - 기본적으로 404는 EXCEPTION 상황이 아님
1. DispatcherServlet을 등록할 때 throwExceptionIfHandlerFound 초기화 파라미터를 true로 설정
    - application.properties
    ```
    spring.mvc.throw-exception-if-no-handler-found=true
    spring.web.resources.add-mappings=false
    ```

