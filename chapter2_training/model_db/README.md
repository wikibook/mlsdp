# 모델 관리 DB

## 목적

모델을 관리하기 위한 데이터베이스 및 서비스용 REST API 를 구축합니다. 
이 프로그램에서는 다음과 같은 구성의 모델 관리 서비스를 구축합니다. 

![img](./img/model_db.png)

## 전제

- Python 3.8 이상
- Docker
- Docker compose

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter2_training/model_db
```

1. 모델 DB 서비스（REST API Docker 이미지 빌드

```sh
$ make build
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:model_db_0.0.1 \
#     -f Dockerfile \
#     .
# 출력 생략
# dockerイメージ로 shibui/ml-system-in-actions:model_db_0.0.1 이 빌드됩니다.
```

2. Docker compose 로 모델 DB 기동

```sh
$ make c_up
# 실행 커맨드
# docker-compose \
#     -f ./docker-compose.yml \
#     up -d
```

3. 모델 DB 서비스 기동 확인

10초 정도 안에 기동됩니다.
브라우저에서 `localhost:8000/docs` 를 열고, Swagger 가 기동하는지 확인합니다. 

![img](./img/model_swagger.png)

4. 모델 DB 서비스 정지

```sh
$ make c_down
# 실행 커맨드
# docker-compose \
#     -f ./docker-compose.yml \
#     down
```
