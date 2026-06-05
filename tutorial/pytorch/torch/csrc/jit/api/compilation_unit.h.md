# compilation_unit.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/api/compilation_unit.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or implements the public C++ TorchScript/JIT API surface used by modules, methods, functions, and objects. This specific file centers on `compilation_unit.h`. Alias and mutation tracking are important here. Python-facing bindings and object conversions are central to the implementation.
- **Purpose (CN)**: 定义或实现模块、方法、函数与对象使用的公开 C++ TorchScript/JIT API。 该文件具体围绕 `compilation_unit.h` 展开。 别名关系与可变状态跟踪是这里的核心。 Python 侧绑定以及对象转换是实现重点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#pragma once
#include <ATen/core/function.h>
#include <c10/util/Exception.h>
#include <torch/csrc/jit/api/function_impl.h>
#include <torch/csrc/jit/frontend/name_mangler.h>
#include <torch/csrc/jit/frontend/source_range.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/runtime/graph_executor.h>

#include <torch/csrc/Export.h>

#include <ATen/core/function_schema.h>
#include <ATen/core/qualified_name.h>
#include <c10/util/ArrayRef.h>
#include <optional>

#include <functional>
#include <memory>
#include <mutex>
#include <ostream>
#include <string>
#include <unordered_map>
#include <vector>

