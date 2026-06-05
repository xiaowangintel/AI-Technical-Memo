# extrema.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/extrema.cpp` | `flang-rt/lib/runtime/extrema.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. In this file, the main focus is `extrema`; the header comment highlights: Implements MAXLOC, MINLOC, MAXVAL, & MINVAL for all required operand types and shapes and (for MAXLOC & MINLOC) result integer kinds. Also implements NORM2 using common infrastructure.. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件的核心主题是 `extrema`；文件头注释强调：Implements MAXLOC, MINLOC, MAXVAL, & MINVAL for all required operand types and shapes and (for MAXLOC & MINLOC) result integer kinds. Also implements NORM2 using common infrastructure.。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/runtime/extrema.cpp ---------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// Implements MAXLOC, MINLOC, MAXVAL, & MINVAL for all required operand types
// and shapes and (for MAXLOC & MINLOC) result integer kinds.  Also implements
// NORM2 using common infrastructure.

#include "flang-rt/runtime/reduction-templates.h"
#include "flang/Common/float128.h"
#include "flang/Runtime/character.h"
#include "flang/Runtime/reduction.h"
#include <algorithm>
#include <cfloat>
#include <cinttypes>
#include <cmath>
#include <type_traits>

namespace Fortran::runtime {

````

- **L1 EN**: Comment documents intent or context: `lib/runtime/extrema.cpp ---------------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/extrema.cpp ---------------------------------*- C++ -*-===//`。
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
- **L9 EN**: Comment documents intent or context: `Implements MAXLOC, MINLOC, MAXVAL, & MINVAL for all required operand types`.
  **L9 CN**: 注释记录了意图或上下文：`Implements MAXLOC, MINLOC, MAXVAL, & MINVAL for all required operand types`。
- **L10 EN**: Comment documents intent or context: `and shapes and (for MAXLOC & MINLOC) result integer kinds. Also implements`.
  **L10 CN**: 注释记录了意图或上下文：`and shapes and (for MAXLOC & MINLOC) result integer kinds. Also implements`。
- **L11 EN**: Comment documents intent or context: `NORM2 using common infrastructure.`.
  **L11 CN**: 注释记录了意图或上下文：`NORM2 using common infrastructure.`。
- **L12 EN**: Blank line separates nearby declarations or logic blocks.
  **L12 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes `flang-rt/runtime/reduction-templates.h` to access Flang runtime public headers.
  **L13 CN**: 引入 `flang-rt/runtime/reduction-templates.h` 以使用 Flang 运行时公共头文件。
- **L14 EN**: Includes `flang/Common/float128.h` to access Flang common data structures and compiler-wide helpers.
  **L14 CN**: 引入 `flang/Common/float128.h` 以使用 Flang 通用数据结构与编译器级辅助工具。
- **L15 EN**: Includes `flang/Runtime/character.h` to access Flang runtime declarations.
  **L15 CN**: 引入 `flang/Runtime/character.h` 以使用 Flang 运行时声明。
- **L16 EN**: Includes `flang/Runtime/reduction.h` to access Flang runtime declarations.
  **L16 CN**: 引入 `flang/Runtime/reduction.h` 以使用 Flang 运行时声明。
- **L17 EN**: Includes `algorithm` to access standard algorithms and helpers.
  **L17 CN**: 引入 `algorithm` 以使用 标准算法与辅助工具。
- **L18 EN**: Includes `cfloat` to access floating-point limits.
  **L18 CN**: 引入 `cfloat` 以使用 浮点数范围定义。
- **L19 EN**: Includes `cinttypes` to access fixed-width integer formatting.
  **L19 CN**: 引入 `cinttypes` 以使用 定宽整数格式化支持。
- **L20 EN**: Includes `cmath` to access math functions.
  **L20 CN**: 引入 `cmath` 以使用 数学函数。
- **L21 EN**: Includes `type_traits` to access compile-time type traits.
  **L21 CN**: 引入 `type_traits` 以使用 编译期类型特征。
- **L22 EN**: Blank line separates nearby declarations or logic blocks.
  **L22 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L23 EN**: Enters namespace `Fortran` to scope related declarations.
  **L23 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L24 EN**: Blank line separates nearby declarations or logic blocks.
  **L24 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 25-48

````cpp
// MAXLOC & MINLOC

template <typename T, bool IS_MAX> struct NumericCompare {
  using Type = T;
  explicit RT_API_ATTRS NumericCompare(std::size_t /*elemLen*/, bool back)
      : back_{back} {}
  RT_API_ATTRS bool operator()(const T &value, const T &previous) const {
    if (std::is_floating_point_v<T> && previous != previous) {
      return back_ || value == value; // replace NaN
    } else if (value == previous) {
      return back_;
    } else if constexpr (IS_MAX) {
      return value > previous;
    } else {
      return value < previous;
    }
  }

private:
  bool back_;
};

template <typename T, bool IS_MAX> class CharacterCompare {
public:
````

- **L25 EN**: Comment documents intent or context: `MAXLOC & MINLOC`.
  **L25 CN**: 注释记录了意图或上下文：`MAXLOC & MINLOC`。
- **L26 EN**: Blank line separates nearby declarations or logic blocks.
  **L26 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L27 EN**: Begins a template declaration parameterizing subsequent code.
  **L27 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L28 EN**: Defines type alias `Type` for readability or ABI convenience.
  **L28 CN**: 定义类型别名 `Type`，以提升可读性或满足 ABI 便利性。
- **L29 EN**: Declares or defines callable `NumericCompare`.
  **L29 CN**: 声明或定义可调用实体 `NumericCompare`。
- **L30 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L30 CN**: 延续周围的声明、表达式或控制流结构。
- **L31 EN**: Declares or defines callable `operator`.
  **L31 CN**: 声明或定义可调用实体 `operator`。
- **L32 EN**: Introduces conditional control flow with an `if` statement.
  **L32 CN**: 通过 `if` 语句引入条件控制流。
- **L33 EN**: Returns from the current function, often propagating a computed result.
  **L33 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L34 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L34 CN**: 延续周围的声明、表达式或控制流结构。
- **L35 EN**: Returns from the current function, often propagating a computed result.
  **L35 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L36 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L36 CN**: 延续周围的声明、表达式或控制流结构。
- **L37 EN**: Returns from the current function, often propagating a computed result.
  **L37 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L38 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L38 CN**: 延续周围的声明、表达式或控制流结构。
- **L39 EN**: Returns from the current function, often propagating a computed result.
  **L39 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L40 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L40 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L41 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L41 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L42 EN**: Blank line separates nearby declarations or logic blocks.
  **L42 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L43 EN**: Defines label or access section `private`.
  **L43 CN**: 定义标签或访问区段 `private`。
- **L44 EN**: Executes statement `bool back_;`.
  **L44 CN**: 执行语句 `bool back_;`。
- **L45 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L45 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L46 EN**: Blank line separates nearby declarations or logic blocks.
  **L46 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L47 EN**: Begins a template declaration parameterizing subsequent code.
  **L47 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L48 EN**: Defines label or access section `public`.
  **L48 CN**: 定义标签或访问区段 `public`。

### Lines 49-72

````cpp
  using Type = T;
  explicit RT_API_ATTRS CharacterCompare(std::size_t elemLen, bool back)
      : chars_{elemLen / sizeof(T)}, back_{back} {}
  RT_API_ATTRS bool operator()(const T &value, const T &previous) const {
    int cmp{CharacterScalarCompare<T>(&value, &previous, chars_, chars_)};
    if (cmp == 0) {
      return back_;
    } else if constexpr (IS_MAX) {
      return cmp > 0;
    } else {
      return cmp < 0;
    }
  }

private:
  std::size_t chars_;
  bool back_;
};

template <typename COMPARE> class ExtremumLocAccumulator {
public:
  using Type = typename COMPARE::Type;
  RT_API_ATTRS ExtremumLocAccumulator(const Descriptor &array, bool back)
      : array_{array}, argRank_{array.rank()},
````

- **L49 EN**: Defines type alias `Type` for readability or ABI convenience.
  **L49 CN**: 定义类型别名 `Type`，以提升可读性或满足 ABI 便利性。
- **L50 EN**: Declares or defines callable `CharacterCompare`.
  **L50 CN**: 声明或定义可调用实体 `CharacterCompare`。
- **L51 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L51 CN**: 延续周围的声明、表达式或控制流结构。
- **L52 EN**: Declares or defines callable `operator`.
  **L52 CN**: 声明或定义可调用实体 `operator`。
- **L53 EN**: Executes statement `int cmp{CharacterScalarCompare<T>(&value, &previous, chars_, chars_)};`.
  **L53 CN**: 执行语句 `int cmp{CharacterScalarCompare<T>(&value, &previous, chars_, chars_)};`。
- **L54 EN**: Introduces conditional control flow with an `if` statement.
  **L54 CN**: 通过 `if` 语句引入条件控制流。
- **L55 EN**: Returns from the current function, often propagating a computed result.
  **L55 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L56 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L56 CN**: 延续周围的声明、表达式或控制流结构。
- **L57 EN**: Returns from the current function, often propagating a computed result.
  **L57 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L58 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L58 CN**: 延续周围的声明、表达式或控制流结构。
- **L59 EN**: Returns from the current function, often propagating a computed result.
  **L59 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L60 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L60 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L61 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L61 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L62 EN**: Blank line separates nearby declarations or logic blocks.
  **L62 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L63 EN**: Defines label or access section `private`.
  **L63 CN**: 定义标签或访问区段 `private`。
- **L64 EN**: Executes statement `std::size_t chars_;`.
  **L64 CN**: 执行语句 `std::size_t chars_;`。
- **L65 EN**: Executes statement `bool back_;`.
  **L65 CN**: 执行语句 `bool back_;`。
- **L66 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L66 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L67 EN**: Blank line separates nearby declarations or logic blocks.
  **L67 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L68 EN**: Begins a template declaration parameterizing subsequent code.
  **L68 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L69 EN**: Defines label or access section `public`.
  **L69 CN**: 定义标签或访问区段 `public`。
- **L70 EN**: Defines type alias `Type` for readability or ABI convenience.
  **L70 CN**: 定义类型别名 `Type`，以提升可读性或满足 ABI 便利性。
- **L71 EN**: Declares or defines callable `ExtremumLocAccumulator`.
  **L71 CN**: 声明或定义可调用实体 `ExtremumLocAccumulator`。
- **L72 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L72 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 73-96

````cpp
        compare_{array.ElementBytes(), back} {
    Reinitialize();
  }
  RT_API_ATTRS void Reinitialize() {
    // per standard: result indices are all zero if no data
    for (int j{0}; j < argRank_; ++j) {
      extremumLoc_[j] = 0;
    }
    previous_ = nullptr;
  }
  RT_API_ATTRS int argRank() const { return argRank_; }
  template <typename A>
  RT_API_ATTRS void GetResult(A *p, int zeroBasedDim = -1) {
    if (zeroBasedDim >= 0) {
      *p = extremumLoc_[zeroBasedDim];
    } else {
      for (int j{0}; j < argRank_; ++j) {
        p[j] = extremumLoc_[j];
      }
    }
  }
  template <typename IGNORED>
  RT_API_ATTRS bool AccumulateAt(const SubscriptValue at[]) {
    const auto &value{*array_.Element<Type>(at)};
````

- **L73 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L73 CN**: 延续周围的声明、表达式或控制流结构。
- **L74 EN**: Executes statement involving `Reinitialize`.
  **L74 CN**: 执行涉及 `Reinitialize` 的语句。
- **L75 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L75 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L76 EN**: Declares or defines callable `Reinitialize`.
  **L76 CN**: 声明或定义可调用实体 `Reinitialize`。
- **L77 EN**: Comment documents intent or context: `per standard: result indices are all zero if no data`.
  **L77 CN**: 注释记录了意图或上下文：`per standard: result indices are all zero if no data`。
- **L78 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L78 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L79 EN**: Initializes or updates `extremumLoc_[j]`.
  **L79 CN**: 初始化或更新 `extremumLoc_[j]`。
- **L80 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L80 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L81 EN**: Initializes or updates `previous_`.
  **L81 CN**: 初始化或更新 `previous_`。
- **L82 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L82 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L83 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L83 CN**: 延续周围的声明、表达式或控制流结构。
- **L84 EN**: Begins a template declaration parameterizing subsequent code.
  **L84 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L85 EN**: Declares or defines callable `GetResult`.
  **L85 CN**: 声明或定义可调用实体 `GetResult`。
- **L86 EN**: Introduces conditional control flow with an `if` statement.
  **L86 CN**: 通过 `if` 语句引入条件控制流。
- **L87 EN**: Comment documents intent or context: `p = extremumLoc_[zeroBasedDim];`.
  **L87 CN**: 注释记录了意图或上下文：`p = extremumLoc_[zeroBasedDim];`。
- **L88 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L88 CN**: 延续周围的声明、表达式或控制流结构。
- **L89 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L89 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L90 EN**: Initializes or updates `p[j]`.
  **L90 CN**: 初始化或更新 `p[j]`。
- **L91 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L91 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L92 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L92 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L93 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L93 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L94 EN**: Begins a template declaration parameterizing subsequent code.
  **L94 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L95 EN**: Declares or defines callable `AccumulateAt`.
  **L95 CN**: 声明或定义可调用实体 `AccumulateAt`。
- **L96 EN**: Executes statement `const auto &value{*array_.Element<Type>(at)};`.
  **L96 CN**: 执行语句 `const auto &value{*array_.Element<Type>(at)};`。

### Lines 97-120

````cpp
    if (!previous_ || compare_(value, *previous_)) {
      previous_ = &value;
      for (int j{0}; j < argRank_; ++j) {
        extremumLoc_[j] = at[j] - array_.GetDimension(j).LowerBound() + 1;
      }
    }
    return true;
  }

private:
  const Descriptor &array_;
  int argRank_;
  SubscriptValue extremumLoc_[maxRank];
  const Type *previous_{nullptr};
  COMPARE compare_;
};

template <typename ACCUMULATOR, typename CPPTYPE>
static RT_API_ATTRS void LocationHelper(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, const Descriptor *mask,
    Terminator &terminator, bool back) {
  ACCUMULATOR accumulator{x, back};
  DoTotalReduction<CPPTYPE>(x, 0, mask, accumulator, intrinsic, terminator);
  ApplyIntegerKind<LocationResultHelper<ACCUMULATOR>::template Functor, void>(
````

- **L97 EN**: Introduces conditional control flow with an `if` statement.
  **L97 CN**: 通过 `if` 语句引入条件控制流。
- **L98 EN**: Initializes or updates `previous_`.
  **L98 CN**: 初始化或更新 `previous_`。
- **L99 EN**: Starts a `for` loop to iterate over a range, collection, or index space.
  **L99 CN**: 开始一个 `for` 循环以遍历范围、集合或索引空间。
- **L100 EN**: Initializes or updates `extremumLoc_[j]`.
  **L100 CN**: 初始化或更新 `extremumLoc_[j]`。
- **L101 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L101 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L102 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L102 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L103 EN**: Returns from the current function, often propagating a computed result.
  **L103 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L104 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L104 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L105 EN**: Blank line separates nearby declarations or logic blocks.
  **L105 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L106 EN**: Defines label or access section `private`.
  **L106 CN**: 定义标签或访问区段 `private`。
- **L107 EN**: Executes statement `const Descriptor &array_;`.
  **L107 CN**: 执行语句 `const Descriptor &array_;`。
- **L108 EN**: Executes statement `int argRank_;`.
  **L108 CN**: 执行语句 `int argRank_;`。
- **L109 EN**: Executes statement `SubscriptValue extremumLoc_[maxRank];`.
  **L109 CN**: 执行语句 `SubscriptValue extremumLoc_[maxRank];`。
- **L110 EN**: Executes statement `const Type *previous_{nullptr};`.
  **L110 CN**: 执行语句 `const Type *previous_{nullptr};`。
- **L111 EN**: Executes statement `COMPARE compare_;`.
  **L111 CN**: 执行语句 `COMPARE compare_;`。
- **L112 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L112 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L113 EN**: Blank line separates nearby declarations or logic blocks.
  **L113 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L114 EN**: Begins a template declaration parameterizing subsequent code.
  **L114 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L115 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L115 CN**: 延续周围的声明、表达式或控制流结构。
- **L116 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L116 CN**: 延续周围的声明、表达式或控制流结构。
- **L117 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L117 CN**: 延续周围的声明、表达式或控制流结构。
- **L118 EN**: Executes statement `ACCUMULATOR accumulator{x, back};`.
  **L118 CN**: 执行语句 `ACCUMULATOR accumulator{x, back};`。
- **L119 EN**: Executes statement `DoTotalReduction<CPPTYPE>(x, 0, mask, accumulator, intrinsic, terminator);`.
  **L119 CN**: 执行语句 `DoTotalReduction<CPPTYPE>(x, 0, mask, accumulator, intrinsic, terminator);`。
- **L120 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L120 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 121-144

````cpp
      kind, terminator, accumulator, result);
}

template <TypeCategory CAT, int KIND, bool IS_MAX,
    template <typename, bool> class COMPARE>
inline RT_API_ATTRS void DoMaxOrMinLoc(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, const char *source,
    int line, const Descriptor *mask, bool back) {
  using CppType = CppTypeFor<CAT, KIND>;
  Terminator terminator{source, line};
  LocationHelper<ExtremumLocAccumulator<COMPARE<CppType, IS_MAX>>, CppType>(
      intrinsic, result, x, kind, mask, terminator, back);
}

template <bool IS_MAX> struct CharacterMaxOrMinLocHelper {
  template <int KIND> struct Functor {
    RT_API_ATTRS void operator()(const char *intrinsic, Descriptor &result,
        const Descriptor &x, int kind, const char *source, int line,
        const Descriptor *mask, bool back) const {
      DoMaxOrMinLoc<TypeCategory::Character, KIND, IS_MAX, CharacterCompare>(
          intrinsic, result, x, kind, source, line, mask, back);
    }
  };
};
````

- **L121 EN**: Executes statement `kind, terminator, accumulator, result);`.
  **L121 CN**: 执行语句 `kind, terminator, accumulator, result);`。
- **L122 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L122 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L123 EN**: Blank line separates nearby declarations or logic blocks.
  **L123 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L124 EN**: Begins a template declaration parameterizing subsequent code.
  **L124 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L125 EN**: Begins a template declaration parameterizing subsequent code.
  **L125 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L126 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L126 CN**: 延续周围的声明、表达式或控制流结构。
- **L127 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L127 CN**: 延续周围的声明、表达式或控制流结构。
- **L128 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L128 CN**: 延续周围的声明、表达式或控制流结构。
- **L129 EN**: Defines type alias `CppType` for readability or ABI convenience.
  **L129 CN**: 定义类型别名 `CppType`，以提升可读性或满足 ABI 便利性。
- **L130 EN**: Executes statement `Terminator terminator{source, line};`.
  **L130 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L131 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L131 CN**: 延续周围的声明、表达式或控制流结构。
- **L132 EN**: Executes statement `intrinsic, result, x, kind, mask, terminator, back);`.
  **L132 CN**: 执行语句 `intrinsic, result, x, kind, mask, terminator, back);`。
- **L133 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L133 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L134 EN**: Blank line separates nearby declarations or logic blocks.
  **L134 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L135 EN**: Begins a template declaration parameterizing subsequent code.
  **L135 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L136 EN**: Begins a template declaration parameterizing subsequent code.
  **L136 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L137 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L137 CN**: 延续周围的声明、表达式或控制流结构。
- **L138 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L138 CN**: 延续周围的声明、表达式或控制流结构。
- **L139 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L139 CN**: 延续周围的声明、表达式或控制流结构。
- **L140 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L140 CN**: 延续周围的声明、表达式或控制流结构。
- **L141 EN**: Executes statement `intrinsic, result, x, kind, source, line, mask, back);`.
  **L141 CN**: 执行语句 `intrinsic, result, x, kind, source, line, mask, back);`。
- **L142 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L142 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L143 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L143 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L144 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L144 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 145-168

````cpp

template <bool IS_MAX>
inline RT_API_ATTRS void CharacterMaxOrMinLoc(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, const char *source,
    int line, const Descriptor *mask, bool back) {
  int rank{x.rank()};
  SubscriptValue extent[1]{rank};
  result.Establish(TypeCategory::Integer, kind, nullptr, 1, extent,
      CFI_attribute_allocatable);
  result.GetDimension(0).SetBounds(1, extent[0]);
  Terminator terminator{source, line};
  if (int stat{result.Allocate(kNoAsyncObject)}) {
    terminator.Crash(
        "%s: could not allocate memory for result; STAT=%d", intrinsic, stat);
  }
  CheckIntegerKind(terminator, kind, intrinsic);
  auto catKind{x.type().GetCategoryAndKind()};
  RUNTIME_CHECK(terminator, catKind.has_value());
  switch (catKind->first) {
  case TypeCategory::Character:
    ApplyCharacterKind<CharacterMaxOrMinLocHelper<IS_MAX>::template Functor,
        void>(catKind->second, terminator, intrinsic, result, x, kind, source,
        line, mask, back);
    break;
````

- **L145 EN**: Blank line separates nearby declarations or logic blocks.
  **L145 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L146 EN**: Begins a template declaration parameterizing subsequent code.
  **L146 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L147 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L147 CN**: 延续周围的声明、表达式或控制流结构。
- **L148 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L148 CN**: 延续周围的声明、表达式或控制流结构。
- **L149 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L149 CN**: 延续周围的声明、表达式或控制流结构。
- **L150 EN**: Executes statement involving `rank`.
  **L150 CN**: 执行涉及 `rank` 的语句。
- **L151 EN**: Executes statement `SubscriptValue extent[1]{rank};`.
  **L151 CN**: 执行语句 `SubscriptValue extent[1]{rank};`。
- **L152 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L152 CN**: 延续周围的声明、表达式或控制流结构。
- **L153 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L153 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L154 EN**: Executes statement involving `GetDimension`.
  **L154 CN**: 执行涉及 `GetDimension` 的语句。
- **L155 EN**: Executes statement `Terminator terminator{source, line};`.
  **L155 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L156 EN**: Introduces conditional control flow with an `if` statement.
  **L156 CN**: 通过 `if` 语句引入条件控制流。
- **L157 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L157 CN**: 延续周围的声明、表达式或控制流结构。
- **L158 EN**: Executes statement `"%s: could not allocate memory for result; STAT=%d", intrinsic, stat);`.
  **L158 CN**: 执行语句 `"%s: could not allocate memory for result; STAT=%d", intrinsic, stat);`。
- **L159 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L159 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L160 EN**: Executes statement involving `CheckIntegerKind`.
  **L160 CN**: 执行涉及 `CheckIntegerKind` 的语句。
- **L161 EN**: Executes statement involving `type`.
  **L161 CN**: 执行涉及 `type` 的语句。
- **L162 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L162 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L163 EN**: Begins a `switch` dispatch over discrete cases.
  **L163 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L164 EN**: Marks one `switch` case label.
  **L164 CN**: 标记一个 `switch` 的 case 标签。
- **L165 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L165 CN**: 延续周围的声明、表达式或控制流结构。
- **L166 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L166 CN**: 延续周围的声明、表达式或控制流结构。
- **L167 EN**: Executes statement `line, mask, back);`.
  **L167 CN**: 执行语句 `line, mask, back);`。
- **L168 EN**: Breaks out of the current loop or switch.
  **L168 CN**: 跳出当前循环或 switch。

### Lines 169-192

````cpp
  default:
    terminator.Crash(
        "%s: bad data type code (%d) for array", intrinsic, x.type().raw());
  }
}

template <TypeCategory CAT, int KIND, bool IS_MAXVAL>
inline RT_API_ATTRS void TotalNumericMaxOrMinLoc(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, const char *source,
    int line, const Descriptor *mask, bool back) {
  int rank{x.rank()};
  SubscriptValue extent[1]{rank};
  result.Establish(TypeCategory::Integer, kind, nullptr, 1, extent,
      CFI_attribute_allocatable);
  result.GetDimension(0).SetBounds(1, extent[0]);
  Terminator terminator{source, line};
  if (int stat{result.Allocate(kNoAsyncObject)}) {
    terminator.Crash(
        "%s: could not allocate memory for result; STAT=%d", intrinsic, stat);
  }
  CheckIntegerKind(terminator, kind, intrinsic);
  RUNTIME_CHECK(terminator, TypeCode(CAT, KIND) == x.type());
  DoMaxOrMinLoc<CAT, KIND, IS_MAXVAL, NumericCompare>(
      intrinsic, result, x, kind, source, line, mask, back);
````

- **L169 EN**: Provides the default branch for a `switch` statement.
  **L169 CN**: 为 `switch` 语句提供默认分支。
- **L170 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L170 CN**: 延续周围的声明、表达式或控制流结构。
- **L171 EN**: Executes statement involving `code`.
  **L171 CN**: 执行涉及 `code` 的语句。
- **L172 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L172 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L173 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L173 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L174 EN**: Blank line separates nearby declarations or logic blocks.
  **L174 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L175 EN**: Begins a template declaration parameterizing subsequent code.
  **L175 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L176 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L176 CN**: 延续周围的声明、表达式或控制流结构。
- **L177 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L177 CN**: 延续周围的声明、表达式或控制流结构。
- **L178 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L178 CN**: 延续周围的声明、表达式或控制流结构。
- **L179 EN**: Executes statement involving `rank`.
  **L179 CN**: 执行涉及 `rank` 的语句。
- **L180 EN**: Executes statement `SubscriptValue extent[1]{rank};`.
  **L180 CN**: 执行语句 `SubscriptValue extent[1]{rank};`。
- **L181 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L181 CN**: 延续周围的声明、表达式或控制流结构。
- **L182 EN**: Executes statement `CFI_attribute_allocatable);`.
  **L182 CN**: 执行语句 `CFI_attribute_allocatable);`。
- **L183 EN**: Executes statement involving `GetDimension`.
  **L183 CN**: 执行涉及 `GetDimension` 的语句。
- **L184 EN**: Executes statement `Terminator terminator{source, line};`.
  **L184 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L185 EN**: Introduces conditional control flow with an `if` statement.
  **L185 CN**: 通过 `if` 语句引入条件控制流。
- **L186 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L186 CN**: 延续周围的声明、表达式或控制流结构。
- **L187 EN**: Executes statement `"%s: could not allocate memory for result; STAT=%d", intrinsic, stat);`.
  **L187 CN**: 执行语句 `"%s: could not allocate memory for result; STAT=%d", intrinsic, stat);`。
- **L188 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L188 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L189 EN**: Executes statement involving `CheckIntegerKind`.
  **L189 CN**: 执行涉及 `CheckIntegerKind` 的语句。
- **L190 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L190 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L191 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L191 CN**: 延续周围的声明、表达式或控制流结构。
- **L192 EN**: Executes statement `intrinsic, result, x, kind, source, line, mask, back);`.
  **L192 CN**: 执行语句 `intrinsic, result, x, kind, source, line, mask, back);`。

### Lines 193-216

````cpp
}

extern "C" {
RT_EXT_API_GROUP_BEGIN

void RTDEF(MaxlocCharacter)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  CharacterMaxOrMinLoc<true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocInteger1)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 1, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocInteger2)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 2, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocInteger4)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 4, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
````

