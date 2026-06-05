# character.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/character.cpp` | `flang-rt/lib/runtime/character.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. This file centers on `character`. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件聚焦于 `character`。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````cpp
//===-- lib/runtime/character.cpp -------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Runtime/character.h"
#include "flang-rt/runtime/descriptor.h"
#include "flang-rt/runtime/terminator.h"
#include "flang-rt/runtime/tools.h"
#include "flang/Common/bit-population-count.h"
#include "flang/Common/uint128.h"
#include "flang/Runtime/character.h"
#include "flang/Runtime/cpp-type.h"
#include "flang/Runtime/freestanding-tools.h"
#include <algorithm>
#include <cstring>

namespace Fortran::runtime {

template <typename CHAR>
inline RT_API_ATTRS int CompareToBlankPadding(
    const CHAR *x, std::size_t chars) {
  using UNSIGNED_CHAR = std::make_unsigned_t<CHAR>;
  const auto blank{static_cast<UNSIGNED_CHAR>(' ')};
  for (; chars-- > 0; ++x) {
    const UNSIGNED_CHAR ux{*reinterpret_cast<const UNSIGNED_CHAR *>(x)};
    if (ux < blank) {
      return -1;
    }
````

- **L1 EN**: Comment documents intent or context: `lib/runtime/character.cpp -------------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/character.cpp -------------------------------*- C++ -*-===//`。
- **L2 EN**: Comment line provides narrative context.
  **L2 CN**: 注释行提供叙述性上下文。
- **L3 EN**: Comment documents intent or context: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释记录了意图或上下文：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment documents intent or context: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释记录了意图或上下文：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment documents intent or context: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释记录了意图或上下文：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Comment line provides narrative context.
  **L6 CN**: 注释行提供叙述性上下文。
- **L7 EN**: Comment documents intent or context: `//`.
  **L7 CN**: 注释记录了意图或上下文：`//`。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes `flang/Runtime/character.h` to access Flang runtime declarations.
  **L9 CN**: 引入 `flang/Runtime/character.h` 以使用 Flang 运行时声明。
- **L10 EN**: Includes `flang-rt/runtime/descriptor.h` to access Flang runtime public headers.
  **L10 CN**: 引入 `flang-rt/runtime/descriptor.h` 以使用 Flang 运行时公共头文件。
- **L11 EN**: Includes `flang-rt/runtime/terminator.h` to access Flang runtime public headers.
  **L11 CN**: 引入 `flang-rt/runtime/terminator.h` 以使用 Flang 运行时公共头文件。
- **L12 EN**: Includes `flang-rt/runtime/tools.h` to access Flang runtime public headers.
  **L12 CN**: 引入 `flang-rt/runtime/tools.h` 以使用 Flang 运行时公共头文件。
- **L13 EN**: Includes `flang/Common/bit-population-count.h` to access Flang common data structures and compiler-wide helpers.
  **L13 CN**: 引入 `flang/Common/bit-population-count.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L14 EN**: Includes `flang/Common/uint128.h` to access Flang common data structures and compiler-wide helpers.
  **L14 CN**: 引入 `flang/Common/uint128.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L15 EN**: Includes `flang/Runtime/character.h` to access Flang runtime declarations.
  **L15 CN**: 引入 `flang/Runtime/character.h` 以使用 Flang 运行时声明。
- **L16 EN**: Includes `flang/Runtime/cpp-type.h` to access Flang runtime declarations.
  **L16 CN**: 引入 `flang/Runtime/cpp-type.h` 以使用 Flang 运行时声明。
- **L17 EN**: Includes `flang/Runtime/freestanding-tools.h` to access Flang runtime declarations.
  **L17 CN**: 引入 `flang/Runtime/freestanding-tools.h` 以使用 Flang 运行时声明。
- **L18 EN**: Includes `algorithm` to access standard algorithms and helpers.
  **L18 CN**: 引入 `algorithm` 以使用 标准算法与辅助工具。
- **L19 EN**: Includes `cstring` to access C string and memory utilities.
  **L19 CN**: 引入 `cstring` 以使用 C 字符串与内存工具。
- **L20 EN**: Blank line separates nearby declarations or logic blocks.
  **L20 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L21 EN**: Enters namespace `Fortran` to scope related declarations.
  **L21 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L22 EN**: Blank line separates nearby declarations or logic blocks.
  **L22 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L23 EN**: Begins a template declaration parameterizing subsequent code.
  **L23 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L24 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L24 CN**: 延续周围的声明、表达式或控制流结构。
- **L25 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L25 CN**: 延续周围的声明、表达式或控制流结构。
- **L26 EN**: Defines type alias `UNSIGNED_CHAR` for readability or ABI convenience.
  **L26 CN**: 定义类型别名 `UNSIGNED_CHAR`，以提升可读性或满足 ABI 便利性。
- **L27 EN**: Executes statement `const auto blank{static_cast<UNSIGNED_CHAR>(' ')};`.
  **L27 CN**: 执行语句 `const auto blank{static_cast<UNSIGNED_CHAR>(' ')};`。
- **L28 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L28 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L29 EN**: Executes statement `const UNSIGNED_CHAR ux{*reinterpret_cast<const UNSIGNED_CHAR *>(x)};`.
  **L29 CN**: 执行语句 `const UNSIGNED_CHAR ux{*reinterpret_cast<const UNSIGNED_CHAR *>(x)};`。
- **L30 EN**: Introduces conditional control flow with an `if` statement.
  **L30 CN**: 通过 `if` 语句引入条件控制流。
- **L31 EN**: Returns from the current function, often propagating a computed result.
  **L31 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L32 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L32 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 33-64

````cpp
    if (ux > blank) {
      return 1;
    }
  }
  return 0;
}

RT_OFFLOAD_API_GROUP_BEGIN

template <typename CHAR>
RT_API_ATTRS int CharacterScalarCompare(
    const CHAR *x, const CHAR *y, std::size_t xChars, std::size_t yChars) {
  auto minChars{std::min(xChars, yChars)};
  if constexpr (sizeof(CHAR) == 1) {
    // don't use for kind=2 or =4, that would fail on little-endian machines
    int cmp{Fortran::runtime::memcmp(x, y, minChars)};
    if (cmp < 0) {
      return -1;
    }
    if (cmp > 0) {
      return 1;
    }
    if (xChars == yChars) {
      return 0;
    }
    x += minChars;
    y += minChars;
  } else {
    for (std::size_t n{minChars}; n-- > 0; ++x, ++y) {
      if (*x < *y) {
        return -1;
      }
````

- **L33 EN**: Introduces conditional control flow with an `if` statement.
  **L33 CN**: 通过 `if` 语句引入条件控制流。
- **L34 EN**: Returns from the current function, often propagating a computed result.
  **L34 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L35 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L35 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L36 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L36 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L37 EN**: Returns from the current function, often propagating a computed result.
  **L37 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L38 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L38 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L39 EN**: Blank line separates nearby declarations or logic blocks.
  **L39 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L40 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L40 CN**: 延续周围的声明、表达式或控制流结构。
- **L41 EN**: Blank line separates nearby declarations or logic blocks.
  **L41 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L42 EN**: Begins a template declaration parameterizing subsequent code.
  **L42 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L43 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L43 CN**: 延续周围的声明、表达式或控制流结构。
- **L44 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L44 CN**: 延续周围的声明、表达式或控制流结构。
- **L45 EN**: Executes statement involving `min`.
  **L45 CN**: 执行涉及 `min` 的语句。
- **L46 EN**: Introduces conditional control flow with an `if` statement.
  **L46 CN**: 通过 `if` 语句引入条件控制流。
- **L47 EN**: Comment documents intent or context: `don't use for kind=2 or =4, that would fail on little-endian machines`.
  **L47 CN**: 注释记录了意图或上下文：`don't use for kind=2 or =4, that would fail on little-endian machines`。
- **L48 EN**: Executes statement involving `memcmp`.
  **L48 CN**: 执行涉及 `memcmp` 的语句。
- **L49 EN**: Introduces conditional control flow with an `if` statement.
  **L49 CN**: 通过 `if` 语句引入条件控制流。
- **L50 EN**: Returns from the current function, often propagating a computed result.
  **L50 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L51 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L51 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L52 EN**: Introduces conditional control flow with an `if` statement.
  **L52 CN**: 通过 `if` 语句引入条件控制流。
- **L53 EN**: Returns from the current function, often propagating a computed result.
  **L53 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L54 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L54 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L55 EN**: Introduces conditional control flow with an `if` statement.
  **L55 CN**: 通过 `if` 语句引入条件控制流。
- **L56 EN**: Returns from the current function, often propagating a computed result.
  **L56 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L57 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L57 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L58 EN**: Initializes or updates `+`.
  **L58 CN**: 初始化或更新 `+`。
- **L59 EN**: Initializes or updates `+`.
  **L59 CN**: 初始化或更新 `+`。
- **L60 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L60 CN**: 延续周围的声明、表达式或控制流结构。
- **L61 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L61 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L62 EN**: Introduces conditional control flow with an `if` statement.
  **L62 CN**: 通过 `if` 语句引入条件控制流。
- **L63 EN**: Returns from the current function, often propagating a computed result.
  **L63 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L64 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L64 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 65-96

````cpp
      if (*x > *y) {
        return 1;
      }
    }
  }
  if (int cmp{CompareToBlankPadding(x, xChars - minChars)}) {
    return cmp;
  }
  return -CompareToBlankPadding(y, yChars - minChars);
}

template RT_API_ATTRS int CharacterScalarCompare<char>(
    const char *x, const char *y, std::size_t xChars, std::size_t yChars);
template RT_API_ATTRS int CharacterScalarCompare<char16_t>(const char16_t *x,
    const char16_t *y, std::size_t xChars, std::size_t yChars);
template RT_API_ATTRS int CharacterScalarCompare<char32_t>(const char32_t *x,
    const char32_t *y, std::size_t xChars, std::size_t yChars);

RT_OFFLOAD_API_GROUP_END

// Shift count to use when converting between character lengths
// and byte counts.
template <typename CHAR>
constexpr int shift{common::TrailingZeroBitCount(sizeof(CHAR))};

template <typename CHAR>
static RT_API_ATTRS void Compare(Descriptor &result, const Descriptor &x,
    const Descriptor &y, const Terminator &terminator) {
  RUNTIME_CHECK(
      terminator, x.rank() == y.rank() || x.rank() == 0 || y.rank() == 0);
  int rank{std::max(x.rank(), y.rank())};
  SubscriptValue ub[maxRank], xAt[maxRank], yAt[maxRank];
````

- **L65 EN**: Introduces conditional control flow with an `if` statement.
  **L65 CN**: 通过 `if` 语句引入条件控制流。
- **L66 EN**: Returns from the current function, often propagating a computed result.
  **L66 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L67 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L67 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L68 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L68 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L69 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L69 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L70 EN**: Introduces conditional control flow with an `if` statement.
  **L70 CN**: 通过 `if` 语句引入条件控制流。
- **L71 EN**: Returns from the current function, often propagating a computed result.
  **L71 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L72 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L72 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L73 EN**: Returns from the current function, often propagating a computed result.
  **L73 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L74 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L74 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L75 EN**: Blank line separates nearby declarations or logic blocks.
  **L75 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L76 EN**: Begins a template declaration parameterizing subsequent code.
  **L76 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L77 EN**: Executes statement `const char *x, const char *y, std::size_t xChars, std::size_t yChars);`.
  **L77 CN**: 执行语句 `const char *x, const char *y, std::size_t xChars, std::size_t yChars);`。
- **L78 EN**: Begins a template declaration parameterizing subsequent code.
  **L78 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L79 EN**: Executes statement `const char16_t *y, std::size_t xChars, std::size_t yChars);`.
  **L79 CN**: 执行语句 `const char16_t *y, std::size_t xChars, std::size_t yChars);`。
- **L80 EN**: Begins a template declaration parameterizing subsequent code.
  **L80 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L81 EN**: Executes statement `const char32_t *y, std::size_t xChars, std::size_t yChars);`.
  **L81 CN**: 执行语句 `const char32_t *y, std::size_t xChars, std::size_t yChars);`。
- **L82 EN**: Blank line separates nearby declarations or logic blocks.
  **L82 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L83 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L83 CN**: 延续周围的声明、表达式或控制流结构。
- **L84 EN**: Blank line separates nearby declarations or logic blocks.
  **L84 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L85 EN**: Comment documents intent or context: `Shift count to use when converting between character lengths`.
  **L85 CN**: 注释记录了意图或上下文：`Shift count to use when converting between character lengths`。
- **L86 EN**: Comment documents intent or context: `and byte counts.`.
  **L86 CN**: 注释记录了意图或上下文：`and byte counts.`。
- **L87 EN**: Begins a template declaration parameterizing subsequent code.
  **L87 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L88 EN**: Executes statement involving `TrailingZeroBitCount`.
  **L88 CN**: 执行涉及 `TrailingZeroBitCount` 的语句。
- **L89 EN**: Blank line separates nearby declarations or logic blocks.
  **L89 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L90 EN**: Begins a template declaration parameterizing subsequent code.
  **L90 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L91 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L91 CN**: 延续周围的声明、表达式或控制流结构。
- **L92 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L92 CN**: 延续周围的声明、表达式或控制流结构。
- **L93 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L93 CN**: 延续周围的声明、表达式或控制流结构。
- **L94 EN**: Executes statement involving `rank`.
  **L94 CN**: 执行涉及 `rank` 的语句。
- **L95 EN**: Executes statement involving `max`.
  **L95 CN**: 执行涉及 `max` 的语句。
- **L96 EN**: Executes statement `SubscriptValue ub[maxRank], xAt[maxRank], yAt[maxRank];`.
  **L96 CN**: 执行语句 `SubscriptValue ub[maxRank], xAt[maxRank], yAt[maxRank];`。

### Lines 97-128

````cpp
  SubscriptValue elements{1};
  for (int j{0}; j < rank; ++j) {
    if (x.rank() > 0 && y.rank() > 0) {
      SubscriptValue xUB{x.GetDimension(j).Extent()};
      SubscriptValue yUB{y.GetDimension(j).Extent()};
      if (xUB != yUB) {
        terminator.Crash("Character array comparison: operands are not "
                         "conforming on dimension %d (%jd != %jd)",
            j + 1, static_cast<std::intmax_t>(xUB),
            static_cast<std::intmax_t>(yUB));
      }
      ub[j] = xUB;
    } else {
      ub[j] = (x.rank() ? x : y).GetDimension(j).Extent();
    }
    elements *= ub[j];
  }
  x.GetLowerBounds(xAt);
  y.GetLowerBounds(yAt);
  result.Establish(
      TypeCategory::Logical, 1, nullptr, rank, ub, CFI_attribute_allocatable);
  for (int j{0}; j < rank; ++j) {
    result.GetDimension(j).SetBounds(1, ub[j]);
  }
  if (result.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("Compare: could not allocate storage for result");
  }
  std::size_t xChars{x.ElementBytes() >> shift<CHAR>};
  std::size_t yChars{y.ElementBytes() >> shift<char>};
  for (SubscriptValue resultAt{0}; elements-- > 0;
       ++resultAt, x.IncrementSubscripts(xAt), y.IncrementSubscripts(yAt)) {
    *result.OffsetElement<char>(resultAt) = CharacterScalarCompare<CHAR>(
````

- **L97 EN**: Executes statement `SubscriptValue elements{1};`.
  **L97 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L98 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L98 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L99 EN**: Introduces conditional control flow with an `if` statement.
  **L99 CN**: 通过 `if` 语句引入条件控制流。
- **L100 EN**: Executes statement involving `GetDimension`.
  **L100 CN**: 执行涉及 `GetDimension` 的语句。
- **L101 EN**: Executes statement involving `GetDimension`.
  **L101 CN**: 执行涉及 `GetDimension` 的语句。
- **L102 EN**: Introduces conditional control flow with an `if` statement.
  **L102 CN**: 通过 `if` 语句引入条件控制流。
- **L103 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L103 CN**: 延续周围的声明、表达式或控制流结构。
- **L104 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L104 CN**: 延续周围的声明、表达式或控制流结构。
- **L105 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L105 CN**: 延续周围的声明、表达式或控制流结构。
- **L106 EN**: Executes statement `static_cast<std::intmax_t>(yUB));`.
  **L106 CN**: 执行语句 `static_cast<std::intmax_t>(yUB));`。
- **L107 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L107 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L108 EN**: Initializes or updates `ub[j]`.
  **L108 CN**: 初始化或更新 `ub[j]`。
- **L109 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L109 CN**: 延续周围的声明、表达式或控制流结构。
- **L110 EN**: Initializes or updates `ub[j]`.
  **L110 CN**: 初始化或更新 `ub[j]`。
- **L111 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L111 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L112 EN**: Initializes or updates `*`.
  **L112 CN**: 初始化或更新 `*`。
- **L113 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L113 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L114 EN**: Executes statement involving `GetLowerBounds`.
  **L114 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L115 EN**: Executes statement involving `GetLowerBounds`.
  **L115 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L116 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L116 CN**: 延续周围的声明、表达式或控制流结构。
- **L117 EN**: Executes statement `TypeCategory::Logical, 1, nullptr, rank, ub, CFI_attribute_allocatable);`.
  **L117 CN**: 执行语句 `TypeCategory::Logical, 1, nullptr, rank, ub, CFI_attribute_allocatable);`。
- **L118 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L118 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L119 EN**: Executes statement involving `GetDimension`.
  **L119 CN**: 执行涉及 `GetDimension` 的语句。
- **L120 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L120 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L121 EN**: Introduces conditional control flow with an `if` statement.
  **L121 CN**: 通过 `if` 语句引入条件控制流。
- **L122 EN**: Executes statement involving `Crash`.
  **L122 CN**: 执行涉及 `Crash` 的语句。
- **L123 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L123 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L124 EN**: Executes statement involving `ElementBytes`.
  **L124 CN**: 执行涉及 `ElementBytes` 的语句。
- **L125 EN**: Executes statement involving `ElementBytes`.
  **L125 CN**: 执行涉及 `ElementBytes` 的语句。
- **L126 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L126 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L127 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L127 CN**: 延续周围的声明、表达式或控制流结构。
- **L128 EN**: Comment documents intent or context: `result.OffsetElement<char>(resultAt) = CharacterScalarCompare<CHAR>(`.
  **L128 CN**: 注释记录了意图或上下文：`result.OffsetElement<char>(resultAt) = CharacterScalarCompare<CHAR>(`。

### Lines 129-160

````cpp
        x.Element<CHAR>(xAt), y.Element<CHAR>(yAt), xChars, yChars);
  }
}

template <typename CHAR, bool ADJUSTR>
static RT_API_ATTRS void Adjust(CHAR *to, const CHAR *from, std::size_t chars) {
  if constexpr (ADJUSTR) {
    std::size_t j{chars}, k{chars};
    for (; k > 0 && from[k - 1] == ' '; --k) {
    }
    while (k > 0) {
      to[--j] = from[--k];
    }
    while (j > 0) {
      to[--j] = ' ';
    }
  } else { // ADJUSTL
    std::size_t j{0}, k{0};
    for (; k < chars && from[k] == ' '; ++k) {
    }
    while (k < chars) {
      to[j++] = from[k++];
    }
    while (j < chars) {
      to[j++] = ' ';
    }
  }
}

template <typename CHAR, bool ADJUSTR>
static RT_API_ATTRS void AdjustLRHelper(Descriptor &result,
    const Descriptor &string, const Terminator &terminator) {
````

- **L129 EN**: Executes statement `x.Element<CHAR>(xAt), y.Element<CHAR>(yAt), xChars, yChars);`.
  **L129 CN**: 执行语句 `x.Element<CHAR>(xAt), y.Element<CHAR>(yAt), xChars, yChars);`。
- **L130 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L130 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L131 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L131 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L132 EN**: Blank line separates nearby declarations or logic blocks.
  **L132 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L133 EN**: Begins a template declaration parameterizing subsequent code.
  **L133 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L134 EN**: Declares or defines callable `Adjust`.
  **L134 CN**: 声明或定义可调用实体 `Adjust`。
- **L135 EN**: Introduces conditional control flow with an `if` statement.
  **L135 CN**: 通过 `if` 语句引入条件控制流。
- **L136 EN**: Executes statement `std::size_t j{chars}, k{chars};`.
  **L136 CN**: 执行语句 `std::size_t j{chars}, k{chars};`。
- **L137 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L137 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L138 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L138 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L139 EN**: Starts a `while` loop controlled by a runtime condition.
  **L139 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L140 EN**: Initializes or updates `to[--j]`.
  **L140 CN**: 初始化或更新 `to[--j]`。
- **L141 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L141 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L142 EN**: Starts a `while` loop controlled by a runtime condition.
  **L142 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L143 EN**: Initializes or updates `to[--j]`.
  **L143 CN**: 初始化或更新 `to[--j]`。
- **L144 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L144 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L145 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L145 CN**: 延续周围的声明、表达式或控制流结构。
- **L146 EN**: Executes statement `std::size_t j{0}, k{0};`.
  **L146 CN**: 执行语句 `std::size_t j{0}, k{0};`。
- **L147 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L147 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L148 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L148 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L149 EN**: Starts a `while` loop controlled by a runtime condition.
  **L149 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L150 EN**: Initializes or updates `to[j++]`.
  **L150 CN**: 初始化或更新 `to[j++]`。
- **L151 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L151 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L152 EN**: Starts a `while` loop controlled by a runtime condition.
  **L152 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L153 EN**: Initializes or updates `to[j++]`.
  **L153 CN**: 初始化或更新 `to[j++]`。
- **L154 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L154 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L155 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L155 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L156 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L156 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L157 EN**: Blank line separates nearby declarations or logic blocks.
  **L157 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L158 EN**: Begins a template declaration parameterizing subsequent code.
  **L158 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L159 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L159 CN**: 延续周围的声明、表达式或控制流结构。
- **L160 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L160 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 161-192

````cpp
  int rank{string.rank()};
  SubscriptValue ub[maxRank], stringAt[maxRank];
  SubscriptValue elements{1};
  for (int j{0}; j < rank; ++j) {
    ub[j] = string.GetDimension(j).Extent();
    elements *= ub[j];
    stringAt[j] = 1;
  }
  string.GetLowerBounds(stringAt);
  std::size_t elementBytes{string.ElementBytes()};
  result.Establish(string.type(), elementBytes, nullptr, rank, ub,
      CFI_attribute_allocatable);
  for (int j{0}; j < rank; ++j) {
    result.GetDimension(j).SetBounds(1, ub[j]);
  }
  if (result.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("ADJUSTL/R: could not allocate storage for result");
  }
  for (SubscriptValue resultAt{0}; elements-- > 0;
       resultAt += elementBytes, string.IncrementSubscripts(stringAt)) {
    Adjust<CHAR, ADJUSTR>(result.OffsetElement<CHAR>(resultAt),
        string.Element<const CHAR>(stringAt), elementBytes >> shift<CHAR>);
  }
}

template <bool ADJUSTR>
RT_API_ATTRS void AdjustLR(Descriptor &result, const Descriptor &string,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  switch (string.raw().type) {
  case CFI_type_char:
    AdjustLRHelper<char, ADJUSTR>(result, string, terminator);
````

- **L161 EN**: Executes statement involving `rank`.
  **L161 CN**: 执行涉及 `rank` 的语句。
- **L162 EN**: Executes statement `SubscriptValue ub[maxRank], stringAt[maxRank];`.
  **L162 CN**: 执行语句 `SubscriptValue ub[maxRank], stringAt[maxRank];`。
- **L163 EN**: Executes statement `SubscriptValue elements{1};`.
  **L163 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L164 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L164 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L165 EN**: Initializes or updates `ub[j]`.
  **L165 CN**: 初始化或更新 `ub[j]`。
- **L166 EN**: Initializes or updates `*`.
  **L166 CN**: 初始化或更新 `*`。
- **L167 EN**: Initializes or updates `stringAt[j]`.
  **L167 CN**: 初始化或更新 `stringAt[j]`。
- **L168 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L168 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L169 EN**: Executes statement involving `GetLowerBounds`.
  **L169 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L170 EN**: Executes statement involving `ElementBytes`.
  **L170 CN**: 执行涉及 `ElementBytes` 的语句。
- **L171 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L171 CN**: 延续周围的声明、表达式或控制流结构。
- **L172 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L172 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L173 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L173 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L174 EN**: Executes statement involving `GetDimension`.
  **L174 CN**: 执行涉及 `GetDimension` 的语句。
- **L175 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L175 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L176 EN**: Introduces conditional control flow with an `if` statement.
  **L176 CN**: 通过 `if` 语句引入条件控制流。
- **L177 EN**: Executes statement involving `Crash`.
  **L177 CN**: 执行涉及 `Crash` 的语句。
- **L178 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L178 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L179 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L179 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L180 EN**: Initializes or updates `+`.
  **L180 CN**: 初始化或更新 `+`。
- **L181 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L181 CN**: 延续周围的声明、表达式或控制流结构。
- **L182 EN**: Executes statement `string.Element<const CHAR>(stringAt), elementBytes >> shift<CHAR>);`.
  **L182 CN**: 执行语句 `string.Element<const CHAR>(stringAt), elementBytes >> shift<CHAR>);`。
- **L183 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L183 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L184 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L184 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L185 EN**: Blank line separates nearby declarations or logic blocks.
  **L185 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L186 EN**: Begins a template declaration parameterizing subsequent code.
  **L186 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L187 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L187 CN**: 延续周围的声明、表达式或控制流结构。
- **L188 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L188 CN**: 延续周围的声明、表达式或控制流结构。
- **L189 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L189 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L190 EN**: Begins a `switch` dispatch over discrete cases.
  **L190 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L191 EN**: Marks one `switch` case label.
  **L191 CN**: 标记一个 `switch` 的 case 标签。
- **L192 EN**: Executes statement `AdjustLRHelper<char, ADJUSTR>(result, string, terminator);`.
  **L192 CN**: 执行语句 `AdjustLRHelper<char, ADJUSTR>(result, string, terminator);`。

### Lines 193-224

````cpp
    break;
  case CFI_type_char16_t:
    AdjustLRHelper<char16_t, ADJUSTR>(result, string, terminator);
    break;
  case CFI_type_char32_t:
    AdjustLRHelper<char32_t, ADJUSTR>(result, string, terminator);
    break;
  default:
    terminator.Crash("ADJUSTL/R: bad string type code %d",
        static_cast<int>(string.raw().type));
  }
}

template <typename CHAR>
inline RT_API_ATTRS std::size_t LenTrim(const CHAR *x, std::size_t chars) {
  while (chars > 0 && x[chars - 1] == ' ') {
    --chars;
  }
  return chars;
}

template <typename INT, typename CHAR>
static RT_API_ATTRS void LenTrim(Descriptor &result, const Descriptor &string,
    const Terminator &terminator) {
  int rank{string.rank()};
  SubscriptValue ub[maxRank], stringAt[maxRank];
  SubscriptValue elements{1};
  for (int j{0}; j < rank; ++j) {
    ub[j] = string.GetDimension(j).Extent();
    elements *= ub[j];
  }
  string.GetLowerBounds(stringAt);
````

- **L193 EN**: Breaks out of the current loop or switch.
  **L193 CN**: 跳出当前循环或 switch。
- **L194 EN**: Marks one `switch` case label.
  **L194 CN**: 标记一个 `switch` 的 case 标签。
- **L195 EN**: Executes statement `AdjustLRHelper<char16_t, ADJUSTR>(result, string, terminator);`.
  **L195 CN**: 执行语句 `AdjustLRHelper<char16_t, ADJUSTR>(result, string, terminator);`。
- **L196 EN**: Breaks out of the current loop or switch.
  **L196 CN**: 跳出当前循环或 switch。
- **L197 EN**: Marks one `switch` case label.
  **L197 CN**: 标记一个 `switch` 的 case 标签。
- **L198 EN**: Executes statement `AdjustLRHelper<char32_t, ADJUSTR>(result, string, terminator);`.
  **L198 CN**: 执行语句 `AdjustLRHelper<char32_t, ADJUSTR>(result, string, terminator);`。
- **L199 EN**: Breaks out of the current loop or switch.
  **L199 CN**: 跳出当前循环或 switch。
- **L200 EN**: Provides the default branch for a `switch` statement.
  **L200 CN**: 为 `switch` 语句提供默认分支。
- **L201 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L201 CN**: 延续周围的声明、表达式或控制流结构。
- **L202 EN**: Executes statement involving `raw`.
  **L202 CN**: 执行涉及 `raw` 的语句。
- **L203 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L203 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L204 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L204 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L205 EN**: Blank line separates nearby declarations or logic blocks.
  **L205 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L206 EN**: Begins a template declaration parameterizing subsequent code.
  **L206 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L207 EN**: Declares or defines callable `LenTrim`.
  **L207 CN**: 声明或定义可调用实体 `LenTrim`。
- **L208 EN**: Starts a `while` loop controlled by a runtime condition.
  **L208 CN**: 开始一个由运行时条件控制的 `while` 循环。
- **L209 EN**: Executes statement `--chars;`.
  **L209 CN**: 执行语句 `--chars;`。
- **L210 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L210 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L211 EN**: Returns from the current function, often propagating a computed result.
  **L211 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L212 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L212 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L213 EN**: Blank line separates nearby declarations or logic blocks.
  **L213 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L214 EN**: Begins a template declaration parameterizing subsequent code.
  **L214 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L215 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L215 CN**: 延续周围的声明、表达式或控制流结构。
- **L216 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L216 CN**: 延续周围的声明、表达式或控制流结构。
- **L217 EN**: Executes statement involving `rank`.
  **L217 CN**: 执行涉及 `rank` 的语句。
- **L218 EN**: Executes statement `SubscriptValue ub[maxRank], stringAt[maxRank];`.
  **L218 CN**: 执行语句 `SubscriptValue ub[maxRank], stringAt[maxRank];`。
- **L219 EN**: Executes statement `SubscriptValue elements{1};`.
  **L219 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L220 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L220 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L221 EN**: Initializes or updates `ub[j]`.
  **L221 CN**: 初始化或更新 `ub[j]`。
- **L222 EN**: Initializes or updates `*`.
  **L222 CN**: 初始化或更新 `*`。
- **L223 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L223 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L224 EN**: Executes statement involving `GetLowerBounds`.
  **L224 CN**: 执行涉及 `GetLowerBounds` 的语句。

### Lines 225-256

````cpp
  result.Establish(TypeCategory::Integer, sizeof(INT), nullptr, rank, ub,
      CFI_attribute_allocatable);
  for (int j{0}; j < rank; ++j) {
    result.GetDimension(j).SetBounds(1, ub[j]);
  }
  if (result.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("LEN_TRIM: could not allocate storage for result");
  }
  std::size_t stringElementChars{string.ElementBytes() >> shift<CHAR>};
  for (SubscriptValue resultAt{0}; elements-- > 0;
       resultAt += sizeof(INT), string.IncrementSubscripts(stringAt)) {
    *result.OffsetElement<INT>(resultAt) =
        LenTrim(string.Element<CHAR>(stringAt), stringElementChars);
  }
}

template <typename CHAR>
static RT_API_ATTRS void LenTrimKind(Descriptor &result,
    const Descriptor &string, int kind, const Terminator &terminator) {
  switch (kind) {
  case 1:
    LenTrim<CppTypeFor<TypeCategory::Integer, 1>, CHAR>(
        result, string, terminator);
    break;
  case 2:
    LenTrim<CppTypeFor<TypeCategory::Integer, 2>, CHAR>(
        result, string, terminator);
    break;
  case 4:
    LenTrim<CppTypeFor<TypeCategory::Integer, 4>, CHAR>(
        result, string, terminator);
    break;
````

- **L225 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L225 CN**: 延续周围的声明、表达式或控制流结构。
- **L226 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L226 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L227 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L227 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L228 EN**: Executes statement involving `GetDimension`.
  **L228 CN**: 执行涉及 `GetDimension` 的语句。
- **L229 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L229 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L230 EN**: Introduces conditional control flow with an `if` statement.
  **L230 CN**: 通过 `if` 语句引入条件控制流。
- **L231 EN**: Executes statement involving `Crash`.
  **L231 CN**: 执行涉及 `Crash` 的语句。
- **L232 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L232 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L233 EN**: Executes statement involving `ElementBytes`.
  **L233 CN**: 执行涉及 `ElementBytes` 的语句。
- **L234 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L234 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L235 EN**: Initializes or updates `+`.
  **L235 CN**: 初始化或更新 `+`。
- **L236 EN**: Comment documents intent or context: `result.OffsetElement<INT>(resultAt) =`.
  **L236 CN**: 注释记录了意图或上下文：`result.OffsetElement<INT>(resultAt) =`。
- **L237 EN**: Executes statement involving `LenTrim`.
  **L237 CN**: 执行涉及 `LenTrim` 的语句。
- **L238 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L238 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L239 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L239 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L240 EN**: Blank line separates nearby declarations or logic blocks.
  **L240 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L241 EN**: Begins a template declaration parameterizing subsequent code.
  **L241 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L242 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L242 CN**: 延续周围的声明、表达式或控制流结构。
- **L243 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L243 CN**: 延续周围的声明、表达式或控制流结构。
- **L244 EN**: Begins a `switch` dispatch over discrete cases.
  **L244 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L245 EN**: Marks one `switch` case label.
  **L245 CN**: 标记一个 `switch` 的 case 标签。
- **L246 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L246 CN**: 延续周围的声明、表达式或控制流结构。
- **L247 EN**: Executes statement `result, string, terminator);`.
  **L247 CN**: 执行语句 `result, string, terminator);`。
- **L248 EN**: Breaks out of the current loop or switch.
  **L248 CN**: 跳出当前循环或 switch。
- **L249 EN**: Marks one `switch` case label.
  **L249 CN**: 标记一个 `switch` 的 case 标签。
- **L250 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L250 CN**: 延续周围的声明、表达式或控制流结构。
- **L251 EN**: Executes statement `result, string, terminator);`.
  **L251 CN**: 执行语句 `result, string, terminator);`。
- **L252 EN**: Breaks out of the current loop or switch.
  **L252 CN**: 跳出当前循环或 switch。
- **L253 EN**: Marks one `switch` case label.
  **L253 CN**: 标记一个 `switch` 的 case 标签。
- **L254 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L254 CN**: 延续周围的声明、表达式或控制流结构。
- **L255 EN**: Executes statement `result, string, terminator);`.
  **L255 CN**: 执行语句 `result, string, terminator);`。
- **L256 EN**: Breaks out of the current loop or switch.
  **L256 CN**: 跳出当前循环或 switch。

### Lines 257-288

````cpp
  case 8:
    LenTrim<CppTypeFor<TypeCategory::Integer, 8>, CHAR>(
        result, string, terminator);
    break;
  case 16:
    LenTrim<CppTypeFor<TypeCategory::Integer, 16>, CHAR>(
        result, string, terminator);
    break;
  default:
    terminator.Crash(
        "not yet implemented: CHARACTER(KIND=%d) in LEN_TRIM intrinsic", kind);
  }
}

// INDEX implementation
template <typename CHAR>
inline RT_API_ATTRS std::size_t Index(const CHAR *x, std::size_t xLen,
    const CHAR *want, std::size_t wantLen, bool back) {
  if (xLen < wantLen) {
    return 0;
  }
  if (xLen == 0) {
    return 1; // wantLen is also 0, so trivial match
  }
  if (back) {
    // If wantLen==0, returns xLen + 1 per standard (and all other compilers)
    std::size_t at{xLen - wantLen + 1};
    for (; at > 0; --at) {
      std::size_t j{1};
      for (; j <= wantLen; ++j) {
        if (x[at + j - 2] != want[j - 1]) {
          break;
````

- **L257 EN**: Marks one `switch` case label.
  **L257 CN**: 标记一个 `switch` 的 case 标签。
- **L258 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L258 CN**: 延续周围的声明、表达式或控制流结构。
- **L259 EN**: Executes statement `result, string, terminator);`.
  **L259 CN**: 执行语句 `result, string, terminator);`。
- **L260 EN**: Breaks out of the current loop or switch.
  **L260 CN**: 跳出当前循环或 switch。
- **L261 EN**: Marks one `switch` case label.
  **L261 CN**: 标记一个 `switch` 的 case 标签。
- **L262 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L262 CN**: 延续周围的声明、表达式或控制流结构。
- **L263 EN**: Executes statement `result, string, terminator);`.
  **L263 CN**: 执行语句 `result, string, terminator);`。
- **L264 EN**: Breaks out of the current loop or switch.
  **L264 CN**: 跳出当前循环或 switch。
- **L265 EN**: Provides the default branch for a `switch` statement.
  **L265 CN**: 为 `switch` 语句提供默认分支。
- **L266 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L266 CN**: 延续周围的声明、表达式或控制流结构。
- **L267 EN**: Executes statement involving `CHARACTER`.
  **L267 CN**: 执行涉及 `CHARACTER` 的语句。
- **L268 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L268 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L269 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L269 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L270 EN**: Blank line separates nearby declarations or logic blocks.
  **L270 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L271 EN**: Comment documents intent or context: `INDEX implementation`.
  **L271 CN**: 注释记录了意图或上下文：`INDEX implementation`。
- **L272 EN**: Begins a template declaration parameterizing subsequent code.
  **L272 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L273 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L273 CN**: 延续周围的声明、表达式或控制流结构。
- **L274 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L274 CN**: 延续周围的声明、表达式或控制流结构。
- **L275 EN**: Introduces conditional control flow with an `if` statement.
  **L275 CN**: 通过 `if` 语句引入条件控制流。
- **L276 EN**: Returns from the current function, often propagating a computed result.
  **L276 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L277 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L277 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L278 EN**: Introduces conditional control flow with an `if` statement.
  **L278 CN**: 通过 `if` 语句引入条件控制流。
- **L279 EN**: Returns from the current function, often propagating a computed result.
  **L279 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L280 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L280 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L281 EN**: Introduces conditional control flow with an `if` statement.
  **L281 CN**: 通过 `if` 语句引入条件控制流。
- **L282 EN**: Comment documents intent or context: `If wantLen==0, returns xLen + 1 per standard (and all other compilers)`.
  **L282 CN**: 注释记录了意图或上下文：`If wantLen==0, returns xLen + 1 per standard (and all other compilers)`。
- **L283 EN**: Executes statement `std::size_t at{xLen - wantLen + 1};`.
  **L283 CN**: 执行语句 `std::size_t at{xLen - wantLen + 1};`。
- **L284 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L284 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L285 EN**: Executes statement `std::size_t j{1};`.
  **L285 CN**: 执行语句 `std::size_t j{1};`。
- **L286 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L286 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L287 EN**: Introduces conditional control flow with an `if` statement.
  **L287 CN**: 通过 `if` 语句引入条件控制流。
- **L288 EN**: Breaks out of the current loop or switch.
  **L288 CN**: 跳出当前循环或 switch。

### Lines 289-320

````cpp
        }
      }
      if (j > wantLen) {
        return at;
      }
    }
    return 0;
  }
  if (wantLen == 1) {
    // Trivial case for single character lookup.
    // We can use simple forward search.
    CHAR ch{want[0]};
    if constexpr (std::is_same_v<CHAR, char>) {
      if (auto pos{reinterpret_cast<const CHAR *>(
              Fortran::runtime::memchr(x, ch, xLen))}) {
        return pos - x + 1;
      }
    } else {
      for (std::size_t at{0}; at < xLen; ++at) {
        if (x[at] == ch) {
          return at + 1;
        }
      }
    }
    return 0;
  }
  // Non-trivial forward substring search: use a simplified form of
  // Boyer-Moore substring searching.
  for (std::size_t at{1}; at + wantLen - 1 <= xLen;) {
    // Compare x(at:at+wantLen-1) with want(1:wantLen).
    // The comparison proceeds from the ends of the substrings forward
    // so that we can skip ahead by multiple positions on a miss.
````

- **L289 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L289 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L290 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L290 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L291 EN**: Introduces conditional control flow with an `if` statement.
  **L291 CN**: 通过 `if` 语句引入条件控制流。
- **L292 EN**: Returns from the current function, often propagating a computed result.
  **L292 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L293 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L293 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L294 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L294 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L295 EN**: Returns from the current function, often propagating a computed result.
  **L295 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L296 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L296 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L297 EN**: Introduces conditional control flow with an `if` statement.
  **L297 CN**: 通过 `if` 语句引入条件控制流。
- **L298 EN**: Comment documents intent or context: `Trivial case for single character lookup.`.
  **L298 CN**: 注释记录了意图或上下文：`Trivial case for single character lookup.`。
- **L299 EN**: Comment documents intent or context: `We can use simple forward search.`.
  **L299 CN**: 注释记录了意图或上下文：`We can use simple forward search.`。
- **L300 EN**: Executes statement `CHAR ch{want[0]};`.
  **L300 CN**: 执行语句 `CHAR ch{want[0]};`。
- **L301 EN**: Introduces conditional control flow with an `if` statement.
  **L301 CN**: 通过 `if` 语句引入条件控制流。
- **L302 EN**: Introduces conditional control flow with an `if` statement.
  **L302 CN**: 通过 `if` 语句引入条件控制流。
- **L303 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L303 CN**: 延续周围的声明、表达式或控制流结构。
- **L304 EN**: Returns from the current function, often propagating a computed result.
  **L304 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L305 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L305 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L306 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L306 CN**: 延续周围的声明、表达式或控制流结构。
- **L307 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L307 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L308 EN**: Introduces conditional control flow with an `if` statement.
  **L308 CN**: 通过 `if` 语句引入条件控制流。
- **L309 EN**: Returns from the current function, often propagating a computed result.
  **L309 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L310 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L310 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L311 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L311 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L312 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L312 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L313 EN**: Returns from the current function, often propagating a computed result.
  **L313 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L314 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L314 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L315 EN**: Comment documents intent or context: `Non-trivial forward substring search: use a simplified form of`.
  **L315 CN**: 注释记录了意图或上下文：`Non-trivial forward substring search: use a simplified form of`。
- **L316 EN**: Comment documents intent or context: `Boyer-Moore substring searching.`.
  **L316 CN**: 注释记录了意图或上下文：`Boyer-Moore substring searching.`。
- **L317 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L317 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L318 EN**: Comment documents intent or context: `Compare x(at:at+wantLen-1) with want(1:wantLen).`.
  **L318 CN**: 注释记录了意图或上下文：`Compare x(at:at+wantLen-1) with want(1:wantLen).`。
- **L319 EN**: Comment documents intent or context: `The comparison proceeds from the ends of the substrings forward`.
  **L319 CN**: 注释记录了意图或上下文：`The comparison proceeds from the ends of the substrings forward`。
- **L320 EN**: Comment documents intent or context: `so that we can skip ahead by multiple positions on a miss.`.
  **L320 CN**: 注释记录了意图或上下文：`so that we can skip ahead by multiple positions on a miss.`。

### Lines 321-352

````cpp
    std::size_t j{wantLen};
    CHAR ch;
    for (; j > 0; --j) {
      ch = x[at + j - 2];
      if (ch != want[j - 1]) {
        break;
      }
    }
    if (j == 0) {
      return at; // found a match
    }
    // Suppose we have at==2:
    // "THAT FORTRAN THAT I RAN" <- the string (x) in which we search
    //   "THAT I RAN"            <- the string (want) for which we search
    //          ^------------------ j==7, ch=='T'
    // We can shift ahead 3 positions to at==5 to align the 'T's:
    // "THAT FORTRAN THAT I RAN"
    //      "THAT I RAN"
    std::size_t shift{1};
    for (; shift < j; ++shift) {
      if (want[j - shift - 1] == ch) {
        break;
      }
    }
    at += shift;
  }
  return 0;
}

// SCAN and VERIFY implementation help.  These intrinsic functions
// do pretty much the same thing, so they're templatized with a
// distinguishing flag.
````

- **L321 EN**: Executes statement `std::size_t j{wantLen};`.
  **L321 CN**: 执行语句 `std::size_t j{wantLen};`。
- **L322 EN**: Executes statement `CHAR ch;`.
  **L322 CN**: 执行语句 `CHAR ch;`。
- **L323 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L323 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L324 EN**: Initializes or updates `ch`.
  **L324 CN**: 初始化或更新 `ch`。
- **L325 EN**: Introduces conditional control flow with an `if` statement.
  **L325 CN**: 通过 `if` 语句引入条件控制流。
- **L326 EN**: Breaks out of the current loop or switch.
  **L326 CN**: 跳出当前循环或 switch。
- **L327 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L327 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L328 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L328 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L329 EN**: Introduces conditional control flow with an `if` statement.
  **L329 CN**: 通过 `if` 语句引入条件控制流。
- **L330 EN**: Returns from the current function, often propagating a computed result.
  **L330 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L331 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L331 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L332 EN**: Comment documents intent or context: `Suppose we have at==2:`.
  **L332 CN**: 注释记录了意图或上下文：`Suppose we have at==2:`。
- **L333 EN**: Comment documents intent or context: `"THAT FORTRAN THAT I RAN" <- the string (x) in which we search`.
  **L333 CN**: 注释记录了意图或上下文：`"THAT FORTRAN THAT I RAN" <- the string (x) in which we search`。
- **L334 EN**: Comment documents intent or context: `"THAT I RAN" <- the string (want) for which we search`.
  **L334 CN**: 注释记录了意图或上下文：`"THAT I RAN" <- the string (want) for which we search`。
- **L335 EN**: Comment documents intent or context: `^------------------ j==7, ch=='T'`.
  **L335 CN**: 注释记录了意图或上下文：`^------------------ j==7, ch=='T'`。
- **L336 EN**: Comment documents intent or context: `We can shift ahead 3 positions to at==5 to align the 'T's:`.
  **L336 CN**: 注释记录了意图或上下文：`We can shift ahead 3 positions to at==5 to align the 'T's:`。
- **L337 EN**: Comment documents intent or context: `"THAT FORTRAN THAT I RAN"`.
  **L337 CN**: 注释记录了意图或上下文：`"THAT FORTRAN THAT I RAN"`。
- **L338 EN**: Comment documents intent or context: `"THAT I RAN"`.
  **L338 CN**: 注释记录了意图或上下文：`"THAT I RAN"`。
- **L339 EN**: Executes statement `std::size_t shift{1};`.
  **L339 CN**: 执行语句 `std::size_t shift{1};`。
- **L340 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L340 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L341 EN**: Introduces conditional control flow with an `if` statement.
  **L341 CN**: 通过 `if` 语句引入条件控制流。
- **L342 EN**: Breaks out of the current loop or switch.
  **L342 CN**: 跳出当前循环或 switch。
- **L343 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L343 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L344 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L344 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L345 EN**: Initializes or updates `+`.
  **L345 CN**: 初始化或更新 `+`。
- **L346 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L346 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L347 EN**: Returns from the current function, often propagating a computed result.
  **L347 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L348 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L348 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L349 EN**: Blank line separates nearby declarations or logic blocks.
  **L349 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L350 EN**: Comment documents intent or context: `SCAN and VERIFY implementation help. These intrinsic functions`.
  **L350 CN**: 注释记录了意图或上下文：`SCAN and VERIFY implementation help. These intrinsic functions`。
- **L351 EN**: Comment documents intent or context: `do pretty much the same thing, so they're templatized with a`.
  **L351 CN**: 注释记录了意图或上下文：`do pretty much the same thing, so they're templatized with a`。
- **L352 EN**: Comment documents intent or context: `distinguishing flag.`.
  **L352 CN**: 注释记录了意图或上下文：`distinguishing flag.`。

### Lines 353-384

````cpp

enum class CharFunc { Index, Scan, Verify };

template <typename CHAR, CharFunc FUNC>
inline RT_API_ATTRS std::size_t ScanVerify(const CHAR *x, std::size_t xLen,
    const CHAR *set, std::size_t setLen, bool back) {
  std::size_t at{back ? xLen : 1};
  int increment{back ? -1 : 1};
  for (; xLen-- > 0; at += increment) {
    CHAR ch{x[at - 1]};
    bool inSet{false};
    // TODO: If set is sorted, could use binary search
    for (std::size_t j{0}; j < setLen; ++j) {
      if (set[j] == ch) {
        inSet = true;
        break;
      }
    }
    if (inSet != (FUNC == CharFunc::Verify)) {
      return at;
    }
  }
  return 0;
}

// Specialization for one-byte characters
template <bool IS_VERIFY = false>
inline RT_API_ATTRS std::size_t ScanVerify(const char *x, std::size_t xLen,
    const char *set, std::size_t setLen, bool back) {
  std::size_t at{back ? xLen : 1};
  int increment{back ? -1 : 1};
  if (xLen > 0) {
````

- **L353 EN**: Blank line separates nearby declarations or logic blocks.
  **L353 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L354 EN**: Declares or defines enum class `CharFunc`.
  **L354 CN**: 声明或定义 enum class `CharFunc`。
- **L355 EN**: Blank line separates nearby declarations or logic blocks.
  **L355 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L356 EN**: Begins a template declaration parameterizing subsequent code.
  **L356 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L357 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L357 CN**: 延续周围的声明、表达式或控制流结构。
- **L358 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L358 CN**: 延续周围的声明、表达式或控制流结构。
- **L359 EN**: Executes statement `std::size_t at{back ? xLen : 1};`.
  **L359 CN**: 执行语句 `std::size_t at{back ? xLen : 1};`。
- **L360 EN**: Executes statement `int increment{back ? -1 : 1};`.
  **L360 CN**: 执行语句 `int increment{back ? -1 : 1};`。
- **L361 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L361 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L362 EN**: Executes statement `CHAR ch{x[at - 1]};`.
  **L362 CN**: 执行语句 `CHAR ch{x[at - 1]};`。
- **L363 EN**: Executes statement `bool inSet{false};`.
  **L363 CN**: 执行语句 `bool inSet{false};`。
- **L364 EN**: Comment documents intent or context: `TODO: If set is sorted, could use binary search`.
  **L364 CN**: 注释记录了意图或上下文：`TODO: If set is sorted, could use binary search`。
- **L365 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L365 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L366 EN**: Introduces conditional control flow with an `if` statement.
  **L366 CN**: 通过 `if` 语句引入条件控制流。
- **L367 EN**: Initializes or updates `inSet`.
  **L367 CN**: 初始化或更新 `inSet`。
- **L368 EN**: Breaks out of the current loop or switch.
  **L368 CN**: 跳出当前循环或 switch。
- **L369 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L369 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L370 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L370 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L371 EN**: Introduces conditional control flow with an `if` statement.
  **L371 CN**: 通过 `if` 语句引入条件控制流。
- **L372 EN**: Returns from the current function, often propagating a computed result.
  **L372 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L373 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L373 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L374 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L374 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L375 EN**: Returns from the current function, often propagating a computed result.
  **L375 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L376 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L376 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L377 EN**: Blank line separates nearby declarations or logic blocks.
  **L377 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L378 EN**: Comment documents intent or context: `Specialization for one-byte characters`.
  **L378 CN**: 注释记录了意图或上下文：`Specialization for one-byte characters`。
- **L379 EN**: Begins a template declaration parameterizing subsequent code.
  **L379 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L380 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L380 CN**: 延续周围的声明、表达式或控制流结构。
- **L381 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L381 CN**: 延续周围的声明、表达式或控制流结构。
- **L382 EN**: Executes statement `std::size_t at{back ? xLen : 1};`.
  **L382 CN**: 执行语句 `std::size_t at{back ? xLen : 1};`。
- **L383 EN**: Executes statement `int increment{back ? -1 : 1};`.
  **L383 CN**: 执行语句 `int increment{back ? -1 : 1};`。
- **L384 EN**: Introduces conditional control flow with an `if` statement.
  **L384 CN**: 通过 `if` 语句引入条件控制流。

### Lines 385-416

````cpp
    std::uint64_t bitSet[256 / 64]{0};
    std::uint64_t one{1};
    for (std::size_t j{0}; j < setLen; ++j) {
      unsigned setCh{static_cast<unsigned char>(set[j])};
      bitSet[setCh / 64] |= one << (setCh % 64);
    }
    for (; xLen-- > 0; at += increment) {
      unsigned ch{static_cast<unsigned char>(x[at - 1])};
      bool inSet{((bitSet[ch / 64] >> (ch % 64)) & 1) != 0};
      if (inSet != IS_VERIFY) {
        return at;
      }
    }
  }
  return 0;
}

template <typename INT, typename CHAR, CharFunc FUNC>
static RT_API_ATTRS void GeneralCharFunc(Descriptor &result,
    const Descriptor &string, const Descriptor &arg, const Descriptor *back,
    const Terminator &terminator) {
  int rank{string.rank() ? string.rank()
          : arg.rank()   ? arg.rank()
          : back         ? back->rank()
                         : 0};
  SubscriptValue ub[maxRank], stringAt[maxRank], argAt[maxRank],
      backAt[maxRank];
  SubscriptValue elements{1};
  for (int j{0}; j < rank; ++j) {
    ub[j] = string.rank() ? string.GetDimension(j).Extent()
        : arg.rank()      ? arg.GetDimension(j).Extent()
        : back            ? back->GetDimension(j).Extent()
````

- **L385 EN**: Executes statement `std::uint64_t bitSet[256 / 64]{0};`.
  **L385 CN**: 执行语句 `std::uint64_t bitSet[256 / 64]{0};`。
- **L386 EN**: Executes statement `std::uint64_t one{1};`.
  **L386 CN**: 执行语句 `std::uint64_t one{1};`。
- **L387 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L387 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L388 EN**: Executes statement `unsigned setCh{static_cast<unsigned char>(set[j])};`.
  **L388 CN**: 执行语句 `unsigned setCh{static_cast<unsigned char>(set[j])};`。
- **L389 EN**: Initializes or updates `|`.
  **L389 CN**: 初始化或更新 `|`。
- **L390 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L390 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L391 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L391 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L392 EN**: Executes statement `unsigned ch{static_cast<unsigned char>(x[at - 1])};`.
  **L392 CN**: 执行语句 `unsigned ch{static_cast<unsigned char>(x[at - 1])};`。
- **L393 EN**: Executes statement `bool inSet{((bitSet[ch / 64] >> (ch % 64)) & 1) != 0};`.
  **L393 CN**: 执行语句 `bool inSet{((bitSet[ch / 64] >> (ch % 64)) & 1) != 0};`。
- **L394 EN**: Introduces conditional control flow with an `if` statement.
  **L394 CN**: 通过 `if` 语句引入条件控制流。
- **L395 EN**: Returns from the current function, often propagating a computed result.
  **L395 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L396 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L396 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L397 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L397 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L398 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L398 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L399 EN**: Returns from the current function, often propagating a computed result.
  **L399 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L400 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L400 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L401 EN**: Blank line separates nearby declarations or logic blocks.
  **L401 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L402 EN**: Begins a template declaration parameterizing subsequent code.
  **L402 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L403 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L403 CN**: 延续周围的声明、表达式或控制流结构。
- **L404 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L404 CN**: 延续周围的声明、表达式或控制流结构。
- **L405 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L405 CN**: 延续周围的声明、表达式或控制流结构。
- **L406 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L406 CN**: 延续周围的声明、表达式或控制流结构。
- **L407 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L407 CN**: 延续周围的声明、表达式或控制流结构。
- **L408 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L408 CN**: 延续周围的声明、表达式或控制流结构。
- **L409 EN**: Executes statement `: 0};`.
  **L409 CN**: 执行语句 `: 0};`。
- **L410 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L410 CN**: 延续周围的声明、表达式或控制流结构。
- **L411 EN**: Executes statement `backAt[maxRank];`.
  **L411 CN**: 执行语句 `backAt[maxRank];`。
- **L412 EN**: Executes statement `SubscriptValue elements{1};`.
  **L412 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L413 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L413 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L414 EN**: Initializes or updates `ub[j]`.
  **L414 CN**: 初始化或更新 `ub[j]`。
- **L415 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L415 CN**: 延续周围的声明、表达式或控制流结构。
- **L416 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L416 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 417-448

````cpp
                          : 1;
    elements *= ub[j];
  }
  string.GetLowerBounds(stringAt);
  arg.GetLowerBounds(argAt);
  if (back) {
    back->GetLowerBounds(backAt);
  }
  result.Establish(TypeCategory::Integer, sizeof(INT), nullptr, rank, ub,
      CFI_attribute_allocatable);
  for (int j{0}; j < rank; ++j) {
    result.GetDimension(j).SetBounds(1, ub[j]);
  }
  if (result.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash(
        "INDEX/SCAN/VERIFY: could not allocate storage for result");
  }
  std::size_t stringElementChars{string.ElementBytes() >> shift<CHAR>};
  std::size_t argElementChars{arg.ElementBytes() >> shift<CHAR>};
  for (SubscriptValue resultAt{0}; elements-- > 0; resultAt += sizeof(INT),
       string.IncrementSubscripts(stringAt), arg.IncrementSubscripts(argAt),
       back && back->IncrementSubscripts(backAt)) {
    if constexpr (FUNC == CharFunc::Index) {
      *result.OffsetElement<INT>(resultAt) =
          Index<CHAR>(string.Element<CHAR>(stringAt), stringElementChars,
              arg.Element<CHAR>(argAt), argElementChars,
              back && IsLogicalElementTrue(*back, backAt));
    } else if constexpr (FUNC == CharFunc::Scan) {
      *result.OffsetElement<INT>(resultAt) =
          ScanVerify<CHAR, CharFunc::Scan>(string.Element<CHAR>(stringAt),
              stringElementChars, arg.Element<CHAR>(argAt), argElementChars,
              back && IsLogicalElementTrue(*back, backAt));
````

- **L417 EN**: Executes statement `: 1;`.
  **L417 CN**: 执行语句 `: 1;`。
- **L418 EN**: Initializes or updates `*`.
  **L418 CN**: 初始化或更新 `*`。
- **L419 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L419 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L420 EN**: Executes statement involving `GetLowerBounds`.
  **L420 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L421 EN**: Executes statement involving `GetLowerBounds`.
  **L421 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L422 EN**: Introduces conditional control flow with an `if` statement.
  **L422 CN**: 通过 `if` 语句引入条件控制流。
- **L423 EN**: Executes statement involving `GetLowerBounds`.
  **L423 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L424 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L424 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L425 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L425 CN**: 延续周围的声明、表达式或控制流结构。
- **L426 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L426 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L427 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L427 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L428 EN**: Executes statement involving `GetDimension`.
  **L428 CN**: 执行涉及 `GetDimension` 的语句。
- **L429 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L429 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L430 EN**: Introduces conditional control flow with an `if` statement.
  **L430 CN**: 通过 `if` 语句引入条件控制流。
- **L431 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L431 CN**: 延续周围的声明、表达式或控制流结构。
- **L432 EN**: Executes statement `"INDEX/SCAN/VERIFY: could not allocate storage for result");`.
  **L432 CN**: 执行语句 `"INDEX/SCAN/VERIFY: could not allocate storage for result");`。
- **L433 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L433 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L434 EN**: Executes statement involving `ElementBytes`.
  **L434 CN**: 执行涉及 `ElementBytes` 的语句。
- **L435 EN**: Executes statement involving `ElementBytes`.
  **L435 CN**: 执行涉及 `ElementBytes` 的语句。
- **L436 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L436 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L437 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L437 CN**: 延续周围的声明、表达式或控制流结构。
- **L438 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L438 CN**: 延续周围的声明、表达式或控制流结构。
- **L439 EN**: Introduces conditional control flow with an `if` statement.
  **L439 CN**: 通过 `if` 语句引入条件控制流。
- **L440 EN**: Comment documents intent or context: `result.OffsetElement<INT>(resultAt) =`.
  **L440 CN**: 注释记录了意图或上下文：`result.OffsetElement<INT>(resultAt) =`。
- **L441 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L441 CN**: 延续周围的声明、表达式或控制流结构。
- **L442 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L442 CN**: 延续周围的声明、表达式或控制流结构。
- **L443 EN**: Executes statement involving `IsLogicalElementTrue`.
  **L443 CN**: 执行涉及 `IsLogicalElementTrue` 的语句。
- **L444 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L444 CN**: 延续周围的声明、表达式或控制流结构。
- **L445 EN**: Comment documents intent or context: `result.OffsetElement<INT>(resultAt) =`.
  **L445 CN**: 注释记录了意图或上下文：`result.OffsetElement<INT>(resultAt) =`。
- **L446 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L446 CN**: 延续周围的声明、表达式或控制流结构。
- **L447 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L447 CN**: 延续周围的声明、表达式或控制流结构。
- **L448 EN**: Executes statement involving `IsLogicalElementTrue`.
  **L448 CN**: 执行涉及 `IsLogicalElementTrue` 的语句。

### Lines 449-480

````cpp
    } else if constexpr (FUNC == CharFunc::Verify) {
      *result.OffsetElement<INT>(resultAt) =
          ScanVerify<CHAR, CharFunc::Verify>(string.Element<CHAR>(stringAt),
              stringElementChars, arg.Element<CHAR>(argAt), argElementChars,
              back && IsLogicalElementTrue(*back, backAt));
    } else {
      static_assert(FUNC == CharFunc::Index || FUNC == CharFunc::Scan ||
          FUNC == CharFunc::Verify);
    }
  }
}

template <typename CHAR, CharFunc FUNC>
static RT_API_ATTRS void GeneralCharFuncKind(Descriptor &result,
    const Descriptor &string, const Descriptor &arg, const Descriptor *back,
    int kind, const Terminator &terminator) {
  switch (kind) {
  case 1:
    GeneralCharFunc<CppTypeFor<TypeCategory::Integer, 1>, CHAR, FUNC>(
        result, string, arg, back, terminator);
    break;
  case 2:
    GeneralCharFunc<CppTypeFor<TypeCategory::Integer, 2>, CHAR, FUNC>(
        result, string, arg, back, terminator);
    break;
  case 4:
    GeneralCharFunc<CppTypeFor<TypeCategory::Integer, 4>, CHAR, FUNC>(
        result, string, arg, back, terminator);
    break;
  case 8:
    GeneralCharFunc<CppTypeFor<TypeCategory::Integer, 8>, CHAR, FUNC>(
        result, string, arg, back, terminator);
````

- **L449 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L449 CN**: 延续周围的声明、表达式或控制流结构。
- **L450 EN**: Comment documents intent or context: `result.OffsetElement<INT>(resultAt) =`.
  **L450 CN**: 注释记录了意图或上下文：`result.OffsetElement<INT>(resultAt) =`。
- **L451 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L451 CN**: 延续周围的声明、表达式或控制流结构。
- **L452 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L452 CN**: 延续周围的声明、表达式或控制流结构。
- **L453 EN**: Executes statement involving `IsLogicalElementTrue`.
  **L453 CN**: 执行涉及 `IsLogicalElementTrue` 的语句。
- **L454 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L454 CN**: 延续周围的声明、表达式或控制流结构。
- **L455 EN**: Performs a compile-time assertion to enforce invariants.
  **L455 CN**: 执行编译期断言以约束不变量。
- **L456 EN**: Initializes or updates `FUNC`.
  **L456 CN**: 初始化或更新 `FUNC`。
- **L457 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L457 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L458 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L458 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L459 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L459 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L460 EN**: Blank line separates nearby declarations or logic blocks.
  **L460 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L461 EN**: Begins a template declaration parameterizing subsequent code.
  **L461 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L462 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L462 CN**: 延续周围的声明、表达式或控制流结构。
- **L463 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L463 CN**: 延续周围的声明、表达式或控制流结构。
- **L464 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L464 CN**: 延续周围的声明、表达式或控制流结构。
- **L465 EN**: Begins a `switch` dispatch over discrete cases.
  **L465 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L466 EN**: Marks one `switch` case label.
  **L466 CN**: 标记一个 `switch` 的 case 标签。
- **L467 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L467 CN**: 延续周围的声明、表达式或控制流结构。
- **L468 EN**: Executes statement `result, string, arg, back, terminator);`.
  **L468 CN**: 执行语句 `result, string, arg, back, terminator);`。
- **L469 EN**: Breaks out of the current loop or switch.
  **L469 CN**: 跳出当前循环或 switch。
- **L470 EN**: Marks one `switch` case label.
  **L470 CN**: 标记一个 `switch` 的 case 标签。
- **L471 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L471 CN**: 延续周围的声明、表达式或控制流结构。
- **L472 EN**: Executes statement `result, string, arg, back, terminator);`.
  **L472 CN**: 执行语句 `result, string, arg, back, terminator);`。
- **L473 EN**: Breaks out of the current loop or switch.
  **L473 CN**: 跳出当前循环或 switch。
- **L474 EN**: Marks one `switch` case label.
  **L474 CN**: 标记一个 `switch` 的 case 标签。
- **L475 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L475 CN**: 延续周围的声明、表达式或控制流结构。
- **L476 EN**: Executes statement `result, string, arg, back, terminator);`.
  **L476 CN**: 执行语句 `result, string, arg, back, terminator);`。
- **L477 EN**: Breaks out of the current loop or switch.
  **L477 CN**: 跳出当前循环或 switch。
- **L478 EN**: Marks one `switch` case label.
  **L478 CN**: 标记一个 `switch` 的 case 标签。
- **L479 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L479 CN**: 延续周围的声明、表达式或控制流结构。
- **L480 EN**: Executes statement `result, string, arg, back, terminator);`.
  **L480 CN**: 执行语句 `result, string, arg, back, terminator);`。

### Lines 481-512

````cpp
    break;
  case 16:
    GeneralCharFunc<CppTypeFor<TypeCategory::Integer, 16>, CHAR, FUNC>(
        result, string, arg, back, terminator);
    break;
  default:
    terminator.Crash("not yet implemented: CHARACTER(KIND=%d) in "
                     "INDEX/SCAN/VERIFY intrinsic",
        kind);
  }
}

template <typename CHAR, bool ISMIN>
static RT_API_ATTRS void MaxMinHelper(Descriptor &accumulator,
    const Descriptor &x, const Terminator &terminator) {
  RUNTIME_CHECK(terminator,
      accumulator.rank() == 0 || x.rank() == 0 ||
          accumulator.rank() == x.rank());
  SubscriptValue ub[maxRank], xAt[maxRank];
  SubscriptValue elements{1};
  std::size_t accumChars{accumulator.ElementBytes() >> shift<CHAR>};
  std::size_t xChars{x.ElementBytes() >> shift<CHAR>};
  std::size_t chars{std::max(accumChars, xChars)};
  bool reallocate{accumulator.raw().base_addr == nullptr ||
      accumChars != chars || (accumulator.rank() == 0 && x.rank() > 0)};
  int rank{std::max(accumulator.rank(), x.rank())};
  for (int j{0}; j < rank; ++j) {
    if (x.rank() > 0) {
      ub[j] = x.GetDimension(j).Extent();
      if (accumulator.rank() > 0) {
        SubscriptValue accumExt{accumulator.GetDimension(j).Extent()};
        if (accumExt != ub[j]) {
````

- **L481 EN**: Breaks out of the current loop or switch.
  **L481 CN**: 跳出当前循环或 switch。
- **L482 EN**: Marks one `switch` case label.
  **L482 CN**: 标记一个 `switch` 的 case 标签。
- **L483 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L483 CN**: 延续周围的声明、表达式或控制流结构。
- **L484 EN**: Executes statement `result, string, arg, back, terminator);`.
  **L484 CN**: 执行语句 `result, string, arg, back, terminator);`。
- **L485 EN**: Breaks out of the current loop or switch.
  **L485 CN**: 跳出当前循环或 switch。
- **L486 EN**: Provides the default branch for a `switch` statement.
  **L486 CN**: 为 `switch` 语句提供默认分支。
- **L487 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L487 CN**: 延续周围的声明、表达式或控制流结构。
- **L488 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L488 CN**: 延续周围的声明、表达式或控制流结构。
- **L489 EN**: Executes statement `kind);`.
  **L489 CN**: 执行语句 `kind);`。
- **L490 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L490 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L491 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L491 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L492 EN**: Blank line separates nearby declarations or logic blocks.
  **L492 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L493 EN**: Begins a template declaration parameterizing subsequent code.
  **L493 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L494 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L494 CN**: 延续周围的声明、表达式或控制流结构。
- **L495 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L495 CN**: 延续周围的声明、表达式或控制流结构。
- **L496 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L496 CN**: 延续周围的声明、表达式或控制流结构。
- **L497 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L497 CN**: 延续周围的声明、表达式或控制流结构。
- **L498 EN**: Executes statement involving `rank`.
  **L498 CN**: 执行涉及 `rank` 的语句。
- **L499 EN**: Executes statement `SubscriptValue ub[maxRank], xAt[maxRank];`.
  **L499 CN**: 执行语句 `SubscriptValue ub[maxRank], xAt[maxRank];`。
- **L500 EN**: Executes statement `SubscriptValue elements{1};`.
  **L500 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L501 EN**: Executes statement involving `ElementBytes`.
  **L501 CN**: 执行涉及 `ElementBytes` 的语句。
- **L502 EN**: Executes statement involving `ElementBytes`.
  **L502 CN**: 执行涉及 `ElementBytes` 的语句。
- **L503 EN**: Executes statement involving `max`.
  **L503 CN**: 执行涉及 `max` 的语句。
- **L504 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L504 CN**: 延续周围的声明、表达式或控制流结构。
- **L505 EN**: Executes statement involving `rank`.
  **L505 CN**: 执行涉及 `rank` 的语句。
- **L506 EN**: Executes statement involving `max`.
  **L506 CN**: 执行涉及 `max` 的语句。
- **L507 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L507 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L508 EN**: Introduces conditional control flow with an `if` statement.
  **L508 CN**: 通过 `if` 语句引入条件控制流。
- **L509 EN**: Initializes or updates `ub[j]`.
  **L509 CN**: 初始化或更新 `ub[j]`。
- **L510 EN**: Introduces conditional control flow with an `if` statement.
  **L510 CN**: 通过 `if` 语句引入条件控制流。
- **L511 EN**: Executes statement involving `GetDimension`.
  **L511 CN**: 执行涉及 `GetDimension` 的语句。
- **L512 EN**: Introduces conditional control flow with an `if` statement.
  **L512 CN**: 通过 `if` 语句引入条件控制流。

### Lines 513-544

````cpp
          terminator.Crash("Character MAX/MIN: operands are not "
                           "conforming on dimension %d (%jd != %jd)",
              j + 1, static_cast<std::intmax_t>(accumExt),
              static_cast<std::intmax_t>(ub[j]));
        }
      }
    } else {
      ub[j] = accumulator.GetDimension(j).Extent();
    }
    elements *= ub[j];
  }
  x.GetLowerBounds(xAt);
  void *old{nullptr};
  const CHAR *accumData{accumulator.OffsetElement<CHAR>()};
  if (reallocate) {
    old = accumulator.raw().base_addr;
    accumulator.set_base_addr(nullptr);
    accumulator.raw().elem_len = chars << shift<CHAR>;
    for (int j{0}; j < rank; ++j) {
      accumulator.GetDimension(j).SetBounds(1, ub[j]);
    }
    RUNTIME_CHECK(
        terminator, accumulator.Allocate(kNoAsyncObject) == CFI_SUCCESS);
  }
  for (CHAR *result{accumulator.OffsetElement<CHAR>()}; elements-- > 0;
       accumData += accumChars, result += chars, x.IncrementSubscripts(xAt)) {
    const CHAR *xData{x.Element<CHAR>(xAt)};
    int cmp{CharacterScalarCompare(accumData, xData, accumChars, xChars)};
    if constexpr (ISMIN) {
      cmp = -cmp;
    }
    if (cmp < 0) {
````

- **L513 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L513 CN**: 延续周围的声明、表达式或控制流结构。
- **L514 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L514 CN**: 延续周围的声明、表达式或控制流结构。
- **L515 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L515 CN**: 延续周围的声明、表达式或控制流结构。
- **L516 EN**: Executes statement `static_cast<std::intmax_t>(ub[j]));`.
  **L516 CN**: 执行语句 `static_cast<std::intmax_t>(ub[j]));`。
- **L517 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L517 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L518 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L518 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L519 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L519 CN**: 延续周围的声明、表达式或控制流结构。
- **L520 EN**: Initializes or updates `ub[j]`.
  **L520 CN**: 初始化或更新 `ub[j]`。
- **L521 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L521 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L522 EN**: Initializes or updates `*`.
  **L522 CN**: 初始化或更新 `*`。
- **L523 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L523 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L524 EN**: Executes statement involving `GetLowerBounds`.
  **L524 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L525 EN**: Executes statement `void *old{nullptr};`.
  **L525 CN**: 执行语句 `void *old{nullptr};`。
- **L526 EN**: Executes statement `const CHAR *accumData{accumulator.OffsetElement<CHAR>()};`.
  **L526 CN**: 执行语句 `const CHAR *accumData{accumulator.OffsetElement<CHAR>()};`。
- **L527 EN**: Introduces conditional control flow with an `if` statement.
  **L527 CN**: 通过 `if` 语句引入条件控制流。
- **L528 EN**: Initializes or updates `old`.
  **L528 CN**: 初始化或更新 `old`。
- **L529 EN**: Executes statement involving `set_base_addr`.
  **L529 CN**: 执行涉及 `set_base_addr` 的语句。
- **L530 EN**: Initializes or updates `accumulator.raw().elem_len`.
  **L530 CN**: 初始化或更新 `accumulator.raw().elem_len`。
- **L531 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L531 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L532 EN**: Executes statement involving `GetDimension`.
  **L532 CN**: 执行涉及 `GetDimension` 的语句。
- **L533 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L533 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L534 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L534 CN**: 延续周围的声明、表达式或控制流结构。
- **L535 EN**: Executes statement involving `Allocate`.
  **L535 CN**: 执行涉及 `Allocate` 的语句。
- **L536 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L536 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L537 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L537 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L538 EN**: Initializes or updates `+`.
  **L538 CN**: 初始化或更新 `+`。
- **L539 EN**: Executes statement `const CHAR *xData{x.Element<CHAR>(xAt)};`.
  **L539 CN**: 执行语句 `const CHAR *xData{x.Element<CHAR>(xAt)};`。
- **L540 EN**: Executes statement involving `CharacterScalarCompare`.
  **L540 CN**: 执行涉及 `CharacterScalarCompare` 的语句。
- **L541 EN**: Introduces conditional control flow with an `if` statement.
  **L541 CN**: 通过 `if` 语句引入条件控制流。
- **L542 EN**: Initializes or updates `cmp`.
  **L542 CN**: 初始化或更新 `cmp`。
- **L543 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L543 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L544 EN**: Introduces conditional control flow with an `if` statement.
  **L544 CN**: 通过 `if` 语句引入条件控制流。

### Lines 545-576

````cpp
      CopyAndPad(result, xData, chars, xChars);
    } else if (result != accumData) {
      CopyAndPad(result, accumData, chars, accumChars);
    }
  }
  FreeMemory(old);
}

template <bool ISMIN>
static RT_API_ATTRS void MaxMin(Descriptor &accumulator, const Descriptor &x,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  RUNTIME_CHECK(terminator, accumulator.raw().type == x.raw().type);
  switch (accumulator.raw().type) {
  case CFI_type_char:
    MaxMinHelper<char, ISMIN>(accumulator, x, terminator);
    break;
  case CFI_type_char16_t:
    MaxMinHelper<char16_t, ISMIN>(accumulator, x, terminator);
    break;
  case CFI_type_char32_t:
    MaxMinHelper<char32_t, ISMIN>(accumulator, x, terminator);
    break;
  default:
    terminator.Crash(
        "Character MAX/MIN: result does not have a character type");
  }
}

template <typename CHAR>
static inline RT_API_ATTRS bool TokenizeIsInSet(
    CHAR ch, const CHAR *set, std::size_t setChars) {
````

- **L545 EN**: Executes statement involving `CopyAndPad`.
  **L545 CN**: 执行涉及 `CopyAndPad` 的语句。
- **L546 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L546 CN**: 延续周围的声明、表达式或控制流结构。
- **L547 EN**: Executes statement involving `CopyAndPad`.
  **L547 CN**: 执行涉及 `CopyAndPad` 的语句。
- **L548 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L548 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L549 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L549 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L550 EN**: Executes statement involving `FreeMemory`.
  **L550 CN**: 执行涉及 `FreeMemory` 的语句。
- **L551 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L551 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L552 EN**: Blank line separates nearby declarations or logic blocks.
  **L552 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L553 EN**: Begins a template declaration parameterizing subsequent code.
  **L553 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L554 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L554 CN**: 延续周围的声明、表达式或控制流结构。
- **L555 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L555 CN**: 延续周围的声明、表达式或控制流结构。
- **L556 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L556 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L557 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L557 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L558 EN**: Begins a `switch` dispatch over discrete cases.
  **L558 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L559 EN**: Marks one `switch` case label.
  **L559 CN**: 标记一个 `switch` 的 case 标签。
- **L560 EN**: Executes statement `MaxMinHelper<char, ISMIN>(accumulator, x, terminator);`.
  **L560 CN**: 执行语句 `MaxMinHelper<char, ISMIN>(accumulator, x, terminator);`。
- **L561 EN**: Breaks out of the current loop or switch.
  **L561 CN**: 跳出当前循环或 switch。
- **L562 EN**: Marks one `switch` case label.
  **L562 CN**: 标记一个 `switch` 的 case 标签。
- **L563 EN**: Executes statement `MaxMinHelper<char16_t, ISMIN>(accumulator, x, terminator);`.
  **L563 CN**: 执行语句 `MaxMinHelper<char16_t, ISMIN>(accumulator, x, terminator);`。
- **L564 EN**: Breaks out of the current loop or switch.
  **L564 CN**: 跳出当前循环或 switch。
- **L565 EN**: Marks one `switch` case label.
  **L565 CN**: 标记一个 `switch` 的 case 标签。
- **L566 EN**: Executes statement `MaxMinHelper<char32_t, ISMIN>(accumulator, x, terminator);`.
  **L566 CN**: 执行语句 `MaxMinHelper<char32_t, ISMIN>(accumulator, x, terminator);`。
- **L567 EN**: Breaks out of the current loop or switch.
  **L567 CN**: 跳出当前循环或 switch。
- **L568 EN**: Provides the default branch for a `switch` statement.
  **L568 CN**: 为 `switch` 语句提供默认分支。
- **L569 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L569 CN**: 延续周围的声明、表达式或控制流结构。
- **L570 EN**: Executes statement `"Character MAX/MIN: result does not have a character type");`.
  **L570 CN**: 执行语句 `"Character MAX/MIN: result does not have a character type");`。
- **L571 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L571 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L572 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L572 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L573 EN**: Blank line separates nearby declarations or logic blocks.
  **L573 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L574 EN**: Begins a template declaration parameterizing subsequent code.
  **L574 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L575 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L575 CN**: 延续周围的声明、表达式或控制流结构。
- **L576 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L576 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 577-608

````cpp
  for (std::size_t j{0}; j < setChars; ++j) {
    if (set[j] == ch) {
      return true;
    }
  }
  return false;
}

// Pad the token with spaces.
template <typename CHAR>
static inline RT_API_ATTRS void TokenizeFillBlanks(
    CHAR *to, std::size_t chars) {
  if (chars == 0) {
    return;
  }
  if constexpr (std::is_same_v<CHAR, char>) {
    runtime::memset(to, ' ', chars);
  } else {
    for (std::size_t j{0}; j < chars; ++j) {
      to[j] = static_cast<CHAR>(' ');
    }
  }
}

struct TokenizeAnalysis {
  std::size_t tokenCount{0};
  std::size_t maxTokenLen{0}; // in characters
};

template <typename CHAR>
static RT_API_ATTRS TokenizeAnalysis AnalyzeTokenize(const CHAR *str,
    std::size_t strChars, const CHAR *set, std::size_t setChars) {
````

- **L577 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L577 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L578 EN**: Introduces conditional control flow with an `if` statement.
  **L578 CN**: 通过 `if` 语句引入条件控制流。
- **L579 EN**: Returns from the current function, often propagating a computed result.
  **L579 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L580 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L580 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L581 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L581 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L582 EN**: Returns from the current function, often propagating a computed result.
  **L582 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L583 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L583 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L584 EN**: Blank line separates nearby declarations or logic blocks.
  **L584 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L585 EN**: Comment documents intent or context: `Pad the token with spaces.`.
  **L585 CN**: 注释记录了意图或上下文：`Pad the token with spaces.`。
- **L586 EN**: Begins a template declaration parameterizing subsequent code.
  **L586 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L587 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L587 CN**: 延续周围的声明、表达式或控制流结构。
- **L588 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L588 CN**: 延续周围的声明、表达式或控制流结构。
- **L589 EN**: Introduces conditional control flow with an `if` statement.
  **L589 CN**: 通过 `if` 语句引入条件控制流。
- **L590 EN**: Returns from the current function, often propagating a computed result.
  **L590 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L591 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L591 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L592 EN**: Introduces conditional control flow with an `if` statement.
  **L592 CN**: 通过 `if` 语句引入条件控制流。
- **L593 EN**: Executes statement involving `memset`.
  **L593 CN**: 执行涉及 `memset` 的语句。
- **L594 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L594 CN**: 延续周围的声明、表达式或控制流结构。
- **L595 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L595 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L596 EN**: Initializes or updates `to[j]`.
  **L596 CN**: 初始化或更新 `to[j]`。
- **L597 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L597 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L598 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L598 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L599 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L599 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L600 EN**: Blank line separates nearby declarations or logic blocks.
  **L600 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L601 EN**: Declares or defines struct `TokenizeAnalysis`.
  **L601 CN**: 声明或定义 struct `TokenizeAnalysis`。
- **L602 EN**: Executes statement `std::size_t tokenCount{0};`.
  **L602 CN**: 执行语句 `std::size_t tokenCount{0};`。
- **L603 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L603 CN**: 延续周围的声明、表达式或控制流结构。
- **L604 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L604 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L605 EN**: Blank line separates nearby declarations or logic blocks.
  **L605 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L606 EN**: Begins a template declaration parameterizing subsequent code.
  **L606 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L607 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L607 CN**: 延续周围的声明、表达式或控制流结构。
- **L608 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L608 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 609-640

````cpp
  TokenizeAnalysis analysis;

  // Empty STRING should return one empty token, per Fortran standard.
  if (strChars == 0) {
    analysis.tokenCount = 1;
    analysis.maxTokenLen = 0;
    return analysis;
  }
  if (setChars == 0) {
    analysis.tokenCount = 1;
    analysis.maxTokenLen = strChars;
    return analysis;
  }

  // Split STRING at each delimiter character. This produces empty tokens
  // when delimiters are consecutive or when STRING starts/ends with a
  // delimiter.
  std::size_t tokenStart{0};
  for (std::size_t pos{0}; pos < strChars; ++pos) {
    if (TokenizeIsInSet(str[pos], set, setChars)) {
      analysis.maxTokenLen = std::max(analysis.maxTokenLen, pos - tokenStart);
      analysis.tokenCount++;
      tokenStart = pos + 1;
    }
  }
  analysis.maxTokenLen = std::max(analysis.maxTokenLen, strChars - tokenStart);
  analysis.tokenCount++;

  return analysis;
}
// Allocates and populates the result arrays for TOKENIZE Form 1.
template <typename CHAR>
````

- **L609 EN**: Executes statement `TokenizeAnalysis analysis;`.
  **L609 CN**: 执行语句 `TokenizeAnalysis analysis;`。
- **L610 EN**: Blank line separates nearby declarations or logic blocks.
  **L610 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L611 EN**: Comment documents intent or context: `Empty STRING should return one empty token, per Fortran standard.`.
  **L611 CN**: 注释记录了意图或上下文：`Empty STRING should return one empty token, per Fortran standard.`。
- **L612 EN**: Introduces conditional control flow with an `if` statement.
  **L612 CN**: 通过 `if` 语句引入条件控制流。
- **L613 EN**: Initializes or updates `analysis.tokenCount`.
  **L613 CN**: 初始化或更新 `analysis.tokenCount`。
- **L614 EN**: Initializes or updates `analysis.maxTokenLen`.
  **L614 CN**: 初始化或更新 `analysis.maxTokenLen`。
- **L615 EN**: Returns from the current function, often propagating a computed result.
  **L615 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L616 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L616 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L617 EN**: Introduces conditional control flow with an `if` statement.
  **L617 CN**: 通过 `if` 语句引入条件控制流。
- **L618 EN**: Initializes or updates `analysis.tokenCount`.
  **L618 CN**: 初始化或更新 `analysis.tokenCount`。
- **L619 EN**: Initializes or updates `analysis.maxTokenLen`.
  **L619 CN**: 初始化或更新 `analysis.maxTokenLen`。
- **L620 EN**: Returns from the current function, often propagating a computed result.
  **L620 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L621 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L621 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L622 EN**: Blank line separates nearby declarations or logic blocks.
  **L622 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L623 EN**: Comment documents intent or context: `Split STRING at each delimiter character. This produces empty tokens`.
  **L623 CN**: 注释记录了意图或上下文：`Split STRING at each delimiter character. This produces empty tokens`。
- **L624 EN**: Comment documents intent or context: `when delimiters are consecutive or when STRING starts/ends with a`.
  **L624 CN**: 注释记录了意图或上下文：`when delimiters are consecutive or when STRING starts/ends with a`。
- **L625 EN**: Comment documents intent or context: `delimiter.`.
  **L625 CN**: 注释记录了意图或上下文：`delimiter.`。
- **L626 EN**: Executes statement `std::size_t tokenStart{0};`.
  **L626 CN**: 执行语句 `std::size_t tokenStart{0};`。
- **L627 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L627 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L628 EN**: Introduces conditional control flow with an `if` statement.
  **L628 CN**: 通过 `if` 语句引入条件控制流。
- **L629 EN**: Initializes or updates `analysis.maxTokenLen`.
  **L629 CN**: 初始化或更新 `analysis.maxTokenLen`。
- **L630 EN**: Executes statement `analysis.tokenCount++;`.
  **L630 CN**: 执行语句 `analysis.tokenCount++;`。
- **L631 EN**: Initializes or updates `tokenStart`.
  **L631 CN**: 初始化或更新 `tokenStart`。
- **L632 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L632 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L633 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L633 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L634 EN**: Initializes or updates `analysis.maxTokenLen`.
  **L634 CN**: 初始化或更新 `analysis.maxTokenLen`。
- **L635 EN**: Executes statement `analysis.tokenCount++;`.
  **L635 CN**: 执行语句 `analysis.tokenCount++;`。
- **L636 EN**: Blank line separates nearby declarations or logic blocks.
  **L636 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L637 EN**: Returns from the current function, often propagating a computed result.
  **L637 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L638 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L638 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L639 EN**: Comment documents intent or context: `Allocates and populates the result arrays for TOKENIZE Form 1.`.
  **L639 CN**: 注释记录了意图或上下文：`Allocates and populates the result arrays for TOKENIZE Form 1.`。
- **L640 EN**: Begins a template declaration parameterizing subsequent code.
  **L640 CN**: 开始一个模板声明，为后续代码提供参数化能力。

### Lines 641-672

````cpp
static RT_API_ATTRS void TokenizeFillForm1(Descriptor &tokens,
    Descriptor *separator, const Descriptor &string, const CHAR *str,
    std::size_t strChars, const CHAR *set, std::size_t setChars,
    const TokenizeAnalysis &analysis, Terminator &terminator) {

  // (Re)allocate TOKENS.
  if (tokens.IsAllocated()) {
    tokens.Deallocate();
  }
  SubscriptValue tokensExtent[1]{
      static_cast<SubscriptValue>(analysis.tokenCount)};
  std::size_t tokenElemBytes{
      analysis.tokenCount == 0 ? 0 : analysis.maxTokenLen * sizeof(CHAR)};
  tokens.Establish(string.type(), tokenElemBytes, nullptr, 1, tokensExtent,
      CFI_attribute_allocatable);
  tokens.GetDimension(0).SetBounds(1, tokensExtent[0]);
  if (tokens.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("TOKENIZE: could not allocate TOKENS array");
  }

  // (Re)allocate SEPARATOR if present.
  std::size_t sepCount{analysis.tokenCount > 0 ? analysis.tokenCount - 1 : 0};
  std::size_t sepElemBytes{sizeof(CHAR)};
  if (separator) {
    if (separator->IsAllocated()) {
      separator->Deallocate();
    }
    SubscriptValue sepExtent[1]{static_cast<SubscriptValue>(sepCount)};
    separator->Establish(string.type(), sepElemBytes, nullptr, 1, sepExtent,
        CFI_attribute_allocatable);
    separator->GetDimension(0).SetBounds(1, sepExtent[0]);
    if (separator->Allocate(kNoAsyncObject) != CFI_SUCCESS) {
````

- **L641 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L641 CN**: 延续周围的声明、表达式或控制流结构。
- **L642 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L642 CN**: 延续周围的声明、表达式或控制流结构。
- **L643 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L643 CN**: 延续周围的声明、表达式或控制流结构。
- **L644 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L644 CN**: 延续周围的声明、表达式或控制流结构。
- **L645 EN**: Blank line separates nearby declarations or logic blocks.
  **L645 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L646 EN**: Comment documents intent or context: `(Re)allocate TOKENS.`.
  **L646 CN**: 注释记录了意图或上下文：`(Re)allocate TOKENS.`。
- **L647 EN**: Introduces conditional control flow with an `if` statement.
  **L647 CN**: 通过 `if` 语句引入条件控制流。
- **L648 EN**: Executes statement involving `Deallocate`.
  **L648 CN**: 执行涉及 `Deallocate` 的语句。
- **L649 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L649 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L650 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L650 CN**: 延续周围的声明、表达式或控制流结构。
- **L651 EN**: Executes statement `static_cast<SubscriptValue>(analysis.tokenCount)};`.
  **L651 CN**: 执行语句 `static_cast<SubscriptValue>(analysis.tokenCount)};`。
- **L652 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L652 CN**: 延续周围的声明、表达式或控制流结构。
- **L653 EN**: Executes statement involving `sizeof`.
  **L653 CN**: 执行涉及 `sizeof` 的语句。
- **L654 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L654 CN**: 延续周围的声明、表达式或控制流结构。
- **L655 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L655 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L656 EN**: Executes statement involving `GetDimension`.
  **L656 CN**: 执行涉及 `GetDimension` 的语句。
- **L657 EN**: Introduces conditional control flow with an `if` statement.
  **L657 CN**: 通过 `if` 语句引入条件控制流。
- **L658 EN**: Executes statement involving `Crash`.
  **L658 CN**: 执行涉及 `Crash` 的语句。
- **L659 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L659 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L660 EN**: Blank line separates nearby declarations or logic blocks.
  **L660 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L661 EN**: Comment documents intent or context: `(Re)allocate SEPARATOR if present.`.
  **L661 CN**: 注释记录了意图或上下文：`(Re)allocate SEPARATOR if present.`。
- **L662 EN**: Executes statement `std::size_t sepCount{analysis.tokenCount > 0 ? analysis.tokenCount - 1 : 0};`.
  **L662 CN**: 执行语句 `std::size_t sepCount{analysis.tokenCount > 0 ? analysis.tokenCount - 1 : 0};`。
- **L663 EN**: Executes statement involving `sizeof`.
  **L663 CN**: 执行涉及 `sizeof` 的语句。
- **L664 EN**: Introduces conditional control flow with an `if` statement.
  **L664 CN**: 通过 `if` 语句引入条件控制流。
- **L665 EN**: Introduces conditional control flow with an `if` statement.
  **L665 CN**: 通过 `if` 语句引入条件控制流。
- **L666 EN**: Executes statement involving `Deallocate`.
  **L666 CN**: 执行涉及 `Deallocate` 的语句。
- **L667 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L667 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L668 EN**: Executes statement `SubscriptValue sepExtent[1]{static_cast<SubscriptValue>(sepCount)};`.
  **L668 CN**: 执行语句 `SubscriptValue sepExtent[1]{static_cast<SubscriptValue>(sepCount)};`。
- **L669 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L669 CN**: 延续周围的声明、表达式或控制流结构。
- **L670 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L670 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L671 EN**: Executes statement involving `GetDimension`.
  **L671 CN**: 执行涉及 `GetDimension` 的语句。
- **L672 EN**: Introduces conditional control flow with an `if` statement.
  **L672 CN**: 通过 `if` 语句引入条件控制流。

### Lines 673-704

````cpp
      terminator.Crash("TOKENIZE: could not allocate SEPARATOR array");
    }
  }

  if (analysis.tokenCount == 0) {
    return;
  }

  // Populate tokens and separators.
  if (setChars == 0) {
    // One token (possibly empty) equal to STRING.
    if (tokenElemBytes > 0) {
      CHAR *tokDest{tokens.OffsetElement<CHAR>(0)};
      if (strChars > 0) {
        runtime::memcpy(tokDest, str, strChars * sizeof(CHAR));
      }
      TokenizeFillBlanks(tokDest + strChars, analysis.maxTokenLen - strChars);
    }
    return;
  }

  std::size_t tokenIndex{0};
  std::size_t sepIndex{0};

  auto storeToken = [&](std::size_t tokenStart, std::size_t tokenEnd) {
    std::size_t tokenLen{tokenEnd - tokenStart};
    if (tokenElemBytes > 0) {
      // Each element is stored in a fixed-size slot of `tokenElemBytes`.
      CHAR *tokDest{tokens.OffsetElement<CHAR>(tokenIndex * tokenElemBytes)};
      if (tokenLen > 0) {
        runtime::memcpy(tokDest, str + tokenStart, tokenLen * sizeof(CHAR));
      }
````

- **L673 EN**: Executes statement involving `Crash`.
  **L673 CN**: 执行涉及 `Crash` 的语句。
- **L674 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L674 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L675 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L675 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L676 EN**: Blank line separates nearby declarations or logic blocks.
  **L676 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L677 EN**: Introduces conditional control flow with an `if` statement.
  **L677 CN**: 通过 `if` 语句引入条件控制流。
- **L678 EN**: Returns from the current function, often propagating a computed result.
  **L678 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L679 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L679 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L680 EN**: Blank line separates nearby declarations or logic blocks.
  **L680 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L681 EN**: Comment documents intent or context: `Populate tokens and separators.`.
  **L681 CN**: 注释记录了意图或上下文：`Populate tokens and separators.`。
- **L682 EN**: Introduces conditional control flow with an `if` statement.
  **L682 CN**: 通过 `if` 语句引入条件控制流。
- **L683 EN**: Comment documents intent or context: `One token (possibly empty) equal to STRING.`.
  **L683 CN**: 注释记录了意图或上下文：`One token (possibly empty) equal to STRING.`。
- **L684 EN**: Introduces conditional control flow with an `if` statement.
  **L684 CN**: 通过 `if` 语句引入条件控制流。
- **L685 EN**: Executes statement `CHAR *tokDest{tokens.OffsetElement<CHAR>(0)};`.
  **L685 CN**: 执行语句 `CHAR *tokDest{tokens.OffsetElement<CHAR>(0)};`。
- **L686 EN**: Introduces conditional control flow with an `if` statement.
  **L686 CN**: 通过 `if` 语句引入条件控制流。
- **L687 EN**: Executes statement involving `memcpy`.
  **L687 CN**: 执行涉及 `memcpy` 的语句。
- **L688 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L688 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L689 EN**: Executes statement involving `TokenizeFillBlanks`.
  **L689 CN**: 执行涉及 `TokenizeFillBlanks` 的语句。
- **L690 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L690 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L691 EN**: Returns from the current function, often propagating a computed result.
  **L691 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L692 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L692 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L693 EN**: Blank line separates nearby declarations or logic blocks.
  **L693 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L694 EN**: Executes statement `std::size_t tokenIndex{0};`.
  **L694 CN**: 执行语句 `std::size_t tokenIndex{0};`。
- **L695 EN**: Executes statement `std::size_t sepIndex{0};`.
  **L695 CN**: 执行语句 `std::size_t sepIndex{0};`。
- **L696 EN**: Blank line separates nearby declarations or logic blocks.
  **L696 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L697 EN**: Initializes or updates `storeToken`.
  **L697 CN**: 初始化或更新 `storeToken`。
- **L698 EN**: Executes statement `std::size_t tokenLen{tokenEnd - tokenStart};`.
  **L698 CN**: 执行语句 `std::size_t tokenLen{tokenEnd - tokenStart};`。
- **L699 EN**: Introduces conditional control flow with an `if` statement.
  **L699 CN**: 通过 `if` 语句引入条件控制流。
- **L700 EN**: Comment documents intent or context: `Each element is stored in a fixed-size slot of `tokenElemBytes`.`.
  **L700 CN**: 注释记录了意图或上下文：`Each element is stored in a fixed-size slot of `tokenElemBytes`.`。
- **L701 EN**: Executes statement `CHAR *tokDest{tokens.OffsetElement<CHAR>(tokenIndex * tokenElemBytes)};`.
  **L701 CN**: 执行语句 `CHAR *tokDest{tokens.OffsetElement<CHAR>(tokenIndex * tokenElemBytes)};`。
- **L702 EN**: Introduces conditional control flow with an `if` statement.
  **L702 CN**: 通过 `if` 语句引入条件控制流。
- **L703 EN**: Executes statement involving `memcpy`.
  **L703 CN**: 执行涉及 `memcpy` 的语句。
- **L704 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L704 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 705-736

````cpp
      TokenizeFillBlanks(tokDest + tokenLen, analysis.maxTokenLen - tokenLen);
    }
    ++tokenIndex;
  };

  // Split at each delimiter character, producing empty tokens at boundaries
  // and between consecutive delimiters.
  std::size_t tokenStart{0};
  for (std::size_t pos{0}; pos < strChars; ++pos) {
    if (TokenizeIsInSet(str[pos], set, setChars)) {
      storeToken(tokenStart, pos);
      if (separator) {
        CHAR *sepDest{separator->OffsetElement<CHAR>(sepIndex * sepElemBytes)};
        sepDest[0] = str[pos];
        ++sepIndex;
      }
      tokenStart = pos + 1;
    }
  }
  storeToken(tokenStart, strChars);
}

template <int KIND>
static RT_API_ATTRS void TokenizeStoreIntAt(
    const Descriptor &result, std::size_t at, std::int64_t value) {
  StoreIntegerAt<KIND>{}(result, at, value);
}

using TokenizeStoreIntFn = void (*)(
    const Descriptor &, std::size_t, std::int64_t);

static RT_API_ATTRS TokenizeStoreIntFn GetTokenizeStoreIntFn(
````

- **L705 EN**: Executes statement involving `TokenizeFillBlanks`.
  **L705 CN**: 执行涉及 `TokenizeFillBlanks` 的语句。
- **L706 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L706 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L707 EN**: Executes statement `++tokenIndex;`.
  **L707 CN**: 执行语句 `++tokenIndex;`。
- **L708 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L708 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L709 EN**: Blank line separates nearby declarations or logic blocks.
  **L709 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L710 EN**: Comment documents intent or context: `Split at each delimiter character, producing empty tokens at boundaries`.
  **L710 CN**: 注释记录了意图或上下文：`Split at each delimiter character, producing empty tokens at boundaries`。
- **L711 EN**: Comment documents intent or context: `and between consecutive delimiters.`.
  **L711 CN**: 注释记录了意图或上下文：`and between consecutive delimiters.`。
- **L712 EN**: Executes statement `std::size_t tokenStart{0};`.
  **L712 CN**: 执行语句 `std::size_t tokenStart{0};`。
- **L713 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L713 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L714 EN**: Introduces conditional control flow with an `if` statement.
  **L714 CN**: 通过 `if` 语句引入条件控制流。
- **L715 EN**: Executes statement involving `storeToken`.
  **L715 CN**: 执行涉及 `storeToken` 的语句。
- **L716 EN**: Introduces conditional control flow with an `if` statement.
  **L716 CN**: 通过 `if` 语句引入条件控制流。
- **L717 EN**: Executes statement `CHAR *sepDest{separator->OffsetElement<CHAR>(sepIndex * sepElemBytes)};`.
  **L717 CN**: 执行语句 `CHAR *sepDest{separator->OffsetElement<CHAR>(sepIndex * sepElemBytes)};`。
- **L718 EN**: Initializes or updates `sepDest[0]`.
  **L718 CN**: 初始化或更新 `sepDest[0]`。
- **L719 EN**: Executes statement `++sepIndex;`.
  **L719 CN**: 执行语句 `++sepIndex;`。
- **L720 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L720 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L721 EN**: Initializes or updates `tokenStart`.
  **L721 CN**: 初始化或更新 `tokenStart`。
- **L722 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L722 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L723 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L723 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L724 EN**: Executes statement involving `storeToken`.
  **L724 CN**: 执行涉及 `storeToken` 的语句。
- **L725 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L725 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L726 EN**: Blank line separates nearby declarations or logic blocks.
  **L726 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L727 EN**: Begins a template declaration parameterizing subsequent code.
  **L727 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L728 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L728 CN**: 延续周围的声明、表达式或控制流结构。
- **L729 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L729 CN**: 延续周围的声明、表达式或控制流结构。
- **L730 EN**: Executes statement `StoreIntegerAt<KIND>{}(result, at, value);`.
  **L730 CN**: 执行语句 `StoreIntegerAt<KIND>{}(result, at, value);`。
- **L731 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L731 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L732 EN**: Blank line separates nearby declarations or logic blocks.
  **L732 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L733 EN**: Defines type alias `TokenizeStoreIntFn` for readability or ABI convenience.
  **L733 CN**: 定义类型别名 `TokenizeStoreIntFn`，以提升可读性或满足 ABI 便利性。
- **L734 EN**: Executes statement `const Descriptor &, std::size_t, std::int64_t);`.
  **L734 CN**: 执行语句 `const Descriptor &, std::size_t, std::int64_t);`。
- **L735 EN**: Blank line separates nearby declarations or logic blocks.
  **L735 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L736 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L736 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 737-768

````cpp
    int kind, Terminator &terminator, const char *which) {
  switch (kind) {
  case 1:
    return &TokenizeStoreIntAt<1>;
  case 2:
    return &TokenizeStoreIntAt<2>;
  case 4:
    return &TokenizeStoreIntAt<4>;
  case 8:
    return &TokenizeStoreIntAt<8>;
  case 16:
    return &TokenizeStoreIntAt<16>;
  default:
    terminator.Crash(
        "TOKENIZE: unsupported INTEGER kind=%d for %s", kind, which);
  }
}

template <typename CHAR>
static RT_API_ATTRS void TokenizeFillPositions(Descriptor &first,
    Descriptor &last, const CHAR *str, std::size_t strChars, const CHAR *set,
    std::size_t setChars, TokenizeStoreIntFn storeFirst,
    TokenizeStoreIntFn storeLast, Terminator &terminator) {

  // Empty STRING should return one empty token, per Fortran standard.
  if (strChars == 0) {
    storeFirst(first, 0, 1);
    storeLast(last, 0, 0);
    return;
  }
  if (setChars == 0) {
    storeFirst(first, 0, 1);
````

- **L737 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L737 CN**: 延续周围的声明、表达式或控制流结构。
- **L738 EN**: Begins a `switch` dispatch over discrete cases.
  **L738 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L739 EN**: Marks one `switch` case label.
  **L739 CN**: 标记一个 `switch` 的 case 标签。
- **L740 EN**: Returns from the current function, often propagating a computed result.
  **L740 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L741 EN**: Marks one `switch` case label.
  **L741 CN**: 标记一个 `switch` 的 case 标签。
- **L742 EN**: Returns from the current function, often propagating a computed result.
  **L742 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L743 EN**: Marks one `switch` case label.
  **L743 CN**: 标记一个 `switch` 的 case 标签。
- **L744 EN**: Returns from the current function, often propagating a computed result.
  **L744 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L745 EN**: Marks one `switch` case label.
  **L745 CN**: 标记一个 `switch` 的 case 标签。
- **L746 EN**: Returns from the current function, often propagating a computed result.
  **L746 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L747 EN**: Marks one `switch` case label.
  **L747 CN**: 标记一个 `switch` 的 case 标签。
- **L748 EN**: Returns from the current function, often propagating a computed result.
  **L748 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L749 EN**: Provides the default branch for a `switch` statement.
  **L749 CN**: 为 `switch` 语句提供默认分支。
- **L750 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L750 CN**: 延续周围的声明、表达式或控制流结构。
- **L751 EN**: Executes statement `"TOKENIZE: unsupported INTEGER kind=%d for %s", kind, which);`.
  **L751 CN**: 执行语句 `"TOKENIZE: unsupported INTEGER kind=%d for %s", kind, which);`。
- **L752 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L752 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L753 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L753 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L754 EN**: Blank line separates nearby declarations or logic blocks.
  **L754 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L755 EN**: Begins a template declaration parameterizing subsequent code.
  **L755 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L756 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L756 CN**: 延续周围的声明、表达式或控制流结构。
- **L757 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L757 CN**: 延续周围的声明、表达式或控制流结构。
- **L758 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L758 CN**: 延续周围的声明、表达式或控制流结构。
- **L759 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L759 CN**: 延续周围的声明、表达式或控制流结构。
- **L760 EN**: Blank line separates nearby declarations or logic blocks.
  **L760 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L761 EN**: Comment documents intent or context: `Empty STRING should return one empty token, per Fortran standard.`.
  **L761 CN**: 注释记录了意图或上下文：`Empty STRING should return one empty token, per Fortran standard.`。
- **L762 EN**: Introduces conditional control flow with an `if` statement.
  **L762 CN**: 通过 `if` 语句引入条件控制流。
- **L763 EN**: Executes statement involving `storeFirst`.
  **L763 CN**: 执行涉及 `storeFirst` 的语句。
- **L764 EN**: Executes statement involving `storeLast`.
  **L764 CN**: 执行涉及 `storeLast` 的语句。
- **L765 EN**: Returns from the current function, often propagating a computed result.
  **L765 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L766 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L766 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L767 EN**: Introduces conditional control flow with an `if` statement.
  **L767 CN**: 通过 `if` 语句引入条件控制流。
- **L768 EN**: Executes statement involving `storeFirst`.
  **L768 CN**: 执行涉及 `storeFirst` 的语句。

### Lines 769-800

````cpp
    storeLast(last, 0, static_cast<std::int64_t>(strChars));
    return;
  }

  std::size_t tokenIndex{0};
  std::size_t tokenStart{0};
  for (std::size_t pos{0}; pos < strChars; ++pos) {
    if (TokenizeIsInSet(str[pos], set, setChars)) {
      storeFirst(first, tokenIndex, static_cast<std::int64_t>(tokenStart + 1));
      storeLast(last, tokenIndex, static_cast<std::int64_t>(pos));
      ++tokenIndex;
      tokenStart = pos + 1;
    }
  }
  storeFirst(first, tokenIndex, static_cast<std::int64_t>(tokenStart + 1));
  storeLast(last, tokenIndex, static_cast<std::int64_t>(strChars));
  ++tokenIndex;

  // Sanity check: we should have filled exactly the allocated extent.
  if (tokenIndex != static_cast<std::size_t>(first.GetDimension(0).Extent())) {
    terminator.Crash("TOKENIZE: internal error populating FIRST/LAST");
  }
}

// Tokenize Form 1 implementation.
static RT_API_ATTRS void TokenizeImpl(Descriptor &tokens, Descriptor *separator,
    const Descriptor &string, const Descriptor &set, Terminator &terminator) {
  RUNTIME_CHECK(terminator, string.rank() == 0);
  RUNTIME_CHECK(terminator, set.rank() == 0);
  RUNTIME_CHECK(terminator, string.raw().type == set.raw().type);
  RUNTIME_CHECK(terminator, tokens.rank() == 1);
  RUNTIME_CHECK(terminator, tokens.IsAllocatable());
````

- **L769 EN**: Executes statement involving `storeLast`.
  **L769 CN**: 执行涉及 `storeLast` 的语句。
- **L770 EN**: Returns from the current function, often propagating a computed result.
  **L770 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L771 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L771 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L772 EN**: Blank line separates nearby declarations or logic blocks.
  **L772 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L773 EN**: Executes statement `std::size_t tokenIndex{0};`.
  **L773 CN**: 执行语句 `std::size_t tokenIndex{0};`。
- **L774 EN**: Executes statement `std::size_t tokenStart{0};`.
  **L774 CN**: 执行语句 `std::size_t tokenStart{0};`。
- **L775 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L775 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L776 EN**: Introduces conditional control flow with an `if` statement.
  **L776 CN**: 通过 `if` 语句引入条件控制流。
- **L777 EN**: Executes statement involving `storeFirst`.
  **L777 CN**: 执行涉及 `storeFirst` 的语句。
- **L778 EN**: Executes statement involving `storeLast`.
  **L778 CN**: 执行涉及 `storeLast` 的语句。
- **L779 EN**: Executes statement `++tokenIndex;`.
  **L779 CN**: 执行语句 `++tokenIndex;`。
- **L780 EN**: Initializes or updates `tokenStart`.
  **L780 CN**: 初始化或更新 `tokenStart`。
- **L781 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L781 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L782 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L782 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L783 EN**: Executes statement involving `storeFirst`.
  **L783 CN**: 执行涉及 `storeFirst` 的语句。
- **L784 EN**: Executes statement involving `storeLast`.
  **L784 CN**: 执行涉及 `storeLast` 的语句。
- **L785 EN**: Executes statement `++tokenIndex;`.
  **L785 CN**: 执行语句 `++tokenIndex;`。
- **L786 EN**: Blank line separates nearby declarations or logic blocks.
  **L786 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L787 EN**: Comment documents intent or context: `Sanity check: we should have filled exactly the allocated extent.`.
  **L787 CN**: 注释记录了意图或上下文：`Sanity check: we should have filled exactly the allocated extent.`。
- **L788 EN**: Introduces conditional control flow with an `if` statement.
  **L788 CN**: 通过 `if` 语句引入条件控制流。
- **L789 EN**: Executes statement involving `Crash`.
  **L789 CN**: 执行涉及 `Crash` 的语句。
- **L790 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L790 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L791 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L791 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L792 EN**: Blank line separates nearby declarations or logic blocks.
  **L792 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L793 EN**: Comment documents intent or context: `Tokenize Form 1 implementation.`.
  **L793 CN**: 注释记录了意图或上下文：`Tokenize Form 1 implementation.`。
- **L794 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L794 CN**: 延续周围的声明、表达式或控制流结构。
- **L795 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L795 CN**: 延续周围的声明、表达式或控制流结构。
- **L796 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L796 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L797 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L797 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L798 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L798 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L799 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L799 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L800 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L800 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。

### Lines 801-832

````cpp
  if (separator) {
    RUNTIME_CHECK(terminator, separator->rank() == 1);
    RUNTIME_CHECK(terminator, separator->IsAllocatable());
  }

  switch (string.raw().type) {
  case CFI_type_char: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes};
    std::size_t setChars{setBytes};
    const char *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char>()};
    const char *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char>()};
    auto analysis{AnalyzeTokenize(str, strChars, setPtr, setChars)};
    TokenizeFillForm1(tokens, separator, string, str, strChars, setPtr,
        setChars, analysis, terminator);
    break;
  }
  case CFI_type_char16_t: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 1};
    std::size_t setChars{setBytes >> 1};
    const char16_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char16_t>()};
    const char16_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char16_t>()};
    auto analysis{AnalyzeTokenize(str, strChars, setPtr, setChars)};
    TokenizeFillForm1(tokens, separator, string, str, strChars, setPtr,
        setChars, analysis, terminator);
