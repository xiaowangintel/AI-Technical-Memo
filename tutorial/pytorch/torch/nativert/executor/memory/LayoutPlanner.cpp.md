# LayoutPlanner.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/executor/memory/LayoutPlanner.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime executor behavior for LayoutPlanner, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 executor 子模块里与 LayoutPlanner 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```cpp
#include <torch/nativert/executor/memory/LayoutPlanner.h>

#include <c10/util/Enumerate.h>

#include <torch/nativert/executor/ExecutionPlanner.h>
#include <torch/nativert/executor/memory/AliasAnalyzer.h>
#include <torch/nativert/executor/memory/Bump.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `torch/nativert/executor/memory/LayoutPlanner.h`, `c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`, `torch/nativert/executor/memory/AliasAnalyzer.h`, `torch/nativert/executor/memory/Bump.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`torch/nativert/executor/memory/LayoutPlanner.h`, `c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`, `torch/nativert/executor/memory/AliasAnalyzer.h`, `torch/nativert/executor/memory/Bump.h`；外部依赖：无。

### Lines 8-21
```cpp
#include <torch/nativert/executor/memory/DisjointStorageGroups.h>
#include <torch/nativert/executor/memory/GreedyBySize.h>

namespace torch::nativert {

LayoutPlanner::LayoutPlanner(
    const Graph& graph,
    const c10::FastMap<std::string /* target */, FunctionSchema>& kernelSchemas,
    const std::vector<bool>& persistentValues,
    const torch::nativert::LayoutPlannerSettings& settings)
    : managed_values_(graph.values().size()),
#ifndef NDEBUG
      alias_analyzer_(graph, kernelSchemas),
#endif
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; coordinates runtime execution state; reuses computed state to reduce repeated work. Key symbols: `LayoutPlanner`, `managed_values_`, `values`, `size`, `alias_analyzer_`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；协调运行时执行状态；复用已计算状态以减少重复工作。关键符号：`LayoutPlanner`, `managed_values_`, `values`, `size`, `alias_analyzer_`。

### Lines 22-28
```cpp
      settings_(settings) {
#ifndef NDEBUG
  auto& alias_analyzer = alias_analyzer_;
#else
  auto alias_analyzer = AliasAnalyzer(graph, kernelSchemas);
#endif

```
- EN: This block manipulates graph-like program structures. Key symbols: `settings_`, `AliasAnalyzer`.
- CN: 该代码块操作图状程序结构。关键符号：`settings_`, `AliasAnalyzer`。

### Lines 29-37
```cpp
  auto value_to_allocation_spec = c10::FastMap<const Value*, AllocationSpec>{};

  std::set<const Value*> input_values_set_;
  for (const auto* nv : graph.userInputs()) {
    if (nv->type() == Type::Kind::Tensor) {
      input_values_set_.insert(nv);
    }
  }

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `userInputs`, `type`, `insert`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`userInputs`, `type`, `insert`。

### Lines 38-49
```cpp
  const auto& tensor_meta = graph.tensorValuesMeta();

  for (auto&& [i, node] : at::enumerate(graph.nodes())) {
    // only manage out variant values
    if (const auto schemaIt = kernelSchemas.find(std::string(node.target()));
        schemaIt == kernelSchemas.end() ||
        schemaIt->second.kernel_kind() != OpKernelKind::kStaticDispatchKernel) {
      VLOG(1) << "not able to plan outputs for node " << node.target()
              << " as it is derived from an unsupported kernel kind.";
      continue;
    }

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `tensorValuesMeta`, `enumerate`, `nodes`, `find`, `string`, `target`, `...`.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`tensorValuesMeta`, `enumerate`, `nodes`, `find`, `string`, `target`, `...`。

### Lines 50-58
```cpp
    for (const auto& output : node.outputs()) {
      // don't manage persistent values
      if (bool is_persistent = persistentValues[output->id()]; is_persistent) {
        VLOG(1)
            << "not planning " << output->name()
            << " as it is a persistent value (likely a weight or const-folded)";
        continue;
      }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: `outputs`, `id`, `name`, `value`.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：`outputs`, `id`, `name`, `value`。

### Lines 59-66
```cpp
      // only manage tensors
      if (bool is_tensor = output->type().kind() == Type::Kind::Tensor;
          !is_tensor) {
        VLOG(1) << "not planning " << output->name()
                << " as it is not a raw tensor. type: " << output->type();
        continue;
      }

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `type`, `kind`, `name`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`type`, `kind`, `name`。

### Lines 67-77
```cpp
      // output storage ownership must be given to the caller.
      if (const auto& values_associated_with_output =
              alias_analyzer.values_associated_with_output_storage();
          values_associated_with_output.find(output) !=
          values_associated_with_output.end()) {
        VLOG(1)
            << "not planning " << output->name()
            << " as its underlying storage may be associated with a graph output";
        continue;
      }

```
- EN: This block manipulates graph-like program structures; handles conditional control flow. Key symbols: `values_associated_with_output_storage`, `find`, `end`, `name`.
- CN: 该代码块操作图状程序结构；处理条件控制流。关键符号：`values_associated_with_output_storage`, `find`, `end`, `name`。

### Lines 78-84
```cpp
      // inputs are borrowed -- this is merely a sanity check
      if (input_values_set_.find(output) != input_values_set_.end()) {
        VLOG(1) << "not planning " << output->name()
                << " as it is a graph input that is borrowed from the user";
        continue;
      }

```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; handles conditional control flow. Key symbols: `find`, `end`, `name`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；处理条件控制流。关键符号：`find`, `end`, `name`。

### Lines 85-91
```cpp
      // don't plan aliases -- they don't own the associated dataptr
      if (bool is_alias = alias_analyzer.is_alias(output); is_alias) {
        VLOG(1) << "not planning " << output->name() << " as it is an alias";
        continue;
      }

      if (bool is_not_consumed = output->users().empty(); is_not_consumed) {
```
- EN: This block handles conditional control flow. Key symbols: `is_alias`, `name`, `users`, `empty`.
- CN: 该代码块处理条件控制流。关键符号：`is_alias`, `name`, `users`, `empty`。

### Lines 92-98
```cpp
        VLOG(1) << "not planning " << output->name() << " as it has no users";
        continue;
      }

      if (auto meta_it = tensor_meta.find(std::string(output->name()));
          meta_it != tensor_meta.end()) {
        if (const auto& meta = meta_it->second; meta.device() == c10::kCPU) {
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `name`, `find`, `string`, `end`, `device`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`name`, `find`, `string`, `end`, `device`。

### Lines 99-110
```cpp
          auto& spec = value_to_allocation_spec[output];
          spec.lifetime = alias_analyzer.lifetime(output);
          managed_values_[output->id()] = true;
          continue;
        } else {
          VLOG(1) << "tensor " << output->name()
                  << " not placed on cpu so we cannot plan it";
        }
      } else /* possible if runtime pass didn't populate meta info */ {
        VLOG(1) << "tensor " << output->name() << " has no meta information";
      }

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `lifetime`, `id`, `name`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`lifetime`, `id`, `name`。

### Lines 111-119
```cpp
      managed_values_[output->id()] = true;
      value_to_allocation_spec[output].lifetime =
          alias_analyzer.lifetime(output);
    }
  }

  LOG(INFO) << "layout planner created with " << value_to_allocation_spec.size()
            << " values";

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state. Key symbols: `id`, `lifetime`, `size`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态。关键符号：`id`, `lifetime`, `size`。

### Lines 120-133
```cpp
  switch (settings_.algorithmType()) {
    case torch::nativert::LayoutPlannerAlgorithmType::Bump: {
      algorithm_ = &BumpAllocationPlanner;
      break;
    }
    case torch::nativert::LayoutPlannerAlgorithmType::GreedyBySize: {
      algorithm_ = &GreedyBySizeAllocationPlanner;
      break;
    }
    case LayoutPlannerAlgorithmType::DisjointStorageGroups: {
      algorithm_ = &DisjointStorageGroupsPlanner;
      break;
    }
  }
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state. Key symbols: `algorithmType`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态。关键符号：`algorithmType`。

### Lines 134-140
```cpp

  TORCH_CHECK(algorithm_ != nullptr, "algorithm can't be null");

  initialize_vectors(value_to_allocation_spec);

  auto exec_planner = ExecutionPlanner{graph};
  auto p = exec_planner.createPlan();
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; coordinates runtime execution state. Key symbols: `initialize_vectors`, `createPlan`.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；协调运行时执行状态。关键符号：`initialize_vectors`, `createPlan`。

### Lines 141-149
```cpp
  for (const auto& freeable : p->valuesToFree) {
    for (const auto v : freeable) {
      if (!is_managed(v)) {
        unplanned_values_.push_back(v);
      }
    }
  }
}

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: `is_managed`, `push_back`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：`is_managed`, `push_back`。

### Lines 150-158
```cpp
void LayoutPlanner::initialize_vectors(
    c10::FastMap<const Value*, AllocationSpec> value_to_allocation_spec) {
  size_t num_managed = value_to_allocation_spec.size();

  planned_values_.resize(num_managed);
  planned_allocation_specs_.resize(num_managed);
  planned_values_historical_max_nbytes_ =
      std::vector<std::atomic_size_t>(num_managed);

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; protects shared state or ordering guarantees. Key symbols: `initialize_vectors`, `size`, `resize`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；保护共享状态或执行顺序保证。关键符号：`initialize_vectors`, `size`, `resize`。

### Lines 159-168
```cpp
  size_t i = 0;
  for (auto& [v, spec] : value_to_allocation_spec) {
    TORCH_CHECK(
        spec.lifetime.start <= spec.lifetime.end,
        "lifetime start must be before lifetime end");

    planned_values_[i] = v->id();
    planned_values_historical_max_nbytes_[i] = spec.size;
    planned_allocation_specs_[i] = spec;

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: `id`.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：`id`。

### Lines 169-176
```cpp
    i++;
  }

  // for sanity in case anyone tries to use this after this method
  // is called with a bunch of junk (i.e., moved specs) in it
  value_to_allocation_spec.clear();
}

```
- EN: This block iterates over collections or execution units. Key symbols: `clear`.
- CN: 该代码块遍历集合或执行单元。关键符号：`clear`。

### Lines 177-184
```cpp
const std::vector<ValueId>& LayoutPlanner::get_planned_values() const {
  return planned_values_;
}

const std::vector<ValueId>& LayoutPlanner::get_unplanned_values() const {
  return unplanned_values_;
}

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `get_planned_values`, `get_unplanned_values`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`get_planned_values`, `get_unplanned_values`。

### Lines 185-194
```cpp
void LayoutPlanner::start_worker_if_not_started() {
  c10::call_once(worker_once_flag_, [&]() {
    // make sure plan is populated by the time this
    // returns for the first time :P
    create_plan();
    worker_ =
        std::thread([this]() { run_periodic([this] { create_plan(); }); });
  });
}

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `start_worker_if_not_started`, `call_once`, `create_plan`, `thread`, `run_periodic`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`start_worker_if_not_started`, `call_once`, `create_plan`, `thread`, `run_periodic`。

### Lines 195-201
```cpp
LayoutPlanner::~LayoutPlanner() {
  {
    std::unique_lock<std::mutex> l(mutex_);
    stopped_ = true;
  }
  cv_.notify_one();
  if (worker_.joinable()) {
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; handles conditional control flow; protects shared state or ordering guarantees. Key symbols: `~LayoutPlanner`, `l`, `notify_one`, `joinable`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；处理条件控制流；保护共享状态或执行顺序保证。关键符号：`~LayoutPlanner`, `l`, `notify_one`, `joinable`。

### Lines 202-208
```cpp
    worker_.join();
  }
}

void LayoutPlanner::run_periodic(const std::function<void()>& f) {
  std::unique_lock<std::mutex> l(mutex_);
  while (!cv_.wait_for(
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; iterates over collections or execution units; protects shared state or ordering guarantees. Key symbols: `join`, `run_periodic`, `void`, `l`, `wait_for`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；遍历集合或执行单元；保护共享状态或执行顺序保证。关键符号：`join`, `run_periodic`, `void`, `l`, `wait_for`。

### Lines 209-217
```cpp
      l, settings_.planningInterval(), [&]() { return stopped_; })) {
    f();
  }
}

