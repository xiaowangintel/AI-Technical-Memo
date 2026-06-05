# qdq_8.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/qdq_8.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides the 8-bit GPTQ unpack/dequant helpers that convert packed bytes into `half2` vectors for fused dot products. / [CN] 提供 8 位 GPTQ 的解包与反量化辅助函数，把打包字节转换成供融合点积使用的 `half2` 向量。

## Line-by-Line Analysis / 逐行分析
### [5-13] Header guard and uniform helper interface
```cpp
#ifndef _qdq_8_cuh
#define _qdq_8_cuh

#include "qdq_util.cuh"

namespace vllm {
namespace gptq {

__forceinline__ __device__ void shuffle_8bit_4(uint32_t* q, int stride) {}
```
**EN:** 8-bit values are already byte-aligned inside each 32-bit word, so there is no expensive bit permutation step. The empty `shuffle_8bit_4` exists only to keep the same API shape as the 2/3/4-bit helpers.
**CN:** 8 位值在 32 位字内已经按字节对齐，因此不需要额外的复杂位重排。空实现的 `shuffle_8bit_4` 只是为了与 2/3/4 位辅助函数保持统一接口。

### [15-25] Extract bytes and subtract the zero-point
```cpp
__forceinline__ __device__ void dequant_8bit_8(const uint32_t q_0,
                                               const uint32_t q_1,
                                               half2 (&dq)[4], int stride,
                                               const uint32_t zero) {
  half dqh[8];
  for (int i = 0; i < 4; i++) dqh[i] = dq_ns(exb(q_0, i * 8, 0xff), zero);
  for (int i = 0; i < 4; i++) dqh[i + 4] = dq_ns(exb(q_1, i * 8, 0xff), zero);

  for (int i = 0; i < 4; i++)
    dq[i] = __halves2half2(dqh[i * 2], dqh[i * 2 + 1]);
}
```
**EN:** `exb` pulls out each quantized byte, `dq_ns` converts it to half precision after removing the zero-point, and the final loop groups the eight scalars into four `half2` lanes. The routine intentionally does not apply scales, because group scales are handled later by the GEMM kernels.
**CN:** `exb` 逐个提取量化字节，`dq_ns` 在扣除零点后把结果转成半精度，最后一轮循环再把 8 个标量组合为 4 个 `half2` 通道。这里故意不乘缩放系数，因为 group scale 会在后续 GEMM 内核中统一处理。

## Key Concepts / 关键概念
- **EN:** The 8-bit path is the simplest GPTQ format because the packed layout already matches byte boundaries.
- **CN:** 8 位路径是最简单的 GPTQ 格式，因为其打包布局天然按字节边界组织。
- **EN:** Producing `half2` directly avoids extra packing work in the compute kernels.
- **CN:** 直接生成 `half2` 可以避免计算内核再次打包数据。

## Dependencies / 依赖关系
- **EN:** Depends on `qdq_util.cuh` for `exb` and `dq_ns`.
- **CN:** 依赖 `qdq_util.cuh` 提供的 `exb` 与 `dq_ns`。
- **EN:** Used by the 8-bit GPTQ GEMM and reconstruction kernels in `csrc/quantization/gptq/q_gemm.cu`.
- **CN:** 被 `csrc/quantization/gptq/q_gemm.cu` 中的 8 位 GPTQ GEMM 与重建内核调用。
