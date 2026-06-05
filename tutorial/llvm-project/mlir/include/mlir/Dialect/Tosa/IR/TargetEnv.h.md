# TargetEnv.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Tosa/IR/TargetEnv.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR TargetEnv component. The leading comments describe it as: This file declares utilities for Tosa target environment (implementation).
- **用途（CN）**: 声明 MLIR TargetEnv 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- TargetEnv.h - Tosa target environment utilities ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares utilities for Tosa target environment (implementation).
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TOSA_IR_TARGETENV_H
#define MLIR_DIALECT_TOSA_IR_TARGETENV_H

#include "mlir/Dialect/Tosa/IR/TosaOps.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/SmallSet.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-65
````cpp
namespace mlir {
namespace tosa {

struct TosaLevel {
  int32_t MAX_RANK = 0;
  int32_t MAX_KERNEL = 0;
  int32_t MAX_STRIDE = 0;
  int32_t MAX_SCALE = 0;
  int32_t MAX_LOG2_SIZE = 0;
  int32_t MAX_NESTING = 0;
  int32_t MAX_TENSOR_LIST_SIZE = 0;
  int32_t MAX_SHAPE_LEN = 0;

  bool operator==(const TosaLevel &rhs) {
    return MAX_RANK == rhs.MAX_RANK && MAX_KERNEL == rhs.MAX_KERNEL &&
           MAX_STRIDE == rhs.MAX_STRIDE && MAX_SCALE == rhs.MAX_SCALE &&
           MAX_LOG2_SIZE == rhs.MAX_LOG2_SIZE &&
           MAX_NESTING == rhs.MAX_NESTING &&
           MAX_TENSOR_LIST_SIZE == rhs.MAX_TENSOR_LIST_SIZE &&
           MAX_SHAPE_LEN == rhs.MAX_SHAPE_LEN;
  }
};

static constexpr TosaLevel TOSA_LEVEL_EIGHTK = {6,  8192, 8192, 256,
                                                31, 6,    64,   16};
static constexpr TosaLevel TOSA_LEVEL_NONE = {32, 2147483647, 2147483647, 2048,
                                              63, 256,        256,        64};

TargetEnvAttr lookupTargetEnv(Operation *op);
TargetEnvAttr getDefaultTargetEnv(MLIRContext *context);

/// Queries the target environment recursively from enclosing symbol table ops
/// containing the given `op` or returns the default target environment as
/// returned by getDefaultTargetEnv() if not provided.
TargetEnvAttr lookupTargetEnvOrDefault(Operation *op);

/// A thin wrapper around the SpecificationVersion enum to represent
/// and provide utilities around the TOSA specification version.
class TosaSpecificationVersion {
public:
  TosaSpecificationVersion() = default;

  TosaSpecificationVersion(uint32_t major, uint32_t minor, bool draft = false)
      : majorVersion(major), minorVersion(minor), draft(draft) {}
  TosaSpecificationVersion(SpecificationVersion version)
      : TosaSpecificationVersion(fromVersionEnum(version)) {}
````
- **EN**: This C++ declaration introduces `TosaLevel` and establishes part of the API surface for `TargetEnv`. Representative entry points here include `lookupTargetEnv`, `getDefaultTargetEnv`, `lookupTargetEnvOrDefault`, `TosaSpecificationVersion`.
- **CN**: 该 C++ 声明引入了 `TosaLevel`，并构成 `TargetEnv` API 表面的一部分。 这一段可见的代表性接口包括 `lookupTargetEnv`, `getDefaultTargetEnv`, `lookupTargetEnvOrDefault`, `TosaSpecificationVersion`。

### Lines 66-109
````cpp
  bool isBackwardsCompatibleWith(TosaSpecificationVersion baseVersion) const {
    if (this->majorVersion != baseVersion.majorVersion)
      return false;
    if (this->minorVersion < baseVersion.minorVersion)
      return false;
    // An unreleased version is not expected to be backwards compatible with
    // a corresponding released version. However, an unreleased version is
    // expected to be backwards compatible with all released versions prior to
    // it.
    //
    // For example:
    // - 1.1.draft is not expected to be backwards compatible with 1.1
    // - 1.1.draft is expected to be backwards compatible with 1.0
    // - 1.1.draft is not expected to be backwards compatible with 1.0.draft
    if (this->draft && !baseVersion.draft &&
        this->minorVersion == baseVersion.minorVersion)
      return false;
    return true;
  }

  uint32_t getMajor() const { return majorVersion; }
  uint32_t getMinor() const { return minorVersion; }
  bool isDraft() const { return draft; }

private:
  uint32_t majorVersion = 0;
  uint32_t minorVersion = 0;
  bool draft = false;

