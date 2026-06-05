# FunctionalStorageImpl.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/FunctionalStorageImpl.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines shared ATen abstractions, tensor utilities, device helpers, or interop glue used across subsystems. This specific file centers on `FunctionalStorageImpl.cpp`. Transform-aware tensor semantics used by functorch are part of the file scope. Tensor metadata shaping and storage/layout checks are part of the local logic.
- **Purpose (CN)**: 定义跨子系统复用的共享 ATen 抽象、张量工具、设备辅助逻辑或互操作胶水代码。 该文件具体围绕 `FunctionalStorageImpl.cpp` 展开。 该文件范围内还包含 functorch 所需的变换感知张量语义。 张量元数据组织以及存储/布局检查也是局部逻辑的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10 / 第 1-10 行

```cpp
0001: #include <ATen/FunctionalStorageImpl.h>
0002: 
0003: #include <ATen/EmptyTensor.h>
0004: #include <ATen/FunctionalTensorWrapper.h>
0005: #include <ATen/SparseCsrTensorUtils.h>
0006: #include <ATen/core/LegacyTypeDispatch.h>
0007: #include <c10/util/Exception.h>
0008: #include <vector>
0009: 
0010: namespace at::functionalization {
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 11-30 / 第 11-30 行

```cpp
0011: 
0012: // Note [Functionalization: Alias Removal Part 2]
0013: // See Note [Functionalization: Alias Removal] for more details.
0014: // This function applies a single update from one of the views to the StorageImpl.
0015: // We start out with <original_base> and <mutated_view>, and our goal is to end up with <mutated_base>.
0016: // Consider this program:
0017: //
0018: // base = ...
0019: // a = base.view1()
0020: // b = a.view2()
0021: // c = b.view3()
0022: // c.add_(3)
0023: //
0024: // Then the functionalization pass will queue an update as follows:
0025: //
0026: // update.new_val = c  # the updated value of c
0027: // update.view_metas = [view1_meta, view2_meta, view3_meta]
0028: //
0029: // Syncing any of a, b or c will eventually call apply_update() on the storage, and the following will run:
0030: //
```

- **EN:** Documents design intent, constraints, compatibility notes, or usage expectations for the surrounding implementation.
- **CN:** 说明周围实现的设计目标、约束条件、兼容性注意事项或使用预期。

### Lines 31-40 / 第 31-40 行

```cpp
0031: // tmp_values = [base, a, b]  # NB: c is not necessary
0032: // t = update.new_val
0033: // t = view3_inverse(b, t, 0)  # 0 is output index, these are all single output views so it's 0
0034: // t = view2_inverse(a, t, 0)
0035: // t = view1_inverse(base, t, 0)  # t now represents the updated storage.
0036: // storage.base_ = t
0037: static const Tensor apply_update(const FunctionalStorageImpl::Update& update, const Tensor& base) {
0038:   at::Tensor t = update.new_val;
0039:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(t));
0040:   if (update.view_metas.empty()) { return t; }
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `apply_update`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`apply_update`。

### Lines 41-52 / 第 41-52 行

