# Lambda&Stream

## Lambda

- 메서드를 하나의 식으로 표현한 것(익명메소드(함수) 생성 문법 )
- 자바는 메서드 자체로 선언 불가 → 람다식은 인터페이스를 가진 익명구현객체

인터페이스 + 익명클래스 

```java
public interface Calc {
    int calc(int n);
}

public class Driver {
    public static void main(String[] args) {
        Calc ret = new Calc() {
            @Override
            public int calc(int n) {
                return n + 1;
            }
        };

        System.out.println(ret.calc(2)); // 3
    }
}
```

람다 사용 

```java
public class Driver {
    public static void main(String[] args) {

//      Calc ret = (int n) -> {return n + 1;};
        Calc ret = n -> n+1;
        System.out.println(ret.calc(2)); // 3
    }
}
```

과정 

1. 매개변수의 타입 생략 가능 (인터페이스에서 참조하므로)
2. 매개변수가 하나일 경우 () 생략가능 ( 두 개 이상일 경우 생략 불가, 없을 경우도 생략불가)
3. 로직이 한줄 일 경우 {}과 return 생략 가능 

### 타겟타입

- 익명구현객체(여기서는 람다식)의 기반이 되는 인터페이스를 말함
- @Funtionallnterface로 명시적으로 선언 가능
- 메서드를 하나만 가지고 있어야 함
- 람다식의 매개변수는 final 취급함 ( 불변상수 취급)

```java
Calculator cal = n -> ++n; // 매개변수를 변경시키려 하면 에러발생
```

## Stream

- 배열 또는 컬렉션을 다루는 방법
- 람다를 활용 할 수 있는 기술 중 하나
- 장점 :
    - 간결한 코드
    - 병렬처리

  

생성하기 → 가공하기 → 결과 만들기 과정을 거침 

### 생성하기

배열 스트림 

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> streamOfArrayPart = 
  Arrays.stream(arr, 1, 3); // 1~2 요소 [b, c]
```

컬렉션 스트림 

```java
public interface Collection<E> extends Iterable<E> {
  default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
  } 
  // ...
}
// 인터페이스에 추가된 디폴트 메서드 .stream을 통해서 스트림 생성 가능  

List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림
```

### 가공하기

- 밑에 나올 api를 이용해서 원하는 값만 뽑아낼 수 있음
- 이런 작업은 스트림을 반환하기 때문에 이어 붙여서 (chaining) 작성 가능

filtering  

- 요소들을 하나씩 평가해서 걸러냄

```java
Stream<T> filter(Predicate<? super T> predicate);
// predicate는 boolean을 리턴하는 함수형 인터페이스로 평가식이 들어가야함 

Stream<String> stream = 
  names.stream()
  .filter(name -> name.contains("a"));
// 사용
```

Mapping 

- 요소들을 하나씩 특정 값으로 변환해줌

```java
Stream<String> stream = 
  names.stream()
  .map(String::toUpperCase);

//또는 
Stream<Integer> stream = 
  productList.stream()
  .map(Product::getAmount);
```

Sorting

```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator); 
//comparator  사용

IntStream.of(14, 11, 20, 39, 23)
  .sorted()
  .boxed()
  .collect(Collectors.toList());
```

### 결과 만들기

Calculating 

- 최소, 최대, 합, 평균 등 기본형 타입 반환

```java
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum = LongStream.of(1, 3, 5, 7, 9).sum();
```

Reduction

- 여러 요소를 섞을 수 있음
- reduce 메소드는 세가지 파라미터를 가짐
    - accumulater : 각 요소 처리하는 로직
    - identity : 계산의 초기값
    - combiner : 병렬 스트림에서 나눠 결과를 하나로 합치는 로직 (?)

```java
// 1개 (accumulator)
Optional<T> reduce(BinaryOperator<T> accumulator);

// 2개 (identity)
T reduce(T identity, BinaryOperator<T> accumulator);

// 3개 (combiner)
<U> U reduce(U identity,
  BiFunction<U, ? super T, U> accumulator,
  BinaryOperator<U> combiner);
```

```java
OptionalInt reduced = 
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce((a, b) -> {
    return Integer.sum(a, b);
  });
// 인자 하나만 받는 경우 ( 1+2+3 을 리턴함 )
```

Collecting 

- Collector 타입 인자를 받아서 처리함

```java

List<Product> productList = 
  Arrays.asList(new Product(23, "potatoes"),
                new Product(14, "orange"),
                new Product(13, "lemon"),
                new Product(23, "bread"),
                new Product(13, "sugar"));

List<String> collectorCollection =
  productList.stream()
    .map(Product::getName)
    .collect(Collectors.toList());
// [potatoes, orange, lemon, bread, sugar]

//또는 
String listToString = 
 productList.stream()
  .map(Product::getName)
  .collect(Collectors.joining());
// potatoesorangelemonbreadsugar
```

### 참고 : stream의 **groupingBy() 메서드**

- groupingBy는 영어 그대로에서도 알 수 있듯이, 특정 속성(property)값에 의해서 그룹핑을 짓는 것이다.
- 결과값으로 항상 Map<K, V> 형태를 리턴하게 된다. ( SQL문에서도 사용하는 `group by`와 유사)
- groupingBy 메서드는 최대 3가지 파라미터를 받는 메서드들로 구성

1. **classifier (Function<? super T,? extends K> ): 분류 기준을 나타낸다.**
2. **mapFactory (Supplier) : 결과 Map 구성 방식을 변경할 수 있다.**
3. **downStream (Collector<? super T,A,D>): 집계 방식을 변경할 수 있다.**

---  
참고자료:   
[https://sehun-kim.github.io/sehun/java-lambda-stream/](https://sehun-kim.github.io/sehun/java-lambda-stream/)  
[https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)   
[https://umanking.github.io/2021/07/31/java-stream-grouping-by-example/](https://umanking.github.io/2021/07/31/java-stream-grouping-by-example/)