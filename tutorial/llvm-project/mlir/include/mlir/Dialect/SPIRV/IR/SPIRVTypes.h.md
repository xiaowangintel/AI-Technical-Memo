# SPIRVTypes.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/SPIRV/IR/SPIRVTypes.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR SPIRVTypes component. The leading comments describe it as: This file declares the types in the SPIR-V dialect.
- **用途（CN）**: 声明 MLIR SPIRVTypes 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
````cpp
//===- SPIRVTypes.h - MLIR SPIR-V Types -------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the types in the SPIR-V dialect.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_SPIRV_IR_SPIRVTYPES_H_
#define MLIR_DIALECT_SPIRV_IR_SPIRVTYPES_H_

#include "mlir/Dialect/SPIRV/IR/SPIRVEnums.h"
#include "mlir/IR/BuiltinTypeInterfaces.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/TypeSupport.h"
#include "mlir/IR/Types.h"

#include <cstdint>
#include <tuple>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 27-73
````cpp
namespace mlir {
namespace spirv {

namespace detail {
struct ArrayTypeStorage;
struct CooperativeMatrixTypeStorage;
struct TensorArmTypeStorage;
struct ImageTypeStorage;
struct MatrixTypeStorage;
struct PointerTypeStorage;
struct RuntimeArrayTypeStorage;
struct SampledImageTypeStorage;
struct StructTypeStorage;

} // namespace detail

// Base SPIR-V type for providing availability queries.
class SPIRVType : public Type {
public:
  using Type::Type;

  static bool classof(Type type);

  bool isScalarOrVector();

  /// The extension requirements for each type are following the
  /// ((Extension::A OR Extension::B) AND (Extension::C OR Extension::D))
  /// convention.
  using ExtensionArrayRefVector = SmallVectorImpl<ArrayRef<Extension>>;

  /// Appends to `extensions` the extensions needed for this type to appear in
  /// the given `storage` class. This method does not guarantee the uniqueness
  /// of extensions; the same extension may be appended multiple times.
  void getExtensions(ExtensionArrayRefVector &extensions,
                     std::optional<StorageClass> storage = std::nullopt);

  /// The capability requirements for each type are following the
  /// ((Capability::A OR Extension::B) AND (Capability::C OR Capability::D))
  /// convention.
  using CapabilityArrayRefVector = SmallVectorImpl<ArrayRef<Capability>>;

  /// Appends to `capabilities` the capabilities needed for this type to appear
  /// in the given `storage` class. This method does not guarantee the
  /// uniqueness of capabilities; the same capability may be appended multiple
  /// times.
  void getCapabilities(CapabilityArrayRefVector &capabilities,
                       std::optional<StorageClass> storage = std::nullopt);
````
- **EN**: This C++ declaration introduces `ArrayTypeStorage` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `classof`, `isScalarOrVector`, `getExtensions`, `getCapabilities`.
- **CN**: 该 C++ 声明引入了 `ArrayTypeStorage`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `classof`, `isScalarOrVector`, `getExtensions`, `getCapabilities`。

### Lines 74-117
````cpp
  /// Returns the size in bytes for each type. If no size can be calculated,
  /// returns `std::nullopt`. Note that if the type has explicit layout, it is
  /// also taken into account in calculation.
  std::optional<int64_t> getSizeInBytes();
};

// SPIR-V scalar type: bool type, integer type, floating point type.
class ScalarType : public SPIRVType {
public:
  using SPIRVType::SPIRVType;

  static bool classof(Type type);

  /// Returns true if the given float type is valid for the SPIR-V dialect.
  static bool isValid(FloatType);
  /// Returns true if the given integer type is valid for the SPIR-V dialect.
  static bool isValid(IntegerType);
};

// SPIR-V composite type: VectorType, SPIR-V ArrayType, SPIR-V
// StructType, or SPIR-V TensorArmType.
class CompositeType : public SPIRVType {
public:
  using SPIRVType::SPIRVType;

  static bool classof(Type type);

