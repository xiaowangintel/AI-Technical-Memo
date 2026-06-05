# 10-block-scaled-matmul.py — Code Analysis / 代码分析
## Source / 来源
- **Source file / 源文件:** `/root/xw/triton/python/tutorials/10-block-scaled-matmul.py`
- **Analysis file / 分析文件:** `/root/xw/triton/tutorial/python/tutorials/10-block-scaled-matmul.py.md`
- **Scope / 范围:** This note analyzes the tutorial from the opening documentation through backend detection, kernel logic, initialization, validation, benchmarking, AMD-specific handling, and CLI control flow. / 本文从开头文档说明一路分析到后端检测、内核逻辑、初始化、校验、基准测试、AMD 专用路径以及命令行控制流程。
- **Snippet policy / 代码片段说明:** Each code block below is copied verbatim from the source for the indicated line range. / 下方每个代码块都按对应行号范围从源文件中原样摘录。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24 — Tutorial overview / 教程概览
```python
"""
Block Scaled Matrix Multiplication
==================================
This tutorial demonstrates a Triton implementation of block scaled matrix multiplication
which is generic over FP4 and FP8 formats on NVIDIA and AMD GPUs.
The tutorial supports OCP microscaling formats such as mxfp4 and mxfp8, and NVIDIA's nvfp4
(on NVIDIA GPUs) and mxfp4 (on AMD GPUs). These matrix multiplications are hardware-accelerated
using fifth-generation Tensor Cores on NVIDIA GPUs with compute capability 10, and by the CDNA4
matrix cores on AMD GPUs.
Users can run the tutorial with each of the supported formats by passing the `--format`
argument and can benchmark the performance of each by specifying matrix dimensions
and iteration steps.

.. code-block:: bash

    # FP4
    python 10-block-scaled-matmul.py --format nvfp4
    python 10-block-scaled-matmul.py --format mxfp4 --K_range 512 8192 --bench

    # FP8
    python 10-block-scaled-matmul.py --format mxfp8 --K_range 8192 16384 --K_step 2048 --bench

Future updates to this tutorial which support mixed precision block scaled matmul are planned.
"""
```
**EN:** The module docstring states the tutorial's scope: block-scaled GEMM for FP4/FP8 on NVIDIA and AMD, the supported microscaling formats, example CLI invocations, and the plan to extend toward mixed-precision support.

**CN:** 模块文档字符串先给出教程范围：它演示了面向 NVIDIA 与 AMD 的 FP4/FP8 块缩放 GEMM，列出支持的微缩放格式、示例命令行用法，以及未来继续扩展到混合精度的方向。

### Lines 26-68 — NVIDIA scale-layout background / NVIDIA 缩放布局背景
```python
# %%
# Background
# ----------
# Scale preshuffling on NVIDIA GPUs
#
# CUDA devices that support PTX 8.7 and later can utlize block scaled matrix multiply
# instructions. In order for low latency access to these scale factors in the fast
# inner loop over tensor core MMAs, it is important to ensure that the blocked
# scale factors are stored in a contiguous memory layout according to their access
# pattern.
#
# The block scaled matmul tensor core instructions compute the following product:
#
#     C = (A * scale_a) @ (B * scale_b)
#
# where scale_a and scale_b are the blocked scale factors for the A and B matrices.
# Under block scaled matmul, each scale factor is broadcast and multiplied across a
# vector of elements from the A and B matrices, usually along their respective K axes.
# The number of elements of A and B over which each scale factor is broadcast is herein
# refered to as the vector size (VEC_SIZE).
#
# In a linear row-major layout, the scale factors would take the shape
#
#     (M, K // VEC_SIZE) and (N, K // VEC_SIZE)   [1]
#
# in global memory. However, to avoid non-contiguous memory access, it is beneficial to
# instead store the scale factors in a packed block layout. For the LHS matrix this layout
# is given by
#
#     (M // 32 // 4, K // VEC_SIZE // 4, 32, 4, 4)   [2].
#
# In this way, each tensor core MMA in the fast inner loop over K blocks can achieve contiguous
# access of a block of 128 rows of scale factors along the M axis, for each BLOCK_M x BLOCK_K
# subtile of the matrix A.
#
# In order to conform with Triton's language semantics for dot_scaled, the scale factors
# are prepared in the above 5D layout [2], but are then logically transposed and reshaped into
# the 2D layout [1] expected by tl.dot_scaled.
#
# For more detailed information on the scale factor layout, see
#  1. https://docs.nvidia.com/cuda/parallel-thread-execution/#tcgen05-mma-scale-factor-a-layout-1x
#  2. https://docs.nvidia.com/cuda/cublas/#d-block-scaling-factors-layout
#
```
**EN:** This comment block explains why NVIDIA block scales are preshuffled. The core idea is that `C = (A * scale_a) @ (B * scale_b)`, but the scales must be stored in a packed layout so tensor-core inner loops can fetch contiguous scale data instead of paying for strided accesses.

**CN:** 这一段注释解释了为什么 NVIDIA 路径需要预先重排缩放因子。核心公式是 `C = (A * scale_a) @ (B * scale_b)`，但为了让 Tensor Core 内层循环以连续方式读取缩放数据，缩放张量必须采用打包后的布局，而不是简单的行主序二维布局。

