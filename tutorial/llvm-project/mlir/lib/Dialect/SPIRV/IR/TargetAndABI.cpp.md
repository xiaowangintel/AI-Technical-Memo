# TargetAndABI.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/IR/TargetAndABI.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements SPIRV dialect support for dialect IR definitions such as operations, attributes, types, and parsers/printers, centered on `TargetAndABI`.
  - **CN**: 实现 SPIRV 方言中围绕 `TargetAndABI` 的方言 IR 定义，如操作、属性、类型以及解析/打印逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- TargetAndABI.cpp - SPIR-V target and ABI utilities -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-16
```cpp

#include "mlir/Dialect/SPIRV/IR/TargetAndABI.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVEnums.h"
#include "mlir/IR/Builders.h"
#include "mlir/IR/Operation.h"
#include "mlir/IR/SymbolTable.h"
#include "mlir/Interfaces/FunctionInterfaces.h"
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/IR/TargetAndABI.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/Builders.h`, `mlir/IR/Operation.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/IR/TargetAndABI.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/Builders.h`, `mlir/IR/Operation.h`。

### Lines 17-26
```cpp
using namespace mlir;

//===----------------------------------------------------------------------===//
// TargetEnv
//===----------------------------------------------------------------------===//

spirv::TargetEnv::TargetEnv(spirv::TargetEnvAttr targetAttr)
    : targetAttr(targetAttr) {
  givenExtensions.insert_range(targetAttr.getExtensions());

```
- **EN**: Introduces declarations for `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `mlir` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 27-33
```cpp
  // Add extensions implied by the current version.
  givenExtensions.insert_range(
      spirv::getImpliedExtensions(targetAttr.getVersion()));

  for (spirv::Capability cap : targetAttr.getCapabilities()) {
    givenCapabilities.insert(cap);

```
- **EN**: Implements logic around `insert_range`, `getImpliedExtensions`, `getCapabilities`, `insert`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `insert_range`, `getImpliedExtensions`, `getCapabilities`, `insert` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 34-42
```cpp
    // Add capabilities implied by the current capability.
    givenCapabilities.insert_range(spirv::getRecursiveImpliedCapabilities(cap));
  }
}

spirv::Version spirv::TargetEnv::getVersion() const {
  return targetAttr.getVersion();
}

```
- **EN**: Implements logic around `insert_range`, `getVersion`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `insert_range`, `getVersion` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 43-56
```cpp
bool spirv::TargetEnv::allows(spirv::Capability capability) const {
  return givenCapabilities.count(capability);
}

std::optional<spirv::Capability>
spirv::TargetEnv::allows(ArrayRef<spirv::Capability> caps) const {
  const auto *chosen = llvm::find_if(caps, [this](spirv::Capability cap) {
    return givenCapabilities.count(cap);
  });
  if (chosen != caps.end())
    return *chosen;
  return std::nullopt;
}

```
- **EN**: Implements logic around `allows`, `count`, `find_if`, `end`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `allows`, `count`, `find_if`, `end` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 57-70
```cpp
bool spirv::TargetEnv::allows(spirv::Extension extension) const {
  return givenExtensions.count(extension);
}

std::optional<spirv::Extension>
spirv::TargetEnv::allows(ArrayRef<spirv::Extension> exts) const {
  const auto *chosen = llvm::find_if(exts, [this](spirv::Extension ext) {
    return givenExtensions.count(ext);
  });
  if (chosen != exts.end())
    return *chosen;
  return std::nullopt;
}

```
- **EN**: Implements logic around `allows`, `count`, `find_if`, `end`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `allows`, `count`, `find_if`, `end` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 71-78
```cpp
spirv::Vendor spirv::TargetEnv::getVendorID() const {
  return targetAttr.getVendorID();
}

spirv::DeviceType spirv::TargetEnv::getDeviceType() const {
  return targetAttr.getDeviceType();
}

```
- **EN**: Implements logic around `getVendorID`, `getDeviceType`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getVendorID`, `getDeviceType` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 79-86
```cpp
uint32_t spirv::TargetEnv::getDeviceID() const {
  return targetAttr.getDeviceID();
}

spirv::ResourceLimitsAttr spirv::TargetEnv::getResourceLimits() const {
  return targetAttr.getResourceLimits();
}

```
- **EN**: Implements logic around `getDeviceID`, `getResourceLimits`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getDeviceID`, `getResourceLimits` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 87-93
```cpp
MLIRContext *spirv::TargetEnv::getContext() const {
  return targetAttr.getContext();
}

