# shape.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Evaluate/shape.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements compile-time Fortran expression evaluation logic and helpers for shape.
- **Purpose (CN)**: 实现 shape 相关的 Fortran 编译期表达式求值逻辑与辅助功能。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Evaluate/shape.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Evaluate/shape.h"
#include "flang/Common/idioms.h"
#include "flang/Common/template.h"
#include "flang/Evaluate/characteristics.h"
#include "flang/Evaluate/check-expression.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/intrinsics.h"
#include "flang/Evaluate/tools.h"
#include "flang/Evaluate/type.h"
#include "flang/Parser/message.h"
#include "flang/Semantics/semantics.h"
#include "flang/Semantics/symbol.h"
#include <functional>

using namespace std::placeholders; // _1, _2, &c. for std::bind()

````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, intent, or metadata: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明附近代码的逻辑、意图或元数据：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, intent, or metadata: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明附近代码的逻辑、意图或元数据：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, intent, or metadata: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明附近代码的逻辑、意图或元数据：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Includes "flang/Evaluate/shape.h" to access Fortran constant-folding and evaluation facilities.
  **L9 CN**: 引入 "flang/Evaluate/shape.h" 以使用Fortran 常量折叠与求值能力。
- **L10 EN**: Includes "flang/Common/idioms.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/idioms.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Common/template.h" to access shared Flang utility infrastructure.
  **L11 CN**: 引入 "flang/Common/template.h" 以使用Flang 共享工具基础设施。
- **L12 EN**: Includes "flang/Evaluate/characteristics.h" to access Fortran constant-folding and evaluation facilities.
  **L12 CN**: 引入 "flang/Evaluate/characteristics.h" 以使用Fortran 常量折叠与求值能力。
- **L13 EN**: Includes "flang/Evaluate/check-expression.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/check-expression.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L14 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L15 EN**: Includes "flang/Evaluate/intrinsics.h" to access Fortran constant-folding and evaluation facilities.
  **L15 CN**: 引入 "flang/Evaluate/intrinsics.h" 以使用Fortran 常量折叠与求值能力。
- **L16 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L16 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L17 EN**: Includes "flang/Evaluate/type.h" to access Fortran constant-folding and evaluation facilities.
  **L17 CN**: 引入 "flang/Evaluate/type.h" 以使用Fortran 常量折叠与求值能力。
- **L18 EN**: Includes "flang/Parser/message.h" to access parse-tree, token, or source representation support.
  **L18 CN**: 引入 "flang/Parser/message.h" 以使用语法树、词法单元或源码表示支持。
- **L19 EN**: Includes "flang/Semantics/semantics.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/semantics.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes "flang/Semantics/symbol.h" to access Fortran semantic analysis, symbol, and type information.
  **L20 CN**: 引入 "flang/Semantics/symbol.h" 以使用Fortran 语义分析、符号与类型信息。
- **L21 EN**: Includes <functional> to access supporting declarations used by this translation unit.
  **L21 CN**: 引入 <functional> 以使用当前编译单元使用的辅助声明。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Brings namespace `std::placeholders; // _1, _2, &c. for std::bind()` into the local scope.
  **L23 CN**: 将命名空间 `std::placeholders; // _1, _2, &c. for std::bind()` 引入当前作用域。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 25-48

