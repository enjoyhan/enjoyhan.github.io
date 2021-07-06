
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
