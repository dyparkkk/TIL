3가지 전략 

## 1. 조인전략

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPAmapping00.png)

- 정규화 가능

## 2. 단일 테이블

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPAmapping01.png)

- 단순함

## 3. 구현클래스마다 테이블

![Untitled](https://raw.githubusercontent.com/dyparkkk/TIL/main/spring/img/JPAmapping02.png)

- 조회할때 힘듬 → 잘 안씀

Annotation

```java
@Inheritance(strategy=InheritanceType.xxx)
// joined , single_table, table_per_class

@DiscriminatorColumn 
// 구분 칼럼

```

## MappedSuperclass

- 공통 매핑 정보가 필요할 때 사용 ex) createdDate ...
- 자식 클래스에 매핑 정보만 제공
- 엔티티 x, 테이블과 매핑 x → 조회, 검색 불
- 추상 클래스 권장

```java
@MappedSuperclass
@Getter @Setter
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private LocalDateTime lastModifiedDate;
}
```
