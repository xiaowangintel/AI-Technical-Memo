# predicated_tile_iterator_conv.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/threadblock/predicated_tile_iterator_conv.h`

- **Purpose (EN):** Epilogue for threadblock scoped GEMMs using Tensor Ops. The epilogue rearranges the result of a matrix product through shared memory to match canonical tensor layouts in global memory. Epilogues support conversion and reduction operations.

- **作用 (CN):** 定义 `predicated tile iterator conv` 迭代器，用于在寄存器、共享内存和全局内存之间搬运 epilogue 数据。


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
  \brief Epilogue for threadblock scoped GEMMs using Tensor Ops.
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 34-35

```cpp
  The epilogue rearranges the result of a matrix product through shared memory to match canonical
  tensor layouts in global memory. Epilogues support conversion and reduction operations.
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 37

```cpp
*/
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 39

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 41-55

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"
#include "cutlass/array.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/layout/permute.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/tensor_ref.h"
#include "cutlass/transform/pitch_linear_thread_map.h"
#include "cutlass/epilogue/threadblock/output_tile_thread_map.h"
#include "cutlass/arch/arch.h"
#include "cutlass/arch/memory.h"
#include "cutlass/epilogue/threadblock/predicated_tile_iterator_params.h"
#include "cutlass/conv/conv2d_problem_size.h"
#include "cutlass/conv/conv3d_problem_size.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `cutlass/numeric_types.h`, `cutlass/array.h`, `cutlass/layout/matrix.h`, and 11 more headers. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`cutlass/numeric_types.h`，`cutlass/array.h`，`cutlass/layout/matrix.h`，以及另外 11 个头文件。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 59

