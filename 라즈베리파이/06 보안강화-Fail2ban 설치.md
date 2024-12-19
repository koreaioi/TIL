# Fail2ban
fail2ban은 외부 침입자들로부터 계정 또는 비밀번호 시도 횟수 초과시 특정 시간동안 아예 접근을 못하도록 막아버리는 프로그램이다.


ssh를 사용하는 나의 라즈베리파이 서버에 로그인 시도하는 해커들을 방지하기위해 사용한다.   

### fail2ban 설치

```bash
sudo apt install fail2ban # 설치

sudo vi /etc/fail2ban/jail.conf # jail.conf 설정
```

![image](https://github.com/user-attachments/assets/ad08c20b-f0c9-4077-a1bf-f42afe611159)

bantime은 일주일 (1w)로 설정했다.

findtime은 maxretry 횟수를 시도하는 총 시간이다. 100분으로 설정했다.

### 재시작
```bash
systemctl retstart fail2ban # 재시작 
```