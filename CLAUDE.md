# CLAUDE.md — Stockops-Gitops

> 이 파일을 Stockops-Gitops 저장소 루트의 `CLAUDE.md`로 저장하세요.

## 이 저장소

ArgoCD가 감시하는 GitOps 매니페스트 저장소. 이 레포에 push되면 ArgoCD가 자동으로 sync한다.

## 자동 실행 원칙

- 작업 시작 시 항상 먼저 `git pull --rebase`로 최신화한다. 사용자에게 묻지 않는다.
- 매니페스트를 수정한 뒤에는 바로 커밋 + push한다. push 전에 "이대로 push할까요?" 같은 확인을 구하지 않는다.
- push 직후, ArgoCD가 정상적으로 sync를 시작했는지 확인한다:
  ```
  kubectl get applications -n argocd
  ```
  `SyncStatus`가 기대와 다르거나 `Health`가 `Degraded`/`Unknown`으로 오래 머물면, 원인을 조사해서 직접 해결을 시도한다(예: manifest 문법 오류, 이미지 태그 오타, 잘못된 리소스 참조). 막연히 "에러가 났습니다"라고만 보고하지 않고, 원인과 조치를 같이 보고한다.
- force push는 하지 않는다.

## 절대 묻지 않아도 되는 것

pull/push 시점, ArgoCD sync 확인 여부는 묻지 않는다. 단, **변경하려는 매니페스트가 production 트래픽에 영향을 주는 게 명확하고 되돌리기 어려운 경우**(예: 프로덕션 replica를 0으로 내리는 변경)는 진행 전에 한 번 알린다.
