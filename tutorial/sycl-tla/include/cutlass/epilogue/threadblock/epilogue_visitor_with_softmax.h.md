# epilogue_visitor_with_softmax.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/threadblock/epilogue_visitor_with_softmax.h`

- **Purpose (EN):** Epilogue visitor for threadblock scoped GEMMs that process softmax computations in epilogue. The epilogue finds max values in each row of the row-major output matrix and stores them. The max values are also used for a further round of threadblock scoped reduction operation, where the partial reduction results are stored in a pre-allocated array and used for further full reduction.

- **作用 (CN):** 实现 `epilogue visitor with Softmax` 访问者逻辑，用于遍历融合后的 epilogue 操作树。


## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
  \brief Epilogue visitor for threadblock scoped GEMMs that process softmax computations in epilogue.
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 34-36

```cpp
  The epilogue finds max values in each row of the row-major output matrix and stores them.
  The max values are also used for a further round of threadblock scoped reduction operation, where
  the partial reduction results are stored in a pre-allocated array and used for further full reduction.
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 38

```cpp
*/
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 40

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 44-48

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/arch/memory.h"
#include "cutlass/arch/memory_sm75.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/fast_math.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `cutlass/arch/memory.h`, `cutlass/arch/memory_sm75.h`, `cutlass/numeric_conversion.h`, and 1 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`cutlass/arch/memory.h`，`cutlass/arch/memory_sm75.h`，`cutlass/numeric_conversion.h`，以及另外 1 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 50-52

```cpp
namespace cutlass {
namespace epilogue {
namespace threadblock {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 54-66

```cpp
template <
  typename ThreadblockShape_,
  int ThreadCount,
  typename OutputTileIterator_,
  typename ElementAccumulator_,
  typename ElementNorm_,
  typename ElementSum_,
  typename ElementSoftmaxCompute_,
  typename ElementwiseFunctor_,
  bool UseMasking_ = false
