# SPIRVTypes.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SPIRV/IR/SPIRVTypes.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file defines the types in the SPIR-V dialect.
  - **CN**: 该文件位于 `mlir/lib/Dialect/SPIRV/IR`，围绕 SPIRV 方言实现相关能力。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```cpp
//===- SPIRVTypes.cpp - MLIR SPIR-V Types ---------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the types in the SPIR-V dialect.
//
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/SPIRV/IR/SPIRVTypes.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVDialect.h"
#include "mlir/Dialect/SPIRV/IR/SPIRVEnums.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/ErrorHandling.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/BuiltinTypes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/BuiltinTypes.h`。

### Lines 22-41
```cpp
#include <cstdint>
#include <optional>

using namespace mlir;
using namespace mlir::spirv;

namespace {
// Helper function to collect extensions implied by a type by visiting all its
// subtypes. Maintains a set of `seen` types to avoid recursion in structs.
//
// Serves as the source-of-truth for type extension information. All extension
// logic should be added to this class, while the
// `SPIRVType::getExtensions` function should not handle extension-related logic
// directly and only invoke `TypeExtensionVisitor::add(Type *)`.
class TypeExtensionVisitor {
public:
  TypeExtensionVisitor(SPIRVType::ExtensionArrayRefVector &extensions,
                       std::optional<StorageClass> storage)
      : extensions(extensions), storage(storage) {}

```
- **EN**: Pulls in the headers needed by this translation unit, including `cstdint`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `cstdint`, `optional`。

### Lines 42-64
```cpp
  // Main visitor entry point. Adds all extensions to the vector. Saves `type`
  // as seen and dispatches to the right concrete `.add` function.
  void add(SPIRVType type) {
    if (auto [_it, inserted] = seen.insert({type, storage}); !inserted)
      return;

    TypeSwitch<SPIRVType>(type)
        .Case<CooperativeMatrixType, ImageType, PointerType, ScalarType,
              TensorArmType>(
            [this](auto concreteType) { addConcrete(concreteType); })
        .Case<ArrayType, MatrixType, RuntimeArrayType, VectorType>(
            [this](auto concreteType) { add(concreteType.getElementType()); })
        .Case([this](SampledImageType concreteType) {
          add(concreteType.getImageType());
        })
        .Case([this](StructType concreteType) {
          for (Type elementType : concreteType.getElementTypes())
            add(elementType);
        })
        .Case<SamplerType, NamedBarrierType>([](auto) { /* no extensions */ })
        .DefaultUnreachable("Unhandled type");
  }

```
- **EN**: Implements logic around `add`, `insert`, `TypeSwitch`, `TensorArmType>`, and 6 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `add`, `insert`, `TypeSwitch`, `TensorArmType>`, and 6 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 65-85
```cpp
  void add(Type type) { add(cast<SPIRVType>(type)); }

private:
  // Types that add unique extensions.
  void addConcrete(CooperativeMatrixType type);
  void addConcrete(ImageType type);
  void addConcrete(PointerType type);
  void addConcrete(ScalarType type);
  void addConcrete(TensorArmType type);

  template <Extension... Es>
  void pushExts() {
    static constexpr Extension exts[] = {Es...};
    extensions.push_back(exts);
  }

  SPIRVType::ExtensionArrayRefVector &extensions;
  std::optional<StorageClass> storage;
  llvm::SmallDenseSet<std::pair<Type, std::optional<StorageClass>>> seen;
};

```
- **EN**: Implements logic around `add`, `addConcrete`, `pushExts`, `push_back`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `add`, `addConcrete`, `pushExts`, `push_back` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 86-104
```cpp
// Helper function to collect capabilities implied by a type by visiting all its
// subtypes. Maintains a set of `seen` types to avoid recursion in structs.
//
// Serves as the source-of-truth for type capability information. All capability
// logic should be added to this class, while the
// `SPIRVType::getCapabilities` function should not handle capability-related
// logic directly and only invoke `TypeCapabilityVisitor::add(Type *)`.
class TypeCapabilityVisitor {
public:
  TypeCapabilityVisitor(SPIRVType::CapabilityArrayRefVector &capabilities,
                        std::optional<StorageClass> storage)
      : capabilities(capabilities), storage(storage) {}

  // Main visitor entry point. Adds all extensions to the vector. Saves `type`
  // as seen and dispatches to the right concrete `.add` function.
  void add(SPIRVType type) {
    if (auto [_it, inserted] = seen.insert({type, storage}); !inserted)
      return;

```
- **EN**: Introduces declarations for `TypeCapabilityVisitor`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `TypeCapabilityVisitor` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 105-124
```cpp
    TypeSwitch<SPIRVType>(type)
        .Case<CooperativeMatrixType, ImageType, MatrixType, PointerType,
              RuntimeArrayType, ScalarType, TensorArmType, VectorType>(
            [this](auto concreteType) { addConcrete(concreteType); })
        .Case([this](ArrayType concreteType) {
          add(concreteType.getElementType());
        })
        .Case([this](SampledImageType concreteType) {
          add(concreteType.getImageType());
        })
        .Case([this](StructType concreteType) {
          for (Type elementType : concreteType.getElementTypes())
            add(elementType);
        })
        .Case([](SamplerType) { /* no capabilities */ })
        .Case(
            [this](NamedBarrierType) { pushCaps<Capability::NamedBarrier>(); })
        .DefaultUnreachable("Unhandled type");
  }

```
- **EN**: Implements logic around `TypeSwitch`, `VectorType>`, `addConcrete`, `Case`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `TypeSwitch`, `VectorType>`, `addConcrete`, `Case`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 125-143
```cpp
  void add(Type type) { add(cast<SPIRVType>(type)); }

private:
  // Types that add unique extensions.
  void addConcrete(CooperativeMatrixType type);
  void addConcrete(ImageType type);
  void addConcrete(MatrixType type);
  void addConcrete(PointerType type);
  void addConcrete(RuntimeArrayType type);
  void addConcrete(ScalarType type);
  void addConcrete(TensorArmType type);
  void addConcrete(VectorType type);

  template <Capability... Cs>
  void pushCaps() {
    static constexpr Capability caps[] = {Cs...};
    capabilities.push_back(caps);
  }

```
- **EN**: Implements logic around `add`, `addConcrete`, `pushCaps`, `push_back`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `add`, `addConcrete`, `pushCaps`, `push_back` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 144-162
```cpp
  SPIRVType::CapabilityArrayRefVector &capabilities;
  std::optional<StorageClass> storage;
  llvm::SmallDenseSet<std::pair<Type, std::optional<StorageClass>>> seen;
};

} // namespace

//===----------------------------------------------------------------------===//
// ArrayType
//===----------------------------------------------------------------------===//

struct spirv::detail::ArrayTypeStorage : public TypeStorage {
  using KeyTy = std::tuple<Type, unsigned, unsigned>;

  static ArrayTypeStorage *construct(TypeStorageAllocator &allocator,
                                     const KeyTy &key) {
    return new (allocator.allocate<ArrayTypeStorage>()) ArrayTypeStorage(key);
  }

```
- **EN**: Introduces declarations for `spirv::detail::ArrayTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::ArrayTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 163-181
```cpp
  bool operator==(const KeyTy &key) const {
    return key == KeyTy(elementType, elementCount, stride);
  }

  ArrayTypeStorage(const KeyTy &key)
      : elementType(std::get<0>(key)), elementCount(std::get<1>(key)),
        stride(std::get<2>(key)) {}

  Type elementType;
  unsigned elementCount;
  unsigned stride;
};

ArrayType ArrayType::get(Type elementType, unsigned elementCount) {
  assert(elementCount && "ArrayType needs at least one element");
  return Base::get(elementType.getContext(), elementType, elementCount,
                   /*stride=*/0);
}

```
- **EN**: Implements logic around `KeyTy`, `ArrayTypeStorage`, `elementType`, `stride`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `KeyTy`, `ArrayTypeStorage`, `elementType`, `stride`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 182-205
```cpp
ArrayType ArrayType::get(Type elementType, unsigned elementCount,
                         unsigned stride) {
  assert(elementCount && "ArrayType needs at least one element");
  return Base::get(elementType.getContext(), elementType, elementCount, stride);
}

unsigned ArrayType::getNumElements() const { return getImpl()->elementCount; }

