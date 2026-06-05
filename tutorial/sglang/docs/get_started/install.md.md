# install.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/get_started/install.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: You can install SGLang using one of the methods below. This page primarily applies to common NVIDIA GPU platforms. / 该文档围绕 Install SGLang 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** You can install SGLang using one of the methods below. This page primarily applies to common NVIDIA GPU platforms.
**CN:** 本节围绕 Overview 展开，概述了 NVIDIA, TPU, Intel, platforms 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 1: With pip or uv
**EN:** It is recommended to use uv for faster installation: ``bash pip install --upgrade pip pip install uv uv pip install sglang ``
**CN:** 本节围绕 Method 1: With pip or uv 展开，概述了 pip, install, faster, upgrade 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For CUDA 13
**EN:** Docker is recommended (see Method 3 note on B300/GB300/CUDA 13). If you do not have Docker access, follow these steps: 1.
**CN:** 本节围绕 For CUDA 13 展开，概述了 X.Y.Z, install, pip, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: **Quick fixes to common problems**
**EN:** If you encounter OSError: CUDA_HOME environment variable is not set. Please set it to your CUDA install root with either of the following solutions: 1.
**CN:** 本节围绕 **Quick fixes to common problems** 展开，概述了 CUDA_HOME, FlashInfer, CUDA, Please 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 2: From source
**EN:** ``bash # Use the last release branch git clone -b v0.5.9 https://github.com/sgl-project/sglang.git cd sglang # Install the python packages pip install --upgrade pip pip install -e "python" ` **Quick fixes to common problems** - If you want to develop SGLang, you can try the dev docker image.
**CN:** 本节围绕 Method 2: From source 展开，概述了 pip, docker, Quick, image 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 3: Using docker
**EN:** The docker images are available on Docker Hub at lmsysorg/sglang, built from Dockerfile. Replace <secret> below with your huggingface hub token.
**CN:** 本节围绕 Method 3: Using docker 展开，概述了 docker, host, image, HF_TOKEN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 4: Using Kubernetes
**EN:** Please check out OME, a Kubernetes operator for enterprise-grade management and serving of large language models (LLMs). <details> <summary>More</summary> 1.
**CN:** 本节围绕 Method 4: Using Kubernetes 展开，概述了 Option, serving, node, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 5: Using docker compose
**EN:** <details> <summary>More</summary> > This method is recommended if you plan to serve it as a service. > A better approach is to use the k8s-sglang-service.yaml.
**CN:** 本节围绕 Method 5: Using docker compose 展开，概述了 Copy, Execute, details, summary 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 6: Run on Kubernetes or Clouds with SkyPilot
**EN:** <details> <summary>More</summary> To deploy on Kubernetes or 12+ clouds, you can use SkyPilot. Install SkyPilot and set up Kubernetes cluster or cloud access: see SkyPilot's documentation.
**CN:** 本节围绕 Method 6: Run on Kubernetes or Clouds with SkyPilot 展开，概述了 SkyPilot, HF_TOKEN, Kubernetes, cloud 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 7: Run on AWS SageMaker
**EN:** <details> <summary>More</summary> To deploy on SGLang on AWS SageMaker, check out AWS SageMaker Inference Amazon Web Services provide supports for SGLang containers along with routine security patching.
**CN:** 本节围绕 Method 7: Run on AWS SageMaker 展开，概述了 IMAGE_URI, AWS, Build, SageMaker 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common Notes
**EN:** FlashInfer is the default attention kernel backend. It only supports sm75 and above.
**CN:** 本节围绕 Common Notes 展开，概述了 A10, A100, L40S, H100 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Method / **CN:** Method
- **EN:** docker / **CN:** docker
- **EN:** install / **CN:** install
- **EN:** pip / **CN:** pip
- **EN:** details / **CN:** details
- **EN:** summary / **CN:** summary
- **EN:** Kubernetes / **CN:** Kubernetes
- **EN:** CUDA / **CN:** CUDA

## Dependencies / 依赖关系
- `../platforms/amd_gpu.md`
- `../platforms/cpu_server.md`
- `../platforms/tpu.md`
- `../platforms/nvidia_jetson.md`
- `../platforms/ascend/ascend_npu.md`
- `../platforms/xpu.md`
- `../developer_guide/development_guide_using_docker.md`
- `kubectl apply -f docker/k8s-sglang-service.yaml`
- `kubectl apply -f docker/k8s-sglang-distributed-sts.yaml`
- `docker/k8s-sglang-service.yaml`
- `docker/k8s-sglang-distributed-sts.yaml`
- `k8s-sglang-service.yaml`
