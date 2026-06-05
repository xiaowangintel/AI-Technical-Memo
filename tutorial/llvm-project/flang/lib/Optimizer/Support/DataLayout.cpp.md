# DataLayout.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `flang/lib/Optimizer/Support/DataLayout.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements FIR/MLIR optimizer-side utilities and infrastructure for Data Layout.
- **Purpose (CN)**: 实现 Data Layout 相关的 FIR/MLIR 优化器辅助工具与基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===-- Optimizer/Support/DataLayout.cpp ----------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "flang/Optimizer/Support/DataLayout.h"
#include "flang/Optimizer/Dialect/Support/FIRContext.h"
#include "flang/Optimizer/Support/FatalError.h"
#include "mlir/Dialect/DLTI/DLTI.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/Interfaces/DataLayoutInterfaces.h"
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
- **L9 EN**: Includes "flang/Optimizer/Support/DataLayout.h" to access optimizer-side support routines and utilities.
  **L9 CN**: 引入 "flang/Optimizer/Support/DataLayout.h" 以使用优化器侧支持例程与工具。
- **L10 EN**: Includes "flang/Optimizer/Dialect/Support/FIRContext.h" to access FIR, HLFIR, or related MLIR dialect definitions.
  **L10 CN**: 引入 "flang/Optimizer/Dialect/Support/FIRContext.h" 以使用FIR、HLFIR 或相关 MLIR 方言定义。
- **L11 EN**: Includes "flang/Optimizer/Support/FatalError.h" to access optimizer-side support routines and utilities.
  **L11 CN**: 引入 "flang/Optimizer/Support/FatalError.h" 以使用优化器侧支持例程与工具。
- **L12 EN**: Includes "mlir/Dialect/DLTI/DLTI.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L12 CN**: 引入 "mlir/Dialect/DLTI/DLTI.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L13 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L13 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L14 EN**: Includes "mlir/Dialect/LLVMIR/LLVMDialect.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L14 CN**: 引入 "mlir/Dialect/LLVMIR/LLVMDialect.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L15 EN**: Includes "mlir/IR/BuiltinOps.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L15 CN**: 引入 "mlir/IR/BuiltinOps.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L16 EN**: Includes "mlir/Interfaces/DataLayoutInterfaces.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L16 CN**: 引入 "mlir/Interfaces/DataLayoutInterfaces.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。

### Lines 17-32

````cpp
#include "mlir/Support/LLVM.h"
#include "mlir/Target/LLVMIR/Import.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/TargetSelect.h"
#include "llvm/Target/TargetMachine.h"