- **L193 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L193 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L194 EN**: Blank line separates nearby declarations or logic blocks.
  **L194 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L195 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L195 CN**: 延续周围的声明、表达式或控制流结构。
- **L196 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L196 CN**: 延续周围的声明、表达式或控制流结构。
- **L197 EN**: Blank line separates nearby declarations or logic blocks.
  **L197 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L198 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L198 CN**: 延续周围的声明、表达式或控制流结构。
- **L199 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L199 CN**: 延续周围的声明、表达式或控制流结构。
- **L200 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L200 CN**: 延续周围的声明、表达式或控制流结构。
- **L201 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L201 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L202 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L202 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L203 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L203 CN**: 延续周围的声明、表达式或控制流结构。
- **L204 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L204 CN**: 延续周围的声明、表达式或控制流结构。
- **L205 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L205 CN**: 延续周围的声明、表达式或控制流结构。
- **L206 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L206 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L207 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L207 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L208 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L208 CN**: 延续周围的声明、表达式或控制流结构。
- **L209 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L209 CN**: 延续周围的声明、表达式或控制流结构。
- **L210 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L210 CN**: 延续周围的声明、表达式或控制流结构。
- **L211 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L211 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L212 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L212 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L213 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L213 CN**: 延续周围的声明、表达式或控制流结构。
- **L214 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L214 CN**: 延续周围的声明、表达式或控制流结构。
- **L215 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L215 CN**: 延续周围的声明、表达式或控制流结构。
- **L216 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L216 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。

