# SSH 기본 포트 변경

라즈베리파이 OS 설치 시 SSH를 기본으로 허용했기 때문에 ssh를 설치할 필요는 없다.   

총 2가지 파일을 변경한다.

```bash
sudo vi /etc/ssh/sshd_config
```

해당 주석 해제 후 원하는 포트로 변경
![image](https://github.com/user-attachments/assets/97175c1f-e12c-4529-9d17-395a26ddbca3)


```bash
vi /etc/services
```

포트 번호인 22를 원하는 포트로 변경한다.
![image](https://github.com/user-attachments/assets/a24d9c07-2a70-4816-a361-ad601db6fe9a)

- 재부팅
```bash
sudo reboot 
```