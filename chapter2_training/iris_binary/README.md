# Iris 데이터셋의 이진 분류 모델 학습 파이프라인

## 목적

MLFlow 를 사용한 Iris 데이터셋의 이진 분류 모델 학습 파이프라인입니다.

## 전제

- Python 3.8 이상
- Docker
- MLFlow

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter2_training/iris_binary
```

1. 라이브러리 인스톨
   이 프로그램에서 사용하는 라이브러리는[requirements.txt](./requirements.txt)에 기재되어 있습니다.

```sh
$ make dev
# 실행 커맨드
# pip install -r requirements.txt
# 출력 생략
```

2. 학습용 Docker 이미지 빌드
   학습에 사용할 Docker 이미지를 빌드합니다.

```sh
$ make d_build
# 실행 커맨드
# docker build \
#     -t shibui/ml-system-in-actions:training_pattern_iris_binary_0.0.1 \
#     -f Dockerfile .
# 출력 생략
# docker 이미지로 shibui/ml-system-in-actions:training_pattern_iris_binary_0.0.1 이 빌드됩니다.
```

3. 학습 파이프라인 실행
   mlflow 로 학습 파이프라인을 실행합니다.

```sh
$ make train
# 실행 커맨드
# mlflow run . --no-conda
# 출력 예
# 2021/02/11 11:19:57 INFO mlflow.projects.docker: === Building docker image iris_binary:6fa928e ===
# 2021/02/11 11:20:08 INFO mlflow.projects.utils: === Created directory /var/folders/v8/bvkzgn8j1ws6y76t4z5nt6280000gn/T/tmptboh_ho_ for downloading remote URIs passed to arguments of type 'path' ===
# 2021/02/11 11:20:08 INFO mlflow.projects.backend.local: === Running command 'docker run --rm -v ~/book/ml-system-in-actions/chapter2_training/iris_binary/mlruns:/mlflow/tmp/mlruns -v ~/book/ml-system-in-actions/chapter2_training_patterns/iris_binary/mlruns/0/45a2f7c5aebc49519d21f1fe6e2033c7/artifacts:~/book/ml-system-in-actions/chapter2_training_patterns/iris_binary/mlruns/0/45a2f7c5aebc49519d21f1fe6e2033c7/artifacts -e MLFLOW_RUN_ID=45a2f7c5aebc49519d21f1fe6e2033c7 -e MLFLOW_TRACKING_URI=file:///mlflow/tmp/mlruns -e MLFLOW_EXPERIMENT_ID=0 iris_binary:6fa928e python -m iris_train \
#   --test_size 0.3 \
#   --target_iris virginica' in run with ID '45a2f7c5aebc49519d21f1fe6e2033c7' ===
# 2021/02/11 11:20:15 INFO mlflow.projects: === Run (ID '45a2f7c5aebc49519d21f1fe6e2033c7') succeeded ===
```

학습은 수분 이내로 완료됩니다.
