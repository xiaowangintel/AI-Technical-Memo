# memory_planner.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/runtime/static/memory_planner.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
#pragma once

#include <torch/csrc/jit/runtime/static/impl.h>

namespace torch::jit {

// A StorageGroup represents a collection of tensors that share backing storage.
class StorageGroup {
 public:
  // Every storage group must contain at least one tensor.
  explicit StorageGroup(at::Tensor* tensor) : group_{tensor} {}

  void addTensor(at::Tensor* tensor) {
    group_.push_back(tensor);
  }
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/runtime/static/impl.h. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem. It introduces or extends StorageGroup, which define the primary data structures or interfaces for this portion of the file. This chunk defines `addTensor`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/runtime/static/impl.h。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。 它引入或扩展了 StorageGroup，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `addTensor`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 17-31
```cpp
  const std::vector<at::Tensor*>& group() const {
    return group_;
  }

  size_t maxTensorSize() const {
    return max_tensor_size_;
  }

  void setMaxTensorSize(size_t new_size) {
    max_tensor_size_ = new_size;
  }

  size_t numManagedTensors() const {
    return group_.size();
  }
```
- **EN**: This chunk defines `numManagedTensors`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `numManagedTensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 33-46
```cpp
 private:
  // The size attribute represents the amount of memory that will be
  // allocated for all tensors in this storage group. Initially it
  // is zero, eventually it gets updated by the MemoryPlanner.
  size_t max_tensor_size_ = 0;
  std::vector<at::Tensor*> group_;
};

// A contiguous buffer of `StorageImpl`s
class ManagedStorages {
 public:
  ManagedStorages();

  ~ManagedStorages();
```
- **EN**: It introduces or extends ManagedStorages, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 ManagedStorages，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 48-58
```cpp
  void allocate(size_t capacity);

  void deallocate();

  bool is_allocated() const {
    return storages_ != nullptr;
  }

  // Append a new StorageImpl to the buffer. The new StorageImpl is given the
  // same size and allocator as `storageImpl` argument
  void append(at::StorageImpl& storageImpl);
```
- **EN**: This chunk defines `append`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `append`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 60-72
```cpp
  at::StorageImpl& operator[](size_t idx) {
    TORCH_INTERNAL_ASSERT(storages_ != nullptr);
    return storages_[idx];
  }

  const at::StorageImpl& operator[](size_t idx) const {
    TORCH_INTERNAL_ASSERT(storages_ != nullptr);
    return storages_[idx];
  }

  size_t size() const {
    return size_;
  }
```
- **EN**: This chunk defines `size`, which implements a focused step in the TorchScript execution pipeline. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `size`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 74-87
```cpp
  bool empty() const {
    return size_ == 0;
  }

  size_t capacity() const {
    return capacity_;
  }

 private:
  // We will use placement-new to add new storages to this buffer
  at::StorageImpl* storages_{nullptr};

