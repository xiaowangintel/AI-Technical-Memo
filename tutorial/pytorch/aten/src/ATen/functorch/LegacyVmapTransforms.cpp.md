# LegacyVmapTransforms.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/LegacyVmapTransforms.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `LegacyVmapTransforms.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `LegacyVmapTransforms.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/LegacyVmapTransforms.h>
0008: #include <ATen/functorch/DynamicLayer.h>
0009: 
0010: #include <c10/util/irange.h>
0011: 
0012: namespace at::functorch {
0013: 
0014: // Takes a BatchedTensorImpl, permutes all of the batch dims to the front,
0015: // and then returns a physical version of the Tensor.
0016: static Tensor permuteBatchDimsToFront(const BatchedTensorImpl* batched) {
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `permuteBatchDimsToFront`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`permuteBatchDimsToFront`。

### Lines 17-33 / 第 17-33 行

```cpp
0017:   const Tensor& physical_tensor = batched->value();
0018:   if (batched->bdim() == 0) {
0019:     return physical_tensor;
0020:   }
0021:   const auto sizes = physical_tensor.sym_sizes();
0022:   VmapDimVector permutation(sizes.size(), 0);
0023:   permutation.reserve(sizes.size());
0024:   const auto is_bdim = createBatchDimBitset(batched->bdim());
0025:   int64_t idx = 0;
0026:   permutation[idx++] = batched->bdim();
0027:   for (const auto ptr : c10::irange(0, sizes.size())) {
0028:     if (is_bdim[ptr]) {
0029:       continue;
0030:     }
0031:     permutation[idx++] = static_cast<int64_t>(ptr);
0032:   }
0033:   return physical_tensor.permute(permutation);
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `permutation`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`permutation`。

### Lines 34-47 / 第 34-47 行

```cpp
0034: }
0035: 
0036: VmapPhysicalView MultiBatchVmapTransform::logicalToPhysical(const Tensor& logical_tensor) {
0037:   auto* batched = maybeGetBatchedImpl(logical_tensor);
0038:   TORCH_INTERNAL_ASSERT(
0039:       batched,
0040:       "logicalToPhysical(tensor) should only be passed a BatchedTensor");
0041:   return { permuteBatchDimsToFront(batched), createVmapLevelsBitset(batched->level()) };
0042: }
0043: 
0044: int64_t VmapPhysicalView::numBatchDims() const {
0045:   return static_cast<int64_t>(levels_.count());
0046: }
0047: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `logicalToPhysical`, `numBatchDims`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`logicalToPhysical`, `numBatchDims`。

### Lines 48-62 / 第 48-62 行

```cpp
0048: int64_t VmapPhysicalView::numLogicalDims() const {
0049:   return /*physical*/tensor_.dim() - numBatchDims();
0050: }
0051: 
0052: VmapDimVector VmapPhysicalView::getPhysicalDims(IntArrayRef logical_dims) const {
0053:   auto logical_ndim = numLogicalDims();
0054:   // NB: fmap doesn't have a SmallVector variant, so we don't use it here.
0055:   VmapDimVector result;
0056:   result.reserve(logical_ndim);
0057:   for (auto dim : logical_dims) {
0058:     result.push_back(maybe_wrap_dim(dim, logical_ndim) + numBatchDims());
0059:   }
0060:   return result;
0061: }
0062: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `numLogicalDims`, `getPhysicalDims`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`numLogicalDims`, `getPhysicalDims`。

### Lines 63-76 / 第 63-76 行

```cpp
0063: int64_t VmapPhysicalView::getPhysicalDim(int64_t logical_dim) const {
0064:   auto logical_ndim = numLogicalDims();
0065:   return maybe_wrap_dim(logical_dim, logical_ndim) + numBatchDims();
0066: }
0067: 
0068: VmapDimVector VmapPhysicalView::getPhysicalShape(IntArrayRef logical_shape) const {
0069:   VmapDimVector result;
0070:   result.reserve(logical_shape.size() + numBatchDims());
0071:   auto tensor_sizes = tensor_.sizes();
0072:   result.insert(result.end(), tensor_sizes.begin(), tensor_sizes.begin() + numBatchDims());
0073:   result.insert(result.end(), logical_shape.begin(), logical_shape.end());
0074:   return result;
0075: }
0076: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `getPhysicalDim`, `maybe_wrap_dim`, `getPhysicalShape`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`getPhysicalDim`, `maybe_wrap_dim`, `getPhysicalShape`。

### Lines 77-90 / 第 77-90 行

```cpp
0077: SymDimVector VmapPhysicalView::getPhysicalShape(c10::SymIntArrayRef logical_shape) const {
0078:   SymDimVector result;
0079:   result.reserve(logical_shape.size() + numBatchDims());
0080:   auto tensor_sizes = tensor_.sym_sizes();
0081:   result.insert(result.end(), tensor_sizes.begin(), tensor_sizes.begin() + numBatchDims());
0082:   result.insert(result.end(), logical_shape.begin(), logical_shape.end());
0083:   return result;
0084: }
0085: 
0086: static std::tuple<int64_t, int64_t> computeFrontBatchDimsFromLevels(std::bitset<kVmapNumLevels> levels_bitset) {
0087:   int64_t level = 0;
0088:   int64_t dim = 0;
0089:   for (; level < kVmapNumLevels; level++) {
0090:     if (!levels_bitset[level]) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `getPhysicalShape`, `computeFrontBatchDimsFromLevels`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`getPhysicalShape`, `computeFrontBatchDimsFromLevels`。

### Lines 91-107 / 第 91-107 行

```cpp
0091:       continue;
0092:     }
0093:     break;
0094:   }
0095:   return std::make_tuple(dim, level);
0096: }
0097: 
0098: static Tensor moveDimToFrontAndExpand(Tensor tensor, std::optional<int64_t> dim, c10::SymInt size) {
0099:   if (dim) {
0100:     tensor = tensor.movedim(*dim, 0);
0101:   } else {
0102:     tensor = tensor.unsqueeze(0);
0103:     auto expanded_sizes = tensor.sym_sizes().vec();
0104:     expanded_sizes[0] = std::move(size);
0105:     tensor = tensor.expand_symint(expanded_sizes);
0106:   }
0107:   return tensor;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `moveDimToFrontAndExpand`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `moveDimToFrontAndExpand`。

### Lines 108-123 / 第 108-123 行

```cpp
0108: }
0109: 
0110: // The algorithm is as follows:
0111: // 1. Figure out what all of the collective levels in `logical_tensors` is.
0112: // 2. Move all batch dims to the front of the tensors and add extra dims
0113: //    of size 1. At this point, every tensor will have a dimension for
0114: //    each of the collective levels.
0115: // 3. Compute the batch_sizes.
0116: // 4. Expand each physical tensor so that they have output batch size equal
0117: //    to `batch_sizes`
0118: VmapPhysicalViewVec
0119: MultiBatchVmapTransform::logicalToPhysical(ITensorListRef logical_tensors) {
0120:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0121:   auto cur_level = maybeCurrentDynamicLayer().value().layerId();
0122:   c10::SymInt bdim_size = -1;
0123: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `logicalToPhysical`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`logicalToPhysical`。

### Lines 124-139 / 第 124-139 行

```cpp
0124:   // Figure out the batch size first
0125:   for (const auto& logical_tensor : logical_tensors) {
0126:     auto* batched = maybeGetBatchedImpl(logical_tensor);
0127:     if (!batched) {
0128:       continue;
0129:     }
0130:     if (batched->level() != cur_level) {
0131:       continue;
0132:     }
0133:     bdim_size = batched->value().sym_size(batched->bdim());
0134:   }
0135:   TORCH_INTERNAL_ASSERT(bdim_size != -1);
0136: 
0137:   std::bitset<kVmapNumLevels> levels;
0138:   levels[cur_level] = true;
0139: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 140-155 / 第 140-155 行

```cpp
0140:   VmapPhysicalViewVec result;
0141:   for (const auto& logical_tensor : logical_tensors) {
0142:     auto* batched = maybeGetBatchedImpl(logical_tensor);
0143:     if (!batched || (batched->level() != cur_level)) {
0144:       // Unsqueeze dim 0, expand it to the correct shape
0145:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0146:       auto value = moveDimToFrontAndExpand(logical_tensor, {}, bdim_size);
0147:       result.emplace_back(std::move(value), levels);
0148:       continue;
0149:     }
0150:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0151:     auto physical = batched->value();
0152:     auto value = moveDimToFrontAndExpand(physical, batched->bdim(), bdim_size);
0153:     result.emplace_back(std::move(value), levels);
0154:   }
0155: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `guard`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`guard`。

### Lines 156-169 / 第 156-169 行

```cpp
0156:   return result;
0157: }
0158: 
0159: static Tensor moveDimToFrontAndUnsqueeze(Tensor tensor, std::optional<int64_t> dim, int64_t example_ndim) {
0160:   if (dim) {
0161:     tensor = tensor.movedim(*dim, 0);
0162:   } else {
0163:     tensor = tensor.unsqueeze(0);
0164:   }
0165:   auto ndim = tensor.dim() - 1;
0166:   for (int64_t i = 0; i < example_ndim - ndim; i++) {
0167:     tensor = tensor.unsqueeze(1);
0168:   }
0169:   return tensor;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `moveDimToFrontAndUnsqueeze`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`moveDimToFrontAndUnsqueeze`。

### Lines 170-184 / 第 170-184 行

```cpp
0170: }
0171: 
0172: VmapPhysicalViewVec BroadcastingVmapTransform::logicalToPhysical(TensorList logical_tensors) {
0173:   auto cur_level = maybeCurrentDynamicLayer().value().layerId();
0174:   int64_t bdim_size = -1;
0175: 
0176:   // Figure out the batch size first
0177:   for (const auto& logical_tensor : logical_tensors) {
0178:     auto* batched = maybeGetBatchedImpl(logical_tensor);
0179:     if (!batched || (batched->level() != cur_level)) {
0180:       continue;
0181:     }
0182:     bdim_size = batched->value().size(batched->bdim());
0183:   }
0184:   TORCH_INTERNAL_ASSERT(bdim_size != -1);
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `logicalToPhysical`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`logicalToPhysical`。

### Lines 185-198 / 第 185-198 行

```cpp
0185: 
0186:   std::bitset<kVmapNumLevels> levels;
0187:   levels[cur_level] = true;
0188: 
0189:   // figure out the example ndim
0190:   int64_t max_example_dim = -1;
0191:   for (const auto& logical_tensor : logical_tensors) {
0192:     max_example_dim = std::max(logical_tensor.dim(), max_example_dim);
0193:   }
0194: 
0195:   VmapPhysicalViewVec result;
0196:   for (const auto& logical_tensor : logical_tensors) {
0197:     auto* batched = maybeGetBatchedImpl(logical_tensor);
0198:     if (!batched || (batched->level() != cur_level)) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 199-213 / 第 199-213 行

```cpp
0199:       // Unsqueeze dim 0, expand it to the correct shape
0200:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0201:       auto value = moveDimToFrontAndUnsqueeze(logical_tensor, {}, max_example_dim);
0202:       result.emplace_back(std::move(value), levels);
0203:       continue;
0204:     }
0205:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0206:     auto physical = batched->value();
0207:     auto value = moveDimToFrontAndUnsqueeze(physical, batched->bdim(), max_example_dim);
0208:     result.emplace_back(std::move(value), levels);
0209:   }
0210: 
0211:   return result;
0212: }
0213: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `guard`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`guard`。

### Lines 214-228 / 第 214-228 行

```cpp
0214: VmapPhysicalToLogicalMap VmapPhysicalView::getPhysicalToLogicalMap() const {
0215:   return VmapPhysicalToLogicalMap(levels_);
0216: }
0217: 
0218: Tensor VmapPhysicalToLogicalMap::apply(const Tensor& physical_tensor) const {
0219:   auto bdim_level = computeFrontBatchDimsFromLevels(levels_);
0220:   return makeBatched(physical_tensor, std::get<0>(bdim_level), std::get<1>(bdim_level));
0221: }
0222: 
0223: void VmapPhysicalToLogicalMap::applyInplace(std::vector<Tensor>& physical_tensors) const {
0224:   for (const auto idx : c10::irange(0, physical_tensors.size())) {
0225:     physical_tensors[idx] = apply(physical_tensors[idx]);
0226:   }
0227: }
0228: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `getPhysicalToLogicalMap`, `VmapPhysicalToLogicalMap`, `apply`, `makeBatched`, `applyInplace`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`getPhysicalToLogicalMap`, `VmapPhysicalToLogicalMap`, `apply`, `makeBatched`, `applyInplace`。

### Lines 229-229 / 第 229-229 行

```cpp
0229: } // namespace at::functorch
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: permuteBatchDimsToFront, permutation, logicalToPhysical, numBatchDims, numLogicalDims, getPhysicalDims, getPhysicalDim, maybe_wrap_dim** — 核心符号：permuteBatchDimsToFront、permutation、logicalToPhysical、numBatchDims、numLogicalDims、getPhysicalDims、getPhysicalDim、maybe_wrap_dim

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/LegacyVmapTransforms.h`, `ATen/functorch/DynamicLayer.h`, `c10/util/irange.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `permuteBatchDimsToFront`, `permutation`, `logicalToPhysical`, `numBatchDims`, `numLogicalDims`, `getPhysicalDims`, `getPhysicalDim`, `maybe_wrap_dim`, `getPhysicalShape`, `computeFrontBatchDimsFromLevels`, `make_tuple`, `moveDimToFrontAndExpand`, `...`
