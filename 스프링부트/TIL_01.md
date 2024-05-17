# Dto 매핑하기

```java
package study_pratice01.pratice01.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
//@NoArgsConstructor
@ToString
public class RequestDto {

    private String name;
    private int age;

    public RequestDto() {}

    public RequestDto(String name, int age) {
        this.name = name;
        this.age = age;

    }

}

```

위처럼 기본 생성자를 만들어 주거나 @NoArgsConstructor 어노테이션을 사용해주자.

# Dto - controller 매핑

```java
package study_pratice01.pratice01.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Getter
//@Setter
@NoArgsConstructor
@ToString
public class RequestDto {

    private String name;
    private int age;

}
```

```java
package study_pratice01.pratice01.controller;

@RestController
public class DtoTestController {

    @PostMapping("/dto-test")
    public Object requestTest(@RequestBody RequestDto requestDto) {
        System.out.println(requestDto.toString());
        return requestDto;
    }

}

```

위와 같이 @Setter가 없어도 매핑하는데는 문제가 없다.
![image](https://github.com/koreaioi/TIL/assets/147616203/b2258c92-cee6-4f9e-a5d9-43616c8d2686)

```java
package study_pratice01.pratice01.controller;
@RestController
public class DtoTestController {

    @PostMapping("/dto-test")
    public Object requestTest(RequestDto requestDto) {
        System.out.println(requestDto.toString());
        return requestDto;
    }

}

```

하지만 위와 같이 Controller에서 @RequestBody를 지우면 Dto의 필드가 default값으로 채워진다. -> 즉 매핑이 되지 않는다.
- @setter를 추가해도 똑같다. @RequestBody가 업으면 매핑이 제대로 되지 않음

# 정리
- @RequestBody 어노테이션 사용시 기본적으로 Jackson 라이브러리가 사용된다
- Jackson은 필드가 아닌 setXXX, getXXX 메소드를 찾아서 매핑한다
- 기본 생성자는 필수다


