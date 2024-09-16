# PasswordEncoder

저번 시간에 비밀번호를 암호화하지 않고 저장했다.    
이 때문에 UserDetailsService에서 유효한 유저인지 검증할 때 자꾸 거부되었다.   

강의를 통해 패스워드 인코더를 적용해보자!

- 참고
  - 실제 비밀번호 비교는 DaoAuthenticationProvider에서 일어난다.
  - 이 클래스에서 인코딩된 비밀번호를 비교하므로, DB에 인코딩되지 않은 패스워드가 있으면 401 에러가 뜨게된다.
  - additionalAuthenticationChecks메서드에서 비교한다.

<hr>

## 내가 직접 추가한 코드 (강의 보기 전)

내가 직접 BCryptPasswordEncoder를 추가해서 적용했다.   
강의랑 별 다를게 없다.

```java
@Controller
public class LoginController {

    CustomerRepository customerRepository;
    BCryptPasswordEncoder bCryptPasswordEncoder = bCryptPasswordEncoder();

    LoginController(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }
    
//     @PostMapping("/register")
//     ... 생략
//     customer.setPwd(bCryptPasswordEncoder.encode(customer.getPwd()));
//     savedCustomer = customerRepository.save(customer);
//     ... 생략

}
```

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
        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain)http.build();
    }

    // BCryptPasswordEncoder 빈 등록
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

}

```

## 회원가입 하면?

![image](https://github.com/user-attachments/assets/ce7ddce2-80dc-44fa-a1db-72f48564a5fb)

Password Encode가 잘 된다.


## 로그인도 잘 될까?

비밀번호 인코딩 후 로그인 과정에도 문제가 없어졌다!!

![image](https://github.com/user-attachments/assets/8a39ec98-1613-4ad5-8496-08e2b35776b2)