# 페이징 API

- JPA는 페이징을 다음 두 API로 추상화한다. (데이터베이스마다 페이징 처리 문법은 다르다.)
- setFristResult(int startPosition): 조회 시작 위치 (0부터 시작)
- setMaxResult(int maxResult): 조회할 데이터 수

## 코드

```java
for (int i = 0; i < 100; i++) {
    Member member = new Member();
    member.setUsername("member" + i);
    member.setAge(i);
    em.persist(member);
}

em.flush(); // member1 DB에 저장
em.clear(); // 영속성 컨텍스트 비우기

List<Member> result = em.createQuery("select m from Member m order by m.age desc", Member.class)
        .setFirstResult(0) // 첫번째 값부터
        .setMaxResults(10) // 10개 조회
        .getResultList();

System.out.println("result.size() = " + result.size());
for (Member member1 : result) {
    System.out.println("member1 = " + member1);
}


tx.commit();
```
## 실행 결과
내림차순 desc
![image](https://github.com/user-attachments/assets/dc77ad4e-c997-44e2-a4ea-649d57e51d34)

오름차순 asc
![image](https://github.com/user-attachments/assets/3517f4a3-4650-4b6a-b2d6-0af523362947)

## DB에 따른 문법

HSQLDB, MySQL, ORACLE, PostgreSQL 등등 페이징 문법이 다르다.   
하지만 데이터베이스 방언을 설정만 잘해주면 JPA가 알아서 해당 데이터베이스 문법에 맞춰서 페이징 해준다.   