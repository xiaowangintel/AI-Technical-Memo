# GPUBase.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/IR/GPUBase.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines the GPU dialect.
  - **CN**: 该文件位于 `mlir/include/mlir/Dialect/GPU/IR`，围绕 GPU 方言公开 `GPUBase` 相关的接口、规则或生成式定义。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```tablegen
//===-- GPUBase.td - GPU dialect definitions ---------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-15
```tablegen
//
// Defines the GPU dialect
//
//===----------------------------------------------------------------------===//

#ifndef GPU_BASE
#define GPU_BASE

```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

### Lines 16-22
```tablegen
include "mlir/IR/AttrTypeBase.td"
include "mlir/IR/EnumAttr.td"
include "mlir/IR/OpBase.td"

//===----------------------------------------------------------------------===//
// GPU Dialect.
//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 23-28
```tablegen

def GPU_Dialect : Dialect {
  let name = "gpu";
  let cppNamespace = "::mlir::gpu";
  let hasOperationAttrVerify = 1;

```
- **EN**: Introduces declarations for `GPU_Dialect`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_Dialect` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 29-35
```tablegen
  let extraClassDeclaration = [{
    /// Get the name of the attribute used to annotate the modules that contain
    /// kernel modules.
    static StringRef getContainerModuleAttrName() {
      return "gpu.container_module";
    }
    /// Get the name of the attribute used to annotate external kernel
```
- **EN**: Implements logic around `getContainerModuleAttrName`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getContainerModuleAttrName` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 36-42
```tablegen
    /// functions.
    static StringRef getKernelFuncAttrName() { return "gpu.kernel"; }

    /// Returns whether the given function is a kernel function, i.e., has the
    /// 'gpu.kernel' attribute.
    static bool isKernel(Operation *op);

```
- **EN**: Implements logic around `getKernelFuncAttrName`, `isKernel`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getKernelFuncAttrName`, `isKernel` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 43-48
```tablegen
    /// Returns the number of workgroup (thread, block) dimensions supported in
    /// the GPU dialect.
    // TODO: consider generalizing this.
    static unsigned getNumWorkgroupDimensions() { return 3; }

    /// Returns the numeric value used to identify the workgroup memory address
```
- **EN**: Implements logic around `getNumWorkgroupDimensions`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getNumWorkgroupDimensions` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 49-55
```tablegen
    /// space.
    static AddressSpace getWorkgroupAddressSpace() { return AddressSpace::Workgroup; }

    /// Returns the numeric value used to identify the private memory address
    /// space.
    static AddressSpace getPrivateAddressSpace() { return AddressSpace::Private; }

```
- **EN**: Implements logic around `getWorkgroupAddressSpace`, `getPrivateAddressSpace`.
- **CN**: 围绕 `getWorkgroupAddressSpace`, `getPrivateAddressSpace` 实现具体逻辑。

### Lines 56-61
```tablegen
    /// Returns the numeric value used to identify the constant memory address
    /// space.
    static AddressSpace getConstantAddressSpace() { return AddressSpace::Constant; }

    /// Return true if the given MemRefType has an address space that matches
    /// with the gpu::AddressSpaceAttr attribute with value 'workgroup`.
```
- **EN**: Implements logic around `getConstantAddressSpace`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getConstantAddressSpace` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 62-67
```tablegen
    static bool hasWorkgroupMemoryAddressSpace(MemRefType type);

    /// Return true if the given Attribute is an gpu::AddressSpaceAttr
    /// attribute with value 'workgroup`.
    static bool isWorkgroupMemoryAddressSpace(Attribute memorySpace);

```
- **EN**: Declares APIs or declarative rules around `hasWorkgroupMemoryAddressSpace`, `isWorkgroupMemoryAddressSpace`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `hasWorkgroupMemoryAddressSpace`, `isWorkgroupMemoryAddressSpace` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 68-73
```tablegen
    /// Return true if the given MemRefType has an address space that matches
    /// with the gpu::AddressSpaceAttr attribute with value 'constant`.
    static bool hasConstantMemoryAddressSpace(MemRefType type);

    /// Return true if the given Attribute is an gpu::AddressSpaceAttr
    /// attribute with value 'constant`.
```
- **EN**: Declares APIs or declarative rules around `hasConstantMemoryAddressSpace`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `hasConstantMemoryAddressSpace` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 74-81
```tablegen
    static bool isConstantMemoryAddressSpace(Attribute memorySpace);
  }];
  let discardableAttrs = (ins
    "::mlir::DenseI32ArrayAttr":$known_block_size,
    "::mlir::DenseI32ArrayAttr":$known_grid_size,
    "::mlir::DenseI32ArrayAttr":$known_cluster_size
  );

```
- **EN**: Declares APIs or declarative rules around `isConstantMemoryAddressSpace`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 声明与 `isConstantMemoryAddressSpace` 相关的 API 或声明式规则；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 82-87
```tablegen
  let dependentDialects = ["arith::ArithDialect"];
  let useDefaultAttributePrinterParser = 1;
  let useDefaultTypePrinterParser = 1;
}