### Lines 70-119 — AMD scale-shuffle background / AMD 缩放重排背景
```python
# Scale preshuffling on AMD GPUs
#
# Similar to NVIDIA GPUs, on AMD GPUs with CDNA4 architecture, scaled MFMA instructions natively
# support scaled matrix multiplication. Since it only supports OCP microscaling formats each
# scale is an 8-bit value that scales 32 elements from A or B operand tensors.
# Scales are stored as 8-bit tensors. Since MFMA instructions are warp-level instructions, that
# means that each thread provides a fixed set of operand values to MFMA instructions.
#
# For example, in an MFMA instruction with shape 16x16x128:
# - 4 threads contribute elements along the K dimension.
# - 16 threads contribute elements along the M or N dimension.
#
# From the perspective of the scales tensor, even if the K dimension is stored contiguously in
# shared memory, each thread sees its elements along K dim as strided due to interleaving with
# other threads. This striding limits the ability to load scale values using vectorized memory
# access.
#
# Our goal is to reorganize the scale tensor so that:
# 1. Each thread stores the 4 scale values it needs for 4 MFMA ops in contiguous memory.
# 2. Continuous threads access contiguous memory locations improving global memory coalescing when
# bypassing LDS, which is especially beneficial for "skinny" matmuls.
#
# We consider two MFMA cases: one with non-K dimension 16, and one with 32.
# In both, the minimum tile size for preshuffling is 32x32x256.
# For example, for a 32x256 operand tile, the corresponding scale tensor has shape 32x8,
# where each scale covers 32 elements along the K dimension.
#
# Each thread holds one scale per MFMA operation. We pack the 4 scale values
# (for 4 different MFMA ops) next to each other in memory.
#
# Case 1: mfma_scaled_16x16x128
#
# Packing order: mfma_op_0, mfma_op_2, mfma_op_1, mfma_op_3
#
#            K = 128       K = 128
#        +------------+ +------------+
#    M=16|  MFMA op 0 | |  MFMA op 1 |
#        +------------+ +------------+
#    M=16|  MFMA op 2 | |  MFMA op 3 |
#        +------------+ +------------+
#
# Case 2: mfma_scaled_32x32x64
#
# Packing order: mfma_op_0, mfma_op_1, mfma_op_2, mfma_op_3
#
#            K=64     K=64     K=64     K=64
#        +--------+ +--------+ +--------+ +--------+
#    M=32| op 0   | | op 1   | | op 2   | | op 3   |
#        +--------+ +--------+ +--------+ +--------+
```
**EN:** The AMD notes describe the same problem from the MFMA execution model. Because threads observe K-dimension scales with interleaving, the code reorganizes each 32x32x256 scale tile so every thread can read the four scales it needs contiguously; two packing cases are described for non-K dimensions 16 and 32.

**CN:** AMD 注释从 MFMA 的执行模型出发说明同一个问题：线程看到的 K 维缩放值会与其他线程交错，因此需要把每个 32x32x256 的缩放块重新组织，使每个线程都能连续读取自己需要的 4 个缩放值；这里分别给出了非 K 维为 16 和 32 的两种打包方案。

### Lines 120-149 — Imports, backend checks, and optional cuBLAS setup / 导入、后端检测与可选 cuBLAS 初始化
```python
import argparse

import torch
import triton
import triton.language as tl
import triton.profiler as proton
from triton.tools.tensor_descriptor import TensorDescriptor
from triton.tools.mxfp import MXFP4Tensor, MXScaleTensor


def is_cuda():
    return triton.runtime.driver.active.get_current_target().backend == "cuda"


def is_hip_cdna4():
    target = triton.runtime.driver.active.get_current_target()
    return target is not None and target.backend == 'hip' and target.arch == 'gfx950'


def supports_block_scaling():
    return (is_cuda() and torch.cuda.get_device_capability()[0] in [10, 11]) or is_hip_cdna4()


if is_cuda() and torch.cuda.get_device_capability()[0] in [10, 11]:
    from triton._C.libtriton import nvidia
    cublas_workspace = torch.empty(32 * 1024 * 1024, device="cuda", dtype=torch.uint8)
    cublas = nvidia.cublas.CublasLt(cublas_workspace)
else:
    cublas = None
```
**EN:** The script imports PyTorch, Triton, profiling helpers, and the MXFP tensor utilities. `is_cuda`, `is_hip_cdna4`, and `supports_block_scaling` centralize backend gating, while the conditional import creates a cuBLASLt handle only on CUDA devices with compute capability 10 or 11.

**CN:** 脚本首先导入 PyTorch、Triton、性能分析工具以及 MXFP 张量辅助类。`is_cuda`、`is_hip_cdna4` 和 `supports_block_scaling` 统一封装了后端能力判断；只有在 CUDA 且计算能力为 10 或 11 时，才会额外初始化 cuBLASLt 句柄作为基线实现。

### Lines 151-167 — Kernel launch metadata / 内核启动元数据
```python
def _matmul_launch_metadata(grid, kernel, args):
    ret = {}
    M, N, K = args["M"], args["N"], args["K"]
    kernel_name = kernel.name
    if "ELEM_PER_BYTE_A" and "ELEM_PER_BYTE_B" and "VEC_SIZE" in args:
        if args["ELEM_PER_BYTE_A"] == 1 and args["ELEM_PER_BYTE_B"] == 1:
            kernel_name += "_mxfp8"
        elif args["ELEM_PER_BYTE_A"] == 1 and args["ELEM_PER_BYTE_B"] == 2:
            kernel_name += "_mixed"
        elif args["ELEM_PER_BYTE_A"] == 2 and args["ELEM_PER_BYTE_B"] == 2:
            if args["VEC_SIZE"] == 16:
                kernel_name += "_nvfp4"
            elif args["VEC_SIZE"] == 32:
                kernel_name += "_mxfp4"
    ret["name"] = f"{kernel_name} [M={M}, N={N}, K={K}]"
    ret["flops"] = 2.0 * M * N * K
    return ret
```
**EN:** This helper annotates each launch for profiling. It derives a suffix such as `mxfp8`, `mixed`, `nvfp4`, or `mxfp4` from the packed-element configuration, then records a human-readable kernel name and the nominal FLOP count.

**CN:** 这个辅助函数为每次内核启动补充分析元数据。它根据元素打包方式和向量大小推断出 `mxfp8`、`mixed`、`nvfp4` 或 `mxfp4` 后缀，再生成可读的内核名称，并记录理论 FLOP 数。

