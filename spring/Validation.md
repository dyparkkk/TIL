# Validation

목차

- 컨트롤러의 중요한 역할 중 하나는 HTTP 요청이 정상인지 검증하는 것

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/validation.png)

## BindingResult

- 스프링이 제공하는 검증 오류를 보관하는 객체, 검증 오류가 발생하면 여기에 보관
- BindingResult 가 있으면 @ModelAttribute에 데이터 바인딩 시 **오류가 발생해도 컨트롤러가 호출**

BindingResult에 검증 오류 적용하는 3가지 방법

1.  @ModelAttribute의 객체 타입 오류 등으로 바인딩 실패 시 FieldError 생성해서 넣어줌
2. 개발자가 직접 넣음
3. Validator 사용

!! BindingResult는 매개변수로 검증할 대상 뒤에 와야함.

 → @ModelAttribute 변수 바로 다음에 와야함

### rejectValue()

```java
void rejectValue(@Nullable String field, String errorCode, 
			@Nullable Object[] errorArgs, @Nullable String defaultMessage);
```

- BindingResult 에 error 담기
- reject() 는 글로벌 에러 처리

### MessageCodesResolver

메시지 코드 처리를 도와줌

- 메시지 코드 생성 전략에 따라서 메시지를 만들어줌 !!
- 구체적인 것에서 덜 구체적인 것으로

### Validator

```java
public interface Validator {
	// WebDataBinder에서 현재 class에 맞는 validator를 찾기 위함
	boolean supports(Class<?> clazz);

	// 검증해서 오류를 errors( bindingResult 부모) 에 넣어줌 
	void validate(Object target, Errors errors);
}
```

- validator 를 인터페이스 상속해서 validator를 구현할 수 있음

### WebDataBinder

```java
@InitBinder
    public void init(WebDataBinder dataBinder) {
        dataBinder.addValidators(itemValidator);
    }
```

스프링의 파라미터 바인딩 역할을 해주고 검증기능도 내부에 포함함

- 컨트롤러에 선언해서 사용 (해당 컨트롤러에 적용)

타임리프는 스프링의 BindingResult를 활용해서 검증 오루 표현 기능 제공

- #field : BindingResult 가 제공하는 검증 오류 접근
- th:errors : 해당 필드의 오류가 있는 경우에 태그를 출력함
- th:errorclass : th:field 에 지정한 필드의 오류가 있으면 class 정보를 추가

```html
<!--예시-->
<div>
	<label for="quantity" th:text="#{label.item.quantity}">수량</label>
	<input type="text" id="quantity" th:field="*{quantity}"
		th:errorclass="field-error" class="form-control"
		placeholder="수량을 입력하세요">
	<div class="field-error" th:errors="*{quantity}">
		수량 오류
	</div>
</div>
```

## Bean Validation

Java의 기술 표준 (검증 에노테이션과 여러 인터페이스 모음) 

구현체는 하이버네이트 Validator를 일반적으로 사용함

```java
@Data
public class Item {

    private Long id;

		@NotBlank
    private String itemName;

		@NotNull
		@Range(min = 1000, max = 1000000)
    private Integer price;

		@NotNull
		@Max(9999)
    private Integer quantity;
}
```

검증 에노테이션 참고 : 

[https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

- 스프링 부트는 LocalValidatorFactoryBean 을 글로벌 Validator로 등록함
    
    → 이 Validator 는 @NotNull 등의 에너테이션을 보고 검증 (@Valid, @Validated 만 적용하면 됨)
    

- 바인딩에 성공한 필드만 Bean Validation이 적용됨
    - 실패하면 typeMismatch로 FieldError 추가함
    

### 에러코드

ex) @NotBlank

> NotBlank.item.itemName
NotBlank.itemName
NotBlank.java.lang.String
NotBlank
> 

메시지 등록

```java
// error.properties
NotBlank={0} 공백은 허용되지 않습니다. 
...
```

{0} : 필드명

{1}, {2} ... : 에노테이션마다 다름

 

### 전체 예외 (글로벌 에러)

@ScriptAssert가 있지만 컨트롤러 안에 로직을 짜고 BindingResult에 직접 넣어주는 것이 좋음

 

### HTTP 메시지 컨버터

@Valid, @Validated 는 **HttpMessageConverter (@RequestBody) 에도 적용가능** 

> 참고 
@ModelAttribute  는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)을 다룰 때 사용
@RequestBody 는 Http Body의 데이터를 객체로 변환 할 때 사용(JSON 요청 다룰 때)
> 

```java
// 예시
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }

        log.info("성공 로직 실행");
        return form;
    }
}
```

### API 처리

1. 성공
2. 실패 요청 : JSON을 객체로 생성하는 것 자체가 실패 
    - 컨트롤러가 호출되지 않음 (Validator 적용 불가) → Exception 으로 처리해야함
3. 검증 오류 요청 : Json을 객체로 생성은 했고 검증에서 실패

---

참고자료 : 

Spring MVC 2 - 김영한님 강의