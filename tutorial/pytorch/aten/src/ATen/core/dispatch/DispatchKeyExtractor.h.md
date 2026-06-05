# DispatchKeyExtractor.h — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/dispatch/DispatchKeyExtractor.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares core ATen/C10 abstractions, with primary focus on `MultiDispatchKeySet`, `DispatchKeyExtractor`, `CachedTypes`.
- 用途（中文）: 该文件声明ATen/C10 核心抽象，核心关注对象是 `MultiDispatchKeySet`, `DispatchKeyExtractor`, `CachedTypes`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
#pragma once

#include <ATen/core/Variadic.h>
#include <ATen/core/function_schema.h>
#include <ATen/core/jit_type.h>
#include <ATen/core/stack.h>
#include <c10/core/DispatchKeySet.h>
#include <c10/util/Bitset.h>
#include <c10/util/irange.h>
#include <cstdint>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 12-21
```cpp
namespace c10 {

namespace impl {

// Take a DispatchKeySet for a Tensor and determine what the actual dispatch
// DispatchKey should be, taking into account TLS, and skipping backends which
// fall through.
//
// Unlike Tensor::key_set(), the value of this on a tensor can change depending
// on TLS.
```
- EN: Focus symbols: `c10`, `impl`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`c10`, `impl`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 22-31
```cpp
//
// NB: If there is no valid dispatch key, this will return Undefined
inline DispatchKeySet computeDispatchKeySet(
    DispatchKeySet ks,
    // The key mask lets us eliminate (by zero entries) keys which should not
    // be considered for dispatch.  There are two cases when we use this:
    //
    // - If an operator's dispatch table contains a fallthrough entry, we
    //   should bypass it entirely when finding the key
    // - If a user invokes with redispatch, the mask lets us
```
- EN: Focus symbols: `computeDispatchKeySet`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`computeDispatchKeySet`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 32-41
```cpp
    //   zero out the key the user asked us to stop.
    //
    // These excluded backends are NOT tracked in the TLS, but must be applied
    // AFTER TLS (since the backend may have been introduced for consideration
    // by the included TLS), which is why you have to pass them in to this
    // function (as opposed to just applying it to the input 'ks').
    DispatchKeySet key_mask) {
  c10::impl::LocalDispatchKeySet local =
      c10::impl::tls_local_dispatch_key_set();
  // TODO: It's a bit irritating that we have to do logical ORs here, it would
```
- EN: Focus symbols: `tls_local_dispatch_key_set`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`tls_local_dispatch_key_set`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 42-51
```cpp
  // be nice to only do one.  Can always_included be folded into the TLS?  Well,
  // it's a bit troublesome, because fastpath TLS access requires the type of
  // the TLS in question to be zero-initialized, so you don't actually win
  // anything in that case.
  return (((ks | local.included_) - local.excluded_) & key_mask);
}

} // namespace impl

namespace detail {
```
- EN: Focus symbols: `impl`, `detail`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`impl`, `detail`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 52-61
```cpp
// A small gadget to extract the DispatchKeySet from types which are known
// to have it.  Used to extract dispatch keys from unboxed calls.
struct MultiDispatchKeySet : at::IterArgs<MultiDispatchKeySet> {
  DispatchKeySet ts;
  void operator()(const at::Tensor& x) {
    ts = ts | x.key_set();
  }
  void operator()(const std::optional<at::Tensor>& x) {
    if (x.has_value()) {
      ts = ts | x->key_set();
```
- EN: Focus symbols: `MultiDispatchKeySet`, `operator`, `key_set`, `has_value`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`MultiDispatchKeySet`, `operator`, `key_set`, `has_value`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 62-71
```cpp
    }
  }
  void operator()(at::ArrayRef<at::Tensor> xs) {
    for (const auto& x : xs) {
      ts = ts | x.key_set();
    }
  }
  // Tensor?[] translates to this case.
  void operator()(const c10::List<std::optional<at::Tensor>>& xs) {
    for (std::optional<at::Tensor> x : xs) {
```
- EN: Focus symbols: `operator`, `key_set`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`operator`, `key_set`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 72-81
```cpp
      if (x.has_value()) {
        ts = ts | x.value().key_set();
      }
    }
  }
  // Structured Tensor[] translates to this case
  void operator()(const at::ITensorListRef& xs) {
    for (const auto& x : xs) {
      ts = ts | x.key_set();
    }
```
- EN: Focus symbols: `has_value`, `value`, `key_set`, `operator`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`has_value`, `value`, `key_set`, `operator`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 82-91
```cpp
  }
  [[noreturn]] void operator()(
      at::ArrayRef<std::optional<at::Tensor>> /*unused*/) {
    // Just checking that the handling of Tensor?[] didn't change.
    TORCH_INTERNAL_ASSERT(false);
  }
  void operator()(const at::Generator& gen) {
    if (gen.defined()) {
      ts = ts | gen.key_set();
    }
```
- EN: Focus symbols: `operator`, `TORCH_INTERNAL_ASSERT`, `defined`, `key_set`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`operator`, `TORCH_INTERNAL_ASSERT`, `defined`, `key_set`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 92-103
```cpp
  }
  void operator()(const std::optional<at::Generator>& gen) {
    if (gen.has_value() && gen->defined()) {
      ts = ts | gen->key_set();
    }
  }
  template <typename T>
  void operator()(const T& /*unused*/) {
    // do nothing
  }
};

```
- EN: Focus symbols: `operator`, `has_value`, `defined`, `key_set`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`operator`, `has_value`, `defined`, `key_set`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 104-113
```cpp
// NB: take by const reference (Don't do universal forwarding here! You
// don't want to move into this function!)
template <typename... Args>
DispatchKeySet multi_dispatch_key_set(const Args&... args) {
  return MultiDispatchKeySet().apply(args...).ts;
}
} // namespace detail

/**
 * An instance of DispatchKeyExtractor knows how to get a dispatch key given
```
- EN: Focus symbols: `detail`, `multi_dispatch_key_set`, `MultiDispatchKeySet`, `apply`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`detail`, `multi_dispatch_key_set`, `MultiDispatchKeySet`, `apply`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 114-123
```cpp
 * a list of arguments for an operator call.
 *
 * The instance is specific for a certain operator as:
 *  - In boxed dispatch, different operators have different ways to extract
 *    the dispatch key (e.g. different numbers of arguments), and we precompute
 *    the stack locations we should look at; and
 *  - In all dispatch, some backends should be excluded from dispatch because
 *    they have been registered as fallthrough.  The set of excluded backends
 *    varies from operator, as some operators may have overridden the
 *    fallthrough with custom behavior.
```
- EN: Focus symbols: `key`. This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 关注符号：`key`。该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

