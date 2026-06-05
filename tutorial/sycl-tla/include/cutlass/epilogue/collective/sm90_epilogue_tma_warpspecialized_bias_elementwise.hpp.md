# sm90_epilogue_tma_warpspecialized_bias_elementwise.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/collective/sm90_epilogue_tma_warpspecialized_bias_elementwise.hpp`

- **Purpose (EN):** Functor performing pipelined epilogues with bias add and elementwise activation functions. This collective is now DEPRECATED, will be removed in the next release. Use EVT instead.

- **作用 (CN):** 实现 `SM90 epilogue TMA warpspecialized bias elementwise` 这一 epilogue 流水线组件。


## Line-by-Line Analysis / 逐行分析

### Lines 1-34

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
  \brief Functor performing pipelined epilogues with bias add and elementwise activation functions.
         This collective is now DEPRECATED, will be removed in the next release. Use EVT instead.
*/
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Line 36

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Line 38

```cpp
#include "sm90_epilogue_tma_warpspecialized.hpp"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `sm90_epilogue_tma_warpspecialized.hpp`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `sm90_epilogue_tma_warpspecialized.hpp`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 42-44

```cpp
namespace cutlass {
namespace epilogue {
namespace collective {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 48-111

```cpp
template <
  int StagesC_,
  int StagesD_,
  int FragmentSize_,
  class BlockTileShape_,    //     (BLK_M,BLK_N,BLK_K)
  class EpilogueTileShape_, // (EPI_TILE_M,EPI_TILE_N)
  class ElementC_,
  class StrideC_,
  class ElementD_,
  class StrideD_,
  class FusionCallbacks_,
  class CopyOpG2S_,
  class SmemLayoutAtomC_,
  class CopyOpS2R_,
  class CopyOpS2G_,
  class SmemLayoutAtomD_,
  class CopyOpR2S_,
  class CopyAtomC_,
  class CopyOpR2R_
>
class Sm90EpilogueTmaWarpSpecializedBiasElementwise
  : public CollectiveEpilogue<
      Sm90TmaWarpSpecialized<StagesC_, StagesD_, FragmentSize_, false, false>,
      BlockTileShape_,
      EpilogueTileShape_,
      ElementC_,
      StrideC_,
      ElementD_,
      StrideD_,
      FusionCallbacks_,
      CopyOpG2S_,
      SmemLayoutAtomC_,
      CopyOpS2R_,
      CopyOpS2G_,
      SmemLayoutAtomD_,
      CopyOpR2S_,
      CopyAtomC_,
      CopyOpR2R_
> {
private:
  using Impl =
    CollectiveEpilogue<
      Sm90TmaWarpSpecialized<StagesC_, StagesD_, FragmentSize_, false, false>,
      BlockTileShape_,
      EpilogueTileShape_,
      ElementC_,
      StrideC_,
      ElementD_,
      StrideD_,
      FusionCallbacks_,
      CopyOpG2S_,
      SmemLayoutAtomC_,
      CopyOpS2R_,
      CopyOpS2G_,
      SmemLayoutAtomD_,
      CopyOpR2S_,
      CopyAtomC_,
      CopyOpR2R_
    >;
public:
  using DispatchPolicy = Sm90TmaWarpSpecializedBiasElementwise<StagesC_, StagesD_, FragmentSize_>;
  using ElementCompute = typename Impl::ThreadEpilogueOp::ElementCompute;
  using ElementBias = typename Impl::ThreadEpilogueOp::ElementBias;
  using ElementT = typename Impl::ThreadEpilogueOp::ElementAux;
```

**EN:** Declares the templated `BlockTileShape_` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `BlockTileShape_`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 113-114

```cpp
  // Constructor inheritance
  using Impl::Impl;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 116-130

```cpp
  // Host side epilogue arguments
  struct [[deprecated("use Sm90TmaWarpSpecialized Arguments instead")]]
  Arguments {
    struct ThreadArgs {
      ElementCompute alpha{1};
      ElementCompute beta{0};
      ElementCompute const *alpha_ptr{nullptr};
      ElementCompute const *beta_ptr{nullptr};
    } thread;
    ElementC_ const* ptr_C{nullptr};
    StrideC_ dC{};
    ElementD_* ptr_D{nullptr};
    StrideD_ dD{};
    ElementBias const* ptr_Bias{nullptr};
    ElementT* ptr_T{nullptr};
```

**EN:** Defines `ThreadArgs`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Host side epilogue arguments.

**CN:** 定义 `ThreadArgs`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 132-149

```cpp
    CUTLASS_HOST_DEVICE
    operator typename Impl::Arguments() const {
      typename Impl::Arguments arguments;
      arguments.thread.alpha = thread.alpha;
      arguments.thread.beta = thread.beta;
      arguments.thread.alpha_ptr = thread.alpha_ptr;
      arguments.thread.beta_ptr = thread.beta_ptr;
      if constexpr (not cute::is_void_v<ElementBias>) {
        arguments.thread.bias_ptr = ptr_Bias;
      }
      if constexpr (not cute::is_void_v<ElementT>) {
        arguments.thread.aux_ptr = ptr_T;
        arguments.thread.dAux = dD;
      }
      arguments.ptr_C = ptr_C;
      arguments.dC = dC;
      arguments.ptr_D = ptr_D;
      arguments.dD = dD;
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 151-153

```cpp
      return arguments;
    }
  };
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


## Key Concepts / 关键概念

- **Collective epilogue / Collective epilogue:** Coordinates epilogue work at the collective level, often tying dispatch policy, tile shape, and fusion callbacks together. / 在 collective 层面组织 epilogue 工作，通常把调度策略、tile 形状和融合回调连接在一起。

- **Post-ops / 后处理算子:** Fuses nonlinear activation or other elementwise transforms directly into the epilogue. / 把非线性激活或其他逐元素变换直接融合进 epilogue。

- **Broadcast inputs / 广播输入:** Consumes auxiliary tensors such as bias or broadcast values alongside accumulator fragments. / 在处理累加器片段时同时消费 bias 或其他广播输入张量。

- **Architecture specialization / 架构特化:** Selects data movement and compute behavior for a particular GPU architecture or programming backend. / 针对特定 GPU 架构或编程后端选择数据搬运与计算行为。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `sm90_epilogue_tma_warpspecialized.hpp`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::collective`

- **Feature macros / 特性宏:** `CUTLASS_HOST_DEVICE`