Type ArrayType::getElementType() const { return getImpl()->elementType; }

unsigned ArrayType::getArrayStride() const { return getImpl()->stride; }

//===----------------------------------------------------------------------===//
// CompositeType
//===----------------------------------------------------------------------===//

bool CompositeType::classof(Type type) {
  if (auto vectorType = dyn_cast<VectorType>(type))
    return isValid(vectorType);
  return isa<spirv::ArrayType, spirv::CooperativeMatrixType, spirv::MatrixType,
             spirv::RuntimeArrayType, spirv::StructType, spirv::TensorArmType>(
      type);
}

```
- **EN**: Implements logic around `get`, `assert`, `getNumElements`, `getElementType`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `get`, `assert`, `getNumElements`, `getElementType`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 206-229
```cpp
bool CompositeType::isValid(VectorType type) {
  return type.getRank() == 1 &&
         llvm::is_contained({2, 3, 4, 8, 16}, type.getNumElements()) &&
         (isa<ScalarType>(type.getElementType()) ||
          isa<PointerType>(type.getElementType()));
}

Type CompositeType::getElementType(unsigned index) const {
  return TypeSwitch<Type, Type>(*this)
      .Case<ArrayType, CooperativeMatrixType, RuntimeArrayType, VectorType,
            TensorArmType>([](auto type) { return type.getElementType(); })
      .Case([](MatrixType type) { return type.getColumnType(); })
      .Case([index](StructType type) { return type.getElementType(index); })
      .DefaultUnreachable("Invalid composite type");
}

unsigned CompositeType::getNumElements() const {
  return TypeSwitch<SPIRVType, unsigned>(*this)
      .Case<ArrayType, StructType, TensorArmType, VectorType>(
          [](auto type) { return type.getNumElements(); })
      .Case([](MatrixType type) { return type.getNumColumns(); })
      .DefaultUnreachable("Invalid type for number of elements query");
}

```
- **EN**: Implements logic around `isValid`, `getRank`, `is_contained`, `getElementType`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isValid`, `getRank`, `is_contained`, `getElementType`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 230-262
```cpp
bool CompositeType::hasCompileTimeKnownNumElements() const {
  return !isa<CooperativeMatrixType, RuntimeArrayType>(*this);
}

void TypeCapabilityVisitor::addConcrete(VectorType type) {
  add(type.getElementType());

  int64_t vecSize = type.getNumElements();
  if (vecSize == 8 || vecSize == 16)
    pushCaps<Capability::Vector16>();
}

//===----------------------------------------------------------------------===//
// CooperativeMatrixType
//===----------------------------------------------------------------------===//

struct spirv::detail::CooperativeMatrixTypeStorage final : TypeStorage {
  // In the specification dimensions of the Cooperative Matrix are 32-bit
  // integers --- the initial implementation kept those values as such. However,
  // the `ShapedType` expects the shape to be `int64_t`. We could keep the shape
  // as 32-bits and expose it as int64_t through `getShape`, however, this
  // method returns an `ArrayRef`, so returning `ArrayRef<int64_t>` having two
  // 32-bits integers would require an extra logic and storage. So, we diverge
  // from the spec and internally represent the dimensions as 64-bit integers,
  // so we can easily return an `ArrayRef` from `getShape` without any extra
  // logic. Alternatively, we could store both rows and columns (both 32-bits)
  // and shape (64-bits), assigning rows and columns to shape whenever
  // `getShape` is called. This would be at the cost of extra logic and storage.
  // Note: Because `ArrayRef` is returned we cannot construct an object in
  // `getShape` on the fly.
  using KeyTy =
      std::tuple<Type, int64_t, int64_t, Scope, CooperativeMatrixUseKHR>;

```
- **EN**: Introduces declarations for `spirv::detail::CooperativeMatrixTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::CooperativeMatrixTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 263-284
```cpp
  static CooperativeMatrixTypeStorage *
  construct(TypeStorageAllocator &allocator, const KeyTy &key) {
    return new (allocator.allocate<CooperativeMatrixTypeStorage>())
        CooperativeMatrixTypeStorage(key);
  }

  bool operator==(const KeyTy &key) const {
    return key == KeyTy(elementType, shape[0], shape[1], scope, use);
  }

  CooperativeMatrixTypeStorage(const KeyTy &key)
      : elementType(std::get<0>(key)),
        shape({std::get<1>(key), std::get<2>(key)}), scope(std::get<3>(key)),
        use(std::get<4>(key)) {}

  Type elementType;
  // [#rows, #columns]
  std::array<int64_t, 2> shape;
  Scope scope;
  CooperativeMatrixUseKHR use;
};

```
- **EN**: Implements logic around `construct`, `new`, `CooperativeMatrixTypeStorage`, `KeyTy`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `construct`, `new`, `CooperativeMatrixTypeStorage`, `KeyTy`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 285-306
```cpp
CooperativeMatrixType CooperativeMatrixType::get(Type elementType,
                                                 uint32_t rows,
                                                 uint32_t columns, Scope scope,
                                                 CooperativeMatrixUseKHR use) {
  return Base::get(elementType.getContext(), elementType, rows, columns, scope,
                   use);
}

Type CooperativeMatrixType::getElementType() const {
  return getImpl()->elementType;
}

uint32_t CooperativeMatrixType::getRows() const {
  assert(getImpl()->shape[0] != ShapedType::kDynamic);
  return static_cast<uint32_t>(getImpl()->shape[0]);
}

uint32_t CooperativeMatrixType::getColumns() const {
  assert(getImpl()->shape[1] != ShapedType::kDynamic);
  return static_cast<uint32_t>(getImpl()->shape[1]);
}

```
- **EN**: Implements logic around `get`, `getElementType`, `getImpl`, `getRows`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `get`, `getElementType`, `getImpl`, `getRows`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 307-331
```cpp
ArrayRef<int64_t> CooperativeMatrixType::getShape() const {
  return getImpl()->shape;
}

Scope CooperativeMatrixType::getScope() const { return getImpl()->scope; }

CooperativeMatrixUseKHR CooperativeMatrixType::getUse() const {
  return getImpl()->use;
}

void TypeExtensionVisitor::addConcrete(CooperativeMatrixType type) {
  add(type.getElementType());
  pushExts<Extension::SPV_KHR_cooperative_matrix>();
}

void TypeCapabilityVisitor::addConcrete(CooperativeMatrixType type) {
  Type elementType = type.getElementType();
  add(elementType);
  pushCaps<Capability::CooperativeMatrixKHR>();
  if (elementType.isBF16())
    pushCaps<Capability::BFloat16CooperativeMatrixKHR>();
  if (elementType.isF8E4M3FN() || elementType.isF8E5M2())
    pushCaps<Capability::Float8CooperativeMatrixEXT>();
}

```
- **EN**: Implements logic around `getShape`, `getImpl`, `getScope`, `getUse`, and 9 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getShape`, `getImpl`, `getScope`, `getUse`, and 9 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 332-367
```cpp
//===----------------------------------------------------------------------===//
// ImageType
//===----------------------------------------------------------------------===//

template <typename T>
static constexpr unsigned getNumBits() {
  return 0;
}
template <>
constexpr unsigned getNumBits<Dim>() {
  static_assert((1 << 3) > getMaxEnumValForDim(),
                "Not enough bits to encode Dim value");
  return 3;
}
template <>
constexpr unsigned getNumBits<ImageDepthInfo>() {
  static_assert((1 << 2) > getMaxEnumValForImageDepthInfo(),
                "Not enough bits to encode ImageDepthInfo value");
  return 2;
}
template <>
constexpr unsigned getNumBits<ImageArrayedInfo>() {
  static_assert((1 << 1) > getMaxEnumValForImageArrayedInfo(),
                "Not enough bits to encode ImageArrayedInfo value");
  return 1;
}
template <>
constexpr unsigned getNumBits<ImageSamplingInfo>() {
  static_assert((1 << 1) > getMaxEnumValForImageSamplingInfo(),
                "Not enough bits to encode ImageSamplingInfo value");
  return 1;
}
template <>
constexpr unsigned getNumBits<ImageSamplerUseInfo>() {
  static_assert((1 << 2) > getMaxEnumValForImageSamplerUseInfo(),
                "Not enough bits to encode ImageSamplerUseInfo value");
```
- **EN**: Implements logic around `getNumBits`, `static_assert`.
- **CN**: 围绕 `getNumBits`, `static_assert` 实现具体逻辑。

