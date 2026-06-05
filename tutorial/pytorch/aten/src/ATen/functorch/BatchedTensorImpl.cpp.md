# BatchedTensorImpl.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchedTensorImpl.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchedTensorImpl.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchedTensorImpl.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10 / 第 1-10 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: #include <ATen/functorch/BatchedTensorImpl.h>
0007: 
0008: #include <ATen/WrapDimUtils.h>
0009: #include <c10/util/Exception.h>
0010: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 11-26 / 第 11-26 行

```cpp
0011: #include <c10/util/irange.h>
0012: 
0013: namespace at::functorch {
0014: 
0015: BatchedTensorImpl::BatchedTensorImpl(DispatchKeySet key_set, Tensor value, int64_t bdim, int64_t level)
0016:   : TensorImpl(
0017:       key_set.add(
0018:           value.is_nested() ? DispatchKey::BatchedNestedTensor : DispatchKey::FuncTorchBatched),
0019:       value.dtype(),
0020:       value.device()
0021:     )
0022:   , value_(std::move(value))
0023:   , level_(level)
0024:   , bdim_(bdim)
0025: {
0026:   TORCH_INTERNAL_ASSERT(value_.defined());
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `BatchedTensorImpl`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`BatchedTensorImpl`。

### Lines 27-39 / 第 27-39 行

```cpp
0027:   if (value_.is_nested() || value_.key_set().has(DispatchKey::BatchedNestedTensor)) {
0028:     TORCH_CHECK(bdim_ == 0,
0029:         "Nested tensors can only be vmapped over dim=0, but got dim=", bdim_);
0030:     TORCH_CHECK(level_ == 1,
0031:         "Only one level of vmap is supported when vmapping over nested tensors");
0032:   }
0033:   set_storage_access_should_throw();
0034:   set_custom_sizes_strides(
0035:       value_.is_nested() ? SizesStridesPolicy::CustomSizes : SizesStridesPolicy::CustomStrides);
0036:   checkInvariants();
0037:   refreshTensorMetadata();
0038: }
0039: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `set_storage_access_should_throw`, `set_custom_sizes_strides`, `checkInvariants`, `refreshTensorMetadata`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`set_storage_access_should_throw`, `set_custom_sizes_strides`, `checkInvariants`, `refreshTensorMetadata`。

### Lines 40-52 / 第 40-52 行

```cpp
0040: void BatchedTensorImpl::refreshTensorMetadata() {
0041:   const auto public_dims = value_.dim() - 1;
0042:   if (value_.is_nested()) {
0043:     sizes_and_strides_.resize(public_dims);
0044:     storage_offset_= value_.storage_offset();
0045:     refresh_numel();
0046:     refresh_contiguous();
0047:   } else {
0048:     c10::SymDimVector new_sizes;
0049:     c10::SymDimVector new_strides;
0050:     new_sizes.reserve(public_dims);
0051:     new_strides.reserve(public_dims);
0052: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values. Key symbols: `refreshTensorMetadata`, `refresh_numel`, `refresh_contiguous`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值。关键符号：`refreshTensorMetadata`, `refresh_numel`, `refresh_contiguous`。

### Lines 53-63 / 第 53-63 行

```cpp
0053:     // update size, strides and storage_offset
0054:     // for tensor with symbolic size and strides
0055:     const auto value_sizes = value_.sym_sizes();
0056:     const auto value_strides = value_.sym_strides();
0057: 
0058:     for (const auto dim : c10::irange(0, public_dims)) {
0059:       auto actual_dim = actualDim(dim, /*wrap_dim=*/false);
0060:       new_sizes.push_back(value_sizes.at(actual_dim));
0061:       new_strides.push_back(value_strides.at(actual_dim));
0062:     }
0063: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 64-75 / 第 64-75 行

