# dequant.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/marlin/dequant.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Local implementation details
```cpp
/*
Fast Dequantization (Converting INT4/INT8/FP4/FP8 to FP16/BF16)

The process of fast dequantization can be summarized as a combination
of bitwise operations and floating-point computations:

weight =>(bit_op / bitwise operations)=>
f16_value =>(flop / floating-point computation)=>
dequantized_weight

Since the dequantized weights typically require subtracting the zero point and
applying a scale factor, the floating-point computation step can be fused with
the zero-point subtraction and scaling operations.

The following are the parts that need to be modified for the fused operation
of zero-point subtraction and scaling.

## INT4 => FP16/BF16 or INT8 => FP16

The floating-point computation is `__hsub2`

If has zero points:
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 24-48: Local implementation details
```cpp
    flop(bit_op(weight)) - flop(bit_op(zp))
  = sub(bit_op(weight), bias) - sub(bit_op(zp), bias)
  = bit_op(weight) - bit_op(zp)

so we don't need additional modification.

If has float zero points:

    flop(bit_op(weight)) - fzp
  = sub(bit_op(weight), bias) - fzp
  = bit_op(weight) - (fzp + bias)

where the `fzp + bias` can be computed at weight loading. But this
may have accuracy issue, so we should not use this in most cases.

If has not zero points:

    scale(flop(bit_op(weight)))
  = scale(sub(bit_op(weight), bias))
  = scale(bit_op(weight)) - scale(bias)
  = fma(bit_op(weight), scale_factor, scale(bias))

where the `scale(bias)` can be cached. But this may have accuracy issue,
so we should not use this in most cases.
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 49-69: Local implementation details
```cpp

## INT8 => BF16

INT8 => BF16 is a special case, it use byte_perm instead of flop.
We cannot fused byte_perm with scaling.


## FP4/FP8 => FP16/BF16

    scale(flop(bit_op(weight)))
  = scale(mul(bit_op(weight), multiplier))
  = mul(bit_op(weight), scale_factor * multiplier)

where `scale_factor * multiplier` can be computed at weight loading.

*/

#include "marlin_dtypes.cuh"

namespace MARLIN_NAMESPACE_NAME {
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 70-92: Device helpers and synchronization
```cpp
#if !defined(__CUDA_ARCH__) || __CUDA_ARCH__ >= 800
// Lookup-table based 3-input logical operation; explicitly used for
// dequantization as the compiler does not seem to automatically recognize it in
// all cases.
template <int lut>
__device__ inline int lop3(int a, int b, int c) {
  int res;
  asm volatile("lop3.b32 %0, %1, %2, %3, %4;\n" : "=r"(res) : "r"(a), "r"(b), "r"(c), "n"(lut));
  return res;
}

// Constructs destination register by taking bytes from 2 sources (based on
// mask)
template <int start_byte, int mask>
__device__ inline uint32_t prmt(uint32_t a) {
  uint32_t res;
  asm volatile("prmt.b32 %0, %1, %2, %3;\n" : "=r"(res) : "r"(a), "n"(start_byte), "n"(mask));
  return res;
}

