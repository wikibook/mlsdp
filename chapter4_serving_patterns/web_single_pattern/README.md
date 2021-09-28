# Web 싱글 패턴

## 목적

Web API 로 추론기를 공개합니다.

## 전제

- Python 3.8 이상
- Docker

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/web_single_pattern
```

1. Docker 이미지 빌드

```sh
$ make build_all
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:web_single_pattern_0.0.1 \
#     -f Dockerfile \
#     .
```

2. Docker 로 서비스를 기동

```sh
$ make run
# 실행 커맨드
# docker run \
#     -d \
#     --name web_single_pattern \
#     -p 8000:8000 \
#     shibui/ml-system-in-actions:web_single_pattern_0.0.1
```

3. 기동한 API 에 클라이언트로부터 요청

```sh
# 헬스 체크
$ curl localhost:8000/health
# 출력
# {
#   "health":"ok"
# }

# 메타 데이터
$ curl localhost:8000/metadata
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
#   "prediction_structure": "(1,3)",
#   "prediction_sample": [
#     0.97093159,
#     0.01558308,
#     0.01348537
#   ]
# }


# 라벨 목록
$ curl localhost:8000/label
# 출력
# {
#   "0": "setosa",
#   "1": "versicolor",
#   "2": "virginica"
# }


# 테스트 데이터로 추론 요청
$ curl localhost:8000/predict/test
# 출력
# {
#   "prediction": [
#     0.9709315896034241,
#     0.015583082102239132,
#     0.013485366478562355
#   ]
# }


# 추론 요청
$ curl \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"data": [[1.0, 2.0, 3.0, 4.0]]}' \
    localhost:8000/predict
# 출력
# {
#   "prediction": [
#     0.3613327741622925,
#     0.2574760615825653,
#     0.3811912536621094
#   ]
# }
```

4. Docker 컨테이너 정지

```sh
$ make stop
# 실행 커맨드
# docker rm \
#   -f web_single_pattern
```
