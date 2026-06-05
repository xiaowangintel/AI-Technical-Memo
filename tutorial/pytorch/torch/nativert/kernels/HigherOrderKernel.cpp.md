# HigherOrderKernel.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/kernels/HigherOrderKernel.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime runtime behavior for HigherOrderKernel, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 runtime 子模块里与 HigherOrderKernel 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <torch/nativert/kernels/HigherOrderKernel.h>

#include <c10/util/Exception.h>
#include <c10/util/string_view.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/kernels/HigherOrderKernel.h`, `c10/util/Exception.h`, `c10/util/string_view.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/kernels/HigherOrderKernel.h`, `c10/util/Exception.h`, `c10/util/string_view.h`；外部依赖：无。

### Lines 6-12
```cpp
namespace torch::nativert {

HigherOrderKernel::HigherOrderKernel(
    const Node* node,
    std::vector<std::unique_ptr<GraphExecutorBase>> graphExecutors)
    : OpKernel(node), graphExecutors_(std::move(graphExecutors)) {
  static constexpr std::string_view prefix = "torch.ops.higher_order.";
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `HigherOrderKernel`, `OpKernel`, `graphExecutors_`, `move`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`HigherOrderKernel`, `OpKernel`, `graphExecutors_`, `move`。

### Lines 13-22
```cpp
  TORCH_CHECK(c10::starts_with(node->target(), prefix));
  auto opName = node->target().substr(prefix.size());
  if (opName == "cond") {
    opType_ = OpType::COND;
    // Checking torch.cond schema is as expected:
    // torch.cond(Tensor predicate, Graph graph1, Graph graph2, Tensor[] args)
    // -> Tensor[]
    TORCH_CHECK(node_->attributes().size() == 2);
    TORCH_CHECK(node_->inputs().size() == 2);
  } else if (opName == "while_loop") {
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow. Key symbols: `starts_with`, `target`, `substr`, `size`, `attributes`, `inputs`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；处理条件控制流。关键符号：`starts_with`, `target`, `substr`, `size`, `attributes`, `inputs`。

### Lines 23-32
```cpp
    opType_ = OpType::WHILE_LOOP;
    // Checking torch.while_loop schema is as expected:
    // torch.while_loop(Graph cond, Graph body, Tensor[] args, Tensor[]
    // additional) -> Tensor[]
    TORCH_CHECK(node_->attributes().size() == 2);
    TORCH_CHECK(node_->inputs().size() == 2);
  } else if (opName == "run_const_graph") {
    opType_ = OpType::RUN_CONST_GRAPH;
    // Checking torch.run_const_graph schema is as expected:
    // torch.run_const_graph(Graph graph, Tensor[] args) -> Tensor[]
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow. Key symbols: `attributes`, `size`, `inputs`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；处理条件控制流。关键符号：`attributes`, `size`, `inputs`。

### Lines 33-39
```cpp
    TORCH_CHECK(!node_->attributes().empty());
    TORCH_CHECK(node_->inputs().size() == 1);
  } else {
    TORCH_CHECK(false, "Unknown higher order op: ", opName);
  }
}

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures. Key symbols: `attributes`, `empty`, `inputs`, `size`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构。关键符号：`attributes`, `empty`, `inputs`, `size`。

### Lines 40-47
```cpp
void HigherOrderKernel::computeInternal(ExecutionFrame& executionFrame) const {
  switch (opType_) {
    case OpType::COND: {
      auto inputs = executionFrame.getIValue(node_->inputs()[1].value->id())
                        .toList()
                        .vec();
      std::vector<c10::IValue> outputs;
      auto cond = executionFrame.getIValue(node_->inputs()[0].value->id());
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `computeInternal`, `getIValue`, `inputs`, `id`, `toList`, `vec`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`computeInternal`, `getIValue`, `inputs`, `id`, `toList`, `vec`。

### Lines 48-57
```cpp
      size_t branchIdx = 0;
      if (cond.isTensor()) {
        branchIdx = cond.toTensor().item().toBool() ? 0 : 1;
      } else if (cond.isBool()) {
        branchIdx = cond.toBool() ? 0 : 1;
      } else {
        TORCH_CHECK(false, "Unsupported type for cond predicate");
      }
      ExecutionFrame branchFrame(*std::get<std::unique_ptr<Graph>>(
          node_->attributes()[branchIdx].value));
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; manipulates graph-like program structures; coordinates runtime execution state; handles conditional control flow; iterates over collections or execution units. Key symbols: `isTensor`, `toTensor`, `item`, `toBool`, `isBool`, `branchFrame`, `...`.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；操作图状程序结构；协调运行时执行状态；处理条件控制流；遍历集合或执行单元。关键符号：`isTensor`, `toTensor`, `item`, `toBool`, `isBool`, `branchFrame`, `...`。

