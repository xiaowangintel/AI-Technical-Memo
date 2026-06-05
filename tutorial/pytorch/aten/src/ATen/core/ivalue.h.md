# ivalue.h — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/ivalue.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file declares core ATen/C10 abstractions, with primary focus on `CustomClassHolder`, `Function`, `CompilationUnit`.
- 用途（中文）: 该文件声明ATen/C10 核心抽象，核心关注对象是 `CustomClassHolder`, `Function`, `CompilationUnit`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
#pragma once

#include <ATen/core/DimVector.h>
#include <ATen/core/TensorBody.h>
#include <ATen/core/blob.h>
#include <ATen/core/custom_class.h>
#include <ATen/core/ivalue_to.h>
#include <ATen/core/jit_type_base.h>
#include <ATen/core/type_factory.h>
#include <c10/core/SymBool.h>
#include <c10/core/SymFloat.h>
#include <c10/macros/Export.h>
#include <c10/util/MaybeOwned.h>
#include <c10/util/intrusive_ptr.h>
#include <limits>
#include <type_traits>
#include <unordered_map>
#include <unordered_set>
#include <utility>

C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wswitch-default")

namespace torch {
class TORCH_API CustomClassHolder : public c10::intrusive_ptr_target {};
```
- EN: Focus symbols: `CustomClassHolder`, `torch`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`CustomClassHolder`, `torch`, `C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 25-52
```cpp
namespace jit {
using ::torch::CustomClassHolder;
struct Function;
struct CompilationUnit;
struct Module;
} // namespace jit
} // namespace torch
namespace c10 {
template <class Key, class Value>
class Dict;
template <class T>
class List;
template <class T>
class IListRef;
struct IValue;
struct ClassType;
struct Type;
class RRefInterface;

struct ClassType;
using ClassTypePtr = std::shared_ptr<ClassType>;

TORCH_API bool _fastEqualsForContainer(const IValue& lhs, const IValue& rhs);

TORCH_API torch::jit::Function* checkObjectSortSchema(
    const c10::ClassTypePtr& t,
    std::stringstream& why_not);

```
- EN: Focus symbols: `Function`, `CompilationUnit`, `Module`, `Key`, `Value`, `Dict`, `T`, `List`, `IListRef`, `IValue`, `ClassType`, `Type`, `RRefInterface`, `ClassTypePtr`, `jit`, `torch`, `c10`, `_fastEqualsForContainer`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Function`, `CompilationUnit`, `Module`, `Key`, `Value`, `Dict`, `T`, `List`, `IListRef`, `IValue`, `ClassType`, `Type`, `RRefInterface`, `ClassTypePtr`, `jit`, `torch`, `c10`, `_fastEqualsForContainer`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 53-76
```cpp
// A comparator that checks ordering of two IValues of same type.
typedef std::function<bool(const IValue& a, const IValue& b)> IValueComparator;

TORCH_API IValueComparator getLessThanComparator(const IValue& v);
TORCH_API IValueComparator getGreaterThanComparator(const IValue& v);

namespace ivalue {
struct Tuple;
struct Future;
struct Await;
struct ConstantString;
struct GenericDict;
struct Object;
struct PyObjectHolder;
struct EnumHolder;
// We need a ComplexHolder because currently the payloads in the Union
// only take 64 bits. Since ComplexDouble takes up 128 bits, and is too big
// to fit in the IValue directly, we indirect complex numbers through an
// intrusive pointer to ComplexHolder (which contains a c10::complex).
struct ComplexHolder : c10::intrusive_ptr_target {
 public:
  template <typename T>
  ComplexHolder(c10::complex<T> c) {
    val = convert<decltype(val), c10::complex<T>>(c);
```
- EN: Focus symbols: `Tuple`, `Future`, `Await`, `ConstantString`, `GenericDict`, `Object`, `PyObjectHolder`, `EnumHolder`, `ComplexHolder`, `ivalue`, `bool`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Tuple`, `Future`, `Await`, `ConstantString`, `GenericDict`, `Object`, `PyObjectHolder`, `EnumHolder`, `ComplexHolder`, `ivalue`, `bool`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 77-103
```cpp
  }
  ComplexHolder() = default;
  c10::complex<double> val;
};

// Similar to ComplexHolder, for StreamData3
struct StreamData3Holder : c10::intrusive_ptr_target {
 public:
  StreamData3Holder(struct c10::StreamData3 d) : val(d) {}
  StreamData3Holder() = delete;
  struct c10::StreamData3 val;
};

} // namespace ivalue

// This is an owning wrapper for a std::optional<std::vector<T>>
// that can be implicitly converted to a (non-owning) std::optional<ArrayRef<T>>.
// Its purpose is to be used in generated code to keep the vector alive
// either until the end of a statement (as a temporary), or as a saved arg
// in autograd.
template <typename T>
struct OptionalArray {
  std::optional<std::vector<T>> list;

  OptionalArray() = default;
  OptionalArray(std::vector<T> val) : list(std::move(val)) {}

```
- EN: Focus symbols: `StreamData3Holder`, `c10`, `OptionalArray`, `ivalue`, `ComplexHolder`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`StreamData3Holder`, `c10`, `OptionalArray`, `ivalue`, `ComplexHolder`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 104-130
```cpp
  // Used when saving an argument for the backwards pass.
  OptionalArray& operator=(std::optional<ArrayRef<T>> ref) {
    if (ref) {
      list = std::vector<T>(ref->begin(), ref->end());
    } else {
      list = std::nullopt;
    }
    return *this;
  }

  // Used when saving an argument for the backwards pass.
  OptionalArray& operator=(c10::OptionalArrayRef<T> ref) {
    if (ref) {
      list = std::vector<T>(ref->begin(), ref->end());
    } else {
      list = std::nullopt;
    }
    return *this;
  }

  operator std::optional<c10::ArrayRef<T>>() {
    if (!list) {
      return std::nullopt;
    }
    return *list;
  }

```
- EN: Focus symbols: `begin`, `end`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`begin`, `end`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 131-158
```cpp
  operator c10::OptionalArrayRef<T>() {
    if (!list) {
      return std::nullopt;
    }
    return *list;
  }
};

// Capsule is an internal implementation detail of custom C++ classes. We
// define it as an owning wrapper for
// c10::intrusive_ptr<torch::CustomClassHolder> This wrapper is here to serve as
// an abstraction of the type erased custom class object pointer. It also allow
// pybind11 to treat this as a standalone class to register as a separate type
// caster, instead of a custom pointer holder which the pointer holder type
// caster try to "unwrap" it automatically.
struct Capsule {
  c10::intrusive_ptr<torch::CustomClassHolder> obj_ptr;
  explicit Capsule(c10::intrusive_ptr<torch::CustomClassHolder> ptr)
      : obj_ptr(std::move(ptr)) {}
};

// IValue is the generic tagged union used by the interpreter to hold
// all value types.
// It is a 16-byte object with an 8-byte payload and an 8-byte tag.
// The tag is currently 4 bytes to determine the type, and 1 byte
// to mark whether that type is a subtype of c10::intrusive_ptr_target and needs
// retain/release calls.

```
- EN: Focus symbols: `object`, `to`, `Capsule`, `obj_ptr`, `move`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`object`, `to`, `Capsule`, `obj_ptr`, `move`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 159-182
```cpp
#define TORCH_FORALL_TAGS(_) \
  _(None)                    \
  _(Tensor)                  \
  _(Storage)                 \
  _(Double)                  \
  _(ComplexDouble)           \
  _(Int)                     \
  _(UInt)                    \
  _(SymInt)                  \
  _(SymFloat)                \
  _(SymBool)                 \
  _(Bool)                    \
  _(Tuple)                   \
  _(String)                  \
  _(Blob)                    \
  _(GenericList)             \
  _(GenericDict)             \
  _(Future)                  \
  _(Await)                   \
  _(Device)                  \
  _(Stream)                  \
  _(Object)                  \
  _(PyObject)                \
  _(Uninitialized)           \
```
- EN: Focus symbols: `TORCH_FORALL_TAGS`, `_`. This block enumerates repetitive per-type or per-backend mappings. The repeated invocations make policy explicit without hand-writing every specialization body.
- CN: 关注符号：`TORCH_FORALL_TAGS`, `_`。该代码块枚举按类型或按后端展开的重复映射。这些重复调用显式表达策略，同时避免手写每个特化主体。

### Lines 183-206
```cpp
  _(Capsule)                 \
  _(RRef)                    \
  _(Quantizer)               \
  _(Generator)               \
  _(Enum)

// [doxygen private]
// These methods are not actually private but we don't want to document them, so
// they are marked `@private`, which hides them on the doxygen documentation for
// this page.

/// IValue (Interpreter Value) is a tagged union over the types
/// supported by the TorchScript interpreter. IValues contain their
/// values as an `IValue::Payload`, which holds primitive types
/// (`int64_t`, `bool`, `double`, `Device`) and `Tensor` as values,
/// and all other types as a `c10::intrusive_ptr`. In order to
/// optimize performance of the destructor and related operations by
/// making the `Tensor` and `c10::intrusive_ptr` paths generate the
/// same code, we represent a null `c10::intrusive_ptr` as
/// `UndefinedTensorImpl::singleton()`, *not* `nullptr`.
///
/// IValues are used as inputs to and outputs from the TorchScript interpreter.
/// To retrieve the value contained within an IValue, use the `.toX()` methods,
/// where `X` is the type you are trying to get. Note that neither the `.toX()`
```
- EN: Focus symbols: `_`. This block enumerates repetitive per-type or per-backend mappings. The repeated invocations make policy explicit without hand-writing every specialization body.
- CN: 关注符号：`_`。该代码块枚举按类型或按后端展开的重复映射。这些重复调用显式表达策略，同时避免手写每个特化主体。

### Lines 207-232
```cpp
/// methods nor the templated `.to<T>` functions do any kind of casting, they
/// only unwrap the contained value. For example:
///
/// \rst
/// .. code-block:: cpp
///
///   // Make the IValue
///   torch::IValue my_ivalue(26);
///   std::cout << my_ivalue << "\n";
///
///   // Unwrap the IValue
///   int64_t my_int = my_ivalue.toInt();
///   std::cout << my_int << "\n";
///
///   // This will throw an error!
///   // `my_ivalue` is tagged as an int and cannot be used as another type
///   torch::Tensor my_tensor = my_ivalue.toTensor();
/// \endrst
struct TORCH_API IValue final {
  IValue(const IValue& rhs) : IValue(rhs.payload, rhs.tag) {
    if (isIntrusivePtr() &&
        payload.u.as_intrusive_ptr != c10::UndefinedTensorImpl::singleton()) {
      c10::raw::intrusive_ptr::incref(payload.u.as_intrusive_ptr);
    }
  }

```
- EN: Focus symbols: `IValue`, `isIntrusivePtr`, `singleton`, `incref`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`IValue`, `isIntrusivePtr`, `singleton`, `incref`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 233-258
```cpp
  IValue(IValue&& rhs) noexcept : tag(rhs.tag) {
    moveFrom(std::move(rhs));
  }

  /// @private [doxygen private]
  ~IValue() {
    destroy();
  }

  C10_ALWAYS_INLINE IValue& operator=(IValue&& rhs) & noexcept {
    if (&rhs == this) {
      return *this;
    }

    destroy();
    moveFrom(std::move(rhs));
    return *this;
  }

  IValue& operator=(IValue const& rhs) & {
    *this = IValue(rhs);
    return *this;
  }

  void dump() const;

```
- EN: Focus symbols: `IValue`, `tag`, `moveFrom`, `move`, `~IValue`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `tag`, `moveFrom`, `move`, `~IValue`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 259-285
```cpp
  /**
   * Equality comparison. The semantics are the same as Python's `==`:
   * 1. Numerical types are compared by value.
   * 2. Tensors compute element-wise equality, returning a BoolTensor (see:
   * `torch.eq()`)
   * 3. Strings are compared by value.
   * 4. Sequence types (list, tuple) are compared lexicographically by
   *    comparing their elements. Different sequence types never compare equal.
   * 5. Mappings (dict) must have equal (key, value) pairs.
   * 6. If not listed above, the default behavior for is to test identity
   * equality (e.g. pointer equality).
   *
   * Why does this return an IValue instead of a bool? Because in PyTorch,
   * `tensor1 == tensor2` returns a `BoolTensor`, not a bool.
   *
   * NOTE: we (like Python) assume that identity equality implies value equality
   * for efficiency.
   * TODO: need to support customizing equality
   */
  IValue equals(const IValue& rhs) const;
  /**
   * This implements the same semantics as `bool(lhs == rhs)` in Python. which
   * is the same as `equals()` except for Tensor types.
   */
  TORCH_API friend bool operator==(const IValue& lhs, const IValue& rhs);
  TORCH_API friend bool operator!=(const IValue& lhs, const IValue& rhs);

```
- EN: Focus symbols: `BoolTensor`, `eq`, `types`, `Mappings`, `equal`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`BoolTensor`, `eq`, `types`, `Mappings`, `equal`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 286-313
```cpp
  /**
   * Identity comparison. Checks if `this` is the same object as `rhs`. The
   * semantics are the same as Python's `is` operator.
   *
   * NOTE: Like in Python, this operation is poorly defined for primitive types
   * like numbers and strings. Prefer to use `==` unless you really want to
   * check identity equality.
   */
  bool is(const IValue& rhs) const;

  /**
   * Hashing for IValues. Returns an IValue-boxed int.
   *
   * Some notes:
   * - Like eager, Tensors are hashed by looking at the pointer. This is not
   *   strictly correct because two value-equal tensors with different tensor
   *   pointers will hash differently, but we choose to reproduce the eager
   *   semantics.
   * - Hashing is not defined on all built-in IValue types (e.g. list and
   *   dict), following Python. Calling `hash()` on these types will throw.
   */
  IValue hash() const {
    return (int64_t)IValue::hash(*this);
  }
  // This is defined because `c10::hash` dispatches to a function of this
  // signature. See the member function `hash()`.
  static size_t hash(const IValue& iv);

```
- EN: Focus symbols: `is`, `types`, `hash`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is`, `types`, `hash`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 314-337
```cpp
  /**
   * @private [doxygen private]
   * [container equality]
   * This is an equality implementation that assumes objects with the same
   * identity equal themselves, for efficiency reasons. We primarily have this
   * for consistency, because Python does the same thing. This actually
   * provokes user-visible changes in behavior due to quirks in torch:
   *      [tensor1] == [tensor1] -> True (because container equality will first
   * compare identity) [tensor1] == [tensor1_copy] -> RuntimeError:
   * Boolean value of Tensor with more than one value is ambiguous
   */
  TORCH_API friend bool _fastEqualsForContainer(
      const IValue& lhs,
      const IValue& rhs);

 private:
  static bool isAliasOf(const at::Tensor& a, const at::Tensor& b) {
    if (a.is_sparse()) {
      return isAliasOf(a._values(), b) || isAliasOf(a._indices(), b);
    }
    if (b.is_sparse()) {
      return isAliasOf(a, b._values()) || isAliasOf(a, b._indices());
    }
    if (a.is_sparse_csr()) {
```
- EN: Focus symbols: `True`, `_fastEqualsForContainer`, `isAliasOf`, `is_sparse`, `_values`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`True`, `_fastEqualsForContainer`, `isAliasOf`, `is_sparse`, `_values`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 338-361
```cpp
      return isAliasOf(a.values(), b) || isAliasOf(a.crow_indices(), b) ||
          isAliasOf(a.col_indices(), b);
    }
    if (b.is_sparse_csr()) {
      return isAliasOf(a, b.values()) || isAliasOf(a, b.crow_indices()) ||
          isAliasOf(a, b.col_indices());
    }

    // Opaque tensors such as the ones constructed by the MKL-DNN backend
    // don't have storage so we just compare their TensorImpls.
    // TODO: Find way to expose alias info for opaque tensors.
    if (!a.has_storage() || !b.has_storage()) {
      return a.unsafeGetTensorImpl() == b.unsafeGetTensorImpl();
    }

    return a.is_alias_of(b);
  }

  template <typename T>
  bool isListOf() const;

 public:
  /// @private [doxygen private]
  bool isAliasOf(const IValue& rhs) const {
```
- EN: Focus symbols: `isAliasOf`, `values`, `crow_indices`, `col_indices`, `is_sparse_csr`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`isAliasOf`, `values`, `crow_indices`, `col_indices`, `is_sparse_csr`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 362-388
```cpp
    if (this->tag != rhs.tag) {
      // Trivially don't alias if the type is different
      return false;
    }

    // Tensors should be compared based on internal storage
    if (this->isTensor()) {
      return isAliasOf(this->toTensor(), rhs.toTensor());
    }

    if (!isIntrusivePtr()) {
      // Primitive types don't alias anything
      return false;
    }

    AT_ASSERT(rhs.isIntrusivePtr());

    // Other types can be compared by their ptr value
    return this->payload.u.as_intrusive_ptr == rhs.payload.u.as_intrusive_ptr;
  }

  /// @private [doxygen private]
  size_t use_count() const noexcept {
    if (isTensor()) {
      return payload.as_tensor.use_count();
    }

```
- EN: Focus symbols: `isTensor`, `isAliasOf`, `toTensor`, `isIntrusivePtr`, `AT_ASSERT`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isTensor`, `isAliasOf`, `toTensor`, `isIntrusivePtr`, `AT_ASSERT`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 389-412
```cpp
    if (!isIntrusivePtrLegacyBehavior()) {
      return 1;
    }

    if (payload.u.as_intrusive_ptr == c10::UndefinedTensorImpl::singleton()) {
      return 0;
    }
    return c10::raw::intrusive_ptr::use_count(payload.u.as_intrusive_ptr);
  }

  /// @private [doxygen private]
  void swap(IValue& rhs) noexcept {
    if (isTensor() && rhs.isTensor()) {
      std::swap(payload.as_tensor, rhs.payload.as_tensor);
    } else if (isTensor()) {
      at::Tensor t = std::move(payload.as_tensor);
      // As far as I can tell, omitting the usual explicit destructor call
      // is not UB in and of itself, and it's a slight perf win. The
      // destructor is a no-op, because the moved-from Tensor is
      // effectively an intrusive_ptr in the null state, so we don't need
      // the behavior for correctness reasons either. Leaving this
      // explanatory comment, including commented-out destructor call, to
      // make this abundantly clear.
      //
```
- EN: Focus symbols: `isIntrusivePtrLegacyBehavior`, `singleton`, `use_count`, `swap`, `isTensor`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isIntrusivePtrLegacyBehavior`, `singleton`, `use_count`, `swap`, `isTensor`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 413-439
```cpp
      // payload.as_tensor.~Tensor();
      payload.u = rhs.payload.u;
      new (&rhs.payload.as_tensor) at::Tensor(std::move(t));
    } else if (rhs.isTensor()) {
      rhs.swap(*this);
      return;
    } else {
      std::swap(payload.u, rhs.payload.u);
    }
    std::swap(tag, rhs.tag);
  }

