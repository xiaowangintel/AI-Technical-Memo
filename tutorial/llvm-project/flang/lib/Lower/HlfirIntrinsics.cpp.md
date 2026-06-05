# HlfirIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Lower/HlfirIntrinsics.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements lowering from Fortran semantics into FIR/MLIR for Hlfir Intrinsics.
- **Purpose (CN)**: 实现 Hlfir Intrinsics 相关的从 Fortran 语义到 FIR/MLIR 的 lowering 过程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===-- HlfirIntrinsics.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide/
//
//===----------------------------------------------------------------------===//

#include "flang/Lower/HlfirIntrinsics.h"

#include "flang/Optimizer/Builder/BoxValue.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Builder/HLFIRTools.h"
#include "flang/Optimizer/Builder/IntrinsicCall.h"
#include "flang/Optimizer/Builder/MutableBox.h"
#include "flang/Optimizer/Builder/Todo.h"
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
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, intent, or metadata: `Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`.
  **L9 CN**: 注释说明附近代码的逻辑、意图或元数据：`Coding style: https://mlir.llvm.org/getting_started/DeveloperGuide`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes "flang/Lower/HlfirIntrinsics.h" to access lowering utilities that map semantics into FIR/MLIR.
  **L13 CN**: 引入 "flang/Lower/HlfirIntrinsics.h" 以使用将语义映射到 FIR/MLIR 的 lowering 工具。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes "flang/Optimizer/Builder/BoxValue.h" to access FIR builder helpers and runtime-construction utilities.
  **L15 CN**: 引入 "flang/Optimizer/Builder/BoxValue.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L16 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L16 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L17 EN**: Includes "flang/Optimizer/Builder/HLFIRTools.h" to access FIR builder helpers and runtime-construction utilities.
  **L17 CN**: 引入 "flang/Optimizer/Builder/HLFIRTools.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L18 EN**: Includes "flang/Optimizer/Builder/IntrinsicCall.h" to access FIR builder helpers and runtime-construction utilities.
  **L18 CN**: 引入 "flang/Optimizer/Builder/IntrinsicCall.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L19 EN**: Includes "flang/Optimizer/Builder/MutableBox.h" to access FIR builder helpers and runtime-construction utilities.
  **L19 CN**: 引入 "flang/Optimizer/Builder/MutableBox.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L20 EN**: Includes "flang/Optimizer/Builder/Todo.h" to access FIR builder helpers and runtime-construction utilities.
  **L20 CN**: 引入 "flang/Optimizer/Builder/Todo.h" 以使用FIR Builder 辅助工具与运行时构造能力。

### Lines 21-40

````cpp
#include "flang/Optimizer/Dialect/FIRType.h"
#include "flang/Optimizer/HLFIR/HLFIRDialect.h"
#include "flang/Optimizer/HLFIR/HLFIROps.h"
#include "mlir/IR/Value.h"
#include "llvm/ADT/SmallVector.h"
#include <mlir/IR/ValueRange.h>

namespace {

class HlfirTransformationalIntrinsic {
public:
  explicit HlfirTransformationalIntrinsic(fir::FirOpBuilder &builder,
                                          mlir::Location loc)
      : builder(builder), loc(loc) {}

  virtual ~HlfirTransformationalIntrinsic() = default;

