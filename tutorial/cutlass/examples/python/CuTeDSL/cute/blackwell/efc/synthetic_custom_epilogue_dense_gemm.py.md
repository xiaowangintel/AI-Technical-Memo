# synthetic_custom_epilogue_dense_gemm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/efc/synthetic_custom_epilogue_dense_gemm.py`  
**Purpose / 用途**: Kernel example implementing synthetic custom epilogue dense gemm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 synthetic custom epilogue dense gemm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行

~~~~python
# Copyright (c) 2025 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause

# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:

# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.

# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.

# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.

# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
~~~~

**EN**: Records the copyright, SPDX tags, and license conditions that govern how this example may be used and redistributed.
**CN**: 记录版权、SPDX 标记和许可证条件，说明该示例如何被使用和再分发。

### Lines 29-40 / 第 29-40 行

~~~~python
import traceback
import typing

import cuda.bindings.driver as cuda
import torch

import cutlass
import cutlass.cute.testing as testing
import cutlass.torch as cutlass_torch

from common_dense_gemm_efc import DenseGemmEFC
import common_efc
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 42-111 / 第 42-111 行

~~~~python
"""
A high-performance persistent batched dense GEMM (D = alpha * A * B + beta * C) example for the NVIDIA Blackwell SM100 architecture
using CUTE DSL.
- Matrix A is MxKxL, L is batch dimension, A can be row-major("K") or column-major("M")
- Matrix B is NxKxL, L is batch dimension, B can be row-major("N") or column-major("K")
- Matrix C is MxNxL, L is batch dimension, C can be row-major("N") or column-major("M")
- Matrix D is MxNxL, L is batch dimension, D can be row-major("N") or column-major("M")
- alpha and beta are float scalars

This GEMM kernel supports the following features:
    - Utilizes Tensor Memory Access (TMA) for efficient memory operations
    - Utilizes Blackwell's tcgen05.mma for matrix multiply-accumulate (MMA) operations (including 2cta mma instructions)
    - Implements TMA multicast with cluster to reduce L2 memory traffic
    - Support persistent tile scheduling to better overlap memory load/store with mma between tiles
    - Support warp specialization to avoid explicit pipelining between mainloop load and mma

This GEMM works as follows:
1. DMA warp: Load A and B matrices from global memory (GMEM) to shared memory (SMEM) using TMA operations.
2. MMA warp: Perform matrix multiply-accumulate (MMA) operations using tcgen05.mma instruction.
3. EPILOGUE warp:
    - Load completed accumulator from tensor memory (TMEM) to registers (RMEM) using tcgen05.ld.
    - Load C matrix from global memory (GMEM) to shared memory (SMEM) using TMA operations and then copied to registers (RMEM).
    - Compute D = alpha * accumulator + beta * C.
    - Type convert D matrix to output type.
    - Store D matrix from registers (RMEM) to shared memory (SMEM) to global memory (GMEM) with TMA operations,
    - Optionally accept an elementwise lambda function epilogue_op to apply to the output tensor:
      e.g., relu can set epilogue_op = lambda x: cute.where(x > 0, x, cute.full_like(x, 0))

SM100 tcgen05.mma instructions operate as follows:
- Read matrix A from SMEM
- Read matrix B from SMEM
- Write accumulator to TMEM
The accumulator in TMEM must then be loaded to registers before writing back to GMEM.

Input arguments to this example is same as dense_gemm.py.

.. code-block:: bash

    python examples/internal/blackwell/epilogue/synthetic_custom_epilogue_dense_gemm.py \
      --ab_dtype Float16 --c_dtype Float16 --d_dtype Float16 --acc_dtype Float32 --epi_dtype Float32 \
      --mma_tiler_mn 256,128 --cluster_shape_mn 2,1 \
      --mnkl 8192,8192,8192,1 \
      --use_2cta_instrs --alpha 2.0 --beta 1.0 --t_dtype Float32 --read_tensors 2 --written_tensors 3

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python examples/internal/blackwell/epilogue/synthetic_custom_epilogue_dense_gemm.py \
      --ab_dtype Float16 --c_dtype Float16 --d_dtype Float16 --acc_dtype Float32 --epi_dtype Float32 \
      --mma_tiler_mn 256,128 --cluster_shape_mn 2,1 \
      --mnkl 8192,8192,8192,1 \
      --use_2cta_instrs --alpha 2.0 --beta 1.0 --t_dtype Float32 --read_tensors 2 --written_tensors 3 \
      --warmup_iterations 1 --iterations 10 --skip_ref_check


Constraints are same as dense_gemm.py:
* Supported input data types: fp16, bf16, tf32, int8, uint8, fp8 (e4m3fn, e5m2),
  see detailed valid dtype combinations in below SM100PersistentDenseGemmAlphaBetaKernel class documentation
* A/B tensor must have the same data type
* C/D tensor must have the same major order
* Mma tiler M must be 64/128 (use_2cta_instrs=False) or 128/256 (use_2cta_instrs=True)
* Mma tiler N must be 32-256, step 32
* Cluster shape M/N must be positive and power of 2, total cluster size <= 16
* Cluster shape M must be multiple of 2 if use_2cta_instrs=True
* The contiguous dimension of A/B/C/D tensors must be at least 16 bytes aligned,
  i.e, number of elements is a multiple of 4, 8, and 16 for TFloat32,
  Float16/BFloat16, and Int8/Uint8/Float8, respectively.
* OOB tiles are not allowed when TMA store is disabled
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 114-132 / 第 114-132 行

~~~~python
def format_as_cli_args(
    mnkl: typing.Tuple[int, int, int, int],
    ab_dtype: typing.Type[cutlass.Numeric],
    acc_dtype: typing.Type[cutlass.Numeric],
    epi_dtype: typing.Type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    cd_major: str,
    mma_tiler_mn: typing.Tuple[int, int],
    cluster_shape_mn: typing.Tuple[int, int],
    use_2cta_instrs: bool,
    t_dtype: typing.Type[cutlass.Numeric],
    alpha: float,
    beta: float,
    read_tensors: int,
    written_tensors: int,
    tolerance: float,
) -> str:
    """Format test parameters as CLI arguments for synthetic_custom_epilogue_dense_gemm.py"""
~~~~

**EN**: Defines `format_as_cli_args`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `format_as_cli_args`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 134-134 / 第 134-134 行

~~~~python
    # Get base command from DenseGemmEFC class
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 135-148 / 第 135-148 行

~~~~python
    base_cmd = DenseGemmEFC.format_as_cli_args(
        "synthetic_custom_epilogue_dense_gemm.py",
        mnkl,
        ab_dtype,
        acc_dtype,
        epi_dtype,
        a_major,
        b_major,
        cd_major,
        mma_tiler_mn,
        cluster_shape_mn,
        use_2cta_instrs,
        tolerance,
    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 150-150 / 第 150-150 行

~~~~python
    # Add synthetic epilogue-specific parameters
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 151-157 / 第 151-157 行

~~~~python
    specific_args = (
        f" --alpha {alpha} "
        f"--beta {beta} "
        f"--t_dtype {DenseGemmEFC.dtype_name(t_dtype)} "
        f"--read_tensors {read_tensors} "
        f"--written_tensors {written_tensors}"
    )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 159-159 / 第 159-159 行

~~~~python
    return base_cmd + specific_args
~~~~

**EN**: Returns the constructed object or computed result to the caller. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把构造好的对象或计算结果返回给调用方。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 162-189 / 第 162-189 行

~~~~python
def run(
    mnkl: typing.Tuple[int, int, int, int],
    ab_dtype: typing.Type[cutlass.Numeric],
    acc_dtype: typing.Type[cutlass.Numeric],
    epi_dtype: typing.Type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    cd_major: str,
    alpha: float,
    beta: float,
    t_dtype: typing.Type[cutlass.Numeric],
    mma_tiler_mn: typing.Tuple[int, int],
    cluster_shape_mn: typing.Tuple[int, int],
    use_2cta_instrs: bool,
    tolerance: float,
    warmup_iterations: int = 0,
    iterations: int = 1,
    skip_ref_check: bool = False,
    read_tensors: int = 1,
    written_tensors: int = 1,
    verbose: bool = False,
):
    """
    Prepare A/B/C/D tensors, launch GPU kernel, and reference checking.
    """
    print("Running Blackwell Persistent Dense GEMM test with:")
    print(f"mnkl: {mnkl}")
    print(f"AB dtype: {ab_dtype}, Acc dtype: {acc_dtype}, Epi dtype: {epi_dtype}")
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 190-202 / 第 190-202 行

~~~~python
    print(
        f"Matrix majors - A: {a_major}, B: {b_major}, loaded: {cd_major}, stored: {cd_major}"
    )
    print(f"Mma Tiler (M, N): {mma_tiler_mn}, Cluster Shape (M, N): {cluster_shape_mn}")
    print(f"2CTA MMA instructions: {'True' if use_2cta_instrs else 'False'}")
    print(f"Tolerance: {tolerance}")
    print(f"Warmup iterations: {warmup_iterations}")
    print(f"Iterations: {iterations}")
    print(f"Skip reference checking: {skip_ref_check}")
    print("Epilogue:")
    print(f"\t{alpha = }, {beta = }")
    print(f"\t{t_dtype = !s}")
    print(f"\t{read_tensors = }, {written_tensors = }")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 204-204 / 第 204-204 行

~~~~python
    # Unpack parameters
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 205-205 / 第 205-205 行

~~~~python
    m, n, k, l = mnkl
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 207-208 / 第 207-208 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("A GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 210-210 / 第 210-210 行

~~~~python
    # Get current CUDA stream from PyTorch
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 211-213 / 第 211-213 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    # Get the raw stream pointer as a CUstream
    current_stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 215-217 / 第 215-217 行

~~~~python
    def meta_epilogue(read_tensors, written_tensors):
        """Build a synthetic epilogue function with parameters
        (self, alpha, beta, read_t0, read_t1,..., read_t{read__tensors-1}, written_t0, written_t1,..., written_t{written_tensors-1}"""
~~~~

**EN**: Defines `meta_epilogue`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `meta_epilogue`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 219-223 / 第 219-223 行

~~~~python
        param_names = (
            ["efc_config", "alpha", "beta"]
            + [f"read_t{i}" for i in range(read_tensors)]
            + [f"written_t{i}" for i in range(written_tensors)]
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 225-227 / 第 225-227 行

~~~~python
        assert written_tensors > 0, (
            "At least one tensor must be written in the epilogue."
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 229-239 / 第 229-239 行

~~~~python
        def computation_impl(efc_config, alpha, beta, *tensors):
            """Implementation of the epilogue computation."""
            read = beta
            for tensor in tensors[:read_tensors]:
                read += tensor.load() * alpha
            if read_tensors > 0:
                # Can use some CuTe/PyTorch-like functions exposed under
                # efc_config namespace for portability:
                read = efc_config.where(
                    read < 1, read, read * efc_config.full_like(read, 2)
                )
~~~~

**EN**: Defines `computation_impl`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `computation_impl`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 241-244 / 第 241-244 行

~~~~python
            t = efc_config.accum()
            for tensor in tensors[read_tensors:]:
                t = t * alpha + read + 5000
                tensor.store(t)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 246-247 / 第 246-247 行

~~~~python
        # Wrap the implementation with a function with the correct parameter
        # names.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 248-248 / 第 248-248 行

~~~~python
        return common_efc.create_named_epilogue(param_names, computation_impl)
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 250-250 / 第 250-250 行

~~~~python
    epilogue = meta_epilogue(read_tensors, written_tensors)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 252-253 / 第 252-253 行

~~~~python
    # Build GEMM object with EFC configuration:
    # TODO: generalize acc_dtype and epi_dtype
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 254-267 / 第 254-267 行

~~~~python
    gemm = DenseGemmEFC(
        acc_dtype,
        epi_dtype,
        use_2cta_instrs,
        mma_tiler_mn,
        cluster_shape_mn,
        epilogue,
    )
    (
        a_tensor,
        b_tensor,
        a_torch_cpu,
        b_torch_cpu,
    ) = gemm.create_arguments(l, m, n, k, a_major, b_major, cd_major, ab_dtype)
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 269-269 / 第 269-269 行

~~~~python
    # Create all the supplemental tensors.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 270-277 / 第 270-277 行

~~~~python
    t_torch_cpu, t_torch_gpu, t_tensor = ([], [], [])
    for i in range(read_tensors + written_tensors):
        t_torch_cpu.append(cutlass_torch.matrix(l, m, n, cd_major == "m", t_dtype))
        tensor, torch_gpu = cutlass_torch.cute_tensor_like(
            t_torch_cpu[i], t_dtype, is_dynamic_layout=True, assumed_align=16
        )
        t_tensor.append(tensor)
        t_torch_gpu.append(torch_gpu)
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 279-279 / 第 279-279 行

~~~~python
    # Check if configuration can be implemented
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 280-280 / 第 280-280 行

~~~~python
    gemm.check_implementable(a_tensor, b_tensor, t_tensor[0])
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 282-284 / 第 282-284 行

~~~~python
    max_active_clusters = cutlass.utils.HardwareInfo().get_max_active_clusters(
        cluster_shape_mn[0] * cluster_shape_mn[1]
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 286-297 / 第 286-297 行

~~~~python
    compiled_gemm = gemm.compile(
        a_tensor,
        b_tensor,
        max_active_clusters,
        current_stream,
        # Here are the supplemental arguments in the same order as for the
        # epilogue configuration function.
        alpha,
        beta,
        *t_tensor,
    )
    torch.cuda.synchronize()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 299-299 / 第 299-299 行

~~~~python
    # TODO: unify with modern way to do benchmarking.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 300-315 / 第 300-315 行

~~~~python
    exec_time = testing.benchmark(
        compiled_gemm,
        kernel_arguments=testing.JitArguments(
            a_tensor,
            b_tensor,
            current_stream,
            # Here are the supplemental arguments in the same order as for the
            # epilogue configuration function.
            alpha,
            beta,
            *t_tensor,
        ),
        stream=current_stream,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
    )
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 317-317 / 第 317-317 行

~~~~python
    print(f"Execution time: {exec_time} us")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 319-319 / 第 319-319 行

~~~~python
    # Evaluate the epilogue on the host:
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 320-328 / 第 320-328 行

~~~~python
    gemm.evaluate_on_cpu(
        a_torch_cpu,
        b_torch_cpu,
        epi_dtype,
        # The EFC arguments:
        alpha,
        beta,
        *t_torch_cpu,
    )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 330-330 / 第 330-330 行

~~~~python
    # Print tensors if verbose mode is enabled
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 331-335 / 第 331-335 行

~~~~python
    if verbose:
        print("\n=== Read Tensors ===")
        for i in range(read_tensors):
            print(f"\nRead Tensor {i} (GPU):")
            print(t_torch_gpu[i].cpu())
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 337-342 / 第 337-342 行

~~~~python
        print("\n=== Written Tensors ===")
        for i in range(written_tensors):
            idx = read_tensors + i
            print(f"\nWritten Tensor {i} (GPU):")
            print(t_torch_gpu[idx].cpu())
        print()
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 344-344 / 第 344-344 行

~~~~python
    # Assert close results between the values computed on GPU and CPU.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 345-348 / 第 345-348 行

~~~~python
    for torch_gpu, torch_cpu in zip(t_torch_gpu, t_torch_cpu):
        torch.testing.assert_close(
            torch_gpu.cpu(), torch_cpu, atol=tolerance, rtol=1e-03
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。

### Lines 351-375 / 第 351-375 行

~~~~python
if __name__ == "__main__":
    cli = DenseGemmEFC.CLIParser()
    cli.parser.add_argument(
        "--alpha", type=float, default=1.0, help="alpha scale factor"
    )
    cli.parser.add_argument("--beta", type=float, default=0.0, help="beta scale factor")
    cli.parser.add_argument("--t_dtype", type=cutlass.dtype, default=cutlass.Float32)
    cli.parser.add_argument(
        "--read_tensors",
        type=int,
        default=1,
        help="number of tensors to read inside the epilogue",
    )
    cli.parser.add_argument(
        "--written_tensors",
        type=int,
        default=1,
        help="number of tensors to write inside the epilogue",
    )
    cli.parser.add_argument(
        "--verbose",
        action="store_true",
        help="print read and written tensors",
    )
    args = cli.parse()
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 377-403 / 第 377-403 行

~~~~python
    try:
        run(
            args.mnkl,
            args.ab_dtype,
            args.acc_dtype,
            args.epi_dtype,
            args.a_major,
            args.b_major,
            args.cd_major,
            args.alpha,
            args.beta,
            args.t_dtype,
            args.mma_tiler_mn,
            args.cluster_shape_mn,
            args.use_2cta_instrs,
            args.tolerance,
            args.warmup_iterations,
            args.iterations,
            args.skip_ref_check,
            args.read_tensors,
            args.written_tensors,
            args.verbose,
        )
        print("PASS")
    except Exception as exc:
        traceback.print_exception(exc)
        raise
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

---

## Key Concepts / 关键概念

- CuTeDSL kernels and JIT specialization / CuTeDSL 内核与 JIT 特化
- Blackwell GPU features and scheduling / Blackwell GPU 特性与调度
- Tiled GEMM and MMA mapping / 分块 GEMM 与 MMA 映射
- Asynchronous GMEM↔SMEM transfer / GMEM↔SMEM 异步传输
- Tensor layout transforms and tiling / 张量布局变换与分块
- Framework interoperability and export / 框架互操作与导出

## Dependencies / 依赖项

- `traceback` — used by this example / 供该示例使用
- `typing` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `cuda.bindings.driver` — exposes CUDA driver-level stream and launch types / 提供 CUDA Driver 层的流与启动类型
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `common_dense_gemm_efc.DenseGemmEFC` — used by this example / 供该示例使用
- `common_efc` — used by this example / 供该示例使用
