# setup_musa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/setup_musa.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This build/setup script prepares the `MUSA build` path, checks prerequisites, and compiles or installs native pieces when required. / 该构建/安装脚本用于准备 `MUSA build` 路径，检查前置条件，并在需要时编译或安装原生组件。

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

### Lines 16-20: Imports and module setup
````python
import os
import platform
import subprocess
import sys
from pathlib import Path
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 21-22: Comments and local context
````python

# isort: off
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 23-24: Imports and module setup
````python
import torch
import torchada  # noqa: F401
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 25-26: Comments and local context
````python

# isort: on
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 27-28: Imports and module setup
````python
from setuptools import find_packages, setup
from torch.utils.cpp_extension import BuildExtension, CUDAExtension
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 30-32: Constants and configuration
````python
root = Path(__file__).parent.resolve()
third_party = Path(os.environ.get("SGLANG_MUSA_THIRD_PARTY_DIR", "build/_deps"))
arch = platform.machine().lower()
````
**EN:** This block defines shared constants or configuration values such as `root`, `third_party`, `arch`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `root`, `third_party`, `arch`），供后续函数或控制流程复用。

### Lines 35-43: `_RepoInfo` definition
````python
class _RepoInfo:
    """Configuration for a third-party git repository."""

    def __init__(self, name, git_repository, git_tag, git_shallow=False):
        self.name = name
        self.git_repository = git_repository
        self.git_tag = git_tag
        self.git_shallow = git_shallow
        self.source_dir = third_party / name
````
**EN:** This section defines the class `_RepoInfo`, grouping related state and behavior around Repo Info. Docstring summary: Configuration for a third-party git repository.
**CN:** 该部分定义类 `_RepoInfo`，把与 Repo Info 相关的状态和行为组织在一起。 文档字符串摘要：Configuration for a third-party git repository.

### Lines 46-58: Constants and configuration
````python
_FLASHINFER_REPO = _RepoInfo(
    name="flashinfer",
    git_repository="https://github.com/flashinfer-ai/flashinfer.git",
    git_tag="bc29697ba20b7e6bdb728ded98f04788e16ee021",
    git_shallow=False,
)

