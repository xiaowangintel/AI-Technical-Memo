# cpu_attn_fp8.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/cpu_attn_fp8.hpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CPU attention kernels, dispatch paths, and ISA-specific specializations. / 实现 CPU 注意力内核、分派路径以及不同 ISA 的专用实现。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-10)
```cpp
// SPDX-License-Identifier: Apache-2.0
// SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#pragma once
#include <algorithm>
#include <cmath>
#include <cstdint>
#include <limits>
#include <type_traits>

#include "cpu/utils.hpp"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: fp8e4m3_to_float_scalar (lines 17-27)
```cpp
inline float fp8e4m3_to_float_scalar(uint8_t b, float scale) noexcept {
  // NaN encoding in E4M3
  if ((b & 0x7F) == 0x7F) return std::numeric_limits<float>::quiet_NaN();
  uint32_t b_u32 = static_cast<uint32_t>(b);
  uint32_t sign = (b_u32 & 0x80) << 24;
  uint32_t payload = (b_u32 & 0x7F) << 20;
  uint32_t bits = sign | payload;
  float b_f32_unscaled = *reinterpret_cast<const f32_alias_t*>(&bits);
  float b_f32_scaled = b_f32_unscaled * scale * 0x1p120f;
  return b_f32_scaled;
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: float_to_fp8e4m3_scalar (lines 29-44)
```cpp
inline uint8_t float_to_fp8e4m3_scalar(float v, float inv_scale) noexcept {
  v *= inv_scale;
  constexpr float fp8_max = 448.0f;
  v = std::max(-fp8_max, std::min(fp8_max, v));
  if (v == 0.0f) return 0;

  // Inverse mapping of fp8e4m3_to_float_scalar: shift the effective exponent
  // bias from fp32 (127) back to fp8 e4m3 (7), then pack sign|payload.
  float v_f32_unscaled = v * 0x1p-120f;
  uint32_t bits = *reinterpret_cast<const u32_alias_t*>(&v_f32_unscaled);
  uint8_t sign = static_cast<uint8_t>((bits >> 24) & 0x80);
  uint8_t payload = static_cast<uint8_t>((bits >> 20) & 0x7F);
  if (payload == 0) return sign;
  payload = std::min<uint8_t>(payload, 0x7E);  // keep 0x7F as NaN encoding
  return static_cast<uint8_t>(sign | payload);
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: reshape_and_cache_fp8_amx_impl (lines 53-114)
```cpp
template <typename scalar_t, uint8_t (*quant_fn)(float, float)>
inline void reshape_and_cache_fp8_amx_impl(
    const scalar_t* key_ptr, const scalar_t* value_ptr, uint8_t* key_cache_ptr,
    uint8_t* value_cache_ptr, const int64_t* slot_ptr, int64_t token_num,
    int64_t head_num, int64_t head_dim, int64_t block_size, int64_t k_stride0,
    int64_t k_stride1, int64_t v_stride0, int64_t v_stride1, int64_t kc_stride0,
    int64_t kc_stride1, int64_t vc_stride0, int64_t vc_stride1, float k_inv,
    float v_inv) {
  constexpr int64_t token_num_per_group = 16;  // AMX_TILE_ROW_NUM
  const int64_t halfword_num = head_dim / 2;   // 2 FP8 per uint16
  const int64_t halfword_num_per_group = token_num_per_group * halfword_num;
  constexpr int64_t head_elems_per_group = 16;
  constexpr int64_t token_num_per_sub_group = 2;  // = 4 / sizeof(BF16)
  const int64_t group_num = head_dim / head_elems_per_group;
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: fp8e5m2_to_float_scalar (lines 123-144)
```cpp
inline float fp8e5m2_to_float_scalar(uint8_t b, float scale) noexcept {
  const uint8_t exp_bits = (b >> 2) & 0x1F;
  const uint8_t mant_bits = b & 0x03;
  // NaN: exp=11111, mant!=00
  if (exp_bits == 0x1F && mant_bits != 0)
    return std::numeric_limits<float>::quiet_NaN();
  const uint32_t sign = static_cast<uint32_t>(b & 0x80) << 24;
  if (exp_bits == 0x1F)
    return sign ? -std::numeric_limits<float>::infinity()
                : std::numeric_limits<float>::infinity();
  if (exp_bits == 0) {  // subnormal: (-1)^s * 2^-14 * mant/4
    if (mant_bits == 0) return 0.0f;
    float v = mant_bits * 0x1p-16f;
    return (sign ? -v : v) * scale;
// ...
  float val = *reinterpret_cast<const f32_alias_t*>(&fp32_bits);
  return val * scale;
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: float_to_fp8e5m2_scalar (lines 146-164)
```cpp
inline uint8_t float_to_fp8e5m2_scalar(float v, float inv_scale) noexcept {
  v *= inv_scale;
  constexpr float fp8_e5m2_max = 57344.0f;
  v = std::max(-fp8_e5m2_max, std::min(fp8_e5m2_max, v));
  if (v == 0.0f) return 0;
  uint32_t bits = *reinterpret_cast<const u32_alias_t*>(&v);
  const uint8_t sign = static_cast<uint8_t>((bits >> 24) & 0x80);
  const int32_t exp_fp32 = static_cast<int32_t>((bits >> 23) & 0xFF) - 127;
  const uint8_t mant2 = static_cast<uint8_t>((bits >> 21) & 0x03);
  if (exp_fp32 < -14) {  // subnormal in E5M2
    const int shift = -14 - exp_fp32;
    if (shift + 21 >= 32)
      return sign;  // underflow: too small for E5M2 subnormal
    const uint32_t m = (0x800000u | (bits & 0x7FFFFFu)) >> (shift + 21);
// ...
  const uint8_t exp5 = static_cast<uint8_t>(exp_fp32 + 15);
  return sign | (exp5 << 2) | mant2;
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Compile-time setup: select_fp8_quant_fn (lines 170-170)
```cpp
constexpr auto select_fp8_quant_fn() {
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

### Function / Kernel: reshape_and_cache_fp8_vec_impl (lines 183-214)
```cpp
template <typename scalar_t, uint8_t (*quant_fn)(float, float)>
inline void reshape_and_cache_fp8_vec_impl(
    const scalar_t* key_ptr, const scalar_t* value_ptr, uint8_t* key_cache_ptr,
    uint8_t* value_cache_ptr, const int64_t* slot_ptr, int64_t token_num,
    int64_t head_num, int64_t head_dim, int64_t block_size, int64_t k_stride0,
    int64_t k_stride1, int64_t v_stride0, int64_t v_stride1, int64_t kc_stride0,
    int64_t kc_stride1, int64_t vc_stride0, int64_t vc_stride1, float k_inv,
    float v_inv) {
#pragma omp parallel for collapse(2) schedule(static)
  for (int64_t tok = 0; tok < token_num; ++tok) {
    for (int64_t h = 0; h < head_num; ++h) {
      const int64_t slot = slot_ptr[tok];
      if (slot < 0) continue;
      const int64_t block_idx = slot / block_size;
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Attention computation / 注意力计算
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cpu/utils.hpp`
- **External libraries / 外部库**: C++ standard library, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
