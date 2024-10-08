# AWS EC2 RDS 연결

<details>
<summary>terminus ec2 ssh 연결</summary>

![image](https://github.com/user-attachments/assets/24ede246-3b74-48b7-876c-6736b657c16d)

</details>

termius로 ssh 실행해서 ec2로 접속!
- termius 에서는 복사 붙여넣기가 ctrl + shift + c, v 이다.

### EC2에 MySQL 설치

```bash
sudo apt update
sudo apt install mysql-server
```

```bash
mysql -h [엔드포인트 주소] -u [마스터 사용자 이름] -p

mysql -h database-1.cjk0wmik6w46.ap-northeast-2.rds.amazonaws.com -u admin -p
```

### 실행 결과

비밀 번호를 잘 입력해도 들어가 지지 않는다. 에러 발생!

![image 1](https://github.com/user-attachments/assets/eaa4f098-93f8-4d9f-a150-4a036574863c)

### MySQL 접속 안되는 이유

RDS 보안 그룹에서 EC2 보안 그룹이 접속 할 수 있도록 인바운드 규칙 설정을 해줘야한다!!!

EC2 보안 그룹: security-group-1

RDS 보안 그룹:rds-security-group

RDS 보안 그룹을 보니 내 IP로 접속이 허용되어 있었다!

→ SSH로 EC2에 접속하여 RDS에 접근 하려니 당연히 안됨!

![image 2](https://github.com/user-attachments/assets/f1df7f08-2fd2-4062-812b-ecfe24a535ab)

### 해결

따라서 인바운드 규칙의 소스를 EC2 보안 그룹으로 설정하면 된다!

- **보안 그룹 인바운드 규칙 트러블 슈팅**
    
    인바운드 규칙에 보안 그룹 규칙 ID가 있는 경우 → 소스로 보안 그룹 자체를 선택할 수 없다.
    
    - 규칙 ID 기반 인바운드 규칙은 세분화된 보안 그룹간 트래픽 허용을 위해서 
    소스로 **다른 보안 그룹(security-group-1)**을 지정할 수 없음
    - 규칙 ID 기반 인바운드 규칙은 비교적 최근에 나온거임
    - **해결 방법:** 규칙 ID가 없는 인바운드 규칙을 사용하면 됨
    - ![image 3](https://github.com/user-attachments/assets/006500e6-b3eb-4601-b421-afa21f47a8a4)
    
- 새 인바운드 규칙을 생성한다.
- MySQL 선택
- RDS 보안 그룹 인바운드 규칙에서 소스를 **security-group-1**을 선택해준다. (EC2 보안 그룹)

새 인바운드 규칙(보안 그룹 규칙 ID가 없음)을 만들어서 security-group-1 선택 후 저장!

![image 4](https://github.com/user-attachments/assets/37d616de-230b-4ff7-a1c5-547b03db9097)

### 잘 되나 확인

RDS 보안 그룹 인바운드 규칙에

EC2 보안 그룹이 접속할 수 있도록 포트를 뚫어놨으니 잘 되는지 확인해보자!

잘 된다!

![image 5](https://github.com/user-attachments/assets/f3cb6e20-473f-40a2-9ad5-75ae67008780)

---

## AWS 내부 그림

라우터 등은 생략

![AWS%EA%B7%B8%EB%A6%BC2](https://github.com/user-attachments/assets/3a38cf78-a7fb-4f36-b0dc-2341ca026cb3)

---

## 아쉬운 점

기본 서브넷 4개는 퍼블릿 서브넷이다.
실제로 RDS가 속하는 서브넷은 private 서브넷으로 알고 있다.

[private 서브넷 만들기 참고](https://minjii-ya.tistory.com/33)