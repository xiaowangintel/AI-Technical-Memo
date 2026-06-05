# installation.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/installation.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: You can install SGLang-Diffusion using one of the methods below. The standard installation already includes SGLang's optimized kernel stack, including both sgl-kernel and JIT kernels used by diffusion workloads. / 该文档围绕 Install SGLang-Diffusion 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** You can install SGLang-Diffusion using one of the methods below. The standard installation already includes SGLang's optimized kernel stack, including both sgl-kernel and JIT kernels used by diffusion workloads.
**CN:** 本节围绕 Overview 展开，概述了 JIT, sgl-kernel, SGLang-Diffusion, one 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 1: With pip or uv
**EN:** It is recommended to use uv for a faster installation: ``bash pip install --upgrade pip pip install uv uv pip install "sglang[diffusion]" --prerelease=allow ``
**CN:** 本节围绕 Method 1: With pip or uv 展开，概述了 pip, install, allow, faster 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 2: From source
**EN:** ``bash # Use the latest release branch git clone https://github.com/sgl-project/sglang.git cd sglang # Install the Python packages pip install --upgrade pip pip install -e "python[diffusion]" # With uv uv pip install -e "python[diffusion]" --prerelease=allow ``
**CN:** 本节围绕 Method 2: From source 展开，概述了 pip, install, diffusion, git 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 3: Using Docker
**EN:** The Docker images are available on Docker Hub at lmsysorg/sglang, built from the Dockerfile. Replace <secret> below with your HuggingFace Hub token.
**CN:** 本节围绕 Method 3: Using Docker 展开，概述了 Dockerfile, Hub, echo, Docker 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform-Specific: ROCm (AMD GPUs)
**EN:** This content focuses on Platform-Specific: ROCm (AMD GPUs) and highlights AMD, ROCm, GPUs, MI300X.
**CN:** 本节围绕 Platform-Specific: ROCm (AMD GPUs) 展开，概述了 AMD, ROCm, GPUs, MI300X 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform-Specific: MUSA (Moore Threads GPUs)
**EN:** This content focuses on Platform-Specific: MUSA (Moore Threads GPUs) and highlights pip, install, MUSA, MTGPU.
**CN:** 本节围绕 Platform-Specific: MUSA (Moore Threads GPUs) 展开，概述了 pip, install, MUSA, MTGPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform-Specific: Intel XPU
**EN:** For Intel Data Center GPU Max or Arc GPUs, follow the XPU installation guide to set up the base environment, then install diffusion dependencies: ``bash pip install -e "python[diffusion]" ``
**CN:** 本节围绕 Platform-Specific: Intel XPU 展开，概述了 XPU, install, diffusion, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform-Specific: Ascend NPU
**EN:** For Ascend NPU, please follow the NPU installation guide. Quick test: ``bash sglang generate --model-path black-forest-labs/FLUX.1-dev \ --prompt "A logo With Bold Large text: SGL Diffusion" \ --save-output ``
**CN:** 本节围绕 Platform-Specific: Ascend NPU 展开，概述了 NPU, Quick, SGL, test 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform-Specific: Apple MPS
**EN:** This content focuses on Platform-Specific: Apple MPS and highlights Install, pip, brew, Clone.
**CN:** 本节围绕 Platform-Specific: Apple MPS 展开，概述了 Install, pip, brew, Clone 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** install / **CN:** install
- **EN:** pip / **CN:** pip
- **EN:** Platform-Specific / **CN:** Platform-Specific
- **EN:** diffusion / **CN:** diffusion
- **EN:** GPUs / **CN:** GPUs
- **EN:** Method / **CN:** Method
- **EN:** source / **CN:** source
- **EN:** installation / **CN:** 安装

## Dependencies / 依赖关系
- `../platforms/amd_gpu.md`
- `../platforms/xpu.md`
- `../platforms/ascend/ascend_npu.md`