  // Accessors for subtypes are arranged together below
  // While some of these accessors could be generated through templates,
  // we prefer to write them manually for clarity

  IValue(at::TensorBase t) : tag(Tag::Tensor) {
    new (&payload.as_tensor) at::Tensor(std::move(t));
  }
  bool isTensor() const {
    return Tag::Tensor == tag;
  }

 private:
  // Outlined error path so that toTensor() can be inlined.
  [[noreturn]] void reportToTensorTypeError() const;

```
- EN: Focus symbols: `Tensor`, `move`, `isTensor`, `swap`, `IValue`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`Tensor`, `move`, `isTensor`, `swap`, `IValue`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 440-465
```cpp
 public:
  at::Tensor toTensor() &&;
  at::Tensor& toTensor() &;
  const at::Tensor& toTensor() const&;
  at::TensorImpl* unsafeToTensorImpl() const {
    TORCH_INTERNAL_ASSERT(isTensor());
    return payload.as_tensor.unsafeGetTensorImpl();
  }

  IValue(at::Storage s) : tag(Tag::Storage) {
    payload.u.as_intrusive_ptr =
        null_to_undefined_tensor(s.unsafeReleaseStorageImpl());
  }
  bool isStorage() const {
    return Tag::Storage == tag;
  }
  c10::Storage toStorage() &&;
  c10::Storage toStorage() const&;

