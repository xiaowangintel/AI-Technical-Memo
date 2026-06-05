# Bindings.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/python/Bindings.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for Bindings, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 Bindings 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <unordered_map>

#include <torch/csrc/jit/python/pybind_utils.h>
#include <torch/csrc/utils/pybind.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/csrc/jit/python/pybind_utils.h`, `torch/csrc/utils/pybind.h`; external includes: `unordered_map`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/csrc/jit/python/pybind_utils.h`, `torch/csrc/utils/pybind.h`；外部依赖：`unordered_map`。

### Lines 6-10
```cpp
#ifdef FBCODE_CAFFE2
#include <torch/nativert/ModelRunner.h>

namespace py = pybind11;

```
- EN: This block implements local helper logic for Bindings. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 Bindings 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 11-15
```cpp
template <typename T>
using shared_ptr_class_ = py::class_<T, std::shared_ptr<T>>;

namespace torch::nativert {

```
- EN: This block implements local helper logic for Bindings. Key symbols: `shared_ptr_class_`.
- CN: 该代码块实现与 Bindings 相关的局部辅助逻辑。关键符号：`shared_ptr_class_`。

### Lines 16-25
```cpp
using torch::nativert::detail::argsToIValue;

void initModelRunnerPybind(py::module& m) {
#if !defined(OVRSOURCE)
  shared_ptr_class_<ModelRunner>(m, "PyModelRunner")
      .def(
          py::init<const std::string&, const std::string&>(),
          py::arg("packagePath"),
          py::arg("modelName"))
      .def(
```
- EN: This block handles conditional control flow. Key symbols: `torch`, `initModelRunnerPybind`, `def`, `arg`.
- CN: 该代码块处理条件控制流。关键符号：`torch`, `initModelRunnerPybind`, `def`, `arg`。

### Lines 26-32
```cpp
          "run",
          [](torch::nativert::ModelRunner& self,
             py::args pyargs,
             const py::kwargs& pykwargs) {
            std::vector<c10::IValue> args;
            args.reserve(pyargs.size());
            for (const auto i : c10::irange(pyargs.size())) {
```
- EN: This block iterates over collections or execution units. Key symbols: `reserve`, `size`, `irange`.
- CN: 该代码块遍历集合或执行单元。关键符号：`reserve`, `size`, `irange`。

### Lines 33-39
```cpp
              auto ivalue =
                  torch::jit::toIValue(pyargs[i], c10::AnyType::get());
              args.push_back(std::move(ivalue));
            }
            std::unordered_map<std::string, c10::IValue> kwargs;
            kwargs.reserve(pykwargs.size());
            for (const auto& [key, pyarg] : pykwargs) {
```
- EN: This block iterates over collections or execution units. Key symbols: `toIValue`, `get`, `push_back`, `move`, `reserve`, `size`.
- CN: 该代码块遍历集合或执行单元。关键符号：`toIValue`, `get`, `push_back`, `move`, `reserve`, `size`。

### Lines 40-44
```cpp
              auto ivalue = torch::jit::toIValue(pyarg, c10::AnyType::get());
              kwargs[py::str(key)] = std::move(ivalue);
            }
            c10::IValue ret = self.run(args, kwargs);
            return torch::jit::createPyObjectForStack({ret});
```
- EN: This block returns results to callers or downstream stages. Key symbols: `toIValue`, `get`, `str`, `move`, `run`, `createPyObjectForStack`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`toIValue`, `get`, `str`, `move`, `run`, `createPyObjectForStack`。

### Lines 45-53
```cpp
          })
      .def(
          "__call__",
          [](torch::nativert::ModelRunner& self,
             py::args pyargs,
             const py::kwargs& pykwargs) {
            std::vector<c10::IValue> args;
            args.reserve(pyargs.size());
            for (const auto i : c10::irange(pyargs.size())) {
```
- EN: This block iterates over collections or execution units. Key symbols: `def`, `reserve`, `size`, `irange`.
- CN: 该代码块遍历集合或执行单元。关键符号：`def`, `reserve`, `size`, `irange`。

