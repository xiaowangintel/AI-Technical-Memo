# xe_visitor_splitk.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/fusion/xe_visitor_splitk.hpp`

- **Purpose (EN):** Visitor tree Softmax fusion operation for the Intel PVC epilogue.

- **作用 (CN):** 实现 `Xe visitor Split-K` 访问者逻辑，用于遍历融合后的 epilogue 操作树。


## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
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


### Lines 32-34

```cpp
/*! \file
  \brief Visitor tree Softmax fusion operation for the Intel PVC epilogue
*/
```

**EN:** This documentation block explains the intent of the next declaration: ! Visitor tree Softmax fusion operation for the Intel PVC epilogue.

**CN:** 这一段文档注释说明了紧随其后的声明意图，帮助理解后续模板或实现要解决的问题。


### Line 36

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 38-39

```cpp
#include "cutlass/cutlass.h"
#include <sycl/sycl.hpp>
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `sycl/sycl.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`sycl/sycl.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 42

```cpp
namespace cutlass::epilogue::fusion {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 46-63

```cpp
template <
  class CtaTileShapeMNK,
  class EpilogueTile,
  class ElementOutput,
  class ElementCompute,
  class CopyOpR2G,
  FloatRoundStyle RoundStyle
>
struct XeSplitK
{
public:
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


### Lines 65-70

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


### Line 72

```cpp
  struct SharedStorage { };
```

**EN:** Defines `SharedStorage`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `SharedStorage`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 74-82

```cpp
  struct Arguments {
    ElementOutput* ptr_output;
    ElementOutput* ptr_output1;
    ElementOutput* ptr_output2;
    size_t NUM_HEAD;
    size_t NOPE_DIM;
    size_t ROPE_DIM;
    // StrideOutput dOutput;
  };
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 84-91

```cpp
  struct Params {
    XE_Copy_output xe_store_output;
    XE_Copy_output xe_store_output1;
    XE_Copy_output xe_store_output2;
    size_t NUM_HEAD;
    size_t NOPE_DIM;
    size_t ROPE_DIM;
  };
```

**EN:** Defines `Params`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Params`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 93-105

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
    auto NUM_HEAD = args.NUM_HEAD;
    auto NOPE_DIM = args.NOPE_DIM;
    auto ROPE_DIM = args.ROPE_DIM;
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 107-112

```cpp
    XE_Copy_output output1 = make_tiled_copy(
        Copy_Atom<Copy_Traits<CopyOpR2G>, ElementOutput>{}.with(
            args.ptr_output1, M, NUM_HEAD *NOPE_DIM),
        Layout<Shape<_1, Int<IntelXeXMX16::SubgroupSize>>>{},
        make_layout(make_shape(get<0>(typename XE_Copy_output::BlockShape{}),
                               get<1>(typename XE_Copy_output::BlockShape{}) / Int<IntelXeXMX16::SubgroupSize>{})));
```

**EN:** This method block implements `make_tiled_copy`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tiled_copy`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 114-119

```cpp
    XE_Copy_output output2 = make_tiled_copy(
        Copy_Atom<Copy_Traits<CopyOpR2G>, ElementOutput>{}.with(
            args.ptr_output2, M, NUM_HEAD *ROPE_DIM),
        Layout<Shape<_1, Int<IntelXeXMX16::SubgroupSize>>>{},
        make_layout(make_shape(get<0>(typename XE_Copy_output::BlockShape{}),
                               get<1>(typename XE_Copy_output::BlockShape{}) / Int<IntelXeXMX16::SubgroupSize>{})));
```

**EN:** This method block implements `make_tiled_copy`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tiled_copy`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 121-122

```cpp
    return {output, output1, output2, NUM_HEAD, NOPE_DIM, ROPE_DIM};
  }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 124-135

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


### Lines 137-141

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args) {
    return 0;
  }
```

**EN:** Declares the templated `ProblemShape` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `ProblemShape`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 143-148

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


### Lines 150-153

```cpp
  CUTLASS_DEVICE bool
  is_producer_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_producer_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_producer_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 155-158