void LayoutPlanner::create_plan() {
  // update spec sizes to use historical maximums set
  // by execution frames before creating the new plan
  bool updated = false;
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; returns results to callers or downstream stages. Key symbols: `planningInterval`, `f`, `create_plan`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；向调用方或后续阶段返回结果。关键符号：`planningInterval`, `f`, `create_plan`。

### Lines 218-227
```cpp
  for (const auto i : c10::irange(planned_allocation_specs_.size())) {
    auto& spec = planned_allocation_specs_[i];
    if (const auto new_size = planned_values_historical_max_nbytes_[i].load(
            std::memory_order_relaxed);
        new_size > spec.size) {
      spec.size = new_size;
      updated = true;
    }
  }

```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: `irange`, `size`, `load`.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：`irange`, `size`, `load`。

### Lines 228-234
```cpp
  if (updated) {
    plan_.write([p_new = (*algorithm_)(planned_allocation_specs_)](
                    LayoutPlan& plan) { plan = p_new; });
  }
}

} // namespace torch::nativert
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: `write`.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：`write`。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
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
- Internal includes / 内部头文件: `torch/nativert/executor/memory/LayoutPlanner.h`, `c10/util/Enumerate.h`, `torch/nativert/executor/ExecutionPlanner.h`, `torch/nativert/executor/memory/AliasAnalyzer.h`, `torch/nativert/executor/memory/Bump.h`, `torch/nativert/executor/memory/DisjointStorageGroups.h`, `torch/nativert/executor/memory/GreedyBySize.h`
- External includes / 外部头文件: 无
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `LayoutPlanner`, `managed_values_`, `values`, `size`, `alias_analyzer_`, `settings_`, `AliasAnalyzer`, `userInputs`, `type`, `insert`, `...`
