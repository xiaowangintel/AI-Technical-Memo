# ExternalNameConversion.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Transforms/ExternalNameConversion.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements optimizer or code-generation passes for External Name Conversion.
- **Purpose (CN)**: 实现 External Name Conversion 相关的优化或代码生成 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

````cpp
//===- ExternalNameConversion.cpp -- convert name with external convention ===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Dialect/FIRDialect.h"
#include "flang/Optimizer/Dialect/FIROps.h"
#include "flang/Optimizer/Dialect/FIROpsSupport.h"
#include "flang/Optimizer/Support/InternalNames.h"
#include "flang/Optimizer/Transforms/Passes.h"
#include "flang/Support/Fortran.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/IR/Attributes.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Pass/Pass.h"
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
- **L9 EN**: Includes "flang/Optimizer/Dialect/FIRDialect.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L9 CN**: 引入 "flang/Optimizer/Dialect/FIRDialect.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L10 EN**: Includes "flang/Optimizer/Dialect/FIROps.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L10 CN**: 引入 "flang/Optimizer/Dialect/FIROps.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L11 EN**: Includes "flang/Optimizer/Dialect/FIROpsSupport.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L11 CN**: 引入 "flang/Optimizer/Dialect/FIROpsSupport.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L12 EN**: Includes "flang/Optimizer/Support/InternalNames.h" to access optimizer-side support routines and utilities.
  **L12 CN**: 引入 "flang/Optimizer/Support/InternalNames.h" 以使用优化器侧支持例程与工具。
- **L13 EN**: Includes "flang/Optimizer/Transforms/Passes.h" to access local declarations paired with this implementation.
  **L13 CN**: 引入 "flang/Optimizer/Transforms/Passes.h" 以使用与该实现配套的本地声明。
- **L14 EN**: Includes "flang/Support/Fortran.h" to access shared Flang utility infrastructure.
  **L14 CN**: 引入 "flang/Support/Fortran.h" 以使用Flang 共享工具基础设施。
- **L15 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L15 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L16 EN**: Includes "mlir/IR/Attributes.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/IR/Attributes.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L17 EN**: Includes "mlir/IR/SymbolTable.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/IR/SymbolTable.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "mlir/Pass/Pass.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/Pass/Pass.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 19-36

````cpp

namespace fir {
#define GEN_PASS_DEF_EXTERNALNAMECONVERSION
#include "flang/Optimizer/Transforms/Passes.h.inc"
} // namespace fir

using namespace mlir;

//===----------------------------------------------------------------------===//
// Helper functions
//===----------------------------------------------------------------------===//

/// Mangle the name with gfortran convention.
std::string
mangleExternalName(const std::pair<fir::NameUniquer::NameKind,
                                   fir::NameUniquer::DeconstructedName>
                       result,
                   bool appendUnderscore) {
````
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `fir`.
  **L20 CN**: 打开命名空间作用域 `fir`。
- **L21 EN**: Defines macro `GEN_PASS_DEF_EXTERNALNAMECONVERSION` for conditional compilation or local shorthand.
  **L21 CN**: 定义宏 `GEN_PASS_DEF_EXTERNALNAMECONVERSION`，用于条件编译或本地简写。
- **L22 EN**: Includes "flang/Optimizer/Transforms/Passes.h.inc" to access supporting declarations used by this translation unit.
  **L22 CN**: 引入 "flang/Optimizer/Transforms/Passes.h.inc" 以使用当前编译单元使用的辅助声明。
- **L23 EN**: Closes a namespace scope with a trailing comment: `} // namespace fir`.
  **L23 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace fir`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L25 EN**: Brings namespace `mlir` into the local scope.
  **L25 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L27 EN**: Banner comment marking a file or section boundary.
  **L27 CN**: 横幅注释，用于标记文件或章节边界。
- **L28 EN**: Comment explains nearby logic, intent, or metadata: `Helper functions`.
  **L28 CN**: 注释说明附近代码的逻辑、意图或元数据：`Helper functions`。
- **L29 EN**: Banner comment marking a file or section boundary.
  **L29 CN**: 横幅注释，用于标记文件或章节边界。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Comment explains nearby logic, intent, or metadata: `Mangle the name with gfortran convention.`.
  **L31 CN**: 注释说明附近代码的逻辑、意图或元数据：`Mangle the name with gfortran convention.`。
- **L32 EN**: Continues the surrounding expression or declaration: `std::string`.
  **L32 CN**: 继续构造周围的表达式或声明：`std::string`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mangleExternalName(const std::pair<fir::NameUniquer::NameKind,`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`mangleExternalName(const std::pair<fir::NameUniquer::NameKind,`。
- **L34 EN**: Continues the surrounding expression or declaration: `fir::NameUniquer::DeconstructedName>`.
  **L34 CN**: 继续构造周围的表达式或声明：`fir::NameUniquer::DeconstructedName>`。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `result,`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`result,`。
- **L36 EN**: Continues the surrounding expression or declaration: `bool appendUnderscore) {`.
  **L36 CN**: 继续构造周围的表达式或声明：`bool appendUnderscore) {`。

### Lines 37-54

````cpp
  if (result.first == fir::NameUniquer::NameKind::COMMON &&
      result.second.name.empty())
    return Fortran::common::blankCommonObjectName;
  return Fortran::common::GetExternalAssemblyName(result.second.name,
                                                  appendUnderscore);
}

