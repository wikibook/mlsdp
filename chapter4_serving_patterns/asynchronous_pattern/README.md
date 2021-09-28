# 비동기 추론 패턴

## 목적

비동기 추론 API 를 제공합니다.

## 전제

- Python 3.8 이상
- Docker
- Docker compose

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/asynchronous_pattern
```

1. 비동기 추론을 위한 Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:asynchronous_pattern_asynchronous_proxy_0.0.1 \
#     -f ./Dockerfile.proxy .
# docker build \
#     -t shibui/ml-system-in-actions:asynchronous_pattern_imagenet_inception_v3_0.0.1 \
#     -f ./imagenet_inception_v3/Dockerfile .
# docker build \
#     -t shibui/ml-system-in-actions:asynchronous_pattern_asynchronous_backend_0.0.1 \
#     -f ./Dockerfile.backend .
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
# 헬스 체크
$ curl localhost:8000/health
# 출력
# {"health":"ok"}

# 메타 데이터
$ curl localhost:8000/metadata
# 출력
# {
#   "model_spec": {
#     "name": "inception_v3",
#     "signature_name": "",
#     "version": "0"
#   },
#   "metadata": {
#     "signature_def": {
#       "signature_def": {
#         "serving_default": {
#           "inputs": {
#             "image": {
#               "dtype": "DT_STRING",
#               "tensor_shape": {
#                 "dim": [
#                   {
#                     "size": "-1",
#                     "name": ""
#                   }
#                 ],
#                 "unknown_rank": false
#               },
#               "name": "serving_default_image:0"
#             }
#           },
#           "outputs": {
#             "output_0": {
#               "dtype": "DT_STRING",
#               "tensor_shape": {
#                 "dim": [],
#                 "unknown_rank": true
#               },
#               "name": "StatefulPartitionedCall:0"
#             }
#           },
#           "method_name": "tensorflow/serving/predict"
#         },
#         "__saved_model_init_op": {
#           "inputs": {},
#           "outputs": {
#             "__saved_model_init_op": {
#               "dtype": "DT_INVALID",
#               "tensor_shape": {
#                 "dim": [],
#                 "unknown_rank": true
#               },
#               "name": "NoOp"
#             }
#           },
#           "method_name": ""
#         }
#       }
#     }
#   }
# }

# 라벨 목록
$ curl localhost:8000/label
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
$ curl localhost:8000/predict/test
# 출력
# {
#   "job_id": "f22689"
# }


# 이미지를 요청
$ (echo \
    -n '{"image_data": "'; \
    base64 imagenet_inception_v3/data/cat.jpg; \
    echo '"}') | \
   curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d @- \
    localhost:8000/predict
# 출력
# {
#   "job_id":"2f49aa"
# }

# 이미지 요청의 작업 ID 로부터 추론 결과를 요청
$ curl localhost:8000/job/2f49aa
# 출력
# {
#   "2f49aa": {
#     "prediction": "Siamese cat"
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
