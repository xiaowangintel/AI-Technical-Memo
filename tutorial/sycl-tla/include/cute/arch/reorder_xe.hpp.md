# reorder_xe.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/reorder_xe.hpp`
- **EN:** Defines low-level data/lane reordering helpers for Intel Xe or generic backends.
- **CN:** 为 Intel Xe 或通用后端定义底层数据/lane 重排辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
* Copyright (C) 2025 Intel Corporation, All rights reserved.
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

#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 34-35
```cpp
#include <cute/util/sycl_vec.hpp>           // native vector types
#include <cute/arch/reorder.hpp>            // Universal_Reorder_UU
```
- **EN:** Imports `cute/util/sycl_vec.hpp` (related definitions from `cute/util/sycl_vec.hpp`); `cute/arch/reorder.hpp` (related definitions from `cute/arch/reorder.hpp`).
- **CN:** 引入 `cute/util/sycl_vec.hpp`（来自 `cute/util/sycl_vec.hpp` 的相关定义）；`cute/arch/reorder.hpp`（来自 `cute/arch/reorder.hpp` 的相关定义）。

### Lines 37-39
```cpp
#if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
#define CUTE_ARCH_REORDER_XE_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__SYCL_DEVICE_ONLY__, CUTE_ARCH_REORDER_XE_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__SYCL_DEVICE_ONLY__, CUTE_ARCH_REORDER_XE_ENABLED）只启用当前目标有效的构建路径。

### Lines 41-41
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 43-44
```cpp
template <typename SrcType, typename DstType>
struct Xe_Reorder<ReorderKind::UU, SrcType, DstType> : Universal_Reorder_UU<SrcType, DstType> {};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 46-56
```cpp
template <typename T>
struct Xe_Reorder<ReorderKind::UU, T, T> {
  using StorageT = conditional_t<(sizeof_bits_v<T> >= 8), T, uint8_t>;
  using SRegisters = StorageT[1];
  using DRegisters = StorageT[1];

  CUTE_HOST_DEVICE static void
  reorder(StorageT const& src0, StorageT& dst0) {
    dst0 = src0;
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 58-85
```cpp
// Optimized reorder for float -> float_e5m2_t/float_e4m3_t (BF8/FP8) when classified as UU_Universal (e.g. size 16)
// This happens when the data chunk is smaller than a full GRF (e.g. 16 elements = 64B float -> 16B bf8)
template <>
struct Xe_Reorder<ReorderKind::UU_Universal, float, cutlass::float_e5m2_t>
{
  using SRegisters = intel::vector_t<float, 1>[1];
  using DRegisters = intel::vector_t<uint8_t, 1>[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::vector_t<float, 1> const& src, intel::vector_t<uint8_t, 1>& dst)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (
      "{\n"
      ".decl IN_F v_type=G type=F num_elts=16 alias=<%1,0>\n" 
      ".decl OUT_UB v_type=G type=UB num_elts=16 alias=<%0,0>\n"
      ".decl TMP_HF v_type=G type=HF num_elts=16 align=32\n"
      "mov (M1, 16) TMP_HF(0,0)<1> IN_F(0,0)<1;1,0>\n"
      "fcvt (M1_NM, 16) OUT_UB(0,0)<1> TMP_HF(0,0)<1;1,0>\n"
      "}\n"
      : "=rw"(dst) 
      : "rw"(src)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 87-112
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU_Universal, float, cutlass::float_e4m3_t>
{
  using SRegisters = intel::vector_t<float, 1>[1];
  using DRegisters = intel::vector_t<uint8_t, 1>[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::vector_t<float, 1> const& src, intel::vector_t<uint8_t, 1>& dst)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (
      "{\n"
      ".decl IN_F  v_type=G type=F  num_elts=16 alias=<%1,0>\n" 
      ".decl OUT_B v_type=G type=B  num_elts=16 alias=<%0,0>\n"
      ".decl TMP_HF v_type=G type=HF num_elts=16 align=32\n"
      "mov (M1, 16) TMP_HF(0,0)<1> IN_F(0,0)<1;1,0>\n"
      "fcvt (M1_NM, 16) OUT_B(0,0)<1> TMP_HF(0,0)<1;1,0>\n"
      "}\n"
      : "=rw"(dst) 
      : "rw"(src)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 114-141
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float, float_e5m2_t>
{
  using SRegisters = intel::float4[1];
  using DRegisters = intel::uchar4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::float4 const& src0, intel::uchar4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (
      "{\n"
        ".decl IN_F v_type=G type=F num_elts=64 alias=<%1,0>\n"
        ".decl OUT_UB v_type=G type=UB num_elts=64 alias=<%0,0>\n"
        ".decl TMP_HF v_type=G type=HF num_elts=64 align=64\n"
        "mov (M1_NM, 32) TMP_HF(0,0)<1> IN_F(0,0)<1;1,0>\n"
        "mov (M1_NM, 32) TMP_HF(1,0)<1> IN_F(2,0)<1;1,0>\n"
        "fcvt (M1_NM, 32) OUT_UB(0,0)<1> TMP_HF(0,0)<1;1,0>\n"
        "fcvt (M1_NM, 32) OUT_UB(0,32)<1> TMP_HF(1,0)<1;1,0>\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 143-170
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float, float_e4m3_t>
{
  using SRegisters = intel::float4[1];
  using DRegisters = intel::uchar4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::float4 const& src0, intel::uchar4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (
      "{\n"
        ".decl IN_F v_type=G type=F num_elts=64 alias=<%1,0>\n"
        ".decl OUT_B v_type=G type=B num_elts=64 alias=<%0,0>\n"
        ".decl TMP_HF v_type=G type=HF num_elts=64 align=64\n"
        "mov (M1_NM, 32) TMP_HF(0,0)<1> IN_F(0,0)<1;1,0>\n"
        "mov (M1_NM, 32) TMP_HF(1,0)<1> IN_F(2,0)<1;1,0>\n"
        "fcvt (M1_NM, 32) OUT_B(0,0)<1> TMP_HF(0,0)<1;1,0>\n"
        "fcvt (M1_NM, 32) OUT_B(0,32)<1> TMP_HF(1,0)<1;1,0>\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 172-199
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, uint8_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 2 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n"
      "or  (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0>  0x6400:uw\n"
      "or  (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 0x6400:uw\n"
      "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE400:hf\n"
      "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE400:hf\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 201-228
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, uint8_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 2 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n"
      "or  (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1>  0x6400:uw\n"
      "or  (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1>  0x6400:uw\n"
      "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE400:hf\n"
      "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE400:hf\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 230-257
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, int8_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 2 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n"
      "xor (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0>  0x6480:uw\n"
      "xor (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 0x6480:uw\n"
      "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE480:hf\n"
      "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE480:hf\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 259-286
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, int8_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 2 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n"
      "xor (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1>  0x6480:uw\n"
      "xor (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1>  0x6480:uw\n"
      "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE480:hf\n"
      "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE480:hf\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 288-297
```cpp
// Common uint8 -> bfloat16 conversion sequence, after unpacking bytes to words.
// This is defined as a macro as the compiler produces more efficient code
//   when inline asm blocks are merged.
#define CUTE_XE_REORDER_U8_BF16_SEQ \
    ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_BF v_type=G type=BF num_elts=64 alias=<%0,0>\n" \
    "mul (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0x7E000000:f\n" \
    "mul (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0x7E000000:f\n" \
    "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x4000:hf\n" \
    "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x4000:hf\n"
```
- **EN:** Defines or forwards `mul` as part of this header's executable interface.
- **CN:** 定义或转发 `mul`，作为该头文件可执行接口的一部分。

### Lines 300-325
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, uint8_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "mov (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0>\n"
      "mov (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0>\n"
      CUTE_XE_REORDER_U8_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 327-352
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, uint8_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "mov (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1>\n"
      "mov (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1>\n"
      CUTE_XE_REORDER_U8_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 355-364
```cpp
// Common int8 -> bfloat16 conversion sequence, after unpacking bytes to words.
#define CUTE_XE_REORDER_S8_BF16_SEQ \
    ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_BF v_type=G type=BF num_elts=64 alias=<%0,0>\n" \
    ".decl F_7E000000 v_type=G type=F num_elts=1 alias=<%2,0>\n" \
    ".decl F_BF000000 v_type=G type=F num_elts=1 alias=<%3,0>\n" \
    "mad (M1_NM, 32) OUT_BF(0,0)<1> F_7E000000(0,0)<0;1,0> OUT_BF(0,0)<1;1,0> F_BF000000(0,0)<0;1,0>\n" \
    "mad (M1_NM, 32) OUT_BF(1,0)<1> F_7E000000(0,0)<0;1,0> OUT_BF(1,0)<1;1,0> F_BF000000(0,0)<0;1,0>\n" \
    "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x4000:hf\n" \
    "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x4000:hf\n"
```
- **EN:** Defines or forwards `mad` as part of this header's executable interface.
- **CN:** 定义或转发 `mad`，作为该头文件可执行接口的一部分。

### Lines 367-394
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, int8_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t scale = 0x7E000000;
    const uint32_t shift = 0xBF000000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "xor (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0>  0x80:uw\n"
      "xor (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 0x80:uw\n"
      CUTE_XE_REORDER_S8_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(scale), "rw.u"(shift)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 396-423
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, int8_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t scale = 0x7E000000;
    const uint32_t shift = 0xBF000000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "xor (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1> 0x80:uw\n"
      "xor (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1> 0x80:uw\n"
      CUTE_XE_REORDER_S8_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(scale), "rw.u"(shift)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 425-449
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e5m2_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 1 cycle/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 8:uw\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 451-475
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e5m2_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 1 cycle/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1> 8:uw\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 477-486
```cpp
// Common e5m2 -> bfloat16 conversion sequence, after shl by 8.
#define CUTE_XE_REORDER_E5M2_BF16_SEQ \
    ".decl OUT_W v_type=G type=W num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_UD v_type=G type=UD num_elts=32 alias=<%0,0>\n" \
    ".decl OUT_BF v_type=G type=BF num_elts=64 alias=<%0,0>\n" \
    "asr (M1_NM, 32) OUT_W(0,0)<1> OUT_W(0,0)<1;1,0> 3:uw\n" \
    "asr (M1_NM, 32) OUT_W(1,0)<1> OUT_W(1,0)<1;1,0> 3:uw\n" \
    "and (M1_NM, 32) OUT_UD(0,0)<1> OUT_UD(0,0)<1;1,0> 0x8FFF8FFF:ud\n" \
    "mul (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0x77800000:f\n" \
    "mul (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0x77800000:f\n"
```
- **EN:** Defines or forwards `asr` as part of this header's executable interface.
- **CN:** 定义或转发 `asr`，作为该头文件可执行接口的一部分。

### Lines 488-513
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e5m2_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 8:uw\n"
      CUTE_XE_REORDER_E5M2_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 515-540
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e5m2_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1> 8:uw\n"
      CUTE_XE_REORDER_E5M2_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 543-559
```cpp
// Common e4m3 -> half conversion sequence, after shl by 8.
#define CUTE_XE_REORDER_E4M3_HALF_SEQ \
    ".decl OUT_W v_type=G type=W num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_UD v_type=G type=UD num_elts=32 alias=<%0,0>\n" \
    ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n" \
    "asr (M1_NM, 32) OUT_W(0,0)<1> OUT_W(0,0)<1;1,0> 1:uw\n" \
    "asr (M1_NM, 32) OUT_W(1,0)<1> OUT_W(1,0)<1;1,0> 1:uw\n" \
    "and (M1_NM, 32) OUT_UD(0,0)<1> OUT_UD(0,0)<1;1,0> 0xBFFFBFFF:ud\n" \
    /* If no NaN inputs, the rest of the sequence can be replaced with: */ \
    /*  "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x5C00:hf\n" */ \
    /*  "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x5C00:hf\n" */ \
    "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x7880:hf\n" \
    "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x7880:hf\n" \
    "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x1F1C:hf\n" \
    "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x1F1C:hf\n" \
    "mad (M1_NM, 32) OUT_HF(0,0)<1> 0x0:hf OUT_HF(0,0)<1;1,0> OUT_HF(0,0)<1;1,0>\n" \
    "mad (M1_NM, 32) OUT_HF(1,0)<1> 0x0:hf OUT_HF(1,0)<1;1,0> OUT_HF(1,0)<1;1,0>\n"
```
- **EN:** Defines or forwards `asr` as part of this header's executable interface.
- **CN:** 定义或转发 `asr`，作为该头文件可执行接口的一部分。

### Lines 561-586
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e4m3_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 6 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 8:uw\n"
      CUTE_XE_REORDER_E4M3_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 588-613
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e4m3_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 6 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1> 8:uw\n"
      CUTE_XE_REORDER_E4M3_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 615-631
```cpp
// Common e4m3 -> bfloat16 conversion sequence, after shl by 8.
#define CUTE_XE_REORDER_E4M3_BF16_SEQ \
    ".decl OUT_W v_type=G type=W num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_UD v_type=G type=UD num_elts=32 alias=<%0,0>\n" \
    ".decl OUT_HF v_type=G type=HF num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_BF v_type=G type=BF num_elts=64 alias=<%0,0>\n" \
    ".decl NZ_PRED0 v_type=P num_elts=32\n" \
    ".decl NZ_PRED1 v_type=P num_elts=32\n" \
    "asr (M1_NM, 32) OUT_W(0,0)<1> OUT_W(0,0)<1;1,0> 4:uw\n" \
    "asr (M1_NM, 32) OUT_W(1,0)<1> OUT_W(1,0)<1;1,0> 4:uw\n" \
    "and (M1_NM, 32) OUT_UD(0,0)<1> OUT_UD(0,0)<1;1,0> 0x87FF87FF:ud\n" \
    "cmp.ge (M1_NM, 32) NZ_PRED0 (abs)OUT_HF(0,0)<1;1,0> 0x07F0:hf\n" \
    "cmp.ge (M1_NM, 32) NZ_PRED1 (abs)OUT_HF(1,0)<1;1,0> 0x07F0:hf\n" \
    "mul (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0x7B800000:f\n" \
    "mul (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0x7B800000:f\n" \
    "(NZ_PRED0) mov (M1_NM, 32) OUT_UW(0,0)<1> 0x7FC0:uw\n" \
    "(NZ_PRED1) mov (M1_NM, 32) OUT_UW(1,0)<1> 0x7FC0:uw\n"
```
- **EN:** Defines or forwards `asr` as part of this header's executable interface.
- **CN:** 定义或转发 `asr`，作为该头文件可执行接口的一部分。

### Lines 633-658
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e4m3_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 7 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;1,0> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,32)<1;1,0> 8:uw\n"
      CUTE_XE_REORDER_E4M3_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 660-685
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e4m3_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 7 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=64 alias=<%0,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,1> 8:uw\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,2)<4;2,1> 8:uw\n"
      CUTE_XE_REORDER_E4M3_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 687-697
```cpp
// Common uint4 -> half conversion sequence, after expanding nybbles to words.
#define CUTE_XE_REORDER_U4_HALF_SEQ \
    ".decl OUT_HF v_type=G type=HF num_elts=128 alias=<%0,0>\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(0,0)<1> 0x6400:uw OUT_UW(0,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(1,0)<1> 0x6400:uw OUT_UW(1,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(2,0)<1> 0x6400:uw OUT_UW(2,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(3,0)<1> 0x6400:uw OUT_UW(3,0)<1;1,0> 0xF:uw\n" \
    "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE400:hf\n" \
    "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE400:hf\n" \
    "add (M1_NM, 32) OUT_HF(2,0)<1> OUT_HF(2,0)<1;1,0> 0xE400:hf\n" \
    "add (M1_NM, 32) OUT_HF(3,0)<1> OUT_HF(3,0)<1;1,0> 0xE400:hf\n"
```
- **EN:** Defines or forwards `xCA` as part of this header's executable interface.
- **CN:** 定义或转发 `xCA`，作为该头文件可执行接口的一部分。

### Lines 700-729
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, uint4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 3 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 731-764
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, uint4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 766-795
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, uint4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 3 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 797-807
```cpp
// Common int4 -> half conversion sequence, after expanding nybbles to words.
#define CUTE_XE_REORDER_S4_HALF_SEQ \
    ".decl OUT_HF v_type=G type=HF num_elts=128 alias=<%0,0>\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(0,0)<1> 0x6408:uw OUT_UW(0,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(1,0)<1> 0x6408:uw OUT_UW(1,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(2,0)<1> 0x6408:uw OUT_UW(2,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(3,0)<1> 0x6408:uw OUT_UW(3,0)<1;1,0> 0xF:uw\n" \
    "add (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0xE408:hf\n" \
    "add (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0xE408:hf\n" \
    "add (M1_NM, 32) OUT_HF(2,0)<1> OUT_HF(2,0)<1;1,0> 0xE408:hf\n" \
    "add (M1_NM, 32) OUT_HF(3,0)<1> OUT_HF(3,0)<1;1,0> 0xE408:hf\n"
```
- **EN:** Defines or forwards `x6A` as part of this header's executable interface.
- **CN:** 定义或转发 `x6A`，作为该头文件可执行接口的一部分。

### Lines 810-839
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, int4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 3 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 841-874
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, int4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 876-905
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, int4_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 3 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 907-917
```cpp
// Common uint4 -> bfloat16 conversion sequence, after expanding nybbles to words.
#define CUTE_XE_REORDER_U4_BF16_SEQ \
    ".decl OUT_BF v_type=G type=BF num_elts=128 alias=<%0,0>\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(0,0)<1> 0x4300:uw OUT_UW(0,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(1,0)<1> 0x4300:uw OUT_UW(1,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(2,0)<1> 0x4300:uw OUT_UW(2,0)<1;1,0> 0xF:uw\n" \
    "bfn.xCA (M1_NM, 32) OUT_UW(3,0)<1> 0x4300:uw OUT_UW(3,0)<1;1,0> 0xF:uw\n" \
    "add (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0xC3000000:f\n" \
    "add (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0xC3000000:f\n" \
    "add (M1_NM, 32) OUT_BF(2,0)<1> OUT_BF(2,0)<1;1,0> 0xC3000000:f\n" \
    "add (M1_NM, 32) OUT_BF(3,0)<1> OUT_BF(3,0)<1;1,0> 0xC3000000:f\n"
```
- **EN:** Defines or forwards `xCA` as part of this header's executable interface.
- **CN:** 定义或转发 `xCA`，作为该头文件可执行接口的一部分。

### Lines 920-949
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, uint4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 951-984
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, uint4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 986-1015
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, uint4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_U4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1017-1027
```cpp
// Common int4 -> bfloat16 conversion sequence, after expanding nybbles to words.
#define CUTE_XE_REORDER_S4_BF16_SEQ \
    ".decl OUT_BF v_type=G type=BF num_elts=128 alias=<%0,0>\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(0,0)<1> 0x4308:uw OUT_UW(0,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(1,0)<1> 0x4308:uw OUT_UW(1,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(2,0)<1> 0x4308:uw OUT_UW(2,0)<1;1,0> 0xF:uw\n" \
    "bfn.x6A (M1_NM, 32) OUT_UW(3,0)<1> 0x4308:uw OUT_UW(3,0)<1;1,0> 0xF:uw\n" \
    "add (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0xC3080000:f\n" \
    "add (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0xC3080000:f\n" \
    "add (M1_NM, 32) OUT_BF(2,0)<1> OUT_BF(2,0)<1;1,0> 0xC3080000:f\n" \
    "add (M1_NM, 32) OUT_BF(3,0)<1> OUT_BF(3,0)<1;1,0> 0xC3080000:f\n"
```
- **EN:** Defines or forwards `x6A` as part of this header's executable interface.
- **CN:** 定义或转发 `x6A`，作为该头文件可执行接口的一部分。

### Lines 1030-1059
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, int4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1061-1094
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, int4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1096-1125
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, int4_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x00040000;
    asm (     /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shr (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_S4_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1127-1141
```cpp
// Common e2m1 -> half conversion sequence, after moving nybbles to highest 4 bits of each word.
#define CUTE_XE_REORDER_E2M1_HALF_SEQ \
    ".decl OUT_W v_type=G type=W num_elts=128 alias=<%0,0>\n" \
    ".decl OUT_UD v_type=G type=UD num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_HF v_type=G type=HF num_elts=128 alias=<%0,0>\n" \
    "asr (M1_NM, 32) OUT_W(0,0)<1> OUT_W(0,0)<1;1,0> 3:uw\n" \
    "asr (M1_NM, 32) OUT_W(1,0)<1> OUT_W(1,0)<1;1,0> 3:uw\n" \
    "asr (M1_NM, 32) OUT_W(2,0)<1> OUT_W(2,0)<1;1,0> 3:uw\n" \
    "asr (M1_NM, 32) OUT_W(3,0)<1> OUT_W(3,0)<1;1,0> 3:uw\n" \
    "and (M1_NM, 32) OUT_UD(0,0)<1> OUT_UD(0,0)<1;1,0> 0x8E008E00:ud\n" \
    "and (M1_NM, 32) OUT_UD(2,0)<1> OUT_UD(2,0)<1;1,0> 0x8E008E00:ud\n" \
    "mul (M1_NM, 32) OUT_HF(0,0)<1> OUT_HF(0,0)<1;1,0> 0x7400:hf\n" \
    "mul (M1_NM, 32) OUT_HF(1,0)<1> OUT_HF(1,0)<1;1,0> 0x7400:hf\n" \
    "mul (M1_NM, 32) OUT_HF(2,0)<1> OUT_HF(2,0)<1;1,0> 0x7400:hf\n" \
    "mul (M1_NM, 32) OUT_HF(3,0)<1> OUT_HF(3,0)<1;1,0> 0x7400:hf\n"
```
- **EN:** Defines or forwards `asr` as part of this header's executable interface.
- **CN:** 定义或转发 `asr`，作为该头文件可执行接口的一部分。

### Lines 1143-1172
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e2m1_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (   /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1174-1207
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, float_e2m1_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (     /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1209-1238
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e2m1_t, half_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (   /* 4 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_HALF_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1240-1254
```cpp
// Common e2m1 -> bfloat16 conversion sequence, after moving nybbles to highest 4 bits of each word.
#define CUTE_XE_REORDER_E2M1_BF16_SEQ \
    ".decl OUT_W v_type=G type=W num_elts=128 alias=<%0,0>\n" \
    ".decl OUT_UD v_type=G type=UD num_elts=64 alias=<%0,0>\n" \
    ".decl OUT_BF v_type=G type=BF num_elts=128 alias=<%0,0>\n" \
    "asr (M1_NM, 32) OUT_W(0,0)<1> OUT_W(0,0)<1;1,0> 6:uw\n" \
    "asr (M1_NM, 32) OUT_W(1,0)<1> OUT_W(1,0)<1;1,0> 6:uw\n" \
    "asr (M1_NM, 32) OUT_W(2,0)<1> OUT_W(2,0)<1;1,0> 6:uw\n" \
    "asr (M1_NM, 32) OUT_W(3,0)<1> OUT_W(3,0)<1;1,0> 6:uw\n" \
    "and (M1_NM, 32) OUT_UD(0,0)<1> OUT_UD(0,0)<1;1,0> 0x81C081C0:ud\n" \
    "and (M1_NM, 32) OUT_UD(2,0)<1> OUT_UD(2,0)<1;1,0> 0x81C081C0:ud\n" \
    "mul (M1_NM, 32) OUT_BF(0,0)<1> OUT_BF(0,0)<1;1,0> 0x7E800000:f\n" \
    "mul (M1_NM, 32) OUT_BF(1,0)<1> OUT_BF(1,0)<1;1,0> 0x7E800000:f\n" \
    "mul (M1_NM, 32) OUT_BF(2,0)<1> OUT_BF(2,0)<1;1,0> 0x7E800000:f\n" \
    "mul (M1_NM, 32) OUT_BF(3,0)<1> OUT_BF(3,0)<1;1,0> 0x7E800000:f\n"
```
- **EN:** Defines or forwards `asr` as part of this header's executable interface.
- **CN:** 定义或转发 `asr`，作为该头文件可执行接口的一部分。

### Lines 1256-1285
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_e2m1_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (   /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<1;2,0>  SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1287-1320
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, float_e2m1_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (     /* 6 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 16) OUT_UW(0,0)<2> IN_UB(0,0)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(0,1)<2> IN_UB(0,8)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(1,0)<2> IN_UB(0,16)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(1,1)<2> IN_UB(0,24)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(2,0)<2> IN_UB(0,32)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(2,1)<2> IN_UB(0,40)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(3,0)<2> IN_UB(0,48)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) OUT_UW(3,1)<2> IN_UB(0,56)<1;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1322-1351
```cpp
template <>
struct Xe_Reorder<ReorderKind::VV, float_e2m1_t, bfloat16_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::ushort8[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::ushort8& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t shifts = 0x0008000C;
    asm (   /* 5 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      ".decl SHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      "shl (M1_NM, 32) OUT_UW(0,0)<1> IN_UB(0,0)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(1,0)<1> IN_UB(0,1)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(2,0)<1> IN_UB(0,2)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 32) OUT_UW(3,0)<1> IN_UB(0,3)<4;2,0> SHIFTS(0,0)<0;2,1>\n"
      CUTE_XE_REORDER_E2M1_BF16_SEQ
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(shifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1353-1381
```cpp
template <>
struct Xe_Reorder<ReorderKind::UU, float_ue8m0_t, float>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::float4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::float4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    asm (     /* 3 cycles/output register */
      "{\n"
      ".decl IN_UB v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=128 alias=<%0,0>\n"
      "shl (M1_NM, 32)     OUT_UW(0,1)<2>  IN_UB(0,0)<1;1,0>   7:uw\n"
      "shl (M1_NM, 32)     OUT_UW(2,1)<2>  IN_UB(0,32)<1;1,0>  7:uw\n"
      "add.sat (M1_NM, 32) OUT_UW(0,0)<2>  IN_UB(0,0)<1;1,0>   -254:w\n"
      "add.sat (M1_NM, 32) OUT_UW(2,0)<2>  IN_UB(0,32)<1;1,0>  -254:w\n"
      "max (M1_NM, 32)     OUT_UW(0,1)<2>  OUT_UW(0,1)<2>      0x40:uw\n"
      "max (M1_NM, 32)     OUT_UW(2,1)<2>  OUT_UW(2,1)<2>      0x40:uw\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1383-1421
```cpp
template <>
struct Xe_Reorder<ReorderKind::UV, uint4_t, uint4_t>
{
  using SRegisters = intel::uchar4[1];
  using DRegisters = intel::uchar4[1];

  CUTE_HOST_DEVICE static void
  reorder(intel::uchar4 const& src0, intel::uchar4& dst0)
  {
#if defined(CUTE_ARCH_REORDER_XE_ENABLED)
    const uint32_t lshifts = 0x00000004;
    const uint32_t rshifts = 0x00040000;
    asm (     /* 9 cycles/output register */
      "{\n"
      ".decl IN_UB  v_type=G type=UB num_elts=64 alias=<%1,0>\n"
      ".decl OUT_UB v_type=G type=UB num_elts=64 alias=<%0,0>\n"
      ".decl OUT_UW v_type=G type=UW num_elts=32 alias=<%0,0>\n"
      ".decl LSHIFTS v_type=G type=UW num_elts=2 alias=<%2,0>\n"
      ".decl RSHIFTS v_type=G type=UW num_elts=2 alias=<%3,0>\n"
      ".decl TMP_UB v_type=G type=UB num_elts=64 align=64\n"
      ".decl TMP_UW v_type=G type=UW num_elts=32 alias=<TMP_UB,0>\n"
      "shr (M1_NM, 16) OUT_UB(0,0)<4> IN_UB(0, 0)<1;2,0> RSHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UB(0,1)<4> IN_UB(0,16)<1;2,0> RSHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UB(0,2)<4> IN_UB(0,32)<1;2,0> RSHIFTS(0,0)<0;2,1>\n"
      "shr (M1_NM, 16) OUT_UB(0,3)<4> IN_UB(0,48)<1;2,0> RSHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) TMP_UB(0,0)<4> IN_UB(0, 8)<1;2,0> LSHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) TMP_UB(0,1)<4> IN_UB(0,24)<1;2,0> LSHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) TMP_UB(0,2)<4> IN_UB(0,40)<1;2,0> LSHIFTS(0,0)<0;2,1>\n"
      "shl (M1_NM, 16) TMP_UB(0,3)<4> IN_UB(0,56)<1;2,0> LSHIFTS(0,0)<0;2,1>\n"
      "bfn.xCA (M1_NM, 32) OUT_UW(0,0)<1> OUT_UW(0,0)<1;1,0> TMP_UW(0,0)<1;1,0> 0xF0F0:uw\n"
      "}\n"
      : "=rw"(dst0)
      : "rw"(src0), "rw.u"(lshifts), "rw.u"(rshifts)
    );
#else
  CUTE_INVALID_CONTROL_PATH("Not Xe");
#endif
  }
};
```
- **EN:** Defines `Xe_Reorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Xe_Reorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1423-1424
```cpp
template <> struct Xe_Reorder<ReorderKind::UV, int4_t, int4_t>             : Xe_Reorder<ReorderKind::UV, uint4_t, uint4_t> {};
template <> struct Xe_Reorder<ReorderKind::UV, float_e2m1_t, float_e2m1_t> : Xe_Reorder<ReorderKind::UV, uint4_t, uint4_t> {};
```
- **EN:** Implements a supporting fragment of the surrounding algorithm, trait, or architecture wrapper.
- **CN:** 实现周边算法、trait 或体系结构封装所需的辅助片段。

### Lines 1426-1426
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
- **EN:** `cute/arch/reorder.hpp` supplies related definitions from `cute/arch/reorder.hpp`.
  **CN:** `cute/arch/reorder.hpp` 提供了来自 `cute/arch/reorder.hpp` 的相关定义。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
