# permute_info.h — Code Analysis / 代码分析
**Source / 源文件**: `examples/39_gemm_permute/permute_info.h`
**Purpose / 用途**: Provides compile-time metadata that links CUTLASS permute layouts to tensor shapes and coordinate transforms. / 提供编译期元数据，将 CUTLASS 置换布局与张量形状和坐标变换关联起来。
---
## Line-by-Line Analysis / 逐行分析

### Lines 1–41 — License, Includes & Primary Template

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
    \brief Contains additional metadata about layout permute functions used in the example.
*/

#include "cutlass/tensor_coord.h"
#include "cutlass/layout/permute.h"

/// Additional permutation metadata to facilitate testing/printing 
template<typename PermuteLayout>
struct PermuteInfo;
```
**EN**: The two includes bring in CUTLASS coordinate types (`MatrixCoord`, `Tensor4DCoord`, `Tensor5DCoord`) and all permute layout declarations. The primary template `PermuteInfo` is intentionally left **incomplete** — instantiating it with an unsupported layout type will produce a compile-time error, acting as a static assertion that every used permute layout must have a registered specialization.

**CN**: 两个头文件引入了 CUTLASS 坐标类型（`MatrixCoord`、`Tensor4DCoord`、`Tensor5DCoord`）以及所有置换布局声明。主模板 `PermuteInfo` 有意保持**不完整**——使用不支持的布局类型实例化它将产生编译期错误，相当于静态断言，要求每个使用的置换布局必须有已注册的特化版本。

---

### Lines 42–83 — `PermuteInfo<NoPermute>` — Identity Case

```cpp
/// Specialization for default case (no permute). Other specializations must follow this template.
template<>
struct PermuteInfo<cutlass::layout::NoPermute> {

  /// Whether this is a BMM or GEMM permutation (NoPermute can actually be either)
  static bool constexpr kBatched = false;

  /// Minimal divisor for row extent
  static int  constexpr kRowFactor = 1;

  /// Minimum divisor for column extent
  static int  constexpr kColumnFactor = 1;

  /// Minimum divisor for batch size dimension
  static int  constexpr kBatchFactor = 1;

  /// Tensor layout used in permutation operation
  using Layout = cutlass::layout::PackedVectorLayout;

  static std::string name() {
    return "NoPermute";
  }

  /// User-friendly description of the permute operation
  static std::string desc() {
    return "no permutation";
  }

  /// Infer original higher-rank tensor shape from GEMM/BMM matrix extents.
  /// For direct (output) permutations, must be a simple reshape of extent.
  /// For inverse (input) permutations, must return shape *before* permute operation.
  /// In case of NoPermute, simply use a linear (rank 1) view of the memory
  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    return Layout::TensorCoord(extent.row() * extent.column() * batch_count);
  }

  /// Compute the permuted higher-rank tensor shape from the original shape.
  static Layout::TensorCoord permute(Layout::TensorCoord const &s) {
    return s;
  }
};
```
**EN**: The identity specialization. All factor constants are `1` meaning no divisibility constraints are imposed on the GEMM dimensions. `Layout` is `PackedVectorLayout` — CUTLASS's rank-1 flat layout — so `original_shape` returns the total element count as a 1-D coordinate. `permute()` is the identity function. This case is used when a tensor participates in the GEMM but requires no reordering of its elements in global memory.

**CN**: 恒等特化版本。所有因子常量均为 `1`，意味着对 GEMM 维度不施加任何整除约束。`Layout` 为 `PackedVectorLayout`——CUTLASS 的秩-1 平坦布局——因此 `original_shape` 将总元素数作为 1-D 坐标返回。`permute()` 是恒等函数。当张量参与 GEMM 但不需要对全局内存中的元素进行重排序时，使用此情况。

---

### Lines 84–113 — `PermuteInfo<Tensor4DPermuteBMM0213RowMajor<D1>>` — BMM [0,2,1,3] Row

```cpp
template<int D1>
struct PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>> {

  static bool constexpr kBatched = true;
  static int  constexpr kRowFactor = 1;
  static int  constexpr kColumnFactor = 1;
  static int  constexpr kBatchFactor = D1;

  using Layout = cutlass::layout::TensorNHWC;

  static std::string name() {
    return "Tensor4DPermuteBMM0213<" + std::to_string(D1) + ">";
  }