  const IValue& toIValue() const {
    return *this;
  }
  IValue& toIValue() {
    return *this;
  }

```
- EN: Focus symbols: `toTensor`, `unsafeToTensorImpl`, `TORCH_INTERNAL_ASSERT`, `isTensor`, `unsafeGetTensorImpl`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`toTensor`, `unsafeToTensorImpl`, `TORCH_INTERNAL_ASSERT`, `isTensor`, `unsafeGetTensorImpl`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 466-493
```cpp
  /// @private [doxygen private]
  IValue(intrusive_ptr<caffe2::Blob> blob) : tag(Tag::Blob) {
    // TODO (after Tensor merge) If we pass in a Blob holding a Tensor, extract
    // and store it as a Tensor instead.
    payload.u.as_intrusive_ptr = null_to_undefined_tensor(blob.release());
  }

  /// @private [doxygen private]
  bool isBlob() const {
    return Tag::Blob == tag;
  }

  /// @private [doxygen private]
  c10::intrusive_ptr<caffe2::Blob> toBlob() &&;

  /// @private [doxygen private]
  c10::intrusive_ptr<caffe2::Blob> toBlob() const&;

  // Capsule. No new callsites of these APIs should
  // be introduced.
  static inline IValue make_capsule(
      intrusive_ptr<torch::CustomClassHolder> blob);
  bool isCapsule() const {
    return Tag::Capsule == tag;
  }
  c10::intrusive_ptr<torch::CustomClassHolder> toCapsule() &&;
  c10::intrusive_ptr<torch::CustomClassHolder> toCapsule() const&;

```
- EN: Focus symbols: `IValue`, `tag`, `null_to_undefined_tensor`, `release`, `isBlob`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `tag`, `null_to_undefined_tensor`, `release`, `isBlob`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 494-517
```cpp
  // Custom C++ classes
  template <
      typename T,
      std::enable_if_t<std::is_base_of_v<torch::CustomClassHolder, T>, int> = 0>
  IValue(intrusive_ptr<T> custom_class);
  bool isCustomClass() const;
  template <typename T>
  c10::intrusive_ptr<T> toCustomClass() &&;
  template <typename T>
  c10::intrusive_ptr<T> toCustomClass() const&;

  // Tuple
  IValue(c10::intrusive_ptr<ivalue::Tuple> v);

  template <
      typename... Args,
      std::enable_if_t<
          !std::disjunction_v<
              std::is_lvalue_reference<Args>...,
              std::negation<std::is_constructible<IValue, Args>>...>,
          std::nullptr_t> = nullptr>
  IValue(const std::tuple<Args...>& t);
  template <
      typename... Args,
```
- EN: Focus symbols: `IValue`, `isCustomClass`, `toCustomClass`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`IValue`, `isCustomClass`, `toCustomClass`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 518-541
```cpp
      std::enable_if_t<
          !std::disjunction_v<
              std::is_lvalue_reference<Args>...,
              std::negation<std::is_constructible<IValue, Args>>...>,
          std::nullptr_t> = nullptr>
  IValue(std::tuple<Args...>&& t);
  bool isTuple() const {
    return Tag::Tuple == tag;
  }
  c10::intrusive_ptr<ivalue::Tuple> toTuple() &&;
  c10::intrusive_ptr<ivalue::Tuple> toTuple() const&;
  [[nodiscard]] ivalue::Tuple& toTupleRef() const;

  // Double
  IValue(double d) : tag(Tag::Double) {
    payload.u.as_double = d;
  }
  bool isDouble() const {
    return Tag::Double == tag;
  }
  double toDouble() const {
    if (isDouble()) {
      return payload.u.as_double;
    } else if (isSymFloat()) {
```
- EN: Focus symbols: `IValue`, `isTuple`, `toTuple`, `toTupleRef`, `tag`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `isTuple`, `toTuple`, `toTupleRef`, `tag`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 542-565
```cpp
      return toSymFloat().guard_float(__FILE__, __LINE__);
    } else {
      TORCH_INTERNAL_ASSERT(0, "expected double");
    }
  }

  // ComplexDouble
  template <typename T>
  IValue(c10::complex<T> c);
  bool isComplexDouble() const {
    return Tag::ComplexDouble == tag;
  }
  c10::complex<double> toComplexDouble() const;

  // Future
  IValue(c10::intrusive_ptr<ivalue::Future> v);
  bool isFuture() const {
    return Tag::Future == tag;
  }
  c10::intrusive_ptr<ivalue::Future> toFuture() &&;
  c10::intrusive_ptr<ivalue::Future> toFuture() const&;

  IValue(c10::intrusive_ptr<ivalue::Await> v);
  bool isAwait() const {
```
- EN: Focus symbols: `toSymFloat`, `guard_float`, `TORCH_INTERNAL_ASSERT`, `IValue`, `isComplexDouble`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`toSymFloat`, `guard_float`, `TORCH_INTERNAL_ASSERT`, `IValue`, `isComplexDouble`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 566-591
```cpp
    return Tag::Await == tag;
  }
  c10::intrusive_ptr<ivalue::Await> toAwait() &&;
  c10::intrusive_ptr<ivalue::Await> toAwait() const&;

  // RRef
  IValue(c10::intrusive_ptr<c10::RRefInterface> v);
  bool isRRef() const {
    return Tag::RRef == tag;
  }
  c10::intrusive_ptr<c10::RRefInterface> toRRef() &&;
  c10::intrusive_ptr<c10::RRefInterface> toRRef() const&;

  // Quantizer
  IValue(c10::intrusive_ptr<at::Quantizer> v);
  bool isQuantizer() const {
    return Tag::Quantizer == tag;
  }
  c10::intrusive_ptr<at::Quantizer> toQuantizer() &&;
  c10::intrusive_ptr<at::Quantizer> toQuantizer() const&;

  // Int
  IValue(int64_t i) : tag(Tag::Int) {
    payload.u.as_int = i;
  }

```
- EN: Focus symbols: `toAwait`, `IValue`, `isRRef`, `toRRef`, `isQuantizer`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toAwait`, `IValue`, `isRRef`, `toRRef`, `isQuantizer`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 592-618
```cpp
  IValue(const c10::SymInt& i) {
    if (auto mi = i.maybe_as_int()) {
      tag = Tag::Int;
      payload.u.as_int = *mi;
    } else {
      tag = Tag::SymInt;
      payload.u.as_intrusive_ptr = i.toSymNode().release();
    }
  }

  bool isSymInt() const {
    return Tag::SymInt == tag;
  }

  c10::SymInt toSymInt() &&;
  c10::SymInt toSymInt() const&;

  IValue(const c10::SymFloat& i) {
    if (i.is_symbolic()) {
      tag = Tag::SymFloat;
      payload.u.as_intrusive_ptr = i.toSymNodeImpl().release();
    } else {
      tag = Tag::Double;
      payload.u.as_double = i.as_float_unchecked();
    }
  }

```
- EN: Focus symbols: `IValue`, `maybe_as_int`, `toSymNode`, `release`, `isSymInt`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `maybe_as_int`, `toSymNode`, `release`, `isSymInt`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 619-646
```cpp
  bool isSymFloat() const {
    return Tag::SymFloat == tag;
  }

  c10::SymFloat toSymFloat() &&;
  c10::SymFloat toSymFloat() const&;

  IValue(const c10::SymBool& i) {
    if (auto mi = i.maybe_as_bool()) {
      tag = Tag::Bool;
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
      payload.u.as_int = *mi;
#elif __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
      /* due to byteorder if value assigned as_int, as_bool actually is not set correctly */
      payload.u.as_bool = *mi;
#else
#error Unexpected or undefined __BYTE_ORDER__
#endif
    } else {
      tag = Tag::SymBool;
      payload.u.as_intrusive_ptr = i.toSymNodeImpl().release();
    }
  }

  bool isSymBool() const {
    return Tag::SymBool == tag;
  }

```
- EN: Focus symbols: `isSymFloat`, `toSymFloat`, `IValue`, `maybe_as_bool`, `toSymNodeImpl`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isSymFloat`, `toSymFloat`, `IValue`, `maybe_as_bool`, `toSymNodeImpl`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 647-671
```cpp
  c10::SymBool toSymBool() &&;
  c10::SymBool toSymBool() const&;

