# xe_visitor_softmax.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/xe_visitor_softmax.hpp`

- **Purpose (EN):** Visitor tree Softmax fusion operation for the Intel Xe epilogue.

- **作用 (CN):** 实现 `Xe visitor Softmax` 访问者逻辑，用于遍历融合后的 epilogue 操作树。


## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
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
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 33-35

```cpp
/*! \file
  \brief Visitor tree Softmax fusion operation for the Intel Xe epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Visitor tree Softmax fusion operation for the Intel Xe epilogue.

**CN:** 这一段文档注释说明了紧随其后的声明意图，帮助理解后续模板或实现要解决的问题。


### Line 37

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 39-40

```cpp
#include "cutlass/cutlass.h"
#include <sycl/sycl.hpp>
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `sycl/sycl.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`sycl/sycl.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 43

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Line 47

```cpp
namespace detail {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 49-53

```cpp
template <class STensor, uint32_t row, uint32_t SgN, class RTensor, class OutTensor>
CUTLASS_DEVICE
void group_reduce_sum_partial(STensor &stensor, RTensor &vec, OutTensor &out) {
  auto sg = compat::get_nd_item<1>().get_sub_group();
  auto group = compat::get_nd_item<1>().get_group();
```

**EN:** Declares the templated `STensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `STensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 55-58

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < size(vec); i++) {
    vec(i) = reduce_over_group(sg, vec(i), sycl::plus<>());    
  }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 60-62

```cpp
  auto sg_group_id = sg.get_group_id();
  auto sg_group_id_n = sg_group_id % SgN;
  auto sg_local_id = sg.get_local_id()[0];
```

**EN:** This method block implements `get_group_id`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_group_id`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 64-66

```cpp
  auto slm_base = stensor(_, _, sg_group_id / SgN);
  static constexpr auto step = SgN / IntelXeXMX16::SubgroupSize;
  static constexpr auto n_step = row / SgN;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 68-78

```cpp
  if constexpr (row < IntelXeXMX16::SubgroupSize) {
    if (sg_local_id < row) {
      slm_base(sg_local_id, sg_group_id_n) = vec(sg_local_id);
    }
  }
  else {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < step; i++) {
      slm_base(sg_local_id * step + i, sg_group_id_n) = vec(i + step * sg_local_id);
    }
  }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 80

```cpp
  sycl::group_barrier(group);
```

**EN:** This method block implements `group_barrier`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `group_barrier`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 82-83

```cpp
  Tensor s_slice = make_tensor(static_cast<decltype(slm_base) &&>(slm_base).data(), 
                        make_shape(Int<n_step>{}, Int<SgN>{}, Int<step>{}, Int<IntelXeXMX16::SubgroupSize>{}));
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 85-89

```cpp
  if constexpr (SgN <= row) {
    auto local_vec = s_slice(_, sg_group_id_n, _, sg_local_id);
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < n_step; i++) {
      auto sum = 0.f;
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 91-94

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < step; j++) {
        sum += local_vec(i, j);
      }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 96

```cpp
      auto group_sum = reduce_over_group(sg, sum, sycl::plus<>());
```

