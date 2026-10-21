# Monolith Kubernetes 배포

현재 실습은 MSA가 아닌 Monolith 구조를 기본으로 사용합니다.

## 배포 흐름

```text
my-app
  ↓ GitHub Actions
Amazon ECR
  ↓ Argo CD 또는 kubectl
EKS
  ├── FastAPI Backend
  └── MariaDB
```

## 파일 구성

| 파일 | 역할 |
|---|---|
| `frontend.yml` | 웹 요청을 처리하는 애플리케이션 Deployment/Service |
| `db.yml` | MariaDB Deployment, PVC, Service |
| `secret.yml` | DB 접속 정보 Secret |
| `ingress.yml` | 외부 요청을 Kubernetes Service로 전달 |

## 이미지

애플리케이션 이미지는 Amazon ECR을 사용합니다.

```yaml
image: 938917088179.dkr.ecr.ap-northeast-2.amazonaws.com/fastapi-backend:latest
```

EKS 노드 IAM Role에 ECR Pull 권한이 있으면 별도의
`imagePullSecrets`는 필요하지 않습니다.

## 배포 전 확인

```bash
kubectl get storageclass
kubectl apply --dry-run=client -f devops/my-k8s-manifests/monolith/secret.yml
```

실제 배포는 리소스 중복 여부를 확인한 후 진행합니다.

```bash
kubectl apply -f devops/my-k8s-manifests/monolith/secret.yml
kubectl apply -f devops/my-k8s-manifests/monolith/db.yml
kubectl apply -f devops/my-k8s-manifests/monolith/frontend.yml
kubectl apply -f devops/my-k8s-manifests/monolith/ingress.yml
```

`msa/` 디렉터리는 Auth·Board 서비스를 별도 Deployment로 나누는 선택형
실습이며, 현재 기본 배포에서는 사용하지 않습니다.
