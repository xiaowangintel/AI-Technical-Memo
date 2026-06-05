# BatchRulesNorm.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesNorm.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesNorm.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesNorm.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

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
0008: #include <ATen/functorch/PlumbingHelper.h>
0009: 
0010: namespace at::functorch {
0011: 
0012: static bool is_empty_tensor(const Tensor& tensor) {
0013:   const auto shape = tensor.sizes();
0014:   return shape.size() == 1 && shape[0] == 0;
0015: }
0016: 
0017: static std::optional<int64_t> compute_stat_bdim(
0018:     std::optional<int64_t> input_bdim,
0019:     const Tensor& stat) {
0020:   // There's a weird case where mean, rstd can both have shape (0,).
0021:   // It's possible that this is a bug on the PyTorch side.
0022:   // When that happens we don't want to return a BatchedTensor.
0023:   if (input_bdim.has_value() && !is_empty_tensor(stat)) {
0024:     return 0;
0025:   }
0026:   return std::nullopt;
0027: }
0028: 
0029: static Tensor padRight(const Tensor& tensor, std::optional<int64_t> has_bdim, int64_t logical_rank) {
0030:   // NB: Batch dim, if it exists, is assumed to be the first dim
0031:   auto tensor_logical_rank = rankWithoutBatchDim(tensor, has_bdim);
0032:   if (tensor_logical_rank >= logical_rank) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `is_empty_tensor`, `compute_stat_bdim`, `padRight`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`is_empty_tensor`, `compute_stat_bdim`, `padRight`。

### Lines 33-69 / 第 33-69 行

```cpp
0033:     return tensor;
0034:   }
0035:   VmapDimVector new_sizes(tensor.sizes().begin(), tensor.sizes().end());
0036:   for (int64_t i = 0; i < logical_rank - tensor_logical_rank; i++) {
0037:     new_sizes.push_back(1);
0038:   }
0039:   return tensor.view(new_sizes);
0040: }
0041: 
0042: template<typename F, F Func>
0043: static
0044: std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>>
0045: batch_norm_batch_rule(
0046:     const Tensor& input, std::optional<int64_t> input_bdim,
0047:     const std::optional<Tensor>& weight_opt, std::optional<int64_t> weight_bdim,
0048:     const std::optional<Tensor>& bias_opt, std::optional<int64_t> bias_bdim,
0049:     const std::optional<Tensor>& running_mean_opt, std::optional<int64_t> running_mean_bdim,
0050:     const std::optional<Tensor>& running_var_opt, std::optional<int64_t> running_var_bdim,
0051:     bool training, double momentum, double eps) {
0052:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0053:   const Tensor& weight = *weight_maybe_owned;
0054:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0055:   const Tensor& bias = *bias_maybe_owned;
0056:   c10::MaybeOwned<Tensor> running_mean_maybe_owned = at::borrow_from_optional_tensor(running_mean_opt);
0057:   const auto& running_mean = *running_mean_maybe_owned;
0058:   c10::MaybeOwned<Tensor> running_var_maybe_owned = at::borrow_from_optional_tensor(running_var_opt);
0059:   const auto& running_var = *running_var_maybe_owned;
0060:   TORCH_CHECK(!training || (!input_bdim || ((!running_mean.defined() || running_mean_bdim) && (!running_var.defined() || running_var_bdim))),
0061:       "Batch norm got a batched tensor as input while the running_mean or running_var, which will be updated in place, ",
0062:       "were not batched.\nIf you are using a module and do not need eval mode, please set `track_running_stats` to be False.",
0063:       "If you are using a prebuilt module and do not need eval mode, please see the functorch website for resources on ",
0064:       "how to patch your module to work with vmap");
0065:   std::optional<int64_t> bdim_size;
0066:   Tensor result0;
0067:   Tensor mean;
0068:   Tensor rstd;
0069:   if (!input_bdim && !running_mean_bdim && !running_var_bdim) {
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesNorm`. Key symbols: `new_sizes`, `batch_norm_batch_rule`.
- **CN:** 围绕 `BatchRulesNorm` 构建可复用的模板或辅助层。关键符号：`new_sizes`, `batch_norm_batch_rule`。

### Lines 70-105 / 第 70-105 行

```cpp
0070:     const auto dummy_weight = at::ones(input.size(1), input.options());  // cudnn and miopen require a weight
0071:     const auto dummy_bias = at::zeros(input.size(1), input.options());   // without this, get "strides() called on undefined Tensor" on cuda
0072:     auto result = Func(input, dummy_weight, dummy_bias, running_mean_opt, running_var_opt, training, momentum, eps);
0073:     result0 = std::get<0>(result).transpose(0, 1);          // [C, B, *]
0074:     mean = std::move(std::get<1>(result));
0075:     rstd = std::move(std::get<2>(result));
0076:   } else {
0077:     bdim_size = get_bdim_size3(input, input_bdim, running_mean, running_mean_bdim, running_var, running_var_bdim);
0078:     auto input_ = moveBatchDimToFront(input, input_bdim);
0079:     input_ = ensure_has_bdim(input_, input_bdim.has_value(), bdim_size.value());
0080:     input_ = reshape_dim_into(0, /*channels dim*/1, input_);
0081: 
0082:     std::optional<Tensor> running_mean_;
0083:     std::optional<Tensor> running_var_;
0084:     if (running_mean.defined()) {
0085:       running_mean_ = moveBatchDimToFront(running_mean, running_mean_bdim);
0086:       running_mean_ = ensure_has_bdim(*running_mean_, running_mean_bdim.has_value(), bdim_size.value());
0087:       running_mean_ = reshape_dim_into(0, 0, *running_mean_).contiguous();
0088:     }
0089:     if (running_var.defined()) {
0090:       running_var_ = moveBatchDimToFront(running_var, running_var_bdim);
0091:       running_var_ = ensure_has_bdim(*running_var_, running_var_bdim.has_value(), bdim_size.value());
0092:       running_var_ = reshape_dim_into(0, 0, *running_var_).contiguous();
0093:     }
0094: 
0095:     const auto dummy_weight = at::ones(input_.size(1), input_.options());  // cudnn and miopen require a weight
0096:     const auto dummy_bias = at::zeros(input_.size(1), input_.options());   // without this, get "strides() called on undefined Tensor" on cuda
0097:     auto result = Func(input_, dummy_weight, dummy_bias, running_mean_, running_var_, training, momentum, eps);
0098:     result0 = std::get<0>(result).transpose(0, 1);                // [(B0, C), B, *]
0099:     mean = std::move(std::get<1>(result));
0100:     rstd = std::move(std::get<2>(result));
0101:     result0 = reshape_dim_outof(0, bdim_size.value(), result0);   // [B0, C, B, *]
0102:     mean = reshape_dim_outof(0, bdim_size.value(), mean);         // [B0, C]
0103:     rstd = reshape_dim_outof(0, bdim_size.value(), rstd);         // [B0, C]
0104:   }
0105: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：无明显局部符号。

### Lines 106-139 / 第 106-139 行

```cpp
0106:   const auto stats_bdim = compute_stat_bdim(bdim_size, mean);
0107:   if (weight.defined()) {
0108:     const auto input_logical_rank = rankWithoutBatchDim(input, input_bdim);
0109:     auto weight_ = moveBatchDimToFront(weight, weight_bdim);
0110:     weight_ = padRight(weight_, weight_bdim, input_logical_rank);
0111:     result0 = result0 * weight_;
0112:   }
0113:   if (bias.defined()) {
0114:     const auto result_logical_rank = rankWithoutBatchDim(
0115:         result0,
0116:         bdim_size.has_value() || weight_bdim.has_value() ? std::optional<int64_t>(0) : std::optional<int64_t>(std::nullopt));
0117:     auto bias_ = moveBatchDimToFront(bias, bias_bdim);
0118:     bias_ = padRight(bias_, bias_bdim, result_logical_rank);
0119:     result0 = result0 + bias_;
0120:   }
0121:   result0 = result0.transpose(1, 2);  // [B0, B, C, *], because some arg must have been batched, the output must be batched
0122:   return std::make_tuple(std::move(result0), 0, std::move(mean), stats_bdim, std::move(rstd), stats_bdim);
0123: }
0124: 
0125: template<typename F, F Func>
0126: static
0127: std::tuple<at::Tensor, std::optional<int64_t>> batch_norm_backward_no_weight_bias_batch_rule(
0128:     const at::Tensor & grad_out, std::optional<int64_t> grad_out_bdim,
0129:     const at::Tensor & input, std::optional<int64_t> input_bdim,
0130:     const std::optional<at::Tensor> & running_mean_opt, std::optional<int64_t> running_mean_bdim,
0131:     const std::optional<at::Tensor> & running_var_opt, std::optional<int64_t> running_var_bdim,
0132:     const at::Tensor & mean, std::optional<int64_t> mean_bdim,
0133:     const at::Tensor & rstd, std::optional<int64_t> rstd_bdim,
0134:     bool training, double eps) {
0135:   c10::MaybeOwned<Tensor> running_mean_maybe_owned = at::borrow_from_optional_tensor(running_mean_opt);
0136:   const Tensor& running_mean = *running_mean_maybe_owned;
0137:   c10::MaybeOwned<Tensor> running_var_maybe_owned = at::borrow_from_optional_tensor(running_var_opt);
0138:   const Tensor& running_var = *running_var_maybe_owned;
0139: 
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesNorm`. Key symbols: `make_tuple`, `batch_norm_backward_no_weight_bias_batch_rule`.
- **CN:** 围绕 `BatchRulesNorm` 构建可复用的模板或辅助层。关键符号：`make_tuple`, `batch_norm_backward_no_weight_bias_batch_rule`。

### Lines 140-169 / 第 140-169 行

```cpp
0140:   if (!grad_out_bdim.has_value() && !input_bdim.has_value() && !running_mean_bdim.has_value() && !running_var_bdim.has_value()) {
0141:     // for either of these to have bdims, the input, running_mean, or running_var must have had a bdim
0142:     TORCH_INTERNAL_ASSERT(!mean_bdim);
0143:     TORCH_INTERNAL_ASSERT(!rstd_bdim);
0144:     const auto dummy_weight = at::ones(input.size(1), input.options());
0145:     auto result =Func(
0146:         grad_out, input, dummy_weight, running_mean_opt, running_var_opt, mean, rstd, training, eps, {true, false, false});
0147:     return {std::move(std::get<0>(result)), std::nullopt};
0148:   }
0149: 
0150:   auto grad_out_ = moveBatchDimToFront(grad_out, grad_out_bdim);
0151:   auto input_ = moveBatchDimToFront(input, input_bdim);
0152:   auto mean_ = moveBatchDimToFront(mean, mean_bdim);
0153:   auto rstd_ = moveBatchDimToFront(rstd, rstd_bdim);
0154: 
0155:   // ensure all inputs have bdim.
0156:   const auto bdim_size = get_bdim_size4(grad_out, grad_out_bdim, input, input_bdim, running_mean, running_mean_bdim, running_var, running_var_bdim);
0157:   grad_out_ = ensure_has_bdim(grad_out_, grad_out_bdim.has_value(), bdim_size);
0158:   input_ = ensure_has_bdim(input_, input_bdim.has_value(), bdim_size);
0159:   mean_ = ensure_has_bdim(mean_, mean_bdim.has_value(), bdim_size);
0160:   rstd_ = ensure_has_bdim(rstd_, rstd_bdim.has_value(), bdim_size);
0161: 
0162:   std::optional<Tensor> running_mean_;
0163:   std::optional<Tensor> running_var_;
0164:   if (running_mean.defined()) {
0165:     running_mean_ = moveBatchDimToFront(running_mean, running_mean_bdim);
0166:     running_mean_ = ensure_has_bdim(*running_mean_, running_mean_bdim.has_value(), bdim_size);
0167:     running_mean_ = reshape_dim_into(0, 0, *running_mean_).contiguous();
0168:   }
0169:   if (running_var.defined()) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 170-219 / 第 170-219 行

```cpp
0170:     running_var_ = moveBatchDimToFront(running_var, running_var_bdim);
0171:     running_var_ = ensure_has_bdim(*running_var_, running_var_bdim.has_value(), bdim_size);
0172:     running_var_ = reshape_dim_into(0, 0, *running_var_).contiguous();
0173:   }
0174: 
0175:   input_ = reshape_dim_into(0, /*channels dim*/1, input_);
0176:   TORCH_INTERNAL_ASSERT(mean_.dim() == 2);
0177:   TORCH_INTERNAL_ASSERT(rstd_.dim() == 2);
0178:   mean_ = reshape_dim_into(0, 0, mean_);
0179:   rstd_ = reshape_dim_into(0, 0, rstd_);
0180:   grad_out_ = grad_out_.transpose(0, 1).flatten(1, 2); // [B0, B, C, *] -> [B, (B0, C), *]
0181: 
0182:   const auto dummy_weight = at::ones(input_.size(1), input_.options());
0183:   auto result = at::native_batch_norm_backward(
0184:       grad_out_.contiguous(),
0185:       input_.contiguous(),
0186:       dummy_weight,
0187:       running_mean_,  // contiguous called if there is a tensor given
0188:       running_var_,   // contiguous called if there is a tensor given
0189:       mean_.contiguous(),
0190:       rstd_.contiguous(),
0191:       training, eps, {true, false, false});
0192:   auto& result0 = std::get<0>(result);
0193:   result0 = reshape_dim_outof(1, bdim_size, result0); // [B, B0, C, *]
0194:   result0 = result0.transpose(0, 1); // [B0, B, C, *]
0195:   return std::make_tuple(std::move(result0), 0);
0196: }
0197: 
0198: template<typename F, F Func>
0199: static
0200: std::tuple<at::Tensor,at::Tensor,at::Tensor> batch_norm_backward_plumbing(
0201:     const at::Tensor & grad_out,
0202:     const at::Tensor & input,
0203:     const std::optional<at::Tensor> & weight_opt,
0204:     const std::optional<at::Tensor> & running_mean_opt,
0205:     const std::optional<at::Tensor> & running_var_opt,
0206:     const std::optional<at::Tensor> & save_mean_opt,
0207:     const std::optional<at::Tensor> & save_rstd_opt,
0208:     bool training,
0209:     double eps,
0210:     std::array<bool,3> output_mask) {
0211:   // See [Note: hacky wrapper removal for optional tensor]
0212:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0213:   const Tensor& weight = *weight_maybe_owned;
0214:   c10::MaybeOwned<Tensor> running_mean_maybe_owned = at::borrow_from_optional_tensor(running_mean_opt);
0215:   const Tensor& running_mean = *running_mean_maybe_owned;
0216:   c10::MaybeOwned<Tensor> running_var_maybe_owned = at::borrow_from_optional_tensor(running_var_opt);
0217:   const Tensor& running_var = *running_var_maybe_owned;
0218:   // NB: not sure why these are optional...these are required from the forward
0219:   TORCH_INTERNAL_ASSERT(save_mean_opt.has_value());
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesNorm`. Key symbols: `make_tuple`, `batch_norm_backward_plumbing`.
- **CN:** 围绕 `BatchRulesNorm` 构建可复用的模板或辅助层。关键符号：`make_tuple`, `batch_norm_backward_plumbing`。

### Lines 220-252 / 第 220-252 行

```cpp
0220:   TORCH_INTERNAL_ASSERT(save_rstd_opt.has_value());
0221:   const Tensor& save_mean = *save_mean_opt;
0222:   const Tensor& save_rstd = *save_rstd_opt;
0223:   TORCH_INTERNAL_ASSERT(save_mean.defined());
0224:   TORCH_INTERNAL_ASSERT(save_rstd.defined());
0225: 
0226:   // plumbing
0227:   auto maybe_layer = maybeCurrentDynamicLayer();
0228:   vmap_check_escaped(maybe_layer, "batch_norm_backward_plumbing");
0229:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0230:   int64_t cur_level = maybe_layer->layerId();
0231: 
0232:   auto [grad_out_value, grad_out_bdim] = unwrapTensorAtLevel(grad_out, cur_level);
0233:   auto [input_value, input_bdim] = unwrapTensorAtLevel(input, cur_level);
0234:   Tensor mean_value;
0235:   std::optional<Tensor> weight_value;
0236:   std::optional<int64_t> weight_bdim;
0237:   if (weight.defined()) {
0238:     std::tie(weight_value, weight_bdim) = unwrapTensorAtLevel(weight, cur_level);
0239:   }
0240:   std::optional<Tensor> running_mean_value;
0241:   std::optional<int64_t> running_mean_bdim;
0242:   if (running_mean.defined()) {
0243:     std::tie(running_mean_value, running_mean_bdim) = unwrapTensorAtLevel(running_mean, cur_level);
0244:   }
0245:   std::optional<Tensor> running_var_value;
0246:   std::optional<int64_t> running_var_bdim;
0247:   if (running_var.defined()) {
0248:     std::tie(running_var_value, running_var_bdim) = unwrapTensorAtLevel(running_var, cur_level);
0249:   }
0250:   auto [save_mean_value, save_mean_bdim] = unwrapTensorAtLevel(save_mean, cur_level);
0251:   auto [save_rstd_value, save_rstd_bdim] = unwrapTensorAtLevel(save_rstd, cur_level);
0252: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `vmap_check_escaped`, `tie`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`vmap_check_escaped`, `tie`。

### Lines 253-287 / 第 253-287 行

```cpp
0253:   // results
0254:   Tensor grad_bias;
0255:   Tensor grad_weight;
0256:   Tensor grad_input;
0257: 
0258:   TORCH_INTERNAL_ASSERT(grad_out_value.dim() > 1);  // batch_norm can't operate on 1D tensors so the output will be at least 2D
0259:   if (output_mask[2]) {
0260:     grad_bias = grad_out.transpose(0, 1).sum(range(1, grad_out.dim()));
0261:   }
0262:   if (output_mask[1] && weight_value.has_value()) {
0263:     // NB: output isn't saved...
0264:     auto mean = training ? save_mean : running_mean;
0265:     auto var = training ? save_rstd : (1 / at::sqrt(running_var + eps));
0266:     const auto normalized_input = (input.transpose(0, 1) - padRight(mean, std::nullopt, input.dim())) * padRight(var, std::nullopt, input.dim());
0267:     const auto expanded_grad_weight = normalized_input * grad_out.transpose(0, 1);
0268:     grad_weight = expanded_grad_weight.sum(range(1, grad_out.dim()));
0269:   }
0270:   if (output_mask[0]) {
0271:     const auto grad_normalized_input = weight.defined() ?
0272:       grad_out.transpose(0, 1) * padRight(weight, std::nullopt, grad_out.dim()) : grad_out.transpose(0, 1);           // [B0, C, B, *]
0273:     auto [grad_normalized_input_value, grad_normalized_input_bdim] =
0274:         unwrapTensorAtLevel(grad_normalized_input.transpose(0, 1), cur_level);       // [B0, B, C, *]
0275: 
0276:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0277:     auto results = batch_norm_backward_no_weight_bias_batch_rule<F, Func>(
0278:         grad_normalized_input_value, grad_normalized_input_bdim,
0279:         input_value, input_bdim,
0280:         running_mean_value, running_mean_bdim,
0281:         running_var_value, running_var_bdim,
0282:         save_mean_value, save_mean_bdim,
0283:         save_rstd_value, save_rstd_bdim,
0284:         training, eps);
0285:     grad_input = makeBatched(std::move(std::get<0>(results)), std::get<1>(results), cur_level);
0286:   }
0287:   return std::make_tuple(std::move(grad_input), std::move(grad_weight), std::move(grad_bias));
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `unwrapTensorAtLevel`, `guard`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`unwrapTensorAtLevel`, `guard`, `make_tuple`。

### Lines 288-318 / 第 288-318 行

```cpp
0288: }
0289: 
0290: static std::tuple<Tensor,Tensor,Tensor> native_group_norm_plumbing(
0291:     const Tensor & input, const std::optional<Tensor> & weight_opt,
0292:     const std::optional<Tensor> & bias_opt, int64_t N, int64_t C,
0293:     int64_t HxW, int64_t group, double eps) {
0294:   // See [Note: hacky wrapper removal for optional tensor]
0295:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0296:   const Tensor& weight = *weight_maybe_owned;
0297:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0298:   const Tensor& bias = *bias_maybe_owned;
0299: 
0300:   auto maybe_layer = maybeCurrentDynamicLayer();
0301:   vmap_check_escaped(maybe_layer, "native_group_norm_plumbing");
0302:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0303:   int64_t cur_level = maybe_layer->layerId();
0304: 
0305:   if (!areAnyBatchedAtLevel({input, weight_opt, bias_opt}, cur_level)) {
0306:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0307:     return at::native_group_norm(input, weight_opt, bias_opt, N, C, HxW, group, eps);
0308:   }
0309: 
0310:   auto [input_value, input_bdim] = unwrapTensorAtLevel(input, cur_level);
0311: 
0312:   Tensor result0;
0313:   Tensor mean;
0314:   Tensor rstd;
0315:   if (input_bdim) {
0316:     const auto input_ = reshape_dim_into(*input_bdim, 0, input_value);
0317:     const auto bdim_size = input_value.size(*input_bdim);
0318: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `native_group_norm_plumbing`, `vmap_check_escaped`, `guard`, `native_group_norm`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`native_group_norm_plumbing`, `vmap_check_escaped`, `guard`, `native_group_norm`。

### Lines 319-352 / 第 319-352 行

```cpp
0319:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0320:     std::tie(result0, mean, rstd) = at::native_group_norm(input_, std::nullopt, std::nullopt, N * bdim_size, C, HxW, group, eps);
0321:     result0 = makeBatched(reshape_dim_outof(0, bdim_size, result0), 0, cur_level);
0322:     mean = makeBatched(reshape_dim_outof(0, bdim_size, mean), 0, cur_level);
0323:     rstd = makeBatched(reshape_dim_outof(0, bdim_size, rstd), 0, cur_level);
0324:   } else {
0325:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0326:     std::tie(result0, mean, rstd) = at::native_group_norm(input_value, std::nullopt, std::nullopt, N, C, HxW, group, eps);
0327:   }
0328: 
0329:   if (weight.defined()) {
0330:     const auto padded_weight = padRight(weight, std::nullopt, result0.dim() - 1);
0331:     result0 = result0 * padded_weight;
0332:   }
0333: 
0334:   if (bias.defined()) {
0335:     const auto padded_bias = padRight(bias, std::nullopt, result0.dim() - 1);
0336:     result0 = result0 + padded_bias;
0337:   }
0338: 
0339:   return std::make_tuple(std::move(result0), std::move(mean), std::move(rstd));
0340: }
0341: 
0342: static at::Tensor group_norm_backward_no_weight_bias_batch_rule(
0343:     const at::Tensor & grad_out, std::optional<int64_t> grad_out_bdim,
0344:     const at::Tensor & input, std::optional<int64_t> input_bdim,
0345:     const at::Tensor & mean, std::optional<int64_t> mean_bdim,
0346:     const at::Tensor & rstd, std::optional<int64_t> rstd_bdim,
0347:     int64_t N, int64_t C, int64_t HxW, int64_t group) {
0348:   auto grad_out_ = moveBatchDimToFront(grad_out, grad_out_bdim);
0349:   auto input_ = moveBatchDimToFront(input, input_bdim);
0350:   auto mean_ = moveBatchDimToFront(mean, mean_bdim);
0351:   auto rstd_ = moveBatchDimToFront(rstd, rstd_bdim);
0352: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `guard`, `tie`, `make_tuple`, `group_norm_backward_no_weight_bias_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`guard`, `tie`, `make_tuple`, `group_norm_backward_no_weight_bias_batch_rule`。

### Lines 353-387 / 第 353-387 行

```cpp
0353:   const auto bdim_size = get_bdim_size2(grad_out, grad_out_bdim, input, input_bdim);
0354:   grad_out_ = ensure_has_bdim(grad_out_, grad_out_bdim.has_value(), bdim_size);
0355:   input_ = ensure_has_bdim(input_, input_bdim.has_value(), bdim_size);
0356:   mean_ = ensure_has_bdim(mean_, mean_bdim.has_value(), bdim_size);
0357:   rstd_ = ensure_has_bdim(rstd_, rstd_bdim.has_value(), bdim_size);
0358: 
0359:   grad_out_ = reshape_dim_into(0, 0, grad_out_); // [B0 * N, C, *]
0360:   input_ = reshape_dim_into(0, 0, input_);       // [B0 * N, C, *]
0361:   mean_ = reshape_dim_into(0, 0, mean_);         // [B0 * N, G]
0362:   rstd_ = reshape_dim_into(0, 0, rstd_);         // [B0 * N, G]
0363: 
0364:   auto result0 = std::get<0>(native_group_norm_backward(
0365:       grad_out_.contiguous(),
0366:       input_.contiguous(),
0367:       mean_.contiguous(),
0368:       rstd_.contiguous(),
0369:       std::nullopt, N * bdim_size, C, HxW, group, {true, false, false}));
0370:   return reshape_dim_outof(0, bdim_size, result0);
0371: }
0372: 
0373: static std::tuple<Tensor,Tensor,Tensor> native_group_norm_backward_plumbing(
0374:   const Tensor & grad_out, const Tensor & input, const Tensor & mean,
0375:   const Tensor & rstd, const std::optional<Tensor> & weight_opt,
0376:   int64_t N, int64_t C, int64_t HxW, int64_t group, std::array<bool,3> output_mask
0377: ) {
0378:   // See [Note: hacky wrapper removal for optional tensor]
0379:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0380:   const Tensor& weight = *weight_maybe_owned;
0381: 
0382:   // plumbing
0383:   auto maybe_layer = maybeCurrentDynamicLayer();
0384:   vmap_check_escaped(maybe_layer, "native_group_norm_backward_plumbing");
0385:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0386:   int64_t cur_level = maybe_layer->layerId();
0387: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `reshape_dim_outof`, `native_group_norm_backward_plumbing`, `vmap_check_escaped`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`reshape_dim_outof`, `native_group_norm_backward_plumbing`, `vmap_check_escaped`。

### Lines 388-418 / 第 388-418 行

```cpp
0388:   if (!areAnyBatchedAtLevel({grad_out, input, mean, rstd, weight_opt}, cur_level)) {
0389:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0390:     return at::native_group_norm_backward(grad_out, input, mean, rstd, weight_opt, N, C, HxW, group, output_mask);
0391:   }
0392: 
0393:   auto [input_value, input_bdim] = unwrapTensorAtLevel(input, cur_level);
0394:   Tensor weight_value;
0395:   std::optional<int64_t> weight_bdim;
0396:   if (weight.defined()){
0397:     std::tie(weight_value, weight_bdim) = unwrapTensorAtLevel(weight, cur_level);
0398:   }
0399:   auto [mean_value, mean_bdim] = unwrapTensorAtLevel(mean, cur_level);
0400:   auto [rstd_value, rstd_bdim] = unwrapTensorAtLevel(rstd, cur_level);
0401: 
0402:   // results
0403:   Tensor grad_input;
0404:   Tensor grad_weight;
0405:   Tensor grad_bias;
0406: 
0407:   TORCH_INTERNAL_ASSERT(grad_out.dim() > 1);  // group_norm can't operate on 1D tensors so the output will be at least 2D
0408:   if (output_mask[2]) {
0409:     grad_bias = grad_out.transpose(0, 1).sum(range(1, grad_out.dim()));
0410:   }
0411: 
0412:   if (output_mask[1] && weight.defined()) {
0413:     const auto reshaped_input = reshape_dim_outof(1, group, input);
0414:     const auto normalized_input = (reshaped_input - padRight(mean, std::nullopt, reshaped_input.dim())) * padRight(rstd, std::nullopt, reshaped_input.dim());
0415:     const auto expanded_grad_weight = reshape_dim_into(1, 1, normalized_input) * grad_out;
0416:     grad_weight = expanded_grad_weight.transpose(0, 1).sum(range(1, expanded_grad_weight.dim()));
0417:   }
0418: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `guard`, `native_group_norm_backward`, `tie`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`guard`, `native_group_norm_backward`, `tie`。

### Lines 419-448 / 第 419-448 行

```cpp
0419:   if (output_mask[0]) {
0420:     const auto grad_normalized_input = weight.defined() ?
0421:       grad_out * padRight(weight, std::nullopt, grad_out.dim() - 1) : grad_out;
0422:     auto [grad_normalized_input_value, grad_normalized_input_bdim] =
0423:         unwrapTensorAtLevel(grad_normalized_input, cur_level);
0424: 
0425:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0426:     auto tensor = group_norm_backward_no_weight_bias_batch_rule(
0427:         grad_normalized_input_value, grad_normalized_input_bdim,
0428:         input_value, input_bdim,
0429:         mean_value, mean_bdim,
0430:         rstd_value, rstd_bdim,
0431:         N, C, HxW, group
0432:     );
0433:     grad_input = makeBatched(std::move(tensor), 0, cur_level);
0434:   }
0435:   return std::make_tuple(std::move(grad_input), std::move(grad_weight), std::move(grad_bias));
0436: }
0437: 
0438: static bool has_same_shape(
0439:     const Tensor& tensor, std::optional<int64_t> tensor_bdim,
0440:     c10::SymIntArrayRef normalized_shape) {
0441:   if (!tensor.defined()) {
0442:     return true;
0443:   }
0444:   if (rankWithoutBatchDim(tensor, tensor_bdim) != static_cast<int64_t>(normalized_shape.size())) {
0445:     return false;
0446:   }
0447:   const auto tensor_shape = tensor.sizes();
0448:   for (const auto i : c10::irange(normalized_shape.size())) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `unwrapTensorAtLevel`, `guard`, `make_tuple`, `has_same_shape`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`unwrapTensorAtLevel`, `guard`, `make_tuple`, `has_same_shape`。

### Lines 449-479 / 第 449-479 行

```cpp
0449:     auto j = i;
0450:     // (0, 1, 2), 1 -> (0, 2, 3)
0451:     if (tensor_bdim.has_value() && static_cast<int64_t>(i) >= tensor_bdim.value()) {
0452:       j = j + 1;
0453:     }
0454:     if (normalized_shape[i] != tensor_shape[j]) {
0455:       return false;
0456:     }
0457:   }
0458:   return true;
0459: }
0460: 
0461: static C10_ALWAYS_INLINE void check_same_shape(
0462:     const Tensor& tensor, std::optional<int64_t> tensor_bdim,
0463:     c10::SymIntArrayRef normalized_shape, const std::string& name) {
0464:   TORCH_CHECK(has_same_shape(tensor, tensor_bdim, normalized_shape),
0465:       "Expected ", name, " to be of same shape as normalized_shape, but got ",
0466:       name, " of shape ",
0467:       tensor.sizes(),
0468:       " and normalized_shape = ",
0469:       normalized_shape);
0470: }
0471: 
0472: // Ugh, hard to deduplicate
0473: static C10_ALWAYS_INLINE void _check_layer_norm_inputs(
0474:     SymIntArrayRef normalized_shape,
0475:     const Tensor& weight, std::optional<int64_t> weight_bdim,
0476:     const Tensor& bias, std::optional<int64_t> bias_bdim) {
0477: 
0478:   const auto normalized_ndim = normalized_shape.size();
0479:   TORCH_CHECK(
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `check_same_shape`, `_check_layer_norm_inputs`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`check_same_shape`, `_check_layer_norm_inputs`。

### Lines 480-513 / 第 480-513 行

```cpp
0480:       normalized_ndim >= 1,
0481:       "Expected normalized_shape to be at least 1-dimensional, i.e., ",
0482:       "containing at least one element, but got normalized_shape = ",
0483:       normalized_shape);
0484:   check_same_shape(weight, weight_bdim, normalized_shape, "weight");
0485:   check_same_shape(bias, bias_bdim, normalized_shape, "weight");
0486: }
0487: 
0488: static std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>>
0489: native_layer_norm_batch_rule(
0490:     const Tensor& input, std::optional<int64_t> input_bdim,
0491:     c10::SymIntArrayRef normalized_shape,
0492:     const std::optional<Tensor>& weight_opt, std::optional<int64_t> weight_bdim,
0493:     const std::optional<Tensor>& bias_opt, std::optional<int64_t> bias_bdim,
0494:     double eps) {
0495:   auto input_ = moveBatchDimToFront(input, input_bdim);
0496:   if (!weight_bdim && !bias_bdim) {
0497:     auto [result0, mean, rstd] = at::native_layer_norm_symint(input_, normalized_shape, weight_opt, bias_opt, eps);
0498:     const auto stats_bdim = compute_stat_bdim(input_bdim, mean);
0499:     return std::make_tuple(std::move(result0), 0, std::move(mean), stats_bdim, std::move(rstd), stats_bdim);
0500:   }
0501: 
0502:   // See [Note: hacky wrapper removal for optional tensor]
0503:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0504:   const Tensor& weight = *weight_maybe_owned;
0505:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0506:   const Tensor& bias = *bias_maybe_owned;
0507:   _check_layer_norm_inputs(normalized_shape, weight, weight_bdim, bias, bias_bdim);
0508: 
0509:   const auto input_logical_rank = rankWithoutBatchDim(input, input_bdim);
0510:   const auto result = at::native_layer_norm_symint(input_, normalized_shape, std::nullopt, std::nullopt, eps);
0511:   auto [result0, mean, rstd] = result;
0512:   const auto stats_bdim = compute_stat_bdim(input_bdim, mean);
0513: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `check_same_shape`, `native_layer_norm_batch_rule`, `make_tuple`, `_check_layer_norm_inputs`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`check_same_shape`, `native_layer_norm_batch_rule`, `make_tuple`, `_check_layer_norm_inputs`。

### Lines 514-543 / 第 514-543 行

```cpp
0514:   if (weight.defined()) {
0515:     auto weight_ = moveBatchDimToFront(weight, weight_bdim);
0516:     weight_ = maybePadToLogicalRank(weight_, /*has_bdim*/weight_bdim, input_logical_rank);
0517:     result0 = result0 * weight_;
0518:   }
0519:   if (bias.defined()) {
0520:     const auto result_logical_rank = rankWithoutBatchDim(
0521:         result0,
0522:         input_bdim.has_value() || weight_bdim.has_value() ? std::optional<int64_t>(0) : std::optional<int64_t>(std::nullopt));
0523:     auto bias_ = moveBatchDimToFront(bias, bias_bdim);
0524:     bias_ = maybePadToLogicalRank(bias_, /*has_bdim*/bias_bdim, result_logical_rank);
0525:     result0 = result0 + bias_;
0526:   }
0527:   return std::make_tuple(std::move(result0), 0, std::move(mean), stats_bdim, std::move(rstd), stats_bdim);
0528: }
0529: 
0530: static std::tuple<at::Tensor, std::optional<int64_t>> native_layer_norm_backward_no_weight_bias_batch_rule(
0531:     const at::Tensor & grad_out, std::optional<int64_t> grad_out_bdim,
0532:     const at::Tensor & input, std::optional<int64_t> input_bdim,
0533:     at::IntArrayRef normalized_shape,
0534:     const at::Tensor & mean, std::optional<int64_t> mean_bdim,
0535:     const at::Tensor & rstd, std::optional<int64_t> rstd_bdim) {
0536: 
0537:   if (!grad_out_bdim.has_value() && !input_bdim.has_value() &&
0538:       !mean_bdim.has_value() && !rstd_bdim.has_value()) {
0539:     auto result = at::native_layer_norm_backward(
0540:         grad_out, input, normalized_shape, mean, rstd, std::nullopt, std::nullopt, {true, false, false});
0541:     return std::make_tuple(std::get<0>(std::move(result)), std::nullopt);
0542:   }
0543: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `native_layer_norm_backward_no_weight_bias_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `native_layer_norm_backward_no_weight_bias_batch_rule`。

### Lines 544-581 / 第 544-581 行

```cpp
0544:   auto grad_out_ = moveBatchDimToFront(grad_out, grad_out_bdim);
0545:   auto input_ = moveBatchDimToFront(input, input_bdim);
0546:   auto mean_ = moveBatchDimToFront(mean, mean_bdim);
0547:   auto rstd_ = moveBatchDimToFront(rstd, rstd_bdim);
0548: 
0549:   // ensure grad_out / input have bdim.
0550:   const auto bdim_size = get_bdim_size2(grad_out, grad_out_bdim, input, input_bdim);
0551:   grad_out_ = ensure_has_bdim(grad_out_, grad_out_bdim.has_value(), bdim_size);
0552:   input_ = ensure_has_bdim(input_, input_bdim.has_value(), bdim_size);
0553:   mean_ = ensure_has_bdim(mean_, mean_bdim.has_value(), bdim_size);
0554:   rstd_ = ensure_has_bdim(rstd_, rstd_bdim.has_value(), bdim_size);
0555: 
0556:   auto result = at::native_layer_norm_backward(
0557:       grad_out_.contiguous(),
0558:       input_.contiguous(),
0559:       normalized_shape,
0560:       mean_.contiguous(),
0561:       rstd_.contiguous(),
0562:       std::nullopt, std::nullopt, {true, false, false});
0563: 
0564:   return std::make_tuple(std::get<0>(result), 0);
0565: }
0566: 
0567: static std::tuple<at::Tensor,at::Tensor,at::Tensor> native_layer_norm_backward_plumbing(
0568:     const at::Tensor & grad_out,
0569:     const at::Tensor & input,
0570:     at::IntArrayRef normalized_shape,
0571:     const at::Tensor & mean,
0572:     const at::Tensor & rstd,
0573:     const std::optional<at::Tensor> & weight_opt,
0574:     const std::optional<at::Tensor> & bias_opt,
0575:     std::array<bool,3> output_mask) {
0576:   // See [Note: hacky wrapper removal for optional tensor]
0577:   c10::MaybeOwned<Tensor> weight_maybe_owned = at::borrow_from_optional_tensor(weight_opt);
0578:   const Tensor& weight = *weight_maybe_owned;
0579:   c10::MaybeOwned<Tensor> bias_maybe_owned = at::borrow_from_optional_tensor(bias_opt);
0580:   const Tensor& bias = *bias_maybe_owned;
0581: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `native_layer_norm_backward_plumbing`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `native_layer_norm_backward_plumbing`。

### Lines 582-611 / 第 582-611 行

```cpp
0582:   // plumbing
0583:   auto maybe_layer = maybeCurrentDynamicLayer();
0584:   vmap_check_escaped(maybe_layer, "native_layer_norm_backward_plumbing");
0585:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0586:   int64_t cur_level = maybe_layer->layerId();
0587:   if (!areAnyBatchedAtLevel({grad_out, input, mean, rstd, weight_opt, bias_opt}, cur_level)) {
0588:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0589:     return at::native_layer_norm_backward(grad_out, input, normalized_shape, mean, rstd,
0590:         weight_opt, bias_opt, output_mask);
0591:   }
0592:   auto [grad_out_value, grad_out_bdim] = unwrapTensorAtLevel(grad_out, cur_level);
0593:   auto [input_value, input_bdim] = unwrapTensorAtLevel(input, cur_level);
0594:   auto [mean_value, mean_bdim] = unwrapTensorAtLevel(mean, cur_level);
0595:   auto [rstd_value, rstd_bdim] = unwrapTensorAtLevel(rstd, cur_level);
0596:   std::optional<Tensor> weight_value;
0597:   std::optional<int64_t> weight_bdim;
0598:   if (weight.defined()) {
0599:     std::tie(weight_value, weight_bdim) = unwrapTensorAtLevel(weight, cur_level);
0600:   }
0601:   std::optional<Tensor> bias_value;
0602:   std::optional<int64_t> bias_bdim;
0603:   if (bias.defined()) {
0604:     std::tie(bias_value, bias_bdim) = unwrapTensorAtLevel(bias, cur_level);
0605:   }
0606: 
0607:   // results
0608:   Tensor grad_bias;
0609:   Tensor grad_weight;
0610:   Tensor grad_input;
0611: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `vmap_check_escaped`, `guard`, `native_layer_norm_backward`, `tie`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`vmap_check_escaped`, `guard`, `native_layer_norm_backward`, `tie`。

### Lines 612-647 / 第 612-647 行

```cpp
0612:   if (output_mask[2] && bias_value.has_value()) {
0613:     const auto num_front_dims_to_reduce = grad_out.dim() - normalized_shape.size();
0614:     if (num_front_dims_to_reduce == 0) {
0615:       grad_bias = grad_out;
0616:     } else {
0617:       grad_bias = grad_out.sum(range(0, static_cast<int64_t>(num_front_dims_to_reduce)));
0618:     }
0619:   }
0620:   if (output_mask[1] && weight_value.has_value()) {
0621:     // NB: output isn't saved...
0622:     const auto normalized_input = (input - mean) * rstd;
0623:     const auto expanded_grad_weight = normalized_input * grad_out;
0624:     const auto num_front_dims_to_reduce =
0625:         expanded_grad_weight.dim() - normalized_shape.size();
0626:     if (num_front_dims_to_reduce == 0) {
0627:       grad_weight = expanded_grad_weight;
0628:     } else {
0629:       grad_weight = expanded_grad_weight.sum(range(0, static_cast<int64_t>(num_front_dims_to_reduce)));
0630:     }
0631:   }
0632:   if (output_mask[0]) {
0633:     const auto grad_normalized_input = weight.defined() ?
0634:       grad_out * weight : grad_out;
0635:     auto [grad_normalized_input_value, grad_normalized_input_bdim] =
0636:         unwrapTensorAtLevel(grad_normalized_input, cur_level);
0637: 
0638:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0639:     auto results = native_layer_norm_backward_no_weight_bias_batch_rule(
0640:         grad_normalized_input_value, grad_normalized_input_bdim,
0641:         input_value, input_bdim,
0642:         normalized_shape,
0643:         mean_value, mean_bdim,
0644:         rstd_value, rstd_bdim);
0645:     grad_input = makeBatched(std::get<0>(results), std::get<1>(results), cur_level);
0646:   }
0647:   return std::make_tuple(std::move(grad_input), std::move(grad_weight), std::move(grad_bias));
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; implements scan/reduction-style dataflow; supports transform-aware functorch semantics. Key symbols: `unwrapTensorAtLevel`, `guard`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；实现扫描/归约式数据流；支持面向变换的 functorch 语义。关键符号：`unwrapTensorAtLevel`, `guard`, `make_tuple`。

### Lines 648-678 / 第 648-678 行

```cpp
0648: }
0649: 
0650: template <typename F, F Func>
0651: struct NativeBatchNormBatchRuleHelper {
0652:   static std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>> apply(
0653:     const Tensor& input, std::optional<int64_t> input_bdim,
0654:     const std::optional<Tensor>& weight_opt, std::optional<int64_t> weight_bdim,
0655:     const std::optional<Tensor>& bias_opt, std::optional<int64_t> bias_bdim,
0656:     const std::optional<Tensor>& running_mean_opt, std::optional<int64_t> running_mean_bdim,
0657:     const std::optional<Tensor>& running_var_opt, std::optional<int64_t> running_var_bdim,
0658:     bool training, double momentum, double eps) {
0659:     return batch_norm_batch_rule<F, Func>(
0660:         input, input_bdim, weight_opt, weight_bdim, bias_opt, bias_bdim,
0661:         running_mean_opt, running_mean_bdim, running_var_opt, running_var_bdim, training, momentum, eps);
0662:   }
0663: };
0664: 
0665: template <typename F, F Func>
0666: struct CudnnBatchNormBatchRuleHelper {
0667:   static std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>> apply(
0668:     const Tensor& input, std::optional<int64_t> input_bdim,
0669:     const Tensor& weight_opt, std::optional<int64_t> weight_bdim,
0670:     const std::optional<Tensor>& bias_opt, std::optional<int64_t> bias_bdim,
0671:     const std::optional<Tensor>& running_mean_opt, std::optional<int64_t> running_mean_bdim,
0672:     const std::optional<Tensor>& running_var_opt, std::optional<int64_t> running_var_bdim,
0673:     bool training, double momentum, double eps) {
0674:     auto reserve = at::empty({0}, input.options().dtype(kByte));  // in experiments, reserve was never set to anything other than empty by cuda
0675:     auto res = batch_norm_batch_rule<F, Func>(
0676:         input, input_bdim, weight_opt, weight_bdim, bias_opt, bias_bdim,
0677:         running_mean_opt, running_mean_bdim, running_var_opt, running_var_bdim, training, momentum, eps);
0678:     return std::tuple_cat(res, std::make_tuple(reserve, std::nullopt));
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NativeBatchNormBatchRuleHelper`, `CudnnBatchNormBatchRuleHelper`, `apply`, `tuple_cat`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NativeBatchNormBatchRuleHelper`, `CudnnBatchNormBatchRuleHelper`, `apply`, `tuple_cat`。

### Lines 679-711 / 第 679-711 行

```cpp
0679:   }
0680: };
0681: 
0682: template <typename F, F Func>
0683: struct MiopenBatchNormBatchRuleHelper {
0684:   static std::tuple<Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>,Tensor, std::optional<int64_t>> apply(
0685:     const Tensor& input, std::optional<int64_t> input_bdim,
0686:     const Tensor& weight_opt, std::optional<int64_t> weight_bdim,
0687:     const std::optional<Tensor>& bias_opt, std::optional<int64_t> bias_bdim,
0688:     const std::optional<Tensor>& running_mean_opt, std::optional<int64_t> running_mean_bdim,
0689:     const std::optional<Tensor>& running_var_opt, std::optional<int64_t> running_var_bdim,
0690:     bool training, double momentum, double eps) {
0691:     return batch_norm_batch_rule<F, Func>(
0692:         input, input_bdim, weight_opt, weight_bdim, bias_opt, bias_bdim,
0693:         running_mean_opt, running_mean_bdim, running_var_opt, running_var_bdim, training, momentum, eps);
0694:   }
0695: };
0696: 
0697: #define NATIVE_BATCH_NORM_BATCH_RULE(fn) SINGLE_ARG(\
0698:     NativeBatchNormBatchRuleHelper<\
0699:       decltype(&ATEN_FN(fn)),\
0700:       &ATEN_FN(fn)>::apply)
0701: 
0702: #define CUDNN_BATCH_NORM_BATCH_RULE(fn) SINGLE_ARG(\
0703:    CudnnBatchNormBatchRuleHelper<\
0704:       decltype(&ATEN_FN(fn)),\
0705:       &ATEN_FN(fn)>::apply)
0706: 
0707: #define MIOPEN_BATCH_NORM_BATCH_RULE(fn) SINGLE_ARG(\
0708:     MiopenBatchNormBatchRuleHelper<\
0709:       decltype(&ATEN_FN(fn)),\
0710:       &ATEN_FN(fn)>::apply)
0711: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `MiopenBatchNormBatchRuleHelper`, `apply`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`MiopenBatchNormBatchRuleHelper`, `apply`。

### Lines 712-742 / 第 712-742 行

```cpp
0712: template <typename F, F Func>
0713: struct NativeBatchNormBackwardBatchRuleHelper {
0714:   static std::tuple<Tensor,Tensor,Tensor> apply(
0715:     const at::Tensor & grad_out,
0716:     const at::Tensor & input,
0717:     const std::optional<at::Tensor> & weight_opt,
0718:     const std::optional<at::Tensor> & running_mean_opt,
0719:     const std::optional<at::Tensor> & running_var_opt,
0720:     const std::optional<at::Tensor> & save_mean_opt,
0721:     const std::optional<at::Tensor> & save_rstd_opt,
0722:     bool training,
0723:     double eps,
0724:     std::array<bool,3> output_mask) {
0725: 
0726:     auto maybe_layer = maybeCurrentDynamicLayer();
0727:     vmap_check_escaped(maybe_layer, "NativeBatchNormBackwardBatchRuleHelper.apply");
0728:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0729:     int64_t cur_level = maybe_layer->layerId();
0730: 
0731:     if (!areAnyBatchedAtLevel({grad_out, input, weight_opt, running_mean_opt,
0732:           running_var_opt, save_mean_opt, save_rstd_opt}, cur_level)) {
0733:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0734:       return at::native_batch_norm_backward(grad_out, input, weight_opt,
0735:           running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt,
0736:           training, eps, output_mask);
0737:     }
0738: 
0739:     return batch_norm_backward_plumbing<F, Func>(
0740:         grad_out, input, weight_opt, running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, training, eps, output_mask);
0741:   }
0742: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NativeBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `native_batch_norm_backward`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NativeBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `native_batch_norm_backward`。

### Lines 743-772 / 第 743-772 行

```cpp
0743: 
0744: template <typename F, F Func>
0745: struct CudnnBatchNormBackwardBatchRuleHelper {
0746:   static std::tuple<Tensor,Tensor,Tensor> apply(
0747:     const at::Tensor & input,
0748:     const at::Tensor & grad_out,
0749:     const at::Tensor & weight,
0750:     const std::optional<at::Tensor> & running_mean_opt,
0751:     const std::optional<at::Tensor> & running_var_opt,
0752:     const std::optional<at::Tensor> & save_mean_opt,
0753:     const std::optional<at::Tensor> & save_rstd_opt,
0754:     double eps,
0755:     const at::Tensor & reserve) {
0756: 
0757:     auto maybe_layer = maybeCurrentDynamicLayer();
0758:     vmap_check_escaped(maybe_layer, "CudnnBatchNormBackwardBatchRuleHelper.apply");
0759:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0760:     int64_t cur_level = maybe_layer->layerId();
0761: 
0762:     if (!areAnyBatchedAtLevel({input, grad_out, weight, running_mean_opt,
0763:           running_var_opt, save_mean_opt, save_rstd_opt, reserve}, cur_level)) {
0764:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0765:       return at::cudnn_batch_norm_backward(input, grad_out, weight,
0766:           running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, eps, reserve);
0767:     }
0768: 
0769:     return batch_norm_backward_plumbing<F, Func>(
0770:         grad_out, input, weight, running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, true, eps, {true, true, true});
0771:   }
0772: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `CudnnBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `cudnn_batch_norm_backward`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`CudnnBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `cudnn_batch_norm_backward`。

### Lines 773-802 / 第 773-802 行

```cpp
0773: 
0774: template <typename F, F Func>
0775: struct MiopenBatchNormBackwardBatchRuleHelper {
0776:   static std::tuple<Tensor,Tensor,Tensor> apply(
0777:     const at::Tensor & input,
0778:     const at::Tensor & grad_out,
0779:     const at::Tensor & weight,
0780:     const std::optional<at::Tensor> & running_mean_opt,
0781:     const std::optional<at::Tensor> & running_var_opt,
0782:     const std::optional<at::Tensor> & save_mean_opt,
0783:     const std::optional<at::Tensor> & save_rstd_opt,
0784:     double eps) {
0785: 
0786:     auto maybe_layer = maybeCurrentDynamicLayer();
0787:     vmap_check_escaped(maybe_layer, "MiopenBatchNormBackwardBatchRuleHelper.apply");
0788:     // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0789:     int64_t cur_level = maybe_layer->layerId();
0790: 
0791:     if (!areAnyBatchedAtLevel({input, grad_out, weight, running_mean_opt,
0792:           running_var_opt, save_mean_opt, save_rstd_opt}, cur_level)) {
0793:       c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0794:       return at::miopen_batch_norm_backward(input, grad_out, weight,
0795:           running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, eps);
0796:     }
0797: 
0798:     return batch_norm_backward_plumbing<F, Func>(
0799:         grad_out, input, weight, running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, true, eps, {true, true, true});
0800:   }
0801: };
0802: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `MiopenBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `miopen_batch_norm_backward`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`MiopenBatchNormBackwardBatchRuleHelper`, `apply`, `vmap_check_escaped`, `guard`, `miopen_batch_norm_backward`。

### Lines 803-833 / 第 803-833 行

```cpp
0803: #define NATIVE_BATCH_NORM_BACKWARD_BATCH_RULE(fn) SINGLE_ARG(\
0804:     NativeBatchNormBackwardBatchRuleHelper<\
0805:       decltype(&ATEN_FN(fn)),\
0806:       &ATEN_FN(fn)>::apply)
0807: 
0808: #define CUDNN_BATCH_NORM_BACKWARD_BATCH_RULE(fn) SINGLE_ARG(\
0809:    CudnnBatchNormBackwardBatchRuleHelper<\
0810:       decltype(&fn),\
0811:       &fn>::apply)
0812: 
0813: #define MIOPEN_BATCH_NORM_BACKWARD_BATCH_RULE(fn) SINGLE_ARG(\
0814:     MiopenBatchNormBackwardBatchRuleHelper<\
0815:       decltype(&fn),\
0816:       &fn>::apply)
0817: 
0818: static std::tuple<at::Tensor,at::Tensor,at::Tensor> cudnn_batch_norm_backward_wrapper(
0819:     const at::Tensor & grad_out,
0820:     const at::Tensor & input,
0821:     const at::Tensor& weight_opt,
0822:     const std::optional<at::Tensor> & running_mean_opt,
0823:     const std::optional<at::Tensor> & running_var_opt,
0824:     const std::optional<at::Tensor> & save_mean_opt,
0825:     const std::optional<at::Tensor> & save_rstd_opt,
0826:     bool training,
0827:     double eps,
0828:     std::array<bool,3> output_mask) {
0829:     TORCH_INTERNAL_ASSERT(!training);
0830:     auto reserve = at::empty({0}, input.options().dtype(kByte));
0831:     return at::cudnn_batch_norm_backward(input, grad_out, weight_opt, running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, eps, reserve);
0832:   }
0833: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `cudnn_batch_norm_backward`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`cudnn_batch_norm_backward`。

