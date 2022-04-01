# 이펙티브 자바


목차  
- [아이템2 생성자에 매개변수가 많다면 빌더를 고려하라](#아이템2-생성자에-매개변수가-많다면-빌더를-고려하라)
- [아이템3 private 생성자나 열거 타입으로 싱글턴임을 보증하라](#아이템3-private-생성자나-열거-타입으로-싱글턴임을-보증하라)
- [아이템17 변경 가능성을 최소화하라](#아이템17-변경-가능성을-최소화하라)
- [아이템20 추상 클래스보다는 인터페이스를 우선하라](#아이템20-추상-클래스보다는-인터페이스를-우선하라)
- [아이템24 멤버 클래스는 되도록 static으로 만들라](#아이템24-멤버-클래스는-되도록-static으로-만들라)
- [아이템28 배열보다는 리스트를 사용하라](#아이템28-배열보다는-리스트를-사용하라)
- [아이템31 한정적 와일드카드를 사용해 API 유연성을 높이라](#아이템31-한정적-와일드카드를-사용해-API-유연성을-높이라)
- 


## 아이템2 생성자에 매개변수가 많다면 빌더를 고려하라

- 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움

코드가 흥미로움

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;

		// static class 로 만듬
    public static class Builder {
        // 필수 파라미터
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }
					// 자신을 반환하는 메서드 
        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100)
                .sodium(35)
    }
}
```

## 아이템3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 어플리케이션이 시작할 때 한 번만 메모리 할당(static)하고 **하나의 인스턴스만 사용을 보장**
- **글로벌하게 접근** 할 수 있어야 함

### 단점

- 미래에도 하나의 인스턴스만 사용한다고 보장 할 수 없다( 서비스의 규모가 커지면 하나의 인스턴스로 감당 안 될 수 있음)
- test가 어렵다 → 테스트에 기존에 사용하던 인스턴스를 사용해야 하는데 곤란함
- 싱글톤 객체를 사용하는 곳에서 사이드 이펙트 발생 확률이 생기게 되며, 멀티 쓰레드 환경에서 동기화 처리 문제가 있을 수 있다.

### 구현

- 생성자를 private로 구현 → new 사용 x
- class method 사용해야함 → instance method 사용 x
- static 변수로 인스턴스 만들기

```java
public class Singleton {
  public static Singleton instance;

  private Singleton(){}
  public static Singleton getInstance(){
    if(instance == null){
      instance = new Singleton();
    }
    return instance;
  }
}
```

```java
// 정적 팩토리 방식
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton(){}
    public static Singleton getInstance() {
        return INSTANCE;
    }
    /*
    직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로 부족
    모은 인스턴스 필드를 일시적(transient)라고 선언하고
     readResolve 메서드를 제공해야함(가짜 singleton 객체가 생성됨 방지)
     */
}
```

### **열거타입 싱글턴**

- **간결**하고 **직렬화도** 되며 **리플렉션 공격에서도 인스턴스가 생기는 것을 막아줌**
- 원소가 하나뿐인 열거타입
- leaveTheBuilding → 이건 뭘까??

```java
public enum Singleton {
    INSTANCE;

    public void leaveTheBuilding(){}
}
```

## 아이템17 변경 가능성을 최소화하라

- 불변 인스턴스는 정보가 고정되어 객체가 죽을 때까지 변하지 않음

### 만드는 규칙

- 객체 상태를 변경하는 메서드 제공 x
- 클래스 확장 못하게 (final, private생성자 + 정적 팩터리 메서드)
- 모든 필드 private + final( 과하긴 한데 설계자의 의도를 명확히 들어냄)
- 자신 외에는 내부 가변 컴포넌트에 접근 못하게 함 ( getter  사용 조심)

  

예시

```java

// Immutable complex number class 
public final class Complex {
    private final double re;
    private final double im;

	// 자주 사용하는 상수를 제공할 수 잇음 
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // Static factory, used in conjunction with private constructor (Page 85)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // float와 double은 == 비교 사용 X
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

### 장점

- 스레드 안전( 자유롭게 공유 가능)
- 다루기 쉬움 (복잡해도 불변식 유지하기 쉬움)
- 원자성제공

## 아이템20 추상 클래스보다는 인터페이스를 우선하라

- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
- 인터페이스를 구현한 클래스는 ‘주된 타입’ 외에 특정 선택적 행위를 제공한다고 선언하는 효과를 줌
- 인터페이스는 계층구조가 없는 타입 프레임워크를 만들 수 있음

### 추상 골격 구현

인터페이스와 추상 클래스의 장점을 모두 취하는 방법

AbstractList 등 ...

```java
// 골격 구현을 사용해 완성한 구체 클래스
static List<Integer> intArraysList(int[] a){
        Objects.requireNonNull(a);

				// 다이아몬드 연산자( 자바 9부터 )
        return new AbstractList<>() {
            @Override
            public Integer get(int index) {
                return a[index];
            }

            @Override
            public Integer set(int index, Integer element) {
                int oldVal = a[index];
                a[index] = element; // 오토언박싱
                return oldVal; // 오토박싱
            }

            @Override
            public int size() {
                return a.length;
            }
        };
    } // 템플릿 메서드 패턴
```

## 아이템 24 : 멤버 클래스는 되도록 static으로 만들라

- 멤버 클래스에서 바깥 인스턴스에 접근 할 일이 없다면 static을 붙여서 정적 멤버 클래스로 만들어라
- 비정적 멤버 클래스는 바깥 클래스에 대한 참조를 가지고 있어서 메모리 누수 가능성이 있음

### 정적 멤버 클래스

- 다른 클래스 안에 선언됨 + 바깥 클래스의 private에 접근 가능
- 위 두개 빼고 클래스와 같음
- 보통 바깥 클래스와 같이 쓰일 때 유용한 경우에 사용

### 비정적 멤버 클래스

- 바깥 클래스의 인스턴스를 생성하고 그 후에 생성가능
- 관계가 바깥 클래스의 인스턴스가 생성될 때 확립되어 변하지 않음

사용 → 보통 어댑터를 정의할 때 많이 사용

```java
// 반복자(Iterator) 구현에 주로 사용
public class MySet<E> extends AbstractSet<E> {
    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ... 
    }
}
```

```java
// HashMap 의 keySet
// Map의 Key를 Set으로 반환하기 위해 사용
public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

// 비정적 멤버 클래스 안에서 Map을 this로 호출해서 사용할 일이 있어서 static 못씀
final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { **HashMap.this**.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(**HashMap.this**, 0, -1, 0, 0);
        }
...
}
```

## 아이템 28 : 배열보다는 리스트를 사용하라

### 배열은 공변이다

- Sub가 Super의 하위타입일 때 배열 Sub[]는 배열 Super[]의 하위 타입이 됨
- 제네릭은 불공변

배열은 런타임에 실패한다. 

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 다르다" // ArrayStoreException을 던짐
```

제네릭은 컴파일 되지 않음

```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않음 -> 컴파일 x
```

### 배열은 실체화(reify)된다.

- 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- 제네릭은 타입 정보가 런타임에 소거됨

⇒ 배열은 제네릭 타입, 타입 매개변수로 사용 할 수 없음 ( 컴파일에서 막음)

```java
// 컬렉션 안의 무작위 원소를 골라주는 메서드를 제공하는 클래스
public class Choose<T> {
    private final T[] choiceArray;

    public Choose(Collection<T> choices) {
        choiceArray = choices.toArray();
    }
		...
}
// 컴파일러 에러 
// Incompatible types. Found: 'java.lang.Object[]', required: 'T[]'
```

배열 → 리스트로 리펙토링

```java
// 컬렉션 안의 무작위 원소를 골라주는 메서드를 제공하는 클래스
public class Choose<T> {
    private final List<T> choiceList;

    public Choose(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }
    
    public T ChooseOne(){
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

## 아이템 31 : 한정적 와일드카드를 사용해 API 유연성을 높이라

> List<String>은 List<Object>의 하위 타입이 아님.
List<Object>의 Object를 넣는 일을 List<String>는 못함 → 리스코프 치환 원칙 위배
> 

- 생산자 : 매개변수에서 본인 컬렉션으로 원소를 옮겨 담음 < ? extends E > 사용
- 소비자 : 매개변수로 컬렉션의 원소를 옮겨 담음 <? super E> 사용

  

예시

```java
public class RecursiveTypeBound {
    public static <E extends Comparable<? super E>> E max(
        List<? extends E> list) {
        /**
         * List<? extends E> : 원소를 부모인 E로 치환해서 사용가능해서 
         *     (리스코프 치환원칙) 유연성을 위해 
         * Comparable<? super E> : 부모에서 Comparable을 구현하고 
         *      그것을 확장한 E를 지원하기 위해 (어렵다...)
         */
        if (list.isEmpty())
            throw new IllegalArgumentException("Empty list");

        E result = null;
        for (E e : list)
            if (result == null || e.compareTo(result) > 0)
                result = e;

        return result;
    }

    public static void main(String[] args) {
        List<String> argList = Arrays.asList(args);
        System.out.println(max(argList));
    }
}
```