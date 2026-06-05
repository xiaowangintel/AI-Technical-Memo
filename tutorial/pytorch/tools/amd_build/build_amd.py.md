# build_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/amd_build/build_amd.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements a PyTorch developer-tool script that transforms metadata, validates state, or orchestrates workflow steps.
- **Purpose (CN)**: 实现一个 PyTorch 开发工具脚本，用于转换元数据、校验状态或编排工作流步骤。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
#!/usr/bin/env python3


import argparse
import os
import sys
from pathlib import Path
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, os, sys, and 1 more. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、os、sys 等共 4 项。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 10-19
```python
# NOTE: `tools/amd_build/build_amd.py` could be a symlink.
# The behavior of `symlink / '..'` is different from `symlink.parent`.
# Use `pardir` three times rather than using `path.parents[2]`.
REPO_ROOT = (
    Path(__file__).absolute() / os.path.pardir / os.path.pardir / os.path.pardir
).resolve()
sys.path.append(str(REPO_ROOT / "torch" / "utils"))

from hipify import hipify_python  # type: ignore[import]
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as hipify. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 hipify。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 20-29
```python

parser = argparse.ArgumentParser(
    description="Top-level script for HIPifying, filling in most common parameters"
)
parser.add_argument(
    "--out-of-place-only",
    action="store_true",
    help="Whether to only run hipify out-of-place on source files",
)
```
- **EN**: Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 30-45
```python
parser.add_argument(
    "--project-directory",
    type=str,
    default="",
    help="The root of the project.",
    required=False,
)

parser.add_argument(
    "--output-directory",
    type=str,
    default="",
    help="The directory to store the hipified project",
    required=False,
)
```
- **EN**: Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 46-54
```python
parser.add_argument(
    "--extra-include-dir",
    type=str,
    default=[],
    nargs="+",
    help="The list of extra directories in caffe2 to hipify",
    required=False,
)
```
- **EN**: Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 55-63
```python
args = parser.parse_args()

# NOTE: `tools/amd_build/build_amd.py` could be a symlink.
amd_build_dir = os.path.dirname(os.path.realpath(__file__))
proj_dir = os.path.dirname(os.path.dirname(amd_build_dir))

if args.project_directory:
    proj_dir = args.project_directory
```
- **EN**: Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 64-81
```python
out_dir = proj_dir
if args.output_directory:
    out_dir = args.output_directory

includes = [
    "caffe2/operators/*",
    "caffe2/sgd/*",
    "caffe2/image/*",
    "caffe2/transforms/*",
    "caffe2/video/*",
    "caffe2/distributed/*",
    "caffe2/queue/*",
    "caffe2/contrib/aten/*",
    "binaries/*",
    "caffe2/**/*_test*",
    "caffe2/core/*",
    "caffe2/db/*",
    "caffe2/utils/*",
```
- **EN**: Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 82-99
```python
    "caffe2/contrib/gloo/*",
    "caffe2/contrib/nccl/*",
    "c10/cuda/*",
    "c10/cuda/test/CMakeLists.txt",
    "modules/*",
    "third_party/nvfuser/*",
    # PyTorch paths
    # Keep this synchronized with is_pytorch_file in hipify_python.py
    "aten/src/ATen/cuda/*",
    "aten/src/ATen/native/cuda/*",
    "aten/src/ATen/native/cudnn/*",
    "aten/src/ATen/native/quantized/cudnn/*",
    "aten/src/ATen/native/nested/cuda/*",
    "aten/src/ATen/native/sparse/cuda/*",
    "aten/src/ATen/native/quantized/cuda/*",
    "aten/src/ATen/native/transformers/cuda/attention_backward.cu",
    "aten/src/ATen/native/transformers/cuda/attention.cu",
    "aten/src/ATen/native/transformers/cuda/sdp_utils.cpp",
```
- **EN**: This chunk contributes a small but necessary piece of the pytorch tooling implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了PyTorch 工具链实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 100-114
```python
    "aten/src/ATen/native/transformers/cuda/sdp_utils.h",
    "aten/src/ATen/native/transformers/cuda/mem_eff_attention/debug_utils.h",
    "aten/src/ATen/native/transformers/cuda/mem_eff_attention/gemm_kernel_utils.h",
    "aten/src/ATen/native/transformers/cuda/mem_eff_attention/pytorch_utils.h",
    "aten/src/THC/*",
    "aten/src/ATen/test/*",
    # CMakeLists.txt isn't processed by default, but there are a few
    # we do want to handle, so explicitly specify them
    "aten/src/THC/CMakeLists.txt",
    "torch/*",
    "tools/autograd/templates/python_variable_methods.cpp",
    "torch/csrc/stable/*",
    "test/cpp/c10d/*",
]
```
- **EN**: This chunk contributes a small but necessary piece of the pytorch tooling implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了PyTorch 工具链实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 115-132
```python
includes = [os.path.join(proj_dir, include) for include in includes]