**EN:** This method block implements `reduce_over_group`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce_over_group`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 98-104

```cpp
      if (sg_local_id == i) {
        s_slice(i, sg_group_id_n, 0, 0) = group_sum;
      }
    }
  }
  else {
    auto sum = 0.f;
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 106-110

```cpp
    if (sg_group_id_n < row) {
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < step; j++) {
        sum += s_slice(0, sg_group_id_n, j, sg_local_id);
      }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 112

```cpp
      auto group_sum = reduce_over_group(sg, sum, sycl::plus<>());
```

**EN:** This method block implements `reduce_over_group`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce_over_group`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 114-119

```cpp
      if (sg_local_id == 0) {
        s_slice(0, sg_group_id_n,0, 0) = group_sum;
      }
    }
  }
  sycl::group_barrier(group);
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 121-125

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < row; i++) {
    out(i) = slm_base(i);
  }
}
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 127-131

```cpp
template <class STensor, uint32_t row, uint32_t SgN, class RTensor, class OutTensor>
CUTLASS_DEVICE
void group_reduce_max_partial(STensor &stensor, RTensor &vec, OutTensor &out) {
  auto sg = compat::get_nd_item<1>().get_sub_group();
  auto group = compat::get_nd_item<1>().get_group();
```

**EN:** Declares the templated `STensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `STensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 133-136

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < size(vec); i++) {
    vec(i) = reduce_over_group(sg, vec(i), sycl::maximum<>());    
  }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 138-140

```cpp
  auto sg_group_id = sg.get_group_id();
  auto sg_group_id_n = sg_group_id % SgN;
  auto sg_local_id = sg.get_local_id()[0];
```

**EN:** This method block implements `get_group_id`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_group_id`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 142-144

```cpp
  auto slm_base = stensor(_, _, sg_group_id / SgN);
  static constexpr auto step = SgN / IntelXeXMX16::SubgroupSize;
  static constexpr auto n_step = row / SgN;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 146-156

```cpp
  if constexpr (row < IntelXeXMX16::SubgroupSize) {
    if (sg_local_id < row) {
      slm_base(sg_local_id, sg_group_id_n) = vec(sg_local_id);
    }
  }
  else {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < step; i++) {
      slm_base(sg_local_id * step  + i, sg_group_id_n) = vec(i + step * sg_local_id);
    }
  }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 158

```cpp
  sycl::group_barrier(group);
```

**EN:** This method block implements `group_barrier`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `group_barrier`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 160-161

```cpp
  Tensor s_slice = make_tensor(static_cast<decltype(slm_base) &&>(slm_base).data(), 
                        make_shape(Int<n_step>{}, Int<SgN>{}, Int<step>{}, Int<IntelXeXMX16::SubgroupSize>{}));
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 163-164

```cpp
  if constexpr (SgN <= row) {
    auto local_vec = s_slice(_, sg_group_id_n, _, sg_local_id);
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 166-168

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < n_step; i++) {
      auto local_max = local_vec(i, 0);
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 170-173

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int j = 1; j < step; j++) {
        local_max = sycl::max(local_max, local_vec(i, j));
      }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 175

```cpp
      auto group_max = reduce_over_group(sg, local_max, sycl::maximum<>());
```

**EN:** This method block implements `reduce_over_group`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce_over_group`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 177-183

```cpp
      if (sg_local_id == i) {
        s_slice(i, sg_group_id_n, 0, 0) = group_max;
      }
    }
  } 
  else {
    auto local_max = s_slice(0, sg_group_id_n, 0, sg_local_id);
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 185-189

```cpp
    if (sg_group_id_n < row) {
      CUTLASS_PRAGMA_UNROLL
      for (int j = 1; j < step; j++) {
        local_max = sycl::max(local_max, s_slice(0, sg_group_id_n, j, sg_local_id));
      }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 191

```cpp
      auto group_max = reduce_over_group(sg, local_max, sycl::maximum<>());
```

**EN:** This method block implements `reduce_over_group`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce_over_group`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 193-198

```cpp
      if (sg_local_id == 0) {
        s_slice(0, sg_group_id_n,0, 0) = group_max;
      }
    }
  }
  sycl::group_barrier(group);
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 200-204

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < row; i++) {
    out(i) = slm_base(i);
  }
}
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 206-210

```cpp
template <uint32_t SgN, class STensor, class RTensor, class OutTensor>
CUTLASS_DEVICE
auto group_reduce_sum(STensor &stensor, RTensor const &rtensor, OutTensor &out) {
  static constexpr auto row = decltype(size<0>(rtensor))::value;
  static constexpr auto col = decltype(size<1>(rtensor))::value;
```

**EN:** Declares the templated `STensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `STensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 212

```cpp
  Tensor local_sum = make_tensor<float>(Shape<Int<row>>{});
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 214-217

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < row; i++) {
    local_sum(i) = rtensor(i, 0);
  }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 219-221

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 1; i < col; i++) {
    auto r_col = rtensor(_, i);
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 223-229

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < row; j++) {
      local_sum(j) += r_col(j);
    }
  }
  group_reduce_sum_partial<STensor, row, SgN>(stensor, local_sum, out);
}
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 231-235

