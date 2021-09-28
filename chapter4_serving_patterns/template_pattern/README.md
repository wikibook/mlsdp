# 추론기 템플릿 패턴

## 목적

추론기에 필요한 리소스를 디렉토리 구성과 jinja2 템플릿으로 제공합니다.

## 전제

- Python 3.8 이상

## 사용법

0. 현재 디렉토리

```sh
$ pwd
~/ml-system-in-actions/chapter4_serving_patterns/template_pattern
```

1. 라이브러리 인스톨

```sh
$ pip install -r requirements.txt
```

2. 템플릿으로 샘플 추론기를 빌드

추론기 템플릿은 다음의 디렉토리, 파일 구성으로 되어있습니다.

- template: 템플릿 디렉토리 
- template_files: 변수의 변환이 필요한 파일들
- correspond_file_path.yaml: template_files 의 파일의 변환 후에 배치될 경로를 지정 
- vars.yaml: 변수
- builder.py: 템플릿으로 추론기 프로젝트를 작성하는 스크립트

```sh
# 템플릿 파일 목록과 변환 후의 파일 경로
$ cat correspond_file_path.yaml
# Dockerfile.j2: "{}/Dockerfile"
# prediction.py.j2: "{}/src/ml/prediction.py"
# routers.py.j2: "{}/src/app/routers/routers.py"
# deployment.yml.j2: "{}/manifests/deployment.yml"
# namespace.yml.j2: "{}/manifests/namespace.yml"
# makefile.j2: "{}/makefile"
# docker-compose.yml.j2: "{}/docker-compose.yml"

# 변수
$ cat vars.yaml
# name: sample
# model_file_name: iris_svc.onnx
# label_file_name: label.json
# data_type: float32
# data_structure: (1,4)
# data_sample: [[5.1, 3.5, 1.4, 0.2]]
# prediction_type: float32
# prediction_structure: (1,3)
# prediction_sample: [0.97093159, 0.01558308, 0.01348537]

# 추론기 프로젝트 빌드
$ python \
    -m builder \
    --name sample \
    --variable_file vars.yaml \
    --correspond_file_path correspond_file_path.yaml

# 이것으로 ./sample 디렉토리 이하 추론기 프로그램이 완성됩니다.
```

3. 샘플 추론기 프로젝트 확인

작성된 프로그램은 다음과 같은 구성으로 되어있습니다.
모델과 라벨 파일을 `sample/models/` 디렉토리에 배치하면 완성입니다.

```sh
$ tree sample
# sample
# ├── Dockerfile
# ├── __init__.py
# ├── docker-compose.yml
# ├── makefile
# ├── manifests
# │   ├── deployment.yml
# │   └── namespace.yml
# ├── models
# ├── requirements.txt
# ├── run.sh
# └── src
#     ├── __init__.py
#     ├── app
#     │   ├── __init__.py
#     │   ├── app.py
#     │   └── routers
#     │       ├── __init__.py
#     │       └── routers.py
#     ├── configurations.py
#     ├── constants.py
#     ├── ml
#     │   ├── __init__.py
#     │   └── prediction.py
#     └── utils
#         ├── __init__.py
#         ├── logging.conf
#         └── profiler.py
```
