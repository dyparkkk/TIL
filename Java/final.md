> java에서 한번만 할당할 수 있는 entity를 정의할 때 사용. final로 선언된 변수가 할당되면 항상 같은 값을 가진다.
-위키피디아
> 

## final class

```java
public final class MyFinalClass {...}

public class ThisIsWrong extends MyFinalClass {...} // 상속불가 
```

예시) String class

immutable object로 얻을 수 있는 이점

- 하나의 instance를 여러 곳에서 참조할 수 있다
    - 만약 final이 아니면 비슷해보이는 string이 다른 instance 일 수도 있음
- read-Only가 됨으로 바뀔 걱정이 없음
- thread-safe함

## final method

```java
public class Base {
    public       void m1() {...}
    public final void m2() {...}
}

public class Derived extends Base {
    public void m1() {...}  // 상속
    public void m2() {...}  // 오버라이딩 불가
}
```

자식클래스에서 메소드를 재정의(오버라이딩)할 때 오류발생.

→ 명시적으로 override method를 막을 수 있음

## final variable

final을 선언해서 read-only로 만들수 있음

## 결론

- 개발자의 의도를 명시
- 가독성 확보

---
  
참고자료 :  
[https://makemethink.tistory.com/184](https://makemethink.tistory.com/184)  
[https://blog.lulab.net/programming-java/java-final-when-should-i-use-it/#fn:2](https://blog.lulab.net/programming-java/java-final-when-should-i-use-it/#fn:2)  
[https://stackoverflow.com/questions/2068804/why-is-the-string-class-declared-final-in-java](https://stackoverflow.com/questions/2068804/why-is-the-string-class-declared-final-in-java)  
