# flash_attn_combinedQK.hpp — Code Analysis / 代码分析

## Source / 来源

- **Requested Path / 请求路径:** `applications/flash_attention_v2/flash_attn_combinedQK.hpp`
- **Analyzed Source / 实际分析源码:** `applications/flash_attention_v2/kernel/xe_fmha_fwd_kernel.hpp`
- **Purpose / 用途:** Combined-Q/K forward kernel wrapper; the current Xe FMHA forward kernel covers that integrated behavior.
- **Note / 说明:** The requested combined-Q/K file is unavailable here; the closest unified kernel wrapper is analyzed.

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
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
```
- **EN:** Provides the license header and ownership notice for this source file.
- **CN:** 给出该源文件的许可证头和版权归属说明。

### Lines 32-32

```cpp
#pragma once
```
- **EN:** Uses `#pragma once` so the header is included only once per translation unit.
- **CN:** 使用 `#pragma once`，确保头文件在同一编译单元中只被包含一次。

### Lines 34-37

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/gemm.h"
#include "cutlass/kernel_hardware_info.hpp"
```
- **EN:** Imports dependencies such as `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 39-43

```cpp
#include "flash_attention_v2/collective/xe_fmha_fwd_mainloop.hpp"
#include "flash_attention_v2/collective/xe_fmha_fwd_epilogue.hpp"
#include "cute/util/type_traits.hpp"
#include "flash_attention_v2/collective/fmha_fusion.hpp"
#include "flash_attention_v2/kernel/xe_tile_scheduler.hpp"
```
- **EN:** Imports dependencies such as `flash_attention_v2/collective/xe_fmha_fwd_mainloop.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_epilogue.hpp`, `cute/util/type_traits.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp` so this file can reuse CUTLASS/CUTE, benchmark, or FlashAttention helpers.
- **CN:** 引入依赖头文件，例如 `flash_attention_v2/collective/xe_fmha_fwd_mainloop.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_epilogue.hpp`, `cute/util/type_traits.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, `flash_attention_v2/kernel/xe_tile_scheduler.hpp`，使本文件能够复用 CUTLASS/CUTE、benchmark 或 FlashAttention 辅助组件。

### Lines 45-45

```cpp
namespace cutlass::fmha::kernel {
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

### Lines 47-47

```cpp
using namespace cute;
```
- **EN:** Defines aliases such as `namespace` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `namespace`，用于简化冗长的模板表达式或命名空间限定。

### Lines 49-57

```cpp
///////////////////////////////////////////////////////////////////////////////
template <bool IsVarLen_ = false>
struct FMHAProblemShape {
  using SeqLenType = cute::conditional_t<IsVarLen_, cutlass::fmha::collective::VariableLength, int>;
  int batch;
  int num_heads_q, num_heads_kv;
  SeqLenType seq_len_qo, seq_len_kv, seq_len_kv_cache;
  int head_size_qk, head_size_vo;
};
```
- **EN:** Defines templated type `FMHAProblemShape` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `FMHAProblemShape`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 59-59

```cpp
///////////////////////////////////////////////////////////////////////////////
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 61-62

```cpp
template <class ProblemShape_, class CollectiveMainloop_, class CollectiveEpilogue_, class TileScheduler_>
class XeFMHAFwdKernel {
```
- **EN:** Defines templated type `XeFMHAFwdKernel` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `XeFMHAFwdKernel`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 64-74

```cpp
public:
  //
  // Type Aliases
  //
  using ProblemShape = ProblemShape_;
  using VariableLength = cutlass::fmha::collective::VariableLength;
  static constexpr bool is_var_len = cutlass::fmha::collective::is_variable_length_v<typename ProblemShape::SeqLenType>;
  // Mainloop derived types
  using CollectiveMainloop = CollectiveMainloop_;
  using MainloopArguments = typename CollectiveMainloop::Arguments;
  using MainloopParams = typename CollectiveMainloop::Params;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 76-83

```cpp
  using TiledMMAQK = typename CollectiveMainloop::TiledMMAQK;
  using TiledMMAPV = typename CollectiveMainloop::TiledMMAPV;
  using TileShapeQK = typename CollectiveMainloop::TileShapeQK;
  using TileShapePV = typename CollectiveMainloop::TileShapePV;
  using SubgroupLayoutQK = typename CollectiveMainloop::SubgroupLayoutQK;
  using ElementQ = typename CollectiveMainloop::TensorQ::element_type;
  using ElementK = typename CollectiveMainloop::TensorK::element_type;
  using ElementV = typename CollectiveMainloop::TensorV::element_type;
```
- **EN:** Defines aliases such as `TiledMMAQK`, `TiledMMAPV`, `TileShapeQK`, `TileShapePV`, `SubgroupLayoutQK`, `ElementQ`, ... (+2) to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TiledMMAQK`, `TiledMMAPV`, `TileShapeQK`, `TileShapePV`, `SubgroupLayoutQK`, `ElementQ`, ... (+2)，用于简化冗长的模板表达式或命名空间限定。

### Lines 85-87

```cpp
  using StrideQ = decltype(stride(typename CollectiveMainloop::TensorQ{}));
  using StrideK = decltype(stride(typename CollectiveMainloop::TensorK{}));
  using StrideV = decltype(stride(typename CollectiveMainloop::TensorV{}));
```
- **EN:** Defines aliases such as `StrideQ`, `StrideK`, `StrideV` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `StrideQ`, `StrideK`, `StrideV`，用于简化冗长的模板表达式或命名空间限定。

### Lines 89-89

```cpp
  using SGPerWG = typename CollectiveMainloop::SGPerWG;
```
- **EN:** Defines aliases such as `SGPerWG` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `SGPerWG`，用于简化冗长的模板表达式或命名空间限定。

### Lines 91-92

```cpp
  using FragA = typename CollectiveMainloop::FragA;
  using FragARow = typename CollectiveMainloop::FragARow;
```
- **EN:** Defines aliases such as `FragA`, `FragARow` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `FragA`, `FragARow`，用于简化冗长的模板表达式或命名空间限定。

### Lines 94-96

```cpp
  // Tile scheduler derived types
  using TileScheduler = TileScheduler_;
  using TileSchedulerParams = typename TileScheduler::Params;
```
- **EN:** Defines aliases such as `TileScheduler`, `TileSchedulerParams` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TileScheduler`, `TileSchedulerParams`，用于简化冗长的模板表达式或命名空间限定。

### Lines 98-101

```cpp
  // Epilogue derived types
  using CollectiveEpilogue = CollectiveEpilogue_;
  using EpilogueArguments = typename CollectiveEpilogue::Arguments;
  using EpilogueParams = typename CollectiveEpilogue::Params;
```
- **EN:** Defines aliases such as `CollectiveEpilogue`, `EpilogueArguments`, `EpilogueParams` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveEpilogue`, `EpilogueArguments`, `EpilogueParams`，用于简化冗长的模板表达式或命名空间限定。

### Lines 103-105

```cpp
  using TileShapeO = typename CollectiveEpilogue::TileShapeO;
  using ElementO = typename CollectiveEpilogue::TensorO::element_type;
  using StrideO = decltype(stride(typename CollectiveEpilogue::TensorO{}));
```
- **EN:** Defines aliases such as `TileShapeO`, `ElementO`, `StrideO` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TileShapeO`, `ElementO`, `StrideO`，用于简化冗长的模板表达式或命名空间限定。

### Lines 107-113

```cpp
  // Kernel level shared memory storage
  using MainloopSharedStorage = typename CollectiveMainloop::SharedStorage;
  using EpilogueSharedStorage = typename CollectiveEpilogue::SharedStorage;
  union SharedStorage {
    MainloopSharedStorage mainloop;
    EpilogueSharedStorage epilogue;
  };
```
- **EN:** Declares union `SharedStorage` to overlay shared-storage views or related low-level buffers.
- **CN:** 声明联合体 `SharedStorage`，用于复用共享存储视图或相关底层缓冲区。

### Lines 115-116

```cpp
  static constexpr int SharedStorageSize = is_empty_v<SharedStorage> ? size_t(0)
                                                                     : sizeof(SharedStorage);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 118-134

```cpp
  // Device side arguments
  struct KernelArguments {
    ProblemShape shape;
    const ElementQ *Q;
    StrideQ dQ;
    const ElementK *K;
    StrideK dK;
    const ElementV *V;
    StrideV dV;
    ElementO *O;
    StrideO dO;
    const ElementK *K_cache;
    StrideK dK_cache{};
    const ElementV *V_cache;
    StrideV dV_cache{};
  };
  using KernelParams = KernelArguments;
```
- **EN:** Declares `KernelArguments` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `KernelArguments` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 136-141

```cpp
  struct Arguments {
    KernelArguments kernel{};
    MainloopArguments mainloop{};
    EpilogueArguments epilogue{};
    KernelHardwareInfo hw_info{};
  };
```
- **EN:** Declares `Arguments` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Arguments` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 143-149

```cpp
  // Kernel entry point API
  struct Params {
    KernelParams kernel;
    MainloopParams mainloop;
    EpilogueParams epilogue;
    TileSchedulerParams scheduler;
  };
```
- **EN:** Declares `Params` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Params` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 151-153

```cpp
  //
  // Methods
  //
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 155-160

```cpp
  static Params to_underlying_arguments(Arguments const &args, void *workspace) {
    return {args.kernel,
            CollectiveMainloop::to_underlying_arguments(args.mainloop, workspace),
            CollectiveEpilogue::to_underlying_arguments(args.epilogue, workspace),
            TileScheduler::to_underlying_arguments(args.kernel.shape, args.hw_info, TileShapeO{})};
  }
```
- **EN:** Converts the public argument bundle into lower-level parameter objects consumed by the kernel components.
- **CN:** 把公开参数包转换为内核组件实际使用的底层参数对象。

### Lines 162-165

```cpp
  static bool can_implement(Arguments const &args) {
    return CollectiveMainloop::can_implement(args.mainloop)
        && CollectiveEpilogue::can_implement(args.epilogue);
  }
```
- **EN:** Checks whether the selected configuration is supported before kernel launch.
- **CN:** 在启动内核之前检查所选配置是否受支持。

### Lines 167-167

```cpp
  static int get_workspace_size(Arguments const &args) { return 0; }
```
- **EN:** Reports how much auxiliary workspace this specialization needs.
- **CN:** 报告该特化所需的辅助工作空间大小。

### Lines 169-172

```cpp
  static cutlass::Status initialize_workspace(Arguments const &args, void *workspace = nullptr,
                                              cudaStream_t stream = nullptr, CudaHostAdapter *cuda_adapter = nullptr) {
    return Status::kSuccess;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 174-176

```cpp
  static dim3 get_grid_shape(Params const &params) {
    return TileScheduler::template get_grid_shape<SGPerWG::value>(params.scheduler);
  }
```
- **EN:** Computes the launch grid from scheduler parameters and tiling choices.
- **CN:** 根据调度参数和分块选择计算启动网格。

### Lines 178-178

```cpp
  static dim3 get_block_shape() { return dim3(SGPerWG::value * intel::sg_size, 1, 1); }
```
- **EN:** Returns the thread-block shape used to launch the kernel on the device.
- **CN:** 返回在设备上启动该内核时使用的线程块形状。

### Lines 180-187

```cpp
  CUTLASS_DEVICE
  Shape<int, int, int> get_sequence_length_shape(ProblemShape const& problem_shape, int const& batch) {
    if constexpr (is_var_len) {
      return cutlass::fmha::collective::apply_variable_length(Shape<VariableLength, VariableLength, VariableLength>{problem_shape.seq_len_qo, problem_shape.seq_len_kv, problem_shape.seq_len_kv_cache}, batch);
    } else {
      return Shape<int, int, int>{problem_shape.seq_len_qo, problem_shape.seq_len_kv, problem_shape.seq_len_kv_cache};
    }
  }
```
- **EN:** Resolves per-batch sequence-length metadata, including the variable-length case.
- **CN:** 解析每个 batch 的序列长度元数据，包括变长场景。

### Lines 189-192

```cpp
  CUTLASS_DEVICE
  void operator()(Params const &params, char *smem_buf)
  {
    using namespace sycl::ext::oneapi::this_work_item;
```
- **EN:** Implements the core device-side operator that walks tiles, moves data, and performs attention math.
- **CN:** 实现核心设备端调用算子，负责遍历 tile、搬运数据并执行注意力计算。

### Lines 194-194

```cpp
    SharedStorage& shared_storage = *reinterpret_cast<SharedStorage *>(smem_buf);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 196-198

```cpp
    auto &p = params.kernel;
    ProblemShape const& s = p.shape;
    int head_group_q = s.num_heads_q / s.num_heads_kv;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 200-202

```cpp
    int thr_id = int(ThreadIdxX());
    int sub_group_id = thr_id / intel::sg_size;
    int q_sg_tile = get<0>(shape_div(TileShapeQK{}, shape(SubgroupLayoutQK{})));
```
- **EN:** Selects the subgroup lane and copy primitive used for the upcoming data movement step.
- **CN:** 选择子组 lane 以及后续数据搬运要使用的复制原语。

### Lines 204-207

```cpp
    auto cS = make_identity_tensor(take<0,2>(TiledMMAQK{}.tile_mnk()));
    auto tScS = TiledMMAQK{}.get_slice(thr_id).partition_C(cS);
    auto q_offset_wi = get<0>(tScS(0));
    auto q_offset_sg = group_broadcast(sycl::ext::oneapi::this_work_item::get_sub_group(), q_offset_wi, 0);
```
- **EN:** Defines MMA-related helpers or specializations used by Xe matrix instructions.
- **CN:** 定义与 MMA 相关的辅助逻辑或特化，用于 Xe 矩阵指令。

### Lines 209-209

```cpp
    TileScheduler tile_scheduler{params.scheduler};
```
- **EN:** Describes scheduling logic that maps FlashAttention tiles onto work-groups or subgroups.
- **CN:** 描述将 FlashAttention tile 映射到工作组或子组的调度逻辑。

### Lines 211-215

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for (; tile_scheduler.is_valid(); ++tile_scheduler) {
      auto [blk_q, blk_v, head_q, idx_b] = tile_scheduler.get_block_coord(); // (Q,V,h,b)
      auto blk_qv = make_coord(blk_q, blk_v);
      int head = head_q / head_group_q;
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 217-219

```cpp
      auto sequence_length_shape = get_sequence_length_shape(s, idx_b);
      auto [seq_len_qo, seq_len_kv, seq_len_kv_cache] = sequence_length_shape;
      if (blk_q * get<0>(TileShapeQK{}) >= seq_len_qo) continue;
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 221-224

```cpp
      auto offset = cute::min(seq_len_qo, seq_len_kv);
      auto discard_seq_coord = seq_len_qo - offset;
      auto full_tile_offset = seq_len_kv - offset;
      int seq_coord = cute::min(seq_len_qo, (blk_q * get<0>(TileShapeQK{}) + q_offset_sg));
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 226-229

```cpp
      if (CollectiveMainloop::CausalMask && seq_coord < discard_seq_coord) continue;
      const int seq_len_new = CollectiveMainloop::CausalMask ? full_tile_offset + cute::min(seq_len_kv, seq_coord - discard_seq_coord) + q_sg_tile : seq_len_kv;
      const int seq_len = seq_len_new + seq_len_kv_cache;
      const int k_blocks = cute::ceil_div(seq_len, get<1>(TileShapeQK{}));
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 231-245

```cpp
      int offset_q = 0, offset_k = 0, offset_v = 0, offset_o = 0;
      int offset_k_cache = 0, offset_v_cache = 0;
      if constexpr (is_var_len) {
        auto qo_cumulative = s.seq_len_qo.cumulative_length;
        auto kv_cumulative = s.seq_len_kv.cumulative_length;
        offset_q = s.num_heads_q * s.head_size_qk * qo_cumulative[idx_b];
        offset_k = s.num_heads_kv * s.head_size_qk * kv_cumulative[idx_b];
        offset_v = s.num_heads_kv * s.head_size_vo * kv_cumulative[idx_b];
        offset_o = s.num_heads_q * s.head_size_vo * qo_cumulative[idx_b];
        if (s.seq_len_kv_cache.cumulative_length) {
          auto kv_cumulative_cache = s.seq_len_kv_cache.cumulative_length;
          offset_k_cache = s.num_heads_kv * s.head_size_qk * kv_cumulative_cache[idx_b];
          offset_v_cache = s.num_heads_kv * s.head_size_vo * kv_cumulative_cache[idx_b];
        }
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 247-251

```cpp
      auto batch_dim = is_var_len ? 1 : s.batch;
      auto shape_Q = make_shape(seq_len_qo, s.head_size_qk, s.num_heads_q, batch_dim);
      auto shape_K = make_shape(seq_len_kv, s.head_size_qk, s.num_heads_kv, batch_dim);
      auto shape_V = make_shape(s.head_size_vo, seq_len_kv, s.num_heads_kv, batch_dim);
      auto shape_O = make_shape(seq_len_qo, s.head_size_vo, s.num_heads_q, batch_dim);
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 253-254

```cpp
      auto shape_K_cache = make_shape(seq_len_kv_cache, s.head_size_qk, s.num_heads_kv, batch_dim);
      auto shape_V_cache = make_shape(s.head_size_vo, seq_len_kv_cache, s.num_heads_kv, batch_dim);
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 256-261

```cpp
      auto dcQ = const_cast<ElementQ*>(p.Q + offset_q);
      auto dcK = const_cast<ElementK*>(p.K + offset_k);
      auto dcV = const_cast<ElementV*>(p.V + offset_v);
      auto dcK_cache = const_cast<ElementK*>(p.K_cache + offset_k_cache);
      auto dcV_cache = const_cast<ElementV*>(p.V_cache + offset_v_cache);
      auto ptrO = p.O + offset_o;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 263-268

```cpp
      auto stride_q = is_var_len ? cutlass::make_cute_packed_stride(StrideQ{}, shape_Q) : p.dQ;
      auto stride_k = is_var_len ? cutlass::make_cute_packed_stride(StrideK{}, shape_K) : p.dK;
      auto stride_v = is_var_len ? cutlass::make_cute_packed_stride(StrideV{}, shape_V) : p.dV;
      auto stride_o = is_var_len ? cutlass::make_cute_packed_stride(StrideO{}, shape_O) : p.dO;
      auto stride_k_cache = is_var_len ? cutlass::make_cute_packed_stride(StrideK{}, shape_K_cache) : p.dK_cache;
      auto stride_v_cache = is_var_len ? cutlass::make_cute_packed_stride(StrideV{}, shape_V_cache) : p.dV_cache;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 270-275

```cpp
      Tensor Q = make_tensor(make_gmem_ptr(dcQ), make_layout(shape_Q, stride_q));
      Tensor K = make_tensor(make_gmem_ptr(dcK), make_layout(shape_K, stride_k));
      Tensor V = make_tensor(make_gmem_ptr(dcV), make_layout(shape_V, stride_v));
      Tensor K_cache = make_tensor(make_gmem_ptr(dcK_cache), make_layout(shape_K_cache, stride_k_cache));
      Tensor V_cache = make_tensor(make_gmem_ptr(dcV_cache), make_layout(shape_V_cache, stride_v_cache));
      Tensor O = make_tensor(make_gmem_ptr(ptrO), make_layout(shape_O, stride_o));
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 278-280

```cpp
      // O accumulator types
      FragA tArA;
      FragARow tA_max, tA_sum;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 282-293

```cpp
      // Main loop
      int l_coord = is_var_len ? 0 : idx_b;
      CollectiveMainloop mainloop(params.mainloop, shared_storage.mainloop);
      mainloop(Q(_,_,head_q,l_coord),
               K(_,_,head,l_coord),
               V(_,_,head,l_coord),
               tArA, tA_max, tA_sum,
               blk_qv, 0, k_blocks, k_blocks,
               thr_id, seq_len, seq_len_kv_cache, idx_b,
               full_tile_offset, discard_seq_coord,
               K_cache(_,_,head,l_coord),
               V_cache(_,_,head,l_coord));
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 295-297

```cpp
      if constexpr (!is_empty_v<MainloopSharedStorage> && !is_empty_v<EpilogueSharedStorage>) {
        sycl::group_barrier(get_work_group<3>());
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 299-306

```cpp
      // Epilogue
      CollectiveEpilogue epilogue{params.epilogue, shared_storage.epilogue};
      epilogue(O(_,_,head_q,l_coord),
               tArA, tA_max, tA_sum,
               blk_qv, thr_id);
    }
  }
};
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 308-309