### Lines 834-864 / 第 834-864 行

```cpp
0834: static std::tuple<at::Tensor,at::Tensor,at::Tensor> miopen_batch_norm_backward_wrapper(
0835:     const at::Tensor & grad_out,
0836:     const at::Tensor & input,
0837:     const at::Tensor& weight_opt,
0838:     const std::optional<at::Tensor> & running_mean_opt,
0839:     const std::optional<at::Tensor> & running_var_opt,
0840:     const std::optional<at::Tensor> & save_mean_opt,
0841:     const std::optional<at::Tensor> & save_rstd_opt,
0842:     bool training,
0843:     double eps,
0844:     std::array<bool,3> output_mask) {
0845:     TORCH_INTERNAL_ASSERT(!training); // this should be ensured by batch_norm_impl
0846:     return at::miopen_batch_norm_backward(input, grad_out, weight_opt, running_mean_opt, running_var_opt, save_mean_opt, save_rstd_opt, eps);
0847:   }
0848: 
0849: // NB: This is NOT good. In the ideal world, we do NOT want to convert the new legit op back into native_batch_norm
0850: // as native_batch_norm has a problematic schema--it promises it is functional when it is not. However, vmap doesn't
0851: // work with dynamo anyway so we gain some buffer room to do wrong things here. The (reasonable) hope is that we will
0852: // make native_batch_norm composite implicit within a few weeks and we can fix this before vmap works with dynamo.
0853: static std::tuple<at::Tensor,at::Tensor,at::Tensor> _native_batch_norm_legit_batch(
0854:   const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt,
0855:   Tensor& running_mean, Tensor& running_var, bool train, double momentum, double eps) {
0856:     return at::native_batch_norm(self, weight_opt, bias_opt, running_mean, running_var, train, momentum, eps);
0857: }
0858: 
0859: static std::tuple<at::Tensor,at::Tensor,at::Tensor> _native_batch_norm_legit_no_stats_batch(
0860:   const Tensor& self, const std::optional<Tensor>& weight_opt, const std::optional<Tensor>& bias_opt,
0861:   bool train, double momentum, double eps) {
0862:     return at::native_batch_norm(self, weight_opt, bias_opt, Tensor(), Tensor(), train, momentum, eps);
0863: }
0864: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; manages memory allocation, buffers, or ownership boundaries; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `miopen_batch_norm_backward_wrapper`, `miopen_batch_norm_backward`, `_native_batch_norm_legit_batch`, `native_batch_norm`, `_native_batch_norm_legit_no_stats_batch`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；管理内存分配、缓冲区或所有权边界；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`miopen_batch_norm_backward_wrapper`, `miopen_batch_norm_backward`, `_native_batch_norm_legit_batch`, `native_batch_norm`, `_native_batch_norm_legit_no_stats_batch`。

### Lines 865-880 / 第 865-880 行

```cpp
0865: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0866:   VMAP_SUPPORT(native_batch_norm, NATIVE_BATCH_NORM_BATCH_RULE(native_batch_norm));
0867:   VMAP_SUPPORT(cudnn_batch_norm, CUDNN_BATCH_NORM_BATCH_RULE(cudnn_batch_norm));
0868:   VMAP_SUPPORT(miopen_batch_norm, MIOPEN_BATCH_NORM_BATCH_RULE(miopen_batch_norm));
0869:   m.impl("_native_batch_norm_legit", _native_batch_norm_legit_batch);
0870:   m.impl("_native_batch_norm_legit.no_stats", _native_batch_norm_legit_no_stats_batch);
0871:   m.impl("native_batch_norm_backward", NATIVE_BATCH_NORM_BACKWARD_BATCH_RULE(native_batch_norm_backward));
0872:   m.impl("cudnn_batch_norm_backward", CUDNN_BATCH_NORM_BACKWARD_BATCH_RULE(at::functorch::cudnn_batch_norm_backward_wrapper));
0873:   m.impl("miopen_batch_norm_backward", MIOPEN_BATCH_NORM_BACKWARD_BATCH_RULE(at::functorch::miopen_batch_norm_backward_wrapper));
0874:   m.impl("native_group_norm", native_group_norm_plumbing);
0875:   m.impl("native_group_norm_backward", native_group_norm_backward_plumbing);
0876:   VMAP_SUPPORT(native_layer_norm, native_layer_norm_batch_rule);
0877:   m.impl("native_layer_norm_backward", native_layer_norm_backward_plumbing);
0878: }
0879: 
0880: } // namespace at::functorch
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesNorm` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesNorm` 的行为。符号：无明显局部符号。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Device and stream coordination** — 设备与流协调
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: NativeBatchNormBatchRuleHelper, CudnnBatchNormBatchRuleHelper, MiopenBatchNormBatchRuleHelper, NativeBatchNormBackwardBatchRuleHelper, CudnnBatchNormBackwardBatchRuleHelper, MiopenBatchNormBackwardBatchRuleHelper, is_empty_tensor, compute_stat_bdim** — 核心符号：NativeBatchNormBatchRuleHelper、CudnnBatchNormBatchRuleHelper、MiopenBatchNormBatchRuleHelper、NativeBatchNormBackwardBatchRuleHelper、CudnnBatchNormBackwardBatchRuleHelper、MiopenBatchNormBackwardBatchRuleHelper、is_empty_tensor、compute_stat_bdim

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `ATen/functorch/PlumbingHelper.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `NativeBatchNormBatchRuleHelper`, `CudnnBatchNormBatchRuleHelper`, `MiopenBatchNormBatchRuleHelper`, `NativeBatchNormBackwardBatchRuleHelper`, `CudnnBatchNormBackwardBatchRuleHelper`, `MiopenBatchNormBackwardBatchRuleHelper`, `is_empty_tensor`, `compute_stat_bdim`, `padRight`, `new_sizes`, `batch_norm_batch_rule`, `make_tuple`, `...`