### Lines 170-235 — Primary Triton kernel / 主 Triton 内核
```python
@triton.jit(launch_metadata=_matmul_launch_metadata)
def block_scaled_matmul_kernel(  #
        a_desc,  #
        a_scale_desc,  #
        b_desc,  #
        b_scale_desc,  #
        c_desc,  #
        M: tl.constexpr,  #
        N: tl.constexpr,  #
        K: tl.constexpr,  #
        output_type: tl.constexpr,  #
        ELEM_PER_BYTE_A: tl.constexpr,  #
        ELEM_PER_BYTE_B: tl.constexpr,  #
        VEC_SIZE: tl.constexpr,  #
        BLOCK_M: tl.constexpr,  #
        BLOCK_N: tl.constexpr,  #
        BLOCK_K: tl.constexpr,  #
        rep_m: tl.constexpr,  #
        rep_n: tl.constexpr,  #
        rep_k: tl.constexpr,  #
        NUM_STAGES: tl.constexpr,  #
):  #
    if output_type == 0:
        output_dtype = tl.float32
    elif output_type == 1:
        output_dtype = tl.float16
    elif output_type == 2:
        output_dtype = tl.float8e4nv

    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = pid_m * BLOCK_M
    offs_bn = pid_n * BLOCK_N
    offs_k_a = 0
    offs_k_b = 0
    offs_scale_m = pid_m * rep_m
    offs_scale_n = pid_n * rep_n
    offs_scale_k = 0

    MIXED_PREC: tl.constexpr = ELEM_PER_BYTE_A == 1 and ELEM_PER_BYTE_B == 2

    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = a_desc.load([offs_am, offs_k_a])
        b = b_desc.load([offs_bn, offs_k_b])
        scale_a = a_scale_desc.load([0, offs_scale_m, offs_scale_k, 0, 0])
        scale_b = b_scale_desc.load([0, offs_scale_n, offs_scale_k, 0, 0])

        scale_a = scale_a.reshape(rep_m, rep_k, 32, 4, 4).trans(0, 3, 2, 1, 4).reshape(BLOCK_M, BLOCK_K // VEC_SIZE)
        scale_b = scale_b.reshape(rep_n, rep_k, 32, 4, 4).trans(0, 3, 2, 1, 4).reshape(BLOCK_N, BLOCK_K // VEC_SIZE)

        if MIXED_PREC:
            accumulator = tl.dot_scaled(a, scale_a, "e4m3", b.T, scale_b, "e2m1", accumulator)
        elif ELEM_PER_BYTE_A == 2 and ELEM_PER_BYTE_B == 2:
            accumulator = tl.dot_scaled(a, scale_a, "e2m1", b.T, scale_b, "e2m1", accumulator)
        else:
            accumulator = tl.dot_scaled(a, scale_a, "e4m3", b.T, scale_b, "e4m3", accumulator)

        offs_k_a += BLOCK_K // ELEM_PER_BYTE_A
        offs_k_b += BLOCK_K // ELEM_PER_BYTE_B
        offs_scale_k += rep_k

    c_desc.store([offs_am, offs_bn], accumulator.to(output_dtype))
```
**EN:** This is the main CUDA-style block-scaled matmul kernel. It maps the program ID to an `(M, N)` tile, loads packed A/B operands plus their scale tiles through tensor descriptors, reshapes the 5D scale layout into the 2D form expected by `tl.dot_scaled`, chooses the correct micro-format combination, accumulates in FP32, and stores the final tile in the requested output type.

**CN:** 这是主 CUDA 风格块缩放矩阵乘内核。它把 program ID 映射到 `(M, N)` 输出块，通过张量描述符读取打包后的 A/B 操作数及其缩放块，再把 5D 缩放布局重排成 `tl.dot_scaled` 需要的 2D 形式，按具体格式选择合适的微格式组合，在 FP32 中累加，最后以指定输出类型写回结果。

### Lines 237-274 — Python launch wrapper / Python 启动包装器
```python
def block_scaled_matmul(a_desc, a_scale_desc, b_desc, b_scale_desc, dtype_dst, M, N, K, rep_m, rep_n, rep_k, configs):
    output = torch.empty((M, N), dtype=dtype_dst, device="cuda")
    if dtype_dst == torch.float32:
        dtype_dst = 0
    elif dtype_dst == torch.float16:
        dtype_dst = 1
    elif dtype_dst == torch.float8_e4m3fn:
        dtype_dst = 2
    else:
        raise ValueError(f"Unsupported dtype: {dtype_dst}")

    BLOCK_M = configs["BLOCK_SIZE_M"]
    BLOCK_N = configs["BLOCK_SIZE_N"]
    c_desc = TensorDescriptor.from_tensor(output, [BLOCK_M, BLOCK_N])

    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    block_scaled_matmul_kernel[grid](
        a_desc,
        a_scale_desc,
        b_desc,
        b_scale_desc,
        c_desc,
        M,
        N,
        K,
        dtype_dst,
        configs["ELEM_PER_BYTE_A"],
        configs["ELEM_PER_BYTE_B"],
        configs["VEC_SIZE"],
        configs["BLOCK_SIZE_M"],
        configs["BLOCK_SIZE_N"],
        configs["BLOCK_SIZE_K"],
        rep_m,
        rep_n,
        rep_k,
        configs["num_stages"],
    )
    return output
```
**EN:** The wrapper allocates the output tensor, translates a PyTorch dtype into the constexpr enum consumed by the kernel, creates the output tensor descriptor, computes the launch grid, and dispatches `block_scaled_matmul_kernel` with format-dependent configuration values.

**CN:** 这个 Python 包装器负责分配输出张量，把 PyTorch 的 dtype 映射为内核使用的 constexpr 枚举，构造输出张量描述符，计算 launch grid，并带着与格式相关的配置参数启动 `block_scaled_matmul_kernel`。

### Lines 277-320 — cuBLAS reference path / cuBLAS 参考路径
```python
def cublas_block_scaled_matmul(a, a_scale, b, b_scale, block_scale_type="mxfp8"):
    """
    cuBLAS block-scaled matmul baseline.

    Args:
        a: Input matrix A
            - For mxfp8: (M, K) in FP8 E4M3
            - For nvfp4: (M, K//2) in uint8 packed FP4 (2 elements per byte)
        a_scale: Scale factors for A
            - For mxfp8: E8M0 scales (flattened)
            - For nvfp4: FP8 E4M3 scales in cublas layout (M, K//16)
        b: Input matrix B
            - For mxfp8: (N, K) in FP8 E4M3
            - For nvfp4: (N, K//2) in uint8 packed FP4 (2 elements per byte)
        b_scale: Scale factors for B
            - For mxfp8: E8M0 scales (flattened)
            - For nvfp4: FP8 E4M3 scales in cublas layout (N, K//16)
        block_scale_type: Format type ("mxfp8" or "nvfp4")

    Returns:
        output: Result matrix (M, N) in FP16
    """
    M, K_a = a.shape
    N, K_b = b.shape

    if block_scale_type == "mxfp8":
        assert K_a == K_b, "K dimensions must match"
        assert a.dtype == torch.float8_e4m3fn, "Only FP8 E4M3 inputs supported for mxfp8"
        assert b.dtype == torch.float8_e4m3fn, "Only FP8 E4M3 inputs supported for mxfp8"
        # MXFP8 cuBLAS outputs FP16
        output = torch.empty((M, N), dtype=torch.float16, device="cuda")
        cublas.block_scaled_matmul_mxfp8(a, b, output, a_scale, b_scale)
    elif block_scale_type == "nvfp4":
        # For packed FP4, K_a and K_b are in bytes (K = K_a * 2 in elements)
        assert K_a == K_b, "K dimensions must match"
        assert a.dtype == torch.uint8, "Only uint8 packed FP4 inputs supported for nvfp4"
        assert b.dtype == torch.uint8, "Only uint8 packed FP4 inputs supported for nvfp4"
        # NVFP4 cuBLAS outputs FP16
        output = torch.empty((M, N), dtype=torch.float16, device="cuda")
        cublas.block_scaled_matmul_nvfp4(a, b, output, a_scale, b_scale)
    else:
        raise ValueError(f"Unsupported block_scale_type: {block_scale_type}")

    return output
```
**EN:** This helper provides a vendor-library baseline for formats currently supported by cuBLAS block-scaled GEMM. It validates the packed input dtypes and shapes, allocates an FP16 result, and dispatches the corresponding `block_scaled_matmul_mxfp8` or `block_scaled_matmul_nvfp4` routine.

