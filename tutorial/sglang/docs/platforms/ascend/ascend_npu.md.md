# ascend_npu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_npu.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: You can install SGLang using any of the methods below. Please go through System Settings section to ensure the clusters are roaring at max performance. / 该文档围绕 SGLang 安装 with NPUs support 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** You can install SGLang using any of the methods below. Please go through System Settings section to ensure the clusters are roaring at max performance.
**CN:** 本节围绕 Overview 展开，概述了 Feel, Please, System Settings, max 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Component Version Mapping For SGLang
**EN:** This section provides a comparison table for Component Version Mapping For SGLang, covering columns such as Component, Version, Obtain Way and examples such as HDK, CANN, Pytorch Adapter, MemFabric.
**CN:** 本节围绕 Component Version Mapping For SGLang 展开，概述了 link, HDK, pip, CANN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Obtain CANN Image
**EN:** You can obtain the dependency of a specified version of CANN through an image.
**CN:** 本节围绕 Obtain CANN Image 展开，概述了 Atlas, Ubuntu, CANN, pull 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 1: Installing from source with prerequisites
**EN:** #### Python Version Only python==3.11 is supported currently. If you don't want to break system pre-installed python, try installing with conda.
**CN:** 本节围绕 Method 1: Installing from source with prerequisites 展开，概述了 Ascend, install, pip, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method 2: Using Docker Image
**EN:** #### Obtain Image You can download the SGLang image or build an image based on Dockerfile to obtain the Ascend NPU image.
**CN:** 本节围绕 Method 2: Using Docker Image 展开，概述了 device, --device, Atlas, image 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CPU performance power scheme
**EN:** The default power scheme on Ascend hardware is ondemand which could affect performance, changing it to performance is recommended.
**CN:** 本节围绕 CPU 性能 power scheme 展开，概述了 performance, Make, Ascend, ondemand 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Disable NUMA balancing
**EN:** ``shell sudo sysctl -w kernel.numa_balancing=0 # Check cat /proc/sys/kernel/numa_balancing # shows 0 ``
**CN:** 本节围绕 Disable NUMA balancing 展开，概述了 Check, cat, sudo, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prevent swapping out system memory
**EN:** ``shell sudo sysctl -w vm.swappiness=10 # Check cat /proc/sys/vm/swappiness # shows 10 ``
**CN:** 本节围绕 Prevent swapping out system memory 展开，概述了 Check, cat, sudo, shell 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Running Service For Large Language Models
**EN:** #### PD Mixed Scene ``shell # Enabling CPU Affinity export SGLANG_SET_CPU_AFFINITY=1 python3 -m sglang.launch_server --model-path meta-llama/Llama-3.1-8B-Instruct --attention-backend ascend ` #### PD Disaggregation Scene 1.
**CN:** 本节围绕 Running Service For Large Language 模型s 展开，概述了 PIP, PORT, export, ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Running Service For Multimodal Language Models
**EN:** This content focuses on Running Service For Multimodal Language Models and highlights ascend, Qwen3-VL-30B-A3B-Instruct, npu, --tp.
**CN:** 本节围绕 Running Service For Multimodal Language 模型s 展开，概述了 ascend, Qwen3-VL-30B-A3B-Instruct, npu, --tp 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Ascend / **CN:** Ascend
- **EN:** device / **CN:** device
- **EN:** --device / **CN:** --device
- **EN:** shell / **CN:** shell
- **EN:** Atlas / **CN:** Atlas
- **EN:** install / **CN:** install
- **EN:** PIP / **CN:** PIP
- **EN:** PORT / **CN:** PORT

## Dependencies / 依赖关系
- `//github.com/Ascend/pytorch/blob/master/README.md`
- `//gitcode.com/Ascend/triton-ascend/blob/master/docs/sources/getting-started/installation.md`
- `//github.com/sgl-project/sgl-kernel-npu/blob/main/python/sgl_kernel_npu/README.md`
- `//github.com/sgl-project/sgl-kernel-npu/blob/main/python/deep_ep/README.md`