### Lines 217-240

````cpp
}
void RTDEF(MaxlocInteger8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 8, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#ifdef __SIZEOF_INT128__
void RTDEF(MaxlocInteger16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 16, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#endif
void RTDEF(MaxlocUnsigned1)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 1, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocUnsigned2)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 2, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocUnsigned4)(Descriptor &result, const Descriptor &x, int kind,
````

- **L217 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L217 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L218 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L218 CN**: 延续周围的声明、表达式或控制流结构。
- **L219 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L219 CN**: 延续周围的声明、表达式或控制流结构。
- **L220 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L220 CN**: 延续周围的声明、表达式或控制流结构。
- **L221 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L221 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L222 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L222 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L223 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L223 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L224 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L224 CN**: 延续周围的声明、表达式或控制流结构。
- **L225 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L225 CN**: 延续周围的声明、表达式或控制流结构。
- **L226 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L226 CN**: 延续周围的声明、表达式或控制流结构。
- **L227 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L227 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L228 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L228 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L229 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L229 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L230 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L230 CN**: 延续周围的声明、表达式或控制流结构。
- **L231 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L231 CN**: 延续周围的声明、表达式或控制流结构。
- **L232 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L232 CN**: 延续周围的声明、表达式或控制流结构。
- **L233 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L233 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L234 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L234 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L235 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L235 CN**: 延续周围的声明、表达式或控制流结构。
- **L236 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L236 CN**: 延续周围的声明、表达式或控制流结构。
- **L237 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L237 CN**: 延续周围的声明、表达式或控制流结构。
- **L238 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L238 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L239 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L239 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L240 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L240 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 241-264

````cpp
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 4, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocUnsigned8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 8, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#ifdef __SIZEOF_INT128__
void RTDEF(MaxlocUnsigned16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 16, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#endif
void RTDEF(MaxlocReal4)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 4, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MaxlocReal8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 8, true>(
````

- **L241 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L241 CN**: 延续周围的声明、表达式或控制流结构。
- **L242 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L242 CN**: 延续周围的声明、表达式或控制流结构。
- **L243 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L243 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L244 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L244 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L245 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L245 CN**: 延续周围的声明、表达式或控制流结构。
- **L246 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L246 CN**: 延续周围的声明、表达式或控制流结构。
- **L247 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L247 CN**: 延续周围的声明、表达式或控制流结构。
- **L248 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L248 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L249 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L249 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L250 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L250 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L251 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L251 CN**: 延续周围的声明、表达式或控制流结构。
- **L252 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L252 CN**: 延续周围的声明、表达式或控制流结构。
- **L253 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L253 CN**: 延续周围的声明、表达式或控制流结构。
- **L254 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L254 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L255 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L255 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L256 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L256 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L257 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L257 CN**: 延续周围的声明、表达式或控制流结构。
- **L258 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L258 CN**: 延续周围的声明、表达式或控制流结构。
- **L259 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L259 CN**: 延续周围的声明、表达式或控制流结构。
- **L260 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L260 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L261 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L261 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L262 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L262 CN**: 延续周围的声明、表达式或控制流结构。
- **L263 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L263 CN**: 延续周围的声明、表达式或控制流结构。
- **L264 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L264 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 265-288

````cpp
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#if HAS_FLOAT80
void RTDEF(MaxlocReal10)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 10, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
void RTDEF(MaxlocReal16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 16, true>(
      "MAXLOC", result, x, kind, source, line, mask, back);
}
#endif
void RTDEF(MinlocCharacter)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  CharacterMaxOrMinLoc<false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocInteger1)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 1, false>(
````

- **L265 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L265 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L266 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L266 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L267 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L267 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L268 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L268 CN**: 延续周围的声明、表达式或控制流结构。
- **L269 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L269 CN**: 延续周围的声明、表达式或控制流结构。
- **L270 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L270 CN**: 延续周围的声明、表达式或控制流结构。
- **L271 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L271 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L272 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L272 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L273 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L273 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L274 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L274 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L275 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L275 CN**: 延续周围的声明、表达式或控制流结构。
- **L276 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L276 CN**: 延续周围的声明、表达式或控制流结构。
- **L277 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L277 CN**: 延续周围的声明、表达式或控制流结构。
- **L278 EN**: Executes statement `"MAXLOC", result, x, kind, source, line, mask, back);`.
  **L278 CN**: 执行语句 `"MAXLOC", result, x, kind, source, line, mask, back);`。
- **L279 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L279 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L280 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L280 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L281 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L281 CN**: 延续周围的声明、表达式或控制流结构。
- **L282 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L282 CN**: 延续周围的声明、表达式或控制流结构。
- **L283 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L283 CN**: 延续周围的声明、表达式或控制流结构。
- **L284 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L284 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L285 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L285 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L286 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L286 CN**: 延续周围的声明、表达式或控制流结构。
- **L287 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L287 CN**: 延续周围的声明、表达式或控制流结构。
- **L288 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L288 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 289-312

````cpp
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocInteger2)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 2, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocInteger4)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 4, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocInteger8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 8, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
#ifdef __SIZEOF_INT128__
void RTDEF(MinlocInteger16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Integer, 16, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
#endif
````

- **L289 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L289 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L290 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L290 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L291 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L291 CN**: 延续周围的声明、表达式或控制流结构。
- **L292 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L292 CN**: 延续周围的声明、表达式或控制流结构。
- **L293 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L293 CN**: 延续周围的声明、表达式或控制流结构。
- **L294 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L294 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L295 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L295 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L296 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L296 CN**: 延续周围的声明、表达式或控制流结构。
- **L297 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L297 CN**: 延续周围的声明、表达式或控制流结构。
- **L298 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L298 CN**: 延续周围的声明、表达式或控制流结构。
- **L299 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L299 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L300 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L300 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L301 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L301 CN**: 延续周围的声明、表达式或控制流结构。
- **L302 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L302 CN**: 延续周围的声明、表达式或控制流结构。
- **L303 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L303 CN**: 延续周围的声明、表达式或控制流结构。
- **L304 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L304 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L305 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L305 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L306 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L306 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L307 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L307 CN**: 延续周围的声明、表达式或控制流结构。
- **L308 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L308 CN**: 延续周围的声明、表达式或控制流结构。
- **L309 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L309 CN**: 延续周围的声明、表达式或控制流结构。
- **L310 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L310 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L311 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L311 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L312 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L312 CN**: 预处理指令管理条件编译或宏：`#endif`。

### Lines 313-336

````cpp
void RTDEF(MinlocUnsigned1)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 1, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocUnsigned2)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 2, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocUnsigned4)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 4, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocUnsigned8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 8, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
#ifdef __SIZEOF_INT128__
void RTDEF(MinlocUnsigned16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Unsigned, 16, false>(
````

- **L313 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L313 CN**: 延续周围的声明、表达式或控制流结构。
- **L314 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L314 CN**: 延续周围的声明、表达式或控制流结构。
- **L315 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L315 CN**: 延续周围的声明、表达式或控制流结构。
- **L316 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L316 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L317 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L317 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L318 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L318 CN**: 延续周围的声明、表达式或控制流结构。
- **L319 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L319 CN**: 延续周围的声明、表达式或控制流结构。
- **L320 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L320 CN**: 延续周围的声明、表达式或控制流结构。
- **L321 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L321 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L322 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L322 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L323 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L323 CN**: 延续周围的声明、表达式或控制流结构。
- **L324 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L324 CN**: 延续周围的声明、表达式或控制流结构。
- **L325 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L325 CN**: 延续周围的声明、表达式或控制流结构。
- **L326 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L326 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L327 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L327 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L328 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L328 CN**: 延续周围的声明、表达式或控制流结构。
- **L329 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L329 CN**: 延续周围的声明、表达式或控制流结构。
- **L330 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L330 CN**: 延续周围的声明、表达式或控制流结构。
- **L331 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L331 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L332 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L332 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L333 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L333 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L334 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L334 CN**: 延续周围的声明、表达式或控制流结构。
- **L335 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L335 CN**: 延续周围的声明、表达式或控制流结构。
- **L336 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L336 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 337-360

````cpp
      "MINLOC", result, x, kind, source, line, mask, back);
}
#endif
void RTDEF(MinlocReal4)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 4, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
void RTDEF(MinlocReal8)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 8, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
#if HAS_FLOAT80
void RTDEF(MinlocReal10)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 10, false>(
      "MINLOC", result, x, kind, source, line, mask, back);
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
void RTDEF(MinlocReal16)(Descriptor &result, const Descriptor &x, int kind,
    const char *source, int line, const Descriptor *mask, bool back) {
  TotalNumericMaxOrMinLoc<TypeCategory::Real, 16, false>(
````

- **L337 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L337 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L338 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L338 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L339 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L339 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L340 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L340 CN**: 延续周围的声明、表达式或控制流结构。
- **L341 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L341 CN**: 延续周围的声明、表达式或控制流结构。
- **L342 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L342 CN**: 延续周围的声明、表达式或控制流结构。
- **L343 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L343 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L344 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L344 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L345 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L345 CN**: 延续周围的声明、表达式或控制流结构。
- **L346 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L346 CN**: 延续周围的声明、表达式或控制流结构。
- **L347 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L347 CN**: 延续周围的声明、表达式或控制流结构。
- **L348 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L348 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L349 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L349 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L350 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L350 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L351 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L351 CN**: 延续周围的声明、表达式或控制流结构。
- **L352 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L352 CN**: 延续周围的声明、表达式或控制流结构。
- **L353 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L353 CN**: 延续周围的声明、表达式或控制流结构。
- **L354 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L354 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L355 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L355 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L356 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L356 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L357 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L357 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L358 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L358 CN**: 延续周围的声明、表达式或控制流结构。
- **L359 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L359 CN**: 延续周围的声明、表达式或控制流结构。
- **L360 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L360 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 361-384

````cpp
      "MINLOC", result, x, kind, source, line, mask, back);
}
#endif