namespace torch::jit {

struct Def;
struct Property;
```

- **EN:** This range collects the headers or imports needed by the following declarations and implementation blocks.
- **CN:** 这一段集中引入后续声明与实现所需的头文件或模块。
- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including Def, Property.
- **CN:** 该代码块声明或细化了 Def, Property 等核心类型。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
struct ClassDef;
struct SugaredValue;
struct Resolver;

using ResolverPtr = std::shared_ptr<Resolver>;
struct Self {
  virtual ~Self() = default;
  virtual std::shared_ptr<SugaredValue> makeSugared(Value* v) const = 0;
  virtual ClassTypePtr getClassType() const = 0;
};

// A CompilationUnit is a list of named Functions
// with helper methods to iterate the list or invoke the function.
// Classes have a CompilationUnit holding the class methods,
// and Modules have a CompilationUnit holding the Functions that
// are used to implement their Methods

struct TORCH_API CompilationUnit {
  enum class FunctionType { Method, Hook, PreHook };
  // constructor that takes a set of functions to compile using the native
  // resolver
  explicit CompilationUnit(const std::string& source);
  CompilationUnit() = default;

  CompilationUnit& operator=(CompilationUnit&&) = default;
  CompilationUnit(CompilationUnit&&) = default;
  CompilationUnit& operator=(const CompilationUnit&) = delete;
  CompilationUnit(const CompilationUnit&) = delete;
```

- **EN:** The block declares or refines core types including ClassDef, SugaredValue, Resolver, Self, methods.
- **CN:** 该代码块声明或细化了 ClassDef, SugaredValue, Resolver, Self, methods 等核心类型。
- **EN:** Alias declarations such as ResolverPtr simplify later API usage.
- **CN:** ResolverPtr 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include CompilationUnit.
- **CN:** 这一段的重要可调用入口包括 CompilationUnit。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 57-84 / 第 57-84 行

```cpp

  Function* find_function(const c10::QualifiedName& name) const {
    auto it = dict_.find(name);
    if (it == dict_.end()) {
      return nullptr;
    }
    return functions_[it->second].get();
  }

  Function& get_function(const c10::QualifiedName& name) const {
    if (auto r = find_function(name)) {
      return *r;
    }
    TORCH_CHECK(false, "attempted to get undefined function ", name.name());
  }

  void set_optimized(bool o) {
    TORCH_WARN(
        "CompilationUnit::set_optimized() is deprecated and has no effect. "
        "Please use setGraphExecutorOptimize()");
  }

  bool is_optimized() const {
    TORCH_WARN(
        "CompilationUnit::is_optimized() is deprecated and always returns true. "
        "Please use getGraphExecutorOptimize()");
    return true;
  }
```

- **EN:** Important callable entry points in this range include find_function, get_function, TORCH_CHECK, set_optimized, TORCH_WARN, is_optimized.
- **CN:** 这一段的重要可调用入口包括 find_function, get_function, TORCH_CHECK, set_optimized, TORCH_WARN, is_optimized。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 85-112 / 第 85-112 行

```cpp

  // for historic reasons, these are defined in ir_emitter.cpp
  // Returns the list of Functions just defined.
  std::vector<Function*> define(
      const std::optional<c10::QualifiedName>& prefix,
      const std::vector<Property>& properties,
      const std::vector<ResolverPtr>& propResolvers,
      const std::vector<Def>& definitions,
      const std::vector<ResolverPtr>&
          defResolvers, /* determines how we handle free
                     variables in each definition*/
      // if non-null, the first argument to each def, is bound to this value
      const Self* self,
      // see [name mangling]
      bool shouldMangle = false,
      std::optional<size_t> operator_set_version = std::nullopt);

  void define_hooks(
      const std::optional<c10::QualifiedName>& prefix,
      const std::vector<Def>& hookDefs,
      const std::vector<ResolverPtr>& hookResolvers,
      const std::vector<Def>& preHookDefs,
      const std::vector<ResolverPtr>& preHookResolvers,
      const Self* self,
      bool shouldMangle = false);

  // same as above but parse the definitions from source
  // Returns the list of Functions just defined.
```

- **EN:** Important callable entry points in this range include define, define_hooks.
- **CN:** 这一段的重要可调用入口包括 define, define_hooks。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Operator schema / 算子模式, Registration / 注册机制.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Operator schema / 算子模式, Registration / 注册机制。

### Lines 113-140 / 第 113-140 行

```cpp
  std::vector<Function*> define(
      // prefix namespace to put all the defined functions into
      const std::optional<c10::QualifiedName>& prefix,
      const std::string& source,
      const ResolverPtr& resolver,
      const Self* self);

  void define_interface(
      const c10::QualifiedName& qualifiedName,
      const ClassDef& classDef,
      ResolverPtr rcb,
      bool is_module = false);

  Function* create_function(
      c10::QualifiedName name,
      std::shared_ptr<Graph> graph,
      bool shouldMangle = false) {
    if (shouldMangle) {
      name = mangle(name);
    }
    auto fn = std::make_unique<GraphFunction>(
        std::move(name), std::move(graph), nullptr);
    auto ret = fn.get();
    register_function(std::move(fn));
    return ret;
  }

  std::vector<Function*> get_functions() const {
```

- **EN:** Important callable entry points in this range include define, define_interface, create_function, move, register_function, get_functions.
- **CN:** 这一段的重要可调用入口包括 define, define_interface, create_function, move, register_function, get_functions。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 141-168 / 第 141-168 行

```cpp
    return fmap(functions_, [](const std::unique_ptr<Function>& fn) {
      return fn.get();
    });
  }

  /// Run a method from this compilation.
  ///
  /// For example:
  /// @code
  ///   IValue output = module->run("relu_script", a, b);
  /// @endcode
  ///
  /// To get a compile a module from a source string, see torch::jit::compile
  ///
  /// @param method_name The name of the method to run
  /// @param args Arguments to be passed to the method
  /// @return An IValue containing the return value (or values if it is a tuple)
  /// from the method
  template <typename... Types>
  IValue run_method(const c10::QualifiedName& method_name, Types&&... args) {
    return get_function(method_name)({IValue(std::forward<Types>(args))...});
  }

  void drop_all_functions() {
    dict_.clear();
    functions_.clear();
  }

```

- **EN:** Important callable entry points in this range include fmap, run_method, drop_all_functions.
- **CN:** 这一段的重要可调用入口包括 fmap, run_method, drop_all_functions。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Backend integration / 后端集成, Optimization pass / 优化 pass.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Backend integration / 后端集成, Optimization pass / 优化 pass。

### Lines 169-196 / 第 169-196 行

```cpp
  /**
   * Register a class as being owned by this compilation unit.
   */
  void register_type(c10::NamedTypePtr namedType) {
    // TODO: class types cannot be redefined because we have no way right now
    // of invalidating their methods. NamedTuples are fine though, since they
    // don't have methods.
    TORCH_CHECK(
        0 == classDict_.count(*namedType->name()),
        "class '",
        namedType->name()->qualifiedName(),
        "' already defined.");
    classes_.push_back(std::move(namedType));
    classDict_[*classes_.back()->name()] = classes_.size() - 1;
  }

  c10::ClassTypePtr get_class(const c10::QualifiedName& name) const {
    auto type = get_type(name);
    if (!type) {
      return nullptr;
    }
    return type->cast<c10::ClassType>();
  }

  c10::InterfaceTypePtr get_interface(const c10::QualifiedName& name) const {
    auto type = get_type(name);
    if (!type) {
      return nullptr;
```

- **EN:** The block declares or refines core types including as, types.
- **CN:** 该代码块声明或细化了 as, types 等核心类型。
- **EN:** Important callable entry points in this range include register_type, TORCH_CHECK, get_class, get_interface.
- **CN:** 这一段的重要可调用入口包括 register_type, TORCH_CHECK, get_class, get_interface。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 197-224 / 第 197-224 行

```cpp
    }
    return type->cast<c10::InterfaceType>();
  }

  c10::TupleTypePtr get_named_tuple(const c10::QualifiedName& name) const {
    for (const auto& cls : classes_) {
      if (cls->name()->qualifiedName() == name.qualifiedName()) {
        return cls->expect<TupleType>();
      }
    }
    return nullptr;
  }

  c10::NamedTypePtr get_type(const c10::QualifiedName& name) const {
    auto it = classDict_.find(name);
    if (it == classDict_.end()) {
      return nullptr;
    }
    return classes_[it->second];
  }

  // For testing: clear all Python-defined classes to ensure that unit tests
  // have isolation.
  void _clear_python_cu() {
    // Delete all the associated class methods
    for (const auto& type : classes_) {
      if (auto cls = type->cast<ClassType>()) {
        for (auto method : cls->methods()) {
```

- **EN:** The block declares or refines core types including methods.
- **CN:** 该代码块声明或细化了 methods 等核心类型。
- **EN:** Important callable entry points in this range include get_named_tuple, get_type, _clear_python_cu.
- **CN:** 这一段的重要可调用入口包括 get_named_tuple, get_type, _clear_python_cu。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
          // Tombstone the method in the compilation unit.
          // Don't erase because the dict_
          auto it = dict_.find(method->qualname());
          if (it != dict_.end()) {
            functions_[it->second] = nullptr;
            // Erase in our big lookup table
            dict_.erase(it);
          }
        }
        // Classes can have multiple pointers to the same hook,
        // need to make sure to not delete it twice
        std::unordered_set<Function*> hooks_to_delete;
        for (const auto& hook : cls->getForwardHooks()) {
          hooks_to_delete.insert(hook);
        }
        for (const auto& pre_hook : cls->getForwardPreHooks()) {
          hooks_to_delete.insert(pre_hook);
        }
        for (const auto& hook : hooks_to_delete) {
          // Tombstone the hook in the compilation unit.
          auto it = dict_.find(hook->qualname());
          if (it != dict_.end()) {
            functions_[it->second] = nullptr;
            // Erase in our big lookup table
            dict_.erase(it);
          }
        }
      }
