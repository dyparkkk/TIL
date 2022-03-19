# JVM

## “write once run anywhere”

→ OS에 종속되지 않고, 어떤 기계든 jvm만 설치하면 동일한 자바 바이트 코드를 돌릴 수 있다. 

![jvm 구조](https://raw.githubusercontent.com/dyparkkk/TIL/6809b01902850148042beab73a09cf317eb0e407/img/jvm%EA%B5%AC%EC%A1%B0.png)

jvm 구조

## Class Loader

jvm내로 클래스파일(.class)를 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈

- **Runtime시에 동적으로 클래스를 로드함.**
- 즉, 클래스를 처음으로 참조할 때, 해당 클래스를 로드하고 링크함
- runtime data area에 바이트 코드 배치

## Execution Engine

class loader가 배치한 바이트 코드 실행 → 실제 기계가 실행 할 수 있는 형태로 변환

### Interpreter

### JIT (just-in-time)

인터프리터 방식의 컴파일러를 보완하기 위해 도입된 JIT 컴파일러.

- 많이 실행되는 메서드는 인터프리팅 방식으로 컴파일시 비효율적 →  한번에 네이티브 코드로  직접실행
- 네이티브 코드는 캐시에 저장 → 여러 번 수행시 빠르게 수행됨


## Runtime Data Area

![Runtime Data Area](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/thread_memory.png)

Runtime Data Area

### pc register

- thread가 생성될 때 마다 생성되며, 스레드마다 하나씩 존재
- 현재 수행중인 jvm명령의 주소를 가짐

### JVM stack 영역

- 프로그램 실행 과정에서 일시적으로 할당되었다가 메서드를 빠져나가면서 소멸되는 데이터를 저장하기 위한 영역
- 메서드 호출시에 각각의 스택영역이 할당되고 메서드 수행이 끝나면 삭제된다.
- ex) 매개변수, 지역변수, 리턴값, 연산시 발생하는 값

### Native method stack

- 자바 바이트 코드가 아닌 실행가능한 기계어로 작성된 프로그램을 실행시키는 영역
- 다른 언어로 작성된 코드를 위한 공간

### Method area (class area, static area)

- 클래스 정보를 처음으로 메모리 공간에 올릴 때 초기화 되는 대상을 저장하기 위한 공간
- Runtime Constant pool
    - 상수 자료형을 저장하고 참조하여 중복을 막는 역할을 함
    

### Heap 영역

![heap 영역](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/heap_area.png)

heap 영역

- 객체를 저장하는 가상 메모리 공간
- class area에 올라온 클래스로만 객체를 만들 수 있음
- Parmanent Generation
    - 클래스 로더에 의해 로드되는 class, method등에 대한 Meta정보가 저장되고 jvm이 사용함
    - java 8 에서 metaSpace로 이름이 바뀜
    - 원래 있던 String Pool이
- ...

## Java Garbage Collecotion

- **동적으로 할당된 영역 중 사용하지 않는 영역을 탐지하여 해제하는 역할 (heap영역)**
- stack 영역에 있는 값을 스캔해서 heap에 있는 객체에 대한 참조를 찾음 ( 이게 전부는 아님)

‘**stop-the-world**’  : GC를 실행하기 위해 GC를 실행하는 스레드를 제외하고 나머지 스레드들이 작업을 멈추는 것 

→ GC 튜닝이란 ‘stop-the-world’의 시간을 줄이는 것

GC의 전제 조건 ( 가정 )

- 대부분의 객체는 금방 접근 불가능 상태 (unreachable)가 된다
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

가정에서 가장 효율적인 메모리를 사용하기 위해 Heap 영역은 두 부분으로 나눈다. young과 old

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/young.png)

## Young 영역

## GC 종류(old 영역)

### Serial GC

- Mark-Sweep-compact 알고리즘 사용
    - old영역의 살아있는 객체를 체크(Mark)
    - 마크된 객체만 남김(Sweep)
    - 한곳에 모아줌(compact)
- 처리하는 스레드가 하나
- 적은 메모리와 하나의 코어가 있을때 사용하는 방식
    
    → 성능이 매우 떨어짐
    

### Parallel GC

- serial gc와 기본적인 알고리즘은 같지만 처리하는 스레드가 여러개라서 빠름

![Untitled](JVM%20c7029/Untitled%203.png)

### Parallel old GC

- Mark-Summary-Compaction

### CMS GC

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/CMS_GC.png)

과정

1. initial mark에서 클래스 로더에 가까운 객체 중 살아있는 것만 찾음 → stop-the-world가 매우 짧음
2. concurrent mark에서 위에 참조한 객체를 따라가면서 마크함
    - 다른 스레드 실행중인 상태임
3. remark 에서는 새로 생성되거나 참조가 끊긴 객체 확인 (정지)
4. concurrent sweep

- stop-the-world가 매우 짧아서 응답속도가 매우 중요한 서비스에서 사용
- 다른 GC보다 메모리와 CPU 많이 사용
- Compaction 단계가 제공되지 않음 → 문제 발생 가능성

### G1 GC

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/Java/img/G1_GC.png)

- young, old 영역이 없음
- 각 영역에 객체를 넣고 꽉 차면 다른 영역으로 옮김
- 성능이 가장 좋음
- 아직 검증 필요

---

참고자료 :

[https://doozi0316.tistory.com/entry/1주차-JVM은-무엇이며-자바-코드는-어떻게-실행하는-것인가](https://doozi0316.tistory.com/entry/1%EC%A3%BC%EC%B0%A8-JVM%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EB%A9%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%8B%A4%ED%96%89%ED%95%98%EB%8A%94-%EA%B2%83%EC%9D%B8%EA%B0%80)

[https://www.youtube.com/watch?v=UzaGOXKVhwU](https://www.youtube.com/watch?v=UzaGOXKVhwU)

[https://www.youtube.com/watch?v=vZRmCbl871I](https://www.youtube.com/watch?v=vZRmCbl871I)

[https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)  - garbage collection

[https://asfirstalways.tistory.com/158](https://asfirstalways.tistory.com/158) - jvm 자세한 정리

[https://yeon-kr.tistory.com/114](https://yeon-kr.tistory.com/114) - java memory model

[https://asfirstalways.tistory.com/159](https://asfirstalways.tistory.com/159) - GC가 삭제하는 객체 선정