```cpp
0064:     // `set_sizes_and_strides` takes care of calling `refresh_numel` and
0065:     // `refresh_contiguous`
0066:     set_sizes_and_strides(new_sizes, new_strides, value_.sym_storage_offset());
0067:   }
0068: }
0069: 
0070: int64_t BatchedTensorImpl::actualDim(int64_t dim, bool wrap_dim) const {
0071:   if (wrap_dim) {
0072:     const auto ndim = sizes_and_strides_.size();
0073:     dim = maybe_wrap_dim(dim, static_cast<int64_t>(ndim));
0074:   }
0075:   if (bdim_ <= dim) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values. Key symbols: `set_sizes_and_strides`, `actualDim`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值。关键符号：`set_sizes_and_strides`, `actualDim`。

### Lines 76-85 / 第 76-85 行

```cpp
0076:     return dim + 1;
0077:   } else {
0078:     return dim;
0079:   }
0080: }
0081: 
0082: void BatchedTensorImpl::checkInvariants() const {
0083:   TORCH_INTERNAL_ASSERT(level_ > -1);
0084: }
0085: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `checkInvariants`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`checkInvariants`。

### Lines 86-95 / 第 86-95 行

```cpp
0086: int64_t BatchedTensorImpl::size_custom(int64_t d) const {
0087:   if (!value_.is_nested()) {
0088:     d = maybe_wrap_dim(d, dim(), /*wrap_scalar=*/false);
0089:     return sizes_default()[d];
0090:   }
0091:   // TODO: Error messages will mention the actualDim, which could be confusing; fix this
0092:   auto actual_dim = actualDim(d, /*wrap_dim=*/ true);
0093:   return value_.size(actual_dim);
0094: }
0095: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `size_custom`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`size_custom`。

### Lines 96-105 / 第 96-105 行

```cpp
0096: c10::SymInt BatchedTensorImpl::sym_size_custom(int64_t d) const {
0097:   if (!value_.is_nested()) {
0098:     d = maybe_wrap_dim(d, dim(), /*wrap_scalar=*/false);
0099:     return sym_sizes_default()[d];
0100:   }
0101:   // TODO: Error messages will mention the actualDim, which could be confusing; fix this
0102:   auto actual_dim = actualDim(d, /*wrap_dim=*/ true);
0103:   return value_.sym_size(actual_dim);
0104: }
0105: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `sym_size_custom`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`sym_size_custom`。

### Lines 106-115 / 第 106-115 行

```cpp
0106: IntArrayRef BatchedTensorImpl::sizes_custom() const {
0107:   TORCH_CHECK(!value_.is_nested(), "sizes() is not supported for batched nested tensors");
0108:   return sizes_default();
0109: }
0110: 
0111: SymIntArrayRef BatchedTensorImpl::sym_sizes_custom() const {
0112:   TORCH_CHECK(!value_.is_nested(), "sizes() is not supported for batched nested tensors");
0113:   return sym_sizes_default();
0114: }
0115: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `sizes_custom`, `sizes_default`, `sym_sizes_custom`, `sym_sizes_default`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`sizes_custom`, `sizes_default`, `sym_sizes_custom`, `sym_sizes_default`。

### Lines 116-125 / 第 116-125 行

```cpp
0116: // The following are publicly exposed as methods of Tensor
0117: 
0118: IntArrayRef BatchedTensorImpl::strides_custom() const {
0119:   return strides_default();
0120: }
0121: 
0122: SymIntArrayRef BatchedTensorImpl::sym_strides_custom() const {
0123:   return sym_strides_default();
0124: }
0125: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `strides_custom`, `strides_default`, `sym_strides_custom`, `sym_strides_default`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`strides_custom`, `strides_default`, `sym_strides_custom`, `sym_strides_default`。

### Lines 126-135 / 第 126-135 行

