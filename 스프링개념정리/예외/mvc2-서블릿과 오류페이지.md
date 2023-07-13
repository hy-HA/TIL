| 참고 김영한 스프링mvc2
# 서블릿이 오류페이지를 보여주는 원리
## **순수 서블릿 컨테이너(WAS)가 예외를 알아차리고 처리하는 2가지 방법**

### **1. Exception (예외)**

```
@GetMapping("/error-ex")
public void errorEx() {
    throw new RuntimeException("예외 발생!"); 
}
```
- 자바의 예외 매커니즘
    - 자바의 메인 메서드를 직접 실행하는 경우 main 이라는 이름의 쓰레드가 실행된다.
    - 실행 도중에 예외를 잡지 못하고 처음 실행한 main() 메서드를 넘어서 예외가 던져지면, 예외 정보를 남기고 해당 쓰레드는 종료된다.
- 웹 애플리케이션
    - 웹 애플리케이션은 사용자 요청별로 별도의 쓰레드가 할당되고, 서블릿 컨테이너 안에서 실행된다. 
        - 웹 어플리케이션은 쓰레드가 여러개.(mvc1 참고)
        - http요청이 오면 별도의 스레드 할당됨
        - 그 스레드가 서블릿을 실행하고 컨트롤러까지 돌아다님
    - 애플리케이션에서 예외가 발생했는데, 어디선가 try ~ catch로 예외를 잡아서 처리하면 아무런 문제가 없다. 
- 그런데 만약에 애플리케이션에서 예외를 잡지 못하고, 서블릿 밖으로 까지 예외가 전달되면 어떻게 동작할까?
    - HTTP Status 500 – Internal Server Error 에러 발생
    - tomcat이 기본으로 제공하는 오류 화면이 보임
    ```
    WAS(여기까지 전파된다면?) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
    ```
    

### **2. response.sendError(HTTP 상태 코드, 오류 메시지)**
- 오류가 발생했을 때 HttpServletResponse 가 제공하는 sendError() 메서드 사용 가능 
- 이것을 호출한다고 당장 예외가 발생하는 것은 아니지만, 서블릿 컨테이너에게 오류가 발생했다는 점을 전달할 수 있다.
- 이 메서드를 사용하면 HTTP 상태 코드와 오류 메시지도 추가할 수 있다.
    - response.sendError(HTTP 상태 코드) 
    - response.sendError(HTTP 상태 코드, 오류 메시지)
    - exception은 상태코드와 메시지를 넣을 수 없음
```
@GetMapping("/error-404")
public void error404(HttpServletResponse response) throws IOException {
    response.sendError(404, "404 오류!"); 
}
```

- sendError 흐름
```
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러 (response.sendError())
```
1. response.sendError() 를 호출하면 response 내부에는 오류가 발생했다는 상태를 저장
2. 그리고 서블릿 컨테이너는 고객에게 응답 전에 response 에 sendError() 가 호출되었는지 확인
3. 그리고 호출되었다면 설정한 오류 코드에 맞추어 기본 오류 페이지를 보여준다.

<br>
<br>


## **스프링부트를 활용한 오류페이지 등록**
1. 서블릿 컨테이너(톰캣 등)가 제공하는 기본 예외처리 화면은 고객친화적이지 않음
2. 서블릿이 제공하는 오류 처리 기능 이용
    - 서블릿은 Exception (예외)가 발생해서 서블릿 밖으로 전달되거나, 또는 response.sendError() 가 호출 되었을 때 각각의 상황에 맞춘 오류 처리 기능을 제공
    - 이 기능을 사용하면 친절한 오류 처리 화면을 준비해서 고객에게 보여줄 수 있다.
3. 스프링부트를 활용하여 서블릿 오류페이지 등록
    - 과거에는 web.xml 이라는 파일에 오류 화면을 등록
    - 스프링부트를 통해서 서블릿 컨테이너를 실행하는 경우, 스프링 부트가 제공하는 기능을 사용하여 서블릿 오류 페이지 등록 가능
        ```
        public class WebServerCustomizer implements   WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

            @Override
            public void customize(ConfigurableWebServerFactory factory) {
                
            ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
            
            ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
            
            ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
            
            factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
            }
        }
        ```
        - response.sendError(404)가 발생하면 
            - 스프링부트는 errorPage404 호출 => 컨트롤러가 "/error-Page/404" 호출 
        - response.sendError(500)가 발생하면
            - 스프링부트는 errorPage500 호출 => 컨트롤러가 "/error-Page/500" 호출 
        - RuntimeException 또는 그 자식 타입의 예외가 발생하면
            - 스프링부트는 errorPageEx 호출 => 컨트롤러가 "/error-Page/500" 호출 

