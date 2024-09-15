# Spring Security flow 소개

## Flow 그림

![image](https://github.com/user-attachments/assets/ea4ff906-111c-490f-b8ad-c46c6fa4007c)

1. 사용자 자격 증명 입력   
사용자의 자격 증명과 함께 요청을 백엔드 Application에 전송   


2. 유저가 이미 로그인했는 지, 아닌 지 확인   


3. Spring Security 필터가 사용자가 보낸 username과 password를 추출하여 인증 객체로 변환   


4. 인증 객체가 형성되면, Spring Security 필터가 Authentication Manager에게 인증 객체를 넘긴다.      
실질적인 인증 로직을 관리하는 인터페이스 or 클래스이다.   


5. Authentication Manager는 어플리케이션 안에 어떤 Authentication Providers가 존재하는 지 확인한다.      
Authentication Providers 내부에 실제 인증 로직을 정의한다.   


6. UserDetails Manager와 UserDetailsService는 DB같은 저장소로 부터 유저 정보를 불러와 인증 객체와 비교한다.


- 비밀번호 그대로 사용하면 보안에 취약하므로 이를 인코딩와 디코딩하는데 도와주는 PasswordEndcoder를 사용한다.


7. (9단계) 인증 객체를 보안 컨텍스트에 저장 - 인증 성공 여부와 세션 ID를 가지며 저장된다.


![image](https://github.com/user-attachments/assets/1dc4367a-df23-4199-a6ee-30105b769212)