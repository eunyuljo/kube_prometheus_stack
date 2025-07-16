# EKS PrometheusRule Collection

EKS 클러스터를 위한 계층별 Prometheus 알림 규칙 모음입니다.

## 📋 파일 구조

| 파일명 | 설명 |
|--------|------|
| `01_eks-cluster-alerts.yaml` | 클러스터 레벨 (API 서버, 전체 리소스) |
| `02_eks-node-alerts.yaml` | 노드 레벨 (CPU, 메모리, 디스크) |
| `03_eks-deployment-alerts.yaml` | 디플로이먼트 레벨 (롤아웃, 레플리카) |
| `04_eks-pod-alerts.yaml` | 파드 레벨 (상태, 재시작, 실패) |
| `05_eks-service-alerts.yaml` | 서비스 레벨 (엔드포인트, 인그레스) |
| `06_eks-namespace-alerts.yaml` | 네임스페이스 레벨 (쿼터, 리소스) |
| `07_eks-workload-alerts.yaml` | 워크로드 레벨 (StatefulSet, DaemonSet, Job) |
| `08_eks-network-alerts.yaml` | 네트워크 레벨 (CNI, DNS, 로드밸런서) |
| `09_eks-rbac-security-alerts.yaml` | 보안 레벨 (RBAC, 파드 보안) |
| `10_eks-aws-services-alerts.yaml` | AWS 서비스 (IAM, ECR, CloudWatch) |
| `11_eks-additional-alerts.yaml` | 기타 (스토리지, 오토스케일링, 백업) |

## 🚀 빠른 시작

### 전제 조건
- kube-prometheus-stack 설치됨
- Prometheus가 `monitoring` 네임스페이스에 배포됨

### 설치
```bash
# 1. 모든 알림 규칙 적용
kubectl apply -f .

# 2. 적용 확인
kubectl get prometheusrules -n monitoring
```

### 단계별 설치 (권장)
```bash
# 1단계: 핵심 모니터링
kubectl apply -f 01_eks-cluster-alerts.yaml
kubectl apply -f 02_eks-node-alerts.yaml

# 2단계: 워크로드 모니터링
kubectl apply -f 03_eks-deployment-alerts.yaml
kubectl apply -f 04_eks-pod-alerts.yaml
kubectl apply -f 05_eks-service-alerts.yaml

# 3단계: 고급 모니터링
kubectl apply -f 06_eks-namespace-alerts.yaml
kubectl apply -f 07_eks-workload-alerts.yaml
kubectl apply -f 08_eks-network-alerts.yaml

# 4단계: 보안 및 AWS 서비스
kubectl apply -f 09_eks-rbac-security-alerts.yaml
kubectl apply -f 10_eks-aws-services-alerts.yaml
kubectl apply -f 11_eks-additional-alerts.yaml
```

## ⚙️ 커스터마이징

### 네임스페이스 변경
```bash
# monitoring이 아닌 다른 네임스페이스 사용 시
sed -i 's/namespace: monitoring/namespace: YOUR_NAMESPACE/g' *.yaml
```

### 라벨 셀렉터 변경
```bash
# kube-prometheus-stack이 아닌 다른 릴리스 사용 시
sed -i 's/release: kube-prometheus-stack/release: YOUR_RELEASE/g' *.yaml
```

### 임계값 조정 예시
```yaml
# CPU 사용률 임계값 변경 (80% → 90%)
- alert: EKSNodeHighCPUUsage
  expr: |
    (
      100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    ) > 90  # 80에서 90으로 변경
```

## 🎯 주요 알림 카테고리

| 심각도 | 설명 | 대응 시간 |
|--------|------|----------|
| `critical` | 즉시 대응 필요 | 5분 이내 |
| `warning` | 모니터링 필요 | 30분 이내 |
| `info` | 참고용 | 추적 목적 |

## 🔍 문제 해결

### 적용 오류 시
```bash
# 문법 검증
kubectl apply --dry-run=client -f FILENAME.yaml

# 로그 확인
kubectl logs -n monitoring prometheus-kube-prometheus-prometheus-0 -c prometheus
```

### 메트릭 확인
```bash
# Prometheus UI에서 메트릭 존재 여부 확인
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
```

### 일반적인 오류 해결
```bash
# PrometheusRule 상태 확인
kubectl describe prometheusrule RULE_NAME -n monitoring

# 적용된 규칙 확인
kubectl get prometheusrules -n monitoring -o wide
```

## 📊 권장 Grafana 대시보드

| 대시보드 | ID | 설명 |
|----------|----|----|
| Kubernetes Cluster Overview | 7249 | 클러스터 전체 현황 |
| Node Exporter Full | 1860 | 노드 상세 메트릭 |
| Kubernetes Pods | 6417 | 파드 리소스 사용량 |
| Kubernetes Networking | 12124 | 네트워크 메트릭 |

## 🔧 고급 설정

### AlertManager 라우팅 예시
```yaml
route:
  group_by: ['cluster', 'namespace']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  routes:
  - match:
      severity: critical
    receiver: 'critical-alerts'
  - match:
      severity: warning
    receiver: 'warning-alerts'
```

### 특정 네임스페이스 제외
```yaml
# 시스템 네임스페이스 제외 예시
expr: |
  rate(kube_pod_container_status_restarts_total{namespace!~"kube-system|kube-public|monitoring"}[15m]) > 0
```