```cpp
template <class ProblemShape_, class CollectiveMainloop_, class CollectiveEpilogue_, class TileScheduler_>
class XeFMHAFwdDynamicSplitKernel {
```
- **EN:** Defines templated type `XeFMHAFwdDynamicSplitKernel` that packages policy, tile, datatype, or layout choices into a reusable configuration.
- **CN:** 定义模板类型 `XeFMHAFwdDynamicSplitKernel`，把策略、tile、数据类型或布局选择打包成可复用配置。

### Lines 311-315

```cpp
public:
  //
  // Type Aliases
  //
  using ProblemShape = ProblemShape_;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 317-320

```cpp
  // Mainloop derived types
  using CollectiveMainloop = CollectiveMainloop_;
  using MainloopArguments = typename CollectiveMainloop::Arguments;
  using MainloopParams = typename CollectiveMainloop::Params;
```
- **EN:** Defines aliases such as `CollectiveMainloop`, `MainloopArguments`, `MainloopParams` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveMainloop`, `MainloopArguments`, `MainloopParams`，用于简化冗长的模板表达式或命名空间限定。

### Lines 322-325

```cpp
  using TiledMMAQK = typename CollectiveMainloop::TiledMMAQK;
  using TiledMMAPV = typename CollectiveMainloop::TiledMMAPV;
  using TileShapeQK = typename CollectiveMainloop::TileShapeQK;
  using TileShapePV = typename CollectiveMainloop::TileShapePV;
```
- **EN:** Defines aliases such as `TiledMMAQK`, `TiledMMAPV`, `TileShapeQK`, `TileShapePV` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TiledMMAQK`, `TiledMMAPV`, `TileShapeQK`, `TileShapePV`，用于简化冗长的模板表达式或命名空间限定。

### Lines 327-329

```cpp
  using ElementQ = typename CollectiveMainloop::TensorQ::element_type;
  using ElementK = typename CollectiveMainloop::TensorK::element_type;
  using ElementV = typename CollectiveMainloop::TensorV::element_type;
