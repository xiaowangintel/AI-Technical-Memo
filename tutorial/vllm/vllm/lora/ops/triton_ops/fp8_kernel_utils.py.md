# fp8_kernel_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/ops/triton_ops/fp8_kernel_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Supply Triton helper kernels for FP8/INT8-aware LoRA matrix multiplication. / 提供面向 FP8/INT8 LoRA 矩阵乘的 Triton 辅助内核。

## Line-by-Line Analysis / 逐行分析
### Quantized accumulation helper / 量化累加辅助函数
```python
@triton.jit
def _accumulate_mm(...):
    if use_fp8_w8a8:
        if group_k > 0 and group_n > 0:
            a_scale = tl.load(a_scale_ptr + offs_ks * a_scale_k_stride)
            b_scale = tl.load(b_scale_ptr + offs_ks * b_scale_k_stride)
            accumulator += tl.dot(tiled_a, tiled_b) * a_scale[:, None] * b_scale[None, :]
        else:
            accumulator = tl.dot(tiled_a, tiled_b, acc=accumulator)
    else:
        accumulator += tl.dot(tiled_a, tiled_b)
```
**EN:** `_accumulate_mm` centralizes how one tile product is accumulated. For block-wise FP8 quantization it loads per-block scales immediately; otherwise it either defers scaling or runs an unquantized dot product.
**CN:** `_accumulate_mm` 统一处理单个 tile 的乘加过程。对于块级 FP8 量化，它会立刻加载块尺度；否则要么延后缩放，要么走非量化点积路径。

### K-loop matrix multiply / 沿 K 维循环的矩阵乘
```python
@triton.jit
def fp8_mm_k(...):
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
    for k in range(num_iters):
        tiled_b = tl.load(...)
        tiled_a = tl.load(...)
        accumulator = _accumulate_mm(...)
```
**EN:** `fp8_mm_k` is the reusable GEMM core. It iterates over the K dimension, handles full and tail tiles, optionally casts activations, and supports split-K as well as GDC synchronization hints.
**CN:** `fp8_mm_k` 是可复用的 GEMM 核心：它沿 K 维迭代，处理完整块与尾块，可选地转换激活 dtype，并支持 split-K 与 GDC 同步提示。

### FP8 shrink kernel / FP8 shrink 内核
```python
@triton.jit
def do_shrink_kernel_fp8(...):
    cur_lora_ptr = ...
    a_ptr = input_ptr + ...
    b_ptr = cur_lora_ptr + ...
    accumulator = fp8_mm_k(...)
    accumulator *= scaling
    tl.store(...) or tl.atomic_add(...)
```
**EN:** `do_shrink_kernel_fp8` maps a token block and one LoRA slice to the LoRA-A projection. It prepares weight/scale pointers, invokes `fp8_mm_k`, applies post-accumulation scaling, and writes results with direct store or atomic reduction.
**CN:** `do_shrink_kernel_fp8` 将一个 token 块与一个 LoRA 分片映射到 LoRA-A 投影过程。它先准备权重与尺度指针，再调用 `fp8_mm_k`，随后应用缩放并通过普通写回或原子累加输出结果。

### FP8 expand kernel / FP8 expand 内核
```python
@triton.jit
def do_expand_kernel_fp8(...):
    cur_input_ptr = ...
    cur_lora_ptr = ...
    accumulator = fp8_mm_k(..., SPLIT_K=1, ...)
    if ADD_INPUTS:
        tiled_out = tl.load(c_ptr, mask=c_mask)
        tiled_c += tiled_out
    tl.store(c_ptr, tiled_c, mask=c_mask)
```
**EN:** `do_expand_kernel_fp8` performs the LoRA-B expansion from rank space back to model space. It reuses the same GEMM helper, computes the output slice offset, and optionally accumulates into an existing output tensor.
**CN:** `do_expand_kernel_fp8` 完成从低秩空间回到模型空间的 LoRA-B expand。它复用同一个 GEMM 辅助函数，计算输出分片偏移，并可选择在已有输出上累加。

## Key Concepts / 关键概念
- **EN:** Shared FP8/INT8 GEMM logic extracted for multiple LoRA kernels.
- **CN:** 将共享的 FP8/INT8 GEMM 逻辑抽取出来，供多个 LoRA 内核复用。
- **EN:** Supports block-wise, per-channel, and tensor-wise scaling strategies.
- **CN:** 支持块级、按通道以及整张量三种缩放策略。
- **EN:** Split-K and atomic add enable parallel reduction over the K dimension.
- **CN:** split-K 与原子加支持在 K 维上进行并行归约。

## Dependencies / 依赖关系
- **EN:** Imports `tl` and `triton` from `vllm.triton_utils`.
- **CN:** 从 `vllm.triton_utils` 导入 `tl` 与 `triton`。
- **EN:** Intended to be called by higher-level Triton LoRA operators.
- **CN:** 设计上供更高层的 Triton LoRA 算子调用。
