# TosaProfileCompliance.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TosaProfileCompliance.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR TosaProfileCompliance component. The leading comments describe it as: Type Compilance Definition.
- **用途（CN）**: 声明 MLIR TosaProfileCompliance 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
````cpp
//===- TosaProfileCompliance.h - Tosa Profile-based Compliance Validation -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TOSA_TRANSFORMS_TOSAPROFILECOMPILANCE_H
#define MLIR_DIALECT_TOSA_TRANSFORMS_TOSAPROFILECOMPILANCE_H

#include <unordered_map>

#include "mlir/Dialect/Tosa/IR/TargetEnv.h"
#include "mlir/Dialect/Tosa/Transforms/Passes.h"

#include "mlir/Support/TypeID.h"

using namespace mlir;
using namespace mlir::tosa;
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 22-29
````cpp
//===----------------------------------------------------------------------===//
// Type Compilance Definition
//===----------------------------------------------------------------------===//

typedef struct {
  mlir::TypeID typeID;
  uint32_t bitWidth;
} TypeInfo;
````
- **EN**: This section focuses on type compilance definition, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“Type Compilance Definition”这一主题，把相关声明与辅助接口组织在一起。

### Lines 31-40
````cpp
enum CheckCondition {
  invalid,
  // Valid when any of the profile (extension) requirement is meet.
  anyOf,
  // Valid when all of the profile (extension) requirement are meet.
  allOf
};

using VersionedTypeInfo =
    std::pair<SmallVector<TypeInfo>, SpecificationVersion>;
````
- **EN**: This C++ declaration introduces `CheckCondition` and establishes part of the API surface for `TosaProfileCompliance`.
- **CN**: 该 C++ 声明引入了 `CheckCondition`，并构成 `TosaProfileCompliance` API 表面的一部分。

### Lines 42-42
````cpp
template <typename T>
````
- **EN**: This section focuses on template <typename t>, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“template <typename T>”这一主题，把相关声明与辅助接口组织在一起。

### Lines 43-54
````cpp
struct OpComplianceInfo {
  // Certain operations require multiple modes enabled.
  // e.g. cast bf16 to fp8e4m3 requires EXT-BF16 and EXT-FP8E4M3.
  SmallVector<T> mode;
  SmallVector<VersionedTypeInfo> operandTypeInfoSet;
  CheckCondition condition = CheckCondition::anyOf;
};

using OperationProfileComplianceMap =
    std::unordered_map<std::string, SmallVector<OpComplianceInfo<Profile>>>;
using OperationExtensionComplianceMap =
    std::unordered_map<std::string, SmallVector<OpComplianceInfo<Extension>>>;
````
- **EN**: This C++ declaration introduces `OpComplianceInfo` and establishes part of the API surface for `TosaProfileCompliance`.
- **CN**: 该 C++ 声明引入了 `OpComplianceInfo`，并构成 `TosaProfileCompliance` API 表面的一部分。

### Lines 56-101
````cpp
//===----------------------------------------------------------------------===//
// Tosa Profile And Extension Information Depot
//===----------------------------------------------------------------------===//

class ProfileInfoDepot {
public:
  ProfileInfoDepot(Operation *op) {
    if (failed(populatationDispatch(op)))
      op->emitOpError() << "fail to populate the profile info\n";
  }

  void addType(Type t) { tyInfo.push_back(convertTypeToInfo(t)); }
  void addValue(Value v) { tyInfo.push_back(convertValueToInfo(v)); }
  SmallVector<TypeInfo> getInfo() { return tyInfo; }

private:
  TypeInfo convertTypeToInfo(Type type) {
    return {type.getTypeID(), tosa::getBitWidth(type)};
  }

  TypeInfo convertValueToInfo(Value value) {
    return convertTypeToInfo(getElementTypeOrSelf(value.getType()));
  }

  LogicalResult populatationDispatch(Operation *op);

  // Add input operands and output results to the profile type info list
  LogicalResult populateProfileInfo(ValueRange operands, ValueRange results);

  // Base
  template <typename T>
  LogicalResult populateProfileInfo(T op) {
    return op->emitOpError()
           << "profile requirement for this op has not been defined";
  }
  // For conv2d, conv3d, transpose_conv2d, and depthwise_conv2d.
  template <typename T>
  LogicalResult populateProfileInfoConv(T op);