  hlfir::EntityWithAttributes
  lower(const Fortran::lower::PreparedActualArguments &loweredActuals,
        const fir::IntrinsicArgumentLoweringRules *argLowering,
````
- **L21 EN**: Includes "flang/Optimizer/Dialect/FIRType.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L21 CN**: 引入 "flang/Optimizer/Dialect/FIRType.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L22 EN**: Includes "flang/Optimizer/HLFIR/HLFIRDialect.h" to access HLFIR abstractions and transformation support.
  **L22 CN**: 引入 "flang/Optimizer/HLFIR/HLFIRDialect.h" 以使用HLFIR 抽象与变换支持。
- **L23 EN**: Includes "flang/Optimizer/HLFIR/HLFIROps.h" to access HLFIR abstractions and transformation support.
  **L23 CN**: 引入 "flang/Optimizer/HLFIR/HLFIROps.h" 以使用HLFIR 抽象与变换支持。
- **L24 EN**: Includes "mlir/IR/Value.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L24 CN**: 引入 "mlir/IR/Value.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L25 EN**: Includes "llvm/ADT/SmallVector.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L25 CN**: 引入 "llvm/ADT/SmallVector.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L26 EN**: Includes <mlir/IR/ValueRange.h> to access MLIR core IR, pass, or dialect infrastructure.
  **L26 CN**: 引入 <mlir/IR/ValueRange.h> 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Opens namespace scope ``.
  **L28 CN**: 打开命名空间作用域 ``。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Declares class `HlfirTransformationalIntrinsic`.
  **L30 CN**: 声明 class `HlfirTransformationalIntrinsic`。
- **L31 EN**: Sets the following members to `public` access.
  **L31 CN**: 将后续成员的访问级别设为 `public`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `explicit HlfirTransformationalIntrinsic(fir::FirOpBuilder &builder,`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`explicit HlfirTransformationalIntrinsic(fir::FirOpBuilder &builder,`。
- **L33 EN**: Continues the surrounding expression or declaration: `mlir::Location loc)`.
  **L33 CN**: 继续构造周围的表达式或声明：`mlir::Location loc)`。
- **L34 EN**: Continues logic associated with callable symbol `builder`.
  **L34 CN**: 继续与可调用符号 `builder` 相关的逻辑。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Executes a call or declaration centered on `~HlfirTransformationalIntrinsic`.
  **L36 CN**: 执行以 `~HlfirTransformationalIntrinsic` 为核心的调用或声明。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Continues the surrounding expression or declaration: `hlfir::EntityWithAttributes`.
  **L38 CN**: 继续构造周围的表达式或声明：`hlfir::EntityWithAttributes`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lower(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`lower(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。

### Lines 41-60

````cpp
        mlir::Type stmtResultType) {
    mlir::Value res = lowerImpl(loweredActuals, argLowering, stmtResultType);
    for (const hlfir::CleanupFunction &fn : cleanupFns)
      fn();
    return {hlfir::EntityWithAttributes{res}};
  }

protected:
  fir::FirOpBuilder &builder;
  mlir::Location loc;
  llvm::SmallVector<hlfir::CleanupFunction, 3> cleanupFns;

  virtual mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) = 0;

  llvm::SmallVector<mlir::Value> getOperandVector(
      const Fortran::lower::PreparedActualArguments &loweredActuals,
      const fir::IntrinsicArgumentLoweringRules *argLowering);
````
- **L41 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L41 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L42 EN**: Initializes variable `res` from the right-hand expression.
  **L42 CN**: 使用右侧表达式初始化变量 `res`。
- **L43 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L43 CN**: 开始 `for` 控制流语句并计算其条件。
- **L44 EN**: Executes a call or declaration centered on `fn`.
  **L44 CN**: 执行以 `fn` 为核心的调用或声明。
- **L45 EN**: Returns from the current function with `{hlfir::EntityWithAttributes{res}}`.
  **L45 CN**: 以 `{hlfir::EntityWithAttributes{res}}` 从当前函数返回。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Sets the following members to `protected` access.
  **L48 CN**: 将后续成员的访问级别设为 `protected`。
- **L49 EN**: Executes a standalone statement or declaration: `fir::FirOpBuilder &builder;`.
  **L49 CN**: 执行一条独立语句或声明：`fir::FirOpBuilder &builder;`。
- **L50 EN**: Executes a standalone statement or declaration: `mlir::Location loc;`.
  **L50 CN**: 执行一条独立语句或声明：`mlir::Location loc;`。
- **L51 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<hlfir::CleanupFunction, 3> cleanupFns;`.
  **L51 CN**: 执行一条独立语句或声明：`llvm::SmallVector<hlfir::CleanupFunction, 3> cleanupFns;`。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Continues the surrounding expression or declaration: `virtual mlir::Value`.
  **L53 CN**: 继续构造周围的表达式或声明：`virtual mlir::Value`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L56 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) = 0;`.
  **L56 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) = 0;`。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L58 EN**: Continues logic associated with callable symbol `getOperandVector`.
  **L58 CN**: 继续与可调用符号 `getOperandVector` 相关的逻辑。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L60 EN**: Executes a standalone statement or declaration: `const fir::IntrinsicArgumentLoweringRules *argLowering);`.
  **L60 CN**: 执行一条独立语句或声明：`const fir::IntrinsicArgumentLoweringRules *argLowering);`。

### Lines 61-80

````cpp

  mlir::Type computeResultType(mlir::Value argArray, mlir::Type stmtResultType);

  template <typename OP, typename... BUILD_ARGS>
  inline OP createOp(BUILD_ARGS... args) {
    return OP::create(builder, loc, args...);
  }

  mlir::Value loadBoxAddress(
      const std::optional<Fortran::lower::PreparedActualArgument> &arg);

  mlir::Value
  loadTrivialScalar(const Fortran::lower::PreparedActualArgument &arg);

  mlir::Value loadOptionalValue(Fortran::lower::PreparedActualArgument &arg);

  void addCleanup(std::optional<hlfir::CleanupFunction> cleanup) {
    if (cleanup)
      cleanupFns.emplace_back(std::move(*cleanup));
  }
````
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Executes a call or declaration centered on `computeResultType`.
  **L62 CN**: 执行以 `computeResultType` 为核心的调用或声明。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L64 EN**: Introduces template parameters or specialization context: `template <typename OP, typename... BUILD_ARGS>`.
  **L64 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP, typename... BUILD_ARGS>`。
- **L65 EN**: Starts a function, method, lambda, or structured scope: `inline OP createOp(BUILD_ARGS... args) {`.
  **L65 CN**: 开始一个函数、方法、lambda 或结构化作用域：`inline OP createOp(BUILD_ARGS... args) {`。
- **L66 EN**: Returns from the current function with `OP::create(builder, loc, args...)`.
  **L66 CN**: 以 `OP::create(builder, loc, args...)` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Continues logic associated with callable symbol `loadBoxAddress`.
  **L69 CN**: 继续与可调用符号 `loadBoxAddress` 相关的逻辑。
- **L70 EN**: Executes a standalone statement or declaration: `const std::optional<Fortran::lower::PreparedActualArgument> &arg);`.
  **L70 CN**: 执行一条独立语句或声明：`const std::optional<Fortran::lower::PreparedActualArgument> &arg);`。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L72 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L72 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L73 EN**: Executes a call or declaration centered on `loadTrivialScalar`.
  **L73 CN**: 执行以 `loadTrivialScalar` 为核心的调用或声明。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Executes a call or declaration centered on `loadOptionalValue`.
  **L75 CN**: 执行以 `loadOptionalValue` 为核心的调用或声明。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Starts a function, method, lambda, or structured scope: `void addCleanup(std::optional<hlfir::CleanupFunction> cleanup) {`.
  **L77 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void addCleanup(std::optional<hlfir::CleanupFunction> cleanup) {`。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Executes a call or declaration centered on `cleanupFns.emplace_back`.
  **L79 CN**: 执行以 `cleanupFns.emplace_back` 为核心的调用或声明。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。

### Lines 81-100

````cpp
};

template <typename OP, bool HAS_MASK>
class HlfirReductionIntrinsic : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};
using HlfirSumLowering = HlfirReductionIntrinsic<hlfir::SumOp, true>;
using HlfirProductLowering = HlfirReductionIntrinsic<hlfir::ProductOp, true>;
using HlfirMaxvalLowering = HlfirReductionIntrinsic<hlfir::MaxvalOp, true>;
using HlfirMinvalLowering = HlfirReductionIntrinsic<hlfir::MinvalOp, true>;
using HlfirAnyLowering = HlfirReductionIntrinsic<hlfir::AnyOp, false>;
using HlfirAllLowering = HlfirReductionIntrinsic<hlfir::AllOp, false>;

````
- **L81 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L81 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Introduces template parameters or specialization context: `template <typename OP, bool HAS_MASK>`.
  **L83 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP, bool HAS_MASK>`。
- **L84 EN**: Declares class `HlfirReductionIntrinsic`.
  **L84 CN**: 声明 class `HlfirReductionIntrinsic`。
- **L85 EN**: Sets the following members to `public` access.
  **L85 CN**: 将后续成员的访问级别设为 `public`。
- **L86 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L86 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Sets the following members to `protected` access.
  **L88 CN**: 将后续成员的访问级别设为 `protected`。
- **L89 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L89 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L92 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L92 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L93 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L93 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L94 EN**: Defines alias `HlfirSumLowering` to simplify later code.
  **L94 CN**: 定义别名 `HlfirSumLowering` 以简化后续代码。
- **L95 EN**: Defines alias `HlfirProductLowering` to simplify later code.
  **L95 CN**: 定义别名 `HlfirProductLowering` 以简化后续代码。
- **L96 EN**: Defines alias `HlfirMaxvalLowering` to simplify later code.
  **L96 CN**: 定义别名 `HlfirMaxvalLowering` 以简化后续代码。
- **L97 EN**: Defines alias `HlfirMinvalLowering` to simplify later code.
  **L97 CN**: 定义别名 `HlfirMinvalLowering` 以简化后续代码。
- **L98 EN**: Defines alias `HlfirAnyLowering` to simplify later code.
  **L98 CN**: 定义别名 `HlfirAnyLowering` 以简化后续代码。
- **L99 EN**: Defines alias `HlfirAllLowering` to simplify later code.
  **L99 CN**: 定义别名 `HlfirAllLowering` 以简化后续代码。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 101-120

````cpp
template <typename OP>
class HlfirMinMaxLocIntrinsic : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};
using HlfirMinlocLowering = HlfirMinMaxLocIntrinsic<hlfir::MinlocOp>;
using HlfirMaxlocLowering = HlfirMinMaxLocIntrinsic<hlfir::MaxlocOp>;

template <typename OP>
class HlfirProductIntrinsic : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
````
- **L101 EN**: Introduces template parameters or specialization context: `template <typename OP>`.
  **L101 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP>`。
- **L102 EN**: Declares class `HlfirMinMaxLocIntrinsic`.
  **L102 CN**: 声明 class `HlfirMinMaxLocIntrinsic`。
- **L103 EN**: Sets the following members to `public` access.
  **L103 CN**: 将后续成员的访问级别设为 `public`。
- **L104 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L104 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L106 EN**: Sets the following members to `protected` access.
  **L106 CN**: 将后续成员的访问级别设为 `protected`。
- **L107 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L107 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L110 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L110 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L111 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L111 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L112 EN**: Defines alias `HlfirMinlocLowering` to simplify later code.
  **L112 CN**: 定义别名 `HlfirMinlocLowering` 以简化后续代码。
- **L113 EN**: Defines alias `HlfirMaxlocLowering` to simplify later code.
  **L113 CN**: 定义别名 `HlfirMaxlocLowering` 以简化后续代码。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L115 EN**: Introduces template parameters or specialization context: `template <typename OP>`.
  **L115 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP>`。
- **L116 EN**: Declares class `HlfirProductIntrinsic`.
  **L116 CN**: 声明 class `HlfirProductIntrinsic`。
- **L117 EN**: Sets the following members to `public` access.
  **L117 CN**: 将后续成员的访问级别设为 `public`。
- **L118 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L118 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L120 EN**: Sets the following members to `protected` access.
  **L120 CN**: 将后续成员的访问级别设为 `protected`。

### Lines 121-140

````cpp
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};
using HlfirMatmulLowering = HlfirProductIntrinsic<hlfir::MatmulOp>;
using HlfirDotProductLowering = HlfirProductIntrinsic<hlfir::DotProductOp>;

class HlfirTransposeLowering : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

class HlfirCountLowering : public HlfirTransformationalIntrinsic {
````
- **L121 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L121 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L124 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L124 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L125 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L125 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L126 EN**: Defines alias `HlfirMatmulLowering` to simplify later code.
  **L126 CN**: 定义别名 `HlfirMatmulLowering` 以简化后续代码。
- **L127 EN**: Defines alias `HlfirDotProductLowering` to simplify later code.
  **L127 CN**: 定义别名 `HlfirDotProductLowering` 以简化后续代码。
- **L128 EN**: Blank line separating nearby declarations or logic blocks.
  **L128 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L129 EN**: Declares class `HlfirTransposeLowering`.
  **L129 CN**: 声明 class `HlfirTransposeLowering`。
- **L130 EN**: Sets the following members to `public` access.
  **L130 CN**: 将后续成员的访问级别设为 `public`。
- **L131 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L131 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Sets the following members to `protected` access.
  **L133 CN**: 将后续成员的访问级别设为 `protected`。
- **L134 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L134 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L137 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L137 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L138 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L138 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L139 EN**: Blank line separating nearby declarations or logic blocks.
  **L139 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L140 EN**: Declares class `HlfirCountLowering`.
  **L140 CN**: 声明 class `HlfirCountLowering`。

### Lines 141-160

````cpp
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

class HlfirCharExtremumLowering : public HlfirTransformationalIntrinsic {
public:
  HlfirCharExtremumLowering(fir::FirOpBuilder &builder, mlir::Location loc,
                            hlfir::CharExtremumPredicate pred)
      : HlfirTransformationalIntrinsic(builder, loc), pred{pred} {}

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
````
- **L141 EN**: Sets the following members to `public` access.
  **L141 CN**: 将后续成员的访问级别设为 `public`。
- **L142 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L142 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Sets the following members to `protected` access.
  **L144 CN**: 将后续成员的访问级别设为 `protected`。
- **L145 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L145 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L148 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L148 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L149 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L149 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L151 EN**: Declares class `HlfirCharExtremumLowering`.
  **L151 CN**: 声明 class `HlfirCharExtremumLowering`。
- **L152 EN**: Sets the following members to `public` access.
  **L152 CN**: 将后续成员的访问级别设为 `public`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HlfirCharExtremumLowering(fir::FirOpBuilder &builder, mlir::Location loc,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`HlfirCharExtremumLowering(fir::FirOpBuilder &builder, mlir::Location loc,`。
- **L154 EN**: Continues the surrounding expression or declaration: `hlfir::CharExtremumPredicate pred)`.
  **L154 CN**: 继续构造周围的表达式或声明：`hlfir::CharExtremumPredicate pred)`。
- **L155 EN**: Continues logic associated with callable symbol `HlfirTransformationalIntrinsic`.
  **L155 CN**: 继续与可调用符号 `HlfirTransformationalIntrinsic` 相关的逻辑。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L157 EN**: Sets the following members to `protected` access.
  **L157 CN**: 将后续成员的访问级别设为 `protected`。
- **L158 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L158 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。

### Lines 161-180

````cpp
            mlir::Type stmtResultType) override;

protected:
  hlfir::CharExtremumPredicate pred;
};

class HlfirCharTrimLowering : public HlfirTransformationalIntrinsic {
public:
  HlfirCharTrimLowering(fir::FirOpBuilder &builder, mlir::Location loc)
      : HlfirTransformationalIntrinsic(builder, loc) {}

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

class HlfirCShiftLowering : public HlfirTransformationalIntrinsic {
public:
````
- **L161 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L161 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L163 EN**: Sets the following members to `protected` access.
  **L163 CN**: 将后续成员的访问级别设为 `protected`。
- **L164 EN**: Executes a standalone statement or declaration: `hlfir::CharExtremumPredicate pred;`.
  **L164 CN**: 执行一条独立语句或声明：`hlfir::CharExtremumPredicate pred;`。
- **L165 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L165 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L167 EN**: Declares class `HlfirCharTrimLowering`.
  **L167 CN**: 声明 class `HlfirCharTrimLowering`。
- **L168 EN**: Sets the following members to `public` access.
  **L168 CN**: 将后续成员的访问级别设为 `public`。
- **L169 EN**: Continues logic associated with callable symbol `HlfirCharTrimLowering`.
  **L169 CN**: 继续与可调用符号 `HlfirCharTrimLowering` 相关的逻辑。
- **L170 EN**: Continues logic associated with callable symbol `HlfirTransformationalIntrinsic`.
  **L170 CN**: 继续与可调用符号 `HlfirTransformationalIntrinsic` 相关的逻辑。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L172 EN**: Sets the following members to `protected` access.
  **L172 CN**: 将后续成员的访问级别设为 `protected`。
- **L173 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L173 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L176 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L176 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L177 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L177 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Declares class `HlfirCShiftLowering`.
  **L179 CN**: 声明 class `HlfirCShiftLowering`。
- **L180 EN**: Sets the following members to `public` access.
  **L180 CN**: 将后续成员的访问级别设为 `public`。

### Lines 181-200

````cpp
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

class HlfirEOShiftLowering : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

````
- **L181 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L181 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L183 EN**: Sets the following members to `protected` access.
  **L183 CN**: 将后续成员的访问级别设为 `protected`。
- **L184 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L184 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L187 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L187 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L188 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L188 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Declares class `HlfirEOShiftLowering`.
  **L190 CN**: 声明 class `HlfirEOShiftLowering`。
- **L191 EN**: Sets the following members to `public` access.
  **L191 CN**: 将后续成员的访问级别设为 `public`。
- **L192 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L192 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L194 EN**: Sets the following members to `protected` access.
  **L194 CN**: 将后续成员的访问级别设为 `protected`。
- **L195 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L195 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L198 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L198 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L199 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L199 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 201-220

````cpp
class HlfirReshapeLowering : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

class HlfirIndexLowering : public HlfirTransformationalIntrinsic {
public:
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;

protected:
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
````
- **L201 EN**: Declares class `HlfirReshapeLowering`.
  **L201 CN**: 声明 class `HlfirReshapeLowering`。
- **L202 EN**: Sets the following members to `public` access.
  **L202 CN**: 将后续成员的访问级别设为 `public`。
- **L203 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L203 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L204 EN**: Blank line separating nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Sets the following members to `protected` access.
  **L205 CN**: 将后续成员的访问级别设为 `protected`。
- **L206 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L206 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L209 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L209 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L210 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L210 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L212 EN**: Declares class `HlfirIndexLowering`.
  **L212 CN**: 声明 class `HlfirIndexLowering`。
- **L213 EN**: Sets the following members to `public` access.
  **L213 CN**: 将后续成员的访问级别设为 `public`。
- **L214 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L214 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Sets the following members to `protected` access.
  **L216 CN**: 将后续成员的访问级别设为 `protected`。
- **L217 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L217 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L220 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L220 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。

### Lines 221-240

````cpp
};

struct HlfirLenLowering : public HlfirTransformationalIntrinsic {
  using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;
  mlir::Value
  lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,
            const fir::IntrinsicArgumentLoweringRules *argLowering,
            mlir::Type stmtResultType) override;
};

} // namespace

mlir::Value HlfirTransformationalIntrinsic::loadBoxAddress(
    const std::optional<Fortran::lower::PreparedActualArgument> &arg) {
  if (!arg)
    return mlir::Value{};

  hlfir::Entity actual = arg->getActual(loc, builder);

  if (!arg->handleDynamicOptional()) {
````
- **L221 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L221 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L223 EN**: Declares struct `HlfirLenLowering`.
  **L223 CN**: 声明 struct `HlfirLenLowering`。
- **L224 EN**: Executes a standalone statement or declaration: `using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`.
  **L224 CN**: 执行一条独立语句或声明：`using HlfirTransformationalIntrinsic::HlfirTransformationalIntrinsic;`。
- **L225 EN**: Continues the surrounding expression or declaration: `mlir::Value`.
  **L225 CN**: 继续构造周围的表达式或声明：`mlir::Value`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`lowerImpl(const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L228 EN**: Executes a standalone statement or declaration: `mlir::Type stmtResultType) override;`.
  **L228 CN**: 执行一条独立语句或声明：`mlir::Type stmtResultType) override;`。
- **L229 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L229 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L231 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L233 EN**: Continues logic associated with callable symbol `loadBoxAddress`.
  **L233 CN**: 继续与可调用符号 `loadBoxAddress` 相关的逻辑。
- **L234 EN**: Continues the surrounding expression or declaration: `const std::optional<Fortran::lower::PreparedActualArgument> &arg) {`.
  **L234 CN**: 继续构造周围的表达式或声明：`const std::optional<Fortran::lower::PreparedActualArgument> &arg) {`。
- **L235 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L235 CN**: 开始 `if` 控制流语句并计算其条件。
- **L236 EN**: Returns from the current function with `mlir::Value{}`.
  **L236 CN**: 以 `mlir::Value{}` 从当前函数返回。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L238 EN**: Initializes variable `actual` from the right-hand expression.
  **L238 CN**: 使用右侧表达式初始化变量 `actual`。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L240 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L240 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 241-260

````cpp
    if (actual.isMutableBox()) {
      // this is a box address type but is not dynamically optional. Just load
      // the box, assuming it is well formed (!fir.ref<!fir.box<...>> ->
      // !fir.box<...>)
      return fir::LoadOp::create(builder, loc, actual.getBase());
    }
    return actual;
  }

