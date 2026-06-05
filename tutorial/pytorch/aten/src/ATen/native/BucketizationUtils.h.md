# BucketizationUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/BucketizationUtils.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Bucketization Utils. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 bucketization、utils 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <ATen/core/Tensor.h>
0004: #include <ATen/native/TypeProperties.h>
0005: #include <ATen/ScalarOps.h>
0006: 
0007: #ifndef AT_PER_OPERATOR_HEADERS
0008: #include <ATen/NativeFunctions.h>
0009: #else
0010: #include <ATen/ops/result_type.h>
0011: #endif
0012: 
0013: namespace at::native {
0014: 
0015: // original values given by raw_*. If an original value is not contiguous, will make a contiguous copy to
0016: // the corresponding trimmed_* value. Additionally, if the dtypes of the boundary and input tensor do not
0017: // match, will change them to be a common super type so comparisons are done between the same types.
0018: // For any trimmed_* tensor, if its outgoing value matches what it was incoming (typically null), then the
0019: // corresponding raw_* version should be used since it was already contiguous of the right type.
0020: inline void searchsorted_maybe_trim_input_tensors(
0021:     Tensor& trimmed_input,
0022:     Tensor& trimmed_boundaries,
0023:     Tensor& trimmed_sorter,
0024:     const Tensor& raw_input,
0025:     const Tensor& raw_boundaries,
0026:     const Tensor& raw_sorter) {
0027:   bool in_is_contiguous = raw_input.is_contiguous();
0028:   bool bd_is_contiguous = raw_boundaries.is_contiguous();
0029:   bool sort_is_contiguous = raw_sorter.is_contiguous();
0030: 
```
- **EN**: Lines 1-30 mainly cover header inclusion, comments/documentation, expressions/calls. Notable symbols: incoming, searchsorted_maybe_trim_input_tensors, is_contiguous.
- **CN**: 第 1-30 行主要涉及头文件包含、注释或说明、表达式或调用。 值得关注的符号包括：incoming, searchsorted_maybe_trim_input_tensors, is_contiguous。

### Lines 31-60 / 第 31-60 行
```cpp
0031:   if (!in_is_contiguous) {
0032:     TORCH_WARN_ONCE("torch.searchsorted(): input value tensor is non-contiguous, this will lower the performance due "
0033:       "to extra data copy when converting non-contiguous tensor to contiguous, please use contiguous input value "
0034:       "tensor if possible. This message will only appear once per program.");
0035:     trimmed_input = raw_input.contiguous();
0036:   }
0037:   if (!bd_is_contiguous) {
0038:     TORCH_WARN_ONCE("torch.searchsorted(): boundary tensor is non-contiguous, this will lower the performance due "
0039:       "to extra data copy when converting non-contiguous tensor to contiguous, please use contiguous boundary "
0040:       "tensor if possible. This message will only appear once per program.");
0041:     trimmed_boundaries = raw_boundaries.contiguous();
0042:   }
0043:   if (!sort_is_contiguous) {
0044:     TORCH_WARN_ONCE("torch.searchsorted(): sorter tensor is non-contiguous, this will lower the performance due "
0045:       "to extra data copy when converting non-contiguous tensor to contiguous, please use contiguous sorter "
0046:       "tensor if possible. This message will only appear once per program.");
0047:     trimmed_sorter = raw_sorter.contiguous();
0048:   }
0049:   if (raw_input.dtype() != raw_boundaries.dtype()) {
0050:     at::native::ResultTypeState state = {};
0051:     state = at::native::update_result_type_state(raw_boundaries, state);
0052:     state = at::native::update_result_type_state(raw_input, state);
0053:     ScalarType common_stype = at::native::result_type(state);
0054: 
0055:     TORCH_INTERNAL_ASSERT(common_stype != ScalarType::Undefined);
0056:     if (common_stype != raw_input.scalar_type()) {
0057:       trimmed_input = in_is_contiguous ? raw_input.to(common_stype) : trimmed_input.to(common_stype);
0058:     }
0059:     if (common_stype != raw_boundaries.scalar_type()) {
0060:       trimmed_boundaries = bd_is_contiguous ? raw_boundaries.to(common_stype) : trimmed_boundaries.to(common_stype);
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: TORCH_WARN_ONCE, searchsorted, contiguous, dtype.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：TORCH_WARN_ONCE, searchsorted, contiguous, dtype。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     }
0062:   }
0063: }
0064: 
0065: /* unused but needed for internal jagged tensor class */
0066: inline void searchsorted_maybe_trim_input_tensors(
0067:     Tensor& trimmed_input,
0068:     Tensor& trimmed_boundaries,
0069:     const Tensor& raw_input,
0070:     const Tensor& raw_boundaries) {
0071:   Tensor trimmed_sorter;
0072:   Tensor raw_sorter;
0073:   searchsorted_maybe_trim_input_tensors(
0074:       trimmed_input,
0075:       trimmed_boundaries,
0076:       trimmed_sorter,
0077:       raw_input,
0078:       raw_boundaries,
0079:       raw_sorter);
0080: }
0081: 
0082: inline bool searchsorted_dims_matched_before_last_dim(const Tensor& boundaries, const Tensor& input) {
0083:   if (boundaries.dim() != input.dim()) {
0084:     return false;
0085:   }
0086:   const auto& dims_bd = boundaries.sizes();
0087:   const auto& dims_in = input.sizes();
0088:   for (int64_t dim = 0; dim + 1 < boundaries.dim(); ++dim) {
0089:     if (dims_bd[dim] != dims_in[dim]) {
0090:       return false;
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, control-flow checks. Notable symbols: searchsorted_maybe_trim_input_tensors, searchsorted_dims_matched_before_last_dim, dim, sizes.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、控制流逻辑。 值得关注的符号包括：searchsorted_maybe_trim_input_tensors, searchsorted_dims_matched_before_last_dim, dim, sizes。

### Lines 91-120 / 第 91-120 行
```cpp
0091:     }
0092:   }
0093:   return true;
0094: }
0095: 
0096: inline Tensor searchsorted_scalar_tensor(const Scalar& scalar, const c10::Device& device) {
0097:   auto tensor = c10::scalar_to_tensor(scalar, device);
0098:   // This is to adopt the scalar promotion rules defined in native/TypeProperties.h
0099:   // So we have the same type promotion rules as binary operations.
0100:   tensor.unsafeGetTensorImpl()->set_wrapped_number(true);
0101:   return tensor;
0102: }
0103: 
0104: inline void searchsorted_pre_check(
0105:     const Tensor& boundaries,
0106:     const Tensor& input,
0107:     const Tensor& output,
0108:     const bool out_int32,
0109:     const bool right,
0110:     const std::optional<std::string_view> side_opt,
0111:     const Tensor& sorter) {
0112:   if (side_opt) {
0113:     const std::string_view side = *side_opt;
0114:     TORCH_CHECK(side == "left" || side == "right", "torch.searchsorted(): side can only be 'left' or 'right' but ",
0115:       "got ", side);
0116: 
0117:     // assume the user has not explicitly set (right=False, side="right")
0118:     TORCH_CHECK(!right || side == "right", "torch.searchsorted(): side and right can't be set to opposites, got side "
0119:     "of ", side, " while right was True");
0120:   }
```
- **EN**: Lines 91-120 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: searchsorted_scalar_tensor, scalar_to_tensor, unsafeGetTensorImpl, set_wrapped_number.
- **CN**: 第 91-120 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：searchsorted_scalar_tensor, scalar_to_tensor, unsafeGetTensorImpl, set_wrapped_number。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122:   TORCH_CHECK(boundaries.device() == input.device(), "torch.searchsorted(): boundaries and input value tensors ",
0123:     "should have same device type, but got boundaries tensor device type ", boundaries.device(), " and input value ",
0124:     "tensor device type ", input.device());
0125: 
0126:   if (sorter.defined()) {
0127:     TORCH_CHECK(sorter.device() == boundaries.device(), "torch.searchsorted(): sorter and boundary tensors should ",
0128:       "have same device type, but got sorter tensor device type ", sorter.device(), " and input value tensor ",
0129:       "device type ", boundaries.device());
0130: 
0131:     TORCH_CHECK(sorter.sizes() == boundaries.sizes(), "torch.searchsorted(): boundary and sorter must have the same "
0132:       "size, but got boundary tensor ", boundaries.sizes(), "and got sorter tensor ", sorter.sizes());
0133: 
0134:     TORCH_CHECK(sorter.scalar_type() == ScalarType::Long, "torch.searchsorted(): sorter must be a tensor of long ",
0135:       "dtype but got dtype ", sorter.scalar_type());
0136: 
0137:     if (sorter.numel() > 0) {
0138:       auto minmax = sorter.aminmax();
0139:       int64_t vmin = std::get<0>(minmax).item().toLong();
0140:       int64_t vmax = std::get<1>(minmax).item().toLong();
0141:       TORCH_CHECK(vmin >= 0 && vmax < sorter.sizes().back(), "torch.searchsorted(): sorter index out of range");
0142:     }
0143:   }
0144: 
0145:   TORCH_CHECK(input.dim() > 0 || (input.dim() == 0 && input.numel() == 1 && boundaries.dim() == 1),
0146:     "torch.searchsorted(): input value can be a scalar only when boundaries tensor dimension is 1, but we got ",
0147:     "boundaries tensor dim(", boundaries.dim(), ") and input value's dim(", input.dim(), ") numel(",
0148:     input.numel(), ")");
0149: 
0150:   TORCH_CHECK(boundaries.dim() != 0, "torch.searchsorted(): boundaries tensor should have positive dimension, but ",
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, macro-based glue, function signatures/definitions. Notable symbols: TORCH_CHECK, device, searchsorted, defined.
- **CN**: 第 121-150 行主要涉及变量/别名声明、宏定义或宏调用、函数签名或实现。 值得关注的符号包括：TORCH_CHECK, device, searchsorted, defined。

### Lines 151-173 / 第 151-173 行
```cpp
0151:     "got 0 dimension");
0152: 
0153:   TORCH_CHECK(boundaries.dim() == 1 || searchsorted_dims_matched_before_last_dim(boundaries, input),
0154:     "torch.searchsorted(): boundaries tensor should be 1 dimension or the first N-1 dimensions of boundaries tensor ",
0155:     "and input value tensor must match, but we got boundaries tensor ", boundaries.sizes(), " and input value tensor ",
0156:     input.sizes());
0157: 
0158:   ScalarType output_dtype = output.scalar_type();
0159:   TORCH_CHECK(
0160:       (output_dtype == ScalarType::Long && !out_int32) ||
0161:           (output_dtype == ScalarType::Int && out_int32),
0162:       "torch.searchsorted(): output tensor's dtype is wrong, it can only be Int(int32) or Long(int64) depending on ",
0163:       "whether out_int32 flag is True, but we got output tensor's dtype ", output_dtype,
0164:       " and out_int32 flag is ", (out_int32 ? "True" : "False"));
0165: 
0166:   if (out_int32) {
0167:     TORCH_CHECK(boundaries.sizes().back() < INT_MAX,
0168:       "torch.searchsorted(): the size of boundaries' last dimension should be less than ", INT_MAX, ", but we got ",
0169:       boundaries.sizes().back());
0170:   }
0171: }
0172: 
0173: } // namespace at::native
```
- **EN**: Lines 151-173 mainly cover function signatures/definitions, state/variable declarations, macro-based glue. Notable symbols: TORCH_CHECK, dim, searchsorted_dims_matched_before_last_dim, searchsorted.
- **CN**: 第 151-173 行主要涉及函数签名或实现、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, dim, searchsorted_dims_matched_before_last_dim, searchsorted。

## Key Concepts / 关键概念
- **EN**: Runtime validation with TORCH_CHECK  
  **CN**: 使用 TORCH_CHECK 进行运行时校验
- **EN**: ATen namespace layering  
  **CN**: ATen 命名空间分层
- **EN**: Tensor-centric operator implementation  
  **CN**: 以 Tensor 为中心的算子实现
- **EN**: Scalar/tensor mixed arithmetic  
  **CN**: 标量与张量混合运算
- **EN**: Native operator implementation path  
  **CN**: 原生算子实现路径

## Dependencies / 依赖关系
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/native/TypeProperties.h>`, `<ATen/ScalarOps.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/result_type.h>`
- **Macros / 宏**: `TORCH_CHECK`, `TORCH_WARN`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
