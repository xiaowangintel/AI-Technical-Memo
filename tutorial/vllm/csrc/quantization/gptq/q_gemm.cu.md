# q_gemm.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/q_gemm.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements the CUDA/ROCm backend for GPTQ matrix multiplication, dequantization, reconstruction fallback, weight shuffling, and PyTorch bindings. / [CN] 实现 GPTQ 矩阵乘法的 CUDA/ROCm 后端，包括直接量化计算、反量化重建回退、权重重排以及 PyTorch 绑定。

## Line-by-Line Analysis / 逐行分析
### [15-54] Shared constants and ROCm BLAS compatibility
```cpp
#include "compat.cuh"
#include "matrix_view.cuh"
#include "qdq_2.cuh"
#include "qdq_3.cuh"
#include "qdq_4.cuh"
#include "qdq_8.cuh"

#define BLOCK_KN_SIZE 128
#define BLOCK_M_SIZE_MAX 8
#define MAX_Q_GEMM_ROWS 50
...
#if defined(USE_ROCM)
__host__ __forceinline__ hipblasStatus_t __compat_hipblasHgemm(...)
```
**EN:** The file gathers all bit-width-specific dequant helpers and defines tile sizes that control the custom kernels. The ROCm block wraps `hipblasHgemm` so the later fallback path can call a uniform half-GEMM API regardless of backend.
**CN:** 该文件集中引入各个位宽的反量化辅助头，并定义自定义内核使用的分块常量。ROCm 兼容段把 `hipblasHgemm` 包装成统一接口，使后续回退路径无需区分后端即可调用半精度 GEMM。

### [56-182] Dot-product helpers for dequantized fragments
```cpp
__forceinline__ __device__ half2 dot22_8(half2 (&dq)[4], const half* a_ptr,
                                         const half2 g_result) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 4; i++) result = __hfma2(dq[i], *a2_ptr++, result);
  return __hadd2(result, g_result);
}
```
**EN:** These helper families (`dot22_8`, `dot22_16`, `dot22_32`, plus `_f` and `_h` variants) encapsulate the inner multiply-accumulate for dequantized blocks. Different return types are used to balance precision and throughput: float accumulators for some 4-bit/8-bit paths, half accumulators for cheaper low-bit kernels.
**CN:** 这一组辅助函数（`dot22_8`、`dot22_16`、`dot22_32` 及其 `_f`、`_h` 变体）封装了反量化块上的内层乘加。不同返回类型在精度与吞吐之间做权衡：某些 4 位/8 位路径使用 float 累加，而低位宽路径则采用更便宜的 half 累加。

### [190-321] Direct 4-bit GPTQ kernel: preload, dequantize, FMA, atomic writeback
```cpp
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_4bit_kernel(...)
{
  __shared__ half block_a[m_count][BLOCK_KN_SIZE];
  ...
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_f(scales, group, n);
  dequant_4bit_8_prep_zero(...);
  ...
  dequant_4bit_8_gptq(load_int4.x, dq[0], z1z16[0], y1y16[0], size_n, false);
  ...
  block_c[m][0] = fma(dot22_8_f(dq[0], a_ptr + m * a_stride), scales[0], block_c[m][0]);
  ...
  atomicAdd(out, result01);
}
```
**EN:** Each block processes a `128 x 4` slice of the quantized weight matrix across up to `m_count` rows of `A`. The kernel caches a tile of activations in shared memory, fetches per-group zeros/scales, dequantizes four output columns at a time, and atomically accumulates into `C` because the K dimension is split across grid-z blocks.
**CN:** 每个线程块会处理量化权重矩阵的一个 `128 x 4` 切片，并同时覆盖 `A` 中最多 `m_count` 行。内核先把激活块缓存到共享内存，再按 group 读取 zero/scale，随后每次反量化 4 个输出列并累加到 `C`；由于 K 维被沿 grid-z 拆分，所以最终写回需要使用原子加。

