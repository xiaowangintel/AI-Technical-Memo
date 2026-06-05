# ConstantArgumentGlobalisation.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/ConstantArgumentGlobalisation.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for Constant Argument Globalisation.
- **Purpose (CN)**: 实现 Constant Argument Globalisation 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===- ConstantArgumentGlobalisation.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Dialect/FIRDialect.h"
#include "flang/Optimizer/Dialect/FIROps.h"
#include "flang/Optimizer/Dialect/FIRType.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Dominance.h"
#include "mlir/Pass/Pass.h"
#include "mlir/Transforms/GreedyPatternRewriteDriver.h"
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
- **L10 EN**: Includes "flang/Optimizer/Dialect/FIRDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L10 CN**: 引入 "flang/Optimizer/Dialect/FIRDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L11 EN**: Includes "flang/Optimizer/Dialect/FIROps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/FIROps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "flang/Optimizer/Dialect/FIRType.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L12 CN**: 引入 "flang/Optimizer/Dialect/FIRType.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L13 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L13 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L14 EN**: Includes "mlir/Dialect/Func/IR/FuncOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L14 CN**: 引入 "mlir/Dialect/Func/IR/FuncOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L15 EN**: Includes "mlir/IR/Diagnostics.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L15 CN**: 引入 "mlir/IR/Diagnostics.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L16 EN**: Includes "mlir/IR/Dominance.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/IR/Dominance.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L17 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "mlir/Transforms/GreedyPatternRewriteDriver.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/Transforms/GreedyPatternRewriteDriver.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 19-36

````cpp

namespace fir {
#define GEN_PASS_DEF_CONSTANTARGUMENTGLOBALISATIONOPT
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

#define DEBUG_TYPE "flang-constant-argument-globalisation-opt"

namespace {
unsigned uniqueLitId = 1;

class CallOpRewriter : public mlir::OpRewritePattern<fir::CallOp> {
protected:
  const mlir::DominanceInfo &di;

public:
  using OpRewritePattern::OpRewritePattern;

````
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `fir`.
  **L20 CN**: 打开命名空间作用域 `fir`。
- **L21 EN**: Defines macro `GEN_PASS_DEF_CONSTANTARGUMENTGLOBALISATIONOPT` for conditional compilation or local shorthand.
  **L21 CN**: 定义宏 `GEN_PASS_DEF_CONSTANTARGUMENTGLOBALISATIONOPT`，用于条件编译或本地简写。
- **L22 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L22 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L23 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L23 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Defines macro `DEBUG_TYPE` for conditional compilation or local shorthand.
  **L25 CN**: 定义宏 `DEBUG_TYPE`，用于条件编译或本地简写。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Opens namespace scope ``.
  **L27 CN**: 打开命名空间作用域 ``。
- **L28 EN**: Initializes variable `uniqueLitId` from the right-hand expression.
  **L28 CN**: 使用右侧表达式初始化变量 `uniqueLitId`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Declares class `CallOpRewriter`.
  **L30 CN**: 声明 class `CallOpRewriter`。
- **L31 EN**: Sets the following members to `protected` access.
  **L31 CN**: 将后续成员的访问级别设为 `protected`。
- **L32 EN**: Executes a standalone statement or declaration: `const mlir::DominanceInfo &di;`.
  **L32 CN**: 执行一条独立语句或声明：`const mlir::DominanceInfo &di;`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Sets the following members to `public` access.
  **L34 CN**: 将后续成员的访问级别设为 `public`。
- **L35 EN**: Executes a standalone statement or declaration: `using OpRewritePattern::OpRewritePattern;`.
  **L35 CN**: 执行一条独立语句或声明：`using OpRewritePattern::OpRewritePattern;`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 37-54

````cpp
  CallOpRewriter(mlir::MLIRContext *ctx, const mlir::DominanceInfo &_di)
      : OpRewritePattern(ctx), di(_di) {}