````

- **L801 EN**: Introduces conditional control flow with an `if` statement.
  **L801 CN**: 通过 `if` 语句引入条件控制流。
- **L802 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L802 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L803 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L803 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L804 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L804 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L805 EN**: Blank line separates nearby declarations or logic blocks.
  **L805 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L806 EN**: Begins a `switch` dispatch over discrete cases.
  **L806 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L807 EN**: Marks one `switch` case label.
  **L807 CN**: 标记一个 `switch` 的 case 标签。
- **L808 EN**: Executes statement involving `ElementBytes`.
  **L808 CN**: 执行涉及 `ElementBytes` 的语句。
- **L809 EN**: Executes statement involving `ElementBytes`.
  **L809 CN**: 执行涉及 `ElementBytes` 的语句。
- **L810 EN**: Executes statement `std::size_t strChars{strBytes};`.
  **L810 CN**: 执行语句 `std::size_t strChars{strBytes};`。
- **L811 EN**: Executes statement `std::size_t setChars{setBytes};`.
  **L811 CN**: 执行语句 `std::size_t setChars{setBytes};`。
- **L812 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L812 CN**: 延续周围的声明、表达式或控制流结构。
- **L813 EN**: Initializes or updates `strBytes`.
  **L813 CN**: 初始化或更新 `strBytes`。
