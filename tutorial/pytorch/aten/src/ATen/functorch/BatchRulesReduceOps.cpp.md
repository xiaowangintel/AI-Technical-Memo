# BatchRulesReduceOps.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesReduceOps.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesReduceOps.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Descriptor/handle lifecycle management is important here. The code emphasizes scan/reduction style data movement and parallel aggregation. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesReduceOps.cpp` 展开。 文件头部注释也概括了其核心职责。 描述符/句柄的生命周期管理是这里的重要内容。 代码强调扫描/归约式的数据移动与并行聚合。 该文件范围内还包含 functorch 所需的变换感知张量语义。

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
0009: #include <ATen/core/dispatch/Dispatcher.h>
0010: 
0011: #include <utility>
0012: 
0013: // NOLINTBEGIN(bugprone-unchecked-optional-access)
0014: namespace at::functorch {
0015: 
0016: static bool is_allowed_dim_on_scalar_tensor(int64_t dim) {
0017:   return dim == 0 || dim == -1;
0018: }
0019: 
0020: static Tensor sum_decomp(
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `is_allowed_dim_on_scalar_tensor`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`is_allowed_dim_on_scalar_tensor`。

### Lines 21-40 / 第 21-40 行

```cpp
0021:     const Tensor& self, std::optional<ScalarType> dtype) {
0022:   return at::sum(self, range(0, self.dim()), false, dtype);
0023: }
0024: 
0025: static std::tuple<Tensor, std::optional<int64_t>> _is_all_true_batch_rule(
0026:     const Tensor& self, std::optional<int64_t> self_bdim) {
0027:   return std::make_tuple(at::_is_all_true(self), std::nullopt);
0028: }
0029: 
0030: static std::tuple<Tensor, std::optional<int64_t>> _is_any_true_batch_rule(
0031:      const Tensor& self, std::optional<int64_t> self_bdim) {
0032:    return std::make_tuple(at::_is_any_true(self), std::nullopt);
0033:  }
0034: 
0035: static Tensor mean_decomp(
0036:     const Tensor& self, std::optional<ScalarType> dtype) {
0037:   return at::mean(self, range(0, self.dim()), false, dtype);
0038: }
0039: 
0040: static Tensor prod_decomp(
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `sum`, `_is_all_true_batch_rule`, `make_tuple`, `_is_any_true_batch_rule`, `mean_decomp`, `mean`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`sum`, `_is_all_true_batch_rule`, `make_tuple`, `_is_any_true_batch_rule`, `mean_decomp`, `mean`。

### Lines 41-60 / 第 41-60 行

```cpp
0041:     const Tensor& self, std::optional<ScalarType> dtype) {
0042:   return at::prod(self.flatten(), 0, false, dtype);
0043: }
0044: 
0045: static Tensor max_decomp(
0046:     const Tensor& self) {
0047:   return std::get<0>(at::max(self.flatten(), 0, false));
0048: }
0049: 
0050: static Tensor min_decomp(
0051:     const Tensor& self) {
0052:   return std::get<0>(at::min(self.flatten(), 0, false));
0053: }
0054: 
0055: static Tensor norm_scalar_decomp(
0056:     const Tensor& self, const Scalar& p) {
0057:   return at::norm(self, p, range(0, self.dim()), false);
0058: }
0059: 
0060: static Tensor nanmedian_decomp(
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: `prod`, `max_decomp`, `min_decomp`, `norm_scalar_decomp`, `norm`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：`prod`, `max_decomp`, `min_decomp`, `norm_scalar_decomp`, `norm`。

### Lines 61-83 / 第 61-83 行

```cpp
0061:     const Tensor& self) {
0062:   return std::get<0>(at::nanmedian(self.flatten(), 0, false));
0063: }
0064: 
0065: static Tensor median_decomp(
0066:     const Tensor& self) {
0067:   return std::get<0>(at::median(self.flatten(), 0, false));
0068: }
0069: 
0070: static Tensor all_decomp(const Tensor& self) {
0071:   return at::all(self.flatten(), 0, false);
0072: }
0073: 
0074: static Tensor any_decomp(const Tensor& self) {
0075:   return at::any(self.flatten(), 0, false);
0076: }
0077: 
0078: enum class ReductionCase:uint8_t { DimArray, Dim };
0079: 
0080: // Macros and templates have a difficult time dealing with enums,
0081: // so we didn't turn this into an enum.
0082: // See NOTE: [keepdim cases] for explanation of what these are.
0083: static constexpr int KEEPDIM_CASE_FALSE = 0;
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ReductionCase`, `median_decomp`, `all_decomp`, `all`, `any_decomp`, `any`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ReductionCase`, `median_decomp`, `all_decomp`, `all`, `any_decomp`, `any`。

### Lines 84-106 / 第 84-106 行

```cpp
0084: static constexpr int KEEPDIM_CASE_TRUE = 1;
0085: static constexpr int KEEPDIM_CASE_VARIABLE = 2;
0086: 
0087: // dim_arg_pos allows us to specify the location of the dim/dim array argument.
0088: // For most PyTorch ops, this is equal to 1.
0089: //
0090: // NOTE: [keepdim cases]
0091: // The operator in question either:
0092: // - has a keepdim argument (KeepdimCase.Variable)
0093: //   In this case, `maybe_keepdim_arg_pos` says where the index of the keepdim arg is.
0094: //   example: sum(tensor, dim, keepdim)
0095: // - always does a reduction with no keepdim (KeepdimCase.False)
0096: //   that is, the rank of the output tensor is less than the rank of the input tensor.
0097: // - always does a reduction with keepdim=True semantics (KeepdimCase.True)
0098: //   That is, the rank of the output tensor is always the same as that of the input.
0099: //   examples: log_softmax(tensor, dim), cumsum(tensor, dim)
0100: template<
0101:   int dim_arg_pos,
0102:   int keepdim_case,
0103:   // optional cannot be used in a template, otherwise we would use it here.
0104:   int maybe_keepdim_arg_pos
0105: >
0106: static void boxed_reduction_batch_rule(const c10::OperatorHandle& op, torch::jit::Stack* stack) {
```

- **EN:** Builds a reusable template/helper layer around `BatchRulesReduceOps`. Key symbols: `boxed_reduction_batch_rule`.
- **CN:** 围绕 `BatchRulesReduceOps` 构建可复用的模板或辅助层。关键符号：`boxed_reduction_batch_rule`。

### Lines 107-127 / 第 107-127 行

```cpp
0107:   const auto& schema = op.schema();
0108:   const auto num_returns = schema.returns().size();
0109:   const auto num_arguments = schema.arguments().size();
0110: 
0111:   c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0112:   auto maybe_layer = maybeCurrentDynamicLayer();
0113:   vmap_check_escaped(maybe_layer, "boxed_reduction_batch_rule");
0114:   int64_t cur_level = maybe_layer->layerId();
0115: 
0116:   auto orig_arguments = torch::jit::last(*stack, num_arguments);
0117:   if (std::none_of(orig_arguments.begin(), orig_arguments.end(), ivalueParticipatesInCurrentLevel)) {
0118:     c10::impl::ExcludeDispatchKeyGuard guard_2(DispatchKey::FuncTorchBatched);
0119:     op.callBoxed(stack);
0120:     return;
0121:   }
0122: 
0123:   auto arguments = torch::jit::pop(*stack, num_arguments);
0124: 
0125:   TORCH_INTERNAL_ASSERT(arguments[0].isTensor());
0126:   auto [self, self_bdim] = unwrapTensorAtLevel(arguments[0].toTensor(), cur_level);
0127: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `guard`, `vmap_check_escaped`, `guard_2`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`guard`, `vmap_check_escaped`, `guard_2`。

### Lines 128-147 / 第 128-147 行

```cpp
0128:   self = moveBatchDimToFront(self, self_bdim);
0129: 
0130:   auto logical_dim = rankWithoutBatchDim(self, self_bdim);
0131:   std::vector<int64_t> dims;
0132:   ReductionCase reduction_case{};
0133:   if (arguments[dim_arg_pos].isIntList()) {
0134:     reduction_case = ReductionCase::DimArray;
0135:     dims = arguments[dim_arg_pos].toIntList().vec();
0136:     if (dims.empty()) {
0137:       auto all_dims = range(0, std::max(static_cast<int64_t>(1), logical_dim));
0138:       dims = std::vector<int64_t>(all_dims.begin(), all_dims.end());
0139:     }
0140:   } else if (arguments[dim_arg_pos].isInt()) {
0141:     reduction_case = ReductionCase::Dim;
0142:     dims = {arguments[dim_arg_pos].toInt()};
0143:   } else if (arguments[dim_arg_pos].isNone())  {
0144:     auto param_type = schema.arguments()[dim_arg_pos].type()->expect<OptionalType>()->getElementType();
0145:     if (param_type->kind() == IntType::Kind) {
0146:       reduction_case = ReductionCase::Dim;
0147:       if (self.dim() > 1) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 148-168 / 第 148-168 行

```cpp
0148:         self = self.flatten(1);
0149:       }
0150:       dims = {0};
0151:     } else if (param_type->kind() == ListType::Kind) {
0152:       reduction_case = ReductionCase::DimArray;
0153:       if (logical_dim == 0) {
0154:         dims = {0};
0155:       } else {
0156:         auto all_dims = range(0, self.dim() - 1);
0157:         dims = std::vector<int64_t>(all_dims.begin(), all_dims.end());
0158:       }
0159:     } else {
0160:       TORCH_INTERNAL_ASSERT(false, "Unexpected dtype found at dims");
0161:     }
0162:   } else{
0163:     TORCH_INTERNAL_ASSERT(false, "Unexpected dtype found at dims");
0164:   }
0165: 
0166:   VmapDimVector new_dims;
0167:   new_dims.reserve(dims.size());
0168:   for (auto dim: dims) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 169-191 / 第 169-191 行

