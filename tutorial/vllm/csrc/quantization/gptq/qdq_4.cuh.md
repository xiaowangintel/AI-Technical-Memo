# qdq_4.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/qdq_4.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements the 4-bit shuffle and dequantization helpers used by GPTQ kernels, including optimized paths for cached zero-point/scale preparation. / [CN] 实现 GPTQ 内核使用的 4 位重排与反量化辅助逻辑，并提供预处理零点/缩放系数的优化路径。

## Line-by-Line Analysis / 逐行分析
### [12-29] Reorder packed nibbles into the execution-friendly layout
```cpp
// Permutation:
//
// 77775555 33331111  66664444 22220000

__forceinline__ __device__ void shuffle_4bit_8(uint32_t* q, int stride) {
  uint32_t qa = q[0];
  uint32_t qb = 0;

#pragma unroll
  for (int i = 0; i < 4; i++) {
    uint32_t qa0 = qa & 0x0f;
    uint32_t qa1 = (qa & 0xf0) >> 4;
    qa >>= 8;
    qb |= (qa1 << (i * 4 + 16));
    qb |= (qa0 << (i * 4));
  }
  q[0] = qb;
}
```
**EN:** A single 32-bit word stores eight 4-bit values. This loop separates low/high nibbles from each byte and repacks them into the lane order expected by the compute kernels, so later vector loads can recover adjacent values efficiently.
**CN:** 一个 32 位字中包含 8 个 4 位值。该循环把每个字节里的低/高 nibble 拆开，并重新排列成计算内核更容易消费的顺序，方便后续向量化加载按相邻元素恢复数据。

### [31-53] Fast dequantization without external scale preparation
```cpp
__forceinline__ __device__ void dequant_4bit_8(const uint32_t q_0,
                                               half2 (&dq)[4], int stride,
                                               const uint32_t zero) {
  const uint32_t c0 = 0x64006400;
  const half y16_ = __float2half_rn(1.0f / 16.0f);
  const half2 y16 = __halves2half2(y16_, y16_);
  const half_uint16 z1_(0xe400 | zero);
  const half z16_ = __hsub(__int2half_rn(-64), __int2half_rn(zero));
  ...
  dq[0] = __hadd2(q0.as_half2, z1);
  dq[1] = __hfma2(q1.as_half2, y16, z16);
  dq[2] = __hadd2(q2.as_half2, z1);
  dq[3] = __hfma2(q3.as_half2, y16, z16);
}
```
**EN:** The function uses bit masks plus half-precision “magic constants” to reinterpret packed 4-bit integers as halves cheaply. Values that sit in the high nibble are multiplied by `1/16`, while the precomputed `z1`/`z16` offsets subtract the zero-point in vector form.
**CN:** 该函数利用位掩码与半精度“魔数”快速把 4 位整数重解释为 half。位于高 nibble 的值通过乘 `1/16` 还原，`z1`/`z16` 则以向量形式完成零点扣除。

### [55-87] Precompute zero-point and optional scale terms
```cpp
__forceinline__ __device__ void dequant_4bit_8_prep_zero_scale(
    const uint32_t zero, const half scale, half2 (&z1z16)[2],
    half2 (&y1y16)[2]) {
  ...
  z1z16[0] = __hmul2(scale2, __half2half2(z1.as_half));
  z1z16[1] = __hmul2(scale2, __half2half2(z16));
  ...
  y1y16[0] = __hmul2(scale2, __half2half2(y1));
  y1y16[1] = __hmul2(scale2, __half2half2(y16));
}
```
**EN:** GPTQ applies one scale/zero pair per group of columns. These helper routines pre-expand the constants into `half2` form so the main kernel can avoid rebuilding them inside its innermost loop.
**CN:** GPTQ 会对每个 group 使用一组 scale/zero。这里先把这些常量展开成 `half2` 形式，避免主内核在最内层循环里反复构造同样的数据。

### [89-121] GPTQ-specific dequant path with optional pre-scaled constants
```cpp
__forceinline__ __device__ void dequant_4bit_8_gptq(const uint32_t q_0,
                                                    half2 (&dq)[4],
                                                    half2 (&z1z16)[2],
                                                    half2 (&y1y16)[2],
                                                    int stride, bool scaled) {
  ...
  if (scaled) {
    dq[0] = __hfma2(q0.as_half2, y1y16[0], z1z16[0]);
    ...
  } else {
    dq[0] = __hadd2(q0.as_half2, z1z16[0]);
    ...
  }
}
```
**EN:** This is the version actually used by `q_gemm.cu`: it accepts prebuilt constants and can operate in either “scale already folded in” or “subtract zero now, multiply scale later” mode. That flexibility lets different kernels trade register pressure against arithmetic cost.
**CN:** 这是 `q_gemm.cu` 真正使用的路径：它接收预构造常量，并可在“已折叠 scale”与“先减 zero、后乘 scale”两种模式间切换。这样不同内核就能在寄存器压力和算术开销之间做取舍。

## Key Concepts / 关键概念
- **EN:** 4-bit values are packed two per byte, so layout reshaping is essential before vectorized math.
- **CN:** 4 位值以每字节两个元素打包，因此在向量计算前必须先重排布局。
- **EN:** The file relies heavily on `half2` math to dequantize two values at once.
- **CN:** 该文件大量使用 `half2` 同时处理两个反量化结果。
- **EN:** GPTQ performance improves when zero-point and scale constants are prepared outside the hot loop.
- **CN:** 在热循环外预处理 zero/scale 常量，是 GPTQ 提升性能的重要手段。

## Dependencies / 依赖关系
- **EN:** Depends on `qdq_util.cuh` for `half2_uint32`, `half_uint16`, and extraction helpers.
- **CN:** 依赖 `qdq_util.cuh` 中的 `half2_uint32`、`half_uint16` 以及位提取辅助函数。
- **EN:** Used by 4-bit direct GEMM, ExLlama reconstruction, and shuffle paths in `q_gemm.cu`.
- **CN:** 被 `q_gemm.cu` 中的 4 位直接 GEMM、ExLlama 重建和重排路径调用。