### Lines 124-137
```cpp
 *
 *   Note - this should maintain identical impl to the py dispatcher key
 * extraction logic at pytorch/torch/dispatcher.py
 */
struct TORCH_API DispatchKeyExtractor final {
 public:
  static DispatchKeyExtractor make(const FunctionSchema& schema) {
    return DispatchKeyExtractor(makeBitsetForDispatchArgs(schema));
  }

  static DispatchKeyExtractor makeUninitialized() {
    return DispatchKeyExtractor(c10::utils::bitset());
  }

```
- EN: Focus symbols: `DispatchKeyExtractor`, `make`, `makeBitsetForDispatchArgs`, `makeUninitialized`, `bitset`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`DispatchKeyExtractor`, `make`, `makeBitsetForDispatchArgs`, `makeUninitialized`, `bitset`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 138-147
```cpp
  void registerSchema(const FunctionSchema& schema) {
    TORCH_INTERNAL_ASSERT(dispatch_arg_indices_reverse_.is_entirely_unset());
    dispatch_arg_indices_reverse_ = makeBitsetForDispatchArgs(schema);
  }
  void deregisterSchema() {
    dispatch_arg_indices_reverse_ = c10::utils::bitset();
  }

  DispatchKeySet getDispatchKeySetBoxed(const torch::jit::Stack* stack) const {
    DispatchKeySet ks;
```
- EN: Focus symbols: `registerSchema`, `TORCH_INTERNAL_ASSERT`, `is_entirely_unset`, `makeBitsetForDispatchArgs`, `deregisterSchema`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`registerSchema`, `TORCH_INTERNAL_ASSERT`, `is_entirely_unset`, `makeBitsetForDispatchArgs`, `deregisterSchema`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 148-157
```cpp
    dispatch_arg_indices_reverse_.for_each_set_bit([&](size_t
                                                           reverse_arg_index) {
      const auto& ivalue = torch::jit::peek(*stack, 0, reverse_arg_index + 1);
      if (C10_LIKELY(ivalue.isTensor())) {
        // NB: Take care not to introduce a refcount bump (there's
        // no safe toTensorRef method, alas)
        ks = ks | ivalue.unsafeToTensorImpl()->key_set();
      } else if (C10_UNLIKELY(ivalue.isTensorList())) {
        // NB: use toListRef as it doesn't induce refcount bumps
        // (toTensorListRef is not a thing)
```
- EN: Focus symbols: `for_each_set_bit`, `peek`, `C10_LIKELY`, `isTensor`, `unsafeToTensorImpl`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`for_each_set_bit`, `peek`, `C10_LIKELY`, `isTensor`, `unsafeToTensorImpl`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 158-167
```cpp
        for (const auto& nv : ivalue.toListRef()) {
          auto* tensor = nv.unsafeToTensorImpl();
          ks = ks | tensor->key_set();
        }
      }
      // Tensor?[] translates to a c10::List<IValue> so we need to peek inside
      else if (C10_UNLIKELY(ivalue.isList())) {
        for (const auto& elt : ivalue.toListRef()) {
          if (elt.isTensor()) {
            ks = ks | elt.toTensor().key_set();
```
- EN: Focus symbols: `toListRef`, `unsafeToTensorImpl`, `key_set`, `C10_UNLIKELY`, `isList`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toListRef`, `unsafeToTensorImpl`, `key_set`, `C10_UNLIKELY`, `isList`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 168-177
```cpp
          }
        }
      }
    });
    // Keys that are fallthrough should be skipped
    if (requiresBitsetPerBackend_) {
      c10::impl::LocalDispatchKeySet tls =
          c10::impl::tls_local_dispatch_key_set();
      auto backend_idx =
          ((ks | tls.included_) - tls.excluded_).getBackendIndex();
```
- EN: Focus symbols: `tls_local_dispatch_key_set`, `getBackendIndex`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`tls_local_dispatch_key_set`, `getBackendIndex`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 178-187
```cpp
      return impl::computeDispatchKeySet(
          ks, nonFallthroughKeysPerBackend_[backend_idx]);
    } else {
      return impl::computeDispatchKeySet(ks, nonFallthroughKeys_);
    }
  }

  template <class... Args>
  DispatchKeySet getDispatchKeySetUnboxed(const Args&... args) const {
    auto ks = detail::multi_dispatch_key_set(args...);
```
- EN: Focus symbols: `computeDispatchKeySet`, `getDispatchKeySetUnboxed`, `multi_dispatch_key_set`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`computeDispatchKeySet`, `getDispatchKeySetUnboxed`, `multi_dispatch_key_set`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 188-200
```cpp
    // Keys that are fallthrough should be skipped
    if (requiresBitsetPerBackend_) {
      c10::impl::LocalDispatchKeySet tls =
          c10::impl::tls_local_dispatch_key_set();
      auto backend_idx =
          ((ks | tls.included_) - tls.excluded_).getBackendIndex();
      return impl::computeDispatchKeySet(
          ks, nonFallthroughKeysPerBackend_[backend_idx]);
    } else {
      return impl::computeDispatchKeySet(ks, nonFallthroughKeys_);
    }
  }