**CN:** 这个辅助函数为当前被 cuBLAS 块缩放 GEMM 支持的格式提供厂商库基线。它先检查打包输入的 dtype 与形状，再分配 FP16 输出，并调用对应的 `block_scaled_matmul_mxfp8` 或 `block_scaled_matmul_nvfp4` 实现。

### Lines 323-398 — Data initialization and descriptor preparation / 数据初始化与描述符准备
```python
def initialize_block_scaled(M, N, K, block_scale_type="nvfp4", compute_reference=False):
    BLOCK_M = 128
    BLOCK_N = 256
    BLOCK_K = 256 if "fp4" in block_scale_type else 128
    VEC_SIZE = 16 if block_scale_type == "nvfp4" else 32
    assert block_scale_type in ["nvfp4", "mxfp4", "mxfp8", "mixed"], f"Invalid block scale type: {block_scale_type}"
    ELEM_PER_BYTE_A = 2 if "fp4" in block_scale_type else 1
    ELEM_PER_BYTE_B = 1 if block_scale_type == "mxfp8" else 2

    device = "cuda"
    a_ref = MXFP4Tensor(size=(M, K), device=device).random()
    # Similar to Hopper's wgmma symmetric fp8 instruction, the RHS is expected
    # to be in col-major layout for Blackwell's tcgen05.mma when using fp4 operands.
    # To conform to the expected semantics of tl.dot_scaled, (M, K) x (K, N),
    # the data is generated in col-major layout, packed along K for fp4, and then
    # logically transposed. Note that if one operand is of fp8 precision, unlike Hopper,
    # Blackwell supports both row-major and col-major layouts for the RHS matrix.
    # For the mixed-precision case, the fp4 RHS can be either in row or col-major layout.
    # But for performance reason, it is recommended to use col-major layout. If TMA is used
    # for the fp4 RHS operand load in mixed-precision dot, as in this tutorial, it must be
    # in col-major layout.
    b_ref = MXFP4Tensor(size=(N, K), device=device).random()
    if block_scale_type in ["mxfp8", "mixed"]:
        a_ref = a_ref.to(torch.float32)
        a = a_ref.to(torch.float8_e4m3fn)
    else:
        # Pack two fp4 elements per byte along K
        a = a_ref.to_packed_tensor(dim=1)

    if block_scale_type == "mxfp8":
        b_ref = b_ref.to(torch.float32)
        b = b_ref.to(torch.float8_e4m3fn)
    else:
        b = b_ref.to_packed_tensor(dim=1)

    b_ref = b_ref.to(torch.float32).T

    a_desc = TensorDescriptor.from_tensor(a, [BLOCK_M, BLOCK_K // ELEM_PER_BYTE_A])
    b_desc = TensorDescriptor.from_tensor(b, [BLOCK_N, BLOCK_K // ELEM_PER_BYTE_B])

    a_scale_shape = [M // 128, K // VEC_SIZE // 4, 32, 16]
    b_scale_shape = [N // 128, K // VEC_SIZE // 4, 32, 16]
    epsilon = 1e-8
    a_scale = torch.rand(a_scale_shape, device=device) + epsilon
    b_scale = torch.rand(b_scale_shape, device=device) + epsilon

    # Store original scales for cublas nvfp4 before any layout conversion.
    # For cublas nvfp4, the scales are in the original 4D layout.
    a_scale_orig = a_scale.clone()
    b_scale_orig = b_scale.clone()

    if block_scale_type == "nvfp4":
        a_scale = a_scale.to(torch.float8_e4m3fn)
        b_scale = b_scale.to(torch.float8_e4m3fn)
        a_scale_ref = a_scale
        b_scale_ref = b_scale
    elif block_scale_type in ["mxfp4", "mxfp8", "mixed"]:
        a_scale_ref = MXScaleTensor(a_scale)
        b_scale_ref = MXScaleTensor(b_scale)
        a_scale = a_scale_ref.data
        b_scale = b_scale_ref.data

    rep_m = BLOCK_M // 128
    rep_n = BLOCK_N // 128
    rep_k = BLOCK_K // VEC_SIZE // 4

    # Use 5D TMA descriptor [1, rep_m, rep_k, 2, 256] with uint8 elements.
    # With 256 elements we better utilize the L2 and don't require the TMA
    # engine to emit many small messages (16B) messages as with 32x16xu8.
    a_scale_block_shape = [1, rep_m, rep_k, 2, 256]
    b_scale_block_shape = [1, rep_n, rep_k, 2, 256]
    a_scale = a_scale.reshape(1, a_scale_shape[0], a_scale.shape[1], 2, 256)
    b_scale = b_scale.reshape(1, b_scale_shape[0], b_scale.shape[1], 2, 256)
    a_scale_desc = TensorDescriptor.from_tensor(a_scale, block_shape=a_scale_block_shape)
    b_scale_desc = TensorDescriptor.from_tensor(b_scale, block_shape=b_scale_block_shape)
```
**EN:** This section chooses tile sizes and packing factors from the requested format, synthesizes random MXFP inputs, applies the special RHS layout rules for FP4, converts or packs operands, creates tensor descriptors, generates random scale tensors, and preserves an original copy of the scales when cuBLAS needs its own layout.

**CN:** 这一段根据目标格式选择分块尺寸与打包因子，生成随机 MXFP 输入，针对 FP4 应用右操作数的特殊布局规则，对输入执行转换或打包，创建张量描述符，生成随机缩放张量，并在 cuBLAS 需要原始布局时保留一份缩放副本。