```
- **EN:** Defines aliases such as `ElementQ`, `ElementK`, `ElementV` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `ElementQ`, `ElementK`, `ElementV`，用于简化冗长的模板表达式或命名空间限定。

### Lines 331-333

```cpp
  using StrideQ = decltype(stride(typename CollectiveMainloop::TensorQ{}));
  using StrideK = decltype(stride(typename CollectiveMainloop::TensorK{}));
  using StrideV = decltype(stride(typename CollectiveMainloop::TensorV{}));
```
- **EN:** Defines aliases such as `StrideQ`, `StrideK`, `StrideV` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `StrideQ`, `StrideK`, `StrideV`，用于简化冗长的模板表达式或命名空间限定。

### Lines 335-335

```cpp
  using SGPerWG = typename CollectiveMainloop::SGPerWG;
```
- **EN:** Defines aliases such as `SGPerWG` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `SGPerWG`，用于简化冗长的模板表达式或命名空间限定。

### Lines 337-341

```cpp
  using FragA = typename CollectiveMainloop::FragA;
  using SingleFragA = typename CollectiveMainloop::SingleFragA;
  using FragARow = typename CollectiveMainloop::FragARow;
  // element dtype for MmaPV results
  using ElementA = typename CollectiveMainloop::ElementA;
```
- **EN:** Defines aliases such as `FragA`, `SingleFragA`, `FragARow`, `ElementA` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `FragA`, `SingleFragA`, `FragARow`, `ElementA`，用于简化冗长的模板表达式或命名空间限定。

### Lines 343-346

```cpp
  // Tile scheduler derived types
  static_assert(is_same_v<TileScheduler_, XeFHMAIndividualPersistentTileScheduler>);
  using TileScheduler = TileScheduler_;
  using TileSchedulerParams = typename TileScheduler::Params;