  // Current number of storages that have been placed into the storage buffer
  size_t size_{0};
```
- **EN**: This chunk defines `capacity`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `capacity`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 89-104
```cpp
  // Total allocated capacity of the storage buffer
  size_t capacity_{0};
};

TORCH_API std::vector<StorageGroup> assignStorageToManagedTensors(
    graph_node_list nodes,
    const ManagedTensorRanges& ranges,
    const c10::FastMap<const Value*, at::Tensor*>& tensor_value_to_tensor);

// There are three types of ops in a processed graph in Static Runtime:
//   1. op with _out variant
//   2. view-producing op
//   3. tensor-producing op (could be replaced with type 1 by adding the _out
//      variant to Static Runtime)
// In Static Runtime, type 2 ops are replaced with their corresponding copy
// versions when enable_out_variant is enabled and become type 1 ops.The memory
```
- **EN**: This chunk defines `assignStorageToManagedTensors`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `assignStorageToManagedTensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 105-120
```cpp
// planner only manages tensors that are outputs of type 1 ops. For type 3, the
// output tensors are allocated inside the operator and can't be directly
// managed by memory planner.
//
// Memory planner tries to minimize the number of memory allocations by
// tracking the output tensors of ops with _out variants with unique DataPtr
// (part of StorageImpl). It tries to do this in several steps:
//   1. record the max memory usage for each Tensor with unique DataPtr at the
//      end of each iteration
//   2. in the next iteration, allocate the buffer for the max total usage and
//      compute the offset of each allocation with regard to the single memory
//      buffer, optionally reusing memory. In the first iteration, we rely on
//      the default allocator for memory allocation.
//   3. free the buffer at the end of each iteration
// Steps 1 and 3 are handled by `deallocate()`, and step 2 by `allocate()`.
// Only models with simple output types are supported, i.e. None, Tensor or
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 121-136
```cpp
// List/Tuple/Dict of Tensors. Complex output types such as List of Lists are
// not supported.
//
// Additional Optimizations:
//
// [Borrowed IValue Outputs]
// A few native ops (notably, `static_runtime::dict_unpack` and
// `static_runtime::VarTupleUnpack`) simply unpack IValues to a bunch of
// outputs without modification. For example, `dict_unpack` does the following:
// for each key in inputs:
//     output[i] = dict_input[key]
// To avoid refcount bumps, the outputs of these ops are non-owning references.
// This requires special logic in the memory planner - when adding an op that
// borrows outputs, be sure that the memory planner is updated accordingly!
//
// [Managed Output Tensors]
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 137-150
```cpp
// The memory planner is able to manage output tensors if the appropriate
// `StaticModuleOptions` are set. However, the memory planner handles output
// tensors separately from regular intermediate tensors:
// 1. They don't participate in memory reuse.
// 2. The memory planner cannot reclaim their backing storage until they have
//    been explicitly freed by the client.

class MemoryPlanner {
 public:
  MemoryPlanner(
      BlockRunner* block_runner,
      const BlockInfo& block_info,
      bool enable_out_variant,
      bool manage_output_tensors);
```
- **EN**: It introduces or extends MemoryPlanner, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 MemoryPlanner，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 152-165
```cpp
  // disable copying and moving
  MemoryPlanner(const MemoryPlanner&) = delete;
  MemoryPlanner& operator=(const MemoryPlanner&) = delete;
  MemoryPlanner(MemoryPlanner&&) = delete;
  MemoryPlanner& operator=(MemoryPlanner&&) = delete;
  virtual ~MemoryPlanner() = default;

  void allocate();
  void deallocate();
  void deallocateOutputTensors();

  size_t total_num_managed_tensors() const {
    return num_managed_tensors_;
  }
```
- **EN**: This chunk defines `total_num_managed_tensors`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `total_num_managed_tensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 167-181
```cpp
  size_t total_reused_tensors() const {
    return reused_tensors_;
  }

  size_t total_num_managed_output_tensors() const {
    return managed_output_tensors_.size();
  }

  [[nodiscard]] size_t total_num_unmanaged() const {
    return num_unmanaged_non_scalars() + num_unmanaged_scalars();
  }

  [[nodiscard]] size_t num_unmanaged_non_scalars() const {
    return unmanaged_ivalues_.size() + unmanaged_borrowed_ivalues_.size();
  }
```
- **EN**: This chunk defines `total_num_managed_output_tensors`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `total_num_managed_output_tensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 183-193
```cpp
  [[nodiscard]] size_t num_unmanaged_scalars() const {
    return num_unmanaged_scalar_ivalues_;
  }

  size_t total_managed() const {
    return managed_bytes_;
  }

  size_t numOutputBufferBytes() const {
    return output_buffer_bytes_;
  }
```
- **EN**: This chunk defines `numOutputBufferBytes`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `numOutputBufferBytes`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 195-210
```cpp
  // Check if `ivalue` is contained as a managed tensor. Only used in DCHECK().
  bool isManagedOutputTensor(const IValue& ivalue) const {
    if (!output_buffer_ || // output buffer got already deallocated.
        output_buffer_bytes_ == 0 || // memory planning is not yet initialized.
        !ivalue.isTensor() // a non-tensor is never managed
    ) {
      return false;
    }
    const auto& tensor = ivalue.toTensor();
    if (!tensor.has_storage() || !tensor.storage().data_ptr()) {
      return false;
    }
    // TODO: Improve this once D31357486 is landed.
    uint8_t* tensor_ptr =
        static_cast<uint8_t*>(tensor.storage().data_ptr().get());
    uint8_t* buffer_start = static_cast<uint8_t*>(output_buffer_.get());
```
- **EN**: This chunk defines `isManagedOutputTensor`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isManagedOutputTensor`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 211-226
```cpp
    uint8_t* buffer_end = buffer_start + output_buffer_bytes_;
    return buffer_start <= tensor_ptr && tensor_ptr < buffer_end;
  }

  bool isManagedStorageImpl(const at::StorageImpl* impl) const {
    if (storages_.empty()) {
      return false;
    }
    // Comparing pointers that aren't within the same array is
    // UB. We're doing fancy memory allocation stuff, so we cast to an
    // integer type and carry on.
    const auto impl_p = reinterpret_cast<uintptr_t>(impl);
    const auto start = reinterpret_cast<uintptr_t>(&storages_[0]);
    const auto end =
        reinterpret_cast<uintptr_t>(&storages_[0] + storages_.size());
    return impl_p >= start && impl_p < end;
```
- **EN**: This chunk defines `isManagedStorageImpl`, which implements a focused step in the TorchScript execution pipeline. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `isManagedStorageImpl`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 227-237
```cpp
  }

  bool overlapWithInternalBuffer(void* data_ptr) {
    return buffer_start_ <= data_ptr && data_ptr < buffer_end_;
  }

 protected:
  uint8_t* allocateBuffer(size_t num_bytes);

  size_t managed_bytes_{0};
  size_t reused_tensors_{0};
