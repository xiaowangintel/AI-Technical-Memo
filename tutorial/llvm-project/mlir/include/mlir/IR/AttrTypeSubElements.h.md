# AttrTypeSubElements.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AttrTypeSubElements.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AttrTypeSubElements component. The leading comments describe it as: This file contains utilities for querying the sub elements of an attribute or.
- **用途（CN）**: 声明 MLIR AttrTypeSubElements 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
````cpp
//===- AttrTypeSubElements.h - Attr and Type SubElements -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains utilities for querying the sub elements of an attribute or
// type.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_ATTRTYPESUBELEMENTS_H
#define MLIR_IR_ATTRTYPESUBELEMENTS_H

#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/Visitors.h"
#include "mlir/Support/CyclicReplacerCache.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include <optional>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 24-26
````cpp
namespace mlir {
class Attribute;
class Type;
````
- **EN**: This C++ declaration introduces `Attribute` and establishes part of the API surface for `AttrTypeSubElements`.
- **CN**: 该 C++ 声明引入了 `Attribute`，并构成 `AttrTypeSubElements` API 表面的一部分。

### Lines 28-35
````cpp
//===----------------------------------------------------------------------===//
/// AttrTypeWalker
//===----------------------------------------------------------------------===//

/// This class provides a utility for walking attributes/types, and their sub
/// elements. Multiple walk functions may be registered.
class AttrTypeWalker {
public:
````
- **EN**: This C++ declaration introduces `AttrTypeWalker` and establishes part of the API surface for `AttrTypeSubElements`.
- **CN**: 该 C++ 声明引入了 `AttrTypeWalker`，并构成 `AttrTypeSubElements` API 表面的一部分。

### Lines 36-48
````cpp
  //===--------------------------------------------------------------------===//
  // Application
  //===--------------------------------------------------------------------===//

  /// Walk the given attribute/type, and recursively walk any sub elements.
  template <WalkOrder Order, typename T>
  WalkResult walk(T element) {
    return walkImpl(element, Order);
  }
  template <typename T>
  WalkResult walk(T element) {
    return walk<WalkOrder::PostOrder, T>(element);
  }
````
- **EN**: This block groups callable interfaces such as `walk`, `walkImpl`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `walk`, `walkImpl` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 50-94
````cpp
  //===--------------------------------------------------------------------===//
  // Registration
  //===--------------------------------------------------------------------===//

  template <typename T>
  using WalkFn = std::function<WalkResult(T)>;

  /// Register a walk function for a given attribute or type. A walk function
  /// must be convertible to any of the following forms(where `T` is a class
  /// derived from `Type` or `Attribute`:
  ///
  ///   * WalkResult(T)
  ///     - Returns a walk result, which can be used to control the walk
  ///
  ///   * void(T)
  ///     - Returns void, i.e. the walk always continues.
  ///
  /// Note: When walking, the mostly recently added walk functions will be
  ///       invoked first.
  void addWalk(WalkFn<Attribute> &&fn) {
    attrWalkFns.emplace_back(std::move(fn));
  }
  void addWalk(WalkFn<Type> &&fn) { typeWalkFns.push_back(std::move(fn)); }

  /// Register a replacement function that doesn't match the default signature,
  /// either because it uses a derived parameter type, or it uses a simplified
  /// result type.
  template <typename FnT,
            typename T = typename llvm::function_traits<
                std::decay_t<FnT>>::template arg_t<0>,
            typename BaseT = std::conditional_t<std::is_base_of_v<Attribute, T>,
                                                Attribute, Type>,
            typename ResultT = std::invoke_result_t<FnT, T>>
  std::enable_if_t<!std::is_same_v<T, BaseT> || std::is_same_v<ResultT, void>>
  addWalk(FnT &&callback) {
    addWalk([callback = std::forward<FnT>(callback)](BaseT base) -> WalkResult {
      if (auto derived = dyn_cast<T>(base)) {
        if constexpr (std::is_convertible_v<ResultT, WalkResult>)
          return callback(derived);
        else
          callback(derived);
      }
      return WalkResult::advance();
    });
  }
````
- **EN**: This block groups callable interfaces such as `WalkResult`, `addWalk`, `emplace_back`, `move`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `WalkResult`, `addWalk`, `emplace_back`, `move` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 95-113
````cpp
private:
  WalkResult walkImpl(Attribute attr, WalkOrder order);
  WalkResult walkImpl(Type type, WalkOrder order);