RT_EXT_API_GROUP_END
} // extern "C"

// MAXLOC/MINLOC with DIM=

template <TypeCategory CAT, int KIND, bool IS_MAX,
    template <typename, bool> class COMPARE>
static RT_API_ATTRS void DoPartialMaxOrMinLoc(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, int dim,
    const Descriptor *mask, Terminator &terminator, bool back) {
  using CppType = CppTypeFor<CAT, KIND>;
  using Accumulator = ExtremumLocAccumulator<COMPARE<CppType, IS_MAX>>;
  Accumulator accumulator{x, back};
  ApplyIntegerKind<PartialLocationHelper<Accumulator>::template Functor, void>(
      kind, terminator, result, x, dim, mask, terminator, intrinsic,
      accumulator);
}

template <TypeCategory CAT, bool IS_MAX,
    template <typename, bool> class COMPARE>
````

- **L361 EN**: Executes statement `"MINLOC", result, x, kind, source, line, mask, back);`.
  **L361 CN**: 执行语句 `"MINLOC", result, x, kind, source, line, mask, back);`。
- **L362 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L362 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L363 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L363 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L364 EN**: Blank line separates nearby declarations or logic blocks.
  **L364 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L365 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L365 CN**: 延续周围的声明、表达式或控制流结构。
- **L366 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L366 CN**: 延续周围的声明、表达式或控制流结构。
- **L367 EN**: Blank line separates nearby declarations or logic blocks.
  **L367 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L368 EN**: Comment documents intent or context: `MAXLOC/MINLOC with DIM=`.
  **L368 CN**: 注释记录了意图或上下文：`MAXLOC/MINLOC with DIM=`。
- **L369 EN**: Blank line separates nearby declarations or logic blocks.
  **L369 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L370 EN**: Begins a template declaration parameterizing subsequent code.
  **L370 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L371 EN**: Begins a template declaration parameterizing subsequent code.
  **L371 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L372 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L372 CN**: 延续周围的声明、表达式或控制流结构。
- **L373 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L373 CN**: 延续周围的声明、表达式或控制流结构。
- **L374 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L374 CN**: 延续周围的声明、表达式或控制流结构。
- **L375 EN**: Defines type alias `CppType` for readability or ABI convenience.
  **L375 CN**: 定义类型别名 `CppType`，以提升可读性或满足 ABI 便利性。
- **L376 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L376 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L377 EN**: Executes statement `Accumulator accumulator{x, back};`.
  **L377 CN**: 执行语句 `Accumulator accumulator{x, back};`。
- **L378 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L378 CN**: 延续周围的声明、表达式或控制流结构。
- **L379 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L379 CN**: 延续周围的声明、表达式或控制流结构。
- **L380 EN**: Executes statement `accumulator);`.
  **L380 CN**: 执行语句 `accumulator);`。
- **L381 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L381 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L382 EN**: Blank line separates nearby declarations or logic blocks.
  **L382 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L383 EN**: Begins a template declaration parameterizing subsequent code.
  **L383 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L384 EN**: Begins a template declaration parameterizing subsequent code.
  **L384 CN**: 开始一个模板声明，为后续代码提供参数化能力。

### Lines 385-408

````cpp
struct DoPartialMaxOrMinLocHelper {
  template <int KIND> struct Functor {
    // NVCC inlines more aggressively which causes too many specializations of
    // this function to be inlined causing compiler timeouts. Set as
    // noinline to allow compilation to complete.
    RT_API_ATTRS RT_DEVICE_NOINLINE void operator()(const char *intrinsic,
        Descriptor &result, const Descriptor &x, int kind, int dim,
        const Descriptor *mask, bool back, Terminator &terminator) const {
      DoPartialMaxOrMinLoc<CAT, KIND, IS_MAX, COMPARE>(
          intrinsic, result, x, kind, dim, mask, terminator, back);
    }
  };
};

template <bool IS_MAX>
inline RT_API_ATTRS void TypedPartialMaxOrMinLoc(const char *intrinsic,
    Descriptor &result, const Descriptor &x, int kind, int dim,
    const char *source, int line, const Descriptor *mask, bool back) {
  Terminator terminator{source, line};
  CheckIntegerKind(terminator, kind, intrinsic);
  auto catKind{x.type().GetCategoryAndKind()};
  RUNTIME_CHECK(terminator, catKind.has_value());
  const Descriptor *maskToUse{mask};
  SubscriptValue maskAt[maxRank]; // contents unused
````

- **L385 EN**: Declares or defines struct `DoPartialMaxOrMinLocHelper`.
  **L385 CN**: 声明或定义 struct `DoPartialMaxOrMinLocHelper`。
- **L386 EN**: Begins a template declaration parameterizing subsequent code.
  **L386 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L387 EN**: Comment documents intent or context: `NVCC inlines more aggressively which causes too many specializations of`.
  **L387 CN**: 注释记录了意图或上下文：`NVCC inlines more aggressively which causes too many specializations of`。
- **L388 EN**: Comment documents intent or context: `this function to be inlined causing compiler timeouts. Set as`.
  **L388 CN**: 注释记录了意图或上下文：`this function to be inlined causing compiler timeouts. Set as`。
- **L389 EN**: Comment documents intent or context: `noinline to allow compilation to complete.`.
  **L389 CN**: 注释记录了意图或上下文：`noinline to allow compilation to complete.`。
- **L390 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L390 CN**: 延续周围的声明、表达式或控制流结构。
- **L391 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L391 CN**: 延续周围的声明、表达式或控制流结构。
- **L392 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L392 CN**: 延续周围的声明、表达式或控制流结构。
- **L393 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L393 CN**: 延续周围的声明、表达式或控制流结构。
- **L394 EN**: Executes statement `intrinsic, result, x, kind, dim, mask, terminator, back);`.
  **L394 CN**: 执行语句 `intrinsic, result, x, kind, dim, mask, terminator, back);`。
- **L395 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L395 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L396 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L396 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L397 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L397 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L398 EN**: Blank line separates nearby declarations or logic blocks.
  **L398 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L399 EN**: Begins a template declaration parameterizing subsequent code.
  **L399 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L400 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L400 CN**: 延续周围的声明、表达式或控制流结构。
- **L401 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L401 CN**: 延续周围的声明、表达式或控制流结构。
- **L402 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L402 CN**: 延续周围的声明、表达式或控制流结构。
- **L403 EN**: Executes statement `Terminator terminator{source, line};`.
  **L403 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L404 EN**: Executes statement involving `CheckIntegerKind`.
  **L404 CN**: 执行涉及 `CheckIntegerKind` 的语句。
- **L405 EN**: Executes statement involving `type`.
  **L405 CN**: 执行涉及 `type` 的语句。
- **L406 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L406 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L407 EN**: Executes statement `const Descriptor *maskToUse{mask};`.
  **L407 CN**: 执行语句 `const Descriptor *maskToUse{mask};`。
- **L408 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L408 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 409-432

````cpp
  if (mask && mask->rank() == 0) {
    if (IsLogicalElementTrue(*mask, maskAt)) {
      // A scalar MASK that's .TRUE.  In this case, just get rid of the MASK.
      maskToUse = nullptr;
    } else {
      // For scalar MASK arguments that are .FALSE., return all zeroes

      // Element size of the destination descriptor is the size
      // of {TypeCategory::Integer, kind}.
      CreatePartialReductionResult(result, x,
          Descriptor::BytesFor(TypeCategory::Integer, kind), dim, terminator,
          intrinsic, TypeCode{TypeCategory::Integer, kind});
      runtime::memset(
          result.OffsetElement(), 0, result.Elements() * result.ElementBytes());
      return;
    }
  }
  switch (catKind->first) {
  case TypeCategory::Integer:
    ApplyIntegerKind<DoPartialMaxOrMinLocHelper<TypeCategory::Integer, IS_MAX,
                         NumericCompare>::template Functor,
        void>(catKind->second, terminator, intrinsic, result, x, kind, dim,
        maskToUse, back, terminator);
    break;
````

- **L409 EN**: Introduces conditional control flow with an `if` statement.
  **L409 CN**: 通过 `if` 语句引入条件控制流。
- **L410 EN**: Introduces conditional control flow with an `if` statement.
  **L410 CN**: 通过 `if` 语句引入条件控制流。
- **L411 EN**: Comment documents intent or context: `A scalar MASK that's .TRUE. In this case, just get rid of the MASK.`.
  **L411 CN**: 注释记录了意图或上下文：`A scalar MASK that's .TRUE. In this case, just get rid of the MASK.`。
- **L412 EN**: Initializes or updates `maskToUse`.
  **L412 CN**: 初始化或更新 `maskToUse`。
- **L413 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L413 CN**: 延续周围的声明、表达式或控制流结构。
- **L414 EN**: Comment documents intent or context: `For scalar MASK arguments that are .FALSE., return all zeroes`.
  **L414 CN**: 注释记录了意图或上下文：`For scalar MASK arguments that are .FALSE., return all zeroes`。
- **L415 EN**: Blank line separates nearby declarations or logic blocks.
  **L415 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L416 EN**: Comment documents intent or context: `Element size of the destination descriptor is the size`.
  **L416 CN**: 注释记录了意图或上下文：`Element size of the destination descriptor is the size`。
- **L417 EN**: Comment documents intent or context: `of {TypeCategory::Integer, kind}.`.
  **L417 CN**: 注释记录了意图或上下文：`of {TypeCategory::Integer, kind}.`。
- **L418 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L418 CN**: 延续周围的声明、表达式或控制流结构。
- **L419 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L419 CN**: 延续周围的声明、表达式或控制流结构。
- **L420 EN**: Executes statement `intrinsic, TypeCode{TypeCategory::Integer, kind});`.
  **L420 CN**: 执行语句 `intrinsic, TypeCode{TypeCategory::Integer, kind});`。
- **L421 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L421 CN**: 延续周围的声明、表达式或控制流结构。
- **L422 EN**: Executes statement involving `OffsetElement`.
  **L422 CN**: 执行涉及 `OffsetElement` 的语句。
- **L423 EN**: Returns from the current function, often propagating a computed result.
  **L423 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L424 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L424 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L425 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L425 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L426 EN**: Begins a `switch` dispatch over discrete cases.
  **L426 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L427 EN**: Marks one `switch` case label.
  **L427 CN**: 标记一个 `switch` 的 case 标签。
- **L428 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L428 CN**: 延续周围的声明、表达式或控制流结构。
- **L429 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L429 CN**: 延续周围的声明、表达式或控制流结构。
- **L430 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L430 CN**: 延续周围的声明、表达式或控制流结构。
- **L431 EN**: Executes statement `maskToUse, back, terminator);`.
  **L431 CN**: 执行语句 `maskToUse, back, terminator);`。
- **L432 EN**: Breaks out of the current loop or switch.
  **L432 CN**: 跳出当前循环或 switch。

### Lines 433-456

````cpp
  case TypeCategory::Unsigned:
    ApplyIntegerKind<DoPartialMaxOrMinLocHelper<TypeCategory::Unsigned, IS_MAX,
                         NumericCompare>::template Functor,
        void>(catKind->second, terminator, intrinsic, result, x, kind, dim,
        maskToUse, back, terminator);
    break;
  case TypeCategory::Real:
    ApplyFloatingPointKind<DoPartialMaxOrMinLocHelper<TypeCategory::Real,
                               IS_MAX, NumericCompare>::template Functor,
        void>(catKind->second, terminator, intrinsic, result, x, kind, dim,
        maskToUse, back, terminator);
    break;
  case TypeCategory::Character:
    ApplyCharacterKind<DoPartialMaxOrMinLocHelper<TypeCategory::Character,
                           IS_MAX, CharacterCompare>::template Functor,
        void>(catKind->second, terminator, intrinsic, result, x, kind, dim,
        maskToUse, back, terminator);
    break;
  default:
    terminator.Crash(
        "%s: bad data type code (%d) for array", intrinsic, x.type().raw());
  }
}

````

- **L433 EN**: Marks one `switch` case label.
  **L433 CN**: 标记一个 `switch` 的 case 标签。
- **L434 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L434 CN**: 延续周围的声明、表达式或控制流结构。
- **L435 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L435 CN**: 延续周围的声明、表达式或控制流结构。
- **L436 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L436 CN**: 延续周围的声明、表达式或控制流结构。
- **L437 EN**: Executes statement `maskToUse, back, terminator);`.
  **L437 CN**: 执行语句 `maskToUse, back, terminator);`。
- **L438 EN**: Breaks out of the current loop or switch.
  **L438 CN**: 跳出当前循环或 switch。
- **L439 EN**: Marks one `switch` case label.
  **L439 CN**: 标记一个 `switch` 的 case 标签。