### [324-742] Bit-width variants and launch selection
```cpp
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_2bit_kernel(...)
...
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_3bit_kernel(...)
...
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_8bit_kernel(...)
...
fp_gemm_half_q_half_gptq_kernel pick_gemm_half_q_half_gptq_kernel(...)
void gemm_half_q_half_cuda_part(...)
```
**EN:** The 2-bit, 3-bit, and 8-bit kernels follow the same structure as the 4-bit path, but use different unpack widths (`16`, `32`, or `8` values) and different dequant helpers. `pick_gemm_half_q_half_gptq_kernel` specializes on `m_count` and bit-width at compile time, while `gemm_half_q_half_cuda_part` computes launch dimensions and dispatches the selected kernel.
**CN:** 2 位、3 位和 8 位内核与 4 位路径共享同一骨架，只是解包宽度分别变成 `16`、`32` 和 `8` 个值，并调用不同的反量化辅助函数。`pick_gemm_half_q_half_gptq_kernel` 会按 `m_count` 和位宽选择编译期特化版本，`gemm_half_q_half_cuda_part` 负责计算 launch 配置并发射对应内核。

### [744-1171] Reconstruct ExLlama-formatted weights back to FP16
```cpp
__global__ void reconstruct_exllama_8bit_kernel(...)
...
__global__ void reconstruct_exllama_4bit_kernel(...)
...
__global__ void reconstruct_exllama_3bit_kernel(...)
...
__global__ void reconstruct_exllama_2bit_kernel(...)
...
void reconstruct_exllama(...)
```
**EN:** These kernels dequantize the packed ExLlama/GPTQ weights into a dense FP16 matrix `temp_dq`. They reuse the same bit-width helpers as the direct GEMM path, but instead of dot products they write each reconstructed weight back to its original row, optionally applying `b_q_perm` to undo permutation.
**CN:** 这些内核把 ExLlama/GPTQ 打包权重反量化为稠密的 FP16 矩阵 `temp_dq`。它们复用了直接 GEMM 路径中的位宽辅助函数，但不再进行点积，而是把每个恢复出的权重写回原始行位置；若存在 `b_q_perm`，还会顺带完成反置换。

### [1173-1383] Alternate non-ExLlama kernels for 4-bit and 8-bit GPTQ
```cpp
__global__ void gemm_half_q_half_alt_4bit_kernel(...)
{
  __shared__ half2 deq2[256][8];
  ...
  deq2[val][off] = __halves2half2(__int2half_rn(val & 0xF), __int2half_rn(val >> 4));
  ...
  res2 = __hfma2(__hfma2(deq2[(tmp >> 0) & 0xff][off], scales_tmp[0], zeros_tmp[0]),
                 blockvec[m][k + 0], res2);
}
```
**EN:** The alternate kernels target the original GPTQ layout with explicit `g_idx` group mappings. The 4-bit version precomputes a tiny shared-memory lookup table for nibble pairs, while both variants fetch per-row group indices, build zero/scale pairs, and accumulate directly against `half2` chunks of the activation matrix.
**CN:** 备用内核针对的是原始 GPTQ 布局，并显式依赖 `g_idx` 做 group 映射。4 位版本会先在共享内存里构造一个小型 nibble 查表，而两种版本都会按行读取 group 索引、构造 zero/scale 对，并直接与激活矩阵中的 `half2` 块做累加。

### [1385-1492] Generic GPTQ reconstruction and 3-bit special-case decoder
```cpp
template <class T, int bit>
__global__ void reconstruct_gptq_kernel(...)
{
  uint32_t w_read = w[blockIdx.y * width + column];
  ...
  half w_item = __hmul(__int2half_rn((int)((w_read >> s) & ((1 << bit) - 1)) - w_zero),
                       w_scale);
}
```
**EN:** For standard GPTQ weights, the generic reconstruction kernel can decode 2/4/8-bit formats by reading one packed word and iterating over its bit fields. 3-bit packing is too irregular for that template, so `reconstruct_gptq_3bit_kernel` manually handles the cross-word cases at indices 10 and 21.
**CN:** 对于标准 GPTQ 权重，通用重建内核可以通过读取一个打包字并遍历其位域来解码 2/4/8 位格式。3 位格式过于不规则，因此 `reconstruct_gptq_3bit_kernel` 单独处理，并特别照顾索引 10 和 21 处的跨字边界情况。