```cpp
namespace cutlass {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 63-64

```cpp
namespace epilogue {
namespace threadblock {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 72-83

```cpp
template <
  typename ThreadMap_,       ///< Thread map (conept: OutputTileThreadMap)
  typename Element_,         ///< Element data type
  bool ScatterD = false,     ///< Scatter D operand or not
  typename PermuteDLayout = layout::NoPermute, ///< Permute D operand or not
  bool UseCUDAStore = false,
  int Rank = 4
>
class PredicatedTileIteratorConv {
public:
  using ThreadMap = ThreadMap_;
  using Shape = typename ThreadMap::Shape;
```

**EN:** Declares the templated `PredicatedTileIteratorConv` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time.

**CN:** 声明模板类型 `PredicatedTileIteratorConv`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Line 85

```cpp
  using Element = Element_;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 87-90

```cpp
  static int const kRank = Rank;
  using Layout = typename platform::conditional<kRank == 4,
                                       layout::TensorNHWC,
                                       layout::TensorNDHWC>::type;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 92-93

```cpp
  using Stride = typename Layout::Stride;
  static int const kStrideRank = Layout::kStrideRank;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 95-96

```cpp
  using TensorRef = TensorRef<Element, Layout>;
  using ConstTensorRef = typename TensorRef::ConstTensorRef;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 98-101

```cpp
  using MappedLayout = layout::RowMajor;
  using Index = typename MappedLayout::Index;
  using LongIndex = typename MappedLayout::LongIndex;
  using TensorCoord = typename MappedLayout::TensorCoord;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 103-105

```cpp
  static int const kElementsPerAccess = ThreadMap::kElementsPerAccess;
  static int const kThreads = ThreadMap::kThreads;
  static int const kIterations = ThreadMap::Count::kTile;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Line 107

```cpp
  static bool constexpr PermuteD = !layout::is_trivial_permute<PermuteDLayout>;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 109-112

```cpp
  static_assert( ThreadMap::Iterations::kRow > 0,"ThreadMap::Iterations::kRow must be > 0");
  static_assert( ThreadMap::Iterations::kGroup > 0,"ThreadMap::Iterations::kGroup must be > 0");
  static_assert( ThreadMap::Iterations::kCluster > 0,"ThreadMap::Iterations::kCluster must be > 0");
  static_assert( ThreadMap::Iterations::kColumn > 0,"ThreadMap::Iterations::kColumn must be > 0");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Lines 114-120

```cpp
  /// Fragment object
  using Fragment = Array<
    Element,
    ThreadMap::Iterations::kColumn *
    ThreadMap::Iterations::kRow *
    ThreadMap::Iterations::kGroup *
    ThreadMap::Iterations::kCluster * ThreadMap::kElementsPerAccess>;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 122-123

```cpp
  /// Memory access size
  using AccessType = AlignedArray<Element, ThreadMap::kElementsPerAccess>;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 129-131

```cpp
  /// Uses a non-template class
  struct Params : PredicatedTileIteratorParams {
    using Base = PredicatedTileIteratorParams;
```

**EN:** Defines `struct`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Uses a non-template class.

**CN:** 定义 `struct`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Lines 133-135

```cpp
    /// Fast divmod objects divided by tensor extents
    FastDivmod divmod[kStrideRank - 1];
    Stride tensor_stride;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 137-138

```cpp
    CUTLASS_HOST_DEVICE
    Params() { }
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 140-147

```cpp
    CUTLASS_HOST_DEVICE
    Params(Layout const &layout, cutlass::Tensor4DCoord const &tensor_extent):
      PredicatedTileIteratorParams(
        layout.stride()[0] * int(sizeof(AccessType)) / kElementsPerAccess,
        make_OutputTileThreadMapDesc<ThreadMap>()
      ) {
      divmod[0] = FastDivmod(tensor_extent[2] /* Q for Fprop & W for Deconv*/);
      divmod[1] = FastDivmod(tensor_extent[1] /* P for Fprop & H for Deconv*/);
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 149-153

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kStrideRank; ++i) {
        tensor_stride[i] = layout.stride()[i];
      }
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 155-163

```cpp
    CUTLASS_HOST_DEVICE
    Params(Layout const &layout, cutlass::Tensor5DCoord const &tensor_extent):
      PredicatedTileIteratorParams(
        layout.stride()[0] * int(sizeof(AccessType)) / kElementsPerAccess,
        make_OutputTileThreadMapDesc<ThreadMap>()
      ) {
      divmod[0] = FastDivmod(tensor_extent[3] /* Q for Fprop & W for Deconv*/);
      divmod[1] = FastDivmod(tensor_extent[2] /* P for Fprop & H for Deconv*/);
      divmod[2] = FastDivmod(tensor_extent[1] /* Z for Fprop & D for Deconv*/);
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 165-169

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kStrideRank; ++i) {
        tensor_stride[i] = layout.stride()[i];
      }
    }
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 171-174

```cpp
    CUTLASS_HOST_DEVICE
    Params(Base const &base) :
      Base(base) { }
  };
```

**EN:** This configuration block defines or initializes the lightweight parameter object passed from host code into the epilogue functor.

**CN:** 该配置代码块定义或初始化从主机代码传入 epilogue 函子的轻量参数对象。


### Lines 176-177

```cpp
  /// Mask object
  struct Mask {
```

**EN:** Defines `Mask`, a local type that packages related state, aliases, and helper logic for this epilogue component. The comment highlights: Mask object.

**CN:** 定义 `Mask`，这个本地类型把该 epilogue 组件相关的状态、类型别名和辅助逻辑组织在一起。


### Line 179

```cpp
    static int const kCount = ThreadMap::Iterations::kColumn;
```

**EN:** This block defines helper aliases or constants so the remaining code stays readable while still being fully generic.

**CN:** 该代码块定义辅助别名或常量，使剩余代码在保持泛型的同时更易阅读。


### Lines 181-182

```cpp
    /// Predicate state
    bool predicates[kCount];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 184-190

```cpp
    //
    // Mask
    //
    CUTLASS_HOST_DEVICE
    Mask() {
      enable();
    }
```

**EN:** This method block implements `Mask`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `Mask`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 192-198

```cpp
    ///< Efficiently disables all accesses guarded by mask
    CUTLASS_HOST_DEVICE void clear() {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kCount; ++i) {
        predicates[i] = false;
      }
    }
```

**EN:** This method block implements `clear`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `clear`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 200-207

```cpp
    ///< CUTLASS_HOST_DEVICE enables all accesses guarded by mask
    CUTLASS_DEVICE void enable() {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kCount; ++i) {
        predicates[i] = true;
      }
    }
  };
```

**EN:** This method block implements `enable`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `enable`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 209

```cpp
private:
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


### Lines 215-216

