# mixed_input_utils.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/detail/collective/mixed_input_utils.hpp`

- **EN:** Internal helper utilities used by higher-level CUTLASS components.

- **CN:** 该头文件主要提供上层 CUTLASS 组件使用的内部辅助工具。文件级摘要：Internal helper utilities used by higher-level CUTLASS components.

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
```

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 31-31

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 33-34

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/numeric_conversion.h"
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, `cutlass/numeric_conversion.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h`, `cutlass/numeric_conversion.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 36-38

```cpp
#include "cute/util/type_traits.hpp"
#include "cute/arch/copy_sm90.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
```

**EN:** This block imports dependencies such as `cute/util/type_traits.hpp`, `cute/arch/copy_sm90.hpp`, `cute/numeric/arithmetic_tuple.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cute/util/type_traits.hpp`, `cute/arch/copy_sm90.hpp`, `cute/numeric/arithmetic_tuple.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 41-42

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 45-52

```cpp
// The universal converter
template <
  class SrcType,
  class DstType,
  class LayoutIn,
  class LayoutOut
>
struct LayoutAwareConvertImpl {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SrcType`, a `class` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SrcType` 这个 `class`，其成员会在后续代码中展开。

### Lines 53-75

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn, LayoutIn> const& src,
    cute::Tensor<EngineOut, LayoutOut>    & dst) {

    static_assert(cute::is_same_v<SrcType, typename EngineIn::value_type> &&
                  cute::is_same_v<DstType, typename EngineOut::value_type>);
    static_assert(cute::cosize_v<LayoutIn> == cute::cosize_v<LayoutOut>);
    constexpr int N = decltype(cute::max_common_vector(LayoutIn{}, LayoutOut{})){};
    using SrcArray = cutlass::Array<SrcType, N>;
    using DstArray = cutlass::Array<DstType, N>;
    using Converter = cutlass::NumericArrayConverter<DstType,
                                                     SrcType,
                                                     N,
                                                     cutlass::FloatRoundStyle::round_to_nearest>;
    auto&& src_vm = cute::recast<SrcArray>(src);
    auto&& dst_vm = cute::recast<DstArray>(dst);
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < src_vm.size(); ++i) {
      dst_vm(i) = Converter::convert(src_vm(i));
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 78-85

```cpp
// Specialization for INT4 -> BF16 with [02461357] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::int4b_t,
  cutlass::bfloat16_t,
  cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>,
  cute::Layout<_8>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 86-127

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                 cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                 cute::Layout<_8>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::int4b_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::bfloat16_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::int4b_t, 8>;
    using DstArray = cutlass::Array<cutlass::bfloat16_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (size_t ii = 0; ii < RegArray::kElements; ++ii) {
      r[ii] = src_reg >> (4 * (ii));
      static constexpr uint32_t xor_mask = 0x43084308;
      static constexpr uint32_t lo_mask  = 0x000F000F;
      static constexpr uint32_t immLut   = (0xf0 & 0xcc) ^ 0xaa;
#if defined(CUTLASS_ENABLE_SYCL)
      CUTE_INVALID_CONTROL_PATH("Unimplemented");
#else
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii])
          : "n"(lo_mask), "n"(xor_mask), "n"(immLut));
      static constexpr uint32_t lo_bias = xor_mask; // 0x43084308, {136, 136}
      {
        __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
        bf16x2_val = __hsub2(bf16x2_val,
                              reinterpret_cast<const __nv_bfloat162&>(lo_bias));
      }
#endif
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 130-137

```cpp
// Specialization for UINT4 -> BF16 with [02461357] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::uint4b_t,
  cutlass::bfloat16_t,
  cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>,
  cute::Layout<_8>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 138-179

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                 cute::Layout<_8>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::uint4b_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::bfloat16_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::uint4b_t, 8>;
    using DstArray = cutlass::Array<cutlass::bfloat16_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (size_t ii = 0; ii < RegArray::kElements; ++ii) {
      r[ii] = src_reg >> (4 * (ii));
      static constexpr uint32_t or_mask = 0x43004300;
      static constexpr uint32_t lo_mask = 0x000F000F;
      static constexpr uint32_t immLut  = (0xf0 & 0xcc) | 0xaa;
#if defined(CUTLASS_ENABLE_SYCL)
      CUTE_INVALID_CONTROL_PATH("Unimplemented");
#else
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii])
          : "n"(lo_mask), "n"(or_mask), "n"(immLut));
      static constexpr uint32_t lo_bias = or_mask; // 0x43004300, {128, 128}
      {
        __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
        bf16x2_val = __hsub2(bf16x2_val,
                             reinterpret_cast<const __nv_bfloat162&>(lo_bias));
      }
#endif
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 182-189

```cpp
// Specialization for INT4 -> FP16 with [02461357] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::int4b_t,
  cutlass::half_t,
  cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>,
  cute::Layout<_8>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 190-258

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                cute::Layout<_8>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::int4b_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::half_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::int4b_t, 8>;
    using DstArray = cutlass::Array<cutlass::half_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (int ii = 0; ii < RegArray::kElements; ii += 2) {
      auto src_ = src_reg >> (4 * (ii));
      r[ii + 0] = src_;
      r[ii + 1] = src_;
      static constexpr uint32_t lo_xor_mask = 0x64086408;
      static constexpr uint32_t hi_xor_mask = 0x64806480;
      static constexpr uint32_t lo_mask     = 0x000F000F;
      static constexpr uint32_t hi_mask     = 0x00F000F0;
      static constexpr uint32_t immLut      = (0xf0 & 0xcc) ^ 0xaa;
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii + 0])
          : "n"(lo_mask), "n"(lo_xor_mask), "n"(immLut));
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii + 1])
          : "n"(hi_mask), "n"(hi_xor_mask), "n"(immLut));
      static constexpr uint32_t lo_bias  = 0x64086408; // {1032, 1032}
      static constexpr uint32_t hi_bias  = 0xD480D480; // {-72, -72}
      static constexpr uint32_t hi_scale = 0x2C002C00; // {1/16, 1/16}
#if defined(CUTLASS_ENABLE_SYCL)
      {
        half2& fp16x2_val = reinterpret_cast<half2&>(r[ii + 0]);
        fp16x2_val = fp16x2_val - reinterpret_cast<const half2&>(lo_bias);
      }
      {
        half2& fp16x2_val = reinterpret_cast<half2&>(r[ii + 1]);
        fp16x2_val = sycl::fma(fp16x2_val,
                               reinterpret_cast<const half2&>(hi_scale),
                               reinterpret_cast<const half2&>(hi_bias));
      }
#else
      {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 0]);
        fp16x2_val = __hsub2(fp16x2_val,
                             reinterpret_cast<const half2&>(lo_bias));
      }
      {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 1]);
        fp16x2_val = __hfma2(fp16x2_val,
                              reinterpret_cast<const half2&>(hi_scale),
                              reinterpret_cast<const half2&>(hi_bias));
      }
#endif
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 261-268

```cpp
// Specialization for UINT4 -> FP16 with [02461357] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::uint4b_t,
  cutlass::half_t,
  cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>,
  cute::Layout<_8>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 269-336

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_4>, cute::Stride<_4,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                cute::Layout<_8>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::uint4b_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::half_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::uint4b_t, 8>;
    using DstArray = cutlass::Array<cutlass::half_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (int ii = 0; ii < RegArray::kElements; ii += 2) {
      auto src_ = src_reg >> (4 * (ii));
      r[ii + 0] = src_;
      r[ii + 1] = src_;
      static constexpr uint32_t or_mask = 0x64006400;
      static constexpr uint32_t lo_mask = 0x000F000F;
      static constexpr uint32_t hi_mask = 0x00F000F0;
      static constexpr uint32_t immLut  = (0xf0 & 0xcc) | 0xaa;
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii])
          : "n"(lo_mask), "n"(or_mask), "n"(immLut));
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii + 1])
          : "n"(hi_mask), "n"(or_mask), "n"(immLut));
      static constexpr uint32_t lo_bias  = or_mask;    // 0x64006400, {1024, 1024}
      static constexpr uint32_t hi_bias  = 0xD400D400; // {-64, -64}
      static constexpr uint32_t hi_scale = 0x2C002C00; // {1/16, 1/16}
