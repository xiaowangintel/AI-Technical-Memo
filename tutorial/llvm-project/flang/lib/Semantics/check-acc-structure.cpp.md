# check-acc-structure.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Semantics/check-acc-structure.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements semantic checking, symbol reasoning, and language-rule enforcement for check acc structure.
- **Purpose (CN)**: 实现 check acc structure 相关的语义检查、符号推理与语言规则约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- lib/Semantics/check-acc-structure.cpp -----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
#include "check-acc-structure.h"
#include "resolve-names-utils.h"
#include "flang/Common/enum-set.h"
#include "flang/Evaluate/tools.h"
#include "flang/Parser/parse-tree.h"
#include "flang/Parser/tools.h"
#include "flang/Semantics/symbol.h"
#include "flang/Semantics/tools.h"
#include "flang/Semantics/type.h"
#include "flang/Support/Fortran.h"
#include "llvm/Support/AtomicOrdering.h"

#include <optional>

#define CHECK_SIMPLE_CLAUSE(X, Y) \
  void AccStructureChecker::Enter(const parser::AccClause::X &) { \
    CheckAllowed(llvm::acc::Clause::Y); \
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
- **L8 EN**: Includes "check-acc-structure.h" to access local declarations paired with this implementation.
  **L8 CN**: 引入 "check-acc-structure.h" 以使用与该实现配套的本地声明。
- **L9 EN**: Includes "resolve-names-utils.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "resolve-names-utils.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Common/enum-set.h" to access shared Flang utility infrastructure.
  **L10 CN**: 引入 "flang/Common/enum-set.h" 以使用Flang 共享工具基础设施。
- **L11 EN**: Includes "flang/Evaluate/tools.h" to access Fortran constant-folding and evaluation facilities.
  **L11 CN**: 引入 "flang/Evaluate/tools.h" 以使用Fortran 常量折叠与求值能力。
- **L12 EN**: Includes "flang/Parser/parse-tree.h" to access parse-tree, token, or source representation support.
  **L12 CN**: 引入 "flang/Parser/parse-tree.h" 以使用语法树、词法单元或源码表示支持。
- **L13 EN**: Includes "flang/Parser/tools.h" to access parse-tree, token, or source representation support.
  **L13 CN**: 引入 "flang/Parser/tools.h" 以使用语法树、词法单元或源码表示支持。
- **L14 EN**: Includes "flang/Semantics/symbol.h" to access Fortran semantic analysis, symbol, and type information.
  **L14 CN**: 引入 "flang/Semantics/symbol.h" 以使用Fortran 语义分析、符号与类型信息。
- **L15 EN**: Includes "flang/Semantics/tools.h" to access Fortran semantic analysis, symbol, and type information.
  **L15 CN**: 引入 "flang/Semantics/tools.h" 以使用Fortran 语义分析、符号与类型信息。
- **L16 EN**: Includes "flang/Semantics/type.h" to access Fortran semantic analysis, symbol, and type information.
  **L16 CN**: 引入 "flang/Semantics/type.h" 以使用Fortran 语义分析、符号与类型信息。
- **L17 EN**: Includes "flang/Support/Fortran.h" to access shared Flang utility infrastructure.
  **L17 CN**: 引入 "flang/Support/Fortran.h" 以使用Flang 共享工具基础设施。
- **L18 EN**: Includes "llvm/Support/AtomicOrdering.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L18 CN**: 引入 "llvm/Support/AtomicOrdering.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Includes <optional> to access supporting declarations used by this translation unit.
  **L20 CN**: 引入 <optional> 以使用当前编译单元使用的辅助声明。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L22 EN**: Defines macro `CHECK_SIMPLE_CLAUSE(X,` for conditional compilation or local shorthand.
  **L22 CN**: 定义宏 `CHECK_SIMPLE_CLAUSE(X,`，用于条件编译或本地简写。
- **L23 EN**: Continues logic associated with callable symbol `Enter`.
  **L23 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L24 EN**: Continues logic associated with callable symbol `CheckAllowed`.
  **L24 CN**: 继续与可调用符号 `CheckAllowed` 相关的逻辑。

### Lines 25-48

````cpp
  }

#define CHECK_REQ_SCALAR_INT_CONSTANT_CLAUSE(X, Y) \
  void AccStructureChecker::Enter(const parser::AccClause::X &c) { \
    CheckAllowed(llvm::acc::Clause::Y); \
    RequiresConstantPositiveParameter(llvm::acc::Clause::Y, c.v); \
  }

using ReductionOpsSet =
    Fortran::common::EnumSet<Fortran::parser::ReductionOperator::Operator,
        Fortran::parser::ReductionOperator::Operator_enumSize>;

static ReductionOpsSet reductionIntegerSet{
    Fortran::parser::ReductionOperator::Operator::Plus,
    Fortran::parser::ReductionOperator::Operator::Multiply,
    Fortran::parser::ReductionOperator::Operator::Max,
    Fortran::parser::ReductionOperator::Operator::Min,
    Fortran::parser::ReductionOperator::Operator::Iand,
    Fortran::parser::ReductionOperator::Operator::Ior,
    Fortran::parser::ReductionOperator::Operator::Ieor};

static ReductionOpsSet reductionRealSet{
    Fortran::parser::ReductionOperator::Operator::Plus,
    Fortran::parser::ReductionOperator::Operator::Multiply,
````
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Defines macro `CHECK_REQ_SCALAR_INT_CONSTANT_CLAUSE(X,` for conditional compilation or local shorthand.
  **L27 CN**: 定义宏 `CHECK_REQ_SCALAR_INT_CONSTANT_CLAUSE(X,`，用于条件编译或本地简写。
- **L28 EN**: Continues logic associated with callable symbol `Enter`.
  **L28 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L29 EN**: Continues logic associated with callable symbol `CheckAllowed`.
  **L29 CN**: 继续与可调用符号 `CheckAllowed` 相关的逻辑。
- **L30 EN**: Continues logic associated with callable symbol `RequiresConstantPositiveParameter`.
  **L30 CN**: 继续与可调用符号 `RequiresConstantPositiveParameter` 相关的逻辑。
- **L31 EN**: Closes the current lexical scope or compound statement.
  **L31 CN**: 结束当前词法作用域或复合语句块。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Defines alias `ReductionOpsSet` to simplify later code.
  **L33 CN**: 定义别名 `ReductionOpsSet` 以简化后续代码。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::common::EnumSet<Fortran::parser::ReductionOperator::Operator,`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::common::EnumSet<Fortran::parser::ReductionOperator::Operator,`。
- **L35 EN**: Executes a standalone statement or declaration: `Fortran::parser::ReductionOperator::Operator_enumSize>;`.
  **L35 CN**: 执行一条独立语句或声明：`Fortran::parser::ReductionOperator::Operator_enumSize>;`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Continues the surrounding expression or declaration: `static ReductionOpsSet reductionIntegerSet{`.
  **L37 CN**: 继续构造周围的表达式或声明：`static ReductionOpsSet reductionIntegerSet{`。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Plus,`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Plus,`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Multiply,`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Multiply,`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Max,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Max,`。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Min,`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Min,`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Iand,`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Iand,`。
- **L43 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Ior,`.
  **L43 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Ior,`。
- **L44 EN**: Executes a standalone statement or declaration: `Fortran::parser::ReductionOperator::Operator::Ieor};`.
  **L44 CN**: 执行一条独立语句或声明：`Fortran::parser::ReductionOperator::Operator::Ieor};`。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Continues the surrounding expression or declaration: `static ReductionOpsSet reductionRealSet{`.
  **L46 CN**: 继续构造周围的表达式或声明：`static ReductionOpsSet reductionRealSet{`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Plus,`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Plus,`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Multiply,`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Multiply,`。

### Lines 49-72

````cpp
    Fortran::parser::ReductionOperator::Operator::Max,
    Fortran::parser::ReductionOperator::Operator::Min};

static ReductionOpsSet reductionComplexSet{
    Fortran::parser::ReductionOperator::Operator::Plus,
    Fortran::parser::ReductionOperator::Operator::Multiply};

static ReductionOpsSet reductionLogicalSet{
    Fortran::parser::ReductionOperator::Operator::And,
    Fortran::parser::ReductionOperator::Operator::Or,
    Fortran::parser::ReductionOperator::Operator::Eqv,
    Fortran::parser::ReductionOperator::Operator::Neqv};

namespace Fortran::semantics {

static constexpr inline AccClauseSet
    computeConstructOnlyAllowedAfterDeviceTypeClauses{
        llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait,
        llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_num_workers,
        llvm::acc::Clause::ACCC_vector_length};

static constexpr inline AccClauseSet loopOnlyAllowedAfterDeviceTypeClauses{
    llvm::acc::Clause::ACCC_auto, llvm::acc::Clause::ACCC_collapse,
    llvm::acc::Clause::ACCC_independent, llvm::acc::Clause::ACCC_gang,
````
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Max,`.
  **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Max,`。
- **L50 EN**: Executes a standalone statement or declaration: `Fortran::parser::ReductionOperator::Operator::Min};`.
  **L50 CN**: 执行一条独立语句或声明：`Fortran::parser::ReductionOperator::Operator::Min};`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Continues the surrounding expression or declaration: `static ReductionOpsSet reductionComplexSet{`.
  **L52 CN**: 继续构造周围的表达式或声明：`static ReductionOpsSet reductionComplexSet{`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Plus,`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Plus,`。
- **L54 EN**: Executes a standalone statement or declaration: `Fortran::parser::ReductionOperator::Operator::Multiply};`.
  **L54 CN**: 执行一条独立语句或声明：`Fortran::parser::ReductionOperator::Operator::Multiply};`。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Continues the surrounding expression or declaration: `static ReductionOpsSet reductionLogicalSet{`.
  **L56 CN**: 继续构造周围的表达式或声明：`static ReductionOpsSet reductionLogicalSet{`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::And,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::And,`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Or,`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Or,`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Fortran::parser::ReductionOperator::Operator::Eqv,`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`Fortran::parser::ReductionOperator::Operator::Eqv,`。
- **L60 EN**: Executes a standalone statement or declaration: `Fortran::parser::ReductionOperator::Operator::Neqv};`.
  **L60 CN**: 执行一条独立语句或声明：`Fortran::parser::ReductionOperator::Operator::Neqv};`。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Opens namespace scope `Fortran::semantics`.
  **L62 CN**: 打开命名空间作用域 `Fortran::semantics`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Continues the surrounding expression or declaration: `static constexpr inline AccClauseSet`.
  **L64 CN**: 继续构造周围的表达式或声明：`static constexpr inline AccClauseSet`。
- **L65 EN**: Continues the surrounding expression or declaration: `computeConstructOnlyAllowedAfterDeviceTypeClauses{`.
  **L65 CN**: 继续构造周围的表达式或声明：`computeConstructOnlyAllowedAfterDeviceTypeClauses{`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait,`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_num_workers,`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_num_workers,`。
- **L68 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_vector_length};`.
  **L68 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_vector_length};`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Continues the surrounding expression or declaration: `static constexpr inline AccClauseSet loopOnlyAllowedAfterDeviceTypeClauses{`.
  **L70 CN**: 继续构造周围的表达式或声明：`static constexpr inline AccClauseSet loopOnlyAllowedAfterDeviceTypeClauses{`。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_auto, llvm::acc::Clause::ACCC_collapse,`.
  **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_auto, llvm::acc::Clause::ACCC_collapse,`。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_independent, llvm::acc::Clause::ACCC_gang,`.
  **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_independent, llvm::acc::Clause::ACCC_gang,`。

### Lines 73-96

````cpp
    llvm::acc::Clause::ACCC_seq, llvm::acc::Clause::ACCC_tile,
    llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker};

static constexpr inline AccClauseSet updateOnlyAllowedAfterDeviceTypeClauses{
    llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait};

static constexpr inline AccClauseSet routineOnlyAllowedAfterDeviceTypeClauses{
    llvm::acc::Clause::ACCC_bind, llvm::acc::Clause::ACCC_gang,
    llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker,
    llvm::acc::Clause::ACCC_seq};

static constexpr inline AccClauseSet routineMutuallyExclusiveClauses{
    llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_worker,
    llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_seq};

bool AccStructureChecker::CheckAllowedModifier(llvm::acc::Clause clause) {
  if (GetContext().directive == llvm::acc::ACCD_enter_data ||
      GetContext().directive == llvm::acc::ACCD_exit_data) {
    context_.Say(GetContext().clauseSource,
        "Modifier is not allowed for the %s clause "
        "on the %s directive"_err_en_US,
        parser::ToUpperCaseLetters(getClauseName(clause).str()),
        ContextDirectiveAsFortran());
    return true;
````
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_seq, llvm::acc::Clause::ACCC_tile,`.
  **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_seq, llvm::acc::Clause::ACCC_tile,`。
- **L74 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker};`.
  **L74 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker};`。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Continues the surrounding expression or declaration: `static constexpr inline AccClauseSet updateOnlyAllowedAfterDeviceTypeClauses{`.
  **L76 CN**: 继续构造周围的表达式或声明：`static constexpr inline AccClauseSet updateOnlyAllowedAfterDeviceTypeClauses{`。
- **L77 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait};`.
  **L77 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_async, llvm::acc::Clause::ACCC_wait};`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L79 EN**: Continues the surrounding expression or declaration: `static constexpr inline AccClauseSet routineOnlyAllowedAfterDeviceTypeClauses{`.
  **L79 CN**: 继续构造周围的表达式或声明：`static constexpr inline AccClauseSet routineOnlyAllowedAfterDeviceTypeClauses{`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_bind, llvm::acc::Clause::ACCC_gang,`.
  **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_bind, llvm::acc::Clause::ACCC_gang,`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker,`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_worker,`。
- **L82 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_seq};`.
  **L82 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_seq};`。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Continues the surrounding expression or declaration: `static constexpr inline AccClauseSet routineMutuallyExclusiveClauses{`.
  **L84 CN**: 继续构造周围的表达式或声明：`static constexpr inline AccClauseSet routineMutuallyExclusiveClauses{`。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_worker,`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_worker,`。
