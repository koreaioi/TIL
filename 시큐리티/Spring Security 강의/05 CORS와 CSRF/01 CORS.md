# CORS

CORS는 CROSS-ORIGIN RESOURECE SHARING이다.      

두 가지의 출처가 다름을 의미한다.   
여기서 두 가지의 출처는 프론트, 백엔드 어플리케이션을 의미한다.   

- 출처: URL (URL(도메인) + HTTP프로토콜 + 포트)

따라서 프론트 어플리케이션 포트가 3000이고, 백엔드 어플리케이션 포트가 8080 이므로 CORS에 의해서 차단된다.   

## CORS 해결 방법

### @CrossOrgin 어노테이션 사용

REST API를 정의한 곳에 @CrossOrigin 주석을 사용하자!

```java
@CrossOrigin(origins = "http://localhost:3000") // 특정 도메인 허용
```

하지만 일일히 모든 controller에 @CrossOrigin 어노테이션을 추가하기는 힘들다.


## SecurityFilterChain에 CorsConfig 설정 추가하기

### CorsConfig

Cors관련 정책을 설정할 클래스를 따로 분리하기 위하기 위해 CorsConfig 클래스를 만든다.      

config를 반환하는 static 메서드 함수를 추가로 만든다.   

```java
public class CorsConfig {

    public CorsConfig(){}

    public static CorsConfigurationSource corsConfigurationSource() {
        // Cors 설정 객체 - Spring Security가 제공
        CorsConfiguration configuration = new CorsConfiguration();

        // 리소스를 허용할 URL 지정하기
        List<String> allowedOriginPatterns = new ArrayList<>();
        allowedOriginPatterns.add("http://localhost:3000");
        allowedOriginPatterns.add("http://127.0.0.1:3000");
        configuration.setAllowedOrigins(allowedOriginPatterns);

        // 허용하는 HTPP METHOD 지정
        ArrayList<String> allowedHttpMethods = new ArrayList<>();
        allowedHttpMethods.add("GET");
        allowedHttpMethods.add("POST");
        allowedHttpMethods.add("PUT");
        allowedHttpMethods.add("DELETE");
        configuration.setAllowedMethods(allowedHttpMethods);

        configuration.setAllowedHeaders(Collections.singletonList("*"));
        //configuration.setAllowedHeaders(List.of(HttpHeaders.AUTHORIZATION, HttpHeaders.CONTENT_TYPE));

        // 인증, 인가를 위한 credentials를 TRUE로 설정
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }

}

```

### SecurityFilterChain 설정에 등록하기

```java
@Configuration // Spring이 이 Class를 Bean으로서 스캔할 수 있도록 어노테이션 설정
public class ProjectSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {


        http.csrf(auth -> auth.disable()); //csrf 코드
        http.authorizeHttpRequests((requests) -> {
            requests.requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards")
                    .authenticated()
                    .requestMatchers("/notices", "/contact", "/register").permitAll();
        });
        // cors 설정
        http.cors(cors -> cors.configurationSource(CorsConfig.corsConfigurationSource()));

        return (SecurityFilterChain)http.build();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

}

```

## 번외 - 경로별로 다른 CORS를 설정하고 싶을 때

Order를 통해서 빈의 로드 순서를정의할 수 있다.


```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	@Order(0)
	public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
		http
			.securityMatcher("/api/**")
			.cors((cors) -> cors
				.configurationSource(apiConfigurationSource())
			);
			//...
		return http.build();
	}

	@Bean
	@Order(1)
	public SecurityFilterChain myOtherFilterChain(HttpSecurity http) throws Exception {
		http
			.cors((cors) -> cors
				.configurationSource(myWebsiteConfigurationSource())
			);
			//...
		return http.build();
	}

	CorsConfigurationSource apiConfigurationSource() {
		CorsConfiguration configuration = new CorsConfiguration();
		configuration.setAllowedOrigins(Arrays.asList("https://api.example.com"));
		configuration.setAllowedMethods(Arrays.asList("GET","POST"));
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", configuration);
		return source;
	}

	CorsConfigurationSource myWebsiteConfigurationSource() {
		CorsConfiguration configuration = new CorsConfiguration();
		configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
		configuration.setAllowedMethods(Arrays.asList("GET","POST"));
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", configuration);
		return source;
	}

}
```