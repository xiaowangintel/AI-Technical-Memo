# mma_tensor_op_fragment_iterator_without_output_op.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/44_multi_gemm_ir_and_codegen/fixed_impl/gemm/warp/mma_tensor_op_fragment_iterator_without_output_op.h`
**Purpose / 用途**: Provides a pure accumulator fragment iterator for Tensor Core MMA paths without an output operator. / 提供一个面向 Tensor Core MMA 路径、且不带输出算子的纯累加器片段迭代器。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–34 — License Header / 许可证头

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

#pragma once

#include "cutlass/cutlass.h"
```
**EN**: Standard NVIDIA BSD-3-Clause license block covering all contributors from 2017 to 2026.

**CN**: 标准 NVIDIA BSD-3 条款许可证声明，覆盖 2017 至 2026 年的所有贡献者。

---

### Lines 35–44 — Includes / 头文件包含

```cpp

#include "cutlass/array.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/layout/tensor.h"
#include "cutlass/numeric_conversion.h"

namespace cutlass {
namespace gemm {
namespace warp {
```
**EN**: `cutlass.h` pulls in core macros (`CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`). `array.h` / `matrix_shape.h` provide `Array<T,N>` and `MatrixShape<R,C>`. The layout headers supply `ColumnMajor` / `RowMajor` tags used as specialisation discriminants. `numeric_conversion.h` is included for potential element-type conversion (used in the row-major specialisation).

**CN**: `cutlass.h` 引入核心宏（`CUTLASS_HOST_DEVICE`、`CUTLASS_PRAGMA_UNROLL`）。`array.h` / `matrix_shape.h` 提供 `Array<T,N>` 和 `MatrixShape<R,C>`。布局头文件提供 `ColumnMajor` / `RowMajor` 标签，用作特化的区分标志。`numeric_conversion.h` 为潜在的元素类型转换而包含（在行主序特化中使用）。

---

### Lines 45–50 — Namespace Opening / 命名空间开启

```cpp


////////////////////////////////////////////////////////////////////////////////

template <
    /// Size of the matrix to load (concept: MatrixShape)
```
**EN**: The iterator lives in `cutlass::gemm::warp`, consistent with other warp-level MMA components in CUTLASS.

**CN**: 迭代器位于 `cutlass::gemm::warp` 命名空间中，与 CUTLASS 中其他 warp 级 MMA 组件保持一致。

---

### Lines 51–70 — Primary Template Declaration / 主模板声明

```cpp
    typename Shape_,
    /// Size of the accumulation tile shape (concept: MatrixShape)
    typename AccumulatorShape_,
    /// KBlocks columns to compute residual
    int KBlocksColumn_,
    /// Accumulator Element type
    typename ElementAccumulator_,    
    /// Element type
    typename Element_,
    /// Layout of operand in memory
    typename Layout_,
    /// Shape of one matrix product operation (concept: MatrixShape)
    typename InstructionShape_,
    /// Whether beta is zero
    bool IsBetaZero_ >
class MmaTensorOpPureFragmentIterator;


// Partial specialization for col-major accumulator tile
// And Element type is the same as Accumulator Element type
```
**EN**: Declares the primary template with eight parameters. Key design choices: `KBlocksColumn_` encodes how many accumulator columns form one K-block (needed for residual computation); `IsBetaZero_` gates whether previous partial sums must be loaded and added (currently only `true` is specialised, meaning beta=0 path); `ElementAccumulator_` and `Element_` may differ to support type-narrowing from e.g. `float` accumulator to `half` output.

**CN**: 声明带有八个参数的主模板。关键设计要点：`KBlocksColumn_` 编码一个 K 块包含多少列累加器（用于余数计算）；`IsBetaZero_` 控制是否需要加载并累加先前的部分和（目前只特化了 `true` 即 beta=0 路径）；`ElementAccumulator_` 与 `Element_` 可以不同，以支持从 `float` 累加器到 `half` 输出等类型收窄转换。

---

### Lines 71–83 — Col-Major Specialisation Template Header / 列主序特化模板头

```cpp

template <
    /// Shape of warp tile to load (concept: MatrixShape)
    typename Shape_,
    /// Shape of the warp accumulation tile (concept: MatrixShape)
    typename AccumulatorShape_,
    /// KBlocks columns to compute residual
    int KBlocksColumn_,    
    /// Element type
    typename Element_,
    /// Shape of one matrix product operation (concept: MatrixShape)
    typename InstructionShape_>
class MmaTensorOpPureFragmentIterator<Shape_, AccumulatorShape_, KBlocksColumn_, Element_, Element_,
```
**EN**: Specialises for `Layout = ColumnMajor`, `ElementAccumulator = Element` (same type), and `IsBetaZero = true`. Because accumulator and output share the same element type, no numeric conversion is needed in `load`. The `ColumnMajor` path uses a different loop order in `load` (m-outer, n-inner) vs. the `RowMajor` path.

**CN**: 针对 `Layout = ColumnMajor`、`ElementAccumulator = Element`（相同类型）以及 `IsBetaZero = true` 进行特化。由于累加器与输出共享同一元素类型，`load` 中无需进行数值类型转换。与行主序路径相比，列主序路径在 `load` 中使用不同的循环顺序（m 为外层，n 为内层）。

---

### Lines 84–107 — Col-Major Public Type Aliases / 列主序公有类型别名

```cpp
                                         cutlass::layout::ColumnMajor,
                                         InstructionShape_, true> {
 public:

  /// Shape of warp tile to load (concept: MatrixShape)
  using Shape = Shape_;
    
  /// Shape of the warp accumulation tile (concept: MatrixShape)
  using AccumulatorShape = AccumulatorShape_;

  /// KBlocks columns to compute residual
  static int const kKBlockColumn = KBlocksColumn_;

  /// Element type
  using Element = Element_;

  /// Layout of source tile
  using Layout = cutlass::layout::ColumnMajor;

  /// Shape of one matrix product operation (concept: MatrixShape)
  using InstructionShape = InstructionShape_;

  /// Whether beta is zero
  static bool const IsBetaZero = true;
```
**EN**: Re-exposes template parameters as clean public names and fixes two integer constants: `kThreads = 32` (a warp is always 32 threads in CUDA) and `IsBetaZero = true`. These aliases make downstream code readable without template noise.

**CN**: 将模板参数重新以简洁的公有名称暴露，并固定两个整型常量：`kThreads = 32`（CUDA 中一个 warp 始终为 32 个线程）和 `IsBetaZero = true`。这些别名使下游代码在不受模板噪声干扰的情况下保持可读性。

---

### Lines 108–131 — Col-Major `Policy` Nested Struct / 列主序 `Policy` 嵌套结构体

```cpp

  /// Number of participating threads
  static int const kThreads = 32;

  /// Internal structure of iterator - made public to enable introspection
  struct Policy {
    static_assert(
        !(Shape::kRow % InstructionShape::kM) &&
            !(Shape::kColumn % InstructionShape::kN),
        "Shape of warp-level Mma must be divisible by operator shape.");
    static_assert(
        !(AccumulatorShape::kRow % Shape::kRow) &&
            !(AccumulatorShape::kColumn % Shape::kColumn),
        "Shape of Warp Accumulator must be divisible by warp shape.");
    static_assert(
        !(kKBlockColumn % Shape::kColumn),
        "KBlock size must be divisible by warp shape.");

    /// Number of times this iterator can be incremented
    static int const kIterations = AccumulatorShape::kCount / Shape::kCount;
  };

private:
```
**EN**: The `Policy` struct enforces three compile-time divisibility constraints and derives `kIterations` (total number of warp-shape tiles that partition the full accumulator). The `static_assert`s catch mis-matched shapes at compile time, avoiding silent silent incorrect behaviour at runtime.

**CN**: `Policy` 结构体强制执行三项编译期整除约束，并推导出 `kIterations`（划分完整累加器所需的 warp 形状 tile 总数）。`static_assert` 在编译期捕获形状不匹配的问题，避免在运行时出现静默的错误行为。

---

### Lines 132–155 — Col-Major Private Derived Constants / 列主序私有推导常量

```cpp
  static int const kElementsPerAccess = InstructionShape::kM * InstructionShape::kN / kThreads;

  /// Number of mma operations performed by a warp
  using MmaIterations = MatrixShape<Shape::kRow / InstructionShape::kM,
                                    Shape::kColumn / InstructionShape::kN>;
  /// Number of mma operations performed by the entire accumulator
  using AccumulatorIterations = MatrixShape<AccumulatorShape::kRow / InstructionShape::kM,
                                              AccumulatorShape::kColumn / InstructionShape::kN>;

  /// Number of K iterations    
  static int const kKBlockIterations = (AccumulatorShape::kColumn + kKBlockColumn - 1) / kKBlockColumn;
  static int const kResidualColumn = AccumulatorShape::kColumn - (kKBlockIterations - 1) * kKBlockColumn;
  static int const kKBlockColumnIterations = kKBlockColumn / Shape::kColumn 
                                     * (AccumulatorShape::kRow / Shape::kRow);
  static int const kResidualIndex = kResidualColumn / Shape::kColumn
                                     * (AccumulatorShape::kRow / Shape::kRow);

public:

  //
  // Derived quantities
  //

  /// Fragment object holding a thread's part of a tile
```
**EN**: `kElementsPerAccess` is the number of C-matrix elements a single instruction produces per thread. `MmaIterations` and `AccumulatorIterations` are 2-D counts used to convert a flat `index_` into `(index_m, index_n)` coordinates. The `kKBlock*` constants implement the residual logic: the last K-block may have fewer columns than the others, so `kResidualColumn` captures the shorter width and `kResidualIndex` marks the first fragment index that falls in that residual region.

**CN**: `kElementsPerAccess` 是单条指令每线程产生的 C 矩阵元素数量。`MmaIterations` 和 `AccumulatorIterations` 是二维计数，用于将平坦的 `index_` 转换为 `(index_m, index_n)` 坐标。`kKBlock*` 系列常量实现余数逻辑：最后一个 K 块的列数可能少于其他块，`kResidualColumn` 记录较短的宽度，`kResidualIndex` 标记落入余数区域的第一个片段索引。

---

### Lines 156–173 — Col-Major Public Fragment Types / 列主序公有片段类型

```cpp
  /// This is the fragment size produced by one access of the iterator.
  using Fragment = Array<Element, Shape::kCount / kThreads>;

  /// Accumulator Fragment object
  using AccumulatorFragment = Array<Element, AccumulatorShape::kCount / kThreads>;


private:

  /// Internal access type
  using AccessType = Array<Element, kElementsPerAccess>;

private:
  //
  // Data members
  //

  /// Accumulator tile
```
**EN**: `Fragment` is what a single `load` call produces — the thread's share of one `Shape`-sized warp tile. `AccumulatorFragment` is the complete per-thread accumulator (passed to the constructor). `AccessType` is the internal granularity matching one instruction's output per thread.

**CN**: `Fragment` 是单次 `load` 调用的输出——一个线程在一个 `Shape` 大小 warp tile 中持有的数据份额。`AccumulatorFragment` 是完整的每线程累加器（传入构造函数）。`AccessType` 是内部粒度，对应一条指令每线程的输出。

---

### Lines 174–187 — Col-Major Data Members & Constructor / 列主序数据成员与构造函数

```cpp
  AccessType const *accumulators_;

  /// Internal index
  int index_;

  /// Used to access residual tile first
  bool is_residual_tile_;

public:
  /// Constructs an iterator
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator(AccumulatorFragment const &accum)
      : accumulators_(reinterpret_cast<AccessType const *>(&accum)),
        index_(0), is_residual_tile_(true) {}
```
**EN**: Three data members: the reinterpreted pointer into the accumulator, the current linear index, and `is_residual_tile_` which starts `true` so that the first K-block is treated as potentially residual. This avoids requiring callers to special-case the first iteration.

**CN**: 三个数据成员：重解释后指向累加器的指针、当前线性索引，以及初始值为 `true` 的 `is_residual_tile_`，使第一个 K 块被视为潜在余数块。这样调用者无需对第一次迭代进行特殊处理。

---

### Lines 188–209 — Col-Major `add_offset` and Increment/Decrement / 列主序 `add_offset` 与自增/自减

```cpp

  /// Add offset
  CUTLASS_HOST_DEVICE
  void add_offset(int index_offset) {
    index_ += index_offset; 
    if(is_residual_tile_ && index_ >= kKBlockColumnIterations) {
      index_ = index_ - kKBlockColumnIterations + kResidualIndex;
      is_residual_tile_ = false;
    }
  }

  /// Increments
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator &operator++() {
    add_offset(1);
    return *this;
  }

  /// Decrements
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator &operator--() {
    add_offset(-1);
```
**EN**: `add_offset` is the single location where residual-tile remapping happens: once `index_` exceeds `kKBlockColumnIterations` (the number of full-K-block tiles), it is remapped to the residual region and `is_residual_tile_` is cleared. `operator++` and `operator--` delegate to `add_offset(±1)`.

**CN**: `add_offset` 是唯一进行余数 tile 重映射的地方：一旦 `index_` 超过 `kKBlockColumnIterations`（满 K 块的 tile 数量），就将其重映射到余数区域并清除 `is_residual_tile_`。`operator++` 和 `operator--` 委托给 `add_offset(±1)`。

---

### Lines 210–247 — Col-Major `load` Method / 列主序 `load` 方法

```cpp
    return *this;
  }

  /// Loads a fragment from the referenced part of the accumulator tile
  CUTLASS_HOST_DEVICE
  void load(Fragment &frag) const {

    AccessType src_fragment;
    src_fragment.clear();


    AccessType *frag_ptr = reinterpret_cast<AccessType *>(&frag);

    int index_m = (index_ * MmaIterations::kRow) % AccumulatorIterations::kRow;
    int index_n = (index_ * MmaIterations::kRow) / AccumulatorIterations::kRow 
                    * MmaIterations::kColumn;

    CUTLASS_PRAGMA_UNROLL
    for (int n = 0; n < MmaIterations::kColumn; n++) {
      for (int m = 0; m < MmaIterations::kRow; m++) {
        int accumulator_access_offset = 
            (n + index_n) * AccumulatorIterations::kRow + m + index_m;
            
        frag_ptr[n * MmaIterations::kRow + m].clear();
        if(!(is_residual_tile_ && index_ >= kResidualIndex))
            frag_ptr[n * MmaIterations::kRow + m] = accumulators_[accumulator_access_offset];
            // frag_ptr[n * MmaIterations::kRow + m] = output_op(accumulators_[accumulator_access_offset], src_fragment);
      }
    }
  }

};

// Partial specialization for row-major accumulator tile

template <
    /// Shape of warp tile to load (concept: MatrixShape)
    typename Shape_,
```
**EN**: Converts the flat `index_` into 2-D `(index_m, index_n)` coordinates using modular arithmetic. For column-major layout the loop is `n`-outer / `m`-inner, matching the column-major memory stride. The residual guard `!(is_residual_tile_ && index_ >= kResidualIndex)` zeros the fragment for out-of-range column positions, ensuring correctness at the K-dimension boundary.

**CN**: 使用模运算将平坦的 `index_` 转换为二维 `(index_m, index_n)` 坐标。对于列主序布局，循环采用 `n` 为外层、`m` 为内层的顺序，与列主序内存步幅相匹配。余数守卫 `!(is_residual_tile_ && index_ >= kResidualIndex)` 对越界列位置的片段执行清零，确保在 K 维度边界处的正确性。

---

### Lines 248–271 — Row-Major Specialisation Template Header / 行主序特化模板头

```cpp
    /// Shape of the warp accumulation tile (concept: MatrixShape)
    typename AccumulatorShape_,
    /// KBlocks columns to compute residual
    int KBlocksColumn_,    
    /// Accumulator Element type
    typename ElementAccumulator_,    
    /// Element type
    typename Element_,
    /// Shape of one matrix product operation (concept: MatrixShape)
    typename InstructionShape_>
class MmaTensorOpPureFragmentIterator<Shape_, AccumulatorShape_, KBlocksColumn_, ElementAccumulator_, Element_,
                                         cutlass::layout::RowMajor,
                                         InstructionShape_, true> {
 public:

  /// Shape of warp tile to load (concept: MatrixShape)
  using Shape = Shape_;
    
  /// Shape of the warp accumulation tile (concept: MatrixShape)
  using AccumulatorShape = AccumulatorShape_;

  /// KBlocks columns to compute residual
  static int const kKBlockColumn = KBlocksColumn_;
```
**EN**: The row-major specialisation differs from the column-major one in two ways: (1) `ElementAccumulator_` and `Element_` are separate parameters (allowing accumulation in `float` while outputting `half`), and (2) the loop order inside `load` is reversed (`m`-outer / `n`-inner) to follow row-major strides.

**CN**: 行主序特化与列主序特化有两点不同：（1）`ElementAccumulator_` 和 `Element_` 是独立参数（允许以 `float` 累加同时以 `half` 输出）；（2）`load` 内部的循环顺序相反（`m` 为外层，`n` 为内层），以遵循行主序步幅。

---

### Lines 272–300 — Row-Major Public Type Aliases / 行主序公有类型别名

```cpp
  /// Accumulator Element type
  using ElementAccumulator = ElementAccumulator_;

  /// Element type
  using Element = Element_;

  /// Layout of source tile
  using Layout = cutlass::layout::RowMajor;

  /// Shape of one matrix product operation (concept: MatrixShape)
  using InstructionShape = InstructionShape_;

  /// Whether beta is zero
  static bool const IsBetaZero = true;

  /// Number of participating threads
  static int const kThreads = 32;

  /// Internal structure of iterator - made public to enable introspection
  struct Policy {
    static_assert(
        !(Shape::kRow % InstructionShape::kM) &&
            !(Shape::kColumn % InstructionShape::kN),
        "Shape of warp-level Mma must be divisible by operator shape.");
    static_assert(
        !(AccumulatorShape::kRow % Shape::kRow) &&
            !(AccumulatorShape::kColumn % Shape::kColumn),
        "Shape of Warp Accumulator must be divisible by warp shape.");
    static_assert(
```
**EN**: Same pattern as the column-major version, but with the additional `ElementAccumulator` alias to distinguish the accumulation type from the fragment output type. This separation enables implicit numeric narrowing in the `load` loop.

**CN**: 与列主序版本相同的模式，但增加了 `ElementAccumulator` 别名，用于区分累加类型与片段输出类型。这种分离使 `load` 循环中能够进行隐式数值收窄转换。

---

### Lines 301–336 — Row-Major Policy and Private Constants / 行主序策略与私有常量

```cpp
        !(kKBlockColumn % Shape::kColumn),
        "KBlock size must be divisible by warp shape.");

    /// Number of times this iterator can be incremented
    static int const kIterations = AccumulatorShape::kCount / Shape::kCount;
  };

private:

  static int const kElementsPerAccess = InstructionShape::kM * InstructionShape::kN / kThreads;

  /// Number of mma operations performed by a warp
  using MmaIterations = MatrixShape<Shape::kRow / InstructionShape::kM,
                                    Shape::kColumn / InstructionShape::kN>;
  /// Number of mma operations performed by the entire accumulator
  using AccumulatorIterations = MatrixShape<AccumulatorShape::kRow / InstructionShape::kM,
                                              AccumulatorShape::kColumn / InstructionShape::kN>;

  /// Number of K iterations    
  static int const kKBlockIterations = (AccumulatorShape::kColumn + kKBlockColumn - 1) / kKBlockColumn;
  static int const kResidualColumn = AccumulatorShape::kColumn - (kKBlockIterations - 1) * kKBlockColumn;
  static int const kKBlockColumnIterations = kKBlockColumn / Shape::kColumn 
                                     * (AccumulatorShape::kRow / Shape::kRow);
  static int const kResidualIndex = kResidualColumn / Shape::kColumn
                                     * (AccumulatorShape::kRow / Shape::kRow);

public:

  //
  // Derived quantities
  //

  /// Fragment object holding a thread's part of a tile
  /// This is the fragment size produced by one access of the iterator.
  using Fragment = Array<Element, Shape::kCount / kThreads>;
```
**EN**: Identical structure to the column-major counterpart. The same three `static_assert`s enforce divisibility; the same K-residual constants are derived. The only difference is that row-major layout changes how `index_m` and `index_n` are decoded in the `load` loop.

**CN**: 与列主序版本结构完全相同。三项 `static_assert` 同样强制整除约束；K 余数常量以相同方式推导。唯一的区别在于行主序布局改变了 `load` 循环中 `index_m` 和 `index_n` 的解码方式。

---

### Lines 337–359 — Row-Major Public Fragment Types / 行主序公有片段类型

```cpp
  /// Accumulator Fragment object
  using AccumulatorFragment = Array<ElementAccumulator, AccumulatorShape::kCount / kThreads>;


private:

  /// Internal access type
  using AccessType = Array<ElementAccumulator, kElementsPerAccess>;
  using FragmentAccessType = Array<Element, kElementsPerAccess>;

private:
  //
  // Data members
  //

  /// Accumulator tile
  AccessType const *accumulators_;

  /// Internal index
  int index_;

  /// Used to access residual tile first
  bool is_residual_tile_;
```
**EN**: The row-major version introduces `FragmentAccessType` (typed to `Element` rather than `ElementAccumulator`) alongside `AccessType`. This permits an element-wise type cast from the accumulator's wider type to the fragment's narrower type during the `load` operation.

**CN**: 行主序版本在 `AccessType` 之外引入了 `FragmentAccessType`（以 `Element` 而非 `ElementAccumulator` 为类型参数）。这允许在 `load` 操作中将累加器的较宽类型逐元素转换为片段的较窄类型。

---

### Lines 360–381 — Row-Major Data Members, Constructor, `add_offset` / 行主序数据成员、构造函数与 `add_offset`

```cpp

public:
  /// Constructs an iterator
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator(AccumulatorFragment const &accum)
      : accumulators_(reinterpret_cast<AccessType const *>(&accum)),
        index_(0), is_residual_tile_(true) {}

  /// Add offset
  CUTLASS_HOST_DEVICE
  void add_offset(int index_offset) {
    index_ += index_offset; 
    if(is_residual_tile_ && index_ >= kKBlockColumnIterations) {
      index_ = index_ - kKBlockColumnIterations + kResidualIndex;
      is_residual_tile_ = false;
    }
  }

  /// Increments
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator &operator++() {
    add_offset(1);
```
**EN**: Identical semantics to the column-major version. `accumulators_` stores the accumulator as `AccessType*` (typed to `ElementAccumulator`). `add_offset` contains the same K-residual remapping logic.

**CN**: 语义与列主序版本完全相同。`accumulators_` 将累加器存储为 `AccessType*`（以 `ElementAccumulator` 为类型参数）。`add_offset` 包含相同的 K 余数重映射逻辑。

---

### Lines 382–427 — Row-Major `load` Method and Namespace Close / 行主序 `load` 方法与命名空间关闭

```cpp
    return *this;
  }

  /// Decrements
  CUTLASS_HOST_DEVICE
  MmaTensorOpPureFragmentIterator &operator--() {
    add_offset(-1);
    return *this;
  }

  /// Loads a fragment from the referenced part of the accumulator tile
  CUTLASS_HOST_DEVICE
  void load(Fragment &frag) const {


    FragmentAccessType src_fragment;
    src_fragment.clear();

    FragmentAccessType *frag_ptr = reinterpret_cast<FragmentAccessType *>(&frag);

    int index_m = (index_ * MmaIterations::kRow) % AccumulatorIterations::kRow;
    int index_n = (index_ * MmaIterations::kRow) / AccumulatorIterations::kRow 
                    * MmaIterations::kColumn;

    CUTLASS_PRAGMA_UNROLL
    for (int m = 0; m < MmaIterations::kRow; m++) {
      for (int n = 0; n < MmaIterations::kColumn; n++) {
        int accumulator_access_offset = 
            (m + index_m) * AccumulatorIterations::kColumn + n + index_n;

        frag_ptr[m * MmaIterations::kColumn + n].clear();
        if(!(is_residual_tile_ && index_ >= kResidualIndex))
           frag_ptr[m * MmaIterations::kColumn + n] = (accumulators_[accumulator_access_offset]);
      }
    }
  }

};

////////////////////////////////////////////////////////////////////////////////

} // namespace warp
} // namespace gemm
} // namespace cutlass

