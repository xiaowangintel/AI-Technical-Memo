# prefetch.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/algorithm/prefetch.hpp`
- **EN:** Provides prefetch helpers that stage data or metadata before the main operation.
- **CN:** 提供预取辅助工具，在主操作前提前搬运数据或元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
#include <cute/config.hpp>          // CUTE_HOST_DEVICE
#include <cute/tensor_impl.hpp>     // cute::Tensor
#include <cute/atom/copy_atom.hpp>  // cute::Copy_Atom
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/tensor_impl.hpp` (tensor storage, indexing, and layout implementation details); `cute/atom/copy_atom.hpp` (copy atoms that combine traits with tiled tensor views).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/tensor_impl.hpp`（张量存储、索引与布局实现细节）；`cute/atom/copy_atom.hpp`（把 traits 与分块张量视图结合起来的 copy atom）。

### Lines 37-38
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 40-72
```cpp
//
// Prefetch global tensors into L2
//

template <uint32_t NumThreads, uint32_t FetchBytes = 64,
          class GEngine, class GLayout>
CUTE_HOST_DEVICE
void
cooperative_prefetch(uint32_t                 const& tid,
                     Tensor<GEngine, GLayout> const& src)
{
  static_assert(is_gmem<GEngine>::value, "Expected global tensor for prefetch");

  constexpr int V = decltype(max_common_vector(src, src))::value;

  if constexpr (V > 1) {
    // L2 sector is 32B, default fetch granularity is 64B
    using VecType = conditional_t<(V * sizeof_bits_v<typename GEngine::value_type>) < (FetchBytes * 8),
                                  ArrayEngine<typename GEngine::value_type, V>,
                                  uint8_t[FetchBytes]                         >;

    Tensor src_v = recast<VecType const>(src);
    CUTE_UNROLL
    for (int i = tid; i < size(src_v); i += NumThreads) {
      prefetch(raw_pointer_cast(&src_v(i)));
    }
  } else {
    CUTE_UNROLL
    for (int i = tid; i < size(src); i += NumThreads) {
      prefetch(raw_pointer_cast(&src(i)));
    }
  }
}
```
- **EN:** Defines `cooperative_prefetch`, iterating over tensor elements or coordinates to perform the intended operation.
- **CN:** 定义 `cooperative_prefetch`，通过遍历张量元素或坐标来执行目标操作。

### Lines 74-80
```cpp
template <class GEngine, class GLayout>
CUTE_HOST_DEVICE
void
prefetch(Tensor<GEngine, GLayout> const& src)
{
  return cooperative_prefetch<1>(0, src);
}
```
- **EN:** Defines `prefetch`, issuing a prefetch request so upcoming data reaches a closer cache level before it is consumed.
- **CN:** 定义 `prefetch`：发出预取请求，使即将访问的数据在被消费前进入更近的缓存层级。

### Lines 82-91
```cpp
// Prefetch with copy atom
namespace detail {

template <class CopyOp, class = void>
constexpr bool has_prefetch = false;

template <class CopyOp>
constexpr bool has_prefetch<CopyOp, void_t<typename CopyOp::PREFETCH>> = true;

} // end namespace detail
```
- **EN:** Enters or leaves namespace scope `detail, detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `detail, detail`，以便把相关符号组织在一起。

### Lines 93-110
```cpp
template <class CopyOp, class... CT_Args, class CopyType,
          class GEngine, class GLayout>