```
- **EN**: This chunk defines `allocateBuffer`, which implements a focused step in the TorchScript execution pipeline. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `allocateBuffer`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 239-250
```cpp
  // We allocate StorageImpls ourselves so that 1) we don't have to do
  // an extra two loads per Tensor (which will likely miss in the CPU
  // data cache) first reading the Storage (i.e., StorageImpl pointer)
  // from the TensorImpl object and then second dereferencing it and
  // 2) our memory access pattern during allocate() has high locality.
  // We don't have any guarantee that the model doesn't change the
  // Storage for managed tensors out from under us during execution,
  // so we have to check the StorageImpls each time we deallocate.
  ManagedStorages storages_;

  // Contains the size (in bytes) of the data to be allocated for each storage
  std::vector<size_t> storages_nbytes_;
```
- **EN**: This chunk continues `allocateBuffer` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `allocateBuffer`，进一步展开其内部控制流或数据流转。

### Lines 252-264
```cpp
 private:
  // ivalues created in one run but not managed by MemoryPlanner
  std::vector<IValue*> unmanaged_ivalues_;

  // Special class of unmanaged values: some native ops create IValues
  // in a "borrowed" state that can and must be cleaned up without a
  // reference count decrement.
  std::vector<IValue*> unmanaged_borrowed_ivalues_;

  // Even more special class of unmanaged values: if select_tensor
  // outputs are outputs of the graph, then they need to be restored
  // to an ordinary "strong reference" state.
  std::vector<IValue*> borrowed_ivalues_needing_incref_;
```
- **EN**: This chunk continues `allocateBuffer` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `allocateBuffer`，进一步展开其内部控制流或数据流转。

### Lines 266-280
```cpp
  std::vector<std::pair<size_t, at::Tensor*>> managed_output_tensors_;
  at::DataPtr buffer_; // allocated each time we call Run()
  uint8_t* buffer_start_{nullptr};
  uint8_t* buffer_end_{nullptr};
  size_t num_managed_tensors_{0};
  size_t num_unmanaged_scalar_ivalues_{0};

  at::DataPtr output_buffer_;
  size_t output_buffer_bytes_{0};

  virtual void allocateManagedTensors() = 0;
  virtual void deallocateManagedTensors() = 0;

  void allocateOutputTensors();
};
```
- **EN**: This chunk defines `allocateOutputTensors`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 这一段定义了 `allocateOutputTensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 282-296
```cpp
class StandardMemoryPlanner : public MemoryPlanner {
 public:
  StandardMemoryPlanner(
      BlockRunner* block_runner,
      const BlockInfo& block_info,
      bool enable_out_variant,
      bool manage_output_tensors,
      bool optimize_memory);

 protected:
  void allocateManagedTensors() override;
  void deallocateManagedTensors() override;

  std::vector<StorageGroup> managed_tensors_;
};
```
- **EN**: It introduces or extends StandardMemoryPlanner, which define the primary data structures or interfaces for this portion of the file. This chunk defines `deallocateManagedTensors`, which implements a focused step in the TorchScript execution pipeline.
- **CN**: 它引入或扩展了 StandardMemoryPlanner，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `deallocateManagedTensors`，其作用是实现 TorchScript 执行流水线中的一个关键步骤。

### Lines 298-298
```cpp
} // namespace torch::jit
```
- **EN**: This chunk continues `deallocateManagedTensors` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `deallocateManagedTensors`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **TorchScript runtime**
  - EN: Implements TorchScript runtime execution, specialization, operator dispatch, and static-runtime support.
  - CN: 实现 TorchScript 运行时执行、特化、算子分发以及静态运行时支持。
- **StorageGroup**
  - EN: `StorageGroup` is a central symbol declared or implemented in this file.
  - CN: `StorageGroup` 是本文件声明或实现的核心符号。
- **ManagedStorages**
  - EN: `ManagedStorages` is a central symbol declared or implemented in this file.
  - CN: `ManagedStorages` 是本文件声明或实现的核心符号。
- **Static runtime**
  - EN: Optimizes execution around precomputed layouts, generated operators, and lightweight runtime state.
  - CN: 围绕预计算布局、生成算子以及轻量运行时状态来优化执行。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/runtime/static/impl.h`
- **Primary symbols in this file / 本文件核心符号**: `StorageGroup`, `ManagedStorages`, `MemoryPlanner`, `addTensor`, `group`, `maxTensorSize`, `setMaxTensorSize`, `numManagedTensors`