  /// Internal implementation of the `walk` methods above.
  template <typename T, typename WalkFns>
  WalkResult walkImpl(T element, WalkFns &walkFns, WalkOrder order);

  /// Walk the sub elements of the given interface.
  template <typename T>
  WalkResult walkSubElements(T interface, WalkOrder order);

  /// The set of walk functions that map sub elements.
  std::vector<WalkFn<Attribute>> attrWalkFns;
  std::vector<WalkFn<Type>> typeWalkFns;

  /// The set of visited attributes/types.
  DenseMap<std::pair<const void *, int>, WalkResult> visitedAttrTypes;
};
````
- **EN**: This block groups callable interfaces such as `walkImpl`, `walkSubElements`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `walkImpl`, `walkSubElements` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 116-135
````cpp
//===----------------------------------------------------------------------===//
/// AttrTypeReplacer
//===----------------------------------------------------------------------===//

namespace detail {

/// This class provides a base utility for replacing attributes/types, and their
/// sub elements. Multiple replacement functions may be registered.
///
/// This base utility is uncached. Users can choose between two cached versions
/// of this replacer:
///   * For non-cyclic replacer logic, use `AttrTypeReplacer`.
///   * For cyclic replacer logic, use `CyclicAttrTypeReplacer`.
///
/// Concrete implementations implement the following `replace` entry functions:
///   * Attribute replace(Attribute attr);
///   * Type replace(Type type);
template <typename Concrete>
class AttrTypeReplacerBase {
public:
````
- **EN**: This C++ declaration introduces `AttrTypeReplacerBase` and establishes part of the API surface for `AttrTypeSubElements`.
- **CN**: 该 C++ 声明引入了 `AttrTypeReplacerBase`，并构成 `AttrTypeSubElements` API 表面的一部分。

### Lines 136-153
````cpp
  //===--------------------------------------------------------------------===//
  // Application
  //===--------------------------------------------------------------------===//

  /// Replace the elements within the given operation. If `replaceAttrs` is
  /// true, this updates the attribute dictionary of the operation. If
  /// `replaceLocs` is true, this also updates its location, and the locations
  /// of any nested block arguments. If `replaceTypes` is true, this also
  /// updates the result types of the operation, and the types of any nested
  /// block arguments.
  void replaceElementsIn(Operation *op, bool replaceAttrs = true,
                         bool replaceLocs = false, bool replaceTypes = false);

  /// Replace the elements within the given operation, and all nested
  /// operations.
  void recursivelyReplaceElementsIn(Operation *op, bool replaceAttrs = true,
                                    bool replaceLocs = false,
                                    bool replaceTypes = false);
````
- **EN**: This block groups callable interfaces such as `replaceElementsIn`, `recursivelyReplaceElementsIn`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `replaceElementsIn`, `recursivelyReplaceElementsIn` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 155-214
````cpp
  //===--------------------------------------------------------------------===//
  // Registration
  //===--------------------------------------------------------------------===//

  /// A replacement mapping function, which returns either std::nullopt (to
  /// signal the element wasn't handled), or a pair of the replacement element
  /// and a WalkResult.
  template <typename T>
  using ReplaceFnResult = std::optional<std::pair<T, WalkResult>>;
  template <typename T>
  using ReplaceFn = std::function<ReplaceFnResult<T>(T)>;

  /// Register a replacement function for mapping a given attribute or type. A
  /// replacement function must be convertible to any of the following
  /// forms(where `T` is a class derived from `Type` or `Attribute`, and `BaseT`
  /// is either `Type` or `Attribute` respectively):
  ///
  ///   * std::optional<BaseT>(T)
  ///     - This either returns a valid Attribute/Type in the case of success,
  ///       nullptr in the case of failure, or `std::nullopt` to signify that
  ///       additional replacement functions may be applied (i.e. this function
  ///       doesn't handle that instance).
  ///
  ///   * std::optional<std::pair<BaseT, WalkResult>>(T)
  ///     - Similar to the above, but also allows specifying a WalkResult to
  ///       control the replacement of sub elements of a given attribute or
  ///       type. Returning a `skip` result, for example, will not recursively
  ///       process the resultant attribute or type value.
  ///
  /// Note: When replacing, the mostly recently added replacement functions will
  ///       be invoked first.
  void addReplacement(ReplaceFn<Attribute> fn);
  void addReplacement(ReplaceFn<Type> fn);

