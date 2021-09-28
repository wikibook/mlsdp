# 섀도우 A/B 테스트 패턴

## 목적

섀도우로 A/B 테스트를 실시합니다.

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
~/ml-system-in-actions/chapter6_operation_management/shadow_ab_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
# 	-t shibui/ml-system-in-actions:shadow_ab_pattern_api_0.0.1 \
# 	-f Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:shadow_ab_pattern_loader_0.0.1 \
# 	-f model_loader/Dockerfile \
# 	.
# docker build \
# 	-t shibui/ml-system-in-actions:shadow_ab_pattern_client_0.0.1 \
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
$ kubectl -n shadow-ab get all
# 출력
# NAME                            READY   STATUS    RESTARTS   AGE
# pod/client                      2/2     Running   0          44s
# pod/iris-rf-7cd8cb9d78-lcsq6    2/2     Running   0          43s
# pod/iris-svc-74dc7654b8-xthmh   2/2     Running   0          44s

# NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
# service/iris   ClusterIP   10.4.0.14    <none>        8000/TCP   43s

# NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/iris-rf    3/3     3            3           44s
# deployment.apps/iris-svc   3/3     3            3           44s

# NAME                                  DESIRED   CURRENT   READY   AGE
# replicaset.apps/iris-rf-7cd8cb9d78    3         3         3       44s
# replicaset.apps/iris-svc-74dc7654b8   3         3         3       44s

# NAME                                           REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
# horizontalpodautoscaler.autoscaling/iris-rf    Deployment/iris-rf    <unknown>/70%   3         10        3          43s
# horizontalpodautoscaler.autoscaling/iris-svc   Deployment/iris-svc   <unknown>/70%   3         10        3          43s
```

3. 기동한 API 에 요청

```sh
# 클라이언트에 접속
$ kubectl -n shadow-ab exec -it pod/client bash

# 같은 엔드포인트에서 여러번 요청을 보냄
$ curl http://iris.shadow-ab.svc.cluster.local:8000/predict-test/000000
# {"job_id":"000000","prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355]}
$ curl http://iris.shadow-ab.svc.cluster.local:8000/predict-test/000000
# {"job_id":"000000","prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355]}
$ curl http://iris.shadow-ab.svc.cluster.local:8000/predict-test/000000
# {"job_id":"000000","prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355]}
$ curl http://iris.shadow-ab.svc.cluster.local:8000/predict-test/000000
# {"job_id":"000000","prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355]}
$ curl http://iris.shadow-ab.svc.cluster.local:8000/predict-test/000000
# {"job_id":"000000","prediction":[0.9709315896034241,0.015583082102239132,0.013485366478562355]}

# 클라이언트를 exit 하고, iris-rf 및 iris-svc 로그를 참조합니다. iris-rf、iris-svc 양측에서 추론이 실행되었음을 확인할 수 있습니다.
$ kubectl -n shadow-ab logs pod/iris-rf-7cd8cb9d78-lcsq6 iris-rf
# 출력
# [2021-02-06 09:51:35] [INFO] [14] [src.app.routers.routers] [_predict_test] [37] execute: [000000]
# [2021-02-06 09:51:35] [INFO] [14] [src.ml.prediction] [predict] [50] predict proba [0.99999994 0.         0.        ]
# [2021-02-06 09:51:35] [INFO] [14] [src.app.routers.routers] [wrapper] [33] [iris_rf.onnx] [/predict-test] [000000] [1.0628700256347656 ms] [None] [[0.9999999403953552, 0.0, 0.0]]
# [2021-02-06 09:51:35] [INFO] [14] [uvicorn.access] [send] [458] 10.0.2.36:0 - "GET /predict-test/000000 HTTP/1.1" 200

$ kubectl -n shadow-ab logs pod/iris-svc-74dc7654b8-xthmh iris-svc
# 출력
# [2021-02-06 09:51:35] [INFO] [8] [src.app.routers.routers] [_predict_test] [37] execute: [000000]
# [2021-02-06 09:51:35] [INFO] [8] [src.ml.prediction] [predict] [50] predict proba [0.97093159 0.01558308 0.01348537]
# [2021-02-06 09:51:35] [INFO] [8] [src.app.routers.routers] [wrapper] [33] [iris_svc.onnx] [/predict-test] [000000] [0.8084774017333984 ms] [None] [[0.9709315896034241, 0.015583082102239132, 0.013485366478562355]]
# [2021-02-06 09:51:35] [INFO] [8] [uvicorn.access] [send] [458] 127.0.0.1:48148 - "GET /predict-test/000000 HTTP/1.1" 200
```

4. Kubernetes 에서 서비스 삭제

```sh
$ kubectl delete ns shadow-ab
$ istioctl x uninstall --purge -y
$ kubectl delete ns istio-system
```