  // allow you to pass literals (3, 4) without ambiguity
  IValue(int32_t i) : IValue(static_cast<int64_t>(i)) {}

  bool isInt() const {
    return Tag::Int == tag;
  }

  int64_t toInt() const {
    if (isInt()) {
      return payload.u.as_int;
    } else if (isSymInt()) {
      return toSymInt().guard_int(__FILE__, __LINE__);
    } else {
      TORCH_INTERNAL_ASSERT(0, "expected int");
    }
  }

  // Unsigned
  IValue(uint64_t u) : tag( u <= std::numeric_limits<int64_t>::max() ? Tag::Int : Tag::UInt) {
    payload.u.as_uint = u;
  }

```
- EN: Focus symbols: `toSymBool`, `IValue`, `isInt`, `toInt`, `isSymInt`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`toSymBool`, `IValue`, `isInt`, `toInt`, `isSymInt`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 672-695
```cpp

  // See Note [Meaning of HAS_u]
  // IValue type model closely follows that of c10::Scalar
  // Where all integers are upcast to 64-bit representation, and `as_int` is used as default
  // representation unless value could not be represented as signed int
  bool isUnsigned() const {
    return Tag::UInt == tag || (Tag::Int == tag && payload.u.as_int >= 0);
  }

  uint64_t toUInt() const {
    if (isUnsigned()) {
      return payload.u.as_uint;
    } else {
      TORCH_INTERNAL_ASSERT(0, "expected unsigned int");
    }
  }


  // Bool
  IValue(bool b) : tag(Tag::Bool) {
#if defined(__clang__) && defined(__x86_64__)
    // Initializing entire payload stops valgrind's from reporting
    // "jump or move depends on uninitialised value" in IValue copy constructor
    // See https://github.com/pytorch/pytorch/issues/37117
```
- EN: Focus symbols: `isUnsigned`, `toUInt`, `TORCH_INTERNAL_ASSERT`, `IValue`, `tag`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`isUnsigned`, `toUInt`, `TORCH_INTERNAL_ASSERT`, `IValue`, `tag`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 696-719
```cpp
    payload.u.as_int = b;
#else
    payload.u.as_bool = b;
#endif
  }
  bool isBool() const {
    return Tag::Bool == tag;
  }
  bool toBool() const {
    if (isBool()) {
      return payload.u.as_bool;
    } else if (isSymBool()) {
      return toSymBool().guard_bool(__FILE__, __LINE__);
    } else {
      TORCH_INTERNAL_ASSERT(0, "expected bool");
    }
  }

  // IntList
  bool isIntList() const;
  bool isSymIntList() const;
  c10::List<int64_t> toIntList() &&;
  c10::List<int64_t> toIntList() const&;
  std::vector<int64_t> toIntVector() const;
```
- EN: Focus symbols: `isBool`, `toBool`, `isSymBool`, `toSymBool`, `guard_bool`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`isBool`, `toBool`, `isSymBool`, `toSymBool`, `guard_bool`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 720-745
```cpp
  c10::List<c10::SymInt> toSymIntList() &&;
  c10::List<c10::SymInt> toSymIntList() const&;
  std::vector<c10::SymInt> toSymIntVector() const;
  at::DimVector toDimVector() const;

  // ConstantString
  IValue(c10::intrusive_ptr<ivalue::ConstantString> v);
  IValue(std::string v);
  IValue(const char* v) : IValue(std::string(v)) {}
  IValue(std::string_view v) : IValue(std::string(v)){}
  bool isString() const {
    return Tag::String == tag;
  }
  c10::intrusive_ptr<ivalue::ConstantString> toString() &&;
  c10::intrusive_ptr<ivalue::ConstantString> toString() const&;
  const std::string& toStringRef() const;
  std::optional<std::reference_wrapper<const std::string>> toOptionalStringRef()
      const;
  std::string_view toStringView() const;

  // DoubleList
  bool isDoubleList() const;
  c10::List<double> toDoubleList() &&;
  c10::List<double> toDoubleList() const&;
  std::vector<double> toDoubleVector() const;

```
- EN: Focus symbols: `toSymIntList`, `toSymIntVector`, `toDimVector`, `IValue`, `string`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toSymIntList`, `toSymIntVector`, `toDimVector`, `IValue`, `string`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 746-769
```cpp
  // ComplexDoubleList
  bool isComplexDoubleList() const;
  c10::List<c10::complex<double>> toComplexDoubleList() &&;
  c10::List<c10::complex<double>> toComplexDoubleList() const&;
  std::vector<c10::complex<double>> toComplexDoubleVector() const;

  // BoolList
  bool isBoolList() const;
  c10::List<bool> toBoolList() &&;
  c10::List<bool> toBoolList() const&;

  // TensorList
  bool isTensorList() const;
  c10::List<at::Tensor> toTensorList() &&;
  c10::List<at::Tensor> toTensorList() const&;
  std::vector<at::Tensor> toTensorVector() const;

  // OptionalTensorList
  bool isOptionalTensorList() const;
  c10::List<std::optional<at::Tensor>> toOptionalTensorList() &&;
  c10::List<std::optional<at::Tensor>> toOptionalTensorList() const&;
  std::vector<std::optional<at::Tensor>> toOptionalTensorVector() const;

  // GenericList
```
- EN: Focus symbols: `isComplexDoubleList`, `toComplexDoubleList`, `toComplexDoubleVector`, `isBoolList`, `toBoolList`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isComplexDoubleList`, `toComplexDoubleList`, `toComplexDoubleVector`, `isBoolList`, `toBoolList`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 770-796
```cpp
  IValue(c10::List<IValue> v);
  bool isList() const {
    return Tag::GenericList == tag;
  }
  c10::List<IValue> toList() &&;
  c10::List<IValue> toList() const&;
  c10::ArrayRef<IValue> toListRef() const;

  // Some template constructors of IValue calls another constructor recursively.
  // This SFINAEs the called constructor exists.
  template <class T>
  using enable_if_ivalue_constructible =
      std::enable_if_t<std::is_constructible_v<IValue, T>, std::nullptr_t>;

  // The rule for lists is more complicated; the generic constructor is only
  // acceptable if your element isn't SymInt.  If you do have a SymInt element,
  // then you must also, at construction time, check if you can decay the list
  // into an int list (this is MANDATORY, as at a use site we may expect
  // toIntList to work even if at the call site you had a SymIntArrayRef
  // argument).  In practice, only SymIntArrayRef is used this way, so we
  // didn't bother making it work for the other constructors, we just make sure
  // they're not selectable.
  template <class T>
  using enable_if_list_is_ivalue_constructible = std::enable_if_t<
      std::is_constructible_v<IValue, T> && !std::is_same_v<T, c10::SymInt>,
      std::nullptr_t>;

```
- EN: Focus symbols: `T`, `enable_if_ivalue_constructible`, `enable_if_list_is_ivalue_constructible`, `IValue`, `isList`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`T`, `enable_if_ivalue_constructible`, `enable_if_list_is_ivalue_constructible`, `IValue`, `isList`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 797-820
```cpp
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(c10::List<T>&& v);
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(const c10::List<T>& v);
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(at::ArrayRef<T> v);
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(const std::vector<T>& v);
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(std::vector<T>&& v);
  template <class T, size_t N>
  IValue(std::array<T, N> v);

  // Manual constructors for lists of symints, which decay to int list if
  // possible.  To avoid ambiguous overload situations, we template them
  // to prevent implicit conversions
  template <class T>
  using enable_if_symint =
      std::enable_if_t<std::is_same_v<T, c10::SymInt>, std::nullptr_t>;

  template <class T, enable_if_symint<T> = nullptr>
  IValue(at::ArrayRef<T> v);
  template <class T, enable_if_symint<T> = nullptr>
  IValue(at::OptionalArrayRef<T> v);
```
- EN: Focus symbols: `T`, `enable_if_symint`, `IValue`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`T`, `enable_if_symint`, `IValue`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 821-846
```cpp
  template <class T, enable_if_symint<T> = nullptr>
  IValue(const std::vector<T>& v);
  template <class T, enable_if_symint<T> = nullptr>
  IValue(std::vector<T>&& v);

  template <class T>
  using enable_if_ilist_is_ivalue_constructible = std::enable_if_t<
      std::is_constructible_v<IValue, T> &&
          std::is_constructible_v<IValue, typename IListRef<T>::boxed_type> &&
          !std::is_same_v<T, c10::SymInt>,
      std::nullptr_t>;

  template <class T, enable_if_ilist_is_ivalue_constructible<T> = nullptr>
  IValue(c10::IListRef<T> v);

  // GenericDict
  IValue(c10::Dict<IValue, IValue> v);
  bool isGenericDict() const {
    return Tag::GenericDict == tag;
  }
  c10::Dict<IValue, IValue> toGenericDict() &&;
  c10::Dict<IValue, IValue> toGenericDict() const&;

