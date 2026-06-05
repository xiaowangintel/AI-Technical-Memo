# gather_gemm.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/52_hopper_gather_scatter_fusion/gather_gemm.hpp`  
**Purpose / 用途**: This header implements a Hopper warp-specialized GEMM kernel wrapper that fuses gather-based operand access into the CUTLASS mainloop and epilogue machinery. It adapts standard GEMM tensor views so A and B can be loaded through indirection instead of contiguous addressing. / 该头文件实现了一个 Hopper warp-specialized GEMM 内核封装，把基于 gather 的操作数访问融合进 CUTLASS 的 mainloop 与 epilogue 体系中。它会把标准 GEMM 张量视图改造成带间接寻址的形式，使 A 和 B 可以经由索引表而不是连续地址读取。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-136 — Template shell, type aliases, shared storage, and warp-group sizing

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

#include "cutlass/cutlass.h"
#include "cutlass/kernel_hardware_info.hpp"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/dispatch_policy.hpp"

#include "cute/tensor.hpp"

#include "gather_tensor.hpp"

namespace cutlass {
  ///Forward declaration
  struct CudaHostAdapter;
}

namespace cutlass::gemm::kernel {

///////////////////////////////////////////////////////////////////////////////

template <
  class ProblemShape_,
  class CollectiveMainloop_,
  class CollectiveEpilogue_,
  class TileScheduler_,
  class GatherA_,
  class GatherB_
>
class GemmGather
{
public:
  //
  // Type Aliases
  //
  using ProblemShape = ProblemShape_;
  static_assert(cute::rank(ProblemShape{}) == 3 or cute::rank(ProblemShape{}) == 4,
    "ProblemShape{} should be <M,N,K> or <M,N,K,L>");

  // Mainloop derived types
  using CollectiveMainloop = CollectiveMainloop_;
  using TileShape = typename CollectiveMainloop::TileShape;
  using TiledMma  = typename CollectiveMainloop::TiledMma;
  using ArchTag   = typename CollectiveMainloop::ArchTag;
  using ElementA  = typename CollectiveMainloop::ElementA;
  using StrideA   = typename CollectiveMainloop::StrideA;
  using ElementB  = typename CollectiveMainloop::ElementB;
  using StrideB   = typename CollectiveMainloop::StrideB;
  using DispatchPolicy = typename CollectiveMainloop::DispatchPolicy;
  using ElementAccumulator = typename CollectiveMainloop::ElementAccumulator;
  using ClusterShape = typename DispatchPolicy::ClusterShape;
  using MainloopArguments = typename CollectiveMainloop::Arguments;
  using MainloopParams = typename CollectiveMainloop::Params;
  static_assert(ArchTag::kMinComputeCapability >= 90);

  // Epilogue derived types
  using CollectiveEpilogue = CollectiveEpilogue_;
  using ElementC = typename CollectiveEpilogue::ElementC;
  using StrideC  = typename CollectiveEpilogue::StrideC;
  using ElementD = typename CollectiveEpilogue::ElementD;
  using StrideD  = typename CollectiveEpilogue::StrideD;
  using EpilogueArguments = typename CollectiveEpilogue::Arguments;
  using EpilogueParams = typename CollectiveEpilogue::Params;

  static_assert(cute::is_void_v<TileScheduler_> or cute::is_same_v<TileScheduler_, PersistentScheduler>,
    "Non-persistent warp-specialized kernel does not support specializing the tile scheduler.");
  using TileSchedulerTag = TileScheduler_;
  using TileScheduler = typename detail::TileSchedulerSelector<
    TileScheduler_, ArchTag, TileShape, ClusterShape>::Scheduler;
  using TileSchedulerArguments = typename TileScheduler::Arguments;

  using GatherA = GatherA_;
  using GatherB = GatherB_;

  // Kernel level shared memory storage
  struct SharedStorage {
    union TensorStorage {
      using MainloopTensorStorage = typename CollectiveMainloop::TensorStorage;
      using EpilogueTensorStorage = typename CollectiveEpilogue::TensorStorage;

      MainloopTensorStorage mainloop;
      EpilogueTensorStorage epilogue;
    } tensors;

    struct PipelineStorage : cute::aligned_struct<16, _2> {
      using MainloopPipelineStorage = typename CollectiveMainloop::PipelineStorage;
      using EpiLoadPipelineStorage = typename CollectiveEpilogue::PipelineStorage;