### Lines 368-386
```cpp
  return 2;
}
template <>
constexpr unsigned getNumBits<ImageFormat>() {
  static_assert((1 << 6) > getMaxEnumValForImageFormat(),
                "Not enough bits to encode ImageFormat value");
  return 6;
}

struct spirv::detail::ImageTypeStorage : public TypeStorage {
public:
  using KeyTy = std::tuple<Type, Dim, ImageDepthInfo, ImageArrayedInfo,
                           ImageSamplingInfo, ImageSamplerUseInfo, ImageFormat>;

  static ImageTypeStorage *construct(TypeStorageAllocator &allocator,
                                     const KeyTy &key) {
    return new (allocator.allocate<ImageTypeStorage>()) ImageTypeStorage(key);
  }

```
- **EN**: Introduces declarations for `spirv::detail::ImageTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::ImageTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 387-406
```cpp
  bool operator==(const KeyTy &key) const {
    return key == KeyTy(elementType, dim, depthInfo, arrayedInfo, samplingInfo,
                        samplerUseInfo, format);
  }

  ImageTypeStorage(const KeyTy &key)
      : elementType(std::get<0>(key)), dim(std::get<1>(key)),
        depthInfo(std::get<2>(key)), arrayedInfo(std::get<3>(key)),
        samplingInfo(std::get<4>(key)), samplerUseInfo(std::get<5>(key)),
        format(std::get<6>(key)) {}

  Type elementType;
  Dim dim : getNumBits<Dim>();
  ImageDepthInfo depthInfo : getNumBits<ImageDepthInfo>();
  ImageArrayedInfo arrayedInfo : getNumBits<ImageArrayedInfo>();
  ImageSamplingInfo samplingInfo : getNumBits<ImageSamplingInfo>();
  ImageSamplerUseInfo samplerUseInfo : getNumBits<ImageSamplerUseInfo>();
  ImageFormat format : getNumBits<ImageFormat>();
};

```
- **EN**: Implements logic around `KeyTy`, `ImageTypeStorage`, `elementType`, `depthInfo`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `KeyTy`, `ImageTypeStorage`, `elementType`, `depthInfo`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 407-427
```cpp
ImageType
ImageType::get(std::tuple<Type, Dim, ImageDepthInfo, ImageArrayedInfo,
                          ImageSamplingInfo, ImageSamplerUseInfo, ImageFormat>
                   value) {
  return Base::get(std::get<0>(value).getContext(), value);
}

Type ImageType::getElementType() const { return getImpl()->elementType; }

Dim ImageType::getDim() const { return getImpl()->dim; }

ImageDepthInfo ImageType::getDepthInfo() const { return getImpl()->depthInfo; }

ImageArrayedInfo ImageType::getArrayedInfo() const {
  return getImpl()->arrayedInfo;
}

ImageSamplingInfo ImageType::getSamplingInfo() const {
  return getImpl()->samplingInfo;
}

```
- **EN**: Implements logic around `get`, `getElementType`, `getDim`, `getDepthInfo`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `get`, `getElementType`, `getDim`, `getDepthInfo`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 428-454
```cpp
ImageSamplerUseInfo ImageType::getSamplerUseInfo() const {
  return getImpl()->samplerUseInfo;
}

ImageFormat ImageType::getImageFormat() const { return getImpl()->format; }

void TypeExtensionVisitor::addConcrete(ImageType type) {
  // OpTypeImage with a 64-bit integer Sampled Type requires the
  // SPV_EXT_shader_image_int64 extension (companion to Int64ImageEXT).
  if (auto intTy = dyn_cast<IntegerType>(type.getElementType());
      intTy && intTy.getWidth() == 64)
    pushExts<Extension::SPV_EXT_shader_image_int64>();
  add(type.getElementType());
}

void TypeCapabilityVisitor::addConcrete(ImageType type) {
  // Capability requirements for OpTypeImage are determined jointly by Dim,
  // Sampled, MS, and Arrayed - see the SPIR-V spec's "Capabilities" column on
  // OpTypeImage.
  Dim dim = type.getDim();
  bool isMultisampled =
      type.getSamplingInfo() == ImageSamplingInfo::MultiSampled;
  bool isArrayed = type.getArrayedInfo() == ImageArrayedInfo::Arrayed;
  ImageSamplerUseInfo sampler = type.getSamplerUseInfo();
  bool noSampler = sampler == ImageSamplerUseInfo::NoSampler;
  bool needSampler = sampler == ImageSamplerUseInfo::NeedSampler;

```
- **EN**: Implements logic around `getSamplerUseInfo`, `getImpl`, `getImageFormat`, `addConcrete`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getSamplerUseInfo`, `getImpl`, `getImageFormat`, `addConcrete`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 455-490
```cpp
  switch (dim) {
  case Dim::Dim1D:
    if (needSampler)
      pushCaps<Capability::Sampled1D>();
    else if (noSampler)
      pushCaps<Capability::Image1D>();
    else
      pushCaps<Capability::Image1D, Capability::Sampled1D>();
    break;
  case Dim::Dim2D:
    if (isMultisampled && noSampler)
      pushCaps<Capability::StorageImageMultisample>();
    if (isMultisampled && isArrayed)
      pushCaps<Capability::ImageMSArray>();
    break;
  case Dim::Dim3D:
    break;
  case Dim::Cube:
    pushCaps<Capability::Shader>();
    if (isArrayed)
      pushCaps<Capability::ImageCubeArray>();
    break;
  case Dim::Rect:
    pushCaps<Capability::ImageRect, Capability::SampledRect>();
    break;
  case Dim::Buffer:
    if (needSampler)
      pushCaps<Capability::SampledBuffer>();
    else if (noSampler)
      pushCaps<Capability::ImageBuffer>();
    else
      pushCaps<Capability::ImageBuffer, Capability::SampledBuffer>();
    break;
  case Dim::SubpassData:
    pushCaps<Capability::InputAttachment>();
    break;
```
- **EN**: Implements logic around `Sampled1D>`, `Image1D>`, `StorageImageMultisample>`, `ImageMSArray>`, and 6 more symbols.
- **CN**: 围绕 `Sampled1D>`, `Image1D>`, `StorageImageMultisample>`, `ImageMSArray>`, and 6 more symbols 实现具体逻辑。

### Lines 491-508
```cpp
  }

  if (auto fmtCaps = spirv::getCapabilities(type.getImageFormat()))
    capabilities.push_back(*fmtCaps);

  // OpTypeImage with a 64-bit integer Sampled Type requires Int64ImageEXT.
  if (auto intTy = dyn_cast<IntegerType>(type.getElementType());
      intTy && intTy.getWidth() == 64)
    pushCaps<Capability::Int64ImageEXT>();

  add(type.getElementType());
}

//===----------------------------------------------------------------------===//
// PointerType
//===----------------------------------------------------------------------===//

struct spirv::detail::PointerTypeStorage : public TypeStorage {
```
- **EN**: Introduces declarations for `spirv::detail::PointerTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::PointerTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 509-529
```cpp
  // (Type, StorageClass) as the key: Type stored in this struct, and
  // StorageClass stored as TypeStorage's subclass data.
  using KeyTy = std::pair<Type, StorageClass>;

  static PointerTypeStorage *construct(TypeStorageAllocator &allocator,
                                       const KeyTy &key) {
    return new (allocator.allocate<PointerTypeStorage>())
        PointerTypeStorage(key);
  }

  bool operator==(const KeyTy &key) const {
    return key == KeyTy(pointeeType, storageClass);
  }

  PointerTypeStorage(const KeyTy &key)
      : pointeeType(key.first), storageClass(key.second) {}

  Type pointeeType;
  StorageClass storageClass;
};

```
- **EN**: Implements logic around `construct`, `new`, `PointerTypeStorage`, `KeyTy`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `construct`, `new`, `PointerTypeStorage`, `KeyTy`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 530-547
```cpp
PointerType PointerType::get(Type pointeeType, StorageClass storageClass) {
  return Base::get(pointeeType.getContext(), pointeeType, storageClass);
}

Type PointerType::getPointeeType() const { return getImpl()->pointeeType; }

StorageClass PointerType::getStorageClass() const {
  return getImpl()->storageClass;
}