  template <class Key, class Value>
  IValue(c10::Dict<Key, Value> v);

```
- EN: Focus symbols: `T`, `Key`, `Value`, `enable_if_ilist_is_ivalue_constructible`, `IValue`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`T`, `Key`, `Value`, `enable_if_ilist_is_ivalue_constructible`, `IValue`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 847-872
```cpp
  template <class Key, class Value>
  /// \cond
  /// DOXYGEN_CANNOT_HANDLE_CONSTRUCTORS_WITH_MACROS_SO_EXCLUDE_THIS_LINE_FROM_DOXYGEN
  C10_DEPRECATED_MESSAGE(
      "IValues based on std::unordered_map<K, V> are slow and deprecated. Please use c10::Dict<K, V> instead.")
      /// \endcond
      IValue(std::unordered_map<Key, Value> v);

  template <class T, enable_if_ivalue_constructible<T> = nullptr>
  IValue(std::optional<T> v);
  template <class T, enable_if_list_is_ivalue_constructible<T> = nullptr>
  IValue(c10::OptionalArrayRef<T> v);
  IValue(std::nullopt_t /*unused*/);

  // ClassType
  IValue(c10::intrusive_ptr<ivalue::Object> v);
  bool isObject() const {
    return tag == Tag::Object;
  }
  c10::intrusive_ptr<ivalue::Object> toObject() &&;
  c10::intrusive_ptr<ivalue::Object> toObject() const&;
  ivalue::Object& toObjectRef() const;

  torch::jit::Module toModule() const;
  bool isModule() const;

```
- EN: Focus symbols: `Key`, `Value`, `T`, `C10_DEPRECATED_MESSAGE`, `IValue`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Key`, `Value`, `T`, `C10_DEPRECATED_MESSAGE`, `IValue`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 873-899
```cpp
  // PyObject
  IValue(c10::intrusive_ptr<ivalue::PyObjectHolder> v);
  bool isPyObject() const {
    return tag == Tag::PyObject;
  }
  c10::intrusive_ptr<ivalue::PyObjectHolder> toPyObjectHolder() &&;
  c10::intrusive_ptr<ivalue::PyObjectHolder> toPyObjectHolder() const&;
  PyObject* toPyObject() const;

  // Enum
  explicit IValue(c10::intrusive_ptr<ivalue::EnumHolder> v);
  bool isEnum() const {
    return tag == Tag::Enum;
  }
  c10::intrusive_ptr<ivalue::EnumHolder> toEnumHolder() &&;
  c10::intrusive_ptr<ivalue::EnumHolder> toEnumHolder() const&;

  // None
  IValue() = default;
  bool isNone() const {
    return Tag::None == tag;
  }
  std::string toNone() const {
    AT_ASSERT(isNone());
    return "None";
  }

```
- EN: Focus symbols: `IValue`, `isPyObject`, `toPyObjectHolder`, `toPyObject`, `isEnum`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `isPyObject`, `toPyObjectHolder`, `toPyObject`, `isEnum`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 900-923
```cpp
  static IValue uninitialized() {
    auto i = IValue();
    i.tag = Tag::Uninitialized;
    return i;
  }

  // Scalar, which gets encoded as either an Int, a Double or a ComplexDouble
  IValue(const at::Scalar& s) : IValue() {
    // NB: do the symbolic versions first, as isFloatingPoint is true
    // for both SymFloat and double
    if (s.isSymInt()) {
      tag = Tag::SymInt;
      payload.u.as_intrusive_ptr = s.toSymInt().toSymNode().release();
    } else if (s.isSymFloat()) {
      tag = Tag::SymFloat;
      payload.u.as_intrusive_ptr = s.toSymFloat().toSymNodeImpl().release();
    } else if (s.isSymBool()) {
      tag = Tag::SymBool;
      payload.u.as_intrusive_ptr = s.toSymBool().toSymNodeImpl().release();
    } else if (s.isFloatingPoint()) {
      tag = Tag::Double;
      payload.u.as_double = s.toDouble();
    } else if (s.isComplex()) {
      *this = s.toComplexDouble();
```
- EN: Focus symbols: `uninitialized`, `IValue`, `isSymInt`, `toSymInt`, `toSymNode`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`uninitialized`, `IValue`, `isSymInt`, `toSymInt`, `toSymNode`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 924-947
```cpp
    } else if (s.isBoolean()) {
      tag = Tag::Bool;
      payload.u.as_bool = s.toBool();
    } else {
      TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
          s.isIntegral(false), "Unknown type in Scalar");
      if (s.isUnsigned()) {
        const auto val = s.toUInt64();
        payload.u.as_uint = val;
        tag = val <= std::numeric_limits<int64_t>::max() ? Tag::Int : Tag::UInt;
      } else {
        payload.u.as_int = s.toLong();
        tag = Tag::Int;
      }
    }
  }

  bool isScalar() const {
    return isDouble() || isInt() || isComplexDouble() || isBool() ||
        isSymInt() || isSymFloat() || isSymBool();
  }

  at::Scalar toScalar() const {
    if (isDouble())
```
- EN: Focus symbols: `isBoolean`, `toBool`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `isIntegral`, `isUnsigned`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`isBoolean`, `toBool`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`, `isIntegral`, `isUnsigned`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 948-971
```cpp
      return toDouble();
    else if (isInt())
      return toInt();
    else if (isComplexDouble())
      return toComplexDouble();
    else if (isBool())
      return toBool();
    else if (isSymInt())
      return toSymInt();
    else if (isSymFloat())
      return toSymFloat();
    else if (isSymBool())
      return toSymBool();
    else if (isUnsigned())
      return toUInt();
    TORCH_CHECK(false, "IValue is not a Scalar");
  }

  // Device
  IValue(c10::Device d) : tag(Tag::Device) {
    payload.u.as_device.type = d.type();
    payload.u.as_device.index = d.index();
  }
  bool isDevice() const {
```
- EN: Focus symbols: `toDouble`, `isInt`, `toInt`, `isComplexDouble`, `toComplexDouble`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`toDouble`, `isInt`, `toInt`, `isComplexDouble`, `toComplexDouble`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 972-996
```cpp
    return Tag::Device == tag;
  }
  c10::Device toDevice() const {
    AT_ASSERT(isDevice());
    return c10::Device(payload.u.as_device.type, payload.u.as_device.index);
  }

  // Stream
  IValue(c10::Stream s) : tag(Tag::Stream) {
    auto v = c10::make_intrusive<ivalue::StreamData3Holder>(s.pack3());
    payload.u.as_intrusive_ptr = v.release();
  }
  c10::Stream toStream() &&;
  c10::Stream toStream() const&;
  bool isStream() const {
    return Tag::Stream == tag;
  }

  // ScalarType
  IValue(ScalarType t)
      : IValue(static_cast<std::underlying_type_t<ScalarType>>(t)) {}
  at::ScalarType toScalarType() const {
    return static_cast<at::ScalarType>(toInt());
  }

```
- EN: Focus symbols: `toDevice`, `AT_ASSERT`, `isDevice`, `Device`, `IValue`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`toDevice`, `AT_ASSERT`, `isDevice`, `Device`, `IValue`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 997-1021
```cpp
  // Layout
  IValue(Layout l) : IValue(static_cast<std::underlying_type_t<Layout>>(l)) {}
  at::Layout toLayout() const {
    return static_cast<at::Layout>(toInt());
  }

  // MemoryFormat
  IValue(MemoryFormat m)
      : IValue(static_cast<std::underlying_type_t<MemoryFormat>>(m)) {}
  at::MemoryFormat toMemoryFormat() const {
    return static_cast<at::MemoryFormat>(toInt());
  }

  // QScheme
  IValue(at::QScheme qscheme) : tag(Tag::Int) {
    payload.u.as_int = static_cast<int64_t>(qscheme);
  }

  at::QScheme toQScheme() const {
    return static_cast<at::QScheme>(toInt());
  }

  // Dimname
  IValue(at::Dimname dimname) : IValue(dimname.symbol().toQualString()) {}

```
- EN: Focus symbols: `IValue`, `toLayout`, `toInt`, `toMemoryFormat`, `tag`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `toLayout`, `toInt`, `toMemoryFormat`, `tag`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1022-1048
```cpp
  at::Dimname toDimname() const {
    return at::Dimname::fromSymbol(Symbol::fromQualString(toStringRef()));
  }

  // Generator
  IValue(at::Generator g) : tag(Tag::Generator) {
    payload.u.as_intrusive_ptr =
        null_to_undefined_tensor(g.unsafeReleaseGeneratorImpl());
  }
  bool isGenerator() const {
    return Tag::Generator == tag;
  }
  at::Generator toGenerator() &&;
  at::Generator toGenerator() const&;

  // for debugging
  std::string tagKind() const {
    switch (tag) {
#define DEFINE_CASE(x) \
  case Tag::x:         \
    return #x;
      TORCH_FORALL_TAGS(DEFINE_CASE)
#undef DEFINE_CASE
    }
    return "InvalidTag(" + std::to_string(static_cast<int>(tag)) + ")";
  }

```
- EN: Focus symbols: `DEFINE_CASE`, `toDimname`, `fromSymbol`, `fromQualString`, `toStringRef`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`DEFINE_CASE`, `toDimname`, `fromSymbol`, `fromQualString`, `toStringRef`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1049-1075
```cpp
  // generic v.to<at::Tensor>() implementations
  // that can be used in special functions like pop/push
  // that use template meta-programming.
  // prefer the directly named methods when you can,
  // since they are simpler to understand