/// Process a symbol reference and return the updated symbol reference if
/// needed.
std::optional<mlir::SymbolRefAttr>
processSymbolRef(mlir::SymbolRefAttr symRef, mlir::Operation *nestedOp,
                 const llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr>
                     &remappings) {
  if (auto remap = remappings.find(symRef.getLeafReference());
      remap != remappings.end()) {
    mlir::SymbolRefAttr symAttr = mlir::FlatSymbolRefAttr(remap->second);
    if (mlir::isa<mlir::gpu::LaunchFuncOp>(nestedOp))
      symAttr = mlir::SymbolRefAttr::get(
````
- **L37 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L37 CN**: 开始 `if` 控制流语句并计算其条件。
- **L38 EN**: Continues logic associated with callable symbol `empty`.
  **L38 CN**: 继续与可调用符号 `empty` 相关的逻辑。
- **L39 EN**: Returns from the current function with `Fortran::common::blankCommonObjectName`.
  **L39 CN**: 以 `Fortran::common::blankCommonObjectName` 从当前函数返回。
- **L40 EN**: Returns from the current function with `Fortran::common::GetExternalAssemblyName(result.second.name,`.
  **L40 CN**: 以 `Fortran::common::GetExternalAssemblyName(result.second.name,` 从当前函数返回。
- **L41 EN**: Executes a standalone statement or declaration: `appendUnderscore);`.
  **L41 CN**: 执行一条独立语句或声明：`appendUnderscore);`。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Comment explains nearby logic, intent, or metadata: `Process a symbol reference and return the updated symbol reference if`.
  **L44 CN**: 注释说明附近代码的逻辑、意图或元数据：`Process a symbol reference and return the updated symbol reference if`。
- **L45 EN**: Comment explains nearby logic, intent, or metadata: `needed.`.
  **L45 CN**: 注释说明附近代码的逻辑、意图或元数据：`needed.`。
- **L46 EN**: Continues the surrounding expression or declaration: `std::optional<mlir::SymbolRefAttr>`.
  **L46 CN**: 继续构造周围的表达式或声明：`std::optional<mlir::SymbolRefAttr>`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `processSymbolRef(mlir::SymbolRefAttr symRef, mlir::Operation *nestedOp,`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`processSymbolRef(mlir::SymbolRefAttr symRef, mlir::Operation *nestedOp,`。
- **L48 EN**: Continues the surrounding expression or declaration: `const llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr>`.
  **L48 CN**: 继续构造周围的表达式或声明：`const llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr>`。
- **L49 EN**: Continues the surrounding expression or declaration: `&remappings) {`.
  **L49 CN**: 继续构造周围的表达式或声明：`&remappings) {`。
- **L50 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L50 CN**: 开始 `if` 控制流语句并计算其条件。
- **L51 EN**: Starts a function, method, lambda, or structured scope: `remap != remappings.end()) {`.
  **L51 CN**: 开始一个函数、方法、lambda 或结构化作用域：`remap != remappings.end()) {`。
- **L52 EN**: Initializes variable `symAttr` from the right-hand expression.
  **L52 CN**: 使用右侧表达式初始化变量 `symAttr`。
- **L53 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L53 CN**: 开始 `if` 控制流语句并计算其条件。
- **L54 EN**: Continues logic associated with callable symbol `get`.
  **L54 CN**: 继续与可调用符号 `get` 相关的逻辑。

### Lines 55-72

````cpp
          symRef.getRootReference(), {mlir::FlatSymbolRefAttr(remap->second)});
    return symAttr;
  }
  return std::nullopt;
}

