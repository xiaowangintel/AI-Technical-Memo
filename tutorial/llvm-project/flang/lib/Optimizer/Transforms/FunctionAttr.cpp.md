# FunctionAttr.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/FunctionAttr.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: \file This is a generic pass for adding attributes to functions.
- **Purpose (CN)**: 实现 Function Attr 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===- FunctionAttr.cpp ---------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
/// \file
/// This is a generic pass for adding attributes to functions.
//===----------------------------------------------------------------------===//
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "flang/Optimizer/Support/InternalNames.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "mlir/Dialect/LLVMIR/LLVMAttrs.h"
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
- **L9 EN**: Banner comment marking a file or section boundary.
  **L9 CN**: 横幅注释，用于标记文件或章节边界。
- **L10 EN**: Comment explains nearby logic, intent, or metadata: `\file`.
  **L10 CN**: 注释说明附近代码的逻辑、意图或元数据：`\file`。
- **L11 EN**: Comment explains nearby logic, intent, or metadata: `This is a generic pass for adding attributes to functions.`.
  **L11 CN**: 注释说明附近代码的逻辑、意图或元数据：`This is a generic pass for adding attributes to functions.`。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L13 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L14 EN**: Includes "flang/Optimizer/Support/InternalNames.h" to access optimizer-side support routines and utilities.
  **L14 CN**: 引入 "flang/Optimizer/Support/InternalNames.h" 以使用优化器侧支持例程与工具。
- **L15 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L15 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L16 EN**: Includes "mlir/Dialect/LLVMIR/LLVMAttrs.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/Dialect/LLVMIR/LLVMAttrs.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 17-32

````cpp
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "llvm/ADT/Twine.h"
#include <string>