#if defined(CUTLASS_ENABLE_SYCL)
      {
        half2& fp16x2_val = reinterpret_cast<half2&>(r[ii + 0]);
        fp16x2_val = fp16x2_val - reinterpret_cast<const half2&>(lo_bias);
      }
      {
        half2& fp16x2_val = reinterpret_cast<half2&>(r[ii + 1]);
        fp16x2_val = sycl::fma(fp16x2_val,
                               reinterpret_cast<const half2&>(hi_scale),
                               reinterpret_cast<const half2&>(hi_bias));
      }
#else
      {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 0]);
        fp16x2_val = __hsub2(fp16x2_val,
                             reinterpret_cast<const half2&>(lo_bias));
      }
      {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 1]);
        fp16x2_val = __hfma2(fp16x2_val,
                             reinterpret_cast<const half2&>(hi_scale),
                             reinterpret_cast<const half2&>(hi_bias));
      }
#endif
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 338-380

```cpp
/*
// Specialization for E5M2 -> FP16 with [3120] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::float_e5m2_t,
  cutlass::half_t,
  cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>,
  cute::Layout<_4>
> {
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                cute::Layout<_4>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::float_e5m2_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::half_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::float_e5m2_t, 8>;
    using DstArray = cutlass::Array<cutlass::half_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (int ii = 0; ii < RegArray::kElements; ++ii) {
      // in registers: a3, a1, a2, a0
      r[RegArray::kElements - ii - 1] = src_reg << (8 * (ii));

      static constexpr uint32_t and_mask = 0xFF00FF00;
      asm volatile(
          "{\n"
          "  and.b32 %0, %0, %1;\n"
          "}\n"
          : "+r"(r[ii])
          : "n"(and_mask));
    }
  }
};
*/
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 381-388

```cpp
// Specialization for INT8 -> BF16 with [3120] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::int8_t,
  cutlass::bfloat16_t,
  cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>,
  cute::Layout<_4>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 389-437

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                cute::Layout<_4>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::int8_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::bfloat16_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::int8_t, 8>;
    using DstArray = cutlass::Array<cutlass::bfloat16_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (int ii = 0; ii < RegArray::kElements; ++ii) {
      uint32_t tmp0, tmp1;
      r[ii] = src_reg >> (8 * (ii));
      static constexpr uint32_t or_mask    = 0x43004300;
      static constexpr uint32_t and_mask_0 = 0x007F007F;
      static constexpr uint32_t and_mask_1 = 0x00800080;
      static constexpr uint32_t immLut     = (0xf0 & 0xcc) | 0xaa;
#if defined(CUTLASS_ENABLE_SYCL)
      CUTE_INVALID_CONTROL_PATH("Unimplemented");
#else
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %1, %2, %3, %4;\n"
          "}\n"
          : "=r"(tmp0)
          : "r"(r[ii]), "n"(and_mask_0), "n"(or_mask), "n"(immLut));
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %1, %2, %3, %4;\n"
          "}\n"
          : "=r"(tmp1)
          : "r"(r[ii]), "n"(and_mask_1), "n"(or_mask), "n"(immLut));
      {
        __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
        bf16x2_val = __hsub2(reinterpret_cast<__nv_bfloat162 const&>(tmp0),
                             reinterpret_cast<__nv_bfloat162 const&>(tmp1));
      }
#endif
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 440-447

```cpp
// Specialization for INT8 -> FP16 with [3120] value order
template <>
struct LayoutAwareConvertImpl<
  cutlass::int8_t,
  cutlass::half_t,
  cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>,
  cute::Layout<_4>
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `LayoutAwareConvertImpl`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `LayoutAwareConvertImpl` 这个 `struct`，其成员会在后续代码中展开。

### Lines 448-490

```cpp
  template<class EngineIn, class EngineOut>
  CUTLASS_DEVICE
  static void convert(
    cute::Tensor<EngineIn,
                cute::Layout<cute::Shape<_2,_2>, cute::Stride<_2,_1>>
                > const& src,
    cute::Tensor<EngineOut,
                cute::Layout<_4>
                >& dst) {

    static_assert(cute::is_same_v<cutlass::int8_t, typename EngineIn::value_type> &&
                  cute::is_same_v<cutlass::half_t, typename EngineOut::value_type>);
    using SrcArray = cutlass::Array<cutlass::int8_t, 8>;
    using DstArray = cutlass::Array<cutlass::half_t, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, 4, sizeof(DstArray)>;

    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);
    CUTLASS_PRAGMA_UNROLL
    for (int ii = 0; ii < RegArray::kElements; ++ii) {
      r[ii] = src_reg >> (8 * (ii));
      static constexpr uint32_t xor_mask = 0x64806480;
      static constexpr uint32_t and_mask = 0x00FF00FF;
      static constexpr uint32_t immLut   = (0xf0 & 0xcc) ^ 0xaa;
      asm volatile(
          "{\n"
          "  lop3.b32 %0, %0, %1, %2, %3;\n"
          "}\n"
          : "+r"(r[ii])
          : "n"(and_mask), "n"(xor_mask), "n"(immLut));
      {
        static constexpr uint32_t bias = 0x64806480;
#if defined(CUTLASS_ENABLE_SYCL)
        half2& fp16x2_val = reinterpret_cast<half2&>(r[ii]);
        fp16x2_val = fp16x2_val - reinterpret_cast<half2 const&>(bias);
#else
        __half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii]);
        fp16x2_val = __hsub2(fp16x2_val,
                             reinterpret_cast<__half2 const&>(bias));
#endif
      }
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 493-505

```cpp
template <
  class EngineIn,
  class EngineOut,
  class LayoutIn,
  class LayoutOut
>
CUTLASS_DEVICE
void LayoutAwareConvert( // Accept mutable temporaries
  cute::Tensor<EngineIn, LayoutIn>   const& src,
  cute::Tensor<EngineOut, LayoutOut>     && dst) {

  LayoutAwareConvert(src, dst);
}
```