  auto [exv, cleanup] = hlfir::translateToExtendedValue(loc, builder, actual);
  addCleanup(cleanup);

  mlir::Value isPresent = arg->getIsPresent();
  // createBox will not do create any invalid memory dereferences if exv is
  // absent. The created fir.box will not be usable, but the SelectOp below
  // ensures it won't be.
  mlir::Value box = builder.createBox(loc, exv);
  mlir::Type boxType = box.getType();
  auto absent = fir::AbsentOp::create(builder, loc, boxType);
  auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,
````
- **L241 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L241 CN**: 开始 `if` 控制流语句并计算其条件。
- **L242 EN**: Comment explains nearby logic, intent, or metadata: `this is a box address type but is not dynamically optional. Just load`.
  **L242 CN**: 注释说明附近代码的逻辑、意图或元数据：`this is a box address type but is not dynamically optional. Just load`。
- **L243 EN**: Comment explains nearby logic, intent, or metadata: `the box, assuming it is well formed (!fir.ref<!fir.box<...>> ->`.
  **L243 CN**: 注释说明附近代码的逻辑、意图或元数据：`the box, assuming it is well formed (!fir.ref<!fir.box<...>> ->`。
- **L244 EN**: Comment explains nearby logic, intent, or metadata: `fir.box<...>)`.
  **L244 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.box<...>)`。
- **L245 EN**: Returns from the current function with `fir::LoadOp::create(builder, loc, actual.getBase())`.
  **L245 CN**: 以 `fir::LoadOp::create(builder, loc, actual.getBase())` 从当前函数返回。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Returns from the current function with `actual`.
  **L247 CN**: 以 `actual` 从当前函数返回。
- **L248 EN**: Closes the current lexical scope or compound statement.
  **L248 CN**: 结束当前词法作用域或复合语句块。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Executes a call or declaration centered on `hlfir::translateToExtendedValue`.
  **L250 CN**: 执行以 `hlfir::translateToExtendedValue` 为核心的调用或声明。
- **L251 EN**: Executes a call or declaration centered on `addCleanup`.
  **L251 CN**: 执行以 `addCleanup` 为核心的调用或声明。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L253 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L253 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L254 EN**: Comment explains nearby logic, intent, or metadata: `createBox will not do create any invalid memory dereferences if exv is`.
  **L254 CN**: 注释说明附近代码的逻辑、意图或元数据：`createBox will not do create any invalid memory dereferences if exv is`。
- **L255 EN**: Comment explains nearby logic, intent, or metadata: `absent. The created fir.box will not be usable, but the SelectOp below`.
  **L255 CN**: 注释说明附近代码的逻辑、意图或元数据：`absent. The created fir.box will not be usable, but the SelectOp below`。
- **L256 EN**: Comment explains nearby logic, intent, or metadata: `ensures it won't be.`.
  **L256 CN**: 注释说明附近代码的逻辑、意图或元数据：`ensures it won't be.`。
- **L257 EN**: Initializes variable `box` from the right-hand expression.
  **L257 CN**: 使用右侧表达式初始化变量 `box`。
- **L258 EN**: Initializes variable `boxType` from the right-hand expression.
  **L258 CN**: 使用右侧表达式初始化变量 `boxType`。
