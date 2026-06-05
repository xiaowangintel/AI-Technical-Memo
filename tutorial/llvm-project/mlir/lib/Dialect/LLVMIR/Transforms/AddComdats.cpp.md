# AddComdats.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/LLVMIR/Transforms/AddComdats.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements rewrite patterns and transformation passes for the LLVM dialect family and LLVM IR bridging support.
  - **CN**: 实现 LLVM 方言家族与 LLVM IR 桥接支持 的重写模式与变换 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- AddComdats.cpp - Add comdats to linkonce functions -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-12
```cpp

#include "mlir/Dialect/LLVMIR/Transforms/AddComdats.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/Pass/Pass.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/LLVMIR/Transforms/AddComdats.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Pass/Pass.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/LLVMIR/Transforms/AddComdats.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Pass/Pass.h`。

### Lines 13-19
```cpp
namespace mlir {
namespace LLVM {
#define GEN_PASS_DEF_LLVMADDCOMDATS
#include "mlir/Dialect/LLVMIR/Transforms/Passes.h.inc"
} // namespace LLVM
} // namespace mlir

```
- **EN**: Introduces declarations for `mlir`, `LLVM`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `mlir`, `LLVM` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 20-27
```cpp
using namespace mlir;

static void addComdat(LLVM::LLVMFuncOp &op, OpBuilder &builder,
                      SymbolTable &symbolTable, ModuleOp &module) {
  const char *comdatName = "__llvm_comdat";
  mlir::LLVM::ComdatOp comdatOp =
      symbolTable.lookup<mlir::LLVM::ComdatOp>(comdatName);
  if (!comdatOp) {
```
- **EN**: Implements logic around `addComdat`, `ComdatOp>`.
- **CN**: 围绕 `addComdat`, `ComdatOp>` 实现具体逻辑。

### Lines 28-34
```cpp
    PatternRewriter::InsertionGuard guard(builder);
    builder.setInsertionPointToStart(module.getBody());
    comdatOp =
        mlir::LLVM::ComdatOp::create(builder, module.getLoc(), comdatName);
    symbolTable.insert(comdatOp);
  }

```
- **EN**: Implements logic around `guard`, `setInsertionPointToStart`, `create`, `insert`.
- **CN**: 围绕 `guard`, `setInsertionPointToStart`, `create`, `insert` 实现具体逻辑。

### Lines 35-42
```cpp
  PatternRewriter::InsertionGuard guard(builder);
  builder.setInsertionPointToStart(&comdatOp.getBody().back());
  auto selectorOp = mlir::LLVM::ComdatSelectorOp::create(
      builder, comdatOp.getLoc(), op.getSymName(),
      mlir::LLVM::comdat::Comdat::Any);
  op.setComdatAttr(mlir::SymbolRefAttr::get(
      builder.getContext(), comdatName,
      mlir::FlatSymbolRefAttr::get(selectorOp.getSymNameAttr())));
```
- **EN**: Implements logic around `guard`, `setInsertionPointToStart`, `create`, `getLoc`, and 3 more symbols.
- **CN**: 围绕 `guard`, `setInsertionPointToStart`, `create`, `getLoc`, and 3 more symbols 实现具体逻辑。

### Lines 43-46
```cpp
}

namespace {
struct AddComdatsPass : public LLVM::impl::LLVMAddComdatsBase<AddComdatsPass> {
```
- **EN**: Introduces declarations for `AddComdatsPass`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `AddComdatsPass` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 47-50
```cpp
  void runOnOperation() override {
    OpBuilder builder{&getContext()};
    ModuleOp mod = getOperation();

```
- **EN**: Implements logic around `runOnOperation`, `getContext`, `getOperation`; this block participates in pass execution or pass construction.
- **CN**: 围绕 `runOnOperation`, `getContext`, `getOperation` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造。

### Lines 51-58
```cpp
    std::unique_ptr<SymbolTable> symbolTable;
    auto getSymTab = [&]() -> SymbolTable & {
      if (!symbolTable)
        symbolTable = std::make_unique<SymbolTable>(mod);
      return *symbolTable;
    };
    for (auto op : mod.getBody()->getOps<LLVM::LLVMFuncOp>()) {
      if (op.getLinkage() == LLVM::Linkage::Linkonce ||
```
- **EN**: Implements logic around `make_unique`, `getBody`, `getLinkage`.
- **CN**: 围绕 `make_unique`, `getBody`, `getLinkage` 实现具体逻辑。

### Lines 59-65
```cpp
          op.getLinkage() == LLVM::Linkage::LinkonceODR) {
        addComdat(op, builder, getSymTab(), mod);
      }
    }
  }
};
} // namespace
```
- **EN**: Implements logic around `getLinkage`, `addComdat`.
- **CN**: 围绕 `getLinkage`, `addComdat` 实现具体逻辑。

## Key Concepts / 关键概念

- **Pattern rewriting and passes / 模式重写与 Pass**:
  - **EN**: Applies conversion patterns, canonicalizations, or pass pipelines over MLIR operations.
  - **CN**: 对 MLIR 操作应用转换模式、规范化或 pass 流水线。
- **LLVM dialect bridging / LLVM 方言桥接**:
  - **EN**: Maps MLIR constructs to LLVM-compatible types, ops, intrinsics, and metadata.
  - **CN**: 将 MLIR 构造映射到兼容 LLVM 的类型、操作、Intrinsic 与元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/LLVMIR/Transforms/AddComdats.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/Pass/Pass.h`, `mlir/Dialect/LLVMIR/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (3), pass manager and pass registration support / Pass 管理器与 pass 注册支持 (1)