_MUTLASS_REPO = _RepoInfo(
    name="mutlass",
    git_repository="https://github.com/MooreThreads/mutlass.git",
    git_tag="3abd6a728aacd190df0d922514aca8a8bc3c46b7",
    git_shallow=False,
)
````
**EN:** This block defines shared constants or configuration values such as `_FLASHINFER_REPO`, `_MUTLASS_REPO`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_FLASHINFER_REPO`, `_MUTLASS_REPO`），供后续函数或控制流程复用。

### Lines 61-65: `_get_version` definition
````python
def _get_version():
    with open(root / "pyproject.toml") as f:
        for line in f:
            if line.startswith("version"):
                return line.split("=")[1].strip().strip('"')
````
**EN:** This section defines `_get_version` and implements the core logic associated with get version.
**CN:** 该部分定义 `_get_version`，并实现与 get version 相关的核心逻辑。

### Lines 68-128: Constants and configuration
````python
operator_namespace = "sgl_kernel"
include_dirs = [
    root / "include",
    root / "include" / "impl",
    root / "csrc",
    root / _FLASHINFER_REPO.source_dir / "include",
    root / _FLASHINFER_REPO.source_dir / "csrc",
    root / _MUTLASS_REPO.source_dir / "include",
]

sources = [
    "csrc/allreduce/custom_all_reduce.cu",
    "csrc/attention/merge_attn_states.cu",
    "csrc/common_extension_musa.cc",
    "csrc/elementwise/activation.cu",
    "csrc/elementwise/concat_mla.cu",
    "csrc/elementwise/pos_enc.cu",
    "csrc/elementwise/fused_add_rms_norm_kernel.mu",
    "csrc/grammar/apply_token_bitmask_inplace_cuda.cu",
    "csrc/moe/moe_align_kernel.cu",
    "csrc/moe/moe_fused_gate_musa.cu",
    "csrc/moe/kimi_k2_moe_fused_gate.cu",
    "csrc/moe/moe_sum.cu",
    "csrc/moe/moe_sum_reduce.cu",
    "csrc/moe/moe_topk_softmax_kernels.cu",
    "csrc/quantization/gguf/gguf_kernel.cu",
    "csrc/speculative/eagle_utils.cu",
    "csrc/speculative/ngram_utils.cu",
    "csrc/speculative/packbit.cu",
    "csrc/speculative/speculative_sampling.cu",
    "csrc/kvcacheio/transfer.cu",
    "csrc/gemm/awq_kernel.cu",
    "csrc/gemm/bmm_fp8.cu",
    "csrc/gemm/dsv3_fused_a_gemm.cu",
    "csrc/gemm/dsv3_router_gemm_bf16_out.cu",
    "csrc/gemm/dsv3_router_gemm_entry.cu",
    "csrc/gemm/dsv3_router_gemm_float_out.cu",
    "csrc/gemm/per_token_quant_fp8.cu",
    "csrc/gemm/per_token_group_quant_8bit.cu",
    "csrc/gemm/per_token_group_quant_8bit_v2.cu",
    "csrc/memory/weak_ref_tensor.cpp",
    str(_FLASHINFER_REPO.source_dir / "csrc/norm.cu"),
    str(_FLASHINFER_REPO.source_dir / "csrc/renorm.cu"),
    str(_FLASHINFER_REPO.source_dir / "csrc/sampling.cu"),
    # XXX (MUSA): The following files contain MUSA-specific implementations.
    "csrc/musa/pos_encoding_contiguous.mu",
    "csrc/musa/moe_gemv_swiglu.mu",
    "csrc/musa/ternary.mu",
    "csrc/musa/top_k_top_p_sampling.mu",
]

cxx_flags = ["force_mcc"]
libraries = ["c10", "torch", "torch_python"]
extra_link_args = [
    "-Wl,-rpath,$ORIGIN/../../torch/lib",
    f"-L/usr/lib/{arch}-linux-gnu",
    "-lmublasLt",
]

default_target = "mp_31"
mtgpu_target = os.environ.get("MTGPU_TARGET", default_target)
````
**EN:** This block defines shared constants or configuration values such as `operator_namespace`, `include_dirs`, `sources`, `cxx_flags`, `libraries`, `extra_link_args`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `operator_namespace`, `include_dirs`, `sources`, `cxx_flags`, `libraries`, `extra_link_args`），供后续函数或控制流程复用。

### Lines 130-137: Conditional logic
````python
if torch.musa.is_available():
    try:
        prop = torch.musa.get_device_properties(0)
        mtgpu_target = f"mp_{prop.major}{prop.minor}"
    except Exception as e:
        print(f"Warning: Failed to detect GPU properties: {e}")
else:
    print(f"Warning: torch.musa not available. Using default target: {mtgpu_target}")
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 139-143: Conditional logic
````python
if mtgpu_target not in ["mp_22", "mp_31"]:
    print(
        f"Warning: Unsupported GPU architecture detected '{mtgpu_target}'. Expected 'mp_22' or 'mp_31'."
    )
    sys.exit(1)
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 145-163: Constants and configuration
````python
mcc_flags = [
    "-DNDEBUG",
    f"-DOPERATOR_NAMESPACE={operator_namespace}",
    "-O3",
    "-fPIC",
    "-std=c++17",
    f"--cuda-gpu-arch={mtgpu_target}",
    "-x",
    "musa",
    "-mtgpu",
    "-Od3",
    "-ffast-math",
    "-fmusa-flush-denormals-to-zero",
    "-fno-strict-aliasing",
    "-DUSE_MUSA",
    "-DENABLE_BF16",
    "-DFLASHINFER_ENABLE_F16",
    "-DFLASHINFER_ENABLE_BF16",
]
````
**EN:** This block defines shared constants or configuration values such as `mcc_flags`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `mcc_flags`），供后续函数或控制流程复用。

### Lines 165-173: Conditional logic
````python
if mtgpu_target == "mp_31":
    mcc_flags.extend(
        [
            "-DENABLE_FP8",
            "-DFLASHINFER_ENABLE_FP8",
            "-DFLASHINFER_ENABLE_FP8_E4M3",
            "-DFLASHINFER_ENABLE_FP8_E5M2",
        ]
    )
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 175-188: Constants and configuration
````python
ext_modules = [
    CUDAExtension(
        name="sgl_kernel.common_ops",
        sources=sources,
        include_dirs=include_dirs,
        extra_compile_args={
            "mcc": mcc_flags,
            "cxx": cxx_flags,
        },
        libraries=libraries,
        extra_link_args=extra_link_args,
        py_limited_api=False,
    ),
]
````
**EN:** This block defines shared constants or configuration values such as `ext_modules`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `ext_modules`），供后续函数或控制流程复用。

### Lines 191-230: `_CustomBuildExt` definition
````python
class _CustomBuildExt(BuildExtension):
    """Custom build extension that clones third-party repositories before building."""

    @staticmethod
    def _clone_and_checkout(repo_path, repo_url, git_tag, git_shallow):
        """Clone a git repository and checkout a specific tag/commit."""
        repo_path.parent.mkdir(parents=True, exist_ok=True)
        if not repo_path.exists():
            clone_cmd = ["git", "clone"]
            if git_shallow:
                clone_cmd += ["--depth", "1"]
            clone_cmd += [repo_url, str(repo_path)]
            subprocess.check_call(clone_cmd)
            subprocess.check_call(["git", "checkout", git_tag], cwd=repo_path)
        else:
            subprocess.check_call(["git", "fetch", "--all"], cwd=repo_path)
            subprocess.check_call(["git", "checkout", git_tag], cwd=repo_path)

    def run(self):
        if os.environ.get("SGLANG_MUSA_SKIP_THIRD_PARTY", "0") == "1":
            print(
                "Skipping third-party repositories cloning (SGLANG_MUSA_SKIP_THIRD_PARTY=1)"
            )
        else:
            print("Cloning third-party repositories...")
            self._clone_and_checkout(
                _MUTLASS_REPO.source_dir,
                _MUTLASS_REPO.git_repository,
                _MUTLASS_REPO.git_tag,
                _MUTLASS_REPO.git_shallow,
            )
            self._clone_and_checkout(
                _FLASHINFER_REPO.source_dir,
                _FLASHINFER_REPO.git_repository,
                _FLASHINFER_REPO.git_tag,
                _FLASHINFER_REPO.git_shallow,
            )
            print("Third-party repositories ready.")

        super().run()
````
**EN:** This section defines the class `_CustomBuildExt`, grouping related state and behavior around Custom Build Ext. Docstring summary: Custom build extension that clones third-party repositories before building.
**CN:** 该部分定义类 `_CustomBuildExt`，把与 Custom Build Ext 相关的状态和行为组织在一起。 文档字符串摘要：Custom build extension that clones third-party repositories before building.

### Lines 233-241: Top-level logic
````python
setup(
    name="sglang-kernel",
    version=_get_version(),
    packages=find_packages(where="python"),
    package_dir={"": "python"},
    ext_modules=ext_modules,
    cmdclass={"build_ext": _CustomBuildExt.with_options(use_ninja=True)},
    options={"bdist_wheel": {"py_limited_api": "cp39"}},
)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Build helper / 构建辅助脚本
- **Primary symbols / 主要符号**: `_RepoInfo`, `_get_version`, `_CustomBuildExt`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, toolchain subprocess control / 工具链子进程控制

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `os`, `pathlib`, `platform`, `setuptools`, `subprocess`, `sys`, `torch`, `torch.utils.cpp_extension`, `torchada`
