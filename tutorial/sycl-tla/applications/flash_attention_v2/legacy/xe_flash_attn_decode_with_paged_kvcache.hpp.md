# xe_flash_attn_decode_with_paged_kvcache.hpp — Code Analysis / 代码分析

## Source / 来源

- **Requested Path / 请求路径:** `applications/flash_attention_v2/legacy/xe_flash_attn_decode_with_paged_kvcache.hpp`
- **Analyzed Source / 实际分析源码:** `applications/flash_attention_v2/kernel/legacy/xe_flash_attn_prefill_cachedKV.hpp`
- **Purpose / 用途:** Legacy paged/cached-KV specialization; the current cached-KV kernel is the nearest maintained implementation.
- **Note / 说明:** The requested paged-KV decode header is absent; the current cached-KV legacy kernel is analyzed as the closest available path.

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
- **EN:** Provides the license header and ownership notice for this source file.
- **CN:** 给出该源文件的许可证头和版权归属说明。

### Lines 34-37

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/gemm.h"
#include "cutlass/kernel_hardware_info.hpp"
```
- **EN:** Imports dependencies such as `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 39-39

```cpp
#include "flash_attention_v2/collective/legacy/xe_flash_attn_prefill_mma_cachedKV.hpp"
```
- **EN:** Imports dependencies such as `flash_attention_v2/collective/legacy/xe_flash_attn_prefill_mma_cachedKV.hpp` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `flash_attention_v2/collective/legacy/xe_flash_attn_prefill_mma_cachedKV.hpp`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 41-41

```cpp
namespace cutlass::flash_attention::kernel {
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

### Lines 43-44

```cpp
template <class ProblemShape, class CollectiveMainloop, class CollectiveSoftmaxEpilogue_, class CollectiveEpilogue, class TileScheduler_ = void>
class FMHAPrefillCached;
```
- **EN:** Defines templated type `FMHAPrefillCached` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAPrefillCached`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 46-46

