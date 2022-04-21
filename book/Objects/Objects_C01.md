# Objects_C01

작성 코드 확인 :  

[https://github.com/dyparkkk/codeSample/tree/09c54e49d25235861931336d41bff6a451ae9091](https://github.com/dyparkkk/codeSample/tree/09c54e49d25235861931336d41bff6a451ae9091)

```java
/**
 * 관람객을 맞이함
 */
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        Ticket ticket = ticketSeller.getTicketOffice().getTicket();
        if (audience.getBag().hasInvitation()) {
            audience.getBag().setTicket(ticket);
        } else{
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

### 무엇이 문제인가

- 소프트 웨어의 목적
    - 실행 중에 제대로 동작
    - 변경이 쉬워야 함
    - 읽기 쉬워야 함
- ticketSeller, ticketOffice, audience 모두 수동적인 존재 → 읽기 어려워 짐
    - 객체의 구조를 이해해야 함
    - 사용자가 데이터를 관리해야 함
- 변경에 취약하다
    - Theater가 너무 많은 객체에 의존성을 가짐 → 결합도가 높음

## 개선

```java
/**
 * 관람객을 맞이함
 */
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
       ticketSeller.sellTo(audience);
    }
}

/**
 * 판매원을 초대장을 티켓으로 교환해주거나 티켓을 판매하는 역할을 수행
 */
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}

public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Long buy(Ticket ticket) {
        if(bag.hasInvitation()){
            bag.setTicket(ticket);
            return 0L;
        } else{
            bag.setTicket(ticket);
            bag.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

- 자율성을 높임 → 다른 객체의 데이터에 직접 접근하지 못하게 바꿈
- 캡슐화 : 개념적, 물리적으로 객체 내부의 세부사항을 감추는 것
    - 목적 : 변경에 유연한 객체를 만듬
    - 객체 사이의 결합도를 낮춤
- 수정된 audence와 TicketSeller는 자신이 가지고 있는 소지품을 스스로 관리함 !!
- 자신과 연관있는 작업만 수행하고 연관성 없는 작업은 다른 객체에게 위임 → **응집도가 높음**

### 책임의 이동

- 개선 전 코드는 Theater에 책임이 모두 집중되어 있었음
- 객체지향 설계의 핵심은 적절한 객체에 적절한 책임을 할당하는 것
- 설계를 어렵게 만드는 것은 **의존성**이다 !

### 트레이드 오프

- 어떤 기능을 설계하는 방법은 한 가지 이상일 수 있다
- 결국 설계는 트레이드오프의 산물이다
    - 의존성 추가 vs 자율성 추가 등