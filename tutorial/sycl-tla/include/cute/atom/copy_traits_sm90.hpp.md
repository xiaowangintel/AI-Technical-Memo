# copy_traits_sm90.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/atom/copy_traits_sm90.hpp`
- **EN:** Defines copy-trait specializations that map logical copy layouts onto NVIDIA SM90 instructions.
- **CN:** 定义 copy trait 特化，把逻辑拷贝布局映射到 NVIDIA SM90 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
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
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-35
```cpp
#include <cute/arch/copy_sm90.hpp>
#include <cute/atom/copy_traits.hpp>
#include <cute/atom/copy_traits_sm75.hpp>
```
- **EN:** Imports `cute/arch/copy_sm90.hpp` (related definitions from `cute/arch/copy_sm90.hpp`); `cute/atom/copy_traits.hpp` (generic copy-trait interfaces and utilities); `cute/atom/copy_traits_sm75.hpp` (related definitions from `cute/atom/copy_traits_sm75.hpp`).
- **CN:** 引入 `cute/arch/copy_sm90.hpp`（来自 `cute/arch/copy_sm90.hpp` 的相关定义）；`cute/atom/copy_traits.hpp`（通用 copy trait 接口与工具）；`cute/atom/copy_traits_sm75.hpp`（来自 `cute/atom/copy_traits_sm75.hpp` 的相关定义）。

### Lines 37-37
```cpp
#include <cute/layout.hpp>
```
- **EN:** Imports `cute/layout.hpp` (related definitions from `cute/layout.hpp`).
- **CN:** 引入 `cute/layout.hpp`（来自 `cute/layout.hpp` 的相关定义）。

### Lines 39-40
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 42-55
```cpp
template <>
struct Copy_Traits<SM90_U32x1_STSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U32x1_LDSM_N>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U32x1_LDSM_N>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 57-70
```cpp
template <>
struct Copy_Traits<SM90_U32x2_STSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U32x2_LDSM_N>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U32x2_LDSM_N>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 72-85
```cpp
template <>
struct Copy_Traits<SM90_U32x4_STSM_N>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U32x4_LDSM_N>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U32x4_LDSM_N>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 87-100
```cpp
template <>
struct Copy_Traits<SM90_U16x2_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U16x2_LDSM_T>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U16x2_LDSM_T>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 102-115
```cpp
template <>
struct Copy_Traits<SM90_U16x4_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U16x4_LDSM_T>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U16x4_LDSM_T>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 117-130
```cpp
template <>
struct Copy_Traits<SM90_U16x8_STSM_T>
{
  // Logical thread id to thread idx (warp)
  using ThrID   = Layout<_32>;

  // Map from (src-thr,src-val) to bit
  using SrcLayout = typename Copy_Traits<SM75_U16x8_LDSM_T>::DstLayout;
  // Map from (dst-thr,dst-val) to bit
  using DstLayout = typename Copy_Traits<SM75_U16x8_LDSM_T>::SrcLayout;

  // Reference map from (thr,val) to bit
  using RefLayout = SrcLayout;
};
```
- **EN:** Defines or specializes `Copy_Traits`, mapping a copy operation onto logical thread/value layouts and stored parameters.
- **CN:** 定义或特化 `Copy_Traits`，把拷贝操作映射到逻辑线程/值布局以及保存的参数。

### Lines 132-132
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Compile-time layouts/strides describe how logical coordinates map onto storage.
  **CN:** 编译期布局/步长描述了逻辑坐标如何映射到实际存储。
- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。

## Dependencies / 依赖关系

- **EN:** `cute/arch/copy_sm90.hpp` supplies related definitions from `cute/arch/copy_sm90.hpp`.
  **CN:** `cute/arch/copy_sm90.hpp` 提供了来自 `cute/arch/copy_sm90.hpp` 的相关定义。
- **EN:** `cute/atom/copy_traits.hpp` supplies generic copy-trait interfaces and utilities.
  **CN:** `cute/atom/copy_traits.hpp` 提供了通用 copy trait 接口与工具。
- **EN:** `cute/atom/copy_traits_sm75.hpp` supplies related definitions from `cute/atom/copy_traits_sm75.hpp`.
  **CN:** `cute/atom/copy_traits_sm75.hpp` 提供了来自 `cute/atom/copy_traits_sm75.hpp` 的相关定义。
- **EN:** `cute/layout.hpp` supplies related definitions from `cute/layout.hpp`.
  **CN:** `cute/layout.hpp` 提供了来自 `cute/layout.hpp` 的相关定义。
