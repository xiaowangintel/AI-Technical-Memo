# activation_custom_epilogue_dense_gemm.py — Code Analysis / 代码分析

**Source / 源文件**: `examples/python/CuTeDSL/cute/blackwell/efc/activation_custom_epilogue_dense_gemm.py`  
**Purpose / 用途**: Kernel example implementing activation custom epilogue dense gemm with CuTeDSL. / 这是一个使用 CuTeDSL 实现 activation custom epilogue dense gemm 的内核示例，重点说明 GPU 执行与数据搬运逻辑。

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

### Lines 29-32 / 第 29-32 行

~~~~python
import traceback
import typing

import cuda.bindings.driver as cuda
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 34-34 / 第 34-34 行

~~~~python
# Required for pre-Python 3.12 instead of typing.override.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 35-43 / 第 35-43 行

~~~~python
from typing_extensions import override
import torch

import cutlass
import cutlass.cute.testing as testing
import cutlass.torch as cutlass_torch

from common_dense_gemm_efc import DenseGemmEFC
from common_efc import ACTIVATION_FUNCTIONS
~~~~

**EN**: Imports the Python, CUTLASS, and CuTeDSL symbols used later. These modules provide kernel decorators, tensor/layout utilities, low-level GPU primitives, and framework interop helpers.
**CN**: 导入后续会用到的 Python、CUTLASS 与 CuTeDSL 符号。这些模块提供内核装饰器、张量/布局工具、底层 GPU 原语以及框架互操作辅助函数。

### Lines 45-140 / 第 45-140 行