**EN:** This block begins the definition of `LayoutAwareConvert`, a `class` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `LayoutAwareConvert` 这个 `class`，其成员会在后续代码中展开。

### Lines 506-527

```cpp
template <
  class EngineIn,
  class EngineOut,
  class LayoutIn,
  class LayoutOut
>
CUTLASS_DEVICE
void LayoutAwareConvert(
  cute::Tensor<EngineIn, LayoutIn>   const& src,
  cute::Tensor<EngineOut, LayoutOut>      & dst) {

  using SrcType = typename EngineIn::value_type;
  using DstType = typename EngineOut::value_type;
  Tensor src_vm = coalesce(src);
  Tensor dst_vm = coalesce(dst);
  Layout src_layout = src_vm.layout();
  Layout dst_layout = dst_vm.layout();
  LayoutAwareConvertImpl<SrcType,
                         DstType,
                         decltype(src_layout),
                         decltype(dst_layout)>::convert(src_vm, dst_vm);
}
```

**EN:** This alias defines `SrcType` as `typename EngineIn::value_type`, shortening later template or member declarations.

**CN:** 这里把 `SrcType` 定义为 `typename EngineIn::value_type` 的别名，以简化后续模板或成员声明。

### Lines 532-533

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 535-535

```cpp
  namespace detail {
```

**EN:** This block opens the namespace scope `detail` for the declarations that follow.

**CN:** 该代码块打开了 `detail` 命名空间作用域，以容纳后续声明。

### Lines 536-536

```cpp
    enum class ConversionMode {
```

**EN:** This block begins the definition of `class`, a `enum` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `class` 这个 `enum`，其成员会在后续代码中展开。

### Lines 537-540

