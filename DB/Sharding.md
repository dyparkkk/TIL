# Sharding


같은 테이블 스키마를 사진 데이터를 다수의 데이터베이스에 분산해서 저장하는 방법

## 프로그래밍이나 운영적인 복잡성이 증가하므로 가능하면 피하자

- scale-in : 하드웨어 업그레이드
- read 부하가 크면 : Cache 나 db replication 적용 고려
- 일부 칼럼만 자주 사용 : vertically Partition ( 컬럼으로 테이블 자르기 ), cold storage(

[NAVER D2](https://d2.naver.com/helloworld/526125)

cold storage 참고 자료

분산된 데이터를 어떻게 read? 어떻게 잘 save?

분산이 잘 되지 않으면 hot spot 문제가 발생→ 성능 저하

균일한 분산이 목표

### hash sharding

shared key : db_id를 hash 해서 나눔 

2의 배수로 db를 늘리거나 줄이면 데이터 재분배(Resharding)가 절반으로 줄어듬 

분산이 잘 안될 수가 있음 ( 짝수 id에만 데이터 저장 등 )

### Dynamic Sharding

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/DB/img/sharding00.png)

Locator Service를 통해 Shard key를 얻는 방식 

유동적으로 DB node 개수를 늘릴 수 있음 

단점 

data relocation 하면 추가로 shard key 일치시켜줘야함 

locator를 성능을 위해 cache나 replication하면 잘못된 라우팅으로 인해 error 발생

### Entity Group

같은 partition에 관련있는 entity를 저장. 

single physical shard내에서 쿼리가 효율적임. 하나의 샤드에서 강력한 consistency가 이루어짐.

단점 : 

다수의 partition에 걸쳐 이루어 지는 쿼리는 당연히 비효율적 → 효율적인 read를 위해 두 파티션에 모두 저장되는 방식이 있음. ex) a와 b 의 메시지는 partition 두 곳에 저장 - read 빨라짐 

...