## **오류 페이지 작동 원리**
- 서블릿은 Exception (예외)가 발생해서 서블릿 밖으로 전달되거나 또는 response.sendError() 가 호출되었을 때 설정된 오류 페이지를 찾는다.

1. 예외 발생
    ```
    예외 발생 흐름

    WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
    ```
    ```
    sendError 흐름

    WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러 (response.sendError())
    ```
2. WAS - 해당 예외를 처리하는 오류 페이지 정보를 확인
    - 예를 들어서 RuntimeException 예외가 WAS까지 전달되면, WAS는 등록된 오류 페이지 정보를 확인한다. 
        - new ErrorPage(RuntimeException.class, "/error-page/500")
        - 확인해보니 RuntimeException 오류가 발생한 경우 오류 페이지로 /error-page/500 이 지정되어 있다.
        - WAS는 오류 페이지를 출력하기 위해 /error-page/500 를 다시 요청한다.
3. WAS - 오류 페이지 출력 위해 /error-page/500 를 다시 요청
    - 오류 정보 추가 기능
        - WAS는 오류 페이지를 단순히 다시 요청만 하는 것이 아니라, 오류 정보를 request 의 attribute 에 추가해서 넘겨준다.
        - 필요하면 오류 페이지에서 이렇게 전달된 오류 정보를 사용할 수 있다.
4. 예외 발생과 오류 페이지 요청 흐름
    - 중요한 점은 웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어나는지 전혀 모른다는 점이다. 
    - 오직 서버 내부에서 오류 페이지를 찾기 위해 추가적인 호출을 한다.
```
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error- page/500) -> View
```

## **필터~인터셉터까지 중복 요청되는 문제점**
- 서버 내부에서 오류 페이지를 호출한다고 해서 해당 필터나 인터셉트가 한번 더 호출되는 것은 매우 비효율적

### **필터는 DispatcherType으로 필터 적용여부를 설정가능**
- 서블릿은 DispatcherType 이라는 추가 정보를 제공
    - 클라이언트로 부터 발생한 정상 요청인지, 아니면 오류 페이지를 출력하기 위한 내부 요청인지 구분할 수 있다
```
DispatcherType

REQUEST : 클라이언트 요청 
ERROR : 오류 요청
FORWARD : MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때     
    - RequestDispatcher.forward(request, response);
INCLUDE : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때 
    - RequestDispatcher.include(request, response);
ASYNC : 서블릿 비동기 호출
```
### **인터셉터는 요청 경로에 따라 추가 제외 가능**
- 인터셉터는 요청 경로에 따라서 추가하거나 제외하기 쉽게 되어 있음 
- 이러한 설정을 사용해서 오류 페이지 경로를 excludePathPatterns 를 사용해서 빼주면 된다.

### **전체 흐름 정리**
- /hello 정상 요청
```
WAS(/hello, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러 -> View
```
- /error-ex 오류 요청
    - 필터는 DispatchType 으로 중복 호출 제거 ( dispatchType=REQUEST )
    - 인터셉터는 경로 정보로 중복 호출 제거( excludePathPatterns("/error-page/**") )


```
1. WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
2. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
3. WAS 오류 페이지 확인
4. WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 컨트롤러(/error-page/500) -> View
```

## **스프링부트와 오류페이지**

### 스프링 부트는 이런 과정을 모두 기본으로 제공한다.
- ErrorPage 를 자동으로 등록한다. 이때 /error 라는 경로로 기본 오류 페이지를 설정한다.
    - new ErrorPage("/error") , 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다. 
    - 서블릿 밖으로 예외가 발생하거나, response.sendError(...) 가 호출되면 모든 오류는 /error 를 호출하게 된다.
- BasicErrorController 라는 스프링 컨트롤러를 자동으로 등록한다. ErrorPage 에서 등록한 /error 를 매핑해서 처리하는 컨트롤러다.

> 참고
> ErrorMvcAutoConfiguration 이라는 클래스가 오류 페이지를 자동으로 등록하는 역할을 한다.

### 개발자는 오류 페이지만 등록

### BasicErrorController가 제공하는 기본 정보들