### Lines 54-60
```cpp
              auto ivalue =
                  torch::jit::toIValue(pyargs[i], c10::AnyType::get());
              args.push_back(std::move(ivalue));
            }
            std::unordered_map<std::string, c10::IValue> kwargs;
            kwargs.reserve(pykwargs.size());
            for (const auto& [key, pyarg] : pykwargs) {
```
- EN: This block iterates over collections or execution units. Key symbols: `toIValue`, `get`, `push_back`, `move`, `reserve`, `size`.
- CN: 该代码块遍历集合或执行单元。关键符号：`toIValue`, `get`, `push_back`, `move`, `reserve`, `size`。

### Lines 61-65
```cpp
              auto ivalue = torch::jit::toIValue(pyarg, c10::AnyType::get());
              kwargs[py::str(key)] = std::move(ivalue);
            }
            c10::IValue ret = self.run(args, kwargs);
            return torch::jit::createPyObjectForStack({ret});
```
- EN: This block returns results to callers or downstream stages. Key symbols: `toIValue`, `get`, `str`, `move`, `run`, `createPyObjectForStack`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`toIValue`, `get`, `str`, `move`, `run`, `createPyObjectForStack`。

### Lines 66-72
```cpp
          })
      .def(
          "run_with_flat_inputs_and_outputs",
          [](torch::nativert::ModelRunner& self, py::args pyargs) {
            std::vector<c10::IValue> args;
            args.reserve(pyargs.size());
            for (const auto i : c10::irange(pyargs.size())) {
```
- EN: This block iterates over collections or execution units. Key symbols: `def`, `reserve`, `size`, `irange`.
- CN: 该代码块遍历集合或执行单元。关键符号：`def`, `reserve`, `size`, `irange`。

### Lines 73-77
```cpp
              auto ivalue =
                  torch::jit::toIValue(pyargs[i], c10::AnyType::get());
              args.push_back(std::move(ivalue));
            }

```
- EN: This block implements local helper logic for Bindings. Key symbols: `toIValue`, `get`, `push_back`, `move`.
- CN: 该代码块实现与 Bindings 相关的局部辅助逻辑。关键符号：`toIValue`, `get`, `push_back`, `move`。

### Lines 78-83
```cpp
            auto rets = self.runWithFlatInputsAndOutputs(std::move(args));
            return torch::jit::createPyObjectForStack(std::move(rets));
          });
#endif // !defined(OVRSOURCE)
}

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `runWithFlatInputsAndOutputs`, `move`, `createPyObjectForStack`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`runWithFlatInputsAndOutputs`, `move`, `createPyObjectForStack`。

### Lines 84-88
```cpp
} // namespace torch::nativert

#else // !FBCODE_CAFFE2

namespace py = pybind11;
```
- EN: This block implements local helper logic for Bindings. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 Bindings 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 89-93
```cpp

namespace torch::nativert {

class StubModelRunner {};

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `StubModelRunner`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`StubModelRunner`。

### Lines 94-100
```cpp
// PyModelRunner is referenced from
// https://github.com/pytorch/benchmark/blob/b8d35ba51a3149b7212888b4010ddee97f19947f/userbenchmark/dynamo/dynamobench/common.py#L45
void initModelRunnerPybind(py::module& m) {
  py::class_<StubModelRunner, std::shared_ptr<StubModelRunner>>(
      m, "PyModelRunner");
}

```
- EN: This block implements local helper logic for Bindings. Key symbols: `initModelRunnerPybind`.
- CN: 该代码块实现与 Bindings 相关的局部辅助逻辑。关键符号：`initModelRunnerPybind`。

### Lines 101-103
```cpp
} // namespace torch::nativert

#endif // FBCODE_CAFFE2
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/csrc/jit/python/pybind_utils.h`, `torch/csrc/utils/pybind.h`, `torch/nativert/ModelRunner.h`
- External includes / 外部头文件: `unordered_map`
- Namespaces / 命名空间: `py`, `torch::nativert`
- Representative symbols / 代表性符号: `shared_ptr_class_`, `torch`, `initModelRunnerPybind`, `def`, `arg`, `reserve`, `size`, `irange`, `toIValue`, `get`, `...`