### Lines 399-436 — Reference reconstruction and returned config / 参考结果重建与返回配置
```python
    reference = None
    if compute_reference:
        a_scale_ref = a_scale_ref.to(torch.float32)
        b_scale_ref = b_scale_ref.to(torch.float32)

        def unpack_scale(packed):
            packed = packed.reshape(*packed.shape[:-2], 32, 4, 4)
            num_chunk_m, num_chunk_k, _, _, _ = packed.shape
            return packed.permute(0, 3, 2, 1, 4).reshape(num_chunk_m * 128, num_chunk_k * 4).contiguous()

        a_scale_ref = unpack_scale(a_scale_ref).repeat_interleave(VEC_SIZE, dim=1)[:M, :K]
        b_scale_ref = unpack_scale(b_scale_ref).repeat_interleave(VEC_SIZE, dim=1).T.contiguous()[:K, :N]
        reference = torch.matmul(a_ref.to(torch.float32) * a_scale_ref, b_ref * b_scale_ref)

    configs = {
        "BLOCK_SIZE_M": BLOCK_M,
        "BLOCK_SIZE_N": BLOCK_N,
        "BLOCK_SIZE_K": BLOCK_K,
        "num_stages": 4,
        "ELEM_PER_BYTE_A": ELEM_PER_BYTE_A,
        "ELEM_PER_BYTE_B": ELEM_PER_BYTE_B,
        "VEC_SIZE": VEC_SIZE,
    }

    # Flatten scales for cuBLAS
    if block_scale_type == "mxfp8":
        a_scale_cublas = a_scale.contiguous().flatten()
        b_scale_cublas = b_scale.contiguous().flatten()
    elif block_scale_type == "nvfp4":
        a_scale_orig = a_scale_orig.to(torch.float8_e4m3fn)
        b_scale_orig = b_scale_orig.to(torch.float8_e4m3fn)
        a_scale_cublas = a_scale_orig.contiguous().flatten()
        b_scale_cublas = b_scale_orig.contiguous().flatten()
    else:
        a_scale_cublas = None
        b_scale_cublas = None

    return a_desc, a_scale_desc, b_desc, b_scale_desc, rep_m, rep_n, rep_k, configs, reference, a, b, a_scale_cublas, b_scale_cublas
```
**EN:** When `compute_reference` is enabled, the code reconstructs dense per-element scales by unpacking the packed representation, repeating each scale across its vector group, and then evaluating a full-precision PyTorch matmul. The same function also assembles the kernel config dictionary and flattens scales into cuBLAS-friendly buffers when applicable.

**CN:** 当 `compute_reference` 打开时，代码会先解包压缩后的缩放表示，再把每个缩放值按向量组重复扩展成逐元素缩放，最后用全精度 PyTorch matmul 计算参考结果。这个函数同时还会组装内核配置字典，并在适用时把缩放张量展平成 cuBLAS 需要的一维缓冲区。

### Lines 439-499 — Validation, benchmarking, and profile viewing / 校验、基准测试与性能展示
```python
def validate_block_scaled(M, N, K, block_scale_type="nvfp4"):
    results = initialize_block_scaled(M, N, K, block_scale_type, compute_reference=True)
    a_desc, a_scale_desc, b_desc, b_scale_desc, rep_m, rep_n, rep_k, configs, reference = results[:9]
    a, b, a_scale_cublas, b_scale_cublas = results[9:]

    # Test Triton implementation
    output = block_scaled_matmul(a_desc, a_scale_desc, b_desc, b_scale_desc, torch.float16, M, N, K, rep_m, rep_n,
                                 rep_k, configs)
    torch.testing.assert_close(reference, output.to(torch.float32), atol=1e-3, rtol=1e-3)

    # Test cuBLAS implementation if available (available for mxfp8 and nvfp4 only as of 13.1)
    if cublas and block_scale_type in ["mxfp8", "nvfp4"]:
        cublas_output = cublas_block_scaled_matmul(a, a_scale_cublas, b, b_scale_cublas,
                                                   block_scale_type=block_scale_type)
        torch.testing.assert_close(reference, cublas_output.to(torch.float32), atol=1e-3, rtol=1e-3)
        print(f"✅ (pass {block_scale_type} - Triton and cuBLAS)")
    else:
        print(f"✅ (pass {block_scale_type} - Triton only)")


def bench_block_scaled(K, block_scale_type="nvfp4", reps=10, warmup_reps=10):
    assert K % 128 == 0
    M = 8192
    N = 8192
    print(f"Problem Shape = {M}x{N}x{K}")

    results = initialize_block_scaled(M, N, K, block_scale_type, compute_reference=False)
    a_desc, a_scale_desc, b_desc, b_scale_desc, rep_m, rep_n, rep_k, configs, _ = results[:9]
    a, b, a_scale_cublas, b_scale_cublas = results[9:]

    # Warmup
    for _ in range(warmup_reps):
        _ = block_scaled_matmul(a_desc, a_scale_desc, b_desc, b_scale_desc, torch.float16, M, N, K, rep_m, rep_n, rep_k,
                                configs)
        if cublas is not None and supports_block_scaling() and block_scale_type in ["mxfp8", "nvfp4"]:
            _ = cublas_block_scaled_matmul(a, a_scale_cublas, b, b_scale_cublas, block_scale_type=block_scale_type)

    # Benchmark
    proton.activate()
    for _ in range(reps):
        _ = block_scaled_matmul(a_desc, a_scale_desc, b_desc, b_scale_desc, torch.float16, M, N, K, rep_m, rep_n, rep_k,
                                configs)
        if cublas is not None and supports_block_scaling() and block_scale_type in ["mxfp8", "nvfp4"]:
            bytes_per_elem = a.element_size()
            # For nvfp4, K is in elements but a.shape[1] is in bytes, so use K/2 for byte calculation
            K_bytes = K if block_scale_type == "mxfp8" else K // 2
            with proton.scope(f"cublas [M={M}, N={N}, K={K}]",
                              {"bytes": bytes_per_elem * (M * K_bytes + N * K_bytes + M * N), "flops": 2. * M * N * K}):
                _ = cublas_block_scaled_matmul(a, a_scale_cublas, b, b_scale_cublas, block_scale_type=block_scale_type)
    proton.deactivate()
    print("Done benchmarking")


def show_profile(profile_name):
    import triton.profiler.viewer as proton_viewer

    metric_names = ["time/ms"]
    metric_names = ["tflop/s"] + metric_names
    file_name = f"{profile_name}.hatchet"
    tree, metrics = proton_viewer.parse(metric_names, file_name)
    proton_viewer.print_tree(tree, metrics)
```
**EN:** These utilities close the CUDA-side workflow. `validate_block_scaled` compares Triton output against the reconstructed reference and, if available, against cuBLAS; `bench_block_scaled` warms up both implementations and measures them under Proton; `show_profile` parses the generated Hatchet profile and prints a tree view.

**CN:** 这些工具函数补齐了 CUDA 路径的完整工作流。`validate_block_scaled` 会把 Triton 输出与重建得到的参考结果比较，并在可用时继续对比 cuBLAS；`bench_block_scaled` 先热身再用 Proton 记录性能；`show_profile` 则解析生成的 Hatchet profile 并打印树状视图。

