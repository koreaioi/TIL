# 영속성 전이: CASCADE

**즉시로딩, 지연로딩, 연관관계 세팅과 전혀 연관이없다.**

- 특정 엔티티를 영속 상태로 만들 때,연관된 엔티티도 함께 영속상태로 만들고 싶을때 **영속성 전이**를 사용한다.
- ex: 부모 엔티티를 저장할 때, 자식 엔티티도 같이 저장.

## 예시 코드

### Parent
```java
@Entity
public class Parent{
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> childList = new ArrayLisc<>();
    
    public void addChild(Child child){ // 연관관계 편의 메서드
        childList.add(child);
        child.setParent(this);
    }
    
    // Getter, Setter ...
}

```

### Child
```java
@Entity
public class Child{    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
    
    // Getter, Setter ...
}
```

### psvm 코드

```java
// 영속성 전이가 없는 코드
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
em.persist(child1);
em.persist(child2);
```

```java
// 영속성 전이를 설정한 코드
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

// 영속성 전이 설정으로 parent만 영속성 컨텍스트에 저장해도 연관 객체가 같이 저장된다.
em.persist(parent);
//em.persist(child1);
//em.persist(child2);
```

## 영속성 전이 주의사항
- 영속성 전이는 연관관계를 매핑하는 것과 아무런 관련이 없다.
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공하는 도구이다.
- 어떤 단일 엔티티에 완전 종속적일때(단일 소유자일때), 라이프사이클이 거의 유사할 경우 사용하도록하자.

## CASCADE의 종류
- ALL: 모두 적용
- PERSIST: 영속
- REMOVE: 삭제
다른건 잘 사용안함

<hr>

# 고아 객체

- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능
- orphanRemoval = true

### Parent에 고아 객체 삭제 기능 코드
```java
@Entity
public class Parent{
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayLisc<>();
    
    public void addChild(Child child){ // 연관관계 편의 메서드
        childList.add(child);
        child.setParent(this);
    }
    
    // Getter, Setter ...
}
```

```java
Parent findparent = em.find(Parent.class, id);
em.remove(findparent); // parent가 삭제되면서 소유한 childList 엔티티도 같이 삭제된다.
```

## 고아 객체 - 주의사항
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 인식하고 삭제하는 기능이다.
- 참조하는 곳이 하나일 때, 사용해야한다.
- **특정 엔티티가 개인 소유할 때 사용한다.**
- @OneToOne, @OneToMany만 사용가능


# 영속성 전이 + 고아 객체, 생명주기

- ** CascadeType.ALL + orphanRemoval=true

- 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거할 수 있다.
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때, 유용하다.