//===----------------------------------------------------------------------===//
// Utility functions
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getContext`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getContext` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 94-106
```cpp

StringRef spirv::getInterfaceVarABIAttrName() {
  return "spirv.interface_var_abi";
}

spirv::InterfaceVarABIAttr
spirv::getInterfaceVarABIAttr(unsigned descriptorSet, unsigned binding,
                              std::optional<spirv::StorageClass> storageClass,
                              MLIRContext *context) {
  return spirv::InterfaceVarABIAttr::get(descriptorSet, binding, storageClass,
                                         context);
}

```
- **EN**: Implements logic around `getInterfaceVarABIAttrName`, `getInterfaceVarABIAttr`, `get`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getInterfaceVarABIAttrName`, `getInterfaceVarABIAttr`, `get` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 107-116
```cpp
bool spirv::needsInterfaceVarABIAttrs(spirv::TargetEnvAttr targetAttr) {
  for (spirv::Capability cap : targetAttr.getCapabilities()) {
    if (cap == spirv::Capability::Kernel)
      return false;
    if (cap == spirv::Capability::Shader)
      return true;
  }
  return false;
}

```
- **EN**: Implements logic around `needsInterfaceVarABIAttrs`, `getCapabilities`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `needsInterfaceVarABIAttrs`, `getCapabilities` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 117-130
```cpp
StringRef spirv::getEntryPointABIAttrName() { return "spirv.entry_point_abi"; }

spirv::EntryPointABIAttr spirv::getEntryPointABIAttr(
    MLIRContext *context, ArrayRef<int32_t> workgroupSize,
    std::optional<int> subgroupSize, std::optional<int> targetWidth) {
  DenseI32ArrayAttr workgroupSizeAttr;
  if (!workgroupSize.empty()) {
    assert(workgroupSize.size() == 3);
    workgroupSizeAttr = DenseI32ArrayAttr::get(context, workgroupSize);
  }
  return spirv::EntryPointABIAttr::get(context, workgroupSizeAttr, subgroupSize,
                                       targetWidth);
}

```
- **EN**: Implements logic around `getEntryPointABIAttrName`, `getEntryPointABIAttr`, `empty`, `assert`, and 1 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getEntryPointABIAttrName`, `getEntryPointABIAttr`, `empty`, `assert`, and 1 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 131-140
```cpp
spirv::EntryPointABIAttr spirv::lookupEntryPointABI(Operation *op) {
  while (op && !isa<FunctionOpInterface>(op))
    op = op->getParentOp();
  if (!op)
    return {};

  if (auto attr = op->getAttrOfType<spirv::EntryPointABIAttr>(
          spirv::getEntryPointABIAttrName()))
    return attr;

```
- **EN**: Implements logic around `lookupEntryPointABI`, `getParentOp`, `EntryPointABIAttr>`, `getEntryPointABIAttrName`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookupEntryPointABI`, `getParentOp`, `EntryPointABIAttr>`, `getEntryPointABIAttrName` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 141-147
```cpp
  return {};
}

DenseI32ArrayAttr spirv::lookupLocalWorkGroupSize(Operation *op) {
  if (auto entryPoint = spirv::lookupEntryPointABI(op))
    return entryPoint.getWorkgroupSize();

```
- **EN**: Implements logic around `lookupLocalWorkGroupSize`, `lookupEntryPointABI`, `getWorkgroupSize`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookupLocalWorkGroupSize`, `lookupEntryPointABI`, `getWorkgroupSize` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 148-161
```cpp
  return {};
}

spirv::ResourceLimitsAttr
spirv::getDefaultResourceLimits(MLIRContext *context) {
  // All the fields have default values. Here we just provide a nicer way to
  // construct a default resource limit attribute.
  Builder b(context);
  return spirv::ResourceLimitsAttr::get(
      context,
      /*max_compute_shared_memory_size=*/16384,
      /*max_compute_workgroup_invocations=*/128,
      /*max_compute_workgroup_size=*/b.getI32ArrayAttr({128, 128, 64}),
      /*subgroup_size=*/32,
```
- **EN**: Implements logic around `getDefaultResourceLimits`, `b`, `get`, `getI32ArrayAttr`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getDefaultResourceLimits`, `b`, `get`, `getI32ArrayAttr` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 162-169
```cpp
      /*min_subgroup_size=*/std::nullopt,
      /*max_subgroup_size=*/std::nullopt,
      /*cooperative_matrix_properties_khr=*/ArrayAttr{},
      /*cooperative_matrix_properties_nv=*/ArrayAttr{});
}

StringRef spirv::getLoopControlAttrName() { return "spirv.loop_control"; }