````cpp
namespace Fortran::evaluate {

FoldingContext &GetFoldingContextFrom(const Symbol &symbol) {
  return symbol.owner().context().foldingContext();
}

bool IsImpliedShape(const Symbol &original) {
  const Symbol &symbol{ResolveAssociations(original)};
  const auto *details{symbol.detailsIf<semantics::ObjectEntityDetails>()};
  return details && symbol.attrs().test(semantics::Attr::PARAMETER) &&
      details->shape().CanBeImpliedShape();
}

bool IsExplicitShape(const Symbol &original) {
  const Symbol &symbol{ResolveAssociations(original)};
  if (const auto *details{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
    const auto &shape{details->shape()};
    return shape.Rank() == 0 ||
        shape.IsExplicitShape(); // true when scalar, too
  } else {
    return symbol
        .has<semantics::AssocEntityDetails>(); // exprs have explicit shape
  }
}
````
- **L25 EN**: Opens namespace scope `Fortran::evaluate`.
  **L25 CN**: 打开命名空间作用域 `Fortran::evaluate`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Starts a function, method, lambda, or structured scope: `FoldingContext &GetFoldingContextFrom(const Symbol &symbol) {`.
  **L27 CN**: 开始一个函数、方法、lambda 或结构化作用域：`FoldingContext &GetFoldingContextFrom(const Symbol &symbol) {`。
- **L28 EN**: Returns from the current function with `symbol.owner().context().foldingContext()`.
  **L28 CN**: 以 `symbol.owner().context().foldingContext()` 从当前函数返回。
- **L29 EN**: Closes the current lexical scope or compound statement.
  **L29 CN**: 结束当前词法作用域或复合语句块。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Starts a function, method, lambda, or structured scope: `bool IsImpliedShape(const Symbol &original) {`.
  **L31 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IsImpliedShape(const Symbol &original) {`。
- **L32 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L32 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L33 EN**: Executes a call or declaration centered on `*details{symbol.detailsIf<semantics::ObjectEntityDetails>`.
  **L33 CN**: 执行以 `*details{symbol.detailsIf<semantics::ObjectEntityDetails>` 为核心的调用或声明。
- **L34 EN**: Returns from the current function with `details && symbol.attrs().test(semantics::Attr::PARAMETER) &&`.
  **L34 CN**: 以 `details && symbol.attrs().test(semantics::Attr::PARAMETER) &&` 从当前函数返回。
- **L35 EN**: Executes a call or declaration centered on `details->shape`.
  **L35 CN**: 执行以 `details->shape` 为核心的调用或声明。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Starts a function, method, lambda, or structured scope: `bool IsExplicitShape(const Symbol &original) {`.
  **L38 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool IsExplicitShape(const Symbol &original) {`。
- **L39 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L39 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L40 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L40 CN**: 开始 `if` 控制流语句并计算其条件。
- **L41 EN**: Executes a call or declaration centered on `&shape{details->shape`.
  **L41 CN**: 执行以 `&shape{details->shape` 为核心的调用或声明。
- **L42 EN**: Returns from the current function with `shape.Rank() == 0 ||`.
  **L42 CN**: 以 `shape.Rank() == 0 ||` 从当前函数返回。
- **L43 EN**: Continues logic associated with callable symbol `IsExplicitShape`.
  **L43 CN**: 继续与可调用符号 `IsExplicitShape` 相关的逻辑。
- **L44 EN**: Transitions from the previous branch into the alternative path.
  **L44 CN**: 从前一个分支过渡到备选路径。
- **L45 EN**: Returns from the current function with `symbol`.
  **L45 CN**: 以 `symbol` 从当前函数返回。
- **L46 EN**: Continues logic associated with callable symbol `AssocEntityDetails>`.
  **L46 CN**: 继续与可调用符号 `AssocEntityDetails>` 相关的逻辑。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-72

````cpp

Shape GetShapeHelper::ConstantShape(const Constant<ExtentType> &arrayConstant) {
  CHECK(arrayConstant.Rank() == 1);
  Shape result;
  std::size_t dimensions{arrayConstant.size()};
  for (std::size_t j{0}; j < dimensions; ++j) {
    Scalar<ExtentType> extent{arrayConstant.values().at(j)};
    result.emplace_back(MaybeExtentExpr{ExtentExpr{std::move(extent)}});
  }
  return result;
}

auto GetShapeHelper::AsShapeResult(ExtentExpr &&arrayExpr) const -> Result {
  if (context_) {
    arrayExpr = Fold(*context_, std::move(arrayExpr));
  }
  if (const auto *constArray{UnwrapConstantValue<ExtentType>(arrayExpr)}) {
    return ConstantShape(*constArray);
  }
  if (auto *constructor{UnwrapExpr<ArrayConstructor<ExtentType>>(arrayExpr)}) {
    Shape result;
    for (auto &value : *constructor) {
      auto *expr{std::get_if<ExtentExpr>(&value.u)};
      if (expr && expr->Rank() == 0) {
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Starts a function, method, lambda, or structured scope: `Shape GetShapeHelper::ConstantShape(const Constant<ExtentType> &arrayConstant) {`.
  **L50 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetShapeHelper::ConstantShape(const Constant<ExtentType> &arrayConstant) {`。
- **L51 EN**: Executes a call or declaration centered on `CHECK`.
  **L51 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L52 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L52 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L53 EN**: Executes a call or declaration centered on `dimensions{arrayConstant.size`.
  **L53 CN**: 执行以 `dimensions{arrayConstant.size` 为核心的调用或声明。
- **L54 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L54 CN**: 开始 `for` 控制流语句并计算其条件。
- **L55 EN**: Executes a call or declaration centered on `extent{arrayConstant.values`.
  **L55 CN**: 执行以 `extent{arrayConstant.values` 为核心的调用或声明。
- **L56 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L56 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Returns from the current function with `result`.
  **L58 CN**: 以 `result` 从当前函数返回。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::AsShapeResult(ExtentExpr &&arrayExpr) const -> Result {`.
  **L61 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::AsShapeResult(ExtentExpr &&arrayExpr) const -> Result {`。
- **L62 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L62 CN**: 开始 `if` 控制流语句并计算其条件。
- **L63 EN**: Executes a call or declaration centered on `Fold`.
  **L63 CN**: 执行以 `Fold` 为核心的调用或声明。
- **L64 EN**: Closes the current lexical scope or compound statement.
  **L64 CN**: 结束当前词法作用域或复合语句块。
- **L65 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `if` 控制流语句并计算其条件。
- **L66 EN**: Returns from the current function with `ConstantShape(*constArray)`.
  **L66 CN**: 以 `ConstantShape(*constArray)` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L68 CN**: 开始 `if` 控制流语句并计算其条件。
- **L69 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L69 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L70 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L70 CN**: 开始 `for` 控制流语句并计算其条件。
- **L71 EN**: Executes a call or declaration centered on `*expr{std::get_if<ExtentExpr>`.
  **L71 CN**: 执行以 `*expr{std::get_if<ExtentExpr>` 为核心的调用或声明。
- **L72 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L72 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 73-96

````cpp
        result.emplace_back(std::move(*expr));
      } else {
        return std::nullopt;
      }
    }
    return result;
  } else {
    return std::nullopt;
  }
}

Shape GetShapeHelper::CreateShape(int rank, NamedEntity &base) const {
  Shape shape;
  for (int dimension{0}; dimension < rank; ++dimension) {
    shape.emplace_back(GetExtent(base, dimension, invariantOnly_));
  }
  return shape;
}

std::optional<ExtentExpr> AsExtentArrayExpr(const Shape &shape) {
  ArrayConstructorValues<ExtentType> values;
  for (const auto &dim : shape) {
    if (dim) {
      values.Push(common::Clone(*dim));
````
- **L73 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L73 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L74 EN**: Transitions from the previous branch into the alternative path.
  **L74 CN**: 从前一个分支过渡到备选路径。
- **L75 EN**: Returns from the current function with `std::nullopt`.
  **L75 CN**: 以 `std::nullopt` 从当前函数返回。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Returns from the current function with `result`.
  **L78 CN**: 以 `result` 从当前函数返回。
- **L79 EN**: Transitions from the previous branch into the alternative path.
  **L79 CN**: 从前一个分支过渡到备选路径。
- **L80 EN**: Returns from the current function with `std::nullopt`.
  **L80 CN**: 以 `std::nullopt` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Starts a function, method, lambda, or structured scope: `Shape GetShapeHelper::CreateShape(int rank, NamedEntity &base) const {`.
  **L84 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetShapeHelper::CreateShape(int rank, NamedEntity &base) const {`。
- **L85 EN**: Executes a standalone statement or declaration: `Shape shape;`.
  **L85 CN**: 执行一条独立语句或声明：`Shape shape;`。
- **L86 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L86 CN**: 开始 `for` 控制流语句并计算其条件。
- **L87 EN**: Executes a call or declaration centered on `shape.emplace_back`.
  **L87 CN**: 执行以 `shape.emplace_back` 为核心的调用或声明。
- **L88 EN**: Closes the current lexical scope or compound statement.
  **L88 CN**: 结束当前词法作用域或复合语句块。
- **L89 EN**: Returns from the current function with `shape`.
  **L89 CN**: 以 `shape` 从当前函数返回。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Starts a function, method, lambda, or structured scope: `std::optional<ExtentExpr> AsExtentArrayExpr(const Shape &shape) {`.
  **L92 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::optional<ExtentExpr> AsExtentArrayExpr(const Shape &shape) {`。
- **L93 EN**: Executes a standalone statement or declaration: `ArrayConstructorValues<ExtentType> values;`.
  **L93 CN**: 执行一条独立语句或声明：`ArrayConstructorValues<ExtentType> values;`。
- **L94 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L94 CN**: 开始 `for` 控制流语句并计算其条件。
- **L95 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L95 CN**: 开始 `if` 控制流语句并计算其条件。
- **L96 EN**: Executes a call or declaration centered on `values.Push`.
  **L96 CN**: 执行以 `values.Push` 为核心的调用或声明。

### Lines 97-120

````cpp
    } else {
      return std::nullopt;
    }
  }
  return ExtentExpr{ArrayConstructor<ExtentType>{std::move(values)}};
}

std::optional<Constant<ExtentType>> AsConstantShape(
    FoldingContext &context, const Shape &shape) {
  if (auto shapeArray{AsExtentArrayExpr(shape)}) {
    auto folded{Fold(context, std::move(*shapeArray))};
    if (auto *p{UnwrapConstantValue<ExtentType>(folded)}) {
      return std::move(*p);
    }
  }
  return std::nullopt;
}

Constant<SubscriptInteger> AsConstantShape(const ConstantSubscripts &shape) {
  using IntType = Scalar<SubscriptInteger>;
  std::vector<IntType> result;
  for (auto dim : shape) {
    result.emplace_back(dim);
  }
````
- **L97 EN**: Transitions from the previous branch into the alternative path.
  **L97 CN**: 从前一个分支过渡到备选路径。
- **L98 EN**: Returns from the current function with `std::nullopt`.
  **L98 CN**: 以 `std::nullopt` 从当前函数返回。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。
- **L101 EN**: Returns from the current function with `ExtentExpr{ArrayConstructor<ExtentType>{std::move(values)}}`.
  **L101 CN**: 以 `ExtentExpr{ArrayConstructor<ExtentType>{std::move(values)}}` 从当前函数返回。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L104 EN**: Continues logic associated with callable symbol `AsConstantShape`.
  **L104 CN**: 继续与可调用符号 `AsConstantShape` 相关的逻辑。
- **L105 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, const Shape &shape) {`.
  **L105 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, const Shape &shape) {`。
- **L106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L107 EN**: Executes a call or declaration centered on `folded{Fold`.
  **L107 CN**: 执行以 `folded{Fold` 为核心的调用或声明。
- **L108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L108 CN**: 开始 `if` 控制流语句并计算其条件。
- **L109 EN**: Returns from the current function with `std::move(*p)`.
  **L109 CN**: 以 `std::move(*p)` 从当前函数返回。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Returns from the current function with `std::nullopt`.
  **L112 CN**: 以 `std::nullopt` 从当前函数返回。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Starts a function, method, lambda, or structured scope: `Constant<SubscriptInteger> AsConstantShape(const ConstantSubscripts &shape) {`.
  **L115 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Constant<SubscriptInteger> AsConstantShape(const ConstantSubscripts &shape) {`。
- **L116 EN**: Defines alias `IntType` to simplify later code.
  **L116 CN**: 定义别名 `IntType` 以简化后续代码。
- **L117 EN**: Executes a standalone statement or declaration: `std::vector<IntType> result;`.
  **L117 CN**: 执行一条独立语句或声明：`std::vector<IntType> result;`。
- **L118 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L118 CN**: 开始 `for` 控制流语句并计算其条件。
- **L119 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L119 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-144

````cpp
  return {std::move(result), ConstantSubscripts{GetRank(shape)}};
}

ConstantSubscripts AsConstantExtents(const Constant<ExtentType> &shape) {
  ConstantSubscripts result;
  for (const auto &extent : shape.values()) {
    result.push_back(extent.ToInt64());
  }
  return result;
}

std::optional<ConstantSubscripts> AsConstantExtents(
    FoldingContext &context, const Shape &shape) {
  if (auto shapeConstant{AsConstantShape(context, shape)}) {
    return AsConstantExtents(*shapeConstant);
  } else {
    return std::nullopt;
  }
}

Shape AsShape(const ConstantSubscripts &shape) {
  Shape result;
  for (const auto &extent : shape) {
    result.emplace_back(ExtentExpr{extent});
````
- **L121 EN**: Returns from the current function with `{std::move(result), ConstantSubscripts{GetRank(shape)}}`.
  **L121 CN**: 以 `{std::move(result), ConstantSubscripts{GetRank(shape)}}` 从当前函数返回。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L124 EN**: Starts a function, method, lambda, or structured scope: `ConstantSubscripts AsConstantExtents(const Constant<ExtentType> &shape) {`.
  **L124 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ConstantSubscripts AsConstantExtents(const Constant<ExtentType> &shape) {`。
- **L125 EN**: Executes a standalone statement or declaration: `ConstantSubscripts result;`.
  **L125 CN**: 执行一条独立语句或声明：`ConstantSubscripts result;`。
- **L126 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L126 CN**: 开始 `for` 控制流语句并计算其条件。
- **L127 EN**: Executes a call or declaration centered on `result.push_back`.
  **L127 CN**: 执行以 `result.push_back` 为核心的调用或声明。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Returns from the current function with `result`.
  **L129 CN**: 以 `result` 从当前函数返回。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L132 EN**: Continues logic associated with callable symbol `AsConstantExtents`.
  **L132 CN**: 继续与可调用符号 `AsConstantExtents` 相关的逻辑。
- **L133 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, const Shape &shape) {`.
  **L133 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, const Shape &shape) {`。
- **L134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L135 EN**: Returns from the current function with `AsConstantExtents(*shapeConstant)`.
  **L135 CN**: 以 `AsConstantExtents(*shapeConstant)` 从当前函数返回。
- **L136 EN**: Transitions from the previous branch into the alternative path.
  **L136 CN**: 从前一个分支过渡到备选路径。
- **L137 EN**: Returns from the current function with `std::nullopt`.
  **L137 CN**: 以 `std::nullopt` 从当前函数返回。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L141 EN**: Starts a function, method, lambda, or structured scope: `Shape AsShape(const ConstantSubscripts &shape) {`.
  **L141 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape AsShape(const ConstantSubscripts &shape) {`。
- **L142 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L142 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L143 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L143 CN**: 开始 `for` 控制流语句并计算其条件。
- **L144 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L144 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。

### Lines 145-168

````cpp
  }
  return result;
}

std::optional<Shape> AsShape(const std::optional<ConstantSubscripts> &shape) {
  if (shape) {
    return AsShape(*shape);
  } else {
    return std::nullopt;
  }
}

Shape Fold(FoldingContext &context, Shape &&shape) {
  for (auto &dim : shape) {
    dim = Fold(context, std::move(dim));
  }
  return std::move(shape);
}

std::optional<Shape> Fold(
    FoldingContext &context, std::optional<Shape> &&shape) {
  if (shape) {
    return Fold(context, std::move(*shape));
  } else {
````
- **L145 EN**: Closes the current lexical scope or compound statement.
  **L145 CN**: 结束当前词法作用域或复合语句块。
- **L146 EN**: Returns from the current function with `result`.
  **L146 CN**: 以 `result` 从当前函数返回。
- **L147 EN**: Closes the current lexical scope or compound statement.
  **L147 CN**: 结束当前词法作用域或复合语句块。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Starts a function, method, lambda, or structured scope: `std::optional<Shape> AsShape(const std::optional<ConstantSubscripts> &shape) {`.
  **L149 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::optional<Shape> AsShape(const std::optional<ConstantSubscripts> &shape) {`。
- **L150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L151 EN**: Returns from the current function with `AsShape(*shape)`.
  **L151 CN**: 以 `AsShape(*shape)` 从当前函数返回。
- **L152 EN**: Transitions from the previous branch into the alternative path.
  **L152 CN**: 从前一个分支过渡到备选路径。
- **L153 EN**: Returns from the current function with `std::nullopt`.
  **L153 CN**: 以 `std::nullopt` 从当前函数返回。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L157 EN**: Starts a function, method, lambda, or structured scope: `Shape Fold(FoldingContext &context, Shape &&shape) {`.
  **L157 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape Fold(FoldingContext &context, Shape &&shape) {`。
- **L158 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `for` 控制流语句并计算其条件。
- **L159 EN**: Executes a call or declaration centered on `Fold`.
  **L159 CN**: 执行以 `Fold` 为核心的调用或声明。
- **L160 EN**: Closes the current lexical scope or compound statement.
  **L160 CN**: 结束当前词法作用域或复合语句块。
- **L161 EN**: Returns from the current function with `std::move(shape)`.
  **L161 CN**: 以 `std::move(shape)` 从当前函数返回。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Continues logic associated with callable symbol `Fold`.
  **L164 CN**: 继续与可调用符号 `Fold` 相关的逻辑。
- **L165 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, std::optional<Shape> &&shape) {`.
  **L165 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, std::optional<Shape> &&shape) {`。
- **L166 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L166 CN**: 开始 `if` 控制流语句并计算其条件。
- **L167 EN**: Returns from the current function with `Fold(context, std::move(*shape))`.
  **L167 CN**: 以 `Fold(context, std::move(*shape))` 从当前函数返回。
- **L168 EN**: Transitions from the previous branch into the alternative path.
  **L168 CN**: 从前一个分支过渡到备选路径。

### Lines 169-192

````cpp
    return std::nullopt;
  }
}

static ExtentExpr ComputeTripCount(
    ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {
  ExtentExpr strideCopy{common::Clone(stride)};
  ExtentExpr span{
      (std::move(upper) - std::move(lower) + std::move(strideCopy)) /
      std::move(stride)};
  return ExtentExpr{
      Extremum<ExtentType>{Ordering::Greater, std::move(span), ExtentExpr{0}}};
}

ExtentExpr CountTrips(
    ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {
  return ComputeTripCount(
      std::move(lower), std::move(upper), std::move(stride));
}

ExtentExpr CountTrips(const ExtentExpr &lower, const ExtentExpr &upper,
    const ExtentExpr &stride) {
  return ComputeTripCount(
      common::Clone(lower), common::Clone(upper), common::Clone(stride));
````
- **L169 EN**: Returns from the current function with `std::nullopt`.
  **L169 CN**: 以 `std::nullopt` 从当前函数返回。
- **L170 EN**: Closes the current lexical scope or compound statement.
  **L170 CN**: 结束当前词法作用域或复合语句块。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Continues logic associated with callable symbol `ComputeTripCount`.
  **L173 CN**: 继续与可调用符号 `ComputeTripCount` 相关的逻辑。
- **L174 EN**: Continues the surrounding expression or declaration: `ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {`.
  **L174 CN**: 继续构造周围的表达式或声明：`ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {`。
- **L175 EN**: Executes a call or declaration centered on `strideCopy{common::Clone`.
  **L175 CN**: 执行以 `strideCopy{common::Clone` 为核心的调用或声明。
- **L176 EN**: Continues the surrounding expression or declaration: `ExtentExpr span{`.
  **L176 CN**: 继续构造周围的表达式或声明：`ExtentExpr span{`。
- **L177 EN**: Continues logic associated with callable symbol `move`.
  **L177 CN**: 继续与可调用符号 `move` 相关的逻辑。
- **L178 EN**: Executes a call or declaration centered on `std::move`.
  **L178 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L179 EN**: Returns from the current function with `ExtentExpr{`.
  **L179 CN**: 以 `ExtentExpr{` 从当前函数返回。
- **L180 EN**: Executes a call or declaration centered on `std::move`.
  **L180 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L183 EN**: Continues logic associated with callable symbol `CountTrips`.
  **L183 CN**: 继续与可调用符号 `CountTrips` 相关的逻辑。
- **L184 EN**: Continues the surrounding expression or declaration: `ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {`.
  **L184 CN**: 继续构造周围的表达式或声明：`ExtentExpr &&lower, ExtentExpr &&upper, ExtentExpr &&stride) {`。
- **L185 EN**: Returns from the current function with `ComputeTripCount(`.
  **L185 CN**: 以 `ComputeTripCount(` 从当前函数返回。
- **L186 EN**: Executes a call or declaration centered on `std::move`.
  **L186 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExtentExpr CountTrips(const ExtentExpr &lower, const ExtentExpr &upper,`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExtentExpr CountTrips(const ExtentExpr &lower, const ExtentExpr &upper,`。
- **L190 EN**: Continues the surrounding expression or declaration: `const ExtentExpr &stride) {`.
  **L190 CN**: 继续构造周围的表达式或声明：`const ExtentExpr &stride) {`。
- **L191 EN**: Returns from the current function with `ComputeTripCount(`.
  **L191 CN**: 以 `ComputeTripCount(` 从当前函数返回。
- **L192 EN**: Executes a call or declaration centered on `common::Clone`.
  **L192 CN**: 执行以 `common::Clone` 为核心的调用或声明。

### Lines 193-216

````cpp
}

MaybeExtentExpr CountTrips(MaybeExtentExpr &&lower, MaybeExtentExpr &&upper,
    MaybeExtentExpr &&stride) {
  std::function<ExtentExpr(ExtentExpr &&, ExtentExpr &&, ExtentExpr &&)> bound{
      std::bind(ComputeTripCount, _1, _2, _3)};
  return common::MapOptional(
      std::move(bound), std::move(lower), std::move(upper), std::move(stride));
}

MaybeExtentExpr GetSize(Shape &&shape) {
  ExtentExpr extent{1};
  for (auto &&dim : std::move(shape)) {
    if (dim) {
      extent = std::move(extent) * std::move(*dim);
    } else {
      return std::nullopt;
    }
  }
  return extent;
}

ConstantSubscript GetSize(const ConstantSubscripts &shape) {
  ConstantSubscript size{1};
````
- **L193 EN**: Closes the current lexical scope or compound statement.
  **L193 CN**: 结束当前词法作用域或复合语句块。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr CountTrips(MaybeExtentExpr &&lower, MaybeExtentExpr &&upper,`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr CountTrips(MaybeExtentExpr &&lower, MaybeExtentExpr &&upper,`。
- **L196 EN**: Continues the surrounding expression or declaration: `MaybeExtentExpr &&stride) {`.
  **L196 CN**: 继续构造周围的表达式或声明：`MaybeExtentExpr &&stride) {`。
- **L197 EN**: Starts a function, method, lambda, or structured scope: `std::function<ExtentExpr(ExtentExpr &&, ExtentExpr &&, ExtentExpr &&)> bound{`.
  **L197 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::function<ExtentExpr(ExtentExpr &&, ExtentExpr &&, ExtentExpr &&)> bound{`。
- **L198 EN**: Executes a call or declaration centered on `std::bind`.
  **L198 CN**: 执行以 `std::bind` 为核心的调用或声明。
- **L199 EN**: Returns from the current function with `common::MapOptional(`.
  **L199 CN**: 以 `common::MapOptional(` 从当前函数返回。
- **L200 EN**: Executes a call or declaration centered on `std::move`.
  **L200 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L201 EN**: Closes the current lexical scope or compound statement.
  **L201 CN**: 结束当前词法作用域或复合语句块。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L203 EN**: Starts a function, method, lambda, or structured scope: `MaybeExtentExpr GetSize(Shape &&shape) {`.
  **L203 CN**: 开始一个函数、方法、lambda 或结构化作用域：`MaybeExtentExpr GetSize(Shape &&shape) {`。
- **L204 EN**: Executes a standalone statement or declaration: `ExtentExpr extent{1};`.
  **L204 CN**: 执行一条独立语句或声明：`ExtentExpr extent{1};`。
- **L205 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L205 CN**: 开始 `for` 控制流语句并计算其条件。
- **L206 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L206 CN**: 开始 `if` 控制流语句并计算其条件。
- **L207 EN**: Executes a call or declaration centered on `std::move`.
  **L207 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L208 EN**: Transitions from the previous branch into the alternative path.
  **L208 CN**: 从前一个分支过渡到备选路径。
- **L209 EN**: Returns from the current function with `std::nullopt`.
  **L209 CN**: 以 `std::nullopt` 从当前函数返回。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Returns from the current function with `extent`.
  **L212 CN**: 以 `extent` 从当前函数返回。
- **L213 EN**: Closes the current lexical scope or compound statement.
  **L213 CN**: 结束当前词法作用域或复合语句块。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Starts a function, method, lambda, or structured scope: `ConstantSubscript GetSize(const ConstantSubscripts &shape) {`.
  **L215 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ConstantSubscript GetSize(const ConstantSubscripts &shape) {`。
- **L216 EN**: Executes a standalone statement or declaration: `ConstantSubscript size{1};`.
  **L216 CN**: 执行一条独立语句或声明：`ConstantSubscript size{1};`。

### Lines 217-240

````cpp
  for (auto dim : shape) {
    CHECK(dim >= 0);
    size *= dim;
  }
  return size;
}

bool ContainsAnyImpliedDoIndex(const ExtentExpr &expr) {
  struct MyVisitor : public AnyTraverse<MyVisitor> {
    using Base = AnyTraverse<MyVisitor>;
    MyVisitor() : Base{*this} {}
    using Base::operator();
    bool operator()(const ImpliedDoIndex &) { return true; }
  };
  return MyVisitor{}(expr);
}

// Determines lower bound on a dimension.  This can be other than 1 only
// for a reference to a whole array object or component. (See LBOUND, 16.9.109).
// ASSOCIATE construct entities may require traversal of their referents.
template <typename RESULT, bool LBOUND_SEMANTICS>
class GetLowerBoundHelper
    : public Traverse<GetLowerBoundHelper<RESULT, LBOUND_SEMANTICS>, RESULT> {
public:
````
- **L217 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L217 CN**: 开始 `for` 控制流语句并计算其条件。
- **L218 EN**: Executes a call or declaration centered on `CHECK`.
  **L218 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L219 EN**: Executes a standalone statement or declaration: `size *= dim;`.
  **L219 CN**: 执行一条独立语句或声明：`size *= dim;`。
- **L220 EN**: Closes the current lexical scope or compound statement.
  **L220 CN**: 结束当前词法作用域或复合语句块。
- **L221 EN**: Returns from the current function with `size`.
  **L221 CN**: 以 `size` 从当前函数返回。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Starts a function, method, lambda, or structured scope: `bool ContainsAnyImpliedDoIndex(const ExtentExpr &expr) {`.
  **L224 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool ContainsAnyImpliedDoIndex(const ExtentExpr &expr) {`。
- **L225 EN**: Declares struct `MyVisitor`.
  **L225 CN**: 声明 struct `MyVisitor`。
- **L226 EN**: Defines alias `Base` to simplify later code.
  **L226 CN**: 定义别名 `Base` 以简化后续代码。
- **L227 EN**: Continues logic associated with callable symbol `MyVisitor`.
  **L227 CN**: 继续与可调用符号 `MyVisitor` 相关的逻辑。
- **L228 EN**: Executes a call or declaration centered on `Base::operator`.
  **L228 CN**: 执行以 `Base::operator` 为核心的调用或声明。
- **L229 EN**: Continues logic associated with callable symbol `operator`.
  **L229 CN**: 继续与可调用符号 `operator` 相关的逻辑。
- **L230 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L230 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L231 EN**: Returns from the current function with `MyVisitor{}(expr)`.
  **L231 CN**: 以 `MyVisitor{}(expr)` 从当前函数返回。
- **L232 EN**: Closes the current lexical scope or compound statement.
  **L232 CN**: 结束当前词法作用域或复合语句块。
- **L233 EN**: Blank line separating nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L234 EN**: Comment explains nearby logic, intent, or metadata: `Determines lower bound on a dimension.  This can be other than 1 only`.
  **L234 CN**: 注释说明附近代码的逻辑、意图或元数据：`Determines lower bound on a dimension.  This can be other than 1 only`。
- **L235 EN**: Comment explains nearby logic, intent, or metadata: `for a reference to a whole array object or component. (See LBOUND, 16.9.109).`.
  **L235 CN**: 注释说明附近代码的逻辑、意图或元数据：`for a reference to a whole array object or component. (See LBOUND, 16.9.109).`。
- **L236 EN**: Comment explains nearby logic, intent, or metadata: `ASSOCIATE construct entities may require traversal of their referents.`.
  **L236 CN**: 注释说明附近代码的逻辑、意图或元数据：`ASSOCIATE construct entities may require traversal of their referents.`。
- **L237 EN**: Introduces template parameters or specialization context: `template <typename RESULT, bool LBOUND_SEMANTICS>`.
  **L237 CN**: 为后续声明引入模板参数或特化上下文：`template <typename RESULT, bool LBOUND_SEMANTICS>`。
- **L238 EN**: Declares class `GetLowerBoundHelper`.
  **L238 CN**: 声明 class `GetLowerBoundHelper`。
- **L239 EN**: Continues the surrounding expression or declaration: `: public Traverse<GetLowerBoundHelper<RESULT, LBOUND_SEMANTICS>, RESULT> {`.
  **L239 CN**: 继续构造周围的表达式或声明：`: public Traverse<GetLowerBoundHelper<RESULT, LBOUND_SEMANTICS>, RESULT> {`。
- **L240 EN**: Sets the following members to `public` access.
  **L240 CN**: 将后续成员的访问级别设为 `public`。

### Lines 241-264

````cpp
  using Result = RESULT;
  using Base = Traverse<GetLowerBoundHelper, RESULT>;
  using Base::operator();
  explicit GetLowerBoundHelper(
      int d, FoldingContext *context, bool invariantOnly)
      : Base{*this}, dimension_{d}, context_{context},
        invariantOnly_{invariantOnly} {}
  static Result Default() { return Result{1}; }
  static Result Combine(Result &&, Result &&) {
    // Operator results and array references always have lower bounds == 1
    return Result{1};
  }

  Result GetLowerBound(const Symbol &symbol0, NamedEntity &&base) const {
    const Symbol &symbol{symbol0.GetUltimate()};
    if (const auto *object{
            symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
      int rank{object->shape().Rank()};
      if (dimension_ < rank) {
        const semantics::ShapeSpec &shapeSpec{object->shape()[dimension_]};
        if (shapeSpec.lbound().isExplicit()) {
          if (const auto &lbound{shapeSpec.lbound().GetExplicit()};
              lbound && lbound->Rank() == 0) {
            if constexpr (LBOUND_SEMANTICS) {
````
- **L241 EN**: Defines alias `Result` to simplify later code.
  **L241 CN**: 定义别名 `Result` 以简化后续代码。
- **L242 EN**: Defines alias `Base` to simplify later code.
  **L242 CN**: 定义别名 `Base` 以简化后续代码。
- **L243 EN**: Executes a call or declaration centered on `Base::operator`.
  **L243 CN**: 执行以 `Base::operator` 为核心的调用或声明。
- **L244 EN**: Continues logic associated with callable symbol `GetLowerBoundHelper`.
  **L244 CN**: 继续与可调用符号 `GetLowerBoundHelper` 相关的逻辑。
- **L245 EN**: Continues the surrounding expression or declaration: `int d, FoldingContext *context, bool invariantOnly)`.
  **L245 CN**: 继续构造周围的表达式或声明：`int d, FoldingContext *context, bool invariantOnly)`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Base{*this}, dimension_{d}, context_{context},`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Base{*this}, dimension_{d}, context_{context},`。
- **L247 EN**: Continues the surrounding expression or declaration: `invariantOnly_{invariantOnly} {}`.
  **L247 CN**: 继续构造周围的表达式或声明：`invariantOnly_{invariantOnly} {}`。
- **L248 EN**: Continues logic associated with callable symbol `Default`.
  **L248 CN**: 继续与可调用符号 `Default` 相关的逻辑。
- **L249 EN**: Starts a function, method, lambda, or structured scope: `static Result Combine(Result &&, Result &&) {`.
  **L249 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static Result Combine(Result &&, Result &&) {`。
- **L250 EN**: Comment explains nearby logic, intent, or metadata: `Operator results and array references always have lower bounds == 1`.
  **L250 CN**: 注释说明附近代码的逻辑、意图或元数据：`Operator results and array references always have lower bounds == 1`。
- **L251 EN**: Returns from the current function with `Result{1}`.
  **L251 CN**: 以 `Result{1}` 从当前函数返回。
- **L252 EN**: Closes the current lexical scope or compound statement.
  **L252 CN**: 结束当前词法作用域或复合语句块。
- **L253 EN**: Blank line separating nearby declarations or logic blocks.
  **L253 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L254 EN**: Starts a function, method, lambda, or structured scope: `Result GetLowerBound(const Symbol &symbol0, NamedEntity &&base) const {`.
  **L254 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result GetLowerBound(const Symbol &symbol0, NamedEntity &&base) const {`。
- **L255 EN**: Executes a call or declaration centered on `&symbol{symbol0.GetUltimate`.
  **L255 CN**: 执行以 `&symbol{symbol0.GetUltimate` 为核心的调用或声明。
- **L256 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L256 CN**: 开始 `if` 控制流语句并计算其条件。
- **L257 EN**: Starts a function, method, lambda, or structured scope: `symbol.detailsIf<semantics::ObjectEntityDetails>()}) {`.
  **L257 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.detailsIf<semantics::ObjectEntityDetails>()}) {`。
- **L258 EN**: Executes a call or declaration centered on `rank{object->shape`.
  **L258 CN**: 执行以 `rank{object->shape` 为核心的调用或声明。
- **L259 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L259 CN**: 开始 `if` 控制流语句并计算其条件。
- **L260 EN**: Executes a call or declaration centered on `&shapeSpec{object->shape`.
  **L260 CN**: 执行以 `&shapeSpec{object->shape` 为核心的调用或声明。
- **L261 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L261 CN**: 开始 `if` 控制流语句并计算其条件。
- **L262 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L262 CN**: 开始 `if` 控制流语句并计算其条件。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `lbound && lbound->Rank() == 0) {`.
  **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`lbound && lbound->Rank() == 0) {`。
- **L264 EN**: Continues logic associated with callable symbol `constexpr`.
  **L264 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。

### Lines 265-288

````cpp
              bool ok{false};
              auto lbValue{ToInt64(*lbound)};
              if (dimension_ == rank - 1 &&
                  semantics::IsAssumedSizeArray(symbol)) {
                // last dimension of assumed-size dummy array: don't worry
                // about handling an empty dimension
                ok = !invariantOnly_ || IsScopeInvariantExpr(*lbound, context_);
              } else if (lbValue.value_or(0) == 1) {
                // Lower bound is 1, regardless of extent
                ok = true;
              } else if (const auto &ubound{shapeSpec.ubound().GetExplicit()};
                  ubound && ubound->Rank() == 0) {
                // If we can't prove that the dimension is nonempty,
                // we must be conservative.
                // TODO: simple symbolic math in expression rewriting to
                // cope with cases like A(J:J)
                if (context_) {
                  auto extent{ToInt64(Fold(*context_,
                      ExtentExpr{*ubound} - ExtentExpr{*lbound} +
                          ExtentExpr{1}))};
                  if (extent) {
                    if (extent <= 0) {
                      return Result{1};
                    }
````
- **L265 EN**: Executes a standalone statement or declaration: `bool ok{false};`.
  **L265 CN**: 执行一条独立语句或声明：`bool ok{false};`。
- **L266 EN**: Executes a call or declaration centered on `lbValue{ToInt64`.
  **L266 CN**: 执行以 `lbValue{ToInt64` 为核心的调用或声明。
- **L267 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L267 CN**: 开始 `if` 控制流语句并计算其条件。
- **L268 EN**: Starts a function, method, lambda, or structured scope: `semantics::IsAssumedSizeArray(symbol)) {`.
  **L268 CN**: 开始一个函数、方法、lambda 或结构化作用域：`semantics::IsAssumedSizeArray(symbol)) {`。
- **L269 EN**: Comment explains nearby logic, intent, or metadata: `last dimension of assumed-size dummy array: don't worry`.
  **L269 CN**: 注释说明附近代码的逻辑、意图或元数据：`last dimension of assumed-size dummy array: don't worry`。
- **L270 EN**: Comment explains nearby logic, intent, or metadata: `about handling an empty dimension`.
  **L270 CN**: 注释说明附近代码的逻辑、意图或元数据：`about handling an empty dimension`。
- **L271 EN**: Executes a call or declaration centered on `IsScopeInvariantExpr`.
  **L271 CN**: 执行以 `IsScopeInvariantExpr` 为核心的调用或声明。
- **L272 EN**: Transitions from the previous branch into an `else if` condition.
  **L272 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L273 EN**: Comment explains nearby logic, intent, or metadata: `Lower bound is 1, regardless of extent`.
  **L273 CN**: 注释说明附近代码的逻辑、意图或元数据：`Lower bound is 1, regardless of extent`。
- **L274 EN**: Executes a standalone statement or declaration: `ok = true;`.
  **L274 CN**: 执行一条独立语句或声明：`ok = true;`。
- **L275 EN**: Transitions from the previous branch into an `else if` condition.
  **L275 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L276 EN**: Starts a function, method, lambda, or structured scope: `ubound && ubound->Rank() == 0) {`.
  **L276 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ubound && ubound->Rank() == 0) {`。
- **L277 EN**: Comment explains nearby logic, intent, or metadata: `If we can't prove that the dimension is nonempty,`.
  **L277 CN**: 注释说明附近代码的逻辑、意图或元数据：`If we can't prove that the dimension is nonempty,`。
- **L278 EN**: Comment explains nearby logic, intent, or metadata: `we must be conservative.`.
  **L278 CN**: 注释说明附近代码的逻辑、意图或元数据：`we must be conservative.`。
- **L279 EN**: Comment records a pending task or caution: `TODO: simple symbolic math in expression rewriting to`.
  **L279 CN**: 注释记录待办事项或注意点：`TODO: simple symbolic math in expression rewriting to`。
- **L280 EN**: Comment explains nearby logic, intent, or metadata: `cope with cases like A(J:J)`.
  **L280 CN**: 注释说明附近代码的逻辑、意图或元数据：`cope with cases like A(J:J)`。
- **L281 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L281 CN**: 开始 `if` 控制流语句并计算其条件。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto extent{ToInt64(Fold(*context_,`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto extent{ToInt64(Fold(*context_,`。
- **L283 EN**: Continues the surrounding expression or declaration: `ExtentExpr{*ubound} - ExtentExpr{*lbound} +`.
  **L283 CN**: 继续构造周围的表达式或声明：`ExtentExpr{*ubound} - ExtentExpr{*lbound} +`。
- **L284 EN**: Executes a standalone statement or declaration: `ExtentExpr{1}))};`.
  **L284 CN**: 执行一条独立语句或声明：`ExtentExpr{1}))};`。
- **L285 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L285 CN**: 开始 `if` 控制流语句并计算其条件。
- **L286 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L286 CN**: 开始 `if` 控制流语句并计算其条件。
- **L287 EN**: Returns from the current function with `Result{1}`.
  **L287 CN**: 以 `Result{1}` 从当前函数返回。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。

### Lines 289-312

````cpp
                    ok = true;
                  } else {
                    ok = false;
                  }
                } else {
                  auto ubValue{ToInt64(*ubound)};
                  if (lbValue && ubValue) {
                    if (*lbValue > *ubValue) {
                      return Result{1};
                    }
                    ok = true;
                  } else {
                    ok = false;
                  }
                }
              }
              return ok ? *lbound : Result{};
            } else {
              return *lbound;
            }
          } else {
            return Result{1};
          }
        }
````
- **L289 EN**: Executes a standalone statement or declaration: `ok = true;`.
  **L289 CN**: 执行一条独立语句或声明：`ok = true;`。
- **L290 EN**: Transitions from the previous branch into the alternative path.
  **L290 CN**: 从前一个分支过渡到备选路径。
- **L291 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L291 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Transitions from the previous branch into the alternative path.
  **L293 CN**: 从前一个分支过渡到备选路径。
- **L294 EN**: Executes a call or declaration centered on `ubValue{ToInt64`.
  **L294 CN**: 执行以 `ubValue{ToInt64` 为核心的调用或声明。
- **L295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L296 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L296 CN**: 开始 `if` 控制流语句并计算其条件。
- **L297 EN**: Returns from the current function with `Result{1}`.
  **L297 CN**: 以 `Result{1}` 从当前函数返回。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Executes a standalone statement or declaration: `ok = true;`.
  **L299 CN**: 执行一条独立语句或声明：`ok = true;`。
- **L300 EN**: Transitions from the previous branch into the alternative path.
  **L300 CN**: 从前一个分支过渡到备选路径。
- **L301 EN**: Executes a standalone statement or declaration: `ok = false;`.
  **L301 CN**: 执行一条独立语句或声明：`ok = false;`。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Closes the current lexical scope or compound statement.
  **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Closes the current lexical scope or compound statement.
  **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Returns from the current function with `ok ? *lbound : Result{}`.
  **L305 CN**: 以 `ok ? *lbound : Result{}` 从当前函数返回。
- **L306 EN**: Transitions from the previous branch into the alternative path.
  **L306 CN**: 从前一个分支过渡到备选路径。
- **L307 EN**: Returns from the current function with `*lbound`.
  **L307 CN**: 以 `*lbound` 从当前函数返回。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Transitions from the previous branch into the alternative path.
  **L309 CN**: 从前一个分支过渡到备选路径。
- **L310 EN**: Returns from the current function with `Result{1}`.
  **L310 CN**: 以 `Result{1}` 从当前函数返回。
- **L311 EN**: Closes the current lexical scope or compound statement.
  **L311 CN**: 结束当前词法作用域或复合语句块。
- **L312 EN**: Closes the current lexical scope or compound statement.
  **L312 CN**: 结束当前词法作用域或复合语句块。

### Lines 313-336

````cpp
        if (IsDescriptor(symbol)) {
          return ExtentExpr{DescriptorInquiry{std::move(base),
              DescriptorInquiry::Field::LowerBound, dimension_}};
        }
      }
    } else if (const auto *assoc{
                   symbol.detailsIf<semantics::AssocEntityDetails>()}) {
      if (assoc->IsAssumedSize()) { // RANK(*)
        return Result{1};
      } else if (assoc->IsAssumedRank()) { // RANK DEFAULT
      } else if (assoc->rank()) { // RANK(n)
        const Symbol &resolved{ResolveAssociations(symbol)};
        if (IsDescriptor(resolved) && dimension_ < *assoc->rank()) {
          return ExtentExpr{DescriptorInquiry{std::move(base),
              DescriptorInquiry::Field::LowerBound, dimension_}};
        }
      } else {
        Result exprLowerBound{((*this)(assoc->expr()))};
        if (IsActuallyConstant(exprLowerBound)) {
          return std::move(exprLowerBound);
        } else {
          // If the lower bound of the associated entity is not resolved to a
          // constant expression at the time of the association, it is unsafe
          // to re-evaluate it later in the associate construct. Statements
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{std::move(base),`.
  **L314 CN**: 以 `ExtentExpr{DescriptorInquiry{std::move(base),` 从当前函数返回。
- **L315 EN**: Executes a standalone statement or declaration: `DescriptorInquiry::Field::LowerBound, dimension_}};`.
  **L315 CN**: 执行一条独立语句或声明：`DescriptorInquiry::Field::LowerBound, dimension_}};`。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Closes the current lexical scope or compound statement.
  **L317 CN**: 结束当前词法作用域或复合语句块。
- **L318 EN**: Transitions from the previous branch into an `else if` condition.
  **L318 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L319 EN**: Starts a function, method, lambda, or structured scope: `symbol.detailsIf<semantics::AssocEntityDetails>()}) {`.
  **L319 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.detailsIf<semantics::AssocEntityDetails>()}) {`。
- **L320 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L320 CN**: 开始 `if` 控制流语句并计算其条件。
- **L321 EN**: Returns from the current function with `Result{1}`.
  **L321 CN**: 以 `Result{1}` 从当前函数返回。
- **L322 EN**: Transitions from the previous branch into an `else if` condition.
  **L322 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L323 EN**: Transitions from the previous branch into an `else if` condition.
  **L323 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L324 EN**: Executes a call or declaration centered on `&resolved{ResolveAssociations`.
  **L324 CN**: 执行以 `&resolved{ResolveAssociations` 为核心的调用或声明。
- **L325 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L325 CN**: 开始 `if` 控制流语句并计算其条件。
- **L326 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{std::move(base),`.
  **L326 CN**: 以 `ExtentExpr{DescriptorInquiry{std::move(base),` 从当前函数返回。
- **L327 EN**: Executes a standalone statement or declaration: `DescriptorInquiry::Field::LowerBound, dimension_}};`.
  **L327 CN**: 执行一条独立语句或声明：`DescriptorInquiry::Field::LowerBound, dimension_}};`。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Transitions from the previous branch into the alternative path.
  **L329 CN**: 从前一个分支过渡到备选路径。
- **L330 EN**: Executes a call or declaration centered on `exprLowerBound{`.
  **L330 CN**: 执行以 `exprLowerBound{` 为核心的调用或声明。
- **L331 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L331 CN**: 开始 `if` 控制流语句并计算其条件。
- **L332 EN**: Returns from the current function with `std::move(exprLowerBound)`.
  **L332 CN**: 以 `std::move(exprLowerBound)` 从当前函数返回。
- **L333 EN**: Transitions from the previous branch into the alternative path.
  **L333 CN**: 从前一个分支过渡到备选路径。
- **L334 EN**: Comment explains nearby logic, intent, or metadata: `If the lower bound of the associated entity is not resolved to a`.
  **L334 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the lower bound of the associated entity is not resolved to a`。
- **L335 EN**: Comment explains nearby logic, intent, or metadata: `constant expression at the time of the association, it is unsafe`.
  **L335 CN**: 注释说明附近代码的逻辑、意图或元数据：`constant expression at the time of the association, it is unsafe`。
- **L336 EN**: Comment explains nearby logic, intent, or metadata: `to re-evaluate it later in the associate construct. Statements`.
  **L336 CN**: 注释说明附近代码的逻辑、意图或元数据：`to re-evaluate it later in the associate construct. Statements`。

### Lines 337-360

````cpp
          // in between may have modified its operands value.
          return ExtentExpr{DescriptorInquiry{std::move(base),
              DescriptorInquiry::Field::LowerBound, dimension_}};
        }
      }
    }
    if constexpr (LBOUND_SEMANTICS) {
      return Result{};
    } else {
      return Result{1};
    }
  }

  Result operator()(const Symbol &symbol) const {
    return GetLowerBound(symbol, NamedEntity{symbol});
  }

  Result operator()(const Component &component) const {
    if (component.base().Rank() == 0) {
      return GetLowerBound(
          component.GetLastSymbol(), NamedEntity{common::Clone(component)});
    }
    return Result{1};
  }
````
- **L337 EN**: Comment explains nearby logic, intent, or metadata: `in between may have modified its operands value.`.
  **L337 CN**: 注释说明附近代码的逻辑、意图或元数据：`in between may have modified its operands value.`。
- **L338 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{std::move(base),`.
  **L338 CN**: 以 `ExtentExpr{DescriptorInquiry{std::move(base),` 从当前函数返回。
- **L339 EN**: Executes a standalone statement or declaration: `DescriptorInquiry::Field::LowerBound, dimension_}};`.
  **L339 CN**: 执行一条独立语句或声明：`DescriptorInquiry::Field::LowerBound, dimension_}};`。
- **L340 EN**: Closes the current lexical scope or compound statement.
  **L340 CN**: 结束当前词法作用域或复合语句块。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Continues logic associated with callable symbol `constexpr`.
  **L343 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L344 EN**: Returns from the current function with `Result{}`.
  **L344 CN**: 以 `Result{}` 从当前函数返回。
- **L345 EN**: Transitions from the previous branch into the alternative path.
  **L345 CN**: 从前一个分支过渡到备选路径。
- **L346 EN**: Returns from the current function with `Result{1}`.
  **L346 CN**: 以 `Result{1}` 从当前函数返回。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Starts a function, method, lambda, or structured scope: `Result operator()(const Symbol &symbol) const {`.
  **L350 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result operator()(const Symbol &symbol) const {`。
- **L351 EN**: Returns from the current function with `GetLowerBound(symbol, NamedEntity{symbol})`.
  **L351 CN**: 以 `GetLowerBound(symbol, NamedEntity{symbol})` 从当前函数返回。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L354 EN**: Starts a function, method, lambda, or structured scope: `Result operator()(const Component &component) const {`.
  **L354 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result operator()(const Component &component) const {`。
- **L355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L356 EN**: Returns from the current function with `GetLowerBound(`.
  **L356 CN**: 以 `GetLowerBound(` 从当前函数返回。
- **L357 EN**: Executes a call or declaration centered on `component.GetLastSymbol`.
  **L357 CN**: 执行以 `component.GetLastSymbol` 为核心的调用或声明。
- **L358 EN**: Closes the current lexical scope or compound statement.
  **L358 CN**: 结束当前词法作用域或复合语句块。
- **L359 EN**: Returns from the current function with `Result{1}`.
  **L359 CN**: 以 `Result{1}` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````cpp

  template <typename T> Result operator()(const Expr<T> &expr) const {
    if (const Symbol * whole{UnwrapWholeSymbolOrComponentDataRef(expr)}) {
      return (*this)(*whole);
    } else if constexpr (common::HasMember<Constant<T>, decltype(expr.u)>) {
      if (const auto *con{std::get_if<Constant<T>>(&expr.u)}) {
        ConstantSubscripts lb{con->lbounds()};
        if (dimension_ < GetRank(lb)) {
          return Result{lb[dimension_]};
        }
      } else { // operation
        return Result{1};
      }
    } else {
      return (*this)(expr.u);
    }
    if constexpr (LBOUND_SEMANTICS) {
      return Result{};
    } else {
      return Result{1};
    }
  }

private:
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L362 EN**: Introduces template parameters or specialization context: `template <typename T> Result operator()(const Expr<T> &expr) const {`.
  **L362 CN**: 为后续声明引入模板参数或特化上下文：`template <typename T> Result operator()(const Expr<T> &expr) const {`。
- **L363 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L363 CN**: 开始 `if` 控制流语句并计算其条件。
- **L364 EN**: Returns from the current function with `(*this)(*whole)`.
  **L364 CN**: 以 `(*this)(*whole)` 从当前函数返回。
- **L365 EN**: Transitions from the previous branch into an `else if` condition.
  **L365 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L366 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L366 CN**: 开始 `if` 控制流语句并计算其条件。
- **L367 EN**: Executes a call or declaration centered on `lb{con->lbounds`.
  **L367 CN**: 执行以 `lb{con->lbounds` 为核心的调用或声明。
- **L368 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L368 CN**: 开始 `if` 控制流语句并计算其条件。
- **L369 EN**: Returns from the current function with `Result{lb[dimension_]}`.
  **L369 CN**: 以 `Result{lb[dimension_]}` 从当前函数返回。
- **L370 EN**: Closes the current lexical scope or compound statement.
  **L370 CN**: 结束当前词法作用域或复合语句块。
- **L371 EN**: Transitions from the previous branch into the alternative path.
  **L371 CN**: 从前一个分支过渡到备选路径。
- **L372 EN**: Returns from the current function with `Result{1}`.
  **L372 CN**: 以 `Result{1}` 从当前函数返回。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Transitions from the previous branch into the alternative path.
  **L374 CN**: 从前一个分支过渡到备选路径。
- **L375 EN**: Returns from the current function with `(*this)(expr.u)`.
  **L375 CN**: 以 `(*this)(expr.u)` 从当前函数返回。
- **L376 EN**: Closes the current lexical scope or compound statement.
  **L376 CN**: 结束当前词法作用域或复合语句块。
- **L377 EN**: Continues logic associated with callable symbol `constexpr`.
  **L377 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L378 EN**: Returns from the current function with `Result{}`.
  **L378 CN**: 以 `Result{}` 从当前函数返回。
- **L379 EN**: Transitions from the previous branch into the alternative path.
  **L379 CN**: 从前一个分支过渡到备选路径。
- **L380 EN**: Returns from the current function with `Result{1}`.
  **L380 CN**: 以 `Result{1}` 从当前函数返回。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L384 EN**: Sets the following members to `private` access.
  **L384 CN**: 将后续成员的访问级别设为 `private`。

### Lines 385-408

````cpp
  int dimension_; // zero-based
  FoldingContext *context_{nullptr};
  bool invariantOnly_{false};
};

ExtentExpr GetRawLowerBound(
    const NamedEntity &base, int dimension, bool invariantOnly) {
  return GetLowerBoundHelper<ExtentExpr, false>{
      dimension, nullptr, invariantOnly}(base);
}

ExtentExpr GetRawLowerBound(FoldingContext &context, const NamedEntity &base,
    int dimension, bool invariantOnly) {
  return Fold(context,
      GetLowerBoundHelper<ExtentExpr, false>{
          dimension, &context, invariantOnly}(base));
}

MaybeExtentExpr GetLBOUND(
    const NamedEntity &base, int dimension, bool invariantOnly) {
  return GetLowerBoundHelper<MaybeExtentExpr, true>{
      dimension, nullptr, invariantOnly}(base);
}

````
- **L385 EN**: Continues the surrounding expression or declaration: `int dimension_; // zero-based`.
  **L385 CN**: 继续构造周围的表达式或声明：`int dimension_; // zero-based`。
- **L386 EN**: Executes a standalone statement or declaration: `FoldingContext *context_{nullptr};`.
  **L386 CN**: 执行一条独立语句或声明：`FoldingContext *context_{nullptr};`。
- **L387 EN**: Executes a standalone statement or declaration: `bool invariantOnly_{false};`.
  **L387 CN**: 执行一条独立语句或声明：`bool invariantOnly_{false};`。
- **L388 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L388 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L390 EN**: Continues logic associated with callable symbol `GetRawLowerBound`.
  **L390 CN**: 继续与可调用符号 `GetRawLowerBound` 相关的逻辑。
- **L391 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L391 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L392 EN**: Returns from the current function with `GetLowerBoundHelper<ExtentExpr, false>{`.
  **L392 CN**: 以 `GetLowerBoundHelper<ExtentExpr, false>{` 从当前函数返回。
- **L393 EN**: Executes a call or declaration centered on `invariantOnly}`.
  **L393 CN**: 执行以 `invariantOnly}` 为核心的调用或声明。
- **L394 EN**: Closes the current lexical scope or compound statement.
  **L394 CN**: 结束当前词法作用域或复合语句块。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExtentExpr GetRawLowerBound(FoldingContext &context, const NamedEntity &base,`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExtentExpr GetRawLowerBound(FoldingContext &context, const NamedEntity &base,`。
- **L397 EN**: Continues the surrounding expression or declaration: `int dimension, bool invariantOnly) {`.
  **L397 CN**: 继续构造周围的表达式或声明：`int dimension, bool invariantOnly) {`。
- **L398 EN**: Returns from the current function with `Fold(context,`.
  **L398 CN**: 以 `Fold(context,` 从当前函数返回。
- **L399 EN**: Continues the surrounding expression or declaration: `GetLowerBoundHelper<ExtentExpr, false>{`.
  **L399 CN**: 继续构造周围的表达式或声明：`GetLowerBoundHelper<ExtentExpr, false>{`。
- **L400 EN**: Executes a call or declaration centered on `invariantOnly}`.
  **L400 CN**: 执行以 `invariantOnly}` 为核心的调用或声明。
- **L401 EN**: Closes the current lexical scope or compound statement.
  **L401 CN**: 结束当前词法作用域或复合语句块。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L403 EN**: Continues logic associated with callable symbol `GetLBOUND`.
  **L403 CN**: 继续与可调用符号 `GetLBOUND` 相关的逻辑。
- **L404 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L404 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L405 EN**: Returns from the current function with `GetLowerBoundHelper<MaybeExtentExpr, true>{`.
  **L405 CN**: 以 `GetLowerBoundHelper<MaybeExtentExpr, true>{` 从当前函数返回。
- **L406 EN**: Executes a call or declaration centered on `invariantOnly}`.
  **L406 CN**: 执行以 `invariantOnly}` 为核心的调用或声明。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 409-432

````cpp
MaybeExtentExpr GetLBOUND(FoldingContext &context, const NamedEntity &base,
    int dimension, bool invariantOnly) {
  return Fold(context,
      GetLowerBoundHelper<MaybeExtentExpr, true>{
          dimension, &context, invariantOnly}(base));
}

Shape GetRawLowerBounds(const NamedEntity &base, bool invariantOnly) {
  Shape result;
  int rank{base.Rank()};
  for (int dim{0}; dim < rank; ++dim) {
    result.emplace_back(GetRawLowerBound(base, dim, invariantOnly));
  }
  return result;
}

Shape GetRawLowerBounds(
    FoldingContext &context, const NamedEntity &base, bool invariantOnly) {
  Shape result;
  int rank{base.Rank()};
  for (int dim{0}; dim < rank; ++dim) {
    result.emplace_back(GetRawLowerBound(context, base, dim, invariantOnly));
  }
  return result;
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetLBOUND(FoldingContext &context, const NamedEntity &base,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetLBOUND(FoldingContext &context, const NamedEntity &base,`。
- **L410 EN**: Continues the surrounding expression or declaration: `int dimension, bool invariantOnly) {`.
  **L410 CN**: 继续构造周围的表达式或声明：`int dimension, bool invariantOnly) {`。
- **L411 EN**: Returns from the current function with `Fold(context,`.
  **L411 CN**: 以 `Fold(context,` 从当前函数返回。
- **L412 EN**: Continues the surrounding expression or declaration: `GetLowerBoundHelper<MaybeExtentExpr, true>{`.
  **L412 CN**: 继续构造周围的表达式或声明：`GetLowerBoundHelper<MaybeExtentExpr, true>{`。
- **L413 EN**: Executes a call or declaration centered on `invariantOnly}`.
  **L413 CN**: 执行以 `invariantOnly}` 为核心的调用或声明。
- **L414 EN**: Closes the current lexical scope or compound statement.
  **L414 CN**: 结束当前词法作用域或复合语句块。
- **L415 EN**: Blank line separating nearby declarations or logic blocks.
  **L415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L416 EN**: Starts a function, method, lambda, or structured scope: `Shape GetRawLowerBounds(const NamedEntity &base, bool invariantOnly) {`.
  **L416 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetRawLowerBounds(const NamedEntity &base, bool invariantOnly) {`。
- **L417 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L417 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L418 EN**: Executes a call or declaration centered on `rank{base.Rank`.
  **L418 CN**: 执行以 `rank{base.Rank` 为核心的调用或声明。
- **L419 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L419 CN**: 开始 `for` 控制流语句并计算其条件。
- **L420 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L420 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Returns from the current function with `result`.
  **L422 CN**: 以 `result` 从当前函数返回。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Continues logic associated with callable symbol `GetRawLowerBounds`.
  **L425 CN**: 继续与可调用符号 `GetRawLowerBounds` 相关的逻辑。
- **L426 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`.
  **L426 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`。
- **L427 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L427 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L428 EN**: Executes a call or declaration centered on `rank{base.Rank`.
  **L428 CN**: 执行以 `rank{base.Rank` 为核心的调用或声明。
- **L429 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L429 CN**: 开始 `for` 控制流语句并计算其条件。
- **L430 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L430 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L431 EN**: Closes the current lexical scope or compound statement.
  **L431 CN**: 结束当前词法作用域或复合语句块。
- **L432 EN**: Returns from the current function with `result`.
  **L432 CN**: 以 `result` 从当前函数返回。

### Lines 433-456

````cpp
}

Shape GetLBOUNDs(const NamedEntity &base, bool invariantOnly) {
  Shape result;
  int rank{base.Rank()};
  for (int dim{0}; dim < rank; ++dim) {
    result.emplace_back(GetLBOUND(base, dim, invariantOnly));
  }
  return result;
}

Shape GetLBOUNDs(
    FoldingContext &context, const NamedEntity &base, bool invariantOnly) {
  Shape result;
  int rank{base.Rank()};
  for (int dim{0}; dim < rank; ++dim) {
    result.emplace_back(GetLBOUND(context, base, dim, invariantOnly));
  }
  return result;
}

// If the upper and lower bounds are constant, return a constant expression for
// the extent.  In particular, if the upper bound is less than the lower bound,
// return zero.
````
- **L433 EN**: Closes the current lexical scope or compound statement.
  **L433 CN**: 结束当前词法作用域或复合语句块。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L435 EN**: Starts a function, method, lambda, or structured scope: `Shape GetLBOUNDs(const NamedEntity &base, bool invariantOnly) {`.
  **L435 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetLBOUNDs(const NamedEntity &base, bool invariantOnly) {`。
- **L436 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L436 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L437 EN**: Executes a call or declaration centered on `rank{base.Rank`.
  **L437 CN**: 执行以 `rank{base.Rank` 为核心的调用或声明。
- **L438 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L438 CN**: 开始 `for` 控制流语句并计算其条件。
- **L439 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L439 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Returns from the current function with `result`.
  **L441 CN**: 以 `result` 从当前函数返回。
- **L442 EN**: Closes the current lexical scope or compound statement.
  **L442 CN**: 结束当前词法作用域或复合语句块。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Continues logic associated with callable symbol `GetLBOUNDs`.
  **L444 CN**: 继续与可调用符号 `GetLBOUNDs` 相关的逻辑。
- **L445 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`.
  **L445 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`。
- **L446 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L446 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L447 EN**: Executes a call or declaration centered on `rank{base.Rank`.
  **L447 CN**: 执行以 `rank{base.Rank` 为核心的调用或声明。
- **L448 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L448 CN**: 开始 `for` 控制流语句并计算其条件。
- **L449 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L449 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L450 EN**: Closes the current lexical scope or compound statement.
  **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Returns from the current function with `result`.
  **L451 CN**: 以 `result` 从当前函数返回。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L454 EN**: Comment explains nearby logic, intent, or metadata: `If the upper and lower bounds are constant, return a constant expression for`.
  **L454 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the upper and lower bounds are constant, return a constant expression for`。
- **L455 EN**: Comment explains nearby logic, intent, or metadata: `the extent.  In particular, if the upper bound is less than the lower bound,`.
  **L455 CN**: 注释说明附近代码的逻辑、意图或元数据：`the extent.  In particular, if the upper bound is less than the lower bound,`。
- **L456 EN**: Comment explains nearby logic, intent, or metadata: `return zero.`.
  **L456 CN**: 注释说明附近代码的逻辑、意图或元数据：`return zero.`。

### Lines 457-480

````cpp
static MaybeExtentExpr GetNonNegativeExtent(
    const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {
  const auto &ubound{shapeSpec.ubound().GetExplicit()};
  const auto &lbound{shapeSpec.lbound().GetExplicit()};
  std::optional<ConstantSubscript> uval{ToInt64(ubound)};
  std::optional<ConstantSubscript> lval{ToInt64(lbound)};
  if (uval && lval) {
    if (*uval < *lval) {
      return ExtentExpr{0};
    } else {
      return ExtentExpr{*uval - *lval + 1};
    }
  } else if (lbound && ubound && lbound->Rank() == 0 && ubound->Rank() == 0 &&
      (!invariantOnly ||
          (IsScopeInvariantExpr(*lbound) && IsScopeInvariantExpr(*ubound)))) {
    // Apply effective IDIM (MAX calculation with 0) so thet the
    // result is never negative
    if (lval.value_or(0) == 1) {
      return ExtentExpr{Extremum<SubscriptInteger>{
          Ordering::Greater, ExtentExpr{0}, common::Clone(*ubound)}};
    } else {
      return ExtentExpr{
          Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},
              common::Clone(*ubound) - common::Clone(*lbound) + ExtentExpr{1}}};
````
- **L457 EN**: Continues logic associated with callable symbol `GetNonNegativeExtent`.
  **L457 CN**: 继续与可调用符号 `GetNonNegativeExtent` 相关的逻辑。
- **L458 EN**: Continues the surrounding expression or declaration: `const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {`.
  **L458 CN**: 继续构造周围的表达式或声明：`const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {`。
- **L459 EN**: Executes a call or declaration centered on `&ubound{shapeSpec.ubound`.
  **L459 CN**: 执行以 `&ubound{shapeSpec.ubound` 为核心的调用或声明。
- **L460 EN**: Executes a call or declaration centered on `&lbound{shapeSpec.lbound`.
  **L460 CN**: 执行以 `&lbound{shapeSpec.lbound` 为核心的调用或声明。
- **L461 EN**: Executes a call or declaration centered on `uval{ToInt64`.
  **L461 CN**: 执行以 `uval{ToInt64` 为核心的调用或声明。
- **L462 EN**: Executes a call or declaration centered on `lval{ToInt64`.
  **L462 CN**: 执行以 `lval{ToInt64` 为核心的调用或声明。
- **L463 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L463 CN**: 开始 `if` 控制流语句并计算其条件。
- **L464 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L464 CN**: 开始 `if` 控制流语句并计算其条件。
- **L465 EN**: Returns from the current function with `ExtentExpr{0}`.
  **L465 CN**: 以 `ExtentExpr{0}` 从当前函数返回。
- **L466 EN**: Transitions from the previous branch into the alternative path.
  **L466 CN**: 从前一个分支过渡到备选路径。
- **L467 EN**: Returns from the current function with `ExtentExpr{*uval - *lval + 1}`.
  **L467 CN**: 以 `ExtentExpr{*uval - *lval + 1}` 从当前函数返回。
- **L468 EN**: Closes the current lexical scope or compound statement.
  **L468 CN**: 结束当前词法作用域或复合语句块。
- **L469 EN**: Transitions from the previous branch into an `else if` condition.
  **L469 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L470 EN**: Continues the surrounding expression or declaration: `(!invariantOnly ||`.
  **L470 CN**: 继续构造周围的表达式或声明：`(!invariantOnly ||`。
- **L471 EN**: Starts a function, method, lambda, or structured scope: `(IsScopeInvariantExpr(*lbound) && IsScopeInvariantExpr(*ubound)))) {`.
  **L471 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(IsScopeInvariantExpr(*lbound) && IsScopeInvariantExpr(*ubound)))) {`。
- **L472 EN**: Comment explains nearby logic, intent, or metadata: `Apply effective IDIM (MAX calculation with 0) so thet the`.
  **L472 CN**: 注释说明附近代码的逻辑、意图或元数据：`Apply effective IDIM (MAX calculation with 0) so thet the`。
- **L473 EN**: Comment explains nearby logic, intent, or metadata: `result is never negative`.
  **L473 CN**: 注释说明附近代码的逻辑、意图或元数据：`result is never negative`。
- **L474 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L474 CN**: 开始 `if` 控制流语句并计算其条件。
- **L475 EN**: Returns from the current function with `ExtentExpr{Extremum<SubscriptInteger>{`.
  **L475 CN**: 以 `ExtentExpr{Extremum<SubscriptInteger>{` 从当前函数返回。
- **L476 EN**: Executes a call or declaration centered on `common::Clone`.
  **L476 CN**: 执行以 `common::Clone` 为核心的调用或声明。
- **L477 EN**: Transitions from the previous branch into the alternative path.
  **L477 CN**: 从前一个分支过渡到备选路径。
- **L478 EN**: Returns from the current function with `ExtentExpr{`.
  **L478 CN**: 以 `ExtentExpr{` 从当前函数返回。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},`。
- **L480 EN**: Executes a call or declaration centered on `common::Clone`.
  **L480 CN**: 执行以 `common::Clone` 为核心的调用或声明。

### Lines 481-504

````cpp
    }
  } else {
    return std::nullopt;
  }
}

static MaybeExtentExpr GetAssociatedExtent(
    const Symbol &symbol, int dimension) {
  if (const auto *assoc{symbol.detailsIf<semantics::AssocEntityDetails>()};
      assoc && !assoc->rank()) { // not SELECT RANK case
    if (auto shape{GetShape(GetFoldingContextFrom(symbol), assoc->expr())};
        shape && dimension < static_cast<int>(shape->size())) {
      if (auto &extent{shape->at(dimension)};
          // Don't return a non-constant extent, as the variables that
          // determine the shape of the selector's expression may change
          // during execution of the construct.
          extent && IsActuallyConstant(*extent)) {
        return std::move(extent);
      }
    }
  }
  return ExtentExpr{DescriptorInquiry{
      NamedEntity{symbol}, DescriptorInquiry::Field::Extent, dimension}};
}
````
- **L481 EN**: Closes the current lexical scope or compound statement.
  **L481 CN**: 结束当前词法作用域或复合语句块。
- **L482 EN**: Transitions from the previous branch into the alternative path.
  **L482 CN**: 从前一个分支过渡到备选路径。
- **L483 EN**: Returns from the current function with `std::nullopt`.
  **L483 CN**: 以 `std::nullopt` 从当前函数返回。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Continues logic associated with callable symbol `GetAssociatedExtent`.
  **L487 CN**: 继续与可调用符号 `GetAssociatedExtent` 相关的逻辑。
- **L488 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol, int dimension) {`.
  **L488 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol, int dimension) {`。
- **L489 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L489 CN**: 开始 `if` 控制流语句并计算其条件。
- **L490 EN**: Continues logic associated with callable symbol `rank`.
  **L490 CN**: 继续与可调用符号 `rank` 相关的逻辑。
- **L491 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L491 CN**: 开始 `if` 控制流语句并计算其条件。
- **L492 EN**: Starts a function, method, lambda, or structured scope: `shape && dimension < static_cast<int>(shape->size())) {`.
  **L492 CN**: 开始一个函数、方法、lambda 或结构化作用域：`shape && dimension < static_cast<int>(shape->size())) {`。
- **L493 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L493 CN**: 开始 `if` 控制流语句并计算其条件。
- **L494 EN**: Comment explains nearby logic, intent, or metadata: `Don't return a non-constant extent, as the variables that`.
  **L494 CN**: 注释说明附近代码的逻辑、意图或元数据：`Don't return a non-constant extent, as the variables that`。
- **L495 EN**: Comment explains nearby logic, intent, or metadata: `determine the shape of the selector's expression may change`.
  **L495 CN**: 注释说明附近代码的逻辑、意图或元数据：`determine the shape of the selector's expression may change`。
- **L496 EN**: Comment explains nearby logic, intent, or metadata: `during execution of the construct.`.
  **L496 CN**: 注释说明附近代码的逻辑、意图或元数据：`during execution of the construct.`。
- **L497 EN**: Starts a function, method, lambda, or structured scope: `extent && IsActuallyConstant(*extent)) {`.
  **L497 CN**: 开始一个函数、方法、lambda 或结构化作用域：`extent && IsActuallyConstant(*extent)) {`。
- **L498 EN**: Returns from the current function with `std::move(extent)`.
  **L498 CN**: 以 `std::move(extent)` 从当前函数返回。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Closes the current lexical scope or compound statement.
  **L500 CN**: 结束当前词法作用域或复合语句块。
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{`.
  **L502 CN**: 以 `ExtentExpr{DescriptorInquiry{` 从当前函数返回。
- **L503 EN**: Executes a standalone statement or declaration: `NamedEntity{symbol}, DescriptorInquiry::Field::Extent, dimension}};`.
  **L503 CN**: 执行一条独立语句或声明：`NamedEntity{symbol}, DescriptorInquiry::Field::Extent, dimension}};`。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。

### Lines 505-528

````cpp

MaybeExtentExpr GetExtent(
    const NamedEntity &base, int dimension, bool invariantOnly) {
  CHECK(dimension >= 0);
  const Symbol &last{base.GetLastSymbol()};
  const Symbol &symbol{ResolveAssociations(last)};
  if (const auto *assoc{last.detailsIf<semantics::AssocEntityDetails>()}) {
    if (assoc->IsAssumedSize() || assoc->IsAssumedRank()) { // RANK(*)/DEFAULT
      return std::nullopt;
    } else if (assoc->rank()) { // RANK(n)
      if (semantics::IsDescriptor(symbol) && dimension < *assoc->rank()) {
        return ExtentExpr{DescriptorInquiry{
            NamedEntity{base}, DescriptorInquiry::Field::Extent, dimension}};
      } else {
        return std::nullopt;
      }
    } else {
      return GetAssociatedExtent(last, dimension);
    }
  }
  if (const auto *details{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
    if (IsImpliedShape(symbol) && details->init()) {
      if (auto shape{
              GetShape(GetFoldingContextFrom(symbol), symbol, invariantOnly)}) {
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L506 EN**: Continues logic associated with callable symbol `GetExtent`.
  **L506 CN**: 继续与可调用符号 `GetExtent` 相关的逻辑。
- **L507 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L507 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L508 EN**: Executes a call or declaration centered on `CHECK`.
  **L508 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L509 EN**: Executes a call or declaration centered on `&last{base.GetLastSymbol`.
  **L509 CN**: 执行以 `&last{base.GetLastSymbol` 为核心的调用或声明。
- **L510 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L510 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L511 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L511 CN**: 开始 `if` 控制流语句并计算其条件。
- **L512 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L512 CN**: 开始 `if` 控制流语句并计算其条件。
- **L513 EN**: Returns from the current function with `std::nullopt`.
  **L513 CN**: 以 `std::nullopt` 从当前函数返回。
- **L514 EN**: Transitions from the previous branch into an `else if` condition.
  **L514 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L515 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L515 CN**: 开始 `if` 控制流语句并计算其条件。
- **L516 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{`.
  **L516 CN**: 以 `ExtentExpr{DescriptorInquiry{` 从当前函数返回。
- **L517 EN**: Executes a standalone statement or declaration: `NamedEntity{base}, DescriptorInquiry::Field::Extent, dimension}};`.
  **L517 CN**: 执行一条独立语句或声明：`NamedEntity{base}, DescriptorInquiry::Field::Extent, dimension}};`。
- **L518 EN**: Transitions from the previous branch into the alternative path.
  **L518 CN**: 从前一个分支过渡到备选路径。
- **L519 EN**: Returns from the current function with `std::nullopt`.
  **L519 CN**: 以 `std::nullopt` 从当前函数返回。
- **L520 EN**: Closes the current lexical scope or compound statement.
  **L520 CN**: 结束当前词法作用域或复合语句块。
- **L521 EN**: Transitions from the previous branch into the alternative path.
  **L521 CN**: 从前一个分支过渡到备选路径。
- **L522 EN**: Returns from the current function with `GetAssociatedExtent(last, dimension)`.
  **L522 CN**: 以 `GetAssociatedExtent(last, dimension)` 从当前函数返回。
- **L523 EN**: Closes the current lexical scope or compound statement.
  **L523 CN**: 结束当前词法作用域或复合语句块。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L525 CN**: 开始 `if` 控制流语句并计算其条件。
- **L526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L527 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L527 CN**: 开始 `if` 控制流语句并计算其条件。
- **L528 EN**: Starts a function, method, lambda, or structured scope: `GetShape(GetFoldingContextFrom(symbol), symbol, invariantOnly)}) {`.
  **L528 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetShape(GetFoldingContextFrom(symbol), symbol, invariantOnly)}) {`。

### Lines 529-552

````cpp
        if (dimension < static_cast<int>(shape->size())) {
          return std::move(shape->at(dimension));
        }
      }
    } else {
      int j{0};
      for (const auto &shapeSpec : details->shape()) {
        if (j++ == dimension) {
          if (auto extent{GetNonNegativeExtent(shapeSpec, invariantOnly)}) {
            return extent;
          } else if (semantics::IsAssumedSizeArray(symbol) &&
              j == symbol.Rank()) {
            break;
          } else if (semantics::IsDescriptor(symbol)) {
            return ExtentExpr{DescriptorInquiry{NamedEntity{base},
                DescriptorInquiry::Field::Extent, dimension}};
          } else {
            break;
          }
        }
      }
    }
  }
  return std::nullopt;
````
- **L529 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L529 CN**: 开始 `if` 控制流语句并计算其条件。
- **L530 EN**: Returns from the current function with `std::move(shape->at(dimension))`.
  **L530 CN**: 以 `std::move(shape->at(dimension))` 从当前函数返回。
- **L531 EN**: Closes the current lexical scope or compound statement.
  **L531 CN**: 结束当前词法作用域或复合语句块。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Transitions from the previous branch into the alternative path.
  **L533 CN**: 从前一个分支过渡到备选路径。
- **L534 EN**: Executes a standalone statement or declaration: `int j{0};`.
  **L534 CN**: 执行一条独立语句或声明：`int j{0};`。
- **L535 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L535 CN**: 开始 `for` 控制流语句并计算其条件。
- **L536 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L536 CN**: 开始 `if` 控制流语句并计算其条件。
- **L537 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L537 CN**: 开始 `if` 控制流语句并计算其条件。
- **L538 EN**: Returns from the current function with `extent`.
  **L538 CN**: 以 `extent` 从当前函数返回。
- **L539 EN**: Transitions from the previous branch into an `else if` condition.
  **L539 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L540 EN**: Starts a function, method, lambda, or structured scope: `j == symbol.Rank()) {`.
  **L540 CN**: 开始一个函数、方法、lambda 或结构化作用域：`j == symbol.Rank()) {`。
- **L541 EN**: Exits the nearest loop or switch statement.
  **L541 CN**: 退出最近的循环或 switch 语句。
- **L542 EN**: Transitions from the previous branch into an `else if` condition.
  **L542 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L543 EN**: Returns from the current function with `ExtentExpr{DescriptorInquiry{NamedEntity{base},`.
  **L543 CN**: 以 `ExtentExpr{DescriptorInquiry{NamedEntity{base},` 从当前函数返回。
- **L544 EN**: Executes a standalone statement or declaration: `DescriptorInquiry::Field::Extent, dimension}};`.
  **L544 CN**: 执行一条独立语句或声明：`DescriptorInquiry::Field::Extent, dimension}};`。
- **L545 EN**: Transitions from the previous branch into the alternative path.
  **L545 CN**: 从前一个分支过渡到备选路径。
- **L546 EN**: Exits the nearest loop or switch statement.
  **L546 CN**: 退出最近的循环或 switch 语句。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Closes the current lexical scope or compound statement.
  **L549 CN**: 结束当前词法作用域或复合语句块。
- **L550 EN**: Closes the current lexical scope or compound statement.
  **L550 CN**: 结束当前词法作用域或复合语句块。
- **L551 EN**: Closes the current lexical scope or compound statement.
  **L551 CN**: 结束当前词法作用域或复合语句块。
- **L552 EN**: Returns from the current function with `std::nullopt`.
  **L552 CN**: 以 `std::nullopt` 从当前函数返回。

### Lines 553-576

````cpp
}

MaybeExtentExpr GetExtent(FoldingContext &context, const NamedEntity &base,
    int dimension, bool invariantOnly) {
  return Fold(context, GetExtent(base, dimension, invariantOnly));
}

MaybeExtentExpr GetExtent(const Subscript &subscript, const NamedEntity &base,
    int dimension, bool invariantOnly) {
  return common::visit(
      common::visitors{
          [&](const Triplet &triplet) -> MaybeExtentExpr {
            MaybeExtentExpr upper{triplet.upper()};
            if (!upper) {
              upper = GetUBOUND(base, dimension, invariantOnly);
            }
            MaybeExtentExpr lower{triplet.lower()};
            if (!lower) {
              lower = GetLBOUND(base, dimension, invariantOnly);
            }
            return CountTrips(std::move(lower), std::move(upper),
                MaybeExtentExpr{triplet.stride()});
          },
          [&](const IndirectSubscriptIntegerExpr &subs) -> MaybeExtentExpr {
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetExtent(FoldingContext &context, const NamedEntity &base,`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetExtent(FoldingContext &context, const NamedEntity &base,`。
- **L556 EN**: Continues the surrounding expression or declaration: `int dimension, bool invariantOnly) {`.
  **L556 CN**: 继续构造周围的表达式或声明：`int dimension, bool invariantOnly) {`。
- **L557 EN**: Returns from the current function with `Fold(context, GetExtent(base, dimension, invariantOnly))`.
  **L557 CN**: 以 `Fold(context, GetExtent(base, dimension, invariantOnly))` 从当前函数返回。
- **L558 EN**: Closes the current lexical scope or compound statement.
  **L558 CN**: 结束当前词法作用域或复合语句块。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetExtent(const Subscript &subscript, const NamedEntity &base,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetExtent(const Subscript &subscript, const NamedEntity &base,`。
- **L561 EN**: Continues the surrounding expression or declaration: `int dimension, bool invariantOnly) {`.
  **L561 CN**: 继续构造周围的表达式或声明：`int dimension, bool invariantOnly) {`。
- **L562 EN**: Returns from the current function with `common::visit(`.
  **L562 CN**: 以 `common::visit(` 从当前函数返回。
- **L563 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L563 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L564 EN**: Starts a function, method, lambda, or structured scope: `[&](const Triplet &triplet) -> MaybeExtentExpr {`.
  **L564 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const Triplet &triplet) -> MaybeExtentExpr {`。
- **L565 EN**: Executes a call or declaration centered on `upper{triplet.upper`.
  **L565 CN**: 执行以 `upper{triplet.upper` 为核心的调用或声明。
- **L566 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L566 CN**: 开始 `if` 控制流语句并计算其条件。
- **L567 EN**: Executes a call or declaration centered on `GetUBOUND`.
  **L567 CN**: 执行以 `GetUBOUND` 为核心的调用或声明。
- **L568 EN**: Closes the current lexical scope or compound statement.
  **L568 CN**: 结束当前词法作用域或复合语句块。
- **L569 EN**: Executes a call or declaration centered on `lower{triplet.lower`.
  **L569 CN**: 执行以 `lower{triplet.lower` 为核心的调用或声明。
- **L570 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L570 CN**: 开始 `if` 控制流语句并计算其条件。
- **L571 EN**: Executes a call or declaration centered on `GetLBOUND`.
  **L571 CN**: 执行以 `GetLBOUND` 为核心的调用或声明。
- **L572 EN**: Closes the current lexical scope or compound statement.
  **L572 CN**: 结束当前词法作用域或复合语句块。
- **L573 EN**: Returns from the current function with `CountTrips(std::move(lower), std::move(upper),`.
  **L573 CN**: 以 `CountTrips(std::move(lower), std::move(upper),` 从当前函数返回。
- **L574 EN**: Executes a call or declaration centered on `MaybeExtentExpr{triplet.stride`.
  **L574 CN**: 执行以 `MaybeExtentExpr{triplet.stride` 为核心的调用或声明。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L576 EN**: Starts a function, method, lambda, or structured scope: `[&](const IndirectSubscriptIntegerExpr &subs) -> MaybeExtentExpr {`.
  **L576 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const IndirectSubscriptIntegerExpr &subs) -> MaybeExtentExpr {`。

### Lines 577-600

````cpp
            if (auto shape{GetShape(
                    GetFoldingContextFrom(base.GetLastSymbol()), subs.value())};
                shape && GetRank(*shape) == 1) {
              // vector-valued subscript
              return std::move(shape->at(0));
            } else {
              return std::nullopt;
            }
          },
      },
      subscript.u);
}

MaybeExtentExpr GetExtent(FoldingContext &context, const Subscript &subscript,
    const NamedEntity &base, int dimension, bool invariantOnly) {
  return Fold(context, GetExtent(subscript, base, dimension, invariantOnly));
}

MaybeExtentExpr ComputeUpperBound(
    ExtentExpr &&lower, MaybeExtentExpr &&extent) {
  if (extent) {
    if (ToInt64(lower).value_or(0) == 1) {
      return std::move(*extent);
    } else {
````
- **L577 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L577 CN**: 开始 `if` 控制流语句并计算其条件。
- **L578 EN**: Executes a call or declaration centered on `GetFoldingContextFrom`.
  **L578 CN**: 执行以 `GetFoldingContextFrom` 为核心的调用或声明。
- **L579 EN**: Starts a function, method, lambda, or structured scope: `shape && GetRank(*shape) == 1) {`.
  **L579 CN**: 开始一个函数、方法、lambda 或结构化作用域：`shape && GetRank(*shape) == 1) {`。
- **L580 EN**: Comment explains nearby logic, intent, or metadata: `vector-valued subscript`.
  **L580 CN**: 注释说明附近代码的逻辑、意图或元数据：`vector-valued subscript`。
- **L581 EN**: Returns from the current function with `std::move(shape->at(0))`.
  **L581 CN**: 以 `std::move(shape->at(0))` 从当前函数返回。
- **L582 EN**: Transitions from the previous branch into the alternative path.
  **L582 CN**: 从前一个分支过渡到备选路径。
- **L583 EN**: Returns from the current function with `std::nullopt`.
  **L583 CN**: 以 `std::nullopt` 从当前函数返回。
- **L584 EN**: Closes the current lexical scope or compound statement.
  **L584 CN**: 结束当前词法作用域或复合语句块。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L587 EN**: Executes a standalone statement or declaration: `subscript.u);`.
  **L587 CN**: 执行一条独立语句或声明：`subscript.u);`。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetExtent(FoldingContext &context, const Subscript &subscript,`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetExtent(FoldingContext &context, const Subscript &subscript,`。
- **L591 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L591 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L592 EN**: Returns from the current function with `Fold(context, GetExtent(subscript, base, dimension, invariantOnly))`.
  **L592 CN**: 以 `Fold(context, GetExtent(subscript, base, dimension, invariantOnly))` 从当前函数返回。
- **L593 EN**: Closes the current lexical scope or compound statement.
  **L593 CN**: 结束当前词法作用域或复合语句块。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L595 EN**: Continues logic associated with callable symbol `ComputeUpperBound`.
  **L595 CN**: 继续与可调用符号 `ComputeUpperBound` 相关的逻辑。
- **L596 EN**: Continues the surrounding expression or declaration: `ExtentExpr &&lower, MaybeExtentExpr &&extent) {`.
  **L596 CN**: 继续构造周围的表达式或声明：`ExtentExpr &&lower, MaybeExtentExpr &&extent) {`。
- **L597 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L597 CN**: 开始 `if` 控制流语句并计算其条件。
- **L598 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L598 CN**: 开始 `if` 控制流语句并计算其条件。
- **L599 EN**: Returns from the current function with `std::move(*extent)`.
  **L599 CN**: 以 `std::move(*extent)` 从当前函数返回。
- **L600 EN**: Transitions from the previous branch into the alternative path.
  **L600 CN**: 从前一个分支过渡到备选路径。

### Lines 601-624

````cpp
      return std::move(*extent) + std::move(lower) - ExtentExpr{1};
    }
  } else {
    return std::nullopt;
  }
}

MaybeExtentExpr ComputeUpperBound(
    FoldingContext &context, ExtentExpr &&lower, MaybeExtentExpr &&extent) {
  return Fold(context, ComputeUpperBound(std::move(lower), std::move(extent)));
}

MaybeExtentExpr GetRawUpperBound(
    const NamedEntity &base, int dimension, bool invariantOnly) {
  const Symbol &symbol{ResolveAssociations(base.GetLastSymbol())};
  if (const auto *details{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
    int rank{details->shape().Rank()};
    if (dimension < rank) {
      const auto &bound{details->shape()[dimension].ubound().GetExplicit()};
      if (bound && bound->Rank() == 0 &&
          (!invariantOnly || IsScopeInvariantExpr(*bound))) {
        return *bound;
      } else if (semantics::IsAssumedSizeArray(symbol) &&
          dimension + 1 == symbol.Rank()) {
````
- **L601 EN**: Returns from the current function with `std::move(*extent) + std::move(lower) - ExtentExpr{1}`.
  **L601 CN**: 以 `std::move(*extent) + std::move(lower) - ExtentExpr{1}` 从当前函数返回。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Transitions from the previous branch into the alternative path.
  **L603 CN**: 从前一个分支过渡到备选路径。
- **L604 EN**: Returns from the current function with `std::nullopt`.
  **L604 CN**: 以 `std::nullopt` 从当前函数返回。
- **L605 EN**: Closes the current lexical scope or compound statement.
  **L605 CN**: 结束当前词法作用域或复合语句块。
- **L606 EN**: Closes the current lexical scope or compound statement.
  **L606 CN**: 结束当前词法作用域或复合语句块。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L608 EN**: Continues logic associated with callable symbol `ComputeUpperBound`.
  **L608 CN**: 继续与可调用符号 `ComputeUpperBound` 相关的逻辑。
- **L609 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, ExtentExpr &&lower, MaybeExtentExpr &&extent) {`.
  **L609 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, ExtentExpr &&lower, MaybeExtentExpr &&extent) {`。
- **L610 EN**: Returns from the current function with `Fold(context, ComputeUpperBound(std::move(lower), std::move(extent)))`.
  **L610 CN**: 以 `Fold(context, ComputeUpperBound(std::move(lower), std::move(extent)))` 从当前函数返回。
- **L611 EN**: Closes the current lexical scope or compound statement.
  **L611 CN**: 结束当前词法作用域或复合语句块。
- **L612 EN**: Blank line separating nearby declarations or logic blocks.
  **L612 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L613 EN**: Continues logic associated with callable symbol `GetRawUpperBound`.
  **L613 CN**: 继续与可调用符号 `GetRawUpperBound` 相关的逻辑。
- **L614 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L614 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L615 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L615 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L616 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L616 CN**: 开始 `if` 控制流语句并计算其条件。
- **L617 EN**: Executes a call or declaration centered on `rank{details->shape`.
  **L617 CN**: 执行以 `rank{details->shape` 为核心的调用或声明。
- **L618 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L618 CN**: 开始 `if` 控制流语句并计算其条件。
- **L619 EN**: Executes a call or declaration centered on `&bound{details->shape`.
  **L619 CN**: 执行以 `&bound{details->shape` 为核心的调用或声明。
- **L620 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L620 CN**: 开始 `if` 控制流语句并计算其条件。
- **L621 EN**: Starts a function, method, lambda, or structured scope: `(!invariantOnly || IsScopeInvariantExpr(*bound))) {`.
  **L621 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!invariantOnly || IsScopeInvariantExpr(*bound))) {`。
- **L622 EN**: Returns from the current function with `*bound`.
  **L622 CN**: 以 `*bound` 从当前函数返回。
- **L623 EN**: Transitions from the previous branch into an `else if` condition.
  **L623 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L624 EN**: Starts a function, method, lambda, or structured scope: `dimension + 1 == symbol.Rank()) {`.
  **L624 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dimension + 1 == symbol.Rank()) {`。

### Lines 625-648

````cpp
        return std::nullopt;
      } else if (IsSafelyCopyable(base, /*admitPureCall=*/true)) {
        return ComputeUpperBound(
            GetRawLowerBound(base, dimension), GetExtent(base, dimension));
      }
    }
  } else if (const auto *assoc{
                 symbol.detailsIf<semantics::AssocEntityDetails>()}) {
    if (assoc->IsAssumedSize() || assoc->IsAssumedRank()) {
      return std::nullopt;
    } else if (assoc->rank() && dimension >= *assoc->rank()) {
      return std::nullopt;
    } else if (auto extent{GetAssociatedExtent(symbol, dimension)}) {
      return ComputeUpperBound(
          GetRawLowerBound(base, dimension), std::move(extent));
    }
  }
  return std::nullopt;
}

MaybeExtentExpr GetRawUpperBound(FoldingContext &context,
    const NamedEntity &base, int dimension, bool invariantOnly) {
  return Fold(context, GetRawUpperBound(base, dimension, invariantOnly));
}
````
- **L625 EN**: Returns from the current function with `std::nullopt`.
  **L625 CN**: 以 `std::nullopt` 从当前函数返回。
- **L626 EN**: Transitions from the previous branch into an `else if` condition.
  **L626 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L627 EN**: Returns from the current function with `ComputeUpperBound(`.
  **L627 CN**: 以 `ComputeUpperBound(` 从当前函数返回。
- **L628 EN**: Executes a call or declaration centered on `GetRawLowerBound`.
  **L628 CN**: 执行以 `GetRawLowerBound` 为核心的调用或声明。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Closes the current lexical scope or compound statement.
  **L630 CN**: 结束当前词法作用域或复合语句块。
- **L631 EN**: Transitions from the previous branch into an `else if` condition.
  **L631 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L632 EN**: Starts a function, method, lambda, or structured scope: `symbol.detailsIf<semantics::AssocEntityDetails>()}) {`.
  **L632 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.detailsIf<semantics::AssocEntityDetails>()}) {`。
- **L633 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L633 CN**: 开始 `if` 控制流语句并计算其条件。
- **L634 EN**: Returns from the current function with `std::nullopt`.
  **L634 CN**: 以 `std::nullopt` 从当前函数返回。
- **L635 EN**: Transitions from the previous branch into an `else if` condition.
  **L635 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L636 EN**: Returns from the current function with `std::nullopt`.
  **L636 CN**: 以 `std::nullopt` 从当前函数返回。
- **L637 EN**: Transitions from the previous branch into an `else if` condition.
  **L637 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L638 EN**: Returns from the current function with `ComputeUpperBound(`.
  **L638 CN**: 以 `ComputeUpperBound(` 从当前函数返回。
- **L639 EN**: Executes a call or declaration centered on `GetRawLowerBound`.
  **L639 CN**: 执行以 `GetRawLowerBound` 为核心的调用或声明。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Closes the current lexical scope or compound statement.
  **L641 CN**: 结束当前词法作用域或复合语句块。
- **L642 EN**: Returns from the current function with `std::nullopt`.
  **L642 CN**: 以 `std::nullopt` 从当前函数返回。
- **L643 EN**: Closes the current lexical scope or compound statement.
  **L643 CN**: 结束当前词法作用域或复合语句块。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetRawUpperBound(FoldingContext &context,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetRawUpperBound(FoldingContext &context,`。
- **L646 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L646 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L647 EN**: Returns from the current function with `Fold(context, GetRawUpperBound(base, dimension, invariantOnly))`.
  **L647 CN**: 以 `Fold(context, GetRawUpperBound(base, dimension, invariantOnly))` 从当前函数返回。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````cpp

static MaybeExtentExpr GetExplicitUBOUND(FoldingContext *context,
    const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {
  const auto &ubound{shapeSpec.ubound().GetExplicit()};
  if (ubound && ubound->Rank() == 0 &&
      (!invariantOnly || IsScopeInvariantExpr(*ubound, context))) {
    if (auto extent{GetNonNegativeExtent(shapeSpec, invariantOnly)}) {
      if (auto cstExtent{ToInt64(
              context ? Fold(*context, std::move(*extent)) : *extent)}) {
        if (cstExtent > 0) {
          return *ubound;
        } else if (cstExtent == 0) {
          return ExtentExpr{0};
        }
      }
    }
  }
  return std::nullopt;
}

static MaybeExtentExpr GetUBOUND(FoldingContext *context,
    const NamedEntity &base, int dimension, bool invariantOnly) {
  const Symbol &symbol{ResolveAssociations(base.GetLastSymbol())};
  if (const auto *details{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static MaybeExtentExpr GetExplicitUBOUND(FoldingContext *context,`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`static MaybeExtentExpr GetExplicitUBOUND(FoldingContext *context,`。
- **L651 EN**: Continues the surrounding expression or declaration: `const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {`.
  **L651 CN**: 继续构造周围的表达式或声明：`const semantics::ShapeSpec &shapeSpec, bool invariantOnly) {`。
- **L652 EN**: Executes a call or declaration centered on `&ubound{shapeSpec.ubound`.
  **L652 CN**: 执行以 `&ubound{shapeSpec.ubound` 为核心的调用或声明。
- **L653 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L653 CN**: 开始 `if` 控制流语句并计算其条件。
- **L654 EN**: Starts a function, method, lambda, or structured scope: `(!invariantOnly || IsScopeInvariantExpr(*ubound, context))) {`.
  **L654 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!invariantOnly || IsScopeInvariantExpr(*ubound, context))) {`。
- **L655 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L655 CN**: 开始 `if` 控制流语句并计算其条件。
- **L656 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L656 CN**: 开始 `if` 控制流语句并计算其条件。
- **L657 EN**: Starts a function, method, lambda, or structured scope: `context ? Fold(*context, std::move(*extent)) : *extent)}) {`.
  **L657 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context ? Fold(*context, std::move(*extent)) : *extent)}) {`。
- **L658 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L658 CN**: 开始 `if` 控制流语句并计算其条件。
- **L659 EN**: Returns from the current function with `*ubound`.
  **L659 CN**: 以 `*ubound` 从当前函数返回。
- **L660 EN**: Transitions from the previous branch into an `else if` condition.
  **L660 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L661 EN**: Returns from the current function with `ExtentExpr{0}`.
  **L661 CN**: 以 `ExtentExpr{0}` 从当前函数返回。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Closes the current lexical scope or compound statement.
  **L665 CN**: 结束当前词法作用域或复合语句块。
- **L666 EN**: Returns from the current function with `std::nullopt`.
  **L666 CN**: 以 `std::nullopt` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static MaybeExtentExpr GetUBOUND(FoldingContext *context,`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`static MaybeExtentExpr GetUBOUND(FoldingContext *context,`。
- **L670 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L670 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L671 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L671 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L672 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L672 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 673-696

````cpp
    int rank{details->shape().Rank()};
    if (dimension < rank) {
      const semantics::ShapeSpec &shapeSpec{details->shape()[dimension]};
      if (auto ubound{GetExplicitUBOUND(context, shapeSpec, invariantOnly)}) {
        return *ubound;
      } else if (semantics::IsAssumedSizeArray(symbol) &&
          dimension + 1 == symbol.Rank()) {
        return std::nullopt; // UBOUND() folding replaces with -1
      } else if (IsSafelyCopyable(base, /*admitPureCall=*/true)) {
        if (auto lb{GetLBOUND(base, dimension, invariantOnly)}) {
          return ComputeUpperBound(
              std::move(*lb), GetExtent(base, dimension, invariantOnly));
        }
      }
    }
  } else if (const auto *assoc{
                 symbol.detailsIf<semantics::AssocEntityDetails>()}) {
    if (assoc->IsAssumedSize() || assoc->IsAssumedRank()) {
      return std::nullopt;
    } else if (assoc->rank()) { // RANK (n)
      const Symbol &resolved{ResolveAssociations(symbol)};
      if (IsDescriptor(resolved) && dimension < *assoc->rank()) {
        ExtentExpr lb{DescriptorInquiry{NamedEntity{base},
            DescriptorInquiry::Field::LowerBound, dimension}};
````
- **L673 EN**: Executes a call or declaration centered on `rank{details->shape`.
  **L673 CN**: 执行以 `rank{details->shape` 为核心的调用或声明。
- **L674 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L674 CN**: 开始 `if` 控制流语句并计算其条件。
- **L675 EN**: Executes a call or declaration centered on `&shapeSpec{details->shape`.
  **L675 CN**: 执行以 `&shapeSpec{details->shape` 为核心的调用或声明。
- **L676 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L676 CN**: 开始 `if` 控制流语句并计算其条件。
- **L677 EN**: Returns from the current function with `*ubound`.
  **L677 CN**: 以 `*ubound` 从当前函数返回。
- **L678 EN**: Transitions from the previous branch into an `else if` condition.
  **L678 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L679 EN**: Starts a function, method, lambda, or structured scope: `dimension + 1 == symbol.Rank()) {`.
  **L679 CN**: 开始一个函数、方法、lambda 或结构化作用域：`dimension + 1 == symbol.Rank()) {`。
- **L680 EN**: Returns from the current function with `std::nullopt; // UBOUND() folding replaces with -1`.
  **L680 CN**: 以 `std::nullopt; // UBOUND() folding replaces with -1` 从当前函数返回。
- **L681 EN**: Transitions from the previous branch into an `else if` condition.
  **L681 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L682 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L682 CN**: 开始 `if` 控制流语句并计算其条件。
- **L683 EN**: Returns from the current function with `ComputeUpperBound(`.
  **L683 CN**: 以 `ComputeUpperBound(` 从当前函数返回。
- **L684 EN**: Executes a call or declaration centered on `std::move`.
  **L684 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Closes the current lexical scope or compound statement.
  **L686 CN**: 结束当前词法作用域或复合语句块。
- **L687 EN**: Closes the current lexical scope or compound statement.
  **L687 CN**: 结束当前词法作用域或复合语句块。
- **L688 EN**: Transitions from the previous branch into an `else if` condition.
  **L688 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L689 EN**: Starts a function, method, lambda, or structured scope: `symbol.detailsIf<semantics::AssocEntityDetails>()}) {`.
  **L689 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.detailsIf<semantics::AssocEntityDetails>()}) {`。
- **L690 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L690 CN**: 开始 `if` 控制流语句并计算其条件。
- **L691 EN**: Returns from the current function with `std::nullopt`.
  **L691 CN**: 以 `std::nullopt` 从当前函数返回。
- **L692 EN**: Transitions from the previous branch into an `else if` condition.
  **L692 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L693 EN**: Executes a call or declaration centered on `&resolved{ResolveAssociations`.
  **L693 CN**: 执行以 `&resolved{ResolveAssociations` 为核心的调用或声明。
- **L694 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L694 CN**: 开始 `if` 控制流语句并计算其条件。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExtentExpr lb{DescriptorInquiry{NamedEntity{base},`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExtentExpr lb{DescriptorInquiry{NamedEntity{base},`。
- **L696 EN**: Executes a standalone statement or declaration: `DescriptorInquiry::Field::LowerBound, dimension}};`.
  **L696 CN**: 执行一条独立语句或声明：`DescriptorInquiry::Field::LowerBound, dimension}};`。

### Lines 697-720

````cpp
        ExtentExpr extent{DescriptorInquiry{
            std::move(base), DescriptorInquiry::Field::Extent, dimension}};
        return ComputeUpperBound(std::move(lb), std::move(extent));
      }
    } else if (auto extent{GetAssociatedExtent(symbol, dimension)}) {
      if (auto lb{GetLBOUND(base, dimension, invariantOnly)}) {
        return ComputeUpperBound(std::move(*lb), std::move(extent));
      }
    }
  }
  return std::nullopt;
}

MaybeExtentExpr GetUBOUND(
    const NamedEntity &base, int dimension, bool invariantOnly) {
  return GetUBOUND(nullptr, base, dimension, invariantOnly);
}

MaybeExtentExpr GetUBOUND(FoldingContext &context, const NamedEntity &base,
    int dimension, bool invariantOnly) {
  return Fold(context, GetUBOUND(&context, base, dimension, invariantOnly));
}

static Shape GetUBOUNDs(
````
- **L697 EN**: Continues the surrounding expression or declaration: `ExtentExpr extent{DescriptorInquiry{`.
  **L697 CN**: 继续构造周围的表达式或声明：`ExtentExpr extent{DescriptorInquiry{`。
- **L698 EN**: Executes a call or declaration centered on `std::move`.
  **L698 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L699 EN**: Returns from the current function with `ComputeUpperBound(std::move(lb), std::move(extent))`.
  **L699 CN**: 以 `ComputeUpperBound(std::move(lb), std::move(extent))` 从当前函数返回。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Transitions from the previous branch into an `else if` condition.
  **L701 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L702 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L702 CN**: 开始 `if` 控制流语句并计算其条件。
- **L703 EN**: Returns from the current function with `ComputeUpperBound(std::move(*lb), std::move(extent))`.
  **L703 CN**: 以 `ComputeUpperBound(std::move(*lb), std::move(extent))` 从当前函数返回。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Closes the current lexical scope or compound statement.
  **L705 CN**: 结束当前词法作用域或复合语句块。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Returns from the current function with `std::nullopt`.
  **L707 CN**: 以 `std::nullopt` 从当前函数返回。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Blank line separating nearby declarations or logic blocks.
  **L709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L710 EN**: Continues logic associated with callable symbol `GetUBOUND`.
  **L710 CN**: 继续与可调用符号 `GetUBOUND` 相关的逻辑。
- **L711 EN**: Continues the surrounding expression or declaration: `const NamedEntity &base, int dimension, bool invariantOnly) {`.
  **L711 CN**: 继续构造周围的表达式或声明：`const NamedEntity &base, int dimension, bool invariantOnly) {`。
- **L712 EN**: Returns from the current function with `GetUBOUND(nullptr, base, dimension, invariantOnly)`.
  **L712 CN**: 以 `GetUBOUND(nullptr, base, dimension, invariantOnly)` 从当前函数返回。
- **L713 EN**: Closes the current lexical scope or compound statement.
  **L713 CN**: 结束当前词法作用域或复合语句块。
- **L714 EN**: Blank line separating nearby declarations or logic blocks.
  **L714 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaybeExtentExpr GetUBOUND(FoldingContext &context, const NamedEntity &base,`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaybeExtentExpr GetUBOUND(FoldingContext &context, const NamedEntity &base,`。
- **L716 EN**: Continues the surrounding expression or declaration: `int dimension, bool invariantOnly) {`.
  **L716 CN**: 继续构造周围的表达式或声明：`int dimension, bool invariantOnly) {`。
- **L717 EN**: Returns from the current function with `Fold(context, GetUBOUND(&context, base, dimension, invariantOnly))`.
  **L717 CN**: 以 `Fold(context, GetUBOUND(&context, base, dimension, invariantOnly))` 从当前函数返回。
- **L718 EN**: Closes the current lexical scope or compound statement.
  **L718 CN**: 结束当前词法作用域或复合语句块。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L720 EN**: Continues logic associated with callable symbol `GetUBOUNDs`.
  **L720 CN**: 继续与可调用符号 `GetUBOUNDs` 相关的逻辑。

### Lines 721-744

````cpp
    FoldingContext *context, const NamedEntity &base, bool invariantOnly) {
  Shape result;
  int rank{base.Rank()};
  for (int dim{0}; dim < rank; ++dim) {
    result.emplace_back(GetUBOUND(context, base, dim, invariantOnly));
  }
  return result;
}

Shape GetUBOUNDs(
    FoldingContext &context, const NamedEntity &base, bool invariantOnly) {
  return Fold(context, GetUBOUNDs(&context, base, invariantOnly));
}

Shape GetUBOUNDs(const NamedEntity &base, bool invariantOnly) {
  return GetUBOUNDs(nullptr, base, invariantOnly);
}

MaybeExtentExpr GetLCOBOUND(
    const Symbol &symbol0, int dimension, bool invariantOnly) {
  const Symbol &symbol{ResolveAssociations(symbol0)};
  if (const auto *object{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
    int corank{object->coshape().Rank()};
    if (dimension < corank) {
````
- **L721 EN**: Continues the surrounding expression or declaration: `FoldingContext *context, const NamedEntity &base, bool invariantOnly) {`.
  **L721 CN**: 继续构造周围的表达式或声明：`FoldingContext *context, const NamedEntity &base, bool invariantOnly) {`。
- **L722 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L722 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L723 EN**: Executes a call or declaration centered on `rank{base.Rank`.
  **L723 CN**: 执行以 `rank{base.Rank` 为核心的调用或声明。
- **L724 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L724 CN**: 开始 `for` 控制流语句并计算其条件。
- **L725 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L725 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Returns from the current function with `result`.
  **L727 CN**: 以 `result` 从当前函数返回。
- **L728 EN**: Closes the current lexical scope or compound statement.
  **L728 CN**: 结束当前词法作用域或复合语句块。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L730 EN**: Continues logic associated with callable symbol `GetUBOUNDs`.
  **L730 CN**: 继续与可调用符号 `GetUBOUNDs` 相关的逻辑。
- **L731 EN**: Continues the surrounding expression or declaration: `FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`.
  **L731 CN**: 继续构造周围的表达式或声明：`FoldingContext &context, const NamedEntity &base, bool invariantOnly) {`。
- **L732 EN**: Returns from the current function with `Fold(context, GetUBOUNDs(&context, base, invariantOnly))`.
  **L732 CN**: 以 `Fold(context, GetUBOUNDs(&context, base, invariantOnly))` 从当前函数返回。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L735 EN**: Starts a function, method, lambda, or structured scope: `Shape GetUBOUNDs(const NamedEntity &base, bool invariantOnly) {`.
  **L735 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetUBOUNDs(const NamedEntity &base, bool invariantOnly) {`。
- **L736 EN**: Returns from the current function with `GetUBOUNDs(nullptr, base, invariantOnly)`.
  **L736 CN**: 以 `GetUBOUNDs(nullptr, base, invariantOnly)` 从当前函数返回。
- **L737 EN**: Closes the current lexical scope or compound statement.
  **L737 CN**: 结束当前词法作用域或复合语句块。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L739 EN**: Continues logic associated with callable symbol `GetLCOBOUND`.
  **L739 CN**: 继续与可调用符号 `GetLCOBOUND` 相关的逻辑。
- **L740 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol0, int dimension, bool invariantOnly) {`.
  **L740 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol0, int dimension, bool invariantOnly) {`。
- **L741 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L741 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L742 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L742 CN**: 开始 `if` 控制流语句并计算其条件。
- **L743 EN**: Executes a call or declaration centered on `corank{object->coshape`.
  **L743 CN**: 执行以 `corank{object->coshape` 为核心的调用或声明。
- **L744 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L744 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 745-768

````cpp
      const semantics::ShapeSpec &shapeSpec{object->coshape()[dimension]};
      if (const auto &lcobound{shapeSpec.lbound().GetExplicit()}) {
        if (lcobound->Rank() == 0 &&
            (!invariantOnly || IsScopeInvariantExpr(*lcobound))) {
          return *lcobound;
        }
      }
    }
  }
  return std::nullopt;
}

MaybeExtentExpr GetUCOBOUND(
    const Symbol &symbol0, int dimension, bool invariantOnly) {
  const Symbol &symbol{ResolveAssociations(symbol0)};
  if (const auto *object{symbol.detailsIf<semantics::ObjectEntityDetails>()}) {
    int corank{object->coshape().Rank()};
    if (dimension < corank - 1) {
      const semantics::ShapeSpec &shapeSpec{object->coshape()[dimension]};
      if (const auto ucobound{shapeSpec.ubound().GetExplicit()}) {
        if (ucobound->Rank() == 0 &&
            (!invariantOnly || IsScopeInvariantExpr(*ucobound))) {
          return *ucobound;
        }
````
- **L745 EN**: Executes a call or declaration centered on `&shapeSpec{object->coshape`.
  **L745 CN**: 执行以 `&shapeSpec{object->coshape` 为核心的调用或声明。
- **L746 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L746 CN**: 开始 `if` 控制流语句并计算其条件。
- **L747 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L747 CN**: 开始 `if` 控制流语句并计算其条件。
- **L748 EN**: Starts a function, method, lambda, or structured scope: `(!invariantOnly || IsScopeInvariantExpr(*lcobound))) {`.
  **L748 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!invariantOnly || IsScopeInvariantExpr(*lcobound))) {`。
- **L749 EN**: Returns from the current function with `*lcobound`.
  **L749 CN**: 以 `*lcobound` 从当前函数返回。
- **L750 EN**: Closes the current lexical scope or compound statement.
  **L750 CN**: 结束当前词法作用域或复合语句块。
- **L751 EN**: Closes the current lexical scope or compound statement.
  **L751 CN**: 结束当前词法作用域或复合语句块。
- **L752 EN**: Closes the current lexical scope or compound statement.
  **L752 CN**: 结束当前词法作用域或复合语句块。
- **L753 EN**: Closes the current lexical scope or compound statement.
  **L753 CN**: 结束当前词法作用域或复合语句块。
- **L754 EN**: Returns from the current function with `std::nullopt`.
  **L754 CN**: 以 `std::nullopt` 从当前函数返回。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L757 EN**: Continues logic associated with callable symbol `GetUCOBOUND`.
  **L757 CN**: 继续与可调用符号 `GetUCOBOUND` 相关的逻辑。
- **L758 EN**: Continues the surrounding expression or declaration: `const Symbol &symbol0, int dimension, bool invariantOnly) {`.
  **L758 CN**: 继续构造周围的表达式或声明：`const Symbol &symbol0, int dimension, bool invariantOnly) {`。
- **L759 EN**: Executes a call or declaration centered on `&symbol{ResolveAssociations`.
  **L759 CN**: 执行以 `&symbol{ResolveAssociations` 为核心的调用或声明。
- **L760 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L760 CN**: 开始 `if` 控制流语句并计算其条件。
- **L761 EN**: Executes a call or declaration centered on `corank{object->coshape`.
  **L761 CN**: 执行以 `corank{object->coshape` 为核心的调用或声明。
- **L762 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L762 CN**: 开始 `if` 控制流语句并计算其条件。
- **L763 EN**: Executes a call or declaration centered on `&shapeSpec{object->coshape`.
  **L763 CN**: 执行以 `&shapeSpec{object->coshape` 为核心的调用或声明。
- **L764 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L764 CN**: 开始 `if` 控制流语句并计算其条件。
- **L765 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L765 CN**: 开始 `if` 控制流语句并计算其条件。
- **L766 EN**: Starts a function, method, lambda, or structured scope: `(!invariantOnly || IsScopeInvariantExpr(*ucobound))) {`.
  **L766 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!invariantOnly || IsScopeInvariantExpr(*ucobound))) {`。
- **L767 EN**: Returns from the current function with `*ucobound`.
  **L767 CN**: 以 `*ucobound` 从当前函数返回。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````cpp
      }
    }
  }
  return std::nullopt;
}

Shape GetLCOBOUNDs(const Symbol &symbol, bool invariantOnly) {
  Shape result;
  int corank{symbol.Corank()};
  for (int dim{0}; dim < corank; ++dim) {
    result.emplace_back(GetLCOBOUND(symbol, dim, invariantOnly));
  }
  return result;
}

Shape GetUCOBOUNDs(const Symbol &symbol, bool invariantOnly) {
  Shape result;
  int corank{symbol.Corank()};
  for (int dim{0}; dim < corank; ++dim) {
    result.emplace_back(GetUCOBOUND(symbol, dim, invariantOnly));
  }
  return result;
}

````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Returns from the current function with `std::nullopt`.
  **L772 CN**: 以 `std::nullopt` 从当前函数返回。
- **L773 EN**: Closes the current lexical scope or compound statement.
  **L773 CN**: 结束当前词法作用域或复合语句块。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L775 EN**: Starts a function, method, lambda, or structured scope: `Shape GetLCOBOUNDs(const Symbol &symbol, bool invariantOnly) {`.
  **L775 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetLCOBOUNDs(const Symbol &symbol, bool invariantOnly) {`。
- **L776 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L776 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L777 EN**: Executes a call or declaration centered on `corank{symbol.Corank`.
  **L777 CN**: 执行以 `corank{symbol.Corank` 为核心的调用或声明。
- **L778 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L778 CN**: 开始 `for` 控制流语句并计算其条件。
- **L779 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L779 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L780 EN**: Closes the current lexical scope or compound statement.
  **L780 CN**: 结束当前词法作用域或复合语句块。
- **L781 EN**: Returns from the current function with `result`.
  **L781 CN**: 以 `result` 从当前函数返回。
- **L782 EN**: Closes the current lexical scope or compound statement.
  **L782 CN**: 结束当前词法作用域或复合语句块。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L784 EN**: Starts a function, method, lambda, or structured scope: `Shape GetUCOBOUNDs(const Symbol &symbol, bool invariantOnly) {`.
  **L784 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Shape GetUCOBOUNDs(const Symbol &symbol, bool invariantOnly) {`。
- **L785 EN**: Executes a standalone statement or declaration: `Shape result;`.
  **L785 CN**: 执行一条独立语句或声明：`Shape result;`。
- **L786 EN**: Executes a call or declaration centered on `corank{symbol.Corank`.
  **L786 CN**: 执行以 `corank{symbol.Corank` 为核心的调用或声明。
- **L787 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L787 CN**: 开始 `for` 控制流语句并计算其条件。
- **L788 EN**: Executes a call or declaration centered on `result.emplace_back`.
  **L788 CN**: 执行以 `result.emplace_back` 为核心的调用或声明。
- **L789 EN**: Closes the current lexical scope or compound statement.
  **L789 CN**: 结束当前词法作用域或复合语句块。
- **L790 EN**: Returns from the current function with `result`.
  **L790 CN**: 以 `result` 从当前函数返回。
- **L791 EN**: Closes the current lexical scope or compound statement.
  **L791 CN**: 结束当前词法作用域或复合语句块。
- **L792 EN**: Blank line separating nearby declarations or logic blocks.
  **L792 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 793-816

````cpp
auto GetShapeHelper::operator()(const Symbol &symbol) const -> Result {
  return common::visit(
      common::visitors{
          [&](const semantics::ObjectEntityDetails &object) {
            if (IsImpliedShape(symbol) && object.init()) {
              return (*this)(object.init());
            } else if (IsAssumedRank(symbol)) {
              return Result{};
            } else {
              int n{object.shape().Rank()};
              NamedEntity base{symbol};
              return Result{CreateShape(n, base)};
            }
          },
          [](const semantics::EntityDetails &) {
            return ScalarShape(); // no dimensions seen
          },
          [&](const semantics::ProcEntityDetails &proc) {
            if (const Symbol * interface{proc.procInterface()}) {
              return (*this)(*interface);
            } else {
              return ScalarShape();
            }
          },
````
- **L793 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const Symbol &symbol) const -> Result {`.
  **L793 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const Symbol &symbol) const -> Result {`。
- **L794 EN**: Returns from the current function with `common::visit(`.
  **L794 CN**: 以 `common::visit(` 从当前函数返回。
- **L795 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L795 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L796 EN**: Starts a function, method, lambda, or structured scope: `[&](const semantics::ObjectEntityDetails &object) {`.
  **L796 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const semantics::ObjectEntityDetails &object) {`。
- **L797 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L797 CN**: 开始 `if` 控制流语句并计算其条件。
- **L798 EN**: Returns from the current function with `(*this)(object.init())`.
  **L798 CN**: 以 `(*this)(object.init())` 从当前函数返回。
- **L799 EN**: Transitions from the previous branch into an `else if` condition.
  **L799 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L800 EN**: Returns from the current function with `Result{}`.
  **L800 CN**: 以 `Result{}` 从当前函数返回。
- **L801 EN**: Transitions from the previous branch into the alternative path.
  **L801 CN**: 从前一个分支过渡到备选路径。
- **L802 EN**: Executes a call or declaration centered on `n{object.shape`.
  **L802 CN**: 执行以 `n{object.shape` 为核心的调用或声明。
- **L803 EN**: Executes a standalone statement or declaration: `NamedEntity base{symbol};`.
  **L803 CN**: 执行一条独立语句或声明：`NamedEntity base{symbol};`。
- **L804 EN**: Returns from the current function with `Result{CreateShape(n, base)}`.
  **L804 CN**: 以 `Result{CreateShape(n, base)}` 从当前函数返回。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L807 EN**: Starts a function, method, lambda, or structured scope: `[](const semantics::EntityDetails &) {`.
  **L807 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[](const semantics::EntityDetails &) {`。
- **L808 EN**: Returns from the current function with `ScalarShape(); // no dimensions seen`.
  **L808 CN**: 以 `ScalarShape(); // no dimensions seen` 从当前函数返回。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L810 EN**: Starts a function, method, lambda, or structured scope: `[&](const semantics::ProcEntityDetails &proc) {`.
  **L810 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const semantics::ProcEntityDetails &proc) {`。
- **L811 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L811 CN**: 开始 `if` 控制流语句并计算其条件。
- **L812 EN**: Returns from the current function with `(*this)(*interface)`.
  **L812 CN**: 以 `(*this)(*interface)` 从当前函数返回。
- **L813 EN**: Transitions from the previous branch into the alternative path.
  **L813 CN**: 从前一个分支过渡到备选路径。
- **L814 EN**: Returns from the current function with `ScalarShape()`.
  **L814 CN**: 以 `ScalarShape()` 从当前函数返回。
- **L815 EN**: Closes the current lexical scope or compound statement.
  **L815 CN**: 结束当前词法作用域或复合语句块。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 817-840

````cpp
          [&](const semantics::AssocEntityDetails &assoc) {
            NamedEntity base{symbol};
            if (assoc.rank()) { // SELECT RANK case
              int n{assoc.rank().value()};
              return Result{CreateShape(n, base)};
            } else {
              auto exprShape{((*this)(assoc.expr()))};
              if (exprShape) {
                int rank{static_cast<int>(exprShape->size())};
                for (int dimension{0}; dimension < rank; ++dimension) {
                  auto &extent{(*exprShape)[dimension]};
                  if (extent && !IsActuallyConstant(*extent)) {
                    extent = GetExtent(base, dimension);
                  }
                }
              }
              return exprShape;
            }
          },
          [&](const semantics::SubprogramDetails &subp) -> Result {
            if (subp.isFunction()) {
              auto resultShape{(*this)(subp.result())};
              if (resultShape && !useResultSymbolShape_) {
                // Ensure the shape is constant. Otherwise, it may be reerring
````
- **L817 EN**: Starts a function, method, lambda, or structured scope: `[&](const semantics::AssocEntityDetails &assoc) {`.
  **L817 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const semantics::AssocEntityDetails &assoc) {`。
- **L818 EN**: Executes a standalone statement or declaration: `NamedEntity base{symbol};`.
  **L818 CN**: 执行一条独立语句或声明：`NamedEntity base{symbol};`。
- **L819 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L819 CN**: 开始 `if` 控制流语句并计算其条件。
- **L820 EN**: Executes a call or declaration centered on `n{assoc.rank`.
  **L820 CN**: 执行以 `n{assoc.rank` 为核心的调用或声明。
- **L821 EN**: Returns from the current function with `Result{CreateShape(n, base)}`.
  **L821 CN**: 以 `Result{CreateShape(n, base)}` 从当前函数返回。
- **L822 EN**: Transitions from the previous branch into the alternative path.
  **L822 CN**: 从前一个分支过渡到备选路径。
- **L823 EN**: Executes a call or declaration centered on `exprShape{`.
  **L823 CN**: 执行以 `exprShape{` 为核心的调用或声明。
- **L824 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L824 CN**: 开始 `if` 控制流语句并计算其条件。
- **L825 EN**: Executes a call or declaration centered on `rank{static_cast<int>`.
  **L825 CN**: 执行以 `rank{static_cast<int>` 为核心的调用或声明。
- **L826 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L826 CN**: 开始 `for` 控制流语句并计算其条件。
- **L827 EN**: Executes a call or declaration centered on `&extent{`.
  **L827 CN**: 执行以 `&extent{` 为核心的调用或声明。
- **L828 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L828 CN**: 开始 `if` 控制流语句并计算其条件。
- **L829 EN**: Executes a call or declaration centered on `GetExtent`.
  **L829 CN**: 执行以 `GetExtent` 为核心的调用或声明。
- **L830 EN**: Closes the current lexical scope or compound statement.
  **L830 CN**: 结束当前词法作用域或复合语句块。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Returns from the current function with `exprShape`.
  **L833 CN**: 以 `exprShape` 从当前函数返回。
- **L834 EN**: Closes the current lexical scope or compound statement.
  **L834 CN**: 结束当前词法作用域或复合语句块。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L836 EN**: Starts a function, method, lambda, or structured scope: `[&](const semantics::SubprogramDetails &subp) -> Result {`.
  **L836 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const semantics::SubprogramDetails &subp) -> Result {`。
- **L837 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L837 CN**: 开始 `if` 控制流语句并计算其条件。
- **L838 EN**: Executes a call or declaration centered on `resultShape{`.
  **L838 CN**: 执行以 `resultShape{` 为核心的调用或声明。
- **L839 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L839 CN**: 开始 `if` 控制流语句并计算其条件。
- **L840 EN**: Comment explains nearby logic, intent, or metadata: `Ensure the shape is constant. Otherwise, it may be reerring`.
  **L840 CN**: 注释说明附近代码的逻辑、意图或元数据：`Ensure the shape is constant. Otherwise, it may be reerring`。

### Lines 841-864

````cpp
                // to symbols that belong to the function's scope and are
                // meaningless on the caller side without the related call
                // expression.
                for (auto &extent : *resultShape) {
                  if (extent && !IsActuallyConstant(*extent)) {
                    extent.reset();
                  }
                }
              }
              return resultShape;
            } else {
              return Result{};
            }
          },
          [&](const semantics::ProcBindingDetails &binding) {
            return (*this)(binding.symbol());
          },
          [](const semantics::TypeParamDetails &) { return ScalarShape(); },
          [](const auto &) { return Result{}; },
      },
      symbol.GetUltimate().details());
}

auto GetShapeHelper::operator()(const Component &component) const -> Result {
````
- **L841 EN**: Comment explains nearby logic, intent, or metadata: `to symbols that belong to the function's scope and are`.
  **L841 CN**: 注释说明附近代码的逻辑、意图或元数据：`to symbols that belong to the function's scope and are`。
- **L842 EN**: Comment explains nearby logic, intent, or metadata: `meaningless on the caller side without the related call`.
  **L842 CN**: 注释说明附近代码的逻辑、意图或元数据：`meaningless on the caller side without the related call`。
- **L843 EN**: Comment explains nearby logic, intent, or metadata: `expression.`.
  **L843 CN**: 注释说明附近代码的逻辑、意图或元数据：`expression.`。
- **L844 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L844 CN**: 开始 `for` 控制流语句并计算其条件。
- **L845 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L845 CN**: 开始 `if` 控制流语句并计算其条件。
- **L846 EN**: Executes a call or declaration centered on `extent.reset`.
  **L846 CN**: 执行以 `extent.reset` 为核心的调用或声明。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Closes the current lexical scope or compound statement.
  **L848 CN**: 结束当前词法作用域或复合语句块。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Returns from the current function with `resultShape`.
  **L850 CN**: 以 `resultShape` 从当前函数返回。
- **L851 EN**: Transitions from the previous branch into the alternative path.
  **L851 CN**: 从前一个分支过渡到备选路径。
- **L852 EN**: Returns from the current function with `Result{}`.
  **L852 CN**: 以 `Result{}` 从当前函数返回。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L855 EN**: Starts a function, method, lambda, or structured scope: `[&](const semantics::ProcBindingDetails &binding) {`.
  **L855 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const semantics::ProcBindingDetails &binding) {`。
- **L856 EN**: Returns from the current function with `(*this)(binding.symbol())`.
  **L856 CN**: 以 `(*this)(binding.symbol())` 从当前函数返回。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const semantics::TypeParamDetails &) { return ScalarShape(); },`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const semantics::TypeParamDetails &) { return ScalarShape(); },`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const auto &) { return Result{}; },`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const auto &) { return Result{}; },`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L861 EN**: Executes a call or declaration centered on `symbol.GetUltimate`.
  **L861 CN**: 执行以 `symbol.GetUltimate` 为核心的调用或声明。
- **L862 EN**: Closes the current lexical scope or compound statement.
  **L862 CN**: 结束当前词法作用域或复合语句块。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L864 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const Component &component) const -> Result {`.
  **L864 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const Component &component) const -> Result {`。

### Lines 865-888

````cpp
  const Symbol &symbol{component.GetLastSymbol()};
  int rank{symbol.Rank()};
  if (rank == 0) {
    return (*this)(component.base());
  } else if (symbol.has<semantics::ObjectEntityDetails>()) {
    NamedEntity base{Component{component}};
    return CreateShape(rank, base);
  } else {
    return (*this)(symbol);
  }
}

auto GetShapeHelper::operator()(const ArrayRef &arrayRef) const -> Result {
  Shape shape;
  int dimension{0};
  const NamedEntity &base{arrayRef.base()};
  for (const Subscript &ss : arrayRef.subscript()) {
    if (ss.Rank() > 0) {
      shape.emplace_back(GetExtent(ss, base, dimension));
    }
    ++dimension;
  }
  if (shape.empty()) {
    if (const Component * component{base.UnwrapComponent()}) {
````
- **L865 EN**: Executes a call or declaration centered on `&symbol{component.GetLastSymbol`.
  **L865 CN**: 执行以 `&symbol{component.GetLastSymbol` 为核心的调用或声明。
- **L866 EN**: Executes a call or declaration centered on `rank{symbol.Rank`.
  **L866 CN**: 执行以 `rank{symbol.Rank` 为核心的调用或声明。
- **L867 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L867 CN**: 开始 `if` 控制流语句并计算其条件。
- **L868 EN**: Returns from the current function with `(*this)(component.base())`.
  **L868 CN**: 以 `(*this)(component.base())` 从当前函数返回。
- **L869 EN**: Transitions from the previous branch into an `else if` condition.
  **L869 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L870 EN**: Executes a standalone statement or declaration: `NamedEntity base{Component{component}};`.
  **L870 CN**: 执行一条独立语句或声明：`NamedEntity base{Component{component}};`。
- **L871 EN**: Returns from the current function with `CreateShape(rank, base)`.
  **L871 CN**: 以 `CreateShape(rank, base)` 从当前函数返回。
- **L872 EN**: Transitions from the previous branch into the alternative path.
  **L872 CN**: 从前一个分支过渡到备选路径。
- **L873 EN**: Returns from the current function with `(*this)(symbol)`.
  **L873 CN**: 以 `(*this)(symbol)` 从当前函数返回。
- **L874 EN**: Closes the current lexical scope or compound statement.
  **L874 CN**: 结束当前词法作用域或复合语句块。
- **L875 EN**: Closes the current lexical scope or compound statement.
  **L875 CN**: 结束当前词法作用域或复合语句块。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L877 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const ArrayRef &arrayRef) const -> Result {`.
  **L877 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const ArrayRef &arrayRef) const -> Result {`。
- **L878 EN**: Executes a standalone statement or declaration: `Shape shape;`.
  **L878 CN**: 执行一条独立语句或声明：`Shape shape;`。
- **L879 EN**: Executes a standalone statement or declaration: `int dimension{0};`.
  **L879 CN**: 执行一条独立语句或声明：`int dimension{0};`。
- **L880 EN**: Executes a call or declaration centered on `&base{arrayRef.base`.
  **L880 CN**: 执行以 `&base{arrayRef.base` 为核心的调用或声明。
- **L881 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L881 CN**: 开始 `for` 控制流语句并计算其条件。
- **L882 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L882 CN**: 开始 `if` 控制流语句并计算其条件。
- **L883 EN**: Executes a call or declaration centered on `shape.emplace_back`.
  **L883 CN**: 执行以 `shape.emplace_back` 为核心的调用或声明。
- **L884 EN**: Closes the current lexical scope or compound statement.
  **L884 CN**: 结束当前词法作用域或复合语句块。
- **L885 EN**: Executes a standalone statement or declaration: `++dimension;`.
  **L885 CN**: 执行一条独立语句或声明：`++dimension;`。
- **L886 EN**: Closes the current lexical scope or compound statement.
  **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L887 CN**: 开始 `if` 控制流语句并计算其条件。
- **L888 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L888 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 889-912

````cpp
      return (*this)(component->base());
    }
  }
  return shape;
}

auto GetShapeHelper::operator()(const CoarrayRef &coarrayRef) const -> Result {
  return (*this)(coarrayRef.base());
}

auto GetShapeHelper::operator()(const Substring &substring) const -> Result {
  return (*this)(substring.parent());
}

auto GetShapeHelper::operator()(const ProcedureRef &call) const -> Result {
  if (call.Rank() == 0) {
    return ScalarShape();
  } else if (call.IsElemental()) {
    // Use the shape of an actual array argument associated with a
    // non-OPTIONAL dummy object argument.
    if (context_) {
      if (auto chars{characteristics::Procedure::FromActuals(
              call.proc(), call.arguments(), *context_)}) {
        std::size_t j{0};
````
- **L889 EN**: Returns from the current function with `(*this)(component->base())`.
  **L889 CN**: 以 `(*this)(component->base())` 从当前函数返回。
- **L890 EN**: Closes the current lexical scope or compound statement.
  **L890 CN**: 结束当前词法作用域或复合语句块。
- **L891 EN**: Closes the current lexical scope or compound statement.
  **L891 CN**: 结束当前词法作用域或复合语句块。
- **L892 EN**: Returns from the current function with `shape`.
  **L892 CN**: 以 `shape` 从当前函数返回。
- **L893 EN**: Closes the current lexical scope or compound statement.
  **L893 CN**: 结束当前词法作用域或复合语句块。
- **L894 EN**: Blank line separating nearby declarations or logic blocks.
  **L894 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L895 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const CoarrayRef &coarrayRef) const -> Result {`.
  **L895 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const CoarrayRef &coarrayRef) const -> Result {`。
- **L896 EN**: Returns from the current function with `(*this)(coarrayRef.base())`.
  **L896 CN**: 以 `(*this)(coarrayRef.base())` 从当前函数返回。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L899 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const Substring &substring) const -> Result {`.
  **L899 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const Substring &substring) const -> Result {`。
- **L900 EN**: Returns from the current function with `(*this)(substring.parent())`.
  **L900 CN**: 以 `(*this)(substring.parent())` 从当前函数返回。
- **L901 EN**: Closes the current lexical scope or compound statement.
  **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L903 EN**: Starts a function, method, lambda, or structured scope: `auto GetShapeHelper::operator()(const ProcedureRef &call) const -> Result {`.
  **L903 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto GetShapeHelper::operator()(const ProcedureRef &call) const -> Result {`。
- **L904 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L904 CN**: 开始 `if` 控制流语句并计算其条件。
- **L905 EN**: Returns from the current function with `ScalarShape()`.
  **L905 CN**: 以 `ScalarShape()` 从当前函数返回。
- **L906 EN**: Transitions from the previous branch into an `else if` condition.
  **L906 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L907 EN**: Comment explains nearby logic, intent, or metadata: `Use the shape of an actual array argument associated with a`.
  **L907 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use the shape of an actual array argument associated with a`。
- **L908 EN**: Comment explains nearby logic, intent, or metadata: `non-OPTIONAL dummy object argument.`.
  **L908 CN**: 注释说明附近代码的逻辑、意图或元数据：`non-OPTIONAL dummy object argument.`。
- **L909 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L909 CN**: 开始 `if` 控制流语句并计算其条件。
- **L910 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L910 CN**: 开始 `if` 控制流语句并计算其条件。
- **L911 EN**: Starts a function, method, lambda, or structured scope: `call.proc(), call.arguments(), *context_)}) {`.
  **L911 CN**: 开始一个函数、方法、lambda 或结构化作用域：`call.proc(), call.arguments(), *context_)}) {`。
- **L912 EN**: Executes a standalone statement or declaration: `std::size_t j{0};`.
  **L912 CN**: 执行一条独立语句或声明：`std::size_t j{0};`。

### Lines 913-936

````cpp
        const ActualArgument *nonOptionalArrayArg{nullptr};
        int anyArrayArgRank{0};
        for (const auto &arg : call.arguments()) {
          if (arg && arg->Rank() > 0 && j < chars->dummyArguments.size()) {
            if (!anyArrayArgRank) {
              anyArrayArgRank = arg->Rank();
            } else if (arg->Rank() != anyArrayArgRank) {
              return std::nullopt; // error recovery
            }
            if (!nonOptionalArrayArg &&
                !chars->dummyArguments[j].IsOptional()) {
              nonOptionalArrayArg = &*arg;
            }
          }
          ++j;
        }
        if (anyArrayArgRank) {
          if (nonOptionalArrayArg) {
            return (*this)(*nonOptionalArrayArg);
          } else {
            // All dummy array arguments of the procedure are OPTIONAL.
            // We cannot take the shape from just any array argument,
            // because all of them might be OPTIONAL dummy arguments
            // of the caller. Return unknown shape ranked according
````
- **L913 EN**: Executes a standalone statement or declaration: `const ActualArgument *nonOptionalArrayArg{nullptr};`.
  **L913 CN**: 执行一条独立语句或声明：`const ActualArgument *nonOptionalArrayArg{nullptr};`。
- **L914 EN**: Executes a standalone statement or declaration: `int anyArrayArgRank{0};`.
  **L914 CN**: 执行一条独立语句或声明：`int anyArrayArgRank{0};`。
- **L915 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L915 CN**: 开始 `for` 控制流语句并计算其条件。
- **L916 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L916 CN**: 开始 `if` 控制流语句并计算其条件。
- **L917 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L917 CN**: 开始 `if` 控制流语句并计算其条件。
- **L918 EN**: Executes a call or declaration centered on `arg->Rank`.
  **L918 CN**: 执行以 `arg->Rank` 为核心的调用或声明。
- **L919 EN**: Transitions from the previous branch into an `else if` condition.
  **L919 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L920 EN**: Returns from the current function with `std::nullopt; // error recovery`.
  **L920 CN**: 以 `std::nullopt; // error recovery` 从当前函数返回。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L922 CN**: 开始 `if` 控制流语句并计算其条件。
- **L923 EN**: Starts a function, method, lambda, or structured scope: `!chars->dummyArguments[j].IsOptional()) {`.
  **L923 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!chars->dummyArguments[j].IsOptional()) {`。
- **L924 EN**: Executes a standalone statement or declaration: `nonOptionalArrayArg = &*arg;`.
  **L924 CN**: 执行一条独立语句或声明：`nonOptionalArrayArg = &*arg;`。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Closes the current lexical scope or compound statement.
  **L926 CN**: 结束当前词法作用域或复合语句块。
- **L927 EN**: Executes a standalone statement or declaration: `++j;`.
  **L927 CN**: 执行一条独立语句或声明：`++j;`。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。
- **L929 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L929 CN**: 开始 `if` 控制流语句并计算其条件。
- **L930 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L930 CN**: 开始 `if` 控制流语句并计算其条件。
- **L931 EN**: Returns from the current function with `(*this)(*nonOptionalArrayArg)`.
  **L931 CN**: 以 `(*this)(*nonOptionalArrayArg)` 从当前函数返回。
- **L932 EN**: Transitions from the previous branch into the alternative path.
  **L932 CN**: 从前一个分支过渡到备选路径。
- **L933 EN**: Comment explains nearby logic, intent, or metadata: `All dummy array arguments of the procedure are OPTIONAL.`.
  **L933 CN**: 注释说明附近代码的逻辑、意图或元数据：`All dummy array arguments of the procedure are OPTIONAL.`。
- **L934 EN**: Comment explains nearby logic, intent, or metadata: `We cannot take the shape from just any array argument,`.
  **L934 CN**: 注释说明附近代码的逻辑、意图或元数据：`We cannot take the shape from just any array argument,`。
- **L935 EN**: Comment explains nearby logic, intent, or metadata: `because all of them might be OPTIONAL dummy arguments`.
  **L935 CN**: 注释说明附近代码的逻辑、意图或元数据：`because all of them might be OPTIONAL dummy arguments`。
- **L936 EN**: Comment explains nearby logic, intent, or metadata: `of the caller. Return unknown shape ranked according`.
  **L936 CN**: 注释说明附近代码的逻辑、意图或元数据：`of the caller. Return unknown shape ranked according`。

### Lines 937-960

````cpp
            // to the last actual array argument.
            return Shape(anyArrayArgRank, MaybeExtentExpr{});
          }
        }
      }
    }
    return ScalarShape();
  } else if (const Symbol * symbol{call.proc().GetSymbol()}) {
    auto restorer{common::ScopedSet(useResultSymbolShape_, false)};
    return (*this)(*symbol);
  } else if (const auto *intrinsic{call.proc().GetSpecificIntrinsic()}) {
    if (intrinsic->name == "shape" || intrinsic->name == "lbound" ||
        intrinsic->name == "ubound") {
      // For LBOUND/UBOUND, these are the array-valued cases (no DIM=)
      if (!call.arguments().empty() && call.arguments().front()) {
        if (semantics::IsAssumedRank(*call.arguments().front())) {
          return Shape{MaybeExtentExpr{}};
        } else {
          return Shape{
              MaybeExtentExpr{ExtentExpr{call.arguments().front()->Rank()}}};
        }
      }
    } else if (intrinsic->name == "all" || intrinsic->name == "any" ||
        intrinsic->name == "count" || intrinsic->name == "iall" ||
````
- **L937 EN**: Comment explains nearby logic, intent, or metadata: `to the last actual array argument.`.
  **L937 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the last actual array argument.`。
- **L938 EN**: Returns from the current function with `Shape(anyArrayArgRank, MaybeExtentExpr{})`.
  **L938 CN**: 以 `Shape(anyArrayArgRank, MaybeExtentExpr{})` 从当前函数返回。
- **L939 EN**: Closes the current lexical scope or compound statement.
  **L939 CN**: 结束当前词法作用域或复合语句块。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Closes the current lexical scope or compound statement.
  **L941 CN**: 结束当前词法作用域或复合语句块。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Returns from the current function with `ScalarShape()`.
  **L943 CN**: 以 `ScalarShape()` 从当前函数返回。
- **L944 EN**: Transitions from the previous branch into an `else if` condition.
  **L944 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L945 EN**: Executes a call or declaration centered on `restorer{common::ScopedSet`.
  **L945 CN**: 执行以 `restorer{common::ScopedSet` 为核心的调用或声明。
- **L946 EN**: Returns from the current function with `(*this)(*symbol)`.
  **L946 CN**: 以 `(*this)(*symbol)` 从当前函数返回。
- **L947 EN**: Transitions from the previous branch into an `else if` condition.
  **L947 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L948 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L948 CN**: 开始 `if` 控制流语句并计算其条件。
- **L949 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "ubound") {`.
  **L949 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "ubound") {`。
- **L950 EN**: Comment explains nearby logic, intent, or metadata: `For LBOUND/UBOUND, these are the array-valued cases (no DIM=)`.
  **L950 CN**: 注释说明附近代码的逻辑、意图或元数据：`For LBOUND/UBOUND, these are the array-valued cases (no DIM=)`。
- **L951 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L951 CN**: 开始 `if` 控制流语句并计算其条件。
- **L952 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L952 CN**: 开始 `if` 控制流语句并计算其条件。
- **L953 EN**: Returns from the current function with `Shape{MaybeExtentExpr{}}`.
  **L953 CN**: 以 `Shape{MaybeExtentExpr{}}` 从当前函数返回。
- **L954 EN**: Transitions from the previous branch into the alternative path.
  **L954 CN**: 从前一个分支过渡到备选路径。
- **L955 EN**: Returns from the current function with `Shape{`.
  **L955 CN**: 以 `Shape{` 从当前函数返回。
- **L956 EN**: Executes a call or declaration centered on `MaybeExtentExpr{ExtentExpr{call.arguments`.
  **L956 CN**: 执行以 `MaybeExtentExpr{ExtentExpr{call.arguments` 为核心的调用或声明。
- **L957 EN**: Closes the current lexical scope or compound statement.
  **L957 CN**: 结束当前词法作用域或复合语句块。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Transitions from the previous branch into an `else if` condition.
  **L959 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L960 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "count" || intrinsic->name == "iall" ||`.
  **L960 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "count" || intrinsic->name == "iall" ||`。

### Lines 961-984

````cpp
        intrinsic->name == "iany" || intrinsic->name == "iparity" ||
        intrinsic->name == "maxval" || intrinsic->name == "minval" ||
        intrinsic->name == "norm2" || intrinsic->name == "parity" ||
        intrinsic->name == "product" || intrinsic->name == "sum") {
      // Reduction with DIM=
      if (call.arguments().size() >= 2) {
        auto arrayShape{
            (*this)(UnwrapExpr<Expr<SomeType>>(call.arguments().at(0)))};
        const auto *dimArg{UnwrapExpr<Expr<SomeType>>(call.arguments().at(1))};
        if (arrayShape && dimArg) {
          if (auto dim{ToInt64(*dimArg)}) {
            if (*dim >= 1 &&
                static_cast<std::size_t>(*dim) <= arrayShape->size()) {
              arrayShape->erase(arrayShape->begin() + (*dim - 1));
              return std::move(*arrayShape);
            }
          }
        }
      }
    } else if (intrinsic->name == "findloc" || intrinsic->name == "maxloc" ||
        intrinsic->name == "minloc") {
      std::size_t dimIndex{intrinsic->name == "findloc" ? 2u : 1u};
      if (call.arguments().size() > dimIndex) {
        if (auto arrayShape{
````
- **L961 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "iany" || intrinsic->name == "iparity" ||`.
  **L961 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "iany" || intrinsic->name == "iparity" ||`。
- **L962 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "maxval" || intrinsic->name == "minval" ||`.
  **L962 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "maxval" || intrinsic->name == "minval" ||`。
- **L963 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "norm2" || intrinsic->name == "parity" ||`.
  **L963 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "norm2" || intrinsic->name == "parity" ||`。
- **L964 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "product" || intrinsic->name == "sum") {`.
  **L964 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "product" || intrinsic->name == "sum") {`。
- **L965 EN**: Comment explains nearby logic, intent, or metadata: `Reduction with DIM=`.
  **L965 CN**: 注释说明附近代码的逻辑、意图或元数据：`Reduction with DIM=`。
- **L966 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L966 CN**: 开始 `if` 控制流语句并计算其条件。
- **L967 EN**: Continues the surrounding expression or declaration: `auto arrayShape{`.
  **L967 CN**: 继续构造周围的表达式或声明：`auto arrayShape{`。
- **L968 EN**: Executes a call or declaration centered on `statement`.
  **L968 CN**: 执行以 `statement` 为核心的调用或声明。
- **L969 EN**: Executes a call or declaration centered on `*dimArg{UnwrapExpr<Expr<SomeType>>`.
  **L969 CN**: 执行以 `*dimArg{UnwrapExpr<Expr<SomeType>>` 为核心的调用或声明。
- **L970 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L970 CN**: 开始 `if` 控制流语句并计算其条件。
- **L971 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L971 CN**: 开始 `if` 控制流语句并计算其条件。
- **L972 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L972 CN**: 开始 `if` 控制流语句并计算其条件。
- **L973 EN**: Starts a function, method, lambda, or structured scope: `static_cast<std::size_t>(*dim) <= arrayShape->size()) {`.
  **L973 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static_cast<std::size_t>(*dim) <= arrayShape->size()) {`。
- **L974 EN**: Executes a call or declaration centered on `arrayShape->erase`.
  **L974 CN**: 执行以 `arrayShape->erase` 为核心的调用或声明。
- **L975 EN**: Returns from the current function with `std::move(*arrayShape)`.
  **L975 CN**: 以 `std::move(*arrayShape)` 从当前函数返回。
- **L976 EN**: Closes the current lexical scope or compound statement.
  **L976 CN**: 结束当前词法作用域或复合语句块。
- **L977 EN**: Closes the current lexical scope or compound statement.
  **L977 CN**: 结束当前词法作用域或复合语句块。
- **L978 EN**: Closes the current lexical scope or compound statement.
  **L978 CN**: 结束当前词法作用域或复合语句块。
- **L979 EN**: Closes the current lexical scope or compound statement.
  **L979 CN**: 结束当前词法作用域或复合语句块。
- **L980 EN**: Transitions from the previous branch into an `else if` condition.
  **L980 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L981 EN**: Continues the surrounding expression or declaration: `intrinsic->name == "minloc") {`.
  **L981 CN**: 继续构造周围的表达式或声明：`intrinsic->name == "minloc") {`。
- **L982 EN**: Executes a standalone statement or declaration: `std::size_t dimIndex{intrinsic->name == "findloc" ? 2u : 1u};`.
  **L982 CN**: 执行一条独立语句或声明：`std::size_t dimIndex{intrinsic->name == "findloc" ? 2u : 1u};`。
- **L983 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L983 CN**: 开始 `if` 控制流语句并计算其条件。
- **L984 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L984 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 985-1008

````cpp
                (*this)(UnwrapExpr<Expr<SomeType>>(call.arguments().at(0)))}) {
          auto rank{static_cast<int>(arrayShape->size())};
          if (const auto *dimArg{
                  UnwrapExpr<Expr<SomeType>>(call.arguments()[dimIndex])}) {
            auto dim{ToInt64(*dimArg)};
            if (dim && *dim >= 1 && *dim <= rank) {
              arrayShape->erase(arrayShape->begin() + (*dim - 1));
              return std::move(*arrayShape);
            }
          } else {
            // xxxLOC(no DIM=) result is vector(1:RANK(ARRAY=))
            return Shape{ExtentExpr{rank}};
          }
        }
      }
    } else if (intrinsic->name == "cshift" || intrinsic->name == "eoshift") {
      if (!call.arguments().empty()) {
        return (*this)(call.arguments()[0]);
      }
    } else if (intrinsic->name == "lcobound" || intrinsic->name == "ucobound") {
      if (call.arguments().size() == 3 && !call.arguments().at(1).has_value()) {
        return Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))});
      }
    } else if (intrinsic->name == "matmul") {
````
- **L985 EN**: Starts a function, method, lambda, or structured scope: `(*this)(UnwrapExpr<Expr<SomeType>>(call.arguments().at(0)))}) {`.
  **L985 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(*this)(UnwrapExpr<Expr<SomeType>>(call.arguments().at(0)))}) {`。
- **L986 EN**: Executes a call or declaration centered on `rank{static_cast<int>`.
  **L986 CN**: 执行以 `rank{static_cast<int>` 为核心的调用或声明。
- **L987 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L987 CN**: 开始 `if` 控制流语句并计算其条件。
- **L988 EN**: Starts a function, method, lambda, or structured scope: `UnwrapExpr<Expr<SomeType>>(call.arguments()[dimIndex])}) {`.
  **L988 CN**: 开始一个函数、方法、lambda 或结构化作用域：`UnwrapExpr<Expr<SomeType>>(call.arguments()[dimIndex])}) {`。
- **L989 EN**: Executes a call or declaration centered on `dim{ToInt64`.
  **L989 CN**: 执行以 `dim{ToInt64` 为核心的调用或声明。
- **L990 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L990 CN**: 开始 `if` 控制流语句并计算其条件。
- **L991 EN**: Executes a call or declaration centered on `arrayShape->erase`.
  **L991 CN**: 执行以 `arrayShape->erase` 为核心的调用或声明。
- **L992 EN**: Returns from the current function with `std::move(*arrayShape)`.
  **L992 CN**: 以 `std::move(*arrayShape)` 从当前函数返回。
- **L993 EN**: Closes the current lexical scope or compound statement.
  **L993 CN**: 结束当前词法作用域或复合语句块。
- **L994 EN**: Transitions from the previous branch into the alternative path.
  **L994 CN**: 从前一个分支过渡到备选路径。
- **L995 EN**: Comment explains nearby logic, intent, or metadata: `xxxLOC(no DIM=) result is vector(1:RANK(ARRAY=))`.
  **L995 CN**: 注释说明附近代码的逻辑、意图或元数据：`xxxLOC(no DIM=) result is vector(1:RANK(ARRAY=))`。
- **L996 EN**: Returns from the current function with `Shape{ExtentExpr{rank}}`.
  **L996 CN**: 以 `Shape{ExtentExpr{rank}}` 从当前函数返回。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Closes the current lexical scope or compound statement.
  **L998 CN**: 结束当前词法作用域或复合语句块。
- **L999 EN**: Closes the current lexical scope or compound statement.
  **L999 CN**: 结束当前词法作用域或复合语句块。
- **L1000 EN**: Transitions from the previous branch into an `else if` condition.
  **L1000 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1001 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1001 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1002 EN**: Returns from the current function with `(*this)(call.arguments()[0])`.
  **L1002 CN**: 以 `(*this)(call.arguments()[0])` 从当前函数返回。
- **L1003 EN**: Closes the current lexical scope or compound statement.
  **L1003 CN**: 结束当前词法作用域或复合语句块。
- **L1004 EN**: Transitions from the previous branch into an `else if` condition.
  **L1004 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1005 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1005 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1006 EN**: Returns from the current function with `Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))})`.
  **L1006 CN**: 以 `Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))})` 从当前函数返回。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Transitions from the previous branch into an `else if` condition.
  **L1008 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 1009-1032

````cpp
      if (call.arguments().size() == 2) {
        if (auto ashape{(*this)(call.arguments()[0])}) {
          if (auto bshape{(*this)(call.arguments()[1])}) {
            if (ashape->size() == 1 && bshape->size() == 2) {
              bshape->erase(bshape->begin());
              return std::move(*bshape); // matmul(vector, matrix)
            } else if (ashape->size() == 2 && bshape->size() == 1) {
              ashape->pop_back();
              return std::move(*ashape); // matmul(matrix, vector)
            } else if (ashape->size() == 2 && bshape->size() == 2) {
              (*ashape)[1] = std::move((*bshape)[1]);
              return std::move(*ashape); // matmul(matrix, matrix)
            }
          }
        }
      }
    } else if (intrinsic->name == "pack") {
      if (call.arguments().size() >= 3 && call.arguments().at(2)) {
        // SHAPE(PACK(,,VECTOR=v)) -> SHAPE(v)
        return (*this)(call.arguments().at(2));
      } else if (call.arguments().size() >= 2 && context_) {
        if (auto maskShape{(*this)(call.arguments().at(1))}) {
          if (maskShape->size() == 0) {
            // Scalar MASK= -> [MERGE(SIZE(ARRAY=), 0, mask)]
````
- **L1009 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1009 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1010 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1010 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1011 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1011 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1012 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1012 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1013 EN**: Executes a call or declaration centered on `bshape->erase`.
  **L1013 CN**: 执行以 `bshape->erase` 为核心的调用或声明。
- **L1014 EN**: Returns from the current function with `std::move(*bshape); // matmul(vector, matrix)`.
  **L1014 CN**: 以 `std::move(*bshape); // matmul(vector, matrix)` 从当前函数返回。
- **L1015 EN**: Transitions from the previous branch into an `else if` condition.
  **L1015 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1016 EN**: Executes a call or declaration centered on `ashape->pop_back`.
  **L1016 CN**: 执行以 `ashape->pop_back` 为核心的调用或声明。
- **L1017 EN**: Returns from the current function with `std::move(*ashape); // matmul(matrix, vector)`.
  **L1017 CN**: 以 `std::move(*ashape); // matmul(matrix, vector)` 从当前函数返回。
- **L1018 EN**: Transitions from the previous branch into an `else if` condition.
  **L1018 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1019 EN**: Executes a call or declaration centered on `statement`.
  **L1019 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1020 EN**: Returns from the current function with `std::move(*ashape); // matmul(matrix, matrix)`.
  **L1020 CN**: 以 `std::move(*ashape); // matmul(matrix, matrix)` 从当前函数返回。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Closes the current lexical scope or compound statement.
  **L1023 CN**: 结束当前词法作用域或复合语句块。
- **L1024 EN**: Closes the current lexical scope or compound statement.
  **L1024 CN**: 结束当前词法作用域或复合语句块。
- **L1025 EN**: Transitions from the previous branch into an `else if` condition.
  **L1025 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1026 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1026 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1027 EN**: Comment explains nearby logic, intent, or metadata: `SHAPE(PACK(,,VECTOR=v)) -> SHAPE(v)`.
  **L1027 CN**: 注释说明附近代码的逻辑、意图或元数据：`SHAPE(PACK(,,VECTOR=v)) -> SHAPE(v)`。
- **L1028 EN**: Returns from the current function with `(*this)(call.arguments().at(2))`.
  **L1028 CN**: 以 `(*this)(call.arguments().at(2))` 从当前函数返回。
- **L1029 EN**: Transitions from the previous branch into an `else if` condition.
  **L1029 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1030 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1030 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1031 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1031 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1032 EN**: Comment explains nearby logic, intent, or metadata: `Scalar MASK= -> [MERGE(SIZE(ARRAY=), 0, mask)]`.
  **L1032 CN**: 注释说明附近代码的逻辑、意图或元数据：`Scalar MASK= -> [MERGE(SIZE(ARRAY=), 0, mask)]`。

### Lines 1033-1056

````cpp
            if (auto arrayShape{(*this)(call.arguments().at(0))}) {
              if (auto arraySize{GetSize(std::move(*arrayShape))}) {
                ActualArguments toMerge{
                    ActualArgument{AsGenericExpr(std::move(*arraySize))},
                    ActualArgument{AsGenericExpr(ExtentExpr{0})},
                    common::Clone(call.arguments().at(1))};
                auto specific{context_->intrinsics().Probe(
                    CallCharacteristics{"merge"}, toMerge, *context_)};
                CHECK(specific);
                return Shape{ExtentExpr{FunctionRef<ExtentType>{
                    ProcedureDesignator{std::move(specific->specificIntrinsic)},
                    std::move(specific->arguments)}}};
              }
            }
          } else {
            // Non-scalar MASK= -> [COUNT(mask, KIND=extent_kind)]
            ActualArgument kindArg{
                AsGenericExpr(Constant<ExtentType>{ExtentType::kind})};
            kindArg.set_keyword(context_->SaveTempName("kind"));
            ActualArguments toCount{
                ActualArgument{common::Clone(
                    DEREF(call.arguments().at(1).value().UnwrapExpr()))},
                std::move(kindArg)};
            auto specific{context_->intrinsics().Probe(
````
- **L1033 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1033 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1034 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1034 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1035 EN**: Continues the surrounding expression or declaration: `ActualArguments toMerge{`.
  **L1035 CN**: 继续构造周围的表达式或声明：`ActualArguments toMerge{`。
- **L1036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ActualArgument{AsGenericExpr(std::move(*arraySize))},`.
  **L1036 CN**: 继续一个多行参数列表、初始化器或聚合项：`ActualArgument{AsGenericExpr(std::move(*arraySize))},`。
- **L1037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ActualArgument{AsGenericExpr(ExtentExpr{0})},`.
  **L1037 CN**: 继续一个多行参数列表、初始化器或聚合项：`ActualArgument{AsGenericExpr(ExtentExpr{0})},`。
- **L1038 EN**: Executes a call or declaration centered on `common::Clone`.
  **L1038 CN**: 执行以 `common::Clone` 为核心的调用或声明。
- **L1039 EN**: Continues logic associated with callable symbol `intrinsics`.
  **L1039 CN**: 继续与可调用符号 `intrinsics` 相关的逻辑。
- **L1040 EN**: Executes a standalone statement or declaration: `CallCharacteristics{"merge"}, toMerge, *context_)};`.
  **L1040 CN**: 执行一条独立语句或声明：`CallCharacteristics{"merge"}, toMerge, *context_)};`。
- **L1041 EN**: Executes a call or declaration centered on `CHECK`.
  **L1041 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1042 EN**: Returns from the current function with `Shape{ExtentExpr{FunctionRef<ExtentType>{`.
  **L1042 CN**: 以 `Shape{ExtentExpr{FunctionRef<ExtentType>{` 从当前函数返回。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ProcedureDesignator{std::move(specific->specificIntrinsic)},`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`ProcedureDesignator{std::move(specific->specificIntrinsic)},`。
- **L1044 EN**: Executes a call or declaration centered on `std::move`.
  **L1044 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1045 EN**: Closes the current lexical scope or compound statement.
  **L1045 CN**: 结束当前词法作用域或复合语句块。
- **L1046 EN**: Closes the current lexical scope or compound statement.
  **L1046 CN**: 结束当前词法作用域或复合语句块。
- **L1047 EN**: Transitions from the previous branch into the alternative path.
  **L1047 CN**: 从前一个分支过渡到备选路径。
- **L1048 EN**: Comment explains nearby logic, intent, or metadata: `Non-scalar MASK= -> [COUNT(mask, KIND=extent_kind)]`.
  **L1048 CN**: 注释说明附近代码的逻辑、意图或元数据：`Non-scalar MASK= -> [COUNT(mask, KIND=extent_kind)]`。
- **L1049 EN**: Continues the surrounding expression or declaration: `ActualArgument kindArg{`.
  **L1049 CN**: 继续构造周围的表达式或声明：`ActualArgument kindArg{`。
- **L1050 EN**: Executes a call or declaration centered on `AsGenericExpr`.
  **L1050 CN**: 执行以 `AsGenericExpr` 为核心的调用或声明。
- **L1051 EN**: Executes a call or declaration centered on `kindArg.set_keyword`.
  **L1051 CN**: 执行以 `kindArg.set_keyword` 为核心的调用或声明。
- **L1052 EN**: Continues the surrounding expression or declaration: `ActualArguments toCount{`.
  **L1052 CN**: 继续构造周围的表达式或声明：`ActualArguments toCount{`。
- **L1053 EN**: Continues logic associated with callable symbol `Clone`.
  **L1053 CN**: 继续与可调用符号 `Clone` 相关的逻辑。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DEREF(call.arguments().at(1).value().UnwrapExpr()))},`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`DEREF(call.arguments().at(1).value().UnwrapExpr()))},`。
- **L1055 EN**: Executes a call or declaration centered on `std::move`.
  **L1055 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1056 EN**: Continues logic associated with callable symbol `intrinsics`.
  **L1056 CN**: 继续与可调用符号 `intrinsics` 相关的逻辑。

### Lines 1057-1080

````cpp
                CallCharacteristics{"count"}, toCount, *context_)};
            CHECK(specific);
            return Shape{ExtentExpr{FunctionRef<ExtentType>{
                ProcedureDesignator{std::move(specific->specificIntrinsic)},
                std::move(specific->arguments)}}};
          }
        }
      }
    } else if (intrinsic->name == "reshape") {
      if (call.arguments().size() >= 2 && call.arguments().at(1)) {
        // SHAPE(RESHAPE(array,shape)) -> shape
        if (const auto *shapeExpr{
                call.arguments().at(1).value().UnwrapExpr()}) {
          auto shapeArg{std::get<Expr<SomeInteger>>(shapeExpr->u)};
          if (auto result{AsShapeResult(
                  ConvertToType<ExtentType>(std::move(shapeArg)))}) {
            return result;
          }
        }
      }
    } else if (intrinsic->name == "spread") {
      // SHAPE(SPREAD(ARRAY,DIM,NCOPIES)) = SHAPE(ARRAY) with MAX(0,NCOPIES)
      // inserted at position DIM.
      if (call.arguments().size() == 3) {
````
- **L1057 EN**: Executes a standalone statement or declaration: `CallCharacteristics{"count"}, toCount, *context_)};`.
  **L1057 CN**: 执行一条独立语句或声明：`CallCharacteristics{"count"}, toCount, *context_)};`。
- **L1058 EN**: Executes a call or declaration centered on `CHECK`.
  **L1058 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1059 EN**: Returns from the current function with `Shape{ExtentExpr{FunctionRef<ExtentType>{`.
  **L1059 CN**: 以 `Shape{ExtentExpr{FunctionRef<ExtentType>{` 从当前函数返回。
- **L1060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ProcedureDesignator{std::move(specific->specificIntrinsic)},`.
  **L1060 CN**: 继续一个多行参数列表、初始化器或聚合项：`ProcedureDesignator{std::move(specific->specificIntrinsic)},`。
- **L1061 EN**: Executes a call or declaration centered on `std::move`.
  **L1061 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1062 EN**: Closes the current lexical scope or compound statement.
  **L1062 CN**: 结束当前词法作用域或复合语句块。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Closes the current lexical scope or compound statement.
  **L1064 CN**: 结束当前词法作用域或复合语句块。
- **L1065 EN**: Transitions from the previous branch into an `else if` condition.
  **L1065 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1066 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1066 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1067 EN**: Comment explains nearby logic, intent, or metadata: `SHAPE(RESHAPE(array,shape)) -> shape`.
  **L1067 CN**: 注释说明附近代码的逻辑、意图或元数据：`SHAPE(RESHAPE(array,shape)) -> shape`。
- **L1068 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1068 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1069 EN**: Starts a function, method, lambda, or structured scope: `call.arguments().at(1).value().UnwrapExpr()}) {`.
  **L1069 CN**: 开始一个函数、方法、lambda 或结构化作用域：`call.arguments().at(1).value().UnwrapExpr()}) {`。
- **L1070 EN**: Executes a call or declaration centered on `shapeArg{std::get<Expr<SomeInteger>>`.
  **L1070 CN**: 执行以 `shapeArg{std::get<Expr<SomeInteger>>` 为核心的调用或声明。
- **L1071 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1071 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1072 EN**: Starts a function, method, lambda, or structured scope: `ConvertToType<ExtentType>(std::move(shapeArg)))}) {`.
  **L1072 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ConvertToType<ExtentType>(std::move(shapeArg)))}) {`。
- **L1073 EN**: Returns from the current function with `result`.
  **L1073 CN**: 以 `result` 从当前函数返回。
- **L1074 EN**: Closes the current lexical scope or compound statement.
  **L1074 CN**: 结束当前词法作用域或复合语句块。
- **L1075 EN**: Closes the current lexical scope or compound statement.
  **L1075 CN**: 结束当前词法作用域或复合语句块。
- **L1076 EN**: Closes the current lexical scope or compound statement.
  **L1076 CN**: 结束当前词法作用域或复合语句块。
- **L1077 EN**: Transitions from the previous branch into an `else if` condition.
  **L1077 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1078 EN**: Comment explains nearby logic, intent, or metadata: `SHAPE(SPREAD(ARRAY,DIM,NCOPIES)) = SHAPE(ARRAY) with MAX(0,NCOPIES)`.
  **L1078 CN**: 注释说明附近代码的逻辑、意图或元数据：`SHAPE(SPREAD(ARRAY,DIM,NCOPIES)) = SHAPE(ARRAY) with MAX(0,NCOPIES)`。
- **L1079 EN**: Comment explains nearby logic, intent, or metadata: `inserted at position DIM.`.
  **L1079 CN**: 注释说明附近代码的逻辑、意图或元数据：`inserted at position DIM.`。
- **L1080 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1080 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1081-1104

````cpp
        auto arrayShape{
            (*this)(UnwrapExpr<Expr<SomeType>>(call.arguments().at(0)))};
        const auto *dimArg{UnwrapExpr<Expr<SomeType>>(call.arguments().at(1))};
        const auto *nCopies{
            UnwrapExpr<Expr<SomeInteger>>(call.arguments().at(2))};
        if (arrayShape && dimArg && nCopies) {
          if (auto dim{ToInt64(*dimArg)}) {
            if (*dim >= 1 &&
                static_cast<std::size_t>(*dim) <= arrayShape->size() + 1) {
              arrayShape->emplace(arrayShape->begin() + *dim - 1,
                  Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},
                      ConvertToType<ExtentType>(common::Clone(*nCopies))});
              return std::move(*arrayShape);
            }
          }
        }
      }
    } else if (intrinsic->name == "transfer") {
      if (call.arguments().size() == 3 && call.arguments().at(2)) {
        // SIZE= is present; shape is vector [SIZE=]
        if (const auto *size{
                UnwrapExpr<Expr<SomeInteger>>(call.arguments().at(2))}) {
          return Shape{
              MaybeExtentExpr{ConvertToType<ExtentType>(common::Clone(*size))}};
````
- **L1081 EN**: Continues the surrounding expression or declaration: `auto arrayShape{`.
  **L1081 CN**: 继续构造周围的表达式或声明：`auto arrayShape{`。
- **L1082 EN**: Executes a call or declaration centered on `statement`.
  **L1082 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1083 EN**: Executes a call or declaration centered on `*dimArg{UnwrapExpr<Expr<SomeType>>`.
  **L1083 CN**: 执行以 `*dimArg{UnwrapExpr<Expr<SomeType>>` 为核心的调用或声明。
- **L1084 EN**: Continues the surrounding expression or declaration: `const auto *nCopies{`.
  **L1084 CN**: 继续构造周围的表达式或声明：`const auto *nCopies{`。
- **L1085 EN**: Executes a call or declaration centered on `UnwrapExpr<Expr<SomeInteger>>`.
  **L1085 CN**: 执行以 `UnwrapExpr<Expr<SomeInteger>>` 为核心的调用或声明。
- **L1086 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1086 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1087 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1087 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1088 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1088 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1089 EN**: Starts a function, method, lambda, or structured scope: `static_cast<std::size_t>(*dim) <= arrayShape->size() + 1) {`.
  **L1089 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static_cast<std::size_t>(*dim) <= arrayShape->size() + 1) {`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `arrayShape->emplace(arrayShape->begin() + *dim - 1,`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`arrayShape->emplace(arrayShape->begin() + *dim - 1,`。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},`.
  **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`Extremum<SubscriptInteger>{Ordering::Greater, ExtentExpr{0},`。
- **L1092 EN**: Executes a call or declaration centered on `ConvertToType<ExtentType>`.
  **L1092 CN**: 执行以 `ConvertToType<ExtentType>` 为核心的调用或声明。
- **L1093 EN**: Returns from the current function with `std::move(*arrayShape)`.
  **L1093 CN**: 以 `std::move(*arrayShape)` 从当前函数返回。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Closes the current lexical scope or compound statement.
  **L1095 CN**: 结束当前词法作用域或复合语句块。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Closes the current lexical scope or compound statement.
  **L1097 CN**: 结束当前词法作用域或复合语句块。
- **L1098 EN**: Transitions from the previous branch into an `else if` condition.
  **L1098 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1099 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1099 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1100 EN**: Comment explains nearby logic, intent, or metadata: `SIZE= is present; shape is vector [SIZE=]`.
  **L1100 CN**: 注释说明附近代码的逻辑、意图或元数据：`SIZE= is present; shape is vector [SIZE=]`。
- **L1101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1102 EN**: Starts a function, method, lambda, or structured scope: `UnwrapExpr<Expr<SomeInteger>>(call.arguments().at(2))}) {`.
  **L1102 CN**: 开始一个函数、方法、lambda 或结构化作用域：`UnwrapExpr<Expr<SomeInteger>>(call.arguments().at(2))}) {`。
- **L1103 EN**: Returns from the current function with `Shape{`.
  **L1103 CN**: 以 `Shape{` 从当前函数返回。
- **L1104 EN**: Executes a call or declaration centered on `MaybeExtentExpr{ConvertToType<ExtentType>`.
  **L1104 CN**: 执行以 `MaybeExtentExpr{ConvertToType<ExtentType>` 为核心的调用或声明。

### Lines 1105-1128

````cpp
        }
      } else if (context_) {
        if (auto moldTypeAndShape{characteristics::TypeAndShape::Characterize(
                call.arguments().at(1), *context_)}) {
          if (moldTypeAndShape->Rank() == 0) {
            // SIZE= is absent and MOLD= is scalar: result is scalar
            return ScalarShape();
          } else {
            // SIZE= is absent and MOLD= is array: result is vector whose
            // length is determined by sizes of types.  See 16.9.193p4 case(ii).
            // Note that if sourceBytes is not known to be empty, we
            // can fold only when moldElementBytes is known to not be zero;
            // the most general case risks a division by zero otherwise.
            if (auto sourceTypeAndShape{
                    characteristics::TypeAndShape::Characterize(
                        call.arguments().at(0), *context_)}) {
              if (auto sourceBytes{
                      sourceTypeAndShape->MeasureSizeInBytes(*context_)}) {
                *sourceBytes = Fold(*context_, std::move(*sourceBytes));
                if (auto sourceBytesConst{ToInt64(*sourceBytes)}) {
                  if (*sourceBytesConst == 0) {
                    return Shape{ExtentExpr{0}};
                  }
                }
````
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Transitions from the previous branch into an `else if` condition.
  **L1106 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1107 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1107 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1108 EN**: Starts a function, method, lambda, or structured scope: `call.arguments().at(1), *context_)}) {`.
  **L1108 CN**: 开始一个函数、方法、lambda 或结构化作用域：`call.arguments().at(1), *context_)}) {`。
- **L1109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1110 EN**: Comment explains nearby logic, intent, or metadata: `SIZE= is absent and MOLD= is scalar: result is scalar`.
  **L1110 CN**: 注释说明附近代码的逻辑、意图或元数据：`SIZE= is absent and MOLD= is scalar: result is scalar`。
- **L1111 EN**: Returns from the current function with `ScalarShape()`.
  **L1111 CN**: 以 `ScalarShape()` 从当前函数返回。
- **L1112 EN**: Transitions from the previous branch into the alternative path.
  **L1112 CN**: 从前一个分支过渡到备选路径。
- **L1113 EN**: Comment explains nearby logic, intent, or metadata: `SIZE= is absent and MOLD= is array: result is vector whose`.
  **L1113 CN**: 注释说明附近代码的逻辑、意图或元数据：`SIZE= is absent and MOLD= is array: result is vector whose`。
- **L1114 EN**: Comment explains nearby logic, intent, or metadata: `length is determined by sizes of types.  See 16.9.193p4 case(ii).`.
  **L1114 CN**: 注释说明附近代码的逻辑、意图或元数据：`length is determined by sizes of types.  See 16.9.193p4 case(ii).`。
- **L1115 EN**: Comment explains nearby logic, intent, or metadata: `Note that if sourceBytes is not known to be empty, we`.
  **L1115 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that if sourceBytes is not known to be empty, we`。
- **L1116 EN**: Comment explains nearby logic, intent, or metadata: `can fold only when moldElementBytes is known to not be zero;`.
  **L1116 CN**: 注释说明附近代码的逻辑、意图或元数据：`can fold only when moldElementBytes is known to not be zero;`。
- **L1117 EN**: Comment explains nearby logic, intent, or metadata: `the most general case risks a division by zero otherwise.`.
  **L1117 CN**: 注释说明附近代码的逻辑、意图或元数据：`the most general case risks a division by zero otherwise.`。
- **L1118 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1118 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1119 EN**: Continues logic associated with callable symbol `Characterize`.
  **L1119 CN**: 继续与可调用符号 `Characterize` 相关的逻辑。
- **L1120 EN**: Starts a function, method, lambda, or structured scope: `call.arguments().at(0), *context_)}) {`.
  **L1120 CN**: 开始一个函数、方法、lambda 或结构化作用域：`call.arguments().at(0), *context_)}) {`。
- **L1121 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1121 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1122 EN**: Starts a function, method, lambda, or structured scope: `sourceTypeAndShape->MeasureSizeInBytes(*context_)}) {`.
  **L1122 CN**: 开始一个函数、方法、lambda 或结构化作用域：`sourceTypeAndShape->MeasureSizeInBytes(*context_)}) {`。
- **L1123 EN**: Comment explains nearby logic, intent, or metadata: `sourceBytes = Fold(*context_, std::move(*sourceBytes));`.
  **L1123 CN**: 注释说明附近代码的逻辑、意图或元数据：`sourceBytes = Fold(*context_, std::move(*sourceBytes));`。
- **L1124 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1124 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1125 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1125 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1126 EN**: Returns from the current function with `Shape{ExtentExpr{0}}`.
  **L1126 CN**: 以 `Shape{ExtentExpr{0}}` 从当前函数返回。
- **L1127 EN**: Closes the current lexical scope or compound statement.
  **L1127 CN**: 结束当前词法作用域或复合语句块。
- **L1128 EN**: Closes the current lexical scope or compound statement.
  **L1128 CN**: 结束当前词法作用域或复合语句块。

### Lines 1129-1152

````cpp
                if (auto moldElementBytes{
                        moldTypeAndShape->MeasureElementSizeInBytes(
                            *context_, true)}) {
                  *moldElementBytes =
                      Fold(*context_, std::move(*moldElementBytes));
                  auto moldElementBytesConst{ToInt64(*moldElementBytes)};
                  if (moldElementBytesConst && *moldElementBytesConst != 0) {
                    ExtentExpr extent{Fold(*context_,
                        (std::move(*sourceBytes) +
                            common::Clone(*moldElementBytes) - ExtentExpr{1}) /
                            common::Clone(*moldElementBytes))};
                    return Shape{MaybeExtentExpr{std::move(extent)}};
                  }
                }
              }
            }
          }
        }
      }
    } else if (intrinsic->name == "this_image") {
      if (call.arguments().size() == 2) {
        // THIS_IMAGE(coarray, no DIM, [TEAM])
        return Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))});
      }
````
- **L1129 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1129 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1130 EN**: Continues logic associated with callable symbol `MeasureElementSizeInBytes`.
  **L1130 CN**: 继续与可调用符号 `MeasureElementSizeInBytes` 相关的逻辑。
- **L1131 EN**: Comment explains nearby logic, intent, or metadata: `context_, true)}) {`.
  **L1131 CN**: 注释说明附近代码的逻辑、意图或元数据：`context_, true)}) {`。
- **L1132 EN**: Comment explains nearby logic, intent, or metadata: `moldElementBytes =`.
  **L1132 CN**: 注释说明附近代码的逻辑、意图或元数据：`moldElementBytes =`。
- **L1133 EN**: Executes a call or declaration centered on `Fold`.
  **L1133 CN**: 执行以 `Fold` 为核心的调用或声明。
- **L1134 EN**: Executes a call or declaration centered on `moldElementBytesConst{ToInt64`.
  **L1134 CN**: 执行以 `moldElementBytesConst{ToInt64` 为核心的调用或声明。
- **L1135 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1135 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ExtentExpr extent{Fold(*context_,`.
  **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`ExtentExpr extent{Fold(*context_,`。
- **L1137 EN**: Continues logic associated with callable symbol `move`.
  **L1137 CN**: 继续与可调用符号 `move` 相关的逻辑。
- **L1138 EN**: Continues logic associated with callable symbol `Clone`.
  **L1138 CN**: 继续与可调用符号 `Clone` 相关的逻辑。
- **L1139 EN**: Executes a call or declaration centered on `common::Clone`.
  **L1139 CN**: 执行以 `common::Clone` 为核心的调用或声明。
- **L1140 EN**: Returns from the current function with `Shape{MaybeExtentExpr{std::move(extent)}}`.
  **L1140 CN**: 以 `Shape{MaybeExtentExpr{std::move(extent)}}` 从当前函数返回。
- **L1141 EN**: Closes the current lexical scope or compound statement.
  **L1141 CN**: 结束当前词法作用域或复合语句块。
- **L1142 EN**: Closes the current lexical scope or compound statement.
  **L1142 CN**: 结束当前词法作用域或复合语句块。
- **L1143 EN**: Closes the current lexical scope or compound statement.
  **L1143 CN**: 结束当前词法作用域或复合语句块。
- **L1144 EN**: Closes the current lexical scope or compound statement.
  **L1144 CN**: 结束当前词法作用域或复合语句块。
- **L1145 EN**: Closes the current lexical scope or compound statement.
  **L1145 CN**: 结束当前词法作用域或复合语句块。
- **L1146 EN**: Closes the current lexical scope or compound statement.
  **L1146 CN**: 结束当前词法作用域或复合语句块。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Transitions from the previous branch into an `else if` condition.
  **L1148 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1150 EN**: Comment explains nearby logic, intent, or metadata: `THIS_IMAGE(coarray, no DIM, [TEAM])`.
  **L1150 CN**: 注释说明附近代码的逻辑、意图或元数据：`THIS_IMAGE(coarray, no DIM, [TEAM])`。
- **L1151 EN**: Returns from the current function with `Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))})`.
  **L1151 CN**: 以 `Shape(1, ExtentExpr{GetCorank(call.arguments().at(0))})` 从当前函数返回。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````cpp
    } else if (intrinsic->name == "transpose") {
      if (call.arguments().size() >= 1) {
        if (auto shape{(*this)(call.arguments().at(0))}) {
          if (shape->size() == 2) {
            std::swap((*shape)[0], (*shape)[1]);
            return shape;
          }
        }
      }
    } else if (intrinsic->name == "unpack") {
      if (call.arguments().size() >= 2) {
        return (*this)(call.arguments()[1]); // MASK=
      }
    } else if (intrinsic->characteristics.value().attrs.test(
                   characteristics::Procedure::Attr::NullPointer) ||
        intrinsic->characteristics.value().attrs.test(
            characteristics::Procedure::Attr::NullAllocatable)) { // NULL(MOLD=)
      return (*this)(call.arguments());
    } else {
      // TODO: shapes of other non-elemental intrinsic results
    }
  }
  // The rank is always known even if the extents are not.
  return Shape(static_cast<std::size_t>(call.Rank()), MaybeExtentExpr{});
````
- **L1153 EN**: Transitions from the previous branch into an `else if` condition.
  **L1153 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1155 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1155 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1156 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1156 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1157 EN**: Executes a call or declaration centered on `std::swap`.
  **L1157 CN**: 执行以 `std::swap` 为核心的调用或声明。
- **L1158 EN**: Returns from the current function with `shape`.
  **L1158 CN**: 以 `shape` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Closes the current lexical scope or compound statement.
  **L1161 CN**: 结束当前词法作用域或复合语句块。
- **L1162 EN**: Transitions from the previous branch into an `else if` condition.
  **L1162 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1163 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1163 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1164 EN**: Returns from the current function with `(*this)(call.arguments()[1]); // MASK=`.
  **L1164 CN**: 以 `(*this)(call.arguments()[1]); // MASK=` 从当前函数返回。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Transitions from the previous branch into an `else if` condition.
  **L1166 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1167 EN**: Continues the surrounding expression or declaration: `characteristics::Procedure::Attr::NullPointer) ||`.
  **L1167 CN**: 继续构造周围的表达式或声明：`characteristics::Procedure::Attr::NullPointer) ||`。
- **L1168 EN**: Continues logic associated with callable symbol `value`.
  **L1168 CN**: 继续与可调用符号 `value` 相关的逻辑。
- **L1169 EN**: Continues logic associated with callable symbol `NULL`.
  **L1169 CN**: 继续与可调用符号 `NULL` 相关的逻辑。
- **L1170 EN**: Returns from the current function with `(*this)(call.arguments())`.
  **L1170 CN**: 以 `(*this)(call.arguments())` 从当前函数返回。
- **L1171 EN**: Transitions from the previous branch into the alternative path.
  **L1171 CN**: 从前一个分支过渡到备选路径。
- **L1172 EN**: Comment records a pending task or caution: `TODO: shapes of other non-elemental intrinsic results`.
  **L1172 CN**: 注释记录待办事项或注意点：`TODO: shapes of other non-elemental intrinsic results`。
- **L1173 EN**: Closes the current lexical scope or compound statement.
  **L1173 CN**: 结束当前词法作用域或复合语句块。
- **L1174 EN**: Closes the current lexical scope or compound statement.
  **L1174 CN**: 结束当前词法作用域或复合语句块。
- **L1175 EN**: Comment explains nearby logic, intent, or metadata: `The rank is always known even if the extents are not.`.
  **L1175 CN**: 注释说明附近代码的逻辑、意图或元数据：`The rank is always known even if the extents are not.`。
- **L1176 EN**: Returns from the current function with `Shape(static_cast<std::size_t>(call.Rank()), MaybeExtentExpr{})`.
  **L1176 CN**: 以 `Shape(static_cast<std::size_t>(call.Rank()), MaybeExtentExpr{})` 从当前函数返回。

### Lines 1177-1200

````cpp
}

void GetShapeHelper::AccumulateExtent(
    ExtentExpr &result, ExtentExpr &&n) const {
  result = std::move(result) + std::move(n);
  if (context_) {
    // Fold during expression creation to avoid creating an expression so
    // large we can't evaluate it without overflowing the stack.
    result = Fold(*context_, std::move(result));
  }
}

// Check conformance of the passed shapes.
std::optional<bool> CheckConformance(parser::ContextualMessages &messages,
    const Shape &left, const Shape &right, CheckConformanceFlags::Flags flags,
    const char *leftIs, const char *rightIs) {
  int n{GetRank(left)};
  if (n == 0 && (flags & CheckConformanceFlags::LeftScalarExpandable)) {
    return true;
  }
  int rn{GetRank(right)};
  if (rn == 0 && (flags & CheckConformanceFlags::RightScalarExpandable)) {
    return true;
  }
````
- **L1177 EN**: Closes the current lexical scope or compound statement.
  **L1177 CN**: 结束当前词法作用域或复合语句块。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1179 EN**: Continues logic associated with callable symbol `AccumulateExtent`.
  **L1179 CN**: 继续与可调用符号 `AccumulateExtent` 相关的逻辑。
- **L1180 EN**: Continues the surrounding expression or declaration: `ExtentExpr &result, ExtentExpr &&n) const {`.
  **L1180 CN**: 继续构造周围的表达式或声明：`ExtentExpr &result, ExtentExpr &&n) const {`。
- **L1181 EN**: Executes a call or declaration centered on `std::move`.
  **L1181 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L1182 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1182 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1183 EN**: Comment explains nearby logic, intent, or metadata: `Fold during expression creation to avoid creating an expression so`.
  **L1183 CN**: 注释说明附近代码的逻辑、意图或元数据：`Fold during expression creation to avoid creating an expression so`。
- **L1184 EN**: Comment explains nearby logic, intent, or metadata: `large we can't evaluate it without overflowing the stack.`.
  **L1184 CN**: 注释说明附近代码的逻辑、意图或元数据：`large we can't evaluate it without overflowing the stack.`。
- **L1185 EN**: Executes a call or declaration centered on `Fold`.
  **L1185 CN**: 执行以 `Fold` 为核心的调用或声明。
- **L1186 EN**: Closes the current lexical scope or compound statement.
  **L1186 CN**: 结束当前词法作用域或复合语句块。
- **L1187 EN**: Closes the current lexical scope or compound statement.
  **L1187 CN**: 结束当前词法作用域或复合语句块。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1189 EN**: Comment explains nearby logic, intent, or metadata: `Check conformance of the passed shapes.`.
  **L1189 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check conformance of the passed shapes.`。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::optional<bool> CheckConformance(parser::ContextualMessages &messages,`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::optional<bool> CheckConformance(parser::ContextualMessages &messages,`。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Shape &left, const Shape &right, CheckConformanceFlags::Flags flags,`.
  **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Shape &left, const Shape &right, CheckConformanceFlags::Flags flags,`。
- **L1192 EN**: Continues the surrounding expression or declaration: `const char *leftIs, const char *rightIs) {`.
  **L1192 CN**: 继续构造周围的表达式或声明：`const char *leftIs, const char *rightIs) {`。
- **L1193 EN**: Executes a call or declaration centered on `n{GetRank`.
  **L1193 CN**: 执行以 `n{GetRank` 为核心的调用或声明。
- **L1194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1195 EN**: Returns from the current function with `true`.
  **L1195 CN**: 以 `true` 从当前函数返回。
- **L1196 EN**: Closes the current lexical scope or compound statement.
  **L1196 CN**: 结束当前词法作用域或复合语句块。
- **L1197 EN**: Executes a call or declaration centered on `rn{GetRank`.
  **L1197 CN**: 执行以 `rn{GetRank` 为核心的调用或声明。
- **L1198 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1198 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1199 EN**: Returns from the current function with `true`.
  **L1199 CN**: 以 `true` 从当前函数返回。
- **L1200 EN**: Closes the current lexical scope or compound statement.
  **L1200 CN**: 结束当前词法作用域或复合语句块。

### Lines 1201-1224

````cpp
  if (n != rn) {
    messages.Say("Rank of %1$s is %2$d, but %3$s has rank %4$d"_err_en_US,
        leftIs, n, rightIs, rn);
    return false;
  }
  for (int j{0}; j < n; ++j) {
    if (auto leftDim{ToInt64(left[j])}) {
      if (auto rightDim{ToInt64(right[j])}) {
        if (*leftDim != *rightDim) {
          messages.Say("Dimension %1$d of %2$s has extent %3$jd, "
                       "but %4$s has extent %5$jd"_err_en_US,
              j + 1, leftIs, *leftDim, rightIs, *rightDim);
          return false;
        }
      } else if (!(flags & CheckConformanceFlags::RightIsDeferredShape)) {
        return std::nullopt;
      }
    } else if (!(flags & CheckConformanceFlags::LeftIsDeferredShape)) {
      return std::nullopt;
    }
  }
  return true;
}

````
- **L1201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `messages.Say("Rank of %1$s is %2$d, but %3$s has rank %4$d"_err_en_US,`.
  **L1202 CN**: 继续一个多行参数列表、初始化器或聚合项：`messages.Say("Rank of %1$s is %2$d, but %3$s has rank %4$d"_err_en_US,`。
- **L1203 EN**: Executes a standalone statement or declaration: `leftIs, n, rightIs, rn);`.
  **L1203 CN**: 执行一条独立语句或声明：`leftIs, n, rightIs, rn);`。
- **L1204 EN**: Returns from the current function with `false`.
  **L1204 CN**: 以 `false` 从当前函数返回。
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1206 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1207 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1207 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1208 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1209 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1209 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1210 EN**: Continues logic associated with callable symbol `Say`.
  **L1210 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"but %4$s has extent %5$jd"_err_en_US,`.
  **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`"but %4$s has extent %5$jd"_err_en_US,`。
- **L1212 EN**: Executes a standalone statement or declaration: `j + 1, leftIs, *leftDim, rightIs, *rightDim);`.
  **L1212 CN**: 执行一条独立语句或声明：`j + 1, leftIs, *leftDim, rightIs, *rightDim);`。
- **L1213 EN**: Returns from the current function with `false`.
  **L1213 CN**: 以 `false` 从当前函数返回。
- **L1214 EN**: Closes the current lexical scope or compound statement.
  **L1214 CN**: 结束当前词法作用域或复合语句块。
- **L1215 EN**: Transitions from the previous branch into an `else if` condition.
  **L1215 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1216 EN**: Returns from the current function with `std::nullopt`.
  **L1216 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1217 EN**: Closes the current lexical scope or compound statement.
  **L1217 CN**: 结束当前词法作用域或复合语句块。
- **L1218 EN**: Transitions from the previous branch into an `else if` condition.
  **L1218 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1219 EN**: Returns from the current function with `std::nullopt`.
  **L1219 CN**: 以 `std::nullopt` 从当前函数返回。
- **L1220 EN**: Closes the current lexical scope or compound statement.
  **L1220 CN**: 结束当前词法作用域或复合语句块。
- **L1221 EN**: Closes the current lexical scope or compound statement.
  **L1221 CN**: 结束当前词法作用域或复合语句块。
- **L1222 EN**: Returns from the current function with `true`.
  **L1222 CN**: 以 `true` 从当前函数返回。
- **L1223 EN**: Closes the current lexical scope or compound statement.
  **L1223 CN**: 结束当前词法作用域或复合语句块。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1225-1243

````cpp
bool IncrementSubscripts(
    ConstantSubscripts &indices, const ConstantSubscripts &extents) {
  std::size_t rank(indices.size());
  CHECK(rank <= extents.size());
  for (std::size_t j{0}; j < rank; ++j) {
    if (extents[j] < 1) {
      return false;
    }
  }
  for (std::size_t j{0}; j < rank; ++j) {
    if (indices[j]++ < extents[j]) {
      return true;
    }
    indices[j] = 1;
  }
  return false;
}

} // namespace Fortran::evaluate
````
- **L1225 EN**: Continues logic associated with callable symbol `IncrementSubscripts`.
  **L1225 CN**: 继续与可调用符号 `IncrementSubscripts` 相关的逻辑。
- **L1226 EN**: Continues the surrounding expression or declaration: `ConstantSubscripts &indices, const ConstantSubscripts &extents) {`.
  **L1226 CN**: 继续构造周围的表达式或声明：`ConstantSubscripts &indices, const ConstantSubscripts &extents) {`。
- **L1227 EN**: Executes a call or declaration centered on `rank`.
  **L1227 CN**: 执行以 `rank` 为核心的调用或声明。
- **L1228 EN**: Executes a call or declaration centered on `CHECK`.
  **L1228 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L1229 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1229 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1230 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1230 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1231 EN**: Returns from the current function with `false`.
  **L1231 CN**: 以 `false` 从当前函数返回。
- **L1232 EN**: Closes the current lexical scope or compound statement.
  **L1232 CN**: 结束当前词法作用域或复合语句块。
- **L1233 EN**: Closes the current lexical scope or compound statement.
  **L1233 CN**: 结束当前词法作用域或复合语句块。
- **L1234 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1234 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1235 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1235 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1236 EN**: Returns from the current function with `true`.
  **L1236 CN**: 以 `true` 从当前函数返回。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Executes a standalone statement or declaration: `indices[j] = 1;`.
  **L1238 CN**: 执行一条独立语句或声明：`indices[j] = 1;`。
- **L1239 EN**: Closes the current lexical scope or compound statement.
  **L1239 CN**: 结束当前词法作用域或复合语句块。
- **L1240 EN**: Returns from the current function with `false`.
  **L1240 CN**: 以 `false` 从当前函数返回。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Blank line separating nearby declarations or logic blocks.
  **L1242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1243 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::evaluate`.
  **L1243 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::evaluate`。

## Key Concepts / 关键概念

- **Compile-time expression evaluation / 编译期表达式求值**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Constant folding / 常量折叠**
- **Evaluation context management / 求值上下文管理**
- **Fortran descriptor management / Fortran 描述符管理**

## Dependencies / 依赖关系

- `flang/Evaluate/shape.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Common/idioms.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Common/template.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/characteristics.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/check-expression.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/intrinsics.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/type.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/message.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/semantics.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/symbol.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `functional`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
