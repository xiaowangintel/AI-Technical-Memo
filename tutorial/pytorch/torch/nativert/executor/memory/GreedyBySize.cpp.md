# GreedyBySize.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `torch/nativert/executor/memory/GreedyBySize.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements Native Runtime executor behavior for GreedyBySize, including execution, delegation, or low-level utilities.
- 用途 (CN): 实现 Native Runtime 中 executor 子模块里与 GreedyBySize 相关的行为，包括执行、delegate 协调或底层工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```cpp
#include <iomanip>
#include <limits>
#include <optional>

#include <c10/util/Enumerate.h>
```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/Enumerate.h`; external includes: `iomanip`, `limits`, `optional`.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/Enumerate.h`；外部依赖：`iomanip`, `limits`, `optional`。

### Lines 6-10
```cpp
#include <c10/util/Logging.h>
#include <c10/util/irange.h>

#include <torch/nativert/executor/memory/GreedyBySize.h>

```
- EN: Pulls in compile-time configuration or headers required here. Internal includes: `c10/util/Logging.h`, `c10/util/irange.h`, `torch/nativert/executor/memory/GreedyBySize.h`; external includes: none.
- CN: 引入此处所需的编译期开关或头文件。内部依赖：`c10/util/Logging.h`, `c10/util/irange.h`, `torch/nativert/executor/memory/GreedyBySize.h`；外部依赖：无。

### Lines 11-17
```cpp
namespace {

using namespace torch::nativert;

// we need to track the original order in which allocations were made
// since they will be re-sorted between iterations
struct GreedyAllocation : public Allocation {
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `namespace`, `GreedyAllocation`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`namespace`, `GreedyAllocation`。

### Lines 18-27
```cpp
  explicit GreedyAllocation(
      Allocation allocation,
      size_t allocation_idx,
      size_t input_spec_idx)
      : Allocation(allocation),
        allocation_index(allocation_idx),
        input_spec_index(input_spec_idx) {}
  // we need to maintain the allocation ordering s.t., we can look up
  // previous allocations directly from descending_allocation_specs_
  // even after allocations has been re-sorted, which happens after
```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `GreedyAllocation`, `Allocation`, `allocation_index`, `input_spec_index`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`GreedyAllocation`, `Allocation`, `allocation_index`, `input_spec_index`。

### Lines 28-37
```cpp
  // each allocation is complete.
  //
  // i.e., this index represents the index of the spec that was used
  // to create this allocation inside descending_allocation_specs_
  // AFTER the sorting was completed.
  size_t allocation_index{0};
  // index of the spec associated with this allocation
  // in the event that the specs get re-ordered
  // in the process of creating allocations
  // e.g.,
```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 38-47
```cpp
  //              allocation_specs[sX, sY, sZ]
  //                                ^   ^   ^
  //                        values[vX, vY, vZ]
  //
  // means that an allocation created from sY
  // will have an input_spec_index of 1
  //
  // this allows us to return to the original
  // ordering before returning the allocations
  size_t input_spec_index{0};
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 48-53
```cpp
};

struct AllocationSpecWithIndex {
  const AllocationSpec* spec;
  size_t index;
};
```
- EN: Declares C++ types or containers needed in this file. Representative symbols: `AllocationSpecWithIndex`.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：`AllocationSpecWithIndex`。