//===----------------------------------------------------------------------===//
```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 88-96
```tablegen
// GPU Enums.
//===----------------------------------------------------------------------===//

class GPU_I32Enum<string name, string description, list<I32EnumAttrCase> cases>
    : I32EnumAttr<name, description, cases> {
  let genSpecializedAttr = 0;
  let cppNamespace = "::mlir::gpu";
}
class GPU_I32EnumAttr<string mnemonic, GPU_I32Enum enumInfo> :
```
- **EN**: Introduces declarations for `GPU_I32Enum`, `GPU_I32EnumAttr`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_I32Enum`, `GPU_I32EnumAttr` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 97-102
```tablegen
    EnumAttr<GPU_Dialect, enumInfo, mnemonic> {
  let assemblyFormat = "`<` $value `>`";
}

def GPU_AddressSpaceGlobal : I32EnumAttrCase<"Global", 1, "global">;
def GPU_AddressSpaceWorkgroup : I32EnumAttrCase<"Workgroup", 2, "workgroup">;
```
- **EN**: Introduces declarations for `GPU_AddressSpaceGlobal`, `GPU_AddressSpaceWorkgroup`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AddressSpaceGlobal`, `GPU_AddressSpaceWorkgroup` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 103-112
```tablegen
def GPU_AddressSpacePrivate : I32EnumAttrCase<"Private", 3, "private">;
def GPU_AddressSpaceConstant : I32EnumAttrCase<"Constant", 4, "constant">;
def GPU_AddressSpaceEnum : GPU_I32Enum<
  "AddressSpace", "GPU address space", [
    GPU_AddressSpaceGlobal,
    GPU_AddressSpaceWorkgroup,
    GPU_AddressSpacePrivate,
    GPU_AddressSpaceConstant
  ]>;

```
- **EN**: Introduces declarations for `GPU_AddressSpacePrivate`, `GPU_AddressSpaceConstant`, `GPU_AddressSpaceEnum`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AddressSpacePrivate`, `GPU_AddressSpaceConstant`, `GPU_AddressSpaceEnum` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 113-118
```tablegen
def GPU_AddressSpaceAttr :
  GPU_I32EnumAttr<"address_space", GPU_AddressSpaceEnum>;

def GPU_AddressSpaceAttrArray : TypedArrayAttrBase<GPU_AddressSpaceAttr, "GPU Address Space array">;

def GPU_Dimension : GPU_I32Enum<"Dimension",
```
- **EN**: Introduces declarations for `GPU_AddressSpaceAttr`, `GPU_AddressSpaceAttrArray`, `GPU_Dimension`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AddressSpaceAttr`, `GPU_AddressSpaceAttrArray`, `GPU_Dimension` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 119-125
```tablegen
    "a dimension, either 'x', 'y', or 'z'",
    [
      I32EnumAttrCase<"x", 0>,
      I32EnumAttrCase<"y", 1>,
      I32EnumAttrCase<"z", 2>
    ]>;

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 126-137
```tablegen
def GPU_DimensionAttr : EnumAttr<GPU_Dialect, GPU_Dimension, "dim">;

def GPU_DimensionKind : I32Enum<"DimensionKind",
  "the possible kinds of launch dimension",
  [
    I32EnumCase<"Other", 0, "other">,
    I32EnumCase<"Block", 1, "block">,
    I32EnumCase<"Grid", 2, "grid">,
    I32EnumCase<"Cluster", 3, "cluster">
  ]> {
  let cppNamespace = "::mlir::gpu";
}
```
- **EN**: Introduces declarations for `GPU_DimensionAttr`, `GPU_DimensionKind`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_DimensionAttr`, `GPU_DimensionKind` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 138-143
```tablegen

//===----------------------------------------------------------------------===//
// GPU Types.
//===----------------------------------------------------------------------===//

def GPU_AsyncToken : DialectType<
```
- **EN**: Introduces declarations for `GPU_AsyncToken`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AsyncToken` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 144-149
```tablegen
  GPU_Dialect, CPred<"::llvm::isa<::mlir::gpu::AsyncTokenType>($_self)">, "async token type">,
             BuildableType<"mlir::gpu::AsyncTokenType::get($_builder.getContext())">;

// Predicat to check if type is gpu::MMAMatrixType.
def IsMMAMatrixTypePred : CPred<"::llvm::isa<::mlir::gpu::MMAMatrixType>($_self)">;

```
- **EN**: Introduces declarations for `IsMMAMatrixTypePred`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `IsMMAMatrixTypePred` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 150-155
```tablegen
def GPU_MMAMatrix : DialectType<
  GPU_Dialect, IsMMAMatrixTypePred, "MMAMatrix type">;

// Memref type acceptable to gpu.subgroup_mma_{load|store}_matrix ops.
def GPU_MMAMemRef : MemRefOf<[I8, I32, F16, F32, F64, VectorOfRankAndType<[1], [I8, I32, F16, F32, F64]>]>;

```
- **EN**: Introduces declarations for `GPU_MMAMatrix`, `GPU_MMAMemRef`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_MMAMatrix`, `GPU_MMAMemRef` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 156-162
```tablegen
class MMAMatrixOf<list<Type> allowedTypes> :
  ContainerType<AnyTypeOf<allowedTypes>, IsMMAMatrixTypePred,
  "::llvm::cast<::mlir::gpu::MMAMatrixType>($_self).getElementType()",
  "gpu.mma_matrix", "::mlir::gpu::MMAMatrixType">;

