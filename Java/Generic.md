# Generic


데이터 타입을 일반화한 것 

클래스나 메서드에 사용할 내부 데이터 타입을 사용자가 지정하고 컴파일 시  변환

## 제네릭의 장점

- 미리 변환 시에 객체의 타입 안정성을 높임 (잘못된것이 들어올 수 있음)
- 반환값에 대한 타입 변환 및 검사에 들어가는 비용 감소
- 제네릭을 사용하지 않는 코드와 호환성 → 컴파일 된 class 파일에는 제네릭 사용 x
- 코드의 재사용성 높임

```java
class MyArray<T> {
    T element;
    void setElement(T element) { this.element = element; }
    T getElement() { return element; }
}
```

**제네릭 자리에 기본타입 사용 x , 참조 타입만 올 수 있음**

→ **wrapper 클래스를 사용해야함!**

![generic type](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/generic_type.png)

## 제한된 제네릭

```java
<K extends T>	// T와 T의 자손 타입만 가능 (K는 들어오는 타입으로 지정 됨)
<K super T>	// T와 T의 부모(조상) 타입만 가능 (K는 들어오는 타입으로 지정 됨)
 
<? extends T>	// T와 T의 자손 타입만 가능
<? super T>	// T와 T의 부모(조상) 타입만 가능
<?>		// 모든 타입 가능. <? extends Object>랑 같은 의미
```

? 는 와일드 카드이며 ‘알 수 없는 타입’이다. 

예시

```java
public class ClassName <E extends Comparable<? super E> { ... }
```

- 특히 PriorityQueue(우선순위 큐), TreeSet, TreeMap 같이 값을 정렬하는 클래스 만약 여러분이 특정 제네릭에 대한 자기 참조 비교를 하고싶을 경우 대부분 공통적으로 위와 같은 형식을 취한다.

- E 는 Comparable을 implements 받아야 하고, Comparable은 매개 변수로 E와 E의 상위 클래스만 받아야 한다.
    → Comparable.compareTo(T o) 가 업캐스팅으로 구현되어있을 수도 있어서... 

<br>

참고자료 :

[https://st-lab.tistory.com/153](https://st-lab.tistory.com/153)
