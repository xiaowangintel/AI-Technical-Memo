# qdq_2.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/qdq_2.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements 2-bit GPTQ bit shuffling and dequantization helpers for 16 values per packed word. / [CN] 实现 2 位 GPTQ 的位重排与反量化辅助逻辑，每个打包字可恢复 16 个量化值。

## Line-by-Line Analysis / 逐行分析
### [13-30] Reorder sixteen 2-bit values into two aligned halves
```cpp
// Permutation:
//
// ffddbb99 77553311  eeccaa88 66442200

__forceinline__ __device__ void shuffle_2bit_16(uint32_t* q, int stride) {
  uint32_t qa = q[0];
  uint32_t qb = 0;
  ...
  for (int i = 0; i < 8; i++) {
    uint32_t qa0 = qa & 0x03;
    uint32_t qa1 = (qa & 0x0c) >> 2;
    qa >>= 4;
    qb |= (qa1 << (i * 2 + 16));
    qb |= (qa0 << (i * 2));
  }
  q[0] = qb;
}
```
**EN:** Each byte contains four 2-bit values. This shuffle groups alternating pairs into low and high 16-bit regions, producing the pattern expected by later vectorized extraction code.
**CN:** 每个字节包含 4 个 2 位值。该重排把交错的值对分别聚合到低 16 位和高 16 位区域，生成后续向量提取代码所需的布局。

### [32-50] Prepare multiple scaling lanes for differently shifted values
```cpp
__forceinline__ __device__ void dequant_2bit_16(const uint32_t q_0,
                                                half2 (&dq)[8], int stride,
                                                const uint32_t zero) {
  const uint32_t c0 = 0x64006400;
  const half y4_ = __float2half_rn(1.0f / 4.0f);
  const half y16_ = __float2half_rn(1.0f / 16.0f);
  const half y64_ = __float2half_rn(1.0f / 64.0f);
  ...
  const half2 z64 = __half2half2(z64_);
}
```
**EN:** After masking different 2-bit fields, the resulting half reinterpretations correspond to values shifted by `1`, `4`, `16`, or `64`. The helper therefore creates matching multipliers and zero-point corrections for each extraction pattern.
**CN:** 不同 2 位字段在被掩码取出后，对应的 half 重解释值会带有 `1`、`4`、`16`、`64` 等不同缩放因子。因此这里要为每种提取模式分别准备对应的乘数和零点修正。

### [52-70] Decode a packed word into eight `half2` vectors
```cpp
uint32_t qa = q_0;
half2_uint32 q0((qa & 0x00030003) | c0);
half2_uint32 q1((qa & 0x000c000c) | c0);
...
dq[0] = __hadd2(q0.as_half2, z1);
dq[1] = __hfma2(q1.as_half2, y4, z4);
...
dq[7] = __hfma2(q7.as_half2, y64, z64);
```
**EN:** The function extracts sixteen 2-bit values as eight pairs, maps them through the correct affine transform, and returns the results in `half2` form. The 2-bit path is compact, but still uses the same vectorized contract as the higher-bit helpers.
**CN:** 该函数把 16 个 2 位值解成 8 对，通过正确的仿射变换恢复数值，并以 `half2` 形式返回。虽然 2 位路径更紧凑，但仍遵守与高位宽辅助函数相同的向量化接口。

## Key Concepts / 关键概念
- **EN:** 2-bit packing is dense enough that one 32-bit word represents 16 quantized values.
- **CN:** 2 位打包非常紧凑，一个 32 位字就能表示 16 个量化值。
- **EN:** Different bit offsets require different rescaling factors during half reinterpretation.
- **CN:** 不同位偏移在 half 重解释时需要匹配不同的还原因子。

## Dependencies / 依赖关系
- **EN:** Depends on `qdq_util.cuh` for packed half helper unions.
- **CN:** 依赖 `qdq_util.cuh` 中的打包 half 辅助联合体。
- **EN:** Used by 2-bit direct GEMM, reconstruction, and shuffle code in `q_gemm.cu`.
- **CN:** 被 `q_gemm.cu` 中的 2 位直接 GEMM、重建和重排代码调用。