template <typename scalar_t2, sglang::ScalarTypeId w_type_id, bool skip_flop = false>
__device__ inline void dequant(int q, scalar_t2* frag_b);
```
**EN:** This section implements `defined`, `prmt`, `dequant`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`defined`、`prmt`、`dequant`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 93-114: Device helpers and synchronization
```cpp
//
// Efficiently dequantize 4bit values packed in an int32 value into a full
// B-fragment of 4 fp16 values. We mostly follow the strategy in the link below,
// with some small changes:
// - FP16:
// https://github.com/NVIDIA/FasterTransformer/blob/release/v5.3_tag/src/fastertransformer/cutlass_extensions/include/cutlass_extensions/interleaved_numeric_conversion.h#L215-L287
// - BF16:
// https://github.com/NVIDIA/FasterTransformer/blob/release/v5.3_tag/src/fastertransformer/cutlass_extensions/include/cutlass_extensions/interleaved_numeric_conversion.h#L327-L385
//
template <>
__device__ inline void dequant<half2, sglang::kU4B8.id(), true>(int q, half2* frag_b) {
  const int MASK = 0x000f000f;
  const int EX = 0x64006400;
  // Guarantee that the `(a & b) | c` operations are LOP3s.
  int lo = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  q >>= 4;
  int hi = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);

  frag_b[0] = *reinterpret_cast<half2*>(&lo);
  frag_b[1] = *reinterpret_cast<half2*>(&hi);
}
```
**EN:** This section implements `id`, `MASK`, `EX`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`MASK`、`EX`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 115-138: Device helpers and synchronization
```cpp
template <>
__device__ inline void dequant<half2, sglang::kU4B8.id(), false>(int q, half2* frag_b) {
  const int LO = 0x000f000f;
  const int HI = 0x00f000f0;
  const int EX = 0x64006400;
  // Guarantee that the `(a & b) | c` operations are LOP3s.
  // clang-format off
  int lo = lop3<(0xf0 & 0xcc) | 0xaa>(q, LO, EX);
  int hi = lop3<(0xf0 & 0xcc) | 0xaa>(q, HI, EX);
  // clang-format on
  // We want signed int4 outputs, hence we fuse the `-8` symmetric zero point
  // directly into `SUB` and `ADD`.
  const int SUB = 0x64086408;
  const int MUL = 0x2c002c00;
  const int ADD = 0xd480d480;
  frag_b[0] = __hsub2(*reinterpret_cast<half2*>(&lo), *reinterpret_cast<const half2*>(&SUB));
  frag_b[1] = __hfma2(
      *reinterpret_cast<half2*>(&hi), *reinterpret_cast<const half2*>(&MUL), *reinterpret_cast<const half2*>(&ADD));
}

template <>
__device__ inline void dequant<half2, sglang::kU4.id(), true>(int q, half2* frag_b) {
  dequant<half2, sglang::kU4B8.id(), true>(q, frag_b);
}
```
**EN:** This section implements `id`, `__hsub2`, `__hfma2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`__hfma2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 139-164: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant<half2, sglang::kU4.id(), false>(int q, half2* frag_b) {
  const int LO = 0x000f000f;
  const int HI = 0x00f000f0;
  const int EX = 0x64006400;
  // Guarantee that the `(a & b) | c` operations are LOP3s.
  // clang-format off
  int lo = lop3<(0xf0 & 0xcc) | 0xaa>(q, LO, EX);
  int hi = lop3<(0xf0 & 0xcc) | 0xaa>(q, HI, EX);
  // clang-format on
  // We want signed int4 outputs, hence we fuse the `-8` symmetric zero point
  // directly into `SUB` and `ADD`.
  const int SUB = 0x64006400;
  const int MUL = 0x2c002c00;
  const int ADD = 0xd400d400;
  frag_b[0] = __hsub2(*reinterpret_cast<half2*>(&lo), *reinterpret_cast<const half2*>(&SUB));
  frag_b[1] = __hfma2(
      *reinterpret_cast<half2*>(&hi), *reinterpret_cast<const half2*>(&MUL), *reinterpret_cast<const half2*>(&ADD));
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU4B8.id(), true>(int q, nv_bfloat162* frag_b) {
  static constexpr uint32_t MASK = 0x000f000f;
  static constexpr uint32_t EX = 0x43004300;
```
**EN:** This section implements `id`, `__hsub2`, `__hfma2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`__hfma2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 165-189: Device helpers and synchronization
```cpp
  // Guarantee that the `(a & b) | c` operations are LOP3s.
  // clang-format off
  int lo = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  q >>= 4;
  int hi = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  // clang-format on

  frag_b[0] = *reinterpret_cast<nv_bfloat162*>(&lo);
  frag_b[1] = *reinterpret_cast<nv_bfloat162*>(&hi);
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU4B8.id(), false>(int q, nv_bfloat162* frag_b) {
  dequant<nv_bfloat162, sglang::kU4B8.id(), true>(q, frag_b);

  static constexpr uint32_t SUB = 0x43084308;

  frag_b[0] = __hsub2(frag_b[0], *reinterpret_cast<const nv_bfloat162*>(&SUB));
  frag_b[1] = __hsub2(frag_b[1], *reinterpret_cast<const nv_bfloat162*>(&SUB));
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU4.id(), true>(int q, nv_bfloat162* frag_b) {
  dequant<nv_bfloat162, sglang::kU4B8.id(), true>(q, frag_b);
}
```
**EN:** This section implements `id`, `__hsub2`, `SUB`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`SUB`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 190-214: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU4.id(), false>(int q, nv_bfloat162* frag_b) {
  dequant<nv_bfloat162, sglang::kU4.id(), true>(q, frag_b);

  static constexpr uint32_t SUB = 0x43004300;

  frag_b[0] = __hsub2(frag_b[0], *reinterpret_cast<const nv_bfloat162*>(&SUB));
  frag_b[1] = __hsub2(frag_b[1], *reinterpret_cast<const nv_bfloat162*>(&SUB));
}

