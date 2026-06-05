# TritonKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/TritonKernel.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for TritonKernel, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 TritonKernel 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/nativert/kernels/TritonKernel.h>

#include <c10/util/Exception.h>

#include <torch/nativert/executor/DelegateExecutor.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/kernels/TritonKernel.h`, `c10/util/Exception.h`, `torch/nativert/executor/DelegateExecutor.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/kernels/TritonKernel.h`, `c10/util/Exception.h`, `torch/nativert/executor/DelegateExecutor.h`；外部依赖：无。

### Lines 6-11
```cpp

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#endif

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `ATen/Functions.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`ATen/Functions.h`；外部依赖：无。

### Lines 12-16
```cpp
namespace torch::nativert {

// in this case, we want to use the symbol from torch_cpu.dll
#ifndef NATIVERT_MSVC_TEST
C10_DEFINE_TYPED_REGISTRY(
```
- EN: This block implements local helper logic for TritonKernel. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 TritonKernel 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 17-25
```cpp
    TritonKernelManagerRegistry,
    c10::DeviceType,
    TritonKernelManager,
    std::unique_ptr,
    std::string /* kernel_name */,
    std::string /* kernel_bin_path */,
    std::string /* kernel_launcher_bin_path */)
#endif

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 26-31
```cpp
TritonKernel::TritonKernel(
    const Node* node,
    caffe2::serialize::PyTorchStreamReader* reader)
    : OpKernel(node, OpKernelKind::kTritonKernel) {
  TORCH_CHECK(reader != nullptr, "reader is null");

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures. Key symbols: `TritonKernel`, `OpKernel`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构。关键符号：`TritonKernel`, `OpKernel`。

### Lines 32-37
```cpp
  std::string kernel_name{};
  std::string symbol_name{};

  // To prevent vector reallocation and dangling pointers
  size_t num_double_attrs = 0;
  for (const auto& attr : node_->attributes()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `attributes`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`attributes`。

### Lines 38-43
```cpp
    if (attr.name.empty() && std::holds_alternative<double>(attr.value)) {
      ++num_double_attrs;
    }
  }
  float_attrs_.reserve(num_double_attrs);

```
- EN: This block handles conditional control flow. Key symbols: `empty`, `reserve`.
- CN: 该代码块处理条件控制流。关键符号：`empty`, `reserve`。

### Lines 44-48
```cpp
  // Parse only TritonKernel-specific attributes here.
  // Launch parameters (grid, num_warps, etc.) are parsed by the target-specific
  // TritonKernelManager via createLaunchParams().
  for (const auto& attr : node_->attributes()) {
    if (attr.name.empty()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `attributes`, `empty`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`attributes`, `empty`。

### Lines 49-53
```cpp
      attr_ptrs_.emplace_back(std::visit(
          [this](auto&& arg) -> void* {
            using T = std::decay_t<decltype(arg)>;
            if constexpr (std::is_same_v<T, None>) {
              return nullptr;
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `emplace_back`, `visit`, `T`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`emplace_back`, `visit`, `T`。

### Lines 54-61
```cpp
            } else if constexpr (std::is_same_v<T, double>) {
              // Triton always uses fp32 for floats. See
              // create_specialize_impl in jit.py. However, due to the
              // Thrift schema, floats are serialized as doubles here. But,
              // Triton kernels read them as floats. So, we need to downcast
              // double to float here.
              float_attrs_.push_back(static_cast<float>(arg));
              return static_cast<void*>(&float_attrs_.back());
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `push_back`, `back`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`push_back`, `back`。

### Lines 62-71
```cpp
            }
            return static_cast<void*>(const_cast<T*>(&arg));
          },
          attr.value));
    } else if (attr.name == "name") {
      kernel_name = std::get<std::string>(attr.value);
      size_t last_underscore = kernel_name.find_last_of('_');
      symbol_name = kernel_name.substr(0, last_underscore);
    } else if (attr.name == "output_indices") {
      output_indices_ = std::get<std::vector<int64_t>>(attr.value);
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: `find_last_of`, `substr`.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：`find_last_of`, `substr`。

### Lines 72-81
```cpp
      kernel_input_params_.output_indices = output_indices_;
    } else if (attr.name == "kernel_param_names") {
      kernel_input_params_.kernel_param_names =
          std::get<std::vector<std::string>>(attr.value);
    } else if (attr.name == "kernel_param_types") {
      kernel_input_params_.kernel_param_types =
          std::get<std::vector<std::string>>(attr.value);
    }
  }

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 82-86
```cpp
  TORCH_CHECK(!kernel_name.empty(), "kernel name not found");
  TORCH_CHECK(!symbol_name.empty(), "symbol_name not found");
  TORCH_CHECK(!output_indices_.empty(), "output_indices attribute not found");

  auto kernel_prefix = std::string("data/triton") + "/" + kernel_name;
```
- EN: This block checks invariants or expected outcomes. Key symbols: `empty`, `string`.
- CN: 该代码块检查不变量或预期结果。关键符号：`empty`, `string`。

