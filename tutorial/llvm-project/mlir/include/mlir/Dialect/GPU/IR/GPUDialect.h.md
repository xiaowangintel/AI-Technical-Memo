# GPUDialect.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/IR/GPUDialect.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the GPU kernel-related operations and puts them in the corresponding dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/GPU/IR`，围绕 GPU 方言公开 `GPUDialect` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- GPUDialect.h - MLIR Dialect for GPU Kernels --------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp
//
// This file defines the GPU kernel-related operations and puts them in the
// corresponding dialect.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 14-25
```cpp
#ifndef MLIR_DIALECT_GPU_IR_GPUDIALECT_H
#define MLIR_DIALECT_GPU_IR_GPUDIALECT_H

#include "mlir/Bytecode/BytecodeOpInterface.h"
#include "mlir/Dialect/DLTI/Traits.h"
#include "mlir/Dialect/GPU/IR/CompilationInterfaces.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Dialect.h"
#include "mlir/IR/OpDefinition.h"
#include "mlir/IR/OpImplementation.h"
#include "mlir/IR/RegionKindInterface.h"
```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Bytecode/BytecodeOpInterface.h`, `mlir/Dialect/DLTI/Traits.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/IR/Builders.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Bytecode/BytecodeOpInterface.h`, `mlir/Dialect/DLTI/Traits.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/IR/Builders.h`。

### Lines 26-33
```cpp
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/ControlFlowInterfaces.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include "mlir/Interfaces/InferIntRangeInterface.h"
#include "mlir/Interfaces/InferTypeOpInterface.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "llvm/ADT/STLExtras.h"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/IR/SymbolTable.h`, `mlir/Interfaces/ControlFlowInterfaces.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Interfaces/InferIntRangeInterface.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/IR/SymbolTable.h`, `mlir/Interfaces/ControlFlowInterfaces.h`, `mlir/Interfaces/FunctionInterfaces.h`, `mlir/Interfaces/InferIntRangeInterface.h`。

### Lines 34-39
```cpp
namespace mlir {
namespace gpu {

/// Utility class for the GPU dialect to represent triples of `Value`s
/// accessible through `.x`, `.y`, and `.z` similarly to CUDA notation.
struct KernelDim3 {
```
- **EN**: Introduces declarations for `mlir`, `gpu`, `for`, `KernelDim3`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `gpu`, `for`, `KernelDim3` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 40-45
```cpp
  Value x;
  Value y;
  Value z;
};

class AsyncTokenType
```
- **EN**: Introduces declarations for `AsyncTokenType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `AsyncTokenType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 46-53
```cpp
    : public Type::TypeBase<AsyncTokenType, Type, TypeStorage> {
public:
  // Used for generic hooks in TypeBase.
  using Base::Base;

  static constexpr StringLiteral name = "gpu.async_token";
};

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 54-61
```cpp
/// MMAMatrixType storage and uniquing. Array is uniqued based on its shape
/// and type.
struct MMAMatrixStorageType : public TypeStorage {
  MMAMatrixStorageType(unsigned numDims, const int64_t *dimShapes,
                       Type elementType, StringRef operand)
      : dimShapes(dimShapes), numDims(numDims), elementType(elementType),
        operand(operand) {}

```
- **EN**: Introduces declarations for `MMAMatrixStorageType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MMAMatrixStorageType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 62-67
```cpp
  /// The hash key for uniquing.
  using KeyTy = std::tuple<ArrayRef<int64_t>, Type, StringRef>;
  bool operator==(const KeyTy &key) const {
    return key == KeyTy(getShape(), elementType, operand);
  }

```
- **EN**: Implements logic around `KeyTy`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `KeyTy` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 68-73
```cpp
  /// Construction.
  static MMAMatrixStorageType *construct(TypeStorageAllocator &allocator,
                                         const KeyTy &key) {
    ArrayRef<int64_t> shape = allocator.copyInto(std::get<0>(key));
    StringRef operand = allocator.copyInto(std::get<2>(key));

```
- **EN**: Implements logic around `construct`, `copyInto`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `construct`, `copyInto` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 74-82
```cpp
    return new (allocator.allocate<MMAMatrixStorageType>())
        MMAMatrixStorageType(shape.size(), shape.data(), std::get<1>(key),
                             operand);
  }

  ArrayRef<int64_t> getShape() const {
    return ArrayRef<int64_t>(dimShapes, numDims);
  }

```
- **EN**: Implements logic around `new`, `MMAMatrixStorageType`, `getShape`, `ArrayRef`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `new`, `MMAMatrixStorageType`, `getShape`, `ArrayRef` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 83-88
```cpp
  StringRef getOperand() const { return operand; }

  /// Reference to the shape of the MMA matrix.
  const int64_t *dimShapes;

  /// Number of dimensions in the MMA matrix.
```
- **EN**: Implements logic around `getOperand`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getOperand` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 89-94
```cpp
  unsigned numDims;

  /// Element type of elements held in the MMA matrix.
  Type elementType;