```
- **EN:** Adds a compile-time constraint so unsupported combinations fail early during template instantiation.
- **CN:** 添加编译期约束，使不受支持的组合在模板实例化阶段及早失败。

### Lines 348-351

```cpp
  // Epilogue derived types
  using CollectiveEpilogue = CollectiveEpilogue_;
  using EpilogueArguments = typename CollectiveEpilogue::Arguments;
  using EpilogueParams = typename CollectiveEpilogue::Params;
```
- **EN:** Defines aliases such as `CollectiveEpilogue`, `EpilogueArguments`, `EpilogueParams` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `CollectiveEpilogue`, `EpilogueArguments`, `EpilogueParams`，用于简化冗长的模板表达式或命名空间限定。

### Lines 353-355

```cpp
  using TileShapeO = typename CollectiveEpilogue::TileShapeO;
  using ElementO = typename CollectiveEpilogue::TensorO::element_type;
  using StrideO = decltype(stride(typename CollectiveEpilogue::TensorO{}));
```
- **EN:** Defines aliases such as `TileShapeO`, `ElementO`, `StrideO` to shorten verbose template expressions or namespace qualifiers.
- **CN:** 定义别名，例如 `TileShapeO`, `ElementO`, `StrideO`，用于简化冗长的模板表达式或命名空间限定。

### Lines 357-363

```cpp
  // Kernel level shared memory storage
  using MainloopSharedStorage = typename CollectiveMainloop::SharedStorage;
  using EpilogueSharedStorage = typename CollectiveEpilogue::SharedStorage;
  union SharedStorage {
    MainloopSharedStorage mainloop;
    EpilogueSharedStorage epilogue;
  };
