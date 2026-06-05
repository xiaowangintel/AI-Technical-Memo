# ModelRunner.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/ModelRunner.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for ModelRunner, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 ModelRunner 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/nativert/ModelRunner.h>

#include <variant>

#include <nlohmann/json.hpp>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/ModelRunner.h`; external includes: `variant`, `nlohmann/json.hpp`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/ModelRunner.h`；外部依赖：`variant`, `nlohmann/json.hpp`。

### Lines 6-10
```cpp

#include <caffe2/serialize/file_adapter.h>
#include <torch/csrc/export/pt2_archive_constants.h>
#include <torch/nativert/executor/Placement.h>
#include <torch/nativert/graph/GraphPasses.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/csrc/export/pt2_archive_constants.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphPasses.h`; external includes: `caffe2/serialize/file_adapter.h`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/csrc/export/pt2_archive_constants.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphPasses.h`；外部依赖：`caffe2/serialize/file_adapter.h`。

### Lines 11-15
```cpp
#include <torch/nativert/graph/Serialization.h>
#include <torch/nativert/kernels/KernelHandlerRegistry.h>

namespace torch::nativert {

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 16-23
```cpp
using torch::nativert::jsonToGraph;
using torch::nativert::detail::itreeSpecLoads;

ModelRunner::ModelRunner(
    const std::string& packagePath,
    const std::string& modelName) {
  register_kernel_handlers();
  auto pytorchStreamReader =
```
- EN: This block manipulates graph-like program structures. Key symbols: `torch`, `ModelRunner`, `register_kernel_handlers`.
- CN: 该代码块操作图状程序结构。关键符号：`torch`, `ModelRunner`, `register_kernel_handlers`。

### Lines 24-29
```cpp
      std::make_shared<caffe2::serialize::PyTorchStreamReader>(
          std::make_unique<caffe2::serialize::FileAdapter>(packagePath));
  std::string modelFilePath = fmt::format(
      torch::_export::archive_spec::MODELS_FILENAME_FORMAT, modelName);
  LOG(INFO) << "Loading model from: " << modelFilePath;

```
- EN: This block advances ONNX export translation. Key symbols: `format`.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：`format`。

### Lines 30-38
```cpp
  TORCH_CHECK(
      pytorchStreamReader->hasRecord(modelFilePath),
      modelFilePath,
      " not found in package");
  const auto& [modelData, modelSize] =
      pytorchStreamReader->getRecord(modelFilePath);
  const std::string modelSerialized{
      reinterpret_cast<char*>(modelData.get()), modelSize};

```
- EN: This block checks invariants or expected outcomes. Key symbols: `hasRecord`, `getRecord`, `get`.
- CN: 该代码块检查不变量或预期结果。关键符号：`hasRecord`, `getRecord`, `get`。

### Lines 39-46
```cpp
  exportedProgram_ = nlohmann::json::parse(modelSerialized)
                         .template get<torch::_export::ExportedProgram>();

  TORCH_CHECK(exportedProgram_.get_graph_module()
                  .get_module_call_graph()[0]
                  .get_fqn()
                  .empty());

```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: `parse`, `get_graph_module`, `get_module_call_graph`, `get_fqn`, `empty`.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：`parse`, `get_graph_module`, `get_module_call_graph`, `get_fqn`, `empty`。

### Lines 47-51
```cpp
  tensorPaths_ = getPayloadConfig(
      pytorchStreamReader,
      torch::_export::archive_spec::WEIGHTS_CONFIG_FILENAME_FORMAT,
      modelName);

```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation. Key symbols: `getPayloadConfig`.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程。关键符号：`getPayloadConfig`。

### Lines 52-56
```cpp
  constantPaths_ = getPayloadConfig(
      pytorchStreamReader,
      torch::_export::archive_spec::CONSTANTS_CONFIG_FILENAME_FORMAT,
      modelName);

```
- EN: This block advances ONNX export translation. Key symbols: `getPayloadConfig`.
- CN: 该代码块推进 ONNX 导出翻译流程。关键符号：`getPayloadConfig`。

### Lines 57-66
```cpp
  graph_ = jsonToGraph(exportedProgram_.get_graph_module());

  std::vector<const Value*> userInputs(
      graph_->userInputs().begin(), graph_->userInputs().end());
  const auto& signatureOpt = exportedProgram_.get_graph_module()
                                 .get_module_call_graph()[0]
                                 .get_signature();
  // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
  const auto& signature = signatureOpt.value();
  inputSpec_ = itreeSpecLoads(signature.get_in_spec(), userInputs);
```
- EN: This block checks invariants or expected outcomes; advances ONNX export translation; manipulates graph-like program structures. Key symbols: `jsonToGraph`, `get_graph_module`, `userInputs`, `begin`, `end`, `get_module_call_graph`, `...`.
- CN: 该代码块检查不变量或预期结果；推进 ONNX 导出翻译流程；操作图状程序结构。关键符号：`jsonToGraph`, `get_graph_module`, `userInputs`, `begin`, `end`, `get_module_call_graph`, `...`。

### Lines 67-71
```cpp

  const auto& userOutputs = graph_->userOutputs();
  std::vector<const Value*> updatedUserOutput(userOutputs.size(), nullptr);
  for (size_t i = 0; i < userOutputs.size(); ++i) {
    if (const auto* valuePtr = std::get_if<Value*>(&userOutputs[i])) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `userOutputs`, `updatedUserOutput`, `size`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`userOutputs`, `updatedUserOutput`, `size`。

### Lines 72-76
```cpp
      updatedUserOutput[i] = *valuePtr;
    }
  }
  outputSpec_ = itreeSpecLoads(signature.get_out_spec(), updatedUserOutput);

