+++
title = "Google Colaboratory (Colab) 에서 vLLM 설치시 torchaudio 충돌 해결법"
date = "2024-10-31T10:07:09Z"
draft = false

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."
description = "Colab 에서 vLLM 설치 시에 torchaudio 패키지 충돌 원인에 대하여 파악 및 해결 방법에 대하여 공유"

tags = ["vllm", "google", "colab", "llm"]
+++

Google Colab 에서 `vLLM` 패키지 설치 중에 `torchvision` 패키지가 충돌이 나서
해당 원인과 해결 방법에 대하여 공유하고자 한다.

## TLDR (간단요약)

원인은 기존에 설치된 `torch`, `torchaudio` 및 `torchvison` 패키지의 조합이 `2.5.x` 에 맞게 되어 있어서
그런 것으로, vLLM 설치 시에 `torchaudio` 도 2.4.0 으로 재설치 되도록 추가하면 정상적으로 설치할
수 있다.

```bash
pip install vllm torchaudio==2.4.0
```

## 설치 중 마주한 문제

공식 문서에서 설명하는 설치 가이드에 따라서, 아래와 같은 명령어를 실행 후 기다렸다.

```bash
pip install vllm
```

그리고 아래와 같은 에러가 표출되면서 정상적으로 설치되지 않았다.

```
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed.
This behaviour is the source of the following dependency conflicts.
torchaudio 2.5.0+cu121 requires torch==2.5.0, but you have torch 2.4.0 which is incompatible.
```

대충 위의 에러를 보면, 대충 `torch` 와 연관되어 있는 패키지의 버전이 충돌이 나는 것 같다.
Python 은 이게 문제인데, `vLLM` 이 어떤 패키지를 필요로 하는지부터 차근차근히 살펴보기로 했다.

### 원인 파악 1 - vLLM 의 Requirements

`vLLM` 의 `requirements.txt` 를 찾아보니, 기본적으로 사용하는 `requirements-common.txt` 파일과
GPU 버전인 경우 필요로 하는 `requirements-cuda.txt` 가 있다.

아래는 `requirements-common.txt` 파일의 내용이다.

```
psutil
sentencepiece  # Required for LLaMA tokenizer.
numpy < 2.0.0
requests >= 2.26.0
tqdm
py-cpuinfo
transformers >= 4.45.2  # Required for Llama 3.2 and Qwen2-VL.
tokenizers >= 0.19.1  # Required for Llama 3.
protobuf # Required by LlamaTokenizer.
fastapi >= 0.107.0, < 0.113.0; python_version < '3.9'
fastapi >= 0.107.0, != 0.113.*, != 0.114.0; python_version >= '3.9'
aiohttp
openai >= 1.40.0 # Ensure modern openai package (ensure types module present)
uvicorn[standard]
pydantic >= 2.9  # Required for fastapi >= 0.113.0
pillow  # Required for image processing
prometheus_client >= 0.18.0
prometheus-fastapi-instrumentator >= 7.0.0
tiktoken >= 0.6.0  # Required for DBRX tokenizer
lm-format-enforcer == 0.10.6
outlines >= 0.0.43, < 0.1
typing_extensions >= 4.10
filelock >= 3.10.4 # filelock starts to support `mode` argument from 3.10.4
partial-json-parser # used for parsing partial JSON outputs
pyzmq
msgspec
gguf == 0.10.0
importlib_metadata
mistral_common[opencv] >= 1.4.4
pyyaml
six>=1.16.0; python_version > '3.11' # transitive dependency of pandas that needs to be the latest version for python 3.12
setuptools>=74.1.1; python_version > '3.11' # Setuptools is used by triton, we need to ensure a modern version is installed for 3.12+ so that it does not try to import distutils, which was removed in 3.12
einops # Required for Qwen2-VL.
compressed-tensors == 0.6.0 # required for compressed-tensors
```

그리고, `requirements-cuda.txt` 파일은 다음과 같고, 위에서 참고한 `requirements-common.txt` 파일을 상속하여 설치하는 것을
확인할 수 있었으며, Pytorch `2.4.0` 버전을 필요로 하는 것 까지 확인할 수 있었다.

```
# Common dependencies
-r requirements-common.txt

# Dependencies for NVIDIA GPUs
ray >= 2.9
nvidia-ml-py # for pynvml package
torch == 2.4.0
# These must be updated alongside torch
torchvision == 0.19   # Required for phi3v processor. See https://github.com/pytorch/vision?tab=readme-ov-file#installation for corresponding version
xformers == 0.0.27.post2; platform_system == 'Linux' and platform_machine == 'x86_64'  # Requires PyTorch 2.4.0
```