- **L814 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L814 CN**: 延续周围的声明、表达式或控制流结构。
- **L815 EN**: Initializes or updates `setBytes`.
  **L815 CN**: 初始化或更新 `setBytes`。
- **L816 EN**: Executes statement involving `AnalyzeTokenize`.
  **L816 CN**: 执行涉及 `AnalyzeTokenize` 的语句。
- **L817 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L817 CN**: 延续周围的声明、表达式或控制流结构。
- **L818 EN**: Executes statement `setChars, analysis, terminator);`.
  **L818 CN**: 执行语句 `setChars, analysis, terminator);`。
- **L819 EN**: Breaks out of the current loop or switch.
  **L819 CN**: 跳出当前循环或 switch。
- **L820 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L820 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L821 EN**: Marks one `switch` case label.
  **L821 CN**: 标记一个 `switch` 的 case 标签。
- **L822 EN**: Executes statement involving `ElementBytes`.
  **L822 CN**: 执行涉及 `ElementBytes` 的语句。
- **L823 EN**: Executes statement involving `ElementBytes`.
  **L823 CN**: 执行涉及 `ElementBytes` 的语句。
- **L824 EN**: Executes statement `std::size_t strChars{strBytes >> 1};`.
  **L824 CN**: 执行语句 `std::size_t strChars{strBytes >> 1};`。
