# 전처리 추론 패턴

## 목적

전처리와 추론을 분할해 유연성을 향상합니다.

## 전제

- Python 3.8 이상
- Docker
- Docker compose

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/prep_pred_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:prep_pred_pattern_prep_0.0.1 \
#     -f ./Dockerfile.prep .
# docker build \
#     -t shibui/ml-system-in-actions:prep_pred_pattern_pred_0.0.1 \
#     -f ./Dockerfile.pred .
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
#   "data_type": "str",
#   "data_structure": "(1,1)",
#   "data_sample": "base64 encoded image file",
#   "prediction_type": "float32",
#   "prediction_structure": "(1,1000)",
#   "prediction_sample": "[0.07093159, 0.01558308, 0.01348537, ...]"
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
$ curl localhost:8000/predict/test/label
# 출력
# {
#   "prediction": "Siamese cat"
# }


# 이미지 요청
$ (echo -n '{"data": "'; base64 data/cat.jpg; echo '"}') | \
    curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d @- \
    localhost:8000/predict/label
# 출력
# {
#   "prediction": "Siamese cat"
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