~~~~python
"""
A high-performance persistent batched dense GEMM with activation functions in custom epilogue fusion
for the NVIDIA Blackwell SM100 architecture using CUTE DSL and Epilogue Fusion Configuration (EFC).

This example demonstrates GEMMs with custom fused epilogues inspired by Ada FP8 GEMM epilogue
from https://github.com/NVIDIA/cutlass/blob/main/examples/58_ada_fp8_gemm/ada_fp8_gemm.cu :
  Aux = ((alpha * scale_a * scale_b) * accumulator) + ((beta * scale_c) * source) + bias
  D = activation(Aux)

The scale factors (scale_a, scale_b, scale_c) default to 1.0 but can be customized via CLI:

Tensor dimensions:
- Matrix A is MxKxL, L is batch dimension, A can be row-major("K") or column-major("M")
- Matrix B is NxKxL, L is batch dimension, B can be row-major("N") or column-major("K")
- Matrix C is MxNxL (read-only input, "source"), C can be row-major("N") or column-major("M")
- Matrix Aux is MxNxL (auxiliary output, pre-activation), same layout as C/D
- Matrix D is MxNxL (final output, post-activation), same layout as C
- alpha, beta are scalar scale factors
- scale_a, scale_b, scale_c are scalar scale factors for A, B, and C matrices
- bias is a scalar bias term

This GEMM kernel supports the following features:
    - Utilizes Tensor Memory Access (TMA) for efficient memory operations
    - Utilizes Blackwell's tcgen05.mma for matrix multiply-accumulate (MMA) operations (including 2cta mma instructions)
    - Implements TMA multicast with cluster to reduce L2 memory traffic
    - Supports persistent tile scheduling to better overlap memory load/store with mma between tiles
    - Supports warp specialization to avoid explicit pipelining between mainloop load and mma
    - Uses Epilogue Fusion Configuration (EFC) to define custom epilogue operations with activation functions

Supported activation functions:
    - identity: f(x) = x
    - relu: f(x) = max(0, x)
    - leaky_relu: f(x) = max(0, x) + negative_slope * min(0, x)
    - tanh: f(x) = tanh(x)
    - sigmoid: f(x) = 1 / (1 + exp(-x))
    - silu: f(x) = x * sigmoid(x)
    - hardswish: f(x) = x * relu6(x + 3) / 6
    - gelu: f(x) = 0.5 * x * (1 + erf(x / sqrt(2)))

This GEMM works as follows:
1. DMA warp: Load A and B matrices from global memory (GMEM) to shared memory (SMEM) using TMA operations.
2. MMA warp: Perform matrix multiply-accumulate (MMA) operations using tcgen05.mma instruction.
3. EPILOGUE warp (defined via EFC):
    - Load completed accumulator from tensor memory (TMEM) to registers (RMEM) using tcgen05.ld.
    - Load C (source) matrix from global memory (GMEM) to shared memory (SMEM) using TMA, then to registers (RMEM).
    - Compute Aux = (alpha * scale_a * scale_b) * accumulator + (beta * scale_c) * C + bias
    - Compute D = activation(Aux)
    - Type convert Aux and D matrices to output types.
    - Store Aux and D matrices from registers (RMEM) to shared memory (SMEM) to global memory (GMEM) with TMA operations

SM100 tcgen05.mma instructions operate as follows:
- Read matrix A from SMEM
- Read matrix B from SMEM
- Write accumulator to TMEM
The accumulator in TMEM must then be loaded to registers before writing back to GMEM.

Example usage:

.. code-block:: bash

    python activation_custom_epilogue_dense_gemm.py \
      --activation relu \
      --ab_dtype Float16 --c_dtype Float16 --aux_dtype Float16 --d_dtype Float16 \
      --acc_dtype Float32 --epi_dtype Float32 \
      --mma_tiler_mn 128,128 --cluster_shape_mn 2,1 \
      --mnkl 8192,8192,8192,1 \
      --use_2cta_instrs --alpha 1.0 --beta 1.0 --bias 0.0 \
      --scale_a 1.0 --scale_b 1.0 --scale_c 1.0

To collect performance with NCU profiler:

.. code-block:: bash

    ncu python activation_custom_epilogue_dense_gemm.py \
      --activation gelu \
      --ab_dtype Float16 --c_dtype Float16 --aux_dtype Float16 --d_dtype Float16 \
      --acc_dtype Float32 --epi_dtype Float32 \
      --mma_tiler_mn 128,128 --cluster_shape_mn 2,1 \
      --mnkl 8192,8192,8192,1 \
      --use_2cta_instrs --alpha 1.0 --beta 1.0 --bias 0.0 \
      --scale_a 1.0 --scale_b 1.0 --scale_c 1.0 \
      --warmup_iterations 1 --iterations 10 --skip_ref_check

Constraints:
* Supported input data types: fp16, bf16, tf32, int8, uint8, fp8 (e4m3fn, e5m2)
* A/B tensors must have the same data type
* C/D/Aux tensors must have the same major order
* MMA tiler M must be 64/128 (use_2cta_instrs=False) or 128/256 (use_2cta_instrs=True)
* MMA tiler N must be 32-256, step 32
* Cluster shape M/N must be positive and power of 2, total cluster size <= 16
* Cluster shape M must be multiple of 2 if use_2cta_instrs=True
* The contiguous dimension of all tensors must be at least 16 bytes aligned,
  i.e, number of elements is a multiple of 4, 8, and 16 for TFloat32,
  Float16/BFloat16, and Int8/Uint8/Float8, respectively.
* OOB tiles are not allowed when TMA store is disabled
"""
~~~~

**EN**: Provides module-level narrative: what the example demonstrates, how to run it, and what GPU/DSL ideas the reader should focus on.
**CN**: 提供模块级说明：这个示例演示什么、如何运行，以及读者应关注哪些 GPU/DSL 概念。

### Lines 143-156 / 第 143-156 行

~~~~python
class DenseGemmActivation(DenseGemmEFC):
    """Implements batched GEMM with activation function in epilogue using EFC.

    This class extends DenseGemmEFC to provide a fused epilogue inspired by
    Ada FP8 GEMM that:
    - Reads from input tensor C (source)
    - Writes to output tensors Aux (auxiliary, pre-activation) and D (final, post-activation)
    - Performs: Aux = alpha * accumulator + beta * C + bias
                D = activation(Aux)

    The class provides CLI argument parsing and tensor creation for the
    specific epilogue configuration with C, Aux, D tensors and alpha,
    beta, bias scalar parameters.
    """
~~~~

**EN**: Defines `DenseGemmActivation`, a reusable Python class that packages configuration and behavior for this example. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `DenseGemmActivation`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 158-185 / 第 158-185 行

