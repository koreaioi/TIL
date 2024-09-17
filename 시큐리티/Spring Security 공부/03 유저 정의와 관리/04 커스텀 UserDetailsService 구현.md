# 커스텀 UserDetailsService 구현하기 - 준비


## 디렉토리 구조

![image](https://github.com/user-attachments/assets/6c87ec08-6a11-4156-94b1-7f4e5f62b4b9)


### Entity 만들기
```java
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String pwd;
    private String role;

    // Getter, Setter
}

```

### Repository 만들기

```java
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {

    Customer findByEmail(String email);

}
```

# 커스텀 UserDetailsService 구현하기

## Interface UserDetailsService 알아보기
UserDetailsService인터페이스를 구현하기 위해서는,   
loadUserByUsername만 구현하면 된다!
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.security.core.userdetails;

public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}

```


## Custom UserDetailsService 구현하기

위에서 말한 것처럼 CustomUsreDetailsService를 만들고 loadUserByUsername을 구현하자!

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return null;
    }
}
```

## loadUserByUsername()의 역할

loadUserByUsername()은 Username이라는 매개인자를 사용해 저장소로 부터 해당 username을 지닌 user가 존재하는지 아닌지를 판단한다.   
존재하면 해당 유저의 세부 정보를 가져오고 이 세부 정보를 UserDetails라고 한다.   
UserDetails는 다양하게 커스텀할 수 있다 (상속)   

즉 loadUserByUsername이 잘 작동하기 위해서는 Repository가 필요! (DB에서 조회하기 위함)   
따라서 생성자 주입을 통해 CustomerRepostiory를 주입해주자.   


```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    // 생성자 주입
    CustomerRepository customerRepository;

    CustomUserDetailsService(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return null;
    }
}
```

## loadUserByUsername()이 비지니스 로직 작성

DB로부터 조회할 수 있도록 CustomerRepository도 DI를 해줬다.   

1. 실제 유저가 있는 지 조회한다.
2. 실제로 있으면 조회 정보를 통해 UserDetails를 만들어 반환한다.
   - SpringSecurity는 UserDetails를 상속받아 구현한 User라는 클래스가 있다.   
   - 하지만 나는 CustomUserDetails를 만들어 진행하겠다.

### CustomUserDetails 만들기

```java
public class CustomUserDetails implements UserDetails {

    private final Customer customerEntity;

    CustomUserDetails(Customer customerEntity) {
        this.customerEntity = customerEntity;
    }

    @Override
    public boolean isAccountNonExpired() {
        return UserDetails.super.isAccountNonExpired(); //ture
    }

    @Override
    public boolean isAccountNonLocked() {
        return UserDetails.super.isAccountNonLocked(); //ture
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return UserDetails.super.isCredentialsNonExpired(); //ture
    }

    @Override
    public boolean isEnabled() {
        return UserDetails.super.isEnabled(); //ture
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return customerEntity.getPwd();
    }

    // Customer 엔티티는 Username을 사용하지 X
    @Override
    public String getUsername() {
        return null;
    }

    public Customer getCustomerEntity() {
        return customerEntity;
    }
}


```

이제 실제 비지니스 로직을 작성해보자!

### CustomUserDetailsService 비지니스 로직 작성

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    // 생성자 주입
    CustomerRepository customerRepository;

    CustomUserDetailsService(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        Customer customerEntity = customerRepository.findByEmail(username);

        if (customerEntity != null) { // 조회한 값이 있으면
            // 해당 정보를 CustomUserDetails로 만들고 반환
            return new CustomUserDetails(customerEntity);
        }

        // 조회값 없으면 nulll
        return null;
    }
}

```

## 실행

이렇게 하고 실행 하면??

안된다!

![image](https://github.com/user-attachments/assets/2155a828-fb9c-4951-922c-b7b3c9438bb1)

Spring Security에서 비밀번호를 저장하거나 인증할 때 반드시 PasswordEncoder를 사용해야 하기 때문이다.

다음 장에 해결해보자!   



<hr>

# 새로운 유저 등록하는 새 REST API 만들기

http://localhost:8080/register   
위 경로로 새 Customer 정보를 JSON에 담아서 POST요청하면 새 유저를 등록하는 REST API를 추가한다.

## LoginController 코드

```java
@Controller
public class LoginController {

    CustomerRepository customerRepository;

    LoginController(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    @PostMapping("/register")
    public ResponseEntity<String> registeruser(@RequestBody Customer customer) {
        Customer savedCustomer = null;
        ResponseEntity response = null;
        try {
            savedCustomer = customerRepository.save(customer);
            if (savedCustomer.getId() > 0) {
                response = ResponseEntity
                        .status(HttpStatus.CREATED)
                        .body("Given user details are successfully registerd");
            }

        } catch (Exception e) {
            response = ResponseEntity
                    .status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("An exception occured due to" + e.getMessage());
        }
        return response;
    }

}
```

## 주의점

1. /register 경로를 permitALl 해야한다.
2. Id가 비어있을 때, DB에서 값을 알아서 채우도록 위임하고자 @GeneratedValue(strategy = GenerationType.IDENTITY)를 사용한다.   
   이 때, 해당 DB(나는 MySQL)에 AUTO INCREAMENT 설정을 해줘야한다.
3. **중요** CSRF를 비활성화 해줘야한다.

따라서 위 주의점을 고려했을 때 ProjectSecurityConfig의 코드는 아래와 같다.   
```java
@Configuration // Spring이 이 Class를 Bean으로서 스캔할 수 있도록 어노테이션 설정
public class ProjectSecurityConfig {

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(auth -> auth.disable()); //csrf 비활성화
        http.authorizeHttpRequests((requests) -> {
            requests.requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards")
                    .authenticated()
                    .requestMatchers("/notices", "/contact", "/register").permitAll(); // "/register" 추가
        });
        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());
        return (SecurityFilterChain) http.build();
    }
}
```