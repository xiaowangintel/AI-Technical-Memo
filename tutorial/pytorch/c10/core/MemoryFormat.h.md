# MemoryFormat.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/core/MemoryFormat.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
- **Purpose (CN)**: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16
```cpp
#pragma once

#include <c10/util/ArrayRef.h>
#include <c10/util/Exception.h>

#include <torch/headeronly/core/MemoryFormat.h>

#include <cstdint>
#include <vector>

namespace c10 {

// If you are seeing this, it means that this call site was not checked if
// the memory format could be preserved, and it was switched to old default
// behaviour of contiguous
#define LEGACY_CONTIGUOUS_MEMORY_FORMAT c10::get_contiguous_memory_format()
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/util/ArrayRef.h, c10/util/Exception.h, torch/headeronly/core/MemoryFormat.h; standard-library headers such as cstdint, vector. The preprocessor guard keeps declarations single-instanced when this header is included transitively. The namespace declarations place the code inside c10, matching the surrounding subsystem. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/util/ArrayRef.h、c10/util/Exception.h、torch/headeronly/core/MemoryFormat.h；标准库头文件，如 cstdint、vector。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 c10 中，与周边子系统保持一致。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 18-34
```cpp
inline std::ostream& operator<<(
    std::ostream& stream,
    at::MemoryFormat memory_format) {
  switch (memory_format) {
    case MemoryFormat::Preserve:
      return stream << "Preserve";
    case MemoryFormat::Contiguous:
      return stream << "Contiguous";
    case MemoryFormat::ChannelsLast:
      return stream << "ChannelsLast";
    case MemoryFormat::ChannelsLast3d:
      return stream << "ChannelsLast3d";
    case MemoryFormat::NumOptions:
    default:
      TORCH_CHECK(false, "Unknown memory format ", memory_format);
  }
}
```
- **EN**: Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Device- or stream-related details connect this code to backend execution context and asynchronous ordering rules. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 与设备或流相关的细节把这里的代码连接到后端执行上下文与异步顺序规则。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 36-53
```cpp
// Note: Hardcoded the channel last stride indices here to get better
// performance
template <typename T>
inline std::vector<T> get_channels_last_strides_2d(ArrayRef<T> sizes) {
  std::vector<T> strides(sizes.size());
  switch (sizes.size()) {
    case 4:
      strides[1] = 1;
      strides[3] = sizes[1];
      strides[2] = strides[3] * sizes[3];
      strides[0] = strides[2] * sizes[2];
      return strides;
    case 3:
      strides[0] = 1;
      strides[2] = sizes[0];
      strides[1] = strides[2] * sizes[2];
      return strides;
    default:
```
- **EN**: This chunk defines `strides`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `strides`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 54-71
```cpp
      TORCH_INTERNAL_ASSERT(
          false, "ChannelsLast2d doesn't support size ", sizes.size());
  }
}

inline std::vector<int64_t> get_channels_last_strides_2d(IntArrayRef sizes) {
  return get_channels_last_strides_2d<int64_t>(sizes);
}

template <typename T>
std::vector<T> get_channels_last_strides_3d(ArrayRef<T> sizes) {
  std::vector<T> strides(sizes.size());
  switch (sizes.size()) {
    case 5:
      strides[1] = 1;
      strides[4] = sizes[1];
      strides[3] = strides[4] * sizes[4];
      strides[2] = strides[3] * sizes[3];
```
- **EN**: This chunk defines `strides`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `strides`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 72-88
```cpp
      strides[0] = strides[2] * sizes[2];
      return strides;
    case 4:
      strides[0] = 1;
      strides[3] = sizes[0];
      strides[2] = strides[3] * sizes[3];
      strides[1] = strides[2] * sizes[2];
      return strides;
    default:
      TORCH_INTERNAL_ASSERT(
          false, "ChannelsLast3d doesn't support size ", sizes.size());
  }
}

