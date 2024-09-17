# Custom Filter

Custom Filter를 만들고 싶다면! Filter 인터페이스를 확장하면 된다!

<hr>

## Filter Interface

Filter 인터페이스
```java
package jakarta.servlet;

import java.io.IOException;

public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```

doFilter 메소드는 3가지 인자를 가진다.

1. ServletRequest - 엔드 유저로부터 오는 HTTP input 요청
2. ServletResponse - 엔드유저나 클라이언트에게 다시 보낼 HTTP 응답
3. FilterChain - FilterChain에 속한 다음 Filter에게 넘겨주기



## Spring Security Filter Chain에 CustomFilter 주입하기

1. addFilterBefore(filter, class) - adds a filter before the position of the specified filter class
2. addFilterAfter(filter, class) - adds a filter after the position of the pecified filter class
3. addFilterAt(filter, class) - adds a filter at the location of the specified filter class


# 실습 해보기

아래와 같이 동작하는 테스트 필터를 만들어 실습해보자!   

![image](https://github.com/user-attachments/assets/8c6a63dd-e634-4166-babe-4c255006b55a)


## RequestValidationFilter.class
헤더 값으로 부터 AUTHORIZATION이 있는 지 확인하고 해당 헤더 값이 test인지 확인하고 일찍 종료하는 의도를 가진 Filter

```java
public class RequestValidationFilter implements Filter {
    // 클래스 레벨 필드
    public static final String AUTHENTICATION_SCHEME_BASIC = "Basic";
    private Charset credentialsCharset = StandardCharsets.UTF_8;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {

        HttpsServletRequest req = (HttpsServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        String header = req.getHeader(AUTHORIZATION);
        if(header != null) { // 헤더에 AUTHORIZATION이라는 값이 있으면
            header = header.trim();
            if (StringUtils.startsWithIgnoreCase(header, AUTHENTICATION_SCHEME_BASIC)) {
                byte[] base64Token = header.substring(6).getBytes(StandardCharsets.UTF_8);
                byte[] decoded;

                try {
                    decoded = Base64.getDecoder().decode(base64Token);
                    String token = new String(decoded, credentialsCharset);
                    int delim = token.indexOf(":");
                    if (delim == -1) {
                        throw new BadCredentialsException("Invalid basic authentication token");
                    }
                    String email = token.substring(0, delim);
                    if (email.toLowerCase().contains("test")) {
                        res.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                        return;
                    }
                } catch (IllegalArgumentException e) {
                    throw new BadCredentialsException("Failed to decode basic authentication token");
                }
            }
            filterChain.doFilter(request, response);
        }

    }
}

```

## CustomFilter 주입하기

여기선 addFilterBefore를 사용해서 주입한다.   

```java
@Configuration // Spring이 이 Class를 Bean으로서 스캔할 수 있도록 어노테이션 설정
public class ProjectSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        CsrfTokenRequestAttributeHandler requestHandler = new CsrfTokenRequestAttributeHandler();
        requestHandler.setCsrfRequestAttributeName("_csrf"); // 명시하지 않아도 default가 _csrf 이다.

        http.csrf(auth -> auth.disable()); //csrf 코드
        http.authorizeHttpRequests((requests) -> {
            requests.requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards")
                    .authenticated()
                    .requestMatchers("/notices", "/contact", "/register").permitAll();
        });
        http.cors(cors -> cors.configurationSource(CorsConfig.corsConfigurationSource()));
        http.csrf(csrf -> csrf.csrfTokenRequestHandler(requestHandler).ignoringRequestMatchers("/contact", "register")
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()));

        // CustomFilter 주입하기 BasicAuthenticationFilter가 실행되기 전(앞 단계)에 위치
        http.addFilterBefore(new RequestValidationFilter(), BasicAuthenticationFilter.class);

        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain)http.build();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

}

```

## addFilterAfter()

![image](https://github.com/user-attachments/assets/e9a5e1ec-3ff6-4a5c-9061-818c7c7838b0)
위와 같은 경우   

```java
    http.addFilterAfter(new LoggingFilter(), BasicAuthenticationFilter.class);
```

## addFilterAt()

![image](https://github.com/user-attachments/assets/d0c54010-a7e1-4c3f-a6af-2015445120fd)

위와 같은 경우

```java
    http.addFilterAt(new LoggingFIlter(), BasicAuthenticationFilter.class);
```
