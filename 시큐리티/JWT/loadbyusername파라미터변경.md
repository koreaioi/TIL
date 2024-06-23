# LoadbyUsername 파라미터 변경하기

우리가 커스텀한 LoginFilter가 상속하는 UsernamePasswordAuthenticationFilter에는   
setUsernameParameter가 있다.

```java
	/**
	 * Sets the parameter name which will be used to obtain the username from the login
	 * request.
	 * @param usernameParameter the parameter name. Defaults to "username".
	 */
	public void setUsernameParameter(String usernameParameter) {
		Assert.hasText(usernameParameter, "Username parameter must not be empty or null");
		this.usernameParameter = usernameParameter;
	}

```

LoginFiler객체를 생성하고 setUsernameParameter로 파라미터명을 커스텀하면 된다.

## 실제 코드

```java
    // 로그인은 email, password 이므로 loadbyUsername의 파라미터를 email로 변경한다.
    LoginFilter loginFilter = new LoginFilter(authenticationManager(authenticationConfiguration), jwtUtil, refreshRepository);
    loginFilter.setUsernameParameter("email");
    http
            .addFilterAt(loginFilter, UsernamePasswordAuthenticationFilter.class);

```

헷갈리지 않게 loadByusername 파라미터도 변경하자

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {

        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {

        User userData = userRepository.findByEmail(email);

        if (userData != null) {

            return new CustomUserDetails(userData);
        }

        return null;
    }
}

```