inline std::vector<int64_t> get_channels_last_strides_3d(IntArrayRef sizes) {
  return get_channels_last_strides_3d<int64_t>(sizes);
}
```
- **EN**: This chunk defines `get_channels_last_strides_3d<int64_t>`, which implements a focused piece of c10 core logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Assertions and failure paths make invalid states fail early instead of silently propagating corruption. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `get_channels_last_strides_3d<int64_t>`，其作用是实现一段聚焦的 c10 核心逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 断言与失败路径让非法状态尽早暴露，而不是悄悄传播错误。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 90-107
```cpp
// NOTE:
// Below are Helper functions for is_channels_last_strides_xd.
// 1. Please do not combine these helper functions, each helper function handles
// exactly one case of sizes + memory_format, by doing this, the strides indices
// will be a constant array and we can access it using constant index number,
// the compiler will fully unroll the loop on strides indices to gain a better
// performance.
// 2. No error check in helper function, caller ensures the correctness of the
// input
// 3. All helper functions have similar comments, only 1st helper function is
// commented here.
template <typename T>
inline bool is_channels_last_strides_2d_s4(
    const ArrayRef<T> sizes,
    const ArrayRef<T> strides) {
  T min = 0;
  // special case for trivial C dimension. default to NCHW
  if (strides[1] == 0) {
```
- **EN**: It introduces or extends constant, which define the main data structures or interfaces for this portion of the file. This chunk defines `is_channels_last_strides_2d_s4`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 它引入或扩展了 constant，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `is_channels_last_strides_2d_s4`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 108-125
```cpp
    return false;
  }
  // loop strides indices
  for (auto& d : {1, 3, 2, 0}) {
    if (sizes[d] == 0) {
      return false;
    }
    if (strides[d] < min) {
      return false;
    }
    // Fallback to NCHW as default layout for ambiguous cases
    // This is the flaw of implicit memory_format from strides.
    // N111 tensor with identical strides for size 1 dimension;
    // Two cases could lead us here:
    // a. N111 contiguous Tensor ([N,1,1,1]@[1,1,1,1])
    // b. N11W contiguous Tensor sliced on the W-dimension.
    // ([N,1,1,1]@[W,W,W,W])
    if (d == 0 && min == strides[1]) {
```
- **EN**: This chunk defines `Tensor`, which implements a focused piece of c10 core logic. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `Tensor`，其作用是实现一段聚焦的 c10 核心逻辑。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-141
```cpp
      return false;
    }
    // This is necessary to:
    // 1. distinguish the memory_format of N1H1;
    //     [H, 1, 1, 1] channels_last stride
    //     [H, H, 1, 1] contiguous stride
    // 2. permutation of 1C1W:
    //     [1, C, 1, H]@[HC, H, H, 1] transpose(1, 3)
    //     [1, H, 1, C]@[HC, 1, H, H] shouldn't be identified as channels_last
    min = strides[d];
    if (sizes[d] > 1) {
      min *= sizes[d];
    }
  }
  return true;
}
```
- **EN**: This chunk continues `Tensor` and expands its control flow, data movement, or edge-case handling. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `Tensor`，进一步展开其控制流、数据流转或边界处理逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 143-160
```cpp
template <typename T>
inline bool is_channels_last_strides_3d_s5(
    const ArrayRef<T> sizes,
    const ArrayRef<T> strides) {
  T min = 0;
  if (strides[1] == 0) {
    return false;
  }
  for (auto& d : {1, 4, 3, 2, 0}) {
    if (sizes[d] == 0) {
      return false;
    }
    if (strides[d] < min) {
      return false;
    }
    if (d == 0 && min == strides[1]) {
      return false;
    }
```
- **EN**: This chunk defines `is_channels_last_strides_3d_s5`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `is_channels_last_strides_3d_s5`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 161-178
```cpp
    min = strides[d];
    if (sizes[d] > 1) {
      min *= sizes[d];
    }
  }
  return true;
}

// Note [Ambiguous is_channels_last_strides_xd]
// ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// The flaw of carrying memory_format implicitly through strides is very hard
// to WAR properly. issue #24090
// Without the history of permutation, we can't infer the memory_format of a
// tensor from the snapshot of its size & stride
// e.g.
//
// 1. We can NOT specify the memory_format of N111 tensor through strides in a
//  meaningful way;
```
- **EN**: This chunk continues `is_channels_last_strides_3d_s5` and expands its control flow, data movement, or edge-case handling. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段延续了 `is_channels_last_strides_3d_s5`，进一步展开其控制流、数据流转或边界处理逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 179-196
```cpp
//
// 2. Two path that ended up with identical size/stride
//  N11W contiguous tensor sliced at w-dimension becomes [N,1,1,1]@[W,W,W,W]
//  NC11 channels_last tensor sliced at c-dimension becomes [N,1,1,1]@[C,C,C,C]
//    So if we see a tensor [N,1,1,1]@[X,X,X,X], there's no way for us to infer
//    the memory_format of the original tensor.
//
// Due to the limitations, our temporary WAR `is_channels_last_strides` does the
// best effort to infer whether the original memory_format of a tensor is
// at::MemoryFormat::ChannelsLast. The two objectives of this function (ordered
// by their importance):
//   1. Ensure that normal shape manipulation does not accidentally change the
//      MemoryFormat of an existing tensor.
//   2. Allows user to mark MemoryFormat::ChannelsLast to tensors;
//
// The function does so via checking strides of the tensor, including strides of
// size-1 dimensions. Although conventionally PyTorch implies no restriction on
// trivial stride (stride for size-1 dimension).
```
- **EN**: This chunk declares `function`, which implements a focused piece of c10 core logic. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段声明了 `function`，其作用是实现一段聚焦的 c10 核心逻辑。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 197-214
```cpp
//
// Note that this approach is a compromise. We did not solve the problem
// completely. Many cases we will not be able to infer the correct memory
// format.
// The implementation of `is_channels_last_strides` is to serve the objectives:
// MemoryFormat::ChannelsLast has to be explicitly opted-in (no accidental
// conversion); Best effort to maintain the ChannelsLast flag.
//
// Due to the fact that this is not a bulletproof solution, through testing
// (aten/src/ATen/test/memory_format_test.cpp)
//   a. we ensure that the common tasks are supported;
//   a. we identify corner cases where the implementation compromises on.
//
// By the time accumulated permutation is enabled to replace implicit
// memory_format through strides, we should be updating our tests and fix the
// issues in our tests.
//
// We use Channels Last 2d as an example above.
```
- **EN**: This chunk declares `in`, which implements a focused piece of c10 core logic.
- **CN**: 这一段声明了 `in`，其作用是实现一段聚焦的 c10 核心逻辑。

### Lines 215-232
```cpp
// This is a general problem for all the is_channels_last_strides_xd
// implementation. Please check the helper functions
// (is_channels_last_strides_*d_s*) for more details.