  /// Returns true if the given vector type is valid for the SPIR-V dialect.
  static bool isValid(VectorType);

  /// Return the number of elements of the type. This should only be called if
  /// hasCompileTimeKnownNumElements is true.
  unsigned getNumElements() const;

  Type getElementType(unsigned) const;

  /// Return true if the number of elements is known at compile time and is not
  /// implementation dependent.
  bool hasCompileTimeKnownNumElements() const;
};

// SPIR-V array type
class ArrayType : public Type::TypeBase<ArrayType, CompositeType,
                                        detail::ArrayTypeStorage> {
````
- **EN**: This C++ declaration introduces `ScalarType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `getSizeInBytes`, `classof`, `isValid`, `getNumElements`.
- **CN**: 该 C++ 声明引入了 `ScalarType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `getSizeInBytes`, `classof`, `isValid`, `getNumElements`。

### Lines 119-163
````cpp
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.array";

  static ArrayType get(Type elementType, unsigned elementCount);

  /// Returns an array type with the given stride in bytes.
  static ArrayType get(Type elementType, unsigned elementCount,
                       unsigned stride);

  unsigned getNumElements() const;

  Type getElementType() const;

  /// Returns the array stride in bytes. 0 means no stride decorated on this
  /// type.
  unsigned getArrayStride() const;
};

// SPIR-V image type
class ImageType
    : public Type::TypeBase<ImageType, SPIRVType, detail::ImageTypeStorage> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.image";

  static ImageType
  get(Type elementType, Dim dim,
      ImageDepthInfo depth = ImageDepthInfo::DepthUnknown,
      ImageArrayedInfo arrayed = ImageArrayedInfo::NonArrayed,
      ImageSamplingInfo samplingInfo = ImageSamplingInfo::SingleSampled,
      ImageSamplerUseInfo samplerUse = ImageSamplerUseInfo::SamplerUnknown,
      ImageFormat format = ImageFormat::Unknown) {
    return ImageType::get(
        std::tuple<Type, Dim, ImageDepthInfo, ImageArrayedInfo,
                   ImageSamplingInfo, ImageSamplerUseInfo, ImageFormat>(
            elementType, dim, depth, arrayed, samplingInfo, samplerUse,
            format));
  }

  static ImageType
      get(std::tuple<Type, Dim, ImageDepthInfo, ImageArrayedInfo,
                     ImageSamplingInfo, ImageSamplerUseInfo, ImageFormat>);
````
- **EN**: This C++ declaration introduces `ImageType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `get`, `getNumElements`, `getElementType`, `getArrayStride`.
- **CN**: 该 C++ 声明引入了 `ImageType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `get`, `getNumElements`, `getElementType`, `getArrayStride`。

### Lines 164-209
````cpp
  Type getElementType() const;
  Dim getDim() const;
  ImageDepthInfo getDepthInfo() const;
  ImageArrayedInfo getArrayedInfo() const;
  ImageSamplingInfo getSamplingInfo() const;
  ImageSamplerUseInfo getSamplerUseInfo() const;
  ImageFormat getImageFormat() const;
  // TODO: Add support for Access qualifier
};

// SPIR-V pointer type
class PointerType
    : public Type::TypeBase<PointerType, SPIRVType, detail::PointerTypeStorage,
                            VectorElementTypeInterface::Trait> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.pointer";

  static PointerType get(Type pointeeType, StorageClass storageClass);

  Type getPointeeType() const;

  StorageClass getStorageClass() const;
};

// SPIR-V run-time array type
class RuntimeArrayType
    : public Type::TypeBase<RuntimeArrayType, SPIRVType,
                            detail::RuntimeArrayTypeStorage> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.rtarray";

  static RuntimeArrayType get(Type elementType);

  /// Returns a runtime array type with the given stride in bytes.
  static RuntimeArrayType get(Type elementType, unsigned stride);

  Type getElementType() const;

  /// Returns the array stride in bytes. 0 means no stride decorated on this
  /// type.
  unsigned getArrayStride() const;
};
````
- **EN**: This C++ declaration introduces `PointerType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `getElementType`, `getDim`, `getDepthInfo`, `getArrayedInfo`.
- **CN**: 该 C++ 声明引入了 `PointerType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `getElementType`, `getDim`, `getDepthInfo`, `getArrayedInfo`。

