# GroupedMMUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File / 文件**: `aten/src/ATen/native/GroupedMMUtils.h`
- **Repository / 仓库**: `/root/xw/pytorch/`
- **Purpose (EN)**: Implements or declares ATen native logic related to Grouped MMUtils. As a header, it exposes declarations and shared helpers.
- **Purpose (CN)**: 实现或声明与 grouped、mmutils 相关的 ATen 原生逻辑。 作为头文件，它暴露声明与共享辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```cpp
0001: #pragma once
0002: 
0003: #include <ATen/core/Tensor.h>
0004: #include <ATen/TensorUtils.h>
0005: 
0006: #ifndef AT_PER_OPERATOR_HEADERS
0007: #include <ATen/CPUFunctions.h>
0008: #include <ATen/Functions.h>
0009: #include <ATen/NativeFunctions.h>
0010: #else
0011: #include <ATen/ops/bmm.h>
0012: #include <ATen/ops/empty.h>
0013: #include <ATen/ops/empty_strided.h>
0014: #include <ATen/ops/mm.h>
0015: #include <ATen/ops/zeros.h>
0016: #endif
0017: 
0018: namespace at::native {
0019: 
0020: inline bool check_valid_strides_and_return_transposed(const Tensor& mat) {
0021:   IntArrayRef tensor_strides = mat.strides();
0022:   IntArrayRef tensor_sizes = mat.sizes();
0023:   int end_dim = mat.dim() - 1;
0024:   int alignment = 16 / mat.element_size();
0025:   bool is_cpu = mat.device().is_cpu();
0026:   TORCH_CHECK(is_cpu || uint64_t(mat.data_ptr()) % 16 == 0, "expected data_ptr to be aligned to 16 bytes");
0027:   if ((tensor_strides[end_dim - 1] == 1) && (tensor_strides[end_dim] >= std::max<int64_t>(1, tensor_sizes[end_dim - 1]))) {
0028:     TORCH_CHECK(tensor_strides[end_dim] % alignment == 0, "strides should be multiple of 16 bytes");
0029:     return true;
0030:   } else if ((tensor_strides[end_dim] == 1) && (tensor_strides[end_dim - 1] >= std::max<int64_t>(1, tensor_sizes[end_dim]))) {
```
- **EN**: Lines 1-30 mainly cover header inclusion, state/variable declarations, macro-based glue. Notable symbols: check_valid_strides_and_return_transposed, strides, sizes, dim.
- **CN**: 第 1-30 行主要涉及头文件包含、变量/别名声明、宏定义或宏调用。 值得关注的符号包括：check_valid_strides_and_return_transposed, strides, sizes, dim。

### Lines 31-60 / 第 31-60 行
```cpp
0031:     TORCH_CHECK(tensor_strides[end_dim - 1] % alignment == 0, "strides should be multiple of 16 bytes");
0032:     return false;
0033:   } else {
0034:     TORCH_CHECK(false, "Invalid strides/sizes, got ", mat.strides(), " for strides and ", mat.sizes(), " for sizes");
0035:   }
0036: }
0037: 
0038: inline at::Tensor create_grouped_gemm_output_tensor(const Tensor& mat_a,
0039: const Tensor& mat_b,
0040: const std::optional<at::Tensor>& offs,
0041: c10::ScalarType out_dtype
0042: ) {
0043:   c10::SmallVector<int64_t, 3> out_size;
0044:   const bool a_is_2d = mat_a.dim() == 2;
0045:   const bool b_is_2d = mat_b.dim() == 2;
0046:   if (a_is_2d) {
0047:     if (b_is_2d) {
0048:       out_size = {offs->size(0), mat_a.size(0), mat_b.size(1)};
0049:     } else {
0050:       TORCH_CHECK(offs->size(0) == mat_b.size(0), "matrix batch sizes have to match");
0051:       out_size = {mat_a.size(0), mat_b.size(-1)};
0052:     }
0053:   } else {
0054:     if (b_is_2d) {
0055:       // this case is not actually encountered for MoE gemms
0056:       TORCH_CHECK(offs->size(0) == mat_a.size(0), "matrix batch sizes have to match");
0057:       out_size = {mat_a.size(1), mat_b.size(1)};
0058:     } else { // regular bmm
0059:       TORCH_CHECK(mat_a.size(0) == mat_b.size(0), "batched dimension has to match");
0060:       out_size = {mat_a.size(0), mat_a.size(1), mat_b.size(-1)};
```
- **EN**: Lines 31-60 mainly cover state/variable declarations, expressions/calls, macro-based glue. Notable symbols: TORCH_CHECK, strides, sizes, create_grouped_gemm_output_tensor.
- **CN**: 第 31-60 行主要涉及变量/别名声明、表达式或调用、宏定义或宏调用。 值得关注的符号包括：TORCH_CHECK, strides, sizes, create_grouped_gemm_output_tensor。