```
- EN: Focus symbols: `tls_local_dispatch_key_set`, `getBackendIndex`, `computeDispatchKeySet`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`tls_local_dispatch_key_set`, `getBackendIndex`, `computeDispatchKeySet`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 201-210
```cpp
  void setOperatorHasFallthroughForKey(DispatchKey k, bool has_fallthrough);

  std::string dumpState() const;
  void checkInvariants(const FunctionSchema& schema) const;

 private:
  static bool isDispatchType(const Type& type) {
    // Checking isSubtypeOf on a DynamicType heap-allocates a
    // DynamicType version of the argument if it's not a DynamicType
    // already, and this has measurable overhead during startup.
```
- EN: Focus symbols: `setOperatorHasFallthroughForKey`, `dumpState`, `checkInvariants`, `isDispatchType`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`setOperatorHasFallthroughForKey`, `dumpState`, `checkInvariants`, `isDispatchType`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 211-220
```cpp
#ifdef C10_MOBILE
    struct CachedTypes {
      DynamicTypePtr listOfTensors;
      DynamicTypePtr listOfOptionalTensors;
      DynamicTypePtr optionalOfTensor;
    };
    static const CachedTypes ct = {
        DynamicType::create(*ListType::ofTensors()),
        DynamicType::create(*ListType::ofOptionalTensors()),
        DynamicType::create(*OptionalType::ofTensor())};
```
- EN: Focus symbols: `CachedTypes`, `create`, `ofTensors`, `ofOptionalTensors`, `ofTensor`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CachedTypes`, `create`, `ofTensors`, `ofOptionalTensors`, `ofTensor`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 221-230
```cpp
    return type.isSubtypeOf(c10::TypeFactory::get<TensorType>()) ||
        type.isSubtypeOf(ct.listOfTensors) ||
        type.isSubtypeOf(ct.listOfOptionalTensors) ||
        type.isSubtypeOf(ct.optionalOfTensor);
#else // C10_MOBILE
    return type.isSubtypeOf(*TensorType::get()) ||
        type.isSubtypeOf(*ListType::ofTensors()) ||
        type.isSubtypeOf(*ListType::ofOptionalTensors()) ||
        type.isSubtypeOf(*OptionalType::ofTensor());
#endif // C10_MOBILE
```
- EN: Focus symbols: `isSubtypeOf`, `get`, `ofTensors`, `ofOptionalTensors`, `ofTensor`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isSubtypeOf`, `get`, `ofTensors`, `ofOptionalTensors`, `ofTensor`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 231-240
```cpp
  }
  static c10::utils::bitset makeBitsetForDispatchArgs(
      const FunctionSchema& schema) {
    TORCH_CHECK(
        schema.arguments().size() <= c10::utils::bitset::NUM_BITS(),
        "The function schema has ",
        schema.arguments().size(),
        " arguments but this PyTorch build only supports ",
        c10::utils::bitset::NUM_BITS());
    c10::utils::bitset dispatch_arg_indices_reverse;
```
- EN: Focus symbols: `makeBitsetForDispatchArgs`, `TORCH_CHECK`, `arguments`, `size`, `NUM_BITS`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`makeBitsetForDispatchArgs`, `TORCH_CHECK`, `arguments`, `size`, `NUM_BITS`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 241-250
```cpp
    for (const auto index : c10::irange(schema.arguments().size())) {
      if (isDispatchType(*schema.arguments()[index].type())) {
        dispatch_arg_indices_reverse.set(schema.arguments().size() - 1 - index);
      }
    }
    return dispatch_arg_indices_reverse;
  }

  explicit DispatchKeyExtractor(c10::utils::bitset dispatch_arg_indices_reverse)
      : dispatch_arg_indices_reverse_(dispatch_arg_indices_reverse),
