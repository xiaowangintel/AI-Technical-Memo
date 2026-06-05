# check-cuda.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-cuda.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check cuda.
- **Purpose (CN)**: 实现 check cuda 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-cuda.cpp ----------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "check-cuda.h"
#include "flang/Common/template.h"
#include "flang/Evaluate/fold.h"
#include "flang/Evaluate/tools.h"
#include "flang/Evaluate/traverse.h"
#include "flang/Parser/parse-tree-visitor.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/tools.h"
#include "flang/Semantics/expression.h"
#include "flang/Semantics/symbol.h"
#include "flang/Semantics/tools.h"
#include "llvm/ADT/StringSet.h"

// Once labeled DO constructs have been canonicalized and their parse subtrees
// transformed into parser::DoConstructs, scan the parser::Blocks of the program
// and merge adjacent CUFKernelDoConstructs and DoConstructs whenever the
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
- **L9 EN**: Includes "check-cuda.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "check-cuda.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Common/template.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/template.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Evaluate/fold.h" to access Fortran constant-folding and evaluation facilities.
  **L11 CN**: 引入 "flang/Evaluate/fold.h" 以使用Fortran 常量折叠与求值能力。
- **L12 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L12 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L13 EN**: Includes "flang/Evaluate/traverse.h" to access Fortran constant-folding and evaluation facilities.
  **L13 CN**: 引入 "flang/Evaluate/traverse.h" 以使用Fortran 常量折叠与求值能力。
- **L14 EN**: Includes "flang/Parser/parse-tree-visitor.h" to access parse-tree, token, or source representation support.
  **L14 CN**: 引入 "flang/Parser/parse-tree-visitor.h" 以使用语法树、词法单元或源码表示支持。
- **L15 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L15 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L16 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L16 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。
- **L17 EN**: Includes "flang/Semantics/expression.h" to access Fortran semantic analysis, symbol, and type information.
  **L17 CN**: 引入 "flang/Semantics/expression.h" 以使用Fortran 语义分析、符号与类型信息。
- **L18 EN**: Includes "flang/Semantics/symbol.h" to access Fortran semantic analysis, symbol, and type information.
  **L18 CN**: 引入 "flang/Semantics/symbol.h" 以使用Fortran 语义分析、符号与类型信息。
- **L19 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L19 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L20 EN**: Includes "llvm/ADT/StringSet.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L20 CN**: 引入 "llvm/ADT/StringSet.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Comment explains nearby logic, intent, or metadata: `Once labeled DO constructs have been canonicalized and their parse subtrees`.
  **L22 CN**: 注释说明附近代码的逻辑、意图或元数据：`Once labeled DO constructs have been canonicalized and their parse subtrees`。
- **L23 EN**: Comment explains nearby logic, intent, or metadata: `transformed into parser::DoConstructs, scan the parser::Blocks of the program`.
  **L23 CN**: 注释说明附近代码的逻辑、意图或元数据：`transformed into parser::DoConstructs, scan the parser::Blocks of the program`。
- **L24 EN**: Comment explains nearby logic, intent, or metadata: `and merge adjacent CUFKernelDoConstructs and DoConstructs whenever the`.
  **L24 CN**: 注释说明附近代码的逻辑、意图或元数据：`and merge adjacent CUFKernelDoConstructs and DoConstructs whenever the`。

### Lines 25-48

````cpp
// CUFKernelDoConstruct doesn't already have an embedded DoConstruct.  Also
// emit errors about improper or missing DoConstructs.

