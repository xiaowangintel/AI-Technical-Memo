# gemm_with_layernorm.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/37_gemm_layernorm_gemm_fusion/gemm_with_layernorm.h`
**Purpose / 用途**: Implements a three-part fused pipeline: GEMM0 produces intermediate activations and partial LayerNorm statistics, a small reduction kernel finalizes mean and inverse standard deviation, and GEMM1 consumes the normalized result with fused scale/bias work / 实现三段式融合流程：GEMM0 生成中间激活并累积 LayerNorm 部分统计量，轻量归约核完成均值与逆标准差，GEMM1 再消费归一化结果并融合 scale/bias 计算。
---
## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
    \brief A file contains all functioning classes needed by GemmLayernorm.

    GemmLayernorm example =  GEMM0 with partial reduction fused in epilogue (EpilogueVisitorLayerNorm)
                          +  lightweight full reduction kernel (ApplyFinalReduction)
                          +  GEMM1 with elemenwise operations fused in mainloop (GemmLayernormMainloopFusion)

*/
```
**EN**: Top comments do two jobs: the BSD license and a one-paragraph architecture summary. The summary already explains why the implementation is split into GEMM0, a final reduction kernel, and GEMM1.
**CN**: 顶部注释有两层作用：一是 BSD 许可声明，二是整体架构摘要。摘要已经点明实现为何拆成 GEMM0、最终归约核和 GEMM1 三部分。

```cpp
#pragma once

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <cmath>
#include <iostream>
#include <vector>
#include <limits>

#include "cutlass/cutlass.h"
#include "cutlass/arch/memory.h"
#include "cutlass/arch/memory_sm75.h"
#include "cutlass/gemm/device/gemm_layernorm_mainloop_fusion.h"
#include "cutlass/gemm/kernel/gemm_transpose_operands.h"
#include "cutlass/gemm/kernel/default_gemm.h"
#include "cutlass/gemm/kernel/default_gemm_complex.h"
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/epilogue/threadblock/epilogue_with_visitor.h"

/////////////////////////////////////////////////////////////////////////////////////////////////

