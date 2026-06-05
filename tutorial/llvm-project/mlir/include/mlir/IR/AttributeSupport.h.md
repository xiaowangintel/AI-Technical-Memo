# AttributeSupport.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AttributeSupport.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AttributeSupport component. The leading comments describe it as: This file defines support types for registering dialect extended attributes.
- **用途（CN）**: 声明 MLIR AttributeSupport 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
````cpp
//===- AttributeSupport.h ---------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines support types for registering dialect extended attributes.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_ATTRIBUTESUPPORT_H
#define MLIR_IR_ATTRIBUTESUPPORT_H

#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/StorageUniquerSupport.h"
#include "mlir/IR/Types.h"
#include "llvm/ADT/PointerIntPair.h"
#include "llvm/ADT/Twine.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 22-22
````cpp
namespace mlir {
````
- **EN**: This C++ declaration introduces `mlir` and establishes part of the API surface for `AttributeSupport`.
- **CN**: 该 C++ 声明引入了 `mlir`，并构成 `AttributeSupport` API 表面的一部分。

### Lines 23-69
````cpp
//===----------------------------------------------------------------------===//
// AbstractAttribute
//===----------------------------------------------------------------------===//

/// This class contains all of the static information common to all instances of
/// a registered Attribute.
class AbstractAttribute {
public:
  using HasTraitFn = llvm::unique_function<bool(TypeID) const>;
  using WalkImmediateSubElementsFn = function_ref<void(
      Attribute, function_ref<void(Attribute)>, function_ref<void(Type)>)>;
  using ReplaceImmediateSubElementsFn =
      function_ref<Attribute(Attribute, ArrayRef<Attribute>, ArrayRef<Type>)>;

  /// Look up the specified abstract attribute in the MLIRContext and return a
  /// reference to it.
  static const AbstractAttribute &lookup(TypeID typeID, MLIRContext *context);

  /// Look up the specified abstract attribute in the MLIRContext and return a
  /// reference to it if it exists.
  static std::optional<std::reference_wrapper<const AbstractAttribute>>
  lookup(StringRef name, MLIRContext *context);

  /// This method is used by Dialect objects when they register the list of
  /// attributes they contain.
  template <typename T>
  static AbstractAttribute get(Dialect &dialect) {
    return AbstractAttribute(dialect, T::getInterfaceMap(), T::getHasTraitFn(),
                             T::getWalkImmediateSubElementsFn(),
                             T::getReplaceImmediateSubElementsFn(),
                             T::getTypeID(), T::name);
  }

  /// This method is used by Dialect objects to register attributes with
  /// custom TypeIDs.
  /// The use of this method is in general discouraged in favor of
  /// 'get<CustomAttribute>(dialect)'.
  static AbstractAttribute
  get(Dialect &dialect, detail::InterfaceMap &&interfaceMap,
      HasTraitFn &&hasTrait,
      WalkImmediateSubElementsFn walkImmediateSubElementsFn,
      ReplaceImmediateSubElementsFn replaceImmediateSubElementsFn,
      TypeID typeID, StringRef name) {
    return AbstractAttribute(dialect, std::move(interfaceMap),
                             std::move(hasTrait), walkImmediateSubElementsFn,
                             replaceImmediateSubElementsFn, typeID, name);
  }
````
- **EN**: This C++ declaration introduces `AbstractAttribute` and establishes part of the API surface for `AttributeSupport`. Representative entry points here include `bool`, `void`, `Attribute`, `lookup`.
- **CN**: 该 C++ 声明引入了 `AbstractAttribute`，并构成 `AttributeSupport` API 表面的一部分。 这一段可见的代表性接口包括 `bool`, `void`, `Attribute`, `lookup`。

### Lines 70-122
````cpp
  /// Return the dialect this attribute was registered to.
  Dialect &getDialect() const { return const_cast<Dialect &>(dialect); }

  /// Returns an instance of the concept object for the given interface if it
  /// was registered to this attribute, null otherwise. This should not be used
  /// directly.
  template <typename T>
  typename T::Concept *getInterface() const {
    return interfaceMap.lookup<T>();
  }

