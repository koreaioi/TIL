# 어플리케이션 REST API 소개

## 보안 없이 사용 가능한 서비스

- /contact 
- /notices

## 보안 서비스

- /myAccount
- /myBalance
- /myLoans
- /myCards

## 어플리케이션 환경 설정

springsecuritybasic을 springsecuritysection2로 이름 변경 후   
아래 그럼 처럼 폴더 구조 변경 후 이름도 적합하게 변경한다.   

SpringApplication 이름은 EazyBankBackendApplication으로 변경 후 해당 빌드로 변경

![image](https://github.com/user-attachments/assets/7d2864cf-8dc1-42b7-ab02-2de04a4bef1e)


<details>
<summary> 코드 추가 </summary>


# 코드 추가

## AccountController 코드

추후 DB로 부터 Account를 가져오게 할 컨트롤러

```java
package com.eazybytes.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AccountController {

    @GetMapping("/myAccount")
    public String getAccountDetails(){
        return "Here are the account details from the DB";
    }
}

```

## BalanceController

잔고 정보를 반환하는 컨트롤러

```java
package com.eazybytes.controller;


import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BalanceController {

    @GetMapping("/myBalance")
    public String getBalanceDetails(){
        return "Here are the balance details from the DB";
    }
}

```

## LoansController

```java
package com.eazybytes.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LoansController {

    @GetMapping("/myLoans")
    public String getBalanceDetails(){
        return "Here are the loan details from the DB";
    }

}

```


## CardsController

```java
package com.eazybytes.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CardsController {

    @GetMapping("/myCards")
    public String getBalanceDetails(){
        return "Here are the card details from the DB";
    }

}

```

## ContactController

문의 정보를 반환하는 컨트롤러

```java
package com.eazybytes.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ContactController {

    @GetMapping("/contact")
    public String getBalanceDetails(){
        return "Inquiry details are saved to the DB";
    }

}

```

## noticesController

```java
package com.eazybytes.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class NoticesController {

    @GetMapping("/notices")
    public String getBalanceDetails(){
        return "Here are the notices details from the DB";
    }

}

```

</details>

# 실행

이 상태로 어플리케이션을 실행하면   
설정한 모든 url에 security 로그인 화면이 뜬다!   

하지만 우리는 일부 url(/notice, /contact)과 같은 url은 로그인 없이 보여주고 싶다.!!   

몇몇 URL은 보호하고 일부 URL은 보호하지 않는 방법을 다음 시간에 천천히 알아보자!

