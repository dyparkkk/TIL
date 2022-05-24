# Spring MVC

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/springMvc00.png)

## DispatcherServlet

- 프론트 컨트롤러 패턴으로 구현, 스프링 MVC의 핵심
- 부모클래스에서 HttpServlet을 상속받아서 사용
- 요청 흐름
    - 부모인 FrameworkServlet.service()에서 시작
    - DispatcherServlet.doDispatch()가 호출 → 핵심로직
- DispatcherServlet의 수정 없이 원하는 기능을 추가 할 수 있음 → 스프링 MVC의 장점
    - 애노테이션 기반의 컨트롤러도 쉽게 추가 가능

```java
// 대부분 생략한 doDispatch()
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		ModelAndView mv = null;
			
		// Determine handler for the current request.
		mappedHandler = getHandler(processedRequest);

		// Determine handler adapter for the current request.
		HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

		// Actually invoke the handler.
		mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

		// 이 안에서 뷰 렌더링 ( 뷰 리졸버 호출 포함)
		processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);

	}
```

컨트롤러 호출을 위해 거치는 과정

### HandlerMapping

- 핸들러 매핑에서 컨트롤러를 찾아야 함

### HandlerAdapter

- 핸들러 매핑에서 찾은 핸들러(컨트롤러)를 실행 할 핸들러가 있어야 함

현재는 ReqeustMappingHandlerMapping과 RequestMappingHandlerAdapter가 가장 우선순위이며 이미 구현되어 있음 

### ViewResolver

- HandlerAdapter로부터 반환된 ModelAndView의 이름에 맞는 ViewResolver를 찾아서 호출
- 뷰를 반환
    - 뷰에서 렌더링 실행

### @Controller

- componentScan의 대상이 됨
- 스프링 MVC에서 애노테이션 기반의 컨트롤러로 인식
    - 핸들러로 등록되고, 핸들러어댑터 사용이 가능

### @RequestMapping

- 요청정보를 매핑 → 해당 URL이 호출되면 이 메서드가 실행됨

### @RequestParam

- 요청 파라미터를 편하게 처리
- required = false 일때 null로 처리됨 → int 사용시 오류 발생
    - Integer로 처리
- @RequestParam Map<String, Object> paramMap 을 사용해서 여러가지 파라미터 처리 가능

```java
		@ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = false, defaultValue = "guest") String username,
            @RequestParam(required = false) Integer age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
```

 

### @ModelAttribute

- 객체를 생성 후 요청 파라미터 값을 세팅
    - 객체 생성 → 필드에 set 메서드로 값을 바인딩
- 바인딩 실패 시 BindException 발생
- 생략 가능

```java
		@ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData ) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }
```

### HttpEntity

- Http 헤더, 바디의 정보를 편하게 조회
- 요청 파라미터와 관계 없음
- 응답에서 사용 할 수 있음
- HttpEntity를 상속받은 RequestEntity와 ResponseEntity가 있음
    - RequestEntity : Http 메서드, URL등 추가 정보 제공
    - ResponseEntity : 상태 코드 설정 가능

```java
		@PostMapping("/request-body-string-v3")
    public HttpEntity requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {
        String message = httpEntity.getBody();
        log.info("messageBody = {}", message);

        return new HttpEntity<>("ok");
    }
```

### @RequestBody

- 편리하게 바디 조회 가능
- 참고로 헤더 정보가 필요하다면 @RequestHeader 사용, HttpEntity사용
- application/json 형식의 요청의 경우 객체로 변환 가능
- 생략 불가

```java
// json 요청 처리 		
		@ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData data) {
        log.info("username={}, age={}", data.getUsername(), data.getAge());
        return "ok";
    }
```

### HTTP 메시지 컨버터

- Http 요청에 따라서 다른 처리를 도와주는 컨버터
- 사용
    - 요청 : @RequestBody, HttpEntity
    - 응답 : @ResponseBody, HttpEntity
- ByteArrayHttpMessageConverter - 0순위
    - 클래스타입 : byte[], 미디어타입 : */*
    - ex ) @ResponseBody byte[] data
- StringHttpMessageConverter - 1순위
    - 클래스 타입: String , 미디어타입: **/**
- MappingJackson2HttpMessageConverter
    - 클래스 타입: 객체 또는 HashMap , 미디어타입 application/json 관련
- **순서대로 CanRead 할 수 있는지 확인 …**

### 요청 매핑 핸들러 구조

- @RequestMapping 을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/springMvc01.png)

- 파라미터를 유연하게 처리할 수 있는 이유 → ArgumentResolver
- 스프링에는 30개가 넘는 ArgumentResolver가 미리 등록되어있음
- 반환은 ReturnValueHandler가 처리한다

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/springMvc02.png)

---  
참고자료 :  
https://www.inflearn.com/course/스프링-mvc-1/  