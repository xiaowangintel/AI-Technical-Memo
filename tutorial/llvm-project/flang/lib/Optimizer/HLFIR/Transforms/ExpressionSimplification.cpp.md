# ExpressionSimplification.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/HLFIR/Transforms/ExpressionSimplification.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for Expression Simplification.
- **Purpose (CN)**: 实现 Expression Simplification 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===- ExpressionSimplification.cpp - Simplify HLFIR expressions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/HLFIR/HLFIROps.h"
#include "flang/Optimizer/HLFIR/Passes.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"

namespace hlfir {
#define GEN_PASS_DEF_EXPRESSIONSIMPLIFICATION
#include "flang/Optimizer/HLFIR/Passes.h.inc"
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
- **L9 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L9 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L10 EN**: Includes "flang/Optimizer/HLFIR/HLFIROps.h" to access HLFIR abstractions and transformation support.
  **L10 CN**: 引入 "flang/Optimizer/HLFIR/HLFIROps.h" 以使用HLFIR 抽象与变换支持。
- **L11 EN**: Includes "flang/Optimizer/HLFIR/Passes.h" to access HLFIR abstractions and transformation support.
  **L11 CN**: 引入 "flang/Optimizer/HLFIR/Passes.h" 以使用HLFIR 抽象与变换支持。
- **L12 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L12 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L14 EN**: Opens namespace scope `hlfir`.
  **L14 CN**: 打开命名空间作用域 `hlfir`。
- **L15 EN**: Defines macro `GEN_PASS_DEF_EXPRESSIONSIMPLIFICATION` for conditional compilation or local shorthand.
  **L15 CN**: 定义宏 `GEN_PASS_DEF_EXPRESSIONSIMPLIFICATION`，用于条件编译或本地简写。
- **L16 EN**: Includes "flang/Optimizer/HLFIR/Passes.h.inc" to access HLFIR abstractions and transformation support.
  **L16 CN**: 引入 "flang/Optimizer/HLFIR/Passes.h.inc" 以使用HLFIR 抽象与变换支持。

### Lines 17-32

````cpp
} // namespace hlfir

// Get the first user of `op`.
// Note that we consider the first user to be the one on the lowest line of
// the emitted HLFIR. The user iterator considers the opposite.
template <typename UserOp>
static UserOp getFirstUser(mlir::Operation *op) {
  auto it = op->user_begin(), end = op->user_end(), prev = it;
  for (; it != end; prev = it++)
    ;
  if (prev != end)
    if (auto userOp = mlir::dyn_cast<UserOp>(*prev))
      return userOp;
  return {};
}

````
- **L17 EN**: Closes a namespace scope with a trailing comment: `} // namespace hlfir`.
  **L17 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace hlfir`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, intent, or metadata: `Get the first user of `op`.`.
  **L19 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get the first user of `op`.`。
