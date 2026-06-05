# BatchRulesBinaryOps.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesBinaryOps.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesBinaryOps.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Descriptor/handle lifecycle management is important here. Random-number generation or reproducibility semantics are explicitly encoded. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesBinaryOps.cpp` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 该文件显式编码了随机数生成或可复现性语义。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: // Copyright (c) Facebook, Inc. and its affiliates.
0002: // All rights reserved.
0003: //
0004: // This source code is licensed under the BSD-style license found in the
0005: // LICENSE file in the root directory of this source tree.
0006: 
0007: #include <ATen/functorch/BatchRulesHelper.h>
0008: #include <ATen/functorch/PlumbingHelper.h>
0009: #include <ATen/Operators.h>
0010: 
0011: #include <utility>
0012: 
0013: namespace at::functorch {
0014: 
0015: template <typename F, F Func, typename... ExtraArgs>
0016: static Tensor _binary_pointwise_batch_rule(
0017:     const Tensor& tensor, std::optional<int64_t> tensor_batch_dim,
0018:     const Tensor& other, std::optional<int64_t> other_batch_dim,
0019:     ExtraArgs... extra_args) {
0020: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesBinaryOps`. Key symbols: `_binary_pointwise_batch_rule`.
- **CN:** 围绕 `BatchRulesBinaryOps` 构建可复用的模板或辅助层。关键符号：`_binary_pointwise_batch_rule`。

### Lines 21-40 / 第 21-40 行

```cpp
0021:   auto [tensor_, other_]= _binary_pointwise_helper(
0022:       tensor, tensor_batch_dim, other, other_batch_dim);
0023: 
0024:   return Func(tensor_, std::move(other_), std::forward<ExtraArgs>(extra_args)...);
0025: }
0026: 
0027: template <typename A, A a, typename C>
0028: struct BinaryPointwiseBatchRuleHelper;
0029: 
0030: template <typename F, F Func, typename T1, typename T2, typename... T>
0031: struct BinaryPointwiseBatchRuleHelper<F, Func, typelist<T1, T2, T...>> {
0032:   static std::tuple<Tensor, std::optional<int64_t>> apply(
0033:       const Tensor& tensor, std::optional<int64_t> tensor_batch_dim,
0034:       const Tensor& other, std::optional<int64_t> other_batch_dim,
0035:       T... extra_args) {
0036:     return std::tuple(_binary_pointwise_batch_rule<F, Func, T...>(
0037:         tensor, tensor_batch_dim, other, other_batch_dim,
0038:         std::forward<T>(extra_args)...), 0);
0039:   }
0040: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BinaryPointwiseBatchRuleHelper`, `Func`, `apply`, `tuple`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BinaryPointwiseBatchRuleHelper`, `Func`, `apply`, `tuple`。

### Lines 41-60 / 第 41-60 行

```cpp
0041: 
0042: #define BINARY_POINTWISE_BATCH_RULE(fn) SINGLE_ARG(\
0043:     BinaryPointwiseBatchRuleHelper<\
0044:       decltype(&fn),\
0045:       &fn,\
0046:       c10::guts::function_traits<decltype(fn)>::parameter_types>::apply)
0047: 
0048: template <typename A, A a, typename C>
0049: struct BinaryRandomPointwiseBatchRuleHelper;
0050: 
0051: template <typename F, F Func, typename T1, typename T2, typename... T>
0052: struct BinaryRandomPointwiseBatchRuleHelper<F, Func, typelist<T1, T2, T...>> {
0053:   static Tensor apply(const Tensor& tensor, const Tensor& other, T... extra_args) {
0054:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchVmapMode);
0055:     auto maybe_layer = maybeCurrentDynamicLayer();
0056:     TORCH_INTERNAL_ASSERT(maybe_layer.has_value())
0057:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0058:     auto cur_level = maybe_layer->layerId();
0059:     RandomnessType randomness = maybe_layer->randomness();
0060: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `BinaryRandomPointwiseBatchRuleHelper`, `apply`, `guard`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`BinaryRandomPointwiseBatchRuleHelper`, `apply`, `guard`。

### Lines 61-83 / 第 61-83 行

```cpp
0061:     auto [tensor_value, tensor_bdim] = unwrapTensorAtLevel(tensor, cur_level);
0062: 
0063:     auto [other_value, other_bdim] = unwrapTensorAtLevel(other, cur_level);
0064: 
0065:     check_randomness(randomness, (tensor_bdim || other_bdim));
0066:     if (randomness == RandomnessType::Different && !tensor_bdim && !other_bdim) {
0067:       auto shape = tensor_value.sizes();
0068:       VmapSymDimVector shapeVec(1, maybe_layer->batchSize());
0069:       shapeVec.reserve(shape.size() + 1);
0070:       shapeVec.insert(shapeVec.end(), shape.begin(), shape.end());
0071: 
0072:       // not taken care of with binary batch rule, which assumes at least one input is batched
0073:       tensor_value = tensor_value.expand_symint(shapeVec);
0074:       tensor_bdim = 0;
0075:     } else if (randomness == RandomnessType::Same && !tensor_bdim && !other_bdim) {
0076: 
0077:       // avoids unnecessary checks and batch rule assuming output is batched
0078:       return Func(tensor_value, other_value, std::forward<T>(extra_args)...);
0079:     }
0080:     auto res = _binary_pointwise_batch_rule<F, Func, T...>(
0081:       tensor_value, tensor_bdim, other_value, other_bdim,
0082:       std::forward<T>(extra_args)...);
0083:     return makeBatched(std::move(res), 0, cur_level);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; encodes random-number generation or reproducibility semantics; supports transform-aware functorch semantics. Key symbols: `check_randomness`, `shapeVec`, `Func`, `makeBatched`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；编码随机数生成或可复现性语义；支持面向变换的 functorch 语义。关键符号：`check_randomness`, `shapeVec`, `Func`, `makeBatched`。

### Lines 84-106 / 第 84-106 行

```cpp
0084:   }
0085: };
0086: 
0087: #define BINARY_RANDOM_POINTWISE_BATCH_RULE(fn) SINGLE_ARG(\
0088:     BinaryRandomPointwiseBatchRuleHelper<\
0089:       decltype(&fn),\
0090:       &fn,\
0091:       c10::guts::function_traits<decltype(fn)>::parameter_types>::apply)
0092: 
0093: template <typename M, M Meth, typename... ExtraArgs>
0094: static void binary_pointwise_inplace_batch_rule(
0095:     Tensor& tensor, std::optional<int64_t> tensor_batch_dim,
0096:     const Tensor& other, std::optional<int64_t> other_batch_dim,
0097:     ExtraArgs... extra_args) {
0098:   if (!tensor_batch_dim && other_batch_dim) {
0099:     vmapIncompatibleInplaceError("inplace arithmetic");
0100:   }
0101: 
0102:   // compute max logical rank
0103:   auto tensor_logical_rank = rankWithoutBatchDim(tensor, tensor_batch_dim);
0104:   auto other_logical_rank = rankWithoutBatchDim(other, other_batch_dim);
0105:   auto max_logical_rank = std::max(tensor_logical_rank, other_logical_rank);
0106: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesBinaryOps`. Key symbols: `vmapIncompatibleInplaceError`.
- **CN:** 围绕 `BatchRulesBinaryOps` 构建可复用的模板或辅助层。关键符号：`vmapIncompatibleInplaceError`。

### Lines 107-128 / 第 107-128 行

```cpp
0107:   auto tensor_ = moveBatchDimToFront(tensor, tensor_batch_dim);
0108:   auto other_ = moveBatchDimToFront(other, other_batch_dim);
0109: 
0110:   // If the dimensions aren't aligned, we need to line them up.
0111:   // Tensor[B, 3] + Tensor[2, 5, 3] -> Tensor[B, 1, 1, 3] + Tensor[2, 5, 3]
0112:   // Note that only tensors that have a batch dim need to be modified.
0113:   // Tensor[B, 2, 3, 5] + Tensor[5] -> no changes needed
0114:   tensor_ = maybePadToLogicalRank(tensor_, tensor_batch_dim, max_logical_rank);
0115:   other_ = maybePadToLogicalRank(other_, other_batch_dim, max_logical_rank);
0116: 
0117:   (tensor_.*Meth)(other_, std::forward<ExtraArgs>(extra_args)...);
0118: }
0119: 
0120: template <typename F, F Func>
0121: static std::tuple<Tensor, std::optional<int64_t>> comparison_pointwise_batch_rule(
0122:     const Tensor& tensor, std::optional<int64_t> tensor_batch_dim,
0123:     const Tensor& other, std::optional<int64_t> other_batch_dim) {
0124:   // compute max logical rank
0125:   auto tensor_logical_rank = rankWithoutBatchDim(tensor, tensor_batch_dim);
0126:   auto other_logical_rank = rankWithoutBatchDim(other, other_batch_dim);
0127:   auto max_logical_rank = std::max(tensor_logical_rank, other_logical_rank);
0128: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesBinaryOps`. Key symbols: `comparison_pointwise_batch_rule`.
- **CN:** 围绕 `BatchRulesBinaryOps` 构建可复用的模板或辅助层。关键符号：`comparison_pointwise_batch_rule`。

### Lines 129-150 / 第 129-150 行

```cpp
0129:   auto tensor_ = moveBatchDimToFront(tensor, tensor_batch_dim);
0130:   auto other_ = moveBatchDimToFront(other, other_batch_dim);
0131: 
0132:   // If the dimensions aren't aligned, we need to line them up.
0133:   // Tensor[B, 3] + Tensor[2, 5, 3] -> Tensor[B, 1, 1, 3] + Tensor[2, 5, 3]
0134:   // Note that only tensors that have a batch dim need to be modified.
0135:   // Tensor[B, 2, 3, 5] + Tensor[5] -> no changes needed
0136:   tensor_ = maybePadToLogicalRank(tensor_, tensor_batch_dim, max_logical_rank);
0137:   other_ = maybePadToLogicalRank(other_, other_batch_dim, max_logical_rank);
0138: 
0139:   auto result = Func(tensor_, other_);
0140:   return std::make_tuple( std::move(result), 0 );
0141: }
0142: 
0143: static std::tuple<Tensor, std::optional<int64_t>> where_self_batch_rule(
0144:     const Tensor& condition, std::optional<int64_t> condition_bdim,
0145:     const Tensor& self, std::optional<int64_t> self_bdim, const Tensor& other, std::optional<int64_t> other_bdim) {
0146:   auto condition_logical_rank = rankWithoutBatchDim(condition, condition_bdim);
0147:   auto tensor_logical_rank = rankWithoutBatchDim(self, self_bdim);
0148:   auto other_logical_rank = rankWithoutBatchDim(other, other_bdim);
0149:   auto max_logical_rank = std::max({tensor_logical_rank, other_logical_rank, condition_logical_rank});
0150: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `make_tuple`, `where_self_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`make_tuple`, `where_self_batch_rule`。

### Lines 151-172 / 第 151-172 行

```cpp
0151:   auto condition_ = moveBatchDimToFront(condition, condition_bdim);
0152:   auto self_ = moveBatchDimToFront(self, self_bdim);
0153:   auto other_ = moveBatchDimToFront(other, other_bdim);
0154: 
0155:   condition_ = maybePadToLogicalRank(condition_, condition_bdim, max_logical_rank);
0156:   self_ = maybePadToLogicalRank(self_, self_bdim, max_logical_rank);
0157:   other_ = maybePadToLogicalRank(other_, other_bdim, max_logical_rank);
0158:   return std::make_tuple(at::where(condition_, self_, other_), 0);
0159: }
0160: 
0161: static std::tuple<Tensor, std::optional<int64_t>> gelu_backward_batch_rule(
0162:     const Tensor& grad_out, std::optional<int64_t> grad_out_bdim, const Tensor& input, std::optional<int64_t> input_bdim,
0163:     std::string_view approximate) {
0164: 
0165:   // repeat the preprocessing from _binary_pointwise_batch_rule
0166:   auto [grad_out_, input_]= _binary_pointwise_helper(grad_out, grad_out_bdim, input, input_bdim);
0167: 
0168:   // gelu_backward doesn't broadcast well so we need to insist all inputs have a bdim
0169:   const auto batch_size = get_bdim_size2(grad_out, grad_out_bdim, input, input_bdim);
0170:   grad_out_ = ensure_has_bdim(grad_out_, grad_out_bdim.has_value(), batch_size);
0171:   input_ = ensure_has_bdim(input_, input_bdim.has_value(), batch_size);
0172: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `gelu_backward_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `gelu_backward_batch_rule`。

### Lines 173-192 / 第 173-192 行

```cpp
0173:   return std::make_tuple(at::gelu_backward(grad_out_, input_, approximate), 0);
0174: }
0175: 
0176: static std::tuple<Tensor, std::optional<int64_t>> masked_select_batch_rule(
0177:     const Tensor& self, std::optional<int64_t> self_bdim,
0178:     const Tensor& mask, std::optional<int64_t> mask_bdim) {
0179:   TORCH_CHECK(!mask_bdim.has_value(),
0180:       "vmap: Attempted to vmap over `mask` in torch.masked_select(self, mask) ",
0181:       "We cannot support this because for each batch this would return a ",
0182:       "differently shaped Tensor. "
0183:       "Please voice your support in https://github.com/pytorch/functorch/issues/256");
0184:   auto self_ = moveBatchDimToFront(self, self_bdim);
0185:   const auto batch_size = self_.size(0);
0186:   const auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0187:   const auto max_logical_rank = std::max(self_logical_rank, mask.dim());
0188:   self_ = maybePadToLogicalRank(self_, 0, max_logical_rank);
0189: 
0190:   // masked_select returns a 1D tensor, so we have to reshape it into 2D
0191:   auto result = at::masked_select(self_, mask).view({ batch_size, -1 });
0192:   return std::make_tuple(std::move(result), 0);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `masked_select_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `masked_select_batch_rule`。

### Lines 193-215 / 第 193-215 行

```cpp
0193: }
0194: 
0195: static std::tuple<Tensor, std::optional<int64_t>> masked_select_backward_batch_rule(
0196:     const Tensor& grad, std::optional<int64_t> grad_bdim,
0197:     const Tensor& self, std::optional<int64_t> self_bdim,
0198:     const Tensor& mask, std::optional<int64_t> mask_bdim) {
0199:   TORCH_CHECK(!mask_bdim.has_value(),
0200:       "vmap: Attempted to vmap over `mask` in torch.masked_select_backward(grad, self, mask) ",
0201:       "We cannot support this because for each batch this would return a ",
0202:       "differently shaped Tensor. "
0203:       "Please voice your support in https://github.com/pytorch/functorch/issues/256");
0204:   auto self_ = moveBatchDimToFront(self, self_bdim);
0205:   auto grad_ = moveBatchDimToFront(grad, grad_bdim);
0206: 
0207:   const auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0208:   const auto max_logical_rank = std::max(self_logical_rank, mask.dim());
0209: 
0210:   self_ = maybePadToLogicalRank(self_, self_bdim, max_logical_rank);
0211: 
0212:   const auto batch_size = get_bdim_size2(grad, grad_bdim, self, self_bdim);
0213:   self_ = ensure_has_bdim(self_, self_bdim.has_value(), batch_size);
0214:   grad_ = ensure_has_bdim(grad_, grad_bdim.has_value(), batch_size);
0215: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `masked_select_backward_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`masked_select_backward_batch_rule`。

### Lines 216-236 / 第 216-236 行

```cpp
0216:   auto result = at::masked_select_backward(grad_, self_.contiguous(), mask);
0217:   return std::make_tuple(std::move(result), 0);
0218: }
0219: 
0220: static std::tuple<Tensor, std::optional<int64_t>> cdist_backward_batch_rule(
0221:     const Tensor& grad, std::optional<int64_t> grad_bdim,
0222:     Tensor x1, std::optional<int64_t> x1_bdim,
0223:     Tensor x2, std::optional<int64_t> x2_bdim,
0224:     const double p,
0225:     const Tensor& cdist, std::optional<int64_t> cdist_bdim) {
0226: 
0227:   if (cdist_bdim && !x1_bdim) {
0228:     // We need to make sure that x1 has batch dim if cdist has one
0229:     // otherwise, we get
0230:     // RuntimeError: Function CdistBackward0 returned an invalid gradient at index 1 - got [5]
0231:     // but expected shape compatible with [4, 5]
0232:     auto bs = cdist.size(*cdist_bdim);
0233:     x1 = ensure_has_bdim(x1, false, bs).contiguous();
0234:     x1_bdim = 0;
0235:   }
0236: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `cdist_backward_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `cdist_backward_batch_rule`。

### Lines 237-257 / 第 237-257 行

```cpp
0237:   // We need to apply the same preprocessing on x1 and x2 as in the forward pass
0238:   // _binary_pointwise_batch_rule
0239:   std::tie(x1, x2)= _binary_pointwise_helper(x1, x1_bdim, x2, x2_bdim);
0240: 
0241:   auto grad_ = moveBatchDimToFront(grad, grad_bdim);
0242:   if ((x1_bdim || x2_bdim) && !grad_bdim) {
0243:     // We need to make sure that grad has batch dim if x1 or x2 have one
0244:     // Probably, there is an assumption on the strides.
0245:     // Otherwise grad input contains thrash values, e.g. -7.0816e+29, 7.0816e+29
0246:     auto bs = get_bdim_size2(x1, 0, x2, 0);
0247:     grad_ = ensure_has_bdim(grad_, grad_bdim.has_value(), bs);
0248:     grad_ = grad_.contiguous();
0249:   }
0250: 
0251:   auto out = at::_cdist_backward(grad_, x1, x2, p, cdist);
0252: 
0253:   std::optional<int64_t> out_bdim = std::nullopt;
0254:   if (x1_bdim || x2_bdim) {
0255:     out_bdim = 0;
0256:   }
0257: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `tie`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`tie`。

### Lines 258-279 / 第 258-279 行

```cpp
0258:   return std::make_tuple(std::move(out), out_bdim);
0259: }
0260: 
0261: static void fill__Tensor_batch_rule(
0262:     Tensor& self,
0263:     std::optional<int64_t> self_bdim,
0264:     const Tensor& other,
0265:     std::optional<int64_t> other_bdim) {
0266:   if (!other_bdim.has_value()) {
0267:     // Optimization: fill_ is faster than the other path which does
0268:     // reshaping + copy_
0269:     self.fill_(other);
0270:     return;
0271:   }
0272:   if (!self_bdim) {
0273:     vmapIncompatibleInplaceError("fill_");
0274:   }
0275:   auto self_and_other = _binary_pointwise_helper(
0276:       self, self_bdim, other, other_bdim, /*do_type_promotion*/false);
0277:   std::get<0>(self_and_other).copy_(std::get<1>(self_and_other));
0278: }
0279: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `fill__Tensor_batch_rule`, `vmapIncompatibleInplaceError`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `fill__Tensor_batch_rule`, `vmapIncompatibleInplaceError`。

### Lines 280-299 / 第 280-299 行

```cpp
0280: static
0281: std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>>
0282: rrelu_with_noise_batch_rule(
0283:     const Tensor& self,
0284:     std::optional<int64_t> self_bdim,
0285:     Tensor& noise,
0286:     std::optional<int64_t> noise_bdim,
0287:     const at::Scalar& lower,
0288:     const at::Scalar& upper,
0289:     bool training,
0290:     std::optional<at::Generator> generator) {
0291: 
0292:   auto self_ = moveBatchDimToFront(self, self_bdim);
0293:   auto noise_ = moveBatchDimToFront(self, noise_bdim);
0294: 
0295:   auto ret = at::rrelu_with_noise(self_, noise_, lower, upper, training, std::move(generator));
0296: 
0297:   return std::make_tuple(std::move(ret), 0, std::move(noise_), 0);
0298: }
0299: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `rrelu_with_noise_batch_rule`, `make_tuple`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`rrelu_with_noise_batch_rule`, `make_tuple`。

### Lines 300-319 / 第 300-319 行

```cpp
0300: static Tensor rrelu_with_noise_batch(
0301:     const Tensor& self,
0302:     Tensor& noise,
0303:     const Scalar& lower,
0304:     const Scalar& upper,
0305:     bool training,
0306:     std::optional<Generator> generator) {
0307:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0308:   auto maybe_layer = maybeCurrentDynamicLayer();
0309:   vmap_check_escaped(maybe_layer, "gen_vmap_plumbing");
0310:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0311:   int64_t cur_level = maybe_layer->layerId();
0312:   auto [self_value, self_bdim] = unwrapTensorAtLevel(self, cur_level);
0313:   auto [noise_value, noise_bdim] = unwrapTensorAtLevel(noise, cur_level);
0314:   TORCH_CHECK(!noise_bdim.has_value(), "vmap: Attempted to vmap over 'noise' in torch.rrelu_with_noise. This is not supported.");
0315:   auto res = rrelu_with_noise_batch_rule(self_value, self_bdim, noise_value, noise_bdim, lower, upper, training, std::move(generator));
0316:   return makeBatched(std::move(std::get<0>(res)), std::get<1>(res), cur_level);
0317: }
0318: 
0319: static std::tuple<Tensor, std::optional<int64_t>> log_sigmoid_backward_batch_rule(
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `rrelu_with_noise_batch`, `guard`, `vmap_check_escaped`, `makeBatched`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`rrelu_with_noise_batch`, `guard`, `vmap_check_escaped`, `makeBatched`。

### Lines 320-339 / 第 320-339 行

```cpp
0320:   Tensor& grad, std::optional<int64_t> grad_bdim,
0321:   Tensor& self, std::optional<int64_t> self_bdim,
0322:   Tensor& buffer, std::optional<int64_t> buffer_bdim) {
0323:   // NB: This emulates handle_pointwise_ops except we ignore the last argument, buffer
0324:   // when any of the inputs are on cuda/xpu.
0325:   // We do this because on cuda/xpu, buffer is a dummy tensor always of logical rank 1 and
0326:   // it becomes an issue when the rest of the inputs are scalar
0327:   int64_t out_logical_rank = std::max(rankWithoutBatchDim(grad, grad_bdim), rankWithoutBatchDim(self, self_bdim));
0328:   bool inputs_on_cuda = grad.is_cuda() || self.is_cuda() || buffer.is_cuda();
0329:   bool inputs_on_xpu = grad.is_xpu() || self.is_xpu() || buffer.is_xpu();
0330:   if (!inputs_on_cuda && !inputs_on_xpu) {
0331:     out_logical_rank = std::max(out_logical_rank, rankWithoutBatchDim(buffer, buffer_bdim));
0332:   }
0333:   Tensor out_grad = maybePadToLogicalRank(moveBatchDimToFront(grad, grad_bdim), grad_bdim, out_logical_rank);
0334:   Tensor out_self = maybePadToLogicalRank(moveBatchDimToFront(self, self_bdim), self_bdim, out_logical_rank);
0335:   Tensor out_buffer = maybePadToLogicalRank(moveBatchDimToFront(buffer, buffer_bdim), buffer_bdim, out_logical_rank);
0336:   return std::make_tuple(at::log_sigmoid_backward(out_grad, out_self, out_buffer), 0);
0337: }
0338: 
0339: static Tensor binomial_wrapper(const Tensor& count, const Tensor& prob, std::optional<Generator> gen) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `binomial_wrapper`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `binomial_wrapper`。

### Lines 340-364 / 第 340-364 行

```cpp
0340:   return at::binomial(count, prob.contiguous(), std::move(gen)); // Bug in PyTorch, prob shouldn't need to be contiguous
0341: }
0342: 
0343: TORCH_LIBRARY_IMPL(aten, FuncTorchVmapMode, m) {
0344:   #define BINARY_RANDOM_POINTWISE(op) \
0345:     m.impl(#op, BINARY_RANDOM_POINTWISE_BATCH_RULE(ATEN_FN(op)));
0346:   #define BINARY_RANDOM_POINTWISE2(op, overload) \
0347:     m.impl(#op"."#overload, BINARY_RANDOM_POINTWISE_BATCH_RULE(ATEN_FN2(op, overload)));
0348: 
0349:   BINARY_RANDOM_POINTWISE2(normal, Tensor_Tensor);
0350:   m.impl("binomial", BINARY_RANDOM_POINTWISE_BATCH_RULE(at::functorch::binomial_wrapper));
0351: }
0352: 
0353: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0354: #define BINARY_POINTWISE2(op, overload) \
0355:   VMAP_SUPPORT2(op, overload, BINARY_POINTWISE_BATCH_RULE(ATEN_FN2(op, overload)));
0356: #define BINARY_POINTWISE(op) \
0357:   VMAP_SUPPORT(op, BINARY_POINTWISE_BATCH_RULE(ATEN_FN(op)));
0358: #define UNARY_POINTWISE2(op, overload) \
0359:   VMAP_SUPPORT2(op, overload, BASIC_UNARY_BATCH_RULE(ATEN_FN2(op, overload)));
0360: #define UNARY_POINTWISE(op) \
0361:   VMAP_SUPPORT(op, BASIC_UNARY_BATCH_RULE(ATEN_FN(op)));
0362: #define UNARY_SCALAR_POINTWISE2(op, overload) \
0363:   VMAP_SUPPORT(op, overload, SCALAR_UNARY_BATCH_RULE(ATEN_FN2(op, overload)));
0364: 
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesBinaryOps` behavior. Symbols: `binomial`.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesBinaryOps` 的行为。符号：`binomial`。

### Lines 365-387 / 第 365-387 行

```cpp
0365: #define BINARY_SCALAR_2(op, tensor_tensor, tensor_scalar) \
0366:   BINARY_POINTWISE2(op, tensor_tensor);\
0367:   UNARY_POINTWISE2(op, tensor_scalar);
0368: 
0369: // For all 3 combinations of Tensor x Tensor, Tensor x Scalar, Scalar x Tensor
0370: #define BINARY_SCALAR_3(op, tensor_tensor, tensor_scalar, scalar_tensor) \
0371:   BINARY_POINTWISE2(op, tensor_tensor);\
0372:   UNARY_POINTWISE2(op, tensor_scalar);\
0373:   POINTWISE_BOXED(op.scalar_tensor);
0374: 
0375: #define BINARY_SCALAR_3_Tensor(op, tensor_scalar, scalar_tensor) \
0376:   BINARY_POINTWISE(op);\
0377:   UNARY_POINTWISE2(op, tensor_scalar);\
0378:   POINTWISE_BOXED(op.scalar_tensor);
0379: 
0380:   // Batching rule registrations start
0381:   POINTWISE_BOXED(__ilshift__.Tensor);
0382:   POINTWISE_BOXED(__ilshift__.Scalar);
0383:   POINTWISE_BOXED(__irshift__.Tensor)
0384:   POINTWISE_BOXED(__irshift__.Scalar)
0385:   BINARY_SCALAR_2(__lshift__, Tensor, Scalar);
0386:   BINARY_SCALAR_2(__rshift__, Tensor, Scalar);
0387: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 388-410 / 第 388-410 行

```cpp
0388:   BINARY_SCALAR_2(add, Tensor, Scalar);
0389:   POINTWISE_BOXED(addcdiv);
0390:   POINTWISE_BOXED(addcmul);
0391:   BINARY_POINTWISE(atan2);
0392:   BINARY_SCALAR_2(bitwise_and, Tensor, Scalar);
0393:   BINARY_POINTWISE2(bitwise_and_, Tensor);
0394:   POINTWISE_BOXED(bitwise_and_.Scalar);
0395:   POINTWISE_BOXED(bitwise_and.Scalar_Tensor);
0396:   BINARY_SCALAR_2(bitwise_or, Tensor, Scalar);
0397:   BINARY_POINTWISE2(bitwise_or_, Tensor);
0398:   POINTWISE_BOXED(bitwise_or_.Scalar);
0399:   POINTWISE_BOXED(bitwise_or.Scalar_Tensor);
0400:   BINARY_SCALAR_2(bitwise_xor, Tensor, Scalar);
0401:   BINARY_POINTWISE2(bitwise_xor_, Tensor);
0402:   POINTWISE_BOXED(bitwise_xor_.Scalar);
0403:   POINTWISE_BOXED(bitwise_xor.Scalar_Tensor);
0404:   BINARY_SCALAR_3(bitwise_left_shift, Tensor, Tensor_Scalar, Scalar_Tensor);
0405:   POINTWISE_BOXED(bitwise_left_shift_.Tensor_Scalar);
0406:   POINTWISE_BOXED(bitwise_left_shift_.Tensor);
0407:   BINARY_SCALAR_3(bitwise_right_shift, Tensor, Tensor_Scalar, Scalar_Tensor);
0408:   POINTWISE_BOXED(bitwise_right_shift_.Tensor_Scalar);
0409:   POINTWISE_BOXED(bitwise_right_shift_.Tensor);
0410: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 411-432 / 第 411-432 行

```cpp
0411:   UNARY_POINTWISE(clamp);
0412:   POINTWISE_BOXED(clamp.Tensor);
0413:   BINARY_POINTWISE2(clamp_min, Tensor);
0414:   UNARY_POINTWISE(clamp_min);
0415:   POINTWISE_BOXED(clamp_min_);
0416:   BINARY_POINTWISE2(clamp_max, Tensor);
0417:   UNARY_POINTWISE(clamp_max);
0418:   POINTWISE_BOXED(clamp_max_);
0419:   BINARY_POINTWISE(complex);
0420: 
0421:   VARIADIC_BDIMS_BOXED(_euclidean_dist);
0422:   // Implementation note: _binary_pointwise_helper performs a dtype promotion if args are scalars,
0423:   // but cdist can't work with scalars, at least 2d tensors.
0424:   BINARY_POINTWISE(_cdist_forward);
0425:   VMAP_SUPPORT(_cdist_backward, cdist_backward_batch_rule);
0426: 
0427:   BINARY_SCALAR_2(copysign, Tensor, Scalar);
0428:   POINTWISE_BOXED(copysign_.Tensor);
0429:   POINTWISE_BOXED(copysign_.Scalar);
0430:   BINARY_SCALAR_2(div, Tensor, Scalar);
0431:   BINARY_SCALAR_2(div, Tensor_mode, Scalar_mode);
0432: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 433-454 / 第 433-454 行

```cpp
0433:   BINARY_POINTWISE(floor_divide);
0434:   UNARY_POINTWISE2(floor_divide, Scalar);
0435: 
0436:   BINARY_POINTWISE(fmax);
0437:   BINARY_POINTWISE(fmin);
0438:   BINARY_SCALAR_2(fmod, Tensor, Scalar);
0439:   POINTWISE_BOXED(frexp.Tensor);
0440:   BINARY_POINTWISE(heaviside);
0441:   BINARY_POINTWISE(hypot);
0442:   BINARY_POINTWISE(gcd);
0443:   BINARY_POINTWISE(igamma);
0444:   BINARY_POINTWISE(igammac);
0445:   BINARY_POINTWISE2(ldexp, Tensor);
0446:   BINARY_POINTWISE(logaddexp);
0447:   BINARY_POINTWISE(logaddexp2);
0448:   POINTWISE_BOXED(lerp.Scalar);
0449:   POINTWISE_BOXED(lerp.Tensor);
0450:   BINARY_POINTWISE(lcm);
0451:   POINTWISE_BOXED(log_sigmoid_forward);
0452:   BINARY_POINTWISE(maximum);
0453:   BINARY_POINTWISE(minimum);
0454: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 455-489 / 第 455-489 行

```cpp
0455:   BINARY_SCALAR_2(mul, Tensor, Scalar);
0456:   BINARY_POINTWISE(nextafter);
0457:   BINARY_SCALAR_3(pow, Tensor_Tensor, Tensor_Scalar, Scalar);
0458:   POINTWISE_BOXED2(pow_, Scalar);
0459:   BINARY_POINTWISE(polar);
0460:   POINTWISE_BOXED(polygamma);
0461:   BINARY_SCALAR_2(sub, Tensor, Scalar);
0462:   BINARY_SCALAR_3(remainder, Tensor, Scalar, Scalar_Tensor);
0463:   BINARY_SCALAR_2(rsub, Tensor, Scalar);
0464: 
0465:   BINARY_SCALAR_3_Tensor(special_xlog1py, other_scalar, self_scalar);
0466:   BINARY_SCALAR_3_Tensor(special_zeta, other_scalar, self_scalar);
0467: 
0468:   VMAP_SUPPORT2(where, self, where_self_batch_rule);
0469: 
0470:   BINARY_SCALAR_3(xlogy, Tensor, Scalar_Other, Scalar_Self);
0471: 
0472:   POINTWISE_BOXED(elu_backward);
0473:   BINARY_POINTWISE(hardsigmoid_backward);
0474:   BINARY_POINTWISE(hardtanh_backward);
0475:   BINARY_POINTWISE(hardshrink_backward);
0476:   BINARY_POINTWISE(hardswish_backward);
0477:   BINARY_POINTWISE(_prelu_kernel);
0478:   VARIADIC_BDIMS_BOXED(_prelu_kernel_backward);
0479:   BINARY_POINTWISE(leaky_relu_backward);
0480:   BINARY_POINTWISE(logit_backward);
0481:   VMAP_SUPPORT(log_sigmoid_backward, log_sigmoid_backward_batch_rule);
0482:   VMAP_SUPPORT(gelu_backward, gelu_backward_batch_rule);
0483:   BINARY_POINTWISE(sigmoid_backward);
0484:   POINTWISE_BOXED(softplus_backward);
0485:   BINARY_POINTWISE(softshrink_backward);
0486:   BINARY_POINTWISE(tanh_backward);
0487:   BINARY_POINTWISE(threshold_backward);
0488:   BINARY_POINTWISE(silu_backward);
0489: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `BINARY_SCALAR_3_Tensor`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`BINARY_SCALAR_3_Tensor`。

### Lines 490-513 / 第 490-513 行

```cpp
0490:   using TensorScalarInplaceT = Tensor& (Tensor::*)(const Tensor&, const Scalar&) const;
0491:   using ScalarScalarInplaceT = Tensor& (Tensor::*)(const Scalar&, const Scalar&) const;
0492:   using TensorInplaceT = Tensor& (Tensor::*)(const Tensor&) const;
0493:   using TensorInplaceModeT = Tensor& (Tensor::*)(const Tensor&, std::optional<std::string_view>) const;
0494:   using ScalarInplaceT = Tensor& (Tensor::*)(const Scalar&) const;
0495:   using CopyT = Tensor& (Tensor::*)(const Tensor&, bool) const;
0496: 
0497:   POINTWISE_BOXED(add_.Tensor); // just testing
0498:   POINTWISE_BOXED(atan2_);
0499:   POINTWISE_BOXED(gcd_);
0500:   POINTWISE_BOXED(lcm_);
0501:   VMAP_SUPPORT2(add_, Scalar, SINGLE_ARG(unary_inplace_batch_rule<ScalarScalarInplaceT, &Tensor::add_, const Scalar&, const Scalar&>));
0502:   VMAP_SUPPORT2(sub_, Tensor, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorScalarInplaceT, &Tensor::sub_, const Scalar&>));
0503:   VMAP_SUPPORT2(sub_, Scalar, SINGLE_ARG(unary_inplace_batch_rule<ScalarScalarInplaceT, &Tensor::sub_, const Scalar&, const Scalar&>));
0504:   VMAP_SUPPORT2(mul_, Tensor, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceT, &Tensor::mul_>));
0505:   VMAP_SUPPORT2(mul_, Scalar, SINGLE_ARG(unary_inplace_batch_rule<ScalarInplaceT, &Tensor::mul_, const Scalar&>));
0506:   VMAP_SUPPORT2(div_, Tensor, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceT, &Tensor::div_>));
0507:   VMAP_SUPPORT2(div_, Tensor_mode, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceModeT, &Tensor::div_, std::optional<std::string_view>>));
0508:   VMAP_SUPPORT2(div_, Scalar, SINGLE_ARG(unary_inplace_batch_rule<ScalarInplaceT, &Tensor::div_, const Scalar&>));
0509:   VMAP_SUPPORT2(clamp_min_, Tensor, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceT, &Tensor::clamp_min_>));
0510:   VMAP_SUPPORT2(clamp_max_, Tensor, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceT, &Tensor::clamp_max_>));
0511:   VMAP_SUPPORT2(masked_fill_, Scalar, SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorScalarInplaceT, &Tensor::masked_fill_, const Scalar&>));
0512:   VMAP_SUPPORT(copy_, SINGLE_ARG(binary_pointwise_inplace_batch_rule<CopyT, &Tensor::copy_, bool>));
0513: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `TensorScalarInplaceT`, `ScalarScalarInplaceT`, `TensorInplaceT`, `TensorInplaceModeT`, `ScalarInplaceT`, `CopyT`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`TensorScalarInplaceT`, `ScalarScalarInplaceT`, `TensorInplaceT`, `TensorInplaceModeT`, `ScalarInplaceT`, `CopyT`。

