---
title: NVIDIA Jetson Ultralytics YOLO 설정
description: Jetson Ultralytics YOLO 환경 설정 및 설치
date: 2025-10-31 15:40:00 -0400
categories: [NVIDIA]
tags: [nvidia, jetson, YOLO, Ultralytics]     # TAG names should always be lowercase
---

# Ultralytics 설치

[ultralytics](https://docs.ultralytics.com/ko/guides/nvidia-jetson/)에 가이드가 있습니다.

Jetpack6.2가 설치되어 있으면 아래의 방법을 사용하면 됩니다.

```bash
sudo apt update
sudo apt install python3-pip -y
pip install -U pip
pip install ultralytics
sudo reboot
```

공식 가이드에 있는 ```pip install ultralytics[export]```를 실행하면
```
# pip install ultralytics[export] 실행

error: resolution-too-deep

× Dependency resolution exceeded maximum depth
╰─> Pip cannot resolve the current dependencies as the dependency graph is too complex for pip to solve efficiently.

hint: Try adding lower bounds to constrain your dependencies, for example: 'package>=2.0.0' instead of just 'package'.
```
해당 에러가 발생해서 'ultralytics'으로 설치했습니다.

# torch 설치

저는 ```nvidia-smi```로 확인했을 때  "CUDA Version: 12.6"가 나타나서
아래의 torch 버전을 다운로드 받았습니다.

```bash
python -m pip install --index-url https://pypi.jetson-ai-lab.io/jp6/cu126 torch==2.8.0 torchvision==0.23.0
```

위와 같이 실행하면
```bash
Failed to initialize NumPy:
A module that was compiled using NumPy 1.x cannot be run in
NumPy 2.2.6 as it may crash. To support both 1.x and 2.x
versions of NumPy, modules must be compiled with NumPy 2.0.
Some module may need to rebuild instead e.g. with 'pybind11>=2.12'.

If you are a user of the module, the easiest solution will be to
downgrade to 'numpy<2' or try to upgrade the affected module.
We expect that some modules will need time to support NumPy 2.
```

경고가 발생합니다. numpy 버전을 낮춰줘야합니다.

```bash
pip uninstall -y numpy
pip install --no-cache-dir "numpy==1.26.4"
```

결과로
```bash
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
opencv-python 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
```
에러가 발생하지만 문제없이 잘 실행 되었습니다.

추가적으로 공식 가이드에서 Docker를 사용하는 방법도 나와있습니다.
저는 venv 가상환경을 사용해서 ultralytics와 torch를 설치했습니다.
