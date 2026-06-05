# BatchedTensorImpl.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchedTensorImpl.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchedTensorImpl.h`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchedTensorImpl.h` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10 / 第 1-10 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #pragma once
0008: 
0009: #include <bitset>
0010: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。

### Lines 11-22 / 第 11-22 行

```cpp
0011: #include <ATen/ArrayRef.h>
0012: #include <ATen/SmallVector.h>
0013: #include <ATen/Tensor.h>
0014: 
0015: namespace at::functorch {
0016: 
0017: using Tensor = at::Tensor;
0018: 
0019: // We assume this in a few other places in the codebase,
0020: // but there isn't a centralized definition.
0021: constexpr int64_t kVmapMaxTensorDims = 64;
0022: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `Tensor`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`Tensor`。

### Lines 23-42 / 第 23-42 行

```cpp
0023: // The valid vmap levels range from [0, 64). This effectively means that we
0024: // support a maximum of 64 nested vmaps.
0025: constexpr int64_t kVmapNumLevels = 64;
0026: 
0027: // Store this number of elements of BatchDims on the stack. Most people will
0028: // probably use <= 5 nested vmaps, but adjust this number as necessary.
0029: constexpr int64_t kBatchDimsStackSize = 5;
0030: 
0031: // A BatchedTensorImpl holds an underlying Tensor and a single batch dim
0032: // NB: We use the term "BatchedTensor" to mean a Tensor that is backed with a
0033: // BatchedTensorImpl.
0034: //
0035: // The batch dimensions are treated as being "private"; they are not user-visible.
0036: // For example, in the following Tensor,
0037: //    bt = BatchedTensorImpl(ones(2, 3, 5, 7), lvl=1, dim=0)
0038: // dimension 0 is batch dimension.
0039: //
0040: // bt.sizes() returns (5, 7); bt.sum(0) performs a reduction over the (public)
0041: // dim 0, which is equivalent to dim 3 in the underlying ones(2, 3, 5, 7) tensor.
0042: struct TORCH_API BatchedTensorImpl : public c10::TensorImpl {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BatchedTensorImpl`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BatchedTensorImpl`。

### Lines 43-53 / 第 43-53 行

```cpp
0043:   explicit BatchedTensorImpl(at::DispatchKeySet key_set, Tensor value, int64_t dim, int64_t level);
0044: 
0045:   // Returns batch dimension of this tensor
0046:   int64_t bdim() const { return bdim_; }
0047: 
0048:   // Returns batch dimension of this tensor
0049:   int64_t level() const { return level_; }
0050: 
0051:   // BatchedTensorImpl wraps a Tensor
0052:   const Tensor& value() const { return value_; }
0053: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `BatchedTensorImpl`, `bdim`, `level`, `value`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`BatchedTensorImpl`, `bdim`, `level`, `value`。

### Lines 54-63 / 第 54-63 行

```cpp
0054:   // Given a public dimension index, return the dimension index in the underlying
0055:   // value() tensor.
0056:   // For example, if we have
0057:   //    bt = BatchedTensorImpl(ones(2, 3, 5, 7), lvl=1, dim=0)
0058:   // bt.actualDim(0) -> 1
0059:   // bt.actualDim(1) -> 2
0060:   // bt.actualDim(2) -> 3
0061:   // bt.actualDim(3) -> Error
0062:   int64_t actualDim(int64_t dim, bool wrap_dim = true) const;
0063: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `actualDim`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`actualDim`。

### Lines 64-82 / 第 64-82 行

```cpp
0064:   IntArrayRef sizes_custom() const override;
0065:   SymIntArrayRef sym_sizes_custom() const override;
0066:   int64_t size_custom(int64_t d) const override;
0067:   c10::SymInt sym_size_custom(int64_t d) const override;
0068:   // We have to override this because we opted into CustomStrides
0069:   IntArrayRef strides_custom() const override;
0070:   SymIntArrayRef sym_strides_custom() const override;
0071:   // Override a bunch of methods inherited from TensorImpl to return error messages.
0072:   c10::SymBool sym_is_contiguous_custom(at::MemoryFormat memory_format) const override;
0073:   void set_size(int64_t dim, int64_t new_size) override;
0074:   void set_stride(int64_t dim, int64_t new_stride) override;
0075:   c10::intrusive_ptr<TensorImpl> shallow_copy_and_detach(
0076:     const c10::VariableVersion& version_counter,
0077:     bool allow_tensor_metadata_change) const override;
0078:   c10::intrusive_ptr<TensorImpl> shallow_copy_and_detach(
0079:       c10::VariableVersion&& version_counter,
0080:       bool allow_tensor_metadata_change) const override;
0081:   void shallow_copy_from(const c10::intrusive_ptr<TensorImpl>& impl) override;
0082: #ifdef DEBUG
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `sizes_custom`, `sym_sizes_custom`, `size_custom`, `sym_size_custom`, `strides_custom`, `sym_strides_custom`, `sym_is_contiguous_custom`, `set_size`, `...`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`sizes_custom`, `sym_sizes_custom`, `size_custom`, `sym_size_custom`, `strides_custom`, `sym_strides_custom`, `sym_is_contiguous_custom`, `set_size`, `...`。

### Lines 83-94 / 第 83-94 行

```cpp
0083:   bool has_storage() const override;
0084: #endif
0085: 
0086:   void refreshTensorMetadata();
0087: 
0088:   // Used in torchdim. torchdim uses non-lexical BatchedTensor; the way it
0089:   // accomplishes this is a hack where it is able to modify the levels of
0090:   // BatchedTensor to match the level of the current vmap transform.
0091:   void _unsafe_set_level(int64_t level) {
0092:     level_ = level;
0093:   }
0094: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `has_storage`, `refreshTensorMetadata`, `_unsafe_set_level`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`has_storage`, `refreshTensorMetadata`, `_unsafe_set_level`。

### Lines 95-105 / 第 95-105 行

```cpp
0095:   // Used in batching rule for in-place view operations that can change
0096:   // the index of the bdim (think squeeze_, unsqueeze_)
0097:   void unsafe_set_bdim(int64_t bdim) {
0098:     // NB: you MUST call refreshTensorMetadata after doing this.
0099:     bdim_ = bdim;
0100:   }
0101:  private:
0102:   // see NOTE: [BatchedTensorImpl levels invariant]
0103:   void checkInvariants() const;
0104:   const char* tensorimpl_type_name() const override;
0105: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: `unsafe_set_bdim`, `checkInvariants`, `tensorimpl_type_name`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：`unsafe_set_bdim`, `checkInvariants`, `tensorimpl_type_name`。

### Lines 106-115 / 第 106-115 行

```cpp
0106:   Tensor value_;
0107: 
0108:   int64_t level_;
0109:   int64_t bdim_;
0110: };
0111: 
0112: // NB: We use the term "BatchedTensor" to mean a Tensor that is backed with a
0113: // BatchedTensorImpl.
0114: inline bool isBatchedTensor(const Tensor& tensor) {
0115:   return tensor.unsafeGetTensorImpl()->key_set().has(DispatchKey::FuncTorchBatched) ||
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `isBatchedTensor`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`isBatchedTensor`。

### Lines 116-125 / 第 116-125 行

```cpp
0116:       tensor.unsafeGetTensorImpl()->key_set().has(DispatchKey::BatchedNestedTensor);
0117: }
0118: 
0119: // It is unsafe to call this on a Tensor that is not backed by a
0120: // BatchedTensorImpl. Please use `maybeGetBatchedImpl` whenever possible.
0121: inline BatchedTensorImpl* unsafeGetBatchedImpl(const Tensor& tensor) {
0122:   return static_cast<BatchedTensorImpl*>(tensor.unsafeGetTensorImpl());
0123: }
0124: 
0125: inline BatchedTensorImpl* maybeGetBatchedImpl(const Tensor& tensor) {
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `unsafeGetBatchedImpl`, `maybeGetBatchedImpl`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`unsafeGetBatchedImpl`, `maybeGetBatchedImpl`。

### Lines 126-136 / 第 126-136 行

```cpp
0126:   if (!isBatchedTensor(tensor)) {
0127:     return nullptr;
0128:   }
0129:   return unsafeGetBatchedImpl(tensor);
0130: }
0131: 
0132: // Returns a bitset. If bit i is set, then that means dim i is a batchdim.
0133: inline std::bitset<kVmapMaxTensorDims> createBatchDimBitset(int64_t dim) {
0134:   std::bitset<kVmapMaxTensorDims> is_bdim;
0135:   is_bdim.set(dim);
0136:   return is_bdim;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `unsafeGetBatchedImpl`, `createBatchDimBitset`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`unsafeGetBatchedImpl`, `createBatchDimBitset`。

### Lines 137-147 / 第 137-147 行

```cpp
0137: }
0138: 
0139: // Creates a bitset for the given level
0140: inline std::bitset<kVmapNumLevels> createVmapLevelsBitset(int64_t level) {
0141:   std::bitset<kVmapNumLevels> result;
0142:   result.set(level);
0143:   return result;
0144: }
0145: 
0146: // Use this to construct a BatchedTensor from a regular Tensor
0147: TORCH_API Tensor makeBatched(Tensor tensor, int64_t dim, int64_t level);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `createVmapLevelsBitset`, `makeBatched`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`createVmapLevelsBitset`, `makeBatched`。

### Lines 148-167 / 第 148-167 行

```cpp
0148: 
0149: // Adds a batch dim to `tensor`, returning a BatchedTensor
0150: TORCH_API Tensor addBatchDim(Tensor tensor, int64_t dim, int64_t level);
0151: 
0152: // Certain dispatch keys must be propagated to the BatchedTensor (or, in general,
0153: // any wrapper Tensor subclasses). This is because there are methods on Tensor
0154: // that skip dispatch and check for the presence of a dispatch key (e.g. is_cpu()).
0155: // TODO: should probably contain more (or all?) backend keys
0156: constexpr DispatchKeySet kKeysToPropagateToWrapper({
0157:   DispatchKey::Negative,
0158:   DispatchKey::Conjugate,
0159:   DispatchKey::XLA,
0160:   DispatchKey::XPU,
0161:   DispatchKey::HPU,
0162:   DispatchKey::CUDA,
0163:   DispatchKey::CPU,
0164:   DispatchKey::PrivateUse1,
0165:   DispatchKey::SparseCPU,
0166:   DispatchKey::SparseCUDA,
0167:   DispatchKey::SparseCsrCPU,
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: `addBatchDim`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`addBatchDim`。

### Lines 168-176 / 第 168-176 行

```cpp
0168:   DispatchKey::SparseCsrCUDA,
0169: });
0170: 
0171: inline DispatchKeySet getKeysToPropagateToWrapper(const Tensor& tensor, DispatchKeySet to_propagate=kKeysToPropagateToWrapper) {
0172:   auto key_set = tensor.unsafeGetTensorImpl()->key_set();
0173:   return key_set & kKeysToPropagateToWrapper;
0174: }
0175: 
0176: } // namespace at::functorch
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details. Key symbols: `getKeysToPropagateToWrapper`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`getKeysToPropagateToWrapper`。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: BatchedTensorImpl, Tensor, bdim, level, value, actualDim, sizes_custom, sym_sizes_custom** — 核心符号：BatchedTensorImpl、Tensor、bdim、level、value、actualDim、sizes_custom、sym_sizes_custom

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/ArrayRef.h`, `ATen/SmallVector.h`, `ATen/Tensor.h`
- **External includes / 外部头文件**: `bitset`
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `BatchedTensorImpl`, `Tensor`, `bdim`, `level`, `value`, `actualDim`, `sizes_custom`, `sym_sizes_custom`, `size_custom`, `sym_size_custom`, `strides_custom`, `sym_strides_custom`, `...`
