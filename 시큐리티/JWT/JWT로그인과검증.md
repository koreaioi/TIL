# 1. JWT 로그인 과정

![JWT 로그인 과정](https://github.com/koreaioi/TIL/assets/147616203/b043f646-d670-460c-adbc-927ee4fa9be2)


1. 클라이언트로부터 온 요청에서 username과 password를 꺼낸다.
2. 꺼낸 값으로 UsernamePasswordAuthenticationToken을 만든다.
3. 만든 UsernamePasswordAuthenticationToken을 authenticationManager.authenticate(여기)에 넣어준다.
4. authenticate는 DetailService가 구현된 클래스를 찾아서 해당 클래스의 loadUserByUsername 실행한다.
5. loadUserByUsername은 UsernamePasswordAuthenticationToken에서 username에 해당하는 DB에 저장된 객체를 꺼내와 반환한다.
6. 즉 authenticate를 실행해서 실제 DB에 요청으로부터 온 username과 password가 일치하면 반환값이 있다는 말이다.
7. authenticate의 반환값이 들어오면 LoginFilter의 successfulAuthentication가 실행된다.
8. successfulAuthentication에서 JWT토큰을 발급해서 response header에 발급한 토큰을 넣어준다.


# 2. JWT 토큰 검증
```java

package com.example.springjwt.jwt;

import com.example.springjwt.dto.CustomUserDetails;
import com.example.springjwt.entity.UserEntity;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
/*
 * 로그인한 유저의 JWT토큰을 검증하는 필터
 * 검증하기 위해서 JWTUtil을 사용
 * */
@Component
public class JWTFilter extends OncePerRequestFilter {

    private final JWTUtil jwtUtil;

    public JWTFilter(JWTUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        //request의 헤더에서 Authorization를 찾는다. -> 담긴 값이 존재하는 지 확인
        String authorization = request.getHeader("Authorization");

        //authorization이 null or 접두사가 이상하면 종료
        if (authorization == null || !authorization.startsWith("Bearer ")) {

            System.out.println("token null");
            filterChain.doFilter(request, response); // 현재 필터 종료 -> 다음 필터로 넘긴다.

            // request헤더에서 추출한 authorization이 유효하지 않으면 종료
            return;
        }

        // "Bearer "부분 제거 후 순수 토큰만 획득하기 위함.
        String token = authorization.split(" ")[1];

        //토큰 소멸 시간 검증
        if (jwtUtil.isExpired(token)) {

            System.out.println("token expired");
            filterChain.doFilter(request, response);

            //조건이 해당되면 메소드 종료 (필수)
            return;
        }

        //토큰에서 username과 role 획득
        String username = jwtUtil.getUsername(token);
        String role = jwtUtil.getRole(token);

        //이 아래 과정은 임시 세션에 담기 위한 과정이다.
        //userEntity를 생성하여 값 set
        UserEntity userEntity = new UserEntity();
        userEntity.setUsername(username);
        userEntity.setPassword("temppassword"); //패스워드는 임시로 사용
        userEntity.setRole(role);

        //UserDetails에 회원 정보 객체 담기
        CustomUserDetails customUserDetails = new CustomUserDetails(userEntity);

        //스프링 시큐리티 인증 토큰 생성
        Authentication authToken =
                new UsernamePasswordAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());
        //세션에 사용자 등록
        SecurityContextHolder.getContext().setAuthentication(authToken);

        filterChain.doFilter(request, response);

        // 이제 이 필터를 SecurityConfig에 등록해야한다.
    }
}


```

## JWTFilter 등록하기
LoginFilter 가 실행되기전(Before) JWTFilter 검증을 수행한다.

```java

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final AuthenticationConfiguration authenticationConfiguration;
    private final JWTUtil jwtUtil;

    public SecurityConfig(AuthenticationConfiguration authenticationConfiguration, JWTUtil jwtUtil) {

        this.authenticationConfiguration = authenticationConfiguration;
        this.jwtUtil = jwtUtil;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {


        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());
				
				//JWTFilter 등록
        http
                .addFilterBefore(new JWTFilter(jwtUtil), LoginFilter.class);

        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil), UsernamePasswordAuthenticationFilter.class);

        http
                .sessionManagement((session) -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}

```
