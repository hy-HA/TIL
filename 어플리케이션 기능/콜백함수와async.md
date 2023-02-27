[참고](https://ko.javascript.info/try-catch)
# try/catch와 에러 핸들링

- 아무리 프로그래밍에 능한 사람이더라도 에러가 있는 스크립트를 작성할 수 있습니다. 
    - 원인은 아마도 실수, 예상치 못한 사용자 입력, 잘못된 서버 응답 등의 수천만 가지 이유 때문일 겁니다.

- 에러가 발생하면 스크립트는 ‘죽고’(즉시 중단되고), 콘솔에 에러가 출력됩니다.

- 그러나 try..catch 문법을 사용하면 스크립트가 죽는 걸 방지하고, 에러를 ‘잡아서(catch)’ 더 합당한 무언가를 할 수 있게 됩니다.

# try/catch 문법

- ‘try…catch’ 문법은 'try’와 'catch’라는 두 개의 주요 블록으로 구성됩니다.
    1. try {} 안의 코드가 실행됨
    2. 에러가 없다면, try안의 마지막 줄까지 실행되고, catch 블록은 건너뜀
    3. 에러가 있다면, try안 코드의 실행이 중단되고, catch 블록으로 제어 흐르이 넘어감. 
        - 변수 err(아무 이름이나 사용가능)은 무슨 일이 일어났는지에 대한 설명이 담긴 에러 객체를 포함
- 이렇게 try 블록 안에서 에러가 발생해도 catch에서 에러를 처리하기 때문에 스크립트는 죽지 않음.

```
try {

  // 코드

} catch (err) {
  
  // 에러 핸들링

}
```

# 에러 객체

- 내장 에러 전체와 에러 객체는 두 가지 주요 프로퍼티를 가집니다.

1. name
    - 에러 이름. 정의되지 않은 변수 때문에 발생한 에러라면 "ReferenceError"가 이름이 됩니다.
2. message
    - 에러 상세 내용을 담고 있는 문자 메시지
    - 표준은 아니지만, name과 message 이외에 대부분의 호스트 환경에서 지원하는 프로퍼티도 있습니다. stack은 가장 널리 사용되는 비표준 프로퍼티 중 하나입니다.

3. stack
    - 현재 호출 스택. 에러를 유발한 중첩 호출들의 순서 정보를 가진 문자열로 디버깅 목적으로 사용됩니다.

```
try {
 
  lalala; // 에러, 변수가 정의되지 않음!

} catch(err) {

  alert(err.name); // ReferenceError
  alert(err.message); // lalala is not defined
  alert(err.stack); // ReferenceError: lalala is not defined at ... (호출 스택)

  // 에러 전체를 보여줄 수도 있습니다.
  // 이때, 에러 객체는 "name: message" 형태의 문자열로 변환됩니다.
  alert(err); // ReferenceError: lalala is not defined

}
```

# ‘try…catch’ 사용하기

### 잘못된 형식의 json이 들어온 경우 try..catch를 사용하여 처리하기1
### json이 문법적으로 잘못된 경우

```
let json = "{ bad json }";

try {

  let user = JSON.parse(json); // <-- 여기서 에러가 발생하므로
  alert( user.name ); // 이 코드는 동작하지 않습니다.

} catch (e) {

  // 에러가 발생하면 제어 흐름이 catch 문으로 넘어옵니다.
  alert( "데이터에 에러가 있어 재요청을 시도합니다." );
  alert( e.name );
  alert( e.message );

}
```

### 잘못된 형식의 json이 들어온 경우 try..catch를 사용하여 처리하기2
### json이 문법적으로는 정상, 필수 프로퍼티를 가지고 있지 않은 경우

```
let json = '{ "age": 30 }'; // 불완전한 데이터

try {

  let user = JSON.parse(json); // <-- 에러 없음
  alert( user.name ); // 이름이 없습니다!

} catch (e) {

  alert( "실행되지 않습니다." );

}
```

# throw 연산자를 사용해 에러 처리를 통합
- throw 연산자는 에러를 생성.

- 문법은 다음과 같습니다.
    ```
    throw <error object>
    ```

- SyntaxError에러가 발생하는 경우

```
try {
  JSON.parse("{ 잘못된 형식의 json o_O }");
} catch(e) {
  alert(e.name); // SyntaxError
  alert(e.message); // Unexpected token b in JSON at position 2
}
```
- throw 연산자를 사용해 에러를 던지기
```
let json = '{ "age": 30 }'; // 불완전한 데이터

try {

  let user = JSON.parse(json); // <-- 에러 없음

  if (!user.name) {
    throw new SyntaxError("불완전한 데이터: 이름 없음"); // (*)
  }

  alert( user.name );

} catch(e) {
  alert( "JSON Error: " + e.message ); // JSON Error: 불완전한 데이터: 이름 없음
}
```

# try…catch…finally
- finally 절은 무언가를 실행하고, 실행 결과에 상관없이 실행을 완료하고 싶을 경우 사용됩니다.

```
try {
  // 이곳의 코드를 실행
} catch(err) {
  // 에러가 발생하면, 여기부터 실행됨
  // err는 에러 객체
} finally {
  // 에러 발생 여부와 상관없이 try/catch 이후에 실행됨
}
```