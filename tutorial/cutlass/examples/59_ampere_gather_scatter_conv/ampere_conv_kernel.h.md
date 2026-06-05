# ampere_conv_kernel.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/59_ampere_gather_scatter_conv/ampere_conv_kernel.h`  
**Purpose / 用途**: Defines an Ampere TF32 forward-convolution kernel and CPU reference whose activation/output tensor layouts can be affine or gather/scatter without changing the kernel body. / 定义一个面向 Ampere 的 TF32 前向卷积 kernel 与 CPU 参考实现；其激活/输出张量布局既可以是普通仿射布局，也可以是 gather/scatter 布局，而无需改动 kernel 主体。

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
- **EN**: BSD-3-Clause license text plus `#pragma once`. This section is administrative, but it also ensures the header is only included once during compilation.
- **CN**: 这一段是 BSD-3-Clause 许可证声明以及 `#pragma once`。它本身不参与算法，但可防止头文件被重复包含。

```cpp
#include "cute/tensor.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/atom/copy_atom.hpp"
#include <random>

#include "cutlass/util/print_error.hpp"

#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_mma.hpp"

using namespace cute;

```
- **EN**: Includes CuTe tensor/layout machinery, MMA/copy atoms, a small C++ utility header, and CUTLASS collective support. `using namespace cute;` makes the shape/layout DSL concise throughout the file.
- **CN**: 这里引入了 CuTe 的张量/布局基础设施、MMA 与 copy atom、一个标准库工具头，以及 CUTLASS 的 collective 支持。`using namespace cute;` 让后续的 shape/layout DSL 写法更紧凑。

```cpp
struct AmpereUnpredicatedFprop {
  //
  // Static config for conv problem shape
  //
  using D = _6;
  using H = _4;
  using W = _4;

  using T = _3;
  using R = _3;
  using S = _3;

  using Z = _4;
  using P = _2;
  using Q = _2;

  using C = _64;
  using K = _128;

  // Tiler config
  using Tiler_K = decltype(cute::min(K{}, _128{}));
  using Tiler_C = decltype(cute::min(C{}, _32{}));
  using Tiler_N = _4;
  using TileM = Tiler_K;
  using TileN = Shape<Tiler_N, Z, P, Q>;
  using TileK = Shape<Tiler_C,_1,_1,_1>;
  using PIPE  = _3;
  using TilerFlt = Shape<TileM, TileK>;
  using TilerAct = Shape<TileN, TileK>;
  using TilerOut = Shape<TileM, TileN>;

  using TileSizeM = Int<size(TileM{})>;
  using TileSizeN = Int<size(TileN{})>;
  using TileSizeK = Int<size(TileK{})>;
  static constexpr int Stages = PIPE::value;

```
- **EN**: Defines a fixed 3D forward-propagation convolution problem at compile time: input spatial shape `D/H/W`, filter shape `T/R/S`, output spatial shape `Z/P/Q`, channels `C`, and output channels `K`. It then derives threadblock tiling: `TileM` maps the output-channel/K dimension, `TileN` maps `(N,Z,P,Q)`, `TileK` maps the contracted `(C,T,R,S)` slice, and `PIPE=_3` requests a 3-stage mainloop pipeline.
- **CN**: 这里用编译期常量定义了一个固定的 3D 前向卷积问题：输入空间尺寸 `D/H/W`、卷积核尺寸 `T/R/S`、输出空间尺寸 `Z/P/Q`、输入通道 `C` 和输出通道 `K`。随后派生线程块分块方式：`TileM` 对应输出通道/K 维，`TileN` 对应 `(N,Z,P,Q)`，`TileK` 对应被约简的 `(C,T,R,S)` 分片，`PIPE=_3` 表示主循环采用 3 级流水。