>
class EpilogueVisitorSoftmax {
public:
```

**EN:** Declares the templated `EpilogueVisitorSoftmax` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `EpilogueVisitorSoftmax`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 68-69

```cpp
  using ThreadblockShape   = ThreadblockShape_;
  static int const kThreadCount = ThreadCount;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 71-72

```cpp
  using OutputTileIterator = OutputTileIterator_;
  using ElementwiseFunctor = ElementwiseFunctor_;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 74-75

```cpp
  static int const kIterations = OutputTileIterator::kIterations;
  static int const kElementsPerAccess = OutputTileIterator::kElementsPerAccess;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 77-79

```cpp
  using ElementOutput = typename OutputTileIterator::Element;
  using LayoutOutput = cutlass::layout::RowMajor;
  using ElementAccumulator = ElementAccumulator_;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 81-83

```cpp
  using ElementNorm = ElementNorm_;
  using ElementSum = ElementSum_;
  using ElementSoftmaxCompute = ElementSoftmaxCompute_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 85-88

```cpp
  using AccumulatorFragment = Array<ElementAccumulator, kElementsPerAccess>;
  using SoftmaxFragment = Array<ElementSoftmaxCompute, kElementsPerAccess>;
  using OutputVector = Array<ElementOutput, kElementsPerAccess>;
  using TensorRefD = TensorRef<ElementOutput, LayoutOutput>;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 90-92

```cpp
  static int const kThreadsPerRow = OutputTileIterator::ThreadMap::Detail::kAccessWidth;
  static bool const kHasMultiStepsInRow = (OutputTileIterator::ThreadMap::Iterations::kColumn > 1);
  static bool const kUseMasking = UseMasking_;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 94-95

```cpp
  /// Argument structure
  struct Arguments {
```

**EN:** Defines `Arguments`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Argument structure.

**CN:** 定义 `Arguments`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 97-101

```cpp
    typename ElementwiseFunctor::Params   elementwise;
    int64_t                               batch_stride_C;
    int64_t                               batch_stride_D;
    int64_t                               batch_stride_Max;
    int64_t                               batch_stride_Sum;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 103-111

```cpp
    //
    // Methods
    //
    Arguments():
      batch_stride_C(0),
      batch_stride_D(0),
      batch_stride_Max(0),
      batch_stride_Sum(0)
    {
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 115-123

```cpp
    Arguments(
      typename ElementwiseFunctor::Params   elementwise_
    ):
      elementwise(elementwise_),
      batch_stride_C(0),
      batch_stride_D(0),
      batch_stride_Max(0),
      batch_stride_Sum(0)
    {
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 127-139

```cpp
    Arguments(
      typename ElementwiseFunctor::Params   elementwise_,
      int64_t                               batch_stride_C_,
      int64_t                               batch_stride_D_,
      int64_t                               batch_stride_Max_,
      int64_t                               batch_stride_Sum_
    ):
      elementwise(elementwise_),
      batch_stride_C(batch_stride_C_),
      batch_stride_D(batch_stride_D_),
      batch_stride_Max(batch_stride_Max_),
      batch_stride_Sum(batch_stride_Sum_)
    {
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Line 145

```cpp
  struct Params {
```

**EN:** Defines `Params`, a local type that packages related state, aliases, and helper logic for this epilogue component.

**CN:** 定义 `Params`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 147-157

```cpp
    typename ElementwiseFunctor::Params   elementwise;
    int64_t                               batch_stride_C;
    int64_t                               batch_stride_D;
    int64_t                               batch_stride_Max;
    int64_t                               batch_stride_Sum;
    //
    // Methods
    //
    CUTLASS_HOST_DEVICE
    Params()
    {
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 161-168

```cpp
    CUTLASS_HOST_DEVICE
    Params(Arguments const &args):
      elementwise(args.elementwise),
      batch_stride_C(args.batch_stride_C),
      batch_stride_D(args.batch_stride_D),
      batch_stride_Max(args.batch_stride_Max),
      batch_stride_Sum(args.batch_stride_Sum)
    {
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 173-174

```cpp
  /// Shared storage
  struct SharedStorage {
```

**EN:** Defines `SharedStorage`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Shared storage.

**CN:** 定义 `SharedStorage`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Line 178

```cpp
private:
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


### Lines 180-184

```cpp
  Params const &                        params_;
  SharedStorage &                       shared_storage_;
  MatrixCoord                           extent_;
  MatrixCoord                           extent_real_;
  ElementwiseFunctor                    elementwise_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 186-189

```cpp
  OutputTileIterator                    iterator_C_;
  OutputTileIterator                    iterator_D_;
  typename OutputTileIterator::Fragment fragment_C_;
  typename OutputTileIterator::Fragment fragment_D_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 191-192

```cpp
  ElementAccumulator                    alpha_;
  ElementAccumulator                    beta_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 194-195

```cpp
  ElementNorm                           *ptr_Max_;
  ElementSum                            *ptr_Sum_;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 197

```cpp
  int                                   column_offset_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 199-200

```cpp
  ElementSoftmaxCompute                 accum_max_;
  ElementSoftmaxCompute                 accum_sum_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Line 202

```cpp
  MatrixCoord                           thread_offset_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Line 204

```cpp
  float                                 infinity_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Line 206

```cpp
public:
```

**EN:** This access-specifier block switches the following declarations into the `public` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `public` 区域。


### Lines 208-240

```cpp
  CUTLASS_DEVICE
  EpilogueVisitorSoftmax(
    Params const &params,
    SharedStorage &shared_storage,
    cutlass::MatrixCoord const &problem_size,
    int thread_idx,
    int warp_idx,
    int lane_idx,
    typename OutputTileIterator::Params params_C,
    typename OutputTileIterator::Params params_D,
    typename OutputTileIterator::Element *ptr_C,
    typename OutputTileIterator::Element *ptr_D,
    ElementNorm *ptr_Max = nullptr,
    ElementSum *ptr_Sum = nullptr,
    cutlass::MatrixCoord const &threadblock_offset = cutlass::MatrixCoord(0, 0),
    int column_offset = 0,
    cutlass::MatrixCoord const &problem_size_real = cutlass::MatrixCoord(0, 0),
    float infinity = 10000.0f
  ):
    params_(params),
    shared_storage_(shared_storage),
    extent_(problem_size),
    elementwise_(params.elementwise),
    iterator_C_(params_C, ptr_C, problem_size, thread_idx, threadblock_offset),
    iterator_D_(params_D, ptr_D, problem_size, thread_idx, threadblock_offset),
    ptr_Max_(ptr_Max),
    ptr_Sum_(ptr_Sum),
    column_offset_(column_offset),
    extent_real_(problem_size_real),
    infinity_(infinity)
  {
    alpha_ = (params.elementwise.alpha_ptr ? *params.elementwise.alpha_ptr : params.elementwise.alpha);
    beta_ =  (params.elementwise.beta_ptr ? *params.elementwise.beta_ptr : params.elementwise.beta);
```

**EN:** This method block implements `EpilogueVisitorSoftmax`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `EpilogueVisitorSoftmax`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 242-245

```cpp
    if (beta_ == ElementAccumulator()) {
      iterator_C_.clear_mask();
    }
  }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 247-251

```cpp
  /// Helper to indicate split-K behavior
  CUTLASS_DEVICE
  void set_k_partition(
    int split_k_index,                                            ///< Index of this threadblock within split-K partitioned scheme
    int split_k_slices) {                                         ///< Total number of split-K slices
```

**EN:** This helper method adjusts or queries runtime behavior so the epilogue can avoid unnecessary work for special scaling modes or split-K cases.

**CN:** 这个辅助方法用于调整或查询运行时行为，使 epilogue 在特殊缩放模式或 split-K 场景下避免不必要的工作。


### Lines 255-260

```cpp
  /// Called to set the batch index
  CUTLASS_DEVICE
  void set_batch_index(int batch_idx) {
    iterator_C_.add_pointer_offset(batch_idx * params_.batch_stride_C);
    iterator_D_.add_pointer_offset(batch_idx * params_.batch_stride_D);
  }
```

**EN:** This method block implements `set_batch_index`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `set_batch_index`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 262-264

```cpp
  /// Called at the start of the epilogue just before iterating over accumulator slices
  CUTLASS_DEVICE
  void begin_epilogue() {
```

**EN:** This method block implements `begin_epilogue`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `begin_epilogue`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 268-272

```cpp
  /// Called at the start of one step before starting accumulator exchange
  CUTLASS_DEVICE
  void begin_step(int step_idx) {
    fragment_D_.clear();
    fragment_C_.clear();
```

**EN:** This method block implements `begin_step`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `begin_step`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 274-277

```cpp
    if (elementwise_.kScale != cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling) {
      iterator_C_.load(fragment_C_);
      ++iterator_C_;
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 281-285

```cpp
  /// Called at the start of a row
  CUTLASS_DEVICE
  void begin_row(int row_idx) {
    // Clear accumulators for max and sum when starting a whole row
    clear_accum_();
```

**EN:** This method block implements `begin_row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `begin_row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 289-296

```cpp
  /// Called after accumulators have been exchanged for each accumulator vector
  CUTLASS_DEVICE
  void visit(
    int iter_idx,
    int row_idx,
    int column_idx,
    int frag_idx,
    AccumulatorFragment const &accum) {
```

**EN:** This method block implements `visit`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `visit`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 298-300

```cpp
    using Mul = cutlass::multiplies<SoftmaxFragment>;
    using Minus = cutlass::minus<SoftmaxFragment>;
    using Exp   = cutlass::fast_exp_op<SoftmaxFragment>;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 302-303

```cpp
    Minus     minus;
    Exp       exponential;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 305

```cpp
    SoftmaxFragment result;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 307-308

```cpp
    NumericArrayConverter<ElementSoftmaxCompute, ElementOutput, kElementsPerAccess> source_converter;
    OutputVector &source_vector = reinterpret_cast<OutputVector *>(&fragment_C_)[frag_idx];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 310-314

```cpp
    if (elementwise_.kScale == cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling) {
      result = source_converter(elementwise_(accum));
    }else{
      result = source_converter(elementwise_(accum, source_vector));
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 316-318

```cpp
    thread_offset_ =
      iterator_D_.thread_start() +
      OutputTileIterator::ThreadMap::iteration_offset(frag_idx);
```

**EN:** This method block implements `thread_start`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `thread_start`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 320

```cpp
    bool column_guard = (thread_offset_.column() < extent_.column());
```

**EN:** This method block implements `column`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `column`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 322-326

```cpp
    if (kUseMasking) {
      int elements_in_boundary = extent_real_.column() - thread_offset_.column();
      elements_in_boundary = (elements_in_boundary > kElementsPerAccess) ? kElementsPerAccess : elements_in_boundary;
      elementwise_padding_(result, elements_in_boundary);
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 328

```cpp
    ElementSoftmaxCompute accum_max_prev = accum_max_;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 330-342

```cpp
    // Compute the maximum within one row
    if (!column_idx) {
      // This is the first fragment in a new row
      if (column_guard) {
        accum_max_ = maximum_accumulator_(result);
      }
    }
    else {
      // This is an additional fragment in the same row
      if (column_guard) {
        accum_max_ = maximum_accumulator_(result, accum_max_);
      }
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 344-345

```cpp
    // proactively compute max in warps
    accum_max_ = warp_reduce_max_(accum_max_);
```

**EN:** This method block implements `warp_reduce_max_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `warp_reduce_max_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 347

```cpp
    ElementSoftmaxCompute updater = fast_exp(accum_max_prev - accum_max_);
```

**EN:** This method block implements `fast_exp`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `fast_exp`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 349

```cpp
    SoftmaxFragment intermediate = exponential(minus(result, accum_max_));
```

**EN:** This method block implements `exponential`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `exponential`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 351-363

```cpp
    if (kHasMultiStepsInRow) {
      if (!column_idx) {
        accum_sum_ = (column_guard) ? \
          sum_accumulator_(intermediate) : ElementSoftmaxCompute(0);
      } else {
        // Algorithm in $3.1, https://arxiv.org/pdf/2205.14135v1.pdf
        // S* = S* x updater + sum_row(P'), where updater = exp(M* - M_row)
        accum_sum_ = (column_guard) ? \
          sum_accumulator_(intermediate, accum_sum_ * updater) : accum_sum_ * updater;
      }
    } else {
      accum_sum_ = (column_guard) ? sum_accumulator_(intermediate, accum_sum_) : ElementSoftmaxCompute(0);
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 365-369

```cpp
    // Convert to the output
    NumericArrayConverter<ElementOutput, ElementSoftmaxCompute, kElementsPerAccess> output_converter;
    OutputVector &output = reinterpret_cast<OutputVector *>(&fragment_D_)[frag_idx];
    output = output_converter(result);
  }
```

**EN:** This method block implements `output_converter`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `output_converter`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 371-373

```cpp
  /// Called at the end of a row
  CUTLASS_DEVICE
  void end_row(int row_idx) {
```

**EN:** This method block implements `end_row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end_row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 375-376

```cpp
    using ConvertSumOutput = cutlass::NumericConverter<ElementSum, ElementSoftmaxCompute>;
    using ConvertNormOutput = cutlass::NumericConverter<ElementNorm, ElementSoftmaxCompute>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 378-379

```cpp
    ConvertSumOutput   convert_sum_output;
    ConvertNormOutput  convert_norm_output;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 381-382

```cpp
    // Compute accumulate sum only in the last step
    accum_sum_ = warp_reduce_sum_(accum_sum_);
```

**EN:** This method block implements `warp_reduce_sum_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `warp_reduce_sum_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 384-386

```cpp
    bool is_first_thread_in_tile = ((threadIdx.x % kThreadsPerRow) == 0);
    bool row_guard = thread_offset_.row() < extent_.row();
    bool is_write_thread = row_guard && is_first_thread_in_tile;
```

**EN:** This method block implements `row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 388

```cpp
    int block_batch = blockIdx.z;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 390-391

```cpp
    ElementNorm *curr_ptr_max = ptr_Max_ + thread_offset_.row() + column_offset_ + block_batch * params_.batch_stride_Max;
    ElementSum *curr_ptr_sum = ptr_Sum_ + thread_offset_.row() + column_offset_ + block_batch * params_.batch_stride_Sum;
```

**EN:** This method block implements `row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 393-396

```cpp
    arch::global_store<ElementNorm, sizeof(ElementNorm)>(
              convert_norm_output(accum_max_),
              (void *)curr_ptr_max,
              is_write_thread);
```

**EN:** This method block implements `sizeof`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `sizeof`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 398-401

```cpp
    arch::global_store<ElementSum, sizeof(ElementSum)>(
              convert_sum_output(accum_sum_),
              (void *)curr_ptr_sum,
              is_write_thread);
```

**EN:** This method block implements `sizeof`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `sizeof`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 403-404

```cpp
    // Clear accumulators for max and sum when finishing a whole row
    clear_accum_();
```

**EN:** This method block implements `clear_accum_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `clear_accum_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 408-410

```cpp
  /// Called after all accumulator elements have been visited
  CUTLASS_DEVICE
  void end_step(int step_idx) {
```

**EN:** This method block implements `end_step`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end_step`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 412-414

```cpp
    iterator_D_.store(fragment_D_);
    ++iterator_D_;
  }
```

**EN:** Implements a store path that writes a fragment back to memory in the arrangement expected by the epilogue pipeline.

**CN:** 实现存储路径，以 epilogue 流水线期望的排列方式把片段写回内存。


### Lines 416-418

```cpp
  /// Called after all steps have been completed
  CUTLASS_DEVICE
  void end_epilogue() {
```

**EN:** This method block implements `end_epilogue`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `end_epilogue`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 422

```cpp
private:
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


### Lines 424-430

```cpp
  CUTLASS_DEVICE
  void elementwise_padding_(SoftmaxFragment &result, int elements_in_boundary) {
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < SoftmaxFragment::kElements; ++i) {
      result[i] = (i < elements_in_boundary) ? result[i] : ElementSoftmaxCompute(-infinity_);
    }
  }
```

**EN:** This method block implements `elementwise_padding_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `elementwise_padding_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 432-441

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute warp_reduce_sum_(ElementSoftmaxCompute sum_) {
    int half_thread_in_row = (kThreadsPerRow >> 1);
    CUTLASS_PRAGMA_UNROLL
    for (int i = half_thread_in_row; i > 0; i >>= 1) {
      ElementSoftmaxCompute tmp = __shfl_xor_sync(0xFFFFFFFF, sum_, i);
      sum_ += tmp;
    }
    return sum_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 443-452

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute warp_reduce_max_(ElementSoftmaxCompute max_) {
    int half_thread_in_row = (kThreadsPerRow >> 1);
    CUTLASS_PRAGMA_UNROLL
    for (int i = half_thread_in_row; i > 0; i >>= 1) {
      ElementSoftmaxCompute tmp = __shfl_xor_sync(0xFFFFFFFF, max_, i);
      max_ = fast_max(max_, tmp);
    }
    return max_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 454-455

```cpp
  CUTLASS_DEVICE
  void clear_accum_() {
```

**EN:** This method block implements `clear_accum_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `clear_accum_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 457-461

```cpp
    uint32_t float_max_bits = 0xff7fffff;   // -FLT_MAX
    float min_float = reinterpret_cast<float const &>(float_max_bits);
    accum_max_ = ElementSoftmaxCompute(min_float);
    accum_sum_ = ElementSoftmaxCompute(0);
  }
```

**EN:** This method block implements `ElementSoftmaxCompute`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `ElementSoftmaxCompute`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 463-465

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute sum_accumulator_(SoftmaxFragment const &accum) {
    ElementSoftmaxCompute sum_ = ElementSoftmaxCompute(0);
```

**EN:** This method block implements `sum_accumulator_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `sum_accumulator_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 467-470

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < SoftmaxFragment::kElements; ++i) {
      sum_ += ElementSoftmaxCompute(accum[i]);
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 472-473

```cpp
    return sum_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 475-477

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute sum_accumulator_(SoftmaxFragment const &accum, ElementSoftmaxCompute sum_) {
    // ElementSoftmaxCompute sum_ = ElementSoftmaxCompute(0);
```

**EN:** This method block implements `sum_accumulator_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `sum_accumulator_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 479-482

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < SoftmaxFragment::kElements; ++i) {
      sum_ += ElementSoftmaxCompute(accum[i]);
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 484-485

```cpp
    return sum_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 487-489

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute maximum_accumulator_(SoftmaxFragment const &accum) {
    ElementSoftmaxCompute max_ = accum[0];
```

**EN:** This method block implements `maximum_accumulator_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `maximum_accumulator_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 491-494

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 1; i < SoftmaxFragment::kElements; ++i) {
      max_ = fast_max(max_, ElementSoftmaxCompute(accum[i]));
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 496-497

```cpp
    return max_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 499-500

```cpp
  CUTLASS_DEVICE
  ElementSoftmaxCompute maximum_accumulator_(SoftmaxFragment const &accum, ElementSoftmaxCompute max_) {
```

**EN:** This method block implements `maximum_accumulator_`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `maximum_accumulator_`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 502-505

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < SoftmaxFragment::kElements; ++i) {
      max_ = fast_max(max_, ElementSoftmaxCompute(accum[i]));
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 507-509

```cpp
    return max_;
  }
};
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


## Key Concepts / 关键概念

- **Fusion framework / 融合框架:** Uses callbacks, visitors, or operation tags to compose multiple post-processing steps into the epilogue. / 通过回调、访问者或操作标签把多个后处理步骤组合进 epilogue。

- **Threadblock staging / 线程块级暂存:** Organizes shared-memory staging, tile iterators, and output traversal for threadblock-scope epilogues. / 为线程块范围的 epilogue 组织共享内存暂存、tile 迭代器和输出遍历。

- **Post-ops / 后处理算子:** Fuses nonlinear activation or other elementwise transforms directly into the epilogue. / 把非线性激活或其他逐元素变换直接融合进 epilogue。

- **Scaling semantics / 缩放语义:** Tracks how alpha/beta parameters mix accumulator values with source tensors during output generation. / 描述 alpha/beta 参数如何在生成输出时混合累加器值与源张量。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cutlass/arch/memory.h`, `cutlass/arch/memory_sm75.h`, `cutlass/numeric_conversion.h`, `cutlass/fast_math.h`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::threadblock`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`