CUTE_HOST_DEVICE
void
prefetch(Copy_Atom<Copy_Traits<CopyOp, CT_Args...>, CopyType> const& atom,
         Tensor<GEngine, GLayout>                             const& src)
{
  if constexpr (detail::has_prefetch<CopyOp>) {
    using Prefetch_Traits = Copy_Traits<typename CopyOp::PREFETCH, CT_Args...>;
    using Prefetch_Atom = Copy_Atom<Prefetch_Traits, CopyType>;
    Prefetch_Atom prefetch_atom{atom};
    //auto& dst = const_cast<Tensor<GEngine, GLayout>&>(src); // dst is ignored for prefetch atoms
    Tensor dst = make_tensor(make_smem_ptr<CopyType>(nullptr), shape(src));
    return copy(prefetch_atom, src, dst);
  } else {
    return prefetch(src);
  }
}
```
- **EN:** Defines `prefetch`, issuing a prefetch request so upcoming data reaches a closer cache level before it is consumed.
- **CN:** 定义 `prefetch`：发出预取请求，使即将访问的数据在被消费前进入更近的缓存层级。

### Lines 112-121
```cpp
#if defined(CUTE_COPY_ATOM_TMA_SM90_ENABLED)
template <class... CT_Args,
          class SrcEngine, class SrcLayout>
CUTE_HOST_DEVICE
void
prefetch(Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...> const& atom,
         Tensor<SrcEngine, SrcLayout>                 const& src)
{
  using SrcType = typename SrcEngine::value_type;
  static_assert(is_gmem<SrcEngine>::value, "Expected global tensor for L2 prefetch");
```
- **EN:** Defines `prefetch`, issuing a prefetch request so upcoming data reaches a closer cache level before it is consumed.
- **CN:** 定义 `prefetch`：发出预取请求，使即将访问的数据在被消费前进入更近的缓存层级。

### Lines 123-126
```cpp
  auto tiler = max_common_layout(src, src);
  constexpr int vec_elem = decltype(size(tiler))::value;
  constexpr int vec_bits = vec_elem * sizeof_bits_v<SrcType>;
  static_assert(vec_bits >= 128, "Expected at least 128-bits for BLKCP");
```
- **EN:** Defines or forwards `max_common_layout` as part of this header's executable interface.
- **CN:** 定义或转发 `max_common_layout`，作为该头文件可执行接口的一部分。

### Lines 128-129
```cpp
  // Construct a new concrete Atom of the vector size
  auto bulk_atom = Copy_Atom<Copy_Traits<SM90_BULK_COPY_G2S, Int<vec_bits>>, SrcType>{};
```
- **EN:** Implements a supporting fragment of the surrounding algorithm, trait, or architecture wrapper.
- **CN:** 实现周边算法、trait 或体系结构封装所需的辅助片段。

### Lines 131-132
```cpp
  return prefetch(bulk_atom, logical_divide(src, tiler));
}
```
- **EN:** Defines `prefetch`, issuing a prefetch request so upcoming data reaches a closer cache level before it is consumed.
- **CN:** 定义 `prefetch`：发出预取请求，使即将访问的数据在被消费前进入更近的缓存层级。

### Lines 134-144
```cpp
// Backwards-compat. Throw out any extra Copy_Atom args.
template <class... CT_Args, class... CA_Args,
          class SrcEngine, class SrcLayout>
CUTE_HOST_DEVICE
void
prefetch(Copy_Atom<Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...>, CA_Args...> const& atom,
         Tensor<SrcEngine, SrcLayout>                                        const& src)
{
  return prefetch(static_cast<Copy_Traits<SM90_BULK_COPY_AUTO, CT_Args...> const&>(atom), src);
}
#endif // #if defined(CUTE_COPY_ATOM_TMA_SM90_ENABLED)
```
- **EN:** Defines `prefetch`, issuing a prefetch request so upcoming data reaches a closer cache level before it is consumed.
- **CN:** 定义 `prefetch`：发出预取请求，使即将访问的数据在被消费前进入更近的缓存层级。

### Lines 146-146
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Tensor abstractions organize element access, shapes, and coordinate transforms.
  **CN:** 张量抽象负责组织元素访问、形状以及坐标变换。
- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/tensor_impl.hpp` supplies tensor storage, indexing, and layout implementation details.
  **CN:** `cute/tensor_impl.hpp` 提供了张量存储、索引与布局实现细节。
- **EN:** `cute/atom/copy_atom.hpp` supplies copy atoms that combine traits with tiled tensor views.
  **CN:** `cute/atom/copy_atom.hpp` 提供了把 traits 与分块张量视图结合起来的 copy atom。