```cpp
  /// Parameters structure containing reference and precomputed state.
  Params params_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 218-219

```cpp
  /// Byte-level pointer. This pointer is usually for both load() and store(), unless PermuteD is performed. When having PermuteD, byte_pointer_ is only for load().
  uint8_t *byte_pointer_;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 221-222

```cpp
  /// Array of boolean values to contain steady-state predicates
  Mask mask_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 224-225

```cpp
  /// Extent of the matrix tile in rows
  Index extent_row_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 227-228

```cpp
  /// Extent of the matrix tile in rows
  Index extent_column_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 230-231

```cpp
  /// A thread's starting row position (assuming steady-state predicates have been computed)
  Index thread_start_row_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 233-234

```cpp
  /// A thread's starting column
  Index thread_start_column_;
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 236-237

```cpp
  /// Internal state counter
  int state_[3];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 243-245

```cpp
  static_assert(sizeof(extent_row_) == 4, "Expected 32b extents");
  static_assert(sizeof(thread_start_row_) == 4, "Expected 32b extents");
  static_assert(sizeof(PredicatedTileIteratorParams::stride) == 8, "Expected 64b strides");
```

**EN:** This block encodes compile-time invariants, rejecting unsupported combinations before any kernel is instantiated.

**CN:** 该代码块通过 `static_assert` 编码编译期约束，在实例化任何 kernel 之前就拒绝不支持的组合。


### Line 247

```cpp
private:
```

**EN:** This access-specifier block switches the following declarations into the `private` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `private` 区域。


### Line 253

```cpp
public:
```

**EN:** This access-specifier block switches the following declarations into the `public` section of the surrounding type.

**CN:** 这个访问控制块把后续声明切换到外围类型的 `public` 区域。


### Lines 259-269

```cpp
  /// Constructor
  CUTLASS_DEVICE
  PredicatedTileIteratorConv(
    Params const & params,
    Element *pointer,
    TensorCoord extent,
    int thread_idx,
    TensorCoord threadblock_offset = TensorCoord()
  ):
    params_(params)
  {
```

**EN:** This method block implements `PredicatedTileIteratorConv`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `PredicatedTileIteratorConv`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 271

```cpp
    TensorCoord thread_offset = ThreadMap::initial_offset(thread_idx) + threadblock_offset;
```

**EN:** This method block implements `initial_offset`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `initial_offset`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 273-274

```cpp
    extent_row_ = extent.row();
    extent_column_ = extent.column();
```

**EN:** This method block implements `row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 276-277

```cpp
    thread_start_row_ = thread_offset.row();
    thread_start_column_ = thread_offset.column();
```

**EN:** This method block implements `row`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `row`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 279-281

```cpp
    // Initialize predicates
    CUTLASS_PRAGMA_UNROLL
    for (int c = 0; c < ThreadMap::Iterations::kColumn; ++c) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 283-285

```cpp
      mask_.predicates[c] = ((thread_offset.column()
        + ThreadMap::Delta::kColumn * c) < extent.column());
    }
```

**EN:** This method block implements `column`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `column`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 287-290

```cpp
    // Null pointer performs no accesses
    if (!pointer) {
      mask_.clear();
    }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 292-294

```cpp
    // Initialize byte_pointer_
    byte_pointer_ = reinterpret_cast<uint8_t *>(pointer) +
      LongIndex(thread_offset.column()) * sizeof(AccessType) / kElementsPerAccess;
```

**EN:** This method block implements `LongIndex`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `LongIndex`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 296-298

```cpp
    // Initialize internal state counter
    state_[0] = state_[1] = state_[2] = 0;
  }
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 300-304

```cpp
  /// Adds a pointer offset in units of Element
  CUTLASS_HOST_DEVICE
  void add_pointer_offset(LongIndex pointer_offset) {
    byte_pointer_ += pointer_offset * sizeof_bits<Element>::value / 8;
  }
```

**EN:** Advances the iterator state so the same object can address a different logical tile or memory location on the next access.

**CN:** 推进迭代器状态，使同一个对象在下一次访问时能够定位到不同的逻辑 tile 或内存位置。


### Lines 306-308

```cpp
  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load_with_byte_offset(Fragment &frag, int64_t byte_offset) const {
```

**EN:** This method block implements `load_with_byte_offset`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `load_with_byte_offset`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 310-311

