# 부하 테스트 패턴

## 목적

추론기에 부하 테스트를 실시합니다.

## 전제

- Python 3.8 以上
- Docker
- Kubernetes 클러스터 또는 minikube

이 프로그램은 Kubernetes 클러스터 또는 minikube 가 필요합니다.
Kubernetes 클러스터는 독자적으로 구축하거나, 각 클라우드 매니지드 서비스（GCP GKE、AWS EKS、MS Azure AKS 等）를 이용해 주십시오.
GCP GKE 클러스터로 가동을 확인했습니다.

- [Kubernetes 클러스터 구축](https://kubernetes.io/ja/docs/setup/)
- [minikube](https://kubernetes.io/ja/docs/setup/learning-environment/minikube/)

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter6_operation_management/load_test_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:load_test_pattern_api_0.0.1 \
#     -f Dockerfile \
#     .
# docker build \
# 	-t shibui/ml-system-in-actions:load_test_pattern_loader_0.0.1 \
# 	-f model_loader/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:load_test_pattern_client_0.0.1 \
# 	-f Dockerfile.client \
# 	.
```

2. Kubernetes 로 서비스 기동

```sh
$ make deploy
# 실행 커맨드
# kubectl apply -f manifests/namespace.yml
# kubectl apply -f manifests/

# 가동확인
$ kubectl -n load-test get all
# 출력
# NAME                            READY   STATUS    RESTARTS   AGE
# pod/client                      1/1     Running   0          45s
# pod/iris-svc-598875545c-5bvpq   1/1     Running   0          44s
# pod/iris-svc-598875545c-8t47v   1/1     Running   0          44s
# pod/iris-svc-598875545c-mqgcd   1/1     Running   0          44s

# NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
# service/iris-svc   ClusterIP   10.4.7.177   <none>        8000/TCP   44s

# NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/iris-svc   3/3     3            3           44s

# NAME                                  DESIRED   CURRENT   READY   AGE
# replicaset.apps/iris-svc-598875545c   3         3         3       44s
```

3. 기동한 API 에 요청

```sh
# 클라이언트에 접속
$ kubectl -n load-test exec -it pod/client bash

# 부하 테스트를 실행
$ vegeta attack -duration=60s -rate=100 -targets=vegeta/post-target | vegeta report -type=text
# 출력
# Requests      [total, rate, throughput]         6000, 100.02, 100.01
# Duration      [total, attack, wait]             59.992s, 59.99s, 1.915ms
# Latencies     [min, mean, 50, 90, 95, 99, max]  1.246ms, 1.939ms, 1.928ms, 2.182ms, 2.279ms, 2.72ms, 23.222ms
# Bytes In      [total, mean]                     438000, 73.00
# Bytes Out     [total, mean]                     210000, 35.00
# Success       [ratio]                           100.00%
# Status Codes  [code:count]                      200:6000
# Error Set:
```

4. Kubernetes 에서 서비스 삭제

```sh
$ kubectl delete ns load-test
```