### Lines 502-580 — AMD CDNA4 Triton kernel / AMD CDNA4 Triton 内核
```python
@triton.jit
def block_scaled_matmul_kernel_cdna4(a_ptr, b_ptr, c_ptr, a_scales_ptr, b_scales_ptr, M, N, K, stride_am, stride_ak,
                                     stride_bk, stride_bn, stride_ck, stride_cm, stride_cn, stride_asm, stride_ask,
                                     stride_bsn, stride_bsk,
                                     # Meta-parameters
                                     BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,
                                     mfma_nonkdim: tl.constexpr):
    """Kernel for computing the matmul C = A x B.
    A and B inputs are in the microscale fp4 (mxfp4) format.
    A_scales and B_scales are in e8m0 format.
    A has shape (M, K), B has shape (K, N) and C has shape (M, N)
    """

    pid = tl.program_id(axis=0)

    num_pid_n = tl.cdiv(N, BLOCK_N)
    pid_m = pid // num_pid_n
    pid_n = pid % num_pid_n

    # We assume 32 elements along K share the same scale.
    SCALE_GROUP_SIZE: tl.constexpr = 32
    num_k_iter = tl.cdiv(K, BLOCK_K // 2)
    # Create pointers for first block of A and B input matrices
    # The BLOCK sizes are of the elements and in fp4 we pack 2 per uint8 container.
    offs_k = tl.arange(0, BLOCK_K // 2)
    offs_k_split = offs_k
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k_split[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k_split[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

    # Create pointers for the first block of A and B scales
    offs_asn = (pid_n * (BLOCK_N // 32) + tl.arange(0, (BLOCK_N // 32))) % N
    offs_ks = tl.arange(0, BLOCK_K // SCALE_GROUP_SIZE * 32)

    # B scales are N x K even though B operand is K x N.
    b_scale_ptrs = (b_scales_ptr + offs_asn[:, None] * stride_bsn + offs_ks[None, :] * stride_bsk)
    offs_asm = (pid_m * (BLOCK_M // 32) + tl.arange(0, (BLOCK_M // 32))) % M
    a_scale_ptrs = (a_scales_ptr + offs_asm[:, None] * stride_asm + offs_ks[None, :] * stride_ask)
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)

    for k in range(0, num_k_iter):
        # Here we "undo" the shuffle done in global memory (shuffle_scales_cdna4 function).
        if mfma_nonkdim == 32:
            a_scales = tl.load(a_scale_ptrs).reshape(BLOCK_M // 32, BLOCK_K // SCALE_GROUP_SIZE // 8, 2, 32, 4,
                                                     1).permute(0, 3, 1, 4, 2,
                                                                5).reshape(BLOCK_M, BLOCK_K // SCALE_GROUP_SIZE)
            b_scales = tl.load(b_scale_ptrs).reshape(BLOCK_N // 32, BLOCK_K // SCALE_GROUP_SIZE // 8, 2, 32, 4,
                                                     1).permute(0, 3, 1, 4, 2,
                                                                5).reshape(BLOCK_N, BLOCK_K // SCALE_GROUP_SIZE)
        elif mfma_nonkdim == 16:
            a_scales = tl.load(a_scale_ptrs).reshape(BLOCK_M // 32, BLOCK_K // SCALE_GROUP_SIZE // 8, 4, 16, 2, 2,
                                                     1).permute(0, 5, 3, 1, 4, 2,
                                                                6).reshape(BLOCK_M, BLOCK_K // SCALE_GROUP_SIZE)
            b_scales = tl.load(b_scale_ptrs).reshape(BLOCK_N // 32, BLOCK_K // SCALE_GROUP_SIZE // 8, 4, 16, 2, 2,
                                                     1).permute(0, 5, 3, 1, 4, 2,
                                                                6).reshape(BLOCK_N, BLOCK_K // SCALE_GROUP_SIZE)

        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs, cache_modifier=None)

        accumulator += tl.dot_scaled(a, a_scales, "e2m1", b, b_scales, "e2m1")

        # Advance the ptrs to the next K block.
        a_ptrs += (BLOCK_K // 2) * stride_ak
        b_ptrs += (BLOCK_K // 2) * stride_bk

        a_scale_ptrs += BLOCK_K * stride_ask
        b_scale_ptrs += BLOCK_K * stride_bsk

    c = accumulator.to(c_ptr.type.element_ty)

    # Write back the block of the output matrix C with masks.
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M).to(tl.int64)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N).to(tl.int64)
    c_ptrs = (c_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :])
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)

    tl.store(c_ptrs, c, mask=c_mask, cache_modifier=".wt")
```
**EN:** This kernel is the AMD-specific execution path for MXFP4. It computes tile coordinates, builds raw pointer views into packed operands and shuffled scale tensors, reverses the preshuffle inside the loop according to the MFMA non-K dimension, performs `tl.dot_scaled` accumulation, advances pointers across K tiles, and writes the masked output tile back to memory.

**CN:** 这个内核是 AMD CDNA4 上 MXFP4 的专用执行路径。它先计算输出块坐标，再为打包后的输入和已重排的缩放张量构造原始指针视图，然后在循环中根据 MFMA 的非 K 维度撤销预重排，执行 `tl.dot_scaled` 累加，沿着 K 方向推进各类指针，最后把带掩码的输出块写回显存。

