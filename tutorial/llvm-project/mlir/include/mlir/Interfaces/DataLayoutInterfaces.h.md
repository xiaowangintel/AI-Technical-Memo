# DataLayoutInterfaces.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Interfaces/DataLayoutInterfaces.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR DataLayoutInterfaces component. The leading comments describe it as: Defines the interfaces for the data layout specification, operations to which.
- **用途（CN）**: 声明 MLIR DataLayoutInterfaces 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
````cpp
//===- DataLayoutInterfaces.h - Data Layout Interface Decls -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Defines the interfaces for the data layout specification, operations to which
// they can be attached, types subject to data layout and dialects containing
// data layout entries.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_INTERFACES_DATALAYOUTINTERFACES_H
#define MLIR_INTERFACES_DATALAYOUTINTERFACES_H

#include "mlir/IR/Attributes.h"
#include "mlir/IR/DialectInterface.h"
#include "mlir/IR/OpDefinition.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/Support/TypeSize.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 25-74
````cpp
namespace mlir {
class DataLayout;
class DataLayoutEntryInterface;
class DLTIQueryInterface;
class TargetDeviceSpecInterface;
class TargetSystemSpecInterface;
using DataLayoutEntryKey = llvm::PointerUnion<Type, StringAttr>;
// Using explicit SmallVector size because we cannot infer the size from the
// forward declaration, and we need the typedef in the actual declaration.
using DataLayoutEntryList = llvm::SmallVector<DataLayoutEntryInterface, 4>;
using DataLayoutEntryListRef = llvm::ArrayRef<DataLayoutEntryInterface>;
using TargetDeviceSpecListRef = llvm::ArrayRef<TargetDeviceSpecInterface>;
using TargetDeviceSpecEntry = std::pair<StringAttr, TargetDeviceSpecInterface>;
using DataLayoutIdentifiedEntryMap =
    ::llvm::MapVector<::mlir::StringAttr, ::mlir::DataLayoutEntryInterface>;
class DataLayoutOpInterface;
class DataLayoutSpecInterface;
class ModuleOp;

namespace detail {
/// Default handler for the type size request. Computes results for built-in
/// types and dispatches to the DataLayoutTypeInterface for other types.
llvm::TypeSize getDefaultTypeSize(Type type, const DataLayout &dataLayout,
                                  DataLayoutEntryListRef params);

/// Default handler for the type size in bits request. Computes results for
/// built-in types and dispatches to the DataLayoutTypeInterface for other
/// types.
llvm::TypeSize getDefaultTypeSizeInBits(Type type, const DataLayout &dataLayout,
                                        DataLayoutEntryListRef params);

/// Default handler for the required alignment request. Computes results for
/// built-in types and dispatches to the DataLayoutTypeInterface for other
/// types.
uint64_t getDefaultABIAlignment(Type type, const DataLayout &dataLayout,
                                ArrayRef<DataLayoutEntryInterface> params);

/// Default handler for the preferred alignment request. Computes results for
/// built-in types and dispatches to the DataLayoutTypeInterface for other
/// types.
uint64_t
getDefaultPreferredAlignment(Type type, const DataLayout &dataLayout,
                             ArrayRef<DataLayoutEntryInterface> params);

/// Default handler for the index bitwidth request. Computes the result for
/// the built-in index type and dispatches to the DataLayoutTypeInterface for
/// other types.
std::optional<uint64_t>
getDefaultIndexBitwidth(Type type, const DataLayout &dataLayout,
                        ArrayRef<DataLayoutEntryInterface> params);
````
- **EN**: This C++ declaration introduces `DataLayout` and establishes part of the API surface for `DataLayoutInterfaces`. Representative entry points here include `getDefaultTypeSize`, `getDefaultTypeSizeInBits`, `getDefaultABIAlignment`, `getDefaultPreferredAlignment`.
- **CN**: 该 C++ 声明引入了 `DataLayout`，并构成 `DataLayoutInterfaces` API 表面的一部分。 这一段可见的代表性接口包括 `getDefaultTypeSize`, `getDefaultTypeSizeInBits`, `getDefaultABIAlignment`, `getDefaultPreferredAlignment`。

### Lines 75-119
````cpp
/// Default handler for endianness request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultEndianness(DataLayoutEntryInterface entry);

/// Default handler for the default memory space request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultMemorySpace(DataLayoutEntryInterface entry);

/// Default handler for alloca memory space request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultAllocaMemorySpace(DataLayoutEntryInterface entry);

/// Default handler for mangling mode request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultManglingMode(DataLayoutEntryInterface entry);

/// Default handler for program memory space request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultProgramMemorySpace(DataLayoutEntryInterface entry);

/// Default handler for global memory space request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultGlobalMemorySpace(DataLayoutEntryInterface entry);

/// Default handler for the stack alignment request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
uint64_t getDefaultStackAlignment(DataLayoutEntryInterface entry);

/// Default handler for the function pointer alignment request. Dispatches to
/// the DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultFunctionPointerAlignment(DataLayoutEntryInterface entry);

/// Default handler for the legal int widths request. Dispatches to the
/// DataLayoutInterface if specified, otherwise returns the default.
Attribute getDefaultLegalIntWidths(DataLayoutEntryInterface entry);

/// Returns the value of the property from the specified DataLayoutEntry. If the
/// property is missing from the entry, returns std::nullopt.
std::optional<Attribute> getDevicePropertyValue(DataLayoutEntryInterface entry);

/// Given a list of data layout entries, returns a new list containing the
/// entries with keys having the given type ID, i.e. belonging to the same type
/// class.
DataLayoutEntryList filterEntriesForType(DataLayoutEntryListRef entries,
                                         TypeID typeID);
````
- **EN**: This block groups callable interfaces such as `getDefaultEndianness`, `getDefaultMemorySpace`, `getDefaultAllocaMemorySpace`, `getDefaultManglingMode`, indicating how `DataLayoutInterfaces` is queried or updated.
- **CN**: 该代码块聚合了 `getDefaultEndianness`, `getDefaultMemorySpace`, `getDefaultAllocaMemorySpace`, `getDefaultManglingMode` 等可调用接口，展示了如何查询或更新 `DataLayoutInterfaces`。

### Lines 121-152
````cpp
/// Given a list of data layout entries, returns the entry that has the given
/// identifier as key, if such an entry exists in the list.
DataLayoutEntryInterface
filterEntryForIdentifier(DataLayoutEntryListRef entries, StringAttr id);

/// Given a list of target device entries, returns the entry that has the given
/// identifier as key, if such an entry exists in the list.
TargetDeviceSpecInterface
filterEntryForIdentifier(TargetDeviceSpecListRef entries, StringAttr id);

/// Verifies that the operation implementing the data layout interface, or a
/// module operation, is valid. This calls the verifier of the spec attribute
/// and checks if the layout is compatible with specs attached to the enclosing
/// operations.
LogicalResult verifyDataLayoutOp(Operation *op);

/// Verifies that a data layout spec is valid. This dispatches to individual
/// entry verifiers, and then to the verifiers implemented by the relevant type
/// and dialect interfaces for type and identifier keys respectively.
LogicalResult verifyDataLayoutSpec(DataLayoutSpecInterface spec, Location loc);

/// Verifies that a target system desc spec is valid. This dispatches to
/// individual entry verifiers, and then to the verifiers implemented by the
/// relevant dialect interfaces for identifier keys.
LogicalResult verifyTargetSystemSpec(TargetSystemSpecInterface spec,
                                     Location loc);

/// Divides the known min value of the numerator by the denominator and rounds
/// the result up to the next integer. Preserves the scalable flag.
llvm::TypeSize divideCeil(llvm::TypeSize numerator, uint64_t denominator);
} // namespace detail
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `filterEntryForIdentifier`, `verifyDataLayoutOp`, `verifyDataLayoutSpec`, `verifyTargetSystemSpec`, indicating how `DataLayoutInterfaces` is queried or updated.
- **CN**: 该代码块聚合了 `filterEntryForIdentifier`, `verifyDataLayoutOp`, `verifyDataLayoutSpec`, `verifyTargetSystemSpec` 等可调用接口，展示了如何查询或更新 `DataLayoutInterfaces`。

### Lines 155-158
````cpp
#include "mlir/Interfaces/DataLayoutAttrInterface.h.inc"
#include "mlir/Interfaces/DataLayoutDialectInterface.h.inc"
#include "mlir/Interfaces/DataLayoutOpInterface.h.inc"
#include "mlir/Interfaces/DataLayoutTypeInterface.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 160-160
````cpp
namespace mlir {
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `DataLayoutInterfaces`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `DataLayoutInterfaces` API 表面的一部分。

### Lines 162-208
````cpp
//===----------------------------------------------------------------------===//
// DataLayout
//===----------------------------------------------------------------------===//

/// The main mechanism for performing data layout queries. Instances of this
/// class can be created for an operation implementing DataLayoutOpInterface.
/// Upon construction, a layout spec combining that of the given operation with
/// all its ancestors will be computed and used to handle further requests. For
/// efficiency, results to all requests will be cached in this object.
/// Therefore, if the data layout spec for the scoping operation, or any of the
/// enclosing operations, changes, the cache is no longer valid. The user is
/// responsible creating a new DataLayout object after any spec change. In debug
/// mode, the cache validity is being checked in every request.
class DataLayout {
public:
  explicit DataLayout();
  explicit DataLayout(DataLayoutOpInterface op);
  explicit DataLayout(ModuleOp op);