for new_dir in args.extra_include_dir:
    abs_new_dir = os.path.join(proj_dir, new_dir)
    if os.path.exists(abs_new_dir):
        abs_new_dir = os.path.join(abs_new_dir, "**/*")
        includes.append(abs_new_dir)

ignores = [
    "caffe2/operators/depthwise_3x3_conv_op_cudnn.cu",
    "caffe2/operators/pool_op_cudnn.cu",
    "*/hip/*",
    # These files are compatible with both cuda and hip
    "aten/src/ATen/core/*",
    # Correct path to generate HIPConfig.h:
    #   CUDAConfig.h.in -> (amd_build) HIPConfig.h.in -> (cmake) HIPConfig.h
    "aten/src/ATen/cuda/CUDAConfig.h",
    "third_party/nvfuser/csrc/codegen.cpp",
```
- **EN**: Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 133-146
```python
    "third_party/nvfuser/runtime/block_reduction.cu",
    "third_party/nvfuser/runtime/block_sync_atomic.cu",
    "third_party/nvfuser/runtime/block_sync_default_rocm.cu",
    "third_party/nvfuser/runtime/broadcast.cu",
    "third_party/nvfuser/runtime/grid_reduction.cu",
    "third_party/nvfuser/runtime/helpers.cu",
    "torch/csrc/jit/codegen/fuser/cuda/resource_strings.h",
    "torch/csrc/jit/tensorexpr/ir_printer.cpp",
    "torch/csrc/jit/ir/ir.h",
    # generated files we shouldn't frob
    "torch/lib/tmp_install/*",
    "torch/include/*",
]
```
- **EN**: This chunk contributes a small but necessary piece of the pytorch tooling implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了PyTorch 工具链实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 147-156
```python
ignores = [os.path.join(proj_dir, ignore) for ignore in ignores]


# Check if the compiler is hip-clang.
#
# This used to be a useful function but now we can safely always assume hip-clang.
# Leaving the function here avoids bc-linter errors.
def is_hip_clang() -> bool:
    return True
```
- **EN**: This chunk defines `is_hip_clang`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_hip_clang`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 157-174
```python

# TODO Remove once the following submodules are updated
hip_platform_files = [
    "third_party/fbgemm/fbgemm_gpu/CMakeLists.txt",
    "third_party/fbgemm/fbgemm_gpu/cmake/Hip.cmake",
    "third_party/fbgemm/fbgemm_gpu/codegen/embedding_backward_dense_host.cpp",
    "third_party/fbgemm/fbgemm_gpu/codegen/embedding_backward_split_host_template.cpp",
    "third_party/fbgemm/fbgemm_gpu/codegen/embedding_backward_split_template.cu",
    "third_party/fbgemm/fbgemm_gpu/codegen/embedding_forward_quantized_split_lookup.cu",
    "third_party/fbgemm/fbgemm_gpu/include/fbgemm_gpu/utils/cuda_prelude.cuh",
    "third_party/fbgemm/fbgemm_gpu/include/fbgemm_gpu/utils/stochastic_rounding.cuh",
    "third_party/fbgemm/fbgemm_gpu/include/fbgemm_gpu/utils/vec4.cuh",
    "third_party/fbgemm/fbgemm_gpu/include/fbgemm_gpu/utils/weight_row.cuh",
    "third_party/fbgemm/fbgemm_gpu/include/fbgemm_gpu/sparse_ops.cuh",
    "third_party/fbgemm/fbgemm_gpu/src/jagged_tensor_ops.cu",
    "third_party/fbgemm/fbgemm_gpu/src/quantize_ops.cu",
    "third_party/fbgemm/fbgemm_gpu/src/sparse_ops.cu",
    "third_party/fbgemm/fbgemm_gpu/src/split_embeddings_cache_cuda.cu",
```
- **EN**: This chunk continues `is_hip_clang` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `is_hip_clang`，进一步展开其内部控制流或数据流转。

### Lines 175-183
```python
    "third_party/fbgemm/fbgemm_gpu/src/topology_utils.cpp",
    "third_party/fbgemm/src/EmbeddingSpMDM.cc",
    "third_party/gloo/cmake/Dependencies.cmake",
    "third_party/gloo/gloo/cuda.cu",
    "third_party/kineto/libkineto/CMakeLists.txt",
    "third_party/nvfuser/CMakeLists.txt",
    "third_party/tensorpipe/cmake/Hip.cmake",
]
```
- **EN**: This chunk continues `is_hip_clang` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `is_hip_clang`，进一步展开其内部控制流或数据流转。

