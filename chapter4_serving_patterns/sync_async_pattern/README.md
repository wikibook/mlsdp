# 시간차 추론 패턴

## 목적

동기적 추론기와 비동기적 추론기를 조합해 여러 추론의 지연을 보조합니다.

## 전제

- Python 3.8 이상
- Docker
- Docker compose

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/sync_async_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:sync_async_pattern_sync_async_proxy_0.0.1 \
#     -f ./Dockerfile.proxy .
# docker build \
#     -t shibui/ml-system-in-actions:sync_async_pattern_imagenet_mobilenet_v2_0.0.1 \
#     -f ./imagenet_mobilenet_v2/Dockerfile .
# docker build \
#     -t shibui/ml-system-in-actions:sync_async_pattern_imagenet_inception_v3_0.0.1 \
#     -f ./imagenet_inception_v3/Dockerfile .
# docker build \
#     -t shibui/ml-system-in-actions:sync_async_pattern_sync_async_backend_0.0.1 \
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
# {
#   "health":"ok"
# }

# 백엔드 헬스체크
$ curl localhost:8000/health/all
# 출력
# {
#   "mobilenet_v2": "ok",
#   "inception_v3": "ok"
# }


# 메타 데이터
$ curl localhost:8000/metadata
# 출력
# {
#   "data_type": "str",
#   "data_structure": "(1,1)",
#   "data_sample": "base64 encoded image file",
#   "prediction_type": "float32",
#   "prediction_structure": "(1,1001)",
#   "prediction_sample": "[0.07093159, 0.01558308, 0.01348537, ...]"
# }


# 테스트 데이터로 추론 요청
$ curl localhost:8000/predict/test
# 출력
# {
#   "job_id": "6195b8",
#   "mobilenet_v2": "Persian cat"
# }


# 이미지 요청
$ (echo -n '{"image_data": "'; base64 data/cat.jpg; echo '"}') | \
    curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d @- \
    localhost:8000/predict
# 출력
# {
#   "job_id": "9e987f",
#   "mobilenet_v2": "Persian cat"
# }

# 이미지 요청의 작업 ID 로부터 추론결과를 요청
$ curl localhost:8000/job/2f49aa
# 출력
# {
#   "6195b8": {
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