```cpp
  using ElementFlt = tfloat32_t;
  using ElementAct = tfloat32_t;
  using ElementOut = float;

  using TiledMma = TiledMMA<
    MMA_Atom<SM80_16x8x8_F32TF32TF32F32_TN>,
    Layout<Shape<_2,_2,_1>>,
    Tile<_32,_32,Underscore>>;

  static constexpr int MaxThreadsPerBlock = size(TiledMma{});
  static constexpr int MinBlocksPerMultiprocessor = 1;

```
- **EN**: Chooses TF32 for filter/activation inputs and FP32 for outputs. `TiledMma` is built from the Ampere SM80 Tensor Core instruction `16x8x8` TF32 MMA and arranged into a larger tiled MMA object. The block uses exactly `size(TiledMma{})` threads, matching the collective’s expected thread layout.
- **CN**: 这里选择滤波器/激活输入为 TF32，输出为 FP32。`TiledMma` 基于 Ampere SM80 的 `16x8x8` TF32 Tensor Core 指令构造，再拼成更大的 tiled MMA 对象。线程块线程数直接取 `size(TiledMma{})`，与 collective 所需的线程映射保持一致。

```cpp
  union SharedStorage {
    struct {
      ElementFlt sAMatrix[size(TileM{}) * size(TileK{}) * size(PIPE{})];
      ElementAct sBMatrix[size(TileN{}) * size(TileK{}) * size(PIPE{})];
    } mainloop;

    struct {
      ElementOut sCMatrix[size(TileM{}) * size(TileN{})];
    } epilogue;
  };

```
- **EN**: Defines dynamic shared memory as a union. During the mainloop it stores staged A/B tiles for all pipeline stages; during the epilogue the same memory is reused for the output tile. This is a classic shared-memory footprint optimization.
- **CN**: 这里把动态共享内存定义为联合体。主循环阶段它保存 A/B 操作数在各流水级中的分块；进入 epilogue 后又复用同一块内存存放输出 tile。这是典型的共享内存占用优化手法。

```cpp
  //
  // Stencil tensor
  //

  using GmemLayoutFlt = decltype(make_ordered_layout(
    Shape< K, Shape< C, T, R, S>>{},
    tuple<_4, tuple<_0,_3,_2,_1>>{}));

```
- **EN**: Introduces the logical/global-memory filter layout. The filter is modeled as `(K, (C,T,R,S))`, i.e. output channels by flattened stencil coordinates, with a static CuTe ordering chosen for efficient tiling and slicing.
- **CN**: 这里定义滤波器的逻辑/全局内存布局。滤波器被看作 `(K, (C,T,R,S))`：外层是输出通道，内层是展平后的卷积核坐标，并使用静态 CuTe 排布以便高效分块和切片。

```cpp
  // We have 64 elements * 32b each in the major mode that we can vectorize
  // Max vector size is 128b, so lay 16 threads along the major mode with a vector size of 4
  // Rest along the minor mode
  using GmemTiledCopyFlt = decltype(make_tiled_copy(
    Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, ElementFlt>{},
    Layout<Shape <_16, _8>,
           Stride< _8, _1>>{},
    Layout<Shape < _1, _4>>{}));

```
- **EN**: Configures the global-to-shared copy for filter tiles. The comments explain the vectorization strategy: because the major mode has many contiguous 32-bit elements, the copy is organized as 128-bit `cp.async` transfers with threads spread first along the vectorized major mode and then along the minor mode.
- **CN**: 这里配置滤波器 tile 从全局内存到共享内存的搬运方式。注释解释了向量化思路：由于主方向上有很多连续的 32 位元素，所以采用 128 位 `cp.async` 传输，并让线程先沿可向量化主方向分布，再覆盖次方向。

```cpp
  // Following layout is also correct, but trades off dynamic strides in the slice for bank conflict free accesses
  // using SmemLayoutFlt = decltype(
  //     composition(Swizzle<3,2,3>{},
  //                 make_ordered_layout(
  //                     Shape<TileSizeM,TileSizeK,PIPE>{},
  //                     tuple<       _1,       _0,  _2>{})));

  using SmemLayoutAtomFlt = decltype(
    composition(Swizzle<1,2,3>{},
                Layout<Shape <_8,Shape <_4, _2>>,
                       Stride<_4,Stride<_1,_32>>>{}));

  using SmemCopyAtomFlt = Copy_Atom<SM75_U32x4_LDSM_N, ElementFlt>;

```
- **EN**: Shows a commented alternative shared-memory layout that would favor bank-conflict-free accesses at the cost of more dynamic slicing complexity. The active choice uses a lighter swizzled layout atom plus `SM75_U32x4_LDSM_N`, which matches Tensor Core-style matrix loads from shared memory.
- **CN**: 这一段先给出一个被注释掉的备选共享内存布局：它更偏向避免 bank conflict，但切片时需要处理更多动态步长。实际启用的是较轻量的 swizzle 布局 atom，并配合 `SM75_U32x4_LDSM_N`，以匹配 Tensor Core 风格的共享内存矩阵读取。