  // Note: if you get linker errors saying one of these is missing,
  // change it to ... && = delete; and you will see better error messages for
  // why However, we cannot commit this because some compiler versions barf on
  // it.
  template <typename T>
  T to() &&;
  template <typename T>
  typename c10::detail::ivalue_to_const_ref_overload_return<T>::type to()
      const&;

  // ToOptional: convert a IValue to the Optional obj that accepts both T and
  // None
  template <typename T>
  std::optional<T> toOptional();
  template <typename T>
  std::optional<T> toOptional() const;

  /// @private [doxygen private]
  /// this is a shallow comparison of two IValues to test the object identity
  bool isSameIdentity(const IValue& rhs) const;

```
- EN: Focus symbols: `to`, `toOptional`, `isSameIdentity`. This block builds generic/template machinery for multiple dtypes, devices, or call patterns. Templates let ATen share one design across many concrete instantiations.
- CN: 关注符号：`to`, `toOptional`, `isSameIdentity`。该代码块构建适配多种数据类型、设备或调用模式的模板机制。模板让 ATen 能在多个具体实例之间共享同一套设计。

### Lines 1076-1099
```cpp
  // Computes the "official" string representation of an IValue. This produces a
  // TorchScript expression that can be used to recreate an IValue with the same
  // value (e.g. when we are printing constants in the serializer).
  //
  // Callers can use `customFormatter` to override how `repr()` prints out an
  // IValue. This is useful if you have some other environment where you can
  // look up values, and you want to print a reference to that environment (like
  // the serializer's constant table).
  //
  // repr() is not necessarily defined on all objects!
  std::ostream& repr(
      std::ostream& stream,
      std::function<bool(std::ostream&, const IValue& v)> customFormatter)
      const;

  // Computes an "informal" string representation of an IValue. This should be
  // used for debugging, or servicing `print()`-like functions.
  // This is different from `repr()` in that there is no expectation that we can
  // exactly reconstruct an IValue from the output; feel free to use a
  // concise/pretty form
  TORCH_API friend std::ostream& operator<<(std::ostream& out, const IValue& v);

  bool isPtrType() const {
    if (isTensor()) {
```
- EN: Focus symbols: `repr`, `bool`, `isPtrType`, `isTensor`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`repr`, `bool`, `isPtrType`, `isTensor`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1100-1123
```cpp
      return payload.as_tensor.defined();
    }
    return isIntrusivePtrLegacyBehavior();
  }

  /// @private [doxygen private]
  const void* internalToPointer() const {
    TORCH_INTERNAL_ASSERT(
        isPtrType(), "Can only call internalToPointer() for pointer types");
    if (isTensor()) {
      return payload.as_tensor.unsafeGetTensorImpl();
    } else {
      return payload.u.as_intrusive_ptr != c10::UndefinedTensorImpl::singleton()
          ? payload.u.as_intrusive_ptr
          : nullptr;
    }
  }

  template <typename T = c10::PlatformType>
  TypePtr type() const;

  // Detect aliased tensors.
  struct HashAliasedIValue {
    size_t hashTensor(const at::Tensor& ten) const {
```
- EN: Focus symbols: `HashAliasedIValue`, `defined`, `isIntrusivePtrLegacyBehavior`, `internalToPointer`, `TORCH_INTERNAL_ASSERT`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HashAliasedIValue`, `defined`, `isIntrusivePtrLegacyBehavior`, `internalToPointer`, `TORCH_INTERNAL_ASSERT`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1124-1147
```cpp
      if (ten.is_sparse()) {
        // COO sparse tensors have a "values" tensor and an "indices" tensor
        // so this will detect overlap of sparse tensors that share a values
        // tensor, but not sparse tensors that share an indices tensor.
        return hashTensor(ten._values());
      } else if (ten.is_sparse_csr()) {
        // COO sparse tensors have a "values" tensor and an "indices" tensor
        // so this will detect overlap of sparse tensors that share a values
        // tensor, but not sparse tensors that share an indices tensor.
        return hashTensor(ten.values());
      } else if (!ten.has_storage()) {
        // Opaque tensors such as the ones constructed by the MKL-DNN backend
        // don't have storage so we just use their TensorImpls.
        // TODO: Find way to expose alias info for opaque tensors.
        return reinterpret_cast<size_t>(ten.unsafeGetTensorImpl());
      } else {
        return reinterpret_cast<size_t>(ten.storage().unsafeGetStorageImpl());
      }
    }
    size_t operator()(const IValue& val) const {
      if (val.isTensor()) {
        return hashTensor(val.toTensor());
      }
      // If it is not a Tensor, then two mutable IValues alias each other only
```
- EN: Focus symbols: `is_sparse`, `hashTensor`, `_values`, `is_sparse_csr`, `values`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_sparse`, `hashTensor`, `_values`, `is_sparse_csr`, `values`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1148-1175
```cpp
      // if they are the same pointer.
      return val.payload.u.as_int;
    }
  };

  struct CompAliasedIValues {
    bool operator()(const IValue& lhs, const IValue& rhs) const {
      return lhs.isAliasOf(rhs);
    }
  };

  using HashAliasedIValues =
      std::unordered_set<IValue, HashAliasedIValue, CompAliasedIValues>;
  using HashAliasedIValueMap =
      std::unordered_map<IValue, IValue, HashAliasedIValue, CompAliasedIValues>;

  struct HashIdentityIValue {
    size_t operator()(const IValue& val) const {
      return val.payload.u.as_int;
    }
  };

  struct CompIdentityIValues {
    bool operator()(const IValue& lhs, const IValue& rhs) const {
      return lhs.is(rhs);
    }
  };

```
- EN: Focus symbols: `CompAliasedIValues`, `HashIdentityIValue`, `CompIdentityIValues`, `HashAliasedIValues`, `HashAliasedIValueMap`, `operator`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`CompAliasedIValues`, `HashIdentityIValue`, `CompIdentityIValues`, `HashAliasedIValues`, `HashAliasedIValueMap`, `operator`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1176-1199
```cpp
  using HashIdentityIValues =
      std::unordered_set<IValue, HashIdentityIValue, CompIdentityIValues>;
  using HashIdentityIValueMap =
      std::unordered_map<IValue, IValue, HashIdentityIValue, CompIdentityIValues>;

  // Checks if this and rhs has a subvalues in common.
  // [t1,t2] and [t2, t3] returns true.
  bool overlaps(const IValue& rhs) const;

  // Inserts all subvalues of this in subValues.
  void getSubValues(HashAliasedIValues& subValues) const;

  // Apply visitor to every subvalue.
  // TODO: There are several places that recurse over IValue. This is fragile.
  // This visitor should be used to recurse over ivalues.
  void visit(const std::function<bool(const IValue&)>& visitor) const;
  IValue deepcopy(std::optional<at::Device> device = std::nullopt) const;
  IValue deepcopy(
      HashIdentityIValueMap& memo,
      std::optional<at::Device> device = std::nullopt) const;

 private:
  static c10::intrusive_ptr_target* null_to_undefined_tensor(
      c10::intrusive_ptr_target* p) {
```
- EN: Focus symbols: `HashIdentityIValues`, `HashIdentityIValueMap`, `overlaps`, `getSubValues`, `visit`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`HashIdentityIValues`, `HashIdentityIValueMap`, `overlaps`, `getSubValues`, `visit`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1200-1223
```cpp
    return p ? p
             : static_cast<c10::intrusive_ptr_target*>(
                   c10::UndefinedTensorImpl::singleton());
  }

  static bool ptrEqual(const IValue& lhs, const IValue& rhs);
  // NOTE: IValue tags are intentionally private. In the future we may encode
  // this value different (e.g. using NaN boxing), and this would make it more
  // costly to determine the tag for all types vs just determining if something
  // is a particular type. Instead we want clients to use the `isX` methods when
  // possible. If for performance reasons you really, absolutely, must have a jump
  // table, then we can revisit this.
  enum class Tag : uint32_t {
#define DEFINE_TAG(x) x,
    TORCH_FORALL_TAGS(DEFINE_TAG)
#undef DEFINE_TAG
  };

#define COUNT_TAG(x) 1 +
  static constexpr auto kNumTags = TORCH_FORALL_TAGS(COUNT_TAG) 0;
#undef COUNT_TAG

  template <
      class T,
```
- EN: Focus symbols: `Tag`, `T`, `DEFINE_TAG`, `COUNT_TAG`, `singleton`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`Tag`, `T`, `DEFINE_TAG`, `COUNT_TAG`, `singleton`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1224-1247
```cpp
      class NullType = c10::detail::intrusive_target_default_null_type<T>>
  c10::intrusive_ptr<T, NullType> moveToIntrusivePtr();
  template <
      typename T,
      class NullType = c10::detail::intrusive_target_default_null_type<T>>
  c10::intrusive_ptr<T, NullType> toIntrusivePtr() const;

  void destroy() {
    // We carefully construct this call to both 1) avoid UB by using
    // the "wrong" one of as_tensor and as_intrusive_ptr and 2) enable
    // the compiler to generate the same code for each case. It is
    // surprisingly difficult to get this right.
    if (isTensor() || isIntrusivePtr()) {
      c10::intrusive_ptr_target* p = isTensor()
          ? payload.as_tensor.unsafeGetTensorImpl()
          : payload.u.as_intrusive_ptr;
      c10::intrusive_ptr<intrusive_ptr_target, c10::UndefinedTensorImpl>::
          reclaim(p);
      // No need to make this destructor call!
      // payload.as_tensor.~Tensor();
    }
  }

