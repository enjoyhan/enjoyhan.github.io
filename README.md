
## AWS Client 환경설정 (aws,EKS,ECR)
#### AWS 관리콘솔 IAM에서 자격증명 생성
AWS 관리콘솔 접속
IAM 서비스의 ‘사용자’ 메뉴 접속
사용자의 Access Key와 Secret Access Key 복사
#### Cloud IDE - AWS Client Config
aws configure 
Input Access Key 
Input Secret Access Key
Input your Region Code 
Input 'json' in Output format 
#### EKS (Elastic Kubernetes Service) 생성

eksctl create cluster --name (Cluster-Name) --version 1.17 --nodegroup-name standard-workers --node-type t3.medium --nodes 2 --nodes-min 1 --nodes-max 2
#### K8s Client 에 target EKS Context 설정

aws eks --region (Region-Code) update-kubeconfig --name (Cluster-Name)
kubectl get all
kubectl config current-context
#### AWS ECR Login 설정

docker login --username AWS -p $(aws ecr get-login-password --region (Region-Code)) (Account-Id).dkr.ecr.(Region-Code).amazonaws.com/
#### AWS ECR에 Image Repository 생성

aws ecr create-repository --repository-name (Image-Repository-Name) --image

# 도커 이미지 무작정 따라하기
#### 이미지 기반 컨테이너 생성
```
docker image ls
docker run --name my-nginx -d -p 8080:80 nginx
docker run --name my-new-nginx -d -p 8081:80 nginx


docker image ls
docker container ls
```

#### 서비스 확인
Cloud IDE 메뉴 Labs > 포트열기 > 8080
Cloud IDE 메뉴 Labs > 포트열기 > 8081
#### 컨테이너와 이미지 삭제하기
삭제하려는 이미지를 사용하는 컨테이너 정리가 우선
```
docker container ls ; 실행중인 컨테이너 확인
docker container stop my-nginx
docker container stop my-new-nginx
docker container rm my-nginx
docker container rm my-new-nginx
docker image rm nginx
docker image ls
```
이미지 생성하고 Remote Registry(Hub.docker.com)에 푸시하기
어플리케이션 및 이미지 빌드 스크립트(Dockerfile) 생성
Cloud IDE 메뉴 > File > Folder > Docker 입력
생성한 폴더 하위에 아래 2개 파일 생성
Cloud IDE 메뉴 > File > New File > index.html 입력
파일 내용에
    Hi~ My name is Hong Gil-Dong...~~~
입력 후 저장
Cloud IDE 메뉴 > File > New File > Dockfile (확장자 없음)
파일 내용에
```
    FROM nginx
    COPY index.html /usr/share/nginx/html/
```
입력 후, 저장
이미지 빌드하기

    docker build -t apexacme/welcome:v1 .
    docker image ls
이미지 원격 저장소에 푸시하기

도커허브 계정 생성

https://hub.docker.com 접속

가입(Sign-Up) 및 E-Mail verification 
```
docker login 
docker push apexacme/welcome:v1
```

Docker Hub에 생성된 이미지 확인
https://hub.docker.com 접속
repositories 메뉴 Reload 후 Push된 이미지 확인
Docker Hub 이미지 기반 컨테이너 생성
```
docker image rm apexacme/welcome:v1
docker run --name=welcome -d -p 8080:80 apexacme/welcome:v1
```


