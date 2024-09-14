# 조건식 - CASE 식
<hr>

### 기본 CASE 식
```jpaql
select
    case when m.age <= 10 then '학생요금'
         when m.age >= 60 then '경로요금'
         else '일반 요금'
    end
from Member m
```

#### 코드

    Member member = new Member();
    member.setUsername("member1");
    member.setAge(10);
    member.setType(MemberType.USER);
    member.changeTeam(team); // 연관관계 편의 메서드
    
    Member member1 = new Member();
    member1.setUsername("member2");
    member1.setAge(60);
    member1.setType(MemberType.USER);
    member1.changeTeam(team); // 연관관계 편의 메서드
    
    em.persist(member);
    em.persist(member1);
    
    em.flush(); // member1 DB에 저장
    em.clear(); // 영속성 컨텍스트 비우기
    
    System.out.println("==================");
    String query =
            " select " +
                    " case when m.age <= 10 then '학생요금'" +
                    "      when m.age >= 60 then '경로요금'" +
                    " else '일반 요금' " +
                " end " +
            " from Member m ";
    
    List<String> result = em.createQuery(query, String.class).getResultList();
    
    for(String o : result){
        System.out.println("o = " + o);
    }
    
    System.out.println("==================");


#### 실행 결과

![image](https://github.com/user-attachments/assets/ebe75a85-2efa-4199-b9a1-05e718e956cc)


### 단순 CASE 식
```jpaql
select
    case t.name
        when '팀A' then '인센티브110%'
        when '팀B' then '인센티브120%'
        else '인센티브 105%'
    end
from Team t
```

#### 실습 코드 PASS

### COALESCE
하나씩 조회해서 null이 아니면 반환

```jpaql
// 사용자 이름이 없으면 이름 없는 회원을 반환
select coalesce(m.username, '이름 없는 회원') from Member m
```

#### 실습 코드

        Member member = new Member();
        member.setUsername(null);
        member.setAge(10);
        member.setType(MemberType.USER);
        member.changeTeam(team); // 연관관계 편의 메서드


        em.persist(member);

        em.flush(); // member1 DB에 저장
        em.clear(); // 영속성 컨텍스트 비우기

        System.out.println("==================");
        String query = "select coalesce(m.username, '이름 없는 회원') from Member m ";

        List<String> result = em.createQuery(query, String.class)
        .getResultList();
        for(String o : result){
        System.out.println("o = " + o);
        }

        System.out.println("==================");

        tx.commit();

#### 실행 결과

![image](https://github.com/user-attachments/assets/df64c123-6e38-4d06-95c7-3f2763471965)

### NULLIF
두 값이 같으면 null 반환, 다르면 첫 번째 값 반환
```jpaql
// 사용자 이름이 '관리자'면 null 반환, 나머지는 본인의 이름을 반환
// 주로 관리자의 이름을 숨기기위해 사용
select NULLIF(m.username, '관리자') from Member m
```

#### 실습 코드

            Member member = new Member();
            member.setUsername("관리자");
            member.setAge(10);
            member.setType(MemberType.USER);
            member.changeTeam(team); // 연관관계 편의 메서드

            Member member1 = new Member();
            member1.setUsername("wooseok");
            member1.setAge(10);
            member1.setType(MemberType.USER);
            member1.changeTeam(team); // 연관관계 편의 메서드

            em.persist(member);
            em.persist(member1);

            em.flush(); //DB에 저장
            em.clear(); // 영속성 컨텍스트 비우기

            System.out.println("==================");
            String query = "select nullif(m.username, '관리자') from Member m ";

            List<String> result = em.createQuery(query, String.class)
                    .getResultList();
            for(String o : result){
                System.out.println("o = " + o);
            }

            System.out.println("==================");
            tx.commit();

#### 실행 결과

![image](https://github.com/user-attachments/assets/7debbea3-9a0d-422f-b99d-9f7982d550df)