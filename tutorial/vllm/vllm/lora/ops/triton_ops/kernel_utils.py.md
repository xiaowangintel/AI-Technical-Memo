# kernel_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/kernel_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provide reusable Triton JIT helpers for the core LoRA expand/shrink matrix math. / 提供可复用的 Triton JIT 辅助函数，用于 LoRA expand/shrink 的核心矩阵计算。

## Line-by-Line Analysis / 逐行分析
### Shared K-loop matrix multiplication helper
```python
@triton.jit
def mm_k(...):
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
    STEP_K = BLOCK_K * SPLIT_K
    num_iters = tl.cdiv(K, STEP_K)
    for k in range(num_iters):
        iter_k = k * STEP_K + base_k
        if EVEN_K:
            tiled_b = tl.load(b_ptr)
            if USE_GDC:
                tl.extra.cuda.gdc_wait()
            tiled_a = tl.load(a_ptr)
            if CAST_TYPE:
                tiled_a = tiled_a.to(b_dtype)
            accumulator += tl.dot(tiled_a, tiled_b)
        else:
            ...
        a_ptr += STEP_K * ak_stride
        b_ptr += STEP_K * bk_stride
    return accumulator
```
**EN:** `mm_k` is the common GEMM inner loop. It iterates over the K dimension, optionally masks tail tiles when `K` is not aligned, casts A to B's dtype if needed, and accumulates into FP32 for numerical stability.
**CN:** `mm_k` 是共享的 GEMM 内层循环。它沿 K 维迭代，在 `K` 未对齐时对尾块做掩码处理，必要时把 A 转成 B 的数据类型，并以 FP32 累加以提升数值稳定性。

### Expand-path pointer setup and optional residual add
```python
@triton.jit
def do_expand_kernel(...):
    if SAME_STRIDE:
        cur_lora_d0_stride = ls_d0_ptr
        cur_lora_d1_stride = ls_d1_ptr
        cur_lora_d2_stride = ls_d2_ptr
    else:
        cur_lora_d0_stride = tl.load(ls_d0_ptr + slice_id)
        cur_lora_d1_stride = tl.load(ls_d1_ptr + slice_id)
        cur_lora_d2_stride = tl.load(ls_d2_ptr + slice_id)
    ...
    accumulator = mm_k(..., SPLIT_K=1, ...)
    tiled_c = accumulator.to(cur_lora_ptr.dtype.element_ty)
    ...
    if ADD_INPUTS:
        tiled_out = tl.load(c_ptr, mask=c_mask)
        tiled_c += tiled_out
    tl.store(c_ptr, tiled_c, mask=c_mask)
```
**EN:** `do_expand_kernel` resolves the correct input slice, LoRA-B slice, and output region, performs a single-pass matrix multiply, and optionally adds the existing output tensor to support residual-style accumulation.
**CN:** `do_expand_kernel` 解析当前输入切片、LoRA-B 切片和输出区域，执行单次矩阵乘法，并可选地与已有输出相加，以支持类似残差的累加行为。

### Shrink-path split-K writeback
```python
@triton.jit
def do_shrink_kernel(...):
    offset_k = pid_sk * BLOCK_K + tl.arange(0, BLOCK_K)
    a_ptr = input_ptr + ram[:, None] * input_d0_stride + offset_k[None, :] * input_d1_stride
    b_ptr = cur_lora_ptr + lora_d0_stride * lora_index + rbn[None, :] * lora_d1_stride + offset_k[:, None] * lora_d2_stride
    accumulator = mm_k(..., SPLIT_K, False, cur_lora_ptr.dtype.element_ty, False, base_k=pid_sk * BLOCK_K)
    if USE_GDC:
        tl.extra.cuda.gdc_launch_dependents()
    accumulator *= scaling
    if SPLIT_K == 1:
        tl.store(c_ptr, accumulator, mask=c_mask)
    else:
        tl.atomic_add(c_ptr, accumulator, mask=c_mask, sem="relaxed")
```
**EN:** `do_shrink_kernel` computes LoRA-A projection blocks. It supports split-K parallelism, applies the LoRA scaling factor, and uses `atomic_add` when multiple CTAs contribute partial sums to the same output tile.
**CN:** `do_shrink_kernel` 计算 LoRA-A 投影块，支持 split-K 并行，应用 LoRA 缩放系数，并在多个 CTA 共同写回同一输出块时使用 `atomic_add` 归约。

## Key Concepts / 关键概念
- EN: Shared Triton helper kernels reduce duplication between expand and shrink implementations. / CN: 共享 Triton 辅助内核减少了 expand 与 shrink 实现中的重复逻辑。
- EN: Tail masking, dtype conversion, and split-K reduction are the main low-level concerns. / CN: 尾块掩码、数据类型转换和 split-K 归约是主要底层关注点。
- EN: GDC hooks are present for programmatic dependent launch integration. / CN: 代码中预留了 GDC 钩子以支持程序化依赖启动。

## Dependencies / 依赖关系
- EN: Depends on `vllm.triton_utils.tl` and `vllm.triton_utils.triton`; called by `lora_expand_op.py` and `lora_shrink_op.py`. / CN: 依赖 `vllm.triton_utils.tl` 与 `vllm.triton_utils.triton`；由 `lora_expand_op.py` 和 `lora_shrink_op.py` 调用。
