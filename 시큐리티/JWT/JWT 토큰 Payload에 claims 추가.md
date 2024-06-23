# JWT 토큰 Payload에 claims 추가하기

JWT Access 토큰을 발급할 때 Payload에 username과 role뿐 만 아니라, 다른 claims도 추가하고자 한다.   

1. LoginFilter에서 attemptAuthentication()가 실행된다.   
2. 이 메서드 안에서 username과 password를 꺼내 authToken을 만든다.
3. authenticate(authToken)이 실행되고 이후 UserDetailsService를 구현한 구현체의 loadUserByUsername이 실행된다.
4. 별 이상 없으면 다시 LoginFilter의 successfulAuthentication가 실행된다.
5. JWT 토큰을 successfulAuthentication에서 만드므로 여기서 추가하고자 하는 claims를 JWTUtil에 넘겨주면된다.


## UserDetails에서 getId() 메서드 만들기
나는 user_id를 넘겨주고 싶다.   
그러므로 내가 커스텀한 UserDetails에서 getUsername(), getRole() 말고도 getId()를 만들어주자.

```java

public class CustomUserDetails  implements UserDetails {

    private final User userEntity;

    public CustomUserDetails(User userEntity) {

        this.userEntity = userEntity;
    }

    /* 생략 ... */

    @Override
    public String getPassword() {

        return userEntity.getPassword();
    }

    @Override
    public String getUsername() {

        return userEntity.getUsername();
    }

    public Long getId(){

        return userEntity.getId();
    }

    /* 생략 ... */
}


```

## successfulAuthentication에서 유저 정보 가져오기

```java

@Override
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication){
    
        //내가 커스텀한 UserDetails에서 user_id 꺼내오기
        CustomUserDetails customUserDetails=(CustomUserDetails)authentication.getPrincipal();
        Long userId=customUserDetails.getId();

        }

```

## JWTUtil의 createJwt에 넘겨주기
        String access = jwtUtil.createJwt(userId,"access", username, role, 600000L);

이러고 실행하면 잘 적용된다.

![image](https://github.com/koreaioi/TIL/assets/147616203/00cea65b-c829-498f-b019-d8253e8aeafb)