### Lines 211-273
````cpp
// SPIR-V sampled image type
class SampledImageType
    : public Type::TypeBase<SampledImageType, SPIRVType,
                            detail::SampledImageTypeStorage> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.sampled_image";

  static SampledImageType get(Type imageType);

  static SampledImageType
  getChecked(function_ref<InFlightDiagnostic()> emitError, Type imageType);

  static LogicalResult
  verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                   Type imageType);

  Type getImageType() const;
};

// SPIR-V sampler type
class SamplerType : public Type::TypeBase<SamplerType, SPIRVType, TypeStorage> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.sampler";

  static SamplerType get(MLIRContext *context);
};

// SPIR-V named barrier type (OpTypeNamedBarrier)
class NamedBarrierType
    : public Type::TypeBase<NamedBarrierType, SPIRVType, TypeStorage> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.named_barrier";

  static NamedBarrierType get(MLIRContext *context);
};

/// SPIR-V struct type. Two kinds of struct types are supported:
/// - Literal: a literal struct type is uniqued by its fields (types + offset
/// info + decoration info).
/// - Identified: an indentified struct type is uniqued by its string identifier
/// (name). This is useful in representing recursive structs. For example, the
/// following C struct:
///
/// struct A {
///   A* next;
/// };
///
/// would be represented in MLIR as:
///
/// !spirv.struct<A, (!spirv.ptr<!spirv.struct<A>, Generic>)>
///
/// In the above, expressing recursive struct types is accomplished by giving a
/// recursive struct a unique identified and using that identifier in the struct
/// definition for recursive references.
class StructType
    : public Type::TypeBase<StructType, CompositeType,
                            detail::StructTypeStorage, TypeTrait::IsMutable> {
````
- **EN**: This C++ declaration introduces `SampledImageType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `get`, `getChecked`, `InFlightDiagnostic`, `verifyInvariants`.
- **CN**: 该 C++ 声明引入了 `SampledImageType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `get`, `getChecked`, `InFlightDiagnostic`, `verifyInvariants`。

### Lines 275-324
````cpp
public:
  using Base::Base;

  // Type for specifying the offset of the struct members
  using OffsetInfo = uint32_t;

  static constexpr StringLiteral name = "spirv.struct";

  // Type for specifying the decoration(s) on struct members.
  // If `decorationValue` is UnitAttr then decoration has no
  // value.
  struct MemberDecorationInfo {
    uint32_t memberIndex;
    Decoration decoration;
    Attribute decorationValue;

    MemberDecorationInfo(uint32_t index, Decoration decoration,
                         Attribute decorationValue)
        : memberIndex(index), decoration(decoration),
          decorationValue(decorationValue) {}

    friend bool operator==(const MemberDecorationInfo &lhs,
                           const MemberDecorationInfo &rhs) {
      return lhs.memberIndex == rhs.memberIndex &&
             lhs.decoration == rhs.decoration &&
             lhs.decorationValue == rhs.decorationValue;
    }

    friend bool operator<(const MemberDecorationInfo &lhs,
                          const MemberDecorationInfo &rhs) {
      return std::tuple(lhs.memberIndex, llvm::to_underlying(lhs.decoration)) <
             std::tuple(rhs.memberIndex, llvm::to_underlying(rhs.decoration));
    }

    bool hasValue() const { return !isa<UnitAttr>(decorationValue); }
  };

  // Type for specifying the decoration(s) on the struct itself.
  struct StructDecorationInfo {
    Decoration decoration;
    Attribute decorationValue;

    StructDecorationInfo(Decoration decoration, Attribute decorationValue)
        : decoration(decoration), decorationValue(decorationValue) {}

    friend bool operator==(const StructDecorationInfo &lhs,
                           const StructDecorationInfo &rhs) {
      return lhs.decoration == rhs.decoration &&
             lhs.decorationValue == rhs.decorationValue;
    }
````
- **EN**: This C++ declaration introduces `MemberDecorationInfo` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `MemberDecorationInfo`, `memberIndex`, `decoration`, `decorationValue`.
- **CN**: 该 C++ 声明引入了 `MemberDecorationInfo`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `MemberDecorationInfo`, `memberIndex`, `decoration`, `decorationValue`。

### Lines 325-368
````cpp
    friend bool operator<(const StructDecorationInfo &lhs,
                          const StructDecorationInfo &rhs) {
      return llvm::to_underlying(lhs.decoration) <
             llvm::to_underlying(rhs.decoration);
    }

    bool hasValue() const { return !isa<UnitAttr>(decorationValue); }
  };

  /// Construct a literal StructType with at least one member.
  static StructType get(ArrayRef<Type> memberTypes,
                        ArrayRef<OffsetInfo> offsetInfo = {},
                        ArrayRef<MemberDecorationInfo> memberDecorations = {},
                        ArrayRef<StructDecorationInfo> structDecorations = {});

  /// Construct an identified StructType. This creates a StructType whose body
  /// (member types, offset info, and decorations) is not set yet. A call to
  /// StructType::trySetBody(...) must follow when the StructType contents are
  /// available (e.g. parsed or deserialized).
  ///
  /// Note: If another thread creates (or had already created) a struct with the
  /// same identifier, that struct will be returned as a result.
  static StructType getIdentified(MLIRContext *context, StringRef identifier);

  /// Construct a (possibly identified) StructType with no members.
  ///
  /// Note: this method might fail in a multi-threaded setup if another thread
  /// created an identified struct with the same identifier but with different
  /// contents before returning. In which case, an empty (default-constructed)
  /// StructType is returned.
  static StructType getEmpty(MLIRContext *context, StringRef identifier = "");

  /// For literal structs, return an empty string.
  /// For identified structs, return the struct's identifier.
  StringRef getIdentifier() const;

  /// Returns true if the StructType is identified.
  bool isIdentified() const;

  unsigned getNumElements() const;

  Type getElementType(unsigned) const;

  TypeRange getElementTypes() const;
````
- **EN**: This block groups callable interfaces such as `to_underlying`, `hasValue`, `get`, `getIdentified`, indicating how `SPIRVTypes` is queried or updated.
- **CN**: 该代码块聚合了 `to_underlying`, `hasValue`, `get`, `getIdentified` 等可调用接口，展示了如何查询或更新 `SPIRVTypes`。

### Lines 370-414
````cpp
  bool hasOffset() const;

  /// Returns true if the struct has a specified decoration.
  bool hasDecoration(spirv::Decoration decoration) const;

  uint64_t getMemberOffset(unsigned) const;

  // Returns in `memberDecorations` the Decorations (apart from Offset)
  // associated with all members of the StructType.
  void getMemberDecorations(SmallVectorImpl<StructType::MemberDecorationInfo>
                                &memberDecorations) const;

  // Returns in `decorationsInfo` all the Decorations (apart from Offset)
  // associated with the `i`-th member of the StructType.
  void getMemberDecorations(
      unsigned i,
      SmallVectorImpl<StructType::MemberDecorationInfo> &decorationsInfo) const;

  // Returns in `structDecorations` the Decorations associated with the
  // StructType.
  void getStructDecorations(SmallVectorImpl<StructType::StructDecorationInfo>
                                &structDecorations) const;

  /// Sets the contents of an incomplete identified StructType. This method must
  /// be called only for identified StructTypes and it must be called only once
  /// per instance. Otherwise, failure() is returned.
  LogicalResult
  trySetBody(ArrayRef<Type> memberTypes, ArrayRef<OffsetInfo> offsetInfo = {},
             ArrayRef<MemberDecorationInfo> memberDecorations = {},
             ArrayRef<StructDecorationInfo> structDecorations = {});
};

