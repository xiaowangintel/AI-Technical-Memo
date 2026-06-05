# BatchRulesRandomness.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesRandomness.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesRandomness.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Random-number generation or reproducibility semantics are explicitly encoded. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesRandomness.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件显式编码了随机数生成或可复现性语义。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/DynamicLayer.h>
0008: #include <ATen/functorch/BatchRulesHelper.h>
0009: 
0010: #include <utility>
0011: 
0012: // This file contains batching rules for random operations. These are different
0013: // from our regular batching rules: regular batching rules get registered to the
0014: // FuncTorchBatched key, but batching rules for random operations get
0015: // registered to FuncTorchVmapMode. This is because we need to interpose on
0016: // random operations even if they're not on a BatchedTensor.
0017: 
0018: // NOLINTBEGIN(bugprone-unchecked-optional-access)
0019: namespace at::functorch {
0020: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 21-42 / 第 21-42 行

```cpp
0021: template <typename F, F Func, typename... ExtraArgs>
0022: static Tensor random_batching_rule(SymIntArrayRef shape, ExtraArgs... extra_args) {
0023:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0024:   auto maybe_layer = maybeCurrentDynamicLayer();
0025:   TORCH_INTERNAL_ASSERT(maybe_layer.has_value());
0026:   c10::SmallVector<SymInt> shapeVec(1, maybe_layer->batchSize());
0027:   shapeVec.reserve(shape.size() + 1);
0028:   shapeVec.insert(shapeVec.end(), shape.begin(), shape.end());
0029:   RandomnessType randomness = maybe_layer->randomness();
0030:   check_randomness(randomness);
0031:   if (randomness == RandomnessType::Different) {
0032:     return makeBatched(Func(shapeVec, std::forward<ExtraArgs>(extra_args)...), 0, maybe_layer->layerId());
0033:   } else {
0034:     return Func(shape, std::forward<ExtraArgs>(extra_args)...);
0035:   }
0036: }
0037: 
0038: template <typename F, F Func, typename... ExtraArgs>
0039: static Tensor& random_inplace_batching_rule(Tensor& self, ExtraArgs... extra_args) {
0040:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0041:   auto maybe_layer = maybeCurrentDynamicLayer();
0042:   TORCH_INTERNAL_ASSERT(maybe_layer.has_value());
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesRandomness`. Key symbols: `random_batching_rule`, `guard`, `shapeVec`, `check_randomness`, `makeBatched`, `Func`, `random_inplace_batching_rule`.
- **CN:** 围绕 `BatchRulesRandomness` 构建可复用的模板或辅助层。关键符号：`random_batching_rule`, `guard`, `shapeVec`, `check_randomness`, `makeBatched`, `Func`, `random_inplace_batching_rule`。

### Lines 43-62 / 第 43-62 行

```cpp
0043:   const auto cur_level = maybe_layer->layerId();
0044:   auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0045:   self_value = moveBatchDimToFront(std::move(self_value), self_bdim);
0046:   RandomnessType randomness = maybe_layer->randomness();
0047:   check_randomness(randomness);
0048:   TORCH_CHECK(
0049:     !(randomness == RandomnessType::Different && !self_bdim),
0050:     "vmap: Cannot ask for different inplace randomness on an unbatched tensor. This will appear like same randomness. ",
0051:     "If this is necessary for your usage, please file an issue with functorch.");
0052:   if (randomness == RandomnessType::Same && self_bdim) {
0053:     auto intermediate = empty(self.sizes(), self.options());
0054:     Func(intermediate, std::forward<ExtraArgs>(extra_args)...);
0055:     self.copy_(intermediate); // batching should make this just work out...
0056:     return self;
0057:   } else {
0058:     Func(self_value, std::forward<ExtraArgs>(extra_args)...);
0059:     return self;
0060:   }
0061: }
0062: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: `check_randomness`, `Func`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：`check_randomness`, `Func`。

### Lines 63-84 / 第 63-84 行

```cpp
0063: static Tensor& bernoulli_inplace_Tensor_batching_rule(Tensor& self, const Tensor& p_, std::optional<Generator> gen) {
0064:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0065:   auto maybe_layer = maybeCurrentDynamicLayer();
0066:   TORCH_INTERNAL_ASSERT(maybe_layer.has_value());
0067:   auto cur_level = maybe_layer->layerId();
0068:   RandomnessType randomness = maybe_layer->randomness();
0069: 
0070:   auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0071: 
0072:   auto [other_value, other_bdim] = unwrapTensorAtLevel(p_, cur_level);
0073: 
0074:   check_randomness(randomness, other_bdim.has_value());
0075: 
0076:   if (!self_bdim && other_bdim) {
0077:     vmapIncompatibleInplaceError("inplace bernoulli");
0078:   }
0079: 
0080:   // compute max logical rank
0081:   auto self_logical_rank = rankWithoutBatchDim(self_value, self_bdim);
0082:   auto other_logical_rank = rankWithoutBatchDim(other_value, other_bdim);
0083:   auto max_logical_rank = std::max(self_logical_rank, other_logical_rank);
0084: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: `bernoulli_inplace_Tensor_batching_rule`, `guard`, `check_randomness`, `vmapIncompatibleInplaceError`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：`bernoulli_inplace_Tensor_batching_rule`, `guard`, `check_randomness`, `vmapIncompatibleInplaceError`。

### Lines 85-105 / 第 85-105 行

```cpp
0085:   auto self_ = moveBatchDimToFront(self_value, self_bdim);
0086:   auto other_ = moveBatchDimToFront(other_value, other_bdim);
0087: 
0088:   // If the dimensions aren't aligned, we need to line them up.
0089:   // Tensor[B, 3] + Tensor[2, 5, 3] -> Tensor[B, 1, 1, 3] + Tensor[2, 5, 3]
0090:   // Note that only tensors that have a batch dim need to be modified.
0091:   // Tensor[B, 2, 3, 5] + Tensor[5] -> no changes needed
0092:   self_ = maybePadToLogicalRank(self_, self_bdim, max_logical_rank);
0093:   other_ = maybePadToLogicalRank(other_, other_bdim, max_logical_rank);
0094:   TORCH_CHECK(
0095:     !(randomness == RandomnessType::Different && !self_bdim),
0096:     "vmap: Cannot ask for different inplace randomness on an unbatched tensor. This will appear like same randomness. ",
0097:     "If this is necessary for your usage, please file an issue with functorch.");
0098:   if (randomness == RandomnessType::Same && self_bdim) {
0099:     auto intermediate = empty(self.sizes(), self.options());
0100:     intermediate.bernoulli_(other_, std::move(gen));
0101:     self.copy_(intermediate); // batching should make this just work out...
0102:     return self;
0103:   } else {
0104:     self_.bernoulli_(other_, std::move(gen));
0105:     return self;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 106-127 / 第 106-127 行

```cpp
0106:   }
0107: }
0108: 
0109: template <typename F, F Func, typename... ExtraArgs>
0110: static Tensor randperm_batching_rule(int64_t n, ExtraArgs... extra_args) {
0111:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0112:   auto maybe_layer = maybeCurrentDynamicLayer();
0113:   auto const batch_size = maybe_layer->batchSize();
0114:   RandomnessType randomness = maybe_layer->randomness();
0115:   check_randomness(randomness);
0116:   if (randomness == RandomnessType::Different) {
0117:     std::vector<at::Tensor> stackedList(batch_size.guard_int(__FILE__, __LINE__));
0118:     for (int64_t idx = 0; idx < batch_size; ++idx) {
0119:       // since this is done in a loop, need to pass by reference for generator to update
0120:       stackedList[idx] = Func(n, extra_args...);
0121:     }
0122:     return makeBatched(at::stack(stackedList), 0, maybe_layer->layerId());
0123:   } else {
0124:     return Func(n, std::forward<ExtraArgs>(extra_args)...);
0125:   }
0126: }
0127: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesRandomness`. Key symbols: `randperm_batching_rule`, `guard`, `check_randomness`, `stackedList`, `makeBatched`, `Func`.
- **CN:** 围绕 `BatchRulesRandomness` 构建可复用的模板或辅助层。关键符号：`randperm_batching_rule`, `guard`, `check_randomness`, `stackedList`, `makeBatched`, `Func`。

### Lines 128-148 / 第 128-148 行

```cpp
0128: template <typename F, F Func, typename... ExtraArgs>
0129: static Tensor unary_pointwise_random_batch_rule(const Tensor& tensor, ExtraArgs... extra_args) {
0130:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0131:   auto maybe_layer = maybeCurrentDynamicLayer();
0132:   const auto cur_level = maybe_layer->layerId();
0133: 
0134:   auto [tensor_value, tensor_bdim] = unwrapTensorAtLevel(tensor, cur_level);
0135:   tensor_value = moveBatchDimToFront(tensor_value, tensor_bdim);
0136: 
0137:   RandomnessType randomness = maybe_layer->randomness();
0138:   check_randomness(randomness, tensor_bdim.has_value());
0139:   auto shape = tensor_value.sizes();
0140:   VmapSymDimVector shapeVec(1, maybe_layer->batchSize());
0141:   shapeVec.reserve(shape.size() + 1);
0142:   shapeVec.insert(shapeVec.end(), shape.begin(), shape.end());
0143: 
0144:   if (randomness == RandomnessType::Different && !tensor_bdim) {
0145:     tensor_value = tensor_value.expand_symint(shapeVec);
0146:   }
0147:   auto out = Func(tensor_value, std::forward<ExtraArgs>(extra_args)...);
0148:   if (randomness == RandomnessType::Same && !tensor_bdim) {
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesRandomness`. Key symbols: `unary_pointwise_random_batch_rule`, `guard`, `check_randomness`, `shapeVec`.
- **CN:** 围绕 `BatchRulesRandomness` 构建可复用的模板或辅助层。关键符号：`unary_pointwise_random_batch_rule`, `guard`, `check_randomness`, `shapeVec`。

### Lines 149-174 / 第 149-174 行

```cpp
0149:     return out;
0150:   }
0151:   return makeBatched(out, 0, cur_level);
0152: }
0153: 
0154: template<typename F, F Func, typename... ExtraArgs>
0155: static Tensor tensor_like_random_batch_rule(const Tensor& self, ExtraArgs... extra_args) {
0156:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0157:   auto maybe_layer = maybeCurrentDynamicLayer();
0158:   const auto cur_level = maybe_layer->layerId();
0159:   RandomnessType randomness = maybe_layer->randomness();
0160:   check_randomness(randomness);
0161: 
0162:   auto [tensor_value, tensor_bdim] = unwrapTensorAtLevel(self, cur_level);
0163:   tensor_value = moveBatchDimToFront(tensor_value, tensor_bdim);
0164: 
0165:   if (randomness == RandomnessType::Same && tensor_bdim) {
0166:     tensor_value = tensor_value[0];
0167:   } else if (randomness == RandomnessType::Different && !tensor_bdim) {
0168:     auto shape = tensor_value.sizes();
0169:     VmapSymDimVector shapeVec(1, maybe_layer->batchSize());
0170:     shapeVec.reserve(shape.size() + 1);
0171:     shapeVec.insert(shapeVec.end(), shape.begin(), shape.end());
0172:     tensor_value = tensor_value.expand_symint(shapeVec);
0173:   }
0174: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesRandomness`. Key symbols: `makeBatched`, `tensor_like_random_batch_rule`, `guard`, `check_randomness`, `shapeVec`.
- **CN:** 围绕 `BatchRulesRandomness` 构建可复用的模板或辅助层。关键符号：`makeBatched`, `tensor_like_random_batch_rule`, `guard`, `check_randomness`, `shapeVec`。

### Lines 175-194 / 第 175-194 行

```cpp
0175:   auto res = Func(tensor_value, std::forward<ExtraArgs>(extra_args)...);
0176:   return (randomness == RandomnessType::Same) ? res : makeBatched(res, 0, cur_level);
0177: }
0178: 
0179: static std::tuple<Tensor,Tensor> native_dropout_batching_rule(const Tensor& tensor, double p, std::optional<bool> train) {
0180:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0181:   auto maybe_layer = maybeCurrentDynamicLayer();
0182:   const auto cur_level = maybe_layer->layerId();
0183:   RandomnessType randomness = maybe_layer->randomness();
0184: 
0185:   auto [tensor_value, tensor_bdim] = unwrapTensorAtLevel(tensor, cur_level);
0186:   tensor_value = moveBatchDimToFront(tensor_value, tensor_bdim);
0187: 
0188:   if (!train.has_value() || *train) {
0189:     check_randomness(randomness); // if we are in eval mode, we don't use about randomness
0190:   }
0191: 
0192:   if ((train.has_value() && !*train) ||
0193:       randomness == RandomnessType::Different) {
0194:     if (!tensor_bdim) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: `native_dropout_batching_rule`, `guard`, `check_randomness`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：`native_dropout_batching_rule`, `guard`, `check_randomness`。

### Lines 195-216 / 第 195-216 行

```cpp
0195:       // if tensor is unbatched, add batch dim before
0196:       // calling dropout.
0197:       auto shape = tensor_value.sizes();
0198:       VmapSymDimVector shapeVec(1, maybe_layer->batchSize());
0199:       shapeVec.reserve(shape.size() + 1);
0200:       shapeVec.insert(shapeVec.end(), shape.begin(), shape.end());
0201:       tensor_value = tensor_value.expand_symint(shapeVec);
0202:     }
0203:     auto [output, mask] = at::native_dropout(tensor_value, p, train);
0204:     return std::make_tuple(
0205:         makeBatched(output, 0, cur_level),
0206:         makeBatched(mask, 0, cur_level));
0207:   }
0208: 
0209:   // repeated code from the CPU kernel since the CUDA one doesn't call bernoulli_ explicitly
0210:   double p1m = 1. - p;
0211:   // Check for probability of zero to avoid divide by zero and NaN results
0212:   double scale = p1m == 0 ? 0. : 1. / p1m;
0213:   Tensor mask = at::empty_like(tensor, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
0214:   mask.bernoulli_(p1m);
0215:   const auto output = tensor.mul(mask).mul_(scale);
0216:   return std::make_tuple(output, mask);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; supports transform-aware functorch semantics. Key symbols: `shapeVec`, `make_tuple`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；支持面向变换的 functorch 语义。关键符号：`shapeVec`, `make_tuple`。

### Lines 217-242 / 第 217-242 行

```cpp
0217: }
0218: 
0219: static Tensor native_dropout_backward_batch_rule(const Tensor& grad_out, const Tensor& mask, double scale){
0220:   Tensor result = grad_out * mask * scale;
0221:   return result;
0222: }
0223: 
0224: static Tensor multinomial_batching_rule(const Tensor& self, const int64_t num_samples, const bool replacement, std::optional<Generator> generator) {
0225:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0226:   auto maybe_layer = maybeCurrentDynamicLayer();
0227:   const auto cur_level = maybe_layer->layerId();
0228: 
0229:   auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0230:   self_value = moveBatchDimToFront(self_value, self_bdim);
0231: 
0232:   RandomnessType randomness = maybe_layer->randomness();
0233:   check_randomness(randomness, self_bdim.has_value());
0234: 
0235:   if (randomness == RandomnessType::Different) {
0236:     // 1D cases: S -> BS -> multinomial(BS)
0237:     //           BS -> multinomial(BS)
0238:     //
0239:     // 2D cases: MS -> BMS -> (BM)S -> multinomial((BM)S) -> (BM)S -> BMS
0240:     //           BMS -> (BM)S -> multinomial((BM)S) -> (BM)S -> BMS
0241:     const auto is_2D_case = rankWithoutBatchDim(self_value, self_bdim) == 2;
0242:     if (!self_bdim.has_value()) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: `native_dropout_backward_batch_rule`, `multinomial_batching_rule`, `guard`, `check_randomness`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：`native_dropout_backward_batch_rule`, `multinomial_batching_rule`, `guard`, `check_randomness`。

### Lines 243-262 / 第 243-262 行

```cpp
0243:       self_value = ensure_has_bdim(self_value, self_bdim.has_value(), maybe_layer->batchSize());
0244:     }
0245:     if (is_2D_case) {
0246:       self_value = reshape_dim_into(0, 0, self_value);
0247:     }
0248:     auto out = multinomial(self_value, num_samples, replacement, std::move(generator));
0249:     if (is_2D_case) {
0250:       out = reshape_dim_outof_symint(0, maybe_layer->batchSize(), out);
0251:     }
0252:     return makeBatched(out, 0, cur_level);;
0253:   }
0254: 
0255:   TORCH_INTERNAL_ASSERT(randomness == RandomnessType::Same); // check_randomness eliminates error randomness
0256:   TORCH_INTERNAL_ASSERT(!self_bdim.has_value()); // check_randomness eliminates same randomness with batched input
0257:   // Must be same randomness with unbatched input
0258:   // 1D case: S -> multinomial(S) -> S
0259:   // 2D case: MS -> multinomial(MS) -> MS
0260:   return multinomial(self_value, num_samples, replacement, std::move(generator));
0261: }
0262: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; validates runtime invariants before continuing; encodes random-number generation or reproducibility semantics. Key symbols: `makeBatched`, `multinomial`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；在继续执行前校验运行时不变量；编码随机数生成或可复现性语义。关键符号：`makeBatched`, `multinomial`。

### Lines 263-282 / 第 263-282 行

```cpp
0263: template <typename A, A a, typename C>
0264: struct RandomBatchRuleHelper;
0265: 
0266: template <typename F, F Func, typename T1, typename... T>
0267: struct RandomBatchRuleHelper<F, Func, typelist<T1, T...>> {
0268:   static Tensor apply(SymIntArrayRef shape, T... extra_args) {
0269:     return random_batching_rule<F, Func, T...>(shape, std::forward<T>(extra_args)...);
0270:   }
0271: };
0272: 
0273: template <typename F, F Func, typename... T>
0274: static Tensor rand_int_wrapper(SymIntArrayRef shape, c10::SymInt high, T... extra_args) {
0275:   return Func(high, shape, std::forward<T>(extra_args)...);
0276: }
0277: 
0278: template <typename A, A a, typename C>
0279: struct RandomInplaceBatchRuleHelper;
0280: 
0281: template <typename F, F Func, typename T1, typename... T>
0282: struct RandomInplaceBatchRuleHelper<F, Func, typelist<T1, T...>> {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RandomBatchRuleHelper`, `RandomInplaceBatchRuleHelper`, `apply`, `rand_int_wrapper`, `Func`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RandomBatchRuleHelper`, `RandomInplaceBatchRuleHelper`, `apply`, `rand_int_wrapper`, `Func`。

### Lines 283-302 / 第 283-302 行

```cpp
0283:   static Tensor& apply(Tensor& self, T... extra_args) {
0284:     return random_inplace_batching_rule<F, Func, T...>(self, std::forward<T>(extra_args)...);
0285:   }
0286: };
0287: 
0288: template <typename A, A a, typename C>
0289: struct RandIntBatchRuleHelper;
0290: 
0291: template <typename F, F Func, typename T1, typename T2, typename... T>
0292: struct RandIntBatchRuleHelper<F, Func, typelist<T1, T2, T...>> {
0293:   static Tensor apply(c10::SymInt high, SymIntArrayRef shape, T... extra_args) {
0294:     return random_batching_rule<decltype(&rand_int_wrapper<F, Func, T...>),
0295:                                 &rand_int_wrapper<F, Func, T...>,
0296:                                 c10::SymInt, T...>(shape, std::move(high), std::forward<T>(extra_args)...);
0297:   }
0298: };
0299: 
0300: template <typename F, F Func, typename T0, typename T1, typename... T>
0301: static Tensor rand_int_low_wrapper(SymIntArrayRef shape, T0 scalar0, T1 scalar1, T... extra_args) {
0302:   return Func(scalar0, scalar1, shape, std::forward<T>(extra_args)...);
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RandIntBatchRuleHelper`, `apply`, `rand_int_low_wrapper`, `Func`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RandIntBatchRuleHelper`, `apply`, `rand_int_low_wrapper`, `Func`。

### Lines 303-322 / 第 303-322 行

```cpp
0303: }
0304: 
0305: template <typename A, A a, typename C>
0306: struct RandTwoLeadingScalarsBatchRuleHelper;
0307: 
0308: template <typename F, F Func, typename T0, typename T1, typename T2, typename... T>
0309: struct RandTwoLeadingScalarsBatchRuleHelper<F, Func, typelist<T0, T1, T2, T...>> {
0310:   static Tensor apply(T0 scalar0, T1 scalar1, SymIntArrayRef shape, T... extra_args) {
0311:     return random_batching_rule<decltype(&rand_int_low_wrapper<F, Func, T0, T1, T...>),
0312:                                 &rand_int_low_wrapper<F, Func, T0, T1, T...>,
0313:                                 T0, T1, T...>(shape, scalar0, scalar1, std::forward<T>(extra_args)...);
0314:   }
0315: };
0316: 
0317: template <typename A, A a, typename C>
0318: struct RandpermBatchRuleHelper;
0319: 
0320: template <typename F, F Func, typename T1, typename... T>
0321: struct RandpermBatchRuleHelper<F, Func, typelist<T1, T...>> {
0322:   static Tensor apply(int64_t n, T... extra_args) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RandTwoLeadingScalarsBatchRuleHelper`, `RandpermBatchRuleHelper`, `apply`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RandTwoLeadingScalarsBatchRuleHelper`, `RandpermBatchRuleHelper`, `apply`。

### Lines 323-342 / 第 323-342 行

```cpp
0323:     return randperm_batching_rule<F, Func, T...>(n, std::forward<T>(extra_args)...);
0324:   }
0325: };
0326: 
0327: template <typename A, A a, typename C>
0328: struct UnaryPointwiseRandomBatchRule;
0329: 
0330: template <typename F, F Func, typename A0, typename... T>
0331: struct UnaryPointwiseRandomBatchRule<F, Func, typelist<A0, T...>> {
0332:   static Tensor apply(const Tensor& tensor, T... extra_args) {
0333:     return unary_pointwise_random_batch_rule<F, Func, T...>(tensor, std::forward<T>(extra_args)...);
0334:   }
0335: };
0336: 
0337: template <typename A, A a, typename C>
0338: struct NormalPointwiseBatchRule;
0339: 
0340: template <typename F, F Func, typename A0, typename... T>
0341: struct NormalPointwiseBatchRule<F, Func, typelist<A0, T...>> {
0342:   static Tensor apply(const Tensor& tensor, T... extra_args) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `UnaryPointwiseRandomBatchRule`, `NormalPointwiseBatchRule`, `apply`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`UnaryPointwiseRandomBatchRule`, `NormalPointwiseBatchRule`, `apply`。

### Lines 343-362 / 第 343-362 行

```cpp
0343:     return unary_pointwise_random_batch_rule<F, Func, T...>(tensor, std::forward<T>(extra_args)...);
0344:   }
0345: };
0346: 
0347: template<typename F, F Func, typename... T>
0348: static Tensor normal_wrapper(const Tensor& tensor, double scalar, T... extra_args) {
0349:   return Func(scalar, tensor, extra_args...);
0350: }
0351: 
0352: template <typename A, A a, typename C>
0353: struct UnaryPointwiseRandomLeadingFloatBatchRule;
0354: 
0355: template <typename F, F Func, typename A0, typename A1, typename... T>
0356: struct UnaryPointwiseRandomLeadingFloatBatchRule<F, Func, typelist<A0, A1, T...>> {
0357:   static Tensor apply(double scalar, const Tensor& tensor, T... extra_args) {
0358:     return unary_pointwise_random_batch_rule<decltype(&normal_wrapper<F, Func, T...>),
0359:                                          &normal_wrapper<F, Func, T...>, double,
0360:                                          T...>(tensor, scalar, std::forward<T>(extra_args)...);
0361:   }
0362: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `UnaryPointwiseRandomLeadingFloatBatchRule`, `normal_wrapper`, `Func`, `apply`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`UnaryPointwiseRandomLeadingFloatBatchRule`, `normal_wrapper`, `Func`, `apply`。

### Lines 363-385 / 第 363-385 行

```cpp
0363: 
0364: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0365:   #define RANDOM_INPLACE_BATCH_RULE2(op, overload) \
0366:     m.impl(#op"."#overload, SINGLE_ARG(\
0367:       RandomInplaceBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0368:                             c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0369: 
0370:   RANDOM_INPLACE_BATCH_RULE2(bernoulli_, float);
0371: 
0372:   #undef RANDOM_INPLACE_BATCH_RULE2
0373: }
0374: 
0375: TORCH_LIBRARY_IMPL(aten, FuncTorchVmapMode, m) {
0376:   #define RANDOM_BATCH_RULE(op) \
0377:     m.impl(#op, SINGLE_ARG(\
0378:       RandomBatchRuleHelper<decltype(&ATEN_FN(op)), &ATEN_FN(op), \
0379:                             c10::guts::function_traits<decltype(ATEN_FN(op))>::parameter_types>::apply))
0380: 
0381:   #define RANDOM_BATCH_RULE2(op, overload) \
0382:     m.impl(#op"."#overload, SINGLE_ARG(\
0383:       RandomBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0384:                             c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0385: 
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesRandomness` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesRandomness` 的行为。符号：无明显局部符号。

### Lines 386-405 / 第 386-405 行

```cpp
0386:   #define RANDOM_INPLACE_BATCH_RULE(op) \
0387:     m.impl(#op, SINGLE_ARG(\
0388:       RandomInplaceBatchRuleHelper<decltype(&ATEN_FN(op)), &ATEN_FN(op), \
0389:                             c10::guts::function_traits<decltype(ATEN_FN(op))>::parameter_types>::apply))
0390: 
0391:   #define RANDOM_INPLACE_BATCH_RULE2(op, overload) \
0392:     m.impl(#op"."#overload, SINGLE_ARG(\
0393:       RandomInplaceBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0394:                             c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0395: 
0396:   #define RANDINT_BATCH_RULE(op) \
0397:     m.impl(#op, SINGLE_ARG(\
0398:       RandIntBatchRuleHelper<decltype(&ATEN_FN(op)), &ATEN_FN(op), \
0399:                              c10::guts::function_traits<decltype(ATEN_FN(op))>::parameter_types>::apply))
0400: 
0401:   #define RANDINT_BATCH_RULE2(op, overload) \
0402:     m.impl(#op"."#overload, SINGLE_ARG(\
0403:       RandIntBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0404:                             c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0405: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 406-429 / 第 406-429 行

```cpp
0406:   #define RAND_TWO_LEADING_SCALARS_BATCH_RULE(op, overload) \
0407:     m.impl(#op"."#overload, SINGLE_ARG(\
0408:       RandTwoLeadingScalarsBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0409:                                 c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0410:   #define RANDPERM_BATCH_RULE(op) \
0411:     m.impl(#op, SINGLE_ARG(\
0412:       RandpermBatchRuleHelper<decltype(&ATEN_FN(op)), &ATEN_FN(op), \
0413:                             c10::guts::function_traits<decltype(ATEN_FN(op))>::parameter_types>::apply))
0414: 
0415:   #define RANDPERM_BATCH_RULE2(op, overload) \
0416:     m.impl(#op"."#overload, SINGLE_ARG(\
0417:       RandpermBatchRuleHelper<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0418:                             c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0419: 
0420:   #define UNARY_POINTWISE_RANDOM(op) \
0421:     m.impl(#op, SINGLE_ARG(\
0422:       UnaryPointwiseRandomBatchRule<decltype(&ATEN_FN(op)), &ATEN_FN(op), \
0423:                                     c10::guts::function_traits<decltype(ATEN_FN(op))>::parameter_types>::apply))
0424: 
0425:   #define UNARY_POINTWISE_RANDOM2(op, overload) \
0426:     m.impl(#op"."#overload, SINGLE_ARG(\
0427:       UnaryPointwiseRandomBatchRule<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0428:                                     c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0429: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 430-455 / 第 430-455 行

```cpp
0430:   #define UNARY_POINTWISE_RANDOM_LEADING_FLOAT(op, overload) \
0431:     m.impl(#op"."#overload, SINGLE_ARG(\
0432:       UnaryPointwiseRandomLeadingFloatBatchRule<decltype(&ATEN_FN2(op, overload)), &ATEN_FN2(op, overload), \
0433:                                                 c10::guts::function_traits<decltype(ATEN_FN2(op, overload))>::parameter_types>::apply))
0434: 
0435:   RANDOM_BATCH_RULE(randn);
0436:   RANDOM_BATCH_RULE2(randn, generator);
0437:   RANDOM_BATCH_RULE2(randn, generator_with_names);
0438:   RANDOM_BATCH_RULE2(randn, names);
0439: 
0440:   RANDOM_BATCH_RULE(rand);
0441:   RANDOM_BATCH_RULE2(rand, generator);
0442:   RANDOM_BATCH_RULE2(rand, generator_with_names);
0443:   RANDOM_BATCH_RULE2(rand, names);
0444: 
0445:   RANDOM_INPLACE_BATCH_RULE(random_);
0446:   RANDOM_INPLACE_BATCH_RULE2(random_, from);
0447:   RANDOM_INPLACE_BATCH_RULE2(random_, to);
0448: 
0449:   RANDOM_INPLACE_BATCH_RULE(cauchy_);
0450:   RANDOM_INPLACE_BATCH_RULE(exponential_);
0451:   RANDOM_INPLACE_BATCH_RULE(geometric_);
0452:   RANDOM_INPLACE_BATCH_RULE(log_normal_);
0453:   RANDOM_INPLACE_BATCH_RULE(normal_);
0454:   RANDOM_INPLACE_BATCH_RULE(uniform_);
0455: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 456-480 / 第 456-480 行

```cpp
0456:   RANDINT_BATCH_RULE(randint);
0457:   RANDINT_BATCH_RULE2(randint, generator);
0458:   RAND_TWO_LEADING_SCALARS_BATCH_RULE(randint, low);
0459:   RAND_TWO_LEADING_SCALARS_BATCH_RULE(randint, low_generator);
0460: 
0461:   m.impl("bernoulli_.Tensor", at::functorch::bernoulli_inplace_Tensor_batching_rule);
0462:   RANDOM_INPLACE_BATCH_RULE2(bernoulli_, float);
0463:   UNARY_POINTWISE_RANDOM2(bernoulli, p);
0464: 
0465:   RANDPERM_BATCH_RULE(randperm);
0466:   RANDPERM_BATCH_RULE2(randperm, generator);
0467: 
0468:   RAND_TWO_LEADING_SCALARS_BATCH_RULE(normal, float_float);
0469:   UNARY_POINTWISE_RANDOM2(normal, Tensor_float);
0470:   UNARY_POINTWISE_RANDOM_LEADING_FLOAT(normal, float_Tensor);
0471: 
0472:   m.impl("native_dropout", native_dropout_batching_rule); // needs special casing because cuda version doesn't call bernoulli
0473:   m.impl("native_dropout_backward", native_dropout_backward_batch_rule);
0474: 
0475:   UNARY_POINTWISE_RANDOM(_standard_gamma);
0476:   UNARY_POINTWISE_RANDOM(_sample_dirichlet);
0477:   m.impl("multinomial", multinomial_batching_rule);
0478:   UNARY_POINTWISE_RANDOM(poisson);
0479:   UNARY_POINTWISE_RANDOM(bernoulli);
0480: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; encodes random-number generation or reproducibility semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；编码随机数生成或可复现性语义。关键符号：无明显局部符号。

### Lines 481-502 / 第 481-502 行

```cpp
0481:   #define TENSOR_LIKE_COMMON_ARG_TYPES std::optional<ScalarType>, std::optional<Layout>, std::optional<Device>, std::optional<bool>, std::optional<MemoryFormat>
0482:   m.impl("randint_like", tensor_like_random_batch_rule<decltype(&ATEN_FN(randint_like)), &ATEN_FN(randint_like), int64_t, TENSOR_LIKE_COMMON_ARG_TYPES>);
0483:   m.impl("randint_like.low_dtype", tensor_like_random_batch_rule<\
0484:     decltype(&ATEN_FN2(randint_like, low_dtype)), &ATEN_FN2(randint_like, low_dtype), int64_t, int64_t, TENSOR_LIKE_COMMON_ARG_TYPES>);
0485:   m.impl("rand_like", tensor_like_random_batch_rule<decltype(&ATEN_FN(rand_like)), &ATEN_FN(rand_like), TENSOR_LIKE_COMMON_ARG_TYPES>);
0486:   m.impl("randn_like", tensor_like_random_batch_rule<decltype(&ATEN_FN(randn_like)), &ATEN_FN(randn_like), TENSOR_LIKE_COMMON_ARG_TYPES>);
0487: 
0488:   #undef RANDOM_BATCH_RULE
0489:   #undef RANDOM_BATCH_RULE2
0490:   #undef RANDOM_INPLACE_BATCH_RULE
0491:   #undef RANDOM_INPLACE_BATCH_RULE2
0492:   #undef RANDINT_BATCH_RULE
0493:   #undef RANDINT_BATCH_RULE2
0494:   #undef RAND_TWO_LEADING_SCALARS_BATCH_RULE
0495:   #undef RANDPERM_BATCH_RULE
0496:   #undef RANDPERM_BATCH_RULE2
0497:   #undef UNARY_POINTWISE_RANDOM
0498:   #undef UNARY_POINTWISE_RANDOM2
0499:   #undef UNARY_POINTWISE_RANDOM_LEADING_FLOAT
0500:   #undef TENSOR_LIKE_COMMON_ARG_TYPES
0501: }
0502: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; encodes random-number generation or reproducibility semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；编码随机数生成或可复现性语义。关键符号：无明显局部符号。

### Lines 503-504 / 第 503-504 行

```cpp
0503: } // namespace at::functorch
0504: // NOLINTEND(bugprone-unchecked-optional-access)
```

- **EN:** This block implements local helper logic for `BatchRulesRandomness`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `BatchRulesRandomness` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Device and stream coordination** — 设备与流协调
- **Random-number generation** — 随机数生成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: RandomBatchRuleHelper, RandomInplaceBatchRuleHelper, RandIntBatchRuleHelper, RandTwoLeadingScalarsBatchRuleHelper, RandpermBatchRuleHelper, UnaryPointwiseRandomBatchRule, NormalPointwiseBatchRule, UnaryPointwiseRandomLeadingFloatBatchRule** — 核心符号：RandomBatchRuleHelper、RandomInplaceBatchRuleHelper、RandIntBatchRuleHelper、RandTwoLeadingScalarsBatchRuleHelper、RandpermBatchRuleHelper、UnaryPointwiseRandomBatchRule、NormalPointwiseBatchRule、UnaryPointwiseRandomLeadingFloatBatchRule

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/DynamicLayer.h`, `ATen/functorch/BatchRulesHelper.h`
- **External includes / 外部头文件**: `utility`
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `RandomBatchRuleHelper`, `RandomInplaceBatchRuleHelper`, `RandIntBatchRuleHelper`, `RandTwoLeadingScalarsBatchRuleHelper`, `RandpermBatchRuleHelper`, `UnaryPointwiseRandomBatchRule`, `NormalPointwiseBatchRule`, `UnaryPointwiseRandomLeadingFloatBatchRule`, `random_batching_rule`, `guard`, `shapeVec`, `check_randomness`, `...`
