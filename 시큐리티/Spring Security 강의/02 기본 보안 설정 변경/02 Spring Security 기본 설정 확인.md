# Spring Security 프레임워크 기본 설정 확인

- shift 두번 -> classes -> SpringBootWebSecurityConfiguration.java 파일 클릭   

해당 파일을 알아보자.   

## SpringBootWebSecurityConfiguration.java 코드 살펴보기

<details>
<summary>SpringBootWebSecurityConfiguration.java 코드</summary>

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.boot.autoconfigure.security.servlet;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication.Type;
import org.springframework.boot.autoconfigure.security.ConditionalOnDefaultWebSecurity;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AuthorizeHttpRequestsConfigurer;
import org.springframework.security.web.SecurityFilterChain;

@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
class SpringBootWebSecurityConfiguration {
    SpringBootWebSecurityConfiguration() {
    }

    @Configuration(
        proxyBeanMethods = false
    )
    @ConditionalOnMissingBean(
        name = {"springSecurityFilterChain"}
    )
    @ConditionalOnClass({EnableWebSecurity.class})
    @EnableWebSecurity
    static class WebSecurityEnablerConfiguration {
        WebSecurityEnablerConfiguration() {
        }
    }

    @Configuration(
        proxyBeanMethods = false
    )
    @ConditionalOnDefaultWebSecurity
    static class SecurityFilterChainConfiguration {
        SecurityFilterChainConfiguration() {
        }

        @Bean
        @Order(2147483642)
        SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
            http.authorizeHttpRequests((requests) -> {
                ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
            });
            http.formLogin(Customizer.withDefaults());
            http.httpBasic(Customizer.withDefaults());
            return (SecurityFilterChain)http.build();
        }
    }
}

```

</details>

## defaultSecurityFilterChain 알아보기

해당 메서드는 매개인자로 HttpSecurity Http를 받아들인다.

```java
    @ConditionalOnDefaultWebSecurity
    static class SecurityFilterChainConfiguration {
        SecurityFilterChainConfiguration() {
        }

        @Bean
        @Order(2147483642)
        SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
            http.authorizeHttpRequests((requests) -> {
                ((AuthorizeHttpRequestsConfigurer.AuthorizedUrl)requests.anyRequest()).authenticated();
            });
            http.formLogin(Customizer.withDefaults());
            http.httpBasic(Customizer.withDefaults());
            return (SecurityFilterChain)http.build();
        }
    }
```

#### authorizeHttpRequests
HttpSecurity를 사용해 authorizeHttpRequests를 호출한다.   
이 메서드를 통해 나의 웹 어플리케이션에 들어온 모든 요청은 증명되어야 한다는 것을 의미한다.   
(추후 커스텀하자!)   

#### formLogin vs httpBasic

위 두 방식도 결국 HttpSecurity의 도움을 받아서 호출한다. (결국 http 변수를 사용해 호출한다.)   

**formLogin 인증 방식**: 서버에 해당 사용자의 session 상태가 유효한지를 판단해서 처리하는 인증 방식입니다.   
**httpBasic 인증 방식**: Http 프로토콜에서 정의한 기본 인증 방식입니다.   

#### 반환 타입 SecurityFilterChain

defaultSecurityFilterChain 메서드의 반환타입은 SecurityFilterChain이다.   
여기서는 기본 HttpSecurity http를 (SecurityFilterChain으로 형변환 한 후 반환한다.         
즉, 스프링 기본 SecurityFilterChain을 반환하는 것이다.   


그러므로 우리가 SecurityFilterChain을 커스텀하여 지정한다면, 일반적인 Security 기본 설정 개입은 중단된다.   

**따라서 SecurityFilterChain을 커스텀 하여 우리 입맛에 맞게 요청에 대한 보안 로직을 필터링할 수 있다.**


## 다음 시간

SecurityFilterChain을 Bean에 등록하는 방법을 알아보자.