```cpp
    uint8_t *byte_pointer = byte_pointer_;
    AccessType *frag_ptr = reinterpret_cast<AccessType *>(&frag);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 313-314

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster; ++cluster) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 316-317

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 319-320

```cpp
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 322-323

```cpp
          int frag_row_idx =
            (row + ThreadMap::Iterations::kRow * (group + ThreadMap::Iterations::kGroup * cluster));
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 325-327

```cpp
          int row_offset = row * ThreadMap::Delta::kRow
            + group * ThreadMap::Delta::kGroup
            + cluster * ThreadMap::Delta::kCluster;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 329

```cpp
          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 331

```cpp
          AccessType *memory_pointer = reinterpret_cast<AccessType *>(byte_pointer + byte_offset);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 333

```cpp
          Stride tensor_coord = CoordinateDecompositionLittleEndian<kStrideRank>(row_offset + thread_start_row_, params_.divmod);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 335

```cpp
          LongIndex tensor_offset = dot(tensor_coord, params_.tensor_stride);
```

**EN:** This method block implements `dot`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `dot`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 337-338

```cpp
          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn; ++column) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 340

```cpp
            bool guard = row_guard && mask_.predicates[column];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 342-355

```cpp
            cutlass::arch::global_load<
              AccessType,
              sizeof(AccessType)
            >(
                frag_ptr[frag_row_idx * ThreadMap::Iterations::kColumn +
                         column],
                (void *)&memory_pointer[column * ThreadMap::Delta::kColumn /
                                        kElementsPerAccess + tensor_offset / kElementsPerAccess],
                guard);
          }
        }
      }
    }
  }
```

**EN:** This method block implements `sizeof`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `sizeof`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 357-359

```cpp
  /// Loads a fragment from memory
  CUTLASS_DEVICE
  void load(Fragment &frag) const {
```

**EN:** Implements a load path that gathers data into a fragment while honoring the iterator layout and any pointer offset supplied by the caller.

**CN:** 实现加载路径，在遵守迭代器布局以及调用者提供的指针偏移的同时，把数据收集到片段对象中。


### Lines 361-362

```cpp
    load_with_byte_offset(frag, 0);
  }
```

**EN:** This method block implements `load_with_byte_offset`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `load_with_byte_offset`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 364-368

```cpp
  /// Stores a fragment to memory
  CUTLASS_DEVICE
  void store_with_byte_offset(Fragment const &frag, int64_t byte_offset) const {
    uint8_t *byte_pointer = byte_pointer_;
    AccessType const *frag_ptr = reinterpret_cast<AccessType const *>(&frag);
```

**EN:** This method block implements `store_with_byte_offset`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `store_with_byte_offset`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 370-371

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int cluster = 0; cluster < ThreadMap::Iterations::kCluster; ++cluster) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 373-374

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int group = 0; group < ThreadMap::Iterations::kGroup; ++group) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 376-377

```cpp
        CUTLASS_PRAGMA_UNROLL
        for (int row = 0; row < ThreadMap::Iterations::kRow; ++row) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 379-380

```cpp
          int frag_row_idx =
            (row + ThreadMap::Iterations::kRow * (group + ThreadMap::Iterations::kGroup * cluster));
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 382-384

```cpp
          int row_offset = row * ThreadMap::Delta::kRow
            + group * ThreadMap::Delta::kGroup
            + cluster * ThreadMap::Delta::kCluster;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 386

```cpp
          bool row_guard = ((row_offset + thread_start_row_) < extent_row_);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 388

```cpp
          Stride tensor_coord = CoordinateDecompositionLittleEndian<kStrideRank>((row_offset + thread_start_row_), params_.divmod);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 390

```cpp
          LongIndex tensor_offset = dot(tensor_coord, params_.tensor_stride);
```

**EN:** This method block implements `dot`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `dot`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 392

```cpp
          AccessType *memory_pointer = reinterpret_cast<AccessType *>(byte_pointer + byte_offset);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 394-395

```cpp
          CUTLASS_PRAGMA_UNROLL
          for (int column = 0; column < ThreadMap::Iterations::kColumn; ++column) {
```

**EN:** This method block implements `for`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `for`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Line 397

```cpp
            bool guard = row_guard && mask_.predicates[column];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 399-409

```cpp
            if (UseCUDAStore) {
              if (guard) {
                memory_pointer[tensor_offset / kElementsPerAccess] =
                    frag_ptr[frag_row_idx * ThreadMap::Iterations::kColumn + column];
              }
            } else {
              cutlass::arch::global_store<AccessType, sizeof(AccessType)>(
                  frag_ptr[frag_row_idx * ThreadMap::Iterations::kColumn + column],
                  (void *)&memory_pointer[tensor_offset / kElementsPerAccess],
                  guard);
            }
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 411-416

```cpp
            memory_pointer += (ThreadMap::Delta::kColumn / kElementsPerAccess);
          }
        }
      }
    }
  }
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 418-420

