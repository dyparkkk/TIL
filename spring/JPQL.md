# JPQL

- JPA의 쿼리 방법 중 하나
- 객체지향 쿼리 언어, 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리
- SQL을 추상화해서 특정 SQL에 의존하지 않음
- JPQL은 결국 SQL로 변환됨

## 기본 문법

```java
// 예시 코드
public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try{
            Member member = new Member();
            member.setName("member1");
            em.persist(member);
						
						// JPQL 문법
            TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
						
						List<Member> resultList = query.getResultList();
            Member singleResult = query.getSingleResult();
						...
        }
    }
```

- 엔티티와 속성은 대소문자를 구분한다
- JPQL 키워드(select, from 등)은 대소문자 구분 X
- **별칭은 필수(Member m)**

결과 조회

- query.getResultList() : 결과가 없으면 빈 리스트 반환
- query.getSingleResult()

> NoResultException – if there is no result
NonUniqueResultException – if more than one result
...
> 

### 파라미터 바인딩

```java
TypedQuery<Member> query = em.createQuery("select m from Member m" +
                    " where m.name = :name", Member.class);
query.setParameter("name", "member1");
```

### 페이징 API

```java
List<Member> memberList = em.createQuery("select m from Member m", Member.class)
                    .setFirstResult(0)
                    .setMaxResults(10)
                    .getResultList();
```

### 조인

- 내부조인, 외부조인, 세타조인( 연관관계 없는 객체끼리 조인, 카테이션 곱)

```java
// 조인 대상 필터링 
JPQL:
SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'

SQL:
SELECT m.*, t.* FROM
Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'
```

### 경로 표현식

```java
select m.username -> 상태 필드
	from Member m
		join m.team t -> 단일 값 연관 필드
		join m.orders o -> 컬렉션 값 연관 필드
	where t.name = '팀A'
```

- 상태필드 : 단순히 값을 저장하기 위한 필드
    - 경로 탐색의 끝
- 단일 값 연관 필드 : @ManyToOne, @OneToOne, 대상이 엔티티 ex) m.team
    - 묵시적 내부 조인 발생, 추가 탐색 가능
- 컬렉션 값 연관 필드 : @OneToMany, 대상이 컬렉션
    - 묵시적 내부 조인, 추가 탐색 X

묵시적 내부 조인

- 항상 내부 조인
- 경로 탐색은 주로 select, where 절에서 사용 → sql의 from에 영향을 줌
- 사용하지 말라 → 조인은 sql의 중요 튜닝 포인트

### fetch join

- JPQL에서 성능 최적화를 위해 제공하는 기능 ( sql에서 제공하는 조인 X)
- 연관된 엔티티나 컬렉션을 SQL 한 번으로 조회 가능
- **즉시로딩 + 객체 그래프를 sql 한 번으로 조회**

```java
[JPQL]
select m from Member m join fetch m.team
-> [SQL]
SELECT M.*, T.* FROM MEMBER M
	INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

```java
일대다 관계, 컬렉션 페치 조인
• [JPQL]
select t from Team t join fetch t.members
	where t.name = ‘팀A'
-> [SQL]
SELECT T.*, M.*
	FROM TEAM T
		INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
	WHERE T.NAME = '팀A'
```

- 대부분의 N + 1 문제 해결
- 일대다 관계는 엔티티 중복을 조심 !!

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPQL00.png)

Distinct

- sql distinct 는 중복된 결과 제거
- jpql은 추가로 어플리케이션에서 중복된 엔티티 제거

한계

- 페치 조인 대상에는 별칭을 줄 수 없음
    - 하이버네이트는 가능하지만 가급적 사용 x
    - JPA의 매핑 원칙과 맞지 않음 → 컬렉션에는 모든 엔티티가 있어야함
- 둘 이상의 컬렉션은 페치 조인 X → 1:N:M ( 너무 커짐)
- 페치 조인에서는 페이징 API를 사용할 수 없음
    - 단일값 연관필드(일대일, 다대일)은 사용가능
    - 컬렉션 값들을 distinct하고 페이징 해야함 → 하이버네이트는 메모리에서 해줌 (매우 위험)

정리

- 모든 것을 페치조인으로 해결 불가
    - ( @BatchSize - lazy 로딩시 size를 지정해서 가져옴) 등 사용
- 여러 테이블을 조인해서 엔티티가 아닌 다른 모양을 내야한다면 일반조인 후 DTO에 반환하는 것이 효과적

### 엔티티 직접 사용

```java
JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용
• [JPQL]
select count(m.id) from Member m //엔티티의 아이디를 사용
select count(m) from Member m //엔티티를 직접 사용

-> [SQL](JPQL 둘다 같은 다음 SQL 실행)
select count(m.id) as cnt from Member m
```

- 어찌보면 당연함 ( 엔티티 객체는 DB에서 결국 pk로  식별하기때문)

### Named 쿼리 - 정적 쿼리

- 미리 이름을 정해두고 사용하는  JPQL
- 어플리케이션 로딩시점에 초기화되고 캐시에 저장되서 사용 → 성능 좋아짐
- **어플리케이션 로딩 시점에 쿼리를 검증해줌**
- Spring Data JPA 에서는 인터페이스에 선언하고 사용 (보통 이렇게 씀)

```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query="select m from Member m where m.username = :username")
public class Member {
	...
}

List<Member> resultList =
	em.createNamedQuery("Member.findByUsername", Member.class)
		.setParameter("username", "회원1")
		.getResultList();
```

### 벌크 연산

- 쿼리 한번으로 여러 테이블 로우 변경
- JPA 변경 감지 기능으로 실행하기엔 SQL이 많이 필요할 때 사용
    1. select
    2. 가격증가
    3. 트랜잭션 커밋 → 변경감지 작동
    - 변경된 데이터가 100이라면 update문 100개 필요...
- executeUpdate()는 영향 받은 엔티티 수 반환
- Update, Delete 지원 (Insert into ... select - 하이버네이트 지원)

```java
String qlString = "update Product p " +
		"set p.price = p.price * 1.1 " +
		"where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
		.setParameter("stockAmount", 10)
		.executeUpdate();
```

주의 

- 벌크 연산은 영속성 컨텍스트를 무시하고 실행됨 !!
- 벌크 연산을 먼저 실행하던지, 벌크 연산 후 영속성 컨택스트 초기화 진행

---
참고자료 :  
자바 ORM 표준 JPA 프로그래밍 - 김영한님  