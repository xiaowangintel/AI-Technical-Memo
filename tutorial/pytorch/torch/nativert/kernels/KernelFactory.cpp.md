# KernelFactory.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/KernelFactory.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for KernelFactory, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 KernelFactory 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <string_view>

#include <c10/util/string_view.h>
#include <fmt/ranges.h>

#include <torch/nativert/executor/DelegateExecutor.h>
#include <torch/nativert/executor/OpKernel.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/string_view.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/OpKernel.h`; external includes: `string_view`, `fmt/ranges.h`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/string_view.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/OpKernel.h`；外部依赖：`string_view`, `fmt/ranges.h`。

### Lines 8-14
```cpp
#include <torch/nativert/executor/ParallelGraphExecutor.h>
#include <torch/nativert/executor/SerialGraphExecutor.h>
#include <torch/nativert/graph/Graph.h>
#include <torch/nativert/kernels/AutoFunctionalizeKernel.h>
#include <torch/nativert/kernels/C10Kernel.h>
#include <torch/nativert/kernels/CallTorchBindKernel.h>
#include <torch/nativert/kernels/HigherOrderKernel.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/executor/ParallelGraphExecutor.h`, `torch/nativert/executor/SerialGraphExecutor.h`, `torch/nativert/graph/Graph.h`, `torch/nativert/kernels/AutoFunctionalizeKernel.h`, `torch/nativert/kernels/C10Kernel.h`, `torch/nativert/kernels/CallTorchBindKernel.h`, `...`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/executor/ParallelGraphExecutor.h`, `torch/nativert/executor/SerialGraphExecutor.h`, `torch/nativert/graph/Graph.h`, `torch/nativert/kernels/AutoFunctionalizeKernel.h`, `torch/nativert/kernels/C10Kernel.h`, `torch/nativert/kernels/CallTorchBindKernel.h`, `...`；外部依赖：无。

### Lines 15-21
```cpp
#include <torch/nativert/kernels/KernelFactory.h>
#include <torch/nativert/kernels/PrimKernelRegistry.h>
#include <torch/nativert/kernels/TritonKernel.h>

namespace torch::nativert {

inline constexpr std::array<std::string_view, 7> kSymIntOps = {
```
- EN: This block implements local helper logic for KernelFactory. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 KernelFactory 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 22-29
```cpp
    "_operator.floordiv",
    "_operator.mod",
    "torch.sym_int",
    "torch.sym_float",
    "torch.sym_ite",
    "torch.sym_max",
    "torch.sym_min",
};
```
- EN: This block implements local helper logic for KernelFactory. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 KernelFactory 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 30-40
```cpp

inline constexpr std::array<std::string_view, 8> kSymBoolOps = {
    "_operator.eq",
    "_operator.ne",
    "_operator.le",
    "_operator.ge",
    "_operator.lt",
    "_operator.gt",
    "_operator.and_",
    "torch.sym_not",
};
```
- EN: This block implements local helper logic for KernelFactory. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 KernelFactory 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 41-47
```cpp

inline constexpr std::array<std::string_view, 4> kSymFloatOps = {
    "torch._sym_sqrt",
    "math.trunc",
    "_operator.neg",
    "_operator.truediv",
};
```
- EN: This block implements local helper logic for KernelFactory. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 KernelFactory 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 48-54
```cpp

inline constexpr std::array<std::string_view, 4> kScalarBinaryOps = {
    "_operator.mul",
    "_operator.add",
    "_operator.sub",
    "_operator.pow",
};
```
- EN: This block implements local helper logic for KernelFactory. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 KernelFactory 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 55-61
```cpp

namespace {

struct KernelFactoryRegistry {
  std::unordered_map<std::string, KernelFactoryHandler> handlers;
};

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `KernelFactoryRegistry`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`KernelFactoryRegistry`。

### Lines 62-68
```cpp
c10::Synchronized<KernelFactoryRegistry>& getKernelFactoryRegistry() {
  static auto* registry = new c10::Synchronized<KernelFactoryRegistry>();
  return *registry;
}

} // namespace

```
- EN: This block returns results to callers or downstream stages. Key symbols: `getKernelFactoryRegistry`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`getKernelFactoryRegistry`。

### Lines 69-80
```cpp
void KernelFactory::registerHandler(
    const std::string& name,
    KernelFactoryHandler handler) {
  auto& registry = getKernelFactoryRegistry();
  registry.withLock([&](auto&& reg) {
    if (reg.handlers.find(name) != reg.handlers.end()) {
      TORCH_CHECK(false, "Handler for ", name, " already registered");
    }
    reg.handlers.emplace(name, std::move(handler));
  });
}

```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `registerHandler`, `getKernelFactoryRegistry`, `withLock`, `find`, `end`, `emplace`, `...`.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`registerHandler`, `getKernelFactoryRegistry`, `withLock`, `find`, `end`, `emplace`, `...`。

### Lines 81-87
```cpp
/* static */ bool KernelFactory::isHandlerRegistered(
    const std::string& handler) {
  return getKernelFactoryRegistry().withLock([&](auto&& reg) {
    return reg.handlers.find(handler) != reg.handlers.end();
  });
}

```
- EN: This block returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `isHandlerRegistered`, `getKernelFactoryRegistry`, `withLock`, `find`, `end`.
- CN: 该代码块向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`isHandlerRegistered`, `getKernelFactoryRegistry`, `withLock`, `find`, `end`。

### Lines 88-97
```cpp
ExecutionKernels KernelFactory::initializeNodeKernels(
    const Graph& graph,
    const std::shared_ptr<Weights>& weights,
    const torch::nativert::ExecutorConfig& executorConfig,
    const std::shared_ptr<caffe2::serialize::PyTorchStreamReader>&
        pytorchStreamReader) {
  std::vector<std::unique_ptr<OpKernel>> nodeKernels;
  std::vector<std::unique_ptr<DelegateExecutor>> delegateExecutors;
  std::vector<ConstFoldingExecution> constFoldingExecutions;

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `initializeNodeKernels`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`initializeNodeKernels`。

### Lines 98-106
```cpp
  std::unordered_map<std::string, int> opsWithoutStaticDispatchCount;