```cpp
  //
  // Activation tensor
  //

  // Activation tensor is major in the contraction mode, so vectorize that mode first
  // Then lay out the rest of the threads along the other mode
  using GmemTiledCopyAct = decltype(make_tiled_copy(
    Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<uint128_t>, ElementAct>{},
    Layout<Shape <_16, _8>,
           Stride< _8, _1>>{},
    Layout<Shape < _1, _4>>{}));

```
- **EN**: Sets up the activation-side global copy. The activation tensor is also vectorized first along the contraction-major direction, so A and B operands are staged with a symmetric 128-bit `cp.async` policy.
- **CN**: 这里配置激活张量一侧的全局搬运。激活张量同样优先沿约简主方向做向量化，因此 A/B 两个操作数都采用对称的 128 位 `cp.async` 预取策略。

```cpp
  // Following layout is also correct, but trades off dynamic strides in the slice for bank conflict free accesses
  // using SmemLayoutAct = decltype(
  //     composition(Swizzle<3,2,3>{},
  //                 make_ordered_layout(
  //                     Shape<TileSizeN,TileSizeK,PIPE>{},
  //                     tuple<       _1,       _0,  _2>{})));

  using SmemLayoutAtomAct = decltype(
    composition(Swizzle<1,2,3>{},
                Layout<Shape <_8,Shape <_4, _2>>,
                       Stride<_4,Stride<_1,_32>>>{}));

  using SmemCopyAtomAct = Copy_Atom<SM75_U32x4_LDSM_N, ElementAct>;

```
- **EN**: Mirrors the filter-side shared-memory arrangement for the activation operand. Again, an alternative layout is documented but disabled; the selected `SmemLayoutAtomAct` and `SmemCopyAtomAct` prepare operand B for efficient Tensor Core consumption.
- **CN**: 这一段为激活操作数提供与滤波器侧相呼应的共享内存布局。这里同样保留了一个未启用的备选布局；当前选择的 `SmemLayoutAtomAct` 与 `SmemCopyAtomAct` 负责把 B 操作数整理成适合 Tensor Core 消费的形式。

```cpp
  //
  // Output tensor
  //

  using GmemTiledCopyOut = decltype(make_tiled_copy(
    Copy_Atom<UniversalCopy<uint128_t>, ElementAct>{},
    Layout<Shape <_8, _16>,
           Stride<_1,  _8>>{},
    Layout<Shape <_4,  _1>>{}));

  using SmemCopyAtomOut = Copy_Atom<UniversalCopy<uint32_t>, ElementOut>;

  // This can be optimized to make accesses BCF, but we use a col-major layout here to show off composability
  using SmemLayoutOut = Layout<Shape<TileSizeM, TileSizeN>>;

```
- **EN**: Defines the epilogue/output movement. Results are written through a vectorized global copy pattern, staged in shared memory with 32-bit copies, and stored in a simple `Layout<Shape<TileSizeM, TileSizeN>>`. The comment notes this layout prioritizes composability/clarity over full bank-conflict optimization.
- **CN**: 这里定义输出阶段的数据搬运。结果先通过 32 位共享内存复制写入 `sC`，再按向量化模式写回全局内存，并采用简单的 `Layout<Shape<TileSizeM, TileSizeN>>` 作为输出共享布局。注释特别说明：这里优先展示可组合性与清晰性，而不是把 bank conflict 优化到极致。

