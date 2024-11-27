# Deployment (디플로이먼트)

여러개의 백엔드 서버를 띄우기 위해서는 Deployment를 사용한다.

- Deployment: 파드를 묶음으로 쉽게 관리할 수 있는 기능
- 파드의 수를 지정하는 대로 쉽게 생성할 수 있다.
- 파드가 비정상적으로 종료된 경우, 알아서 새로 파드를 생성해 파드 수를 유지한다. (회복성)
- 동일한 구성의 여러 파드를 일괄적으로 일시 중지, 삭제, 업데이트 하기가 쉽다.


## Deployment의 구조

- Deployment가 레플리카(복제본)의 세트를 관리한다.
- 레플리카셋이 여러 파드를 관리하는 구조이다.

![image](https://github.com/user-attachments/assets/2ab68da8-8ed6-4faf-b2f7-fbcf73505ed2)


## Deployment.yaml 작성해서 여러 서버 띄우기

다음과 같이 spring-deployment.yaml 파일을 작성한다.
```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: spring-deployment
# Deployment 세부 정보
spec:
  replicas: 3  # 생성할 Pod의 복제본 수
  selector:
    matchLabels:
      app: backend-app # 연결할 pod의 label(카테고리)
  
  # 배포할 Pod 정의
  template:
    metadata:
      labels:
        app: backend-app # pod의 카테고리
    spec:
      containers:
        - name: spring-container # pod안에 띄울 도커 컨테이너 이름
          image: spring-server 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

다음 명령어로 spring-deployment.yaml을 실행후 확인하자.
```bash
# 실행
kubectl apply -f spring-deployment.yaml
# 확인
kubectl get deployment
# 레플리카 셋 확인
kubectl get replicaset 
# Pod 확인
kubectl get pods
```