  static std::string desc() {
    return "batched GEMM permutation [0, 2, 1, 3]";
  }

  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = batch_count / D1;
    int D2 = extent.row();
    int D3 = extent.column();
    return {D0, D1, D2, D3};
  }

  static Layout::TensorCoord permute(Layout::TensorCoord const &s) {
    return {s[0], s[2], s[1], s[3]};
  }
};
```
**EN**: This BMM permute treats the batched output tensor of shape `[B, M, N]` as a 4-D tensor `[D0, D1, M, N]` (where `D0 = B / D1`) and applies permutation `[0, 2, 1, 3]` → `[D0, M, D1, N]`. Key design points:
- `kBatched = true`: signals that the batch dimension participates in the permutation.
- `kBatchFactor = D1`: the batch count must be divisible by the compile-time constant `D1`.
- `Layout = TensorNHWC`: uses NHWC as the reference layout for coordinate arithmetic, interpreting the 4 logical dims as `(N=D0, H=D1, W=M, C=N_gemm)`.
- `permute()` swaps indices 1 and 2, i.e., the `D1` and `M` dimensions.

**CN**: 该 BMM 置换将形状为 `[B, M, N]` 的批处理输出张量视为 4-D 张量 `[D0, D1, M, N]`（其中 `D0 = B / D1`），并应用置换 `[0, 2, 1, 3]` → `[D0, M, D1, N]`。关键设计点：
- `kBatched = true`：表示批次维度参与置换。
- `kBatchFactor = D1`：批次数必须能被编译期常量 `D1` 整除。
- `Layout = TensorNHWC`：使用 NHWC 作为坐标运算的参考布局，将 4 个逻辑维度解释为 `(N=D0, H=D1, W=M, C=N_gemm)`。
- `permute()` 交换索引 1 和 2，即 `D1` 和 `M` 维度。

---

### Lines 114–133 — `PermuteInfo<Tensor4DPermuteBMM0213RowMajorInverse<D1>>`

```cpp
template<int D1>
struct PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0213RowMajorInverse<D1>> 
: public PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>> {

  static bool constexpr kBatched = true;
  static int  constexpr kRowFactor = 1;
  static int  constexpr kColumnFactor = D1;
  static int  constexpr kBatchFactor = 1;

  using Base = PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0213RowMajor<D1>>;
  using Layout = typename Base::Layout;

  static typename Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = batch_count;
    int D2 = extent.row();
    int D3 = extent.column() / D1;
    return {D0, D1, D2, D3};
  }
};
```
**EN**: The **inverse** specialization inherits most fields from the forward permute but overrides the shape reconstruction and factor constraints. For an input tensor that has *already been permuted* (i.e., reading from a permuted source), `kColumnFactor = D1` requires the column extent to be divisible by `D1` and `kBatchFactor = 1` lifts the batch constraint. `original_shape` divides the column extent by `D1` to recover the pre-permute shape. Inheriting `permute()` and `Layout` from the base avoids code duplication.

**CN**: **逆置换**特化版本从正向置换继承大多数字段，但重写了形状重建和因子约束。对于*已经被置换*的输入张量（即从已置换的数据源读取），`kColumnFactor = D1` 要求列范围能被 `D1` 整除，而 `kBatchFactor = 1` 取消了批次约束。`original_shape` 将列范围除以 `D1` 以恢复置换前的形状。从基类继承 `permute()` 和 `Layout` 避免了代码重复。

---

### Lines 134–183 — `PermuteInfo<Tensor4DPermuteBMM0321ColumnMajor<D1>>` and its Inverse

```cpp
template<int D1>
struct PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>> {
  
  static bool constexpr kBatched = true;
  static int  constexpr kRowFactor = 1;
  static int  constexpr kColumnFactor = 1;
  static int  constexpr kBatchFactor = D1;

  using Layout = cutlass::layout::TensorNHCW;

  static std::string name() {
    return "Tensor4DPermuteBMM0321<" + std::to_string(D1) + ">";
  }

  static std::string desc() {
    return "batched GEMM permutation [0, 3, 2, 1]";
  }

  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = batch_count / D1;
    int D2 = extent.row();
    int D3 = extent.column();
    return {D0, D1, D2, D3};
  }

  static Layout::TensorCoord permute(Layout::TensorCoord const &s) {
    return {s[0], s[3], s[2], s[1]};
  }
};