- **L825 EN**: Executes statement `std::size_t setChars{setBytes >> 1};`.
  **L825 CN**: 执行语句 `std::size_t setChars{setBytes >> 1};`。
- **L826 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L826 CN**: 延续周围的声明、表达式或控制流结构。
- **L827 EN**: Initializes or updates `strBytes`.
  **L827 CN**: 初始化或更新 `strBytes`。
- **L828 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L828 CN**: 延续周围的声明、表达式或控制流结构。
- **L829 EN**: Initializes or updates `setBytes`.
  **L829 CN**: 初始化或更新 `setBytes`。
- **L830 EN**: Executes statement involving `AnalyzeTokenize`.
  **L830 CN**: 执行涉及 `AnalyzeTokenize` 的语句。
- **L831 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L831 CN**: 延续周围的声明、表达式或控制流结构。
- **L832 EN**: Executes statement `setChars, analysis, terminator);`.
  **L832 CN**: 执行语句 `setChars, analysis, terminator);`。

### Lines 833-864

````cpp
    break;
  }
  case CFI_type_char32_t: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 2};
    std::size_t setChars{setBytes >> 2};
    const char32_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char32_t>()};
    const char32_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char32_t>()};
    auto analysis{AnalyzeTokenize(str, strChars, setPtr, setChars)};
    TokenizeFillForm1(tokens, separator, string, str, strChars, setPtr,
        setChars, analysis, terminator);
    break;
  }
  default:
    terminator.Crash("TOKENIZE: bad string type code %d",
        static_cast<int>(string.raw().type));
  }
}