```cpp
      DirectConvert,              // A * B
      ConvertAndScale,            // (scale * A) * B
      ConvertAndScaleWithZero     // (scale * A + zeros) * B
    };
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 544-545

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass::gemm::collective::detail {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass::gemm::collective::detail` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass::gemm::collective::detail` 命名空间作用域，以容纳后续声明。

### Lines 548-553

```cpp
template <class PointerType>
static constexpr
CUTLASS_HOST_DEVICE
auto get_logical_ptr(PointerType const* ptr) {
  return cute::recast_ptr<PointerType const>(ptr);
}
```

**EN:** The function `get_logical_ptr` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `get_logical_ptr` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 554-568

```cpp
template<int Stages, class LayoutAtom, class TileShape, class Stride>
static constexpr
CUTLASS_HOST_DEVICE
auto get_smem_layout(LayoutAtom layout_atom, TileShape const& tile_shape, Stride const& stride) {
  if constexpr (not cute::is_layout<Stride>::value) {
    return tile_to_shape(
      layout_atom,
      append(tile_shape, Int<Stages>{}),
      cute::conditional_t< ::cutlass::gemm::detail::is_major<0,Stride>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{});
  }
  else {
    auto gmem_tile = composition(stride, tile_shape);
    return make_layout_like(append(gmem_tile, make_layout(Int<Stages>{}, 0)));
  }
}
```

**EN:** The function `gmem_tile` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `gmem_tile` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 569-579

```cpp
template<class Shape, class Stride>
static constexpr
CUTLASS_HOST_DEVICE
auto get_gmem_layout(Shape const& shape, Stride const& stride) {
  if constexpr (not cute::is_layout<Stride>::value) {
    return make_layout(shape, stride);
  }
  else {
    return stride;
  }
}
```

**EN:** The function `get_gmem_layout` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `get_gmem_layout` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 581-582

```cpp
template<class Collective>
struct MixedInputUtils {
```

**EN:** This block begins the definition of `MixedInputUtils`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `MixedInputUtils` 这个 `struct`，其成员会在后续代码中展开。

### Lines 584-584

```cpp
  using ConversionMode = cutlass::detail::ConversionMode;
```

**EN:** This alias defines `ConversionMode` as `cutlass::detail::ConversionMode`, shortening later template or member declarations.

**CN:** 这里把 `ConversionMode` 定义为 `cutlass::detail::ConversionMode` 的别名，以简化后续模板或成员声明。

### Lines 585-585

```cpp
  using KernelSchedule = typename Collective::KernelSchedule;
```

**EN:** This alias defines `KernelSchedule` as `typename Collective::KernelSchedule`, shortening later template or member declarations.

**CN:** 这里把 `KernelSchedule` 定义为 `typename Collective::KernelSchedule` 的别名，以简化后续模板或成员声明。

### Lines 586-586

```cpp
  using SmemLayoutA = typename Collective::SmemLayoutA;
```

**EN:** This alias defines `SmemLayoutA` as `typename Collective::SmemLayoutA`, shortening later template or member declarations.

**CN:** 这里把 `SmemLayoutA` 定义为 `typename Collective::SmemLayoutA` 的别名，以简化后续模板或成员声明。

### Lines 587-587

```cpp
  using SmemLayoutB = typename Collective::SmemLayoutB;
```

**EN:** This alias defines `SmemLayoutB` as `typename Collective::SmemLayoutB`, shortening later template or member declarations.

**CN:** 这里把 `SmemLayoutB` 定义为 `typename Collective::SmemLayoutB` 的别名，以简化后续模板或成员声明。

### Lines 588-588

```cpp
  using SmemLayoutScale = typename Collective::SmemLayoutScale;
```

**EN:** This alias defines `SmemLayoutScale` as `typename Collective::SmemLayoutScale`, shortening later template or member declarations.

**CN:** 这里把 `SmemLayoutScale` 定义为 `typename Collective::SmemLayoutScale` 的别名，以简化后续模板或成员声明。

### Lines 589-589

```cpp
  using SwappedElementA = typename Collective::SwappedElementA;
```

**EN:** This alias defines `SwappedElementA` as `typename Collective::SwappedElementA`, shortening later template or member declarations.

**CN:** 这里把 `SwappedElementA` 定义为 `typename Collective::SwappedElementA` 的别名，以简化后续模板或成员声明。

### Lines 590-590

```cpp
  using SwappedElementB = typename Collective::SwappedElementB;
```

**EN:** This alias defines `SwappedElementB` as `typename Collective::SwappedElementB`, shortening later template or member declarations.

**CN:** 这里把 `SwappedElementB` 定义为 `typename Collective::SwappedElementB` 的别名，以简化后续模板或成员声明。

### Lines 591-591

```cpp
  using RealSwappedElementA = typename Collective::RealSwappedElementA;
```

**EN:** This alias defines `RealSwappedElementA` as `typename Collective::RealSwappedElementA`, shortening later template or member declarations.

**CN:** 这里把 `RealSwappedElementA` 定义为 `typename Collective::RealSwappedElementA` 的别名，以简化后续模板或成员声明。

### Lines 592-592

```cpp
  using RealSwappedElementB = typename Collective::RealSwappedElementB;
```

**EN:** This alias defines `RealSwappedElementB` as `typename Collective::RealSwappedElementB`, shortening later template or member declarations.

**CN:** 这里把 `RealSwappedElementB` 定义为 `typename Collective::RealSwappedElementB` 的别名，以简化后续模板或成员声明。

### Lines 593-593

```cpp
  using ElementScale = typename Collective::ElementScale;
```

**EN:** This alias defines `ElementScale` as `typename Collective::ElementScale`, shortening later template or member declarations.

**CN:** 这里把 `ElementScale` 定义为 `typename Collective::ElementScale` 的别名，以简化后续模板或成员声明。

### Lines 594-594

```cpp
  using ElementZero = typename Collective::ElementZero;
```

**EN:** This alias defines `ElementZero` as `typename Collective::ElementZero`, shortening later template or member declarations.

**CN:** 这里把 `ElementZero` 定义为 `typename Collective::ElementZero` 的别名，以简化后续模板或成员声明。

### Lines 595-595

```cpp
  using SmemCopyAtomScale = typename Collective::SmemCopyAtomScale;
```

**EN:** This alias defines `SmemCopyAtomScale` as `typename Collective::SmemCopyAtomScale`, shortening later template or member declarations.

**CN:** 这里把 `SmemCopyAtomScale` 定义为 `typename Collective::SmemCopyAtomScale` 的别名，以简化后续模板或成员声明。

### Lines 596-596

```cpp
  static constexpr auto KernelConversionMode = Collective::KernelConversionMode;
```

**EN:** This declaration defines `KernelConversionMode` and assigns it the compile-time expression `Collective::KernelConversionMode`.

**CN:** 这个声明定义了 `KernelConversionMode`，并把它设为编译期表达式 `Collective::KernelConversionMode`。

### Lines 597-597

```cpp
  static constexpr auto ModeHasScales = Collective::ModeHasScales;
```

**EN:** This declaration defines `ModeHasScales` and assigns it the compile-time expression `Collective::ModeHasScales`.

**CN:** 这个声明定义了 `ModeHasScales`，并把它设为编译期表达式 `Collective::ModeHasScales`。

### Lines 598-598

```cpp
  static constexpr auto UseScaleLookupTable = Collective::UseScaleLookupTable;
```

**EN:** This declaration defines `UseScaleLookupTable` and assigns it the compile-time expression `Collective::UseScaleLookupTable`.

**CN:** 这个声明定义了 `UseScaleLookupTable`，并把它设为编译期表达式 `Collective::UseScaleLookupTable`。

### Lines 601-612

```cpp
  static constexpr auto
  elements_per_smem_scale() {
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return 0;
    }
    else if constexpr (ModeHasScales) {
      return cute::cosize_v<SmemLayoutScale>;
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in scale smem allocation.");
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 614-626

```cpp
  static constexpr auto
  elements_per_smem_zero() {
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert ||
                  KernelConversionMode == ConversionMode::ConvertAndScale ) {
      return 0;
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      return cute::cosize_v<SmemLayoutScale>;
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in scale smem allocation.");
    }
  }
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 628-632

```cpp
  // These methods use some the public members of the class. For that reason, we define them after the public section.
  static constexpr uint32_t
  compute_tma_transaction_bytes_mk() {
    return cutlass::bits_to_bytes(size<0>(SmemLayoutA{}) * size<1>(SmemLayoutA{}) * static_cast<uint32_t>(cute::sizeof_bits_v<SwappedElementA>));
  }
```

**EN:** The preceding comment documents this block. The function `compute_tma_transaction_bytes_mk` implements a concrete operation in this abstraction. It performs explicit type conversion to keep the representation precise. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`compute_tma_transaction_bytes_mk` 函数实现了该抽象中的一个具体操作。它通过显式类型转换来保持表示精确。函数体主要根据当前状态或输入计算并返回结果。

### Lines 634-637

```cpp
  static constexpr uint32_t
  compute_tma_transaction_bytes_nk() {
    return cutlass::bits_to_bytes(size<0>(SmemLayoutB{}) * size<1>(SmemLayoutB{}) * static_cast<uint32_t>(cute::sizeof_bits_v<SwappedElementB>));
  }
```

**EN:** The function `compute_tma_transaction_bytes_nk` implements a concrete operation in this abstraction. It performs explicit type conversion to keep the representation precise. The body mainly computes and returns a value from the current state or inputs.

**CN:** `compute_tma_transaction_bytes_nk` 函数实现了该抽象中的一个具体操作。它通过显式类型转换来保持表示精确。函数体主要根据当前状态或输入计算并返回结果。

### Lines 639-663

```cpp
  static constexpr uint32_t
  compute_tma_transaction_bytes_extra() {
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return 0;
    }
    else if constexpr (ModeHasScales) {
      constexpr uint32_t scale_tx_bytes = cutlass::bits_to_bytes(size<0>(SmemLayoutScale{}) * size<1>(SmemLayoutScale{}) * static_cast<uint32_t>(cute::sizeof_bits_v<ElementScale>));
      static_assert(scale_tx_bytes % 128 == 0, "Each scale stage must be 128B aligned."); // required by TMA
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return scale_tx_bytes;
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        // Scale and zero share smem layout
        constexpr uint32_t zero_tx_bytes = cutlass::bits_to_bytes(size<0>(SmemLayoutScale{}) * size<1>(SmemLayoutScale{}) * static_cast<uint32_t>(cute::sizeof_bits_v<ElementZero>));
        static_assert(zero_tx_bytes % 128 == 0, "Each zero stage must be 128B aligned."); // required by TMA
        return scale_tx_bytes + zero_tx_bytes;
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in tma transaction bytes computation.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in tma transaction bytes computation.");
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 665-689

```cpp
  static constexpr uint32_t
  compute_tma_transaction_bytes_extra_transform() {
    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return 0;
    }
    else if constexpr (ModeHasScales) {
      constexpr uint32_t scale_tx_bytes = cutlass::bits_to_bytes(size<0>(filter_zeros(SmemLayoutScale{})) * size<1>(filter_zeros(SmemLayoutScale{})) * static_cast<uint32_t>(cute::sizeof_bits_v<ElementScale>));
      static_assert(scale_tx_bytes % 128 == 0, "Each scale stage must be 128B aligned."); // required by TMA
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return scale_tx_bytes;
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        // Scale and zero share smem layout
        constexpr uint32_t zero_tx_bytes = cutlass::bits_to_bytes(size<0>(filter_zeros(SmemLayoutScale{})) * size<1>(filter_zeros(SmemLayoutScale{})) * static_cast<uint32_t>(cute::sizeof_bits_v<ElementZero>));
        static_assert(zero_tx_bytes % 128 == 0, "Each zero stage must be 128B aligned."); // required by TMA
        return scale_tx_bytes + zero_tx_bytes;
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in tma transaction bytes computation.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Type not handled in tma transaction bytes computation.");
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 691-734

```cpp
  /// Utilities to copy A and extra inputs from smem to RF
  template <class SmemTiledCopyA,
            class TensorASmemView,
            class TensorACopyView,
            class... Ts,
            class... Us
            >
  CUTLASS_DEVICE
  static void copy_tensors_MK(
    SmemTiledCopyA const& smem_tiled_copy_A,
    TensorASmemView const& tCsA,
    TensorACopyView& tCrA_copy_view,
    cute::tuple<Ts...> const& partitioned_mma_extra_info,
    cute::tuple<Us...> const& tiled_copy_and_views,
    int k_block,
    int read_stage) {

    copy(smem_tiled_copy_A, tCsA(_,_,k_block,read_stage), tCrA_copy_view(_,_,k_block));

    if (k_block == 0) {
      // We are starting a new k-tile so copy the scale
      if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
        // nothing to do
      }
      else if constexpr (ModeHasScales) {
        auto smem_tiled_copy_S = cute::get<0>(tiled_copy_and_views);
        auto tCrS_copy_view    = cute::get<1>(tiled_copy_and_views);
        auto tCsS              = cute::get<0>(partitioned_mma_extra_info);
        copy(smem_tiled_copy_S, tCsS(_,_,k_block,read_stage), tCrS_copy_view(_,_,k_block));
        if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
          // Nothing extra to do
        } else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
          auto tCsZ              = cute::get<2>(partitioned_mma_extra_info);
          auto tCrZ_copy_view    = cute::get<2>(tiled_copy_and_views);
          copy(smem_tiled_copy_S, tCsZ(_,_,k_block,read_stage), tCrZ_copy_view(_,_,k_block));
        } else {
          static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
        }
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
      }
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 736-771

```cpp
  /// (Designed for separate transform pipeline in Blackwell)
  /// Utilities to copy extra inputs from smem to RF
  template <class... Ts>
  CUTLASS_DEVICE
  static void copy_scale_zeros_for_transform(
    cute::tuple<Ts...> & partitioned_transform_extra_info,
    int load2transform_consumer_index) {

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      // nothing to do
    }
    else if constexpr (ModeHasScales) {
      auto smem_tiled_copy_S = cute::get<0>(partitioned_transform_extra_info);
      auto&& scales          = cute::get<1>(partitioned_transform_extra_info);
      using ScaleType        = decltype(scales);
      auto tSrS              = make_tensor(scales.data(), scales.layout());
      auto tSsS              = cute::get<2>(partitioned_transform_extra_info);
      copy(smem_tiled_copy_S, tSsS(_,_,_,_,load2transform_consumer_index), tSrS);

      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        // Nothing extra to do
      } else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        auto&& zeros           = cute::get<3>(partitioned_transform_extra_info);
        using ZeroType         = decltype(zeros);
        auto tZrZ              = make_tensor(zeros.data(), zeros.layout());
        auto tZsZ              = cute::get<4>(partitioned_transform_extra_info);
        copy(smem_tiled_copy_S, tZsZ(_,_,_,_,load2transform_consumer_index), tZrZ);

      } else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 773-777

```cpp
  // Helper functions to select packing for conversion
  template <class SrcType,
            class DstType,
            int Cosize>
  struct select_packing { // Naive packing policy
```

**EN:** The preceding comment documents this block. This block begins the definition of `DstType`, a `class` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `DstType` 这个 `class`，其成员会在后续代码中展开。

### Lines 778-780

```cpp
    static constexpr auto value() {
      return Int<cute::gcd(Cosize, 32 / cute::min(sizeof_bits_v<SrcType>, sizeof_bits_v<DstType>))>{};
    }
```

**EN:** The function `value` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `value` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 783-798

```cpp
  // The core converter uses a lookup table to converts i4 -> 8 bit value.
  template <class EngineIn,
            class LayoutIn,
            class EngineOut,
            class LayoutOut,
            class EngineScale,
            class LayoutScale>
  CUTLASS_DEVICE
  static void lookup_table_convert( // Accept mutable temporaries
    Tensor<EngineIn, LayoutIn>       const& src,
    Tensor<EngineOut, LayoutOut>         && dst,
    Tensor<EngineScale, LayoutScale> const& scales_neg,
    Tensor<EngineScale, LayoutScale> const& scales_pos) {

    lookup_table_convert(src, dst, scales_neg, scales_pos);
  }
```

**EN:** The preceding comment documents this block. This block begins the definition of `lookup_table_convert`, a `class` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `lookup_table_convert` 这个 `class`，其成员会在后续代码中展开。

### Lines 799-855

```cpp
  template <class EngineIn,
            class LayoutIn,
            class EngineOut,
            class LayoutOut,
            class EngineScale,
            class LayoutScale>
  CUTLASS_DEVICE
  static void lookup_table_convert(
    Tensor<EngineIn, LayoutIn>       const& src,
    Tensor<EngineOut, LayoutOut>          & dst,
    Tensor<EngineScale, LayoutScale> const& scales_neg,
    Tensor<EngineScale, LayoutScale> const& scales_pos) {

    constexpr int N = cute::cosize(LayoutIn{});
    static_assert(N == 4 || N == 8);
    static_assert(cosize(LayoutScale{}) <= N / 4,
                  "at least 4 consecutive weights must share the same scale.");
    using SrcArray = cutlass::Array<cutlass::int4b_t, 8>;
    using DstArray = cutlass::Array<RealSwappedElementB, 8>;
    using RegArray = cutlass::AlignedArray<uint32_t, N / 4, sizeof(DstArray)>;

    // View the input as reg
    auto&& src_reg = cute::recast<uint32_t>(src)(0);
    auto&& r       = cute::recast<RegArray>(dst)(0);

    // Determines if to get from the signed or unsigned candidates
    static constexpr uint32_t immLut = (0xf0 & 0xcc) | 0xaa;
    uint32_t sign; // ((reg & 0x88888888) | 0x64206420) >> 1
    asm volatile(
      "{\n"
      "  lop3.b32 %0, %1, %2, %3, %4;\n" \
      "}\n"
      : "=r"(sign)
      : "r"(src_reg), "n"(0x88888888), "n"(0x64206420), "n"(immLut)
    );
    sign = sign >> 1;

    // Ignore sign bit when indexing into LUT
    uint32_t lut_idx = src_reg & 0x77777777;
    Tensor scales_neg_ = cute::filter(scales_neg);
    Tensor scales_pos_ = cute::filter(scales_pos);
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < N / 4; ++i, lut_idx >>=16, sign >>=16) {
      auto&& scale_neg_ = reinterpret_cast<cutlass::Array<uint32_t, 2> const&>(scales_neg_(i));
      auto&& scale_pos_ = reinterpret_cast<cutlass::Array<uint32_t, 2> const&>(scales_pos_(i));
      asm volatile(
        "{\n"
        "  .reg .b32 pos, neg                    ;\n" \
        "  prmt .b32 neg, %3, %4, %1             ;\n" \
        "  prmt .b32 pos, %5, %6, %1             ;\n" \
        "  prmt .b32 %0, pos, neg, %2            ;\n" \
        "}\n"
        : "=r"(r[i])
        : "r"(lut_idx), "r"(sign), "r"(scale_neg_[0]), "r"(scale_neg_[1]), "r"(scale_pos_[0]), "r"(scale_pos_[1])
      );
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 857-862

```cpp
  /// Utilities to dequantize A.
  template <class Layout>
  CUTLASS_DEVICE
  static void static_check_scale(Layout const& tensor) {
    static_assert(shape<0>(Layout{}) >= 4 && stride<0>(Layout{}) == 0, "At least 4 adjacent weights in a thread must share the same scale.");
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 863-868

```cpp
  template <class Engine,
            class Layout>
  CUTLASS_DEVICE
  static void static_check_scale(Tensor<Engine, Layout> const& tensor) {
    static_check_scale(flatten(Layout{}));
  }
```

**EN:** This block declares `static_check_scale` as a lightweight `class`, often used as a tag or thin wrapper.

**CN:** 该代码块将 `static_check_scale` 声明为一个轻量级 `class`，通常用作标签类型或薄封装。

### Lines 869-1010

```cpp
  template <class EngineIn,
            class EngineOut,
            class LayoutIn,
            class LayoutOut,
            class... Ts>
  CUTLASS_DEVICE
  static void dequantize_A_kblock(
    Tensor<EngineIn, LayoutIn> const& tCrA_load,
    Tensor<EngineOut, LayoutOut>& tCrA_mma,
    cute::tuple<Ts...>& partitioned_extra_info,
    int const k_block) {

    static_assert(is_rmem<EngineIn>::value, "Input tensor for A conversion must come from registers");
    static_assert(is_rmem<EngineOut>::value, "Output tensor for A conversion must come from registers");
    static_assert(cosize_v<LayoutIn> == cosize_v<LayoutOut>);
    static_assert(size_v<LayoutIn> == cosize_v<LayoutIn>);
    static_assert(size_v<LayoutOut> == cosize_v<LayoutOut>);
    using SrcType = typename EngineIn::value_type;
    using DstType = typename EngineOut::value_type;

    Tensor src = tCrA_load(_, _, k_block);
    Tensor dst = tCrA_mma(_, _, k_block);

    CUTE_STATIC_ASSERT_V(size(src(_, 0)) == cosize(src(_, 0).layout()),
                         "The first mode of tensor src must be contiguous in memory");
    // try to make the size of the first mode equal to 32bit
    int constexpr NumValPerSrcReg = cute::min(decltype(size(src(_, 0)))::value,
                                              ceil_div(32, sizeof_bits_v<SrcType>));
    Tensor src_vm = cute::group_modes<1,-1>(cute::zipped_divide(src, Int<NumValPerSrcReg>{}));
    Tensor dst_vm = cute::group_modes<1,-1>(cute::zipped_divide(dst, Int<NumValPerSrcReg>{}));

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size<1>(dst_vm); ++i) {
        LayoutAwareConvert(src_vm(_, i), dst_vm(_, i));
      }
    }
    else if constexpr (UseScaleLookupTable) {
      constexpr int num_elements = decltype(size(src))::value;
      static_assert(is_same_v<RealSwappedElementA, cutlass::int4b_t>, "Lookup table only supports int4 being the quant type now.");
      static_assert(sizeof_bits_v<ElementScale> == 64, "Lookup table only supports 8 8bit scale values now.");
      static_assert(num_elements % 4 == 0 && num_elements >= 4, "Lookup table requires a vector size of 4x when converting.");

      Tensor tCrS_neg = cute::get<1>(partitioned_extra_info);
      auto&& tCrS_pos = cute::get<2>(partitioned_extra_info); // modification to its value is needed
      Tensor scales_neg = tCrS_neg(_, _, k_block);
      Tensor scales_pos = tCrS_pos(_, _, k_block);
      CUTE_STATIC_ASSERT_V(cute::size(src) == cute::size(scales_neg));

      static_check_scale(scales_neg);
      static_check_scale(scales_pos);
      Tensor scales_neg_vm = cute::group_modes<1,-1>(cute::zipped_divide(scales_neg, Int<NumValPerSrcReg>{}));
      Tensor scales_pos_vm = cute::group_modes<1,-1>(cute::zipped_divide(scales_pos, Int<NumValPerSrcReg>{}));

      if (k_block == 0) {
        Tensor scales_neg_vm_ = filter(scales_neg_vm);
        Tensor scales_pos_vm_ = filter(scales_pos_vm);
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(scales_neg_vm_.layout()); ++i)
        {
          auto&& scale_neg_ = reinterpret_cast<cutlass::Array<uint32_t, 2> const&>(scales_neg_vm_(i));
          auto&& scale_pos_ = reinterpret_cast<cutlass::Array<uint32_t, 2>      &>(scales_pos_vm_(i));
          constexpr uint32_t immLut = (0xf0 & 0xcc) ^ 0xaa;
          asm volatile(
              "{\n"
              "  lop3 .b32 %0, %2, %4, %5, %6;\n" \
              "  xor  .b32 %1, %3, %5;        \n" \
              "}\n"
              : "=r"(scale_pos_[0]), "=r"(scale_pos_[1])
              : "r"(scale_neg_[0]), "r"(scale_neg_[1]), "n"(0xFFFFFF00), "n"(0x80808080), "n"(immLut)
            );
        }
      }
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size<1>(dst_vm); ++i) {
        lookup_table_convert(src_vm(_, i), dst_vm(_, i), scales_neg_vm(_, i), scales_pos_vm(_, i));
      }
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
      Tensor scales = cute::get<1>(partitioned_extra_info)(_, _, k_block);
      CUTE_STATIC_ASSERT_V(size(src) == size(scales));
      Tensor scales_vm = cute::group_modes<1,-1>(cute::zipped_divide(scales, Int<NumValPerSrcReg>{}));

      if constexpr (is_same_v<DstType, ElementScale>) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size<1>(dst_vm); ++i) {
          LayoutAwareConvert(src_vm(_, i), dst_vm(_, i));
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < size<0>(dst_vm); ++j) {
            dst_vm(j, i) *= scales_vm(j, i);
          }
        }
      }
      else {
        auto stage = make_tensor_like<ElementScale>(src_vm(_, 0));
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size<1>(dst_vm); ++i) {
          LayoutAwareConvert(src_vm(_, i), stage);
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < size<0>(dst_vm); ++j) {
            stage(j) *= scales_vm(j, i);
          }
          LayoutAwareConvert(stage, dst_vm(_, i));
        }
      }
    }
    else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
      static_assert(is_same_v<ElementScale, ElementZero>, "ElementScale and ElementZero must be the same.");
      Tensor scales = cute::get<1>(partitioned_extra_info)(_, _, k_block);
      Tensor zeros  = cute::get<3>(partitioned_extra_info)(_, _, k_block);
      CUTE_STATIC_ASSERT_V(size(src) == size(scales));
      CUTE_STATIC_ASSERT_V(size(src) == size(zeros));
      Tensor scales_vm = cute::group_modes<1,-1>(cute::zipped_divide(scales, Int<NumValPerSrcReg>{}));
      Tensor zeros_vm = cute::group_modes<1,-1>(cute::zipped_divide(zeros, Int<NumValPerSrcReg>{}));

      if constexpr (is_same_v<DstType, ElementScale>) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size<1>(dst_vm); ++i) {
          LayoutAwareConvert(src_vm(_, i), dst_vm(_, i));
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < size<0>(dst_vm); ++j) {
            dst_vm(j, i) = dst_vm(j, i) * scales_vm(j, i) + zeros_vm(j, i);
          }
        }
      }
      else {
        auto stage = make_tensor_like<ElementScale>(src_vm(_, 0));
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size<1>(dst_vm); ++i) {
          LayoutAwareConvert(src_vm(_, i), stage);
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < size<0>(dst_vm); ++j) {
            stage(j) = stage(j) * scales_vm(j, i) + zeros_vm(j, i);
          }
          LayoutAwareConvert(stage, dst_vm(_, i));
        }
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "No A data is loaded.");
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 1012-1293

