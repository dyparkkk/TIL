# Data JPA 2

## 사용자 정의 리포지터리 구현

- Spring Data JPA는 인터페이스만 정의, 구현체는 스프링이 자동 생성
- 직접 구현할 내용이 많음
- Querydsl, JDBC templete 등 사용 시 직접 정의 필요

```java
// 사용자 정의 인터페이스 
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}

// 구현체 -> 
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}

// 인터페이스 상속
public interface MemberRepository extends 
				JpaRepository<Member, Long>, MemberRepositoryCustom{
 ..
}
```

- 이름 규칙 : 리포지터리 인터페이스 이름 + Impl
    - 사용자 정의 리포지터리 인터페이스 이름 + Impl 도 가능 !! → 이걸 권장
- 스프링 데이터 JPA가 인식해서 빈으로 등록
- 참고 !!
    - 항상 필요한 것은 아님
    - 기능에 따라 여러 리포지토리를 만드는 것이 도움이 될 수 있음 ( 사용자 정의 리포지터리 말고)
    

## Auditing

- 엔티티를 생성, 변경할 때 시간 및 사람을 추적하기 위해 사용
    - 등록, 수정 시간, 등록자, 수정자

```java
// 순수 JPA 사용
@Getter
@MappedSuperclass
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createDate;
    private LocalDateTime updateDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createDate = now;
        updateDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updateDate = LocalDateTime.now();
    }
}
```

### Data JPA 사용

```java
// @EnableJpaAuditing 적어주어야 함
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}

// 등록자, 수정자를 처리해주는 AuditorAware
	@Bean
	public AuditorAware<String> auditorProvider(){
		// createBy, ModifiedBy 처리
		// 보통 시큐리티 컨택스트 홀더나 세션에서 값 가져와서 사용해줌
		return ()-> Optional.of(UUID.randomUUID().toString());
	}
}

// @EntityListeners(AuditingEntityListener.class) 필수 !! 
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass // jpa에서 컬럼만 상속받아서 사용하기 위한 어노테이션
@Getter
public class BaseTimeEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass // jpa에서 컬럼만 상속받아서 사용하기 위한 어노테이션
@Getter
public class BaseEntity extends BaseTimeEntity{

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

## Web 확장 - 페이징과 정렬

- 스프링 MVC를 더욱 편하게 사용하도록 하는 기술
- 파라미터로 pageable을 받는다
- Pageable은 인터페이스,  실제는 *org.springframework.data.domain.PageRequest* 객체 생성

```java
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("members")
    public Page<MemberDto> pageMembers(@PageableDefault(size = 5) Pageable pageable) {
        Page<MemberDto> map = memberRepository
                .findAll(pageable).map(MemberDto::new);
        return map;
    }

    @PostConstruct
    public void init() {
        for (int i =0; i< 100; i++){
            memberRepository.save(new Member("user" + i, i));
        }
    }
}
```

- 요청 파라미터 예시 */members?page=0&size=3&sort=id,desc&sort=username,desc*
- page : 현재 페이지, **0부터 시작**
- @PageableDefault() 로 size나 sort 기본 값 변경 가능
- Dto 사용해야 함(당연한 내용)
- page 1부터 시작하기
    - 직접 클래스 만들어서 처리

## 스프링 데이터 구현체

- *org.springframework.data.jpa.repository.support.SimpleJpaRepository*
- @Repository 적용 : JPA 예외를 스프링이 추상화한 예외로 변경
- 기본으로 Transaction이 걸려있음
    - 서비스 계층에서 트랜잭션이 시작하면 받아서 사용
    - 서비스 계층에서 트랜잭션 안하면 동작은 하지만 영속성 컨택스트에 저장은 안됨
        - 레포지터리에서 트랜잭션이 끝남 → flush, clear 해버림
- save() 메서드는 주의 !!
    
    

```java
@Transactional
	@Override
	public <S extends T> S save(S entity) {

		Assert.notNull(entity, "Entity must not be null.");

		if (entityInformation.isNew(entity)) {
			em.persist(entity);
			return entity;
		} else {
			return em.merge(entity);
		}
	}
```

- 새로운 객체 아니면 merge해버림 ( select 후에 insert ) → 비효율적
- 새로운 객체 판단 :
    - 식별자가 객체일 경우 : NULL
    - 자바 기본 타입일 경우 : 0
- persistable 인터페이스를 상속 받아서 해결가능 ( id를 특별한 값 사용할 때)

```java
@Entity
public class Item extends BaseTimeEntity implements Persistable<String> {

//    @Id @GeneratedValue
//    private Long id;

    @Id
    private String id;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return super.getCreateDate() == null;
    }
}
```

참고자료 : 김영한님 강의