### Lines 61-90 / 第 61-90 行
```cpp
0061:     }
0062:   }
0063: 
0064:   #ifndef USE_ROCM
0065:   // For TMA transfers, strides of output tensor have to be either
0066:   // 1, or aligned to 16 bytes.
0067:   const auto last_dim = out_size.size() - 1;
0068:   const auto alignment = 16 / c10::elementSize(out_dtype);
0069:   const int64_t size_padded = (out_size[last_dim] + alignment - 1) / alignment * alignment;
0070:   std::vector<int64_t> out_stride;
0071:   if (a_is_2d != b_is_2d) {
0072:     out_stride = {size_padded, 1};
0073:   } else {
0074:     out_stride = {out_size[1] * size_padded, size_padded, 1};
0075:   }
0076:   return at::empty_strided(out_size, out_stride, mat_a.options().dtype(out_dtype));
0077:   #else
0078:   // For ROCm 2D-2D case (output is 3D), zero-initialize to handle K=0 or small K
0079:   // groups correctly. When K=0, the mathematically correct result is zeros,
0080:   // but CK kernel may not write to the output region.
0081:   if (a_is_2d && b_is_2d) {
0082:     return at::zeros(out_size, mat_a.options().dtype(out_dtype));
0083:   }
0084:   return at::empty(out_size, mat_a.options().dtype(out_dtype));
0085:   #endif
0086: }
0087: 
0088: inline void _grouped_mm_validate_inputs(const Tensor& mat_a, const Tensor& mat_b,
0089: const std::optional<at::Tensor>& offs,
0090: const std::optional<at::Tensor>& bias,
```
- **EN**: Lines 61-90 mainly cover expressions/calls, state/variable declarations, comments/documentation. Notable symbols: size, elementSize, empty_strided, options.
- **CN**: 第 61-90 行主要涉及表达式或调用、变量/别名声明、注释或说明。 值得关注的符号包括：size, elementSize, empty_strided, options。

### Lines 91-120 / 第 91-120 行
```cpp
0091: std::optional<c10::ScalarType> out_dtype) {
0092:   TORCH_CHECK((mat_a.dtype() == at::kBFloat16) || (mat_a.dtype() == at::kFloat) || (mat_a.dtype() == at::kHalf), "Expected mat_a to be Float32, BFloat16 or Float16 matrix, got ", mat_a.scalar_type());
0093:   TORCH_CHECK((mat_b.dtype() == at::kBFloat16) || (mat_b.dtype() == at::kFloat) || (mat_b.dtype() == at::kHalf), "Expected mat_b to be Float32, BFloat16 or Float16 matrix, got ", mat_b.scalar_type());
0094:   TORCH_CHECK(mat_a.dim() == 2 || mat_a.dim() == 3, "mat_a has to be 2 or 3d");
0095:   TORCH_CHECK(mat_b.dim() == 2 || mat_b.dim() == 3, "mat_b has to be 2 or 3d");
0096:   const bool a_is_2d = mat_a.dim() == 2;
0097:   const bool b_is_2d = mat_b.dim() == 2;
0098:   if (!a_is_2d || !b_is_2d) {
0099:     TORCH_CHECK(mat_a.size(-1) == mat_b.size(-2), "contraction dimension of mat_a and mat_b must match");
0100:   }
0101: 
0102:   // check that the strides are valid, the fn will throw an error if not
0103:   check_valid_strides_and_return_transposed(mat_a);
0104:   check_valid_strides_and_return_transposed(mat_b);
0105:   TORCH_CHECK(offs.has_value() ==  (a_is_2d || b_is_2d), "Have to provide offsets if there is a 2d matrix, or no offset if both matrices are 3d");
0106: 
0107:   if (offs.has_value()) {
0108:     TORCH_CHECK(offs->dim() == 1, "offs has to be 1D");
0109:     TORCH_CHECK(offs->dtype() == at::kInt, "Offsets have to be int32");
0110:   }
0111:   TORCH_CHECK(!bias.has_value(), "Bias not supported yet");
0112: }
0113: 
0114: inline c10::ScalarType _resolve_grouped_mm_out_dtype(const Tensor& mat_a, [[maybe_unused]] const Tensor& mat_b,
0115: std::optional<c10::ScalarType> out_dtype) {
0116:   const auto out_dtype_ = out_dtype.value_or(mat_a.scalar_type());
0117:   // TODO(future PR): enable float32 output dtype for bfloat16 and float16 inputs
0118:   TORCH_CHECK(out_dtype_ == mat_a.dtype(), "Grouped gemm output dtype must match `mat_a` dtype");
0119:   return out_dtype_;
0120: }
```
- **EN**: Lines 91-120 mainly cover macro-based glue, state/variable declarations, expressions/calls. Notable symbols: TORCH_CHECK, dtype, scalar_type, dim.
- **CN**: 第 91-120 行主要涉及宏定义或宏调用、变量/别名声明、表达式或调用。 值得关注的符号包括：TORCH_CHECK, dtype, scalar_type, dim。

