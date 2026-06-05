# ConvertComplexPow.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/ConvertComplexPow.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for Convert Complex Pow.
- **Purpose (CN)**: 实现 Convert Complex Pow 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===- ConvertComplexPow.cpp - Convert complex.pow to library calls -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Common/static-multimap-view.h"
#include "flang/Optimizer/Builder/FIRBuilder.h"
#include "flang/Optimizer/Dialect/FIRDialect.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "flang/Runtime/entry-names.h"
#include "mlir/Dialect/Arith/IR/Arith.h"
#include "mlir/Dialect/Complex/IR/Complex.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
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
- **L9 EN**: Includes "flang/Common/static-multimap-view.h" to access shared Flang utility infrastructure.
  **L9 CN**: 引入 "flang/Common/static-multimap-view.h" 以使用Flang 共享工具基础设施。
- **L10 EN**: Includes "flang/Optimizer/Builder/FIRBuilder.h" to access FIR builder helpers and runtime-construction utilities.
  **L10 CN**: 引入 "flang/Optimizer/Builder/FIRBuilder.h" 以使用FIR Builder 辅助工具与运行时构造能力。
- **L11 EN**: Includes "flang/Optimizer/Dialect/FIRDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/FIRDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L12 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L13 EN**: Includes "flang/Runtime/entry-names.h" to access Fortran runtime entry points and descriptor helpers.
  **L13 CN**: 引入 "flang/Runtime/entry-names.h" 以使用Fortran 运行时入口与描述符辅助能力。
- **L14 EN**: Includes "mlir/Dialect/Arith/IR/Arith.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L14 CN**: 引入 "mlir/Dialect/Arith/IR/Arith.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L15 EN**: Includes "mlir/Dialect/Complex/IR/Complex.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L15 CN**: 引入 "mlir/Dialect/Complex/IR/Complex.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L16 EN**: Includes "mlir/Dialect/Func/IR/FuncOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/Dialect/Func/IR/FuncOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 17-32

````cpp
#include "mlir/Pass/Pass.h"

