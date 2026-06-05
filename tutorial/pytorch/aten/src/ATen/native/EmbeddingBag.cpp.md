# EmbeddingBag.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/EmbeddingBag.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Embedding Bag.
- **Purpose (CN)**: 实现或声明与 嵌入、bag 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/Dispatch.h>
0003: #include <ATen/Parallel.h>
0004: #include <ATen/TensorOperators.h>
0005: #include <ATen/TensorSubclassLikeUtils.h>
0006: #include <ATen/TensorUtils.h>
0007: #include <ATen/cpu/vec/vec.h>
0008: #include <ATen/native/EmbeddingBag.h>
0009: 
0010: #include <ATen/native/CPUBlas.h>
0011: #include <ATen/native/NonSymbolicBC.h>
0012: 
0013: #include <c10/util/irange.h>
0014: #include <c10/util/Half.h>
0015: 
0016: #ifdef USE_FBGEMM
0017: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wextra-semi")
0018: #include <fbgemm/Fbgemm.h>
0019: #include <fbgemm/FbgemmConvert.h>
0020: C10_DIAGNOSTIC_POP()
0021: #else
0022: #include <caffe2/perfkernels/embedding_lookup_idx.h>
0023: #endif
0024: 
0025: #include <cstring>
0026: #include <tuple>
0027: #include <utility>
0028: #include <vector>
0029: 
0030: #ifndef AT_PER_OPERATOR_HEADERS
```
- **EN**: Lines 1-30 mainly cover header inclusion, conditional compilation, macro-based glue. Notable symbols: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED, C10_DIAGNOSTIC_POP.
- **CN**: 第 1-30 行主要涉及头文件包含、预处理条件、宏定义或宏调用。 值得关注的符号包括：C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED, C10_DIAGNOSTIC_POP。

### Lines 31-60 / 第 31-60 行
```cpp
0031: #include <ATen/Functions.h>
0032: #include <ATen/NativeFunctions.h>
0033: #else
0034: #include <ATen/ops/_embedding_bag.h>
0035: #include <ATen/ops/_embedding_bag_backward_native.h>
0036: #include <ATen/ops/_embedding_bag_dense_backward.h>
0037: #include <ATen/ops/_embedding_bag_dense_backward_native.h>
0038: #include <ATen/ops/_embedding_bag_forward_only.h>
0039: #include <ATen/ops/_embedding_bag_forward_only_native.h>
0040: #include <ATen/ops/_embedding_bag_native.h>
0041: #include <ATen/ops/_embedding_bag_per_sample_weights_backward_native.h>
0042: #include <ATen/ops/_embedding_bag_sparse_backward.h>
0043: #include <ATen/ops/_embedding_bag_sparse_backward_native.h>
0044: #include <ATen/ops/embedding_backward_native.h>
0045: #include <ATen/ops/embedding_bag_native.h>
0046: #include <ATen/ops/empty.h>
0047: #include <ATen/ops/max.h>
0048: #include <ATen/ops/ones_like.h>
0049: #include <ATen/ops/resize_native.h>
0050: #include <ATen/ops/zero_native.h>
0051: #include <ATen/ops/zeros.h>
0052: #endif
0053: 
0054: namespace at::native {
0055: 
0056: template<typename scalar_t>
0057: scalar_t dot_impl(int64_t n, const scalar_t *x, int64_t incx, const scalar_t *y, int64_t incy);
0058: 
0059: static void make_offset2bag(const Tensor &offsets, Tensor& offset2bag) {
0060:   offset2bag.index_add_(
```
- **EN**: Lines 31-60 mainly cover header inclusion, conditional compilation, state/variable declarations. Notable symbols: dot_impl, make_offset2bag, index_add_.
- **CN**: 第 31-60 行主要涉及头文件包含、预处理条件、变量/别名声明。 值得关注的符号包括：dot_impl, make_offset2bag, index_add_。

### Lines 61-90 / 第 61-90 行
```cpp
0061:       0, offsets, at::ones_like(offsets, LEGACY_CONTIGUOUS_MEMORY_FORMAT)); // offset2bag = [1 0 1 0 1]
0062:   offset2bag[0] -= 1;                     // offset2bag = [0 0 1 0 1]
0063:   offset2bag = offset2bag.cumsum(0, offset2bag.scalar_type());     // offset2bag = [0 0 1 1 2]
0064: }
0065: 
0066: namespace {
0067: 
0068: std::pair<c10::MaybeOwned<Tensor>, c10::MaybeOwned<Tensor>> promoteIndicesAndOffsets(
0069:     const Tensor& indices,
0070:     const Tensor& offsets) {
0071:   const auto commonType =
0072:       promoteTypes(offsets.scalar_type(), indices.scalar_type());
0073:   return {
0074:       indices.scalar_type() == commonType ? c10::MaybeOwned<Tensor>::borrowed(indices)
0075:                                           : c10::MaybeOwned<Tensor>::owned(indices.toType(commonType)),
0076:       offsets.scalar_type() == commonType ? c10::MaybeOwned<Tensor>::borrowed(offsets)
0077:                                           : c10::MaybeOwned<Tensor>::owned(offsets.toType(commonType))};
0078: }
0079: 
0080: // Determines if we can use a fast implementation for index_select_add, which
0081: // is only applicable if special conditions are met
0082: template<typename index_t>
0083: bool is_fast_path_index_select(const Tensor& src, Tensor& output, index_t padding_idx) {
0084:   return (src.scalar_type() == kFloat || src.scalar_type() == kHalf ||
0085:           src.scalar_type() == kBFloat16) &&
0086:       src.strides()[1] == 1 && output.strides()[1] == 1 &&
0087:       padding_idx < static_cast<index_t>(0);
0088: }
0089: 
0090: // Determines if we can use a fast implementation for index_select_scale_add,
```
- **EN**: Lines 61-90 mainly cover function signatures/definitions, expressions/calls, state/variable declarations. Notable symbols: ones_like, cumsum, scalar_type, promoteIndicesAndOffsets.
- **CN**: 第 61-90 行主要涉及函数签名或实现、表达式或调用、变量/别名声明。 值得关注的符号包括：ones_like, cumsum, scalar_type, promoteIndicesAndOffsets。

### Lines 91-120 / 第 91-120 行
```cpp
0091: // which is only applicable if special conditions are met
0092: template<typename index_t>
0093: bool is_fast_path_index_select_scale(const Tensor& src, const Tensor& scale, Tensor& output, index_t padding_idx) {
0094:   return (src.scalar_type() == kFloat || src.scalar_type() == kHalf ||
0095:           src.scalar_type() == kBFloat16) &&
0096:       src.strides()[1] == 1 && output.strides()[1] == 1 &&
0097:       scale.strides()[0] == 1 && padding_idx < static_cast<index_t>(0);
0098: }
0099: 
0100: template<typename index_t>
0101: bool is_fast_path(const Tensor& src, const std::optional<Tensor>& scale, Tensor& output, index_t padding_idx) {
0102:   return (scale.has_value() && scale.value().defined()) ?
0103:          is_fast_path_index_select_scale(src, scale.value(), output, padding_idx) :
0104:          is_fast_path_index_select(src, output, padding_idx);
0105: }
0106: 
0107: // This function combines index_select (using select_indices as the index) and
0108: // index_add (using add_indices as the index), without creating an intermediary
0109: // tensor to hold the selected embeddings
0110: template <typename data_t, typename index_t>
0111: std::enable_if_t<std::is_same_v<data_t, double>, void>
0112: index_select_add(
0113:     const Tensor& select_indices,
0114:     const Tensor& add_indices,
0115:     const Tensor& src,
0116:     Tensor& output,
0117:     [[maybe_unused]] const Tensor& offsets,
0118:     [[maybe_unused]] bool include_last_offset,
0119:     Tensor& bag_size,
0120:     index_t padding_idx,
```
- **EN**: Lines 91-120 mainly cover expressions/calls, function signatures/definitions, comments/documentation. Notable symbols: is_fast_path_index_select_scale, scalar_type, strides, is_fast_path.
- **CN**: 第 91-120 行主要涉及表达式或调用、函数签名或实现、注释或说明。 值得关注的符号包括：is_fast_path_index_select_scale, scalar_type, strides, is_fast_path。

### Lines 121-150 / 第 121-150 行
```cpp
0121:     [[maybe_unused]] _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0122:   TORCH_CHECK(select_indices.numel() == add_indices.numel());
0123:   auto* add_indices_data = add_indices.const_data_ptr<index_t>();
0124:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0125:   auto* src_data = src.const_data_ptr<data_t>();
0126:   auto* output_data = output.data_ptr<data_t>();
0127:   index_t* bag_size_data = nullptr;
0128:   if (bag_size.defined()) {
0129:     bag_size_data = bag_size.data_ptr<index_t>();
0130:   }
0131:   auto numel = add_indices.numel();
0132:   int64_t ddim = src.size(1);
0133:   auto vocab_size = src.size(0);
0134:   auto src_stride0 = src.strides()[0];
0135:   auto src_stride1 = src.strides()[1];
0136:   auto output_stride0 = output.strides()[0];
0137:   auto output_stride1 = output.strides()[1];
0138: 
0139:   for (const auto i : c10::irange(numel)) {
0140:     // We can skip indices equal to padding_idx so they are not included in
0141:     // the reduction
0142:     auto idx = select_indices_data[i];
0143:     TORCH_CHECK(
0144:         idx >= 0 && idx < vocab_size,
0145:         "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0146:         idx);
0147:     if (idx != padding_idx) {
0148:       at::native::cpublas::axpy<data_t>(ddim, 1,
0149:               src_data + src_stride0 * idx, src_stride1,
0150:               output_data + output_stride0 * add_indices_data[i], output_stride1);
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: TORCH_CHECK, numel, defined, size.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：TORCH_CHECK, numel, defined, size。

### Lines 151-180 / 第 151-180 行
```cpp
0151:     } else if (bag_size_data) {
0152:       // Decrement bag_size to reflect that the index is padded
0153:       bag_size_data[add_indices_data[i]]--;
0154:     }
0155:   }
0156: }
0157: 
0158: namespace {
0159: template <typename index_t>
0160: void fbgemm_spmdm_report_error_(
0161:     int64_t output_size,
0162:     int index_size,
0163:     int64_t N,
0164:     const index_t* offsets,
0165:     const index_t* indices) {
0166:   for (const auto m : c10::irange(output_size)) {
0167:     for (index_t i = offsets[m]; i < offsets[m + 1]; ++i) {
0168:       TORCH_CHECK(i < index_size);
0169:       index_t idx = indices[i];
0170:       TORCH_CHECK(
0171:           0 <= idx && idx < N,
0172:           "Index ",
0173:           i,
0174:           " of input takes value ",
0175:           idx,
0176:           " which is not in the valid range [0, ",
0177:           N,
0178:           ")");
0179:     }
0180:   }
```
- **EN**: Lines 151-180 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: fbgemm_spmdm_report_error_, irange, TORCH_CHECK.
- **CN**: 第 151-180 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：fbgemm_spmdm_report_error_, irange, TORCH_CHECK。

### Lines 181-210 / 第 181-210 行
```cpp
0181:   TORCH_CHECK(
0182:       offsets[output_size] == index_size,
0183:       "Your input appears to be incorrect: the last offset value should be "
0184:        "the size of the indices tensor, but it seems not to be the case.");
0185: }
0186: } // namespace
0187: 
0188: template <typename data_t, typename index_t>
0189: std::enable_if_t<
0190:     std::is_same_v<data_t, at::Half> || std::is_same_v<data_t, at::BFloat16>,
0191:     void>
0192: index_select_add(
0193:     const Tensor& select_indices,
0194:     const Tensor& add_indices,
0195:     const Tensor& src,
0196:     Tensor& output,
0197:     const Tensor& offsets,
0198:     bool include_last_offset,
0199:     Tensor& bag_size,
0200:     index_t padding_idx,
0201:     _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0202:   int64_t ddim = src.size(1);
0203:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0204:   auto* output_data = output.data_ptr<data_t>();
0205: 
0206:   if (is_fast_path_index_select(src, output, padding_idx)) {
0207:     auto src_contig = src.contiguous();
0208:     auto* src_data = src_contig.const_data_ptr<data_t>();
0209:     int64_t output_size = offsets.numel() - 1;
0210:     auto* offsets_data = offsets.const_data_ptr<index_t>();
```
- **EN**: Lines 181-210 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TORCH_CHECK, index_select_add, size, is_fast_path_index_select.
- **CN**: 第 181-210 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, index_select_add, size, is_fast_path_index_select。

### Lines 211-240 / 第 211-240 行
```cpp
0211:     std::vector<index_t> offsets_include_last;
0212: 
0213:     if (include_last_offset) {
0214:       output_size = offsets.numel() - 1;
0215:     } else {
0216:       output_size = offsets.numel();
0217:       offsets_include_last.resize(offsets.numel() + 1);
0218:       if (offsets.numel() > 0) {
0219:         std::memcpy(
0220:             offsets_include_last.data(),
0221:             offsets.const_data_ptr<index_t>(),
0222:             sizeof(index_t) * offsets.numel());
0223:       }
0224:       offsets_include_last[offsets.numel()] = select_indices.numel();
0225:       offsets_data = offsets_include_last.data();
0226:     }
0227: #if defined(USE_FBGEMM)
0228:     constexpr bool isbf16 = std::is_same_v<data_t, at::Half> ? false : true;
0229:     auto kernel_16bit_index_t = fbgemm_kernel_cache
0230:         ? fbgemm_kernel_cache
0231:               ->getCallback</* has_weight */ false, index_t, uint16_t>(ddim)
0232:         : fbgemm::GenerateEmbeddingSpMDM<uint16_t, index_t, index_t, uint16_t>(
0233:               /* block_size */ ddim,
0234:               /* has_weight */ false,
0235:               /* normalize_by_lengths */ false,
0236:               /* prefetch */ 16,
0237:               /* is_weight_positional */ false,
0238:               /* use_offsets */ true,
0239:               /* is_bf16_out */ isbf16,
0240:               /* is_bf16_in */ isbf16);
```
- **EN**: Lines 211-240 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: numel, resize, memcpy, data.
- **CN**: 第 211-240 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：numel, resize, memcpy, data。

### Lines 241-270 / 第 241-270 行
```cpp
0241:     at::parallel_for(
0242:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
0243:           bool success = kernel_16bit_index_t(
0244:               /* output_size */ end_idx - start_idx,
0245:               /* index_size */ offsets_data[end_idx] - offsets_data[start_idx],
0246:               /* data_size */ src.size(0),
0247:               /* input */ reinterpret_cast<const uint16_t*>(src_data),
0248:               /* indices */ select_indices_data + offsets_data[start_idx],
0249:               /* offsets_or_lengths */ offsets_data + start_idx,
0250:               /* weights */ nullptr,
0251:               /* output */
0252:               reinterpret_cast<uint16_t*>(output_data + start_idx * ddim));
0253:           if (!success) {
0254:             fbgemm_spmdm_report_error_(
0255:                 end_idx - start_idx,
0256:                 offsets_data[end_idx] - offsets_data[start_idx],
0257:                 src.size(0),
0258:                 offsets_data + start_idx,
0259:                 select_indices_data + offsets_data[start_idx]);
0260:           }
0261:         });
0262: #else
0263:     // Initialize the intermediate output buffer to be 0.
0264:     Tensor output_fp32 = at::zeros({output_size, ddim}, output.options().dtype(at::kFloat));
0265:     auto* output_data_fp32 = output_fp32.data_ptr<float>();
0266:     using bVec = vec::Vectorized<BFloat16>;
0267:     using fVec = vec::Vectorized<float>;
0268:     at::parallel_for(
0269:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
0270:           caffe2::EmbeddingLookupIdx(
```
- **EN**: Lines 241-270 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: parallel_for, kernel_16bit_index_t, size, fbgemm_spmdm_report_error_.
- **CN**: 第 241-270 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：parallel_for, kernel_16bit_index_t, size, fbgemm_spmdm_report_error_。

### Lines 271-300 / 第 271-300 行
```cpp
0271:               /*block_size=*/ddim,
0272:               /*output_size=*/end_idx - start_idx,
0273:               /*index_size=*/offsets_data[end_idx] - offsets_data[start_idx],
0274:               /*data_size=*/src.size(0),
0275:               /*input=*/src_data,
0276:               /*indices=*/select_indices_data + offsets_data[start_idx],
0277:               /*offsets=*/offsets_data + start_idx,
0278:               /*weights=*/nullptr,
0279:               /*scale_bias=*/nullptr,
0280:               /*normalize_by_lengths=*/false,
0281:               /*out=*/output_data_fp32 + start_idx * ddim);
0282:           for (int64_t i = start_idx; i < end_idx; i++) {
0283:             // Convert FP32 intermediate buffer result back to 16 bit for
0284:             // output dtype
0285:             if constexpr (std::is_same_v<data_t, at::Half>) {
0286:               // FP16
0287:               for (const auto d : c10::irange(ddim)) {
0288:                 (output_data + i * ddim)[d] =
0289:                     static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0290:               }
0291:             } else {
0292:               // BF16
0293:               int64_t d = 0;
0294:               for (; d < ddim - (ddim % bVec::size()); d += bVec::size()) {
0295:                 fVec temp_fp32_0 = fVec::loadu(output_data_fp32 + ddim * i + d);
0296:                 fVec temp_fp32_1 =
0297:                     fVec::loadu(output_data_fp32 + ddim * i + d + fVec::size());
0298:                 convert_float_bfloat16(temp_fp32_0, temp_fp32_1)
0299:                     .store(output_data + i * ddim + d);
0300:               }
```
- **EN**: Lines 271-300 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: size, constexpr, irange, loadu.
- **CN**: 第 271-300 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：size, constexpr, irange, loadu。

### Lines 301-330 / 第 301-330 行
```cpp
0301:               for (; d < ddim; d++) {
0302:                 (output_data + i * ddim)[d] =
0303:                     static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0304:               }
0305:             }
0306:           }
0307:         });
0308: #endif
0309:   } else {
0310:     TORCH_CHECK(select_indices.numel() == add_indices.numel());
0311:     auto* src_data = src.const_data_ptr<data_t>();
0312:     auto* add_indices_data = add_indices.const_data_ptr<index_t>();
0313:     index_t* bag_size_data = nullptr;
0314:     if (bag_size.defined()) {
0315:       bag_size_data = bag_size.data_ptr<index_t>();
0316:     }
0317:     auto vocab_size = src.size(0);
0318:     auto src_stride0 = src.strides()[0];
0319:     auto src_stride1 = src.strides()[1];
0320:     auto output_stride0 = output.strides()[0];
0321:     auto output_stride1 = output.strides()[1];
0322:     auto numel = add_indices.numel();
0323: 
0324:     Tensor src_fp32 = at::empty({ddim}, src.options().dtype(at::kFloat));
0325:     auto* src_data_fp32 = src_fp32.mutable_data_ptr<float>();
0326: 
0327:     // Initialize the intermediate output buffer to be 0.
0328:     Tensor output_fp32 =
0329:         at::zeros({output.size(0), ddim}, output.options().dtype(at::kFloat));
0330:     auto* output_data_fp32 = output_fp32.data_ptr<float>();
```
- **EN**: Lines 301-330 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: TORCH_CHECK, numel, defined, size.
- **CN**: 第 301-330 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：TORCH_CHECK, numel, defined, size。

### Lines 331-360 / 第 331-360 行
```cpp
0331: 
0332:     for (const auto i : c10::irange(numel)) {
0333:       // We can skip indices equal to padding_idx so they are not included in
0334:       // the reduction
0335:       auto idx = select_indices_data[i];
0336:       TORCH_CHECK(
0337:           idx >= 0 && idx < vocab_size,
0338:           "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0339:           idx);
0340:       if (idx != padding_idx) {
0341:         // Copy src_data + src_stride0 * idx to src_data_fp32
0342:         for (const auto d : c10::irange(ddim)) {
0343:           src_data_fp32[d] = static_cast<float>(
0344:               (src_data + src_stride0 * idx)[d * src_stride1]);
0345:         }
0346:         at::native::cpublas::axpy<float>(
0347:             ddim,
0348:             1,
0349:             src_data_fp32,
0350:             1,
0351:             output_data_fp32 + ddim * add_indices_data[i],
0352:             1);
0353: 
0354:       } else if (bag_size_data) {
0355:         // Decrement bag_size to reflect that the index is padded
0356:         bag_size_data[add_indices_data[i]]--;
0357:       }
0358:     }
0359:     for (const auto i : c10::irange(output.size(0))) {
0360:       // Convert FP32 intermediate buffer result back to 16 bit for output
```
- **EN**: Lines 331-360 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: irange, TORCH_CHECK, size.
- **CN**: 第 331-360 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：irange, TORCH_CHECK, size。

### Lines 361-390 / 第 361-390 行
```cpp
0361:       // dtype
0362:       for (const auto d : c10::irange(ddim)) {
0363:         (output_data + output_stride0 * i)[d * output_stride1] =
0364:             static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0365:       }
0366:     }
0367:   }
0368: }
0369: template<typename data_t, typename index_t>
0370: std::enable_if_t<std::is_same_v<data_t, float>, void>
0371: index_select_add(const Tensor &select_indices,
0372:                              const Tensor &add_indices,
0373:                              const Tensor &src,
0374:                              Tensor &output,
0375:                              const Tensor& offsets,
0376:                              bool include_last_offset,
0377:                              Tensor &bag_size,
0378:                              index_t padding_idx,
0379:                              _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0380:   int64_t ddim = src.size(1);
0381:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0382:   auto* output_data = output.data_ptr<float>();
0383: 
0384:   if (is_fast_path_index_select(src, output, padding_idx)) {
0385:     auto src_contig = src.contiguous();
0386:     auto* src_data = src_contig.const_data_ptr<float>();
0387:     int64_t output_size = offsets.numel() - 1;
0388:     auto* offsets_data = offsets.const_data_ptr<index_t>();
0389:     std::vector<index_t> offsets_include_last;
0390: 
```
- **EN**: Lines 361-390 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: irange, index_select_add, size, is_fast_path_index_select.
- **CN**: 第 361-390 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：irange, index_select_add, size, is_fast_path_index_select。

### Lines 391-420 / 第 391-420 行
```cpp
0391:     if (include_last_offset) {
0392:       output_size = offsets.numel() - 1;
0393:     } else {
0394:       output_size = offsets.numel();
0395:       offsets_include_last.resize(offsets.numel() + 1);
0396:       if (offsets.numel() > 0) {
0397:         std::memcpy(
0398:             offsets_include_last.data(),
0399:             offsets.const_data_ptr<index_t>(),
0400:             sizeof(index_t) * offsets.numel());
0401:       }
0402:       offsets_include_last[offsets.numel()] = select_indices.numel();
0403:       offsets_data = offsets_include_last.data();
0404:     }
0405: 
0406: #ifdef USE_FBGEMM
0407:     auto kernel_fp32_index_t =
0408:       fbgemm_kernel_cache ?
0409:       fbgemm_kernel_cache->getCallback</* has_weight */ false, index_t, float>(ddim) :
0410:       fbgemm::GenerateEmbeddingSpMDM<float, index_t, index_t>(
0411:         /* block_size */ddim,
0412:         /* has_weight */false,
0413:         /* normalize_by_lengths */false,
0414:         /* prefetch */16,
0415:         /* is_weight_positional */false,
0416:         /* use_offsets */true
0417:       );
0418: #endif
0419:     at::parallel_for(
0420:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
```
- **EN**: Lines 391-420 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: numel, resize, memcpy, data.
- **CN**: 第 391-420 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：numel, resize, memcpy, data。

### Lines 421-450 / 第 421-450 行
```cpp
0421: #ifdef USE_FBGEMM
0422:           bool success = kernel_fp32_index_t(
0423:             /* output_size */end_idx - start_idx,
0424:             /* index_size */offsets_data[end_idx] - offsets_data[start_idx],
0425:             /* data_size */src.size(0),
0426:             /* input */src_data,
0427:             /* indices */select_indices_data + offsets_data[start_idx],
0428:             /* offsets_or_lengths */offsets_data + start_idx,
0429:             /* weights */nullptr,
0430:             /* output */output_data + start_idx * ddim);
0431:           if (!success) {
0432:             fbgemm_spmdm_report_error_(
0433:                 end_idx - start_idx,
0434:                 offsets_data[end_idx] - offsets_data[start_idx],
0435:                 src.size(0),
0436:                 offsets_data + start_idx,
0437:                 select_indices_data + offsets_data[start_idx]);
0438:           }
0439: #else
0440:           caffe2::EmbeddingLookupIdx(
0441:               /*block_size=*/ddim,
0442:               /*output_size=*/end_idx - start_idx,
0443:               /*index_size=*/offsets_data[end_idx] - offsets_data[start_idx],
0444:               /*data_size=*/src.size(0),
0445:               /*input=*/src_data,
0446:               /*indices=*/select_indices_data + offsets_data[start_idx],
0447:               /*offsets=*/offsets_data + start_idx,
0448:               /*weights=*/nullptr,
0449:               /*scale_bias=*/nullptr,
0450:               /*normalize_by_lengths=*/false,
```
- **EN**: Lines 421-450 mainly cover comments/documentation, expressions/calls, conditional compilation. Notable symbols: kernel_fp32_index_t, size, fbgemm_spmdm_report_error_, EmbeddingLookupIdx.
- **CN**: 第 421-450 行主要涉及注释或说明、表达式或调用、预处理条件。 值得关注的符号包括：kernel_fp32_index_t, size, fbgemm_spmdm_report_error_, EmbeddingLookupIdx。

### Lines 451-480 / 第 451-480 行
```cpp
0451:               /*out=*/output_data + start_idx * ddim);
0452: #endif
0453:         });
0454:   } else {
0455:     AT_ASSERT(select_indices.numel() == add_indices.numel());
0456:     auto* src_data = src.const_data_ptr<float>();
0457:     auto* add_indices_data = add_indices.const_data_ptr<index_t>();
0458:     index_t* bag_size_data = nullptr;
0459:     if (bag_size.defined()) {
0460:       bag_size_data = bag_size.data_ptr<index_t>();
0461:     }
0462:     auto vocab_size = src.size(0);
0463:     auto src_stride0 = src.strides()[0];
0464:     auto src_stride1 = src.strides()[1];
0465:     auto output_stride0 = output.strides()[0];
0466:     auto output_stride1 = output.strides()[1];
0467:     auto numel = add_indices.numel();
0468:     for (const auto i : c10::irange(numel)) {
0469:       // We can skip indices equal to padding_idx so they are not included in
0470:       // the reduction
0471:       auto idx = select_indices_data[i];
0472:       TORCH_CHECK(
0473:           idx >= 0 && idx < vocab_size,
0474:           "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0475:           idx);
0476:       if (idx != padding_idx) {
0477:         at::native::cpublas::axpy<float>(
0478:             ddim,
0479:             1,
0480:             src_data + src_stride0 * idx,
```
- **EN**: Lines 451-480 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: AT_ASSERT, numel, defined, size.
- **CN**: 第 451-480 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：AT_ASSERT, numel, defined, size。

### Lines 481-510 / 第 481-510 行
```cpp
0481:             src_stride1,
0482:             output_data + output_stride0 * add_indices_data[i],
0483:             output_stride1);
0484:       } else if (bag_size_data) {
0485:         // Decrement bag_size to reflect that the index is padded
0486:         bag_size_data[add_indices_data[i]]--;
0487:       }
0488:     }
0489:   }
0490: }
0491: 
0492: // This function fuses the following three fns:
0493: // index_select (using select_indices as the index)
0494: // mul (scaling by per_sample_weights)
0495: // index_add (using add_indices as the index)
0496: template <typename data_t, typename index_t>
0497: std::enable_if_t<std::is_same_v<data_t, double>, void>
0498: index_select_scale_add(
0499:     const Tensor& select_indices,
0500:     const Tensor& add_indices,
0501:     const Tensor& scale,
0502:     const Tensor& src,
0503:     Tensor& output,
0504:     [[maybe_unused]] const Tensor& offsets,
0505:     [[maybe_unused]] bool include_last_offset,
0506:     Tensor& bag_size,
0507:     index_t padding_idx,
0508:     [[maybe_unused]] _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0509:   AT_ASSERT(select_indices.numel() == add_indices.numel());
0510:   auto* add_indices_data = add_indices.const_data_ptr<index_t>();
```
- **EN**: Lines 481-510 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: index_select, mul, index_add, index_select_scale_add.
- **CN**: 第 481-510 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：index_select, mul, index_add, index_select_scale_add。

### Lines 511-540 / 第 511-540 行
```cpp
0511:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0512:   auto* src_data = src.const_data_ptr<data_t>();
0513:   auto* output_data = output.data_ptr<data_t>();
0514:   index_t* bag_size_data = nullptr;
0515:   if (bag_size.defined()) {
0516:     bag_size_data = bag_size.data_ptr<index_t>();
0517:   }
0518:   auto numel = add_indices.numel();
0519:   int64_t ddim = src.size(1);
0520:   auto vocab_size = src.size(0);
0521:   auto src_stride0 = src.strides()[0];
0522:   auto src_stride1 = src.strides()[1];
0523:   auto output_stride0 = output.strides()[0];
0524:   auto output_stride1 = output.strides()[1];
0525: 
0526:   auto* scale_data = scale.const_data_ptr<data_t>();
0527:   auto scale_stride = scale.strides()[0];
0528: 
0529:   for (const auto i : c10::irange(numel)) {
0530:     // We can skip indices equal to padding_idx so they are not included in
0531:     // the reduction
0532:     auto idx = select_indices_data[i];
0533:     TORCH_CHECK(
0534:         idx >= 0 && idx < vocab_size,
0535:         "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0536:         idx);
0537:     if (idx != padding_idx) {
0538:       auto* src_base = src_data + src_stride0 * idx;
0539:       auto* output_base = output_data + output_stride0 * add_indices_data[i];
0540:       auto element_scale = scale_data[i * scale_stride];
```
- **EN**: Lines 511-540 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: defined, numel, size, strides.
- **CN**: 第 511-540 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：defined, numel, size, strides。

### Lines 541-570 / 第 541-570 行
```cpp
0541:       for (const auto j : c10::irange(ddim)) {
0542:         output_base[j * output_stride1] += src_base[j * src_stride1] * element_scale;
0543:       }
0544:     } else if (bag_size_data) {
0545:       // Decrement bag_size to reflect that the index is padded
0546:       bag_size_data[add_indices_data[i]]--;
0547:     }
0548:   }
0549: }
0550: 
0551: template <typename data_t, typename index_t>
0552: std::enable_if_t<
0553:     std::is_same_v<data_t, at::Half> || std::is_same_v<data_t, at::BFloat16>,
0554:     void>
0555: index_select_scale_add(
0556:     const Tensor& select_indices,
0557:     const Tensor& add_indices,
0558:     const Tensor& scale,
0559:     const Tensor& src,
0560:     Tensor& output,
0561:     const Tensor& offsets,
0562:     bool include_last_offset,
0563:     Tensor& bag_size,
0564:     index_t padding_idx,
0565:     _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0566:   int64_t ddim = src.size(1);
0567:   auto* scale_data = scale.const_data_ptr<data_t>();
0568:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0569:   auto* output_data = output.data_ptr<data_t>();
0570: 
```
- **EN**: Lines 541-570 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: irange, index_select_scale_add, size.
- **CN**: 第 541-570 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：irange, index_select_scale_add, size。

### Lines 571-600 / 第 571-600 行
```cpp
0571:   if (is_fast_path_index_select_scale(src, scale, output, padding_idx)) {
0572:     auto src_contig = src.contiguous();
0573:     auto* src_data = src_contig.const_data_ptr<data_t>();
0574:     int64_t output_size = offsets.numel() - 1;
0575:     auto* offsets_data = offsets.const_data_ptr<index_t>();
0576:     std::vector<index_t> offsets_include_last;
0577: 
0578:     if (include_last_offset) {
0579:       output_size = offsets.numel() - 1;
0580:     } else {
0581:       output_size = offsets.numel();
0582:       offsets_include_last.resize(offsets.numel() + 1);
0583:       std::memcpy(
0584:           offsets_include_last.data(),
0585:           offsets.const_data_ptr<index_t>(),
0586:           sizeof(index_t) * offsets.numel());
0587:       offsets_include_last[offsets.numel()] = select_indices.numel();
0588:       offsets_data = offsets_include_last.data();
0589:     }
0590: 
0591:     Tensor scale_fp32 = at::empty(scale.sizes(), scale.options().dtype(at::kFloat));
0592:     auto* scale_data_fp32 = scale_fp32.mutable_data_ptr<float>();
0593: 
0594: #if defined(USE_FBGEMM)
0595:     constexpr bool isbf16 = std::is_same_v<data_t, at::Half> ? false : true;
0596:     if constexpr (isbf16) {
0597:       fbgemm::Bfloat16ToFloat_simd(
0598:           reinterpret_cast<const fbgemm::bfloat16*>(scale_data),
0599:           scale_data_fp32,
0600:           scale_fp32.numel());
```
- **EN**: Lines 571-600 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: is_fast_path_index_select_scale, contiguous, numel, resize.
- **CN**: 第 571-600 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：is_fast_path_index_select_scale, contiguous, numel, resize。

### Lines 601-630 / 第 601-630 行
```cpp
0601:     } else {
0602:       fbgemm::Float16ToFloat_simd(
0603:           reinterpret_cast<const fbgemm::float16*>(scale_data),
0604:           scale_data_fp32,
0605:           scale_fp32.numel());
0606:     }
0607:     auto kernel_16bit_index_t = fbgemm_kernel_cache
0608:         ? fbgemm_kernel_cache
0609:               ->getCallback</* has_weight */ true, index_t, uint16_t>(ddim)
0610:         : fbgemm::GenerateEmbeddingSpMDM<uint16_t, index_t, index_t, uint16_t>(
0611:               /* block_size */ ddim,
0612:               /* has_weight */ true,
0613:               /* normalize_by_lengths */ false,
0614:               /* prefetch */ 16,
0615:               /* is_weight_positional */ false,
0616:               /* use_offsets */ true,
0617:               /* is_bf16_out */ isbf16,
0618:               /* is_bf16_in */ isbf16);
0619:     at::parallel_for(
0620:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
0621:           bool success = kernel_16bit_index_t(
0622:               /* output_size */ end_idx - start_idx,
0623:               /* index_size */ offsets_data[end_idx] - offsets_data[start_idx],
0624:               /* data_size */ src.size(0),
0625:               /* input */ reinterpret_cast<const uint16_t*>(src_data),
0626:               /* indices */ select_indices_data + offsets_data[start_idx],
0627:               /* offsets_or_lengths */ offsets_data + start_idx,
0628:               /* weights */ scale_data_fp32 + offsets_data[start_idx],
0629:               /* output */
0630:               reinterpret_cast<uint16_t*>(output_data + start_idx * ddim));
```
- **EN**: Lines 601-630 mainly cover comments/documentation, expressions/calls, function signatures/definitions. Notable symbols: Float16ToFloat_simd, numel, parallel_for, kernel_16bit_index_t.
- **CN**: 第 601-630 行主要涉及注释或说明、表达式或调用、函数签名或实现。 值得关注的符号包括：Float16ToFloat_simd, numel, parallel_for, kernel_16bit_index_t。

### Lines 631-660 / 第 631-660 行
```cpp
0631:           if (!success) {
0632:             fbgemm_spmdm_report_error_(
0633:                 end_idx - start_idx,
0634:                 offsets_data[end_idx] - offsets_data[start_idx],
0635:                 src.size(0),
0636:                 offsets_data + start_idx,
0637:                 select_indices_data + offsets_data[start_idx]);
0638:           }
0639:         });
0640: #else
0641:     // Initialize the intermediate output buffer to be 0.
0642:     Tensor output_fp32 =
0643:         at::zeros({output_size, ddim}, output.options().dtype(at::kFloat));
0644:     auto* output_data_fp32 = output_fp32.data_ptr<float>();
0645:     for (const auto i : c10::irange(scale.numel())) {
0646:       scale_data_fp32[i] = static_cast<float>(scale_data[i]);
0647:     }
0648:     using bVec = vec::Vectorized<BFloat16>;
0649:     using fVec = vec::Vectorized<float>;
0650:     at::parallel_for(
0651:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
0652:           caffe2::EmbeddingLookupIdx(
0653:               /*block_size=*/ddim,
0654:               /*output_size=*/end_idx - start_idx,
0655:               /*index_size=*/offsets_data[end_idx] - offsets_data[start_idx],
0656:               /*data_size=*/src.size(0),
0657:               /*input=*/src_data,
0658:               /*indices=*/select_indices_data + offsets_data[start_idx],
0659:               /*offsets=*/offsets_data + start_idx,
0660:               /*weights=*/scale_data_fp32 + offsets_data[start_idx],
```
- **EN**: Lines 631-660 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: fbgemm_spmdm_report_error_, size, zeros, options.
- **CN**: 第 631-660 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：fbgemm_spmdm_report_error_, size, zeros, options。

### Lines 661-690 / 第 661-690 行
```cpp
0661:               /*scale_bias=*/nullptr,
0662:               /*normalize_by_lengths=*/false,
0663:               /*out=*/output_data_fp32 + start_idx * ddim);
0664:           for (int64_t i = start_idx; i < end_idx; i++) {
0665:             // Convert FP32 intermediate buffer result back to 16 bit for
0666:             // output dtype
0667:             if constexpr (std::is_same_v<data_t, at::Half>) {
0668:               // FP16
0669:               for (const auto d : c10::irange(ddim)) {
0670:                 (output_data + i * ddim)[d] =
0671:                     static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0672:               }
0673:             } else {
0674:               // BF16
0675:               int64_t d = 0;
0676:               for (; d < ddim - (ddim % bVec::size()); d += bVec::size()) {
0677:                 fVec temp_fp32_0 = fVec::loadu(output_data_fp32 + ddim * i + d);
0678:                 fVec temp_fp32_1 =
0679:                     fVec::loadu(output_data_fp32 + ddim * i + d + fVec::size());
0680:                 convert_float_bfloat16(temp_fp32_0, temp_fp32_1)
0681:                     .store(output_data + i * ddim + d);
0682:               }
0683:               for (; d < ddim; d++) {
0684:                 (output_data + i * ddim)[d] =
0685:                     static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0686:               }
0687:             }
0688:           }
0689:         });
0690: #endif
```
- **EN**: Lines 661-690 mainly cover comments/documentation, state/variable declarations, expressions/calls. Notable symbols: constexpr, irange, size, loadu.
- **CN**: 第 661-690 行主要涉及注释或说明、变量/别名声明、表达式或调用。 值得关注的符号包括：constexpr, irange, size, loadu。

### Lines 691-720 / 第 691-720 行
```cpp
0691:   } else {
0692:     AT_ASSERT(select_indices.numel() == add_indices.numel());
0693:     auto* src_data = src.const_data_ptr<data_t>();
0694:     auto* add_indices_data = add_indices.const_data_ptr<index_t>();
0695:     index_t* bag_size_data = nullptr;
0696:     if (bag_size.defined()) {
0697:       bag_size_data = bag_size.data_ptr<index_t>();
0698:     }
0699:     auto vocab_size = src.size(0);
0700:     auto src_stride0 = src.strides()[0];
0701:     auto src_stride1 = src.strides()[1];
0702:     auto output_stride0 = output.strides()[0];
0703:     auto output_stride1 = output.strides()[1];
0704:     auto scale_stride = scale.strides()[0];
0705:     auto numel = add_indices.numel();
0706: 
0707:     // Initialize the intermediate output buffer to be 0.
0708:     Tensor output_fp32 =
0709:         at::zeros({output.size(0), ddim}, output.options().dtype(at::kFloat));
0710:     auto* output_data_fp32 = output_fp32.data_ptr<float>();
0711: 
0712:     for (const auto i : c10::irange(numel)) {
0713:       // We can skip indices equal to padding_idx so they are not included in
0714:       // the reduction
0715:       auto idx = select_indices_data[i];
0716:       TORCH_CHECK(
0717:           idx >= 0 && idx < vocab_size,
0718:           "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0719:           idx);
0720:       if (idx != padding_idx) {
```
- **EN**: Lines 691-720 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: AT_ASSERT, numel, defined, size.
- **CN**: 第 691-720 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：AT_ASSERT, numel, defined, size。

### Lines 721-750 / 第 721-750 行
```cpp
0721:         auto* src_base = src_data + src_stride0 * idx;
0722:         auto* output_base_fp32 = output_data_fp32 + ddim * add_indices_data[i];
0723:         auto element_scale = scale_data[i * scale_stride];
0724:         for (const auto j : c10::irange(ddim)) {
0725:           output_base_fp32[j] += static_cast<float>(src_base[j * src_stride1]) *
0726:               static_cast<float>(element_scale);
0727:         }
0728:       } else if (bag_size_data) {
0729:         // Decrement bag_size to reflect that the index is padded
0730:         bag_size_data[add_indices_data[i]]--;
0731:       }
0732:     }
0733:     for (const auto i : c10::irange(output.size(0))) {
0734:       // Convert FP32 intermediate buffer result back to 16 bit for output
0735:       // dtype
0736:       for (const auto d : c10::irange(ddim)) {
0737:         (output_data + output_stride0 * i)[d * output_stride1] =
0738:             static_cast<data_t>((output_data_fp32 + ddim * i)[d]);
0739:       }
0740:     }
0741:   }
0742: }
0743: template<typename data_t, typename index_t>
0744: std::enable_if_t<std::is_same_v<data_t, float>, void>
0745: index_select_scale_add(const Tensor &select_indices,
0746:                                           const Tensor &add_indices,
0747:                                           const Tensor &scale,
0748:                                           const Tensor &src,
0749:                                           Tensor &output,
0750:                                           const Tensor& offsets,
```
- **EN**: Lines 721-750 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: irange, size, index_select_scale_add.
- **CN**: 第 721-750 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：irange, size, index_select_scale_add。

### Lines 751-780 / 第 751-780 行
```cpp
0751:                                           bool include_last_offset,
0752:                                           Tensor &bag_size,
0753:                                           index_t padding_idx,
0754:                                           _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
0755:   int64_t ddim = src.size(1);
0756:   auto* scale_data = scale.const_data_ptr<float>();
0757:   auto* select_indices_data = select_indices.const_data_ptr<index_t>();
0758:   auto* output_data = output.data_ptr<float>();
0759: 
0760:   if (is_fast_path_index_select_scale(src, scale, output, padding_idx)) {
0761:     auto src_contig = src.contiguous();
0762:     auto* src_data = src_contig.const_data_ptr<float>();
0763:     int64_t output_size = offsets.numel() - 1;
0764:     auto* offsets_data = offsets.const_data_ptr<index_t>();
0765:     std::vector<index_t> offsets_include_last;
0766: 
0767:     if (include_last_offset) {
0768:       output_size = offsets.numel() - 1;
0769:     } else {
0770:       output_size = offsets.numel();
0771:       offsets_include_last.resize(offsets.numel() + 1);
0772:       std::memcpy(
0773:           offsets_include_last.data(),
0774:           offsets.const_data_ptr<index_t>(),
0775:           sizeof(index_t) * offsets.numel());
0776:       offsets_include_last[offsets.numel()] = select_indices.numel();
0777:       offsets_data = offsets_include_last.data();
0778:     }
0779: 
0780: #ifdef USE_FBGEMM
```
- **EN**: Lines 751-780 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: size, is_fast_path_index_select_scale, contiguous, numel.
- **CN**: 第 751-780 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：size, is_fast_path_index_select_scale, contiguous, numel。

### Lines 781-810 / 第 781-810 行
```cpp
0781:     auto kernel_fp32_index_t =
0782:       fbgemm_kernel_cache ?
0783:       fbgemm_kernel_cache->getCallback</* has_weight */ true, index_t, float>(ddim) :
0784:       fbgemm::GenerateEmbeddingSpMDM<float, index_t, index_t>(
0785:         /* block_size */ddim,
0786:         /* has_weight */true,
0787:         /* normalize_by_lengths */false,
0788:         /* prefetch */16,
0789:         /* is_weight_positional */false,
0790:         /* use_offsets */true
0791:       );
0792: #endif
0793:     at::parallel_for(
0794:         0, output_size, 1, [&](index_t start_idx, index_t end_idx) {
0795: #ifdef USE_FBGEMM
0796:           bool success = kernel_fp32_index_t(
0797:             /* output_size */end_idx - start_idx,
0798:             /* index_size */offsets_data[end_idx] - offsets_data[start_idx],
0799:             /* data_size */src.size(0),
0800:             /* input */src_data,
0801:             /* indices */select_indices_data + offsets_data[start_idx],
0802:             /* offsets_or_lengths */offsets_data + start_idx,
0803:             /* weights */scale_data + offsets_data[start_idx],
0804:             /* output */output_data + start_idx * ddim);
0805:           if (!success) {
0806:             fbgemm_spmdm_report_error_(
0807:                 end_idx - start_idx,
0808:                 offsets_data[end_idx] - offsets_data[start_idx],
0809:                 src.size(0),
0810:                 offsets_data + start_idx,
```
- **EN**: Lines 781-810 mainly cover comments/documentation, expressions/calls, function signatures/definitions. Notable symbols: parallel_for, kernel_fp32_index_t, size, fbgemm_spmdm_report_error_.
- **CN**: 第 781-810 行主要涉及注释或说明、表达式或调用、函数签名或实现。 值得关注的符号包括：parallel_for, kernel_fp32_index_t, size, fbgemm_spmdm_report_error_。

### Lines 811-840 / 第 811-840 行
```cpp
0811:                 select_indices_data + offsets_data[start_idx]);
0812:           }
0813: #else
0814:           caffe2::EmbeddingLookupIdx(
0815:               /*block_size=*/ddim,
0816:               /*output_size=*/end_idx - start_idx,
0817:               /*index_size=*/offsets_data[end_idx] - offsets_data[start_idx],
0818:               /*data_size=*/src.size(0),
0819:               /*input=*/src_data,
0820:               /*indices=*/select_indices_data + offsets_data[start_idx],
0821:               /*offsets=*/offsets_data + start_idx,
0822:               /*weights=*/scale_data + offsets_data[start_idx],
0823:               /*scale_bias=*/nullptr,
0824:               /*normalize_by_lengths=*/false,
0825:               /*out=*/output_data + start_idx * ddim);
0826: #endif
0827:         });
0828:   } else {
0829:     AT_ASSERT(select_indices.numel() == add_indices.numel());
0830:     auto* src_data = src.const_data_ptr<float>();
0831:     auto* add_indices_data = add_indices.const_data_ptr<index_t>();
0832:     index_t* bag_size_data = nullptr;
0833:     if (bag_size.defined()) {
0834:       bag_size_data = bag_size.data_ptr<index_t>();
0835:     }
0836:     auto vocab_size = src.size(0);
0837:     auto src_stride0 = src.strides()[0];
0838:     auto src_stride1 = src.strides()[1];
0839:     auto output_stride0 = output.strides()[0];
0840:     auto output_stride1 = output.strides()[1];
```
- **EN**: Lines 811-840 mainly cover state/variable declarations, comments/documentation, expressions/calls. Notable symbols: EmbeddingLookupIdx, size, AT_ASSERT, numel.
- **CN**: 第 811-840 行主要涉及变量/别名声明、注释或说明、表达式或调用。 值得关注的符号包括：EmbeddingLookupIdx, size, AT_ASSERT, numel。

### Lines 841-870 / 第 841-870 行
```cpp
0841:     auto scale_stride = scale.strides()[0];
0842:     auto numel = add_indices.numel();
0843: 
0844: 
0845:     for (const auto i : c10::irange(numel)) {
0846:       // We can skip indices equal to padding_idx so they are not included in
0847:       // the reduction
0848:       auto idx = select_indices_data[i];
0849:       TORCH_CHECK(
0850:           idx >= 0 && idx < vocab_size,
0851:           "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
0852:           idx);
0853:       if (idx != padding_idx) {
0854:         auto* src_base = src_data + src_stride0 * idx;
0855:         auto* output_base = output_data + output_stride0 * add_indices_data[i];
0856:         auto element_scale = scale_data[i * scale_stride];
0857:         for (const auto j : c10::irange(ddim)) {
0858:           output_base[j * output_stride1] += src_base[j * src_stride1] * element_scale;
0859:         }
0860:       } else if (bag_size_data) {
0861:         // Decrement bag_size to reflect that the index is padded
0862:         bag_size_data[add_indices_data[i]]--;
0863:       }
0864:     }
0865:   }
0866: }
0867: 
0868: }  // namespace
0869: 
0870: void check_arguments(
```
- **EN**: Lines 841-870 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: strides, numel, irange, TORCH_CHECK.
- **CN**: 第 841-870 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：strides, numel, irange, TORCH_CHECK。

### Lines 871-900 / 第 871-900 行
```cpp
0871:     const Tensor& weight,
0872:     const Tensor& indices,
0873:     const Tensor& offsets,
0874:     const int64_t mode,
0875:     const std::optional<Tensor>& per_sample_weights,
0876:     bool include_last_offset) {
0877:   auto indices_arg = TensorArg(indices, "indices", 1);
0878:   checkScalarTypes("embedding_bag", indices_arg, {kLong, kInt});
0879:   auto offsets_arg = TensorArg(offsets, "offsets", 1);
0880:   checkScalarTypes("embedding_bag", offsets_arg, {kLong, kInt});
0881:   checkSameType("embedding_bag", indices_arg, offsets_arg);
0882:   auto weight_arg = TensorArg(weight, "weight", 1);
0883:   checkScalarTypes(
0884:       "embedding_bag", weight_arg, {kHalf, kBFloat16, kFloat, kDouble});
0885: 
0886:   AT_DISPATCH_INDEX_TYPES(offsets.scalar_type(), "_embedding_bag_cpu_impl", [&]() {
0887:     if (offsets.size(0) > 0) {
0888:       index_t offset_0 = offsets.const_data_ptr<index_t>()[0];
0889:       index_t offset_n = offsets.const_data_ptr<index_t>()[offsets.size(0)-1];
0890:       TORCH_CHECK(offset_0 == 0, "offsets[0] has to be 0, i.e., the first sequence "
0891:                                 "in the mini-batch has to start from position 0. "
0892:                                 "However, got ", offsets[0]);
0893:       TORCH_CHECK(offset_n <= indices.size(0), "offsets[-1] can not "
0894:                   "be greater than input's length ", indices.size(0), " but got offsets[-1] of ",
0895:                   offset_n);
0896:     }
0897:   });
0898: 
0899:   if (per_sample_weights.has_value() && per_sample_weights.value().defined()) {
0900:     TORCH_CHECK(
```
- **EN**: Lines 871-900 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TensorArg, checkScalarTypes, checkSameType, AT_DISPATCH_INDEX_TYPES.
- **CN**: 第 871-900 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TensorArg, checkScalarTypes, checkSameType, AT_DISPATCH_INDEX_TYPES。

### Lines 901-930 / 第 901-930 行
```cpp
0901:         mode == EmbeddingBagMode::SUM,
0902:         "embedding_bag: per_sample_weights only supported with mode='sum'");
0903:     auto per_input_weights_arg = TensorArg(
0904:         per_sample_weights.value(),"per_sample_weights", 1);
0905:     checkSameType("embedding_bag", weight_arg, per_input_weights_arg);
0906:     TORCH_CHECK(per_sample_weights.value().dim() == 1);
0907:     TORCH_CHECK(per_sample_weights.value().numel() == indices.numel());
0908:   }
0909: 
0910:   if (include_last_offset) {
0911:     TORCH_CHECK(
0912:         offsets.size(0) >= 1,
0913:         "include_last_offset: number of offset should be at least 1");
0914:   }
0915: }
0916: 
0917: void make_bag_size_out(
0918:     Tensor& bag_size_out,
0919:     const Tensor& offsets,
0920:     const Tensor& indices,
0921:     const int64_t mode,
0922:     const bool include_last_offset,
0923:     const bool requires_grad) {
0924:   if (requires_grad || mode == EmbeddingBagMode::MEAN ||
0925:       mode == EmbeddingBagMode::MAX) {
0926:     auto num_bags = offsets.size(0) - (include_last_offset ? 1 : 0);
0927:     at::native::resize_(bag_size_out, {num_bags}, std::nullopt);
0928:     // Compute this for EmbeddingBagMode::MEAN and EmbeddingBagMode::MAX (latter
0929:     // needed for backwards)
0930:     if (num_bags != 1) {
```
- **EN**: Lines 901-930 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: TensorArg, value, checkSameType, TORCH_CHECK.
- **CN**: 第 901-930 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TensorArg, value, checkSameType, TORCH_CHECK。

### Lines 931-960 / 第 931-960 行
```cpp
0931:       bag_size_out.slice(0, 0, bag_size_out.size(0) - 1, 1) =
0932:           offsets.slice(0, 1, num_bags, 1) -
0933:           offsets.slice(0, 0, num_bags - 1, 1);
0934:     }
0935:     if (num_bags > 0) {
0936:       bag_size_out[-1] = indices.size(0) - offsets[num_bags - 1];
0937:     }
0938:   } else {
0939:     at::native::resize_(bag_size_out, offsets.sizes(), std::nullopt);
0940:   }
0941: }
0942: 
0943: void make_max_indices_out(
0944:     Tensor& max_indices_out,
0945:     const Tensor& weight,
0946:     [[maybe_unused]] const Tensor& indices,
0947:     const Tensor& offsets,
0948:     const Tensor& bag_size,
0949:     const int64_t mode,
0950:     bool include_last_offset) {
0951:   int64_t numBags = offsets.size(0);
0952:   if (mode == EmbeddingBagMode::MAX) {
0953:     if (include_last_offset) {
0954:       TORCH_CHECK(
0955:         numBags >= 1, "include_last_offset: numBags should be at least 1");
0956:       numBags -= 1;
0957:     }
0958:     at::native::resize_(max_indices_out, {numBags, weight.sizes()[1]}, std::nullopt);
0959:     at::native::zero_(max_indices_out);
0960:   } else {
```
- **EN**: Lines 931-960 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: slice, size, resize_, sizes.
- **CN**: 第 931-960 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：slice, size, resize_, sizes。

### Lines 961-990 / 第 961-990 行
```cpp
0961:     at::native::resize_(max_indices_out, bag_size.sizes(), std::nullopt);
0962:   }
0963: }
0964: 
0965: void make_offset2bag_out(
0966:     Tensor& offset2bag,
0967:     Tensor& output,
0968:     const Tensor& weight,
0969:     const Tensor& indices,
0970:     const Tensor& offsets,
0971:     const int64_t mode,
0972:     const std::optional<Tensor>& per_sample_weights,
0973:     const int64_t padding_idx) {
0974:   // To save compute, if we are going to go down the fast path case for the 'sum'
0975:   // mode, we skip calculating offset2bag, since it is not going to be used.
0976:   bool fast_path_sum = is_fast_path(weight, per_sample_weights, output, padding_idx);
0977: 
0978:   if (mode == EmbeddingBagMode::MEAN || mode == EmbeddingBagMode::MAX ||
0979:       !fast_path_sum) {
0980:     at::native::resize_(offset2bag, {indices.size(0) + 1}, std::nullopt);
0981:     at::native::zero_(offset2bag);
0982: 
0983:     int64_t offsets_size = offsets.size(0);
0984:     bool include_last_offset = (output.size(0) == offsets_size - 1);
0985:     // when include_last_offset is true, ignore the last index in offset.
0986:     // fix segfault when include_last_offset is true and offsets[-1] != indices.size(0)
0987:     // see https://github.com/pytorch/pytorch/issues/89677 for more details.
0988:     Tensor _offsets = offsets;
0989:     if (include_last_offset) {
0990:       _offsets = offsets.narrow(0, 0, offsets_size - 1);
```
- **EN**: Lines 961-990 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: resize_, sizes, make_offset2bag_out, is_fast_path.
- **CN**: 第 961-990 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：resize_, sizes, make_offset2bag_out, is_fast_path。

### Lines 991-1020 / 第 991-1020 行
```cpp
0991:     }
0992:     make_offset2bag(_offsets, offset2bag);
0993:     at::native::resize_(offset2bag, {indices.size(0)}, std::nullopt);
0994:     // only initialize output in slow path
0995:     at::native::zero_(output);
0996:   }
0997: }
0998: 
0999: static Tensor make_bag_size(
1000:     const Tensor& offsets,
1001:     const Tensor& indices,
1002:     const int64_t mode,
1003:     const bool include_last_offset,
1004:     const bool requires_grad) {
1005:   Tensor bag_size = at::empty(offsets.sizes(), offsets.options());
1006:   make_bag_size_out(bag_size, offsets, indices, mode, include_last_offset, requires_grad);
1007:   return bag_size;
1008: }
1009: 
1010: static Tensor make_max_indices(
1011:     const Tensor& weight,
1012:     const Tensor& indices,
1013:     const Tensor& offsets,
1014:     const Tensor& bag_size,
1015:     const int64_t mode,
1016:     bool include_last_offset) {
1017:   Tensor max_indices = at::empty(bag_size.sizes(), offsets.options());
1018:   make_max_indices_out(max_indices, weight, indices, offsets, bag_size, mode, include_last_offset);
1019:   return max_indices;
1020: }
```
- **EN**: Lines 991-1020 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: make_offset2bag, resize_, size, zero_.
- **CN**: 第 991-1020 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：make_offset2bag, resize_, size, zero_。

### Lines 1021-1050 / 第 1021-1050 行
```cpp
1021: 
1022: static Tensor make_offset2bag(
1023:     Tensor& output,
1024:     const Tensor& weight,
1025:     const Tensor& indices,
1026:     const Tensor& offsets,
1027:     const int64_t mode,
1028:     const std::optional<Tensor>& per_sample_weights,
1029:     const int64_t padding_idx) {
1030:   Tensor offset2bag = at::empty({0}, offsets.options());
1031:   make_offset2bag_out(offset2bag, output, weight, indices, offsets, mode, per_sample_weights, padding_idx);
1032:   return offset2bag;
1033: }
1034: 
1035: static Tensor apply_bag_size(
1036:     const int64_t mode,
1037:     Tensor &output,
1038:     const Tensor &bag_size) {
1039:   if (mode == EmbeddingBagMode::MEAN) {
1040:     auto bag_size_ = at::max(bag_size, at::ones_like(bag_size, LEGACY_CONTIGUOUS_MEMORY_FORMAT))
1041:                          .to(output.options())
1042:                          .unsqueeze(1)
1043:                          .expand_as(output);
1044:     output /= bag_size_;
1045:   }
1046:   return output;
1047: }
1048: 
1049: static Tensor apply_bag_size_backward(
1050:     const int64_t mode,
```
- **EN**: Lines 1021-1050 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: make_offset2bag, empty, options, make_offset2bag_out.
- **CN**: 第 1021-1050 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：make_offset2bag, empty, options, make_offset2bag_out。

### Lines 1051-1080 / 第 1051-1080 行
```cpp
1051:     Tensor &output,
1052:     const Tensor &offset2bag,
1053:     const Tensor &bag_size) {
1054:   if (mode == EmbeddingBagMode::MEAN) {
1055:     auto inv_bag_size_ = (1 / bag_size.to(output.options()))
1056:                            .unsqueeze(1)
1057:                            .index_select(0, offset2bag);
1058:     output *= inv_bag_size_;
1059:   }
1060:   return output;
1061: }
1062: 
1063: template <typename scalar_t>
1064: static void embedding_bag_cpu_max_out(
1065:     Tensor* max_indices,
1066:     const Tensor& weight,
1067:     const Tensor& indices,
1068:     const Tensor& offset2bag,
1069:     const Tensor& output,
1070:     [[maybe_unused]] bool include_last_offset,
1071:     Tensor& bag_size,
1072:     int64_t padding_idx) {
1073:   int64_t numIndices = indices.numel();
1074:   int64_t featureSize = weight.size(1);
1075:   int64_t vocab_size = weight.size(0);
1076:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_bag_cpu_max_out", [&] {
1077:     auto* indices_data = indices.const_data_ptr<index_t>();
1078:     auto* offset2bag_data = offset2bag.data_ptr<index_t>();
1079: 
1080:     index_t* max_indices_data = nullptr;
```
- **EN**: Lines 1051-1080 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: to, options, unsqueeze, index_select.
- **CN**: 第 1051-1080 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：to, options, unsqueeze, index_select。

### Lines 1081-1110 / 第 1081-1110 行
```cpp
1081:     int64_t max_indices_stride = 0;
1082:     if (max_indices) {
1083:       max_indices_data = max_indices->data_ptr<index_t>();
1084:       max_indices_stride = max_indices->strides()[0];
1085:     }
1086: 
1087:     auto* weight_data = weight.const_data_ptr<scalar_t>();
1088:     auto* output_data = output.data_ptr<scalar_t>();
1089:     auto* bag_size_data = bag_size.data_ptr<index_t>();
1090:     auto weight_stride0 = weight.strides()[0];
1091:     auto weight_stride1 = weight.strides()[1];
1092:     auto output_stride = output.strides()[0];
1093:     int64_t numBags = bag_size.size(0);
1094:     std::vector<bool> bag_empty(numBags, true);
1095: 
1096:     for (const auto i : c10::irange(numIndices)) {
1097:       auto bag = offset2bag_data[i];
1098:       auto word_idx = indices_data[i];
1099:       TORCH_CHECK(
1100:           word_idx >= 0 && word_idx < vocab_size,
1101:           "embedding_bag: Expected idx >= 0 && idx < num_embeddings but found idx to be ",
1102:           word_idx);
1103:       if (word_idx != static_cast<index_t>(padding_idx)) {
1104:         bool is_first_for_bag = bag_empty[bag];
1105:         for (const auto dim : c10::irange(featureSize)) {
1106:           auto& current_item = output_data[output_stride * bag + dim];
1107:           auto weight_item =
1108:               weight_data[weight_stride0 * word_idx + dim * weight_stride1];
1109: 
1110:           if (is_first_for_bag || (weight_item > current_item)) {
```
- **EN**: Lines 1081-1110 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: strides, size, bag_empty, irange.
- **CN**: 第 1081-1110 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：strides, size, bag_empty, irange。

### Lines 1111-1140 / 第 1111-1140 行
```cpp
1111:             current_item = weight_item;
1112:             if (max_indices_data) {
1113:               max_indices_data[max_indices_stride * bag + dim] = word_idx;
1114:             }
1115:           }
1116:         }
1117:         if (is_first_for_bag) {
1118:           bag_empty[bag] = false;
1119:         }
1120:       } else {
1121:         // Decrement bag_size to reflect that the index is padded
1122:         bag_size_data[bag]--;
1123:       }
1124:     }
1125:   });
1126: }
1127: 
1128: void _embedding_bag_cpu_impl_out(Tensor& output, Tensor& offset2bag,
1129:                             Tensor& bag_size, Tensor* max_indices,
1130:                             const Tensor &weight, const Tensor &indices,
1131:                             const Tensor &offsets, const int64_t mode,
1132:                             const std::optional<Tensor>& per_sample_weights,
1133:                             bool include_last_offset, int64_t padding_idx, _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
1134:   if (mode == EmbeddingBagMode::MEAN || mode == EmbeddingBagMode::SUM) {
1135:     AT_DISPATCH_FLOATING_TYPES_AND2(at::ScalarType::Half, at::ScalarType::BFloat16, weight.scalar_type(), "embedding_bag_no_grad_cpu_out",
1136:       [&indices, &offset2bag, &per_sample_weights, &weight, &output, &offsets, &include_last_offset, &mode, &bag_size, &padding_idx, &fbgemm_kernel_cache]() {
1137:       AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "embedding_bag_no_grad_cpu_out",
1138:         [&indices, &offset2bag, &per_sample_weights, &weight, &output, &offsets, &include_last_offset, &mode, &bag_size, &padding_idx, &fbgemm_kernel_cache]() {
1139:         if (per_sample_weights.has_value() && per_sample_weights.value().defined()) {
1140:           TORCH_INTERNAL_ASSERT(mode == EmbeddingBagMode::SUM);
```
- **EN**: Lines 1111-1140 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: _embedding_bag_cpu_impl_out, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, AT_DISPATCH_INDEX_TYPES.
- **CN**: 第 1111-1140 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：_embedding_bag_cpu_impl_out, AT_DISPATCH_FLOATING_TYPES_AND2, scalar_type, AT_DISPATCH_INDEX_TYPES。

### Lines 1141-1170 / 第 1141-1170 行
```cpp
1141:           index_select_scale_add<scalar_t, index_t>(
1142:             indices, offset2bag, per_sample_weights.value(), weight, output, offsets, include_last_offset, bag_size, padding_idx, fbgemm_kernel_cache);
1143:         } else {
1144:           index_select_add<scalar_t, index_t>(indices, offset2bag, weight, output, offsets, include_last_offset, bag_size, padding_idx, fbgemm_kernel_cache);
1145:         }
1146:       });
1147:     });
1148:     apply_bag_size(mode, output, bag_size);
1149:     if (mode == EmbeddingBagMode::SUM) {
1150:       // make bag_size output deterministic
1151:       at::native::zero_(bag_size);
1152:     }
1153:     if (max_indices) {
1154:       max_indices->copy_(bag_size);
1155:     }
1156:   } else { // EmbeddingBagMode::MAX
1157:     AT_DISPATCH_FLOATING_TYPES_AND2(
1158:         at::ScalarType::Half,
1159:         at::ScalarType::BFloat16,
1160:         weight.scalar_type(),
1161:         "embedding_bag_cpu_max_out",
1162:         [&]() {
1163:           embedding_bag_cpu_max_out<scalar_t>(
1164:               max_indices,
1165:               weight,
1166:               indices,
1167:               offset2bag,
1168:               output,
1169:               include_last_offset,
1170:               bag_size,
```
- **EN**: Lines 1141-1170 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: value, apply_bag_size, zero_, copy_.
- **CN**: 第 1141-1170 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：value, apply_bag_size, zero_, copy_。

### Lines 1171-1200 / 第 1171-1200 行
```cpp
1171:               padding_idx);
1172:         });
1173:   }
1174: }
1175: 
1176: // Assumes all input tensors except for `weight` are contiguous.
1177: // See NOTE [ embedding_bag Native Functions ] in native_functions.yaml for details
1178: static std::tuple<Tensor, Tensor, Tensor, Tensor> _embedding_bag_cpu_impl(
1179:     const Tensor& weight,
1180:     const Tensor& indices_,
1181:     const Tensor& offsets_,
1182:     const int64_t mode,
1183:     const Tensor& per_sample_weights,
1184:     bool include_last_offset,
1185:     int64_t padding_idx,
1186:     bool requires_grad) {
1187:   TORCH_CHECK(indices_.dim() == 1 || indices_.dim() == 2,
1188:       "input has to be a 1D or 2D Tensor, but got Tensor of dimension ",
1189:       indices_.dim());
1190:   if (indices_.dim() == 1) {
1191:     TORCH_CHECK(offsets_.dim() == 1,
1192:         "offsets has to be a 1D Tensor, but got Tensor of dimension ",
1193:         offsets_.dim());
1194:   }
1195:   TORCH_CHECK(weight.dim() == 2,
1196:       "weight has to be a 2D Tensor, but got Tensor of dimension ",
1197:       weight.dim());
1198:   auto [indicesMaybeOwned, offsetsMaybeOwned] = promoteIndicesAndOffsets(indices_, offsets_);
1199:   const auto& indices = *indicesMaybeOwned;
1200:   const auto& offsets = *offsetsMaybeOwned;
```
- **EN**: Lines 1171-1200 mainly cover expressions/calls, state/variable declarations, macro-based glue. Notable symbols: _embedding_bag_cpu_impl, TORCH_CHECK, dim, promoteIndicesAndOffsets.
- **CN**: 第 1171-1200 行主要涉及表达式或调用、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：_embedding_bag_cpu_impl, TORCH_CHECK, dim, promoteIndicesAndOffsets。

### Lines 1201-1230 / 第 1201-1230 行
```cpp
1201:   check_arguments(weight, indices, offsets, mode, per_sample_weights, include_last_offset);
1202: 
1203:   Tensor output = at::empty(
1204:       {include_last_offset ? offsets.size(0) - 1 : offsets.size(0),
1205:        weight.sizes()[1]},
1206:       weight.options());
1207: 
1208:   Tensor offset2bag = make_offset2bag(output, weight, indices, offsets, mode, per_sample_weights, padding_idx);
1209: 
1210:   Tensor bag_size = make_bag_size(offsets, indices, mode, include_last_offset, requires_grad);
1211: 
1212:   Tensor max_indices = make_max_indices(weight, indices, offsets, bag_size, mode, include_last_offset);
1213: 
1214:   _embedding_bag_cpu_impl_out(output, offset2bag,
1215:                           bag_size, &max_indices,
1216:                           weight, indices, offsets,
1217:                           mode, per_sample_weights,
1218:                           include_last_offset, padding_idx);
1219: 
1220:   return std::make_tuple(std::move(output), std::move(offset2bag), std::move(bag_size), std::move(max_indices));
1221: }
1222: 
1223: // embedding_bag wrapper to enforce contiguity in tensors other than `weight`.
1224: // This is created to save extra `.contiguous()` call in backward.
1225: // See NOTE [ embedding_bag Native Functions ] in native_functions.yaml for details
1226: std::tuple<Tensor, Tensor, Tensor, Tensor>
1227: embedding_bag(const Tensor &weight, const Tensor &indices,
1228:               const Tensor &offsets, const bool scale_grad_by_freq,
1229:               const int64_t mode, bool sparse, const std::optional<Tensor>& per_sample_weights_opt,
1230:               bool include_last_offset, std::optional<int64_t> padding_idx_opt) {
```
- **EN**: Lines 1201-1230 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: check_arguments, empty, size, sizes.
- **CN**: 第 1201-1230 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：check_arguments, empty, size, sizes。

### Lines 1231-1260 / 第 1231-1260 行
```cpp
1231:   // See [Note: hacky wrapper removal for optional tensor]
1232:   c10::MaybeOwned<Tensor> per_sample_weights_maybe_owned = at::borrow_from_optional_tensor(per_sample_weights_opt);
1233:   const Tensor& per_sample_weights = *per_sample_weights_maybe_owned;
1234:   int64_t padding_idx = -1;
1235: 
1236:   if (padding_idx_opt.has_value()) {
1237:     auto num_embeddings = weight.size(0);
1238:     padding_idx = padding_idx_opt.value();
1239:     TORCH_CHECK(
1240:       (padding_idx >= -num_embeddings) && (padding_idx < num_embeddings),
1241:       "padding_idx must be within the number of embeddings, -", num_embeddings,
1242:       " through ", num_embeddings - 1, ", but got ", padding_idx);
1243:     padding_idx = maybe_wrap_dim(padding_idx, weight.size(0));
1244:   }
1245:   std::tuple<Tensor, Tensor, Tensor, Tensor> out;
1246:   bool needs_grad_path = weight.requires_grad() ||
1247:                       weight._fw_grad(/*level=*/0).defined() ||
1248:                       (per_sample_weights_opt.has_value() &&
1249:                        per_sample_weights_opt.value().defined() &&
1250:                        per_sample_weights_opt.value().requires_grad());
1251: 
1252:   if (!needs_grad_path) {
1253:     out = at::_embedding_bag_forward_only(
1254:       weight, indices.contiguous(), offsets.contiguous(), scale_grad_by_freq,
1255:       mode, sparse, per_sample_weights, include_last_offset, padding_idx);
1256:   } else {
1257:     out = at::_embedding_bag(
1258:       weight, indices.contiguous(), offsets.contiguous(), scale_grad_by_freq,
1259:       mode, sparse, per_sample_weights, include_last_offset, padding_idx);
1260:   }
```
- **EN**: Lines 1231-1260 mainly cover state/variable declarations, function signatures/definitions, expressions/calls. Notable symbols: borrow_from_optional_tensor, has_value, size, value.
- **CN**: 第 1231-1260 行主要涉及变量/别名声明、函数签名或实现、表达式或调用。 值得关注的符号包括：borrow_from_optional_tensor, has_value, size, value。

### Lines 1261-1290 / 第 1261-1290 行
```cpp
1261:   return out;
1262: }
1263: 
1264: std::tuple<Tensor, Tensor, Tensor, Tensor>
1265: embedding_bag(const Tensor &weight, const Tensor &indices,
1266:               const Tensor &offsets, const bool scale_grad_by_freq,
1267:               const int64_t mode, bool sparse, const std::optional<Tensor>& per_sample_weights_opt,
1268:               bool include_last_offset) {
1269:   return at::native::embedding_bag(weight, indices, offsets, scale_grad_by_freq,
1270:       mode, sparse, per_sample_weights_opt, include_last_offset, std::nullopt);
1271: }
1272: 
1273: // Assumes all input tensors except for `weight` are contiguous.
1274: // See NOTE [ embedding_bag Native Functions ] in native_functions.yaml for details
1275: std::tuple<Tensor, Tensor, Tensor, Tensor>
1276: _embedding_bag_forward_only_cpu(const Tensor &weight, const Tensor &indices,
1277:                   const Tensor &offsets, const bool scale_grad_by_freq,
1278:                   const int64_t mode, bool sparse, const std::optional<Tensor>& per_sample_weights_opt, bool include_last_offset,
1279:                   int64_t padding_idx) {
1280:   // See [Note: hacky wrapper removal for optional tensor]
1281:   c10::MaybeOwned<Tensor> per_sample_weights_maybe_owned = at::borrow_from_optional_tensor(per_sample_weights_opt);
1282:   const Tensor& per_sample_weights = *per_sample_weights_maybe_owned;
1283:   std::ignore = scale_grad_by_freq;
1284:   std::ignore = sparse;
1285:   return _embedding_bag_cpu_impl(
1286:       weight,
1287:       indices,
1288:       offsets,
1289:       mode,
1290:       per_sample_weights,
```
- **EN**: Lines 1261-1290 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: embedding_bag, _embedding_bag_forward_only_cpu, borrow_from_optional_tensor, _embedding_bag_cpu_impl.
- **CN**: 第 1261-1290 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：embedding_bag, _embedding_bag_forward_only_cpu, borrow_from_optional_tensor, _embedding_bag_cpu_impl。

### Lines 1291-1320 / 第 1291-1320 行
```cpp
1291:       include_last_offset,
1292:       padding_idx,
1293:       /*requires_grad=*/false);
1294: }
1295: 
1296: // Assumes all input tensors except for `weight` are contiguous.
1297: // See NOTE [ embedding_bag Native Functions ] in native_functions.yaml for details
1298: std::tuple<Tensor, Tensor, Tensor, Tensor>
1299: _embedding_bag_cpu(const Tensor &weight, const Tensor &indices,
1300:                   const Tensor &offsets, const bool scale_grad_by_freq,
1301:                   const int64_t mode, bool sparse, const std::optional<Tensor>& per_sample_weights_opt, bool include_last_offset,
1302:                   int64_t padding_idx) {
1303:   // See [Note: hacky wrapper removal for optional tensor]
1304:   c10::MaybeOwned<Tensor> per_sample_weights_maybe_owned = at::borrow_from_optional_tensor(per_sample_weights_opt);
1305:   const Tensor& per_sample_weights = *per_sample_weights_maybe_owned;
1306: 
1307:   std::ignore = scale_grad_by_freq;
1308:   std::ignore = sparse;
1309:   return _embedding_bag_cpu_impl(
1310:       weight,
1311:       indices,
1312:       offsets,
1313:       mode,
1314:       per_sample_weights,
1315:       include_last_offset,
1316:       padding_idx,
1317:       /*requires_grad=*/true);
1318: }
1319: 
1320: void _embedding_bag_cpu_out(
```
- **EN**: Lines 1291-1320 mainly cover expressions/calls, comments/documentation, state/variable declarations. Notable symbols: _embedding_bag_cpu, borrow_from_optional_tensor, _embedding_bag_cpu_impl, _embedding_bag_cpu_out.
- **CN**: 第 1291-1320 行主要涉及表达式或调用、注释或说明、变量/别名声明。 值得关注的符号包括：_embedding_bag_cpu, borrow_from_optional_tensor, _embedding_bag_cpu_impl, _embedding_bag_cpu_out。

### Lines 1321-1350 / 第 1321-1350 行
```cpp
1321:     at::Tensor& output,
1322:     at::Tensor& offset2bag,
1323:     at::Tensor& bag_size,
1324:     at::Tensor* p_max_indices,
1325:     const at::Tensor& weight,
1326:     const at::Tensor& indices_,
1327:     const at::Tensor& offsets_,
1328:     [[maybe_unused]] const bool scale_grad_by_freq,
1329:     const int64_t mode,
1330:     [[maybe_unused]] const bool sparse,
1331:     const std::optional<at::Tensor>& per_sample_weights,
1332:     const bool include_last_offset,
1333:     const std::optional<int64_t>& padding_idx,
1334:     _EmbeddingBagKernelCache* fbgemm_kernel_cache) {
1335:   auto [indicesMaybeOwned, offsetsMaybeOwned] = promoteIndicesAndOffsets(indices_, offsets_);
1336:   const auto& indices = *indicesMaybeOwned;
1337:   const auto& offsets = *offsetsMaybeOwned;
1338:   at::native::check_arguments(
1339:       weight, indices, offsets, mode, per_sample_weights, include_last_offset);
1340: 
1341:   at::native::make_offset2bag_out(
1342:       offset2bag,
1343:       output,
1344:       weight,
1345:       indices,
1346:       offsets,
1347:       mode,
1348:       per_sample_weights,
1349:       padding_idx.value_or(-1));
1350: 
```
- **EN**: Lines 1321-1350 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: promoteIndicesAndOffsets, check_arguments, make_offset2bag_out, value_or.
- **CN**: 第 1321-1350 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：promoteIndicesAndOffsets, check_arguments, make_offset2bag_out, value_or。

### Lines 1351-1380 / 第 1351-1380 行
```cpp
1351:   at::native::make_bag_size_out(
1352:       bag_size, offsets, indices, mode, include_last_offset, false);
1353: 
1354:   if (p_max_indices) {
1355:     at::native::make_max_indices_out(
1356:         *p_max_indices,
1357:         weight,
1358:         indices,
1359:         offsets,
1360:         bag_size,
1361:         mode,
1362:         include_last_offset);
1363:   }
1364: 
1365:   at::native::_embedding_bag_cpu_impl_out(
1366:       output,
1367:       offset2bag,
1368:       bag_size,
1369:       p_max_indices,
1370:       weight,
1371:       indices,
1372:       offsets,
1373:       mode,
1374:       per_sample_weights,
1375:       include_last_offset,
1376:       padding_idx.value_or(-1),
1377:       fbgemm_kernel_cache);
1378: }
1379: 
1380: Tensor _embedding_bag_backward(const Tensor &grad, const Tensor &indices_,
```
- **EN**: Lines 1351-1380 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: make_bag_size_out, make_max_indices_out, _embedding_bag_cpu_impl_out, value_or.
- **CN**: 第 1351-1380 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：make_bag_size_out, make_max_indices_out, _embedding_bag_cpu_impl_out, value_or。

### Lines 1381-1410 / 第 1381-1410 行
```cpp
1381:                               const Tensor &offsets_,
1382:                               const Tensor &offset2bag,
1383:                               const Tensor &bag_size_,
1384:                               const Tensor &max_indices_,
1385:                               int64_t num_weights,
1386:                               bool scale_grad_by_freq, int64_t mode,
1387:                               bool sparse, const std::optional<Tensor>& per_sample_weights_opt,
1388:                               int64_t padding_idx) {
1389:     return at::native::_embedding_bag_backward_symint(
1390:         grad, indices_, offsets_, offset2bag, bag_size_, max_indices_, num_weights, scale_grad_by_freq, mode, sparse, per_sample_weights_opt, padding_idx);
1391: }
1392: 
1393: // Assumes all input tensors are contiguous.
1394: // See NOTE [ embedding_bag Native Functions ] in native_functions.yaml for details
1395: Tensor _embedding_bag_backward_symint(const Tensor &grad, const Tensor &indices_,
1396:                               const Tensor &offsets_,
1397:                               const Tensor &offset2bag,
1398:                               const Tensor &bag_size_,
1399:                               const Tensor &max_indices_,
1400:                               c10::SymInt num_weights,
1401:                               bool scale_grad_by_freq, int64_t mode,
1402:                               bool sparse, const std::optional<Tensor>& per_sample_weights_opt,
1403:                               int64_t padding_idx) {
1404:   // See [Note: hacky wrapper removal for optional tensor]
1405:   c10::MaybeOwned<Tensor> per_sample_weights_maybe_owned = at::borrow_from_optional_tensor(per_sample_weights_opt);
1406:   const Tensor& per_sample_weights = *per_sample_weights_maybe_owned;
1407: 
1408:   auto [indicesMaybeOwned, offsetsMaybeOwned] = promoteIndicesAndOffsets(indices_, offsets_);
1409:   const auto& indices = *indicesMaybeOwned;
1410:   const auto& offsets = *offsetsMaybeOwned;
```
- **EN**: Lines 1381-1410 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: _embedding_bag_backward_symint, borrow_from_optional_tensor, promoteIndicesAndOffsets.
- **CN**: 第 1381-1410 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：_embedding_bag_backward_symint, borrow_from_optional_tensor, promoteIndicesAndOffsets。

### Lines 1411-1440 / 第 1411-1440 行
```cpp
1411:   auto indices_arg = TensorArg(indices, "indices", 1);
1412:   checkScalarTypes("embedding_bag", indices_arg, {kLong, kInt});
1413:   checkContiguous("embedding_bag", indices_arg);
1414:   auto offsets_arg = TensorArg(offsets, "offsets", 1);
1415:   checkScalarTypes("embedding_bag", offsets_arg, {kLong, kInt});
1416:   checkSameType("embedding_bag", indices_arg, offsets_arg);
1417:   checkContiguous("embedding_bag", offsets_arg);
1418: 
1419:   Tensor offset2bag_;
1420:   if (indices.sym_numel() != 0 && offset2bag.sym_numel() == 0) {
1421:     offset2bag_ = offsets.new_zeros(
1422:       {indices.size(0) + 1}, offsets.options()); // offset2bag = [0 0 0 0 0]
1423: 
1424:     make_offset2bag(offsets, offset2bag_);
1425:     // For Composite Compliance, if `offset2bag_` is CCT
1426:     // then we can't call `resize_`. Instead we call `narrow`
1427:     // to slice the tensor.
1428:     if (isTensorSubclassLike(offset2bag_)) {
1429:       offset2bag_ = offset2bag_.narrow(0, 0, indices.size(0));
1430:     } else {
1431:       offset2bag_.resize_({indices.size(0)});
1432:     }
1433:   } else {
1434:     auto offset2bag_arg = TensorArg(offset2bag, "offset2bag", 1);
1435:     checkScalarTypes("embedding_bag", offset2bag_arg, {kLong, kInt});
1436:     checkContiguous("embedding_bag", offset2bag_arg);
1437:     offset2bag_ = offset2bag;
1438:   }
1439: 
1440:   if (sparse) {
```
- **EN**: Lines 1411-1440 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: TensorArg, checkScalarTypes, checkContiguous, checkSameType.
- **CN**: 第 1411-1440 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：TensorArg, checkScalarTypes, checkContiguous, checkSameType。

### Lines 1441-1470 / 第 1441-1470 行
```cpp
1441:     return at::_embedding_bag_sparse_backward_symint(
1442:         grad, indices, offsets, offset2bag_, bag_size_, std::move(num_weights),
1443:         scale_grad_by_freq, mode, per_sample_weights, padding_idx);
1444:   } else {
1445:     return at::_embedding_bag_dense_backward_symint(
1446:         grad, indices, offset2bag_, bag_size_, max_indices_, std::move(num_weights),
1447:         scale_grad_by_freq, mode, per_sample_weights, padding_idx);
1448:   }
1449: }
1450: 
1451: static Tensor _embedding_bag_dense_backward_cpu_max(
1452:     const Tensor& grad,
1453:     const Tensor& bag_size,
1454:     const Tensor& max_indices,
1455:     int64_t num_weights) {
1456:   AT_ASSERT(max_indices.defined());
1457:   auto index_grad_weight =
1458:       at::zeros({num_weights, grad.sizes()[1]}, grad.options());
1459:   auto nonempty_max_indices = max_indices.index_select(0, bag_size.nonzero().view(-1));
1460:   auto nonempty_grad = grad.index_select(0, bag_size.nonzero().view(-1));
1461: 
1462:   for (const auto dim : c10::irange(grad.sizes()[1])) {
1463:     index_grad_weight.select(1, dim).index_add_(
1464:       0, nonempty_max_indices.select(1, dim), nonempty_grad.select(1, dim));
1465:   }
1466:   return index_grad_weight;
1467: }
1468: 
1469: template<typename index_t>
1470: static std::vector<index_t> compute_counts(
```
- **EN**: Lines 1441-1470 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: _embedding_bag_sparse_backward_symint, move, _embedding_bag_dense_backward_symint, _embedding_bag_dense_backward_cpu_max.
- **CN**: 第 1441-1470 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：_embedding_bag_sparse_backward_symint, move, _embedding_bag_dense_backward_symint, _embedding_bag_dense_backward_cpu_max。

### Lines 1471-1500 / 第 1471-1500 行
```cpp
1471:     int64_t num_weights,
1472:     const index_t* indices_data,
1473:     int64_t indices_length) {
1474:   std::vector<index_t> counts(num_weights, 0);
1475:   for (const auto i : c10::irange(indices_length)) {
1476:     counts[indices_data[i]]++;
1477:   }
1478:   return counts;
1479: }
1480: 
1481: // counts_uniq stores the index of the NEXT unique element
1482: // of the (sorted) indices vector.
1483: //
1484: // For example:
1485: // indices: [0, 0, 0, 1, 3, 3, 4]
1486: // counts: [3, 1, 0, 2, 1, 0]
1487: // counts_uniq: [3, 4, 6, 7]
1488: //
1489: // The unique indices can be found at index 0, 3, 4, 6.
1490: template<typename index_t>
1491: static std::vector<index_t> compute_counts_uniq(
1492:     int64_t num_weights,
1493:     const index_t* indices_data,
1494:     int64_t indices_length,
1495:     const std::vector<index_t>& counts) {
1496:   std::vector<index_t> counts_uniq;
1497:   counts_uniq.reserve(num_weights);
1498:   int64_t o = 0;
1499:   for (int64_t i = 0; i < indices_length; i += counts[indices_data[i]]) {
1500:     counts_uniq.push_back(counts[indices_data[i]]);
```
- **EN**: Lines 1471-1500 mainly cover comments/documentation, expressions/calls, state/variable declarations. Notable symbols: counts, irange, the, compute_counts_uniq.
- **CN**: 第 1471-1500 行主要涉及注释或说明、表达式或调用、变量/别名声明。 值得关注的符号包括：counts, irange, the, compute_counts_uniq。

### Lines 1501-1530 / 第 1501-1530 行
```cpp
1501:     if (o > 0) {
1502:       counts_uniq[o] += counts_uniq[o - 1];
1503:     }
1504:     o++;
1505:   }
1506:   return counts_uniq;
1507: }
1508: 
1509: template <typename scalar_t>
1510: static void _embedding_bag_dense_backward_cpu_sum_mean(
1511:     const Tensor& grad,
1512:     const Tensor& indices_,
1513:     const Tensor& offset2bag_,
1514:     const Tensor& bag_size_,
1515:     int64_t num_weights,
1516:     bool scale_grad_by_freq,
1517:     int64_t mode,
1518:     const Tensor& per_sample_weights_,
1519:     Tensor& index_grad_weight,
1520:     int64_t padding_idx) {
1521: 
1522:   auto ind_sort_ = indices_.sort();
1523:   auto const& indices = std::get<0>(ind_sort_);
1524:   auto const& ind_sort = std::get<1>(ind_sort_);
1525:   auto offset2bag = offset2bag_.index_select(0, ind_sort);
1526: 
1527:   std::optional<Tensor> per_sample_weights;
1528:   const scalar_t* per_sample_weights_data = nullptr;
1529:   std::optional<int64_t> per_sample_weights_stride;
1530:   if (per_sample_weights_.defined()) {
```
- **EN**: Lines 1501-1530 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: _embedding_bag_dense_backward_cpu_sum_mean, sort, index_select, defined.
- **CN**: 第 1501-1530 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：_embedding_bag_dense_backward_cpu_sum_mean, sort, index_select, defined。

### Lines 1531-1560 / 第 1531-1560 行
```cpp
1531:     per_sample_weights = per_sample_weights_.index_select(0, ind_sort);
1532:     per_sample_weights_data = per_sample_weights->const_data_ptr<scalar_t>();
1533:     per_sample_weights_stride = per_sample_weights->strides()[0];
1534:   }
1535: 
1536:   int64_t numel = indices.numel();
1537: 
1538:   // explicitly capture all required variables to work around windows build
1539:   // TODO: fix this when windows can correctly capture variables in nested lambda
1540:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "_embedding_bag_dense_backward_cpu_sum_mean",
1541:     [&indices, &offset2bag, &bag_size_, &num_weights, &numel, &per_sample_weights,
1542:       &per_sample_weights_data, &per_sample_weights_stride, &mode, &scale_grad_by_freq,
1543:       &grad, &index_grad_weight, &padding_idx] {
1544:     auto* indices_data = indices.const_data_ptr<index_t>();
1545:     auto* offset2bag_data = offset2bag.const_data_ptr<index_t>();
1546:     auto* bag_size_data = bag_size_.const_data_ptr<index_t>();
1547: 
1548:     auto counts = compute_counts(num_weights, indices_data, numel);
1549:     auto next_unique_index_idx =
1550:         compute_counts_uniq(num_weights, indices_data, numel, counts);
1551: 
1552:     auto loop =
1553:       [&next_unique_index_idx, &indices_data, &offset2bag_data, &bag_size_data, &per_sample_weights,
1554:         &mode, &per_sample_weights_data, &per_sample_weights_stride, &scale_grad_by_freq,
1555:         &counts, &grad, &index_grad_weight, &padding_idx
1556:       ](index_t start, index_t end) {
1557:       for (index_t i = start; i < end; i++) {
1558:         index_t indices_start = i == 0 ? 0 : next_unique_index_idx[i - 1];
1559:         index_t index = indices_data[indices_start];
1560: 
```
- **EN**: Lines 1531-1560 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: index_select, strides, numel, AT_DISPATCH_INDEX_TYPES.
- **CN**: 第 1531-1560 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：index_select, strides, numel, AT_DISPATCH_INDEX_TYPES。

### Lines 1561-1590 / 第 1561-1590 行
```cpp
1561:         if (index != static_cast<index_t>(padding_idx)) {
1562:           for (index_t j = indices_start; j < next_unique_index_idx[i]; j++) {
1563:             index_t source = offset2bag_data[j];
1564:             double scale = 1.0;
1565:             if (per_sample_weights) {
1566:               AT_ASSERT(mode == EmbeddingBagMode::SUM);
1567:               scale = per_sample_weights_data[*per_sample_weights_stride * j];
1568:             }
1569:             if (scale_grad_by_freq) {
1570:               scale /= counts[indices_data[i]];
1571:             }
1572:             if (mode == EmbeddingBagMode::MEAN) {
1573:               auto bag_size = bag_size_data[source];
1574:               if (bag_size != 0) {
1575:                 scale /= bag_size;
1576:               }
1577:             }
1578:             int64_t ddim = grad.size(1);
1579:             auto igwd = index_grad_weight.data_ptr<scalar_t>();
1580:             auto gd = grad.const_data_ptr<scalar_t>();
1581:             at::native::cpublas::axpy<scalar_t>(ddim, (scalar_t)scale, gd + ddim * source, 1,
1582:                         igwd + ddim * index, 1);
1583:           }
1584:         }
1585:       }
1586:     };
1587: 
1588:     if (numel > 1000) {
1589:       at::parallel_for(0, (int64_t)next_unique_index_idx.size(), 0, loop);
1590:     } else {
```
- **EN**: Lines 1561-1590 mainly cover state/variable declarations, control-flow checks, expressions/calls. Notable symbols: AT_ASSERT, size, parallel_for.
- **CN**: 第 1561-1590 行主要涉及变量/别名声明、控制流逻辑、表达式或调用。 值得关注的符号包括：AT_ASSERT, size, parallel_for。

### Lines 1591-1620 / 第 1591-1620 行
```cpp
1591:       loop(0, (int64_t)next_unique_index_idx.size());
1592:     }
1593:   });
1594: }
1595: 
1596: Tensor _embedding_bag_dense_backward_cpu(const Tensor &grad_, const Tensor &indices_,
1597:                                   const Tensor &offset2bag__,
1598:                                   const Tensor &bag_size_,
1599:                                   const Tensor& max_indices_, int64_t num_weights,
1600:                                   bool scale_grad_by_freq, int64_t mode, const std::optional<Tensor>& per_sample_weights__opt,
1601:                                   int64_t padding_idx) {
1602:   // See [Note: hacky wrapper removal for optional tensor]
1603:   c10::MaybeOwned<Tensor> per_sample_weights__maybe_owned = at::borrow_from_optional_tensor(per_sample_weights__opt);
1604:   const Tensor& per_sample_weights_ = *per_sample_weights__maybe_owned;
1605: 
1606:   // indices_, offsets_ and offset2bag__ are assumed having correct dtypes and
1607:   // contiguous here due to the checks in _embedding_bag_backward above.
1608:   // Also see NOTE [ embedding_bag Native Functions ] in native_functions.yaml
1609:   // for more details.
1610:   auto grad = grad_.contiguous();
1611:   auto grad_arg = TensorArg(grad, "grad_", 1);
1612:   checkScalarTypes(
1613:       "embedding_bag", grad_arg, {kHalf, kBFloat16, kFloat, kDouble});
1614: 
1615:   if (mode == EmbeddingBagMode::MAX) {
1616:     return _embedding_bag_dense_backward_cpu_max(
1617:         grad_, bag_size_, max_indices_, num_weights);
1618:   }
1619:   AT_ASSERT(mode == EmbeddingBagMode::MEAN || mode == EmbeddingBagMode::SUM);
1620: 
```
- **EN**: Lines 1591-1620 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: loop, size, _embedding_bag_dense_backward_cpu, borrow_from_optional_tensor.
- **CN**: 第 1591-1620 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：loop, size, _embedding_bag_dense_backward_cpu, borrow_from_optional_tensor。

### Lines 1621-1650 / 第 1621-1650 行
```cpp
1621:   auto index_grad_weight =
1622:       at::zeros({num_weights, grad.sizes()[1]}, grad.options());
1623: 
1624:   AT_DISPATCH_FLOATING_TYPES_AND2(
1625:       at::ScalarType::Half,
1626:       at::ScalarType::BFloat16,
1627:       grad.scalar_type(),
1628:       "embedding_bag_backward",
1629:       [&] {
1630:         _embedding_bag_dense_backward_cpu_sum_mean<scalar_t>(
1631:             grad,
1632:             indices_,
1633:             offset2bag__,
1634:             bag_size_,
1635:             num_weights,
1636:             scale_grad_by_freq,
1637:             mode,
1638:             per_sample_weights_,
1639:             index_grad_weight,
1640:             padding_idx);
1641:       });
1642:   return index_grad_weight;
1643: }
1644: 
1645: template<typename scalar_t>
1646: static Tensor _embedding_bag_per_sample_weights_backward_cpu_template(
1647:     const Tensor& grad,
1648:     const Tensor& weight,  // NB: embedding table, not per_sample_weights
1649:     const Tensor& indices_,
1650:     const Tensor& offsets_,
```
- **EN**: Lines 1621-1650 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: zeros, sizes, options, AT_DISPATCH_FLOATING_TYPES_AND2.
- **CN**: 第 1621-1650 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：zeros, sizes, options, AT_DISPATCH_FLOATING_TYPES_AND2。

### Lines 1651-1680 / 第 1651-1680 行
```cpp
1651:     const Tensor& offset2bag,
1652:     int64_t mode,
1653:     int64_t padding_idx) {
1654:   TORCH_CHECK(
1655:       mode == EmbeddingBagMode::SUM,
1656:       "embedding_bag_backward: per_sample_weights only supported for mode='sum'");
1657: 
1658:   AT_ASSERT(grad.dim() == 2);
1659:   auto embedding_features = grad.sizes()[1];
1660: 
1661:   auto [indicesMaybeOwned, offsetsMaybeOwned] = promoteIndicesAndOffsets(indices_, offsets_);
1662:   const auto& indices = *indicesMaybeOwned;
1663:   const auto& offsets = *offsetsMaybeOwned;
1664: 
1665:   AT_ASSERT(indices.dim() == 1);
1666:   auto num_samples = indices.size(0);
1667: 
1668:   AT_ASSERT(weight.dim() == 2);
1669:   AT_ASSERT(weight.sizes()[1] == embedding_features);
1670: 
1671:   auto output = at::zeros({num_samples}, grad.options());
1672: 
1673:   auto indices_arg = TensorArg(indices, "indices", 1);
1674:   checkScalarTypes("embedding_bag", indices_arg, {kLong, kInt});
1675:   checkContiguous("embedding_bag", indices_arg);
1676: 
1677:   Tensor offset2bag_;
1678:   if (indices.numel() != 0 && offset2bag.numel() == 0) {
1679:     offset2bag_ = at::zeros(
1680:        {indices.size(0) + 1}, offset2bag.options()); // offset2bag = [0 0 0 0 0]
```
- **EN**: Lines 1651-1680 mainly cover state/variable declarations, macro-based glue, expressions/calls. Notable symbols: TORCH_CHECK, AT_ASSERT, dim, sizes.
- **CN**: 第 1651-1680 行主要涉及变量/别名声明、宏定义或宏调用、表达式或调用。 值得关注的符号包括：TORCH_CHECK, AT_ASSERT, dim, sizes。

### Lines 1681-1710 / 第 1681-1710 行
```cpp
1681: 
1682:     make_offset2bag(offsets, offset2bag_);
1683: 
1684:     at::native::resize_(offset2bag_, {indices.size(0)}, std::nullopt);
1685:   } else {
1686:     auto offset2bag_arg = TensorArg(offset2bag, "offset2bag", 1);
1687:     checkScalarTypes("embedding_bag", offset2bag_arg, {kLong, kInt});
1688:     checkContiguous("embedding_bag", offset2bag_arg);
1689:     offset2bag_ = offset2bag;
1690:   }
1691: 
1692:   auto* grad_data = grad.const_data_ptr<scalar_t>();
1693:   auto grad_stride0 = grad.strides()[0];
1694:   auto grad_stride1 = grad.strides()[1];
1695: 
1696:   auto* weight_data = weight.const_data_ptr<scalar_t>();
1697:   auto weight_stride0 = weight.strides()[0];
1698:   auto weight_stride1 = weight.strides()[1];
1699: 
1700:   // explicitly capture all required variables to work around windows build
1701:   // TODO: fix this when windows can correctly capture variables in nested lambda
1702:   AT_DISPATCH_INDEX_TYPES(indices.scalar_type(), "_embedding_bag_per_sample_weights_backward_cpu_template",
1703:     [&indices, &output, &offset2bag_, &num_samples, &embedding_features,
1704:       &grad_data, &grad_stride0, &grad_stride1, &weight_data, &weight_stride0, &weight_stride1,
1705:       &padding_idx] () {
1706:     auto* indices_data = indices.const_data_ptr<index_t>();
1707: 
1708:     // The following are contiguous
1709:     auto* output_data = output.data_ptr<scalar_t>();
1710:     auto* offset2bag_data = offset2bag_.const_data_ptr<index_t>();
```
- **EN**: Lines 1681-1710 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: make_offset2bag, resize_, size, TensorArg.
- **CN**: 第 1681-1710 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：make_offset2bag, resize_, size, TensorArg。

### Lines 1711-1740 / 第 1711-1740 行
```cpp
1711: 
1712:     // XXX: 64 was arbitrarily chosen. There is probably a sweet spot for this number.
1713:     parallel_for(0, num_samples, 64,
1714:       [&embedding_features, &grad_data, &grad_stride0, &grad_stride1, &weight_data, &weight_stride0,
1715:         &weight_stride1, &offset2bag_data, &indices_data, &output_data, &padding_idx](index_t begin, index_t end) {
1716:       for (index_t sample_idx = begin; sample_idx < end; sample_idx++) {
1717:         auto bag_idx = offset2bag_data[sample_idx];
1718:         auto embedding_idx = indices_data[sample_idx];
1719: 
1720:         if (embedding_idx != static_cast<index_t>(padding_idx)) {
1721:           output_data[sample_idx] = dot_impl<scalar_t>(
1722:               embedding_features, grad_data + grad_stride0 * bag_idx, grad_stride1,
1723:                  weight_data + weight_stride0 * embedding_idx, weight_stride1);
1724:         }
1725:       }
1726:     });
1727:   });
1728:   return output;
1729: }
1730: 
1731: Tensor _embedding_bag_per_sample_weights_backward_cpu(
1732:     const Tensor& grad,
1733:     const Tensor& weight,  // NB: embedding table, not per_sample_weights
1734:     const Tensor& indices,
1735:     const Tensor& offsets,
1736:     const Tensor& offset2bag,
1737:     int64_t mode,
1738:     int64_t padding_idx) {
1739:   return AT_DISPATCH_FLOATING_TYPES_AND2(
1740:       at::ScalarType::Half,
```
- **EN**: Lines 1711-1740 mainly cover expressions/calls, state/variable declarations, function signatures/definitions. Notable symbols: parallel_for, _embedding_bag_per_sample_weights_backward_cpu, AT_DISPATCH_FLOATING_TYPES_AND2.
- **CN**: 第 1711-1740 行主要涉及表达式或调用、变量/别名声明、函数签名或实现。 值得关注的符号包括：parallel_for, _embedding_bag_per_sample_weights_backward_cpu, AT_DISPATCH_FLOATING_TYPES_AND2。

### Lines 1741-1770 / 第 1741-1770 行
```cpp
1741:       at::ScalarType::BFloat16,
1742:       grad.scalar_type(),
1743:       "_embedding_bag_per_sample_weights_backward_cpu",
1744:       [&]() {
1745:         return _embedding_bag_per_sample_weights_backward_cpu_template<
1746:             scalar_t>(
1747:             grad, weight, indices, offsets, offset2bag, mode, padding_idx);
1748:       });
1749: }
1750: 
1751: Tensor _embedding_bag_sparse_backward_symint(
1752:     const Tensor &grad_, const Tensor &indices, [[maybe_unused]] const Tensor &offsets,
1753:     const Tensor &offset2bag, const Tensor &bag_size_, SymInt num_weights,
1754:     bool scale_grad_by_freq, int64_t mode, const std::optional<Tensor>& per_sample_weights_opt,
1755:     int64_t padding_idx) {
1756:   // See [Note: hacky wrapper removal for optional tensor]
1757:   c10::MaybeOwned<Tensor> per_sample_weights_maybe_owned = at::borrow_from_optional_tensor(per_sample_weights_opt);
1758:   const Tensor& per_sample_weights = *per_sample_weights_maybe_owned;
1759: 
1760:   // indices, offsets and offset2bag are assumed having correct dtypes and
1761:   // contiguous here due to the checks in _embedding_bag_backward above.
1762:   // Also see NOTE [ embedding_bag Native Functions ] in native_functions.yaml
1763:   // for more details.
1764: 
1765:   Tensor index_grad = grad_.index_select(0, offset2bag);
1766: 
1767:   index_grad = apply_bag_size_backward(mode, index_grad, offset2bag, bag_size_);
1768: 
1769:   if (per_sample_weights.defined()) {
1770:     AT_ASSERT(mode == EmbeddingBagMode::SUM);
```
- **EN**: Lines 1741-1770 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: scalar_type, _embedding_bag_sparse_backward_symint, borrow_from_optional_tensor, index_select.
- **CN**: 第 1741-1770 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：scalar_type, _embedding_bag_sparse_backward_symint, borrow_from_optional_tensor, index_select。

### Lines 1771-1776 / 第 1771-1776 行
```cpp
1771:     index_grad.mul_(per_sample_weights.unsqueeze(1));
1772:   }
1773:   return native::embedding_backward_symint(index_grad, indices, std::move(num_weights), padding_idx,
1774:                                     scale_grad_by_freq, true);
1775: }
1776: } // namespace at::native
```
- **EN**: Lines 1771-1776 mainly cover state/variable declarations, expressions/calls, return paths. Notable symbols: mul_, unsqueeze, embedding_backward_symint, move.
- **CN**: 第 1771-1776 行主要涉及变量/别名声明、表达式或调用、返回路径。 值得关注的符号包括：mul_, unsqueeze, embedding_backward_symint, move。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: Parallel loop scheduling  
  **CN**: 并行循环调度
- **EN**: Template-based specialization  
  **CN**: 基于模板的特化
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/TensorOperators.h>`, `<ATen/TensorSubclassLikeUtils.h>`, `<ATen/TensorUtils.h>`, `<ATen/cpu/vec/vec.h>`, `<ATen/native/EmbeddingBag.h>`, `<ATen/native/CPUBlas.h>`, `<ATen/native/NonSymbolicBC.h>`, `<c10/util/irange.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