// Types for all sparse handles.
class GPU_SparseHandle<string typeStr, string description> :
```
- **EN**: Introduces declarations for `MMAMatrixOf`, `GPU_SparseHandle`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `MMAMatrixOf`, `GPU_SparseHandle` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 163-168
```tablegen
    DialectType<GPU_Dialect,
    CPred<"llvm::isa<::mlir::gpu::"#typeStr#">($_self)">,
    description#" handle type">,
  BuildableType<"mlir::gpu::"#typeStr#"::get($_builder.getContext())">;

def GPU_SparseDnTensorHandle : GPU_SparseHandle<"SparseDnTensorHandleType", "dense tensor">;
```
- **EN**: Introduces declarations for `GPU_SparseDnTensorHandle`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SparseDnTensorHandle` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 169-175
```tablegen
def GPU_SparseSpGEMMOpHandle : GPU_SparseHandle<"SparseSpGEMMOpHandleType", "SpGEMM operation">;
def GPU_SparseSpMatHandle : GPU_SparseHandle<"SparseSpMatHandleType", "sparse matrix">;


//===----------------------------------------------------------------------===//
// GPU Interfaces.
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `GPU_SparseSpGEMMOpHandle`, `GPU_SparseSpMatHandle`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_SparseSpGEMMOpHandle`, `GPU_SparseSpMatHandle` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 176-183
```tablegen

def GPU_AsyncOpInterface : OpInterface<"AsyncOpInterface"> {
  let description = [{
    Interface for GPU operations that execute asynchronously on the device.

    GPU operations implementing this interface take a list of dependencies
    as `gpu.async.token` arguments and optionally return a `gpu.async.token`.

```
- **EN**: Introduces declarations for `GPU_AsyncOpInterface`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPU_AsyncOpInterface` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 184-193
```tablegen
    The op doesn't start executing until all depent ops producing the async
    dependency tokens have finished executing.

    If the op returns a token, the op merely schedules the execution on the
    device and returns immediately, without waiting for the execution to
    complete. On the hand, if the op does not return a token, the op will wait
    for the execution to complete.
  }];
  let cppNamespace = "::mlir::gpu";

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 194-205
```tablegen
  let methods = [
    InterfaceMethod<[{
        Query the operands that represent async dependency tokens.
      }],
      "OperandRange", "getAsyncDependencies", (ins), [{}], [{
        ConcreteOp op = cast<ConcreteOp>(this->getOperation());
        return op.getAsyncDependencies();
      }]
    >,
    InterfaceMethod<[{
        Adds a new token to the list of async dependencies if it is not already there.
      }],
```
- **EN**: Implements logic around `getOperation`, `getAsyncDependencies`.
- **CN**: 围绕 `getOperation`, `getAsyncDependencies` 实现具体逻辑。

### Lines 206-217
```tablegen
      "void", "addAsyncDependency", (ins "Value":$token),
      [{}], [{
        if (!::llvm::is_contained(this->getAsyncDependencies(), token))
          ::mlir::gpu::addAsyncDependency(this->getOperation(), token);
      }]
    >,
    InterfaceMethod<[{
        Query the result that represents the async token to depend on.
      }],
      "Value", "getAsyncToken"
    >
  ];
```
- **EN**: Implements logic around `is_contained`, `addAsyncDependency`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `is_contained`, `addAsyncDependency` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 218-223
```tablegen
}

//===----------------------------------------------------------------------===//
// GPU Attributes.
//===----------------------------------------------------------------------===//

```
- **EN**: Contains supporting declaration details for the surrounding MLIR include component.
- **CN**: 包含周边 MLIR 头文件组件所需的辅助声明细节。

### Lines 224-229
```tablegen
class GPU_Attr<string attrName, string attrMnemonic, list<Trait> traits = []>
    : AttrDef<GPU_Dialect, attrName, traits> {
  let mnemonic = attrMnemonic;
}

#endif // GPU_BASE
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
- **Declarative specifications / 声明式规格**:
  - **EN**: Uses TableGen records to express operations, constraints, interfaces, or canonicalization rules compactly.
  - **CN**: 使用 TableGen 记录紧凑表达操作、约束、接口或规范化规则。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **TableGen includes / TableGen 包含**: `mlir/IR/AttrTypeBase.td`, `mlir/IR/EnumAttr.td`, `mlir/IR/OpBase.td`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3)
