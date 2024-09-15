

# SecurityFilterChain 커스텀하기 (Bean 생성하기)

SecurityFilterChain을 커스텀한다는 건 해당 Bean을 정의한다는 의미와 같다.   

아래와 같은 패키지 구조와 Class를 생성해준다.   
![image](https://github.com/user-attachments/assets/8af5a8c0-b2d4-4009-89e6-cbf8ae3620aa)

```java
@Configuration // Spring이 이 Class를 Bean으로서 스캔할 수 있도록 어노테이션 설정
public class ProjectSecurityConfig {
    
    // SpringBootWebSecurityConfiguration 코드를 그대로 복붙 + @Order는 필요 X
    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        
        // 모든 요청이 default로 보호된다는 뜻
        http.authorizeHttpRequests((requests) -> {
            ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
        });
        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain)http.build();
    }
}

```

위와 같이 defaultSecurityFilterChain 메서드를 @Bean 등록해주고 실행하면 기존 처럼 잘 작동 된다.   
기존 SpringBootWebSecurityConfiguration 코드를 그대로 복사해서 새로운 SecurityFilterChain을 만든 것이므로 전과 후 결과는 같다.   
하지만 ProjectSecurityConfig 클래스의 SecurityFilterChain으로 동작하게 된다.   

## 다음 시간

다음은 SecurityFilterChain 커스텀 방법을 통해 자체적인 맞춤 요구사항들을 적용해보겠다.   