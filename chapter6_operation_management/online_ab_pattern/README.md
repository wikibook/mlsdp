# 온라인 A/B 테스트 패턴

## 목적

온라인으로 A/B 테스트를 실시합니다.

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
~/ml-system-in-actions/chapter6_operation_management/online_ab_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
# 	-t shibui/ml-system-in-actions:online_ab_pattern_api_0.0.1 \
# 	-f Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:online_ab_pattern_loader_0.0.1 \
# 	-f model_loader/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:online_ab_pattern_client_0.0.1 \
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
$ kubectl -n online-ab get all
# 출력
# NAME                            READY   STATUS    RESTARTS   AGE
# pod/client                      2/2     Running   0          70s
# pod/iris-rf-f9b9d8b98-5rpms     2/2     Running   0          70s
# pod/iris-rf-f9b9d8b98-m8h4j     2/2     Running   0          70s
# pod/iris-rf-f9b9d8b98-ws7x9     2/2     Running   0          70s
# pod/iris-svc-85c897f9f4-78n69   2/2     Running   0          70s
# pod/iris-svc-85c897f9f4-ccs69   2/2     Running   0          70s
# pod/iris-svc-85c897f9f4-rmqct   2/2     Running   0          70s

# NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
# service/iris   ClusterIP   10.4.7.184   <none>        8000/TCP   69s

# NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/iris-rf    3/3     3            3           70s
# deployment.apps/iris-svc   3/3     3            3           71s

# NAME                                  DESIRED   CURRENT   READY   AGE
# replicaset.apps/iris-rf-f9b9d8b98     3         3         3       71s
# replicaset.apps/iris-svc-85c897f9f4   3         3         3       71s

# NAME                                           REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
# horizontalpodautoscaler.autoscaling/iris-rf    Deployment/iris-rf    <unknown>/70%   3         10        3          70s
# horizontalpodautoscaler.autoscaling/iris-svc   Deployment/iris-svc   <unknown>/70%   3         10        3          70s
```

3. 기동한 API 에 요청

```sh
# 클라이언트에 접속
$ kubectl -n online-ab exec -it pod/client bash

# 같은 엔드포인트에 여러번 요청을 보내, 두 종류의 응답을 얻는지 확인
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355],"mode":"iris_svc.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355],"mode":"iris_svc.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355],"mode":"iris_svc.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355],"mode":"iris_svc.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9999999403953552,0.0,0.0],"mode":"iris_rf.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9999999403953552,0.0,0.0],"mode":"iris_rf.onnx"}
$ curl http://iris.online-ab.svc.cluster.local:8000/predict/test
# {"prediction":[0.9999999403953552,0.0,0.0],"mode":"iris_rf.onnx"}
```

4. Kubernetes 에서 서비스 삭제

```sh
$ kubectl delete ns online-ab
$ istioctl x uninstall --purge -y
$ kubectl delete ns istio-system
```
