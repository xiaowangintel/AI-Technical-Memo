# profiling_record.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/profiling_record.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Captures runtime profiling information used to refine graph optimization and specialization.
- **Purpose (CN)**: 采集运行时 profiling 信息，用于改进图优化和特化。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
#pragma once

#include <ATen/ATen.h>
#include <ATen/core/ivalue.h>
#include <ATen/core/jit_type.h>
#include <ATen/core/stack.h>
#include <torch/csrc/Export.h>
#include <torch/csrc/jit/ir/ir.h>

#include <list>
#include <map>
#include <unordered_map>
#include <vector>
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/Export.h, torch/csrc/jit/ir/ir.h; ATen/c10 facilities such as ATen/ATen.h, ATen/core/ivalue.h, ATen/core/jit_type.h, and 1 more; standard-library headers such as list, map, unordered_map, and 1 more. The preprocessor guard keeps declarations single-instanced when the header is included transitively.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/Export.h、torch/csrc/jit/ir/ir.h；ATen/c10 基础设施，如 ATen/ATen.h、ATen/core/ivalue.h、ATen/core/jit_type.h 等共 4 项；标准库头文件，如 list、map、unordered_map 等共 4 项。 预处理器保护用于避免头文件在传递包含时被重复展开。

### Lines 15-30
```cpp
// We would like to assign each position/axis of a tensor an abstract size
// * For each `tensor` we have a profiled `Value` of a `TensorType` describing
// the properties of the `tensor`.
// * `TensorType` has a property called `symbolic_sizes_` to describe observed
// `tensor.sizes()`
// * `symbolic_sizes_` is a vector of abstract sizes (or
// `std::vector<ShapeSymbol>`) where
//   * `ShapeSymbol`at `symbolic_sizes_[i]`  describes the size value
//   (`Dimension`) at `tensor.sizes()[i]`
// * We may see the same `Dimension` at different positions `i` in
// `tensor.sizes()` or even in different `tensor`
//   * First, we would like associate the same `ShapeSymbol` to the same
//   `Dimension` across **one** profiling execution or run of a TorchScript
//   function.
//     * The same `ShapeSymbol`s in different positions of `symbolic_shapes_` in
//     possibly different `TensorType`s (i.e. `TensorType`s for different
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 31-46
```cpp
//     profiled values) form an implicit set. The elements of such a set are
//     called *dimension locations*.
//     * These sets allow us to track how the shapes of input arguments of some
//     operation relate to operation's output shapes as the input and output
//     shapes might share the same `ShapeSymbol`s
// * For **every** profiling run, we would like to maintain the invariant that
// *the same `ShapeSymbol` is always associated with the same `Dimension`*.
// * To maintain this invariant we merge the profiling information from all
// profiling runs,
//   * For every two runs, we iterate over all `symbic_shapes_`  and compare
//   their `ShapeSymbol`s in the same position.
//     * if we observe that for every dimension location that has
//     the`ShapeSymbol S1`  in run #1 there is **only one** `ShapeSymbol S2` in
//     the same dimension location in run #2, we conclude that the invariant
//     holds.
//     * However, if we observe some dimension locations in run #2 have
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 47-62
```cpp
//     `ShapeSymbol S2` and the other ones have `ShapeSymbol S3` we would like
//     to partition the virtual set of dimension locations associated with
//     `ShapeSymbol S1` into two new subsets, so the invariant holds.
//     * The partitioning works by assigning a new symbol to the dimension
//     locations (associated with `ShapeSymbol S1`) that have `ShapeSymbol S2`
//     and another new symbol to the dimension locations that have `ShapeSymbol
//     S3`. In other words,
//       * Subset #1 will consist of the dimension locations that in run #2 have
//       `ShapeSymbol S2`  and will have `ShapeSymbol S4`  in those dimension
//       locations
//       * Subset #2 will consist of the dimension locations that in run #2 have
//       `ShapeSymbol S4`  and will have `ShapeSymbol S5`  in those dimension
//       locations
//     * The effective result of merging the profiling information from two runs
//     is new `TensorTypes` whose `symbolic_sizes_` /dimension locations have
//     either `ShapeSymbol S4` or `ShapeSymbol S5`.
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 63-77
```cpp
//     * Partitioning can be done even before we have seen all the dimension
//     locations associated with `ShapeSymbol S1`
//       * We use `getSymbolInSet` of `ShapeSymbolTable` to remember all
//       `ShapeSymbols` from run #2 we observed in the dimension locations
//       associated with `ShapeSymbol S1` .
//       * For every `ShapeSymbol` from run #2 in the dimension location
//       associated with `ShapeSymbol S1`  `getSymbolInSet` returns a symbol
//       that we assign to the dimension location in a new TensorType.
//         * It's important to point out that the same `ShapeSymbol S2` from run
//         #2 in two dimension locations that have different `ShapeSymbol`s in
//         run #1 are different! These dimension locations will belong to
//         different subsets and have different `ShapeSymbol`s after merge.
//         * On the other hand, for the same `ShapeSymbol S2` in two dimension
//         locations that have `ShapeSymbol S1` in run #1`getSymbolInSet` will
//         return the same symbol.
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 79-94
```cpp
namespace torch::jit {

using ::c10::TensorTypePtr;
using Dimension = int64_t;

TORCH_API void RegisterProfilingNode(
    const std::function<bool(const Node*)>& /*func*/);

struct ProfilingRecord;

// `SetPartitioningHelper` is used to maintain the following invariant:
// For **every** profiling run, *the same `ShapeSymbol` is always associated
// with the same `Dimension`*.
// while merging the profiling information from multiple runs.
struct SetPartitioningHelper {
  std::map<c10::ShapeSymbol, std::map<Dimension, c10::ShapeSymbol>>
```
- **EN**: The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem. It introduces or extends ProfilingRecord, SetPartitioningHelper, which define the primary data structures or interfaces for this portion of the file. This chunk defines `RegisterProfilingNode`, which registers schemas, operators, or passes with the surrounding runtime.
- **CN**: 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。 它引入或扩展了 ProfilingRecord、SetPartitioningHelper，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `RegisterProfilingNode`，其作用是向周边运行时注册 schema、算子或 pass。

### Lines 95-107
```cpp
      sets2subsets_;