- **L20 EN**: Comment explains nearby logic, intent, or metadata: `Note that we consider the first user to be the one on the lowest line of`.
  **L20 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that we consider the first user to be the one on the lowest line of`。
- **L21 EN**: Comment explains nearby logic, intent, or metadata: `the emitted HLFIR. The user iterator considers the opposite.`.
  **L21 CN**: 注释说明附近代码的逻辑、意图或元数据：`the emitted HLFIR. The user iterator considers the opposite.`。
- **L22 EN**: Introduces template parameters or specialization context: `template <typename UserOp>`.
  **L22 CN**: 为后续声明引入模板参数或特化上下文：`template <typename UserOp>`。
- **L23 EN**: Starts a function, method, lambda, or structured scope: `static UserOp getFirstUser(mlir::Operation *op) {`.
  **L23 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static UserOp getFirstUser(mlir::Operation *op) {`。
- **L24 EN**: Initializes variable `it` from the right-hand expression.
  **L24 CN**: 使用右侧表达式初始化变量 `it`。
- **L25 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L25 CN**: 开始 `for` 控制流语句并计算其条件。
- **L26 EN**: Executes a standalone statement or declaration: `;`.
  **L26 CN**: 执行一条独立语句或声明：`;`。
- **L27 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L27 CN**: 开始 `if` 控制流语句并计算其条件。
- **L28 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L28 CN**: 开始 `if` 控制流语句并计算其条件。
- **L29 EN**: Returns from the current function with `userOp`.
  **L29 CN**: 以 `userOp` 从当前函数返回。
- **L30 EN**: Returns from the current function with `{}`.
  **L30 CN**: 以 `{}` 从当前函数返回。
- **L31 EN**: Closes the current lexical scope or compound statement.
  **L31 CN**: 结束当前词法作用域或复合语句块。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 33-48

````cpp
// Get the last user of `op`.
// Note that we consider the last user to be the one on the highest line of
// the emitted HLFIR. The user iterator considers the opposite.
template <typename UserOp>
static UserOp getLastUser(mlir::Operation *op) {
  if (!op->getUsers().empty())
    if (auto userOp = mlir::dyn_cast<UserOp>(*op->user_begin()))
      return userOp;
  return {};
}

namespace {

// Trim operations can be erased in certain expressions, such as character
// comparisons.
// Since a character comparison appends spaces to the shorter character,
````
- **L33 EN**: Comment explains nearby logic, intent, or metadata: `Get the last user of `op`.`.
  **L33 CN**: 注释说明附近代码的逻辑、意图或元数据：`Get the last user of `op`.`。
- **L34 EN**: Comment explains nearby logic, intent, or metadata: `Note that we consider the last user to be the one on the highest line of`.
  **L34 CN**: 注释说明附近代码的逻辑、意图或元数据：`Note that we consider the last user to be the one on the highest line of`。
- **L35 EN**: Comment explains nearby logic, intent, or metadata: `the emitted HLFIR. The user iterator considers the opposite.`.
  **L35 CN**: 注释说明附近代码的逻辑、意图或元数据：`the emitted HLFIR. The user iterator considers the opposite.`。
- **L36 EN**: Introduces template parameters or specialization context: `template <typename UserOp>`.
  **L36 CN**: 为后续声明引入模板参数或特化上下文：`template <typename UserOp>`。
- **L37 EN**: Starts a function, method, lambda, or structured scope: `static UserOp getLastUser(mlir::Operation *op) {`.
  **L37 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static UserOp getLastUser(mlir::Operation *op) {`。
- **L38 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L38 CN**: 开始 `if` 控制流语句并计算其条件。
- **L39 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L39 CN**: 开始 `if` 控制流语句并计算其条件。
- **L40 EN**: Returns from the current function with `userOp`.
  **L40 CN**: 以 `userOp` 从当前函数返回。
- **L41 EN**: Returns from the current function with `{}`.
  **L41 CN**: 以 `{}` 从当前函数返回。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Opens namespace scope ``.
  **L44 CN**: 打开命名空间作用域 ``。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L46 EN**: Comment explains nearby logic, intent, or metadata: `Trim operations can be erased in certain expressions, such as character`.
  **L46 CN**: 注释说明附近代码的逻辑、意图或元数据：`Trim operations can be erased in certain expressions, such as character`。
- **L47 EN**: Comment explains nearby logic, intent, or metadata: `comparisons.`.
  **L47 CN**: 注释说明附近代码的逻辑、意图或元数据：`comparisons.`。
- **L48 EN**: Comment explains nearby logic, intent, or metadata: `Since a character comparison appends spaces to the shorter character,`.
  **L48 CN**: 注释说明附近代码的逻辑、意图或元数据：`Since a character comparison appends spaces to the shorter character,`。

### Lines 49-64

````cpp
// calls to trim() that are used only in the comparison can be eliminated.
//
// Example:
// `trim(x) == trim(y)`
// can be simplified to
// `x == y`
class EraseTrim : public mlir::OpRewritePattern<hlfir::CharTrimOp> {
public:
  using mlir::OpRewritePattern<hlfir::CharTrimOp>::OpRewritePattern;

  llvm::LogicalResult
  matchAndRewrite(hlfir::CharTrimOp trimOp,
                  mlir::PatternRewriter &rewriter) const override {
    int trimUses = std::distance(trimOp->use_begin(), trimOp->use_end());
    auto cmpCharOp = getFirstUser<hlfir::CmpCharOp>(trimOp);
    auto destroyOp = getLastUser<hlfir::DestroyOp>(trimOp);
````
- **L49 EN**: Comment explains nearby logic, intent, or metadata: `calls to trim() that are used only in the comparison can be eliminated.`.
  **L49 CN**: 注释说明附近代码的逻辑、意图或元数据：`calls to trim() that are used only in the comparison can be eliminated.`。
- **L50 EN**: Separator comment used for visual grouping.
  **L50 CN**: 用于视觉分组的分隔注释。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `Example:`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`Example:`。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: ``trim(x) == trim(y)``.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：``trim(x) == trim(y)``。
- **L53 EN**: Comment explains nearby logic, intent, or metadata: `can be simplified to`.
  **L53 CN**: 注释说明附近代码的逻辑、意图或元数据：`can be simplified to`。
- **L54 EN**: Comment explains nearby logic, intent, or metadata: ``x == y``.
  **L54 CN**: 注释说明附近代码的逻辑、意图或元数据：``x == y``。
- **L55 EN**: Declares class `EraseTrim`.
  **L55 CN**: 声明 class `EraseTrim`。
- **L56 EN**: Sets the following members to `public` access.
  **L56 CN**: 将后续成员的访问级别设为 `public`。
- **L57 EN**: Executes a standalone statement or declaration: `using mlir::OpRewritePattern<hlfir::CharTrimOp>::OpRewritePattern;`.
  **L57 CN**: 执行一条独立语句或声明：`using mlir::OpRewritePattern<hlfir::CharTrimOp>::OpRewritePattern;`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Continues the surrounding expression or declaration: `llvm::LogicalResult`.
  **L59 CN**: 继续构造周围的表达式或声明：`llvm::LogicalResult`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(hlfir::CharTrimOp trimOp,`.
  **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(hlfir::CharTrimOp trimOp,`。
- **L61 EN**: Continues the surrounding expression or declaration: `mlir::PatternRewriter &rewriter) const override {`.
  **L61 CN**: 继续构造周围的表达式或声明：`mlir::PatternRewriter &rewriter) const override {`。
- **L62 EN**: Initializes variable `trimUses` from the right-hand expression.
  **L62 CN**: 使用右侧表达式初始化变量 `trimUses`。
- **L63 EN**: Initializes variable `cmpCharOp` from the right-hand expression.
  **L63 CN**: 使用右侧表达式初始化变量 `cmpCharOp`。
- **L64 EN**: Initializes variable `destroyOp` from the right-hand expression.
  **L64 CN**: 使用右侧表达式初始化变量 `destroyOp`。

### Lines 65-80

````cpp
    if (!cmpCharOp || !destroyOp || trimUses != 2)
      return rewriter.notifyMatchFailure(
          trimOp, "hlfir.char_trim is not used (only) by hlfir.cmpchar");

    rewriter.eraseOp(destroyOp);
    rewriter.replaceOp(trimOp, trimOp.getChr());
    return mlir::success();
  }
};

class ExpressionSimplificationPass
    : public hlfir::impl::ExpressionSimplificationBase<
          ExpressionSimplificationPass> {
public:
  void runOnOperation() override {
    mlir::MLIRContext *context = &getContext();
````
- **L65 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `if` 控制流语句并计算其条件。
- **L66 EN**: Returns from the current function with `rewriter.notifyMatchFailure(`.
  **L66 CN**: 以 `rewriter.notifyMatchFailure(` 从当前函数返回。
- **L67 EN**: Executes a call or declaration centered on `used`.
  **L67 CN**: 执行以 `used` 为核心的调用或声明。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L69 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L70 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L70 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L71 EN**: Returns from the current function with `mlir::success()`.
  **L71 CN**: 以 `mlir::success()` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L73 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L75 EN**: Declares class `ExpressionSimplificationPass`.
  **L75 CN**: 声明 class `ExpressionSimplificationPass`。
- **L76 EN**: Continues the surrounding expression or declaration: `: public hlfir::impl::ExpressionSimplificationBase<`.
  **L76 CN**: 继续构造周围的表达式或声明：`: public hlfir::impl::ExpressionSimplificationBase<`。
- **L77 EN**: Continues the surrounding expression or declaration: `ExpressionSimplificationPass> {`.
  **L77 CN**: 继续构造周围的表达式或声明：`ExpressionSimplificationPass> {`。
- **L78 EN**: Sets the following members to `public` access.
  **L78 CN**: 将后续成员的访问级别设为 `public`。
- **L79 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L79 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L80 EN**: Executes a call or declaration centered on `&getContext`.
  **L80 CN**: 执行以 `&getContext` 为核心的调用或声明。

### Lines 81-96

````cpp

    mlir::GreedyRewriteConfig config;
    // Prevent the pattern driver from merging blocks.
    config.setRegionSimplificationLevel(
        mlir::GreedySimplifyRegionLevel::Disabled);

    mlir::RewritePatternSet patterns(context);
    patterns.insert<EraseTrim>(context);

    if (mlir::failed(mlir::applyPatternsGreedily(
            getOperation(), std::move(patterns), config))) {
      mlir::emitError(getOperation()->getLoc(),
                      "failure in HLFIR expression simplification");
      signalPassFailure();
    }
  }
````
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Executes a standalone statement or declaration: `mlir::GreedyRewriteConfig config;`.
  **L82 CN**: 执行一条独立语句或声明：`mlir::GreedyRewriteConfig config;`。
- **L83 EN**: Comment explains nearby logic, intent, or metadata: `Prevent the pattern driver from merging blocks.`.
  **L83 CN**: 注释说明附近代码的逻辑、意图或元数据：`Prevent the pattern driver from merging blocks.`。
- **L84 EN**: Continues logic associated with callable symbol `setRegionSimplificationLevel`.
  **L84 CN**: 继续与可调用符号 `setRegionSimplificationLevel` 相关的逻辑。
- **L85 EN**: Executes a standalone statement or declaration: `mlir::GreedySimplifyRegionLevel::Disabled);`.
  **L85 CN**: 执行一条独立语句或声明：`mlir::GreedySimplifyRegionLevel::Disabled);`。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Executes a call or declaration centered on `patterns`.
  **L87 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L88 EN**: Executes a call or declaration centered on `patterns.insert<EraseTrim>`.
  **L88 CN**: 执行以 `patterns.insert<EraseTrim>` 为核心的调用或声明。
- **L89 EN**: Blank line separating nearby declarations or logic blocks.
  **L89 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L90 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L90 CN**: 开始 `if` 控制流语句并计算其条件。
- **L91 EN**: Starts a function, method, lambda, or structured scope: `getOperation(), std::move(patterns), config))) {`.
  **L91 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getOperation(), std::move(patterns), config))) {`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::emitError(getOperation()->getLoc(),`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::emitError(getOperation()->getLoc(),`。
- **L93 EN**: Executes a standalone statement or declaration: `"failure in HLFIR expression simplification");`.
  **L93 CN**: 执行一条独立语句或声明：`"failure in HLFIR expression simplification");`。
- **L94 EN**: Executes a call or declaration centered on `signalPassFailure`.
  **L94 CN**: 执行以 `signalPassFailure` 为核心的调用或声明。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-99

````cpp
};

} // namespace
````
- **L97 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L97 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L99 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **HLFIR-specific abstractions / HLFIR 专用抽象**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **Pattern-driven IR rewriting / 基于模式的 IR 重写**
- **Operation rewrite patterns / 操作重写模式**
- **IR builder orchestration / IR Builder 编排**
- **Driver-level compilation flow / 驱动级编译流程**
- **Expression-centric processing / 以表达式为中心的处理**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/HLFIR/HLFIROps.h`: Provides HLFIR abstractions and transformation support. / 提供HLFIR 抽象与变换支持。
- `flang/Optimizer/HLFIR/Passes.h`: Provides HLFIR abstractions and transformation support. / 提供HLFIR 抽象与变换支持。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `flang/Optimizer/HLFIR/Passes.h.inc`: Provides HLFIR abstractions and transformation support. / 提供HLFIR 抽象与变换支持。