```cpp
0169:     new_dims.push_back(getPhysicalDim(self, self_bdim.has_value(), dim));
0170:   }
0171:   bool is_scalar_case = logical_dim == 0 && dims.size() == 1 && is_allowed_dim_on_scalar_tensor(dims[0]);
0172:   std::optional<bool> maybe_keepdim;
0173:   if (is_scalar_case) {
0174:     // NOTE: [boxed_reduction_batch_rule scalar tensor handling]
0175:     // Reduction operations in PyTorch have an edge case where they allow
0176:     // dim=0 and dim=-1 if the tensor has shape [].
0177:     //
0178:     // This can come up if we do something like
0179:     // vmap(lambda x: x.sum(0))(torch.tensor([10.])),
0180:     //
0181:     // In order to handle this edge case, we unsqueeze a dimension on the Tensor,
0182:     // run the operation (with dim=1 instead), and then process the output tensor.
0183:     // There are two cases:
0184:     // - keepdim = True
0185:     //     unsqueeze   op      squeeze
0186:     //   [B] -> [B, 1] -> [B, 1] -> [B]
0187:     // - keepdim = False
0188:     //     unsqueeze   op     no need to squeeze
0189:     //   [B] -> [B, 1] -> [B]
0190:     // if keepdim is True, then we need to squeeze the dimension of size 1.
0191: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 192-214 / 第 192-214 行

```cpp
0192:     // Determine the value of keepdim
0193:     switch (keepdim_case) {
0194:       case KEEPDIM_CASE_FALSE:
0195:         maybe_keepdim = false;
0196:         break;
0197:       case KEEPDIM_CASE_TRUE:
0198:         maybe_keepdim = true;
0199:         break;
0200:       case KEEPDIM_CASE_VARIABLE:
0201:         TORCH_INTERNAL_ASSERT(maybe_keepdim_arg_pos >= 0);
0202:         maybe_keepdim = arguments[maybe_keepdim_arg_pos].toBool();
0203:         break;
0204:     }
0205:     self = self.unsqueeze(-1);
0206:     new_dims = {1};
0207:   }
0208:   arguments[0] = std::move(self);
0209:   if (reduction_case == ReductionCase::DimArray) {
0210:     arguments[dim_arg_pos] = std::vector<int64_t>(new_dims.begin(), new_dims.end());
0211:   } else if (reduction_case == ReductionCase::Dim) {
0212:     arguments[dim_arg_pos] = new_dims[0];
0213:   }
0214:   for (const auto arg_idx : c10::irange(0, num_arguments)) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; validates runtime invariants before continuing. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；在继续执行前校验运行时不变量。关键符号：无明显局部符号。

