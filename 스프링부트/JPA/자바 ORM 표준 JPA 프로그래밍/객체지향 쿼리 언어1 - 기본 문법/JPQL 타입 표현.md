# JPQL 타입 표현
<hr>

- 문자: 'HELLO', 'She''s'
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE
- ENUM: jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)

### 사용 예시
```java
Team team = new Team();
team.setName("team1");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setAge(10);
member.setType(MemberType.USER);
member.changeTeam(team); // 연관관계 편의 메서드

em.persist(member);

em.flush(); // member1 DB에 저장
em.clear(); // 영속성 컨텍스트 비우기

System.out.println("==================");
String query = "select m.username, 'HELLO', true from Member m"+
        " where m.type = hellojpa.jpql.MemberType.USER";

// 파라미터 사용
//String query = "select m.username, 'HELLO', true from Member m"+
//        " where m.type = :userType";

List<Object[]> result = em.createQuery(query)
//        .setParameter("userType", MemberType.USER); // 타입 파라미터 사용 시
        .getResultList();

for(Object[] objects : result){
    System.out.println("objects[0] = " + objects[0]);
    System.out.println("objects[1] = " + objects[1]);
    System.out.println("objects[2] = " + objects[2]);
}
System.out.println("==================");

tx.commit();
```


<details>
<summary> ENUM MemberType 코드 </summary>


```java
package hellojpa.jpql;

public enum MemberType {
    ADMIN, USER
}
```

</details>

<details>
<summary>Member에 MemberType추가 코드</summary>

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
    private int age;

    @Enumerated(EnumType.STRING)
    private MemberType type;

    public MemberType getType() {
        return type;
    }

    public void setType(MemberType type) {
        this.type = type;
    }
}
```

</details>

### 실행 결과
![image](https://github.com/user-attachments/assets/5c147e54-24cd-4eda-983e-f136ffc22c2a)

# JPQL 기타
<hr>

- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, **IS NULL**