```cpp
  //
  // Conv functor
  //
  template <class EngineFlt, class TensorActivation, class TensorOutput>
  void __device__
  operator()(cute::Tensor<EngineFlt, GmemLayoutFlt> mFlt, // ( K,        (C,T,R,S))
             TensorActivation                       mAct, // ((N,Z,P,Q), (C,T,R,S))
             TensorOutput                           mOut, // ( K,        (N,Z,P,Q))
             char* smem_buf) const {
    using namespace cute;
    using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveMma<
        cutlass::gemm::MainloopSm80CpAsyncUnpredicated<PIPE::value>,
        Shape<TileM,TileN,TileK>,
        ElementFlt,
        Underscore, // Ignore the stride, we are passing full cute::Tensor to operator()
        ElementAct,
        Underscore, // Ignore the stride, we are passing full cute::Tensor to operator()
        TiledMma,
        GmemTiledCopyFlt,
        SmemLayoutAtomFlt,
        SmemCopyAtomFlt,
        cute::identity,
        GmemTiledCopyAct,
        SmemLayoutAtomAct,
        SmemCopyAtomAct,
        cute::identity>;

```
- **EN**: This device call operator is the actual kernel body. The filter tensor type is fixed to `GmemLayoutFlt`, while `TensorActivation` and `TensorOutput` are template parameters, so the same kernel can accept plain affine tensors or externally composed gather/scatter tensors. `CollectiveMainloop` instantiates CUTLASS’s SM80 `cp.async` unpredicated MMA pipeline over the chosen tile shape and copy atoms.
- **CN**: 这个设备端 `operator()` 就是真正的 kernel 主体。滤波器张量类型固定为 `GmemLayoutFlt`，而 `TensorActivation` 与 `TensorOutput` 则保持模板化，因此同一份 kernel 既能接收普通仿射布局张量，也能接收外部组合出来的 gather/scatter 张量。`CollectiveMainloop` 在这里实例化了 CUTLASS 的 SM80 `cp.async` 无 predication MMA 主循环。

```cpp
    TiledMma tiled_mma;
    Tensor accum = partition_fragment_C(tiled_mma, TilerOut{});
    clear(accum);

```
- **EN**: Creates the tiled MMA object, partitions the accumulator fragment for the output tile, and explicitly zeros it before accumulation begins.
- **CN**: 这里创建 tiled MMA 对象，按输出 tile 形状切分累加寄存器片段，并在进入主循环前将其清零。

```cpp
    // Set up tensors
    // NOTE: blockIdx.x projects onto act-NDHW mode, y along the flt-K mode for the sake of higher dynamic range in NDHW
    Tensor gA_mk = local_tile(mFlt, TilerFlt{}, make_coord(_,_));                              // (BLK_M,BLK_K,m',k')
    Tensor gB_nk = local_tile(mAct, TilerAct{}, make_coord(_,_));                              // (BLK_N,BLK_K,n',_1)
    Tensor gC_mn = local_tile(mOut, TilerOut{}, make_coord(_,_));                              // (BLK_M,BLK_N,m',n')

    // Compute m_coord and n_coord with their post-tiled shapes
    auto m_coord = idx2crd(int(blockIdx.y), shape<2>(gA_mk));
    auto n_coord = idx2crd(int(blockIdx.x), shape<2>(gB_nk));
    Tensor gA = gA_mk(_,_,m_coord,_);                                                          // (BLK_M,BLK_K,k')
    Tensor gB = gB_nk(_,_,n_coord,_);                                                          // (BLK_N,BLK_K,_1)
    Tensor gC = gC_mn(_,_,m_coord,n_coord);                                                    // (BLK_M,BLK_N)

```
- **EN**: Builds per-block tensor views. `local_tile` forms tiled views of filter, activation, and output tensors; then `blockIdx.y` selects the filter/output-channel tile and `blockIdx.x` selects the `(N,Z,P,Q)` tile. `idx2crd` converts linear grid indices into coordinates of the post-tiled tensor spaces before slicing out `gA`, `gB`, and `gC`.
- **CN**: 这里构造每个线程块对应的张量视图。`local_tile` 先把滤波器、激活和输出张量变成按 tile 组织的视图；随后 `blockIdx.y` 选择滤波器/输出通道 tile，`blockIdx.x` 选择 `(N,Z,P,Q)` tile。`idx2crd` 负责把线性网格索引转为分块后张量空间中的坐标，再切出 `gA`、`gB` 和 `gC`。