### Lines 215-236 / 第 215-236 行

```cpp
0215:     torch::jit::push(stack, arguments[arg_idx]);
0216:   }
0217:   op.callBoxed(stack);
0218: 
0219:   auto returns = torch::jit::pop(*stack, num_returns);
0220:   for (auto& ret : returns) {
0221:     if (ret.isTensor()) {
0222:       auto res = ret.toTensor();
0223:       // see NOTE: [boxed_reduction_batch_rule scalar tensor handling]
0224:       if (is_scalar_case && maybe_keepdim.value()) {
0225:         // squeeze(-1) is a no-op if the shape of the dim is not 1.
0226:         // To make it safer, we internal assert here.
0227:         TORCH_INTERNAL_ASSERT(res.size(-1) == 1);
0228:         res = res.squeeze(-1);
0229:       }
0230:       torch::jit::push(stack, makeBatched(std::move(res), 0, cur_level));
0231:     } else {
0232:       TORCH_INTERNAL_ASSERT(false, "This boxed batching rule does not currently support ops that return non-tensor values");
0233:     }
0234:   }
0235: }
0236: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `push`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`push`。

### Lines 237-258 / 第 237-258 行

```cpp
0237: // Skipping all/any since they don't have opinfo tests right now :P
0238: 
0239: static Tensor dist_decomp(const Tensor& self, const Tensor& other, const Scalar& p) {
0240:   return at::norm((self - other), p);
0241: }
0242: 
0243: static std::tuple<Tensor, Tensor> expand_bdims(
0244:     const Tensor& a, bool a_has_bdim,
0245:     const Tensor& b, bool b_has_bdim) {
0246:   Tensor flagpole;
0247:   if (a_has_bdim) {
0248:     flagpole = a;
0249:   } else if (b_has_bdim) {
0250:     flagpole = b;
0251:   } else {
0252:     TORCH_INTERNAL_ASSERT(false);
0253:   }
0254:   return std::make_tuple(
0255:       a_has_bdim ? a : a.expand_as(flagpole),
0256:       b_has_bdim ? b : b.expand_as(flagpole));
0257: }
0258: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `dist_decomp`, `norm`, `expand_bdims`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`dist_decomp`, `norm`, `expand_bdims`, `make_tuple`。