  /// Register a replacement function that doesn't match the default signature,
  /// either because it uses a derived parameter type, or it uses a simplified
  /// result type.
  template <typename FnT,
            typename T = typename llvm::function_traits<
                std::decay_t<FnT>>::template arg_t<0>,
            typename BaseT = std::conditional_t<std::is_base_of_v<Attribute, T>,
                                                Attribute, Type>,
            typename ResultT = std::invoke_result_t<FnT, T>>
  std::enable_if_t<!std::is_same_v<T, BaseT> ||
                   !std::is_convertible_v<ResultT, ReplaceFnResult<BaseT>>>
  addReplacement(FnT &&callback) {
    addReplacement([callback = std::forward<FnT>(callback)](
                       BaseT base) -> ReplaceFnResult<BaseT> {
      if (auto derived = dyn_cast<T>(base)) {
        if constexpr (std::is_convertible_v<ResultT, std::optional<BaseT>>) {
          std::optional<BaseT> result = callback(derived);
          return result ? std::make_pair(*result, WalkResult::advance())
                        : ReplaceFnResult<BaseT>();
        } else {
          return callback(derived);
        }
      }
      return ReplaceFnResult<BaseT>();
    });
  }
````
- **EN**: This block groups callable interfaces such as `addReplacement`, `constexpr`, `callback`, `make_pair`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `addReplacement`, `constexpr`, `callback`, `make_pair` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 215-255
````cpp
protected:
  /// Invokes the registered replacement functions from most recently registered
  /// to least recently registered until a successful replacement is returned.
  /// Unless skipping is requested, invokes `replace` on sub-elements of the
  /// current attr/type.
  Attribute replaceBase(Attribute attr);
  Type replaceBase(Type type);

private:
  /// The set of replacement functions that map sub elements.
  std::vector<ReplaceFn<Attribute>> attrReplacementFns;
  std::vector<ReplaceFn<Type>> typeReplacementFns;
};

} // namespace detail

/// This is an attribute/type replacer that is naively cached. It is best used
/// when the replacer logic is guaranteed to not contain cycles. Otherwise, any
/// re-occurrence of an in-progress element will be skipped.
class AttrTypeReplacer : public detail::AttrTypeReplacerBase<AttrTypeReplacer> {
public:
  Attribute replace(Attribute attr);
  Type replace(Type type);

private:
  /// Shared concrete implementation of the public `replace` functions. Invokes
  /// `replaceBase` with caching.
  template <typename T>
  T cachedReplaceImpl(T element);