// Tokenize Form 2 implementation.
static RT_API_ATTRS void TokenizePositionsImpl(Descriptor &first,
    Descriptor &last, const Descriptor &string, const Descriptor &set,
    Terminator &terminator) {
  RUNTIME_CHECK(terminator, string.rank() == 0);
  RUNTIME_CHECK(terminator, set.rank() == 0);
  RUNTIME_CHECK(terminator, string.raw().type == set.raw().type);
  RUNTIME_CHECK(terminator, first.rank() == 1);
  RUNTIME_CHECK(terminator, last.rank() == 1);
  RUNTIME_CHECK(terminator, first.IsAllocatable());
````

- **L833 EN**: Breaks out of the current loop or switch.
  **L833 CN**: 跳出当前循环或 switch。
- **L834 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L834 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L835 EN**: Marks one `switch` case label.
  **L835 CN**: 标记一个 `switch` 的 case 标签。
- **L836 EN**: Executes statement involving `ElementBytes`.
  **L836 CN**: 执行涉及 `ElementBytes` 的语句。
- **L837 EN**: Executes statement involving `ElementBytes`.
  **L837 CN**: 执行涉及 `ElementBytes` 的语句。
- **L838 EN**: Executes statement `std::size_t strChars{strBytes >> 2};`.
  **L838 CN**: 执行语句 `std::size_t strChars{strBytes >> 2};`。
- **L839 EN**: Executes statement `std::size_t setChars{setBytes >> 2};`.
  **L839 CN**: 执行语句 `std::size_t setChars{setBytes >> 2};`。
- **L840 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L840 CN**: 延续周围的声明、表达式或控制流结构。
- **L841 EN**: Initializes or updates `strBytes`.
  **L841 CN**: 初始化或更新 `strBytes`。
- **L842 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L842 CN**: 延续周围的声明、表达式或控制流结构。
- **L843 EN**: Initializes or updates `setBytes`.
  **L843 CN**: 初始化或更新 `setBytes`。
- **L844 EN**: Executes statement involving `AnalyzeTokenize`.
  **L844 CN**: 执行涉及 `AnalyzeTokenize` 的语句。
- **L845 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L845 CN**: 延续周围的声明、表达式或控制流结构。
- **L846 EN**: Executes statement `setChars, analysis, terminator);`.
  **L846 CN**: 执行语句 `setChars, analysis, terminator);`。
- **L847 EN**: Breaks out of the current loop or switch.
  **L847 CN**: 跳出当前循环或 switch。
- **L848 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L848 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L849 EN**: Provides the default branch for a `switch` statement.
  **L849 CN**: 为 `switch` 语句提供默认分支。
- **L850 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L850 CN**: 延续周围的声明、表达式或控制流结构。
- **L851 EN**: Executes statement involving `raw`.
  **L851 CN**: 执行涉及 `raw` 的语句。
- **L852 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L852 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L853 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L853 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L854 EN**: Blank line separates nearby declarations or logic blocks.
  **L854 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L855 EN**: Comment documents intent or context: `Tokenize Form 2 implementation.`.
  **L855 CN**: 注释记录了意图或上下文：`Tokenize Form 2 implementation.`。
- **L856 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L856 CN**: 延续周围的声明、表达式或控制流结构。
- **L857 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L857 CN**: 延续周围的声明、表达式或控制流结构。
- **L858 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L858 CN**: 延续周围的声明、表达式或控制流结构。
- **L859 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L859 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L860 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L860 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L861 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L861 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L862 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L862 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L863 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L863 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L864 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L864 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。

### Lines 865-896

````cpp
  RUNTIME_CHECK(terminator, last.IsAllocatable());

  auto firstCK{first.type().GetCategoryAndKind()};
  auto lastCK{last.type().GetCategoryAndKind()};
  if (!firstCK || firstCK->first != TypeCategory::Integer) {
    terminator.Crash("TOKENIZE: FIRST is not an INTEGER array");
  }
  if (!lastCK || lastCK->first != TypeCategory::Integer) {
    terminator.Crash("TOKENIZE: LAST is not an INTEGER array");
  }
  int firstKind{firstCK->second};
  int lastKind{lastCK->second};
  auto storeFirst{GetTokenizeStoreIntFn(firstKind, terminator, "FIRST")};
  auto storeLast{GetTokenizeStoreIntFn(lastKind, terminator, "LAST")};

  // Count tokens.
  std::size_t tokenCount{0};
  switch (string.raw().type) {
  case CFI_type_char: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes};
    std::size_t setChars{setBytes};
    const char *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char>()};
    const char *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char>()};
    tokenCount = AnalyzeTokenize(str, strChars, setPtr, setChars).tokenCount;
    break;
  }
  case CFI_type_char16_t: {
    std::size_t strBytes{string.ElementBytes()};
````

- **L865 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L865 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L866 EN**: Blank line separates nearby declarations or logic blocks.
  **L866 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L867 EN**: Executes statement involving `type`.
  **L867 CN**: 执行涉及 `type` 的语句。
- **L868 EN**: Executes statement involving `type`.
  **L868 CN**: 执行涉及 `type` 的语句。
- **L869 EN**: Introduces conditional control flow with an `if` statement.
  **L869 CN**: 通过 `if` 语句引入条件控制流。
- **L870 EN**: Executes statement involving `Crash`.
  **L870 CN**: 执行涉及 `Crash` 的语句。
- **L871 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L871 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L872 EN**: Introduces conditional control flow with an `if` statement.
  **L872 CN**: 通过 `if` 语句引入条件控制流。
- **L873 EN**: Executes statement involving `Crash`.
  **L873 CN**: 执行涉及 `Crash` 的语句。
- **L874 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L874 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L875 EN**: Executes statement `int firstKind{firstCK->second};`.
  **L875 CN**: 执行语句 `int firstKind{firstCK->second};`。
- **L876 EN**: Executes statement `int lastKind{lastCK->second};`.
  **L876 CN**: 执行语句 `int lastKind{lastCK->second};`。
- **L877 EN**: Executes statement involving `GetTokenizeStoreIntFn`.
  **L877 CN**: 执行涉及 `GetTokenizeStoreIntFn` 的语句。
- **L878 EN**: Executes statement involving `GetTokenizeStoreIntFn`.
  **L878 CN**: 执行涉及 `GetTokenizeStoreIntFn` 的语句。
- **L879 EN**: Blank line separates nearby declarations or logic blocks.
  **L879 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L880 EN**: Comment documents intent or context: `Count tokens.`.
  **L880 CN**: 注释记录了意图或上下文：`Count tokens.`。
- **L881 EN**: Executes statement `std::size_t tokenCount{0};`.
  **L881 CN**: 执行语句 `std::size_t tokenCount{0};`。
- **L882 EN**: Begins a `switch` dispatch over discrete cases.
  **L882 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L883 EN**: Marks one `switch` case label.
  **L883 CN**: 标记一个 `switch` 的 case 标签。
- **L884 EN**: Executes statement involving `ElementBytes`.
  **L884 CN**: 执行涉及 `ElementBytes` 的语句。
- **L885 EN**: Executes statement involving `ElementBytes`.
  **L885 CN**: 执行涉及 `ElementBytes` 的语句。
- **L886 EN**: Executes statement `std::size_t strChars{strBytes};`.
  **L886 CN**: 执行语句 `std::size_t strChars{strBytes};`。
- **L887 EN**: Executes statement `std::size_t setChars{setBytes};`.
  **L887 CN**: 执行语句 `std::size_t setChars{setBytes};`。
- **L888 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L888 CN**: 延续周围的声明、表达式或控制流结构。
- **L889 EN**: Initializes or updates `strBytes`.
  **L889 CN**: 初始化或更新 `strBytes`。
- **L890 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L890 CN**: 延续周围的声明、表达式或控制流结构。
- **L891 EN**: Initializes or updates `setBytes`.
  **L891 CN**: 初始化或更新 `setBytes`。
- **L892 EN**: Initializes or updates `tokenCount`.
  **L892 CN**: 初始化或更新 `tokenCount`。
- **L893 EN**: Breaks out of the current loop or switch.
  **L893 CN**: 跳出当前循环或 switch。
- **L894 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L894 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L895 EN**: Marks one `switch` case label.
  **L895 CN**: 标记一个 `switch` 的 case 标签。
- **L896 EN**: Executes statement involving `ElementBytes`.
  **L896 CN**: 执行涉及 `ElementBytes` 的语句。

### Lines 897-928

````cpp
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 1};
    std::size_t setChars{setBytes >> 1};
    const char16_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char16_t>()};
    const char16_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char16_t>()};
    tokenCount = AnalyzeTokenize(str, strChars, setPtr, setChars).tokenCount;
    break;
  }
  case CFI_type_char32_t: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 2};
    std::size_t setChars{setBytes >> 2};
    const char32_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char32_t>()};
    const char32_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char32_t>()};
    tokenCount = AnalyzeTokenize(str, strChars, setPtr, setChars).tokenCount;
    break;
  }
  default:
    terminator.Crash("TOKENIZE: bad string type code %d",
        static_cast<int>(string.raw().type));
  }

  // (Re)allocate FIRST/LAST.
  if (first.IsAllocated()) {
    first.Deallocate();
  }
  if (last.IsAllocated()) {
````

- **L897 EN**: Executes statement involving `ElementBytes`.
  **L897 CN**: 执行涉及 `ElementBytes` 的语句。
- **L898 EN**: Executes statement `std::size_t strChars{strBytes >> 1};`.
  **L898 CN**: 执行语句 `std::size_t strChars{strBytes >> 1};`。
- **L899 EN**: Executes statement `std::size_t setChars{setBytes >> 1};`.
  **L899 CN**: 执行语句 `std::size_t setChars{setBytes >> 1};`。
- **L900 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L900 CN**: 延续周围的声明、表达式或控制流结构。
- **L901 EN**: Initializes or updates `strBytes`.
  **L901 CN**: 初始化或更新 `strBytes`。
- **L902 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L902 CN**: 延续周围的声明、表达式或控制流结构。
- **L903 EN**: Initializes or updates `setBytes`.
  **L903 CN**: 初始化或更新 `setBytes`。
- **L904 EN**: Initializes or updates `tokenCount`.
  **L904 CN**: 初始化或更新 `tokenCount`。
- **L905 EN**: Breaks out of the current loop or switch.
  **L905 CN**: 跳出当前循环或 switch。
- **L906 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L906 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L907 EN**: Marks one `switch` case label.
  **L907 CN**: 标记一个 `switch` 的 case 标签。
- **L908 EN**: Executes statement involving `ElementBytes`.
  **L908 CN**: 执行涉及 `ElementBytes` 的语句。
- **L909 EN**: Executes statement involving `ElementBytes`.
  **L909 CN**: 执行涉及 `ElementBytes` 的语句。
- **L910 EN**: Executes statement `std::size_t strChars{strBytes >> 2};`.
  **L910 CN**: 执行语句 `std::size_t strChars{strBytes >> 2};`。
- **L911 EN**: Executes statement `std::size_t setChars{setBytes >> 2};`.
  **L911 CN**: 执行语句 `std::size_t setChars{setBytes >> 2};`。
- **L912 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L912 CN**: 延续周围的声明、表达式或控制流结构。
- **L913 EN**: Initializes or updates `strBytes`.
  **L913 CN**: 初始化或更新 `strBytes`。
- **L914 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L914 CN**: 延续周围的声明、表达式或控制流结构。
- **L915 EN**: Initializes or updates `setBytes`.
  **L915 CN**: 初始化或更新 `setBytes`。
- **L916 EN**: Initializes or updates `tokenCount`.
  **L916 CN**: 初始化或更新 `tokenCount`。
- **L917 EN**: Breaks out of the current loop or switch.
  **L917 CN**: 跳出当前循环或 switch。
- **L918 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L918 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L919 EN**: Provides the default branch for a `switch` statement.
  **L919 CN**: 为 `switch` 语句提供默认分支。
- **L920 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L920 CN**: 延续周围的声明、表达式或控制流结构。
- **L921 EN**: Executes statement involving `raw`.
  **L921 CN**: 执行涉及 `raw` 的语句。
- **L922 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L922 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L923 EN**: Blank line separates nearby declarations or logic blocks.
  **L923 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L924 EN**: Comment documents intent or context: `(Re)allocate FIRST/LAST.`.
  **L924 CN**: 注释记录了意图或上下文：`(Re)allocate FIRST/LAST.`。
- **L925 EN**: Introduces conditional control flow with an `if` statement.
  **L925 CN**: 通过 `if` 语句引入条件控制流。
- **L926 EN**: Executes statement involving `Deallocate`.
  **L926 CN**: 执行涉及 `Deallocate` 的语句。
- **L927 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L927 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L928 EN**: Introduces conditional control flow with an `if` statement.
  **L928 CN**: 通过 `if` 语句引入条件控制流。

### Lines 929-960

````cpp
    last.Deallocate();
  }
  SubscriptValue extent[1]{static_cast<SubscriptValue>(tokenCount)};
  first.Establish(TypeCategory::Integer, firstKind, nullptr, 1, extent,
      CFI_attribute_allocatable);
  first.GetDimension(0).SetBounds(1, extent[0]);
  last.Establish(TypeCategory::Integer, lastKind, nullptr, 1, extent,
      CFI_attribute_allocatable);
  last.GetDimension(0).SetBounds(1, extent[0]);
  if (first.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("TOKENIZE: could not allocate FIRST array");
  }
  if (last.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("TOKENIZE: could not allocate LAST array");
  }

  if (tokenCount == 0) {
    return;
  }

  // Populate FIRST/LAST.
  switch (string.raw().type) {
  case CFI_type_char: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes};
    std::size_t setChars{setBytes};
    const char *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char>()};
    const char *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char>()};
    TokenizeFillPositions(first, last, str, strChars, setPtr, setChars,
````

- **L929 EN**: Executes statement involving `Deallocate`.
  **L929 CN**: 执行涉及 `Deallocate` 的语句。
- **L930 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L930 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L931 EN**: Executes statement `SubscriptValue extent[1]{static_cast<SubscriptValue>(tokenCount)};`.
  **L931 CN**: 执行语句 `SubscriptValue extent[1]{static_cast<SubscriptValue>(tokenCount)};`。
- **L932 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L932 CN**: 延续周围的声明、表达式或控制流结构。
- **L933 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L933 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L934 EN**: Executes statement involving `GetDimension`.
  **L934 CN**: 执行涉及 `GetDimension` 的语句。
- **L935 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L935 CN**: 延续周围的声明、表达式或控制流结构。
- **L936 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L936 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L937 EN**: Executes statement involving `GetDimension`.
  **L937 CN**: 执行涉及 `GetDimension` 的语句。
- **L938 EN**: Introduces conditional control flow with an `if` statement.
  **L938 CN**: 通过 `if` 语句引入条件控制流。
- **L939 EN**: Executes statement involving `Crash`.
  **L939 CN**: 执行涉及 `Crash` 的语句。
- **L940 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L940 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L941 EN**: Introduces conditional control flow with an `if` statement.
  **L941 CN**: 通过 `if` 语句引入条件控制流。
- **L942 EN**: Executes statement involving `Crash`.
  **L942 CN**: 执行涉及 `Crash` 的语句。
- **L943 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L943 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L944 EN**: Blank line separates nearby declarations or logic blocks.
  **L944 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L945 EN**: Introduces conditional control flow with an `if` statement.
  **L945 CN**: 通过 `if` 语句引入条件控制流。
- **L946 EN**: Returns from the current function, often propagating a computed result.
  **L946 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L947 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L947 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L948 EN**: Blank line separates nearby declarations or logic blocks.
  **L948 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L949 EN**: Comment documents intent or context: `Populate FIRST/LAST.`.
  **L949 CN**: 注释记录了意图或上下文：`Populate FIRST/LAST.`。
- **L950 EN**: Begins a `switch` dispatch over discrete cases.
  **L950 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L951 EN**: Marks one `switch` case label.
  **L951 CN**: 标记一个 `switch` 的 case 标签。
- **L952 EN**: Executes statement involving `ElementBytes`.
  **L952 CN**: 执行涉及 `ElementBytes` 的语句。
- **L953 EN**: Executes statement involving `ElementBytes`.
  **L953 CN**: 执行涉及 `ElementBytes` 的语句。
- **L954 EN**: Executes statement `std::size_t strChars{strBytes};`.
  **L954 CN**: 执行语句 `std::size_t strChars{strBytes};`。
- **L955 EN**: Executes statement `std::size_t setChars{setBytes};`.
  **L955 CN**: 执行语句 `std::size_t setChars{setBytes};`。
- **L956 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L956 CN**: 延续周围的声明、表达式或控制流结构。
- **L957 EN**: Initializes or updates `strBytes`.
  **L957 CN**: 初始化或更新 `strBytes`。
- **L958 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L958 CN**: 延续周围的声明、表达式或控制流结构。
- **L959 EN**: Initializes or updates `setBytes`.
  **L959 CN**: 初始化或更新 `setBytes`。
- **L960 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L960 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 961-992

````cpp
        storeFirst, storeLast, terminator);
    break;
  }
  case CFI_type_char16_t: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 1};
    std::size_t setChars{setBytes >> 1};
    const char16_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char16_t>()};
    const char16_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char16_t>()};
    TokenizeFillPositions(first, last, str, strChars, setPtr, setChars,
        storeFirst, storeLast, terminator);
    break;
  }
  case CFI_type_char32_t: {
    std::size_t strBytes{string.ElementBytes()};
    std::size_t setBytes{set.ElementBytes()};
    std::size_t strChars{strBytes >> 2};
    std::size_t setChars{setBytes >> 2};
    const char32_t *str{
        strBytes == 0 ? nullptr : string.OffsetElement<const char32_t>()};
    const char32_t *setPtr{
        setBytes == 0 ? nullptr : set.OffsetElement<const char32_t>()};
    TokenizeFillPositions(first, last, str, strChars, setPtr, setChars,
        storeFirst, storeLast, terminator);
    break;
  }
  default:
    break;
  }
