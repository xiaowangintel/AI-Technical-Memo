# Embedding.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Embedding.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Embedding.
- **Purpose (CN)**: 实现或声明与 嵌入 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/core/List.h>
0004: #include <ATen/Dispatch.h>
0005: #include <ATen/Parallel.h>
0006: #include <ATen/TensorIterator.h>
0007: #include <ATen/TensorOperators.h>
0008: #include <ATen/TensorUtils.h>
0009: #include <ATen/native/BinaryOps.h>
0010: 
0011: #ifndef AT_PER_OPERATOR_HEADERS
0012: #include <ATen/Functions.h>
0013: #include <ATen/NativeFunctions.h>
0014: #else
0015: #include <ATen/ops/_sparse_coo_tensor_unsafe.h>
0016: #include <ATen/ops/embedding_backward_native.h>
0017: #include <ATen/ops/embedding_dense_backward.h>
0018: #include <ATen/ops/embedding_dense_backward_native.h>
0019: #include <ATen/ops/embedding_native.h>
0020: #include <ATen/ops/embedding_renorm_native.h>
0021: #include <ATen/ops/embedding_sparse_backward.h>
0022: #include <ATen/ops/embedding_sparse_backward_native.h>
0023: #include <ATen/ops/empty.h>
0024: #include <ATen/ops/zeros.h>
0025: #endif
0026: 
0027: #include <c10/util/irange.h>
0028: 
0029: #include <cstring>
0030: #include <memory>
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <utility>
0032: #include <vector>
0033: 
0034: 
0035: namespace at::native {
0036: 
0037: Tensor embedding_symint(const Tensor & weight, const Tensor & indices,
0038:                         c10::SymInt padding_idx, bool scale_grad_by_freq, bool sparse) {
0039:   TORCH_CHECK(weight.dim() == 2,  "'weight' must be 2-D");
0040:   auto indices_arg = TensorArg(indices, "indices", 1);
0041:   checkScalarTypes("embedding", indices_arg, {kLong, kInt});
0042: 
0043:   // TODO: use tensor.index() after improving perf
0044:   if (indices.dim() == 1) {
0045:     return weight.index_select(0, indices);
0046:   }
0047: 
0048:   auto size = indices.sym_sizes().vec();
0049:   for (const auto& d : weight.sym_sizes().slice(1)) {
0050:     size.push_back(d);
0051:   }
0052: 
0053:   return weight.index_select(0, indices.reshape(-1)).view_symint(size);
0054: }
0055: 
0056: Tensor embedding_backward_symint(
0057:     const Tensor & grad, const Tensor & indices, c10::SymInt num_weights,
0058:     c10::SymInt padding_idx, bool scale_grad_by_freq, bool sparse) {
0059:   if (sparse) {
0060:     // TODO: if we teach sparse tensor how to propagate symints, the guard
```
- **EN**: Lines 31-60 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: embedding_symint, TORCH_CHECK, dim, TensorArg.
- **CN**: 第 31-60 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：embedding_symint, TORCH_CHECK, dim, TensorArg。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     // here is not strictly necessary.  However, we think it is fine as is
0062:     // because num weights is derived from a parameter and therefore
0063:     // typically not varying.
0064:     return at::embedding_sparse_backward(
0065:       grad, indices,
0066:       num_weights.guard_int(__FILE__, __LINE__),
0067:       padding_idx.guard_int(__FILE__, __LINE__),
0068:       scale_grad_by_freq);
0069:   } else {
0070:     return at::embedding_dense_backward_symint(
0071:       grad, indices, std::move(num_weights), padding_idx, scale_grad_by_freq);
0072:   }
0073: }
0074: 
0075: Tensor embedding_sparse_backward(
0076:     const Tensor & grad_, const Tensor & indices_, int64_t num_weights,
0077:     int64_t padding_idx, bool scale_grad_by_freq) {
0078: 
0079:   auto indices_arg = TensorArg(indices_, "indices", 2);
0080:   checkScalarTypes("embedding_backward", indices_arg, {kLong, kInt});
0081: 
0082:   // TODO: implement scale_grad_by_freq
0083:   if (scale_grad_by_freq) {
0084:     TORCH_CHECK(false,
0085:         "embedding_backward: scale_grad_by_freq not supported with sparse gradients");
0086:   }
0087: 
0088:   Tensor indices = indices_;
0089:   Tensor grad = grad_;
0090:   if (padding_idx != -1) {
```
- **EN**: Lines 61-90 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: embedding_sparse_backward, guard_int, embedding_dense_backward_symint, move.
- **CN**: 第 61-90 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：embedding_sparse_backward, guard_int, embedding_dense_backward_symint, move。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     c10::List<std::optional<Tensor>> c({indices != padding_idx});
0092:     indices = indices.index(c);
0093:     grad = grad.index(c);
0094:   }
0095: 
0096:   auto num_features = grad_.sym_size(-1);
0097:   auto weight_size = std::array<c10::SymInt, 2>{{ num_weights, num_features }};
0098:   auto dense_options = grad.options();
0099: 
0100:   // check if all our grad come from padding_idx
0101:   if (grad.sym_numel() == 0) {
0102:     return at::_sparse_coo_tensor_unsafe_symint(at::empty({1, 0}, indices_.options().dtype(kLong)),
0103:                                          at::empty_symint({c10::SymInt(0), std::move(num_features)}, dense_options),
0104:                                          weight_size);
0105:   }
0106: 
0107:   auto index = indices.reshape({1, -1});
0108:   auto values = grad.reshape_symint({c10::SymInt(-1), std::move(num_features)});
0109:   return at::_sparse_coo_tensor_unsafe_symint(index.to(kLong), values, weight_size);
0110: }
0111: 
0112: Tensor embedding_dense_backward_cpu(
0113:     const Tensor & grad_, const Tensor & indices, int64_t num_weights,
0114:     int64_t padding_idx, bool scale_grad_by_freq) {
0115: 
0116:   auto indices_arg = TensorArg(indices, "indices", 2);
0117:   checkScalarTypes("embedding_backward", indices_arg, {kLong, kInt});
0118: 
0119:   auto grad_weight = at::zeros({num_weights, grad_.size(-1)}, grad_.options());
0120:   auto indices_contig = indices.contiguous();
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, expressions/calls, return paths. Notable symbols: c, index, sym_size, options.
- **CN**: 第 91-120 行主要涉及变量/别名声明、表达式或调用、返回路径。 值得关注的符号包括：c, index, sym_size, options。

### Lines 121-150 / 第 121-150 行
```cpp
0121:   int64_t numel = indices.numel();
0122:   auto grad = grad_.contiguous().view({numel, grad_.size(-1)});
0123: 
0124:   auto add_iter = TensorIteratorConfig()
0125:     .add_output(grad_weight)
0126:     .add_input(grad_weight)
0127:     .add_const_input(grad)
0128:     .resize_outputs(false)
0129:     .declare_static_shape(grad.sizes(), /*squash_dims=*/0)
0130:     .build();
0131: 
0132:   const auto gW_data = reinterpret_cast<char*>(grad_weight.data_ptr());
0133:   const auto gO_data = reinterpret_cast<const char*>(grad.const_data_ptr());
0134:   const auto gW_stride = grad_weight.strides()[0] * grad_weight.element_size();
0135:   const auto gO_stride = grad.strides()[0] * grad.element_size();
0136: 
0137:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_dense_backward_cpu", [&] () {
0138:     auto indices_data = indices_contig.const_data_ptr<index_t>();
0139: 
0140:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0141:     std::unique_ptr<index_t[]> counts;
0142:     if (scale_grad_by_freq) {
0143:       counts.reset(new index_t[num_weights]);
0144:       for (const auto i : c10::irange(numel)) {
0145:         counts[indices_data[i]] = 0;
0146:       }
0147:       for (const auto i : c10::irange(numel)) {
0148:         counts[indices_data[i]]++;
0149:       }
0150:     }
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, function signatures/definitions, control-flow checks. Notable symbols: numel, contiguous, view, size.
- **CN**: 第 121-150 行主要涉及变量/别名声明、函数签名或实现、控制流逻辑。 值得关注的符号包括：numel, contiguous, view, size。

### Lines 151-180 / 第 151-180 行
```cpp
0151: 
0152:     auto parallel_section = [&](index_t start, index_t end) {
0153:       TensorIterator iter(add_iter);
0154:       for (const auto i : c10::irange(numel)) {
0155:         if (indices_data[i] != padding_idx) {
0156:           index_t k = indices_data[i];
0157:           if (k >= start && k < end) {
0158:             double scale = 1.0;
0159:             if (scale_grad_by_freq) {
0160:               // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
0161:               scale /= counts[k];
0162:             }
0163: 
0164:             // grad_weight[k].add_(grad[i], scale);
0165:             iter.unsafe_replace_operand(0, gW_data + k * gW_stride);
0166:             iter.unsafe_replace_operand(1, gW_data + k * gW_stride);
0167:             iter.unsafe_replace_operand(2, const_cast<char*>(gO_data + i * gO_stride));
0168:             add_stub(kCPU, iter, scale);
0169:           }
0170:         }
0171:       }
0172:     };
0173: 
0174:     at::parallel_for(0, num_weights, 1000, parallel_section);
0175: 
0176:   });
0177: 
0178:   return grad_weight;
0179: }
0180: 
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: iter, irange, NOLINTNEXTLINE, add_.
- **CN**: 第 151-180 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：iter, irange, NOLINTNEXTLINE, add_。

