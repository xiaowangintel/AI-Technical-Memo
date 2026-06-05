# GroupedBlas.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/GroupedBlas.cpp`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Provides host-side CUDA entry points, orchestration, or dispatch glue around `_scaled_mm_allowed_device`, `_mx8_mx8_bf16_grouped_mm_mslk`, `_f8_f8_bf16_rowwise_grouped_mm_cuda`, `_f4_f4_bf16_grouped_mm_mslk`.
- 用途（中文）: 提供围绕 `_scaled_mm_allowed_device`, `_mx8_mx8_bf16_grouped_mm_mslk`, `_f8_f8_bf16_rowwise_grouped_mm_cuda`, `_f4_f4_bf16_grouped_mm_mslk` 的主机端 CUDA 入口、调度编排或分发胶水代码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
   1: #include <cstdint>
   2: #include <c10/util/typeid.h>
   3: #include <c10/util/Exception.h>
   4: #include <c10/util/SmallVector.h>
   5: #include <c10/core/Scalar.h>
   6: #include <c10/core/ScalarType.h>
   7: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   8: #include <ATen/Context.h>
   9: #include <ATen/core/Tensor.h>
  10: #include <ATen/core/NamedTensor.h>
  11: #include <ATen/Dispatch.h>
  12: #include <ATen/ExpandUtils.h>
  13: #include <ATen/OpMathType.h>
  14: #include <ATen/TensorUtils.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cstdint>`, `<c10/util/typeid.h>`, `<c10/util/Exception.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cstdint>`, `<c10/util/typeid.h>`, `<c10/util/Exception.h>`。

### Lines 15-28
```cpp
  15: #include <ATen/cuda/CUDABlas.h>
  16: #include <ATen/native/ScaledBlasUtils.h>
  17: #include <ATen/cuda/tunable/Tunable.h>
  18: #include <ATen/cuda/tunable/TunableGemm.h>
  19: #include <ATen/native/Resize.h>
  20: #include <c10/util/MaybeOwned.h>
  21: #include <ATen/native/GroupedMMUtils.h>
  22: #include <ATen/native/cuda/RowwiseScaledMM.h>
  23: #include <ATen/native/cuda/ScaledGroupMM.h>
  24: #include <ATen/native/cuda/GroupMM.h>
  25: #ifdef USE_ROCM
  26: #include <ATen/native/hip/ck_group_gemm.h>
  27: #endif
  28: #include <ATen/ceil_div.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/cuda/CUDABlas.h>`, `<ATen/native/ScaledBlasUtils.h>`, `<ATen/cuda/tunable/Tunable.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/cuda/CUDABlas.h>`, `<ATen/native/ScaledBlasUtils.h>`, `<ATen/cuda/tunable/Tunable.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-42
```cpp
  29: 
  30: #ifdef USE_MSLK
  31: #include <mslk/gemm/gemm_torch.h>
  32: #endif
  33: 
  34: #ifndef AT_PER_OPERATOR_HEADERS
  35: #include <ATen/Functions.h>
  36: #include <ATen/NativeFunctions.h>
  37: #else
  38: #include <ATen/ops/_addmm_activation_native.h>
  39: #include <ATen/ops/_efficientzerotensor.h>
  40: #include <ATen/ops/_grouped_mm_native.h>
  41: #include <ATen/ops/_scaled_mm_native.h>
  42: #include <ATen/ops/_unsafe_view_native.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<mslk/gemm/gemm_torch.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<mslk/gemm/gemm_torch.h>`, `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 43-56
```cpp
  43: #include <ATen/ops/abs.h>
  44: #include <ATen/ops/addmm_native.h>
  45: #include <ATen/ops/addmv_native.h>
  46: #include <ATen/ops/baddbmm_native.h>
  47: #include <ATen/ops/bmm_native.h>
  48: #include <ATen/ops/copy_native.h>
  49: #include <ATen/ops/dot_native.h>
  50: #include <ATen/ops/empty.h>
  51: #include <ATen/ops/empty_strided.h>
  52: #include <ATen/ops/gelu.h>
  53: #include <ATen/ops/max.h>
  54: #include <ATen/ops/mm_native.h>
  55: #include <ATen/ops/mul.h>
  56: #include <ATen/ops/relu.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/abs.h>`, `<ATen/ops/addmm_native.h>`, `<ATen/ops/addmv_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/abs.h>`, `<ATen/ops/addmm_native.h>`, `<ATen/ops/addmv_native.h>`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 57-60
```cpp
  57: #include <ATen/ops/ones.h>
  58: #include <ATen/ops/scalar_tensor_native.h>
  59: #include <ATen/ops/vdot_native.h>
  60: #endif
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/ones.h>`, `<ATen/ops/scalar_tensor_native.h>`, `<ATen/ops/vdot_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/ones.h>`, `<ATen/ops/scalar_tensor_native.h>`, `<ATen/ops/vdot_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 62-63
```cpp
  62: using at::blas::ScalingType;
  63: using at::blas::SwizzleType;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 65-67
