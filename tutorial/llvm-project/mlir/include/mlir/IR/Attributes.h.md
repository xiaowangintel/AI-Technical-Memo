# Attributes.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/Attributes.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR Attributes component. The leading comments describe it as: Attributes are known-constant values of operations.
- **用途（CN）**: 声明 MLIR Attributes 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13
````cpp
//===- Attributes.h - MLIR Attribute Classes --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_ATTRIBUTES_H
#define MLIR_IR_ATTRIBUTES_H

#include "mlir/IR/AttributeSupport.h"
#include "llvm/Support/PointerLikeTypeTraits.h"
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 15-60
````cpp
namespace mlir {
class AsmState;
class StringAttr;

/// Attributes are known-constant values of operations.
///
/// Instances of the Attribute class are references to immortal key-value pairs
/// with immutable, uniqued keys owned by MLIRContext. As such, an Attribute is
/// a thin wrapper around an underlying storage pointer. Attributes are usually
/// passed by value.
class Attribute {
public:
  /// Utility class for implementing attributes.
  template <typename ConcreteType, typename BaseType, typename StorageType,
            template <typename T> class... Traits>
  using AttrBase = detail::StorageUserBase<ConcreteType, BaseType, StorageType,
                                           detail::AttributeUniquer, Traits...>;

  using ImplType = AttributeStorage;
  using ValueType = void;
  using AbstractTy = AbstractAttribute;

  constexpr Attribute() = default;
  /* implicit */ Attribute(const ImplType *impl)
      : impl(const_cast<ImplType *>(impl)) {}

  Attribute(const Attribute &other) = default;
  Attribute &operator=(const Attribute &other) = default;

  bool operator==(Attribute other) const { return impl == other.impl; }
  bool operator!=(Attribute other) const { return !(*this == other); }
  explicit operator bool() const { return impl; }

  bool operator!() const { return impl == nullptr; }

  /// Return a unique identifier for the concrete attribute type. This is used
  /// to support dynamic type casting.
  TypeID getTypeID() { return impl->getAbstractAttribute().getTypeID(); }

  /// Return the context this attribute belongs to.
  MLIRContext *getContext() const;

  /// Get the dialect this attribute is registered to.
  Dialect &getDialect() const {
    return impl->getAbstractAttribute().getDialect();
  }
````
- **EN**: This C++ declaration introduces `AsmState` and establishes part of the API surface for `Attributes`. Representative entry points here include `Attribute`, `impl`, `bool`, `getTypeID`.
- **CN**: 该 C++ 声明引入了 `AsmState`，并构成 `Attributes` API 表面的一部分。 这一段可见的代表性接口包括 `Attribute`, `impl`, `bool`, `getTypeID`。

### Lines 61-106
````cpp
  /// Print the attribute. If `elideType` is set, the attribute is printed
  /// without a trailing colon type if it has one.
  void print(raw_ostream &os, bool elideType = false) const;
  void print(raw_ostream &os, AsmState &state, bool elideType = false) const;
  void dump() const;

  /// Print the attribute without dialect wrapping.
  void printStripped(raw_ostream &os) const;
  void printStripped(raw_ostream &os, AsmState &state) const;

  /// Get an opaque pointer to the attribute.
  const void *getAsOpaquePointer() const { return impl; }
  /// Construct an attribute from the opaque pointer representation.
  static Attribute getFromOpaquePointer(const void *ptr) {
    return Attribute(reinterpret_cast<const ImplType *>(ptr));
  }

  friend ::llvm::hash_code hash_value(Attribute arg);

  /// Returns true if `InterfaceT` has been promised by the dialect or
  /// implemented.
  template <typename InterfaceT>
  bool hasPromiseOrImplementsInterface() {
    return dialect_extension_detail::hasPromisedInterface(
               getDialect(), getTypeID(), InterfaceT::getInterfaceID()) ||
           mlir::isa<InterfaceT>(*this);
  }

  /// Returns true if the type was registered with a particular trait.
  template <template <typename T> class Trait>
  bool hasTrait() {
    return getAbstractAttribute().hasTrait<Trait>();
  }