```
- **EN:** Declares union `SharedStorage` to overlay shared-storage views or related low-level buffers.
- **CN:** 声明联合体 `SharedStorage`，用于复用共享存储视图或相关底层缓冲区。

### Lines 365-366

```cpp
  static constexpr int SharedStorageSize = is_empty_v<SharedStorage> ? size_t(0)
                                                                     : sizeof(SharedStorage);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 368-371

```cpp
  // Important: make sure multiple of 16 element for each copy
  // this is for storing partial results from different KV partitions
  static constexpr int num_elem_per_thread = (size(FragA{}.shape()) + 2 * size(FragARow{}.shape()) + 15) / 16 * 16;
  static const int max_num_partitions = 8;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 373-389

```cpp
  // Device side arguments
  struct KernelArguments {
    ProblemShape shape;
    const ElementQ *Q;
    StrideQ dQ;
    const ElementK *K;
    StrideK dK;
    const ElementV *V;
    StrideV dV;
    ElementO *O;
    StrideO dO;
    const ElementK *K_cache = nullptr;
    StrideK dK_cache{};
    const ElementV *V_cache = nullptr;
    StrideV dV_cache{};
  };
  using KernelParams = KernelArguments;
```
- **EN:** Declares `KernelArguments` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `KernelArguments` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 391-396

```cpp
  struct Arguments {
    KernelArguments kernel{};
    MainloopArguments mainloop{};
    EpilogueArguments epilogue{};
    KernelHardwareInfo hw_info{};
  };
```
- **EN:** Declares `Arguments` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Arguments` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 398-408

```cpp
  // Kernel entry point API
  struct Params {
    KernelParams kernel;
    MainloopParams mainloop;
    EpilogueParams epilogue;
    TileSchedulerParams scheduler;
    // workspace for storing partial results of different KV partitions
    ElementA *partial_results_ptr = nullptr;
    // for atomic add
    int32_t *atomic_reduce_cnt_ptr = nullptr;
  };
```
- **EN:** Declares `Params` as a data structure that groups related arguments, parameters, or helper state.
- **CN:** 声明 `Params` 数据结构，用于组织相关参数、配置或辅助状态。

### Lines 410-412

```cpp
  //
  // Methods
  //
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 414-424

```cpp
  static Params to_underlying_arguments(Arguments const &args, void *workspace) {
    int num_batch_heads = args.kernel.shape.batch * args.kernel.shape.num_heads_q;
    int32_t *atomic_reduce_cnt_ptr = reinterpret_cast<int32_t *>(workspace);
    ElementA *partial_results_ptr = reinterpret_cast<ElementA *>(atomic_reduce_cnt_ptr + num_batch_heads);
    return {args.kernel,
            CollectiveMainloop::to_underlying_arguments(args.mainloop, workspace),
            CollectiveEpilogue::to_underlying_arguments(args.epilogue, workspace),
            TileScheduler::to_underlying_arguments(args.kernel.shape, args.hw_info, TileShapeO{}),
            partial_results_ptr, atomic_reduce_cnt_ptr
          };
  }
```
- **EN:** Converts the public argument bundle into lower-level parameter objects consumed by the kernel components.
- **CN:** 把公开参数包转换为内核组件实际使用的底层参数对象。

### Lines 426-437

```cpp
  static bool can_implement(Arguments const &args) {
    // current kernel only support decode
    if (args.kernel.shape.seq_len_qo > 1) {
      return false;
    }
    // current kernel only support num batch heads less than total XeCore count
    if (args.kernel.shape.batch * args.kernel.shape.num_heads_q > args.hw_info.sm_count) {
      return false;
    }
    return CollectiveMainloop::can_implement(args.mainloop)
        && CollectiveEpilogue::can_implement(args.epilogue);
  }
