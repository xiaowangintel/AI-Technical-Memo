# BatchRulesScatterOps.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesScatterOps.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesScatterOps.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Descriptor/handle lifecycle management is important here. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesScatterOps.cpp` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32 / 第 1-32 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/BatchRulesHelper.h>
0008: #include <ATen/Operators.h>
0009: #include <ATen/functorch/PlumbingHelper.h>
0010: #include <ATen/functorch/BatchedFallback.h>
0011: #include <ATen/native/IndexKernel.h>
0012: #include <ATen/native/IndexingUtils.h>
0013: #include <torch/library.h>
0014: #include <c10/util/Exception.h>
0015: 
0016: 
0017: // NOLINTBEGIN(bugprone-unchecked-optional-access)
0018: namespace at::functorch {
0019: 
0020: namespace {
0021: bool any_has_value(ArrayRef<std::optional<int64_t>> bdims) {
0022:   for (const auto& bdim : bdims) {
0023:     if (bdim.has_value()) {
0024:       return true;
0025:     }
0026:   }
0027:   return false;
0028: }
0029: 
0030: int64_t get_num_leading_nones(ArrayRef<std::optional<Tensor>> indices) {
0031:   int64_t result = 0;
0032:   for (const auto& idx : indices) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `any_has_value`, `get_num_leading_nones`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`any_has_value`, `get_num_leading_nones`。

### Lines 33-88 / 第 33-88 行

```cpp
0033:     if (!idx.has_value() || !idx->defined()) {
0034:       result++;
0035:     } else {
0036:       return result;
0037:     }
0038:   }
0039:   return result;
0040: }
0041: 
0042: int64_t get_max_index_logical_dim(
0043:     ArrayRef<std::optional<Tensor>> indices,
0044:     ArrayRef<std::optional<int64_t>> indices_bdims) {
0045:   int64_t max_logical_dim = -1;
0046:   TORCH_INTERNAL_ASSERT(indices.size() == indices_bdims.size());
0047:   TORCH_INTERNAL_ASSERT(!indices.empty());
0048:   for (const auto i : c10::irange(0, indices.size())) {
0049:     const auto& maybe_tensor = indices[i];
0050:     if (!maybe_tensor.has_value() || !maybe_tensor->defined()) {
0051:       continue;
0052:     }
0053:     auto logical_dim = rankWithoutBatchDim(maybe_tensor.value(), indices_bdims[i]);
0054:     max_logical_dim = std::max(logical_dim, max_logical_dim);
0055:   }
0056:   return max_logical_dim;
0057: }
0058: 
0059: std::vector<std::optional<Tensor>> batchIndices(
0060:   at::TensorOptions options,
0061:   ArrayRef<std::optional<Tensor>> indices,
0062:   ArrayRef<std::optional<int64_t>> indices_bdims,
0063:   const c10::SymInt& batch_size,
0064:   std::optional<int64_t> self_bdim,
0065:   std::optional<int64_t> values_bdim = std::nullopt) {
0066:   // There are 3 main cases:
0067:   // 1. self is batched, indices/values are not batched
0068:   // In this case, we just need to augment indices with a None at the front to
0069:   // basically broadcast the indexing across the batch dimension of self.
0070:   //
0071:   // 2. self is not batched, some indices are batched.
0072:   // In this case, we don't need to do anything - indices will automatically
0073:   // broadcast to work with the unbatched self.
0074:   //
0075:   // 3. self is batched, some indices are batched.
0076:   // In this case, we simply need to add an arange that indexes along the first
0077:   // dimension (i.e. the batch dimension). We also need to make sure this
0078:   // broadcasts with the rest of the indices.
0079:   //
0080:   // In all three cases, depending on if advanced indices are adjacent we will
0081:   // have to permute the output.
0082:   // See NOTE: [advanced indexing (index.Tensor) batch rule] for more details
0083:   //
0084:   // There is one more case worth mentioning - boolean tensor indices. If we
0085:   // have "batched" boolean tensor indices, that is unrepresentable, as each
0086:   // batch would result in a tensor with different values.
0087:   std::vector<std::optional<Tensor>> indices_;
0088: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `get_max_index_logical_dim`, `batchIndices`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`get_max_index_logical_dim`, `batchIndices`。

### Lines 89-123 / 第 89-123 行

```cpp
0089:   int64_t maxLogicalRank = get_max_index_logical_dim(indices, indices_bdims);
0090:   bool indices_batched = any_has_value(indices_bdims);
0091: 
0092:   for (size_t i = 0; i < indices.size(); i++) {
0093:     auto const & index = indices[i];
0094:     if (index.has_value() && index->sym_numel() != 0) {
0095:       const auto idx_bdim = indices_bdims[i];
0096:       indices_.emplace_back(maybePadToLogicalRank(moveBatchDimToFront(index.value(), idx_bdim), idx_bdim, maxLogicalRank));
0097:       TORCH_CHECK(
0098:         !(index.value().dtype() == kBool) || !indices_bdims[i].has_value(),
0099:         "vmap: We do not support batching operators that can support dynamic shape. Attempting to batch over indexing with a boolean mask."
0100:       );
0101:     } else {
0102:       indices_.push_back(index);
0103:     }
0104:   }
0105: 
0106:   auto maxIndexDim = maxLogicalRank;
0107:   if (indices_batched || values_bdim.has_value()) {
0108:     maxIndexDim += 1;
0109:   }
0110: 
0111:   if (!indices_batched && self_bdim.has_value()) {
0112:     indices_.insert(indices_.begin(), std::nullopt);
0113:   } else if (indices_batched && !self_bdim.has_value()) {
0114:     // do nothing
0115:   } else if (indices_batched && (self_bdim.has_value() || values_bdim.has_value())) {
0116:     auto arange_index = at::arange(batch_size, options.dtype(kLong));
0117:     while (arange_index.dim() < maxIndexDim) {
0118:       arange_index = arange_index.unsqueeze(-1);
0119:     }
0120:     // TODO: this is O(N)
0121:     indices_.insert(indices_.begin(), arange_index);
0122:   }
0123:   return indices_;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 124-153 / 第 124-153 行

```cpp
0124: }
0125: 
0126: // Define an "advanced index" to be a selection object that is
0127: // a non-trivial Tensor (i.e. it does not represent :).
0128: bool is_advanced_index(const std::optional<Tensor>& idx) {
0129:   if (!idx.has_value()) {
0130:     return false;
0131:   }
0132:   if (!idx->defined()) {
0133:     return false;
0134:   }
0135:   return true;
0136: }
0137: 
0138: // See NOTE: [advanced indices adjacent] for definition
0139: bool are_advanced_indices_adjacent(ArrayRef<std::optional<Tensor>> indices) {
0140:   int64_t num_advanced_indices_regions = 0;
0141:   bool in_advanced_indices_region = false;
0142:   for (const auto& idx : indices) {
0143:     if (!in_advanced_indices_region && is_advanced_index(idx)) {
0144:       num_advanced_indices_regions++;
0145:       in_advanced_indices_region = true;
0146:       continue;
0147:     }
0148:     if (in_advanced_indices_region && !is_advanced_index(idx)) {
0149:       in_advanced_indices_region = false;
0150:       continue;
0151:     }
0152:   }
0153:   return num_advanced_indices_regions <= 1;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `is_advanced_index`, `are_advanced_indices_adjacent`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`is_advanced_index`, `are_advanced_indices_adjacent`。

### Lines 154-213 / 第 154-213 行

```cpp
0154: }
0155: 
0156: // Given a Tensor[B, <first_region>, <second_region>, ...]
0157: // Swaps the regions to produce Tensor[B, <second_region>, <first_region>, ...]
0158: //
0159: // Concretely speaking, given
0160: // - tensor: Tensor[B, 2, 3, 4, 5, 6, 7, 8]
0161: // - first_region_size: 2
0162: // - second_region_size: 3
0163: // Produces:
0164: // - result: Tensor[B, 4, 5, 6, 2, 3, 7, 8]
0165: //                     -------  ----
0166: //                     region2  region1
0167: Tensor swap_regions(const Tensor& tensor, int64_t first_region_size, int64_t second_region_size) {
0168:   VmapDimVector permutation(tensor.dim(), 0);
0169:   std::iota(permutation.begin(), permutation.end(), 0);
0170:   std::rotate(
0171:       permutation.begin() + 1,
0172:       permutation.begin() + 1 + first_region_size,
0173:       permutation.begin() + 1 + first_region_size + second_region_size);
0174:   return tensor.permute(permutation);
0175: }
0176: 
0177: std::tuple<Tensor, std::optional<int64_t>> index_batch_rule(
0178:     const Tensor& self,
0179:     std::optional<int64_t> self_bdim,
0180:     ArrayRef<std::optional<Tensor>> indices,
0181:     ArrayRef<std::optional<int64_t>> indices_bdims) {
0182: 
0183:   // NOTE: [advanced indexing (index.Tensor) batch rule]
0184:   //
0185:   // This is a three step procedure:
0186:   // 1. batch `indices`. Depends on self_bdim and indices_bdim.
0187:   // 2. call at::index
0188:   // 3. (maybe) reorder the dimensions in the result.
0189:   // Why is step 3 necessary? Let's take a detour first.
0190:   //
0191:   // NOTE: [advanced indices adjacent]
0192:   // Definition: In a list of std::optional<Tensor> indices,
0193:   // we say that "advanced indices are adjacent" if ALL advanced indices are
0194:   // not separated by a None (slice).
0195:   //
0196:   // So, for example,
0197:   // [:, :, (0, 1), (0, 1), :] -> True
0198:   // [:, (0, 1), :, (0, 1), :] -> False, the advanced indices are separated by a slice
0199:   //
0200:   // See https://numpy.org/doc/stable/user/basics.indexing.html#combining-advanced-and-basic-indexing
0201:   // for more details.
0202:   //
0203:   // NOTE: [Why is step 3 necessary?]
0204:   //
0205:   // In the original self[*indices] expression,
0206:   // depending on whether or not the "advanced indices inside `indices` are
0207:   // adjacent", something different happens.
0208:   //
0209:   // For example:
0210:   // - self: Tensor[4, 5, 6, 7]
0211:   // - indices: [:, (0, 1), (0, 1), :] (advanced indices are adjacent)
0212:   // - self[*indices]: Tensor[4, 2, 7]
0213:   // If advanced indices are adjacent, you get the output you would expect.
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `swap_regions`, `permutation`, `iota`, `rotate`, `index_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`swap_regions`, `permutation`, `iota`, `rotate`, `index_batch_rule`。

### Lines 214-244 / 第 214-244 行

```cpp
0214:   // (0, 1), (0, 1) says "please index these two dimensions at (0, 0) and (1, 1)
0215:   // to produce two elements".
0216:   //
0217:   // If advanced indices are not adjacent, it is ambiguous to where the new
0218:   // dimension of size 2 should go. The numpy spec says it should go at the very
0219:   // front of the Tensor.
0220:   //
0221:   // - self: Tensor[4, 5, 6, 7]
0222:   // - indices: [:, (0, 1), :, (0, 1)] (advanced indices not adjacent)
0223:   // - self[*indices]: Tensor[2, 4, 6]
0224:   //
0225:   // Now, this leads to some weird interactions with vmap.
0226:   // The indices might originally have adjacent advanced indices, but after
0227:   // batching them with "batchIndices", they may no longer be adjacent!
0228:   // - indices: [:, (0, 1), (0, 1)]
0229:   // - batched_indices (for example): [(0, 1), :, (0, 1), (0, 1)]
0230:   // This leads to the dimension of size 2 appearing somewhere else.
0231:   //
0232:   // There are a couple of different cases that we walk through in the code below.
0233:   //
0234:   // Background reading for why we care about if the advanced indices are adjacent:
0235:   // https://numpy.org/doc/stable/user/basics.indexing.html#combining-advanced-and-basic-indexing
0236:   auto self_ = moveBatchDimToFront(self, self_bdim);
0237:   TORCH_INTERNAL_ASSERT(indices.size() == indices_bdims.size());
0238:   bool advanced_indices_are_adjacent = are_advanced_indices_adjacent(indices);
0239: 
0240:   // Step 1
0241:   const auto batched_indices = batchIndices(self.options(), indices, indices_bdims, self_.sym_size(0), self_bdim);
0242:   auto num_leading_nones = get_num_leading_nones(indices);
0243:   auto max_index_dim = get_max_index_logical_dim(indices, indices_bdims);
0244: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 245-278 / 第 245-278 行

```cpp
0245:   // Step 2
0246:   auto res = at::index(self_, List<std::optional<Tensor>>(batched_indices));
0247: 
0248:   // Step 3: There are three cases (these match the cases outlined in batchIndices)
0249:   bool self_batched = self_bdim.has_value();
0250:   bool indices_batched = any_has_value(indices_bdims);
0251: 
0252:   TORCH_INTERNAL_ASSERT(self_batched || indices_batched, "Requires at least one batched to get here");
0253: 
0254:   // Case 1
0255:   if (self_batched && !indices_batched) {
0256:     if (advanced_indices_are_adjacent) {
0257:       // self: Tensor[B, 5, 6, 7, 8]
0258:       // indices: [:, Tensor[2, 2], Tensor[2, 2], :]
0259:       // batched_indices: [:, :, Tensor[2, 2], Tensor[2, 2], :]
0260:       // res: Tensor[B, 5, 2, 2, 8]
0261:       return std::make_tuple(res, 0);
0262:     } else {
0263:       // self: Tensor[B, 5, 6, 7]
0264:       // indices: [Tensor[2, 2], :, Tensor[2, 2]]
0265:       // batched_indices: [:, Tensor[2, 2], :, Tensor[2, 2]]
0266:       // res: Tensor[2, 2, B, 6]
0267:       return std::make_tuple(res, max_index_dim);
0268:     }
0269:   }
0270: 
0271:   // Case 2
0272:   if (!self_batched && indices_batched) {
0273:     if (advanced_indices_are_adjacent) {
0274:       // self: Tensor[5, 6, 7, 8]
0275:       // indices: [:, :, Tensor[B, 2, 2], Tensor[2, 2]]
0276:       // batched_indices: indices (no change)
0277:       // res: Tensor[5, 6, B, 2, 2]
0278:       return std::make_tuple(res, num_leading_nones);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`make_tuple`。

### Lines 279-318 / 第 279-318 行

```cpp
0279:     } else {
0280:       // self: Tensor[5, 6, 7, 8, 9]
0281:       // indices: [:, :, Tensor[B, 2, 2], :, Tensor[2, 2]]
0282:       // batched_indices: indices (no change)
0283:       // res: Tensor[B, 2, 2, 5, 6, 8]
0284:       return std::make_tuple(res, 0);
0285:     }
0286:   }
0287: 
0288:   // Case 3: self_batched and indices_batched
0289:   TORCH_INTERNAL_ASSERT(self_batched && indices_batched);
0290:   if (!advanced_indices_are_adjacent) {
0291:     // self: Tensor[B, 5, 6, 7, 8]
0292:     // indices: [:, Tensor[B, 2, 2], :, Tensor[2, 2]]
0293:     // batched_indices: [arange(B).expand(B, 2, 2), :, Tensor[B, 2, 2], :, Tensor[2, 2]]
0294:     // res: Tensor[B, 2, 2, 5, 7]
0295:     return std::make_tuple(res, 0);
0296:   }
0297:   // In other words, in batched_indices, advanced indices are adjacent
0298:   if (num_leading_nones == 0) {
0299:     // self: Tensor[B, 5, 6, 7, 8]
0300:     // indices: [Tensor[B, 2, 2], Tensor[2, 2], :, :]
0301:     // batched_indices: [arange(B).expand(B, 2, 2), Tensor[B, 2, 2], Tensor[2, 2], :, :]
0302:     // res: Tensor[B, 2, 2, 7, 8]
0303:     return std::make_tuple(res, 0);
0304:   }
0305:   // This is the tricky case. In indices, advanced indices are adjacent.
0306:   // In batched_indices, advanced indices are no longer adjacent
0307:   //
0308:   // self: Tensor[B, 5, 6, 7, 8, 9]
0309:   // indices: [:, :, Tensor[B, 2, 3], Tensor[2, 3], :]
0310:   // batched_indices: [arange(B).expand(B, 2, 3), :, :, Tensor[B, 2, 3], Tensor[2, 3], :]
0311:   // res: Tensor[B, 2, 3, 5, 6, 9]
0312:   // expected: Tensor[B, 5, 6, 2, 3, 9]
0313:   //
0314:   // The resolution is to move dims around until we get the right shape.
0315:   // The result is set up as [B, <maxIndexDim>, <leading_nones>, ...]
0316:   // we just have to move the <leading_nones> to before the <maxIndexDim> to produce
0317:   // [B, <leading_nones>, <maxIndexDim>, ...]
0318:   return std::make_tuple(swap_regions(res, max_index_dim, num_leading_nones), 0);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`make_tuple`。

### Lines 319-348 / 第 319-348 行

```cpp
0319: }
0320: 
0321: // plumbing done since we don't support List<std::optional<Tensor>> in codegen
0322: Tensor index_plumbing(const Tensor & self, const List<std::optional<Tensor>> & indices
0323: ) {
0324:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0325:   auto maybe_layer = maybeCurrentDynamicLayer();
0326:   vmap_check_escaped(maybe_layer, "index_plumbing");
0327:   int64_t cur_level = maybe_layer->layerId();
0328:   if (!isBatchedAtLevel(self, cur_level) && !isBatchedAtLevel(indices, cur_level)) {
0329:     return at::index(self, indices);
0330:   }
0331:   auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0332:   std::vector<std::optional<Tensor>> indices_value;
0333:   std::vector<std::optional<int64_t>> indices_bdims;
0334:   for (const auto&& indRef : indices) {
0335:       std::optional<Tensor> ind = indRef;
0336:       std::optional<Tensor> index;
0337:       std::optional<int64_t> index_bdim;
0338:       if (ind.has_value()) {
0339:         std::tie(index, index_bdim) = unwrapTensorAtLevel(ind.value(), cur_level);
0340:       }
0341:     indices_value.push_back(index);
0342:     indices_bdims.push_back(index_bdim);
0343:   }
0344:   auto results = index_batch_rule(self_value, self_bdim, indices_value, indices_bdims);
0345:   return makeBatched(std::get<0>(results), std::get<1>(results), cur_level);
0346: }
0347: 
0348: namespace {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `index_plumbing`, `guard`, `vmap_check_escaped`, `index`, `tie`, `makeBatched`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`index_plumbing`, `guard`, `vmap_check_escaped`, `index`, `tie`, `makeBatched`。

### Lines 349-390 / 第 349-390 行

```cpp
0349:   // Code is mostly duplicated from
0350:   // https://github.com/pytorch/pytorch/blob/fb0e27d38a8fdab4e1c14d6378c9e41cb30fd6a3
0351:   // /aten/src/ATen/native/TensorAdvancedIndexing.cpp#L294-L312
0352:   VmapSymDimVector compute_indexed_shape(const Tensor &src, TensorList indices_list)
0353:   {
0354:     int64_t dims_before = 0, dims_indexed = 0;
0355:     SymIntArrayRef replacement_shape;
0356:     for (const auto dim : c10::irange(indices_list.size())) {
0357:       if (!indices_list[dim].defined()) {
0358:         if (dims_indexed == 0) {
0359:           dims_before++;
0360:         }
0361:       } else {
0362:         dims_indexed++;
0363:         replacement_shape = indices_list[dim].sym_sizes();
0364:       }
0365:     }
0366: 
0367:     // Replace indexed dimensions in src with stride 0 and the size of the result tensor.
0368:     // The offset in these dimensions is computed by the kernel using the index tensor's
0369:     // values and the stride of src. The new shape is not meaningful. It's used to make
0370:     // the shape compatible with the result tensor.
0371:     auto shape = VmapSymDimVector(src.sym_sizes());
0372:     int64_t end = dims_before + dims_indexed;
0373:     shape.erase(shape.begin() + dims_before, shape.begin() + end);
0374:     shape.insert(shape.begin() + dims_before, replacement_shape.begin(), replacement_shape.end());
0375:     return shape;
0376:   }
0377: 
0378:   // Code is mostly duplicated from
0379:   // https://github.com/pytorch/pytorch/blob/fb0e27d38a8fdab4e1c14d6378c9e41cb30fd6a3
0380:   // /aten/src/ATen/native/TensorAdvancedIndexing.cpp#L379-L405
0381:   VmapSymDimVector get_indexed_shape(Tensor self, const torch::List<std::optional<at::Tensor>> &orig)
0382:   {
0383:     at::native::checkIndexTensorTypes(orig, /*allow_int*/ true);
0384:     // first expand BoolTensor (masks) or ByteTensor (masks) into 1 or more LongTensors
0385:     auto indices = at::native::expandTensors(self, orig);
0386:     // next broadcast all index tensors together
0387:     try {
0388:       indices = at::expand_outplace(indices);
0389:     } catch (std::exception&) {
0390:       TORCH_CHECK_INDEX(
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `compute_indexed_shape`, `get_indexed_shape`, `checkIndexTensorTypes`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`compute_indexed_shape`, `get_indexed_shape`, `checkIndexTensorTypes`。

### Lines 391-423 / 第 391-423 行

```cpp
0391:           false,
0392:           "shape mismatch: indexing tensors could not be broadcast together"
0393:           " with shapes ");
0394:     }
0395:     // add missing null Tensors so that it matches self.dim()
0396:     while (indices.size() < static_cast<size_t>(self.dim())) {
0397:       indices.emplace_back();
0398:     }
0399:     // if the non-null indices are not all adjacent, transpose self and indices
0400:     // together so that they're adjacent at the front
0401:     if (!at::native::hasContiguousSubspace(indices)) {
0402:       std::tie(self, indices) = at::native::transposeToFront(self, indices);
0403:     }
0404:     return compute_indexed_shape(self, indices);
0405:   }
0406: 
0407:   std::tuple<Tensor, std::vector<std::optional<Tensor>>, Tensor>
0408:   index_put_batch_rule_helper(const Tensor &self,
0409:                               std::optional<int64_t> self_bdim,
0410:                               ArrayRef<std::optional<Tensor>> indices,
0411:                               ArrayRef<std::optional<int64_t>> indices_bdims,
0412:                               const Tensor &values,
0413:                               std::optional<int64_t> values_bdim,
0414:                               std::optional<c10::SymInt> opt_batch_size = {}) {
0415: 
0416:     Tensor self_ = moveBatchDimToFront(self, self_bdim);
0417:     Tensor values_ = moveBatchDimToFront(values, values_bdim);
0418:     // for inplace variants `index_put_` and `_index_put_impl_` we find the batch_size
0419:     // here while for `index_put` does it outside of this function.
0420:     const auto batch_size = opt_batch_size ? opt_batch_size.value() : self_.sym_size(0);
0421:     self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0422:     values_ = ensure_has_bdim(values_, values_bdim.has_value(), batch_size);
0423:     TORCH_INTERNAL_ASSERT(indices.size() == indices_bdims.size());
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `tie`, `compute_indexed_shape`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`tie`, `compute_indexed_shape`。

### Lines 424-455 / 第 424-455 行

```cpp
0424: 
0425:     // we've already made sure that self has bdim at 0.
0426:     const auto indices_ = batchIndices(self.options(), indices, indices_bdims, batch_size, /*self_bdim=*/0, values_bdim);
0427: 
0428:     auto indexed_shape = get_indexed_shape(self_, List<std::optional<Tensor>>(indices_));
0429: 
0430:     // handle broadcasting support for values
0431:     // Eg. Given `indexed_shape.size()` is 5 and
0432:     // shape of `values` is (N, 2, 3), then following block
0433:     // will reshape `values` to (N, 1, 1, 2, 3).
0434:     if ( static_cast<int64_t>(indexed_shape.size()) > values_.dim()) {
0435:       auto values_sizes = values_.sym_sizes();
0436: 
0437:       // number of unit dims (for broadcasting value to indexed_shape)
0438:       auto n_unit_dims = indexed_shape.size() - values_sizes.size();
0439:       VmapSymDimVector new_values_shape(values_sizes.size() + n_unit_dims);
0440: 
0441:       // add the batch-dim
0442:       new_values_shape[0] = batch_size;
0443: 
0444:       // insert the unit dims for broadcasting.
0445:       for (const auto idx : c10::irange(n_unit_dims)) {
0446:         // since batch-dim is already be filled.
0447:         new_values_shape[idx + 1] = 1;
0448:       }
0449:       for (const auto idx: c10::irange(1, values_sizes.size())) {
0450:         // since batch and unit dims are already be filled.
0451:         new_values_shape[idx + n_unit_dims] = values_sizes[idx];
0452:       }
0453:       values_ = values_.view_symint(new_values_shape);
0454:     }
0455: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `new_values_shape`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`new_values_shape`。

### Lines 456-491 / 第 456-491 行

```cpp
0456:     return std::make_tuple(self_, indices_, values_);
0457:   }
0458: 
0459:   auto unpackSelfAndIndicesAndValuesAtCurrentLevel(const Tensor &self,
0460:                                                    const List<std::optional<Tensor>> &indices,
0461:                                                    const Tensor &values, int64_t cur_level)
0462:   {
0463:     auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0464:     std::vector<std::optional<Tensor>> indices_value;
0465:     std::vector<std::optional<int64_t>> indices_bdims;
0466:     for (const auto &&indRef : indices)
0467:     {
0468:       std::optional<Tensor> ind = indRef;
0469:       std::optional<Tensor> index;
0470:       std::optional<int64_t> index_bdim;
0471:       if (ind.has_value()) {
0472:         std::tie(index, index_bdim) = unwrapTensorAtLevel(ind.value(), cur_level);
0473:       }
0474:       indices_value.push_back(index);
0475:       indices_bdims.push_back(index_bdim);
0476:     }
0477:     auto [values_value, values_bdim] = unwrapTensorAtLevel(values, cur_level);
0478:     return std::make_tuple(self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim);
0479:   }
0480: 
0481: }  // namespace
0482: 
0483: void index_put__batch_rule(
0484:     const Tensor& self,
0485:     std::optional<int64_t> self_bdim,
0486:     ArrayRef<std::optional<Tensor>> indices,
0487:     ArrayRef<std::optional<int64_t>> indices_bdims,
0488:     const Tensor& values,
0489:     std::optional<int64_t> values_bdim,
0490:     bool accumulate) {
0491:   if (!self_bdim.has_value()) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `tie`, `index_put__batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `tie`, `index_put__batch_rule`。

### Lines 492-521 / 第 492-521 行

```cpp
0492:     vmapIncompatibleInplaceError("index_put_");
0493:   }
0494:   auto [self_, indices_, values_] = index_put_batch_rule_helper(
0495:       self, self_bdim, indices, indices_bdims, values, values_bdim);
0496:   at::index_put_(self_, List<std::optional<Tensor>>(indices_), values_, accumulate);
0497: }
0498: 
0499: // plumbing done since we don't support List<std::optional<Tensor>> in codegen
0500: Tensor& index_put__plumbing(Tensor & self, const List<std::optional<Tensor>> & indices
0501: , const Tensor & values, bool accumulate) {
0502:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0503:   auto maybe_layer = maybeCurrentDynamicLayer();
0504:   vmap_check_escaped(maybe_layer, "index_put__plumbing");
0505:   int64_t cur_level = maybe_layer->layerId();
0506: 
0507:   // on device mismatch, we can move 0d tensors to self device
0508:   auto values_ = values;
0509:   if (values.device() != self.device() && values.numel() == 1 && values.dim() == 0) {
0510:     values_ = values.to(self.device());
0511:   }
0512: 
0513:   if (!isBatchedAtLevel(self, cur_level) && !isBatchedAtLevel(indices, cur_level) && !isBatchedAtLevel(values_, cur_level)) {
0514:     return self.index_put_(indices, values_, accumulate);
0515:   }
0516:   auto [self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim] =
0517:       unpackSelfAndIndicesAndValuesAtCurrentLevel(self, indices, values_, cur_level);
0518:   index_put__batch_rule(self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim, accumulate);
0519:   return self;
0520: }
0521: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `vmapIncompatibleInplaceError`, `index_put_`, `index_put__plumbing`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `index_put__batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`vmapIncompatibleInplaceError`, `index_put_`, `index_put__plumbing`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `index_put__batch_rule`。

### Lines 522-552 / 第 522-552 行

```cpp
0522: void _index_put_impl__batch_rule(
0523:     const Tensor& self,
0524:     std::optional<int64_t> self_bdim,
0525:     ArrayRef<std::optional<Tensor>> indices,
0526:     ArrayRef<std::optional<int64_t>> indices_bdims,
0527:     const Tensor& values,
0528:     std::optional<int64_t> values_bdim,
0529:     bool accumulate,
0530:     bool unsafe) {
0531:   if (!self_bdim.has_value()) {
0532:     vmapIncompatibleInplaceError("_index_put_impl_");
0533:   }
0534:   auto [self_, indices_, values_] = index_put_batch_rule_helper(
0535:       self, self_bdim, indices, indices_bdims, values, values_bdim);
0536:   at::_index_put_impl_(self_, List<std::optional<Tensor>>(indices_), values_, accumulate, unsafe);
0537: }
0538: 
0539: // plumbing done since we don't support List<std::optional<Tensor>> in codegen
0540: Tensor &_index_put_impl__plumbing(Tensor &self, const List<std::optional<Tensor>> &indices,
0541:                                   const Tensor &values, bool accumulate, bool unsafe) {
0542:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0543:   auto maybe_layer = maybeCurrentDynamicLayer();
0544:   vmap_check_escaped(maybe_layer, "_index_put_impl__plumbing");
0545:   int64_t cur_level = maybe_layer->layerId();
0546:   if (!isBatchedAtLevel(self, cur_level) && !isBatchedAtLevel(indices, cur_level) && !isBatchedAtLevel(values, cur_level)) {
0547:     return at::_index_put_impl_(self, indices, values, accumulate, unsafe);
0548:   }
0549:   auto [self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim] =
0550:       unpackSelfAndIndicesAndValuesAtCurrentLevel(self, indices, values, cur_level);
0551:   _index_put_impl__batch_rule(self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim, accumulate, unsafe);
0552:   return self;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `_index_put_impl__batch_rule`, `vmapIncompatibleInplaceError`, `_index_put_impl_`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`_index_put_impl__batch_rule`, `vmapIncompatibleInplaceError`, `_index_put_impl_`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`。

### Lines 553-584 / 第 553-584 行

```cpp
0553: }
0554: 
0555: Tensor maybe_permute_values(
0556:     const Tensor& values,
0557:     ArrayRef<std::optional<Tensor>> orig_indices,
0558:     ArrayRef<std::optional<int64_t>> orig_indices_bdims) {
0559:   bool indices_batched = any_has_value(orig_indices_bdims);
0560:   bool advanced_indices_are_adjacent = are_advanced_indices_adjacent(orig_indices);
0561:   auto num_leading_nones = get_num_leading_nones(orig_indices);
0562:   auto max_index_dim = get_max_index_logical_dim(orig_indices, orig_indices_bdims);
0563: 
0564:   // NB: values has its B dimension at the front
0565:   if (!indices_batched) {
0566:     if (advanced_indices_are_adjacent) {
0567:       // self: Tensor[B, 5, 6, 7, 8]
0568:       // indices: [:, Tensor[2, 2], Tensor[2, 2], :]
0569:       // batched_indices: [:, :, Tensor[2, 2], Tensor[2, 2], :]
0570:       // required values: Tensor[B, 5, 2, 2, 8]
0571:       return values;
0572:     }
0573:     // self: Tensor[B, 5, 6, 7]
0574:     // indices: [Tensor[2, 2], :, Tensor[2, 2]]
0575:     // batched_indices: [:, Tensor[2, 2], :, Tensor[2, 2]]
0576:     // required values: Tensor[2, 2, B, 6]
0577:     return values.movedim(0, max_index_dim);
0578:   }
0579:   if (!advanced_indices_are_adjacent) {
0580:     // self: Tensor[B, 5, 6, 7, 8]
0581:     // indices: [:, Tensor[B, 2, 2], :, Tensor[2, 2]]
0582:     // batched_indices: [arange(B).expand(B, 2, 2), :, Tensor[B, 2, 2], :, Tensor[2, 2]]
0583:     // required values: Tensor[B, 2, 2, 5, 7]
0584:     return values;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `maybe_permute_values`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`maybe_permute_values`。

### Lines 585-618 / 第 585-618 行

```cpp
0585:   }
0586:   // In other words, in batched_indices, advanced indices are adjacent
0587:   if (num_leading_nones == 0) {
0588:     // self: Tensor[B, 5, 6, 7, 8]
0589:     // indices: [Tensor[B, 2, 2], Tensor[2, 2], :, :]
0590:     // batched_indices: [arange(B).expand(B, 2, 2), Tensor[B, 2, 2], Tensor[2, 2], :, :]
0591:     // required values: Tensor[B, 2, 2, 7, 8]
0592:     return values;
0593:   }
0594:   // This is the tricky case. In indices, advanced indices are adjacent.
0595:   // In batched_indices, advanced indices are no longer adjacent
0596:   //
0597:   // self: Tensor[B, 5, 6, 7, 8, 9]
0598:   // indices: [:, :, Tensor[B, 2, 3], Tensor[2, 3], :]
0599:   // batched_indices: [arange(B).expand(B, 2, 3), :, :, Tensor[B, 2, 3], Tensor[2, 3], :]
0600:   // required values: Tensor[B, 2, 3, 5, 6, 9]
0601:   // actual values: Tensor[B, 5, 6, 2, 3, 9]
0602:   //
0603:   // The resolution is to move dims around until we get the right shape.
0604:   // The values is set up as [B, <leading_nones>, <maxIndexDim>, ...]
0605:   // we just have to move the <maxIndexDim> to before the <leading_nones> to produce
0606:   // [B, <maxIndexDim>, <leading_nones>, ...]
0607:   return swap_regions(values, num_leading_nones, max_index_dim);
0608: }
0609: 
0610: std::tuple<Tensor, std::optional<int64_t>> index_put_batch_rule(
0611:     const Tensor& self,
0612:     std::optional<int64_t> self_bdim,
0613:     ArrayRef<std::optional<Tensor>> indices,
0614:     ArrayRef<std::optional<int64_t>> indices_bdims,
0615:     const Tensor& values,
0616:     std::optional<int64_t> values_bdim,
0617:     bool accumulate) {
0618:   TORCH_INTERNAL_ASSERT(indices.size() == indices_bdims.size());
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `swap_regions`, `index_put_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`swap_regions`, `index_put_batch_rule`。

### Lines 619-648 / 第 619-648 行

```cpp
0619: 
0620:   // find the batch_size
0621:   c10::SymInt batch_size = 0;
0622:   if (self_bdim || values_bdim) {
0623:     batch_size = get_bdim_size2_symint(self, self_bdim, values, values_bdim);
0624:   } else {
0625:     // one or more of the indices is batched.
0626:     for (size_t i = 0; i < indices.size(); i++) {
0627:       if (indices_bdims[i] && indices[i].has_value()) {
0628:         batch_size = indices[i].value().sym_size(*indices_bdims[i]);
0629:         break;
0630:       }
0631:     }
0632:   }
0633: 
0634:   auto [self_, indices_, values_] = index_put_batch_rule_helper(
0635:       self, self_bdim, indices, indices_bdims, values, values_bdim, batch_size);
0636: 
0637:   // Why do we need to permute values?
0638:   // See NOTE [Advanced indexing (index.Tensor) batch rule] for details,
0639:   // but the gist is that index_put effectively does the following:
0640:   // - result = self_.clone()
0641:   // - result[indices_] = values
0642:   // - return result
0643:   // Now, the problem is, result[indices_] might return a Tensor whose shape is
0644:   // the shape of values, but permuted. This is because the shape of result[indices_]
0645:   // depends on if the original indices "have adjacent advanced indices"
0646:   // and the batched `indices_` might change the "have adjacent advanced indices" property
0647:   values_ = maybe_permute_values(values_, indices, indices_bdims);
0648: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 649-678 / 第 649-678 行

```cpp
0649:   auto result = at::index_put(self_, List<std::optional<Tensor>>(indices_), values_, accumulate);
0650:   return std::make_tuple(std::move(result), 0);
0651: }
0652: 
0653: // plumbing done since we don't support List<std::optional<Tensor>> in codegen
0654: Tensor index_put_plumbing(const Tensor & self, const List<std::optional<Tensor>> & indices,
0655:                           const Tensor & values, bool accumulate) {
0656:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0657:   auto maybe_layer = maybeCurrentDynamicLayer();
0658:   vmap_check_escaped(maybe_layer, "index_put_plumbing");
0659:   int64_t cur_level = maybe_layer->layerId();
0660: 
0661:   // on device mismatch, we can move 0d tensors to self device
0662:   auto values_ = values;
0663:   if (values.device() != self.device() && values.numel() == 1 && values.dim() == 0) {
0664:     values_ = values.to(self.device());
0665:   }
0666: 
0667:   if (!isBatchedAtLevel(self, cur_level) && !isBatchedAtLevel(indices, cur_level) && !isBatchedAtLevel(values_, cur_level)) {
0668:     return self.index_put(indices, values_, accumulate);
0669:   }
0670:   auto [self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim] =
0671:       unpackSelfAndIndicesAndValuesAtCurrentLevel(self, indices, values_, cur_level);
0672:   auto results = index_put_batch_rule(self_value, self_bdim, indices_value, indices_bdims, values_value, values_bdim, accumulate);
0673:   return makeBatched(std::get<0>(results), std::get<1>(results), cur_level);
0674: }
0675: 
0676: namespace {
0677: 
0678: template<typename Func, typename ...Args>
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesScatterOps`. Key symbols: `make_tuple`, `index_put_plumbing`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `makeBatched`.
- **CN:** 围绕 `BatchRulesScatterOps` 构建可复用的模板或辅助层。关键符号：`make_tuple`, `index_put_plumbing`, `guard`, `vmap_check_escaped`, `unpackSelfAndIndicesAndValuesAtCurrentLevel`, `makeBatched`。

### Lines 679-709 / 第 679-709 行

```cpp
0679: std::tuple<Tensor, std::optional<int64_t>> scatter_batch_rule(
0680:     Func f,
0681:     const Tensor& self, std::optional<int64_t> self_bdim,
0682:     int64_t dim,
0683:     const Tensor& index, std::optional<int64_t> index_bdim,
0684:     const Scalar& value, Args... args) {
0685:   auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0686:   auto index_logical_rank = rankWithoutBatchDim(index, index_bdim);
0687:   auto batch_size = get_bdim_size2(self, self_bdim, index, index_bdim);
0688: 
0689:   auto self_ = moveBatchDimToFront(self, self_bdim);
0690:   auto index_ = moveBatchDimToFront(index, index_bdim);
0691: 
0692:   if (self_logical_rank == 0) {
0693:     self_ = self_.unsqueeze(-1);
0694:   }
0695:   if (index_logical_rank == 0) {
0696:     index_ = index_.unsqueeze(-1);
0697:   }
0698:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0699:   index_ = ensure_has_bdim(index_, index_bdim.has_value(), batch_size);
0700:   auto physical_dim = getPhysicalDim(self_, /*has_batch_dim*/true, dim);
0701: 
0702:   auto result = f(self_, physical_dim, index_, value, args...);
0703:   // result should have same shape as self
0704:   if (self_logical_rank == 0) {
0705:     result = result.squeeze(-1);
0706:   }
0707:   return std::make_tuple(std::move(result), 0);
0708: }
0709: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `scatter_batch_rule`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`scatter_batch_rule`, `make_tuple`。

### Lines 710-739 / 第 710-739 行

```cpp
0710: template <typename Func, typename ...Args>
0711: inline std::tuple<Tensor, std::optional<int64_t>> scatter_batch_rule(
0712:     Func f,
0713:     const Tensor& self, std::optional<int64_t> self_bdim,
0714:     int64_t dim,
0715:     const Tensor& index, std::optional<int64_t> index_bdim,
0716:     const Tensor& src, std::optional<int64_t> src_bdim, Args... args) {
0717:   auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0718:   auto index_logical_rank = rankWithoutBatchDim(index, index_bdim);
0719:   auto src_logical_rank = rankWithoutBatchDim(src, src_bdim);
0720:   auto batch_size = get_bdim_size3(self, self_bdim, index, index_bdim, src, src_bdim);
0721: 
0722:   auto self_ = moveBatchDimToFront(self, self_bdim);
0723:   auto index_ = moveBatchDimToFront(index, index_bdim);
0724:   auto src_ = moveBatchDimToFront(src, src_bdim);
0725: 
0726:   if (self_logical_rank == 0) {
0727:     self_ = self_.unsqueeze(-1);
0728:   }
0729:   if (index_logical_rank == 0) {
0730:     index_ = index_.unsqueeze(-1);
0731:   }
0732:   if (src_logical_rank == 0) {
0733:     src_ = src_.unsqueeze(-1);
0734:   }
0735:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0736:   index_ = ensure_has_bdim(index_, index_bdim.has_value(), batch_size);
0737:   src_ = ensure_has_bdim(src_, src_bdim.has_value(), batch_size);
0738:   auto physical_dim = getPhysicalDim(self_, /*has_batch_dim*/true, dim);
0739: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesScatterOps`. Key symbols: `scatter_batch_rule`.
- **CN:** 围绕 `BatchRulesScatterOps` 构建可复用的模板或辅助层。关键符号：`scatter_batch_rule`。

### Lines 740-773 / 第 740-773 行

```cpp
0740:   auto result = f(self_, physical_dim, index_, src_, args...);
0741:   // result should have same shape as self
0742:   if (self_logical_rank == 0) {
0743:     result = result.squeeze(-1);
0744:   }
0745:   return std::make_tuple(std::move(result), 0);
0746: }
0747: 
0748: } // namespace
0749: 
0750: std::tuple<Tensor, std::optional<int64_t>> scatter_value_batch_rule(
0751:     const Tensor& self, std::optional<int64_t> self_bdim,
0752:     int64_t dim,
0753:     const Tensor& index, std::optional<int64_t> index_bdim,
0754:     const Scalar& value) {
0755:   return scatter_batch_rule(ATEN_FN2(scatter, value),
0756:                             self, self_bdim, dim, index, index_bdim, value);
0757: }
0758: 
0759: std::tuple<Tensor, std::optional<int64_t>> scatter_src_batch_rule(
0760:     const Tensor& self, std::optional<int64_t> self_bdim,
0761:     int64_t dim,
0762:     const Tensor& index, std::optional<int64_t> index_bdim,
0763:     const Tensor& src, std::optional<int64_t> src_bdim) {
0764:   return scatter_batch_rule(ATEN_FN2(scatter, src),
0765:                             self, self_bdim, dim, index, index_bdim, src, src_bdim);
0766: }
0767: 
0768: std::tuple<Tensor, std::optional<int64_t>> scatter_add_batch_rule(
0769:     const Tensor& self, std::optional<int64_t> self_bdim,
0770:     int64_t dim,
0771:     const Tensor& index, std::optional<int64_t> index_bdim,
0772:     const Tensor& src, std::optional<int64_t> src_bdim) {
0773:   return scatter_batch_rule(ATEN_FN(scatter_add),
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `scatter_value_batch_rule`, `scatter_batch_rule`, `scatter_src_batch_rule`, `scatter_add_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `scatter_value_batch_rule`, `scatter_batch_rule`, `scatter_src_batch_rule`, `scatter_add_batch_rule`。

### Lines 774-803 / 第 774-803 行

```cpp
0774:                             self, self_bdim, dim, index, index_bdim, src, src_bdim);
0775: }
0776: 
0777: static void check_scatter_inplace_bdim(
0778:     std::optional<int64_t> self_bdim,
0779:     std::optional<int64_t> index_bdim,
0780:     std::optional<int64_t> src_bdim,
0781:     const char* schema_name) {
0782:   if (!self_bdim.has_value() && (index_bdim.has_value() || src_bdim.has_value())) {
0783:     vmapIncompatibleInplaceError(schema_name);
0784:   }
0785: }
0786: 
0787: std::tuple<Tensor, std::optional<int64_t>> scatter_add__batch_rule(
0788:     const Tensor& self, std::optional<int64_t> self_bdim,
0789:     int64_t dim,
0790:     const Tensor& index, std::optional<int64_t> index_bdim,
0791:     const Tensor& src, std::optional<int64_t> src_bdim) {
0792:   check_scatter_inplace_bdim(self_bdim, index_bdim, src_bdim, "scatter_add_");
0793:   return scatter_batch_rule(ATEN_FN(scatter_add_),
0794:                             self, self_bdim, dim, index, index_bdim, src, src_bdim);
0795: }
0796: 
0797: std::tuple<Tensor, std::optional<int64_t>> scatter_reduce_batch_rule(
0798:     const Tensor& self, std::optional<int64_t> self_bdim,
0799:     int64_t dim,
0800:     const Tensor& index, std::optional<int64_t> index_bdim,
0801:     const Tensor& src, std::optional<int64_t> src_bdim,
0802:     const std::string_view reduce) {
0803:   return scatter_batch_rule(ATEN_FN2(scatter, reduce),
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; implements scan/reduction-style dataflow; supports transform-aware functorch semantics. Key symbols: `check_scatter_inplace_bdim`, `vmapIncompatibleInplaceError`, `scatter_add__batch_rule`, `scatter_batch_rule`, `scatter_reduce_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；实现扫描/归约式数据流；支持面向变换的 functorch 语义。关键符号：`check_scatter_inplace_bdim`, `vmapIncompatibleInplaceError`, `scatter_add__batch_rule`, `scatter_batch_rule`, `scatter_reduce_batch_rule`。

### Lines 804-837 / 第 804-837 行

```cpp
0804:                             self, self_bdim, dim, index, index_bdim, src, src_bdim, reduce);
0805: }
0806: 
0807: std::tuple<Tensor, std::optional<int64_t>> scatter_reduce_two_batch_rule(
0808:     const Tensor& self, std::optional<int64_t> self_bdim,
0809:     int64_t dim,
0810:     const Tensor& index, std::optional<int64_t> index_bdim,
0811:     const Tensor& src, std::optional<int64_t> src_bdim,
0812:     const std::string_view reduce,
0813:     bool include_self) {
0814:   return scatter_batch_rule(ATEN_FN2(scatter_reduce, two),
0815:                             self, self_bdim, dim, index, index_bdim, src, src_bdim, reduce, include_self);
0816: }
0817: 
0818: std::tuple<Tensor, std::optional<int64_t>> scatter_reduce__two_batch_rule(
0819:     const Tensor& self, std::optional<int64_t> self_bdim,
0820:     int64_t dim,
0821:     const Tensor& index, std::optional<int64_t> index_bdim,
0822:     const Tensor& src, std::optional<int64_t> src_bdim,
0823:     const std::string_view reduce,
0824:     bool include_self) {
0825:   check_scatter_inplace_bdim(
0826:       self_bdim, index_bdim, src_bdim, "scatter_reduce_");
0827:   return scatter_batch_rule(ATEN_FN2(scatter_reduce_, two),
0828:                             self, self_bdim, dim, index, index_bdim, src, src_bdim, reduce, include_self);
0829: }
0830: 
0831: std::tuple<Tensor, std::optional<int64_t>> scatter_value_reduce_batch_rule(
0832:     const Tensor& self, std::optional<int64_t> self_bdim,
0833:     int64_t dim,
0834:     const Tensor& index, std::optional<int64_t> index_bdim,
0835:     const Scalar& src,
0836:     const std::string_view reduce) {
0837:   return scatter_batch_rule(ATEN_FN2(scatter, value_reduce),
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; implements scan/reduction-style dataflow. Key symbols: `scatter_reduce_two_batch_rule`, `scatter_batch_rule`, `scatter_reduce__two_batch_rule`, `check_scatter_inplace_bdim`, `scatter_value_reduce_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；实现扫描/归约式数据流。关键符号：`scatter_reduce_two_batch_rule`, `scatter_batch_rule`, `scatter_reduce__two_batch_rule`, `check_scatter_inplace_bdim`, `scatter_value_reduce_batch_rule`。

### Lines 838-868 / 第 838-868 行

```cpp
0838:                             self, self_bdim, dim, index, index_bdim, src, reduce);
0839: }
0840: 
0841: std::tuple<Tensor, std::optional<int64_t>> gather_batch_rule(
0842:     const Tensor& self, std::optional<int64_t> self_bdim,
0843:     int64_t dim,
0844:     const Tensor& index, std::optional<int64_t> index_bdim,
0845:     bool sparse_grad) {
0846:   auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0847:   auto index_logical_rank = rankWithoutBatchDim(index, index_bdim);
0848:   auto batch_size = get_bdim_size2(self, self_bdim, index, index_bdim);
0849: 
0850:   auto self_ = moveBatchDimToFront(self, self_bdim);
0851:   auto index_ = moveBatchDimToFront(index, index_bdim);
0852: 
0853:   if (self_logical_rank == 0) {
0854:     self_ = self_.unsqueeze(-1);
0855:   }
0856:   if (index_logical_rank == 0) {
0857:     index_ = index_.unsqueeze(-1);
0858:   }
0859:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0860:   index_ = ensure_has_bdim(index_, index_bdim.has_value(), batch_size);
0861:   auto physical_dim = getPhysicalDim(self_, /*has_batch_dim*/true, dim);
0862: 
0863:   auto result = at::gather(self_, physical_dim, index_, sparse_grad);
0864:   // result should have same rank as index
0865:   if (index_logical_rank == 0) {
0866:     result = result.squeeze(-1);
0867:   }
0868:   return std::make_tuple(std::move(result), 0);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; implements scan/reduction-style dataflow; supports transform-aware functorch semantics. Key symbols: `gather_batch_rule`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；实现扫描/归约式数据流；支持面向变换的 functorch 语义。关键符号：`gather_batch_rule`, `make_tuple`。

### Lines 869-898 / 第 869-898 行

```cpp
0869: }
0870: 
0871: Tensor get_expanded_index(const Tensor& index, SymIntArrayRef self_size, int64_t dim) {
0872:   if (index.dim() == 0) {
0873:     return index.expand_symint(self_size);
0874:   }
0875:   dim = maybe_wrap_dim(dim, static_cast<int64_t>(self_size.size()));
0876: 
0877:   // setup new_index_shape as [BS, 1, ..., idx_size, ..., 1]
0878:   // to reshape index_
0879:   auto idx_size = index.sym_size(0);  // get non-batch size of index tensor
0880:   Tensor index_;
0881:   {
0882:     VmapSymDimVector new_index_shape(self_size.size(), 1);
0883:     new_index_shape[dim] = idx_size;
0884:     index_ = index.view_symint(new_index_shape);
0885:   }
0886:   // Now apply expand to index_
0887:   {
0888:     VmapSymDimVector new_index_shape = {self_size.begin(), self_size.end()};
0889:     new_index_shape[dim] = idx_size;
0890:     index_ = index_.expand_symint(new_index_shape);
0891:   }
0892:   return index_;
0893: }
0894: 
0895: Tensor index_select_decomp(const Tensor &self, int64_t dim, const Tensor &index)
0896: {
0897:   Tensor index_ = index;
0898:   if (self.dim() > index.dim()) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `get_expanded_index`, `new_index_shape`, `index_select_decomp`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`get_expanded_index`, `new_index_shape`, `index_select_decomp`。

### Lines 899-928 / 第 899-928 行

```cpp
0899:     index_ = get_expanded_index(index, self.sym_sizes(), dim);
0900:   }
0901: 
0902:   auto result = at::gather(self, dim, index_);
0903: 
0904:   // output of gather has same dimension as `index` while
0905:   // output of index_select has same dimension as self
0906:   // Eg. t = torch.tensor(1)
0907:   //     idx = torch.tensor([0])
0908:   //     torch.index_select(t, 0, idx) # 0-D
0909:   //     torch.gather(t, 0, idx) # 1-D
0910:   if (self.dim() == 0 && result.dim() != 0) {
0911:     result = result.squeeze(-1);
0912:   }
0913: 
0914:   return result;
0915: }
0916: 
0917: Tensor index_copy_decomp(
0918:     const Tensor &self, int64_t dim,
0919:     const Tensor &index, const Tensor &source)
0920: {
0921:   Tensor index_ = index;
0922:   if (self.dim() > index.dim()) {
0923:     index_ = get_expanded_index(index, self.sym_sizes(), dim);
0924:   }
0925: 
0926:   return at::scatter(self, dim, index_, source);  ;
0927: }
0928: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `index_copy_decomp`, `scatter`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`index_copy_decomp`, `scatter`。

### Lines 929-961 / 第 929-961 行

```cpp
0929: // Note [Fix vmap slice_scatter]
0930: // registers a decomposition for `slice_scatter` that calls into `slice.src`
0931: // *_scatter operators have some special semantics though, that we can't easily
0932: // through a decomposition: slice_scatter's output needs to have the same
0933: // size, size, strides and storage_offset as the input.
0934: Tensor slice_scatter_decomp(const Tensor &self, const Tensor &src,
0935:                             int64_t dim, std::optional<int64_t> start,
0936:                             std::optional<int64_t> end, int64_t step)
0937: {
0938:   auto idx = at::arange(start.value_or(0), end.value_or(self.size(dim)), step, self.options().dtype(kLong));
0939:   idx = get_expanded_index(idx, self.sym_sizes(), dim);
0940:   return at::scatter(self, dim, idx, src);
0941: }
0942: 
0943: Tensor select_scatter_decomp(
0944:     const Tensor &self, const Tensor &source,
0945:     int64_t dim, int64_t index)
0946: {
0947:   // supports negative index
0948:   index = maybe_wrap_dim(index, self.size(dim));
0949:   auto index_ = at::scalar_tensor(index, self.options().dtype(kLong));
0950: 
0951:   return at::scatter(self, dim, index_.expand_as(self), source.unsqueeze(dim).expand_as(self));
0952: }
0953: 
0954: std::tuple<Tensor, std::optional<int64_t>> diagonal_scatter_batch_rule(
0955:     const Tensor &self, std::optional<int64_t> self_bdim,
0956:     const Tensor &src, std::optional<int64_t> src_bdim,
0957:     int64_t offset, int64_t dim1, int64_t dim2)
0958: {
0959:   auto self_ = moveBatchDimToFront(self, self_bdim);
0960:   auto src_ = moveBatchDimToFront(src, src_bdim);
0961: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `slice_scatter_decomp`, `scatter`, `select_scatter_decomp`, `diagonal_scatter_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`slice_scatter_decomp`, `scatter`, `select_scatter_decomp`, `diagonal_scatter_batch_rule`。

### Lines 962-991 / 第 962-991 行

```cpp
0962:   auto batch_size = get_bdim_size2(self, self_bdim, src, src_bdim);
0963: 
0964:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0965:   src_ = ensure_has_bdim(src_, src_bdim.has_value(), batch_size);
0966: 
0967:   auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0968:   dim1 = maybe_wrap_dim(dim1, self_logical_rank) + 1;
0969:   dim2 = maybe_wrap_dim(dim2, self_logical_rank) + 1;
0970: 
0971:   return std::make_tuple(at::diagonal_scatter(self_, src_, offset, dim1, dim2), 0);
0972: }
0973: 
0974: std::tuple<Tensor, std::optional<int64_t>> index_add_batch_rule_impl(
0975:     Tensor& self, std::optional<int64_t> self_bdim,
0976:     int64_t dim,
0977:     const Tensor& index, std::optional<int64_t> index_bdim,
0978:     const Tensor& other, std::optional<int64_t> other_bdim,
0979:     const Scalar& alpha,
0980:     const bool inplace) {
0981: 
0982:   if (inplace && !self_bdim.has_value()){
0983:     vmapIncompatibleInplaceError("index_add_");
0984:   }
0985: 
0986:   if (!index_bdim) {
0987:     // Handle scalar tensors... self, other can be scalar tensors
0988:     const auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0989:     const auto other_logical_rank = rankWithoutBatchDim(other, other_bdim);
0990:     auto self_ = moveBatchDimToFront(self, self_bdim);
0991:     if (self_logical_rank == 0) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `index_add_batch_rule_impl`, `vmapIncompatibleInplaceError`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `index_add_batch_rule_impl`, `vmapIncompatibleInplaceError`。

### Lines 992-1023 / 第 992-1023 行

```cpp
0992:       self_ = self_.unsqueeze(-1);
0993:     }
0994:     auto other_ = moveBatchDimToFront(other, other_bdim);
0995:     if (other_logical_rank == 0) {
0996:       other_ = other_.unsqueeze(-1);
0997:     }
0998:     dim = maybe_wrap_dim(dim, self_logical_rank);
0999: 
1000:     const auto batch_size = get_bdim_size2(self, self_bdim, other, other_bdim);
1001:     self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
1002:     other_ = ensure_has_bdim(other_, other_bdim.has_value(), batch_size);
1003: 
1004:     if (inplace) {
1005:       self_.index_add_(dim + 1, index, other_, alpha);
1006:       if (self_logical_rank == 0) {
1007:         self_ = self_.squeeze(-1);
1008:       }
1009:       return std::make_tuple(self, 0);
1010:     }
1011: 
1012:     auto result = self_.index_add(dim + 1, index, other_, alpha);
1013:     if (self_logical_rank == 0) {
1014:       result = result.squeeze(-1);
1015:     }
1016:     return std::make_tuple(std::move(result), 0);
1017:   }
1018: 
1019:   // Index is batched. For-loop and stack is the best thing I can come up with
1020:   // right now. We really want generalized index_add kernel in PyTorch
1021:   auto batch_size = get_bdim_size3(self, self_bdim, other, other_bdim, index, index_bdim);
1022:   std::vector<Tensor> results;
1023:   if (!inplace) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`。

### Lines 1024-1055 / 第 1024-1055 行

```cpp
1024:     results.reserve(batch_size);
1025:   }
1026:   for (const auto i : c10::irange(0, batch_size)) {
1027:     const auto& self_slice = self_bdim.has_value() ?
1028:       self.select(*self_bdim, i) : self;
1029:     const auto& other_slice = other_bdim.has_value() ?
1030:       other.select(*other_bdim, i) : other;
1031:     const auto& index_slice = index_bdim.has_value() ?
1032:       index.select(*index_bdim, i) : index;
1033: 
1034:     if (inplace) {
1035:       self_slice.index_add_(dim, index_slice, other_slice, alpha);
1036:     } else {
1037:       results.push_back(at::index_add(self_slice, dim, index_slice, other_slice, alpha));
1038:     }
1039:   }
1040:   if (inplace) {
1041:     return std::make_tuple(at::stack(self), 0);
1042:   }
1043:   return std::make_tuple(at::stack(results), 0);
1044: }
1045: 
1046: void index_add__batch_rule(
1047:     Tensor& self, std::optional<int64_t> self_bdim,
1048:     int64_t dim,
1049:     const Tensor& index, std::optional<int64_t> index_bdim,
1050:     const Tensor& other, std::optional<int64_t> other_bdim,
1051:     const Scalar& alpha) {
1052:   index_add_batch_rule_impl(self, self_bdim, dim, index, index_bdim, other,
1053:                             other_bdim, alpha, true);
1054: }
1055: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `index_add__batch_rule`, `index_add_batch_rule_impl`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `index_add__batch_rule`, `index_add_batch_rule_impl`。