```cpp
    auto k_tile_iter = cute::make_coord_iterator(size<2>(gA));
    int k_tile_count = size<2>(gA);

    CollectiveMainloop collective_mma;
    collective_mma(
      accum,
      gA,
      gB,
      accum,
      k_tile_iter, k_tile_count,
      Underscore{}, // no residue since we do not support predication
      threadIdx.x,
      smem_buf);

```
- **EN**: Creates an iterator over K-tiles and launches the collective mainloop. The collective loads staged A/B tiles, advances through the contraction dimension, and performs Tensor Core MMA into `accum`. Passing `Underscore{}` as residue handling means no predication or edge-tile cleanup is provided here, which is why the struct is explicitly named `Unpredicated`.
- **CN**: 这里先为 K 方向的 tile 构造迭代器，然后调用 collective 主循环。该 collective 会分阶段加载 A/B tile，沿约简维推进，并把 Tensor Core MMA 的结果累加到 `accum`。这里把 `Underscore{}` 作为 residue 参数传入，表示不处理边界 predication/残块，因此结构体才明确命名为 `Unpredicated`。

```cpp
    //
    // Epilogue
    //
    SharedStorage& storage = *reinterpret_cast<SharedStorage*>(smem_buf);
    Tensor sC = make_tensor(make_smem_ptr(&storage.epilogue.sCMatrix[0]), SmemLayoutOut{});

    auto smem_tiled_copy_C = make_tiled_copy_C(SmemCopyAtomOut{}, tiled_mma);
    auto smem_thr_copy_C = smem_tiled_copy_C.get_slice(threadIdx.x);
    auto tCrC = smem_thr_copy_C.retile_S(accum);
    auto tCsC = smem_thr_copy_C.partition_D(sC);
    copy(smem_tiled_copy_C, tCrC, tCsC);

    __syncthreads();

    GmemTiledCopyOut gmem_tiled_copy_C;
    auto gmem_thr_copy_C = gmem_tiled_copy_C.get_slice(threadIdx.x);
    auto tDsC = gmem_thr_copy_C.partition_S(sC);
    auto tDgC = gmem_thr_copy_C.partition_D(gC);
    copy(gmem_tiled_copy_C, tDsC, tDgC);

```
- **EN**: Implements the epilogue. Shared memory is reinterpreted as output storage, accumulator fragments are retiled into the shared-memory layout, a block-wide sync guarantees visibility, and then a second tiled copy moves the completed output tile to global memory. `make_tiled_copy_C` derives a thread mapping consistent with the MMA C-fragment layout.
- **CN**: 这里实现 epilogue。代码把共享内存重新解释为输出暂存区，将累加寄存器片段重新映射到共享内存布局中，经过一次块级同步后，再通过第二次 tiled copy 把完整输出 tile 写回全局内存。`make_tiled_copy_C` 会自动生成与 MMA 的 C 片段布局一致的线程映射。

