# mmvq.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gguf/mmvq.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements quantized matrix-vector kernels that multiply GGUF weights by Q8_1 vectors, one output row per warp. / [CN] 实现 GGUF 权重与 Q8_1 向量相乘的量化矩阵向量内核，每个 warp 负责一个输出行。

## Line-by-Line Analysis / 逐行分析
### Generic GEMV kernel (L2-L20)
```cpp
template <typename scalar_t, int qk, int qi, typename block_q_t, int vdr, vec_dot_q_cuda_t vec_dot_q_cuda>
static __global__ void mul_mat_vec_q(...) {
    const auto row = blockIdx.x*blockDim.y + threadIdx.y;
    const auto vec = blockIdx.y;
    ...
    const block_q_t  * x = (const block_q_t  *) vx;
    const block_q8_1 * y = (const block_q8_1 *) vy;
}
```
**EN:** The kernel maps `blockIdx.x` to output rows and `blockIdx.y` to input vectors. It assumes the right-hand side has already been quantized into Q8_1 blocks.
**CN:** 内核把 `blockIdx.x` 映射到输出行，把 `blockIdx.y` 映射到输入向量。它假设右侧输入已经量化成 Q8_1 块。

### Block walk and warp reduction (L22-L41)
```cpp
for (auto i = threadIdx.x / (qi/vdr); i < blocks_per_row; i += blocks_per_warp) {
    const int ibx = row*blocks_per_row + i;
    const int iby = vec*(nrows_y/QK8_1) + i * (qk/QK8_1);
    tmp += vec_dot_q_cuda(&x[ibx], &y[iby], iqs);
}
...
if (threadIdx.x == 0) {
    dst[vec*nrows + row] = tmp;
}
```
**EN:** Threads divide the quant blocks of one row, accumulate partial dot products, and reduce them with `VLLM_SHFL_XOR_SYNC`. The padded `nrows_y` keeps the Q8_1 layout aligned to the block format.
**CN:** 线程会分摊一行中的量化块，累加局部点积，再通过 `VLLM_SHFL_XOR_SYNC` 规约。这里的 `nrows_y` 做了 padding，以保证 Q8_1 布局与块格式对齐。

### Standard and K-quant wrappers (L43-L131)
```cpp
mul_mat_vec_q<scalar_t, QK4_0, QI4_0, block_q4_0, VDR_Q4_0_Q8_1_MMVQ,
              vec_dot_q4_0_q8_1><<<block_nums, block_dims, 0, stream>>>(...);
```
**EN:** The first half of the wrappers covers classic formats (`q4_0` to `q8_0`) and K-quants (`q2_K` to `q6_K`). Each wrapper only plugs in the correct block type and `vec_dot` backend.
**CN:** 前半部分包装函数覆盖经典格式（`q4_0` 到 `q8_0`）和 K 系列量化（`q2_K` 到 `q6_K`）。每个包装函数只负责插入正确的块类型和 `vec_dot` 后端。

### IQ wrappers (L133-L212)
```cpp
mul_mat_vec_q<scalar_t, QK_K, QI2_XXS, block_iq2_xxs, 1, vec_dot_iq2_xxs_q8_1>
    <<<block_nums, block_dims, 0, stream>>>(...);
```
**EN:** The second half extends the same GEMV kernel to non-linear IQ formats. This works because the only format-specific operation is the vector dot product callback.
**CN:** 后半部分把同一个 GEMV 内核扩展到非线性 IQ 格式。之所以能复用，是因为真正与格式相关的部分只有向量点积回调。

## Key Concepts / 关键概念
- Warp-per-row GEMV / 每个 warp 负责一行的 GEMV
- Q8_1 as the shared activation format / 以 Q8_1 作为统一激活格式
- Type-specific dot-product callbacks / 按类型切换的点积回调
- Shared launch shape across many quant formats / 多种量化格式共用启动形状

## Dependencies / 依赖关系
- `ggml-common.h`: quant block structs and shape macros
- `vecdotq.cuh`: `vec_dot_*_q8_1` implementations
- `gguf_kernel.cu`: dispatches to these wrappers in `ggml_mul_mat_vec_a8`