//
// Fast Int8ToFp16/Int8ToBf16: Efficiently dequantize 8bit int values to fp16 or
// bf16 Reference:
// - FP16:
// https://github.com/NVIDIA/FasterTransformer/blob/release/v5.3_tag/src/fastertransformer/cutlass_extensions/include/cutlass_extensions/interleaved_numeric_conversion.h#L53-L85
// - BF16:
// https://github.com/NVIDIA/FasterTransformer/blob/release/v5.3_tag/src/fastertransformer/cutlass_extensions/include/cutlass_extensions/interleaved_numeric_conversion.h#L125-L175
//
template <>
__device__ inline void dequant<half2, sglang::kU8B128.id(), true>(int q, half2* frag_b) {
  static constexpr uint32_t mask_for_elt_01 = 0x5250;
  static constexpr uint32_t mask_for_elt_23 = 0x5351;
  static constexpr uint32_t start_byte_for_fp16 = 0x64646464;
```
**EN:** This section implements `id`, `__hsub2`, `SUB`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`SUB`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 215-239: Device helpers and synchronization
```cpp
  uint32_t lo = prmt<start_byte_for_fp16, mask_for_elt_01>(q);
  uint32_t hi = prmt<start_byte_for_fp16, mask_for_elt_23>(q);

  frag_b[0] = *reinterpret_cast<half2*>(&lo);
  frag_b[1] = *reinterpret_cast<half2*>(&hi);
}

