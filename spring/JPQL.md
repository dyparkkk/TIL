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

---

참고자료 : 자바 ORM 표준 JPA 프로그래밍 - 김영한님