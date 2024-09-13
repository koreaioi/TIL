# 기본 문법과 쿼리 API
## JPQL소개
- JPQL은 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.
- JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JPQL은 결국 SQL로 변환된다.
- 매핑 정보 + 방언의 조합으로 SQL로 변환됨

![image](https://github.com/user-attachments/assets/0499d85c-2a99-4512-9aa0-02525ad72f46)


## 실습 코드
<details>
<summary>Memeber, Team 코드</summary>

### JpaMain

```java
package hellojpa;

import hellojpa.jpql.Member;
import jakarta.persistence.*;

import java.util.concurrent.ExecutionException;

public class JpaMain {

    public static void main(String[] args) {

        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member = new Member();
            member.setUsername("member1");
            em.persist(member);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            e.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();
    }
}

```

### Member
```java
package hellojpa.jpql;

import jakarta.persistence.*;

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
    private int age;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    // Getter, Setter

}
```

### Team
```java
package hellojpa.jpql;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.OneToMany;

import java.util.ArrayList;
import java.util.List;

@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    // Getter, Setter

}


```

### Order
```java
package hellojpa.jpql;

import jakarta.persistence.*;

@Entity
@Table(name = "ORDERS")
public class Order {

    @Id
    @GeneratedValue
    private Long id;
    private int orderAmount;

    @Embedded
    private Address address;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    // Getter, Setter
}


```

### Address (값 타입)

```java
package hellojpa.jpql;

import jakarta.persistence.Embeddable;

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    // Getter, Setter
}

```

### Product
```java
package hellojpa.jpql;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;

@Entity
public class Product {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int price;
    private int stockAmount;

    // Getter, Setter
}


```

</details>

## JPQL 문법

![image](https://github.com/user-attachments/assets/3fef856f-0507-43ed-ac28-b50883a10a8f)

- select m from Member as m where m.age > 18
- 엔티티와 속성은 대소문자로 구분한다. (Member, age)
- JPQL 키워드는 대소문자 구분 X (SELECT, FROM where)
- 엔티티 이름을 사용한다. 테이블 이름이 아니다.(Member)
- 별칭은 필수(m) (as 생략 가능)

### 집합과 정렬
집합과 정렬을 위해 사용한 SQL 문과 같이 JPQL에서도 사용 가능하다.
GROUP BY, HAVING
ORDER BY
![image](https://github.com/user-attachments/assets/c5b37f5e-8662-44e4-bdf2-e943c2bc6686)

### TypeQuery, Query
- TypeQuery: 반환 타입이 명확할 때 사용
- Query: 반환 타입이 명확하지 않을 때 사용한다.

```java
//SQL의 결과가 Member로 명확
TypeQuery<Member> query1 = em.createQuery(
        "select m from Member m", Member.class);

//SQL의 결과가 Member의 username을 명확
TypeQuery<String> query2 = em.createQuery(
        "select m.username from Member m", String.class);

// String 타입의 username과 int 타입의 age가 결과이다. -> Query 타입으로 반환
Query query3 = em.createQuery(
        "select m.username, m.age from Member m");
```


## 결과 조회 API
- query.getResultList(): 결과가 하나 이상일 때, 리스트 반환
  - 결과가 없으면 빈 리스트 반환 -> NPE 걱정 안해도 됨
- query.getSingleResult(): 결과가 정확히 하나만 나올 때 사용, 단일 객체 반환
  - 결과가 없으면: javax.persistence.NoResultException 
    - Spring 사용 시 NPE가 터지면 Null반환 or Optional로 감싸도록 설계됨
    - Spring Data JPA 쓸때랑 혼동하지 말자
  - 둘 이상이면: javax.persistence.NonUniqueResultExecption

### 실습 코드

```java
    public static void main(String[] args) {

        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member = new Member();
            member.setUsername("member1");
            member.setAge(10);
            em.persist(member);

            TypedQuery<Member> query = em.createQuery("select m from Member m where m.username = :username", Member.class);
            query.setParameter("username", "member1");
            Member singleResult = query.getSingleResult();
            System.out.println(singleResult);
    
            /* 메서드 체인            
            Member singleResult1 = em.createQuery("select m from Member m where m.username = :username", Member.class)
            .setParameter("username", "member1")
            .getSingleResult();
            */
            
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
            e.printStackTrace();
        } finally {
            em.close();
        }

        emf.close();
    }
```