  /// Return the abstract descriptor for this attribute.
  const AbstractTy &getAbstractAttribute() const {
    return impl->getAbstractAttribute();
  }

  /// Walk all of the immediately nested sub-attributes and sub-types. This
  /// method does not recurse into sub elements.
  void walkImmediateSubElements(function_ref<void(Attribute)> walkAttrsFn,
                                function_ref<void(Type)> walkTypesFn) const {
    getAbstractAttribute().walkImmediateSubElements(*this, walkAttrsFn,
                                                    walkTypesFn);
  }
````
- **EN**: This block groups callable interfaces such as `print`, `dump`, `printStripped`, `getAsOpaquePointer`, indicating how `Attributes` is queried or updated.
- **CN**: 该代码块聚合了 `print`, `dump`, `printStripped`, `getAsOpaquePointer` 等可调用接口，展示了如何查询或更新 `Attributes`。

### Lines 108-152
````cpp
  /// Replace the immediately nested sub-attributes and sub-types with those
  /// provided. The order of the provided elements is derived from the order of
  /// the elements returned by the callbacks of `walkImmediateSubElements`. The
  /// element at index 0 would replace the very first attribute given by
  /// `walkImmediateSubElements`. On success, the new instance with the values
  /// replaced is returned. If replacement fails, nullptr is returned.
  auto replaceImmediateSubElements(ArrayRef<Attribute> replAttrs,
                                   ArrayRef<Type> replTypes) const {
    return getAbstractAttribute().replaceImmediateSubElements(*this, replAttrs,
                                                              replTypes);
  }

  /// Walk this attribute and all attibutes/types nested within using the
  /// provided walk functions. See `AttrTypeWalker` for information on the
  /// supported walk function types.
  template <WalkOrder Order = WalkOrder::PostOrder, typename... WalkFns>
  auto walk(WalkFns &&...walkFns) {
    AttrTypeWalker walker;
    (walker.addWalk(std::forward<WalkFns>(walkFns)), ...);
    return walker.walk<Order>(*this);
  }

  /// Recursively replace all of the nested sub-attributes and sub-types using
  /// the provided map functions. Returns nullptr in the case of failure. See
  /// `AttrTypeReplacer` for information on the support replacement function
  /// types.
  template <typename... ReplacementFns>
  auto replace(ReplacementFns &&...replacementFns) {
    AttrTypeReplacer replacer;
    (replacer.addReplacement(std::forward<ReplacementFns>(replacementFns)),
     ...);
    return replacer.replace(*this);
  }

  /// Return the internal Attribute implementation.
  ImplType *getImpl() const { return impl; }

protected:
  ImplType *impl{nullptr};
};

inline raw_ostream &operator<<(raw_ostream &os, Attribute attr) {
  attr.print(os);
  return os;
}
````
- **EN**: This block groups callable interfaces such as `replaceImmediateSubElements`, `getAbstractAttribute`, `walk`, `addWalk`, indicating how `Attributes` is queried or updated.
- **CN**: 该代码块聚合了 `replaceImmediateSubElements`, `getAbstractAttribute`, `walk`, `addWalk` 等可调用接口，展示了如何查询或更新 `Attributes`。

### Lines 154-156
````cpp
inline ::llvm::hash_code hash_value(Attribute arg) {
  return DenseMapInfo<const Attribute::ImplType *>::getHashValue(arg.impl);
}
````
- **EN**: This block groups callable interfaces such as `hash_value`, `getHashValue`, indicating how `Attributes` is queried or updated.
- **CN**: 该代码块聚合了 `hash_value`, `getHashValue` 等可调用接口，展示了如何查询或更新 `Attributes`。

### Lines 159-205
````cpp
//===----------------------------------------------------------------------===//
// NamedAttribute
//===----------------------------------------------------------------------===//

/// NamedAttribute represents a combination of a name and an Attribute value.
class NamedAttribute {
public:
  NamedAttribute(StringAttr name, Attribute value);
  NamedAttribute(StringRef name, Attribute value);

  /// Return the name of the attribute.
  StringAttr getName() const;