  // `partitionSetByDimension` partitions a virtual set
  // of dimension locations associated with ShapeSymbol `symbol` into subsets.
  // Partitioning is equivalent to giving (or renaming) a particular
  // dimension location a new `ShapeSymbol`.
  // The same `Dimension` value in different dimension locations
  // that used to have `symbol` will receive the same
  // new `ShapeSymbol`, effectively forming a new set.
  c10::ShapeSymbol partitionSetByDimension(
      Dimension new_size,
      c10::ShapeSymbol symbol) {
    auto& dims2symbols = getSetForSymbol(symbol);
```
- **EN**: This chunk defines `partitionSetByDimension`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `partitionSetByDimension`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 109-124
```cpp
    if (dims2symbols.count(new_size) == 0) {
      auto new_sym = c10::ShapeSymbol::newSymbol();
      dims2symbols[new_size] = new_sym;
      return new_sym;
    }

    return dims2symbols[new_size];
  }

 private:
  std::map<Dimension, c10::ShapeSymbol>& getSetForSymbol(c10::ShapeSymbol s) {
    auto& set = sets2subsets_[s];
    // N.B. adding a mapping { s.static_size(), s }
    // makes sure we preserve the fact that
    // some dimension values remain the same
    // across all profiled runs
```
- **EN**: This chunk continues `partitionSetByDimension` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `partitionSetByDimension`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 125-140
```cpp
    if (s.is_static()) {
      set.insert({s.static_size(), s});
    }
    return set;
  }
};

// ShapeSymbolTable is used by Interpreter
// to assign dimension values to ShapeSymbols
// and fail a guard if the same symbol
// is assigned more than one dimension value.
struct ShapeSymbolTable {
  // N.B. we treat static symbols as always assigned
  // to themselves
  bool isBound(c10::ShapeSymbol s) {
    if (s.is_static()) {
```
- **EN**: It introduces or extends ShapeSymbolTable, which define the primary data structures or interfaces for this portion of the file. This chunk defines `isBound`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 ShapeSymbolTable，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `isBound`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 141-156
```cpp
      return true;
    }
    return data_.count(s) != 0;
  }