- **L440 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L440 CN**: 延续周围的声明、表达式或控制流结构。
- **L441 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L441 CN**: 延续周围的声明、表达式或控制流结构。
- **L442 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L442 CN**: 延续周围的声明、表达式或控制流结构。
- **L443 EN**: Executes statement `maskToUse, back, terminator);`.
  **L443 CN**: 执行语句 `maskToUse, back, terminator);`。
- **L444 EN**: Breaks out of the current loop or switch.
  **L444 CN**: 跳出当前循环或 switch。
- **L445 EN**: Marks one `switch` case label.
  **L445 CN**: 标记一个 `switch` 的 case 标签。
- **L446 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L446 CN**: 延续周围的声明、表达式或控制流结构。
- **L447 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L447 CN**: 延续周围的声明、表达式或控制流结构。
- **L448 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L448 CN**: 延续周围的声明、表达式或控制流结构。
- **L449 EN**: Executes statement `maskToUse, back, terminator);`.
  **L449 CN**: 执行语句 `maskToUse, back, terminator);`。
- **L450 EN**: Breaks out of the current loop or switch.
  **L450 CN**: 跳出当前循环或 switch。
- **L451 EN**: Provides the default branch for a `switch` statement.
  **L451 CN**: 为 `switch` 语句提供默认分支。
- **L452 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L452 CN**: 延续周围的声明、表达式或控制流结构。
- **L453 EN**: Executes statement involving `code`.
  **L453 CN**: 执行涉及 `code` 的语句。
- **L454 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L454 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L455 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L455 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L456 EN**: Blank line separates nearby declarations or logic blocks.
  **L456 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 457-480

````cpp
extern "C" {
RT_EXT_API_GROUP_BEGIN

void RTDEF(MaxlocDim)(Descriptor &result, const Descriptor &x, int kind,
    int dim, const char *source, int line, const Descriptor *mask, bool back) {
  TypedPartialMaxOrMinLoc<true>(
      "MAXLOC", result, x, kind, dim, source, line, mask, back);
}
void RTDEF(MinlocDim)(Descriptor &result, const Descriptor &x, int kind,
    int dim, const char *source, int line, const Descriptor *mask, bool back) {
  TypedPartialMaxOrMinLoc<false>(
      "MINLOC", result, x, kind, dim, source, line, mask, back);
}

RT_EXT_API_GROUP_END
} // extern "C"

// MAXVAL and MINVAL

template <TypeCategory CAT, int KIND, bool IS_MAXVAL>
class NumericExtremumAccumulator {
public:
  using Type = CppTypeFor<CAT, KIND>;
  explicit RT_API_ATTRS NumericExtremumAccumulator(const Descriptor &array)
````

- **L457 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L457 CN**: 延续周围的声明、表达式或控制流结构。
- **L458 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L458 CN**: 延续周围的声明、表达式或控制流结构。
- **L459 EN**: Blank line separates nearby declarations or logic blocks.
  **L459 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L460 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L460 CN**: 延续周围的声明、表达式或控制流结构。
- **L461 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L461 CN**: 延续周围的声明、表达式或控制流结构。
- **L462 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L462 CN**: 延续周围的声明、表达式或控制流结构。
- **L463 EN**: Executes statement `"MAXLOC", result, x, kind, dim, source, line, mask, back);`.
  **L463 CN**: 执行语句 `"MAXLOC", result, x, kind, dim, source, line, mask, back);`。
- **L464 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L464 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L465 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L465 CN**: 延续周围的声明、表达式或控制流结构。
- **L466 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L466 CN**: 延续周围的声明、表达式或控制流结构。
- **L467 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L467 CN**: 延续周围的声明、表达式或控制流结构。
- **L468 EN**: Executes statement `"MINLOC", result, x, kind, dim, source, line, mask, back);`.
  **L468 CN**: 执行语句 `"MINLOC", result, x, kind, dim, source, line, mask, back);`。
- **L469 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L469 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L470 EN**: Blank line separates nearby declarations or logic blocks.
  **L470 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L471 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L471 CN**: 延续周围的声明、表达式或控制流结构。
- **L472 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L472 CN**: 延续周围的声明、表达式或控制流结构。
- **L473 EN**: Blank line separates nearby declarations or logic blocks.
  **L473 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L474 EN**: Comment documents intent or context: `MAXVAL and MINVAL`.
  **L474 CN**: 注释记录了意图或上下文：`MAXVAL and MINVAL`。
- **L475 EN**: Blank line separates nearby declarations or logic blocks.
  **L475 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L476 EN**: Begins a template declaration parameterizing subsequent code.
  **L476 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L477 EN**: Declares or defines class `NumericExtremumAccumulator`.
  **L477 CN**: 声明或定义 class `NumericExtremumAccumulator`。
- **L478 EN**: Defines label or access section `public`.
  **L478 CN**: 定义标签或访问区段 `public`。
- **L479 EN**: Defines type alias `Type` for readability or ABI convenience.
  **L479 CN**: 定义类型别名 `Type`，以提升可读性或满足 ABI 便利性。
- **L480 EN**: Declares or defines callable `NumericExtremumAccumulator`.
  **L480 CN**: 声明或定义可调用实体 `NumericExtremumAccumulator`。

### Lines 481-504

````cpp
      : array_{array} {}
  RT_API_ATTRS void Reinitialize() {
    any_ = false;
    extremum_ = MaxOrMinIdentity<CAT, KIND, IS_MAXVAL>::Value();
  }
  template <typename A>
  RT_API_ATTRS void GetResult(A *p, int /*zeroBasedDim*/ = -1) const {
    *p = extremum_;
  }
  RT_API_ATTRS bool Accumulate(Type x) {
    if (!any_) {
      extremum_ = x;
      any_ = true;
    } else if (CAT == TypeCategory::Real && extremum_ != extremum_) {
      extremum_ = x; // replace NaN
    } else if constexpr (IS_MAXVAL) {
      if (x > extremum_) {
        extremum_ = x;
      }
    } else if (x < extremum_) {
      extremum_ = x;
    }
    return true;
  }
````

- **L481 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L481 CN**: 延续周围的声明、表达式或控制流结构。
- **L482 EN**: Declares or defines callable `Reinitialize`.
  **L482 CN**: 声明或定义可调用实体 `Reinitialize`。
- **L483 EN**: Initializes or updates `any_`.
  **L483 CN**: 初始化或更新 `any_`。
- **L484 EN**: Initializes or updates `extremum_`.
  **L484 CN**: 初始化或更新 `extremum_`。
- **L485 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L485 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L486 EN**: Begins a template declaration parameterizing subsequent code.
  **L486 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L487 EN**: Declares or defines callable `GetResult`.
  **L487 CN**: 声明或定义可调用实体 `GetResult`。
- **L488 EN**: Comment documents intent or context: `p = extremum_;`.
  **L488 CN**: 注释记录了意图或上下文：`p = extremum_;`。
- **L489 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L489 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L490 EN**: Declares or defines callable `Accumulate`.
  **L490 CN**: 声明或定义可调用实体 `Accumulate`。
- **L491 EN**: Introduces conditional control flow with an `if` statement.
  **L491 CN**: 通过 `if` 语句引入条件控制流。
- **L492 EN**: Initializes or updates `extremum_`.
  **L492 CN**: 初始化或更新 `extremum_`。
- **L493 EN**: Initializes or updates `any_`.
  **L493 CN**: 初始化或更新 `any_`。
- **L494 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L494 CN**: 延续周围的声明、表达式或控制流结构。
- **L495 EN**: Initializes or updates `extremum_`.
  **L495 CN**: 初始化或更新 `extremum_`。
- **L496 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L496 CN**: 延续周围的声明、表达式或控制流结构。
- **L497 EN**: Introduces conditional control flow with an `if` statement.
  **L497 CN**: 通过 `if` 语句引入条件控制流。
- **L498 EN**: Initializes or updates `extremum_`.
  **L498 CN**: 初始化或更新 `extremum_`。
- **L499 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L499 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L500 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L500 CN**: 延续周围的声明、表达式或控制流结构。
- **L501 EN**: Initializes or updates `extremum_`.
  **L501 CN**: 初始化或更新 `extremum_`。
- **L502 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L502 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L503 EN**: Returns from the current function, often propagating a computed result.
  **L503 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L504 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L504 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 505-528

````cpp
  template <typename A>
  RT_API_ATTRS bool AccumulateAt(const SubscriptValue at[]) {
    return Accumulate(*array_.Element<A>(at));
  }

private:
  const Descriptor &array_;
  bool any_{false};
  Type extremum_{MaxOrMinIdentity<CAT, KIND, IS_MAXVAL>::Value()};
};

template <TypeCategory CAT, int KIND, bool IS_MAXVAL>
inline RT_API_ATTRS CppTypeFor<CAT, KIND> TotalNumericMaxOrMin(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask, const char *intrinsic) {
  return GetTotalReduction<CAT, KIND>(x, source, line, dim, mask,
      NumericExtremumAccumulator<CAT, KIND, IS_MAXVAL>{x}, intrinsic);
}

template <TypeCategory CAT, bool IS_MAXVAL> struct MaxOrMinHelper {
  template <int KIND> struct Functor {
    RT_API_ATTRS void operator()(Descriptor &result, const Descriptor &x,
        int dim, const Descriptor *mask, const char *intrinsic,
        Terminator &terminator) const {
````

- **L505 EN**: Begins a template declaration parameterizing subsequent code.
  **L505 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L506 EN**: Declares or defines callable `AccumulateAt`.
  **L506 CN**: 声明或定义可调用实体 `AccumulateAt`。
- **L507 EN**: Returns from the current function, often propagating a computed result.
  **L507 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L508 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L508 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L509 EN**: Blank line separates nearby declarations or logic blocks.
  **L509 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L510 EN**: Defines label or access section `private`.
  **L510 CN**: 定义标签或访问区段 `private`。
- **L511 EN**: Executes statement `const Descriptor &array_;`.
  **L511 CN**: 执行语句 `const Descriptor &array_;`。
- **L512 EN**: Executes statement `bool any_{false};`.
  **L512 CN**: 执行语句 `bool any_{false};`。
- **L513 EN**: Executes statement involving `Value`.
  **L513 CN**: 执行涉及 `Value` 的语句。
- **L514 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L514 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L515 EN**: Blank line separates nearby declarations or logic blocks.
  **L515 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L516 EN**: Begins a template declaration parameterizing subsequent code.
  **L516 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L517 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L517 CN**: 延续周围的声明、表达式或控制流结构。
- **L518 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L518 CN**: 延续周围的声明、表达式或控制流结构。
- **L519 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L519 CN**: 延续周围的声明、表达式或控制流结构。
- **L520 EN**: Returns from the current function, often propagating a computed result.
  **L520 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L521 EN**: Executes statement `NumericExtremumAccumulator<CAT, KIND, IS_MAXVAL>{x}, intrinsic);`.
  **L521 CN**: 执行语句 `NumericExtremumAccumulator<CAT, KIND, IS_MAXVAL>{x}, intrinsic);`。
- **L522 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L522 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L523 EN**: Blank line separates nearby declarations or logic blocks.
  **L523 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L524 EN**: Begins a template declaration parameterizing subsequent code.
  **L524 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L525 EN**: Begins a template declaration parameterizing subsequent code.
  **L525 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L526 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L526 CN**: 延续周围的声明、表达式或控制流结构。
- **L527 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L527 CN**: 延续周围的声明、表达式或控制流结构。
- **L528 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L528 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 529-552

````cpp
      DoMaxMinNorm2<CAT, KIND,
          NumericExtremumAccumulator<CAT, KIND, IS_MAXVAL>>(
          result, x, dim, mask, intrinsic, terminator);
    }
  };
};

