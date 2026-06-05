# xe_epilogue.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/collective/xe_epilogue.hpp`

- **Purpose (EN):** Implements the `Xe epilogue` epilogue pipeline component in `collective`.

- **作用 (CN):** 实现 `Xe epilogue` 这一 epilogue 流水线组件。


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


### Line 33

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 35-44

```cpp
#include <sycl/sycl.hpp>
#include "cutlass/cutlass.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/epilogue/collective/collective_epilogue.hpp"
#include "cutlass/epilogue/collective/detail.hpp"
#include "cutlass/epilogue/fusion/callbacks.hpp"
#include "cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp"
#include "cutlass/detail/layout.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `sycl/sycl.hpp`, `cutlass/cutlass.h`, `cutlass/numeric_conversion.h`, `cutlass/epilogue/dispatch_policy.hpp`, and 6 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `sycl/sycl.hpp`，`cutlass/cutlass.h`，`cutlass/numeric_conversion.h`，`cutlass/epilogue/dispatch_policy.hpp`，以及另外 6 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 46

```cpp
#include "cute/tensor.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cute/tensor.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cute/tensor.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 50-52

```cpp
namespace cutlass {
namespace epilogue {
namespace collective {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 56-83

```cpp
template <
  class WGTileMNK_,
  class EpilogueTile_,
  class ElementC_,
  class StrideC_,
  class ElementD_,
  class StrideD_,
  class FusionCallbacks_,
  class CopyOpG2R_,
  class CopyOpR2G_
>
class CollectiveEpilogue<
    IntelXeGeneric,
    WGTileMNK_,
    EpilogueTile_,
    ElementC_,
    StrideC_,
    ElementD_,
    StrideD_,
    FusionCallbacks_,
    CopyOpG2R_,
    CopyOpR2G_