### Lines 58-66
```cpp
      auto ret =
          graphExecutors_[branchIdx]->execute(branchFrame, std::move(inputs));
      for (size_t i = 0; i < ret.size(); i++) {
        executionFrame.setIValue(node_->outputs()[i]->id(), std::move(ret[i]));
      }
      break;
    }
    case OpType::WHILE_LOOP: {
      auto carriedVals =
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `execute`, `move`, `size`, `setIValue`, `outputs`, `id`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；遍历集合或执行单元。关键符号：`execute`, `move`, `size`, `setIValue`, `outputs`, `id`。

### Lines 67-76
```cpp
          executionFrame.getIValue(node_->inputs()[0].value->id())
              .toList()
              .vec();
      auto additonalVals =
          executionFrame.getIValue(node_->inputs()[1].value->id())
              .toList()
              .vec();
      size_t numCarriedVals = carriedVals.size();
      ExecutionFrame condFrame(
          *std::get<std::unique_ptr<Graph>>(node_->attributes()[0].value));
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `getIValue`, `inputs`, `id`, `toList`, `vec`, `size`, `...`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`getIValue`, `inputs`, `id`, `toList`, `vec`, `size`, `...`。

### Lines 77-82
```cpp
      ExecutionFrame bodyFrame(
          *std::get<std::unique_ptr<Graph>>(node_->attributes()[1].value));
      while (true) {
        auto inputs = carriedVals;
        inputs.insert(inputs.end(), additonalVals.begin(), additonalVals.end());
        auto cond = graphExecutors_[0]->execute(condFrame, inputs);
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `bodyFrame`, `attributes`, `insert`, `end`, `begin`, `execute`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；遍历集合或执行单元。关键符号：`bodyFrame`, `attributes`, `insert`, `end`, `begin`, `execute`。

### Lines 83-87
```cpp

        if (cond.at(0).isTensor() && !cond[0].toTensor().item().toBool()) {
          break;
        }
        if (cond.at(0).isBool() && !cond[0].toBool()) {
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `isTensor`, `toTensor`, `item`, `toBool`, `isBool`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`isTensor`, `toTensor`, `item`, `toBool`, `isBool`。

### Lines 88-94
```cpp
          break;
        }
        auto out = graphExecutors_[1]->execute(bodyFrame, std::move(inputs));
        TORCH_CHECK(out.size() == numCarriedVals);
        carriedVals = std::move(out);
      }
      for (size_t i = 0; i < carriedVals.size(); i++) {
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units. Key symbols: `execute`, `move`, `size`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；协调运行时执行状态；遍历集合或执行单元。关键符号：`execute`, `move`, `size`。

### Lines 95-104
```cpp
        executionFrame.setIValue(
            node_->outputs()[i]->id(), std::move(carriedVals[i]));
      }
      break;
    }
    case OpType::RUN_CONST_GRAPH: {
      // run_const_graph op is a special case of higher order op which has
      // been executed during weights loading, therefore at runtime we can
      // just make this a no-op.
      break;
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `setIValue`, `outputs`, `id`, `move`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态。关键符号：`setIValue`, `outputs`, `id`, `move`。

### Lines 105-110
```cpp
    }
    default:
      TORCH_CHECK(false, "Unknown higher order op");
  }
}

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 111-111
```cpp
} // namespace torch::nativert
```
- EN: This block implements local helper logic for HigherOrderKernel. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 HigherOrderKernel 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `torch/nativert/kernels/HigherOrderKernel.h`, `c10/util/Exception.h`, `c10/util/string_view.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `HigherOrderKernel`, `OpKernel`, `graphExecutors_`, `move`, `starts_with`, `target`, `substr`, `size`, `attributes`, `inputs`, `...`
