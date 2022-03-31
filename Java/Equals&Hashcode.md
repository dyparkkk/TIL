# Equals&HashCode

## String 변수 생성시 주소 할당

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/equals&hashcode00.png)

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

- 같은 객체는 항상 같은 해시코드를 반환하고, 다른 객체도 같은 해시 코드를 반환 할 수 는 있다.

- hash table에 사용된다 ex) java.util.HashMap 등
- equals() 메서드를 재정의 하면 꼭 hashCode() 메서드를 재정의 해야함
    - equals() 로 true를 반환하는 같은 두 객체는 hashcode도 같은 값을 반환해야 함

## String의 hashCode()

```java
public static int hashCode(byte[] value) {
        int h = 0;
        int length = value.length >> 1;
        for (int i = 0; i < length; i++) {
            h = 31 * h + getChar(value, i);
        }
        return h;
    }
```

- String의 모든 글자를 반영하기 위한 해시 함수 → 재귀적으로 작성
- 31은 소수여서 균등한 분포에 좋다고 함
- 31 = 32 -1 인데 32는  1<<5 여서 계산이 빠름

---

참고자료:  

[https://coding-factory.tistory.com/536](https://coding-factory.tistory.com/536)  
[https://mangkyu.tistory.com/101](https://mangkyu.tistory.com/101)  |  hashcode와 equals  
[https://d2.naver.com/helloworld/831311](https://d2.naver.com/helloworld/831311)