void TypeExtensionVisitor::addConcrete(PointerType type) {
  // Use this pointer type's storage class because this pointer indicates we are
  // using the pointee type in that specific storage class.
  std::optional<StorageClass> oldStorageClass = storage;
  storage = type.getStorageClass();
  add(type.getPointeeType());
  storage = oldStorageClass;

```
- **EN**: Introduces declarations for `because`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `because` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 548-566
```cpp
  if (auto scExts = spirv::getExtensions(type.getStorageClass()))
    extensions.push_back(*scExts);
}

void TypeCapabilityVisitor::addConcrete(PointerType type) {
  // Use this pointer type's storage class because this pointer indicates we are
  // using the pointee type in that specific storage class.
  std::optional<StorageClass> oldStorageClass = storage;
  storage = type.getStorageClass();
  add(type.getPointeeType());
  storage = oldStorageClass;

  if (auto scCaps = spirv::getCapabilities(type.getStorageClass()))
    capabilities.push_back(*scCaps);
}

//===----------------------------------------------------------------------===//
// RuntimeArrayType
//===----------------------------------------------------------------------===//
```
- **EN**: Introduces declarations for `because`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `because` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 567-587
```cpp

struct spirv::detail::RuntimeArrayTypeStorage : public TypeStorage {
  using KeyTy = std::pair<Type, unsigned>;

  static RuntimeArrayTypeStorage *construct(TypeStorageAllocator &allocator,
                                            const KeyTy &key) {
    return new (allocator.allocate<RuntimeArrayTypeStorage>())
        RuntimeArrayTypeStorage(key);
  }

  bool operator==(const KeyTy &key) const {
    return key == KeyTy(elementType, stride);
  }

  RuntimeArrayTypeStorage(const KeyTy &key)
      : elementType(key.first), stride(key.second) {}

  Type elementType;
  unsigned stride;
};

```
- **EN**: Introduces declarations for `spirv::detail::RuntimeArrayTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::RuntimeArrayTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 588-605
```cpp
RuntimeArrayType RuntimeArrayType::get(Type elementType) {
  return Base::get(elementType.getContext(), elementType, /*stride=*/0);
}

RuntimeArrayType RuntimeArrayType::get(Type elementType, unsigned stride) {
  return Base::get(elementType.getContext(), elementType, stride);
}

Type RuntimeArrayType::getElementType() const { return getImpl()->elementType; }

unsigned RuntimeArrayType::getArrayStride() const { return getImpl()->stride; }

void TypeCapabilityVisitor::addConcrete(RuntimeArrayType type) {
  add(type.getElementType());
  pushCaps<Capability::Shader>();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `get`, `getElementType`, `getArrayStride`, `addConcrete`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `get`, `getElementType`, `getArrayStride`, `addConcrete`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 606-624
```cpp
// ScalarType
//===----------------------------------------------------------------------===//

bool ScalarType::classof(Type type) {
  if (auto floatType = dyn_cast<FloatType>(type)) {
    return isValid(floatType);
  }
  if (auto intType = dyn_cast<IntegerType>(type)) {
    return isValid(intType);
  }
  return false;
}

bool ScalarType::isValid(FloatType type) {
  if (type.isF8E4M3FN() || type.isF8E5M2())
    return true;
  return llvm::is_contained({16u, 32u, 64u}, type.getWidth());
}

```
- **EN**: Implements logic around `classof`, `isValid`, `isF8E4M3FN`, `is_contained`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `classof`, `isValid`, `isF8E4M3FN`, `is_contained` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 625-658
```cpp
bool ScalarType::isValid(IntegerType type) {
  return llvm::is_contained({1u, 8u, 16u, 32u, 64u}, type.getWidth());
}

void TypeExtensionVisitor::addConcrete(ScalarType type) {
  if (type.isBF16())
    pushExts<Extension::SPV_KHR_bfloat16>();

  if (type.isF8E4M3FN() || type.isF8E5M2())
    pushExts<Extension::SPV_EXT_float8>();

  // 8- or 16-bit integer/floating-point numbers will require extra extensions
  // to appear in interface storage classes. See SPV_KHR_16bit_storage and
  // SPV_KHR_8bit_storage for more details.
  if (!storage)
    return;

  switch (*storage) {
  case StorageClass::PushConstant:
  case StorageClass::StorageBuffer:
  case StorageClass::Uniform:
    if (type.getIntOrFloatBitWidth() == 8)
      pushExts<Extension::SPV_KHR_8bit_storage>();
    [[fallthrough]];
  case StorageClass::Input:
  case StorageClass::Output:
    if (type.getIntOrFloatBitWidth() == 16)
      pushExts<Extension::SPV_KHR_16bit_storage>();
    break;
  default:
    break;
  }
}

```
- **EN**: Implements logic around `isValid`, `is_contained`, `addConcrete`, `isBF16`, and 6 more symbols; this block expresses reusable interface-based behavior; works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isValid`, `is_contained`, `addConcrete`, `isBF16`, and 6 more symbols 实现具体逻辑；该代码块表达基于接口的可复用行为，并处理方言 IR 实体，如操作、类型或属性。

### Lines 659-679
```cpp
void TypeCapabilityVisitor::addConcrete(ScalarType type) {
  unsigned bitwidth = type.getIntOrFloatBitWidth();

  // 8- or 16-bit integer/floating-point numbers will require extra capabilities
  // to appear in interface storage classes. See SPV_KHR_16bit_storage and
  // SPV_KHR_8bit_storage for more details.

#define STORAGE_CASE(storage, cap8, cap16)                                     \
  case StorageClass::storage: {                                                \
    if (bitwidth == 8) {                                                       \
      pushCaps<Capability::cap8>();                                            \
      return;                                                                  \
    }                                                                          \
    if (bitwidth == 16) {                                                      \
      pushCaps<Capability::cap16>();                                           \
      return;                                                                  \
    }                                                                          \
    /* For 64-bit integers/floats, Int64/Float64 enables support for all */    \
    /* storage classes. Fall through to the next section. */                   \
  } break

```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 680-702
```cpp
  // This part only handles the cases where special bitwidths appearing in
  // interface storage classes.
  if (storage) {
    switch (*storage) {
      STORAGE_CASE(PushConstant, StoragePushConstant8, StoragePushConstant16);
      STORAGE_CASE(StorageBuffer, StorageBuffer8BitAccess,
                   StorageBuffer16BitAccess);
      STORAGE_CASE(Uniform, UniformAndStorageBuffer8BitAccess,
                   StorageUniform16);
    case StorageClass::Input:
    case StorageClass::Output: {
      if (bitwidth == 16) {
        pushCaps<Capability::StorageInputOutput16>();
        return;
      }
      break;
    }
    default:
      break;
    }
  }
#undef STORAGE_CASE

```
- **EN**: Implements logic around `STORAGE_CASE`, `StorageInputOutput16>`; this block expresses reusable interface-based behavior.
- **CN**: 围绕 `STORAGE_CASE`, `StorageInputOutput16>` 实现具体逻辑；该代码块表达基于接口的可复用行为。

### Lines 703-738
```cpp
  // For other non-interface storage classes, require a different set of
  // capabilities for special bitwidths.

#define WIDTH_CASE(type, width)                                                \
  case width:                                                                  \
    pushCaps<Capability::type##width>();                                       \
    break

  if (auto intType = dyn_cast<IntegerType>(type)) {
    switch (bitwidth) {
      WIDTH_CASE(Int, 8);
      WIDTH_CASE(Int, 16);
      WIDTH_CASE(Int, 64);
    case 1:
    case 32:
      break;
    default:
      llvm_unreachable("invalid bitwidth to getCapabilities");
    }
  } else {
    assert(isa<FloatType>(type));
    switch (bitwidth) {
    case 8: {
      if (type.isF8E4M3FN() || type.isF8E5M2())
        pushCaps<Capability::Float8EXT>();
      else
        llvm_unreachable("invalid 8-bit float type to getCapabilities");
      break;
    }
    case 16: {
      if (type.isBF16())
        pushCaps<Capability::BFloat16TypeKHR>();
      else
        pushCaps<Capability::Float16>();
      break;
    }
```
- **EN**: Defines preprocessor-controlled structure, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成式钩子或编译期常量。