namespace {
template <typename ModOpTy>
static void setDataLayout(ModOpTy mlirModule, const llvm::DataLayout &dl) {
  mlir::MLIRContext *context = mlirModule.getContext();
  mlirModule->setAttr(
      mlir::LLVM::LLVMDialect::getDataLayoutAttrName(),
      mlir::StringAttr::get(context, dl.getStringRepresentation()));
  mlir::DataLayoutSpecInterface dlSpec = mlir::translateDataLayout(dl, context);
  mlirModule->setAttr(mlir::DLTIDialect::kDataLayoutAttrName, dlSpec);
````
- **L17 EN**: Includes "mlir/Support/LLVM.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L17 CN**: 引入 "mlir/Support/LLVM.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L18 EN**: Includes "mlir/Target/LLVMIR/Import.h" to access MLIR core IR, pass, or dialect infrastructure.
  **L18 CN**: 引入 "mlir/Target/LLVMIR/Import.h" 以使用MLIR 核心 IR、Pass 或方言基础设施。
- **L19 EN**: Includes "llvm/IR/DataLayout.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L19 CN**: 引入 "llvm/IR/DataLayout.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L20 EN**: Includes "llvm/MC/TargetRegistry.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L20 CN**: 引入 "llvm/MC/TargetRegistry.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L21 EN**: Includes "llvm/Support/TargetSelect.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L21 CN**: 引入 "llvm/Support/TargetSelect.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L22 EN**: Includes "llvm/Target/TargetMachine.h" to access LLVM support libraries, ADTs, or target-independent infrastructure.
  **L22 CN**: 引入 "llvm/Target/TargetMachine.h" 以使用LLVM 支持库、ADT 或目标无关基础设施。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L24 EN**: Opens namespace scope ``.
  **L24 CN**: 打开命名空间作用域 ``。
- **L25 EN**: Introduces template parameters or specialization context: `template <typename ModOpTy>`.
  **L25 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ModOpTy>`。
- **L26 EN**: Starts a function, method, lambda, or structured scope: `static void setDataLayout(ModOpTy mlirModule, const llvm::DataLayout &dl) {`.
  **L26 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static void setDataLayout(ModOpTy mlirModule, const llvm::DataLayout &dl) {`。
- **L27 EN**: Executes a call or declaration centered on `mlirModule.getContext`.
  **L27 CN**: 执行以 `mlirModule.getContext` 为核心的调用或声明。
- **L28 EN**: Continues logic associated with callable symbol `setAttr`.
  **L28 CN**: 继续与可调用符号 `setAttr` 相关的逻辑。
- **L29 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mlir::LLVM::LLVMDialect::getDataLayoutAttrName(),`.
  **L29 CN**: 继续一个多行参数列表、初始化器或聚合项：`mlir::LLVM::LLVMDialect::getDataLayoutAttrName(),`。
- **L30 EN**: Executes a call or declaration centered on `mlir::StringAttr::get`.
  **L30 CN**: 执行以 `mlir::StringAttr::get` 为核心的调用或声明。
- **L31 EN**: Initializes variable `dlSpec` from the right-hand expression.
  **L31 CN**: 使用右侧表达式初始化变量 `dlSpec`。
- **L32 EN**: Executes a call or declaration centered on `mlirModule->setAttr`.
  **L32 CN**: 执行以 `mlirModule->setAttr` 为核心的调用或声明。

### Lines 33-48

````cpp
}

template <typename ModOpTy>
static void setDataLayoutFromAttributes(ModOpTy mlirModule,
                                        bool allowDefaultLayout) {
  if (mlirModule.getDataLayoutSpec())
    return; // Already set.
  if (auto dataLayoutString =
          mlirModule->template getAttrOfType<mlir::StringAttr>(
              mlir::LLVM::LLVMDialect::getDataLayoutAttrName())) {
    llvm::DataLayout llvmDataLayout(dataLayoutString);
    fir::support::setMLIRDataLayout(mlirModule, llvmDataLayout);
    return;
  }
  if (!allowDefaultLayout)
    return;
````
- **L33 EN**: Closes the current lexical scope or compound statement.
  **L33 CN**: 结束当前词法作用域或复合语句块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Introduces template parameters or specialization context: `template <typename ModOpTy>`.
  **L35 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ModOpTy>`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void setDataLayoutFromAttributes(ModOpTy mlirModule,`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void setDataLayoutFromAttributes(ModOpTy mlirModule,`。
- **L37 EN**: Continues the surrounding expression or declaration: `bool allowDefaultLayout) {`.
  **L37 CN**: 继续构造周围的表达式或声明：`bool allowDefaultLayout) {`。
- **L38 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L38 CN**: 开始 `if` 控制流语句并计算其条件。
- **L39 EN**: Returns from the current function with `; // Already set.`.
  **L39 CN**: 以 `; // Already set.` 从当前函数返回。
- **L40 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L40 CN**: 开始 `if` 控制流语句并计算其条件。
- **L41 EN**: Continues logic associated with callable symbol `StringAttr>`.
  **L41 CN**: 继续与可调用符号 `StringAttr>` 相关的逻辑。
- **L42 EN**: Starts a function, method, lambda, or structured scope: `mlir::LLVM::LLVMDialect::getDataLayoutAttrName())) {`.
  **L42 CN**: 开始一个函数、方法、lambda 或结构化作用域：`mlir::LLVM::LLVMDialect::getDataLayoutAttrName())) {`。
- **L43 EN**: Executes a call or declaration centered on `llvmDataLayout`.
  **L43 CN**: 执行以 `llvmDataLayout` 为核心的调用或声明。
- **L44 EN**: Executes a call or declaration centered on `fir::support::setMLIRDataLayout`.
  **L44 CN**: 执行以 `fir::support::setMLIRDataLayout` 为核心的调用或声明。
- **L45 EN**: Returns from the current function with `void`.
  **L45 CN**: 以 `void` 从当前函数返回。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L47 CN**: 开始 `if` 控制流语句并计算其条件。
- **L48 EN**: Returns from the current function with `void`.
  **L48 CN**: 以 `void` 从当前函数返回。

### Lines 49-64

````cpp
  llvm::DataLayout llvmDataLayout("");
  fir::support::setMLIRDataLayout(mlirModule, llvmDataLayout);
}

template <typename ModOpTy>
static std::optional<mlir::DataLayout>
getOrSetDataLayout(ModOpTy mlirModule, bool allowDefaultLayout) {
  if (!mlirModule.getDataLayoutSpec())
    fir::support::setMLIRDataLayoutFromAttributes(mlirModule,
                                                  allowDefaultLayout);
  if (!mlirModule.getDataLayoutSpec() &&
      !mlir::isa<mlir::gpu::GPUModuleOp>(mlirModule))
    return std::nullopt;
  return mlir::DataLayout(mlirModule);
}

````
- **L49 EN**: Executes a call or declaration centered on `llvmDataLayout`.
  **L49 CN**: 执行以 `llvmDataLayout` 为核心的调用或声明。
- **L50 EN**: Executes a call or declaration centered on `fir::support::setMLIRDataLayout`.
  **L50 CN**: 执行以 `fir::support::setMLIRDataLayout` 为核心的调用或声明。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Introduces template parameters or specialization context: `template <typename ModOpTy>`.
  **L53 CN**: 为后续声明引入模板参数或特化上下文：`template <typename ModOpTy>`。
- **L54 EN**: Continues the surrounding expression or declaration: `static std::optional<mlir::DataLayout>`.
  **L54 CN**: 继续构造周围的表达式或声明：`static std::optional<mlir::DataLayout>`。
- **L55 EN**: Starts a function, method, lambda, or structured scope: `getOrSetDataLayout(ModOpTy mlirModule, bool allowDefaultLayout) {`.
  **L55 CN**: 开始一个函数、方法、lambda 或结构化作用域：`getOrSetDataLayout(ModOpTy mlirModule, bool allowDefaultLayout) {`。
- **L56 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L56 CN**: 开始 `if` 控制流语句并计算其条件。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::support::setMLIRDataLayoutFromAttributes(mlirModule,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::support::setMLIRDataLayoutFromAttributes(mlirModule,`。
- **L58 EN**: Executes a standalone statement or declaration: `allowDefaultLayout);`.
  **L58 CN**: 执行一条独立语句或声明：`allowDefaultLayout);`。
- **L59 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L59 CN**: 开始 `if` 控制流语句并计算其条件。
- **L60 EN**: Continues logic associated with callable symbol `GPUModuleOp>`.
  **L60 CN**: 继续与可调用符号 `GPUModuleOp>` 相关的逻辑。
- **L61 EN**: Returns from the current function with `std::nullopt`.
  **L61 CN**: 以 `std::nullopt` 从当前函数返回。
- **L62 EN**: Returns from the current function with `mlir::DataLayout(mlirModule)`.
  **L62 CN**: 以 `mlir::DataLayout(mlirModule)` 从当前函数返回。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 65-80

````cpp
} // namespace

void fir::support::setMLIRDataLayout(mlir::ModuleOp mlirModule,
                                     const llvm::DataLayout &dl) {
  setDataLayout(mlirModule, dl);
}

void fir::support::setMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,
                                     const llvm::DataLayout &dl) {
  setDataLayout(mlirModule, dl);
}

void fir::support::setMLIRDataLayoutFromAttributes(mlir::ModuleOp mlirModule,
                                                   bool allowDefaultLayout) {
  setDataLayoutFromAttributes(mlirModule, allowDefaultLayout);
}
````
- **L65 EN**: Closes a namespace scope with a trailing comment: `} // namespace`.
  **L65 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::support::setMLIRDataLayout(mlir::ModuleOp mlirModule,`.
  **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::support::setMLIRDataLayout(mlir::ModuleOp mlirModule,`。
- **L68 EN**: Continues the surrounding expression or declaration: `const llvm::DataLayout &dl) {`.
  **L68 CN**: 继续构造周围的表达式或声明：`const llvm::DataLayout &dl) {`。
- **L69 EN**: Executes a call or declaration centered on `setDataLayout`.
  **L69 CN**: 执行以 `setDataLayout` 为核心的调用或声明。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::support::setMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,`.
  **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::support::setMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,`。
- **L73 EN**: Continues the surrounding expression or declaration: `const llvm::DataLayout &dl) {`.
  **L73 CN**: 继续构造周围的表达式或声明：`const llvm::DataLayout &dl) {`。
- **L74 EN**: Executes a call or declaration centered on `setDataLayout`.
  **L74 CN**: 执行以 `setDataLayout` 为核心的调用或声明。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void fir::support::setMLIRDataLayoutFromAttributes(mlir::ModuleOp mlirModule,`.
  **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`void fir::support::setMLIRDataLayoutFromAttributes(mlir::ModuleOp mlirModule,`。
- **L78 EN**: Continues the surrounding expression or declaration: `bool allowDefaultLayout) {`.
  **L78 CN**: 继续构造周围的表达式或声明：`bool allowDefaultLayout) {`。
- **L79 EN**: Executes a call or declaration centered on `setDataLayoutFromAttributes`.
  **L79 CN**: 执行以 `setDataLayoutFromAttributes` 为核心的调用或声明。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。

### Lines 81-96

````cpp

void fir::support::setMLIRDataLayoutFromAttributes(
    mlir::gpu::GPUModuleOp mlirModule, bool allowDefaultLayout) {
  setDataLayoutFromAttributes(mlirModule, allowDefaultLayout);
}

std::optional<mlir::DataLayout>
fir::support::getOrSetMLIRDataLayout(mlir::ModuleOp mlirModule,
                                     bool allowDefaultLayout) {
  return getOrSetDataLayout(mlirModule, allowDefaultLayout);
}

std::optional<mlir::DataLayout>
fir::support::getOrSetMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,
                                     bool allowDefaultLayout) {
  return getOrSetDataLayout(mlirModule, allowDefaultLayout);
````
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Continues logic associated with callable symbol `setMLIRDataLayoutFromAttributes`.
  **L82 CN**: 继续与可调用符号 `setMLIRDataLayoutFromAttributes` 相关的逻辑。
- **L83 EN**: Continues the surrounding expression or declaration: `mlir::gpu::GPUModuleOp mlirModule, bool allowDefaultLayout) {`.
  **L83 CN**: 继续构造周围的表达式或声明：`mlir::gpu::GPUModuleOp mlirModule, bool allowDefaultLayout) {`。
- **L84 EN**: Executes a call or declaration centered on `setDataLayoutFromAttributes`.
  **L84 CN**: 执行以 `setDataLayoutFromAttributes` 为核心的调用或声明。
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L87 EN**: Continues the surrounding expression or declaration: `std::optional<mlir::DataLayout>`.
  **L87 CN**: 继续构造周围的表达式或声明：`std::optional<mlir::DataLayout>`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::support::getOrSetMLIRDataLayout(mlir::ModuleOp mlirModule,`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::support::getOrSetMLIRDataLayout(mlir::ModuleOp mlirModule,`。
- **L89 EN**: Continues the surrounding expression or declaration: `bool allowDefaultLayout) {`.
  **L89 CN**: 继续构造周围的表达式或声明：`bool allowDefaultLayout) {`。
- **L90 EN**: Returns from the current function with `getOrSetDataLayout(mlirModule, allowDefaultLayout)`.
  **L90 CN**: 以 `getOrSetDataLayout(mlirModule, allowDefaultLayout)` 从当前函数返回。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Continues the surrounding expression or declaration: `std::optional<mlir::DataLayout>`.
  **L93 CN**: 继续构造周围的表达式或声明：`std::optional<mlir::DataLayout>`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fir::support::getOrSetMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`fir::support::getOrSetMLIRDataLayout(mlir::gpu::GPUModuleOp mlirModule,`。
- **L95 EN**: Continues the surrounding expression or declaration: `bool allowDefaultLayout) {`.
  **L95 CN**: 继续构造周围的表达式或声明：`bool allowDefaultLayout) {`。
- **L96 EN**: Returns from the current function with `getOrSetDataLayout(mlirModule, allowDefaultLayout)`.
  **L96 CN**: 以 `getOrSetDataLayout(mlirModule, allowDefaultLayout)` 从当前函数返回。

### Lines 97-97

````cpp
}
````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **FIR/MLIR optimizer infrastructure / FIR/MLIR 优化器基础设施**
- **FIR dialect operations and helpers / FIR 方言操作与辅助能力**
- **MLIR IR and pass infrastructure / MLIR IR 与 Pass 基础设施**

## Dependencies / 依赖关系

- `flang/Optimizer/Support/DataLayout.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `flang/Optimizer/Dialect/Support/FIRContext.h`: Provides FIR, HLFIR, or related MLIR dialect definitions. / 提供FIR、HLFIR 或相关 MLIR 方言定义。
- `flang/Optimizer/Support/FatalError.h`: Provides optimizer-side support routines and utilities. / 提供优化器侧支持例程与工具。
- `mlir/Dialect/DLTI/DLTI.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Dialect/LLVMIR/LLVMDialect.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/IR/BuiltinOps.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Interfaces/DataLayoutInterfaces.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Support/LLVM.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `mlir/Target/LLVMIR/Import.h`: Provides MLIR core IR, pass, or dialect infrastructure. / 提供MLIR 核心 IR、Pass 或方言基础设施。
- `llvm/IR/DataLayout.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `llvm/MC/TargetRegistry.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `llvm/Support/TargetSelect.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
- `llvm/Target/TargetMachine.h`: Provides LLVM support libraries, ADTs, or target-independent infrastructure. / 提供LLVM 支持库、ADT 或目标无关基础设施。
