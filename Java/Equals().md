# Equals()


## String 변수 생성시 주소 할당

![https://raw.githubusercontent.com/dyparkkk/TIL/main/img/string_heap.png](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/string_heap.png)

- String을 리터럴로 생성 할 경우 intern() 메서드 호출
- intern() 메서드는 string constant pool에서 문자열 검색 후 반환(없으면 생성 후 반환)

## 주소값비교(==)와 값비교(equals)  | String 기준

- 기본타입의 int, char 등 call by value로 주소값을 가지지 않음
- string은 클래스로서 생성시 주소값이 부여됨 → 같은 값이여도 주소값이 달라질 수 있음
- 

## Object.equals()

- 두 객체가 동일한지 검사한다.
- 물리적 동치성 뿐만 아니라 ‘논리적 동치성’을 비교하도록 재정의 할 때 사용할 수 있다.
    - String 클래스는 Object.equals()를 오버라이드해서 값비교를 위해 사용
    

```java
// 실제 String 클래스에 오버라이딩된 equals() 메서드
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                                  : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }

// Object class에 정의된 equals() 메서드
public boolean equals(Object obj) {
        return (this == obj);
    }
```

## hashCode() 란?

실행중에(runtime) 객체의 유일한 Integer 값을 반환한다. 

Object 클래스에서는 heap에 저장된 객체의 메모리 주소를 반환한다. ( 항상 그런 것은 아님)

- hash table에 사용된다 ex) java.util.HashMap 등
- equals() 메서드를 재정의 하면 꼭 hashCode() 메서드를 재정의 해야함
    - equals() 로 true를 반환하는 같은 두 객체는 hashcode도 같은 값을 반환해야 함

<br>
참고자료

[https://coding-factory.tistory.com/536](https://coding-factory.tistory.com/536)

[https://mangkyu.tistory.com/101](https://mangkyu.tistory.com/101)  |  hashcode와 equals