### Lines 259-278 / 第 259-278 行

```cpp
0259: static std::tuple<Tensor, std::optional<int64_t>> _softmax_backward_batch_rule(
0260:     const Tensor& grad_output, std::optional<int64_t> grad_output_bdim,
0261:     const Tensor& output, std::optional<int64_t> output_bdim,
0262:     int64_t dim,
0263:     ScalarType input_dtype) {
0264:   // softmax_backward's decomposition is y * gy - y * (y * gy).sum(dim, keepdim=True)
0265:   // NB: the CUDA kernel handles strides so we can just expand
0266:   // all of the tensors and call it a day. The CPU kernel is not as good but
0267:   // idk if the perf on that really matters
0268:   auto grad_output_ = moveBatchDimToFront(grad_output, grad_output_bdim);
0269:   auto output_ = moveBatchDimToFront(output, output_bdim);
0270: 
0271:   // Expand out that extra dimension for everyone
0272:   std::tie(grad_output_, output_) = expand_bdims(
0273:       grad_output_, grad_output_bdim.has_value(),
0274:       output_, output_bdim.has_value());
0275: 
0276:   // Scalar tensor case. softmax turns into the identity when this happens.
0277:   // I don't know why the output is zeros, though, but that's what softmax tells me...
0278:   if (output_.dim() == 1 && (dim == 0 || dim == -1)) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: `_softmax_backward_batch_rule`, `tie`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：`_softmax_backward_batch_rule`, `tie`。

### Lines 279-301 / 第 279-301 行

```cpp
0279:     return std::make_tuple(at::zeros_like(grad_output_), 0);
0280:   }
0281: 
0282:   dim = getPhysicalDim(output_, /*has_batch_dim*/true, dim);
0283: 
0284:   // Not sure why output_ needs to be marked as .contiguous(). Something must
0285:   // have changed in PyTorch (and output of softmax is probably always contiguous)
0286:   return std::make_tuple(at::_softmax_backward_data(grad_output_, output_.contiguous(), dim, input_dtype), 0);
0287: }
0288: 
0289: static std::tuple<Tensor, std::optional<int64_t>> _log_softmax_backward_batch_rule(
0290:     const Tensor& grad_output, std::optional<int64_t> grad_output_bdim,
0291:     const Tensor& output, std::optional<int64_t> output_bdim,
0292:     int64_t dim,
0293:     c10::ScalarType input_dtype) {
0294:   // NB: It turns out that expanding + calling log_softmax_backward is generally
0295:   // faster than the decomposition.
0296:   // Benchmark here: https://gist.github.com/zou3519/ae3b33b5730a84aae8a80a05c89e078a
0297:   // Decomposition is (grad_output - grad_output.sum(dim, keepdim=True) * result.exp())
0298:   // We can squeeze out a last mile of performance by writing custom kernels.
0299:   auto grad_output_ = moveBatchDimToFront(grad_output, grad_output_bdim);
0300:   auto output_ = moveBatchDimToFront(output, output_bdim);
0301: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `_log_softmax_backward_batch_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `_log_softmax_backward_batch_rule`。

### Lines 302-329 / 第 302-329 行

```cpp
0302:   // Expand out that extra dimension for everyone
0303:   std::tie(grad_output_, output_) = expand_bdims(
0304:       grad_output_, grad_output_bdim.has_value(),
0305:       output_, output_bdim.has_value());
0306: 
0307:   // Scalar tensor case. log_softmax returns zeros when this happens
0308:   if (output_.dim() == 1 && (dim == 0 || dim == -1)) {
0309:     return std::make_tuple(at::zeros_like(grad_output_), 0);
0310:   }
0311: 
0312:   dim = getPhysicalDim(output_, /*has_batch_dim*/true, dim);
0313: 
0314:   return std::make_tuple(at::_log_softmax_backward_data(grad_output_, output_, dim, input_dtype), 0);
0315: }
0316: 
0317: static std::tuple<Tensor, std::optional<int64_t>> searchsorted_batch_rule(
0318:     const Tensor& sorted_sequence,
0319:     std::optional<int64_t> sorted_sequence_bdim,
0320:     const Tensor& self,
0321:     std::optional<int64_t> self_bdim,
0322:     bool out_int32,
0323:     bool right,
0324:     std::optional<std::string_view> side,
0325:     const std::optional<Tensor>& sorter,
0326:     std::optional<int64_t> sorter_bdim) {
0327:   auto buckets_logical_rank = rankWithoutBatchDim(sorted_sequence, sorted_sequence_bdim);
0328:   auto self_logical_rank = rankWithoutBatchDim(self, self_bdim);
0329: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `tie`, `make_tuple`, `searchsorted_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`tie`, `make_tuple`, `searchsorted_batch_rule`。

### Lines 330-352 / 第 330-352 行

```cpp
0330:   // Preprocess sorter and sorted_sequence.
0331:   // If they both exist, and only one has a bdim, then we need to make sure both do.
0332:   // After this step, we can forget about sorter for a bit.
0333:   auto buckets = moveBatchDimToFront(sorted_sequence, sorted_sequence_bdim);
0334:   std::optional<int64_t> buckets_bdim;
0335:   if (sorted_sequence_bdim.has_value()) {
0336:     buckets_bdim = 0;
0337:   }
0338: 
0339:   std::optional<Tensor> sorter_;
0340:   if (sorter.has_value() && sorter->defined()) {
0341:     auto sorter__ = moveBatchDimToFront(*sorter, sorter_bdim);
0342:     if (sorted_sequence_bdim.has_value() != sorter_bdim.has_value()) {
0343:       auto bdim_size = get_bdim_size2(
0344:           sorted_sequence, sorted_sequence_bdim,
0345:           sorter.value(), sorter_bdim);
0346:       sorter__ = ensure_has_bdim(sorter__, sorter_bdim.has_value(), bdim_size);
0347:       buckets = ensure_has_bdim(buckets, sorted_sequence_bdim.has_value(), bdim_size);
0348:       buckets_bdim = 0;
0349:     }
0350:     sorter_ = sorter__;
0351:   }
0352: 
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 353-376 / 第 353-376 行

```cpp
0353:   // Two cases: buckets_logical_rank is 1, or it is greater than 1.
0354:   // searchsorted is basically two operators with different semantics jammed
0355:   // into one
0356:   if (buckets_logical_rank > 1) {
0357:     // B<...>D, B<...>V -> no change
0358:     if (buckets_bdim.has_value() && self_bdim.has_value()) {
0359:       auto self_ = moveBatchDimToFront(self, self_bdim);
0360:       auto result = at::searchsorted(buckets, self_, out_int32, right, side, sorter_);
0361:       return std::make_tuple(std::move(result), 0);
0362:     }
0363:     // B<...>D, <...>V -> B<...>D, B<...>V
0364:     if (buckets_bdim.has_value() && !self_bdim.has_value()) {
0365:       auto self_ = moveBatchDimToFront(self, self_bdim);
0366:       self_ = ensure_has_bdim(self_, self_bdim.has_value(), buckets.size(0));
0367:       auto result = at::searchsorted(buckets, self_, out_int32, right, side, sorter_);
0368:       return std::make_tuple(std::move(result), 0);
0369:     }
0370:     // <...>D, B<...>V -> <...>D, <...>(BV)
0371:     if (!buckets_bdim.has_value() && self_bdim.has_value()) {
0372:       auto bdim_size = self.size(*self_bdim);
0373:       auto self_ = reshape_dim_into(*self_bdim, -1, self);
0374:       auto result = at::searchsorted(buckets, self_, out_int32, right, side, sorter_);
0375:       result = reshape_dim_outof(-1, bdim_size, result);
0376:       return std::make_tuple(result, result.dim() - 2);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`make_tuple`。

### Lines 377-396 / 第 377-396 行

```cpp
0377:     }
0378:     TORCH_INTERNAL_ASSERT(false);
0379:   }
0380:   // buckets_logical_rank == 1 case.
0381:   // BD, B* -> BD, B flat(*)
0382:   if (buckets_bdim.has_value() && self_bdim.has_value()) {
0383:     auto self_ = moveBatchDimToFront(self, self_bdim);
0384:     auto self_view_ = self_logical_rank == 0 ? self_.unsqueeze(-1) : self_.flatten(1);
0385:     auto result = at::searchsorted(buckets, self_view_, out_int32, right, side, sorter_);
0386:     result = self_logical_rank == 0 ? result.squeeze(-1) : result.view(self_.sizes());
0387:     return std::make_tuple(std::move(result), 0);
0388:   }
0389:   // BD, * -> BD, flat(*) -> BD, B flat(*)
0390:   if (buckets_bdim.has_value() && !self_bdim.has_value()) {
0391:     auto bdim_size = buckets.size(*buckets_bdim);
0392:     auto self_ = ensure_has_bdim(self, false, bdim_size);
0393:     auto self_view_ = self_logical_rank == 0 ? self_.unsqueeze(-1) : self_.flatten(1);
0394:     auto result = at::searchsorted(buckets, self_view_, out_int32, right, side, sorter_);
0395:     result = self_logical_rank == 0 ? result.squeeze(-1) : result.view(self_.sizes());
0396:     return std::make_tuple(std::move(result), 0);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`make_tuple`。

### Lines 397-416 / 第 397-416 行

```cpp
0397:   }
0398:   // D, B* -> no change
0399:   if (!buckets_bdim.has_value() && self_bdim.has_value()) {
0400:     auto result = at::searchsorted(buckets, self, out_int32, right, side, sorter_);
0401:     return std::make_tuple(std::move(result), self_bdim);
0402:   }
0403:   TORCH_INTERNAL_ASSERT(false);
0404: }
0405: 
0406: static Tensor bucketize_decomp_Tensor(
0407:     const Tensor& self,
0408:     const Tensor& boundaries,
0409:     bool out_int32,
0410:     bool right) {
0411:   // checking logical rank
0412:   TORCH_CHECK(boundaries.dim() == 1, "bucketize: boundaries tensor must be 1 dimension, but got dim(", boundaries.dim(), ")");
0413:   return at::searchsorted(boundaries, self, out_int32, right, std::nullopt, std::nullopt);
0414: }
0415: 
0416: static Tensor bucketize_decomp_Scalar(
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `make_tuple`, `bucketize_decomp_Tensor`, `searchsorted`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`make_tuple`, `bucketize_decomp_Tensor`, `searchsorted`。

### Lines 417-441 / 第 417-441 行

```cpp
0417:     const Scalar& self,
0418:     const Tensor& boundaries,
0419:     bool out_int32,
0420:     bool right) {
0421:   // checking logical rank
0422:   TORCH_CHECK(boundaries.dim() == 1, "bucketize: boundaries tensor must be 1 dimension, but got dim(", boundaries.dim(), ")");
0423:   return at::searchsorted(boundaries, self, out_int32, right, std::nullopt, std::nullopt);
0424: }
0425: 
0426: // Use when the other macros don't work out.
0427: // - dim_pos: index of the dim argument
0428: // - keepdim_case: either True, False, or Variable.
0429: //   See NOTE: [keepdim cases] for more details.
0430: // - maybe_keepdim_pos. The index of the keepdim argument,
0431: //   if exists. Otherwise, the value is ignored.
0432: #define REDUCTION_BOXED_ARGS(op, dim_pos, keepdim_case, maybe_keepdim_pos) \
0433:   m.impl(#op, torch::CppFunction::makeFromBoxedFunction< \
0434:       SINGLE_ARG(boxed_reduction_batch_rule<dim_pos, keepdim_case, maybe_keepdim_pos>)>());
0435: 
0436: // Provided for your convenience; most operators that have a keepdim arg
0437: // will work with this macro.
0438: // Assumes the dim arg is at position 1 and the keepdim arg is at pos 2.
0439: #define REDUCTION_WITH_KEEPDIM_ARG(op) \
0440:   REDUCTION_BOXED_ARGS(op, 1, KEEPDIM_CASE_VARIABLE, 2)
0441: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: `searchsorted`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：`searchsorted`。