  VLOG(1) << fmt::format(
      "PrimKernelRegistry: {}", fmt::join(PrimKernelRegistry()->Keys(), ", "));

  std::unordered_map<std::string, KernelFactoryHandler> handlers;
  getKernelFactoryRegistry().withLock(
      [&](auto&& reg) { handlers = reg.handlers; });

```
- EN: This block protects shared state or ordering guarantees. Key symbols: `format`, `join`, `PrimKernelRegistry`, `Keys`, `getKernelFactoryRegistry`, `withLock`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`format`, `join`, `PrimKernelRegistry`, `Keys`, `getKernelFactoryRegistry`, `withLock`。

### Lines 107-113
```cpp
  for (const auto& node : graph.nodes()) {
    std::string target = std::string(node.target());

    bool matched = false;
    for (const auto& [_, handler] : handlers) {
      if (handler.match(node, executorConfig)) {
        auto [kernel, delegate] =
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `nodes`, `string`, `target`, `match`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`nodes`, `string`, `target`, `match`。

### Lines 114-125
```cpp
            handler(node, weights, executorConfig, pytorchStreamReader.get());
        if (kernel) {
          nodeKernels.push_back(std::move(kernel));
        }
        if (delegate) {
          delegateExecutors.push_back(std::move(delegate));
        }
        matched = true;
        break;
      }
    }
    if (matched) {
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow. Key symbols: `handler`, `get`, `push_back`, `move`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流。关键符号：`handler`, `get`, `push_back`, `move`。

### Lines 126-139
```cpp
      continue;
    }

    if (PrimKernelRegistry()->Has(target)) {
      nodeKernels.push_back(PrimKernelRegistry()->Create(target, &node));
    } else if (c10::starts_with(
                   node.target(), "torch.ops.higher_order.call_torchbind")) {
      nodeKernels.push_back(std::make_unique<CallTorchBindKernel>(&node));
    } else if (c10::starts_with(
                   node.target(),
                   "torch.ops.higher_order.triton_kernel_wrapper_functional")) {
      nodeKernels.push_back(
          std::make_unique<TritonKernel>(&node, pytorchStreamReader.get()));
    } else if (
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `PrimKernelRegistry`, `Has`, `push_back`, `Create`, `starts_with`, `target`, `...`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`PrimKernelRegistry`, `Has`, `push_back`, `Create`, `starts_with`, `target`, `...`。

### Lines 140-153
```cpp
        c10::starts_with(
            node.target(),
            "torch.ops.higher_order.auto_functionalized") ||
        c10::starts_with( // TODO Remove this condition once the old
                          // pt2 archives are expired.
            node.target(),
            "torch._higher_order_ops.auto_functionalize.auto_functionalized")) {
      nodeKernels.push_back(
          std::make_unique<UnsafeAutoFunctionalizeKernel>(&node));
    } else if (
        std::find(
            std::begin(kSymIntOps), std::end(kSymIntOps), node.target()) !=
        std::end(kSymIntOps)) {
      nodeKernels.push_back(std::make_unique<SymIntOpKernel>(&node));
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `starts_with`, `target`, `push_back`, `find`, `begin`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`starts_with`, `target`, `push_back`, `find`, `begin`, `end`。

### Lines 154-167
```cpp
    } else if (
        std::find(
            std::begin(kSymBoolOps), std::end(kSymBoolOps), node.target()) !=
        std::end(kSymBoolOps)) {
      nodeKernels.push_back(std::make_unique<SymBoolOpKernel>(&node));
    } else if (
        std::find(
            std::begin(kSymFloatOps), std::end(kSymFloatOps), node.target()) !=
        std::end(kSymFloatOps)) {
      nodeKernels.push_back(std::make_unique<SymFloatOpKernel>(&node));
    } else if (
        std::find(
            std::begin(kScalarBinaryOps),
            std::end(kScalarBinaryOps),
```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `find`, `begin`, `end`, `target`, `push_back`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`find`, `begin`, `end`, `target`, `push_back`。

### Lines 168-175
```cpp
            node.target()) != std::end(kScalarBinaryOps)) {
      nodeKernels.push_back(std::make_unique<ScalarBinaryOpKernel>(&node));
    } else if (c10::starts_with(node.target(), "torch.ops.higher_order")) {
      std::vector<std::unique_ptr<GraphExecutorBase>> graphExecutors;
      for (const auto& attr : node.attributes()) {
        if (std::holds_alternative<std::unique_ptr<Graph>>(attr.value)) {
          const auto& subgraph = std::get<std::unique_ptr<Graph>>(attr.value);
          auto executionKernels =
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `target`, `end`, `push_back`, `starts_with`, `attributes`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`target`, `end`, `push_back`, `starts_with`, `attributes`。

### Lines 176-183
```cpp
              initializeNodeKernels(*subgraph, weights, executorConfig);
          TORCH_CHECK(
              executionKernels.delegateExecutors.empty(),
              "HigherOrderKernel does not support delegates");
          TORCH_CHECK(
              executionKernels.constFoldingExecutions.empty(),
              "HigherOrderKernel does not support const folding");
          if (executorConfig.maxParallelOps > 1) {
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow. Key symbols: `initializeNodeKernels`, `empty`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；协调运行时执行状态；处理条件控制流。关键符号：`initializeNodeKernels`, `empty`。

### Lines 184-197
```cpp
            graphExecutors.emplace_back(std::make_unique<ParallelGraphExecutor>(
                *subgraph,
                std::move(executionKernels.nodeKernels),
                executorConfig));
          } else {
            graphExecutors.emplace_back(
                std::make_unique<torch::nativert::SerialGraphExecutor>(
                    *subgraph,
                    std::move(executionKernels.nodeKernels),
                    executorConfig));
          }
        }
      }
      if (node.target() == "torch.ops.higher_order.run_const_graph") {
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow. Key symbols: `emplace_back`, `move`, `target`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流。关键符号：`emplace_back`, `move`, `target`。

### Lines 198-205
```cpp
        constFoldingExecutions.push_back(
            ConstFoldingExecution{std::move(graphExecutors[0])});
      }
      nodeKernels.push_back(std::make_unique<HigherOrderKernel>(
          &node, std::move(graphExecutors)));
    } else if (c10::starts_with(node.target(), "torch.ops")) {
      nodeKernels.push_back(std::make_unique<C10Kernel>(&node));

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow. Key symbols: `push_back`, `move`, `starts_with`, `target`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；处理条件控制流。关键符号：`push_back`, `move`, `starts_with`, `target`。

### Lines 206-216
```cpp
      std::string opName = std::string(node.target());
      if (opsWithoutStaticDispatchCount.find(opName) ==
          opsWithoutStaticDispatchCount.end()) {
        opsWithoutStaticDispatchCount[opName] = 0;
      }
      opsWithoutStaticDispatchCount[opName] += 1;
    } else {
      TORCH_CHECK(false, "Unsupported operator: ", target);
    }
  }

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; handles conditional control flow. Key symbols: `string`, `target`, `find`, `end`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；处理条件控制流。关键符号：`string`, `target`, `find`, `end`。

### Lines 217-226
```cpp
  if (executorConfig.enableStaticCPUKernels &&
      !opsWithoutStaticDispatchCount.empty()) {
    std::stringstream ss;
    for (const auto& [op, count] : opsWithoutStaticDispatchCount) {
      ss << op << ": " << count << ", \n";
    }
    LOG(WARNING) << "Following ops are missing static dispatched kernels: \n"
                 << ss.str();
  }

```
- EN: This block coordinates runtime execution state; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: `empty`, `str`.
- CN: 该代码块协调运行时执行状态；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：`empty`, `str`。

### Lines 227-232
```cpp
  return {
      std::move(nodeKernels),
      std::move(delegateExecutors),
      std::move(constFoldingExecutions)};
}
} // namespace torch::nativert
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `move`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`move`。


## Key Concepts / 关键概念
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/string_view.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/OpKernel.h`, `torch/nativert/executor/ParallelGraphExecutor.h`, `torch/nativert/executor/SerialGraphExecutor.h`, `torch/nativert/graph/Graph.h`, `torch/nativert/kernels/AutoFunctionalizeKernel.h`, `torch/nativert/kernels/C10Kernel.h`, `torch/nativert/kernels/CallTorchBindKernel.h`, `torch/nativert/kernels/HigherOrderKernel.h`, `...`
- External includes / 外部头文件: `string_view`, `fmt/ranges.h`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `KernelFactoryRegistry`, `getKernelFactoryRegistry`, `registerHandler`, `withLock`, `find`, `end`, `emplace`, `move`, `isHandlerRegistered`, `initializeNodeKernels`, `...`