template <bool IS_MAXVAL>
inline RT_API_ATTRS void NumericMaxOrMin(Descriptor &result,
    const Descriptor &x, int dim, const char *source, int line,
    const Descriptor *mask, const char *intrinsic) {
  Terminator terminator{source, line};
  auto type{x.type().GetCategoryAndKind()};
  RUNTIME_CHECK(terminator, type);
  switch (type->first) {
  case TypeCategory::Integer:
    ApplyIntegerKind<
        MaxOrMinHelper<TypeCategory::Integer, IS_MAXVAL>::template Functor,
        void>(
        type->second, terminator, result, x, dim, mask, intrinsic, terminator);
    break;
  case TypeCategory::Unsigned:
    ApplyIntegerKind<
        MaxOrMinHelper<TypeCategory::Unsigned, IS_MAXVAL>::template Functor,
````

- **L529 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L529 CN**: 延续周围的声明、表达式或控制流结构。
- **L530 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L530 CN**: 延续周围的声明、表达式或控制流结构。
- **L531 EN**: Executes statement `result, x, dim, mask, intrinsic, terminator);`.
  **L531 CN**: 执行语句 `result, x, dim, mask, intrinsic, terminator);`。
- **L532 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L532 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L533 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L533 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L534 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L534 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L535 EN**: Blank line separates nearby declarations or logic blocks.
  **L535 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L536 EN**: Begins a template declaration parameterizing subsequent code.
  **L536 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L537 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L537 CN**: 延续周围的声明、表达式或控制流结构。
- **L538 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L538 CN**: 延续周围的声明、表达式或控制流结构。
- **L539 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L539 CN**: 延续周围的声明、表达式或控制流结构。
- **L540 EN**: Executes statement `Terminator terminator{source, line};`.
  **L540 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L541 EN**: Executes statement involving `type`.
  **L541 CN**: 执行涉及 `type` 的语句。
- **L542 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L542 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L543 EN**: Begins a `switch` dispatch over discrete cases.
  **L543 CN**: 开始一个针对离散分支的 `switch` 分派。
- **L544 EN**: Marks one `switch` case label.
  **L544 CN**: 标记一个 `switch` 的 case 标签。
- **L545 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L545 CN**: 延续周围的声明、表达式或控制流结构。
- **L546 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L546 CN**: 延续周围的声明、表达式或控制流结构。
- **L547 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L547 CN**: 延续周围的声明、表达式或控制流结构。
- **L548 EN**: Executes statement `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`.
  **L548 CN**: 执行语句 `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`。
- **L549 EN**: Breaks out of the current loop or switch.
  **L549 CN**: 跳出当前循环或 switch。
- **L550 EN**: Marks one `switch` case label.
  **L550 CN**: 标记一个 `switch` 的 case 标签。
- **L551 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L551 CN**: 延续周围的声明、表达式或控制流结构。
- **L552 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L552 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 553-576

````cpp
        void>(
        type->second, terminator, result, x, dim, mask, intrinsic, terminator);
    break;
  case TypeCategory::Real:
    ApplyFloatingPointKind<
        MaxOrMinHelper<TypeCategory::Real, IS_MAXVAL>::template Functor, void>(
        type->second, terminator, result, x, dim, mask, intrinsic, terminator);
    break;
  default:
    terminator.Crash("%s: bad type code %d", intrinsic, x.type().raw());
  }
}

template <int KIND, bool IS_MAXVAL> class CharacterExtremumAccumulator {
public:
  using Type = CppTypeFor<TypeCategory::Character, KIND>;
  explicit RT_API_ATTRS CharacterExtremumAccumulator(const Descriptor &array)
      : array_{array}, charLen_{array_.ElementBytes() / KIND} {}
  RT_API_ATTRS void Reinitialize() { extremum_ = nullptr; }
  template <typename A>
  RT_API_ATTRS void GetResult(A *p, int /*zeroBasedDim*/ = -1) const {
    static_assert(std::is_same_v<A, Type>);
    std::size_t byteSize{array_.ElementBytes()};
    if (extremum_) {
````

- **L553 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L553 CN**: 延续周围的声明、表达式或控制流结构。
- **L554 EN**: Executes statement `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`.
  **L554 CN**: 执行语句 `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`。
- **L555 EN**: Breaks out of the current loop or switch.
  **L555 CN**: 跳出当前循环或 switch。
- **L556 EN**: Marks one `switch` case label.
  **L556 CN**: 标记一个 `switch` 的 case 标签。
- **L557 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L557 CN**: 延续周围的声明、表达式或控制流结构。
- **L558 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L558 CN**: 延续周围的声明、表达式或控制流结构。
- **L559 EN**: Executes statement `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`.
  **L559 CN**: 执行语句 `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`。
- **L560 EN**: Breaks out of the current loop or switch.
  **L560 CN**: 跳出当前循环或 switch。
- **L561 EN**: Provides the default branch for a `switch` statement.
  **L561 CN**: 为 `switch` 语句提供默认分支。
- **L562 EN**: Executes statement involving `Crash`.
  **L562 CN**: 执行涉及 `Crash` 的语句。
- **L563 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L563 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L564 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L564 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L565 EN**: Blank line separates nearby declarations or logic blocks.
  **L565 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L566 EN**: Begins a template declaration parameterizing subsequent code.
  **L566 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L567 EN**: Defines label or access section `public`.
  **L567 CN**: 定义标签或访问区段 `public`。
- **L568 EN**: Defines type alias `Type` for readability or ABI convenience.
  **L568 CN**: 定义类型别名 `Type`，以提升可读性或满足 ABI 便利性。
- **L569 EN**: Declares or defines callable `CharacterExtremumAccumulator`.
  **L569 CN**: 声明或定义可调用实体 `CharacterExtremumAccumulator`。
- **L570 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L570 CN**: 延续周围的声明、表达式或控制流结构。
- **L571 EN**: Initializes or updates `extremum_`.
  **L571 CN**: 初始化或更新 `extremum_`。
- **L572 EN**: Begins a template declaration parameterizing subsequent code.
  **L572 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L573 EN**: Declares or defines callable `GetResult`.
  **L573 CN**: 声明或定义可调用实体 `GetResult`。
- **L574 EN**: Performs a compile-time assertion to enforce invariants.
  **L574 CN**: 执行编译期断言以约束不变量。
- **L575 EN**: Executes statement involving `ElementBytes`.
  **L575 CN**: 执行涉及 `ElementBytes` 的语句。
- **L576 EN**: Introduces conditional control flow with an `if` statement.
  **L576 CN**: 通过 `if` 语句引入条件控制流。

### Lines 577-600

````cpp
      runtime::memcpy(p, extremum_, byteSize);
    } else {
      // Empty array; fill with character 0 for MAXVAL.
      // For MINVAL, set all of the bits.
      runtime::memset(p, IS_MAXVAL ? 0 : 255, byteSize);
    }
  }
  RT_API_ATTRS bool Accumulate(const Type *x) {
    if (!extremum_) {
      extremum_ = x;
    } else {
      int cmp{CharacterScalarCompare(x, extremum_, charLen_, charLen_)};
      if (IS_MAXVAL == (cmp > 0)) {
        extremum_ = x;
      }
    }
    return true;
  }
  template <typename A>
  RT_API_ATTRS bool AccumulateAt(const SubscriptValue at[]) {
    return Accumulate(array_.Element<A>(at));
  }

private:
````

- **L577 EN**: Executes statement involving `memcpy`.
  **L577 CN**: 执行涉及 `memcpy` 的语句。
- **L578 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L578 CN**: 延续周围的声明、表达式或控制流结构。
- **L579 EN**: Comment documents intent or context: `Empty array; fill with character 0 for MAXVAL.`.
  **L579 CN**: 注释记录了意图或上下文：`Empty array; fill with character 0 for MAXVAL.`。
- **L580 EN**: Comment documents intent or context: `For MINVAL, set all of the bits.`.
  **L580 CN**: 注释记录了意图或上下文：`For MINVAL, set all of the bits.`。
- **L581 EN**: Executes statement involving `memset`.
  **L581 CN**: 执行涉及 `memset` 的语句。
- **L582 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L582 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L583 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L583 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L584 EN**: Declares or defines callable `Accumulate`.
  **L584 CN**: 声明或定义可调用实体 `Accumulate`。
- **L585 EN**: Introduces conditional control flow with an `if` statement.
  **L585 CN**: 通过 `if` 语句引入条件控制流。
- **L586 EN**: Initializes or updates `extremum_`.
  **L586 CN**: 初始化或更新 `extremum_`。
- **L587 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L587 CN**: 延续周围的声明、表达式或控制流结构。
- **L588 EN**: Executes statement involving `CharacterScalarCompare`.
  **L588 CN**: 执行涉及 `CharacterScalarCompare` 的语句。
- **L589 EN**: Introduces conditional control flow with an `if` statement.
  **L589 CN**: 通过 `if` 语句引入条件控制流。
- **L590 EN**: Initializes or updates `extremum_`.
  **L590 CN**: 初始化或更新 `extremum_`。
- **L591 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L591 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L592 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L592 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L593 EN**: Returns from the current function, often propagating a computed result.
  **L593 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L594 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L594 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L595 EN**: Begins a template declaration parameterizing subsequent code.
  **L595 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L596 EN**: Declares or defines callable `AccumulateAt`.
  **L596 CN**: 声明或定义可调用实体 `AccumulateAt`。
- **L597 EN**: Returns from the current function, often propagating a computed result.
  **L597 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L598 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L598 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L599 EN**: Blank line separates nearby declarations or logic blocks.
  **L599 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L600 EN**: Defines label or access section `private`.
  **L600 CN**: 定义标签或访问区段 `private`。

### Lines 601-624

````cpp
  const Descriptor &array_;
  std::size_t charLen_;
  const Type *extremum_{nullptr};
};

template <bool IS_MAXVAL> struct CharacterMaxOrMinHelper {
  template <int KIND> struct Functor {
    RT_API_ATTRS void operator()(Descriptor &result, const Descriptor &x,
        int dim, const Descriptor *mask, const char *intrinsic,
        Terminator &terminator) const {
      DoMaxMinNorm2<TypeCategory::Character, KIND,
          CharacterExtremumAccumulator<KIND, IS_MAXVAL>>(
          result, x, dim, mask, intrinsic, terminator);
    }
  };
};

template <bool IS_MAXVAL>
inline RT_API_ATTRS void CharacterMaxOrMin(Descriptor &result,
    const Descriptor &x, int dim, const char *source, int line,
    const Descriptor *mask, const char *intrinsic) {
  Terminator terminator{source, line};
  auto type{x.type().GetCategoryAndKind()};
  RUNTIME_CHECK(terminator, type && type->first == TypeCategory::Character);
````

- **L601 EN**: Executes statement `const Descriptor &array_;`.
  **L601 CN**: 执行语句 `const Descriptor &array_;`。
- **L602 EN**: Executes statement `std::size_t charLen_;`.
  **L602 CN**: 执行语句 `std::size_t charLen_;`。
- **L603 EN**: Executes statement `const Type *extremum_{nullptr};`.
  **L603 CN**: 执行语句 `const Type *extremum_{nullptr};`。
- **L604 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L604 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L605 EN**: Blank line separates nearby declarations or logic blocks.
  **L605 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L606 EN**: Begins a template declaration parameterizing subsequent code.
  **L606 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L607 EN**: Begins a template declaration parameterizing subsequent code.
  **L607 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L608 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L608 CN**: 延续周围的声明、表达式或控制流结构。
- **L609 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L609 CN**: 延续周围的声明、表达式或控制流结构。
- **L610 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L610 CN**: 延续周围的声明、表达式或控制流结构。
- **L611 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L611 CN**: 延续周围的声明、表达式或控制流结构。
- **L612 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L612 CN**: 延续周围的声明、表达式或控制流结构。
- **L613 EN**: Executes statement `result, x, dim, mask, intrinsic, terminator);`.
  **L613 CN**: 执行语句 `result, x, dim, mask, intrinsic, terminator);`。
- **L614 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L614 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L615 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L615 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L616 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L616 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L617 EN**: Blank line separates nearby declarations or logic blocks.
  **L617 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L618 EN**: Begins a template declaration parameterizing subsequent code.
  **L618 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L619 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L619 CN**: 延续周围的声明、表达式或控制流结构。
- **L620 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L620 CN**: 延续周围的声明、表达式或控制流结构。
- **L621 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L621 CN**: 延续周围的声明、表达式或控制流结构。
- **L622 EN**: Executes statement `Terminator terminator{source, line};`.
  **L622 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L623 EN**: Executes statement involving `type`.
  **L623 CN**: 执行涉及 `type` 的语句。
- **L624 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L624 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。

### Lines 625-648

````cpp
  ApplyCharacterKind<CharacterMaxOrMinHelper<IS_MAXVAL>::template Functor,
      void>(
      type->second, terminator, result, x, dim, mask, intrinsic, terminator);
}

