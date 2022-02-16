# SRP (single responsibility principle)

하나의 객체는 하나의 책임(기능)만 가져야 한다. 

```java
public class Student {
    	public void getCourse(){	}
    	public void addCourse() {	}
    	public void save(){	}
    	public Student load() {	}
    	public void printOnReportCard() {	}
    	public void printOnAttendanceBook() {	}
    }
```

위 코드를 보면 student는 강의를 저장하고 db에서 가져오고 출력하는 책임을 담당한다. 

만약 load()에 변경이 있을 때 add나 print 모두 테스트 필요 → 유지 보수의 어려움

# OCP (open-closed principle)

- 기존 코드는 수정하지 않으면서 기능을 추가 할 수 있어야함
    
    (변경을 최소화, 확장은 유연하게)
    

```java
public class Computer {
    	private KakaoMessenger kakaoMessenger;
    	
    	public static void main(String[] args) {
    		Computer computer = new Computer();
    		computer.boot();
    	}
    
    	private void boot() {
    		System.out.println("BOOTING.....");
    		kakaoMessenger = new KakaoMessenger();
    		kakaoMessenger.boot();
    	}
    }
```

```java
public class KakaoMessenger {
    	public void boot() {
    		System.out.println("Kakao BOOTING....");
    	}
 }
```

컴퓨터를 실행하면 카카오톡이 실행되는 코드 

→ 카카오톡에서 라인으로 변경되면 컴퓨터를 수정해야함

```java
public class Computer {
    	private Messenger messenger;
    	
    	public static void main(String[] args) {
    		Computer computer = new Computer();
    		computer.setMessenger(new LineMessenger());
    		computer.boot();
    	}
    
    	private void setMessenger(Messenger messenger) {
    		this.messenger = messenger;
    	}
    
    	private void boot() {
    		System.out.println("BOOTING.....");
    		messenger.boot();
    	}
    }
```

```java
public class LineMessenger implements Messenger{
    
    	@Override
    	public void boot() {
    		System.out.println("Line BOOTING....");
    	}
    }
```

```java
public interface Messenger {
    	void boot();
    }
```

interface로 메신저를 분리하고 클래스에서 오버라이딩을 통해 수정

- 기존 코드 변경을 최소화하고 단순히 클래스 추가 후 오버라이딩으로 변경이 가능
- 내부 동작 원리를 알지 못해도 됨 ! → 유지 보수에 유연

# LSP (liskov substitution principle)

- 자식 클래스는 부모 클래스에서 가능한 행위를 수행 할 수 있어야 함
    
    → 부모 클래스의 인스턴스 대신 자식 클래스의 인스턴스를 사용해도 프로그램 유지되어야 함
    

```java
public class Bag {
    	private double price;
    
    	public double getPrice() {
    		return price;
    	}
    
    	public void setPrice(double price) {
    		this.price = price;
    	}
    }
```

```java
public class DiscountedBag extends Bag{
    	private double discountRate;
    
    	public void setDiscountRate(double discountRate) {
    		this.discountRate = discountRate;
    	}
    
    	public void applyDiscount(int price) {
    		super.setPrice(price- (int)(discountRate * price));
    	}
    }
```

discountedBag의 applyDiscount 메소드를 사용하면 부모클래스는 자식클래스로 대체되지 않는다. 

- 즉 서브 클래스가 슈퍼 클래스의 책임을 무시하거나 재정의 하지 않고 확장만 수행
- 다시 말해 부모가 수행하고 있는 책임을 그대로 수행하면서 추가적인 필드나 기능을 제공하려는 경우에만 상속을 하는 것
    - 부모 클래스의 책임을 변화시키는 기능은 LSP 법칙에 위배됨
    

# DIP (dependency inversion principle)

의존관계 역전 원칙

의존 관계에서 자주 변하는 것 보다는 변하지 않는 것에 의존하라

- 변하기 어려운 것 : 정책이나 흐름, 개념 → 추상적인 것
- 변하기 쉬운 것 : 구체적인 방식, 사물

어떤 클래스가 의존 관계를 맺을 때 (도움을 받을 때) 추상화된 클래스에 의존 관계를 맺도록 설계

```java
public class Kid {
    	private Toy toy;
    
    	public void setToy(Toy toy) {
    		this.toy = toy;
    	}
    
    	public void play() {
    		System.out.println(toy.toString());
    	}
    }
```

```java
public abstract class Toy{
    	public abstract String toString();
    }
```

```java
public class Lego extends Toy{
    	@Override
    	public String toString() {
    		return "Lego";
    	}
    }
```

```java
public class Main {
    	public static void main(String[] args) {
    		Kid kid = new Kid();
    		kid.setToy(new Lego());
    		kid.play();
    	}
    }
```

아이가 장난감을 가지고 놀 때 장난감의 종류는 언제든 변할 수 있다. 

아이와 장난감이라는 개념은 변하기 어렵다. 

인터페이스로 장난감 관리 → ocp와 dip 모두 만족

# ISP (interface segregation principle)

인터페이스를 클라이언트에 특화되도록 분리시켜라

ex) 복합기에서 나는 프린트만 사용한다면 팩스 등 다른 기능에 영향을 받지 않아야 함

- **다양한 기능을 인터페이스화함으로써 클라이언트에서 인터페이스를 사용할 때 타 인터페이스의 영향을 받지 않고 본인이 구현하고자 하는 기능만을 선택할 수 있게 한다**

참고 자료 : 

[객체지향 SOLID 원칙 이란?](https://velog.io/@kyle/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-SOLID-%EC%9B%90%EC%B9%99-%EC%9D%B4%EB%9E%80)