  /// Returns the layout of the closest parent operation carrying layout info.
  static DataLayout closest(Operation *op);

  /// Returns the size of the given type in the current scope.
  llvm::TypeSize getTypeSize(Type t) const;

  /// Returns the size in bits of the given type in the current scope.
  llvm::TypeSize getTypeSizeInBits(Type t) const;

  /// Returns the required alignment of the given type in the current scope.
  uint64_t getTypeABIAlignment(Type t) const;

  /// Returns the preferred of the given type in the current scope.
  uint64_t getTypePreferredAlignment(Type t) const;

  /// Returns the bitwidth that should be used when performing index
  /// computations for the given pointer-like type in the current scope. If the
  /// type is not a pointer-like type, it returns std::nullopt.
  std::optional<uint64_t> getTypeIndexBitwidth(Type t) const;

  /// Returns the specified endianness.
  Attribute getEndianness() const;

  /// Returns the default memory space used for memory operations.
  Attribute getDefaultMemorySpace() const;

  /// Returns the memory space used for AllocaOps.
  Attribute getAllocaMemorySpace() const;
````
- **EN**: This C++ declaration introduces `DataLayout` and establishes part of the API surface for `DataLayoutInterfaces`. Representative entry points here include `DataLayout`, `closest`, `getTypeSize`, `getTypeSizeInBits`.
- **CN**: 该 C++ 声明引入了 `DataLayout`，并构成 `DataLayoutInterfaces` API 表面的一部分。 这一段可见的代表性接口包括 `DataLayout`, `closest`, `getTypeSize`, `getTypeSizeInBits`。

### Lines 209-253
````cpp
  /// Returns the mangling mode.
  Attribute getManglingMode() const;