llvm::hash_code
hash_value(const StructType::MemberDecorationInfo &memberDecorationInfo);

llvm::hash_code
hash_value(const StructType::StructDecorationInfo &structDecorationInfo);

// SPIR-V KHR cooperative matrix type
class CooperativeMatrixType
    : public Type::TypeBase<CooperativeMatrixType, CompositeType,
                            detail::CooperativeMatrixTypeStorage,
                            ShapedType::Trait> {
public:
  using Base::Base;
````
- **EN**: This C++ declaration introduces `CooperativeMatrixType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `hasOffset`, `hasDecoration`, `getMemberOffset`, `getMemberDecorations`.
- **CN**: 该 C++ 声明引入了 `CooperativeMatrixType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `hasOffset`, `hasDecoration`, `getMemberOffset`, `getMemberDecorations`。

### Lines 416-461
````cpp
  static constexpr StringLiteral name = "spirv.coopmatrix";

  static CooperativeMatrixType get(Type elementType, uint32_t rows,
                                   uint32_t columns, Scope scope,
                                   CooperativeMatrixUseKHR use);
  Type getElementType() const;

  /// Returns the scope of the matrix.
  Scope getScope() const;
  /// Returns the number of rows of the matrix.
  uint32_t getRows() const;
  /// Returns the number of columns of the matrix.
  uint32_t getColumns() const;
  /// Returns the use parameter of the cooperative matrix.
  CooperativeMatrixUseKHR getUse() const;

  operator ShapedType() const { return cast<ShapedType>(*this); }

  ArrayRef<int64_t> getShape() const;

  bool hasRank() const { return true; }

  CooperativeMatrixType cloneWith(std::optional<ArrayRef<int64_t>> shape,
                                  Type elementType) const {
    if (!shape)
      return get(elementType, getRows(), getColumns(), getScope(), getUse());

    assert(shape.value().size() == 2);
    return get(elementType, shape.value()[0], shape.value()[1], getScope(),
               getUse());
  }
};

