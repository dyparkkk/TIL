# Exception


## Servelt Container

서블릿을 관리, client의 request를 받아주고 response 할 수 있게 해줌. 

서블릿은 2가지 방식으로 예외 처리 지원 

- Exception
- response.sendError(http_status_code, error_message)

## 웹 어플리케이션 예외 처리 과정

**WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)**

was == ( ex. 톰캣)

sendError 흐름

1. response.sendError() 호출하면 response에 오류 발생 상태 저장.
2. 서블릿 컨테이너는 고객에게 응답 전에 response에 sendError()가 호출되었는지 확인.
3. 호출되었다면 설정한 오류 코드에 맞춰 기본 오류 페이지를 보여줌 

## 서블릿 오류 페이지 등록

```java
// 
@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");

        ErrorPage errorPageE = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageE);

    }
}
```

오류에 따라서 그에 맞는 주소를 호출 

ex) RuntimeException → "/error-page/500" 

```java

@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest req, HttpServletResponse res) {
        log.info("errorPage 404");
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest req, HttpServletResponse res) {
        log.info("errorPage 500");
        return "error-page/500";
    }
}
```

## 예외 발생과 오류 페이지 요청 흐름

1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/errorpage/500) → View

## Filter와 DispatcherType

예외 발생의 경우 filter를 두 번 거침 → DispatcherType으로 해결

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

		@Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterRegistrationBean;
    }
}
```

FilterRegistrationBean 에 setDispatcherType으로 error 타입을 거를 수 있음 

!! 인터셉터는 excludePathPatterns을 사용해서 거름 

ex) “error-page/**” ....

## Spring boot - error page 처리

스프링 부트는 **BasicErrorController** 라는 스프링 컨트롤러를 자동으로 등록 

“/error”를 기본값으로 매핑해서 에러를 처리함

- 뷰 템플릿의 resources/templates/error/xxx.html 을 먼저 처리
- 후에 정적리소스 등 처리

## API 예외 처리

spring boot에서는 api 예외도 BasicErrorController에서 처리 

```java
// BasicErrorController
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}
```

클라이언트 요청의 Accept 해더 값이 text/html 인 경우에 errorHtml() 호출 

application/json인 경우 error() 호출로 잘 나누어져 있음 

## HandlerExceptionResolver

![exceptionResolver.PNG](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/exceptionResolver.png)

**exceptionResolver를 사용함으로 예외처리의 흐름을 바꿈**

**→ resolver에서 예외를 마무리 할 수 있음** 

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

        try{
            if (ex instanceof IllegalArgumentException) {
                log.info("IllegalArgumentException resolver to 400");
                response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                return new ModelAndView();
            }

        } catch ( IOException e){
            log.error("resolver ex " , e);
        }
        return null;
    }
}
```

- 빈 ModelAndView:빈 ModelAndView 를 반환하면 뷰를 렌더링 하지않고, 정상 흐름으로 서블릿이 리턴
- ModelAndView 지정: ModelAndView 에 View , Model 등의 정보를 지정해서 반환하면 뷰를 렌더링
- null: null 을 반환하면, 다음 ExceptionResolver 를 찾아서 실행한다. 만약 처리할 수 있는
ExceptionResolver 가 없으면 예외 처리가 안되고, 기존에 발생한 예외를 서블릿 밖으로 던짐

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        resolvers.add(new MyHandlerExceptionResolver());
        resolvers.add(new UserHandlerExceptionResolver());
    }
}
```

exceptionResolver 등록

## spring boot의 기본 제공 exceptionResolver

1. ExceptionHandlerExceptionResolver
2. ResponseStatusExceptionResolver
3. DefaultHandlerExceptionResolver → 우선순위 낮음

### ExceptionHandler annotation

```java
		@ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex ", e);
        return new ErrorResult("BAD", e.getMessage());
    } 
// ErrorResult 따로 만들어준 class

```

IllegalArgumentException  와 그 자식 클래스를 처리하는 handler

!! ExceptionHandler({AException.class, BException.class})

다양한 예외를 한꺼번에 처리할 수 있음 

<br>
참고 자료 : 

https://www.inflearn.com/course/스프링-mvc-2/dashboard - 인프런, spring boot MVC 2( 김영한님 강의 )