  /// Return the dialect of the name of this attribute, if the name is prefixed
  /// by a dialect namespace. For example, `llvm.fast_math` would return the
  /// LLVM dialect (if it is loaded). Returns nullptr if the dialect isn't
  /// loaded, or if the name is not prefixed by a dialect namespace.
  Dialect *getNameDialect() const;

  /// Return the value of the attribute.
  Attribute getValue() const { return value; }

  /// Set the name of this attribute.
  void setName(StringAttr newName);

  /// Set the value of this attribute.
  void setValue(Attribute newValue) {
    assert(value && "expected valid attribute value");
    value = newValue;
  }

  /// Compare this attribute to the provided attribute, ordering by name.
  bool operator<(const NamedAttribute &rhs) const;
  /// Compare this attribute to the provided string, ordering by name.
  bool operator<(StringRef rhs) const;

  bool operator==(const NamedAttribute &rhs) const {
    return name == rhs.name && value == rhs.value;
  }
  bool operator!=(const NamedAttribute &rhs) const { return !(*this == rhs); }

private:
  NamedAttribute(Attribute name, Attribute value) : name(name), value(value) {}

  /// Allow access to internals to enable hashing.
  friend ::llvm::hash_code hash_value(const NamedAttribute &arg);
  friend DenseMapInfo<NamedAttribute>;
````
- **EN**: This C++ declaration introduces `NamedAttribute` and establishes part of the API surface for `Attributes`. Representative entry points here include `NamedAttribute`, `getName`, `getNameDialect`, `getValue`.
- **CN**: 该 C++ 声明引入了 `NamedAttribute`，并构成 `Attributes` API 表面的一部分。 这一段可见的代表性接口包括 `NamedAttribute`, `getName`, `getNameDialect`, `getValue`。

### Lines 206-232
````cpp
  /// The name of the attribute. This is represented as a StringAttr, but
  /// type-erased to Attribute in the field.
  Attribute name;
  /// The value of the attribute.
  Attribute value;
};

inline ::llvm::hash_code hash_value(const NamedAttribute &arg) {
  using AttrPairT = std::pair<Attribute, Attribute>;
  return DenseMapInfo<AttrPairT>::getHashValue(AttrPairT(arg.name, arg.value));
}

/// Allow walking and replacing the subelements of a NamedAttribute.
template <>
struct AttrTypeSubElementHandler<NamedAttribute> {
  template <typename T>
  static void walk(T param, AttrTypeImmediateSubElementWalker &walker) {
    walker.walk(param.getName());
    walker.walk(param.getValue());
  }
  template <typename T>
  static T replace(T param, AttrSubElementReplacements &attrRepls,
                   TypeSubElementReplacements &typeRepls) {
    ArrayRef<Attribute> paramRepls = attrRepls.take_front(2);
    return T(cast<decltype(param.getName())>(paramRepls[0]), paramRepls[1]);
  }
};
````
- **EN**: This C++ declaration introduces `AttrTypeSubElementHandler` and establishes part of the API surface for `Attributes`. Representative entry points here include `hash_value`, `getHashValue`, `AttrPairT`, `walk`.
- **CN**: 该 C++ 声明引入了 `AttrTypeSubElementHandler`，并构成 `Attributes` API 表面的一部分。 这一段可见的代表性接口包括 `hash_value`, `getHashValue`, `AttrPairT`, `walk`。

### Lines 235-243
````cpp
//===----------------------------------------------------------------------===//
// AttributeTraitBase
//===----------------------------------------------------------------------===//

namespace AttributeTrait {
/// This class represents the base of an attribute trait.
template <typename ConcreteType, template <typename> class TraitType>
using TraitBase = detail::StorageUserTraitBase<ConcreteType, TraitType>;
} // namespace AttributeTrait
````
- **EN**: This C++ declaration introduces `AttributeTrait` and establishes part of the API surface for `Attributes`.
- **CN**: 该 C++ 声明引入了 `AttributeTrait`，并构成 `Attributes` API 表面的一部分。

### Lines 245-277
````cpp
//===----------------------------------------------------------------------===//
// AttributeInterface
//===----------------------------------------------------------------------===//

/// This class represents the base of an attribute interface. See the definition
/// of `detail::Interface` for requirements on the `Traits` type.
template <typename ConcreteType, typename Traits>
class AttributeInterface
    : public detail::Interface<ConcreteType, Attribute, Traits, Attribute,
                               AttributeTrait::TraitBase> {
public:
  using Base = AttributeInterface<ConcreteType, Traits>;
  using InterfaceBase = detail::Interface<ConcreteType, Attribute, Traits,
                                          Attribute, AttributeTrait::TraitBase>;
  using InterfaceBase::InterfaceBase;

protected:
  /// Returns the impl interface instance for the given type.
  static typename InterfaceBase::Concept *getInterfaceFor(Attribute attr) {
#ifndef NDEBUG
    // Check that the current interface isn't an unresolved promise for the
    // given attribute.
    dialect_extension_detail::handleUseOfUndefinedPromisedInterface(
        attr.getDialect(), attr.getTypeID(), ConcreteType::getInterfaceID(),
        llvm::getTypeName<ConcreteType>());
#endif

    return attr.getAbstractAttribute().getInterface<ConcreteType>();
  }

  /// Allow access to 'getInterfaceFor'.
  friend InterfaceBase;
};
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

### Lines 279-298
````cpp
//===----------------------------------------------------------------------===//
// Core AttributeTrait
//===----------------------------------------------------------------------===//

namespace AttributeTrait {
/// This trait is used to determine if an attribute is mutable or not. It is
/// attached on an attribute if the corresponding ConcreteType defines a
/// `mutate` function with proper signature.
template <typename ConcreteType>
using IsMutable = detail::StorageUserTrait::IsMutable<ConcreteType>;

/// This trait is used to determine if an attribute is a location or not. It is
/// attached to an attribute by the user if they intend the attribute to be used
/// as a location.
template <typename ConcreteType>
struct IsLocation : public AttributeTrait::TraitBase<ConcreteType, IsLocation> {
};
} // namespace AttributeTrait

} // namespace mlir.
````
- **EN**: This C++ declaration introduces `IsLocation` and establishes part of the API surface for `Attributes`.
- **CN**: 该 C++ 声明引入了 `IsLocation`，并构成 `Attributes` API 表面的一部分。