### Lines 181-210 / 第 181-210 行
```cpp
0181: Tensor & embedding_renorm_cpu_(
0182:     Tensor & self, const Tensor & indices, double max_norm, double norm_type) {
0183:   auto self_arg = TensorArg(self, "self", 1);
0184:   auto indices_arg = TensorArg(indices, "indices", 2);
0185:   checkDim("embedding_renorm_", self_arg, 2);
0186:   checkScalarTypes("embedding_renorm_", indices_arg, {kLong, kInt});
0187: 
0188:   auto indices_contig = indices.contiguous();
0189:   auto num_indices = indices.numel();
0190: 
0191:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_renorm_cpu_", [&]() {
0192:     auto data_ptr = indices_contig.const_data_ptr<index_t>();
0193:     auto sorted_indices = std::vector<index_t>(data_ptr, data_ptr + num_indices);
0194:     std::sort(sorted_indices.begin(), sorted_indices.end());
0195: 
0196:     // Note that we cannot use at::parallel_for here because we perform operations on
0197:     // Tensor inside the loop. See github.com/pytorch/pytorch/issues/28370 for more details.
0198:     for (const auto i : c10::irange(num_indices)) {
0199:       if (i > 0 && sorted_indices[i] == sorted_indices[i - 1]) {
0200:         continue;
0201:       }
0202:       auto row = self[sorted_indices[i]];
0203:       auto norm = row.norm(norm_type).item<double>();
0204:       if (norm > max_norm) {
0205:         auto scale = max_norm / (norm + 1e-7);
0206:         row *= scale;
0207:       }
0208:     }
0209:   });
0210: 
```
- **EN**: Lines 181-210 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: embedding_renorm_cpu_, TensorArg, checkDim, checkScalarTypes.
- **CN**: 第 181-210 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：embedding_renorm_cpu_, TensorArg, checkDim, checkScalarTypes。

### Lines 211-215 / 第 211-215 行
```cpp
0211:   return self;
0212: }
0213: 
0214: 
0215: }  // namespace at::native
```
- **EN**: Lines 211-215 mainly cover expressions/calls, return paths.
- **CN**: 第 211-215 行主要涉及表达式或调用、返回路径。

## Key Concepts / 关键概念
- **EN**: TensorIterator-driven traversal  
  **CN**: 基于 TensorIterator 的遍历
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/core/List.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorIterator.h>`, `<ATen/TensorOperators.h>`, `<ATen/TensorUtils.h>`, `<ATen/native/BinaryOps.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