```
- EN: This block implements local helper logic for ModelRunner. Key symbols: `itreeSpecLoads`, `get_out_spec`.
- CN: 该代码块实现与 ModelRunner 相关的局部辅助逻辑。关键符号：`itreeSpecLoads`, `get_out_spec`。

### Lines 77-81
```cpp
  torch::nativert::Placement placement;

  graph_->applyDevicePlacement(placement);
  selectScalarOverload(graph_.get());

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: `applyDevicePlacement`, `selectScalarOverload`, `get`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：`applyDevicePlacement`, `selectScalarOverload`, `get`。

### Lines 82-88
```cpp
  auto weights = loadWeightsDefault(*graph_, pytorchStreamReader);

  weights->validateAllWeightsLoaded();

  torch::nativert::ExecutorConfig config;
  config.modelName = modelName;

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `loadWeightsDefault`, `validateAllWeightsLoaded`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；协调运行时执行状态。关键符号：`loadWeightsDefault`, `validateAllWeightsLoaded`。

### Lines 89-98
```cpp
  executor_ = std::make_unique<Executor>(
      config, graph_, std::move(weights), pytorchStreamReader);
}

std::unordered_map<std::string, std::string> ModelRunner::getPayloadConfig(
    const std::shared_ptr<caffe2::serialize::PyTorchStreamReader>&
        pytorchStreamReader,
    std::string_view configFilenameFormat,
    const std::string& modelName) {
  std::string configPath =
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `move`, `getPayloadConfig`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`move`, `getPayloadConfig`。

### Lines 99-105
```cpp
      fmt::format(fmt::runtime(configFilenameFormat), modelName);

  TORCH_CHECK(
      pytorchStreamReader->hasRecord(configPath),
      configPath,
      " not found in package");

```
- EN: This block checks invariants or expected outcomes. Key symbols: `format`, `runtime`, `hasRecord`.
- CN: 该代码块检查不变量或预期结果。关键符号：`format`, `runtime`, `hasRecord`。

### Lines 106-110
```cpp
  const auto& [configData, configSize] =
      pytorchStreamReader->getRecord(configPath);
  const std::string configSerialized{
      reinterpret_cast<char*>(configData.get()), configSize};

```
- EN: This block implements local helper logic for ModelRunner. Key symbols: `getRecord`, `get`.
- CN: 该代码块实现与 ModelRunner 相关的局部辅助逻辑。关键符号：`getRecord`, `get`。

### Lines 111-115
```cpp
  auto configJson = nlohmann::json::parse(configSerialized)
                        .template get<torch::_export::PayloadConfig>();
  auto config = configJson.get_config();
  std::unordered_map<std::string, std::string> targetPaths;
  for (const auto& configEntry : config) {
```
- EN: This block advances ONNX export translation; iterates over collections or execution units. Key symbols: `parse`, `get_config`.
- CN: 该代码块推进 ONNX 导出翻译流程；遍历集合或执行单元。关键符号：`parse`, `get_config`。

### Lines 116-120
```cpp
    targetPaths[configEntry.first] = configEntry.second.get_path_name();
  }
  return targetPaths;
}

```
- EN: This block returns results to callers or downstream stages. Key symbols: `get_path_name`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`get_path_name`。

### Lines 121-130
```cpp
std::shared_ptr<Weights> ModelRunner::loadWeightsDefault(
    Graph& graph,
    const std::shared_ptr<caffe2::serialize::PyTorchStreamReader>& reader) {
  return std::make_shared<Weights>(
      &graph,
      reader,
      tensorPaths_,
      torch::_export::archive_spec::WEIGHTS_DIR,
      constantPaths_,
      torch::_export::archive_spec::CONSTANTS_DIR);
```
- EN: This block handles tensor metadata or sample values; advances ONNX export translation; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `loadWeightsDefault`.
- CN: 该代码块处理张量元数据或示例值；推进 ONNX 导出翻译流程；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`loadWeightsDefault`。

### Lines 131-137
```cpp
}

c10::IValue ModelRunner::run(
    const std::vector<c10::IValue>& args,
    const std::unordered_map<std::string, c10::IValue>& kwargs) {
  TORCH_CHECK(executor_, "ModelRunner not initialized");

```
- EN: This block checks invariants or expected outcomes; coordinates runtime execution state. Key symbols: `run`.
- CN: 该代码块检查不变量或预期结果；协调运行时执行状态。关键符号：`run`。

### Lines 138-144
```cpp
  // ModelRunner is only used for inference
  c10::InferenceMode mode;

  return itreeUnflatten(
      executor_->execute(args, kwargs, inputSpec_), outputSpec_);
}

```
- EN: This block coordinates runtime execution state; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `itreeUnflatten`, `execute`.
- CN: 该代码块协调运行时执行状态；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`itreeUnflatten`, `execute`。

### Lines 145-151
```cpp
std::vector<c10::IValue> ModelRunner::runWithFlatInputsAndOutputs(
    std::vector<c10::IValue> flatInputs) {
  TORCH_CHECK(executor_, "ModelRunner not initialized");

  // ModelRunner is only used for inference
  c10::InferenceMode mode;

```
- EN: This block checks invariants or expected outcomes; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `runWithFlatInputsAndOutputs`.
- CN: 该代码块检查不变量或预期结果；协调运行时执行状态；遍历集合或执行单元。关键符号：`runWithFlatInputsAndOutputs`。

### Lines 152-157
```cpp
  return executor_->execute(std::move(flatInputs));
}

uint64_t ModelRunner::numOutputs() const {
  TORCH_CHECK(executor_, "ModelRunner not initialized");
  return executor_->graphSignature().userOutputs().size();
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `execute`, `move`, `numOutputs`, `graphSignature`, `userOutputs`, `size`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`execute`, `move`, `numOutputs`, `graphSignature`, `userOutputs`, `size`。

### Lines 158-165
```cpp
}