### Lines 739-766
```cpp
      WIDTH_CASE(Float, 64);
    case 32:
      break;
    default:
      llvm_unreachable("invalid bitwidth to getCapabilities");
    }
  }

#undef WIDTH_CASE
}

//===----------------------------------------------------------------------===//
// SPIRVType
//===----------------------------------------------------------------------===//

bool SPIRVType::classof(Type type) {
  // Allow SPIR-V dialect types
  if (isa<SPIRVDialect>(type.getDialect()))
    return true;
  if (isa<ScalarType>(type))
    return true;
  if (auto vectorType = dyn_cast<VectorType>(type))
    return CompositeType::isValid(vectorType);
  if (auto tensorArmType = dyn_cast<TensorArmType>(type))
    return isa<ScalarType>(tensorArmType.getElementType());
  return false;
}

```
- **EN**: Implements logic around `WIDTH_CASE`, `classof`, `getDialect`, `isValid`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `WIDTH_CASE`, `classof`, `getDialect`, `isValid`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 767-802
```cpp
bool SPIRVType::isScalarOrVector() {
  return isIntOrFloat() || isa<VectorType>(*this);
}

void SPIRVType::getExtensions(SPIRVType::ExtensionArrayRefVector &extensions,
                              std::optional<StorageClass> storage) {
  TypeExtensionVisitor{extensions, storage}.add(*this);
}

void SPIRVType::getCapabilities(
    SPIRVType::CapabilityArrayRefVector &capabilities,
    std::optional<StorageClass> storage) {
  TypeCapabilityVisitor{capabilities, storage}.add(*this);
}

std::optional<int64_t> SPIRVType::getSizeInBytes() {
  return TypeSwitch<SPIRVType, std::optional<int64_t>>(*this)
      .Case([](ScalarType type) -> std::optional<int64_t> {
        // According to the SPIR-V spec:
        // "There is no physical size or bit pattern defined for values with
        // boolean type. If they are stored (in conjunction with OpVariable),
        // they can only be used with logical addressing operations, not
        // physical, and only with non-externally visible shader Storage
        // Classes: Workgroup, CrossWorkgroup, Private, Function, Input, and
        // Output."
        int64_t bitWidth = type.getIntOrFloatBitWidth();
        if (bitWidth == 1)
          return std::nullopt;
        return bitWidth / 8;
      })
      .Case([](ArrayType type) -> std::optional<int64_t> {
        // Since array type may have an explicit stride declaration (in bytes),
        // we also include it in the calculation.
        auto elementType = cast<SPIRVType>(type.getElementType());
        if (std::optional<int64_t> size = elementType.getSizeInBytes())
          return (*size + type.getArrayStride()) * type.getNumElements();
```
- **EN**: Implements logic around `isScalarOrVector`, `isIntOrFloat`, `getExtensions`, `add`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isScalarOrVector`, `isIntOrFloat`, `getExtensions`, `add`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 803-821
```cpp
        return std::nullopt;
      })
      .Case<VectorType, TensorArmType>([](auto type) -> std::optional<int64_t> {
        if (std::optional<int64_t> elementSize =
                cast<ScalarType>(type.getElementType()).getSizeInBytes())
          return *elementSize * type.getNumElements();
        return std::nullopt;
      })
      .Default(std::nullopt);
}

//===----------------------------------------------------------------------===//
// SampledImageType
//===----------------------------------------------------------------------===//
struct spirv::detail::SampledImageTypeStorage : public TypeStorage {
  using KeyTy = Type;