```

- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Module API / 模块 API, Branching logic / 分支逻辑, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Module API / 模块 API, Branching logic / 分支逻辑, Iteration / 迭代处理。

### Lines 253-280 / 第 253-280 行

```cpp
    }
    classes_.clear();
    classDict_.clear();
  }

  // [Internal Only] Remove method.
  // Note Used for freezing.
  void unsafeRemoveMethod(const c10::QualifiedName& method_name) {
    auto it = dict_.find(method_name);
    TORCH_CHECK(
        it != dict_.end(),
        "method '",
        method_name.qualifiedName(),
        "' does not exist.");
    functions_[it->second] = nullptr;
    dict_.erase(it);
  }

  // [name mangling] All code objects must have a unique qualified name in a
  // CompilationUnit. In Python, sometimes functions won't have unique qualified
  // name (for example, nested functions). So we mangle Python functions to
  // ensure that they are uniquely named.
  //
  // We also use mangling to distinguish different Module instances. Since each
  // Module is a singleton class instance, different instances of the same
  // Python Module will have different types but the same qualified name.
  c10::QualifiedName mangle(const c10::QualifiedName& name) const {
    auto mangled = name;
```

- **EN:** The block declares or refines core types including instance.
- **CN:** 该代码块声明或细化了 instance 等核心类型。
- **EN:** Important callable entry points in this range include unsafeRemoveMethod, TORCH_CHECK, mangle.
- **CN:** 这一段的重要可调用入口包括 unsafeRemoveMethod, TORCH_CHECK, mangle。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Declared symbols / 声明的符号, Type definition / 类型定义.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Declared symbols / 声明的符号, Type definition / 类型定义。

### Lines 281-308 / 第 281-308 行

```cpp
    while (get_type(mangled) || find_function(mangled)) {
      mangled = mangler_.mangle(mangled);
    }
    return mangled;
  }

 private:
  std::unique_ptr<Function> define(
      const std::optional<c10::QualifiedName>& prefix,
      const Def& def,
      const ResolverPtr& resolver,
      const Self* self,
      const std::unordered_map<std::string, Function*>& function_table,
      bool shouldMangle = false,
      FunctionType type = FunctionType::Method,
      std::optional<size_t> version = std::nullopt) const;

  // Define a property on \p self.
  struct PropertyPair;
  PropertyPair define_property(
      const std::optional<c10::QualifiedName>& prefix,
      const Property& prop,
      const ResolverPtr& resolver,
      const Self* self,
      const std::unordered_map<std::string, Function*>& function_table,
      bool shouldMangle = false) const;

  Function& register_function(std::unique_ptr<Function> fn) {
```

- **EN:** The block declares or refines core types including PropertyPair.
- **CN:** 该代码块声明或细化了 PropertyPair 等核心类型。
- **EN:** Important callable entry points in this range include define, define_property, register_function.
- **CN:** 这一段的重要可调用入口包括 define, define_property, register_function。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 309-336 / 第 309-336 行

```cpp
    TORCH_CHECK(
        0 == dict_.count(fn->qualname().qualifiedName()),
        "method '",
        fn->qualname().qualifiedName(),
        "' already defined.");
    functions_.emplace_back(std::move(fn));
    dict_[functions_.back()->qualname()] = functions_.size() - 1;
    return *functions_.back();
  }
  std::vector<std::unique_ptr<Function>> functions_;
  // for fast lookup
  std::unordered_map<c10::QualifiedName, size_t> dict_;
  std::unordered_map<c10::QualifiedName, size_t> classDict_;

  // [class ownership] Right now there are two relationships between classes
  // and compilation units:
  // 1. Classes have compilation units internally that hold their methods.
  // 2. On load, the TypePtrs of any imported classes are owned by the main
  // module's compilation unit.
  std::vector<c10::NamedTypePtr> classes_;

  mutable NameMangler mangler_;
};