  /// MMA operand that this MMAMatrix holds. The general form of operation this
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 95-100
```cpp
  /// type supports is given by the equation C += A*B. This field specifies
  /// which operand in the given equation is held by this type. The valid values
  /// are "AOp", "BOp" and "COp".
  StringRef operand;
};

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 101-106
```cpp
/// MMAMatrix represents a matrix held by a subgroup for matrix-matrix multiply
/// accumulate operations. MMAMatrices are taken as direct operands by these
/// operations and are also produced as results. These matrices are meant to
/// reside in the registers. A limited number of pointwise operations can be
/// performed on these matrices, i.e., operations which operate uniformly on
/// all the elements in the matrix and do not change the order of matrix
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 107-112
```cpp
/// elements. The above conditions exist because the layout of matrix elements
/// inside the matrix is opaque i.e., the elements may be present in the
/// matrix in any order. The general usage of this type is shown as follows:-
///
///   %0 = gpu.subgroup_mma_load_matrix %arg0[%c0, %c0] {leadDimension = 16 :
///           index} : memref<16x16xf16> -> !gpu.mma_matrix<16x16xf16, "AOp">
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 113-118
```cpp
///
/// The MMAMatrixType describes the shape of the matrix being loaded and the
/// operand being loaded too. The operand needs to be specified to aid the
/// lowering of this type to dialects such as NVVM where each workitem may
/// hold different amount of elements depending on the elementType of the
/// matrix. For e.g., Each workitem holds 4 vector<2xf16>s for f16 data type
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 119-124
```cpp
/// and 8 f32s for f32 data type of MMAMatrix. Some other instances of usage
/// are:-
///
///   %3 = gpu.subgroup_mma_compute %0, %1, %2 :
///   !gpu.mma_matrix<16x16xf16, "AOp">, !gpu.mma_matrix<16x16xf16, "BOp">
///    -> !gpu.mma_matrix<16x16xf32, "COp">
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 125-130
```cpp
///
///
///   gpu.subgroup_mma_store_matrix %3, %arg22[%c0, %c0] {leadDimension = 16
///           : index}: !gpu.mma_matrix<16x16xf32, "COp">, memref<16x16xf32>
// TODO: consider moving this to ODS.
class MMAMatrixType
```
- **EN**: Introduces declarations for `MMAMatrixType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MMAMatrixType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 131-136
```cpp
    : public Type::TypeBase<MMAMatrixType, Type, MMAMatrixStorageType> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "gpu.mma_matrix";

```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 137-142
```cpp
  /// Get MMAMatrixType and verify construction Invariants.
  static MMAMatrixType get(ArrayRef<int64_t> shape, Type elementType,
                           StringRef operand);

  /// Get MMAMatrixType at a particular location and verify construction
  /// Invariants.
```
- **EN**: Declares APIs or declarative rules around `get`; this block checks structural or semantic invariants; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `get` 相关的 API 或声明式规则；该代码块检查结构或语义不变式，并处理方言 IR 实体，如操作、类型或属性。

### Lines 143-149
```cpp
  static MMAMatrixType getChecked(function_ref<InFlightDiagnostic()> emitError,
                                  ArrayRef<int64_t> shape, Type elementType,
                                  StringRef operand);

  /// Check if a type is valid a MMAMatrixType elementType.
  static bool isValidElementType(Type elementType);

```
- **EN**: Declares APIs or declarative rules around `getChecked`, `isValidElementType`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `getChecked`, `isValidElementType` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 150-156
```cpp
  /// Verify that shape and elementType are actually allowed for the
  /// MMAMatrixType.
  static LogicalResult
  verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                   ArrayRef<int64_t> shape, Type elementType,
                   StringRef operand);

```
- **EN**: Declares APIs or declarative rules around `verifyInvariants`; this block checks structural or semantic invariants; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `verifyInvariants` 相关的 API 或声明式规则；该代码块检查结构或语义不变式，并处理方言 IR 实体，如操作、类型或属性。

### Lines 157-162
```cpp
  /// Get number of dims.
  unsigned getNumDims() const;

  /// Get shape of the matrix.
  ArrayRef<int64_t> getShape() const;

```
- **EN**: Declares APIs or declarative rules around `getNumDims`, `getShape`.
- **CN**: 声明与 `getNumDims`, `getShape` 相关的 API 或声明式规则。

### Lines 163-168
```cpp
  /// Get elementType of a single element.
  Type getElementType() const;

  /// The general form of operation this type supports is given by the equation
  /// C += A*B. This function returns which operand in the given equation is
  /// held by this type. String returned can be one of"AOp", "BOp" and "COp".
```
- **EN**: Declares APIs or declarative rules around `getElementType`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `getElementType` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 169-174
```cpp
  StringRef getOperand() const;
};

// Adds a `gpu.async.token` to the front of the argument list.
void addAsyncDependency(Operation *op, Value token);

```
- **EN**: Declares APIs or declarative rules around `getOperand`, `addAsyncDependency`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `getOperand`, `addAsyncDependency` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 175-180
```cpp
// Handle types for sparse.
enum class SparseHandleKind { SpMat, DnTensor, SpGEMMOp };

class SparseDnTensorHandleType
    : public Type::TypeBase<SparseDnTensorHandleType, Type, TypeStorage> {
public:
```
- **EN**: Introduces declarations for `SparseHandleKind`, `SparseDnTensorHandleType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SparseHandleKind`, `SparseDnTensorHandleType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 181-187
```cpp
  using Base =
      Type::TypeBase<SparseDnTensorHandleType, Type, TypeStorage>::Base;
  using Base::Base;

