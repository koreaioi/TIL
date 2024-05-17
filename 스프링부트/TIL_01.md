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
