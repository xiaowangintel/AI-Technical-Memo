# block_striped.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/block_striped.h`

- **EN:** Utilities for performing block-striped access (load, store, reduce) of trivially-copyable, statically-sized array types to global memory.

- **CN:** 该头文件属于 CUTLASS 的核心类型或工具定义。文件级摘要：Utilities for performing block-striped access (load, store, reduce) of trivially-copyable, statically-sized array types to global memory.

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

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
```

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 31-34

```cpp
/*! \file
    \brief Utilities for performing block-striped access (load, store, reduce) of trivially-copyable,
    statically-sized array types to global memory.
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 36-36

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 38-42

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/array.h"
#include "cutlass/wmma_array.h"
#include "cutlass/functional.h"
#include "cutlass/complex.h"
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, `cutlass/array.h`, `cutlass/wmma_array.h`, `cutlass/functional.h`, `cutlass/complex.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h`, `cutlass/array.h`, `cutlass/wmma_array.h`, `cutlass/functional.h`, `cutlass/complex.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 44-44

```cpp
namespace cutlass {
```

**EN:** This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 46-48

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
// AccessWidth
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 50-55

```cpp
/// Computes the maximal power-of-two that evenly divides the size of T, capped at Limit
template <
  typename T,
  int Limit>
struct AccessWidth
{
```

**EN:** The preceding comment documents this block. This block begins the definition of `AccessWidth`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `AccessWidth` 这个 `struct`，其成员会在后续代码中展开。

### Lines 56-166

```cpp
  // Inductive case
  template <
      int ObjectBytes,        /// Size of T in bytes
      int AlignBytes,         /// Template induction variable
      bool IsAligned  =       /// Whether ObjectBytes is an even multiple of AlignBytes
        ((AlignBytes <= Limit) &&  (ObjectBytes % AlignBytes == 0))>
  struct Detail
  {
      static const int value = Detail<ObjectBytes, AlignBytes * 2>::value;
  };

  // Base case (ObjectBytes is not an even multiple of AlignBytes)
  template <
      int ObjectBytes,        /// Size of T in bytes
      int AlignBytes>         /// Template induction variable
  struct Detail<ObjectBytes, AlignBytes, false>
  {
      static const int value = AlignBytes / 2;
  };

  /// The maximal power-of-two that evenly divides the size of T
  static const int value = Detail<
    (int) sizeof(T),
    1>::value;
};



/////////////////////////////////////////////////////////////////////////////////////////////////
// StripedAccessType
/////////////////////////////////////////////////////////////////////////////////////////////////

/// ReinterpretCast type for striping a trivially-copyable type in global memory
/// (Default specialization.  Striping granularity is type T.)
template <
    typename T,           /// Data type
    int TransferBytes =   /// Data access width (16 byte max for global memory access on current architectures)
      AccessWidth<T, 16>::value>
struct alignas(TransferBytes) StripedAccessType : public T
{};


/// ReinterpretCast type for striping a trivially-copyable type in global memory
/// (Specialization for cutlass::Array<T>.  Striping granularity is a multiple of T.)
template <
    typename T,           /// Array element type
    int N,                /// Number of elements in array
    bool RegisterSized,   /// T is register-sized
    int TransferBytes>    /// Data access width
struct StripedAccessType<
    Array<T, N, RegisterSized>,
    TransferBytes>
: public AlignedArray<
            T,                                                  // Element type of StripedAccessType
            __NV_STD_MAX(1, TransferBytes / (int) sizeof(T)),   // Number of elements T in StripedAccessType
            TransferBytes>                                      // Alignment of StripedAccessType
{};


#if defined(CUTLASS_ARCH_WMMA_ENABLED)

/// ReinterpretCast type for striping a trivially-copyable type in global memory
/// (Specialization for cutlass::WmmaFragmentArray<T>.  Striping granularity is a multiple of T.)
template<
    typename Use,
    int m,
    int n,
    int k,
    typename ElementT,
    typename Layout,
    int kFragments,
    int TransferBytes>
struct StripedAccessType<
    WmmaFragmentArray<nvcuda::wmma::fragment<Use, m, n, k, ElementT, Layout>, kFragments>,
    TransferBytes>
: public AlignedArray<
            ElementT,
            __NV_STD_MAX(1, TransferBytes / (int) sizeof(ElementT)),
            TransferBytes>
{};

#endif // if defined(CUTLASS_ARCH_WMMA_ENABLED)


/////////////////////////////////////////////////////////////////////////////////////////////////
// BlockStriped
/////////////////////////////////////////////////////////////////////////////////////////////////

/// Utility for performing block-striped access (load, store) of trivially-copyable,
/// statically-sized array types to global memory
template <
  int BlockThreads,
  typename ArrayT,
  typename AccessT = StripedAccessType<ArrayT> >
struct BlockStriped
{
  /// Number of striped accesses
  static const int kStripes = int(sizeof(ArrayT) / sizeof(AccessT));
  static_assert(kStripes > 0, "AccessT type must be smaller than or equal to ArrayT type");
  /// Load
  CUTLASS_DEVICE
  static void load(ArrayT &data, ArrayT *ptr, int thread_idx)
  {
    AccessT *access_input = reinterpret_cast<AccessT*>(ptr);
    AccessT *access_data = reinterpret_cast<AccessT*>(&data);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kStripes; ++i) {
      access_data[i] = access_input[(BlockThreads * i) + thread_idx];
    }
  }
```

**EN:** The preceding comment documents this block. This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 前面的注释说明了这个代码块。这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 169-183

```cpp
  /// Load & Add
  CUTLASS_DEVICE
  static void load_add(ArrayT &data, ArrayT *ptr, int thread_idx)
  {
    AccessT *access_input = reinterpret_cast<AccessT*>(ptr);
    AccessT *access_data = reinterpret_cast<AccessT*>(&data);

    plus<AccessT> add;

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kStripes; ++i)
    {
      access_data[i] = add(access_data[i], access_input[(BlockThreads * i) + thread_idx]);
    }
  }
```

**EN:** The preceding comment documents this block. The function `access_input` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`access_input` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 185-196

```cpp
  /// Store
  CUTLASS_DEVICE
  static void store(ArrayT *ptr, const ArrayT &data, int thread_idx)
  {
    AccessT *access_output = reinterpret_cast<AccessT*>(ptr);
    const AccessT *access_data = reinterpret_cast<const AccessT*>(&data);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < kStripes; ++i) {
      access_output[(BlockThreads * i) + thread_idx] = access_data[i];
    }
  }
```

**EN:** The preceding comment documents this block. The function `access_output` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`access_output` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 201-203

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
// BlockStripedReduce
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 206-218

```cpp
/// Utility for performing block-striped access (load, store, reduce) of trivially-copyable,
/// statically-sized array types to global memory.
/// (Default specialization)
template <
  int BlockThreads,
  typename ArrayT,
  typename ElementT = typename StripedAccessType<ArrayT>::Element>
struct BlockStripedReduce :
  BlockStriped<
    BlockThreads,
    ArrayT,
    ElementT>
{
```

**EN:** The preceding comment documents this block. This block begins the definition of `BlockStripedReduce`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `BlockStripedReduce` 这个 `struct`，其成员会在后续代码中展开。

### Lines 219-231

```cpp
  /// Reduce
  CUTLASS_DEVICE
  static void reduce(ArrayT *ptr, const ArrayT &data, int thread_idx)
  {
    cutlass::atomic_add<ElementT> reduce;
    ElementT *access_output = reinterpret_cast<ElementT*>(ptr);
    const ElementT *access_data = reinterpret_cast<const ElementT*>(&data);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < BlockStripedReduce::kStripes; ++i) {
      reduce(access_output + (BlockThreads * i) + thread_idx, access_data[i]);
    }
  }
```

**EN:** The preceding comment documents this block. The function `access_output` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`access_output` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 235-246

```cpp
/// Utility for performing block-striped access (load, store, reduce) of trivially-copyable,
/// statically-sized array types to global memory.
/// (Specialization for half_t.  Uses half2 vectorized-reduction.)
template <
  int BlockThreads,
  typename ArrayT>
struct BlockStripedReduce<BlockThreads, ArrayT, half_t> :
  BlockStriped<
    BlockThreads,
    ArrayT,
    half2>
{
```

**EN:** The preceding comment documents this block. This block begins the definition of `BlockStripedReduce`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `BlockStripedReduce` 这个 `struct`，其成员会在后续代码中展开。

### Lines 247-247

```cpp
  static_assert(BlockStripedReduce::kStripes % 2 == 0, "Array of half must be even number in length");
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 249-262

```cpp
  /// Reduce
  CUTLASS_DEVICE
  static void reduce(ArrayT *ptr, const ArrayT &data, int thread_idx)
  {
    cutlass::atomic_add<half2> reduce;
    half2 *access_output = reinterpret_cast<half2*>(ptr);
    const half2 *access_data = reinterpret_cast<const half2*>(&data);

    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < BlockStripedReduce::kStripes; ++i)
    {
      reduce(access_output + (BlockThreads * i) + thread_idx, access_data[i]);
    }
  }
```

**EN:** The preceding comment documents this block. The function `access_output` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`access_output` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** The file handles specialized numeric formats or packed data representations used by accelerators.
  **CN:** 该文件处理加速器常用的特种数值格式或打包数据表示。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `cutlass/array.h`, `cutlass/wmma_array.h`, `cutlass/functional.h`, `cutlass/complex.h`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `cutlass/array.h`, `cutlass/wmma_array.h`, `cutlass/functional.h`, `cutlass/complex.h`。

- **EN:** Primary namespaces: `cutlass`.
  **CN:** 主要命名空间：`cutlass`。

- **EN:** Important macros or compile flags: `CUTLASS_ARCH_WMMA_ENABLED`, `CUTLASS_DEVICE`, `CUTLASS_PRAGMA_UNROLL`.
  **CN:** 重要宏或编译开关：`CUTLASS_ARCH_WMMA_ENABLED`, `CUTLASS_DEVICE`, `CUTLASS_PRAGMA_UNROLL`。
