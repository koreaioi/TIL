# 맞춤 보안 설정하기

- /myAccount, /myBalance, /myLoans, /myCards -> 보안 O   
- /notices, /contact -> 보안 X   

## authorizeHttpRequests 변경하기

authorizeHttpRequests에는 기존에 모든 요청이 보호되는 default 설정이 있었다.   
```java
    http.authorizeHttpRequests((requests) -> {
        ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
    });
```

이 대신에 맞춤 보안 설정을 해주자.

        http.authorizeHttpRequests((requests) -> {
            requests.requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards").authenticated()
                    .requestMatchers("/notices", "/contact").permitAll();
        });

위 코드를 작성하면    
- /myAccount, /myBalance, /myLoans, /myCards 경로는 인증이 요구된다.   
- /notices, /contact 경로는 permitAll로 보안 설정 없이 허용한다.   
위 로직대로 구동된다.


## 실행

맞춤 보안 의도대로 잘 적용된다!   

## 최종 코드

```java
@Configuration // Spring이 이 Class를 Bean으로서 스캔할 수 있도록 어노테이션 설정
public class ProjectSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((requests) -> {
            requests.requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards")
                    .authenticated()
                    .requestMatchers("/notices", "/contact").permitAll();
        });
        
        // default formLogin과 httpBasic 사용
        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain)http.build();
    }
}

```