      alignas(16) MainloopPipelineStorage mainloop;
      alignas(16) EpiLoadPipelineStorage epi_load;
    } pipelines;
  };

  static constexpr int SharedStorageSize = sizeof(SharedStorage);

  using GmemTiledCopyA = typename CollectiveMainloop::GmemTiledCopyA;
  using GmemTiledCopyB = typename CollectiveMainloop::GmemTiledCopyB;
  static_assert(cute::size(GmemTiledCopyA{}) == cute::size(GmemTiledCopyB{}), "Number of threads in A/B tiled copies must be the same.");

  static constexpr uint32_t NumLoadWarpGroups = cute::size(GmemTiledCopyA{}) / NumThreadsPerWarpGroup;
  static constexpr uint32_t NumMmaWarpGroups = CUTE_STATIC_V(cute::size(TiledMma{})) / NumThreadsPerWarpGroup;
  static constexpr uint32_t NumWarpGroups = NumLoadWarpGroups + NumMmaWarpGroups;
  static_assert(NumWarpGroups == 2 || NumWarpGroups == 3, "Number of warp groups must be 2 or 3 for good performance.");

  static constexpr uint32_t MaxThreadsPerBlock = NumWarpGroups * NumThreadsPerWarpGroup;
  static constexpr uint32_t MinBlocksPerMultiprocessor = 1;
```

**EN**: The class template parameterizes the fused kernel by problem shape, mainloop, epilogue, scheduler, and two gather descriptors. The long list of `using` aliases pulls architectural and layout information out of the CUTLASS collectives, then defines a shared-memory union so mainloop and epilogue tensor storage can reuse the same shared allocation. The warp-group constants at the end of this block are Hopper-specific performance plumbing: they split the thread block into producer warp groups for TMA-style loads and consumer warp groups for WGMMA-style math.

**CN**: 这个类模板把问题形状、mainloop、epilogue、调度器以及两个 gather 描述符都做成了模板参数。大量 `using` 别名从 CUTLASS collective 中提取出架构、布局和张量类型信息，并通过一个 shared-memory union 让 mainloop 与 epilogue 共用同一片共享内存。该块末尾的 warp-group 常量则是 Hopper 特有的性能骨架：线程块会被拆成生产者 warp group（负责 TMA 风格加载）和消费者 warp group（负责 WGMMA 风格计算）。

### Lines 137-223 — Public argument structures and launch-shape helpers

```cpp
  // Device side arguments
  struct Arguments {
    GemmUniversalMode mode{};
    ProblemShape problem_shape{};
    MainloopArguments mainloop{};
    EpilogueArguments epilogue{};
    KernelHardwareInfo hw_info{};
    TileSchedulerArguments scheduler{};
    GatherA gather_A{};
    GatherB gather_B{};
  };

  // Kernel entry point API
  struct Params {
    GemmUniversalMode mode{};
    ProblemShape problem_shape{};
    MainloopParams mainloop{};
    EpilogueParams epilogue{};
    GatherA gather_A{};
    GatherB gather_B{};
  };

  //
  // Methods
  //

  // Convert to underlying arguments. In this case, a simple copy for the aliased type.
  static
  Params
  to_underlying_arguments(Arguments const& args, void* workspace) {
    (void) workspace;
    auto problem_shape = args.problem_shape;
    if constexpr (detail::Has_SwapAB_v<CollectiveMainloop>) {
      // swap M/N
      get<0>(problem_shape) = get<1>(args.problem_shape);
      get<1>(problem_shape) = get<0>(args.problem_shape);
    }
    return {
      args.mode,
      problem_shape,
      CollectiveMainloop::to_underlying_arguments(args.problem_shape, args.mainloop, workspace),
      CollectiveEpilogue::to_underlying_arguments(args.problem_shape, args.epilogue, workspace),
      args.gather_A,
      args.gather_B
    };
  }

  static bool
  can_implement(Arguments const& args) {
    bool implementable = (args.mode == GemmUniversalMode::kGemm) or
        (args.mode == GemmUniversalMode::kBatched && cute::rank(ProblemShape{}) == 4);
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Arguments or Problem Shape don't meet the requirements.\n");
      return implementable;
    }
    implementable &= CollectiveMainloop::can_implement(args.problem_shape, args.mainloop);
    implementable &= CollectiveEpilogue::can_implement(args.problem_shape, args.epilogue);
    return implementable;
  }