```cpp
template <uint32_t SgN, class STensor, class RTensor, class OutTensor>
CUTLASS_DEVICE
auto group_reduce_max(STensor &stensor, RTensor const &rtensor, OutTensor &out) {
  static constexpr auto row = decltype(size<0>(rtensor))::value;
  static constexpr auto col = decltype(size<1>(rtensor))::value;
```

**EN:** Declares the templated `STensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `STensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 237

```cpp
  Tensor local_max = make_tensor<float>(Shape<Int<row>>{});
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 239-242

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 0; i < row; i++) {
    local_max(i) = rtensor(i, 0);
  }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 244-246

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int i = 1; i < col; i++) {
    auto r_col = rtensor(_, i);
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 248-254

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < row; j++) {
      local_max(j) = sycl::max(local_max(j), r_col(j));
    }
  }
  group_reduce_max_partial<STensor, row, SgN>(stensor, local_max, out);
}
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 258-276

```cpp
template <
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class CopyOpR2G,
  FloatRoundStyle RoundStyle
>
struct XeSoftmaxRowReduction
{
public:
  static constexpr int FragmentSize = 8;
  static constexpr auto Tile_M = get<0>(CtaTileShapeMNK{});
  static constexpr auto Tile_N = get<1>(CtaTileShapeMNK{});
  static constexpr auto Epi_M = get<0>(EpilogueTile{});
  static constexpr auto Epi_N = get<1>(EpilogueTile{});
  static constexpr auto Sg_M = Tile_M / Epi_M;
  static constexpr auto Sg_N = Tile_N / Epi_N;
  static constexpr auto Sg_Nums = Sg_M * Sg_N;
```

**EN:** Declares the templated `CtaTileShapeMNK` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `CtaTileShapeMNK`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 278-283

```cpp
  using Trait_Output = Copy_Traits<CopyOpR2G>;
  using XE_Copy_output = decltype(make_tiled_copy(Copy_Atom<Trait_Output, ElementOutput>{}
                                             .with(static_cast<ElementOutput const*>(nullptr),int32_t(0), int32_t(0)),
                                             Layout<Shape<_1, Int<IntelXeXMX16::SubgroupSize>>>{},
                                             make_layout(make_shape(get<0>(typename Trait_Output::BlockShape{}),
                                                                    get<1>(typename Trait_Output::BlockShape{}) / Int<IntelXeXMX16::SubgroupSize>{}))));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Line 285

```cpp
  struct SharedStorage { };
```

**EN:** Defines `SharedStorage`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `SharedStorage`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 287-290

```cpp
  struct Arguments {
    ElementOutput* ptr_output;
    // StrideOutput dOutput;
  };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 292-294

```cpp
  struct Params {
    XE_Copy_output xe_store_output;
  };
```

**EN:** Defines `Params`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Params`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 296-305

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shape, Arguments const& args, void* workspace) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
    XE_Copy_output output = make_tiled_copy(Copy_Atom<Copy_Traits<CopyOpR2G>, ElementOutput>{}.with(
                            args.ptr_output, M, N),
                            Layout<Shape<_1, Int<IntelXeXMX16::SubgroupSize>>>{},
                            make_layout(make_shape(get<0>(typename XE_Copy_output::BlockShape{}),
                                                   get<1>(typename XE_Copy_output::BlockShape{}) / Int<IntelXeXMX16::SubgroupSize>{})));
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 307-308

