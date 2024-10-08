# AWS EC2 생성하기

## 이름 생성, OS 선택

1. 이름: cicd-study-ec2
2. OS - ubuntu 선택

![image](https://github.com/user-attachments/assets/b76460c9-84f5-4985-8da2-a4aea1abc61a)
## 인스턴스 유형 선택

국룰 t2.micro선택

![image 1](https://github.com/user-attachments/assets/7fbab3c5-8041-4ce5-8258-bfdceef0bae3)
## 키 페어 설정

1. 새 키 페어 생성 클릭
2. 키페어 이름 입력
3. 키페어 유형 RSA
4. 키 파일 형식 .pem 
    - 잘 보관하고 있자.

![image 2](https://github.com/user-attachments/assets/0525f365-158e-45bf-84d1-d725c95d414c)
## 네트워크 설정

1. VPC는 기본 VPC로 설정한다.
2. 서브넷은 172.31.0.0/20 으로 첫번째 서브네팅된 기본 서브넷을 선택해보겠다. 

![image 3](https://github.com/user-attachments/assets/9e7c3b51-c478-4e3d-a374-56fad31172d1)

![tempFileForShare_20241008-230529](https://github.com/user-attachments/assets/b3a15694-5d2b-4d50-8e9d-9fc5269a009c)

아마 이런 식으로 
구조가 그려질 것이다. (예상)

### 퍼블릭 IP 자동할당

활성화로 시작

**활성화** 

- 인스턴스가 생성될 때 AWS가 자동으로 퍼블릭 IP 주소를 할당한다.
- 인스턴스 종료 시 IP 주소 소멸
- [750시간까지 무료](https://aws.amazon.com/ko/about-aws/whats-new/2024/02/aws-free-tier-750-hours-free-public-ipv4-addresses/)

**비활성화**

- 사용자가 수동으로 퍼블릭 IP를 할당한다.
- Elastic IP를 할당하여 고정 IP를 사용할 수 있습니다.

### 보안 그룹 설정 (보안 규칙)

- ssh 프로토콜 인바운드 규칙 22포트 오픈
    - ssh 프로토콜을 사용해 원격으로 ec2 서버에 접속하기 위함
    - 22번 포트를 오픈하지 않으면 EC2 인스턴스에 SSH를 통해 원격 접속할 수 없음
    - putty, termius와 같은 ssh 클라이언트로 접속 가능

![image 4](https://github.com/user-attachments/assets/15ee8509-bf32-42cf-a9fb-fff55a8dabf3)

### 스토리지 구성

- 과금을 막기 위해 gp2로 설정
- 30기가 설정

![image 5](https://github.com/user-attachments/assets/18fe5752-8176-49c3-8acf-a42043be145e)

### 고급 세부정보

건드릴 필요 없음

### 인스턴스 시작

- 인스턴스 ID 클릭  → 보안 → 인바운드 규칙

cicd-study-ec2의 보안 그룹은 security-group-1이다.
해당 보안 그룹은 ssh 프로토콜 22번 포트를 열어둔 상태 

![image 6](https://github.com/user-attachments/assets/d91b3eba-afcc-4016-a585-5caa683baeb2)
