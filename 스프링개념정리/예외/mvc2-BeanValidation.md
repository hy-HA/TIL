| 김영한 mvc2

# Bean Validation
- 검증 로직을 모든 프로젝트에 적용할 수 있게 공통화하고, 표준화 한 것
    - 검증 애노테이션과 여러 인터페이스의 모음
    - 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준
    - Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator
1. 의존관계 추가
    - 'org.springframework.boot:spring-boot-starter-validation'
2. `스프링부트`가 `글로벌 Validator`를 자동으로 등록하고 @Valid 혹은 @Validated이 등록돤 컨트롤러의 검증을 수행
    - `글로벌 validator`는 @NotNull같은 어노테이션을 보고 검증 수행
3. `글로벌 validator`는 검증 오류 발생 시 에러 객체를 BindingResult에 담아준다. 
4. 검증에 성공한 필드만 Bean Validation 적용

# 스프링 MVC가 Bean Validator를 검증하는 원리
1. 스프링 부트 
    1. spring-boot-starter-validation 라이브러리를 설치하면,
        - 자동으로 Bean Validator를 인지하고 스프링에 통합한다.
    2. 자동으로 글로벌 Validator로 등록
        - LocalValidatorFactoryBean 을 글로벌 Validator로 등록
            - 이 글로벌 Validator는 @NotNull 같은 애노테이션을 보고 검증을 수행.
    3. 검증할 컨트롤러에 @Valid 혹은 @Validated 적용
        - 글로벌 Validator가 @Valid 혹은 @Validated 적용된 컨트롤러에서 @NotNull 같은 애노테이션을 보고 검증을 수행
2. 글로벌 Validator(LocalValidatorFactoryBean) (공룡같은 놈)
    1. 검증 오류가 발생하면, FieldError , ObjectError 를 생성해서 BindingResult 에 담아준다
        1. @ModelAttribute 각각의 필드에 타입 변환 시도 
            1. 성공하면 2번으로
            2. 실패하면 typeMismatch 로 FieldError 추가 
        2. Validator 적용
    2. 참고
        - 바인딩에 성공한 필드만 Bean Validation 적용
            - BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.
            - 생각해보면 타입 변환에 성공해서 바인딩에 성공한 필드여야 BeanValidation 적용이 의미 있다.


# Bean Validation의 기본 오류 메시지를 자세히 변경하기

1. Bean Validation을 적용하고 bindingResult 에 등록된 검증 오류 코드를 보자.
    ```
    if(bindingResult.hasErrors()){
        log.info("errors={}", bindingResult);
        return "validation/v3/addForm";
    }
    ```
    - 오류 코드가 애노테이션 이름으로 등록된다. 마치 typeMismatch 와 유사
    - NotBlank 라는 오류 코드를 기반으로 MessageCodesResolver 를 통해 다양한 메시지 코드가 순서대로 생성
    ```
    @NotBlank
        NotBlank.item.itemName 
        NotBlank.itemName 
        NotBlank.java.lang.String 
        NotBlank
    @Range
        Range.item.price 
        Range.price 
        Range.java.lang.Integer 
        Range
    ```
2. 메시지 등록
- errors.properties
    - {0} 은 필드명이고, {1} , {2} ...은 각 애노테이션 마다 다르다.
```
#Bean Validation 추가 
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용 
Max={0}, 최대 {1}
```
- BeanValidation 메시지 찾는 순서
    1. 생성된 메시지 코드 순서대로 messageSource 에서 메시지 찾기
        - NotBlank.item.itemName 
        - NotBlank={0} 공백X 
    2. 애노테이션의 message 속성 사용 
        - @NotBlank(message = "공백! {0}") 
    3. 라이브러리가 제공하는 기본 값 사용 
        - 공백일 수 없습니다.

# 오브젝트 관련 오류(ObjectError) 처리방법
- Bean Validation에서 특정 필드( FieldError )가 아닌 해당 오브젝트 관련 오류( ObjectError )는 어떻게 처리할 수 있을까?

- @ScriptAssert() 를 사용
```
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >=
10000")
    public class Item {
    //...
}
```

- 메시지 코드
    ScriptAssert.item
    ScriptAssert

- 그런데 실제 사용해보면 제약이 많고 복잡하다. 그리고 실무에서는 검증 기능이 해당 객체의 범위를 넘어서는 경우들도 종종 등장하는데, 그런 경우 대응이 어렵다.

- 따라서 오브젝트 오류(글로벌 오류)의 경우 @ScriptAssert 을 억지로 사용하는 것 보다는 다음과 같이 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것을 권장

```
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult
bindingResult, RedirectAttributes redirectAttributes) {

    //특정 필드 예외가 아닌 전체 예외
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }
    }

    if (bindingResult.hasErrors()) {
        log.info("errors={}", bindingResult);
        return "validation/v3/addForm";
    } 
```
# BeanValidation의 groups 기능
- 데이터를 등록할 때와 수정할 때는 요구사항이 다를 경우
```
등록시 - quantity 수량을 최대 9999까지 등록
수정시 - 수량을 무제한으로 변경 가능

등록시 - id 에 값이 없어도 되지만
수정시 - id 값이 필수인 경우
```
- 해결방안
    1. BeanValidation의 groups 기능을 사용
    2. Form 전송 객체 분리
        - Item을 직접 사용하지 않고, 
        - ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용

