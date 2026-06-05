# ExecutionPlanner.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/executor/ExecutionPlanner.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime executor behavior for ExecutionPlanner, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 executor 子模块里与 ExecutionPlanner 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <unordered_map>

#include <c10/util/Enumerate.h>
#include <torch/nativert/executor/ExecutionPlanner.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`; external includes: `unordered_map`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`；外部依赖：`unordered_map`。

### Lines 6-10
```cpp
namespace torch::nativert {

std::unique_ptr<ExecutionPlan> ExecutionPlanner::createPlan() {
  auto plan = std::make_unique<ExecutionPlan>();

```
- EN: This block coordinates runtime execution state. Key symbols: `createPlan`.
- CN: 该代码块协调运行时执行状态。关键符号：`createPlan`。

### Lines 11-15
```cpp
  // Current implementation assume that nodes will be executed
  // in the same order as the thrift graph.
  // In the future, we can do execution order plan, as long as it's
  // comply with topological order

```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 16-25
```cpp
  generateDeallocationPlan(*plan);
  return plan;
}

/* static */ c10::FastSet<ValueId> ExecutionPlanner::staticValues(
    const Graph& graph) {
  c10::FastSet<ValueId> staticValues;
  // Filter lastUsedBy by graph inputs
  // parameters/buffer values should not be freed
  // It's a policy decision to whether to free user inputs. For now, we don't
```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `generateDeallocationPlan`, `staticValues`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`generateDeallocationPlan`, `staticValues`。

### Lines 26-30
```cpp
  // free user inputs.
  // TODO: It should be fine to "free" the user inputs. If the user holds a ref
  // to it, it won't be deallocated.
  for (const auto* input : graph.inputs()) {
    if (input) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `inputs`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`inputs`。

### Lines 31-35
```cpp
      const auto& id = input->id();
      staticValues.insert(id);
    }
  }

```
- EN: This block implements local helper logic for ExecutionPlanner. Key symbols: `id`, `insert`.
- CN: 该代码块实现与 ExecutionPlanner 相关的局部辅助逻辑。关键符号：`id`, `insert`。

### Lines 36-41
```cpp
  // Filter lastUsedBy by graph outputs, as they are still needed to be returned
  for (const auto& output : graph.outputs()) {
    const auto& id = output->id();
    staticValues.insert(id);
  }

```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `outputs`, `id`, `insert`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`outputs`, `id`, `insert`。

### Lines 42-46
```cpp
  for (const auto& [id, _] : graph.getConstantSymIntValues()) {
    staticValues.insert(id);
  }

  for (const Node& node : graph.nodes()) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `getConstantSymIntValues`, `insert`, `nodes`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`getConstantSymIntValues`, `insert`, `nodes`。

### Lines 47-54
```cpp
    if (node.target() == "torch.ops.higher_order.run_const_graph") {
      for (const auto& output : node.outputs()) {
        // Do not free the outputs of run_const_graph, as they are newly
        // produced folded constants
        staticValues.insert(output->id());
      }
    } else {
      for (const auto& input : node.inputs()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `target`, `outputs`, `insert`, `id`, `inputs`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`target`, `outputs`, `insert`, `id`, `inputs`。

### Lines 55-61
```cpp
        if (input.value->isFolded()) {
          staticValues.insert(input.value->id());
        }
      }
    }
  }

```
- EN: This block handles conditional control flow. Key symbols: `isFolded`, `insert`, `id`.
- CN: 该代码块处理条件控制流。关键符号：`isFolded`, `insert`, `id`。

### Lines 62-68
```cpp
  return staticValues;
}

void ExecutionPlanner::generateDeallocationPlan(ExecutionPlan& plan) {
  const auto& nodes = graph_.nodes();
  size_t numNodes = nodes.size();

```
- EN: This block manipulates graph-like program structures; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `generateDeallocationPlan`, `nodes`, `size`.
- CN: 该代码块操作图状程序结构；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`generateDeallocationPlan`, `nodes`, `size`。

### Lines 69-75
```cpp
  std::unordered_map<ValueId, NodeIndex> lastUsedBy;

  // Traverse from the last node to the first node
  // For each Value, find out which is the last node that uses it
  // the Value can freed after executing the node
  size_t nodeIdx = nodes.size() - 1;
  for (auto it = std::rbegin(nodes); it != std::rend(nodes); it++) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `size`, `rbegin`, `rend`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`size`, `rbegin`, `rend`。

### Lines 76-85
```cpp
    const auto& inputs = it->inputs();
    for (const auto& input : inputs) {
      const auto& id = input.value->id();
      if (lastUsedBy.find(id) == lastUsedBy.end()) {
        lastUsedBy.insert({id, nodeIdx});
      }
    }
    nodeIdx--;
  }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `inputs`, `id`, `find`, `end`, `insert`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`inputs`, `id`, `find`, `end`, `insert`。

### Lines 86-90
```cpp
  std::vector<std::vector<ValueId>> valuesToFree(numNodes);

  const auto& statics = staticValues(graph_);
  for (auto& [id, nodeIndex] : lastUsedBy) {
    if (statics.find(id) == statics.end()) {
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `valuesToFree`, `staticValues`, `find`, `end`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`valuesToFree`, `staticValues`, `find`, `end`。

### Lines 91-96
```cpp
      valuesToFree[nodeIndex].push_back(id);
    }
  }

  plan.valuesToFree = std::move(valuesToFree);

```
- EN: This block manipulates graph-like program structures. Key symbols: `push_back`, `move`.
- CN: 该代码块操作图状程序结构。关键符号：`push_back`, `move`。

### Lines 97-102
```cpp
  // print allocation plan
  VLOG(2) << plan;

  return;
}

```
- EN: This block implements local helper logic for ExecutionPlanner. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ExecutionPlanner 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 103-107
```cpp
std::ostream& operator<<(std::ostream& out, const ExecutionPlan& plan) {
  out << "****** Deallocation Plan ******\n";
  for (auto&& [i, values] : c10::enumerate(plan.valuesToFree)) {
    out << "Node #" << i << ", valuesToFree = [";
    for (const auto& value : values) {
```
- EN: This block manipulates graph-like program structures; iterates over collections or execution units. Key symbols: `enumerate`.
- CN: 该代码块操作图状程序结构；遍历集合或执行单元。关键符号：`enumerate`。

### Lines 108-112
```cpp
      out << value << ", ";
    }
    out << "]\n";
  }
  return out;
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 113-115
```cpp
}

} // namespace torch::nativert
```
- EN: This block implements local helper logic for ExecutionPlanner. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 ExecutionPlanner 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`
- External includes / 外部头文件: `unordered_map`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `createPlan`, `generateDeallocationPlan`, `staticValues`, `inputs`, `id`, `insert`, `outputs`, `getConstantSymIntValues`, `nodes`, `target`, `...`