  static
  size_t
  get_workspace_size(Arguments const& args) {
    return 0;
  }

  static
  cutlass::Status
  initialize_workspace(Arguments const& args, void* workspace = nullptr, cudaStream_t stream = nullptr,
    CudaHostAdapter* cuda_adapter = nullptr) {
    return Status::kSuccess;
  }

  // Computes the kernel launch grid shape based on runtime parameters
  static dim3
  get_grid_shape(Params const& params) {
    auto cluster_shape = Shape<_1,_1,_1>{};
    auto tile_shape = TileShape{};
    auto problem_shape_MNKL = append<4>(params.problem_shape, Int<1>{});
    return TileScheduler::get_tiled_cta_shape_mnl(
        problem_shape_MNKL, tile_shape, cluster_shape);
  }

  static dim3
  get_block_shape() {
    return dim3(MaxThreadsPerBlock, 1, 1);
  }
```

**EN**: `Arguments` is the host-facing API, while `Params` is the lower-level device-facing form after collective-specific translation. `to_underlying_arguments()` forwards most fields directly and optionally swaps M/N if the chosen mainloop requires operand swapping. `can_implement()` delegates legality checks to the mainloop and epilogue, `get_workspace_size()` reports no extra workspace, and the grid/block helpers derive the launch geometry from the tile shape and scheduler. Gather support appears here as first-class runtime data: `gather_A` and `gather_B` are carried alongside the standard GEMM arguments rather than being bolted on externally.

**CN**: `Arguments` 是主机端看到的 API，而 `Params` 则是经过 collective-specific 转换之后的设备端参数形式。`to_underlying_arguments()` 基本按原样转发字段，并在 mainloop 需要交换操作数时可选地交换 M/N。`can_implement()` 把合法性检查委托给 mainloop 与 epilogue，`get_workspace_size()` 表示无需额外 workspace，而网格/线程块辅助函数则根据 tile 形状和调度器推导 launch 维度。值得注意的是 gather 在这里是一级运行时数据：`gather_A` 与 `gather_B` 和标准 GEMM 参数一起传递，而不是外部额外挂接。

### Lines 224-343 — Kernel prologue: pipeline setup, role assignment, and gather tensor construction

```cpp