```
- **EN**: Implements logic around `getLoopControlAttrName`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getLoopControlAttrName` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 170-183
```cpp
StringRef spirv::getSelectionControlAttrName() {
  return "spirv.selection_control";
}

StringRef spirv::getTargetEnvAttrName() { return "spirv.target_env"; }

spirv::TargetEnvAttr spirv::getDefaultTargetEnv(MLIRContext *context) {
  auto triple = spirv::VerCapExtAttr::get(spirv::Version::V_1_0,
                                          {spirv::Capability::Shader},
                                          ArrayRef<Extension>(), context);
  return spirv::TargetEnvAttr::get(
      triple, spirv::getDefaultResourceLimits(context),
      spirv::ClientAPI::Unknown, spirv::Vendor::Unknown,
      spirv::DeviceType::Unknown, spirv::TargetEnvAttr::kUnknownDeviceID);
```
- **EN**: Implements logic around `getSelectionControlAttrName`, `getTargetEnvAttrName`, `getDefaultTargetEnv`, `get`, and 2 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getSelectionControlAttrName`, `getTargetEnvAttrName`, `getDefaultTargetEnv`, `get`, and 2 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 184-191
```cpp
}

spirv::TargetEnvAttr spirv::lookupTargetEnv(Operation *op) {
  while (op) {
    op = SymbolTable::getNearestSymbolTable(op);
    if (!op)
      break;

```
- **EN**: Implements logic around `lookupTargetEnv`, `getNearestSymbolTable`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookupTargetEnv`, `getNearestSymbolTable` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 192-198
```cpp
    if (auto attr = op->getAttrOfType<spirv::TargetEnvAttr>(
            spirv::getTargetEnvAttrName()))
      return attr;

    op = op->getParentOp();
  }

```
- **EN**: Implements logic around `TargetEnvAttr>`, `getTargetEnvAttrName`, `getParentOp`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `TargetEnvAttr>`, `getTargetEnvAttrName`, `getParentOp` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 199-205
```cpp
  return {};
}

spirv::TargetEnvAttr spirv::lookupTargetEnvOrDefault(Operation *op) {
  if (spirv::TargetEnvAttr attr = spirv::lookupTargetEnv(op))
    return attr;

```
- **EN**: Implements logic around `lookupTargetEnvOrDefault`, `lookupTargetEnv`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `lookupTargetEnvOrDefault`, `lookupTargetEnv` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 206-219
```cpp
  return getDefaultTargetEnv(op->getContext());
}

spirv::AddressingModel
spirv::getAddressingModel(spirv::TargetEnvAttr targetAttr,
                          bool use64bitAddress) {
  for (spirv::Capability cap : targetAttr.getCapabilities()) {
    if (cap == Capability::Kernel)
      return use64bitAddress ? spirv::AddressingModel::Physical64
                             : spirv::AddressingModel::Physical32;
    // TODO PhysicalStorageBuffer64 is hard-coded here, but some information
    // should come from TargetEnvAttr to select between PhysicalStorageBuffer64
    // and PhysicalStorageBuffer64EXT
    if (cap == Capability::PhysicalStorageBufferAddresses)
```
- **EN**: Implements logic around `getDefaultTargetEnv`, `getAddressingModel`, `getCapabilities`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getDefaultTargetEnv`, `getAddressingModel`, `getCapabilities` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 220-233
```cpp
      return spirv::AddressingModel::PhysicalStorageBuffer64;
  }
  // Logical addressing doesn't need any capabilities so return it as default.
  return spirv::AddressingModel::Logical;
}

FailureOr<spirv::ExecutionModel>
spirv::getExecutionModel(spirv::TargetEnvAttr targetAttr) {
  for (spirv::Capability cap : targetAttr.getCapabilities()) {
    if (cap == spirv::Capability::Kernel)
      return spirv::ExecutionModel::Kernel;
    if (cap == spirv::Capability::Shader)
      return spirv::ExecutionModel::GLCompute;
  }
```
- **EN**: Implements logic around `getExecutionModel`, `getCapabilities`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `getExecutionModel`, `getCapabilities` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 234-246
```cpp
  return failure();
}

FailureOr<spirv::MemoryModel>
spirv::getMemoryModel(spirv::TargetEnvAttr targetAttr) {
  for (spirv::Capability cap : targetAttr.getCapabilities()) {
    if (cap == spirv::Capability::Kernel)
      return spirv::MemoryModel::OpenCL;
    if (cap == spirv::Capability::Shader)
      return spirv::MemoryModel::GLSL450;
  }
  return failure();
}
```
- **EN**: Implements logic around `failure`, `getMemoryModel`, `getCapabilities`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `failure`, `getMemoryModel`, `getCapabilities` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SPIRV/IR/TargetAndABI.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/Builders.h`, `mlir/IR/Operation.h`, `mlir/IR/SymbolTable.h`, `mlir/Interfaces/FunctionInterfaces.h`
- **Standard-library headers / 标准库头文件**: `<optional>`
- **Subsystem categories / 子系统类别**: MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (3), dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (2), cross-dialect interfaces and capability contracts / 跨方言接口与能力约定 (1)