```cpp
    return {output};
  }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 310-321

```cpp
  template <class ProblemShape>
  static bool
  can_implement(ProblemShape const& problem_shape, Arguments const& args) {
    auto [M, N, K, L] = problem_shape;
    auto [tile_M, tile_N, tile_K] = CtaTileShapeMNK{};
    // Cross CTA reduction is not possible because there is no guarantee that all CTAs run
    // concurrently.
    // Cross epilogue tile reduction is possible, but re-visiting and applying reduction
    // to accumulators is only possible for the current epilogue tile.
    auto [epi_M, epi_N] = EpilogueTile{};
    return N <= tile_N;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 323-327

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args) {
    return 0;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 329-334

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream,
    CudaHostAdapter* cuda_adapter = nullptr) {
    return Status::kSuccess;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 336-339

```cpp
  CUTLASS_DEVICE bool
  is_producer_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_producer_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_producer_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 341-344

```cpp
  CUTLASS_DEVICE bool
  is_C_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_C_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_C_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 346-347

```cpp
  CUTLASS_HOST_DEVICE
  XeSoftmaxRowReduction() { }
```

**EN:** This method block implements `XeSoftmaxRowReduction`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `XeSoftmaxRowReduction`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 349-351

```cpp
  CUTLASS_HOST_DEVICE
  XeSoftmaxRowReduction(Params const& params, SharedStorage const& shared_storage)
      : params(params) { }
```

**EN:** This method block implements `XeSoftmaxRowReduction`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `XeSoftmaxRowReduction`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 353

```cpp
  Params params;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 355-359

```cpp
  template <class... Args>
  CUTLASS_DEVICE auto
  get_producer_load_callbacks(ProducerLoadArgs<Args...> const& args) {
    return EmptyProducerLoadCallbacks{};
  }
```

**EN:** This method block implements `get_producer_load_callbacks`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_producer_load_callbacks`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 361-362

```cpp
  template<class RTensor, class CoordTensor>
  struct ConsumerStoreCallbacks : EmptyConsumerStoreCallbacks {
```

**EN:** Defines `RTensor`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `RTensor`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 364-368

```cpp
    CUTLASS_DEVICE
    ConsumerStoreCallbacks(RTensor&& res_tensor, CoordTensor&& coord, Params const& params)
      : res_tensor(cute::forward<RTensor>(res_tensor)),
        coord(cute::forward<CoordTensor>(coord)),
        params(params) {}
```

**EN:** This method block implements `ConsumerStoreCallbacks`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ConsumerStoreCallbacks`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 370-376

```cpp
    RTensor res_tensor;
    CoordTensor coord;
    Params const& params;
    template <typename ElementInput, typename ElementAccumulator, int FragmentSize>
    CUTLASS_DEVICE auto
    visit(Array<ElementAccumulator, FragmentSize> const& frg_acc, int epi_v, int epi_m, int epi_n,
          Array<ElementInput, FragmentSize> const& frg_input) {
```

**EN:** This method block implements `visit`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `visit`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 378-379

```cpp
      return frg_acc;
    }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 381-387

```cpp
    template<class STensor, class SyncFn, class VTensor>
    CUTLASS_DEVICE void
    reduce(STensor&& smem_buffer, SyncFn const& sync_fn, int epi_m, int epi_n, bool is_last_iteration, VTensor visit_results) {
      if(is_last_iteration) {
      for(int epi_v = 0; epi_v < visit_results(0).size(); epi_v++) {
        res_tensor(epi_v, epi_m, epi_n) = visit_results(0)[epi_v];
      }
```

**EN:** Defines `STensor`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `STensor`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 389-390

```cpp
      constexpr auto vec_size = min(Epi_M, Sg_N);
      constexpr auto vec_folds = Epi_M / vec_size;
```

**EN:** This method block implements `min`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `min`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 392-393

```cpp
      auto smem = compat::local_mem<float[Sg_Nums * vec_size]>();
      Tensor stensor = make_tensor(make_smem_ptr(smem), make_shape(Int<vec_size>{}, Int<Sg_N>{}, Int<Sg_M>{}));
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 395-397

```cpp
      Tensor res =
          make_tensor(static_cast<decltype(res_tensor) &&>(res_tensor).data(),
                      make_shape(Int<vec_size>{}, Int<vec_folds>{}, Int<Epi_N / IntelXeXMX16::SubgroupSize>{}));
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 399-427

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int loop = 0; loop < vec_folds; loop++) {
        auto loop_t = res(_, loop, _);
        Tensor group_max = make_tensor<float>(make_shape(Int<vec_size>{}));
        group_reduce_max<Sg_N>(stensor, loop_t, group_max);
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < Epi_N / IntelXeXMX16::SubgroupSize; i++) {
          auto element_vec = loop_t(_, i);
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < vec_size; j++) {
            element_vec(j) -= group_max(j);
          }
        }
      }
      CUTLASS_PRAGMA_UNROLL
      for (int loop = 0; loop < vec_folds; loop++) {
        auto loop_t = res(_, loop, _);
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < Epi_N / IntelXeXMX16::SubgroupSize; i++) {
          auto exp_vec = loop_t(_, i);
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < vec_size; j++) {
            exp_vec(j) = sycl::native::exp(exp_vec(j));
          }
        }
      }
      CUTLASS_PRAGMA_UNROLL
      for (int loop = 0; loop < vec_folds; loop++) {
        auto loop_t = res(_, loop, _);
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 429-430

```cpp
        Tensor group_sum = make_tensor<float>(make_shape(Int<vec_size>{}));
        group_reduce_sum<Sg_N>(stensor, loop_t, group_sum);
```

**EN:** This method block implements `make_shape`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_shape`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 432-440

```cpp
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < Epi_N / IntelXeXMX16::SubgroupSize; i++) {
          auto softmax_vec = loop_t(_, i);
          CUTLASS_PRAGMA_UNROLL
          for (int j = 0; j < vec_size; j++) {
            softmax_vec(j) = sycl::native::divide(softmax_vec(j), group_sum(j));
          }
        }
      }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 442-450

