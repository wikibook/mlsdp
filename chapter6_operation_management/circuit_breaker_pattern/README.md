# 추론 서킷브레이커 패턴

## 목적

서킷브레이커로 고부하에 대처합니다.

## 전제

- Python 3.8 이상
- Docker
- Kubernetes 클러스터 또는 minikube

이 프로그램은 Kubernetes 클러스터 또는 minikube 가 필요합니다.
Kubernetes 클러스터는 독자적으로 구축하거나, 각 클라우드 매니지드 서비스（GCP GKE、AWS EKS、MS Azure AKS 등）를 이용해 주십시오.
GCP GKE 클러스터로 가동을 확인했습니다.

- [Kubernetes 클러스터 구축](https://kubernetes.io/ja/docs/setup/)
- [minikube](https://kubernetes.io/ja/docs/setup/learning-environment/minikube/)

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter6_operation_management/circuit_breaker_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
# 	-t shibui/ml-system-in-actions:circuit_breaker_pattern_api_0.0.1 \
# 	-f Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:circuit_breaker_pattern_loader_0.0.1 \
# 	-f model_loader/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:circuit_breaker_pattern_client_0.0.1 \
# 	-f Dockerfile.client \
# 	.
```

2. Kubernetes 에 Istio 를 인스톨하고, 각 서비스 기동

```sh
$ make deploy
# 실행 커맨드
# istioctl install -y
# kubectl apply -f manifests/namespace.yml
# kubectl apply -f manifests/

# 서비스 기동 확인
$ kubectl -n circuit-breaker get all
# 출력
# NAME                            READY   STATUS    RESTARTS   AGE
# pod/client                      2/2     Running   0          74s
# pod/iris-svc-56758cc7cf-6gdhb   2/2     Running   0          74s
# pod/iris-svc-56758cc7cf-k6wq7   2/2     Running   0          74s
# pod/iris-svc-56758cc7cf-lcgqp   2/2     Running   0          74s

# NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
# service/iris-svc   ClusterIP   10.4.3.84    <none>        8000/TCP   74s

# NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/iris-svc   3/3     3            3           75s

# NAME                                  DESIRED   CURRENT   READY   AGE
# replicaset.apps/iris-svc-56758cc7cf   3         3         3       75s
```

3. 기동한 API 부하 테스트

```sh
# 부하 테스트 클라이언트에 접속
$ kubectl -n circuit-breaker exec -it pod/client bash
# 출력
# kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
# Defaulting container name to client.
# Use 'kubectl describe pod/client -n circuit-breaker' to see all of the containers in this pod.

# 부하 테스트 실행
$ vegeta attack -duration=10s -rate=1000 -targets=vegeta/post-target | vegeta report -type=text
# 출력
# Requests      [total, rate, throughput]         10000, 1000.04, 764.34
# Duration      [total, attack, wait]             10.242s, 10s, 241.972ms
# Latencies     [min, mean, 50, 90, 95, 99, max]  362.662µs, 177.282ms, 90.438ms, 427.767ms, 672.361ms, 1.419s, 2.929s
# Bytes In      [total, mean]                     747376, 74.74
# Bytes Out     [total, mean]                     350000, 35.00
# Success       [ratio]                           78.28%
# Status Codes  [code:count]                      200:7828  503:2172
# Error Set:
# 503 Service Unavailable
```

4. Kubernetes 에서 서비스 삭제

```sh
$ kubectl delete ns circuit-breaker
$ istioctl x uninstall --purge -y
$ kubectl delete ns istio-system
```