### BeanValidation의 groups 기능을 사용
```
[저장용 groups 생성]

public interface SaveCheck { }

[수정용 groups 생성]

public interface UpdateCheck { }
```
```
@Data
public class Item {
    
    @NotNull(groups = UpdateCheck.class) //수정시에만 적용 
    private Long id;
    
    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;
    
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class,
UpdateCheck.class})
    private Integer price;
    
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용 private Integer quantity;
```
```
[저장Controller]  - 저장 로직에 SaveCheck Groups 적용

@PostMapping("/add")
public String addItemV2(
        @Validated(SaveCheck.class) @ModelAttribute Item item, 
        BindingResult bindingResult, 
        RedirectAttributes redirectAttributes) {
    
    //...
} 

[수정Controller] - 수정 로직에 UpdateCheck Groups 적용

@PostMapping("/{itemId}/edit")
public String editV2(
        @PathVariable Long itemId, 
        @Validated(UpdateCheck.class)@ModelAttribute Item item, 
        BindingResult bindingResult) {

    //...
}
```

# Form 전송 객체 분리
- 사실 groups 기능은 실제 잘 사용되지는 않는데, 그 이유는 실무에서는 주로 다음에 등장하는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문

### 폼 데이터 전달에 Item 도메인 객체 사용
HTML Form -> Item -> Controller -> Item -> Repository

- 장점: 
    - Item 도메인 객체를 컨트롤러, 리포지토리 까지 직접 전달해서 중간에 Item을 만드는 과정이 없어서 간단하다.
- 단점: 
    - 간단한 경우에만 적용할 수 있다. 수정시 검증이 중복될 수 있고, groups를 사용해야 한다.

### 폼 데이터 전달을 위한 별도의 객체 사용
HTML Form -> ItemSaveForm -> Controller -> Item 생성 -> Repository

- 장점: 
    - 전송하는 폼 데이터가 복잡해도 거기에 맞춘 별도의 폼 객체를 사용해서 데이터를 전달 받을 수 있다. 
    - 보통 등록과, 수정용으로 별도의 폼 객체를 만들기 때문에 검증이 중복되지 않는다. 
- 단점: 
    - 폼 데이터를 기반으로 컨트롤러에서 Item 객체를 생성하는 변환 과정이 추가된다.
        ```
        Item item = new Item(); 
        item.setItemName(form.getItemName()); 
        item.setPrice(form.getPrice()); 
        item.setQuantity(form.getQuantity());

        Item savedItem = itemRepository.save(item);
        ```
        - 폼 객체의 데이터를 기반으로 Item 객체를 생성한다. 
        - 이렇게 폼 객체 처럼 중간에 다른 객체가 추가되면 변환하는 과정이 추가된다.

# Bean Validation - HTTP 메시지 컨버터
- @Valid , @Validated 는 HttpMessageConverter ( @RequestBody )에도 적용할 수 있다.

> 참고

> @ModelAttribute 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용한다.

> @RequestBody 는 HTTP Body의 데이터를 객체로 변환할 때 사용한다. 주로 API JSON 요청을 다룰 때 사용한다.

### API의 경우 3가지 경우를 나누어 생각해야 한다.
1. 성공 요청: 성공
2. 실패 요청: JSON을 객체로 생성하는 것 자체가 실패함
    1. HttpMessageConverter에서 요청 JSON을 ItemSaveForm 객체로 생성하는데 실패한다.
    - 이 경우는 ItemSaveForm 객체를 만들지 못함 
    - 컨트롤러 자체가 호출되지 않고 그 전에 예외가 발생 
    - 물론 Validator도 실행되지 않는다.
        - 실패 요청 결과
            ```
            {
            "timestamp": "2021-04-20T00:00:00.000+00:00",
            "status": 400,
            "error": "Bad Request",
            "message": "",
            "path": "/validation/api/items/add"
            }
            ```
3. 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함
    - HttpMessageConverter 는 성공하지만 검증(Validator)에서 오류가 발생하는 경우
    1. return bindingResult.getAllErrors(); 는 ObjectError 와 FieldError 를 반환
    2. 스프링이 이 객체를 JSON으로 변환해서 클라이언트에 전달 
    - 여기서는 예시로 보여주기 위해서 검증 오류 객체들을 그대로 반환했다. 
        - 실제 개발할 때는 이 객체들을 그대로 사용하지 말고, 필요한 데이터만 뽑아서 별도의 API 스펙을 정의하고 그에 맞는 객체를 만들어서 반환해야 한다.
    - 검증 오류 결과
        ```
        {
            "codes": [

                "Max.itemSaveForm.quantity",
                "Max.quantity",
                "Max.java.lang.Integer",
                "Max"
            ],
            "arguments": [
                {
                    "codes": [
                        "itemSaveForm.quantity",
            "quantity"
                    ],
                    "arguments": null,
                    "defaultMessage": "quantity",
                    "code": "quantity"
                },
                9999
            ],
            "defaultMessage": "9999 이하여야 합니다",
            "objectName": "itemSaveForm",
            "field": "quantity",
            "rejectedValue": 10000,
            "bindingFailure": false,
            "code": "Max"
        }
        ```

### @ModelAttribute vs @RequestBody
- @ModelAttribute 
    - HTTP 요청 파리미터를 처리하는 @ModelAttribute
    - 필드 단위로 정교하게 바인딩이 적용된다. 
    - 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있다.
        - 특정 필드에 타입이 맞지 않는 오류가 발생해도 나머지 필드는 정상 처리할 수 있었다. 
- @RequestBody 
    - HttpMessageConverter 는 @ModelAttribute 와 다르게 각각의 필드 단위로 적용되는 것이 아니라, 전체 객체 단위로 적용
    - 따라서 메시지 컨버터의 작동이 성공해서 ItemSaveForm 객체를 만들어야 @Valid , @Validated 가 적용된다.
    - HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생한다. 
    - 컨트롤러도 호출되지 않고, Validator도 적용할 수 없다.