template<int D1>
struct PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajorInverse<D1>> 
: public PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>> {
  
  static bool constexpr kBatched = true;
  static int  constexpr kRowFactor = D1;
  static int  constexpr kColumnFactor = 1;
  static int  constexpr kBatchFactor = 1;

  using Base = PermuteInfo<cutlass::layout::Tensor4DPermuteBMM0321ColumnMajor<D1>>;
  using Layout = typename Base::Layout;

  static typename Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = batch_count;
    int D2 = extent.row() / D1;
    int D3 = extent.column();
    return {D0, D1, D2, D3};
  }
};
```
**EN**: The column-major BMM variant permutes `[D0, D1, M, N]` → `[D0, N, M, D1]` via index order `[0, 3, 2, 1]`. Because of column-major storage, the dimension that maps to the contiguous axis changes compared to the row-major case. The reference `Layout = TensorNHCW` (N→H→C→W memory order) matches the column-major access pattern. The inverse specialization overrides `kRowFactor = D1` (the row extent must be divisible by `D1`) and recovers `D2 = row / D1` in `original_shape`.

**CN**: 列主序 BMM 变体通过索引顺序 `[0, 3, 2, 1]` 将 `[D0, D1, M, N]` 置换为 `[D0, N, M, D1]`。由于列主序存储，映射到连续轴的维度与行主序情况不同。参考布局 `Layout = TensorNHCW`（N→H→C→W 内存顺序）与列主序访问模式匹配。逆置换特化重写 `kRowFactor = D1`（行范围必须能被 `D1` 整除），并在 `original_shape` 中通过 `D2 = row / D1` 恢复维度值。

---

### Lines 184–243 — `PermuteInfo<Tensor4DPermute0213RowMajor<D1,D2>>` Family (Normal GEMM 4-D)

```cpp
template<int D1, int D2>
struct PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajor<D1, D2>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = D1;
  static int  constexpr kColumnFactor = D2;
  static int  constexpr kBatchFactor = 1;

  using Layout = cutlass::layout::TensorNHWC;

  static std::string name() {
    return "Tensor4DPermute0213<" + std::to_string(D1) + "," + std::to_string(D2) + ">";
  }

  static std::string desc() {
    return "normal GEMM permutation [0, 2, 1, 3]";
  }

  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = extent.row() / D1;
    int D3 = extent.column() / D2;
    return {D0, D1, D2, D3};
  }

  static Layout::TensorCoord permute(Layout::TensorCoord const &s) {
    return {s[0], s[2], s[1], s[3]};
  }
};

template<int D1, int D2>
struct PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajorInverse<D1, D2>>
: public PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajor<D1, D2>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = D2;
  static int  constexpr kColumnFactor = D1;
  static int  constexpr kBatchFactor = 1;

  using Base = PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajor<D1, D2>>;
  using Layout = typename Base::Layout;

  static typename Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int D0 = extent.row() / D2;
    int D3 = extent.column() / D1;
    return {D0, D1, D2, D3};
  }
};

template<int D1, int D2>
struct PermuteInfo<cutlass::layout::Tensor4DPermute0213ColumnMajor<D1, D2>>
: public PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajor<D1, D2>> {
  using Layout = cutlass::layout::TensorCWHN;
};

