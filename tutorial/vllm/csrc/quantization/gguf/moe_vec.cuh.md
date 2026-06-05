# moe_vec.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/moe_vec.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements GGUF quantized GEMV kernels for MoE routing, where each `(token, expert)` pair selects one expert matrix and one quantized activation row. / [CN] 实现面向 MoE 路由的 GGUF 量化 GEMV 内核，其中每个 `(token, expert)` 对都会选择一个专家矩阵和一行量化激活。

## Line-by-Line Analysis / 逐行分析
### Generic routed GEMV kernel (L3-L29)
```cpp
template <typename scalar_t, int qk, int qi, typename block_q_t, int vdr,
          vec_dot_q_cuda_t vec_dot_q_cuda>
static __global__ void moe_vec_q(...) {
  const auto row = blockIdx.x * blockDim.y + threadIdx.y;
  const auto token = blockIdx.z / topk;
  const auto expert = (topk_ids)[blockIdx.z];
  ...
  const block_q_t* x = ((const block_q_t*)vx) + expert * nrows * blocks_per_row;
  const block_q8_1* y = (const block_q8_1*)(((const int*)vy) + token * token_stride);
}
```
**EN:** `blockIdx.z` encodes the routed `(token, expert-slot)` pair. The kernel jumps to the selected expert's quantized weight matrix and to the token's Q8_1 activation row, so the same kernel can evaluate many routed experts in parallel.
**CN:** `blockIdx.z` 编码了路由后的 `(token, expert-slot)` 对。内核据此跳到被选中的专家权重矩阵和该 token 的 Q8_1 激活行，因此同一个内核可以并行处理多个路由专家。

### Per-row accumulation and warp reduction (L30-L52)
```cpp
for (auto i = threadIdx.x / (qi / vdr); i < blocks_per_row; i += blocks_per_warp) {
  ...
  tmp += vec_dot_q_cuda(&x[ibx], &y[iby], iqs);
}
for (int mask = WARP_SIZE / 2; mask > 0; mask >>= 1) {
  tmp += VLLM_SHFL_XOR_SYNC(tmp, mask);
}
if (threadIdx.x == 0) {
  dst[blockIdx.z * nrows + row] = tmp;
}
```
**EN:** Threads in one warp split the quant blocks of a single output row, accumulate partial dot products, and then reduce them with warp shuffles. Only lane 0 writes the final row value.
**CN:** 同一 warp 内的线程会分担一个输出行上的量化块，先累加局部点积，再通过 warp shuffle 完成规约，最后只由 lane 0 写回结果。

### Standard GGUF wrappers (L55-L203)
```cpp
moe_vec_q<scalar_t, QK4_0, QI4_0, block_q4_0, VDR_Q4_0_Q8_1_MMVQ,
          vec_dot_q4_0_q8_1><<<block_nums, block_dims, 0, stream>>>(...);
```
**EN:** Wrappers for `q4_0`, `q4_1`, `q5_0`, `q5_1`, `q8_0`, and `q2_K`~`q6_K` only configure the quant block type and matching `vec_dot` implementation. Launch geometry is shared across all of them.
**CN:** `q4_0`、`q4_1`、`q5_0`、`q5_1`、`q8_0` 以及 `q2_K`~`q6_K` 的包装函数主要负责指定量化块类型和匹配的 `vec_dot` 实现，启动配置则保持一致。

### IQ wrappers for non-linear quant formats (L205-L338)
```cpp
moe_vec_q<scalar_t, QK_K, QI2_XXS, block_iq2_xxs, 1, vec_dot_iq2_xxs_q8_1>
    <<<block_nums, block_dims, 0, stream>>>(...);
```
**EN:** The IQ wrappers reuse the same routing kernel but swap in lookup-table-based dot-product functions such as `vec_dot_iq2_xxs_q8_1` and `vec_dot_iq4_xs_q8_1`. This keeps MoE control flow identical across linear and non-linear GGUF quantizers.
**CN:** IQ 包装函数复用了同一个路由内核，只是换成了基于查表的点积函数，如 `vec_dot_iq2_xxs_q8_1` 和 `vec_dot_iq4_xs_q8_1`。因此线性量化和非线性 GGUF 量化在 MoE 控制流上保持一致。

## Key Concepts / 关键概念
- Routing encoded on `blockIdx.z` / 用 `blockIdx.z` 编码路由结果
- Expert-local weight selection / 选择专家局部权重矩阵
- Warp-level reduction for quantized GEMV / 面向量化 GEMV 的 warp 规约
- Shared launcher shape across many GGUF formats / 多种 GGUF 格式共用启动形状

## Dependencies / 依赖关系
- `ggml-common.h`: `block_q*`, `block_iq*`, `QK* / QI*`, `GGML_CUDA_MMV_Y`
- `vecdotq.cuh`: all `vec_dot_*_q8_1` functions and VDR macros
- `gguf_kernel.cu`: public entry point `ggml_moe_a8_vec`