  // N.B. we treat static symbols as always assigned
  // to themselves
  Dimension getValue(c10::ShapeSymbol s) {
    if (s.is_static()) {
      return s.static_size();
    }
    return data_[s];
  }
  void assign(c10::ShapeSymbol s, Dimension v) {
    TORCH_INTERNAL_ASSERT(!s.is_static());
    data_[s] = v;
```
- **EN**: This chunk defines `assign`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `assign`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 157-171
```cpp
  }
  std::map<c10::ShapeSymbol, Dimension> data_;
  // Tries to assign dimension values from `new_sizes` to
  // `ShapeSymbol`s `sym_shapes`.
  // Returns `true` if every dimension value from `new_sizes`
  // can be assigned to the corresponding `ShapeSymbol` from
  // `sym_shapes`
  // A dimension value can be assigned to a `ShapeSymbol`
  // * if the symbol isn't assigned yet any dimension value
  // * if the symbol is assigned and its value is equal to
  // the dimension value from `new_sizes`
  bool bindSymbolicShapes(
      at::IntArrayRef new_sizes,
      const c10::SymbolicShape& sym_shapes);
};
```
- **EN**: This chunk declares `bindSymbolicShapes`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段声明了 `bindSymbolicShapes`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 173-186
```cpp
struct ProfilingRecord {
  // N.B. ProfilingRecord's copy and move c-tor are disabled, so we won't
  // end up accidentally copying or moving ProfilingRecords whose addresses
  // are captured in callbacks_
  ProfilingRecord(const ProfilingRecord&) = delete;
  ProfilingRecord(ProfilingRecord&&) noexcept = delete;
  TORCH_API static std::unique_ptr<ProfilingRecord> instrumentGraph(
      const std::shared_ptr<Graph>& graph);
  TORCH_API static void removeProfilingNodes(Block* b);
  TORCH_API static void removeProfileCounter(Block* b);

  std::shared_ptr<Graph> profiled_graph_;
  mutable std::mutex mutex_;
  size_t profiling_count_;
```
- **EN**: It introduces or extends ProfilingRecord, which define the primary data structures or interfaces for this portion of the file. This chunk defines `removeProfileCounter`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 它引入或扩展了 ProfilingRecord，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `removeProfileCounter`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 188-203
```cpp
  bool ready() const;

  std::shared_ptr<Graph> graph() const {
    return profiled_graph_;
  }

  TORCH_API ProfileIValueOp* createProfileIValueNode(Value* in_val);
  TORCH_API ProfileIValueOp* createProfileIValueNode(ArrayRef<Value*> inputs);

 private:
  ProfileOp* createProfileNode(
      const std::function<void(Stack&)>& fp,
      at::ArrayRef<Value*> inputs);
  void instrumentBlock(Block* block);
  void insertShapeProfile(Node* n, size_t offset, const TypePtr& input_type);
  ProfilingRecord(std::shared_ptr<Graph> g);
```
- **EN**: This chunk defines `insertShapeProfile`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `insertShapeProfile`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 204-206
```cpp
};

} // namespace torch::jit
```
- **EN**: This chunk continues `insertShapeProfile` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `insertShapeProfile`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **ProfilingRecord**
  - EN: `ProfilingRecord` is a central symbol declared or implemented in this file.
  - CN: `ProfilingRecord` 是本文件声明或实现的核心符号。
- **SetPartitioningHelper**
  - EN: `SetPartitioningHelper` is a central symbol declared or implemented in this file.
  - CN: `SetPartitioningHelper` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Runtime profiling**
  - EN: Captures observed types or shapes so later passes can specialize execution.
  - CN: 捕获观测到的类型或形状，以便后续 pass 进行特化。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/Export.h`, `torch/csrc/jit/ir/ir.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/ATen.h`, `ATen/core/ivalue.h`, `ATen/core/jit_type.h`, `ATen/core/stack.h`
- **Standard library / 标准库**: `list`, `map`, `unordered_map`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `ProfilingRecord`, `SetPartitioningHelper`, `ShapeSymbolTable`, `RegisterProfilingNode`, `partitionSetByDimension`, `isBound`, `getValue`, `assign`
