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

의존관계 주입 방법 

1. 생성자 주입
2. 메서드 주입(setter() 등)
3. 필드 주입

장점 

1. 의존성이 줄어듬

주입받는 대상이 변하더라도 구현을 수정하지 않아도 됨

1. 재사용성이 높은 코드가 됨
2. 테스트하기 좋은 코드가 됨
3. 가독성

참고 자료 : 

[의존관계 주입(Dependency Injection) 쉽게 이해하기](https://tecoble.techcourse.co.kr/post/2021-04-27-dependency-injection/)