### 원인 파악 2 - Pytorch 의 Requirements

[원인 파악 1](#원인-파악-1-vllm-은-무엇을-필요로-하는가) 에서 우리는 torch (Pytorch) `2.4.0` 버전을
요구하는 것을 확인하였다.

로그를 살펴보니 `torch-2.4.0-cp310-cp310-manylinux1_x86_64.whl` 파일을 다운로드하는 것을 확인할 수 있었고
PyPI Package Index 에서 해당 파일을 다운로드하여 확인하였다.

* 파일 리스트 Index 링크: https://pypi.org/project/torch/2.4.0/#files

참고로, `whl` (Wheel) 파일은 `zip` archive 의 형태와 같기 때문에, 파일 확장자를 `whl` 에서 `zip` 파일로 바꾸면
해당 패키지 파일의 내용을 확인할 수 있다.

압축파일을 열면, 아래와 같이 4개의 디렉토리가 리스팅되는 것을 확인할 수 있다.

```
functorch
torch
torch-2.4.0.dist-info
torchgen
```

[Python Packaging User Guide](https://packaging.python.org/en/latest) 의
[Recording installed projects](https://packaging.python.org/en/latest/specifications/recording-installed-packages/)
문서를 보면, `*.dist-info` 디렉토리에 해당 패키지에 관한 메타데이터 정보가 포함되어 있다고 한다.

해당 디렉토리에 뭐가 들어가는지 찬한히 찾아보니, `METADATA` 파일에 Requirements 가 있을 것 같아서
[Core metadata specifications](https://packaging.python.org/en/latest/specifications/core-metadata/)
문서를 읽어보었더니, `Requires-Dist` 에 필요한 외부 패키지를 추가할 수 있다고 한다.

METADATA 파일을 열어보면 `Requires-Dist` 가 포함되어 있는 것을 확인할 수 있었다.

```
Requires-Dist: filelock
Requires-Dist: typing-extensions (>=4.8.0)
Requires-Dist: sympy
Requires-Dist: networkx
Requires-Dist: jinja2
Requires-Dist: fsspec
Requires-Dist: setuptools
Requires-Dist: nvidia-cuda-nvrtc-cu12 (==12.1.105) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cuda-runtime-cu12 (==12.1.105) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cuda-cupti-cu12 (==12.1.105) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cudnn-cu12 (==9.1.0.70) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cublas-cu12 (==12.1.3.1) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cufft-cu12 (==11.0.2.54) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-curand-cu12 (==10.3.2.106) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cusolver-cu12 (==11.4.5.107) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-cusparse-cu12 (==12.1.0.106) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-nccl-cu12 (==2.20.5) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: nvidia-nvtx-cu12 (==12.1.105) ; platform_system == "Linux" and platform_machine == "x86_64"
Requires-Dist: triton (==3.0.0) ; platform_system == "Linux" and platform_machine == "x86_64" and python_version < "3.13"
Provides-Extra: opt-einsum
Requires-Dist: opt-einsum (>=3.3) ; extra == 'opt-einsum'
Provides-Extra: optree
Requires-Dist: optree (>=0.11.0) ; extra == 'optree'
```

뭔가 이상은 없고, `torch` 를 설치하면 **"CUDA Toolkit 을 설치할 필요가 없다"**는 것은 잘 확인할 수 있었다.

### 원인 파악 3 - Colab 에 설치되어 있는 패키지 버전은?

아래의 명령어를 사용하여 Colab 에 설치되어 있는 `torch` 와 `nvidia` 패키지의 버전을 확인할 수 있다.
(Jupyter 커널에서 실행하려면 앞에 `!` 을 추가하여 실행)

```bash
pip list | grep -E 'torch|nvidia'
```

```
nvidia-cublas-cu12                 12.6.3.3
nvidia-cuda-cupti-cu12             12.6.80
nvidia-cuda-nvcc-cu12              12.6.77
nvidia-cuda-runtime-cu12           12.6.77
nvidia-cudnn-cu12                  9.5.1.17
nvidia-cufft-cu12                  11.3.0.4
nvidia-curand-cu12                 10.3.7.77
nvidia-cusolver-cu12               11.7.1.2
nvidia-cusparse-cu12               12.5.4.2
nvidia-nccl-cu12                   2.23.4
nvidia-nvjitlink-cu12              12.6.77
torch                              2.5.0+cu121
torchaudio                         2.5.0+cu121
torchsummary                       1.5.1
torchvision                        0.20.0+cu121
```

### 원인 파악 4 - pytorch 의 조합 문제는 아닐까?

Pytorch 는 설치 시에 `torch`, `torchaudio` 및 `torchvision` 패키지들에 대해서
버전 조합을 제공하고 있으며, `2.4.0` 버전 같은 경우에는 아래와 같이
설치하도록 하고 있다.

* 참고 링크: https://pytorch.org/get-started/previous-versions/

```bash
pip install torch==2.4.0 torchvision==0.19.0 torchaudio==2.4.0 --index-url https://download.pytorch.org/whl/cu121
```

여기까지 오면 느낌상 `vLLM` 에서 `torchaudio` 를 포함하지 않거나 가이드하지 않아서
생긴 것으로 파악되고, 간단히 `torchaudio` 의 버전만 2.4.0 으로 명시하면 될 것 같은
느낌이 들었다.

시도해 보자.

### 시도 1 - pip 설치 시에 torchaudio 버전도 같이 명시해보기

`vLLM` 설치 시에 `torchaudio` 도 `torch` 의 `2.4.0` 버전에 맞도록 명시하여 설치해보면
될 것 같아서 아래와 같이 추가하여 시도해보았다.

```
pip install vllm torchaudio==2.4.0
```

해보니 `torchaudio` 도 `torchvision` 과 같이 기존의 설치되어 있는 패키지 삭제 후
재설치 되었고, 정상적으로 설치되는 것을 확인할 수 있었다.

```
...

Attempting uninstall: torchvision
    Found existing installation: torchvision 0.20.0+cu121
    Uninstalling torchvision-0.20.0+cu121:
      Successfully uninstalled torchvision-0.20.0+cu121
Attempting uninstall: torchaudio
    Found existing installation: torchaudio 2.5.0+cu121
    Uninstalling torchaudio-2.5.0+cu121:
      Successfully uninstalled torchaudio-2.5.0+cu121
Successfully installed compressed-tensors-0.6.0 datasets-3.0.2 dill-0.3.8 diskcache-5.6.3 fastapi-0.115.4 gguf-0.10.0 httptools-0.6.4 interegular-0.3.3 lark-1.2.2 lm-format-enforcer-0.10.6 mistral-common-1.4.4 msgspec-0.18.6 multiprocess-0.70.16 nvidia-cublas-cu12-12.1.3.1 nvidia-cuda-cupti-cu12-12.1.105 nvidia-cuda-nvrtc-cu12-12.1.105 nvidia-cuda-runtime-cu12-12.1.105 nvidia-cudnn-cu12-9.1.0.70 nvidia-cufft-cu12-11.0.2.54 nvidia-curand-cu12-10.3.2.106 nvidia-cusolver-cu12-11.4.5.107 nvidia-cusparse-cu12-12.1.0.106 nvidia-ml-py-12.560.30 nvidia-nccl-cu12-2.20.5 nvidia-nvtx-cu12-12.1.105 outlines-0.0.46 partial-json-parser-0.2.1.1.post4 prometheus-fastapi-instrumentator-7.0.0 pyairports-2.1.1 pycountry-24.6.1 python-dotenv-1.0.1 ray-2.38.0 starlette-0.41.2 tiktoken-0.7.0 tokenizers-0.20.1 torch-2.4.0 torchaudio-2.4.0 torchvision-0.19.0 transformers-4.46.1 triton-3.0.0 uvicorn-0.32.0 uvloop-0.21.0 vllm-0.6.3.post1 watchfiles-0.24.0 websockets-13.1 xformers-0.0.27.post2 xxhash-3.5.0
```

이후 다음의 명령어를 통해서 `vLLM` 의 패키지를 import 해보면 정상적으로 되는 것을
확인할 수 있었다.

```python3
from vllm import LLM, SamplingParams
```

다만, CPU Instance 에서 실행하는 경우에는 아래와 같이 `libcuda.so.1` 이 없다는 WARNING 이
나올 수는 있으나 문제는 아니니 무시해도 괜찮다.

```
WARNING 10-31 11:40:14 _custom_ops.py:19] Failed to import from vllm._C with ImportError('libcuda.so.1: cannot open shared object file: No such file or directory')
```

## 결론

~~Python 은 이런 맛으로 쓰는 것이다.~~

추가로, Torch 에서 CUDA Toolkit 과 관련된 패키지를 함께 설치해주기 때문에,
혹여나 Containerize 시에 CUDA Toolkit 을 함께 포함하지 않기를 권장한다.

끝.
