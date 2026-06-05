# default_epilogue.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/collective/default_epilogue.hpp`

- **Purpose (EN):** Functor performing elementwise operations used by epilogues.

- **作用 (CN):** 为特定 epilogue 后端提供默认的 `default epilogue` 组合规则。


## Line-by-Line Analysis / 逐行分析

### Lines 1-33

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
/*! \file
  \brief Functor performing elementwise operations used by epilogues.
*/
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Line 35

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 37-40

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/arch/memory.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/detail.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `cutlass/arch/memory.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/epilogue/collective/detail.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`cutlass/arch/memory.h`，`cutlass/gemm/dispatch_policy.hpp`，`cutlass/epilogue/collective/detail.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 42-44

```cpp
#include "cute/tensor.hpp"
#include "cute/numeric/numeric_types.hpp"
#include "cutlass/cuda_host_adapter.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/tensor.hpp`, `cute/numeric/numeric_types.hpp`, `cutlass/cuda_host_adapter.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/tensor.hpp`，`cute/numeric/numeric_types.hpp`，`cutlass/cuda_host_adapter.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 48-50

```cpp
namespace cutlass {
namespace epilogue {
namespace collective {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 54-69

```cpp
/// Applies an element wise operation to all elements within the fragment
/// and writes them out to destination storage.
template <
  class ElementC_,
  class StrideC_,
  class StrideD_,
  class ThreadEpilogueOp_,
  class EpilogueSchedule_
>
class DefaultEpilogue {
public:
  //
  // Type Aliases
  //
  using EpilogueSchedule = EpilogueSchedule_;
  using DispatchPolicy = EpilogueSchedule_;
```

**EN:** Declares the templated `ElementC_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Applies an element wise operation to all elements within the fragment and writes them out to destination storage.

**CN:** 声明模板类型 `ElementC_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 71-80

```cpp
  // derived types of output thread level operator
  using ThreadEpilogueOp = ThreadEpilogueOp_;
  using ElementOutput = typename ThreadEpilogueOp::ElementOutput;
  using ElementAccumulator = typename ThreadEpilogueOp::ElementAccumulator;
  using ElementCompute = typename ThreadEpilogueOp::ElementCompute;
  using ElementScalar = ElementCompute;
  using ElementC = ElementC_;
  using StrideC = StrideC_;
  using ElementD = typename ThreadEpilogueOp::ElementD;
  using StrideD = StrideD_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 82

```cpp
  using GmemElementC = cute::conditional_t<cute::is_void_v<ElementC>, ElementD, ElementC>; // prevents void ref breakages
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 84-85

```cpp
  using GmemTiledCopyC = void;
  using GmemTiledCopyD = void;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 87-88

```cpp
  static const int kOutputAlignment = ThreadEpilogueOp::kCount;
  using AlignmentType = typename cute::uint_bit<sizeof_bits<ElementOutput>::value * kOutputAlignment>::type;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 90-91

```cpp
  static_assert(cute::rank(StrideC{}) == 3, "StrideCD must be rank-3: [M, N, L]");
  static_assert(cute::rank(StrideD{}) == 3, "StrideCD must be rank-3: [M, N, L]");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Line 93

```cpp
  struct SharedStorage { };
```

**EN:** Defines `SharedStorage`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `SharedStorage`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Line 95

```cpp
  using TensorStorage = SharedStorage;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 97-104

```cpp
  // Host side epilogue arguments
  struct Arguments {
    typename ThreadEpilogueOp::Params thread{};
    ElementC const* ptr_C = nullptr;
    StrideC dC{};
    ElementD* ptr_D = nullptr;
    StrideD dD{};
  };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Host side epilogue arguments.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 106-107

```cpp
  // Device side epilogue params
  using Params = Arguments;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 113-120

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
      [[maybe_unused]] ProblemShape const& _,
      Arguments const& args,
      [[maybe_unused]] void* workspace) {
    return args;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 122-126

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args) {
    return 0;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 128-133

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream,
    CudaHostAdapter* cuda_adapter = nullptr) {
    return cutlass::Status::kSuccess;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 135-141

```cpp
  template<class ProblemShape>
  static bool
  can_implement(
      [[maybe_unused]] ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
    return true;
  }
```

**EN:** Defines `ProblemShape`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `ProblemShape`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 143-146

```cpp
  // Note: SharedStorage is unused for DefaultEpilogue
  CUTLASS_HOST_DEVICE
  DefaultEpilogue(Params const& params_, SharedStorage const& shared_storage = SharedStorage())
      : params(params_), epilogue_op(params_.thread) { }
```

**EN:** This method block implements `DefaultEpilogue`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `DefaultEpilogue`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 148-152

```cpp
  CUTLASS_DEVICE
  bool
  is_source_needed() {
    return epilogue_op.is_source_needed();
  }
```

**EN:** This helper method adjusts or queries runtime behavior so the epilogue can avoid unnecessary work for special scaling modes or split-K cases.

**CN:** 这个辅助方法用于调整或查询运行时行为，使 epilogue 在特殊缩放模式或 split-K 场景下避免不必要的工作。


### Lines 154-174

```cpp
  template<
    class ProblemShapeMNKL,
    class BlockShapeMNK,
    class BlockCoordMNKL,
    class FrgEngine, class FrgLayout,
    class TiledMma,
    class ResidueMNK
  >
  CUTLASS_DEVICE void
  operator()(
      ProblemShapeMNKL problem_shape_mnkl,
      BlockShapeMNK blk_shape_MNK,
      BlockCoordMNKL blk_coord_mnkl,
      cute::Tensor<FrgEngine, FrgLayout> const& accumulators,
      TiledMma tiled_mma,
      [[maybe_unused]] ResidueMNK,
      int thread_idx,
      [[maybe_unused]] char*)
  {
    using namespace cute;
    using X = Underscore;
```

**EN:** Defines `ProblemShapeMNKL`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `ProblemShapeMNKL`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 176-179

```cpp
    static_assert(cute::rank(ProblemShapeMNKL{}) == 4, "ProblemShapeMNKL must be rank 4");
    static_assert(is_static<BlockShapeMNK>::value, "ThreadBlock tile shape must be static");
    static_assert(cute::rank(BlockShapeMNK{}) == 3, "BlockShapeMNK must be rank 3");
    static_assert(cute::rank(BlockCoordMNKL{}) == 4, "BlockCoordMNKL must be rank 3");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 181-184

```cpp
    // Separate out problem shape for convenience
    auto M = get<0>(problem_shape_mnkl);
    auto N = get<1>(problem_shape_mnkl);
    auto L = get<3>(problem_shape_mnkl);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 186-187

```cpp
    auto stride_c = detail::get_epilogue_stride<EpilogueSchedule>(params.dC);
    auto stride_d = detail::get_epilogue_stride<EpilogueSchedule>(params.dD);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 189-193

```cpp
    // Represent the full output tensor
    Tensor mC_mnl = make_tensor(make_gmem_ptr<GmemElementC>(params.ptr_C), make_shape(M,N,L), stride_c);     // (m,n,l)
    Tensor mD_mnl = make_tensor(make_gmem_ptr(params.ptr_D), make_shape(M,N,L), stride_d);                   // (m,n,l)
    Tensor gC_mnl = local_tile(mC_mnl, blk_shape_MNK, make_coord(_,_,_), Step<_1,_1, X>{});      // (BLK_M,BLK_N,m,n,l)
    Tensor gD_mnl = local_tile(mD_mnl, blk_shape_MNK, make_coord(_,_,_), Step<_1,_1, X>{});      // (BLK_M,BLK_N,m,n,l)
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 195-198

```cpp
    // Slice to get the tile this CTA is responsible for
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord_mnkl;
    Tensor gC = gC_mnl(_,_,m_coord,n_coord,l_coord);                                                   // (BLK_M,BLK_N)
    Tensor gD = gD_mnl(_,_,m_coord,n_coord,l_coord);                                                   // (BLK_M,BLK_N)
```

**EN:** This method block implements `gC_mnl`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `gC_mnl`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 200-203

```cpp
    // Partition source and destination tiles to match the accumulator partitioning
    auto thr_mma = tiled_mma.get_thread_slice(thread_idx);
    Tensor tCgD = thr_mma.partition_C(gD);                                                         // (VEC,THR_M,THR_N)
    Tensor tCgC = thr_mma.partition_C(gC);                                                         // (VEC,THR_M,THR_N)
```

**EN:** This method block implements `get_thread_slice`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_thread_slice`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 205-209

```cpp
    static_assert(is_static<FrgLayout>::value, "Accumulator layout must be static");
    CUTE_STATIC_ASSERT_V(size(tCgC) == size(tCgD),
        "Source and destination must have the same number of elements.");
    CUTE_STATIC_ASSERT_V(size(tCgD) == size(accumulators),
        "Accumulator count must have the same destination element count.");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 211-222

```cpp
    // OOB predication for tile quantization "residue"
    // Absolute coordinate tensors (dynamic)
    auto shape_MN = make_shape(M,N);
    Tensor mD_crd = make_identity_tensor(shape_MN);                                                            // (M,N)
    Tensor cD_mn = local_tile(mD_crd, take<0,2>(blk_shape_MNK), make_coord(m_coord, n_coord));         // (BLK_M,BLK_N)
    Tensor tCcD_mn = thr_mma.partition_C(cD_mn);                                                   // (VEC,THR_M,THR_N)
    // Relative coordinate tensors (static)
    Tensor cD = make_coord_tensor(cD_mn.layout());                                                  // (BLK_M,BLK_N)
    Tensor tCcD = make_coord_tensor(tCcD_mn.layout());                                          // (VEC,THR_M,THR_N)
    // Subtract the global "bottom right" corner from the local "top left" corner to get the max relative coordinate
    auto residue_cD = shape_MN - cD_mn(_0{});                                                                  // (m,n)
    auto residue_tCcD = shape_MN - tCcD_mn(_0{});                                                              // (m,n)
```

**EN:** This method block implements `make_shape`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_shape`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 224-227

```cpp
    // Fully OOB tile
    if (not elem_less(repeat_like(residue_cD, _0{}), residue_cD)) {
      return;
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 229-230

```cpp
    using FragCType = remove_cvref_t<decltype(tCgC(0))>;
    using FragDType = remove_cvref_t<decltype(tCgD(0))>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 232-252

```cpp
    // source is needed
    if (epilogue_op.is_source_needed()) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(accumulators); ++i) {
        FragCType fragC;
        bool pred = elem_less(tCcD(i), residue_tCcD);
        arch::global_load<FragCType, sizeof(FragCType)>(fragC, &tCgC(i), pred);
        FragDType fragD = epilogue_op(accumulators(i), fragC);
        arch::global_store<FragDType, sizeof(FragDType)>(fragD, &tCgD(i), pred);
      }
    }
    // source is not needed, avoid load
    else {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(accumulators); ++i) {
        bool pred = elem_less(tCcD(i), residue_tCcD);
        FragDType fragD = epilogue_op(accumulators(i));
        arch::global_store<FragDType, sizeof(FragDType)>(fragD, &tCgD(i), pred);
      }
    }
  }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 254-257

```cpp
private:
  Params params;
  ThreadEpilogueOp epilogue_op;
};
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


## Key Concepts / 关键概念

- **Collective epilogue / Collective epilogue:** Coordinates epilogue work at the collective level, often tying dispatch policy, tile shape, and fusion callbacks together. / 在 collective 层面组织 epilogue 工作，通常把调度策略、tile 形状和融合回调连接在一起。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cutlass/arch/memory.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/epilogue/collective/detail.hpp`, `cute/tensor.hpp`, `cute/numeric/numeric_types.hpp`, `cutlass/cuda_host_adapter.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::collective`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/collective/detail.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`, `cute/numeric/numeric_types.hpp`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`
