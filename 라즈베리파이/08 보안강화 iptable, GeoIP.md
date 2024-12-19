
# 공유기 이슈

iptime 공유기는 해외 IP 차단을 할 수 있는데 내가 사용하는 공유기는 해외에서 주로 사용하는 IP 대역을 차단할 수 없고 일일히 IP 대역에 대해서 필터링 조건을 추가해야한다.

주로 중국이나 러시아에서 해킹 시도를 많이 한다는데….

# iptable, GeoIP

어떻게 할까 하다가 iptable, GeoIP를 알았다.

- **iptables**는 리눅스 커널에 내장된 방화벽 기능으로, 네트워크 패킷을 필터링하거나 제어하는 도구이다.
- **GeoIP**는 IP 주소를 기반으로 해당 IP의 지리적 위치(국가, 도시 등)를 매핑하는 데이터베이스 및 소프트웨어이다.

# 동작 과정

1. 패킷 수신
    - 네트워크를 통해서 패킷이 라즈베리파이로 들어온다.
2. iptables 규칙을 확인한다.
    - iptable에 설정된 규칙을 확인한다.
    - GeoIP 모듈을 같이 사용하는 경우 패킷의 출발지 IP를 GeoIP데이터 베이스와 비교
3. 국가 코드 매칭
    - GeoIP DB를 통해서 어느 국가에서 온 패킷인지 확인할 수 있는데 이때 출발지 IP가 한국(KR)인 패킷을 허용하도록 한다.

# 설치 과정

### 패키지 설치

```bash
sudo apt update

sudo apt-get install geoip-bin geoip-database

```

### Shell 스크립트 작성

/usr/local/bin/sshfilter.sh 여기에 ssh 필터역할을 할 스크립트를 작성한다

```bash
sudo vi /usr/local/bin/sshfilter.sh
```

내용은 다음과 같다.

ALLOW_COUNTRIES="KR"가 보이는가? 아래 스크립트를 간단히 말하면

- 받은 패킷의 출발지 ip를 geoiplookup으로 어딘지 찾아내고 해당 지역이 한국(KR)가 아니면 허용하지 않는 것이다.
- 맨 아래 if then else fi 는 로그를 찍는 부분이다.

```bash
#!/bin/bash
 
# UPPERCASE space-separated country codes to ACCEPT
ALLOW_COUNTRIES="KR"
 
if [ $# -ne 1 ]; then
  echo "Usage:  `basename $0` <ip>" 1>&2
  exit 0 # return true in case of config issue
fi
 
COUNTRY=`/usr/bin/geoiplookup $1 | awk -F ": " '{ print $2 }' | awk -F "," '{ print $1 }' | head -n 1`
 
[[ $COUNTRY = "IP Address not found" || $ALLOW_COUNTRIES =~ $COUNTRY ]] && RESPONSE="ALLOW" || RESPONSE="DENY"
 
if [ $RESPONSE = "ALLOW" ]
then
  exit 0
else
  logger "$RESPONSE sshd connection from $1 ($COUNTRY)"
  exit 1
fi
```

### 작성한 스크립트 권한 부여

```bash
#sudo chown root.root /usr/local/bin/sshfilter.sh
sudo chown root:root /usr/local/bin/sshfilter.sh
sudo chmod 775 /usr/local/bin/sshfilter.sh
```

### **SSH 잠금 설정**

두가지 파일에 한줄 씩 추가한다.

```bash
sudo vi /etc/hosts.deny # 파일 편집
# 아래 문구를 추가한다.
sshd: ALL
```

```bash
sudo vi /etc/hosts.allow 
# 아래 문구를 추가한다.
sshd: ALL: aclexec /usr/local/bin/sshfilter.sh %a

```

### 테스트 해보기

테스트 방법

/usr/local/bin/sshfilter.sh 8.8.8.8 와 같이 명령어를 입력하면 sshfilter.sh에 어떻게 작동하는지 결과가  journalctl -f 의 화면에 나온다.
여기서 8.8.8.8은 미국 대역 IP이다.

1. Putty로 라즈베리파이 추가로 접속하기

Putty로 라즈베리파이에 접속 후 다음 명령어를 입력한다.

```bash
journalctl -f 
```

2. Putty가 아닌 다른 SSH툴 (나는 Termius)에서 다음 명령어를 입력한다.

```bash
/usr/local/bin/sshfilter.sh 8.8.8.8 # 미국 (US)
/usr/local/bin/sshfilter.sh 2.2.2.2 # 프랑스(FR)
/usr/local/bin/sshfilter.sh 211.239.124.250 # 대한민국(KR)
```

3. 결과 확인하기
![image](https://github.com/user-attachments/assets/db38c274-2fb1-459b-b41b-e804a6f90206)


### 문제

iptables 방화벽의 문제는 다음과 같다.

1. GeoIP 데이터베이스는 주기적으로 업데이트되어야 한다.
2. `iptables`를 통한 차단은 모든 트래픽을 대상으로 하므로 성능에 영향을 미칠 수 있다. 특히 대량 트래픽에 대한 성능 저하가 있을것이다.
3. `iptables` 규칙은 서버 재부팅 시 사라지므로, 규칙을 영구적으로 저장하려면 규칙을 `/etc/iptables/rules.v4`에 저장하거나, `iptables-save`와 `iptables-restore`를 사용하여 규칙을 저장하고 복원하는 방법을 사용해야 한다.