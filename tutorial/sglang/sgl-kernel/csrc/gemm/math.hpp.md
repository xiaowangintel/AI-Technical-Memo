# math.hpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/math.hpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Templates, aliases, and constants
```cpp
#pragma once

#include <climits>
#include <iostream>

inline constexpr uint32_t next_pow_2(uint32_t const num) {
  if (num <= 1) return num;
  return 1 << (CHAR_BIT * sizeof(num) - __builtin_clz(num - 1));
}

template <typename A, typename B>
static inline constexpr auto div_ceil(A a, B b) {
  return (a + b - 1) / b;
}

// Round a down to the next multiple of b. The caller is responsible for making
// sure that b is non-zero
template <typename T>
inline constexpr T round_to_previous_multiple_of(T a, T b) {
  return a % b == 0 ? a : (a / b) * b;
}
```
**EN:** This section defines `next_pow_2`, `div_ceil`, `round_to_previous_multiple_of`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`next_pow_2`、`div_ceil`、`round_to_previous_multiple_of`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 22-28: Templates, aliases, and constants
```cpp

// Round a up to the next multiple of b. The caller is responsible for making
// sure that b is non-zero
template <typename T>
inline constexpr T round_to_next_multiple_of(T a, T b) {
  return a % b == 0 ? a : ((a / b) + 1) * b;
}
```
**EN:** This section defines `round_to_next_multiple_of`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`round_to_next_multiple_of`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `climits`, `iostream`
- **Path context / 路径上下文**: gemm / math.hpp