  CUTLASS_DEVICE
  void
  operator()(Params const& params, char* smem_buf) {
    using namespace cute;
    using X = Underscore;

    // Any Tensor Op MMA Atom in the WGMMA ISA is arch conditional to sm90a.
    #if ! defined(__CUDA_ARCH_FEAT_SM90_ALL)
      if constexpr(size<0>(typename TiledMma::AtomShape_MNK{}) == 64) {
        CUTE_INVALID_CONTROL_PATH("ERROR : Arch conditional MMA instruction used without targeting sm90a compute capability. Aborting.\n");
        return;
      }
    #endif

    enum class WarpGroupRole {
      Producer = 0,
      Consumer = 1,
    };

    // Kernel level shared memory storage
    SharedStorage& shared_storage = *reinterpret_cast<SharedStorage*>(smem_buf);

    int thread_idx = int(threadIdx.x);
    int warp_group_thread_idx = thread_idx % NumThreadsPerWarpGroup;
    int warp_group_idx = canonical_warp_group_idx();
    CUTLASS_ASSERT(warp_group_idx < NumWarpGroups);
    WarpGroupRole warp_group_role = warp_group_idx < NumLoadWarpGroups ? WarpGroupRole::Producer : WarpGroupRole::Consumer;

    // Mainloop Load pipeline
    using MainloopPipeline = typename CollectiveMainloop::MainloopPipeline;
    typename MainloopPipeline::Params mainloop_pipeline_params;
    if (warp_group_role == WarpGroupRole::Producer) {
      mainloop_pipeline_params.role = MainloopPipeline::ThreadCategory::Producer;
    }
    if (warp_group_role == WarpGroupRole::Consumer) {
      mainloop_pipeline_params.role = MainloopPipeline::ThreadCategory::Consumer;
    }
    mainloop_pipeline_params.producer_arv_count = NumLoadWarpGroups * NumThreadsPerWarpGroup;
    mainloop_pipeline_params.consumer_arv_count = NumMmaWarpGroups * NumThreadsPerWarpGroup;
    MainloopPipeline mainloop_pipeline(shared_storage.pipelines.mainloop, mainloop_pipeline_params);

    // Epilogue Load pipeline
    using EpiLoadPipeline = typename CollectiveEpilogue::LoadPipeline;
    typename EpiLoadPipeline::Params epi_load_pipeline_params;
    if (warp_group_role == WarpGroupRole::Producer) {
      epi_load_pipeline_params.role = EpiLoadPipeline::ThreadCategory::Producer;
    }
    if (warp_group_role == WarpGroupRole::Consumer) {
      epi_load_pipeline_params.role = EpiLoadPipeline::ThreadCategory::Consumer;
    }
    epi_load_pipeline_params.producer_arv_count = NumLoadWarpGroups * NumThreadsPerWarpGroup;
    epi_load_pipeline_params.consumer_arv_count = NumMmaWarpGroups * NumThreadsPerWarpGroup;
    EpiLoadPipeline epi_load_pipeline(shared_storage.pipelines.epi_load, epi_load_pipeline_params);

    // Epilogue Store pipeline
    using EpiStorePipeline = typename CollectiveEpilogue::StorePipeline;
    typename EpiStorePipeline::Params epi_store_pipeline_params;
    epi_store_pipeline_params.always_wait = true;
    EpiStorePipeline epi_store_pipeline(epi_store_pipeline_params);

    // Initialize starting pipeline states for the collectives
    typename CollectiveMainloop::PipelineState mainloop_pipe_consumer_state;
    typename CollectiveEpilogue::LoadPipelineState epi_load_pipe_consumer_state;

    // For the DMA Load (producer) we start with an opposite phase
    // i.e., we skip all waits since we know that the buffer is indeed empty
    PipelineState mainloop_pipe_producer_state = cutlass::make_producer_start_state<MainloopPipeline>();
    PipelineState epi_load_pipe_producer_state = cutlass::make_producer_start_state<EpiLoadPipeline>();
    PipelineState epi_store_pipe_producer_state = cutlass::make_producer_start_state<EpiStorePipeline>();

    // Preconditions
    static_assert(cute::rank(StrideA{}) == 3, "StrideA must be rank-3: [M, K, L]. If batch mode is not needed, set L stride to Int<0>.");
    static_assert(cute::rank(StrideB{}) == 3, "StrideB must be rank-3: [N, K, L]. If batch mode is not needed, set L stride to Int<0>.");
    static_assert(cute::rank(StrideC{}) == 3, "StrideC must be rank-3: [M, N, L]. If batch mode is not needed, set L stride to Int<0>.");
    static_assert(cute::rank(StrideD{}) == 3, "StrideD must be rank-3: [M, N, L]. If batch mode is not needed, set L stride to Int<0>.");

    // Separate out problem shape for convenience
    // Optionally append 1s until problem shape is rank-4 in case its is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(params.problem_shape, Int<1>{});
    auto M = get<0>(problem_shape_MNKL);
    auto N = get<1>(problem_shape_MNKL);
    auto K = get<2>(problem_shape_MNKL);
    auto L = get<3>(problem_shape_MNKL);

    // Represent the full tensors
    Tensor mA_mkl = make_gather_tensor(make_gmem_ptr(params.mainloop.ptr_A), make_shape(M,K,L), params.mainloop.dA, params.gather_A); //(m,k,l)
    Tensor mB_nkl = make_gather_tensor(make_gmem_ptr(params.mainloop.ptr_B), make_shape(N,K,L), params.mainloop.dB, params.gather_B); //(n,k,l)

    // Get the appropriate blocks for this thread block -- potential for thread block locality
    auto blk_shape = TileShape{};                                                                // (BLK_M,BLK_N,BLK_K)
    TiledMma tiled_mma;

    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, blk_shape, make_coord(_,_,_), Step<_1, X,_1>{});          // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, blk_shape, make_coord(_,_,_), Step< X,_1,_1>{});          // (BLK_N,BLK_K,n,k,l)

    // Compute m_coord, n_coord, and l_coord with their post-tiled shapes
    auto m_coord = idx2crd(int(blockIdx.x), shape<2>(gA_mkl));
    auto n_coord = idx2crd(int(blockIdx.y), shape<2>(gB_nkl));
    auto l_coord = idx2crd(int(blockIdx.z), shape<4>(gB_nkl));
    auto blk_coord = make_coord(m_coord, n_coord, _, l_coord);

