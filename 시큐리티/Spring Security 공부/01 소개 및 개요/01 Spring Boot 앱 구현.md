# 보안 없이 Spring Boot 앱 구현

## Start Spring io
![image](https://github.com/user-attachments/assets/374195f6-d48d-4fb5-9162-57c03042f8d3)

## 예시 컨트롤러 코드

### 코드

```java
package com.eazybytes.springsecuritybasic.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class WelcomeController {

    @GetMapping("/welcome")
    public String sayWelcome(){
        return "Welcome to Spring Application with out Security";
    }

}

```

### 실행 결과

![image](https://github.com/user-attachments/assets/48ed09e3-87f9-4c0d-b849-a091a866d99d)