### Lines 184-193
```python

def remove_hcc(line: str) -> str:
    line = line.replace("HIP_PLATFORM_HCC", "HIP_PLATFORM_AMD")
    line = line.replace("HIP_HCC_FLAGS", "HIP_CLANG_FLAGS")
    return line


for hip_platform_file in hip_platform_files:
    do_write = False
    if os.path.exists(hip_platform_file):
```
- **EN**: This chunk defines `remove_hcc`, which implements a focused step inside the pytorch tooling pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `remove_hcc`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 194-204
```python
        with open(hip_platform_file) as sources:
            lines = sources.readlines()
        newlines = [remove_hcc(line) for line in lines]
        if lines == newlines:
            print(f"{hip_platform_file} skipped")
        else:
            with open(hip_platform_file, "w") as sources:
                for line in newlines:
                    sources.write(line)
            print(f"{hip_platform_file} updated")
```
- **EN**: This chunk continues `remove_hcc` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `remove_hcc`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 205-216
```python
# NOTE: MSLK sources needing hipify
# MSLK is its own project with its own build system. pytorch uses mslk as
# a submodule to acquire some gpu source files but compiles only those sources
# instead of using mslk's own build system. One of the source files refers
# to a header file that is the result of running hipify, but mslk uses
# slightly different hipify settings than pytorch. mslk normally hipifies
# and renames tuning_cache.cuh to tuning_cache_hip.cuh, but pytorch's settings
# for hipify puts it into its own 'hip' directory. After hipify runs below with
# the added mslk file, we move it to its expected location.
# NOTE: Internal meta builds (using buck) don't need this step, so conditionally disable it
buck_build = os.environ.get("FBCODE_BUILD_TOOL", "") == "buck"
```
- **EN**: This chunk continues `remove_hcc` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `remove_hcc`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 217-225
```python
extra_files = [
    "torch/_inductor/codegen/cuda/device_op_overrides.py",
    "torch/_inductor/codegen/cpp_wrapper_cpu.py",
    "torch/_inductor/codegen/cpp_wrapper_gpu.py",
    "torch/_inductor/codegen/wrapper.py",
]

mslk_dir = REPO_ROOT / "third_party/mslk/include/mslk/utils/"
```
- **EN**: This chunk continues `remove_hcc` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `remove_hcc`，进一步展开其内部控制流或数据流转。

### Lines 226-243
```python

if not buck_build:
    mslk_original = mslk_dir / "tuning_cache.cuh"
    if mslk_original.exists():
        extra_files.append(mslk_original.as_posix())

# TODO Remove once the following submodules are updated to use hipify v2
hipify_v1_to_v2_files = [
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/gemm/ck_extensions.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/bf16_grouped/bf16_grouped_gemm.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/bf16_grouped/kernels/bf16_grouped_common.h",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/ck_utility.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_blockwise_gemm.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_rowwise_batched/kernels/fp8_rowwise_batched_common.h",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_rowwise_grouped/fp8_rowwise_grouped_gemm.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_rowwise_grouped/kernels/fp8_rowwise_grouped_common.h",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_rowwise/kernels/fp8_rowwise_common.h",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_rowwise_preshuffle/kernels/fp8_rowwise_preshuffle_common.h",
```
- **EN**: This chunk continues `remove_hcc` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `remove_hcc`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 244-258
```python
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fp8_tensorwise_gemm.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/ck_extensions/fused_moe/fused_moe_kernel.hip",
    "third_party/fbgemm/fbgemm_gpu/experimental/gen_ai/src/quantize/common/include/fbgemm_gpu/quantize/tuning_cache.hpp",
    "third_party/mslk/csrc/moe/ck_extensions/fused_moe_kernel.hip",
    "third_party/mslk/csrc/gemm/ck/bf16_grouped/bf16_grouped_gemm.hip",
    "third_party/mslk/csrc/gemm/ck/bf16_grouped/kernels/bf16_grouped_common.h",
    "third_party/mslk/csrc/gemm/ck/fp8_rowwise/kernels/fp8_rowwise_common.h",
    "third_party/mslk/csrc/gemm/ck/fp8_rowwise_grouped/kernels/fp8_rowwise_grouped_common.h",
    "third_party/mslk/csrc/gemm/ck/fp8_rowwise_grouped/fp8_rowwise_grouped_gemm.hip",
    "third_party/mslk/csrc/gemm/ck/fp8_rowwise_batched/kernels/fp8_rowwise_batched_common.h",
    "third_party/mslk/csrc/gemm/ck/fp8_rowwise_preshuffle/kernels/fp8_rowwise_preshuffle_common.h",
    "third_party/mslk/csrc/gemm/ck/fp8_tensorwise_gemm.hip",
    "third_party/mslk/csrc/gemm/ck/fp8_blockwise_gemm.hip",
]
```
- **EN**: This chunk continues `remove_hcc` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `remove_hcc`，进一步展开其内部控制流或数据流转。

