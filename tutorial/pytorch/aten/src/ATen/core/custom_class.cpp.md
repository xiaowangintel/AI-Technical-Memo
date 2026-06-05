# custom_class.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/custom_class.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `id`, `type`, `with`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `id`, `type`, `with`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
#include <ATen/core/function_schema.h>
#include <ATen/core/functional.h>
#include <ATen/core/jit_type.h>
#include <ATen/core/type_factory.h>
#include <c10/util/flat_hash_map.h>
#include <torch/custom_class.h>
#include <torch/custom_class_detail.h>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 9-18
```cpp
#include <unordered_map>

namespace c10 {

static ska::flat_hash_map<std::type_index, c10::ClassTypePtr>&
getCustomClassTypeMap() {
  static ska::flat_hash_map<std::type_index, c10::ClassTypePtr> tmap;
  return tmap;
}

```
- EN: Focus symbols: `c10`, `getCustomClassTypeMap`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`c10`, `getCustomClassTypeMap`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 19-24
```cpp
c10::ClassTypePtr getCustomClassTypeImpl(const std::type_index& tindex) {
  auto& tmap = c10::getCustomClassTypeMap();
  auto res = tmap.find(tindex);
  if (C10_UNLIKELY(res == tmap.end())) {
    // type_index is not guaranteed to be unique across shared libraries on some
    // platforms For example see
```
- EN: Focus symbols: `getCustomClassTypeImpl`, `getCustomClassTypeMap`, `find`, `C10_UNLIKELY`, `end`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCustomClassTypeImpl`, `getCustomClassTypeMap`, `find`, `C10_UNLIKELY`, `end`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 25-30
```cpp
    // https://github.com/llvm-mirror/libcxx/blob/78d6a7767ed57b50122a161b91f59f19c9bd0d19/include/typeinfo#L133
    // Also, this is not the case if RTLD_LOCAL option is used, see
    // https://github.com/pybind/pybind11/blob/f791dc8648e1f6ec33f402d679b6b116a76d4e1b/include/pybind11/detail/internals.h#L101-L106
    // Take a slow path of iterating over all registered types and compare their
    // names
    auto class_name = std::string(tindex.name());
```
- EN: Focus symbols: `string`, `name`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`string`, `name`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 31-36
```cpp
    for (const auto& it : tmap) {
      if (class_name == it.first.name()) {
        // Do not modify existing type map here as this template is supposed to
        // be called only once per type from getCustomClassTypeImpl()
        return it.second;
      }
```
- EN: Focus symbols: `name`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`name`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 37-45
```cpp
    }
    TORCH_CHECK(
        false,
        "Can't find class id in custom class type map for ",
        tindex.name());
  }
  return res->second;
}

```
- EN: Focus symbols: `id`, `type`, `TORCH_CHECK`, `name`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`id`, `type`, `TORCH_CHECK`, `name`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 46-51
```cpp
} // namespace c10