```cpp
  /// Stores a fragment to memory
  CUTLASS_DEVICE
  void store(Fragment const &frag) const {
```

**EN:** Implements a store path that writes a fragment back to memory in the arrangement expected by the epilogue pipeline.

**CN:** 实现存储路径，以 epilogue 流水线期望的排列方式把片段写回内存。


### Lines 422-423

```cpp
    store_with_byte_offset(frag, 0);
  }
```

**EN:** This method block implements `store_with_byte_offset`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `store_with_byte_offset`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 425-428

```cpp
  CUTLASS_DEVICE
  MatrixCoord thread_start() const {
    return MatrixCoord(thread_start_row_, thread_start_column_);
  }
```

**EN:** This method block implements `thread_start`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `thread_start`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 430-434

```cpp
  /// Need to get the thread start row from the tile iterator
  CUTLASS_DEVICE
  int32_t thread_start_row() const {
    return thread_start_row_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 436-440

```cpp
  /// Need to get the thread start row from the tile iterator
  CUTLASS_DEVICE
  int32_t thread_start_column() const {
    return thread_start_column_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 442-446

```cpp
  /// Extent of the matrix in rows
  CUTLASS_DEVICE
  Index extent_row() const {
    return extent_row_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 448-452

```cpp
  /// Extent of the matrix in columns
  CUTLASS_DEVICE
  Index extent_column() const {
    return extent_column_;
  }
```

**EN:** These lines declare the private or protected data members that preserve iterator state or cached runtime parameters.

**CN:** 这些语句声明私有/受保护数据成员，用于保存迭代器状态或缓存运行时参数。


### Lines 454-456

```cpp
  /// Advances to the next position to load or store
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorConv &operator++() {
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 458

```cpp
    ++state_[0];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 460

```cpp
    thread_start_row_ += ThreadMap::Shape::kRow;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 462

```cpp
    if (state_[0] == ThreadMap::Count::kRow) {
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 464-465

```cpp
      state_[0] = 0;
      ++state_[1];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 467-468

```cpp
      thread_start_row_ += (ThreadMap::Shape::kGroup - 1) *
        ThreadMap::Shape::kRow * ThreadMap::Count::kRow;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 470

```cpp
      if (state_[1] == ThreadMap::Count::kGroup) {
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 472-473

```cpp
        state_[1] = 0;
        ++state_[2];
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 475-476

```cpp
        thread_start_row_ += ThreadMap::Count::kGroup *
          ThreadMap::Shape::kGroup * ThreadMap::Count::kRow * ThreadMap::Shape::kRow;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 478-479

```cpp
        if (state_[2] == ThreadMap::Count::kCluster) {
          state_[2] = 0;
```

**EN:** This method block implements `if`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `if`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 481-485

```cpp
          thread_start_row_ += ThreadMap::Shape::kGroup * ThreadMap::Shape::kRow
            * ThreadMap::Shape::kCluster * ThreadMap::Shape::kTile;
        }
      }
    }
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 487-488

```cpp
    return *this;
  }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 490-497

```cpp
  /// Advances a number of positions to load or store
  CUTLASS_HOST_DEVICE
  PredicatedTileIteratorConv &operator+=(int increment)
  {
    // Row
    state_[0] += increment;
    int increment_row = state_[0] / ThreadMap::Count::kRow;
    state_[0] = state_[0] % ThreadMap::Count::kRow;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 499

```cpp
    thread_start_row_ += (ThreadMap::Shape::kRow * increment);
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 501-504

```cpp
    // Group
    state_[1] += increment_row;
    int increment_group = state_[1] / ThreadMap::Count::kGroup;
    state_[1] = state_[1] % ThreadMap::Count::kGroup;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 506-510

```cpp
    thread_start_row_ +=
        (ThreadMap::Shape::kGroup - 1) *
        ThreadMap::Shape::kRow *
        ThreadMap::Count::kRow *
        increment_row;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 512-515

```cpp
    // Cluster
    state_[2] += increment_group;
    int increment_cluster = state_[2] / ThreadMap::Count::kCluster;
    state_[2] = state_[2] % ThreadMap::Count::kCluster;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 517-522

```cpp
    thread_start_row_ +=
        ThreadMap::Count::kGroup *
        ThreadMap::Shape::kGroup *
        ThreadMap::Count::kRow *
        ThreadMap::Shape::kRow *
        increment_group;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 524-530

```cpp
    // Tile
    thread_start_row_ +=
        ThreadMap::Shape::kGroup *
        ThreadMap::Shape::kRow *
        ThreadMap::Shape::kCluster *
        ThreadMap::Shape::kTile *
        increment_cluster;
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Lines 532-533

```cpp
    return *this;
  }
```

**EN:** This logic block performs local control flow such as predication, iteration, branching, or value selection inside the algorithm.

**CN:** 这个逻辑代码块负责算法内部的局部控制流程，例如边界判断、循环、分支选择或值的筛选。


### Lines 535-538

```cpp
  ///< Efficiently disables all accesses guarded by mask
  CUTLASS_DEVICE void clear_mask() {
    mask_.clear();
  }
```

**EN:** This method block implements `clear_mask`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `clear_mask`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 540-543

```cpp
  ///< Efficiently enables all accesses guarded by mask
  CUTLASS_DEVICE void enable_mask() {
    mask_.enable();
  }
```

**EN:** This method block implements `enable_mask`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `enable_mask`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 545-548

```cpp
  ///< Sets the mask
  CUTLASS_DEVICE void get_mask(Mask &mask) const {
    mask = mask_;
  }
```

**EN:** This method block implements `get_mask`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `get_mask`，为外围 epilogue 组件提供一个聚焦的行为片段。


### Lines 550-554

```cpp
  ///< Sets the mask
  CUTLASS_DEVICE void set_mask(Mask const &mask) {
    mask_ = mask;
  }
};
```

**EN:** This method block implements `set_mask`, contributing a focused piece of behavior needed by the surrounding epilogue component.

**CN:** 这个方法代码块实现了 `set_mask`，为外围 epilogue 组件提供一个聚焦的行为片段。


## Key Concepts / 关键概念

- **Threadblock staging / 线程块级暂存:** Organizes shared-memory staging, tile iterators, and output traversal for threadblock-scope epilogues. / 为线程块范围的 epilogue 组织共享内存暂存、tile 迭代器和输出遍历。

- **Iterator abstraction / 迭代器抽象:** Encapsulates pointer arithmetic, tile stepping, and fragment load/store details. / 封装指针运算、tile 步进以及片段读写细节。

- **Predication / 谓词保护:** Protects boundary tiles so out-of-range accesses are masked instead of written blindly. / 通过谓词保护边界 tile，避免越界位置被盲目读写。

- **Tensor-op path / Tensor-op 路径:** Matches the epilogue layout and iterator strategy to tensor-core style MMA pipelines when applicable. / 在适用时让 epilogue 的布局和迭代策略与 tensor core 风格的 MMA 流水线相匹配。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cutlass/numeric_types.h`, `cutlass/array.h`, `cutlass/layout/matrix.h`, `cutlass/layout/tensor.h`, `cutlass/layout/permute.h`, `cutlass/matrix_shape.h`, `cutlass/tensor_ref.h`, `cutlass/transform/pitch_linear_thread_map.h`, `cutlass/epilogue/threadblock/output_tile_thread_map.h`, `cutlass/arch/arch.h`, `cutlass/arch/memory.h`, `cutlass/epilogue/threadblock/predicated_tile_iterator_params.h`, `cutlass/conv/conv2d_problem_size.h`, `cutlass/conv/conv3d_problem_size.h`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::threadblock`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/threadblock/output_tile_thread_map.h`, `cutlass/epilogue/threadblock/predicated_tile_iterator_params.h`

- **Feature macros / 特性宏:** `CUTLASS_DEVICE`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`
