# dequant.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/dequant.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provide ultra-low-overhead dequantization helpers for Marlin weight formats, including fused scale/zero-point handling. / [CN] 为 Marlin 权重格式提供超低开销反量化辅助函数，并支持与 scale/zero-point 的融合处理。

## Line-by-Line Analysis / 逐行分析
### Design note: when bit-ops can absorb scale or zero-point work
```cpp
/*
 * Fast Dequantization (Converting INT4/INT8/FP4/FP8 to FP16/BF16)
 * ...
 * Since the dequantized weights typically require subtracting the zero point and
 * applying a scale factor, the floating-point computation step can be fused ...
 */
```
**EN:** The opening comment is not decorative; it explains the optimization strategy used throughout the file. Marlin tries to move work from expensive floating-point instructions into cheap bit manipulation whenever the math permits.
**CN:** 开头这段注释并不是装饰，而是整份文件的优化思路说明。Marlin 会在数学上可行时，把原本昂贵的浮点操作尽量前移到廉价的位运算中完成。

### Low-level PTX helpers
```cpp
template <int lut>
__device__ inline int lop3(int a, int b, int c) {
  asm volatile("lop3.b32 %0, %1, %2, %3, %4;
" ...);
}

template <int start_byte, int mask>
__device__ inline uint32_t prmt(uint32_t a) {
  asm volatile("prmt.b32 %0, %1, %2, %3;
" ...);
}
```
**EN:** `lop3` and `prmt` expose two important PTX instructions: ternary logic and byte permutation. The specialized dequantizers use them to unpack low-bit weights without temporary scalar loops.
**CN:** `lop3` 和 `prmt` 暴露了两个关键 PTX 指令：三输入逻辑和字节重排。各个特化的反量化函数正是靠它们在没有临时标量循环的情况下解包低比特权重。

### INT4 / INT8 to FP16 or BF16
```cpp
template <>
__device__ inline void dequant<half2, vllm::kU4B8.id(), false>(int q,
                                                               half2* frag_b) {
  int lo = lop3<(0xf0 & 0xcc) | 0xaa>(q, LO, EX);
  int hi = lop3<(0xf0 & 0xcc) | 0xaa>(q, HI, EX);
  frag_b[0] = __hsub2(*reinterpret_cast<half2*>(&lo), ...);
  frag_b[1] = __hfma2(*reinterpret_cast<half2*>(&hi), ...);
}
```
**EN:** These specializations unpack packed 4-bit or 8-bit integers directly into `half2`/`bfloat162` fragments. The `skip_flop` template flag selects whether the function emits only the bit-level reconstruction or also folds in the implicit symmetric offset.
**CN:** 这些特化直接把打包后的 4-bit / 8-bit 整数解包成 `half2` / `bfloat162` 片段。`skip_flop` 模板参数决定函数是只做位级重建，还是顺便融合掉隐含的对称偏移。

### FP8 / FP4 dequantization paths
```cpp
template <>
__device__ inline void dequant<half2, vllm::kFE4M3fn.id(), false>(
    int q, half2* frag_b) {
  dequant<half2, vllm::kFE4M3fn.id(), true>(q, frag_b);
  const half2 bias_reg = __float2half2_rn(float(1 << BIAS_OFFSET));
  frag_b[1] = __hmul2(frag_b[1], bias_reg);
  frag_b[0] = __hmul2(frag_b[0], bias_reg);
}
```
**EN:** FP8/FP4 formats are reconstructed by moving sign/exponent bits into a wider floating format and then applying an exponent-bias multiplier. The code has parallel half/BF16 versions because the target exponent layouts differ.
**CN:** FP8/FP4 的重建方式是先把符号位和指数位搬运到更宽的浮点格式，再乘上指数偏置因子。由于目标格式的指数布局不同，因此这里分别提供了 half 和 BF16 两套实现。

### Special dequant outputs for 8-bit activation paths
```cpp
template <>
__device__ inline void dequant<int32_t, vllm::kU4B8.id(), true>(
    int q, int32_t* frag_b) {
  frag_b[0] = ((q & 0x0F0F0F0F | MASK) - repeated_zp) ^ MASK;
}

template <>
__device__ inline void dequant<__nv_fp8x4_e4m3, vllm::kU4B8.id(), true>(
    int q, __nv_fp8x4_e4m3* frag_b) {
```
**EN:** Not all paths dequantize into FP16/BF16. For W4A8 and related kernels, Marlin also materializes INT8 or FP8 fragments directly so that later MMA instructions can consume them without another conversion step.
**CN:** 并非所有路径都会反量化到 FP16/BF16。对于 W4A8 等内核，Marlin 还会直接生成 INT8 或 FP8 片段，以便后续 MMA 指令无需再次转换即可直接消费。

### FP8 scale decoding
```cpp
template <typename scalar_t2, vllm::ScalarTypeId s_type_id>
__device__ inline void dequant_fp8_scales(int q, scalar_t2* frag_b);
```
**EN:** Weight values are not the only packed data in Marlin. Scale tensors can also be stored in FP8 variants, so this helper decodes scale fragments before they are multiplied into dequantized weights or accumulators.
**CN:** 在 Marlin 中，被打包的数据不只有权重。scale 张量也可能以 FP8 形式存储，因此这里的辅助函数会先把 scale 片段解码，再参与到反量化权重或累加器的缩放中。

### Fused subtract-zero-point-and-dequant path
```cpp
template <typename scalar_t2, vllm::ScalarTypeId w_type_id,
          bool skip_flop = false>
__device__ inline void sub_zp_and_dequant(int q, scalar_t2* frag_b, int zp);
```
**EN:** This final group specializes the case where a packed integer zero-point should be removed before or during dequantization. It is especially important for the newer AWQ/HQQ-style paths that want to keep the hot loop branch-free.
**CN:** 最后一组特化处理的是“在反量化前或反量化过程中减去打包整数 zero-point”的情况。这对较新的 AWQ/HQQ 风格路径尤其关键，因为它们希望热点循环保持无分支。

## Key Concepts / 关键概念
- Dequantization is format-specific and highly bit-manipulation-heavy. / 反量化高度依赖具体格式，并大量使用位操作。
- `skip_flop` decides whether scale/offset work is deferred or fused. / `skip_flop` 决定缩放/偏移操作是延后执行还是提前融合。
- The file supports integer, FP8, and FP4 weight encodings. / 该文件同时支持整数、FP8 和 FP4 权重编码。

## Dependencies / 依赖关系
- Depends on `marlin_dtypes.cuh` for target fragment types. / 依赖 `marlin_dtypes.cuh` 提供目标片段类型。
- Used by `marlin_template.h` inside the inner dequantize-and-matmul loop. / 在 `marlin_template.h` 的内层“反量化+矩阵乘”循环中被调用。
