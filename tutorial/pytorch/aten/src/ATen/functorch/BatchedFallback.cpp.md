# BatchedFallback.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchedFallback.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchedFallback.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Descriptor/handle lifecycle management is important here. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchedFallback.cpp` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/BatchedFallback.h>
0008: #include <ATen/functorch/LegacyVmapTransforms.h>
0009: #include <ATen/functorch/DynamicLayer.h>
0010: #include <ATen/functorch/PlumbingHelper.h>
0011: 
0012: #include <ATen/Context.h>
0013: #include <ATen/MatrixRef.h>
0014: #include <ATen/core/dispatch/Dispatcher.h>
0015: #include <c10/util/accumulate.h>
0016: #include <c10/util/llvmMathExtras.h>
0017: #include <c10/util/irange.h>
0018: 
0019: namespace at::functorch {
0020: 
```

- **EN:** This block supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 21-40 / 第 21-40 行

```cpp
0021: static bool kVmapFallbackWarningEnabled = true;
0022: 
0023: bool isVmapFallbackWarningEnabled() {
0024:   return kVmapFallbackWarningEnabled;
0025: }
0026: 
0027: void setVmapFallbackWarningEnabled(bool enabled) {
0028:   kVmapFallbackWarningEnabled = enabled;
0029: }
0030: 
0031: static bool kVmapFallbackEnabled = true;
0032: 
0033: bool isVmapFallbackEnabled() {
0034:   return kVmapFallbackEnabled;
0035: }
0036: 
0037: void setVmapFallbackEnabled(bool enabled) {
0038:   kVmapFallbackEnabled = enabled;
0039: }
0040: 
```

- **EN:** This block produces a result or forwards a computed value; supports transform-aware functorch semantics. Key symbols: `isVmapFallbackWarningEnabled`, `setVmapFallbackWarningEnabled`, `isVmapFallbackEnabled`, `setVmapFallbackEnabled`.
- **CN:** 该代码块返回结果或转发已计算的值；支持面向变换的 functorch 语义。关键符号：`isVmapFallbackWarningEnabled`, `setVmapFallbackWarningEnabled`, `isVmapFallbackEnabled`, `setVmapFallbackEnabled`。

### Lines 41-63 / 第 41-63 行

```cpp
0041: // Given a linear index, return the actual index.
0042: // Example: Given linear_idx = 3, sizes = [5, 2], we would return [1, 0]
0043: static at::SmallVector<indexing::TensorIndex,kVmapStaticDimVecSize>
0044: computeIndex(int64_t linear_idx, IntArrayRef sizes) {
0045:   at::SmallVector<indexing::TensorIndex,kVmapStaticDimVecSize> result;
0046:   result.reserve(sizes.size());
0047:   for (auto it = sizes.rbegin(); it != sizes.rend(); it++) {
0048:     auto remainder = linear_idx % *it;
0049:     result.push_back(remainder);
0050:     linear_idx -= remainder;
0051:     linear_idx /= *it;
0052:   }
0053:   std::reverse(std::begin(result), std::end(result));
0054:   return result;
0055: }
0056: 
0057: static bool areAllReturnsTensors(const at::FunctionSchema& schema) {
0058:   return std::all_of(
0059:       schema.returns().begin(),
0060:       schema.returns().end(),
0061:       [] (const Argument& arg) { return arg.type() == TensorType::get(); });
0062: }
0063: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `computeIndex`, `reverse`, `areAllReturnsTensors`, `all_of`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`computeIndex`, `reverse`, `areAllReturnsTensors`, `all_of`。

### Lines 64-85 / 第 64-85 行

```cpp
0064: static bool areAnyArgumentsTensorList(const at::FunctionSchema& schema) {
0065:   return std::any_of(
0066:       schema.arguments().begin(),
0067:       schema.arguments().end(),
0068:       [] (const Argument& arg) {
0069:         return arg.type()->isSubtypeOf(ListType::ofTensors()) ||
0070:           arg.type()->isSubtypeOf(ListType::ofOptionalTensors());
0071:       });
0072: }
0073: 
0074: static void warnFallback(const c10::FunctionSchema& schema, bool is_inplace, bool is_nested=false) {
0075:   TORCH_CHECK(isVmapFallbackEnabled(),
0076:       schema.operator_name(), " hit the vmap fallback which is currently disabled");
0077:   if (!isVmapFallbackWarningEnabled()) {
0078:     return;
0079:   }
0080:   TORCH_WARN("There is a performance drop because we have not yet implemented ",
0081:              "the ", (is_nested ? "nested " : "") , "batching rule for ",
0082:              schema.operator_name(), ". Please file us an issue on GitHub so that ",
0083:              "we can prioritize its implementation.");
0084: }
0085: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `areAnyArgumentsTensorList`, `any_of`, `warnFallback`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`areAnyArgumentsTensorList`, `any_of`, `warnFallback`。

### Lines 86-109 / 第 86-109 行

```cpp
0086: // The general flow of the algorithm is as follows.
0087: // - First, we figure out which arguments are BatchedTensors and save them
0088: //   to a vector. We also store a vector of which index of the arguments list
0089: //   each BatchedTensor appears in. This will be useful for bookkeeping later.
0090: // - Next, we apply the MultiBatchVmapTransform to all of the BatchedTensors.
0091: //   This returns a vector of VmapPhysicalView that hold tensors that contain
0092: //   all of the collective batch dimensions at the front of the tensors.
0093: // - Then, we attempt to call `op` once per slice of the inputs. To do this,
0094: //   we repeatedly we slice the input arguments (if they are BatchedTensors),
0095: //   put the sliced (or a not-sliced) version of the input onto the stack, invoke
0096: //   the operator, and then pop the results off the stack.
0097: static void batchedTensorInplaceForLoopFallback(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
0098:   const auto& schema = op.schema();
0099:   warnFallback(schema, /*in_place*/true);
0100: 
0101:   const auto num_arguments = schema.arguments().size();
0102:   const auto arguments = torch::jit::last(stack, num_arguments);
0103:   const auto arguments_begin = stack->size() - num_arguments;
0104: 
0105:   // `self` is the Tensor being modified in-place
0106:   Tensor self = arguments[0].toTensor();
0107:   const auto* self_impl = maybeGetBatchedImpl(self);
0108:   std::bitset<kVmapMaxTensorDims> self_vmap_levels;
0109:   if (self_impl) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `batchedTensorInplaceForLoopFallback`, `warnFallback`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`batchedTensorInplaceForLoopFallback`, `warnFallback`。

### Lines 110-130 / 第 110-130 行

```cpp
0110:     self_vmap_levels = createVmapLevelsBitset(self_impl->level());
0111:   }
0112: 
0113:   // Figure out which arguments are BatchedTensor. Save them to a vector.
0114:   // For each BatchedTensor, also record what position of `arguments` they came from.
0115:   at::SmallVector<Tensor,kVmapTransformStaticInputSize> batched_tensor_inputs;
0116:   VmapDimVector batched_tensor_inputs_position;
0117:   for (const auto idx : c10::irange(0, arguments.size())) {
0118:     const auto& ivalue = arguments[idx];
0119:     if (!ivalue.isTensor()) {
0120:       continue;
0121:     }
0122:     const auto& tensor = ivalue.toTensor();
0123:     if (!tensor.defined()) {
0124:       continue;
0125:     }
0126:     const auto* batched = maybeGetBatchedImpl(tensor);
0127:     if (!batched) {
0128:       continue;
0129:     }
0130: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 131-151 / 第 131-151 行

```cpp
0131:     // NOTE: [vmap-incompatible in-place operations]
0132:     // In-place operations on `self` are not possible if there exists some vmap
0133:     // level `l` such that `self` is not being vmapped on that level but another
0134:     // argument is. For example, let B0 be a batch dim inside vmap and consider
0135:     // vmap(Tensor.add_, in_dims=(None, 0))(torch.ones(3), torch.ones(B0, 3))
0136:     // - self is torch.ones(3) and does not participate in this vmap
0137:     // - other is BatchedTensor(torch.ones(B0, 3))
0138:     // There's no way to do self.add_(other) because `other` has more elements
0139:     // elements than `self` due to being vmapped over.
0140:     //
0141:     // In the vmap fallback, we should error out when we detect this.
0142:     auto other_vmap_levels = createVmapLevelsBitset(batched->level());
0143:     if (self_vmap_levels != (self_vmap_levels | other_vmap_levels)) {
0144:       // Find one vmap level to complain about
0145:       auto additional_bdims = (self_vmap_levels | other_vmap_levels) ^ self_vmap_levels;
0146:       [[maybe_unused]] auto offending_level = llvm::findLastSet(additional_bdims.to_ulong());
0147:       // The following prints out "vmap: aten::add_(tensor, ...) is not possible",
0148:       // but it would be better to print out "tensor.add_(...) is not possible".
0149:       // Afaict there's no official way to get the add_ and there is no way to
0150:       // tell if an operator has method or function variants.
0151:       TORCH_CHECK(false,
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 152-178 / 第 152-178 行

```cpp
0152:         "vmap: ", schema.name(), "(self, *extra_args) is not possible because ",
0153:         "there exists a Tensor `other` in extra_args that has more elements ",
0154:         "than `self`. This happened due to `other` being vmapped over but ",
0155:         "`self` not being vmapped over at level ", offending_level, ". ",
0156:         "Please try to use out-of-place operators instead of ", schema.name(), ". ",
0157:         "If said operator is being called inside the PyTorch framework, ",
0158:         "please file a bug report instead.");
0159:     }
0160:     batched_tensor_inputs.push_back(tensor);
0161:     batched_tensor_inputs_position.push_back(static_cast<int64_t>(idx));
0162:   }
0163:   TORCH_INTERNAL_ASSERT(!batched_tensor_inputs.empty());
0164: 
0165:   // MultiBatchVmapTransform the BatchedTensor arguments. This returns
0166:   // VmapPhysicalViews that contain all of the batch dimensions.
0167:   const auto input_physical_views = MultiBatchVmapTransform::logicalToPhysical(
0168:       batched_tensor_inputs);
0169: 
0170:   // Compute the total number of batches
0171:   auto num_batch_dims = input_physical_views.front().numBatchDims();
0172:   auto first_physical_view_sizes = input_physical_views.front().tensor().sizes();
0173:   auto batch_sizes = ArrayRef<int64_t>(
0174:       first_physical_view_sizes.begin(), first_physical_view_sizes.begin() + num_batch_dims);
0175:   const auto num_batches = c10::multiply_integers(batch_sizes);
0176:   // Without a shape-checking API, we're unable to compute the correct shape of
0177:   // the output so we just error out.
0178:   TORCH_CHECK(num_batches > 0,
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 179-199 / 第 179-199 行

```cpp
0179:       "Batching rule not implemented for ", schema.operator_name(), ". ",
0180:       "The fallback path does not support vmap over dims of size 0.");
0181: 
0182:   // Strategy: For each batch, we are going to push slices (where applicable)
0183:   // of the arguments onto `stack`, and call `op`.
0184:   for (int64_t linear_idx = 0; linear_idx < num_batches; ++linear_idx) {
0185:     auto index = computeIndex(linear_idx, batch_sizes);
0186:     auto batched_tensor_inputs_pos_iter = batched_tensor_inputs_position.begin();
0187:     auto input_physical_views_iter = input_physical_views.begin();
0188:     for (const auto arg_idx : c10::irange(0, num_arguments)) {
0189:       // We assume that torch::jit::Stack is backed by vector<IValue> for
0190:       // simplicity. When that is not the case, this code should be updated.
0191:       const auto& argument = (*stack)[arguments_begin + arg_idx];
0192:       if (batched_tensor_inputs_pos_iter == batched_tensor_inputs_position.end()
0193:           || static_cast<int64_t>(arg_idx) != *batched_tensor_inputs_pos_iter) {
0194:         // argument isn't a BatchedTensor
0195:         torch::jit::push(stack, argument);
0196:         continue;
0197:       }
0198:       // argument is a BatchedTensor
0199:       TORCH_INTERNAL_ASSERT(input_physical_views_iter != input_physical_views.end());
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `push`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`push`。

### Lines 200-219 / 第 200-219 行

```cpp
0200:       const auto& physical_view_for_argument = *input_physical_views_iter;
0201:       auto thing = physical_view_for_argument.tensor().index(index);
0202:       torch::jit::push(stack, thing);
0203:       batched_tensor_inputs_pos_iter++;
0204:       input_physical_views_iter++;
0205:     }
0206: 
0207:     op.callBoxed(stack);
0208:     torch::jit::drop(stack, 1);
0209:   }
0210: 
0211:   // Return the tensor that was written to in-place
0212:   torch::jit::drop(stack, num_arguments);
0213:   torch::jit::push(stack, self);
0214: }
0215: 
0216: static Tensor safeStack(TensorList tensors) {
0217:   auto is_defined = [](const Tensor& t) { return t.defined(); };
0218:   if (std::all_of(tensors.begin(), tensors.end(), is_defined)) {
0219:     return at::stack(tensors);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `push`, `drop`, `safeStack`, `stack`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`push`, `drop`, `safeStack`, `stack`。

### Lines 220-241 / 第 220-241 行

```cpp
0220:   }
0221:   // NOTE [vmap through backward and undefined grad]
0222:   // While vmapping through backward functions (to compute batched grad), it
0223:   // is possible for the backward function to return an undefined grad for some
0224:   // grad_input for each example. In that case, we return an undefined grad.
0225:   //
0226:   // It is theoretically possible for *some* of the examples to produce an
0227:   // undefined grad (a kernel could peek at the gradient values and return an
0228:   // undefined tensor if it determines the gradient is full of zeros). We
0229:   // could handle this by treating the undefined grad as a zero-filled tensor
0230:   // of the correct shape while stacking the tensors together. However I expect
0231:   // this to happen very rarely (I have not been able to find an example in our
0232:   // codebase) so we just error out in this case.
0233:   if (std::none_of(tensors.begin(), tensors.end(), is_defined)) {
0234:     return Tensor();
0235:   }
0236:   TORCH_CHECK(false,
0237:       "vmap: slow fallback received a mix of undefined and defined tensors ",
0238:       "as the result of an operation. This is not supported, please file us ",
0239:       "an issue on github.");
0240: }
0241: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `Tensor`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`Tensor`。

### Lines 242-263 / 第 242-263 行

```cpp
0242: // TODO: Consider rewriting the following to look like:
0243: // https://gist.github.com/zou3519/7b7c6a4a258d580f62d1d969851be6b1<Paste>
0244: 
0245: // The general flow of the algorithm is as follows.
0246: // - First, we figure out which arguments are BatchedTensors and save them
0247: //   to a vector. We also store a vector of which index of the arguments list
0248: //   each BatchedTensor appears in. This will be useful for bookkeeping later.
0249: // - Next, we apply the MultiBatchVmapTransform to all of the BatchedTensors.
0250: //   This returns a vector of VmapPhysicalView that hold tensors that contain
0251: //   all of the collective batch dimensions at the front of the tensors.
0252: // - Then, we attempt to call `op` once per slice of the inputs. To do this,
0253: //   we repeatedly we slice the input arguments (if they are BatchedTensors),
0254: //   put the sliced (or a not-sliced) version of the input onto the stack, invoke
0255: //   the operator, and then pop the results off the stack.
0256: // - Each result obtained from the previous step is a slice of the total result,
0257: //   so we stack those tensors together to form the final result.
0258: void batchedTensorForLoopFallback(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
0259:   const auto& schema = op.schema();
0260:   const auto num_returns = schema.returns().size();
0261:   const auto num_arguments = schema.arguments().size();
0262:   const auto arguments = torch::jit::last(stack, num_arguments);
0263: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `batchedTensorForLoopFallback`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`batchedTensorForLoopFallback`。

### Lines 264-285 / 第 264-285 行

```cpp
0264:   TORCH_CHECK(areAllReturnsTensors(schema) && !areAnyArgumentsTensorList(schema),
0265:               "Batching rule not implemented for ", schema.operator_name(), ". ",
0266:               "We could not generate a fallback.");
0267: 
0268:   if (std::none_of(arguments.begin(), arguments.end(), ivalueParticipatesInCurrentLevel)) {
0269:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0270:     op.callBoxed(stack);
0271:     return;
0272:   }
0273: 
0274:   if (isInplaceOp(schema)) {
0275:     batchedTensorInplaceForLoopFallback(op, stack);
0276:     return;
0277:   }
0278:   TORCH_CHECK(!schema.is_mutable() && !schema.hasAnyAliasInfo(),
0279:               "Batching rule not implemented for ", schema.operator_name(), "; ",
0280:               "the fallback path doesn't work on out= or view ops.");
0281:   TORCH_CHECK(num_returns >= 1,
0282:               "Batching rule not implemented for ", schema.operator_name(), ". ",
0283:               "The fallback path does not support operations with no returns.");
0284:   warnFallback(schema, /*in_place*/false);
0285: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `guard`, `batchedTensorInplaceForLoopFallback`, `warnFallback`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`guard`, `batchedTensorInplaceForLoopFallback`, `warnFallback`。

### Lines 286-308 / 第 286-308 行

```cpp
0286:   const auto arguments_begin = stack->size() - num_arguments;
0287: 
0288:   // Figure out which arguments are BatchedTensor. Save them to a vector.
0289:   // For each BatchedTensor, also record what position of `arguments` they came from.
0290:   at::SmallVector<Tensor,kVmapTransformStaticInputSize> batched_tensor_inputs;
0291:   VmapDimVector batched_tensor_inputs_position;
0292:   for (const auto idx : c10::irange(0, arguments.size())) {
0293:     const auto& ivalue = arguments[idx];
0294:     if (!ivalue.isTensor()) {
0295:       continue;
0296:     }
0297:     const auto& tensor = ivalue.toTensor();
0298:     if (!tensor.defined()) {
0299:       continue;
0300:     }
0301:     const auto* batched = maybeGetBatchedImpl(tensor);
0302:     if (!batched) {
0303:       continue;
0304:     }
0305:     batched_tensor_inputs.push_back(tensor);
0306:     batched_tensor_inputs_position.push_back(static_cast<int64_t>(idx));
0307:   }
0308:   TORCH_INTERNAL_ASSERT(!batched_tensor_inputs.empty());
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 309-337 / 第 309-337 行

```cpp
0309: 
0310:   // MultiBatchVmapTransform the BatchedTensor arguments. This returns
0311:   // VmapPhysicalViews that contain all of the batch dimensions.
0312:   const auto input_physical_views = MultiBatchVmapTransform::logicalToPhysical(
0313:       batched_tensor_inputs);
0314: 
0315:   // Compute the total number of batches
0316:   auto num_batch_dims = input_physical_views.front().numBatchDims();
0317:   auto some_sizes = input_physical_views.front().tensor().sizes();
0318:   auto batch_sizes = ArrayRef<int64_t>(some_sizes.begin(), some_sizes.begin() + num_batch_dims);
0319:   const auto num_batches = c10::multiply_integers(batch_sizes);
0320:   // Without a shape-checking API, we're unable to compute the correct shape of
0321:   // the output so we just error out.
0322:   TORCH_CHECK(num_batches > 0,
0323:       "Batching rule not implemented for ", schema.operator_name(), ". ",
0324:       "The fallback path does not support vmap over dims of size 0.");
0325: 
0326:   // Strategy: For each batch, we are going to push slices (where applicable)
0327:   // of the arguments onto `stack`, call `op`, and store the result in
0328:   // `output_shards`.
0329:   //
0330:   // NOTE: [Output shards layout]
0331:   // Assume that the operator has three outputs: a, b, c.
0332:   // The layout of output_shards is as follows:
0333:   // [ a0, a1, a2, a3, b0, b1, b2, b3, c0, c1, c2, c3]
0334:   // This is so that we can call at::stack([a0...a3]), at::stack([b0...b3])
0335:   // more easily in the next step.
0336:   std::vector<Tensor> output_shards(num_batches * num_returns);
0337: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `output_shards`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`output_shards`。

### Lines 338-360 / 第 338-360 行

```cpp
0338:   for (int64_t linear_idx = 0; linear_idx < num_batches; ++linear_idx) {
0339:     auto index = computeIndex(linear_idx, batch_sizes);
0340:     auto batched_tensor_inputs_pos_iter = batched_tensor_inputs_position.begin();
0341:     auto input_physical_views_iter = input_physical_views.begin();
0342:     for (const auto arg_idx : c10::irange(0, num_arguments)) {
0343:       // We assume that torch::jit::Stack is backed by vector<IValue> for
0344:       // simplicity. When that is not the case, this code should be updated.
0345:       const auto& argument = (*stack)[arguments_begin + arg_idx];
0346:       if (batched_tensor_inputs_pos_iter == batched_tensor_inputs_position.end()
0347:           || static_cast<int64_t>(arg_idx) != *batched_tensor_inputs_pos_iter) {
0348:         // argument isn't a BatchedTensor
0349:         torch::jit::push(stack, argument);
0350:         continue;
0351:       }
0352:       // argument is a BatchedTensor
0353:       TORCH_INTERNAL_ASSERT(input_physical_views_iter != input_physical_views.end());
0354:       const auto& physical_view_for_argument = *input_physical_views_iter;
0355:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0356:       torch::jit::push(stack, physical_view_for_argument.tensor().index(index));
0357:       batched_tensor_inputs_pos_iter++;
0358:       input_physical_views_iter++;
0359:     }
0360: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `push`, `guard`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`push`, `guard`。

### Lines 361-383 / 第 361-383 行

```cpp
0361:     // std::cout << "[Fallback]: ";
0362:     // at::dump_tensor((*stack)[stack->size() - 1].toTensor());
0363:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0364:     op.callBoxed(stack);
0365: 
0366:     // Store the result into `output_shards`. See NOTE: [Output shards layout]
0367:     // to learn about the details of how we store the shards.
0368:     const auto returns = torch::jit::last(stack, num_returns);
0369:     for (const auto  return_idx : c10::irange(0, returns.size())) {
0370:       output_shards[num_batches * return_idx + linear_idx] = returns[return_idx].toTensor();
0371:     }
0372:     torch::jit::drop(stack, num_returns);
0373:   }
0374: 
0375:   // For each output Tensor, stack the shards of the tensor together to form a return
0376:   torch::jit::drop(stack, num_arguments);
0377:   auto output_shards_chunks = MatrixRef<Tensor>(output_shards, num_batches);
0378:   for (const auto return_idx : c10::irange(0, num_returns)) {
0379:     auto shards = output_shards_chunks[return_idx];
0380:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0381:     auto flat_output = safeStack(shards);
0382:     // See NOTE [vmap through backward and undefined grad]
0383:     if (!flat_output.defined()) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `guard`, `drop`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`guard`, `drop`。

### Lines 384-403 / 第 384-403 行

```cpp
0384:       torch::jit::push(stack, flat_output);
0385:       continue;
0386:     }
0387:     VmapDimVector output_sizes(batch_sizes);
0388:     output_sizes.insert(
0389:         output_sizes.end(),
0390:         flat_output.sizes().begin() + 1,
0391:         flat_output.sizes().end());
0392:     torch::jit::push(
0393:         stack,
0394:         input_physical_views.front().getPhysicalToLogicalMap().apply(flat_output.view(output_sizes)));
0395:   }
0396: }
0397: 
0398: void batchedNestedTensorForLoopFallback(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
0399:   const auto& schema = op.schema();
0400:   const auto num_returns = schema.returns().size();
0401:   const auto num_arguments = schema.arguments().size();
0402:   const auto arguments = torch::jit::last(stack, num_arguments);
0403: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `push`, `output_sizes`, `batchedNestedTensorForLoopFallback`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`push`, `output_sizes`, `batchedNestedTensorForLoopFallback`。

### Lines 404-424 / 第 404-424 行

```cpp
0404:   TORCH_CHECK(areAllReturnsTensors(schema) && !areAnyArgumentsTensorList(schema),
0405:               "Nested batching rule not implemented for ", schema.operator_name(), ". ",
0406:               "We could not generate a fallback.");
0407: 
0408:   if (std::none_of(arguments.begin(), arguments.end(), ivalueParticipatesInCurrentLevel)) {
0409:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0410:     c10::impl::ExcludeDispatchKeyGuard nt_guard(DispatchKey::BatchedNestedTensor);
0411:     op.callBoxed(stack);
0412:     return;
0413:   }
0414: 
0415:   TORCH_INTERNAL_ASSERT(!isInplaceOp(schema), "vmap fallback not supported for in-place ops on nested tensors");
0416: 
0417:   TORCH_CHECK(!schema.is_mutable() && !schema.hasAnyAliasInfo(),
0418:               "Nested batching rule not implemented for ", schema.operator_name(), "; ",
0419:               "the fallback path doesn't work on out= or view ops.");
0420:   TORCH_CHECK(num_returns >= 1,
0421:               "Nested batching rule not implemented for ", schema.operator_name(), ". ",
0422:               "The fallback path does not support operations with no returns.");
0423:   warnFallback(schema, /*in_place*/false, /*is_nested*/true);
0424: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `guard`, `nt_guard`, `warnFallback`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`guard`, `nt_guard`, `warnFallback`。

### Lines 425-447 / 第 425-447 行

```cpp
0425:   const auto arguments_begin = stack->size() - num_arguments;
0426: 
0427:   // Figure out which arguments are BatchedTensor. Save them to a vector.
0428:   // For each BatchedTensor, also record what position of `arguments` they came from.
0429:   at::SmallVector<Tensor,kVmapTransformStaticInputSize> batched_tensor_inputs;
0430:   VmapDimVector batched_tensor_inputs_position;
0431:   for (const auto idx : c10::irange(0, arguments.size())) {
0432:     const auto& ivalue = arguments[idx];
0433:     if (!ivalue.isTensor()) {
0434:       continue;
0435:     }
0436:     const auto& tensor = ivalue.toTensor();
0437:     if (!tensor.defined()) {
0438:       continue;
0439:     }
0440:     const auto* batched = maybeGetBatchedImpl(tensor);
0441:     if (!batched) {
0442:       continue;
0443:     }
0444:     batched_tensor_inputs.push_back(tensor);
0445:     batched_tensor_inputs_position.push_back(static_cast<int64_t>(idx));
0446:   }
0447:   TORCH_INTERNAL_ASSERT(!batched_tensor_inputs.empty());
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 448-468 / 第 448-468 行

```cpp
0448: 
0449:   std::vector<std::vector<Tensor>> unbound;
0450:   for (auto const &batched_tensor_input: batched_tensor_inputs) {
0451:     auto *batched_impl = maybeGetBatchedImpl(batched_tensor_input);
0452:     TORCH_INTERNAL_ASSERT(batched_impl->value().is_nested() || batched_impl->bdim() == 0,
0453:         "Fallback not supported for mixed nested / non-nested arguments without bdim=0");
0454:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::BatchedNestedTensor);
0455:     auto this_unbound = batched_impl->value().unbind();
0456:     if (!unbound.empty()) {
0457:       TORCH_INTERNAL_ASSERT(unbound.front().size() == this_unbound.size(),
0458:           "Fallback not supported for differently-sized nested arguments");
0459:     }
0460:     unbound.push_back(this_unbound);
0461:   }
0462: 
0463:   const auto num_components = unbound.front().size();
0464:   std::vector<Tensor> output_shards(num_components * num_returns);
0465:   for (const auto component_idx : c10::irange(0, num_components)) {
0466:     auto batched_idx = 0;
0467:     auto batched_tensor_inputs_pos_iter = batched_tensor_inputs_position.begin();
0468:     for (const auto arg_idx : c10::irange(0, num_arguments)) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `guard`, `output_shards`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`guard`, `output_shards`。

### Lines 469-490 / 第 469-490 行

```cpp
0469:       // We assume that torch::jit::Stack is backed by vector<IValue> for
0470:       // simplicity. When that is not the case, this code should be updated.
0471:       const auto& argument = (*stack)[arguments_begin + arg_idx];
0472:       if (batched_tensor_inputs_pos_iter == batched_tensor_inputs_position.end()
0473:           || static_cast<int64_t>(arg_idx) != *batched_tensor_inputs_pos_iter) {
0474:         // argument isn't a BatchedTensor
0475:         torch::jit::push(stack, argument);
0476:         continue;
0477:       }
0478:       // argument is a BatchedTensor
0479:       torch::jit::push(stack, unbound[batched_idx][component_idx]);
0480:       ++batched_idx;
0481:       ++batched_tensor_inputs_pos_iter;
0482:     }
0483: 
0484:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::BatchedNestedTensor);
0485:     op.callBoxed(stack);
0486: 
0487:     // Store the result into `output_shards`. See NOTE: [Output shards layout]
0488:     // to learn about the details of how we store the shards.
0489:     const auto returns = torch::jit::last(stack, num_returns);
0490:     for (const auto return_idx : c10::irange(0, returns.size())) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values. Key symbols: `push`, `guard`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值。关键符号：`push`, `guard`。

### Lines 491-510 / 第 491-510 行

```cpp
0491:       output_shards[num_components * return_idx + component_idx] = returns[return_idx].toTensor();
0492:     }
0493:     torch::jit::drop(stack, num_returns);
0494:   }
0495: 
0496:   // For each output Tensor, stack the shards of the tensor together to form a nested return
0497:   // TODO: Determine when the output needs to be nested and when it can be non-nested?
0498:   torch::jit::drop(stack, num_arguments);
0499:   auto output_shards_chunks = MatrixRef<Tensor>(output_shards, num_components);
0500:   for (const auto return_idx : c10::irange(0, num_returns)) {
0501:     auto shards = output_shards_chunks[return_idx];
0502:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::BatchedNestedTensor);
0503:     auto out_nt = at::_nested_tensor_from_tensor_list(shards);
0504:     // NB: NTs only support batching over dim 0
0505:     torch::jit::push(stack, makeBatched(out_nt, 0, maybeCurrentDynamicLayer()->layerId()));
0506:   }
0507: }
0508: 
0509: void vmapErrorFallback(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
0510:   TORCH_CHECK(false, "Error: ", op.operator_name(), " requires special handling, and does not yet have a batching rule. Feel free to file a github issue!");
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `drop`, `guard`, `push`, `vmapErrorFallback`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`drop`, `guard`, `push`, `vmapErrorFallback`。

### Lines 511-513 / 第 511-513 行

```cpp
0511: }
0512: 
0513: } // namespace at::functorch
```

- **EN:** Opens or closes namespace scopes so the remaining declarations stay inside the intended ATen/backend module.
- **CN:** 打开或关闭命名空间作用域，使后续声明保持在预期的 ATen/后端模块之内。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchedFallback.h`, `ATen/functorch/LegacyVmapTransforms.h`, `ATen/functorch/DynamicLayer.h`, `ATen/functorch/PlumbingHelper.h`, `ATen/Context.h`, `ATen/MatrixRef.h`, `ATen/core/dispatch/Dispatcher.h`, `c10/util/accumulate.h`, `c10/util/llvmMathExtras.h`, `c10/util/irange.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `isVmapFallbackWarningEnabled`, `setVmapFallbackWarningEnabled`, `isVmapFallbackEnabled`, `setVmapFallbackEnabled`, `computeIndex`, `reverse`, `areAllReturnsTensors`, `all_of`, `areAnyArgumentsTensorList`, `any_of`, `warnFallback`, `batchedTensorInplaceForLoopFallback`, `...`