### Lines 121-150 / 第 121-150 行
```cpp
0121: 
0122: 
0123: inline void _grouped_mm_fallback(const Tensor& mat_a, const Tensor& mat_b,
0124: const std::optional<at::Tensor>& offs,
0125: const std::optional<at::Tensor>& bias,
0126: std::optional<c10::ScalarType> out_dtype,
0127: Tensor out) {
0128:   LOG(INFO) << "fallback path for `torch._grouped_mm`, performance may not be optimal";
0129:   const bool a_is_2d = mat_a.dim() == 2;
0130:   const bool b_is_2d = mat_b.dim() == 2;
0131:   if (a_is_2d && !b_is_2d) {
0132:     // 2d x 3d with offsets
0133:     int group_start_idx = 0;
0134:     auto offs_cpu = offs.value().cpu();
0135:     for (int group_idx = 0; group_idx < offs_cpu.size(0); group_idx++) {
0136:       int group_end_idx = offs_cpu[group_idx].item<int>();
0137:       auto mat_a_slice = mat_a.slice(0, group_start_idx, group_end_idx);
0138:       auto out_slice = out.slice(0, group_start_idx, group_end_idx);
0139:       at::mm_out(out_slice, mat_a_slice, mat_b[group_idx]);
0140:       group_start_idx = group_end_idx;
0141:     }
0142: 
0143:   } else if (!a_is_2d && b_is_2d) {
0144:     // 3d x 2d with offsets
0145:     int group_start_idx = 0;
0146:     auto offs_cpu = offs.value().cpu();
0147:     for (int group_idx = 0; group_idx < offs_cpu.size(0); group_idx++) {
0148:       int group_end_idx = offs_cpu[group_idx].item<int>();
0149:       auto mat_b_slice = mat_b.slice(1, group_start_idx, group_end_idx);
0150:       auto out_slice = out.slice(1, group_start_idx, group_end_idx);
```
- **EN**: Lines 121-150 mainly cover state/variable declarations, expressions/calls, control-flow checks. Notable symbols: _grouped_mm_fallback, LOG, dim, value.
- **CN**: 第 121-150 行主要涉及变量/别名声明、表达式或调用、控制流逻辑。 值得关注的符号包括：_grouped_mm_fallback, LOG, dim, value。

### Lines 151-175 / 第 151-175 行
```cpp
0151:       at::mm_out(out_slice, mat_a[group_idx], mat_b_slice);
0152:       group_start_idx = group_end_idx;
0153:     }
0154: 
0155:   } else if (a_is_2d && b_is_2d) {
0156:     // 2d x 2d with offsets
0157:     int group_start_idx = 0;
0158:     auto offs_cpu = offs.value().cpu();
0159:     for (int group_idx = 0; group_idx < offs_cpu.size(0); group_idx++) {
0160:       int group_end_idx = offs_cpu[group_idx].item<int>();
0161:       auto mat_a_slice = mat_a.slice(1, group_start_idx, group_end_idx);
0162:       auto mat_b_slice = mat_b.slice(0, group_start_idx, group_end_idx);
0163:       auto out_slice = out[group_idx];
0164:       at::mm_out(out_slice, mat_a_slice, mat_b_slice);
0165:       group_start_idx = group_end_idx;
0166:     }
0167: 
0168:   } else {
0169:     // 3d x 3d without offsets - regular bmm
0170:     at::bmm_out(out, mat_a, mat_b);
0171:   }
0172: }
0173: 
0174: 
0175: } // namespace at::native
```
- **EN**: Lines 151-175 mainly cover state/variable declarations, expressions/calls, function signatures/definitions. Notable symbols: mm_out, value, cpu, size.
- **CN**: 第 151-175 行主要涉及变量/别名声明、表达式或调用、函数签名或实现。 值得关注的符号包括：mm_out, value, cpu, size。

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
- **Headers / 头文件**: `<ATen/core/Tensor.h>`, `<ATen/TensorUtils.h>`, `<ATen/CPUFunctions.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/bmm.h>`, `<ATen/ops/empty.h>`, `<ATen/ops/empty_strided.h>`, `<ATen/ops/mm.h>`, `<ATen/ops/zeros.h>` ...
- **Macros / 宏**: `TORCH_CHECK`
- **Namespaces / 命名空间**: `at::native`, `at::`, `c10::`, `std::`
