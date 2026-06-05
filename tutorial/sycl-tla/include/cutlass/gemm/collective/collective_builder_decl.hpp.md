# collective_builder_decl.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/collective_builder_decl.hpp`
- **Purpose (EN):** Declares stage-count tags, the automatic kernel-schedule tag, and the primary `CollectiveBuilder` template.
- **用途 (CN):** 声明阶段数标签、自动内核调度标签，以及主 `CollectiveBuilder` 模板。
- **Lines / 行数:** 100

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
**EN:** This opening block combines the license banner with `#pragma once`, documenting legal terms and making the header safe to include multiple times.
**CN:** 这个开头块同时包含许可证说明和 `#pragma once`：前者给出法律信息，后者保证头文件可被重复包含而不重复定义。

### Lines 33-34

```cpp
#include <cute/numeric/integral_constant.hpp>
#include <cutlass/detail/dependent_false.hpp>
```
**EN:** This include block imports integral_constant.hpp, dependent_false.hpp. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 integral_constant.hpp、dependent_false.hpp。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 36-36

```cpp
namespace cutlass::gemm::collective {
```
**EN:** This line opens the `namespace cutlass::gemm::collective` scope so the following declarations live in the intended namespace.
**CN:** 这一行打开 `namespace cutlass::gemm::collective` 作用域，使后续声明位于预期命名空间中。

### Lines 38-38

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 40-43

```cpp
// Used to specify stage counts or dispatch to automatic computation of stage count
template<int num_stages>
struct StageCount {
  static constexpr int value = num_stages;
```
**EN:** This template defines an explicit stage-count tag. The builder reads `value` as a compile-time knob when selecting a mainloop configuration.
**CN:** 这个模板定义了显式阶段数标签。builder 会把 `value` 当作编译期旋钮，用于选择主循环配置。

### Lines 45-47

```cpp
  StageCount() = default;
  explicit StageCount(cute::Int<num_stages>) {}
};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 49-51

```cpp
template<int carveout_bytes>
struct StageCountAutoCarveout {
  static constexpr int bytes = carveout_bytes;
```
**EN:** This template represents an automatic stage-count policy parameterized by a shared-memory carve-out budget.
**CN:** 这个模板表示一种自动阶段数策略，并以共享内存 carve-out 预算作为参数。

### Lines 53-55

```cpp
  StageCountAutoCarveout() = default;
  explicit StageCountAutoCarveout(cute::Int<carveout_bytes>) {}
};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 57-57

```cpp
namespace detail {
```
**EN:** This line opens the `namespace detail` scope so the following declarations live in the intended namespace.
**CN:** 这一行打开 `namespace detail` 作用域，使后续声明位于预期命名空间中。

### Lines 59-62

```cpp
// Forward Declaration
template<class CollectiveEpilogue>
constexpr int
compute_carveout_from_epi();
```
**EN:** This block introduces `CollectiveEpilogue` and groups related declarations around that symbol.
**CN:** 这一段引入了 `CollectiveEpilogue`，并围绕该符号组织相关声明。

### Lines 64-64

```cpp
} // namespace detail
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 66-67

```cpp
template<class CollectiveEpilogue>
struct StageCountAutoCarveoutEpi : StageCountAutoCarveout<detail::compute_carveout_from_epi<CollectiveEpilogue>()> {};
```
**EN:** This alias ties epilogue carve-out computation to stage-count selection, letting the builder derive shared-memory pressure from the epilogue type.
**CN:** 这个别名把 epilogue 的 carve-out 计算接入阶段数选择逻辑，使 builder 能从 epilogue 类型推导共享内存压力。

### Lines 69-69

```cpp
using StageCountAuto = StageCountAutoCarveout<0>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 71-73

```cpp
// Used to automatically let the builder pick the kernel schedule.
// Can be overridden with kernel schedule tags in cutlass/gemm/dispatch_policy.hpp
struct KernelScheduleAuto final {};
```
**EN:** This tag requests automatic kernel-schedule selection unless a more specific schedule tag overrides it.
**CN:** 这个标签表示优先使用自动内核调度选择，除非外部提供了更具体的调度标签。

### Lines 75-75

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 77-95

```cpp
template <
  class ArchTag,
  class OpClass,
  class ElementA,
  class GmemLayoutA,
  int AlignmentA,
  class ElementB,
  class GmemLayoutB,
  int AlignmentB,
  class ElementAccumulator,
  class TileShape_MNK,
  class ClusterShape_MNK,
  class StageCountType,
  class KernelScheduleType,
  class Enable = void
>
struct CollectiveBuilder {
  static_assert(sizeof(ElementA) == 0, "Could not build a collective for given parameters.");
};
```
**EN:** This template block declares or specializes `CollectiveBuilder`, the compile-time factory that maps GEMM traits to a concrete collective.
**CN:** 这个模板块声明或特化了 `CollectiveBuilder`，它是把 GEMM 特征映射为具体 collective 的编译期工厂。

### Lines 97-97

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 99-99

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

## Key Concepts / 关键概念

- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** compile-time validation  
  **CN:** 编译期校验

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cute/numeric/integral_constant.hpp`
  - `cutlass/detail/dependent_false.hpp`
- **Primary symbols / 主要符号:**
  - `StageCount`
  - `StageCountAutoCarveout`
  - `CollectiveEpilogue`
  - `StageCountAutoCarveoutEpi`
  - `KernelScheduleAuto`
  - `ArchTag`
  - `OpClass`
  - `ElementA`
  - `GmemLayoutA`
  - `ElementB`
  - `GmemLayoutB`
  - `ElementAccumulator`
  - `TileShape_MNK`
  - `ClusterShape_MNK`
  - `StageCountType`
  - `KernelScheduleType`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。