```
- **EN:** Checks whether the selected configuration is supported before kernel launch.
- **CN:** 在启动内核之前检查所选配置是否受支持。

### Lines 439-442

```cpp
  static int get_workspace_size(Arguments const &args) {
    int ws_size = 0;
    int num_batch_heads = args.kernel.shape.batch * args.kernel.shape.num_heads_q;
    const int wg_size = SGPerWG::value * intel::sg_size;
```
- **EN:** Reports how much auxiliary workspace this specialization needs.
- **CN:** 报告该特化所需的辅助工作空间大小。

### Lines 444-449

```cpp
    // partial attn outputs, exp sum and max logits
    ws_size += (max_num_partitions * num_batch_heads) * wg_size * num_elem_per_thread * sizeof(ElementA);
    // atomic counter
    ws_size += num_batch_heads * sizeof(int32_t);
    return ws_size;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 451-459

```cpp
  static cutlass::Status initialize_workspace(Arguments const &args, void *workspace = nullptr,
                                              cudaStream_t stream = nullptr, CudaHostAdapter *cuda_adapter = nullptr) {
    int num_batch_heads = args.kernel.shape.batch * args.kernel.shape.num_heads_q;
    compat::fill(reinterpret_cast<int32_t*>(workspace), (int32_t)0, num_batch_heads);
    auto partial_ws_count = (get_workspace_size(args) - num_batch_heads * sizeof(int32_t)) / sizeof(ElementA);
    auto* partial_results_ptr = reinterpret_cast<ElementA*>(reinterpret_cast<int32_t*>(workspace) + num_batch_heads);
    compat::fill(partial_results_ptr, (ElementA)0, partial_ws_count);
    return Status::kSuccess;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 461-463

```cpp
  static dim3 get_grid_shape(Params const &params) {
    return TileScheduler::template get_grid_shape<SGPerWG::value>(params.scheduler);
  }
```
- **EN:** Computes the launch grid from scheduler parameters and tiling choices.
- **CN:** 根据调度参数和分块选择计算启动网格。

### Lines 465-465

```cpp
  static dim3 get_block_shape() { return dim3(SGPerWG::value * intel::sg_size, 1, 1); }
```
- **EN:** Returns the thread-block shape used to launch the kernel on the device.
- **CN:** 返回在设备上启动该内核时使用的线程块形状。

### Lines 467-476

```cpp
  CUTLASS_DEVICE
  int get_partition_id(const int cur_wg_id, const int batch_head_id, const int num_blocks_per_wg, const int local_k_blocks) {
    int partition_id = 0;
    if (batch_head_id == 0) {
      return cur_wg_id;
    }
    int start_wg_id = batch_head_id * local_k_blocks / num_blocks_per_wg;
    partition_id = cur_wg_id - start_wg_id;
    return partition_id;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 478-489

```cpp
  CUTLASS_DEVICE
  int get_num_partitions(const int batch_head_id, const int num_blocks_per_wg, const int local_k_blocks) {
    int num_partitions = 1;
    int start_wg_id = batch_head_id * local_k_blocks / num_blocks_per_wg;
    int end_wg_id = (batch_head_id + 1) * local_k_blocks / num_blocks_per_wg;
    num_partitions = end_wg_id - start_wg_id + 1;
    // end_wg_id is the starting wg id of next batch head id
    if (((batch_head_id + 1) * local_k_blocks) % num_blocks_per_wg == 0) {
      num_partitions -= 1;
    }
    return num_partitions;
  }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 491-496

```cpp
  template <class Params, class FragA, class FragARow>
  CUTLASS_DEVICE
  void reduce_split2(const Params &params, FragA &out1, FragARow& max_val1, FragARow& exp_sum_val1, FragA &out2, FragARow& max_val2, FragARow& exp_sum_val2) {
    // global max value
    FragARow max_prev1 = max_val1;
    FragARow max_prev2 = max_val2;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 498-498

```cpp
    auto scale = params.mainloop.scale;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 500-503

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < max_val1.size(); i++) {
      max_val1(i) = sycl::max(max_val1(i), max_val2(i));
    }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 505-510

```cpp
    FragARow rescale1, rescale2;
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < max_val1.size(); i++) {
      rescale1(i) = sycl::native::exp2(max_prev1(i) - max_val1(i));
      rescale2(i) = sycl::native::exp2(max_prev2(i) - max_val1(i));
    }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 512-515

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < exp_sum_val1.size(); i++) {
      exp_sum_val1(i) = exp_sum_val1(i) * rescale1(i) + exp_sum_val2(i) * rescale2(i);
    }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 517-520

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < out1.size(); i++)
      out1(i) = out1(i) * broadcast<0>(rescale1, out1, i) + out2(i) * broadcast<0>(rescale2, out2, i);
  }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 522-525

```cpp
  CUTLASS_DEVICE
  void operator()(Params const &params, char *smem_buf)
  {
    using namespace sycl::ext::oneapi::this_work_item;
```
- **EN:** Implements the core device-side operator that walks tiles, moves data, and performs attention math.
- **CN:** 实现核心设备端调用算子，负责遍历 tile、搬运数据并执行注意力计算。

### Lines 527-527

```cpp
    SharedStorage& shared_storage = *reinterpret_cast<SharedStorage *>(smem_buf);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 529-531

```cpp
    auto &p = params.kernel;
    ProblemShape const& s = p.shape;
    int head_group_q = s.num_heads_q / s.num_heads_kv;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 533-534

```cpp
    int thr_id = int(ThreadIdxX());
    int wg_id = int(BlockIdxZ());
```
- **EN:** Selects the subgroup lane and copy primitive used for the upcoming data movement step.
- **CN:** 选择子组 lane 以及后续数据搬运要使用的复制原语。

### Lines 536-538

```cpp
    int sg_id = thr_id / intel::sg_size;
    int tid_in_sg = thr_id % intel::sg_size;
    int num_batch_heads = s.batch * s.num_heads_q;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 540-544

```cpp
    int local_k_blocks = cute::ceil_div(s.seq_len_kv, get<1>(TileShapeQK{}));
    // total number of blocks need to be processed across all wgs
    int total_k_blocks = local_k_blocks * num_batch_heads;
    // to guarantee all wg process similar number of blocks of KV
    int num_blocks_per_wg = cute::ceil_div(total_k_blocks, GridDimZ());
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 546-546