```cpp
      copy(params.xe_store_output, res_tensor, coord);
    }
    else {
      for(int epi_v = 0; epi_v < visit_results(0).size(); epi_v++) {
        res_tensor(epi_v, epi_m, epi_n) = visit_results(0)[epi_v];
      }
    }
    }
  };
```

**EN:** This method block implements `copy`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `copy`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 452-461

```cpp
  template <
  bool ReferenceSrc, // do register tensors reference the src or dst layout of the tiled copy
  class... Args
  >
  CUTLASS_DEVICE auto
  get_consumer_store_callbacks(ConsumerStoreArgs<Args...> const& args) {
    using MmaAtomShape = typename decltype(args.tiled_mma)::AtomShape_MNK;
    static constexpr int FragsM = get<0>(EpilogueTile{}) / get<0>(MmaAtomShape()); // A frags per sub_group
    static constexpr int FragsN = get<1>(EpilogueTile{}) / get<1>(MmaAtomShape()); // B frags per sub_group
    Tensor res = make_tensor<ElementOutput>(Shape<Int<FragmentSize>, Int<FragsM>, Int<FragsN>>{});
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 463-468

```cpp
    auto [sg_m_coord, sg_n_coord, k_coord, l_offset] = args.tile_coord_mnkl;
    auto [M, N, K, L] = args.problem_shape_mnkl;
    Tensor mAux_mnl = cute::get_xe_tensor(make_shape(M,N,L));
    // Tiling is done differently than in epilogue as we get in coordinates of subgroup in kernel
    Tensor gAux = local_tile(mAux_mnl, select<0,1>(EpilogueTile{}), make_coord(sg_m_coord,sg_n_coord,l_offset));
    Tensor tCgAux = args.tiled_copy.get_thread_slice(args.thread_idx).partition_D(gAux);
```

**EN:** This method block implements `get_xe_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_xe_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 470-474

```cpp
    return ConsumerStoreCallbacks<decltype(res),decltype(tCgAux)>(
      cute::move(res), 
      cute::move(tCgAux),
      params);
  }
```

**EN:** This method block implements `decltype`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `decltype`，为外围 epilogue 组件提供一个聚焦的行为片段。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Post-ops / 后处理算子:** Fuses nonlinear activation or other elementwise transforms directly into the epilogue. / 把非线性激活或其他逐元素变换直接融合进 epilogue。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `sycl/sycl.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`