- **L259 EN**: Initializes variable `absent` from the right-hand expression.
  **L259 CN**: 使用右侧表达式初始化变量 `absent`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto boxOrAbsent = mlir::arith::SelectOp::create(builder, loc, boxType,`。

### Lines 261-280

````cpp
                                                   isPresent, box, absent);

  return boxOrAbsent;
}

mlir::Value HlfirTransformationalIntrinsic::loadOptionalValue(
    Fortran::lower::PreparedActualArgument &arg) {
  mlir::Type eleType = arg.getFortranElementType();

  // For an elemental call, getActual() may produce
  // a designator denoting the array element to be passed
  // to the subprogram. If the actual array is dynamically
  // optional the designator must be generated under
  // isPresent check (see also genIntrinsicRefCore).
  return builder
      .genIfOp(loc, {eleType}, arg.getIsPresent(),
               /*withElseRegion=*/true)
      .genThen([&]() {
        hlfir::Entity actual = arg.getActual(loc, builder);
        assert(eleType == actual.getFortranElementType() &&
````
- **L261 EN**: Executes a standalone statement or declaration: `isPresent, box, absent);`.
  **L261 CN**: 执行一条独立语句或声明：`isPresent, box, absent);`。
- **L262 EN**: Blank line separating nearby declarations or logic blocks.
  **L262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L263 EN**: Returns from the current function with `boxOrAbsent`.
  **L263 CN**: 以 `boxOrAbsent` 从当前函数返回。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Continues logic associated with callable symbol `loadOptionalValue`.
  **L266 CN**: 继续与可调用符号 `loadOptionalValue` 相关的逻辑。
- **L267 EN**: Continues the surrounding expression or declaration: `Fortran::lower::PreparedActualArgument &arg) {`.
  **L267 CN**: 继续构造周围的表达式或声明：`Fortran::lower::PreparedActualArgument &arg) {`。
- **L268 EN**: Initializes variable `eleType` from the right-hand expression.
  **L268 CN**: 使用右侧表达式初始化变量 `eleType`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L270 EN**: Comment explains nearby logic, intent, or metadata: `For an elemental call, getActual() may produce`.
  **L270 CN**: 注释说明附近代码的逻辑、意图或元数据：`For an elemental call, getActual() may produce`。
- **L271 EN**: Comment explains nearby logic, intent, or metadata: `a designator denoting the array element to be passed`.
  **L271 CN**: 注释说明附近代码的逻辑、意图或元数据：`a designator denoting the array element to be passed`。
- **L272 EN**: Comment explains nearby logic, intent, or metadata: `to the subprogram. If the actual array is dynamically`.
  **L272 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the subprogram. If the actual array is dynamically`。
- **L273 EN**: Comment explains nearby logic, intent, or metadata: `optional the designator must be generated under`.
  **L273 CN**: 注释说明附近代码的逻辑、意图或元数据：`optional the designator must be generated under`。
- **L274 EN**: Comment explains nearby logic, intent, or metadata: `isPresent check (see also genIntrinsicRefCore).`.
  **L274 CN**: 注释说明附近代码的逻辑、意图或元数据：`isPresent check (see also genIntrinsicRefCore).`。
- **L275 EN**: Returns from the current function with `builder`.
  **L275 CN**: 以 `builder` 从当前函数返回。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `.genIfOp(loc, {eleType}, arg.getIsPresent(),`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`.genIfOp(loc, {eleType}, arg.getIsPresent(),`。
- **L277 EN**: Comment explains nearby logic, intent, or metadata: `withElseRegion=*/true)`.
  **L277 CN**: 注释说明附近代码的逻辑、意图或元数据：`withElseRegion=*/true)`。
- **L278 EN**: Starts a function, method, lambda, or structured scope: `.genThen([&]() {`.
  **L278 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genThen([&]() {`。
- **L279 EN**: Initializes variable `actual` from the right-hand expression.
  **L279 CN**: 使用右侧表达式初始化变量 `actual`。
- **L280 EN**: Checks an internal invariant in debug builds.
  **L280 CN**: 在调试构建中检查内部不变式。

### Lines 281-300

````cpp
               "result type mismatch in genOptionalValue");
        assert(actual.isScalar() && fir::isa_trivial(eleType) &&
               "must be a numerical or logical scalar");
        hlfir::Entity val = hlfir::loadTrivialScalar(loc, builder, actual);
        fir::ResultOp::create(builder, loc, val);
      })
      .genElse([&]() {
        mlir::Value zero = fir::factory::createZeroValue(builder, loc, eleType);
        fir::ResultOp::create(builder, loc, zero);
      })
      .getResults()[0];
}

mlir::Value HlfirTransformationalIntrinsic::loadTrivialScalar(
    const Fortran::lower::PreparedActualArgument &arg) {
  hlfir::Entity actual = arg.getActual(loc, builder);
  return hlfir::loadTrivialScalar(loc, builder, actual);
}

llvm::SmallVector<mlir::Value> HlfirTransformationalIntrinsic::getOperandVector(
````
- **L281 EN**: Executes a standalone statement or declaration: `"result type mismatch in genOptionalValue");`.
  **L281 CN**: 执行一条独立语句或声明：`"result type mismatch in genOptionalValue");`。
- **L282 EN**: Checks an internal invariant in debug builds.
  **L282 CN**: 在调试构建中检查内部不变式。
- **L283 EN**: Executes a standalone statement or declaration: `"must be a numerical or logical scalar");`.
  **L283 CN**: 执行一条独立语句或声明：`"must be a numerical or logical scalar");`。
- **L284 EN**: Initializes variable `val` from the right-hand expression.
  **L284 CN**: 使用右侧表达式初始化变量 `val`。
- **L285 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L285 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L286 EN**: Continues the surrounding expression or declaration: `})`.
  **L286 CN**: 继续构造周围的表达式或声明：`})`。
- **L287 EN**: Starts a function, method, lambda, or structured scope: `.genElse([&]() {`.
  **L287 CN**: 开始一个函数、方法、lambda 或结构化作用域：`.genElse([&]() {`。
- **L288 EN**: Initializes variable `zero` from the right-hand expression.
  **L288 CN**: 使用右侧表达式初始化变量 `zero`。
- **L289 EN**: Executes a call or declaration centered on `fir::ResultOp::create`.
  **L289 CN**: 执行以 `fir::ResultOp::create` 为核心的调用或声明。
- **L290 EN**: Continues the surrounding expression or declaration: `})`.
  **L290 CN**: 继续构造周围的表达式或声明：`})`。
- **L291 EN**: Executes a call or declaration centered on `.getResults`.
  **L291 CN**: 执行以 `.getResults` 为核心的调用或声明。
- **L292 EN**: Closes the current lexical scope or compound statement.
  **L292 CN**: 结束当前词法作用域或复合语句块。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L294 EN**: Continues logic associated with callable symbol `loadTrivialScalar`.
  **L294 CN**: 继续与可调用符号 `loadTrivialScalar` 相关的逻辑。
- **L295 EN**: Continues the surrounding expression or declaration: `const Fortran::lower::PreparedActualArgument &arg) {`.
  **L295 CN**: 继续构造周围的表达式或声明：`const Fortran::lower::PreparedActualArgument &arg) {`。
- **L296 EN**: Initializes variable `actual` from the right-hand expression.
  **L296 CN**: 使用右侧表达式初始化变量 `actual`。
- **L297 EN**: Returns from the current function with `hlfir::loadTrivialScalar(loc, builder, actual)`.
  **L297 CN**: 以 `hlfir::loadTrivialScalar(loc, builder, actual)` 从当前函数返回。
- **L298 EN**: Closes the current lexical scope or compound statement.
  **L298 CN**: 结束当前词法作用域或复合语句块。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L300 EN**: Continues logic associated with callable symbol `getOperandVector`.
  **L300 CN**: 继续与可调用符号 `getOperandVector` 相关的逻辑。

### Lines 301-320

````cpp
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering) {
  llvm::SmallVector<mlir::Value> operands;
  operands.reserve(loweredActuals.size());

  for (size_t i = 0; i < loweredActuals.size(); ++i) {
    std::optional<Fortran::lower::PreparedActualArgument> arg =
        loweredActuals[i];
    if (!arg) {
      operands.emplace_back();
      continue;
    }
    mlir::Value valArg;
    if (!argLowering) {
      valArg = loadTrivialScalar(*arg);
      operands.emplace_back(valArg);
      continue;
    }
    fir::ArgLoweringRule argRules =
        fir::lowerIntrinsicArgumentAs(*argLowering, i);
````
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L302 EN**: Continues the surrounding expression or declaration: `const fir::IntrinsicArgumentLoweringRules *argLowering) {`.
  **L302 CN**: 继续构造周围的表达式或声明：`const fir::IntrinsicArgumentLoweringRules *argLowering) {`。
- **L303 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> operands;`.
  **L303 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> operands;`。
- **L304 EN**: Executes a call or declaration centered on `operands.reserve`.
  **L304 CN**: 执行以 `operands.reserve` 为核心的调用或声明。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L306 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L306 CN**: 开始 `for` 控制流语句并计算其条件。
- **L307 EN**: Continues the surrounding expression or declaration: `std::optional<Fortran::lower::PreparedActualArgument> arg =`.
  **L307 CN**: 继续构造周围的表达式或声明：`std::optional<Fortran::lower::PreparedActualArgument> arg =`。
- **L308 EN**: Executes a standalone statement or declaration: `loweredActuals[i];`.
  **L308 CN**: 执行一条独立语句或声明：`loweredActuals[i];`。
- **L309 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `if` 控制流语句并计算其条件。
- **L310 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L310 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L311 EN**: Skips to the next loop iteration.
  **L311 CN**: 跳到下一次循环迭代。
- **L312 EN**: Closes the current lexical scope or compound statement.
  **L312 CN**: 结束当前词法作用域或复合语句块。
- **L313 EN**: Executes a standalone statement or declaration: `mlir::Value valArg;`.
  **L313 CN**: 执行一条独立语句或声明：`mlir::Value valArg;`。
- **L314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L315 EN**: Executes a call or declaration centered on `loadTrivialScalar`.
  **L315 CN**: 执行以 `loadTrivialScalar` 为核心的调用或声明。
- **L316 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L316 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。
- **L317 EN**: Skips to the next loop iteration.
  **L317 CN**: 跳到下一次循环迭代。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Continues the surrounding expression or declaration: `fir::ArgLoweringRule argRules =`.
  **L319 CN**: 继续构造周围的表达式或声明：`fir::ArgLoweringRule argRules =`。
- **L320 EN**: Executes a call or declaration centered on `fir::lowerIntrinsicArgumentAs`.
  **L320 CN**: 执行以 `fir::lowerIntrinsicArgumentAs` 为核心的调用或声明。

### Lines 321-340

````cpp
    if (argRules.lowerAs == fir::LowerIntrinsicArgAs::Box) {
      valArg = loadBoxAddress(arg);
    } else if (argRules.handleDynamicOptional) {
      if (argRules.lowerAs == fir::LowerIntrinsicArgAs::Value) {
        if (arg->handleDynamicOptional())
          valArg = loadOptionalValue(*arg);
        else
          valArg = loadTrivialScalar(*arg);
      } else {
        TODO(loc, "hlfir transformational intrinsic dynamically optional "
                  "argument without box lowering");
      }
    } else {
      hlfir::Entity actual = arg->getActual(loc, builder);
      if (argRules.lowerAs != fir::LowerIntrinsicArgAs::Inquired)
        valArg = hlfir::derefPointersAndAllocatables(loc, builder, actual);
      else
        valArg = actual.getBase();
    }
    operands.emplace_back(valArg);
````
- **L321 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L321 CN**: 开始 `if` 控制流语句并计算其条件。
- **L322 EN**: Executes a call or declaration centered on `loadBoxAddress`.
  **L322 CN**: 执行以 `loadBoxAddress` 为核心的调用或声明。
- **L323 EN**: Transitions from the previous branch into an `else if` condition.
  **L323 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L324 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L324 CN**: 开始 `if` 控制流语句并计算其条件。
- **L325 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L325 CN**: 开始 `if` 控制流语句并计算其条件。
- **L326 EN**: Executes a call or declaration centered on `loadOptionalValue`.
  **L326 CN**: 执行以 `loadOptionalValue` 为核心的调用或声明。
- **L327 EN**: Transitions from the previous branch into the alternative path.
  **L327 CN**: 从前一个分支过渡到备选路径。
- **L328 EN**: Executes a call or declaration centered on `loadTrivialScalar`.
  **L328 CN**: 执行以 `loadTrivialScalar` 为核心的调用或声明。
- **L329 EN**: Transitions from the previous branch into the alternative path.
  **L329 CN**: 从前一个分支过渡到备选路径。
- **L330 EN**: Continues logic associated with callable symbol `TODO`.
  **L330 CN**: 继续与可调用符号 `TODO` 相关的逻辑。
- **L331 EN**: Executes a standalone statement or declaration: `"argument without box lowering");`.
  **L331 CN**: 执行一条独立语句或声明：`"argument without box lowering");`。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Transitions from the previous branch into the alternative path.
  **L333 CN**: 从前一个分支过渡到备选路径。
- **L334 EN**: Initializes variable `actual` from the right-hand expression.
  **L334 CN**: 使用右侧表达式初始化变量 `actual`。
- **L335 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L335 CN**: 开始 `if` 控制流语句并计算其条件。
- **L336 EN**: Executes a call or declaration centered on `hlfir::derefPointersAndAllocatables`.
  **L336 CN**: 执行以 `hlfir::derefPointersAndAllocatables` 为核心的调用或声明。
- **L337 EN**: Transitions from the previous branch into the alternative path.
  **L337 CN**: 从前一个分支过渡到备选路径。
- **L338 EN**: Executes a call or declaration centered on `actual.getBase`.
  **L338 CN**: 执行以 `actual.getBase` 为核心的调用或声明。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Executes a call or declaration centered on `operands.emplace_back`.
  **L340 CN**: 执行以 `operands.emplace_back` 为核心的调用或声明。

### Lines 341-360

````cpp
  }
  return operands;
}

mlir::Type
HlfirTransformationalIntrinsic::computeResultType(mlir::Value argArray,
                                                  mlir::Type stmtResultType) {
  mlir::Type normalisedResult =
      hlfir::getFortranElementOrSequenceType(stmtResultType);
  if (auto array = mlir::dyn_cast<fir::SequenceType>(normalisedResult)) {
    hlfir::ExprType::Shape resultShape =
        hlfir::ExprType::Shape{array.getShape()};
    mlir::Type elementType = array.getEleTy();
    return hlfir::ExprType::get(builder.getContext(), resultShape, elementType,
                                fir::isPolymorphicType(stmtResultType));
  } else if (auto resCharType =
                 mlir::dyn_cast<fir::CharacterType>(stmtResultType)) {
    normalisedResult = hlfir::ExprType::get(
        builder.getContext(), hlfir::ExprType::Shape{}, resCharType,
        /*polymorphic=*/false);
````
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Returns from the current function with `operands`.
  **L342 CN**: 以 `operands` 从当前函数返回。
- **L343 EN**: Closes the current lexical scope or compound statement.
  **L343 CN**: 结束当前词法作用域或复合语句块。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L345 EN**: Continues the surrounding expression or declaration: `mlir::Type`.
  **L345 CN**: 继续构造周围的表达式或声明：`mlir::Type`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HlfirTransformationalIntrinsic::computeResultType(mlir::Value argArray,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`HlfirTransformationalIntrinsic::computeResultType(mlir::Value argArray,`。
- **L347 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L347 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L348 EN**: Continues the surrounding expression or declaration: `mlir::Type normalisedResult =`.
  **L348 CN**: 继续构造周围的表达式或声明：`mlir::Type normalisedResult =`。
- **L349 EN**: Executes a call or declaration centered on `hlfir::getFortranElementOrSequenceType`.
  **L349 CN**: 执行以 `hlfir::getFortranElementOrSequenceType` 为核心的调用或声明。
- **L350 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L350 CN**: 开始 `if` 控制流语句并计算其条件。
- **L351 EN**: Continues the surrounding expression or declaration: `hlfir::ExprType::Shape resultShape =`.
  **L351 CN**: 继续构造周围的表达式或声明：`hlfir::ExprType::Shape resultShape =`。
- **L352 EN**: Executes a call or declaration centered on `hlfir::ExprType::Shape{array.getShape`.
  **L352 CN**: 执行以 `hlfir::ExprType::Shape{array.getShape` 为核心的调用或声明。
- **L353 EN**: Initializes variable `elementType` from the right-hand expression.
  **L353 CN**: 使用右侧表达式初始化变量 `elementType`。
- **L354 EN**: Returns from the current function with `hlfir::ExprType::get(builder.getContext(), resultShape, elementType,`.
  **L354 CN**: 以 `hlfir::ExprType::get(builder.getContext(), resultShape, elementType,` 从当前函数返回。
- **L355 EN**: Executes a call or declaration centered on `fir::isPolymorphicType`.
  **L355 CN**: 执行以 `fir::isPolymorphicType` 为核心的调用或声明。
- **L356 EN**: Transitions from the previous branch into an `else if` condition.
  **L356 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L357 EN**: Starts a function, method, lambda, or structured scope: `mlir::dyn_cast<fir::CharacterType>(stmtResultType)) {`.
  **L357 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::dyn_cast<fir::CharacterType>(stmtResultType)) {`。
- **L358 EN**: Continues logic associated with callable symbol `get`.
  **L358 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder.getContext(), hlfir::ExprType::Shape{}, resCharType,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder.getContext(), hlfir::ExprType::Shape{}, resCharType,`。
- **L360 EN**: Comment explains nearby logic, intent, or metadata: `polymorphic=*/false);`.
  **L360 CN**: 注释说明附近代码的逻辑、意图或元数据：`polymorphic=*/false);`。

### Lines 361-380

````cpp
  }
  return normalisedResult;
}

template <typename OP, bool HAS_MASK>
mlir::Value HlfirReductionIntrinsic<OP, HAS_MASK>::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  mlir::Value array = operands[0];
  mlir::Value dim = operands[1];
  // dim, mask can be NULL if these arguments are not given
  if (dim)
    dim = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{dim});

  mlir::Type resultTy = computeResultType(array, stmtResultType);

  OP op;
  if constexpr (HAS_MASK)
````
- **L361 EN**: Closes the current lexical scope or compound statement.
  **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Returns from the current function with `normalisedResult`.
  **L362 CN**: 以 `normalisedResult` 从当前函数返回。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L365 EN**: Introduces template parameters or specialization context: `template <typename OP, bool HAS_MASK>`.
  **L365 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP, bool HAS_MASK>`。
- **L366 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L366 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L369 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L369 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L370 EN**: Initializes variable `operands` from the right-hand expression.
  **L370 CN**: 使用右侧表达式初始化变量 `operands`。
- **L371 EN**: Initializes variable `array` from the right-hand expression.
  **L371 CN**: 使用右侧表达式初始化变量 `array`。
- **L372 EN**: Initializes variable `dim` from the right-hand expression.
  **L372 CN**: 使用右侧表达式初始化变量 `dim`。
- **L373 EN**: Comment explains nearby logic, intent, or metadata: `dim, mask can be NULL if these arguments are not given`.
  **L373 CN**: 注释说明附近代码的逻辑、意图或元数据：`dim, mask can be NULL if these arguments are not given`。
- **L374 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L374 CN**: 开始 `if` 控制流语句并计算其条件。
- **L375 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L375 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L377 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L377 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Executes a standalone statement or declaration: `OP op;`.
  **L379 CN**: 执行一条独立语句或声明：`OP op;`。
- **L380 EN**: Continues logic associated with callable symbol `constexpr`.
  **L380 CN**: 继续与可调用符号 `constexpr` 相关的逻辑。

### Lines 381-400

````cpp
    op = createOp<OP>(resultTy, array, dim,
                      /*mask=*/operands[2]);
  else
    op = createOp<OP>(resultTy, array, dim);
  return op;
}

template <typename OP>
mlir::Value HlfirMinMaxLocIntrinsic<OP>::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  mlir::Value array = operands[0];
  mlir::Value dim = operands[1];
  mlir::Value mask = operands[2];
  mlir::Value back = operands[4];
  // dim, mask and back can be NULL if these arguments are not given.
  if (dim)
    dim = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{dim});
````
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op = createOp<OP>(resultTy, array, dim,`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`op = createOp<OP>(resultTy, array, dim,`。
- **L382 EN**: Comment explains nearby logic, intent, or metadata: `mask=*/operands[2]);`.
  **L382 CN**: 注释说明附近代码的逻辑、意图或元数据：`mask=*/operands[2]);`。
- **L383 EN**: Transitions from the previous branch into the alternative path.
  **L383 CN**: 从前一个分支过渡到备选路径。
- **L384 EN**: Executes a call or declaration centered on `createOp<OP>`.
  **L384 CN**: 执行以 `createOp<OP>` 为核心的调用或声明。
- **L385 EN**: Returns from the current function with `op`.
  **L385 CN**: 以 `op` 从当前函数返回。
- **L386 EN**: Closes the current lexical scope or compound statement.
  **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L388 EN**: Introduces template parameters or specialization context: `template <typename OP>`.
  **L388 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP>`。
- **L389 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L389 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L392 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L392 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L393 EN**: Initializes variable `operands` from the right-hand expression.
  **L393 CN**: 使用右侧表达式初始化变量 `operands`。
- **L394 EN**: Initializes variable `array` from the right-hand expression.
  **L394 CN**: 使用右侧表达式初始化变量 `array`。
- **L395 EN**: Initializes variable `dim` from the right-hand expression.
  **L395 CN**: 使用右侧表达式初始化变量 `dim`。
- **L396 EN**: Initializes variable `mask` from the right-hand expression.
  **L396 CN**: 使用右侧表达式初始化变量 `mask`。
- **L397 EN**: Initializes variable `back` from the right-hand expression.
  **L397 CN**: 使用右侧表达式初始化变量 `back`。
- **L398 EN**: Comment explains nearby logic, intent, or metadata: `dim, mask and back can be NULL if these arguments are not given.`.
  **L398 CN**: 注释说明附近代码的逻辑、意图或元数据：`dim, mask and back can be NULL if these arguments are not given.`。
- **L399 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L399 CN**: 开始 `if` 控制流语句并计算其条件。
- **L400 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L400 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。

### Lines 401-420

````cpp
  if (back)
    back = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{back});

  mlir::Type resultTy = computeResultType(array, stmtResultType);

  return createOp<OP>(resultTy, array, dim, mask, back);
}