template <>
__device__ inline void dequant<half2, sglang::kU8B128.id(), false>(int q, half2* frag_b) {
  dequant<half2, sglang::kU8B128.id(), true>(q, frag_b);

  static constexpr uint32_t I8s_TO_F16s_MAGIC_NUM = 0x64806480;
  frag_b[0] = __hsub2(frag_b[0], *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
  frag_b[1] = __hsub2(frag_b[1], *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
}

template <>
__device__ inline void dequant<half2, sglang::kU8.id(), true>(int q, half2* frag_b) {
  dequant<half2, sglang::kU8B128.id(), true>(q, frag_b);
}

template <>
__device__ inline void dequant<half2, sglang::kU8.id(), false>(int q, half2* frag_b) {
  dequant<half2, sglang::kU8.id(), true>(q, frag_b);
```
**EN:** This section implements `id`, `__hsub2`, `I8s_TO_F16s_MAGIC_NUM`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`I8s_TO_F16s_MAGIC_NUM`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 240-264: Device helpers and synchronization
```cpp
  static constexpr uint32_t I8s_TO_F16s_MAGIC_NUM = 0x64006400;
  frag_b[0] = __hsub2(frag_b[0], *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
  frag_b[1] = __hsub2(frag_b[1], *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU8B128.id(), false>(int q, nv_bfloat162* frag_b) {
  float fp32_intermediates[4];
  uint32_t* fp32_intermediates_casted = reinterpret_cast<uint32_t*>(fp32_intermediates);

  static constexpr uint32_t fp32_base = 0x4B000000;
  fp32_intermediates_casted[0] = __byte_perm(q, fp32_base, 0x7650);
  fp32_intermediates_casted[1] = __byte_perm(q, fp32_base, 0x7652);
  fp32_intermediates_casted[2] = __byte_perm(q, fp32_base, 0x7651);
  fp32_intermediates_casted[3] = __byte_perm(q, fp32_base, 0x7653);

  fp32_intermediates[0] -= 8388736.f;
  fp32_intermediates[1] -= 8388736.f;
  fp32_intermediates[2] -= 8388736.f;
  fp32_intermediates[3] -= 8388736.f;

  uint32_t* bf16_result_ptr = reinterpret_cast<uint32_t*>(frag_b);
  bf16_result_ptr[0] = __byte_perm(fp32_intermediates_casted[0], fp32_intermediates_casted[1], 0x7632);
  bf16_result_ptr[1] = __byte_perm(fp32_intermediates_casted[2], fp32_intermediates_casted[3], 0x7632);
}
```
**EN:** This section implements `id`, `__hsub2`, `__byte_perm`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__hsub2`、`__byte_perm`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 265-286: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kU8.id(), false>(int q, nv_bfloat162* frag_b) {
  float fp32_intermediates[4];
  uint32_t* fp32_intermediates_casted = reinterpret_cast<uint32_t*>(fp32_intermediates);

  static constexpr uint32_t fp32_base = 0x4B000000;
  fp32_intermediates_casted[0] = __byte_perm(q, fp32_base, 0x7650);
  fp32_intermediates_casted[1] = __byte_perm(q, fp32_base, 0x7652);
  fp32_intermediates_casted[2] = __byte_perm(q, fp32_base, 0x7651);
  fp32_intermediates_casted[3] = __byte_perm(q, fp32_base, 0x7653);

  fp32_intermediates[0] -= 8388608.f;
  fp32_intermediates[1] -= 8388608.f;
  fp32_intermediates[2] -= 8388608.f;
  fp32_intermediates[3] -= 8388608.f;

  uint32_t* bf16_result_ptr = reinterpret_cast<uint32_t*>(frag_b);
  bf16_result_ptr[0] = __byte_perm(fp32_intermediates_casted[0], fp32_intermediates_casted[1], 0x7632);
  bf16_result_ptr[1] = __byte_perm(fp32_intermediates_casted[2], fp32_intermediates_casted[3], 0x7632);
}
```
**EN:** This section implements `id`, `__byte_perm`, `fp32_base`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__byte_perm`、`fp32_base`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 287-310: Device helpers and synchronization
```cpp
template <>
__device__ inline void dequant<half2, sglang::kFE4M3fn.id(), true>(int q, half2* frag_b) {
  // Constants for FP8 (E4M3) and FP16 formats
  constexpr int FP8_EXPONENT = 4, FP16_EXPONENT = 5;
  constexpr int RIGHT_SHIFT = FP16_EXPONENT - FP8_EXPONENT;
  constexpr int MASK = 0x7F007F00;

  // Extract and shift FP8 values to FP16 format
  int Out1 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 8;
  int Out2 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const half2*>(&Out1);
  frag_b[0] = *reinterpret_cast<const half2*>(&Out2);
}

template <>
__device__ inline void dequant<half2, sglang::kFE4M3fn.id(), false>(int q, half2* frag_b) {
  dequant<half2, sglang::kFE4M3fn.id(), true>(q, frag_b);

  // Constants for FP8 (E4M3) and FP16 formats
  constexpr int FP8_EXPONENT = 4, FP16_EXPONENT = 5;
```
**EN:** This section implements `id`, `FP8_EXPONENT`, `RIGHT_SHIFT`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`FP8_EXPONENT`、`RIGHT_SHIFT`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 311-336: Device helpers and synchronization
```cpp
  // Construct and apply exponent bias
  constexpr int BIAS_OFFSET = (1 << (FP16_EXPONENT - 1)) - (1 << (FP8_EXPONENT - 1));
  const half2 bias_reg = __float2half2_rn(float(1 << BIAS_OFFSET));

  // Convert to half2 and apply bias
  frag_b[1] = __hmul2(frag_b[1], bias_reg);
  frag_b[0] = __hmul2(frag_b[0], bias_reg);
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kFE4M3fn.id(), true>(int q, nv_bfloat162* frag_b) {
  // Constants for FP8 (E4M3) and BF16 formats
  constexpr int FP8_EXPONENT = 4, BF16_EXPONENT = 8;
  constexpr int RIGHT_SHIFT = BF16_EXPONENT - FP8_EXPONENT;

  constexpr int MASK = 0x7F007F00;

  // Extract and shift FP8 values to BF16 format
  int Out1 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 8;
  int Out2 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const nv_bfloat162*>(&Out1);
  frag_b[0] = *reinterpret_cast<const nv_bfloat162*>(&Out2);
}
```
**EN:** This section implements `id`, `__float2half2_rn`, `__hmul2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__float2half2_rn`、`__hmul2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 337-363: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kFE4M3fn.id(), false>(int q, nv_bfloat162* frag_b) {
  dequant<nv_bfloat162, sglang::kFE4M3fn.id(), true>(q, frag_b);

  // Constants for FP8 (E4M3) and BF16 formats
  constexpr int FP8_EXPONENT = 4, BF16_EXPONENT = 8;

  // Construct and apply exponent bias
  constexpr int BIAS_OFFSET = (1 << (BF16_EXPONENT - 1)) - (1 << (FP8_EXPONENT - 1));
  // Add 127 (float exponent bias) to BIAS_OFFSET and shift to float exponent
  // position
  constexpr uint32_t BIAS = (BIAS_OFFSET + 127) << 23;
  const nv_bfloat162 bias_reg = __float2bfloat162_rn(*reinterpret_cast<const float*>(&BIAS));

  // Convert to bfloat162 and apply bias
  frag_b[1] = __hmul2(frag_b[1], bias_reg);
  frag_b[0] = __hmul2(frag_b[0], bias_reg);
}

template <>
__device__ inline void dequant<half2, sglang::kFE2M1f.id(), true>(int q, half2* frag_b) {
  // Constants for FP4 (E2M1) and FP16 formats
  constexpr int FP4_EXPONENT = 2, FP16_EXPONENT = 5;
  constexpr int RIGHT_SHIFT = FP16_EXPONENT - FP4_EXPONENT;
  constexpr int MASK = 0x70007000;
```
**EN:** This section implements `id`, `__float2bfloat162_rn`, `__hmul2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__float2bfloat162_rn`、`__hmul2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 364-388: Device helpers and synchronization
```cpp
  // Extract and shift FP4 values to FP16 format
  int Out1 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 4;
  int Out2 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const half2*>(&Out1);
  frag_b[0] = *reinterpret_cast<const half2*>(&Out2);
}

template <>
__device__ inline void dequant<half2, sglang::kFE2M1f.id(), false>(int q, half2* frag_b) {
  dequant<half2, sglang::kFE2M1f.id(), true>(q, frag_b);

  // Constants for FP4 (E2M1) and FP16 formats
  constexpr int FP4_EXPONENT = 2, FP16_EXPONENT = 5;

  // Construct and apply exponent bias
  constexpr int BIAS_OFFSET = (1 << (FP16_EXPONENT - 1)) - (1 << (FP4_EXPONENT - 1));
  const half2 bias_reg = __float2half2_rn(float(1 << BIAS_OFFSET));

  // Convert to half2 and apply bias
  frag_b[1] = __hmul2(frag_b[1], bias_reg);
  frag_b[0] = __hmul2(frag_b[0], bias_reg);
}
```
**EN:** This section implements `id`, `__float2half2_rn`, `__hmul2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`__float2half2_rn`、`__hmul2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 389-413: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kFE2M1f.id(), true>(int q, nv_bfloat162* frag_b) {
  // Constants for FP4 (E2M1) and FP16 formats
  constexpr int FP4_EXPONENT = 2, BF16_EXPONENT = 8;
  constexpr int RIGHT_SHIFT = BF16_EXPONENT - FP4_EXPONENT;
  constexpr int MASK = 0x70007000;

  // Extract and shift FP4 values to FP16 format
  int Out1 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 4;
  int Out2 = (q & 0x80008000) | ((q & MASK) >> RIGHT_SHIFT);

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const nv_bfloat162*>(&Out1);
  frag_b[0] = *reinterpret_cast<const nv_bfloat162*>(&Out2);
}

template <>
__device__ inline void dequant<nv_bfloat162, sglang::kFE2M1f.id(), false>(int q, nv_bfloat162* frag_b) {
  dequant<nv_bfloat162, sglang::kFE2M1f.id(), true>(q, frag_b);

  // Constants for FP4 (E2M1) and BF16 formats
  constexpr int FP4_EXPONENT = 2, BF16_EXPONENT = 8;
```
**EN:** This section implements `id`, `FP4_EXPONENT`, `RIGHT_SHIFT`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`FP4_EXPONENT`、`RIGHT_SHIFT`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 414-439: Device helpers and synchronization
```cpp
  // Construct and apply exponent bias
  constexpr int BIAS_OFFSET = (1 << (BF16_EXPONENT - 1)) - (1 << (FP4_EXPONENT - 1));
  // Add 127 (float exponent bias) to BIAS_OFFSET and shift to float exponent
  // position
  constexpr uint32_t BIAS = (BIAS_OFFSET + 127) << 23;
  const nv_bfloat162 bias_reg = __float2bfloat162_rn(*reinterpret_cast<const float*>(&BIAS));

  // Convert to half2 and apply bias
  frag_b[1] = __hmul2(frag_b[1], bias_reg);
  frag_b[0] = __hmul2(frag_b[0], bias_reg);
}

template <typename scalar_t2>
__device__ inline void dequant_fp8_scales(int q, scalar_t2* frag_b);

template <>
__device__ inline void dequant_fp8_scales<half2>(int q, half2* frag_b) {
  int Out1 = (q & 0xFF00FF00) >> 1;
  ;
  q <<= 8;
  int Out2 = (q & 0xFF00FF00) >> 1;

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const half2*>(&Out1);
  frag_b[0] = *reinterpret_cast<const half2*>(&Out2);
};
```
**EN:** This section implements `__float2bfloat162_rn`, `__hmul2`, `dequant_fp8_scales`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__float2bfloat162_rn`、`__hmul2`、`dequant_fp8_scales`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 440-460: Device helpers and synchronization
```cpp

template <>
__device__ inline void dequant_fp8_scales<nv_bfloat162>(int q, nv_bfloat162* frag_b) {
  constexpr int FP8_EXPONENT = 4, BF16_EXPONENT = 8;
  constexpr int RIGHT_SHIFT = BF16_EXPONENT - FP8_EXPONENT;
  constexpr int MASK = 0x7F007F00;

  // Extract and shift FP8 values to BF16 format
  int Out1 = ((q & 0x80008000) >> 1) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 8;
  int Out2 = ((q & 0x80008000) >> 1) | ((q & MASK) >> RIGHT_SHIFT);

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const nv_bfloat162*>(&Out1);
  frag_b[0] = *reinterpret_cast<const nv_bfloat162*>(&Out2);
};

// New version with s_type_id parameter for marlin_moe_wna16_v2
template <typename scalar_t2, sglang::ScalarTypeId s_type_id>
__device__ inline void dequant_fp8_scales(int q, scalar_t2* frag_b);
```
**EN:** This section implements `dequant_fp8_scales`, `FP8_EXPONENT`, `RIGHT_SHIFT`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dequant_fp8_scales`、`FP8_EXPONENT`、`RIGHT_SHIFT`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 461-483: Device helpers and synchronization
```cpp
template <>
__device__ inline void dequant_fp8_scales<half2, sglang::kFE4M3fn.id()>(int q, half2* frag_b) {
  int Out1 = (q & 0xFF00FF00) >> 1;
  ;
  q <<= 8;
  int Out2 = (q & 0xFF00FF00) >> 1;

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const half2*>(&Out1);
  frag_b[0] = *reinterpret_cast<const half2*>(&Out2);
};

template <>
__device__ inline void dequant_fp8_scales<nv_bfloat162, sglang::kFE4M3fn.id()>(int q, nv_bfloat162* frag_b) {
  constexpr int FP8_EXPONENT = 4, BF16_EXPONENT = 8;
  constexpr int RIGHT_SHIFT = BF16_EXPONENT - FP8_EXPONENT;
  constexpr int MASK = 0x7F007F00;

  // Extract and shift FP8 values to BF16 format
  int Out1 = ((q & 0x80008000) >> 1) | ((q & MASK) >> RIGHT_SHIFT);
  q <<= 8;
  int Out2 = ((q & 0x80008000) >> 1) | ((q & MASK) >> RIGHT_SHIFT);
```
**EN:** This section implements `id`, `FP8_EXPONENT`, `RIGHT_SHIFT`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`、`FP8_EXPONENT`、`RIGHT_SHIFT`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 484-503: Device helpers and synchronization
```cpp
  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const nv_bfloat162*>(&Out1);
  frag_b[0] = *reinterpret_cast<const nv_bfloat162*>(&Out2);
}

template <>
__device__ inline void dequant_fp8_scales<nv_bfloat162, sglang::kFE8M0fnu.id()>(int q, nv_bfloat162* frag_b) {
  // In this conversion, 2 ** -127 in FP8E8M0 would become 0 in BF16,
  // but we assume that such a extreme value would not occur in real models.
  int Out1 = (q & 0xFF00FF00) >> 1;
  q <<= 7;
  int Out2 = q & 0x7F807F80;

  // Note: reverse indexing is intentional because weights are permuted
  frag_b[1] = *reinterpret_cast<const nv_bfloat162*>(&Out1);
  frag_b[0] = *reinterpret_cast<const nv_bfloat162*>(&Out2);
}

#endif
```
**EN:** This section implements `id`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`id`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 504-504: Local implementation details
```cpp
}  // namespace MARLIN_NAMESPACE_NAME
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `marlin_dtypes.cuh`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: gemm / marlin / dequant.h
