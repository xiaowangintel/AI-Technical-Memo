# CUFFunctionRewrite.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/CUDA/CUFFunctionRewrite.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for CUF Function Rewrite.
- **Purpose (CN)**: 实现 CUF Function Rewrite 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===-- CUFFunctionRewrite.cpp --------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/CodeGen/TypeConverter.h"
#include "flang/Optimizer/Dialect/FIRDialect.h"
#include "flang/Optimizer/Dialect/FIROps.h"
#include "flang/Optimizer/Dialect/FIRType.h"
#include "flang/Optimizer/Support/DataLayout.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/IR/BuiltinTypes.h"
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
- **L9 EN**: Includes "flang/Optimizer/CodeGen/TypeConverter.h" to access local declarations paired with this implementation.
  **L9 CN**: 引入 "flang/Optimizer/CodeGen/TypeConverter.h" 以使用与该实现配套的本地声明。
- **L10 EN**: Includes "flang/Optimizer/Dialect/FIRDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L10 CN**: 引入 "flang/Optimizer/Dialect/FIRDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L11 EN**: Includes "flang/Optimizer/Dialect/FIROps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/FIROps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "flang/Optimizer/Dialect/FIRType.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L12 CN**: 引入 "flang/Optimizer/Dialect/FIRType.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L13 EN**: Includes "flang/Optimizer/Support/DataLayout.h" to access optimizer-side support routines and utilities.
  **L13 CN**: 引入 "flang/Optimizer/Support/DataLayout.h" 以使用优化器侧支持例程与工具。
- **L14 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L14 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L15 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L15 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L16 EN**: Includes "mlir/IR/BuiltinTypes.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/IR/BuiltinTypes.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 17-32

````cpp
#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/IR/ValueRange.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Support/LogicalResult.h"
#include "mlir/Transforms/DialectConversion.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSet.h"
#include "llvm/Support/Debug.h"
#include <string_view>

#define DEBUG_TYPE "flang-cuf-function-rewrite"

