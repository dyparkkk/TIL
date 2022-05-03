# Component scan


- **빈으로 등록 될 준비를 마친 클래스들을 스캔하여, 빈으로 등록해주는 것**

```java
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION,
        classes = Configuration.class))
public class AutoAppConfig {
}
```

- 스프링 컨테이너는 @Configuration을 파싱함
    - 빈을 등록, 설정 정보로 인식
    - 빈의 싱글톤을 유지
- @ComponentScan으로 빈을 찾아서 등록함
    - 디폴트는 현재 패키지를 기준으로 빈을 찾음
    - @Component가 붙은 클래스를 찾음 → 다시 @Configuration을 찾을 수도 있음

> 참고 :
@SpringBootApplication 에는 @SpringBootConfiguration이 @Configuration을 포함한다.
따라서 별도의 @Configuration을 작성하지 않아도 자동으로 스프링 컨테이너가 위의 역할을 수행함.
> 
- @Autowired가 지정된 필드 주입은 스프링 컨테이너가 등록된 빈 중에서 찾아서 자동 주입해줌
    
    

- 탐색 패키지 지정

```java
@ComponentScan(
	basePackages = "hello.core",
}
```

- 필터 설정

```java
@Configuration
@ComponentScan(
	includeFilters = @Filter(type = FilterType.ANNOTATION, classes =
		MyIncludeComponent.class),
	excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
		MyExcludeComponent.class)
)
static class ComponentFilterAppConfig {
}

```

> 참고 :
옵션을 변경해서 사용할 일이 많지 않음. → 기본 설정에 최대한 맞춰서 사용하는 것이 좋음
- 김영한님
>

---
참고자료 :  
[](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/)  