////////////////////////////////////////////////////////////////////////////////
```
**EN**: The row-major `load` reverses the loop order (`m`-outer / `n`-inner) and uses `AccumulatorIterations::kColumn` as the stride (row-major stride). `frag_ptr` is typed to `FragmentAccessType` (`Element`) while `accumulators_` is typed to `AccessType` (`ElementAccumulator`); the implicit conversion on assignment narrows the type. The closing braces end the class and the three namespaces.

**CN**: 行主序 `load` 颠倒循环顺序（`m` 为外层，`n` 为内层），并以 `AccumulatorIterations::kColumn` 作为步幅（行主序步幅）。`frag_ptr` 的类型为 `FragmentAccessType`（`Element`），而 `accumulators_` 的类型为 `AccessType`（`ElementAccumulator`）；赋值时的隐式转换完成类型收窄。最后的闭合括号结束类定义和三层命名空间。

---
## Key Concepts / 关键概念
- Pure fragment iterators read accumulator tiles without applying beta scaling or element conversion. / 纯片段迭代器读取累加器 tile 时不会应用 beta 缩放或元素类型转换。
- Partial specializations adapt iterator math to row-major and column-major accumulator layouts. / 偏特化让迭代器数学规则适配行主序与列主序累加器布局。
- Residual-tile handling prevents undefined reads when the final K block is smaller than the steady-state tile. / 余数 tile 处理可避免最后一个 K 块小于稳态 tile 时出现未定义读取。
## Dependencies / 依赖项
- `cutlass/matrix_shape.h` — Compile-time shapes for MMA instruction tiles and fragments / MMA 指令 tile 与片段使用的编译期形状
- `cutlass/layout/tensor.h` — Tensor layouts used by the row-major specialization / 行主序特化使用的张量布局
- `cutlass/numeric_conversion.h` — Numeric conversion helpers used when exposing fragments / 暴露片段时使用的数值转换辅助工具