  // Stores the opaque pointer of an attribute or type.
  DenseMap<const void *, const void *> cache;
};

/// This is an attribute/type replacer that supports custom handling of cycles
/// in the replacer logic. In addition to registering replacer functions, it
/// allows registering cycle-breaking functions in the same style.
class CyclicAttrTypeReplacer
    : public detail::AttrTypeReplacerBase<CyclicAttrTypeReplacer> {
public:
  CyclicAttrTypeReplacer();
````
- **EN**: This C++ declaration introduces `AttrTypeReplacer` and establishes part of the API surface for `AttrTypeSubElements`. Representative entry points here include `replaceBase`, `replace`, `cachedReplaceImpl`, `CyclicAttrTypeReplacer`.
- **CN**: 该 C++ 声明引入了 `AttrTypeReplacer`，并构成 `AttrTypeSubElements` API 表面的一部分。 这一段可见的代表性接口包括 `replaceBase`, `replace`, `cachedReplaceImpl`, `CyclicAttrTypeReplacer`。

### Lines 258-263
````cpp
  //===--------------------------------------------------------------------===//
  // Application
  //===--------------------------------------------------------------------===//

  Attribute replace(Attribute attr);
  Type replace(Type type);
````
- **EN**: This block groups callable interfaces such as `replace`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `replace` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 265-309
````cpp
  //===--------------------------------------------------------------------===//
  // Registration
  //===--------------------------------------------------------------------===//

  /// A cycle-breaking function. This is invoked if the same element is asked to
  /// be replaced again when the first instance of it is still being replaced.
  /// This function must not perform any more recursive `replace` calls.
  /// If it is able to break the cycle, it should return a replacement result.
  /// Otherwise, it can return std::nullopt to defer cycle breaking to the next
  /// repeated element. However, the user must guarantee that, in any possible
  /// cycle, there always exists at least one element that can break the cycle.
  template <typename T>
  using CycleBreakerFn = std::function<std::optional<T>(T)>;

  /// Register a cycle-breaking function.
  /// When breaking cycles, the mostly recently added cycle-breaking functions
  /// will be invoked first.
  void addCycleBreaker(CycleBreakerFn<Attribute> fn);
  void addCycleBreaker(CycleBreakerFn<Type> fn);

  /// Register a cycle-breaking function that doesn't match the default
  /// signature.
  template <typename FnT,
            typename T = typename llvm::function_traits<
                std::decay_t<FnT>>::template arg_t<0>,
            typename BaseT = std::conditional_t<std::is_base_of_v<Attribute, T>,
                                                Attribute, Type>>
  std::enable_if_t<!std::is_same_v<T, BaseT>> addCycleBreaker(FnT &&callback) {
    addCycleBreaker([callback = std::forward<FnT>(callback)](
                        BaseT base) -> std::optional<BaseT> {
      if (auto derived = dyn_cast<T>(base))
        return callback(derived);
      return std::nullopt;
    });
  }

private:
  /// Invokes the registered cycle-breaker functions from most recently
  /// registered to least recently registered until a successful result is
  /// returned.
  std::optional<const void *> breakCycleImpl(void *element);

  /// Shared concrete implementation of the public `replace` functions.
  template <typename T>
  T cachedReplaceImpl(T element);
````
- **EN**: This block groups callable interfaces such as `addCycleBreaker`, `callback`, `breakCycleImpl`, `cachedReplaceImpl`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `addCycleBreaker`, `callback`, `breakCycleImpl`, `cachedReplaceImpl` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 310-323
````cpp
  /// The set of registered cycle-breaker functions.
  std::vector<CycleBreakerFn<Attribute>> attrCycleBreakerFns;
  std::vector<CycleBreakerFn<Type>> typeCycleBreakerFns;

  /// A cache of previously-replaced attr/types.
  /// The key of the cache is the opaque value of an AttrOrType. Using
  /// AttrOrType allows distinguishing between the two types when invoking
  /// cycle-breakers. Using its opaque value avoids the cyclic dependency issue
  /// of directly using `AttrOrType` to instantiate the cache.
  /// The value of the cache is just the opaque value of the attr/type itself
  /// (not the PointerUnion).
  using AttrOrType = PointerUnion<Attribute, Type>;
  CyclicReplacerCache<void *, const void *> cache;
};
````
- **EN**: This section focuses on the set of registered cycle-breaker functions., grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“The set of registered cycle-breaker functions.”这一主题，把相关声明与辅助接口组织在一起。

### Lines 326-374
````cpp
//===----------------------------------------------------------------------===//
/// AttrTypeSubElementHandler
//===----------------------------------------------------------------------===//

/// This class is used by AttrTypeSubElementHandler instances to walking sub
/// attributes and types.
class AttrTypeImmediateSubElementWalker {
public:
  AttrTypeImmediateSubElementWalker(function_ref<void(Attribute)> walkAttrsFn,
                                    function_ref<void(Type)> walkTypesFn)
      : walkAttrsFn(walkAttrsFn), walkTypesFn(walkTypesFn) {}

  /// Walk an attribute.
  void walk(Attribute element);
  /// Walk a type.
  void walk(Type element);
  /// Walk a range of attributes or types.
  template <typename RangeT>
  void walkRange(RangeT &&elements) {
    for (auto element : elements)
      walk(element);
  }

private:
  function_ref<void(Attribute)> walkAttrsFn;
  function_ref<void(Type)> walkTypesFn;
};

/// This class is used by AttrTypeSubElementHandler instances to process sub
/// element replacements.
template <typename T>
class AttrTypeSubElementReplacements {
public:
  AttrTypeSubElementReplacements(ArrayRef<T> repls) : repls(repls) {}

