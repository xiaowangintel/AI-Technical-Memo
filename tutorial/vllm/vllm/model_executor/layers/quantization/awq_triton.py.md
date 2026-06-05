# awq_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/awq_triton.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides helpers such as `awq_dequantize_kernel`, `awq_gemm_kernel`, `awq_dequantize_triton` for quantization backends, schemes, and utilities. / 提供诸如 `awq_dequantize_kernel`, `awq_gemm_kernel`, `awq_dequantize_triton` 之类的辅助函数，用于量化后端、方案与工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-6)
```python
import torch

from vllm.triton_utils import tl, triton
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.triton_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.triton_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 8-8)
```python
AWQ_TRITON_SUPPORTED_GROUP_SIZES = [-1, 32, 64, 128]
```
**EN:** This block defines module-level metadata or constants such as `AWQ_TRITON_SUPPORTED_GROUP_SIZES`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `AWQ_TRITON_SUPPORTED_GROUP_SIZES`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `awq_dequantize_kernel` (lines 12-105)
```python
def awq_dequantize_kernel(
    qweight_ptr,  # quantized matrix
    scales_ptr,  # scales, per group
    zeros_ptr,  # zeros, per group
    group_size,  # Should always be one of the supported group sizes
    result_ptr,  # Output matrix
    num_cols,  # input num cols in qweight
    num_rows,  # input num rows in qweight
    BLOCK_SIZE_X: tl.constexpr,
    BLOCK_SIZE_Y: tl.constexpr,
):
    # Set up the pids.
    pid_x = tl.program_id(axis=0)
    pid_y = tl.program_id(axis=1)

    # Compute offsets and masks for qweight_ptr.
    offsets_y = pid_y * BLOCK_SIZE_Y + tl.arange(0, BLOCK_SIZE_Y)
    offsets_x = pid_x * BLOCK_SIZE_X + tl.arange(0, BLOCK_SIZE_X)
    offsets = num_cols * offsets_y[:, None] + offsets_x[None, :]

    masks_y = offsets_y < num_rows
    masks_x = offsets_x < num_cols

    masks = masks_y[:, None] & masks_x[None, :]

    # Compute offsets and masks for result output ptr.
    result_offsets_y = pid_y * BLOCK_SIZE_Y + tl.arange(0, BLOCK_SIZE_Y)
    result_offsets_x = pid_x * BLOCK_SIZE_X * 8 + tl.arange(0, BLOCK_SIZE_X * 8)
    result_offsets = (
        8 * num_cols * result_offsets_y[:, None] + result_offsets_x[None, :]
    )

    result_masks_y = result_offsets_y < num_rows
    result_masks_x = result_offsets_x < num_cols * 8
    result_masks = result_masks_y[:, None] & result_masks_x[None, :]

    # Load the weights.
    iweights = tl.load(qweight_ptr + offsets, masks, 0.0)
# ... truncated for analysis ...
    scale_masks_y = scale_offsets_y < num_rows // group_size
    scale_masks_x = scale_offsets_x < num_cols * 8
    scale_masks = scale_masks_y[:, None] & scale_masks_x[None, :]

    # Load the scales.
    scales = tl.load(scales_ptr + scale_offsets, scale_masks, 0.0)
    scales = tl.broadcast_to(scales, (BLOCK_SIZE_Y, BLOCK_SIZE_X * 8))

    # Dequantize.
    iweights = (iweights - zeros) * scales
    iweights = iweights.to(result_ptr.type.element_ty)

    # Finally, store.
    tl.store(result_ptr + result_offsets, iweights, result_masks)
