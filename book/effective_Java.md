# 이펙티브 자바


목차
- [아이템 20 : 추상 클래스보다는 인터페이스를 우선하라](#아이템-20-:-추상-클래스보다는-인터페이스를-우선하라)
- [아이템 24 : 멤버 클래스는 되도록 static으로 만들라](#아이템-24-:-멤버-클래스는-되도록-static으로-만들라)

## 아이템 20 : 추상 클래스보다는 인터페이스를 우선하라

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
        return new AbstractList<>() -
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