- **L86 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_seq};`.
  **L86 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_vector, llvm::acc::Clause::ACCC_seq};`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Starts a function, method, lambda, or structured scope: `bool AccStructureChecker::CheckAllowedModifier(llvm::acc::Clause clause) {`.
  **L88 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AccStructureChecker::CheckAllowedModifier(llvm::acc::Clause clause) {`。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Starts a function, method, lambda, or structured scope: `GetContext().directive == llvm::acc::ACCD_exit_data) {`.
  **L90 CN**: 开始一个函数、方法、lambda 或结构化作用域：`GetContext().directive == llvm::acc::ACCD_exit_data) {`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L92 EN**: Continues the surrounding expression or declaration: `"Modifier is not allowed for the %s clause "`.
  **L92 CN**: 继续构造周围的表达式或声明：`"Modifier is not allowed for the %s clause "`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `parser::ToUpperCaseLetters(getClauseName(clause).str()),`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`parser::ToUpperCaseLetters(getClauseName(clause).str()),`。
- **L95 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L95 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L96 EN**: Returns from the current function with `true`.
  **L96 CN**: 以 `true` 从当前函数返回。

### Lines 97-120

````cpp
  }
  return false;
}

bool AccStructureChecker::IsComputeConstruct(
    llvm::acc::Directive directive) const {
  return directive == llvm::acc::ACCD_parallel ||
      directive == llvm::acc::ACCD_parallel_loop ||
      directive == llvm::acc::ACCD_serial ||
      directive == llvm::acc::ACCD_serial_loop ||
      directive == llvm::acc::ACCD_kernels ||
      directive == llvm::acc::ACCD_kernels_loop;
}

bool AccStructureChecker::IsLoopConstruct(
    llvm::acc::Directive directive) const {
  return directive == llvm::acc::Directive::ACCD_loop ||
      directive == llvm::acc::ACCD_parallel_loop ||
      directive == llvm::acc::ACCD_serial_loop ||
      directive == llvm::acc::ACCD_kernels_loop;
}

std::optional<llvm::acc::Directive>
AccStructureChecker::getParentComputeConstruct() const {
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Returns from the current function with `false`.
  **L98 CN**: 以 `false` 从当前函数返回。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Continues logic associated with callable symbol `IsComputeConstruct`.
  **L101 CN**: 继续与可调用符号 `IsComputeConstruct` 相关的逻辑。
- **L102 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive directive) const {`.
  **L102 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive directive) const {`。
- **L103 EN**: Returns from the current function with `directive == llvm::acc::ACCD_parallel ||`.
  **L103 CN**: 以 `directive == llvm::acc::ACCD_parallel ||` 从当前函数返回。
- **L104 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_parallel_loop ||`.
  **L104 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_parallel_loop ||`。
- **L105 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_serial ||`.
  **L105 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_serial ||`。
- **L106 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_serial_loop ||`.
  **L106 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_serial_loop ||`。
- **L107 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_kernels ||`.
  **L107 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_kernels ||`。
- **L108 EN**: Executes a standalone statement or declaration: `directive == llvm::acc::ACCD_kernels_loop;`.
  **L108 CN**: 执行一条独立语句或声明：`directive == llvm::acc::ACCD_kernels_loop;`。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Continues logic associated with callable symbol `IsLoopConstruct`.
  **L111 CN**: 继续与可调用符号 `IsLoopConstruct` 相关的逻辑。
- **L112 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive directive) const {`.
  **L112 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive directive) const {`。
- **L113 EN**: Returns from the current function with `directive == llvm::acc::Directive::ACCD_loop ||`.
  **L113 CN**: 以 `directive == llvm::acc::Directive::ACCD_loop ||` 从当前函数返回。
- **L114 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_parallel_loop ||`.
  **L114 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_parallel_loop ||`。
- **L115 EN**: Continues the surrounding expression or declaration: `directive == llvm::acc::ACCD_serial_loop ||`.
  **L115 CN**: 继续构造周围的表达式或声明：`directive == llvm::acc::ACCD_serial_loop ||`。
- **L116 EN**: Executes a standalone statement or declaration: `directive == llvm::acc::ACCD_kernels_loop;`.
  **L116 CN**: 执行一条独立语句或声明：`directive == llvm::acc::ACCD_kernels_loop;`。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L119 EN**: Continues the surrounding expression or declaration: `std::optional<llvm::acc::Directive>`.
  **L119 CN**: 继续构造周围的表达式或声明：`std::optional<llvm::acc::Directive>`。
- **L120 EN**: Starts a function, method, lambda, or structured scope: `AccStructureChecker::getParentComputeConstruct() const {`.
  **L120 CN**: 开始一个函数、方法、lambda 或结构化作用域：`AccStructureChecker::getParentComputeConstruct() const {`。

### Lines 121-144

````cpp
  // Check all nested context skipping the first one.
  for (std::size_t i = dirContext_.size() - 1; i > 0; --i)
    if (IsComputeConstruct(dirContext_[i - 1].directive))
      return dirContext_[i - 1].directive;
  return std::nullopt;
}

bool AccStructureChecker::IsInsideComputeConstruct() const {
  return getParentComputeConstruct().has_value();
}

void AccStructureChecker::CheckNotInComputeConstruct() {
  if (IsInsideComputeConstruct()) {
    context_.Say(GetContext().directiveSource,
        "Directive %s may not be called within a compute region"_err_en_US,
        ContextDirectiveAsFortran());
  }
}

bool AccStructureChecker::IsInsideKernelsConstruct() const {
  if (auto directive = getParentComputeConstruct())
    if (*directive == llvm::acc::ACCD_kernels ||
        *directive == llvm::acc::ACCD_kernels_loop)
      return true;
````
- **L121 EN**: Comment explains nearby logic, intent, or metadata: `Check all nested context skipping the first one.`.
  **L121 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check all nested context skipping the first one.`。
- **L122 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L122 CN**: 开始 `for` 控制流语句并计算其条件。
- **L123 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L123 CN**: 开始 `if` 控制流语句并计算其条件。
- **L124 EN**: Returns from the current function with `dirContext_[i - 1].directive`.
  **L124 CN**: 以 `dirContext_[i - 1].directive` 从当前函数返回。
- **L125 EN**: Returns from the current function with `std::nullopt`.
  **L125 CN**: 以 `std::nullopt` 从当前函数返回。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Starts a function, method, lambda, or structured scope: `bool AccStructureChecker::IsInsideComputeConstruct() const {`.
  **L128 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AccStructureChecker::IsInsideComputeConstruct() const {`。
- **L129 EN**: Returns from the current function with `getParentComputeConstruct().has_value()`.
  **L129 CN**: 以 `getParentComputeConstruct().has_value()` 从当前函数返回。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L132 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::CheckNotInComputeConstruct() {`.
  **L132 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::CheckNotInComputeConstruct() {`。
- **L133 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L133 CN**: 开始 `if` 控制流语句并计算其条件。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Directive %s may not be called within a compute region"_err_en_US,`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Directive %s may not be called within a compute region"_err_en_US,`。
- **L136 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L136 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L140 EN**: Starts a function, method, lambda, or structured scope: `bool AccStructureChecker::IsInsideKernelsConstruct() const {`.
  **L140 CN**: 开始一个函数、方法、lambda 或结构化作用域：`bool AccStructureChecker::IsInsideKernelsConstruct() const {`。
- **L141 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L141 CN**: 开始 `if` 控制流语句并计算其条件。
- **L142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L143 EN**: Comment explains nearby logic, intent, or metadata: `directive == llvm::acc::ACCD_kernels_loop)`.
  **L143 CN**: 注释说明附近代码的逻辑、意图或元数据：`directive == llvm::acc::ACCD_kernels_loop)`。
- **L144 EN**: Returns from the current function with `true`.
  **L144 CN**: 以 `true` 从当前函数返回。

### Lines 145-168

````cpp
  return false;
}

void AccStructureChecker::Enter(const parser::AccClause &x) {
  SetContextClause(x);
}

void AccStructureChecker::Leave(const parser::AccClauseList &) {}

void AccStructureChecker::Enter(const parser::OpenACCBlockConstruct &x) {
  const auto &beginBlockDir{std::get<parser::AccBeginBlockDirective>(x.t)};
  const auto &endBlockDir{std::get<parser::AccEndBlockDirective>(x.t)};
  const auto &beginAccBlockDir{
      std::get<parser::AccBlockDirective>(beginBlockDir.t)};

  CheckMatching(beginAccBlockDir, endBlockDir.v);
  PushContextAndClauseSets(beginAccBlockDir.source, beginAccBlockDir.v);
}

void AccStructureChecker::Leave(const parser::OpenACCBlockConstruct &x) {
  const auto &beginBlockDir{std::get<parser::AccBeginBlockDirective>(x.t)};
  const auto &blockDir{std::get<parser::AccBlockDirective>(beginBlockDir.t)};
  const parser::Block &block{std::get<parser::Block>(x.t)};
  switch (blockDir.v) {
````
- **L145 EN**: Returns from the current function with `false`.
  **L145 CN**: 以 `false` 从当前函数返回。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause &x) {`.
  **L148 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause &x) {`。
- **L149 EN**: Executes a call or declaration centered on `SetContextClause`.
  **L149 CN**: 执行以 `SetContextClause` 为核心的调用或声明。
- **L150 EN**: Closes the current lexical scope or compound statement.
  **L150 CN**: 结束当前词法作用域或复合语句块。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Continues logic associated with callable symbol `Leave`.
  **L152 CN**: 继续与可调用符号 `Leave` 相关的逻辑。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCBlockConstruct &x) {`.
  **L154 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCBlockConstruct &x) {`。
- **L155 EN**: Executes a call or declaration centered on `&beginBlockDir{std::get<parser::AccBeginBlockDirective>`.
  **L155 CN**: 执行以 `&beginBlockDir{std::get<parser::AccBeginBlockDirective>` 为核心的调用或声明。
- **L156 EN**: Executes a call or declaration centered on `&endBlockDir{std::get<parser::AccEndBlockDirective>`.
  **L156 CN**: 执行以 `&endBlockDir{std::get<parser::AccEndBlockDirective>` 为核心的调用或声明。
- **L157 EN**: Continues the surrounding expression or declaration: `const auto &beginAccBlockDir{`.
  **L157 CN**: 继续构造周围的表达式或声明：`const auto &beginAccBlockDir{`。
- **L158 EN**: Executes a call or declaration centered on `std::get<parser::AccBlockDirective>`.
  **L158 CN**: 执行以 `std::get<parser::AccBlockDirective>` 为核心的调用或声明。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Executes a call or declaration centered on `CheckMatching`.
  **L160 CN**: 执行以 `CheckMatching` 为核心的调用或声明。
- **L161 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L161 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCBlockConstruct &x) {`.
  **L164 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCBlockConstruct &x) {`。
- **L165 EN**: Executes a call or declaration centered on `&beginBlockDir{std::get<parser::AccBeginBlockDirective>`.
  **L165 CN**: 执行以 `&beginBlockDir{std::get<parser::AccBeginBlockDirective>` 为核心的调用或声明。
- **L166 EN**: Executes a call or declaration centered on `&blockDir{std::get<parser::AccBlockDirective>`.
  **L166 CN**: 执行以 `&blockDir{std::get<parser::AccBlockDirective>` 为核心的调用或声明。
- **L167 EN**: Executes a call or declaration centered on `&block{std::get<parser::Block>`.
  **L167 CN**: 执行以 `&block{std::get<parser::Block>` 为核心的调用或声明。
- **L168 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L168 CN**: 开始 `switch` 控制流语句并计算其条件。

### Lines 169-192

````cpp
  case llvm::acc::Directive::ACCD_kernels:
  case llvm::acc::Directive::ACCD_parallel:
  case llvm::acc::Directive::ACCD_serial:
    // Restriction - line 1004-1005
    CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,
        computeConstructOnlyAllowedAfterDeviceTypeClauses);
    // Restriction - line 1001
    CheckNoBranching(block, GetContext().directive, blockDir.source);
    break;
  case llvm::acc::Directive::ACCD_data:
    // Restriction - 2.6.5 pt 1
    // Only a warning is emitted here for portability reason.
    CheckRequireAtLeastOneOf(/*warnInsteadOfError=*/true);
    // Restriction is not formally in the specification but all compilers emit
    // an error and it is likely to be omitted from the spec.
    CheckNoBranching(block, GetContext().directive, blockDir.source);
    break;
  case llvm::acc::Directive::ACCD_host_data:
    // Restriction - line 1746
    CheckRequireAtLeastOneOf();
    break;
  default:
    break;
  }
````
- **L169 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_kernels:`.
  **L169 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_kernels:`。
- **L170 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_parallel:`.
  **L170 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_parallel:`。
- **L171 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_serial:`.
  **L171 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_serial:`。
- **L172 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1004-1005`.
  **L172 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1004-1005`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`。
- **L174 EN**: Executes a standalone statement or declaration: `computeConstructOnlyAllowedAfterDeviceTypeClauses);`.
  **L174 CN**: 执行一条独立语句或声明：`computeConstructOnlyAllowedAfterDeviceTypeClauses);`。
- **L175 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1001`.
  **L175 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1001`。
- **L176 EN**: Executes a call or declaration centered on `CheckNoBranching`.
  **L176 CN**: 执行以 `CheckNoBranching` 为核心的调用或声明。
- **L177 EN**: Exits the nearest loop or switch statement.
  **L177 CN**: 退出最近的循环或 switch 语句。
- **L178 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_data:`.
  **L178 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_data:`。
- **L179 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - 2.6.5 pt 1`.
  **L179 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - 2.6.5 pt 1`。
- **L180 EN**: Comment explains nearby logic, intent, or metadata: `Only a warning is emitted here for portability reason.`.
  **L180 CN**: 注释说明附近代码的逻辑、意图或元数据：`Only a warning is emitted here for portability reason.`。
- **L181 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L181 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L182 EN**: Comment explains nearby logic, intent, or metadata: `Restriction is not formally in the specification but all compilers emit`.
  **L182 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction is not formally in the specification but all compilers emit`。
- **L183 EN**: Comment explains nearby logic, intent, or metadata: `an error and it is likely to be omitted from the spec.`.
  **L183 CN**: 注释说明附近代码的逻辑、意图或元数据：`an error and it is likely to be omitted from the spec.`。
- **L184 EN**: Executes a call or declaration centered on `CheckNoBranching`.
  **L184 CN**: 执行以 `CheckNoBranching` 为核心的调用或声明。
- **L185 EN**: Exits the nearest loop or switch statement.
  **L185 CN**: 退出最近的循环或 switch 语句。
- **L186 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_host_data:`.
  **L186 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_host_data:`。
- **L187 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1746`.
  **L187 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1746`。
- **L188 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L188 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L189 EN**: Exits the nearest loop or switch statement.
  **L189 CN**: 退出最近的循环或 switch 语句。
- **L190 EN**: Introduces a switch dispatch label: `default:`.
  **L190 CN**: 引入一个 switch 分发标签：`default:`。
- **L191 EN**: Exits the nearest loop or switch statement.
  **L191 CN**: 退出最近的循环或 switch 语句。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。

### Lines 193-216

````cpp
  dirContext_.pop_back();
}

void AccStructureChecker::Enter(
    const parser::OpenACCStandaloneDeclarativeConstruct &x) {
  const auto &declarativeDir{std::get<parser::AccDeclarativeDirective>(x.t)};
  PushContextAndClauseSets(declarativeDir.source, declarativeDir.v);
}

void AccStructureChecker::Leave(
    const parser::OpenACCStandaloneDeclarativeConstruct &x) {
  // Restriction - line 2409
  CheckAtLeastOneClause();

  // Restriction - line 2417-2418 - In a Fortran module declaration section,
  // only create, copyin, device_resident, and link clauses are allowed.
  const auto &declarativeDir{std::get<parser::AccDeclarativeDirective>(x.t)};
  const auto &scope{context_.FindScope(declarativeDir.source)};
  const Scope &containingScope{GetProgramUnitContaining(scope)};
  if (containingScope.kind() == Scope::Kind::Module) {
    for (auto cl : GetContext().actualClauses) {
      if (cl != llvm::acc::Clause::ACCC_create &&
          cl != llvm::acc::Clause::ACCC_copyin &&
          cl != llvm::acc::Clause::ACCC_device_resident &&
````
- **L193 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L193 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L196 EN**: Continues logic associated with callable symbol `Enter`.
  **L196 CN**: 继续与可调用符号 `Enter` 相关的逻辑。
- **L197 EN**: Continues the surrounding expression or declaration: `const parser::OpenACCStandaloneDeclarativeConstruct &x) {`.
  **L197 CN**: 继续构造周围的表达式或声明：`const parser::OpenACCStandaloneDeclarativeConstruct &x) {`。
- **L198 EN**: Executes a call or declaration centered on `&declarativeDir{std::get<parser::AccDeclarativeDirective>`.
  **L198 CN**: 执行以 `&declarativeDir{std::get<parser::AccDeclarativeDirective>` 为核心的调用或声明。
- **L199 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L199 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L202 EN**: Continues logic associated with callable symbol `Leave`.
  **L202 CN**: 继续与可调用符号 `Leave` 相关的逻辑。
- **L203 EN**: Continues the surrounding expression or declaration: `const parser::OpenACCStandaloneDeclarativeConstruct &x) {`.
  **L203 CN**: 继续构造周围的表达式或声明：`const parser::OpenACCStandaloneDeclarativeConstruct &x) {`。
- **L204 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2409`.
  **L204 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2409`。
- **L205 EN**: Executes a call or declaration centered on `CheckAtLeastOneClause`.
  **L205 CN**: 执行以 `CheckAtLeastOneClause` 为核心的调用或声明。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L207 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2417-2418 - In a Fortran module declaration section,`.
  **L207 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2417-2418 - In a Fortran module declaration section,`。
- **L208 EN**: Comment explains nearby logic, intent, or metadata: `only create, copyin, device_resident, and link clauses are allowed.`.
  **L208 CN**: 注释说明附近代码的逻辑、意图或元数据：`only create, copyin, device_resident, and link clauses are allowed.`。
- **L209 EN**: Executes a call or declaration centered on `&declarativeDir{std::get<parser::AccDeclarativeDirective>`.
  **L209 CN**: 执行以 `&declarativeDir{std::get<parser::AccDeclarativeDirective>` 为核心的调用或声明。
- **L210 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L210 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L211 EN**: Executes a call or declaration centered on `&containingScope{GetProgramUnitContaining`.
  **L211 CN**: 执行以 `&containingScope{GetProgramUnitContaining` 为核心的调用或声明。
- **L212 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L212 CN**: 开始 `if` 控制流语句并计算其条件。
- **L213 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L213 CN**: 开始 `for` 控制流语句并计算其条件。
- **L214 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L214 CN**: 开始 `if` 控制流语句并计算其条件。
- **L215 EN**: Continues the surrounding expression or declaration: `cl != llvm::acc::Clause::ACCC_copyin &&`.
  **L215 CN**: 继续构造周围的表达式或声明：`cl != llvm::acc::Clause::ACCC_copyin &&`。
- **L216 EN**: Continues the surrounding expression or declaration: `cl != llvm::acc::Clause::ACCC_device_resident &&`.
  **L216 CN**: 继续构造周围的表达式或声明：`cl != llvm::acc::Clause::ACCC_device_resident &&`。

### Lines 217-240

````cpp
          cl != llvm::acc::Clause::ACCC_link) {
        context_.Say(GetContext().directiveSource,
            "%s clause is not allowed on the %s directive in module "
            "declaration "
            "section"_err_en_US,
            parser::ToUpperCaseLetters(
                llvm::acc::getOpenACCClauseName(cl).str()),
            ContextDirectiveAsFortran());
      }
    }
  }
  dirContext_.pop_back();
}

void AccStructureChecker::Enter(const parser::OpenACCCombinedConstruct &x) {
  const auto &beginCombinedDir{
      std::get<parser::AccBeginCombinedDirective>(x.t)};
  const auto &combinedDir{
      std::get<parser::AccCombinedDirective>(beginCombinedDir.t)};

  // check matching, End directive is optional
  if (const auto &endCombinedDir{
          std::get<std::optional<parser::AccEndCombinedDirective>>(x.t)}) {
    CheckMatching<parser::AccCombinedDirective>(combinedDir, endCombinedDir->v);
````
- **L217 EN**: Continues the surrounding expression or declaration: `cl != llvm::acc::Clause::ACCC_link) {`.
  **L217 CN**: 继续构造周围的表达式或声明：`cl != llvm::acc::Clause::ACCC_link) {`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L219 EN**: Continues the surrounding expression or declaration: `"%s clause is not allowed on the %s directive in module "`.
  **L219 CN**: 继续构造周围的表达式或声明：`"%s clause is not allowed on the %s directive in module "`。
- **L220 EN**: Continues the surrounding expression or declaration: `"declaration "`.
  **L220 CN**: 继续构造周围的表达式或声明：`"declaration "`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"section"_err_en_US,`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`"section"_err_en_US,`。
- **L222 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L222 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::getOpenACCClauseName(cl).str()),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::getOpenACCClauseName(cl).str()),`。
- **L224 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L224 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L225 EN**: Closes the current lexical scope or compound statement.
  **L225 CN**: 结束当前词法作用域或复合语句块。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L228 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L229 EN**: Closes the current lexical scope or compound statement.
  **L229 CN**: 结束当前词法作用域或复合语句块。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCCombinedConstruct &x) {`.
  **L231 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCCombinedConstruct &x) {`。
- **L232 EN**: Continues the surrounding expression or declaration: `const auto &beginCombinedDir{`.
  **L232 CN**: 继续构造周围的表达式或声明：`const auto &beginCombinedDir{`。
- **L233 EN**: Executes a call or declaration centered on `std::get<parser::AccBeginCombinedDirective>`.
  **L233 CN**: 执行以 `std::get<parser::AccBeginCombinedDirective>` 为核心的调用或声明。
- **L234 EN**: Continues the surrounding expression or declaration: `const auto &combinedDir{`.
  **L234 CN**: 继续构造周围的表达式或声明：`const auto &combinedDir{`。
- **L235 EN**: Executes a call or declaration centered on `std::get<parser::AccCombinedDirective>`.
  **L235 CN**: 执行以 `std::get<parser::AccCombinedDirective>` 为核心的调用或声明。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Comment explains nearby logic, intent, or metadata: `check matching, End directive is optional`.
  **L237 CN**: 注释说明附近代码的逻辑、意图或元数据：`check matching, End directive is optional`。
- **L238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L239 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::AccEndCombinedDirective>>(x.t)}) {`.
  **L239 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::AccEndCombinedDirective>>(x.t)}) {`。
- **L240 EN**: Executes a call or declaration centered on `CheckMatching<parser::AccCombinedDirective>`.
  **L240 CN**: 执行以 `CheckMatching<parser::AccCombinedDirective>` 为核心的调用或声明。

### Lines 241-264

````cpp
  }

  PushContextAndClauseSets(combinedDir.source, combinedDir.v);
}

void AccStructureChecker::Leave(const parser::OpenACCCombinedConstruct &x) {
  const auto &beginBlockDir{std::get<parser::AccBeginCombinedDirective>(x.t)};
  const auto &combinedDir{
      std::get<parser::AccCombinedDirective>(beginBlockDir.t)};
  auto &doCons{std::get<std::optional<parser::DoConstruct>>(x.t)};
  switch (combinedDir.v) {
  case llvm::acc::Directive::ACCD_kernels_loop:
  case llvm::acc::Directive::ACCD_parallel_loop:
  case llvm::acc::Directive::ACCD_serial_loop:
    // Restriction - line 1004-1005
    CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,
        computeConstructOnlyAllowedAfterDeviceTypeClauses |
            loopOnlyAllowedAfterDeviceTypeClauses);
    if (doCons) {
      const parser::Block &block{std::get<parser::Block>(doCons->t)};
      CheckNoBranching(block, GetContext().directive, beginBlockDir.source);
    }
    break;
  default:
````
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L243 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L243 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCCombinedConstruct &x) {`.
  **L246 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCCombinedConstruct &x) {`。
- **L247 EN**: Executes a call or declaration centered on `&beginBlockDir{std::get<parser::AccBeginCombinedDirective>`.
  **L247 CN**: 执行以 `&beginBlockDir{std::get<parser::AccBeginCombinedDirective>` 为核心的调用或声明。
- **L248 EN**: Continues the surrounding expression or declaration: `const auto &combinedDir{`.
  **L248 CN**: 继续构造周围的表达式或声明：`const auto &combinedDir{`。
- **L249 EN**: Executes a call or declaration centered on `std::get<parser::AccCombinedDirective>`.
  **L249 CN**: 执行以 `std::get<parser::AccCombinedDirective>` 为核心的调用或声明。
- **L250 EN**: Executes a call or declaration centered on `&doCons{std::get<std::optional<parser::DoConstruct>>`.
  **L250 CN**: 执行以 `&doCons{std::get<std::optional<parser::DoConstruct>>` 为核心的调用或声明。
- **L251 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L251 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L252 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_kernels_loop:`.
  **L252 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_kernels_loop:`。
- **L253 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_parallel_loop:`.
  **L253 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_parallel_loop:`。
- **L254 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_serial_loop:`.
  **L254 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_serial_loop:`。
- **L255 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1004-1005`.
  **L255 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1004-1005`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`。
- **L257 EN**: Continues the surrounding expression or declaration: `computeConstructOnlyAllowedAfterDeviceTypeClauses |`.
  **L257 CN**: 继续构造周围的表达式或声明：`computeConstructOnlyAllowedAfterDeviceTypeClauses |`。
- **L258 EN**: Executes a standalone statement or declaration: `loopOnlyAllowedAfterDeviceTypeClauses);`.
  **L258 CN**: 执行一条独立语句或声明：`loopOnlyAllowedAfterDeviceTypeClauses);`。
- **L259 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L259 CN**: 开始 `if` 控制流语句并计算其条件。
- **L260 EN**: Executes a call or declaration centered on `&block{std::get<parser::Block>`.
  **L260 CN**: 执行以 `&block{std::get<parser::Block>` 为核心的调用或声明。
- **L261 EN**: Executes a call or declaration centered on `CheckNoBranching`.
  **L261 CN**: 执行以 `CheckNoBranching` 为核心的调用或声明。
- **L262 EN**: Closes the current lexical scope or compound statement.
  **L262 CN**: 结束当前词法作用域或复合语句块。
- **L263 EN**: Exits the nearest loop or switch statement.
  **L263 CN**: 退出最近的循环或 switch 语句。
- **L264 EN**: Introduces a switch dispatch label: `default:`.
  **L264 CN**: 引入一个 switch 分发标签：`default:`。

### Lines 265-288

````cpp
    break;
  }
  dirContext_.pop_back();
}

std::optional<std::int64_t> AccStructureChecker::getGangDimensionSize(
    DirectiveContext &dirContext) {
  for (auto it : dirContext.clauseInfo) {
    const auto *clause{it.second};
    if (const auto *gangClause{
            std::get_if<parser::AccClause::Gang>(&clause->u)})
      if (gangClause->v) {
        const Fortran::parser::AccGangArgList &x{*gangClause->v};
        for (const Fortran::parser::AccGangArg &gangArg : x.v)
          if (const auto *dim{
                  std::get_if<Fortran::parser::AccGangArg::Dim>(&gangArg.u)})
            if (const auto v{EvaluateInt64(context_, dim->v)})
              return *v;
      }
  }
  return std::nullopt;
}

void AccStructureChecker::CheckNotInSameOrSubLevelLoopConstruct() {
````
- **L265 EN**: Exits the nearest loop or switch statement.
  **L265 CN**: 退出最近的循环或 switch 语句。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L267 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L268 EN**: Closes the current lexical scope or compound statement.
  **L268 CN**: 结束当前词法作用域或复合语句块。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Continues logic associated with callable symbol `getGangDimensionSize`.
  **L270 CN**: 继续与可调用符号 `getGangDimensionSize` 相关的逻辑。
- **L271 EN**: Continues the surrounding expression or declaration: `DirectiveContext &dirContext) {`.
  **L271 CN**: 继续构造周围的表达式或声明：`DirectiveContext &dirContext) {`。
- **L272 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L272 CN**: 开始 `for` 控制流语句并计算其条件。
- **L273 EN**: Executes a standalone statement or declaration: `const auto *clause{it.second};`.
  **L273 CN**: 执行一条独立语句或声明：`const auto *clause{it.second};`。
- **L274 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L274 CN**: 开始 `if` 控制流语句并计算其条件。
- **L275 EN**: Continues logic associated with callable symbol `Gang>`.
  **L275 CN**: 继续与可调用符号 `Gang>` 相关的逻辑。
- **L276 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L276 CN**: 开始 `if` 控制流语句并计算其条件。
- **L277 EN**: Executes a standalone statement or declaration: `const Fortran::parser::AccGangArgList &x{*gangClause->v};`.
  **L277 CN**: 执行一条独立语句或声明：`const Fortran::parser::AccGangArgList &x{*gangClause->v};`。
- **L278 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L278 CN**: 开始 `for` 控制流语句并计算其条件。
- **L279 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L279 CN**: 开始 `if` 控制流语句并计算其条件。
- **L280 EN**: Continues logic associated with callable symbol `Dim>`.
  **L280 CN**: 继续与可调用符号 `Dim>` 相关的逻辑。
- **L281 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L281 CN**: 开始 `if` 控制流语句并计算其条件。
- **L282 EN**: Returns from the current function with `*v`.
  **L282 CN**: 以 `*v` 从当前函数返回。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Returns from the current function with `std::nullopt`.
  **L285 CN**: 以 `std::nullopt` 从当前函数返回。
- **L286 EN**: Closes the current lexical scope or compound statement.
  **L286 CN**: 结束当前词法作用域或复合语句块。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L288 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::CheckNotInSameOrSubLevelLoopConstruct() {`.
  **L288 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::CheckNotInSameOrSubLevelLoopConstruct() {`。

### Lines 289-312

````cpp
  for (std::size_t i = dirContext_.size() - 1; i > 0; --i) {
    auto &parent{dirContext_[i - 1]};
    if (IsLoopConstruct(parent.directive)) {
      for (auto parentClause : parent.actualClauses) {
        for (auto cl : GetContext().actualClauses) {
          bool invalid{false};
          if (parentClause == llvm::acc::Clause::ACCC_gang &&
              cl == llvm::acc::Clause::ACCC_gang) {
            if (IsInsideKernelsConstruct()) {
              context_.Say(GetContext().clauseSource,
                  "Nested GANG loops are not allowed in the region of a KERNELS construct"_err_en_US);
            } else {
              auto parentDim = getGangDimensionSize(parent);
              auto currentDim = getGangDimensionSize(GetContext());
              std::int64_t parentDimNum = 1, currentDimNum = 1;
              if (parentDim)
                parentDimNum = *parentDim;
              if (currentDim)
                currentDimNum = *currentDim;
              if (parentDimNum <= currentDimNum) {
                std::string parentDimStr, currentDimStr;
                if (parentDim)
                  parentDimStr = "(dim:" + std::to_string(parentDimNum) + ")";
                if (currentDim)
````
- **L289 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L289 CN**: 开始 `for` 控制流语句并计算其条件。
- **L290 EN**: Executes a standalone statement or declaration: `auto &parent{dirContext_[i - 1]};`.
  **L290 CN**: 执行一条独立语句或声明：`auto &parent{dirContext_[i - 1]};`。
- **L291 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L291 CN**: 开始 `if` 控制流语句并计算其条件。
- **L292 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L292 CN**: 开始 `for` 控制流语句并计算其条件。
- **L293 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L293 CN**: 开始 `for` 控制流语句并计算其条件。
- **L294 EN**: Executes a standalone statement or declaration: `bool invalid{false};`.
  **L294 CN**: 执行一条独立语句或声明：`bool invalid{false};`。
- **L295 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L295 CN**: 开始 `if` 控制流语句并计算其条件。
- **L296 EN**: Continues the surrounding expression or declaration: `cl == llvm::acc::Clause::ACCC_gang) {`.
  **L296 CN**: 继续构造周围的表达式或声明：`cl == llvm::acc::Clause::ACCC_gang) {`。
- **L297 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L297 CN**: 开始 `if` 控制流语句并计算其条件。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L299 EN**: Executes a standalone statement or declaration: `"Nested GANG loops are not allowed in the region of a KERNELS construct"_err_en_US);`.
  **L299 CN**: 执行一条独立语句或声明：`"Nested GANG loops are not allowed in the region of a KERNELS construct"_err_en_US);`。
- **L300 EN**: Transitions from the previous branch into the alternative path.
  **L300 CN**: 从前一个分支过渡到备选路径。
- **L301 EN**: Initializes variable `parentDim` from the right-hand expression.
  **L301 CN**: 使用右侧表达式初始化变量 `parentDim`。
- **L302 EN**: Initializes variable `currentDim` from the right-hand expression.
  **L302 CN**: 使用右侧表达式初始化变量 `currentDim`。
- **L303 EN**: Initializes variable `parentDimNum` from the right-hand expression.
  **L303 CN**: 使用右侧表达式初始化变量 `parentDimNum`。
- **L304 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L304 CN**: 开始 `if` 控制流语句并计算其条件。
- **L305 EN**: Executes a standalone statement or declaration: `parentDimNum = *parentDim;`.
  **L305 CN**: 执行一条独立语句或声明：`parentDimNum = *parentDim;`。
- **L306 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L306 CN**: 开始 `if` 控制流语句并计算其条件。
- **L307 EN**: Executes a standalone statement or declaration: `currentDimNum = *currentDim;`.
  **L307 CN**: 执行一条独立语句或声明：`currentDimNum = *currentDim;`。
- **L308 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L308 CN**: 开始 `if` 控制流语句并计算其条件。
- **L309 EN**: Executes a standalone statement or declaration: `std::string parentDimStr, currentDimStr;`.
  **L309 CN**: 执行一条独立语句或声明：`std::string parentDimStr, currentDimStr;`。
- **L310 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L310 CN**: 开始 `if` 控制流语句并计算其条件。
- **L311 EN**: Executes a call or declaration centered on `"`.
  **L311 CN**: 执行以 `"` 为核心的调用或声明。
- **L312 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L312 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 313-336

````cpp
                  currentDimStr = "(dim:" + std::to_string(currentDimNum) + ")";
                context_.Say(GetContext().clauseSource,
                    "%s%s clause is not allowed in the region of a loop with the %s%s clause"_err_en_US,
                    parser::ToUpperCaseLetters(
                        llvm::acc::getOpenACCClauseName(cl).str()),
                    currentDimStr,
                    parser::ToUpperCaseLetters(
                        llvm::acc::getOpenACCClauseName(parentClause).str()),
                    parentDimStr);
                continue;
              }
            }
          } else if (parentClause == llvm::acc::Clause::ACCC_worker &&
              (cl == llvm::acc::Clause::ACCC_gang ||
                  cl == llvm::acc::Clause::ACCC_worker)) {
            invalid = true;
          } else if (parentClause == llvm::acc::Clause::ACCC_vector &&
              (cl == llvm::acc::Clause::ACCC_gang ||
                  cl == llvm::acc::Clause::ACCC_worker ||
                  cl == llvm::acc::Clause::ACCC_vector)) {
            invalid = true;
          }
          if (invalid)
            context_.Say(GetContext().clauseSource,
````
- **L313 EN**: Executes a call or declaration centered on `"`.
  **L313 CN**: 执行以 `"` 为核心的调用或声明。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s%s clause is not allowed in the region of a loop with the %s%s clause"_err_en_US,`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s%s clause is not allowed in the region of a loop with the %s%s clause"_err_en_US,`。
- **L316 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L316 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::getOpenACCClauseName(cl).str()),`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::getOpenACCClauseName(cl).str()),`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `currentDimStr,`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`currentDimStr,`。
- **L319 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L319 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::getOpenACCClauseName(parentClause).str()),`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::getOpenACCClauseName(parentClause).str()),`。
- **L321 EN**: Executes a standalone statement or declaration: `parentDimStr);`.
  **L321 CN**: 执行一条独立语句或声明：`parentDimStr);`。
- **L322 EN**: Skips to the next loop iteration.
  **L322 CN**: 跳到下一次循环迭代。
- **L323 EN**: Closes the current lexical scope or compound statement.
  **L323 CN**: 结束当前词法作用域或复合语句块。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Transitions from the previous branch into an `else if` condition.
  **L325 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L326 EN**: Continues the surrounding expression or declaration: `(cl == llvm::acc::Clause::ACCC_gang ||`.
  **L326 CN**: 继续构造周围的表达式或声明：`(cl == llvm::acc::Clause::ACCC_gang ||`。
- **L327 EN**: Continues the surrounding expression or declaration: `cl == llvm::acc::Clause::ACCC_worker)) {`.
  **L327 CN**: 继续构造周围的表达式或声明：`cl == llvm::acc::Clause::ACCC_worker)) {`。
- **L328 EN**: Executes a standalone statement or declaration: `invalid = true;`.
  **L328 CN**: 执行一条独立语句或声明：`invalid = true;`。
- **L329 EN**: Transitions from the previous branch into an `else if` condition.
  **L329 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L330 EN**: Continues the surrounding expression or declaration: `(cl == llvm::acc::Clause::ACCC_gang ||`.
  **L330 CN**: 继续构造周围的表达式或声明：`(cl == llvm::acc::Clause::ACCC_gang ||`。
- **L331 EN**: Continues the surrounding expression or declaration: `cl == llvm::acc::Clause::ACCC_worker ||`.
  **L331 CN**: 继续构造周围的表达式或声明：`cl == llvm::acc::Clause::ACCC_worker ||`。
- **L332 EN**: Continues the surrounding expression or declaration: `cl == llvm::acc::Clause::ACCC_vector)) {`.
  **L332 CN**: 继续构造周围的表达式或声明：`cl == llvm::acc::Clause::ACCC_vector)) {`。
- **L333 EN**: Executes a standalone statement or declaration: `invalid = true;`.
  **L333 CN**: 执行一条独立语句或声明：`invalid = true;`。
- **L334 EN**: Closes the current lexical scope or compound statement.
  **L334 CN**: 结束当前词法作用域或复合语句块。
- **L335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。

### Lines 337-360

````cpp
                "%s clause is not allowed in the region of a loop with the %s clause"_err_en_US,
                parser::ToUpperCaseLetters(
                    llvm::acc::getOpenACCClauseName(cl).str()),
                parser::ToUpperCaseLetters(
                    llvm::acc::getOpenACCClauseName(parentClause).str()));
        }
      }
    }
    if (IsComputeConstruct(parent.directive))
      break;
  }
}

void AccStructureChecker::Enter(const parser::CallStmt &call) {
  if (dirContext_.empty() || !call.typedCall) {
    return;
  }
  const Symbol *sym{call.typedCall->proc().GetSymbol()};
  if (!sym) {
    return;
  }
  const Symbol &ult{sym->GetUltimate()};
  const auto *subp{ult.detailsIf<SubprogramDetails>()};
  if (!subp || subp->openACCRoutineInfos().empty()) {
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s clause is not allowed in the region of a loop with the %s clause"_err_en_US,`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s clause is not allowed in the region of a loop with the %s clause"_err_en_US,`。
- **L338 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L338 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::getOpenACCClauseName(cl).str()),`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::getOpenACCClauseName(cl).str()),`。
- **L340 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L340 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L341 EN**: Executes a call or declaration centered on `llvm::acc::getOpenACCClauseName`.
  **L341 CN**: 执行以 `llvm::acc::getOpenACCClauseName` 为核心的调用或声明。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Closes the current lexical scope or compound statement.
  **L344 CN**: 结束当前词法作用域或复合语句块。