  llvm::LogicalResult
  matchAndRewrite(fir::CallOp callOp,
                  mlir::PatternRewriter &rewriter) const override {
    LLVM_DEBUG(llvm::dbgs() << "Processing call op: " << callOp << "\n");
    auto module = callOp->getParentOfType<mlir::ModuleOp>();
    bool needUpdate = false;
    fir::FirOpBuilder builder(rewriter, module);
    llvm::SmallVector<mlir::Value> newOperands;
    llvm::SmallVector<std::pair<mlir::Operation *, mlir::Operation *>> allocas;
    for (const mlir::Value &a : callOp.getArgs()) {
      auto alloca = mlir::dyn_cast_or_null<fir::AllocaOp>(a.getDefiningOp());
      // We can convert arguments that are alloca, and that has
      // the value by reference attribute. All else is just added
      // to the argument list.
      if (!alloca || !alloca->hasAttr(fir::getAdaptToByRefAttrName())) {
````
- **L37 EN**: Continues logic associated with callable symbol `CallOpRewriter`.
  **L37 CN**: 继续与可调用符号 `CallOpRewriter` 相关的逻辑。
- **L38 EN**: Continues logic associated with callable symbol `OpRewritePattern`.
  **L38 CN**: 继续与可调用符号 `OpRewritePattern` 相关的逻辑。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Continues the surrounding expression or declaration: `llvm::LogicalResult`.
  **L40 CN**: 继续构造周围的表达式或声明：`llvm::LogicalResult`。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `matchAndRewrite(fir::CallOp callOp,`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`matchAndRewrite(fir::CallOp callOp,`。
- **L42 EN**: Continues the surrounding expression or declaration: `mlir::PatternRewriter &rewriter) const override {`.
  **L42 CN**: 继续构造周围的表达式或声明：`mlir::PatternRewriter &rewriter) const override {`。
- **L43 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L43 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L44 EN**: Initializes variable `module` from the right-hand expression.
  **L44 CN**: 使用右侧表达式初始化变量 `module`。
- **L45 EN**: Initializes variable `needUpdate` from the right-hand expression.
  **L45 CN**: 使用右侧表达式初始化变量 `needUpdate`。
- **L46 EN**: Executes a call or declaration centered on `builder`.
  **L46 CN**: 执行以 `builder` 为核心的调用或声明。
- **L47 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Value> newOperands;`.
  **L47 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Value> newOperands;`。
- **L48 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<std::pair<mlir::Operation *, mlir::Operation *>> allocas;`.
  **L48 CN**: 执行一条独立语句或声明：`llvm::SmallVector<std::pair<mlir::Operation *, mlir::Operation *>> allocas;`。
- **L49 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L49 CN**: 开始 `for` 控制流语句并计算其条件。
- **L50 EN**: Initializes variable `alloca` from the right-hand expression.
  **L50 CN**: 使用右侧表达式初始化变量 `alloca`。
- **L51 EN**: Comment explains nearby logic, intent, or metadata: `We can convert arguments that are alloca, and that has`.
  **L51 CN**: 注释说明附近代码的逻辑、意图或元数据：`We can convert arguments that are alloca, and that has`。
- **L52 EN**: Comment explains nearby logic, intent, or metadata: `the value by reference attribute. All else is just added`.
  **L52 CN**: 注释说明附近代码的逻辑、意图或元数据：`the value by reference attribute. All else is just added`。
- **L53 EN**: Comment explains nearby logic, intent, or metadata: `to the argument list.`.
  **L53 CN**: 注释说明附近代码的逻辑、意图或元数据：`to the argument list.`。
- **L54 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L54 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 55-72

````cpp
        newOperands.push_back(a);
        continue;
      }

      mlir::Type varTy = alloca.getInType();
      assert(!fir::hasDynamicSize(varTy) &&
             "only expect statically sized scalars to be by value");

      // Find immediate store with const argument
      mlir::Operation *store = nullptr;
      for (mlir::Operation *s : alloca->getUsers()) {
        if (mlir::isa<fir::StoreOp>(s) && di.dominates(s, callOp)) {
          // We can only deal with ONE store - if already found one,
          // set to nullptr and exit the loop.
          if (store) {
            store = nullptr;
            break;
          }
````
- **L55 EN**: Executes a call or declaration centered on `newOperands.push_back`.
  **L55 CN**: 执行以 `newOperands.push_back` 为核心的调用或声明。
- **L56 EN**: Skips to the next loop iteration.
  **L56 CN**: 跳到下一次循环迭代。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Initializes variable `varTy` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `varTy`。
- **L60 EN**: Checks an internal invariant in debug builds.
  **L60 CN**: 在调试构建中检查内部不变式。
- **L61 EN**: Executes a standalone statement or declaration: `"only expect statically sized scalars to be by value");`.
  **L61 CN**: 执行一条独立语句或声明：`"only expect statically sized scalars to be by value");`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Comment explains nearby logic, intent, or metadata: `Find immediate store with const argument`.
  **L63 CN**: 注释说明附近代码的逻辑、意图或元数据：`Find immediate store with const argument`。
- **L64 EN**: Executes a standalone statement or declaration: `mlir::Operation *store = nullptr;`.
  **L64 CN**: 执行一条独立语句或声明：`mlir::Operation *store = nullptr;`。
- **L65 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L65 CN**: 开始 `for` 控制流语句并计算其条件。
- **L66 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L66 CN**: 开始 `if` 控制流语句并计算其条件。
- **L67 EN**: Comment explains nearby logic, intent, or metadata: `We can only deal with ONE store - if already found one,`.
  **L67 CN**: 注释说明附近代码的逻辑、意图或元数据：`We can only deal with ONE store - if already found one,`。
- **L68 EN**: Comment explains nearby logic, intent, or metadata: `set to nullptr and exit the loop.`.
  **L68 CN**: 注释说明附近代码的逻辑、意图或元数据：`set to nullptr and exit the loop.`。
- **L69 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L69 CN**: 开始 `if` 控制流语句并计算其条件。
- **L70 EN**: Executes a standalone statement or declaration: `store = nullptr;`.
  **L70 CN**: 执行一条独立语句或声明：`store = nullptr;`。
- **L71 EN**: Exits the nearest loop or switch statement.
  **L71 CN**: 退出最近的循环或 switch 语句。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-90

````cpp
          store = s;
        }
      }

      // If we didn't find any store, or multiple stores, add argument as is
      // and move on.
      if (!store) {
        newOperands.push_back(a);
        continue;
      }

      LLVM_DEBUG(llvm::dbgs() << " found store " << *store << "\n");

      mlir::Operation *definingOp = store->getOperand(0).getDefiningOp();
      // If not a constant, add to operands and move on.
      if (!mlir::isa<mlir::arith::ConstantOp>(definingOp)) {
        // Unable to remove alloca arg
        newOperands.push_back(a);
````
- **L73 EN**: Executes a standalone statement or declaration: `store = s;`.
  **L73 CN**: 执行一条独立语句或声明：`store = s;`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Comment explains nearby logic, intent, or metadata: `If we didn't find any store, or multiple stores, add argument as is`.
  **L77 CN**: 注释说明附近代码的逻辑、意图或元数据：`If we didn't find any store, or multiple stores, add argument as is`。
- **L78 EN**: Comment explains nearby logic, intent, or metadata: `and move on.`.
  **L78 CN**: 注释说明附近代码的逻辑、意图或元数据：`and move on.`。
- **L79 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L79 CN**: 开始 `if` 控制流语句并计算其条件。
- **L80 EN**: Executes a call or declaration centered on `newOperands.push_back`.
  **L80 CN**: 执行以 `newOperands.push_back` 为核心的调用或声明。
- **L81 EN**: Skips to the next loop iteration.
  **L81 CN**: 跳到下一次循环迭代。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L84 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Executes a call or declaration centered on `store->getOperand`.
  **L86 CN**: 执行以 `store->getOperand` 为核心的调用或声明。
- **L87 EN**: Comment explains nearby logic, intent, or metadata: `If not a constant, add to operands and move on.`.
  **L87 CN**: 注释说明附近代码的逻辑、意图或元数据：`If not a constant, add to operands and move on.`。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Comment explains nearby logic, intent, or metadata: `Unable to remove alloca arg`.
  **L89 CN**: 注释说明附近代码的逻辑、意图或元数据：`Unable to remove alloca arg`。
- **L90 EN**: Executes a call or declaration centered on `newOperands.push_back`.
  **L90 CN**: 执行以 `newOperands.push_back` 为核心的调用或声明。

### Lines 91-108

````cpp
        continue;
      }