  /// Take the first N replacements as an ArrayRef, dropping them from
  /// this replacement list.
  ArrayRef<T> take_front(unsigned n) {
    ArrayRef<T> elements = repls.take_front(n);
    repls = repls.drop_front(n);
    return elements;
  }

private:
  /// The current set of replacements.
  ArrayRef<T> repls;
};
using AttrSubElementReplacements = AttrTypeSubElementReplacements<Attribute>;
using TypeSubElementReplacements = AttrTypeSubElementReplacements<Type>;
````
- **EN**: This C++ declaration introduces `AttrTypeImmediateSubElementWalker` and establishes part of the API surface for `AttrTypeSubElements`. Representative entry points here include `AttrTypeImmediateSubElementWalker`, `void`, `walkAttrsFn`, `walkTypesFn`.
- **CN**: 该 C++ 声明引入了 `AttrTypeImmediateSubElementWalker`，并构成 `AttrTypeSubElements` API 表面的一部分。 这一段可见的代表性接口包括 `AttrTypeImmediateSubElementWalker`, `void`, `walkAttrsFn`, `walkTypesFn`。

### Lines 375-418
````cpp
/// This class provides support for interacting with the
/// SubElementInterfaces for different types of parameters. An
/// implementation of this class should be provided for any parameter class
/// that may contain an attribute or type. There are two main methods of
/// this class that need to be implemented:
///
///  - walk
///
///   This method should traverse into any sub elements of the parameter
///   using the provided walker, or by invoking handlers for sub-types.
///
///  - replace
///
///   This method should extract any necessary sub elements using the
///   provided replacer, or by invoking handlers for sub-types. The new
///   post-replacement parameter value should be returned.
///
template <typename T, typename Enable = void>
struct AttrTypeSubElementHandler {
  /// Default walk implementation that does nothing.
  static inline void walk(const T &param,
                          AttrTypeImmediateSubElementWalker &walker) {}

  /// Default replace implementation just forwards the parameter.
  template <typename ParamT>
  static inline decltype(auto) replace(ParamT &&param,
                                       AttrSubElementReplacements &attrRepls,
                                       TypeSubElementReplacements &typeRepls) {
    return std::forward<ParamT>(param);
  }