```cpp
    #if 0
      if (thread0()) {
        print("mAct = "); print(mAct);          print('\n');
        print("mFlt = "); print(mFlt);          print('\n');
        print("mOut = "); print(mOut);          print('\n');
        print("gA   = "); print(gA);            print('\n');
        print("gB   = "); print(gB);            print('\n');
        print("gC   = "); print(gC);            print('\n');
        print("sA   = "); print(sA.layout());   print('\n');
        print("sB   = "); print(sB.layout());   print('\n');
        print("sC   = "); print(sC.layout());   print('\n');
        print("tAgA = "); print(tAgA.layout()); print('\n');
        print("tBgB = "); print(tBgB.layout()); print('\n');
        print("tAsA = "); print(tAsA.layout()); print('\n');
        print("tBsB = "); print(tBsB.layout()); print('\n');
        print("tCsA = "); print(tCsA.layout()); print('\n');
        print("tCsB = "); print(tCsB.layout()); print('\n');
        print("tCrC = "); print(tCrC.layout()); print('\n');
        print("tCsC = "); print(tCsC.layout()); print('\n');
        print("tDsC = "); print(tDsC.layout()); print('\n');
        print("tDgC = "); print(tDgC.layout()); print('\n');
        print("gmem tiled copy A = "); print(gmem_tiled_copy_A); print('\n');
        print("gmem tiled copy B = "); print(gmem_tiled_copy_B); print('\n');
        print("gmem tiled copy C = "); print(gmem_tiled_copy_C); print('\n');
        print("k_tile_count = "); print(size<2>(gA)); print('\n');
        print("k_tile_iter  = "); print(*k_tile_iter); print('\n');
        print("K_BLOCK_MAX  = "); print(K_BLOCK_MAX); print('\n');
    }
    #endif
  }
};

```
- **EN**: This disabled debug section prints tensor values, layouts, partition layouts, copy objects, and iteration metadata. It is useful for studying CuTe composition and verifying how the collective partitioned work, but `#if 0` keeps it out of normal builds. The final braces close the operator and the kernel-config struct.
- **CN**: 这一段被禁用的调试代码会打印张量值、布局、分区布局、copy 对象以及迭代元数据。它非常适合研究 CuTe 的组合方式以及 collective 的分工结果，但由于被 `#if 0` 包裹，正常构建时不会编译进去。最后的花括号用于结束 `operator()` 和整个 kernel 配置结构体。

```cpp
template <class TensorFlt, class TensorAct, class TensorOut>
inline int
fprop_reference(
    TensorFlt mStencil,    // Logical MK: ( K,        (C,T,R,S))
    TensorAct mActivation, // Logical NK: ((N,Z,P,Q), (C,T,R,S))
    TensorOut mOutput,     // Logical MN: ( K,        (N,Z,P,Q))
    TensorOut mOutputRef) {
  int32_t N = size<1,0>(mOutputRef); 
  int32_t Z = size<1,1>(mOutputRef);
  int32_t P = size<1,2>(mOutputRef);
  int32_t Q = size<1,3>(mOutputRef);
  int32_t T = size<1,3>(mStencil);
  int32_t R = size<1,2>(mStencil);
  int32_t S = size<1,1>(mStencil);
  int32_t C = size<1,0>(mStencil);

  size_t K    = static_cast<size_t>(size<0>(mOutputRef));
  size_t NZPQ = static_cast<size_t>(size<1>(mOutputRef));
  size_t CTRS = static_cast<size_t>(size<1>(mStencil));

```
- **EN**: Defines a host-side reference implementation for correctness checking. The tensors are treated as logically flattened `(M,K)` / `(N,K)` / `(M,N)` objects, while the named shape queries recover the original convolution semantics (`N/Z/P/Q/T/R/S/C`) for clarity. The important reduced extents are `K`, `NZPQ`, and `CTRS`.
- **CN**: 这里定义了一个主机端参考实现，用于正确性校验。张量在计算时被视为逻辑上展平的 `(M,K)` / `(N,K)` / `(M,N)` 对象，而前面的命名形状提取则把它们重新对应回卷积语义上的 `N/Z/P/Q/T/R/S/C` 维度。真正驱动参考收缩计算的关键展平长度是 `K`、`NZPQ` 和 `CTRS`。

```cpp
#if defined(_OPENMP)
  #pragma omp parallel for
#endif
  for (size_t logical_m = 0; logical_m < K; ++logical_m) {
    for (size_t logical_n = 0; logical_n < NZPQ; ++logical_n) {
      auto accumulator = float(0);
      for (size_t logical_k = 0; logical_k < CTRS; ++logical_k) {
        accumulator += mStencil(logical_m, logical_k) * mActivation(logical_n, logical_k);
      }
      mOutputRef(logical_m, logical_n) = accumulator;
    }
  }

  return print_relative_error(mOutput, mOutputRef,  /*print_verbose*/ false,  /*print_error*/ true, /*error_margin*/ 0.01);
}
```
- **EN**: Runs the reference contraction, optionally parallelized with OpenMP across output-channel rows. The nested loops compute `output_ref(m,n) = sum_k stencil(m,k) * activation(n,k)`, then `print_relative_error` compares device and reference outputs with a 1% error margin.
- **CN**: 这里执行参考收缩计算，并可在定义了 OpenMP 时按输出通道行做并行。三层循环完成 `output_ref(m,n) = sum_k stencil(m,k) * activation(n,k)`，最后通过 `print_relative_error` 以 1% 误差阈值比较设备结果与参考结果。

