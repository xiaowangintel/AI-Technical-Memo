# ModuleCombiner.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/Linking/ModuleCombiner.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR ModuleCombiner component. The leading comments describe it as: This file declares the entry point to the SPIR-V module combiner library.
- **用途（CN）**: 声明 MLIR ModuleCombiner 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
````cpp
//===- ModuleCombiner.h - MLIR SPIR-V Module Combiner -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the entry point to the SPIR-V module combiner library.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_LINKING_MODULECOMBINER_H_
#define MLIR_DIALECT_SPIRV_LINKING_MODULECOMBINER_H_

#include "mlir/IR/OwningOpRef.h"
#include "mlir/Support/LLVM.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 19-74
````cpp
namespace mlir {
class OpBuilder;

namespace spirv {
class ModuleOp;

/// The listener function to receive symbol renaming events.
///
/// `originalModule` is the input spirv::ModuleOp that contains the renamed
/// symbol. `oldSymbol` and `newSymbol` are the original and renamed symbol.
/// Note that it's the responsibility of the caller to properly retain the
/// storage underlying the passed StringRefs if the listener callback outlives
/// this function call.
using SymbolRenameListener = function_ref<void(
    spirv::ModuleOp originalModule, StringRef oldSymbol, StringRef newSymbol)>;

/// Combines a list of SPIR-V `inputModules` into one. Returns the combined
/// module on success; returns a null module otherwise.
//
/// \param inputModules the list of modules to combine. They won't be modified.
/// \param combinedMdouleBuilder an OpBuilder for building the combined module.
/// \param symbRenameListener a listener that gets called everytime a symbol in
///                           one of the input modules is renamed.
///
/// To combine multiple SPIR-V modules, we move all the module-level ops
/// from all the input modules into one big combined module. To that end, the
/// combination process proceeds in 2 phases:
///
/// 1. resolve conflicts between pairs of ops from different modules,
/// 2. deduplicate equivalent ops/sub-ops in the merged module.
///
/// For the conflict resolution phase, the following rules are employed to
/// resolve such conflicts:
///
/// - If 2 spirv.func's have the same symbol name, then rename one of the
///   functions.
/// - If an spirv.func and another op have the same symbol name, then rename the
///   other symbol.
/// - If none of the 2 conflicting ops are spirv.func, then rename either.
///
/// For deduplication, the following 3 cases are taken into consideration:
///
/// - If 2 spirv.GlobalVariable's have either the same descriptor set + binding
///   or the same build_in attribute value, then replace one of them using the
///   other.
/// - If 2 spirv.SpecConstant's have the same spec_id attribute value, then
///   replace one of them using the other.
/// - Deduplicating functions are not supported right now.
///
/// In all cases, the references to the updated symbol (whether renamed or
/// deduplicated) are also updated to reflect the change.
OwningOpRef<spirv::ModuleOp> combine(ArrayRef<spirv::ModuleOp> inputModules,
                                     OpBuilder &combinedModuleBuilder,
                                     SymbolRenameListener symRenameListener);
} // namespace spirv
} // namespace mlir
````
- **EN**: This C++ declaration introduces `OpBuilder` and establishes part of the API surface for `ModuleCombiner`. Representative entry points here include `void`, `combine`.
- **CN**: 该 C++ 声明引入了 `OpBuilder`，并构成 `ModuleCombiner` API 表面的一部分。 这一段可见的代表性接口包括 `void`, `combine`。

### Lines 75-75
````cpp
#endif // MLIR_DIALECT_SPIRV_LINKING_MODULECOMBINER_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- mlir/IR/OwningOpRef.h
- mlir/Support/LLVM.h