````

- **L961 EN**: Executes statement `storeFirst, storeLast, terminator);`.
  **L961 CN**: 执行语句 `storeFirst, storeLast, terminator);`。
- **L962 EN**: Breaks out of the current loop or switch.
  **L962 CN**: 跳出当前循环或 switch。
- **L963 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L963 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L964 EN**: Marks one `switch` case label.
  **L964 CN**: 标记一个 `switch` 的 case 标签。
- **L965 EN**: Executes statement involving `ElementBytes`.
  **L965 CN**: 执行涉及 `ElementBytes` 的语句。
- **L966 EN**: Executes statement involving `ElementBytes`.
  **L966 CN**: 执行涉及 `ElementBytes` 的语句。
- **L967 EN**: Executes statement `std::size_t strChars{strBytes >> 1};`.
  **L967 CN**: 执行语句 `std::size_t strChars{strBytes >> 1};`。
- **L968 EN**: Executes statement `std::size_t setChars{setBytes >> 1};`.
  **L968 CN**: 执行语句 `std::size_t setChars{setBytes >> 1};`。
- **L969 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L969 CN**: 延续周围的声明、表达式或控制流结构。
- **L970 EN**: Initializes or updates `strBytes`.
  **L970 CN**: 初始化或更新 `strBytes`。
- **L971 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L971 CN**: 延续周围的声明、表达式或控制流结构。
- **L972 EN**: Initializes or updates `setBytes`.
  **L972 CN**: 初始化或更新 `setBytes`。
- **L973 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L973 CN**: 延续周围的声明、表达式或控制流结构。
- **L974 EN**: Executes statement `storeFirst, storeLast, terminator);`.
  **L974 CN**: 执行语句 `storeFirst, storeLast, terminator);`。
- **L975 EN**: Breaks out of the current loop or switch.
  **L975 CN**: 跳出当前循环或 switch。
- **L976 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L976 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L977 EN**: Marks one `switch` case label.
  **L977 CN**: 标记一个 `switch` 的 case 标签。
- **L978 EN**: Executes statement involving `ElementBytes`.
  **L978 CN**: 执行涉及 `ElementBytes` 的语句。
- **L979 EN**: Executes statement involving `ElementBytes`.
  **L979 CN**: 执行涉及 `ElementBytes` 的语句。
- **L980 EN**: Executes statement `std::size_t strChars{strBytes >> 2};`.
  **L980 CN**: 执行语句 `std::size_t strChars{strBytes >> 2};`。
- **L981 EN**: Executes statement `std::size_t setChars{setBytes >> 2};`.
  **L981 CN**: 执行语句 `std::size_t setChars{setBytes >> 2};`。
- **L982 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L982 CN**: 延续周围的声明、表达式或控制流结构。
- **L983 EN**: Initializes or updates `strBytes`.
  **L983 CN**: 初始化或更新 `strBytes`。
- **L984 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L984 CN**: 延续周围的声明、表达式或控制流结构。
- **L985 EN**: Initializes or updates `setBytes`.
  **L985 CN**: 初始化或更新 `setBytes`。
- **L986 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L986 CN**: 延续周围的声明、表达式或控制流结构。
- **L987 EN**: Executes statement `storeFirst, storeLast, terminator);`.
  **L987 CN**: 执行语句 `storeFirst, storeLast, terminator);`。
- **L988 EN**: Breaks out of the current loop or switch.
  **L988 CN**: 跳出当前循环或 switch。
- **L989 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L989 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L990 EN**: Provides the default branch for a `switch` statement.
  **L990 CN**: 为 `switch` 语句提供默认分支。
- **L991 EN**: Breaks out of the current loop or switch.
  **L991 CN**: 跳出当前循环或 switch。
- **L992 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L992 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 993-1024

````cpp
}

// SPLIT — implemented in terms of SCAN.
// When BACK is false, returns the position of the leftmost character in SET
// at a position > POS, or LEN(STRING)+1 if none.
// When BACK is true, returns the position of the rightmost character in SET
// at a position < POS, or 0 if none.
template <typename CHAR>
static RT_API_ATTRS std::size_t SplitImpl(const CHAR *string,
    std::size_t stringLen, const CHAR *set, std::size_t setLen, std::size_t pos,
    bool back) {
  if (back) {
    std::size_t scanLen{pos > 1 ? pos - 1 : std::size_t{0}};
    if (scanLen > stringLen) {
      scanLen = stringLen;
    }
    if constexpr (sizeof(CHAR) == 1) {
      return ScanVerify<false>(string, scanLen, set, setLen, true);
    } else {
      return ScanVerify<CHAR, CharFunc::Scan>(
          string, scanLen, set, setLen, true);
    }
  } else {
    if (pos >= stringLen) {
      return stringLen + 1;
    }
    std::size_t npos;
    if constexpr (sizeof(CHAR) == 1) {
      npos =
          ScanVerify<false>(string + pos, stringLen - pos, set, setLen, false);
    } else {
      npos = ScanVerify<CHAR, CharFunc::Scan>(
````

- **L993 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L993 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L994 EN**: Blank line separates nearby declarations or logic blocks.
  **L994 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L995 EN**: Comment documents intent or context: `SPLIT — implemented in terms of SCAN.`.
  **L995 CN**: 注释记录了意图或上下文：`SPLIT — implemented in terms of SCAN.`。
- **L996 EN**: Comment documents intent or context: `When BACK is false, returns the position of the leftmost character in SET`.
  **L996 CN**: 注释记录了意图或上下文：`When BACK is false, returns the position of the leftmost character in SET`。
- **L997 EN**: Comment documents intent or context: `at a position > POS, or LEN(STRING)+1 if none.`.
  **L997 CN**: 注释记录了意图或上下文：`at a position > POS, or LEN(STRING)+1 if none.`。
- **L998 EN**: Comment documents intent or context: `When BACK is true, returns the position of the rightmost character in SET`.
  **L998 CN**: 注释记录了意图或上下文：`When BACK is true, returns the position of the rightmost character in SET`。
- **L999 EN**: Comment documents intent or context: `at a position < POS, or 0 if none.`.
  **L999 CN**: 注释记录了意图或上下文：`at a position < POS, or 0 if none.`。
- **L1000 EN**: Begins a template declaration parameterizing subsequent code.
  **L1000 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L1001 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1001 CN**: 延续周围的声明、表达式或控制流结构。
- **L1002 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1002 CN**: 延续周围的声明、表达式或控制流结构。
- **L1003 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1003 CN**: 延续周围的声明、表达式或控制流结构。
- **L1004 EN**: Introduces conditional control flow with an `if` statement.
  **L1004 CN**: 通过 `if` 语句引入条件控制流。
- **L1005 EN**: Executes statement `std::size_t scanLen{pos > 1 ? pos - 1 : std::size_t{0}};`.
  **L1005 CN**: 执行语句 `std::size_t scanLen{pos > 1 ? pos - 1 : std::size_t{0}};`。
- **L1006 EN**: Introduces conditional control flow with an `if` statement.
  **L1006 CN**: 通过 `if` 语句引入条件控制流。
- **L1007 EN**: Initializes or updates `scanLen`.
  **L1007 CN**: 初始化或更新 `scanLen`。
- **L1008 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1008 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1009 EN**: Introduces conditional control flow with an `if` statement.
  **L1009 CN**: 通过 `if` 语句引入条件控制流。
- **L1010 EN**: Returns from the current function, often propagating a computed result.
  **L1010 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1011 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1011 CN**: 延续周围的声明、表达式或控制流结构。
- **L1012 EN**: Returns from the current function, often propagating a computed result.
  **L1012 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1013 EN**: Executes statement `string, scanLen, set, setLen, true);`.
  **L1013 CN**: 执行语句 `string, scanLen, set, setLen, true);`。
- **L1014 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1014 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1015 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1015 CN**: 延续周围的声明、表达式或控制流结构。
- **L1016 EN**: Introduces conditional control flow with an `if` statement.
  **L1016 CN**: 通过 `if` 语句引入条件控制流。
- **L1017 EN**: Returns from the current function, often propagating a computed result.
  **L1017 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1018 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1018 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1019 EN**: Executes statement `std::size_t npos;`.
  **L1019 CN**: 执行语句 `std::size_t npos;`。
- **L1020 EN**: Introduces conditional control flow with an `if` statement.
  **L1020 CN**: 通过 `if` 语句引入条件控制流。
- **L1021 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1021 CN**: 延续周围的声明、表达式或控制流结构。
- **L1022 EN**: Executes statement `ScanVerify<false>(string + pos, stringLen - pos, set, setLen, false);`.
  **L1022 CN**: 执行语句 `ScanVerify<false>(string + pos, stringLen - pos, set, setLen, false);`。
- **L1023 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1023 CN**: 延续周围的声明、表达式或控制流结构。
- **L1024 EN**: Initializes or updates `npos`.
  **L1024 CN**: 初始化或更新 `npos`。

### Lines 1025-1056

````cpp
          string + pos, stringLen - pos, set, setLen, false);
    }
    return npos != 0 ? pos + npos : stringLen + 1;
  }
}