```cpp
0126: 
0127: // TODO: implement proper contiguity on batched tensor, then put
0128: // sizes_strides_policy back to Default
0129: c10::SymBool BatchedTensorImpl::sym_is_contiguous_custom(at::MemoryFormat memory_format) const {
0130:   TORCH_CHECK(memory_format == MemoryFormat::Contiguous,
0131:       "NYI: querying is_contiguous inside of vmap for memory_format ",
0132:       "other than torch.contiguous_format");
0133:   return is_contiguous_default(memory_format);
0134: }
0135: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `sym_is_contiguous_custom`, `is_contiguous_default`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`sym_is_contiguous_custom`, `is_contiguous_default`。

### Lines 136-146 / 第 136-146 行

```cpp
0136: // The following are some internal inherited methods that we do not support.
0137: // They should never get called.
0138: void BatchedTensorImpl::set_size(int64_t dim, int64_t new_size) {
0139:   TORCH_INTERNAL_ASSERT(false, "Can't set_size for BatchedTensorImpl");
0140: }
0141: void BatchedTensorImpl::set_stride(int64_t dim, int64_t new_stride) {
0142:   TORCH_INTERNAL_ASSERT(false, "Can't set_stride for BatchedTensorImpl");
0143: }
0144: #ifdef DEBUG
0145: bool BatchedTensorImpl::has_storage() const {
0146:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(!storage_, "BatchedTensorImpl assumes that storage_ is never set");
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `set_size`, `set_stride`, `has_storage`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`set_size`, `set_stride`, `has_storage`。

### Lines 147-158 / 第 147-158 行

```cpp
0147:   return false;
0148: }
0149: #endif
0150: 
0151: const char* BatchedTensorImpl::tensorimpl_type_name() const {
0152:   return "BatchedTensorImpl";
0153: }
0154: 
0155: c10::intrusive_ptr<TensorImpl> BatchedTensorImpl::shallow_copy_and_detach(
0156:     const c10::VariableVersion& version_counter,
0157:     bool allow_tensor_metadata_change) const {
0158:   TORCH_CHECK(false, "accessing `data` under vmap transform is not allowed");
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `tensorimpl_type_name`, `shallow_copy_and_detach`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`tensorimpl_type_name`, `shallow_copy_and_detach`。

### Lines 159-169 / 第 159-169 行

```cpp
0159: }
0160: 
0161: c10::intrusive_ptr<TensorImpl> BatchedTensorImpl::shallow_copy_and_detach(
0162:     // NOLINTNEXTLINE(cppcoreguidelines-rvalue-reference-param-not-moved)
0163:     c10::VariableVersion&& version_counter,
0164:     bool allow_tensor_metadata_change) const {
0165:   TORCH_CHECK(false, "accessing `data` under vmap transform is not allowed");
0166: }
0167: 
0168: void BatchedTensorImpl::shallow_copy_from(const c10::intrusive_ptr<TensorImpl>& impl) {
0169:   TORCH_CHECK(false, "mutating directly with `.data` under vmap transform is not allowed.");
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `shallow_copy_and_detach`, `shallow_copy_from`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`shallow_copy_and_detach`, `shallow_copy_from`。

### Lines 170-179 / 第 170-179 行

```cpp
0170: }
0171: 
0172: Tensor makeBatched(Tensor tensor, int64_t bdim, int64_t level) {
0173:   DispatchKeySet key_set = getKeysToPropagateToWrapper(tensor);
0174:   auto* batched = maybeGetBatchedImpl(tensor);
0175:   if (batched) {
0176:     auto batched_level = batched->level();
0177:     TORCH_INTERNAL_ASSERT(level > batched_level, " batched_level: ", batched_level, " level: ", level);
0178:   }
0179:   return at::detail::make_tensor<BatchedTensorImpl>(key_set, std::move(tensor), bdim, level);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `makeBatched`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`makeBatched`。

### Lines 180-186 / 第 180-186 行

```cpp
0180: }
0181: 
0182: Tensor addBatchDim(Tensor tensor, int64_t dim, int64_t level) {
0183:   return makeBatched(std::move(tensor), dim, level);
0184: }
0185: 
0186: } // namespace at::functorch
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `addBatchDim`, `makeBatched`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`addBatchDim`, `makeBatched`。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: BatchedTensorImpl, set_storage_access_should_throw, set_custom_sizes_strides, checkInvariants, refreshTensorMetadata, refresh_numel, refresh_contiguous, set_sizes_and_strides** — 核心符号：BatchedTensorImpl、set_storage_access_should_throw、set_custom_sizes_strides、checkInvariants、refreshTensorMetadata、refresh_numel、refresh_contiguous、set_sizes_and_strides

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchedTensorImpl.h`, `ATen/WrapDimUtils.h`, `c10/util/Exception.h`, `c10/util/irange.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `BatchedTensorImpl`, `set_storage_access_should_throw`, `set_custom_sizes_strides`, `checkInvariants`, `refreshTensorMetadata`, `refresh_numel`, `refresh_contiguous`, `set_sizes_and_strides`, `actualDim`, `size_custom`, `sym_size_custom`, `sizes_custom`, `...`
