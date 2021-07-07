
## AWS Client 환경설정 (aws,EKS,ECR)
#### AWS 관리콘솔 IAM에서 자격증명 생성
AWS 관리콘솔 접속
IAM 서비스의 ‘사용자’ 메뉴 접속
사용자의 Access Key와 Secret Access Key 복사
#### Cloud IDE - AWS Client Config
```
aws configure 
Input Access Key 
Input Secret Access Key
Input your Region Code 
Input 'json' in Output format 
```
#### EKS (Elastic Kubernetes Service) 생성
```
eksctl create cluster --name (Cluster-Name) --version 1.17 --nodegroup-name standard-workers --node-type t3.medium --nodes 2 --nodes-min 1 --nodes-max 2
```
#### K8s Client 에 target EKS Context 설정
```
aws eks --region (Region-Code) update-kubeconfig --name (Cluster-Name)
kubectl get all
kubectl config current-context
```
#### AWS ECR Login 설정
```
docker login --username AWS -p $(aws ecr get-login-password --region (Region-Code)) (Account-Id).dkr.ecr.(Region-Code).amazonaws.com/
```
#### AWS ECR에 Image Repository 생성
```
aws ecr create-repository --repository-name (Image-Repository-Name) --image
```
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
```
    docker build -t apexacme/welcome:v1 .
    docker image ls
```    
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


# Container Orchestration 무작정 따라하기
#### 주문서비스 생성하기
도커 허브에 저장된 주문 이미지으로 서비스 배포 및 확인하기
```
kubectl create deploy order --image=apexacme/order
kubectl get all
```
Docker Hub에 Push한 나만의 이미지로 쿠버네티스에 배포해 보기
```
- kubectl get all : 생성된 객체(Pod, Deployment, ReplicaSet) 확인
- kubectl get deploy -o wide : 배포에 사용된 이미지 확인
- kubectl get pod -o wide : 파드가 생성된 워크노드 확인
```
kubectl get pod 에서 Pod의 상태(STATUS)가 Running 이 아닌 경우
```
  - Trouble Shooting #1 : kubectl describe [Pod 객체]
  - Trouble Shooting #2 : kubectl logs -f [Pod 객체]
  - Trouble Shooting #1 : kubectl exec -it [Pod 객체] -- /bin/sh 
```
#### 주문서비스 삭제해 보기
```
kubectl get pod
kubectl delete pod [order Pod 객체] 
kubectl get pod
```
Pod를 삭제해도 새로운 Pod로 서비스가 재생성됨을 확인
#### 클라우드 외부에서도 접근 가능하도록 노출하기
```
kubectl expose deploy order --type=LoadBalancer --port=8080
kubectl get service -w
```
Service 정보의 External IP가 Pending 상태에서 IP정보로 변경시까지 대기하기
엔드포인트를 통해 서비스 확인 - http://(IP정보):8080/orders
Ctrl + C를 눌러 모니터링 모드 종료하기
#### 주문서비스 업데이트(v2) 하기
```
kubectl get deploy -o wide
kubectl set image deploy order order=apexacme/order:v2
kubectl get deploy -o wide
```
주문서비스에 적용된 Image가 apexacme/order에서 apexacme/order:v2로 업데이트 되었음을 확인
#### 주문서비스 롤백(RollBack) 하기
```
kubectl rollout undo deploy order
kubectl get deploy -o wide
```
주문서비스에 적용된 Image가 apexacme/order로 롤백되었음을 확인
#### 주문서비스 인스턴스 확장(Scale-Out) 하기 (수동)
```
kubectl scale deploy order --replicas=3
kubectl get pod
```
주문서비스의 인스턴스(Pod)가 3개로 확장됨을 확인
#### YAML 기반 서비스 배포하기
```Cloud IDE 메뉴 > File > Folder > YAML 입력

```생성한 폴더 하위에 아래 파일 생성

Cloud IDE 메뉴 > File > New File > order.yaml 입력
```
아래 내용 복사하여 붙여넣기
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-by-yaml
  labels:
    app: order
spec:
  replicas: 2
  selector:
    matchLabels:
      app: order
  template:
    metadata:
      labels:
        app: order
    spec:
      containers:
        - name: order
          image: apexacme/order:latest
          ports:
            - containerPort: 8080   
``` 
입력 후, 저장
```
- kubectl apply -f order.yaml 
- kubectl get all 
```

# CodeBuild를 활용한 파이프라인
#### CodeBuild에 사용될 환경변수
AWS_ACCOUNT_ID
KUBE_URL
KUBE_TOKEN
#### CodeBuild와 ECR 연결정책 JSON
```
{
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetAuthorizationToken",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "*",
      "Effect": "Allow"
 }
 ```
#### CodeBuild 에서 EKS 연결
Service Account 생성
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
EOF
```
ClusterRoleBinding 생성
```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
EOF
```
SA로 EKS 접속 토큰 가져오기
```
kubectl -n kube-system describe secret eks-admin
# token 결과값만 공백없이 복사
```
