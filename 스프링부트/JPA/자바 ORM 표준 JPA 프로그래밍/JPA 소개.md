# 1. JPA 소개1

# SQL 중심적인 개발의 문제점

무한 반복되고 지루한 코드 (자바 객체 ↔ SQL)

## CRUD 관련 객체와 SQL 유지보수

기존에는 클래스와 해당하는 필드를 만들고 여기에 대한 SQL을 직접 써내려가야 했다.
여기서 문제는 객체에 필드가 추가되거나 변경되면 이와 관련된 SQL을 추가로 수정해서 유지보수가 힘듦

→ 객체를 DB에 CRUD하려면 너무 많은 코드를 작성 + 반복해야한다.

객체는 RDB, NoSQL, File등 다양하게 저장할 수 있지만, 현실적으로는 관계형 데이터베이스에 저장하게 된다. (+NoSQL은 보조적인 수단)

이런 선택을 함으로써 

- 객체 → SQL로 변환 → SQL이 RDB에서 동작

위와 같은 방식을 피할 수 없다.

이렇게 되면 결국 백엔드 개발자는 SQL 매퍼가 될 수 밖에 없다.

## 패러다임의 불일치

- 객체와 관계형 데이터베이스의 차이

### 1. 상속

RDB에는 객체에서 사용하는 상속 관계를 표현할 수 없다.

- 그나마 차선책이 슈퍼타입 서브타입 관계
부모 같은 테이블과 자식 같은 테이블을 만들어서 필요한 데이터를 조인하여 풀어내는 방식

![image](https://github.com/user-attachments/assets/8ba61764-e937-4344-9108-55c4af231dcb)

조회시 하지만 슈퍼타입 서브타입 관계는 JOIN이 필수적이고 복잡하다.

**JAVA에서는 컬렉션에 .add 를 사용하여 추가하고 get()을 사용하여 조회하듯
JPA에서도 컬렉션에 저장하듯 사용한다.**

---

### 2. 연관 관계

객체는 참조를 사용해 연관된 객체를 조회한다. ex) **member.getTeam()**

테이블은 외래 키를 사용해 조인하여 조회한다. **JOIN ON M.TEAM_ID = T.TEAM_ID**

![image](https://github.com/user-attachments/assets/28f7eb31-d7c6-454f-8b0d-802deb87dc83)

**추가적인 문제 → 객체를 테이블에 맞춰서 설계? or 객체지향의 특징을 살려서 설계?**

위 예시는 객체지향의 특징을 살린 (객체의 연관관계가 참조로 이루어진) 예시이다. 위 예시의 문제는 객체와 테이블이 구성되는 방식이 다르다는 것. 

member.getTeam.getTeamId()로 조인 FK에 쓸 ID를 가져온 뒤 SQL을 짜야하는 귀찮음이 있다. (개발자가 중간 과정에 개입해야한다.)

그렇다면 객체를 애초에 테이블 스럽게 설계하면 되지 않을까? 아래의 예시를 보자

![image](https://github.com/user-attachments/assets/1146844c-5716-47c5-ac7d-8cf56ea65165)

위 예시처럼 객체를 테이블에 맞추어 모델링을 하면 SQL 짤때는 편할 수 있지만, 객체지향의 특징을 잃어버리게 된다 (연관관계는 참조로)

**JPA는  외래키에 필요한 참조를 적절히 변환해서 SQL에 전달한다. (INSERT, SELECT 등에 적절히 사용되어 처리된다)**

---

### 3. 객체 그래프 탐색

![image](https://github.com/user-attachments/assets/7a3d5d09-cac5-4d68-a982-e47dbd130e84)

객체는 조회할 때 자유롭게 참조를 사용하여 탐색할 수 있어야한다. 

ex) order.getOrderItem().getItem().getCategoy() → 참조로 . . . 을 사용하여 자유롭게 탐색

**하지만 테이블은 어떤 SQL이 실행되느냐에 따라서 탐색 범위가 결정된다. 아래의 예시를 보자**

![image](https://github.com/user-attachments/assets/3826ec11-2397-41c5-ae4e-df9895754d9e)

위 SQL은 Member와 Team에 대해서만 조회를 해왔기 때문에 아래와 같은 문제가 생긴다.

member.getTeam()은 값이 있지만, getOrder()는 null이 나오게 된다.
어라? 객체에서는 .get 쓰면 무적인데? 자유롭게 탐색할 수 있는데? DB는 아니네?
이런 방식으로 조회해온 객체는 신뢰할 수 없게된다.

이를 해결하기 위해 즉, 자유로운 탐색을 구현하기 위해서 테이블과 관련된 모든 다른 테이블을 조인하여 조회하는 건 엄청난 손해이다. (모든 객체를 미리 로딩하는건 엄청난 손해이자, 잘못)

**JPA에서는 연관된 객체를 신뢰성 있게 조회하기 위해서 실제 객체가 사용되는 시점까지 DB 조회를 미루는 지연 로딩을 사용한다.**

---

### 4. 비교

![image](https://github.com/user-attachments/assets/417af409-48b6-44fa-9030-d8a76f20e40e)

위 그림을 보자. memberDAO를 통해서 member를 조회한 후 member1과 member2를 비교시 틀리게된다. → 객체 필드 값은 같으나 다른 객체라는 뜻이다.

![image](https://github.com/user-attachments/assets/2cf7bc7b-3d33-4218-b078-ab6f3b7b018e)

**하지만 위와 같이 자바의 컬렉션을 사용하면? 같다!**

---

### 객체를 자바 컬렉션에 저장하듯 DB에 저장할 수 없을까?

이를 위해 만들어 진 JPA!

---

# JPA란?
![image](https://github.com/user-attachments/assets/12d088b0-b6ee-48cb-8efa-c6e215a025ee)

JPA(Java Persistence API)는 자바 진영의 ORM 기술 표준이다.   
어플리케이션과 JDBC 사이에서 동작한다.   

### ORM
Object Relational Mapping - 객체와 관계형 데이터베이스를 매핑한다는 뜻이다.

![image](https://github.com/user-attachments/assets/5e90a441-ed88-43ad-b6c8-65ec15118dcc)

jpa.persist(member)를 통해서 즉, JPA를 사용해서 객체를 저장하는 코드이다.   
- persist: 보관하다.

![image](https://github.com/user-attachments/assets/2912c37c-c5f9-44a3-a578-b6a0fc93841b)
Member member = jpa.find(memberId)를 통해서 즉, JPA를 사용해서 객체를 조회하는 코드이다.   

# JPA를 사용하는 이유

## 생산성
- 자바 컬렉션에 객체를 저장하듯 JPA에게 저장할 객체를 전달한다.   
- INSERT SQL이나 JDBC API를 사용하는 반복적이고 지루한 일을 JPA가 대신 처리한다.
- SQL 중심적인 개발에서 객체 중심으로 개발하게 해준다.

## 패러다임 불일치
상속, 연관관계, 객체 그래프 탐색 같은 패러다임 불일치를 해결해준다.   
동일한 트랜잭션에서 조회한 객체의 동일성을 보장해준다.   

## 트랜잭션을 지원하는 쓰기 지연
1.JPA가 트랜잭션을 커밋할 때까지 INSERT SQL을 모아둔다.
2. JDBC BATCH SQL 기능을 사용해서 한번에 SQL을 전송한다.
3. 결과적으로 네트워크 통신 비용을 줄인다.

## 지연 로딩과 즉시 로딩
지연 로딩: 객체가 실제 사용될 때 로딩한다.   
즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회한다.   



