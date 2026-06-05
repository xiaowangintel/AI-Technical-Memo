# backend_init.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/backends/backend_init.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the extension points, compilation flow, and runtime helpers used by custom JIT backends. This specific file centers on `backend_init.cpp`. Python-facing bindings and object conversions are central to the implementation.
- **Purpose (CN)**: 实现自定义 JIT 后端所需的扩展点、编译流程和运行时辅助逻辑。 该文件具体围绕 `backend_init.cpp` 展开。 Python 侧绑定以及对象转换是实现重点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <torch/csrc/jit/backends/backend_init.h>

#include <pybind11/iostream.h>
#include <torch/csrc/jit/backends/backend_detail.h>
#include <torch/csrc/jit/python/module_python.h>
#include <torch/csrc/jit/python/pybind_utils.h>

namespace torch::jit {

// Get all types that are shared in the module hierarchy rooted at \p mod.
static std::unordered_set<TypePtr> getSharedModuleTypes(Module& mod) {
  // Maintain a set of all TypePtrs.
  std::unordered_set<TypePtr> types;
  // Maintain another set of TypePtrs that have been encountered more than once.
  std::unordered_set<TypePtr> duplicate_types;

  // Iterate over all modules in the hierarchy, including the root.
  for (auto module : mod.modules()) {
    auto module_type = module.type();
    if (types.count(module_type) > 0) {
```

- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include getSharedModuleTypes.
- **CN:** 这一段的重要可调用入口包括 getSharedModuleTypes。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 21-40 / 第 21-40 行

```cpp
      duplicate_types.insert(module_type);
    }

    types.insert(module_type);
  }

  return duplicate_types;
}

// Selectively lower \p mod to a backend. \p to_backend
// is called to lower modules. \p modules_to_lower contains
// qualified names of submodules of \p mod that should be lowered.
static void toBackendSelectiveImpl(
    Module& mod,
    const py::function& to_backend,
    const std::vector<std::string>& modules_to_lower,
    const std::unordered_set<TypePtr>& duplicate_types) {
  // This map will be used later to remap types in ancestor module graphs for
  // all lowered submodules.
  std::unordered_map<TypePtr, TypePtr> type_remap;
```

- **EN:** Important callable entry points in this range include toBackendSelectiveImpl.
- **CN:** 这一段的重要可调用入口包括 toBackendSelectiveImpl。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 41-60 / 第 41-60 行

```cpp

  // For each module that should be lowered:
  for (const auto& module_to_lower : modules_to_lower) {
    // Use QualifiedName to parse the qualified module names.
    c10::QualifiedName qual_module_name(module_to_lower);
    auto& atoms = qual_module_name.atoms();

    // Search through the module hierarchy using the atoms of
    // qual_module_name until current points to the module to
    // be lowered and parent points to its parent.
    Module current = mod;
    Module parent;

    for (size_t i = 0, e = atoms.size(); i < e; ++i) {
      IValue submodule = current.attr(atoms[i]);
      if (submodule.isModule()) {
        if (i == e - 1) {
          parent = current;
        }
        current = submodule.toModule();
```

- **EN:** Important callable entry points in this range include qual_module_name.
- **CN:** 这一段的重要可调用入口包括 qual_module_name。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 61-80 / 第 61-80 行

```cpp
      } else {
        std::stringstream err;
        err << "Attribute named " << atoms[i] << " is not a Module";
        throw std::runtime_error(err.str());
      }
    }

    // Check that the parent type is not shared and therefore can be edited.
    if (duplicate_types.count(parent.type()) > 0) {
      throw py::cast_error(c10::str(
          "Selective lowering is only supported for module hierarchies with unique types for selected modules; ",
          parent.type()->repr_str(),
          " is shared"));
    }

    // Call to_backend on the module that needs to be lowered. It needs to be
    // wrapped before doing so because _to_jit_backend accepts wrapped modules.
    // The result needs to be unwrapped in order to access its type below.
    auto lowered_submodule =
        py::cast<Module>(to_backend(py::module::import("torch.jit._recursive")
```

- **EN:** Important callable entry points in this range include runtime_error.
- **CN:** 这一段的重要可调用入口包括 runtime_error。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成, Branching logic / 分支逻辑。

### Lines 81-100 / 第 81-100 行

```cpp
                                        .attr("wrap_cpp_module")(current))
                             .attr("_c"));

    // Adjust the parent's type so that the type of the submodule matches
    // the type of lowered_submodule.
    auto parent_type = parent.type();

    parent_type->unsafeChangeAttributeType(
        atoms.back(), lowered_submodule.type());
    parent.setattr(atoms.back(), lowered_submodule._ivalue());

    // Record the type mapping from old type -> lowered type.
    type_remap[current.type()] = lowered_submodule.type();
  }

  // Having lowered all of the modules that needed to be lowered, remap types in
  // all graphs in the hierarchy so that the graphs all use the new lowered
  // type.
  auto type_remap_fn = [&type_remap](TypePtr in) {
    auto it = type_remap.find(in);
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Type system / 类型系统, Module API / 模块 API。

### Lines 101-120 / 第 101-120 行

```cpp
    if (it == type_remap.end())
      return in;
    return it->second;
  };

  // modules() iterates over all modules in the hierarchy including the root.
  for (auto module : mod.modules()) {
    auto module_type = module.type();
    for (auto& fn : module_type->methods()) {
      auto method = module.get_method(fn->name());
      auto graph = method.graph();
      graph->remapTypes(type_remap_fn);
      auto new_schema = fn->getSchema().cloneWithRemappedTypes(type_remap_fn);
      fn->setSchema(new_schema);
    }
  }
}

static Module codegen_func(
    const std::string& backend_name,
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 121-140 / 第 121-140 行

```cpp
    const Module& orig_module,
    const py::dict& method_compile_spec) {
  // Represents of a Type of Dict[str, Any].
  auto any_dict_ty = DictType::create(StringType::get(), AnyType::get());
  return detail::codegen_backend_module(
      backend_name,
      orig_module,
      toIValue(method_compile_spec, any_dict_ty).toGenericDict(),
      any_dict_ty);
}

void initJitBackendBindings(PyObject* module) {
  // Bind a function for lowering to each JIT backend. The name of the backend
  // must be the first argument. For example, to lower a Module to
  // "example_backend", declared as
  //
  //  static auto cls = torch::jit::backend<ExampleBackend>("example_backend");
  //
  // this function must be called like
  //
```

- **EN:** Important callable entry points in this range include codegen_backend_module, initJitBackendBindings.
- **CN:** 这一段的重要可调用入口包括 codegen_backend_module, initJitBackendBindings。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-160 / 第 141-160 行

```cpp
  //  torch._C._jit_to_backend("example_backend", module, spec)
  auto m = py::handle(module).cast<py::module>();
  m.def(
      "_jit_to_backend",
      [=](const std::string& backend_name,
          py::handle orig_module,
          const py::dict& method_compile_spec) {
        py::scoped_ostream_redirect cerr(
            std::cerr, py::module_::import("sys").attr("stderr"));
        py::scoped_ostream_redirect cout(
            std::cout, py::module_::import("sys").attr("stdout"));
        return py::module::import("torch.jit._recursive")
            .attr("wrap_cpp_module")(codegen_func(
                backend_name,
                py::cast<Module>(orig_module.attr("_c")),
                method_compile_spec));
      });

  m.def(
      "_jit_to_backend_selective",
```

- **EN:** Important callable entry points in this range include cerr, cout, import.
- **CN:** 这一段的重要可调用入口包括 cerr, cout, import。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成, Code generation / 代码生成, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成, Code generation / 代码生成, Result propagation / 结果传递。

### Lines 161-180 / 第 161-180 行

```cpp
      [=](py::handle orig_module,
          const py::function& to_backend,
          const std::vector<std::string>& modules_to_lower) {
        py::scoped_ostream_redirect cerr(
            std::cerr, py::module_::import("sys").attr("stderr"));
        py::scoped_ostream_redirect cout(
            std::cout, py::module_::import("sys").attr("stdout"));
        if (auto original_module =
                as_module(py::cast<py::object>(orig_module))) {
          // Clone the Module to avoid editing types that are shared with
          // Modules in other instances outside this hierarchy.
          Module& mod = original_module.value();
          auto cloned_mod = mod.clone();
          // Get all shared module types. Type sharing is only a problem if the
          // parent modules of the ones to lower are in this set.
          auto shared_types = getSharedModuleTypes(cloned_mod);
          toBackendSelectiveImpl(
              cloned_mod, to_backend, modules_to_lower, shared_types);
          // Wrap the result in a RecursiveScriptModule because that's what
          // the caller passed in.
```

- **EN:** Important callable entry points in this range include cerr, cout, toBackendSelectiveImpl.
- **CN:** 这一段的重要可调用入口包括 cerr, cout, toBackendSelectiveImpl。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 181-189 / 第 181-189 行

```cpp
          return py::module::import("torch.jit._recursive")
              .attr("wrap_cpp_module")(cloned_mod);
        }

        throw py::cast_error(c10::str(
            "Object ", py::str(orig_module), " is not a ScriptModule"));
      });
}
} // namespace torch::jit
```

- **EN:** Important callable entry points in this range include import, cast_error.
- **CN:** 这一段的重要可调用入口包括 import, cast_error。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Module API / 模块 API, Python binding / Python 绑定, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Module API / 模块 API, Python binding / Python 绑定, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Custom backend pipeline** — 自定义后端流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Python binding** — Python 绑定
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass

## Dependencies / 依赖关系

- `torch/csrc/jit/backends/backend_init.h`
- `torch/csrc/jit/backends/backend_detail.h`
- `torch/csrc/jit/python/module_python.h`
- `torch/csrc/jit/python/pybind_utils.h`