namespace fir {
#define GEN_PASS_DEF_CONVERTCOMPLEXPOW
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

using namespace mlir;

namespace {
class ConvertComplexPowPass
    : public fir::impl::ConvertComplexPowBase<ConvertComplexPowPass> {
public:
  void getDependentDialects(DialectRegistry &registry) const override {
    registry.insert<fir::FIROpsDialect, complex::ComplexDialect,
                    arith::ArithDialect, func::FuncDialect>();
````
- **L17 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Opens namespace scope `fir`.
  **L19 CN**: 打开命名空间作用域 `fir`。
- **L20 EN**: Defines macro `GEN_PASS_DEF_CONVERTCOMPLEXPOW` for conditional compilation or local shorthand.
  **L20 CN**: 定义宏 `GEN_PASS_DEF_CONVERTCOMPLEXPOW`，用于条件编译或本地简写。
- **L21 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L21 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L22 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L22 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Brings namespace `mlir` into the local scope.
  **L24 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Opens namespace scope ``.
  **L26 CN**: 打开命名空间作用域 ``。
- **L27 EN**: Declares class `ConvertComplexPowPass`.
  **L27 CN**: 声明 class `ConvertComplexPowPass`。
- **L28 EN**: Continues the surrounding expression or declaration: `: public fir::impl::ConvertComplexPowBase<ConvertComplexPowPass> {`.
  **L28 CN**: 继续构造周围的表达式或声明：`: public fir::impl::ConvertComplexPowBase<ConvertComplexPowPass> {`。
- **L29 EN**: Sets the following members to `public` access.
  **L29 CN**: 将后续成员的访问级别设为 `public`。
- **L30 EN**: Starts a function, method, lambda, or structured scope: `void getDependentDialects(DialectRegistry &registry) const override {`.
  **L30 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void getDependentDialects(DialectRegistry &registry) const override {`。
- **L31 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `registry.insert<fir::FIROpsDialect, complex::ComplexDialect,`.
  **L31 CN**: 继续一个多行参数列表、初始化器或聚合项：`registry.insert<fir::FIROpsDialect, complex::ComplexDialect,`。
- **L32 EN**: Executes a call or declaration centered on `func::FuncDialect>`.
  **L32 CN**: 执行以 `func::FuncDialect>` 为核心的调用或声明。

### Lines 33-48

````cpp
  }
  void runOnOperation() override;
};
} // namespace

// Helper to declare or get a math library function.
static func::FuncOp getOrDeclare(fir::FirOpBuilder &builder, Location loc,
                                 StringRef name, FunctionType type) {
  if (auto func = builder.getNamedFunction(name))
    return func;
  auto func = builder.createFunction(loc, name, type);
  func->setAttr(fir::getSymbolAttrName(), builder.getStringAttr(name));
  func->setAttr(fir::FIROpsDialect::getFirRuntimeAttrName(),
                builder.getUnitAttr());
  return func;
}
````
- **L33 EN**: Closes the current lexical scope or compound statement.
  **L33 CN**: 结束当前词法作用域或复合语句块。
- **L34 EN**: Executes a call or declaration centered on `runOnOperation`.
  **L34 CN**: 执行以 `runOnOperation` 为核心的调用或声明。
- **L35 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L35 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L36 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L36 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, intent, or metadata: `Helper to declare or get a math library function.`.
  **L38 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper to declare or get a math library function.`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static func::FuncOp getOrDeclare(fir::FirOpBuilder &builder, Location loc,`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`static func::FuncOp getOrDeclare(fir::FirOpBuilder &builder, Location loc,`。
- **L40 EN**: Continues the surrounding expression or declaration: `StringRef name, FunctionType type) {`.
  **L40 CN**: 继续构造周围的表达式或声明：`StringRef name, FunctionType type) {`。
- **L41 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L41 CN**: 开始 `if` 控制流语句并计算其条件。
- **L42 EN**: Returns from the current function with `func`.
  **L42 CN**: 以 `func` 从当前函数返回。
- **L43 EN**: Initializes variable `func` from the right-hand expression.
  **L43 CN**: 使用右侧表达式初始化变量 `func`。
- **L44 EN**: Executes a call or declaration centered on `func->setAttr`.
  **L44 CN**: 执行以 `func->setAttr` 为核心的调用或声明。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `func->setAttr(fir::FIROpsDialect::getFirRuntimeAttrName(),`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`func->setAttr(fir::FIROpsDialect::getFirRuntimeAttrName(),`。
- **L46 EN**: Executes a call or declaration centered on `builder.getUnitAttr`.
  **L46 CN**: 执行以 `builder.getUnitAttr` 为核心的调用或声明。
- **L47 EN**: Returns from the current function with `func`.
  **L47 CN**: 以 `func` 从当前函数返回。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-64

````cpp

void ConvertComplexPowPass::runOnOperation() {
  ModuleOp mod = getOperation();
  fir::FirOpBuilder builder(mod, fir::getKindMapping(mod));

  mod.walk([&](Operation *op) {
    if (auto powIop = dyn_cast<complex::PowiOp>(op)) {
      builder.setInsertionPoint(powIop);
      Location loc = powIop.getLoc();
      auto complexTy = cast<ComplexType>(powIop.getType());
      auto elemTy = complexTy.getElementType();
      Value base = powIop.getLhs();
      Value intExp = powIop.getRhs();
      func::FuncOp callee;
      unsigned realBits = cast<FloatType>(elemTy).getWidth();
      unsigned intBits = cast<IntegerType>(intExp.getType()).getWidth();
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L50 EN**: Starts a function, method, lambda, or structured scope: `void ConvertComplexPowPass::runOnOperation() {`.
  **L50 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ConvertComplexPowPass::runOnOperation() {`。
- **L51 EN**: Initializes variable `mod` from the right-hand expression.
  **L51 CN**: 使用右侧表达式初始化变量 `mod`。
- **L52 EN**: Executes a call or declaration centered on `builder`.
  **L52 CN**: 执行以 `builder` 为核心的调用或声明。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L54 EN**: Starts a function, method, lambda, or structured scope: `mod.walk([&](Operation *op) {`.
  **L54 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mod.walk([&](Operation *op) {`。
- **L55 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L55 CN**: 开始 `if` 控制流语句并计算其条件。
- **L56 EN**: Executes a call or declaration centered on `builder.setInsertionPoint`.
  **L56 CN**: 执行以 `builder.setInsertionPoint` 为核心的调用或声明。
- **L57 EN**: Initializes variable `loc` from the right-hand expression.
  **L57 CN**: 使用右侧表达式初始化变量 `loc`。
- **L58 EN**: Initializes variable `complexTy` from the right-hand expression.
  **L58 CN**: 使用右侧表达式初始化变量 `complexTy`。
- **L59 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L60 EN**: Initializes variable `base` from the right-hand expression.
  **L60 CN**: 使用右侧表达式初始化变量 `base`。
- **L61 EN**: Initializes variable `intExp` from the right-hand expression.
  **L61 CN**: 使用右侧表达式初始化变量 `intExp`。
- **L62 EN**: Executes a standalone statement or declaration: `func::FuncOp callee;`.
  **L62 CN**: 执行一条独立语句或声明：`func::FuncOp callee;`。
- **L63 EN**: Initializes variable `realBits` from the right-hand expression.
  **L63 CN**: 使用右侧表达式初始化变量 `realBits`。
- **L64 EN**: Initializes variable `intBits` from the right-hand expression.
  **L64 CN**: 使用右侧表达式初始化变量 `intBits`。

### Lines 65-80

````cpp
      auto funcTy = builder.getFunctionType(
          {complexTy, builder.getIntegerType(intBits)}, {complexTy});
      if (realBits == 32 && intBits == 32)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(cpowi), funcTy);
      else if (realBits == 32 && intBits == 64)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(cpowk), funcTy);
      else if (realBits == 64 && intBits == 32)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(zpowi), funcTy);
      else if (realBits == 64 && intBits == 64)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(zpowk), funcTy);
      else if (realBits == 128 && intBits == 32)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(cqpowi), funcTy);
      else if (realBits == 128 && intBits == 64)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(cqpowk), funcTy);
      else
        return;
````
- **L65 EN**: Continues logic associated with callable symbol `getFunctionType`.
  **L65 CN**: 继续与可调用符号 `getFunctionType` 相关的逻辑。
- **L66 EN**: Executes a call or declaration centered on `builder.getIntegerType`.
  **L66 CN**: 执行以 `builder.getIntegerType` 为核心的调用或声明。
- **L67 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L67 CN**: 开始 `if` 控制流语句并计算其条件。
- **L68 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L68 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L69 EN**: Starts the alternative branch of the preceding conditional.
  **L69 CN**: 开始前一个条件语句的备选分支。
- **L70 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L70 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L71 EN**: Starts the alternative branch of the preceding conditional.
  **L71 CN**: 开始前一个条件语句的备选分支。
- **L72 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L72 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L73 EN**: Starts the alternative branch of the preceding conditional.
  **L73 CN**: 开始前一个条件语句的备选分支。
- **L74 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L74 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L75 EN**: Starts the alternative branch of the preceding conditional.
  **L75 CN**: 开始前一个条件语句的备选分支。
- **L76 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L76 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L77 EN**: Starts the alternative branch of the preceding conditional.
  **L77 CN**: 开始前一个条件语句的备选分支。
- **L78 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L78 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L79 EN**: Transitions from the previous branch into the alternative path.
  **L79 CN**: 从前一个分支过渡到备选路径。
- **L80 EN**: Returns from the current function with `void`.
  **L80 CN**: 以 `void` 从当前函数返回。

### Lines 81-96

````cpp
      auto call = fir::CallOp::create(builder, loc, callee, {base, intExp});
      if (auto fmf = powIop.getFastmathAttr())
        call.setFastmathAttr(fmf);
      powIop.replaceAllUsesWith(call.getResult(0));
      powIop.erase();
    } else if (auto powOp = dyn_cast<complex::PowOp>(op)) {
      builder.setInsertionPoint(powOp);
      Location loc = powOp.getLoc();
      auto complexTy = cast<ComplexType>(powOp.getType());
      auto elemTy = complexTy.getElementType();
      unsigned realBits = cast<FloatType>(elemTy).getWidth();
      func::FuncOp callee;
      auto funcTy =
          builder.getFunctionType({complexTy, complexTy}, {complexTy});
      if (realBits == 32)
        callee = getOrDeclare(builder, loc, "cpowf", funcTy);
````
- **L81 EN**: Initializes variable `call` from the right-hand expression.
  **L81 CN**: 使用右侧表达式初始化变量 `call`。
- **L82 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L82 CN**: 开始 `if` 控制流语句并计算其条件。
- **L83 EN**: Executes a call or declaration centered on `call.setFastmathAttr`.
  **L83 CN**: 执行以 `call.setFastmathAttr` 为核心的调用或声明。
- **L84 EN**: Executes a call or declaration centered on `powIop.replaceAllUsesWith`.
  **L84 CN**: 执行以 `powIop.replaceAllUsesWith` 为核心的调用或声明。
- **L85 EN**: Executes a call or declaration centered on `powIop.erase`.
  **L85 CN**: 执行以 `powIop.erase` 为核心的调用或声明。
- **L86 EN**: Transitions from the previous branch into an `else if` condition.
  **L86 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L87 EN**: Executes a call or declaration centered on `builder.setInsertionPoint`.
  **L87 CN**: 执行以 `builder.setInsertionPoint` 为核心的调用或声明。
- **L88 EN**: Initializes variable `loc` from the right-hand expression.
  **L88 CN**: 使用右侧表达式初始化变量 `loc`。
- **L89 EN**: Initializes variable `complexTy` from the right-hand expression.
  **L89 CN**: 使用右侧表达式初始化变量 `complexTy`。
- **L90 EN**: Initializes variable `elemTy` from the right-hand expression.
  **L90 CN**: 使用右侧表达式初始化变量 `elemTy`。
- **L91 EN**: Initializes variable `realBits` from the right-hand expression.
  **L91 CN**: 使用右侧表达式初始化变量 `realBits`。
- **L92 EN**: Executes a standalone statement or declaration: `func::FuncOp callee;`.
  **L92 CN**: 执行一条独立语句或声明：`func::FuncOp callee;`。
- **L93 EN**: Continues the surrounding expression or declaration: `auto funcTy =`.
  **L93 CN**: 继续构造周围的表达式或声明：`auto funcTy =`。
- **L94 EN**: Executes a call or declaration centered on `builder.getFunctionType`.
  **L94 CN**: 执行以 `builder.getFunctionType` 为核心的调用或声明。
- **L95 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L95 CN**: 开始 `if` 控制流语句并计算其条件。
- **L96 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L96 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。

### Lines 97-111

````cpp
      else if (realBits == 64)
        callee = getOrDeclare(builder, loc, "cpow", funcTy);
      else if (realBits == 128)
        callee = getOrDeclare(builder, loc, RTNAME_STRING(CPowF128), funcTy);
      else
        return;
      auto call = fir::CallOp::create(builder, loc, callee,
                                      {powOp.getLhs(), powOp.getRhs()});
      if (auto fmf = powOp.getFastmathAttr())
        call.setFastmathAttr(fmf);
      powOp.replaceAllUsesWith(call.getResult(0));
      powOp.erase();
    }
  });
}
````
- **L97 EN**: Starts the alternative branch of the preceding conditional.
  **L97 CN**: 开始前一个条件语句的备选分支。
- **L98 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L98 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L99 EN**: Starts the alternative branch of the preceding conditional.
  **L99 CN**: 开始前一个条件语句的备选分支。
- **L100 EN**: Executes a call or declaration centered on `getOrDeclare`.
  **L100 CN**: 执行以 `getOrDeclare` 为核心的调用或声明。
- **L101 EN**: Transitions from the previous branch into the alternative path.
  **L101 CN**: 从前一个分支过渡到备选路径。
- **L102 EN**: Returns from the current function with `void`.
  **L102 CN**: 以 `void` 从当前函数返回。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto call = fir::CallOp::create(builder, loc, callee,`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto call = fir::CallOp::create(builder, loc, callee,`。
- **L104 EN**: Executes a call or declaration centered on `{powOp.getLhs`.
  **L104 CN**: 执行以 `{powOp.getLhs` 为核心的调用或声明。
- **L105 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L105 CN**: 开始 `if` 控制流语句并计算其条件。
- **L106 EN**: Executes a call or declaration centered on `call.setFastmathAttr`.
  **L106 CN**: 执行以 `call.setFastmathAttr` 为核心的调用或声明。
- **L107 EN**: Executes a call or declaration centered on `powOp.replaceAllUsesWith`.
  **L107 CN**: 执行以 `powOp.replaceAllUsesWith` 为核心的调用或声明。
- **L108 EN**: Executes a call or declaration centered on `powOp.erase`.
  **L108 CN**: 执行以 `powOp.erase` 为核心的调用或声明。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Executes a standalone statement or declaration: `});`.
  **L110 CN**: 执行一条独立语句或声明：`});`。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **Symbol modeling and lookup / 符号建模与查找**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **IR builder orchestration / IR Builder 编排**
- **Runtime call integration / 运行时调用集成**
- **Conversion and lowering flow / 转换与 lowering 流程**

## Dependencies / 依赖关系

- `flang/Common/static-multimap-view.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `flang/Optimizer/Builder/FIRBuilder.h`: Provides FIR builder helpers and runtime-construction utilities. / 提供FIR Builder 辅助工具与运行时构造能力。
- `flang/Optimizer/Dialect/FIRDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Runtime/entry-names.h`: Provides Fortran runtime entry points and descriptor helpers. / 提供Fortran 运行时入口与描述符辅助能力。
- `mlir/Dialect/Arith/IR/Arith.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/Complex/IR/Complex.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/Func/IR/FuncOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Pass/Pass.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `flang/Optimizer/Transforms/Passes.h.inc`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
