# TargetAndABI.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/TargetAndABI.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR TargetAndABI component. The leading comments describe it as: This file declares utilities for SPIR-V target and shader interface ABI.
- **用途（CN）**: 声明 MLIR TargetAndABI 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
````cpp
//===- TargetAndABI.h - SPIR-V target and ABI utilities  --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares utilities for SPIR-V target and shader interface ABI.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_TARGETANDABI_H
#define MLIR_DIALECT_SPIRV_IR_TARGETANDABI_H

#include "mlir/Dialect/SPIRV/IR/SPIRVAttributes.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallSet.h"
#include <optional>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 21-65
````cpp
namespace mlir {
class Operation;

namespace spirv {
enum class StorageClass : uint32_t;

/// A wrapper class around a spirv::TargetEnvAttr to provide query methods for
/// allowed version/capabilities/extensions.
class TargetEnv {
public:
  explicit TargetEnv(TargetEnvAttr targetAttr);

  Version getVersion() const;

  /// Returns true if the given capability is allowed.
  bool allows(Capability) const;
  /// Returns the first allowed one if any of the given capabilities is allowed.
  /// Returns std::nullopt otherwise.
  std::optional<Capability> allows(ArrayRef<Capability>) const;

  /// Returns true if the given extension is allowed.
  bool allows(Extension) const;
  /// Returns the first allowed one if any of the given extensions is allowed.
  /// Returns std::nullopt otherwise.
  std::optional<Extension> allows(ArrayRef<Extension>) const;

  /// Returns the vendor ID.
  Vendor getVendorID() const;

  /// Returns the device type.
  DeviceType getDeviceType() const;

  /// Returns the device ID.
  uint32_t getDeviceID() const;

  /// Returns the MLIRContext.
  MLIRContext *getContext() const;

  /// Returns the target resource limits.
  ResourceLimitsAttr getResourceLimits() const;

  TargetEnvAttr getAttr() const { return targetAttr; }

  /// Allows implicity converting to the underlying spirv::TargetEnvAttr.
  operator TargetEnvAttr() const { return targetAttr; }
````
- **EN**: This C++ declaration introduces `Operation` and establishes part of the API surface for `TargetAndABI`. Representative entry points here include `TargetEnv`, `getVersion`, `allows`, `getVendorID`.
- **CN**: 该 C++ 声明引入了 `Operation`，并构成 `TargetAndABI` API 表面的一部分。 这一段可见的代表性接口包括 `TargetEnv`, `getVersion`, `allows`, `getVendorID`。

### Lines 66-109
````cpp
private:
  TargetEnvAttr targetAttr;
  llvm::SmallSet<Extension, 4> givenExtensions;    /// Allowed extensions
  llvm::SmallSet<Capability, 8> givenCapabilities; /// Allowed capabilities
};

/// Returns the attribute name for specifying argument ABI information.
StringRef getInterfaceVarABIAttrName();

/// Gets the InterfaceVarABIAttr given its fields.
InterfaceVarABIAttr
getInterfaceVarABIAttr(unsigned descriptorSet, unsigned binding,
                       std::optional<StorageClass> storageClass,
                       MLIRContext *context);

/// Returns whether the given SPIR-V target (described by TargetEnvAttr) needs
/// ABI attributes for interface variables (spirv.interface_var_abi).
bool needsInterfaceVarABIAttrs(TargetEnvAttr targetAttr);

/// Returns the attribute name for specifying entry point information.
StringRef getEntryPointABIAttrName();

/// Gets the EntryPointABIAttr given its fields.
/// targetWidth is used by several execution modes. It is the element width
/// of floating-point operations.
/// Refer to Execution Mode in SPIR-V specification.
/// https://registry.khronos.org/SPIR-V/specs/unified1/SPIRV.html#_execution_mode
EntryPointABIAttr getEntryPointABIAttr(MLIRContext *context,
                                       ArrayRef<int32_t> workgroupSize = {},
                                       std::optional<int> subgroupSize = {},
                                       std::optional<int> targetWidth = {});

/// Queries the entry point ABI on the nearest function-like op containing the
/// given `op`. Returns null attribute if not found.
EntryPointABIAttr lookupEntryPointABI(Operation *op);

/// Queries the local workgroup size from entry point ABI on the nearest
/// function-like op containing the given `op`. Returns null attribute if not
/// found.
DenseI32ArrayAttr lookupLocalWorkGroupSize(Operation *op);

/// Returns a default resource limits attribute that uses numbers from
/// "Table 46. Required Limits" of the Vulkan spec.
ResourceLimitsAttr getDefaultResourceLimits(MLIRContext *context);
````
- **EN**: This block groups callable interfaces such as `getInterfaceVarABIAttrName`, `getInterfaceVarABIAttr`, `needsInterfaceVarABIAttrs`, `getEntryPointABIAttrName`, indicating how `TargetAndABI` is queried or updated.
- **CN**: 该代码块聚合了 `getInterfaceVarABIAttrName`, `getInterfaceVarABIAttr`, `needsInterfaceVarABIAttrs`, `getEntryPointABIAttrName` 等可调用接口，展示了如何查询或更新 `TargetAndABI`。

### Lines 111-146
````cpp
/// Returns the attribute name for specifying loop control.
StringRef getLoopControlAttrName();

/// Returns the attribute name for specifying selection control.
StringRef getSelectionControlAttrName();

/// Returns the attribute name for specifying SPIR-V target environment.
StringRef getTargetEnvAttrName();

/// Returns the default target environment: SPIR-V 1.0 with Shader capability
/// and no extra extensions.
TargetEnvAttr getDefaultTargetEnv(MLIRContext *context);

/// Queries the target environment recursively from enclosing symbol table ops
/// containing the given `op`.
TargetEnvAttr lookupTargetEnv(Operation *op);

/// Queries the target environment recursively from enclosing symbol table ops
/// containing the given `op` or returns the default target environment as
/// returned by getDefaultTargetEnv() if not provided.
TargetEnvAttr lookupTargetEnvOrDefault(Operation *op);

/// Returns addressing model selected based on target environment.
AddressingModel getAddressingModel(TargetEnvAttr targetAttr,
                                   bool use64bitAddress);

/// Returns execution model selected based on target environment.
/// Returns failure if it cannot be selected.
FailureOr<ExecutionModel> getExecutionModel(TargetEnvAttr targetAttr);

/// Returns memory model selected based on target environment.
/// Returns failure if it cannot be selected.
FailureOr<MemoryModel> getMemoryModel(TargetEnvAttr targetAttr);

} // namespace spirv
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `getLoopControlAttrName`, `getSelectionControlAttrName`, `getTargetEnvAttrName`, `getDefaultTargetEnv`, indicating how `TargetAndABI` is queried or updated.
- **CN**: 该代码块聚合了 `getLoopControlAttrName`, `getSelectionControlAttrName`, `getTargetEnvAttrName`, `getDefaultTargetEnv` 等可调用接口，展示了如何查询或更新 `TargetAndABI`。

### Lines 149-149
````cpp
#endif // MLIR_DIALECT_SPIRV_IR_TARGETANDABI_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/SPIRV/IR/SPIRVAttributes.h
- mlir/Support/LLVM.h
- llvm/ADT/SmallSet.h