```cpp
  65: namespace scaled_blas = at::native::scaled;
  66: using scaled_blas::ScaledGemmImplementation;
  67: using scaled_blas::convert_int_to_enum;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 69-90
```cpp
  69: namespace at::native {
  70: 
  71: namespace {
  72: 
  73: bool _scaled_mm_allowed_device(bool sm90_only=false, bool sm100_only=false) {
  74: #ifdef USE_ROCM
  75:   static const std::vector<std::string> archs = {
  76:     "gfx942",
  77: #if ROCM_VERSION >= 60300
  78:     "gfx1200", "gfx1201",
  79: #endif
  80: #if ROCM_VERSION >= 60500
  81:     "gfx950"
  82: #endif
  83: };
  84:   return at::detail::getCUDAHooks().isGPUArch(archs);
  85: #else
  86:   auto dprops = at::cuda::getCurrentDeviceProperties();
  87: 
  88:   if (sm90_only || sm100_only) {
  89:     return (sm90_only && dprops->major == 9) || (sm100_only && dprops->major == 10);
  90:   } else {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `_scaled_mm_allowed_device`.
- CN: 该代码块定义或继续实现 `_scaled_mm_allowed_device`。

### Lines 91-94
```cpp
  91:     return dprops->major >= 9 || (dprops->major == 8 && dprops->minor == 9);
  92:   }
  93: #endif
  94: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 96-98
```cpp
  96: // 2d-2d and 2d-3d
  97: // scaling=MXFP8
  98: // CUDA-only
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 99-120
```cpp
  99: Tensor&
 100: _mx8_mx8_bf16_grouped_mm_mslk(
 101:         const Tensor& mat_a,
 102:         const Tensor& mat_b,
 103:         const Tensor& scale_a,
 104:         const SwizzleType swizzle_a,
 105:         const Tensor& scale_b,
 106:         const SwizzleType swizzle_b,
 107:         const std::optional<at::Tensor>& offs,
 108:         Tensor& out) {
 109:     const bool a_is_2d = mat_a.dim() == 2;
 110:     const bool b_is_2d = mat_b.dim() == 2;
 111:     bool b_is_3d = mat_b.dim() == 3;
 112:     bool is_2d_2d = a_is_2d && b_is_2d;
 113:     bool is_2d_3d = a_is_2d && b_is_3d;
 114:     TORCH_CHECK_VALUE(is_2d_2d || is_2d_3d, "MXFP8 grouped GEMM currently only supports 2d-2d and 2d-3d cases");
 115:     TORCH_CHECK_VALUE(offs.has_value(), "MXFP8 2d-2d and 2d-3d grouped GEMMs requires offsets");
 116:     TORCH_CHECK_VALUE(out.scalar_type() == at::kBFloat16, "Only bf16 out_dtype is supported for MXFP8 grouped gemm");
 117:     // MXFP8 expects float8_e8m0fnu scales.
 118:     TORCH_CHECK_VALUE(scale_a.scalar_type() == at::kFloat8_e8m0fnu && scale_b.scalar_type() == at::kFloat8_e8m0fnu,
 119:         "For MXFP8 grouped gemm, both scales must be float8_e8m0fnu tensors.");
 120: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_mx8_mx8_bf16_grouped_mm_mslk`.
- CN: 该代码块定义或继续实现 `_mx8_mx8_bf16_grouped_mm_mslk`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 121-126
```cpp
 121:     TORCH_CHECK_VALUE(swizzle_a == SwizzleType::NO_SWIZZLE && swizzle_b == SwizzleType::NO_SWIZZLE,
 122:         "For ROCM MXFP8 grouped gemm, both scale swizzle types must be SWIZZLE_NONE");
 123: #else
 124:     TORCH_CHECK_VALUE(swizzle_a == SwizzleType::SWIZZLE_32_4_4 && swizzle_b == SwizzleType::SWIZZLE_32_4_4,
 125:         "For CUDA MXFP8 grouped gemm, both scale swizzle types must be SWIZZLE_32_4_4");
 126: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 128-128
```cpp
 128: #if defined(USE_MSLK) and !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 129-140
```cpp
 129:     mslk::gemm::mx8mx8bf16_grouped_mm(
 130:         mat_a,
 131:         mat_b,
 132:         scale_a,
 133:         scale_b,
 134:         offs.value(),
 135:         out);
 136:     return out;
 137: #else
 138:     TORCH_CHECK_NOT_IMPLEMENTED(false, "mxfp8_mxfp8 grouped gemm requires compile with USE_MSLK");
 139: #endif
 140: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 142-144
```cpp
 142: // 2d-2d and 2d-3d cases
 143: // scaling=rowwise
 144: // CUDA-only
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 145-166
```cpp
 145: Tensor&
 146: _f8_f8_bf16_rowwise_grouped_mm_cuda(
 147:           const Tensor& mat_a,
 148:           const Tensor& mat_b,
 149:           const Tensor& scale_a,
 150:           const Tensor& scale_b,
 151:           const std::optional<Tensor>& offs,
 152:           const std::optional<Tensor>& bias,
 153:           const bool use_fast_accum,
 154:           Tensor& out) {
 155:   TORCH_CHECK_VALUE(mat_a.dtype() == at::kFloat8_e4m3fn, "Expected mat_a to be Float8_e4m3 matrix got ", mat_a.scalar_type());
 156:   TORCH_CHECK_VALUE(mat_b.dtype() == at::kFloat8_e4m3fn, "Expected mat_b to be Float8_e4m3 matrix got ", mat_b.scalar_type());
 157: 
 158:   at::cuda::detail::f8f8bf16_grouped_mm(
 159:       mat_a,
 160:       mat_b,
 161:       scale_a,
 162:       scale_b,
 163:       offs,
 164:       bias,
 165:       use_fast_accum,
 166:       out);
```
- EN: This block defines or continues the implementation of `_f8_f8_bf16_rowwise_grouped_mm_cuda`.
- CN: 该代码块定义或继续实现 `_f8_f8_bf16_rowwise_grouped_mm_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 167-168
```cpp
 167:     return out;
 168: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 170-172
```cpp
 170: // 2d-2d and 2d-3d cases
 171: // scaling=rowwise
 172: // only being called for rocm
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 173-173
```cpp
 173: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 174-195
```cpp
 174: Tensor&
 175: _f8_f8_bf16_rowwise_grouped_mm_rocm(
 176:       const Tensor& mat_a,
 177:       const Tensor& mat_b,
 178:       const Tensor& scale_a,
 179:       const Tensor& scale_b,
 180:       const std::optional<Tensor>& offs,
 181:       Tensor& out) {
 182:   bool is_gfx942 = at::detail::getCUDAHooks().isGPUArch({"gfx942"});
 183: 
 184:   if (is_gfx942) {
 185:     TORCH_CHECK_VALUE(mat_a.dtype() == at::kFloat8_e4m3fnuz, "Expected mat_a to be Float8_e4m3fnuz matrix got ", mat_a.scalar_type());
 186:     TORCH_CHECK_VALUE(mat_b.dtype() == at::kFloat8_e4m3fnuz, "Expected mat_b to be Float8_e4m3fnuz matrix got ", mat_b.scalar_type());
 187:   } else {
 188:     TORCH_CHECK_VALUE(mat_a.dtype() == at::kFloat8_e4m3fn, "Expected mat_a to be Float8_e4m3 matrix got ", mat_a.scalar_type());
 189:     TORCH_CHECK_VALUE(mat_b.dtype() == at::kFloat8_e4m3fn, "Expected mat_b to be Float8_e4m3 matrix got ", mat_b.scalar_type());
 190:   }
 191: 
 192: #if defined(USE_MSLK) && defined(USE_ROCM)
 193:   mslk::gemm::f8f8bf16_rowwise_grouped_mm(
 194:       mat_a,
 195:       // FBGEMM expects B matrix shape to be (.., N, K)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_f8_f8_bf16_rowwise_grouped_mm_rocm`.
- CN: 该代码块定义或继续实现 `_f8_f8_bf16_rowwise_grouped_mm_rocm`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 196-204
```cpp
 196:       mat_b.transpose(-2, -1),
 197:       scale_a,
 198:       scale_b,
 199:       offs,
 200:       out);
 201:   return out;
 202: #else
 203:   TORCH_CHECK_NOT_IMPLEMENTED(false, "grouped gemm is not supported without USE_MSLK on ROCM")
 204: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 206-207
```cpp
 206: }
 207: #endif // USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 209-209
```cpp
 209: // Dispatch f8 x f8 -> bf16 row-wise scaled to rocm/cuda
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 210-231
```cpp
 210: Tensor&
 211: _f8_f8_bf16_rowwise_grouped_mm(
 212:       const Tensor& mat_a,
 213:       const Tensor& mat_b,
 214:       const Tensor& scale_a,
 215:       const Tensor& scale_b,
 216:       const std::optional<Tensor>& offs,
 217:       const std::optional<Tensor>& bias,
 218:       bool use_fast_accum,
 219:       Tensor& out) {
 220:   // FP8 per-tensor and per-row scaling expect fp32 scales.
 221:   TORCH_CHECK_VALUE(scale_a.scalar_type() == kFloat && scale_b.scalar_type() == kFloat,
 222:       "For grouped FP8 rowwise, both scales must be float32 tensors");
 223: #ifndef USE_ROCM
 224:   return _f8_f8_bf16_rowwise_grouped_mm_cuda(
 225:       mat_a,
 226:       mat_b,
 227:       scale_a,
 228:       scale_b,
 229:       offs,
 230:       bias,
 231:       use_fast_accum,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_f8_f8_bf16_rowwise_grouped_mm`.
- CN: 该代码块定义或继续实现 `_f8_f8_bf16_rowwise_grouped_mm`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 232-244
```cpp
 232:       out);
 233: #else
 234:   // NOTE: ignore use_fast_accum
 235:   TORCH_CHECK_VALUE(!bias.has_value(), "ROCM grouped gemm does not support bias")
 236:   return _f8_f8_bf16_rowwise_grouped_mm_rocm(
 237:       mat_a,
 238:       mat_b,
 239:       scale_a,
 240:       scale_b,
 241:       offs,
 242:       out);
 243: #endif
 244: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 246-267
```cpp
 246: Tensor&
 247: _f4_f4_bf16_grouped_mm_mslk(
 248:       const Tensor& mat_a,
 249:       const Tensor& mat_b,
 250:       const Tensor& scale_a,
 251:       const std::optional<Tensor>& global_scale_a,
 252:       const Tensor& scale_b,
 253:       const std::optional<Tensor>& global_scale_b,
 254:       const std::optional<Tensor>& offs,
 255:       const std::optional<Tensor>& bias,
 256:       Tensor& out) {
 257: #if !defined(USE_ROCM) && defined(USE_MSLK)
 258:   // Typing checks
 259:   TORCH_CHECK_VALUE(mat_a.scalar_type() == at::kFloat4_e2m1fn_x2,
 260:       "mat_a must be Float4_e2n1fn_2, got: ", mat_a.scalar_type());
 261:   TORCH_CHECK_VALUE(mat_b.scalar_type() == at::kFloat4_e2m1fn_x2,
 262:       "mat_b must be Float4_e2n1fn_2, got: ", mat_b.scalar_type());
 263: 
 264:   std::optional<Tensor> combined_global_scale = std::nullopt;
 265:   if (global_scale_a.has_value() || global_scale_b.has_value()) {
 266:       // NVFP4
 267:       TORCH_CHECK_VALUE(global_scale_a.has_value() && global_scale_b.has_value(),
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_f4_f4_bf16_grouped_mm_mslk`.
- CN: 该代码块定义或继续实现 `_f4_f4_bf16_grouped_mm_mslk`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 268-281
```cpp
 268:           "For NVFP4 grouped gemm both of global_scale_{a,b} must have values")
 269:       TORCH_CHECK_VALUE(scale_a.scalar_type() == at::kFloat8_e4m3fn,
 270:           "scale_a must be Float8_e4m3fn, got: ", scale_a.scalar_type());
 271:       TORCH_CHECK_VALUE(scale_b.scalar_type() == at::kFloat8_e4m3fn,
 272:           "scale_b must be Float8_e4m3fn, got: ", scale_b.scalar_type());
 273:       TORCH_CHECK_VALUE(global_scale_a.value().scalar_type() == at::kFloat,
 274:           "global_scale_a must be Float, got: ", global_scale_a.value().scalar_type());
 275:       TORCH_CHECK_VALUE(global_scale_b.value().scalar_type() == at::kFloat,
 276:           "global_scale_b must be Float, got: ", global_scale_b.value().scalar_type());
 277:       combined_global_scale = global_scale_a.value().mul(global_scale_b.value());
 278:   } else {
 279:       // MXFP4
 280:       TORCH_CHECK_VALUE(scale_a.scalar_type() == at::kFloat8_e8m0fnu,
 281:           "scale_a must be Float8_e8m0fnu, got: ", scale_a.scalar_type());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 282-284
```cpp
 282:       TORCH_CHECK_VALUE(scale_b.scalar_type() == at::kFloat8_e8m0fnu,
 283:           "scale_b must be Float8_e8m0fnu, got: ", scale_b.scalar_type());
 284:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 286-294
```cpp
 286:   auto o = mslk::gemm::f4f4bf16_grouped_mm(
 287:       mat_a,
 288:       mat_b,
 289:       scale_a,
 290:       scale_b,
 291:       offs.value(),
 292:       out,
 293:       combined_global_scale
 294:   );
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 296-300
```cpp
 296:   return out;
 297: #else
 298:   TORCH_CHECK_NOT_IMPLEMENTED(false, "nvfp4 grouped gemm is not supported without USE_MSLK, and only for CUDA")
 299: #endif
 300: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 302-323
```cpp
 302: void _check_scales_fp8_rowwise(const Tensor& mat, const Tensor& scale, const int dim, const int arg_idx, const int scale_multiplier=1) {
 303:   // Checks scales for 2d or 3d target tensors (`mat`).
 304:   if (mat.dim() == 2) {
 305:     TORCH_CHECK(
 306:         scale.dim() == 1,
 307:         "scale must be a 1D tensor, but got ",
 308:         scale.dim(),
 309:         "D, arg ",
 310:         arg_idx);
 311:     TORCH_CHECK(
 312:         scale.is_contiguous(), "scale must be contiguous for arg ", arg_idx);
 313:     TORCH_CHECK(
 314:         scale.size(0) == mat.size(dim) * scale_multiplier,
 315:         "scale must have the same length as mat for arg ",
 316:         arg_idx);
 317:   } else {
 318:     TORCH_CHECK(
 319:         scale.dim() == 2,
 320:         "scale must be a 2D tensor, but got ",
 321:         scale.dim(),
 322:         "D for arg ",
 323:         arg_idx);
```
- EN: This block defines or continues the implementation of `_check_scales_fp8_rowwise`, `tensors`.
- CN: 该代码块定义或继续实现 `_check_scales_fp8_rowwise`, `tensors`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 324-337
```cpp
 324:     TORCH_CHECK(
 325:         scale.stride(1) == 1,
 326:         "scale must be contiguous in the last dimension for arg ",
 327:         arg_idx);
 328:     TORCH_CHECK(
 329:         scale.size(0) == mat.size(0),
 330:         "scale must have the same batch dimension as mat for arg ",
 331:         arg_idx);
 332:     TORCH_CHECK(
 333:         scale.size(1) == mat.size(1 + dim),
 334:         "scale must have the same first dimension as mat for arg ",
 335:         arg_idx);
 336:   }
 337: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 339-360
```cpp
 339: void _check_scales_blocked(const Tensor& mat, const Tensor& scale, const int dim, const int arg_idx) {
 340:   // if {mx,nv}fp4, will need to modify K later
 341:   bool is_fp4 = (mat.scalar_type() == kFloat4_e2m1fn_x2);
 342:   int blocksize = 32;
 343:   // check for nvfp4 vs. mxfp4 to fix blocksize
 344:   if (is_fp4 && scale.scalar_type() == kFloat8_e4m3fn) {
 345:     blocksize = 16;
 346:   }
 347: 
 348:   // Checks scales for 2d or 3d target tensors (`mat`).
 349:   if (mat.dim() == 2) {
 350:     // For MXFP8, 2d tensors have variable size groups represented as subtensors,
 351:     // that are converted to blocked padded format individually,
 352:     // so we can't check the scale sizes without doing a d2h sync to get the group sizes here.
 353:     TORCH_CHECK(
 354:       scale.dim() == mat.dim(),
 355:       "for block-scaled, scale must have same number of dimensions as parent tensor, but got mat.dim() = ", mat.dim(),
 356:       " and scale.dim() = ", scale.dim(), " for arg ", arg_idx
 357:     );
 358: 
 359:     // LHS mat shape (M, total_K) -> scale shape (rounded_up(M, 128), rounded_up_per_group(K/blocksize, 4))
 360:     // RHS mat shape (total_K, N) -> scale shape (rounded_up(N, 128), rounded_up_per_group(K/blocksize, 4))
```
- EN: This block defines or continues the implementation of `_check_scales_blocked`, `tensors`.
- CN: 该代码块定义或继续实现 `_check_scales_blocked`, `tensors`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 361-361
```cpp
 361:     //   * weight is transposed prior to the call, scale stays non-transposed.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 362-374
```cpp
 362:     bool LHS = arg_idx == 0;
 363:     int scale_dim_to_check = 0;
 364:     int mat_dim_to_check = LHS ? 0 : 1;
 365:     TORCH_CHECK(
 366:         scale.size(scale_dim_to_check) >= mat.size(mat_dim_to_check),
 367:         "for block-scaled, arg ", arg_idx, " tensor shape (", mat.size(0), ", ", mat.size(1), ") ",
 368:         "must have scale.shape[", scale_dim_to_check, "] >= ", mat.size(mat_dim_to_check), " but got scale.shape=(", scale.size(0), ", ", scale.size(1), ")");
 369:   } else {
 370:     // For MXFP8, 3d tensors have static group sizes (stack of 2d tensors),
 371:     // so we can check the exact expected scale sizes here without a d2h sync.
 372:     auto round_up = [](auto x, auto y) {
 373:         return ((x + y - 1) / y) * y;
 374:     };
```
- EN: This block defines or continues the implementation of `sizes`.
- CN: 该代码块定义或继续实现 `sizes`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 376-377
```cpp
 376:     // TODO: this is for 3d tensor in 2d-3d case specifically.
 377:     // We'll need to support 3d-3d and 3d-2d cases once mxfp8/nvfp4 grouped gemm supports them.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 378-388
```cpp
 378:     int64_t G = mat.size(0);
 379:     int64_t K = mat.size(1);
 380:     if (is_fp4) {
 381:       // FP4 packs 2 values into a single 8b word - the "real" K is 2x the
 382:       // reported K. Reverse that adjustment.
 383:       const int fp4_elems_per_byte = 2;
 384:       K *= fp4_elems_per_byte;
 385:     }
 386:     int64_t N = mat.size(2);
 387:     int64_t blocked_scale_K = round_up(K/blocksize, 4);
 388:     int64_t blocked_scale_N = round_up(N, 128);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 390-390
```cpp
 390:     // mslk expects stack of flattened blocked scales for 3d tensor, shape (G, blocked_scale_K * blocked_scale_N).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 391-402
```cpp
 391:     TORCH_CHECK(
 392:       scale.dim() == mat.dim() - 1,
 393:       "for block-scaled 2d-3d grouped GEMM, the 3d tensor of shape (G,K,N) must have a 2d scale of shape (G, blocked_scale_K * blocked_scale_N),",
 394:       "but scale is ", scale.dim(), "D for arg ", arg_idx
 395:     );
 396:     TORCH_CHECK(
 397:       scale.size(0) == G && scale.size(1) == blocked_scale_K * blocked_scale_N,
 398:       "for block-scaled grouped GEMM, the tensor shape (", G, ", ", K, ", ", N, ") must have scale shape (", G, ",", blocked_scale_K, ",", blocked_scale_N, ")",
 399:       " for arg ", arg_idx, ", got: ", scale.size(0), ", ", scale.size(1)
 400:     );
 401:   }
 402: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 404-414
```cpp
 404: void check_scale(const Tensor& mat, const Tensor& scale, const int dim, const int arg_idx, const int scale_multiplier=1) {
 405:   bool using_fp8_rowwise = scale.scalar_type() == kFloat;
 406:   bool using_mx = scale.scalar_type() == at::kFloat8_e8m0fnu;
 407:   if (using_fp8_rowwise) {
 408:     _check_scales_fp8_rowwise(mat, scale, dim, arg_idx, scale_multiplier);
 409:   } else if (using_mx) {
 410:     _check_scales_blocked(mat, scale, dim, arg_idx);
 411:   } else {
 412:     TORCH_CHECK(false, "scale must be float32 or float8_e8m0fnu, but got ", scale.dtype());
 413:   }
 414: }
```
- EN: This block defines or continues the implementation of `check_scale`.
- CN: 该代码块定义或继续实现 `check_scale`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 416-416
```cpp
 416: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 418-439
```cpp
 418: Tensor
 419: _scaled_grouped_mm_cuda(
 420:         const Tensor& mat_a,
 421:         const Tensor& mat_b,
 422:         const Tensor& scale_a,
 423:         const Tensor& scale_b,
 424:         const std::optional<at::Tensor>& offs,
 425:         const std::optional<at::Tensor>& bias,
 426:         const std::optional<at::Tensor>& scale_result,
 427:         std::optional<c10::ScalarType> out_dtype,
 428:         bool use_fast_accum) {
 429:   bool allowed_device = _scaled_mm_allowed_device(/*sm90_only*/true, /*sm100_only*/true);
 430:   TORCH_CHECK_VALUE(allowed_device, "torch._scaled_grouped_mm is only supported on CUDA devices with compute capability = [9.0, 10.0], or ROCm MI300+");
 431: 
 432:   TORCH_CHECK_VALUE(!check_valid_strides_and_return_transposed(mat_a), "Expected mat1 to not be transposed");
 433:   TORCH_CHECK_VALUE(check_valid_strides_and_return_transposed(mat_b), "Expected mat2 to be transposed");
 434:   TORCH_CHECK_VALUE(mat_a.dim() == 2 || mat_a.dim() == 3, "mat_a has to be 2 or 3d");
 435:   TORCH_CHECK_VALUE(mat_b.dim() == 2 || mat_b.dim() == 3, "mat_b has to be 2 or 3d");
 436:   const bool a_is_2d = mat_a.dim() == 2;
 437:   const bool b_is_2d = mat_b.dim() == 2;
 438: 
 439:   // NOTE(slayton): For sub-1B formats want contraction_dim argument?
```
- EN: This block defines or continues the implementation of `_scaled_grouped_mm_cuda`.
- CN: 该代码块定义或继续实现 `_scaled_grouped_mm_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 440-453
```cpp
 440:   if (!a_is_2d || !b_is_2d) {
 441:     TORCH_CHECK_VALUE(mat_a.size(-1) == mat_b.size(-2), "contraction dimension of mat_a and mat_b must match");
 442:   }
 443:   TORCH_CHECK_VALUE(
 444:     mat_a.size(-1) % 16 == 0,
 445:     "Expected trailing dimension of mat_a to be divisible by 16 ",
 446:     "but got mat1 shape: (",
 447:     mat_a.sizes(),
 448:     ").");
 449:   TORCH_CHECK_VALUE(mat_b.size(-2) % 16 == 0 && mat_b.size(-1) % 16 == 0,
 450:     "Expected mat_b shape to be divisible by 16 ",
 451:     "but got mat_b shape: (",
 452:     mat_b.sizes(),
 453:     ").");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 456-458
```cpp
 456:   TORCH_CHECK_VALUE(!bias.has_value(), "Bias not supported yet");
 457:   TORCH_CHECK_VALUE(!scale_result.has_value(), "Scale result not supported yet");
 458:   TORCH_CHECK_VALUE(offs.has_value() ==  (a_is_2d || b_is_2d), "Have to provide offsets if there is a 2d matrix");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 460-462
```cpp
 460:   // NOTE: mxfp8 x mxfp8 requires (and asserts later) that offsets is present.
 461:   //       for rowwise, no offsets implies 3d-3d and is handled by lower-level
 462:   //       routines
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 463-472
```cpp
 463:   if (offs.has_value()) {
 464:     TORCH_CHECK_VALUE(offs->dim() == 1, "offs has to be 1D");
 465:     TORCH_CHECK_VALUE(offs->dtype() == at::kInt, "Offsets have to be int32");
 466:   }
 467:   // FP8 per-tensor and per-row scaling expect fp32 scales.
 468:   // MXFP8 expects float8_e8m0fnu scales.
 469:   TORCH_CHECK_VALUE(
 470:       (scale_a.scalar_type() == kFloat && scale_b.scalar_type() == kFloat) ||
 471:       (scale_a.scalar_type() == at::kFloat8_e8m0fnu && scale_b.scalar_type() == at::kFloat8_e8m0fnu),
 472:       "For FP8 tensorwise and rowwise, both scales must both be float32 tensors. For MXFP8, scales must both be float8_e8m0fnu tensors.");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 474-476
```cpp
 474:   const int scale_multiplier = (mat_a.dim() == 2 && mat_b.dim() == 2) ? offs->size(0) : 1;
 475:   check_scale(mat_a, scale_a, 0 ,0, scale_multiplier);
 476:   check_scale(mat_b, scale_b, 1, 1, scale_multiplier);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 478-479
```cpp
 478:   const auto out_dtype_ = out_dtype.value_or(kBFloat16);
 479:   TORCH_CHECK_VALUE(out_dtype_ == kBFloat16, "Only bf16 high precision output types are supported for grouped gemm");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 481-481
```cpp
 481:   Tensor out = create_grouped_gemm_output_tensor(mat_a, mat_b, offs, out_dtype_);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 483-483
```cpp
 483: #if defined(USE_MSLK) && defined(USE_CUDA) && !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 484-484
```cpp
 484:   // MXFP8 grouped GEMM dispatching
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 485-491
```cpp
 485:   bool is_mx8mx8bf16 = (
 486:     mat_a.scalar_type() == at::kFloat8_e4m3fn && mat_b.scalar_type() == at::kFloat8_e4m3fn &&
 487:     scale_a.scalar_type() == at::kFloat8_e8m0fnu && scale_b.scalar_type() == at::kFloat8_e8m0fnu
 488:   );
 489: #else
 490:   bool is_mx8mx8bf16 = false;
 491: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 493-505
```cpp
 493:   if (is_mx8mx8bf16) {
 494:     // Note: Passing implied SwizzleType here, correctness of scale previously checked
 495:     //       in `check_scale` call
 496:     return _mx8_mx8_bf16_grouped_mm_mslk(
 497:         mat_a,
 498:         mat_b,
 499:         scale_a,
 500:         SwizzleType::SWIZZLE_32_4_4,
 501:         scale_b,
 502:         SwizzleType::SWIZZLE_32_4_4,
 503:         offs.value(),
 504:         out);
 505:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 507-507
```cpp
 507:   // If we're not MXFP8, then we're row-wise scaling.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 508-517
```cpp
 508:   return _f8_f8_bf16_rowwise_grouped_mm(
 509:       mat_a,
 510:       mat_b,
 511:       scale_a,
 512:       scale_b,
 513:       offs,
 514:       bias,
 515:       use_fast_accum,
 516:       out);
 517: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 519-529
```cpp
 519: namespace {
 520: 
 521: using acceptance_fn = std::function<bool(c10::ScalarType, std::vector<ScalingType>&, ArrayRef<Tensor>&, c10::ScalarType, std::vector<ScalingType>&, ArrayRef<Tensor>&)>;
 522: 
 523: std::array<std::tuple<std::string, acceptance_fn, ScaledGemmImplementation>, 4> scale_grouped_kernel_dispatch = {{
 524:   { "rowwise_rowwise", scaled_blas::check_rowwise_recipe, ScaledGemmImplementation::ROWWISE_ROWWISE},
 525:   { "mxfp8_mxfp8", scaled_blas::check_mxfp8_recipe, ScaledGemmImplementation::MXFP8_MXFP8},
 526:   { "mxfp4_mxfp4", scaled_blas::check_mxfp4_recipe, ScaledGemmImplementation::MXFP4_MXFP4},
 527:   { "nvfp4_nvfp4", scaled_blas::check_nvfp4_recipe, ScaledGemmImplementation::NVFP4_NVFP4}}};
 528: 
 529: } // anonymous namespace
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 531-552
```cpp
 531: Tensor
 532: _scaled_grouped_mm_cuda_v2(
 533:           const Tensor& mat_a, const Tensor& mat_b,
 534:           ArrayRef<Tensor> scale_a,
 535:           IntArrayRef scale_recipe_a,
 536:           IntArrayRef swizzle_a,
 537:           ArrayRef<Tensor> scale_b,
 538:           IntArrayRef scale_recipe_b,
 539:           IntArrayRef swizzle_b,
 540:           const std::optional<Tensor>& offs,
 541:           const std::optional<Tensor>& bias,
 542:           const std::optional<c10::ScalarType> out_dtype,
 543:           IntArrayRef contraction_dim,
 544:           bool use_fast_accum) {
 545:   bool allowed_device = _scaled_mm_allowed_device(/*sm90_only*/true, /*sm100_only*/true);
 546:   TORCH_CHECK_VALUE(allowed_device, "torch._scaled_grouped_mm is only supported on CUDA devices with compute capability = [9.0, 10.0], or ROCm MI300+");
 547: 
 548:   TORCH_CHECK_VALUE(!check_valid_strides_and_return_transposed(mat_a), "Expected mat1 to not be transposed");
 549:   TORCH_CHECK_VALUE(check_valid_strides_and_return_transposed(mat_b), "Expected mat2 to be transposed");
 550:   TORCH_CHECK_VALUE(mat_a.dim() == 2 || mat_a.dim() == 3, "mat_a has to be 2 or 3d");
 551:   TORCH_CHECK_VALUE(mat_b.dim() == 2 || mat_b.dim() == 3, "mat_b has to be 2 or 3d");
 552:   const bool a_is_2d = mat_a.dim() == 2;
```
- EN: This block defines or continues the implementation of `_scaled_grouped_mm_cuda_v2`.
- CN: 该代码块定义或继续实现 `_scaled_grouped_mm_cuda_v2`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 553-553
```cpp
 553:   const bool b_is_2d = mat_b.dim() == 2;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 555-555
```cpp
 555:   // NOTE(slayton): For sub-1B formats want contraction_dim argument?
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 556-569
```cpp
 556:   if (!a_is_2d || !b_is_2d) {
 557:     if (!contraction_dim.empty()) {
 558:       const int dim_a = contraction_dim[0], dim_b = mat_b.size(contraction_dim[1]);
 559:       TORCH_CHECK_VALUE(mat_a.size(dim_a) == mat_b.size(dim_b),
 560:           "Contraction dimensions (", dim_a, ",", dim_b, ") of mat_a and mat_b must match, got: ", mat_a.size(dim_a), " and ",
 561:           mat_b.size(dim_b));
 562:       // Note: only (-1, -2) is currently supported
 563:       TORCH_CHECK_VALUE(dim_a == -1 && dim_b == -2, "Currently contraction dims must be (-1, -2) only");
 564:     } else {
 565:       TORCH_CHECK_VALUE(mat_a.size(-1) == mat_b.size(-2), "contraction dimension of mat_a and mat_b must match");
 566:     }
 567:   }
 568:   TORCH_CHECK_VALUE(
 569:     mat_a.size(-1) % 16 == 0,
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 570-578
```cpp
 570:     "Expected trailing dimension of mat_a to be divisible by 16 ",
 571:     "but got mat1 shape: (",
 572:     mat_a.sizes(),
 573:     ").");
 574:   TORCH_CHECK_VALUE(mat_b.size(-2) % 16 == 0 && mat_b.size(-1) % 16 == 0,
 575:     "Expected mat_b shape to be divisible by 16 ",
 576:     "but got mat_b shape: (",
 577:     mat_b.sizes(),
 578:     ").");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 580-581
```cpp
 580:   TORCH_CHECK_VALUE(!bias.has_value(), "Bias not supported yet");
 581:   TORCH_CHECK_VALUE(offs.has_value() ==  (a_is_2d || b_is_2d), "Have to provide offsets if there is a 2d matrix");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 583-585
```cpp
 583:   // NOTE: mxfp8 x mxfp8 requires (and asserts later) that offsets is present.
 584:   //       for rowwise, no offsets implies 3d-3d and is handled by lower-level
 585:   //       routines
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 586-589
```cpp
 586:   if (offs.has_value()) {
 587:     TORCH_CHECK_VALUE(offs->dim() == 1, "offs has to be 1D");
 588:     TORCH_CHECK_VALUE(offs->dtype() == at::kInt, "Offsets have to be int32");
 589:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 591-592
```cpp
 591:   const auto out_dtype_ = out_dtype.value_or(kBFloat16);
 592:   TORCH_CHECK_VALUE(out_dtype_ == kBFloat16, "Only bf16 high precision output types are supported for grouped gemm");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 594-594
```cpp
 594:   Tensor out = create_grouped_gemm_output_tensor(mat_a, mat_b, offs, out_dtype_);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 596-596
```cpp
 596:   // Conversion of implicitly-defined enums to explicit
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 597-600
```cpp
 597:   auto scale_recipe_a_enum = convert_int_to_enum<ScalingType>(scale_recipe_a);
 598:   auto swizzle_a_enum = convert_int_to_enum<SwizzleType>(swizzle_a);
 599:   auto scale_recipe_b_enum = convert_int_to_enum<ScalingType>(scale_recipe_b);
 600:   auto swizzle_b_enum = convert_int_to_enum<SwizzleType>(swizzle_b);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 602-605
```cpp
 602:   // at this point we can start working out what we want to be doing
 603:   // Try to do as few steps as possible.
 604:   // NOTE: support is deliberately sparse, can explicitly enumerate all combinations allowed.
 605:   // Do this via a list of defined (name, acceptance, concrete_impl) tuples.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 606-619
```cpp
 606:   ScaledGemmImplementation gemm_impl = ScaledGemmImplementation::NONE;
 607:   for (const auto& fn_entry : scale_grouped_kernel_dispatch) {
 608:     const auto [name, accept_fn, scaled_gemm_impl] = fn_entry;
 609:     bool ok = accept_fn(mat_a.scalar_type(),
 610:                         scale_recipe_a_enum,
 611:                         scale_a,
 612:                         mat_b.scalar_type(),
 613:                         scale_recipe_b_enum,
 614:                         scale_b);
 615:     if (ok) {
 616:       gemm_impl = scaled_gemm_impl;
 617:       break;
 618:     }
 619:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 620-621
```cpp
 620:   TORCH_CHECK_VALUE(gemm_impl != ScaledGemmImplementation::NONE,
 621:       "No gemm implementation was found");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 623-644
```cpp
 623:   switch (gemm_impl) {
 624:     case ScaledGemmImplementation::ROWWISE_ROWWISE: {
 625:       const int scale_multiplier = (mat_a.dim() == 2 && mat_b.dim() == 2) ? offs->size(0) : 1;
 626:       _check_scales_fp8_rowwise(mat_a, scale_a[0], 0 /* dim */ , 0 /* arg_idx */, scale_multiplier);
 627:       _check_scales_fp8_rowwise(mat_b, scale_b[0], 1 /* dim */ , 1 /* arg_idx */, scale_multiplier);
 628:       return _f8_f8_bf16_rowwise_grouped_mm(
 629:           mat_a,
 630:           mat_b,
 631:           scale_a[0],
 632:           scale_b[0],
 633:           offs,
 634:           bias,
 635:           use_fast_accum,
 636:           out);
 637:     }
 638:     case ScaledGemmImplementation::MXFP8_MXFP8: {
 639:       // scale shape checks
 640:       _check_scales_blocked(mat_a, scale_a[0], 0 /* dim */, 0 /* arg_idx */);
 641:       _check_scales_blocked(mat_b, scale_b[0], 1 /* dim */, 1 /* arg_idx */);
 642:       // swizze checks
 643:       TORCH_CHECK_VALUE(swizzle_a_enum.size() == 1 && swizzle_b_enum.size() == 1, "Expected single swizzle argument");
 644:       return _mx8_mx8_bf16_grouped_mm_mslk(
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 645-658
```cpp
 645:           mat_a,
 646:           mat_b,
 647:           scale_a[0],
 648:           swizzle_a_enum[0],
 649:           scale_b[0],
 650:           swizzle_b_enum[0],
 651:           offs.value(),
 652:           out);
 653:     }
 654:     case ScaledGemmImplementation::MXFP4_MXFP4: {
 655:       // scale shape checks
 656:       _check_scales_blocked(mat_a, scale_a[0], 0 /* dim */, 0 /* arg_idx */);
 657:       _check_scales_blocked(mat_b, scale_b[0], 1 /* dim */, 1 /* arg_idx */);
 658:       return _f4_f4_bf16_grouped_mm_mslk(
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 659-672
```cpp
 659:           mat_a,
 660:           mat_b,
 661:           scale_a[0], /* block-scale A */
 662:           std::nullopt, /* global-scale A */
 663:           scale_b[0], /* block-scale B */
 664:           std::nullopt, /* global-scale B */
 665:           offs.value(),
 666:           std::nullopt, /* bias */
 667:           out);
 668:     }
 669:     case ScaledGemmImplementation::NVFP4_NVFP4: {
 670:       // scale shape checks
 671:       _check_scales_blocked(mat_a, scale_a[0], 0 /* dim */, 0 /* arg_idx */);
 672:       _check_scales_blocked(mat_b, scale_b[0], 1 /* dim */, 1 /* arg_idx */);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 673-686
```cpp
 673:       return _f4_f4_bf16_grouped_mm_mslk(
 674:           mat_a,
 675:           mat_b,
 676:           scale_a[0], /* block-scale A */
 677:           scale_a[1], /* global-scale A */
 678:           scale_b[0], /* block-scale B */
 679:           scale_b[1], /* global-scale B */
 680:           offs.value(),
 681:           std::nullopt, /* bias */
 682:           out);
 683:     }
 684:     default:
 685:       TORCH_CHECK_NOT_IMPLEMENTED(false,
 686:           "_scaled_grouped_mm_cuda_v2 is in an inconsistent state - should never reach here");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 687-688
```cpp
 687:   }
 688: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 690-711
```cpp
 690: Tensor _grouped_mm_cuda(const Tensor& mat_a, const Tensor& mat_b,
 691: const std::optional<at::Tensor>& offs,
 692: const std::optional<at::Tensor>& bias,
 693: std::optional<c10::ScalarType> out_dtype) {
 694:   _grouped_mm_validate_inputs(mat_a, mat_b, offs, bias, out_dtype);
 695:   bool a_b_and_out_are_bf16 = (
 696:     mat_a.dtype() == at::kBFloat16 &&
 697:     mat_b.dtype() == at::kBFloat16 &&
 698:     out_dtype.value_or(at::kBFloat16) == at::kBFloat16
 699:   );
 700: #ifndef USE_ROCM
 701:   bool use_fast_path = _scaled_mm_allowed_device(/*sm90_only*/true, /*sm100_only*/true) && a_b_and_out_are_bf16;
 702:   const auto out_dtype_ = _resolve_grouped_mm_out_dtype(mat_a, mat_b, out_dtype);
 703:   Tensor out = create_grouped_gemm_output_tensor(mat_a, mat_b, offs, out_dtype_);
 704:   if (use_fast_path) {
 705:     // fast path, no d2h sync needed
 706:     at::cuda::detail::bf16bf16_grouped_mm(mat_a, mat_b, offs, bias, out);
 707:   } else {
 708:     _grouped_mm_fallback(mat_a, mat_b, offs, bias, out_dtype, out);
 709:   }
 710: #else
 711:   // On ROCm fast path routes to group_gemm_ck and slow path to _grouped_mm_fallback.
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_grouped_mm_cuda`.
- CN: 该代码块定义或继续实现 `_grouped_mm_cuda`。

### Lines 712-713
```cpp
 712:   // Keep use_fast_path as false till ck kernel perf is optimal.
 713:   // To enable CK path, use env variable ROCM_ALLOW_GROUP_GEMM_CK=1.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 714-727
```cpp
 714:   bool use_fast_path = false;
 715:   // ifdef USE_ROCM_CK_GEMM is required since ROCm systems w/o CK should not call ck path.
 716: #if defined(USE_ROCM_CK_GEMM)
 717:   if (at::globalContext().rocmAllowGroupGemmCk() && at::detail::getCUDAHooks().isGPUArch({"gfx942", "gfx950", "gfx90a"})) {
 718:     use_fast_path = true;
 719:   }
 720: #endif //USE_ROCM_CK_GEMM
 721:   const auto out_dtype_ = _resolve_grouped_mm_out_dtype(mat_a, mat_b, out_dtype);
 722:   Tensor out = create_grouped_gemm_output_tensor(mat_a, mat_b, offs, out_dtype_);
 723:   if (use_fast_path) {
 724:     at::hip::detail::group_gemm_ck(mat_a, mat_b, offs, bias, out);
 725:   } else {
 726:     _grouped_mm_fallback(mat_a, mat_b, offs, bias, out_dtype, out);
 727:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 728-728
```cpp
 728: #endif //ifndef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 729-730
```cpp
 729:   return out;
 730: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 732-732
```cpp
 732: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<cstdint>`
  - `<c10/util/typeid.h>`
  - `<c10/util/Exception.h>`
  - `<c10/util/SmallVector.h>`
  - `<c10/core/Scalar.h>`
  - `<c10/core/ScalarType.h>`
  - `<ATen/Context.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/core/NamedTensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/ExpandUtils.h>`
  - `<ATen/OpMathType.h>`
- Runtime symbols / 运行时符号:
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::detail::f8f8bf16_grouped_mm`
  - `at::cuda::detail::bf16bf16_grouped_mm`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