extern "C" {
RT_EXT_API_GROUP_BEGIN

CppTypeFor<TypeCategory::Integer, 1> RTDEF(MaxvalInteger1)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 1, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Integer, 2> RTDEF(MaxvalInteger2)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 2, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Integer, 4> RTDEF(MaxvalInteger4)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 4, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Integer, 8> RTDEF(MaxvalInteger8)(const Descriptor &x,
````

- **L625 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L625 CN**: 延续周围的声明、表达式或控制流结构。
- **L626 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L626 CN**: 延续周围的声明、表达式或控制流结构。
- **L627 EN**: Executes statement `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`.
  **L627 CN**: 执行语句 `type->second, terminator, result, x, dim, mask, intrinsic, terminator);`。
- **L628 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L628 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L629 EN**: Blank line separates nearby declarations or logic blocks.
  **L629 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L630 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L630 CN**: 延续周围的声明、表达式或控制流结构。
- **L631 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L631 CN**: 延续周围的声明、表达式或控制流结构。
- **L632 EN**: Blank line separates nearby declarations or logic blocks.
  **L632 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L633 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L633 CN**: 延续周围的声明、表达式或控制流结构。
- **L634 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L634 CN**: 延续周围的声明、表达式或控制流结构。
- **L635 EN**: Returns from the current function, often propagating a computed result.
  **L635 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L636 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L636 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L637 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L637 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L638 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L638 CN**: 延续周围的声明、表达式或控制流结构。
- **L639 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L639 CN**: 延续周围的声明、表达式或控制流结构。
- **L640 EN**: Returns from the current function, often propagating a computed result.
  **L640 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L641 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L641 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L642 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L642 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L643 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L643 CN**: 延续周围的声明、表达式或控制流结构。
- **L644 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L644 CN**: 延续周围的声明、表达式或控制流结构。
- **L645 EN**: Returns from the current function, often propagating a computed result.
  **L645 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L646 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L646 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L647 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L647 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L648 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L648 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 649-672

````cpp
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 8, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#ifdef __SIZEOF_INT128__
CppTypeFor<TypeCategory::Integer, 16> RTDEF(MaxvalInteger16)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 16, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#endif

CppTypeFor<TypeCategory::Unsigned, 1> RTDEF(MaxvalUnsigned1)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 1, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Unsigned, 2> RTDEF(MaxvalUnsigned2)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 2, true>(
      x, source, line, dim, mask, "MAXVAL");
````

- **L649 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L649 CN**: 延续周围的声明、表达式或控制流结构。
- **L650 EN**: Returns from the current function, often propagating a computed result.
  **L650 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L651 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L651 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L652 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L652 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L653 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L653 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L654 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L654 CN**: 延续周围的声明、表达式或控制流结构。
- **L655 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L655 CN**: 延续周围的声明、表达式或控制流结构。
- **L656 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L656 CN**: 延续周围的声明、表达式或控制流结构。
- **L657 EN**: Returns from the current function, often propagating a computed result.
  **L657 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L658 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L658 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L659 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L659 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L660 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L660 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L661 EN**: Blank line separates nearby declarations or logic blocks.
  **L661 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L662 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L662 CN**: 延续周围的声明、表达式或控制流结构。
- **L663 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L663 CN**: 延续周围的声明、表达式或控制流结构。
- **L664 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L664 CN**: 延续周围的声明、表达式或控制流结构。
- **L665 EN**: Returns from the current function, often propagating a computed result.
  **L665 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L666 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L666 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L667 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L667 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L668 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L668 CN**: 延续周围的声明、表达式或控制流结构。
- **L669 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L669 CN**: 延续周围的声明、表达式或控制流结构。
- **L670 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L670 CN**: 延续周围的声明、表达式或控制流结构。
- **L671 EN**: Returns from the current function, often propagating a computed result.
  **L671 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L672 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L672 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。

### Lines 673-696

````cpp
}
CppTypeFor<TypeCategory::Unsigned, 4> RTDEF(MaxvalUnsigned4)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 4, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Unsigned, 8> RTDEF(MaxvalUnsigned8)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 8, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#ifdef __SIZEOF_INT128__
CppTypeFor<TypeCategory::Unsigned, 16> RTDEF(MaxvalUnsigned16)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 16, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#endif

// TODO: REAL(2 & 3)
CppTypeFor<TypeCategory::Real, 4> RTDEF(MaxvalReal4)(const Descriptor &x,
````

- **L673 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L673 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L674 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L674 CN**: 延续周围的声明、表达式或控制流结构。
- **L675 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L675 CN**: 延续周围的声明、表达式或控制流结构。
- **L676 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L676 CN**: 延续周围的声明、表达式或控制流结构。
- **L677 EN**: Returns from the current function, often propagating a computed result.
  **L677 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L678 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L678 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L679 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L679 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L680 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L680 CN**: 延续周围的声明、表达式或控制流结构。
- **L681 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L681 CN**: 延续周围的声明、表达式或控制流结构。
- **L682 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L682 CN**: 延续周围的声明、表达式或控制流结构。
- **L683 EN**: Returns from the current function, often propagating a computed result.
  **L683 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L684 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L684 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L685 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L685 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L686 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L686 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L687 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L687 CN**: 延续周围的声明、表达式或控制流结构。
- **L688 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L688 CN**: 延续周围的声明、表达式或控制流结构。
- **L689 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L689 CN**: 延续周围的声明、表达式或控制流结构。
- **L690 EN**: Returns from the current function, often propagating a computed result.
  **L690 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L691 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L691 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L692 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L692 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L693 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L693 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L694 EN**: Blank line separates nearby declarations or logic blocks.
  **L694 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L695 EN**: Comment documents intent or context: `TODO: REAL(2 & 3)`.
  **L695 CN**: 注释记录了意图或上下文：`TODO: REAL(2 & 3)`。
- **L696 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L696 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 697-720

````cpp
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 4, true>(
      x, source, line, dim, mask, "MAXVAL");
}
CppTypeFor<TypeCategory::Real, 8> RTDEF(MaxvalReal8)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 8, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#if HAS_FLOAT80
CppTypeFor<TypeCategory::Real, 10> RTDEF(MaxvalReal10)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 10, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
CppTypeFor<TypeCategory::Real, 16> RTDEF(MaxvalReal16)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 16, true>(
      x, source, line, dim, mask, "MAXVAL");
}
#endif

````

- **L697 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L697 CN**: 延续周围的声明、表达式或控制流结构。
- **L698 EN**: Returns from the current function, often propagating a computed result.
  **L698 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L699 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L699 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L700 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L700 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L701 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L701 CN**: 延续周围的声明、表达式或控制流结构。
- **L702 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L702 CN**: 延续周围的声明、表达式或控制流结构。
- **L703 EN**: Returns from the current function, often propagating a computed result.
  **L703 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L704 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L704 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L705 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L705 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L706 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L706 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L707 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L707 CN**: 延续周围的声明、表达式或控制流结构。
- **L708 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L708 CN**: 延续周围的声明、表达式或控制流结构。
- **L709 EN**: Returns from the current function, often propagating a computed result.
  **L709 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L710 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L710 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L711 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L711 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L712 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L712 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L713 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L713 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L714 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L714 CN**: 延续周围的声明、表达式或控制流结构。
- **L715 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L715 CN**: 延续周围的声明、表达式或控制流结构。
- **L716 EN**: Returns from the current function, often propagating a computed result.
  **L716 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L717 EN**: Executes statement `x, source, line, dim, mask, "MAXVAL");`.
  **L717 CN**: 执行语句 `x, source, line, dim, mask, "MAXVAL");`。
- **L718 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L718 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L719 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L719 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L720 EN**: Blank line separates nearby declarations or logic blocks.
  **L720 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 721-744

````cpp
void RTDEF(MaxvalCharacter)(Descriptor &result, const Descriptor &x,
    const char *source, int line, const Descriptor *mask) {
  CharacterMaxOrMin<true>(result, x, 0, source, line, mask, "MAXVAL");
}

CppTypeFor<TypeCategory::Integer, 1> RTDEF(MinvalInteger1)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 1, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Integer, 2> RTDEF(MinvalInteger2)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 2, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Integer, 4> RTDEF(MinvalInteger4)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 4, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Integer, 8> RTDEF(MinvalInteger8)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 8, false>(
      x, source, line, dim, mask, "MINVAL");
````

- **L721 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L721 CN**: 延续周围的声明、表达式或控制流结构。
- **L722 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L722 CN**: 延续周围的声明、表达式或控制流结构。
- **L723 EN**: Executes statement `CharacterMaxOrMin<true>(result, x, 0, source, line, mask, "MAXVAL");`.
  **L723 CN**: 执行语句 `CharacterMaxOrMin<true>(result, x, 0, source, line, mask, "MAXVAL");`。
- **L724 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L724 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L725 EN**: Blank line separates nearby declarations or logic blocks.
  **L725 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L726 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L726 CN**: 延续周围的声明、表达式或控制流结构。
- **L727 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L727 CN**: 延续周围的声明、表达式或控制流结构。
- **L728 EN**: Returns from the current function, often propagating a computed result.
  **L728 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L729 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L729 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L730 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L730 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L731 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L731 CN**: 延续周围的声明、表达式或控制流结构。
- **L732 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L732 CN**: 延续周围的声明、表达式或控制流结构。
- **L733 EN**: Returns from the current function, often propagating a computed result.
  **L733 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L734 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L734 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L735 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L735 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L736 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L736 CN**: 延续周围的声明、表达式或控制流结构。
- **L737 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L737 CN**: 延续周围的声明、表达式或控制流结构。
- **L738 EN**: Returns from the current function, often propagating a computed result.
  **L738 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L739 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L739 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L740 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L740 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L741 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L741 CN**: 延续周围的声明、表达式或控制流结构。
- **L742 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L742 CN**: 延续周围的声明、表达式或控制流结构。
- **L743 EN**: Returns from the current function, often propagating a computed result.
  **L743 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L744 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L744 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。

### Lines 745-768

````cpp
}
#ifdef __SIZEOF_INT128__
CppTypeFor<TypeCategory::Integer, 16> RTDEF(MinvalInteger16)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Integer, 16, false>(
      x, source, line, dim, mask, "MINVAL");
}
#endif

CppTypeFor<TypeCategory::Unsigned, 1> RTDEF(MinvalUnsigned1)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 1, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Unsigned, 2> RTDEF(MinvalUnsigned2)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 2, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Unsigned, 4> RTDEF(MinvalUnsigned4)(
    const Descriptor &x, const char *source, int line, int dim,
````

- **L745 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L745 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L746 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L746 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L747 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L747 CN**: 延续周围的声明、表达式或控制流结构。
- **L748 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L748 CN**: 延续周围的声明、表达式或控制流结构。
- **L749 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L749 CN**: 延续周围的声明、表达式或控制流结构。
- **L750 EN**: Returns from the current function, often propagating a computed result.
  **L750 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L751 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L751 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L752 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L752 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L753 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L753 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L754 EN**: Blank line separates nearby declarations or logic blocks.
  **L754 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L755 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L755 CN**: 延续周围的声明、表达式或控制流结构。
- **L756 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L756 CN**: 延续周围的声明、表达式或控制流结构。
- **L757 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L757 CN**: 延续周围的声明、表达式或控制流结构。
- **L758 EN**: Returns from the current function, often propagating a computed result.
  **L758 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L759 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L759 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L760 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L760 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L761 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L761 CN**: 延续周围的声明、表达式或控制流结构。
- **L762 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L762 CN**: 延续周围的声明、表达式或控制流结构。
- **L763 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L763 CN**: 延续周围的声明、表达式或控制流结构。
- **L764 EN**: Returns from the current function, often propagating a computed result.
  **L764 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L765 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L765 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L766 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L766 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L767 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L767 CN**: 延续周围的声明、表达式或控制流结构。
- **L768 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L768 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 769-792

````cpp
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 4, false>(
      x, source, line, dim, mask, "MINVAL");
}
CppTypeFor<TypeCategory::Unsigned, 8> RTDEF(MinvalUnsigned8)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 8, false>(
      x, source, line, dim, mask, "MINVAL");
}
#ifdef __SIZEOF_INT128__
CppTypeFor<TypeCategory::Unsigned, 16> RTDEF(MinvalUnsigned16)(
    const Descriptor &x, const char *source, int line, int dim,
    const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Unsigned, 16, false>(
      x, source, line, dim, mask, "MINVAL");
}
#endif

// TODO: REAL(2 & 3)
CppTypeFor<TypeCategory::Real, 4> RTDEF(MinvalReal4)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 4, false>(
      x, source, line, dim, mask, "MINVAL");
````

- **L769 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L769 CN**: 延续周围的声明、表达式或控制流结构。
- **L770 EN**: Returns from the current function, often propagating a computed result.
  **L770 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L771 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L771 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L772 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L772 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L773 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L773 CN**: 延续周围的声明、表达式或控制流结构。
- **L774 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L774 CN**: 延续周围的声明、表达式或控制流结构。
- **L775 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L775 CN**: 延续周围的声明、表达式或控制流结构。
- **L776 EN**: Returns from the current function, often propagating a computed result.
  **L776 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L777 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L777 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L778 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L778 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L779 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L779 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L780 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L780 CN**: 延续周围的声明、表达式或控制流结构。
- **L781 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L781 CN**: 延续周围的声明、表达式或控制流结构。
- **L782 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L782 CN**: 延续周围的声明、表达式或控制流结构。
- **L783 EN**: Returns from the current function, often propagating a computed result.
  **L783 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L784 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L784 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L785 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L785 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L786 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L786 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L787 EN**: Blank line separates nearby declarations or logic blocks.
  **L787 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L788 EN**: Comment documents intent or context: `TODO: REAL(2 & 3)`.
  **L788 CN**: 注释记录了意图或上下文：`TODO: REAL(2 & 3)`。
- **L789 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L789 CN**: 延续周围的声明、表达式或控制流结构。
- **L790 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L790 CN**: 延续周围的声明、表达式或控制流结构。
- **L791 EN**: Returns from the current function, often propagating a computed result.
  **L791 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L792 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L792 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。

### Lines 793-816

