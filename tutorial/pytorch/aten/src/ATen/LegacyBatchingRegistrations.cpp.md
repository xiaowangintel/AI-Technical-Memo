# LegacyBatchingRegistrations.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/LegacyBatchingRegistrations.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines shared ATen abstractions, tensor utilities, device helpers, or interop glue used across subsystems. This specific file centers on `LegacyBatchingRegistrations.cpp`. Descriptor/handle lifecycle management is important here. Transform-aware tensor semantics used by functorch are part of the file scope. Tensor metadata shaping and storage/layout checks are part of the local logic.
- **Purpose (CN)**: 定义跨子系统复用的共享 ATen 抽象、张量工具、设备辅助逻辑或互操作胶水代码。 该文件具体围绕 `LegacyBatchingRegistrations.cpp` 展开。 描述符/句柄的生命周期管理是这里的重要内容。 该文件范围内还包含 functorch 所需的变换感知张量语义。 张量元数据组织以及存储/布局检查也是局部逻辑的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47 / 第 1-47 行

```cpp
0001: #include <torch/library.h>
0002: #include <ATen/LegacyVmapTransforms.h>
0003: #include <ATen/LegacyBatchedFallback.h>
0004: #include <ATen/RedispatchFunctions.h>
0005: #include <ATen/native/ResizeCommon.h>
0006: #include <c10/util/irange.h>
0007: 
0008: #include <utility>
0009: 
0010: namespace at {
0011: 
0012: // NOTE: [What is a batching rule?]
0013: //
0014: // A *batching rule* implements the logic of how to call an operator on inputs
0015: // that have zero or more additional batch dimensions. When one does a vmap, the
0016: // dimension(s) being vmap'ed over get recorded as batch dimensions.
0017: //
0018: // For example, vmap(torch.add)(x, y)
0019: // 1. wraps `x` into batched_x = BatchedTensor(x, bdims=[(lvl=1, dim=0)];
0020: // 2. wraps `y` into batched_y = BatchedTensor(y, bdims=[(lvl=1, dim=0)];
0021: // 3. and then runs `torch.add(batched_x, batched_y)`.
0022: 
0023: // NOTE: [When should I add a batching rule?]
0024: // When you are adding a new operator, you'll need to add a batching rule so
0025: // that vmap can work efficiently with said operator. If you do not, we'll attempt
0026: // to generate a slow fallback for the batching rule.
0027: 
0028: // NOTE: [How to write batching rules?]
0029: // The signature of a batching rule should look like exactly like the C++ signature
0030: // of its operator.
0031: //
0032: // First, see NOTE: [Logical vs physical args] in VmapTransforms.h for terminology.
0033: //
0034: // At a high level, what a batching rule does is the following:
0035: // 1. Converts (logical) BatchedTensors to views on physical tensors.
0036: // 2. Converts logical arguments (e.g. dimension indexes, shapes) to physical
0037: //    arguments that correspond to the physical tensors.
0038: // 3. Calls at:: operations on the physical tensors and arguments to produce
0039: //    some physical results.
0040: // 4. Converts physical results back to BatchedTensors.
0041: //
0042: // Steps 1, 2, and 4 differ for operators with different batching behaviors. When
0043: // writing a new batching rule, please select a VmapTransform that matches the
0044: // batching behavior of your operation. The VmapTransform provides helper functions
0045: // to do steps (1), (2), and (4).
0046: // (see NOTE: [What is an VmapTransform?] in VmapTransforms.h)
0047: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 48-78 / 第 48-78 行

```cpp
0048: // Note: [Future plans]
0049: // The API for writing a batching rule isn't stable. In the future, we'd like
0050: // to think about the problem of translating these batching rules to TorchScript.
0051: // Ideally batching rules in eager mode vs TorchScript would look pretty similar,
0052: // if not use the same mechanism. In order to accomplish that we might have to
0053: // do some refactoring.
0054: 
0055: namespace{
0056: 
0057: // PyTorch allows operations to specify dim 0 and dim -1 on a scalar tensor.
0058: bool is_allowed_dim_on_scalar_tensor(int64_t dim) {
0059:   return dim == 0 || dim == -1;
0060: }
0061: 
0062: Tensor sum_batching_rule(const Tensor& self, OptionalIntArrayRef opt_dims, bool keepdim, std::optional<ScalarType> dtype) {
0063:   if (opt_dims.has_value()) {
0064:     auto dims = opt_dims.value();
0065:     // PyTorch has a special case where sum(scalar_tensor, dim=0) does not fail
0066:     // and instead returns a new scalar tensor (this also happens for dim=-1)
0067:     // If the following happens:
0068:     // >>> x = torch.randn(B0)  # the per-examples are all scalars
0069:     // >>> vmap(partial(torch.sum, dim=0), x)
0070:     // then we replicate the behavior of sum(scalar_tensor, dim=0).
0071:     if (/*logical*/self.dim() == 0 && (dims.empty() || (dims.size() == 1 && is_allowed_dim_on_scalar_tensor(dims[0])))) {
0072:       return self.clone();
0073:     }
0074:   }
0075:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0076:   auto dims_physical = self_physical.getPhysicalDims(opt_dims);
0077:   auto result = at::sum(self_physical.tensor(), dims_physical, keepdim, dtype);
0078:   return self_physical.getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `is_allowed_dim_on_scalar_tensor`, `sum_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`is_allowed_dim_on_scalar_tensor`, `sum_batching_rule`。

### Lines 79-108 / 第 79-108 行

```cpp
0079: }
0080: 
0081: bool isPhysicalScalarTensor(const Tensor& logical_tensor) {
0082:   if (logical_tensor.dim() > 0) {
0083:     return false;
0084:   }
0085:   auto* batched = maybeGetBatchedImpl(logical_tensor);
0086:   if (batched) {
0087:     return false;
0088:   }
0089:   return true;
0090: }
0091: 
0092: template <typename F, F Func, typename... ExtraArgs>
0093: Tensor binary_pointwise_batching_rule(
0094:     const Tensor& self, const Tensor& other, ExtraArgs... args) {
0095:   if (self.dim() > 0 && other.dim() > 0) {
0096:     auto physical_args = BroadcastingVmapTransform::logicalToPhysical({self, other});
0097:     auto result = Func(physical_args[0].tensor(), physical_args[1].tensor(), args...);
0098:     return physical_args[0].getPhysicalToLogicalMap().apply(result);
0099:   }
0100:   if (isPhysicalScalarTensor(self)) {
0101:     auto other_physical = MultiBatchVmapTransform::logicalToPhysical(other);
0102:     auto result = Func(self, other_physical.tensor(), args...);
0103:     return other_physical.getPhysicalToLogicalMap().apply(result);
0104:   }
0105:   if (isPhysicalScalarTensor(other)) {
0106:     auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0107:     auto result = Func(self_physical.tensor(), other, args...);
0108:     return self_physical.getPhysicalToLogicalMap().apply(result);
```

- **EN:** Builds a reusable template/helper layer around `LegacyBatchingRegistrations`. Key symbols: `isPhysicalScalarTensor`, `binary_pointwise_batching_rule`.
- **CN:** 围绕 `LegacyBatchingRegistrations` 构建可复用的模板或辅助层。关键符号：`isPhysicalScalarTensor`, `binary_pointwise_batching_rule`。

### Lines 109-140 / 第 109-140 行

```cpp
0109:   }
0110: 
0111:   // At this point, we know at least one of the operands is a logical Scalar tensor.
0112:   // Here we must emulate TensorIterator's special behavior on Scalars.
0113:   //
0114:   // As a motivating example, consider the following:
0115:   //   x = torch.randn(3, 10)
0116:   //   y = torch.randn(3, dtype=torch.double)
0117:   //   vmap(torch.mul)(torch.randn(3, 10), torch.randn(3, dtype=torch.double))
0118:   //
0119:   // At a per-example level, we are adding FloatTensor[10] and DoubleTensor[];
0120:   // Type Promotion dictates that the result should be FloatTensor[10].
0121:   // This means we cannot directly pass the physical tensors (x and y) to
0122:   // TensorIterator (if we did, it would promote them to DoubleTensor).
0123:   //
0124:   // FIXME(rzou): I didn't want to go down the slippery slope of emulating
0125:   // everything TensorIterator does (it would be better to refactor out the
0126:   // TensorIterator logic). The one thing that this code doesn't handle
0127:   // is cross-device logical scalar tensors.
0128:   //   cpu_tensor = torch.randn(3)
0129:   //   cuda_tensor = torch.randn(3, 10, device='cuda')
0130:   //   vmap(torch.mul)(cpu_tensor, cuda_tensor)
0131:   //
0132:   // At a per-example level, we are adding CPUTensor[] and CUDATensor[10].
0133:   // TensorIterator allows for this cross-device operation because one of the
0134:   // tensors is a Scalar CPU tensor. However, the following code will throw an
0135:   // error in that case. I don't expect to see many use cases for this, so
0136:   // this is probably fine as-is.
0137:   auto logical_self = self;
0138:   auto logical_other = other;
0139:   auto result_type = at::native::result_type(logical_self, logical_other);
0140:   if (logical_self.scalar_type() != result_type) {
```

- **EN:** This block handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 141-187 / 第 141-187 行

```cpp
0141:     logical_self = logical_self.to(result_type);
0142:   }
0143:   if (logical_other.scalar_type() != result_type) {
0144:     logical_other = logical_other.to(result_type);
0145:   }
0146:   auto physical_args = BroadcastingVmapTransform::logicalToPhysical(
0147:       {std::move(logical_self), std::move(logical_other)});
0148:   auto result = Func(physical_args[0].tensor(), physical_args[1].tensor(), args...);
0149:   return physical_args[0].getPhysicalToLogicalMap().apply(result);
0150: }
0151: 
0152: Tensor expand_batching_rule(const Tensor& self, IntArrayRef size, bool implicit) {
0153:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0154:   auto size_physical = self_physical.getPhysicalShape(size);
0155:   auto self_physical_dim = self_physical.tensor().dim();
0156: 
0157:   TORCH_CHECK(self_physical_dim <= static_cast<int64_t>(size_physical.size()),
0158:        "expand: the number of sizes provided (", /*logical*/size.size(), ") ",
0159:        "must be greater or equal to the number of dimensions in the tensor (",
0160:        /*logical dim*/self.dim(), ")");
0161: 
0162:   if (self_physical_dim == static_cast<int64_t>(size_physical.size())) {
0163:     auto result = self_physical.tensor().expand(size_physical, implicit);
0164:     return self_physical.getPhysicalToLogicalMap().apply(result);
0165:   }
0166: 
0167:   TORCH_INTERNAL_ASSERT(self_physical_dim < static_cast<int64_t>(size_physical.size()));
0168:   // Here, we know we are expanding a (logical) tensor to a larger number
0169:   // of dimensions. We have to be careful because we can't call expand directly
0170:   // due to the presence of batch dimensions.
0171:   //
0172:   // As an example, let B0 be a batch dimension and consider expand(Tensor[B0, 3], [2, 3]).
0173:   // The result should be a tensor of size [B0, 2, 3].
0174:   // A physical view of size [B0, 3] can't directly be expanded to size [B0, 2, 3]
0175:   // so the strategy here is to view it first as a tensor of size [B0, 1, 3] and
0176:   // then expand.
0177:   auto self_physical_size = self_physical.tensor().sizes();
0178:   auto extra_dims = size_physical.size() - self_physical_dim;
0179:   VmapDimVector view_shape(size_physical.size(), 1);
0180:   std::copy(self_physical_size.begin(),
0181:             self_physical_size.begin() + self_physical.numBatchDims(),
0182:             view_shape.begin());
0183:   std::copy(self_physical_size.begin() + self_physical.numBatchDims(),
0184:             self_physical_size.end(),
0185:             view_shape.begin() + self_physical.numBatchDims() + extra_dims);
0186:   auto result = self_physical.tensor().view(view_shape).expand(size_physical, implicit);
0187:   return self_physical.getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `expand_batching_rule`, `view_shape`, `copy`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`expand_batching_rule`, `view_shape`, `copy`。

### Lines 188-221 / 第 188-221 行

```cpp
0188: }
0189: 
0190: std::vector<Tensor> chunk_batching_rule(const Tensor& self, int64_t chunks, int64_t dim) {
0191:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0192:   auto dim_physical = self_physical.getPhysicalDim(dim);
0193:   auto result = at::chunk(self_physical.tensor(), chunks, dim_physical);
0194:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0195:   return result;
0196: }
0197: 
0198: Tensor clamp_batching_rule(const Tensor& self, const std::optional<Scalar>& min, const std::optional<Scalar>& max) {
0199:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0200:   auto result = at::clamp(self_physical.tensor(), min, max);
0201:   return self_physical.getPhysicalToLogicalMap().apply(result);
0202: }
0203: 
0204: Tensor clamp_min_batching_rule(const Tensor& self, const Scalar& min) {
0205:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0206:   auto result = at::clamp_min(self_physical.tensor(), min);
0207:   return self_physical.getPhysicalToLogicalMap().apply(result);
0208: }
0209: 
0210: Tensor clamp_max_batching_rule(const Tensor& self, const Scalar& max) {
0211:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0212:   auto result = at::clamp_max(self_physical.tensor(), max);
0213:   return self_physical.getPhysicalToLogicalMap().apply(result);
0214: }
0215: 
0216: std::vector<Tensor> tensor_split_sections_batching_rule(const Tensor& self, int64_t sections, int64_t dim) {
0217:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0218:   auto dim_physical = self_physical.getPhysicalDim(dim);
0219:   auto result = at::tensor_split(self_physical.tensor(), sections, dim_physical);
0220:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0221:   return result;
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `chunk_batching_rule`, `clamp_batching_rule`, `clamp_min_batching_rule`, `clamp_max_batching_rule`, `tensor_split_sections_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`chunk_batching_rule`, `clamp_batching_rule`, `clamp_min_batching_rule`, `clamp_max_batching_rule`, `tensor_split_sections_batching_rule`。

### Lines 222-252 / 第 222-252 行

```cpp
0222: }
0223: 
0224: std::vector<Tensor> tensor_split_indices_batching_rule(const Tensor& self, IntArrayRef indices, int64_t dim) {
0225:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0226:   auto dim_physical = self_physical.getPhysicalDim(dim);
0227:   auto result = at::tensor_split(self_physical.tensor(), indices, dim_physical);
0228:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0229:   return result;
0230: }
0231: 
0232: Tensor unsqueeze_batching_rule(const Tensor& self, int64_t dim) {
0233:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0234:   // NB: unsqueeze has some special handling of its `dim` argument so we can't call
0235:   // self_physical.getPhysicalDim directly. In particular, native::unsqueeze
0236:   // wraps the dim to (the logical dimension) + 1, so we need to do that here too.
0237:   // https://github.com/pytorch/pytorch/blob/b623bdeabb0aa8da44285d303246e7f8ac06c2a9/aten/src/ATen/native/TensorShape.cpp#L1413
0238:   auto dim_physical =
0239:       self_physical.numBatchDims() + maybe_wrap_dim(dim, /*logical_dim*/self.dim() + 1);
0240:   auto result = self_physical.tensor().unsqueeze(dim_physical);
0241:   return self_physical.getPhysicalToLogicalMap().apply(result);
0242: }
0243: 
0244: Tensor& fill_inplace_scalar_batching_rule(Tensor& self, const Scalar& value) {
0245:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0246:   self_physical.tensor().fill_(value);
0247:   return self;
0248: }
0249: 
0250: Tensor& fill_inplace_tensor_batching_rule(Tensor& self, const Tensor& value) {
0251:   auto value_batched = isBatchedTensor(value);
0252: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `tensor_split_indices_batching_rule`, `unsqueeze_batching_rule`, `fill_inplace_scalar_batching_rule`, `fill_inplace_tensor_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`tensor_split_indices_batching_rule`, `unsqueeze_batching_rule`, `fill_inplace_scalar_batching_rule`, `fill_inplace_tensor_batching_rule`。

### Lines 253-282 / 第 253-282 行

```cpp
0253:   if (value_batched) {
0254:     auto physical_args =
0255:       BroadcastingVmapTransform::logicalToPhysical({self, value});
0256:     physical_args[0].tensor().copy_(physical_args[1].tensor());
0257:   } else {
0258:     auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0259:     self_physical.tensor().fill_(value);
0260:   }
0261:   return self;
0262: }
0263: 
0264: Tensor& zero_inplace_batching_rule(Tensor &self) {
0265:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0266:   self_physical.tensor().zero_();
0267:   return self;
0268: }
0269: 
0270: Tensor squeeze_batching_rule(const Tensor& self) {
0271:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0272:   auto physical_sizes = self_physical.tensor().sizes();
0273: 
0274:   // Don't squeeze the batch dims!
0275:   VmapDimVector squeezed_sizes;
0276:   int64_t num_batch_dims = self_physical.numBatchDims();
0277:   squeezed_sizes.insert(
0278:       squeezed_sizes.end(),
0279:       physical_sizes.begin(),
0280:       physical_sizes.begin() + num_batch_dims);
0281:   for (auto it = physical_sizes.begin() + num_batch_dims; it != physical_sizes.end(); ++it) {
0282:     if (*it != 1) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `zero_inplace_batching_rule`, `squeeze_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`zero_inplace_batching_rule`, `squeeze_batching_rule`。

### Lines 283-312 / 第 283-312 行

```cpp
0283:       squeezed_sizes.push_back(*it);
0284:     }
0285:   }
0286: 
0287:   auto result = self_physical.tensor().view(squeezed_sizes);
0288:   return self_physical.getPhysicalToLogicalMap().apply(result);
0289: }
0290: 
0291: Tensor squeeze_dim_batching_rule(const Tensor& self, int64_t dim) {
0292:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0293:   auto dim_physical = self_physical.getPhysicalDim(dim);
0294:   auto result = self_physical.tensor().squeeze(dim_physical);
0295:   return self_physical.getPhysicalToLogicalMap().apply(result);
0296: }
0297: 
0298: Tensor squeeze_dims_batching_rule(const Tensor& self, IntArrayRef dims) {
0299:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0300:   auto dims_physical = self_physical.getPhysicalDims(dims);
0301:   auto result = self_physical.tensor().squeeze(dims_physical);
0302:   return self_physical.getPhysicalToLogicalMap().apply(result);
0303: }
0304: 
0305: Tensor trace_batching_rule(const Tensor& self) {
0306:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0307:   // Batched Diagonal View
0308:   auto self_diag = at::diagonal(self_physical.tensor(), /*offset*/0, /*dim1*/-2, /*dim2*/-1);
0309:   auto result =  at::sum(self_diag, -1);
0310:   return self_physical.getPhysicalToLogicalMap().apply(result);
0311: }
0312: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `squeeze_dim_batching_rule`, `squeeze_dims_batching_rule`, `trace_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`squeeze_dim_batching_rule`, `squeeze_dims_batching_rule`, `trace_batching_rule`。

### Lines 313-344 / 第 313-344 行

```cpp
0313: Tensor trace_backward_batching_rule(const Tensor& grad, IntArrayRef input_sizes) {
0314:   auto grad_physical = MultiBatchVmapTransform::logicalToPhysical(grad);
0315:   auto grad_input = at::zeros(grad_physical.getPhysicalShape(input_sizes), grad.options());
0316:   // Batched Diagonal View
0317:   auto grad_input_diag = at::diagonal(grad_input, /*offset*/0, /*dim1*/-2, /*dim2*/-1);
0318:   // Append a dimension of size one to the grad output
0319:   auto grad_physical_tensor = grad_physical.tensor().unsqueeze(-1);
0320:   grad_input_diag.copy_(grad_physical_tensor);
0321:   return grad_physical.getPhysicalToLogicalMap().apply(grad_input);
0322: }
0323: 
0324: Tensor transpose_int_batching_rule(const Tensor& self, int64_t dim0, int64_t dim1) {
0325:   // PyTorch has a special case where scalar_tensor.transpose(dim0, dim1) works
0326:   // for dim0, dim1 in {0, -1} and returns the scalar tensor. If the following happens:
0327:   // >>> x = torch.randn(B0)  # the per-examples are all scalars
0328:   // >>> vmap(lambda x: x.transpose(0, -1), x)
0329:   // then we replicate this behavior.
0330:   if (/*logical*/self.dim() == 0 && is_allowed_dim_on_scalar_tensor(dim0) &&
0331:       is_allowed_dim_on_scalar_tensor(dim1)) {
0332:     return self;
0333:   }
0334:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0335:   auto dim0_physical = self_physical.getPhysicalDim(dim0);
0336:   auto dim1_physical = self_physical.getPhysicalDim(dim1);
0337:   auto result = self_physical.tensor().transpose(dim0_physical, dim1_physical);
0338:   return self_physical.getPhysicalToLogicalMap().apply(result);
0339: }
0340: 
0341: Tensor permute_batching_rule(const Tensor& self, IntArrayRef dims) {
0342:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0343:   auto dims_physical = self_physical.getPhysicalDims(dims);
0344: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `trace_backward_batching_rule`, `transpose_int_batching_rule`, `permute_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`trace_backward_batching_rule`, `transpose_int_batching_rule`, `permute_batching_rule`。

### Lines 345-374 / 第 345-374 行

```cpp
0345:   VmapDimVector all_dims_physical;
0346:   all_dims_physical.reserve(self_physical.tensor().dim());
0347:   for (const auto bdim : c10::irange(self_physical.numBatchDims())) {
0348:     all_dims_physical.push_back(bdim);
0349:   }
0350:   all_dims_physical.insert(
0351:       all_dims_physical.end(),
0352:       dims_physical.begin(),
0353:       dims_physical.end());
0354:   auto result = self_physical.tensor().permute(all_dims_physical);
0355:   return self_physical.getPhysicalToLogicalMap().apply(result);
0356: }
0357: 
0358: Tensor select_batching_rule(const Tensor& self, int64_t dim, int64_t index) {
0359:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0360:   auto dim_physical = self_physical.getPhysicalDim(dim);
0361:   auto result = self_physical.tensor().select(dim_physical, index);
0362:   return self_physical.getPhysicalToLogicalMap().apply(result);
0363: }
0364: 
0365: int64_t getGradInputPhysicalDim(int64_t dim, IntArrayRef input_sizes, int64_t num_batch_dims) {
0366:   return maybe_wrap_dim(dim, static_cast<int64_t>(input_sizes.size())) + num_batch_dims;
0367: }
0368: 
0369: Tensor select_backward_batching_rule(const Tensor& grad, IntArrayRef input_sizes, int64_t dim, int64_t index) {
0370:   auto grad_physical = MultiBatchVmapTransform::logicalToPhysical(grad);
0371:   auto grad_input = at::zeros(grad_physical.getPhysicalShape(input_sizes), grad.options());
0372:   auto physical_dim = getGradInputPhysicalDim(dim, input_sizes, grad_physical.numBatchDims());
0373:   grad_input.select(physical_dim, index).copy_(grad_physical.tensor());
0374:   return grad_physical.getPhysicalToLogicalMap().apply(grad_input);
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `select_batching_rule`, `getGradInputPhysicalDim`, `select_backward_batching_rule`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`select_batching_rule`, `getGradInputPhysicalDim`, `select_backward_batching_rule`。

### Lines 375-404 / 第 375-404 行

```cpp
0375: }
0376: 
0377: Tensor slice_batching_rule(
0378:     const Tensor& self,
0379:     int64_t dim,
0380:     std::optional<int64_t> start,
0381:     std::optional<int64_t> end,
0382:     int64_t step) {
0383:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0384:   auto dim_physical = self_physical.getPhysicalDim(dim);
0385:   auto result = self_physical.tensor().slice(dim_physical, start, end, step);
0386:   return self_physical.getPhysicalToLogicalMap().apply(result);
0387: }
0388: 
0389: Tensor slice_backward_batching_rule(const Tensor& grad, IntArrayRef input_sizes, int64_t dim, int64_t start, int64_t end, int64_t step) {
0390:   auto grad_physical = MultiBatchVmapTransform::logicalToPhysical(grad);
0391:   auto grad_input = at::zeros(grad_physical.getPhysicalShape(input_sizes), grad.options());
0392:   auto physical_dim = getGradInputPhysicalDim(dim, input_sizes, grad_physical.numBatchDims());
0393:   grad_input.slice(physical_dim, start, end, step).copy_(grad_physical.tensor());
0394:   return grad_physical.getPhysicalToLogicalMap().apply(grad_input);
0395: }
0396: 
0397: Tensor diagonal_batching_rule(const Tensor& self, int64_t offset, int64_t dim1, int64_t dim2) {
0398:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0399:   auto dim1_physical = self_physical.getPhysicalDim(dim1);
0400:   auto dim2_physical = self_physical.getPhysicalDim(dim2);
0401:   auto result = at::diagonal(self_physical.tensor(), offset, dim1_physical, dim2_physical);
0402:   return self_physical.getPhysicalToLogicalMap().apply(result);
0403: }
0404: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `slice_batching_rule`, `slice_backward_batching_rule`, `diagonal_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`slice_batching_rule`, `slice_backward_batching_rule`, `diagonal_batching_rule`。

### Lines 405-434 / 第 405-434 行

```cpp
0405: Tensor diagonal_backward_batching_rule(const Tensor& grad, IntArrayRef input_sizes, int64_t offset, int64_t dim1, int64_t dim2) {
0406:   auto grad_physical = MultiBatchVmapTransform::logicalToPhysical(grad);
0407:   auto grad_input = at::zeros(grad_physical.getPhysicalShape(input_sizes), grad.options());
0408:   auto dim1_physical = getGradInputPhysicalDim(dim1, input_sizes, grad_physical.numBatchDims());
0409:   auto dim2_physical = getGradInputPhysicalDim(dim2, input_sizes, grad_physical.numBatchDims());
0410:   grad_input.diagonal(offset, dim1_physical, dim2_physical).copy_(grad_physical.tensor());
0411:   return grad_physical.getPhysicalToLogicalMap().apply(grad_input);
0412: }
0413: 
0414: Tensor movedim_batching_rule(const Tensor& self, IntArrayRef source, IntArrayRef destination) {
0415:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0416:   auto source_physical = self_physical.getPhysicalDims(source);
0417:   auto destination_physical = self_physical.getPhysicalDims(destination);
0418:   auto result = at::movedim(self_physical.tensor(), source_physical, destination_physical);
0419:   return self_physical.getPhysicalToLogicalMap().apply(result);
0420: }
0421: 
0422: Tensor reshape_batching_rule(const Tensor& self, IntArrayRef shape) {
0423:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0424:   auto shape_physical = self_physical.getPhysicalShape(shape);
0425:   auto result = self_physical.tensor().reshape(shape_physical);
0426:   return self_physical.getPhysicalToLogicalMap().apply(result);
0427: }
0428: 
0429: std::vector<Tensor> split_batching_rule(const Tensor& self, int64_t split_size, int64_t dim) {
0430:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0431:   auto dim_physical = self_physical.getPhysicalDim(dim);
0432:   auto result = at::split(self_physical.tensor(), split_size, dim_physical);
0433:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0434:   return result;
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `diagonal_backward_batching_rule`, `movedim_batching_rule`, `reshape_batching_rule`, `split_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`diagonal_backward_batching_rule`, `movedim_batching_rule`, `reshape_batching_rule`, `split_batching_rule`。

### Lines 435-466 / 第 435-466 行

```cpp
0435: }
0436: 
0437: std::vector<Tensor> split_with_sizes_batching_rule(const Tensor& self, IntArrayRef split_sizes, int64_t dim) {
0438:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0439:   auto dim_physical = self_physical.getPhysicalDim(dim);
0440:   auto result = at::split_with_sizes(self_physical.tensor(), split_sizes, dim_physical);
0441:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0442:   return result;
0443: }
0444: 
0445: std::vector<Tensor> unbind_batching_rule(const Tensor& self, int64_t dim) {
0446:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0447:   auto dim_physical = self_physical.getPhysicalDim(dim);
0448:   auto result = at::unbind(self_physical.tensor(), dim_physical);
0449:   self_physical.getPhysicalToLogicalMap().applyInplace(result);
0450:   return result;
0451: }
0452: 
0453: Tensor unfold_batching_rule(const Tensor& self, int64_t dim, int64_t size, int64_t step) {
0454:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0455:   auto dim_physical = self_physical.getPhysicalDim(dim);
0456:   auto result = self_physical.tensor().unfold(dim_physical, size, step);
0457:   return self_physical.getPhysicalToLogicalMap().apply(result);
0458: }
0459: 
0460: Tensor contiguous_batching_rule(const Tensor& self, MemoryFormat memory_format) {
0461:   TORCH_CHECK(memory_format == MemoryFormat::Contiguous,
0462:       "NYI: Tensor.contiguous(...) inside of vmap for memory_format other ",
0463:       "than torch.contiguous_format");
0464:   auto physical_view = MultiBatchVmapTransform::logicalToPhysical(self);
0465:   auto result = physical_view.tensor().contiguous(memory_format);
0466:   return physical_view.getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `split_with_sizes_batching_rule`, `unbind_batching_rule`, `unfold_batching_rule`, `contiguous_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`split_with_sizes_batching_rule`, `unbind_batching_rule`, `unfold_batching_rule`, `contiguous_batching_rule`。

### Lines 467-497 / 第 467-497 行

```cpp
0467: }
0468: 
0469: Tensor view_batching_rule(const Tensor& self, IntArrayRef size) {
0470:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0471:   auto size_physical = self_physical.getPhysicalShape(size);
0472:   auto result = self_physical.tensor().view(size_physical);
0473:   return self_physical.getPhysicalToLogicalMap().apply(result);
0474: }
0475: 
0476: Tensor view_as_complex_batching_rule(const Tensor& self) {
0477:   // guard against the user passing in a batch of scalar tensors with batch
0478:   // size equal to 2.
0479:   TORCH_CHECK(!self.sizes().empty(), "Input tensor must have one or more dimensions");
0480:   auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0481:   auto result = at::view_as_complex(self_physical.tensor());
0482:   return self_physical.getPhysicalToLogicalMap().apply(result);
0483: }
0484: 
0485: // Checks that the smallest batch stride is greater than the largest example
0486: // stride. This is something we can support but we choose not to because it's
0487: // potentially error prone.
0488: void checkBatchDimsAtFrontInLayout(IntArrayRef physical_strides, int64_t num_batch_dims) {
0489:   auto smallest_batch_stride = std::min_element(
0490:       physical_strides.begin(), physical_strides.begin() + num_batch_dims);
0491:   auto largest_example_stride = std::max_element(
0492:       physical_strides.begin() + num_batch_dims, physical_strides.end());
0493:   if (largest_example_stride == physical_strides.end()) {
0494:     // No example dimensions
0495:     return;
0496:   }
0497:   TORCH_CHECK(*smallest_batch_stride >= *largest_example_stride,
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `view_batching_rule`, `view_as_complex_batching_rule`, `checkBatchDimsAtFrontInLayout`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`view_batching_rule`, `view_as_complex_batching_rule`, `checkBatchDimsAtFrontInLayout`。

### Lines 498-530 / 第 498-530 行

```cpp
0498:     "vmap: Calling Tensor.as_strided is not supported unless the batch dims being ",
0499:     "vmapped over are at the front of the tensor (in memory layout). When they are ",
0500:     "not at the front of the tensor this operation can be error prone so we "
0501:     "actively discourage it; please file us a bug report and/or try to ",
0502:     "express the as_strided operation in terms of PyTorch view operations");
0503: }
0504: 
0505: // given (sizes, strides, storage_offset) returns the maximum location that
0506: // can be indexed (or nullopt if such a location doesn't exist, e.g., tensors
0507: // with zero-size dims).
0508: std::optional<int64_t> maximum_indexable_location(
0509:     IntArrayRef sizes, IntArrayRef strides, int64_t storage_offset) {
0510:   auto result = native::storage_size_for(sizes, strides);
0511:   if (result == 0) {
0512:     return std::nullopt;
0513:   }
0514:   return result + storage_offset;
0515: }
0516: 
0517: // Let x be the "first slice" of physical_tensor.
0518: // This checks that the range of possible memory locations accessible by
0519: // x.as_strided(sizes, strides, maybe_storage_offset)
0520: // are within the bounds of possible memory locations accessible by x.
0521: void checkBasicAsStridedValidForSlice(
0522:     const Tensor& physical_tensor,
0523:     int64_t num_batch_dims,
0524:     IntArrayRef sizes,
0525:     IntArrayRef strides,
0526:     std::optional<int64_t> maybe_storage_offset) {
0527:   auto slice_sizes = physical_tensor.sizes().slice(num_batch_dims);
0528:   auto slice_strides = physical_tensor.strides().slice(num_batch_dims);
0529:   auto base_offset = physical_tensor.storage_offset();
0530: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `maximum_indexable_location`, `checkBasicAsStridedValidForSlice`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`maximum_indexable_location`, `checkBasicAsStridedValidForSlice`。

### Lines 531-561 / 第 531-561 行

```cpp
0531:   auto storage_offset = maybe_storage_offset.value_or(base_offset);
0532: 
0533:   auto max_as_strided_loc = maximum_indexable_location(sizes, strides, storage_offset);
0534:   auto max_slice_loc = maximum_indexable_location(slice_sizes, slice_strides, base_offset);
0535: 
0536:   if (!max_as_strided_loc.has_value()) {
0537:     return;
0538:   }
0539:   if (!max_slice_loc.has_value()) {
0540:     TORCH_CHECK(false,
0541:         "result = tensor.as_strided(", sizes, ",",  strides, ",", storage_offset, ")",
0542:         "can access memory outside of `tensor`. `tensor` has no storage but the ",
0543:         "passed-in (size, stride, storage_offset) imply a result with some storage. ",
0544:         "This is not supported inside of vmap, please try to rewrite the ",
0545:         "`as_strided` call as a sequence of PyTorch view operations");
0546:   }
0547: 
0548:   TORCH_CHECK(
0549:       *max_as_strided_loc <= *max_slice_loc && base_offset <= storage_offset,
0550:       "result = tensor.as_strided(", sizes, ",",  strides, ",", storage_offset, ")",
0551:       "can access memory outside of `tensor`. `result` can access some",
0552:       "memory in range [", storage_offset, ", ", *max_as_strided_loc, "], but ",
0553:       "`tensor` can only access some memory in range [", base_offset, ", ",
0554:       *max_slice_loc, "]. This is not supported inside of vmap, please try to",
0555:       "rewrite the `as_strided` call as a sequence of PyTorch view operations");
0556: }
0557: 
0558: Tensor _reshape_alias_batching_rule(const Tensor& self, IntArrayRef sizes, IntArrayRef strides [[maybe_unused]]) {
0559:   return reshape_batching_rule(self, sizes);
0560: }
0561: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `_reshape_alias_batching_rule`, `reshape_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`_reshape_alias_batching_rule`, `reshape_batching_rule`。

### Lines 562-619 / 第 562-619 行

```cpp
0562: Tensor _new_zeros_with_same_feature_meta_batching_rule(
0563:     const Tensor& self,
0564:     const Tensor& other,
0565:     int64_t unused_num_batch_dims) {
0566:   TORCH_CHECK(isBatchedTensor(self) && !isBatchedTensor(other),
0567:     "Only the 'batched grad' use case is supported in PyTorch core.");
0568: 
0569:   TORCH_INTERNAL_ASSERT(unused_num_batch_dims == 0,
0570:     "num_batch_dims should not be explicitly passed in because it will be overridden");
0571:   auto self_physical_view = at::MultiBatchVmapTransform::logicalToPhysical(self);
0572:   const auto& self_physical_tensor = self_physical_view.tensor();
0573:   int64_t num_batch_dims = self_physical_view.numBatchDims();
0574:   checkBatchDimsAtFrontInLayout(self_physical_tensor.strides(), num_batch_dims);
0575:   auto result = at::_new_zeros_with_same_feature_meta(self_physical_tensor, other, num_batch_dims);
0576:   return self_physical_view.getPhysicalToLogicalMap().apply(result);
0577: }
0578: 
0579: bool _has_same_storage_numel_batching_rule(const Tensor& self, const Tensor& other) {
0580:   TORCH_CHECK(isBatchedTensor(self) && !isBatchedTensor(other),
0581:     "Only the 'batched grad' use case is supported in PyTorch core.");
0582:   // The _has_same_storage_numel check is skipped if the tangent is a batched
0583:   // tensor because using as_strided to access storage locations not indexable
0584:   // by the input tensor is not supported in vmap
0585:   return true;
0586: }
0587: 
0588: // What are the semantics of as_strided inside of vmap?
0589: // y = vmap(lambda x: x.as_strided(sizes, strides, offset))(xs)
0590: // This returns a view on `x`, `y`, such that each y[i] has:
0591: // - sizes: `sizes`
0592: // - strides: `strides`
0593: // - storage_offset: offset + i * x.stride(batch_dim)
0594: //
0595: // In other words, it is as if we had treated each x[i] as having storage
0596: // offset equal to xs.offset() and called as_strided(sizes, sizes, offset).
0597: // (that is equivalent to x[i].as_strided(
0598: //    sizes, sizes, offset + x[i].storage_offset() - xs.offset()) for all i)
0599: //
0600: // Note that this *may* be different from actually running as_strided
0601: // in a for-loop. This is due to how as_strided takes in `offset` to be
0602: // an *absolute* offset. As an example, consider:
0603: // >>> x = torch.tensor([0., 1., 2., 3., 4.]).as_strided([4], [1], 1)
0604: // >>> z = [x[i].as_strided([1], [1], 1) for i in range(4)]
0605: // Each z[i] is actually the same view on x (z[i] == torch.tensor([1.]))!
0606: // However, we consider the above for-loop comprehension to be a user error:
0607: // a user should have written the following if they wanted to use as_strided
0608: // in a per-sample way:
0609: // >>> z = [x[i].as_strided([1], [1], 1 + x[i].storage_offset() - 1) for i in range(4)]
0610: Tensor as_strided_batching_rule(
0611:     const Tensor& tensor,
0612:     IntArrayRef sizes,
0613:     IntArrayRef strides,
0614:     std::optional<int64_t> storage_offset) {
0615:   auto physical_view = at::MultiBatchVmapTransform::logicalToPhysical(tensor);
0616:   auto num_batch_dims = physical_view.numBatchDims();
0617:   auto physical_sizes = physical_view.getPhysicalShape(sizes);
0618:   const auto& physical_tensor = physical_view.tensor();
0619: 
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `_new_zeros_with_same_feature_meta_batching_rule`, `checkBatchDimsAtFrontInLayout`, `_has_same_storage_numel_batching_rule`, `as_strided_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`_new_zeros_with_same_feature_meta_batching_rule`, `checkBatchDimsAtFrontInLayout`, `_has_same_storage_numel_batching_rule`, `as_strided_batching_rule`。

### Lines 620-652 / 第 620-652 行

```cpp
0620:   // We can't rely on the physical as_strided call to do this for us because
0621:   // we do some sanity checks on the size/strides before calling into as_strided.
0622:   TORCH_CHECK(sizes.size() == strides.size(),
0623:       "Tensor.as_strided(size, stride, ...): size and stride must have the ",
0624:       "same length! Got size ", sizes, " and stride ", strides);
0625: 
0626:   // Sanity checks:
0627:   // 1. All batch dims are at the front in memory layout (not necessary for
0628:   // correctness, but we are worried the user might be doing crazy things)
0629:   // 2. as_strided(sizes, strides, storage_offset + tensor[i].offset() - tensor.offset())
0630:   // is valid for a slice of the input tensor.
0631:   // See Note: [When will the as_strided batching rule fail?] for details.
0632:   checkBatchDimsAtFrontInLayout(physical_tensor.strides(), num_batch_dims);
0633:   checkBasicAsStridedValidForSlice(
0634:       physical_tensor, num_batch_dims, sizes, strides, storage_offset);
0635: 
0636:   // physical_strides = physical tensor's batch strides + (logical) strides
0637:   auto batch_strides = physical_tensor.strides().slice(0, num_batch_dims);
0638:   at::VmapDimVector physical_strides;
0639:   physical_strides.reserve(num_batch_dims + strides.size());
0640:   physical_strides.insert(
0641:       physical_strides.end(), batch_strides.begin(), batch_strides.end());
0642:   physical_strides.insert(
0643:       physical_strides.end(), strides.begin(), strides.end());
0644: 
0645:   // If zi = xs[i].as_strided(sizes, strides, offset + xs[i].offset() - xs.offset())
0646:   // is valid for all i, then it turns out that
0647:   // xs.as_strided(physical_sizes, physical_strides, offset) always succeeds
0648:   // and creates a tensor y such that each y[i] references the same memory
0649:   // locations as zi. See NOTE: [When will the as_strided batching rule fail?]
0650:   auto result = physical_view.tensor().as_strided(
0651:       physical_sizes, physical_strides, storage_offset);
0652:   return physical_view.getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `checkBatchDimsAtFrontInLayout`, `checkBasicAsStridedValidForSlice`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`checkBatchDimsAtFrontInLayout`, `checkBasicAsStridedValidForSlice`。

### Lines 653-712 / 第 653-712 行

```cpp
0653: }
0654: 
0655: // NOTE: [When will the as_strided batching rule fail?]
0656: // If zi = xs[i].as_strided(sizes, strides, offset + xs[i].offset() - xs.offset())
0657: // is valid for all i, then it turns out that
0658: // xs.as_strided(physical_sizes, physical_strides, offset) always succeeds and
0659: // creates a tensor y such that each y[i] refers to the same memory as zi.
0660: //
0661: // Let's say we have xs[i].as_strided(sizes, strides, offset + xs[i].offset() - xs.offset()).
0662: // Furthermore, let's say that as a part of being "valid" this as_strided call
0663: // does not return a result that can index memory not indexable by xs[i].
0664: //
0665: // WLOG, assume that there's only one batch dim and it is at the front of the
0666: // `xs` tensor. Let B be the batch size and S be the stride of the batch dim.
0667: // - If the batch dim isn't at the front of the tensor, then we can just move it
0668: // to the front with movedim/permute. This is always valid because it just swaps
0669: // some strides around.
0670: // - This proof also works for tensors with multiple batch dims. We just have to
0671: // do a little accounting:
0672: //   - instead of [B], we'd have [B0, B1, ..., Bk].
0673: //   - instead of [S], we'd have [S0, S1, ..., Sk].
0674: //   - instead of i, we'd have a list of indices [I0, I1, ..., Ik]
0675: //   - instead of S * I, we'd have \sum_{i=0}^k S_i * I_i
0676: //
0677: // [Equation 1]
0678: // xs[i].as_strided(sizes, strides, offset + xs[i].offset() - xs.offset()) has:
0679: // - sizes: sizes
0680: // - strides: strides
0681: // - offset: offset + S * i
0682: //
0683: // x.as_strided itself checks that:
0684: // - (sizes, strides, offset) are in bounds for `x`'s storage.
0685: // - strides are positive
0686: // - offset is positive
0687: //
0688: // Claim 1: if xs[i].as_strided(sizes, strides, offset + xs[i].offset() - xs.offset())
0689: // is valid, then
0690: // ([B] + sizes, [S] + strides, offset + xs.offset()) are in bounds for `xs`'s storage.
0691: //
0692: // If we have the claim, then xs.as_strided([B] + sizes, [S] + strides, offset)
0693: // won't error out. So all we need to check is that the memory locations are
0694: // what we expected. See [Hand-wavy proof of Claim 1] for proof (it's not very important)
0695: //
0696: // xs.as_strided(physical_sizes, physical_strides, offset) is equivalent to
0697: // xs.as_strided([B] + sizes, [S] + strides, offset)
0698: //
0699: // xs.as_strided([B] + sizes, [S] + strides, offset) has:
0700: // - sizes: [B] + sizes
0701: // - strides: [S] + strides
0702: // - offset: offset
0703: //
0704: // xs.as_strided([B] + sizes, [S] + strides, offset)[i] has:
0705: // - sizes: sizes
0706: // - strides: strides
0707: // - offset: offset + S * i
0708: // These memory locations are exactly the same as what we got for [Equation 1],
0709: // so the xs.as_strided([B] + sizes, [S] + strides, offset) is valid.
0710: //
0711: // [Hand-wavy proof of Claim 1]
0712: // Part of our definition of being valid is that xs[i].as_strided(...)
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 713-745 / 第 713-745 行

```cpp
0713: // must return a tensor that only uses memory indexable by xs[i].
0714: // This means that (sizes, strides, offset + xs[i].offset() - xs.offset()) satisfies:
0715: //    offset + xs[i].offset() - xs.offset() + 1 + \sum_j (sizes[j] - 1) * strides[j]
0716: //    <= xs[i].offset() + 1 + \sum_j (xs[i].size(j) - 1) * xs[i].stride(j)
0717: // (the largest-index memory location of xs[i].as_strided(...) must be \leq
0718: // the largest-index memory location of xs[i])
0719: //
0720: // Fiddling that inequality gives us:
0721: //    offset - xs.offset() + 1 + \sum_j (sizes[j] - 1) * strides[j]
0722: //    <= 1 + \sum_j (xs[i].size(j) - 1) * xs[i].stride(j)
0723: //
0724: //    offset - xs.offset() + 1 + (B-1)*S + \sum_j (sizes[j] - 1) * strides[j]
0725: //    <= 1 + (B-1)*S + \sum_j (xs[i].size(j) - 1) * xs[i].stride(j)
0726: //
0727: //    offset - xs.offset() + 1 + (B-1)*S + \sum_j (sizes[j] - 1) * strides[j]
0728: //    <= 1 + \sum_j (xs.size(j) - 1) * xs.stride(j)
0729: //
0730: //    offset + 1 + (B-1)*S + \sum_j (sizes[j] - 1) * strides[j]
0731: //    <= xs.offset() + 1 + \sum_j (xs.size(j) - 1) * xs.stride(j)
0732: // (the largest-index memory location of xs.as_strided(size, stride, offset)
0733: // is \leq than the largest-index memory location of xs)
0734: // Under the assumptions we've made, the lower bound (lowest indexed memory)
0735: // is trivially within the storage.
0736: //
0737: // Therefore ([B] + sizes, [S] + strides, offset) are in bounds for
0738: // `xs`'s storage.
0739: 
0740: template <typename F, F Func, typename... ExtraArgs>
0741: Tensor unwrap_and_call(const Tensor& input, ExtraArgs... args) {
0742:   auto* input_batched = unsafeGetBatchedImpl(input);
0743:   auto output_physical = Func(input_batched->value(), args...);
0744:   auto old_bdims = input_batched->bdims();
0745:   return makeBatched(output_physical, BatchDims(old_bdims.begin(), old_bdims.end()));
```

- **EN:** Builds a reusable template/helper layer around `LegacyBatchingRegistrations`. Key symbols: `unwrap_and_call`, `makeBatched`.
- **CN:** 围绕 `LegacyBatchingRegistrations` 构建可复用的模板或辅助层。关键符号：`unwrap_and_call`, `makeBatched`。

### Lines 746-776 / 第 746-776 行

```cpp
0746: }
0747: 
0748: template <typename F, F Func, typename... ExtraArgs>
0749: Tensor unwrap_and_call_method(const Tensor& input, ExtraArgs... extra_args) {
0750:   auto* input_batched = unsafeGetBatchedImpl(input);
0751:   auto output_physical = (input_batched->value().*Func)(extra_args...);
0752:   auto old_bdims = input_batched->bdims();
0753:   return makeBatched(output_physical, BatchDims(old_bdims.begin(), old_bdims.end()));
0754: }
0755: 
0756: Tensor pow_scalar_Tensor_batching_rule(const Scalar& other, const Tensor& self) {
0757:   auto* self_batched = unsafeGetBatchedImpl(self);
0758:   auto output_physical = at::pow(other, self_batched->value());
0759:   auto old_bdims = self_batched->bdims();
0760:   return makeBatched(output_physical, BatchDims(old_bdims.begin(), old_bdims.end()));
0761: }
0762: 
0763: Tensor clone_batching_rule(const Tensor& self, std::optional<MemoryFormat> memory_format) {
0764:   // Memory format support is a little tricky because vmap is allowed to move
0765:   // around batch dimensions and some memory formats are rank-dependent.
0766:   // Another weird case is:
0767:   // - a tensor with MemoryFormat::ChannelsLast MUST have 4 dimensions. Do we
0768:   //   allow the user to clone a Tensor with 3 logical dimensions and 1 batch
0769:   //   dim into a ChannelsLast Tensor? What about a Tensor with 3 logical dims
0770:   //   and N>1 batch dims?
0771:   TORCH_CHECK(!memory_format.has_value() || memory_format == MemoryFormat::Preserve
0772:       || memory_format == MemoryFormat::Contiguous,
0773:       "NYI: Tensor.clone(memory_format) inside vmap is only supported with ",
0774:       "memory_format torch.preserve_format or torch.contiguous_format (got ",
0775:       *memory_format, ")");
0776: 
```

- **EN:** Builds a reusable template/helper layer around `LegacyBatchingRegistrations`. Key symbols: `unwrap_and_call_method`, `makeBatched`, `pow_scalar_Tensor_batching_rule`, `clone_batching_rule`.
- **CN:** 围绕 `LegacyBatchingRegistrations` 构建可复用的模板或辅助层。关键符号：`unwrap_and_call_method`, `makeBatched`, `pow_scalar_Tensor_batching_rule`, `clone_batching_rule`。

### Lines 777-808 / 第 777-808 行

```cpp
0777:   if (memory_format == MemoryFormat::Contiguous) {
0778:     // There is an ambiguity here when the batch dims are not at the front of
0779:     // the tensor.
0780:     // >>> x = torch.randn(3, B0, 5)
0781:     // >>> y = vmap(lambda x: x.clone(torch.contiguous_format), in_dims=1, out_dims=0)(x)
0782:     // >>> y[0].is_contiguous()
0783:     // ???
0784:     // Should we make the whole tensor contiguous, or should we
0785:     // make the non-batch dims contiguous? We've chosen the latter because
0786:     // philosophically vmap hides the batch dims and operates on a per-sample level.
0787:     auto physical_view = MultiBatchVmapTransform::logicalToPhysical(self);
0788:     auto output_physical = at::clone(physical_view.tensor(), memory_format);
0789:     return physical_view.getPhysicalToLogicalMap().apply(output_physical);
0790:   }
0791: 
0792:   TORCH_INTERNAL_ASSERT(!memory_format.has_value() || memory_format == MemoryFormat::Preserve);
0793:   auto* self_batched = unsafeGetBatchedImpl(self);
0794:   auto output_physical = at::clone(self_batched->value(), memory_format);
0795:   auto old_bdims = self_batched->bdims();
0796:   return makeBatched(output_physical, BatchDims(old_bdims.begin(), old_bdims.end()));
0797: }
0798: 
0799: // Note [Batching rules for matmul-like operators]
0800: // at::matmul doesn't "de-expand" arguments to get better performance (maybe
0801: // it should). In the batching rules for matmul-like operators (dot, mv, mm),
0802: // we should be careful not to expand any unnecessary dimensions. e.g., if
0803: // only one of the two arguments is a BatchedTensor, then we should try
0804: // not to expand batch dimensions onto the other arg.
0805: Tensor mv_batching_rule(const Tensor& self, const Tensor& other) {
0806:   auto self_batched = isBatchedTensor(self);
0807:   auto other_batched = isBatchedTensor(other);
0808: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `makeBatched`, `mv_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`makeBatched`, `mv_batching_rule`。

### Lines 809-839 / 第 809-839 行

```cpp
0809:   // A shape checking API would be nice...
0810:   TORCH_CHECK(self.dim() == 2 && other.dim() == 1,
0811:       "mv(self, other): Shape mismatch: expected matrix "
0812:       "(got `self` of size ", self.sizes(), ") ",
0813:       "and vector (got `other` of size ", other.sizes(), ")");
0814: 
0815:   // See Note [Batching rules for matmul-like operators] for why we have cases
0816:   if (self_batched && !other_batched) {
0817:     auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0818:     auto result = at::matmul(self_physical.tensor(), other);
0819:     return self_physical.getPhysicalToLogicalMap().apply(result);
0820:   }
0821:   if (!self_batched && other_batched) {
0822:     // self_physical: [L, K], other_physical: [..., K]
0823:     // We view the tensors as [L, K], [..., K, 1], perform matmul to get
0824:     // a tensor of size [..., L, 1], and unsqueeze the last dim.
0825:     auto other_physical = MultiBatchVmapTransform::logicalToPhysical(other);
0826:     auto result = at::matmul(self, other_physical.tensor().unsqueeze(-1));
0827:     return other_physical.getPhysicalToLogicalMap().apply(result.squeeze(-1));
0828:   }
0829:   if (self_batched && other_batched) {
0830:     // self_physical: [..., L, K], other_physical: [..., K]
0831:     // We view the tensors as [..., L, K], [..., K, 1], perform matmul to get
0832:     // a tensor of size [..., L, 1], and unsqueeze the last dim.
0833:     auto physical_args = MultiBatchVmapTransform::logicalToPhysical({self, other});
0834:     auto result = at::matmul(
0835:         physical_args[0].tensor(),
0836:         physical_args[1].tensor().unsqueeze(-1));
0837:     return physical_args[0].getPhysicalToLogicalMap().apply(result.squeeze(-1));
0838:   }
0839:   TORCH_INTERNAL_ASSERT(false, "either self or other must be a BatchedTensor");
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：无明显局部符号。

### Lines 840-870 / 第 840-870 行

```cpp
0840: }
0841: 
0842: Tensor _make_dual_batching_rule(
0843:   c10::DispatchKeySet ks,
0844:   const Tensor& primal,
0845:   const Tensor& tangent,
0846:   int64_t level
0847: ) {
0848:   DispatchKeySet after_batched_keyset =
0849:       DispatchKeySet(DispatchKeySet::FULL_AFTER, c10::DispatchKey::Batched);
0850:   return at::redispatch::_make_dual(ks & after_batched_keyset, primal, tangent, level);
0851: }
0852: 
0853: Tensor dot_batching_rule(const Tensor& self, const Tensor& other) {
0854:   auto self_batched = isBatchedTensor(self);
0855:   auto other_batched = isBatchedTensor(other);
0856: 
0857:   TORCH_CHECK(/*logical*/self.dim() == 1 && /*logical*/other.dim() == 1,
0858:       "dot(self, other): Shape mismatch: vector "
0859:       "(got `self` of size ", self.sizes(), ") ",
0860:       "and vector (got `other` of size ", other.sizes(), ")");
0861: 
0862:   // See Note [Batching rules for matmul-like operators] for why we have cases
0863:   if (self_batched && !other_batched) {
0864:     // self_physical: [..., K], other_physical: [K]
0865:     // View the tensors as [..., 1, K] and [K], perform matmul, and unsqueeze.
0866:     auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0867:     auto result = at::matmul(self_physical.tensor().unsqueeze(-2), other);
0868:     return self_physical.getPhysicalToLogicalMap().apply(result.squeeze(-1));
0869:   }
0870:   if (!self_batched && other_batched) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `_make_dual_batching_rule`, `DispatchKeySet`, `_make_dual`, `dot_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`_make_dual_batching_rule`, `DispatchKeySet`, `_make_dual`, `dot_batching_rule`。

### Lines 871-903 / 第 871-903 行

```cpp
0871:     // self_physical: [K], other_physical: [..., K]
0872:     // View the tensors as [K] and [..., K, 1], perform matmul, and unsqueeze.
0873:     auto other_physical = MultiBatchVmapTransform::logicalToPhysical(other);
0874:     auto result = at::matmul(self, other_physical.tensor().unsqueeze(-1));
0875:     return other_physical.getPhysicalToLogicalMap().apply(result.squeeze(-1));
0876:   }
0877:   if (self_batched && other_batched) {
0878:     // self_physical: [..., K], other_physical: [..., K]
0879:     // View the tensors as [..., 1, K] and [..., K, 1], perform matmul, and unsqueeze.
0880:     auto physical_args = MultiBatchVmapTransform::logicalToPhysical({self, other});
0881:     auto result = at::matmul(
0882:         physical_args[0].tensor().unsqueeze(-2),
0883:         physical_args[1].tensor().unsqueeze(-1));
0884:     return physical_args[0].getPhysicalToLogicalMap().apply(result.squeeze(-1).squeeze(-1));
0885:   }
0886:   TORCH_INTERNAL_ASSERT(false, "either self or other must be a BatchedTensor");
0887: }
0888: 
0889: Tensor bmm_batching_rule(const Tensor& self, const Tensor& other) {
0890:   TORCH_CHECK(/*logical*/self.dim() == 3 && /*logical*/other.dim() == 3,
0891:       "bmm(self, other): Shape mismatch: expected 3D `self` "
0892:       "(got `self` of size ", self.sizes(), ") ",
0893:       "and 3D `other` (got `other` of size ", other.sizes(), ")");
0894: 
0895:   auto physical_args = BroadcastingVmapTransform::logicalToPhysical({self, other});
0896:   auto result = at::matmul(physical_args[0].tensor(), physical_args[1].tensor());
0897:   return physical_args[0].getPhysicalToLogicalMap().apply(result);
0898: }
0899: 
0900: Tensor mm_batching_rule(const Tensor& self, const Tensor& other) {
0901:   auto self_batched = isBatchedTensor(self);
0902:   auto other_batched = isBatchedTensor(other);
0903: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `bmm_batching_rule`, `mm_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`bmm_batching_rule`, `mm_batching_rule`。

### Lines 904-935 / 第 904-935 行

```cpp
0904:   TORCH_CHECK(/*logical*/self.dim() == 2 && /*logical*/other.dim() == 2,
0905:       "mm(self, other): Shape mismatch: expected matrix "
0906:       "(got `self` of size ", self.sizes(), ") ",
0907:       "and matrix (got `other` of size ", other.sizes(), ")");
0908: 
0909:   // See Note [Batching rules for matmul-like operators] for why we have cases
0910:   if (self_batched && !other_batched) {
0911:     auto self_physical = MultiBatchVmapTransform::logicalToPhysical(self);
0912:     auto result = at::matmul(self_physical.tensor(), other);
0913:     return self_physical.getPhysicalToLogicalMap().apply(result);
0914:   }
0915:   if (!self_batched && other_batched) {
0916:     auto other_physical = MultiBatchVmapTransform::logicalToPhysical(other);
0917:     auto result = at::matmul(self, other_physical.tensor());
0918:     return other_physical.getPhysicalToLogicalMap().apply(result);
0919:   }
0920:   if (self_batched && other_batched) {
0921:     auto physical_args = MultiBatchVmapTransform::logicalToPhysical({self, other});
0922:     auto result = at::matmul(physical_args[0].tensor(), physical_args[1].tensor());
0923:     return physical_args[0].getPhysicalToLogicalMap().apply(result.squeeze(-1).squeeze(-1));
0924:   }
0925:   TORCH_INTERNAL_ASSERT(false, "either self or other must be a BatchedTensor");
0926: }
0927: 
0928: Tensor cat_batching_rule(const ITensorListRef& tensors, int64_t dim) {
0929:   auto physical_views = MultiBatchVmapTransform::logicalToPhysical(tensors);
0930:   auto physical_tensors = fmap(
0931:       physical_views, [](const VmapPhysicalView& view) -> Tensor { return view.tensor(); });
0932:   TORCH_INTERNAL_ASSERT(
0933:       !tensors.empty(), "The dispatcher should not have dispatched here otherwise.");
0934:   auto result = at::cat(physical_tensors, physical_views[0].getPhysicalDim(dim));
0935:   return physical_views[0].getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `cat_batching_rule`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`cat_batching_rule`。

### Lines 936-972 / 第 936-972 行

```cpp
0936: }
0937: 
0938: Tensor stack_batching_rule(TensorList tensors, int64_t dim) {
0939:   auto physical_views = MultiBatchVmapTransform::logicalToPhysical(tensors);
0940:   auto physical_tensors = fmap(
0941:       physical_views, [](const VmapPhysicalView& view) -> Tensor { return view.tensor(); });
0942:   TORCH_INTERNAL_ASSERT(
0943:       !tensors.empty(), "The dispatcher should not have dispatched here otherwise.");
0944:   // NB: stack wraps the dimensionality to (logical dim + 1), so we have to
0945:   // manually handle that here.
0946:   auto dim_physical =
0947:       physical_views[0].numBatchDims() + maybe_wrap_dim(dim, /*logical*/tensors[0].dim() + 1);
0948:   auto result = at::stack(physical_tensors, dim_physical);
0949:   return physical_views[0].getPhysicalToLogicalMap().apply(result);
0950: }
0951: 
0952: // I am quite sad that we need to register operators with exploded TensorOptions,
0953: // even though the native:: implementations can use TensorOptions&.
0954: // This also makes it hard to metaprogram: i.e., we can't use
0955: // unwrap_and_call<..., at::to> because at::to takes TensorOptions& (!!)
0956: Tensor to_dtype_layout_batching_rule(
0957:     const Tensor& self,
0958:     std::optional<ScalarType> dtype,
0959:     std::optional<Layout> layout,
0960:     std::optional<Device> device,
0961:     std::optional<bool> pin_memory,
0962:     bool non_blocking, bool copy,
0963:     std::optional<MemoryFormat> memory_format) {
0964:   auto options = TensorOptions()
0965:     .dtype(dtype)
0966:     .layout(layout)
0967:     .device(device)
0968:     .pinned_memory(pin_memory);
0969:   auto* input_batched = unsafeGetBatchedImpl(self);
0970:   auto output_physical = input_batched->value().to(options, non_blocking, copy, memory_format);
0971:   auto old_bdims = input_batched->bdims();
0972:   return makeBatched(output_physical, BatchDims(old_bdims.begin(), old_bdims.end()));
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; supports transform-aware functorch semantics. Key symbols: `stack_batching_rule`, `to_dtype_layout_batching_rule`, `makeBatched`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；支持面向变换的 functorch 语义。关键符号：`stack_batching_rule`, `to_dtype_layout_batching_rule`, `makeBatched`。

### Lines 973-1003 / 第 973-1003 行

```cpp
0973: }
0974: 
0975: Tensor new_zeros_batching_rule(
0976:     const Tensor& self,
0977:     IntArrayRef size,
0978:     std::optional<ScalarType> dtype,
0979:     std::optional<Layout> layout,
0980:     std::optional<Device> device,
0981:     std::optional<bool> pin_memory) {
0982:   auto physical_view = MultiBatchVmapTransform::logicalToPhysical(self);
0983:   auto physical_size = physical_view.getPhysicalShape(size);
0984:   auto options = TensorOptions()
0985:     .dtype(dtype)
0986:     .layout(layout)
0987:     .device(device)
0988:     .pinned_memory(pin_memory);
0989:   auto result = physical_view.tensor().new_zeros(physical_size, options);
0990:   return physical_view.getPhysicalToLogicalMap().apply(result);
0991: }
0992: 
0993: Tensor new_empty_batching_rule(
0994:     const Tensor& self,
0995:     IntArrayRef size,
0996:     std::optional<ScalarType> dtype,
0997:     std::optional<Layout> layout,
0998:     std::optional<Device> device,
0999:     std::optional<bool> pin_memory) {
1000:   auto physical_view = MultiBatchVmapTransform::logicalToPhysical(self);
1001:   auto physical_size = physical_view.getPhysicalShape(size);
1002:   auto result = physical_view.tensor().new_empty(physical_size, TensorOptions().dtype(dtype).layout(layout).device(device).pinned_memory(pin_memory));
1003:   return physical_view.getPhysicalToLogicalMap().apply(result);
```

- **EN:** This block produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; supports transform-aware functorch semantics. Key symbols: `new_zeros_batching_rule`, `new_empty_batching_rule`.
- **CN:** 该代码块返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；支持面向变换的 functorch 语义。关键符号：`new_zeros_batching_rule`, `new_empty_batching_rule`。

### Lines 1004-1045 / 第 1004-1045 行

```cpp
1004: }
1005: 
1006: Tensor new_empty_strided_batching_rule(
1007:     const Tensor& self,
1008:     IntArrayRef size,
1009:     IntArrayRef stride,
1010:     std::optional<ScalarType> dtype,
1011:     std::optional<Layout> layout,
1012:     std::optional<Device> device,
1013:     std::optional<bool> pin_memory) {
1014:   auto physical_view = MultiBatchVmapTransform::logicalToPhysical(self);
1015:   auto physical_size = physical_view.getPhysicalShape(size);
1016: 
1017:   // Let [B0, B1, B2] be the shape of the batch dims. We're going to create
1018:   // the batch dimensions at the front of the tensor (in memory layout),
1019:   // irrespective of whether or not they are actually at the front (in memory layout)
1020:   // in the original `self` tensor. This is because when a user calls
1021:   // `new_empty_strided` in general, the `strides` they provide are for a new
1022:   // tensor and have no relation to the strides of the original tensor.
1023:   //
1024:   // So, the physical shape of the result should be ([B0, B1, B2] + size),
1025:   // but what about the physical strides?
1026:   //
1027:   // We're actually free to pick whatever stride we want:
1028:   // e.g., for size=[5, 3], stride=[0, 1], we could decide to
1029:   // use
1030:   // - physical size: [B0, B1, B2, 5, 3]
1031:   // - physical stride: [9999*B1*B2, 9999*B2, 9999, 0, 1]
1032:   //
1033:   // Let's select some reasonable strides such that:
1034:   // - The batch dims are "contiguous" with respect to each other
1035:   // - if empty_strided(size, stride) would have created a contiguous Tensor,
1036:   // then this new physical Tensor (with batch dims) is also contiguous
1037:   //
1038:   // Let S be the size of the storage if one were to construct a tensor
1039:   // with `size` and `stride` via empty_strided(size, stride).
1040:   // Then the physical sizes/strides should be:
1041:   // - physical size: [B0, B1, B2, 5, 3]
1042:   // - physical stride: [B1 * B2 * S, B2 * S, S, 0, 1]
1043:   auto batch_shape = IntArrayRef(
1044:       physical_view.tensor().sizes().begin(), physical_view.numBatchDims());
1045: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; manages memory allocation, buffers, or ownership boundaries; supports transform-aware functorch semantics. Key symbols: `new_empty_strided_batching_rule`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；管理内存分配、缓冲区或所有权边界；支持面向变换的 functorch 语义。关键符号：`new_empty_strided_batching_rule`。

### Lines 1046-1075 / 第 1046-1075 行

```cpp
1046:   // physical_strides = [B1 * B2 * S, B2 * S, S]
1047:   auto physical_strides = at::detail::defaultStrides(batch_shape);
1048:   TORCH_CHECK(size.size() == stride.size(),
1049:         "new_empty_strided(sizes, strides): dimensionality of sizes (",
1050:         size.size(), ") must match dimensionality of strides (",
1051:         stride.size(), ")");
1052:   auto storage_size = native::storage_size_for(size, stride);
1053:   for (auto& physical_stride : physical_strides) {
1054:     physical_stride *= storage_size;
1055:   }
1056: 
1057:   // physical_strides = [B1 * B2 * S, B2 * S, S] + strides
1058:   physical_strides.insert(physical_strides.end(), stride.begin(), stride.end());
1059: 
1060:   auto result = physical_view.tensor().new_empty_strided(
1061:       physical_size, physical_strides, dtype, layout, device, pin_memory);
1062:   return physical_view.getPhysicalToLogicalMap().apply(result);
1063: }
1064: 
1065: template <typename F, F Func>
1066: Tensor comparison_pointwise_batching_rule(const Tensor& self, const Tensor& other) {
1067:   auto physical_args = BroadcastingVmapTransform::logicalToPhysical({self, other});
1068:   auto result = Func(physical_args[0].tensor(), physical_args[1].tensor());
1069:   return physical_args[0].getPhysicalToLogicalMap().apply(result);
1070: }
1071: }
1072: TORCH_LIBRARY_IMPL(_, Batched, m) {
1073:   m.fallback(torch::CppFunction::makeFromBoxedFunction<&batchedTensorForLoopFallback>());
1074: }
1075: 
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `LegacyBatchingRegistrations` behavior. Symbols: `comparison_pointwise_batching_rule`.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `LegacyBatchingRegistrations` 的行为。符号：`comparison_pointwise_batching_rule`。

### Lines 1076-1134 / 第 1076-1134 行

```cpp
1076: TORCH_LIBRARY_IMPL(aten, Batched, m) {
1077:   // NB: Ideally we would like some operators, like size.int, to "fallthrough"
1078:   // to the underlying implementation. However, because a BatchedTensor is a
1079:   // Tensor wrapper, it only has one dispatch key (Batched) on it. The resolution
1080:   // here is to just directly call the underlying implementation.
1081:   m.impl("size.int", static_cast<int64_t (*)(const Tensor&, int64_t)>(native::size));
1082:   m.impl("_add_batch_dim", native::_add_batch_dim);
1083:   m.impl("_remove_batch_dim", native::_remove_batch_dim);
1084:   m.impl("_make_dual", _make_dual_batching_rule);
1085:   m.impl("_has_same_storage_numel", _has_same_storage_numel_batching_rule);
1086:   m.impl("is_same_size", native::is_same_size);
1087:   m.impl("_new_zeros_with_same_feature_meta", _new_zeros_with_same_feature_meta_batching_rule);
1088: 
1089:   m.impl("sum.dim_IntList", sum_batching_rule);
1090:   m.impl("is_complex", native::is_complex);
1091: 
1092:   // inplace operations
1093:   m.impl("fill_.Scalar", fill_inplace_scalar_batching_rule);
1094:   m.impl("fill_.Tensor", fill_inplace_tensor_batching_rule);
1095:   m.impl("zero_", zero_inplace_batching_rule);
1096: 
1097:   // view operations
1098:   m.impl("as_strided", as_strided_batching_rule);
1099:   m.impl("chunk", chunk_batching_rule);
1100:   m.impl("tensor_split.sections", tensor_split_sections_batching_rule);
1101:   m.impl("tensor_split.indices", tensor_split_indices_batching_rule);
1102:   m.impl("diagonal", diagonal_batching_rule);
1103:   m.impl("expand", expand_batching_rule);
1104:   m.impl("expand_as", native::expand_as); // composite wrt autograd
1105:   m.impl("movedim.intlist", movedim_batching_rule);
1106:   m.impl("movedim.int", static_cast<Tensor(*)(const Tensor&,int64_t,int64_t)>(native::movedim)); // composite wrt autograd
1107:   // There is another variant of narrow.  However, we don't
1108:   // want to support the other variant yet bc it isn't documented...
1109:   m.impl("narrow", native::narrow_symint); // composite wrt autograd
1110:   m.impl("numpy_T", native::numpy_T);   // composite wrt autograd
1111:   m.impl("matrix_H", native::matrix_H); // composite wrt autograd
1112:   m.impl("mT", native::mT);             // composite wrt autograd
1113:   m.impl("mH", native::mH);             // composite wrt autograd
1114:   m.impl("permute", permute_batching_rule);
1115:   m.impl("reshape", reshape_batching_rule);
1116:   m.impl("_reshape_alias", _reshape_alias_batching_rule);
1117:   m.impl("reshape_as", native::reshape_as); // composite wrt autograd
1118:   m.impl("select.int", select_batching_rule);
1119:   m.impl("slice.Tensor", slice_batching_rule);
1120:   m.impl("split.Tensor", split_batching_rule);
1121:   m.impl("split.sizes", split_with_sizes_batching_rule);
1122:   m.impl("split_with_sizes", split_with_sizes_batching_rule);
1123:   m.impl("squeeze", squeeze_batching_rule);
1124:   m.impl("squeeze.dim", squeeze_dim_batching_rule);
1125:   m.impl("squeeze.dims", squeeze_dims_batching_rule);
1126:   m.impl("t", native::t); // composite wrt autograd
1127:   m.impl("trace", trace_batching_rule);
1128:   m.impl("transpose.int", transpose_int_batching_rule);
1129:   m.impl("unbind.int", unbind_batching_rule);
1130:   m.impl("unfold", unfold_batching_rule);
1131:   m.impl("unsqueeze", unsqueeze_batching_rule);
1132:   m.impl("view", view_batching_rule);
1133:   m.impl("view_as", native::view_as); // composite wrt autograd
1134: 
```

- **EN:** Registers dispatch hooks, operator entry points, or specialization glue so runtime code can discover `LegacyBatchingRegistrations` behavior. Symbols: no prominent local symbols.
- **CN:** 注册调度钩子、算子入口或特化胶水代码，使运行时能够发现 `LegacyBatchingRegistrations` 的行为。符号：无明显局部符号。

### Lines 1135-1187 / 第 1135-1187 行

```cpp
1135:   // clamp operations
1136:   m.impl("clamp", clamp_batching_rule);
1137:   m.impl("clamp_min", clamp_min_batching_rule);
1138:   m.impl("clamp_max", clamp_max_batching_rule);
1139: 
1140:   // unary pointwise, out-of-place, no additional arguments.
1141: #define UNARY_POINTWISE(op) m.impl(#op, \
1142:     unwrap_and_call<Tensor (*)(const Tensor&), at::op>);
1143:   UNARY_POINTWISE(abs);
1144:   UNARY_POINTWISE(acos);
1145:   UNARY_POINTWISE(asin);
1146:   UNARY_POINTWISE(atan);
1147:   UNARY_POINTWISE(ceil);
1148:   UNARY_POINTWISE(cos);
1149:   UNARY_POINTWISE(cosh);
1150:   UNARY_POINTWISE(conj_physical);
1151:   UNARY_POINTWISE(digamma);
1152:   UNARY_POINTWISE(exp);
1153:   UNARY_POINTWISE(expm1);
1154:   UNARY_POINTWISE(floor);
1155:   UNARY_POINTWISE(frac);
1156:   UNARY_POINTWISE(lgamma);
1157:   UNARY_POINTWISE(log);
1158:   UNARY_POINTWISE(log10);
1159:   UNARY_POINTWISE(log1p);
1160:   UNARY_POINTWISE(log2);
1161:   UNARY_POINTWISE(neg);
1162:   UNARY_POINTWISE(reciprocal);
1163:   UNARY_POINTWISE(relu);
1164:   UNARY_POINTWISE(round);
1165:   UNARY_POINTWISE(rsqrt);
1166:   UNARY_POINTWISE(sigmoid);
1167:   UNARY_POINTWISE(sign);
1168:   UNARY_POINTWISE(sin);
1169:   UNARY_POINTWISE(sinh);
1170:   UNARY_POINTWISE(sqrt);
1171:   UNARY_POINTWISE(tan);
1172:   UNARY_POINTWISE(tanh);
1173:   UNARY_POINTWISE(trunc);
1174: #undef UNARY_POINTWISE
1175: #define TO_BATCHING_RULE(name, ...) \
1176:   { \
1177:     using to_type = Tensor(Tensor::*)(__VA_ARGS__) const; \
1178:     m.impl(name, unwrap_and_call_method< \
1179:         to_type, &Tensor::to, __VA_ARGS__>);\
1180:   }
1181:   TO_BATCHING_RULE("to.device", Device, ScalarType, bool, bool, std::optional<MemoryFormat>)
1182:   TO_BATCHING_RULE("to.dtype", ScalarType, bool, bool, std::optional<MemoryFormat>)
1183:   TO_BATCHING_RULE("to.other", const Tensor&, bool, bool, std::optional<MemoryFormat>)
1184:   m.impl("to.dtype_layout", to_dtype_layout_batching_rule);
1185: #undef TO_BATCHING_RULE
1186:   m.impl("clone", clone_batching_rule);
1187: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: `to_type`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：`to_type`。

### Lines 1188-1219 / 第 1188-1219 行

```cpp
1188:   using TensorTensorScalarType = Tensor (*)(const Tensor&, const Tensor&, const Scalar&);
1189:   using TensorTensorType = Tensor (*)(const Tensor&, const Tensor&);
1190:   using TensorScalarType = Tensor (*)(const Tensor&, const Scalar&);
1191: 
1192: #define BINARY_POINTWISE(op) \
1193:   m.impl(#op".Tensor", binary_pointwise_batching_rule<TensorTensorType, at::op>); \
1194:   m.impl(#op".Scalar", unwrap_and_call<TensorScalarType, at::op, const Scalar&>);
1195: #define BINARY_POINTWISE_VA(op, ...) \
1196:   { \
1197:     using Binop = Tensor (*)(const Tensor&, const Tensor&, __VA_ARGS__); \
1198:     using Unop = Tensor (*)(const Tensor&, const Scalar&, __VA_ARGS__); \
1199:     m.impl(#op".Tensor", binary_pointwise_batching_rule<Binop, at::op, __VA_ARGS__>); \
1200:     m.impl(#op".Scalar", unwrap_and_call<Unop, at::op, const Scalar&, __VA_ARGS__>); \
1201:   }
1202: 
1203:   BINARY_POINTWISE_VA(add, const Scalar&);
1204:   BINARY_POINTWISE_VA(sub, const Scalar&);
1205:   BINARY_POINTWISE_VA(rsub, const Scalar&);
1206:   BINARY_POINTWISE(mul);
1207:   BINARY_POINTWISE(div);
1208:   {
1209:     using Binop = Tensor (*)(const Tensor&, const Tensor&, std::optional<std::string_view>);
1210:     using Unop = Tensor (*)(const Tensor&, const Scalar&, std::optional<std::string_view>);
1211:     m.impl("div.Tensor_mode", binary_pointwise_batching_rule<Binop, at::div, std::optional<std::string_view>>);
1212:     m.impl("div.Scalar_mode", unwrap_and_call<Unop, at::div, const Scalar&, std::optional<std::string_view>>);
1213:   }
1214: 
1215:   // at::pow has three out-of-place overloads
1216:   m.impl("pow.Tensor_Tensor", binary_pointwise_batching_rule<TensorTensorType, at::pow>);
1217:   m.impl("pow.Tensor_Scalar", unwrap_and_call<TensorScalarType, at::pow, const Scalar&>);
1218:   m.impl("pow.Scalar", pow_scalar_Tensor_batching_rule);
1219: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: `TensorTensorScalarType`, `TensorTensorType`, `TensorScalarType`, `Binop`, `Unop`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：`TensorTensorScalarType`, `TensorTensorType`, `TensorScalarType`, `Binop`, `Unop`。

### Lines 1220-1252 / 第 1220-1252 行

```cpp
1220:   m.impl("sigmoid_backward", binary_pointwise_batching_rule<TensorTensorType, at::sigmoid_backward>);
1221:   m.impl(
1222:       "threshold_backward",
1223:       binary_pointwise_batching_rule<
1224:           TensorTensorScalarType,
1225:           at::threshold_backward,
1226:           const Scalar&>);
1227: 
1228:   // for at::result_type, call the native::result_type implementation.
1229:   // We don't have to do anything special because native::result_type operates
1230:   // on the logical shape of the tensors.
1231:   m.impl("result_type.Tensor", static_cast<ScalarType (*)(const Tensor&, const Tensor&)>(native::result_type));
1232:   m.impl("result_type.Scalar", static_cast<ScalarType (*)(const Tensor&, const Scalar&)>(native::result_type));
1233:   m.impl("result_type.Scalar_Tensor", static_cast<ScalarType (*)(const Scalar&, const Tensor&)>(native::result_type));
1234:   m.impl("result_type.Scalar_Scalar", static_cast<ScalarType (*)(const Scalar&, const Scalar&)>(native::result_type));
1235: 
1236: #undef BINARY_POINTWISE_VA
1237: #undef BINARY_POINTWISE
1238: 
1239: 
1240: #define TRIVIAL_OP(op) m.impl(#op, \
1241:     unwrap_and_call<Tensor (*)(const Tensor&), at::op>);
1242:   // complex number view operators
1243:   TRIVIAL_OP(imag)
1244:   TRIVIAL_OP(real);
1245:   TRIVIAL_OP(view_as_real);
1246:   TRIVIAL_OP(conj);
1247:   TRIVIAL_OP(_conj);
1248:   TRIVIAL_OP(resolve_conj);
1249:   TRIVIAL_OP(resolve_neg);
1250:   m.impl("view_as_complex", view_as_complex_batching_rule);
1251: #undef TRIVIAL
1252: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 1253-1287 / 第 1253-1287 行

```cpp
1253:   // matmul-like operators
1254:   m.impl("mv", mv_batching_rule);
1255:   m.impl("dot", dot_batching_rule);
1256:   m.impl("bmm", bmm_batching_rule);
1257:   m.impl("mm", mm_batching_rule);
1258: 
1259:   // cat/stack
1260:   m.impl("cat", cat_batching_rule);
1261:   m.impl("stack", stack_batching_rule);
1262: 
1263:   // backward operators
1264:   m.impl("select_backward", select_backward_batching_rule);
1265:   m.impl("slice_backward", slice_backward_batching_rule);
1266:   m.impl("trace_backward", trace_backward_batching_rule);
1267:   m.impl("diagonal_backward", diagonal_backward_batching_rule);
1268: 
1269:   // Tensor.new_* operators
1270:   m.impl("new_empty", new_empty_batching_rule);
1271:   m.impl("new_empty_strided", new_empty_strided_batching_rule);
1272:   m.impl("new_zeros", new_zeros_batching_rule);
1273: 
1274:   m.impl("contiguous", contiguous_batching_rule);
1275: 
1276:   // Comparison ops
1277: #define COMPARISON_POINTWISE(op) \
1278:   m.impl(#op".Tensor", comparison_pointwise_batching_rule<TensorTensorType, at::op>); \
1279:   m.impl(#op".Scalar", unwrap_and_call<TensorScalarType, at::op, const Scalar&>);
1280: 
1281:   COMPARISON_POINTWISE(eq);
1282:   COMPARISON_POINTWISE(gt);
1283:   COMPARISON_POINTWISE(ge);
1284:   COMPARISON_POINTWISE(le);
1285:   COMPARISON_POINTWISE(lt);
1286:   COMPARISON_POINTWISE(ne);
1287: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值。关键符号：无明显局部符号。

### Lines 1288-1291 / 第 1288-1291 行

```cpp
1288: #undef COMPARISON_POINTWISE
1289: }
1290: 
1291: } // namespace at
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **Shared ATen infrastructure** — 共享 ATen 基础设施
- **Tensor metadata and value flow** — 张量元数据与数值流
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Linear algebra backend integration** — 线性代数后端集成
- **Memory allocation strategy** — 内存分配策略
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `torch/library.h`, `ATen/LegacyVmapTransforms.h`, `ATen/LegacyBatchedFallback.h`, `ATen/RedispatchFunctions.h`, `ATen/native/ResizeCommon.h`, `c10/util/irange.h`
- **External includes / 外部头文件**: `utility`
- **Namespaces / 命名空间**: `at`
- **Representative symbols / 代表性符号**: `to_type`, `TensorTensorScalarType`, `TensorTensorType`, `TensorScalarType`, `Binop`, `Unop`, `is_allowed_dim_on_scalar_tensor`, `sum_batching_rule`, `isPhysicalScalarTensor`, `binary_pointwise_batching_rule`, `expand_batching_rule`, `view_shape`, `...`
