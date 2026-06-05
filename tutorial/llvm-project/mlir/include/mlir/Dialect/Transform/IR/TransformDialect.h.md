# TransformDialect.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/Dialect/Transform/IR/TransformDialect.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR TransformDialect component. The leading comments describe it as: Concrete base class for CRTP TransformDialectDataBase. Must not be used.
- **用途（CN）**: 声明 MLIR TransformDialect 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
````cpp
//===- TransformDialect.h - Transform Dialect Definition --------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT_H
#define MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT_H

#include "mlir/IR/Dialect.h"
#include "mlir/IR/PatternMatch.h"
#include "mlir/Support/LLVM.h"
#include "mlir/Support/TypeID.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/StringMap.h"
#include <optional>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 20-67
````cpp
namespace mlir {
namespace transform {

namespace detail {
/// Concrete base class for CRTP TransformDialectDataBase. Must not be used
/// directly.
class TransformDialectDataBase {
public:
  virtual ~TransformDialectDataBase() = default;

  /// Returns the dynamic type ID of the subclass.
  TypeID getTypeID() const { return typeID; }

protected:
  /// Must be called by the subclass with the appropriate type ID.
  explicit TransformDialectDataBase(TypeID typeID, MLIRContext *ctx)
      : typeID(typeID), ctx(ctx) {}

  /// Return the MLIR context.
  MLIRContext *getContext() const { return ctx; }

private:
  /// The type ID of the subclass.
  const TypeID typeID;

  /// The MLIR context.
  MLIRContext *ctx;
};
} // namespace detail

/// Base class for additional data owned by the Transform dialect. Extensions
/// may communicate with each other using this data. The data object is
/// identified by the TypeID of the specific data subclass, querying the data of
/// the same subclass returns a reference to the same object. When a Transform
/// dialect extension is initialized, it can populate the data in the specific
/// subclass. When a Transform op is applied, it can read (but not mutate) the
/// data in the specific subclass, including the data provided by other
/// extensions.
///
/// This follows CRTP: derived classes must list themselves as template
/// argument.
template <typename DerivedTy>
class TransformDialectData : public detail::TransformDialectDataBase {
protected:
  /// Forward the TypeID of the derived class to the base.
  TransformDialectData(MLIRContext *ctx)
      : TransformDialectDataBase(TypeID::get<DerivedTy>(), ctx) {}
};
````
- **EN**: This C++ declaration introduces `TransformDialectDataBase` and establishes part of the API surface for `TransformDialect`. Representative entry points here include `TransformDialectDataBase`, `getTypeID`, `typeID`, `ctx`.
- **CN**: 该 C++ 声明引入了 `TransformDialectDataBase`，并构成 `TransformDialect` API 表面的一部分。 这一段可见的代表性接口包括 `TransformDialectDataBase`, `getTypeID`, `typeID`, `ctx`。

### Lines 68-83
````cpp
#if LLVM_ENABLE_ABI_BREAKING_CHECKS
namespace detail {
/// Asserts that the operations provided as template arguments implement the
/// TransformOpInterface and MemoryEffectsOpInterface. This must be a dynamic
/// assertion since interface implementations may be registered at runtime.
void checkImplementsTransformOpInterface(StringRef name, MLIRContext *context);

/// Asserts that the type provided as template argument implements the
/// TransformHandleTypeInterface. This must be a dynamic assertion since
/// interface implementations may be registered at runtime.
void checkImplementsTransformHandleTypeInterface(TypeID typeID,
                                                 MLIRContext *context);
} // namespace detail
#endif // LLVM_ENABLE_ABI_BREAKING_CHECKS
} // namespace transform
} // namespace mlir
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 86-86
````cpp
#include "mlir/Dialect/Transform/IR/TransformDialect.h.inc"
````
- **EN**: This block pulls in the MLIR/LLVM/TableGen dependencies required by later declarations and helps establish the compilation boundary of the file.
- **CN**: 该代码块引入后续声明所依赖的 MLIR/LLVM/TableGen 头文件或记录文件，用于建立本文件的编译边界。

