# BatchRulesConvolution.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/functorch/BatchRulesConvolution.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Supports functorch-style transforms, batching, functionalization, or derivative-aware ATen behavior. This specific file centers on `BatchRulesConvolution.cpp`. The file header highlights: "All rights reserved. This source code is licensed under the BSD-style license found in the LICENSE file in the root directory of this source tree.." Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 支撑 functorch 风格的变换、批处理、函数化或与导数相关的 ATen 行为。 该文件具体围绕 `BatchRulesConvolution.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24 / 第 1-24 行

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
0012: // convolution_batch_rule translated from jax with modifications:
0013: // https://github.com/google/jax/blob/master/jax/_src/lax/lax.py#L3143
0014: 
0015: // PyTorch's convolution is different from JAX's conv_general_dilated:
0016: // we do not support batch_group_count (which is needed for convolution backwards).
0017: // Instead, there's a convolution_backward op that needs a batching rule.
0018: static std::tuple<Tensor, std::optional<int64_t>>
0019: convolution_batch_rule(const Tensor& lhs, std::optional<int64_t> lhs_bdim, const Tensor& rhs, std::optional<int64_t> rhs_bdim, const std::optional<Tensor>& bias, std::optional<int64_t> bias_bdim, c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed, c10::SymIntArrayRef output_padding, c10::SymInt groups) {
0020:   DimVector lhs_spec(stride.size() + 2);
0021:   std::iota(lhs_spec.begin(), lhs_spec.end(), 0);
0022:   DimVector rhs_spec = lhs_spec;
0023:   DimVector out_spec = lhs_spec;
0024:   if (transposed) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `convolution_batch_rule`, `lhs_spec`, `iota`.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`convolution_batch_rule`, `lhs_spec`, `iota`。

### Lines 25-48 / 第 25-48 行

```cpp
0025:     rhs_spec[0] = 1;
0026:     rhs_spec[1] = 0;
0027:   }
0028: 
0029:   // If we have a batched bias or weight, we need to perform the computation separately.
0030:   std::optional<Tensor> unbatched_bias;
0031:   bool separate_bias = false;
0032:   if ((rhs_bdim && bias && bias->defined()) || bias_bdim) {
0033:     TORCH_INTERNAL_ASSERT(bias.has_value());
0034:     TORCH_INTERNAL_ASSERT(bias->defined());
0035:     unbatched_bias = std::nullopt;
0036:     separate_bias = true;
0037:   } else {
0038:     unbatched_bias = bias;
0039:     separate_bias = false;
0040:   }
0041:   std::tuple<Tensor, std::optional<int64_t>> result;
0042:   if (lhs_bdim && !rhs_bdim) {
0043:     auto new_x = reshape_dim_into(*lhs_bdim, lhs_spec[0], lhs);
0044:     auto out = at::convolution_symint(new_x, rhs, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups);
0045:     out = reshape_dim_outof_symint(out_spec[0], lhs.sizes()[*lhs_bdim], out);
0046:     result = std::make_tuple(out, out_spec[0]);
0047:   } else if (!lhs_bdim && rhs_bdim) {
0048:     if (groups == 1) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量。关键符号：无明显局部符号。

### Lines 49-78 / 第 49-78 行

```cpp
0049:       auto new_w = reshape_dim_into(*rhs_bdim, rhs_spec[0], rhs);
0050:       auto out = at::convolution_symint(lhs, new_w, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups);
0051:       out = reshape_dim_outof_symint(out_spec[1], rhs.size(*rhs_bdim), out);
0052:       result = std::make_tuple(out, out_spec[1]);
0053:     } else {
0054:       if (transposed) {
0055:         // conv_transpose with groups is normally NIHW, IOHW -> N(GO)HW
0056:         // With RHS batched, we do the following:
0057:         // NIHW, BIOHW -> NIHW, I(BO)HW -> N(GBO)HW -> BN(GO)HW
0058:         // NB: the following isn't written using rhs_spec
0059:         // (PyTorch convs have a fixed dimension order)
0060: 
0061:         // BIOHW -> I(BO)HW
0062:         auto new_w = reshape_dim_into(*rhs_bdim, 1, rhs);
0063:         // NIHW, I(BO)HW -> N(GBO)HW
0064:         auto out = at::convolution_symint(lhs, new_w, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups);
0065:         // N(GBO)HW -> NG(BO)HW
0066:         out = reshape_dim_outof_symint(1, groups, out);
0067:         // NG(BO)HW -> NGBOHW
0068:         out = reshape_dim_outof_symint(2, rhs.size(*rhs_bdim), out);
0069:         // NGBOHW -> NB(GO)HW
0070:         out = reshape_dim_into(1, 2, out);
0071:         result = std::make_tuple(out, 1);
0072:       } else {
0073:         // conv with groups is normally N(GI)HW, (GO)IHW -> N(GO)HW
0074:         // With RHS batched, we do the following:
0075:         // N(GI)HW, B(GO)IHW -> N(GI)HW, (GBO)IHW -> N(GBO)HW -> BN(GO)HW
0076:         // NB: the following isn't written using rhs_spec
0077:         // (PyTorch convs have a fixed dimension order)
0078: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：无明显局部符号。

### Lines 79-107 / 第 79-107 行

```cpp
0079:         // B(GO)IHW -> BGOIHW
0080:         auto new_w = reshape_dim_outof_symint(0 + (*rhs_bdim == 0), groups, rhs);
0081:         // BGOIHW -> G(BO)IHW
0082:         new_w = reshape_dim_into(*rhs_bdim + (*rhs_bdim > 0), 1, new_w);
0083:         // G(BO)IHW -> (GBO)IHW
0084:         new_w = reshape_dim_into(0, 0, new_w);
0085:         // N(GI)HW, (GBO)IHW -> N(GBO)HW
0086:         auto out = at::convolution_symint(lhs, new_w, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups);
0087:         // N(GBO)HW -> NG(BO)HW
0088:         out = reshape_dim_outof_symint(1, groups, out);
0089:         // NG(BO)HW -> NGBOHW
0090:         out = reshape_dim_outof_symint(2, rhs.size(*rhs_bdim), out);
0091:         // NGBOHW -> NB(GO)HW
0092:         out = reshape_dim_into(1, 2, out);
0093:         result = std::make_tuple(out, 1);
0094:       }
0095:     }
0096:   } else if (lhs_bdim && rhs_bdim) {
0097:     auto new_x = reshape_dim_into(*lhs_bdim, lhs_spec[1], lhs);
0098:     groups *= lhs.sizes()[*lhs_bdim];
0099:     auto dim_with_groups = transposed ? 1 : 0;
0100:     auto new_w = reshape_dim_into(*rhs_bdim, rhs_spec[dim_with_groups], rhs);
0101:     auto out = at::convolution_symint(new_x, new_w, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups);
0102:     out = reshape_dim_outof_symint(out_spec[1], lhs.sizes()[*lhs_bdim], out);
0103:     result = std::make_tuple(out, out_spec[1]);
0104:   } else {
0105:     result = std::make_tuple(at::convolution_symint(lhs, rhs, unbatched_bias, stride, padding, dilation, transposed, output_padding, groups), std::nullopt);
0106:   }
0107:   if (separate_bias) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 108-135 / 第 108-135 行

```cpp
0108:     auto& [A, A_batch_dim] = result;
0109:     auto B = *bias;
0110:     auto B_batch_dim = bias_bdim;
0111:     A = moveBatchDimToFront(A, A_batch_dim);
0112:     B = moveBatchDimToFront(B, B_batch_dim);
0113:     for (size_t i = 0; i < out_spec.size() - 2; i++) {
0114:       B = B.unsqueeze(-1);
0115:     }
0116:     B = maybePadToLogicalRank(B, B_batch_dim, rankWithoutBatchDim(A, A_batch_dim));
0117: 
0118:     return std::make_tuple(at::add(A, B), 0);
0119:   } else {
0120:     return result;
0121:   }
0122: }
0123: 
0124: static Tensor _convolution_decomp(
0125:     const Tensor& input_r, const Tensor& weight_r, const std::optional<Tensor>& bias_r_opt,
0126:     IntArrayRef stride_, IntArrayRef padding_, IntArrayRef dilation_,
0127:     bool transposed_, IntArrayRef output_padding_, int64_t groups_,
0128:     bool benchmark, bool deterministic, bool cudnn_enabled, bool allow_tf32) {
0129:   // Ignore everything. If the user called this in the normal way,
0130:   // then they should be fine.
0131:   (void) benchmark;
0132:   (void) deterministic;
0133:   (void) cudnn_enabled;
0134:   (void) allow_tf32;
0135:   return at::convolution(
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `make_tuple`, `_convolution_decomp`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`make_tuple`, `_convolution_decomp`。

### Lines 136-158 / 第 136-158 行

```cpp
0136:       input_r, weight_r, bias_r_opt, stride_, padding_, dilation_, transposed_, output_padding_, groups_);
0137: }
0138: 
0139: static Tensor compute_grad_bias(
0140:     const Tensor& grad_output_, std::array<bool, 3> output_mask) {
0141:   if (!output_mask[2]) {
0142:     return Tensor();
0143:   }
0144:   DimVector reduce_dims;
0145:   reduce_dims.resize(grad_output_.dim() - 1);
0146:   reduce_dims[0] = 0;
0147:   std::iota(reduce_dims.begin() + 1, reduce_dims.end(), 2);
0148:   return grad_output_.sum(reduce_dims);
0149: }
0150: 
0151: // reshapes the batch_size into dim
0152: static Tensor make_dummy(
0153:     const Tensor& tensor, std::optional<int64_t> tensor_bdim,
0154:     int64_t dim, int64_t batch_size) {
0155:   auto tensor_ = tensor_bdim ? tensor.select(*tensor_bdim, 0) : tensor;
0156:   auto orig_size = tensor_.size(dim);
0157:   tensor_ = tensor_.slice(dim, 0, 1);
0158: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; implements scan/reduction-style dataflow; supports transform-aware functorch semantics. Key symbols: `compute_grad_bias`, `Tensor`, `iota`, `make_dummy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；实现扫描/归约式数据流；支持面向变换的 functorch 语义。关键符号：`compute_grad_bias`, `Tensor`, `iota`, `make_dummy`。

### Lines 159-184 / 第 159-184 行

```cpp
0159:   DimVector expand_shape(tensor_.sizes().begin(), tensor_.sizes().end());
0160:   expand_shape[dim] = batch_size * orig_size;
0161: 
0162:   return tensor_.new_empty({}).expand(expand_shape);
0163: }
0164: 
0165: static std::tuple<Tensor, std::optional<int64_t>>
0166: convolution_backward_input_batch_rule(
0167:     const Tensor& grad_output, std::optional<int64_t> grad_output_bdim,
0168:     const Tensor& input, std::optional<int64_t> input_bdim,
0169:     const Tensor& weight, std::optional<int64_t> weight_bdim,
0170:     c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed,
0171:     c10::SymIntArrayRef output_padding, const c10::SymInt& groups) {
0172:   const std::array<bool, 3> mask = {true, false, false};
0173:   if (grad_output_bdim && weight_bdim) {
0174:     // regular: BNO, BOI -> N(BO), (BO)I -> N(BI)
0175:     // transposed: BNO, BIO -> N(BO), (BI)O -> N(BI)
0176:     const auto batch_size = weight.size(*weight_bdim);
0177:     const auto grad_output_ = reshape_dim_into(*grad_output_bdim, 1, grad_output);
0178:     const auto weight_ = reshape_dim_into(*weight_bdim, 0, weight);
0179:     auto dummy_input = make_dummy(input, input_bdim, 1, batch_size);
0180:     const auto result = at::convolution_backward_symint(
0181:         grad_output_, dummy_input, weight_, std::nullopt, stride, padding,
0182:         dilation, transposed, output_padding, groups * batch_size, mask);
0183:     auto grad_input = reshape_dim_outof(1, batch_size, std::get<0>(result));
0184:     return std::make_tuple(std::move(grad_input), 1);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `expand_shape`, `convolution_backward_input_batch_rule`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`expand_shape`, `convolution_backward_input_batch_rule`, `make_tuple`。

### Lines 185-208 / 第 185-208 行

```cpp
0185:   } else if (grad_output_bdim && !weight_bdim) {
0186:     // BNO, OI -> (BN)O, OI -> (BN)I
0187:     // transposed is the same.
0188:     const auto batch_size = grad_output.size(*grad_output_bdim);
0189:     const auto grad_output_ = reshape_dim_into(*grad_output_bdim, 0, grad_output);
0190:     auto dummy_input = make_dummy(input, input_bdim, 0, batch_size);
0191:     const auto result = at::convolution_backward_symint(
0192:         grad_output_, dummy_input, weight, std::nullopt, stride, padding,
0193:         dilation, transposed, output_padding, groups, mask);
0194:     auto grad_input = reshape_dim_outof(0, batch_size, std::get<0>(result));
0195:     return std::make_tuple(std::move(grad_input), 0);
0196:   } else if (!grad_output_bdim && weight_bdim) {
0197:     const auto batch_size = weight.size(*weight_bdim);
0198:     if (groups == 1) {
0199:       // regular: NO, BOI -> NO, O(BI) -> N(BI)
0200:       // transposed: NO, BIO -> NO, (BI)O -> N(BI)
0201:       const auto in_ch_dim = transposed ? 0 : 1;
0202:       const auto weight_ = reshape_dim_into(*weight_bdim, in_ch_dim, weight);
0203:       auto dummy_input = make_dummy(input, input_bdim, 1, batch_size);
0204:       const auto result = at::convolution_backward_symint(
0205:           grad_output, dummy_input, weight_, std::nullopt, stride, padding,
0206:           dilation, transposed, output_padding, groups, mask);
0207:       auto grad_input = reshape_dim_outof(1, batch_size, std::get<0>(result));
0208:       return std::make_tuple(std::move(grad_input), 1);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 209-234 / 第 209-234 行

```cpp
0209:     }
0210:     Tensor grad_input;
0211:     if (!transposed) {
0212:       // N(GO), B(GO)I -> N(GO), (GO)(BI) -> N(GBI)
0213:       const auto weight_ = reshape_dim_into(*weight_bdim, 1, weight);
0214:       auto dummy_input = make_dummy(input, input_bdim, 1, batch_size);
0215:       grad_input = std::get<0>(at::convolution_backward_symint(
0216:           grad_output, dummy_input, weight_, std::nullopt, stride, padding,
0217:           dilation, transposed, output_padding, groups, mask)); // N(GBI)
0218:     } else {
0219:       // N(GO), B(GI)O -> N(GO), (GBI)O -> N(GBI)
0220:       auto weight_ = moveBatchDimToFront(weight, weight_bdim); // B(GI)O
0221:       weight_ = reshape_dim_outof_symint(1, groups, weight_);         // BGIO
0222:       weight_ = weight_.transpose(0, 1);                       // GBIO
0223:       weight_ = weight_.flatten(0, 2);                         // (GBI)O
0224:       const auto dummy_input = make_dummy(input, input_bdim, 1, batch_size);
0225:       grad_input = std::get<0>(at::convolution_backward_symint(
0226:           grad_output, dummy_input, weight_, std::nullopt, stride, padding,
0227:           dilation, transposed, output_padding, groups, mask)); // N(GBI)
0228:     }
0229:     // N(GBI) -> NG(BI) -> NGBI -> NBGI -> NB(GI)
0230:     grad_input = reshape_dim_outof_symint(1, groups, grad_input);
0231:     grad_input = reshape_dim_outof_symint(2, batch_size, grad_input);
0232:     grad_input = grad_input.transpose(1, 2);
0233:     grad_input = reshape_dim_into(2, 2, grad_input);
0234:     return std::make_tuple(std::move(grad_input), 1);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 235-263 / 第 235-263 行

```cpp
0235:   } else {
0236:     TORCH_INTERNAL_ASSERT(input_bdim);
0237:     const auto dummy_input = make_dummy(input, input_bdim, 0, 1);
0238:     auto result = at::convolution_backward_symint(
0239:         grad_output, dummy_input, weight, std::nullopt, stride, padding,
0240:         dilation, transposed, output_padding, groups, mask);
0241:     return std::make_tuple(std::move(std::get<0>(result)), std::nullopt);
0242:   }
0243: }
0244: static std::tuple<Tensor, std::optional<int64_t>>
0245: convolution_backward_weight_batch_rule(
0246:     const Tensor& grad_output, std::optional<int64_t> grad_output_bdim,
0247:     const Tensor& input, std::optional<int64_t> input_bdim,
0248:     const Tensor& weight, std::optional<int64_t> weight_bdim,
0249:     c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed,
0250:     c10::SymIntArrayRef output_padding, const c10::SymInt& groups) {
0251:   const std::array<bool, 3> mask = {false, true, false};
0252:   if (grad_output_bdim && input_bdim) {
0253:     // BNO, BNI -> N(BO), N(BI) -> (BO)I (regular) (BI)O (transposed)
0254:     const auto batch_size = input.size(*input_bdim);
0255:     const auto grad_output_ = reshape_dim_into(*grad_output_bdim, 1, grad_output);
0256:     const auto input_ = reshape_dim_into(*input_bdim, 1, input);
0257:     const auto dummy_weight = make_dummy(weight, weight_bdim, 0, batch_size);
0258:     auto result = at::convolution_backward_symint(
0259:         grad_output_, input_, dummy_weight, std::nullopt, stride, padding,
0260:         dilation, transposed, output_padding, groups * batch_size, mask);
0261:     auto& grad_weight = std::get<1>(result);
0262:     grad_weight = reshape_dim_outof_symint(0, batch_size, grad_weight);
0263:     return std::make_tuple(std::move(grad_weight), 0);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `make_tuple`, `convolution_backward_weight_batch_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`make_tuple`, `convolution_backward_weight_batch_rule`。

### Lines 264-283 / 第 264-283 行

```cpp
0264:   } else if (grad_output_bdim && !input_bdim) {
0265:     const auto batch_size = grad_output.size(*grad_output_bdim);
0266:     if (groups == 1) {
0267:       // regular: BNO, NI -> N(BO), NI -> (BO)I
0268:       // transposed: BNO, NI -> N(BO), NI -> I(BO)
0269:       const auto grad_output_ = reshape_dim_into(*grad_output_bdim, 1, grad_output);
0270:       const auto out_ch_dim = transposed ? 1 : 0;
0271:       const auto dummy_weight = make_dummy(weight, weight_bdim, out_ch_dim, batch_size);
0272:       auto result = at::convolution_backward_symint(
0273:           grad_output_, input, dummy_weight, std::nullopt, stride, padding,
0274:           dilation, transposed, output_padding, groups, mask);
0275:       auto& grad_weight = std::get<1>(result);
0276:       grad_weight = reshape_dim_outof_symint(out_ch_dim, batch_size, grad_weight);
0277:       return std::make_tuple(std::move(grad_weight), out_ch_dim);
0278:     } else {
0279:       auto grad_output_ = moveBatchDimToFront(grad_output, grad_output_bdim); // BN(GO)
0280:       grad_output_ = reshape_dim_outof_symint(2, groups, grad_output_);              // BNGO
0281:       grad_output_ = grad_output_.movedim(0, 2);                              // NGBO
0282:       grad_output_ = grad_output_.flatten(1, 3);                              // N(GBO)
0283:       if (!transposed) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 284-307 / 第 284-307 行

```cpp
0284:         // BN(GO), N(GI) -> N(GBO), N(GI) -> (GBO)I
0285:         const auto dummy_weight = make_dummy(weight, weight_bdim, 0, batch_size);
0286:         auto result = at::convolution_backward_symint(
0287:             grad_output_, input, dummy_weight, std::nullopt, stride, padding,
0288:             dilation, transposed, output_padding, groups, mask);
0289:         auto& grad_weight = std::get<1>(result);
0290:         grad_weight = grad_weight.unflatten_symint(0, { groups, batch_size, -1 }); // GBOI
0291:         grad_weight = grad_weight.transpose(0, 1);                          // BGOI
0292:         grad_weight = grad_weight.flatten(1, 2);                            // B(GO)I
0293:         return std::make_tuple(std::move(grad_weight), 0);
0294:       } else {
0295:         // BN(GO), N(GI) -> N(GBO), N(GI) -> (GI)(BO)
0296:         const auto dummy_weight = make_dummy(weight, weight_bdim, 1, batch_size);
0297:         auto result = at::convolution_backward_symint(
0298:             grad_output_, input, dummy_weight, std::nullopt, stride, padding,
0299:             dilation, transposed, output_padding, groups, mask);
0300:         auto& grad_weight = std::get<1>(result);
0301:         grad_weight = reshape_dim_outof_symint(1, batch_size, grad_weight);
0302:         return std::make_tuple(std::move(grad_weight), 1);
0303:       }
0304:     }
0305:   } else if (!grad_output_bdim && input_bdim) {
0306:     const auto batch_size = input.size(*input_bdim);
0307:     if (groups == 1) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 308-332 / 第 308-332 行

```cpp
0308:       // regular: NO, BNI -> NO, N(BI) -> O(BI)
0309:       // transposed: NO, BNI -> NO, N(BI) -> (BI)O
0310:       const auto input_ = reshape_dim_into(*input_bdim, 1, input);
0311:       const auto in_ch_dim = transposed ? 0 : 1;
0312:       const auto dummy_weight = make_dummy(weight, weight_bdim, in_ch_dim, batch_size);
0313:       auto result = at::convolution_backward_symint(
0314:           grad_output, input_, dummy_weight, std::nullopt, stride, padding,
0315:           dilation, transposed, output_padding, groups, mask);
0316:       auto& grad_weight = std::get<1>(result);
0317:       grad_weight = reshape_dim_outof_symint(in_ch_dim, batch_size, grad_weight);
0318:       return std::make_tuple(std::move(grad_weight), in_ch_dim);
0319:     } else {
0320:       auto input_ = moveBatchDimToFront(input, input_bdim); // BN(GI)
0321:       input_ = reshape_dim_outof_symint(2, groups, input_);        // BNGI
0322:       input_ = input_.movedim(0, 2);                        // NGBI
0323:       input_ = input_.flatten(1, 3);                        // N(GBI)
0324:       if (!transposed) {
0325:         // regular: N(GO), BN(GI) -> N(GO), N(GBI) -> (GO)(BI)
0326:         const auto dummy_weight = make_dummy(weight, weight_bdim, 1, batch_size);
0327:         auto result = at::convolution_backward_symint(
0328:             grad_output, input_, dummy_weight, std::nullopt, stride, padding,
0329:             dilation, transposed, output_padding, groups, mask);
0330:         auto& grad_weight = std::get<1>(result);
0331:         grad_weight = reshape_dim_outof_symint(1, batch_size, grad_weight);
0332:         return std::make_tuple(std::move(grad_weight), 1);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 333-352 / 第 333-352 行

```cpp
0333:       } else {
0334:         // transposed: N(GO), BN(GI) -> N(GO), N(GBI) -> (GBI)O
0335:         const auto dummy_weight = make_dummy(weight, weight_bdim, 0, batch_size);
0336:         auto result = at::convolution_backward_symint(
0337:             grad_output, input_, dummy_weight, std::nullopt, stride, padding,
0338:             dilation, transposed, output_padding, groups, mask);
0339:         auto& grad_weight = std::get<1>(result);
0340:         grad_weight = grad_weight.unflatten_symint(0, { groups, batch_size, -1 }); // GBIO
0341:         grad_weight = grad_weight.transpose(0, 1);                          // BGIO
0342:         grad_weight = grad_weight.flatten(1, 2);                            // B(GI)O
0343:         return std::make_tuple(std::move(grad_weight), 0);
0344:       }
0345:     }
0346:   } else {
0347:     TORCH_INTERNAL_ASSERT(weight_bdim);
0348:     const auto dummy_weight = make_dummy(weight, weight_bdim, 0, 1);
0349:     auto result = at::convolution_backward_symint(
0350:         grad_output, input, dummy_weight, std::nullopt, stride, padding,
0351:         dilation, transposed, output_padding, groups, mask);
0352:     return std::make_tuple(std::move(std::get<1>(result)), std::nullopt);
```

- **EN:** This block produces a result or forwards a computed value; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `make_tuple`.
- **CN:** 该代码块返回结果或转发已计算的值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`make_tuple`。

### Lines 353-373 / 第 353-373 行

```cpp
0353: 
0354:   }
0355: }
0356: 
0357: static std::tuple<Tensor,Tensor,Tensor> convolution_backward_plumbing(
0358:     const Tensor& grad_output_, const Tensor& input_, const Tensor& weight_,
0359:     const c10::OptionalArrayRef<SymInt> bias_sizes_opt,
0360:     c10::SymIntArrayRef stride, c10::SymIntArrayRef padding, c10::SymIntArrayRef dilation, bool transposed,
0361:     c10::SymIntArrayRef output_padding, c10::SymInt groups, std::array<bool, 3> output_mask) {
0362:   const auto maybe_layer = maybeCurrentDynamicLayer();
0363:   vmap_check_escaped(maybe_layer, "convolution_backward_plumbing");
0364:   // NOLINTNEXTLINE(bugprone-unchecked-optional-access)
0365:   int64_t cur_level = maybe_layer->layerId();
0366: 
0367:   if (!areAnyBatchedAtLevel({grad_output_, input_, weight_}, cur_level)){
0368:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0369:     return at::convolution_backward_symint(
0370:         grad_output_, input_, weight_, bias_sizes_opt, stride, padding,
0371:         dilation, transposed, output_padding, std::move(groups), output_mask);
0372:   }
0373: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `convolution_backward_plumbing`, `vmap_check_escaped`, `guard`, `convolution_backward_symint`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`convolution_backward_plumbing`, `vmap_check_escaped`, `guard`, `convolution_backward_symint`。

### Lines 374-395 / 第 374-395 行

```cpp
0374:   auto [grad_output, grad_output_bdim] = unwrapTensorAtLevel(grad_output_, cur_level);
0375:   auto [input, input_bdim] = unwrapTensorAtLevel(input_, cur_level);
0376:   auto [weight, weight_bdim] = unwrapTensorAtLevel(weight_, cur_level);
0377: 
0378:   auto grad_bias = compute_grad_bias(grad_output_, output_mask);
0379:   output_mask[2] = false;
0380: 
0381:   // TODO: A little bird says that unfold + matmul is actually faster than
0382:   // group convolution in many cases. We should benchmark some of
0383:   // the common cases and replace things with unfold + matmul as necessary.
0384: 
0385:   // Notation:
0386:   // B - a batch dimension
0387:   // G - groups (sometimes omitted because it doesn't matter)
0388:   // NO - grad_output
0389:   // NI - input
0390:   // OI - weight
0391:   // "(BO)I" - we don't actually care about the values of this Tensor,
0392:   //           we just need to create a tensor on the same device with the
0393:   //           correct shape and pray that the implementation is smart enough
0394:   //           to not do anything with it.
0395: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 396-419 / 第 396-419 行

```cpp
0396:   // BNO, BNI, BOI
0397:   // AKA one of the model ensembling case
0398:   if (grad_output_bdim && input_bdim && weight_bdim) {
0399:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0400:     grad_output = reshape_dim_into(*grad_output_bdim, 1, grad_output);
0401: 
0402:     // BNO, BNI, BOI -> N(BO), N(BI), (BO)I
0403:     const auto batch_size = weight.size(*weight_bdim);
0404:     input = reshape_dim_into(*input_bdim, 1, input);
0405:     weight = reshape_dim_into(*weight_bdim, 0, weight);
0406:     const auto result = at::convolution_backward_symint(
0407:         grad_output, input, weight, std::nullopt, stride, padding, dilation,
0408:         transposed, output_padding, batch_size * groups, output_mask);
0409:     // N(BI), (BO)I -> NBI, BOI
0410:     auto grad_input = output_mask[0] ?
0411:       reshape_dim_outof(1, batch_size, std::get<0>(result)) : Tensor();
0412:     auto grad_weight = output_mask[1] ?
0413:       reshape_dim_outof(0, batch_size, std::get<1>(result)) : Tensor();
0414:     return std::make_tuple(
0415:         output_mask[0] ? makeBatched(std::move(grad_input), 1, cur_level) : std::move(grad_input),
0416:         output_mask[1] ? makeBatched(std::move(grad_weight), 0, cur_level) : std::move(grad_weight),
0417:         std::move(grad_bias));
0418:   }
0419: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `guard`, `reshape_dim_outof`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`guard`, `reshape_dim_outof`, `make_tuple`。

### Lines 420-441 / 第 420-441 行

```cpp
0420:   Tensor grad_input;
0421:   if (output_mask[0]) {
0422:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0423:     auto [tensor, bdim] = convolution_backward_input_batch_rule(
0424:         grad_output, grad_output_bdim,
0425:         input, input_bdim,
0426:         weight, weight_bdim,
0427:         stride, padding, dilation, transposed, output_padding, groups);
0428:     grad_input = makeBatched(std::move(tensor), bdim, cur_level);
0429:   }
0430: 
0431:   Tensor grad_weight;
0432:   if (output_mask[1]) {
0433:     c10::impl::ExcludeDispatchKeyGuard guard(DispatchKey::FuncTorchBatched);
0434:     auto [tensor, bdim] = convolution_backward_weight_batch_rule(
0435:         grad_output, grad_output_bdim,
0436:         input, input_bdim,
0437:         weight, weight_bdim,
0438:         stride, padding, dilation, transposed, output_padding, groups);
0439:     grad_weight = makeBatched(std::move(tensor), bdim, cur_level);
0440:   }
0441:   return std::make_tuple(std::move(grad_input), std::move(grad_weight), std::move(grad_bias));
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `guard`, `make_tuple`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`guard`, `make_tuple`。

### Lines 442-461 / 第 442-461 行

```cpp
0442: 
0443:   // Someone's definitely going to find a problem with this batching rule so
0444:   // I'm leaving the following fallback if we need it back.
0445:   // static auto op = c10::Dispatcher::singleton()
0446:   //   .findSchemaOrThrow("aten::convolution_backward", "");
0447:   // auto result = slow_fallback<Tensor,Tensor,Tensor>(op, {
0448:   //   grad_output_, input_, weight_, bias_sizes_opt,
0449:   //   stride, padding, dilation, transposed, output_padding, groups, output_mask
0450:   // });
0451:   // return std::make_tuple(grad_input, std::get<1>(result), grad_bias);
0452: }
0453: 
0454: 
0455: TORCH_LIBRARY_IMPL(aten, FuncTorchBatched, m) {
0456:   VMAP_SUPPORT(convolution, convolution_batch_rule);
0457:   m.impl("_convolution", _convolution_decomp);
0458:   m.impl("convolution_backward", convolution_backward_plumbing);
0459: }
0460: 
0461: } // namespace at;:functorch
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `BatchRulesConvolution` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `BatchRulesConvolution` 的行为。符号：无明显局部符号。


## Key Concepts / 关键概念
- **functorch-aware tensor transforms** — 面向 functorch 的张量变换
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **Vendor library descriptor management** — 厂商库描述符管理
- **Linear algebra backend integration** — 线性代数后端集成
- **Device and stream coordination** — 设备与流协调
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: convolution_batch_rule, lhs_spec, iota, make_tuple, _convolution_decomp, convolution, compute_grad_bias, Tensor** — 核心符号：convolution_batch_rule、lhs_spec、iota、make_tuple、_convolution_decomp、convolution、compute_grad_bias、Tensor

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/functorch/BatchRulesHelper.h`, `ATen/functorch/PlumbingHelper.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at::functorch`, `at;:functorch`
- **Representative symbols / 代表性符号**: `convolution_batch_rule`, `lhs_spec`, `iota`, `make_tuple`, `_convolution_decomp`, `convolution`, `compute_grad_bias`, `Tensor`, `make_dummy`, `expand_shape`, `convolution_backward_input_batch_rule`, `convolution_backward_weight_batch_rule`, `...`
