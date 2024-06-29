# 백엔드와 프론트

1. 컨트롤러에서 form-data를 받을 때는 아무것도 안써도됨
2. json을 받을떄는 @RequestBody를 사용해야 json형식으로 받을 수 있다.

- LoginFilter에서 formdata와 json 형식 받아오기

연동 오류 첫번째   
JSON과 Form-Data가 같은 의미인줄 알고 무지성 코드 박아버림.   
알고보니, JSON과 Form-Data는 다른 방법으로 요청이 날라와서 이에 대한 처리가 필요했다.
JSON의 컨텐트 타입은 "application/json"이고,   
Form Data의 컨텐트 타입은 "multipart/form-data"

따라서 이에 대한 파싱이 필요하다!
   
프론트에서 보낼 때 userInfo를 보낸다고 치면   
1. client.post("url", userInfo,{}); 방식과   
2. client.post("url", {userInfo},{}); 방식은 다르다
전자는 {email="",password=""} 이렇게 날라오고   
후자는 {userInfo={email="",password=""}} 이렇게 한 번 더 씌워져서 날라온다. 주의하자.   

- 프론트에서 CORS 허용도 해줘야 쿠키에 넣은 refresh 토큰이 잘 들어간다.

```java

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        String email = null;
        String password = null;

//        request.getParameterMap().forEach((key, value) -> {
//            System.out.println("Key: " + key + ", Value: " + Arrays.toString(value));
//        });

        // JSON 요청 처리
        if (request.getContentType().equals("application/json")) {
            try {
                System.out.println("=====json ======");
                // JSON 요청을 Map으로 파싱
                Map<String, String> requestMap = objectMapper.readValue(request.getInputStream(), Map.class);
                System.out.println("requestMap :" + requestMap);
                // userInfo 키의 값을 추출하고 타입 캐스팅
//                Map<String, String> userInfo = (Map<String, String>) requestMap.get("userInfo");
//                System.out.println("userInfo :" + userInfo);
                // email과 password 값을 추출
                email = (String)requestMap.get("email");
                password = (String)requestMap.get("password");
                System.out.println(email);
                System.out.println(password);
            } catch (IOException e) {
                e.printStackTrace();
            }
        } else {
            // 요청이 form-data 형식인경우
            System.out.println("=====form-data======");
            email = obtainUsername(request);
            password = obtainPassword(request);
        }


        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(email, password);
        System.out.println("authToken 실행");
        return authenticationManager.authenticate(authToken);
    }

```