### Lines 1056-1086 / 第 1056-1086 行

```cpp
1056: std::tuple<Tensor, std::optional<int64_t>> index_add_batch_rule(
1057:     Tensor& self, std::optional<int64_t> self_bdim,
1058:     int64_t dim,
1059:     const Tensor& index, std::optional<int64_t> index_bdim,
1060:     const Tensor& other, std::optional<int64_t> other_bdim,
1061:     const Scalar& alpha) {
1062:   auto self_ = self.clone(at::MemoryFormat::Preserve);
1063:   return index_add_batch_rule_impl(self_, self_bdim, dim, index, index_bdim,
1064:                                    other, other_bdim, alpha, false);
1065: }
1066: 
1067: std::tuple<Tensor,Tensor> binary_pointwise_align(
1068:     const Tensor & self,
1069:     std::optional<int64_t> self_bdim,
1070:     const Tensor & mask,
1071:     std::optional<int64_t> mask_bdim) {
1072:   // compute max logical rank
1073:   auto tensor_logical_rank = rankWithoutBatchDim(self, self_bdim);
1074:   auto other_logical_rank = rankWithoutBatchDim(mask, mask_bdim);
1075:   auto max_logical_rank = std::max(tensor_logical_rank, other_logical_rank);
1076: 
1077:   auto tensor_ = moveBatchDimToFront(self, self_bdim);
1078:   auto other_ = moveBatchDimToFront(mask, mask_bdim);
1079: 
1080:   // If the dimensions aren't aligned, we need to line them up.
1081:   // Tensor[B, 3] + Tensor[2, 5, 3] -> Tensor[B, 1, 1, 3] + Tensor[2, 5, 3]
1082:   // Note that only tensors that have a batch dim need to be modified.
1083:   // Tensor[B, 2, 3, 5] + Tensor[5] -> no changes needed
1084:   tensor_ = maybePadToLogicalRank(tensor_, self_bdim, max_logical_rank);
1085:   other_ = maybePadToLogicalRank(other_, mask_bdim, max_logical_rank);
1086: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `index_add_batch_rule`, `index_add_batch_rule_impl`, `binary_pointwise_align`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`index_add_batch_rule`, `index_add_batch_rule_impl`, `binary_pointwise_align`。

### Lines 1087-1123 / 第 1087-1123 行

```cpp
1087:   return std::make_tuple(std::move(tensor_), std::move(other_));
1088: }
1089: 
1090: std::tuple<Tensor, std::optional<int64_t>> masked_fill_scalar_batch_rule(
1091:     const Tensor & self,
1092:     std::optional<int64_t> self_bdim,
1093:     const Tensor & mask,
1094:     std::optional<int64_t> mask_bdim,
1095:     const Scalar& source) {
1096:   auto tensors = binary_pointwise_align(self, self_bdim, mask, mask_bdim);
1097:   auto result = at::masked_fill(std::get<0>(tensors), std::get<1>(tensors), source);
1098:   return std::make_tuple(std::move(result), 0);
1099: }
1100: 
1101: std::tuple<Tensor, std::optional<int64_t>> index_fill_batch_rule_helper(
1102:   int64_t batch_size,
1103:   int64_t self_logical_rank,
1104:   int64_t index_logical_rank,
1105:   Tensor & self_,
1106:   int64_t dim,
1107:   Tensor & index_,
1108:   const Scalar & value
1109:   ){
1110:   if (self_logical_rank != 0){
1111:     auto index_offset = at::arange(
1112:       batch_size,
1113:       at::TensorOptions().dtype(index_.scalar_type()).device(index_.device())
1114:     );
1115:     if (index_logical_rank == 0){
1116:       index_ = index_.unsqueeze(-1);
1117:     }
1118:     index_ = index_.add(index_offset.unsqueeze(-1), self_.size(dim + 1));
1119:     index_ = reshape_dim_into(0, 0, index_);
1120:     self_ = reshape_dim_into(0, dim, self_);
1121:     self_.index_fill_(dim, index_, value);
1122:     self_ = reshape_dim_outof(dim, batch_size, self_);
1123:     return std::make_tuple(self_, dim);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `masked_fill_scalar_batch_rule`, `index_fill_batch_rule_helper`, `TensorOptions`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `masked_fill_scalar_batch_rule`, `index_fill_batch_rule_helper`, `TensorOptions`。

### Lines 1124-1153 / 第 1124-1153 行

```cpp
1124:   }
1125: 
1126:   // If self_logical_rank == 0, the batch dim is certainly 0, and we must apply batched indices to each row.
1127:   if (index_logical_rank != 0){
1128:     index_ = reshape_dim_into(0, 0, index_);
1129:   }
1130:   self_.unsqueeze_(-1);
1131:   self_.index_fill_(dim + 1, index_, value);
1132:   self_.squeeze_(-1);
1133: 
1134:   return std::make_tuple(self_, 0);
1135: }
1136: 
1137: std::tuple<Tensor, std::optional<int64_t>> index_fill_int_scalar_batch_rule_impl(
1138:     Tensor & self, std::optional<int64_t> self_bdim,
1139:     int64_t dim,
1140:     const Tensor & index, std::optional<int64_t> index_bdim,
1141:     const Scalar & value,
1142:     const bool inplace) {
1143:   const auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
1144:   const auto index_logical_rank = rankWithoutBatchDim(index, index_bdim);
1145:   Tensor self_ = moveBatchDimToFront(self, self_bdim);
1146:   Tensor index_ = moveBatchDimToFront(index, index_bdim);
1147:   dim = maybe_wrap_dim(dim, self_logical_rank);
1148: 
1149:   if (inplace && !self_bdim.has_value()) {
1150:     vmapIncompatibleInplaceError("index_fill_");
1151:   }
1152: 
1153:   if (!index_bdim) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `index_fill_int_scalar_batch_rule_impl`, `vmapIncompatibleInplaceError`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `index_fill_int_scalar_batch_rule_impl`, `vmapIncompatibleInplaceError`。

### Lines 1154-1183 / 第 1154-1183 行

```cpp
1154:     if (self_logical_rank == 0){
1155:       self_.unsqueeze_(-1);
1156:     }
1157:     self_.index_fill_(dim + 1, index_, value);
1158:     if (self_logical_rank == 0) {
1159:       self_.squeeze_(-1);
1160:     }
1161:     return std::make_tuple(self_, 0);
1162:   }
1163: 
1164:   auto batch_size = get_bdim_size2(self, self_bdim, index, index_bdim);
1165:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
1166:   index_ = ensure_has_bdim(index_, index_bdim.has_value(), batch_size);
1167: 
1168:   if (inplace) {
1169:     // Do for-loop for in-place because we cannot reshape
1170:     // `self_` having an incompatible stride without copying.
1171:     for (const auto i : c10::irange(0, batch_size)) {
1172:       const auto& self_slice = self_.select(0, i);
1173:       const auto& index_slice = index_.select(0, i);
1174:       self_slice.index_fill_(
1175:         dim,
1176:         index_slice,
1177:         value
1178:       );
1179:     }
1180:     return std::make_tuple(self_, 0);
1181:   }
1182: 
1183:   if (!self_bdim.has_value()) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值。关键符号：`make_tuple`。

### Lines 1184-1215 / 第 1184-1215 行

```cpp
1184:     self_ = self_.clone();
1185:   }
1186: 
1187:   return index_fill_batch_rule_helper(batch_size, self_logical_rank, index_logical_rank, self_, dim, index_, value);
1188: }
1189: 
1190: std::tuple<Tensor, std::optional<int64_t>> index_fill_int_tensor_batch_rule_impl(
1191:     Tensor & self, std::optional<int64_t> self_bdim,
1192:     int64_t dim,
1193:     const Tensor & index, std::optional<int64_t> index_bdim,
1194:     const Tensor & value, std::optional<int64_t> value_bdim,
1195:     const bool inplace) {
1196:   const auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
1197:   const auto index_logical_rank = rankWithoutBatchDim(index, index_bdim);
1198:   Tensor self_ = moveBatchDimToFront(self, self_bdim);
1199:   Tensor index_ = moveBatchDimToFront(index, index_bdim);
1200:   Tensor value_ = moveBatchDimToFront(value, value_bdim);
1201:   dim = maybe_wrap_dim(dim, self_logical_rank);
1202: 
1203:   if (inplace && !self_bdim.has_value()) {
1204:     vmapIncompatibleInplaceError("index_fill_");
1205:   }
1206: 
1207:   if (!index_bdim && !value_bdim) {
1208:     if (self_logical_rank == 0){
1209:       self_.unsqueeze_(-1);
1210:     }
1211:     self_.index_fill_(dim + 1, index_, value);
1212:     if (self_logical_rank == 0) {
1213:       self_.squeeze_(-1);
1214:     }
1215:     return std::make_tuple(self_, 0);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `index_fill_batch_rule_helper`, `index_fill_int_tensor_batch_rule_impl`, `vmapIncompatibleInplaceError`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`index_fill_batch_rule_helper`, `index_fill_int_tensor_batch_rule_impl`, `vmapIncompatibleInplaceError`, `make_tuple`。

### Lines 1216-1246 / 第 1216-1246 行

```cpp
1216:   }
1217: 
1218:   auto batch_size = get_bdim_size3(self, self_bdim, index, index_bdim, value, value_bdim);
1219:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
1220:   index_ = ensure_has_bdim(index_, index_bdim.has_value(), batch_size);
1221: 
1222:   if (inplace || value_bdim.has_value()) {
1223:     // Do for-loop for in-place because we cannot reshape
1224:     // `self_` having an incompatible stride without copying.
1225:     // If value has a batch dim, we do for-loop as well because
1226:     // index_fill_ supports 1-element tensor only.
1227:     for (const auto i : c10::irange(0, batch_size)) {
1228:       const auto& self_slice = self_.select(0, i);
1229:       const auto& index_slice = index_.select(0, i);
1230:       self_slice.index_fill_(
1231:         dim,
1232:         index_slice,
1233:         value_bdim.has_value() ? value_.select(0, i) : value_
1234:       );
1235:     }
1236:     return std::make_tuple(self_, 0);
1237:   }
1238: 
1239:   if (!self_bdim.has_value()) {
1240:     self_ = self_.clone();
1241:   }
1242: 
1243:   // calling .item() on value is safe here because value is guaranteed to not be a batched tensor.
1244:   return index_fill_batch_rule_helper(batch_size, self_logical_rank, index_logical_rank, self_, dim, index_, value.item());
1245: }
1246: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `index_fill_batch_rule_helper`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `index_fill_batch_rule_helper`。

