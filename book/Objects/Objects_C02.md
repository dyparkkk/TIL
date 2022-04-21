# Objects_C02

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/objects2-00.png)

- 어떤 클래스가 필요한지를 고민하기 전에 어떤 객체들이 필요한지 고민해라
- 클래스의 경계의 명확성은 객체의 자유를 보장한다 → 프로그래머에게 구현의 자유 제공

### 객체들의 공동체

```java
public class Money {
    public static final Money ZERO = Money.wons(0);

    private final BigDecimal amount;

    public static Money wons(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public static Money wons(double amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public Money(BigDecimal amount) {
        this.amount = amount;
    }

    public Money minus(Money amount) {
        return new Money(this.amount.subtract(amount.amount));
    }

    public Money plus(Money amount) {
        return new Money(this.amount.add(amount.amount));
    }

    public Money times(double percent) {
        return new Money(this.amount.multiply(
                BigDecimal.valueOf(percent)
        ));
    }

    public boolean isLessThen(Money other) {
        return amount.compareTo(other.amount) < 0;
    }

    public boolean isGreaterThanOrEqual(Money other) {
        return amount.compareTo(other.amount) >= 0;
    }
}
```

- 하나의 인스턴스 변수만 포함하더라도 개념을 명시적으로 표현해줌
    - 전체적인 설계의 명확성과 유연성을 높임
- 금액과 관련된 로직이 서로 다른 곳에 중복되어 구현되는 것을 막아줌

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/objects2-01.png)

- 메시지와 메서드는 다르다
- 메시지는 객체들의 상호작용 방법이고 메서드는 수신된 메시지를 처리하기 위한 자신만의 방법
- 다형성과 관련

```java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    // movie 생성자는 하나의 discountPolicy만 허용 !!
    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

- 어떤 할인 정책을 사용할 것인지를 결정하는 코드가 없음
- **단지 discountPolicy에게 메시지를 전송할 뿐임**

### 할인 정책과 할인 조건

```java
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return Money.ZERO;
    }

    abstract protected Money getDiscountAmount(Screening screening);
}
```

- 인스턴스를 생성할 필요가 없음 → 추상클래스로 구현
- conditions를 변수로 가짐 → 여러 개의 할일 조건을 포함할 수 있음
- 부모 클래스인 DiscountPolicy가 기본적인 알고리즘의 흐름을 구현
    - 중간에 필요한 처리를 자식 클래스에게 위임 → **Template 패턴**
    

## 상속과 다형성

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/objects2-02.png)

- **컴파일 시간 의존성과 실행 시간 의존성이 다름**
    - 컴파일 시간에는 오직 추상 클래스인 DiscountPolicy에만 의존
    - 실행 시간에는 구현 클래스에게 의존
- 더 유연하고 확장 가능해짐, 하지만 코드를 이해하기 어려워짐 → 트레이드오프 존재
- 상속은 부모와 다른 부분만 추가해서 새로운 클래스를 빠르게 만드는 방법 - 차이에 의한 프로그래밍

### 상속과 인터페이스

- 상속은 부모 클래스가 제공하는 모든 인터페이스를 물려받음
    - 컴파일러는 부모 클래스가 나오는 모든 장소에서 자식 클래스를 이용하는 것을 허용함(업캐스팅)
- 인터페이스는 객체가 이해할 수 있는 메시지의 목록을 정의하는 것
- 상속은 코드 재사용 목적이 아니라 인터페이스 상속을 위해 사용해야 함

### 다형성

- Movie는 동일한 메시지를 전송하지만 어떤 메서드가 실행되는 지는 메시지를 수신하는 객체의 클래스가 무엇이냐에 따라 달라짐 → **다형성**
- **컴파일 시간 의존성과 실행 시간 의존성이 다를 수 있다는 사실에 기반함 → 지연 바인딩, 동적 바인딩**

### 추상화

- 세부적인 내용을 무시한 채 상위 정책을 쉽고 간단하게 표현가능
- 어플리케이션의 흐름을 기술
- 기존 구조를 수정하지 않고 새로운 기능을 쉽게 추가 가능
- 유연성이 필요한 곳에 추상화를 사용하자