### Lines 88-134
````cpp
namespace mlir {
namespace transform {

/// Base class for extensions of the Transform dialect that supports injecting
/// operations into the Transform dialect at load time. Concrete extensions are
/// expected to derive this class and register operations in the constructor.
/// They can be registered with the DialectRegistry and automatically applied
/// to the Transform dialect when it is loaded.
///
/// Derived classes are expected to define a `void init()` function in which
/// they can call various protected methods of the base class to register
/// extension operations and declare their dependencies.
///
/// By default, the extension is configured both for construction of the
/// Transform IR and for its application to some payload. If only the
/// construction is desired, the extension can be switched to "build-only" mode
/// that avoids loading the dialects that are only necessary for transforming
/// the payload. To perform the switch, the extension must be wrapped into the
/// `BuildOnly` class template (see below) when it is registered, as in:
///
///    dialectRegistry.addExtension<BuildOnly<MyTransformDialectExt>>();
///
/// instead of:
///
///    dialectRegistry.addExtension<MyTransformDialectExt>();
///
/// Derived classes must reexport the constructor of this class or otherwise
/// forward its boolean argument to support this behavior.
template <typename DerivedTy, typename... ExtraDialects>
class TransformDialectExtension
    : public DialectExtension<DerivedTy, TransformDialect, ExtraDialects...> {
  using Initializer = std::function<void(TransformDialect *)>;
  using DialectLoader = std::function<void(MLIRContext *)>;

public:
  /// Extension application hook. Actually loads the dependent dialects and
  /// registers the additional operations. Not expected to be called directly.
  void apply(MLIRContext *context, TransformDialect *transformDialect,
             ExtraDialects *...) const final {
    for (const DialectLoader &loader : dialectLoaders)
      loader(context);

    // Only load generated dialects if the user intends to apply
    // transformations specified by the extension.
    if (!buildOnly)
      for (const DialectLoader &loader : generatedDialectLoaders)
        loader(context);
````
- **EN**: This C++ declaration introduces `TransformDialectExtension` and establishes part of the API surface for `TransformDialect`. Representative entry points here include `void`, `apply`, `loader`.
- **CN**: 该 C++ 声明引入了 `TransformDialectExtension`，并构成 `TransformDialect` API 表面的一部分。 这一段可见的代表性接口包括 `void`, `apply`, `loader`。

### Lines 135-182
````cpp
    for (const Initializer &init : initializers)
      init(transformDialect);
  }

protected:
  using Base = TransformDialectExtension<DerivedTy, ExtraDialects...>;

  /// Extension constructor. The argument indicates whether to skip generated
  /// dialects when applying the extension.
  explicit TransformDialectExtension(bool buildOnly = false)
      : buildOnly(buildOnly) {
    static_cast<DerivedTy *>(this)->init();
  }

  /// Registers a custom initialization step to be performed when the extension
  /// is applied to the dialect while loading. This is discouraged in favor of
  /// more specific calls `declareGeneratedDialect`, `addDialectDataInitializer`
  /// etc. `Func` must be convertible to the `void (MLIRContext *)` form. It
  /// will be called during the extension initialization and given the current
  /// MLIR context. This may be used to attach additional interfaces that cannot
  /// be attached elsewhere.
  template <typename Func>
  void addCustomInitializationStep(Func &&func) {
    std::function<void(MLIRContext *)> initializer = func;
    dialectLoaders.push_back(
        [init = std::move(initializer)](MLIRContext *ctx) { init(ctx); });
  }

  /// Registers the given function as one of the initializers for the
  /// dialect-owned data of the kind specified as template argument. The
  /// function must be convertible to the `void (DataTy &)` form. It will be
  /// called during the extension initialization and will be given a mutable
  /// reference to `DataTy`. The callback is expected to append data to the
  /// given storage, and is not allowed to remove or destructively mutate the
  /// existing data. The order in which callbacks from different extensions are
  /// executed is unspecified so the callbacks may not rely on data being
  /// already present. `DataTy` must be a class deriving `TransformDialectData`.
  template <typename DataTy, typename Func>
  void addDialectDataInitializer(Func &&func) {
    static_assert(std::is_base_of_v<detail::TransformDialectDataBase, DataTy>,
                  "only classes deriving TransformDialectData are accepted");

    std::function<void(DataTy &)> initializer = func;
    initializers.push_back(
        [init = std::move(initializer)](TransformDialect *transformDialect) {
          init(transformDialect->getOrCreateExtraData<DataTy>());
        });
  }
````
- **EN**: This block groups callable interfaces such as `init`, `TransformDialectExtension`, `buildOnly`, `addCustomInitializationStep`, indicating how `TransformDialect` is queried or updated.
- **CN**: 该代码块聚合了 `init`, `TransformDialectExtension`, `buildOnly`, `addCustomInitializationStep` 等可调用接口，展示了如何查询或更新 `TransformDialect`。

### Lines 184-231
````cpp
  /// Hook for derived classes to inject constructor behavior.
  void init() {}

  /// Injects the operations into the Transform dialect. The operations must
  /// implement the TransformOpInterface and MemoryEffectsOpInterface, and the
  /// implementations must be already available when the operation is injected.
  template <typename... OpTys>
  void registerTransformOps() {
    initializers.push_back([](TransformDialect *transformDialect) {
      transformDialect->addOperationsChecked<OpTys...>();
    });
  }

  /// Injects the attributes into the Transform dialect. The attributes must
  /// provide a `getMnemonic` static method returning an object convertible to
  /// `StringRef` that is unique across all injected attributes.
  template <typename... AttrTys>
  void registerAttributes() {
    initializers.push_back([](TransformDialect *transformDialect) {
      transformDialect->addAttributesChecked<AttrTys...>();
    });
  }

  /// Injects the types into the Transform dialect. The types must implement
  /// the TransformHandleTypeInterface and the implementation must be already
  /// available when the type is injected. Furthermore, the types must provide
  /// a `getMnemonic` static method returning an object convertible to
  /// `StringRef` that is unique across all injected types.
  template <typename... TypeTys>
  void registerTypes() {
    initializers.push_back([](TransformDialect *transformDialect) {
      transformDialect->addTypesChecked<TypeTys...>();
    });
  }

  /// Declares that this Transform dialect extension depends on the dialect
  /// provided as template parameter. When the Transform dialect is loaded,
  /// dependent dialects will be loaded as well. This is intended for dialects
  /// that contain attributes and types used in creation and canonicalization of
  /// the injected operations, similarly to how the dialect definition may list
  /// dependent dialects. This is *not* intended for dialects entities from
  /// which may be produced when applying the transformations specified by ops
  /// registered by this extension.
  template <typename DialectTy>
  void declareDependentDialect() {
    dialectLoaders.push_back(
        [](MLIRContext *context) { context->loadDialect<DialectTy>(); });
  }
````
- **EN**: This block groups callable interfaces such as `init`, `registerTransformOps`, `push_back`, `registerAttributes`, indicating how `TransformDialect` is queried or updated.
- **CN**: 该代码块聚合了 `init`, `registerTransformOps`, `push_back`, `registerAttributes` 等可调用接口，展示了如何查询或更新 `TransformDialect`。

### Lines 233-276
````cpp
  /// Declares that the transformations associated with the operations
  /// registered by this dialect extension may produce operations from the
  /// dialect provided as template parameter while processing payload IR that
  /// does not contain the operations from said dialect. This is similar to
  /// dependent dialects of a pass. These dialects will be loaded along with the
  /// transform dialect unless the extension is in the build-only mode.
  template <typename DialectTy>
  void declareGeneratedDialect() {
    generatedDialectLoaders.push_back(
        [](MLIRContext *context) { context->loadDialect<DialectTy>(); });
  }

private:
  /// Callbacks performing extension initialization, e.g., registering ops,
  /// types and defining the additional data.
  SmallVector<Initializer> initializers;

  /// Callbacks loading the dependent dialects, i.e. the dialect needed for the
  /// extension ops.
  SmallVector<DialectLoader> dialectLoaders;

  /// Callbacks loading the generated dialects, i.e. the dialects produced when
  /// applying the transformations.
  SmallVector<DialectLoader> generatedDialectLoaders;

  /// Indicates that the extension is in build-only mode.
  bool buildOnly;
};

template <typename OpTy>
void TransformDialect::addOperationIfNotRegistered() {
  std::optional<RegisteredOperationName> opName =
      RegisteredOperationName::lookup(TypeID::get<OpTy>(), getContext());
  if (!opName) {
    addOperations<OpTy>();
#if LLVM_ENABLE_ABI_BREAKING_CHECKS
    StringRef name = OpTy::getOperationName();
    detail::checkImplementsTransformOpInterface(name, getContext());
#endif // LLVM_ENABLE_ABI_BREAKING_CHECKS
    return;
  }

  if (LLVM_LIKELY(opName->getTypeID() == TypeID::get<OpTy>()))
    return;
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 278-322
````cpp
  reportDuplicateOpRegistration(OpTy::getOperationName());
}

template <typename AttrTy>
void TransformDialect::addAttributeIfNotRegistered() {
  // Use the address of the parse method as a proxy for identifying whether we
  // are registering the same type class for the same mnemonic.
  StringRef mnemonic = AttrTy::getMnemonic();
  auto [it, inserted] =
      attributeParsingHooks.try_emplace(mnemonic, AttrTy::parse);
  if (!inserted) {
    const ExtensionAttributeParsingHook &parsingHook = it->getValue();
    if (parsingHook != &AttrTy::parse)
      reportDuplicateAttributeRegistration(mnemonic);
    else
      return;
  }
  attributePrintingHooks.try_emplace(
      TypeID::get<AttrTy>(),
      +[](mlir::Attribute attribute, AsmPrinter &printer) {
        printer << AttrTy::getMnemonic();
        cast<AttrTy>(attribute).print(printer);
      });
  addAttributes<AttrTy>();
}

template <typename Type>
void TransformDialect::addTypeIfNotRegistered() {
  // Use the address of the parse method as a proxy for identifying whether we
  // are registering the same type class for the same mnemonic.
  StringRef mnemonic = Type::getMnemonic();
  auto [it, inserted] = typeParsingHooks.try_emplace(mnemonic, Type::parse);
  if (!inserted) {
    const ExtensionTypeParsingHook &parsingHook = it->getValue();
    if (parsingHook != &Type::parse)
      reportDuplicateTypeRegistration(mnemonic);
    else
      return;
  }
  typePrintingHooks.try_emplace(
      TypeID::get<Type>(), +[](mlir::Type type, AsmPrinter &printer) {
        printer << Type::getMnemonic();
        cast<Type>(type).print(printer);
      });
  addTypes<Type>();
````
- **EN**: This block groups callable interfaces such as `reportDuplicateOpRegistration`, `getOperationName`, `addAttributeIfNotRegistered`, `getMnemonic`, indicating how `TransformDialect` is queried or updated.
- **CN**: 该代码块聚合了 `reportDuplicateOpRegistration`, `getOperationName`, `addAttributeIfNotRegistered`, `getMnemonic` 等可调用接口，展示了如何查询或更新 `TransformDialect`。

### Lines 324-348
````cpp
#if LLVM_ENABLE_ABI_BREAKING_CHECKS
  detail::checkImplementsTransformHandleTypeInterface(TypeID::get<Type>(),
                                                      getContext());
#endif // LLVM_ENABLE_ABI_BREAKING_CHECKS
}

template <typename DataTy>
DataTy &TransformDialect::getOrCreateExtraData() {
  TypeID typeID = TypeID::get<DataTy>();
  auto [it, inserted] = extraData.try_emplace(typeID);
  if (inserted)
    it->getSecond() = std::make_unique<DataTy>(getContext());
  return static_cast<DataTy &>(*it->getSecond());
}

/// A wrapper for transform dialect extensions that forces them to be
/// constructed in the build-only mode.
template <typename DerivedTy>
class BuildOnly : public DerivedTy {
public:
  BuildOnly() : DerivedTy(/*buildOnly=*/true) {}
};

} // namespace transform
} // namespace mlir
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 351-351
````cpp
#endif // MLIR_DIALECT_TRANSFORM_IR_TRANSFORMDIALECT_H
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

- mlir/IR/Dialect.h
- mlir/IR/PatternMatch.h
- mlir/Support/LLVM.h
- mlir/Support/TypeID.h
- llvm/ADT/DenseMap.h
- llvm/ADT/StringMap.h
- mlir/Dialect/Transform/IR/TransformDialect.h.inc
- TransformDialectData inherits from public detail::TransformDialectDataBase
- TransformDialectExtension inherits from public DialectExtension<DerivedTy, TransformDialect, ExtraDialects...>
- BuildOnly inherits from public DerivedTy
- TransformDialectData builds on public detail::TransformDialectDataBase
- TransformDialectExtension builds on public DialectExtension<DerivedTy, TransformDialect, ExtraDialects...>
- BuildOnly builds on public DerivedTy
