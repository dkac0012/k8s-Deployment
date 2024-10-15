# k8s-Deployment

**컨테이너 운영자동화 오케스트레이션 도구인 k8s를 사용하였습니다.**
**기업에서 자주 사용되는 기술을 배워보고자 하였습니다.**

### 단계
1. 간단한 jar 파일을 생성하였습니다.
2. docker build를 통해 경량화된 images로 배포하였습니다.
3. 배포된 image를 사용하도록 kube.yaml을 작성하였습니다.
4. service 기능을 활용하여 외부 포트와 연결하였습니다.

### 실습

#### step1&2 

dockerfile 작성
```bash
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY step18_empApp-0.0.1-SNAPSHOT.jar /app/app.jar

EXPOSE 80

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

docker build
``` bash
docker build -t dkac0012/springapp:v1
```

docker push
``` bash
docker push dkac0012/springapp:v1
```

image check
```bash
username@awsclient:~$ docker images
REPOSITORY           TAG           IMAGE ID       CREATED          SIZE
dkac0012/springapp   v1            b46b17b1661d   34 minutes ago   463MB
```

#### step3&4

deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      labels:
        app: springapp
    spec:
      containers:
      - name: springapp
        image: dkac0012/springapp:v1
        ports:
        - containerPort: 80
```

service.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: springapp-service
spec:
  type: LoadBalancer # 외부 IP 할당
  selector:
    app: springapp
  ports:
    - protocol: TCP
      port: 80       
      targetPort: 80
```

외부 ip 생성
```bash
username@awsclient:~$ kubectl get service
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP                                                                   PORT(S)        AGE
kubernetes          ClusterIP      10.100.0.1      <none>                                                                        443/TCP        84m
springapp-service   LoadBalancer   10.100.31.170   ***************************************************************************   80:30529/TCP   102s
```

접속 확인
```bash
username@awsclient:~$ curl http://<EXTERNAL-IP>:80
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
-- 생략 --
```
