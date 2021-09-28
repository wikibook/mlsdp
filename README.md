# AI 엔지니어를 위한 머신러닝 시스템 디자인 패턴 (가제)

## 개요

- 본 리포지토리는 2021년 5월 쇼에이샤(翔泳社)에서 출판된『AI エンジニアのための機械学習システムデザインパターン』의 샘플 코드입니다.
- 본 리포지토리에서는 머신러닝의 모델 학습, 리리스, 추론기 가동, 운용을 위한 코드 및 실행환경을 사례별로 제공합니다.
- 더욱 상세한 내용은 이 책 또는 mercari/ml-system-design-pattern 을 참조하시기 바랍니다.
  - [AI エンジニアのための機械学習システムデザインパターン](https://www.amazon.co.jp/dp/B08YNMRH4J/)
  - [mercari/ml-system-design-pattern](https://github.com/mercari/ml-system-design-pattern)

![img](./hyoshi.jpg)

## 실행환경

- Python 3.8 이상
- Docker
- Docker-compose
- （일부）Kubernetes 또는 minikube
- （일부）Android Studio

본 리포지토리에서는 프로그램의 실행환경으로 Docker、Docker-compose、（일부）Kubernetes/minikube、（일부）Android Studio 를 사용합니다.
또는, 커맨드라인으로 `kubectl`、`istioctl`을 사용합니다.
각종 미들웨어, 개발환경, 커맨드라인은 하기의 공식 document 를 참고하여 인스톨 해주시기 바랍니다.

- [Docker](https://docs.docker.com/get-docker/)
- [Docker-compose](https://docs.docker.jp/compose/toc.html)
- [Kubernetes クラスター構築](https://kubernetes.io/ja/docs/setup/)
- [minikube](https://kubernetes.io/ja/docs/setup/learning-environment/minikube/)
- [kubectl](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
- [istioctl](https://istio.io/latest/docs/setup/getting-started/)
- [Android Studio](https://developer.android.com/studio/install)

### Python 실행환경

본 리포지토리에서 사용중인 Python 라이브러리는 `pipenv` 로 지정되어 있습니다. 아래 순서대로 pipenv 와 같이 라이브러리도 인스톨 해주시기 바랍니다.
샘플코드는 Python3.8 이상에서 실행을 확인했습니다. 실행환경의 Python 버전이 맞지 않는 경우, [pyenv](https://github.com/pyenv/pyenv) 등으로 실행환경을 조정해 주십시오.

```sh
# Python 버전
$ python -V
# 출력
Python 3.8.5

# pyenv 버전
$ pyenv versions
# 출력
  system
* 3.8.5

# pipenv 를 인스톨하고, 셸을 pipenv venv 로 변경
$ make dev
# 출력 예
# pip install pipenv
# Requirement already satisfied: pipenv in ~/.pyenv/versions/3.8.5/lib/python3.8/site-packages (2020.11.15)
# (중략)
# Requirement already satisfied: six<2,>=1.9.0 in ~/.pyenv/versions/3.8.5/lib/python3.8/site-packages (from virtualenv->pipenv) (1.15.0)
# WARNING: You are using pip version 20.1.1; however, version 21.0.1 is available.
# You should consider upgrading via the '~/.pyenv/versions/3.8.5/bin/python3.8 -m pip install --upgrade pip' command.
# PIPENV_VENV_IN_PROJECT=true pipenv shell
# Creating a virtualenv for this project...
# Pipfile: ~/book/ml-system-in-actions/Pipfile
# Using ~/.pyenv/versions/3.8.5/bin/python3.8 (3.8.5) to create virtualenv...
# ⠧ Creating virtual environment...created virtual environment CPython3.8.5.final.0-64 in 433ms
#   creator CPython3Posix(dest=~/book/ml-system-in-actions/.venv, clear=False, no_vcs_ignore=False, global=False)
#   seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=~/Library/Application Support/virtualenv)
#     added seed packages: pip==21.0.1, setuptools==52.0.0, wheel==0.36.2
#   activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator

# ✔ Successfully created virtual environment!
# Virtualenv location: ~/book/ml-system-in-actions/.venv
# Launching subshell in virtual environment...
#  . ~/book/ml-system-in-actions/.venv/bin/activate
# [21-02-27 10:03:37] your_name@your_namenoMacBook-Pro:~/book/ml-system-in-actions
# $  . ~/book/ml-system-in-actions/.venv/bin/activate
# (ml-system-in-actions) [21-02-27 10:03:37] your_name@your_namenoMacBook-Pro:~/book/ml-system-in-actions

# 의존 라이브러리 인스톨
$ make dev_sync
# 출력 예
# pipenv sync --dev
# Installing dependencies from Pipfile.lock (a2c081)...
#   🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 93/93 — 00:02:36
# All dependencies are now up-to-date!

##################################
####### 개발, 프로그램 실행    #######
##################################


# 개발, 프로그램의 실행이 완료되면, pipenv venv 셸을 종료
$ exit
```

단, 일부 샘플 코드에서는 다른 라이브러리를 사용하는 경우가 있습니다. 해당 샘플 코드 디렉토리의 README 를 참조해 주시기 바랍니다.

## 샘플 코드 목록

이 리포지토리에서 제공하는 프로그램은 하기의 디렉토리에서 실행하는 것을 상정해 개발되었습니다.
각 프로그램을 실행할 때는 목적에 맞는 디렉토리로 이동해 주십시오. 
프로그램의 실행방법은 각 디렉토리의 README 를 참조하기 바랍니다.

.</br>
├── [2장. 모델 만들기](./chapter2_training/)</br>
│   ├── [모델 관리 서비스](./chapter2_training/model_db)</br>
│   ├── [cifar10](./chapter2_training/cifar10)</br>
│   ├── [iris_binary](./chapter2_training/iris_binary)</br>
│   ├── [iris_sklearn_outlier](./chapter2_training/iris_sklearn_outlier)</br>
│   ├── [iris_sklearn_rf](./chapter2_training/iris_sklearn_rf)</br>
│   └──[iris_sklearn_svc](./chapter2_training/iris_sklearn_svc)</br>
├── [3장. 모델 릴리스 하기](./chapter3_release_patterns)</br>
│   ├── [모델-인-이미지 패턴](./chapter3_release_patterns/model_in_image_pattern)</br>
│   └── [모델 로드 패턴](./chapter3_release_patterns/model_load_pattern)</br>
├── [4장. 추론 시스템 만들기](./chapter4_serving_patterns)</br>
│   ├── [web 싱글 패턴](./chapter4_serving_patterns/web_single_pattern)</br>
│   ├── [동기 추론 패턴](./chapter4_serving_patterns/synchronous_pattern)</br>
│   ├── [비동기 추론 패턴](./chapter4_serving_patterns/asynchronous_pattern)</br>
│   ├── [배치 추론 패턴](./chapter4_serving_patterns/batch_pattern)</br>
│   ├── [전처리 추론 패턴](./chapter4_serving_patterns/prep_pred_pattern)</br>
│   ├── [병렬 마이크로서비스 패턴](./chapter4_serving_patterns/horizontal_microservice_pattern)</br>
│   ├── [시간차 추론 패턴](./chapter4_serving_patterns/sync_async_pattern)</br>
│   ├── [추론 캐시 패턴](./chapter4_serving_patterns/prediction_cache_pattern)</br>
│   ├── [데이터 캐시 패턴](./chapter4_serving_patterns/data_cache_pattern)</br>
│   ├── [추론기 템플릿 패턴](./chapter4_serving_patterns/template_pattern)</br>
│   └── [Edge AI 패턴](./chapter4_serving_patterns/edge_ai_pattern)</br>
├── [5장. 머신러닝 시스템의 운용](./chapter5_operations)</br>
│   ├── [추론 로그 패턴](./chapter5_operations/prediction_log_pattern)</br>
│   └── [추론 감시 패턴](./chapter5_operations/prediction_monitoring_pattern)</br>
└── [6장. 머신러닝 시스템의 품질관리](./chapter6_operation_management)</br>
   ├── [부하 테스트 패턴](./chapter6_operation_management/load_test_pattern)</br>
   ├── [추론 서킷브레이커 패턴](./chapter6_operation_management/circuit_breaker_pattern)</br>
   ├── [섀도우 A/B 테스트 패턴](./chapter6_operation_management/shadow_ab_pattern)</br>
   ├── [온라인 A/B 테스트 패턴](./chapter6_operation_management/online_ab_pattern)</br>
   ├── [파라미터 기반 추론 패턴](./chapter6_operation_management/paramater_based_pattern)</br>
   └── [조건 분기 추론 패턴](./chapter6_operation_management/condition_based_pattern)</br>