namespace {

class ExternalNameConversionPass
    : public fir::impl::ExternalNameConversionBase<ExternalNameConversionPass> {
public:
  using ExternalNameConversionBase<
      ExternalNameConversionPass>::ExternalNameConversionBase;

  mlir::ModuleOp getModule() { return getOperation(); }
  void runOnOperation() override;
};
} // namespace
````
- **L55 EN**: Executes a call or declaration centered on `symRef.getRootReference`.
  **L55 CN**: 执行以 `symRef.getRootReference` 为核心的调用或声明。
- **L56 EN**: Returns from the current function with `symAttr`.
  **L56 CN**: 以 `symAttr` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Returns from the current function with `std::nullopt`.
  **L58 CN**: 以 `std::nullopt` 从当前函数返回。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Opens namespace scope ``.
  **L61 CN**: 打开命名空间作用域 ``。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Declares class `ExternalNameConversionPass`.
  **L63 CN**: 声明 class `ExternalNameConversionPass`。
- **L64 EN**: Continues the surrounding expression or declaration: `: public fir::impl::ExternalNameConversionBase<ExternalNameConversionPass> {`.
  **L64 CN**: 继续构造周围的表达式或声明：`: public fir::impl::ExternalNameConversionBase<ExternalNameConversionPass> {`。
- **L65 EN**: Sets the following members to `public` access.
  **L65 CN**: 将后续成员的访问级别设为 `public`。
- **L66 EN**: Continues the surrounding expression or declaration: `using ExternalNameConversionBase<`.
  **L66 CN**: 继续构造周围的表达式或声明：`using ExternalNameConversionBase<`。
- **L67 EN**: Executes a standalone statement or declaration: `ExternalNameConversionPass>::ExternalNameConversionBase;`.
  **L67 CN**: 执行一条独立语句或声明：`ExternalNameConversionPass>::ExternalNameConversionBase;`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Continues logic associated with callable symbol `getModule`.
  **L69 CN**: 继续与可调用符号 `getModule` 相关的逻辑。
- **L70 EN**: Executes a call or declaration centered on `runOnOperation`.
  **L70 CN**: 执行以 `runOnOperation` 为核心的调用或声明。
- **L71 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L71 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L72 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L72 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。

### Lines 73-90

````cpp

