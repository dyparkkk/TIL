# HashMap(Java11)


- **Java Collections Framework에 속한 구현체 클래스**
- HashMap은 어떻게 해시 충돌 가능성을 최소화 시키는가 → 보조 해시 함수 사용
- 키에 대한 해시 값을 사용하여 값을 저장하고 조회하며, 키-값 쌍 개수에 따라서 동적으로 크기가 증가하는 associate array(map, dictionary 등으로 불림)

## 해시 분포와 해시 충돌

동일하지 않은 객체 X와 Y (X.equals(Y) == false) 일 경우 X와 Y의 hashCode가 같지 않다면 ( 각 객체는 고유한 해시 코드를 가진다면)   

→  해시 함수를 완전 해시 함수라고 함.

- String이나 POJO는 완전 해시 함수 제작이 사실상 불가능
- 빠르게 동작하는 해시 함수가 있더라도 hashCode() 메서드는 int를 리턴 → 2^32가 한계
- 또한 HashMap에서 O(1)에 접근하려면 2^32를 가진 배열 필요.. → 메모리 낭비

HashMap에서는 M개로 축소해서 사용

```java
int index = X.hashCode() % M;
```

해시 충돌이 난 경우 저장 방식 2가지

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/hashmap00.png)

### Open Addressing

- 데이터를 삽입하려는 버킷이 이미 사용 중인 경우 다른 해시 버킷에 삽입함.
- 연속된 공간에 저장되어 있기 때문에 Seperate Chaining 방식에 비해 캐시 효율이 높음
    - 데이터 크기가 커지면 장점이 사라짐
    - 참고 : [https://stackoverflow.com/questions/49709873/cache-performance-in-hash-tables-with-chaining-vs-open-addressing](https://stackoverflow.com/questions/49709873/cache-performance-in-hash-tables-with-chaining-vs-open-addressing)
- Linear Probing(선형 탐사), Quadratic Probing(제곱 탐사) 등의 방식 사용

**Linear Probing**  

: 충돌이 발생 하면 다음 원소를 검사하는 방식

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/hashmap01.png)

**Quadratic Probing(제곱 탐사)**

: 이동하는 폭이 고정되지 않고 제곱으로 옮겨지는 방식   

!! 이중해싱 등의 방식도 있음  

### 문제점

- 군집화 문제가 발생할 수 있다.
- 삭제 시 효율성이 안좋음 (hashMap에서 remove()호출이 잦음)

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/hashmap02.png)

### Seperate Chaining

- HashMap이 사용하는 방식 → remove() 메서드가 더 효율적이라서 ???
- 저장된 쌍이 많아지면 일반적으로 오픈 어드레싱 방식보다 성능이 좋아짐
- 해시 충돌을 줄일 수 있다면 Worst Case를 줄일 수 있음
    - 오픈 어드레싱은 해시 충돌 몇번으로 계속 밀려서 군집화가 발생 할 수 있음

## Java 8 HashMap이후 Seperate Chaining

- 데이터 개수가 많아지면 세퍼레이트 채이닝에서 링크드 리스트 대신 트리 구조를 사용
    - 시간 복잡도를  M → logM으로 낮춤

Java 11 HashMap의 put

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) // 해시 버킷이 null이면 할당
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 해시 버킷이 비어있으면 newNode 생성
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash && // 첫 노드가 찾는 노드면 e
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode) // 노드가 TreeNode 면 PutTreeVal()
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { //첫 노드가 다르면 Chaining 에서 찾음
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
				// 삽입, 삭제 등으로 이 HashMap 객체가 몇 번이나 변경(modification)되었는지
        // 관리하기 위한 코드라고 함
        ++modCount; 
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

TreeNode class 

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        HashMap.TreeNode<K, V> parent;  // red-black tree links
        HashMap.TreeNode<K, V> left;
        HashMap.TreeNode<K, V> right;
        HashMap.TreeNode<K, V> prev;    // needed to unlink next upon deletion
        boolean red; // 레드 블랙 트리임 !!

        TreeNode(int hash, K key, V val, HashMap.Node<K, V> next) {
            super(hash, key, val, next);
        }
				...

        // TreeNode에서 어떤 두 키의comparator 값이 같다면 서로 동등하게 취급된다.
        // 그런데 어떤 두 개의 키의 hash 값이 서로 같아도 이 둘은 서로 동등하지
        // 않을 수 있다. 따라서 어떤 두 개의 키에 대한 해시 함수 값이 같을 경우,
        // 임의로 대소 관계를 지정할 필요가 있는 경우가 있다.
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                    (d = a.getClass().getName().
                            compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                        -1 : 1);
            return d;
        }
			...
    }
```

## 해시 버킷 동적 확장

- HashMap은 키-값 쌍의 데이터 개수가 일정 수준 이상이 되면 동적으로 해시 버킷의 개수를 2배로 늘림
- 기본값은 16이며 최대는 2^30
- 버킷 개수가 증가할 때마다 모든 키-값을 읽어 새로 Seperate Chaining 해줘야함

Java 11 

```java
final Node<K,V>[] resize() {
...
	do {
		next = e.next;
		if ((e.hash & oldCap) == 0) {
		    if (loTail == null)
		        loHead = e;
		    else
		        loTail.next = e;
		    loTail = e;
		}
		else {
		    if (hiTail == null)
		        hiHead = e;
		    else
		        hiTail.next = e;
		    hiTail = e;
		}
	} while ((e = next) != null);
...
```

Java 11 에서도 여전히 do-while문을 사용해서 모든 노드에 접근해서 채이닝을 재설정해준다. 

- 버킷 개수를 두배로 늘리는 것에 문제가 있음 →
- 버킷 개수 M이 2^a의 형태가 되므로 index = X.hashCode() % M 에서 하위 a개 비트만 사용하게 됨
- 보조 해시 함수로 해결

## 보조 해시 함수

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

- 하위 16비트를 상위로 올리고 XOR 연산
- 1.트리 구조를 사용함으로서 해시 충돌에 더 효율적으로 대응하게됨
- 2.최근 해시 함수는 균등 분포가 잘 되서 효과가 크지 않음 → 간단한 설계

## 정리

- Java의 HashMap은 환경에 맞게 계속 변화하고 있음
- 해시 충돌을 방지하기 위해 Seperate Chaining과 보조 해시 함수 사용
- Seperate Chaining에서 링크드 리스트 뿐 아니라 트리도 함께 사용

---

참고자료:  
[https://d2.naver.com/helloworld/831311](https://d2.naver.com/helloworld/831311)  