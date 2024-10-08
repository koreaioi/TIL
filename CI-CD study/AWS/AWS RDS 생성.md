# AWS RDS 생성

RDS는 서브넷 그룹을 선택해야 하므로
서브넷 그룹을 미리 만들고 시작하겠다!

RDS → 서브넷 그룹 → DB 서브넷 그룹 생성 클릭

- 기본 서브넷 4개는 각각 다른 가용 영역에 할당 되어있다.
- 가용 영역 ap-northeast-2c는 172.31.32.0/20
가용 영역 ap-northeast-2d는 172.31.48.0/20 의 서브넷을 가진다.
- 가용 영역이 다른 두 서브넷을 사용해서 서브넷 그룹(database-subnet-group)을 생성한다.
- 근데 프리티어는 단일 AZ(가용 영역) DB 인스턴스 이므로 RDS가 이 서브넷 그룹을 선택하면 하나의 가용 영역을 선택해야 한다. 
→ 따라서 서브넷 그룹을 선택해도 가용 영역에 따른 서브넷 하나에 속하게 된다. (지피티 피셜)
→ 이럴 거면 왜 서브넷 그룹을 만드는 거지? 프리티어 무시하나

![image](https://github.com/user-attachments/assets/c4e5b71f-e5d0-46fe-91fe-486df2e8e986)
### 생성 완료

![image 1](https://github.com/user-attachments/assets/2011b8c0-ca6b-4d92-9981-ab68386af368)

---

# RDS 생성하기

### DB 생성

![image 2](https://github.com/user-attachments/assets/0d11e358-389b-4cf8-8167-93ef8734f304)

### 엔진 옵션

- MySQL 선택
- 엔진 버전 - 가장 최신 선택

![image 3](https://github.com/user-attachments/assets/7df4e2c6-e020-46ea-9c6f-ef22d3888093)

### 템플릿 선택

- 프리티어 선택

![image 4](https://github.com/user-attachments/assets/d5677fe3-0b24-4679-b161-f42a261cb9be)

### 설정

- RDB 인스턴스 이름: database-1
- 마스터 사용자: admin
- 자격 증명 관리(비밀번호): 자체 관리
    - 비밀번호 89108910 으로 설정

![image 5](https://github.com/user-attachments/assets/2e24103d-9563-4b97-895a-400f463098dd)

### 인스턴스 구성

- 인스턴스 t3.micro 선택

![image 6](https://github.com/user-attachments/assets/598d89d2-64b3-41a1-88e9-6dce4bf2a1c9)

### 스토리지

- 과금 안되게 스토리지 20
- gp3 사용
- **스토리지 자동 조정 해제 (과금 요소)**

![image 7](https://github.com/user-attachments/assets/f788a793-9fe2-4bfd-bf53-8088f080084c)

### 연결

- 자동 연결 말고 수동으로 직접 연결
- 기본 VPC 사용 (EC2와 같은 VPC)
- DB 서브넷 그룹 (맨 위 참고)
- 퍼블릿 액세스: 아니요
    - Yes: RDS 인스턴스에 퍼블릭 IP 주소가 할당되며, 인터넷에서 직접 접근 가능
    - No: RDS 인스턴스는 프라이빗 IP 주소만 가진다. VPC 내의 EC2 인스턴스와 같은 리소스를 통해서만 접근 가능
- VPC 보안 그룹 새로 생성
- 가용 영역: 프리티어 RDB는 단일 AZ RDB 인스턴스 이므로 서브넷 그룹의 가용 영역 中 하나를 선택해야 한다. (걍 아무거나 선택 ㅋ)
    - 나는 2d 가용 영역 선택 → 이러면 서브넷은 172.31.48.0/20

![image 8](https://github.com/user-attachments/assets/a9022f85-45ba-41da-ae52-31ac809f9ada)

### 데이터베이스 인증

- 암호 인증 선택

![image 9](https://github.com/user-attachments/assets/b12cf305-bbe0-4d65-a980-18286867b951)

### 추가 구성

- 초기 DB이름: 귀찮아서 안함
- 백업 활성화 X
- 마이너 버전 업그레이드 X

## 데이터베이스 RDB 생성 확인해보기

![image 10](https://github.com/user-attachments/assets/ddbd3736-4052-4a33-af76-921d3c248b96)

### 의문 점

**서브넷이 왜 2개로 보이지?**

단일 AZ DB 인스턴스를 생성할 때, RDS는 사용자가 선택한 가용 영역에 따라 하나의 서브넷에 인스턴스를 배치한다.
하지만!, RDS 연결 및 보안 정보창에서 서브넷 그룹에 속한 모든 서브넷은 여전히 보인다고 한다.

[관련 문서](https://repost.aws/ko/knowledge-center/rds-launch-in-vpc)

![image 11](https://github.com/user-attachments/assets/4e36088c-68ac-4903-a248-b7d954a28167)

위 AWS 문서 설명에 DB 인스턴스를 시작할 서브넷’만’ 포함하라는 말이 있다. 
이는 해당 서브넷에만 속한다고 생각해도 좋지 않을까?