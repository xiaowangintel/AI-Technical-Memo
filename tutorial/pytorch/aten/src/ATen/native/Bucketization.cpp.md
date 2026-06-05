# Bucketization.cpp — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/Bucketization.cpp`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Bucketization.
- **Purpose (CN)**: 实现或声明与 bucketization 相关的 ATen 原生逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
0002: #include <ATen/core/Tensor.h>
0003: #include <ATen/Dispatch.h>
0004: #include <ATen/Parallel.h>
0005: #include <ATen/native/BucketizationUtils.h>
0006: #include <ATen/native/Resize.h>
0007: #include <c10/util/irange.h>
0008: 
0009: #ifndef AT_PER_OPERATOR_HEADERS
0010: #include <ATen/Functions.h>
0011: #else
0012: #include <ATen/ops/empty.h>
0013: #include <ATen/ops/bucketize_native.h>
0014: #include <ATen/ops/searchsorted_native.h>
0015: #endif
0016: 
0017: /* Implement a numpy like searchsorted and a TF like bucketize function running on cpu
0018:  *
0019:  * - torch.searchsorted(sorted_sequence, values, right=False, side=None, out_int32=False, sorter=None)
0020:  *   sorted_sequence - N*D or 1D (apply to all values) tensor containing sorted sequences in last dimension
0021:  *   values          - N*D tensor or a Scalar (when sorted_sequence is 1D) containing the search values
0022:  *   right           - corresponding to lower bound if False and upper bound if True
0023:  *   side            - (preferred to right) corresponding to lower bound if 'left' and upper bound if 'right'
0024:  *   out_int32       - the output tensor is int64_t type if False and int(32bit normally) type if True.
0025:  *   sorter          - if provided, sorted_sequence may not be sorted and the sorted order is given by this tensor
0026:  *
0027:  * - torch.bucketize(values, boundaries, right=False, out_int32=False)
0028:  *   values     - N*D tensor or a Scalar containing the search value
0029:  *   boundaries - 1D tensor containing a sorted sequences
0030:  *   right      - corresponding to lower bound if False and upper bound if True
```
- **EN**: Lines 1-30 mainly cover comments/documentation, header inclusion, conditional compilation. Notable symbols: searchsorted, D, Scalar, int.
- **CN**: 第 1-30 行主要涉及注释或说明、头文件包含、预处理条件。 值得关注的符号包括：searchsorted, D, Scalar, int。

### Lines 31-60 / 第 31-60 行
```cpp
0031:  *   out_int32  - the output tensor is int64_t type if False and int(32bit normally) type if True.
0032:  *
0033:  * - Restrictions are defined in searchsorted_pre_check()
0034:  */
0035: 
0036: namespace at::native {
0037: 
0038: namespace {
0039: 
0040: // minimal size for searchsorted_cpu_contiguous to run parallel (multithread)
0041: constexpr int64_t SEARCHSORTED_GRAIN_SIZE = 200;
0042: 
0043: // customized lower_bound func to ensure the low bound of 'nan', 'inf' etc. be the end of boundary
0044: // and we can properly handle a sorter argument
0045: // std::lower_bound can not be used here since its customized comparator need strict weak ordering
0046: // and the customized comparators require both arguments to have the same type, which wouldn't
0047: // happen when comparing val of input_t to an indexer value from sorter of int64
0048: template<typename input_t>
0049: int64_t cus_lower_bound(int64_t start, int64_t end, const input_t val, const input_t* bd, const int64_t* sort) {
0050:   // sorter gives relative ordering for ND tensors, so we need to save and add the non-updated start as an offset
0051:   // i.e. the second row of a 3x3 tensors starts at element 3 but sorter's second row only contains 0, 1, or 2
0052:   const int64_t orig_start = start;
0053:   while (start < end) {
0054:     const int64_t mid = start + ((end - start) >> 1);
0055:     const input_t mid_val = sort ? bd[sort[mid] + orig_start] : bd[mid];
0056:     if (!(mid_val >= val)) {
0057:       start = mid + 1;
0058:     }
0059:     else {
0060:       end = mid;
```
- **EN**: Lines 31-60 mainly cover comments/documentation, state/variable declarations, control-flow checks. Notable symbols: int, searchsorted_pre_check, parallel, cus_lower_bound.
- **CN**: 第 31-60 行主要涉及注释或说明、变量/别名声明、控制流逻辑。 值得关注的符号包括：int, searchsorted_pre_check, parallel, cus_lower_bound。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     }
0062:   }
0063:   return start;
0064: }
0065: 
0066: // customized upper_bound func to ensure we can properly handle a sorter argument
0067: // std::upper_bound can not be used here since its customized comparator requires both arguments to have the
0068: // same type, which wouldn't happen when comparing val of input_t to an indexer value from sorter of int64
0069: template<typename input_t>
0070: int64_t cus_upper_bound(int64_t start, int64_t end, const input_t val, const input_t* bd, const int64_t* sort) {
0071:   // sorter gives relative ordering for ND tensors, so we need to save and add the non-updated start as an offset
0072:   // i.e. the second row of a 3x3 tensors starts at element 3 but sorter's second row only contains 0, 1, or 2
0073:   const int64_t orig_start = start;
0074:   while (start < end) {
0075:     const int64_t mid = start + ((end - start) >> 1);
0076:     const input_t mid_val = sort ? bd[sort[mid] + orig_start] : bd[mid];
0077:     if (!(mid_val > val)) {
0078:       start = mid + 1;
0079:     }
0080:     else {
0081:       end = mid;
0082:     }
0083:   }
0084:   return start;
0085: }
0086: 
0087: template<typename input_t, typename output_t>
0088: void searchsorted_cpu_contiguous(Tensor& result, const Tensor& input, const Tensor& boundaries, const bool& right, const Tensor& sorter) {
0089:   int64_t numel_in = input.numel();
0090:   bool is_scalar_input = input.dim() == 0 && numel_in == 1;
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: cus_upper_bound, searchsorted_cpu_contiguous, numel, dim.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：cus_upper_bound, searchsorted_cpu_contiguous, numel, dim。

### Lines 91-120 / 第 91-120 行
```cpp
0091:   // inner most dim size of input and boundaries
0092:   int64_t idim_in = is_scalar_input ? 1 : input.sizes().back();
0093:   int64_t idim_bd = boundaries.sizes().back();
0094: 
0095:   const input_t *data_in = input.const_data_ptr<input_t>();
0096:   const input_t *data_bd = boundaries.const_data_ptr<input_t>();
0097:   const int64_t *data_st = sorter.defined() ? sorter.const_data_ptr<int64_t>() : nullptr;
0098:   output_t *data_out = result.data_ptr<output_t>();
0099: 
0100:   bool is_1d_boundaries = boundaries.dim() == 1;
0101:   at::parallel_for(0, numel_in, SEARCHSORTED_GRAIN_SIZE, [&](int64_t start, int64_t end) {
0102:     for (const auto i : c10::irange(start, end)) {
0103:       // If boundaries tensor is 1d, we always search the entire boundary tensor
0104:       int64_t start_bd = is_1d_boundaries ? 0 : i / idim_in * idim_bd;
0105:       int64_t end_bd = start_bd + idim_bd;
0106: 
0107:       int64_t pos = !right ?
0108:         cus_lower_bound(start_bd, end_bd, data_in[i], data_bd, data_st) - start_bd :
0109:         cus_upper_bound(start_bd, end_bd, data_in[i], data_bd, data_st) - start_bd;
0110: 
0111:       // type conversion might happen here
0112:       data_out[i] = pos;
0113:     }
0114:   });
0115: }
0116: 
0117: void dispatch(Tensor& result, const Tensor& input, const Tensor& boundaries, bool out_int32, bool right, const Tensor& sorter) {
0118:   if (!out_int32) {
0119:     AT_DISPATCH_ALL_TYPES_AND2(
0120:         ScalarType::Half,
```
- **EN**: Lines 91-120 mainly cover state/variable declarations, expressions/calls, comments/documentation. Notable symbols: sizes, back, defined, dim.
- **CN**: 第 91-120 行主要涉及变量/别名声明、表达式或调用、注释或说明。 值得关注的符号包括：sizes, back, defined, dim。

### Lines 121-150 / 第 121-150 行
```cpp
0121:         ScalarType::BFloat16,
0122:         input.scalar_type(),
0123:         "searchsorted_out_cpu",
0124:         [&] {
0125:           searchsorted_cpu_contiguous<scalar_t, int64_t>(
0126:               result, input, boundaries, right, sorter);
0127:         });
0128:   }
0129:   else {
0130:     AT_DISPATCH_ALL_TYPES_AND2(
0131:         ScalarType::Half,
0132:         ScalarType::BFloat16,
0133:         input.scalar_type(),
0134:         "searchsorted_out_cpu",
0135:         [&] {
0136:           searchsorted_cpu_contiguous<scalar_t, int>(
0137:               result, input, boundaries, right, sorter);
0138:         });
0139:   }
0140: }
0141: 
0142: }
0143: 
0144: Tensor& searchsorted_out_cpu(
0145:     const Tensor& sorted_sequence,
0146:     const Tensor& self,
0147:     bool out_int32,
0148:     bool right,
0149:     const std::optional<std::string_view> side_opt,
0150:     const std::optional<Tensor>& sorter_opt,
```
- **EN**: Lines 121-150 mainly cover expressions/calls, function signatures/definitions, state/variable declarations. Notable symbols: scalar_type, AT_DISPATCH_ALL_TYPES_AND2, searchsorted_out_cpu.
- **CN**: 第 121-150 行主要涉及表达式或调用、函数签名或实现、变量/别名声明。 值得关注的符号包括：scalar_type, AT_DISPATCH_ALL_TYPES_AND2, searchsorted_out_cpu。

### Lines 151-180 / 第 151-180 行
```cpp
0151:     Tensor& result) {
0152:   // See [Note: hacky wrapper removal for optional tensor]
0153:   c10::MaybeOwned<Tensor> sorter_maybe_owned = at::borrow_from_optional_tensor(sorter_opt);
0154:   const Tensor& sorter = *sorter_maybe_owned;
0155:   searchsorted_pre_check(sorted_sequence, self, result, out_int32, right, side_opt, sorter);
0156:   resize_output(result, self.sizes());
0157: 
0158:   // we have two inputs to set right, pre_check checks that they aren't set to opposites
0159:   bool is_right = side_opt ? *side_opt == "right" : right;
0160: 
0161:   if (self.numel() == 0) {
0162:     return result;
0163:   }
0164: 
0165:   // for non-contiguous result tensors, we write the output to a contiguous copy so we can later copy back, maintaining the original result tensor
0166:   Tensor out = result;
0167:   if (!result.is_contiguous()) {
0168:     out = result.contiguous();
0169:   }
0170:   if (sorted_sequence.is_contiguous() && self.is_contiguous() && sorted_sequence.dtype() == self.dtype() && sorter.is_contiguous()) {
0171:     dispatch(out, self, sorted_sequence, out_int32, is_right, sorter);
0172:   }
0173:   else {
0174:     Tensor trimmed_input;
0175:     Tensor trimmed_boundaries;
0176:     Tensor trimmed_sorter;
0177:     searchsorted_maybe_trim_input_tensors(trimmed_input, trimmed_boundaries, trimmed_sorter, self, sorted_sequence, sorter);
0178:     const Tensor& final_input = trimmed_input.defined() ? trimmed_input : self;
0179:     const Tensor& final_boundaries = trimmed_boundaries.defined() ? trimmed_boundaries : sorted_sequence;
0180:     const Tensor& final_sorter = trimmed_sorter.defined() ? trimmed_sorter : sorter;
```
- **EN**: Lines 151-180 mainly cover state/variable declarations, control-flow checks, comments/documentation. Notable symbols: borrow_from_optional_tensor, searchsorted_pre_check, resize_output, sizes.
- **CN**: 第 151-180 行主要涉及变量/别名声明、控制流逻辑、注释或说明。 值得关注的符号包括：borrow_from_optional_tensor, searchsorted_pre_check, resize_output, sizes。

### Lines 181-210 / 第 181-210 行
```cpp
0181:     dispatch(out, final_input, final_boundaries, out_int32, is_right, final_sorter);
0182:   }
0183: 
0184:   // if result is non-contiguous, we wrote the answer to a copied version, so we copy back to the original result tensor
0185:   if (!result.is_contiguous()) {
0186:     result.copy_(out);
0187:   }
0188:   return result;
0189: }
0190: 
0191: Tensor& searchsorted_out_cpu(
0192:     const Tensor& sorted_sequence,
0193:     const Scalar& self,
0194:     bool out_int32,
0195:     bool right,
0196:     const std::optional<std::string_view> side_opt,
0197:     const std::optional<Tensor>& sorter_opt,
0198:     Tensor& result) {
0199:   const Tensor& scalar_tensor = searchsorted_scalar_tensor(self, sorted_sequence.device());
0200:   return searchsorted_out_cpu(sorted_sequence, scalar_tensor, out_int32, right, side_opt, sorter_opt, result);
0201: }
0202: 
0203: Tensor searchsorted_cpu(
0204:       const Tensor& sorted_sequence,
0205:       const Tensor& self,
0206:       bool out_int32,
0207:       bool right,
0208:       const std::optional<std::string_view> side_opt,
0209:       const std::optional<Tensor>& sorter_opt) {
0210:   ScalarType scalar_type = out_int32 ? ScalarType::Int : ScalarType::Long;
```
- **EN**: Lines 181-210 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: dispatch, is_contiguous, copy_, searchsorted_out_cpu.
- **CN**: 第 181-210 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：dispatch, is_contiguous, copy_, searchsorted_out_cpu。

### Lines 211-240 / 第 211-240 行
```cpp
0211:   c10::TensorOptions options = TensorOptions().device(self.options().device()).dtype(scalar_type);
0212:   Tensor result = at::empty({0}, options, MemoryFormat::Contiguous);
0213:   at::native::searchsorted_out_cpu(sorted_sequence, self, out_int32, right, side_opt, sorter_opt, result);
0214:   return result;
0215: }
0216: 
0217: Tensor searchsorted_cpu(
0218:     const Tensor& sorted_sequence,
0219:     const Scalar& self,
0220:     bool out_int32,
0221:     bool right,
0222:     const std::optional<std::string_view> side_opt,
0223:     const std::optional<Tensor>& sorter_opt) {
0224:   const Tensor& scalar_tensor = searchsorted_scalar_tensor(self, sorted_sequence.device());
0225:   return searchsorted_cpu(sorted_sequence, scalar_tensor, out_int32, right, side_opt, sorter_opt);
0226: }
0227: 
0228: Tensor& bucketize_out_cpu(const Tensor& self, const Tensor& boundaries, bool out_int32, bool right, Tensor& result) {
0229:   TORCH_CHECK(boundaries.dim() == 1, "boundaries tensor must be 1 dimension, but got dim(", boundaries.dim(), ")");
0230:   at::native::searchsorted_out_cpu(boundaries, self, out_int32, right, std::nullopt, std::nullopt, result);
0231:   return result;
0232: }
0233: 
0234: Tensor bucketize_cpu(const Tensor& self, const Tensor& boundaries, bool out_int32, bool right) {
0235:   ScalarType scalar_type = out_int32 ? ScalarType::Int : ScalarType::Long;
0236:   c10::TensorOptions options = TensorOptions().device(self.options().device()).dtype(scalar_type);
0237:   Tensor result = at::empty({0}, options, MemoryFormat::Contiguous);
0238:   at::native::bucketize_out_cpu(self, boundaries, out_int32, right, result);
0239:   return result;
0240: }
```
- **EN**: Lines 211-240 mainly cover expressions/calls, state/variable declarations, return paths. Notable symbols: TensorOptions, device, options, dtype.
- **CN**: 第 211-240 行主要涉及表达式或调用、变量/别名声明、返回路径。 值得关注的符号包括：TensorOptions, device, options, dtype。

### Lines 241-246 / 第 241-246 行
```cpp
0241: 
0242: Tensor bucketize_cpu(const Scalar& self, const Tensor& boundaries, bool out_int32, bool right) {
0243:   return bucketize_cpu(searchsorted_scalar_tensor(self, boundaries.device()), boundaries, out_int32, right);
0244: }
0245: 
0246: } // namespace at::native
```
- **EN**: Lines 241-246 mainly cover function signatures/definitions, return paths, expressions/calls. Notable symbols: bucketize_cpu, searchsorted_scalar_tensor, device.
- **CN**: 第 241-246 行主要涉及函数签名或实现、返回路径、表达式或调用。 值得关注的符号包括：bucketize_cpu, searchsorted_scalar_tensor, device。

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
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/Dispatch.h>`, `<ATen/Parallel.h>`, `<ATen/native/BucketizationUtils.h>`, `<ATen/native/Resize.h>`, `<c10/util/irange.h>`, `<ATen/Functions.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/bucketize_native.h>`, `<ATen/ops/searchsorted_native.h>` ...
- **Macros / 宏**: `TORCH_CHECK`, `AT_DISPATCH`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