namespace torch {

namespace detail {

```
- EN: Focus symbols: `c10`, `torch`, `detail`. This block opens or closes namespaces to keep symbols in the intended API scope. This keeps the implementation aligned with the broader ATen/C10 module hierarchy.
- CN: 关注符号：`c10`, `torch`, `detail`。该代码块打开或关闭命名空间以维持符号的预期作用域。这使实现与更大的 ATen/C10 模块层次保持一致。

### Lines 52-60
```cpp
#if defined ENABLE_RECORD_KERNEL_FUNCTION_DTYPE
void record_custom_class(std::string name) {
  RECORD_FUNCTION_WITH_SCOPE(
      at::RecordScope::CUSTOM_CLASS,
      std::move(name),
      c10::ArrayRef<const c10::IValue>{});
}
#endif

```
- EN: Focus symbols: `record_custom_class`, `RECORD_FUNCTION_WITH_SCOPE`, `move`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`record_custom_class`, `RECORD_FUNCTION_WITH_SCOPE`, `move`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 61-67
```cpp
} // namespace detail

static std::unordered_map<std::string, at::ClassTypePtr>& customClasses() {
  static std::unordered_map<std::string, at::ClassTypePtr> customClasses;
  return customClasses;
}

```
- EN: Focus symbols: `detail`, `customClasses`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`detail`, `customClasses`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 68-73
```cpp
void registerCustomClass(at::ClassTypePtr class_type) {
  TORCH_INTERNAL_ASSERT(class_type->name());
  // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
  auto name = class_type->name()->qualifiedName();
  TORCH_CHECK(
      !customClasses().count(name),
```
- EN: Focus symbols: `registerCustomClass`, `TORCH_INTERNAL_ASSERT`, `name`, `qualifiedName`, `TORCH_CHECK`. This block implements callable behavior together with runtime validation. The checks reject unsupported states before delegating work to lower-level helpers.
- CN: 关注符号：`registerCustomClass`, `TORCH_INTERNAL_ASSERT`, `name`, `qualifiedName`, `TORCH_CHECK`。该代码块实现可调用行为并包含运行时校验。这些检查会在把工作下发给更底层辅助函数前拒绝不支持的状态。

### Lines 74-79
```cpp
      "Custom class with name ",
      name,
      " is already registered. Ensure that registration with torch::class_ is only called once.");
  customClasses()[name] = std::move(class_type);
}

```
- EN: Focus symbols: `with`, `customClasses`, `move`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`with`, `customClasses`, `move`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 80-88
```cpp
at::ClassTypePtr getCustomClass(const std::string& class_name) {
  auto ret =
      customClasses().count(class_name) ? customClasses()[class_name] : nullptr;
  if (ret) {
    RECORD_CUSTOM_CLASS(class_name);
  }
  return ret;
}

```
- EN: Focus symbols: `getCustomClass`, `customClasses`, `count`, `RECORD_CUSTOM_CLASS`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCustomClass`, `customClasses`, `count`, `RECORD_CUSTOM_CLASS`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 89-96
```cpp
const std::unordered_set<std::string> getAllCustomClassesNames() {
  std::unordered_set<std::string> ret;
  for (const auto& kv : customClasses()) {
    ret.insert(kv.first);
  }
  return ret;
}

```
- EN: Focus symbols: `getAllCustomClassesNames`, `customClasses`, `insert`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getAllCustomClassesNames`, `customClasses`, `insert`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 97-102
```cpp
bool isCustomClass(const c10::IValue& v) {
  return v.isObject() && v.toObject()->type()->name() &&
      // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
      getCustomClass(v.toObject()->type()->name()->qualifiedName());
}

```
- EN: Focus symbols: `isCustomClass`, `isObject`, `toObject`, `type`, `name`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`isCustomClass`, `isObject`, `toObject`, `type`, `name`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 103-111
```cpp
static std::vector<std::unique_ptr<jit::Function>>& customClassMethods() {
  static std::vector<std::unique_ptr<jit::Function>> customClassMethods;
  return customClassMethods;
}

void registerCustomClassMethod(std::unique_ptr<jit::Function> fn) {
  customClassMethods().emplace_back(std::move(fn));
}

```
- EN: Focus symbols: `customClassMethods`, `registerCustomClassMethod`, `emplace_back`, `move`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`customClassMethods`, `registerCustomClassMethod`, `emplace_back`, `move`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 112-118
```cpp
std::vector<c10::FunctionSchema> customClassSchemasForBCCheck() {
  auto& methods = customClassMethods();
  return c10::fmap(methods, [](const std::unique_ptr<jit::Function>& fn) {
    return fn->getSchema();
  });
}

```
- EN: Focus symbols: `customClassSchemasForBCCheck`, `customClassMethods`, `fmap`, `getSchema`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`customClassSchemasForBCCheck`, `customClassMethods`, `fmap`, `getSchema`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 119-124
```cpp
namespace detail {
class_base::class_base(
    const std::string& namespaceName,
    const std::string& className,
    std::string doc_string,
    const std::type_info& intrusivePtrClassTypeid,
```
- EN: Focus symbols: `detail`, `class_base`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`detail`, `class_base`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 125-130
```cpp
    const std::type_info& taggedCapsuleClassTypeid)
    : qualClassName(
          "__torch__.torch.classes." + namespaceName + '.' + className),
      classTypePtr(at::ClassType::create(
          c10::QualifiedName(qualClassName),
          std::weak_ptr<jit::CompilationUnit>(),
```
- EN: Focus symbols: `qualClassName`, `classTypePtr`, `create`, `QualifiedName`. This block connects nearby declarations and implementation details. This glue code keeps the surrounding abstraction consistent and usable.
- CN: 关注符号：`qualClassName`, `classTypePtr`, `create`, `QualifiedName`。该代码块连接相邻的声明与实现细节。这些胶水代码让周边抽象保持一致并可被使用。

### Lines 131-136
```cpp
          /*is_module=*/false,
          std::move(doc_string))) {
  detail::checkValidIdent(namespaceName, "Namespace name");
  detail::checkValidIdent(className, "Class name");
  classTypePtr->addAttribute(
      "capsule", c10::TypeFactory::get<c10::CapsuleType>());
```
- EN: Focus symbols: `move`, `checkValidIdent`, `addAttribute`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`move`, `checkValidIdent`, `addAttribute`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 137-144
```cpp
  c10::getCustomClassTypeMap().insert(
      {std::type_index(intrusivePtrClassTypeid), classTypePtr});
  c10::getCustomClassTypeMap().insert(
      {std::type_index(taggedCapsuleClassTypeid), classTypePtr});

  registerCustomClass(classTypePtr);
}

```
- EN: Focus symbols: `getCustomClassTypeMap`, `insert`, `type_index`, `registerCustomClass`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`getCustomClassTypeMap`, `insert`, `type_index`, `registerCustomClass`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 145-151
```cpp
c10::FunctionSchema class_base::withNewArguments(
    const c10::FunctionSchema& schema,
    std::initializer_list<arg> default_args) {
  const auto& old_args = schema.arguments();
  std::vector<c10::Argument> new_args;
  new_args.reserve(old_args.size());

```
- EN: Focus symbols: `withNewArguments`, `arguments`, `reserve`, `size`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`withNewArguments`, `arguments`, `reserve`, `size`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 152-157
```cpp
  new_args.emplace_back(old_args[0]);
  // Skip self.
  size_t argIdx = 1;
  for (const auto& default_arg : default_args) {
    auto& old_arg = old_args[argIdx++];
    new_args.emplace_back(
```
- EN: Focus symbols: `emplace_back`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`emplace_back`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 158-166
```cpp
        default_arg.name_,
        old_arg.type(),
        old_arg.real_type(),
        old_arg.N(),
        default_arg.value_);
  }
  return schema.cloneWithArguments(std::move(new_args));
}

```
- EN: Focus symbols: `type`, `real_type`, `N`, `cloneWithArguments`, `move`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`type`, `real_type`, `N`, `cloneWithArguments`, `move`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 167-168
```cpp
} // namespace detail
} // namespace torch
```
- EN: Focus symbols: `detail`, `torch`. This block opens or closes namespaces to keep symbols in the intended API scope. This keeps the implementation aligned with the broader ATen/C10 module hierarchy.
- CN: 关注符号：`detail`, `torch`。该代码块打开或关闭命名空间以维持符号的预期作用域。这使实现与更大的 ATen/C10 模块层次保持一致。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/function_schema.h`, `ATen/core/functional.h`, `ATen/core/jit_type.h`, `ATen/core/type_factory.h`, `c10/util/flat_hash_map.h`, `torch/custom_class.h`, `torch/custom_class_detail.h`
- External/system includes / 外部或系统头: `unordered_map`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/core/custom_class.h`
- Inferred semantic dependencies / 推断出的语义依赖: runtime validation / 运行时校验; namespace scoping / 命名空间作用域