- **L345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L346 EN**: Exits the nearest loop or switch statement.
  **L346 CN**: 退出最近的循环或 switch 语句。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::CallStmt &call) {`.
  **L350 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::CallStmt &call) {`。
- **L351 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L351 CN**: 开始 `if` 控制流语句并计算其条件。
- **L352 EN**: Returns from the current function with `void`.
  **L352 CN**: 以 `void` 从当前函数返回。
- **L353 EN**: Closes the current lexical scope or compound statement.
  **L353 CN**: 结束当前词法作用域或复合语句块。
- **L354 EN**: Executes a call or declaration centered on `*sym{call.typedCall->proc`.
  **L354 CN**: 执行以 `*sym{call.typedCall->proc` 为核心的调用或声明。
- **L355 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L355 CN**: 开始 `if` 控制流语句并计算其条件。
- **L356 EN**: Returns from the current function with `void`.
  **L356 CN**: 以 `void` 从当前函数返回。
- **L357 EN**: Closes the current lexical scope or compound statement.
  **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Executes a call or declaration centered on `&ult{sym->GetUltimate`.
  **L358 CN**: 执行以 `&ult{sym->GetUltimate` 为核心的调用或声明。
- **L359 EN**: Executes a call or declaration centered on `*subp{ult.detailsIf<SubprogramDetails>`.
  **L359 CN**: 执行以 `*subp{ult.detailsIf<SubprogramDetails>` 为核心的调用或声明。
- **L360 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L360 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 361-384

````cpp
    return;
  }
  std::string routineParDim;
  unsigned routineGangDim = 0;
  for (const OpenACCRoutineInfo &ri : subp->openACCRoutineInfos()) {
    if (ri.isGang()) {
      if (unsigned gangDim = ri.gangDim()) {
        routineGangDim = gangDim;
        routineParDim = "GANG(" + std::to_string(gangDim) + ")";
      } else {
        routineGangDim = 1;
        routineParDim = "GANG";
      }
    } else if (ri.isWorker()) {
      routineParDim = "WORKER";
    } else if (ri.isVector()) {
      routineParDim = "VECTOR";
    } else if (ri.isSeq()) {
      routineParDim = "SEQ";
    }
  }

  DirectiveContext &inner{dirContext_.back()};
  for (llvm::acc::Clause cl : inner.actualClauses) {
````
- **L361 EN**: Returns from the current function with `void`.
  **L361 CN**: 以 `void` 从当前函数返回。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Executes a standalone statement or declaration: `std::string routineParDim;`.
  **L363 CN**: 执行一条独立语句或声明：`std::string routineParDim;`。
- **L364 EN**: Initializes variable `routineGangDim` from the right-hand expression.
  **L364 CN**: 使用右侧表达式初始化变量 `routineGangDim`。
- **L365 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L365 CN**: 开始 `for` 控制流语句并计算其条件。
- **L366 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L366 CN**: 开始 `if` 控制流语句并计算其条件。
- **L367 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L367 CN**: 开始 `if` 控制流语句并计算其条件。
- **L368 EN**: Executes a standalone statement or declaration: `routineGangDim = gangDim;`.
  **L368 CN**: 执行一条独立语句或声明：`routineGangDim = gangDim;`。
- **L369 EN**: Executes a call or declaration centered on `"GANG`.
  **L369 CN**: 执行以 `"GANG` 为核心的调用或声明。
- **L370 EN**: Transitions from the previous branch into the alternative path.
  **L370 CN**: 从前一个分支过渡到备选路径。
- **L371 EN**: Executes a standalone statement or declaration: `routineGangDim = 1;`.
  **L371 CN**: 执行一条独立语句或声明：`routineGangDim = 1;`。
- **L372 EN**: Executes a standalone statement or declaration: `routineParDim = "GANG";`.
  **L372 CN**: 执行一条独立语句或声明：`routineParDim = "GANG";`。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Transitions from the previous branch into an `else if` condition.
  **L374 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L375 EN**: Executes a standalone statement or declaration: `routineParDim = "WORKER";`.
  **L375 CN**: 执行一条独立语句或声明：`routineParDim = "WORKER";`。
- **L376 EN**: Transitions from the previous branch into an `else if` condition.
  **L376 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L377 EN**: Executes a standalone statement or declaration: `routineParDim = "VECTOR";`.
  **L377 CN**: 执行一条独立语句或声明：`routineParDim = "VECTOR";`。
- **L378 EN**: Transitions from the previous branch into an `else if` condition.
  **L378 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L379 EN**: Executes a standalone statement or declaration: `routineParDim = "SEQ";`.
  **L379 CN**: 执行一条独立语句或声明：`routineParDim = "SEQ";`。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L383 EN**: Executes a call or declaration centered on `&inner{dirContext_.back`.
  **L383 CN**: 执行以 `&inner{dirContext_.back` 为核心的调用或声明。
- **L384 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L384 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 385-408

````cpp
    if (cl == llvm::acc::Clause::ACCC_vector) {
      if (!routineParDim.empty() && routineParDim != "SEQ") {
        context_.Say(GetContext().clauseSource,
            "Calling %s routine inside VECTOR loop is not allowed"_err_en_US,
            routineParDim);
      }
    }
    if (cl == llvm::acc::Clause::ACCC_worker) {
      if (!routineParDim.empty() &&
          (routineParDim != "SEQ" && routineParDim != "VECTOR")) {
        context_.Say(GetContext().clauseSource,
            "Calling %s routine inside WORKER loop is not allowed"_err_en_US,
            routineParDim);
      }
    }
    if (cl == llvm::acc::Clause::ACCC_gang) {
      const std::optional<std::int64_t> loopGangDim{
          getGangDimensionSize(inner)};
      const std::int64_t loopDimNum{loopGangDim.value_or(1)};
      if (routineGangDim && routineGangDim >= loopDimNum) {
        if (loopGangDim) {
          context_.Say(GetContext().clauseSource,
              "Calling %s routine inside GANG(%s) loop is not allowed"_err_en_US,
              routineParDim, std::to_string(*loopGangDim));
````
- **L385 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L385 CN**: 开始 `if` 控制流语句并计算其条件。
- **L386 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L386 CN**: 开始 `if` 控制流语句并计算其条件。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Calling %s routine inside VECTOR loop is not allowed"_err_en_US,`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Calling %s routine inside VECTOR loop is not allowed"_err_en_US,`。
- **L389 EN**: Executes a standalone statement or declaration: `routineParDim);`.
  **L389 CN**: 执行一条独立语句或声明：`routineParDim);`。
- **L390 EN**: Closes the current lexical scope or compound statement.
  **L390 CN**: 结束当前词法作用域或复合语句块。
- **L391 EN**: Closes the current lexical scope or compound statement.
  **L391 CN**: 结束当前词法作用域或复合语句块。
- **L392 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L392 CN**: 开始 `if` 控制流语句并计算其条件。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Starts a function, method, lambda, or structured scope: `(routineParDim != "SEQ" && routineParDim != "VECTOR")) {`.
  **L394 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(routineParDim != "SEQ" && routineParDim != "VECTOR")) {`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Calling %s routine inside WORKER loop is not allowed"_err_en_US,`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Calling %s routine inside WORKER loop is not allowed"_err_en_US,`。
- **L397 EN**: Executes a standalone statement or declaration: `routineParDim);`.
  **L397 CN**: 执行一条独立语句或声明：`routineParDim);`。
- **L398 EN**: Closes the current lexical scope or compound statement.
  **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L400 CN**: 开始 `if` 控制流语句并计算其条件。
- **L401 EN**: Continues the surrounding expression or declaration: `const std::optional<std::int64_t> loopGangDim{`.
  **L401 CN**: 继续构造周围的表达式或声明：`const std::optional<std::int64_t> loopGangDim{`。
- **L402 EN**: Executes a call or declaration centered on `getGangDimensionSize`.
  **L402 CN**: 执行以 `getGangDimensionSize` 为核心的调用或声明。
- **L403 EN**: Executes a call or declaration centered on `loopDimNum{loopGangDim.value_or`.
  **L403 CN**: 执行以 `loopDimNum{loopGangDim.value_or` 为核心的调用或声明。
- **L404 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L404 CN**: 开始 `if` 控制流语句并计算其条件。
- **L405 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L405 CN**: 开始 `if` 控制流语句并计算其条件。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Calling %s routine inside GANG(%s) loop is not allowed"_err_en_US,`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Calling %s routine inside GANG(%s) loop is not allowed"_err_en_US,`。
- **L408 EN**: Executes a call or declaration centered on `std::to_string`.
  **L408 CN**: 执行以 `std::to_string` 为核心的调用或声明。

### Lines 409-432

````cpp
        } else {
          context_.Say(GetContext().clauseSource,
              "Calling %s routine inside GANG loop is not allowed"_err_en_US,
              routineParDim);
        }
      }
    }
  }
}

void AccStructureChecker::Enter(const parser::OpenACCLoopConstruct &x) {
  const auto &beginDir{std::get<parser::AccBeginLoopDirective>(x.t)};
  const auto &loopDir{std::get<parser::AccLoopDirective>(beginDir.t)};
  PushContextAndClauseSets(loopDir.source, loopDir.v);
}

