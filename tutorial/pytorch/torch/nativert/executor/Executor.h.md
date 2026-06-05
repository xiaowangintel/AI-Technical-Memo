# Executor.h — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/executor/Executor.h`
- Repository: `pytorch/pytorch`
- Purpose (EN): Declares Native Runtime executor interfaces, helper types, and contracts for Executor.
- 用途 (CN): 声明 Native Runtime 中 executor 子模块里与 Executor 相关的接口、辅助类型和约定。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#pragma once

#include <atomic>
#include <memory>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: none; external includes: `atomic`, `memory`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：无；外部依赖：`atomic`, `memory`。

### Lines 6-10
```cpp
#include <c10/util/FbcodeMaps.h>
#include <c10/util/Logging.h>
#include <c10/util/Semaphore.h>
#include <c10/util/Synchronized.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/FbcodeMaps.h`, `c10/util/Logging.h`, `c10/util/Semaphore.h`, `c10/util/Synchronized.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/FbcodeMaps.h`, `c10/util/Logging.h`, `c10/util/Semaphore.h`, `c10/util/Synchronized.h`；外部依赖：无。

### Lines 11-15
```cpp
#include <torch/nativert/detail/ITree.h>
#include <torch/nativert/detail/MPMCQueue.h>
#include <torch/nativert/executor/ConstantFolder.h>
#include <torch/nativert/executor/DelegateExecutor.h>
#include <torch/nativert/executor/ExecutionPlanner.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/detail/ITree.h`, `torch/nativert/detail/MPMCQueue.h`, `torch/nativert/executor/ConstantFolder.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/ExecutionPlanner.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/detail/ITree.h`, `torch/nativert/detail/MPMCQueue.h`, `torch/nativert/executor/ConstantFolder.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/ExecutionPlanner.h`；外部依赖：无。

### Lines 16-20
```cpp
#include <torch/nativert/executor/ExecutorConfig.h>
#include <torch/nativert/executor/GraphExecutorBase.h>
#include <torch/nativert/executor/memory/FunctionSchema.h>
#include <torch/nativert/executor/memory/LayoutPlanner.h>
#include <torch/nativert/graph/Graph.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/executor/ExecutorConfig.h`, `torch/nativert/executor/GraphExecutorBase.h`, `torch/nativert/executor/memory/FunctionSchema.h`, `torch/nativert/executor/memory/LayoutPlanner.h`, `torch/nativert/graph/Graph.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/executor/ExecutorConfig.h`, `torch/nativert/executor/GraphExecutorBase.h`, `torch/nativert/executor/memory/FunctionSchema.h`, `torch/nativert/executor/memory/LayoutPlanner.h`, `torch/nativert/graph/Graph.h`；外部依赖：无。

### Lines 21-25
```cpp
#include <torch/nativert/graph/GraphSignature.h>
#include <torch/nativert/kernels/KernelFactory.h>

namespace torch::nativert {

```
- EN: This block manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构。关键符号：无明显局部符号。