### Lines 514-533 / 第 514-533 行

```cpp
0514: #define COMPARISON_POINTWISE(op) \
0515:   VMAP_SUPPORT2(op, Tensor, \
0516:       SINGLE_ARG(comparison_pointwise_batch_rule<decltype(&ATEN_FN2(op, Tensor)), &at::op>)); \
0517:   UNARY_POINTWISE2(op, Scalar)
0518: 
0519:   COMPARISON_POINTWISE(eq);
0520:   COMPARISON_POINTWISE(gt);
0521:   COMPARISON_POINTWISE(ge);
0522:   COMPARISON_POINTWISE(le);
0523:   COMPARISON_POINTWISE(lt);
0524:   COMPARISON_POINTWISE(ne);
0525: 
0526: #undef COMPARISON_POINTWISE
0527: #undef BINARY_POINTWISE2
0528: #undef BINARY_POINTWISE
0529: #undef UNARY_POINTWISE2
0530: #undef UNARY_POINTWISE
0531: #undef UNARY_SCALAR_POINTWISE2
0532: #undef BINARY_SCALAR_3
0533: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 534-553 / 第 534-553 行

```cpp
0534: #define LOGICAL_COMPARISON_POINTWISE(op) \
0535:   VMAP_SUPPORT(op, \
0536:       SINGLE_ARG(comparison_pointwise_batch_rule<decltype(&ATEN_FN(op)), &ATEN_FN(op)>)); \
0537:   VMAP_SUPPORT(op ## _, \
0538:       SINGLE_ARG(binary_pointwise_inplace_batch_rule<TensorInplaceT, &Tensor:: op ## _ >));
0539: 
0540:   LOGICAL_COMPARISON_POINTWISE(logical_and);
0541:   LOGICAL_COMPARISON_POINTWISE(logical_or);
0542:   LOGICAL_COMPARISON_POINTWISE(logical_xor);
0543: 
0544: #undef SINGLE_ARG
0545: #undef LOGICAL_COMPARISON_POINTWISE
0546:   VMAP_SUPPORT(masked_select, masked_select_batch_rule);
0547:   VMAP_SUPPORT(masked_select_backward, masked_select_backward_batch_rule);
0548: 
0549:   VMAP_SUPPORT2(fill_, Tensor, fill__Tensor_batch_rule);
0550:   m.impl("rrelu_with_noise", rrelu_with_noise_batch);
0551: }
0552: 
0553: } // namespace at::functorch
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Device and stream coordination** — 设备与流协调
- **Random-number generation** — 随机数生成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `ATen/functorch/PlumbingHelper.h`, `ATen/Operators.h`
- **External includes / 外部头文件**: `utility`
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `BinaryPointwiseBatchRuleHelper`, `BinaryRandomPointwiseBatchRuleHelper`, `TensorScalarInplaceT`, `ScalarScalarInplaceT`, `TensorInplaceT`, `TensorInplaceModeT`, `ScalarInplaceT`, `CopyT`, `_binary_pointwise_batch_rule`, `Func`, `apply`, `tuple`, `...`