      LLVM_DEBUG(llvm::dbgs() << " found define " << *definingOp << "\n");

      std::string globalName =
          "_global_const_." + std::to_string(uniqueLitId++);
      assert(!builder.getNamedGlobal(globalName) &&
             "We should have a unique name here");

      if (llvm::none_of(allocas,
                        [alloca](auto x) { return x.first == alloca; })) {
        allocas.push_back(std::make_pair(alloca, store));
      }

      auto loc = callOp.getLoc();
      fir::GlobalOp global = builder.createGlobalConstant(
          loc, varTy, globalName,
````
- **L91 EN**: Skips to the next loop iteration.
  **L91 CN**: 跳到下一次循环迭代。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L94 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L94 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Continues the surrounding expression or declaration: `std::string globalName =`.
  **L96 CN**: 继续构造周围的表达式或声明：`std::string globalName =`。
- **L97 EN**: Executes a call or declaration centered on `std::to_string`.
  **L97 CN**: 执行以 `std::to_string` 为核心的调用或声明。
- **L98 EN**: Checks an internal invariant in debug builds.
  **L98 CN**: 在调试构建中检查内部不变式。
- **L99 EN**: Executes a standalone statement or declaration: `"We should have a unique name here");`.
  **L99 CN**: 执行一条独立语句或声明：`"We should have a unique name here");`。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L101 CN**: 开始 `if` 控制流语句并计算其条件。
- **L102 EN**: Starts a function, method, lambda, or structured scope: `[alloca](auto x) { return x.first == alloca; })) {`.
  **L102 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[alloca](auto x) { return x.first == alloca; })) {`。
- **L103 EN**: Executes a call or declaration centered on `allocas.push_back`.
  **L103 CN**: 执行以 `allocas.push_back` 为核心的调用或声明。
- **L104 EN**: Closes the current lexical scope or compound statement.
  **L104 CN**: 结束当前词法作用域或复合语句块。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L106 EN**: Initializes variable `loc` from the right-hand expression.
  **L106 CN**: 使用右侧表达式初始化变量 `loc`。
- **L107 EN**: Continues logic associated with callable symbol `createGlobalConstant`.
  **L107 CN**: 继续与可调用符号 `createGlobalConstant` 相关的逻辑。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `loc, varTy, globalName,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`loc, varTy, globalName,`。

### Lines 109-126

````cpp
          [&](fir::FirOpBuilder &builder) {
            mlir::Operation *cln = definingOp->clone();
            builder.insert(cln);
            mlir::Value val =
                builder.createConvert(loc, varTy, cln->getResult(0));
            fir::HasValueOp::create(builder, loc, val);
          },
          builder.createInternalLinkage());
      mlir::Value addr = fir::AddrOfOp::create(
          builder, loc, global.resultType(), global.getSymbol());
      newOperands.push_back(addr);
      needUpdate = true;
    }

