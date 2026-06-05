# nnapi_backend_preprocess.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/backends/nnapi/nnapi_backend_preprocess.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the extension points, compilation flow, and runtime helpers used by custom JIT backends. This specific file centers on `nnapi_backend_preprocess.cpp`. Python-facing bindings and object conversions are central to the implementation.
- **Purpose (CN)**: 实现自定义 JIT 后端所需的扩展点、编译流程和运行时辅助逻辑。 该文件具体围绕 `nnapi_backend_preprocess.cpp` 展开。 Python 侧绑定以及对象转换是实现重点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <pybind11/pybind11.h>
#include <torch/csrc/jit/backends/backend_preprocess.h>
#include <torch/csrc/utils/pybind.h>

namespace py = pybind11;

// Converts model to Android NNAPI backend and serializes it for mobile
// Returns a dictionary with preprocessed items:
//    "shape_compute_module": torch::jit::Module,
//    "ser_model": at::Tensor,
//    "weights": List[torch.Tensor],
//    "inp_mem_fmts": List[int],
//    "out_mem_fmts": List[int]
//
// method_compile_spec should contain a Tensor or
// Tensor List which bundles several input parameters:
// shape, dtype, quantization, and dimorder (NHWC/NCHW)
// For input shapes, use 0 for run/load time flexible input
//
// The compile_spec should include the format:
```

- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Serialization / 序列化, Mobile runtime / 移动端运行时.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Serialization / 序列化, Mobile runtime / 移动端运行时。

### Lines 21-40 / 第 21-40 行

```cpp
// {"forward": {"inputs": at::Tensor}}
// OR {"forward": {"inputs": c10::List<at::Tensor>}}
// Example input Tensor:
// torch.tensor([[1.0, -1.0, 2.0, -2.0]]).unsqueeze(-1).unsqueeze(-1)
//
// In the future, preprocess will accept a dedicated object
static c10::IValue preprocess(
    const torch::jit::Module& mod,
    const c10::Dict<c10::IValue, c10::IValue>& method_compile_spec,
    const torch::jit::BackendDebugHandleGenerator& generate_debug_handles) {
  // Import the python function for processing modules to Android NNAPI backend
  py::gil_scoped_acquire gil;
  py::object pyModule = py::module_::import("torch.backends._nnapi.prepare");
  py::object pyMethod = pyModule.attr("process_for_nnapi");

  // Wrap the c module in a RecursiveScriptModule
  auto wrapped_mod =
      py::module::import("torch.jit._recursive").attr("wrap_cpp_module")(mod);
  wrapped_mod.attr("eval")();

```

- **EN:** Important callable entry points in this range include preprocess, import.
- **CN:** 这一段的重要可调用入口包括 preprocess, import。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Module API / 模块 API, Python binding / Python 绑定, Backend integration / 后端集成。

### Lines 41-60 / 第 41-60 行

```cpp
  // Test that method_compile_spec contains the necessary keys and
  // Tensor/TensorList input
  c10::IValue inp;
  std::string error;
  if (!method_compile_spec.contains("forward")) {
    error = R"(method_compile_spec does not contain the "forward" key.)";
  } else {
    auto innerDict = method_compile_spec.at("forward");
    if (!innerDict.isGenericDict() ||
        !innerDict.toGenericDict().contains("inputs")) {
      error =
          R"(method_compile_spec does not contain a dictionary with an "inputs" key, under it's "forward" key.)";
    } else {
      inp = innerDict.toGenericDict().at("inputs");
      if (!inp.isTensor() && !inp.isTensorList()) {
        error =
            R"(method_compile_spec does not contain either a Tensor or TensorList, under it's "inputs" key.)";
      }
    }
  }
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Module API / 模块 API, Backend integration / 后端集成, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Module API / 模块 API, Backend integration / 后端集成, Branching logic / 分支逻辑。

### Lines 61-80 / 第 61-80 行

```cpp
  if (!error.empty()) {
    throw std::runtime_error(
        error +
        "\nmethod_compile_spec should contain a Tensor or Tensor List which bundles input parameters:"
        " shape, dtype, quantization, and dimorder."
        "\nFor input shapes, use 0 for run/load time flexible input."
        "\nmethod_compile_spec must use the following format:"
        "\n{\"forward\": {\"inputs\": at::Tensor}} OR {\"forward\": {\"inputs\": c10::List<at::Tensor>}}");
  }

  // Convert input to a Tensor or a python list of Tensors
  py::list nnapi_processed;
  if (inp.isTensor()) {
    nnapi_processed = pyMethod(wrapped_mod, inp.toTensor());
  } else {
    py::list pyInp;
    for (at::Tensor inpElem : inp.toTensorList()) {
      pyInp.append(inpElem);
    }
    nnapi_processed = pyMethod(wrapped_mod, pyInp);
```

- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 81-100 / 第 81-100 行

```cpp
  }

  // Cast and insert processed items into dict
  c10::Dict<c10::IValue, c10::IValue> dict(
      c10::StringType::get(), c10::AnyType::get());
  dict.insert("ser_model", py::cast<at::Tensor>(nnapi_processed[1]));

  // Serialize shape_compute_module for mobile
  auto shape_compute_module =
      py::cast<torch::jit::Module>(nnapi_processed[0].attr("_c"));
  std::stringstream ss;
  shape_compute_module._save_for_mobile(ss);
  dict.insert("shape_compute_module", ss.str());

  // transform Python lists to C++ c10::List
  c10::List<at::Tensor> weights(
      py::cast<std::vector<at::Tensor>>(nnapi_processed[2]));
  for (auto i = 0U; i < weights.size(); i++) {
    weights.set(i, weights.get(i).contiguous());
  }
```

- **EN:** Important callable entry points in this range include dict, weights.
- **CN:** 这一段的重要可调用入口包括 dict, weights。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 101-114 / 第 101-114 行

```cpp
  c10::List<int64_t> inp_mem_fmts(
      py::cast<std::vector<int64_t>>(nnapi_processed[3]));
  c10::List<int64_t> out_mem_fmts(
      py::cast<std::vector<int64_t>>(nnapi_processed[4]));
  dict.insert("weights", weights);
  dict.insert("inp_mem_fmts", inp_mem_fmts);
  dict.insert("out_mem_fmts", out_mem_fmts);

  return dict;
}

constexpr auto backend_name = "nnapi";
static auto pre_reg =
    torch::jit::backend_preprocess_register(backend_name, preprocess);
```

- **EN:** Important callable entry points in this range include inp_mem_fmts, out_mem_fmts, backend_preprocess_register.
- **CN:** 这一段的重要可调用入口包括 inp_mem_fmts, out_mem_fmts, backend_preprocess_register。
- **EN:** Python-facing bindings or conversions appear here, showing how native JIT objects cross the C++/Python boundary.
- **CN:** 这里出现了面向 Python 的绑定或转换逻辑，说明原生 JIT 对象如何跨越 C++/Python 边界。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

## Key Concepts / 关键概念

- **Custom backend pipeline** — 自定义后端流水线
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Serialization** — 序列化
- **Mobile runtime** — 移动端运行时
- **Python binding** — Python 绑定
- **Backend integration** — 后端集成
- **Registration** — 注册机制

## Dependencies / 依赖关系

- `torch/csrc/jit/backends/backend_preprocess.h`
- `torch/csrc/utils/pybind.h`