void AccStructureChecker::Leave(const parser::OpenACCLoopConstruct &x) {
  const auto &beginDir{std::get<parser::AccBeginLoopDirective>(x.t)};
  const auto &loopDir{std::get<parser::AccLoopDirective>(beginDir.t)};
  if (loopDir.v == llvm::acc::Directive::ACCD_loop) {
    // Restriction - line 1818-1819
    CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,
        loopOnlyAllowedAfterDeviceTypeClauses);
    // Restriction - line 1834
````
- **L409 EN**: Transitions from the previous branch into the alternative path.
  **L409 CN**: 从前一个分支过渡到备选路径。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Calling %s routine inside GANG loop is not allowed"_err_en_US,`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Calling %s routine inside GANG loop is not allowed"_err_en_US,`。
- **L412 EN**: Executes a standalone statement or declaration: `routineParDim);`.
  **L412 CN**: 执行一条独立语句或声明：`routineParDim);`。
- **L413 EN**: Closes the current lexical scope or compound statement.
  **L413 CN**: 结束当前词法作用域或复合语句块。
- **L414 EN**: Closes the current lexical scope or compound statement.
  **L414 CN**: 结束当前词法作用域或复合语句块。
- **L415 EN**: Closes the current lexical scope or compound statement.
  **L415 CN**: 结束当前词法作用域或复合语句块。
- **L416 EN**: Closes the current lexical scope or compound statement.
  **L416 CN**: 结束当前词法作用域或复合语句块。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L419 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCLoopConstruct &x) {`.
  **L419 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCLoopConstruct &x) {`。
- **L420 EN**: Executes a call or declaration centered on `&beginDir{std::get<parser::AccBeginLoopDirective>`.
  **L420 CN**: 执行以 `&beginDir{std::get<parser::AccBeginLoopDirective>` 为核心的调用或声明。
- **L421 EN**: Executes a call or declaration centered on `&loopDir{std::get<parser::AccLoopDirective>`.
  **L421 CN**: 执行以 `&loopDir{std::get<parser::AccLoopDirective>` 为核心的调用或声明。
- **L422 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L422 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L425 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCLoopConstruct &x) {`.
  **L425 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCLoopConstruct &x) {`。
- **L426 EN**: Executes a call or declaration centered on `&beginDir{std::get<parser::AccBeginLoopDirective>`.
  **L426 CN**: 执行以 `&beginDir{std::get<parser::AccBeginLoopDirective>` 为核心的调用或声明。
- **L427 EN**: Executes a call or declaration centered on `&loopDir{std::get<parser::AccLoopDirective>`.
  **L427 CN**: 执行以 `&loopDir{std::get<parser::AccLoopDirective>` 为核心的调用或声明。
- **L428 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L428 CN**: 开始 `if` 控制流语句并计算其条件。
- **L429 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1818-1819`.
  **L429 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1818-1819`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`。
- **L431 EN**: Executes a standalone statement or declaration: `loopOnlyAllowedAfterDeviceTypeClauses);`.
  **L431 CN**: 执行一条独立语句或声明：`loopOnlyAllowedAfterDeviceTypeClauses);`。
- **L432 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1834`.
  **L432 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1834`。

### Lines 433-456

````cpp
    CheckNotAllowedIfClause(llvm::acc::Clause::ACCC_seq,
        {llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_vector,
            llvm::acc::Clause::ACCC_worker});
    // Restriction - 2.9.2, 2.9.3, 2.9.4
    CheckNotInSameOrSubLevelLoopConstruct();
  }
  dirContext_.pop_back();
}

void AccStructureChecker::Enter(const parser::OpenACCStandaloneConstruct &x) {
  const auto &standaloneDir{std::get<parser::AccStandaloneDirective>(x.t)};
  PushContextAndClauseSets(standaloneDir.source, standaloneDir.v);
}

void AccStructureChecker::Leave(const parser::OpenACCStandaloneConstruct &x) {
  const auto &standaloneDir{std::get<parser::AccStandaloneDirective>(x.t)};
  switch (standaloneDir.v) {
  case llvm::acc::Directive::ACCD_enter_data:
  case llvm::acc::Directive::ACCD_exit_data:
    // Restriction - line 1310-1311 (ENTER DATA)
    // Restriction - line 1312-1313 (EXIT DATA)
    CheckRequireAtLeastOneOf();
    break;
  case llvm::acc::Directive::ACCD_set:
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckNotAllowedIfClause(llvm::acc::Clause::ACCC_seq,`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckNotAllowedIfClause(llvm::acc::Clause::ACCC_seq,`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_vector,`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{llvm::acc::Clause::ACCC_gang, llvm::acc::Clause::ACCC_vector,`。
- **L435 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_worker});`.
  **L435 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_worker});`。
- **L436 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - 2.9.2, 2.9.3, 2.9.4`.
  **L436 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - 2.9.2, 2.9.3, 2.9.4`。
- **L437 EN**: Executes a call or declaration centered on `CheckNotInSameOrSubLevelLoopConstruct`.
  **L437 CN**: 执行以 `CheckNotInSameOrSubLevelLoopConstruct` 为核心的调用或声明。
- **L438 EN**: Closes the current lexical scope or compound statement.
  **L438 CN**: 结束当前词法作用域或复合语句块。
- **L439 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L439 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L440 EN**: Closes the current lexical scope or compound statement.
  **L440 CN**: 结束当前词法作用域或复合语句块。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L442 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCStandaloneConstruct &x) {`.
  **L442 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCStandaloneConstruct &x) {`。
- **L443 EN**: Executes a call or declaration centered on `&standaloneDir{std::get<parser::AccStandaloneDirective>`.
  **L443 CN**: 执行以 `&standaloneDir{std::get<parser::AccStandaloneDirective>` 为核心的调用或声明。
- **L444 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L444 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L445 EN**: Closes the current lexical scope or compound statement.
  **L445 CN**: 结束当前词法作用域或复合语句块。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L447 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCStandaloneConstruct &x) {`.
  **L447 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCStandaloneConstruct &x) {`。
- **L448 EN**: Executes a call or declaration centered on `&standaloneDir{std::get<parser::AccStandaloneDirective>`.
  **L448 CN**: 执行以 `&standaloneDir{std::get<parser::AccStandaloneDirective>` 为核心的调用或声明。
- **L449 EN**: Begins a `switch` control-flow statement and evaluates its condition.
  **L449 CN**: 开始 `switch` 控制流语句并计算其条件。
- **L450 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_enter_data:`.
  **L450 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_enter_data:`。
- **L451 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_exit_data:`.
  **L451 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_exit_data:`。
- **L452 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1310-1311 (ENTER DATA)`.
  **L452 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1310-1311 (ENTER DATA)`。
- **L453 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 1312-1313 (EXIT DATA)`.
  **L453 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 1312-1313 (EXIT DATA)`。
- **L454 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L454 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L455 EN**: Exits the nearest loop or switch statement.
  **L455 CN**: 退出最近的循环或 switch 语句。
- **L456 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_set:`.
  **L456 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_set:`。

### Lines 457-480

````cpp
    // Restriction - line 2610
    CheckRequireAtLeastOneOf();
    // Restriction - line 2602
    CheckNotInComputeConstruct();
    break;
  case llvm::acc::Directive::ACCD_update:
    // Restriction - line 2636
    CheckRequireAtLeastOneOf();
    // Restriction - line 2669
    CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,
        updateOnlyAllowedAfterDeviceTypeClauses);
    break;
  case llvm::acc::Directive::ACCD_init:
  case llvm::acc::Directive::ACCD_shutdown:
    // Restriction - line 2525 (INIT)
    // Restriction - line 2561 (SHUTDOWN)
    CheckNotInComputeConstruct();
    break;
  default:
    break;
  }
  dirContext_.pop_back();
}

````
- **L457 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2610`.
  **L457 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2610`。
- **L458 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L458 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L459 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2602`.
  **L459 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2602`。
- **L460 EN**: Executes a call or declaration centered on `CheckNotInComputeConstruct`.
  **L460 CN**: 执行以 `CheckNotInComputeConstruct` 为核心的调用或声明。
- **L461 EN**: Exits the nearest loop or switch statement.
  **L461 CN**: 退出最近的循环或 switch 语句。
- **L462 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_update:`.
  **L462 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_update:`。
- **L463 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2636`.
  **L463 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2636`。
- **L464 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L464 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L465 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2669`.
  **L465 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2669`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`。
- **L467 EN**: Executes a standalone statement or declaration: `updateOnlyAllowedAfterDeviceTypeClauses);`.
  **L467 CN**: 执行一条独立语句或声明：`updateOnlyAllowedAfterDeviceTypeClauses);`。
- **L468 EN**: Exits the nearest loop or switch statement.
  **L468 CN**: 退出最近的循环或 switch 语句。
- **L469 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_init:`.
  **L469 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_init:`。
- **L470 EN**: Introduces a switch dispatch label: `case llvm::acc::Directive::ACCD_shutdown:`.
  **L470 CN**: 引入一个 switch 分发标签：`case llvm::acc::Directive::ACCD_shutdown:`。
- **L471 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2525 (INIT)`.
  **L471 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2525 (INIT)`。
- **L472 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2561 (SHUTDOWN)`.
  **L472 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2561 (SHUTDOWN)`。
- **L473 EN**: Executes a call or declaration centered on `CheckNotInComputeConstruct`.
  **L473 CN**: 执行以 `CheckNotInComputeConstruct` 为核心的调用或声明。
- **L474 EN**: Exits the nearest loop or switch statement.
  **L474 CN**: 退出最近的循环或 switch 语句。
- **L475 EN**: Introduces a switch dispatch label: `default:`.
  **L475 CN**: 引入一个 switch 分发标签：`default:`。
- **L476 EN**: Exits the nearest loop or switch statement.
  **L476 CN**: 退出最近的循环或 switch 语句。
- **L477 EN**: Closes the current lexical scope or compound statement.
  **L477 CN**: 结束当前词法作用域或复合语句块。
- **L478 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L478 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 481-504

````cpp
void AccStructureChecker::Enter(const parser::OpenACCRoutineConstruct &x) {
  PushContextAndClauseSets(x.source, llvm::acc::Directive::ACCD_routine);
  const auto &optName{std::get<std::optional<parser::Name>>(x.t)};
  if (!optName) {
    const auto &verbatim{std::get<parser::Verbatim>(x.t)};
    const auto &scope{context_.FindScope(verbatim.source)};
    const Scope &containingScope{GetProgramUnitContaining(scope)};
    if (containingScope.kind() == Scope::Kind::Module) {
      context_.Say(GetContext().directiveSource,
          "ROUTINE directive without name must appear within the specification "
          "part of a subroutine or function definition, or within an interface "
          "body for a subroutine or function in an interface block"_err_en_US);
    }
  }
}
void AccStructureChecker::Leave(const parser::OpenACCRoutineConstruct &) {
  // Restriction - line 2790
  CheckRequireAtLeastOneOf();
  // Restriction - line 2788-2789
  CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,
      routineOnlyAllowedAfterDeviceTypeClauses);
  dirContext_.pop_back();
}

````
- **L481 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCRoutineConstruct &x) {`.
  **L481 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCRoutineConstruct &x) {`。
- **L482 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L482 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L483 EN**: Executes a call or declaration centered on `&optName{std::get<std::optional<parser::Name>>`.
  **L483 CN**: 执行以 `&optName{std::get<std::optional<parser::Name>>` 为核心的调用或声明。
- **L484 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L484 CN**: 开始 `if` 控制流语句并计算其条件。
- **L485 EN**: Executes a call or declaration centered on `&verbatim{std::get<parser::Verbatim>`.
  **L485 CN**: 执行以 `&verbatim{std::get<parser::Verbatim>` 为核心的调用或声明。
- **L486 EN**: Executes a call or declaration centered on `&scope{context_.FindScope`.
  **L486 CN**: 执行以 `&scope{context_.FindScope` 为核心的调用或声明。
- **L487 EN**: Executes a call or declaration centered on `&containingScope{GetProgramUnitContaining`.
  **L487 CN**: 执行以 `&containingScope{GetProgramUnitContaining` 为核心的调用或声明。
- **L488 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L488 CN**: 开始 `if` 控制流语句并计算其条件。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().directiveSource,`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().directiveSource,`。
- **L490 EN**: Continues the surrounding expression or declaration: `"ROUTINE directive without name must appear within the specification "`.
  **L490 CN**: 继续构造周围的表达式或声明：`"ROUTINE directive without name must appear within the specification "`。
- **L491 EN**: Continues the surrounding expression or declaration: `"part of a subroutine or function definition, or within an interface "`.
  **L491 CN**: 继续构造周围的表达式或声明：`"part of a subroutine or function definition, or within an interface "`。
- **L492 EN**: Executes a standalone statement or declaration: `"body for a subroutine or function in an interface block"_err_en_US);`.
  **L492 CN**: 执行一条独立语句或声明：`"body for a subroutine or function in an interface block"_err_en_US);`。
- **L493 EN**: Closes the current lexical scope or compound statement.
  **L493 CN**: 结束当前词法作用域或复合语句块。
- **L494 EN**: Closes the current lexical scope or compound statement.
  **L494 CN**: 结束当前词法作用域或复合语句块。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCRoutineConstruct &) {`.
  **L496 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCRoutineConstruct &) {`。
- **L497 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2790`.
  **L497 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2790`。
- **L498 EN**: Executes a call or declaration centered on `CheckRequireAtLeastOneOf`.
  **L498 CN**: 执行以 `CheckRequireAtLeastOneOf` 为核心的调用或声明。
- **L499 EN**: Comment explains nearby logic, intent, or metadata: `Restriction - line 2788-2789`.
  **L499 CN**: 注释说明附近代码的逻辑、意图或元数据：`Restriction - line 2788-2789`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckOnlyAllowedAfter(llvm::acc::Clause::ACCC_device_type,`。
- **L501 EN**: Executes a standalone statement or declaration: `routineOnlyAllowedAfterDeviceTypeClauses);`.
  **L501 CN**: 执行一条独立语句或声明：`routineOnlyAllowedAfterDeviceTypeClauses);`。
- **L502 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L502 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-528

````cpp
void AccStructureChecker::Enter(const parser::OpenACCWaitConstruct &x) {
  const auto &verbatim{std::get<parser::Verbatim>(x.t)};
  PushContextAndClauseSets(verbatim.source, llvm::acc::Directive::ACCD_wait);
}
void AccStructureChecker::Leave(const parser::OpenACCWaitConstruct &x) {
  dirContext_.pop_back();
}

void AccStructureChecker::Enter(const parser::OpenACCAtomicConstruct &x) {
  PushContextAndClauseSets(x.source, llvm::acc::Directive::ACCD_atomic);
}
void AccStructureChecker::Leave(const parser::OpenACCAtomicConstruct &x) {
  dirContext_.pop_back();
}

void AccStructureChecker::CheckAtomicStmt(
    const parser::AssignmentStmt &assign, const std::string &construct) {
  const auto &var{std::get<parser::Variable>(assign.t)};
  const auto &expr{std::get<parser::Expr>(assign.t)};
  const auto *rhs{GetExpr(context_, expr)};
  const auto *lhs{GetExpr(context_, var)};

  if (lhs) {
    if (lhs->Rank() != 0) {
````
- **L505 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCWaitConstruct &x) {`.
  **L505 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCWaitConstruct &x) {`。
- **L506 EN**: Executes a call or declaration centered on `&verbatim{std::get<parser::Verbatim>`.
  **L506 CN**: 执行以 `&verbatim{std::get<parser::Verbatim>` 为核心的调用或声明。
- **L507 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L507 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L508 EN**: Closes the current lexical scope or compound statement.
  **L508 CN**: 结束当前词法作用域或复合语句块。
- **L509 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCWaitConstruct &x) {`.
  **L509 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCWaitConstruct &x) {`。
- **L510 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L510 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCAtomicConstruct &x) {`.
  **L513 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCAtomicConstruct &x) {`。
- **L514 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L514 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L515 EN**: Closes the current lexical scope or compound statement.
  **L515 CN**: 结束当前词法作用域或复合语句块。
- **L516 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCAtomicConstruct &x) {`.
  **L516 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCAtomicConstruct &x) {`。
- **L517 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L517 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L518 EN**: Closes the current lexical scope or compound statement.
  **L518 CN**: 结束当前词法作用域或复合语句块。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L520 EN**: Continues logic associated with callable symbol `CheckAtomicStmt`.
  **L520 CN**: 继续与可调用符号 `CheckAtomicStmt` 相关的逻辑。
- **L521 EN**: Continues the surrounding expression or declaration: `const parser::AssignmentStmt &assign, const std::string &construct) {`.
  **L521 CN**: 继续构造周围的表达式或声明：`const parser::AssignmentStmt &assign, const std::string &construct) {`。
- **L522 EN**: Executes a call or declaration centered on `&var{std::get<parser::Variable>`.
  **L522 CN**: 执行以 `&var{std::get<parser::Variable>` 为核心的调用或声明。
- **L523 EN**: Executes a call or declaration centered on `&expr{std::get<parser::Expr>`.
  **L523 CN**: 执行以 `&expr{std::get<parser::Expr>` 为核心的调用或声明。
- **L524 EN**: Executes a call or declaration centered on `*rhs{GetExpr`.
  **L524 CN**: 执行以 `*rhs{GetExpr` 为核心的调用或声明。
- **L525 EN**: Executes a call or declaration centered on `*lhs{GetExpr`.
  **L525 CN**: 执行以 `*lhs{GetExpr` 为核心的调用或声明。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L527 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L527 CN**: 开始 `if` 控制流语句并计算其条件。
- **L528 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L528 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 529-552

````cpp
      context_.Say(expr.source,
          "LHS of atomic %s statement must be scalar"_err_en_US, construct);
    }
    // TODO: Check if lhs is intrinsic type.
  }
  if (rhs) {
    if (rhs->Rank() != 0) {
      context_.Say(var.GetSource(),
          "RHS of atomic %s statement must be scalar"_err_en_US, construct);
    }
    // TODO: Check if rhs is intrinsic type.
  }
}

static constexpr evaluate::operation::OperatorSet validAccAtomicUpdateOperators{
    evaluate::operation::Operator::Add, evaluate::operation::Operator::Mul,
    evaluate::operation::Operator::Sub, evaluate::operation::Operator::Div,
    evaluate::operation::Operator::And, evaluate::operation::Operator::Or,
    evaluate::operation::Operator::Eqv, evaluate::operation::Operator::Neqv,
    evaluate::operation::Operator::Max, evaluate::operation::Operator::Min};

static bool IsValidAtomicUpdateOperation(
    const evaluate::operation::Operator &op) {
  return validAccAtomicUpdateOperators.test(op);
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L530 EN**: Executes a standalone statement or declaration: `"LHS of atomic %s statement must be scalar"_err_en_US, construct);`.
  **L530 CN**: 执行一条独立语句或声明：`"LHS of atomic %s statement must be scalar"_err_en_US, construct);`。
- **L531 EN**: Closes the current lexical scope or compound statement.
  **L531 CN**: 结束当前词法作用域或复合语句块。
- **L532 EN**: Comment records a pending task or caution: `TODO: Check if lhs is intrinsic type.`.
  **L532 CN**: 注释记录待办事项或注意点：`TODO: Check if lhs is intrinsic type.`。
- **L533 EN**: Closes the current lexical scope or compound statement.
  **L533 CN**: 结束当前词法作用域或复合语句块。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L535 CN**: 开始 `if` 控制流语句并计算其条件。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(var.GetSource(),`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(var.GetSource(),`。
- **L537 EN**: Executes a standalone statement or declaration: `"RHS of atomic %s statement must be scalar"_err_en_US, construct);`.
  **L537 CN**: 执行一条独立语句或声明：`"RHS of atomic %s statement must be scalar"_err_en_US, construct);`。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Comment records a pending task or caution: `TODO: Check if rhs is intrinsic type.`.
  **L539 CN**: 注释记录待办事项或注意点：`TODO: Check if rhs is intrinsic type.`。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L543 EN**: Continues the surrounding expression or declaration: `static constexpr evaluate::operation::OperatorSet validAccAtomicUpdateOperators{`.
  **L543 CN**: 继续构造周围的表达式或声明：`static constexpr evaluate::operation::OperatorSet validAccAtomicUpdateOperators{`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::operation::Operator::Add, evaluate::operation::Operator::Mul,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::operation::Operator::Add, evaluate::operation::Operator::Mul,`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::operation::Operator::Sub, evaluate::operation::Operator::Div,`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::operation::Operator::Sub, evaluate::operation::Operator::Div,`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::operation::Operator::And, evaluate::operation::Operator::Or,`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::operation::Operator::And, evaluate::operation::Operator::Or,`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `evaluate::operation::Operator::Eqv, evaluate::operation::Operator::Neqv,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`evaluate::operation::Operator::Eqv, evaluate::operation::Operator::Neqv,`。
- **L548 EN**: Executes a standalone statement or declaration: `evaluate::operation::Operator::Max, evaluate::operation::Operator::Min};`.
  **L548 CN**: 执行一条独立语句或声明：`evaluate::operation::Operator::Max, evaluate::operation::Operator::Min};`。
- **L549 EN**: Blank line separating nearby declarations or logic blocks.
  **L549 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L550 EN**: Continues logic associated with callable symbol `IsValidAtomicUpdateOperation`.
  **L550 CN**: 继续与可调用符号 `IsValidAtomicUpdateOperation` 相关的逻辑。
- **L551 EN**: Continues the surrounding expression or declaration: `const evaluate::operation::Operator &op) {`.
  **L551 CN**: 继续构造周围的表达式或声明：`const evaluate::operation::Operator &op) {`。
- **L552 EN**: Returns from the current function with `validAccAtomicUpdateOperators.test(op)`.
  **L552 CN**: 以 `validAccAtomicUpdateOperators.test(op)` 从当前函数返回。

### Lines 553-576

````cpp
}

// Couldn't reproduce this behavior with evaluate::UnwrapConvertedExpr which
// is similar but only works within a single type category.
static SomeExpr GetExprModuloConversion(const SomeExpr &expr) {
  const auto [op, args]{evaluate::GetTopLevelOperation(expr)};
  // Check: if it is a conversion then it must have at least one argument.
  CHECK(((op != evaluate::operation::Operator::Convert &&
             op != evaluate::operation::Operator::Resize) ||
            args.size() >= 1) &&
      "Invalid conversion operation");
  if ((op == evaluate::operation::Operator::Convert ||
          op == evaluate::operation::Operator::Resize) &&
      args.size() >= 1) {
    return args[0];
  }
  return expr;
}

void AccStructureChecker::CheckAtomicUpdateStmt(
    const parser::AssignmentStmt &assign, const SomeExpr &updateVar,
    const SomeExpr *captureVar) {
  CheckAtomicStmt(assign, "update");
  const auto &expr{std::get<parser::Expr>(assign.t)};
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L555 EN**: Comment explains nearby logic, intent, or metadata: `Couldn't reproduce this behavior with evaluate::UnwrapConvertedExpr which`.
  **L555 CN**: 注释说明附近代码的逻辑、意图或元数据：`Couldn't reproduce this behavior with evaluate::UnwrapConvertedExpr which`。
- **L556 EN**: Comment explains nearby logic, intent, or metadata: `is similar but only works within a single type category.`.
  **L556 CN**: 注释说明附近代码的逻辑、意图或元数据：`is similar but only works within a single type category.`。
- **L557 EN**: Starts a function, method, lambda, or structured scope: `static SomeExpr GetExprModuloConversion(const SomeExpr &expr) {`.
  **L557 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static SomeExpr GetExprModuloConversion(const SomeExpr &expr) {`。
- **L558 EN**: Executes a call or declaration centered on `args]{evaluate::GetTopLevelOperation`.
  **L558 CN**: 执行以 `args]{evaluate::GetTopLevelOperation` 为核心的调用或声明。
- **L559 EN**: Comment explains nearby logic, intent, or metadata: `Check: if it is a conversion then it must have at least one argument.`.
  **L559 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check: if it is a conversion then it must have at least one argument.`。
- **L560 EN**: Continues logic associated with callable symbol `CHECK`.
  **L560 CN**: 继续与可调用符号 `CHECK` 相关的逻辑。
- **L561 EN**: Continues the surrounding expression or declaration: `op != evaluate::operation::Operator::Resize) ||`.
  **L561 CN**: 继续构造周围的表达式或声明：`op != evaluate::operation::Operator::Resize) ||`。
- **L562 EN**: Continues logic associated with callable symbol `size`.
  **L562 CN**: 继续与可调用符号 `size` 相关的逻辑。
- **L563 EN**: Executes a standalone statement or declaration: `"Invalid conversion operation");`.
  **L563 CN**: 执行一条独立语句或声明：`"Invalid conversion operation");`。
- **L564 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L564 CN**: 开始 `if` 控制流语句并计算其条件。
- **L565 EN**: Continues the surrounding expression or declaration: `op == evaluate::operation::Operator::Resize) &&`.
  **L565 CN**: 继续构造周围的表达式或声明：`op == evaluate::operation::Operator::Resize) &&`。
- **L566 EN**: Starts a function, method, lambda, or structured scope: `args.size() >= 1) {`.
  **L566 CN**: 开始一个函数、方法、lambda 或结构化作用域：`args.size() >= 1) {`。
- **L567 EN**: Returns from the current function with `args[0]`.
  **L567 CN**: 以 `args[0]` 从当前函数返回。
- **L568 EN**: Closes the current lexical scope or compound statement.
  **L568 CN**: 结束当前词法作用域或复合语句块。
- **L569 EN**: Returns from the current function with `expr`.
  **L569 CN**: 以 `expr` 从当前函数返回。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L572 EN**: Continues logic associated with callable symbol `CheckAtomicUpdateStmt`.
  **L572 CN**: 继续与可调用符号 `CheckAtomicUpdateStmt` 相关的逻辑。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const parser::AssignmentStmt &assign, const SomeExpr &updateVar,`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`const parser::AssignmentStmt &assign, const SomeExpr &updateVar,`。
- **L574 EN**: Continues the surrounding expression or declaration: `const SomeExpr *captureVar) {`.
  **L574 CN**: 继续构造周围的表达式或声明：`const SomeExpr *captureVar) {`。
- **L575 EN**: Executes a call or declaration centered on `CheckAtomicStmt`.
  **L575 CN**: 执行以 `CheckAtomicStmt` 为核心的调用或声明。
- **L576 EN**: Executes a call or declaration centered on `&expr{std::get<parser::Expr>`.
  **L576 CN**: 执行以 `&expr{std::get<parser::Expr>` 为核心的调用或声明。

### Lines 577-600

````cpp
  const auto *rhs{GetExpr(context_, expr)};
  if (rhs) {
    const auto [op, args]{
        evaluate::GetTopLevelOperation(GetExprModuloConversion(*rhs))};
    if (!IsValidAtomicUpdateOperation(op)) {
      context_.Say(expr.source,
          "Invalid atomic update operation, can only use: *, +, -, *, /, and, or, eqv, neqv, max, min, iand, ior, ieor"_err_en_US);
    } else {
      bool foundUpdateVar{false};
      for (const auto &arg : args) {
        if (updateVar == GetExprModuloConversion(arg)) {
          if (foundUpdateVar) {
            context_.Say(expr.source,
                "The updated variable, %s, cannot appear more than once in the atomic update operation"_err_en_US,
                updateVar.AsFortran());
          } else {
            foundUpdateVar = true;
          }
        } else if (evaluate::IsVarSubexpressionOf(updateVar, arg)) {
          // TODO: Get the source location of arg and point to the individual
          // argument.
          context_.Say(expr.source,
              "Arguments to the atomic update operation cannot reference the updated variable, %s, as a subexpression"_err_en_US,
              updateVar.AsFortran());
````
- **L577 EN**: Executes a call or declaration centered on `*rhs{GetExpr`.
  **L577 CN**: 执行以 `*rhs{GetExpr` 为核心的调用或声明。
- **L578 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L578 CN**: 开始 `if` 控制流语句并计算其条件。
- **L579 EN**: Continues the surrounding expression or declaration: `const auto [op, args]{`.
  **L579 CN**: 继续构造周围的表达式或声明：`const auto [op, args]{`。
- **L580 EN**: Executes a call or declaration centered on `evaluate::GetTopLevelOperation`.
  **L580 CN**: 执行以 `evaluate::GetTopLevelOperation` 为核心的调用或声明。
- **L581 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L581 CN**: 开始 `if` 控制流语句并计算其条件。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L583 EN**: Executes a standalone statement or declaration: `"Invalid atomic update operation, can only use: *, +, -, *, /, and, or, eqv, neqv, max, min, iand, ior, ieor"_err_en_US);`.
  **L583 CN**: 执行一条独立语句或声明：`"Invalid atomic update operation, can only use: *, +, -, *, /, and, or, eqv, neqv, max, min, iand, ior, ieor"_err_en_US);`。
- **L584 EN**: Transitions from the previous branch into the alternative path.
  **L584 CN**: 从前一个分支过渡到备选路径。
- **L585 EN**: Executes a standalone statement or declaration: `bool foundUpdateVar{false};`.
  **L585 CN**: 执行一条独立语句或声明：`bool foundUpdateVar{false};`。
- **L586 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L586 CN**: 开始 `for` 控制流语句并计算其条件。
- **L587 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L587 CN**: 开始 `if` 控制流语句并计算其条件。
- **L588 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L588 CN**: 开始 `if` 控制流语句并计算其条件。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The updated variable, %s, cannot appear more than once in the atomic update operation"_err_en_US,`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The updated variable, %s, cannot appear more than once in the atomic update operation"_err_en_US,`。
- **L591 EN**: Executes a call or declaration centered on `updateVar.AsFortran`.
  **L591 CN**: 执行以 `updateVar.AsFortran` 为核心的调用或声明。
- **L592 EN**: Transitions from the previous branch into the alternative path.
  **L592 CN**: 从前一个分支过渡到备选路径。
- **L593 EN**: Executes a standalone statement or declaration: `foundUpdateVar = true;`.
  **L593 CN**: 执行一条独立语句或声明：`foundUpdateVar = true;`。
- **L594 EN**: Closes the current lexical scope or compound statement.
  **L594 CN**: 结束当前词法作用域或复合语句块。
- **L595 EN**: Transitions from the previous branch into an `else if` condition.
  **L595 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L596 EN**: Comment records a pending task or caution: `TODO: Get the source location of arg and point to the individual`.
  **L596 CN**: 注释记录待办事项或注意点：`TODO: Get the source location of arg and point to the individual`。
- **L597 EN**: Comment explains nearby logic, intent, or metadata: `argument.`.
  **L597 CN**: 注释说明附近代码的逻辑、意图或元数据：`argument.`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Arguments to the atomic update operation cannot reference the updated variable, %s, as a subexpression"_err_en_US,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Arguments to the atomic update operation cannot reference the updated variable, %s, as a subexpression"_err_en_US,`。
- **L600 EN**: Executes a call or declaration centered on `updateVar.AsFortran`.
  **L600 CN**: 执行以 `updateVar.AsFortran` 为核心的调用或声明。

### Lines 601-624

````cpp
        }
      }
      if (!foundUpdateVar) {
        context_.Say(expr.source,
            "The RHS of this atomic update statement must reference the updated variable: %s"_err_en_US,
            updateVar.AsFortran());
      }
    }
  }
}

void AccStructureChecker::CheckAtomicWriteStmt(
    const parser::AssignmentStmt &assign, const SomeExpr &updateVar,
    const SomeExpr *captureVar) {
  CheckAtomicStmt(assign, "write");
  const auto &expr{std::get<parser::Expr>(assign.t)};
  const auto *rhs{GetExpr(context_, expr)};
  if (rhs) {
    if (evaluate::IsVarSubexpressionOf(updateVar, *rhs)) {
      context_.Say(expr.source,
          "The RHS of this atomic write statement cannot reference the atomic variable: %s"_err_en_US,
          updateVar.AsFortran());
    }
  }
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L603 CN**: 开始 `if` 控制流语句并计算其条件。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The RHS of this atomic update statement must reference the updated variable: %s"_err_en_US,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The RHS of this atomic update statement must reference the updated variable: %s"_err_en_US,`。
- **L606 EN**: Executes a call or declaration centered on `updateVar.AsFortran`.
  **L606 CN**: 执行以 `updateVar.AsFortran` 为核心的调用或声明。
- **L607 EN**: Closes the current lexical scope or compound statement.
  **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Closes the current lexical scope or compound statement.
  **L609 CN**: 结束当前词法作用域或复合语句块。
- **L610 EN**: Closes the current lexical scope or compound statement.
  **L610 CN**: 结束当前词法作用域或复合语句块。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L612 EN**: Continues logic associated with callable symbol `CheckAtomicWriteStmt`.
  **L612 CN**: 继续与可调用符号 `CheckAtomicWriteStmt` 相关的逻辑。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const parser::AssignmentStmt &assign, const SomeExpr &updateVar,`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`const parser::AssignmentStmt &assign, const SomeExpr &updateVar,`。
- **L614 EN**: Continues the surrounding expression or declaration: `const SomeExpr *captureVar) {`.
  **L614 CN**: 继续构造周围的表达式或声明：`const SomeExpr *captureVar) {`。
- **L615 EN**: Executes a call or declaration centered on `CheckAtomicStmt`.
  **L615 CN**: 执行以 `CheckAtomicStmt` 为核心的调用或声明。
- **L616 EN**: Executes a call or declaration centered on `&expr{std::get<parser::Expr>`.
  **L616 CN**: 执行以 `&expr{std::get<parser::Expr>` 为核心的调用或声明。
- **L617 EN**: Executes a call or declaration centered on `*rhs{GetExpr`.
  **L617 CN**: 执行以 `*rhs{GetExpr` 为核心的调用或声明。
- **L618 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L618 CN**: 开始 `if` 控制流语句并计算其条件。
- **L619 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L619 CN**: 开始 `if` 控制流语句并计算其条件。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(expr.source,`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(expr.source,`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The RHS of this atomic write statement cannot reference the atomic variable: %s"_err_en_US,`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The RHS of this atomic write statement cannot reference the atomic variable: %s"_err_en_US,`。
- **L622 EN**: Executes a call or declaration centered on `updateVar.AsFortran`.
  **L622 CN**: 执行以 `updateVar.AsFortran` 为核心的调用或声明。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。

### Lines 625-648

````cpp
}

void AccStructureChecker::CheckAtomicCaptureStmt(
    const parser::AssignmentStmt &assign, const SomeExpr *updateVar,
    const SomeExpr &captureVar) {
  CheckAtomicStmt(assign, "capture");
}

void AccStructureChecker::Enter(const parser::AccAtomicCapture &capture) {
  const Fortran::parser::AssignmentStmt &stmt1{
      std::get<Fortran::parser::AccAtomicCapture::Stmt1>(capture.t)
          .v.statement};
  const Fortran::parser::AssignmentStmt &stmt2{
      std::get<Fortran::parser::AccAtomicCapture::Stmt2>(capture.t)
          .v.statement};
  const auto &var1{std::get<parser::Variable>(stmt1.t)};
  const auto &var2{std::get<parser::Variable>(stmt2.t)};
  const auto *lhs1{GetExpr(context_, var1)};
  const auto *lhs2{GetExpr(context_, var2)};
  if (!lhs1 || !lhs2) {
    // Not enough information to check.
    return;
  }
  if (*lhs1 == *lhs2) {
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L627 EN**: Continues logic associated with callable symbol `CheckAtomicCaptureStmt`.
  **L627 CN**: 继续与可调用符号 `CheckAtomicCaptureStmt` 相关的逻辑。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const parser::AssignmentStmt &assign, const SomeExpr *updateVar,`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`const parser::AssignmentStmt &assign, const SomeExpr *updateVar,`。
- **L629 EN**: Continues the surrounding expression or declaration: `const SomeExpr &captureVar) {`.
  **L629 CN**: 继续构造周围的表达式或声明：`const SomeExpr &captureVar) {`。
- **L630 EN**: Executes a call or declaration centered on `CheckAtomicStmt`.
  **L630 CN**: 执行以 `CheckAtomicStmt` 为核心的调用或声明。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L633 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccAtomicCapture &capture) {`.
  **L633 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccAtomicCapture &capture) {`。
- **L634 EN**: Continues the surrounding expression or declaration: `const Fortran::parser::AssignmentStmt &stmt1{`.
  **L634 CN**: 继续构造周围的表达式或声明：`const Fortran::parser::AssignmentStmt &stmt1{`。
- **L635 EN**: Continues logic associated with callable symbol `Stmt1>`.
  **L635 CN**: 继续与可调用符号 `Stmt1>` 相关的逻辑。
- **L636 EN**: Executes a standalone statement or declaration: `.v.statement};`.
  **L636 CN**: 执行一条独立语句或声明：`.v.statement};`。
- **L637 EN**: Continues the surrounding expression or declaration: `const Fortran::parser::AssignmentStmt &stmt2{`.
  **L637 CN**: 继续构造周围的表达式或声明：`const Fortran::parser::AssignmentStmt &stmt2{`。
- **L638 EN**: Continues logic associated with callable symbol `Stmt2>`.
  **L638 CN**: 继续与可调用符号 `Stmt2>` 相关的逻辑。
- **L639 EN**: Executes a standalone statement or declaration: `.v.statement};`.
  **L639 CN**: 执行一条独立语句或声明：`.v.statement};`。
- **L640 EN**: Executes a call or declaration centered on `&var1{std::get<parser::Variable>`.
  **L640 CN**: 执行以 `&var1{std::get<parser::Variable>` 为核心的调用或声明。
- **L641 EN**: Executes a call or declaration centered on `&var2{std::get<parser::Variable>`.
  **L641 CN**: 执行以 `&var2{std::get<parser::Variable>` 为核心的调用或声明。
- **L642 EN**: Executes a call or declaration centered on `*lhs1{GetExpr`.
  **L642 CN**: 执行以 `*lhs1{GetExpr` 为核心的调用或声明。
- **L643 EN**: Executes a call or declaration centered on `*lhs2{GetExpr`.
  **L643 CN**: 执行以 `*lhs2{GetExpr` 为核心的调用或声明。
- **L644 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L644 CN**: 开始 `if` 控制流语句并计算其条件。
- **L645 EN**: Comment explains nearby logic, intent, or metadata: `Not enough information to check.`.
  **L645 CN**: 注释说明附近代码的逻辑、意图或元数据：`Not enough information to check.`。
- **L646 EN**: Returns from the current function with `void`.
  **L646 CN**: 以 `void` 从当前函数返回。
- **L647 EN**: Closes the current lexical scope or compound statement.
  **L647 CN**: 结束当前词法作用域或复合语句块。
- **L648 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L648 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 649-672

````cpp
    context_.Say(std::get<parser::Verbatim>(capture.t).source,
        "The variables assigned in this atomic capture construct must be distinct"_err_en_US);
    return;
  }
  const auto &expr1{std::get<parser::Expr>(stmt1.t)};
  const auto &expr2{std::get<parser::Expr>(stmt2.t)};
  const auto *rhs1{GetExpr(context_, expr1)};
  const auto *rhs2{GetExpr(context_, expr2)};
  if (!rhs1 || !rhs2) {
    return;
  }
  bool stmt1CapturesLhs2{*lhs2 == GetExprModuloConversion(*rhs1)};
  bool stmt2CapturesLhs1{*lhs1 == GetExprModuloConversion(*rhs2)};
  if (stmt1CapturesLhs2 && !stmt2CapturesLhs1) {
    if (*lhs2 == GetExprModuloConversion(*rhs2)) {
      // a = b; b = b: Doesn't fit the spec.
      context_.Say(std::get<parser::Verbatim>(capture.t).source,
          "The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);
      // TODO: Add attatchment that a = b seems to be a capture,
      // but b = b is not a valid update or write.
    } else if (evaluate::IsVarSubexpressionOf(*lhs2, *rhs2)) {
      // Take v = x; x = <expr w/ x> as capture; update
      const auto &updateVar{*lhs2};
      const auto &captureVar{*lhs1};
````
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(std::get<parser::Verbatim>(capture.t).source,`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(std::get<parser::Verbatim>(capture.t).source,`。
- **L650 EN**: Executes a standalone statement or declaration: `"The variables assigned in this atomic capture construct must be distinct"_err_en_US);`.
  **L650 CN**: 执行一条独立语句或声明：`"The variables assigned in this atomic capture construct must be distinct"_err_en_US);`。
- **L651 EN**: Returns from the current function with `void`.
  **L651 CN**: 以 `void` 从当前函数返回。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Executes a call or declaration centered on `&expr1{std::get<parser::Expr>`.
  **L653 CN**: 执行以 `&expr1{std::get<parser::Expr>` 为核心的调用或声明。
- **L654 EN**: Executes a call or declaration centered on `&expr2{std::get<parser::Expr>`.
  **L654 CN**: 执行以 `&expr2{std::get<parser::Expr>` 为核心的调用或声明。
- **L655 EN**: Executes a call or declaration centered on `*rhs1{GetExpr`.
  **L655 CN**: 执行以 `*rhs1{GetExpr` 为核心的调用或声明。
- **L656 EN**: Executes a call or declaration centered on `*rhs2{GetExpr`.
  **L656 CN**: 执行以 `*rhs2{GetExpr` 为核心的调用或声明。
- **L657 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L657 CN**: 开始 `if` 控制流语句并计算其条件。
- **L658 EN**: Returns from the current function with `void`.
  **L658 CN**: 以 `void` 从当前函数返回。
- **L659 EN**: Closes the current lexical scope or compound statement.
  **L659 CN**: 结束当前词法作用域或复合语句块。
- **L660 EN**: Executes a call or declaration centered on `GetExprModuloConversion`.
  **L660 CN**: 执行以 `GetExprModuloConversion` 为核心的调用或声明。
- **L661 EN**: Executes a call or declaration centered on `GetExprModuloConversion`.
  **L661 CN**: 执行以 `GetExprModuloConversion` 为核心的调用或声明。
- **L662 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L662 CN**: 开始 `if` 控制流语句并计算其条件。
- **L663 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L663 CN**: 开始 `if` 控制流语句并计算其条件。
- **L664 EN**: Comment explains nearby logic, intent, or metadata: `a = b; b = b: Doesn't fit the spec.`.
  **L664 CN**: 注释说明附近代码的逻辑、意图或元数据：`a = b; b = b: Doesn't fit the spec.`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(std::get<parser::Verbatim>(capture.t).source,`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(std::get<parser::Verbatim>(capture.t).source,`。
- **L666 EN**: Executes a standalone statement or declaration: `"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`.
  **L666 CN**: 执行一条独立语句或声明：`"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`。
- **L667 EN**: Comment records a pending task or caution: `TODO: Add attatchment that a = b seems to be a capture,`.
  **L667 CN**: 注释记录待办事项或注意点：`TODO: Add attatchment that a = b seems to be a capture,`。
- **L668 EN**: Comment explains nearby logic, intent, or metadata: `but b = b is not a valid update or write.`.
  **L668 CN**: 注释说明附近代码的逻辑、意图或元数据：`but b = b is not a valid update or write.`。
- **L669 EN**: Transitions from the previous branch into an `else if` condition.
  **L669 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L670 EN**: Comment explains nearby logic, intent, or metadata: `Take v = x; x = <expr w/ x> as capture; update`.
  **L670 CN**: 注释说明附近代码的逻辑、意图或元数据：`Take v = x; x = <expr w/ x> as capture; update`。
- **L671 EN**: Executes a standalone statement or declaration: `const auto &updateVar{*lhs2};`.
  **L671 CN**: 执行一条独立语句或声明：`const auto &updateVar{*lhs2};`。
- **L672 EN**: Executes a standalone statement or declaration: `const auto &captureVar{*lhs1};`.
  **L672 CN**: 执行一条独立语句或声明：`const auto &captureVar{*lhs1};`。

### Lines 673-696

````cpp
      CheckAtomicCaptureStmt(stmt1, &updateVar, captureVar);
      CheckAtomicUpdateStmt(stmt2, updateVar, &captureVar);
    } else {
      // Take v = x; x = <expr w/o x> as capture; write
      const auto &updateVar{*lhs2};
      const auto &captureVar{*lhs1};
      CheckAtomicCaptureStmt(stmt1, &updateVar, captureVar);
      CheckAtomicWriteStmt(stmt2, updateVar, &captureVar);
    }
  } else if (stmt2CapturesLhs1 && !stmt1CapturesLhs2) {
    if (*lhs1 == GetExprModuloConversion(*rhs1)) {
      // Error a = a; b = a;
      context_.Say(var1.GetSource(),
          "The first assignment in this atomic capture construct doesn't perform a valid update"_err_en_US);
      // Add attatchment that a = a is not considered an update,
      // but b = a seems to be a capture.
    } else {
      // Take x = <expr>; v = x: as update; capture
      const auto &updateVar{*lhs1};
      const auto &captureVar{*lhs2};
      CheckAtomicUpdateStmt(stmt1, updateVar, &captureVar);
      CheckAtomicCaptureStmt(stmt2, &updateVar, captureVar);
    }
  } else if (stmt1CapturesLhs2 && stmt2CapturesLhs1) {
````
- **L673 EN**: Executes a call or declaration centered on `CheckAtomicCaptureStmt`.
  **L673 CN**: 执行以 `CheckAtomicCaptureStmt` 为核心的调用或声明。
- **L674 EN**: Executes a call or declaration centered on `CheckAtomicUpdateStmt`.
  **L674 CN**: 执行以 `CheckAtomicUpdateStmt` 为核心的调用或声明。
- **L675 EN**: Transitions from the previous branch into the alternative path.
  **L675 CN**: 从前一个分支过渡到备选路径。
- **L676 EN**: Comment explains nearby logic, intent, or metadata: `Take v = x; x = <expr w/o x> as capture; write`.
  **L676 CN**: 注释说明附近代码的逻辑、意图或元数据：`Take v = x; x = <expr w/o x> as capture; write`。
- **L677 EN**: Executes a standalone statement or declaration: `const auto &updateVar{*lhs2};`.
  **L677 CN**: 执行一条独立语句或声明：`const auto &updateVar{*lhs2};`。
- **L678 EN**: Executes a standalone statement or declaration: `const auto &captureVar{*lhs1};`.
  **L678 CN**: 执行一条独立语句或声明：`const auto &captureVar{*lhs1};`。
- **L679 EN**: Executes a call or declaration centered on `CheckAtomicCaptureStmt`.
  **L679 CN**: 执行以 `CheckAtomicCaptureStmt` 为核心的调用或声明。
- **L680 EN**: Executes a call or declaration centered on `CheckAtomicWriteStmt`.
  **L680 CN**: 执行以 `CheckAtomicWriteStmt` 为核心的调用或声明。
- **L681 EN**: Closes the current lexical scope or compound statement.
  **L681 CN**: 结束当前词法作用域或复合语句块。
- **L682 EN**: Transitions from the previous branch into an `else if` condition.
  **L682 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L683 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L683 CN**: 开始 `if` 控制流语句并计算其条件。
- **L684 EN**: Comment explains nearby logic, intent, or metadata: `Error a = a; b = a;`.
  **L684 CN**: 注释说明附近代码的逻辑、意图或元数据：`Error a = a; b = a;`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(var1.GetSource(),`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(var1.GetSource(),`。
- **L686 EN**: Executes a standalone statement or declaration: `"The first assignment in this atomic capture construct doesn't perform a valid update"_err_en_US);`.
  **L686 CN**: 执行一条独立语句或声明：`"The first assignment in this atomic capture construct doesn't perform a valid update"_err_en_US);`。
- **L687 EN**: Comment explains nearby logic, intent, or metadata: `Add attatchment that a = a is not considered an update,`.
  **L687 CN**: 注释说明附近代码的逻辑、意图或元数据：`Add attatchment that a = a is not considered an update,`。
- **L688 EN**: Comment explains nearby logic, intent, or metadata: `but b = a seems to be a capture.`.
  **L688 CN**: 注释说明附近代码的逻辑、意图或元数据：`but b = a seems to be a capture.`。
- **L689 EN**: Transitions from the previous branch into the alternative path.
  **L689 CN**: 从前一个分支过渡到备选路径。
- **L690 EN**: Comment explains nearby logic, intent, or metadata: `Take x = <expr>; v = x: as update; capture`.
  **L690 CN**: 注释说明附近代码的逻辑、意图或元数据：`Take x = <expr>; v = x: as update; capture`。
- **L691 EN**: Executes a standalone statement or declaration: `const auto &updateVar{*lhs1};`.
  **L691 CN**: 执行一条独立语句或声明：`const auto &updateVar{*lhs1};`。
- **L692 EN**: Executes a standalone statement or declaration: `const auto &captureVar{*lhs2};`.
  **L692 CN**: 执行一条独立语句或声明：`const auto &captureVar{*lhs2};`。
- **L693 EN**: Executes a call or declaration centered on `CheckAtomicUpdateStmt`.
  **L693 CN**: 执行以 `CheckAtomicUpdateStmt` 为核心的调用或声明。
- **L694 EN**: Executes a call or declaration centered on `CheckAtomicCaptureStmt`.
  **L694 CN**: 执行以 `CheckAtomicCaptureStmt` 为核心的调用或声明。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Transitions from the previous branch into an `else if` condition.
  **L696 CN**: 从前一个分支过渡到 `else if` 条件判断。

### Lines 697-720

````cpp
    // x1 = x2; x2 = x1; Doesn't fit the spec.
    context_.Say(std::get<parser::Verbatim>(capture.t).source,
        "The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);
    // TODO: Add attatchment that both assignments seem to be captures.
  } else { // !stmt1CapturesLhs2 && !stmt2CapturesLhs1
    // a = <expr != b>; b = <expr != a>; Doesn't fit the spec
    context_.Say(std::get<parser::Verbatim>(capture.t).source,
        "The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);
    // TODO: Add attatchment that neither assignment seems to be a capture.
  }
}

void AccStructureChecker::Enter(const parser::AccAtomicUpdate &x) {
  const auto &assign{
      std::get<parser::Statement<parser::AssignmentStmt>>(x.t).statement};
  const auto &var{std::get<parser::Variable>(assign.t)};
  if (const auto *updateVar{GetExpr(context_, var)}) {
    CheckAtomicUpdateStmt(assign, *updateVar, /*captureVar=*/nullptr);
  }
}

void AccStructureChecker::Enter(const parser::AccAtomicWrite &x) {
  const auto &assign{
      std::get<parser::Statement<parser::AssignmentStmt>>(x.t).statement};
````
- **L697 EN**: Comment explains nearby logic, intent, or metadata: `x1 = x2; x2 = x1; Doesn't fit the spec.`.
  **L697 CN**: 注释说明附近代码的逻辑、意图或元数据：`x1 = x2; x2 = x1; Doesn't fit the spec.`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(std::get<parser::Verbatim>(capture.t).source,`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(std::get<parser::Verbatim>(capture.t).source,`。
- **L699 EN**: Executes a standalone statement or declaration: `"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`.
  **L699 CN**: 执行一条独立语句或声明：`"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`。
- **L700 EN**: Comment records a pending task or caution: `TODO: Add attatchment that both assignments seem to be captures.`.
  **L700 CN**: 注释记录待办事项或注意点：`TODO: Add attatchment that both assignments seem to be captures.`。
- **L701 EN**: Transitions from the previous branch into the alternative path.
  **L701 CN**: 从前一个分支过渡到备选路径。
- **L702 EN**: Comment explains nearby logic, intent, or metadata: `a = <expr != b>; b = <expr != a>; Doesn't fit the spec`.
  **L702 CN**: 注释说明附近代码的逻辑、意图或元数据：`a = <expr != b>; b = <expr != a>; Doesn't fit the spec`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(std::get<parser::Verbatim>(capture.t).source,`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(std::get<parser::Verbatim>(capture.t).source,`。
- **L704 EN**: Executes a standalone statement or declaration: `"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`.
  **L704 CN**: 执行一条独立语句或声明：`"The assignments in this atomic capture construct do not update a variable and capture either its initial or final value"_err_en_US);`。
- **L705 EN**: Comment records a pending task or caution: `TODO: Add attatchment that neither assignment seems to be a capture.`.
  **L705 CN**: 注释记录待办事项或注意点：`TODO: Add attatchment that neither assignment seems to be a capture.`。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L709 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccAtomicUpdate &x) {`.
  **L709 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccAtomicUpdate &x) {`。
- **L710 EN**: Continues the surrounding expression or declaration: `const auto &assign{`.
  **L710 CN**: 继续构造周围的表达式或声明：`const auto &assign{`。
- **L711 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::AssignmentStmt>>`.
  **L711 CN**: 执行以 `std::get<parser::Statement<parser::AssignmentStmt>>` 为核心的调用或声明。
- **L712 EN**: Executes a call or declaration centered on `&var{std::get<parser::Variable>`.
  **L712 CN**: 执行以 `&var{std::get<parser::Variable>` 为核心的调用或声明。
- **L713 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L713 CN**: 开始 `if` 控制流语句并计算其条件。
- **L714 EN**: Executes a call or declaration centered on `CheckAtomicUpdateStmt`.
  **L714 CN**: 执行以 `CheckAtomicUpdateStmt` 为核心的调用或声明。
- **L715 EN**: Closes the current lexical scope or compound statement.
  **L715 CN**: 结束当前词法作用域或复合语句块。
- **L716 EN**: Closes the current lexical scope or compound statement.
  **L716 CN**: 结束当前词法作用域或复合语句块。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L718 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccAtomicWrite &x) {`.
  **L718 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccAtomicWrite &x) {`。
- **L719 EN**: Continues the surrounding expression or declaration: `const auto &assign{`.
  **L719 CN**: 继续构造周围的表达式或声明：`const auto &assign{`。
- **L720 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::AssignmentStmt>>`.
  **L720 CN**: 执行以 `std::get<parser::Statement<parser::AssignmentStmt>>` 为核心的调用或声明。

### Lines 721-744

````cpp
  const auto &var{std::get<parser::Variable>(assign.t)};
  if (const auto *updateVar{GetExpr(context_, var)}) {
    CheckAtomicWriteStmt(assign, *updateVar, /*captureVar=*/nullptr);
  }
}

void AccStructureChecker::Enter(const parser::AccAtomicRead &x) {
  const auto &assign{
      std::get<parser::Statement<parser::AssignmentStmt>>(x.t).statement};
  const auto &var{std::get<parser::Variable>(assign.t)};
  if (const auto *captureVar{GetExpr(context_, var)}) {
    CheckAtomicCaptureStmt(assign, /*updateVar=*/nullptr, *captureVar);
  }
}

void AccStructureChecker::Enter(const parser::OpenACCCacheConstruct &x) {
  const auto &verbatim = std::get<parser::Verbatim>(x.t);
  PushContextAndClauseSets(verbatim.source, llvm::acc::Directive::ACCD_cache);
  SetContextDirectiveSource(verbatim.source);
  // Check cache directive array section constraints
  const auto &objectListWithModifier =
      std::get<parser::AccObjectListWithModifier>(x.t);
  const auto &objectList =
      std::get<parser::AccObjectList>(objectListWithModifier.t);
````
- **L721 EN**: Executes a call or declaration centered on `&var{std::get<parser::Variable>`.
  **L721 CN**: 执行以 `&var{std::get<parser::Variable>` 为核心的调用或声明。
- **L722 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L722 CN**: 开始 `if` 控制流语句并计算其条件。
- **L723 EN**: Executes a call or declaration centered on `CheckAtomicWriteStmt`.
  **L723 CN**: 执行以 `CheckAtomicWriteStmt` 为核心的调用或声明。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Closes the current lexical scope or compound statement.
  **L725 CN**: 结束当前词法作用域或复合语句块。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L727 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccAtomicRead &x) {`.
  **L727 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccAtomicRead &x) {`。
- **L728 EN**: Continues the surrounding expression or declaration: `const auto &assign{`.
  **L728 CN**: 继续构造周围的表达式或声明：`const auto &assign{`。
- **L729 EN**: Executes a call or declaration centered on `std::get<parser::Statement<parser::AssignmentStmt>>`.
  **L729 CN**: 执行以 `std::get<parser::Statement<parser::AssignmentStmt>>` 为核心的调用或声明。
- **L730 EN**: Executes a call or declaration centered on `&var{std::get<parser::Variable>`.
  **L730 CN**: 执行以 `&var{std::get<parser::Variable>` 为核心的调用或声明。
- **L731 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L731 CN**: 开始 `if` 控制流语句并计算其条件。
- **L732 EN**: Executes a call or declaration centered on `CheckAtomicCaptureStmt`.
  **L732 CN**: 执行以 `CheckAtomicCaptureStmt` 为核心的调用或声明。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L736 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCCacheConstruct &x) {`.
  **L736 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCCacheConstruct &x) {`。
- **L737 EN**: Executes a call or declaration centered on `std::get<parser::Verbatim>`.
  **L737 CN**: 执行以 `std::get<parser::Verbatim>` 为核心的调用或声明。
- **L738 EN**: Executes a call or declaration centered on `PushContextAndClauseSets`.
  **L738 CN**: 执行以 `PushContextAndClauseSets` 为核心的调用或声明。
- **L739 EN**: Executes a call or declaration centered on `SetContextDirectiveSource`.
  **L739 CN**: 执行以 `SetContextDirectiveSource` 为核心的调用或声明。
- **L740 EN**: Comment explains nearby logic, intent, or metadata: `Check cache directive array section constraints`.
  **L740 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check cache directive array section constraints`。
- **L741 EN**: Continues the surrounding expression or declaration: `const auto &objectListWithModifier =`.
  **L741 CN**: 继续构造周围的表达式或声明：`const auto &objectListWithModifier =`。
- **L742 EN**: Executes a call or declaration centered on `std::get<parser::AccObjectListWithModifier>`.
  **L742 CN**: 执行以 `std::get<parser::AccObjectListWithModifier>` 为核心的调用或声明。
- **L743 EN**: Continues the surrounding expression or declaration: `const auto &objectList =`.
  **L743 CN**: 继续构造周围的表达式或声明：`const auto &objectList =`。
- **L744 EN**: Executes a call or declaration centered on `std::get<parser::AccObjectList>`.
  **L744 CN**: 执行以 `std::get<parser::AccObjectList>` 为核心的调用或声明。

### Lines 745-768

````cpp

  for (const auto &accObject : objectList.v) {
    common::visit(
        common::visitors{
            [&](const parser::Designator &designator) {
              if (const auto *dataRef =
                      std::get_if<parser::DataRef>(&designator.u)) {
                if (const auto *arrayElem =
                        std::get_if<common::Indirection<parser::ArrayElement>>(
                            &dataRef->u)) {
                  for (const auto &subscript :
                      arrayElem->value().Subscripts()) {
                    if (const auto *triplet =
                            std::get_if<parser::SubscriptTriplet>(
                                &subscript.u)) {
                      const auto &stride{std::get<2>(triplet->t)};
                      if (stride) {
                        if (auto strideVal{GetIntValue(*stride)}) {
                          if (*strideVal != 1) {
                            context_.Say(designator.source,
                                "The CACHE directive does not support strided array sections"_err_en_US);
                          }
                        }
                      }
````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L746 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L746 CN**: 开始 `for` 控制流语句并计算其条件。
- **L747 EN**: Continues logic associated with callable symbol `visit`.
  **L747 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L748 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L748 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L749 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Designator &designator) {`.
  **L749 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Designator &designator) {`。
- **L750 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L750 CN**: 开始 `if` 控制流语句并计算其条件。
- **L751 EN**: Starts a function, method, lambda, or structured scope: `std::get_if<parser::DataRef>(&designator.u)) {`.
  **L751 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get_if<parser::DataRef>(&designator.u)) {`。
- **L752 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L752 CN**: 开始 `if` 控制流语句并计算其条件。
- **L753 EN**: Continues logic associated with callable symbol `ArrayElement>>`.
  **L753 CN**: 继续与可调用符号 `ArrayElement>>` 相关的逻辑。
- **L754 EN**: Continues the surrounding expression or declaration: `&dataRef->u)) {`.
  **L754 CN**: 继续构造周围的表达式或声明：`&dataRef->u)) {`。
- **L755 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L755 CN**: 开始 `for` 控制流语句并计算其条件。
- **L756 EN**: Starts a function, method, lambda, or structured scope: `arrayElem->value().Subscripts()) {`.
  **L756 CN**: 开始一个函数、方法、lambda 或结构化作用域：`arrayElem->value().Subscripts()) {`。
- **L757 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L757 CN**: 开始 `if` 控制流语句并计算其条件。
- **L758 EN**: Continues logic associated with callable symbol `SubscriptTriplet>`.
  **L758 CN**: 继续与可调用符号 `SubscriptTriplet>` 相关的逻辑。
- **L759 EN**: Continues the surrounding expression or declaration: `&subscript.u)) {`.
  **L759 CN**: 继续构造周围的表达式或声明：`&subscript.u)) {`。
- **L760 EN**: Executes a call or declaration centered on `&stride{std::get<2>`.
  **L760 CN**: 执行以 `&stride{std::get<2>` 为核心的调用或声明。
- **L761 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L761 CN**: 开始 `if` 控制流语句并计算其条件。
- **L762 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L762 CN**: 开始 `if` 控制流语句并计算其条件。
- **L763 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L763 CN**: 开始 `if` 控制流语句并计算其条件。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(designator.source,`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(designator.source,`。
- **L765 EN**: Executes a standalone statement or declaration: `"The CACHE directive does not support strided array sections"_err_en_US);`.
  **L765 CN**: 执行一条独立语句或声明：`"The CACHE directive does not support strided array sections"_err_en_US);`。
- **L766 EN**: Closes the current lexical scope or compound statement.
  **L766 CN**: 结束当前词法作用域或复合语句块。
- **L767 EN**: Closes the current lexical scope or compound statement.
  **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-792

````cpp
                    }
                  }
                }
              }
            },
            [&](const parser::Name &) {
              // Common block names are not expected in cache directive
            }},
        accObject.u);
  }
}
void AccStructureChecker::Leave(const parser::OpenACCCacheConstruct &x) {
  dirContext_.pop_back();
}

// Clause checkers
CHECK_SIMPLE_CLAUSE(Auto, ACCC_auto)
CHECK_SIMPLE_CLAUSE(Attach, ACCC_attach)
CHECK_SIMPLE_CLAUSE(Bind, ACCC_bind)
CHECK_SIMPLE_CLAUSE(Capture, ACCC_capture)
CHECK_SIMPLE_CLAUSE(Default, ACCC_default)
CHECK_SIMPLE_CLAUSE(DefaultAsync, ACCC_default_async)
CHECK_SIMPLE_CLAUSE(Delete, ACCC_delete)
CHECK_SIMPLE_CLAUSE(Detach, ACCC_detach)
````
- **L769 EN**: Closes the current lexical scope or compound statement.
  **L769 CN**: 结束当前词法作用域或复合语句块。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Closes the current lexical scope or compound statement.
  **L772 CN**: 结束当前词法作用域或复合语句块。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L774 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Name &) {`.
  **L774 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Name &) {`。
- **L775 EN**: Comment explains nearby logic, intent, or metadata: `Common block names are not expected in cache directive`.
  **L775 CN**: 注释说明附近代码的逻辑、意图或元数据：`Common block names are not expected in cache directive`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `}},`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`}},`。
- **L777 EN**: Executes a standalone statement or declaration: `accObject.u);`.
  **L777 CN**: 执行一条独立语句或声明：`accObject.u);`。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Closes the current lexical scope or compound statement.
  **L779 CN**: 结束当前词法作用域或复合语句块。
- **L780 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::OpenACCCacheConstruct &x) {`.
  **L780 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::OpenACCCacheConstruct &x) {`。
- **L781 EN**: Executes a call or declaration centered on `dirContext_.pop_back`.
  **L781 CN**: 执行以 `dirContext_.pop_back` 为核心的调用或声明。
- **L782 EN**: Closes the current lexical scope or compound statement.
  **L782 CN**: 结束当前词法作用域或复合语句块。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L784 EN**: Comment explains nearby logic, intent, or metadata: `Clause checkers`.
  **L784 CN**: 注释说明附近代码的逻辑、意图或元数据：`Clause checkers`。
- **L785 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L785 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L786 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L786 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L787 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L787 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L788 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L788 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L789 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L789 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L790 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L790 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L791 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L791 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L792 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L792 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。

### Lines 793-816

````cpp
CHECK_SIMPLE_CLAUSE(Device, ACCC_device)
CHECK_SIMPLE_CLAUSE(DeviceNum, ACCC_device_num)
CHECK_SIMPLE_CLAUSE(Finalize, ACCC_finalize)
CHECK_SIMPLE_CLAUSE(Firstprivate, ACCC_firstprivate)
CHECK_SIMPLE_CLAUSE(Host, ACCC_host)
CHECK_SIMPLE_CLAUSE(IfPresent, ACCC_if_present)
CHECK_SIMPLE_CLAUSE(Independent, ACCC_independent)
CHECK_SIMPLE_CLAUSE(NoCreate, ACCC_no_create)
CHECK_SIMPLE_CLAUSE(Nohost, ACCC_nohost)
CHECK_SIMPLE_CLAUSE(Private, ACCC_private)
CHECK_SIMPLE_CLAUSE(Read, ACCC_read)
CHECK_SIMPLE_CLAUSE(UseDevice, ACCC_use_device)
CHECK_SIMPLE_CLAUSE(Wait, ACCC_wait)
CHECK_SIMPLE_CLAUSE(Write, ACCC_write)
CHECK_SIMPLE_CLAUSE(Unknown, ACCC_unknown)

void AccStructureChecker::CheckMultipleOccurrenceInDeclare(
    const parser::AccObjectList &list, llvm::acc::Clause clause) {
  if (GetContext().directive != llvm::acc::Directive::ACCD_declare)
    return;
  for (const auto &object : list.v) {
    common::visit(
        common::visitors{
            [&](const parser::Designator &designator) {
````
- **L793 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L793 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L794 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L794 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L795 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L795 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L796 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L796 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L797 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L797 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L798 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L798 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L799 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L799 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L800 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L800 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L801 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L801 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L802 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L802 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L803 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L803 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L804 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L804 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L805 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L805 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L806 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L806 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L807 EN**: Continues logic associated with callable symbol `CHECK_SIMPLE_CLAUSE`.
  **L807 CN**: 继续与可调用符号 `CHECK_SIMPLE_CLAUSE` 相关的逻辑。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L809 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L809 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L810 EN**: Continues the surrounding expression or declaration: `const parser::AccObjectList &list, llvm::acc::Clause clause) {`.
  **L810 CN**: 继续构造周围的表达式或声明：`const parser::AccObjectList &list, llvm::acc::Clause clause) {`。
- **L811 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L811 CN**: 开始 `if` 控制流语句并计算其条件。
- **L812 EN**: Returns from the current function with `void`.
  **L812 CN**: 以 `void` 从当前函数返回。
- **L813 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L813 CN**: 开始 `for` 控制流语句并计算其条件。
- **L814 EN**: Continues logic associated with callable symbol `visit`.
  **L814 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L815 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L815 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L816 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Designator &designator) {`.
  **L816 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Designator &designator) {`。

### Lines 817-840

````cpp
              if (const auto *name =
                      parser::GetDesignatorNameIfDataRef(designator)) {
                if (declareSymbols.contains(&name->symbol->GetUltimate())) {
                  if (declareSymbols[&name->symbol->GetUltimate()] == clause) {
                    context_.Warn(common::UsageWarning::OpenAccUsage,
                        GetContext().clauseSource,
                        "'%s' in the %s clause is already present in the same clause in this module"_warn_en_US,
                        name->symbol->name(),
                        parser::ToUpperCaseLetters(
                            llvm::acc::getOpenACCClauseName(clause).str()));
                  } else {
                    context_.Say(GetContext().clauseSource,
                        "'%s' in the %s clause is already present in another "
                        "%s clause in this module"_err_en_US,
                        name->symbol->name(),
                        parser::ToUpperCaseLetters(
                            llvm::acc::getOpenACCClauseName(clause).str()),
                        parser::ToUpperCaseLetters(
                            llvm::acc::getOpenACCClauseName(
                                declareSymbols[&name->symbol->GetUltimate()])
                                .str()));
                  }
                }
                declareSymbols.insert({&name->symbol->GetUltimate(), clause});
````
- **L817 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L817 CN**: 开始 `if` 控制流语句并计算其条件。
- **L818 EN**: Starts a function, method, lambda, or structured scope: `parser::GetDesignatorNameIfDataRef(designator)) {`.
  **L818 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::GetDesignatorNameIfDataRef(designator)) {`。
- **L819 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L819 CN**: 开始 `if` 控制流语句并计算其条件。
- **L820 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L820 CN**: 开始 `if` 控制流语句并计算其条件。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::OpenAccUsage,`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::OpenAccUsage,`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `GetContext().clauseSource,`.
  **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`GetContext().clauseSource,`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%s' in the %s clause is already present in the same clause in this module"_warn_en_US,`.
  **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%s' in the %s clause is already present in the same clause in this module"_warn_en_US,`。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `name->symbol->name(),`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`name->symbol->name(),`。
- **L825 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L825 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L826 EN**: Executes a call or declaration centered on `llvm::acc::getOpenACCClauseName`.
  **L826 CN**: 执行以 `llvm::acc::getOpenACCClauseName` 为核心的调用或声明。
- **L827 EN**: Transitions from the previous branch into the alternative path.
  **L827 CN**: 从前一个分支过渡到备选路径。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L829 EN**: Continues the surrounding expression or declaration: `"'%s' in the %s clause is already present in another "`.
  **L829 CN**: 继续构造周围的表达式或声明：`"'%s' in the %s clause is already present in another "`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%s clause in this module"_err_en_US,`.
  **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%s clause in this module"_err_en_US,`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `name->symbol->name(),`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`name->symbol->name(),`。
- **L832 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L832 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::acc::getOpenACCClauseName(clause).str()),`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::acc::getOpenACCClauseName(clause).str()),`。
- **L834 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L834 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L835 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L835 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L836 EN**: Continues logic associated with callable symbol `GetUltimate`.
  **L836 CN**: 继续与可调用符号 `GetUltimate` 相关的逻辑。
- **L837 EN**: Executes a call or declaration centered on `.str`.
  **L837 CN**: 执行以 `.str` 为核心的调用或声明。
- **L838 EN**: Closes the current lexical scope or compound statement.
  **L838 CN**: 结束当前词法作用域或复合语句块。
- **L839 EN**: Closes the current lexical scope or compound statement.
  **L839 CN**: 结束当前词法作用域或复合语句块。
- **L840 EN**: Executes a call or declaration centered on `declareSymbols.insert`.
  **L840 CN**: 执行以 `declareSymbols.insert` 为核心的调用或声明。

### Lines 841-864

````cpp
              }
            },
            [&](const parser::Name &name) {
              // TODO: check common block
            }},
        object.u);
  }
}

void AccStructureChecker::CheckMultipleOccurrenceInDeclare(
    const parser::AccObjectListWithModifier &list, llvm::acc::Clause clause) {
  const auto &objectList = std::get<Fortran::parser::AccObjectList>(list.t);
  CheckMultipleOccurrenceInDeclare(objectList, clause);
}

void AccStructureChecker::Enter(const parser::AccClause::Async &c) {
  llvm::acc::Clause crtClause = llvm::acc::Clause::ACCC_async;
  CheckAllowed(crtClause);
  CheckAllowedOncePerGroup(crtClause, llvm::acc::Clause::ACCC_device_type);
}

void AccStructureChecker::Enter(const parser::AccClause::Create &c) {
  CheckAllowed(llvm::acc::Clause::ACCC_create);
  const auto &modifierClause{c.v};
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L843 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Name &name) {`.
  **L843 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Name &name) {`。
- **L844 EN**: Comment records a pending task or caution: `TODO: check common block`.
  **L844 CN**: 注释记录待办事项或注意点：`TODO: check common block`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `}},`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`}},`。
- **L846 EN**: Executes a standalone statement or declaration: `object.u);`.
  **L846 CN**: 执行一条独立语句或声明：`object.u);`。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Closes the current lexical scope or compound statement.
  **L848 CN**: 结束当前词法作用域或复合语句块。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L850 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L850 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L851 EN**: Continues the surrounding expression or declaration: `const parser::AccObjectListWithModifier &list, llvm::acc::Clause clause) {`.
  **L851 CN**: 继续构造周围的表达式或声明：`const parser::AccObjectListWithModifier &list, llvm::acc::Clause clause) {`。
- **L852 EN**: Executes a call or declaration centered on `std::get<Fortran::parser::AccObjectList>`.
  **L852 CN**: 执行以 `std::get<Fortran::parser::AccObjectList>` 为核心的调用或声明。
- **L853 EN**: Executes a call or declaration centered on `CheckMultipleOccurrenceInDeclare`.
  **L853 CN**: 执行以 `CheckMultipleOccurrenceInDeclare` 为核心的调用或声明。
- **L854 EN**: Closes the current lexical scope or compound statement.
  **L854 CN**: 结束当前词法作用域或复合语句块。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L856 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Async &c) {`.
  **L856 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Async &c) {`。
- **L857 EN**: Initializes variable `crtClause` from the right-hand expression.
  **L857 CN**: 使用右侧表达式初始化变量 `crtClause`。
- **L858 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L858 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L859 EN**: Executes a call or declaration centered on `CheckAllowedOncePerGroup`.
  **L859 CN**: 执行以 `CheckAllowedOncePerGroup` 为核心的调用或声明。
- **L860 EN**: Closes the current lexical scope or compound statement.
  **L860 CN**: 结束当前词法作用域或复合语句块。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L862 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Create &c) {`.
  **L862 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Create &c) {`。
- **L863 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L863 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L864 EN**: Executes a standalone statement or declaration: `const auto &modifierClause{c.v};`.
  **L864 CN**: 执行一条独立语句或声明：`const auto &modifierClause{c.v};`。

### Lines 865-888

````cpp
  if (const auto &modifier{
          std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {
    if (modifier->v != parser::AccDataModifier::Modifier::Zero) {
      context_.Say(GetContext().clauseSource,
          "Only the ZERO modifier is allowed for the %s clause "
          "on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_create)
                  .str()),
          ContextDirectiveAsFortran());
    }
    if (GetContext().directive == llvm::acc::Directive::ACCD_declare) {
      context_.Say(GetContext().clauseSource,
          "The ZERO modifier is not allowed for the %s clause "
          "on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_create)
                  .str()),
          ContextDirectiveAsFortran());
    }
  }
  CheckMultipleOccurrenceInDeclare(
      modifierClause, llvm::acc::Clause::ACCC_create);
}
````
- **L865 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L865 CN**: 开始 `if` 控制流语句并计算其条件。
- **L866 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`.
  **L866 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`。
- **L867 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L867 CN**: 开始 `if` 控制流语句并计算其条件。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L869 EN**: Continues the surrounding expression or declaration: `"Only the ZERO modifier is allowed for the %s clause "`.
  **L869 CN**: 继续构造周围的表达式或声明：`"Only the ZERO modifier is allowed for the %s clause "`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L871 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L871 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L872 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L872 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L874 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L874 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L875 EN**: Closes the current lexical scope or compound statement.
  **L875 CN**: 结束当前词法作用域或复合语句块。
- **L876 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L876 CN**: 开始 `if` 控制流语句并计算其条件。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L878 EN**: Continues the surrounding expression or declaration: `"The ZERO modifier is not allowed for the %s clause "`.
  **L878 CN**: 继续构造周围的表达式或声明：`"The ZERO modifier is not allowed for the %s clause "`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L880 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L880 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L881 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L881 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L883 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L883 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L884 EN**: Closes the current lexical scope or compound statement.
  **L884 CN**: 结束当前词法作用域或复合语句块。
- **L885 EN**: Closes the current lexical scope or compound statement.
  **L885 CN**: 结束当前词法作用域或复合语句块。
- **L886 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L886 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L887 EN**: Executes a standalone statement or declaration: `modifierClause, llvm::acc::Clause::ACCC_create);`.
  **L887 CN**: 执行一条独立语句或声明：`modifierClause, llvm::acc::Clause::ACCC_create);`。
- **L888 EN**: Closes the current lexical scope or compound statement.
  **L888 CN**: 结束当前词法作用域或复合语句块。

### Lines 889-912

````cpp

void AccStructureChecker::Enter(const parser::AccClause::Copyin &c) {
  CheckAllowed(llvm::acc::Clause::ACCC_copyin);
  const auto &modifierClause{c.v};
  if (const auto &modifier{
          std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {
    if (CheckAllowedModifier(llvm::acc::Clause::ACCC_copyin)) {
      return;
    }
    if (modifier->v != parser::AccDataModifier::Modifier::ReadOnly) {
      context_.Say(GetContext().clauseSource,
          "Only the READONLY modifier is allowed for the %s clause "
          "on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_copyin)
                  .str()),
          ContextDirectiveAsFortran());
    }
  }
  CheckMultipleOccurrenceInDeclare(
      modifierClause, llvm::acc::Clause::ACCC_copyin);
}

void AccStructureChecker::Enter(const parser::AccClause::Copyout &c) {
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L890 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Copyin &c) {`.
  **L890 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Copyin &c) {`。
- **L891 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L891 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L892 EN**: Executes a standalone statement or declaration: `const auto &modifierClause{c.v};`.
  **L892 CN**: 执行一条独立语句或声明：`const auto &modifierClause{c.v};`。
- **L893 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L893 CN**: 开始 `if` 控制流语句并计算其条件。
- **L894 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`.
  **L894 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`。
- **L895 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L895 CN**: 开始 `if` 控制流语句并计算其条件。
- **L896 EN**: Returns from the current function with `void`.
  **L896 CN**: 以 `void` 从当前函数返回。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L898 CN**: 开始 `if` 控制流语句并计算其条件。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L900 EN**: Continues the surrounding expression or declaration: `"Only the READONLY modifier is allowed for the %s clause "`.
  **L900 CN**: 继续构造周围的表达式或声明：`"Only the READONLY modifier is allowed for the %s clause "`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L902 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L902 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L903 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L903 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L905 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L905 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L906 EN**: Closes the current lexical scope or compound statement.
  **L906 CN**: 结束当前词法作用域或复合语句块。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L908 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L909 EN**: Executes a standalone statement or declaration: `modifierClause, llvm::acc::Clause::ACCC_copyin);`.
  **L909 CN**: 执行一条独立语句或声明：`modifierClause, llvm::acc::Clause::ACCC_copyin);`。
- **L910 EN**: Closes the current lexical scope or compound statement.
  **L910 CN**: 结束当前词法作用域或复合语句块。
- **L911 EN**: Blank line separating nearby declarations or logic blocks.
  **L911 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L912 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Copyout &c) {`.
  **L912 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Copyout &c) {`。

### Lines 913-936

````cpp
  CheckAllowed(llvm::acc::Clause::ACCC_copyout);
  const auto &modifierClause{c.v};
  if (const auto &modifier{
          std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {
    if (CheckAllowedModifier(llvm::acc::Clause::ACCC_copyout)) {
      return;
    }
    if (modifier->v != parser::AccDataModifier::Modifier::Zero) {
      context_.Say(GetContext().clauseSource,
          "Only the ZERO modifier is allowed for the %s clause "
          "on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_copyout)
                  .str()),
          ContextDirectiveAsFortran());
    }
    if (GetContext().directive == llvm::acc::Directive::ACCD_declare) {
      context_.Say(GetContext().clauseSource,
          "The ZERO modifier is not allowed for the %s clause "
          "on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_copyout)
                  .str()),
          ContextDirectiveAsFortran());
````
- **L913 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L913 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L914 EN**: Executes a standalone statement or declaration: `const auto &modifierClause{c.v};`.
  **L914 CN**: 执行一条独立语句或声明：`const auto &modifierClause{c.v};`。
- **L915 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L915 CN**: 开始 `if` 控制流语句并计算其条件。
- **L916 EN**: Starts a function, method, lambda, or structured scope: `std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`.
  **L916 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::get<std::optional<parser::AccDataModifier>>(modifierClause.t)}) {`。
- **L917 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L917 CN**: 开始 `if` 控制流语句并计算其条件。
- **L918 EN**: Returns from the current function with `void`.
  **L918 CN**: 以 `void` 从当前函数返回。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L920 CN**: 开始 `if` 控制流语句并计算其条件。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L922 EN**: Continues the surrounding expression or declaration: `"Only the ZERO modifier is allowed for the %s clause "`.
  **L922 CN**: 继续构造周围的表达式或声明：`"Only the ZERO modifier is allowed for the %s clause "`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L924 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L924 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L925 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L925 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L927 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L927 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。
- **L929 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L929 CN**: 开始 `if` 控制流语句并计算其条件。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L931 EN**: Continues the surrounding expression or declaration: `"The ZERO modifier is not allowed for the %s clause "`.
  **L931 CN**: 继续构造周围的表达式或声明：`"The ZERO modifier is not allowed for the %s clause "`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"on the %s directive"_err_en_US,`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`"on the %s directive"_err_en_US,`。
- **L933 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L933 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L934 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L934 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L936 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L936 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。

### Lines 937-960

````cpp
    }
  }
  CheckMultipleOccurrenceInDeclare(
      modifierClause, llvm::acc::Clause::ACCC_copyout);
}

void AccStructureChecker::Enter(const parser::AccClause::DeviceType &d) {
  CheckAllowed(llvm::acc::Clause::ACCC_device_type);
  if (GetContext().directive == llvm::acc::Directive::ACCD_set &&
      d.v.v.size() > 1) {
    context_.Say(GetContext().clauseSource,
        "The %s clause on the %s directive accepts only one value"_err_en_US,
        parser::ToUpperCaseLetters(
            llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_device_type)
                .str()),
        ContextDirectiveAsFortran());
  }
  ResetCrtGroup();
}

void AccStructureChecker::Enter(const parser::AccClause::Seq &g) {
  llvm::acc::Clause crtClause = llvm::acc::Clause::ACCC_seq;
  if (GetContext().directive == llvm::acc::Directive::ACCD_routine) {
    CheckMutuallyExclusivePerGroup(crtClause,
````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Closes the current lexical scope or compound statement.
  **L938 CN**: 结束当前词法作用域或复合语句块。
- **L939 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L939 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L940 EN**: Executes a standalone statement or declaration: `modifierClause, llvm::acc::Clause::ACCC_copyout);`.
  **L940 CN**: 执行一条独立语句或声明：`modifierClause, llvm::acc::Clause::ACCC_copyout);`。
- **L941 EN**: Closes the current lexical scope or compound statement.
  **L941 CN**: 结束当前词法作用域或复合语句块。
- **L942 EN**: Blank line separating nearby declarations or logic blocks.
  **L942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L943 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::DeviceType &d) {`.
  **L943 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::DeviceType &d) {`。
- **L944 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L944 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L945 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L945 CN**: 开始 `if` 控制流语句并计算其条件。
- **L946 EN**: Starts a function, method, lambda, or structured scope: `d.v.v.size() > 1) {`.
  **L946 CN**: 开始一个函数、方法、lambda 或结构化作用域：`d.v.v.size() > 1) {`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"The %s clause on the %s directive accepts only one value"_err_en_US,`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`"The %s clause on the %s directive accepts only one value"_err_en_US,`。
- **L949 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L949 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L950 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L950 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L952 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L952 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L953 EN**: Closes the current lexical scope or compound statement.
  **L953 CN**: 结束当前词法作用域或复合语句块。
- **L954 EN**: Executes a call or declaration centered on `ResetCrtGroup`.
  **L954 CN**: 执行以 `ResetCrtGroup` 为核心的调用或声明。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L957 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Seq &g) {`.
  **L957 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Seq &g) {`。
- **L958 EN**: Initializes variable `crtClause` from the right-hand expression.
  **L958 CN**: 使用右侧表达式初始化变量 `crtClause`。
- **L959 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L959 CN**: 开始 `if` 控制流语句并计算其条件。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckMutuallyExclusivePerGroup(crtClause,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckMutuallyExclusivePerGroup(crtClause,`。

### Lines 961-984

````cpp
        llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);
  }
  CheckAllowed(crtClause);
}

void AccStructureChecker::Enter(const parser::AccClause::Vector &g) {
  llvm::acc::Clause crtClause = llvm::acc::Clause::ACCC_vector;
  if (GetContext().directive == llvm::acc::Directive::ACCD_routine) {
    CheckMutuallyExclusivePerGroup(crtClause,
        llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);
  }
  CheckAllowed(crtClause);
  if (GetContext().directive != llvm::acc::Directive::ACCD_routine) {
    CheckAllowedOncePerGroup(crtClause, llvm::acc::Clause::ACCC_device_type);
  }
}

void AccStructureChecker::Enter(const parser::AccClause::Worker &g) {
  llvm::acc::Clause crtClause = llvm::acc::Clause::ACCC_worker;
  if (GetContext().directive == llvm::acc::Directive::ACCD_routine) {
    CheckMutuallyExclusivePerGroup(crtClause,
        llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);
  }
  CheckAllowed(crtClause);
````
- **L961 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`.
  **L961 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`。
- **L962 EN**: Closes the current lexical scope or compound statement.
  **L962 CN**: 结束当前词法作用域或复合语句块。
- **L963 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L963 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L964 EN**: Closes the current lexical scope or compound statement.
  **L964 CN**: 结束当前词法作用域或复合语句块。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L966 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Vector &g) {`.
  **L966 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Vector &g) {`。
- **L967 EN**: Initializes variable `crtClause` from the right-hand expression.
  **L967 CN**: 使用右侧表达式初始化变量 `crtClause`。
- **L968 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L968 CN**: 开始 `if` 控制流语句并计算其条件。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckMutuallyExclusivePerGroup(crtClause,`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckMutuallyExclusivePerGroup(crtClause,`。
- **L970 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`.
  **L970 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`。
- **L971 EN**: Closes the current lexical scope or compound statement.
  **L971 CN**: 结束当前词法作用域或复合语句块。
- **L972 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L972 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L973 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L973 CN**: 开始 `if` 控制流语句并计算其条件。
- **L974 EN**: Executes a call or declaration centered on `CheckAllowedOncePerGroup`.
  **L974 CN**: 执行以 `CheckAllowedOncePerGroup` 为核心的调用或声明。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Closes the current lexical scope or compound statement.
  **L976 CN**: 结束当前词法作用域或复合语句块。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L978 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Worker &g) {`.
  **L978 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Worker &g) {`。
- **L979 EN**: Initializes variable `crtClause` from the right-hand expression.
  **L979 CN**: 使用右侧表达式初始化变量 `crtClause`。
- **L980 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L980 CN**: 开始 `if` 控制流语句并计算其条件。
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckMutuallyExclusivePerGroup(crtClause,`.
  **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckMutuallyExclusivePerGroup(crtClause,`。
- **L982 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`.
  **L982 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`。
- **L983 EN**: Closes the current lexical scope or compound statement.
  **L983 CN**: 结束当前词法作用域或复合语句块。
- **L984 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L984 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。

### Lines 985-1008

````cpp
  if (GetContext().directive != llvm::acc::Directive::ACCD_routine) {
    CheckAllowedOncePerGroup(crtClause, llvm::acc::Clause::ACCC_device_type);
  }
}

void AccStructureChecker::Enter(const parser::AccClause::Tile &g) {
  CheckAllowed(llvm::acc::Clause::ACCC_tile);
  CheckAllowedOncePerGroup(
      llvm::acc::Clause::ACCC_tile, llvm::acc::Clause::ACCC_device_type);
}

void AccStructureChecker::Enter(const parser::AccClause::Gang &g) {
  llvm::acc::Clause crtClause = llvm::acc::Clause::ACCC_gang;
  if (GetContext().directive == llvm::acc::Directive::ACCD_routine) {
    CheckMutuallyExclusivePerGroup(crtClause,
        llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);
  }
  CheckAllowed(crtClause);
  if (GetContext().directive != llvm::acc::Directive::ACCD_routine) {
    CheckAllowedOncePerGroup(crtClause, llvm::acc::Clause::ACCC_device_type);
  }

  if (g.v) {
    bool hasNum = false;
````
- **L985 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L985 CN**: 开始 `if` 控制流语句并计算其条件。
- **L986 EN**: Executes a call or declaration centered on `CheckAllowedOncePerGroup`.
  **L986 CN**: 执行以 `CheckAllowedOncePerGroup` 为核心的调用或声明。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Closes the current lexical scope or compound statement.
  **L988 CN**: 结束当前词法作用域或复合语句块。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L990 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Tile &g) {`.
  **L990 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Tile &g) {`。
- **L991 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L991 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L992 EN**: Continues logic associated with callable symbol `CheckAllowedOncePerGroup`.
  **L992 CN**: 继续与可调用符号 `CheckAllowedOncePerGroup` 相关的逻辑。
- **L993 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_tile, llvm::acc::Clause::ACCC_device_type);`.
  **L993 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_tile, llvm::acc::Clause::ACCC_device_type);`。
- **L994 EN**: Closes the current lexical scope or compound statement.
  **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Blank line separating nearby declarations or logic blocks.
  **L995 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L996 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Gang &g) {`.
  **L996 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Gang &g) {`。
- **L997 EN**: Initializes variable `crtClause` from the right-hand expression.
  **L997 CN**: 使用右侧表达式初始化变量 `crtClause`。
- **L998 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L998 CN**: 开始 `if` 控制流语句并计算其条件。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckMutuallyExclusivePerGroup(crtClause,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckMutuallyExclusivePerGroup(crtClause,`。
- **L1000 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`.
  **L1000 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_device_type, routineMutuallyExclusiveClauses);`。
- **L1001 EN**: Closes the current lexical scope or compound statement.
  **L1001 CN**: 结束当前词法作用域或复合语句块。
- **L1002 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1002 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1003 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1003 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1004 EN**: Executes a call or declaration centered on `CheckAllowedOncePerGroup`.
  **L1004 CN**: 执行以 `CheckAllowedOncePerGroup` 为核心的调用或声明。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1007 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1007 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1008 EN**: Initializes variable `hasNum` from the right-hand expression.
  **L1008 CN**: 使用右侧表达式初始化变量 `hasNum`。

### Lines 1009-1032

````cpp
    bool hasDim = false;
    bool hasStatic = false;
    const Fortran::parser::AccGangArgList &x = *g.v;
    for (const Fortran::parser::AccGangArg &gangArg : x.v) {
      if (std::get_if<Fortran::parser::AccGangArg::Num>(&gangArg.u)) {
        hasNum = true;
      } else if (std::get_if<Fortran::parser::AccGangArg::Dim>(&gangArg.u)) {
        hasDim = true;
      } else if (std::get_if<Fortran::parser::AccGangArg::Static>(&gangArg.u)) {
        hasStatic = true;
      }
    }

    if (GetContext().directive == llvm::acc::Directive::ACCD_routine &&
        (hasStatic || hasNum)) {
      context_.Say(GetContext().clauseSource,
          "Only the dim argument is allowed on the %s clause on the %s directive"_err_en_US,
          parser::ToUpperCaseLetters(
              llvm::acc::getOpenACCClauseName(llvm::acc::Clause::ACCC_gang)
                  .str()),
          ContextDirectiveAsFortran());
    }

    if (hasDim && hasNum) {
````
- **L1009 EN**: Initializes variable `hasDim` from the right-hand expression.
  **L1009 CN**: 使用右侧表达式初始化变量 `hasDim`。
- **L1010 EN**: Initializes variable `hasStatic` from the right-hand expression.
  **L1010 CN**: 使用右侧表达式初始化变量 `hasStatic`。
- **L1011 EN**: Executes a standalone statement or declaration: `const Fortran::parser::AccGangArgList &x = *g.v;`.
  **L1011 CN**: 执行一条独立语句或声明：`const Fortran::parser::AccGangArgList &x = *g.v;`。
- **L1012 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1012 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1013 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1013 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1014 EN**: Executes a standalone statement or declaration: `hasNum = true;`.
  **L1014 CN**: 执行一条独立语句或声明：`hasNum = true;`。
- **L1015 EN**: Transitions from the previous branch into an `else if` condition.
  **L1015 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1016 EN**: Executes a standalone statement or declaration: `hasDim = true;`.
  **L1016 CN**: 执行一条独立语句或声明：`hasDim = true;`。
- **L1017 EN**: Transitions from the previous branch into an `else if` condition.
  **L1017 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1018 EN**: Executes a standalone statement or declaration: `hasStatic = true;`.
  **L1018 CN**: 执行一条独立语句或声明：`hasStatic = true;`。
- **L1019 EN**: Closes the current lexical scope or compound statement.
  **L1019 CN**: 结束当前词法作用域或复合语句块。
- **L1020 EN**: Closes the current lexical scope or compound statement.
  **L1020 CN**: 结束当前词法作用域或复合语句块。
- **L1021 EN**: Blank line separating nearby declarations or logic blocks.
  **L1021 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1022 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1022 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1023 EN**: Starts a function, method, lambda, or structured scope: `(hasStatic || hasNum)) {`.
  **L1023 CN**: 开始一个函数、方法、lambda 或结构化作用域：`(hasStatic || hasNum)) {`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Only the dim argument is allowed on the %s clause on the %s directive"_err_en_US,`.
  **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Only the dim argument is allowed on the %s clause on the %s directive"_err_en_US,`。
- **L1026 EN**: Continues logic associated with callable symbol `ToUpperCaseLetters`.
  **L1026 CN**: 继续与可调用符号 `ToUpperCaseLetters` 相关的逻辑。
- **L1027 EN**: Continues logic associated with callable symbol `getOpenACCClauseName`.
  **L1027 CN**: 继续与可调用符号 `getOpenACCClauseName` 相关的逻辑。
- **L1028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.str()),`.
  **L1028 CN**: 继续一个多行参数列表、初始化器或聚合项：`.str()),`。
- **L1029 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L1029 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1032 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1032 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 1033-1056

````cpp
      context_.Say(GetContext().clauseSource,
          "The num argument is not allowed when dim is specified"_err_en_US);
    }
  }
}

void AccStructureChecker::Enter(const parser::AccClause::NumGangs &n) {
  CheckAllowed(llvm::acc::Clause::ACCC_num_gangs,
      /*warnInsteadOfError=*/GetContext().directive ==
              llvm::acc::Directive::ACCD_serial ||
          GetContext().directive == llvm::acc::Directive::ACCD_serial_loop);
  CheckAllowedOncePerGroup(
      llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_device_type);

  if (n.v.size() > 3)
    context_.Say(GetContext().clauseSource,
        "NUM_GANGS clause accepts a maximum of 3 arguments"_err_en_US);
}

void AccStructureChecker::Enter(const parser::AccClause::NumWorkers &n) {
  CheckAllowed(llvm::acc::Clause::ACCC_num_workers,
      /*warnInsteadOfError=*/GetContext().directive ==
              llvm::acc::Directive::ACCD_serial ||
          GetContext().directive == llvm::acc::Directive::ACCD_serial_loop);
````
- **L1033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1033 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1034 EN**: Executes a standalone statement or declaration: `"The num argument is not allowed when dim is specified"_err_en_US);`.
  **L1034 CN**: 执行一条独立语句或声明：`"The num argument is not allowed when dim is specified"_err_en_US);`。
- **L1035 EN**: Closes the current lexical scope or compound statement.
  **L1035 CN**: 结束当前词法作用域或复合语句块。
- **L1036 EN**: Closes the current lexical scope or compound statement.
  **L1036 CN**: 结束当前词法作用域或复合语句块。
- **L1037 EN**: Closes the current lexical scope or compound statement.
  **L1037 CN**: 结束当前词法作用域或复合语句块。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1039 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::NumGangs &n) {`.
  **L1039 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::NumGangs &n) {`。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckAllowed(llvm::acc::Clause::ACCC_num_gangs,`.
  **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckAllowed(llvm::acc::Clause::ACCC_num_gangs,`。
- **L1041 EN**: Comment explains nearby logic, intent, or metadata: `warnInsteadOfError=*/GetContext().directive ==`.
  **L1041 CN**: 注释说明附近代码的逻辑、意图或元数据：`warnInsteadOfError=*/GetContext().directive ==`。
- **L1042 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive::ACCD_serial ||`.
  **L1042 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive::ACCD_serial ||`。
- **L1043 EN**: Executes a call or declaration centered on `GetContext`.
  **L1043 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L1044 EN**: Continues logic associated with callable symbol `CheckAllowedOncePerGroup`.
  **L1044 CN**: 继续与可调用符号 `CheckAllowedOncePerGroup` 相关的逻辑。
- **L1045 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_device_type);`.
  **L1045 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_num_gangs, llvm::acc::Clause::ACCC_device_type);`。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1047 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1047 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1049 EN**: Executes a standalone statement or declaration: `"NUM_GANGS clause accepts a maximum of 3 arguments"_err_en_US);`.
  **L1049 CN**: 执行一条独立语句或声明：`"NUM_GANGS clause accepts a maximum of 3 arguments"_err_en_US);`。
- **L1050 EN**: Closes the current lexical scope or compound statement.
  **L1050 CN**: 结束当前词法作用域或复合语句块。
- **L1051 EN**: Blank line separating nearby declarations or logic blocks.
  **L1051 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1052 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::NumWorkers &n) {`.
  **L1052 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::NumWorkers &n) {`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckAllowed(llvm::acc::Clause::ACCC_num_workers,`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckAllowed(llvm::acc::Clause::ACCC_num_workers,`。
- **L1054 EN**: Comment explains nearby logic, intent, or metadata: `warnInsteadOfError=*/GetContext().directive ==`.
  **L1054 CN**: 注释说明附近代码的逻辑、意图或元数据：`warnInsteadOfError=*/GetContext().directive ==`。
- **L1055 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive::ACCD_serial ||`.
  **L1055 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive::ACCD_serial ||`。
- **L1056 EN**: Executes a call or declaration centered on `GetContext`.
  **L1056 CN**: 执行以 `GetContext` 为核心的调用或声明。

### Lines 1057-1080

````cpp
  CheckAllowedOncePerGroup(
      llvm::acc::Clause::ACCC_num_workers, llvm::acc::Clause::ACCC_device_type);
}

void AccStructureChecker::Enter(const parser::AccClause::VectorLength &n) {
  CheckAllowed(llvm::acc::Clause::ACCC_vector_length,
      /*warnInsteadOfError=*/GetContext().directive ==
              llvm::acc::Directive::ACCD_serial ||
          GetContext().directive == llvm::acc::Directive::ACCD_serial_loop);
  CheckAllowedOncePerGroup(llvm::acc::Clause::ACCC_vector_length,
      llvm::acc::Clause::ACCC_device_type);
}

void AccStructureChecker::Enter(const parser::AccClause::Reduction &reduction) {
  CheckAllowed(llvm::acc::Clause::ACCC_reduction);

  // From OpenACC 3.3
  // At a minimum, the supported data types include Fortran logical as well as
  // the numerical data types (e.g. integer, real, double precision, complex).
  // However, for each reduction operator, the supported data types include only
  // the types permitted as operands to the corresponding operator in the base
  // language where (1) for max and min, the corresponding operator is less-than
  // and (2) for other operators, the operands and the result are the same type.
  //
````
- **L1057 EN**: Continues logic associated with callable symbol `CheckAllowedOncePerGroup`.
  **L1057 CN**: 继续与可调用符号 `CheckAllowedOncePerGroup` 相关的逻辑。
- **L1058 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_num_workers, llvm::acc::Clause::ACCC_device_type);`.
  **L1058 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_num_workers, llvm::acc::Clause::ACCC_device_type);`。
- **L1059 EN**: Closes the current lexical scope or compound statement.
  **L1059 CN**: 结束当前词法作用域或复合语句块。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1061 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::VectorLength &n) {`.
  **L1061 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::VectorLength &n) {`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckAllowed(llvm::acc::Clause::ACCC_vector_length,`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckAllowed(llvm::acc::Clause::ACCC_vector_length,`。
- **L1063 EN**: Comment explains nearby logic, intent, or metadata: `warnInsteadOfError=*/GetContext().directive ==`.
  **L1063 CN**: 注释说明附近代码的逻辑、意图或元数据：`warnInsteadOfError=*/GetContext().directive ==`。
- **L1064 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive::ACCD_serial ||`.
  **L1064 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive::ACCD_serial ||`。
- **L1065 EN**: Executes a call or declaration centered on `GetContext`.
  **L1065 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `CheckAllowedOncePerGroup(llvm::acc::Clause::ACCC_vector_length,`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`CheckAllowedOncePerGroup(llvm::acc::Clause::ACCC_vector_length,`。
- **L1067 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_device_type);`.
  **L1067 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_device_type);`。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1070 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Reduction &reduction) {`.
  **L1070 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Reduction &reduction) {`。
- **L1071 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1071 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1072 EN**: Blank line separating nearby declarations or logic blocks.
  **L1072 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1073 EN**: Comment explains nearby logic, intent, or metadata: `From OpenACC 3.3`.
  **L1073 CN**: 注释说明附近代码的逻辑、意图或元数据：`From OpenACC 3.3`。
- **L1074 EN**: Comment explains nearby logic, intent, or metadata: `At a minimum, the supported data types include Fortran logical as well as`.
  **L1074 CN**: 注释说明附近代码的逻辑、意图或元数据：`At a minimum, the supported data types include Fortran logical as well as`。
- **L1075 EN**: Comment explains nearby logic, intent, or metadata: `the numerical data types (e.g. integer, real, double precision, complex).`.
  **L1075 CN**: 注释说明附近代码的逻辑、意图或元数据：`the numerical data types (e.g. integer, real, double precision, complex).`。
- **L1076 EN**: Comment explains nearby logic, intent, or metadata: `However, for each reduction operator, the supported data types include only`.
  **L1076 CN**: 注释说明附近代码的逻辑、意图或元数据：`However, for each reduction operator, the supported data types include only`。
- **L1077 EN**: Comment explains nearby logic, intent, or metadata: `the types permitted as operands to the corresponding operator in the base`.
  **L1077 CN**: 注释说明附近代码的逻辑、意图或元数据：`the types permitted as operands to the corresponding operator in the base`。
- **L1078 EN**: Comment explains nearby logic, intent, or metadata: `language where (1) for max and min, the corresponding operator is less-than`.
  **L1078 CN**: 注释说明附近代码的逻辑、意图或元数据：`language where (1) for max and min, the corresponding operator is less-than`。
- **L1079 EN**: Comment explains nearby logic, intent, or metadata: `and (2) for other operators, the operands and the result are the same type.`.
  **L1079 CN**: 注释说明附近代码的逻辑、意图或元数据：`and (2) for other operators, the operands and the result are the same type.`。
- **L1080 EN**: Separator comment used for visual grouping.
  **L1080 CN**: 用于视觉分组的分隔注释。

### Lines 1081-1104

````cpp
  // The following check that the reduction operator is supported with the given
  // type.
  const parser::AccObjectListWithReduction &list{reduction.v};
  const auto &op{std::get<parser::ReductionOperator>(list.t)};
  const auto &objects{std::get<parser::AccObjectList>(list.t)};

  for (const auto &object : objects.v) {
    common::visit(
        common::visitors{
            [&](const parser::Designator &designator) {
              if (const auto *name =
                      parser::GetDesignatorNameIfDataRef(designator)) {
                if (name->symbol) {
                  if (const auto *type{name->symbol->GetType()}) {
                    if (type->IsNumeric(TypeCategory::Integer) &&
                        !reductionIntegerSet.test(op.v)) {
                      context_.Say(GetContext().clauseSource,
                          "reduction operator not supported for integer type"_err_en_US);
                    } else if (type->IsNumeric(TypeCategory::Real) &&
                        !reductionRealSet.test(op.v)) {
                      context_.Say(GetContext().clauseSource,
                          "reduction operator not supported for real type"_err_en_US);
                    } else if (type->IsNumeric(TypeCategory::Complex) &&
                        !reductionComplexSet.test(op.v)) {
````
- **L1081 EN**: Comment explains nearby logic, intent, or metadata: `The following check that the reduction operator is supported with the given`.
  **L1081 CN**: 注释说明附近代码的逻辑、意图或元数据：`The following check that the reduction operator is supported with the given`。
- **L1082 EN**: Comment explains nearby logic, intent, or metadata: `type.`.
  **L1082 CN**: 注释说明附近代码的逻辑、意图或元数据：`type.`。
- **L1083 EN**: Executes a standalone statement or declaration: `const parser::AccObjectListWithReduction &list{reduction.v};`.
  **L1083 CN**: 执行一条独立语句或声明：`const parser::AccObjectListWithReduction &list{reduction.v};`。
- **L1084 EN**: Executes a call or declaration centered on `&op{std::get<parser::ReductionOperator>`.
  **L1084 CN**: 执行以 `&op{std::get<parser::ReductionOperator>` 为核心的调用或声明。
- **L1085 EN**: Executes a call or declaration centered on `&objects{std::get<parser::AccObjectList>`.
  **L1085 CN**: 执行以 `&objects{std::get<parser::AccObjectList>` 为核心的调用或声明。
- **L1086 EN**: Blank line separating nearby declarations or logic blocks.
  **L1086 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1087 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1087 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1088 EN**: Continues logic associated with callable symbol `visit`.
  **L1088 CN**: 继续与可调用符号 `visit` 相关的逻辑。
- **L1089 EN**: Continues the surrounding expression or declaration: `common::visitors{`.
  **L1089 CN**: 继续构造周围的表达式或声明：`common::visitors{`。
- **L1090 EN**: Starts a function, method, lambda, or structured scope: `[&](const parser::Designator &designator) {`.
  **L1090 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const parser::Designator &designator) {`。
- **L1091 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1091 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1092 EN**: Starts a function, method, lambda, or structured scope: `parser::GetDesignatorNameIfDataRef(designator)) {`.
  **L1092 CN**: 开始一个函数、方法、lambda 或结构化作用域：`parser::GetDesignatorNameIfDataRef(designator)) {`。
- **L1093 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1093 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1094 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1094 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1095 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1095 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1096 EN**: Starts a function, method, lambda, or structured scope: `!reductionIntegerSet.test(op.v)) {`.
  **L1096 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!reductionIntegerSet.test(op.v)) {`。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1098 EN**: Executes a standalone statement or declaration: `"reduction operator not supported for integer type"_err_en_US);`.
  **L1098 CN**: 执行一条独立语句或声明：`"reduction operator not supported for integer type"_err_en_US);`。
- **L1099 EN**: Transitions from the previous branch into an `else if` condition.
  **L1099 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1100 EN**: Starts a function, method, lambda, or structured scope: `!reductionRealSet.test(op.v)) {`.
  **L1100 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!reductionRealSet.test(op.v)) {`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1102 EN**: Executes a standalone statement or declaration: `"reduction operator not supported for real type"_err_en_US);`.
  **L1102 CN**: 执行一条独立语句或声明：`"reduction operator not supported for real type"_err_en_US);`。
- **L1103 EN**: Transitions from the previous branch into an `else if` condition.
  **L1103 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1104 EN**: Starts a function, method, lambda, or structured scope: `!reductionComplexSet.test(op.v)) {`.
  **L1104 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!reductionComplexSet.test(op.v)) {`。

### Lines 1105-1128

````cpp
                      context_.Say(GetContext().clauseSource,
                          "reduction operator not supported for complex type"_err_en_US);
                    } else if (type->category() ==
                            Fortran::semantics::DeclTypeSpec::Category::
                                Logical &&
                        !reductionLogicalSet.test(op.v)) {
                      context_.Say(GetContext().clauseSource,
                          "reduction operator not supported for logical type"_err_en_US);
                    }
                  }
                  // TODO: check composite type.
                }
              }
            },
            [&](const Fortran::parser::Name &name) {
              // TODO: check common block
            }},
        object.u);
  }
}

void AccStructureChecker::Enter(const parser::AccClause::Self &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_self);
  const std::optional<parser::AccSelfClause> &accSelfClause = x.v;
````
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1106 EN**: Executes a standalone statement or declaration: `"reduction operator not supported for complex type"_err_en_US);`.
  **L1106 CN**: 执行一条独立语句或声明：`"reduction operator not supported for complex type"_err_en_US);`。
- **L1107 EN**: Transitions from the previous branch into an `else if` condition.
  **L1107 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1108 EN**: Continues the surrounding expression or declaration: `Fortran::semantics::DeclTypeSpec::Category::`.
  **L1108 CN**: 继续构造周围的表达式或声明：`Fortran::semantics::DeclTypeSpec::Category::`。
- **L1109 EN**: Continues the surrounding expression or declaration: `Logical &&`.
  **L1109 CN**: 继续构造周围的表达式或声明：`Logical &&`。
- **L1110 EN**: Starts a function, method, lambda, or structured scope: `!reductionLogicalSet.test(op.v)) {`.
  **L1110 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!reductionLogicalSet.test(op.v)) {`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1112 EN**: Executes a standalone statement or declaration: `"reduction operator not supported for logical type"_err_en_US);`.
  **L1112 CN**: 执行一条独立语句或声明：`"reduction operator not supported for logical type"_err_en_US);`。
- **L1113 EN**: Closes the current lexical scope or compound statement.
  **L1113 CN**: 结束当前词法作用域或复合语句块。
- **L1114 EN**: Closes the current lexical scope or compound statement.
  **L1114 CN**: 结束当前词法作用域或复合语句块。
- **L1115 EN**: Comment records a pending task or caution: `TODO: check composite type.`.
  **L1115 CN**: 注释记录待办事项或注意点：`TODO: check composite type.`。
- **L1116 EN**: Closes the current lexical scope or compound statement.
  **L1116 CN**: 结束当前词法作用域或复合语句块。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L1119 EN**: Starts a function, method, lambda, or structured scope: `[&](const Fortran::parser::Name &name) {`.
  **L1119 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](const Fortran::parser::Name &name) {`。
- **L1120 EN**: Comment records a pending task or caution: `TODO: check common block`.
  **L1120 CN**: 注释记录待办事项或注意点：`TODO: check common block`。
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `}},`.
  **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`}},`。
- **L1122 EN**: Executes a standalone statement or declaration: `object.u);`.
  **L1122 CN**: 执行一条独立语句或声明：`object.u);`。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Closes the current lexical scope or compound statement.
  **L1124 CN**: 结束当前词法作用域或复合语句块。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1126 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Self &x) {`.
  **L1126 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Self &x) {`。
- **L1127 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1127 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1128 EN**: Executes a standalone statement or declaration: `const std::optional<parser::AccSelfClause> &accSelfClause = x.v;`.
  **L1128 CN**: 执行一条独立语句或声明：`const std::optional<parser::AccSelfClause> &accSelfClause = x.v;`。

### Lines 1129-1152

````cpp
  if (GetContext().directive == llvm::acc::Directive::ACCD_update &&
      ((accSelfClause &&
           std::holds_alternative<std::optional<parser::ScalarLogicalExpr>>(
               (*accSelfClause).u)) ||
          !accSelfClause)) {
    context_.Say(GetContext().clauseSource,
        "SELF clause on the %s directive must have a var-list"_err_en_US,
        ContextDirectiveAsFortran());
  } else if (GetContext().directive != llvm::acc::Directive::ACCD_update &&
      accSelfClause &&
      std::holds_alternative<parser::AccObjectList>((*accSelfClause).u)) {
    const auto &accObjectList =
        std::get<parser::AccObjectList>((*accSelfClause).u);
    if (accObjectList.v.size() != 1) {
      context_.Say(GetContext().clauseSource,
          "SELF clause on the %s directive only accepts optional scalar logical"
          " expression"_err_en_US,
          ContextDirectiveAsFortran());
    }
  }
}

void AccStructureChecker::Enter(const parser::AccClause::Collapse &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_collapse);
````
- **L1129 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1129 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1130 EN**: Continues the surrounding expression or declaration: `((accSelfClause &&`.
  **L1130 CN**: 继续构造周围的表达式或声明：`((accSelfClause &&`。
- **L1131 EN**: Continues logic associated with callable symbol `ScalarLogicalExpr>>`.
  **L1131 CN**: 继续与可调用符号 `ScalarLogicalExpr>>` 相关的逻辑。
- **L1132 EN**: Continues the surrounding expression or declaration: `(*accSelfClause).u)) ||`.
  **L1132 CN**: 继续构造周围的表达式或声明：`(*accSelfClause).u)) ||`。
- **L1133 EN**: Continues the surrounding expression or declaration: `!accSelfClause)) {`.
  **L1133 CN**: 继续构造周围的表达式或声明：`!accSelfClause)) {`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"SELF clause on the %s directive must have a var-list"_err_en_US,`.
  **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`"SELF clause on the %s directive must have a var-list"_err_en_US,`。
- **L1136 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L1136 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L1137 EN**: Transitions from the previous branch into an `else if` condition.
  **L1137 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L1138 EN**: Continues the surrounding expression or declaration: `accSelfClause &&`.
  **L1138 CN**: 继续构造周围的表达式或声明：`accSelfClause &&`。
- **L1139 EN**: Starts a function, method, lambda, or structured scope: `std::holds_alternative<parser::AccObjectList>((*accSelfClause).u)) {`.
  **L1139 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::holds_alternative<parser::AccObjectList>((*accSelfClause).u)) {`。
- **L1140 EN**: Continues the surrounding expression or declaration: `const auto &accObjectList =`.
  **L1140 CN**: 继续构造周围的表达式或声明：`const auto &accObjectList =`。
- **L1141 EN**: Executes a call or declaration centered on `std::get<parser::AccObjectList>`.
  **L1141 CN**: 执行以 `std::get<parser::AccObjectList>` 为核心的调用或声明。
- **L1142 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1142 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Say(GetContext().clauseSource,`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Say(GetContext().clauseSource,`。
- **L1144 EN**: Continues the surrounding expression or declaration: `"SELF clause on the %s directive only accepts optional scalar logical"`.
  **L1144 CN**: 继续构造周围的表达式或声明：`"SELF clause on the %s directive only accepts optional scalar logical"`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" expression"_err_en_US,`.
  **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`" expression"_err_en_US,`。
- **L1146 EN**: Executes a call or declaration centered on `ContextDirectiveAsFortran`.
  **L1146 CN**: 执行以 `ContextDirectiveAsFortran` 为核心的调用或声明。
- **L1147 EN**: Closes the current lexical scope or compound statement.
  **L1147 CN**: 结束当前词法作用域或复合语句块。
- **L1148 EN**: Closes the current lexical scope or compound statement.
  **L1148 CN**: 结束当前词法作用域或复合语句块。
- **L1149 EN**: Closes the current lexical scope or compound statement.
  **L1149 CN**: 结束当前词法作用域或复合语句块。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1151 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Collapse &x) {`.
  **L1151 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Collapse &x) {`。
- **L1152 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1152 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。

### Lines 1153-1176

````cpp
  CheckAllowedOncePerGroup(
      llvm::acc::Clause::ACCC_collapse, llvm::acc::Clause::ACCC_device_type);
  const parser::AccCollapseArg &accCollapseArg = x.v;
  const auto &collapseValue{
      std::get<parser::ScalarIntConstantExpr>(accCollapseArg.t)};
  RequiresConstantPositiveParameter(
      llvm::acc::Clause::ACCC_collapse, collapseValue);
}

void AccStructureChecker::Enter(const parser::AccClause::Present &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_present);
  CheckMultipleOccurrenceInDeclare(x.v, llvm::acc::Clause::ACCC_present);
}

void AccStructureChecker::Enter(const parser::AccClause::Copy &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_copy);
  CheckMultipleOccurrenceInDeclare(x.v, llvm::acc::Clause::ACCC_copy);
}

void AccStructureChecker::Enter(const parser::AccClause::Deviceptr &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_deviceptr);
  CheckMultipleOccurrenceInDeclare(x.v, llvm::acc::Clause::ACCC_deviceptr);
}

````
- **L1153 EN**: Continues logic associated with callable symbol `CheckAllowedOncePerGroup`.
  **L1153 CN**: 继续与可调用符号 `CheckAllowedOncePerGroup` 相关的逻辑。
- **L1154 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_collapse, llvm::acc::Clause::ACCC_device_type);`.
  **L1154 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_collapse, llvm::acc::Clause::ACCC_device_type);`。
- **L1155 EN**: Executes a standalone statement or declaration: `const parser::AccCollapseArg &accCollapseArg = x.v;`.
  **L1155 CN**: 执行一条独立语句或声明：`const parser::AccCollapseArg &accCollapseArg = x.v;`。
- **L1156 EN**: Continues the surrounding expression or declaration: `const auto &collapseValue{`.
  **L1156 CN**: 继续构造周围的表达式或声明：`const auto &collapseValue{`。
- **L1157 EN**: Executes a call or declaration centered on `std::get<parser::ScalarIntConstantExpr>`.
  **L1157 CN**: 执行以 `std::get<parser::ScalarIntConstantExpr>` 为核心的调用或声明。
- **L1158 EN**: Continues logic associated with callable symbol `RequiresConstantPositiveParameter`.
  **L1158 CN**: 继续与可调用符号 `RequiresConstantPositiveParameter` 相关的逻辑。
- **L1159 EN**: Executes a standalone statement or declaration: `llvm::acc::Clause::ACCC_collapse, collapseValue);`.
  **L1159 CN**: 执行一条独立语句或声明：`llvm::acc::Clause::ACCC_collapse, collapseValue);`。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1162 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Present &x) {`.
  **L1162 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Present &x) {`。
- **L1163 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1163 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1164 EN**: Executes a call or declaration centered on `CheckMultipleOccurrenceInDeclare`.
  **L1164 CN**: 执行以 `CheckMultipleOccurrenceInDeclare` 为核心的调用或声明。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1167 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Copy &x) {`.
  **L1167 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Copy &x) {`。
- **L1168 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1168 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1169 EN**: Executes a call or declaration centered on `CheckMultipleOccurrenceInDeclare`.
  **L1169 CN**: 执行以 `CheckMultipleOccurrenceInDeclare` 为核心的调用或声明。
- **L1170 EN**: Closes the current lexical scope or compound statement.
  **L1170 CN**: 结束当前词法作用域或复合语句块。
- **L1171 EN**: Blank line separating nearby declarations or logic blocks.
  **L1171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1172 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Deviceptr &x) {`.
  **L1172 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Deviceptr &x) {`。
- **L1173 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1173 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1174 EN**: Executes a call or declaration centered on `CheckMultipleOccurrenceInDeclare`.
  **L1174 CN**: 执行以 `CheckMultipleOccurrenceInDeclare` 为核心的调用或声明。
- **L1175 EN**: Closes the current lexical scope or compound statement.
  **L1175 CN**: 结束当前词法作用域或复合语句块。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1177-1200

````cpp
void AccStructureChecker::Enter(const parser::AccClause::DeviceResident &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_device_resident);
  CheckMultipleOccurrenceInDeclare(
      x.v, llvm::acc::Clause::ACCC_device_resident);
}

void AccStructureChecker::Enter(const parser::AccClause::Link &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_link);
  CheckMultipleOccurrenceInDeclare(x.v, llvm::acc::Clause::ACCC_link);
}

void AccStructureChecker::Enter(const parser::AccClause::Shortloop &x) {
  if (CheckAllowed(llvm::acc::Clause::ACCC_shortloop)) {
    context_.Warn(common::UsageWarning::OpenAccUsage, GetContext().clauseSource,
        "Non-standard shortloop clause ignored"_warn_en_US);
  }
}

void AccStructureChecker::Enter(const parser::AccClause::If &x) {
  CheckAllowed(llvm::acc::Clause::ACCC_if);
  if (const auto *expr{GetExpr(x.v)}) {
    if (auto type{expr->GetType()}) {
      if (type->category() == TypeCategory::Integer ||
          type->category() == TypeCategory::Logical) {
````
- **L1177 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::DeviceResident &x) {`.
  **L1177 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::DeviceResident &x) {`。
- **L1178 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1178 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1179 EN**: Continues logic associated with callable symbol `CheckMultipleOccurrenceInDeclare`.
  **L1179 CN**: 继续与可调用符号 `CheckMultipleOccurrenceInDeclare` 相关的逻辑。
- **L1180 EN**: Executes a standalone statement or declaration: `x.v, llvm::acc::Clause::ACCC_device_resident);`.
  **L1180 CN**: 执行一条独立语句或声明：`x.v, llvm::acc::Clause::ACCC_device_resident);`。
- **L1181 EN**: Closes the current lexical scope or compound statement.
  **L1181 CN**: 结束当前词法作用域或复合语句块。
- **L1182 EN**: Blank line separating nearby declarations or logic blocks.
  **L1182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1183 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Link &x) {`.
  **L1183 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Link &x) {`。
- **L1184 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1184 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1185 EN**: Executes a call or declaration centered on `CheckMultipleOccurrenceInDeclare`.
  **L1185 CN**: 执行以 `CheckMultipleOccurrenceInDeclare` 为核心的调用或声明。
- **L1186 EN**: Closes the current lexical scope or compound statement.
  **L1186 CN**: 结束当前词法作用域或复合语句块。
- **L1187 EN**: Blank line separating nearby declarations or logic blocks.
  **L1187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1188 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::Shortloop &x) {`.
  **L1188 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::Shortloop &x) {`。
- **L1189 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1189 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::OpenAccUsage, GetContext().clauseSource,`.
  **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::OpenAccUsage, GetContext().clauseSource,`。
- **L1191 EN**: Executes a standalone statement or declaration: `"Non-standard shortloop clause ignored"_warn_en_US);`.
  **L1191 CN**: 执行一条独立语句或声明：`"Non-standard shortloop clause ignored"_warn_en_US);`。
- **L1192 EN**: Closes the current lexical scope or compound statement.
  **L1192 CN**: 结束当前词法作用域或复合语句块。
- **L1193 EN**: Closes the current lexical scope or compound statement.
  **L1193 CN**: 结束当前词法作用域或复合语句块。
- **L1194 EN**: Blank line separating nearby declarations or logic blocks.
  **L1194 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1195 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::AccClause::If &x) {`.
  **L1195 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::AccClause::If &x) {`。
- **L1196 EN**: Executes a call or declaration centered on `CheckAllowed`.
  **L1196 CN**: 执行以 `CheckAllowed` 为核心的调用或声明。
- **L1197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1198 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1198 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1199 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1199 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1200 EN**: Starts a function, method, lambda, or structured scope: `type->category() == TypeCategory::Logical) {`.
  **L1200 CN**: 开始一个函数、方法、lambda 或结构化作用域：`type->category() == TypeCategory::Logical) {`。

### Lines 1201-1224

````cpp
        return; // LOGICAL and INTEGER type supported for the if clause.
      }
    }
  }
  context_.Say(
      GetContext().clauseSource, "Must have LOGICAL or INTEGER type"_err_en_US);
}

void AccStructureChecker::Enter(const parser::OpenACCEndConstruct &x) {
  context_.Warn(common::UsageWarning::OpenAccUsage, x.source,
      "Misplaced OpenACC end directive"_warn_en_US);
}

void AccStructureChecker::Enter(const parser::Module &) {
  declareSymbols.clear();
}

void AccStructureChecker::Enter(const parser::FunctionSubprogram &x) {
  declareSymbols.clear();
}

void AccStructureChecker::Enter(const parser::SubroutineSubprogram &) {
  declareSymbols.clear();
}
````
- **L1201 EN**: Returns from the current function with `; // LOGICAL and INTEGER type supported for the if clause.`.
  **L1201 CN**: 以 `; // LOGICAL and INTEGER type supported for the if clause.` 从当前函数返回。
- **L1202 EN**: Closes the current lexical scope or compound statement.
  **L1202 CN**: 结束当前词法作用域或复合语句块。
- **L1203 EN**: Closes the current lexical scope or compound statement.
  **L1203 CN**: 结束当前词法作用域或复合语句块。
- **L1204 EN**: Closes the current lexical scope or compound statement.
  **L1204 CN**: 结束当前词法作用域或复合语句块。
- **L1205 EN**: Continues logic associated with callable symbol `Say`.
  **L1205 CN**: 继续与可调用符号 `Say` 相关的逻辑。
- **L1206 EN**: Executes a call or declaration centered on `GetContext`.
  **L1206 CN**: 执行以 `GetContext` 为核心的调用或声明。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1209 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::OpenACCEndConstruct &x) {`.
  **L1209 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::OpenACCEndConstruct &x) {`。
- **L1210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context_.Warn(common::UsageWarning::OpenAccUsage, x.source,`.
  **L1210 CN**: 继续一个多行参数列表、初始化器或聚合项：`context_.Warn(common::UsageWarning::OpenAccUsage, x.source,`。
- **L1211 EN**: Executes a standalone statement or declaration: `"Misplaced OpenACC end directive"_warn_en_US);`.
  **L1211 CN**: 执行一条独立语句或声明：`"Misplaced OpenACC end directive"_warn_en_US);`。
- **L1212 EN**: Closes the current lexical scope or compound statement.
  **L1212 CN**: 结束当前词法作用域或复合语句块。
- **L1213 EN**: Blank line separating nearby declarations or logic blocks.
  **L1213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1214 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::Module &) {`.
  **L1214 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::Module &) {`。
- **L1215 EN**: Executes a call or declaration centered on `declareSymbols.clear`.
  **L1215 CN**: 执行以 `declareSymbols.clear` 为核心的调用或声明。
- **L1216 EN**: Closes the current lexical scope or compound statement.
  **L1216 CN**: 结束当前词法作用域或复合语句块。
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1218 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::FunctionSubprogram &x) {`.
  **L1218 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::FunctionSubprogram &x) {`。
- **L1219 EN**: Executes a call or declaration centered on `declareSymbols.clear`.
  **L1219 CN**: 执行以 `declareSymbols.clear` 为核心的调用或声明。
- **L1220 EN**: Closes the current lexical scope or compound statement.
  **L1220 CN**: 结束当前词法作用域或复合语句块。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1222 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::SubroutineSubprogram &) {`.
  **L1222 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::SubroutineSubprogram &) {`。
- **L1223 EN**: Executes a call or declaration centered on `declareSymbols.clear`.
  **L1223 CN**: 执行以 `declareSymbols.clear` 为核心的调用或声明。
- **L1224 EN**: Closes the current lexical scope or compound statement.
  **L1224 CN**: 结束当前词法作用域或复合语句块。

### Lines 1225-1247

````cpp

void AccStructureChecker::Enter(const parser::SeparateModuleSubprogram &) {
  declareSymbols.clear();
}

void AccStructureChecker::Enter(const parser::DoConstruct &) {
  ++loopNestLevel;
}

void AccStructureChecker::Leave(const parser::DoConstruct &) {
  --loopNestLevel;
}

llvm::StringRef AccStructureChecker::getDirectiveName(
    llvm::acc::Directive directive) {
  return llvm::acc::getOpenACCDirectiveName(directive);
}

llvm::StringRef AccStructureChecker::getClauseName(llvm::acc::Clause clause) {
  return llvm::acc::getOpenACCClauseName(clause);
}

} // namespace Fortran::semantics
````
- **L1225 EN**: Blank line separating nearby declarations or logic blocks.
  **L1225 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1226 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::SeparateModuleSubprogram &) {`.
  **L1226 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::SeparateModuleSubprogram &) {`。
- **L1227 EN**: Executes a call or declaration centered on `declareSymbols.clear`.
  **L1227 CN**: 执行以 `declareSymbols.clear` 为核心的调用或声明。
- **L1228 EN**: Closes the current lexical scope or compound statement.
  **L1228 CN**: 结束当前词法作用域或复合语句块。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1230 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Enter(const parser::DoConstruct &) {`.
  **L1230 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Enter(const parser::DoConstruct &) {`。
- **L1231 EN**: Executes a standalone statement or declaration: `++loopNestLevel;`.
  **L1231 CN**: 执行一条独立语句或声明：`++loopNestLevel;`。
- **L1232 EN**: Closes the current lexical scope or compound statement.
  **L1232 CN**: 结束当前词法作用域或复合语句块。
- **L1233 EN**: Blank line separating nearby declarations or logic blocks.
  **L1233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1234 EN**: Starts a function, method, lambda, or structured scope: `void AccStructureChecker::Leave(const parser::DoConstruct &) {`.
  **L1234 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void AccStructureChecker::Leave(const parser::DoConstruct &) {`。
- **L1235 EN**: Executes a standalone statement or declaration: `--loopNestLevel;`.
  **L1235 CN**: 执行一条独立语句或声明：`--loopNestLevel;`。
- **L1236 EN**: Closes the current lexical scope or compound statement.
  **L1236 CN**: 结束当前词法作用域或复合语句块。
- **L1237 EN**: Blank line separating nearby declarations or logic blocks.
  **L1237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1238 EN**: Continues logic associated with callable symbol `getDirectiveName`.
  **L1238 CN**: 继续与可调用符号 `getDirectiveName` 相关的逻辑。
- **L1239 EN**: Continues the surrounding expression or declaration: `llvm::acc::Directive directive) {`.
  **L1239 CN**: 继续构造周围的表达式或声明：`llvm::acc::Directive directive) {`。
- **L1240 EN**: Returns from the current function with `llvm::acc::getOpenACCDirectiveName(directive)`.
  **L1240 CN**: 以 `llvm::acc::getOpenACCDirectiveName(directive)` 从当前函数返回。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Blank line separating nearby declarations or logic blocks.
  **L1242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1243 EN**: Starts a function, method, lambda, or structured scope: `llvm::StringRef AccStructureChecker::getClauseName(llvm::acc::Clause clause) {`.
  **L1243 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::StringRef AccStructureChecker::getClauseName(llvm::acc::Clause clause) {`。
- **L1244 EN**: Returns from the current function with `llvm::acc::getOpenACCClauseName(clause)`.
  **L1244 CN**: 以 `llvm::acc::getOpenACCClauseName(clause)` 从当前函数返回。
- **L1245 EN**: Closes the current lexical scope or compound statement.
  **L1245 CN**: 结束当前词法作用域或复合语句块。
- **L1246 EN**: Blank line separating nearby declarations or logic blocks.
  **L1246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1247 EN**: Closes a namespace scope with a trailing comment: `} // namespace Fortran::semantics`.
  **L1247 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace Fortran::semantics`。

## Key Concepts / 关键概念

- **Fortran semantic checking / Fortran 语义检查**
- **Fortran parse tree handling / Fortran 语法树处理**
- **Parser data structures and diagnostics / 解析器数据结构与诊断**
- **Scope and symbol resolution / 作用域与符号解析**
- **Symbol modeling and lookup / 符号建模与查找**
- **Fortran type-spec reasoning / Fortran 类型说明推理**
- **Compile-time evaluation helpers / 编译期求值辅助**
- **OpenACC handling / OpenACC 处理**
- **Rule-checking workflow / 规则检查流程**

## Dependencies / 依赖关系

- `check-acc-structure.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `resolve-names-utils.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Common/enum-set.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Evaluate/tools.h`: Provides Fortran constant-folding and evaluation facilities. / 提供Fortran 常量折叠与求值能力。
- `flang/Parser/parse-tree.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Parser/tools.h`: Provides parse-tree, token, or source representation support. / 提供语法树、词法单元或源码表示支持。
- `flang/Semantics/symbol.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/tools.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Semantics/type.h`: Provides Fortran semantic analysis, symbol, and type information. / 提供Fortran 语义分析、符号与类型信息。
- `flang/Support/Fortran.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `llvm/Support/AtomicOrdering.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `optional`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