### Lines 54-61
```cpp

// associate specs with their original (unsorted) index
// and then sort them in descending order by byte size
std::vector<AllocationSpecWithIndex> prepare_allocation_specs(
    const std::vector<AllocationSpec>& allocation_specs) {
  std::vector<AllocationSpecWithIndex> specs;
  specs.reserve(allocation_specs.size());

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `prepare_allocation_specs`, `reserve`, `size`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`prepare_allocation_specs`, `reserve`, `size`。

### Lines 62-67
```cpp
  for (const auto i : c10::irange(allocation_specs.size())) {
    specs.push_back({&allocation_specs[i], i});
  }

  std::sort(specs.begin(), specs.end(), [](auto& lhs, auto& rhs) {
    return lhs.spec->size > rhs.spec->size;
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: `irange`, `size`, `push_back`, `sort`, `begin`, `end`.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：`irange`, `size`, `push_back`, `sort`, `begin`, `end`。

### Lines 68-72
```cpp
  });

  return specs;
}

```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 73-81
```cpp
} // namespace

namespace torch::nativert {

// https://arxiv.org/pdf/2001.03288
LayoutPlan GreedyBySizeAllocationPlanner(
    const std::vector<AllocationSpec>& allocation_specs) {
  LayoutPlan plan;

```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state. Key symbols: `GreedyBySizeAllocationPlanner`.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态。关键符号：`GreedyBySizeAllocationPlanner`。

### Lines 82-86
```cpp
  auto descending_allocation_specs = prepare_allocation_specs(allocation_specs);

  std::vector<GreedyAllocation> allocations;
  allocations.reserve(allocation_specs.size());

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `prepare_allocation_specs`, `reserve`, `size`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`prepare_allocation_specs`, `reserve`, `size`。

### Lines 87-91
```cpp
  auto get_next_offset = [&](const AllocationSpec& spec) -> size_t {
    size_t prev_offset = 0;
    std::optional<size_t> best_offset = std::nullopt;
    size_t smallest_gap = std::numeric_limits<size_t>::max();

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `max`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`max`。

### Lines 92-98
```cpp
    for (const auto& alloc : allocations) {
      if (auto* allocated_spec =
              descending_allocation_specs.at(alloc.allocation_index).spec;
          allocated_spec->not_overlapping_with(spec)) {
        continue;
      }

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: `not_overlapping_with`.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：`not_overlapping_with`。

### Lines 99-106
```cpp
      if (alloc.offset > prev_offset) {
        if (size_t gap = alloc.offset - prev_offset;
            gap >= spec.size && gap < smallest_gap) {
          smallest_gap = gap;
          best_offset = prev_offset;
        }
      }

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 107-111
```cpp
      prev_offset = std::max(prev_offset, alloc.offset + alloc.size);
    }

    return best_offset.value_or(prev_offset);
  };
```
- EN: This block returns results to callers or downstream stages. Key symbols: `max`, `value_or`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`max`, `value_or`。

### Lines 112-117
```cpp

  size_t total_allocation_size = 0;
  for (const auto&& [allocation_index, spec_with_original_index] :
       c10::enumerate(descending_allocation_specs)) {
    auto& spec = spec_with_original_index.spec;

```
- EN: This block iterates over collections or execution units. Key symbols: `enumerate`.
- CN: 该代码块遍历集合或执行单元。关键符号：`enumerate`。

### Lines 118-122
```cpp
    auto new_allocation = GreedyAllocation(
        Allocation{spec->size, get_next_offset(*spec)},
        allocation_index,
        spec_with_original_index.index);

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `GreedyAllocation`, `get_next_offset`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`GreedyAllocation`, `get_next_offset`。

### Lines 123-130
```cpp
    total_allocation_size += new_allocation.size;
    plan.total_size =
        std::max(plan.total_size, new_allocation.offset + new_allocation.size);

    VLOG(1) << "allocation with interval " << spec->lifetime.start << "-->"
            << spec->lifetime.end << " placed at offset "
            << new_allocation.offset;

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `max`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`max`。

### Lines 131-135
```cpp
    // insert new allocation while maintaining relative-offset ordering
    // the algorithm is already quadratic because of get_next_offset
    // so this is negligible

    auto it = std::lower_bound(
```
- EN: This block bridges to backend-specific execution artifacts; iterates over collections or execution units. Key symbols: `lower_bound`.
- CN: 该代码块桥接到特定后端的执行产物；遍历集合或执行单元。关键符号：`lower_bound`。

### Lines 136-142
```cpp
        allocations.begin(),
        allocations.end(),
        new_allocation,
        [](auto& lhs, auto& rhs) { return lhs.offset < rhs.offset; });
    allocations.insert(it, new_allocation);
  }

```
- EN: This block returns results to callers or downstream stages. Key symbols: `begin`, `end`, `insert`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`begin`, `end`, `insert`。

### Lines 143-147
```cpp
  // sort allocations so their ordering is consistent with the input specs
  std::sort(allocations.begin(), allocations.end(), [](auto& lhs, auto& rhs) {
    return lhs.input_spec_index < rhs.input_spec_index;
  });

```
- EN: This block returns results to callers or downstream stages. Key symbols: `sort`, `begin`, `end`.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：`sort`, `begin`, `end`。

### Lines 148-153
```cpp
  plan.allocations.reserve(allocations.size());
  std::move(
      allocations.begin(),
      allocations.end(),
      std::back_inserter(plan.allocations));

```
- EN: This block implements local helper logic for GreedyBySize. Key symbols: `reserve`, `size`, `move`, `begin`, `end`, `back_inserter`.
- CN: 该代码块实现与 GreedyBySize 相关的局部辅助逻辑。关键符号：`reserve`, `size`, `move`, `begin`, `end`, `back_inserter`。

### Lines 154-162
```cpp
  if (plan.total_size > 0) {
    VLOG(1) << std::fixed << std::setprecision(2)
            << "greedy-by-size bytes saved over strictly increasing: "
            << (1.0 - ((float)plan.total_size / (float)total_allocation_size)) *
            100
            << "% (" << total_allocation_size << " - " << plan.total_size
            << " = " << (total_allocation_size - plan.total_size) << " bytes)";
  }

```
- EN: This block handles conditional control flow. Key symbols: `setprecision`.
- CN: 该代码块处理条件控制流。关键符号：`setprecision`。

### Lines 163-166
```cpp
  return plan;
}

} // namespace torch::nativert
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Runtime composition — the file participates in assembling native runtime data flow, execution state, or backend handoff.
  CN: 运行时组合——该文件参与组织原生运行时的数据流、执行状态或后端交接。
- EN: Header-level integration — the translation unit depends on nearby runtime and framework declarations.
  CN: 头文件级集成——该翻译单元依赖周边运行时与框架声明。

## Dependencies / 依赖关系
- Internal includes / 内部头文件: `c10/util/Enumerate.h`, `c10/util/Logging.h`, `c10/util/irange.h`, `torch/nativert/executor/memory/GreedyBySize.h`
- External includes / 外部头文件: `iomanip`, `limits`, `optional`
- Namespaces / 命名空间: `torch::nativert`
- Representative symbols / 代表性符号: `namespace`, `GreedyAllocation`, `Allocation`, `allocation_index`, `input_spec_index`, `AllocationSpecWithIndex`, `prepare_allocation_specs`, `reserve`, `size`, `irange`, `...`
