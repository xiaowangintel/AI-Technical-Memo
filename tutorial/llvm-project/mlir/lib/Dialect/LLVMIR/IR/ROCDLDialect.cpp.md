# ROCDLDialect.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/LLVMIR/IR/ROCDLDialect.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the types and operation details for the ROCDL IR dialect in MLIR, and the LLVM IR dialect.  It also registers the dialect.
  - **CN**: 实现 LLVM 方言家族与 LLVM IR 桥接支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ROCDLDialect.cpp - ROCDL IR Ops and Dialect registration -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-15
```cpp
//
// This file defines the types and operation details for the ROCDL IR dialect in
// MLIR, and the LLVM IR dialect.  It also registers the dialect.
//
// The ROCDL dialect only contains GPU specific additions on top of the general
// LLVM dialect.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 16-28
```cpp

#include "mlir/Dialect/LLVMIR/ROCDLDialect.h"

#include "mlir/Dialect/GPU/IR/CompilationInterfaces.h"
#include "mlir/Dialect/LLVMIR/LLVMDialect.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/DialectImplementation.h"
#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/Operation.h"
#include "mlir/Transforms/InliningUtils.h"
#include "llvm/ADT/TypeSwitch.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/IR/Builders.h`。

### Lines 29-36
```cpp
using namespace mlir;
using namespace ROCDL;

#include "mlir/Dialect/LLVMIR/ROCDLOpsDialect.cpp.inc"

//===----------------------------------------------------------------------===//
// Parsing for ROCDL ops
//===----------------------------------------------------------------------===//
```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