// An owning pointer to a Function. Just a pair of a raw Function ptr and it's
// owning CU. We need this because pybind requires a ref-counted way to refer to
// Functions.
struct StrongFunctionPtr {
```

- **EN:** The block declares or refines core types including ownership, StrongFunctionPtr.
- **CN:** 该代码块声明或细化了 ownership, StrongFunctionPtr 等核心类型。
- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 337-351 / 第 337-351 行

```cpp
  StrongFunctionPtr(std::shared_ptr<CompilationUnit> cu, Function* function)
      : cu_(std::move(cu)), function_(function) {
    TORCH_INTERNAL_ASSERT(cu_);
    TORCH_INTERNAL_ASSERT(function_);
  }
  std::shared_ptr<CompilationUnit> cu_;
  Function* function_;
};

namespace script {
// We once had a `script::` namespace that was deleted. This is for backcompat
// of the public API; new code should not use this type alias.
using CompilationUnit = ::torch::jit::CompilationUnit;
} // namespace script
} // namespace torch::jit
```

- **EN:** It enters or references namespace scopes such as script, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 script 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Alias declarations such as CompilationUnit simplify later API usage.
- **CN:** CompilationUnit 等别名声明简化了后续 API 的使用。
- **EN:** Important callable entry points in this range include StrongFunctionPtr, TORCH_INTERNAL_ASSERT.
- **CN:** 这一段的重要可调用入口包括 StrongFunctionPtr, TORCH_INTERNAL_ASSERT。
- **EN:** State, aliasing, or mutation tracking is emphasized in this block.
- **CN:** 该代码块强调了状态、别名关系或可变性跟踪。

## Key Concepts / 关键概念

- **Public JIT API surface** — 公开 JIT API 接口层
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Python binding** — Python 绑定
- **Backend integration** — 后端集成
- **Optimization pass** — 优化 pass
- **Registration** — 注册机制

## Dependencies / 依赖关系

- `ATen/core/function.h`
- `c10/util/Exception.h`
- `torch/csrc/jit/api/function_impl.h`
- `torch/csrc/jit/frontend/name_mangler.h`
- `torch/csrc/jit/frontend/source_range.h`
- `torch/csrc/jit/ir/ir.h`
- `torch/csrc/jit/runtime/graph_executor.h`
- `torch/csrc/Export.h`
- `ATen/core/function_schema.h`
- `ATen/core/qualified_name.h`
- `c10/util/ArrayRef.h`