```cpp
///////////////////////////////////////////////////////////////////////////////
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 48-49

```cpp
template <class ProblemShape_, class CollectiveMainloop_, class CollectiveSoftmaxEpilogue_, class CollectiveEpilogue_, class TileScheduler_>
class FMHAPrefillCached {
```
- **EN:** Defines templated type `FMHAPrefillCached` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAPrefillCached`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 51-55

```cpp
public:
  //
  // Type Aliases
  //
  using ProblemShape = ProblemShape_;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 57-57

```cpp
  static_assert(rank(ProblemShape{}) == 8, "ProblemShape{} should be <batch, num_heads_q, num_head_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo>");
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 59-75

```cpp
  // Mainloop derived types
  using CollectiveMainloop = CollectiveMainloop_;
  using TileShapeQK = typename CollectiveMainloop::TileShapeQK;
  using TileShapePV = typename CollectiveMainloop::TileShapePV;
  using TiledMmaQK = typename CollectiveMainloop::TiledMmaQK;
  using TiledMmaPV = typename CollectiveMainloop::TiledMmaPV;
  using ArchTag = typename CollectiveMainloop::ArchTag;
  using ElementQ = typename CollectiveMainloop::ElementQ;
  using StrideQ = typename CollectiveMainloop::StrideQ;
  using ElementK = typename CollectiveMainloop::ElementK;
  using StrideK = typename CollectiveMainloop::StrideK;
  using ElementV = typename CollectiveMainloop::ElementV;
  using StrideV = typename CollectiveMainloop::StrideV;
  using DispatchPolicy = typename CollectiveMainloop::DispatchPolicy;
  using ElementAccumulator = typename CollectiveMainloop::ElementAccumulator;
  using MainloopArguments = typename CollectiveMainloop::Arguments;
  using MainloopParams = typename CollectiveMainloop::Params;
```
- **EN:** Defines aliases such as `CollectiveMainloop`, `TileShapeQK`, `TileShapePV`, `TiledMmaQK`, `TiledMmaPV`, `ArchTag`, ... (+10) to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveMainloop`, `TileShapeQK`, `TileShapePV`, `TiledMmaQK`, `TiledMmaPV`, `ArchTag`, ... (+10)，用于简化冗长的模板表达式或命名空间限定。

### Lines 77-79

```cpp
  using CollectiveSoftmaxEpilogue = CollectiveSoftmaxEpilogue_;
  using SoftmaxArguments = typename CollectiveSoftmaxEpilogue::Arguments;
  using SoftmaxParams = typename CollectiveSoftmaxEpilogue::Params;
```
- **EN:** Defines aliases such as `CollectiveSoftmaxEpilogue`, `SoftmaxArguments`, `SoftmaxParams` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveSoftmaxEpilogue`, `SoftmaxArguments`, `SoftmaxParams`，用于简化冗长的模板表达式或命名空间限定。

### Lines 81-85

```cpp
  static_assert(cute::is_void_v<TileScheduler_> or cute::is_same_v<TileScheduler_, PersistentScheduler> or 
    cute::is_same_v<TileScheduler_, IndividualScheduler>, "Unsupported TileScheduler for Intel Xe.");
  using TileSchedulerTag = TileScheduler_;
  using TileScheduler = typename detail::TileSchedulerSelector<TileScheduler_, ArchTag>::Scheduler;
  using TileSchedulerParams = typename TileScheduler::Params;
```
- **EN:** Describes scheduling logic that maps FlashAttention tiles onto work-groups or subgroups.
- **CN:** 描述将 FlashAttention tile 映射到工作组或子组的调度逻辑。

### Lines 87-95

```cpp
  // Epilogue derived types
  using CollectiveEpilogue = CollectiveEpilogue_;
  using ElementO = typename CollectiveEpilogue::ElementO;
  using StrideO = typename CollectiveEpilogue::StrideO;
  using ElementLSE = typename CollectiveEpilogue::ElementLSE;
  using EpilogueArguments = typename CollectiveEpilogue::Arguments;
  using EpilogueParams = typename CollectiveEpilogue::Params;
  using TileShapeOutput = typename CollectiveEpilogue::TileShapeOutput;
  using TiledMmaOutput = typename CollectiveEpilogue::TiledMmaOutput;
```
- **EN:** Defines aliases such as `CollectiveEpilogue`, `ElementO`, `StrideO`, `ElementLSE`, `EpilogueArguments`, `EpilogueParams`, ... (+2) to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveEpilogue`, `ElementO`, `StrideO`, `ElementLSE`, `EpilogueArguments`, `EpilogueParams`, ... (+2)，用于简化冗长的模板表达式或命名空间限定。

### Lines 98-99

```cpp
  static_assert(cute::is_same_v<ElementAccumulator, typename CollectiveEpilogue::ElementAccumulator>,
                "Mainloop and epilogue do not agree on accumulator value type.");
```
- **EN:** Adds a compile-time constraint so unsupported combinations fail early during template instantiation.
- **CN:** 添加编译期约束，使不受支持的组合在模板实例化阶段及早失败。

### Lines 101-102

```cpp
  // MSVC requires the cast to fix a warning-as-error.
  static constexpr int SharedStorageSize = 0;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 104-105

```cpp
  static constexpr bool CausalMask = CollectiveMainloop::CausalMask;
  static constexpr bool PagedKV = CollectiveMainloop::PagedKV;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 107-109

```cpp
  static constexpr int SubgroupSize = CollectiveMainloop::SubgroupSize; // sub_group size
  static constexpr uint32_t MaxThreadsPerBlock = CollectiveMainloop::MaxThreadsPerBlock;
  using MmaAtomShape = typename CollectiveMainloop::MmaAtomShape;           // 8,16,16
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 111-113

```cpp
  static constexpr int QK_BLK_M = CollectiveMainloop::QK_BLK_M;
  static constexpr int QK_BLK_N = CollectiveMainloop::QK_BLK_N;
  static constexpr int QK_BLK_K = CollectiveMainloop::QK_BLK_K;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 115-116

```cpp
  static constexpr int QK_ATOM_N = CollectiveMainloop::QK_ATOM_N;
  static constexpr int QK_ATOM_K = CollectiveMainloop::QK_ATOM_K;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 118-118

```cpp
  static constexpr int QK_SG_M = CollectiveMainloop::QK_SG_M;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 120-121

```cpp
  static constexpr int Epilogue_BLK_N = get<1>(TileShapeOutput{});
  static constexpr int Epilogue_BLK_K = get<2>(TileShapeOutput{});
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 123-125

```cpp
  static constexpr int PV_ATOM_M = CollectiveMainloop::PV_ATOM_M;
  static constexpr int PV_ATOM_N = CollectiveMainloop::PV_ATOM_N;
  static constexpr int PV_ATOM_K = CollectiveMainloop::PV_ATOM_K;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 128-132

```cpp
  static constexpr auto Num_SGs = PV_ATOM_N * PV_ATOM_M * PV_ATOM_K;
  static constexpr int Vec =CollectiveMainloop::Vec; 
  static constexpr int FragsM =CollectiveMainloop::FragsM; 
  // The FragsN here used for Creation of S matrix so we use the FragsN for S shape
  static constexpr int FragsN = CollectiveMainloop::FragsNS; 
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 134-135

```cpp
  static constexpr int VSlicer = get<1>(TileShapeOutput{})/(get<1>(TileShapePV{})* PV_ATOM_N); //ceil_div(FragsNOut,FragsNS);
  using AccumeShape =  decltype(make_shape(Int<Vec>{}, Int<FragsM>{}, get<1>(TileShapePV{})/get<1>(MmaAtomShape()), Int<VSlicer>{}));
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 137-137

```cpp
  static constexpr bool is_var_len = CollectiveMainloop::is_var_len;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 139-143

```cpp
  // Kernel level shared memory storage
  struct SharedStorage {
    using EpilogueTensorStorage = typename CollectiveEpilogue::TensorStorage;
    EpilogueTensorStorage epilogue;
  };
```
- **EN:** Declares `SharedStorage` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `SharedStorage` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 145-153

```cpp
  // Device side arguments
  struct Arguments {
    gemm::GemmUniversalMode mode{};
    ProblemShape problem_shape{};
    MainloopArguments mainloop{};
    SoftmaxArguments softmax{};
    EpilogueArguments epilogue{};
    KernelHardwareInfo hw_info{};
  };
```
- **EN:** Declares `Arguments` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Arguments` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 155-163

```cpp
  // Kernel entry point API
  struct Params {
    gemm::GemmUniversalMode mode;
    ProblemShape problem_shape;
    MainloopParams mainloop;
    SoftmaxParams softmax;
    EpilogueParams epilogue;
    TileSchedulerParams scheduler;
  };
```
- **EN:** Declares `Params` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Params` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 165-167

```cpp
  //
  // Methods
  //
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 169-177

```cpp
  // Convert to underlying arguments. In this case, a simple copy for the aliased type.
  static Params to_underlying_arguments(Arguments const &args, void *workspace) {
    (void)workspace;
    return {args.mode, args.problem_shape,
            CollectiveMainloop::to_underlying_arguments(args.problem_shape, args.mainloop, workspace),
            CollectiveSoftmaxEpilogue::to_underlying_arguments(args.softmax),
            CollectiveEpilogue::to_underlying_arguments(args.problem_shape, args.epilogue, workspace),
            TileScheduler::to_underlying_arguments(args.problem_shape, args.hw_info, TileShapeOutput{})};
  }
```
- **EN:** Converts the public argument bundle into lower-level parameter objects consumed by the kernel components.
- **CN:** 把公开参数包转换为内核组件实际使用的底层参数对象。

### Lines 179-184

```cpp
  static bool can_implement(Arguments const &args) {
    bool mode_implementable = args.mode == gemm::GemmUniversalMode::kGemm or
                              (args.mode == gemm::GemmUniversalMode::kBatched && rank(ProblemShape{}) == 4);
    bool valid_page_size = !PagedKV || (args.mainloop.page_size >= QK_BLK_N && args.mainloop.page_size % QK_BLK_N == 0);
    return mode_implementable && valid_page_size;
  }
```
- **EN:** Checks whether the selected configuration is supported before kernel launch.
- **CN:** 在启动内核之前检查所选配置是否受支持。

### Lines 186-186

```cpp
  static int get_workspace_size(Arguments const &args) { return 0; }
```
- **EN:** Reports how much auxiliary workspace this specialization needs.
- **CN:** 报告该特化所需的辅助工作空间大小。

### Lines 188-191

```cpp
  static cutlass::Status initialize_workspace(Arguments const &args, void *workspace = nullptr,
                                              cudaStream_t stream = nullptr, CudaHostAdapter *cuda_adapter = nullptr) {
    return Status::kSuccess;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 193-195

```cpp
  static dim3 get_grid_shape(Params const &params) {
    return TileScheduler::template get_grid_shape<Num_SGs>(params.scheduler);
  }
```
- **EN:** Computes the launch grid from scheduler parameters and tiling choices.
- **CN:** 根据调度参数和分块选择计算启动网格。

### Lines 197-197

```cpp
  static dim3 get_block_shape() { return dim3(MaxThreadsPerBlock, 1, 1); }
```
- **EN:** Returns the thread-block shape used to launch the kernel on the device.
- **CN:** 返回在设备上启动该内核时使用的线程块形状。

### Lines 199-206

```cpp
  CUTLASS_DEVICE
  Shape<int, int, int> get_sequence_length_shape(ProblemShape const& problem_shape, int const& batch) {
    if constexpr (is_var_len) {
      return cutlass::fmha::collective::apply_variable_length(select<3, 4, 5>(problem_shape), batch);
    } else {
      return select<3, 4, 5>(problem_shape);
    }
  }
```
- **EN:** Resolves per-batch sequence-length metadata, including the variable-length case.
- **CN:** 解析每个 batch 的序列长度元数据，包括变长场景。

### Lines 208-224

```cpp
  CUTLASS_DEVICE
  void operator()(Params const &params, char *smem_buf) {
    SharedStorage &shared_storage = *reinterpret_cast<SharedStorage *>(smem_buf);
    // Preconditions
    CUTE_STATIC_ASSERT(is_static<TileShapeQK>::value);
    CUTE_STATIC_ASSERT(is_static<TileShapePV>::value);
    // Separate out problem shape for convenience
    auto& batch = get<0>(params.problem_shape);
    auto& num_heads_q = get<1>(params.problem_shape);
    auto& num_head_kv = get<2>(params.problem_shape);
    auto group_heads_q = num_heads_q / num_head_kv;
    auto& head_size_qk = get<6>(params.problem_shape);
    auto& head_size_vo = get<7>(params.problem_shape);
    // Preconditions
    static_assert(cute::rank(StrideQ{}) == 3, "StrideQ must be rank-3: [seq_len_qo, head_size_qk, batch * num_heads_q].");
    static_assert(cute::rank(StrideK{}) == 3, "StrideK must be rank-3: [head_size_qk, seq_len_kv, batch * num_heads_kv].");
    static_assert(cute::rank(StrideV{}) == 3, "StrideV must be rank-3: [seq_len_kv, head_size_vo, batch * num_heads_kv].");
```
- **EN:** Implements the core device-side operator that walks tiles, moves data, and performs attention math.
- **CN:** 实现核心设备端调用算子，负责遍历 tile、搬运数据并执行注意力计算。

### Lines 226-227

```cpp
    int thread_idx = int(ThreadIdxX());
    int sub_group_id = thread_idx / SubgroupSize;
```
- **EN:** Selects the subgroup lane and copy primitive used for the upcoming data movement step.
- **CN:** 选择子组 lane 以及后续数据搬运要使用的复制原语。

### Lines 229-229

```cpp
    TileScheduler tile_scheduler{params.scheduler};
```
- **EN:** Describes scheduling logic that maps FlashAttention tiles onto work-groups or subgroups.
- **CN:** 描述将 FlashAttention tile 映射到工作组或子组的调度逻辑。

### Lines 231-233

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for (; tile_scheduler.is_valid(); ++tile_scheduler) {
      auto blk_coord = tile_scheduler.get_block_coord(); // head_size_blk_idx, seq_len_blk_idx, batch_blk_idx, num_heads_blk_idx
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 235-238

```cpp
      auto blk_m_coord = get<1>(blk_coord); // seq_len_blk_idx
      auto blk_n_coord = get<0>(blk_coord); // head_size_blk_idx
      auto batch_coord = get<2>(blk_coord); // batch_blk_idx
      auto num_heads_coord = get<3>(blk_coord); // num_heads_blk_idx
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 240-245

```cpp
      // For variable sequence length case, batch is considered to be 1 (same as group gemm).
      // For fixed sequence length case, the l_coord is the weighted sum of both batch_coord and num_heads_coord.
      // Flash Attention implementation combines batch and num_heads to calculate the total batch_size.
      // iff is_var_len: batch_size = num_heads (as each batch would have it's own seq_len_qo and seq_len_kv)
      // iff !is_var_len: batch_size = batch * num_heads
      auto blk_l_coord = is_var_len ? num_heads_coord : batch_coord * num_heads_q + num_heads_coord;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 247-251

```cpp
      // Get problem shape for the current batch_blk_idx. For variable sequence length, it loads the sequence length
      // from Global memory for the given batch_blk_idx and returns the appropriate problem_shape. For fixed sequence
      // length, sequence_length_shape == select<3, 4, 5>(params.problem_shape).
      // sequence_length_shape = [batch, num_heads_q, num_heads_kv, seq_len_qo, seq_len_kv, seq_len_kv_cache, head_size_qk, head_size_vo]
      auto sequence_length_shape = get_sequence_length_shape(params.problem_shape, batch_coord);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 253-253

```cpp
      auto [seq_len_qo, seq_len_kv, seq_len_kv_cache] = sequence_length_shape;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 255-259

```cpp
      // Calculate the seq_len_idx (blk_m_coord * get<0>(TileShapeOutput{})) and check if it is still
      // within bounds of the actual seq_len_qo (get<0>(sequence_length_shape)).
      if (blk_m_coord * get<0>(TileShapeOutput{}) >= seq_len_qo) {
        continue;
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 261-264

```cpp
      auto offset = cute::min(seq_len_qo, seq_len_kv); //(2048, 1024)
      auto discard_seq_coord = seq_len_qo - offset; //1024
      auto full_tile_offset = seq_len_kv - offset; //0
      const int seq_coord = cute::min(seq_len_qo, blk_m_coord * QK_BLK_M + (sub_group_id / PV_ATOM_N) * QK_SG_M) ;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 266-266

```cpp
      const int seq_len = CausalMask ? full_tile_offset + cute::min(seq_len_kv, seq_coord - discard_seq_coord) + QK_SG_M : seq_len_kv;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 268-270

```cpp
      const int nblock_new = cute::ceil_div(seq_len, QK_BLK_N);
      const int nblock_cache = cute::ceil_div(seq_len_kv_cache, QK_BLK_N);
      const int nblock_limit = nblock_cache + nblock_new;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 272-272

```cpp
      int tiles_per_page = params.mainloop.page_size / QK_BLK_N;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 274-276

```cpp
      if(CausalMask && seq_coord < discard_seq_coord ) { // 1024 =0
        continue;
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 278-282

```cpp
      Tensor mQ_mkl = cute::get_xe_tensor(make_shape(seq_len_qo, head_size_qk, (is_var_len ? 1 : batch) * num_heads_q));   //(m,k,l)
      Tensor mK_nkl = cute::get_xe_tensor(make_shape(seq_len_kv, head_size_qk, (is_var_len ? 1 : batch) * num_head_kv));   //(n,k,l)
      Tensor mV_nkl = cute::get_xe_tensor(make_shape(head_size_vo, seq_len_kv, (is_var_len ? 1 : batch) * num_head_kv));   //(n,k,l)
      Tensor mK_cache_nkl = cute::get_xe_tensor(make_shape(seq_len_kv_cache, head_size_qk, (is_var_len ? 1 : batch) * num_head_kv));   // (n_cache,k,l)
      Tensor mV_cache_nkl = cute::get_xe_tensor(make_shape(head_size_vo, seq_len_kv_cache, (is_var_len ? 1 : batch) * num_head_kv));   // (n_cache,k,l)
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 284-288

```cpp
      Tensor mQ_mk = mQ_mkl(_, _, blk_l_coord);                                                                  // (m,k)
      Tensor mK_nk = mK_nkl(_, _, blk_l_coord/group_heads_q);                                                    // (n,k)
      Tensor mV_nk = mV_nkl(_, _, blk_l_coord/group_heads_q);                                                    // (n,k)
      Tensor mK_cache_nk = mK_cache_nkl(_, _, blk_l_coord/group_heads_q);                                        // (n_cache, k)
      Tensor mV_cache_nk = mV_cache_nkl(_, _, blk_l_coord/group_heads_q);                                        // (n_cache, k)
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 290-294

```cpp
      auto gQ = local_tile(mQ_mk, TileShapeQK{}, make_coord(blk_m_coord, _, _), Step<_1,  X, _1>{});
      auto gK = local_tile(mK_nk, TileShapeQK{}, make_coord(_, _ , _), Step<X, _1, _1>{});
      auto gV = local_tile(mV_nk, TileShapeOutput{}, make_coord(_, blk_n_coord, _), Step<X, _1, _1>{});
      auto gK_cache = local_tile(mK_cache_nk, TileShapeQK{}, make_coord(_, _, _), Step<X, _1, _1>{});
      auto gV_cache = local_tile(mV_cache_nk, TileShapeOutput{}, make_coord(_, blk_n_coord, _), Step<X, _1, _1>{});
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 296-318

```cpp
      auto mainloop_params = CollectiveMainloop::get_updated_copies(params.mainloop, params.problem_shape, sequence_length_shape, batch_coord);
      // we limit the horisontal size to two subgroup, the empirical resutls show that reading the two cacheline side by side in gives better performance and 
      // anything after that does not have an effect on performance. // (64 here for float b float when possible and loop over to cover all the data needed)
      auto tiled_prefetch_q = cute::prefetch_selector<Shape<Int<QK_BLK_M>, Int<cute::max(cute::gcd(QK_BLK_K, 64), 32)>>, Num_SGs>(mainloop_params.gmem_tiled_copy_q);
      auto tiled_prefetch_k = cute::prefetch_selector<Shape<Int<QK_BLK_N>, Int<cute::max(cute::gcd(QK_BLK_K, 64), 32)>>, Num_SGs>(mainloop_params.gmem_tiled_copy_k);
      auto tiled_prefetch_v = cute::prefetch_selector<Shape<Int<cute::max(cute::gcd(Epilogue_BLK_N, 64), 32)>, Int<Epilogue_BLK_K>>, Num_SGs>(mainloop_params.gmem_tiled_copy_v);
      auto tiled_prefetch_k_cache = cute::prefetch_selector<Shape<Int<QK_BLK_N>, Int<cute::max(cute::gcd(QK_BLK_K, 64), 32)>>, Num_SGs>(mainloop_params.gmem_tiled_copy_k_cache);
      auto tiled_prefetch_v_cache = cute::prefetch_selector<Shape<Int<cute::max(cute::gcd(Epilogue_BLK_N, 64), 32)>, Int<Epilogue_BLK_K>>, Num_SGs>(mainloop_params.gmem_tiled_copy_v_cache);
      auto thr_prefetch_Q = tiled_prefetch_q.get_slice(thread_idx);
      auto thr_prefetch_K = tiled_prefetch_k.get_slice(thread_idx);
      auto thr_prefetch_V = tiled_prefetch_v.get_slice(thread_idx);
      auto pQgQ = thr_prefetch_Q.partition_S(gQ);
      auto pKgK = thr_prefetch_K.partition_S(gK);
      auto pVgV = thr_prefetch_V.partition_S(gV);
      // assuming the copy function is the same otherwise this need to have its own tile_prefetch
      auto pKgK_cache = thr_prefetch_K.partition_S(gK_cache);
      auto pVgV_cache = thr_prefetch_V.partition_S(gV_cache);
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size<3>(pQgQ); i++) {
        prefetch(tiled_prefetch_q, pQgQ(_, _, _, i));
      }
      auto& prefetch_K = (seq_len_kv_cache == 0) ? tiled_prefetch_k: tiled_prefetch_k_cache;
      auto& pKgK1_ = (seq_len_kv_cache == 0) ? pKgK: pKgK_cache;
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 320-337

```cpp
      int cached_nblock = 0;
      if constexpr (PagedKV) {
        if (seq_len_kv_cache != 0) {
          int curr_batch_pages = is_var_len ? mainloop_params.num_pages_per_seq[batch_coord + 1] - mainloop_params.num_pages_per_seq[batch_coord]
                                            : ceil_div(seq_len_kv_cache, mainloop_params.page_size);
          int batch_offset = is_var_len ? mainloop_params.num_pages_per_seq[batch_coord] : batch_coord * curr_batch_pages;
          cached_nblock = mainloop_params.ptr_page_table[
                    batch_offset                     // page table for this batch
                ] * tiles_per_page;               // base block idx of physical page
        }
      }
       // The headsize for both cached and non-cached version is the same
      for (int j = 0; j < size<4>(pKgK1_); j++) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = cached_nblock; i < cached_nblock + DispatchPolicy::Stages; i++) {
          prefetch(prefetch_K, pKgK1_(_, _, _ , i, j));
        }
      }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 339-340

```cpp
      // Allocate the tiled_mma and the accumulators for the (M,N) workgroup_shape
      Tensor out_reg = make_tensor<ElementAccumulator>(AccumeShape{});
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 342-346

```cpp
      // There are 16 workitem and 16 max per subgroup, each worktime containt 1 max and cumulatively, they calculate the
      // max per subgroup
      ElementAccumulator max_reg{-INFINITY};
      // The sum reg each contains a 2d tesnor for 8 x 2 This is number of sequence lenght process per subgroup
      Tensor sum_reg = make_tensor<ElementAccumulator>(Shape<Int<Vec>, Int<FragsM>>{});
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 348-358

```cpp
      clear(sum_reg);
      clear(out_reg);
      // Perform the collective scoped MMA
      CollectiveMainloop collective_mma;
      // when causal mask is true. It is not possible to set the scope
      // of the barrier to workgroup level as the number n block is
      // different for each subgroup due to triangular nature of causal based operation
      static constexpr int barrier_scope = CausalMask ? 3 : 2;
      // MAIN LOOP: loop over K and V, perform fused attention + online softmax
      for (int nblock = 0; nblock < nblock_limit - static_cast<int>(CausalMask); nblock++) {
        barrier_arrive(barrier_scope);
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 360-360

```cpp
        bool is_KV_cache = nblock < nblock_cache;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 362-364

```cpp
        // 1) Load KV (performed inside mmaQK)
        auto gK_ = is_KV_cache ? gK_cache(_, _, cached_nblock, _) : gK(_, _, nblock - nblock_cache, _);
        auto gV_ = is_KV_cache ? gV_cache(_, _, cached_nblock) : gV(_, _, nblock - nblock_cache);
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 366-368

```cpp
        // 2) Create Tensor S
        Tensor tSr = make_tensor<ElementAccumulator>(Shape<Int<Vec>, Int<FragsM>, Int<FragsN>>{});
        clear(tSr);
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 370-371

```cpp
        // 3) Perform GEMM S = Q*K
        collective_mma.mmaQK(tSr, gQ, gK_, tSr, ceil_div(head_size_qk, QK_BLK_K), mainloop_params, is_KV_cache);
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 373-381

```cpp
        // we only need one block ahead, there is enough gap to prefetch it while doing softmax. because the gap between the two MMA is big,
        // prefetching it the same way as cutlass K matrix does not make sense
        auto& tiled_prefetch_v_ = is_KV_cache ? tiled_prefetch_v_cache : tiled_prefetch_v;
        auto& pVgV_ = is_KV_cache  ? pVgV_cache : pVgV;
        int v_prefetch_idx = is_KV_cache ? PagedKV ? cached_nblock : nblock
                                         : nblock - nblock_cache;
        for(int i = 0; i < size<1>(pVgV_); i++) {
          prefetch(tiled_prefetch_v_, pVgV_(_, i, _ , v_prefetch_idx));
        }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 383-403

```cpp
        int next_cached_nblock = nblock + 1;
        bool is_next_KV_cache = next_cached_nblock < nblock_cache;
        if constexpr (PagedKV) {
          if (is_next_KV_cache) {
            int curr_batch_pages = is_var_len ? mainloop_params.num_pages_per_seq[batch_coord + 1] - mainloop_params.num_pages_per_seq[batch_coord]
                                              : ceil_div(seq_len_kv_cache, mainloop_params.page_size);
            int next_page_logical_idx = next_cached_nblock * QK_BLK_N / params.mainloop.page_size;
            int batch_offset = is_var_len ? mainloop_params.num_pages_per_seq[batch_coord] : batch_coord * curr_batch_pages;
            bool valid_page = next_page_logical_idx < curr_batch_pages;
            // get physical page idx from page table
            if (valid_page) {
              next_cached_nblock = params.mainloop.ptr_page_table[
                    batch_offset +                  // page table for this batch
                    next_page_logical_idx           // nblock (tile idx) to logical page idx
                    ] * tiles_per_page +            // base block idx of physical page
                    next_cached_nblock % tiles_per_page;        // offset within page
            } else {
              next_cached_nblock = curr_batch_pages * tiles_per_page; // push idx out of bounds to respect the boundary between batches
            }
          }
        }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 405-407

```cpp
        // 4) Fused softmax
        CollectiveSoftmaxEpilogue softmax(params.softmax);
        softmax(nblock == 0, tSr, max_reg, sum_reg, out_reg);
```
- **EN:** Implements or invokes the online softmax update used during tiled attention reduction.
- **CN:** 实现或调用分块注意力归约过程中使用的在线 Softmax 更新。

### Lines 409-410

```cpp
        // 5) Perform GEMM O = S*V
        collective_mma.template mmaPV<VSlicer>(out_reg, tSr, gV_, out_reg, mainloop_params, is_KV_cache);
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 412-416

```cpp
        // Prefetch the next Q tile
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size<3>(pQgQ); i++) {
          prefetch(tiled_prefetch_q, pQgQ(_, _, _, i));
        }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 418-421

```cpp
        is_KV_cache = is_next_KV_cache;
        cached_nblock = next_cached_nblock;
        // Prefetch the next K tile
        // there is no need to gaurd it with if statememt as prefetch will ignore out of bound reading
```
- **EN:** Issues prefetch operations so future K/V/Q tiles are already in faster memory when needed.
- **CN:** 发出预取操作，使未来的 K/V/Q tile 在需要时已进入更快的存储层。

### Lines 423-433

```cpp
        bool sel_prefetch_k = (nblock + DispatchPolicy::Stages) < nblock_cache;
        auto& prefetch_k_selector = sel_prefetch_k ? tiled_prefetch_k_cache: tiled_prefetch_k;
        auto& pKgK_ = sel_prefetch_k  ? pKgK_cache : pKgK;
        int k_prefetch_idx = sel_prefetch_k ? PagedKV ? cached_nblock : nblock + DispatchPolicy::Stages
                                            : nblock + DispatchPolicy::Stages - nblock_cache;
        CUTLASS_PRAGMA_UNROLL
        for (int j = 0; j < size<4>(pKgK_); j++) {
          prefetch(prefetch_k_selector, pKgK_(_, _, _, k_prefetch_idx , j));
        }
        barrier_wait(barrier_scope);
      }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 435-463

```cpp
      if constexpr (CausalMask) {
        // BAND Matrix
        // 1) Load K (performed inside mmaQK)
        // 2) Create Tensor S
        Tensor tSr = make_tensor<ElementAccumulator>(Shape<Int<Vec>, Int<FragsM>, Int<FragsN>>{});
        clear(tSr);
        // 3) Perform GEMM S = Q*K
        collective_mma.mmaQK(tSr, gQ,  gK(_, _, nblock_new - 1, _), tSr, ceil_div(head_size_qk, QK_BLK_K), mainloop_params, false);
        // we only need one block ahead, there is enough gap to prefetch it while doing softmax. because the gap between the two MMA is big,
        // prefetching it the same way as cutlass K matrix does not make sense
        for(int i = 0; i< size<1>(pVgV); i++) {
          prefetch(tiled_prefetch_v, pVgV(_, i, _ , nblock_new - 1));
        }
        // mask the elements of each tile where j > i
        const int item_id = thread_idx % SubgroupSize;
        int col_idx = item_id + (nblock_new - 1) * QK_BLK_N;
        CUTLASS_PRAGMA_UNROLL
        for (int n = 0; n < FragsN; n++, col_idx += get<1>(MmaAtomShape())) { // 4
          CUTLASS_PRAGMA_UNROLL
          for (int m = 0; m < FragsM; m++) { // 2
            int row_idx = m * Vec + seq_coord;
            CUTLASS_PRAGMA_UNROLL
            for (int row = 0; row < Vec; row++, row_idx++) { // 8
              if (col_idx - full_tile_offset > row_idx - discard_seq_coord) {
                tSr(row, m, n) = ElementAccumulator{-INFINITY};
              }
            }
          }
        }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 465-466

```cpp
        CollectiveSoftmaxEpilogue softmax(params.softmax);
        softmax((nblock_limit - 1) == 0, tSr, max_reg, sum_reg, out_reg);
```
- **EN:** Implements or invokes the online softmax update used during tiled attention reduction.
- **CN:** 实现或调用分块注意力归约过程中使用的在线 Softmax 更新。

### Lines 468-469

```cpp
        collective_mma.template mmaPV<VSlicer>(out_reg, tSr,  gV(_, _ , nblock_new - 1), out_reg, mainloop_params, false);
      }
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 471-477

```cpp
      auto epilogue_params = CollectiveEpilogue::template get_updated_copies<is_var_len>(params.epilogue, params.problem_shape, sequence_length_shape, batch_coord);
      CollectiveEpilogue epilogue{epilogue_params, shared_storage.epilogue};
      auto blk_coord_mnkl = make_coord(blk_m_coord, blk_n_coord, _, blk_l_coord);
      epilogue(params.problem_shape, sequence_length_shape, blk_coord_mnkl, out_reg, max_reg, sum_reg);
    }
  }
};
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 479-479

```cpp
///////////////////////////////////////////////////////////////////////////////
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 481-481

```cpp
} // namespace cutlass::flash_attention::kernel
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