template <typename T>
inline bool is_channels_last_strides_2d(
    const ArrayRef<T> sizes,
    const ArrayRef<T> strides) {
  switch (sizes.size()) {
    case 4:
      return is_channels_last_strides_2d_s4(sizes, strides);
      // NOLINTNEXTLINE(bugprone-branch-clone)
    case 3:
      // TODO dim == 3 case will be enabled once it is fully tested
      return false;
    default:
      return false;
  }
```
- **EN**: This chunk defines `is_channels_last_strides_2d_s4`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `is_channels_last_strides_2d_s4`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 233-249
```cpp
}

template <typename T>
inline bool is_channels_last_strides_3d(
    const ArrayRef<T> sizes,
    const ArrayRef<T> strides) {
  switch (sizes.size()) {
    case 5:
      return is_channels_last_strides_3d_s5(sizes, strides);
      // NOLINTNEXTLINE(bugprone-branch-clone)
    case 4:
      // TODO dim == 4 case will be enabled once it is fully tested
      return false;
    default:
      return false;
  }
}
```
- **EN**: This chunk defines `is_channels_last_strides_3d_s5`, which implements a focused piece of c10 core logic. Template machinery keeps the implementation reusable across scalar types, pointer targets, or backend-specific policies. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `is_channels_last_strides_3d_s5`，其作用是实现一段聚焦的 c10 核心逻辑。 模板机制让实现能够在标量类型、指针目标或后端策略之间复用。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 251-263
```cpp
inline bool is_channels_last_strides_2d(
    const IntArrayRef sizes,
    const IntArrayRef strides) {
  return is_channels_last_strides_2d<int64_t>(sizes, strides);
}

inline bool is_channels_last_strides_3d(
    const IntArrayRef sizes,
    const IntArrayRef strides) {
  return is_channels_last_strides_3d<int64_t>(sizes, strides);
}

} // namespace c10
```
- **EN**: This chunk defines `is_channels_last_strides_3d<int64_t>`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `is_channels_last_strides_3d<int64_t>`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **c10 core abstractions**
  - EN: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
  - CN: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。
- **constant**
  - EN: `constant` is one of the dominant symbols declared or implemented in this file.
  - CN: `constant` 是本文件声明或实现的关键符号之一。
- **get_contiguous_memory_format**
  - EN: `get_contiguous_memory_format` is one of the dominant symbols declared or implemented in this file.
  - CN: `get_contiguous_memory_format` 是本文件声明或实现的关键符号之一。
- **Stream semantics**
  - EN: Coordinates asynchronous execution ordering and per-stream resource usage.
  - CN: 协调异步执行顺序以及按流划分的资源使用。
- **Error handling**
  - EN: Surfaces invalid states through assertions, exceptions, and diagnostic messages.
  - CN: 通过断言、异常与诊断消息暴露非法状态。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/util/ArrayRef.h`、`c10/util/Exception.h`、`torch/headeronly/core/MemoryFormat.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: `cstdint`、`vector`
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10`
- **Representative symbols / 代表性符号**: `constant`、`get_contiguous_memory_format`、`get_channels_last_strides_2d`、`strides`、`get_channels_last_strides_2d<int64_t>`、`get_channels_last_strides_3d`、`get_channels_last_strides_3d<int64_t>`、`is_channels_last_strides_2d_s4`、`Tensor`、`is_channels_last_strides_3d_s5`