  /// Returns the memory space used for program memory operations.
  Attribute getProgramMemorySpace() const;

  /// Returns the memory space used for global operations.
  Attribute getGlobalMemorySpace() const;

  /// Returns the natural alignment of the stack in bits. Alignment promotion of
  /// stack variables should be limited to the natural stack alignment to
  /// prevent dynamic stack alignment. Returns zero if the stack alignment is
  /// unspecified.
  uint64_t getStackAlignment() const;

  /// Returns function pointer alignment.
  Attribute getFunctionPointerAlignment() const;

  /// Returns the legal int widths.
  Attribute getLegalIntWidths() const;

  /// Returns the value of the specified property if the property is defined for
  /// the given device ID, otherwise returns std::nullopt.
  std::optional<Attribute>
  getDevicePropertyValue(TargetSystemSpecInterface::DeviceID,
                         StringAttr propertyName) const;

private:
  /// Combined layout spec at the given scope.
  const DataLayoutSpecInterface originalLayout;

  /// Combined target system desc spec at the given scope.
  const TargetSystemSpecInterface originalTargetSystemDesc;

#if LLVM_ENABLE_ABI_BREAKING_CHECKS
  /// List of enclosing layout specs.
  SmallVector<DataLayoutSpecInterface, 2> layoutStack;
#endif

  /// Asserts that the cache is still valid. Expensive in debug mode. No-op in
  /// release mode.
  void checkValid() const;

  /// Operation defining the scope of requests.
  Operation *scope;
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 255-280
````cpp
  /// Caches for individual requests.
  mutable DenseMap<Type, llvm::TypeSize> sizes;
  mutable DenseMap<Type, llvm::TypeSize> bitsizes;
  mutable DenseMap<Type, uint64_t> abiAlignments;
  mutable DenseMap<Type, uint64_t> preferredAlignments;
  mutable DenseMap<Type, std::optional<uint64_t>> indexBitwidths;

  /// Cache for the endianness.
  mutable std::optional<Attribute> endianness;
  /// Cache for the mangling mode.
  mutable std::optional<Attribute> manglingMode;
  /// Cache for default, alloca, global, and program memory spaces.
  mutable std::optional<Attribute> defaultMemorySpace;
  mutable std::optional<Attribute> allocaMemorySpace;
  mutable std::optional<Attribute> programMemorySpace;
  mutable std::optional<Attribute> globalMemorySpace;

  /// Cache for stack alignment.
  mutable std::optional<uint64_t> stackAlignment;
  /// Cache for function pointer alignment.
  mutable std::optional<Attribute> functionPointerAlignment;
  /// Cache for legal int widths.
  mutable std::optional<Attribute> legalIntWidths;
};

} // namespace mlir
````
- **EN**: This section focuses on caches for individual requests., grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“Caches for individual requests.”这一主题，把相关声明与辅助接口组织在一起。

### Lines 283-283
````cpp
#endif // MLIR_INTERFACES_DATALAYOUTINTERFACES_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/Attributes.h
- mlir/IR/DialectInterface.h
- mlir/IR/OpDefinition.h
- llvm/ADT/DenseMap.h
- llvm/ADT/MapVector.h
- llvm/Support/TypeSize.h
- mlir/Interfaces/DataLayoutAttrInterface.h.inc
- mlir/Interfaces/DataLayoutDialectInterface.h.inc
- mlir/Interfaces/DataLayoutOpInterface.h.inc
- mlir/Interfaces/DataLayoutTypeInterface.h.inc
