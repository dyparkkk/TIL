# Data JPA

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/dataJpa00.png)

- JPA를 편하게 사용할 수 있도록 지원해주는 프레임워크
- Spring Data JPA가 인터페이스의 구현 클래스를 생성해서 넣어준다.

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/dataJpa01.png)

## 쿼리 메서드 기능

- 메서드 이름으로 쿼리 자동 생성
- 메서드 이름으로 NamedQuery 호출
- @Query 어노테이션을 사용해서 리포지터리 인터페이스에 쿼리 직접 정의

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);

		@Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

- Dto로 조회 가능

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMemberDto();
```

- 컬렉션으로 in절 지원

```java
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") Collection<String> names);
```

- 다양한 반환 타입 지원

```java
/**
     * return type 변환 가능
     * --- 결과값이 없을때 ---
     * List -> []
     * Member -> null  // jpa 에서는 noResultException ...
     */
    List<Member> findListByUsername(String username);

    Member findMemberByUsername(String username);

    Optional<Member> findOptionalByUsername(String username);
```

- 편리한 page와 slice 지원

```java
// totalCount만 재정의 가능 
//    @Query(value = "select m from Member m left join m.team t",
//            countQuery = "select count(m.username) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);

    Slice<Member> findSliceByAge(int age, Pageable pageable);
```

- 벌크연산 쿼리

```java
/**
     * 벌크쿼리는 persistenceContext를 거치지 않기때문에
     * PC를 초기화 해줘야한다. clearAutomatically
     */
    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
```

- @EntityGraph를 이용한 편리한 페치 조인 지원

```java
		@Query("select m from Member m left join fetch m.team")
    List<Member> findMemberFetchJoin();

	// member를 가져올 때 team도 같이 가져옴
    @Override
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();
```

- 영속성 컨텍스트에 비용을 최적화 하기 위해 힌트 지원
    - 영속성 컨텍스트에 캐시 하지 않음
- 락 기능

```java
		@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
    Member findReadOnlyByUsername(String username);

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    List<Member> findLockByUsername(String username);
```

참고자료 : 김영한님 강의