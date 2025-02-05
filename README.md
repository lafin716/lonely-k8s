# Lonely K8s

로컬 환경에서 Kubernetes 기반의 개발 인프라를 구성하기 위한 프로젝트입니다.

## 사전 요구사항

- [Minikube](https://minikube.sigs.k8s.io/docs/start/) v1.32.0 이상
- [Helm](https://helm.sh/docs/intro/install/) v3.10.0 이상
- [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.25.0 이상
- [Docker](https://docs.docker.com/get-docker/) 24.0.0 이상

## 구성 요소

- Docker Registry: 개인 컨테이너 이미지 저장소
- Jenkins: CI/CD 파이프라인
- MetalLB: 로드밸런서
- Microservices: Spring Boot + PostgreSQL

## 설치 방법

### 1. Minikube 시작
```bash
# Minikube 시작 (리소스 할당)
minikube start --driver=docker --cpus=4 --memory=8192mb

# Ingress 활성화
minikube addons enable ingress

# hosts 파일에 도메인 추가 (관리자 권한 필요)
echo "$(minikube ip) registry.local jenkins.local api.local" | sudo tee -a /etc/hosts
```

### 2. Helm Repository 추가
```bash
# Jenkins 저장소 추가
helm repo add jenkins https://charts.jenkins.io

# PostgreSQL 저장소 추가
helm repo add bitnami https://charts.bitnami.com/bitnami

# MetalLB 저장소 추가
helm repo add metallb https://metallb.github.io/metallb

# 저장소 업데이트
helm repo update
```

### 3. Helm 차트 설치
```bash
# 전체 서비스 설치
helm install lonely-k8s ./charts

# 개별 서비스 설치 (필요한 경우)
helm install registry ./charts/registry
helm install jenkins ./charts/jenkins
helm install metallb ./charts/metallb
helm install microservices ./charts/microservices
```

## 서비스 설정

### Docker Registry

기본 접속 정보:
- URL: http://registry.local
- NodePort URL: http://<노드IP>:30500
- 사용자: admin
- 비밀번호: admin123 (보안을 위해 변경 필요)

설정 변경:
```bash
# values.yaml 수정으로 설정 변경
helm upgrade registry ./charts/registry --set registry.auth.password=새로운비밀번호
```

### Jenkins

기본 접속 정보:
- URL: http://jenkins.local
- NodePort URL: http://<노드IP>:30800
- 사용자: admin
- 비밀번호: admin123 (보안을 위해 변경 필요)

주요 설치된 플러그인:
- Kubernetes
- Docker
- Pipeline
- Blue Ocean
- Git

### MetalLB

IP 범위 설정:
- 기본 범위: 192.168.49.200-192.168.49.240
- 설정 변경:
```bash
helm upgrade metallb ./charts/metallb --set metallb.configInline.address-pools[0].addresses[0]=새로운IP범위
```

### Microservices (Spring Boot + PostgreSQL)

Spring Boot 애플리케이션:
- URL: http://api.local
- 기본 프로필: prod
- Actuator 엔드포인트: /actuator/health

PostgreSQL 데이터베이스:
- 호스트: lonely-k8s-postgresql
- 포트: 5432
- 데이터베이스: myapp
- 사용자: postgres
- 비밀번호: postgres123 (보안을 위해 변경 필요)

## 볼륨 구성

모든 서비스는 영구 볼륨을 사용하여 데이터를 보존합니다:

### Registry
- 마운트 위치: /var/lib/registry
- 크기: 10Gi
- 접근 모드: ReadWriteOnce

### Jenkins
- 마운트 위치: /var/jenkins_home
- 크기: 10Gi
- 접근 모드: ReadWriteOnce

### PostgreSQL
- 마운트 위치: /bitnami/postgresql
- 크기: 8Gi
- 접근 모드: ReadWriteOnce

## 문제 해결

### 서비스 상태 확인
```bash
# 전체 리소스 상태 확인
kubectl get all

# NodePort 확인
kubectl get svc

# 개별 서비스 로그 확인
kubectl logs -f deployment/lonely-k8s-registry
kubectl logs -f deployment/lonely-k8s-jenkins
kubectl logs -f deployment/lonely-k8s-springboot
```

### 볼륨 상태 확인
```bash
# PVC 상태 확인
kubectl get pvc

# PV 상태 확인
kubectl get pv
```

### 인그레스 상태 확인
```bash
# 인그레스 상태 확인
kubectl get ingress
```

## 보안 고려사항

실제 운영 환경에서는 다음 사항을 반드시 변경하세요:

1. Registry 인증 정보
2. Jenkins 관리자 비밀번호
3. PostgreSQL 비밀번호
4. 모든 서비스의 인그레스에 HTTPS 적용
5. 리소스 제한 값 조정
6. NodePort 접근 제한 설정 (필요한 경우 방화벽 설정)

## 업그레이드 및 롤백

```bash
# 업그레이드
helm upgrade lonely-k8s ./charts

# 롤백
helm rollback lonely-k8s 1  # 이전 버전으로 롤백
```

## 제거

```bash
# 전체 서비스 제거
helm uninstall lonely-k8s

# 볼륨 데이터 제거 (선택사항)
kubectl delete pvc --all
``` 