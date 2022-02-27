# Transaction


- 더 이상 쪼갤 수 없는 작업의 최소 단위
- DB에 데이터 부정합을 방지하기 위해 사용

## 조건 : ACID

- Atomicity (원자성) :  all or nothing
- Consistency (일관성) : 수행 전과 후에 일관된 상태 유지
    - 트랜잭션 수행 전/후에 데이터 모델의 모든 제약 조건(기본 키, 외래 키, 도메인, 도메인 제약조건 등)을 만족
    - ‘이체시에 돈 총합 같아야함’ 같은 비명시적 일관성 조건도 포함
- Isolation (고립성) : 독립적으로 시행
- Durabiliy (지속성)

## 직렬 가능 스케줄 ( serialiable schedule)

- 두 개의 트랜잭션이 read만 수행하면, 상호 간섭이 발생하지 않고 연산 순서도 중요하지 않음
- 두 개의 트랜잭션이 같은 데이터 항목에 접근하지 않는다면, 상호 간섭이 발생하지 않고 연산 순서도 중요하지 않음
- T1이 x에 write를 하고 T2가 x에 read or write를 할 때 실행순서가 중요!

## 동시성제어 (Currency Control)

- 다중 사용사 환경에서 둘 이상의 트랜잭션이 동시에 실행될 때, 일관성 해치지 않도록 제어

### 문제점

- Lost Update : 하나의 트랜잭션이 갱신한 내용을 다른 트랜잭션이 덮어씀
- Dirty Read : A 트랜잭션이 쓰기 중에 B 트랜잭션이 읽어오고 A가 롤백한 경우 잘못된 결과 발생
- Inconsistency (모순성, repeatable read 문제)
- Cascading Rollback (연쇄 복귀) : 캡션

![](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/cascading_Rollback.png)


T1의 Read(Y) 이후에 Fail시 Roll back 해야 하는 경우 발생 가정 시 

T1의 개시 최초 상태인 X=500, y=500인 상태로 복귀해야 하지만 T2가 이미 트랜잭션을 완료하고 시스템을 떠난 상태이기 때문에 초기 상태로 복귀 불가함

### 기법

- Locking
    - 데이터를 접근할 때 lock을 검
    - s-lock(shared lock : 읽기만 가능), x-lock(exclusive lock: 읽기, 쓰기 모두 가능)
    - 트랜잭션 격리 수준에 따라서 데이터 쓰고 바로 락 풀지, 아니면 트랜잭션 끝날때까지 안풀지
- timestamp
    
    ![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/transaction_timestamp.png)
    
- 낙관적 검증
- MVCC (다중버전 병행제어 기법)
    - t1이 x를 수정중이면 수정 전에 데이터를 undo log에 저장한다
    - read committd 격리 수준에서는 undo log를 읽는다
    - repeatable read 격리 수준에서는 트랜잭션이 끝날때까지 undo log를 읽는다 ( t1의 수정이 commit되어도 ...)
    - storage engine이 불필요하다고 판단되면 undo-log 삭제
    

## 트랜잭션 고립 수준

### 문제

- dirty read
- Non-repeatable read
- Phantom read
    - T1이 읽고, T2가 데이터를 insert 하고, T1이 다시 읽었을 경우 없던 데이터가 생기는 문제
    

### Read Uncommitted

락 사용 x

### Read Committed

커밋 된거만 읽기 보장 수준

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/img/read_committed.png)

### Repeatable Read

다시 읽기 보장 수준

### Serializable

Gap lock 

Next-key lock : Gap lock + record lock

## Recovery

DB 장애로부터 이전의 상태로 복구시켜서 일관된 상태로 만드는 것

덤프 : 주기적으로 다른 저장장치에 복사 

로그 

참고 자료: 

[wiki.hash.kr/index.php/트랜잭션](wiki.hash.kr/index.php/트랜잭션)

[https://mangkyu.tistory.com/30](https://mangkyu.tistory.com/30)

[https://velog.io/@ha0kim/동시성-제어](https://velog.io/@ha0kim/동시성-제어)

읽어볼것:

[https://d2.naver.com/helloworld/407507](https://d2.naver.com/helloworld/407507)