namespace Fortran::parser {
struct Mutator {
  template <typename A> bool Pre(A &) { return true; }
  template <typename A> void Post(A &) {}
  bool Pre(Block &);
};

bool Mutator::Pre(Block &block) {
  for (auto iter{block.begin()}; iter != block.end(); ++iter) {
    if (auto *kernel{Unwrap<CUFKernelDoConstruct>(*iter)}) {
      auto &nested{std::get<std::optional<DoConstruct>>(kernel->t)};
      if (!nested) {
        if (auto next{iter}; ++next != block.end()) {
          if (auto *doConstruct{Unwrap<DoConstruct>(*next)}) {
            nested = std::move(*doConstruct);
            block.erase(next);
          }
        }
      }
    } else {
      Walk(*iter, *this);
````
- **L25 EN**: Comment explains nearby logic, intent, or metadata: `CUFKernelDoConstruct doesn't already have an embedded DoConstruct.  Also`.
  **L25 CN**: 注释说明附近代码的逻辑、意图或元数据：`CUFKernelDoConstruct doesn't already have an embedded DoConstruct.  Also`。
- **L26 EN**: Comment explains nearby logic, intent, or metadata: `emit errors about improper or missing DoConstructs.`.
  **L26 CN**: 注释说明附近代码的逻辑、意图或元数据：`emit errors about improper or missing DoConstructs.`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Opens namespace scope `Fortran::parser`.
  **L28 CN**: 打开命名空间作用域 `Fortran::parser`。
- **L29 EN**: Declares struct `Mutator`.
  **L29 CN**: 声明 struct `Mutator`。
- **L30 EN**: Introduces template parameters or specialization context: `template <typename A> bool Pre(A &) { return true; }`.
  **L30 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> bool Pre(A &) { return true; }`。
- **L31 EN**: Introduces template parameters or specialization context: `template <typename A> void Post(A &) {}`.
  **L31 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> void Post(A &) {}`。
- **L32 EN**: Executes a call or declaration centered on `Pre`.
  **L32 CN**: 执行以 `Pre` 为核心的调用或声明。
- **L33 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L33 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Starts a function, method, lambda, or structured scope: `bool Mutator::Pre(Block &block) {`.
  **L35 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool Mutator::Pre(Block &block) {`。
- **L36 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L36 CN**: 开始 `for` 控制流语句并计算其条件。
- **L37 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `if` 控制流语句并计算其条件。
- **L38 EN**: Executes a call or declaration centered on `&nested{std::get<std::optional<DoConstruct>>`.
  **L38 CN**: 执行以 `&nested{std::get<std::optional<DoConstruct>>` 为核心的调用或声明。
- **L39 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L39 CN**: 开始 `if` 控制流语句并计算其条件。
- **L40 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L40 CN**: 开始 `if` 控制流语句并计算其条件。
- **L41 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `if` 控制流语句并计算其条件。
- **L42 EN**: Executes a call or declaration centered on `std::move`.
  **L42 CN**: 执行以 `std::move` 为核心的调用或声明。
- **L43 EN**: Executes a call or declaration centered on `block.erase`.
  **L43 CN**: 执行以 `block.erase` 为核心的调用或声明。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Transitions from the previous branch into the alternative path.
  **L47 CN**: 从前一个分支过渡到备选路径。
- **L48 EN**: Executes a call or declaration centered on `Walk`.
  **L48 CN**: 执行以 `Walk` 为核心的调用或声明。

### Lines 49-72

````cpp
    }
  }
  return false;
}
} // namespace Fortran::parser

namespace Fortran::semantics {

bool CanonicalizeCUDA(parser::Program &program) {
  parser::Mutator mutator;
  parser::Walk(program, mutator);
  return true;
}

using MaybeMsg = std::optional<parser::MessageFormattedText>;

static const llvm::StringSet<> warpFunctions_ = {"match_all_syncjj",
    "match_all_syncjx", "match_all_syncjf", "match_all_syncjd",
    "match_any_syncjj", "match_any_syncjx", "match_any_syncjf",
    "match_any_syncjd"};

// Traverses an evaluate::Expr<> in search of unsupported operations
// on the device.

````
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Returns from the current function with `false`.
  **L51 CN**: 以 `false` 从当前函数返回。
- **L52 EN**: Closes the current lexical scope or compound statement.
  **L52 CN**: 结束当前词法作用域或复合语句块。
- **L53 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::parser`.
  **L53 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::parser`。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L55 EN**: Opens namespace scope `Fortran::semantics`.
  **L55 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Starts a function, method, lambda, or structured scope: `bool CanonicalizeCUDA(parser::Program &program) {`.
  **L57 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool CanonicalizeCUDA(parser::Program &program) {`。
- **L58 EN**: Executes a standalone statement or declaration: `parser::Mutator mutator;`.
  **L58 CN**: 执行一条独立语句或声明：`parser::Mutator mutator;`。
- **L59 EN**: Executes a call or declaration centered on `parser::Walk`.
  **L59 CN**: 执行以 `parser::Walk` 为核心的调用或声明。
- **L60 EN**: Returns from the current function with `true`.
  **L60 CN**: 以 `true` 从当前函数返回。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Defines alias `MaybeMsg` to simplify later code.
  **L63 CN**: 定义别名 `MaybeMsg` 以简化后续代码。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static const llvm::StringSet<> warpFunctions_ = {"match_all_syncjj",`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`static const llvm::StringSet<> warpFunctions_ = {"match_all_syncjj",`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"match_all_syncjx", "match_all_syncjf", "match_all_syncjd",`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`"match_all_syncjx", "match_all_syncjf", "match_all_syncjd",`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"match_any_syncjj", "match_any_syncjx", "match_any_syncjf",`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`"match_any_syncjj", "match_any_syncjx", "match_any_syncjf",`。
- **L68 EN**: Executes a standalone statement or declaration: `"match_any_syncjd"};`.
  **L68 CN**: 执行一条独立语句或声明：`"match_any_syncjd"};`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Comment explains nearby logic, intent, or metadata: `Traverses an evaluate::Expr<> in search of unsupported operations`.
  **L70 CN**: 注释说明附近代码的逻辑、意图或元数据：`Traverses an evaluate::Expr<> in search of unsupported operations`。
- **L71 EN**: Comment explains nearby logic, intent, or metadata: `on the device.`.
  **L71 CN**: 注释说明附近代码的逻辑、意图或元数据：`on the device.`。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 73-96

````cpp
struct DeviceExprChecker
    : public evaluate::AnyTraverse<DeviceExprChecker, MaybeMsg> {
  using Result = MaybeMsg;
  using Base = evaluate::AnyTraverse<DeviceExprChecker, Result>;
  explicit DeviceExprChecker(SemanticsContext &c) : Base(*this), context_{c} {}
  using Base::operator();
  Result operator()(const evaluate::ProcedureDesignator &x) const {
    if (const Symbol * sym{x.GetInterfaceSymbol()}) {
      const auto *subp{
          sym->GetUltimate().detailsIf<semantics::SubprogramDetails>()};
      if (subp) {
        if (auto attrs{subp->cudaSubprogramAttrs()}) {
          if (*attrs == common::CUDASubprogramAttrs::HostDevice ||
              *attrs == common::CUDASubprogramAttrs::Device) {
            if (warpFunctions_.contains(sym->name().ToString()) &&
                !context_.languageFeatures().IsEnabled(
                    Fortran::common::LanguageFeature::CudaWarpMatchFunction)) {
              return parser::MessageFormattedText(
                  "warp match function disabled"_err_en_US);
            }
            return {};
          }
        }
      }
````
- **L73 EN**: Declares struct `DeviceExprChecker`.
  **L73 CN**: 声明 struct `DeviceExprChecker`。
- **L74 EN**: Continues the surrounding expression or declaration: `: public evaluate::AnyTraverse<DeviceExprChecker, MaybeMsg> {`.
  **L74 CN**: 继续构造周围的表达式或声明：`: public evaluate::AnyTraverse<DeviceExprChecker, MaybeMsg> {`。
- **L75 EN**: Defines alias `Result` to simplify later code.
  **L75 CN**: 定义别名 `Result` 以简化后续代码。
- **L76 EN**: Defines alias `Base` to simplify later code.
  **L76 CN**: 定义别名 `Base` 以简化后续代码。
- **L77 EN**: Continues logic associated with callable symbol `DeviceExprChecker`.
  **L77 CN**: 继续与可调用符号 `DeviceExprChecker` 相关的逻辑。
- **L78 EN**: Executes a call or declaration centered on `Base::operator`.
  **L78 CN**: 执行以 `Base::operator` 为核心的调用或声明。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `Result operator()(const evaluate::ProcedureDesignator &x) const {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result operator()(const evaluate::ProcedureDesignator &x) const {`。
- **L80 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L80 CN**: 开始 `if` 控制流语句并计算其条件。
- **L81 EN**: Continues the surrounding expression or declaration: `const auto *subp{`.
  **L81 CN**: 继续构造周围的表达式或声明：`const auto *subp{`。
- **L82 EN**: Executes a call or declaration centered on `sym->GetUltimate`.
  **L82 CN**: 执行以 `sym->GetUltimate` 为核心的调用或声明。
- **L83 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L83 CN**: 开始 `if` 控制流语句并计算其条件。
- **L84 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `if` 控制流语句并计算其条件。
- **L85 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L85 CN**: 开始 `if` 控制流语句并计算其条件。
- **L86 EN**: Comment explains nearby logic, intent, or metadata: `attrs == common::CUDASubprogramAttrs::Device) {`.
  **L86 CN**: 注释说明附近代码的逻辑、意图或元数据：`attrs == common::CUDASubprogramAttrs::Device) {`。
- **L87 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L87 CN**: 开始 `if` 控制流语句并计算其条件。
- **L88 EN**: Continues logic associated with callable symbol `languageFeatures`.
  **L88 CN**: 继续与可调用符号 `languageFeatures` 相关的逻辑。
- **L89 EN**: Continues the surrounding expression or declaration: `Fortran::common::LanguageFeature::CudaWarpMatchFunction)) {`.
  **L89 CN**: 继续构造周围的表达式或声明：`Fortran::common::LanguageFeature::CudaWarpMatchFunction)) {`。
- **L90 EN**: Returns from the current function with `parser::MessageFormattedText(`.
  **L90 CN**: 以 `parser::MessageFormattedText(` 从当前函数返回。
- **L91 EN**: Executes a standalone statement or declaration: `"warp match function disabled"_err_en_US);`.
  **L91 CN**: 执行一条独立语句或声明：`"warp match function disabled"_err_en_US);`。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Returns from the current function with `{}`.
  **L93 CN**: 以 `{}` 从当前函数返回。
- **L94 EN**: Closes the current lexical scope or compound statement.
  **L94 CN**: 结束当前词法作用域或复合语句块。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-120

````cpp

      const Symbol &ultimate{sym->GetUltimate()};
      const Scope &scope{ultimate.owner()};
      const Symbol *mod{scope.IsModule() ? scope.symbol() : nullptr};
      // Allow ieee_arithmetic module functions to be called on the device.
      // TODO: Check for unsupported ieee_arithmetic on the device.
      if (mod && mod->name() == "ieee_arithmetic") {
        return {};
      }
    } else if (x.GetSpecificIntrinsic()) {
      // TODO(CUDA): Check for unsupported intrinsics here
      return {};
    }

    return parser::MessageFormattedText(
        "'%s' may not be called in device code"_err_en_US, x.GetName());
  }

  SemanticsContext &context_;
};

static bool IsHostArray(const Symbol &symbol) {
  const Symbol &resolved{GetAssociationRoot(symbol)};
  if (const auto *details{
````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Executes a call or declaration centered on `&ultimate{sym->GetUltimate`.
  **L98 CN**: 执行以 `&ultimate{sym->GetUltimate` 为核心的调用或声明。
- **L99 EN**: Executes a call or declaration centered on `&scope{ultimate.owner`.
  **L99 CN**: 执行以 `&scope{ultimate.owner` 为核心的调用或声明。
- **L100 EN**: Executes a call or declaration centered on `*mod{scope.IsModule`.
  **L100 CN**: 执行以 `*mod{scope.IsModule` 为核心的调用或声明。
- **L101 EN**: Comment explains nearby logic, intent, or metadata: `Allow ieee_arithmetic module functions to be called on the device.`.
  **L101 CN**: 注释说明附近代码的逻辑、意图或元数据：`Allow ieee_arithmetic module functions to be called on the device.`。
- **L102 EN**: Comment records a pending task or caution: `TODO: Check for unsupported ieee_arithmetic on the device.`.
  **L102 CN**: 注释记录待办事项或注意点：`TODO: Check for unsupported ieee_arithmetic on the device.`。
- **L103 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L103 CN**: 开始 `if` 控制流语句并计算其条件。
- **L104 EN**: Returns from the current function with `{}`.
  **L104 CN**: 以 `{}` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Transitions from the previous branch into an `else if` condition.
  **L106 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L107 EN**: Comment records a pending task or caution: `TODO(CUDA): Check for unsupported intrinsics here`.
  **L107 CN**: 注释记录待办事项或注意点：`TODO(CUDA): Check for unsupported intrinsics here`。
- **L108 EN**: Returns from the current function with `{}`.
  **L108 CN**: 以 `{}` 从当前函数返回。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Returns from the current function with `parser::MessageFormattedText(`.
  **L111 CN**: 以 `parser::MessageFormattedText(` 从当前函数返回。
- **L112 EN**: Executes a call or declaration centered on `x.GetName`.
  **L112 CN**: 执行以 `x.GetName` 为核心的调用或声明。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L115 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L116 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L116 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L118 EN**: Starts a function, method, lambda, or structured scope: `static bool IsHostArray(const Symbol &symbol) {`.
  **L118 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsHostArray(const Symbol &symbol) {`。
- **L119 EN**: Executes a call or declaration centered on `&resolved{GetAssociationRoot`.
  **L119 CN**: 执行以 `&resolved{GetAssociationRoot` 为核心的调用或声明。
- **L120 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L120 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 121-144

````cpp
          resolved.detailsIf<semantics::ObjectEntityDetails>()}) {
    if (details->cudaDataAttr() &&
        (*details->cudaDataAttr() == common::CUDADataAttr::Device ||
            *details->cudaDataAttr() == common::CUDADataAttr::Constant ||
            *details->cudaDataAttr() == common::CUDADataAttr::Managed ||
            *details->cudaDataAttr() == common::CUDADataAttr::Shared ||
            *details->cudaDataAttr() == common::CUDADataAttr::Unified ||
            *details->cudaDataAttr() == common::CUDADataAttr::UseDevice)) {
      return false;
    }
  }
  return true;
}

struct FindHostArray
    : public evaluate::AnyTraverse<FindHostArray, const Symbol *> {
  using Result = const Symbol *;
  using Base = evaluate::AnyTraverse<FindHostArray, Result>;
  FindHostArray() : Base(*this) {}
  using Base::operator();
  Result operator()(const evaluate::Component &x) const {
    const Symbol &symbol{x.GetLastSymbol().GetUltimate()};
    const Symbol &baseSymbol{GetAssociationRoot(x.base().GetFirstSymbol())};
    if (symbol.IsFuncResult() || baseSymbol.IsFuncResult()) {
````
- **L121 EN**: Starts a function, method, lambda, or structured scope: `resolved.detailsIf<semantics::ObjectEntityDetails>()}) {`.
  **L121 CN**: 开始一个函数、方法、lambda 或结构化作用域：`resolved.detailsIf<semantics::ObjectEntityDetails>()}) {`。
- **L122 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `if` 控制流语句并计算其条件。
- **L123 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L123 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L124 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() == common::CUDADataAttr::Constant ||`.
  **L124 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() == common::CUDADataAttr::Constant ||`。
- **L125 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() == common::CUDADataAttr::Managed ||`.
  **L125 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() == common::CUDADataAttr::Managed ||`。
- **L126 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() == common::CUDADataAttr::Shared ||`.
  **L126 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() == common::CUDADataAttr::Shared ||`。
- **L127 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() == common::CUDADataAttr::Unified ||`.
  **L127 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() == common::CUDADataAttr::Unified ||`。
- **L128 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() == common::CUDADataAttr::UseDevice)) {`.
  **L128 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() == common::CUDADataAttr::UseDevice)) {`。
- **L129 EN**: Returns from the current function with `false`.
  **L129 CN**: 以 `false` 从当前函数返回。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Returns from the current function with `true`.
  **L132 CN**: 以 `true` 从当前函数返回。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Blank line separating nearby declarations or logic blocks.
  **L134 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L135 EN**: Declares struct `FindHostArray`.
  **L135 CN**: 声明 struct `FindHostArray`。
- **L136 EN**: Continues the surrounding expression or declaration: `: public evaluate::AnyTraverse<FindHostArray, const Symbol *> {`.
  **L136 CN**: 继续构造周围的表达式或声明：`: public evaluate::AnyTraverse<FindHostArray, const Symbol *> {`。
- **L137 EN**: Defines alias `Result` to simplify later code.
  **L137 CN**: 定义别名 `Result` 以简化后续代码。
- **L138 EN**: Defines alias `Base` to simplify later code.
  **L138 CN**: 定义别名 `Base` 以简化后续代码。
- **L139 EN**: Continues logic associated with callable symbol `FindHostArray`.
  **L139 CN**: 继续与可调用符号 `FindHostArray` 相关的逻辑。
- **L140 EN**: Executes a call or declaration centered on `Base::operator`.
  **L140 CN**: 执行以 `Base::operator` 为核心的调用或声明。
- **L141 EN**: Starts a function, method, lambda, or structured scope: `Result operator()(const evaluate::Component &x) const {`.
  **L141 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result operator()(const evaluate::Component &x) const {`。
- **L142 EN**: Executes a call or declaration centered on `&symbol{x.GetLastSymbol`.
  **L142 CN**: 执行以 `&symbol{x.GetLastSymbol` 为核心的调用或声明。
- **L143 EN**: Executes a call or declaration centered on `&baseSymbol{GetAssociationRoot`.
  **L143 CN**: 执行以 `&baseSymbol{GetAssociationRoot` 为核心的调用或声明。
- **L144 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L144 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 145-168

````cpp
      return nullptr;
    }
    if (!IsHostArray(symbol)) {
      return nullptr;
    }
    if (IsDummy(baseSymbol) && IsCUDADeviceContext(&baseSymbol.owner())) {
      return nullptr;
    }
    if (IsAllocatableOrPointer(symbol)) {
      if (Result hostArray{(*this)(symbol)}) {
        return hostArray;
      }
    } else if (const auto *details{symbol.GetUltimate()
                       .detailsIf<semantics::ObjectEntityDetails>()}) {
      if (details->IsArray()) {
        if (!IsHostArray(baseSymbol)) {
          return nullptr;
        }
        return &symbol;
      }
    }
    return (*this)(x.base());
  }
  Result operator()(const Symbol &symbol) const {
````
- **L145 EN**: Returns from the current function with `nullptr`.
  **L145 CN**: 以 `nullptr` 从当前函数返回。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L147 CN**: 开始 `if` 控制流语句并计算其条件。
- **L148 EN**: Returns from the current function with `nullptr`.
  **L148 CN**: 以 `nullptr` 从当前函数返回。
- **L149 EN**: Closes the current lexical scope or compound statement.
  **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L150 CN**: 开始 `if` 控制流语句并计算其条件。
- **L151 EN**: Returns from the current function with `nullptr`.
  **L151 CN**: 以 `nullptr` 从当前函数返回。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L154 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L154 CN**: 开始 `if` 控制流语句并计算其条件。
- **L155 EN**: Returns from the current function with `hostArray`.
  **L155 CN**: 以 `hostArray` 从当前函数返回。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Transitions from the previous branch into an `else if` condition.
  **L157 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L158 EN**: Starts a function, method, lambda, or structured scope: `.detailsIf<semantics::ObjectEntityDetails>()}) {`.
  **L158 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.detailsIf<semantics::ObjectEntityDetails>()}) {`。
- **L159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L160 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L160 CN**: 开始 `if` 控制流语句并计算其条件。
- **L161 EN**: Returns from the current function with `nullptr`.
  **L161 CN**: 以 `nullptr` 从当前函数返回。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Returns from the current function with `&symbol`.
  **L163 CN**: 以 `&symbol` 从当前函数返回。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Closes the current lexical scope or compound statement.
  **L165 CN**: 结束当前词法作用域或复合语句块。
- **L166 EN**: Returns from the current function with `(*this)(x.base())`.
  **L166 CN**: 以 `(*this)(x.base())` 从当前函数返回。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Starts a function, method, lambda, or structured scope: `Result operator()(const Symbol &symbol) const {`.
  **L168 CN**: 开始一个函数、方法、lambda 或结构化作用域：`Result operator()(const Symbol &symbol) const {`。

### Lines 169-192

````cpp
    if (symbol.IsFuncResult()) {
      return nullptr;
    }
    if (const auto *details{
            symbol.GetUltimate().detailsIf<semantics::ObjectEntityDetails>()}) {
      if (details->IsArray() &&
          !symbol.attrs().test(Fortran::semantics::Attr::PARAMETER) &&
          (!details->cudaDataAttr() ||
              (details->cudaDataAttr() &&
                  *details->cudaDataAttr() != common::CUDADataAttr::Device &&
                  *details->cudaDataAttr() != common::CUDADataAttr::Constant &&
                  *details->cudaDataAttr() != common::CUDADataAttr::Managed &&
                  *details->cudaDataAttr() != common::CUDADataAttr::Shared &&
                  *details->cudaDataAttr() != common::CUDADataAttr::Unified &&
                  *details->cudaDataAttr() !=
                      common::CUDADataAttr::UseDevice))) {
        return &symbol;
      }
    }
    return nullptr;
  }
};

template <typename A>
````
- **L169 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L169 CN**: 开始 `if` 控制流语句并计算其条件。
- **L170 EN**: Returns from the current function with `nullptr`.
  **L170 CN**: 以 `nullptr` 从当前函数返回。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L172 CN**: 开始 `if` 控制流语句并计算其条件。
- **L173 EN**: Starts a function, method, lambda, or structured scope: `symbol.GetUltimate().detailsIf<semantics::ObjectEntityDetails>()}) {`.
  **L173 CN**: 开始一个函数、方法、lambda 或结构化作用域：`symbol.GetUltimate().detailsIf<semantics::ObjectEntityDetails>()}) {`。
- **L174 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L174 CN**: 开始 `if` 控制流语句并计算其条件。
- **L175 EN**: Continues logic associated with callable symbol `attrs`.
  **L175 CN**: 继续与可调用符号 `attrs` 相关的逻辑。
- **L176 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L176 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L177 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L177 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L178 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() != common::CUDADataAttr::Device &&`.
  **L178 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() != common::CUDADataAttr::Device &&`。
- **L179 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() != common::CUDADataAttr::Constant &&`.
  **L179 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() != common::CUDADataAttr::Constant &&`。
- **L180 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() != common::CUDADataAttr::Managed &&`.
  **L180 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() != common::CUDADataAttr::Managed &&`。
- **L181 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() != common::CUDADataAttr::Shared &&`.
  **L181 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() != common::CUDADataAttr::Shared &&`。
- **L182 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() != common::CUDADataAttr::Unified &&`.
  **L182 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() != common::CUDADataAttr::Unified &&`。
- **L183 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() !=`.
  **L183 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() !=`。
- **L184 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::UseDevice))) {`.
  **L184 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::UseDevice))) {`。
- **L185 EN**: Returns from the current function with `&symbol`.
  **L185 CN**: 以 `&symbol` 从当前函数返回。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Closes the current lexical scope or compound statement.
  **L187 CN**: 结束当前词法作用域或复合语句块。
- **L188 EN**: Returns from the current function with `nullptr`.
  **L188 CN**: 以 `nullptr` 从当前函数返回。
- **L189 EN**: Closes the current lexical scope or compound statement.
  **L189 CN**: 结束当前词法作用域或复合语句块。
- **L190 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L190 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L192 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。

### Lines 193-216

````cpp
static MaybeMsg CheckUnwrappedExpr(SemanticsContext &context, const A &x) {
  if (const auto *expr{parser::Unwrap<parser::Expr>(x)}) {
    return DeviceExprChecker{context}(expr->typedExpr);
  }
  return {};
}

template <typename A>
static void CheckUnwrappedExpr(
    SemanticsContext &context, SourceName at, const A &x) {
  if (const auto *expr{parser::Unwrap<parser::Expr>(x)}) {
    if (auto msg{DeviceExprChecker{context}(expr->typedExpr)}) {
      context.Say(at, std::move(*msg));
    }
  }
}

template <bool CUF_KERNEL> struct ActionStmtChecker {
  template <typename A>
  static MaybeMsg WhyNotOk(SemanticsContext &context, const A &x) {
    if constexpr (ConstraintTrait<A>) {
      return WhyNotOk(context, x.thing);
    } else if constexpr (WrapperTrait<A>) {
      return WhyNotOk(context, x.v);
````
- **L193 EN**: Starts a function, method, lambda, or structured scope: `static MaybeMsg CheckUnwrappedExpr(SemanticsContext &context, const A &x) {`.
  **L193 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static MaybeMsg CheckUnwrappedExpr(SemanticsContext &context, const A &x) {`。
- **L194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L195 EN**: Returns from the current function with `DeviceExprChecker{context}(expr->typedExpr)`.
  **L195 CN**: 以 `DeviceExprChecker{context}(expr->typedExpr)` 从当前函数返回。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Returns from the current function with `{}`.
  **L197 CN**: 以 `{}` 从当前函数返回。
- **L198 EN**: Closes the current lexical scope or compound statement.
  **L198 CN**: 结束当前词法作用域或复合语句块。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L200 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L200 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L201 EN**: Continues logic associated with callable symbol `CheckUnwrappedExpr`.
  **L201 CN**: 继续与可调用符号 `CheckUnwrappedExpr` 相关的逻辑。
- **L202 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, SourceName at, const A &x) {`.
  **L202 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, SourceName at, const A &x) {`。
- **L203 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L203 CN**: 开始 `if` 控制流语句并计算其条件。
- **L204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L205 EN**: Executes a call or declaration centered on `context.Say`.
  **L205 CN**: 执行以 `context.Say` 为核心的调用或声明。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Closes the current lexical scope or compound statement.
  **L207 CN**: 结束当前词法作用域或复合语句块。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Introduces template parameters or specialization context: `template <bool CUF_KERNEL> struct ActionStmtChecker {`.
  **L210 CN**: 为后续声明引入模板参数或特化上下文：`template <bool CUF_KERNEL> struct ActionStmtChecker {`。
- **L211 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L211 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L212 EN**: Starts a function, method, lambda, or structured scope: `static MaybeMsg WhyNotOk(SemanticsContext &context, const A &x) {`.
  **L212 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static MaybeMsg WhyNotOk(SemanticsContext &context, const A &x) {`。
- **L213 EN**: Continues logic associated with callable symbol `constexpr`.
  **L213 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。
- **L214 EN**: Returns from the current function with `WhyNotOk(context, x.thing)`.
  **L214 CN**: 以 `WhyNotOk(context, x.thing)` 从当前函数返回。
- **L215 EN**: Transitions from the previous branch into an `else if` condition.
  **L215 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L216 EN**: Returns from the current function with `WhyNotOk(context, x.v)`.
  **L216 CN**: 以 `WhyNotOk(context, x.v)` 从当前函数返回。

### Lines 217-240

````cpp
    } else if constexpr (UnionTrait<A>) {
      return WhyNotOk(context, x.u);
    } else if constexpr (TupleTrait<A>) {
      return WhyNotOk(context, x.t);
    } else {
      return parser::MessageFormattedText{
          "Statement may not appear in device code"_err_en_US};
    }
  }
  template <typename A>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const common::Indirection<A> &x) {
    return WhyNotOk(context, x.value());
  }
  template <typename... As>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const std::variant<As...> &x) {
    return common::visit(
        [&context](const auto &x) { return WhyNotOk(context, x); }, x);
  }
  template <std::size_t J = 0, typename... As>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const std::tuple<As...> &x) {
    if constexpr (J == sizeof...(As)) {
````
- **L217 EN**: Transitions from the previous branch into an `else if` condition.
  **L217 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L218 EN**: Returns from the current function with `WhyNotOk(context, x.u)`.
  **L218 CN**: 以 `WhyNotOk(context, x.u)` 从当前函数返回。
- **L219 EN**: Transitions from the previous branch into an `else if` condition.
  **L219 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L220 EN**: Returns from the current function with `WhyNotOk(context, x.t)`.
  **L220 CN**: 以 `WhyNotOk(context, x.t)` 从当前函数返回。
- **L221 EN**: Transitions from the previous branch into the alternative path.
  **L221 CN**: 从前一个分支过渡到备选路径。
- **L222 EN**: Returns from the current function with `parser::MessageFormattedText{`.
  **L222 CN**: 以 `parser::MessageFormattedText{` 从当前函数返回。
- **L223 EN**: Executes a standalone statement or declaration: `"Statement may not appear in device code"_err_en_US};`.
  **L223 CN**: 执行一条独立语句或声明：`"Statement may not appear in device code"_err_en_US};`。
- **L224 EN**: Closes the current lexical scope or compound statement.
  **L224 CN**: 结束当前词法作用域或复合语句块。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L226 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L227 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L227 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L228 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const common::Indirection<A> &x) {`.
  **L228 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const common::Indirection<A> &x) {`。
- **L229 EN**: Returns from the current function with `WhyNotOk(context, x.value())`.
  **L229 CN**: 以 `WhyNotOk(context, x.value())` 从当前函数返回。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Introduces template parameters or specialization context: `template <typename... As>`.
  **L231 CN**: 为后续声明引入模板参数或特化上下文：`template <typename... As>`。
- **L232 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L232 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L233 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const std::variant<As...> &x) {`.
  **L233 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const std::variant<As...> &x) {`。
- **L234 EN**: Returns from the current function with `common::visit(`.
  **L234 CN**: 以 `common::visit(` 从当前函数返回。
- **L235 EN**: Executes a call or declaration centered on `[&context]`.
  **L235 CN**: 执行以 `[&context]` 为核心的调用或声明。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Introduces template parameters or specialization context: `template <std::size_t J = 0, typename... As>`.
  **L237 CN**: 为后续声明引入模板参数或特化上下文：`template <std::size_t J = 0, typename... As>`。
- **L238 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L238 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L239 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const std::tuple<As...> &x) {`.
  **L239 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const std::tuple<As...> &x) {`。
- **L240 EN**: Continues logic associated with callable symbol `constexpr`.
  **L240 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。

### Lines 241-264

````cpp
      return {};
    } else if (auto msg{WhyNotOk(context, std::get<J>(x))}) {
      return msg;
    } else {
      return WhyNotOk<(J + 1)>(context, x);
    }
  }
  template <typename A>
  static MaybeMsg WhyNotOk(SemanticsContext &context, const std::list<A> &x) {
    for (const auto &y : x) {
      if (MaybeMsg result{WhyNotOk(context, y)}) {
        return result;
      }
    }
    return {};
  }
  template <typename A>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const std::optional<A> &x) {
    if (x) {
      return WhyNotOk(context, *x);
    } else {
      return {};
    }
````
- **L241 EN**: Returns from the current function with `{}`.
  **L241 CN**: 以 `{}` 从当前函数返回。
- **L242 EN**: Transitions from the previous branch into an `else if` condition.
  **L242 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L243 EN**: Returns from the current function with `msg`.
  **L243 CN**: 以 `msg` 从当前函数返回。
- **L244 EN**: Transitions from the previous branch into the alternative path.
  **L244 CN**: 从前一个分支过渡到备选路径。
- **L245 EN**: Returns from the current function with `WhyNotOk<(J + 1)>(context, x)`.
  **L245 CN**: 以 `WhyNotOk<(J + 1)>(context, x)` 从当前函数返回。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L248 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L249 EN**: Starts a function, method, lambda, or structured scope: `static MaybeMsg WhyNotOk(SemanticsContext &context, const std::list<A> &x) {`.
  **L249 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static MaybeMsg WhyNotOk(SemanticsContext &context, const std::list<A> &x) {`。
- **L250 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L250 CN**: 开始 `for` 控制流语句并计算其条件。
- **L251 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L251 CN**: 开始 `if` 控制流语句并计算其条件。
- **L252 EN**: Returns from the current function with `result`.
  **L252 CN**: 以 `result` 从当前函数返回。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Closes the current lexical scope or compound statement.
  **L254 CN**: 结束当前词法作用域或复合语句块。
- **L255 EN**: Returns from the current function with `{}`.
  **L255 CN**: 以 `{}` 从当前函数返回。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L257 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L258 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L258 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L259 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const std::optional<A> &x) {`.
  **L259 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const std::optional<A> &x) {`。
- **L260 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L260 CN**: 开始 `if` 控制流语句并计算其条件。
- **L261 EN**: Returns from the current function with `WhyNotOk(context, *x)`.
  **L261 CN**: 以 `WhyNotOk(context, *x)` 从当前函数返回。
- **L262 EN**: Transitions from the previous branch into the alternative path.
  **L262 CN**: 从前一个分支过渡到备选路径。
- **L263 EN**: Returns from the current function with `{}`.
  **L263 CN**: 以 `{}` 从当前函数返回。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。

### Lines 265-288

````cpp
  }
  template <typename A>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::UnlabeledStatement<A> &x) {
    return WhyNotOk(context, x.statement);
  }
  template <typename A>
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::Statement<A> &x) {
    return WhyNotOk(context, x.statement);
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::AllocateStmt &) {
    return {}; // AllocateObjects are checked elsewhere
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::AllocateCoarraySpec &) {
    return parser::MessageFormattedText(
        "A coarray may not be allocated on the device"_err_en_US);
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::DeallocateStmt &) {
    return {}; // AllocateObjects are checked elsewhere
  }
````
- **L265 EN**: Closes the current lexical scope or compound statement.
  **L265 CN**: 结束当前词法作用域或复合语句块。
- **L266 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L266 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L267 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L267 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L268 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::UnlabeledStatement<A> &x) {`.
  **L268 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::UnlabeledStatement<A> &x) {`。
- **L269 EN**: Returns from the current function with `WhyNotOk(context, x.statement)`.
  **L269 CN**: 以 `WhyNotOk(context, x.statement)` 从当前函数返回。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L271 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L272 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L272 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L273 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::Statement<A> &x) {`.
  **L273 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::Statement<A> &x) {`。
- **L274 EN**: Returns from the current function with `WhyNotOk(context, x.statement)`.
  **L274 CN**: 以 `WhyNotOk(context, x.statement)` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L276 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L277 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::AllocateStmt &) {`.
  **L277 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::AllocateStmt &) {`。
- **L278 EN**: Returns from the current function with `{}; // AllocateObjects are checked elsewhere`.
  **L278 CN**: 以 `{}; // AllocateObjects are checked elsewhere` 从当前函数返回。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L280 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L281 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::AllocateCoarraySpec &) {`.
  **L281 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::AllocateCoarraySpec &) {`。
- **L282 EN**: Returns from the current function with `parser::MessageFormattedText(`.
  **L282 CN**: 以 `parser::MessageFormattedText(` 从当前函数返回。
- **L283 EN**: Executes a standalone statement or declaration: `"A coarray may not be allocated on the device"_err_en_US);`.
  **L283 CN**: 执行一条独立语句或声明：`"A coarray may not be allocated on the device"_err_en_US);`。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L285 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L286 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::DeallocateStmt &) {`.
  **L286 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::DeallocateStmt &) {`。
- **L287 EN**: Returns from the current function with `{}; // AllocateObjects are checked elsewhere`.
  **L287 CN**: 以 `{}; // AllocateObjects are checked elsewhere` 从当前函数返回。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。

### Lines 289-312

````cpp
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::AssignmentStmt &x) {
    return DeviceExprChecker{context}(x.typedAssignment);
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::CallStmt &x) {
    return DeviceExprChecker{context}(x.typedCall);
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::ContinueStmt &) {
    return {};
  }
  static MaybeMsg WhyNotOk(SemanticsContext &context, const parser::IfStmt &x) {
    if (auto result{CheckUnwrappedExpr(
            context, std::get<parser::ScalarLogicalExpr>(x.t))}) {
      return result;
    }
    return WhyNotOk(context,
        std::get<parser::UnlabeledStatement<parser::ActionStmt>>(x.t)
            .statement);
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::NullifyStmt &x) {
    for (const auto &y : x.v) {
````
- **L289 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L289 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L290 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::AssignmentStmt &x) {`.
  **L290 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::AssignmentStmt &x) {`。
- **L291 EN**: Returns from the current function with `DeviceExprChecker{context}(x.typedAssignment)`.
  **L291 CN**: 以 `DeviceExprChecker{context}(x.typedAssignment)` 从当前函数返回。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L293 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L294 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::CallStmt &x) {`.
  **L294 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::CallStmt &x) {`。
- **L295 EN**: Returns from the current function with `DeviceExprChecker{context}(x.typedCall)`.
  **L295 CN**: 以 `DeviceExprChecker{context}(x.typedCall)` 从当前函数返回。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L297 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L298 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::ContinueStmt &) {`.
  **L298 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::ContinueStmt &) {`。
- **L299 EN**: Returns from the current function with `{}`.
  **L299 CN**: 以 `{}` 从当前函数返回。
- **L300 EN**: Closes the current lexical scope or compound statement.
  **L300 CN**: 结束当前词法作用域或复合语句块。
- **L301 EN**: Starts a function, method, lambda, or structured scope: `static MaybeMsg WhyNotOk(SemanticsContext &context, const parser::IfStmt &x) {`.
  **L301 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static MaybeMsg WhyNotOk(SemanticsContext &context, const parser::IfStmt &x) {`。
- **L302 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L302 CN**: 开始 `if` 控制流语句并计算其条件。
- **L303 EN**: Starts a function, method, lambda, or structured scope: `context, std::get<parser::ScalarLogicalExpr>(x.t))}) {`.
  **L303 CN**: 开始一个函数、方法、lambda 或结构化作用域：`context, std::get<parser::ScalarLogicalExpr>(x.t))}) {`。
- **L304 EN**: Returns from the current function with `result`.
  **L304 CN**: 以 `result` 从当前函数返回。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Returns from the current function with `WhyNotOk(context,`.
  **L306 CN**: 以 `WhyNotOk(context,` 从当前函数返回。
- **L307 EN**: Continues logic associated with callable symbol `ActionStmt>>`.
  **L307 CN**: 继续与可调用符号 `ActionStmt>>` 相关的逻辑。
- **L308 EN**: Executes a standalone statement or declaration: `.statement);`.
  **L308 CN**: 执行一条独立语句或声明：`.statement);`。
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L310 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L311 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::NullifyStmt &x) {`.
  **L311 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::NullifyStmt &x) {`。
- **L312 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L312 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 313-336

````cpp
      if (MaybeMsg result{DeviceExprChecker{context}(y.typedExpr)}) {
        return result;
      }
    }
    return {};
  }
  static MaybeMsg WhyNotOk(
      SemanticsContext &context, const parser::PointerAssignmentStmt &x) {
    return DeviceExprChecker{context}(x.typedAssignment);
  }
};

template <bool IsCUFKernelDo> class DeviceContextChecker {
public:
  explicit DeviceContextChecker(SemanticsContext &c) : context_{c} {}
  void CheckSubprogram(const parser::Name &name, const parser::Block &body) {
    if (name.symbol) {
      const auto *subp{
          name.symbol->GetUltimate().detailsIf<SubprogramDetails>()};
      if (subp && subp->moduleInterface()) {
        subp = subp->moduleInterface()
                   ->GetUltimate()
                   .detailsIf<SubprogramDetails>();
      }
````
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Returns from the current function with `result`.
  **L314 CN**: 以 `result` 从当前函数返回。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Returns from the current function with `{}`.
  **L317 CN**: 以 `{}` 从当前函数返回。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Continues logic associated with callable symbol `WhyNotOk`.
  **L319 CN**: 继续与可调用符号 `WhyNotOk` 相关的逻辑。
- **L320 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::PointerAssignmentStmt &x) {`.
  **L320 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::PointerAssignmentStmt &x) {`。
- **L321 EN**: Returns from the current function with `DeviceExprChecker{context}(x.typedAssignment)`.
  **L321 CN**: 以 `DeviceExprChecker{context}(x.typedAssignment)` 从当前函数返回。
- **L322 EN**: Closes the current lexical scope or compound statement.
  **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L323 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L324 EN**: Blank line separating nearby declarations or logic blocks.
  **L324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L325 EN**: Introduces template parameters or specialization context: `template <bool IsCUFKernelDo> class DeviceContextChecker {`.
  **L325 CN**: 为后续声明引入模板参数或特化上下文：`template <bool IsCUFKernelDo> class DeviceContextChecker {`。
- **L326 EN**: Sets the following members to `public` access.
  **L326 CN**: 将后续成员的访问级别设为 `public`。
- **L327 EN**: Continues logic associated with callable symbol `DeviceContextChecker`.
  **L327 CN**: 继续与可调用符号 `DeviceContextChecker` 相关的逻辑。
- **L328 EN**: Starts a function, method, lambda, or structured scope: `void CheckSubprogram(const parser::Name &name, const parser::Block &body) {`.
  **L328 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CheckSubprogram(const parser::Name &name, const parser::Block &body) {`。
- **L329 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L329 CN**: 开始 `if` 控制流语句并计算其条件。
- **L330 EN**: Continues the surrounding expression or declaration: `const auto *subp{`.
  **L330 CN**: 继续构造周围的表达式或声明：`const auto *subp{`。
- **L331 EN**: Executes a call or declaration centered on `name.symbol->GetUltimate`.
  **L331 CN**: 执行以 `name.symbol->GetUltimate` 为核心的调用或声明。
- **L332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L333 EN**: Continues logic associated with callable symbol `moduleInterface`.
  **L333 CN**: 继续与可调用符号 `moduleInterface` 相关的逻辑。
- **L334 EN**: Continues logic associated with callable symbol `GetUltimate`.
  **L334 CN**: 继续与可调用符号 `GetUltimate` 相关的逻辑。
- **L335 EN**: Executes a call or declaration centered on `.detailsIf<SubprogramDetails>`.
  **L335 CN**: 执行以 `.detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-360

````cpp
      if (subp &&
          subp->cudaSubprogramAttrs().value_or(
              common::CUDASubprogramAttrs::Host) !=
              common::CUDASubprogramAttrs::Host) {
        isHostDevice = subp->cudaSubprogramAttrs() &&
            subp->cudaSubprogramAttrs() ==
                common::CUDASubprogramAttrs::HostDevice;
        Check(body);
      }
    }
  }
  void Check(const parser::Block &block) {
    for (const auto &epc : block) {
      Check(epc);
    }
  }

private:
  void Check(const parser::ExecutionPartConstruct &epc) {
    common::visit(
        common::visitors{
            [&](const parser::ExecutableConstruct &x) { Check(x); },
            [&](const parser::Statement<common::Indirection<parser::EntryStmt>>
                    &x) {
````
- **L337 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L337 CN**: 开始 `if` 控制流语句并计算其条件。
- **L338 EN**: Continues logic associated with callable symbol `cudaSubprogramAttrs`.
  **L338 CN**: 继续与可调用符号 `cudaSubprogramAttrs` 相关的逻辑。
- **L339 EN**: Continues the surrounding expression or declaration: `common::CUDASubprogramAttrs::Host) !=`.
  **L339 CN**: 继续构造周围的表达式或声明：`common::CUDASubprogramAttrs::Host) !=`。
- **L340 EN**: Continues the surrounding expression or declaration: `common::CUDASubprogramAttrs::Host) {`.
  **L340 CN**: 继续构造周围的表达式或声明：`common::CUDASubprogramAttrs::Host) {`。
- **L341 EN**: Continues logic associated with callable symbol `cudaSubprogramAttrs`.
  **L341 CN**: 继续与可调用符号 `cudaSubprogramAttrs` 相关的逻辑。
- **L342 EN**: Continues logic associated with callable symbol `cudaSubprogramAttrs`.
  **L342 CN**: 继续与可调用符号 `cudaSubprogramAttrs` 相关的逻辑。
- **L343 EN**: Executes a standalone statement or declaration: `common::CUDASubprogramAttrs::HostDevice;`.
  **L343 CN**: 执行一条独立语句或声明：`common::CUDASubprogramAttrs::HostDevice;`。
- **L344 EN**: Executes a call or declaration centered on `Check`.
  **L344 CN**: 执行以 `Check` 为核心的调用或声明。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Closes the current lexical scope or compound statement.
  **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::Block &block) {`.
  **L348 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::Block &block) {`。
- **L349 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L349 CN**: 开始 `for` 控制流语句并计算其条件。
- **L350 EN**: Executes a call or declaration centered on `Check`.
  **L350 CN**: 执行以 `Check` 为核心的调用或声明。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L354 EN**: Sets the following members to `private` access.
  **L354 CN**: 将后续成员的访问级别设为 `private`。
- **L355 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ExecutionPartConstruct &epc) {`.
  **L355 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ExecutionPartConstruct &epc) {`。
- **L356 EN**: Continues logic associated with callable symbol `visit`.
  **L356 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L357 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L357 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const parser::ExecutableConstruct &x) { Check(x); },`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const parser::ExecutableConstruct &x) { Check(x); },`。
- **L359 EN**: Continues the surrounding expression or declaration: `[&](const parser::Statement<common::Indirection<parser::EntryStmt>>`.
  **L359 CN**: 继续构造周围的表达式或声明：`[&](const parser::Statement<common::Indirection<parser::EntryStmt>>`。
- **L360 EN**: Continues the surrounding expression or declaration: `&x) {`.
  **L360 CN**: 继续构造周围的表达式或声明：`&x) {`。

### Lines 361-384

````cpp
              context_.Say(x.source,
                  "Device code may not contain an ENTRY statement"_err_en_US);
            },
            [](const parser::Statement<common::Indirection<parser::FormatStmt>>
                    &) {},
            [](const parser::Statement<common::Indirection<parser::DataStmt>>
                    &) {},
            [](const parser::Statement<
                common::Indirection<parser::NamelistStmt>> &) {},
            [](const parser::ErrorRecovery &) {},
        },
        epc.u);
  }
  void Check(const parser::ExecutableConstruct &ec) {
    common::visit(
        common::visitors{
            [&](const parser::Statement<parser::ActionStmt> &stmt) {
              Check(stmt.statement, stmt.source);
            },
            [&](const common::Indirection<parser::DoConstruct> &x) {
              if (const std::optional<parser::LoopControl> &control{
                      x.value().GetLoopControl()}) {
                common::visit([&](const auto &y) { Check(y); }, control->u);
              }
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(x.source,`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(x.source,`。
- **L362 EN**: Executes a standalone statement or declaration: `"Device code may not contain an ENTRY statement"_err_en_US);`.
  **L362 CN**: 执行一条独立语句或声明：`"Device code may not contain an ENTRY statement"_err_en_US);`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L364 EN**: Continues the surrounding expression or declaration: `[](const parser::Statement<common::Indirection<parser::FormatStmt>>`.
  **L364 CN**: 继续构造周围的表达式或声明：`[](const parser::Statement<common::Indirection<parser::FormatStmt>>`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&) {},`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`&) {},`。
- **L366 EN**: Continues the surrounding expression or declaration: `[](const parser::Statement<common::Indirection<parser::DataStmt>>`.
  **L366 CN**: 继续构造周围的表达式或声明：`[](const parser::Statement<common::Indirection<parser::DataStmt>>`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `&) {},`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`&) {},`。
- **L368 EN**: Continues the surrounding expression or declaration: `[](const parser::Statement<`.
  **L368 CN**: 继续构造周围的表达式或声明：`[](const parser::Statement<`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `common::Indirection<parser::NamelistStmt>> &) {},`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`common::Indirection<parser::NamelistStmt>> &) {},`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const parser::ErrorRecovery &) {},`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`[](const parser::ErrorRecovery &) {},`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L372 EN**: Executes a standalone statement or declaration: `epc.u);`.
  **L372 CN**: 执行一条独立语句或声明：`epc.u);`。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ExecutableConstruct &ec) {`.
  **L374 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ExecutableConstruct &ec) {`。
- **L375 EN**: Continues logic associated with callable symbol `visit`.
  **L375 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L376 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L376 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L377 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Statement<parser::ActionStmt> &stmt) {`.
  **L377 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Statement<parser::ActionStmt> &stmt) {`。
- **L378 EN**: Executes a call or declaration centered on `Check`.
  **L378 CN**: 执行以 `Check` 为核心的调用或声明。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L380 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::DoConstruct> &x) {`.
  **L380 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::DoConstruct> &x) {`。
- **L381 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L381 CN**: 开始 `if` 控制流语句并计算其条件。
- **L382 EN**: Starts a function, method, lambda, or structured scope: `x.value().GetLoopControl()}) {`.
  **L382 CN**: 开始一个函数、方法、lambda 或结构化作用域：`x.value().GetLoopControl()}) {`。
- **L383 EN**: Executes a call or declaration centered on `common::visit`.
  **L383 CN**: 执行以 `common::visit` 为核心的调用或声明。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-408

````cpp
              Check(std::get<parser::Block>(x.value().t));
            },
            [&](const common::Indirection<parser::BlockConstruct> &x) {
              Check(std::get<parser::Block>(x.value().t));
            },
            [&](const common::Indirection<parser::IfConstruct> &x) {
              Check(x.value());
            },
            [&](const common::Indirection<parser::CaseConstruct> &x) {
              const auto &caseList{
                  std::get<std::list<parser::CaseConstruct::Case>>(
                      x.value().t)};
              for (const parser::CaseConstruct::Case &c : caseList) {
                Check(std::get<parser::Block>(c.t));
              }
            },
            [&](const common::Indirection<parser::CompilerDirective> &x) {
              // TODO(CUDA): Check for unsupported compiler directive here.
            },
            [&](const auto &x) {
              if (auto source{parser::GetSource(x)}) {
                context_.Say(*source,
                    "Statement may not appear in device code"_err_en_US);
              }
````
- **L385 EN**: Executes a call or declaration centered on `Check`.
  **L385 CN**: 执行以 `Check` 为核心的调用或声明。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L387 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::BlockConstruct> &x) {`.
  **L387 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::BlockConstruct> &x) {`。
- **L388 EN**: Executes a call or declaration centered on `Check`.
  **L388 CN**: 执行以 `Check` 为核心的调用或声明。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L390 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::IfConstruct> &x) {`.
  **L390 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::IfConstruct> &x) {`。
- **L391 EN**: Executes a call or declaration centered on `Check`.
  **L391 CN**: 执行以 `Check` 为核心的调用或声明。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L393 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::CaseConstruct> &x) {`.
  **L393 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::CaseConstruct> &x) {`。
- **L394 EN**: Continues the surrounding expression or declaration: `const auto &caseList{`.
  **L394 CN**: 继续构造周围的表达式或声明：`const auto &caseList{`。
- **L395 EN**: Continues logic associated with callable symbol `Case>>`.
  **L395 CN**: 继续与可调用符号 `Case>>` 相关的逻辑。
- **L396 EN**: Executes a call or declaration centered on `x.value`.
  **L396 CN**: 执行以 `x.value` 为核心的调用或声明。
- **L397 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L397 CN**: 开始 `for` 控制流语句并计算其条件。
- **L398 EN**: Executes a call or declaration centered on `Check`.
  **L398 CN**: 执行以 `Check` 为核心的调用或声明。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L401 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::CompilerDirective> &x) {`.
  **L401 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::CompilerDirective> &x) {`。
- **L402 EN**: Comment records a pending task or caution: `TODO(CUDA): Check for unsupported compiler directive here.`.
  **L402 CN**: 注释记录待办事项或注意点：`TODO(CUDA): Check for unsupported compiler directive here.`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L404 EN**: Starts a function, method, lambda, or structured scope: `[&](const auto &x) {`.
  **L404 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const auto &x) {`。
- **L405 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L405 CN**: 开始 `if` 控制流语句并计算其条件。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(*source,`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(*source,`。
- **L407 EN**: Executes a standalone statement or declaration: `"Statement may not appear in device code"_err_en_US);`.
  **L407 CN**: 执行一条独立语句或声明：`"Statement may not appear in device code"_err_en_US);`。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````cpp
            },
        },
        ec.u);
  }
  template <typename SEEK, typename A>
  static const SEEK *GetIOControl(const A &stmt) {
    for (const auto &spec : stmt.controls) {
      if (const auto *result{std::get_if<SEEK>(&spec.u)}) {
        return result;
      }
    }
    return nullptr;
  }
  template <typename A> static bool IsInternalIO(const A &stmt) {
    if (stmt.iounit.has_value()) {
      return std::holds_alternative<Fortran::parser::Variable>(stmt.iounit->u);
    }
    if (auto *unit{GetIOControl<Fortran::parser::IoUnit>(stmt)}) {
      return std::holds_alternative<Fortran::parser::Variable>(unit->u);
    }
    return false;
  }
  void WarnOnIoStmt(const parser::CharBlock &source) {
    context_.Warn(common::UsageWarning::CUDAUsage, source,
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L411 EN**: Executes a standalone statement or declaration: `ec.u);`.
  **L411 CN**: 执行一条独立语句或声明：`ec.u);`。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Introduces template parameters or specialization context: `template <typename SEEK, typename A>`.
  **L413 CN**: 为后续声明引入模板参数或特化上下文：`template <typename SEEK, typename A>`。
- **L414 EN**: Starts a function, method, lambda, or structured scope: `static const SEEK *GetIOControl(const A &stmt) {`.
  **L414 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static const SEEK *GetIOControl(const A &stmt) {`。
- **L415 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L415 CN**: 开始 `for` 控制流语句并计算其条件。
- **L416 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L416 CN**: 开始 `if` 控制流语句并计算其条件。
- **L417 EN**: Returns from the current function with `result`.
  **L417 CN**: 以 `result` 从当前函数返回。
- **L418 EN**: Closes the current lexical scope or compound statement.
  **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Closes the current lexical scope or compound statement.
  **L419 CN**: 结束当前词法作用域或复合语句块。
- **L420 EN**: Returns from the current function with `nullptr`.
  **L420 CN**: 以 `nullptr` 从当前函数返回。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Introduces template parameters or specialization context: `template <typename A> static bool IsInternalIO(const A &stmt) {`.
  **L422 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A> static bool IsInternalIO(const A &stmt) {`。
- **L423 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L423 CN**: 开始 `if` 控制流语句并计算其条件。
- **L424 EN**: Returns from the current function with `std::holds_alternative<Fortran::parser::Variable>(stmt.iounit->u)`.
  **L424 CN**: 以 `std::holds_alternative<Fortran::parser::Variable>(stmt.iounit->u)` 从当前函数返回。
- **L425 EN**: Closes the current lexical scope or compound statement.
  **L425 CN**: 结束当前词法作用域或复合语句块。
- **L426 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L426 CN**: 开始 `if` 控制流语句并计算其条件。
- **L427 EN**: Returns from the current function with `std::holds_alternative<Fortran::parser::Variable>(unit->u)`.
  **L427 CN**: 以 `std::holds_alternative<Fortran::parser::Variable>(unit->u)` 从当前函数返回。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Returns from the current function with `false`.
  **L429 CN**: 以 `false` 从当前函数返回。
- **L430 EN**: Closes the current lexical scope or compound statement.
  **L430 CN**: 结束当前词法作用域或复合语句块。
- **L431 EN**: Starts a function, method, lambda, or structured scope: `void WarnOnIoStmt(const parser::CharBlock &source) {`.
  **L431 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void WarnOnIoStmt(const parser::CharBlock &source) {`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::CUDAUsage, source,`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::CUDAUsage, source,`。

### Lines 433-456

````cpp
        "I/O statement might not be supported on device"_warn_en_US);
  }
  template <typename A>
  void WarnIfNotInternal(const A &stmt, const parser::CharBlock &source) {
    if (!IsInternalIO(stmt)) {
      WarnOnIoStmt(source);
    }
  }
  template <typename A>
  void ErrorIfHostSymbol(const A &expr, parser::CharBlock source) {
    if (isHostDevice)
      return;
    if (context_.languageFeatures().IsEnabled(
            common::LanguageFeature::CudaUnified))
      return;
    if (const Symbol * hostArray{FindHostArray{}(expr)}) {
      context_.Say(source,
          "Host array '%s' cannot be present in device context"_err_en_US,
          hostArray->name());
    }
  }
  void ErrorInCUFKernel(parser::CharBlock source) {
    if (IsCUFKernelDo) {
      context_.Say(
````
- **L433 EN**: Executes a standalone statement or declaration: `"I/O statement might not be supported on device"_warn_en_US);`.
  **L433 CN**: 执行一条独立语句或声明：`"I/O statement might not be supported on device"_warn_en_US);`。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L435 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L436 EN**: Starts a function, method, lambda, or structured scope: `void WarnIfNotInternal(const A &stmt, const parser::CharBlock &source) {`.
  **L436 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void WarnIfNotInternal(const A &stmt, const parser::CharBlock &source) {`。
- **L437 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L437 CN**: 开始 `if` 控制流语句并计算其条件。
- **L438 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L438 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Introduces template parameters or specialization context: `template <typename A>`.
  **L441 CN**: 为后续声明引入模板参数或特化上下文：`template <typename A>`。
- **L442 EN**: Starts a function, method, lambda, or structured scope: `void ErrorIfHostSymbol(const A &expr, parser::CharBlock source) {`.
  **L442 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ErrorIfHostSymbol(const A &expr, parser::CharBlock source) {`。
- **L443 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L443 CN**: 开始 `if` 控制流语句并计算其条件。
- **L444 EN**: Returns from the current function with `void`.
  **L444 CN**: 以 `void` 从当前函数返回。
- **L445 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L445 CN**: 开始 `if` 控制流语句并计算其条件。
- **L446 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::CudaUnified))`.
  **L446 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::CudaUnified))`。
- **L447 EN**: Returns from the current function with `void`.
  **L447 CN**: 以 `void` 从当前函数返回。
- **L448 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L448 CN**: 开始 `if` 控制流语句并计算其条件。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source,`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source,`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Host array '%s' cannot be present in device context"_err_en_US,`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Host array '%s' cannot be present in device context"_err_en_US,`。
- **L451 EN**: Executes a call or declaration centered on `hostArray->name`.
  **L451 CN**: 执行以 `hostArray->name` 为核心的调用或声明。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Starts a function, method, lambda, or structured scope: `void ErrorInCUFKernel(parser::CharBlock source) {`.
  **L454 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ErrorInCUFKernel(parser::CharBlock source) {`。
- **L455 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L455 CN**: 开始 `if` 控制流语句并计算其条件。
- **L456 EN**: Continues logic associated with callable symbol `Say`.
  **L456 CN**: 继续与可调用符号 `Say` 相关的逻辑。

### Lines 457-480

````cpp
          source, "Statement may not appear in cuf kernel code"_err_en_US);
    }
  }
  void Check(const parser::ActionStmt &stmt, const parser::CharBlock &source) {
    common::visit(
        common::visitors{
            [&](const common::Indirection<parser::CycleStmt> &) {
              ErrorInCUFKernel(source);
            },
            [&](const common::Indirection<parser::ExitStmt> &) {
              ErrorInCUFKernel(source);
            },
            [&](const common::Indirection<parser::GotoStmt> &) {
              ErrorInCUFKernel(source);
            },
            [&](const common::Indirection<parser::StopStmt> &) { return; },
            [&](const common::Indirection<parser::PrintStmt> &) {},
            [&](const common::Indirection<parser::WriteStmt> &x) {
              if (x.value().format) { // Formatted write to '*' or '6'
                if (std::holds_alternative<Fortran::parser::Star>(
                        x.value().format->u)) {
                  if (x.value().iounit) {
                    if (std::holds_alternative<Fortran::parser::Star>(
                            x.value().iounit->u)) {
````
- **L457 EN**: Executes a standalone statement or declaration: `source, "Statement may not appear in cuf kernel code"_err_en_US);`.
  **L457 CN**: 执行一条独立语句或声明：`source, "Statement may not appear in cuf kernel code"_err_en_US);`。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ActionStmt &stmt, const parser::CharBlock &source) {`.
  **L460 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ActionStmt &stmt, const parser::CharBlock &source) {`。
- **L461 EN**: Continues logic associated with callable symbol `visit`.
  **L461 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L462 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L462 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L463 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::CycleStmt> &) {`.
  **L463 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::CycleStmt> &) {`。
- **L464 EN**: Executes a call or declaration centered on `ErrorInCUFKernel`.
  **L464 CN**: 执行以 `ErrorInCUFKernel` 为核心的调用或声明。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L466 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::ExitStmt> &) {`.
  **L466 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::ExitStmt> &) {`。
- **L467 EN**: Executes a call or declaration centered on `ErrorInCUFKernel`.
  **L467 CN**: 执行以 `ErrorInCUFKernel` 为核心的调用或声明。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L469 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::GotoStmt> &) {`.
  **L469 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::GotoStmt> &) {`。
- **L470 EN**: Executes a call or declaration centered on `ErrorInCUFKernel`.
  **L470 CN**: 执行以 `ErrorInCUFKernel` 为核心的调用或声明。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const common::Indirection<parser::StopStmt> &) { return; },`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const common::Indirection<parser::StopStmt> &) { return; },`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[&](const common::Indirection<parser::PrintStmt> &) {},`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`[&](const common::Indirection<parser::PrintStmt> &) {},`。
- **L474 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::WriteStmt> &x) {`.
  **L474 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::WriteStmt> &x) {`。
- **L475 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L475 CN**: 开始 `if` 控制流语句并计算其条件。
- **L476 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L476 CN**: 开始 `if` 控制流语句并计算其条件。
- **L477 EN**: Starts a function, method, lambda, or structured scope: `x.value().format->u)) {`.
  **L477 CN**: 开始一个函数、方法、lambda 或结构化作用域：`x.value().format->u)) {`。
- **L478 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L478 CN**: 开始 `if` 控制流语句并计算其条件。
- **L479 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L479 CN**: 开始 `if` 控制流语句并计算其条件。
- **L480 EN**: Starts a function, method, lambda, or structured scope: `x.value().iounit->u)) {`.
  **L480 CN**: 开始一个函数、方法、lambda 或结构化作用域：`x.value().iounit->u)) {`。

### Lines 481-504

````cpp
                      return;
                    }
                  }
                }
              }
              WarnIfNotInternal(x.value(), source);
            },
            [&](const common::Indirection<parser::CloseStmt> &x) {
              WarnOnIoStmt(source);
            },
            [&](const common::Indirection<parser::EndfileStmt> &x) {
              WarnOnIoStmt(source);
            },
            [&](const common::Indirection<parser::OpenStmt> &x) {
              WarnOnIoStmt(source);
            },
            [&](const common::Indirection<parser::ReadStmt> &x) {
              WarnIfNotInternal(x.value(), source);
            },
            [&](const common::Indirection<parser::InquireStmt> &x) {
              WarnOnIoStmt(source);
            },
            [&](const common::Indirection<parser::RewindStmt> &x) {
              WarnOnIoStmt(source);
````
- **L481 EN**: Returns from the current function with `void`.
  **L481 CN**: 以 `void` 从当前函数返回。
- **L482 EN**: Closes the current lexical scope or compound statement.
  **L482 CN**: 结束当前词法作用域或复合语句块。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Executes a call or declaration centered on `WarnIfNotInternal`.
  **L486 CN**: 执行以 `WarnIfNotInternal` 为核心的调用或声明。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L488 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::CloseStmt> &x) {`.
  **L488 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::CloseStmt> &x) {`。
- **L489 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L489 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L491 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::EndfileStmt> &x) {`.
  **L491 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::EndfileStmt> &x) {`。
- **L492 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L492 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L494 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::OpenStmt> &x) {`.
  **L494 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::OpenStmt> &x) {`。
- **L495 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L495 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L497 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::ReadStmt> &x) {`.
  **L497 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::ReadStmt> &x) {`。
- **L498 EN**: Executes a call or declaration centered on `WarnIfNotInternal`.
  **L498 CN**: 执行以 `WarnIfNotInternal` 为核心的调用或声明。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L500 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::InquireStmt> &x) {`.
  **L500 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::InquireStmt> &x) {`。
- **L501 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L501 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L503 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::RewindStmt> &x) {`.
  **L503 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::RewindStmt> &x) {`。
- **L504 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L504 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。

### Lines 505-528

````cpp
            },
            [&](const common::Indirection<parser::BackspaceStmt> &x) {
              WarnOnIoStmt(source);
            },
            [&](const common::Indirection<parser::IfStmt> &x) {
              Check(x.value());
            },
            [&](const common::Indirection<parser::AssignmentStmt> &x) {
              if (const evaluate::Assignment *
                  assign{semantics::GetAssignment(x.value())}) {
                ErrorIfHostSymbol(assign->lhs, source);
                ErrorIfHostSymbol(assign->rhs, source);
              }
              if (auto msg{ActionStmtChecker<IsCUFKernelDo>::WhyNotOk(
                      context_, x)}) {
                context_.Say(source, std::move(*msg));
              }
            },
            [&](const auto &x) {
              if (auto msg{ActionStmtChecker<IsCUFKernelDo>::WhyNotOk(
                      context_, x)}) {
                context_.Say(source, std::move(*msg));
              }
            },
````
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L506 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::BackspaceStmt> &x) {`.
  **L506 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::BackspaceStmt> &x) {`。
- **L507 EN**: Executes a call or declaration centered on `WarnOnIoStmt`.
  **L507 CN**: 执行以 `WarnOnIoStmt` 为核心的调用或声明。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L509 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::IfStmt> &x) {`.
  **L509 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::IfStmt> &x) {`。
- **L510 EN**: Executes a call or declaration centered on `Check`.
  **L510 CN**: 执行以 `Check` 为核心的调用或声明。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L512 EN**: Starts a function, method, lambda, or structured scope: `[&](const common::Indirection<parser::AssignmentStmt> &x) {`.
  **L512 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const common::Indirection<parser::AssignmentStmt> &x) {`。
- **L513 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L513 CN**: 开始 `if` 控制流语句并计算其条件。
- **L514 EN**: Starts a function, method, lambda, or structured scope: `assign{semantics::GetAssignment(x.value())}) {`.
  **L514 CN**: 开始一个函数、方法、lambda 或结构化作用域：`assign{semantics::GetAssignment(x.value())}) {`。
- **L515 EN**: Executes a call or declaration centered on `ErrorIfHostSymbol`.
  **L515 CN**: 执行以 `ErrorIfHostSymbol` 为核心的调用或声明。
- **L516 EN**: Executes a call or declaration centered on `ErrorIfHostSymbol`.
  **L516 CN**: 执行以 `ErrorIfHostSymbol` 为核心的调用或声明。
- **L517 EN**: Closes the current lexical scope or compound statement.
  **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L518 CN**: 开始 `if` 控制流语句并计算其条件。
- **L519 EN**: Continues the surrounding expression or declaration: `context_, x)}) {`.
  **L519 CN**: 继续构造周围的表达式或声明：`context_, x)}) {`。
- **L520 EN**: Executes a call or declaration centered on `context_.Say`.
  **L520 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L521 EN**: Closes the current lexical scope or compound statement.
  **L521 CN**: 结束当前词法作用域或复合语句块。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L523 EN**: Starts a function, method, lambda, or structured scope: `[&](const auto &x) {`.
  **L523 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const auto &x) {`。
- **L524 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L524 CN**: 开始 `if` 控制流语句并计算其条件。
- **L525 EN**: Continues the surrounding expression or declaration: `context_, x)}) {`.
  **L525 CN**: 继续构造周围的表达式或声明：`context_, x)}) {`。
- **L526 EN**: Executes a call or declaration centered on `context_.Say`.
  **L526 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L527 EN**: Closes the current lexical scope or compound statement.
  **L527 CN**: 结束当前词法作用域或复合语句块。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 529-552

````cpp
        },
        stmt.u);
  }
  void Check(const parser::IfConstruct &ic) {
    const auto &ifS{std::get<parser::Statement<parser::IfThenStmt>>(ic.t)};
    CheckUnwrappedExpr(context_, ifS.source,
        std::get<parser::ScalarLogicalExpr>(ifS.statement.t));
    Check(std::get<parser::Block>(ic.t));
    for (const auto &eib :
        std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ic.t)) {
      const auto &eIfS{std::get<parser::Statement<parser::ElseIfStmt>>(eib.t)};
      CheckUnwrappedExpr(context_, eIfS.source,
          std::get<parser::ScalarLogicalExpr>(eIfS.statement.t));
      Check(std::get<parser::Block>(eib.t));
    }
    if (const auto &eb{
            std::get<std::optional<parser::IfConstruct::ElseBlock>>(ic.t)}) {
      Check(std::get<parser::Block>(eb->t));
    }
  }
  void Check(const parser::IfStmt &is) {
    const auto &uS{
        std::get<parser::UnlabeledStatement<parser::ActionStmt>>(is.t)};
    CheckUnwrappedExpr(
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L530 EN**: Executes a standalone statement or declaration: `stmt.u);`.
  **L530 CN**: 执行一条独立语句或声明：`stmt.u);`。
- **L531 EN**: Closes the current lexical scope or compound statement.
  **L531 CN**: 结束当前词法作用域或复合语句块。
- **L532 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::IfConstruct &ic) {`.
  **L532 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::IfConstruct &ic) {`。
- **L533 EN**: Executes a call or declaration centered on `&ifS{std::get<parser::Statement<parser::IfThenStmt>>`.
  **L533 CN**: 执行以 `&ifS{std::get<parser::Statement<parser::IfThenStmt>>` 为核心的调用或声明。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckUnwrappedExpr(context_, ifS.source,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckUnwrappedExpr(context_, ifS.source,`。
- **L535 EN**: Executes a call or declaration centered on `std::get<parser::ScalarLogicalExpr>`.
  **L535 CN**: 执行以 `std::get<parser::ScalarLogicalExpr>` 为核心的调用或声明。
- **L536 EN**: Executes a call or declaration centered on `Check`.
  **L536 CN**: 执行以 `Check` 为核心的调用或声明。
- **L537 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L537 CN**: 开始 `for` 控制流语句并计算其条件。
- **L538 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ic.t)) {`.
  **L538 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::IfConstruct::ElseIfBlock>>(ic.t)) {`。
- **L539 EN**: Executes a call or declaration centered on `&eIfS{std::get<parser::Statement<parser::ElseIfStmt>>`.
  **L539 CN**: 执行以 `&eIfS{std::get<parser::Statement<parser::ElseIfStmt>>` 为核心的调用或声明。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckUnwrappedExpr(context_, eIfS.source,`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckUnwrappedExpr(context_, eIfS.source,`。
- **L541 EN**: Executes a call or declaration centered on `std::get<parser::ScalarLogicalExpr>`.
  **L541 CN**: 执行以 `std::get<parser::ScalarLogicalExpr>` 为核心的调用或声明。
- **L542 EN**: Executes a call or declaration centered on `Check`.
  **L542 CN**: 执行以 `Check` 为核心的调用或声明。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L544 CN**: 开始 `if` 控制流语句并计算其条件。
- **L545 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::IfConstruct::ElseBlock>>(ic.t)}) {`.
  **L545 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::IfConstruct::ElseBlock>>(ic.t)}) {`。
- **L546 EN**: Executes a call or declaration centered on `Check`.
  **L546 CN**: 执行以 `Check` 为核心的调用或声明。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::IfStmt &is) {`.
  **L549 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::IfStmt &is) {`。
- **L550 EN**: Continues the surrounding expression or declaration: `const auto &uS{`.
  **L550 CN**: 继续构造周围的表达式或声明：`const auto &uS{`。
- **L551 EN**: Executes a call or declaration centered on `std::get<parser::UnlabeledStatement<parser::ActionStmt>>`.
  **L551 CN**: 执行以 `std::get<parser::UnlabeledStatement<parser::ActionStmt>>` 为核心的调用或声明。
- **L552 EN**: Continues logic associated with callable symbol `CheckUnwrappedExpr`.
  **L552 CN**: 继续与可调用符号 `CheckUnwrappedExpr` 相关的逻辑。

### Lines 553-576

````cpp
        context_, uS.source, std::get<parser::ScalarLogicalExpr>(is.t));
    Check(uS.statement, uS.source);
  }
  void Check(const parser::LoopControl::Bounds &bounds) {
    Check(bounds.Lower());
    Check(bounds.Upper());
    if (auto &step{bounds.Step()}) {
      Check(*step);
    }
  }
  void Check(const parser::LoopControl::Concurrent &x) {
    const auto &header{std::get<parser::ConcurrentHeader>(x.t)};
    for (const auto &cc :
        std::get<std::list<parser::ConcurrentControl>>(header.t)) {
      Check(std::get<1>(cc.t));
      Check(std::get<2>(cc.t));
      if (const auto &step{
              std::get<std::optional<parser::ScalarIntExpr>>(cc.t)}) {
        Check(*step);
      }
    }
    if (const auto &mask{
            std::get<std::optional<parser::ScalarLogicalExpr>>(header.t)}) {
      Check(*mask);
````
- **L553 EN**: Executes a call or declaration centered on `std::get<parser::ScalarLogicalExpr>`.
  **L553 CN**: 执行以 `std::get<parser::ScalarLogicalExpr>` 为核心的调用或声明。
- **L554 EN**: Executes a call or declaration centered on `Check`.
  **L554 CN**: 执行以 `Check` 为核心的调用或声明。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::LoopControl::Bounds &bounds) {`.
  **L556 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::LoopControl::Bounds &bounds) {`。
- **L557 EN**: Executes a call or declaration centered on `Check`.
  **L557 CN**: 执行以 `Check` 为核心的调用或声明。
- **L558 EN**: Executes a call or declaration centered on `Check`.
  **L558 CN**: 执行以 `Check` 为核心的调用或声明。
- **L559 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L559 CN**: 开始 `if` 控制流语句并计算其条件。
- **L560 EN**: Executes a call or declaration centered on `Check`.
  **L560 CN**: 执行以 `Check` 为核心的调用或声明。
- **L561 EN**: Closes the current lexical scope or compound statement.
  **L561 CN**: 结束当前词法作用域或复合语句块。
- **L562 EN**: Closes the current lexical scope or compound statement.
  **L562 CN**: 结束当前词法作用域或复合语句块。
- **L563 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::LoopControl::Concurrent &x) {`.
  **L563 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::LoopControl::Concurrent &x) {`。
- **L564 EN**: Executes a call or declaration centered on `&header{std::get<parser::ConcurrentHeader>`.
  **L564 CN**: 执行以 `&header{std::get<parser::ConcurrentHeader>` 为核心的调用或声明。
- **L565 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L565 CN**: 开始 `for` 控制流语句并计算其条件。
- **L566 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::ConcurrentControl>>(header.t)) {`.
  **L566 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::ConcurrentControl>>(header.t)) {`。
- **L567 EN**: Executes a call or declaration centered on `Check`.
  **L567 CN**: 执行以 `Check` 为核心的调用或声明。
- **L568 EN**: Executes a call or declaration centered on `Check`.
  **L568 CN**: 执行以 `Check` 为核心的调用或声明。
- **L569 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L569 CN**: 开始 `if` 控制流语句并计算其条件。
- **L570 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::ScalarIntExpr>>(cc.t)}) {`.
  **L570 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::ScalarIntExpr>>(cc.t)}) {`。
- **L571 EN**: Executes a call or declaration centered on `Check`.
  **L571 CN**: 执行以 `Check` 为核心的调用或声明。
- **L572 EN**: Closes the current lexical scope or compound statement.
  **L572 CN**: 结束当前词法作用域或复合语句块。
- **L573 EN**: Closes the current lexical scope or compound statement.
  **L573 CN**: 结束当前词法作用域或复合语句块。
- **L574 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L574 CN**: 开始 `if` 控制流语句并计算其条件。
- **L575 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::ScalarLogicalExpr>>(header.t)}) {`.
  **L575 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::ScalarLogicalExpr>>(header.t)}) {`。
- **L576 EN**: Executes a call or declaration centered on `Check`.
  **L576 CN**: 执行以 `Check` 为核心的调用或声明。

### Lines 577-600

````cpp
    }
  }
  void Check(const parser::ScalarLogicalExpr &x) {
    Check(DEREF(parser::Unwrap<parser::Expr>(x)));
  }
  void Check(const parser::ScalarIntExpr &x) {
    Check(DEREF(parser::Unwrap<parser::Expr>(x)));
  }
  void Check(const parser::ScalarExpr &x) {
    Check(DEREF(parser::Unwrap<parser::Expr>(x)));
  }
  void Check(const parser::Expr &expr) {
    if (MaybeMsg msg{DeviceExprChecker{context_}(expr.typedExpr)}) {
      context_.Say(expr.source, std::move(*msg));
    }
  }

  SemanticsContext &context_;
  bool isHostDevice{false};
};

void CUDAChecker::Enter(const parser::SubroutineSubprogram &x) {
  DeviceContextChecker<false>{context_}.CheckSubprogram(
      std::get<parser::Name>(
````
- **L577 EN**: Closes the current lexical scope or compound statement.
  **L577 CN**: 结束当前词法作用域或复合语句块。
- **L578 EN**: Closes the current lexical scope or compound statement.
  **L578 CN**: 结束当前词法作用域或复合语句块。
- **L579 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ScalarLogicalExpr &x) {`.
  **L579 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ScalarLogicalExpr &x) {`。
- **L580 EN**: Executes a call or declaration centered on `Check`.
  **L580 CN**: 执行以 `Check` 为核心的调用或声明。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ScalarIntExpr &x) {`.
  **L582 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ScalarIntExpr &x) {`。
- **L583 EN**: Executes a call or declaration centered on `Check`.
  **L583 CN**: 执行以 `Check` 为核心的调用或声明。
- **L584 EN**: Closes the current lexical scope or compound statement.
  **L584 CN**: 结束当前词法作用域或复合语句块。
- **L585 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::ScalarExpr &x) {`.
  **L585 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::ScalarExpr &x) {`。
- **L586 EN**: Executes a call or declaration centered on `Check`.
  **L586 CN**: 执行以 `Check` 为核心的调用或声明。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Starts a function, method, lambda, or structured scope: `void Check(const parser::Expr &expr) {`.
  **L588 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void Check(const parser::Expr &expr) {`。
- **L589 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L589 CN**: 开始 `if` 控制流语句并计算其条件。
- **L590 EN**: Executes a call or declaration centered on `context_.Say`.
  **L590 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L591 EN**: Closes the current lexical scope or compound statement.
  **L591 CN**: 结束当前词法作用域或复合语句块。
- **L592 EN**: Closes the current lexical scope or compound statement.
  **L592 CN**: 结束当前词法作用域或复合语句块。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L594 EN**: Executes a standalone statement or declaration: `SemanticsContext &context_;`.
  **L594 CN**: 执行一条独立语句或声明：`SemanticsContext &context_;`。
- **L595 EN**: Executes a standalone statement or declaration: `bool isHostDevice{false};`.
  **L595 CN**: 执行一条独立语句或声明：`bool isHostDevice{false};`。
- **L596 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L596 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L597 EN**: Blank line separating nearby declarations or logic blocks.
  **L597 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L598 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::SubroutineSubprogram &x) {`.
  **L598 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::SubroutineSubprogram &x) {`。
- **L599 EN**: Continues logic associated with callable symbol `CheckSubprogram`.
  **L599 CN**: 继续与可调用符号 `CheckSubprogram` 相关的逻辑。
- **L600 EN**: Continues logic associated with callable symbol `Name>`.
  **L600 CN**: 继续与可调用符号 `Name>` 相关的逻辑。

### Lines 601-624

````cpp
          std::get<parser::Statement<parser::SubroutineStmt>>(x.t).statement.t),
      std::get<parser::ExecutionPart>(x.t).v);
}

void CUDAChecker::Enter(const parser::FunctionSubprogram &x) {
  DeviceContextChecker<false>{context_}.CheckSubprogram(
      std::get<parser::Name>(
          std::get<parser::Statement<parser::FunctionStmt>>(x.t).statement.t),
      std::get<parser::ExecutionPart>(x.t).v);
}

void CUDAChecker::Enter(const parser::SeparateModuleSubprogram &x) {
  DeviceContextChecker<false>{context_}.CheckSubprogram(
      std::get<parser::Statement<parser::MpSubprogramStmt>>(x.t).statement.v,
      std::get<parser::ExecutionPart>(x.t).v);
}

// !$CUF KERNEL DO semantic checks

static int DoConstructTightNesting(
    const parser::DoConstruct *doConstruct, const parser::Block *&innerBlock) {
  if (!doConstruct ||
      (!doConstruct->IsDoNormal() && !doConstruct->IsDoConcurrent())) {
    return 0;
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::get<parser::Statement<parser::SubroutineStmt>>(x.t).statement.t),`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::get<parser::Statement<parser::SubroutineStmt>>(x.t).statement.t),`。
- **L602 EN**: Executes a call or declaration centered on `std::get<parser::ExecutionPart>`.
  **L602 CN**: 执行以 `std::get<parser::ExecutionPart>` 为核心的调用或声明。
- **L603 EN**: Closes the current lexical scope or compound statement.
  **L603 CN**: 结束当前词法作用域或复合语句块。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L605 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::FunctionSubprogram &x) {`.
  **L605 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::FunctionSubprogram &x) {`。
- **L606 EN**: Continues logic associated with callable symbol `CheckSubprogram`.
  **L606 CN**: 继续与可调用符号 `CheckSubprogram` 相关的逻辑。
- **L607 EN**: Continues logic associated with callable symbol `Name>`.
  **L607 CN**: 继续与可调用符号 `Name>` 相关的逻辑。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::get<parser::Statement<parser::FunctionStmt>>(x.t).statement.t),`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::get<parser::Statement<parser::FunctionStmt>>(x.t).statement.t),`。
- **L609 EN**: Executes a call or declaration centered on `std::get<parser::ExecutionPart>`.
  **L609 CN**: 执行以 `std::get<parser::ExecutionPart>` 为核心的调用或声明。
- **L610 EN**: Closes the current lexical scope or compound statement.
  **L610 CN**: 结束当前词法作用域或复合语句块。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L612 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::SeparateModuleSubprogram &x) {`.
  **L612 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::SeparateModuleSubprogram &x) {`。
- **L613 EN**: Continues logic associated with callable symbol `CheckSubprogram`.
  **L613 CN**: 继续与可调用符号 `CheckSubprogram` 相关的逻辑。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::get<parser::Statement<parser::MpSubprogramStmt>>(x.t).statement.v,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::get<parser::Statement<parser::MpSubprogramStmt>>(x.t).statement.v,`。
- **L615 EN**: Executes a call or declaration centered on `std::get<parser::ExecutionPart>`.
  **L615 CN**: 执行以 `std::get<parser::ExecutionPart>` 为核心的调用或声明。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L618 EN**: Comment explains nearby logic, intent, or metadata: `$CUF KERNEL DO semantic checks`.
  **L618 CN**: 注释说明附近代码的逻辑、意图或元数据：`$CUF KERNEL DO semantic checks`。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L620 EN**: Continues logic associated with callable symbol `DoConstructTightNesting`.
  **L620 CN**: 继续与可调用符号 `DoConstructTightNesting` 相关的逻辑。
- **L621 EN**: Continues the surrounding expression or declaration: `const parser::DoConstruct *doConstruct, const parser::Block *&innerBlock) {`.
  **L621 CN**: 继续构造周围的表达式或声明：`const parser::DoConstruct *doConstruct, const parser::Block *&innerBlock) {`。
- **L622 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L622 CN**: 开始 `if` 控制流语句并计算其条件。
- **L623 EN**: Starts a function, method, lambda, or structured scope: `(!doConstruct->IsDoNormal() && !doConstruct->IsDoConcurrent())) {`.
  **L623 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(!doConstruct->IsDoNormal() && !doConstruct->IsDoConcurrent())) {`。
- **L624 EN**: Returns from the current function with `0`.
  **L624 CN**: 以 `0` 从当前函数返回。

### Lines 625-648

````cpp
  }
  innerBlock = &std::get<parser::Block>(doConstruct->t);
  if (doConstruct->IsDoConcurrent()) {
    const auto &loopControl = doConstruct->GetLoopControl();
    if (loopControl) {
      if (const auto *concurrentControl{
              std::get_if<parser::LoopControl::Concurrent>(&loopControl->u)}) {
        const auto &concurrentHeader =
            std::get<Fortran::parser::ConcurrentHeader>(concurrentControl->t);
        const auto &controls =
            std::get<std::list<Fortran::parser::ConcurrentControl>>(
                concurrentHeader.t);
        return controls.size();
      }
    }
    return 0;
  }
  if (innerBlock->size() == 1) {
    if (const auto *execConstruct{
            std::get_if<parser::ExecutableConstruct>(&innerBlock->front().u)}) {
      if (const auto *next{
              std::get_if<common::Indirection<parser::DoConstruct>>(
                  &execConstruct->u)}) {
        return 1 + DoConstructTightNesting(&next->value(), innerBlock);
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Executes a call or declaration centered on `&std::get<parser::Block>`.
  **L626 CN**: 执行以 `&std::get<parser::Block>` 为核心的调用或声明。
- **L627 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L627 CN**: 开始 `if` 控制流语句并计算其条件。
- **L628 EN**: Executes a call or declaration centered on `doConstruct->GetLoopControl`.
  **L628 CN**: 执行以 `doConstruct->GetLoopControl` 为核心的调用或声明。
- **L629 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L629 CN**: 开始 `if` 控制流语句并计算其条件。
- **L630 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L630 CN**: 开始 `if` 控制流语句并计算其条件。
- **L631 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<parser::LoopControl::Concurrent>(&loopControl->u)}) {`.
  **L631 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<parser::LoopControl::Concurrent>(&loopControl->u)}) {`。
- **L632 EN**: Continues the surrounding expression or declaration: `const auto &concurrentHeader =`.
  **L632 CN**: 继续构造周围的表达式或声明：`const auto &concurrentHeader =`。
- **L633 EN**: Executes a call or declaration centered on `std::get<Fortran::parser::ConcurrentHeader>`.
  **L633 CN**: 执行以 `std::get<Fortran::parser::ConcurrentHeader>` 为核心的调用或声明。
- **L634 EN**: Continues the surrounding expression or declaration: `const auto &controls =`.
  **L634 CN**: 继续构造周围的表达式或声明：`const auto &controls =`。
- **L635 EN**: Continues logic associated with callable symbol `ConcurrentControl>>`.
  **L635 CN**: 继续与可调用符号 `ConcurrentControl>>` 相关的逻辑。
- **L636 EN**: Executes a standalone statement or declaration: `concurrentHeader.t);`.
  **L636 CN**: 执行一条独立语句或声明：`concurrentHeader.t);`。
- **L637 EN**: Returns from the current function with `controls.size()`.
  **L637 CN**: 以 `controls.size()` 从当前函数返回。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Returns from the current function with `0`.
  **L640 CN**: 以 `0` 从当前函数返回。
- **L641 EN**: Closes the current lexical scope or compound statement.
  **L641 CN**: 结束当前词法作用域或复合语句块。
- **L642 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L642 CN**: 开始 `if` 控制流语句并计算其条件。
- **L643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L644 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<parser::ExecutableConstruct>(&innerBlock->front().u)}) {`.
  **L644 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<parser::ExecutableConstruct>(&innerBlock->front().u)}) {`。
- **L645 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L645 CN**: 开始 `if` 控制流语句并计算其条件。
- **L646 EN**: Continues logic associated with callable symbol `DoConstruct>>`.
  **L646 CN**: 继续与可调用符号 `DoConstruct>>` 相关的逻辑。
- **L647 EN**: Continues the surrounding expression or declaration: `&execConstruct->u)}) {`.
  **L647 CN**: 继续构造周围的表达式或声明：`&execConstruct->u)}) {`。
- **L648 EN**: Returns from the current function with `1 + DoConstructTightNesting(&next->value(), innerBlock)`.
  **L648 CN**: 以 `1 + DoConstructTightNesting(&next->value(), innerBlock)` 从当前函数返回。

### Lines 649-672

````cpp
      }
    }
  }
  return 1;
}

static void CheckReduce(
    SemanticsContext &context, const parser::CUFReduction &reduce) {
  auto op{std::get<parser::CUFReduction::Operator>(reduce.t).v};
  for (const auto &var :
      std::get<std::list<parser::Scalar<parser::Variable>>>(reduce.t)) {
    if (const auto &typedExprPtr{var.thing.typedExpr};
        typedExprPtr && typedExprPtr->v) {
      const auto &expr{*typedExprPtr->v};
      if (auto type{expr.GetType()}) {
        auto cat{type->category()};
        bool isOk{false};
        switch (op) {
        case parser::ReductionOperator::Operator::Plus:
        case parser::ReductionOperator::Operator::Multiply:
        case parser::ReductionOperator::Operator::Max:
        case parser::ReductionOperator::Operator::Min:
          isOk = cat == TypeCategory::Integer || cat == TypeCategory::Real ||
              cat == TypeCategory::Complex;
````
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Closes the current lexical scope or compound statement.
  **L650 CN**: 结束当前词法作用域或复合语句块。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Returns from the current function with `1`.
  **L652 CN**: 以 `1` 从当前函数返回。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L655 EN**: Continues logic associated with callable symbol `CheckReduce`.
  **L655 CN**: 继续与可调用符号 `CheckReduce` 相关的逻辑。
- **L656 EN**: Continues the surrounding expression or declaration: `SemanticsContext &context, const parser::CUFReduction &reduce) {`.
  **L656 CN**: 继续构造周围的表达式或声明：`SemanticsContext &context, const parser::CUFReduction &reduce) {`。
- **L657 EN**: Executes a call or declaration centered on `op{std::get<parser::CUFReduction::Operator>`.
  **L657 CN**: 执行以 `op{std::get<parser::CUFReduction::Operator>` 为核心的调用或声明。
- **L658 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L658 CN**: 开始 `for` 控制流语句并计算其条件。
- **L659 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::Scalar<parser::Variable>>>(reduce.t)) {`.
  **L659 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::Scalar<parser::Variable>>>(reduce.t)) {`。
- **L660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L660 CN**: 开始 `if` 控制流语句并计算其条件。
- **L661 EN**: Continues the surrounding expression or declaration: `typedExprPtr && typedExprPtr->v) {`.
  **L661 CN**: 继续构造周围的表达式或声明：`typedExprPtr && typedExprPtr->v) {`。
- **L662 EN**: Executes a standalone statement or declaration: `const auto &expr{*typedExprPtr->v};`.
  **L662 CN**: 执行一条独立语句或声明：`const auto &expr{*typedExprPtr->v};`。
- **L663 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L663 CN**: 开始 `if` 控制流语句并计算其条件。
- **L664 EN**: Executes a call or declaration centered on `cat{type->category`.
  **L664 CN**: 执行以 `cat{type->category` 为核心的调用或声明。
- **L665 EN**: Executes a standalone statement or declaration: `bool isOk{false};`.
  **L665 CN**: 执行一条独立语句或声明：`bool isOk{false};`。
- **L666 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L666 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L667 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Plus:`.
  **L667 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Plus:`。
- **L668 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Multiply:`.
  **L668 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Multiply:`。
- **L669 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Max:`.
  **L669 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Max:`。
- **L670 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Min:`.
  **L670 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Min:`。
- **L671 EN**: Continues the surrounding expression or declaration: `isOk = cat == TypeCategory::Integer || cat == TypeCategory::Real ||`.
  **L671 CN**: 继续构造周围的表达式或声明：`isOk = cat == TypeCategory::Integer || cat == TypeCategory::Real ||`。
- **L672 EN**: Executes a standalone statement or declaration: `cat == TypeCategory::Complex;`.
  **L672 CN**: 执行一条独立语句或声明：`cat == TypeCategory::Complex;`。

### Lines 673-696

````cpp
          break;
        case parser::ReductionOperator::Operator::Iand:
        case parser::ReductionOperator::Operator::Ior:
        case parser::ReductionOperator::Operator::Ieor:
          isOk = cat == TypeCategory::Integer;
          break;
        case parser::ReductionOperator::Operator::And:
        case parser::ReductionOperator::Operator::Or:
        case parser::ReductionOperator::Operator::Eqv:
        case parser::ReductionOperator::Operator::Neqv:
          isOk = cat == TypeCategory::Logical;
          break;
        }
        if (!isOk) {
          context.Say(var.thing.GetSource(),
              "!$CUF KERNEL DO REDUCE operation is not acceptable for a variable with type %s"_err_en_US,
              type->AsFortran());
        }
      }
    }
  }
}

void CUDAChecker::Enter(const parser::CUFKernelDoConstruct &x) {
````
- **L673 EN**: Exits the nearest loop or switch statement.
  **L673 CN**: 退出最近的循环或 switch 语句。
- **L674 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Iand:`.
  **L674 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Iand:`。
- **L675 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Ior:`.
  **L675 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Ior:`。
- **L676 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Ieor:`.
  **L676 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Ieor:`。
- **L677 EN**: Executes a standalone statement or declaration: `isOk = cat == TypeCategory::Integer;`.
  **L677 CN**: 执行一条独立语句或声明：`isOk = cat == TypeCategory::Integer;`。
- **L678 EN**: Exits the nearest loop or switch statement.
  **L678 CN**: 退出最近的循环或 switch 语句。
- **L679 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::And:`.
  **L679 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::And:`。
- **L680 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Or:`.
  **L680 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Or:`。
- **L681 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Eqv:`.
  **L681 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Eqv:`。
- **L682 EN**: Introduces a switch dispatch label: `case parser::ReductionOperator::Operator::Neqv:`.
  **L682 CN**: 引入一个 switch 分发标签：`case parser::ReductionOperator::Operator::Neqv:`。
- **L683 EN**: Executes a standalone statement or declaration: `isOk = cat == TypeCategory::Logical;`.
  **L683 CN**: 执行一条独立语句或声明：`isOk = cat == TypeCategory::Logical;`。
- **L684 EN**: Exits the nearest loop or switch statement.
  **L684 CN**: 退出最近的循环或 switch 语句。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L686 CN**: 开始 `if` 控制流语句并计算其条件。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context.Say(var.thing.GetSource(),`.
  **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`context.Say(var.thing.GetSource(),`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"!$CUF KERNEL DO REDUCE operation is not acceptable for a variable with type %s"_err_en_US,`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`"!$CUF KERNEL DO REDUCE operation is not acceptable for a variable with type %s"_err_en_US,`。
- **L689 EN**: Executes a call or declaration centered on `type->AsFortran`.
  **L689 CN**: 执行以 `type->AsFortran` 为核心的调用或声明。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Closes the current lexical scope or compound statement.
  **L691 CN**: 结束当前词法作用域或复合语句块。
- **L692 EN**: Closes the current lexical scope or compound statement.
  **L692 CN**: 结束当前词法作用域或复合语句块。
- **L693 EN**: Closes the current lexical scope or compound statement.
  **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L696 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::CUFKernelDoConstruct &x) {`.
  **L696 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::CUFKernelDoConstruct &x) {`。

### Lines 697-720

````cpp
  auto source{std::get<parser::CUFKernelDoConstruct::Directive>(x.t).source};
  const auto &directive{std::get<parser::CUFKernelDoConstruct::Directive>(x.t)};
  std::int64_t depth{1};
  if (auto expr{AnalyzeExpr(context_,
          std::get<std::optional<parser::ScalarIntConstantExpr>>(
              directive.t))}) {
    depth = evaluate::ToInt64(expr).value_or(0);
    if (depth <= 0) {
      context_.Say(source,
          "!$CUF KERNEL DO (%jd): loop nesting depth must be positive"_err_en_US,
          std::intmax_t{depth});
      depth = 1;
    }
  }
  const parser::DoConstruct *doConstruct{common::GetPtrFromOptional(
      std::get<std::optional<parser::DoConstruct>>(x.t))};
  const parser::Block *innerBlock{nullptr};
  if (DoConstructTightNesting(doConstruct, innerBlock) < depth) {
    if (doConstruct && doConstruct->IsDoConcurrent())
      context_.Say(source,
          "!$CUF KERNEL DO (%jd) must be followed by a DO CONCURRENT construct with at least %jd indices"_err_en_US,
          std::intmax_t{depth}, std::intmax_t{depth});
    else
      context_.Say(source,
````
- **L697 EN**: Executes a call or declaration centered on `source{std::get<parser::CUFKernelDoConstruct::Directive>`.
  **L697 CN**: 执行以 `source{std::get<parser::CUFKernelDoConstruct::Directive>` 为核心的调用或声明。
- **L698 EN**: Executes a call or declaration centered on `&directive{std::get<parser::CUFKernelDoConstruct::Directive>`.
  **L698 CN**: 执行以 `&directive{std::get<parser::CUFKernelDoConstruct::Directive>` 为核心的调用或声明。
- **L699 EN**: Executes a standalone statement or declaration: `std::int64_t depth{1};`.
  **L699 CN**: 执行一条独立语句或声明：`std::int64_t depth{1};`。
- **L700 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L700 CN**: 开始 `if` 控制流语句并计算其条件。
- **L701 EN**: Continues logic associated with callable symbol `ScalarIntConstantExpr>>`.
  **L701 CN**: 继续与可调用符号 `ScalarIntConstantExpr>>` 相关的逻辑。
- **L702 EN**: Continues the surrounding expression or declaration: `directive.t))}) {`.
  **L702 CN**: 继续构造周围的表达式或声明：`directive.t))}) {`。
- **L703 EN**: Executes a call or declaration centered on `evaluate::ToInt64`.
  **L703 CN**: 执行以 `evaluate::ToInt64` 为核心的调用或声明。
- **L704 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L704 CN**: 开始 `if` 控制流语句并计算其条件。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source,`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source,`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"!$CUF KERNEL DO (%jd): loop nesting depth must be positive"_err_en_US,`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`"!$CUF KERNEL DO (%jd): loop nesting depth must be positive"_err_en_US,`。
- **L707 EN**: Executes a standalone statement or declaration: `std::intmax_t{depth});`.
  **L707 CN**: 执行一条独立语句或声明：`std::intmax_t{depth});`。
- **L708 EN**: Executes a standalone statement or declaration: `depth = 1;`.
  **L708 CN**: 执行一条独立语句或声明：`depth = 1;`。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Closes the current lexical scope or compound statement.
  **L710 CN**: 结束当前词法作用域或复合语句块。
- **L711 EN**: Continues logic associated with callable symbol `GetPtrFromOptional`.
  **L711 CN**: 继续与可调用符号 `GetPtrFromOptional` 相关的逻辑。
- **L712 EN**: Executes a call or declaration centered on `std::get<std::optional<parser::DoConstruct>>`.
  **L712 CN**: 执行以 `std::get<std::optional<parser::DoConstruct>>` 为核心的调用或声明。
- **L713 EN**: Executes a standalone statement or declaration: `const parser::Block *innerBlock{nullptr};`.
  **L713 CN**: 执行一条独立语句或声明：`const parser::Block *innerBlock{nullptr};`。
- **L714 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L714 CN**: 开始 `if` 控制流语句并计算其条件。
- **L715 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L715 CN**: 开始 `if` 控制流语句并计算其条件。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"!$CUF KERNEL DO (%jd) must be followed by a DO CONCURRENT construct with at least %jd indices"_err_en_US,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`"!$CUF KERNEL DO (%jd) must be followed by a DO CONCURRENT construct with at least %jd indices"_err_en_US,`。
- **L718 EN**: Executes a standalone statement or declaration: `std::intmax_t{depth}, std::intmax_t{depth});`.
  **L718 CN**: 执行一条独立语句或声明：`std::intmax_t{depth}, std::intmax_t{depth});`。
- **L719 EN**: Transitions from the previous branch into the alternative path.
  **L719 CN**: 从前一个分支过渡到备选路径。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(source,`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(source,`。

### Lines 721-744

````cpp
          "!$CUF KERNEL DO (%jd) must be followed by a DO construct with tightly nested outer levels of counted DO loops"_err_en_US,
          std::intmax_t{depth});
  }
  if (innerBlock) {
    DeviceContextChecker<true>{context_}.Check(*innerBlock);
  }
  for (const auto &reduce :
      std::get<std::list<parser::CUFReduction>>(directive.t)) {
    CheckReduce(context_, reduce);
  }
  ++deviceConstructDepth_;
}

static bool IsOpenACCComputeConstruct(const parser::OpenACCBlockConstruct &x) {
  const auto &beginBlockDirective =
      std::get<Fortran::parser::AccBeginBlockDirective>(x.t);
  const auto &blockDirective =
      std::get<Fortran::parser::AccBlockDirective>(beginBlockDirective.t);
  if (blockDirective.v == llvm::acc::ACCD_parallel ||
      blockDirective.v == llvm::acc::ACCD_serial ||
      blockDirective.v == llvm::acc::ACCD_kernels) {
    return true;
  }
  return false;
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"!$CUF KERNEL DO (%jd) must be followed by a DO construct with tightly nested outer levels of counted DO loops"_err_en_US,`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`"!$CUF KERNEL DO (%jd) must be followed by a DO construct with tightly nested outer levels of counted DO loops"_err_en_US,`。
- **L722 EN**: Executes a standalone statement or declaration: `std::intmax_t{depth});`.
  **L722 CN**: 执行一条独立语句或声明：`std::intmax_t{depth});`。
- **L723 EN**: Closes the current lexical scope or compound statement.
  **L723 CN**: 结束当前词法作用域或复合语句块。
- **L724 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L724 CN**: 开始 `if` 控制流语句并计算其条件。
- **L725 EN**: Executes a call or declaration centered on `DeviceContextChecker<true>{context_}.Check`.
  **L725 CN**: 执行以 `DeviceContextChecker<true>{context_}.Check` 为核心的调用或声明。
- **L726 EN**: Closes the current lexical scope or compound statement.
  **L726 CN**: 结束当前词法作用域或复合语句块。
- **L727 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L727 CN**: 开始 `for` 控制流语句并计算其条件。
- **L728 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::list<parser::CUFReduction>>(directive.t)) {`.
  **L728 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::list<parser::CUFReduction>>(directive.t)) {`。
- **L729 EN**: Executes a call or declaration centered on `CheckReduce`.
  **L729 CN**: 执行以 `CheckReduce` 为核心的调用或声明。
- **L730 EN**: Closes the current lexical scope or compound statement.
  **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Executes a standalone statement or declaration: `++deviceConstructDepth_;`.
  **L731 CN**: 执行一条独立语句或声明：`++deviceConstructDepth_;`。
- **L732 EN**: Closes the current lexical scope or compound statement.
  **L732 CN**: 结束当前词法作用域或复合语句块。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L734 EN**: Starts a function, method, lambda, or structured scope: `static bool IsOpenACCComputeConstruct(const parser::OpenACCBlockConstruct &x) {`.
  **L734 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool IsOpenACCComputeConstruct(const parser::OpenACCBlockConstruct &x) {`。
- **L735 EN**: Continues the surrounding expression or declaration: `const auto &beginBlockDirective =`.
  **L735 CN**: 继续构造周围的表达式或声明：`const auto &beginBlockDirective =`。
- **L736 EN**: Executes a call or declaration centered on `std::get<Fortran::parser::AccBeginBlockDirective>`.
  **L736 CN**: 执行以 `std::get<Fortran::parser::AccBeginBlockDirective>` 为核心的调用或声明。
- **L737 EN**: Continues the surrounding expression or declaration: `const auto &blockDirective =`.
  **L737 CN**: 继续构造周围的表达式或声明：`const auto &blockDirective =`。
- **L738 EN**: Executes a call or declaration centered on `std::get<Fortran::parser::AccBlockDirective>`.
  **L738 CN**: 执行以 `std::get<Fortran::parser::AccBlockDirective>` 为核心的调用或声明。
- **L739 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L739 CN**: 开始 `if` 控制流语句并计算其条件。
- **L740 EN**: Continues the surrounding expression or declaration: `blockDirective.v == llvm::acc::ACCD_serial ||`.
  **L740 CN**: 继续构造周围的表达式或声明：`blockDirective.v == llvm::acc::ACCD_serial ||`。
- **L741 EN**: Continues the surrounding expression or declaration: `blockDirective.v == llvm::acc::ACCD_kernels) {`.
  **L741 CN**: 继续构造周围的表达式或声明：`blockDirective.v == llvm::acc::ACCD_kernels) {`。
- **L742 EN**: Returns from the current function with `true`.
  **L742 CN**: 以 `true` 从当前函数返回。
- **L743 EN**: Closes the current lexical scope or compound statement.
  **L743 CN**: 结束当前词法作用域或复合语句块。
- **L744 EN**: Returns from the current function with `false`.
  **L744 CN**: 以 `false` 从当前函数返回。

### Lines 745-768

````cpp
}

void CUDAChecker::Leave(const parser::CUFKernelDoConstruct &) {
  --deviceConstructDepth_;
}
void CUDAChecker::Enter(const parser::OpenACCBlockConstruct &x) {
  if (IsOpenACCComputeConstruct(x)) {
    ++deviceConstructDepth_;
  }
}
void CUDAChecker::Leave(const parser::OpenACCBlockConstruct &x) {
  if (IsOpenACCComputeConstruct(x)) {
    --deviceConstructDepth_;
  }
}
void CUDAChecker::Enter(const parser::OpenACCCombinedConstruct &) {
  ++deviceConstructDepth_;
}
void CUDAChecker::Leave(const parser::OpenACCCombinedConstruct &) {
  --deviceConstructDepth_;
}
void CUDAChecker::Enter(const parser::OpenACCLoopConstruct &) {
  ++deviceConstructDepth_;
}
````
- **L745 EN**: Closes the current lexical scope or compound statement.
  **L745 CN**: 结束当前词法作用域或复合语句块。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L747 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Leave(const parser::CUFKernelDoConstruct &) {`.
  **L747 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Leave(const parser::CUFKernelDoConstruct &) {`。
- **L748 EN**: Executes a standalone statement or declaration: `--deviceConstructDepth_;`.
  **L748 CN**: 执行一条独立语句或声明：`--deviceConstructDepth_;`。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::OpenACCBlockConstruct &x) {`.
  **L750 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::OpenACCBlockConstruct &x) {`。
- **L751 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L751 CN**: 开始 `if` 控制流语句并计算其条件。
- **L752 EN**: Executes a standalone statement or declaration: `++deviceConstructDepth_;`.
  **L752 CN**: 执行一条独立语句或声明：`++deviceConstructDepth_;`。
- **L753 EN**: Closes the current lexical scope or compound statement.
  **L753 CN**: 结束当前词法作用域或复合语句块。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Leave(const parser::OpenACCBlockConstruct &x) {`.
  **L755 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Leave(const parser::OpenACCBlockConstruct &x) {`。
- **L756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L757 EN**: Executes a standalone statement or declaration: `--deviceConstructDepth_;`.
  **L757 CN**: 执行一条独立语句或声明：`--deviceConstructDepth_;`。
- **L758 EN**: Closes the current lexical scope or compound statement.
  **L758 CN**: 结束当前词法作用域或复合语句块。
- **L759 EN**: Closes the current lexical scope or compound statement.
  **L759 CN**: 结束当前词法作用域或复合语句块。
- **L760 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::OpenACCCombinedConstruct &) {`.
  **L760 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::OpenACCCombinedConstruct &) {`。
- **L761 EN**: Executes a standalone statement or declaration: `++deviceConstructDepth_;`.
  **L761 CN**: 执行一条独立语句或声明：`++deviceConstructDepth_;`。
- **L762 EN**: Closes the current lexical scope or compound statement.
  **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Leave(const parser::OpenACCCombinedConstruct &) {`.
  **L763 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Leave(const parser::OpenACCCombinedConstruct &) {`。
- **L764 EN**: Executes a standalone statement or declaration: `--deviceConstructDepth_;`.
  **L764 CN**: 执行一条独立语句或声明：`--deviceConstructDepth_;`。
- **L765 EN**: Closes the current lexical scope or compound statement.
  **L765 CN**: 结束当前词法作用域或复合语句块。
- **L766 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::OpenACCLoopConstruct &) {`.
  **L766 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::OpenACCLoopConstruct &) {`。
- **L767 EN**: Executes a standalone statement or declaration: `++deviceConstructDepth_;`.
  **L767 CN**: 执行一条独立语句或声明：`++deviceConstructDepth_;`。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````cpp
void CUDAChecker::Leave(const parser::OpenACCLoopConstruct &) {
  --deviceConstructDepth_;
}
void CUDAChecker::Enter(const parser::DoConstruct &x) {
  if (x.IsDoConcurrent() &&
      context_.foldingContext().languageFeatures().IsEnabled(
          common::LanguageFeature::DoConcurrentOffload)) {
    ++deviceConstructDepth_;
  }
}
void CUDAChecker::Leave(const parser::DoConstruct &x) {
  if (x.IsDoConcurrent() &&
      context_.foldingContext().languageFeatures().IsEnabled(
          common::LanguageFeature::DoConcurrentOffload)) {
    --deviceConstructDepth_;
  }
}

void CUDAChecker::Enter(const parser::AssignmentStmt &x) {
  auto lhsLoc{std::get<parser::Variable>(x.t).GetSource()};
  const auto &scope{context_.FindScope(lhsLoc)};
  const Scope &progUnit{GetProgramUnitContaining(scope)};
  if (IsCUDADeviceContext(&progUnit) || deviceConstructDepth_ > 0) {
    return; // Data transfer with assignment is only perform on host.
````
- **L769 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Leave(const parser::OpenACCLoopConstruct &) {`.
  **L769 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Leave(const parser::OpenACCLoopConstruct &) {`。
- **L770 EN**: Executes a standalone statement or declaration: `--deviceConstructDepth_;`.
  **L770 CN**: 执行一条独立语句或声明：`--deviceConstructDepth_;`。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::DoConstruct &x) {`.
  **L772 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::DoConstruct &x) {`。
- **L773 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L773 CN**: 开始 `if` 控制流语句并计算其条件。
- **L774 EN**: Continues logic associated with callable symbol `foldingContext`.
  **L774 CN**: 继续与可调用符号 `foldingContext` 相关的逻辑。
- **L775 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::DoConcurrentOffload)) {`.
  **L775 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::DoConcurrentOffload)) {`。
- **L776 EN**: Executes a standalone statement or declaration: `++deviceConstructDepth_;`.
  **L776 CN**: 执行一条独立语句或声明：`++deviceConstructDepth_;`。
- **L777 EN**: Closes the current lexical scope or compound statement.
  **L777 CN**: 结束当前词法作用域或复合语句块。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Leave(const parser::DoConstruct &x) {`.
  **L779 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Leave(const parser::DoConstruct &x) {`。
- **L780 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L780 CN**: 开始 `if` 控制流语句并计算其条件。
- **L781 EN**: Continues logic associated with callable symbol `foldingContext`.
  **L781 CN**: 继续与可调用符号 `foldingContext` 相关的逻辑。
- **L782 EN**: Continues the surrounding expression or declaration: `common::LanguageFeature::DoConcurrentOffload)) {`.
  **L782 CN**: 继续构造周围的表达式或声明：`common::LanguageFeature::DoConcurrentOffload)) {`。
- **L783 EN**: Executes a standalone statement or declaration: `--deviceConstructDepth_;`.
  **L783 CN**: 执行一条独立语句或声明：`--deviceConstructDepth_;`。
- **L784 EN**: Closes the current lexical scope or compound statement.
  **L784 CN**: 结束当前词法作用域或复合语句块。
- **L785 EN**: Closes the current lexical scope or compound statement.
  **L785 CN**: 结束当前词法作用域或复合语句块。
- **L786 EN**: Blank line separating nearby declarations or logic blocks.
  **L786 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L787 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::AssignmentStmt &x) {`.
  **L787 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::AssignmentStmt &x) {`。
- **L788 EN**: Executes a call or declaration centered on `lhsLoc{std::get<parser::Variable>`.
  **L788 CN**: 执行以 `lhsLoc{std::get<parser::Variable>` 为核心的调用或声明。
- **L789 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L789 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L790 EN**: Executes a call or declaration centered on `&progUnit{GetProgramUnitContaining`.
  **L790 CN**: 执行以 `&progUnit{GetProgramUnitContaining` 为核心的调用或声明。
- **L791 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L791 CN**: 开始 `if` 控制流语句并计算其条件。
- **L792 EN**: Returns from the current function with `; // Data transfer with assignment is only perform on host.`.
  **L792 CN**: 以 `; // Data transfer with assignment is only perform on host.` 从当前函数返回。

### Lines 793-816

````cpp
  }

  const evaluate::Assignment *assign{semantics::GetAssignment(x)};
  if (!assign) {
    return;
  }

  int nbLhs{evaluate::GetNbOfCUDADeviceSymbols(assign->lhs)};
  int nbRhs{evaluate::GetNbOfUniqueCUDADeviceSymbols(assign->rhs)};
  int nbRhsManaged{evaluate::GetNbOfCUDAManagedOrUnifiedSymbols(assign->rhs)};

  // device to host transfer with more than one device object on the rhs is not
  // legal.
  if (nbLhs == 0 && nbRhs > 1 && nbRhsManaged != nbRhs) {
    context_.Say(lhsLoc,
        "More than one reference to a CUDA object on the right hand side of the assignment"_err_en_US);
  }

  if (evaluate::HasCUDADeviceAttrs(assign->lhs) &&
      evaluate::HasCUDAImplicitTransfer(assign->rhs)) {
    if (GetNbOfCUDAManagedOrUnifiedSymbols(assign->lhs) == 1 &&
        GetNbOfCUDAManagedOrUnifiedSymbols(assign->rhs) == 1 && nbRhs == 1) {
      return; // This is a special case handled on the host.
    }
````
- **L793 EN**: Closes the current lexical scope or compound statement.
  **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L795 EN**: Executes a call or declaration centered on `*assign{semantics::GetAssignment`.
  **L795 CN**: 执行以 `*assign{semantics::GetAssignment` 为核心的调用或声明。
- **L796 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L796 CN**: 开始 `if` 控制流语句并计算其条件。
- **L797 EN**: Returns from the current function with `void`.
  **L797 CN**: 以 `void` 从当前函数返回。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L800 EN**: Executes a call or declaration centered on `nbLhs{evaluate::GetNbOfCUDADeviceSymbols`.
  **L800 CN**: 执行以 `nbLhs{evaluate::GetNbOfCUDADeviceSymbols` 为核心的调用或声明。
- **L801 EN**: Executes a call or declaration centered on `nbRhs{evaluate::GetNbOfUniqueCUDADeviceSymbols`.
  **L801 CN**: 执行以 `nbRhs{evaluate::GetNbOfUniqueCUDADeviceSymbols` 为核心的调用或声明。
- **L802 EN**: Executes a call or declaration centered on `nbRhsManaged{evaluate::GetNbOfCUDAManagedOrUnifiedSymbols`.
  **L802 CN**: 执行以 `nbRhsManaged{evaluate::GetNbOfCUDAManagedOrUnifiedSymbols` 为核心的调用或声明。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L804 EN**: Comment explains nearby logic, intent, or metadata: `device to host transfer with more than one device object on the rhs is not`.
  **L804 CN**: 注释说明附近代码的逻辑、意图或元数据：`device to host transfer with more than one device object on the rhs is not`。
- **L805 EN**: Comment explains nearby logic, intent, or metadata: `legal.`.
  **L805 CN**: 注释说明附近代码的逻辑、意图或元数据：`legal.`。
- **L806 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L806 CN**: 开始 `if` 控制流语句并计算其条件。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(lhsLoc,`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(lhsLoc,`。
- **L808 EN**: Executes a standalone statement or declaration: `"More than one reference to a CUDA object on the right hand side of the assignment"_err_en_US);`.
  **L808 CN**: 执行一条独立语句或声明：`"More than one reference to a CUDA object on the right hand side of the assignment"_err_en_US);`。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L811 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L811 CN**: 开始 `if` 控制流语句并计算其条件。
- **L812 EN**: Starts a function, method, lambda, or structured scope: `evaluate::HasCUDAImplicitTransfer(assign->rhs)) {`.
  **L812 CN**: 开始一个函数、方法、lambda 或结构化作用域：`evaluate::HasCUDAImplicitTransfer(assign->rhs)) {`。
- **L813 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L813 CN**: 开始 `if` 控制流语句并计算其条件。
- **L814 EN**: Starts a function, method, lambda, or structured scope: `GetNbOfCUDAManagedOrUnifiedSymbols(assign->rhs) == 1 && nbRhs == 1) {`.
  **L814 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetNbOfCUDAManagedOrUnifiedSymbols(assign->rhs) == 1 && nbRhs == 1) {`。
- **L815 EN**: Returns from the current function with `; // This is a special case handled on the host.`.
  **L815 CN**: 以 `; // This is a special case handled on the host.` 从当前函数返回。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

### Lines 817-840

````cpp
    context_.Say(lhsLoc, "Unsupported CUDA data transfer"_err_en_US);
  }
}

void CUDAChecker::Enter(const parser::PrintStmt &x) {
  CHECK(context_.location());
  const Scope &scope{context_.FindScope(*context_.location())};
  const Scope &progUnit{GetProgramUnitContaining(scope)};
  if (IsCUDADeviceContext(&progUnit) || deviceConstructDepth_ > 0) {
    return;
  }

  auto &outputItemList{std::get<std::list<Fortran::parser::OutputItem>>(x.t)};
  for (const auto &item : outputItemList) {
    if (const auto *x{std::get_if<parser::Expr>(&item.u)}) {
      if (const auto *expr{GetExpr(context_, *x)}) {
        for (const Symbol &sym : CollectCudaSymbols(*expr)) {
          if (const auto *details = sym.GetUltimate()
                  .detailsIf<semantics::ObjectEntityDetails>()) {
            if (details->cudaDataAttr() &&
                (*details->cudaDataAttr() == common::CUDADataAttr::Device ||
                    *details->cudaDataAttr() ==
                        common::CUDADataAttr::Constant ||
                    *details->cudaDataAttr() ==
````
- **L817 EN**: Executes a call or declaration centered on `context_.Say`.
  **L817 CN**: 执行以 `context_.Say` 为核心的调用或声明。
- **L818 EN**: Closes the current lexical scope or compound statement.
  **L818 CN**: 结束当前词法作用域或复合语句块。
- **L819 EN**: Closes the current lexical scope or compound statement.
  **L819 CN**: 结束当前词法作用域或复合语句块。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L821 EN**: Starts a function, method, lambda, or structured scope: `void CUDAChecker::Enter(const parser::PrintStmt &x) {`.
  **L821 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void CUDAChecker::Enter(const parser::PrintStmt &x) {`。
- **L822 EN**: Executes a call or declaration centered on `CHECK`.
  **L822 CN**: 执行以 `CHECK` 为核心的调用或声明。
- **L823 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L823 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L824 EN**: Executes a call or declaration centered on `&progUnit{GetProgramUnitContaining`.
  **L824 CN**: 执行以 `&progUnit{GetProgramUnitContaining` 为核心的调用或声明。
- **L825 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L825 CN**: 开始 `if` 控制流语句并计算其条件。
- **L826 EN**: Returns from the current function with `void`.
  **L826 CN**: 以 `void` 从当前函数返回。
- **L827 EN**: Closes the current lexical scope or compound statement.
  **L827 CN**: 结束当前词法作用域或复合语句块。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L829 EN**: Executes a call or declaration centered on `&outputItemList{std::get<std::list<Fortran::parser::OutputItem>>`.
  **L829 CN**: 执行以 `&outputItemList{std::get<std::list<Fortran::parser::OutputItem>>` 为核心的调用或声明。
- **L830 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L830 CN**: 开始 `for` 控制流语句并计算其条件。
- **L831 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L831 CN**: 开始 `if` 控制流语句并计算其条件。
- **L832 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L832 CN**: 开始 `if` 控制流语句并计算其条件。
- **L833 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L833 CN**: 开始 `for` 控制流语句并计算其条件。
- **L834 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L834 CN**: 开始 `if` 控制流语句并计算其条件。
- **L835 EN**: Starts a function, method, lambda, or structured scope: `.detailsIf<semantics::ObjectEntityDetails>()) {`.
  **L835 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.detailsIf<semantics::ObjectEntityDetails>()) {`。
- **L836 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L836 CN**: 开始 `if` 控制流语句并计算其条件。
- **L837 EN**: Continues logic associated with callable symbol `cudaDataAttr`.
  **L837 CN**: 继续与可调用符号 `cudaDataAttr` 相关的逻辑。
- **L838 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() ==`.
  **L838 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() ==`。
- **L839 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::Constant ||`.
  **L839 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::Constant ||`。
- **L840 EN**: Comment explains nearby logic, intent, or metadata: `details->cudaDataAttr() ==`.
  **L840 CN**: 注释说明附近代码的逻辑、意图或元数据：`details->cudaDataAttr() ==`。

### Lines 841-852

````cpp
                        common::CUDADataAttr::UseDevice)) {
              context_.Say(parser::FindSourceLocation(*x),
                  "device data not allowed in I/O statements"_err_en_US);
            }
          }
        }
      }
    }
  }
}

} // namespace Fortran::semantics
````
- **L841 EN**: Continues the surrounding expression or declaration: `common::CUDADataAttr::UseDevice)) {`.
  **L841 CN**: 继续构造周围的表达式或声明：`common::CUDADataAttr::UseDevice)) {`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(parser::FindSourceLocation(*x),`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(parser::FindSourceLocation(*x),`。
- **L843 EN**: Executes a standalone statement or declaration: `"device data not allowed in I/O statements"_err_en_US);`.
  **L843 CN**: 执行一条独立语句或声明：`"device data not allowed in I/O statements"_err_en_US);`。
- **L844 EN**: Closes the current lexical scope or compound statement.
  **L844 CN**: 结束当前词法作用域或复合语句块。
- **L845 EN**: Closes the current lexical scope or compound statement.
  **L845 CN**: 结束当前词法作用域或复合语句块。
- **L846 EN**: Closes the current lexical scope or compound statement.
  **L846 CN**: 结束当前词法作用域或复合语句块。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Closes the current lexical scope or compound statement.
  **L848 CN**: 结束当前词法作用域或复合语句块。
- **L849 EN**: Closes the current lexical scope or compound statement.
  **L849 CN**: 结束当前词法作用域或复合语句块。
- **L850 EN**: Closes the current lexical scope or compound statement.
  **L850 CN**: 结束当前词法作用域或复合语句块。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L852 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L852 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Semantic-context management / 语义上下文管理**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **OpenACC handling / OpenACC 处理**
- **CUDA-specific lowering or runtime handling / CUDA 专用 lowering 或运行时处理**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-cuda.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/template.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/fold.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Evaluate/traverse.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/parse-tree-visitor.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/expression.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/symbol.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `llvm/ADT/StringSet.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