#include "gemm_with_epilogue_visitor.h"
#include "helper.h"
```
**EN**: The file pulls in basic C++ utilities, CUTLASS core headers, GEMM building blocks, and the example-local helpers. The important dependencies are the epilogue-visitor path for GEMM0 and the mainloop-fusion path for GEMM1.
**CN**: 这里引入了基础 C++ 头文件、CUTLASS 核心头文件、GEMM 构件以及示例本地辅助文件。最关键的依赖是给 GEMM0 使用的 epilogue visitor 路径，以及给 GEMM1 使用的 mainloop fusion 路径。

```cpp
namespace cutlass {

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace kernel {

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Everything lives inside cutlass::kernel first, then the public wrapper stays in cutlass. This separation keeps low-level kernels distinct from the higher-level orchestration class.
**CN**: 代码先放在 cutlass::kernel 中实现底层 kernel，再在 cutlass 命名空间中提供对外包装类。这样低层算子与高层编排逻辑被清晰分开。

```cpp
template <
  typename ElementVariance_,
  typename ElementMean_,
  typename ElementLayernormCompute_,
  typename ElementOutput,
  typename ThreadblockShape_,
  bool IsShiftedVariance_ = false
>
class ApplyFinalReduction {
public:

  using ElementVariance = ElementVariance_;
  using ElementMean = ElementMean_;
  using ElementLayernormCompute = ElementLayernormCompute_;
  using ThreadblockShape = ThreadblockShape_;

  // Pre-processing has ensured the layout equivalent to RowMajor
  using Layout = cutlass::layout::RowMajor;

  using TensorVariance = TensorRef<ElementVariance, Layout>;
  using TensorMean = TensorRef<ElementMean, Layout>;

  static bool const kIsShiftedVariance = IsShiftedVariance_;
```
**EN**: ApplyFinalReduction is a tiny CUDA kernel template that finalizes LayerNorm statistics. Its template parameters let the caller choose storage types, compute type, tile shape, and whether shifted-variance math is enabled.
**CN**: ApplyFinalReduction 是一个很小的 CUDA kernel 模板，用来完成 LayerNorm 统计量的最终收敛。模板参数允许调用者选择存储类型、计算类型、tile 形状，以及是否启用 shifted-variance 计算路径。

```cpp
  struct Arguments {

    MatrixCoord     extent;             ///< Extent of D and Layernorm matrices
    TensorVariance  ref_Variance;       ///< Sum Square or Variance tensor (input / output)
    TensorMean      ref_Mean;           ///< Sum or Mean tensor (input / output)
    ElementOutput   *ptr_Shifted_K;     ///< Shifted K tensor pointer

    //
    // Methods
    //
    Arguments(){ }

    Arguments(
      MatrixCoord     extent_,
      TensorVariance  ref_Variance_,
      TensorMean      ref_Mean_,
      ElementOutput   *ptr_Shifted_K_
    ):
      extent(extent_),
      ref_Variance(ref_Variance_),
      ref_Mean(ref_Mean_),
      ptr_Shifted_K(ptr_Shifted_K_)
    {

    }
  };
```
**EN**: The kernel arguments are minimal: the logical extent, tensor refs for partial variance and mean buffers, and an optional Shifted-K vector. The same buffers are used as both input and output, so the kernel reads partial sums then overwrites the first slot with finalized values.
**CN**: 这个 kernel 的参数非常精简：逻辑尺寸、保存部分方差与均值的张量引用，以及可选的 Shifted-K 向量。同一组缓冲区同时充当输入和输出，因此 kernel 会先读取部分和，再把首个槽位改写成最终结果。

```cpp
  struct SharedStorage {


  };

  //
  // Params struct
  //

  struct Params {
    Arguments args;

    //
    // Methods
    //
    Params() { }

    Params(Arguments const &args_): args(args_) { }
  };

private:

public:

  CUTLASS_DEVICE
  ApplyFinalReduction() { }

  CUTLASS_DEVICE
  void operator()(Params const &params, SharedStorage &shared_storage) {

    apply(params, shared_storage);
  }
```
**EN**: SharedStorage is empty because this kernel only needs registers and global memory. Params is a thin CUTLASS-style wrapper around Arguments, and operator() just forwards to apply().
**CN**: SharedStorage 为空，因为这个 kernel 只依赖寄存器和全局内存。Params 是一个很薄的 CUTLASS 风格包装，operator() 只是把调用转发给 apply()。

```cpp
  /// Partial reduction
  CUTLASS_DEVICE
  void apply(Params const &params, SharedStorage &shared_storage) {

    int threadblock_num = (params.args.extent.column() + ThreadblockShape::kM - 1) / ThreadblockShape::kM;

    int block_n = blockIdx.x * blockDim.x;

    int thread_n = threadIdx.x;

    int idx_n = block_n + thread_n;

    if (idx_n >= params.args.extent.row()) {
      return;
    }
```
**EN**: Each CUDA thread is mapped to one logical row of the LayerNorm result. threadblock_num counts how many partial tiles were produced along the reduction axis, so one thread can walk across all partial statistics for its row.
**CN**: 每个 CUDA 线程对应 LayerNorm 结果中的一个逻辑行。threadblock_num 表示归约轴上产生了多少个部分 tile，因此一个线程可以遍历该行对应的全部部分统计量。

```cpp
    using ConvertVarianceOutput = cutlass::NumericConverter<ElementVariance, ElementLayernormCompute>;
    using ConvertMeanOutput = cutlass::NumericConverter<ElementMean, ElementLayernormCompute>;

    using ConvertVariance = cutlass::NumericConverter<ElementLayernormCompute, ElementVariance>;
    using ConvertMean = cutlass::NumericConverter<ElementLayernormCompute, ElementMean>;

    using ConvertShiftK = cutlass::NumericConverter<ElementLayernormCompute, ElementOutput>;

    ConvertVariance   convert_variance;
    ConvertMean  convert_mean;

    ConvertVarianceOutput   convert_variance_output;
    ConvertMeanOutput  convert_mean_output;

    ElementVariance *access_square = params.args.ref_Variance.data() + idx_n;
    ElementMean *access_mean = params.args.ref_Mean.data() + idx_n;

    ElementVariance *access_square_bak = access_square;
    ElementMean *access_mean_bak = access_mean;

    ElementLayernormCompute frag_square_sum = ElementLayernormCompute(0);
    ElementLayernormCompute frag_element_sum = ElementLayernormCompute(0);
    ElementVariance fetch_square;
    ElementMean fetch_mean;

    CUTLASS_PRAGMA_UNROLL
    for (int idx_m = 0; idx_m < threadblock_num; idx_m++) {
      arch::global_load<ElementVariance, sizeof(ElementVariance)>(fetch_square, access_square, true);
      arch::global_load<ElementMean, sizeof(ElementMean)>(fetch_mean, access_mean, true);
      frag_element_sum += convert_mean(fetch_mean);
      frag_square_sum += convert_variance(fetch_square);
      access_square += params.args.extent.row();
      access_mean += params.args.extent.row();
    }
```
**EN**: The kernel converts stored partial sums back to the compute type, then accumulates mean and square-mean contributions across all threadblock slices. The stride by extent.row() reflects how GEMM0 laid out per-row partial statistics in memory.
**CN**: kernel 会先把保存下来的部分和转换回计算类型，然后跨越所有 threadblock 切片累积均值项与平方均值项。按 extent.row() 跨步访问，反映了 GEMM0 在内存中按行组织部分统计量的方式。

```cpp
    ElementLayernormCompute mean = frag_element_sum;
    ElementLayernormCompute square_mean = frag_square_sum;

    ElementLayernormCompute variance;

    if (kIsShiftedVariance && params.args.ptr_Shifted_K != nullptr) {
      ElementOutput *access_shift_k = params.args.ptr_Shifted_K + idx_n;
      ElementOutput fetch_shift_k;
      ConvertShiftK convert_shift_k;
      arch::global_load<ElementOutput, sizeof(ElementOutput)>(fetch_shift_k, access_shift_k, true);
      ElementLayernormCompute shifted_mean =  mean - convert_shift_k(fetch_shift_k);
      variance = cutlass::constants::one<ElementLayernormCompute>() / cutlass::fast_sqrt(square_mean - shifted_mean * shifted_mean + ElementLayernormCompute(1e-6));
    }else{
      variance = cutlass::constants::one<ElementLayernormCompute>() / cutlass::fast_sqrt(square_mean - mean * mean + ElementLayernormCompute(1e-6));
    }

    mean = -mean * variance;

    access_square = access_square_bak;
    access_mean = access_mean_bak;

    access_square[0] = convert_variance_output(variance);
    access_mean[0] = convert_mean_output(mean);

  }
```
**EN**: After accumulation, the code computes inverse standard deviation, not raw variance, using rsqrt with epsilon. It also rewrites mean as -mean * inv_std, which is the affine term GEMM1 needs for fast normalization.
**CN**: 累积完成后，代码计算的不是原始方差，而是带 epsilon 的逆标准差。与此同时，mean 被改写为 -mean * inv_std，这正是 GEMM1 快速做归一化时需要的仿射项。

```cpp
template <
  typename ThreadblockShape_,
  int ThreadCount,
  typename OutputTileIterator_,
  typename AccumulatorTile_,
  typename ElementAccumulator_,
  typename ElementVariance_,
  typename ElementMean_,
  typename ElementLayernormCompute_,
  typename ElementwiseFunctor_,
  bool IsShiftedVariance_ = false
>
class EpilogueVisitorLayerNorm {
public:

  using ElementVariance = ElementVariance_;
  using ElementMean = ElementMean_;
  using ElementLayernormCompute = ElementLayernormCompute_;

  using AccumulatorTile = AccumulatorTile_;

  using ThreadblockShape   = ThreadblockShape_;
  static int const kThreadCount = ThreadCount;

  using OutputTileIterator = OutputTileIterator_;
  using ElementwiseFunctor = ElementwiseFunctor_;

  static int const kIterations = OutputTileIterator::kIterations;
  static int const kElementsPerAccess = OutputTileIterator::kElementsPerAccess;
  static int const kRowIterations = OutputTileIterator::ThreadMap::Iterations::kRow;

  static int const kThreads = OutputTileIterator::ThreadMap::kThreads;

  static bool const kIsShiftedVariance = IsShiftedVariance_;

  using ElementOutput = typename OutputTileIterator::Element;

  static int const kDeltaRow = OutputTileIterator::ThreadMap::Delta::kRow;

  /// Array type used in Shift-K Layernorm
  static int const kRowAccessCount = kIterations * kRowIterations;

  using ConvertedShiftFragment = Array<ElementLayernormCompute, kRowAccessCount>;

  // Conducts manual transpose externally (already supported) for column major
  using LayoutOutput = cutlass::layout::RowMajor;

  using ElementAccumulator = ElementAccumulator_;

  using AccumulatorFragment = Array<ElementAccumulator, kElementsPerAccess>;
  using LayernormFragment = Array<ElementLayernormCompute, kElementsPerAccess>;
  using OutputVector = Array<ElementOutput, kElementsPerAccess>;
  using TensorRefD = TensorRef<ElementOutput, LayoutOutput>;

  static int const kThreadsPerRow = OutputTileIterator::ThreadMap::Detail::RowArrangement::Detail::kShapeWidth;
  static int const kThreadsInColumn = kThreads / kThreadsPerRow;
  static int const kHalfThreadsPerRow = (kThreadsPerRow >> 1);
```
**EN**: EpilogueVisitorLayerNorm is the core customization point for GEMM0. It plugs into CUTLASS's existing epilogue machinery, learns tile geometry from OutputTileIterator, and adds just enough state to collect LayerNorm statistics while writing the intermediate output.
**CN**: EpilogueVisitorLayerNorm 是 GEMM0 的核心定制点。它接入 CUTLASS 现有的 epilogue 机制，从 OutputTileIterator 获取 tile 几何信息，并在写出中间结果的同时补充收集 LayerNorm 统计量所需的状态。

```cpp
  struct Arguments {

    typename ElementwiseFunctor::Params   elementwise;
    TensorRefD                            ref_C;
    TensorRefD                            ref_D;
    ElementVariance                       *ptr_Variance;
    ElementMean                           *ptr_Mean;
    ElementOutput                         *ptr_Shifted_K;

    //
    // Methods
    //
    Arguments():
      ptr_Variance(nullptr),
      ptr_Mean(nullptr),
      ptr_Shifted_K(nullptr)
    {

    }

    Arguments(
      typename ElementwiseFunctor::Params   elementwise_,
      TensorRefD                            ref_C_,
      TensorRefD                            ref_D_,
      ElementVariance                       *ptr_Variance,
      ElementMean                           *ptr_Mean_,
      ElementOutput                         *ptr_Shifted_K_ = nullptr
    ):
      elementwise(elementwise_),
      ref_C(ref_C_),
      ref_D(ref_D_),
      ptr_Variance(ptr_Variance),
      ptr_Mean(ptr_Mean_),
      ptr_Shifted_K(ptr_Shifted_K_)
    {

    }
  };
```
**EN**: The visitor Arguments bundle the normal epilogue elementwise parameters, source/output tensor refs, and pointers to the statistics buffers. Shifted-K is optional, so the same visitor supports standard and shifted-variance LayerNorm.
**CN**: visitor 的 Arguments 同时打包了普通 epilogue 的 elementwise 参数、输入/输出张量引用，以及统计缓冲区指针。Shifted-K 是可选项，因此同一个 visitor 可以同时支持普通 LayerNorm 和 shifted-variance LayerNorm。

```cpp
  struct Params {

    typename ElementwiseFunctor::Params   elementwise;
    typename OutputTileIterator::Params   params_C;
    typename OutputTileIterator::Params   params_D;
    typename OutputTileIterator::Element *ptr_C;
    typename OutputTileIterator::Element *ptr_D;
    ElementVariance                       *ptr_Variance;
    ElementMean                           *ptr_Mean;
    ElementOutput                         *ptr_Shifted_K;

    //
    // Methods
    //
    CUTLASS_HOST_DEVICE
    Params():
      ptr_D(nullptr),
      ptr_Variance(nullptr),
      ptr_Mean(nullptr)
    {

    }

    CUTLASS_HOST_DEVICE
    Params(Arguments const &args):
      elementwise(args.elementwise),
      params_C(args.ref_C.layout()),
      params_D(args.ref_D.layout()),
      ptr_C(args.ref_C.data()),
      ptr_D(args.ref_D.data()),
      ptr_Variance(args.ptr_Variance),
      ptr_Mean(args.ptr_Mean),
      ptr_Shifted_K(args.ptr_Shifted_K)
    {

    }
  };

  /// Shared storage
  struct SharedStorage {

  };

private:

  Params const &                        params_;
  SharedStorage &                       shared_storage_;
  MatrixCoord                           extent_;
  ElementwiseFunctor                    elementwise_;

  OutputTileIterator                    iterator_C_;
  OutputTileIterator                    iterator_D_;
  typename OutputTileIterator::Fragment fragment_C_;
  typename OutputTileIterator::Fragment fragment_D_;

  ElementAccumulator                    alpha_;
  ElementAccumulator                    beta_;
  ConvertedShiftFragment                shift_k_frag_;

  ElementLayernormCompute               accum_sum_square_;
  ElementLayernormCompute               accum_sum_element_;

  MatrixCoord                           thread_offset_;
```
**EN**: Params precomputes iterator layouts and raw pointers for device code. The visitor object then owns iterators for C and D, alpha/beta scaling values, per-thread statistic accumulators, an optional cached Shifted-K fragment, and thread-position metadata.
**CN**: Params 为设备端代码预先整理好迭代器布局和原始指针。随后 visitor 对象持有 C/D 迭代器、alpha/beta 缩放值、每线程统计累加器、可选的 Shifted-K 缓存片段，以及线程位置元数据。

```cpp
  CUTLASS_DEVICE
  EpilogueVisitorLayerNorm(
    Params const &params,                                         ///< Parameters routed to the epilogue
    SharedStorage &shared_storage,                                ///< Shared storage needed by the functors here
    MatrixCoord const &problem_size0,                              ///< Problem size of the output
    int thread_idx,                                               ///< Thread index within the threadblock
    int warp_idx,                                                 ///< Warp index within the threadblock
    int lane_idx,                                                 ///< Lane index within the warp
    MatrixCoord const &threadblock_offset = MatrixCoord(0, 0)
  ):
    params_(params),
    shared_storage_(shared_storage),
    extent_(problem_size0),
    elementwise_(params.elementwise),
    iterator_C_(params.params_C, params.ptr_C, problem_size0, thread_idx, threadblock_offset),
    iterator_D_(params.params_D, params.ptr_D, problem_size0, thread_idx, threadblock_offset)
  {
    alpha_ = (params.elementwise.alpha_ptr ? *params.elementwise.alpha_ptr : params.elementwise.alpha);
    beta_ =  (params.elementwise.beta_ptr ? *params.elementwise.beta_ptr : params.elementwise.beta);

    if (beta_ == ElementAccumulator()) {
      iterator_C_.clear_mask();
    }
  }
```
**EN**: The constructor materializes alpha and beta from either pointer-backed scalars or inline values, exactly like a standard CUTLASS epilogue. If beta is zero, it masks out loads from C to skip unnecessary memory traffic.
**CN**: 构造函数会像标准 CUTLASS epilogue 那样，从指针或内联值中解析 alpha 与 beta。如果 beta 为 0，就屏蔽对 C 的加载，从而避免不必要的访存。

```cpp
  /// Helper to indicate split-K behavior
  CUTLASS_DEVICE
  void set_k_partition(
    int split_k_index,                                            ///< Index of this threadblock within split-K partitioned scheme
    int split_k_slices) {                                         ///< Total number of split-K slices

  }

  /// Called to set the batch index
  CUTLASS_DEVICE
  void set_batch_index(int batch_idx) {

  }

  /// Called at the start of the epilogue just before iterating over accumulator slices
  CUTLASS_DEVICE
  void begin_epilogue() {

    // If shift-K feature is enabled, we load shift-k fragment
    // at the very beginning of an epilogue
    if (kIsShiftedVariance && params_.ptr_Shifted_K != nullptr) {
      shift_k_frag_.clear();
      int thread_offset_row_base = iterator_D_.thread_start_row();

      CUTLASS_PRAGMA_UNROLL
      for (int iter_idx = 0; iter_idx < kIterations; ++iter_idx) {
        int step_offset = iter_idx * OutputTileIterator::Shape::kRow;
        CUTLASS_PRAGMA_UNROLL
        for (int rid = 0; rid < kRowIterations; ++rid) {
          int row_step_offset = rid * kDeltaRow;
          int row_offset = thread_offset_row_base + step_offset + row_step_offset;
          bool is_load = (row_offset < extent_.row());
          shift_k_frag_[iter_idx * kRowIterations + rid] = load_shift_k_(row_offset, is_load);
        }

      }

    }

  }
```
**EN**: The split-K and batch hooks exist because the visitor conforms to the generic epilogue interface, even though this example does not use them. begin_epilogue() optionally preloads one Shifted-K value per row fragment so visit() can reuse it from registers.
**CN**: 之所以保留 split-K 和 batch 相关接口，是因为 visitor 需要符合通用 epilogue 接口，尽管本示例并未真正使用它们。begin_epilogue() 会按需为每个行片段预加载一个 Shifted-K 值，便于 visit() 直接从寄存器复用。

```cpp
  /// Called at the start of one step before starting accumulator exchange
  CUTLASS_DEVICE
  void begin_step(int step_idx) {
    fragment_D_.clear();

    if (elementwise_.kScale != cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling) {
      fragment_C_.clear();
      iterator_C_.load(fragment_C_);
      ++iterator_C_;
    }
  }

  /// Called at the start of a row
  CUTLASS_DEVICE
  void begin_row(int row_idx) {

  }
```
**EN**: begin_step() clears the destination fragment and only loads fragment_C_ when the epilogue needs a source tensor term. begin_row() is intentionally empty because this visitor does its real work in visit() and end_row().
**CN**: begin_step() 会清空目标片段，并且只在 epilogue 需要源张量项时才加载 fragment_C_。begin_row() 刻意留空，因为真正的工作都发生在 visit() 和 end_row() 中。

```cpp
  /// Called after accumulators have been exchanged for each accumulator vector
  CUTLASS_DEVICE
  void visit(
    int iter_idx,
    int row_idx,
    int column_idx,
    int frag_idx,
    AccumulatorFragment const &accum) {

    using Mul = cutlass::multiplies<ElementLayernormCompute>;
    using Minus = cutlass::minus<ElementLayernormCompute>;
    using Exp   = cutlass::fast_exp_op<ElementLayernormCompute>;

    [[maybe_unused]] Minus minus;
    [[maybe_unused]] Mul   mul;
    [[maybe_unused]] Exp   exponential;

    LayernormFragment result;

    thread_offset_ =
      iterator_D_.thread_start() +
      OutputTileIterator::ThreadMap::iteration_offset(frag_idx);

    NumericArrayConverter<ElementLayernormCompute, ElementOutput, kElementsPerAccess> source_converter;
    OutputVector &source_vector = reinterpret_cast<OutputVector *>(&fragment_C_)[frag_idx];

    bool column_guard = (thread_offset_.column() < extent_.column());

    if (elementwise_.kScale == cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling) {
      result = source_converter(elementwise_(accum));
    }else{
      result = source_converter(elementwise_(accum, source_vector));
    }
```
**EN**: visit() receives one accumulator fragment from the GEMM epilogue pipeline, computes its thread-local output coordinates, optionally loads the corresponding C fragment, and applies the chosen epilogue functor. The result is promoted to the LayerNorm compute type so the same values can feed both output storage and statistic reduction.
**CN**: visit() 从 GEMM 的 epilogue 流水线中接收一个 accumulator 片段，计算其线程局部输出坐标，按需加载对应的 C 片段，并应用选定的 epilogue functor。结果随后提升到 LayerNorm 计算类型，以便同时服务于输出写回和统计归约。

```cpp
    ElementLayernormCompute inv_scalar = cutlass::constants::one<ElementLayernormCompute>() / ElementLayernormCompute(extent_.column());

    // Fragment is cleared for non-reachable columns so no need to check against column guard
    accum_sum_element_ = element_sum_accumulator_(result);

    // Square sum is different. Non-reachable columns should've been computed for shift-k
    // Otherwise we will incorrectly have some extra k^2 added into square sum.
    if (column_guard) {
      accum_sum_square_ = (kIsShiftedVariance) ? \
                        square_sum_accumulator_(result, shift_k_frag_[iter_idx * kRowIterations + row_idx]) : \
                        square_sum_accumulator_(result);
    }
    else {
      accum_sum_square_ = ElementLayernormCompute(0);
    }

    accum_sum_element_ *= inv_scalar;
    accum_sum_square_ *= inv_scalar;

    // After performing the in-thread reduction, we then perform cross-thread / in-warp reduction
    CUTLASS_PRAGMA_UNROLL
    for (int i = kHalfThreadsPerRow; i > 0; i >>= 1) {
      accum_sum_element_ += __shfl_xor_sync(0xFFFFFFFF, accum_sum_element_, i);
      accum_sum_square_ += __shfl_xor_sync(0xFFFFFFFF, accum_sum_square_, i);
    }

    // Convert to the output
    NumericArrayConverter<ElementOutput, ElementLayernormCompute, kElementsPerAccess> output_converter;
    OutputVector &output = reinterpret_cast<OutputVector *>(&fragment_D_)[frag_idx];
    output = output_converter(result);
  }
```
**EN**: This block is the first-stage reduction: it sums elements and squared elements for the fragment, handles the shifted-variance path when requested, scales by 1 / row_width, and then uses warp shuffles to combine lanes across a row. Importantly, it still writes the ordinary epilogue result to D; full LayerNorm is not finished yet.
**CN**: 这一段就是第一阶段归约：它为当前片段求元素和与平方和，在需要时走 shifted-variance 路径，再乘以 1 / row_width，并通过 warp shuffle 把同一行上的 lane 合并。需要注意的是，这里写回到 D 的仍是普通 epilogue 结果，完整的 LayerNorm 还没有结束。

```cpp
  /// Called at the start of a row
  CUTLASS_DEVICE
  void end_row(int row_idx) {

    using ConvertVarianceOutput = cutlass::NumericConverter<ElementVariance, ElementLayernormCompute>;
    using ConvertMeanOutput = cutlass::NumericConverter<ElementMean, ElementLayernormCompute>;

    ConvertVarianceOutput   convert_variance_output;
    ConvertMeanOutput  convert_mean_output;

    bool is_write_thread = (thread_offset_.row() < extent_.row() && (threadIdx.x % kThreadsPerRow) == 0);
    int row_offset = thread_offset_.row() + blockIdx.y * extent_.row();

    ElementVariance *curr_ptr_sum_square = params_.ptr_Variance + row_offset;
    ElementMean *curr_ptr_element_sum = params_.ptr_Mean + row_offset;

    arch::global_store<ElementVariance, sizeof(ElementVariance)>(
              convert_variance_output(accum_sum_square_),
              (void *)curr_ptr_sum_square,
              is_write_thread);

    arch::global_store<ElementMean, sizeof(ElementMean)>(
              convert_mean_output(accum_sum_element_),
              (void *)curr_ptr_element_sum,
              is_write_thread);

  }
```
**EN**: At the end of each row, only one thread per logical row stores the partial square-sum and partial mean into global memory. Those values are still per-threadblock partials, so ApplyFinalReduction must merge them later.
**CN**: 每一行结束时，只有每个逻辑行中的一个线程会把部分平方和与部分均值写入全局内存。这些值依然只是每个 threadblock 的局部结果，因此后续还需要 ApplyFinalReduction 再做合并。

```cpp
  /// Called after all accumulator elements have been visited
  CUTLASS_DEVICE
  void end_step(int step_idx) {

    iterator_D_.store(fragment_D_);
    ++iterator_D_;
  }

  /// Called after all steps have been completed
  CUTLASS_DEVICE
  void end_epilogue() {

  }

private:

  CUTLASS_DEVICE
  ElementLayernormCompute load_shift_k_(int row_offset, bool is_load) {
    using ConvertShiftK = cutlass::NumericConverter<ElementLayernormCompute, ElementOutput>;
    ConvertShiftK convert_shift_k;
    ElementOutput shift_k_val;

    // Computes the address to load shift_k element
    ElementOutput *curr_ptr_shift_k = params_.ptr_Shifted_K + row_offset;
    // Conditionally loads from global memory
    arch::global_load<ElementOutput, sizeof(ElementOutput)>(shift_k_val, (void *)curr_ptr_shift_k, is_load);
    // Converts data type to return
    ElementLayernormCompute converted_shift_k_val = convert_shift_k(shift_k_val);

    return converted_shift_k_val;
  }
```
**EN**: end_step() stores the computed output fragment and advances the D iterator. end_epilogue() is empty, and load_shift_k_ is a tiny helper that conditionally reads and converts one Shifted-K scalar from global memory.
**CN**: end_step() 负责存储当前输出片段并推进 D 迭代器。end_epilogue() 为空，而 load_shift_k_ 是一个小型辅助函数，用于按条件从全局内存中读取并转换一个 Shifted-K 标量。

```cpp
  CUTLASS_DEVICE
  ElementLayernormCompute square_sum_accumulator_(LayernormFragment const &accum) {
    ElementLayernormCompute sum_ = ElementLayernormCompute(0);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < LayernormFragment::kElements; ++i) {
      auto accum_ = accum[i];
      sum_ += accum_ * accum_;
    }

    return sum_;
  }

  CUTLASS_DEVICE
  ElementLayernormCompute square_sum_accumulator_(LayernormFragment const &accum, ElementLayernormCompute shift_k_val) {
    ElementLayernormCompute sum_ = ElementLayernormCompute(0);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < LayernormFragment::kElements; ++i) {
      auto accum_ = accum[i] - shift_k_val;
      sum_ += accum_ * accum_;
    }

    return sum_;
  }

  CUTLASS_DEVICE
  ElementLayernormCompute element_sum_accumulator_(LayernormFragment const &accum) {
    ElementLayernormCompute sum_ = ElementLayernormCompute(0);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < LayernormFragment::kElements; ++i) {
      sum_ += accum[i];
    }

    return sum_;
  }

};
```
**EN**: These helpers keep the math readable: one overload computes a normal square sum, one subtracts Shifted-K first, and another computes the plain element sum. Splitting them out avoids duplicating the inner loops in visit().
**CN**: 这些辅助函数让数学逻辑更清晰：一个重载计算普通平方和，一个先减去 Shifted-K 再计算平方和，另一个则计算普通元素和。把它们拆出来可以避免在 visit() 中重复写内部循环。

```cpp
} // namespace kernel

/////////////////////////////////////////////////////////////////////////////////////////////////

///
template <
  typename ElementInputA0_,
  typename LayoutInputA0_,
  typename ElementInputB0_,
  typename LayoutInputB0_,
  typename ElementOutput_,
  typename LayoutOutput_,
  typename ElementCompute_,
  typename EpilogueFunctorOp_,
  typename ThreadblockShape_,
  typename WarpShape_,
  typename InstructionShape_,
  int Stages0,
  int Stages1,
  bool IsShiftedVariance_ = false
>
class GemmLayernorm {
```
**EN**: After the low-level kernels, the file defines GemmLayernorm, the user-facing orchestration class. Its template surface exposes input/output types, layouts, epilogue functor, tile shapes, stage counts, and the optional shifted-variance mode.
**CN**: 在底层 kernel 之后，文件定义了面向用户的编排类 GemmLayernorm。它的模板接口暴露了输入/输出类型、布局、epilogue functor、tile 形状、流水级数，以及可选的 shifted-variance 模式。

```cpp
  ///////////////////////////////////////////////////////////////////////////////////////////////

  //
  // Type definitions
  //

  static bool const kInternalTranspose = cutlass::platform::is_same<LayoutOutput_, cutlass::layout::ColumnMajor>::value;
  static bool const kIsShiftedVariance = IsShiftedVariance_;

  // These is mandatory layout.
  using LayoutInputScaleBias = cutlass::layout::RowMajor;

  // These are mandatory data types.
  using ElementLayernormCompute = float;
  using ElementInputScaleBias = cutlass::half_t;

  // These are mandatory params required by mainloop fusion
  using OperatorClass       = cutlass::arch::OpClassTensorOp;
  using ArchTag             = cutlass::arch::Sm80;

  // These are mandatory layouts and data types
  // that are inheritated from pre-defined params

  using LayoutSumSqr = LayoutInputScaleBias;
  using LayoutSum = LayoutInputScaleBias;

  using ElementMean = ElementInputScaleBias;
  using ElementVariance = ElementInputScaleBias;
```
**EN**: The wrapper also fixes several policy choices needed by this fusion path: LayerNorm compute uses float, gamma/beta and statistic buffers use row-major half precision, and the kernels target Tensor Core execution on SM80. This shows the implementation is optimized for a specific high-performance regime rather than being fully generic.
**CN**: 这个包装类还固定了若干与融合路径强相关的策略：LayerNorm 计算使用 float，gamma/beta 与统计缓冲区使用 row-major half 精度，kernel 目标架构是 SM80 上的 Tensor Core。这说明实现更偏向特定高性能场景，而不是完全泛化。

```cpp
  using LayoutInputA0 = LayoutInputA0_;
  using LayoutInputB0 = LayoutInputB0_;
  using LayoutInputA1 = LayoutOutput_;
  using LayoutInputB1 = LayoutOutput_;
  using LayoutOutputC0 = LayoutOutput_;
  using LayoutOutputC1 = LayoutOutput_;

  using ElementInputA0 = ElementInputA0_;
  using ElementInputB0 = ElementInputB0_;
  using ElementOutputC0 = ElementOutput_;
  using ElementCompute = ElementCompute_;
  using ElementInputB1 = ElementInputB0_;

  using ElementInputA1 = ElementOutputC0;
  using ElementOutputC1 = ElementOutputC0;

  using EpilogueFunctorOp = EpilogueFunctorOp_;

  using TensorRefA = TensorRef<ElementInputA0, LayoutInputA0>;
  using TensorRefB = TensorRef<ElementInputB0, LayoutInputB0>;
  using TensorRefC = TensorRef<ElementOutputC0, LayoutOutputC0>;
  using TensorVariance = TensorRef<ElementVariance, LayoutSumSqr>;
  using TensorMean = TensorRef<ElementMean, LayoutSum>;

  using ThreadblockShape = ThreadblockShape_;
  using WarpShape        = WarpShape_;
  using InstructionShape = InstructionShape_;
```
**EN**: These aliases connect stage 0 and stage 1: GEMM1 consumes the layout and element type produced by GEMM0, while TensorRef typedefs define the external runtime API. The shape aliases carry the compile-time tile policy into all subcomponents.
**CN**: 这些别名把第 0 阶段与第 1 阶段连接起来：GEMM1 直接消费 GEMM0 产生的布局和元素类型，而 TensorRef typedef 则定义了对外的运行时接口。形状相关别名把编译期 tile 策略传递给所有子组件。

```cpp
  static int const kStages0 = Stages0;
  static int const kStages1 = Stages1;

  using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

  ///////////////////////////////////////////////////////////////////////////////////////////////

  using MapArguments = cutlass::gemm::kernel::detail::MapArguments<
    ElementInputA0,
    LayoutInputA0,
    cutlass::ComplexTransform::kNone,
    128 / cutlass::sizeof_bits<ElementInputA0>::value,
    ElementInputB0,
    LayoutInputB0,
    cutlass::ComplexTransform::kNone,
    128 / cutlass::sizeof_bits<ElementInputB0>::value,
    LayoutOutputC0,
    kInternalTranspose
  >;

  using DefaultGemmKernel = typename cutlass::gemm::kernel::DefaultGemm<
    typename MapArguments::ElementA,
    typename MapArguments::LayoutA,
    MapArguments::kAlignmentA,
    typename MapArguments::ElementB,
    typename MapArguments::LayoutB,
    MapArguments::kAlignmentB,
    ElementOutputC0,
    typename MapArguments::LayoutC,
    ElementCompute,
    OperatorClass,
    ArchTag,
    ThreadblockShape,
    WarpShape,
    InstructionShape,
    EpilogueFunctorOp,
    SwizzleThreadBlock,
    kStages0,
    true,
    typename cutlass::gemm::device::DefaultGemmConfiguration<
        OperatorClass, ArchTag, ElementInputA0, ElementInputB0, ElementOutputC0, ElementCompute>::Operator,
    cutlass::gemm::SharedMemoryClearOption::kNone
  >::GemmKernel;
```
**EN**: MapArguments normalizes operand ordering and alignment, especially when the requested output layout is column-major and an internal transpose is needed. DefaultGemmKernel then instantiates the baseline CUTLASS GEMM used for the first stage.
**CN**: MapArguments 会统一操作数顺序和对齐要求，尤其是在输出布局为 column-major、需要内部转置时。随后 DefaultGemmKernel 实例化出第一阶段使用的基础 CUTLASS GEMM。

```cpp
  // Epilogue visitor
  using EpilogueVisitor = kernel::EpilogueVisitorLayerNorm<
    ThreadblockShape,
    DefaultGemmKernel::kThreadCount,
    typename DefaultGemmKernel::Epilogue::OutputTileIterator,
    typename DefaultGemmKernel::Epilogue::AccumulatorFragmentIterator::AccumulatorTile,
    ElementCompute,
    ElementVariance,
    ElementMean,
    ElementLayernormCompute,
    EpilogueFunctorOp,
    kIsShiftedVariance
  >;

  /// Epilogue
  using Epilogue = typename cutlass::epilogue::threadblock::EpilogueWithVisitorFromExistingEpilogue<
    EpilogueVisitor,
    typename DefaultGemmKernel::Epilogue
  >::Epilogue;

  // GEMM
  using GemmEpilogueFusion = gemm::kernel::GemmWithEpilogueVisitor<
    typename DefaultGemmKernel::Mma,
    Epilogue,
    SwizzleThreadBlock
  >;

  using ApplyFinalReductionKernel = kernel::ApplyFinalReduction<
    ElementVariance,
    ElementMean,
    ElementLayernormCompute,
    ElementOutputC0,
    ThreadblockShape,
    kIsShiftedVariance
  >;

using GemmMainloopFusion = typename cutlass::gemm::device::GemmLayernormMainloopFusion<
  ElementInputA1, LayoutInputA1,
  ElementInputB1, LayoutInputB1,
  ElementInputScaleBias, LayoutInputScaleBias,
  ElementOutputC1, LayoutOutputC1,
  ElementCompute,
  OperatorClass,
  ArchTag,
  ThreadblockShape,
  WarpShape,
  InstructionShape,
  EpilogueFunctorOp,
  SwizzleThreadBlock,
  kStages1
>;
```
**EN**: This is where the three-part design is composed. GEMM0 is rebuilt with a custom epilogue visitor, ApplyFinalReductionKernel finalizes LayerNorm statistics, and GemmLayernormMainloopFusion implements GEMM1 so normalization, gamma, and beta can be consumed inside the mainloop instead of in a separate post-kernel.
**CN**: 这里是真正把三段式设计拼装起来的地方。GEMM0 被替换为带自定义 epilogue visitor 的版本，ApplyFinalReductionKernel 负责完成 LayerNorm 统计量，而 GemmLayernormMainloopFusion 则实现 GEMM1，使归一化、gamma 和 beta 可以直接在 mainloop 中被消费，而不必再额外启动一个后处理 kernel。

```cpp
  /// Arguments class
  struct Arguments {

    typename GemmEpilogueFusion::Arguments         gemm0;
    typename GemmMainloopFusion::Arguments         gemm1;
    typename ApplyFinalReductionKernel::Arguments reduction;
    cutlass::gemm::GemmCoord extend;

    //
    // Methods
    //
    Arguments() { }

    Arguments(
      cutlass::gemm::GemmCoord problem_size0,
      cutlass::gemm::GemmCoord problem_size1,
      ElementInputA0 * ptr_A,
      ElementInputB0 * ptr_B,
      ElementOutputC0 * ptr_C,
      ElementOutputC0 * ptr_D,
      ElementOutputC0 * ptr_E,
      ElementOutputC0 * ptr_O,
      int64_t    ldm_A,
      int64_t    ldm_B,
      int64_t    ldm_C,
      int64_t    ldm_D,
      int64_t    ldm_E,
      int64_t    ldm_O,
      typename EpilogueFunctorOp::Params linear_scaling,
      TensorVariance ref_Variance_,
      TensorMean ref_Mean_,
      TensorVariance ref_Gamma_,
      TensorMean ref_Beta_,
      ElementOutputC0 *ptr_Shifted_K = nullptr
```
**EN**: The outer Arguments object simply groups the three internal argument packs plus the original GEMM0 extent. That keeps the public API manageable even though the implementation internally launches multiple kernels.
**CN**: 最外层的 Arguments 只是把三个内部参数包和原始 GEMM0 尺寸放在一起。虽然内部会启动多个 kernel，但这种设计仍让对外 API 保持可管理。

```cpp
    ):
      gemm0(
        cutlass::gemm::GemmUniversalMode::kGemm,
        {kInternalTranspose ? problem_size0.n() : problem_size0.m(),\
         kInternalTranspose ? problem_size0.m() : problem_size0.n(),\
         problem_size0.k()},
        {kInternalTranspose ? ptr_B : ptr_A, \
        kInternalTranspose ? ldm_B : ldm_A},
        {kInternalTranspose ? ptr_A : ptr_B, \
        kInternalTranspose ? ldm_A : ldm_B},
        typename EpilogueVisitor::Arguments(
          linear_scaling,
          {ptr_C, ldm_C},
          {ptr_D, ldm_D},
          ref_Variance_.data(),
          ref_Mean_.data(),
          ptr_Shifted_K
        )
      ),
      reduction(
        MatrixCoord(kInternalTranspose ? problem_size0.n() : problem_size0.m(),\
                    kInternalTranspose ? problem_size0.m() : problem_size0.n()),
        ref_Variance_,
        ref_Mean_,
        ptr_Shifted_K
      ),
```
**EN**: The first half of the constructor builds GEMM0 and the final-reduction arguments. It also hides layout-dependent pointer swapping when the output is column-major, so callers pass a consistent high-level problem description.
**CN**: 构造函数的前半部分负责组装 GEMM0 和最终归约 kernel 的参数。同时它隐藏了 column-major 输出时依赖布局的指针交换逻辑，因此调用者仍可使用一致的高层问题描述来传参。

```cpp
      gemm1(
        cutlass::gemm::GemmUniversalMode::kGemm,
        problem_size1,
        1,
        linear_scaling,
        kInternalTranspose ? ptr_E : ptr_D,
        kInternalTranspose ? ptr_D : ptr_E,
        ref_Variance_.data(),
        ref_Mean_.data(),
        ref_Gamma_.data(),
        ref_Beta_.data(),
        ptr_O,
        ptr_O,
        problem_size1.m() * problem_size1.k(),
        problem_size1.n() * problem_size1.k(),
        problem_size1.n(),
        problem_size1.n(),
        problem_size1.k(),
        problem_size1.k(),
        problem_size1.m() * problem_size1.n(),
        problem_size1.m() * problem_size1.n(),
        kInternalTranspose ? ldm_E : ldm_D,
        kInternalTranspose ? ldm_D : ldm_D,
        ref_Variance_.layout().stride(0),
        ref_Mean_.layout().stride(0),
        ref_Gamma_.layout().stride(0),
        ref_Beta_.layout().stride(0),
        ldm_O,
        ldm_O
      ),
      extend(problem_size0)
    {

    }
  };
```
**EN**: The second half prepares GEMM1 with all tensors it needs: the GEMM operands, finalized mean and inverse-std buffers, gamma and beta, and the output tensor. Many explicit stride arguments are forwarded because the fused runtime API is fully general about tensor leading dimensions and batch spacing.
**CN**: 构造函数的后半部分为 GEMM1 准备好全部所需张量：GEMM 操作数、最终的均值与逆标准差缓冲区、gamma、beta，以及输出张量。之所以显式传递大量 stride 参数，是因为这个融合运行时 API 需要对张量 leading dimension 与批间跨度保持通用。

```cpp
  struct Params {

    typename GemmEpilogueFusion::Params         gemm0;
    typename ApplyFinalReductionKernel::Params reduction;
    MatrixCoord extend;
    //
    // Methods
    //
    Params() { }

    Params(Arguments const &args):
      gemm0(args.gemm0),
      reduction(args.reduction),
      extend(MatrixCoord(args.extend.m(), args.extend.n()))
    {

    }
  };
```
**EN**: Params stores only the state needed for the manually launched pieces: GEMM0, the reduction kernel, and the logical extent. GEMM1 is managed separately by the device-side operator object, so it does not live in this Params struct.
**CN**: Params 只保存手动启动部分所需的状态：GEMM0、归约 kernel 和逻辑尺寸。GEMM1 由设备侧 operator 对象单独管理，因此并不放在这个 Params 结构里。

```cpp
private:

  Params params_;
  GemmMainloopFusion gemm_fusion_op;

public:

  /// Ctor
  GemmLayernorm() {

  }

  /// Initialize
  Status initialize(Arguments const &args) {

    params_ = Params(args);
    cutlass::Status status;
    size_t workspace_size = gemm_fusion_op.get_workspace_size(args.gemm1);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
    status = gemm_fusion_op.can_implement(args.gemm1);
    CUTLASS_CHECK(status);

    status = gemm_fusion_op.initialize(args.gemm1, workspace.get());
    CUTLASS_CHECK(status);

    return cutlass::Status::kSuccess;
  }
```
**EN**: The wrapper keeps two members: packed params for the manual launches and a GemmMainloopFusion operator instance for stage 3. initialize() prepares both, including workspace sizing, capability checks, and operator initialization for the fused GEMM1 runtime path.
**CN**: 这个包装类内部主要保留两类成员：供手动 launch 使用的打包参数，以及第三阶段所需的 GemmMainloopFusion operator 实例。initialize() 会同时准备这两部分，包括 workspace 大小查询、可实现性检查，以及融合 GEMM1 运行时路径的初始化。

```cpp
  /// Run
  Status run(cudaStream_t stream) {

    //
    // Launch the GEMM + layernorm kernel
    //

    dim3 gemm_grid = SwizzleThreadBlock().get_grid_shape(params_.gemm0.grid_tiled_shape);
    dim3 gemm_block(GemmEpilogueFusion::kThreadCount, 1, 1);

    int gemm_smem_size = int(sizeof(typename GemmEpilogueFusion::SharedStorage));

    cutlass::Kernel<GemmEpilogueFusion><<<gemm_grid, gemm_block, gemm_smem_size, stream>>>(params_.gemm0);

    cudaError_t result = cudaGetLastError();

    if (result != cudaSuccess) {
      return cutlass::Status::kErrorInternal;
    }
```
**EN**: run() first launches GEMM0 explicitly with a CUTLASS kernel wrapper, grid swizzle, and the shared-storage size computed from the composed kernel type. A CUDA error check immediately follows so failures are reported before later stages run.
**CN**: run() 首先显式启动 GEMM0：使用 CUTLASS kernel 包装器、grid swizzle，以及从组合 kernel 类型推导出的共享内存大小。随后立刻做一次 CUDA 错误检查，确保后续阶段不会在前一阶段失败的情况下继续执行。

```cpp
    //
    // Launch the ApplyFinalReductionKernel
    //

    // always performs reduction from leading dimension
    int leading_dim_0 = kInternalTranspose ? params_.extend.row() : params_.extend.column();
    int leading_dim_1 = kInternalTranspose ? params_.extend.column() : params_.extend.row();

    int thread_per_block = 128;
    int block_per_row = (leading_dim_1 + thread_per_block - 1) / thread_per_block;
    if (block_per_row < 4) {
      thread_per_block = 32;
      block_per_row = (leading_dim_1 + thread_per_block - 1) / thread_per_block;
    }

    dim3 final_reduction_block(thread_per_block);
    dim3 final_reduction_grid(block_per_row);

    Kernel<ApplyFinalReductionKernel><<<
      final_reduction_grid, final_reduction_block, sizeof(typename ApplyFinalReductionKernel::SharedStorage), stream
    >>>(params_.reduction);

    result = cudaGetLastError();

    if (result != cudaSuccess) {
      return cutlass::Status::kErrorInternal;
    }
```
**EN**: Next, the code launches the lightweight final-reduction kernel. It chooses 128 or 32 threads per block based on the row count, because this kernel is bandwidth-light and should stay cheap relative to the two GEMMs.
**CN**: 接着代码会启动轻量级最终归约 kernel。它会根据行数在每个 block 中选择 128 或 32 个线程，因为这个 kernel 的负担远小于前后的两个 GEMM，应该尽量保持低开销。

```cpp
    //
    // Launch the GEMM + mainloop fusion kernel
    //

    cutlass::Status status = gemm_fusion_op();
    CUTLASS_CHECK(status);

    return cutlass::Status::kSuccess;
  }

  /// Function call operator
  Status operator()(cudaStream_t stream = nullptr) {
    return run(stream);
  }
};

/////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace cutlass

/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Finally, GEMM1 is invoked through the device operator object, then operator() forwards to run() for a simple callable API. The file ends by closing the cutlass namespace, completing the wrapper around the three-stage fusion strategy.
**CN**: 最后，GEMM1 通过设备侧 operator 对象被调用，operator() 只是把接口进一步转发到 run()，从而提供一个简洁的可调用 API。文件最后关闭 cutlass 命名空间，完整封装了这一三段式融合策略。

---
## Key Concepts / 关键概念
- Two-stage GEMM + LayerNorm split / 两阶段 GEMM + LayerNorm 拆分
- Epilogue visitor partial reduction / Epilogue visitor 部分归约
- Final reduction writes mean and inverse standard deviation / 最终归约写回均值与逆标准差
- GEMM1 mainloop fusion for normalization, gamma, and beta / 在 GEMM1 主循环中融合归一化、gamma 与 beta
- Layout normalization and internal transpose handling / 布局规范化与内部转置处理
- Runtime orchestration, workspace, and launch sequencing / 运行时编排、工作区与启动顺序

## Dependencies / 依赖项
- `cutlass/cutlass.h` — core CUTLASS types and utilities / CUTLASS 核心类型与工具
- `cutlass/arch/memory.h` — guarded global memory load/store helpers / 条件化全局内存读写辅助
- `cutlass/arch/memory_sm75.h` — architecture-specific memory helpers used by the example / 示例使用的架构相关内存辅助
- `cutlass/gemm/device/gemm_layernorm_mainloop_fusion.h` — GEMM1 device operator with fused LayerNorm consumption / 融合消费 LayerNorm 统计量的 GEMM1 设备算子
- `cutlass/gemm/kernel/default_gemm.h` — baseline GEMM kernel template for stage 0 / 第一阶段使用的基础 GEMM kernel 模板
- `cutlass/epilogue/threadblock/epilogue_with_visitor.h` — visitor-enabled epilogue infrastructure / 支持 visitor 的 epilogue 基础设施
- `gemm_with_epilogue_visitor.h` — example wrapper that combines GEMM with a custom visitor epilogue / 将 GEMM 与自定义 visitor epilogue 组合的示例封装
- `helper.h` — local helper utilities shared by the example / 示例共享的本地辅助工具