### Lines 87-96
```cpp

  auto tmp_dir = extractToTemporaryFolder(*reader, kernel_prefix) + "/";

  if (reader->hasRecord(kernel_prefix + "/" + kernel_name + ".cubin")) {
    loader_ = TritonKernelManagerRegistry()->Create(
        at::kCUDA, symbol_name, tmp_dir + kernel_name + ".cubin", "");
    TORCH_CHECK(
        loader_ != nullptr,
        "couldn't find cuda loader -- is this a gpu build?");
  } else if (reader->hasRecord(kernel_prefix + "/" + kernel_name + ".hsaco")) {
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: `extractToTemporaryFolder`, `hasRecord`, `TritonKernelManagerRegistry`, `Create`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：`extractToTemporaryFolder`, `hasRecord`, `TritonKernelManagerRegistry`, `Create`。

### Lines 97-106
```cpp
    loader_ = TritonKernelManagerRegistry()->Create(
        at::kHIP, symbol_name, tmp_dir + kernel_name + ".hsaco", "");
    TORCH_CHECK(
        loader_ != nullptr,
        "couldn't find cuda loader -- is this a gpu build?");
  } else if (reader->hasRecord(kernel_prefix + "/" + kernel_name + ".so")) {
    loader_ = TritonKernelManagerRegistry()->Create(
        at::kCPU,
        symbol_name,
        tmp_dir + kernel_name + ".so",
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: `TritonKernelManagerRegistry`, `Create`, `hasRecord`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：`TritonKernelManagerRegistry`, `Create`, `hasRecord`。

### Lines 107-116
```cpp
        tmp_dir + kernel_name + ".launcher.so");
    TORCH_CHECK(
        loader_ != nullptr, "couldn't find CPU loader -- is this a cpu build?");
  } else if (reader->hasRecord(kernel_prefix + "/" + kernel_name + ".bin")) {
    loader_ = TritonKernelManagerRegistry()->Create(
        at::kMTIA, symbol_name, tmp_dir + kernel_name + ".bin", "");
    TORCH_CHECK(
        loader_ != nullptr,
        "couldn't find MTIA loader -- is this a mtia build?");
  }
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: `hasRecord`, `TritonKernelManagerRegistry`, `Create`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：`hasRecord`, `TritonKernelManagerRegistry`, `Create`。

### Lines 117-121
```cpp

  TORCH_CHECK(
      loader_ != nullptr,
      "couldn't find triton kernel loader -- are you trying to run gpu kernels on a cpu build?");

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 122-127
```cpp
  // Create target-specific launch parameters
  launch_params_ = loader_->createLaunchParams(node_);
}

TritonKernel::~TritonKernel() = default;

```
- EN: This block manipulates graph-like program structures. Key symbols: `createLaunchParams`, `~TritonKernel`.
- CN: 该代码块操作图状程序结构。关键符号：`createLaunchParams`, `~TritonKernel`。

### Lines 128-133
```cpp
void TritonKernel::computeInternal(ExecutionFrame& executionFrame) const {
  const auto num_inputs = node_->inputs().size();
  const auto num_attrs = attr_ptrs_.size();

  auto* loader = const_cast<TritonKernelManager*>(loader_.get());

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `computeInternal`, `inputs`, `size`, `get`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`computeInternal`, `inputs`, `size`, `get`。

### Lines 134-140
```cpp
  auto inputs =
      loader->create_inputs(num_inputs, num_attrs, kernel_input_params_);

  for (const auto i : c10::irange(num_inputs)) {
    inputs->add_tensor_arg(input(i, executionFrame).toTensor());
  }

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `create_inputs`, `irange`, `add_tensor_arg`, `input`, `toTensor`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；遍历集合或执行单元。关键符号：`create_inputs`, `irange`, `add_tensor_arg`, `input`, `toTensor`。

### Lines 141-146
```cpp
  for (const auto i : c10::irange(num_attrs)) {
    inputs->add_attribute(attr_ptrs_[i]);
  }

  loader->launch(*launch_params_, inputs->as_void());

```
- EN: This block iterates over collections or execution units. Key symbols: `irange`, `add_attribute`, `launch`, `as_void`.
- CN: 该代码块遍历集合或执行单元。关键符号：`irange`, `add_attribute`, `launch`, `as_void`。

### Lines 147-156
```cpp
  auto& out = output(0, executionFrame);
  if (out.isNone()) {
    auto list = c10::List<at::Tensor>();
    for (const auto& i : output_indices_) {
      list.emplace_back(input(i, executionFrame).toTensor());
    }
    out = c10::IValue(std::move(list));
    return;
  }

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `output`, `isNone`, `emplace_back`, `input`, `toTensor`, `IValue`, `...`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`output`, `isNone`, `emplace_back`, `input`, `toTensor`, `IValue`, `...`。

### Lines 157-163
```cpp
  // todo: check if this is redundant
  auto out_t = out.toTensorList();
  for (const auto i : c10::irange(output_indices_.size())) {
    out_t[i] = input(output_indices_[i], executionFrame).toTensor();
  }
}

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `toTensorList`, `irange`, `size`, `input`, `toTensor`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`toTensorList`, `irange`, `size`, `input`, `toTensor`。

### Lines 164-164
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for TritonKernel. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 TritonKernel 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/kernels/TritonKernel.h`, `c10/util/Exception.h`, `torch/nativert/executor/DelegateExecutor.h`, `ATen/Functions.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `TritonKernel`, `OpKernel`, `attributes`, `empty`, `reserve`, `emplace_back`, `visit`, `T`, `push_back`, `back`, `...`
