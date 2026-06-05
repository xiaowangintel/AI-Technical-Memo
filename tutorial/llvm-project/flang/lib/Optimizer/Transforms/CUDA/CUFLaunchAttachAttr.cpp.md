# CUFLaunchAttachAttr.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/CUDA/CUFLaunchAttachAttr.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for CUF Launch Attach Attr.
- **Purpose (CN)**: 实现 CUF Launch Attach Attr 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

````cpp
//===-- CUFLaunchAttachAttr.cpp -------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Dialect/CUF/CUFDialect.h"
#include "flang/Optimizer/Dialect/FIROps.h"
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Transforms/DialectConversion.h"
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
- **L9 EN**: Includes "flang/Optimizer/Dialect/CUF/CUFDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L9 CN**: 引入 "flang/Optimizer/Dialect/CUF/CUFDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L10 EN**: Includes "flang/Optimizer/Dialect/FIROps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L10 CN**: 引入 "flang/Optimizer/Dialect/FIROps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L11 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L12 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L13 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L13 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L14 EN**: Includes "mlir/Transforms/DialectConversion.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L14 CN**: 引入 "mlir/Transforms/DialectConversion.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 15-28

````cpp

namespace fir {
#define GEN_PASS_DEF_CUFLAUNCHATTACHATTR
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

using namespace mlir;

namespace {

static constexpr llvm::StringRef cudaKernelInfix = "_cufk_";

class CUFGPUAttachAttrPattern
    : public OpRewritePattern<mlir::gpu::LaunchFuncOp> {
````
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Opens namespace scope `fir`.
  **L16 CN**: 打开命名空间作用域 `fir`。
- **L17 EN**: Defines macro `GEN_PASS_DEF_CUFLAUNCHATTACHATTR` for conditional compilation or local shorthand.
  **L17 CN**: 定义宏 `GEN_PASS_DEF_CUFLAUNCHATTACHATTR`，用于条件编译或本地简写。
- **L18 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L18 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L19 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L19 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Brings namespace `mlir` into the local scope.
  **L21 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L23 EN**: Opens namespace scope ``.
  **L23 CN**: 打开命名空间作用域 ``。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Initializes variable `cudaKernelInfix` from the right-hand expression.
  **L25 CN**: 使用右侧表达式初始化变量 `cudaKernelInfix`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Declares class `CUFGPUAttachAttrPattern`.
  **L27 CN**: 声明 class `CUFGPUAttachAttrPattern`。
- **L28 EN**: Continues the surrounding expression or declaration: `: public OpRewritePattern<mlir::gpu::LaunchFuncOp> {`.
  **L28 CN**: 继续构造周围的表达式或声明：`: public OpRewritePattern<mlir::gpu::LaunchFuncOp> {`。

### Lines 29-42

````cpp
  using OpRewritePattern<mlir::gpu::LaunchFuncOp>::OpRewritePattern;
  LogicalResult matchAndRewrite(mlir::gpu::LaunchFuncOp op,
                                PatternRewriter &rewriter) const override {
    op->setAttr(cuf::getProcAttrName(),
                cuf::ProcAttributeAttr::get(op.getContext(),
                                            cuf::ProcAttribute::Global));
    return mlir::success();
  }
};

struct CUFLaunchAttachAttr
    : public fir::impl::CUFLaunchAttachAttrBase<CUFLaunchAttachAttr> {

  void runOnOperation() override {
````
- **L29 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<mlir::gpu::LaunchFuncOp>::OpRewritePattern;`.
  **L29 CN**: 执行一条独立语句或声明：`using OpRewritePattern<mlir::gpu::LaunchFuncOp>::OpRewritePattern;`。
- **L30 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(mlir::gpu::LaunchFuncOp op,`.
  **L30 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(mlir::gpu::LaunchFuncOp op,`。
- **L31 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L31 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `op->setAttr(cuf::getProcAttrName(),`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`op->setAttr(cuf::getProcAttrName(),`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cuf::ProcAttributeAttr::get(op.getContext(),`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`cuf::ProcAttributeAttr::get(op.getContext(),`。
- **L34 EN**: Executes a standalone statement or declaration: `cuf::ProcAttribute::Global));`.
  **L34 CN**: 执行一条独立语句或声明：`cuf::ProcAttribute::Global));`。
- **L35 EN**: Returns from the current function with `mlir::success()`.
  **L35 CN**: 以 `mlir::success()` 从当前函数返回。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L37 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L39 EN**: Declares struct `CUFLaunchAttachAttr`.
  **L39 CN**: 声明 struct `CUFLaunchAttachAttr`。
- **L40 EN**: Continues the surrounding expression or declaration: `: public fir::impl::CUFLaunchAttachAttrBase<CUFLaunchAttachAttr> {`.
  **L40 CN**: 继续构造周围的表达式或声明：`: public fir::impl::CUFLaunchAttachAttrBase<CUFLaunchAttachAttr> {`。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L42 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L42 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。

### Lines 43-56

````cpp
    auto *context = &this->getContext();

    mlir::RewritePatternSet patterns(context);
    patterns.add<CUFGPUAttachAttrPattern>(context);

    mlir::ConversionTarget target(*context);
    target.addIllegalOp<mlir::gpu::LaunchFuncOp>();
    target.addDynamicallyLegalOp<mlir::gpu::LaunchFuncOp>(
        [&](mlir::gpu::LaunchFuncOp op) -> bool {
          if (op.getKernelName().getValue().contains(cudaKernelInfix)) {
            if (op.getOperation()->getAttrOfType<cuf::ProcAttributeAttr>(
                    cuf::getProcAttrName()))
              return true;
            return false;
````
- **L43 EN**: Executes a call or declaration centered on `&this->getContext`.
  **L43 CN**: 执行以 `&this->getContext` 为核心的调用或声明。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Executes a call or declaration centered on `patterns`.
  **L45 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L46 EN**: Executes a call or declaration centered on `patterns.add<CUFGPUAttachAttrPattern>`.
  **L46 CN**: 执行以 `patterns.add<CUFGPUAttachAttrPattern>` 为核心的调用或声明。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Executes a call or declaration centered on `target`.
  **L48 CN**: 执行以 `target` 为核心的调用或声明。
- **L49 EN**: Executes a call or declaration centered on `target.addIllegalOp<mlir::gpu::LaunchFuncOp>`.
  **L49 CN**: 执行以 `target.addIllegalOp<mlir::gpu::LaunchFuncOp>` 为核心的调用或声明。
- **L50 EN**: Continues logic associated with callable symbol `LaunchFuncOp>`.
  **L50 CN**: 继续与可调用符号 `LaunchFuncOp>` 相关的逻辑。
- **L51 EN**: Starts a function, method, lambda, or structured scope: `[&](mlir::gpu::LaunchFuncOp op) -> bool {`.
  **L51 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](mlir::gpu::LaunchFuncOp op) -> bool {`。
- **L52 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L52 CN**: 开始 `if` 控制流语句并计算其条件。
- **L53 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L53 CN**: 开始 `if` 控制流语句并计算其条件。
- **L54 EN**: Continues logic associated with callable symbol `getProcAttrName`.
  **L54 CN**: 继续与可调用符号 `getProcAttrName` 相关的逻辑。
- **L55 EN**: Returns from the current function with `true`.
  **L55 CN**: 以 `true` 从当前函数返回。
- **L56 EN**: Returns from the current function with `false`.
  **L56 CN**: 以 `false` 从当前函数返回。

### Lines 57-70

````cpp
          }
          return true;
        });

    if (mlir::failed(mlir::applyPartialConversion(this->getOperation(), target,
                                                  std::move(patterns)))) {
      mlir::emitError(mlir::UnknownLoc::get(context),
                      "Pattern conversion failed\n");
      this->signalPassFailure();
    }
  }
};

} // end anonymous namespace
````
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Returns from the current function with `true`.
  **L58 CN**: 以 `true` 从当前函数返回。
- **L59 EN**: Executes a standalone statement or declaration: `});`.
  **L59 CN**: 执行一条独立语句或声明：`});`。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L61 CN**: 开始 `if` 控制流语句并计算其条件。
- **L62 EN**: Starts a function, method, lambda, or structured scope: `std::move(patterns)))) {`.
  **L62 CN**: 开始一个函数、方法、lambda 或结构化作用域：`std::move(patterns)))) {`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::emitError(mlir::UnknownLoc::get(context),`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::emitError(mlir::UnknownLoc::get(context),`。
- **L64 EN**: Executes a standalone statement or declaration: `"Pattern conversion failed\n");`.
  **L64 CN**: 执行一条独立语句或声明：`"Pattern conversion failed\n");`。
- **L65 EN**: Executes a call or declaration centered on `this->signalPassFailure`.
  **L65 CN**: 执行以 `this->signalPassFailure` 为核心的调用或声明。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L68 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Continues the surrounding expression or declaration: `} // end anonymous namespace`.
  **L70 CN**: 继续构造周围的表达式或声明：`} // end anonymous namespace`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **Pattern-driven IR rewriting / 基于模式的 IR 重写**
- **Dialect conversion targeting / 方言转换目标设定**
- **Operation rewrite patterns / 操作重写模式**

## Dependencies / 依赖关系

- `flang/Optimizer/Dialect/CUF/CUFDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROps.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROpsSupport.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Pass/Pass.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Transforms/DialectConversion.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `flang/Optimizer/Transforms/Passes.h.inc`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