```cpp
  CUTLASS_DEVICE bool
  is_C_load_needed() const {
    return false;
  }
```

**EN:** This method block implements `is_C_load_needed`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `is_C_load_needed`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 160-161

```cpp
  CUTLASS_HOST_DEVICE
  XeSplitK() { }
```

**EN:** This method block implements `XeSplitK`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `XeSplitK`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 163-165

```cpp
  CUTLASS_HOST_DEVICE
  XeSplitK(Params const& params, SharedStorage const& shared_storage)
      : params(params) { }
```

**EN:** This method block implements `XeSplitK`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `XeSplitK`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 167

```cpp
  Params params;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 169-175

```cpp
  template <class... Args>
  CUTLASS_DEVICE auto
  get_producer_load_callbacks(ProducerLoadArgs<Args...> const& args) {
    return EmptyProducerLoadCallbacks{};
  }
  template<class RTensor, class CoordTensor>
  struct ConsumerStoreCallbacks : EmptyConsumerStoreCallbacks {
```

**EN:** Declares the templated `RTensor` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `RTensor`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 177-183

```cpp
    CUTLASS_DEVICE
    ConsumerStoreCallbacks(RTensor&& res_tensor, CoordTensor&& coord, CoordTensor&& coord1, CoordTensor&& coord2, Params const& params)
      : res_tensor(cute::forward<RTensor>(res_tensor)),
        coord(cute::forward<CoordTensor>(coord)),
        coord1(cute::forward<CoordTensor>(coord1)),
        coord2(cute::forward<CoordTensor>(coord2)),
        params(params) {}
```

**EN:** This method block implements `ConsumerStoreCallbacks`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ConsumerStoreCallbacks`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 185-193

```cpp
    RTensor res_tensor;
    CoordTensor coord;
    CoordTensor coord1;
    CoordTensor coord2;
      Params const& params;
    template <typename ElementInput, typename ElementAccumulator, int FragmentSize>
    CUTLASS_DEVICE auto
    visit(Array<ElementAccumulator, FragmentSize> const& frg_acc, int epi_v, int epi_m, int epi_n,
          Array<ElementInput, FragmentSize> const& frg_input) {
```

**EN:** This method block implements `visit`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `visit`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 195-196

```cpp
      return frg_acc;
    }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 198-206

```cpp
    template<class STensor, class SyncFn, class VTensor>
    CUTLASS_DEVICE void
    reduce(STensor&& smem_buffer, SyncFn const& sync_fn, int epi_m, int epi_n, bool is_last_iteration, VTensor visit_results) {
      if(is_last_iteration) {
      for(int epi_v = 0; epi_v < visit_results(0).size(); epi_v++) {
        res_tensor(epi_v, epi_m, epi_n) = visit_results(0)[epi_v];
      }
      constexpr auto vec_size = min(Epi_M, Sg_N);
      constexpr auto vec_folds = Epi_M / vec_size;
```

**EN:** Defines `STensor`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `STensor`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 208-229

```cpp
      Tensor res =
          make_tensor(static_cast<decltype(res_tensor) &&>(res_tensor).data(),
                      make_shape(Int<vec_size>{}, Int<vec_folds>{}, Int<Epi_N / IntelXeXMX16::SubgroupSize>{}));
      auto n_coord = get<1>(coord[0]);
      auto NOPE_DIM = params.NOPE_DIM;
      auto ROPE_DIM = params.ROPE_DIM;
      auto ROW_DIM = NOPE_DIM + ROPE_DIM;
      int col = n_coord;
      int idx_2 = col % ROW_DIM;
      if (idx_2 < NOPE_DIM) {
          copy(params.xe_store_output1, res_tensor, coord1);
      } else {
          copy(params.xe_store_output2, res_tensor, coord2);
      }
    }
    else {
      for(int epi_v = 0; epi_v < visit_results(0).size(); epi_v++) {
        res_tensor(epi_v, epi_m, epi_n) = visit_results(0)[epi_v];
      }
    }
    }
  };