  static TosaSpecificationVersion
  fromVersionEnum(SpecificationVersion version) {
    switch (version) {
    case SpecificationVersion::V_1_0:
      return TosaSpecificationVersion(1, 0);
    case SpecificationVersion::V_1_1_DRAFT:
      return TosaSpecificationVersion(1, 1, true);
    }
    llvm_unreachable("Unknown TOSA version");
  }
};

TosaSpecificationVersion getMinVersion(const Profile &profile);
TosaSpecificationVersion getMinVersion(const Extension &extension);
TosaSpecificationVersion getMinVersion(const Level &level);
````
- **EN**: This block groups callable interfaces such as `isBackwardsCompatibleWith`, `getMajor`, `getMinor`, `isDraft`, indicating how `TargetEnv` is queried or updated.
- **CN**: 该代码块聚合了 `isBackwardsCompatibleWith`, `getMajor`, `getMinor`, `isDraft` 等可调用接口，展示了如何查询或更新 `TargetEnv`。

### Lines 111-154
````cpp
llvm::SmallString<4> stringifyVersion(TosaSpecificationVersion version);

/// This class represents the capability enabled in the target implementation
/// such as profile, extension, and level. It's a wrapper class around
/// tosa::TargetEnvAttr.
class TargetEnv {
public:
  TargetEnv() {}

  static FailureOr<TargetEnv>
  createTargetEnvFromAttr(TargetEnvAttr targetAttr, Location targetEnvAttrLoc);

  static LogicalResult verifyTargetInformation(TargetEnvAttr targetAttr,
                                               Location targetAttrLoc);

  void addProfile(Profile p) { enabledProfiles.insert(p); }
  void addExtension(Extension e) { enabledExtensions.insert(e); }

  TosaSpecificationVersion getSpecVersion() const {
    return specificationVersion;
  }

  TosaLevel getLevel() const {
    if (level == Level::eightK)
      return TOSA_LEVEL_EIGHTK;
    else if (level == Level::none)
      return TOSA_LEVEL_NONE;
    else
      llvm_unreachable("Unknown TOSA level");
  };

  // Returns true if the given profile is allowed.
  bool allows(Profile prof) const { return enabledProfiles.count(prof) != 0; }

  bool allowsAnyOf(ArrayRef<Profile> profs) const {
    return llvm::any_of(profs, [&](Profile prof) { return allows(prof); });
  }

  bool allowsAllOf(ArrayRef<Profile> profs) const {
    return llvm::all_of(profs, [&](Profile prof) { return allows(prof); });
  }

  // Returns true if the given extension is allowed.
  bool allows(Extension ext) const { return enabledExtensions.count(ext) != 0; }
````
- **EN**: This C++ declaration introduces `TargetEnv` and establishes part of the API surface for `TargetEnv`. Representative entry points here include `stringifyVersion`, `TargetEnv`, `createTargetEnvFromAttr`, `verifyTargetInformation`.
- **CN**: 该 C++ 声明引入了 `TargetEnv`，并构成 `TargetEnv` API 表面的一部分。 这一段可见的代表性接口包括 `stringifyVersion`, `TargetEnv`, `createTargetEnvFromAttr`, `verifyTargetInformation`。

### Lines 156-182
````cpp
  bool allowsAnyOf(ArrayRef<Extension> exts) const {
    return llvm::any_of(exts, [&](Extension ext) { return allows(ext); });
  }

  bool allowsAllOf(ArrayRef<Extension> exts) const {
    return llvm::all_of(exts, [&](Extension ext) { return allows(ext); });
  }

private:
  // Require target information is verified before constructing, via the use of
  // `createTargetEnvFromAttr`.
  explicit TargetEnv(SpecificationVersion specificationVersion, Level level,
                     const ArrayRef<Profile> &profiles,
                     const ArrayRef<Extension> &extensions)
      : specificationVersion(specificationVersion), level(level) {
    enabledProfiles.insert_range(profiles);
    enabledExtensions.insert_range(extensions);
  }

  TosaSpecificationVersion specificationVersion;
  Level level;
  llvm::SmallSet<Profile, 3> enabledProfiles;
  llvm::SmallSet<Extension, 13> enabledExtensions;
};

} // namespace tosa
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `allowsAnyOf`, `any_of`, `allows`, `allowsAllOf`, indicating how `TargetEnv` is queried or updated.
- **CN**: 该代码块聚合了 `allowsAnyOf`, `any_of`, `allows`, `allowsAllOf` 等可调用接口，展示了如何查询或更新 `TargetEnv`。

### Lines 185-185
````cpp
#endif // MLIR_DIALECT_TOSA_IR_TARGETENV_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: IR construction and mutation helpers
  **CN**: IR 构造与变更辅助接口

## Dependencies / 依赖关系

- mlir/Dialect/Tosa/IR/TosaOps.h
- mlir/Support/LLVM.h
- llvm/ADT/SmallSet.h
