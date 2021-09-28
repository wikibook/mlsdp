# 병렬 마이크로서비스 패턴

## 목적

추론기를 병렬 마이크로서비스로 배치합니다.

## 전제

- Python 3.8 이상
- Docker
- Docker compose

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/horizontal_microservice_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:horizontal_microservice_pattern_setosa_0.0.1 \
#     -f ./Dockerfile.service.setosa \
#     .
# docker build \
#     -t shibui/ml-system-in-actions:horizontal_microservice_pattern_versicolor_0.0.1 \
#     -f ./Dockerfile.service.versicolor \
#     .
# docker build \
#     -t shibui/ml-system-in-actions:horizontal_microservice_pattern_virginica_0.0.1 \
#     -f ./Dockerfile.service.virginica \
#     .
# docker build \
#     -t shibui/ml-system-in-actions:horizontal_microservice_pattern_proxy_0.0.1 \
#     -f ./Dockerfile.proxy \
#     .
```

2. Docker compose 로 각 서비스 기동

```sh
$ make c_up
# 실행 커맨드
# docker-compose \
#     -f ./docker-compose.yml \
#     up -d
```

3. 기동한 API 에 요청

```sh
# proxy에 헬스 체크
$ curl localhost:9000/health
# 출력
# {"health":"ok"}

# 추론기에 헬스 체크
$ curl localhost:9000/health/all
# 출력
# {
#   "setosa": {
#     "health": "ok"
#   },
#   "virginica": {
#     "health": "ok"
#   },
#   "versicolor": {
#     "health": "ok"
#   }
# }

# 메타 데이터
$ curl localhost:9000/metadata
# 출력
# {
#   "data_type": "float32",
#   "data_structure": "(1,4)",
#   "data_sample": [
#     [
#       5.1,
#       3.5,
#       1.4,
#       0.2
#     ]
#   ],
#   "prediction_type": "float32",
#   "prediction_structure": "(1,2)",
#   "prediction_sample": {
#     "service_setosa": [
#       0.97,
#       0.03
#     ],
#     "service_versicolor": [
#       0.97,
#       0.03
#     ],
#     "service_virginica": [
#       0.97,
#       0.03
#     ]
#   }
# }

# 테스트 데이터로 추론 요청(GET)
$ curl localhost:9000/predict/get/test
# 출력
# {
#   "setosa": {
#     "prediction": [
#       0.9824145436286926,
#       0.017585478723049164
#     ]
#   },
#   "virginica": {
#     "prediction": [
#       0.011562099680304527,
#       0.9884378910064697
#     ]
#   },
#   "versicolor": {
#     "prediction": [
#       0.0046140821650624275,
#       0.9953859448432922
#     ]
#   }
# }

# 테스트 데이터로 추론 요청(POST)
$ curl localhost:9000/predict/post/test
# 출력
# {
#   "setosa": {
#     "prediction": [
#       0.9824145436286926,
#       0.017585478723049164
#     ]
#   },
#   "virginica": {
#     "prediction": [
#       0.011562099680304527,
#       0.9884378910064697
#     ]
#   },
#   "versicolor": {
#     "prediction": [
#       0.0046140821650624275,
#       0.9953859448432922
#     ]
#   }
# }


# POST 요청
$ curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"data": [[1.0, 2.0, 3.0, 4.0]]}' \
    localhost:9000/predict
# 출력
# {
#   "setosa": {
#     "prediction": [
#       0.2897033989429474,
#       0.710296630859375
#     ]
#   },
#   "virginica": {
#     "prediction": [
#       0.3042130172252655,
#       0.6957869529724121
#     ]
#   },
#   "versicolor": {
#     "prediction": [
#       0.05282164365053177,
#       0.9471783638000488
#     ]
#   }
# }

# 라벨을 요청
$ curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"data": [[1.0, 2.0, 3.0, 4.0]]}' \
    localhost:9000/predict/label
# 출력
# {
#   "prediction": {
#     "proba": 0.3042130172252655,
#     "label": "virginica"
#   }
# }

```

4. Docker compose 정지

```sh
$ make c_down
# 실행 커맨드
# docker-compose \
#     -f ./docker-compose.yml \
#     down
```