---

## Key Concepts / 关键概念

- **Convolution as contraction / 卷积即张量收缩**: The header expresses fprop as a GEMM-like contraction between filter `(K, CTRS)` and activation `((N,Z,P,Q), CTRS)`, producing output `(K, (N,Z,P,Q))`. / 该头文件把前向卷积表达为类似 GEMM 的张量收缩：滤波器 `(K, CTRS)` 与激活 `((N,Z,P,Q), CTRS)` 相乘，得到输出 `(K, (N,Z,P,Q))`。
- **Layout polymorphism enables gather/scatter / 布局多态使 gather/scatter 成立**: The kernel body is generic over activation/output tensor types, so gather/scatter behavior is supplied by the caller through composed CuTe layouts rather than special kernel branches. / kernel 主体对激活/输出张量类型保持泛化，因此 gather/scatter 行为由调用方通过组合 CuTe 布局注入，而不是在 kernel 内写特殊分支。
- **Ampere-oriented pipeline / 面向 Ampere 的流水设计**: `SM80_16x8x8` TF32 MMA, 128-bit `cp.async` global-to-shared copies, a 3-stage pipeline, and swizzled shared-memory layouts are chosen to feed Ampere Tensor Cores efficiently. / `SM80_16x8x8` TF32 MMA、128 位 `cp.async` 全局到共享内存搬运、3 级流水以及带 swizzle 的共享内存布局，共同服务于 Ampere Tensor Core 的高效供数。
- **Unpredicated mainloop / 无 predication 主循环**: The mainloop assumes exact tile divisibility and passes no residue handling object, so edge cases must already be excluded by problem selection or launch-time checks. / 主循环假定各维度能被 tile 整除，并且没有传入残块处理对象，因此边界情形需要在问题规模选择或 launch 前检查中提前排除。
- **Iterator/layout algebra / 迭代器与布局代数**: `local_tile`, `idx2crd`, coordinate iterators, partitioned fragments, and tiled copy objects show the CuTe style of describing movement and computation through type-level layout algebra. / `local_tile`、`idx2crd`、坐标迭代器、分块片段以及 tiled copy 对象体现了 CuTe 通过类型级布局代数来描述数据移动与计算的风格。

## Dependencies / 依赖项

- `cute/tensor.hpp`: core tensor, shape, coordinate, tiling, and layout utilities. / 核心张量、shape、坐标、分块与布局工具。
- `cute/atom/mma_atom.hpp` and `cute/atom/copy_atom.hpp`: Tensor Core MMA atoms and copy atoms used to build the tiled mainloop. / 用于构建 tiled 主循环的 Tensor Core MMA atom 与 copy atom。
- `cutlass/gemm/dispatch_policy.hpp` and `cutlass/gemm/collective/collective_mma.hpp`: CUTLASS collective mainloop infrastructure for the SM80 cp.async MMA pipeline. / CUTLASS 在 SM80 上实现 cp.async MMA 主循环所需的 collective 基础设施。
- `cutlass/util/print_error.hpp`: provides `print_relative_error` for host-side validation. / 提供主机端验证函数 `print_relative_error`。
- External tensor layouts from `ampere_gather_scatter_conv.cu`: dense affine layouts or composed `IndexedGather`/scatter layouts are plugged into this header through the `TensorActivation` and `TensorOutput` template parameters. / 来自 `ampere_gather_scatter_conv.cu` 的外部张量布局：无论是普通仿射布局还是组合出来的 `IndexedGather`/scatter 布局，都会通过 `TensorActivation` 和 `TensorOutput` 模板参数接入本头文件。