```

**EN:** This method block implements `make_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `make_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 231-241

```cpp
  template <
  bool ReferenceSrc, // do register tensors reference the src or dst layout of the tiled copy
  class... Args
  >
  CUTLASS_DEVICE auto
  get_consumer_store_callbacks(ConsumerStoreArgs<Args...> const& args) {
    using MmaAtomShape = typename decltype(args.tiled_mma)::AtomShape_MNK;
    constexpr int FragmentSize = get<0>(MmaAtomShape());
    static constexpr int FragsM = get<0>(EpilogueTile{}) / get<0>(MmaAtomShape()); // A frags per sub_group
    static constexpr int FragsN = get<1>(EpilogueTile{}) / get<1>(MmaAtomShape()); // B frags per sub_group
    Tensor res = make_tensor<ElementOutput>(Shape<Int<FragmentSize>, Int<FragsM>, Int<FragsN>>{});
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 243-255

```cpp
    auto [sg_m_coord, sg_n_coord, k_coord, l_offset] = args.tile_coord_mnkl;
    auto [M, N, K, L] = args.problem_shape_mnkl;
    Tensor mAux_mnl = cute::get_xe_tensor(make_shape(M,N,L));
    // Tiling is done differently than in epilogue as we get in coordinates of subgroup in kernel
    Tensor gAux = local_tile(mAux_mnl, select<0,1>(EpilogueTile{}), make_coord(sg_m_coord,sg_n_coord,l_offset));
    Tensor tCgAux = args.tiled_copy.get_thread_slice(args.thread_idx).partition_D(gAux);
    auto nope_dim = params.NOPE_DIM;
    auto rope_dim = params.ROPE_DIM;
    auto inner_dim_sg = (nope_dim + rope_dim) / 32;
    auto nope_dim_sg = nope_dim / 32;
    auto rope_dim_sg = rope_dim / 32;
    unsigned long sg_n_coord_d1 = sg_n_coord % inner_dim_sg;
    unsigned long sg_n_coord_d0 = sg_n_coord / inner_dim_sg;
```

**EN:** This method block implements `get_xe_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_xe_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 257-263

```cpp
    auto num_head = N / inner_dim_sg;
    unsigned long sg_n_coord_1 = sg_n_coord_d0 * nope_dim_sg + sg_n_coord_d1;
    unsigned long sg_n_coord_2 = sg_n_coord_d0 * rope_dim_sg + sg_n_coord_d1 - nope_dim_sg;
    Tensor mAux_mnl1 = cute::get_xe_tensor(make_shape(M,num_head * nope_dim,L));
    // Tiling is done differently than in epilogue as we get in coordinates of subgroup in kernel
    Tensor gAux1 = local_tile(mAux_mnl1, select<0,1>(EpilogueTile{}), make_coord(sg_m_coord,sg_n_coord_1,l_offset));
    Tensor tCgAux1 = args.tiled_copy.get_thread_slice(args.thread_idx).partition_D(gAux1);
```

**EN:** This method block implements `get_xe_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_xe_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 265-268

```cpp
    Tensor mAux_mnl2 = cute::get_xe_tensor(make_shape(M,num_head * rope_dim,L));
    // Tiling is done differently than in epilogue as we get in coordinates of subgroup in kernel
    Tensor gAux2 = local_tile(mAux_mnl2, select<0,1>(EpilogueTile{}), make_coord(sg_m_coord,sg_n_coord_2,l_offset));
    Tensor tCgAux2 = args.tiled_copy.get_thread_slice(args.thread_idx).partition_D(gAux2);
```

**EN:** This method block implements `get_xe_tensor`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_xe_tensor`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 270-276

```cpp
    return ConsumerStoreCallbacks<decltype(res),decltype(tCgAux)>(
      cute::move(res),
      cute::move(tCgAux),
      cute::move(tCgAux1),
      cute::move(tCgAux2),
      params);
  }
```

**EN:** This method block implements `decltype`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `decltype`，为外围 epilogue 组件提供一个聚焦的行为片段。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Reduction behavior / 归约行为:** Supports accumulation across partitions or computes auxiliary reductions during the epilogue. / 支持跨分区累加，或在 epilogue 期间计算附加归约结果。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `sycl/sycl.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::fusion`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`
