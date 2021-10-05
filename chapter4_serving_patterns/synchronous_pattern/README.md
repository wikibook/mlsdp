# 동기 추론 패턴

## 목적

요청에 대해 동기적으로 추론하고 응답합니다.

## 전제

- Python 3.8 이상
- Docker

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/synchronous_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:synchronous_pattern_imagenet_inception_v3_0.0.1 \
#     -f imagenet_inception_v3/Dockerfile .
```

2. Docker 컨테이너 서비스 기동

```sh
$ make run
# 실행 커맨드
# docker run \
#     -d \
#     --name imagenet_inception_v3 \
#     -p 8500:8500 \
#     -p 8501:8501 \
#     shibui/ml-system-in-actions:synchronous_pattern_imagenet_inception_v3_0.0.1
```

3. 기동한 API 에 클라이언트로부터 요청

```sh
# 메타 데이터
$ curl localhost:8501/v1/models/inception_v3/versions/0/metadata
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



# GRPC 로 이미지 요청
$ python \
    -m client.request_inception_v3 \
    --image_file ./cat.jpg \
    --format grpc
# 출력
# Siamese cat


# REST 로 이미지 요청
$ python \
    -m client.request_inception_v3 \
    --image_file ./cat.jpg \
    --format rest
# 출력
# Siamese cat
```

4. Docker 컨테이너 정지

```sh
$ make stop
# 실행 커맨드
# docker rm \
#   -f imagenet_inception_v3
```