template <typename OP>
mlir::Value HlfirProductIntrinsic<OP>::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  mlir::Type resultType = computeResultType(operands[0], stmtResultType);
  return createOp<OP>(resultType, operands[0], operands[1]);
}

mlir::Value HlfirTransposeLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
````
- **L401 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L401 CN**: 开始 `if` 控制流语句并计算其条件。
- **L402 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L402 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L404 EN**: Initializes variable `resultTy` from the right-hand expression.
  **L404 CN**: 使用右侧表达式初始化变量 `resultTy`。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L406 EN**: Returns from the current function with `createOp<OP>(resultTy, array, dim, mask, back)`.
  **L406 CN**: 以 `createOp<OP>(resultTy, array, dim, mask, back)` 从当前函数返回。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L409 EN**: Introduces template parameters or specialization context: `template <typename OP>`.
  **L409 CN**: 为后续声明引入模板参数或特化上下文：`template <typename OP>`。
- **L410 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L410 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L413 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L413 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L414 EN**: Initializes variable `operands` from the right-hand expression.
  **L414 CN**: 使用右侧表达式初始化变量 `operands`。
- **L415 EN**: Initializes variable `resultType` from the right-hand expression.
  **L415 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L416 EN**: Returns from the current function with `createOp<OP>(resultType, operands[0], operands[1])`.
  **L416 CN**: 以 `createOp<OP>(resultType, operands[0], operands[1])` 从当前函数返回。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Blank line separating nearby declarations or logic blocks.
  **L418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L419 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L419 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。

### Lines 421-440

````cpp
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  hlfir::ExprType::Shape resultShape;
  mlir::Type normalisedResult =
      hlfir::getFortranElementOrSequenceType(stmtResultType);
  auto array = mlir::cast<fir::SequenceType>(normalisedResult);
  llvm::ArrayRef<int64_t> arrayShape = array.getShape();
  assert(arrayShape.size() == 2 && "arguments to transpose have a rank of 2");
  mlir::Type elementType = array.getEleTy();
  resultShape.push_back(arrayShape[0]);
  resultShape.push_back(arrayShape[1]);
  if (auto resCharType = mlir::dyn_cast<fir::CharacterType>(elementType))
    if (!resCharType.hasConstantLen()) {
      // The FunctionRef expression might have imprecise character
      // type at this point, and we can improve it by propagating
      // the constant length from the argument.
      auto argCharType = mlir::dyn_cast<fir::CharacterType>(
          hlfir::getFortranElementType(operands[0].getType()));
      if (argCharType && argCharType.hasConstantLen())
````
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L422 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L422 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L423 EN**: Initializes variable `operands` from the right-hand expression.
  **L423 CN**: 使用右侧表达式初始化变量 `operands`。
- **L424 EN**: Executes a standalone statement or declaration: `hlfir::ExprType::Shape resultShape;`.
  **L424 CN**: 执行一条独立语句或声明：`hlfir::ExprType::Shape resultShape;`。
- **L425 EN**: Continues the surrounding expression or declaration: `mlir::Type normalisedResult =`.
  **L425 CN**: 继续构造周围的表达式或声明：`mlir::Type normalisedResult =`。
- **L426 EN**: Executes a call or declaration centered on `hlfir::getFortranElementOrSequenceType`.
  **L426 CN**: 执行以 `hlfir::getFortranElementOrSequenceType` 为核心的调用或声明。
- **L427 EN**: Initializes variable `array` from the right-hand expression.
  **L427 CN**: 使用右侧表达式初始化变量 `array`。
- **L428 EN**: Initializes variable `arrayShape` from the right-hand expression.
  **L428 CN**: 使用右侧表达式初始化变量 `arrayShape`。
- **L429 EN**: Checks an internal invariant in debug builds.
  **L429 CN**: 在调试构建中检查内部不变式。
- **L430 EN**: Initializes variable `elementType` from the right-hand expression.
  **L430 CN**: 使用右侧表达式初始化变量 `elementType`。
- **L431 EN**: Executes a call or declaration centered on `resultShape.push_back`.
  **L431 CN**: 执行以 `resultShape.push_back` 为核心的调用或声明。
- **L432 EN**: Executes a call or declaration centered on `resultShape.push_back`.
  **L432 CN**: 执行以 `resultShape.push_back` 为核心的调用或声明。
- **L433 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L433 CN**: 开始 `if` 控制流语句并计算其条件。
- **L434 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L434 CN**: 开始 `if` 控制流语句并计算其条件。
- **L435 EN**: Comment explains nearby logic, intent, or metadata: `The FunctionRef expression might have imprecise character`.
  **L435 CN**: 注释说明附近代码的逻辑、意图或元数据：`The FunctionRef expression might have imprecise character`。
- **L436 EN**: Comment explains nearby logic, intent, or metadata: `type at this point, and we can improve it by propagating`.
  **L436 CN**: 注释说明附近代码的逻辑、意图或元数据：`type at this point, and we can improve it by propagating`。
- **L437 EN**: Comment explains nearby logic, intent, or metadata: `the constant length from the argument.`.
  **L437 CN**: 注释说明附近代码的逻辑、意图或元数据：`the constant length from the argument.`。
- **L438 EN**: Continues logic associated with callable symbol `CharacterType>`.
  **L438 CN**: 继续与可调用符号 `CharacterType>` 相关的逻辑。
- **L439 EN**: Executes a call or declaration centered on `hlfir::getFortranElementType`.
  **L439 CN**: 执行以 `hlfir::getFortranElementType` 为核心的调用或声明。
- **L440 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L440 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 441-460

````cpp
        elementType = fir::CharacterType::get(
            builder.getContext(), resCharType.getFKind(), argCharType.getLen());
    }

  mlir::Type resultTy =
      hlfir::ExprType::get(builder.getContext(), resultShape, elementType,
                           fir::isPolymorphicType(stmtResultType));
  return createOp<hlfir::TransposeOp>(resultTy, operands[0]);
}

mlir::Value HlfirCountLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  mlir::Value array = operands[0];
  mlir::Value dim = operands[1];
  if (dim)
    dim = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{dim});
  mlir::Type resultType = computeResultType(array, stmtResultType);
````
- **L441 EN**: Continues logic associated with callable symbol `get`.
  **L441 CN**: 继续与可调用符号 `get` 相关的逻辑。
