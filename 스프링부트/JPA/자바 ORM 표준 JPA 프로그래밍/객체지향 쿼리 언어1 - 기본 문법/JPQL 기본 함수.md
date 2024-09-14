# JPQL 기본 함수
<hr>

- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE, INDEX(JPA 용도)
  - SIZE - 컬렉션의 크기를 돌려줌 "select size(t.members) from Team t"


# 사용자 정의 함수 호출
<hr>

- 하이버네이트는 사용전 방언에 추가해야한다.
  - 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.
  - DB 마다 이미 등록된 사용자 정의 함수들이 있긴하다.

[사용자 정의 함수 등록 방법](https://www.inflearn.com/community/questions/1096265/hibernate-6-custom-%ED%95%A8%EC%88%98-%EB%93%B1%EB%A1%9D-%EB%B0%A9%EB%B2%95-%EA%B3%B5%EC%9C%A0?srsltid=AfmBOopJsJN2FYWMwYxrNaHUF8I1BGscnpEzJIWkAufoAF9lY8F0nnpf)   
강의 내용과 현재 사용자 정의 함수 등록 방법이 달라짐   
위 방법 참고

```jpaql
select function('group_concat', i.name) from Item i 
```