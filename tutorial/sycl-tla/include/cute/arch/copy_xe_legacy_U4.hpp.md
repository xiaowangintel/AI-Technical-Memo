# copy_xe_legacy_U4.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe_legacy_U4.hpp`
- **EN:** Provides legacy Intel Xe copy builtins, SPIR-V wrappers, or typed front-ends.
- **CN:** 提供传统 Intel Xe 拷贝内建、SPIR-V 封装或类型化前端。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
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

### Lines 34-36
```cpp
#include <cute/util/sycl_vec.hpp>
#include "cute/config.hpp"
#include "cute/pointer.hpp"
```
- **EN:** Imports `cute/util/sycl_vec.hpp` (related definitions from `cute/util/sycl_vec.hpp`); `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/pointer.hpp` (related definitions from `cute/pointer.hpp`).
- **CN:** 引入 `cute/util/sycl_vec.hpp`（来自 `cute/util/sycl_vec.hpp` 的相关定义）；`cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/pointer.hpp`（来自 `cute/pointer.hpp` 的相关定义）。

### Lines 38-39
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 41-55
```cpp
  struct XE_2D_Packed_U4x1x128_LD_N {
  using BlockShape = Shape<_1, _128>;
  using inst_dtype = uint32_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockLoad<4, 16, 1, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_Packed_U4x1x128_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_Packed_U4x1x128_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 57-72
```cpp
struct XE_2D_U4x16x8_LD_T {
  using BlockShape = Shape<_8, _16>;
  using inst_dtype = uint32_t;
  static constexpr bool is_transpose = true;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockLoadTranspose<4, 1, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U4x16x8_LD_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U4x16x8_LD_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 74-89
```cpp
struct XE_2D_U4x16x16_LD_T {
  using BlockShape = Shape<_16, _16>;
  using inst_dtype = uint32_t;
  static constexpr bool is_transpose = true;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockLoadTranspose<4, 2, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U4x16x16_LD_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U4x16x16_LD_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 91-106
```cpp
struct XE_2D_U4x32x16_LD_T {
  using BlockShape = Shape<_32, _16>;
  using inst_dtype = uint32_t;
  static constexpr bool is_transpose = true;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    detail::XeSubgroup2DBlockLoadTranspose<4, 4, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U4x32x16_LD_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U4x32x16_LD_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 108-156
```cpp
struct XE_2D_U4x32x64_LD_N {
  using BlockShape = Shape<_32, _64>;
  using inst_dtype = int8_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 1, "Expected T to have size 1");
    detail::XeSubgroup2DBlockLoad<1, 32, 32, 1>{}(baseoffset, width, height, pitch, coord, dst);

   // ================= shuffle begin =================
   // FIXME: the performance of shuffle algorithm here is too bad, we are working with
   // compiler/IGC team to optimize it.

    static constexpr auto subgroup_size = 16;
    static constexpr auto copy_W = decltype(size<1>(BlockShape{}))::value / subgroup_size;
    static constexpr auto copy_H = decltype(size<0>(BlockShape{}))::value;

    auto sg = compat::get_nd_item<1>().get_sub_group();
    auto id = int(ThreadIdxX()) % subgroup_size;

    cute::subbyte_iterator<int4_t> dst_iter(dst);
    cute::array_subbyte<int4_t, copy_W * copy_H> dst_tmp{};

    #pragma unroll
    for (int cw = 0; cw < copy_W; cw++) {
      auto remote_id = (id + cw * subgroup_size) / copy_W;

      // TODO: select 'ushort32' will cause compiling error, use 'ushort16' instead, why?
      intel::ushort16 remote_dst[2];
      remote_dst[0] = sycl::select_from_group(sg, *(reinterpret_cast<intel::ushort16 *>(dst)), remote_id);
      remote_dst[1] = sycl::select_from_group(sg, *((reinterpret_cast<intel::ushort16 *>(dst)) + 1), remote_id);

      cute::subbyte_iterator<int4_t> remote_dst_iter(remote_dst);

      #pragma unroll
      for (int row = 0; row < copy_H; row++) {
        dst_tmp[row + cw * copy_H] = remote_dst_iter[row * copy_W + id % copy_W].get();
      }
    }

   *reinterpret_cast<intel::ushort32 *>(cute::raw_pointer_cast(dst_iter)) = *reinterpret_cast<intel::ushort32 *>(cute::raw_pointer_cast(dst_tmp.begin()));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U4x32x64_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U4x32x64_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 158-205
```cpp
struct XE_2D_U4x16x64_LD_N {
  using BlockShape = Shape<_16, _64>;
  using inst_dtype = int8_t;

  template <class T>
  CUTE_HOST_DEVICE static void copy(const void *baseoffset, int width,
                                    int height, int pitch, intel::coord_t coord,
                                    T *dst) {
#if defined(CUTE_ARCH_COPY_XE_ENABLED)
    static_assert(sizeof(T) == 1, "Expected T to have size 1");
    detail::XeSubgroup2DBlockLoad<1, 32, 16, 1>{}(baseoffset, width, height, pitch, coord, dst);

   // ================= shuffle begin =================
   // FIXME: the performance of shuffle algorithm here is too bad, we are working with
   // compiler/IGC team to optimize it.

    static constexpr auto subgroup_size = 16;
    static constexpr auto copy_W = decltype(size<1>(BlockShape{}))::value / subgroup_size;
    static constexpr auto copy_H = decltype(size<0>(BlockShape{}))::value;

    auto sg = compat::get_nd_item<1>().get_sub_group();
    auto id = int(ThreadIdxX()) % subgroup_size;

    cute::subbyte_iterator<int4_t> dst_iter(dst);
    cute::array_subbyte<int4_t, copy_W * copy_H> dst_tmp{};

    #pragma unroll
    for (int cw = 0; cw < copy_W; cw++) {
      auto remote_id = (id + cw * subgroup_size) / copy_W;

      intel::ushort16 remote_dst;
      remote_dst = sycl::select_from_group(sg, *(reinterpret_cast<intel::ushort16 *>(dst)), remote_id);

      cute::subbyte_iterator<int4_t> remote_dst_iter(&remote_dst);

      #pragma unroll
      for (int row = 0; row < copy_H; row++) {
        dst_tmp[row + cw * copy_H] = remote_dst_iter[row * copy_W + id % copy_W].get();
      }
    }

   *reinterpret_cast<intel::ushort16 *>(cute::raw_pointer_cast(dst_iter)) = *reinterpret_cast<intel::ushort16 *>(cute::raw_pointer_cast(dst_tmp.begin()));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_2D_U4x16x64_LD_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_2D_U4x16x64_LD_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 207-207
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/pointer.hpp` supplies related definitions from `cute/pointer.hpp`.
  **CN:** `cute/pointer.hpp` 提供了来自 `cute/pointer.hpp` 的相关定义。
