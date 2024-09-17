# CustomAuthenticationProvider

Spring Security가 기본으로 제공하는 AuthenticationProvider인   
DaoAuthenticationProvider안에 있는 기본 인증 논리는 많은 작업들을 수행한다.   

하지만 특별한 요구사항 ex) 18세 이상의 나이를 가진 사용자들만 접근 허용, 허용 국가 목록   
- 위와 같은 맞춤 요구사항을 가지고 싶을 때
- 나만의 Custom 인증 로직을 가지고 싶을 때


이런 경우는 나만의 Custom Authentication Provider를 가져야한다.

## AuthenticationProvider Interface

```java
public interface AuthenticationProvider {
    
    // 인증 비지니스 로직은 authenticate 메서드에서 진행
    
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    boolean supports(Class<?> authentication);
}

```

## supports
supports 메소드를 override하여 provider의 동작 여부를 결정할 수 있다.   

예를 들어 AuthenticationProvider를 상속한 AbstractUserDetailsAuthenticationProvider 추상 클래스는 다음과 같은 supports를 구현한다.   
```java
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
```
위 말의 뜻은 여러 AuthenticationProviders 목록들 중에서   
AbstractUserDetailsAuthenticationProvider는 UsernamePasswordAuthenticationToken로된 인증 객체에 대해서만 인증 로직을 수행한다.   

**여기서 추론할 수 있는 한가지**    

- DaoAuthenticaitonProvider가 AbstractUserDetailsAuthenticationProvider를 구현하는거 같은데?   

- UsernamePasswordAuthenticationToken은 분명 Spring Security 기본 로그인 과정에서 사용자 요청으로 부터 받은 username과 password를 가지고 만들어지는 인증 객체이다.   
- Spring Security 기본 로그인 과정은 분명 DaoAuthenticationProvider를 통해 인증이 수행된다.   

그렇다면 DaoAuthenticationProvider에 supports 메서드가 정의되어 있을까? -> NO   
그렇다! DaoAuthenticationProvider는 AbstractUserDetailsAuthenticationProvider를 상속받는다!   

사진이다.   
![image](https://github.com/user-attachments/assets/899ab36a-ad09-4bc9-b16a-bbae95d5c06a)

위 과정의 이해를 통해 **"supports 메소드가 provider의 동작 여부를 결정한다."**라는 말의 뜻을 이해할 수 있었다.

## AuthenticationProvider를 상속한 여러 Provider들

만약 AuthenticationProvider를 상속한 여러 목록들 중 하나가 성공적인 인증을 제공할 수 없다면,   
사용 가능한 다른 AuthenticationProviders가 인증을 시도한다.   

![image](https://github.com/user-attachments/assets/7f6a083c-294c-4232-88d1-d4c0779098a6)