### Lines 442-481 / 第 442-481 行

```cpp
0442: // Provided for your convenience; most operators that do not have a keepdim
0443: // arg will work with this macro.
0444: // Assumes the dim arg is at position 1 and the operation always returns
0445: // a tensor of the same rank (instead of a smaller rank).
0446: #define REDUCTION_NO_KEEPDIM_ARG(op) \
0447:   REDUCTION_BOXED_ARGS(op, 1, KEEPDIM_CASE_TRUE, -1)
0448: 
0449: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0450:   VMAP_SUPPORT2(searchsorted, Tensor, searchsorted_batch_rule);
0451:   REDUCTION_NO_KEEPDIM_ARG(_fft_r2c);
0452:   REDUCTION_NO_KEEPDIM_ARG(_fft_c2r);
0453:   REDUCTION_NO_KEEPDIM_ARG(_fft_c2c);
0454:   REDUCTION_WITH_KEEPDIM_ARG(amax);
0455:   REDUCTION_WITH_KEEPDIM_ARG(amin);
0456:   REDUCTION_WITH_KEEPDIM_ARG(aminmax);
0457:   m.impl("all", all_decomp);
0458:   REDUCTION_WITH_KEEPDIM_ARG(all.dim);
0459:   REDUCTION_WITH_KEEPDIM_ARG(all.dims);
0460:   m.impl("any", any_decomp);
0461:   REDUCTION_WITH_KEEPDIM_ARG(any.dim);
0462:   REDUCTION_WITH_KEEPDIM_ARG(any.dims);
0463:   REDUCTION_WITH_KEEPDIM_ARG(argmax);
0464:   REDUCTION_WITH_KEEPDIM_ARG(argmin);
0465:   m.impl("bucketize.Tensor", bucketize_decomp_Tensor);
0466:   m.impl("bucketize.Scalar", bucketize_decomp_Scalar);
0467:   REDUCTION_BOXED_ARGS(count_nonzero.dim_IntList, 1, KEEPDIM_CASE_FALSE, -1);
0468:   REDUCTION_NO_KEEPDIM_ARG(cummax);
0469:   REDUCTION_NO_KEEPDIM_ARG(cummin);
0470:   REDUCTION_NO_KEEPDIM_ARG(cumprod);
0471:   REDUCTION_NO_KEEPDIM_ARG(cumsum);
0472:   m.impl("dist", dist_decomp);
0473:   REDUCTION_BOXED_ARGS(kthvalue, 2, KEEPDIM_CASE_VARIABLE, 3);
0474:   REDUCTION_BOXED_ARGS(linalg_vector_norm, 2, KEEPDIM_CASE_VARIABLE, 3);
0475:   REDUCTION_NO_KEEPDIM_ARG(logcumsumexp);
0476:   REDUCTION_WITH_KEEPDIM_ARG(logsumexp);
0477:   m.impl("max", max_decomp);
0478:   REDUCTION_WITH_KEEPDIM_ARG(max.dim);
0479:   m.impl("mean", mean_decomp);
0480:   REDUCTION_WITH_KEEPDIM_ARG(mean.dim);
0481:   m.impl("median", median_decomp);
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesReduceOps` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesReduceOps` 的行为。符号：无明显局部符号。

### Lines 482-511 / 第 482-511 行

```cpp
0482:   REDUCTION_WITH_KEEPDIM_ARG(median.dim);
0483:   m.impl("min", min_decomp);
0484:   REDUCTION_WITH_KEEPDIM_ARG(min.dim);
0485:   REDUCTION_WITH_KEEPDIM_ARG(mode);
0486:   m.impl("nanmedian", nanmedian_decomp);
0487:   REDUCTION_WITH_KEEPDIM_ARG(nanmedian.dim);
0488:   REDUCTION_WITH_KEEPDIM_ARG(nansum);
0489:   m.impl("norm.Scalar", norm_scalar_decomp);
0490:   REDUCTION_BOXED_ARGS(norm.ScalarOpt_dim, 2, KEEPDIM_CASE_VARIABLE, 3);
0491:   m.impl("prod", prod_decomp);
0492:   REDUCTION_WITH_KEEPDIM_ARG(prod.dim_int);
0493:   REDUCTION_BOXED_ARGS(std.correction, 1, KEEPDIM_CASE_VARIABLE, 3);
0494:   REDUCTION_NO_KEEPDIM_ARG(_softmax);
0495:   REDUCTION_NO_KEEPDIM_ARG(_safe_softmax);
0496:   REDUCTION_NO_KEEPDIM_ARG(sort);
0497:   REDUCTION_BOXED_ARGS(sort.stable, 2, KEEPDIM_CASE_TRUE, -1);
0498:   REDUCTION_BOXED_ARGS(std_mean.correction, 1, KEEPDIM_CASE_VARIABLE, 3);
0499:   m.impl("sum", sum_decomp);
0500:   REDUCTION_WITH_KEEPDIM_ARG(sum.dim_IntList);
0501:   REDUCTION_BOXED_ARGS(topk, 2, KEEPDIM_CASE_TRUE, -1);
0502:   REDUCTION_BOXED_ARGS(var.correction, 1, KEEPDIM_CASE_VARIABLE, 3);
0503:   REDUCTION_BOXED_ARGS(var_mean.correction, 1, KEEPDIM_CASE_VARIABLE, 3);
0504:   REDUCTION_NO_KEEPDIM_ARG(_log_softmax);
0505:   REDUCTION_BOXED_ARGS(rot90, 2, KEEPDIM_CASE_TRUE, -1);
0506:   VMAP_SUPPORT(_log_softmax_backward_data, _log_softmax_backward_batch_rule);
0507:   VMAP_SUPPORT(_softmax_backward_data, _softmax_backward_batch_rule);
0508:   VMAP_SUPPORT(_is_all_true, _is_all_true_batch_rule);
0509:   VMAP_SUPPORT(_is_any_true, _is_any_true_batch_rule);
0510: }
0511: 
```

- **EN:** This block supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 512-513 / 第 512-513 行

```cpp
0512: } // namespace at::functorch
0513: // NOLINTEND(bugprone-unchecked-optional-access)
```

- **EN:** This block implements local helper logic for `BatchRulesReduceOps`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `BatchRulesReduceOps` 相关的局部辅助逻辑。关键符号：无明显局部符号。


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
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `ATen/functorch/PlumbingHelper.h`, `ATen/core/dispatch/Dispatcher.h`
- **External includes / 外部头文件**: `utility`
- **Namespaces / 命名空间**: `at::functorch`
- **Representative symbols / 代表性符号**: `ReductionCase`, `is_allowed_dim_on_scalar_tensor`, `sum_decomp`, `sum`, `_is_all_true_batch_rule`, `make_tuple`, `_is_any_true_batch_rule`, `mean_decomp`, `mean`, `prod_decomp`, `prod`, `max_decomp`, `...`