namespace fir {
#define GEN_PASS_DEF_FUNCTIONATTR
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

#define DEBUG_TYPE "func-attr"

namespace {

/// Names of LLVM dialect function properties on `func.func` must use the
/// `llvm.` prefix so convert-func-to-llvm can recognize them and lower them
/// into `llvm.func` properties (bare ODS names are ignored as legacy spellings)
````
- **L17 EN**: Includes "mlir/Dialect/LLVMIR/LLVMDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/Dialect/LLVMIR/LLVMDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "llvm/ADT/Twine.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L18 CN**: 引入 "llvm/ADT/Twine.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L19 EN**: Includes <string> to access supporting declarations used by this translation unit.
  **L19 CN**: 引入 <string> 以使用当前编译单元使用的辅助声明。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L21 EN**: Opens namespace scope `fir`.
  **L21 CN**: 打开命名空间作用域 `fir`。
- **L22 EN**: Defines macro `GEN_PASS_DEF_FUNCTIONATTR` for conditional compilation or local shorthand.
  **L22 CN**: 定义宏 `GEN_PASS_DEF_FUNCTIONATTR`，用于条件编译或本地简写。
- **L23 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L23 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L24 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L24 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Defines macro `DEBUG_TYPE` for conditional compilation or local shorthand.
  **L26 CN**: 定义宏 `DEBUG_TYPE`，用于条件编译或本地简写。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Opens namespace scope ``.
  **L28 CN**: 打开命名空间作用域 ``。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, intent, or metadata: `Names of LLVM dialect function properties on `func.func` must use the`.
  **L30 CN**: 注释说明附近代码的逻辑、意图或元数据：`Names of LLVM dialect function properties on `func.func` must use the`。
- **L31 EN**: Comment explains nearby logic, intent, or metadata: ``llvm.` prefix so convert-func-to-llvm can recognize them and lower them`.
  **L31 CN**: 注释说明附近代码的逻辑、意图或元数据：``llvm.` prefix so convert-func-to-llvm can recognize them and lower them`。
- **L32 EN**: Comment explains nearby logic, intent, or metadata: `into `llvm.func` properties (bare ODS names are ignored as legacy spellings)`.
  **L32 CN**: 注释说明附近代码的逻辑、意图或元数据：`into `llvm.func` properties (bare ODS names are ignored as legacy spellings)`。

### Lines 33-48

````cpp
static mlir::StringAttr getLlvmFuncPropertyAttrName(mlir::MLIRContext *ctx,
                                                    mlir::StringAttr baseName) {
  return mlir::StringAttr::get(ctx, llvm::Twine("llvm.") + baseName.getValue());
}

class FunctionAttrPass : public fir::impl::FunctionAttrBase<FunctionAttrPass> {
public:
  FunctionAttrPass(const fir::FunctionAttrOptions &options) : Base{options} {}
  FunctionAttrPass() = default;
  void runOnOperation() override;
};

} // namespace

void FunctionAttrPass::runOnOperation() {
  LLVM_DEBUG(llvm::dbgs() << "=== Begin " DEBUG_TYPE " ===\n");
````
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static mlir::StringAttr getLlvmFuncPropertyAttrName(mlir::MLIRContext *ctx,`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`static mlir::StringAttr getLlvmFuncPropertyAttrName(mlir::MLIRContext *ctx,`。
- **L34 EN**: Continues the surrounding expression or declaration: `mlir::StringAttr baseName) {`.
  **L34 CN**: 继续构造周围的表达式或声明：`mlir::StringAttr baseName) {`。
- **L35 EN**: Returns from the current function with `mlir::StringAttr::get(ctx, llvm::Twine("llvm.") + baseName.getValue())`.
  **L35 CN**: 以 `mlir::StringAttr::get(ctx, llvm::Twine("llvm.") + baseName.getValue())` 从当前函数返回。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Declares class `FunctionAttrPass`.
  **L38 CN**: 声明 class `FunctionAttrPass`。
- **L39 EN**: Sets the following members to `public` access.
  **L39 CN**: 将后续成员的访问级别设为 `public`。
- **L40 EN**: Continues logic associated with callable symbol `FunctionAttrPass`.
  **L40 CN**: 继续与可调用符号 `FunctionAttrPass` 相关的逻辑。
- **L41 EN**: Executes a call or declaration centered on `FunctionAttrPass`.
  **L41 CN**: 执行以 `FunctionAttrPass` 为核心的调用或声明。
- **L42 EN**: Executes a call or declaration centered on `runOnOperation`.
  **L42 CN**: 执行以 `runOnOperation` 为核心的调用或声明。
- **L43 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L43 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L45 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L45 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Starts a function, method, lambda, or structured scope: `void FunctionAttrPass::runOnOperation() {`.
  **L47 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void FunctionAttrPass::runOnOperation() {`。
- **L48 EN**: Sets or uses the LLVM debug logging category.
  **L48 CN**: 设置或使用 LLVM 调试日志类别。

### Lines 49-64

````cpp
  mlir::func::FuncOp func = getOperation();

  LLVM_DEBUG(llvm::dbgs() << "Func-name:" << func.getSymName() << "\n");

  llvm::StringRef name = func.getSymName();
  auto deconstructed = fir::NameUniquer::deconstruct(name);
  bool isFromModule = !deconstructed.second.modules.empty();

  if ((isFromModule || !func.isDeclaration()) &&
      !fir::hasBindcAttr(func.getOperation())) {
    llvm::StringRef nocapture = mlir::LLVM::LLVMDialect::getNoCaptureAttrName();
    llvm::StringRef noalias = mlir::LLVM::LLVMDialect::getNoAliasAttrName();
    mlir::UnitAttr unitAttr = mlir::UnitAttr::get(func.getContext());

    for (auto [index, argType] : llvm::enumerate(func.getArgumentTypes())) {
      bool isNoCapture = false;
````
- **L49 EN**: Initializes variable `func` from the right-hand expression.
  **L49 CN**: 使用右侧表达式初始化变量 `func`。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L51 EN**: Executes a call or declaration centered on `LLVM_DEBUG`.
  **L51 CN**: 执行以 `LLVM_DEBUG` 为核心的调用或声明。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Initializes variable `name` from the right-hand expression.
  **L53 CN**: 使用右侧表达式初始化变量 `name`。
- **L54 EN**: Initializes variable `deconstructed` from the right-hand expression.
  **L54 CN**: 使用右侧表达式初始化变量 `deconstructed`。
- **L55 EN**: Initializes variable `isFromModule` from the right-hand expression.
  **L55 CN**: 使用右侧表达式初始化变量 `isFromModule`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L57 CN**: 开始 `if` 控制流语句并计算其条件。
- **L58 EN**: Starts a function, method, lambda, or structured scope: `!fir::hasBindcAttr(func.getOperation())) {`.
  **L58 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!fir::hasBindcAttr(func.getOperation())) {`。
- **L59 EN**: Initializes variable `nocapture` from the right-hand expression.
  **L59 CN**: 使用右侧表达式初始化变量 `nocapture`。
- **L60 EN**: Initializes variable `noalias` from the right-hand expression.
  **L60 CN**: 使用右侧表达式初始化变量 `noalias`。
- **L61 EN**: Initializes variable `unitAttr` from the right-hand expression.
  **L61 CN**: 使用右侧表达式初始化变量 `unitAttr`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L63 CN**: 开始 `for` 控制流语句并计算其条件。
- **L64 EN**: Initializes variable `isNoCapture` from the right-hand expression.
  **L64 CN**: 使用右侧表达式初始化变量 `isNoCapture`。

### Lines 65-80

````cpp
      bool isNoAlias = false;
      if (mlir::isa<fir::ReferenceType>(argType) &&
          !func.getArgAttr(index, fir::getTargetAttrName()) &&
          !func.getArgAttr(index, fir::getAsynchronousAttrName()) &&
          !func.getArgAttr(index, fir::getVolatileAttrName())) {
        isNoCapture = true;
        isNoAlias = !fir::isPointerType(argType);
      } else if (mlir::isa<fir::BaseBoxType>(argType)) {
        // !fir.box arguments will be passed as descriptor pointers
        // at LLVM IR dialect level - they cannot be captured,
        // and cannot alias with anything within the function.
        isNoCapture = isNoAlias = true;
      }
      if (isNoCapture && setNoCapture)
        func.setArgAttr(index, nocapture, unitAttr);
      if (isNoAlias && setNoAlias)
````
- **L65 EN**: Initializes variable `isNoAlias` from the right-hand expression.
  **L65 CN**: 使用右侧表达式初始化变量 `isNoAlias`。
- **L66 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L66 CN**: 开始 `if` 控制流语句并计算其条件。
- **L67 EN**: Continues logic associated with callable symbol `getArgAttr`.
  **L67 CN**: 继续与可调用符号 `getArgAttr` 相关的逻辑。
- **L68 EN**: Continues logic associated with callable symbol `getArgAttr`.
  **L68 CN**: 继续与可调用符号 `getArgAttr` 相关的逻辑。
- **L69 EN**: Starts a function, method, lambda, or structured scope: `!func.getArgAttr(index, fir::getVolatileAttrName())) {`.
  **L69 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!func.getArgAttr(index, fir::getVolatileAttrName())) {`。
- **L70 EN**: Executes a standalone statement or declaration: `isNoCapture = true;`.
  **L70 CN**: 执行一条独立语句或声明：`isNoCapture = true;`。
- **L71 EN**: Executes a call or declaration centered on `!fir::isPointerType`.
  **L71 CN**: 执行以 `!fir::isPointerType` 为核心的调用或声明。
- **L72 EN**: Transitions from the previous branch into an `else if` condition.
  **L72 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L73 EN**: Comment explains nearby logic, intent, or metadata: `fir.box arguments will be passed as descriptor pointers`.
  **L73 CN**: 注释说明附近代码的逻辑、意图或元数据：`fir.box arguments will be passed as descriptor pointers`。
- **L74 EN**: Comment explains nearby logic, intent, or metadata: `at LLVM IR dialect level - they cannot be captured,`.
  **L74 CN**: 注释说明附近代码的逻辑、意图或元数据：`at LLVM IR dialect level - they cannot be captured,`。
- **L75 EN**: Comment explains nearby logic, intent, or metadata: `and cannot alias with anything within the function.`.
  **L75 CN**: 注释说明附近代码的逻辑、意图或元数据：`and cannot alias with anything within the function.`。
- **L76 EN**: Executes a standalone statement or declaration: `isNoCapture = isNoAlias = true;`.
  **L76 CN**: 执行一条独立语句或声明：`isNoCapture = isNoAlias = true;`。
- **L77 EN**: Closes the current lexical scope or compound statement.
  **L77 CN**: 结束当前词法作用域或复合语句块。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Executes a call or declaration centered on `func.setArgAttr`.
  **L79 CN**: 执行以 `func.setArgAttr` 为核心的调用或声明。
- **L80 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L80 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 81-96

````cpp
        func.setArgAttr(index, noalias, unitAttr);
    }
  }

  mlir::MLIRContext *context = &getContext();
  auto llvmFuncOpName =
      mlir::OperationName(mlir::LLVM::LLVMFuncOp::getOperationName(), context);

  if (framePointerKind != mlir::LLVM::framePointerKind::FramePointerKind::None)
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context,
            mlir::LLVM::LLVMFuncOp::getFramePointerAttrName(llvmFuncOpName)),
        mlir::LLVM::FramePointerKindAttr::get(context, framePointerKind));

  if (!instrumentFunctionEntry.empty())
````
- **L81 EN**: Executes a call or declaration centered on `func.setArgAttr`.
  **L81 CN**: 执行以 `func.setArgAttr` 为核心的调用或声明。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L85 EN**: Executes a call or declaration centered on `&getContext`.
  **L85 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L86 EN**: Continues the surrounding expression or declaration: `auto llvmFuncOpName =`.
  **L86 CN**: 继续构造周围的表达式或声明：`auto llvmFuncOpName =`。
- **L87 EN**: Executes a call or declaration centered on `mlir::OperationName`.
  **L87 CN**: 执行以 `mlir::OperationName` 为核心的调用或声明。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Continues logic associated with callable symbol `setAttr`.
  **L90 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L91 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L91 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `context,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`context,`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::LLVM::LLVMFuncOp::getFramePointerAttrName(llvmFuncOpName)),`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::LLVM::LLVMFuncOp::getFramePointerAttrName(llvmFuncOpName)),`。
- **L94 EN**: Executes a call or declaration centered on `mlir::LLVM::FramePointerKindAttr::get`.
  **L94 CN**: 执行以 `mlir::LLVM::FramePointerKindAttr::get` 为核心的调用或声明。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L96 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L96 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 97-112

````cpp
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context, mlir::LLVM::LLVMFuncOp::getInstrumentFunctionEntryAttrName(
                         llvmFuncOpName)),
        mlir::StringAttr::get(context, instrumentFunctionEntry));
  if (!instrumentFunctionExit.empty())
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context, mlir::LLVM::LLVMFuncOp::getInstrumentFunctionExitAttrName(
                         llvmFuncOpName)),
        mlir::StringAttr::get(context, instrumentFunctionExit));
  if (noSignedZerosFPMath)
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context, mlir::LLVM::LLVMFuncOp::getNoSignedZerosFpMathAttrName(
                         llvmFuncOpName)),
````
- **L97 EN**: Continues logic associated with callable symbol `setAttr`.
  **L97 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L98 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L98 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L99 EN**: Continues logic associated with callable symbol `getInstrumentFunctionEntryAttrName`.
  **L99 CN**: 继续与可调用符号 `getInstrumentFunctionEntryAttrName` 相关的逻辑。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。
- **L101 EN**: Executes a call or declaration centered on `mlir::StringAttr::get`.
  **L101 CN**: 执行以 `mlir::StringAttr::get` 为核心的调用或声明。
- **L102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L103 EN**: Continues logic associated with callable symbol `setAttr`.
  **L103 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L104 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L104 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L105 EN**: Continues logic associated with callable symbol `getInstrumentFunctionExitAttrName`.
  **L105 CN**: 继续与可调用符号 `getInstrumentFunctionExitAttrName` 相关的逻辑。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。
- **L107 EN**: Executes a call or declaration centered on `mlir::StringAttr::get`.
  **L107 CN**: 执行以 `mlir::StringAttr::get` 为核心的调用或声明。
- **L108 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L108 CN**: 开始 `if` 控制流语句并计算其条件。
- **L109 EN**: Continues logic associated with callable symbol `setAttr`.
  **L109 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L110 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L110 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L111 EN**: Continues logic associated with callable symbol `getNoSignedZerosFpMathAttrName`.
  **L111 CN**: 继续与可调用符号 `getNoSignedZerosFpMathAttrName` 相关的逻辑。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。

### Lines 113-128

````cpp
        mlir::BoolAttr::get(context, true));
  if (!reciprocals.empty())
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context, mlir::LLVM::LLVMFuncOp::getReciprocalEstimatesAttrName(
                         llvmFuncOpName)),
        mlir::StringAttr::get(context, reciprocals));
  if (!preferVectorWidth.empty())
    func->setAttr(
        getLlvmFuncPropertyAttrName(
            context, mlir::LLVM::LLVMFuncOp::getPreferVectorWidthAttrName(
                         llvmFuncOpName)),
        mlir::StringAttr::get(context, preferVectorWidth));
  if (UseSampleProfile)
    func->setAttr(
        getLlvmFuncPropertyAttrName(
````
- **L113 EN**: Executes a call or declaration centered on `mlir::BoolAttr::get`.
  **L113 CN**: 执行以 `mlir::BoolAttr::get` 为核心的调用或声明。
- **L114 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L114 CN**: 开始 `if` 控制流语句并计算其条件。
- **L115 EN**: Continues logic associated with callable symbol `setAttr`.
  **L115 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L116 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L116 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L117 EN**: Continues logic associated with callable symbol `getReciprocalEstimatesAttrName`.
  **L117 CN**: 继续与可调用符号 `getReciprocalEstimatesAttrName` 相关的逻辑。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。
- **L119 EN**: Executes a call or declaration centered on `mlir::StringAttr::get`.
  **L119 CN**: 执行以 `mlir::StringAttr::get` 为核心的调用或声明。
- **L120 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L120 CN**: 开始 `if` 控制流语句并计算其条件。
- **L121 EN**: Continues logic associated with callable symbol `setAttr`.
  **L121 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L122 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L122 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。
- **L123 EN**: Continues logic associated with callable symbol `getPreferVectorWidthAttrName`.
  **L123 CN**: 继续与可调用符号 `getPreferVectorWidthAttrName` 相关的逻辑。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。
- **L125 EN**: Executes a call or declaration centered on `mlir::StringAttr::get`.
  **L125 CN**: 执行以 `mlir::StringAttr::get` 为核心的调用或声明。
- **L126 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L126 CN**: 开始 `if` 控制流语句并计算其条件。
- **L127 EN**: Continues logic associated with callable symbol `setAttr`.
  **L127 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L128 EN**: Continues logic associated with callable symbol `getLlvmFuncPropertyAttrName`.
  **L128 CN**: 继续与可调用符号 `getLlvmFuncPropertyAttrName` 相关的逻辑。

### Lines 129-134

````cpp
            context, mlir::LLVM::LLVMFuncOp::getUseSampleProfileAttrName(
                         llvmFuncOpName)),
        mlir::BoolAttr::get(context, true));

  LLVM_DEBUG(llvm::dbgs() << "=== End " DEBUG_TYPE " ===\n");
}
````
- **L129 EN**: Continues logic associated with callable symbol `getUseSampleProfileAttrName`.
  **L129 CN**: 继续与可调用符号 `getUseSampleProfileAttrName` 相关的逻辑。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvmFuncOpName)),`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvmFuncOpName)),`。
- **L131 EN**: Executes a call or declaration centered on `mlir::BoolAttr::get`.
  **L131 CN**: 执行以 `mlir::BoolAttr::get` 为核心的调用或声明。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Sets or uses the LLVM debug logging category.
  **L133 CN**: 设置或使用 LLVM 调试日志类别。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**

## Dependencies / 依赖关系

- `flang/Optimizer/Dialect/FIROpsSupport.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Support/InternalNames.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `mlir/Dialect/LLVMIR/LLVMAttrs.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/LLVMIR/LLVMDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `llvm/ADT/Twine.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `string`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
- `flang/Optimizer/Transforms/Passes.h.inc`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