  /// Returns true if the attribute has the interface with the given ID
  /// registered.
  bool hasInterface(TypeID interfaceID) const {
    return interfaceMap.contains(interfaceID);
  }

  /// Returns true if the attribute has a particular trait.
  template <template <typename T> class Trait>
  bool hasTrait() const {
    return hasTraitFn(TypeID::get<Trait>());
  }

  /// Returns true if the attribute has a particular trait.
  bool hasTrait(TypeID traitID) const { return hasTraitFn(traitID); }

  /// Walk the immediate sub-elements of this attribute.
  void walkImmediateSubElements(Attribute attr,
                                function_ref<void(Attribute)> walkAttrsFn,
                                function_ref<void(Type)> walkTypesFn) const;

  /// Replace the immediate sub-elements of this attribute.
  Attribute replaceImmediateSubElements(Attribute attr,
                                        ArrayRef<Attribute> replAttrs,
                                        ArrayRef<Type> replTypes) const;

  /// Return the unique identifier representing the concrete attribute class.
  TypeID getTypeID() const { return typeID; }

  /// Return the unique name representing the type.
  StringRef getName() const { return name; }

private:
  AbstractAttribute(Dialect &dialect, detail::InterfaceMap &&interfaceMap,
                    HasTraitFn &&hasTraitFn,
                    WalkImmediateSubElementsFn walkImmediateSubElementsFn,
                    ReplaceImmediateSubElementsFn replaceImmediateSubElementsFn,
                    TypeID typeID, StringRef name)
      : dialect(dialect), interfaceMap(std::move(interfaceMap)),
        hasTraitFn(std::move(hasTraitFn)),
        walkImmediateSubElementsFn(walkImmediateSubElementsFn),
        replaceImmediateSubElementsFn(replaceImmediateSubElementsFn),
        typeID(typeID), name(name) {}
````
- **EN**: This block groups callable interfaces such as `getDialect`, `getInterface`, `hasInterface`, `contains`, indicating how `AttributeSupport` is queried or updated.
- **CN**: 该代码块聚合了 `getDialect`, `getInterface`, `hasInterface`, `contains` 等可调用接口，展示了如何查询或更新 `AttributeSupport`。

### Lines 124-155
````cpp
  /// Give StorageUserBase access to the mutable lookup.
  template <typename ConcreteT, typename BaseT, typename StorageT,
            typename UniquerT, template <typename T> class... Traits>
  friend class detail::StorageUserBase;

  /// Look up the specified abstract attribute in the MLIRContext and return a
  /// (mutable) pointer to it. Return a null pointer if the attribute could not
  /// be found in the context.
  static AbstractAttribute *lookupMutable(TypeID typeID, MLIRContext *context);

  /// This is the dialect that this attribute was registered to.
  const Dialect &dialect;

  /// This is a collection of the interfaces registered to this attribute.
  detail::InterfaceMap interfaceMap;

  /// Function to check if the attribute has a particular trait.
  HasTraitFn hasTraitFn;

  /// Function to walk the immediate sub-elements of this attribute.
  WalkImmediateSubElementsFn walkImmediateSubElementsFn;

  /// Function to replace the immediate sub-elements of this attribute.
  ReplaceImmediateSubElementsFn replaceImmediateSubElementsFn;

  /// The unique identifier of the derived Attribute class.
  const TypeID typeID;