template<int D1, int D2>
struct PermuteInfo<cutlass::layout::Tensor4DPermute0213ColumnMajorInverse<D1, D2>>
: public PermuteInfo<cutlass::layout::Tensor4DPermute0213RowMajorInverse<D1, D2>> {
  using Layout = cutlass::layout::TensorCWHN;
};
```
**EN**: For a **normal (non-batched) GEMM**, the 4-D permute reshapes the `[M, N]` output matrix as `[M/D1, D1, D2, N/D2]` then applies `[0, 2, 1, 3]` → `[M/D1, D2, D1, N/D2]`. Two template parameters `D1` and `D2` give the tile sizes along row and column dimensions respectively. The constraints `kRowFactor = D1` and `kColumnFactor = D2` ensure M and N are divisible by these tile sizes.

The column-major specializations simply inherit all logic and swap `Layout` to `TensorCWHN`. The inverse specializations swap the roles of `D1`/`D2` in `kRowFactor`/`kColumnFactor` and adjust `original_shape` accordingly: where the forward permute reads `[M/D1, D1, ...]` the inverse sees `[M/D2, D1, ...]` because after permuting the GEMM matrix dimensions are swapped.

**CN**: 对于**普通（非批处理）GEMM**，4-D 置换将 `[M, N]` 输出矩阵重塑为 `[M/D1, D1, D2, N/D2]`，然后应用 `[0, 2, 1, 3]` → `[M/D1, D2, D1, N/D2]`。两个模板参数 `D1` 和 `D2` 分别指定沿行和列维度的分块大小。约束 `kRowFactor = D1` 和 `kColumnFactor = D2` 确保 M 和 N 能被这些分块大小整除。

列主序特化只需继承所有逻辑并将 `Layout` 替换为 `TensorCWHN`。逆置换特化交换 `kRowFactor`/`kColumnFactor` 中 `D1`/`D2` 的角色，并相应调整 `original_shape`：正向置换读取 `[M/D1, D1, ...]`，而逆置换看到 `[M/D2, D1, ...]`，因为置换后 GEMM 矩阵维度被交换了。

---

### Lines 244–293 — `PermuteInfo<Tensor5DPermute20314RowMajor<T1,T2,T3>>` and Inverse

```cpp
template<int T1, int T2, int T3>
struct PermuteInfo<cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = T1;
  static int  constexpr kColumnFactor = T2 * T3;
  static int  constexpr kBatchFactor = 1;

  using Layout = cutlass::layout::TensorNDHWC;

  static std::string name() {
    return "Tensor5DPermute20314<" + std::to_string(T1) + "," + std::to_string(T2) + "," + std::to_string(T3) + ">";
  }

  static std::string desc() {
    return "normal GEMM permutation [2, 0, 3, 1, 4]";
  }

  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count)
  {
    int const T0 = extent.row() / T1;
    int const T4 = extent.column() / (T2 * T3);
    return {T0, T1, T2, T3, T4};
  }

  static Layout::TensorCoord permute(Layout::TensorCoord const &s)
  {
    return {s[2], s[0], s[3], s[1], s[4]};
  }
};

template<int T1, int T2, int T3>
struct PermuteInfo<cutlass::layout::Tensor5DPermute20314RowMajorInverse<T1, T2, T3>>
: public PermuteInfo<cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = T2;
  static int  constexpr kColumnFactor = T1 * T3;
  static int  constexpr kBatchFactor = 1;

  using Base = PermuteInfo<cutlass::layout::Tensor5DPermute20314RowMajor<T1, T2, T3>>;
  using Layout = typename Base::Layout;

  static typename Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int const T0 = extent.row() / T2;
    int const T4 = extent.column() / (T1 * T3);
    return {T0, T1, T2, T3, T4};
  }
};
```
**EN**: A 5-D row-major permutation. The `[M, N]` matrix is reshaped as `[T0, T1, T2, T3, T4]` (where `T0 = M/T1`, `T4 = N/(T2×T3)`) and permuted by `[2, 0, 3, 1, 4]` → `[T2, T0, T3, T1, T4]`. Viewed as a 2-D matrix this becomes `[M×T2/T1, N×T1/T2]`.

`kColumnFactor = T2 * T3` ensures the column extent is divisible by the product of two compile-time tile dimensions. The reference layout is `TensorNDHWC` (5-D NDHWC memory order). The inverse specialization swaps `T1↔T2` in the row/column factors and adjusts `original_shape`: `T0 = row / T2`, `T4 = col / (T1 * T3)`.

**CN**: 5-D 行主序置换。`[M, N]` 矩阵被重塑为 `[T0, T1, T2, T3, T4]`（其中 `T0 = M/T1`，`T4 = N/(T2×T3)`），并通过 `[2, 0, 3, 1, 4]` → `[T2, T0, T3, T1, T4]` 置换。作为 2-D 矩阵查看时变为 `[M×T2/T1, N×T1/T2]`。

`kColumnFactor = T2 * T3` 确保列范围能被两个编译期分块维度之积整除。参考布局为 `TensorNDHWC`（5-D NDHWC 内存顺序）。逆置换特化在行/列因子中交换 `T1↔T2`，并调整 `original_shape`：`T0 = row / T2`，`T4 = col / (T1 * T3)`。

---

### Lines 294–344 — `PermuteInfo<Tensor5DPermute02413ColumnMajor<T1,T2,T3>>` and Inverse

```cpp
template<int T1, int T2, int T3>
struct PermuteInfo<cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = T1;
  static int  constexpr kColumnFactor = T2 * T3;
  static int  constexpr kBatchFactor = 1;

  using Layout = cutlass::layout::TensorCWHDN;

  static std::string name() {
    return "Tensor5DPermute02413<" + std::to_string(T1) + "," + std::to_string(T2) + "," + std::to_string(T3) + ">";
  }

  static std::string desc() {
    return "normal GEMM permutation [0, 2, 4, 1, 3]";
  }

  using Coord = cutlass::Tensor5DCoord;

  static Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count)
  {
    int const T0 = extent.row() / T1;
    int const T4 = extent.column() / (T2 * T3);
    return {T0, T1, T2, T3, T4};
  }

  static Layout::TensorCoord permute(Layout::TensorCoord const &s)
  {
    return {s[0], s[2], s[4], s[1], s[3]};
  }
};

