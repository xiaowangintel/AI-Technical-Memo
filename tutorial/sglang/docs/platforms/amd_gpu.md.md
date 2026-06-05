# amd_gpu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/amd_gpu.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document describes how to run SGLang on AMD GPUs. If you encounter issues or have questions, please open an issue. / 该文档围绕 AMD GPUs 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document describes how to run SGLang on AMD GPUs. If you encounter issues or have questions, please open an issue.
**CN:** 本节围绕 Overview 展开，概述了 run, AMD, GPUs, open 等要点，并说明相关配置、流程、示例或限制条件。

### Section: System Configuration
**EN:** When using AMD GPUs (such as MI300X), certain system-level optimizations help ensure stable performance. Here we take MI300X as an example.
**CN:** 本节围绕 System 配置 展开，概述了 MI300X, AMD, Instinct, LLM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Update GRUB Settings
**EN:** In /etc/default/grub, append the following to GRUB_CMDLINE_LINUX: ``text pci=realloc=off iommu=pt ` Afterward, run sudo update-grub` (or your distro’s equivalent) and reboot.
**CN:** 本节围绕 Update GRUB Settings 展开，概述了 GRUB_CMDLINE_LINUX, Afterward, pci, off 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Disable NUMA Auto-Balancing
**EN:** ``bash sudo sh -c 'echo 0 > /proc/sys/kernel/numa_balancing' `` You can automate or verify this change using this helpful script. Again, please go through the entire documentation to confirm your system is using the recommended configuration.
**CN:** 本节围绕 Disable NUMA Auto-Balancing 展开，概述了 Again, ROCm, sudo, echo 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install SGLang
**EN:** You can install SGLang using one of the methods below.
**CN:** 本节围绕 Install SGLang 展开，概述了 one, below, install, methods 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install from Source
**EN:** ``bash # Use the last release branch git clone -b v0.5.9 https://github.com/sgl-project/sglang.git cd sglang # Compile sgl-kernel pip install --upgrade pip cd sgl-kernel python setup_rocm.py install # Install sglang python package along with diffusion support cd ..
**CN:** 本节围绕 Install from Source 展开，概述了 pip, install, Compile, sgl-kernel 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install Using Docker (Recommended)
**EN:** The docker images are available on Docker Hub at lmsysorg/sglang, built from rocm.Dockerfile. The steps below show how to build and use an image.
**CN:** 本节围绕 Install Using Docker (Recommended) 展开，概述了 RDMA, sglang_image, NCCL_IB_GID_INDEX, AMD 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization on AMD GPUs
**EN:** The Quantization documentation has a full compatibility matrix. The short version: FP8, AWQ, MXFP4, W8A8, GPTQ, compressed-tensors, Quark, and **petit_nvfp4** (NVFP4 on ROCm via Petit) all work on AMD.
**CN:** 本节围绕 量化 on AMD GPUs 展开，概述了 FP8, AWQ, NVFP4, MXFP4 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Running DeepSeek-V3
**EN:** The only difference when running DeepSeek-V3 is in how you start the server.
**CN:** 本节围绕 Running DeepSeek-V3 展开，概述了 DeepSeek-V3, Here, host, HF_TOKEN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Running Llama3.1
**EN:** Running Llama3.1 is nearly identical to running DeepSeek-V3.
**CN:** 本节围绕 Running Llama3.1 展开，概述了 host, HF_TOKEN, ipc, env 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Warmup Step
**EN:** When the server displays The server is fired up and ready to roll!, it means the startup is successful.
**CN:** 本节围绕 Warmup Step 展开，概述了 server, roll, fired, ready 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** AMD / **CN:** AMD
- **EN:** MI300X / **CN:** MI300X
- **EN:** host / **CN:** host
- **EN:** Install / **CN:** Install
- **EN:** FP8 / **CN:** FP8
- **EN:** sglang_image / **CN:** sglang_image
- **EN:** AWQ / **CN:** AWQ
- **EN:** RDMA / **CN:** RDMA

## Dependencies / 依赖关系
- `../advanced_features/quantization.md`
- `setup_rocm.py`