extern "C" {
RT_EXT_API_GROUP_BEGIN

void RTDEF(CharacterConcatenate)(Descriptor &accumulator,
    const Descriptor &from, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  RUNTIME_CHECK(terminator,
      accumulator.rank() == 0 || from.rank() == 0 ||
          accumulator.rank() == from.rank());
  int rank{std::max(accumulator.rank(), from.rank())};
  SubscriptValue ub[maxRank], fromAt[maxRank];
  SubscriptValue elements{1};
  for (int j{0}; j < rank; ++j) {
    if (accumulator.rank() > 0 && from.rank() > 0) {
      ub[j] = accumulator.GetDimension(j).Extent();
      SubscriptValue fromUB{from.GetDimension(j).Extent()};
      if (ub[j] != fromUB) {
        terminator.Crash("Character array concatenation: operands are not "
                         "conforming on dimension %d (%jd != %jd)",
            j + 1, static_cast<std::intmax_t>(ub[j]),
            static_cast<std::intmax_t>(fromUB));
      }
    } else {
      ub[j] =
          (accumulator.rank() ? accumulator : from).GetDimension(j).Extent();
    }
````

- **L1025 EN**: Executes statement `string + pos, stringLen - pos, set, setLen, false);`.
  **L1025 CN**: 执行语句 `string + pos, stringLen - pos, set, setLen, false);`。
- **L1026 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1026 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1027 EN**: Returns from the current function, often propagating a computed result.
  **L1027 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1028 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1028 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1029 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1029 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1030 EN**: Blank line separates nearby declarations or logic blocks.
  **L1030 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1031 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1031 CN**: 延续周围的声明、表达式或控制流结构。
- **L1032 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1032 CN**: 延续周围的声明、表达式或控制流结构。
- **L1033 EN**: Blank line separates nearby declarations or logic blocks.
  **L1033 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1034 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1034 CN**: 延续周围的声明、表达式或控制流结构。
- **L1035 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1035 CN**: 延续周围的声明、表达式或控制流结构。
- **L1036 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1036 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1037 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1037 CN**: 延续周围的声明、表达式或控制流结构。
- **L1038 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1038 CN**: 延续周围的声明、表达式或控制流结构。
- **L1039 EN**: Executes statement involving `rank`.
  **L1039 CN**: 执行涉及 `rank` 的语句。
- **L1040 EN**: Executes statement involving `max`.
  **L1040 CN**: 执行涉及 `max` 的语句。
- **L1041 EN**: Executes statement `SubscriptValue ub[maxRank], fromAt[maxRank];`.
  **L1041 CN**: 执行语句 `SubscriptValue ub[maxRank], fromAt[maxRank];`。
- **L1042 EN**: Executes statement `SubscriptValue elements{1};`.
  **L1042 CN**: 执行语句 `SubscriptValue elements{1};`。
- **L1043 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L1043 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L1044 EN**: Introduces conditional control flow with an `if` statement.
  **L1044 CN**: 通过 `if` 语句引入条件控制流。
- **L1045 EN**: Initializes or updates `ub[j]`.
  **L1045 CN**: 初始化或更新 `ub[j]`。
- **L1046 EN**: Executes statement involving `GetDimension`.
  **L1046 CN**: 执行涉及 `GetDimension` 的语句。
- **L1047 EN**: Introduces conditional control flow with an `if` statement.
  **L1047 CN**: 通过 `if` 语句引入条件控制流。
- **L1048 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1048 CN**: 延续周围的声明、表达式或控制流结构。
- **L1049 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1049 CN**: 延续周围的声明、表达式或控制流结构。
- **L1050 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1050 CN**: 延续周围的声明、表达式或控制流结构。
- **L1051 EN**: Executes statement `static_cast<std::intmax_t>(fromUB));`.
  **L1051 CN**: 执行语句 `static_cast<std::intmax_t>(fromUB));`。
- **L1052 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1052 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1053 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1053 CN**: 延续周围的声明、表达式或控制流结构。
- **L1054 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1054 CN**: 延续周围的声明、表达式或控制流结构。
- **L1055 EN**: Executes statement involving `rank`.
  **L1055 CN**: 执行涉及 `rank` 的语句。
- **L1056 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1056 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 1057-1088

````cpp
    elements *= ub[j];
  }
  std::size_t oldBytes{accumulator.ElementBytes()};
  void *old{accumulator.raw().base_addr};
  accumulator.set_base_addr(nullptr);
  std::size_t fromBytes{from.ElementBytes()};
  accumulator.raw().elem_len += fromBytes;
  std::size_t newBytes{accumulator.ElementBytes()};
  for (int j{0}; j < rank; ++j) {
    accumulator.GetDimension(j).SetBounds(1, ub[j]);
  }
  if (accumulator.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash(
        "CharacterConcatenate: could not allocate storage for result");
  }
  const char *p{static_cast<const char *>(old)};
  char *to{static_cast<char *>(accumulator.raw().base_addr)};
  from.GetLowerBounds(fromAt);
  for (; elements-- > 0;
       to += newBytes, p += oldBytes, from.IncrementSubscripts(fromAt)) {
    runtime::memcpy(to, p, oldBytes);
    runtime::memcpy(to + oldBytes, from.Element<char>(fromAt), fromBytes);
  }
  FreeMemory(old);
}

void RTDEF(CharacterConcatenateScalar1)(
    Descriptor &accumulator, const char *from, std::size_t chars) {
  Terminator terminator{__FILE__, __LINE__};
  RUNTIME_CHECK(terminator, accumulator.rank() == 0);
  void *old{accumulator.raw().base_addr};
  accumulator.set_base_addr(nullptr);
````

- **L1057 EN**: Initializes or updates `*`.
  **L1057 CN**: 初始化或更新 `*`。
- **L1058 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1058 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1059 EN**: Executes statement involving `ElementBytes`.
  **L1059 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1060 EN**: Executes statement involving `raw`.
  **L1060 CN**: 执行涉及 `raw` 的语句。
- **L1061 EN**: Executes statement involving `set_base_addr`.
  **L1061 CN**: 执行涉及 `set_base_addr` 的语句。
- **L1062 EN**: Executes statement involving `ElementBytes`.
  **L1062 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1063 EN**: Initializes or updates `+`.
  **L1063 CN**: 初始化或更新 `+`。
- **L1064 EN**: Executes statement involving `ElementBytes`.
  **L1064 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1065 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L1065 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L1066 EN**: Executes statement involving `GetDimension`.
  **L1066 CN**: 执行涉及 `GetDimension` 的语句。
- **L1067 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1067 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1068 EN**: Introduces conditional control flow with an `if` statement.
  **L1068 CN**: 通过 `if` 语句引入条件控制流。
- **L1069 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1069 CN**: 延续周围的声明、表达式或控制流结构。
- **L1070 EN**: Executes statement `"CharacterConcatenate: could not allocate storage for result");`.
  **L1070 CN**: 执行语句 `"CharacterConcatenate: could not allocate storage for result");`。
- **L1071 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1071 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1072 EN**: Executes statement `const char *p{static_cast<const char *>(old)};`.
  **L1072 CN**: 执行语句 `const char *p{static_cast<const char *>(old)};`。
- **L1073 EN**: Executes statement involving `raw`.
  **L1073 CN**: 执行涉及 `raw` 的语句。
- **L1074 EN**: Executes statement involving `GetLowerBounds`.
  **L1074 CN**: 执行涉及 `GetLowerBounds` 的语句。
- **L1075 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L1075 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L1076 EN**: Initializes or updates `+`.
  **L1076 CN**: 初始化或更新 `+`。
- **L1077 EN**: Executes statement involving `memcpy`.
  **L1077 CN**: 执行涉及 `memcpy` 的语句。
- **L1078 EN**: Executes statement involving `memcpy`.
  **L1078 CN**: 执行涉及 `memcpy` 的语句。
- **L1079 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1079 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1080 EN**: Executes statement involving `FreeMemory`.
  **L1080 CN**: 执行涉及 `FreeMemory` 的语句。
- **L1081 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1081 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1082 EN**: Blank line separates nearby declarations or logic blocks.
  **L1082 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1083 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1083 CN**: 延续周围的声明、表达式或控制流结构。
- **L1084 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1084 CN**: 延续周围的声明、表达式或控制流结构。
- **L1085 EN**: Executes statement `Terminator terminator{__FILE__, __LINE__};`.
  **L1085 CN**: 执行语句 `Terminator terminator{__FILE__, __LINE__};`。
- **L1086 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1086 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1087 EN**: Executes statement involving `raw`.
  **L1087 CN**: 执行涉及 `raw` 的语句。
- **L1088 EN**: Executes statement involving `set_base_addr`.
  **L1088 CN**: 执行涉及 `set_base_addr` 的语句。

### Lines 1089-1120

````cpp
  std::size_t oldLen{accumulator.ElementBytes()};
  accumulator.raw().elem_len += chars;
  RUNTIME_CHECK(
      terminator, accumulator.Allocate(kNoAsyncObject) == CFI_SUCCESS);
  std::memcpy(accumulator.OffsetElement<char>(oldLen), from, chars);
  FreeMemory(old);
}

int RTDEF(CharacterCompareScalar)(const Descriptor &x, const Descriptor &y) {
  Terminator terminator{__FILE__, __LINE__};
  RUNTIME_CHECK(terminator, x.rank() == 0);
  RUNTIME_CHECK(terminator, y.rank() == 0);
  RUNTIME_CHECK(terminator, x.raw().type == y.raw().type);
  switch (x.raw().type) {
  case CFI_type_char:
    return CharacterScalarCompare<char>(x.OffsetElement<char>(),
        y.OffsetElement<char>(), x.ElementBytes(), y.ElementBytes());
  case CFI_type_char16_t:
    return CharacterScalarCompare<char16_t>(x.OffsetElement<char16_t>(),
        y.OffsetElement<char16_t>(), x.ElementBytes() >> 1,
        y.ElementBytes() >> 1);
  case CFI_type_char32_t:
    return CharacterScalarCompare<char32_t>(x.OffsetElement<char32_t>(),
        y.OffsetElement<char32_t>(), x.ElementBytes() >> 2,
        y.ElementBytes() >> 2);
  default:
    terminator.Crash("CharacterCompareScalar: bad string type code %d",
        static_cast<int>(x.raw().type));
  }
  return 0;
}

````

- **L1089 EN**: Executes statement involving `ElementBytes`.
  **L1089 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1090 EN**: Initializes or updates `+`.
  **L1090 CN**: 初始化或更新 `+`。
- **L1091 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1091 CN**: 延续周围的声明、表达式或控制流结构。
- **L1092 EN**: Executes statement involving `Allocate`.
  **L1092 CN**: 执行涉及 `Allocate` 的语句。
- **L1093 EN**: Executes statement involving `memcpy`.
  **L1093 CN**: 执行涉及 `memcpy` 的语句。
- **L1094 EN**: Executes statement involving `FreeMemory`.
  **L1094 CN**: 执行涉及 `FreeMemory` 的语句。
- **L1095 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1095 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1096 EN**: Blank line separates nearby declarations or logic blocks.
  **L1096 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1097 EN**: Declares or defines callable `RTDEF`.
  **L1097 CN**: 声明或定义可调用实体 `RTDEF`。
- **L1098 EN**: Executes statement `Terminator terminator{__FILE__, __LINE__};`.
  **L1098 CN**: 执行语句 `Terminator terminator{__FILE__, __LINE__};`。
- **L1099 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1099 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1100 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1100 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1101 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1101 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1102 EN**: Begins a `switch` dispatch over discrete cases.
  **L1102 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1103 EN**: Marks one `switch` case label.
  **L1103 CN**: 标记一个 `switch` 的 case 标签。
- **L1104 EN**: Returns from the current function, often propagating a computed result.
  **L1104 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1105 EN**: Executes statement involving `ElementBytes`.
  **L1105 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1106 EN**: Marks one `switch` case label.
  **L1106 CN**: 标记一个 `switch` 的 case 标签。
- **L1107 EN**: Returns from the current function, often propagating a computed result.
  **L1107 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1108 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1108 CN**: 延续周围的声明、表达式或控制流结构。
- **L1109 EN**: Executes statement involving `ElementBytes`.
  **L1109 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1110 EN**: Marks one `switch` case label.
  **L1110 CN**: 标记一个 `switch` 的 case 标签。
- **L1111 EN**: Returns from the current function, often propagating a computed result.
  **L1111 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1112 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1112 CN**: 延续周围的声明、表达式或控制流结构。
- **L1113 EN**: Executes statement involving `ElementBytes`.
  **L1113 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1114 EN**: Provides the default branch for a `switch` statement.
  **L1114 CN**: 为 `switch` 语句提供默认分支。
- **L1115 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1115 CN**: 延续周围的声明、表达式或控制流结构。
- **L1116 EN**: Executes statement involving `raw`.
  **L1116 CN**: 执行涉及 `raw` 的语句。
- **L1117 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1117 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1118 EN**: Returns from the current function, often propagating a computed result.
  **L1118 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1119 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1119 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1120 EN**: Blank line separates nearby declarations or logic blocks.
  **L1120 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 1121-1152

````cpp
int RTDEF(CharacterCompareScalar1)(
    const char *x, const char *y, std::size_t xChars, std::size_t yChars) {
  return CharacterScalarCompare(x, y, xChars, yChars);
}

int RTDEF(CharacterCompareScalar2)(const char16_t *x, const char16_t *y,
    std::size_t xChars, std::size_t yChars) {
  return CharacterScalarCompare(x, y, xChars, yChars);
}

int RTDEF(CharacterCompareScalar4)(const char32_t *x, const char32_t *y,
    std::size_t xChars, std::size_t yChars) {
  return CharacterScalarCompare(x, y, xChars, yChars);
}

void RTDEF(CharacterCompare)(
    Descriptor &result, const Descriptor &x, const Descriptor &y) {
  Terminator terminator{__FILE__, __LINE__};
  RUNTIME_CHECK(terminator, x.raw().type == y.raw().type);
  switch (x.raw().type) {
  case CFI_type_char:
    Compare<char>(result, x, y, terminator);
    break;
  case CFI_type_char16_t:
    Compare<char16_t>(result, x, y, terminator);
    break;
  case CFI_type_char32_t:
    Compare<char32_t>(result, x, y, terminator);
    break;
  default:
    terminator.Crash("CharacterCompareScalar: bad string type code %d",
        static_cast<int>(x.raw().type));
````

- **L1121 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1121 CN**: 延续周围的声明、表达式或控制流结构。
- **L1122 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1122 CN**: 延续周围的声明、表达式或控制流结构。
- **L1123 EN**: Returns from the current function, often propagating a computed result.
  **L1123 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1124 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1124 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1125 EN**: Blank line separates nearby declarations or logic blocks.
  **L1125 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1126 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1126 CN**: 延续周围的声明、表达式或控制流结构。
- **L1127 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1127 CN**: 延续周围的声明、表达式或控制流结构。
- **L1128 EN**: Returns from the current function, often propagating a computed result.
  **L1128 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1129 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1129 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1130 EN**: Blank line separates nearby declarations or logic blocks.
  **L1130 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1131 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1131 CN**: 延续周围的声明、表达式或控制流结构。
- **L1132 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1132 CN**: 延续周围的声明、表达式或控制流结构。
- **L1133 EN**: Returns from the current function, often propagating a computed result.
  **L1133 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1134 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1134 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1135 EN**: Blank line separates nearby declarations or logic blocks.
  **L1135 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1136 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1136 CN**: 延续周围的声明、表达式或控制流结构。
- **L1137 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1137 CN**: 延续周围的声明、表达式或控制流结构。
- **L1138 EN**: Executes statement `Terminator terminator{__FILE__, __LINE__};`.
  **L1138 CN**: 执行语句 `Terminator terminator{__FILE__, __LINE__};`。
- **L1139 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1139 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1140 EN**: Begins a `switch` dispatch over discrete cases.
  **L1140 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1141 EN**: Marks one `switch` case label.
  **L1141 CN**: 标记一个 `switch` 的 case 标签。
- **L1142 EN**: Executes statement `Compare<char>(result, x, y, terminator);`.
  **L1142 CN**: 执行语句 `Compare<char>(result, x, y, terminator);`。
- **L1143 EN**: Breaks out of the current loop or switch.
  **L1143 CN**: 跳出当前循环或 switch。
- **L1144 EN**: Marks one `switch` case label.
  **L1144 CN**: 标记一个 `switch` 的 case 标签。
- **L1145 EN**: Executes statement `Compare<char16_t>(result, x, y, terminator);`.
  **L1145 CN**: 执行语句 `Compare<char16_t>(result, x, y, terminator);`。
- **L1146 EN**: Breaks out of the current loop or switch.
  **L1146 CN**: 跳出当前循环或 switch。
- **L1147 EN**: Marks one `switch` case label.
  **L1147 CN**: 标记一个 `switch` 的 case 标签。
- **L1148 EN**: Executes statement `Compare<char32_t>(result, x, y, terminator);`.
  **L1148 CN**: 执行语句 `Compare<char32_t>(result, x, y, terminator);`。
- **L1149 EN**: Breaks out of the current loop or switch.
  **L1149 CN**: 跳出当前循环或 switch。
- **L1150 EN**: Provides the default branch for a `switch` statement.
  **L1150 CN**: 为 `switch` 语句提供默认分支。
- **L1151 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1151 CN**: 延续周围的声明、表达式或控制流结构。
- **L1152 EN**: Executes statement involving `raw`.
  **L1152 CN**: 执行涉及 `raw` 的语句。

### Lines 1153-1184

````cpp
  }
}

std::size_t RTDEF(CharacterAppend1)(char *lhs, std::size_t lhsBytes,
    std::size_t offset, const char *rhs, std::size_t rhsBytes) {
  if (auto n{std::min(lhsBytes - offset, rhsBytes)}) {
    runtime::memcpy(lhs + offset, rhs, n);
    offset += n;
  }
  return offset;
}

void RTDEF(CharacterPad1)(char *lhs, std::size_t bytes, std::size_t offset) {
  if (bytes > offset) {
    runtime::memset(lhs + offset, ' ', bytes - offset);
  }
}

// Intrinsic function entry points

void RTDEF(Adjustl)(Descriptor &result, const Descriptor &string,
    const char *sourceFile, int sourceLine) {
  AdjustLR<false>(result, string, sourceFile, sourceLine);
}

void RTDEF(Adjustr)(Descriptor &result, const Descriptor &string,
    const char *sourceFile, int sourceLine) {
  AdjustLR<true>(result, string, sourceFile, sourceLine);
}

std::size_t RTDEF(Index1)(const char *x, std::size_t xLen, const char *set,
    std::size_t setLen, bool back) {
````

- **L1153 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1153 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1154 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1154 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1155 EN**: Blank line separates nearby declarations or logic blocks.
  **L1155 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1156 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1156 CN**: 延续周围的声明、表达式或控制流结构。
- **L1157 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1157 CN**: 延续周围的声明、表达式或控制流结构。
- **L1158 EN**: Introduces conditional control flow with an `if` statement.
  **L1158 CN**: 通过 `if` 语句引入条件控制流。
- **L1159 EN**: Executes statement involving `memcpy`.
  **L1159 CN**: 执行涉及 `memcpy` 的语句。
- **L1160 EN**: Initializes or updates `+`.
  **L1160 CN**: 初始化或更新 `+`。
- **L1161 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1161 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1162 EN**: Returns from the current function, often propagating a computed result.
  **L1162 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1163 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1163 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1164 EN**: Blank line separates nearby declarations or logic blocks.
  **L1164 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1165 EN**: Declares or defines callable `RTDEF`.
  **L1165 CN**: 声明或定义可调用实体 `RTDEF`。
- **L1166 EN**: Introduces conditional control flow with an `if` statement.
  **L1166 CN**: 通过 `if` 语句引入条件控制流。
- **L1167 EN**: Executes statement involving `memset`.
  **L1167 CN**: 执行涉及 `memset` 的语句。
- **L1168 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1168 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1169 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1169 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1170 EN**: Blank line separates nearby declarations or logic blocks.
  **L1170 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1171 EN**: Comment documents intent or context: `Intrinsic function entry points`.
  **L1171 CN**: 注释记录了意图或上下文：`Intrinsic function entry points`。
- **L1172 EN**: Blank line separates nearby declarations or logic blocks.
  **L1172 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1173 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1173 CN**: 延续周围的声明、表达式或控制流结构。
- **L1174 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1174 CN**: 延续周围的声明、表达式或控制流结构。
- **L1175 EN**: Executes statement `AdjustLR<false>(result, string, sourceFile, sourceLine);`.
  **L1175 CN**: 执行语句 `AdjustLR<false>(result, string, sourceFile, sourceLine);`。
- **L1176 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1176 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1177 EN**: Blank line separates nearby declarations or logic blocks.
  **L1177 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1178 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1178 CN**: 延续周围的声明、表达式或控制流结构。
- **L1179 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1179 CN**: 延续周围的声明、表达式或控制流结构。
- **L1180 EN**: Executes statement `AdjustLR<true>(result, string, sourceFile, sourceLine);`.
  **L1180 CN**: 执行语句 `AdjustLR<true>(result, string, sourceFile, sourceLine);`。
- **L1181 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1181 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1182 EN**: Blank line separates nearby declarations or logic blocks.
  **L1182 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1183 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1183 CN**: 延续周围的声明、表达式或控制流结构。
- **L1184 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1184 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 1185-1216

````cpp
  return Index<char>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Index2)(const char16_t *x, std::size_t xLen,
    const char16_t *set, std::size_t setLen, bool back) {
  return Index<char16_t>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Index4)(const char32_t *x, std::size_t xLen,
    const char32_t *set, std::size_t setLen, bool back) {
  return Index<char32_t>(x, xLen, set, setLen, back);
}

void RTDEF(Index)(Descriptor &result, const Descriptor &string,
    const Descriptor &substring, const Descriptor *back, int kind,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  switch (string.raw().type) {
  case CFI_type_char:
    GeneralCharFuncKind<char, CharFunc::Index>(
        result, string, substring, back, kind, terminator);
    break;
  case CFI_type_char16_t:
    GeneralCharFuncKind<char16_t, CharFunc::Index>(
        result, string, substring, back, kind, terminator);
    break;
  case CFI_type_char32_t:
    GeneralCharFuncKind<char32_t, CharFunc::Index>(
        result, string, substring, back, kind, terminator);
    break;
  default:
    terminator.Crash(
        "INDEX: bad string type code %d", static_cast<int>(string.raw().type));
  }
````

- **L1185 EN**: Returns from the current function, often propagating a computed result.
  **L1185 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1186 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1186 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1187 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1187 CN**: 延续周围的声明、表达式或控制流结构。
- **L1188 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1188 CN**: 延续周围的声明、表达式或控制流结构。
- **L1189 EN**: Returns from the current function, often propagating a computed result.
  **L1189 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1190 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1190 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1191 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1191 CN**: 延续周围的声明、表达式或控制流结构。
- **L1192 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1192 CN**: 延续周围的声明、表达式或控制流结构。
- **L1193 EN**: Returns from the current function, often propagating a computed result.
  **L1193 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1194 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1194 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1195 EN**: Blank line separates nearby declarations or logic blocks.
  **L1195 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1196 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1196 CN**: 延续周围的声明、表达式或控制流结构。
- **L1197 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1197 CN**: 延续周围的声明、表达式或控制流结构。
- **L1198 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1198 CN**: 延续周围的声明、表达式或控制流结构。
- **L1199 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1199 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1200 EN**: Begins a `switch` dispatch over discrete cases.
  **L1200 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1201 EN**: Marks one `switch` case label.
  **L1201 CN**: 标记一个 `switch` 的 case 标签。
- **L1202 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1202 CN**: 延续周围的声明、表达式或控制流结构。
- **L1203 EN**: Executes statement `result, string, substring, back, kind, terminator);`.
  **L1203 CN**: 执行语句 `result, string, substring, back, kind, terminator);`。
- **L1204 EN**: Breaks out of the current loop or switch.
  **L1204 CN**: 跳出当前循环或 switch。
- **L1205 EN**: Marks one `switch` case label.
  **L1205 CN**: 标记一个 `switch` 的 case 标签。
- **L1206 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1206 CN**: 延续周围的声明、表达式或控制流结构。
- **L1207 EN**: Executes statement `result, string, substring, back, kind, terminator);`.
  **L1207 CN**: 执行语句 `result, string, substring, back, kind, terminator);`。
- **L1208 EN**: Breaks out of the current loop or switch.
  **L1208 CN**: 跳出当前循环或 switch。
- **L1209 EN**: Marks one `switch` case label.
  **L1209 CN**: 标记一个 `switch` 的 case 标签。
- **L1210 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1210 CN**: 延续周围的声明、表达式或控制流结构。
- **L1211 EN**: Executes statement `result, string, substring, back, kind, terminator);`.
  **L1211 CN**: 执行语句 `result, string, substring, back, kind, terminator);`。
- **L1212 EN**: Breaks out of the current loop or switch.
  **L1212 CN**: 跳出当前循环或 switch。
- **L1213 EN**: Provides the default branch for a `switch` statement.
  **L1213 CN**: 为 `switch` 语句提供默认分支。
- **L1214 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1214 CN**: 延续周围的声明、表达式或控制流结构。
- **L1215 EN**: Executes statement involving `raw`.
  **L1215 CN**: 执行涉及 `raw` 的语句。
- **L1216 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1216 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 1217-1248

````cpp
}

std::size_t RTDEF(LenTrim1)(const char *x, std::size_t chars) {
  return LenTrim(x, chars);
}
std::size_t RTDEF(LenTrim2)(const char16_t *x, std::size_t chars) {
  return LenTrim(x, chars);
}
std::size_t RTDEF(LenTrim4)(const char32_t *x, std::size_t chars) {
  return LenTrim(x, chars);
}

void RTDEF(LenTrim)(Descriptor &result, const Descriptor &string, int kind,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  switch (string.raw().type) {
  case CFI_type_char:
    LenTrimKind<char>(result, string, kind, terminator);
    break;
  case CFI_type_char16_t:
    LenTrimKind<char16_t>(result, string, kind, terminator);
    break;
  case CFI_type_char32_t:
    LenTrimKind<char32_t>(result, string, kind, terminator);
    break;
  default:
    terminator.Crash("LEN_TRIM: bad string type code %d",
        static_cast<int>(string.raw().type));
  }
}

std::size_t RTDEF(Scan1)(const char *x, std::size_t xLen, const char *set,
````

- **L1217 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1217 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1218 EN**: Blank line separates nearby declarations or logic blocks.
  **L1218 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1219 EN**: Declares or defines callable `RTDEF`.
  **L1219 CN**: 声明或定义可调用实体 `RTDEF`。
- **L1220 EN**: Returns from the current function, often propagating a computed result.
  **L1220 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1221 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1221 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1222 EN**: Declares or defines callable `RTDEF`.
  **L1222 CN**: 声明或定义可调用实体 `RTDEF`。
- **L1223 EN**: Returns from the current function, often propagating a computed result.
  **L1223 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1224 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1224 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1225 EN**: Declares or defines callable `RTDEF`.
  **L1225 CN**: 声明或定义可调用实体 `RTDEF`。
- **L1226 EN**: Returns from the current function, often propagating a computed result.
  **L1226 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1227 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1227 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1228 EN**: Blank line separates nearby declarations or logic blocks.
  **L1228 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1229 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1229 CN**: 延续周围的声明、表达式或控制流结构。
- **L1230 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1230 CN**: 延续周围的声明、表达式或控制流结构。
- **L1231 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1231 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1232 EN**: Begins a `switch` dispatch over discrete cases.
  **L1232 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1233 EN**: Marks one `switch` case label.
  **L1233 CN**: 标记一个 `switch` 的 case 标签。
- **L1234 EN**: Executes statement `LenTrimKind<char>(result, string, kind, terminator);`.
  **L1234 CN**: 执行语句 `LenTrimKind<char>(result, string, kind, terminator);`。
- **L1235 EN**: Breaks out of the current loop or switch.
  **L1235 CN**: 跳出当前循环或 switch。
- **L1236 EN**: Marks one `switch` case label.
  **L1236 CN**: 标记一个 `switch` 的 case 标签。
- **L1237 EN**: Executes statement `LenTrimKind<char16_t>(result, string, kind, terminator);`.
  **L1237 CN**: 执行语句 `LenTrimKind<char16_t>(result, string, kind, terminator);`。
- **L1238 EN**: Breaks out of the current loop or switch.
  **L1238 CN**: 跳出当前循环或 switch。
- **L1239 EN**: Marks one `switch` case label.
  **L1239 CN**: 标记一个 `switch` 的 case 标签。
- **L1240 EN**: Executes statement `LenTrimKind<char32_t>(result, string, kind, terminator);`.
  **L1240 CN**: 执行语句 `LenTrimKind<char32_t>(result, string, kind, terminator);`。
- **L1241 EN**: Breaks out of the current loop or switch.
  **L1241 CN**: 跳出当前循环或 switch。
- **L1242 EN**: Provides the default branch for a `switch` statement.
  **L1242 CN**: 为 `switch` 语句提供默认分支。
- **L1243 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1243 CN**: 延续周围的声明、表达式或控制流结构。
- **L1244 EN**: Executes statement involving `raw`.
  **L1244 CN**: 执行涉及 `raw` 的语句。
- **L1245 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1245 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1246 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1246 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1247 EN**: Blank line separates nearby declarations or logic blocks.
  **L1247 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1248 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1248 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 1249-1280

````cpp
    std::size_t setLen, bool back) {
  return ScanVerify<false>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Scan2)(const char16_t *x, std::size_t xLen,
    const char16_t *set, std::size_t setLen, bool back) {
  return ScanVerify<char16_t, CharFunc::Scan>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Scan4)(const char32_t *x, std::size_t xLen,
    const char32_t *set, std::size_t setLen, bool back) {
  return ScanVerify<char32_t, CharFunc::Scan>(x, xLen, set, setLen, back);
}

void RTDEF(Scan)(Descriptor &result, const Descriptor &string,
    const Descriptor &set, const Descriptor *back, int kind,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  switch (string.raw().type) {
  case CFI_type_char:
    GeneralCharFuncKind<char, CharFunc::Scan>(
        result, string, set, back, kind, terminator);
    break;
  case CFI_type_char16_t:
    GeneralCharFuncKind<char16_t, CharFunc::Scan>(
        result, string, set, back, kind, terminator);
    break;
  case CFI_type_char32_t:
    GeneralCharFuncKind<char32_t, CharFunc::Scan>(
        result, string, set, back, kind, terminator);
    break;
  default:
    terminator.Crash(
        "SCAN: bad string type code %d", static_cast<int>(string.raw().type));
````

- **L1249 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1249 CN**: 延续周围的声明、表达式或控制流结构。
- **L1250 EN**: Returns from the current function, often propagating a computed result.
  **L1250 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1251 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1251 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1252 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1252 CN**: 延续周围的声明、表达式或控制流结构。
- **L1253 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1253 CN**: 延续周围的声明、表达式或控制流结构。
- **L1254 EN**: Returns from the current function, often propagating a computed result.
  **L1254 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1255 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1255 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1256 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1256 CN**: 延续周围的声明、表达式或控制流结构。
- **L1257 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1257 CN**: 延续周围的声明、表达式或控制流结构。
- **L1258 EN**: Returns from the current function, often propagating a computed result.
  **L1258 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1259 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1259 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1260 EN**: Blank line separates nearby declarations or logic blocks.
  **L1260 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1261 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1261 CN**: 延续周围的声明、表达式或控制流结构。
- **L1262 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1262 CN**: 延续周围的声明、表达式或控制流结构。
- **L1263 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1263 CN**: 延续周围的声明、表达式或控制流结构。
- **L1264 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1264 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1265 EN**: Begins a `switch` dispatch over discrete cases.
  **L1265 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1266 EN**: Marks one `switch` case label.
  **L1266 CN**: 标记一个 `switch` 的 case 标签。
- **L1267 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1267 CN**: 延续周围的声明、表达式或控制流结构。
- **L1268 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1268 CN**: 执行语句 `result, string, set, back, kind, terminator);`。
- **L1269 EN**: Breaks out of the current loop or switch.
  **L1269 CN**: 跳出当前循环或 switch。
- **L1270 EN**: Marks one `switch` case label.
  **L1270 CN**: 标记一个 `switch` 的 case 标签。
- **L1271 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1271 CN**: 延续周围的声明、表达式或控制流结构。
- **L1272 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1272 CN**: 执行语句 `result, string, set, back, kind, terminator);`。
- **L1273 EN**: Breaks out of the current loop or switch.
  **L1273 CN**: 跳出当前循环或 switch。
- **L1274 EN**: Marks one `switch` case label.
  **L1274 CN**: 标记一个 `switch` 的 case 标签。
- **L1275 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1275 CN**: 延续周围的声明、表达式或控制流结构。
- **L1276 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1276 CN**: 执行语句 `result, string, set, back, kind, terminator);`。
- **L1277 EN**: Breaks out of the current loop or switch.
  **L1277 CN**: 跳出当前循环或 switch。
- **L1278 EN**: Provides the default branch for a `switch` statement.
  **L1278 CN**: 为 `switch` 语句提供默认分支。
- **L1279 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1279 CN**: 延续周围的声明、表达式或控制流结构。
- **L1280 EN**: Executes statement involving `raw`.
  **L1280 CN**: 执行涉及 `raw` 的语句。

### Lines 1281-1312

````cpp
  }
}

void RTDEF(Repeat)(Descriptor &result, const Descriptor &string,
    std::int64_t ncopies, const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  if (ncopies < 0) {
    terminator.Crash(
        "REPEAT has negative NCOPIES=%jd", static_cast<std::intmax_t>(ncopies));
  }
  std::size_t origBytes{string.ElementBytes()};
  result.Establish(string.type(), origBytes * ncopies, nullptr, 0, nullptr,
      CFI_attribute_allocatable);
  if (result.Allocate(kNoAsyncObject) != CFI_SUCCESS) {
    terminator.Crash("REPEAT could not allocate storage for result");
  }
  const char *from{string.OffsetElement()};
  for (char *to{result.OffsetElement()}; ncopies-- > 0; to += origBytes) {
    runtime::memcpy(to, from, origBytes);
  }
}

// F_C_STRING - Appends null terminator to create C-compatible string
// If asis is false, trailing blanks are trimmed first
void RTDEF(FCString)(Descriptor &result, const Descriptor &string, bool asis,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  RUNTIME_CHECK(terminator, string.raw().type == CFI_type_char);
  std::size_t chars{string.ElementBytes()};
  if (!asis) {
    chars = LenTrim(string.OffsetElement<const char>(), chars);
  }
````

- **L1281 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1281 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1282 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1282 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1283 EN**: Blank line separates nearby declarations or logic blocks.
  **L1283 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1284 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1284 CN**: 延续周围的声明、表达式或控制流结构。
- **L1285 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1285 CN**: 延续周围的声明、表达式或控制流结构。
- **L1286 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1286 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1287 EN**: Introduces conditional control flow with an `if` statement.
  **L1287 CN**: 通过 `if` 语句引入条件控制流。
- **L1288 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1288 CN**: 延续周围的声明、表达式或控制流结构。
- **L1289 EN**: Executes statement `"REPEAT has negative NCOPIES=%jd", static_cast<std::intmax_t>(ncopies));`.
  **L1289 CN**: 执行语句 `"REPEAT has negative NCOPIES=%jd", static_cast<std::intmax_t>(ncopies));`。
- **L1290 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1290 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1291 EN**: Executes statement involving `ElementBytes`.
  **L1291 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1292 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1292 CN**: 延续周围的声明、表达式或控制流结构。
- **L1293 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L1293 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L1294 EN**: Introduces conditional control flow with an `if` statement.
  **L1294 CN**: 通过 `if` 语句引入条件控制流。
- **L1295 EN**: Executes statement involving `Crash`.
  **L1295 CN**: 执行涉及 `Crash` 的语句。
- **L1296 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1296 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1297 EN**: Executes statement involving `OffsetElement`.
  **L1297 CN**: 执行涉及 `OffsetElement` 的语句。
- **L1298 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L1298 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L1299 EN**: Executes statement involving `memcpy`.
  **L1299 CN**: 执行涉及 `memcpy` 的语句。
- **L1300 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1300 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1301 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1301 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1302 EN**: Blank line separates nearby declarations or logic blocks.
  **L1302 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1303 EN**: Comment documents intent or context: `F_C_STRING - Appends null terminator to create C-compatible string`.
  **L1303 CN**: 注释记录了意图或上下文：`F_C_STRING - Appends null terminator to create C-compatible string`。
- **L1304 EN**: Comment documents intent or context: `If asis is false, trailing blanks are trimmed first`.
  **L1304 CN**: 注释记录了意图或上下文：`If asis is false, trailing blanks are trimmed first`。
- **L1305 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1305 CN**: 延续周围的声明、表达式或控制流结构。
- **L1306 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1306 CN**: 延续周围的声明、表达式或控制流结构。
- **L1307 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1307 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1308 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1308 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1309 EN**: Executes statement involving `ElementBytes`.
  **L1309 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1310 EN**: Introduces conditional control flow with an `if` statement.
  **L1310 CN**: 通过 `if` 语句引入条件控制流。
- **L1311 EN**: Initializes or updates `chars`.
  **L1311 CN**: 初始化或更新 `chars`。
- **L1312 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1312 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 1313-1344

````cpp
  std::size_t resultBytes{chars + 1};
  result.Establish(string.type(), resultBytes, nullptr, 0, nullptr,
      CFI_attribute_allocatable);
  RUNTIME_CHECK(terminator, result.Allocate(kNoAsyncObject) == CFI_SUCCESS);
  if (chars > 0) {
    std::memcpy(result.OffsetElement(), string.OffsetElement(), chars);
  }
  *result.OffsetElement<char>(chars) = '\0';
}

void RTDEF(Trim)(Descriptor &result, const Descriptor &string,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  std::size_t resultBytes{0};
  switch (string.raw().type) {
  case CFI_type_char:
    resultBytes =
        LenTrim(string.OffsetElement<const char>(), string.ElementBytes());
    break;
  case CFI_type_char16_t:
    resultBytes = LenTrim(string.OffsetElement<const char16_t>(),
                      string.ElementBytes() >> 1)
        << 1;
    break;
  case CFI_type_char32_t:
    resultBytes = LenTrim(string.OffsetElement<const char32_t>(),
                      string.ElementBytes() >> 2)
        << 2;
    break;
  default:
    terminator.Crash(
        "TRIM: bad string type code %d", static_cast<int>(string.raw().type));
````

- **L1313 EN**: Executes statement `std::size_t resultBytes{chars + 1};`.
  **L1313 CN**: 执行语句 `std::size_t resultBytes{chars + 1};`。
- **L1314 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1314 CN**: 延续周围的声明、表达式或控制流结构。
- **L1315 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L1315 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L1316 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1316 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1317 EN**: Introduces conditional control flow with an `if` statement.
  **L1317 CN**: 通过 `if` 语句引入条件控制流。
- **L1318 EN**: Executes statement involving `memcpy`.
  **L1318 CN**: 执行涉及 `memcpy` 的语句。
- **L1319 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1319 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1320 EN**: Comment documents intent or context: `result.OffsetElement<char>(chars) = '\0';`.
  **L1320 CN**: 注释记录了意图或上下文：`result.OffsetElement<char>(chars) = '\0';`。
- **L1321 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1321 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1322 EN**: Blank line separates nearby declarations or logic blocks.
  **L1322 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1323 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1323 CN**: 延续周围的声明、表达式或控制流结构。
- **L1324 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1324 CN**: 延续周围的声明、表达式或控制流结构。
- **L1325 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1325 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1326 EN**: Executes statement `std::size_t resultBytes{0};`.
  **L1326 CN**: 执行语句 `std::size_t resultBytes{0};`。
- **L1327 EN**: Begins a `switch` dispatch over discrete cases.
  **L1327 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1328 EN**: Marks one `switch` case label.
  **L1328 CN**: 标记一个 `switch` 的 case 标签。
- **L1329 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1329 CN**: 延续周围的声明、表达式或控制流结构。
- **L1330 EN**: Executes statement involving `LenTrim`.
  **L1330 CN**: 执行涉及 `LenTrim` 的语句。
- **L1331 EN**: Breaks out of the current loop or switch.
  **L1331 CN**: 跳出当前循环或 switch。
- **L1332 EN**: Marks one `switch` case label.
  **L1332 CN**: 标记一个 `switch` 的 case 标签。
- **L1333 EN**: Initializes or updates `resultBytes`.
  **L1333 CN**: 初始化或更新 `resultBytes`。
- **L1334 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1334 CN**: 延续周围的声明、表达式或控制流结构。
- **L1335 EN**: Executes statement `<< 1;`.
  **L1335 CN**: 执行语句 `<< 1;`。
- **L1336 EN**: Breaks out of the current loop or switch.
  **L1336 CN**: 跳出当前循环或 switch。
- **L1337 EN**: Marks one `switch` case label.
  **L1337 CN**: 标记一个 `switch` 的 case 标签。
- **L1338 EN**: Initializes or updates `resultBytes`.
  **L1338 CN**: 初始化或更新 `resultBytes`。
- **L1339 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1339 CN**: 延续周围的声明、表达式或控制流结构。
- **L1340 EN**: Executes statement `<< 2;`.
  **L1340 CN**: 执行语句 `<< 2;`。
- **L1341 EN**: Breaks out of the current loop or switch.
  **L1341 CN**: 跳出当前循环或 switch。
- **L1342 EN**: Provides the default branch for a `switch` statement.
  **L1342 CN**: 为 `switch` 语句提供默认分支。
- **L1343 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1343 CN**: 延续周围的声明、表达式或控制流结构。
- **L1344 EN**: Executes statement involving `raw`.
  **L1344 CN**: 执行涉及 `raw` 的语句。

### Lines 1345-1376

````cpp
  }
  result.Establish(string.type(), resultBytes, nullptr, 0, nullptr,
      CFI_attribute_allocatable);
  RUNTIME_CHECK(terminator, result.Allocate(kNoAsyncObject) == CFI_SUCCESS);
  std::memcpy(result.OffsetElement(), string.OffsetElement(), resultBytes);
}

std::size_t RTDEF(Verify1)(const char *x, std::size_t xLen, const char *set,
    std::size_t setLen, bool back) {
  return ScanVerify<true>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Verify2)(const char16_t *x, std::size_t xLen,
    const char16_t *set, std::size_t setLen, bool back) {
  return ScanVerify<char16_t, CharFunc::Verify>(x, xLen, set, setLen, back);
}
std::size_t RTDEF(Verify4)(const char32_t *x, std::size_t xLen,
    const char32_t *set, std::size_t setLen, bool back) {
  return ScanVerify<char32_t, CharFunc::Verify>(x, xLen, set, setLen, back);
}

void RTDEF(Verify)(Descriptor &result, const Descriptor &string,
    const Descriptor &set, const Descriptor *back, int kind,
    const char *sourceFile, int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  switch (string.raw().type) {
  case CFI_type_char:
    GeneralCharFuncKind<char, CharFunc::Verify>(
        result, string, set, back, kind, terminator);
    break;
  case CFI_type_char16_t:
    GeneralCharFuncKind<char16_t, CharFunc::Verify>(
        result, string, set, back, kind, terminator);
````

- **L1345 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1345 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1346 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1346 CN**: 延续周围的声明、表达式或控制流结构。
- **L1347 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L1347 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L1348 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L1348 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L1349 EN**: Executes statement involving `memcpy`.
  **L1349 CN**: 执行涉及 `memcpy` 的语句。
- **L1350 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1350 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1351 EN**: Blank line separates nearby declarations or logic blocks.
  **L1351 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1352 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1352 CN**: 延续周围的声明、表达式或控制流结构。
- **L1353 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1353 CN**: 延续周围的声明、表达式或控制流结构。
- **L1354 EN**: Returns from the current function, often propagating a computed result.
  **L1354 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1355 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1355 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1356 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1356 CN**: 延续周围的声明、表达式或控制流结构。
- **L1357 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1357 CN**: 延续周围的声明、表达式或控制流结构。
- **L1358 EN**: Returns from the current function, often propagating a computed result.
  **L1358 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1359 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1359 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1360 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1360 CN**: 延续周围的声明、表达式或控制流结构。
- **L1361 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1361 CN**: 延续周围的声明、表达式或控制流结构。
- **L1362 EN**: Returns from the current function, often propagating a computed result.
  **L1362 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1363 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1363 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1364 EN**: Blank line separates nearby declarations or logic blocks.
  **L1364 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1365 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1365 CN**: 延续周围的声明、表达式或控制流结构。
- **L1366 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1366 CN**: 延续周围的声明、表达式或控制流结构。
- **L1367 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1367 CN**: 延续周围的声明、表达式或控制流结构。
- **L1368 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1368 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1369 EN**: Begins a `switch` dispatch over discrete cases.
  **L1369 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L1370 EN**: Marks one `switch` case label.
  **L1370 CN**: 标记一个 `switch` 的 case 标签。
- **L1371 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1371 CN**: 延续周围的声明、表达式或控制流结构。
- **L1372 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1372 CN**: 执行语句 `result, string, set, back, kind, terminator);`。
- **L1373 EN**: Breaks out of the current loop or switch.
  **L1373 CN**: 跳出当前循环或 switch。
- **L1374 EN**: Marks one `switch` case label.
  **L1374 CN**: 标记一个 `switch` 的 case 标签。
- **L1375 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1375 CN**: 延续周围的声明、表达式或控制流结构。
- **L1376 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1376 CN**: 执行语句 `result, string, set, back, kind, terminator);`。

### Lines 1377-1408

````cpp
    break;
  case CFI_type_char32_t:
    GeneralCharFuncKind<char32_t, CharFunc::Verify>(
        result, string, set, back, kind, terminator);
    break;
  default:
    terminator.Crash(
        "VERIFY: bad string type code %d", static_cast<int>(string.raw().type));
  }
}

void RTDEF(CharacterMax)(Descriptor &accumulator, const Descriptor &x,
    const char *sourceFile, int sourceLine) {
  MaxMin<false>(accumulator, x, sourceFile, sourceLine);
}

void RTDEF(CharacterMin)(Descriptor &accumulator, const Descriptor &x,
    const char *sourceFile, int sourceLine) {
  MaxMin<true>(accumulator, x, sourceFile, sourceLine);
}

// TOKENIZE Form 1 entry point
void RTDEF(Tokenize)(Descriptor &tokens, Descriptor *separator,
    const Descriptor &string, const Descriptor &set, const char *sourceFile,
    int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  TokenizeImpl(tokens, separator, string, set, terminator);
}

// TOKENIZE Form 2 entry point
void RTDEF(TokenizePositions)(Descriptor &first, Descriptor &last,
    const Descriptor &string, const Descriptor &set, const char *sourceFile,
````

- **L1377 EN**: Breaks out of the current loop or switch.
  **L1377 CN**: 跳出当前循环或 switch。
- **L1378 EN**: Marks one `switch` case label.
  **L1378 CN**: 标记一个 `switch` 的 case 标签。
- **L1379 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1379 CN**: 延续周围的声明、表达式或控制流结构。
- **L1380 EN**: Executes statement `result, string, set, back, kind, terminator);`.
  **L1380 CN**: 执行语句 `result, string, set, back, kind, terminator);`。
- **L1381 EN**: Breaks out of the current loop or switch.
  **L1381 CN**: 跳出当前循环或 switch。
- **L1382 EN**: Provides the default branch for a `switch` statement.
  **L1382 CN**: 为 `switch` 语句提供默认分支。
- **L1383 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1383 CN**: 延续周围的声明、表达式或控制流结构。
- **L1384 EN**: Executes statement involving `raw`.
  **L1384 CN**: 执行涉及 `raw` 的语句。
- **L1385 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1385 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1386 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1386 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1387 EN**: Blank line separates nearby declarations or logic blocks.
  **L1387 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1388 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1388 CN**: 延续周围的声明、表达式或控制流结构。
- **L1389 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1389 CN**: 延续周围的声明、表达式或控制流结构。
- **L1390 EN**: Executes statement `MaxMin<false>(accumulator, x, sourceFile, sourceLine);`.
  **L1390 CN**: 执行语句 `MaxMin<false>(accumulator, x, sourceFile, sourceLine);`。
- **L1391 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1391 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1392 EN**: Blank line separates nearby declarations or logic blocks.
  **L1392 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1393 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1393 CN**: 延续周围的声明、表达式或控制流结构。
- **L1394 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1394 CN**: 延续周围的声明、表达式或控制流结构。
- **L1395 EN**: Executes statement `MaxMin<true>(accumulator, x, sourceFile, sourceLine);`.
  **L1395 CN**: 执行语句 `MaxMin<true>(accumulator, x, sourceFile, sourceLine);`。
- **L1396 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1396 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1397 EN**: Blank line separates nearby declarations or logic blocks.
  **L1397 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1398 EN**: Comment documents intent or context: `TOKENIZE Form 1 entry point`.
  **L1398 CN**: 注释记录了意图或上下文：`TOKENIZE Form 1 entry point`。
- **L1399 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1399 CN**: 延续周围的声明、表达式或控制流结构。
- **L1400 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1400 CN**: 延续周围的声明、表达式或控制流结构。
- **L1401 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1401 CN**: 延续周围的声明、表达式或控制流结构。
- **L1402 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1402 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1403 EN**: Executes statement involving `TokenizeImpl`.
  **L1403 CN**: 执行涉及 `TokenizeImpl` 的语句。
- **L1404 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1404 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1405 EN**: Blank line separates nearby declarations or logic blocks.
  **L1405 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1406 EN**: Comment documents intent or context: `TOKENIZE Form 2 entry point`.
  **L1406 CN**: 注释记录了意图或上下文：`TOKENIZE Form 2 entry point`。
- **L1407 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1407 CN**: 延续周围的声明、表达式或控制流结构。
- **L1408 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1408 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 1409-1431

````cpp
    int sourceLine) {
  Terminator terminator{sourceFile, sourceLine};
  TokenizePositionsImpl(first, last, string, set, terminator);
}

std::size_t RTDEF(Split1)(const char *string, std::size_t stringLen,
    const char *set, std::size_t setLen, std::size_t pos, bool back) {
  return SplitImpl(string, stringLen, set, setLen, pos, back);
}

std::size_t RTDEF(Split2)(const char16_t *string, std::size_t stringLen,
    const char16_t *set, std::size_t setLen, std::size_t pos, bool back) {
  return SplitImpl(string, stringLen, set, setLen, pos, back);
}

std::size_t RTDEF(Split4)(const char32_t *string, std::size_t stringLen,
    const char32_t *set, std::size_t setLen, std::size_t pos, bool back) {
  return SplitImpl(string, stringLen, set, setLen, pos, back);
}

RT_EXT_API_GROUP_END
}
} // namespace Fortran::runtime
````

