# 스프링 부트와 MySQL 연결하기

## DB 생성

1. MySQL 워크벤치에 접속
2. 커넥션 접속
3. create schema로 테이블 생성

## Application Properties

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/{DB명}?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

spring.jpa.properties.hibernate.dialect= org.hibernate.dialect.MySQLDialect
```

---

만약 spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver 가 빨간줄이 뜬다면,
아래 의존성을 추가하자.
```properties
	runtimeOnly 'com.mysql:mysql-connector-j'
```