  /// The unique name of this attribute. The string is not owned by the context,
  /// so the lifetime of this string should outlive the MLIR context.
  const StringRef name;
};
````
- **EN**: This block groups callable interfaces such as `lookupMutable`, indicating how `AttributeSupport` is queried or updated.
- **CN**: 该代码块聚合了 `lookupMutable` 等可调用接口，展示了如何查询或更新 `AttributeSupport`。

### Lines 158-199
````cpp
//===----------------------------------------------------------------------===//
// AttributeStorage
//===----------------------------------------------------------------------===//

namespace detail {
class AttributeUniquer;
class DistinctAttributeUniquer;
} // namespace detail

/// Base storage class appearing in an attribute. Derived storage classes should
/// only be constructed within the context of the AttributeUniquer.
class alignas(8) AttributeStorage : public StorageUniquer::BaseStorage {
  friend detail::AttributeUniquer;
  friend detail::DistinctAttributeUniquer;
  friend StorageUniquer;

public:
  /// Return the abstract descriptor for this attribute.
  const AbstractAttribute &getAbstractAttribute() const {
    assert(abstractAttribute && "Malformed attribute storage object.");
    return *abstractAttribute;
  }

protected:
  /// Set the abstract attribute for this storage instance. This is used by the
  /// AttributeUniquer when initializing a newly constructed storage object.
  void initializeAbstractAttribute(const AbstractAttribute &abstractAttr) {
    abstractAttribute = &abstractAttr;
  }

  /// Default initialization for attribute storage classes that require no
  /// additional initialization.
  void initialize(MLIRContext *context) {}

private:
  /// The abstract descriptor for this attribute.
  const AbstractAttribute *abstractAttribute = nullptr;
};

/// Default storage type for attributes that require no additional
/// initialization or storage.
using DefaultAttributeStorage = AttributeStorage;
````
- **EN**: This C++ declaration introduces `AttributeUniquer` and establishes part of the API surface for `AttributeSupport`. Representative entry points here include `getAbstractAttribute`, `initializeAbstractAttribute`, `initialize`.
- **CN**: 该 C++ 声明引入了 `AttributeUniquer`，并构成 `AttributeSupport` API 表面的一部分。 这一段可见的代表性接口包括 `getAbstractAttribute`, `initializeAbstractAttribute`, `initialize`。

### Lines 201-207
````cpp
//===----------------------------------------------------------------------===//
// AttributeStorageAllocator
//===----------------------------------------------------------------------===//

// This is a utility allocator used to allocate memory for instances of derived
// Attributes.
using AttributeStorageAllocator = StorageUniquer::StorageAllocator;
````
- **EN**: This section focuses on attributestorageallocator, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“AttributeStorageAllocator”这一主题，把相关声明与辅助接口组织在一起。

### Lines 209-265
````cpp
//===----------------------------------------------------------------------===//
// AttributeUniquer
//===----------------------------------------------------------------------===//
namespace detail {
// A utility class to get, or create, unique instances of attributes within an
// MLIRContext. This class manages all creation and uniquing of attributes.
class AttributeUniquer {
public:
  /// Get an uniqued instance of an attribute T.
  template <typename T, typename... Args>
  static T get(MLIRContext *ctx, Args &&...args) {
    return getWithTypeID<T, Args...>(ctx, T::getTypeID(),
                                     std::forward<Args>(args)...);
  }