- **L442 EN**: Executes a call or declaration centered on `builder.getContext`.
  **L442 CN**: 执行以 `builder.getContext` 为核心的调用或声明。
- **L443 EN**: Closes the current lexical scope or compound statement.
  **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L445 EN**: Continues the surrounding expression or declaration: `mlir::Type resultTy =`.
  **L445 CN**: 继续构造周围的表达式或声明：`mlir::Type resultTy =`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `hlfir::ExprType::get(builder.getContext(), resultShape, elementType,`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`hlfir::ExprType::get(builder.getContext(), resultShape, elementType,`。
- **L447 EN**: Executes a call or declaration centered on `fir::isPolymorphicType`.
  **L447 CN**: 执行以 `fir::isPolymorphicType` 为核心的调用或声明。
- **L448 EN**: Returns from the current function with `createOp<hlfir::TransposeOp>(resultTy, operands[0])`.
  **L448 CN**: 以 `createOp<hlfir::TransposeOp>(resultTy, operands[0])` 从当前函数返回。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L451 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L451 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L454 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L454 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L455 EN**: Initializes variable `operands` from the right-hand expression.
  **L455 CN**: 使用右侧表达式初始化变量 `operands`。
- **L456 EN**: Initializes variable `array` from the right-hand expression.
  **L456 CN**: 使用右侧表达式初始化变量 `array`。
- **L457 EN**: Initializes variable `dim` from the right-hand expression.
  **L457 CN**: 使用右侧表达式初始化变量 `dim`。
- **L458 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L458 CN**: 开始 `if` 控制流语句并计算其条件。
- **L459 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L459 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L460 EN**: Initializes variable `resultType` from the right-hand expression.
  **L460 CN**: 使用右侧表达式初始化变量 `resultType`。

### Lines 461-480

````cpp
  return createOp<hlfir::CountOp>(resultType, array, dim);
}

mlir::Value HlfirCharExtremumLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  assert(operands.size() >= 2);
  return createOp<hlfir::CharExtremumOp>(pred, mlir::ValueRange{operands});
}

mlir::Value HlfirCharTrimLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  assert(operands.size() == 1);
  return createOp<hlfir::CharTrimOp>(operands[0]);
}
````
- **L461 EN**: Returns from the current function with `createOp<hlfir::CountOp>(resultType, array, dim)`.
  **L461 CN**: 以 `createOp<hlfir::CountOp>(resultType, array, dim)` 从当前函数返回。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L464 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L467 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L467 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L468 EN**: Initializes variable `operands` from the right-hand expression.
  **L468 CN**: 使用右侧表达式初始化变量 `operands`。
- **L469 EN**: Checks an internal invariant in debug builds.
  **L469 CN**: 在调试构建中检查内部不变式。
- **L470 EN**: Returns from the current function with `createOp<hlfir::CharExtremumOp>(pred, mlir::ValueRange{operands})`.
  **L470 CN**: 以 `createOp<hlfir::CharExtremumOp>(pred, mlir::ValueRange{operands})` 从当前函数返回。
- **L471 EN**: Closes the current lexical scope or compound statement.
  **L471 CN**: 结束当前词法作用域或复合语句块。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L473 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L473 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L476 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L476 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L477 EN**: Initializes variable `operands` from the right-hand expression.
  **L477 CN**: 使用右侧表达式初始化变量 `operands`。
- **L478 EN**: Checks an internal invariant in debug builds.
  **L478 CN**: 在调试构建中检查内部不变式。
- **L479 EN**: Returns from the current function with `createOp<hlfir::CharTrimOp>(operands[0])`.
  **L479 CN**: 以 `createOp<hlfir::CharTrimOp>(operands[0])` 从当前函数返回。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-500

````cpp

