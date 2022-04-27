# Objects_C04

## 설계 품질과 트레이드오프

- 훌륭한 객체지향 설계는 데이터가 아니라 책임에 초점을 맞춰야 함

```java
// 데이터 중심의 영화 예매 구현

/**
 * 데이터 클래스들을 조합해서 영화 예매 절차를 구현하는 클래스
 */
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for (DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                        condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                        condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }
            if (discountable){
                break;
            }
        }

        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch (movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }

            fee = movie.getFee().minus(discountAmount);
        } else {
            fee = movie.getFee();
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

### 캡슐화

- 상태와 행동을 하나의 객체 안에 모음 → 내부 구현을 감추기 위해
    - 내부 구현은 변경될 가능성이 높음
    - 인터페이스는 상대적으로 안정적임 ( 외부에 공개)
- **불안정한 부분과 안정적인 부분을 분리해서 변경의 영향을 통제하기 위해**
- 복잡성을 다루기 위한 효과적인 추상화 방법

### 응집도와 결합도

- 응집도 : 모듈에 포함된 내부 요소들이 연관되어 있는 정도
- 결합도 : 의존성의 정도, 다른 모듈에 대해서 얼마나 많이 알고 있는지 척도
- 변경과 관련이 깊음
    - 응집도가 높을수록, 결합도가 낮을수록 유연한 구조를 가짐

## 데이터 중심 시스템의 문제점

- 캡슐화 위반
    - 결합도와 응집도에 강하게 영향을 줌
- 높은 결합도
    - 내부 구현이 인터페이스에 드러남 → 클라이언트가 구현에 강하게 결합
    - 내부 구현 변경 → 인터페이스에 의존하는 클라이언트들도 변경
- 낮은 응집도
    - 변경 발생 시 상관 없는 코드들이 영향을 받게됨
    - 하나의 요구사항 수정 반영하기 위해 여러 모듈 수정해야 함

```java
// 조금 더 개선된 데이터 중심의 설계
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Money fee = screening.calculateFee(audienceCount);
        return new Reservation(customer, screening, fee, audienceCount);
    }
}

// 내부 구현을 인터페이스에 노출시키는 Movie
// 할인 정책의 종류 -> 변경된다면 ? 모든 클라이언트가 영향을 받음
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;

    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;

    public MovieType getMovieType() {
        return movieType;
    }

    // 요금을 계산하는 오퍼레이션, 정책 별로 3가지
    public Money calculateAmountDiscountedFee() {
        if (movieType != MovieType.AMOUNT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(discountAmount);
    }

    public Money calculatePercentDiscountedFee() {
        if (movieType != MovieType.PERCENT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(fee.times(discountPercent));
    }

    public Money calculateNoneDiscountedFee() {
        if (movieType != MovieType.NONE_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee;
    }
```

- 캡슐화란 변하는 어떤 것이든 감추는 것

## 데이터 중심 설계의 문제점

- 너무 이른 시기에 데이터에 관해 결정하도록 강요
- 협력 고려  x