ModelRunnerHandle::ModelRunnerHandle(
    const std::string& packagePath,
    const std::string& modelName)
    : impl_(std::make_unique<ModelRunner>(packagePath, modelName)) {}
ModelRunnerHandle::~ModelRunnerHandle() = default;

```
- EN: This block implements local helper logic for ModelRunner. Key symbols: `ModelRunnerHandle`, `impl_`, `~ModelRunnerHandle`.
- CN: 该代码块实现与 ModelRunner 相关的局部辅助逻辑。关键符号：`ModelRunnerHandle`, `impl_`, `~ModelRunnerHandle`。

### Lines 166-171
```cpp
c10::IValue ModelRunnerHandle::run(
    const std::vector<c10::IValue>& args,
    const std::unordered_map<std::string, c10::IValue>& kwargs) {
  return impl_->run(args, kwargs);
}

```
- EN: This block returns results to callers or downstream stages. Key symbols: `run`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`run`。

### Lines 172-176
```cpp
std::vector<c10::IValue> ModelRunnerHandle::runWithFlatInputsAndOutputs(
    std::vector<c10::IValue> flatInputs) {
  return impl_->runWithFlatInputsAndOutputs(std::move(flatInputs));
}

```
- EN: This block returns results to callers or downstream stages. Key symbols: `runWithFlatInputsAndOutputs`, `move`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`runWithFlatInputsAndOutputs`, `move`。

### Lines 177-177
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for ModelRunner. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ModelRunner 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/ModelRunner.h`, `torch/csrc/export/pt2_archive_constants.h`, `torch/nativert/executor/Placement.h`, `torch/nativert/graph/GraphPasses.h`, `torch/nativert/graph/Serialization.h`, `torch/nativert/kernels/KernelHandlerRegistry.h`
- External includes / 外部头文件: `variant`, `nlohmann/json.hpp`, `caffe2/serialize/file_adapter.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `torch`, `ModelRunner`, `register_kernel_handlers`, `format`, `hasRecord`, `getRecord`, `get`, `parse`, `get_graph_module`, `get_module_call_graph`, `...`
