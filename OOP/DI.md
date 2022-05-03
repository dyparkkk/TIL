# DI (dependency injection)

의존관계 주입

> **의존대상 b가 변하면, 그것이 a에 영향을 미친다.**

*이일민, 토비의 스프링 3.1, 에이콘(2012), p113*
> 

예시)

햄버거 요리사는 햄버거 레시피에 의존한다.

```java
class BurgerChef {
    private HamBurgerRecipe hamBurgerRecipe;

    public BurgerChef() {
        hamBurgerRecipe = new HamBurgerRecipe();        
    }
}
```

- 더 다양한 레시피에 의존할 수 있게 의존관계를 인터페이스로 추상화

```java
class BurgerChef {
    private BurgerRecipe burgerRecipe;

    public BurgerChef() {
        burgerRecipe = new HamBurgerRecipe();
        //burgerRecipe = new CheeseBurgerRecipe();
        //burgerRecipe = new ChickenBurgerRecipe();
    }
}

interface BugerRecipe {
    newBurger();
    // 이외의 다양한 메소드
} 

class HamBurgerRecipe implements BurgerRecipe {
    public Burger newBurger() {
        return new HamBerger();
    }
    // ...
}
```

## 의존관계 주입이란 ?

burgerChef가 의존하고 있는 burgerRecipe를 외부에서 결정하고 주입하는 것

의존관계 주입의 3가지 조건

> 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스만 의존하고 있어야 한다.

런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.

*의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

- 이일민, 토비의 스프링 3.1, 에이콘(2012), p114*
> 

## 장점

- 의존성이 줄어듬
    - 주입받는 대상이 변하더라도 구현을 수정하지 않아도 됨
- 재사용성이 높은 코드가 됨
- 테스트하기 좋은 코드가 됨

## 의존관계 주입 방법

- 생성자 주입
- 메서드 주입(setter() 등)
- 필드 주입 (@Autowired를 사용)
    - 외부에서 변경이 불가능함 (테스트가 힘들다는 치명적인 단점 )
    - DI 프레임워크가 없으면 동작 안함
    - @Configuration에서만 사용
    - 사용을 권장하지 않음
- 메서드 주입 -  public void init(...){}

### 옵션 처리

- @Autowired(required=false) : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
- org.springframework.lang.@Nullable : 자동 주입할 대상이 없으면 null이 입력됨
- Optional<> : 자동 주입할 대상이 없으면 Optional.empty 가 입력됨

## 생성자 주입!

- 불변 : 대부분의 의존관계 주입은 어플리케이션 종료 시점까지 변경될 일이 없음
    - 굳이 setter로  열어둘 필요 없음
- 누락 : 수정자 주입의 경우 누락될 수 있음
- final 키워드 : 필드이므로 final을 사용해서 수정이 안되게 할 수 있음
- 

## 조회한 빈이 2개 이상일 경우

- @Autowired의 필드 명을 주입할 클래스 이름과 동일하게 한다
- @Qualifier 애노테이션을 사용해서 명시적으로 주입할 클래스를 지정해준다
- @Primary 애노테이션으로 우선순위를 정해준다

```java
// 주입할 클래스
@Component
@Qualifier("mainDiscountPolicy")
// @Primary
public class RateDiscountPolicy implements DiscountPolicy {

    private int discountPercent = 10;

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.Vip) {
            return price * discountPercent / 100;
        } else {
            return 0;
        }
    }
}

// 주입 받는 클래스
@Component
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository,
                            @MainDiscountPolicy DiscountPolicy discountPolicy) {
														//@Qualifier("mainDiscountPolicy")
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

```

- @MainDiscountPolicy 처럼 @Qualifier를 애노테이션으로 만들어서 사용 가능
    - @Qualifier는 스트링이므로 실수 할 가능성 있음
- @Primary를 사용한다면 명시적으로 지정해주지 않아도 됨 (편리)

### 다형성에 사용되는 빈 조회하기

- 소위말하는 전략 패턴을 간단하게 조회할 수 있음
- 생성자 주입 할 때 스프링 컨테이너가 Map이나 List에 담아주는 것을 편하게 사용하면 됨

```java
		@Test
    void findAllBean(){
        // 스프링 컨테이너에 클래스를 빈으로 등록
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DisCountService.class);
        DisCountService disCountService = ac.getBean(DisCountService.class);
        Member member = new Member(1L, "member1", Grade.Vip);

        int discountPrice = disCountService.discount(member, 10000, "fixDiscountPolicy");
        assertThat(disCountService).isInstanceOf(DisCountService.class);
        assertThat(discountPrice).isEqualTo(1000);

        int discountPrice2 = disCountService.discount(member, 20000, "rateDiscountPolicy");
        assertThat(discountPrice2).isEqualTo(2000);

    }
    
    // 빈으로 등록할 클래스 
    // 생성자 주입 시점에 컨테이너가 자동으로 관련 클래스(빈으로 등록된)을 Map과 List에 담아줌
    static class DisCountService{
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DisCountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policies = " + policies);
        }

        // Map에서 가져오면 됨 ( 상당히 편리함)
        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
```

---
참고 자료 :   
[https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)   
[https://www.inflearn.com/course/스프링-핵심-원리-기본편/](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/)