> {
public:
  //
  // Type Aliases
  //
  using DispatchPolicy = IntelXeXMX16;
```

**EN:** Declares the templated `WGTileMNK_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `WGTileMNK_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 85-92

```cpp
  using WGTileMNK = WGTileMNK_;
  using ElementC = ElementC_;
  using StrideC = StrideC_;
  using ElementD = ElementD_;
  using StrideD = StrideD_;
  using FusionCallbacks = FusionCallbacks_;
  using CopyOpG2R = CopyOpG2R_;
  using CopyOpR2G = CopyOpR2G_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 94

```cpp
  using NonVoidElementC = replace_void_t<ElementC, ElementD>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 96-98

```cpp
  using ThreadEpilogueOp = typename fusion::FusionCallbacksTraits<FusionCallbacks>::Operation;
  using ElementCompute = typename ThreadEpilogueOp::ElementCompute;
  using ElementOutput = ElementD;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 100-101

```cpp
  static constexpr int CopyBitsC = cute::min(sizeof(NonVoidElementC) * 8, 64);
  static constexpr int CopyBitsD = cute::min(sizeof(ElementD) * 8, 64);
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 103-108

```cpp
  // NOTE: GmemTiledCopy* may not be the actual C/D copy operations. They are declared here only so
  //         that GemmUniversalAdapter can inspect their alignment requirements.
  //       The real C/D copy operations are deduced inside operator() once we have access to
  //         the TiledMMA.
  using GmemTiledCopyC = replace_void_t<CopyOpG2R,  XE_LOAD_2D<CopyBitsC, 8, 512 / CopyBitsC>>;
  using GmemTiledCopyD = replace_void_t<CopyOpR2G, XE_STORE_2D<CopyBitsD, 8, 512 / CopyBitsD>>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 110

```cpp
  static constexpr int SubgroupSize = DispatchPolicy::SubgroupSize;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 112-114

```cpp
  static_assert(cute::rank(WGTileMNK{}) == 3, "WGTileMNK must be rank-3: [M, N, K]");
  static_assert(cute::rank(StrideC{}) == 3, "StrideC must be rank-3: [M, N, L]");
  static_assert(cute::rank(StrideD{}) == 3, "StrideD must be rank-3: [M, N, L]");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 116-117

```cpp
  using TensorC = decltype(make_tensor(make_gmem_ptr(static_cast<NonVoidElementC const*>(nullptr)),
                                       Layout<Shape<int,int,int>, StrideC>{}));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 119-120

```cpp
  using TensorD = decltype(make_tensor(make_gmem_ptr(static_cast<ElementD*>(nullptr)),
                                       Layout<Shape<int,int,int>, StrideD>{}));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 122-124

```cpp
private:
  constexpr static bool is_source_supported      = !is_void_v<ElementC>;
  constexpr static bool is_destination_supported = !is_void_v<ElementD>;
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


### Lines 126-130

```cpp
  // SFINAE: detect FusionCallbacks that provide is_identity()
  template <class T, class = void>
  struct has_is_identity : cute::false_type {};
  template <class T>
  struct has_is_identity<T, cute::void_t<decltype(cute::declval<T const&>().is_identity())>> : cute::true_type {};
```

**EN:** Declares the templated `T` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: SFINAE: detect FusionCallbacks that provide is_identity().

**CN:** 声明模板类型 `T`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 132-137

```cpp
public:
  struct SharedStorage {
    using FusionSharedStorage = typename FusionCallbacks::SharedStorage;
    FusionSharedStorage thread;
  };
  using TensorStorage = SharedStorage;    // Compatibility with legacy epilogues
```

**EN:** This access-specifier block switches the following declarations into the `public` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `public` 区域。


### Lines 139-146

```cpp
  // Host side epilogue arguments
  struct Arguments {
    typename FusionCallbacks::Arguments thread{};
    ElementC const* ptr_C;
    StrideC dC;
    ElementD* ptr_D;
    StrideD dD;
  };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Host side epilogue arguments.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 148-153

```cpp
  // Device side epilogue params
  struct Params {
    typename FusionCallbacks::Params thread{};
    TensorC mC;
    TensorD mD;
  };
```

**EN:** Defines `Params`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Device side epilogue params.

**CN:** 定义 `Params`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 159-164

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
      ProblemShape const& problem_shape,
      Arguments const& args,
      [[maybe_unused]] void* workspace) {
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 166-168

```cpp
    // Optionally append 1s until problem shape is rank-4, in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto shape_CD = select<0,1,3>(problem_shape_MNKL);        // (M,N,L)
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 170-173

```cpp
    // Create C/D tensors here; delay TiledCopy creation to the kernel.
    auto non_void_ptr_C = reinterpret_cast<const NonVoidElementC*>(args.ptr_C);
    auto mC = make_tensor(make_gmem_ptr(non_void_ptr_C), make_layout(shape_CD, args.dC));
    auto mD = make_tensor(make_gmem_ptr(args.ptr_D),     make_layout(shape_CD, args.dD));
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 175-180

```cpp
    return {
      FusionCallbacks::to_underlying_arguments(problem_shape, args.thread, workspace),
      mC,
      mD,
    };
  }
```

**EN:** This method block implements `to_underlying_arguments`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `to_underlying_arguments`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 182-186

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args) {
    return 0;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 188-193

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


### Lines 195-199

```cpp
  template <class ProblemShape>
  CUTLASS_HOST_DEVICE static bool
  can_implement(
      ProblemShape const& problem_shapes,
      Arguments const& args) {
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 201-205

```cpp
    // TODO: all these checks should be pushed down to individual copy atoms
    constexpr int copy_alignment_bits = 128;
    constexpr int batch_alignment_bits = 512;
    auto problem_shape_MNKL = append<4>(problem_shapes, 1);
    auto [M,N,K,L] = problem_shape_MNKL;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 207-208

```cpp
    bool implementable = true;
    bool fusion_implementable = true;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 210-217

```cpp
    if constexpr (is_destination_supported) {
      constexpr int min_aligned_elements_D = copy_alignment_bits / sizeof_bits<ElementD>::value;
      implementable &= cutlass::detail::check_alignment<min_aligned_elements_D>(cute::make_shape(M,N,L), args.dD);
      if (L > 1) {
        constexpr int min_batch_aligned_elements_D = batch_alignment_bits / sizeof_bits<ElementD>::value;
        implementable &= get<2>(args.dD) % min_batch_aligned_elements_D == 0;
      }
    }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 219-226

```cpp
    if constexpr (is_source_supported) {
      constexpr int min_aligned_elements_C = copy_alignment_bits / sizeof_bits<ElementC>::value;
      implementable &= cutlass::detail::check_alignment<min_aligned_elements_C>(cute::make_shape(M,N,L), args.dC);
      if (L > 1) {
        constexpr int min_batch_aligned_elements_C = batch_alignment_bits / sizeof_bits<ElementC>::value;
        implementable &= get<2>(args.dC) % min_batch_aligned_elements_C == 0;
      }
    }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 228

```cpp
    fusion_implementable = fusion_implementable && FusionCallbacks::can_implement(problem_shape_MNKL, args.thread);
```

**EN:** This method block implements `can_implement`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `can_implement`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 230-232

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem size doesn't meet the minimum alignment requirements for Xe 2D copy.\n");
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 234-236

```cpp
    if (!fusion_implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem size doesn't meet the minimum requirements for FusionCallbacks.\n");
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 238-239

```cpp
    return implementable && fusion_implementable;
  }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 241-243

```cpp
  CUTLASS_HOST_DEVICE
  CollectiveEpilogue(Params const& params_, SharedStorage const& shared_storage_)
      : params(params_), fusion_callbacks(params_.thread, shared_storage_.thread) {}
```

**EN:** This method block implements `CollectiveEpilogue`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `CollectiveEpilogue`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 245-249

```cpp
  CUTLASS_DEVICE
  bool
  is_producer_load_needed() const {
    return fusion_callbacks.is_producer_load_needed();
  }
```

**EN:** This method block implements `is_producer_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_producer_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 251-265

```cpp
  template<
    class ProblemShapeMNKL,
    class TileShapeMNK, /* compatibility with legacy epilogues */
    class TileCoordMNKL,
    class Accumulator,
    class TiledMMA
  >
  CUTLASS_DEVICE void
  operator() (
      ProblemShapeMNKL problem_shape_mnkl,
      TileShapeMNK,     /* compatibility with legacy epilogues */
      TileCoordMNKL tile_coord_mnkl,
      Accumulator accumulators,
      TiledMMA,
      int thread_idx) {
```

**EN:** Defines `ProblemShapeMNKL`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `ProblemShapeMNKL`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Line 267

```cpp
    using namespace cute;
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Line 269

```cpp
    using MMATile = decltype(take<0,2>(typename TiledMMA::AtomShape_MNK{}));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 271-274

```cpp
    static constexpr int EpiRPreferred = 8;
    static constexpr int EpiCPreferred = 512 / cute::min(sizeof_bits_v<NonVoidElementC>, sizeof_bits_v<ElementD>);    // 1 cache line
    static constexpr int EpiR = cute::gcd(EpiRPreferred, get<0>(MMATile{}));
    static constexpr int EpiC = cute::gcd(EpiCPreferred, get<1>(MMATile{}));
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 276-279

```cpp
    using DefaultEpilogueTile = Shape<Int<EpiR>, Int<EpiC>>;
    using EpilogueTile = conditional_t<is_void_v<EpilogueTile_> || is_same_v<EpilogueTile_, EpilogueTileAuto>,
                                       DefaultEpilogueTile,
                                       EpilogueTile_>;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 281-292

```cpp
    // Check if C is in column-major layout or not
    constexpr bool IsColMajorC = cutlass::gemm::detail::is_major<0, StrideC>();
    // Actual transpose load supports either 32-bit or 64-bit data element size only
    static constexpr int CopyBitsCTranspose = cute::max(CopyBitsC, 32);
    // For sub-32-bit data types, calculate the number of elements packed into 32-bits
    static constexpr int Sub32BitFactor = CopyBitsCTranspose / CopyBitsC;
    // Get copy atom operations for non-transposed and transposed load respectively
    using DefaultCopyOpG2RNonTranspose =  XE_LOAD_2D<CopyBitsC, cute::gcd(8, get<0>(EpilogueTile{})), cute::gcd(512 / CopyBitsC, get<1>(EpilogueTile{}))>;
    using DefaultCopyOpG2RTranspose = XE_LOAD_2D_TRANSPOSE<CopyBitsCTranspose, cute::gcd(512 / CopyBitsC, get<1>(EpilogueTile{})), cute::gcd(8 / Sub32BitFactor, get<0>(EpilogueTile{}))>;
    // Use transpose load if C is in column-major layout
    // Use non-transpose load for C otherwise
    using DefaultCopyOpG2R = conditional_t<IsColMajorC, DefaultCopyOpG2RTranspose, DefaultCopyOpG2RNonTranspose>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 294

```cpp
    using DefaultCopyOpR2G = XE_STORE_2D<CopyBitsD, cute::gcd(8, get<0>(EpilogueTile{})), cute::gcd(512 / CopyBitsD, get<1>(EpilogueTile{}))>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 296-297

```cpp
    using ActualGmemTiledCopyC = replace_void_t<CopyOpG2R, DefaultCopyOpG2R>;
    using ActualGmemTiledCopyD = replace_void_t<CopyOpR2G, DefaultCopyOpR2G>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 299

```cpp
    auto batch_idx = get<3>(tile_coord_mnkl);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 301-303

```cpp
    auto MN = take<0,2>(problem_shape_mnkl);
    auto cCD = make_identity_tensor(MN);                                                // (m,n)
    auto gCD = local_tile(cCD, take<0,2>(WGTileMNK{}), take<0,2>(tile_coord_mnkl));     // (m_in_wg_tile, n_in_wg_tile)
```

**EN:** This method block implements `make_identity_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_identity_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 305-306

```cpp
    auto thr_mma = TiledMMA{}.get_slice(thread_idx);
    auto tCDgCD = thr_mma.partition_C(gCD);                                             // (mma_v,mma_m,mma_n) -> coord
```

**EN:** This method block implements `get_slice`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_slice`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 308-312

```cpp
    // Tile accumulator into epilogue tiles.
    auto mma_per_epi = shape_div(EpilogueTile{}, MMATile{});
    auto tiled_acc_layout = group<0,3>(prepend(flat_divide(remove<0>(accumulators.layout()), mma_per_epi),
                                               get<0>(accumulators.layout())));
    auto tiled_acc = make_tensor(accumulators.data(), tiled_acc_layout);                // ((mma_v,mma_m,mma_n),epi_m,epi_n)
```

**EN:** This method block implements `shape_div`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `shape_div`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 314-316

```cpp
    // Tile subgroup's TV coord layout into epilogue tiles.
    auto sg_v_coord = prepend(flat_divide(remove<0>(tCDgCD.layout()), mma_per_epi),
                              get<0>(tCDgCD.layout()));                                 // (mma_v,mma_m,mma_n,epi_m,epi_n) -> coord
```

**EN:** This method block implements `prepend`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `prepend`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 318-321

```cpp
    // Prepare D store copy objects (needed by both identity and full paths).
    auto copy_d = make_block_2d_copy(ActualGmemTiledCopyD{}, params.mD(_,_,batch_idx));
    int wi_idx = thread_idx % intel::sg_size;
    auto thr_copy_d = copy_d.get_slice(wi_idx);
```

**EN:** This method block implements `make_block_2d_copy`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_block_2d_copy`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 323-327

```cpp
    // Partition global coordinate tensors into epilogue tiles,
    // matching the work-division from the TiledMMA.
    auto gCD_epi_layout = append(append(make_identity_layout(EpilogueTile{}),
                                        get<3>(sg_v_coord)), get<4>(sg_v_coord));
    auto gCD_epi = make_tensor(tCDgCD.data(), gCD_epi_layout);                          // (m,n,epi_m,epi_n) -> coord
```

**EN:** This method block implements `append`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `append`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 329

```cpp
    auto tDgD = thr_copy_d.partition_D(gCD_epi);                                        // (atom_v,atom_m,atom_n,epi_m,epi_n)
```

**EN:** This method block implements `partition_D`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `partition_D`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 331

```cpp
    auto tDrD = thr_copy_d.partition_sg_fragment_S(gCD_epi(_,_,0,0));                   // (atom_v,atom_m,atom_n)
```

**EN:** This method block implements `partition_sg_fragment_S`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `partition_sg_fragment_S`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 333-336

```cpp
    // Accumulator TV layout for reorder operations.
    using AccTVLayout = decltype(thr_mma.partition_sg_fragment_C(gCD).tv_layout());
    auto cd_compute_tv = make_layout(get<0>(AccTVLayout{}),
                                     sg_v_coord(_,_,_,_0{},_0{}));
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 338-339

```cpp
    constexpr auto EpiTilesM = size<2>(gCD_epi);
    constexpr auto EpiTilesN = size<3>(gCD_epi);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 341-349

```cpp
    //
    // Identity fast path: when the epilogue is D = Acc (alpha == 1, beta == 0),
    // skip all callback overhead and directly reorder + store.
    // reorder() handles both type conversion and layout transformation.
    //
    bool is_identity_epilogue = false;
    if constexpr (has_is_identity<FusionCallbacks>::value) {
      is_identity_epilogue = fusion_callbacks.is_identity();
    }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 351-367

```cpp
    if (is_identity_epilogue) {
      CUTLASS_PRAGMA_UNROLL
      for (int epi_m = 0; epi_m < EpiTilesM; epi_m++) {
        CUTLASS_PRAGMA_UNROLL
        for (int epi_n = 0; epi_n < EpiTilesN; epi_n++) {
          auto acc_epi_wi = make_tensor(tiled_acc(_,epi_m,epi_n).data(), tiled_acc(_,_0{},_0{}).layout());
          auto acc_epi = make_subgroup_tensor(acc_epi_wi, cd_compute_tv);
          if constexpr (is_destination_supported) {
            reorder(acc_epi, tDrD);
            copy(copy_d, tDrD, tDgD(_,_,_,epi_m,epi_n));
          }
        }
      }
    } else {
      //
      // Full epilogue path: C load, fusion callbacks, compute, and D store.
      //
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 369

```cpp
      bool is_C_load_needed = is_source_supported && fusion_callbacks.is_C_load_needed();
```

**EN:** This method block implements `is_C_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_C_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 371-375

```cpp
      // Prepare C load copy objects.
      auto copy_c = make_block_2d_copy(ActualGmemTiledCopyC{}, params.mC(_,_,batch_idx));
      auto thr_copy_c = copy_c.get_slice(wi_idx);
      auto tCgC = thr_copy_c.partition_S(gCD_epi);                                        // (atom_v,atom_m,atom_n,epi_m,epi_n)
      auto tCrC = thr_copy_c.partition_sg_fragment_D(gCD_epi(_,_,0,0));                   // (atom_v,atom_m,atom_n)
```

**EN:** This method block implements `make_block_2d_copy`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_block_2d_copy`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 377-378

```cpp
      auto tCrC_compute_wi = make_fragment_like<NonVoidElementC>(tiled_acc(_,_0{},_0{}));
      auto tCrC_compute = make_subgroup_tensor(tCrC_compute_wi, cd_compute_tv);           // (mma_v,mma_m,mma_n)
```

**EN:** This method block implements `tiled_acc`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `tiled_acc`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 380-382

```cpp
      // Calculate residues for boundary checks.
      auto residue_gCD    = MN - gCD(_0{});                                               // (res_m, res_n)
      auto residue_tCDgCD = MN - tCDgCD(_0{});                                            // (res_m, res_n)
```

**EN:** This method block implements `gCD`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `gCD`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 384-400

```cpp
      // Set up fusion visitor callbacks.
      constexpr bool RefSrc = true;
      auto cst_args = cutlass::epilogue::fusion::detail::ConsumerStoreArgs {
          problem_shape_mnkl,
          WGTileMNK{},
          tile_coord_mnkl,
          TiledMMA{},
          EpilogueTile{},
          copy_d,
          gCD,
          residue_gCD,
          tDgD,
          residue_tCDgCD,
          tCrC_compute,
          thread_idx,
      };
      auto cst_callbacks = fusion_callbacks.template get_consumer_store_callbacks<RefSrc>(cst_args);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 402-405

```cpp
      // Recast accumulator as arrays for vectorized visitor operations.
      using ElementAccumulator = typename Accumulator::element_type;
      constexpr int ComputeVectorLen = size<0>(Accumulator{});
      auto tiled_acc_v = recast<Array<ElementAccumulator, ComputeVectorLen>>(tiled_acc);
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 407-412

```cpp
      // Create D subgroup fragments for epilogue compute.
      using FragmentVisit = decltype(cst_callbacks.visit(tiled_acc_v(0), 0, 0, 0));
      using ElementVisit = typename FragmentVisit::Element;
      auto tDrD_compute_wi = make_fragment_like<ElementVisit>(tiled_acc(_,_0{},_0{}));
      auto tDrD_compute = make_subgroup_tensor(tDrD_compute_wi, cd_compute_tv);           // (mma_v,mma_m,mma_n)
      auto tDrD_compute_v = recast<FragmentVisit>(tDrD_compute_wi);
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 414-415

```cpp
      // Epilogue tile loops with fusion callbacks.
      cst_callbacks.begin();
```

**EN:** This method block implements `begin`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `begin`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 417-421

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int epi_m = 0; epi_m < EpiTilesM; epi_m++) {
        CUTLASS_PRAGMA_UNROLL
        for (int epi_n = 0; epi_n < EpiTilesN; epi_n++) {
          cst_callbacks.begin_loop(epi_m, epi_n);
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 423-429

```cpp
          // Load C and reorder to compute layout.
          if constexpr (is_source_supported) {
            if (is_C_load_needed) {
              copy(copy_c, tCgC(_,_,_,epi_m,epi_n), tCrC);
              reorder(tCrC, tCrC_compute);
            }
          }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 431

```cpp
          cst_callbacks.previsit(epi_m, epi_n, 0, is_C_load_needed);
```

**EN:** This method block implements `previsit`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `previsit`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 433-438

```cpp
          // Apply epilogue compute via visitor, one vector at a time.
          CUTLASS_PRAGMA_UNROLL
          for (int epi_v = 0; epi_v < size<0>(tiled_acc_v); ++epi_v) {
            tDrD_compute_v(epi_v) = cst_callbacks.visit(tiled_acc_v(epi_v, epi_m, epi_n),
                                                        epi_v, epi_m, epi_n);
          }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 440-441

```cpp
          bool last_epi = (epi_m == EpiTilesM - 1) && (epi_n == EpiTilesN - 1);
          cst_callbacks.reduce(nullptr, [=]{}, epi_m, epi_n, last_epi, tDrD_compute_v);
```

**EN:** This method block implements `reduce`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `reduce`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 443-447

```cpp
          // Reorder D (type conversion + layout) and store.
          if constexpr (is_destination_supported) {
            reorder(tDrD_compute, tDrD);
            copy(copy_d, tDrD, tDgD(_,_,_,epi_m,epi_n));
          }
```

**EN:** This method block implements `constexpr`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `constexpr`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 449-451

```cpp
          cst_callbacks.end_loop(epi_m, epi_n);
        }
      }
```

**EN:** This method block implements `end_loop`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end_loop`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 453-455

```cpp
      cst_callbacks.end();
    }
  }
```

**EN:** This method block implements `end`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 457-460

```cpp
private:
  Params const& params;
  FusionCallbacks fusion_callbacks;
};
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


## Key Concepts / 关键概念

- **Collective epilogue / Collective epilogue:** Coordinates epilogue work at the collective level, often tying dispatch policy, tile shape, and fusion callbacks together. / 在 collective 层面组织 epilogue 工作，通常把调度策略、tile 形状和融合回调连接在一起。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `sycl/sycl.hpp`, `cutlass/cutlass.h`, `cutlass/numeric_conversion.h`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/collective/collective_builder.hpp`, `cutlass/epilogue/collective/collective_epilogue.hpp`, `cutlass/epilogue/collective/detail.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`, `cutlass/detail/layout.hpp`, `cute/tensor.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::collective`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/epilogue/collective/collective_builder.hpp`, `cutlass/epilogue/collective/collective_epilogue.hpp`, `cutlass/epilogue/collective/detail.hpp`, `cutlass/epilogue/fusion/callbacks.hpp`, `cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`

- **Cute/CuTe dependencies / Cute 依赖:** `cute/tensor.hpp`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`, `CUTLASS_TRACE_HOST`
