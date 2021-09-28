# 조건 분기 추론 패턴

## 목적

상황에 따라 추론의 요청처를 바꾸는 패턴입니다.

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
~/ml-system-in-actions/chapter6_operation_management/condition_based_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
# 	-t shibui/ml-system-in-actions:condition_based_pattern_proxy_0.0.1 \
# 	-f ./Dockerfile.proxy \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:condition_based_pattern_imagenet_mobilenet_v2_0.0.1 \
# 	-f ./imagenet_mobilenet_v2/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:condition_based_pattern_plant_0.0.1 \
# 	-f ./plant/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:condition_based_pattern_client_0.0.1 \
# 	-f Dockerfile.client \
# 	.
```

2. Kubernetes 로 서비스 기동

```sh
$ make deploy
# 실행 커맨드
# istioctl install -y
# kubectl apply -f manifests/namespace.yml
# kubectl apply -f manifests/

# 가동확인
$ kubectl -n condition-based-serving get all
# 출력
# NAME                                     READY   STATUS    RESTARTS   AGE
# pod/client                               2/2     Running   0          59s
# pod/mobilenet-v2-c9dc95c89-66dn9         2/2     Running   0          58s
# pod/mobilenet-v2-c9dc95c89-985sq         2/2     Running   0          58s
# pod/mobilenet-v2-c9dc95c89-qs5ht         2/2     Running   0          58s
# pod/mobilenet-v2-proxy-59d9f899c-7bpbb   1/2     Running   0          57s
# pod/mobilenet-v2-proxy-59d9f899c-blwz9   2/2     Running   0          57s
# pod/mobilenet-v2-proxy-59d9f899c-c2mlz   2/2     Running   0          57s
# pod/plant-9766f44d7-d9wwb                2/2     Running   0          58s
# pod/plant-9766f44d7-jn6m2                2/2     Running   0          58s
# pod/plant-9766f44d7-lnn48                2/2     Running   0          58s
# pod/plant-proxy-558877db5c-d8wvz         2/2     Running   0          57s
# pod/plant-proxy-558877db5c-fwtsh         1/2     Running   0          57s
# pod/plant-proxy-558877db5c-qpxmz         2/2     Running   0          57s

# NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
# service/mobilenet-v2   ClusterIP   10.4.8.216    <none>        8500/TCP,8501/TCP   58s
# service/plant          ClusterIP   10.4.9.128    <none>        9500/TCP,9501/TCP   57s
# service/proxy          ClusterIP   10.4.11.158   <none>        8000/TCP            57s

# NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/mobilenet-v2         3/3     3            3           59s
# deployment.apps/mobilenet-v2-proxy   2/3     3            2           58s
# deployment.apps/plant                3/3     3            3           59s
# deployment.apps/plant-proxy          2/3     3            2           58s

# NAME                                           DESIRED   CURRENT   READY   AGE
# replicaset.apps/mobilenet-v2-c9dc95c89         3         3         3       59s
# replicaset.apps/mobilenet-v2-proxy-59d9f899c   3         3         2       58s
# replicaset.apps/plant-9766f44d7                3         3         3       59s
# replicaset.apps/plant-proxy-558877db5c         3         3         2       58s
```

3. 기동한 API 에 요청

```sh
# 클라이언트에 접속
$ kubectl -n condition-based-serving exec -it pod/client bash

# 헬스 체크
$ curl proxy.condition-based-serving.svc.cluster.local:8000/health
# 출력
# {
#   "health":"ok"
# }

# 메타 데이터
$ curl proxy.condition-based-serving.svc.cluster.local:8000/metadata
# 출력
# {
#   "data_type": "str",
#   "data_structure": "(1,1)",
#   "data_sample": "base64 encoded image file",
#   "prediction_type": "float32",
#   "prediction_structure": "(1,1001)",
#   "prediction_sample": "[0.07093159, 0.01558308, 0.01348537, ...]"
# }

# 라벨 목록
$ curl proxy.condition-based-serving.svc.cluster.local:8000/label
# 출력
# [
#   "background",
#   "tench",
#   "goldfish",
# ...
#   "bolete",
#   "ear",
#   "toilet tissue"
# ]


# 테스트 데이터로 추론 요청
$ curl proxy.condition-based-serving.svc.cluster.local:8000/predict/test
# 출력
# "Persian cat"


# 고양이 이미지를 ImageNet 추론기로 요청
$ (echo -n '{"image_data": "'; base64 cat.jpg; echo '"}') | \
	curl \
	-X POST \
	-H "Content-Type: application/json" \
	-d @- \
	proxy.condition-based-serving.svc.cluster.local:8000/predict
# 출력
# "Persian cat"


# 붓꽃 이미지를 식물 추론기로 요청
$ (echo -n '{"image_data": "'; base64 iris.jpg; echo '"}') | \
	curl \
	-X POST \
	-H "Content-Type: application/json" \
	-H "target: mountain" \
	-d @- \
	proxy.condition-based-serving.svc.cluster.local:8000/predict
# 출력
# "Iris versicolor"


# 고양이 이미지를 식물 추론기로 요청을 보내면, background 로 분류됨
$ (echo -n '{"image_data": "'; base64 cat.jpg; echo '"}') | \
	curl \
	-X POST \
	-H "Content-Type: application/json" \
	-H "target: mountain" \
	-d @- \
	proxy.condition-based-serving.svc.cluster.local:8000/predict
# 출력
# "background"

# 붓꽃 이미지를 ImageNet 추론기로 요청을 보내면, bee 로 분류됨
$ (echo -n '{"image_data": "'; base64 iris.jpg; echo '"}') | \
	curl \
	-X POST \
	-H "Content-Type: application/json" \
	-d @- \
	proxy.condition-based-serving.svc.cluster.local:8000/predict
# 출력
# "bee"
```

4. Kubernetes 에서 서비스 삭제

```sh
$ kubectl delete ns condition-based-serving
$ istioctl x uninstall --purge -y
$ kubectl delete ns istio-system
```