### Lines 26-34
```cpp
using namespace torch::nativert::detail;

struct DistributedRunConfig;

/**
 * A very dumb executor. Basically just runs each node in order and contains a
 * giant unordered map for every intermediate, no optimizations applied.
 */
class Executor {
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `namespace`, `DistributedRunConfig`, `Executor`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`namespace`, `DistributedRunConfig`, `Executor`。

### Lines 35-43
```cpp
  class ExecutorFrameDeleter {
   public:
    explicit ExecutorFrameDeleter(Executor& e) : e_(&e) {}
    ExecutorFrameDeleter(ExecutorFrameDeleter&&) = default;
    ExecutorFrameDeleter& operator=(ExecutorFrameDeleter&&) = default;
    ExecutorFrameDeleter(const ExecutorFrameDeleter&) = default;
    ExecutorFrameDeleter& operator=(const ExecutorFrameDeleter&) = default;
    ~ExecutorFrameDeleter() = default;

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExecutorFrameDeleter`, `e_`, `~ExecutorFrameDeleter`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExecutorFrameDeleter`, `e_`, `~ExecutorFrameDeleter`。

### Lines 44-50
```cpp
    void operator()(ExecutionFrame* p) {
      e_->returnExecutorFrameToPool(std::unique_ptr<ExecutionFrame>(p));
    }

   private:
    Executor* e_;
  };
```
- EN: This block coordinates runtime execution state. Key symbols: `returnExecutorFrameToPool`.
- CN: 该代码块协调运行时执行状态。关键符号：`returnExecutorFrameToPool`。

### Lines 51-60
```cpp
  class ExecutorFramePtr {
   public:
    ExecutorFramePtr(std::unique_ptr<ExecutionFrame> ptr, Executor& e)
        : ptr_(std::unique_ptr<ExecutionFrame, ExecutorFrameDeleter>(
              ptr.release(),
              ExecutorFrameDeleter{e})) {}
    ExecutorFramePtr() = delete;
    ExecutorFramePtr(ExecutorFramePtr&&) = default;
    ExecutorFramePtr& operator=(ExecutorFramePtr&&) = default;
    ExecutorFramePtr(const ExecutorFramePtr&) = delete;
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExecutorFramePtr`, `ptr_`, `release`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExecutorFramePtr`, `ptr_`, `release`。

### Lines 61-65
```cpp
    ExecutorFramePtr& operator=(const ExecutorFramePtr&) = delete;
    ~ExecutorFramePtr() = default;

    ExecutionFrame& operator*() {
      return *ptr_;
```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `~ExecutorFramePtr`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`~ExecutorFramePtr`。

### Lines 66-71
```cpp
    }

    ExecutionFrame* operator->() {
      return ptr_.get();
    }

```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `get`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`get`。

### Lines 72-81
```cpp
   private:
    std::unique_ptr<ExecutionFrame, ExecutorFrameDeleter> ptr_;
  };

 public:
  // Constructor used for Inference Path
  Executor(
      torch::nativert::ExecutorConfig executorConfig,
      std::shared_ptr<Graph> graph,
      const std::shared_ptr<Weights>& weights,
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `Executor`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；遍历集合或执行单元。关键符号：`Executor`。

### Lines 82-88
```cpp
      const std::shared_ptr<caffe2::serialize::PyTorchStreamReader>&
          pytorchStreamReader = nullptr);

  std::shared_ptr<Weights> getWeights() {
    std::shared_ptr<Weights> ret;
    weights_.withLock([&](auto& w) { ret = w; });
    return ret;
```
- EN: This block returns results to callers or downstream stages; protects shared state or ordering guarantees. Key symbols: `getWeights`, `withLock`.
- CN: 该代码块向调用方或后续阶段返回结果；保护共享状态或执行顺序保证。关键符号：`getWeights`, `withLock`。

### Lines 89-93
```cpp
  }

  void processWeights(const std::shared_ptr<Weights>& weights);
  void atomicSwapWeights(std::shared_ptr<Weights> weights);

```
- EN: This block protects shared state or ordering guarantees. Key symbols: `processWeights`, `atomicSwapWeights`.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：`processWeights`, `atomicSwapWeights`。

### Lines 94-99
```cpp
  // This API only returns the flattened UserOutputs,
  // intended to be used for Inference path
  // TODO Investigate whether we should remove this, still seems
  //      useful for testing.
  std::vector<c10::IValue> execute(std::vector<c10::IValue> inputs);

```
- EN: This block iterates over collections or execution units. Key symbols: `execute`.
- CN: 该代码块遍历集合或执行单元。关键符号：`execute`。

### Lines 100-104
```cpp
  std::vector<c10::IValue> execute(
      const std::vector<c10::IValue>& args,
      const std::unordered_map<std::string, c10::IValue>& kwargs,
      const ITreeSpec& inputTreeSpec);

```
- EN: This block implements local helper logic for Executor. Key symbols: `execute`.
- CN: 该代码块实现与 Executor 相关的局部辅助逻辑。关键符号：`execute`。

### Lines 105-109
```cpp
  ProfileMetrics benchmarkIndividualNodes(
      const std::vector<std::vector<c10::IValue>>& inputsList,
      const uint32_t warmupRuns,
      const uint32_t mainRuns);

```
- EN: This block manipulates graph-like program structures. Key symbols: `benchmarkIndividualNodes`.
- CN: 该代码块操作图状程序结构。关键符号：`benchmarkIndividualNodes`。

### Lines 110-114
```cpp
  const torch::nativert::GraphSignature& graphSignature() const {
    return graph_->signature();
  }

  static std::string className() {
```
- EN: This block manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: `graphSignature`, `signature`, `className`.
- CN: 该代码块操作图状程序结构；向调用方或后续阶段返回结果。关键符号：`graphSignature`, `signature`, `className`。

### Lines 115-119
```cpp
    return "Executor.v0";
  }

  const torch::nativert::ExecutorConfig& executorConfig() const {
    return executorConfig_;
```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `executorConfig`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`executorConfig`。

### Lines 120-125
```cpp
  }

  std::vector<DelegateExecutor*> getDelegates();

  // Get the number of execution frames in the pool
  auto getNumExecutionFrames() const {
```
- EN: This block coordinates runtime execution state. Key symbols: `getDelegates`, `getNumExecutionFrames`.
- CN: 该代码块协调运行时执行状态。关键符号：`getDelegates`, `getNumExecutionFrames`。

### Lines 126-131
```cpp
    return numExecutionFrames_.load();
  }

  static c10::FastMap<std::string /* target */, torch::nativert::FunctionSchema>
  getKernelSchemas(const std::vector<std::unique_ptr<OpKernel>>& kernels);

```
- EN: This block coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `load`, `getKernelSchemas`.
- CN: 该代码块协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`load`, `getKernelSchemas`。

### Lines 132-136
```cpp
 protected:
  torch::nativert::ExecutorConfig executorConfig_;

  std::shared_ptr<Graph> graph_;

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：无明显局部符号。

### Lines 137-144
```cpp
  // manages the parameters, buffers and tensor constants
  c10::Synchronized<std::shared_ptr<Weights>> weights_;

  void initialize(
      const std::shared_ptr<Weights>& weights,
      const std::shared_ptr<caffe2::serialize::PyTorchStreamReader>&
          pytorchStreamReader);

```
- EN: This block handles tensor metadata or sample values. Key symbols: `initialize`.
- CN: 该代码块处理张量元数据或示例值。关键符号：`initialize`。

### Lines 145-149
```cpp
  ExecutorFramePtr getExecutorFrameFromPool();
  void returnExecutorFrameToPool(std::unique_ptr<ExecutionFrame> frame);

  // Clears stale execution frames from the pool
  void clearStaleExecutionFrames();
```
- EN: This block coordinates runtime execution state. Key symbols: `getExecutorFrameFromPool`, `returnExecutorFrameToPool`, `clearStaleExecutionFrames`.
- CN: 该代码块协调运行时执行状态。关键符号：`getExecutorFrameFromPool`, `returnExecutorFrameToPool`, `clearStaleExecutionFrames`。

### Lines 150-154
```cpp

 private:
  void maybeRunConstantFolding(const std::shared_ptr<Weights>& weights);
  void validateInputs(const std::vector<c10::IValue>& inputs) const;

```
- EN: This block checks invariants or expected outcomes. Key symbols: `maybeRunConstantFolding`, `validateInputs`.
- CN: 该代码块检查不变量或预期结果。关键符号：`maybeRunConstantFolding`, `validateInputs`。

### Lines 155-159
```cpp
  // Helper method to get current timestamp in seconds
  int64_t getCurrentTimestampSeconds() const;

  void initWeights(const std::shared_ptr<Weights>& weights);

```
- EN: This block implements local helper logic for Executor. Key symbols: `getCurrentTimestampSeconds`, `initWeights`.
- CN: 该代码块实现与 Executor 相关的局部辅助逻辑。关键符号：`getCurrentTimestampSeconds`, `initWeights`。

### Lines 160-164
```cpp
  std::unique_ptr<GraphExecutorBase> graphExecutor_;

  // NOTE: delegateExecutors_ is used by nodeKernels_ inside graphExecutor_.
  std::vector<std::unique_ptr<DelegateExecutor>> delegateExecutors_;

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：无明显局部符号。

### Lines 165-174
```cpp
  std::vector<ConstFoldingExecution> constFoldingExecutions_;

  std::optional<ConstantFolder> constantFolder_;

  c10::Semaphore sem_;
  torch::nativert::detail::MPMCQueue<std::unique_ptr<ExecutionFrame>>
      executionFrames_;
  torch::nativert::detail::MPMCQueue<std::unique_ptr<ExecutionFrame>>
      inactiveExecutionFrames_;
  std::atomic_int64_t numExecutionFrames_;
```
- EN: This block coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 175-180
```cpp

  std::unique_ptr<LayoutPlanner> layoutPlanner_;
  std::atomic_int64_t lastClearedTimestamp_;
  std::mutex cleanupLock_;
  std::atomic_bool clearingInProgress_{false};
};
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 181-182
```cpp

} // namespace torch::nativert
```
- EN: This block implements local helper logic for Executor. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 Executor 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/FbcodeMaps.h`, `c10/util/Logging.h`, `c10/util/Semaphore.h`, `c10/util/Synchronized.h`, `torch/nativert/detail/ITree.h`, `torch/nativert/detail/MPMCQueue.h`, `torch/nativert/executor/ConstantFolder.h`, `torch/nativert/executor/DelegateExecutor.h`, `torch/nativert/executor/ExecutionPlanner.h`, `torch/nativert/executor/ExecutorConfig.h`, `...`
- External includes / 外部头文件: `atomic`, `memory`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `namespace`, `DistributedRunConfig`, `Executor`, `ExecutorFrameDeleter`, `e_`, `~ExecutorFrameDeleter`, `returnExecutorFrameToPool`, `ExecutorFramePtr`, `ptr_`, `release`, `...`
