# reduce.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `flang-rt/lib/runtime/reduce.cpp` | `flang-rt/lib/runtime/reduce.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements the Flang runtime library, including descriptors, I/O, memory management, numerics, and execution support. In this file, the main focus is `reduce`; the header comment highlights: REDUCE() implementation. | 实现 Flang 运行时库，包括描述符、I/O、内存管理、数值计算与执行支持。 本文件的核心主题是 `reduce`；文件头注释强调：REDUCE() implementation。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````cpp
//===-- lib/runtime/reduce.cpp ----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// REDUCE() implementation

#include "flang/Runtime/reduce.h"
#include "flang-rt/runtime/descriptor.h"
#include "flang-rt/runtime/reduction-templates.h"
#include "flang-rt/runtime/terminator.h"
#include "flang-rt/runtime/tools.h"

namespace Fortran::runtime {

template <typename T, bool isByValue> class ReduceAccumulator {
public:
  using Operation = std::conditional_t<isByValue, ValueReductionOperation<T>,
      ReferenceReductionOperation<T>>;
  RT_API_ATTRS ReduceAccumulator(const Descriptor &array, Operation operation,
      const T *identity, Terminator &terminator)
      : array_{array}, operation_{operation}, identity_{identity},
        terminator_{terminator} {}
  RT_API_ATTRS void Reinitialize() { result_.reset(); }
  template <typename A>
  RT_API_ATTRS bool AccumulateAt(const SubscriptValue at[]) {
    const auto *operand{array_.Element<A>(at)};
    if (result_) {
      if constexpr (isByValue) {
````

- **L1 EN**: Comment documents intent or context: `lib/runtime/reduce.cpp ----------------------------------*- C++ -*-===//`.
  **L1 CN**: 注释记录了意图或上下文：`lib/runtime/reduce.cpp ----------------------------------*- C++ -*-===//`。
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
- **L9 EN**: Comment documents intent or context: `REDUCE() implementation`.
  **L9 CN**: 注释记录了意图或上下文：`REDUCE() implementation`。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L11 EN**: Includes `flang/Runtime/reduce.h` to access Flang runtime declarations.
  **L11 CN**: 引入 `flang/Runtime/reduce.h` 以使用 Flang 运行时声明。
- **L12 EN**: Includes `flang-rt/runtime/descriptor.h` to access Flang runtime public headers.
  **L12 CN**: 引入 `flang-rt/runtime/descriptor.h` 以使用 Flang 运行时公共头文件。
- **L13 EN**: Includes `flang-rt/runtime/reduction-templates.h` to access Flang runtime public headers.
  **L13 CN**: 引入 `flang-rt/runtime/reduction-templates.h` 以使用 Flang 运行时公共头文件。
- **L14 EN**: Includes `flang-rt/runtime/terminator.h` to access Flang runtime public headers.
  **L14 CN**: 引入 `flang-rt/runtime/terminator.h` 以使用 Flang 运行时公共头文件。
- **L15 EN**: Includes `flang-rt/runtime/tools.h` to access Flang runtime public headers.
  **L15 CN**: 引入 `flang-rt/runtime/tools.h` 以使用 Flang 运行时公共头文件。
- **L16 EN**: Blank line separates nearby declarations or logic blocks.
  **L16 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L17 EN**: Enters namespace `Fortran` to scope related declarations.
  **L17 CN**: 进入命名空间 `Fortran` 以组织相关声明。
- **L18 EN**: Blank line separates nearby declarations or logic blocks.
  **L18 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L19 EN**: Begins a template declaration parameterizing subsequent code.
  **L19 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L20 EN**: Defines label or access section `public`.
  **L20 CN**: 定义标签或访问区段 `public`。
- **L21 EN**: Defines type alias `Operation` for readability or ABI convenience.
  **L21 CN**: 定义类型别名 `Operation`，以提升可读性或满足 ABI 便利性。
- **L22 EN**: Executes statement `ReferenceReductionOperation<T>>;`.
  **L22 CN**: 执行语句 `ReferenceReductionOperation<T>>;`。
- **L23 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L23 CN**: 延续周围的声明、表达式或控制流结构。
- **L24 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L24 CN**: 延续周围的声明、表达式或控制流结构。
- **L25 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L25 CN**: 延续周围的声明、表达式或控制流结构。
- **L26 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L26 CN**: 延续周围的声明、表达式或控制流结构。
- **L27 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L27 CN**: 延续周围的声明、表达式或控制流结构。
- **L28 EN**: Begins a template declaration parameterizing subsequent code.
  **L28 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L29 EN**: Declares or defines callable `AccumulateAt`.
  **L29 CN**: 声明或定义可调用实体 `AccumulateAt`。
- **L30 EN**: Executes statement `const auto *operand{array_.Element<A>(at)};`.
  **L30 CN**: 执行语句 `const auto *operand{array_.Element<A>(at)};`。
- **L31 EN**: Introduces conditional control flow with an `if` statement.
  **L31 CN**: 通过 `if` 语句引入条件控制流。
- **L32 EN**: Introduces conditional control flow with an `if` statement.
  **L32 CN**: 通过 `if` 语句引入条件控制流。

### Lines 33-64

````cpp
        result_ = operation_(*result_, *operand);
      } else {
        result_ = operation_(&*result_, operand);
      }
    } else {
      result_ = *operand;
    }
    return true;
  }
  template <typename A>
  RT_API_ATTRS void GetResult(A *to, int /*zeroBasedDim*/ = -1) {
    if (result_) {
      *to = *result_;
    } else if (identity_) {
      *to = *identity_;
    } else {
      terminator_.Crash("REDUCE() without IDENTITY= has no result");
    }
  }

private:
  const Descriptor &array_;
  common::optional<T> result_;
  Operation operation_;
  const T *identity_{nullptr};
  Terminator &terminator_;
};

template <typename T, typename OP, bool hasLength>
class BufferedReduceAccumulator {
public:
  RT_API_ATTRS BufferedReduceAccumulator(const Descriptor &array, OP operation,
````

- **L33 EN**: Initializes or updates `result_`.
  **L33 CN**: 初始化或更新 `result_`。
- **L34 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L34 CN**: 延续周围的声明、表达式或控制流结构。
- **L35 EN**: Initializes or updates `result_`.
  **L35 CN**: 初始化或更新 `result_`。
- **L36 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L36 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L37 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L37 CN**: 延续周围的声明、表达式或控制流结构。
- **L38 EN**: Initializes or updates `result_`.
  **L38 CN**: 初始化或更新 `result_`。
- **L39 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L39 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L40 EN**: Returns from the current function, often propagating a computed result.
  **L40 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L41 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L41 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L42 EN**: Begins a template declaration parameterizing subsequent code.
  **L42 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L43 EN**: Declares or defines callable `GetResult`.
  **L43 CN**: 声明或定义可调用实体 `GetResult`。
- **L44 EN**: Introduces conditional control flow with an `if` statement.
  **L44 CN**: 通过 `if` 语句引入条件控制流。
- **L45 EN**: Comment documents intent or context: `to = *result_;`.
  **L45 CN**: 注释记录了意图或上下文：`to = *result_;`。
- **L46 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L46 CN**: 延续周围的声明、表达式或控制流结构。
- **L47 EN**: Comment documents intent or context: `to = *identity_;`.
  **L47 CN**: 注释记录了意图或上下文：`to = *identity_;`。
- **L48 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L48 CN**: 延续周围的声明、表达式或控制流结构。
- **L49 EN**: Executes statement involving `Crash`.
  **L49 CN**: 执行涉及 `Crash` 的语句。
- **L50 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L50 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L51 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L51 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L52 EN**: Blank line separates nearby declarations or logic blocks.
  **L52 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L53 EN**: Defines label or access section `private`.
  **L53 CN**: 定义标签或访问区段 `private`。
- **L54 EN**: Executes statement `const Descriptor &array_;`.
  **L54 CN**: 执行语句 `const Descriptor &array_;`。
- **L55 EN**: Executes statement `common::optional<T> result_;`.
  **L55 CN**: 执行语句 `common::optional<T> result_;`。
- **L56 EN**: Executes statement `Operation operation_;`.
  **L56 CN**: 执行语句 `Operation operation_;`。
- **L57 EN**: Executes statement `const T *identity_{nullptr};`.
  **L57 CN**: 执行语句 `const T *identity_{nullptr};`。
- **L58 EN**: Executes statement `Terminator &terminator_;`.
  **L58 CN**: 执行语句 `Terminator &terminator_;`。
- **L59 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L59 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L60 EN**: Blank line separates nearby declarations or logic blocks.
  **L60 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L61 EN**: Begins a template declaration parameterizing subsequent code.
  **L61 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L62 EN**: Declares or defines class `BufferedReduceAccumulator`.
  **L62 CN**: 声明或定义 class `BufferedReduceAccumulator`。
- **L63 EN**: Defines label or access section `public`.
  **L63 CN**: 定义标签或访问区段 `public`。
- **L64 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L64 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 65-96

````cpp
      const T *identity, Terminator &terminator)
      : array_{array}, operation_{operation}, identity_{identity},
        terminator_{terminator} {}
  RT_API_ATTRS void Reinitialize() { activeTemp_ = -1; }
  template <typename A>
  RT_API_ATTRS bool AccumulateAt(const SubscriptValue at[]) {
    const auto *operand{array_.Element<A>(at)};
    if (activeTemp_ >= 0) {
      if constexpr (hasLength) {
        operation_(&*temp_[1 - activeTemp_], length_, &*temp_[activeTemp_],
            operand, length_, length_);
      } else {
        operation_(&*temp_[1 - activeTemp_], &*temp_[activeTemp_], operand);
      }
      activeTemp_ = 1 - activeTemp_;
    } else {
      activeTemp_ = 0;
      runtime::memcpy(&*temp_[activeTemp_], operand, elementBytes_);
    }
    return true;
  }
  template <typename A>
  RT_API_ATTRS void GetResult(A *to, int /*zeroBasedDim*/ = -1) {
    if (activeTemp_ >= 0) {
      runtime::memcpy(to, &*temp_[activeTemp_], elementBytes_);
    } else if (identity_) {
      runtime::memcpy(to, identity_, elementBytes_);
    } else {
      terminator_.Crash("REDUCE() without IDENTITY= has no result");
    }
  }

````

- **L65 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L65 CN**: 延续周围的声明、表达式或控制流结构。
- **L66 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L66 CN**: 延续周围的声明、表达式或控制流结构。
- **L67 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L67 CN**: 延续周围的声明、表达式或控制流结构。
- **L68 EN**: Initializes or updates `activeTemp_`.
  **L68 CN**: 初始化或更新 `activeTemp_`。
- **L69 EN**: Begins a template declaration parameterizing subsequent code.
  **L69 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L70 EN**: Declares or defines callable `AccumulateAt`.
  **L70 CN**: 声明或定义可调用实体 `AccumulateAt`。
- **L71 EN**: Executes statement `const auto *operand{array_.Element<A>(at)};`.
  **L71 CN**: 执行语句 `const auto *operand{array_.Element<A>(at)};`。
- **L72 EN**: Introduces conditional control flow with an `if` statement.
  **L72 CN**: 通过 `if` 语句引入条件控制流。
- **L73 EN**: Introduces conditional control flow with an `if` statement.
  **L73 CN**: 通过 `if` 语句引入条件控制流。
- **L74 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L74 CN**: 延续周围的声明、表达式或控制流结构。
- **L75 EN**: Executes statement `operand, length_, length_);`.
  **L75 CN**: 执行语句 `operand, length_, length_);`。
- **L76 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L76 CN**: 延续周围的声明、表达式或控制流结构。
- **L77 EN**: Executes statement involving `operation_`.
  **L77 CN**: 执行涉及 `operation_` 的语句。
- **L78 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L78 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L79 EN**: Initializes or updates `activeTemp_`.
  **L79 CN**: 初始化或更新 `activeTemp_`。
- **L80 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L80 CN**: 延续周围的声明、表达式或控制流结构。
- **L81 EN**: Initializes or updates `activeTemp_`.
  **L81 CN**: 初始化或更新 `activeTemp_`。
- **L82 EN**: Executes statement involving `memcpy`.
  **L82 CN**: 执行涉及 `memcpy` 的语句。
- **L83 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L83 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L84 EN**: Returns from the current function, often propagating a computed result.
  **L84 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L85 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L85 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L86 EN**: Begins a template declaration parameterizing subsequent code.
  **L86 CN**: 开始一个模板声明，为后续代码提供参数化能力。
- **L87 EN**: Declares or defines callable `GetResult`.
  **L87 CN**: 声明或定义可调用实体 `GetResult`。
- **L88 EN**: Introduces conditional control flow with an `if` statement.
  **L88 CN**: 通过 `if` 语句引入条件控制流。
- **L89 EN**: Executes statement involving `memcpy`.
  **L89 CN**: 执行涉及 `memcpy` 的语句。
- **L90 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L90 CN**: 延续周围的声明、表达式或控制流结构。
- **L91 EN**: Executes statement involving `memcpy`.
  **L91 CN**: 执行涉及 `memcpy` 的语句。
- **L92 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L92 CN**: 延续周围的声明、表达式或控制流结构。
- **L93 EN**: Executes statement involving `Crash`.
  **L93 CN**: 执行涉及 `Crash` 的语句。
- **L94 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L94 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L95 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L95 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L96 EN**: Blank line separates nearby declarations or logic blocks.
  **L96 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 97-128

````cpp
private:
  const Descriptor &array_;
  OP operation_;
  const T *identity_{nullptr};
  Terminator &terminator_;
  std::size_t elementBytes_{array_.ElementBytes()};
  OwningPtr<T> temp_[2]{SizedNew<T>{terminator_}(elementBytes_),
      SizedNew<T>{terminator_}(elementBytes_)};
  int activeTemp_{-1};
  std::size_t length_{elementBytes_ / sizeof(T)};
};

extern "C" {
RT_EXT_API_GROUP_BEGIN

std::int8_t RTDEF(ReduceInteger1Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 1>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int8_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::int8_t RTDEF(ReduceInteger1Value)(const Descriptor &array,
    ValueReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 1>(array, source, line, dim,
````

- **L97 EN**: Defines label or access section `private`.
  **L97 CN**: 定义标签或访问区段 `private`。
- **L98 EN**: Executes statement `const Descriptor &array_;`.
  **L98 CN**: 执行语句 `const Descriptor &array_;`。
- **L99 EN**: Executes statement `OP operation_;`.
  **L99 CN**: 执行语句 `OP operation_;`。
- **L100 EN**: Executes statement `const T *identity_{nullptr};`.
  **L100 CN**: 执行语句 `const T *identity_{nullptr};`。
- **L101 EN**: Executes statement `Terminator &terminator_;`.
  **L101 CN**: 执行语句 `Terminator &terminator_;`。
- **L102 EN**: Executes statement involving `ElementBytes`.
  **L102 CN**: 执行涉及 `ElementBytes` 的语句。
- **L103 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L103 CN**: 延续周围的声明、表达式或控制流结构。
- **L104 EN**: Executes statement `SizedNew<T>{terminator_}(elementBytes_)};`.
  **L104 CN**: 执行语句 `SizedNew<T>{terminator_}(elementBytes_)};`。
- **L105 EN**: Executes statement `int activeTemp_{-1};`.
  **L105 CN**: 执行语句 `int activeTemp_{-1};`。
- **L106 EN**: Executes statement involving `sizeof`.
  **L106 CN**: 执行涉及 `sizeof` 的语句。
- **L107 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L107 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L108 EN**: Blank line separates nearby declarations or logic blocks.
  **L108 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L109 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L109 CN**: 延续周围的声明、表达式或控制流结构。
- **L110 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L110 CN**: 延续周围的声明、表达式或控制流结构。
- **L111 EN**: Blank line separates nearby declarations or logic blocks.
  **L111 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L112 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L112 CN**: 延续周围的声明、表达式或控制流结构。
- **L113 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L113 CN**: 延续周围的声明、表达式或控制流结构。
- **L114 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L114 CN**: 延续周围的声明、表达式或控制流结构。
- **L115 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L115 CN**: 延续周围的声明、表达式或控制流结构。
- **L116 EN**: Executes statement `Terminator terminator{source, line};`.
  **L116 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L117 EN**: Returns from the current function, often propagating a computed result.
  **L117 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L118 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L118 CN**: 延续周围的声明、表达式或控制流结构。
- **L119 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L119 CN**: 延续周围的声明、表达式或控制流结构。
- **L120 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L120 CN**: 延续周围的声明、表达式或控制流结构。
- **L121 EN**: Executes statement `"REDUCE");`.
  **L121 CN**: 执行语句 `"REDUCE");`。
- **L122 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L122 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L123 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L123 CN**: 延续周围的声明、表达式或控制流结构。
- **L124 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L124 CN**: 延续周围的声明、表达式或控制流结构。
- **L125 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L125 CN**: 延续周围的声明、表达式或控制流结构。
- **L126 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L126 CN**: 延续周围的声明、表达式或控制流结构。
- **L127 EN**: Executes statement `Terminator terminator{source, line};`.
  **L127 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L128 EN**: Returns from the current function, often propagating a computed result.
  **L128 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 129-160

````cpp
      mask,
      ReduceAccumulator<std::int8_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceInteger1DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int8_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 1>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceInteger1DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int8_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 1>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::int16_t RTDEF(ReduceInteger2Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 2>(array, source, line, dim,
      mask,
````

- **L129 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L129 CN**: 延续周围的声明、表达式或控制流结构。
- **L130 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L130 CN**: 延续周围的声明、表达式或控制流结构。
- **L131 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L131 CN**: 延续周围的声明、表达式或控制流结构。
- **L132 EN**: Executes statement `"REDUCE");`.
  **L132 CN**: 执行语句 `"REDUCE");`。
- **L133 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L133 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L134 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L134 CN**: 延续周围的声明、表达式或控制流结构。
- **L135 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L135 CN**: 延续周围的声明、表达式或控制流结构。
- **L136 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L136 CN**: 延续周围的声明、表达式或控制流结构。
- **L137 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L137 CN**: 延续周围的声明、表达式或控制流结构。
- **L138 EN**: Executes statement `Terminator terminator{source, line};`.
  **L138 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L139 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L139 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L140 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L140 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L141 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L141 CN**: 延续周围的声明、表达式或控制流结构。
- **L142 EN**: Executes statement involving `ElementBytes`.
  **L142 CN**: 执行涉及 `ElementBytes` 的语句。
- **L143 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L143 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L144 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L144 CN**: 延续周围的声明、表达式或控制流结构。
- **L145 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L145 CN**: 延续周围的声明、表达式或控制流结构。
- **L146 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L146 CN**: 延续周围的声明、表达式或控制流结构。
- **L147 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L147 CN**: 延续周围的声明、表达式或控制流结构。
- **L148 EN**: Executes statement `Terminator terminator{source, line};`.
  **L148 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L149 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L149 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L150 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L150 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L151 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L151 CN**: 延续周围的声明、表达式或控制流结构。
- **L152 EN**: Executes statement involving `ElementBytes`.
  **L152 CN**: 执行涉及 `ElementBytes` 的语句。
- **L153 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L153 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L154 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L154 CN**: 延续周围的声明、表达式或控制流结构。
- **L155 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L155 CN**: 延续周围的声明、表达式或控制流结构。
- **L156 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L156 CN**: 延续周围的声明、表达式或控制流结构。
- **L157 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L157 CN**: 延续周围的声明、表达式或控制流结构。
- **L158 EN**: Executes statement `Terminator terminator{source, line};`.
  **L158 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L159 EN**: Returns from the current function, often propagating a computed result.
  **L159 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L160 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L160 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 161-192

````cpp
      ReduceAccumulator<std::int16_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::int16_t RTDEF(ReduceInteger2Value)(const Descriptor &array,
    ValueReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 2>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int16_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceInteger2DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int16_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 2>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceInteger2DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int16_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
````

- **L161 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L161 CN**: 延续周围的声明、表达式或控制流结构。
- **L162 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L162 CN**: 延续周围的声明、表达式或控制流结构。
- **L163 EN**: Executes statement `"REDUCE");`.
  **L163 CN**: 执行语句 `"REDUCE");`。
- **L164 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L164 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L165 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L165 CN**: 延续周围的声明、表达式或控制流结构。
- **L166 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L166 CN**: 延续周围的声明、表达式或控制流结构。
- **L167 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L167 CN**: 延续周围的声明、表达式或控制流结构。
- **L168 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L168 CN**: 延续周围的声明、表达式或控制流结构。
- **L169 EN**: Executes statement `Terminator terminator{source, line};`.
  **L169 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L170 EN**: Returns from the current function, often propagating a computed result.
  **L170 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L171 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L171 CN**: 延续周围的声明、表达式或控制流结构。
- **L172 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L172 CN**: 延续周围的声明、表达式或控制流结构。
- **L173 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L173 CN**: 延续周围的声明、表达式或控制流结构。
- **L174 EN**: Executes statement `"REDUCE");`.
  **L174 CN**: 执行语句 `"REDUCE");`。
- **L175 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L175 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L176 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L176 CN**: 延续周围的声明、表达式或控制流结构。
- **L177 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L177 CN**: 延续周围的声明、表达式或控制流结构。
- **L178 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L178 CN**: 延续周围的声明、表达式或控制流结构。
- **L179 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L179 CN**: 延续周围的声明、表达式或控制流结构。
- **L180 EN**: Executes statement `Terminator terminator{source, line};`.
  **L180 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L181 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L181 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L182 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L182 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L183 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L183 CN**: 延续周围的声明、表达式或控制流结构。
- **L184 EN**: Executes statement involving `ElementBytes`.
  **L184 CN**: 执行涉及 `ElementBytes` 的语句。
- **L185 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L185 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L186 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L186 CN**: 延续周围的声明、表达式或控制流结构。
- **L187 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L187 CN**: 延续周围的声明、表达式或控制流结构。
- **L188 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L188 CN**: 延续周围的声明、表达式或控制流结构。
- **L189 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L189 CN**: 延续周围的声明、表达式或控制流结构。
- **L190 EN**: Executes statement `Terminator terminator{source, line};`.
  **L190 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L191 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L191 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L192 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L192 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。

### Lines 193-224

````cpp
  PartialReduction<Accumulator, TypeCategory::Integer, 2>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::int32_t RTDEF(ReduceInteger4Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int32_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::int32_t RTDEF(ReduceInteger4Value)(const Descriptor &array,
    ValueReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int32_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceInteger4DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int32_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
````

- **L193 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L193 CN**: 延续周围的声明、表达式或控制流结构。
- **L194 EN**: Executes statement involving `ElementBytes`.
  **L194 CN**: 执行涉及 `ElementBytes` 的语句。
- **L195 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L195 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L196 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L196 CN**: 延续周围的声明、表达式或控制流结构。
- **L197 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L197 CN**: 延续周围的声明、表达式或控制流结构。
- **L198 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L198 CN**: 延续周围的声明、表达式或控制流结构。
- **L199 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L199 CN**: 延续周围的声明、表达式或控制流结构。
- **L200 EN**: Executes statement `Terminator terminator{source, line};`.
  **L200 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L201 EN**: Returns from the current function, often propagating a computed result.
  **L201 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L202 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L202 CN**: 延续周围的声明、表达式或控制流结构。
- **L203 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L203 CN**: 延续周围的声明、表达式或控制流结构。
- **L204 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L204 CN**: 延续周围的声明、表达式或控制流结构。
- **L205 EN**: Executes statement `"REDUCE");`.
  **L205 CN**: 执行语句 `"REDUCE");`。
- **L206 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L206 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L207 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L207 CN**: 延续周围的声明、表达式或控制流结构。
- **L208 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L208 CN**: 延续周围的声明、表达式或控制流结构。
- **L209 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L209 CN**: 延续周围的声明、表达式或控制流结构。
- **L210 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L210 CN**: 延续周围的声明、表达式或控制流结构。
- **L211 EN**: Executes statement `Terminator terminator{source, line};`.
  **L211 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L212 EN**: Returns from the current function, often propagating a computed result.
  **L212 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L213 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L213 CN**: 延续周围的声明、表达式或控制流结构。
- **L214 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L214 CN**: 延续周围的声明、表达式或控制流结构。
- **L215 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L215 CN**: 延续周围的声明、表达式或控制流结构。
- **L216 EN**: Executes statement `"REDUCE");`.
  **L216 CN**: 执行语句 `"REDUCE");`。
- **L217 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L217 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L218 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L218 CN**: 延续周围的声明、表达式或控制流结构。
- **L219 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L219 CN**: 延续周围的声明、表达式或控制流结构。
- **L220 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L220 CN**: 延续周围的声明、表达式或控制流结构。
- **L221 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L221 CN**: 延续周围的声明、表达式或控制流结构。
- **L222 EN**: Executes statement `Terminator terminator{source, line};`.
  **L222 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L223 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L223 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L224 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L224 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。

### Lines 225-256

````cpp
  PartialReduction<Accumulator, TypeCategory::Integer, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceInteger4DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int32_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::int64_t RTDEF(ReduceInteger8Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int64_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::int64_t RTDEF(ReduceInteger8Value)(const Descriptor &array,
    ValueReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::int64_t, true>{
````

- **L225 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L225 CN**: 延续周围的声明、表达式或控制流结构。
- **L226 EN**: Executes statement involving `ElementBytes`.
  **L226 CN**: 执行涉及 `ElementBytes` 的语句。
- **L227 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L227 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L228 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L228 CN**: 延续周围的声明、表达式或控制流结构。
- **L229 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L229 CN**: 延续周围的声明、表达式或控制流结构。
- **L230 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L230 CN**: 延续周围的声明、表达式或控制流结构。
- **L231 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L231 CN**: 延续周围的声明、表达式或控制流结构。
- **L232 EN**: Executes statement `Terminator terminator{source, line};`.
  **L232 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L233 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L233 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L234 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L234 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L235 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L235 CN**: 延续周围的声明、表达式或控制流结构。
- **L236 EN**: Executes statement involving `ElementBytes`.
  **L236 CN**: 执行涉及 `ElementBytes` 的语句。
- **L237 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L237 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L238 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L238 CN**: 延续周围的声明、表达式或控制流结构。
- **L239 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L239 CN**: 延续周围的声明、表达式或控制流结构。
- **L240 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L240 CN**: 延续周围的声明、表达式或控制流结构。
- **L241 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L241 CN**: 延续周围的声明、表达式或控制流结构。
- **L242 EN**: Executes statement `Terminator terminator{source, line};`.
  **L242 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L243 EN**: Returns from the current function, often propagating a computed result.
  **L243 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L244 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L244 CN**: 延续周围的声明、表达式或控制流结构。
- **L245 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L245 CN**: 延续周围的声明、表达式或控制流结构。
- **L246 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L246 CN**: 延续周围的声明、表达式或控制流结构。
- **L247 EN**: Executes statement `"REDUCE");`.
  **L247 CN**: 执行语句 `"REDUCE");`。
- **L248 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L248 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L249 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L249 CN**: 延续周围的声明、表达式或控制流结构。
- **L250 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L250 CN**: 延续周围的声明、表达式或控制流结构。
- **L251 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L251 CN**: 延续周围的声明、表达式或控制流结构。
- **L252 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L252 CN**: 延续周围的声明、表达式或控制流结构。
- **L253 EN**: Executes statement `Terminator terminator{source, line};`.
  **L253 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L254 EN**: Returns from the current function, often propagating a computed result.
  **L254 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L255 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L255 CN**: 延续周围的声明、表达式或控制流结构。
- **L256 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L256 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 257-288

````cpp
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceInteger8DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int64_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceInteger8DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::int64_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#ifdef __SIZEOF_INT128__
common::int128_t RTDEF(ReduceInteger16Ref)(const Descriptor &array,
    ReferenceReductionOperation<common::int128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const common::int128_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<common::int128_t, false>{
````

- **L257 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L257 CN**: 延续周围的声明、表达式或控制流结构。
- **L258 EN**: Executes statement `"REDUCE");`.
  **L258 CN**: 执行语句 `"REDUCE");`。
- **L259 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L259 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L260 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L260 CN**: 延续周围的声明、表达式或控制流结构。
- **L261 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L261 CN**: 延续周围的声明、表达式或控制流结构。
- **L262 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L262 CN**: 延续周围的声明、表达式或控制流结构。
- **L263 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L263 CN**: 延续周围的声明、表达式或控制流结构。
- **L264 EN**: Executes statement `Terminator terminator{source, line};`.
  **L264 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L265 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L265 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L266 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L266 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L267 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L267 CN**: 延续周围的声明、表达式或控制流结构。
- **L268 EN**: Executes statement involving `ElementBytes`.
  **L268 CN**: 执行涉及 `ElementBytes` 的语句。
- **L269 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L269 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L270 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L270 CN**: 延续周围的声明、表达式或控制流结构。
- **L271 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L271 CN**: 延续周围的声明、表达式或控制流结构。
- **L272 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L272 CN**: 延续周围的声明、表达式或控制流结构。
- **L273 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L273 CN**: 延续周围的声明、表达式或控制流结构。
- **L274 EN**: Executes statement `Terminator terminator{source, line};`.
  **L274 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L275 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L275 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L276 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L276 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L277 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L277 CN**: 延续周围的声明、表达式或控制流结构。
- **L278 EN**: Executes statement involving `ElementBytes`.
  **L278 CN**: 执行涉及 `ElementBytes` 的语句。
- **L279 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L279 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L280 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L280 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L281 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L281 CN**: 延续周围的声明、表达式或控制流结构。
- **L282 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L282 CN**: 延续周围的声明、表达式或控制流结构。
- **L283 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L283 CN**: 延续周围的声明、表达式或控制流结构。
- **L284 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L284 CN**: 延续周围的声明、表达式或控制流结构。
- **L285 EN**: Executes statement `Terminator terminator{source, line};`.
  **L285 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L286 EN**: Returns from the current function, often propagating a computed result.
  **L286 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L287 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L287 CN**: 延续周围的声明、表达式或控制流结构。
- **L288 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L288 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 289-320

````cpp
          array, operation, identity, terminator},
      "REDUCE");
}
common::int128_t RTDEF(ReduceInteger16Value)(const Descriptor &array,
    ValueReductionOperation<common::int128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const common::int128_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Integer, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<common::int128_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceInteger16DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<common::int128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const common::int128_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<common::int128_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceInteger16DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<common::int128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const common::int128_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<common::int128_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Integer, 16>(result, array,
````

- **L289 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L289 CN**: 延续周围的声明、表达式或控制流结构。
- **L290 EN**: Executes statement `"REDUCE");`.
  **L290 CN**: 执行语句 `"REDUCE");`。
- **L291 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L291 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L292 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L292 CN**: 延续周围的声明、表达式或控制流结构。
- **L293 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L293 CN**: 延续周围的声明、表达式或控制流结构。
- **L294 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L294 CN**: 延续周围的声明、表达式或控制流结构。
- **L295 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L295 CN**: 延续周围的声明、表达式或控制流结构。
- **L296 EN**: Executes statement `Terminator terminator{source, line};`.
  **L296 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L297 EN**: Returns from the current function, often propagating a computed result.
  **L297 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L298 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L298 CN**: 延续周围的声明、表达式或控制流结构。
- **L299 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L299 CN**: 延续周围的声明、表达式或控制流结构。
- **L300 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L300 CN**: 延续周围的声明、表达式或控制流结构。
- **L301 EN**: Executes statement `"REDUCE");`.
  **L301 CN**: 执行语句 `"REDUCE");`。
- **L302 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L302 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L303 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L303 CN**: 延续周围的声明、表达式或控制流结构。
- **L304 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L304 CN**: 延续周围的声明、表达式或控制流结构。
- **L305 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L305 CN**: 延续周围的声明、表达式或控制流结构。
- **L306 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L306 CN**: 延续周围的声明、表达式或控制流结构。
- **L307 EN**: Executes statement `Terminator terminator{source, line};`.
  **L307 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L308 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L308 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L309 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L309 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L310 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L310 CN**: 延续周围的声明、表达式或控制流结构。
- **L311 EN**: Executes statement involving `ElementBytes`.
  **L311 CN**: 执行涉及 `ElementBytes` 的语句。
- **L312 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L312 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L313 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L313 CN**: 延续周围的声明、表达式或控制流结构。
- **L314 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L314 CN**: 延续周围的声明、表达式或控制流结构。
- **L315 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L315 CN**: 延续周围的声明、表达式或控制流结构。
- **L316 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L316 CN**: 延续周围的声明、表达式或控制流结构。
- **L317 EN**: Executes statement `Terminator terminator{source, line};`.
  **L317 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L318 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L318 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L319 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L319 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L320 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L320 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 321-352

````cpp
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif

std::uint8_t RTDEF(ReduceUnsigned1Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::uint8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 1>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint8_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::uint8_t RTDEF(ReduceUnsigned1Value)(const Descriptor &array,
    ValueReductionOperation<std::uint8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 1>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint8_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceUnsigned1DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::uint8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint8_t, false>;
````

- **L321 EN**: Executes statement involving `ElementBytes`.
  **L321 CN**: 执行涉及 `ElementBytes` 的语句。
- **L322 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L322 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L323 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L323 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L324 EN**: Blank line separates nearby declarations or logic blocks.
  **L324 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L325 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L325 CN**: 延续周围的声明、表达式或控制流结构。
- **L326 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L326 CN**: 延续周围的声明、表达式或控制流结构。
- **L327 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L327 CN**: 延续周围的声明、表达式或控制流结构。
- **L328 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L328 CN**: 延续周围的声明、表达式或控制流结构。
- **L329 EN**: Executes statement `Terminator terminator{source, line};`.
  **L329 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L330 EN**: Returns from the current function, often propagating a computed result.
  **L330 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L331 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L331 CN**: 延续周围的声明、表达式或控制流结构。
- **L332 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L332 CN**: 延续周围的声明、表达式或控制流结构。
- **L333 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L333 CN**: 延续周围的声明、表达式或控制流结构。
- **L334 EN**: Executes statement `"REDUCE");`.
  **L334 CN**: 执行语句 `"REDUCE");`。
- **L335 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L335 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L336 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L336 CN**: 延续周围的声明、表达式或控制流结构。
- **L337 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L337 CN**: 延续周围的声明、表达式或控制流结构。
- **L338 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L338 CN**: 延续周围的声明、表达式或控制流结构。
- **L339 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L339 CN**: 延续周围的声明、表达式或控制流结构。
- **L340 EN**: Executes statement `Terminator terminator{source, line};`.
  **L340 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L341 EN**: Returns from the current function, often propagating a computed result.
  **L341 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L342 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L342 CN**: 延续周围的声明、表达式或控制流结构。
- **L343 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L343 CN**: 延续周围的声明、表达式或控制流结构。
- **L344 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L344 CN**: 延续周围的声明、表达式或控制流结构。
- **L345 EN**: Executes statement `"REDUCE");`.
  **L345 CN**: 执行语句 `"REDUCE");`。
- **L346 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L346 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L347 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L347 CN**: 延续周围的声明、表达式或控制流结构。
- **L348 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L348 CN**: 延续周围的声明、表达式或控制流结构。
- **L349 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L349 CN**: 延续周围的声明、表达式或控制流结构。
- **L350 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L350 CN**: 延续周围的声明、表达式或控制流结构。
- **L351 EN**: Executes statement `Terminator terminator{source, line};`.
  **L351 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L352 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L352 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。

### Lines 353-384

````cpp
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 1>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceUnsigned1DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::uint8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint8_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint8_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 1>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::uint16_t RTDEF(ReduceUnsigned2Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::uint16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 2>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint16_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::uint16_t RTDEF(ReduceUnsigned2Value)(const Descriptor &array,
    ValueReductionOperation<std::uint16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 2>(array, source, line, dim,
      mask,
````

- **L353 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L353 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L354 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L354 CN**: 延续周围的声明、表达式或控制流结构。
- **L355 EN**: Executes statement involving `ElementBytes`.
  **L355 CN**: 执行涉及 `ElementBytes` 的语句。
- **L356 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L356 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L357 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L357 CN**: 延续周围的声明、表达式或控制流结构。
- **L358 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L358 CN**: 延续周围的声明、表达式或控制流结构。
- **L359 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L359 CN**: 延续周围的声明、表达式或控制流结构。
- **L360 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L360 CN**: 延续周围的声明、表达式或控制流结构。
- **L361 EN**: Executes statement `Terminator terminator{source, line};`.
  **L361 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L362 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L362 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L363 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L363 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L364 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L364 CN**: 延续周围的声明、表达式或控制流结构。
- **L365 EN**: Executes statement involving `ElementBytes`.
  **L365 CN**: 执行涉及 `ElementBytes` 的语句。
- **L366 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L366 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L367 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L367 CN**: 延续周围的声明、表达式或控制流结构。
- **L368 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L368 CN**: 延续周围的声明、表达式或控制流结构。
- **L369 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L369 CN**: 延续周围的声明、表达式或控制流结构。
- **L370 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L370 CN**: 延续周围的声明、表达式或控制流结构。
- **L371 EN**: Executes statement `Terminator terminator{source, line};`.
  **L371 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L372 EN**: Returns from the current function, often propagating a computed result.
  **L372 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L373 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L373 CN**: 延续周围的声明、表达式或控制流结构。
- **L374 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L374 CN**: 延续周围的声明、表达式或控制流结构。
- **L375 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L375 CN**: 延续周围的声明、表达式或控制流结构。
- **L376 EN**: Executes statement `"REDUCE");`.
  **L376 CN**: 执行语句 `"REDUCE");`。
- **L377 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L377 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L378 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L378 CN**: 延续周围的声明、表达式或控制流结构。
- **L379 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L379 CN**: 延续周围的声明、表达式或控制流结构。
- **L380 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L380 CN**: 延续周围的声明、表达式或控制流结构。
- **L381 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L381 CN**: 延续周围的声明、表达式或控制流结构。
- **L382 EN**: Executes statement `Terminator terminator{source, line};`.
  **L382 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L383 EN**: Returns from the current function, often propagating a computed result.
  **L383 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L384 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L384 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 385-416

````cpp
      ReduceAccumulator<std::uint16_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceUnsigned2DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::uint16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint16_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 2>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceUnsigned2DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::uint16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint16_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint16_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 2>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::uint32_t RTDEF(ReduceUnsigned4Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::uint32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint32_t, false>{
````

- **L385 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L385 CN**: 延续周围的声明、表达式或控制流结构。
- **L386 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L386 CN**: 延续周围的声明、表达式或控制流结构。
- **L387 EN**: Executes statement `"REDUCE");`.
  **L387 CN**: 执行语句 `"REDUCE");`。
- **L388 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L388 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L389 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L389 CN**: 延续周围的声明、表达式或控制流结构。
- **L390 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L390 CN**: 延续周围的声明、表达式或控制流结构。
- **L391 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L391 CN**: 延续周围的声明、表达式或控制流结构。
- **L392 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L392 CN**: 延续周围的声明、表达式或控制流结构。
- **L393 EN**: Executes statement `Terminator terminator{source, line};`.
  **L393 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L394 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L394 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L395 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L395 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L396 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L396 CN**: 延续周围的声明、表达式或控制流结构。
- **L397 EN**: Executes statement involving `ElementBytes`.
  **L397 CN**: 执行涉及 `ElementBytes` 的语句。
- **L398 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L398 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L399 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L399 CN**: 延续周围的声明、表达式或控制流结构。
- **L400 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L400 CN**: 延续周围的声明、表达式或控制流结构。
- **L401 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L401 CN**: 延续周围的声明、表达式或控制流结构。
- **L402 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L402 CN**: 延续周围的声明、表达式或控制流结构。
- **L403 EN**: Executes statement `Terminator terminator{source, line};`.
  **L403 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L404 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L404 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L405 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L405 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L406 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L406 CN**: 延续周围的声明、表达式或控制流结构。
- **L407 EN**: Executes statement involving `ElementBytes`.
  **L407 CN**: 执行涉及 `ElementBytes` 的语句。
- **L408 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L408 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L409 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L409 CN**: 延续周围的声明、表达式或控制流结构。
- **L410 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L410 CN**: 延续周围的声明、表达式或控制流结构。
- **L411 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L411 CN**: 延续周围的声明、表达式或控制流结构。
- **L412 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L412 CN**: 延续周围的声明、表达式或控制流结构。
- **L413 EN**: Executes statement `Terminator terminator{source, line};`.
  **L413 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L414 EN**: Returns from the current function, often propagating a computed result.
  **L414 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L415 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L415 CN**: 延续周围的声明、表达式或控制流结构。
- **L416 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L416 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 417-448

````cpp
          array, operation, identity, terminator},
      "REDUCE");
}
std::uint32_t RTDEF(ReduceUnsigned4Value)(const Descriptor &array,
    ValueReductionOperation<std::uint32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint32_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceUnsigned4DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::uint32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint32_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceUnsigned4DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::uint32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint32_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint32_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 4>(result, array,
````

- **L417 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L417 CN**: 延续周围的声明、表达式或控制流结构。
- **L418 EN**: Executes statement `"REDUCE");`.
  **L418 CN**: 执行语句 `"REDUCE");`。
- **L419 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L419 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L420 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L420 CN**: 延续周围的声明、表达式或控制流结构。
- **L421 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L421 CN**: 延续周围的声明、表达式或控制流结构。
- **L422 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L422 CN**: 延续周围的声明、表达式或控制流结构。
- **L423 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L423 CN**: 延续周围的声明、表达式或控制流结构。
- **L424 EN**: Executes statement `Terminator terminator{source, line};`.
  **L424 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L425 EN**: Returns from the current function, often propagating a computed result.
  **L425 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L426 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L426 CN**: 延续周围的声明、表达式或控制流结构。
- **L427 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L427 CN**: 延续周围的声明、表达式或控制流结构。
- **L428 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L428 CN**: 延续周围的声明、表达式或控制流结构。
- **L429 EN**: Executes statement `"REDUCE");`.
  **L429 CN**: 执行语句 `"REDUCE");`。
- **L430 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L430 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L431 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L431 CN**: 延续周围的声明、表达式或控制流结构。
- **L432 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L432 CN**: 延续周围的声明、表达式或控制流结构。
- **L433 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L433 CN**: 延续周围的声明、表达式或控制流结构。
- **L434 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L434 CN**: 延续周围的声明、表达式或控制流结构。
- **L435 EN**: Executes statement `Terminator terminator{source, line};`.
  **L435 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L436 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L436 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L437 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L437 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L438 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L438 CN**: 延续周围的声明、表达式或控制流结构。
- **L439 EN**: Executes statement involving `ElementBytes`.
  **L439 CN**: 执行涉及 `ElementBytes` 的语句。
- **L440 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L440 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L441 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L441 CN**: 延续周围的声明、表达式或控制流结构。
- **L442 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L442 CN**: 延续周围的声明、表达式或控制流结构。
- **L443 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L443 CN**: 延续周围的声明、表达式或控制流结构。
- **L444 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L444 CN**: 延续周围的声明、表达式或控制流结构。
- **L445 EN**: Executes statement `Terminator terminator{source, line};`.
  **L445 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L446 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L446 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L447 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L447 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L448 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L448 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 449-480

````cpp
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
std::uint64_t RTDEF(ReduceUnsigned8Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::uint64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint64_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
std::uint64_t RTDEF(ReduceUnsigned8Value)(const Descriptor &array,
    ValueReductionOperation<std::uint64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<std::uint64_t, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceUnsigned8DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::uint64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint64_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 8>(result, array,
````

- **L449 EN**: Executes statement involving `ElementBytes`.
  **L449 CN**: 执行涉及 `ElementBytes` 的语句。
- **L450 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L450 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L451 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L451 CN**: 延续周围的声明、表达式或控制流结构。
- **L452 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L452 CN**: 延续周围的声明、表达式或控制流结构。
- **L453 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L453 CN**: 延续周围的声明、表达式或控制流结构。
- **L454 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L454 CN**: 延续周围的声明、表达式或控制流结构。
- **L455 EN**: Executes statement `Terminator terminator{source, line};`.
  **L455 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L456 EN**: Returns from the current function, often propagating a computed result.
  **L456 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L457 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L457 CN**: 延续周围的声明、表达式或控制流结构。
- **L458 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L458 CN**: 延续周围的声明、表达式或控制流结构。
- **L459 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L459 CN**: 延续周围的声明、表达式或控制流结构。
- **L460 EN**: Executes statement `"REDUCE");`.
  **L460 CN**: 执行语句 `"REDUCE");`。
- **L461 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L461 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L462 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L462 CN**: 延续周围的声明、表达式或控制流结构。
- **L463 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L463 CN**: 延续周围的声明、表达式或控制流结构。
- **L464 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L464 CN**: 延续周围的声明、表达式或控制流结构。
- **L465 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L465 CN**: 延续周围的声明、表达式或控制流结构。
- **L466 EN**: Executes statement `Terminator terminator{source, line};`.
  **L466 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L467 EN**: Returns from the current function, often propagating a computed result.
  **L467 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L468 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L468 CN**: 延续周围的声明、表达式或控制流结构。
- **L469 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L469 CN**: 延续周围的声明、表达式或控制流结构。
- **L470 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L470 CN**: 延续周围的声明、表达式或控制流结构。
- **L471 EN**: Executes statement `"REDUCE");`.
  **L471 CN**: 执行语句 `"REDUCE");`。
- **L472 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L472 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L473 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L473 CN**: 延续周围的声明、表达式或控制流结构。
- **L474 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L474 CN**: 延续周围的声明、表达式或控制流结构。
- **L475 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L475 CN**: 延续周围的声明、表达式或控制流结构。
- **L476 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L476 CN**: 延续周围的声明、表达式或控制流结构。
- **L477 EN**: Executes statement `Terminator terminator{source, line};`.
  **L477 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L478 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L478 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L479 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L479 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L480 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L480 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 481-512

````cpp
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceUnsigned8DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::uint64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::uint64_t *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<std::uint64_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#ifdef __SIZEOF_INT128__
common::uint128_t RTDEF(ReduceUnsigned16Ref)(const Descriptor &array,
    ReferenceReductionOperation<common::uint128_t> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const common::uint128_t *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<common::uint128_t, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
common::uint128_t RTDEF(ReduceUnsigned16Value)(const Descriptor &array,
    ValueReductionOperation<common::uint128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask,
    const common::uint128_t *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Unsigned, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<common::uint128_t, true>{
````

- **L481 EN**: Executes statement involving `ElementBytes`.
  **L481 CN**: 执行涉及 `ElementBytes` 的语句。
- **L482 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L482 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L483 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L483 CN**: 延续周围的声明、表达式或控制流结构。
- **L484 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L484 CN**: 延续周围的声明、表达式或控制流结构。
- **L485 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L485 CN**: 延续周围的声明、表达式或控制流结构。
- **L486 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L486 CN**: 延续周围的声明、表达式或控制流结构。
- **L487 EN**: Executes statement `Terminator terminator{source, line};`.
  **L487 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L488 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L488 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L489 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L489 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L490 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L490 CN**: 延续周围的声明、表达式或控制流结构。
- **L491 EN**: Executes statement involving `ElementBytes`.
  **L491 CN**: 执行涉及 `ElementBytes` 的语句。
- **L492 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L492 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L493 EN**: Preprocessor directive manages conditional compilation or macros: `#ifdef __SIZEOF_INT128__`.
  **L493 CN**: 预处理指令管理条件编译或宏：`#ifdef __SIZEOF_INT128__`。
- **L494 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L494 CN**: 延续周围的声明、表达式或控制流结构。
- **L495 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L495 CN**: 延续周围的声明、表达式或控制流结构。
- **L496 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L496 CN**: 延续周围的声明、表达式或控制流结构。
- **L497 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L497 CN**: 延续周围的声明、表达式或控制流结构。
- **L498 EN**: Executes statement `Terminator terminator{source, line};`.
  **L498 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L499 EN**: Returns from the current function, often propagating a computed result.
  **L499 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L500 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L500 CN**: 延续周围的声明、表达式或控制流结构。
- **L501 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L501 CN**: 延续周围的声明、表达式或控制流结构。
- **L502 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L502 CN**: 延续周围的声明、表达式或控制流结构。
- **L503 EN**: Executes statement `"REDUCE");`.
  **L503 CN**: 执行语句 `"REDUCE");`。
- **L504 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L504 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L505 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L505 CN**: 延续周围的声明、表达式或控制流结构。
- **L506 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L506 CN**: 延续周围的声明、表达式或控制流结构。
- **L507 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L507 CN**: 延续周围的声明、表达式或控制流结构。
- **L508 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L508 CN**: 延续周围的声明、表达式或控制流结构。
- **L509 EN**: Executes statement `Terminator terminator{source, line};`.
  **L509 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L510 EN**: Returns from the current function, often propagating a computed result.
  **L510 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L511 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L511 CN**: 延续周围的声明、表达式或控制流结构。
- **L512 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L512 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 513-544

````cpp
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceUnsigned16DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<common::uint128_t> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const common::uint128_t *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<common::uint128_t, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceUnsigned16DimValue)(Descriptor &result,
    const Descriptor &array,
    ValueReductionOperation<common::uint128_t> operation, const char *source,
    int line, int dim, const Descriptor *mask,
    const common::uint128_t *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<common::uint128_t, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Unsigned, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif

// TODO: real/complex(2 & 3)
float RTDEF(ReduceReal4Ref)(const Descriptor &array,
    ReferenceReductionOperation<float> operation, const char *source, int line,
    int dim, const Descriptor *mask, const float *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 4>(array, source, line, dim,
````

- **L513 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L513 CN**: 延续周围的声明、表达式或控制流结构。
- **L514 EN**: Executes statement `"REDUCE");`.
  **L514 CN**: 执行语句 `"REDUCE");`。
- **L515 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L515 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L516 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L516 CN**: 延续周围的声明、表达式或控制流结构。
- **L517 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L517 CN**: 延续周围的声明、表达式或控制流结构。
- **L518 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L518 CN**: 延续周围的声明、表达式或控制流结构。
- **L519 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L519 CN**: 延续周围的声明、表达式或控制流结构。
- **L520 EN**: Executes statement `Terminator terminator{source, line};`.
  **L520 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L521 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L521 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L522 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L522 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L523 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L523 CN**: 延续周围的声明、表达式或控制流结构。
- **L524 EN**: Executes statement involving `ElementBytes`.
  **L524 CN**: 执行涉及 `ElementBytes` 的语句。
- **L525 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L525 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L526 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L526 CN**: 延续周围的声明、表达式或控制流结构。
- **L527 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L527 CN**: 延续周围的声明、表达式或控制流结构。
- **L528 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L528 CN**: 延续周围的声明、表达式或控制流结构。
- **L529 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L529 CN**: 延续周围的声明、表达式或控制流结构。
- **L530 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L530 CN**: 延续周围的声明、表达式或控制流结构。
- **L531 EN**: Executes statement `Terminator terminator{source, line};`.
  **L531 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L532 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L532 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L533 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L533 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L534 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L534 CN**: 延续周围的声明、表达式或控制流结构。
- **L535 EN**: Executes statement involving `ElementBytes`.
  **L535 CN**: 执行涉及 `ElementBytes` 的语句。
- **L536 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L536 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L537 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L537 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L538 EN**: Blank line separates nearby declarations or logic blocks.
  **L538 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L539 EN**: Comment documents intent or context: `TODO: real/complex(2 & 3)`.
  **L539 CN**: 注释记录了意图或上下文：`TODO: real/complex(2 & 3)`。
- **L540 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L540 CN**: 延续周围的声明、表达式或控制流结构。
- **L541 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L541 CN**: 延续周围的声明、表达式或控制流结构。
- **L542 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L542 CN**: 延续周围的声明、表达式或控制流结构。
- **L543 EN**: Executes statement `Terminator terminator{source, line};`.
  **L543 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L544 EN**: Returns from the current function, often propagating a computed result.
  **L544 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 545-576

````cpp
      mask,
      ReduceAccumulator<float, false>{array, operation, identity, terminator},
      "REDUCE");
}
float RTDEF(ReduceReal4Value)(const Descriptor &array,
    ValueReductionOperation<float> operation, const char *source, int line,
    int dim, const Descriptor *mask, const float *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<float, true>{array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceReal4DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<float> operation, const char *source, int line,
    int dim, const Descriptor *mask, const float *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<float, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceReal4DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<float> operation, const char *source, int line,
    int dim, const Descriptor *mask, const float *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<float, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
double RTDEF(ReduceReal8Ref)(const Descriptor &array,
````

- **L545 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L545 CN**: 延续周围的声明、表达式或控制流结构。
- **L546 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L546 CN**: 延续周围的声明、表达式或控制流结构。
- **L547 EN**: Executes statement `"REDUCE");`.
  **L547 CN**: 执行语句 `"REDUCE");`。
- **L548 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L548 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L549 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L549 CN**: 延续周围的声明、表达式或控制流结构。
- **L550 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L550 CN**: 延续周围的声明、表达式或控制流结构。
- **L551 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L551 CN**: 延续周围的声明、表达式或控制流结构。
- **L552 EN**: Executes statement `Terminator terminator{source, line};`.
  **L552 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L553 EN**: Returns from the current function, often propagating a computed result.
  **L553 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L554 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L554 CN**: 延续周围的声明、表达式或控制流结构。
- **L555 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L555 CN**: 延续周围的声明、表达式或控制流结构。
- **L556 EN**: Executes statement `"REDUCE");`.
  **L556 CN**: 执行语句 `"REDUCE");`。
- **L557 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L557 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L558 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L558 CN**: 延续周围的声明、表达式或控制流结构。
- **L559 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L559 CN**: 延续周围的声明、表达式或控制流结构。
- **L560 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L560 CN**: 延续周围的声明、表达式或控制流结构。
- **L561 EN**: Executes statement `Terminator terminator{source, line};`.
  **L561 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L562 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L562 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L563 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L563 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L564 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L564 CN**: 延续周围的声明、表达式或控制流结构。
- **L565 EN**: Executes statement involving `ElementBytes`.
  **L565 CN**: 执行涉及 `ElementBytes` 的语句。
- **L566 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L566 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L567 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L567 CN**: 延续周围的声明、表达式或控制流结构。
- **L568 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L568 CN**: 延续周围的声明、表达式或控制流结构。
- **L569 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L569 CN**: 延续周围的声明、表达式或控制流结构。
- **L570 EN**: Executes statement `Terminator terminator{source, line};`.
  **L570 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L571 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L571 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L572 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L572 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L573 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L573 CN**: 延续周围的声明、表达式或控制流结构。
- **L574 EN**: Executes statement involving `ElementBytes`.
  **L574 CN**: 执行涉及 `ElementBytes` 的语句。
- **L575 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L575 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L576 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L576 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 577-608

````cpp
    ReferenceReductionOperation<double> operation, const char *source, int line,
    int dim, const Descriptor *mask, const double *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<double, false>{array, operation, identity, terminator},
      "REDUCE");
}
double RTDEF(ReduceReal8Value)(const Descriptor &array,
    ValueReductionOperation<double> operation, const char *source, int line,
    int dim, const Descriptor *mask, const double *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<double, true>{array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceReal8DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<double> operation, const char *source, int line,
    int dim, const Descriptor *mask, const double *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<double, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceReal8DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<double> operation, const char *source, int line,
    int dim, const Descriptor *mask, const double *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<double, true>;
  Accumulator accumulator{array, operation, identity, terminator};
````

- **L577 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L577 CN**: 延续周围的声明、表达式或控制流结构。
- **L578 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L578 CN**: 延续周围的声明、表达式或控制流结构。
- **L579 EN**: Executes statement `Terminator terminator{source, line};`.
  **L579 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L580 EN**: Returns from the current function, often propagating a computed result.
  **L580 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L581 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L581 CN**: 延续周围的声明、表达式或控制流结构。
- **L582 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L582 CN**: 延续周围的声明、表达式或控制流结构。
- **L583 EN**: Executes statement `"REDUCE");`.
  **L583 CN**: 执行语句 `"REDUCE");`。
- **L584 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L584 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L585 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L585 CN**: 延续周围的声明、表达式或控制流结构。
- **L586 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L586 CN**: 延续周围的声明、表达式或控制流结构。
- **L587 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L587 CN**: 延续周围的声明、表达式或控制流结构。
- **L588 EN**: Executes statement `Terminator terminator{source, line};`.
  **L588 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L589 EN**: Returns from the current function, often propagating a computed result.
  **L589 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L590 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L590 CN**: 延续周围的声明、表达式或控制流结构。
- **L591 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L591 CN**: 延续周围的声明、表达式或控制流结构。
- **L592 EN**: Executes statement `"REDUCE");`.
  **L592 CN**: 执行语句 `"REDUCE");`。
- **L593 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L593 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L594 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L594 CN**: 延续周围的声明、表达式或控制流结构。
- **L595 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L595 CN**: 延续周围的声明、表达式或控制流结构。
- **L596 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L596 CN**: 延续周围的声明、表达式或控制流结构。
- **L597 EN**: Executes statement `Terminator terminator{source, line};`.
  **L597 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L598 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L598 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L599 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L599 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L600 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L600 CN**: 延续周围的声明、表达式或控制流结构。
- **L601 EN**: Executes statement involving `ElementBytes`.
  **L601 CN**: 执行涉及 `ElementBytes` 的语句。
- **L602 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L602 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L603 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L603 CN**: 延续周围的声明、表达式或控制流结构。
- **L604 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L604 CN**: 延续周围的声明、表达式或控制流结构。
- **L605 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L605 CN**: 延续周围的声明、表达式或控制流结构。
- **L606 EN**: Executes statement `Terminator terminator{source, line};`.
  **L606 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L607 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L607 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L608 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L608 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。

### Lines 609-640

````cpp
  PartialReduction<Accumulator, TypeCategory::Real, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#if HAS_FLOAT80
CppTypeFor<TypeCategory::Real, 10> RTDEF(ReduceReal10Ref)(
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Real, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Real, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 10>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
CppTypeFor<TypeCategory::Real, 10> RTDEF(ReduceReal10Value)(
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Real, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Real, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 10>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceReal10DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Real, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Real, 10> *identity, bool ordered) {
````

- **L609 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L609 CN**: 延续周围的声明、表达式或控制流结构。
- **L610 EN**: Executes statement involving `ElementBytes`.
  **L610 CN**: 执行涉及 `ElementBytes` 的语句。
- **L611 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L611 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L612 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L612 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L613 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L613 CN**: 延续周围的声明、表达式或控制流结构。
- **L614 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L614 CN**: 延续周围的声明、表达式或控制流结构。
- **L615 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L615 CN**: 延续周围的声明、表达式或控制流结构。
- **L616 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L616 CN**: 延续周围的声明、表达式或控制流结构。
- **L617 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L617 CN**: 延续周围的声明、表达式或控制流结构。
- **L618 EN**: Executes statement `Terminator terminator{source, line};`.
  **L618 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L619 EN**: Returns from the current function, often propagating a computed result.
  **L619 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L620 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L620 CN**: 延续周围的声明、表达式或控制流结构。
- **L621 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L621 CN**: 延续周围的声明、表达式或控制流结构。
- **L622 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L622 CN**: 延续周围的声明、表达式或控制流结构。
- **L623 EN**: Executes statement `"REDUCE");`.
  **L623 CN**: 执行语句 `"REDUCE");`。
- **L624 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L624 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L625 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L625 CN**: 延续周围的声明、表达式或控制流结构。
- **L626 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L626 CN**: 延续周围的声明、表达式或控制流结构。
- **L627 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L627 CN**: 延续周围的声明、表达式或控制流结构。
- **L628 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L628 CN**: 延续周围的声明、表达式或控制流结构。
- **L629 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L629 CN**: 延续周围的声明、表达式或控制流结构。
- **L630 EN**: Executes statement `Terminator terminator{source, line};`.
  **L630 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L631 EN**: Returns from the current function, often propagating a computed result.
  **L631 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L632 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L632 CN**: 延续周围的声明、表达式或控制流结构。
- **L633 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L633 CN**: 延续周围的声明、表达式或控制流结构。
- **L634 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L634 CN**: 延续周围的声明、表达式或控制流结构。
- **L635 EN**: Executes statement `"REDUCE");`.
  **L635 CN**: 执行语句 `"REDUCE");`。
- **L636 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L636 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L637 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L637 CN**: 延续周围的声明、表达式或控制流结构。
- **L638 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L638 CN**: 延续周围的声明、表达式或控制流结构。
- **L639 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L639 CN**: 延续周围的声明、表达式或控制流结构。
- **L640 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L640 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 641-672

````cpp
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 10>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceReal10DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Real, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Real, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 10>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
CppFloat128Type RTDEF(ReduceReal16Ref)(const Descriptor &array,
    ReferenceReductionOperation<CppFloat128Type> operation, const char *source,
    int line, int dim, const Descriptor *mask, const CppFloat128Type *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppFloat128Type, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
CppFloat128Type RTDEF(ReduceReal16Value)(const Descriptor &array,
````

- **L641 EN**: Executes statement `Terminator terminator{source, line};`.
  **L641 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L642 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L642 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L643 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, false>;`.
  **L643 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, false>;`。
- **L644 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L644 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L645 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L645 CN**: 延续周围的声明、表达式或控制流结构。
- **L646 EN**: Executes statement involving `ElementBytes`.
  **L646 CN**: 执行涉及 `ElementBytes` 的语句。
- **L647 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L647 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L648 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L648 CN**: 延续周围的声明、表达式或控制流结构。
- **L649 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L649 CN**: 延续周围的声明、表达式或控制流结构。
- **L650 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L650 CN**: 延续周围的声明、表达式或控制流结构。
- **L651 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L651 CN**: 延续周围的声明、表达式或控制流结构。
- **L652 EN**: Executes statement `Terminator terminator{source, line};`.
  **L652 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L653 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L653 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L654 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, true>;`.
  **L654 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Real, 10>, true>;`。
- **L655 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L655 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L656 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L656 CN**: 延续周围的声明、表达式或控制流结构。
- **L657 EN**: Executes statement involving `ElementBytes`.
  **L657 CN**: 执行涉及 `ElementBytes` 的语句。
- **L658 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L658 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L659 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L659 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L660 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L660 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L661 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L661 CN**: 延续周围的声明、表达式或控制流结构。
- **L662 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L662 CN**: 延续周围的声明、表达式或控制流结构。
- **L663 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L663 CN**: 延续周围的声明、表达式或控制流结构。
- **L664 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L664 CN**: 延续周围的声明、表达式或控制流结构。
- **L665 EN**: Executes statement `Terminator terminator{source, line};`.
  **L665 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L666 EN**: Returns from the current function, often propagating a computed result.
  **L666 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L667 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L667 CN**: 延续周围的声明、表达式或控制流结构。
- **L668 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L668 CN**: 延续周围的声明、表达式或控制流结构。
- **L669 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L669 CN**: 延续周围的声明、表达式或控制流结构。
- **L670 EN**: Executes statement `"REDUCE");`.
  **L670 CN**: 执行语句 `"REDUCE");`。
- **L671 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L671 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L672 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L672 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 673-704

````cpp
    ValueReductionOperation<CppFloat128Type> operation, const char *source,
    int line, int dim, const Descriptor *mask, const CppFloat128Type *identity,
    bool ordered) {
  Terminator terminator{source, line};
  return GetTotalReduction<TypeCategory::Real, 16>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppFloat128Type, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(ReduceReal16DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<CppFloat128Type> operation, const char *source,
    int line, int dim, const Descriptor *mask, const CppFloat128Type *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<CppFloat128Type, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceReal16DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<CppFloat128Type> operation, const char *source,
    int line, int dim, const Descriptor *mask, const CppFloat128Type *identity,
    bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = ReduceAccumulator<CppFloat128Type, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Real, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif

````

- **L673 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L673 CN**: 延续周围的声明、表达式或控制流结构。
- **L674 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L674 CN**: 延续周围的声明、表达式或控制流结构。
- **L675 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L675 CN**: 延续周围的声明、表达式或控制流结构。
- **L676 EN**: Executes statement `Terminator terminator{source, line};`.
  **L676 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L677 EN**: Returns from the current function, often propagating a computed result.
  **L677 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L678 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L678 CN**: 延续周围的声明、表达式或控制流结构。
- **L679 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L679 CN**: 延续周围的声明、表达式或控制流结构。
- **L680 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L680 CN**: 延续周围的声明、表达式或控制流结构。
- **L681 EN**: Executes statement `"REDUCE");`.
  **L681 CN**: 执行语句 `"REDUCE");`。
- **L682 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L682 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L683 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L683 CN**: 延续周围的声明、表达式或控制流结构。
- **L684 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L684 CN**: 延续周围的声明、表达式或控制流结构。
- **L685 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L685 CN**: 延续周围的声明、表达式或控制流结构。
- **L686 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L686 CN**: 延续周围的声明、表达式或控制流结构。
- **L687 EN**: Executes statement `Terminator terminator{source, line};`.
  **L687 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L688 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L688 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L689 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L689 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L690 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L690 CN**: 延续周围的声明、表达式或控制流结构。
- **L691 EN**: Executes statement involving `ElementBytes`.
  **L691 CN**: 执行涉及 `ElementBytes` 的语句。
- **L692 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L692 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L693 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L693 CN**: 延续周围的声明、表达式或控制流结构。
- **L694 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L694 CN**: 延续周围的声明、表达式或控制流结构。
- **L695 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L695 CN**: 延续周围的声明、表达式或控制流结构。
- **L696 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L696 CN**: 延续周围的声明、表达式或控制流结构。
- **L697 EN**: Executes statement `Terminator terminator{source, line};`.
  **L697 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L698 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L698 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L699 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L699 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L700 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L700 CN**: 延续周围的声明、表达式或控制流结构。
- **L701 EN**: Executes statement involving `ElementBytes`.
  **L701 CN**: 执行涉及 `ElementBytes` 的语句。
- **L702 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L702 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L703 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L703 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L704 EN**: Blank line separates nearby declarations or logic blocks.
  **L704 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 705-736

````cpp
void RTDEF(CppReduceComplex4Ref)(CppTypeFor<TypeCategory::Complex, 4> &result,
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 4>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 4> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex4Value)(CppTypeFor<TypeCategory::Complex, 4> &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 4>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 4> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 4>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex4DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 4>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 4> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, false>;
  Accumulator accumulator{array, operation, identity, terminator};
````

- **L705 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L705 CN**: 延续周围的声明、表达式或控制流结构。
- **L706 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L706 CN**: 延续周围的声明、表达式或控制流结构。
- **L707 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L707 CN**: 延续周围的声明、表达式或控制流结构。
- **L708 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L708 CN**: 延续周围的声明、表达式或控制流结构。
- **L709 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L709 CN**: 延续周围的声明、表达式或控制流结构。
- **L710 EN**: Executes statement `Terminator terminator{source, line};`.
  **L710 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L711 EN**: Initializes or updates `result`.
  **L711 CN**: 初始化或更新 `result`。
- **L712 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L712 CN**: 延续周围的声明、表达式或控制流结构。
- **L713 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L713 CN**: 延续周围的声明、表达式或控制流结构。
- **L714 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L714 CN**: 延续周围的声明、表达式或控制流结构。
- **L715 EN**: Executes statement `"REDUCE");`.
  **L715 CN**: 执行语句 `"REDUCE");`。
- **L716 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L716 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L717 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L717 CN**: 延续周围的声明、表达式或控制流结构。
- **L718 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L718 CN**: 延续周围的声明、表达式或控制流结构。
- **L719 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L719 CN**: 延续周围的声明、表达式或控制流结构。
- **L720 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L720 CN**: 延续周围的声明、表达式或控制流结构。
- **L721 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L721 CN**: 延续周围的声明、表达式或控制流结构。
- **L722 EN**: Executes statement `Terminator terminator{source, line};`.
  **L722 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L723 EN**: Initializes or updates `result`.
  **L723 CN**: 初始化或更新 `result`。
- **L724 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L724 CN**: 延续周围的声明、表达式或控制流结构。
- **L725 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L725 CN**: 延续周围的声明、表达式或控制流结构。
- **L726 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L726 CN**: 延续周围的声明、表达式或控制流结构。
- **L727 EN**: Executes statement `"REDUCE");`.
  **L727 CN**: 执行语句 `"REDUCE");`。
- **L728 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L728 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L729 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L729 CN**: 延续周围的声明、表达式或控制流结构。
- **L730 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L730 CN**: 延续周围的声明、表达式或控制流结构。
- **L731 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L731 CN**: 延续周围的声明、表达式或控制流结构。
- **L732 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L732 CN**: 延续周围的声明、表达式或控制流结构。
- **L733 EN**: Executes statement `Terminator terminator{source, line};`.
  **L733 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L734 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L734 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L735 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, false>;`.
  **L735 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, false>;`。
- **L736 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L736 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。

### Lines 737-768

````cpp
  PartialReduction<Accumulator, TypeCategory::Complex, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(CppReduceComplex4DimValue)(Descriptor &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 4>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 4> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(CppReduceComplex8Ref)(CppTypeFor<TypeCategory::Complex, 8> &result,
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 8>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 8> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex8Value)(CppTypeFor<TypeCategory::Complex, 8> &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 8>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 8> *identity, bool ordered) {
````

- **L737 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L737 CN**: 延续周围的声明、表达式或控制流结构。
- **L738 EN**: Executes statement involving `ElementBytes`.
  **L738 CN**: 执行涉及 `ElementBytes` 的语句。
- **L739 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L739 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L740 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L740 CN**: 延续周围的声明、表达式或控制流结构。
- **L741 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L741 CN**: 延续周围的声明、表达式或控制流结构。
- **L742 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L742 CN**: 延续周围的声明、表达式或控制流结构。
- **L743 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L743 CN**: 延续周围的声明、表达式或控制流结构。
- **L744 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L744 CN**: 延续周围的声明、表达式或控制流结构。
- **L745 EN**: Executes statement `Terminator terminator{source, line};`.
  **L745 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L746 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L746 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L747 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, true>;`.
  **L747 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 4>, true>;`。
- **L748 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L748 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L749 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L749 CN**: 延续周围的声明、表达式或控制流结构。
- **L750 EN**: Executes statement involving `ElementBytes`.
  **L750 CN**: 执行涉及 `ElementBytes` 的语句。
- **L751 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L751 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L752 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L752 CN**: 延续周围的声明、表达式或控制流结构。
- **L753 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L753 CN**: 延续周围的声明、表达式或控制流结构。
- **L754 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L754 CN**: 延续周围的声明、表达式或控制流结构。
- **L755 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L755 CN**: 延续周围的声明、表达式或控制流结构。
- **L756 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L756 CN**: 延续周围的声明、表达式或控制流结构。
- **L757 EN**: Executes statement `Terminator terminator{source, line};`.
  **L757 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L758 EN**: Initializes or updates `result`.
  **L758 CN**: 初始化或更新 `result`。
- **L759 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L759 CN**: 延续周围的声明、表达式或控制流结构。
- **L760 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L760 CN**: 延续周围的声明、表达式或控制流结构。
- **L761 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L761 CN**: 延续周围的声明、表达式或控制流结构。
- **L762 EN**: Executes statement `"REDUCE");`.
  **L762 CN**: 执行语句 `"REDUCE");`。
- **L763 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L763 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L764 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L764 CN**: 延续周围的声明、表达式或控制流结构。
- **L765 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L765 CN**: 延续周围的声明、表达式或控制流结构。
- **L766 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L766 CN**: 延续周围的声明、表达式或控制流结构。
- **L767 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L767 CN**: 延续周围的声明、表达式或控制流结构。
- **L768 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L768 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 769-800

````cpp
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 8>(array, source, line, dim,
      mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex8DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 8>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 8> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(CppReduceComplex8DimValue)(Descriptor &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 8>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 8> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 8>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#if HAS_FLOAT80
void RTDEF(CppReduceComplex10Ref)(CppTypeFor<TypeCategory::Complex, 10> &result,
````

- **L769 EN**: Executes statement `Terminator terminator{source, line};`.
  **L769 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L770 EN**: Initializes or updates `result`.
  **L770 CN**: 初始化或更新 `result`。
- **L771 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L771 CN**: 延续周围的声明、表达式或控制流结构。
- **L772 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L772 CN**: 延续周围的声明、表达式或控制流结构。
- **L773 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L773 CN**: 延续周围的声明、表达式或控制流结构。
- **L774 EN**: Executes statement `"REDUCE");`.
  **L774 CN**: 执行语句 `"REDUCE");`。
- **L775 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L775 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L776 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L776 CN**: 延续周围的声明、表达式或控制流结构。
- **L777 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L777 CN**: 延续周围的声明、表达式或控制流结构。
- **L778 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L778 CN**: 延续周围的声明、表达式或控制流结构。
- **L779 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L779 CN**: 延续周围的声明、表达式或控制流结构。
- **L780 EN**: Executes statement `Terminator terminator{source, line};`.
  **L780 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L781 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L781 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L782 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, false>;`.
  **L782 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, false>;`。
- **L783 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L783 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L784 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L784 CN**: 延续周围的声明、表达式或控制流结构。
- **L785 EN**: Executes statement involving `ElementBytes`.
  **L785 CN**: 执行涉及 `ElementBytes` 的语句。
- **L786 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L786 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L787 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L787 CN**: 延续周围的声明、表达式或控制流结构。
- **L788 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L788 CN**: 延续周围的声明、表达式或控制流结构。
- **L789 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L789 CN**: 延续周围的声明、表达式或控制流结构。
- **L790 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L790 CN**: 延续周围的声明、表达式或控制流结构。
- **L791 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L791 CN**: 延续周围的声明、表达式或控制流结构。
- **L792 EN**: Executes statement `Terminator terminator{source, line};`.
  **L792 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L793 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L793 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L794 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, true>;`.
  **L794 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 8>, true>;`。
- **L795 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L795 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L796 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L796 CN**: 延续周围的声明、表达式或控制流结构。
- **L797 EN**: Executes statement involving `ElementBytes`.
  **L797 CN**: 执行涉及 `ElementBytes` 的语句。
- **L798 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L798 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L799 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_FLOAT80`.
  **L799 CN**: 预处理指令管理条件编译或宏：`#if HAS_FLOAT80`。
- **L800 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L800 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 801-832

````cpp
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 10>>
        operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 10>(array, source, line,
      dim, mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex10Value)(
    CppTypeFor<TypeCategory::Complex, 10> &result, const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 10>(array, source, line,
      dim, mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex10DimRef)(Descriptor &result,
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 10>>
        operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
````

- **L801 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L801 CN**: 延续周围的声明、表达式或控制流结构。
- **L802 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L802 CN**: 延续周围的声明、表达式或控制流结构。
- **L803 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L803 CN**: 延续周围的声明、表达式或控制流结构。
- **L804 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L804 CN**: 延续周围的声明、表达式或控制流结构。
- **L805 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L805 CN**: 延续周围的声明、表达式或控制流结构。
- **L806 EN**: Executes statement `Terminator terminator{source, line};`.
  **L806 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L807 EN**: Initializes or updates `result`.
  **L807 CN**: 初始化或更新 `result`。
- **L808 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L808 CN**: 延续周围的声明、表达式或控制流结构。
- **L809 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L809 CN**: 延续周围的声明、表达式或控制流结构。
- **L810 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L810 CN**: 延续周围的声明、表达式或控制流结构。
- **L811 EN**: Executes statement `"REDUCE");`.
  **L811 CN**: 执行语句 `"REDUCE");`。
- **L812 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L812 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L813 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L813 CN**: 延续周围的声明、表达式或控制流结构。
- **L814 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L814 CN**: 延续周围的声明、表达式或控制流结构。
- **L815 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L815 CN**: 延续周围的声明、表达式或控制流结构。
- **L816 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L816 CN**: 延续周围的声明、表达式或控制流结构。
- **L817 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L817 CN**: 延续周围的声明、表达式或控制流结构。
- **L818 EN**: Executes statement `Terminator terminator{source, line};`.
  **L818 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L819 EN**: Initializes or updates `result`.
  **L819 CN**: 初始化或更新 `result`。
- **L820 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L820 CN**: 延续周围的声明、表达式或控制流结构。
- **L821 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L821 CN**: 延续周围的声明、表达式或控制流结构。
- **L822 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L822 CN**: 延续周围的声明、表达式或控制流结构。
- **L823 EN**: Executes statement `"REDUCE");`.
  **L823 CN**: 执行语句 `"REDUCE");`。
- **L824 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L824 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L825 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L825 CN**: 延续周围的声明、表达式或控制流结构。
- **L826 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L826 CN**: 延续周围的声明、表达式或控制流结构。
- **L827 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L827 CN**: 延续周围的声明、表达式或控制流结构。
- **L828 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L828 CN**: 延续周围的声明、表达式或控制流结构。
- **L829 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L829 CN**: 延续周围的声明、表达式或控制流结构。
- **L830 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L830 CN**: 延续周围的声明、表达式或控制流结构。
- **L831 EN**: Executes statement `Terminator terminator{source, line};`.
  **L831 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L832 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L832 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。

### Lines 833-864

````cpp
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 10>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(CppReduceComplex10DimValue)(Descriptor &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 10>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 10> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 10>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif
#if HAS_LDBL128 || HAS_FLOAT128
void RTDEF(CppReduceComplex16Ref)(CppTypeFor<TypeCategory::Complex, 16> &result,
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 16>>
        operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 16> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 16>(array, source, line,
      dim, mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, false>{
          array, operation, identity, terminator},
      "REDUCE");
}
````

- **L833 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, false>;`.
  **L833 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, false>;`。
- **L834 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L834 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L835 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L835 CN**: 延续周围的声明、表达式或控制流结构。
- **L836 EN**: Executes statement involving `ElementBytes`.
  **L836 CN**: 执行涉及 `ElementBytes` 的语句。
- **L837 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L837 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L838 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L838 CN**: 延续周围的声明、表达式或控制流结构。
- **L839 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L839 CN**: 延续周围的声明、表达式或控制流结构。
- **L840 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L840 CN**: 延续周围的声明、表达式或控制流结构。
- **L841 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L841 CN**: 延续周围的声明、表达式或控制流结构。
- **L842 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L842 CN**: 延续周围的声明、表达式或控制流结构。
- **L843 EN**: Executes statement `Terminator terminator{source, line};`.
  **L843 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L844 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L844 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L845 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, true>;`.
  **L845 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 10>, true>;`。
- **L846 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L846 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L847 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L847 CN**: 延续周围的声明、表达式或控制流结构。
- **L848 EN**: Executes statement involving `ElementBytes`.
  **L848 CN**: 执行涉及 `ElementBytes` 的语句。
- **L849 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L849 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L850 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L850 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L851 EN**: Preprocessor directive manages conditional compilation or macros: `#if HAS_LDBL128 || HAS_FLOAT128`.
  **L851 CN**: 预处理指令管理条件编译或宏：`#if HAS_LDBL128 || HAS_FLOAT128`。
- **L852 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L852 CN**: 延续周围的声明、表达式或控制流结构。
- **L853 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L853 CN**: 延续周围的声明、表达式或控制流结构。
- **L854 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L854 CN**: 延续周围的声明、表达式或控制流结构。
- **L855 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L855 CN**: 延续周围的声明、表达式或控制流结构。
- **L856 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L856 CN**: 延续周围的声明、表达式或控制流结构。
- **L857 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L857 CN**: 延续周围的声明、表达式或控制流结构。
- **L858 EN**: Executes statement `Terminator terminator{source, line};`.
  **L858 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L859 EN**: Initializes or updates `result`.
  **L859 CN**: 初始化或更新 `result`。
- **L860 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L860 CN**: 延续周围的声明、表达式或控制流结构。
- **L861 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L861 CN**: 延续周围的声明、表达式或控制流结构。
- **L862 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L862 CN**: 延续周围的声明、表达式或控制流结构。
- **L863 EN**: Executes statement `"REDUCE");`.
  **L863 CN**: 执行语句 `"REDUCE");`。
- **L864 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L864 CN**: 打开或关闭一个作用域、聚合体或声明块。

### Lines 865-896

````cpp
void RTDEF(CppReduceComplex16Value)(
    CppTypeFor<TypeCategory::Complex, 16> &result, const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 16>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 16> *identity, bool ordered) {
  Terminator terminator{source, line};
  result = GetTotalReduction<TypeCategory::Complex, 16>(array, source, line,
      dim, mask,
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, true>{
          array, operation, identity, terminator},
      "REDUCE");
}
void RTDEF(CppReduceComplex16DimRef)(Descriptor &result,
    const Descriptor &array,
    ReferenceReductionOperation<CppTypeFor<TypeCategory::Complex, 16>>
        operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 16> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(CppReduceComplex16DimValue)(Descriptor &result,
    const Descriptor &array,
    ValueReductionOperation<CppTypeFor<TypeCategory::Complex, 16>> operation,
    const char *source, int line, int dim, const Descriptor *mask,
    const CppTypeFor<TypeCategory::Complex, 16> *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator =
````

- **L865 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L865 CN**: 延续周围的声明、表达式或控制流结构。
- **L866 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L866 CN**: 延续周围的声明、表达式或控制流结构。
- **L867 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L867 CN**: 延续周围的声明、表达式或控制流结构。
- **L868 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L868 CN**: 延续周围的声明、表达式或控制流结构。
- **L869 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L869 CN**: 延续周围的声明、表达式或控制流结构。
- **L870 EN**: Executes statement `Terminator terminator{source, line};`.
  **L870 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L871 EN**: Initializes or updates `result`.
  **L871 CN**: 初始化或更新 `result`。
- **L872 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L872 CN**: 延续周围的声明、表达式或控制流结构。
- **L873 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L873 CN**: 延续周围的声明、表达式或控制流结构。
- **L874 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L874 CN**: 延续周围的声明、表达式或控制流结构。
- **L875 EN**: Executes statement `"REDUCE");`.
  **L875 CN**: 执行语句 `"REDUCE");`。
- **L876 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L876 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L877 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L877 CN**: 延续周围的声明、表达式或控制流结构。
- **L878 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L878 CN**: 延续周围的声明、表达式或控制流结构。
- **L879 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L879 CN**: 延续周围的声明、表达式或控制流结构。
- **L880 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L880 CN**: 延续周围的声明、表达式或控制流结构。
- **L881 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L881 CN**: 延续周围的声明、表达式或控制流结构。
- **L882 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L882 CN**: 延续周围的声明、表达式或控制流结构。
- **L883 EN**: Executes statement `Terminator terminator{source, line};`.
  **L883 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L884 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L884 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L885 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, false>;`.
  **L885 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, false>;`。
- **L886 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L886 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L887 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L887 CN**: 延续周围的声明、表达式或控制流结构。
- **L888 EN**: Executes statement involving `ElementBytes`.
  **L888 CN**: 执行涉及 `ElementBytes` 的语句。
- **L889 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L889 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L890 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L890 CN**: 延续周围的声明、表达式或控制流结构。
- **L891 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L891 CN**: 延续周围的声明、表达式或控制流结构。
- **L892 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L892 CN**: 延续周围的声明、表达式或控制流结构。
- **L893 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L893 CN**: 延续周围的声明、表达式或控制流结构。
- **L894 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L894 CN**: 延续周围的声明、表达式或控制流结构。
- **L895 EN**: Executes statement `Terminator terminator{source, line};`.
  **L895 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L896 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L896 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。

### Lines 897-928

````cpp
      ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Complex, 16>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
#endif

bool RTDEF(ReduceLogical1Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger1Ref)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
bool RTDEF(ReduceLogical1Value)(const Descriptor &array,
    ValueReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger1Value)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
void RTDEF(ReduceLogical1DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger1DimRef)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
void RTDEF(ReduceLogical1DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int8_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int8_t *identity,
    bool ordered) {
````

- **L897 EN**: Executes statement `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, true>;`.
  **L897 CN**: 执行语句 `ReduceAccumulator<CppTypeFor<TypeCategory::Complex, 16>, true>;`。
- **L898 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L898 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L899 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L899 CN**: 延续周围的声明、表达式或控制流结构。
- **L900 EN**: Executes statement involving `ElementBytes`.
  **L900 CN**: 执行涉及 `ElementBytes` 的语句。
- **L901 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L901 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L902 EN**: Preprocessor directive manages conditional compilation or macros: `#endif`.
  **L902 CN**: 预处理指令管理条件编译或宏：`#endif`。
- **L903 EN**: Blank line separates nearby declarations or logic blocks.
  **L903 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L904 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L904 CN**: 延续周围的声明、表达式或控制流结构。
- **L905 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L905 CN**: 延续周围的声明、表达式或控制流结构。
- **L906 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L906 CN**: 延续周围的声明、表达式或控制流结构。
- **L907 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L907 CN**: 延续周围的声明、表达式或控制流结构。
- **L908 EN**: Returns from the current function, often propagating a computed result.
  **L908 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L909 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L909 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L910 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L910 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L911 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L911 CN**: 延续周围的声明、表达式或控制流结构。
- **L912 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L912 CN**: 延续周围的声明、表达式或控制流结构。
- **L913 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L913 CN**: 延续周围的声明、表达式或控制流结构。
- **L914 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L914 CN**: 延续周围的声明、表达式或控制流结构。
- **L915 EN**: Returns from the current function, often propagating a computed result.
  **L915 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L916 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L916 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L917 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L917 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L918 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L918 CN**: 延续周围的声明、表达式或控制流结构。
- **L919 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L919 CN**: 延续周围的声明、表达式或控制流结构。
- **L920 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L920 CN**: 延续周围的声明、表达式或控制流结构。
- **L921 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L921 CN**: 延续周围的声明、表达式或控制流结构。
- **L922 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L922 CN**: 延续周围的声明、表达式或控制流结构。
- **L923 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L923 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L924 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L924 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L925 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L925 CN**: 延续周围的声明、表达式或控制流结构。
- **L926 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L926 CN**: 延续周围的声明、表达式或控制流结构。
- **L927 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L927 CN**: 延续周围的声明、表达式或控制流结构。
- **L928 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L928 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 929-960

````cpp
  RTNAME(ReduceInteger1DimValue)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
bool RTDEF(ReduceLogical2Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger2Ref)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
bool RTDEF(ReduceLogical2Value)(const Descriptor &array,
    ValueReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger2Value)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
void RTDEF(ReduceLogical2DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger2DimRef)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
void RTDEF(ReduceLogical2DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int16_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int16_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger2DimValue)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
bool RTDEF(ReduceLogical4Ref)(const Descriptor &array,
````

- **L929 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L929 CN**: 延续周围的声明、表达式或控制流结构。
- **L930 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L930 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L931 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L931 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L932 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L932 CN**: 延续周围的声明、表达式或控制流结构。
- **L933 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L933 CN**: 延续周围的声明、表达式或控制流结构。
- **L934 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L934 CN**: 延续周围的声明、表达式或控制流结构。
- **L935 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L935 CN**: 延续周围的声明、表达式或控制流结构。
- **L936 EN**: Returns from the current function, often propagating a computed result.
  **L936 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L937 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L937 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L938 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L938 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L939 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L939 CN**: 延续周围的声明、表达式或控制流结构。
- **L940 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L940 CN**: 延续周围的声明、表达式或控制流结构。
- **L941 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L941 CN**: 延续周围的声明、表达式或控制流结构。
- **L942 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L942 CN**: 延续周围的声明、表达式或控制流结构。
- **L943 EN**: Returns from the current function, often propagating a computed result.
  **L943 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L944 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L944 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L945 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L945 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L946 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L946 CN**: 延续周围的声明、表达式或控制流结构。
- **L947 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L947 CN**: 延续周围的声明、表达式或控制流结构。
- **L948 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L948 CN**: 延续周围的声明、表达式或控制流结构。
- **L949 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L949 CN**: 延续周围的声明、表达式或控制流结构。
- **L950 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L950 CN**: 延续周围的声明、表达式或控制流结构。
- **L951 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L951 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L952 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L952 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L953 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L953 CN**: 延续周围的声明、表达式或控制流结构。
- **L954 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L954 CN**: 延续周围的声明、表达式或控制流结构。
- **L955 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L955 CN**: 延续周围的声明、表达式或控制流结构。
- **L956 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L956 CN**: 延续周围的声明、表达式或控制流结构。
- **L957 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L957 CN**: 延续周围的声明、表达式或控制流结构。
- **L958 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L958 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L959 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L959 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L960 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L960 CN**: 延续周围的声明、表达式或控制流结构。

### Lines 961-992

````cpp
    ReferenceReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger4Ref)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
bool RTDEF(ReduceLogical4Value)(const Descriptor &array,
    ValueReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger4Value)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
void RTDEF(ReduceLogical4DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger4DimRef)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
void RTDEF(ReduceLogical4DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int32_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int32_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger4DimValue)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
bool RTDEF(ReduceLogical8Ref)(const Descriptor &array,
    ReferenceReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger8Ref)(
````

- **L961 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L961 CN**: 延续周围的声明、表达式或控制流结构。
- **L962 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L962 CN**: 延续周围的声明、表达式或控制流结构。
- **L963 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L963 CN**: 延续周围的声明、表达式或控制流结构。
- **L964 EN**: Returns from the current function, often propagating a computed result.
  **L964 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L965 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L965 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L966 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L966 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L967 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L967 CN**: 延续周围的声明、表达式或控制流结构。
- **L968 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L968 CN**: 延续周围的声明、表达式或控制流结构。
- **L969 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L969 CN**: 延续周围的声明、表达式或控制流结构。
- **L970 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L970 CN**: 延续周围的声明、表达式或控制流结构。
- **L971 EN**: Returns from the current function, often propagating a computed result.
  **L971 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L972 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L972 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L973 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L973 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L974 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L974 CN**: 延续周围的声明、表达式或控制流结构。
- **L975 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L975 CN**: 延续周围的声明、表达式或控制流结构。
- **L976 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L976 CN**: 延续周围的声明、表达式或控制流结构。
- **L977 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L977 CN**: 延续周围的声明、表达式或控制流结构。
- **L978 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L978 CN**: 延续周围的声明、表达式或控制流结构。
- **L979 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L979 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L980 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L980 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L981 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L981 CN**: 延续周围的声明、表达式或控制流结构。
- **L982 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L982 CN**: 延续周围的声明、表达式或控制流结构。
- **L983 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L983 CN**: 延续周围的声明、表达式或控制流结构。
- **L984 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L984 CN**: 延续周围的声明、表达式或控制流结构。
- **L985 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L985 CN**: 延续周围的声明、表达式或控制流结构。
- **L986 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L986 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L987 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L987 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L988 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L988 CN**: 延续周围的声明、表达式或控制流结构。
- **L989 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L989 CN**: 延续周围的声明、表达式或控制流结构。
- **L990 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L990 CN**: 延续周围的声明、表达式或控制流结构。
- **L991 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L991 CN**: 延续周围的声明、表达式或控制流结构。
- **L992 EN**: Returns from the current function, often propagating a computed result.
  **L992 CN**: 从当前函数返回，通常会传递一个计算结果。

### Lines 993-1024

````cpp
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
bool RTDEF(ReduceLogical8Value)(const Descriptor &array,
    ValueReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  return RTNAME(ReduceInteger8Value)(
             array, operation, source, line, dim, mask, identity, ordered) != 0;
}
void RTDEF(ReduceLogical8DimRef)(Descriptor &result, const Descriptor &array,
    ReferenceReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger8DimRef)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}
void RTDEF(ReduceLogical8DimValue)(Descriptor &result, const Descriptor &array,
    ValueReductionOperation<std::int64_t> operation, const char *source,
    int line, int dim, const Descriptor *mask, const std::int64_t *identity,
    bool ordered) {
  RTNAME(ReduceInteger8DimValue)
  (result, array, operation, source, line, dim, mask, identity, ordered);
}

void RTDEF(ReduceChar1)(char *result, const Descriptor &array,
    ReductionCharOperation<char> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char *identity, bool ordered) {
  Terminator terminator{source, line};
  BufferedReduceAccumulator<char, ReductionCharOperation<char>,
      /*hasLength=*/true>
      accumulator{array, operation, identity, terminator};
  DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);
````

- **L993 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L993 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L994 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L994 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L995 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L995 CN**: 延续周围的声明、表达式或控制流结构。
- **L996 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L996 CN**: 延续周围的声明、表达式或控制流结构。
- **L997 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L997 CN**: 延续周围的声明、表达式或控制流结构。
- **L998 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L998 CN**: 延续周围的声明、表达式或控制流结构。
- **L999 EN**: Returns from the current function, often propagating a computed result.
  **L999 CN**: 从当前函数返回，通常会传递一个计算结果。
- **L1000 EN**: Executes statement `array, operation, source, line, dim, mask, identity, ordered) != 0;`.
  **L1000 CN**: 执行语句 `array, operation, source, line, dim, mask, identity, ordered) != 0;`。
- **L1001 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1001 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1002 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1002 CN**: 延续周围的声明、表达式或控制流结构。
- **L1003 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1003 CN**: 延续周围的声明、表达式或控制流结构。
- **L1004 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1004 CN**: 延续周围的声明、表达式或控制流结构。
- **L1005 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1005 CN**: 延续周围的声明、表达式或控制流结构。
- **L1006 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1006 CN**: 延续周围的声明、表达式或控制流结构。
- **L1007 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L1007 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L1008 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1008 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1009 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1009 CN**: 延续周围的声明、表达式或控制流结构。
- **L1010 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1010 CN**: 延续周围的声明、表达式或控制流结构。
- **L1011 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1011 CN**: 延续周围的声明、表达式或控制流结构。
- **L1012 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1012 CN**: 延续周围的声明、表达式或控制流结构。
- **L1013 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1013 CN**: 延续周围的声明、表达式或控制流结构。
- **L1014 EN**: Executes statement `(result, array, operation, source, line, dim, mask, identity, ordered);`.
  **L1014 CN**: 执行语句 `(result, array, operation, source, line, dim, mask, identity, ordered);`。
- **L1015 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1015 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1016 EN**: Blank line separates nearby declarations or logic blocks.
  **L1016 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1017 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1017 CN**: 延续周围的声明、表达式或控制流结构。
- **L1018 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1018 CN**: 延续周围的声明、表达式或控制流结构。
- **L1019 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1019 CN**: 延续周围的声明、表达式或控制流结构。
- **L1020 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1020 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1021 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1021 CN**: 延续周围的声明、表达式或控制流结构。
- **L1022 EN**: Comment documents intent or context: `hasLength=*/true>`.
  **L1022 CN**: 注释记录了意图或上下文：`hasLength=*/true>`。
- **L1023 EN**: Executes statement `accumulator{array, operation, identity, terminator};`.
  **L1023 CN**: 执行语句 `accumulator{array, operation, identity, terminator};`。
- **L1024 EN**: Executes statement `DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);`.
  **L1024 CN**: 执行语句 `DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);`。

### Lines 1025-1056

````cpp
  accumulator.GetResult(result);
}
void RTDEF(ReduceCharacter1Dim)(Descriptor &result, const Descriptor &array,
    ReductionCharOperation<char> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = BufferedReduceAccumulator<char,
      ReductionCharOperation<char>, /*hasLength=*/true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Character, 1>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}
void RTDEF(ReduceChar2)(char16_t *result, const Descriptor &array,
    ReductionCharOperation<char16_t> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char16_t *identity, bool ordered) {
  Terminator terminator{source, line};
  BufferedReduceAccumulator<char16_t, ReductionCharOperation<char16_t>,
      /*hasLength=*/true>
      accumulator{array, operation, identity, terminator};
  DoTotalReduction<char16_t>(
      array, dim, mask, accumulator, "REDUCE", terminator);
  accumulator.GetResult(result);
}
void RTDEF(ReduceCharacter2Dim)(Descriptor &result, const Descriptor &array,
    ReductionCharOperation<char16_t> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char16_t *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = BufferedReduceAccumulator<char16_t,
      ReductionCharOperation<char16_t>, /*hasLength=*/true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Character, 2>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
````

- **L1025 EN**: Executes statement involving `GetResult`.
  **L1025 CN**: 执行涉及 `GetResult` 的语句。
- **L1026 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1026 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1027 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1027 CN**: 延续周围的声明、表达式或控制流结构。
- **L1028 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1028 CN**: 延续周围的声明、表达式或控制流结构。
- **L1029 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1029 CN**: 延续周围的声明、表达式或控制流结构。
- **L1030 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1030 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1031 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L1031 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L1032 EN**: Executes statement `ReductionCharOperation<char>, /*hasLength=*/true>;`.
  **L1032 CN**: 执行语句 `ReductionCharOperation<char>, /*hasLength=*/true>;`。
- **L1033 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L1033 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L1034 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1034 CN**: 延续周围的声明、表达式或控制流结构。
- **L1035 EN**: Executes statement involving `ElementBytes`.
  **L1035 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1036 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1036 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1037 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1037 CN**: 延续周围的声明、表达式或控制流结构。
- **L1038 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1038 CN**: 延续周围的声明、表达式或控制流结构。
- **L1039 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1039 CN**: 延续周围的声明、表达式或控制流结构。
- **L1040 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1040 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1041 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1041 CN**: 延续周围的声明、表达式或控制流结构。
- **L1042 EN**: Comment documents intent or context: `hasLength=*/true>`.
  **L1042 CN**: 注释记录了意图或上下文：`hasLength=*/true>`。
- **L1043 EN**: Executes statement `accumulator{array, operation, identity, terminator};`.
  **L1043 CN**: 执行语句 `accumulator{array, operation, identity, terminator};`。
- **L1044 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1044 CN**: 延续周围的声明、表达式或控制流结构。
- **L1045 EN**: Executes statement `array, dim, mask, accumulator, "REDUCE", terminator);`.
  **L1045 CN**: 执行语句 `array, dim, mask, accumulator, "REDUCE", terminator);`。
- **L1046 EN**: Executes statement involving `GetResult`.
  **L1046 CN**: 执行涉及 `GetResult` 的语句。
- **L1047 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1047 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1048 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1048 CN**: 延续周围的声明、表达式或控制流结构。
- **L1049 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1049 CN**: 延续周围的声明、表达式或控制流结构。
- **L1050 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1050 CN**: 延续周围的声明、表达式或控制流结构。
- **L1051 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1051 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1052 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L1052 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L1053 EN**: Executes statement `ReductionCharOperation<char16_t>, /*hasLength=*/true>;`.
  **L1053 CN**: 执行语句 `ReductionCharOperation<char16_t>, /*hasLength=*/true>;`。
- **L1054 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L1054 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L1055 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1055 CN**: 延续周围的声明、表达式或控制流结构。
- **L1056 EN**: Executes statement involving `ElementBytes`.
  **L1056 CN**: 执行涉及 `ElementBytes` 的语句。

### Lines 1057-1088

````cpp
}
void RTDEF(ReduceChar4)(char32_t *result, const Descriptor &array,
    ReductionCharOperation<char32_t> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char32_t *identity, bool ordered) {
  Terminator terminator{source, line};
  BufferedReduceAccumulator<char32_t, ReductionCharOperation<char32_t>,
      /*hasLength=*/true>
      accumulator{array, operation, identity, terminator};
  DoTotalReduction<char32_t>(
      array, dim, mask, accumulator, "REDUCE", terminator);
  accumulator.GetResult(result);
}
void RTDEF(ReduceCharacter4Dim)(Descriptor &result, const Descriptor &array,
    ReductionCharOperation<char32_t> operation, const char *source, int line,
    int dim, const Descriptor *mask, const char32_t *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = BufferedReduceAccumulator<char32_t,
      ReductionCharOperation<char32_t>, /*hasLength=*/true>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Character, 4>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}

void RTDEF(ReduceDerivedType)(char *result, const Descriptor &array,
    ReductionDerivedTypeOperation operation, const char *source, int line,
    int dim, const Descriptor *mask, const char *identity, bool ordered) {
  Terminator terminator{source, line};
  BufferedReduceAccumulator<char, ReductionDerivedTypeOperation,
      /*hasLength=*/false>
      accumulator{array, operation, identity, terminator};
  DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);
  accumulator.GetResult(result);
````

- **L1057 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1057 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1058 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1058 CN**: 延续周围的声明、表达式或控制流结构。
- **L1059 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1059 CN**: 延续周围的声明、表达式或控制流结构。
- **L1060 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1060 CN**: 延续周围的声明、表达式或控制流结构。
- **L1061 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1061 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1062 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1062 CN**: 延续周围的声明、表达式或控制流结构。
- **L1063 EN**: Comment documents intent or context: `hasLength=*/true>`.
  **L1063 CN**: 注释记录了意图或上下文：`hasLength=*/true>`。
- **L1064 EN**: Executes statement `accumulator{array, operation, identity, terminator};`.
  **L1064 CN**: 执行语句 `accumulator{array, operation, identity, terminator};`。
- **L1065 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1065 CN**: 延续周围的声明、表达式或控制流结构。
- **L1066 EN**: Executes statement `array, dim, mask, accumulator, "REDUCE", terminator);`.
  **L1066 CN**: 执行语句 `array, dim, mask, accumulator, "REDUCE", terminator);`。
- **L1067 EN**: Executes statement involving `GetResult`.
  **L1067 CN**: 执行涉及 `GetResult` 的语句。
- **L1068 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1068 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1069 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1069 CN**: 延续周围的声明、表达式或控制流结构。
- **L1070 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1070 CN**: 延续周围的声明、表达式或控制流结构。
- **L1071 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1071 CN**: 延续周围的声明、表达式或控制流结构。
- **L1072 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1072 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1073 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L1073 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L1074 EN**: Executes statement `ReductionCharOperation<char32_t>, /*hasLength=*/true>;`.
  **L1074 CN**: 执行语句 `ReductionCharOperation<char32_t>, /*hasLength=*/true>;`。
- **L1075 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L1075 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L1076 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1076 CN**: 延续周围的声明、表达式或控制流结构。
- **L1077 EN**: Executes statement involving `ElementBytes`.
  **L1077 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1078 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1078 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1079 EN**: Blank line separates nearby declarations or logic blocks.
  **L1079 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1080 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1080 CN**: 延续周围的声明、表达式或控制流结构。
- **L1081 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1081 CN**: 延续周围的声明、表达式或控制流结构。
- **L1082 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1082 CN**: 延续周围的声明、表达式或控制流结构。
- **L1083 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1083 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1084 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1084 CN**: 延续周围的声明、表达式或控制流结构。
- **L1085 EN**: Comment documents intent or context: `hasLength=*/false>`.
  **L1085 CN**: 注释记录了意图或上下文：`hasLength=*/false>`。
- **L1086 EN**: Executes statement `accumulator{array, operation, identity, terminator};`.
  **L1086 CN**: 执行语句 `accumulator{array, operation, identity, terminator};`。
- **L1087 EN**: Executes statement `DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);`.
  **L1087 CN**: 执行语句 `DoTotalReduction<char>(array, dim, mask, accumulator, "REDUCE", terminator);`。
- **L1088 EN**: Executes statement involving `GetResult`.
  **L1088 CN**: 执行涉及 `GetResult` 的语句。

### Lines 1089-1103

````cpp
}
void RTDEF(ReduceDerivedTypeDim)(Descriptor &result, const Descriptor &array,
    ReductionDerivedTypeOperation operation, const char *source, int line,
    int dim, const Descriptor *mask, const char *identity, bool ordered) {
  Terminator terminator{source, line};
  using Accumulator = BufferedReduceAccumulator<char,
      ReductionDerivedTypeOperation, /*hasLength=*/false>;
  Accumulator accumulator{array, operation, identity, terminator};
  PartialReduction<Accumulator, TypeCategory::Derived, 0>(result, array,
      array.ElementBytes(), dim, mask, terminator, "REDUCE", accumulator);
}

RT_EXT_API_GROUP_END
} // extern "C"
} // namespace Fortran::runtime
````

- **L1089 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1089 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1090 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1090 CN**: 延续周围的声明、表达式或控制流结构。
- **L1091 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1091 CN**: 延续周围的声明、表达式或控制流结构。
- **L1092 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1092 CN**: 延续周围的声明、表达式或控制流结构。
- **L1093 EN**: Executes statement `Terminator terminator{source, line};`.
  **L1093 CN**: 执行语句 `Terminator terminator{source, line};`。
- **L1094 EN**: Defines type alias `Accumulator` for readability or ABI convenience.
  **L1094 CN**: 定义类型别名 `Accumulator`，以提升可读性或满足 ABI 便利性。
- **L1095 EN**: Executes statement `ReductionDerivedTypeOperation, /*hasLength=*/false>;`.
  **L1095 CN**: 执行语句 `ReductionDerivedTypeOperation, /*hasLength=*/false>;`。
- **L1096 EN**: Executes statement `Accumulator accumulator{array, operation, identity, terminator};`.
  **L1096 CN**: 执行语句 `Accumulator accumulator{array, operation, identity, terminator};`。
- **L1097 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1097 CN**: 延续周围的声明、表达式或控制流结构。
- **L1098 EN**: Executes statement involving `ElementBytes`.
  **L1098 CN**: 执行涉及 `ElementBytes` 的语句。
- **L1099 EN**: Opens or closes a scope, aggregate, or declaration block.
  **L1099 CN**: 打开或关闭一个作用域、聚合体或声明块。
- **L1100 EN**: Blank line separates nearby declarations or logic blocks.
  **L1100 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L1101 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1101 CN**: 延续周围的声明、表达式或控制流结构。
- **L1102 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1102 CN**: 延续周围的声明、表达式或控制流结构。
- **L1103 EN**: Continues the surrounding declaration, expression, or control-flow structure.
  **L1103 CN**: 延续周围的声明、表达式或控制流结构。

## Key Concepts / 关键概念

- **Scale / 规模**: The file contains approximately 1103 source lines, which suggests a substantial implementation unit. / 该文件约有 1103 行源码，说明它是一个较大的实现单元。
- **Fortran execution semantics / Fortran 执行语义**: The implementation materializes language-level behavior such as descriptors, I/O state, allocation, reductions, and error handling. / 实现把语言层面的行为具体化，例如描述符、I/O 状态、分配、归约与错误处理。
- **Runtime layering / 运行时分层**: Source files in this area usually bridge generated code, public runtime headers, and lower-level system facilities. / 此处源码通常桥接生成代码、公共运行时头文件以及更底层的系统设施。
- **Interface surface / 接口表面**: Direct includes such as `flang/Runtime/reduce.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/reduction-templates.h`, `flang-rt/runtime/terminator.h` show which nearby abstractions this file relies on first. / 直接包含的头文件（如 `flang/Runtime/reduce.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/reduction-templates.h`, `flang-rt/runtime/terminator.h`）展示了此文件首先依赖的周边抽象。
- **Primary entry points / 主要入口点**: Notable callables include `AccumulateAt`, `constexpr`, `GetResult`. / 值得关注的可调用实体包括 `AccumulateAt`, `constexpr`, `GetResult`。
- **Core types / 核心类型**: Important declared or referenced types include `Operation`, `BufferedReduceAccumulator`, `Accumulator`. / 重要的已声明或被引用类型包括 `Operation`, `BufferedReduceAccumulator`, `Accumulator`。
- **Namespaces / 命名空间**: The code uses namespaces such as `Fortran` to organize symbols. / 代码使用 `Fortran` 等命名空间来组织符号。

## Dependencies / 依赖关系

- **Project headers / 项目头文件**: `flang/Runtime/reduce.h`, `flang-rt/runtime/descriptor.h`, `flang-rt/runtime/reduction-templates.h`, `flang-rt/runtime/terminator.h`, `flang-rt/runtime/tools.h`. These provide subsystem-specific declarations. / 这些头文件提供子系统专用声明。
- **Callable surface / 可调用表面**: Functions or methods defined here include `AccumulateAt`, `constexpr`, `GetResult`. These are likely the main entry points exported to nearby code. / 此处定义的函数或方法包括 `AccumulateAt`, `constexpr`, `GetResult`，它们通常是对周边代码暴露的主要入口。
- **Type coupling / 类型耦合**: Declared types such as `Operation`, `BufferedReduceAccumulator`, `Accumulator` capture the data model shared with dependent code. / `Operation`, `BufferedReduceAccumulator`, `Accumulator` 等声明类型体现了与依赖方共享的数据模型。