### Lines 583-632 — AMD scale shuffle and input setup / AMD 缩放重排与输入准备
```python
def shuffle_scales_cdna4(scales: torch.Tensor, mfma_nonkdim: int):
    scales_shuffled = scales.clone()
    sm, sn = scales_shuffled.shape

    if mfma_nonkdim == 32:
        scales_shuffled = scales_shuffled.view(sm // 32, 32, sn // 8, 4, 2, 1)
        scales_shuffled = scales_shuffled.permute(0, 2, 4, 1, 3, 5).contiguous()
    elif mfma_nonkdim == 16:
        scales_shuffled = scales_shuffled.view(sm // 32, 2, 16, sn // 8, 2, 4, 1)
        scales_shuffled = scales_shuffled.permute(0, 3, 5, 2, 4, 1, 6).contiguous()

    scales_shuffled = scales_shuffled.view(sm // 32, sn * 32)
    return scales_shuffled


def initialize_block_scaled_amd(M, N, K, mfma_nonkdim):

    BLOCK_M = 128
    BLOCK_N = 128
    BLOCK_K = 256
    configs = {
        "BLOCK_M": BLOCK_M,
        "BLOCK_N": BLOCK_N,
        "BLOCK_K": BLOCK_K,
        "num_stages": 2,
        "num_warps": 8,
        "mfma_nonkdim": mfma_nonkdim,
    }

    torch.manual_seed(5)

    x = MXFP4Tensor(size=(M, K), device="cuda").random()
    w = MXFP4Tensor(size=(N, K), device="cuda").random()

    x_scales = torch.randint(124, 128, (K // 32, M), dtype=torch.uint8, device="cuda")
    w_scales = torch.randint(124, 128, (K // 32, N), dtype=torch.uint8, device="cuda")
    x_scales = x_scales.T
    w_scales = w_scales.T
    x_scales_shuffled = shuffle_scales_cdna4(x_scales, configs["mfma_nonkdim"])
    w_scales_shuffled = shuffle_scales_cdna4(w_scales, configs["mfma_nonkdim"])

    return (
        x,
        w,
        x_scales,
        w_scales,
        x_scales_shuffled,
        w_scales_shuffled,
        configs,
    )
```
**EN:** `shuffle_scales_cdna4` materializes the layout described in the earlier comments so that threads see contiguous scale values. `initialize_block_scaled_amd` then builds random MXFP4 operands, generates synthetic E8M0 scale tensors, transposes them into operand-major form, shuffles them, and returns the configuration bundle needed by the AMD kernel.

**CN:** `shuffle_scales_cdna4` 把前面注释里描述的布局真正编码出来，使线程能够连续读取所需的缩放值。随后 `initialize_block_scaled_amd` 会构造随机 MXFP4 输入，生成模拟的 E8M0 缩放张量，把它们转成与操作数对齐的方向，并执行重排，最终返回 AMD 内核需要的整套配置。

### Lines 635-714 — AMD validation, wrapper, and benchmark / AMD 校验、包装器与基准测试
```python
def validate_block_scaled_amd(M, N, K, block_scale_type="mxfp4", mfma_nonkdim=16):

    def e8m0_to_f32(x):
        x_f32 = 2**((x - 127).to(torch.float32))
        x_f32[x_f32 == 128] = float("nan")
        return x_f32

    def run_torch(x, w, x_scales, w_scales, dtype):
        # First convert the x and w inputs to f32.
        x_f32 = x.to(torch.float32)
        w_f32 = w.to(torch.float32)
        # Next convert the e8m0 scales to f32.
        x_scales = x_scales.repeat_interleave(32, dim=1).to(torch.float32)
        x_scales_f32 = e8m0_to_f32(x_scales)
        x_f32 = x_f32 * x_scales_f32
        w_scales = w_scales.repeat_interleave(32, dim=1).to(torch.float32)
        w_scales_f32 = e8m0_to_f32(w_scales)
        w_f32 = w_f32 * w_scales_f32
        return torch.mm(x_f32, w_f32.T).to(dtype)

    x_mxfp4, w_mxfp4, x_scales, w_scales, x_scales_triton, w_scales_triton, configs = \
    initialize_block_scaled_amd(M, N, K, mfma_nonkdim)

    x = x_mxfp4.to_packed_tensor(dim=1)
    w = w_mxfp4.to_packed_tensor(dim=1)

    triton_out = torch.empty((M, N), device=x.device)
    triton_out = block_scaled_matmul_amd(x, w, x_scales_triton, w_scales_triton, configs)
    triton_out = triton_out.to(torch.float32)

    torch_out = run_torch(x_mxfp4, w_mxfp4, x_scales, w_scales, torch.float32)
    torch.testing.assert_close(torch_out, triton_out)
    print(f"✅ (pass {block_scale_type}, mfma_nonk_dim {mfma_nonkdim})")


def block_scaled_matmul_amd(x, w, x_scales_triton, w_scales_triton, configs):
    M, K = x.shape
    N, K = w.shape
    w = w.T
    triton_out = torch.empty((M, N), device=x.device)

    kernel_kwargs = {}
    kernel_kwargs["matrix_instr_nonkdim"] = configs["mfma_nonkdim"]

    BLOCK_M = configs["BLOCK_M"]
    BLOCK_N = configs["BLOCK_N"]

    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)

    triton_out = torch.empty((M, N), device="cuda")

    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    block_scaled_matmul_kernel_cdna4[grid](x, w, triton_out, x_scales_triton, w_scales_triton, M, N, K, x.stride(0),
                                           x.stride(1), w.stride(0), w.stride(1), 0, triton_out.stride(0),
                                           triton_out.stride(1), x_scales_triton.stride(0), x_scales_triton.stride(1),
                                           w_scales_triton.stride(0), w_scales_triton.stride(1), BLOCK_M, BLOCK_N,
                                           configs["BLOCK_K"], configs["mfma_nonkdim"], num_warps=configs["num_warps"],
                                           num_stages=configs["num_stages"], **kernel_kwargs)
    triton_out = triton_out.to(torch.float32)

    return triton_out


def bench_block_scaled_amd(K, block_scale_type="mxfp4", reps=10, mfma_nonkdim=16):
    assert K % 128 == 0
    M = 8192
    N = 8192
    print(f"Problem Shape = {M}x{N}x{K}")

    x_mxfp4, w_mxfp4, x_scales, w_scales, x_scales_triton, w_scales_triton, configs = \
    initialize_block_scaled_amd(M, N, K, mfma_nonkdim)

    x = x_mxfp4.to_packed_tensor(dim=1)
    w = w_mxfp4.to_packed_tensor(dim=1)

    proton.activate()
    for _ in range(reps):
        _ = block_scaled_matmul_amd(x, w, x_scales_triton, w_scales_triton, configs)
    proton.deactivate()
    print("Done benchmarking")
```
**EN:** The AMD utilities mirror the CUDA flow. `validate_block_scaled_amd` converts E8M0 scales into float values, expands them for a dense PyTorch reference, runs the Triton kernel, and checks exact agreement; `block_scaled_matmul_amd` handles the kernel launch details; `bench_block_scaled_amd` profiles repeated runs with Proton.

**CN:** AMD 侧的辅助函数与 CUDA 工作流保持一致。`validate_block_scaled_amd` 会把 E8M0 缩放转换成浮点值、扩展成逐元素缩放以构造 PyTorch 参考结果，运行 Triton 内核并进行一致性校验；`block_scaled_matmul_amd` 负责具体的内核启动；`bench_block_scaled_amd` 则用 Proton 对重复执行进行计时分析。