  static constexpr StringLiteral name = "gpu.sparse.dntensor_handle";
};

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 188-193
```cpp
class SparseSpMatHandleType
    : public Type::TypeBase<SparseSpMatHandleType, Type, TypeStorage> {
public:
  using Base = Type::TypeBase<SparseSpMatHandleType, Type, TypeStorage>::Base;
  using Base::Base;

```
- **EN**: Introduces declarations for `SparseSpMatHandleType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SparseSpMatHandleType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 194-199
```cpp
  static constexpr StringLiteral name = "gpu.sparse.spmat_handle";
};

class SparseSpGEMMOpHandleType
    : public Type::TypeBase<SparseSpGEMMOpHandleType, Type, TypeStorage> {
public:
```
- **EN**: Introduces declarations for `SparseSpGEMMOpHandleType`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `SparseSpGEMMOpHandleType` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 200-206
```cpp
  using Base =
      Type::TypeBase<SparseSpGEMMOpHandleType, Type, TypeStorage>::Base;
  using Base::Base;

  static constexpr StringLiteral name = "gpu.sparse.spgemmop_handle";
};

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 207-213
```cpp
} // namespace gpu
} // namespace mlir

#include "mlir/Dialect/GPU/IR/GPUOpsEnums.h.inc"

#include "mlir/Dialect/GPU/IR/GPUOpsDialect.h.inc"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/GPU/IR/GPUOpsEnums.h.inc`, `mlir/Dialect/GPU/IR/GPUOpsDialect.h.inc`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/GPU/IR/GPUOpsEnums.h.inc`, `mlir/Dialect/GPU/IR/GPUOpsDialect.h.inc`。

### Lines 214-220
```cpp
#include "mlir/Dialect/GPU/IR/GPUOpInterfaces.h.inc"

#include "mlir/Dialect/SCF/IR/DeviceMappingInterface.h"

#define GET_ATTRDEF_CLASSES
#include "mlir/Dialect/GPU/IR/GPUOpsAttributes.h.inc"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/GPU/IR/GPUOpInterfaces.h.inc`, `mlir/Dialect/SCF/IR/DeviceMappingInterface.h`, `mlir/Dialect/GPU/IR/GPUOpsAttributes.h.inc`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/GPU/IR/GPUOpInterfaces.h.inc`, `mlir/Dialect/SCF/IR/DeviceMappingInterface.h`, `mlir/Dialect/GPU/IR/GPUOpsAttributes.h.inc`。

### Lines 221-226
```cpp
#define GET_OP_CLASSES
#include "mlir/Dialect/GPU/IR/GPUOps.h.inc"

namespace mlir::gpu {
/// Retrieve the constant bounds for a given dimension and dimension kind
/// from the context surrounding `op`, if known, and return them. This will
```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Dialect/GPU/IR/GPUOps.h.inc`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Dialect/GPU/IR/GPUOps.h.inc`。

### Lines 227-232
```cpp
/// check the bounds on an enclosing `gpu.launch`, an enclosing `gpu.func`, and
/// any `gpu.known_*_size` on other function-like operations, in that order.
std::optional<uint32_t>
getKnownDimensionSizeAround(Operation *op, DimensionKind kind, Dimension dim);
} // namespace mlir::gpu
#endif // MLIR_DIALECT_GPU_IR_GPUDIALECT_H
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Describes the declarative or C++ interface for dialect operations, attributes, types, and registration helpers.
  - **CN**: 描述方言操作、属性、类型及注册辅助逻辑的声明式或 C++ 接口。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Bytecode/BytecodeOpInterface.h`, `mlir/Dialect/DLTI/Traits.h`, `mlir/Dialect/GPU/IR/CompilationInterfaces.h`, `mlir/IR/Builders.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Dialect.h`, `mlir/IR/OpDefinition.h`, `mlir/IR/OpImplementation.h`, `mlir/IR/RegionKindInterface.h`, `mlir/IR/SymbolTable.h` ... (+12 more)
- **Subsystem categories / 子系统类别**: dialect-specific operations, attributes, types, transforms, or interface declarations / 方言相关的操作、属性、类型、变换或接口声明 (8), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (7), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (5), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (1)
- **Generated macros / 生成宏**: `GET_ATTRDEF_CLASSES`, `GET_OP_CLASSES`