    // Slice with m_coord and n_coord
    Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                       // (BLK_M,BLK_K,k)
    Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                       // (BLK_N,BLK_K,k)

    // Get pipeline iterators and increments from tensor shapes
    auto k_tile_iter  = cute::make_coord_iterator(shape<2>(gA));
    auto k_tile_count = size<2>(gA);
    auto c_tile_count = CollectiveEpilogue::get_load_pipe_increment(blk_shape);
    auto d_tile_count = CollectiveEpilogue::get_store_pipe_increment(blk_shape);

    // Wait for all threads in the thread block
    __syncthreads();

    // In a warp specialized kernel, collectives expose data movement and compute operations separately
    CollectiveMainloop collective_mainloop;
    CollectiveEpilogue collective_epilogue{params.epilogue, shared_storage.tensors.epilogue};
```

**EN**: The device `operator()` first checks that SM90a-only WGMMA instructions are not used on an incompatible architecture. It then assigns each warp group a producer or consumer role, initializes the mainloop and epilogue pipelines, and normalizes the problem shape to MNKL form. The key gather-fusion step is the call to `make_gather_tensor()` for A and B: instead of reading directly from dense pointer/stride pairs, the kernel wraps those accesses with gather descriptors so later tiled loads automatically follow the indirection pattern. After that, the code builds local tiles, computes block coordinates, and derives K-tile iterators for the upcoming pipeline work.

**CN**: 设备端 `operator()` 首先检查：如果当前架构不支持 SM90a 专属的 WGMMA 指令，就直接拒绝执行。之后代码为每个 warp group 指派生产者或消费者角色，初始化 mainloop 与 epilogue 管道，并把问题形状统一规范到 MNKL 形式。真正的 gather 融合关键在于对 A/B 调用 `make_gather_tensor()`：内核不再直接使用“指针 + 步长”的稠密访问，而是把这些访问包装成带 gather 描述符的张量，从而让后续分块加载自动遵循间接索引模式。随后代码构建本地 tile、计算块坐标，并为后续流水线工作推导 K 维 tile 迭代器。

### Lines 344-421 — Producer load path and consumer MMA / epilogue path

```cpp
    if (warp_group_role == WarpGroupRole::Producer) {
      // Compute tile residues for predication
      auto m_max_coord = M - size<0>(gA) * get<0>(blk_coord);                             // M - BLK_M * m_coord
      auto n_max_coord = N - size<0>(gB) * get<1>(blk_coord);                             // N - BLK_N * n_coord
      auto k_residue   = K - size<1>(gA) * size<2>(gA);                                   // K - BLK_K * k_coord_max
      auto residue_mnk = make_tuple(m_max_coord, n_max_coord, k_residue);

      collective_mainloop.load(
        mainloop_pipeline,
        mainloop_pipe_producer_state,
        gA,
        gB,
        k_tile_iter, k_tile_count,
        residue_mnk,
        thread_idx,
        shared_storage.tensors.mainloop
      );
      // Update starting mainloop pipeline state for the pipeline drain
      mainloop_pipe_producer_state.advance(k_tile_count);
      // Make sure mainloop consumer has been waited upon before issuing epilogue load
      collective_mainloop.load_tail(mainloop_pipeline, mainloop_pipe_producer_state);

      if (collective_epilogue.is_producer_load_needed()) {
        epi_load_pipe_producer_state =
        collective_epilogue.load(
          epi_load_pipeline,
          epi_load_pipe_producer_state,
          problem_shape_MNKL,
          blk_shape,
          blk_coord,
          tiled_mma,
          thread_idx,
          shared_storage.tensors.epilogue
        );
        collective_epilogue.load_tail(epi_load_pipeline, epi_load_pipe_producer_state);
      }
    }
    else if (warp_group_role == WarpGroupRole::Consumer) {
      Tensor accumulators = partition_fragment_C(tiled_mma, take<0,2>(blk_shape));                 // (MMA,MMA_M,MMA_N)

      collective_mainloop.mma(
        mainloop_pipeline,
        mainloop_pipe_consumer_state,
        accumulators,
        k_tile_count,
        warp_group_thread_idx,
        shared_storage.tensors.mainloop,
        params.mainloop
      );

      // Make sure the math instructions are done and free buffers before entering the epilogue
      collective_mainloop.mma_tail(
        mainloop_pipeline,
        mainloop_pipe_consumer_state,
        k_tile_count
      );

      // Epilogue and write to gD
      collective_epilogue.store(
        epi_load_pipeline,
        epi_load_pipe_consumer_state,
        epi_store_pipeline,
        epi_store_pipe_producer_state,
        problem_shape_MNKL,
        blk_shape,
        blk_coord,
        accumulators,
        tiled_mma,
        warp_group_thread_idx,
        shared_storage.tensors.epilogue
      );
    }
  }
};