~~~~python
    def __init__(
        self,
        acc_dtype,
        epi_dtype,
        use_2cta_instrs,
        mma_tiler_mn,
        cluster_shape_mn,
        epilogue_fn,
        activation_name,
    ):
        """Initialize the GEMM with activation epilogue.

        :param acc_dtype: Accumulator data type
        :param epi_dtype: Epilogue computation data type
        :param use_2cta_instrs: Whether to use 2-CTA MMA instructions
        :param mma_tiler_mn: MMA tile shape (M, N)
        :param cluster_shape_mn: Cluster shape (M, N)
        :param epilogue_fn: Epilogue function to use
        :param activation_name: Name of the activation function
        """
        super().__init__(
            acc_dtype,
            epi_dtype,
            use_2cta_instrs,
            mma_tiler_mn,
            cluster_shape_mn,
            epilogue_fn,
        )
~~~~

**EN**: Defines `__init__`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `__init__`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 186-186 / 第 186-186 行

~~~~python
        self.activation_name = activation_name
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 188-215 / 第 188-215 行

~~~~python
    class CLIParser(DenseGemmEFC.CLIParser):
        @override
        def more_parsing(self):
            self.parser.add_argument(
                "--activation",
                type=str,
                default="relu",
                choices=ACTIVATION_FUNCTIONS,
                help="Activation function to use in epilogue",
            )
            self.parser.add_argument(
                "--alpha",
                type=float,
                default=1.0,
                help="alpha scale factor for accumulator",
            )
            self.parser.add_argument(
                "--beta", type=float, default=1.0, help="beta scale factor for source"
            )
            self.parser.add_argument(
                "--bias", type=float, default=0.0, help="bias term to add"
            )
            self.parser.add_argument(
                "--scale_a", type=float, default=1.0, help="scale factor for matrix A"
            )
            self.parser.add_argument(
                "--scale_b", type=float, default=1.0, help="scale factor for matrix B"
            )
~~~~

**EN**: Defines `CLIParser`, a reusable Python class that packages configuration and behavior for this example. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `CLIParser`，这是一个可复用的 Python 类，用来封装该示例的配置与行为。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 216-242 / 第 216-242 行

~~~~python
            self.parser.add_argument(
                "--scale_c", type=float, default=1.0, help="scale factor for source C"
            )
            self.parser.add_argument(
                "--c_dtype",
                type=cutlass.dtype,
                default=cutlass.Float32,
                help="C tensor dtype",
            )
            self.parser.add_argument(
                "--aux_dtype",
                type=cutlass.dtype,
                default=cutlass.Float32,
                help="Aux tensor dtype",
            )
            self.parser.add_argument(
                "--d_dtype",
                type=cutlass.dtype,
                default=cutlass.Float32,
                help="D tensor dtype",
            )
            self.parser.add_argument(
                "--leaky_relu_alpha",
                type=float,
                default=0.01,
                help="negative slope for leaky_relu",
            )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Builds the command-line interface used to configure and launch the example from Python. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 构建命令行接口，用于从 Python 配置并启动该示例。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 244-271 / 第 244-271 行

~~~~python
    @override
    def create_arguments(
        self,
        l,
        m,
        n,
        k,
        a_major,
        b_major,
        cd_major,
        ab_dtype,
        # For the supplemental tensors.
        c_dtype,
        aux_dtype,
        d_dtype,
    ):
        """Create arguments for GEMM operations with epilogue tensors.

        Creates tensors for A, B (from parent class) and epilogue-specific
        tensors C, Aux, D with appropriate data types and layouts.

        :return: Tuple of (a_tensor, b_tensor, a_torch_cpu, b_torch_cpu,
                           c_tensor, c_torch_cpu, c_torch_gpu,
                           aux_tensor, aux_torch_cpu, aux_torch_gpu,
                           d_tensor, d_torch_cpu, d_torch_gpu)
        """
        # Get standard arguments from parent class
        std_args = super().create_arguments(
~~~~

**EN**: Defines `create_arguments`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `create_arguments`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 272-273 / 第 272-273 行

~~~~python
            l, m, n, k, a_major, b_major, cd_major, ab_dtype
        )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 275-275 / 第 275-275 行