````cpp
}
CppTypeFor<TypeCategory::Real, 8> RTDEF(MinvalReal8)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 8, false>(
      x, source, line, dim, mask, "MINVAL");
}
#if HAS_FLOAT80
CppTypeFor<TypeCategory::Real, 10> RTDEF(MinvalReal10)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 10, false>(
      x, source, line, dim, mask, "MINVAL");
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
CppTypeFor<TypeCategory::Real, 16> RTDEF(MinvalReal16)(const Descriptor &x,
    const char *source, int line, int dim, const Descriptor *mask) {
  return TotalNumericMaxOrMin<TypeCategory::Real, 16, false>(
      x, source, line, dim, mask, "MINVAL");
}
#endif

void RTDEF(MinvalCharacter)(Descriptor &result, const Descriptor &x,
    const char *source, int line, const Descriptor *mask) {
  CharacterMaxOrMin<false>(result, x, 0, source, line, mask, "MINVAL");
````

- **L793 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L793 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L794 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L794 CN**: 延续周围的声明、表达式或控制流结构。
- **L795 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L795 CN**: 延续周围的声明、表达式或控制流结构。
- **L796 EN**: Returns from the current function, often propagating a computed result.
  **L796 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L797 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L797 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L798 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L798 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L799 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L799 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L800 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L800 CN**: 延续周围的声明、表达式或控制流结构。
- **L801 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L801 CN**: 延续周围的声明、表达式或控制流结构。
- **L802 EN**: Returns from the current function, often propagating a computed result.
  **L802 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L803 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L803 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L804 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L804 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L805 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L805 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L806 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L806 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L807 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L807 CN**: 延续周围的声明、表达式或控制流结构。
- **L808 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L808 CN**: 延续周围的声明、表达式或控制流结构。
- **L809 EN**: Returns from the current function, often propagating a computed result.
  **L809 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L810 EN**: Executes statement `x, source, line, dim, mask, "MINVAL");`.
  **L810 CN**: 执行语句 `x, source, line, dim, mask, "MINVAL");`。
- **L811 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L811 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L812 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L812 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L813 EN**: Blank line separates nearby declarations or logic blocks.
  **L813 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L814 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L814 CN**: 延续周围的声明、表达式或控制流结构。
- **L815 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L815 CN**: 延续周围的声明、表达式或控制流结构。
- **L816 EN**: Executes statement `CharacterMaxOrMin<false>(result, x, 0, source, line, mask, "MINVAL");`.
  **L816 CN**: 执行语句 `CharacterMaxOrMin<false>(result, x, 0, source, line, mask, "MINVAL");`。

### Lines 817-840

````cpp
}

void RTDEF(MaxvalDim)(Descriptor &result, const Descriptor &x, int dim,
    const char *source, int line, const Descriptor *mask) {
  if (x.type().IsCharacter()) {
    CharacterMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");
  } else {
    NumericMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");
  }
}
void RTDEF(MinvalDim)(Descriptor &result, const Descriptor &x, int dim,
    const char *source, int line, const Descriptor *mask) {
  if (x.type().IsCharacter()) {
    CharacterMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");
  } else {
    NumericMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");
  }
}

RT_EXT_API_GROUP_END
} // extern "C"

// NORM2

````

- **L817 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L817 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L818 EN**: Blank line separates nearby declarations or logic blocks.
  **L818 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L819 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L819 CN**: 延续周围的声明、表达式或控制流结构。
- **L820 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L820 CN**: 延续周围的声明、表达式或控制流结构。
- **L821 EN**: Introduces conditional control flow with an `if` statement.
  **L821 CN**: 通过 `if` 语句引入条件控制流。
- **L822 EN**: Executes statement `CharacterMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");`.
  **L822 CN**: 执行语句 `CharacterMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");`。
- **L823 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L823 CN**: 延续周围的声明、表达式或控制流结构。
- **L824 EN**: Executes statement `NumericMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");`.
  **L824 CN**: 执行语句 `NumericMaxOrMin<true>(result, x, dim, source, line, mask, "MAXVAL");`。
- **L825 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L825 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L826 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L826 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L827 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L827 CN**: 延续周围的声明、表达式或控制流结构。
- **L828 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L828 CN**: 延续周围的声明、表达式或控制流结构。
- **L829 EN**: Introduces conditional control flow with an `if` statement.
  **L829 CN**: 通过 `if` 语句引入条件控制流。
- **L830 EN**: Executes statement `CharacterMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");`.
  **L830 CN**: 执行语句 `CharacterMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");`。
- **L831 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L831 CN**: 延续周围的声明、表达式或控制流结构。
- **L832 EN**: Executes statement `NumericMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");`.
  **L832 CN**: 执行语句 `NumericMaxOrMin<false>(result, x, dim, source, line, mask, "MINVAL");`。
- **L833 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L833 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L834 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L834 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L835 EN**: Blank line separates nearby declarations or logic blocks.
  **L835 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L836 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L836 CN**: 延续周围的声明、表达式或控制流结构。
- **L837 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L837 CN**: 延续周围的声明、表达式或控制流结构。
- **L838 EN**: Blank line separates nearby declarations or logic blocks.
  **L838 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L839 EN**: Comment documents intent or context: `NORM2`.
  **L839 CN**: 注释记录了意图或上下文：`NORM2`。
- **L840 EN**: Blank line separates nearby declarations or logic blocks.
  **L840 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 841-864

````cpp
extern "C" {
RT_EXT_API_GROUP_BEGIN

// TODO: REAL(2 & 3)
CppTypeFor<TypeCategory::Real, 4> RTDEF(Norm2_4)(
    const Descriptor &x, const char *source, int line, int dim) {
  return GetTotalReduction<TypeCategory::Real, 4>(
      x, source, line, dim, nullptr, Norm2Accumulator<4>{x}, "NORM2");
}
CppTypeFor<TypeCategory::Real, 8> RTDEF(Norm2_8)(
    const Descriptor &x, const char *source, int line, int dim) {
  return GetTotalReduction<TypeCategory::Real, 8>(
      x, source, line, dim, nullptr, Norm2Accumulator<8>{x}, "NORM2");
}
#if HAS_FLOAT80
CppTypeFor<TypeCategory::Real, 10> RTDEF(Norm2_10)(
    const Descriptor &x, const char *source, int line, int dim) {
  return GetTotalReduction<TypeCategory::Real, 10>(
      x, source, line, dim, nullptr, Norm2Accumulator<10>{x}, "NORM2");
}
#endif

void RTDEF(Norm2Dim)(Descriptor &result, const Descriptor &x, int dim,
    const char *source, int line) {
````

- **L841 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L841 CN**: 延续周围的声明、表达式或控制流结构。
- **L842 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L842 CN**: 延续周围的声明、表达式或控制流结构。
- **L843 EN**: Blank line separates nearby declarations or logic blocks.
  **L843 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L844 EN**: Comment documents intent or context: `TODO: REAL(2 & 3)`.
  **L844 CN**: 注释记录了意图或上下文：`TODO: REAL(2 & 3)`。
- **L845 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L845 CN**: 延续周围的声明、表达式或控制流结构。
- **L846 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L846 CN**: 延续周围的声明、表达式或控制流结构。
- **L847 EN**: Returns from the current function, often propagating a computed result.
  **L847 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L848 EN**: Executes statement `x, source, line, dim, nullptr, Norm2Accumulator<4>{x}, "NORM2");`.
  **L848 CN**: 执行语句 `x, source, line, dim, nullptr, Norm2Accumulator<4>{x}, "NORM2");`。
- **L849 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L849 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L850 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L850 CN**: 延续周围的声明、表达式或控制流结构。
- **L851 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L851 CN**: 延续周围的声明、表达式或控制流结构。
- **L852 EN**: Returns from the current function, often propagating a computed result.
  **L852 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L853 EN**: Executes statement `x, source, line, dim, nullptr, Norm2Accumulator<8>{x}, "NORM2");`.
  **L853 CN**: 执行语句 `x, source, line, dim, nullptr, Norm2Accumulator<8>{x}, "NORM2");`。
- **L854 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L854 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L855 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L855 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L856 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L856 CN**: 延续周围的声明、表达式或控制流结构。
- **L857 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L857 CN**: 延续周围的声明、表达式或控制流结构。
- **L858 EN**: Returns from the current function, often propagating a computed result.
  **L858 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L859 EN**: Executes statement `x, source, line, dim, nullptr, Norm2Accumulator<10>{x}, "NORM2");`.
  **L859 CN**: 执行语句 `x, source, line, dim, nullptr, Norm2Accumulator<10>{x}, "NORM2");`。
- **L860 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L860 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L861 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L861 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L862 EN**: Blank line separates nearby declarations or logic blocks.
  **L862 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L863 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L863 CN**: 延续周围的声明、表达式或控制流结构。
- **L864 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L864 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 865-878

````cpp
  Terminator terminator{source, line};
  auto type{x.type().GetCategoryAndKind()};
  RUNTIME_CHECK(terminator, type);
  if (type->first == TypeCategory::Real) {
    ApplyFloatingPointKind<Norm2Helper, void, true>(
        type->second, terminator, result, x, dim, nullptr, terminator);
  } else {
    terminator.Crash("NORM2: bad type code %d", x.type().raw());
  }
}

RT_EXT_API_GROUP_END
} // extern "C"
} // namespace Fortran::runtime
````

- **L865 EN**: Executes statement `Terminator terminator{source, line};`.
  **L865 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L866 EN**: Executes statement involving `type`.
  **L866 CN**: 执行涉及 `type` 的语句。
- **L867 EN**: Executes statement involving `RUNTIME_CHECK`.
  **L867 CN**: 执行涉及 `RUNTIME_CHECK` 的语句。
- **L868 EN**: Introduces conditional control flow with an `if` statement.
  **L868 CN**: 通过 `if` 语句引入条件控制流。
- **L869 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L869 CN**: 延续周围的声明、表达式或控制流结构。
- **L870 EN**: Executes statement `type->second, terminator, result, x, dim, nullptr, terminator);`.
  **L870 CN**: 执行语句 `type->second, terminator, result, x, dim, nullptr, terminator);`。
- **L871 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L871 CN**: 延续周围的声明、表达式或控制流结构。
- **L872 EN**: Executes statement involving `Crash`.
  **L872 CN**: 执行涉及 `Crash` 的语句。
- **L873 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L873 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L874 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L874 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L875 EN**: Blank line separates nearby declarations or logic blocks.
  **L875 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L876 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L876 CN**: 延续周围的声明、表达式或控制流结构。
- **L877 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L877 CN**: 延续周围的声明、表达式或控制流结构。
- **L878 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L878 CN**: 延续周围的声明、表达式或控制流结构。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 878 source lines, which suggests a substantial implementation unit. / 该文件约有 878 行源码，说明它是一个较大的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `flang-rt/runtime/reduction-templates.h`, `flang/Common/float128.h`, `flang/Runtime/character.h`, `flang/Runtime/reduction.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `flang-rt/runtime/reduction-templates.h`, `flang/Common/float128.h`, `flang/Runtime/character.h`, `flang/Runtime/reduction.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `NumericCompare`, `operator`, `CharacterCompare`, `ExtremumLocAccumulator`, `Reinitialize`, `GetResult`. / 值得关注的可调用实体包括 `NumericCompare`, `operator`, `CharacterCompare`, `ExtremumLocAccumulator`, `Reinitialize`, `GetResult`。
- **Core types / 核心类型**: Important declared or referenced types include `Type`, `CppType`, `Accumulator`, `DoPartialMaxOrMinLocHelper`, `NumericExtremumAccumulator`. / 重要的已声明或被引用类型包括 `Type`, `CppType`, `Accumulator`, `DoPartialMaxOrMinLocHelper`, `NumericExtremumAccumulator`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `flang-rt/runtime/reduction-templates.h`, `flang/Common/float128.h`, `flang/Runtime/character.h`, `flang/Runtime/reduction.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Standard or platform headers / 标准库或平台头文件**: `algorithm`, `cfloat`, `cinttypes`, `cmath`, `type_traits`. They connect the file to language-runtime or OS services. / 它们将该文件连接到语言运行时或操作系统服务。
- **Callable surface / 可调用表面**: Functions or methods defined here include `NumericCompare`, `operator`, `CharacterCompare`, `ExtremumLocAccumulator`, `Reinitialize`, `GetResult`, `AccumulateAt`, `NumericExtremumAccumulator`, `Accumulate`, `CharacterExtremumAccumulator`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `NumericCompare`, `operator`, `CharacterCompare`, `ExtremumLocAccumulator`, `Reinitialize`, `GetResult`, `AccumulateAt`, `NumericExtremumAccumulator`, `Accumulate`, `CharacterExtremumAccumulator`，它们通常是对周边代码暴露的主要入口。
- **Type coupling / 类型耦合**: Declared types such as `Type`, `CppType`, `Accumulator`, `DoPartialMaxOrMinLocHelper`, `NumericExtremumAccumulator` capture the data model shared with dependent code. / `Type`, `CppType`, `Accumulator`, `DoPartialMaxOrMinLocHelper`, `NumericExtremumAccumulator` 等声明类型体现了与依赖方共享的数据模型。
