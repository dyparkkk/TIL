java진영의 ORM기술 표준

ORM (Object-relational mapping)

- 객체는 객체대로, RDB는 RDB대로 설계
- ORM 프레임워크가 중간에서 매핑

왜 필요? 

- sql 의존적인 설계( query에 맞춘 설계)
- 상속 표현?
- 모델링시 RDB에 맞춘 설계
- ...

mapping 

```java
@Entity
public class Member {

	@Id
	private Long id;
	private String name;
}
```

## 영속성 컨텍스트

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA00.png)

비영속 상태

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA01.png)
```java
Member member = new Member("박도영", 27);
```

영속 상태로

```java
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
//객체를 저장한 상태(영속)
em.persist(member);
```

영속성 컨텍스트의 이점

- 1차 캐시
- 동일성 보장
    - repeatable read 등급의 트랜잭션 격리수준을 db가 아닌 어플리케이션 차원에서 제공
- 트랜잭션을 지원하는 쓰기 지원
- 변경감지(Dirty Checking)
    - em.update  이런 코드가 없음
- 지연로딩(Lazy loading)

## Flush

영속성 컨텍스트의 변경내용을 데이터베이스에 반영

1. 변경감지가 동작해서 영속성 컨텍스트의 모든 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾음 → 쿼리를 만들어서 지연 sql 저장소에 등록
2. 쓰기 지연 sql저장소의 쿼리를 db에 전송

방법

1. em.flush()
2. transaction commit
3. JPQL 쿼리 실행
    - 영속성 컨텍스트에는 있지만 db에는 없는 내용이 있을 수 있어서
    - JPQL : 테이블이 아닌 엔티티 객체 대상으로 검색하는 객체지향쿼리 (sql에 의존 x)

@Column(nullable = false, length = 10) → DDL 자동 생성에만 영향을 주고 실제 JPA에는 영향 x

[[JPA] Entity Mapping Annotation 정리]!(https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA02.png)

## 연관관계

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA02.png)

```java
@Entity
public class Member {
	@Id @GeneratedValue
	private Long id;

	@Column(name = "USERNAME")
	private String name;

	private int age;

	// @Column(name = "TEAM_ID")
	// private Long teamId;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
	…
}
```

참고 : 

- JoinColumn 생략 가능
- name 기본값 : 필드명 + ‘_’ + 참조하는 테이블의 기본키 컬럼명

연관관계 저장

```java
//팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);  // 영속성 컨텍스트에 영속상태 !! ( 수정시에도 고려)

//회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team); //단방향 연관관계 설정, 참조 저장
em.persist(member);
```

### 양방향 연관관계 ( DB 테이블은 동일)

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA03.png)

- 단방향 두개 만든거랑 같음
- member가 연관관계의 주인
- 주인 아닌 쪽은 읽기만 가능

```java
@Entity
public class Team {
	@Id @GeneratedValue
	private Long id;

	private String name;

	@OneToMany(mappedBy = "team")
	List<Member> members = new ArrayList<Member>();
}
```

일대다 단방향 관계 매핑( 웬만하면 사용하지 말자)

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA04.png)

### 일대일 연관관계

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA05.png)

주 테이블에 외래 키 vs 대상 테이블에 외래 키

- 객체지향적으로는 주 테이블 + jpa 매핑 편리 → 이게 좋아보임
- 데이터베이스적으로는 대상 테이블

### 다대다 연관관계

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA06.png)
![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPA07.png)

연결용 테이블 추가해서 사용
@ManyToMany -> @OneToMany, @ManyToOne

---
참고자료 :  
자바 ORM 표준 JPA 프로그래밍 - 김영한님  