~~~~python
        # Create C tensor (source for epilogue)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 276-279 / 第 276-279 行

~~~~python
        c_torch_cpu = cutlass_torch.matrix(l, m, n, cd_major == "m", c_dtype)
        c_tensor, c_torch_gpu = cutlass_torch.cute_tensor_like(
            c_torch_cpu, c_dtype, is_dynamic_layout=True, assumed_align=16
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 281-281 / 第 281-281 行

~~~~python
        # Create Aux tensor (auxiliary/pre-activation output)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 282-285 / 第 282-285 行

~~~~python
        aux_torch_cpu = cutlass_torch.matrix(l, m, n, cd_major == "m", aux_dtype)
        aux_tensor, aux_torch_gpu = cutlass_torch.cute_tensor_like(
            aux_torch_cpu, aux_dtype, is_dynamic_layout=True, assumed_align=16
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 287-287 / 第 287-287 行

~~~~python
        # Create D tensor (final/post-activation output)
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 288-291 / 第 288-291 行

~~~~python
        d_torch_cpu = cutlass_torch.matrix(l, m, n, cd_major == "m", d_dtype)
        d_tensor, d_torch_gpu = cutlass_torch.cute_tensor_like(
            d_torch_cpu, d_dtype, is_dynamic_layout=True, assumed_align=16
        )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 293-304 / 第 293-304 行

~~~~python
        return (
            *std_args,
            c_tensor,
            c_torch_cpu,
            c_torch_gpu,
            aux_tensor,
            aux_torch_cpu,
            aux_torch_gpu,
            d_tensor,
            d_torch_cpu,
            d_torch_gpu,
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Returns the constructed object or computed result to the caller.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 把构造好的对象或计算结果返回给调用方。

### Lines 306-333 / 第 306-333 行

~~~~python
    def compare(
        self,
        a_torch_cpu,
        b_torch_cpu,
        epi_dtype,
        tolerance,
        # For the tensor check.
        c_torch_gpu,
        aux_torch_gpu,
        d_torch_gpu,
        # The EFC epilogue arguments.
        c_torch_cpu,
        aux_torch_cpu,
        alpha,
        beta,
        bias,
        scale_a,
        scale_b,
        scale_c,
        d_torch_cpu,
        leaky_relu_alpha=0.01,
    ):
        """Compare GPU results against CPU reference implementation.

        :param a_torch_cpu: Input tensor A on CPU
        :param b_torch_cpu: Input tensor B on CPU
        :param epi_dtype: Epilogue data type
        :param tolerance: Comparison tolerance
~~~~

**EN**: Defines `compare`, grouping related logic behind a named Python callable so the example stays modular and reusable. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `compare`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 334-347 / 第 334-347 行

~~~~python
        :param c_torch_gpu: GPU result for C
        :param aux_torch_gpu: GPU result for Aux
        :param d_torch_gpu: GPU result for D
        :param c_torch_cpu: CPU reference for C
        :param aux_torch_cpu: CPU reference for Aux
        :param alpha: Alpha scale factor
        :param beta: Beta scale factor
        :param bias: Bias term
        :param scale_a: Scale factor for matrix A
        :param scale_b: Scale factor for matrix B
        :param scale_c: Scale factor for source C
        :param d_torch_cpu: CPU reference for D
        :param leaky_relu_alpha: Negative slope for leaky_relu
        """
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 348-374 / 第 348-374 行

~~~~python
        # Compute reference result
        self.evaluate_on_cpu(
            a_torch_cpu,
            b_torch_cpu,
            epi_dtype,
            c_torch_cpu,
            aux_torch_cpu,
            alpha,
            beta,
            bias,
            scale_a,
            scale_b,
            scale_c,
            d_torch_cpu,
            leaky_relu_alpha,
        )
        # Assert close results for output tensors
        torch.testing.assert_close(
            aux_torch_gpu.cpu(), aux_torch_cpu, atol=tolerance, rtol=1e-03
        )
        torch.testing.assert_close(
            d_torch_gpu.cpu(), d_torch_cpu, atol=tolerance, rtol=1e-03
        )
        # Assert that the read tensor has not been changed
        torch.testing.assert_close(
            c_torch_gpu.cpu(), c_torch_cpu, atol=tolerance, rtol=1e-03
        )
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 376-403 / 第 376-403 行

~~~~python
    @staticmethod
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
        c_dtype: typing.Type[cutlass.Numeric],
        aux_dtype: typing.Type[cutlass.Numeric],
        d_dtype: typing.Type[cutlass.Numeric],
        alpha: float,
        beta: float,
        bias: float,
        scale_a: float,
        scale_b: float,
        scale_c: float,
        activation: str,
        leaky_relu_alpha: float,
        tolerance: float,
    ) -> str:
        """Format test parameters as CLI arguments for activation_custom_epilogue_dense_gemm.py

        Formats all test parameters into a CLI command that can be directly
~~~~

**EN**: Defines `format_as_cli_args`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `format_as_cli_args`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 404-424 / 第 404-424 行

~~~~python
        copy-pasted to reproduce the test case. Includes base parameters from
        DenseGemmEFC and epilogue-specific parameters (c_dtype, aux_dtype, d_dtype,
        alpha, beta, bias, activation).

        :return: Formatted CLI command string
        """
        # Get base command from parent class
        base_cmd = DenseGemmEFC.format_as_cli_args(
            "activation_custom_epilogue_dense_gemm.py",
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

### Lines 426-426 / 第 426-426 行

~~~~python
        # Add epilogue-specific arguments
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 427-439 / 第 427-439 行

~~~~python
        epilogue_args = (
            f" --activation {activation}"
            f" --c_dtype {c_dtype.__name__}"
            f" --aux_dtype {aux_dtype.__name__}"
            f" --d_dtype {d_dtype.__name__}"
            f" --alpha {alpha}"
            f" --beta {beta}"
            f" --bias {bias}"
            f" --scale_a {scale_a}"
            f" --scale_b {scale_b}"
            f" --scale_c {scale_c}"
            f" --leaky_relu_alpha {leaky_relu_alpha}"
        )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 441-441 / 第 441-441 行

~~~~python
        return base_cmd + epilogue_args
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 444-452 / 第 444-452 行

~~~~python
def create_epilogue_function(activation_name: str):
    """Create an epilogue function with the specified activation.

    :param activation_name: Name of the activation function to use
    :return: Epilogue function
    """
    # Validate activation name
    if activation_name not in ACTIVATION_FUNCTIONS:
        raise ValueError(f"Unsupported activation: {activation_name}")
~~~~

**EN**: Defines `create_epilogue_function`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 定义 `create_epilogue_function`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 454-481 / 第 454-481 行

~~~~python
    def epilogue(
        efc_config,
        C,
        Aux,
        alpha,
        beta,
        bias,
        scale_a,
        scale_b,
        scale_c,
        D,
        leaky_relu_alpha,
    ):
        # Aux = ((alpha * scale_a * scale_b) * accumulator) + ((beta * scale_c) * source) + bias
        # Following Ada FP8 GEMM epilogue pattern
        aux_val = (
            (alpha * scale_a * scale_b) * efc_config.accum()
            + (beta * scale_c) * C.load()
            + bias
        )
        Aux.store(aux_val)
        # D = activation(Aux)
        activation_fn = getattr(efc_config, activation_name)
        # leaky_relu needs an extra parameter, others don't
        if activation_name == "leaky_relu":
            D.store(activation_fn(aux_val, leaky_relu_alpha))
        else:
            D.store(activation_fn(aux_val))
~~~~

**EN**: Defines `epilogue`, grouping related logic behind a named Python callable so the example stays modular and reusable. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `epilogue`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 483-483 / 第 483-483 行

~~~~python
    return epilogue
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Returns the constructed object or computed result to the caller.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 把构造好的对象或计算结果返回给调用方。

### Lines 486-513 / 第 486-513 行

~~~~python
def run(
    mnkl: typing.Tuple[int, int, int, int],
    ab_dtype: typing.Type[cutlass.Numeric],
    acc_dtype: typing.Type[cutlass.Numeric],
    epi_dtype: typing.Type[cutlass.Numeric],
    a_major: str,
    b_major: str,
    cd_major: str,
    c_dtype: typing.Type[cutlass.Numeric],
    aux_dtype: typing.Type[cutlass.Numeric],
    d_dtype: typing.Type[cutlass.Numeric],
    alpha: float,
    beta: float,
    bias: float,
    scale_a: float,
    scale_b: float,
    scale_c: float,
    activation: str,
    leaky_relu_alpha: float,
    mma_tiler_mn: typing.Tuple[int, int],
    cluster_shape_mn: typing.Tuple[int, int],
    use_2cta_instrs: bool,
    tolerance: float,
    warmup_iterations: int = 3,
    iterations: int = 100,
    skip_ref_check: bool = False,
):
    """Run GEMM with activation function in epilogue.
~~~~

**EN**: Defines `run`, grouping related logic behind a named Python callable so the example stays modular and reusable. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 定义 `run`，把相关逻辑组织到一个具名的 Python 可调用对象中，使示例保持模块化并便于复用。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 514-541 / 第 514-541 行

~~~~python

    :param mnkl: Tuple of (M, N, K, L) dimensions
    :param ab_dtype: Data type for A and B tensors
    :param acc_dtype: Accumulator data type
    :param epi_dtype: Epilogue computation data type
    :param a_major: Major dimension for A ("m" or "k")
    :param b_major: Major dimension for B ("n" or "k")
    :param cd_major: Major dimension for C/D/Aux ("m" or "n")
    :param c_dtype: Data type for C tensor
    :param aux_dtype: Data type for Aux tensor
    :param d_dtype: Data type for D tensor
    :param alpha: Alpha scale factor
    :param beta: Beta scale factor
    :param bias: Bias term
    :param scale_a: Scale factor for matrix A
    :param scale_b: Scale factor for matrix B
    :param scale_c: Scale factor for source C
    :param activation: Activation function name
    :param leaky_relu_alpha: Negative slope for leaky_relu
    :param mma_tiler_mn: MMA tile shape (M, N)
    :param cluster_shape_mn: Cluster shape (M, N)
    :param use_2cta_instrs: Whether to use 2-CTA MMA instructions
    :param tolerance: Comparison tolerance
    :param warmup_iterations: Number of warmup iterations
    :param iterations: Number of benchmark iterations
    :param skip_ref_check: Whether to skip reference check
    """
    print("Running Blackwell Persistent Dense GEMM test with:")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 542-559 / 第 542-559 行

~~~~python
    print(f"mnkl: {mnkl}")
    print(f"AB dtype: {ab_dtype}, Acc dtype: {acc_dtype}, Epi dtype: {epi_dtype}")
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
    print(f"\t{c_dtype = !s}, {aux_dtype = !s}, {d_dtype = !s}")
    print(f"\t{alpha = }, {beta = }, {bias = }")
    print(f"\t{scale_a = }, {scale_b = }, {scale_c = }")
    print(f"\t{activation = !s}")
    if activation == "leaky_relu":
        print(f"\t{leaky_relu_alpha = }")
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 561-561 / 第 561-561 行

~~~~python
    # Unpack parameters
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 562-562 / 第 562-562 行

~~~~python
    m, n, k, l = mnkl
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 564-565 / 第 564-565 行

~~~~python
    if not torch.cuda.is_available():
        raise RuntimeError("GPU is required to run this example!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Adds validation or error handling so unsupported layouts, types, or runtime conditions fail early with a clear message.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 加入校验或错误处理，使不支持的布局、类型或运行时条件能够尽早并清晰地报错。

### Lines 567-567 / 第 567-567 行

~~~~python
    # Get current CUDA stream from PyTorch
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 568-570 / 第 568-570 行

~~~~python
    torch_stream = torch.cuda.current_stream()
    # Get the raw stream pointer as a CUstream
    current_stream = cuda.CUstream(torch_stream.cuda_stream)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。

### Lines 572-572 / 第 572-572 行

~~~~python
    # Create the epilogue function with the specified activation
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 573-573 / 第 573-573 行

~~~~python
    epilogue_fn = create_epilogue_function(activation)
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 575-575 / 第 575-575 行

~~~~python
    # Build GEMM object with EFC configuration
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 576-584 / 第 576-584 行

~~~~python
    gemm = DenseGemmActivation(
        acc_dtype,
        epi_dtype,
        use_2cta_instrs,
        mma_tiler_mn,
        cluster_shape_mn,
        epilogue_fn,
        activation,
    )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 586-613 / 第 586-613 行

~~~~python
    (
        a_tensor,
        b_tensor,
        a_torch_cpu,
        b_torch_cpu,
        # The supplemental tensors.
        c_tensor,
        c_torch_cpu,
        c_torch_gpu,
        aux_tensor,
        aux_torch_cpu,
        aux_torch_gpu,
        d_tensor,
        d_torch_cpu,
        d_torch_gpu,
    ) = gemm.create_arguments(
        l,
        m,
        n,
        k,
        a_major,
        b_major,
        cd_major,
        ab_dtype,
        # For the supplemental tensors.
        c_dtype,
        aux_dtype,
        d_dtype,
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 614-614 / 第 614-614 行

~~~~python
    )
~~~~

**EN**: Implements a piece of the device-side kernel logic, contributing to how one CTA, warp, or thread processes its assigned tile. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现设备端内核逻辑的一部分，决定一个 CTA、warp 或线程如何处理分配给它的 tile。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 616-617 / 第 616-617 行

~~~~python
    # Check if the configuration can be implemented. Raise a ValueError
    # otherwise.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 618-618 / 第 618-618 行

~~~~python
    gemm.check_implementable(a_tensor, b_tensor, d_tensor)
~~~~

**EN**: Implements attention-style dataflow, where Q/K/V tiles, score accumulation, and normalization are carefully scheduled on the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 实现注意力式数据流，其中 Q/K/V 分块、分数累积和归一化都会被精细地调度到 GPU 上。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 620-622 / 第 620-622 行

~~~~python
    max_active_clusters = cutlass.utils.HardwareInfo().get_max_active_clusters(
        cluster_shape_mn[0] * cluster_shape_mn[1]
    )
~~~~

**EN**: Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 624-641 / 第 624-641 行

~~~~python
    compiled_gemm = gemm.compile(
        a_tensor,
        b_tensor,
        max_active_clusters,
        current_stream,
        # Here are the supplemental arguments in the same order as for the
        # epilogue configuration function.
        c_tensor,
        aux_tensor,
        alpha,
        beta,
        bias,
        scale_a,
        scale_b,
        scale_c,
        d_tensor,
        leaky_relu_alpha,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 643-659 / 第 643-659 行

~~~~python
    compiled_gemm(
        a_tensor,
        b_tensor,
        current_stream,
        # Here are the supplemental arguments in the same order as for the
        # epilogue configuration function.
        c_tensor,
        aux_tensor,
        alpha,
        beta,
        bias,
        scale_a,
        scale_b,
        scale_c,
        d_tensor,
        leaky_relu_alpha,
    )
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 661-661 / 第 661-661 行

~~~~python
    # TODO: unify with modern way to do benchmarking.
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 662-684 / 第 662-684 行

~~~~python
    exec_time = testing.benchmark(
        compiled_gemm,
        kernel_arguments=testing.JitArguments(
            a_tensor,
            b_tensor,
            current_stream,
            # Here are the supplemental arguments in the same order as for the
            # epilogue configuration function.
            c_tensor,
            aux_tensor,
            alpha,
            beta,
            bias,
            scale_a,
            scale_b,
            scale_c,
            d_tensor,
            leaky_relu_alpha,
        ),
        stream=current_stream,
        warmup_iterations=warmup_iterations,
        iterations=iterations,
    )
~~~~

**EN**: Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 686-686 / 第 686-686 行

~~~~python
    print(f"Execution time: {exec_time} us")
~~~~

**EN**: Prints progress, diagnostics, or benchmark results so users can see what the example is doing. From a CUDA perspective, this contributes to how the CTA/warp/thread team handles its tile of work.
**CN**: 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。 从 CUDA 视角看，这一段决定了 CTA/warp/thread 这一协作单元如何处理自己负责的 tile。

### Lines 688-688 / 第 688-688 行

~~~~python
    # Compute reference result
~~~~

**EN**: Uses comments to introduce the next algorithmic section and guide the reader through the example.
**CN**: 用注释引出接下来的算法部分，引导读者理解示例。

### Lines 689-713 / 第 689-713 行

~~~~python
    if not skip_ref_check:
        print("Checking results against CPU reference...")
        gemm.compare(
            # The usual arguments.
            a_torch_cpu,
            b_torch_cpu,
            epi_dtype,
            tolerance,
            # For the tensor check.
            c_torch_gpu,
            aux_torch_gpu,
            d_torch_gpu,
            # The EFC epilogue arguments.
            c_torch_cpu,
            aux_torch_cpu,
            alpha,
            beta,
            bias,
            scale_a,
            scale_b,
            scale_c,
            d_torch_cpu,
            leaky_relu_alpha,
        )
        print("Results match CPU reference!")
~~~~

**EN**: Bridges CuTeDSL with framework tensors so the compiled kernel can consume PyTorch/JAX data without extra copies. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 把 CuTeDSL 与框架张量连接起来，使编译后的内核能在不额外拷贝的情况下使用 PyTorch/JAX 数据。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

### Lines 716-717 / 第 716-717 行

~~~~python
if __name__ == "__main__":
    args = DenseGemmActivation.CLIParser().parse()
~~~~

**EN**: Provides the standard Python entry point so the example can be executed directly as a script. Sets up configuration values, tensor shapes, data types, or runtime knobs that control how the example maps work onto the GPU. Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 提供标准的 Python 程序入口，使该示例可以直接作为脚本运行。 设置配置值、张量形状、数据类型或运行时参数，用来控制示例如何把工作映射到 GPU。 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 719-746 / 第 719-746 行

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
            args.c_dtype,
            args.aux_dtype,
            args.d_dtype,
            args.alpha,
            args.beta,
            args.bias,
            args.scale_a,
            args.scale_b,
            args.scale_c,
            args.activation,
            args.leaky_relu_alpha,
            args.mma_tiler_mn,
            args.cluster_shape_mn,
            args.use_2cta_instrs,
            args.tolerance,
            args.warmup_iterations,
            args.iterations,
            args.skip_ref_check,
        )
~~~~

**EN**: Issues matrix multiply-accumulate work and maps the DSL object model onto the underlying GPU MMA instructions. Measures performance or checks correctness so the example doubles as a validation harness. Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels.
**CN**: 发起矩阵乘加计算，并把 DSL 对象模型映射到底层 GPU MMA 指令。 测量性能或检查正确性，使该示例同时充当验证工具。 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。

### Lines 747-754 / 第 747-754 行

~~~~python
        print("\n" + "=" * 80)
        print(
            f"PASS - {args.activation.upper()} activation test completed successfully!"
        )
        print("=" * 80 + "\n")
    except Exception as exc:
        traceback.print_exception(exc)
        raise
~~~~

**EN**: Highlights numerics, scaling, or epilogue behavior, which is especially important for fused or low-precision kernels. Prints progress, diagnostics, or benchmark results so users can see what the example is doing.
**CN**: 突出数值、缩放或尾处理行为，这对融合内核或低精度内核尤其重要。 打印进度、诊断信息或基准结果，让用户能够看到示例正在执行什么。

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
- `typing_extensions.override` — documents Python-side types for readability and tooling / 为可读性和工具链提供 Python 类型标注
- `torch` — creates CUDA tensors and validates results on the host side / 在宿主端创建 CUDA 张量并验证结果
- `cutlass` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `cutlass.cute.testing` — offers benchmark and correctness utilities / 提供基准测试与正确性检查工具
- `cutlass.torch` — provides numeric types, compilation helpers, and CUTLASS integration / 提供数值类型、编译辅助工具以及 CUTLASS 集成
- `common_dense_gemm_efc.DenseGemmEFC` — used by this example / 供该示例使用
- `common_efc.ACTIVATION_FUNCTIONS` — used by this example / 供该示例使用
