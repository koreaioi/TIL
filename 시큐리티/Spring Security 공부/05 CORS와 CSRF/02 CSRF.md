# CSRF

CSRF: Cross-Site Request Forgery   
사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록)를 특정 웹사이트에 요청하게 하는 공격이다.   

- CSRF는 보안 공격, 보안 위협이다.   
- 단, CSRF 공격은 사용자 정보, 세션 ID 그리고 쿠키를 노리지 않는다.

CSRF 시나리오 예시

1. 사용자가 뱅킹 웹 사이트에 로그인하여 세션을 시작합니다. 이 웹 사이트는 이 사용자를 쿠키를 통해 식별합니다.
2. 사용자가 이 뱅킹 웹 사이트를 닫지 않고, 새 탭에서 공격자가 만든 웹 사이트에 접속합니다.
3. 이 웹사이트에는 <img src="http://bank.com/transfer?to=attacker&amount=1000"> 와 같은 코드가 있습니다. 이 코드는 뱅킹 웹 사이트에 대한 요청을 발생시킵니다.
4. 브라우저는 이 요청을 보내고, 이 때 뱅킹 웹 사이트에 대한 사용자의 쿠키도 함께 보냅니다. 즉, 이 요청은 **사용자가 직접 보낸 것처럼 보입니다.**
5. 뱅킹 웹 사이트는 요청을 받아 사용자의 쿠키를 확인하고, 요청에 따라 1000달러를 공격자의 계좌로 이체합니다

## CSRF 공격 대처 방법

### CSRF 무시

운영상에서 추천하지 않는다.   
Spring Security가 기본적으로 제공하는 CSRF 보호를 완전히 꺼버리기 때문에 위험하다.   

```java
// 비추천 
http.csrf(auth -> auth.disable());
```

### CSRF 무시 경로 설정하기

특정 REST API에서 CSRF 보호를 무시하도록 한다.   


```java
http.csrf(csrf -> csrf.ignoringRequestMatchers("/contact", "/register"));
```

### CSRF - 토큰 활용하기

하지만 기본 Spring Security CSRF로는 부족하다.   
보다 완벽한 CSRF 보호를 위해 토큰을 활용해야한다.   

[CSRF 토큰 활용하기](https://intellectum.tistory.com/196)   

위 방식은 Spring Security 자체적으로 사용하는 세션인 sessionManagement를 사용한다.   
이는 JWT 토큰에 비해서 단점이 많아 실무에서는 잘 사용하지 않는다고 한다.   

그래도 대략 감만 잡아보자.(코드는 위 블로그에 자세히 나와있다.)   

1. CsrfTokenRequestAttributeHandler를 사용한다.   
   - 해당 핸들러는 CSRF 토큰과 요청 속성을 지정하는 핸들러이다.
   - 프론트측에 어떤 이름의 헤더 or 쿠키로 전달할 지 지정할 수 있다.
   - 기본 쿠키 이름은 'XSRF-TOKEN', 헤더 이름은 'X-XSRF-TOEKN'이다.   


2. Csrf 토큰이 있는 지 확인하는 커스텀 필터를 생성한다. (CsrfFilter)
   - 이 필터는 로그인 인증이 완료되는 BasicAuthenticationFilter 뒤에 위치시킨다.


3. 프론트 측에서는 'XSRF-TOKEN' 토큰을 저장하고 이를 헤더로 변경해서 백엔드 단에 넘겨주도록 한다.





