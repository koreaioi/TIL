# 7/11 Refresh + 쿠키 오류 해결

# 1. 기존 방식

기존에 refresh 토큰을 쿠키에 심어서 프론트에 보내준다.

```jsx
response.addCookie(jwtUtil.createCookie("refresh", refresh));
```

```jsx
public Cookie createCookie(String key, String value) {

    Cookie cookie = new Cookie(key, value);
    return cookie;
}
```

### 1 - 1. 기존 방식 문제점

1. 프론트 크롬 브라우저에 있는 쿠키를 꺼낼 수 없다.
2. refresh를 사용하는 reissue, logout 과정 중 백엔드에서 request.getCookies를 사용시 반환값이 null이다.

# 2. 원인

1. 프론트 상황
프론트는 다른 사용자 노트북에서 요청을 보낸다. (해당 노트북의 IP)
2. 백엔드 상황
백엔드 서버는 AWS EC2에 두고있으며, 도메인을 사용 중이다.

프론트와 백엔드가 분리되어 같은 URL을 사용중이지 않아서 쿠키가 제대로 전달되지 않음

# 3. 해결

쿠키를 response에 심을 때 sameSite: None 설정을 추가해야한다.

하지만 기존의 코드에서 Cookie 객체를 만들어서 해당 쿠키를 reponse에 addCookie메서드를 사용해 추가한다. 하지만 Cookie 객체를 만들 때 sameSite 설정을 할 수가 없다.

**따라서 sameSite설정을 할 수 있는 response.setHeader를 이용해서 쿠키를 추가하도록 한다.**

```java
String expires = new Date(System.currentTimeMillis() + 1000L * 60 * 60 * 24).toString(); // 1일 후 만료
response.setHeader("Set-Cookie",
        "refresh=" + refresh + "; Path=/; HttpOnly; SameSite=None; Secure; expires=" + expires);
```

setHeader메서드에 위와 같이 refresh 값을 추가하고 그 뒤는 추가 설정값이다.   
HttpOnly를 설정하면(true) 프론트단(JavaScript)에서 쿠키를 사용할 수 없다.   
SameSite=None을 설정하면 같은 SameSite가 아니어도 쿠키를 사용할 수 있다.   
Secure는 HTTPS 환경이므로 추가한다.   
expires는 만료기간을 의미하는데 expires가 없으면 해당 쿠키는 세션방식으로 동작한다.   
위 방식으로 response를 쿠키에 심어서 자유롭게 사용할 수 있도록 했다.   