### Lines 300-346
````cpp
namespace llvm {

// Attribute hash just like pointers.
template <>
struct DenseMapInfo<mlir::Attribute> {
  static mlir::Attribute getEmptyKey() {
    auto *pointer = llvm::DenseMapInfo<void *>::getEmptyKey();
    return mlir::Attribute(static_cast<mlir::Attribute::ImplType *>(pointer));
  }
  static mlir::Attribute getTombstoneKey() {
    auto *pointer = llvm::DenseMapInfo<void *>::getTombstoneKey();
    return mlir::Attribute(static_cast<mlir::Attribute::ImplType *>(pointer));
  }
  static unsigned getHashValue(mlir::Attribute val) {
    return mlir::hash_value(val);
  }
  static bool isEqual(mlir::Attribute LHS, mlir::Attribute RHS) {
    return LHS == RHS;
  }
};
template <typename T>
struct DenseMapInfo<
    T, std::enable_if_t<std::is_base_of<mlir::Attribute, T>::value &&
                        !mlir::detail::IsInterface<T>::value>>
    : public DenseMapInfo<mlir::Attribute> {
  static T getEmptyKey() {
    const void *pointer = llvm::DenseMapInfo<const void *>::getEmptyKey();
    return T::getFromOpaquePointer(pointer);
  }
  static T getTombstoneKey() {
    const void *pointer = llvm::DenseMapInfo<const void *>::getTombstoneKey();
    return T::getFromOpaquePointer(pointer);
  }
};

/// Allow LLVM to steal the low bits of Attributes.
template <>
struct PointerLikeTypeTraits<mlir::Attribute> {
  static inline void *getAsVoidPointer(mlir::Attribute attr) {
    return const_cast<void *>(attr.getAsOpaquePointer());
  }
  static inline mlir::Attribute getFromVoidPointer(void *ptr) {
    return mlir::Attribute::getFromOpaquePointer(ptr);
  }
  static constexpr int NumLowBitsAvailable = llvm::PointerLikeTypeTraits<
      mlir::AttributeStorage *>::NumLowBitsAvailable;
};
````
- **EN**: This C++ declaration introduces `DenseMapInfo` and establishes part of the API surface for `Attributes`. Representative entry points here include `getEmptyKey`, `Attribute`, `getTombstoneKey`, `getHashValue`.
- **CN**: 该 C++ 声明引入了 `DenseMapInfo`，并构成 `Attributes` API 表面的一部分。 这一段可见的代表性接口包括 `getEmptyKey`, `Attribute`, `getTombstoneKey`, `getHashValue`。

### Lines 347-391
````cpp
template <>
struct DenseMapInfo<mlir::NamedAttribute> {
  static mlir::NamedAttribute getEmptyKey() {
    auto emptyAttr = llvm::DenseMapInfo<mlir::Attribute>::getEmptyKey();
    return mlir::NamedAttribute(emptyAttr, emptyAttr);
  }
  static mlir::NamedAttribute getTombstoneKey() {
    auto tombAttr = llvm::DenseMapInfo<mlir::Attribute>::getTombstoneKey();
    return mlir::NamedAttribute(tombAttr, tombAttr);
  }
  static unsigned getHashValue(mlir::NamedAttribute val) {
    return mlir::hash_value(val);
  }
  static bool isEqual(mlir::NamedAttribute lhs, mlir::NamedAttribute rhs) {
    return lhs == rhs;
  }
};

/// Add support for llvm style casts. We provide a cast between To and From if
/// From is mlir::Attribute or derives from it.
template <typename To, typename From>
struct CastInfo<To, From,
                std::enable_if_t<std::is_same_v<mlir::Attribute,
                                                std::remove_const_t<From>> ||
                                 std::is_base_of_v<mlir::Attribute, From>>>
    : NullableValueCastFailed<To>,
      DefaultDoCastIfPossible<To, From, CastInfo<To, From>> {
  /// Arguments are taken as mlir::Attribute here and not as `From`, because
  /// when casting from an intermediate type of the hierarchy to one of its
  /// children, the val.getTypeID() inside T::classof will use the static
  /// getTypeID of the parent instead of the non-static Type::getTypeID that
  /// returns the dynamic ID. This means that T::classof would end up comparing
  /// the static TypeID of the children to the static TypeID of its parent,
  /// making it impossible to downcast from the parent to the child.
  static inline bool isPossible(mlir::Attribute ty) {
    /// Return a constant true instead of a dynamic true when casting to self or
    /// up the hierarchy.
    if constexpr (std::is_base_of_v<To, From>) {
      return true;
    } else {
      return To::classof(ty);
    }
  }
  static inline To doCast(mlir::Attribute attr) { return To(attr.getImpl()); }
};
````
- **EN**: This C++ declaration introduces `DenseMapInfo` and establishes part of the API surface for `Attributes`. Representative entry points here include `getEmptyKey`, `NamedAttribute`, `getTombstoneKey`, `getHashValue`.
- **CN**: 该 C++ 声明引入了 `DenseMapInfo`，并构成 `Attributes` API 表面的一部分。 这一段可见的代表性接口包括 `getEmptyKey`, `NamedAttribute`, `getTombstoneKey`, `getHashValue`。

### Lines 393-393
````cpp
} // namespace llvm
````
- **EN**: This section focuses on } // namespace llvm, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“} // namespace llvm”这一主题，把相关声明与辅助接口组织在一起。

### Lines 396-396
````cpp
#endif
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/AttributeSupport.h
- llvm/Support/PointerLikeTypeTraits.h
- AttributeInterface inherits from public detail::Interface<ConcreteType, Attribute, Traits, Attribute, AttributeTrait::TraitBase>
- AttributeInterface builds on public detail::Interface<ConcreteType, Attribute, Traits, Attribute,