```
- EN: Focus symbols: `irange`, `arguments`, `size`, `isDispatchType`, `type`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`irange`, `arguments`, `size`, `isDispatchType`, `type`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 251-260
```cpp
        nonFallthroughKeys_(DispatchKeySet::FULL) {
    for (const auto i : c10::irange(nonFallthroughKeysPerBackend_.size())) {
      nonFallthroughKeysPerBackend_[i] = DispatchKeySet::FULL;
    }
  }

  // this is a bitset that has ones for each argument index which has to be
  // considered for dispatch. This avoids having to iterate over the stack
  // to find all the tensors. The bits are stored in reverse order, i.e.
  // dispatch_arg_indices_reverse_[i] == true, then the i-th argument from
```
- EN: Focus symbols: `nonFallthroughKeys_`, `irange`, `size`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`nonFallthroughKeys_`, `irange`, `size`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 261-270
```cpp
  // the top of the stack (i.e. the i-th last argument of the function)
  // is relevant for dispatch.
  // dispatch_arg_indices_reverse_ is allowed to have zero bits set; that just
  // means you must do the fallthrough
  c10::utils::bitset dispatch_arg_indices_reverse_;

  // Set of functionality keys for which the operator does NOT have fallthrough
  // kernel.
  DispatchKeySet nonFallthroughKeys_;
  // Set of functionality keys for which the operator does NOT have fallthrough
```
- EN: This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 271-280
```cpp
  // kernel, defined PER BACKEND. This is only needed if we know that the
  // operator has a different set of fallthroughs defined for some backends.
  std::array<DispatchKeySet, num_backends> nonFallthroughKeysPerBackend_;
  // Flag to tell us if we can use the single set of nonFallthroughKeys_ (fast
  // path), or if we need to fall back to the slower path and check
  // nonFallthroughKeysPerBackend_
  bool requiresBitsetPerBackend_{false};
};

} // namespace c10
```
- EN: Focus symbols: `c10`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`c10`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- C++ templates and specialization / C++ 模板与特化
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- Backend/device dispatch / 后端/设备分发
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/Variadic.h`, `ATen/core/function_schema.h`, `ATen/core/jit_type.h`, `ATen/core/stack.h`, `c10/core/DispatchKeySet.h`, `c10/util/Bitset.h`, `c10/util/irange.h`
- External/system includes / 外部或系统头: `cstdint`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/core/dispatch/DispatchKeyExtractor.cpp`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; C++ templates / C++ 模板; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