  SampledImageTypeStorage(const KeyTy &key) : imageType{key} {}

```
- **EN**: Introduces declarations for `spirv::detail::SampledImageTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::SampledImageTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 822-842
```cpp
  bool operator==(const KeyTy &key) const { return key == KeyTy(imageType); }

  static SampledImageTypeStorage *construct(TypeStorageAllocator &allocator,
                                            const KeyTy &key) {
    return new (allocator.allocate<SampledImageTypeStorage>())
        SampledImageTypeStorage(key);
  }

  Type imageType;
};

SampledImageType SampledImageType::get(Type imageType) {
  return Base::get(imageType.getContext(), imageType);
}

SampledImageType
SampledImageType::getChecked(function_ref<InFlightDiagnostic()> emitError,
                             Type imageType) {
  return Base::getChecked(emitError, imageType.getContext(), imageType);
}

```
- **EN**: Implements logic around `KeyTy`, `construct`, `new`, `SampledImageTypeStorage`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `KeyTy`, `construct`, `new`, `SampledImageTypeStorage`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 843-860
```cpp
Type SampledImageType::getImageType() const { return getImpl()->imageType; }

LogicalResult
SampledImageType::verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                                   Type imageType) {
  auto image = dyn_cast<ImageType>(imageType);
  if (!image)
    return emitError() << "expected image type";

  // As per SPIR-V spec: "It [ImageType] must not have a Dim of SubpassData.
  // Additionally, starting with version 1.6, it must not have a Dim of Buffer.
  // ("3.3.6. Type-Declaration Instructions")
  if (llvm::is_contained({Dim::SubpassData, Dim::Buffer}, image.getDim()))
    return emitError() << "Dim must not be SubpassData or Buffer";

  return success();
}

```
- **EN**: Implements logic around `getImageType`, `verifyInvariants`, `emitError`, `is_contained`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getImageType`, `verifyInvariants`, `emitError`, `is_contained`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 861-879
```cpp
//===----------------------------------------------------------------------===//
// SamplerType
//===----------------------------------------------------------------------===//

SamplerType SamplerType::get(MLIRContext *context) {
  return Base::get(context);
}

//===----------------------------------------------------------------------===//
// NamedBarrierType
//===----------------------------------------------------------------------===//

NamedBarrierType NamedBarrierType::get(MLIRContext *context) {
  return Base::get(context);
}

//===----------------------------------------------------------------------===//
// StructType
//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `get`.
- **CN**: 围绕 `get` 实现具体逻辑。

### Lines 880-897
```cpp

/// Type storage for SPIR-V structure types:
///
/// Structures are uniqued using:
/// - for identified structs:
///   - a string identifier;
/// - for literal structs:
///   - a list of member types;
///   - a list of member offset info;
///   - a list of member decoration info;
///   - a list of struct decoration info.
///
/// Identified structures only have a mutable component consisting of:
/// - a list of member types;
/// - a list of member offset info;
/// - a list of member decoration info;
/// - a list of struct decoration info.
struct spirv::detail::StructTypeStorage : public TypeStorage {
```
- **EN**: Introduces declarations for `decoration`, `spirv::detail::StructTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `decoration`, `spirv::detail::StructTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 898-920
```cpp
  /// Construct a storage object for an identified struct type. A struct type
  /// associated with such storage must call StructType::trySetBody(...) later
  /// in order to mutate the storage object providing the actual content.
  StructTypeStorage(StringRef identifier)
      : memberTypesAndIsBodySet(nullptr, false), offsetInfo(nullptr),
        numMembers(0), numMemberDecorations(0), memberDecorationsInfo(nullptr),
        numStructDecorations(0), structDecorationsInfo(nullptr),
        identifier(identifier) {}

  /// Construct a storage object for a literal struct type. A struct type
  /// associated with such storage is immutable.
  StructTypeStorage(
      unsigned numMembers, Type const *memberTypes,
      StructType::OffsetInfo const *layoutInfo, unsigned numMemberDecorations,
      StructType::MemberDecorationInfo const *memberDecorationsInfo,
      unsigned numStructDecorations,
      StructType::StructDecorationInfo const *structDecorationsInfo)
      : memberTypesAndIsBodySet(memberTypes, false), offsetInfo(layoutInfo),
        numMembers(numMembers), numMemberDecorations(numMemberDecorations),
        memberDecorationsInfo(memberDecorationsInfo),
        numStructDecorations(numStructDecorations),
        structDecorationsInfo(structDecorationsInfo) {}

```
- **EN**: Introduces declarations for `type`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `type` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 921-941
```cpp
  /// A storage key is divided into 2 parts:
  /// - for identified structs:
  ///   - a StringRef representing the struct identifier;
  /// - for literal structs:
  ///   - an ArrayRef<Type> for member types;
  ///   - an ArrayRef<StructType::OffsetInfo> for member offset info;
  ///   - an ArrayRef<StructType::MemberDecorationInfo> for member decoration
  ///     info;
  ///   - an ArrayRef<StructType::StructDecorationInfo> for struct decoration
  ///     info.
  ///
  /// An identified struct type is uniqued only by the first part (field 0)
  /// of the key.
  ///
  /// A literal struct type is uniqued only by the second part (fields 1, 2, 3
  /// and 4) of the key. The identifier field (field 0) must be empty.
  using KeyTy =
      std::tuple<StringRef, ArrayRef<Type>, ArrayRef<StructType::OffsetInfo>,
                 ArrayRef<StructType::MemberDecorationInfo>,
                 ArrayRef<StructType::StructDecorationInfo>>;

```
- **EN**: Introduces declarations for `identifier`, `decoration`, `type`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `identifier`, `decoration`, `type` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 942-959
```cpp
  /// For identified structs, return true if the given key contains the same
  /// identifier.
  ///
  /// For literal structs, return true if the given key contains a matching list
  /// of member types + offset info + decoration info.
  bool operator==(const KeyTy &key) const {
    if (isIdentified()) {
      // Identified types are uniqued by their identifier.
      return getIdentifier() == std::get<0>(key);
    }

    return key == KeyTy(StringRef(), getMemberTypes(), getOffsetInfo(),
                        getMemberDecorationsInfo(), getStructDecorationsInfo());
  }

  /// If the given key contains a non-empty identifier, this method constructs
  /// an identified struct and leaves the rest of the struct type data to be set
  /// through a later call to StructType::trySetBody(...).
```
- **EN**: Introduces declarations for `and`, `type`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `and`, `type` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 960-977
```cpp
  ///
  /// If, on the other hand, the key contains an empty identifier, a literal
  /// struct is constructed using the other fields of the key.
  static StructTypeStorage *construct(TypeStorageAllocator &allocator,
                                      const KeyTy &key) {
    StringRef keyIdentifier = std::get<0>(key);

    if (!keyIdentifier.empty()) {
      StringRef identifier = allocator.copyInto(keyIdentifier);

      // Identified StructType body/members will be set through trySetBody(...)
      // later.
      return new (allocator.allocate<StructTypeStorage>())
          StructTypeStorage(identifier);
    }

    ArrayRef<Type> keyTypes = std::get<1>(key);

```
- **EN**: Introduces declarations for `is`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `is` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 978-1000
```cpp
    // Copy the member type and layout information into the bump pointer
    const Type *typesList = nullptr;
    if (!keyTypes.empty()) {
      typesList = allocator.copyInto(keyTypes).data();
    }

    const StructType::OffsetInfo *offsetInfoList = nullptr;
    if (!std::get<2>(key).empty()) {
      ArrayRef<StructType::OffsetInfo> keyOffsetInfo = std::get<2>(key);
      assert(keyOffsetInfo.size() == keyTypes.size() &&
             "size of offset information must be same as the size of number of "
             "elements");
      offsetInfoList = allocator.copyInto(keyOffsetInfo).data();
    }

    const StructType::MemberDecorationInfo *memberDecorationList = nullptr;
    unsigned numMemberDecorations = 0;
    if (!std::get<3>(key).empty()) {
      auto keyMemberDecorations = std::get<3>(key);
      numMemberDecorations = keyMemberDecorations.size();
      memberDecorationList = allocator.copyInto(keyMemberDecorations).data();
    }

```
- **EN**: Implements logic around `empty`, `copyInto`, `get`, `assert`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `empty`, `copyInto`, `get`, `assert`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1001-1024
```cpp
    const StructType::StructDecorationInfo *structDecorationList = nullptr;
    unsigned numStructDecorations = 0;
    if (!std::get<4>(key).empty()) {
      auto keyStructDecorations = std::get<4>(key);
      numStructDecorations = keyStructDecorations.size();
      structDecorationList = allocator.copyInto(keyStructDecorations).data();
    }

    return new (allocator.allocate<StructTypeStorage>()) StructTypeStorage(
        keyTypes.size(), typesList, offsetInfoList, numMemberDecorations,
        memberDecorationList, numStructDecorations, structDecorationList);
  }

  ArrayRef<Type> getMemberTypes() const {
    return ArrayRef<Type>(memberTypesAndIsBodySet.getPointer(), numMembers);
  }

  ArrayRef<StructType::OffsetInfo> getOffsetInfo() const {
    if (offsetInfo) {
      return ArrayRef<StructType::OffsetInfo>(offsetInfo, numMembers);
    }
    return {};
  }

```
- **EN**: Implements logic around `get`, `size`, `copyInto`, `new`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `get`, `size`, `copyInto`, `new`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1025-1043
```cpp
  ArrayRef<StructType::MemberDecorationInfo> getMemberDecorationsInfo() const {
    if (memberDecorationsInfo) {
      return ArrayRef<StructType::MemberDecorationInfo>(memberDecorationsInfo,
                                                        numMemberDecorations);
    }
    return {};
  }

  ArrayRef<StructType::StructDecorationInfo> getStructDecorationsInfo() const {
    if (structDecorationsInfo)
      return ArrayRef<StructType::StructDecorationInfo>(structDecorationsInfo,
                                                        numStructDecorations);
    return {};
  }

  StringRef getIdentifier() const { return identifier; }

  bool isIdentified() const { return !identifier.empty(); }

```
- **EN**: Implements logic around `getMemberDecorationsInfo`, `MemberDecorationInfo>`, `getStructDecorationsInfo`, `StructDecorationInfo>`, and 2 more symbols.
- **CN**: 围绕 `getMemberDecorationsInfo`, `MemberDecorationInfo>`, `getStructDecorationsInfo`, `StructDecorationInfo>`, and 2 more symbols 实现具体逻辑。

### Lines 1044-1066
```cpp
  /// Sets the struct type content for identified structs. Calling this method
  /// is only valid for identified structs.
  ///
  /// Fails under the following conditions:
  /// - If called for a literal struct;
  /// - If called for an identified struct whose body was set before (through a
  /// call to this method) but with different contents from the passed
  /// arguments.
  LogicalResult
  mutate(TypeStorageAllocator &allocator, ArrayRef<Type> structMemberTypes,
         ArrayRef<StructType::OffsetInfo> structOffsetInfo,
         ArrayRef<StructType::MemberDecorationInfo> structMemberDecorationInfo,
         ArrayRef<StructType::StructDecorationInfo> structDecorationInfo) {
    if (!isIdentified())
      return failure();

    if (memberTypesAndIsBodySet.getInt() &&
        (getMemberTypes() != structMemberTypes ||
         getOffsetInfo() != structOffsetInfo ||
         getMemberDecorationsInfo() != structMemberDecorationInfo ||
         getStructDecorationsInfo() != structDecorationInfo))
      return failure();

```
- **EN**: Introduces declarations for `type`, `whose`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `type`, `whose` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1067-1087
```cpp
    memberTypesAndIsBodySet.setInt(true);
    numMembers = structMemberTypes.size();

    // Copy the member type and layout information into the bump pointer.
    if (!structMemberTypes.empty())
      memberTypesAndIsBodySet.setPointer(
          allocator.copyInto(structMemberTypes).data());

    if (!structOffsetInfo.empty()) {
      assert(structOffsetInfo.size() == structMemberTypes.size() &&
             "size of offset information must be same as the size of number of "
             "elements");
      offsetInfo = allocator.copyInto(structOffsetInfo).data();
    }

    if (!structMemberDecorationInfo.empty()) {
      numMemberDecorations = structMemberDecorationInfo.size();
      memberDecorationsInfo =
          allocator.copyInto(structMemberDecorationInfo).data();
    }

```
- **EN**: Implements logic around `setInt`, `size`, `empty`, `setPointer`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `setInt`, `size`, `empty`, `setPointer`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1088-1105
```cpp
    if (!structDecorationInfo.empty()) {
      numStructDecorations = structDecorationInfo.size();
      structDecorationsInfo = allocator.copyInto(structDecorationInfo).data();
    }

    return success();
  }

  llvm::PointerIntPair<Type const *, 1, bool> memberTypesAndIsBodySet;
  StructType::OffsetInfo const *offsetInfo;
  unsigned numMembers;
  unsigned numMemberDecorations;
  StructType::MemberDecorationInfo const *memberDecorationsInfo;
  unsigned numStructDecorations;
  StructType::StructDecorationInfo const *structDecorationsInfo;
  StringRef identifier;
};

```
- **EN**: Implements logic around `empty`, `size`, `copyInto`, `success`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `empty`, `size`, `copyInto`, `success` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1106-1126
```cpp
StructType
StructType::get(ArrayRef<Type> memberTypes,
                ArrayRef<StructType::OffsetInfo> offsetInfo,
                ArrayRef<StructType::MemberDecorationInfo> memberDecorations,
                ArrayRef<StructType::StructDecorationInfo> structDecorations) {
  assert(!memberTypes.empty() && "Struct needs at least one member type");
  // Sort the decorations.
  SmallVector<StructType::MemberDecorationInfo, 4> sortedMemberDecorations(
      memberDecorations);
  llvm::array_pod_sort(sortedMemberDecorations.begin(),
                       sortedMemberDecorations.end());
  SmallVector<StructType::StructDecorationInfo, 1> sortedStructDecorations(
      structDecorations);
  llvm::array_pod_sort(sortedStructDecorations.begin(),
                       sortedStructDecorations.end());

  return Base::get(memberTypes.vec().front().getContext(),
                   /*identifier=*/StringRef(), memberTypes, offsetInfo,
                   sortedMemberDecorations, sortedStructDecorations);
}

```
- **EN**: Implements logic around `get`, `assert`, `sortedMemberDecorations`, `array_pod_sort`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `get`, `assert`, `sortedMemberDecorations`, `array_pod_sort`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1127-1150
```cpp
StructType StructType::getIdentified(MLIRContext *context,
                                     StringRef identifier) {
  assert(!identifier.empty() &&
         "StructType identifier must be non-empty string");

  return Base::get(context, identifier, ArrayRef<Type>(),
                   ArrayRef<StructType::OffsetInfo>(),
                   ArrayRef<StructType::MemberDecorationInfo>(),
                   ArrayRef<StructType::StructDecorationInfo>());
}

StructType StructType::getEmpty(MLIRContext *context, StringRef identifier) {
  StructType newStructType = Base::get(
      context, identifier, ArrayRef<Type>(), ArrayRef<StructType::OffsetInfo>(),
      ArrayRef<StructType::MemberDecorationInfo>(),
      ArrayRef<StructType::StructDecorationInfo>());
  // Set an empty body in case this is a identified struct.
  if (newStructType.isIdentified() &&
      failed(newStructType.trySetBody(
          ArrayRef<Type>(), ArrayRef<StructType::OffsetInfo>(),
          ArrayRef<StructType::MemberDecorationInfo>(),
          ArrayRef<StructType::StructDecorationInfo>())))
    return StructType();

```
- **EN**: Implements logic around `getIdentified`, `assert`, `get`, `OffsetInfo>`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getIdentified`, `assert`, `get`, `OffsetInfo>`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1151-1169
```cpp
  return newStructType;
}