  // NOLINTNEXTLINE(cppcoreguidelines-rvalue-reference-param-not-moved)
```
- EN: Focus symbols: `NullType`, `moveToIntrusivePtr`, `toIntrusivePtr`, `destroy`, `isTensor`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`NullType`, `moveToIntrusivePtr`, `toIntrusivePtr`, `destroy`, `isTensor`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1248-1271
```cpp
  C10_ALWAYS_INLINE void moveFrom(IValue&& rhs) noexcept {
    if (rhs.isTensor()) {
      new (&payload.as_tensor) at::Tensor(std::move(rhs.payload.as_tensor));
      // As far as I can tell, omitting the usual explicit destructor call
      // is not UB in and of itself, and it's a slight perf win. The
      // destructor is a no-op, because the moved-from Tensor is
      // effectively an intrusive_ptr in the null state, so we don't need
      // the behavior for correctness reasons either. Leaving this
      // explanatory comment, including commented-out destructor call, to
      // make this abundantly clear.
      //
      // rhs.payload.as_tensor.~Tensor();
    } else {
      payload.u = rhs.payload.u;
    }
    tag = rhs.tag;
    rhs.clearToNone();
  }

  void clearToNone() noexcept {
    payload.u.as_int = 0;
    tag = Tag::None;
  }

```
- EN: Focus symbols: `moveFrom`, `isTensor`, `Tensor`, `move`, `clearToNone`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`moveFrom`, `isTensor`, `Tensor`, `move`, `clearToNone`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1272-1295
```cpp
 private:
  // This is the source of truth for isIntrusivePtr; edit results here
  // as needed and isIntrusivePtr will pick them up.
  // NOLINTBEGIN(bugprone-branch-clone)
  static constexpr bool isIntrusivePtrConstexpr(Tag tag) {
    switch (tag) {
      case Tag::None:
        return false;
      case Tag::Tensor:
        return false;
      case Tag::Storage:
        return true;
      case Tag::Generator:
        return true;
      case Tag::Double:
        return false;
      case Tag::ComplexDouble:
        return true;
      case Tag::Int:
        return false;
      case Tag::UInt:
        return false;
      case Tag::SymInt:
        return true;
```
- EN: Focus symbols: `isIntrusivePtrConstexpr`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isIntrusivePtrConstexpr`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1296-1319
```cpp
      case Tag::SymFloat:
        return true;
      case Tag::SymBool:
        return true;
      case Tag::Bool:
        return false;
      case Tag::Tuple:
        return true;
      case Tag::String:
        return true;
      case Tag::Blob:
        return true;
      case Tag::GenericList:
        return true;
      case Tag::GenericDict:
        return true;
      case Tag::Future:
        return true;
      case Tag::Await:
        return true;
      case Tag::Device:
        return false;
      case Tag::Stream:
        return true;
```
- EN: This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 1320-1343
```cpp
      case Tag::Object:
        return true;
      case Tag::PyObject:
        return true;
      case Tag::Uninitialized:
        return false;
      case Tag::Capsule:
        return true;
      case Tag::RRef:
        return true;
      case Tag::Quantizer:
        return true;
      case Tag::Enum:
        return true;
    }
    return false;
  }
  // NOLINTEND(bugprone-branch-clone)

 public:
  // Don't edit this just to add results for new tags; edit
  // isIntrusivePtrConstexpr above.
  bool isIntrusivePtr() const {
    // Implementation NOTE: the switch in isIntrusivePtrConstexpr
```
- EN: Focus symbols: `isIntrusivePtr`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isIntrusivePtr`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1344-1367
```cpp
    // above is the previous production implementation of this
    // function. We observed that, at least on x86_64, the generated
    // instruction sequence was a similar bit vector test to what we
    // have manually implemented below, except that there was an extra
    // "bounds check" branch confirming, essentially, that `tag <
    // kNumTags` and providing a consistent result in that case. We
    // don't care about the result if tag is out of bounds, so we'd
    // like to eliminate that comparison and branch; manually
    // implementing this function as a bit test is the simplest way I
    // could find to accomplish that elimination.
    static constexpr uint32_t kTruthTableBitVector =
#define TRUTH_TABLE_ENTRY(tag) \
  (uint32_t(isIntrusivePtrConstexpr(Tag::tag)) << uint32_t(Tag::tag)) |
        TORCH_FORALL_TAGS(TRUTH_TABLE_ENTRY)
#undef TRUTH_TABLE_ENTRY
            0;

    TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
        static_cast<uint32_t>(tag) < kNumTags,
        "unexpected tag ",
        static_cast<int>(tag));
    return kTruthTableBitVector & (1 << (uint32_t(tag) % 32));
  }

```
- EN: Focus symbols: `TRUTH_TABLE_ENTRY`, `uint32_t`, `isIntrusivePtrConstexpr`, `TORCH_FORALL_TAGS`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TRUTH_TABLE_ENTRY`, `uint32_t`, `isIntrusivePtrConstexpr`, `TORCH_FORALL_TAGS`, `TORCH_INTERNAL_ASSERT_DEBUG_ONLY`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1368-1391
```cpp
  // Storage and Generator were treated specially when
  // is_intrusive_ptr was stored as explicit state. This getter
  // preserves the old behavior for use with WeakIValue for now.
  bool isIntrusivePtrLegacyBehavior() const {
    if (tag == Tag::Storage || tag == Tag::Generator) {
      return payload.u.as_intrusive_ptr !=
          c10::UndefinedTensorImpl::singleton();
    } else {
      return isIntrusivePtr();
    }
  }

  union Payload {
    // [TriviallyCopyablePayload]
    // We use a nested union here so that we can make the copy easy
    // and efficient in the non-tensor (i.e., trivially copyable)
    // case. Specifically, we do not have to do a switch-on-tag to
    // figure out which union member to assign; we can just use
    // TriviallyCopyablePayload::operator=.
    union TriviallyCopyablePayload {
      TriviallyCopyablePayload() : as_int(0) {}
      int64_t as_int;
      // See Note [Meaning of HAS_u]
      uint64_t as_uint;
```
- EN: Focus symbols: `isIntrusivePtrLegacyBehavior`, `singleton`, `isIntrusivePtr`, `TriviallyCopyablePayload`, `as_int`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isIntrusivePtrLegacyBehavior`, `singleton`, `isIntrusivePtr`, `TriviallyCopyablePayload`, `as_int`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1392-1415
```cpp
      double as_double;
      bool as_bool;
      // Invariant: never nullptr; null state is represented as
      // c10::UndefinedTensorImpl::singleton() for consistency of
      // representation with Tensor.
      c10::intrusive_ptr_target* as_intrusive_ptr;
      struct {
        c10::DeviceType type;
        DeviceIndex index;
      } as_device;
    } u;
    static_assert(std::is_trivially_copyable_v<TriviallyCopyablePayload>);
    at::Tensor as_tensor;
    Payload() : u() {}
    Payload(const Payload&) = delete;
    Payload(Payload&&) = delete;
    Payload& operator=(const Payload&) = delete;
    Payload& operator=(Payload&&) = delete;
    // NOLINTNEXTLINE(modernize-use-equals-default)
    ~Payload() {}
  };

  IValue(const Payload& p, Tag t) : tag(t) {
    if (isTensor()) {
```
- EN: Focus symbols: `static_assert`, `Payload`, `u`, `~Payload`, `IValue`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`static_assert`, `Payload`, `u`, `~Payload`, `IValue`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1416-1439
```cpp
      new (&payload.as_tensor) at::Tensor(p.as_tensor);
    } else {
      payload.u = p.u;
    }
  }

  template <typename T>
  struct TagType {};

  friend MaybeOwnedTraits<IValue>;

  Payload payload;
  Tag tag{IValue::Tag::None};
  friend struct WeakIValue;
};

struct TORCH_API WeakIValue final {
  WeakIValue() = default;

  WeakIValue(const WeakIValue& rhs)
      : payload(rhs.payload),
        tag(rhs.tag),
        is_intrusive_ptr(rhs.is_intrusive_ptr) {
    if (is_intrusive_ptr &&
```
- EN: Focus symbols: `TagType`, `WeakIValue`, `Tensor`, `payload`, `tag`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`TagType`, `WeakIValue`, `Tensor`, `payload`, `tag`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1440-1463
```cpp
        payload.as_intrusive_ptr != c10::UndefinedTensorImpl::singleton()) {
      c10::raw::weak_intrusive_ptr::incref(payload.as_intrusive_ptr);
    }
  }
  WeakIValue(const IValue& rhs)
      : tag(rhs.tag), is_intrusive_ptr(rhs.isIntrusivePtrLegacyBehavior()) {
    if (rhs.isTensor()) {
      payload.as_intrusive_ptr = rhs.unsafeToTensorImpl();
      is_intrusive_ptr = true;
    } else {
      payload = rhs.payload.u;
    }
    if (is_intrusive_ptr) {
      if (payload.as_intrusive_ptr != c10::UndefinedTensorImpl::singleton()) {
        c10::raw::weak_intrusive_ptr::incref(payload.as_intrusive_ptr);
      }
    }
  }
  WeakIValue(WeakIValue&& rhs) noexcept : WeakIValue() {
    swap(rhs);
  }
  ~WeakIValue() {
    if (is_intrusive_ptr &&
        payload.as_intrusive_ptr != c10::UndefinedTensorImpl::singleton()) {
```
- EN: Focus symbols: `singleton`, `incref`, `WeakIValue`, `tag`, `is_intrusive_ptr`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`singleton`, `incref`, `WeakIValue`, `tag`, `is_intrusive_ptr`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1464-1487
```cpp
      c10::raw::weak_intrusive_ptr::decref(payload.as_intrusive_ptr);
    }
  }
  WeakIValue& operator=(WeakIValue&& rhs) & noexcept {
    WeakIValue(std::move(rhs)).swap(*this); // this also sets rhs to None
    return *this;
  }
  WeakIValue& operator=(WeakIValue const& rhs) & {
    WeakIValue(rhs).swap(*this);
    return *this;
  }
  void swap(WeakIValue& rhs) noexcept {
    std::swap(payload, rhs.payload);
    std::swap(is_intrusive_ptr, rhs.is_intrusive_ptr);
    std::swap(tag, rhs.tag);
  }