// SPIR-V matrix type
class MatrixType
    : public Type::TypeBase<MatrixType, CompositeType,
                            detail::MatrixTypeStorage, ShapedType::Trait> {
public:
  using Base::Base;

  static constexpr StringLiteral name = "spirv.matrix";

  static MatrixType get(Type columnType, uint32_t columnCount);

  static MatrixType getChecked(function_ref<InFlightDiagnostic()> emitError,
                               Type columnType, uint32_t columnCount);
````
- **EN**: This C++ declaration introduces `MatrixType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `get`, `getElementType`, `getScope`, `getRows`.
- **CN**: 该 C++ 声明引入了 `MatrixType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `get`, `getElementType`, `getScope`, `getRows`。

### Lines 463-508
````cpp
  static LogicalResult
  verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                   Type columnType, uint32_t columnCount);

  /// Returns true if the matrix elements are vectors of float elements.
  static bool isValidColumnType(Type columnType);

  Type getColumnType() const;

  /// Returns the number of rows.
  unsigned getNumRows() const;

  /// Returns the number of columns.
  unsigned getNumColumns() const;

  /// Returns total number of elements (rows*columns).
  unsigned getNumElements() const;

  /// Returns the elements' type (i.e, single element type).
  Type getElementType() const;

  operator ShapedType() const { return cast<ShapedType>(*this); }

  ArrayRef<int64_t> getShape() const;

  bool hasRank() const { return true; }

  MatrixType cloneWith(std::optional<ArrayRef<int64_t>> shape,
                       Type elementType) const {
    if (!shape)
      return get(elementType, getNumColumns());

    assert(shape.value().size() == 2);

    auto vectorType = cast<VectorType>(elementType);
    Type newElementType =
        vectorType.cloneWith({shape.value()[0]}, vectorType.getElementType());

    return get(newElementType, shape.value()[1]);
  }
};