void ExternalNameConversionPass::runOnOperation() {
  auto op = getOperation();
  auto *context = &getContext();

  llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr> remappings;
  mlir::SymbolTable symbolTable(op);

  auto processFctOrGlobal = [&](mlir::Operation &funcOrGlobal) {
    auto symName = funcOrGlobal.getAttrOfType<mlir::StringAttr>(
        mlir::SymbolTable::getSymbolAttrName());
    auto deconstructedName = fir::NameUniquer::deconstruct(symName);
    if (fir::NameUniquer::isExternalFacingUniquedName(deconstructedName)) {
      // Check if this is a private function that would conflict with a common
      // block and get its mangled name.
      if (auto funcOp = llvm::dyn_cast<mlir::func::FuncOp>(funcOrGlobal)) {
        if (funcOp.isPrivate()) {
          std::string mangledName =
````
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Starts a function, method, lambda, or structured scope: `void ExternalNameConversionPass::runOnOperation() {`.
  **L74 CN**: 开始一个函数、方法、lambda 或结构化作用域：`void ExternalNameConversionPass::runOnOperation() {`。
- **L75 EN**: Initializes variable `op` from the right-hand expression.
  **L75 CN**: 使用右侧表达式初始化变量 `op`。
- **L76 EN**: Executes a call or declaration centered on `&getContext`.
  **L76 CN**: 执行以 `&getContext` 为核心的调用或声明。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L78 EN**: Executes a standalone statement or declaration: `llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr> remappings;`.
  **L78 CN**: 执行一条独立语句或声明：`llvm::DenseMap<mlir::StringAttr, mlir::FlatSymbolRefAttr> remappings;`。
- **L79 EN**: Executes a call or declaration centered on `symbolTable`.
  **L79 CN**: 执行以 `symbolTable` 为核心的调用或声明。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L81 EN**: Starts a function, method, lambda, or structured scope: `auto processFctOrGlobal = [&](mlir::Operation &funcOrGlobal) {`.
  **L81 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto processFctOrGlobal = [&](mlir::Operation &funcOrGlobal) {`。
- **L82 EN**: Continues logic associated with callable symbol `StringAttr>`.
  **L82 CN**: 继续与可调用符号 `StringAttr>` 相关的逻辑。
- **L83 EN**: Executes a call or declaration centered on `mlir::SymbolTable::getSymbolAttrName`.
  **L83 CN**: 执行以 `mlir::SymbolTable::getSymbolAttrName` 为核心的调用或声明。
- **L84 EN**: Initializes variable `deconstructedName` from the right-hand expression.
  **L84 CN**: 使用右侧表达式初始化变量 `deconstructedName`。
- **L85 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L85 CN**: 开始 `if` 控制流语句并计算其条件。
- **L86 EN**: Comment explains nearby logic, intent, or metadata: `Check if this is a private function that would conflict with a common`.
  **L86 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if this is a private function that would conflict with a common`。
- **L87 EN**: Comment explains nearby logic, intent, or metadata: `block and get its mangled name.`.
  **L87 CN**: 注释说明附近代码的逻辑、意图或元数据：`block and get its mangled name.`。
- **L88 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L88 CN**: 开始 `if` 控制流语句并计算其条件。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Continues the surrounding expression or declaration: `std::string mangledName =`.
  **L90 CN**: 继续构造周围的表达式或声明：`std::string mangledName =`。

### Lines 91-108

````cpp
              mangleExternalName(deconstructedName, appendUnderscoreOpt);
          auto mod = funcOp->getParentOfType<mlir::ModuleOp>();
          bool hasConflictingCommonBlock = false;

          // Check if any existing global has the same mangled name.
          if (symbolTable.lookup<fir::GlobalOp>(mangledName))
            hasConflictingCommonBlock = true;

          // Skip externalization if the function has a conflicting common block
          // and is not directly called (i.e. procedure pointers or type
          // specifications)
          if (hasConflictingCommonBlock) {
            bool isDirectlyCalled = false;
            std::optional<SymbolTable::UseRange> uses =
                funcOp.getSymbolUses(mod);
            if (uses.has_value()) {
              for (auto use : *uses) {
                mlir::Operation *user = use.getUser();
````
- **L91 EN**: Executes a call or declaration centered on `mangleExternalName`.
  **L91 CN**: 执行以 `mangleExternalName` 为核心的调用或声明。
- **L92 EN**: Initializes variable `mod` from the right-hand expression.
  **L92 CN**: 使用右侧表达式初始化变量 `mod`。
- **L93 EN**: Initializes variable `hasConflictingCommonBlock` from the right-hand expression.
  **L93 CN**: 使用右侧表达式初始化变量 `hasConflictingCommonBlock`。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L95 EN**: Comment explains nearby logic, intent, or metadata: `Check if any existing global has the same mangled name.`.
  **L95 CN**: 注释说明附近代码的逻辑、意图或元数据：`Check if any existing global has the same mangled name.`。
- **L96 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L96 CN**: 开始 `if` 控制流语句并计算其条件。
- **L97 EN**: Executes a standalone statement or declaration: `hasConflictingCommonBlock = true;`.
  **L97 CN**: 执行一条独立语句或声明：`hasConflictingCommonBlock = true;`。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Comment explains nearby logic, intent, or metadata: `Skip externalization if the function has a conflicting common block`.
  **L99 CN**: 注释说明附近代码的逻辑、意图或元数据：`Skip externalization if the function has a conflicting common block`。
- **L100 EN**: Comment explains nearby logic, intent, or metadata: `and is not directly called (i.e. procedure pointers or type`.
  **L100 CN**: 注释说明附近代码的逻辑、意图或元数据：`and is not directly called (i.e. procedure pointers or type`。
- **L101 EN**: Comment explains nearby logic, intent, or metadata: `specifications)`.
  **L101 CN**: 注释说明附近代码的逻辑、意图或元数据：`specifications)`。
- **L102 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L102 CN**: 开始 `if` 控制流语句并计算其条件。
- **L103 EN**: Initializes variable `isDirectlyCalled` from the right-hand expression.
  **L103 CN**: 使用右侧表达式初始化变量 `isDirectlyCalled`。
- **L104 EN**: Continues the surrounding expression or declaration: `std::optional<SymbolTable::UseRange> uses =`.
  **L104 CN**: 继续构造周围的表达式或声明：`std::optional<SymbolTable::UseRange> uses =`。
- **L105 EN**: Executes a call or declaration centered on `funcOp.getSymbolUses`.
  **L105 CN**: 执行以 `funcOp.getSymbolUses` 为核心的调用或声明。
- **L106 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L106 CN**: 开始 `if` 控制流语句并计算其条件。
- **L107 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L107 CN**: 开始 `for` 控制流语句并计算其条件。
- **L108 EN**: Executes a call or declaration centered on `use.getUser`.
  **L108 CN**: 执行以 `use.getUser` 为核心的调用或声明。

### Lines 109-126

````cpp
                if (mlir::isa<fir::CallOp>(user) ||
                    mlir::isa<mlir::func::CallOp>(user)) {
                  isDirectlyCalled = true;
                  break;
                }
              }
            }
            if (!isDirectlyCalled)
              return;
          }
        }
      }

      auto newName = mangleExternalName(deconstructedName, appendUnderscoreOpt);
      auto newAttr = mlir::StringAttr::get(context, newName);
      mlir::SymbolTable::setSymbolName(&funcOrGlobal, newAttr);
      auto newSymRef = mlir::FlatSymbolRefAttr::get(newAttr);
      remappings.try_emplace(symName, newSymRef);
````
- **L109 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L109 CN**: 开始 `if` 控制流语句并计算其条件。
- **L110 EN**: Starts a function, method, lambda, or structured scope: `mlir::isa<mlir::func::CallOp>(user)) {`.
  **L110 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::isa<mlir::func::CallOp>(user)) {`。
- **L111 EN**: Executes a standalone statement or declaration: `isDirectlyCalled = true;`.
  **L111 CN**: 执行一条独立语句或声明：`isDirectlyCalled = true;`。
- **L112 EN**: Exits the nearest loop or switch statement.
  **L112 CN**: 退出最近的循环或 switch 语句。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Closes the current lexical scope or compound statement.
  **L114 CN**: 结束当前词法作用域或复合语句块。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L116 CN**: 开始 `if` 控制流语句并计算其条件。
- **L117 EN**: Returns from the current function with `void`.
  **L117 CN**: 以 `void` 从当前函数返回。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Initializes variable `newName` from the right-hand expression.
  **L122 CN**: 使用右侧表达式初始化变量 `newName`。
- **L123 EN**: Initializes variable `newAttr` from the right-hand expression.
  **L123 CN**: 使用右侧表达式初始化变量 `newAttr`。
- **L124 EN**: Executes a call or declaration centered on `mlir::SymbolTable::setSymbolName`.
  **L124 CN**: 执行以 `mlir::SymbolTable::setSymbolName` 为核心的调用或声明。
- **L125 EN**: Initializes variable `newSymRef` from the right-hand expression.
  **L125 CN**: 使用右侧表达式初始化变量 `newSymRef`。
- **L126 EN**: Executes a call or declaration centered on `remappings.try_emplace`.
  **L126 CN**: 执行以 `remappings.try_emplace` 为核心的调用或声明。

### Lines 127-144

````cpp
      if (llvm::isa<mlir::func::FuncOp>(funcOrGlobal))
        funcOrGlobal.setAttr(fir::getInternalFuncNameAttrName(), symName);
    }
  };

  auto renameFuncOrGlobalInModule = [&](mlir::Operation *module) {
    for (auto &op : module->getRegion(0).front()) {
      if (mlir::isa<mlir::func::FuncOp, fir::GlobalOp>(op)) {
        processFctOrGlobal(op);
      } else if (auto gpuMod = mlir::dyn_cast<mlir::gpu::GPUModuleOp>(op)) {
        for (auto &gpuOp : gpuMod.getBodyRegion().front())
          if (mlir::isa<mlir::func::FuncOp, fir::GlobalOp,
                        mlir::gpu::GPUFuncOp>(gpuOp))
            processFctOrGlobal(gpuOp);
      }
    }
  };

````
- **L127 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L127 CN**: 开始 `if` 控制流语句并计算其条件。
- **L128 EN**: Executes a call or declaration centered on `funcOrGlobal.setAttr`.
  **L128 CN**: 执行以 `funcOrGlobal.setAttr` 为核心的调用或声明。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L130 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L132 EN**: Starts a function, method, lambda, or structured scope: `auto renameFuncOrGlobalInModule = [&](mlir::Operation *module) {`.
  **L132 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto renameFuncOrGlobalInModule = [&](mlir::Operation *module) {`。
- **L133 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L133 CN**: 开始 `for` 控制流语句并计算其条件。
- **L134 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L134 CN**: 开始 `if` 控制流语句并计算其条件。
- **L135 EN**: Executes a call or declaration centered on `processFctOrGlobal`.
  **L135 CN**: 执行以 `processFctOrGlobal` 为核心的调用或声明。
- **L136 EN**: Transitions from the previous branch into an `else if` condition.
  **L136 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L137 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L137 CN**: 开始 `for` 控制流语句并计算其条件。
- **L138 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L138 CN**: 开始 `if` 控制流语句并计算其条件。
- **L139 EN**: Continues logic associated with callable symbol `GPUFuncOp>`.
  **L139 CN**: 继续与可调用符号 `GPUFuncOp>` 相关的逻辑。
- **L140 EN**: Executes a call or declaration centered on `processFctOrGlobal`.
  **L140 CN**: 执行以 `processFctOrGlobal` 为核心的调用或声明。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L143 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 145-162

````cpp
  // Update names of external Fortran functions and names of Common Block
  // globals.
  renameFuncOrGlobalInModule(op);

  if (remappings.empty())
    return;

  // Update all uses of the functions and globals that have been renamed.
  op.walk([&remappings](mlir::Operation *nestedOp) {
    llvm::SmallVector<std::pair<mlir::StringAttr, mlir::SymbolRefAttr>>
        symRefUpdates;
    llvm::SmallVector<std::pair<mlir::StringAttr, mlir::ArrayAttr>>
        arrayUpdates;
    for (const mlir::NamedAttribute &attr : nestedOp->getAttrDictionary())
      if (auto symRef = llvm::dyn_cast<mlir::SymbolRefAttr>(attr.getValue())) {
        if (auto newSymRef = processSymbolRef(symRef, nestedOp, remappings))
          symRefUpdates.emplace_back(
              std::pair<mlir::StringAttr, mlir::SymbolRefAttr>{attr.getName(),
````
- **L145 EN**: Comment explains nearby logic, intent, or metadata: `Update names of external Fortran functions and names of Common Block`.
  **L145 CN**: 注释说明附近代码的逻辑、意图或元数据：`Update names of external Fortran functions and names of Common Block`。
- **L146 EN**: Comment explains nearby logic, intent, or metadata: `globals.`.
  **L146 CN**: 注释说明附近代码的逻辑、意图或元数据：`globals.`。
- **L147 EN**: Executes a call or declaration centered on `renameFuncOrGlobalInModule`.
  **L147 CN**: 执行以 `renameFuncOrGlobalInModule` 为核心的调用或声明。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L149 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L149 CN**: 开始 `if` 控制流语句并计算其条件。
- **L150 EN**: Returns from the current function with `void`.
  **L150 CN**: 以 `void` 从当前函数返回。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Comment explains nearby logic, intent, or metadata: `Update all uses of the functions and globals that have been renamed.`.
  **L152 CN**: 注释说明附近代码的逻辑、意图或元数据：`Update all uses of the functions and globals that have been renamed.`。
- **L153 EN**: Starts a function, method, lambda, or structured scope: `op.walk([&remappings](mlir::Operation *nestedOp) {`.
  **L153 CN**: 开始一个函数、方法、lambda 或结构化作用域：`op.walk([&remappings](mlir::Operation *nestedOp) {`。
- **L154 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<std::pair<mlir::StringAttr, mlir::SymbolRefAttr>>`.
  **L154 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<std::pair<mlir::StringAttr, mlir::SymbolRefAttr>>`。
- **L155 EN**: Executes a standalone statement or declaration: `symRefUpdates;`.
  **L155 CN**: 执行一条独立语句或声明：`symRefUpdates;`。
- **L156 EN**: Continues the surrounding expression or declaration: `llvm::SmallVector<std::pair<mlir::StringAttr, mlir::ArrayAttr>>`.
  **L156 CN**: 继续构造周围的表达式或声明：`llvm::SmallVector<std::pair<mlir::StringAttr, mlir::ArrayAttr>>`。
- **L157 EN**: Executes a standalone statement or declaration: `arrayUpdates;`.
  **L157 CN**: 执行一条独立语句或声明：`arrayUpdates;`。
- **L158 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L158 CN**: 开始 `for` 控制流语句并计算其条件。
- **L159 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L159 CN**: 开始 `if` 控制流语句并计算其条件。
- **L160 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L160 CN**: 开始 `if` 控制流语句并计算其条件。
- **L161 EN**: Continues logic associated with callable symbol `emplace_back`.
  **L161 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `std::pair<mlir::StringAttr, mlir::SymbolRefAttr>{attr.getName(),`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`std::pair<mlir::StringAttr, mlir::SymbolRefAttr>{attr.getName(),`。

### Lines 163-180

````cpp
                                                               *newSymRef});
      } else if (auto arrayAttr =
                     llvm::dyn_cast<mlir::ArrayAttr>(attr.getValue())) {
        llvm::SmallVector<mlir::Attribute> symbolRefs;
        for (auto element : arrayAttr) {
          if (!element) {
            symbolRefs.push_back(element);
            continue;
          }
          auto symRef = llvm::dyn_cast<mlir::SymbolRefAttr>(element);
          std::optional<mlir::SymbolRefAttr> updatedSymRef;
          if (symRef)
            updatedSymRef = processSymbolRef(symRef, nestedOp, remappings);
          if (!symRef || !updatedSymRef)
            symbolRefs.push_back(element);
          else
            symbolRefs.push_back(*updatedSymRef);
        }
````
- **L163 EN**: Comment explains nearby logic, intent, or metadata: `newSymRef});`.
  **L163 CN**: 注释说明附近代码的逻辑、意图或元数据：`newSymRef});`。
- **L164 EN**: Transitions from the previous branch into an `else if` condition.
  **L164 CN**: 从前一个分支过渡到 `else if` 条件判断。
- **L165 EN**: Starts a function, method, lambda, or structured scope: `llvm::dyn_cast<mlir::ArrayAttr>(attr.getValue())) {`.
  **L165 CN**: 开始一个函数、方法、lambda 或结构化作用域：`llvm::dyn_cast<mlir::ArrayAttr>(attr.getValue())) {`。
- **L166 EN**: Executes a standalone statement or declaration: `llvm::SmallVector<mlir::Attribute> symbolRefs;`.
  **L166 CN**: 执行一条独立语句或声明：`llvm::SmallVector<mlir::Attribute> symbolRefs;`。
- **L167 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L167 CN**: 开始 `for` 控制流语句并计算其条件。
- **L168 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L168 CN**: 开始 `if` 控制流语句并计算其条件。
- **L169 EN**: Executes a call or declaration centered on `symbolRefs.push_back`.
  **L169 CN**: 执行以 `symbolRefs.push_back` 为核心的调用或声明。
- **L170 EN**: Skips to the next loop iteration.
  **L170 CN**: 跳到下一次循环迭代。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Initializes variable `symRef` from the right-hand expression.
  **L172 CN**: 使用右侧表达式初始化变量 `symRef`。
- **L173 EN**: Executes a standalone statement or declaration: `std::optional<mlir::SymbolRefAttr> updatedSymRef;`.
  **L173 CN**: 执行一条独立语句或声明：`std::optional<mlir::SymbolRefAttr> updatedSymRef;`。
- **L174 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L174 CN**: 开始 `if` 控制流语句并计算其条件。
- **L175 EN**: Executes a call or declaration centered on `processSymbolRef`.
  **L175 CN**: 执行以 `processSymbolRef` 为核心的调用或声明。
- **L176 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L176 CN**: 开始 `if` 控制流语句并计算其条件。
- **L177 EN**: Executes a call or declaration centered on `symbolRefs.push_back`.
  **L177 CN**: 执行以 `symbolRefs.push_back` 为核心的调用或声明。
- **L178 EN**: Transitions from the previous branch into the alternative path.
  **L178 CN**: 从前一个分支过渡到备选路径。
- **L179 EN**: Executes a call or declaration centered on `symbolRefs.push_back`.
  **L179 CN**: 执行以 `symbolRefs.push_back` 为核心的调用或声明。
- **L180 EN**: Closes the current lexical scope or compound statement.
  **L180 CN**: 结束当前词法作用域或复合语句块。

### Lines 181-190

````cpp
        arrayUpdates.push_back(std::make_pair(
            attr.getName(),
            mlir::ArrayAttr::get(nestedOp->getContext(), symbolRefs)));
      }
    for (auto update : symRefUpdates)
      nestedOp->setAttr(update.first, update.second);
    for (auto update : arrayUpdates)
      nestedOp->setAttr(update.first, update.second);
  });
}
````
- **L181 EN**: Continues logic associated with callable symbol `push_back`.
  **L181 CN**: 继续与可调用符号 `push_back` 相关的逻辑。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `attr.getName(),`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`attr.getName(),`。
- **L183 EN**: Executes a call or declaration centered on `mlir::ArrayAttr::get`.
  **L183 CN**: 执行以 `mlir::ArrayAttr::get` 为核心的调用或声明。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L185 CN**: 开始 `for` 控制流语句并计算其条件。
- **L186 EN**: Executes a call or declaration centered on `nestedOp->setAttr`.
  **L186 CN**: 执行以 `nestedOp->setAttr` 为核心的调用或声明。
- **L187 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L187 CN**: 开始 `for` 控制流语句并计算其条件。
- **L188 EN**: Executes a call or declaration centered on `nestedOp->setAttr`.
  **L188 CN**: 执行以 `nestedOp->setAttr` 为核心的调用或声明。
- **L189 EN**: Executes a standalone statement or declaration: `});`.
  **L189 CN**: 执行一条独立语句或声明：`});`。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **Symbol modeling and lookup / 符号建模与查找**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**

## Dependencies / 依赖关系

- `flang/Optimizer/Dialect/FIRDialect.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROps.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Dialect/FIROpsSupport.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Support/InternalNames.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `flang/Optimizer/Transforms/Passes.h`: Provides local declarations paired with this implementation. / 提供与该实现配套的本地声明。
- `flang/Support/Fortran.h`: Provides shared Flang utility infrastructure. / 提供Flang 共享工具基础设施。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/Attributes.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/SymbolTable.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Pass/Pass.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `flang/Optimizer/Transforms/Passes.h.inc`: Provides supporting declarations used by this translation unit. / 提供当前编译单元使用的辅助声明。