  bool isSameIdentity(const WeakIValue& rhs) const {
    return payload.as_int == rhs.payload.as_int && tag == rhs.tag &&
        is_intrusive_ptr == rhs.is_intrusive_ptr;
  }

  IValue lock() const {
    if (!is_intrusive_ptr) {
```
- EN: Focus symbols: `decref`, `WeakIValue`, `move`, `swap`, `isSameIdentity`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`decref`, `WeakIValue`, `move`, `swap`, `isSameIdentity`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1488-1511
```cpp
      IValue::Payload newPayload;
      newPayload.u = payload;
      return IValue(newPayload, tag);
    }
    if (IValue::Tag::Tensor == tag) {
      auto temp =
          c10::weak_intrusive_ptr<at::TensorImpl, c10::UndefinedTensorImpl>::
              reclaim(static_cast<at::TensorImpl*>(payload.as_intrusive_ptr));
      c10::intrusive_ptr<at::TensorImpl, c10::UndefinedTensorImpl> ip(
          temp.lock());
      temp.release();
      if (!ip) {
        return IValue();
      } else {
        return IValue(at::Tensor(std::move(ip)));
      }
    } else {
      auto temp = c10::weak_intrusive_ptr<c10::intrusive_ptr_target>::reclaim(
          payload.as_intrusive_ptr == c10::UndefinedTensorImpl::singleton()
              ? nullptr
              : payload.as_intrusive_ptr);
      IValue::Payload pl;
      pl.u.as_intrusive_ptr = temp.lock().release();
      temp.release();
```
- EN: Focus symbols: `IValue`, `reclaim`, `ip`, `lock`, `release`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `reclaim`, `ip`, `lock`, `release`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1512-1535
```cpp
      if (!pl.u.as_intrusive_ptr) {
        return IValue();
      } else {
        return IValue(pl, tag);
      }
    }
  }

  size_t use_count() const noexcept {
    if (!is_intrusive_ptr) {
      return 1;
    }
    auto temp = c10::weak_intrusive_ptr<
        c10::intrusive_ptr_target,
        c10::UndefinedTensorImpl>::reclaim(payload.as_intrusive_ptr);
    size_t result = temp.use_count();
    temp.release();
    return result;
  }

  size_t weak_use_count() const noexcept {
    if (!is_intrusive_ptr) {
      return 1;
    }
```
- EN: Focus symbols: `IValue`, `use_count`, `reclaim`, `release`, `weak_use_count`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`IValue`, `use_count`, `reclaim`, `release`, `weak_use_count`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 1536-1563
```cpp
    auto temp = c10::weak_intrusive_ptr<
        c10::intrusive_ptr_target,
        c10::UndefinedTensorImpl>::reclaim(payload.as_intrusive_ptr);
    size_t result = temp.weak_use_count();
    temp.release();
    return result;
  }
  size_t hash() const {
    return payload.as_int;
  }

 private:
  using Payload = IValue::Payload::TriviallyCopyablePayload;
  Payload payload;
  IValue::Tag tag{IValue::Tag::None};
  bool is_intrusive_ptr{false};
};

// An owning pointer to a type. When the type is class type, it requires a pair
// of shared_ptrs to the class type and its owning CU, so that the class type is
// guaranteed to stay alive as long as we hold this object.
struct TORCH_API StrongTypePtr {
  StrongTypePtr(std::shared_ptr<torch::jit::CompilationUnit> cu, TypePtr type);

  std::shared_ptr<torch::jit::CompilationUnit> cu_;
  TypePtr type_;
};

```
- EN: Focus symbols: `type`, `StrongTypePtr`, `Payload`, `reclaim`, `weak_use_count`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`type`, `StrongTypePtr`, `Payload`, `reclaim`, `weak_use_count`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1564-1590
```cpp
// [Constant Object Weak CompilationUnit Reference]
// A non owning pointer to a type. When a class get inserted as a constant
// into a graph, if we used a strong pointer we would have a circular reference
// from Object -> CompilationUnit and CompilationUnit -> Graph (which owns the
// Constant Object)
struct TORCH_API WeakTypePtr {
  WeakTypePtr(std::weak_ptr<torch::jit::CompilationUnit> cu, TypePtr type);

  std::weak_ptr<torch::jit::CompilationUnit> cu_;
  TypePtr type_;
};

// internal build errors with std::variant :/
struct WeakOrStrongCompilationUnit {
  explicit WeakOrStrongCompilationUnit(
      std::shared_ptr<torch::jit::CompilationUnit> shared_cu)
      : strong_ptr_(std::move(shared_cu)), weak_ptr_(std::nullopt) {}

  explicit WeakOrStrongCompilationUnit(
      std::weak_ptr<torch::jit::CompilationUnit> weak_cu)
      : strong_ptr_(std::nullopt), weak_ptr_(std::move(weak_cu)) {}

  std::shared_ptr<torch::jit::CompilationUnit> getStrongRefOrThrow() const {
    TORCH_INTERNAL_ASSERT(strong_ptr_.has_value());
    return *strong_ptr_;
  }

```
- EN: Focus symbols: `get`, `WeakTypePtr`, `WeakOrStrongCompilationUnit`, `strong_ptr_`, `move`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`get`, `WeakTypePtr`, `WeakOrStrongCompilationUnit`, `strong_ptr_`, `move`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1591-1614
```cpp
  std::weak_ptr<torch::jit::CompilationUnit> getWeakRefOrThrow() const {
    TORCH_INTERNAL_ASSERT(weak_ptr_.has_value());
    return *weak_ptr_;
  }

  bool holdingStrongRef() const {
    return strong_ptr_.has_value();
  }

  bool holdingEmptyStrongRef() const {
    return strong_ptr_ == nullptr;
  }

  std::optional<std::shared_ptr<torch::jit::CompilationUnit>> strong_ptr_;
  std::optional<std::weak_ptr<torch::jit::CompilationUnit>> weak_ptr_;
};

// An Object will hold a non-owning Compilation Unit reference if it is a
// Constant in the graph and a Owning reference otherwise
struct TORCH_API WeakOrStrongTypePtr {
  explicit WeakOrStrongTypePtr(WeakTypePtr weak)
      : cu_(WeakOrStrongCompilationUnit(std::move(weak.cu_))),
        type_(std::move(weak.type_)) {}
  explicit WeakOrStrongTypePtr(StrongTypePtr strong)
```
- EN: Focus symbols: `WeakOrStrongTypePtr`, `getWeakRefOrThrow`, `TORCH_INTERNAL_ASSERT`, `has_value`, `holdingStrongRef`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`WeakOrStrongTypePtr`, `getWeakRefOrThrow`, `TORCH_INTERNAL_ASSERT`, `has_value`, `holdingStrongRef`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 1615-1637
```cpp
      : cu_(WeakOrStrongCompilationUnit(std::move(strong.cu_))),
        type_(std::move(strong.type_)) {}
  explicit WeakOrStrongTypePtr(WeakOrStrongCompilationUnit cu, TypePtr type)
      : cu_(std::move(cu)), type_(std::move(type)) {}
  WeakTypePtr asWeakTypePtr() const;

  WeakOrStrongCompilationUnit cu_;
  TypePtr type_;

  bool holds_strong_ref() const {
    return cu_.holdingStrongRef();
  }

  bool holds_empty_strong_ref() const {
    return cu_.holdingEmptyStrongRef();
  }
};

} // namespace c10

C10_DIAGNOSTIC_POP()

#include <ATen/core/ivalue_inl.h> // IWYU pragma: keep
```
- EN: Focus symbols: `c10`, `cu_`, `WeakOrStrongCompilationUnit`, `move`, `type_`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`c10`, `cu_`, `WeakOrStrongCompilationUnit`, `move`, `type_`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- C++ templates and specialization / C++ 模板与特化
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- Backend/device dispatch / 后端/设备分发
- Dtype handling / 数据类型处理
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/DimVector.h`, `ATen/core/TensorBody.h`, `ATen/core/blob.h`, `ATen/core/custom_class.h`, `ATen/core/ivalue_to.h`, `ATen/core/jit_type_base.h`, `ATen/core/type_factory.h`, `c10/core/SymBool.h`, `c10/core/SymFloat.h`, `c10/macros/Export.h`, `c10/util/MaybeOwned.h`, `c10/util/intrusive_ptr.h`
- External/system includes / 外部或系统头: `limits`, `type_traits`, `unordered_map`, `unordered_set`, `utility`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/core/ivalue.cpp`
- Inferred semantic dependencies / 推断出的语义依赖: device dispatch / 设备分发; dtype dispatch / 数据类型分发; tensor abstractions / 张量抽象; C++ templates / C++ 模板; runtime validation / 运行时校验; namespace scoping / 命名空间作用域
