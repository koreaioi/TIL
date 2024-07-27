# 결론
변경 감지를 사용하자   

변경 감지와 Merge의 변경된 값들이 모두 바뀌게된다는 공통점이있다  
 
하지만 변경감지는 변경하고자 하는 필드를 설정할 수 있지만, Merge를 사용하면 데이터의 필드가 변경하고자하는 데이터의 필드들로 모두 변경되므로 혹여나 변경하고자하는 데이터에 Null이 존재하면 필드에 Null이 들어가게 된다. 


# 영속성 컨텍스트 (간단)
Member member = new Member(); 로 객체만 생성한 경우 => 비영속 상태

memberService.save(member) 처럼 DB에 쿼리를 날린 경우 => 영속 상태

쿼리가 끝나고 해당 트랜잭션이 모두 끝남 => 준영속 상태   

## 변경 감지

영속성 컨텍스트에 있는 영속 엔티티에 한해서 변경을 감지함.   
트랜잭션 커밋 시점에 변경을 감지(Dirty Checking)이 동작해 DB에 UPDATE SQL이 실행된다.

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
  Item findItem = em.find(Item.class, itemParam.getId()); //같은 엔티티를 조회한
다.
  findItem.setPrice(itemParam.getPrice()); //데이터를 수정한다.
}
```

위 코드에서 findItem은 em.find로 찾은 영속 엔티티이다. 따라서 트랜잭션 커밋 시점에 변경을 감지해 변경 사항이 DB에 적용된다.

## 병합 (Merge), 비추
병합은 준영속 상태의 엔티티를 영속상태로 변경할 때 사용한다.

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
  Item mergeItem = em.merge(itemParam); //mergeItem은 영속 상태이다.
}
```

![병합merge](https://github.com/user-attachments/assets/88ef5d6d-1e7f-42a5-998e-925aa1388cb2)

1. Merge()를 실행되면 아래의 동작 순서를 따라 Merge()가 종료된다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
   2-1. 만약 1차 캐시에 엔티티가 없으면 DB에서 인티티를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 엔티티(mergeMember)에 member엔티티의 값을 채워넣는다. (여기서 member 엔티티의 모든 값을 mergeMember에 밀어 넣게되고 "회원1" 이라는 이름이 "회원명변경"으로 바뀐다. )
4. 영속 상태인 mergeMember를 반환한다.

#### 병합 3줄 요약
1. 준영속 엔티티의 식별자 값으로 영속 엔티티 조회 (1차 캐시 or DB)
2. 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체(병합)
3. 트랜잭션 커밋 시점에 변경 감지 기능으로 인하여 UPDATE SQL이 실행

### 병합의 주의사항
변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있으나, 병합을 사용하면 모든 속성이 변경된다.   
따라서 병합시 값이 없으면 null로 밀어넣어져 update 되는 위험이 있다.   
따라서 엔티티를 변경할때는 변경 감지를 사용하자.


# 엔티티 변경의 좋은 해결 방법

1. 컨트롤러에서 엔티티를 생성하지 말자 (어설프게 생성하지 말자)   
2. 트랜잭션이 있는 서비스 계층에 식별자와 변경할 데이터를 명확하게 전달하자. (파라미터 or dto)
3. 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고 엔티티의 데이터를 직접 변경하자.
4. 트랜잭션 커밋 시점에 변경 감지가 실행된다.