    if (needUpdate) {
      auto loc = callOp.getLoc();
      llvm::SmallVector<mlir::Type> newResultTypes;
      newResultTypes.append(callOp.getResultTypes().begin(),
````
- **L109 EN**: Starts a function, method, lambda, or structured scope: `[&](fir::FirOpBuilder &builder) {`.
  **L109 CN**: 开始一个函数、方法、lambda 或结构化作用域：`[&](fir::FirOpBuilder &builder) {`。
- **L110 EN**: Executes a call or declaration centered on `definingOp->clone`.
  **L110 CN**: 执行以 `definingOp->clone` 为核心的调用或声明。
- **L111 EN**: Executes a call or declaration centered on `builder.insert`.
  **L111 CN**: 执行以 `builder.insert` 为核心的调用或声明。
- **L112 EN**: Continues the surrounding expression or declaration: `mlir::Value val =`.
  **L112 CN**: 继续构造周围的表达式或声明：`mlir::Value val =`。
- **L113 EN**: Executes a call or declaration centered on `builder.createConvert`.
  **L113 CN**: 执行以 `builder.createConvert` 为核心的调用或声明。
- **L114 EN**: Executes a call or declaration centered on `fir::HasValueOp::create`.
  **L114 CN**: 执行以 `fir::HasValueOp::create` 为核心的调用或声明。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `},`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L116 EN**: Executes a call or declaration centered on `builder.createInternalLinkage`.
  **L116 CN**: 执行以 `builder.createInternalLinkage` 为核心的调用或声明。
- **L117 EN**: Continues logic associated with callable symbol `create`.
  **L117 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L118 EN**: Executes a call or declaration centered on `global.resultType`.
  **L118 CN**: 执行以 `global.resultType` 为核心的调用或声明。
- **L119 EN**: Executes a call or declaration centered on `newOperands.push_back`.
  **L119 CN**: 执行以 `newOperands.push_back` 为核心的调用或声明。
- **L120 EN**: Executes a standalone statement or declaration: `needUpdate = true;`.
  **L120 CN**: 执行一条独立语句或声明：`needUpdate = true;`。
- **L121 EN**: Closes the current lexical scope or compound statement.
  **L121 CN**: 结束当前词法作用域或复合语句块。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L123 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L123 CN**: 开始 `if` 控制流语句并计算其条件。
- **L124 EN**: Initializes variable `loc` from the right-hand expression.
  **L124 CN**: 使用右侧表达式初始化变量 `loc`。
- **L125 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Type> newResultTypes;`.
  **L125 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Type> newResultTypes;`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `newResultTypes.append(callOp.getResultTypes().begin(),`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`newResultTypes.append(callOp.getResultTypes().begin(),`。

### Lines 127-144

````cpp
                            callOp.getResultTypes().end());
      fir::CallOp newOp = fir::CallOp::create(builder, loc,
                                              callOp.getCallee().has_value()
                                                  ? callOp.getCallee().value()
                                                  : mlir::SymbolRefAttr{},
                                              newResultTypes, newOperands);
      // Copy all the attributes from the old to new op.
      newOp->setAttrs(callOp->getAttrs());
      rewriter.replaceOp(callOp, newOp);

      for (auto a : allocas) {
        if (a.first->hasOneUse()) {
          // If the alloca is only used for a store and the call operand, the
          // store is no longer required.
          rewriter.eraseOp(a.second);
          rewriter.eraseOp(a.first);
        }
      }
````
- **L127 EN**: Executes a call or declaration centered on `callOp.getResultTypes`.
  **L127 CN**: 执行以 `callOp.getResultTypes` 为核心的调用或声明。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::CallOp newOp = fir::CallOp::create(builder, loc,`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::CallOp newOp = fir::CallOp::create(builder, loc,`。
- **L129 EN**: Continues logic associated with callable symbol `getCallee`.
  **L129 CN**: 继续与可调用符号 `getCallee` 相关的逻辑。
- **L130 EN**: Continues logic associated with callable symbol `getCallee`.
  **L130 CN**: 继续与可调用符号 `getCallee` 相关的逻辑。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: mlir::SymbolRefAttr{},`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`: mlir::SymbolRefAttr{},`。
- **L132 EN**: Executes a standalone statement or declaration: `newResultTypes, newOperands);`.
  **L132 CN**: 执行一条独立语句或声明：`newResultTypes, newOperands);`。
- **L133 EN**: Comment explains nearby logic, intent, or metadata: `Copy all the attributes from the old to new op.`.
  **L133 CN**: 注释说明附近代码的逻辑、意图或元数据：`Copy all the attributes from the old to new op.`。
- **L134 EN**: Executes a call or declaration centered on `newOp->setAttrs`.
  **L134 CN**: 执行以 `newOp->setAttrs` 为核心的调用或声明。
- **L135 EN**: Executes a call or declaration centered on `rewriter.replaceOp`.
  **L135 CN**: 执行以 `rewriter.replaceOp` 为核心的调用或声明。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L137 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L137 CN**: 开始 `for` 控制流语句并计算其条件。
- **L138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L139 EN**: Comment explains nearby logic, intent, or metadata: `If the alloca is only used for a store and the call operand, the`.
  **L139 CN**: 注释说明附近代码的逻辑、意图或元数据：`If the alloca is only used for a store and the call operand, the`。
- **L140 EN**: Comment explains nearby logic, intent, or metadata: `store is no longer required.`.
  **L140 CN**: 注释说明附近代码的逻辑、意图或元数据：`store is no longer required.`。
- **L141 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L141 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L142 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L142 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-162

````cpp
      LLVM_DEBUG(llvm::dbgs() << "global constant for " << callOp << " as "
                              << newOp << '\n');
      return mlir::success();
    }

    // Failure here just means "we couldn't do the conversion", which is
    // perfectly acceptable to the upper layers of this function.
    return mlir::failure();
  }
};

// this pass attempts to convert immediate scalar literals in function calls
// to global constants to allow transformations such as Dead Argument
// Elimination
class ConstantArgumentGlobalisationOpt
    : public fir::impl::ConstantArgumentGlobalisationOptBase<
          ConstantArgumentGlobalisationOpt> {
public:
````
- **L145 EN**: Continues logic associated with callable symbol `LLVM_DEBUG`.
  **L145 CN**: 继续与可调用符号 `LLVM_DEBUG` 相关的逻辑。
- **L146 EN**: Executes a standalone statement or declaration: `<< newOp << '\n');`.
  **L146 CN**: 执行一条独立语句或声明：`<< newOp << '\n');`。
- **L147 EN**: Returns from the current function with `mlir::success()`.
  **L147 CN**: 以 `mlir::success()` 从当前函数返回。
- **L148 EN**: Closes the current lexical scope or compound statement.
  **L148 CN**: 结束当前词法作用域或复合语句块。
- **L149 EN**: Blank line separating nearby declarations or logic blocks.
  **L149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L150 EN**: Comment explains nearby logic, intent, or metadata: `Failure here just means "we couldn't do the conversion", which is`.
  **L150 CN**: 注释说明附近代码的逻辑、意图或元数据：`Failure here just means "we couldn't do the conversion", which is`。
- **L151 EN**: Comment explains nearby logic, intent, or metadata: `perfectly acceptable to the upper layers of this function.`.
  **L151 CN**: 注释说明附近代码的逻辑、意图或元数据：`perfectly acceptable to the upper layers of this function.`。
- **L152 EN**: Returns from the current function with `mlir::failure()`.
  **L152 CN**: 以 `mlir::failure()` 从当前函数返回。
- **L153 EN**: Closes the current lexical scope or compound statement.
  **L153 CN**: 结束当前词法作用域或复合语句块。
- **L154 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L154 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L156 EN**: Comment explains nearby logic, intent, or metadata: `this pass attempts to convert immediate scalar literals in function calls`.
  **L156 CN**: 注释说明附近代码的逻辑、意图或元数据：`this pass attempts to convert immediate scalar literals in function calls`。
- **L157 EN**: Comment explains nearby logic, intent, or metadata: `to global constants to allow transformations such as Dead Argument`.
  **L157 CN**: 注释说明附近代码的逻辑、意图或元数据：`to global constants to allow transformations such as Dead Argument`。
- **L158 EN**: Comment explains nearby logic, intent, or metadata: `Elimination`.
  **L158 CN**: 注释说明附近代码的逻辑、意图或元数据：`Elimination`。
- **L159 EN**: Declares class `ConstantArgumentGlobalisationOpt`.
  **L159 CN**: 声明 class `ConstantArgumentGlobalisationOpt`。
- **L160 EN**: Continues the surrounding expression or declaration: `: public fir::impl::ConstantArgumentGlobalisationOptBase<`.
  **L160 CN**: 继续构造周围的表达式或声明：`: public fir::impl::ConstantArgumentGlobalisationOptBase<`。
- **L161 EN**: Continues the surrounding expression or declaration: `ConstantArgumentGlobalisationOpt> {`.
  **L161 CN**: 继续构造周围的表达式或声明：`ConstantArgumentGlobalisationOpt> {`。
- **L162 EN**: Sets the following members to `public` access.
  **L162 CN**: 将后续成员的访问级别设为 `public`。

### Lines 163-180

````cpp
  ConstantArgumentGlobalisationOpt() = default;

  void runOnOperation() override {
    mlir::ModuleOp mod = getOperation();
    mlir::DominanceInfo *di = &getAnalysis<mlir::DominanceInfo>();
    auto *context = &getContext();
    mlir::RewritePatternSet patterns(context);
    mlir::GreedyRewriteConfig config;
    config.setRegionSimplificationLevel(
        mlir::GreedySimplifyRegionLevel::Disabled);
    config.setStrictness(mlir::GreedyRewriteStrictness::ExistingOps);

    patterns.insert<CallOpRewriter>(context, *di);
    if (mlir::failed(
            mlir::applyPatternsGreedily(mod, std::move(patterns), config))) {
      mlir::emitError(mod.getLoc(),
                      "error in constant globalisation optimization\n");
      signalPassFailure();
````
- **L163 EN**: Executes a call or declaration centered on `ConstantArgumentGlobalisationOpt`.
  **L163 CN**: 执行以 `ConstantArgumentGlobalisationOpt` 为核心的调用或声明。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Starts a function, method, lambda, or structured scope: `void runOnOperation() override {`.
  **L165 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void runOnOperation() override {`。
- **L166 EN**: Initializes variable `mod` from the right-hand expression.
  **L166 CN**: 使用右侧表达式初始化变量 `mod`。
- **L167 EN**: Executes a call or declaration centered on `&getAnalysis<mlir::DominanceInfo>`.
  **L167 CN**: 执行以 `&getAnalysis<mlir::DominanceInfo>` 为核心的调用或声明。
- **L168 EN**: Executes a call or declaration centered on `&getContext`.
  **L168 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L169 EN**: Executes a call or declaration centered on `patterns`.
  **L169 CN**: 执行以 `patterns` 为核心的调用或声明。
- **L170 EN**: Executes a standalone statement or declaration: `mlir::GreedyRewriteConfig config;`.
  **L170 CN**: 执行一条独立语句或声明：`mlir::GreedyRewriteConfig config;`。
- **L171 EN**: Continues logic associated with callable symbol `setRegionSimplificationLevel`.
  **L171 CN**: 继续与可调用符号 `setRegionSimplificationLevel` 相关的逻辑。
- **L172 EN**: Executes a standalone statement or declaration: `mlir::GreedySimplifyRegionLevel::Disabled);`.
  **L172 CN**: 执行一条独立语句或声明：`mlir::GreedySimplifyRegionLevel::Disabled);`。
- **L173 EN**: Executes a call or declaration centered on `config.setStrictness`.
  **L173 CN**: 执行以 `config.setStrictness` 为核心的调用或声明。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Executes a call or declaration centered on `patterns.insert<CallOpRewriter>`.
  **L175 CN**: 执行以 `patterns.insert<CallOpRewriter>` 为核心的调用或声明。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Starts a function, method, lambda, or structured scope: `mlir::applyPatternsGreedily(mod, std::move(patterns), config))) {`.
  **L177 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::applyPatternsGreedily(mod, std::move(patterns), config))) {`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::emitError(mod.getLoc(),`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::emitError(mod.getLoc(),`。
- **L179 EN**: Executes a standalone statement or declaration: `"error in constant globalisation optimization\n");`.
  **L179 CN**: 执行一条独立语句或声明：`"error in constant globalisation optimization\n");`。
- **L180 EN**: Executes a call or declaration centered on `signalPassFailure`.
  **L180 CN**: 执行以 `signalPassFailure` 为核心的调用或声明。

### Lines 181-184

````cpp
    }
  }
};
} // namespace
````
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L183 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L184 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L184 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **Symbol modeling and lookup / 符号建模与查找**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**
- **Pattern-driven IR rewriting / 基于模式的 IR 重写**
- **Operation rewrite patterns / 操作重写模式**
- **IR builder orchestration / IR Builder 编排**
- **Diagnostic emission / 诊断信息发出**
- **Driver-level compilation flow / 驱动级编译流程**

## Dependencies / 依赖关系

- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Dialect/FIRDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROps.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIRType.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `mlir/Dialect/Func/IR/FuncOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/Diagnostics.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/Dominance.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Pass/Pass.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Transforms/GreedyPatternRewriteDriver.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `flang/Optimizer/Transforms/Passes.h.inc`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