### Lines 259-270
```python

def hipify_v1_to_v2(line: str) -> str:
    line = line.replace("hip::HIPStreamMasqueradingAsCUDA", "cuda::CUDAStream")
    line = line.replace(
        "hip::HIPStreamGuardMasqueradingAsCUDA", "cuda::CUDAStreamGuard"
    )
    line = line.replace(
        "hip::getStreamFromPoolMasqueradingAsCUDA", "cuda::getStreamFromPool"
    )
    line = line.replace("getCurrentHIPStream", "getCurrentCUDAStream")
    return line
```
- **EN**: This chunk defines `hipify_v1_to_v2`, which implements a focused step inside the pytorch tooling pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `hipify_v1_to_v2`，其作用是实现PyTorch 工具链流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 271-282
```python

for hipify_v1_to_v2_file in hipify_v1_to_v2_files:
    do_write = False
    if os.path.exists(hipify_v1_to_v2_file):
        with open(hipify_v1_to_v2_file) as sources:
            lines = sources.readlines()
        newlines = [hipify_v1_to_v2(line) for line in lines]
        if lines == newlines:
            print(f"{hipify_v1_to_v2_file} skipped")
        else:
            with open(hipify_v1_to_v2_file, "w") as sources:
                for line in newlines:
```
- **EN**: This chunk continues `hipify_v1_to_v2` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `hipify_v1_to_v2`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 283-296
```python
                    sources.write(line)
            print(f"{hipify_v1_to_v2_file} updated")


hipify_python.hipify(
    project_directory=proj_dir,
    output_directory=out_dir,
    includes=includes,
    ignores=ignores,
    extra_files=extra_files,
    out_of_place_only=args.out_of_place_only,
    hip_clang_launch=is_hip_clang(),
)
```
- **EN**: This chunk continues `hipify_v1_to_v2` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `hipify_v1_to_v2`，进一步展开其内部控制流或数据流转。

### Lines 297-305
```python
if not buck_build:
    mslk_move_src = mslk_dir / "hip/tuning_cache.cuh"
    mslk_move_dst = mslk_dir / "tuning_cache_hip.cuh"

    # only update the file if it changes or doesn't exist
    do_write = True
    src_lines = None

    if not mslk_move_src.exists():
```
- **EN**: This chunk continues `hipify_v1_to_v2` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `hipify_v1_to_v2`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 306-314
```python
        _error = f"Error: Source file {mslk_move_src} does not exist"
        sys.exit(_error)
    with open(mslk_move_src) as src:
        src_lines = src.readlines()
    if os.path.exists(mslk_move_dst):
        dst_lines = None
        with open(mslk_move_dst) as dst:
            dst_lines = dst.readlines()
        if src_lines == dst_lines:
```
- **EN**: This chunk continues `hipify_v1_to_v2` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `hipify_v1_to_v2`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 315-321
```python
            print(f"{mslk_move_dst} skipped")
            do_write = False
    if do_write:
        with open(mslk_move_dst, "w") as dst:
            for line in src_lines:
                dst.write(line)
        print(f"{mslk_move_dst} updated")
```
- **EN**: This chunk continues `hipify_v1_to_v2` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `hipify_v1_to_v2`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **PyTorch tooling**
  - EN: This file belongs to the pytorch tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于PyTorch 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **Autograd generation**
  - EN: The file participates in turning operator metadata into backward formulas, wrappers, or saved-state logic.
  - CN: 该文件参与把算子元数据转换为反向公式、包装层或保存状态逻辑。
- **Template expansion**
  - EN: The implementation relies on placeholders or structured text expansion to generate source artifacts.
  - CN: 该实现依赖占位符或结构化文本展开来生成源码产物。
- **Build configuration**
  - EN: The file helps discover tools, cache build settings, or translate Python-side configuration into native build arguments.
  - CN: 该文件帮助发现工具、缓存构建设置，或把 Python 侧配置转换为原生构建参数。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `os`, `sys`, `pathlib`
- **External packages / 外部依赖包**: `hipify`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `is_hip_clang`, `remove_hcc`, `hipify_v1_to_v2`
