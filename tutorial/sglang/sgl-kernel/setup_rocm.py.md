# setup_rocm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/setup_rocm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This build/setup script prepares the `ROCm build` path, checks prerequisites, and compiles or installs native pieces when required. / 该构建/安装脚本用于准备 `ROCm build` 路径，检查前置条件，并在需要时编译或安装原生组件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Comments and local context
````python
# Copyright 2025 SGLang Team. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 16-23: Imports and module setup
````python
import os
import platform
import sys
from pathlib import Path

import torch
from setuptools import find_packages, setup
from torch.utils.cpp_extension import BuildExtension, CUDAExtension
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 25-26: Constants and configuration
````python
root = Path(__file__).parent.resolve()
arch = platform.machine().lower()
````
**EN:** This block defines shared constants or configuration values such as `root`, `arch`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `root`, `arch`），供后续函数或控制流程复用。

### Lines 29-33: `_get_version` definition
````python
def _get_version():
    with open(root / "pyproject.toml") as f:
        for line in f:
            if line.startswith("version"):
                return line.split("=")[1].strip().strip('"')
````
**EN:** This section defines `_get_version` and implements the core logic associated with get version.
**CN:** 该部分定义 `_get_version`，并实现与 get version 相关的核心逻辑。

### Lines 36-65: Constants and configuration
````python
operator_namespace = "sgl_kernel"
include_dirs = [
    root / "include",
    root / "include" / "impl",
    root / "csrc",
]

sources = [
    "csrc/allreduce/custom_all_reduce.hip",
    "csrc/allreduce/deterministic_all_reduce.hip",
    "csrc/allreduce/quick_all_reduce.cu",
    "csrc/common_extension_rocm.cc",
    "csrc/elementwise/activation.cu",
    "csrc/elementwise/topk.cu",
    "csrc/grammar/apply_token_bitmask_inplace_cuda.cu",
    "csrc/moe/moe_align_kernel.cu",
    "csrc/moe/moe_topk_softmax_kernels.cu",
    "csrc/moe/moe_topk_sigmoid_kernels.cu",
    "csrc/speculative/eagle_utils.cu",
    "csrc/kvcacheio/transfer.cu",
    "csrc/memory/weak_ref_tensor.cpp",
    "csrc/elementwise/pos_enc.cu",
]

cxx_flags = ["-O3"]
libraries = ["hiprtc", "amdhip64", "c10", "torch", "torch_python"]
extra_link_args = ["-Wl,-rpath,$ORIGIN/../../torch/lib", f"-L/usr/lib/{arch}-linux-gnu"]

default_target = "gfx942"
amdgpu_target = os.environ.get("AMDGPU_TARGET", default_target)
````
**EN:** This block defines shared constants or configuration values such as `operator_namespace`, `include_dirs`, `sources`, `cxx_flags`, `libraries`, `extra_link_args`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `operator_namespace`, `include_dirs`, `sources`, `cxx_flags`, `libraries`, `extra_link_args`），供后续函数或控制流程复用。

### Lines 67-73: Conditional logic
````python
if torch.cuda.is_available():
    try:
        amdgpu_target = torch.cuda.get_device_properties(0).gcnArchName.split(":")[0]
    except Exception as e:
        print(f"Warning: Failed to detect GPU properties: {e}")
else:
    print(f"Warning: torch.cuda not available. Using default target: {amdgpu_target}")
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 75-79: Conditional logic
````python
if amdgpu_target not in ["gfx942", "gfx950"]:
    print(
        f"Warning: Unsupported GPU architecture detected '{amdgpu_target}'. Expected 'gfx942' or 'gfx950'."
    )
    sys.exit(1)
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 81-83: Constants and configuration
````python
fp8_macro = (
    "-DHIP_FP8_TYPE_FNUZ" if amdgpu_target == "gfx942" else "-DHIP_FP8_TYPE_E4M3"
)
````
**EN:** This block defines shared constants or configuration values such as `fp8_macro`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `fp8_macro`），供后续函数或控制流程复用。

### Lines 84-88: Comments and local context
````python

# Dynamic shared-memory budget for the TopK kernels.
# - gfx942 (MI300/MI325): LDS is typically 64KB per workgroup -> keep dynamic smem <= ~48KB
#   (leaves room for static shared allocations in the kernel).
# - gfx95x (MI350): LDS is larger (e.g. 160KB per CU) -> allow the original 128KB dynamic smem.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 89-118: Constants and configuration
````python
topk_dynamic_smem_bytes = 48 * 1024 if amdgpu_target == "gfx942" else 32 * 1024 * 4

hipcc_flags = [
    "-DNDEBUG",
    f"-DOPERATOR_NAMESPACE={operator_namespace}",
    "-O3",
    "-Xcompiler",
    "-fPIC",
    "-std=c++17",
    f"--amdgpu-target={amdgpu_target}",
    "-DENABLE_BF16",
    "-DENABLE_FP8",
    fp8_macro,
    f"-DSGL_TOPK_DYNAMIC_SMEM_BYTES={topk_dynamic_smem_bytes}",
]

ext_modules = [
    CUDAExtension(
        name="sgl_kernel.common_ops",
        sources=sources,
        include_dirs=include_dirs,
        extra_compile_args={
            "nvcc": hipcc_flags,
            "cxx": cxx_flags,
        },
        libraries=libraries,
        extra_link_args=extra_link_args,
        py_limited_api=False,
    ),
]
````
**EN:** This block defines shared constants or configuration values such as `topk_dynamic_smem_bytes`, `hipcc_flags`, `ext_modules`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `topk_dynamic_smem_bytes`, `hipcc_flags`, `ext_modules`），供后续函数或控制流程复用。

### Lines 120-128: Top-level logic
````python
setup(
    name="sglang-kernel",
    version=_get_version(),
    packages=find_packages(where="python"),
    package_dir={"": "python"},
    ext_modules=ext_modules,
    cmdclass={"build_ext": BuildExtension.with_options(use_ninja=True)},
    options={"bdist_wheel": {"py_limited_api": "cp39"}},
)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Build helper / 构建辅助脚本
- **Primary symbols / 主要符号**: `_get_version`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `os`, `pathlib`, `platform`, `setuptools`, `sys`, `torch`, `torch.utils.cpp_extension`
