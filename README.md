# StockOps GitOps

![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.x-%23EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Kustomize](https://img.shields.io/badge/Kustomize-%23326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)

StockOps CD(GitOps) 전용 레포입니다. ArgoCD가 이 레포를 단일 진실(source of truth)로 바라보고 EKS 클러스터에 동기화합니다.

> 인프라(Terraform)는 **[Stockops-Infra](https://github.com/jinuuuKim/Stockops-Infra)**, 애플리케이션 빌드·배포 파이프라인은 **[Stockops-Application](https://github.com/jinuuuKim/Stockops-Application)** 에서 관리합니다.

---

## 디렉토리 구조

```
Stockops-GitOps/
├── apps/
│   └── stockops/
│       ├── seoul/
│       │   ├── base/             # 리전 공통 워크로드 (api-server, ai-module Deployment)
│       │   └── overlays/         # 서울 전용 — 네임스페이스·이미지 태그 (CI가 SHA 갱신)
│       └── ohio/
│           ├── base/             # 리전 공통 워크로드
│           └── overlays/         # 오하이오 전용 — 네임스페이스·이미지 태그
└── argocd/
    ├── stockops-seoul-application.yaml   # ArgoCD Application (서울 클러스터)
    └── stockops-ohio-application.yaml    # ArgoCD Application (오하이오 클러스터)
```

---

## 소유 경계

| 소유자 | 관리 대상 |
|--------|-----------|
| **Terraform (Stockops-Infra)** | VPC · EKS · ALB · RDS · ECR · ESO · LBC · ArgoCD 설치 · aws-auth · IRSA · Service · TargetGroupBinding · HPA · Redis · Namespace |
| **이 레포 (ArgoCD)** | `stockops-api` · `stockops-ai` **Deployment만** |

> `replicas`는 HPA(Terraform 소유)가 관리합니다. manifest에 `replicas` 필드가 없으며, ArgoCD Application에 `ignoreDifferences: /spec/replicas`가 설정되어 있습니다.

---

## 배포 흐름

```
Stockops-Application (GitHub Actions)
    └─ 이미지 빌드 → ECR push (서울/오하이오)
         └─ overlays/kustomization.yaml 이미지 SHA 업데이트 commit/push
              └─ ArgoCD가 변경 감지 → 자동 sync
                   ├─ seoul-cluster: apps/stockops/seoul/overlays
                   └─ ohio-cluster:  apps/stockops/ohio/overlays
```

---

## ArgoCD Application 등록

EKS 클러스터를 새로 생성한 후에는 ArgoCD Application을 수동으로 등록해야 합니다.

```powershell
kubectl apply -f argocd/stockops-seoul-application.yaml \
  --context arn:aws:eks:ap-northeast-2:448768137813:cluster/seoul-cluster

kubectl apply -f argocd/stockops-ohio-application.yaml \
  --context arn:aws:eks:us-east-2:448768137813:cluster/ohio-cluster

# 등록 확인
kubectl get application -n argocd \
  --context arn:aws:eks:ap-northeast-2:448768137813:cluster/seoul-cluster
kubectl get application -n argocd \
  --context arn:aws:eks:us-east-2:448768137813:cluster/ohio-cluster
```

---

## manifest 직접 수정

환경변수·리소스(requests/limits) 변경은 이 레포의 overlay를 직접 수정하고 push하면 ArgoCD가 자동으로 클러스터에 반영합니다.

```bash
# 로컬 렌더 확인 (클러스터에 영향 없음)
kubectl kustomize apps/stockops/seoul/overlays
kubectl kustomize apps/stockops/ohio/overlays
```