  // For reshape, slice, tile, and transpose.
  template <typename T>
  LogicalResult populateProfileInfoDataLayout(T op);

private:
  SmallVector<TypeInfo> tyInfo;
};
````
- **EN**: This C++ declaration introduces `ProfileInfoDepot` and establishes part of the API surface for `TosaProfileCompliance`. Representative entry points here include `ProfileInfoDepot`, `failed`, `populatationDispatch`, `emitOpError`.
- **CN**: 该 C++ 声明引入了 `ProfileInfoDepot`，并构成 `TosaProfileCompliance` API 表面的一部分。 这一段可见的代表性接口包括 `ProfileInfoDepot`, `failed`, `populatationDispatch`, `emitOpError`。

### Lines 102-145
````cpp
//===----------------------------------------------------------------------===//
// Tosa Profile And Extension Compliance Checker
//===----------------------------------------------------------------------===//

class TosaProfileCompliance {
public:
  explicit TosaProfileCompliance();

  // Accessor of the compliance info map.
  template <typename T>
  std::unordered_map<std::string, SmallVector<OpComplianceInfo<T>>>
  getProfileComplianceMap() {
    // Only profile and extension compliance info are provided.
    return {};
  }

  // Verify if the operation is allowed to be executed in the given target
  // environment.
  LogicalResult checkProfile(Operation *op, const tosa::TargetEnv &targetEnv);
  LogicalResult checkExtension(Operation *op, const tosa::TargetEnv &targetEnv);
  LogicalResult checkInvalid(Operation *op);

  template <typename T>
  LogicalResult checkProfileOrExtension(
      Operation *op, const tosa::TargetEnv &targetEnv,
      const SmallVector<ArrayRef<T>> &specDefinedProfileSet);

  bool isSameTypeInfo(TypeInfo a, TypeInfo b) {
    return a.typeID == b.typeID && a.bitWidth == b.bitWidth;
  }

  // Find the required profiles or extensions from the compliance info according
  // to the operand type combination.
  template <typename T>
  SmallVector<OpComplianceInfo<T>>
  findMatchedEntries(Operation *op, SmallVector<OpComplianceInfo<T>> compInfo);

  // Debug utilites.
  template <typename T>
  SmallVector<StringRef> stringifyProfile(ArrayRef<T> profiles);

  template <typename T>
  SmallVector<StringRef>
  stringifyProfile(const SmallVector<ArrayRef<T>> &profileSet);
````
- **EN**: This C++ declaration introduces `TosaProfileCompliance` and establishes part of the API surface for `TosaProfileCompliance`. Representative entry points here include `TosaProfileCompliance`, `getProfileComplianceMap`, `checkProfile`, `checkExtension`.
- **CN**: 该 C++ 声明引入了 `TosaProfileCompliance`，并构成 `TosaProfileCompliance` API 表面的一部分。 这一段可见的代表性接口包括 `TosaProfileCompliance`, `getProfileComplianceMap`, `checkProfile`, `checkExtension`。

### Lines 147-156
````cpp
  static llvm::SmallString<7> stringifyTypeInfo(const TypeInfo &typeInfo);

private:
  template <typename T>
  FailureOr<SmallVector<OpComplianceInfo<T>>>
  getOperatorMatchedEntries(Operation *op);

  OperationProfileComplianceMap profileComplianceMap;
  OperationExtensionComplianceMap extensionComplianceMap;
};
````
- **EN**: This block groups callable interfaces such as `stringifyTypeInfo`, `getOperatorMatchedEntries`, indicating how `TosaProfileCompliance` is queried or updated.
- **CN**: 该代码块聚合了 `stringifyTypeInfo`, `getOperatorMatchedEntries` 等可调用接口，展示了如何查询或更新 `TosaProfileCompliance`。

### Lines 159-159
````cpp
#endif // MLIR_DIALECT_TOSA_TRANSFORMS_TOSAPROFILECOMPILANCE_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/Dialect/Tosa/IR/TargetEnv.h
- mlir/Dialect/Tosa/Transforms/Passes.h
- mlir/Support/TypeID.h
