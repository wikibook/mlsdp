# 모델-인-이미지 패턴

## 목적

모델-인-이미지 패턴에서는 학습으로 모델파일을 생성한 후에, 모델파일을 포함한 추론서버 이미지를 빌드합니다. 서버 이미지를 Pull 하고 추론서버를 기동시키면 본격적으로 가동됩니다.

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
~/ml-system-in-actions/chapter3_release_patterns/model_in_image_pattern
```

1. 추론용 Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:model_in_image_pattern_0.0.1 \
#     -f Dockerfile \
#     .
```

2. 추론기를 Kubernetes 클러스터에 디플로이

```sh
$ make deploy
# 실행 커맨드
# kubectl apply -f manifests/namespace.yml
# kubectl apply -f manifests/deployment.yml

# 디플로이 확인
$ kubectl -n model-in-image get pods,deploy,svc
# NAME                                  READY   STATUS    RESTARTS   AGE
# pod/model-in-image-5c64988c5d-5phxn   1/1     Running   0          28s
# pod/model-in-image-5c64988c5d-lljmg   1/1     Running   0          28s
# pod/model-in-image-5c64988c5d-qhpms   1/1     Running   0          28s
# pod/model-in-image-5c64988c5d-rgwlx   1/1     Running   0          28s

# NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/model-in-image   4/4     4            4           28s

# NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
# service/model-in-image   ClusterIP   10.84.9.167   <none>        8000/TCP   28s
```

3. Kubernetes 에서 model-in-image 삭제

```sh
$ make delete
# 실행 커맨드
# kubectl delete ns model-in-image
```