  /// Get an uniqued instance of a parametric attribute T.
  /// The use of this method is in general discouraged in favor of
  /// 'get<T, Args>(ctx, args)'.
  template <typename T, typename... Args>
  static std::enable_if_t<
      !std::is_same<typename T::ImplType, AttributeStorage>::value, T>
  getWithTypeID(MLIRContext *ctx, TypeID typeID, Args &&...args) {
#ifndef NDEBUG
    if (!ctx->getAttributeUniquer().isParametricStorageInitialized(typeID))
      llvm::report_fatal_error(
          llvm::Twine("can't create Attribute '") + llvm::getTypeName<T>() +
          "' because storage uniquer isn't initialized: the dialect was likely "
          "not loaded, or the attribute wasn't added with addAttributes<...>() "
          "in the Dialect::initialize() method.");
#endif
    return ctx->getAttributeUniquer().get<typename T::ImplType>(
        [typeID, ctx](AttributeStorage *storage) {
          initializeAttributeStorage(storage, ctx, typeID);

          // Execute any additional attribute storage initialization with the
          // context.
          static_cast<typename T::ImplType *>(storage)->initialize(ctx);
        },
        typeID, std::forward<Args>(args)...);
  }
  /// Get an uniqued instance of a singleton attribute T.
  /// The use of this method is in general discouraged in favor of
  /// 'get<T, Args>(ctx, args)'.
  template <typename T>
  static std::enable_if_t<
      std::is_same<typename T::ImplType, AttributeStorage>::value, T>
  getWithTypeID(MLIRContext *ctx, TypeID typeID) {
#ifndef NDEBUG
    if (!ctx->getAttributeUniquer().isSingletonStorageInitialized(typeID))
      llvm::report_fatal_error(
          llvm::Twine("can't create Attribute '") + llvm::getTypeName<T>() +
          "' because storage uniquer isn't initialized: the dialect was likely "
          "not loaded, or the attribute wasn't added with addAttributes<...>() "
          "in the Dialect::initialize() method.");
#endif
    return ctx->getAttributeUniquer().get<typename T::ImplType>(typeID);
  }
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 266-320
````cpp
  template <typename T, typename... Args>
  static LogicalResult mutate(MLIRContext *ctx, typename T::ImplType *impl,
                              Args &&...args) {
    assert(impl && "cannot mutate null attribute");
    return ctx->getAttributeUniquer().mutate(T::getTypeID(), impl,
                                             std::forward<Args>(args)...);
  }

  /// Register an attribute instance T with the uniquer.
  template <typename T>
  static void registerAttribute(MLIRContext *ctx) {
    registerAttribute<T>(ctx, T::getTypeID());
  }

  /// Register a parametric attribute instance T with the uniquer.
  /// The use of this method is in general discouraged in favor of
  /// 'registerAttribute<T>(ctx)'.
  template <typename T>
  static std::enable_if_t<
      !std::is_same<typename T::ImplType, AttributeStorage>::value>
  registerAttribute(MLIRContext *ctx, TypeID typeID) {
    ctx->getAttributeUniquer()
        .registerParametricStorageType<typename T::ImplType>(typeID);
  }
  /// Register a singleton attribute instance T with the uniquer.
  /// The use of this method is in general discouraged in favor of
  /// 'registerAttribute<T>(ctx)'.
  template <typename T>
  static std::enable_if_t<
      std::is_same<typename T::ImplType, AttributeStorage>::value>
  registerAttribute(MLIRContext *ctx, TypeID typeID) {
    ctx->getAttributeUniquer()
        .registerSingletonStorageType<typename T::ImplType>(
            typeID, [ctx, typeID](AttributeStorage *storage) {
              initializeAttributeStorage(storage, ctx, typeID);
            });
  }

private:
  /// Initialize the given attribute storage instance.
  static void initializeAttributeStorage(AttributeStorage *storage,
                                         MLIRContext *ctx, TypeID attrID);
};

// Internal function called by ODS generated code.
// Default initializes the type within a FailureOr<T> if T is default
// constructible and returns a reference to the instance.
// Otherwise, returns a reference to the FailureOr<T>.
template <class T>
decltype(auto) unwrapForCustomParse(FailureOr<T> &failureOr) {
  if constexpr (std::is_default_constructible_v<T>)
    return failureOr.emplace();
  else
    return failureOr;
}
````
- **EN**: This block groups callable interfaces such as `mutate`, `getAttributeUniquer`, `getTypeID`, `registerAttribute`, indicating how `AttributeSupport` is queried or updated.
- **CN**: 该代码块聚合了 `mutate`, `getAttributeUniquer`, `getTypeID`, `registerAttribute` 等可调用接口，展示了如何查询或更新 `AttributeSupport`。

### Lines 322-324
````cpp
} // namespace detail

} // namespace mlir
````
- **EN**: This section focuses on } // namespace detail, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“} // namespace detail”这一主题，把相关声明与辅助接口组织在一起。

### Lines 327-327
````cpp
#endif
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

- mlir/IR/MLIRContext.h
- mlir/IR/StorageUniquerSupport.h
- mlir/IR/Types.h
- llvm/ADT/PointerIntPair.h
- llvm/ADT/Twine.h