StringRef StructType::getIdentifier() const { return getImpl()->identifier; }

bool StructType::isIdentified() const { return getImpl()->isIdentified(); }

unsigned StructType::getNumElements() const { return getImpl()->numMembers; }

Type StructType::getElementType(unsigned index) const {
  assert(getNumElements() > index && "member index out of range");
  return getImpl()->memberTypesAndIsBodySet.getPointer()[index];
}

TypeRange StructType::getElementTypes() const {
  return TypeRange(getImpl()->memberTypesAndIsBodySet.getPointer(),
                   getNumElements());
}

```
- **EN**: Implements logic around `getIdentifier`, `isIdentified`, `getNumElements`, `getElementType`, and 4 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getIdentifier`, `isIdentified`, `getNumElements`, `getElementType`, and 4 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1170-1194
```cpp
bool StructType::hasOffset() const { return getImpl()->offsetInfo; }

bool StructType::hasDecoration(spirv::Decoration decoration) const {
  for (StructType::StructDecorationInfo info :
       getImpl()->getStructDecorationsInfo())
    if (info.decoration == decoration)
      return true;

  return false;
}

uint64_t StructType::getMemberOffset(unsigned index) const {
  assert(getNumElements() > index && "member index out of range");
  return getImpl()->offsetInfo[index];
}

void StructType::getMemberDecorations(
    SmallVectorImpl<StructType::MemberDecorationInfo> &memberDecorations)
    const {
  memberDecorations.clear();
  auto implMemberDecorations = getImpl()->getMemberDecorationsInfo();
  memberDecorations.append(implMemberDecorations.begin(),
                           implMemberDecorations.end());
}

```
- **EN**: Implements logic around `hasOffset`, `hasDecoration`, `getImpl`, `getMemberOffset`, and 5 more symbols; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `hasOffset`, `hasDecoration`, `getImpl`, `getMemberOffset`, and 5 more symbols 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 1195-1219
```cpp
void StructType::getMemberDecorations(
    unsigned index,
    SmallVectorImpl<StructType::MemberDecorationInfo> &decorationsInfo) const {
  assert(getNumElements() > index && "member index out of range");
  auto memberDecorations = getImpl()->getMemberDecorationsInfo();
  decorationsInfo.clear();
  for (const auto &memberDecoration : memberDecorations) {
    if (memberDecoration.memberIndex == index) {
      decorationsInfo.push_back(memberDecoration);
    }
    if (memberDecoration.memberIndex > index) {
      // Early exit since the decorations are stored sorted.
      return;
    }
  }
}

void StructType::getStructDecorations(
    SmallVectorImpl<StructType::StructDecorationInfo> &structDecorations)
    const {
  structDecorations.clear();
  auto implDecorations = getImpl()->getStructDecorationsInfo();
  structDecorations.append(implDecorations.begin(), implDecorations.end());
}

```
- **EN**: Implements logic around `getMemberDecorations`, `assert`, `getImpl`, `clear`, and 3 more symbols.
- **CN**: 围绕 `getMemberDecorations`, `assert`, `getImpl`, `clear`, and 3 more symbols 实现具体逻辑。

### Lines 1220-1239
```cpp
LogicalResult
StructType::trySetBody(ArrayRef<Type> memberTypes,
                       ArrayRef<OffsetInfo> offsetInfo,
                       ArrayRef<MemberDecorationInfo> memberDecorations,
                       ArrayRef<StructDecorationInfo> structDecorations) {
  return Base::mutate(memberTypes, offsetInfo, memberDecorations,
                      structDecorations);
}

llvm::hash_code spirv::hash_value(
    const StructType::MemberDecorationInfo &memberDecorationInfo) {
  return llvm::hash_combine(memberDecorationInfo.memberIndex,
                            memberDecorationInfo.decoration);
}

llvm::hash_code spirv::hash_value(
    const StructType::StructDecorationInfo &structDecorationInfo) {
  return llvm::hash_value(structDecorationInfo.decoration);
}

```
- **EN**: Implements logic around `trySetBody`, `mutate`, `hash_value`, `hash_combine`; this block works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `trySetBody`, `mutate`, `hash_value`, `hash_combine` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 1240-1257
```cpp
//===----------------------------------------------------------------------===//
// MatrixType
//===----------------------------------------------------------------------===//

struct spirv::detail::MatrixTypeStorage : public TypeStorage {
  // Use a 64-bit integer as a column count internally to better support a
  // `ShapedType` interface. See comment in `CooperativeMatrixType` for more
  // context.
  using KeyTy = std::tuple<Type, int64_t>;

  MatrixTypeStorage(const KeyTy &key)
      : columnType(std::get<0>(key)),
        shape({cast<VectorType>(std::get<0>(key)).getShape()[0],
               std::get<1>(key)}) {}

  static MatrixTypeStorage *construct(TypeStorageAllocator &allocator,
                                      const KeyTy &key) {

```
- **EN**: Introduces declarations for `spirv::detail::MatrixTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::MatrixTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1258-1280
```cpp
    // Initialize the memory using placement new.
    return new (allocator.allocate<MatrixTypeStorage>()) MatrixTypeStorage(key);
  }

  bool operator==(const KeyTy &key) const {
    return key == KeyTy(columnType, shape[1]);
  }

  Type columnType;
  // [#rows, #columns]
  std::array<int64_t, 2> shape;
};

MatrixType MatrixType::get(Type columnType, uint32_t columnCount) {
  return Base::get(columnType.getContext(), columnType, columnCount);
}