- **L1409 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1409 CN**: 延续周围的声明、表达式或控制流结构。
- **L1410 EN**: Executes statement `Terminator terminator{sourceFile, sourceLine};`.
  **L1410 CN**: 执行语句 `Terminator terminator{sourceFile, sourceLine};`。
- **L1411 EN**: Executes statement involving `TokenizePositionsImpl`.
  **L1411 CN**: 执行涉及 `TokenizePositionsImpl` 的语句。
- **L1412 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1412 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1413 EN**: Blank line separates nearby declarations or logic blocks.
  **L1413 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1414 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1414 CN**: 延续周围的声明、表达式或控制流结构。
- **L1415 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1415 CN**: 延续周围的声明、表达式或控制流结构。
- **L1416 EN**: Returns from the current function, often propagating a computed result.
  **L1416 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1417 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1417 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1418 EN**: Blank line separates nearby declarations or logic blocks.
  **L1418 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1419 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1419 CN**: 延续周围的声明、表达式或控制流结构。
- **L1420 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1420 CN**: 延续周围的声明、表达式或控制流结构。
- **L1421 EN**: Returns from the current function, often propagating a computed result.
  **L1421 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1422 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1422 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1423 EN**: Blank line separates nearby declarations or logic blocks.
  **L1423 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1424 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1424 CN**: 延续周围的声明、表达式或控制流结构。
- **L1425 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1425 CN**: 延续周围的声明、表达式或控制流结构。
- **L1426 EN**: Returns from the current function, often propagating a computed result.
  **L1426 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1427 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1427 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1428 EN**: Blank line separates nearby declarations or logic blocks.
  **L1428 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1429 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1429 CN**: 延续周围的声明、表达式或控制流结构。
- **L1430 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1430 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1431 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1431 CN**: 延续周围的声明、表达式或控制流结构。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 1431 source lines, which suggests a substantial implementation unit. / 该文件约有 1431 行源码，说明它是一个较大的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `flang/Runtime/character.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/terminator.h`, `flang-rt/runtime/tools.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `flang/Runtime/character.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/terminator.h`, `flang-rt/runtime/tools.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `constexpr`, `Adjust`, `LenTrim`, `RTDEF`. / 值得关注的可调用实体包括 `constexpr`, `Adjust`, `LenTrim`, `RTDEF`。
- **Core types / 核心类型**: Important declared or referenced types include `UNSIGNED_CHAR`, `CharFunc`, `TokenizeAnalysis`, `TokenizeStoreIntFn`. / 重要的已声明或被引用类型包括 `UNSIGNED_CHAR`, `CharFunc`, `TokenizeAnalysis`, `TokenizeStoreIntFn`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `flang/Runtime/character.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/terminator.h`, `flang-rt/runtime/tools.h`, `flang/Common/bit-population-count.h`, `flang/Common/uint128.h`, `flang/Runtime/cpp-type.h`, `flang/Runtime/freestanding-tools.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Standard or platform headers / 标准库或平台头文件**: `algorithm`, `cstring`. They connect the file to language-runtime or OS services. / 它们将该文件连接到语言运行时或操作系统服务。
- **Callable surface / 可调用表面**: Functions or methods defined here include `constexpr`, `Adjust`, `LenTrim`, `RTDEF`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `constexpr`, `Adjust`, `LenTrim`, `RTDEF`，它们通常是对周边代码暴露的主要入口。
- **Type coupling / 类型耦合**: Declared types such as `UNSIGNED_CHAR`, `CharFunc`, `TokenizeAnalysis`, `TokenizeStoreIntFn` capture the data model shared with dependent code. / `UNSIGNED_CHAR`, `CharFunc`, `TokenizeAnalysis`, `TokenizeStoreIntFn` 等声明类型体现了与依赖方共享的数据模型。