```cpp
    TileScheduler tile_scheduler{params.scheduler, get<1>(TileShapeQK{}), local_k_blocks, num_batch_heads};
```
- **EN:** Describes scheduling logic that maps FlashAttention tiles onto work-groups or subgroups.
- **CN:** 描述将 FlashAttention tile 映射到工作组或子组的调度逻辑。

### Lines 548-551

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for (; tile_scheduler.is_valid(); ++tile_scheduler) {
      auto [blk_q, blk_v, start_batch_head_id] = tile_scheduler.get_block_coord(); // (Q,V, batch_head_idx)
      auto blk_qv = make_coord(blk_q, blk_v);
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 553-556

```cpp
      auto shape_Q = make_shape(s.seq_len_qo, s.head_size_qk, s.num_heads_q,  s.batch);
      auto shape_K = make_shape(s.seq_len_kv, s.head_size_qk, s.num_heads_kv, s.batch);
      auto shape_V = make_shape(s.head_size_vo, s.seq_len_kv, s.num_heads_kv, s.batch);
      auto shape_O = make_shape(s.seq_len_qo, s.head_size_vo, s.num_heads_q, s.batch);
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 558-560

```cpp
      auto dcQ = const_cast<ElementQ*>(p.Q);  // de-const these for uniformity
      auto dcK = const_cast<ElementK*>(p.K);
      auto dcV = const_cast<ElementV*>(p.V);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 562-565

```cpp
      Tensor Q = make_tensor(make_gmem_ptr(dcQ), make_layout(shape_Q, p.dQ));    // (q,d,h,b)
      Tensor K = make_tensor(make_gmem_ptr(dcK), make_layout(shape_K, p.dK));    // (k,d,h,b)
      Tensor V = make_tensor(make_gmem_ptr(dcV), make_layout(shape_V, p.dV));    // (v,k,h,b)
      Tensor O = make_tensor(make_gmem_ptr(p.O), make_layout(shape_O, p.dO));    // (q,v,h,b)
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 567-572

```cpp
      auto shape_K_cache = make_shape(s.seq_len_kv_cache, s.head_size_qk, s.num_heads_kv, s.batch);
      auto shape_V_cache = make_shape(s.head_size_vo, s.seq_len_kv_cache, s.num_heads_kv, s.batch);
      auto dcK_cache = const_cast<ElementK*>(p.K_cache);
      auto dcV_cache = const_cast<ElementV*>(p.V_cache);
      Tensor K_cache = make_tensor(make_gmem_ptr(dcK_cache), make_layout(shape_K_cache, p.dK_cache));
      Tensor V_cache = make_tensor(make_gmem_ptr(dcV_cache), make_layout(shape_V_cache, p.dV_cache));
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 574-576

```cpp
      // O accumulator types
      FragA tArA;
      FragARow tA_max, tA_sum;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 578-583

```cpp
      // compute num computed blocks for start batch head id
      int num_computed_blocks = (start_batch_head_id == 0) ? (wg_id * num_blocks_per_wg) : (wg_id * num_blocks_per_wg - start_batch_head_id * local_k_blocks);
      int start_blk, end_blk, head_q, idx_b, head_kv;
      // leader wg is also responsible for reducing partial results, while other
      // worker wg only to compute partial results
      bool is_leader_wg = wg_id < num_batch_heads;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 585-588

```cpp
      if (thr_id == 0 && is_leader_wg) {
        // reset atomic counter before computation
        *(params.atomic_reduce_cnt_ptr + wg_id) = 0;
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 590-591

```cpp
      // Main loop
      CollectiveMainloop mainloop(params.mainloop, shared_storage.mainloop);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 593-602

```cpp
      // compute blocks budget remained for each wg
      int block_budget_remained = num_blocks_per_wg;
      int batch_head_id = start_batch_head_id;
      bool is_update_batch_head_id = false;
      while (block_budget_remained > 0) {
        int num_new_blocks = local_k_blocks - num_computed_blocks;
        if (num_new_blocks <= block_budget_remained) {
          // finished current batch head id
          start_blk = num_computed_blocks;
          end_blk = start_blk + num_new_blocks;
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 604-611

```cpp
          // update states
          num_computed_blocks = 0;
          block_budget_remained -= num_new_blocks;
          is_update_batch_head_id = true;
        } else {
          // budget cannot afford finishing current batch head id
          start_blk = num_computed_blocks;
          end_blk = start_blk + block_budget_remained;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 613-615

```cpp
          block_budget_remained = 0;
          is_update_batch_head_id = false;
        }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 617-626

```cpp
        head_q = batch_head_id % s.num_heads_q;
        idx_b = batch_head_id / s.num_heads_q;
        head_kv = head_q / head_group_q;
        // mainloop
        mainloop(Q(_,_,head_q,idx_b),
              K(_,_,head_kv,idx_b),
              V(_,_,head_kv,idx_b),
              tArA, tA_max, tA_sum,
              blk_qv, start_blk, end_blk, local_k_blocks,
              thr_id, s.seq_len_kv, 0, 0, 0, 0);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 628-629

```cpp
        // partition id of start batch head id in current wg
        int partition_id = get_partition_id(wg_id, batch_head_id, num_blocks_per_wg, local_k_blocks);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 631-637

```cpp
        // store partial result: tArA, tA_max and tA_sum
        int offset = batch_head_id * max_num_partitions * num_elem_per_thread * SGPerWG::value * intel::sg_size
                    + partition_id * num_elem_per_thread * SGPerWG::value * intel::sg_size
                    + sg_id * intel::sg_size * num_elem_per_thread
                    + tid_in_sg * num_elem_per_thread;
        Tensor tPartial = make_tensor(params.partial_results_ptr + offset, make_shape(Int<num_elem_per_thread>{}));
        Tensor merged_res = make_tensor<ElementA>(Int<num_elem_per_thread>{});
```
- **EN:** Builds tensor views and layouts so the same data can be accessed with the tile pattern expected by the kernel.
- **CN:** 构造张量视图与布局，使同一数据能够按内核期望的 tile 模式访问。

### Lines 639-648

```cpp
        CUTLASS_PRAGMA_UNROLL
        for(int i = 0; i < size(FragA{}.shape()); ++i) {
          merged_res(i) = tArA(i);
        }
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(FragARow{}.shape()); ++i) {
          merged_res(2 * i + size(FragA{}.shape())) = tA_max(i);
          merged_res(2 * i + 1 + size(FragA{}.shape())) = tA_sum(i);
        }
        copy(merged_res, tPartial);
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 650-653