### Lines 1247-1278 / 第 1247-1278 行

```cpp
1247: void index_fill__int_scalar_batch_rule(
1248:     Tensor & self, std::optional<int64_t> self_bdim,
1249:     int64_t dim,
1250:     const Tensor & index, std::optional<int64_t> index_bdim,
1251:     const Scalar & value) {
1252:   index_fill_int_scalar_batch_rule_impl(self, self_bdim, dim, index, index_bdim, value, true);
1253: }
1254: 
1255: void index_fill__int_tensor_batch_rule(
1256:     Tensor & self, std::optional<int64_t> self_bdim,
1257:     int64_t dim,
1258:     const Tensor & index, std::optional<int64_t> index_bdim,
1259:     const Tensor & value, std::optional<int64_t> value_bdim) {
1260:   index_fill_int_tensor_batch_rule_impl(self, self_bdim, dim, index, index_bdim, value, value_bdim, true);
1261: }
1262: 
1263: std::tuple<Tensor, std::optional<int64_t>> index_fill_int_scalar_batch_rule(
1264:     const Tensor & self, std::optional<int64_t> self_bdim,
1265:     int64_t dim,
1266:     const Tensor & index, std::optional<int64_t> index_bdim,
1267:     const Scalar & value) {
1268:   auto self_ = self.clone(at::MemoryFormat::Preserve);
1269:   return index_fill_int_scalar_batch_rule_impl(self_, self_bdim, dim, index, index_bdim, value, false);
1270: }
1271: 
1272: std::tuple<Tensor, std::optional<int64_t>> index_fill_int_tensor_batch_rule(
1273:     const Tensor & self, std::optional<int64_t> self_bdim,
1274:     int64_t dim,
1275:     const Tensor & index, std::optional<int64_t> index_bdim,
1276:     const Tensor & value, std::optional<int64_t> value_bdim) {
1277:   auto self_ = self.clone(at::MemoryFormat::Preserve);
1278:   return index_fill_int_tensor_batch_rule_impl(self_, self_bdim, dim, index, index_bdim, value, value_bdim, false);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `index_fill__int_scalar_batch_rule`, `index_fill_int_scalar_batch_rule_impl`, `index_fill__int_tensor_batch_rule`, `index_fill_int_tensor_batch_rule_impl`, `index_fill_int_scalar_batch_rule`, `index_fill_int_tensor_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`index_fill__int_scalar_batch_rule`, `index_fill_int_scalar_batch_rule_impl`, `index_fill__int_tensor_batch_rule`, `index_fill_int_tensor_batch_rule_impl`, `index_fill_int_scalar_batch_rule`, `index_fill_int_tensor_batch_rule`。