### Lines 717-752 — Command-line entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-K", type=int, required=False, default=512)
    parser.add_argument("--K_range", type=int, nargs=2)
    parser.add_argument("--K_step", type=int, default=512)
    parser.add_argument("--bench", action="store_true", default=True)
    parser.add_argument("--format", type=str, choices=["mxfp4", "nvfp4", "mxfp8", "mixed"], default="nvfp4")
    args = parser.parse_args()

    if not supports_block_scaling():
        print("⛔ This example requires GPU support for block scaled matmul")
    else:
        if args.K and args.K_range is None:
            args.K_range = [args.K, args.K]
            args.K_step = 1  # doesn't matter as long as it's not 0

        torch.manual_seed(42)

        if is_cuda():
            validate_block_scaled(8192, 8192, 8192, block_scale_type=args.format)
        elif is_hip_cdna4():
            assert args.format == "mxfp4", "AMD tutorial only supports mxpf4 format currently"
            validate_block_scaled_amd(8192, 8192, 8192, block_scale_type=args.format, mfma_nonkdim=16)
            validate_block_scaled_amd(8192, 8192, 8192, block_scale_type=args.format, mfma_nonkdim=32)

        if args.bench:
            proton.start("block_scaled_matmul", hook="triton")
            proton.deactivate()  # Skip argument creation
            for K in range(args.K_range[0], args.K_range[1] + 1, args.K_step):
                if is_cuda():
                    bench_block_scaled(K, reps=10000, block_scale_type=args.format)
                elif is_hip_cdna4():
                    bench_block_scaled_amd(K, reps=10000, block_scale_type=args.format, mfma_nonkdim=16)
                    bench_block_scaled_amd(K, reps=10000, block_scale_type=args.format, mfma_nonkdim=32)
            proton.finalize()
            show_profile("block_scaled_matmul")
```
**EN:** The `__main__` block defines the CLI, normalizes `-K` into a one-value range, checks hardware support, seeds randomness, runs the appropriate validation path for CUDA or AMD, optionally benchmarks over a K sweep, and finally finalizes plus displays the collected profile.

**CN:** `__main__` 代码块定义了命令行接口，把单个 `-K` 规格统一转换成范围形式，检查硬件是否支持，设置随机种子，再根据 CUDA 或 AMD 选择对应的校验路径；如果开启基准测试，还会对一组 K 值循环执行并在最后收集、展示 profile。

## Key Concepts / 关键概念
- **Block scaling / 块缩放：** One scale value is shared by a short vector of elements, so the kernel must manage both the packed low-precision payload and the corresponding grouped scale tensor. / 一个缩放值会共享给一小组元素，因此内核必须同时管理打包后的低精度数据与按组组织的缩放张量。
- **Layout engineering / 布局工程：** Much of the tutorial is about storing scales in hardware-friendly layouts and then reconstructing the logical view expected by `tl.dot_scaled`. / 本教程的大量代码都围绕如何把缩放值存成硬件友好的布局，再恢复成 `tl.dot_scaled` 需要的逻辑视图。
- **Descriptor-based loads / 基于描述符的加载：** CUDA-side kernels use `TensorDescriptor` objects so TMA-friendly block loads can fetch operands and scale tiles efficiently. / CUDA 路径使用 `TensorDescriptor`，以便通过适合 TMA 的块加载方式高效抓取操作数和缩放块。
- **Format polymorphism / 格式多态：** The same tutorial supports `nvfp4`, `mxfp4`, `mxfp8`, and a mixed mode by changing vector size, packing ratio, and `tl.dot_scaled` format strings. / 教程通过调整向量大小、打包比率以及 `tl.dot_scaled` 的格式字符串，同时支持 `nvfp4`、`mxfp4`、`mxfp8` 和混合模式。
- **Backend split / 后端分流：** NVIDIA and AMD follow different memory-layout and kernel strategies, so the file implements separate kernels and preparation functions while keeping validation and benchmarking patterns similar. / NVIDIA 与 AMD 在内存布局和内核实现上差异明显，因此文件分别实现了各自的内核与准备流程，但在校验与基准测试模式上尽量保持一致。
- **End-to-end workflow / 端到端流程：** The CLI always follows the same high-level structure: detect support, initialize inputs, validate correctness, benchmark if requested, and render a profile. / 命令行总是遵循同样的高层流程：检测支持情况、初始化输入、校验正确性、按需跑基准测试，并输出 profile。

## Dependencies / 依赖关系
- **`argparse`** drives the command-line interface. / **`argparse`** 负责命令行参数解析。
- **`torch`** provides tensor allocation, dtype conversion, reference matmul, assertions, and CUDA device queries. / **`torch`** 提供张量分配、dtype 转换、参考 matmul、断言以及 CUDA 设备查询。
- **`triton`** and **`triton.language as tl`** provide JIT compilation, program IDs, tensor-core dot primitives, and launch configuration. / **`triton`** 与 **`triton.language as tl`** 提供 JIT 编译、program ID、张量核心点乘原语以及启动配置。
- **`triton.profiler as proton`** and `triton.profiler.viewer` are used for benchmarking and reading profile outputs. / **`triton.profiler as proton`** 与 `triton.profiler.viewer` 用于性能采集和 profile 展示。
- **`TensorDescriptor`** is the bridge between PyTorch tensors and descriptor-driven block loads/stores in the CUDA-style kernel. / **`TensorDescriptor`** 是 PyTorch 张量与 CUDA 风格描述符加载/存储之间的桥梁。
- **`MXFP4Tensor`** and **`MXScaleTensor`** generate and transform the tutorial's low-precision data and scale layouts. / **`MXFP4Tensor`** 与 **`MXScaleTensor`** 用于生成并转换教程中的低精度数据和缩放布局。
- **Optional NVIDIA dependency / 可选 NVIDIA 依赖：** `triton._C.libtriton.nvidia.cublas.CublasLt` is only initialized when the active target supports the required CUDA path. / `triton._C.libtriton.nvidia.cublas.CublasLt` 只会在活动目标满足 CUDA 条件时初始化。
- **Hardware assumptions / 硬件前提：** The tutorial expects NVIDIA compute capability 10/11 for CUDA block scaling or AMD `gfx950` for the CDNA4 path. / 本教程默认依赖 NVIDIA 计算能力 10/11 的块缩放支持，或 AMD `gfx950` 的 CDNA4 路径。
- **Call-flow dependency / 调用流程依赖：** CLI -> validation/benchmark -> initialization -> Triton kernel/cuBLAS baseline -> profiling output. / 调用链路基本为：CLI -> 校验/基准测试 -> 初始化 -> Triton 内核或 cuBLAS 基线 -> profile 输出。