## Key Concepts / 关键概念

- **EN:** Decode path: the code focuses on token-by-token attention accumulation over existing KV state.
- **CN:** 解码路径：代码侧重在已有 KV 状态上逐 token 执行注意力累积。
- **EN:** Prefill path: the implementation targets full-sequence attention setup for prompt ingestion.
- **CN:** Prefill 路径：实现面向提示词填充阶段的整段注意力计算。
- **EN:** Kernel orchestration: the file assembles problem shape, arguments, scheduling, and device execution.
- **CN:** 内核编排：该文件组合问题形状、参数、调度与设备端执行逻辑。
- **EN:** Online softmax: max/sum statistics are updated incrementally to avoid storing full score matrices.
- **CN:** 在线 Softmax：逐块更新 max/sum 统计量，避免存储完整分数矩阵。
- **EN:** MMA tiling: matrix-multiply-accumulate building blocks are specialized for Xe subgroup execution.
- **CN:** MMA 分块：矩阵乘加构件针对 Xe 子组执行进行专门化。
- **EN:** KV-cache specialization: the implementation selects between live K/V tensors and cached or paged variants.
- **CN:** KV 缓存特化：实现会在实时 K/V 张量与缓存/分页变体之间进行选择。

## Dependencies / 依赖关系

- **EN:** Direct dependencies referenced here include `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`, `flash_attention_v2/collective/legacy/xe_flash_attn_prefill_mma_cachedKV.hpp`.
- **CN:** 这里引用的直接依赖包括 `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`, `flash_attention_v2/collective/legacy/xe_flash_attn_prefill_mma_cachedKV.hpp`。
- **EN:** It belongs to the legacy FlashAttention stack, so it couples to older kernel, scheduler, or epilogue contracts.
- **CN:** 它属于 legacy FlashAttention 栈，因此会耦合旧版内核、调度器或 epilogue 契约。
- **EN:** Kernel-level code depends on collective components for math/data movement and on scheduler types for grid traversal.
- **CN:** 内核级代码依赖 collective 组件完成数学与数据搬运，并依赖调度器类型遍历网格。