### Lines 1279-1316 / 第 1279-1316 行

```cpp
1279: }
1280: 
1281: }
1282: 
1283: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
1284:   m.impl("index.Tensor", index_plumbing);
1285:   m.impl("index_put_", index_put__plumbing);
1286:   m.impl("index_put", index_put_plumbing);
1287:   m.impl("_index_put_impl_", _index_put_impl__plumbing);
1288:   m.impl("slice_scatter", slice_scatter_decomp);
1289:   m.impl("select_scatter", select_scatter_decomp);
1290:   m.impl("index_copy", index_copy_decomp);
1291:   m.impl("index_select", index_select_decomp);
1292:   VMAP_SUPPORT2(masked_fill, Scalar, masked_fill_scalar_batch_rule);
1293:   VMAP_SUPPORT2(index_fill_, int_Tensor, index_fill__int_tensor_batch_rule);
1294:   VMAP_SUPPORT2(index_fill_, int_Scalar, index_fill__int_scalar_batch_rule);
1295:   VMAP_SUPPORT2(index_fill, int_Tensor, index_fill_int_tensor_batch_rule);
1296:   VMAP_SUPPORT2(index_fill, int_Scalar, index_fill_int_scalar_batch_rule);
1297:   VMAP_SUPPORT(index_add_, index_add__batch_rule);
1298:   VMAP_SUPPORT(index_add, index_add_batch_rule);
1299:   VMAP_SUPPORT(diagonal_scatter, diagonal_scatter_batch_rule);
1300:   VMAP_SUPPORT(gather, gather_batch_rule);
1301:   VMAP_SUPPORT2(scatter, value, scatter_value_batch_rule);
1302:   VMAP_SUPPORT2(scatter, src, scatter_src_batch_rule);
1303:   VMAP_SUPPORT(scatter_add, scatter_add_batch_rule);
1304:   VMAP_SUPPORT(scatter_add_, scatter_add__batch_rule);
1305:   VMAP_SUPPORT2(scatter, reduce, scatter_reduce_batch_rule);
1306:   VMAP_SUPPORT2(scatter, value_reduce, scatter_value_reduce_batch_rule);
1307:   VMAP_SUPPORT2(scatter_reduce, two, scatter_reduce_two_batch_rule);
1308:   VMAP_SUPPORT2(scatter_reduce_, two, scatter_reduce__two_batch_rule);
1309:   // as_strided_scatter does not work with the for-loop fallback today,
1310:   // because as_strided_scatter will return an output that matches
1311:   // the strides/storage_offset of its input.
1312:   // With the for loop fallback, each input tensor is a slice into
1313:   // the larger batched tensor.
1314:   m.impl("as_strided_scatter", torch::CppFunction::makeFromBoxedFunction<&vmapErrorFallback>());
1315: }
1316: 
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesScatterOps` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesScatterOps` 的行为。符号：无明显局部符号。

### Lines 1317-1318 / 第 1317-1318 行

```cpp
1317: } // namespace at::functorch
1318: // NOLINTEND(bugprone-unchecked-optional-access)
```

- **EN:** This block implements local helper logic for `BatchRulesScatterOps`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `BatchRulesScatterOps` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `ATen/Operators.h`, `ATen/functorch/PlumbingHelper.h`, `ATen/functorch/BatchedFallback.h`, `ATen/native/IndexKernel.h`, `ATen/native/IndexingUtils.h`, `torch/library.h`, `c10/util/Exception.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`, `void`, `std::tuple<Tensor,`
- **Representative symbols / 代表性符号**: `any_has_value`, `get_num_leading_nones`, `get_max_index_logical_dim`, `batchIndices`, `is_advanced_index`, `are_advanced_indices_adjacent`, `swap_regions`, `permutation`, `iota`, `rotate`, `index_batch_rule`, `make_tuple`, `...`