```cpp
        // after store, set atomic cnt
        if (thr_id == 0) {
          atomicAdd(params.atomic_reduce_cnt_ptr + batch_head_id, 1);
        }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 655-662

```cpp
        // advance to next batch head id
        if (is_update_batch_head_id) {
          batch_head_id += 1;
          if (batch_head_id >= num_batch_heads) {
            break;
          }
        }
      }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 664-665

```cpp
      if (is_leader_wg) {
        int num_partitions = get_num_partitions(wg_id, num_blocks_per_wg, local_k_blocks);
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 667-668

```cpp
        // check atomic to wait for partial results ready
        while(atomicLoad(params.atomic_reduce_cnt_ptr + wg_id) != num_partitions) {}
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 670-672

```cpp
        clear(tArA);
        clear(tA_max);
        clear(tA_sum);
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 674-681

```cpp
        for (int i = 0; i < num_partitions; ++i) {
          int offset = wg_id * max_num_partitions * SGPerWG::value * intel::sg_size * num_elem_per_thread
                     + i * SGPerWG::value * intel::sg_size * num_elem_per_thread
                     + sg_id * intel::sg_size * num_elem_per_thread
                     + tid_in_sg * num_elem_per_thread;
          Tensor tPartial = make_tensor(params.partial_results_ptr + offset, make_shape(Int<num_elem_per_thread>{}));
          Tensor merged_res = make_tensor<ElementA>(Int<num_elem_per_thread>{});
          copy(tPartial, merged_res);
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 683-687

```cpp
          if (i == 0) {
            CUTLASS_PRAGMA_UNROLL
            for(int i = 0; i < size(FragA{}.shape()); ++i) {
              tArA(i) = merged_res(i);
            }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 689-693

```cpp
            CUTLASS_PRAGMA_UNROLL
            for (int i = 0; i < size(FragARow{}.shape()); ++i) {
              tA_max(i) = merged_res(2 * i + size(FragA{}.shape()));
              tA_sum(i) = merged_res(2 * i + 1 + size(FragA{}.shape()));
            }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 695-696

```cpp
            continue;
          }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 698-703

```cpp
          FragA tArA_2;
          FragARow tA_max_2, tA_sum_2;
          CUTLASS_PRAGMA_UNROLL
          for(int i = 0; i < size(FragA{}.shape()); ++i) {
            tArA_2(i) = merged_res(i);
          }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 705-709

```cpp
          CUTLASS_PRAGMA_UNROLL
          for (int i = 0; i < size(FragARow{}.shape()); ++i) {
            tA_max_2(i) = merged_res(2 * i + size(FragA{}.shape()));
            tA_sum_2(i) = merged_res(2 * i + 1 + size(FragA{}.shape()));
          }
```
- **EN:** Contains loop logic that iterates over tiles, stages, or benchmark registrations.
- **CN:** 包含循环逻辑，用于遍历 tile、流水级或 benchmark 注册项。

### Lines 711-712

```cpp
          reduce_split2(params, tArA, tA_max, tA_sum, tArA_2, tA_max_2, tA_sum_2);
        }
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 714-717

```cpp
        // require group barrier if using SLM
        if constexpr (!is_empty_v<MainloopSharedStorage> && !is_empty_v<EpilogueSharedStorage>) {
          sycl::group_barrier(get_work_group<3>());
        }
```
- **EN:** Applies conditional logic to select a specialization, handle boundary cases, or switch memory sources.
- **CN:** 应用条件逻辑以选择特化、处理边界情况或切换内存来源。

### Lines 719-721

```cpp
        head_q = wg_id % s.num_heads_q;
        idx_b = wg_id / s.num_heads_q;
        head_kv = head_q / head_group_q;
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 723-731

```cpp
        // Epilogue
        CollectiveEpilogue epilogue{params.epilogue, shared_storage.epilogue};
        epilogue(O(_,_,head_q,idx_b),
                tArA, tA_max, tA_sum,
                blk_qv, thr_id);
      }
    }
  }
};
```
- **EN:** Defines a reusable helper, type, or control block used by the FlashAttention implementation.
- **CN:** 定义 FlashAttention 实现中可复用的辅助逻辑、类型或控制块。

### Lines 733-733

```cpp
} // namespace cutlass::fmha::kernel
```
- **EN:** Opens or closes namespaces so the declarations live in the intended CUTLASS, CUTE, or benchmark scope.
- **CN:** 打开或关闭命名空间，使这些声明位于预期的 CUTLASS、CUTE 或 benchmark 作用域中。

## Key Concepts / 关键概念

- **EN:** Kernel orchestration: the file assembles problem shape, arguments, scheduling, and device execution.
- **CN:** 内核编排：该文件组合问题形状、参数、调度与设备端执行逻辑。
- **EN:** MMA tiling: matrix-multiply-accumulate building blocks are specialized for Xe subgroup execution.
- **CN:** MMA 分块：矩阵乘加构件针对 Xe 子组执行进行专门化。

## Dependencies / 依赖关系

- **EN:** Direct dependencies referenced here include `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_mainloop.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_epilogue.hpp`, `cute/util/type_traits.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, ... (+1).
- **CN:** 这里引用的直接依赖包括 `cutlass/cutlass.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/gemm.h`, `cutlass/kernel_hardware_info.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_mainloop.hpp`, `flash_attention_v2/collective/xe_fmha_fwd_epilogue.hpp`, `cute/util/type_traits.hpp`, `flash_attention_v2/collective/fmha_fusion.hpp`, ... (+1)。
- **EN:** Kernel-level code depends on collective components for math/data movement and on scheduler types for grid traversal.
- **CN:** 内核级代码依赖 collective 组件完成数学与数据搬运，并依赖调度器类型遍历网格。