  /// Tag indicating that this handler does not support sub-elements.
  using DefaultHandlerTag = void;
};

/// Detect if any of the given parameter types has a sub-element handler.
namespace detail {
template <typename T>
using has_default_sub_element_handler_t = decltype(T::DefaultHandlerTag);
} // namespace detail
template <typename... Ts>
inline constexpr bool has_sub_attr_or_type_v =
    (!llvm::is_detected<detail::has_default_sub_element_handler_t, Ts>::value ||
     ...);
````
- **EN**: This C++ declaration introduces `AttrTypeSubElementHandler` and establishes part of the API surface for `AttrTypeSubElements`. Representative entry points here include `walk`, `replace`.
- **CN**: 该 C++ 声明引入了 `AttrTypeSubElementHandler`，并构成 `AttrTypeSubElements` API 表面的一部分。 这一段可见的代表性接口包括 `walk`, `replace`。

### Lines 420-495
````cpp
/// Implementation for derived Attributes and Types.
template <typename T>
struct AttrTypeSubElementHandler<
    T, std::enable_if_t<std::is_base_of_v<Attribute, T> ||
                        std::is_base_of_v<Type, T>>> {
  static void walk(T param, AttrTypeImmediateSubElementWalker &walker) {
    walker.walk(param);
  }
  static T replace(T param, AttrSubElementReplacements &attrRepls,
                   TypeSubElementReplacements &typeRepls) {
    if (!param)
      return T();
    if constexpr (std::is_base_of_v<Attribute, T>) {
      return cast<T>(attrRepls.take_front(1)[0]);
    } else {
      return cast<T>(typeRepls.take_front(1)[0]);
    }
  }
};
/// Implementation for derived ArrayRef.
template <typename T>
struct AttrTypeSubElementHandler<ArrayRef<T>,
                                 std::enable_if_t<has_sub_attr_or_type_v<T>>> {
  using EltHandler = AttrTypeSubElementHandler<T>;

  static void walk(ArrayRef<T> param,
                   AttrTypeImmediateSubElementWalker &walker) {
    for (const T &subElement : param)
      EltHandler::walk(subElement, walker);
  }
  static auto replace(ArrayRef<T> param, AttrSubElementReplacements &attrRepls,
                      TypeSubElementReplacements &typeRepls) {
    // Normal attributes/types can extract using the replacer directly.
    if constexpr (std::is_base_of_v<Attribute, T> &&
                  sizeof(T) == sizeof(void *)) {
      ArrayRef<Attribute> attrs = attrRepls.take_front(param.size());
      return ArrayRef<T>((const T *)attrs.data(), attrs.size());
    } else if constexpr (std::is_base_of_v<Type, T> &&
                         sizeof(T) == sizeof(void *)) {
      ArrayRef<Type> types = typeRepls.take_front(param.size());
      return ArrayRef<T>((const T *)types.data(), types.size());
    } else {
      // Otherwise, we need to allocate storage for the new elements.
      SmallVector<T> newElements;
      for (const T &element : param)
        newElements.emplace_back(
            EltHandler::replace(element, attrRepls, typeRepls));
      return newElements;
    }
  }
};
/// Implementation for Tuple.
template <typename... Ts>
struct AttrTypeSubElementHandler<
    std::tuple<Ts...>, std::enable_if_t<has_sub_attr_or_type_v<Ts...>>> {
  static void walk(const std::tuple<Ts...> &param,
                   AttrTypeImmediateSubElementWalker &walker) {
    std::apply(
        [&](const Ts &...params) {
          (AttrTypeSubElementHandler<Ts>::walk(params, walker), ...);
        },
        param);
  }
  static auto replace(const std::tuple<Ts...> &param,
                      AttrSubElementReplacements &attrRepls,
                      TypeSubElementReplacements &typeRepls) {
    return std::apply(
        [&](const Ts &...params)
            -> std::tuple<decltype(AttrTypeSubElementHandler<Ts>::replace(
                params, attrRepls, typeRepls))...> {
          return {AttrTypeSubElementHandler<Ts>::replace(params, attrRepls,
                                                         typeRepls)...};
        },
        param);
  }
};
````
- **EN**: This C++ declaration introduces `AttrTypeSubElementHandler` and establishes part of the API surface for `AttrTypeSubElements`. Representative entry points here include `walk`, `replace`, `T`, `constexpr`.
- **CN**: 该 C++ 声明引入了 `AttrTypeSubElementHandler`，并构成 `AttrTypeSubElements` API 表面的一部分。 这一段可见的代表性接口包括 `walk`, `replace`, `T`, `constexpr`。

### Lines 497-550
````cpp
namespace detail {
template <typename T>
struct is_tuple : public std::false_type {};
template <typename... Ts>
struct is_tuple<std::tuple<Ts...>> : public std::true_type {};

template <typename T>
struct is_pair : public std::false_type {};
template <typename... Ts>
struct is_pair<std::pair<Ts...>> : public std::true_type {};

template <typename T, typename... Ts>
using has_get_method = decltype(T::get(std::declval<Ts>()...));
template <typename T, typename... Ts>
using has_get_as_key = decltype(std::declval<T>().getAsKey());

/// This function provides the underlying implementation for the
/// SubElementInterface walk method, using the key type of the derived
/// attribute/type to interact with the individual parameters.
template <typename T>
void walkImmediateSubElementsImpl(T derived,
                                  function_ref<void(Attribute)> walkAttrsFn,
                                  function_ref<void(Type)> walkTypesFn) {
  using ImplT = typename T::ImplType;
  (void)derived;
  (void)walkAttrsFn;
  (void)walkTypesFn;
  if constexpr (llvm::is_detected<has_get_as_key, ImplT>::value) {
    auto key = static_cast<ImplT *>(derived.getImpl())->getAsKey();

    // If we don't have any sub-elements, there is nothing to do.
    if constexpr (!has_sub_attr_or_type_v<decltype(key)>)
      return;
    AttrTypeImmediateSubElementWalker walker(walkAttrsFn, walkTypesFn);
    AttrTypeSubElementHandler<decltype(key)>::walk(key, walker);
  }
}

/// This function invokes the proper `get` method for  a type `T` with the given
/// values.
template <typename T, typename... Ts>
auto constructSubElementReplacement(MLIRContext *ctx, Ts &&...params) {
  // Prefer a direct `get` method if one exists.
  if constexpr (llvm::is_detected<has_get_method, T, Ts...>::value) {
    (void)ctx;
    return T::get(std::forward<Ts>(params)...);
  } else if constexpr (llvm::is_detected<has_get_method, T, MLIRContext *,
                                         Ts...>::value) {
    return T::get(ctx, std::forward<Ts>(params)...);
  } else {
    // Otherwise, pass to the base get.
    return T::Base::get(ctx, std::forward<Ts>(params)...);
  }
}
````
- **EN**: This C++ declaration introduces `is_tuple` and establishes part of the API surface for `AttrTypeSubElements`. Representative entry points here include `get`, `getAsKey`, `walkImmediateSubElementsImpl`, `void`.
- **CN**: 该 C++ 声明引入了 `is_tuple`，并构成 `AttrTypeSubElements` API 表面的一部分。 这一段可见的代表性接口包括 `get`, `getAsKey`, `walkImmediateSubElementsImpl`, `void`。

### Lines 552-598
````cpp
/// This function provides the underlying implementation for the
/// SubElementInterface replace method, using the key type of the derived
/// attribute/type to interact with the individual parameters.
template <typename T>
auto replaceImmediateSubElementsImpl(T derived, ArrayRef<Attribute> &replAttrs,
                                     ArrayRef<Type> &replTypes) {
  using ImplT = typename T::ImplType;
  if constexpr (llvm::is_detected<has_get_as_key, ImplT>::value) {
    auto key = static_cast<ImplT *>(derived.getImpl())->getAsKey();

    // If we don't have any sub-elements, we can just return the original.
    if constexpr (!has_sub_attr_or_type_v<decltype(key)>) {
      return derived;

      // Otherwise, we need to replace any necessary sub-elements.
    } else {
      // Functor used to build the replacement on success.
      auto buildReplacement = [&](auto newKey, MLIRContext *ctx) {
        if constexpr (is_tuple<decltype(key)>::value ||
                      is_pair<decltype(key)>::value) {
          return std::apply(
              [&](auto &&...params) {
                return constructSubElementReplacement<T>(
                    ctx, std::forward<decltype(params)>(params)...);
              },
              newKey);
        } else {
          return constructSubElementReplacement<T>(ctx, newKey);
        }
      };

      AttrSubElementReplacements attrRepls(replAttrs);
      TypeSubElementReplacements typeRepls(replTypes);
      auto newKey = AttrTypeSubElementHandler<decltype(key)>::replace(
          key, attrRepls, typeRepls);
      MLIRContext *ctx = derived.getContext();
      if constexpr (std::is_convertible_v<decltype(newKey), LogicalResult>)
        return succeeded(newKey) ? buildReplacement(*newKey, ctx) : nullptr;
      else
        return buildReplacement(newKey, ctx);
    }
  } else {
    return derived;
  }
}
} // namespace detail
} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `replaceImmediateSubElementsImpl`, `constexpr`, `getImpl`, `getAsKey`, indicating how `AttrTypeSubElements` is queried or updated.
- **CN**: 该代码块聚合了 `replaceImmediateSubElementsImpl`, `constexpr`, `getImpl`, `getAsKey` 等可调用接口，展示了如何查询或更新 `AttrTypeSubElements`。

### Lines 600-600
````cpp
#endif // MLIR_IR_ATTRTYPESUBELEMENTS_H
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层
- **EN**: Type or attribute abstraction
  **CN**: 类型或属性抽象

## Dependencies / 依赖关系

- mlir/IR/MLIRContext.h
- mlir/IR/Visitors.h
- mlir/Support/CyclicReplacerCache.h
- llvm/ADT/ArrayRef.h
- llvm/ADT/DenseMap.h
- AttrTypeReplacer inherits from public detail::AttrTypeReplacerBase<AttrTypeReplacer>
- CyclicAttrTypeReplacer inherits from public detail::AttrTypeReplacerBase<CyclicAttrTypeReplacer>
- AttrTypeReplacer builds on public detail::AttrTypeReplacerBase<AttrTypeReplacer>
- CyclicAttrTypeReplacer builds on public detail::AttrTypeReplacerBase<CyclicAttrTypeReplacer>