MatrixType MatrixType::getChecked(function_ref<InFlightDiagnostic()> emitError,
                                  Type columnType, uint32_t columnCount) {
  return Base::getChecked(emitError, columnType.getContext(), columnType,
                          columnCount);
}

```
- **EN**: Implements logic around `new`, `KeyTy`, `get`, `getChecked`; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `new`, `KeyTy`, `get`, `getChecked` 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1281-1300
```cpp
LogicalResult
MatrixType::verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                             Type columnType, uint32_t columnCount) {
  if (columnCount < 2 || columnCount > 4)
    return emitError() << "matrix can have 2, 3, or 4 columns only";

  if (!isValidColumnType(columnType))
    return emitError() << "matrix columns must be vectors of floats";

  /// The underlying vectors (columns) must be of size 2, 3, or 4
  ArrayRef<int64_t> columnShape = cast<VectorType>(columnType).getShape();
  if (columnShape.size() != 1)
    return emitError() << "matrix columns must be 1D vectors";

  if (columnShape[0] < 2 || columnShape[0] > 4)
    return emitError() << "matrix columns must be of size 2, 3, or 4";

  return success();
}

```
- **EN**: Implements logic around `verifyInvariants`, `emitError`, `isValidColumnType`, `getShape`, and 2 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `verifyInvariants`, `emitError`, `isValidColumnType`, `getShape`, and 2 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1301-1321
```cpp
/// Returns true if the matrix elements are vectors of float elements
bool MatrixType::isValidColumnType(Type columnType) {
  if (auto vectorType = dyn_cast<VectorType>(columnType)) {
    if (isa<FloatType>(vectorType.getElementType()))
      return true;
  }
  return false;
}

Type MatrixType::getColumnType() const { return getImpl()->columnType; }

Type MatrixType::getElementType() const {
  return cast<VectorType>(getImpl()->columnType).getElementType();
}

unsigned MatrixType::getNumColumns() const {
  assert(getImpl()->shape[1] >= 0); // Also includes ShapedType::kDynamic.
  assert(getImpl()->shape[1] <= std::numeric_limits<unsigned>::max());
  return static_cast<uint32_t>(getImpl()->shape[1]);
}

```
- **EN**: Implements logic around `isValidColumnType`, `getElementType`, `getColumnType`, `getImpl`, and 3 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `isValidColumnType`, `getElementType`, `getColumnType`, `getImpl`, and 3 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1322-1339
```cpp
unsigned MatrixType::getNumRows() const {
  assert(getImpl()->shape[0] >= 0); // Also includes ShapedType::kDynamic.
  assert(getImpl()->shape[0] <= std::numeric_limits<unsigned>::max());
  return static_cast<uint32_t>(getImpl()->shape[0]);
}

unsigned MatrixType::getNumElements() const {
  return getNumColumns() * getNumRows();
}

ArrayRef<int64_t> MatrixType::getShape() const { return getImpl()->shape; }

void TypeCapabilityVisitor::addConcrete(MatrixType type) {
  add(type.getColumnType());
  pushCaps<Capability::Matrix>();
}

//===----------------------------------------------------------------------===//
```
- **EN**: Implements logic around `getNumRows`, `assert`, `static_cast`, `getNumElements`, and 5 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `getNumRows`, `assert`, `static_cast`, `getNumElements`, and 5 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1340-1358
```cpp
// TensorArmType
//===----------------------------------------------------------------------===//

struct spirv::detail::TensorArmTypeStorage final : TypeStorage {
  using KeyTy = std::tuple<ArrayRef<int64_t>, Type>;

  static TensorArmTypeStorage *construct(TypeStorageAllocator &allocator,
                                         const KeyTy &key) {
    auto [shape, elementType] = key;
    shape = allocator.copyInto(shape);
    return new (allocator.allocate<TensorArmTypeStorage>())
        TensorArmTypeStorage(shape, elementType);
  }

  static llvm::hash_code hashKey(const KeyTy &key) {
    auto [shape, elementType] = key;
    return llvm::hash_combine(shape, elementType);
  }

```
- **EN**: Introduces declarations for `spirv::detail::TensorArmTypeStorage`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `spirv::detail::TensorArmTypeStorage` 等声明，建立后续使用的方言级类型或模式记录。

### Lines 1359-1378
```cpp
  bool operator==(const KeyTy &key) const {
    return key == KeyTy(shape, elementType);
  }

  TensorArmTypeStorage(ArrayRef<int64_t> shape, Type elementType)
      : shape(shape), elementType(elementType) {}

  ArrayRef<int64_t> shape;
  Type elementType;
};

TensorArmType TensorArmType::get(ArrayRef<int64_t> shape, Type elementType) {
  return Base::get(elementType.getContext(), shape, elementType);
}

TensorArmType TensorArmType::cloneWith(std::optional<ArrayRef<int64_t>> shape,
                                       Type elementType) const {
  return TensorArmType::get(shape.value_or(getShape()), elementType);
}

```
- **EN**: Implements logic around `KeyTy`, `TensorArmTypeStorage`, `shape`, `get`, and 1 more symbols; this block works with dialect IR entities such as ops, types, or attributes.
- **CN**: 围绕 `KeyTy`, `TensorArmTypeStorage`, `shape`, `get`, and 1 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性。

### Lines 1379-1404
```cpp
Type TensorArmType::getElementType() const { return getImpl()->elementType; }
ArrayRef<int64_t> TensorArmType::getShape() const { return getImpl()->shape; }

void TypeExtensionVisitor::addConcrete(TensorArmType type) {
  add(type.getElementType());
  pushExts<Extension::SPV_ARM_tensors>();
}

void TypeCapabilityVisitor::addConcrete(TensorArmType type) {
  add(type.getElementType());
  pushCaps<Capability::TensorsARM>();
}

LogicalResult
TensorArmType::verifyInvariants(function_ref<InFlightDiagnostic()> emitError,
                                ArrayRef<int64_t> shape, Type elementType) {
  if (llvm::is_contained(shape, 0))
    return emitError() << "arm.tensor do not support dimensions = 0";
  if (llvm::any_of(shape, [](int64_t dim) { return dim < 0; }) &&
      llvm::any_of(shape, [](int64_t dim) { return dim > 0; }))
    return emitError()
           << "arm.tensor shape dimensions must be either fully dynamic or "
              "completed shaped";
  return success();
}

```
- **EN**: Implements logic around `getElementType`, `getShape`, `addConcrete`, `add`, and 7 more symbols; this block works with dialect IR entities such as ops, types, or attributes; coordinates behavior across core structured MLIR dialects.
- **CN**: 围绕 `getElementType`, `getShape`, `addConcrete`, `add`, and 7 more symbols 实现具体逻辑；该代码块处理方言 IR 实体，如操作、类型或属性，并协调核心结构化 MLIR 方言之间的行为。

### Lines 1405-1413
```cpp
//===----------------------------------------------------------------------===//
// SPIR-V Dialect
//===----------------------------------------------------------------------===//

void SPIRVDialect::registerTypes() {
  addTypes<ArrayType, CooperativeMatrixType, ImageType, MatrixType,
           NamedBarrierType, PointerType, RuntimeArrayType, SampledImageType,
           SamplerType, StructType, TensorArmType>();
}
```
- **EN**: Implements logic around `registerTypes`, `TensorArmType>`.
- **CN**: 围绕 `registerTypes`, `TensorArmType>` 实现具体逻辑。

## Key Concepts / 关键概念

- **Dialect IR definitions / 方言 IR 定义**:
  - **EN**: Defines or implements dialect operations, attributes, types, traits, and registration hooks.
  - **CN**: 定义或实现方言操作、属性、类型、trait 以及注册钩子。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/SPIRV/IR/SPIRVTypes.h`, `mlir/Dialect/SPIRV/IR/SPIRVDialect.h`, `mlir/Dialect/SPIRV/IR/SPIRVEnums.h`, `mlir/IR/BuiltinTypes.h`, `mlir/Support/LLVM.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/ErrorHandling.h`
- **Standard-library headers / 标准库头文件**: `<cstdint>`, `<optional>`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), LLVM ADT containers and low-level helpers / LLVM ADT 容器与底层辅助工具 (2), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1), support utilities and LLVM interop helpers / 支持工具与 LLVM 互操作辅助逻辑 (1), LLVM support-library facilities / LLVM Support 库设施 (1)