### [1494-1546] Runtime policy: direct quantized GEMM vs reconstruct+BLAS
```cpp
void gemm_half_q_half_cuda(...)
{
  bool use_reconstruct;
  if (use_exllama) {
    use_reconstruct = ((bit == 8 && size_m > MAX_Q_GEMM_ROWS_8BIT) ||
                       (bit != 8 && size_m > MAX_Q_GEMM_ROWS));
  } else {
    use_reconstruct = (bit < 4 || size_m > MAX_ALT_GEMM_ROWS);
  }
  ...
  cublasHgemm(cublas_handle, CUBLAS_OP_N, CUBLAS_OP_N, size_n, size_m, size_k,
              &alpha, temp_dq, size_n, a, size_k, &beta, c, size_n);
}
```
**EN:** The top-level engine chooses between custom quantized kernels and a reconstruction fallback based on bit-width, batch size, and format. Small ExLlama problems use custom kernels; larger or unsupported cases first reconstruct a dense FP16 matrix and then hand the actual GEMM to cuBLAS/hipBLAS.
**CN:** 顶层执行引擎会根据位宽、batch 大小和量化格式，在自定义量化内核与“先重建、后 BLAS”回退之间做选择。较小的 ExLlama 问题会走专用内核，而较大或不适合直接计算的情况则先恢复为稠密 FP16，再交给 cuBLAS/hipBLAS 完成真正的矩阵乘法。

### [1548-1861] Weight permutation, shuffle utilities, and PyTorch API surface
```cpp
__global__ void shuffle_4bit_kernel(...)
...
__global__ void make_sequential_3bit_kernel(...)
...
void shuffle_exllama_weight(uint32_t* q_weight, int* q_perm, int height,
                            int width, int bit) {
  ...
  shuffle_kernel<<<gridDim, blockDim, 0, stream>>>(q_weight, height, width);
}

torch::Tensor gptq_gemm(torch::Tensor a, torch::Tensor b_q_weight, ...)
void gptq_shuffle(torch::Tensor q_weight, torch::Tensor q_perm, int64_t bit)
```
**EN:** The shuffle helpers reorder quantized weights into the layout expected by ExLlama-style direct kernels. If a permutation exists, `make_sequential_*` first rewrites rows into sequential order, then `shuffle_*` performs the per-word bit permutation. The final two functions expose this functionality to PyTorch: `gptq_gemm` runs inference, while `gptq_shuffle` preprocesses weights offline.
**CN:** 这些重排辅助函数把量化权重转换成 ExLlama 风格直接内核所需的布局。如果存在置换表，`make_sequential_*` 会先把行重新排成顺序形式，然后 `shuffle_*` 再执行每个打包字内部的位重排。最后两个函数将能力暴露给 PyTorch：`gptq_gemm` 用于推理计算，`gptq_shuffle` 用于离线预处理权重。

## Key Concepts / 关键概念
- **EN:** The file mixes two strategies: direct quantized GEMM for small/fast cases and dense FP16 reconstruction for larger or awkward cases.
- **CN:** 该文件融合了两种策略：小规模/高效场景走直接量化 GEMM，大规模或不适合的场景走稠密 FP16 重建。
- **EN:** Per-group zero-points and scales are central to every bit-width path.
- **CN:** 按 group 的零点和缩放系数是所有位宽路径的核心。
- **EN:** `b_q_perm` and `g_idx` encode two different layout conventions: ExLlama permutation vs GPTQ group mapping.
- **CN:** `b_q_perm` 与 `g_idx` 对应两种不同布局约定：前者是 ExLlama 置换，后者是 GPTQ group 映射。
- **EN:** Atomic accumulation is required because the K dimension is split across multiple thread blocks.
- **CN:** 由于 K 维被多个线程块分担，结果累加必须使用原子操作。

## Dependencies / 依赖关系
- **EN:** Depends on `compat.cuh`, `matrix_view.cuh`, and all `qdq_*.cuh` helpers for half atomics, matrix access, and per-bit dequantization.
- **CN:** 依赖 `compat.cuh`、`matrix_view.cuh` 和所有 `qdq_*.cuh`，分别提供 half 原子操作、矩阵访问和按位宽反量化能力。
- **EN:** Uses CUDA/ROCm runtime plus cuBLAS/hipBLAS for fallback GEMM execution.
- **CN:** 使用 CUDA/ROCm 运行时以及 cuBLAS/hipBLAS 执行回退 GEMM。
- **EN:** Exported to Python through the `gptq_gemm` and `gptq_shuffle` entry points.
- **CN:** 通过 `gptq_gemm` 和 `gptq_shuffle` 两个入口暴露给 Python 层。