///////////////////////////////////////////////////////////////////////////////

} // namespace cutlass::gemm::kernel
```

**EN**: The final block shows the warp-specialized split explicitly. Producer warp groups compute boundary residues and call `collective_mainloop.load()` so gathered A/B tiles are asynchronously staged into shared memory; if needed, they also preload epilogue data such as C. Consumer warp groups allocate accumulator fragments, run `collective_mainloop.mma()` on the shared-memory tiles, drain the math pipeline, and finally invoke `collective_epilogue.store()` to fuse the output and write D. This separation is the essence of the helper: gather logic is fused into the load side, while the compute side can remain a normal CUTLASS Hopper GEMM consumer.

**CN**: 最后一段把 warp-specialized 分工展示得非常明确。生产者 warp group 会先计算边界残差，然后调用 `collective_mainloop.load()`，把经由 gather 间接索引得到的 A/B tile 异步搬入共享内存；如果 epilogue 需要，它们还会预加载诸如 C 之类的数据。消费者 warp group 则分配累加器片段，在共享内存 tile 上调用 `collective_mainloop.mma()` 完成计算，排空数学流水线后，再通过 `collective_epilogue.store()` 融合输出并写回 D。这个辅助类的本质就在这里：gather 逻辑完全融合在加载侧，而计算侧仍可保持为标准的 CUTLASS Hopper GEMM 消费者路径。

---

## Key Concepts / 关键概念

- **Gather fusion helper**
  - **EN**: `make_gather_tensor()` is the crucial adapter that turns ordinary global-memory tensors into gather-aware tensors without rewriting the rest of the GEMM pipeline.
  - **CN**: `make_gather_tensor()` 是最关键的适配层：它把普通全局内存张量转换成支持 gather 的张量，而无需重写后续 GEMM 流水线。
- **Warp specialization**
  - **EN**: Producer warp groups focus on data movement while consumer warp groups focus on WGMMA math, allowing memory and compute to overlap.
  - **CN**: 生产者 warp group 专注于数据搬运，消费者 warp group 专注于 WGMMA 计算，从而实现访存与计算重叠。
- **Shared-memory reuse**
  - **EN**: Mainloop and epilogue tensor storage share a union because they are not live in the same way at the same time.
  - **CN**: mainloop 与 epilogue 的张量存储通过 union 复用，因为它们并不会在完全相同的生命周期内同时占满共享内存。
- **Persistent scheduling compatibility**
  - **EN**: The wrapper only permits either the default path or `PersistentScheduler`, which fits Hopper’s long-lived warp-specialized execution model.
  - **CN**: 该封装只允许默认调度或 `PersistentScheduler`，这与 Hopper 上长期驻留的 warp-specialized 执行模型相匹配。

## Dependencies / 依赖项

- CUTLASS collective mainloop / epilogue types
  - **EN**: The wrapper depends on existing CUTLASS collective implementations instead of replacing them.
  - **CN**: 该封装依赖现成的 CUTLASS collective mainloop / epilogue 实现，而不是自行替换整套逻辑。
- `gather_tensor.hpp`
  - **EN**: Supplies the gather-aware tensor wrapper used to inject indirection into A and B loads.
  - **CN**: 提供带 gather 语义的张量包装器，用于把间接索引注入到 A/B 加载路径中。
- CuTe tensor and layout utilities
  - **EN**: CuTe helpers provide shape manipulation, local tiling, coordinate conversion, and pipeline-friendly tensor views.
  - **CN**: CuTe 工具负责形状操作、本地分块、坐标转换以及适合流水线使用的张量视图。
- Hopper pipeline and WGMMA support
  - **EN**: The kernel assumes Hopper-era pipeline primitives and warp-group MMA support exposed by CUTLASS.
  - **CN**: 该内核默认依赖 CUTLASS 暴露的 Hopper 时代流水线原语和 warp-group MMA 支持。
