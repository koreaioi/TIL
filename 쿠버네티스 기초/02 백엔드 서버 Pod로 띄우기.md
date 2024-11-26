# Spring Boot 프로젝트 셋팅

인텔리제이에서 Dockerfile작성하기

```Dockerfile
FROM openjdk:17-jdk

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

- 인텔리제이 터미널에서 ./gradlew clean build
- 빌드가 완료되면 build/libs 폴더 안에 jar 파일이 생성된다.
  - 이렇게 생성된 jar 파일을 가지고 Dockerfile을 활용해 Docker 이미지를 만든다.
- 인텔리제이 터미널에 docker build -t [만드려는 이미지명] . (온점 필수)
  - Dockerfile에 의해서 현재 빌드 파일이 Docker 이미지로 생성됨.

- DockerDesktop에서 확인 가능함.


# 쿠버네티스 yaml 작성하기

다음과 같이 spring-pod.yaml 파일을 작성한다.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod
  
spec: # pod의 상세 정보를 적는다.
  containers:
    - name: spring-container
      image: spring-server # 이미지명
      ports:
        - containerPort: 8080

```

다음 명령어를 실행해서 yaml로 pod를 실행한다.

```bash
# spring-pod.yaml을 실행하여 적힌 pod를 띄운다.
kubectl apply -f spring-pod.yaml

# 실행되고 있는 pod를 확인한다.
kubectl get pods
```

이러면 아마 pod가 제대로 실행되지 않는다.

<hr>

# 쿠버네티스 이미지 풀 정책

쿠버네티스에서 yaml를 실행할때 이미지는 기본적으로 레지스트리(도커 허브)에서 가져온다.
- 정확히는 이미지의 태그가 latest이거나 명시되어있지 않는 경우이다.
- 위 옵션을 Always라 한다.

나는 로컬 이미지를 가져와서 빌드하고 싶다면
- IfNotPresent 옵션을 적용해줘야한다.
- 이미지의 태그가 latest가 아닌 경우 IfNotPresent 옵션이 적용 후 도커 레지스트리에서 가져온다.

따라서 다음과 같이 ImagePullPolicy에 IfNotPresent 옵션을 추가해준다.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod
  
spec: # pod의 상세 정보를 적는다.
  containers:
    - name: spring-container
      image: spring-server # 이미지명
      ports:
        - containerPort: 8080
      imagePullPolicy: IfNotPresent # 로컬 이미지 사용시

```

다음과 같이 spring-pod.yaml를 수정 후 실행하면?

```bash
kubectl apply -f spring-pod.yaml

kubectl get pods

kubectl exec -it spring-pod -- bash

# Pod 내부로 접속 후
curl localhost:8080 # 요청을 보낸다.

```

<hr>

# 쿠버네티스 Pod로 띄운 프로그램 접속하기

![image](https://github.com/user-attachments/assets/078f13a5-ab23-46d6-aa17-1a2eb3962901)

파드(Pod) 네트워크는 로컬 컴퓨터의 네트워크와 독립적으로 분리되어 있다.   
때문에 파드와 로컬 컴퓨터간 포트포워딩을 하지 않으면 로컬 컴퓨터의 브라우저에서 파드에 접속할 수 없다.

## 파드 내부 프로그램 접근 방법
1. 파드 내부로 직접 들어가서 접근
2. 포트포워딩

### 파드 내부로 직접 들어가서 접근

```bash
# pod 내부로 접속
kubectl exec -it [pod명] -- bash

# pod 내부에 접속 후 아래 명령어 실행
curl localhost:8080

```

### 포트포워딩하기 (중요)

![image](https://github.com/user-attachments/assets/aa963f7f-9386-4a1a-bab9-3f00f216c3b2)

다음 명령어로 포트포워딩 할 수 있다.

```bash
kubectl port-forward pod/[포트포워딩할 pod명] 8080:8080

# 우리 같은 경우는 다음과 같이 명령어를 적는다.
kubectl port-forward pod/spring-pod 8080:8080
```

<hr>

# Pod 디버깅 하기
## 에러 메세지 확인하기
다음 명령어는 해당 Pod가 생성되고 난 후의 오류, 에러 메세지를 보여준다.

```bash
kubectl describe pods [pod명]
```

## 로그 확인하기
해당 Pod에서 발생한 로그를 확인할 수 있다.

```bash
kubectl logs [pod명]
```

## 내부 접속 후 디버깅
pod 내부 접속

```bash
kubectl exec -it [pod명] -- bash
```
