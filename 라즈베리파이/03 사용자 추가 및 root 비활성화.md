# 사용자 추가 

사용자 추가
```bash
sudo adduser [사용자명]
```

사용자에게 sudo, adm 그룹 추가
```bash
sudo adduser [사용자명] sudo
sudo adduser [사용자명] adm
```

사용자명 pi의 비밀번호가 잠기게 된다.(비활성화)
```bash
sudo passwd -l pi
```