```
**EN:** Defines function `awq_dequantize_kernel` with signature `awq_dequantize_kernel(qweight_ptr, scales_ptr, zeros_ptr, group_size, result_ptr, num_cols, num_rows, BLOCK_SIZE_X: tl.constexpr, BLOCK_SIZE_Y: tl.constexpr)`. It mainly works with `qweight_ptr`, `scales_ptr`, `zeros_ptr`, `group_size`, `result_ptr`, `num_cols`, `num_rows`, `BLOCK_SIZE_X`; handles quantization-related transformation logic. The body uses tensor/kernel operations. Key calls include `tl.program_id`, `tl.load`, `tl.interleave`, `reshape`, `tl.broadcast_to`, `tl.reshape`.
**CN:** 定义函数 `awq_dequantize_kernel`，其签名为 `awq_dequantize_kernel(qweight_ptr, scales_ptr, zeros_ptr, group_size, result_ptr, num_cols, num_rows, BLOCK_SIZE_X: tl.constexpr, BLOCK_SIZE_Y: tl.constexpr)`。它主要围绕 `qweight_ptr`, `scales_ptr`, `zeros_ptr`, `group_size`, `result_ptr`, `num_cols`, `num_rows`, `BLOCK_SIZE_X` 展开；处理量化相关的变换逻辑。函数体包含张量或内核操作。关键调用包括 `tl.program_id`, `tl.load`, `tl.interleave`, `reshape`, `tl.broadcast_to`, `tl.reshape`。

### Function `awq_gemm_kernel` (lines 109-227)
```python
def awq_gemm_kernel(
    a_ptr,
    b_ptr,
    c_ptr,
    zeros_ptr,
    scales_ptr,
    M,
    N,
    K,
    group_size,
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    SPLIT_K: tl.constexpr,
):
    pid = tl.program_id(axis=0)
    pid_z = tl.program_id(1)

    # NOTE: This doesn't work in TRITON_INTERPRET=1 mode.  Use below instead.
    # num_pid_n = (N + BLOCK_SIZE_N - 1) // BLOCK_SIZE_N
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)

    pid_m = pid // num_pid_n
    pid_n = pid % num_pid_n

    accumulator_dtype = c_ptr.type.element_ty

    # NOTE: This doesn't work in TRITON_INTERPRET=1 mode.  Use below instead.
    # accumulator = tl.arange(0, BLOCK_SIZE_N)
    # accumulator = tl.broadcast_to(accumulator[None, :],
    # (BLOCK_SIZE_M, BLOCK_SIZE_N))
    # accumulator = accumulator & 0x0
    # accumulator = accumulator.to(accumulator_dtype)
    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=accumulator_dtype)

    # Create reverse AWQ order as tensor: [0, 4, 1, 5, 2, 6, 3, 7]
    # that will map given indices to the correct order.
    reverse_awq_order_tensor = (
# ... truncated for analysis ...

        # Accumulate results.
        accumulator = tl.dot(a, b, accumulator, out_dtype=accumulator_dtype)

        offsets_k += BLOCK_SIZE_K * SPLIT_K
        a_ptrs += BLOCK_SIZE_K * SPLIT_K
        b_ptrs += BLOCK_SIZE_K * SPLIT_K * (N // 8)

    c = accumulator.to(c_ptr.type.element_ty)
    offs_cm = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
    offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    c_ptrs = c_ptr + pid_z * N * M + N * offs_cm[:, None] + offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(c_ptrs, c, mask=c_mask)
```
**EN:** Defines function `awq_gemm_kernel` with signature `awq_gemm_kernel(a_ptr, b_ptr, c_ptr, zeros_ptr, scales_ptr, M, N, K, group_size, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, SPLIT_K: tl.constexpr)`. It mainly works with `a_ptr`, `b_ptr`, `c_ptr`, `zeros_ptr`, `scales_ptr`, `M`, `N`, `K`; implements one step in the quantized-weight execution flow. The body uses iteration, tensor/kernel operations. Key calls include `tl.program_id`, `tl.cdiv`, `tl.zeros`, `reshape`, `tl.broadcast_to`, `tl.reshape`.
**CN:** 定义函数 `awq_gemm_kernel`，其签名为 `awq_gemm_kernel(a_ptr, b_ptr, c_ptr, zeros_ptr, scales_ptr, M, N, K, group_size, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr, BLOCK_SIZE_K: tl.constexpr, SPLIT_K: tl.constexpr)`。它主要围绕 `a_ptr`, `b_ptr`, `c_ptr`, `zeros_ptr`, `scales_ptr`, `M`, `N`, `K` 展开；实现量化权重执行流程中的一个步骤。函数体包含循环处理、张量或内核操作。关键调用包括 `tl.program_id`, `tl.cdiv`, `tl.zeros`, `reshape`, `tl.broadcast_to`, `tl.reshape`。

### Function `awq_dequantize_triton` (lines 233-279)
```python
def awq_dequantize_triton(
    qweight: torch.Tensor,
    scales: torch.Tensor,
    zeros: torch.Tensor,
    block_size_x: int = 32,
    block_size_y: int = 32,
) -> torch.Tensor:
    K = qweight.shape[0]
    M = scales.shape[1]
    group_size = qweight.shape[0] // scales.shape[0]

    assert K > 0 and M > 0
    assert scales.shape[0] == K // group_size and scales.shape[1] == M
    assert zeros.shape[0] == K // group_size and zeros.shape[1] == M // 8
    assert group_size <= K
    assert group_size in AWQ_TRITON_SUPPORTED_GROUP_SIZES or group_size == K

    # Result tensor:
    # number of rows = same as input tensor
    # number of cols = 8 x input tensor num cols
    result = torch.empty(
        qweight.shape[0],
        qweight.shape[1] * 8,
        device=qweight.device,
        dtype=scales.dtype,
    )

    Y = qweight.shape[0]  # num rows
    X = qweight.shape[1]  # num cols

    grid = lambda META: (
        triton.cdiv(X, META["BLOCK_SIZE_X"]),
        triton.cdiv(Y, META["BLOCK_SIZE_Y"]),
    )
    awq_dequantize_kernel[grid](
        qweight,
        scales,
        zeros,
        group_size,
        result,
        X,
        Y,
        BLOCK_SIZE_X=block_size_x,
        BLOCK_SIZE_Y=block_size_y,
    )

    return result
```
**EN:** Defines function `awq_dequantize_triton` with signature `awq_dequantize_triton(qweight: torch.Tensor, scales: torch.Tensor, zeros: torch.Tensor, block_size_x: int=32, block_size_y: int=32) -> torch.Tensor`. It mainly works with `qweight`, `scales`, `zeros`, `block_size_x`, `block_size_y`; handles quantization-related transformation logic. The body uses validation/error handling, tensor/kernel operations. Key calls include `torch.empty`, `awq_dequantize_kernel`, `triton.cdiv`.
**CN:** 定义函数 `awq_dequantize_triton`，其签名为 `awq_dequantize_triton(qweight: torch.Tensor, scales: torch.Tensor, zeros: torch.Tensor, block_size_x: int=32, block_size_y: int=32) -> torch.Tensor`。它主要围绕 `qweight`, `scales`, `zeros`, `block_size_x`, `block_size_y` 展开；处理量化相关的变换逻辑。函数体包含校验或报错逻辑、张量或内核操作。关键调用包括 `torch.empty`, `awq_dequantize_kernel`, `triton.cdiv`。

### Function `awq_gemm_triton` (lines 287-337)
```python
def awq_gemm_triton(
    input: torch.Tensor,
    qweight: torch.Tensor,
    scales: torch.Tensor,
    qzeros: torch.Tensor,
    split_k_iters: int,
    block_size_m: int = 32,
    block_size_n: int = 32,
    block_size_k: int = 32,
) -> torch.Tensor:
    M, K = input.shape
    N = qweight.shape[1] * 8
    group_size = qweight.shape[0] // qzeros.shape[0]

    assert N > 0 and K > 0 and M > 0
    assert qweight.shape[0] == K and qweight.shape[1] == N // 8
    assert qzeros.shape[0] == K // group_size and qzeros.shape[1] == N // 8
    assert scales.shape[0] == K // group_size and scales.shape[1] == N
    assert split_k_iters & (split_k_iters - 1) == 0 and split_k_iters != 0
    assert split_k_iters <= 32
    assert group_size <= K
    assert group_size in AWQ_TRITON_SUPPORTED_GROUP_SIZES or group_size == K

    grid = lambda META: (
        triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"]),
        split_k_iters,
    )

    result = torch.zeros((split_k_iters, M, N), dtype=scales.dtype, device=input.device)

    # A = input, B = qweight, C = result
    # A = M x K, B = K x N, C = M x N
    awq_gemm_kernel[grid](
        input,
        qweight,
        result,
        qzeros,
        scales,
        M,
        N,
        K,
        group_size,
        BLOCK_SIZE_M=block_size_m,
        BLOCK_SIZE_N=block_size_n,
        BLOCK_SIZE_K=block_size_k,
        SPLIT_K=split_k_iters,
    )

    result = result.sum(0)

    return result
```
**EN:** Defines function `awq_gemm_triton` with signature `awq_gemm_triton(input: torch.Tensor, qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor, split_k_iters: int, block_size_m: int=32, block_size_n: int=32, block_size_k: int=32) -> torch.Tensor`. It mainly works with `input`, `qweight`, `scales`, `qzeros`, `split_k_iters`, `block_size_m`, `block_size_n`, `block_size_k`; implements one step in the quantized-weight execution flow. The body uses validation/error handling, tensor/kernel operations. Key calls include `torch.zeros`, `awq_gemm_kernel`, `result.sum`, `triton.cdiv`.
**CN:** 定义函数 `awq_gemm_triton`，其签名为 `awq_gemm_triton(input: torch.Tensor, qweight: torch.Tensor, scales: torch.Tensor, qzeros: torch.Tensor, split_k_iters: int, block_size_m: int=32, block_size_n: int=32, block_size_k: int=32) -> torch.Tensor`。它主要围绕 `input`, `qweight`, `scales`, `qzeros`, `split_k_iters`, `block_size_m`, `block_size_n`, `block_size_k` 展开；实现量化权重执行流程中的一个步骤。函数体包含校验或报错逻辑、张量或内核操作。关键调用包括 `torch.zeros`, `awq_gemm_kernel`, `result.sum`, `triton.cdiv`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `awq_dequantize_kernel`, `awq_gemm_kernel`, `awq_dequantize_triton`, `awq_gemm_triton` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `awq_dequantize_kernel`, `awq_gemm_kernel`, `awq_dequantize_triton`, `awq_gemm_triton` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.triton_utils`