/// SPIR-V TensorARM Type
class TensorArmType
    : public Type::TypeBase<TensorArmType, CompositeType,
                            detail::TensorArmTypeStorage, ShapedType::Trait> {
````
- **EN**: This C++ declaration introduces `TensorArmType` and establishes part of the API surface for `SPIRVTypes`. Representative entry points here include `verifyInvariants`, `InFlightDiagnostic`, `isValidColumnType`, `getColumnType`.
- **CN**: 该 C++ 声明引入了 `TensorArmType`，并构成 `SPIRVTypes` API 表面的一部分。 这一段可见的代表性接口包括 `verifyInvariants`, `InFlightDiagnostic`, `isValidColumnType`, `getColumnType`。

### Lines 510-542
````cpp
public:
  using Base::Base;

  using ShapedTypeTraits = ShapedType::Trait<TensorArmType>;
  using ShapedTypeTraits::getDimSize;
  using ShapedTypeTraits::getDynamicDimIndex;
  using ShapedTypeTraits::getElementTypeBitWidth;
  using ShapedTypeTraits::getNumDynamicDims;
  using ShapedTypeTraits::getNumElements;
  using ShapedTypeTraits::getRank;
  using ShapedTypeTraits::hasStaticShape;
  using ShapedTypeTraits::isDynamicDim;

  static constexpr StringLiteral name = "spirv.arm.tensor";

  // TensorArm supports minimum rank of 1, hence an empty shape here means
  // unranked.
  static TensorArmType get(ArrayRef<int64_t> shape, Type elementType);
  TensorArmType cloneWith(std::optional<ArrayRef<int64_t>> shape,
                          Type elementType) const;

  static LogicalResult
  verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                   ArrayRef<int64_t> shape, Type elementType);

  Type getElementType() const;
  ArrayRef<int64_t> getShape() const;
  bool hasRank() const { return !getShape().empty(); }
  operator ShapedType() const { return cast<ShapedType>(*this); }
};

} // namespace spirv
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `get`, `cloneWith`, `verifyInvariants`, `InFlightDiagnostic`, indicating how `SPIRVTypes` is queried or updated.
- **CN**: 该代码块聚合了 `get`, `cloneWith`, `verifyInvariants`, `InFlightDiagnostic` 等可调用接口，展示了如何查询或更新 `SPIRVTypes`。

### Lines 544-544
````cpp
#endif // MLIR_DIALECT_SPIRV_IR_SPIRVTYPES_H_
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

- mlir/Dialect/SPIRV/IR/SPIRVEnums.h
- mlir/IR/BuiltinTypeInterfaces.h
- mlir/IR/BuiltinTypes.h
- mlir/IR/Diagnostics.h
- mlir/IR/Location.h
- mlir/IR/TypeSupport.h
- mlir/IR/Types.h
- SPIRVType inherits from public Type
- ScalarType inherits from public SPIRVType
- CompositeType inherits from public SPIRVType
- ArrayType inherits from public Type::TypeBase<ArrayType, CompositeType, detail::ArrayTypeStorage>
- ImageType inherits from public Type::TypeBase<ImageType, SPIRVType, detail::ImageTypeStorage>
- PointerType inherits from public Type::TypeBase<PointerType, SPIRVType, detail::PointerTypeStorage, VectorElementTypeInterface::Trait>
- RuntimeArrayType inherits from public Type::TypeBase<RuntimeArrayType, SPIRVType, detail::RuntimeArrayTypeStorage>
- SampledImageType inherits from public Type::TypeBase<SampledImageType, SPIRVType, detail::SampledImageTypeStorage>
- SamplerType inherits from public Type::TypeBase<SamplerType, SPIRVType, TypeStorage>
- NamedBarrierType inherits from public Type::TypeBase<NamedBarrierType, SPIRVType, TypeStorage>
- StructType inherits from public Type::TypeBase<StructType, CompositeType, detail::StructTypeStorage, TypeTrait::IsMutable>
- CooperativeMatrixType inherits from public Type::TypeBase<CooperativeMatrixType, CompositeType, detail::CooperativeMatrixTypeStorage, ShapedType::Trait>
- MatrixType inherits from public Type::TypeBase<MatrixType, CompositeType, detail::MatrixTypeStorage, ShapedType::Trait>
