# 모델 로드 패턴

## 목적

모델 로드 패턴에서는 서버 이미지를 Pull 한 뒤에 추론서버를 기동하고 난 다음, 모델 파일을 로드해서 추론서버를 본격적으로 가동합니다.
모델파일의 로드를 환경변수로 변경하면서 추론서버에서 가동할 모델을 유연하게 바꿀 수 있습니다. 

## 전제

- Python 3.8 이상
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
~/ml-system-in-actions/chapter3_release_patterns/model_load_pattern
```

1. 추론용 Docker 이미지 및 모델 로드용 Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:model_load_pattern_api_0.0.1 \
#     -f Dockerfile \
#     .
# docker build \
#     -t shibui/ml-system-in-actions:model_load_pattern_loader_0.0.1 \
#     -f model_loader/Dockerfile \
#     .
```

2. 추론기와 사이드카를 Kubernetes 클러스터에 디플로이

```sh
$ make deploy
# 실행 커맨드
# kubectl apply -f manifests/namespace.yml
# kubectl apply -f manifests/deployment.yml

# 디플로이 확인
$ kubectl -n model-load get pods,deploy,svc
# NAME                              READY   STATUS    RESTARTS   AGE
# pod/model-load-6b4bb6f96c-b95f2   1/1     Running   0          33s
# pod/model-load-6b4bb6f96c-kntxk   1/1     Running   0          33s
# pod/model-load-6b4bb6f96c-s8zjx   1/1     Running   0          33s
# pod/model-load-6b4bb6f96c-zdwqj   1/1     Running   0          33s

# NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/model-load   4/4     4            4           33s

# NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
# service/model-load   ClusterIP   10.84.11.108   <none>        8000/TCP   33s
```

3. Kubernetes 에서 model-load 삭제

```sh
$ make delete
# 실행 커맨드
# kubectl delete ns model-load
```
