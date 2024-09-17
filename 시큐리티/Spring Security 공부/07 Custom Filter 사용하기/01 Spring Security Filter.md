# Spring Security Filter

참고 이미지

![image](https://github.com/user-attachments/assets/7ac6d401-32fa-4cc1-99fa-822d288ae029)


![image](https://github.com/user-attachments/assets/e865c595-217c-4ee4-9a26-41ea1aeaacc2)

## DelegatingFilterProxy

여기서 DelegatingFilterProxy는 실제 보안 처리는 하지 않고 위임만 하는(브릿지 역할) Servlet Container에서 동작하는 Servlet Filter이다.   

Spring Security는 DelegatingFilterProxy라는 필터를 만들어서 Main Filter Chain에 끼워 넣는다.   
그러므로 DelegatingFilterProxy는 Bean으로 등록된 Spring Security의 필터를 사용하는 시작점이 된다.   

![image](https://github.com/user-attachments/assets/e69810a1-1bd7-43b5-a56a-58c1c9e7a1bf)

## FilterChainProxy

사용자가 처음 요청을 하면 DelegatingFilterProxy가 요청을 받고 FilterChainProxy에게 요청을 위임한다.

![image](https://github.com/user-attachments/assets/9984c63c-852e-48d2-8454-497a0ef5b118)

## Filter의 종류

- `SecurityContextPersistenceFilter`: SecurityContextRepository에서 SecurityContext를 가져오거나 생성


- `LogoutFilter`: 로그아웃 요청을 처리


- `UsernamePasswordAuthenticationFilter`: ID와 Password를 사용하는 실제 Form 기반 유저 인증을 처리

  - Authentication 객체를 만들고 AuthenticationManager에게 인증처리를 맡긴다.
  - AuthenticationManager는 실질적인 인증을 검증 단계를 총괄하는 AuthenticationProvider에게 인증 처리를 위임한다. 그렇게 해서 UserDetailService와 같은 서비스를 사용해서 인증을 검증할 수 있는 것이다.

- `ConcurrentSessionFilter`: 동시 세션과 관련된 필터(이중 로그인)


- `RememberMeAuthenticationFilter`: 세션이 사라지거나 만료 되더라도, 쿠키 또는 DB를 사용하여 저장된 토큰 기반으로 인증을 처리


- `AnonymousAuthenticationFilter`: 사용자 정보가 인증되지 않았다면 익명 사용자 토큰을 반환


- `SessionManagementFilter`: 로그인 후 Session과 관련된 작업을 처리


- `ExceptionTranslationFilter`: 필터 체인 내에서 발생되는 인증, 인가 예외를 처리


- `FilterSecurityInterceptor`: 권한 부여와 관련한 결정을 AccessDecisionManager에게 위임해 권한부여 결정 및 접근 제어를 처리


- `HeaderWriterFilter`: Request의 HTTP 헤더를 검사해 Header를 추가하거나 빼주는 필터


- `CorsFilter`: 허가된 사이트나 클라이언트의 요청인지 검사


- `CsrfFilter`: CSRF Tocken을 사용하여 Csrf공격을 막아주는 기능을 제공