template<int T1, int T2, int T3>
struct PermuteInfo<cutlass::layout::Tensor5DPermute02413ColumnMajorInverse<T1, T2, T3>>
: public PermuteInfo<cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>> {

  static bool constexpr kBatched = false;
  static int  constexpr kRowFactor = T2;
  static int  constexpr kColumnFactor = T1 * T3;
  static int  constexpr kBatchFactor = 1;

  using Base = PermuteInfo<cutlass::layout::Tensor5DPermute02413ColumnMajor<T1, T2, T3>>;
  using Layout = typename Base::Layout;

  static typename Layout::TensorCoord original_shape(cutlass::MatrixCoord extent, int batch_count) {
    int const T0 = extent.row() / T2;
    int const T4 = extent.column() / (T1 * T3);
    return {T0, T1, T2, T3, T4};
  }
};
```
**EN**: The column-major counterpart of the 5-D permute. The logical permutation `[0, 2, 4, 1, 3]` picks indices `{T0, T2, T4, T1, T3}` in the output. The critical difference from `Tensor5DPermute20314RowMajor` is that:
1. The permutation formula is different (`[0,2,4,1,3]` vs `[2,0,3,1,4]`).
2. The reference layout is `TensorCWHDN` (from `layouts.h` defined in this example) rather than `TensorNDHWC`, reflecting the column-major axis ordering.

The `using Coord = cutlass::Tensor5DCoord;` alias is unused in the struct body but left as documentation that the coordinates are 5-D. The inverse follows the same swap pattern as the row-major inverse.

**CN**: 5-D 置换的列主序对应版本。逻辑置换 `[0, 2, 4, 1, 3]` 在输出中选取索引 `{T0, T2, T4, T1, T3}`。与 `Tensor5DPermute20314RowMajor` 的关键区别：
1. 置换公式不同（`[0,2,4,1,3]` vs `[2,0,3,1,4]`）。
2. 参考布局使用本示例 `layouts.h` 中定义的 `TensorCWHDN`（而非 `TensorNDHWC`），反映列主序的轴排序方式。

`using Coord = cutlass::Tensor5DCoord;` 别名在结构体主体中未被使用，但作为文档保留，说明坐标是 5-D 的。逆置换遵循与行主序逆置换相同的交换模式。

---

---
## Key Concepts / 关键概念
- Template specialization converts permute tags into compile-time factors and layout bindings. / 模板特化把置换标签转换成编译期因子与布局绑定。
- Shape reconstruction bridges flattened GEMM coordinates back to original tensor dimensions. / 形状重建逻辑把展平后的 GEMM 坐标重新映射回原始张量维度。
- Coordinate permutation helpers mirror the data movement performed by CUTLASS permuted layouts. / 坐标置换辅助函数对应 CUTLASS 置换布局执行的数据重排。
## Dependencies / 依赖项
- `cutlass/layout/permute.h` — Permute layout tag types specialized by `PermuteInfo` / 被 `PermuteInfo` 特化的置换布局标签类型
- `cutlass/tensor_coord.h` — Coordinate containers used to rebuild tensor shapes / 用于重建张量形状的坐标容器
- `layouts.h` — Referenced layout classes returned by specialized metadata / 特化元数据返回的布局类