mlir::Value HlfirCShiftLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  assert(operands.size() == 3);
  mlir::Value dim = operands[2];
  if (!dim) {
    // If DIM is not present, drop the last element which is a null Value.
    operands.truncate(2);
  } else if (loweredActuals[2] && loweredActuals[2]->handleDynamicOptional()) {
    // Use getIsPresent() to select between a present DIM value or
    // the default 1 per Fortran 16.9.68.
    mlir::Value isPresent = loweredActuals[2]->getIsPresent();
    mlir::Type dimType = dim.getType();
    mlir::Value one = builder.createIntegerConstant(loc, dimType, 1);
    dim = mlir::arith::SelectOp::create(builder, loc, isPresent, dim, one);
    operands[2] = dim;
  } else {
````
- **L481 EN**: Blank line separating nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L482 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L482 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L485 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L485 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L486 EN**: Initializes variable `operands` from the right-hand expression.
  **L486 CN**: 使用右侧表达式初始化变量 `operands`。
- **L487 EN**: Checks an internal invariant in debug builds.
  **L487 CN**: 在调试构建中检查内部不变式。
- **L488 EN**: Initializes variable `dim` from the right-hand expression.
  **L488 CN**: 使用右侧表达式初始化变量 `dim`。
- **L489 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L489 CN**: 开始 `if` 控制流语句并计算其条件。
- **L490 EN**: Comment explains nearby logic, intent, or metadata: `If DIM is not present, drop the last element which is a null Value.`.
  **L490 CN**: 注释说明附近代码的逻辑、意图或元数据：`If DIM is not present, drop the last element which is a null Value.`。
- **L491 EN**: Executes a call or declaration centered on `operands.truncate`.
  **L491 CN**: 执行以 `operands.truncate` 为核心的调用或声明。
- **L492 EN**: Transitions from the previous branch into an `else if` condition.
  **L492 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L493 EN**: Comment explains nearby logic, intent, or metadata: `Use getIsPresent() to select between a present DIM value or`.
  **L493 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use getIsPresent() to select between a present DIM value or`。
- **L494 EN**: Comment explains nearby logic, intent, or metadata: `the default 1 per Fortran 16.9.68.`.
  **L494 CN**: 注释说明附近代码的逻辑、意图或元数据：`the default 1 per Fortran 16.9.68.`。
- **L495 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L495 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L496 EN**: Initializes variable `dimType` from the right-hand expression.
  **L496 CN**: 使用右侧表达式初始化变量 `dimType`。
- **L497 EN**: Initializes variable `one` from the right-hand expression.
  **L497 CN**: 使用右侧表达式初始化变量 `one`。
- **L498 EN**: Executes a call or declaration centered on `mlir::arith::SelectOp::create`.
  **L498 CN**: 执行以 `mlir::arith::SelectOp::create` 为核心的调用或声明。
- **L499 EN**: Executes a standalone statement or declaration: `operands[2] = dim;`.
  **L499 CN**: 执行一条独立语句或声明：`operands[2] = dim;`。
- **L500 EN**: Transitions from the previous branch into the alternative path.
  **L500 CN**: 从前一个分支过渡到备选路径。

### Lines 501-520

````cpp
    // If DIM is present, then dereference it if it is a ref.
    dim = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{dim});
    operands[2] = dim;
  }

  mlir::Type resultType = computeResultType(operands[0], stmtResultType);
  return createOp<hlfir::CShiftOp>(resultType, operands);
}

mlir::Value HlfirEOShiftLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  assert(operands.size() == 4);
  mlir::Value array = operands[0];
  mlir::Value shift = operands[1];
  mlir::Value boundary = operands[2];
  mlir::Value dim = operands[3];
  if (loweredActuals[3] && loweredActuals[3]->handleDynamicOptional()) {
````
- **L501 EN**: Comment explains nearby logic, intent, or metadata: `If DIM is present, then dereference it if it is a ref.`.
  **L501 CN**: 注释说明附近代码的逻辑、意图或元数据：`If DIM is present, then dereference it if it is a ref.`。
- **L502 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L502 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L503 EN**: Executes a standalone statement or declaration: `operands[2] = dim;`.
  **L503 CN**: 执行一条独立语句或声明：`operands[2] = dim;`。
- **L504 EN**: Closes the current lexical scope or compound statement.
  **L504 CN**: 结束当前词法作用域或复合语句块。
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L506 EN**: Initializes variable `resultType` from the right-hand expression.
  **L506 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L507 EN**: Returns from the current function with `createOp<hlfir::CShiftOp>(resultType, operands)`.
  **L507 CN**: 以 `createOp<hlfir::CShiftOp>(resultType, operands)` 从当前函数返回。
- **L508 EN**: Closes the current lexical scope or compound statement.
  **L508 CN**: 结束当前词法作用域或复合语句块。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L510 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L510 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L513 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L513 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L514 EN**: Initializes variable `operands` from the right-hand expression.
  **L514 CN**: 使用右侧表达式初始化变量 `operands`。
- **L515 EN**: Checks an internal invariant in debug builds.
  **L515 CN**: 在调试构建中检查内部不变式。
- **L516 EN**: Initializes variable `array` from the right-hand expression.
  **L516 CN**: 使用右侧表达式初始化变量 `array`。
- **L517 EN**: Initializes variable `shift` from the right-hand expression.
  **L517 CN**: 使用右侧表达式初始化变量 `shift`。
- **L518 EN**: Initializes variable `boundary` from the right-hand expression.
  **L518 CN**: 使用右侧表达式初始化变量 `boundary`。
- **L519 EN**: Initializes variable `dim` from the right-hand expression.
  **L519 CN**: 使用右侧表达式初始化变量 `dim`。
- **L520 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L520 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 521-540

````cpp
    // Use getIsPresent() to select between a present DIM value or
    // the default 1 per Fortran 16.9.77.
    mlir::Value isPresent = loweredActuals[3]->getIsPresent();
    mlir::Type dimType = dim.getType();
    mlir::Value one = builder.createIntegerConstant(loc, dimType, 1);
    dim = mlir::arith::SelectOp::create(builder, loc, isPresent, dim, one);
  } else if (dim) {
    // If DIM is statically present, dereference it if it is a ref.
    dim = hlfir::loadTrivialScalar(loc, builder, hlfir::Entity{dim});
  }

  mlir::Type resultType = computeResultType(array, stmtResultType);

  if (boundary && fir::isa_trivial(boundary.getType())) {
    mlir::Type elementType = hlfir::getFortranElementType(resultType);
    if (auto logicalTy = mlir::dyn_cast<fir::LogicalType>(elementType)) {
      // Scalar logical constant boundary might be represented using i1, i2, ...
      // type. We need to cast it to fir.logical type of the ARRAY/result.
      if (boundary.getType() != logicalTy)
        boundary = builder.createConvert(loc, logicalTy, boundary);
````
- **L521 EN**: Comment explains nearby logic, intent, or metadata: `Use getIsPresent() to select between a present DIM value or`.
  **L521 CN**: 注释说明附近代码的逻辑、意图或元数据：`Use getIsPresent() to select between a present DIM value or`。
- **L522 EN**: Comment explains nearby logic, intent, or metadata: `the default 1 per Fortran 16.9.77.`.
  **L522 CN**: 注释说明附近代码的逻辑、意图或元数据：`the default 1 per Fortran 16.9.77.`。
- **L523 EN**: Initializes variable `isPresent` from the right-hand expression.
  **L523 CN**: 使用右侧表达式初始化变量 `isPresent`。
- **L524 EN**: Initializes variable `dimType` from the right-hand expression.
  **L524 CN**: 使用右侧表达式初始化变量 `dimType`。
- **L525 EN**: Initializes variable `one` from the right-hand expression.
  **L525 CN**: 使用右侧表达式初始化变量 `one`。
- **L526 EN**: Executes a call or declaration centered on `mlir::arith::SelectOp::create`.
  **L526 CN**: 执行以 `mlir::arith::SelectOp::create` 为核心的调用或声明。
- **L527 EN**: Transitions from the previous branch into an `else if` condition.
  **L527 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L528 EN**: Comment explains nearby logic, intent, or metadata: `If DIM is statically present, dereference it if it is a ref.`.
  **L528 CN**: 注释说明附近代码的逻辑、意图或元数据：`If DIM is statically present, dereference it if it is a ref.`。
- **L529 EN**: Executes a call or declaration centered on `hlfir::loadTrivialScalar`.
  **L529 CN**: 执行以 `hlfir::loadTrivialScalar` 为核心的调用或声明。
- **L530 EN**: Closes the current lexical scope or compound statement.
  **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Blank line separating nearby declarations or logic blocks.
  **L531 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L532 EN**: Initializes variable `resultType` from the right-hand expression.
  **L532 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Initializes variable `elementType` from the right-hand expression.
  **L535 CN**: 使用右侧表达式初始化变量 `elementType`。
- **L536 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L536 CN**: 开始 `if` 控制流语句并计算其条件。
- **L537 EN**: Comment explains nearby logic, intent, or metadata: `Scalar logical constant boundary might be represented using i1, i2, ...`.
  **L537 CN**: 注释说明附近代码的逻辑、意图或元数据：`Scalar logical constant boundary might be represented using i1, i2, ...`。
- **L538 EN**: Comment explains nearby logic, intent, or metadata: `type. We need to cast it to fir.logical type of the ARRAY/result.`.
  **L538 CN**: 注释说明附近代码的逻辑、意图或元数据：`type. We need to cast it to fir.logical type of the ARRAY/result.`。
- **L539 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L539 CN**: 开始 `if` 控制流语句并计算其条件。
- **L540 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L540 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。

### Lines 541-560

````cpp
    } else {
      // When the boundary is a constant like '1u', the lowering converts
      // it into a signless arith.constant value (which is a requirement
      // of the Arith dialect). If the ARRAY/RESULT is also UNSIGNED,
      // we have to cast the boundary to the same unsigned type.
      auto resultIntTy = mlir::dyn_cast<mlir::IntegerType>(elementType);
      auto boundaryIntTy =
          mlir::dyn_cast<mlir::IntegerType>(boundary.getType());
      if (resultIntTy && boundaryIntTy &&
          resultIntTy.getSignedness() != boundaryIntTy.getSignedness())
        boundary = builder.createConvert(loc, resultIntTy, boundary);
    }
  }

  return createOp<hlfir::EOShiftOp>(resultType, array, shift, boundary, dim);
}

mlir::Value HlfirReshapeLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
````
- **L541 EN**: Transitions from the previous branch into the alternative path.
  **L541 CN**: 从前一个分支过渡到备选路径。
- **L542 EN**: Comment explains nearby logic, intent, or metadata: `When the boundary is a constant like '1u', the lowering converts`.
  **L542 CN**: 注释说明附近代码的逻辑、意图或元数据：`When the boundary is a constant like '1u', the lowering converts`。
- **L543 EN**: Comment explains nearby logic, intent, or metadata: `it into a signless arith.constant value (which is a requirement`.
  **L543 CN**: 注释说明附近代码的逻辑、意图或元数据：`it into a signless arith.constant value (which is a requirement`。
- **L544 EN**: Comment explains nearby logic, intent, or metadata: `of the Arith dialect). If the ARRAY/RESULT is also UNSIGNED,`.
  **L544 CN**: 注释说明附近代码的逻辑、意图或元数据：`of the Arith dialect). If the ARRAY/RESULT is also UNSIGNED,`。
- **L545 EN**: Comment explains nearby logic, intent, or metadata: `we have to cast the boundary to the same unsigned type.`.
  **L545 CN**: 注释说明附近代码的逻辑、意图或元数据：`we have to cast the boundary to the same unsigned type.`。
- **L546 EN**: Initializes variable `resultIntTy` from the right-hand expression.
  **L546 CN**: 使用右侧表达式初始化变量 `resultIntTy`。
- **L547 EN**: Continues the surrounding expression or declaration: `auto boundaryIntTy =`.
  **L547 CN**: 继续构造周围的表达式或声明：`auto boundaryIntTy =`。
- **L548 EN**: Executes a call or declaration centered on `mlir::dyn_cast<mlir::IntegerType>`.
  **L548 CN**: 执行以 `mlir::dyn_cast<mlir::IntegerType>` 为核心的调用或声明。
- **L549 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L549 CN**: 开始 `if` 控制流语句并计算其条件。
- **L550 EN**: Continues logic associated with callable symbol `getSignedness`.
  **L550 CN**: 继续与可调用符号 `getSignedness` 相关的逻辑。
- **L551 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L551 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L555 EN**: Returns from the current function with `createOp<hlfir::EOShiftOp>(resultType, array, shift, boundary, dim)`.
  **L555 CN**: 以 `createOp<hlfir::EOShiftOp>(resultType, array, shift, boundary, dim)` 从当前函数返回。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Blank line separating nearby declarations or logic blocks.
  **L557 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L558 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L558 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。

### Lines 561-580

````cpp
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  assert(operands.size() == 4);
  mlir::Type resultType = computeResultType(operands[0], stmtResultType);
  return createOp<hlfir::ReshapeOp>(resultType, operands[0], operands[1],
                                    operands[2], operands[3]);
}

mlir::Value HlfirIndexLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  auto operands = getOperandVector(loweredActuals, argLowering);
  // 'kind' optional operand is unused here as it has already been
  // translated into result type.
  assert(operands.size() == 4);
  mlir::Value substr = operands[1];
  mlir::Value str = operands[0];
  mlir::Value back = operands[2];
  mlir::Value result =
````
- **L561 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L561 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L562 EN**: Initializes variable `operands` from the right-hand expression.
  **L562 CN**: 使用右侧表达式初始化变量 `operands`。
- **L563 EN**: Checks an internal invariant in debug builds.
  **L563 CN**: 在调试构建中检查内部不变式。
- **L564 EN**: Initializes variable `resultType` from the right-hand expression.
  **L564 CN**: 使用右侧表达式初始化变量 `resultType`。
- **L565 EN**: Returns from the current function with `createOp<hlfir::ReshapeOp>(resultType, operands[0], operands[1],`.
  **L565 CN**: 以 `createOp<hlfir::ReshapeOp>(resultType, operands[0], operands[1],` 从当前函数返回。
- **L566 EN**: Executes a standalone statement or declaration: `operands[2], operands[3]);`.
  **L566 CN**: 执行一条独立语句或声明：`operands[2], operands[3]);`。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L569 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L572 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L572 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L573 EN**: Initializes variable `operands` from the right-hand expression.
  **L573 CN**: 使用右侧表达式初始化变量 `operands`。
- **L574 EN**: Comment explains nearby logic, intent, or metadata: `'kind' optional operand is unused here as it has already been`.
  **L574 CN**: 注释说明附近代码的逻辑、意图或元数据：`'kind' optional operand is unused here as it has already been`。
- **L575 EN**: Comment explains nearby logic, intent, or metadata: `translated into result type.`.
  **L575 CN**: 注释说明附近代码的逻辑、意图或元数据：`translated into result type.`。
- **L576 EN**: Checks an internal invariant in debug builds.
  **L576 CN**: 在调试构建中检查内部不变式。
- **L577 EN**: Initializes variable `substr` from the right-hand expression.
  **L577 CN**: 使用右侧表达式初始化变量 `substr`。
- **L578 EN**: Initializes variable `str` from the right-hand expression.
  **L578 CN**: 使用右侧表达式初始化变量 `str`。
- **L579 EN**: Initializes variable `back` from the right-hand expression.
  **L579 CN**: 使用右侧表达式初始化变量 `back`。
- **L580 EN**: Continues the surrounding expression or declaration: `mlir::Value result =`.
  **L580 CN**: 继续构造周围的表达式或声明：`mlir::Value result =`。

### Lines 581-600

````cpp
      createOp<hlfir::IndexOp>(stmtResultType, substr, str, back);
  return result;
}

mlir::Value HlfirLenLowering::lowerImpl(
    const Fortran::lower::PreparedActualArguments &loweredActuals,
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  // LEN (STRING [, KIND])
  assert((loweredActuals.size() == 1 || loweredActuals.size() == 2) &&
         loweredActuals[0].has_value());
  Fortran::lower::PreparedActualArgument &strArg =
      const_cast<Fortran::lower::PreparedActualArgument &>(*loweredActuals[0]);
  return builder.createConvert(loc, stmtResultType,
                               strArg.genCharLength(loc, builder));
}

std::optional<hlfir::EntityWithAttributes> Fortran::lower::lowerHlfirIntrinsic(
    fir::FirOpBuilder &builder, mlir::Location loc, const std::string &name,
    const Fortran::lower::PreparedActualArguments &loweredActuals,
````
- **L581 EN**: Executes a call or declaration centered on `createOp<hlfir::IndexOp>`.
  **L581 CN**: 执行以 `createOp<hlfir::IndexOp>` 为核心的调用或声明。
- **L582 EN**: Returns from the current function with `result`.
  **L582 CN**: 以 `result` 从当前函数返回。
- **L583 EN**: Closes the current lexical scope or compound statement.
  **L583 CN**: 结束当前词法作用域或复合语句块。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L585 EN**: Continues logic associated with callable symbol `lowerImpl`.
  **L585 CN**: 继续与可调用符号 `lowerImpl` 相关的逻辑。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L588 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L588 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L589 EN**: Comment explains nearby logic, intent, or metadata: `LEN (STRING [, KIND])`.
  **L589 CN**: 注释说明附近代码的逻辑、意图或元数据：`LEN (STRING [, KIND])`。
- **L590 EN**: Checks an internal invariant in debug builds.
  **L590 CN**: 在调试构建中检查内部不变式。
- **L591 EN**: Executes a call or declaration centered on `loweredActuals[0].has_value`.
  **L591 CN**: 执行以 `loweredActuals[0].has_value` 为核心的调用或声明。
- **L592 EN**: Continues the surrounding expression or declaration: `Fortran::lower::PreparedActualArgument &strArg =`.
  **L592 CN**: 继续构造周围的表达式或声明：`Fortran::lower::PreparedActualArgument &strArg =`。
- **L593 EN**: Executes a call or declaration centered on `&>`.
  **L593 CN**: 执行以 `&>` 为核心的调用或声明。
- **L594 EN**: Returns from the current function with `builder.createConvert(loc, stmtResultType,`.
  **L594 CN**: 以 `builder.createConvert(loc, stmtResultType,` 从当前函数返回。
- **L595 EN**: Executes a call or declaration centered on `strArg.genCharLength`.
  **L595 CN**: 执行以 `strArg.genCharLength` 为核心的调用或声明。
- **L596 EN**: Closes the current lexical scope or compound statement.
  **L596 CN**: 结束当前词法作用域或复合语句块。
- **L597 EN**: Blank line separating nearby declarations or logic blocks.
  **L597 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L598 EN**: Continues logic associated with callable symbol `lowerHlfirIntrinsic`.
  **L598 CN**: 继续与可调用符号 `lowerHlfirIntrinsic` 相关的逻辑。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::FirOpBuilder &builder, mlir::Location loc, const std::string &name,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::FirOpBuilder &builder, mlir::Location loc, const std::string &name,`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const Fortran::lower::PreparedActualArguments &loweredActuals,`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`const Fortran::lower::PreparedActualArguments &loweredActuals,`。

### Lines 601-620

````cpp
    const fir::IntrinsicArgumentLoweringRules *argLowering,
    mlir::Type stmtResultType) {
  // If the result is of a derived type that may need finalization,
  // we have to use DestroyOp with 'finalize' attribute for the result
  // of the intrinsic operation.
  if (name == "sum")
    return HlfirSumLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                stmtResultType);
  if (name == "product")
    return HlfirProductLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                    stmtResultType);
  if (name == "any")
    return HlfirAnyLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                stmtResultType);
  if (name == "all")
    return HlfirAllLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                stmtResultType);
  if (name == "matmul")
    return HlfirMatmulLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const fir::IntrinsicArgumentLoweringRules *argLowering,`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`const fir::IntrinsicArgumentLoweringRules *argLowering,`。
- **L602 EN**: Continues the surrounding expression or declaration: `mlir::Type stmtResultType) {`.
  **L602 CN**: 继续构造周围的表达式或声明：`mlir::Type stmtResultType) {`。
- **L603 EN**: Comment explains nearby logic, intent, or metadata: `If the result is of a derived type that may need finalization,`.
  **L603 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the result is of a derived type that may need finalization,`。
- **L604 EN**: Comment explains nearby logic, intent, or metadata: `we have to use DestroyOp with 'finalize' attribute for the result`.
  **L604 CN**: 注释说明附近代码的逻辑、意图或元数据：`we have to use DestroyOp with 'finalize' attribute for the result`。
- **L605 EN**: Comment explains nearby logic, intent, or metadata: `of the intrinsic operation.`.
  **L605 CN**: 注释说明附近代码的逻辑、意图或元数据：`of the intrinsic operation.`。
- **L606 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L606 CN**: 开始 `if` 控制流语句并计算其条件。
- **L607 EN**: Returns from the current function with `HlfirSumLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L607 CN**: 以 `HlfirSumLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L608 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L608 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L609 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L609 CN**: 开始 `if` 控制流语句并计算其条件。
- **L610 EN**: Returns from the current function with `HlfirProductLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L610 CN**: 以 `HlfirProductLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L611 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L611 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L612 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L612 CN**: 开始 `if` 控制流语句并计算其条件。
- **L613 EN**: Returns from the current function with `HlfirAnyLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L613 CN**: 以 `HlfirAnyLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L614 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L614 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L615 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L615 CN**: 开始 `if` 控制流语句并计算其条件。
- **L616 EN**: Returns from the current function with `HlfirAllLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L616 CN**: 以 `HlfirAllLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L617 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L617 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L618 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L618 CN**: 开始 `if` 控制流语句并计算其条件。
- **L619 EN**: Returns from the current function with `HlfirMatmulLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L619 CN**: 以 `HlfirMatmulLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L620 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L620 CN**: 执行一条独立语句或声明：`stmtResultType);`。

### Lines 621-640

````cpp
  if (name == "dot_product")
    return HlfirDotProductLowering{builder, loc}.lower(
        loweredActuals, argLowering, stmtResultType);
  // FIXME: the result may need finalization.
  if (name == "transpose")
    return HlfirTransposeLowering{builder, loc}.lower(
        loweredActuals, argLowering, stmtResultType);
  if (name == "count")
    return HlfirCountLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                  stmtResultType);
  if (name == "maxval")
    return HlfirMaxvalLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
  if (name == "minval")
    return HlfirMinvalLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
  if (name == "minloc")
    return HlfirMinlocLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
  if (name == "maxloc")
````
- **L621 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L621 CN**: 开始 `if` 控制流语句并计算其条件。
- **L622 EN**: Returns from the current function with `HlfirDotProductLowering{builder, loc}.lower(`.
  **L622 CN**: 以 `HlfirDotProductLowering{builder, loc}.lower(` 从当前函数返回。
- **L623 EN**: Executes a standalone statement or declaration: `loweredActuals, argLowering, stmtResultType);`.
  **L623 CN**: 执行一条独立语句或声明：`loweredActuals, argLowering, stmtResultType);`。
- **L624 EN**: Comment records a pending task or caution: `FIXME: the result may need finalization.`.
  **L624 CN**: 注释记录待办事项或注意点：`FIXME: the result may need finalization.`。
- **L625 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L625 CN**: 开始 `if` 控制流语句并计算其条件。
- **L626 EN**: Returns from the current function with `HlfirTransposeLowering{builder, loc}.lower(`.
  **L626 CN**: 以 `HlfirTransposeLowering{builder, loc}.lower(` 从当前函数返回。
- **L627 EN**: Executes a standalone statement or declaration: `loweredActuals, argLowering, stmtResultType);`.
  **L627 CN**: 执行一条独立语句或声明：`loweredActuals, argLowering, stmtResultType);`。
- **L628 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L628 CN**: 开始 `if` 控制流语句并计算其条件。
- **L629 EN**: Returns from the current function with `HlfirCountLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L629 CN**: 以 `HlfirCountLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L630 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L630 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L631 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L631 CN**: 开始 `if` 控制流语句并计算其条件。
- **L632 EN**: Returns from the current function with `HlfirMaxvalLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L632 CN**: 以 `HlfirMaxvalLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L633 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L633 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L634 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L634 CN**: 开始 `if` 控制流语句并计算其条件。
- **L635 EN**: Returns from the current function with `HlfirMinvalLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L635 CN**: 以 `HlfirMinvalLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L636 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L636 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L637 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L637 CN**: 开始 `if` 控制流语句并计算其条件。
- **L638 EN**: Returns from the current function with `HlfirMinlocLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L638 CN**: 以 `HlfirMinlocLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L639 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L639 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L640 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L640 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 641-660

````cpp
    return HlfirMaxlocLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
  if (name == "cshift")
    return HlfirCShiftLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                   stmtResultType);
  if (name == "eoshift")
    return HlfirEOShiftLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                    stmtResultType);
  if (name == "reshape")
    return HlfirReshapeLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                    stmtResultType);
  if (name == "index")
    return HlfirIndexLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                  stmtResultType);
  if (name == "len")
    return HlfirLenLowering{builder, loc}.lower(loweredActuals, argLowering,
                                                stmtResultType);

  if (mlir::isa<fir::CharacterType>(stmtResultType)) {
    if (name == "min")
````
- **L641 EN**: Returns from the current function with `HlfirMaxlocLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L641 CN**: 以 `HlfirMaxlocLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L642 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L642 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L643 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L643 CN**: 开始 `if` 控制流语句并计算其条件。
- **L644 EN**: Returns from the current function with `HlfirCShiftLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L644 CN**: 以 `HlfirCShiftLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L645 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L645 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L646 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L646 CN**: 开始 `if` 控制流语句并计算其条件。
- **L647 EN**: Returns from the current function with `HlfirEOShiftLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L647 CN**: 以 `HlfirEOShiftLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L648 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L648 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L649 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L649 CN**: 开始 `if` 控制流语句并计算其条件。
- **L650 EN**: Returns from the current function with `HlfirReshapeLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L650 CN**: 以 `HlfirReshapeLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L651 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L651 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L652 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L652 CN**: 开始 `if` 控制流语句并计算其条件。
- **L653 EN**: Returns from the current function with `HlfirIndexLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L653 CN**: 以 `HlfirIndexLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L654 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L654 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L655 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L655 CN**: 开始 `if` 控制流语句并计算其条件。
- **L656 EN**: Returns from the current function with `HlfirLenLowering{builder, loc}.lower(loweredActuals, argLowering,`.
  **L656 CN**: 以 `HlfirLenLowering{builder, loc}.lower(loweredActuals, argLowering,` 从当前函数返回。
- **L657 EN**: Executes a standalone statement or declaration: `stmtResultType);`.
  **L657 CN**: 执行一条独立语句或声明：`stmtResultType);`。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L659 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L659 CN**: 开始 `if` 控制流语句并计算其条件。
- **L660 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L660 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 661-673

````cpp
      return HlfirCharExtremumLowering{builder, loc,
                                       hlfir::CharExtremumPredicate::min}
          .lower(loweredActuals, argLowering, stmtResultType);
    if (name == "max")
      return HlfirCharExtremumLowering{builder, loc,
                                       hlfir::CharExtremumPredicate::max}
          .lower(loweredActuals, argLowering, stmtResultType);
    if (name == "trim")
      return HlfirCharTrimLowering{builder, loc}.lower(
          loweredActuals, argLowering, stmtResultType);
  }
  return std::nullopt;
}
````
- **L661 EN**: Returns from the current function with `HlfirCharExtremumLowering{builder, loc,`.
  **L661 CN**: 以 `HlfirCharExtremumLowering{builder, loc,` 从当前函数返回。
- **L662 EN**: Continues the surrounding expression or declaration: `hlfir::CharExtremumPredicate::min}`.
  **L662 CN**: 继续构造周围的表达式或声明：`hlfir::CharExtremumPredicate::min}`。
- **L663 EN**: Executes a call or declaration centered on `.lower`.
  **L663 CN**: 执行以 `.lower` 为核心的调用或声明。
- **L664 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L664 CN**: 开始 `if` 控制流语句并计算其条件。
- **L665 EN**: Returns from the current function with `HlfirCharExtremumLowering{builder, loc,`.
  **L665 CN**: 以 `HlfirCharExtremumLowering{builder, loc,` 从当前函数返回。
- **L666 EN**: Continues the surrounding expression or declaration: `hlfir::CharExtremumPredicate::max}`.
  **L666 CN**: 继续构造周围的表达式或声明：`hlfir::CharExtremumPredicate::max}`。
- **L667 EN**: Executes a call or declaration centered on `.lower`.
  **L667 CN**: 执行以 `.lower` 为核心的调用或声明。
- **L668 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L668 CN**: 开始 `if` 控制流语句并计算其条件。
- **L669 EN**: Returns from the current function with `HlfirCharTrimLowering{builder, loc}.lower(`.
  **L669 CN**: 以 `HlfirCharTrimLowering{builder, loc}.lower(` 从当前函数返回。
- **L670 EN**: Executes a standalone statement or declaration: `loweredActuals, argLowering, stmtResultType);`.
  **L670 CN**: 执行一条独立语句或声明：`loweredActuals, argLowering, stmtResultType);`。
- **L671 EN**: Closes the current lexical scope or compound statement.
  **L671 CN**: 结束当前词法作用域或复合语句块。
- **L672 EN**: Returns from the current function with `std::nullopt`.
  **L672 CN**: 以 `std::nullopt` 从当前函数返回。
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Lowering to FIR/MLIR / 向 FIR/MLIR 的 lowering**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **HLFIR-specific abstractions / HLFIR 专用抽象**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **IR builder orchestration / IR Builder 编排**

## Dependencies / 依赖关系

- `flang/Lower/HlfirIntrinsics.h`: Provides lowering utilities that map semantics into FIR/MLIR. / 提供将语义映射到 FIR/MLIR 的 lowering 工具。
- `flang/Optimizer/Builder/BoxValue.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/HLFIRTools.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/IntrinsicCall.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/MutableBox.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Builder/Todo.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Dialect/FIRType.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/HLFIR/HLFIRDialect.h`: Provides HLFIR abstractions and transformation support. / 提供HLFIR 抽象与变换支持。
- `flang/Optimizer/HLFIR/HLFIROps.h`: Provides HLFIR abstractions and transformation support. / 提供HLFIR 抽象与变换支持。
- `mlir/IR/Value.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `llvm/ADT/SmallVector.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `mlir/IR/ValueRange.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
