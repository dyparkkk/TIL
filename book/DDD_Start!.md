# DDD Start!

## 2. 아키텍처 개요

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/ddd00.png)

### 네 개의 영역

- 아키텍처 설계 - 표현, 응용, 도메인, 인프라스트럭처
- 표현영역 : 사용자 요청을 받아 응용 영역 전달 + 그 반대
    - http 요청 파라미터를 응용 서비스가 요구하는 객체타입으로 변환, 응용 서비스가 리턴한 결과를 JSON 형식으로 변환해서 응답
- 응용 영역 : 시스템이 사용자에게 제공 할 기능 구현
    - 로직은 도메인 모델에 위임

```java
@Service
public class CancelOrderService {

    @Transactional
    public void cancel(OrderNo orderNo, Canceller canceller) {
        Order order = findOrder(orderNo);
        if (!cancelPolicy.hasCancellationPermission(order, canceller)) {
            throw new NoCancellablePermission();
        }
        order.cancel(); // 도메인 모델을 사용해서 기능 구현
    }
}
```

- 인프라스트럭처 : 구현 기술에 대한 것 ex) RDBMS 연동, 메시징 큐에 메시지 전송 등

### 계층 구조 아키텍처

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/ddd01.png)

- 상위 계층에서 하위 계층으로만 의존함
- 구현의 편리함을 위해 응용 계층에서 인프라스트럭처를 사용하는 경우가 있음
    - 인프라스트럭처는 상세한 구현을 다루기때문에 문제가 발생 ( 종속됨, 책 42-43 참고)
    1. 테스트 어려움
    2. 기능 확장의 어려움 ( 구현을 변경하기 어려움)

### DIP

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/ddd02.png)

- 위 종속 문제의 해결책
- 고수준 모듈이 동작하려면 저수준 모듈을 사용해야함 → 앞서 계층 구조의 문제 발생
- **반대로 저수준 모듈이 고수준 모듈을 의존하게 만듬 ( 추상화 인터페이스 사용)**

```java
public interface RuleDiscounter{
	public Money applyRules(Customer customer, List<OrderLine> orderLines);
}
// 인터페이스로 들어오는 값을 고정해버림 ...! 
```

- 응용 계층에서는 이제 인터페이스 사용 (+의존 주입을 사용) → 인프라스트럭처로 부터 의존 탈출
- 인프라스트럭처는 인터페이스를 상속받고 구현 → 인터페이스에 의존

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/ddd03.png)

- 의존관계가 역전됨
- 주의 ! 인터페이스는 응용 계층에 의존해서 만들어야 함
- 인터페이스에 맞는 대용 객체(Mock 등)을 사용해서 저수준 모듈 구현없이 테스트 진행 가능

### DIP와 아키텍처

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/book/img/ddd04.png)

- DIP를 사용하면 아키텍처 계층 구조가 바뀜
- 인프라스트럭처가 응용과 도메인에 의존함

### 도메인 영역의 주요 구성요소

- 엔티티, 벨류, 애그리거트, 리포지터리, 도메인 서비스로 구성
- 도메인 모델의 엔티티는 DB테이블의 엔티티와 다르게 데이터 + 도메인 기능을 제공
    - 두 개 이상의 데이터를 묶어서 벨류 타입으로 표현 가능
- 밸류는 불변으로 구현을 권장
- 애그리거트는 엔티티와 밸류의 군집 → 관리 용이
    - 루트 엔티티를 통해 간접적으로 다른 엔티티에 접근 → 캡슐화 개념 이용
- 리포지터리는 응용 서비스와 밀접한 연관 ( 트랜잭션 관리, 도메인 객체 가져오기 등)

작성중 ...

---
그림참고 [https://minkukjo.github.io/dev/2020/11/02/DDD-02/](https://incheol-jung.gitbook.io/docs/study/ddd-start/1)  