```cpp
  /// (Designed for separate transform pipeline in Blackwell)
  /// Utilities to dequantize A.
  template <class EngineIn,
            class EngineOut,
            class LayoutIn,
            class LayoutOut,
            class... Ts>
  CUTLASS_DEVICE
  static void dequantize_A_kblock_for_transform(
    Tensor<EngineIn, LayoutIn> const& tArA,
    Tensor<EngineOut, LayoutOut>& tArACompute,
    cute::tuple<Ts...> const& partitioned_extra_info,
    int const k_block) {
    #if !defined(CUTLASS_ENABLE_SYCL)
      static_assert(is_rmem<EngineIn>::value, "Input tensor for A conversion must come from registers");
      static_assert(is_rmem<EngineOut>::value, "Output tensor for A conversion must come from registers");
      static_assert(cosize_v<LayoutIn> == cosize_v<LayoutOut>);
      static_assert(size_v<LayoutIn> == cosize_v<LayoutIn>);
      static_assert(size_v<LayoutOut> == cosize_v<LayoutOut>);
      using SrcType = typename EngineIn::value_type;
      using DstType = typename EngineOut::value_type;

      auto src = tArA(_, _, _, k_block);
      auto dst = tArACompute(_, _, _, k_block);
      constexpr int num_elements = decltype(size(src))::value;

      constexpr int pack = decltype(select_packing<SrcType, DstType, num_elements>::value())::value;
      using Converter = cutlass::NumericArrayConverter<DstType, SrcType, pack, cutlass::FloatRoundStyle::round_to_nearest>;
      using SrcArray = cutlass::Array<SrcType, pack>;
      using DstArray = cutlass::Array<DstType, pack>;
      constexpr int DstElementsPerReg = 32 / sizeof_bits_v<DstType>;
      using RegArray = cutlass::AlignedArray<uint32_t, pack / DstElementsPerReg, sizeof(DstArray)>;

      auto src_arr = recast<SrcArray>(src);
      auto dst_arr = recast<DstArray>(dst);

      Tensor src_vm = cute::group_modes<1,-1>(cute::zipped_divide(src, pack));
      Tensor dst_vm = cute::group_modes<1,-1>(cute::zipped_divide(dst, pack));

      cute::transform(src_arr, dst_arr, Converter::convert);
      
      if constexpr (ModeHasScales) {

        auto const& scales = cute::get<1>(partitioned_extra_info)(_,_,_,k_block);

        CUTE_STATIC_ASSERT_V(size(src) == size(scales));

        if constexpr (is_same_v<DstType, ElementScale>) {

          using ScaleArray = cutlass::Array<ElementScale, pack>;
          auto scale_arr = recast<ScaleArray>(filter_zeros(scales));

        if constexpr (is_same_v<DstType, cutlass::bfloat16_t>){
          Tensor scales_vm = cute::group_modes<1,-1>(cute::zipped_divide(scales, pack));

            for (int i = 0; i < size<1>(dst_vm); ++i){
              auto&& r       = cute::recast<RegArray>(dst_vm(_,i))(0);
              auto&& scale_reg = cute::recast<RegArray>(scales_vm(_,i))(0);
              CUTLASS_PRAGMA_UNROLL
              for (size_t ii = 0; ii < RegArray::kElements; ++ii) {
                __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
                bf16x2_val = __hmul2(bf16x2_val,
                                    reinterpret_cast<const __nv_bfloat162&>(scale_reg[ii]));
              }
            }
          }
          else{
            cute::transform(dst_arr, scale_arr, dst_arr, cute::multiplies{});
          }
        }
        if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
          // Do Nothing
        }
        else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
          static_assert(is_same_v<ElementScale, ElementZero>, "ElementScale and ElementZero must be the same.");

          auto const& zeros = cute::get<3>(partitioned_extra_info)(_,_,_,k_block);
          CUTE_STATIC_ASSERT_V(size(src) == size(zeros));

          if constexpr (is_same_v<DstType, ElementZero>) {
            using ZeroArray = cutlass::Array<ElementZero, pack>;
            auto zero_arr = recast<ZeroArray>(filter_zeros(zeros));

          if constexpr (is_same_v<DstType, cutlass::bfloat16_t>) {
            Tensor zeros_vm = cute::group_modes<1,-1>(cute::zipped_divide(zeros, pack));

            for (int i = 0; i < size<1>(dst_vm); ++i){
              auto&& r       = cute::recast<RegArray>(dst_vm(_,i))(0);
              auto&& zero_reg = cute::recast<RegArray>(zeros_vm(_,i))(0);
              CUTLASS_PRAGMA_UNROLL
              for (size_t ii = 0; ii < RegArray::kElements; ++ii) {
                __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
                bf16x2_val = __hadd2(bf16x2_val,
                                    reinterpret_cast<const __nv_bfloat162&>(zero_reg[ii]));
              }
            }
          }
          else{
            cute::transform(dst_arr, zero_arr, dst_arr, cute::plus{});
          }
        }
      }
      else {
          static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for input partitioning.");
      }
    #else
      CUTE_INVALID_CONTROL_PATH("Unimplemented");
    #endif // !defined(CUTLASS_ENABLE_SYCL)
  }


  /// Utilities for any additional inputs inside of the TMA load
  template <
    class Params,
    class TensorStorage,
    class... Ts
  >
  CUTLASS_DEVICE
  static auto partition_extra_tma_inputs(
    Params const& mainloop_params,
    cute::tuple<Ts...> const& load_inputs,
    TensorStorage& shared_tensors,
    uint2 const& cluster_local_block_id,
    int const m_coord,
    int const l_coord) {

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      return cute::make_tuple();
    }
    else if constexpr (ModeHasScales) {
      Tensor sS  = make_tensor(make_smem_ptr(shared_tensors.smem_scale.begin()), SmemLayoutScale{}); // (BLK_M,BLK_K,PIPE)
      Tensor gS_mkl = get<2>(load_inputs);
      auto block_tma_s = mainloop_params.tma_load_scale.get_slice(cluster_local_block_id.y);
      Tensor gS = gS_mkl(_,_,m_coord,_,l_coord);                                                  // (BLK_M,BLK_K,k)

      Tensor tSgS = block_tma_s.partition_S(gS);
      Tensor tSsS = block_tma_s.partition_D(sS);                                              // (TMA,TMA_M,TMA_K,PIPE)
      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(tSgS, tSsS);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor sZ  = make_tensor(make_smem_ptr(shared_tensors.smem_zero.begin()), SmemLayoutScale{}); // (BLK_M,BLK_K,PIPE)
        Tensor gZ_mkl = get<3>(load_inputs);
        auto block_tma_z = mainloop_params.tma_load_zero.get_slice(cluster_local_block_id.y);
        Tensor gZ = gZ_mkl(_,_,m_coord,_,l_coord);                                            // (BLK_M,BLK_K,k)

        Tensor tZgZ = block_tma_z.partition_S(gZ);
        Tensor tZsZ = block_tma_z.partition_D(sZ);                                            // (TMA,TMA_M,TMA_K,PIPE)
        return cute::make_tuple(tSgS, tSsS, tZgZ, tZsZ);
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for input partitioning.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled for input partitioning.");
    }
  }

  /// Utilities for partitioning extra inputs for loading from smem in the mainloop.
  template <
    class ThreadMma,
    class TensorStorage
  >
  CUTLASS_DEVICE
  static auto partition_extra_mma_info(
    ThreadMma const& mma_thread_slice,
    TensorStorage& shared_tensors) {

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      // nothing to do
      return cute::make_tuple();
    }
    else if constexpr (UseScaleLookupTable) {
      Tensor sS = make_tensor(make_smem_ptr(shared_tensors.smem_scale.begin()), SmemLayoutScale{});// (BLK_M,BLK_SCALE_K,PIPE)
      Tensor tCsS = mma_thread_slice.partition_A(sS);
      Tensor tCrS_neg = make_tensor<ElementScale>(mma_thread_slice.partition_fragment_A(sS(_,_,Int<0>{})).layout());
      Tensor tCrS_pos = make_tensor<ElementScale>(mma_thread_slice.partition_fragment_A(sS(_,_,Int<0>{})).layout());

      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(tCsS, tCrS_neg, tCrS_pos);
      }
    }
    else if constexpr (ModeHasScales) {
      Tensor sS = make_tensor(make_smem_ptr(shared_tensors.smem_scale.begin()), SmemLayoutScale{});// (BLK_M,BLK_SCALE_K,PIPE)
      Tensor tCsS = mma_thread_slice.partition_A(sS);
      Tensor tCrS = make_tensor<ElementScale>(mma_thread_slice.partition_fragment_A(sS(_,_,Int<0>{})).layout());

      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(tCsS, tCrS);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor sZ = make_tensor(make_smem_ptr(shared_tensors.smem_zero.begin()), SmemLayoutScale{});// (BLK_M,BLK_SCALE_K,PIPE)
        Tensor tCsZ = mma_thread_slice.partition_A(sZ);
        Tensor tCrZ = make_tensor<ElementZero>(mma_thread_slice.partition_fragment_A(sZ(_,_,Int<0>{})).layout());
        return cute::make_tuple(tCsS, tCrS, tCsZ, tCrZ);
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
    }
  }

  template <
    class TiledMma,
    class TiledCopy,
    class TensorStorage
  >
  CUTLASS_DEVICE
  static auto partition_extra_transform_info(
    TiledMma const& tiled_mma,
    TiledCopy const& smem_tiled_copy_S,
    TensorStorage& shared_storage) {

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      // nothing to do
      return cute::make_tuple();
    }
    else if constexpr (ModeHasScales) {
      ThrMMA cta_mma = TiledMma{}.get_slice(BlockIdxX() % size(typename TiledMma::AtomThrID{}));
      auto smem_thr_copy_S = smem_tiled_copy_S.get_slice(ThreadIdxX() % 128);

      Tensor sS = make_tensor(make_smem_ptr(shared_storage.input.smem_scale.begin()), SmemLayoutScale{}); // (BLK_M,BLK_SCALE_K,PIPE)
      Tensor tCsS = cta_mma.partition_A(sS);
      Tensor tSsS = smem_thr_copy_S.partition_S(tCsS);
      Tensor tSrS = make_tensor<ElementScale>(tSsS(_,_,_,_,0).shape());

      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(smem_tiled_copy_S, tSrS, tSsS);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor sZ = make_tensor(make_smem_ptr(shared_storage.input.smem_zero.begin()), SmemLayoutScale{});// (BLK_M,BLK_SCALE_K,PIPE)
        Tensor tCsZ = cta_mma.partition_A(sZ);
        Tensor tZsZ = smem_thr_copy_S.partition_S(tCsZ);
        Tensor tZrZ = make_tensor<ElementZero>(tZsZ(_,_,_,_,0).shape());
        return cute::make_tuple(smem_tiled_copy_S, tSrS, tSsS, tZrZ, tZsZ);
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
    }
  }

  /// Returns the tiled copy and copy views for the extra inputs.
  template <class TiledMma, class... Ts>
  CUTLASS_DEVICE
  static auto retile_extra_mma_info(
    TiledMma const& tiled_mma,
    cute::tuple<Ts...>& partitioned_extra_info,
    int const warp_group_thread_idx) {

    if constexpr (KernelConversionMode == ConversionMode::DirectConvert) {
      // nothing to do
      return cute::make_tuple();
    }
    else if constexpr (ModeHasScales) {
      auto smem_tiled_copy_S = make_tiled_copy_A(SmemCopyAtomScale{}, tiled_mma);
      auto smem_thr_copy_S   = smem_tiled_copy_S.get_thread_slice(warp_group_thread_idx);
      Tensor tCrS_copy_view  = smem_thr_copy_S.retile_D(cute::get<1>(partitioned_extra_info));        // (CPY,CPY_M,CPY_K)

      if constexpr (KernelConversionMode == ConversionMode::ConvertAndScale) {
        return cute::make_tuple(smem_tiled_copy_S, tCrS_copy_view);
      }
      else if constexpr (KernelConversionMode == ConversionMode::ConvertAndScaleWithZero) {
        Tensor tCrZ_copy_view  = smem_thr_copy_S.retile_D(cute::get<3>(partitioned_extra_info));      // (CPY,CPY_M,CPY_K)
        return cute::make_tuple(smem_tiled_copy_S, tCrS_copy_view, tCrZ_copy_view);
      }
      else {
        static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
      }
    }
    else {
      static_assert(cutlass::detail::dependent_false<KernelSchedule>, "Conversion mode not handled in A -> RF path.");
    }
  }
};
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** The file handles specialized numeric formats or packed data representations used by accelerators.
  **CN:** 该文件处理加速器常用的特种数值格式或打包数据表示。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `cutlass/numeric_conversion.h`, `cute/util/type_traits.hpp`, `cute/arch/copy_sm90.hpp`, `cute/numeric/arithmetic_tuple.hpp`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `cutlass/numeric_conversion.h`, `cute/util/type_traits.hpp`, `cute/arch/copy_sm90.hpp`, `cute/numeric/arithmetic_tuple.hpp`。

- **EN:** Primary namespaces: `cutlass`, `detail`, `cutlass::gemm::collective::detail`.
  **CN:** 主要命名空间：`cutlass`, `detail`, `cutlass::gemm::collective::detail`。

- **EN:** Important macros or compile flags: `CUTLASS_DEVICE`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`.
  **CN:** 重要宏或编译开关：`CUTLASS_DEVICE`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`。
