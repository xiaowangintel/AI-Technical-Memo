# qdq_3.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/gptq/qdq_3.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Handles the irregular 3-bit GPTQ packing format by reshuffling 96-bit chunks and dequantizing them into `half2` fragments. / [CN] 处理不规则的 3 位 GPTQ 打包格式，将 96 位块重新整理并反量化为 `half2` 片段。

## Line-by-Line Analysis / 逐行分析
### [8-79] Repack three 32-bit words into a kernel-friendly 3-bit layout
```cpp
// Permutation:
//
// v9997775 55333111  u8886664 44222000  (u, v lsb)
...
__forceinline__ __device__ void shuffle_3bit_32(uint32_t* q, int stride) {
  uint32_t qa = q[0 * stride];
  uint32_t qb = q[1 * stride];
  uint32_t qc = q[2 * stride];
  ...
  za |= ((qd & 0x01) >> 0) << 15;
  ...
  q[0 * stride] = za;
  q[1 * stride] = zb;
  q[2 * stride] = zc;
}
```
**EN:** 3-bit weights do not align cleanly with byte or word boundaries, so the helper first carries spill bits across `qa`, `qb`, and `qc`, then repacks them into three reordered words (`za`, `zb`, `zc`). The two extra bits collected in `qd` are inserted back at bit positions 15 and 31 to reconstruct a compact pattern that later kernels can decode regularly.
**CN:** 3 位权重无法像 2/4/8 位那样自然对齐到字节或字，因此这个辅助函数先在 `qa`、`qb`、`qc` 之间搬运溢出的边界比特，再把它们重组为三个新字 (`za`、`zb`、`zc`)。额外收集到的两组边界位保存在 `qd` 中，随后被写回到第 15 位和第 31 位，形成后续内核更容易解析的紧凑布局。

### [81-97] Build scale and zero constants for mixed bit positions
```cpp
__forceinline__ __device__ void dequant_3bit_32(const uint32_t q_0,
                                                const uint32_t q_1,
                                                const uint32_t q_2,
                                                half2 (&dq)[16], int stride,
                                                const uint32_t zero) {
  const uint32_t c0 = 0x64006400;
  const half y8_ = __float2half_rn(1.0f / 8.0f);
  const half y64_ = __float2half_rn(1.0f / 64.0f);
  ...
  const half2 z1 = __halves2half2(z1_.as_half, z1_.as_half);
  const half2 z8 = __halves2half2(z8_, z8_);
  const half2 z64 = __halves2half2(z64_, z64_);
}
```
**EN:** Because 3-bit values land at different bit offsets after packing, the dequantizer needs multiple scaling factors (`1`, `1/8`, `1/64`) and matching zero-point offsets. This setup stage constructs those constants once before decoding the 32 values.
**CN:** 由于 3 位值在打包后会落在不同的位偏移位置，反量化时需要同时处理 `1`、`1/8`、`1/64` 三类缩放，以及匹配的零点偏移。这里先把这些常量准备好，再去解码 32 个值。

### [98-127] Decode 32 values across three input words
```cpp
uint32_t qa = q_0;
uint32_t qb = q_1;
uint32_t qc = q_2;

half2_uint32 q0((qa & 0x00070007) | c0);
...
qa >>= 9;
qa &= 0x00010001;
...
qc >>= 7;
qc &= 0x00040004;
half2_uint32 q15((qa | qb | qc) | c0);
```
**EN:** The function peels off pairs of 3-bit values from different bit fields, occasionally carrying single high bits from `qa`, `qb`, and `qc`. `q15` is the special tail case where leftover bits from all three words are OR-ed together to reconstruct the final pair.
**CN:** 该函数从不同的位域中逐步剥离 3 位值对，并在必要时从 `qa`、`qb`、`qc` 中取出高位残留。`q15` 是最特殊的尾部情况，需要把三个输入字中的残余位合并后才能还原最后一对元素。

### [128-143] Materialize sixteen `half2` outputs
```cpp
dq[0] = __hadd2(q0.as_half2, z1);
dq[1] = __hfma2(q1.as_half2, y8, z8);
...
dq[14] = __hfma2(q14.as_half2, y64, z64);
dq[15] = __hadd2(q15.as_half2, z1);
```
**EN:** Each decoded pair is converted into a dequantized `half2`, using the matching multiplier and zero correction for its source bit position. The result is a 32-element fragment ready for the 3-bit GEMM kernels.
**CN:** 每一对解码结果都会结合对应的乘数与零点修正，生成一个反量化后的 `half2`。最终得到 32 个元素组成的片段，可直接供 3 位 GEMM 内核使用。

## Key Concepts / 关键概念
- **EN:** 3-bit packing is irregular enough that both shuffle and dequant paths need custom logic.
- **CN:** 3 位打包非常不规则，因此重排与反量化都必须使用专门逻辑。
- **EN:** Spill bits crossing 32-bit boundaries are explicitly recovered and reinserted.
- **CN:** 跨越 32 位边界的溢出比特需要显式提取并重新写回。
- **EN:** The output still follows the common `half2` contract expected by higher-level kernels.
- **CN:** 尽管编码复杂，输出仍遵守上层内核统一使用的 `half2` 接口。

## Dependencies / 依赖关系
- **EN:** Depends on `qdq_util.cuh` for half/int reinterpret helpers and `exb`-style extraction utilities.
- **CN:** 依赖 `qdq_util.cuh` 中的 half/int 重解释与位提取辅助逻辑。
- **EN:** Used by 3-bit GEMM, shuffle, and reconstruction code in `q_gemm.cu`.
- **CN:** 被 `q_gemm.cu` 中的 3 位 GEMM、重排和重建代码使用。