namespace fir {
````
- **L17 EN**: Includes "mlir/IR/MLIRContext.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/IR/MLIRContext.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "mlir/IR/PatternMatch.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/IR/PatternMatch.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L19 EN**: Includes "mlir/IR/ValueRange.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L19 CN**: 引入 "mlir/IR/ValueRange.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L20 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L20 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L21 EN**: Includes "mlir/Support/LogicalResult.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L21 CN**: 引入 "mlir/Support/LogicalResult.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L22 EN**: Includes "mlir/Transforms/DialectConversion.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L22 CN**: 引入 "mlir/Transforms/DialectConversion.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L23 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L23 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L24 EN**: Includes "llvm/ADT/SmallVector.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L24 CN**: 引入 "llvm/ADT/SmallVector.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L25 EN**: Includes "llvm/ADT/StringRef.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L25 CN**: 引入 "llvm/ADT/StringRef.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L26 EN**: Includes "llvm/ADT/StringSet.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L26 CN**: 引入 "llvm/ADT/StringSet.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L27 EN**: Includes "llvm/Support/Debug.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L27 CN**: 引入 "llvm/Support/Debug.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L28 EN**: Includes <string_view> to access supporting declarations used by this translation unit.
  **L28 CN**: 引入 <string_view> 以使用当前编译单元使用的辅助声明。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Defines macro `DEBUG_TYPE` for conditional compilation or local shorthand.
  **L30 CN**: 定义宏 `DEBUG_TYPE`，用于条件编译或本地简写。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Opens namespace scope `fir`.
  **L32 CN**: 打开命名空间作用域 `fir`。

### Lines 33-48

````cpp
#define GEN_PASS_DEF_CUFFUNCTIONREWRITE
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

using namespace mlir;

namespace {

using genFunctionType =
    std::function<mlir::Value(mlir::PatternRewriter &, fir::CallOp op)>;

class CallConversion : public OpRewritePattern<fir::CallOp> {
public:
  CallConversion(MLIRContext *context)
      : OpRewritePattern<fir::CallOp>(context) {}

````
- **L33 EN**: Defines macro `GEN_PASS_DEF_CUFFUNCTIONREWRITE` for conditional compilation or local shorthand.
  **L33 CN**: 定义宏 `GEN_PASS_DEF_CUFFUNCTIONREWRITE`，用于条件编译或本地简写。
- **L34 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L34 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L35 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L35 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L37 EN**: Brings namespace `mlir` into the local scope.
  **L37 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Opens namespace scope ``.
  **L39 CN**: 打开命名空间作用域 ``。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L41 EN**: Defines alias `genFunctionType` to simplify later code.
  **L41 CN**: 定义别名 `genFunctionType` 以简化后续代码。
- **L42 EN**: Executes a call or declaration centered on `std::function<mlir::Value`.
  **L42 CN**: 执行以 `std::function<mlir::Value` 为核心的调用或声明。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Declares class `CallConversion`.
  **L44 CN**: 声明 class `CallConversion`。
- **L45 EN**: Sets the following members to `public` access.
  **L45 CN**: 将后续成员的访问级别设为 `public`。
- **L46 EN**: Continues logic associated with callable symbol `CallConversion`.
  **L46 CN**: 继续与可调用符号 `CallConversion` 相关的逻辑。
- **L47 EN**: Continues logic associated with callable symbol `CallOp>`.
  **L47 CN**: 继续与可调用符号 `CallOp>` 相关的逻辑。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 49-64

````cpp
  LogicalResult
  matchAndRewrite(fir::CallOp op,
                  mlir::PatternRewriter &rewriter) const override {
    auto callee = op.getCallee();
    if (!callee)
      return failure();
    auto name = callee->getRootReference().getValue();

    if (genMappings_.contains(name)) {
      auto fct = genMappings_.find(name);
      mlir::Value result = fct->second(rewriter, op);
      if (result)
        rewriter.replaceOp(op, result);
      else
        rewriter.eraseOp(op);
      return success();
````
- **L49 EN**: Continues the surrounding expression or declaration: `LogicalResult`.
  **L49 CN**: 继续构造周围的表达式或声明：`LogicalResult`。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(fir::CallOp op,`.
  **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(fir::CallOp op,`。
- **L51 EN**: Continues the surrounding expression or declaration: `mlir::PatternRewriter &rewriter) const override {`.
  **L51 CN**: 继续构造周围的表达式或声明：`mlir::PatternRewriter &rewriter) const override {`。
- **L52 EN**: Initializes variable `callee` from the right-hand expression.
  **L52 CN**: 使用右侧表达式初始化变量 `callee`。
- **L53 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L53 CN**: 开始 `if` 控制流语句并计算其条件。
- **L54 EN**: Returns from the current function with `failure()`.
  **L54 CN**: 以 `failure()` 从当前函数返回。
- **L55 EN**: Initializes variable `name` from the right-hand expression.
  **L55 CN**: 使用右侧表达式初始化变量 `name`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Initializes variable `fct` from the right-hand expression.
  **L58 CN**: 使用右侧表达式初始化变量 `fct`。
- **L59 EN**: Initializes variable `result` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `result`。
- **L60 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L60 CN**: 开始 `if` 控制流语句并计算其条件。
- **L61 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L61 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L62 EN**: Transitions from the previous branch into the alternative path.
  **L62 CN**: 从前一个分支过渡到备选路径。
- **L63 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L63 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L64 EN**: Returns from the current function with `success()`.
  **L64 CN**: 以 `success()` 从当前函数返回。

### Lines 65-80

````cpp
    }
    return failure();
  }

private:
  static mlir::Value genOnDevice(mlir::PatternRewriter &rewriter,
                                 fir::CallOp op) {
    assert(op.getArgs().size() == 0 && "expect 0 arguments");
    mlir::Location loc = op.getLoc();
    unsigned inGPUMod = op->getParentOfType<gpu::GPUModuleOp>() ? 1 : 0;
    mlir::Type i1Ty = rewriter.getIntegerType(1);
    mlir::Value t = mlir::arith::ConstantOp::create(
        rewriter, loc, i1Ty, rewriter.getIntegerAttr(i1Ty, inGPUMod));
    return fir::ConvertOp::create(rewriter, loc, op.getResult(0).getType(), t);
  }

````
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Returns from the current function with `failure()`.
  **L66 CN**: 以 `failure()` 从当前函数返回。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Sets the following members to `private` access.
  **L69 CN**: 将后续成员的访问级别设为 `private`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::Value genOnDevice(mlir::PatternRewriter &rewriter,`.
  **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::Value genOnDevice(mlir::PatternRewriter &rewriter,`。
- **L71 EN**: Continues the surrounding expression or declaration: `fir::CallOp op) {`.
  **L71 CN**: 继续构造周围的表达式或声明：`fir::CallOp op) {`。
- **L72 EN**: Checks an internal invariant in debug builds.
  **L72 CN**: 在调试构建中检查内部不变式。
- **L73 EN**: Initializes variable `loc` from the right-hand expression.
  **L73 CN**: 使用右侧表达式初始化变量 `loc`。
- **L74 EN**: Initializes variable `inGPUMod` from the right-hand expression.
  **L74 CN**: 使用右侧表达式初始化变量 `inGPUMod`。
- **L75 EN**: Initializes variable `i1Ty` from the right-hand expression.
  **L75 CN**: 使用右侧表达式初始化变量 `i1Ty`。
- **L76 EN**: Continues logic associated with callable symbol `create`.
  **L76 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L77 EN**: Executes a call or declaration centered on `rewriter.getIntegerAttr`.
  **L77 CN**: 执行以 `rewriter.getIntegerAttr` 为核心的调用或声明。
- **L78 EN**: Returns from the current function with `fir::ConvertOp::create(rewriter, loc, op.getResult(0).getType(), t)`.
  **L78 CN**: 以 `fir::ConvertOp::create(rewriter, loc, op.getResult(0).getType(), t)` 从当前函数返回。
- **L79 EN**: Closes the current lexical scope or compound statement.
  **L79 CN**: 结束当前词法作用域或复合语句块。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 81-96

````cpp
  const llvm::StringMap<genFunctionType> genMappings_ = {
      {"on_device", &genOnDevice}};
};

class CUFFunctionRewrite
    : public fir::impl::CUFFunctionRewriteBase<CUFFunctionRewrite> {
public:
  void runOnOperation() override {
    auto *ctx = &getContext();
    mlir::RewritePatternSet patterns(ctx);

    patterns.insert<CallConversion>(patterns.getContext());

    if (mlir::failed(
            mlir::applyPatternsGreedily(getOperation(), std::move(patterns)))) {
      mlir::emitError(mlir::UnknownLoc::get(ctx),
````
- **L81 EN**: Continues the surrounding expression or declaration: `const llvm::StringMap<genFunctionType> genMappings_ = {`.
  **L81 CN**: 继续构造周围的表达式或声明：`const llvm::StringMap<genFunctionType> genMappings_ = {`。
- **L82 EN**: Executes a standalone statement or declaration: `{"on_device", &genOnDevice}};`.
  **L82 CN**: 执行一条独立语句或声明：`{"on_device", &genOnDevice}};`。
- **L83 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L83 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Declares class `CUFFunctionRewrite`.
  **L85 CN**: 声明 class `CUFFunctionRewrite`。
- **L86 EN**: Continues the surrounding expression or declaration: `: public fir::impl::CUFFunctionRewriteBase<CUFFunctionRewrite> {`.
  **L86 CN**: 继续构造周围的表达式或声明：`: public fir::impl::CUFFunctionRewriteBase<CUFFunctionRewrite> {`。
- **L87 EN**: Sets the following members to `public` access.
  **L87 CN**: 将后续成员的访问级别设为 `public`。
- **L88 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L88 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L89 EN**: Executes a call or declaration centered on `&getContext`.
  **L89 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L90 EN**: Executes a call or declaration centered on `patterns`.
  **L90 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Executes a call or declaration centered on `patterns.insert<CallConversion>`.
  **L92 CN**: 执行以 `patterns.insert<CallConversion>` 为核心的调用或声明。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L94 CN**: 开始 `if` 控制流语句并计算其条件。
- **L95 EN**: Starts a function, method, lambda, or structured scope: `mlir::applyPatternsGreedily(getOperation(), std::move(patterns)))) {`.
  **L95 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::applyPatternsGreedily(getOperation(), std::move(patterns)))) {`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::emitError(mlir::UnknownLoc::get(ctx),`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::emitError(mlir::UnknownLoc::get(ctx),`。

### Lines 97-103

````cpp
                      "error in CUFFunctionRewrite op conversion\n");
      signalPassFailure();
    }
  }
};

} // namespace
````
- **L97 EN**: Executes a standalone statement or declaration: `"error in CUFFunctionRewrite op conversion\n");`.
  **L97 CN**: 执行一条独立语句或声明：`"error in CUFFunctionRewrite op conversion\n");`。
- **L98 EN**: Executes a call or declaration centered on `signalPassFailure`.
  **L98 CN**: 执行以 `signalPassFailure` 为核心的调用或声明。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。
- **L101 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L101 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L103 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L103 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **Pattern-driven IR rewriting / 基于模式的 IR 重写**
- **Operation rewrite patterns / 操作重写模式**
- **Type conversion rules / 类型转换规则**
- **Driver-level compilation flow / 驱动级编译流程**

## Dependencies / 依赖关系

- `flang/Optimizer/CodeGen/TypeConverter.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Optimizer/Dialect/FIRDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROps.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIRType.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Support/DataLayout.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/BuiltinTypes.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/MLIRContext.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/PatternMatch.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/ValueRange.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Pass/Pass.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Support/LogicalResult.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Transforms/DialectConversion.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `llvm/ADT/SmallVector.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