### Lines 37-48
```cpp

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.load.* %rsrc, %offset, %soffset, %aux
//     : result_type`
ParseResult RawBufferLoadOp::parse(OpAsmParser &parser,
                                   OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 4> ops;
  Type type;
  if (parser.parseOperandList(ops, 4) || parser.parseColonType(type) ||
      parser.addTypeToList(type, result.types))
    return failure();

```
- **EN**: Implements logic around `parse`, `parseOperandList`, `addTypeToList`, `failure`; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `parse`, `parseOperandList`, `addTypeToList`, `failure` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 49-55
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);
  return parser.resolveOperands(ops, {i32x4Ty, int32Ty, int32Ty, int32Ty},
                                parser.getNameLoc(), result.operands);
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 56-69
```cpp
void RawBufferLoadOp::print(OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getRes().getType();
}

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.store.* %vdata, %rsrc,  %offset,
//     %soffset, %aux : result_type`
ParseResult RawBufferStoreOp::parse(OpAsmParser &parser,
                                    OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 5> ops;
  Type type;
  if (parser.parseOperandList(ops, 5) || parser.parseColonType(type))
    return failure();

```
- **EN**: Implements logic around `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 70-79
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);

  if (parser.resolveOperands(ops, {type, i32x4Ty, int32Ty, int32Ty, int32Ty},
                             parser.getNameLoc(), result.operands))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 80-93
```cpp
void RawBufferStoreOp::print(OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getVdata().getType();
}

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.atomic.fadd.* %vdata, %rsrc,  %offset,
//     %soffset, %aux : result_type`
ParseResult RawBufferAtomicFAddOp::parse(OpAsmParser &parser,
                                         OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 5> ops;
  Type type;
  if (parser.parseOperandList(ops, 5) || parser.parseColonType(type))
    return failure();

```
- **EN**: Implements logic around `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 94-103
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);

  if (parser.resolveOperands(ops, {type, i32x4Ty, int32Ty, int32Ty, int32Ty},
                             parser.getNameLoc(), result.operands))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 104-117
```cpp
void RawBufferAtomicFAddOp::print(mlir::OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getVdata().getType();
}

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.atomic.fmax.* %vdata, %rsrc,  %offset,
//     %soffset, %aux : result_type`
ParseResult RawBufferAtomicFMaxOp::parse(OpAsmParser &parser,
                                         OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 5> ops;
  Type type;
  if (parser.parseOperandList(ops, 5) || parser.parseColonType(type))
    return failure();

```
- **EN**: Implements logic around `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 118-127
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);

  if (parser.resolveOperands(ops, {type, i32x4Ty, int32Ty, int32Ty, int32Ty},
                             parser.getNameLoc(), result.operands))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 128-141
```cpp
void RawBufferAtomicFMaxOp::print(mlir::OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getVdata().getType();
}

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.atomic.smax.* %vdata, %rsrc,  %offset,
//     %soffset, %aux : result_type`
ParseResult RawBufferAtomicSMaxOp::parse(OpAsmParser &parser,
                                         OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 5> ops;
  Type type;
  if (parser.parseOperandList(ops, 5) || parser.parseColonType(type))
    return failure();

```
- **EN**: Implements logic around `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 142-151
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);

  if (parser.resolveOperands(ops, {type, i32x4Ty, int32Ty, int32Ty, int32Ty},
                             parser.getNameLoc(), result.operands))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 152-165
```cpp
void RawBufferAtomicSMaxOp::print(mlir::OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getVdata().getType();
}

// <operation> ::=
//     `llvm.amdgcn.raw.buffer.atomic.umin.* %vdata, %rsrc,  %offset,
//     %soffset, %aux : result_type`
ParseResult RawBufferAtomicUMinOp::parse(OpAsmParser &parser,
                                         OperationState &result) {
  SmallVector<OpAsmParser::UnresolvedOperand, 5> ops;
  Type type;
  if (parser.parseOperandList(ops, 5) || parser.parseColonType(type))
    return failure();

```
- **EN**: Implements logic around `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols; this block implements verifier, folding, parsing, or printing hooks; moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `print`, `getOperands`, `parse`, `parseOperandList`, and 1 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并在张量风格值与显式缓冲区之间移动数据。

### Lines 166-175
```cpp
  auto bldr = parser.getBuilder();
  auto int32Ty = bldr.getI32Type();
  auto i32x4Ty = VectorType::get({4}, int32Ty);

  if (parser.resolveOperands(ops, {type, i32x4Ty, int32Ty, int32Ty, int32Ty},
                             parser.getNameLoc(), result.operands))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `getBuilder`, `getI32Type`, `get`, `resolveOperands`, and 3 more symbols 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 176-182
```cpp
void RawBufferAtomicUMinOp::print(mlir::OpAsmPrinter &p) {
  p << " " << getOperands() << " : " << getVdata().getType();
}

//===----------------------------------------------------------------------===//
// ROCDLDialect initialization, type parsing, and registration.
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `print`, `getOperands`; this block implements verifier, folding, parsing, or printing hooks.
- **CN**: 围绕 `print`, `getOperands` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子。

### Lines 183-192
```cpp

namespace {
struct ROCDLInlinerInterface final : DialectInlinerInterface {
  using DialectInlinerInterface::DialectInlinerInterface;
  bool isLegalToInline(Operation *, Region *, bool, IRMapping &) const final {
    return true;
  }
};
} // namespace

```
- **EN**: Introduces declarations for `ROCDLInlinerInterface`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `ROCDLInlinerInterface` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 193-199
```cpp
// TODO: This should be the llvm.rocdl dialect once this is supported.
void ROCDLDialect::initialize() {
  addOperations<
#define GET_OP_LIST
#include "mlir/Dialect/LLVMIR/ROCDLOps.cpp.inc"
      >();

```
- **EN**: Implements logic around `initialize`; this block registers dialect entities or dialect-level hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `initialize` 实现具体逻辑；该代码块注册方言实体或方言级钩子，并编码加速器专用执行或 lowering 规则。

### Lines 200-210
```cpp
  addAttributes<
#define GET_ATTRDEF_LIST
#include "mlir/Dialect/LLVMIR/ROCDLOpsAttributes.cpp.inc"
      >();

  // Support unknown operations because not all ROCDL operations are registered.
  allowUnknownOperations();
  addInterfaces<ROCDLInlinerInterface>();
  declarePromisedInterface<gpu::TargetAttrInterface, ROCDLTargetAttr>();
}

```
- **EN**: Implements logic around `allowUnknownOperations`, `addInterfaces`, `ROCDLTargetAttr>`; this block registers dialect entities or dialect-level hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `allowUnknownOperations`, `addInterfaces`, `ROCDLTargetAttr>` 实现具体逻辑；该代码块注册方言实体或方言级钩子，并编码加速器专用执行或 lowering 规则。

### Lines 211-222
```cpp
LogicalResult ROCDLDialect::verifyOperationAttribute(Operation *op,
                                                     NamedAttribute attr) {
  // Kernel function attribute should be attached to functions.
  if (kernelAttrName.getName() == attr.getName()) {
    if (!isa<LLVM::LLVMFuncOp>(op)) {
      return op->emitError() << "'" << kernelAttrName.getName()
                             << "' attribute attached to unexpected op";
    }
  }
  return success();
}

```
- **EN**: Implements logic around `verifyOperationAttribute`, `getName`, `emitError`, `success`.
- **CN**: 围绕 `verifyOperationAttribute`, `getName`, `emitError`, `success` 实现具体逻辑。

### Lines 223-236
```cpp
//===----------------------------------------------------------------------===//
// ROCDL target attribute.
//===----------------------------------------------------------------------===//
LogicalResult
ROCDLTargetAttr::verify(function_ref<InFlightDiagnostic()> emitError,
                        int optLevel, StringRef triple, StringRef chip,
                        StringRef features, StringRef abiVersion,
                        DictionaryAttr flags, ArrayAttr files) {
  if (optLevel < 0 || optLevel > 3) {
    emitError() << "The optimization level must be a number between 0 and 3.";
    return failure();
  }
  if (triple.empty()) {
    emitError() << "The target triple cannot be empty.";
```
- **EN**: Implements logic around `verify`, `emitError`, `failure`, `empty`; this block implements verifier, folding, parsing, or printing hooks; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `verify`, `emitError`, `failure`, `empty` 实现具体逻辑；该代码块实现验证、折叠、解析或打印钩子，并编码加速器专用执行或 lowering 规则。

### Lines 237-250
```cpp
    return failure();
  }
  if (chip.empty()) {
    emitError() << "The target chip cannot be empty.";
    return failure();
  }
  if (abiVersion != "400" && abiVersion != "500" && abiVersion != "600") {
    emitError() << "Invalid ABI version, it must be `400`, `500` or '600'.";
    return failure();
  }
  if (files && !llvm::all_of(files, [](::mlir::Attribute attr) {
        return attr && mlir::isa<StringAttr>(attr);
      })) {
    emitError() << "All the elements in the `link` array must be strings.";
```
- **EN**: Implements logic around `failure`, `empty`, `emitError`, `all_of`, and 1 more symbols.
- **CN**: 围绕 `failure`, `empty`, `emitError`, `all_of`, and 1 more symbols 实现具体逻辑。

### Lines 251-258
```cpp
    return failure();
  }
  return success();
}

#define GET_OP_CLASSES
#include "mlir/Dialect/LLVMIR/ROCDLOps.cpp.inc"

```
- **EN**: Implements logic around `failure`, `success`.
- **CN**: 围绕 `failure`, `success` 实现具体逻辑。

### Lines 259-260
```cpp
#define GET_ATTRDEF_CLASSES
#include "mlir/Dialect/LLVMIR/ROCDLOpsAttributes.cpp.inc"
```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **LLVM dialect bridging / LLVM 方言桥接**:
  - **EN**: Maps MLIR constructs to LLVM-compatible types, ops, intrinsics, and metadata.
  - **CN**: 将 MLIR 构造映射到兼容 LLVM 的类型、操作、Intrinsic 与元数据。
- **Dialect registration / 方言注册**:
  - **EN**: Registers operations, types, attributes, or interfaces into an MLIR dialect object.
  - **CN**: 把操作、类型、属性或接口注册到 MLIR 方言对象中。
- **Assembly format hooks / 汇编格式钩子**:
  - **EN**: Custom parsers and printers define how operations or attributes appear in MLIR assembly.
  - **CN**: 自定义解析器和打印器定义操作或属性在 MLIR 汇编中的呈现方式。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/Dialect/LLVMIR/LLVMDialect.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/DialectImplementation.h`, `mlir/IR/MLIRContext.h`, `mlir/IR/Operation.h`, `mlir/Transforms/InliningUtils.h`, `llvm/ADT/TypeSwitch.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: other MLIR dialect declarations / 其他 MLIR 方言声明 (6), MLIR IR core abstractions / MLIR IR 核心抽象 (5), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
- **Generated macros / 生成宏**: `GET_OP_LIST`, `GET_ATTRDEF_LIST`, `GET_OP_CLASSES`, `GET_ATTRDEF_CLASSES`
