# Mapped Superclass - 매핑 정보 상속
- 공통 매핑 정보가 필욯라 때 사용(id, name)

![image](https://github.com/user-attachments/assets/2adfd371-4c8f-487c-aede-e3b90827f1b3)

## 설명

- MappedSuperclass는 상속 관계 매핑이 아니다. (속성만 상속)
- MaapedSuperclass는 엔티티가 아니라, 테이블과 매핑되지 않는다.
- 부모 클래스를 상속 받는 자식클래스에 매핑 정보만 제공한다.
- 조회, 검색 불가(BaseEntity) 불가 (em.find(BaseEntity.class) 사용 불가)
- 즉, 직접 생성해서 사용할 일이 없으므로 추상 클래스로 만드는 것을 권장한다.
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용한다.


## 사용 예시
생성 시간, 수정 시간을 기록하도록한다. (기본적으로 필요하다)   
모든 엔티티에 일일이 아래의 필드를 추가해야한다.   

```java
private String createdBy;
private LocalDateTime createdDate;
private String lastModifiedBy;
private LocalDateTime lastModifiedDate;
```

상위 클래스에 두고 이를 상속받게하면 편할텐데... 이럴 때 사용하는게 MappedSuperclass이다.

```java
@MappedSuperclass
public abstract class BaseEntity { //BaseEntity 테이블은 CREATE되지 않는다.

    @Column(name = "INSERT_MEMBER") //@Column 속성도 사용할 수 있다.
    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
    
    // ...
}

```

상위(부모, 슈퍼) 클래스 역할을 하는 BaseEntity를 만들고 사용하고자 하는 엔티티에서 상속받는다.   

```java
@Entity
public class Member extends BaseEntity{

    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;
    
    // ...
}
```