```cpp
0041: 
0042:   std::vector<at::Tensor> tmp_values({base});
0043:   tmp_values.reserve(update.view_metas.size());
0044:   for (size_t i = 0; i < update.view_metas.size() - 1; ++i) {
0045:     at::Tensor next_view = update.view_metas[i]->forward(tmp_values.back());
0046:     // NB: We only actually need tmp_values for ops like select/slice/diagonal/squeeze/as_strided
0047:     // All of these ops require additional information to recover the sizes of the original tensor.
0048:     // If need to, we could probably apply this optimization and only bother computing tmp_values
0049:     // for those necessary view ops.
0050:     tmp_values.push_back(std::move(next_view));
0051:   }
0052:   for(int64_t i = static_cast<int64_t>(update.view_metas.size()) - 1; i >= 0; --i) {
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 53-69 / 第 53-69 行

```cpp
0053:     // Each view inverse is implemented in ViewInverses.cpp.
0054:     t = update.view_metas[i]->reverse(tmp_values[i], t);
0055:   }
0056:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(t));
0057:   return t;
0058: }
0059: 
0060: 
0061: static c10::SymInt get_nbytes(const Tensor& value) {
0062:   // The functionalization story when wrapping tensors that don't have storage
0063:   // is a bit wonky, but fortunately for some models (e.g., dlrm) we never
0064:   // actually perform mutations on these tensors, so you never really get
0065:   // called out on it.  For now, functionalization still creates "storages"
0066:   // for these tensors (which is wrong), but we don't give them any space.
0067:   // A more proper fix would be to have a SparseFunctionalTensorWrapper that
0068:   // models sparse correctly.
0069:   if (value.is_sparse() || at::sparse_csr::is_sparse_compressed(value)) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `get_nbytes`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`get_nbytes`。

### Lines 70-80 / 第 70-80 行

```cpp
0070:     return 0;
0071:   }
0072:   if (value.unsafeGetTensorImpl()->has_symbolic_sizes_strides()) {
0073:     // Today, the two implementations of SymInt are in Python (proxy tensor),
0074:     // and lazy tensor (LTC/XLA).
0075:     // LTC hasn't implemented SymInt support yet though
0076:     // Once it does, we should remove this check.
0077:     if (value.key_set().has(c10::DispatchKey::Python)) {
0078:       return value.storage().sym_nbytes();
0079:     }
0080:     return at::detail::computeStorageNbytes(value.sym_sizes(), value.sym_strides(),static_cast<int64_t>(value.dtype().itemsize()), value.sym_storage_offset());
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `computeStorageNbytes`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`computeStorageNbytes`。

### Lines 81-99 / 第 81-99 行

```cpp
0081:   }
0082:   // XLA storage objects also do not properly track nbytes.
0083:   return static_cast<int64_t>(at::detail::computeStorageNbytes(value.sizes(), value.strides(), value.dtype().itemsize(), value.storage_offset()));
0084: }
0085: 
0086: FunctionalStorageImpl::FunctionalStorageImpl(const Tensor& base)
0087:   : c10::StorageImpl(
0088:       c10::StorageImpl::use_byte_size_t(),
0089:       get_nbytes(base),
0090:       DataPtr{nullptr, base.device()},
0091:       GetAllocator(kMeta),
0092:       /*resizable=*/true
0093:     ),
0094:     base_(base)
0095: {
0096:   // SparseTensorImpl has no storage, so we cannot query its nbytes.
0097:   // (original_storage_size is only used for storage resizing in fsdp anyway, which does not apply to sparse)
0098:   // Same for XLA
0099:   if (base.unsafeGetTensorImpl()->has_storage() && data_ptr().device().type() != c10::DeviceType::XLA) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; manages memory allocation, buffers, or ownership boundaries. Key symbols: `GetAllocator`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；管理内存分配、缓冲区或所有权边界。关键符号：`GetAllocator`。

### Lines 100-109 / 第 100-109 行

```cpp
0100:     original_storage_size_ = base.unsafeGetTensorImpl()->unsafe_storage().unsafeGetStorageImpl()->sym_nbytes();
0101:   } else {
0102:     original_storage_size_ = -1;
0103:   }
0104:   curr_storage_size_ = original_storage_size_;
0105:   TORCH_INTERNAL_ASSERT(!at::functionalization::impl::isFunctionalTensor(base_));
0106: }
0107: 
0108: void FunctionalStorageImpl::add_update(const Tensor& updated_val, const std::vector<std::shared_ptr<ViewMeta>>& metas) {
0109:   TORCH_CHECK(!frozen_, "cannot mutate tensors with frozen storage");
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `add_update`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`add_update`。

### Lines 110-125 / 第 110-125 行

```cpp
0110: 
0111:   if (metas.size() > 1) {
0112:     for (size_t i = 1; i < metas.size(); ++i) {
0113:       // Skipping this check for XLA. Would be good to add it back, but it is failing XLA CI
0114:       TORCH_CHECK(updated_val.device().type() == c10::DeviceType::XLA || !metas[i]->is_as_strided,
0115: "During torch.compile, encountered a mutation on a view chain of length ", metas.size(), ", where view ", i,
0116: " was an as_strided() call. as_strided() is non-compositional, and therefore is not possible to functionalize properly today,"
0117: "so this behavior is banned in compile. As a workaround, you can either remove the mutation from the model code, or you "
0118: "can insert a graph break right before the mutation with torch._dynamo.graph_break(). If you would like this behavior to "
0119: "work properly, please comment on https://github.com/pytorch/pytorch/issues/104505.");
0120:     }
0121:   }
0122:   updates_.push_back({updated_val, metas});
0123:   generation_++;
0124: }
0125: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 126-137 / 第 126-137 行

```cpp
0126: bool FunctionalStorageImpl::apply_updates() {
0127:   // N.B:none of the tensors used in this function should be FunctionalTensorWrappers at this point.
0128:   // The only reason we currently need the TLS exclude guard here is because of functorch's DynamicLayer stack.
0129:   // It adds the Functionalize key into TLS before redispatching to the functionalization kernels,
0130:   // which means that we need to explicitly exclude it here before doing any other work underneath the pass.
0131:   at::AutoDispatchSkipFunctionalize guard;
0132:   bool any_updates = !updates_.empty();
0133:   for (auto& update_data: updates_) {
0134:     base_ = apply_update(update_data, base_);
0135:   }
0136:   updates_.clear();
0137:   return any_updates;
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `apply_updates`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`apply_updates`。

### Lines 138-140 / 第 138-140 行

```cpp
0138: }
0139: 
0140: } // namespace at::functionalization
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **Shared ATen infrastructure** — 共享 ATen 基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: apply_update, get_nbytes, computeStorageNbytes, GetAllocator, add_update, apply_updates** — 核心符号：apply_update、get_nbytes、computeStorageNbytes、GetAllocator、add_update、apply_updates

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/FunctionalStorageImpl.h`, `ATen/EmptyTensor.h`, `ATen/FunctionalTensorWrapper.h`, `ATen/SparseCsrTensorUtils.h`, `ATen/core/LegacyTypeDispatch.h`, `c10/util/Exception.h`
- **External includes / 外部头文件**: `vector`
- **Namespaces / 命名空间**: `at::functionalization`
- **Representative symbols / 代表性符号**: `apply_update`, `get_nbytes`, `computeStorageNbytes`, `GetAllocator`, `add_update`, `apply_updates`
