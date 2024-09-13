# 프로젝션
- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입 (숫자, 문자등 기본 데이터 타입)   


- SELECT m FROM Member m -> 엔티티 프로젝션
- SECECT m.team FROM Member m -> 엔티티 프로젝션
- SECECT m.address FROM Member m -> 엔티티 프로젝션
- SECECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션
- DISTINCT로 중복 제거

## 엔티티 프로젝션

SELECT 절로 가져온 List 컬렉션 객체들은 영속성 컨텍스트의 관리를 받을까?

### 코드
```java
    Member member = new Member();
    member.setUsername("member1");
    member.setAge(10);
    em.persist(member);

    em.flush(); // member1 DB에 저장
    em.clear(); // 영속성 컨텍스트 비우기

    List<Member> result = em.createQuery("select m from Member m", Member.class)
            .getResultList();

    //UPDATE 쿼리가 나가면 findMember가 영속성 컨텍스트의 관리를 받는다는 뜻.
    Member findMember = result.get(0);
    findMember.setAge(20);

    tx.commit();
```

```java
em.createQuery("select m.team from Member m", Team.class)
        .getResultList();
```
위 코드의 경우 SQL로 번역시 Member의 team.id와 Team의 id로 JOIN하는 쿼리가 나가게 된다.
따라서 가독성이 좋도록 아래와 같이 코드를 짜는 것이 좋다.

```java
em.createQuery("select t from Member m join m.team t")
        .getResultList();
```
위 코드는 직전의 코드와 SQL 결과는 같지만 코드로 한눈에 JOIN이 일어남을 캐치할 수 있다.   
* t는 m.team의 별칭이다.
* Member 내부에 있는 Team을 join 하므로 Member와 Team이 id로 JOIN하는 쿼리가 일어난다.      

### 실행 결과
즉, JPQL을 통해 조회된 엔티티(프로젝션)은 영속성 컨텍스트의 관리를 받는다!
![ima  ge](https://github.com/user-attachments/assets/2649f39f-2c07-4a1f-9eac-8d5aea69fd1c)

## 임베디드 타입 프로젝션
### 코드
```java
    Member member = new Member();
    member.setUsername("member1");
    member.setAge(10);
    em.persist(member);

    em.flush(); // member1 DB에 저장
    em.clear(); // 영속성 컨텍스트 비우기

    List<Member> result = em.createQuery("select m from Member m", Member.class)
            .getResultList();

    //UPDATE 쿼리가 나가면 findMember가 영속성 컨텍스트의 관리를 받는다는 뜻.
    Member findMember = result.get(0);
    findMember.setAge(20);

    tx.commit();
```

### 실행 결과

Address는 Order의 값타입 (내부에 존재하는 필드)이므로 JOIN없이 SELECT문 하나면 된다.   

![image](https://github.com/user-attachments/assets/585a05f5-ef71-4e03-ba2a-c6803e47627e)

## 스칼라 타입 프로젝션

숫자, 문자, 날짜와 같은 기본 데이터 타입들을 스칼라 타입이라 한다.   
(벡터타입은 왜 없징ㅋ)   

내가 원하는걸 SQL 짜듯 막 가져온다.   

### 코드
```java
em.createQuery("select m.username, m.age from Member m")
        .getResultList();
```

### 실행 결과

![image](https://github.com/user-attachments/assets/9a83444e-1925-4bd1-b87a-80b88cdf19f4)

## 프로젝션 - 여러 값 조회

엔티티를 대상으로 조회하면 편리하긴 하다.   
하지만 꼭 필요한 데이터들만 선택해서 조회해야 할 때도 있다.   
프로젝션에 여러 값을 선택하면 TypeQuery를 사용할 수 없고 대신 Query를 사용해야 한다.   

- 1. Query 타입으로 조회 -> 추후 타입 캐스팅 
- 2. Object[] 타입으로 조회
- 3. new 명령어로 조회
  - 단순 값을 DTO로 바로 조회한다. SELECT new jpabook.jpql.UserDTO(m.uesrname, m.age) FROM Member m
  - 패키지 명을 포함한 전체 클래스 명 입력
  - 순서와 타입이 일치하는 생성자 필요

### TypeQuery, Query

```java
List resultList = em.createQuery("select m.username, m.age from Member m")
        .getResultList();

Object o = resultList.get(0);
Object[] result = (Object[]) o;
System.out.println("username = " + result[0]);
System.out.println("age = " + result[1]);
```

위 과정을 생략하기 위해 다음과 같이 제네릭에 Object[]를 넣어줘도 된다.   

```java
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m")
        .getResultList();
Object[] result = resultList.get(0);
```

### new 명령어로 조회

패키지 명을 다 적어야 한다.(java폴더 아래부터)   
querydsl을 사용하면 순수 자바 코드로 짜므로 이런 문제(패키지명을 다 적어야하는)를 해결 가능

```java
System.out.println("==================");
List<MemberDTO> resultList = em.createQuery("select new hellojpa.jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
                .getResultList();

MemberDTO memberDTO = resultList.get(0);
System.out.println("memberDTO.getUsername() = " + memberDTO.getUsername());
System.out.println("memberDTO.getAge() = " + memberDTO.getAge());
```

MemberDTO 클래스 코드
```java
public class MemberDTO {

  private String username;
  private int age;

  public MemberDTO(String username, int age) {
    this.username = username;
    this.age = age;
  }
  
  // Getter, Setter
}
```


#### 실행 결과
![image](https://github.com/user-attachments/assets/f2f7978d-e38e-495a-8d44-c9546ba9ef45)