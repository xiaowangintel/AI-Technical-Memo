# ScaledBlas.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ScaledBlas.cpp`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Provides host-side CUDA entry points, orchestration, or dispatch glue around `_scaled_mm_allowed_device`, `is_tensorwise_scaling`, `is_rowwise_scaling`, `check_size_stride`.
- 用途（中文）: 提供围绕 `_scaled_mm_allowed_device`, `is_tensorwise_scaling`, `is_rowwise_scaling`, `check_size_stride` 的主机端 CUDA 入口、调度编排或分发胶水代码。

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
   8: #include <ATen/core/Tensor.h>
   9: #include <ATen/core/NamedTensor.h>
  10: #include <ATen/Dispatch.h>
  11: #include <ATen/ExpandUtils.h>
  12: #include <ATen/OpMathType.h>
  13: #include <ATen/TensorUtils.h>
  14: #include <ATen/cuda/CUDABlas.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cstdint>`, `<c10/util/typeid.h>`, `<c10/util/Exception.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cstdint>`, `<c10/util/typeid.h>`, `<c10/util/Exception.h>`。

### Lines 15-28
```cpp
  15: #include <ATen/native/ScaledBlasUtils.h>
  16: #include <ATen/cuda/tunable/Tunable.h>
  17: #include <ATen/cuda/tunable/TunableGemm.h>
  18: #include <ATen/native/Resize.h>
  19: #include <c10/util/MaybeOwned.h>
  20: #include <ATen/native/GroupedMMUtils.h>
  21: #include <ATen/native/cuda/RowwiseScaledMM.h>
  22: #include <ATen/native/cuda/ScaledGroupMM.h>
  23: #include <ATen/native/cuda/GroupMM.h>
  24: #include <ATen/native/cuda/cuBlasCommonArgs.h>
  25: #include <ATen/ceil_div.h>
  26: 
  27: #ifdef USE_MSLK
  28: #include <mslk/gemm/gemm_torch.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/ScaledBlasUtils.h>`, `<ATen/cuda/tunable/Tunable.h>`, `<ATen/cuda/tunable/TunableGemm.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/ScaledBlasUtils.h>`, `<ATen/cuda/tunable/Tunable.h>`, `<ATen/cuda/tunable/TunableGemm.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 29-42
```cpp
  29: #endif
  30: 
  31: #ifndef AT_PER_OPERATOR_HEADERS
  32: #include <ATen/Functions.h>
  33: #include <ATen/NativeFunctions.h>
  34: #else
  35: #include <ATen/ops/_addmm_activation_native.h>
  36: #include <ATen/ops/_efficientzerotensor.h>
  37: #include <ATen/ops/_scaled_mm_native.h>
  38: #include <ATen/ops/_scaled_mm_v2_native.h>
  39: #include <ATen/ops/_unsafe_view_native.h>
  40: #include <ATen/ops/abs.h>
  41: #include <ATen/ops/addmm_native.h>
  42: #include <ATen/ops/addmv_native.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_addmm_activation_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Functions.h>`, `<ATen/NativeFunctions.h>`, `<ATen/ops/_addmm_activation_native.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 43-56
```cpp
  43: #include <ATen/ops/baddbmm_native.h>
  44: #include <ATen/ops/bmm_native.h>
  45: #include <ATen/ops/copy_native.h>
  46: #include <ATen/ops/dot_native.h>
  47: #include <ATen/ops/empty.h>
  48: #include <ATen/ops/empty_strided.h>
  49: #include <ATen/ops/gelu.h>
  50: #include <ATen/ops/max.h>
  51: #include <ATen/ops/mm_native.h>
  52: #include <ATen/ops/mul.h>
  53: #include <ATen/ops/relu.h>
  54: #include <ATen/ops/ones.h>
  55: #include <ATen/ops/scalar_tensor_native.h>
  56: #include <ATen/ops/vdot_native.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/ops/baddbmm_native.h>`, `<ATen/ops/bmm_native.h>`, `<ATen/ops/copy_native.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/ops/baddbmm_native.h>`, `<ATen/ops/bmm_native.h>`, `<ATen/ops/copy_native.h>`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 57-57
```cpp
  57: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 59-59
```cpp
  59: // forward declare
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 60-60
```cpp
  60: class cublasCommonArgs;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

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
  71: namespace{
  72: 
  73: bool _scaled_mm_allowed_device(bool sm90_only=false, bool sm100_only=false) {
  74: #ifdef USE_ROCM
  75:     static const std::vector<std::string> archs = {
  76:         "gfx942",
  77: #if ROCM_VERSION >= 60300
  78:         "gfx1200", "gfx1201",
  79: #endif
  80: #if ROCM_VERSION >= 60500
  81:         "gfx950"
  82: #endif
  83:     };
  84:     return at::detail::getCUDAHooks().isGPUArch(archs);
  85: #else
  86:     auto dprops = at::cuda::getCurrentDeviceProperties();
  87: 
  88:     if (sm90_only || sm100_only) {
  89:       return (sm90_only && dprops->major == 9) || (sm100_only && dprops->major == 10);
  90:     } else {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `_scaled_mm_allowed_device`.
- CN: 该代码块定义或继续实现 `_scaled_mm_allowed_device`。

### Lines 91-94
```cpp
  91:       return dprops->major >= 9 || (dprops->major == 8 && dprops->minor == 9);
  92:     }
  93: #endif
  94: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 96-96
```cpp
  96: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 97-100
```cpp
  97: bool _scaled_mm_is_fnuz() {
  98:     return at::detail::getCUDAHooks().isGPUArch({"gfx942"});
  99: }
 100: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_mm_is_fnuz`.
- CN: 该代码块定义或继续实现 `_scaled_mm_is_fnuz`。

### Lines 102-115
```cpp
 102: /*
 103:  * Scaling Type Determination:
 104:  * ---------------------------
 105:  * Conditions and corresponding Scaling Types:
 106:  *
 107:  * - If scale tensor is `Float8_e8m0fnu` or `Float8_e4m3fn`:
 108:  *   - Returns BlockWise (with additional size checks).
 109:  *
 110:  * - Else if scale.numel() == 1:
 111:  *   - Returns TensorWise.
 112:  *
 113:  * - Else if scale.dim() == 2 && scale.size(0) == outer_dim && scale.size(1) == 1:
 114:  *   - Returns RowWise.
 115:  *
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 116-124
```cpp
 116:  * - Else if scale.dim() == 2 && scale.size(0) == outer_dim && scale.size(1) == inner_dim / 128:
 117:  *   - Returns BlockWise 1x128.
 118:  *
 119:  * - Else if scale.dim() == 2 && scale.size(0) == outer_dim / 128 && scale.size(1) == inner_dim / 128:
 120:  *   - Returns BlockWise 128x128.
 121:  *
 122:  * - Otherwise:
 123:  *   - Returns Error.
 124:  */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 126-126
```cpp
 126: using at::blas::ScalingType;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 128-130
```cpp
 128: bool is_tensorwise_scaling(const at::Tensor& t, const at::Tensor& scale) {
 129:   return isFloat8Type(t.scalar_type()) && scale.scalar_type() == kFloat && scale.numel() == 1;
 130: }
```
- EN: This block defines or continues the implementation of `is_tensorwise_scaling`.
- CN: 该代码块定义或继续实现 `is_tensorwise_scaling`。

### Lines 132-136
```cpp
 132: bool is_rowwise_scaling(const at::Tensor& t, const at::Tensor& scale) {
 133:   return (isFloat8Type(t.scalar_type()) && scale.scalar_type() == kFloat && scale.dim() == 2
 134:       && scale.size(0) == t.size(0) && scale.size(1) == 1
 135:       && scale.is_contiguous());
 136: }
```
- EN: This block defines or continues the implementation of `is_rowwise_scaling`.
- CN: 该代码块定义或继续实现 `is_rowwise_scaling`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 138-147
```cpp
 138: bool check_size_stride(const at::Tensor& scale, int dim, int size, int stride) {
 139:   // For Blockwise1x128 and Blockwise128x128,
 140:   // when the scale tensor has a dimension of size 1, the stride is effectively
 141:   // "meaningless", i.e. PyTorch decides to use a stride of 1. Thus, the regular
 142:   // stride check fails. Here, we relax the stride check when the effective
 143:   // stride is 1.
 144: 
 145:   return (
 146:       scale.size(dim) == size && (size <= 1 || scale.stride(dim) == stride));
 147: }
```
- EN: This block defines or continues the implementation of `check_size_stride`.
- CN: 该代码块定义或继续实现 `check_size_stride`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 149-149
```cpp
 149: // 1x16 blocks for packed nvfp4 data and fp8_e4m3fn scales
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 150-157
```cpp
 150: bool is_blockwise_1x16_scaling(const at::Tensor& t, const at::Tensor& scale) {
 151:   // Multiply t.size(1) by 2 to adjust for fp4x2 packing
 152:   // TODO: We might want to enforce some structure on the shapes of the scale
 153:   // tensors
 154:   return (t.scalar_type() == ScalarType::Float4_e2m1fn_x2 && scale.scalar_type() == at::kFloat8_e4m3fn
 155:       && scale.numel() == round_up<int64_t>(t.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(t.size(1) * 2, 16), 4)
 156:       && scale.is_contiguous());
 157: }
```
- EN: This block defines or continues the implementation of `is_blockwise_1x16_scaling`.
- CN: 该代码块定义或继续实现 `is_blockwise_1x16_scaling`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 159-159
```cpp
 159: // 1x32 blocks for microscaled fp8 data and fp8_e8m0fnu scales
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 160-171
```cpp
 160: bool is_blockwise_1x32_scaling(const at::Tensor& t, const at::Tensor& scale) {
 161:   // TODO: We might want to enforce some structure on the shapes of the scale
 162:   // tensors
 163:   bool is_fp8_path = (isFloat8Type(t.scalar_type()) && scale.scalar_type() == at::kFloat8_e8m0fnu
 164:       && scale.numel() == round_up<int64_t>(t.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(t.size(1), 32), 4));
 165:   bool is_packed_fp4_path = false;
 166: #ifdef USE_ROCM
 167:   is_packed_fp4_path = (t.scalar_type() == ScalarType::Float4_e2m1fn_x2 && scale.scalar_type() == at::kFloat8_e8m0fnu
 168:       && scale.numel() == round_up<int64_t>(t.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(t.size(1) * 2, 32), 4));
 169: #endif
 170:   return (is_fp8_path || is_packed_fp4_path) && scale.is_contiguous();
 171: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `is_blockwise_1x32_scaling`.
- CN: 该代码块定义或继续实现 `is_blockwise_1x32_scaling`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 173-179
```cpp
 173: bool is_blockwise_1x128_scaling(const at::Tensor& t, const at::Tensor& scale) {
 174:   return (
 175:       isFloat8Type(t.scalar_type()) && scale.scalar_type() == kFloat &&
 176:       scale.dim() == 2 && check_size_stride(scale, 0, t.size(0), 1) &&
 177:       check_size_stride(
 178:           scale, 1, ceil_div<int64_t>(t.size(1), 128), t.size(0)));
 179: }
```
- EN: This block defines or continues the implementation of `is_blockwise_1x128_scaling`.
- CN: 该代码块定义或继续实现 `is_blockwise_1x128_scaling`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 181-192
```cpp
 181: bool is_blockwise_128x128_scaling(const at::Tensor& t, const at::Tensor& scale) {
 182:   return (
 183:       isFloat8Type(t.scalar_type()) && scale.scalar_type() == kFloat &&
 184:       scale.dim() == 2 &&
 185:       check_size_stride(
 186:           scale,
 187:           0,
 188:           ceil_div<int64_t>(t.size(0), 128),
 189:           ceil_div<int64_t>(t.size(1), 128)) &&
 190:       check_size_stride(
 191:           scale, 1, ceil_div<int64_t>(t.size(1), 128), 1));
 192: }
```
- EN: This block defines or continues the implementation of `is_blockwise_128x128_scaling`.
- CN: 该代码块定义或继续实现 `is_blockwise_128x128_scaling`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 194-211
```cpp
 194: bool is_desired_scaling(const at::Tensor& t, const at::Tensor& scale, ScalingType desired_scaling) {
 195:   switch (desired_scaling) {
 196:     case ScalingType::TensorWise:
 197:       return is_tensorwise_scaling(t, scale);
 198:     case ScalingType::RowWise:
 199:       return is_rowwise_scaling(t, scale);
 200:     case ScalingType::BlockWise1x16:
 201:       return is_blockwise_1x16_scaling(t, scale);
 202:     case ScalingType::BlockWise1x32:
 203:       return is_blockwise_1x32_scaling(t, scale);
 204:     case ScalingType::BlockWise1x128:
 205:       return is_blockwise_1x128_scaling(t, scale);
 206:     case ScalingType::BlockWise128x128:
 207:       return is_blockwise_128x128_scaling(t, scale);
 208:     default:
 209:       TORCH_CHECK(false, "Unknown scaling type");
 210:   }
 211: }
```
- EN: This block defines or continues the implementation of `is_desired_scaling`.
- CN: 该代码块定义或继续实现 `is_desired_scaling`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 213-234
```cpp
 213: std::pair<ScalingType, ScalingType> get_joint_scaling(
 214:     std::initializer_list<std::pair<ScalingType, ScalingType>> options,
 215:     const at::Tensor& a, const at::Tensor& b,
 216:     const at::Tensor& scale_a, const at::Tensor& scale_b) {
 217:   for (auto [lhs, rhs] : options) {
 218:     // For blockwise 1x16 and 1x32 scaling, the scale tensors are swizzled/blocked
 219:     // and should not be transposed as their structure is based on the original tensor dimensions
 220:     bool use_swizzled_scale = (rhs == ScalingType::BlockWise1x16 || rhs == ScalingType::BlockWise1x32);
 221:     const at::Tensor& scale_b_check = use_swizzled_scale ? scale_b : scale_b.t();
 222: 
 223:     if (is_desired_scaling(a, scale_a, lhs) && is_desired_scaling(b.t(), scale_b_check, rhs)) {
 224:       return {lhs, rhs};
 225:     }
 226:   }
 227:   TORCH_CHECK(
 228:     false,
 229:     "Invalid scaling configuration.\n"
 230:     "- For TensorWise scaling, a and b should be float8, scales should be float and singletons.\n"
 231:     "- For RowWise scaling, a and b should be float8, scales should be float, scale_a should be (", a.size(0), ", 1) and scale_b should be (1, ", b.size(1), "), and both should be contiguous.\n"
 232:     "- For BlockWise 1x128 scaling, a and b should be float8, scales should be float, scale_a should be (", a.size(0), ", ", ceil_div<int64_t>(a.size(1), 128), ") and scale_b should be (", ceil_div<int64_t>(b.size(0), 128), ", ", b.size(1), "), and both should be outer-dim-major.\n"
 233:     "- For BlockWise 128x128 scaling, a and b should be float8, scales should be float, scale_a should be (", ceil_div<int64_t>(a.size(0), 128), ", ", ceil_div<int64_t>(a.size(1), 128), ") and scale_b should be (", ceil_div<int64_t>(b.size(0), 128), ", ", ceil_div<int64_t>(b.size(1), 128), "), and both should be near-inner-dim-major (with 16-byte aligned strides).\n"
 234:     "- For Blockwise 1x32 scaling, a and b should be float8, scales should be float8_e8m0fnu, scale_a should have ", round_up<int64_t>(a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(a.size(1), 32), 4), " elements and scale_b should have ", round_up<int64_t>(b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(b.size(0), 32), 4), " elements, and both should be contiguous.\n"
```
- EN: This block defines or continues the implementation of `get_joint_scaling`.
- CN: 该代码块定义或继续实现 `get_joint_scaling`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 235-239
```cpp
 235:     "- For Blockwise 1x16 scaling, a and b should be float4 (packed 2x), scales should be float8_e4m3fn, scale_a should have ", round_up<int64_t>(a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(a.size(1) * 2, 16), 4), " elements and scale_b should have ", round_up<int64_t>(b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(b.size(0) * 2, 16), 4), " elements, and both should be contiguous.\n"
 236:     "Got a.dtype()=", a.scalar_type(), ", scale_a.dtype()=", scale_a.scalar_type(), ", scale_a.size()=", scale_a.sizes(), ", scale_a.stride()=", scale_a.strides(), ", ",
 237:     "b.dtype()=", b.scalar_type(), ", scale_b.dtype()=", scale_b.scalar_type(), ", scale_b.size()=", scale_b.sizes(), " and scale_b.stride()=", scale_b.strides()
 238:   );
 239: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 241-262
```cpp
 241: Tensor&
 242: _tunable_scaled_gemm_rocm(
 243:           cublasCommonArgs& args,
 244:           const Tensor& mat1, const Tensor& mat2,
 245:           const Tensor& scale_a, const Tensor& scale_b,
 246:           const ScalingType scaling_choice_a, const ScalingType scaling_choice_b,
 247:           const std::optional<Tensor>& bias,
 248:           const bool use_fast_accum,
 249:           const at::ScalarType out_dtype,
 250:           Tensor& out) {
 251: #ifdef USE_ROCM
 252: #define TUNABLE_DISPATCH(BLASOP_A, BLASOP_B)                            \
 253:       if (mat1.scalar_type() == ScalarType::Float8_e4m3fnuz) {        \
 254:         if (mat2.scalar_type() == ScalarType::Float8_e4m3fnuz) {      \
 255:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 256:               at::Float8_e4m3fnuz, at::Float8_e4m3fnuz, scalar_t,     \
 257:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 258:           scaledgemm(&params);                                        \
 259:         }                                                             \
 260:         else if (mat2.scalar_type() == ScalarType::Float8_e5m2fnuz) { \
 261:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 262:               at::Float8_e4m3fnuz, at::Float8_e5m2fnuz, scalar_t,     \
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_tunable_scaled_gemm_rocm`.
- CN: 该代码块定义或继续实现 `_tunable_scaled_gemm_rocm`。

### Lines 263-276
```cpp
 263:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 264:           scaledgemm(&params);                                        \
 265:         }                                                             \
 266:       }                                                               \
 267:       else if (mat1.scalar_type() == ScalarType::Float8_e5m2fnuz) {   \
 268:         if (mat2.scalar_type() == ScalarType::Float8_e4m3fnuz) {      \
 269:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 270:               at::Float8_e5m2fnuz, at::Float8_e4m3fnuz, scalar_t,     \
 271:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 272:           scaledgemm(&params);                                        \
 273:         }                                                             \
 274:         else if (mat2.scalar_type() == ScalarType::Float8_e5m2fnuz) { \
 275:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 276:               at::Float8_e5m2fnuz, at::Float8_e5m2fnuz, scalar_t,     \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 277-290
```cpp
 277:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 278:           scaledgemm(&params);                                        \
 279:         }                                                             \
 280:       }                                                               \
 281:       else if (mat1.scalar_type() == ScalarType::Float8_e4m3fn) {     \
 282:         if (mat2.scalar_type() == ScalarType::Float8_e4m3fn) {        \
 283:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 284:               at::Float8_e4m3fn, at::Float8_e4m3fn, scalar_t,         \
 285:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 286:           scaledgemm(&params);                                        \
 287:         }                                                             \
 288:         else if (mat2.scalar_type() == ScalarType::Float8_e5m2) {     \
 289:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 290:               at::Float8_e4m3fn, at::Float8_e5m2, scalar_t,           \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 291-304
```cpp
 291:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 292:           scaledgemm(&params);                                        \
 293:         }                                                             \
 294:       }                                                               \
 295:       else if (mat1.scalar_type() == ScalarType::Float8_e5m2) {       \
 296:         if (mat2.scalar_type() == ScalarType::Float8_e4m3fn) {        \
 297:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 298:               at::Float8_e5m2, at::Float8_e4m3fn, scalar_t,           \
 299:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 300:           scaledgemm(&params);                                        \
 301:         }                                                             \
 302:         else if (mat2.scalar_type() == ScalarType::Float8_e5m2) {     \
 303:           static at::cuda::tunable::ScaledGemmTunableOp<              \
 304:               at::Float8_e5m2, at::Float8_e5m2, scalar_t,             \
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 305-318
```cpp
 305:               BLASOP_A, BLASOP_B> scaledgemm{};                       \
 306:           scaledgemm(&params);                                        \
 307:         }                                                             \
 308:       }
 309:   AT_DISPATCH_V2(out_dtype, "_tunable_scaled_gemm", AT_WRAP([&] {
 310:     bool transa_ = ((args.transa != 'n') && (args.transa != 'N'));
 311:     bool transb_ = ((args.transb != 'n') && (args.transb != 'N'));
 312:     at::cuda::tunable::ScaledGemmParams<scalar_t> params;
 313:     params.transa = args.transa;
 314:     params.transb = args.transb;
 315:     params.m = args.m;
 316:     params.n = args.n;
 317:     params.k = args.k;
 318:     params.a = args.mata->data_ptr();
```
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 319-332
```cpp
 319:     params.a_scale_ptr = args.scale_mata_ptr;
 320:     params.a_scale_dtype = args.scale_mata_dtype.value();
 321:     params.lda = args.lda;
 322:     params.a_dtype = args.mata->scalar_type();
 323:     params.a_scale_dtype = args.scale_mata_dtype.value();
 324:     params.a_scaling_type = args.scaling_mata_type.value();
 325:     params.b = args.matb->data_ptr();
 326:     params.b_scale_ptr = args.scale_matb_ptr;
 327:     params.b_scale_dtype = args.scale_matb_dtype.value();
 328:     params.ldb = args.ldb;
 329:     params.b_dtype = args.matb->scalar_type();
 330:     params.b_scale_dtype = args.scale_matb_dtype.value();
 331:     params.b_scaling_type = args.scaling_matb_type.value();
 332:     params.bias_ptr = bias ? bias->data_ptr(): nullptr;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 333-347
```cpp
 333:     params.bias_dtype = bias ? bias->scalar_type() : isFloat8Type(out_dtype) ? at::ScalarType::Half : out_dtype;
 334:     params.c = args.result->data_ptr();
 335:     params.c_scale_ptr = args.scale_result_ptr;
 336:     params.ldc = args.result_ld;
 337:     params.c_dtype = out_dtype;
 338:     params.use_fast_accum = use_fast_accum;
 339:     if (transa_ && transb_) {
 340:       TUNABLE_DISPATCH(at::cuda::tunable::BlasOp::T, at::cuda::tunable::BlasOp::T)
 341:     }
 342:     else if (transa_ && !transb_) {
 343:       TUNABLE_DISPATCH(at::cuda::tunable::BlasOp::T, at::cuda::tunable::BlasOp::N)
 344:     }
 345:     else if (!transa_ && transb_) {
 346:       TUNABLE_DISPATCH(at::cuda::tunable::BlasOp::N, at::cuda::tunable::BlasOp::T)
 347:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 348-361
```cpp
 348:     else if (!transa_ && !transb_) {
 349:       TUNABLE_DISPATCH(at::cuda::tunable::BlasOp::N, at::cuda::tunable::BlasOp::N)
 350:     }
 351:     else {
 352:       TORCH_CHECK(false, "unreachable");
 353:     }
 354:   }),
 355:   kHalf, kBFloat16, AT_EXPAND(AT_FLOAT8_TYPES), AT_EXPAND(AT_FLOATING_TYPES));
 356: #undef TUNABLE_DISPATCH
 357:   return out;
 358: #else
 359:   TORCH_CHECK_NOT_IMPLEMENTED(false, "_scaled_gemm_rocm only callable on ROCM devices");
 360: #endif
 361: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 363-384
```cpp
 363: Tensor&
 364: _scaled_gemm(
 365:           const Tensor& mat1, const Tensor& mat2,
 366:           const Tensor& scale_a, const Tensor& scale_b,
 367:           const ScalingType scaling_choice_a, const ScalingType scaling_choice_b,
 368:           const std::optional<Tensor>& bias,
 369:           const bool use_fast_accum,
 370:           Tensor& out,
 371:           const std::optional<Tensor>& alpha = std::nullopt) {
 372:   cublasCommonArgs args(mat1, mat2, out, scale_a, scale_b, std::nullopt, scaling_choice_a, scaling_choice_b);
 373:   const auto out_dtype_ = args.result->scalar_type();
 374:   // H100 only supports row-major x column-major, but all permutaitons are supported on Blackwells
 375:   if (_scaled_mm_allowed_device(true, false)) {
 376:     TORCH_CHECK(args.transa == 't' && args.transb == 'n', "Only multiplication of row-major and column-major matrices is supported by cuBLASLt");
 377:   }
 378: // ROCM enables the TunableOp path only
 379: // but can fallback to at::cuda::blas::scaled_gemm
 380: #ifdef USE_ROCM
 381:   auto tuning_ctx = at::cuda::tunable::getTuningContext();
 382:   bool tunable_op_enabled = tuning_ctx->IsTunableOpEnabled();
 383: #else
 384:   bool tunable_op_enabled = false;
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_gemm`.
- CN: 该代码块定义或继续实现 `_scaled_gemm`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 385-385
```cpp
 385: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 386-407
```cpp
 386:   if (tunable_op_enabled) {
 387:       // Only available on ROCM
 388:       return _tunable_scaled_gemm_rocm(
 389:           args,
 390:           mat1, mat2,
 391:           scale_a, scale_b,
 392:           scaling_choice_a, scaling_choice_b,
 393:           bias,
 394:           use_fast_accum,
 395:           out_dtype_,
 396:           out);
 397:   }
 398:   else
 399:   {
 400:       at::cuda::blas::scaled_gemm(
 401:           args.transa,
 402:           args.transb,
 403:           args.m,
 404:           args.n,
 405:           args.k,
 406:           args.mata->data_ptr(),
 407:           args.scale_mata_ptr,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 408-421
```cpp
 408:           args.lda,
 409:           args.mata->scalar_type(),
 410:           args.scale_mata_dtype.value(),
 411:           args.scaling_mata_type.value(),
 412:           args.matb->data_ptr(),
 413:           args.scale_matb_ptr,
 414:           args.ldb,
 415:           args.matb->scalar_type(),
 416:           args.scale_matb_dtype.value(),
 417:           args.scaling_matb_type.value(),
 418:           bias ? bias->data_ptr(): nullptr,
 419:           bias ? bias->scalar_type() : isFloat8Type(out_dtype_) ? at::ScalarType::Half : out_dtype_,
 420:           args.result->data_ptr(),
 421:           args.scale_result_ptr,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 422-428
```cpp
 422:           args.result_ld,
 423:           out_dtype_,
 424:           use_fast_accum,
 425:           alpha);
 426:       return out;
 427:   }
 428: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 431-432
```cpp
 431: // NOTE(slayton58): This is defined as part of the _v2 code (way) below - declare the signature here
 432: //                  to help cleanup v1 call structure.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 433-440
```cpp
 433: Tensor&
 434: _scaled_rowwise_rowwise(
 435:           const Tensor& /*mat_a*/, const Tensor& /*mat_b*/,
 436:           const Tensor& /*scale_a*/, const Tensor& /*scale_b*/,
 437:           const std::optional<Tensor>& /*bias*/,
 438:           const c10::ScalarType /*out_dtype*/,
 439:           bool /*use_fast_accum*/,
 440:           Tensor& /*out*/);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 442-442
```cpp
 442: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 445-462
```cpp
 445: // Computes matrix multiply + bias while applying scaling to input and output matrices
 446: // Scales are only applicable when matrices are of Float8 type and assumed to be equal to 1.0 by default.
 447: // If output matrix type is 16 or 32-bit type, scale_result is not applied.
 448: // Known limitations:
 449: //  - Only works if mat1 is row-major and mat2 is column-major
 450: //  - Only works if matrices sizes are divisible by 32
 451: //  - If 1-dimensional tensors are used then scale_a should be size = mat1.size(0)
 452: //    and scale_b should have size = to mat2.size(1)
 453: //  Arguments:
 454: //    - `mat1`: the first operand of the matrix multiply, can be type `torch.float8_e4m3fn` or `torch.float8_e5m2`
 455: //    - `mat2`: the second operand of the matrix multiply, can be type `torch.float8_e4m3fn` or `torch.float8_e5m2`
 456: //    - `bias`: the bias, can be type `torch.float16` or `torch.bfloat16`
 457: //    - `out_dtype`: the output dtype, can either be a float8 or a higher precision floating point type
 458: //    - `scale_a`: a tensor with the inverse scale of `mat1`, whose shape/strides/dtype depend on the scaling scheme
 459: //    - `scale_b`: a tensor with the inverse scale of `mat2`, whose shape/strides/dtype depend on the scaling scheme
 460: //    - `scale_result`: a scalar tensor with the scale of the output, only utilized if the output is a float8 type
 461: //    - `use_fast_accum`: if true, enables fast float8 accumulation. Backends may ignore this option if not applicable.
 462: //    - `out`: a reference to the output tensor
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 464-485
```cpp
 464: Tensor&
 465: _scaled_mm_out_cuda(const Tensor& mat1, const Tensor& mat2,
 466:           const Tensor& scale_a,
 467:           const Tensor& scale_b,
 468:           const std::optional<at::Tensor>& bias,
 469:           const std::optional<at::Tensor>& scale_result,
 470:           std::optional<c10::ScalarType> out_dtype,
 471:           bool use_fast_accum,
 472:           Tensor& out) {
 473:   // Check sizes
 474:   bool allowed_device = _scaled_mm_allowed_device();
 475:   TORCH_CHECK(allowed_device, "torch._scaled_mm is only supported on CUDA devices with compute capability >= 9.0 or 8.9, or ROCm MI300+");
 476:   TORCH_CHECK(mat1.dim() == 2, "mat1 must be a matrix");
 477:   TORCH_CHECK(mat2.dim() == 2, "mat2 must be a matrix");
 478:   TORCH_CHECK(
 479:       mat1.sizes()[1] == mat2.sizes()[0], "mat1 and mat2 shapes cannot be multiplied (",
 480:       mat1.sizes()[0], "x", mat1.sizes()[1], " and ", mat2.sizes()[0], "x", mat2.sizes()[1], ")");
 481: 
 482:   // Check what type of scaling we are doing based on inputs. This list is sorted
 483:   // by decreasing priority. We prefer "simpler" schemes as they are supported
 484:   // more broadly (more GPU archs, more CUDA versions) and because they are more
 485:   // efficient. This tends to matter only for small matmuls (e.g., 1x1x128).
```
- EN: This block defines or continues the implementation of `_scaled_mm_out_cuda`.
- CN: 该代码块定义或继续实现 `_scaled_mm_out_cuda`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 487-488
```cpp
 487:   // List of supported BlockWise pairs for FP8:
 488:   // https://docs.nvidia.com/cuda/cublas/#element-1d-and-128x128-2d-block-scaling-for-fp8-data-types
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 490-500
```cpp
 490:   auto [scaling_choice_a, scaling_choice_b] = get_joint_scaling(
 491:     {
 492:       std::make_pair(ScalingType::TensorWise, ScalingType::TensorWise),
 493:       std::make_pair(ScalingType::RowWise, ScalingType::RowWise),
 494:       std::make_pair(ScalingType::BlockWise128x128, ScalingType::BlockWise1x128),
 495:       std::make_pair(ScalingType::BlockWise1x128, ScalingType::BlockWise128x128),
 496:       std::make_pair(ScalingType::BlockWise1x128, ScalingType::BlockWise1x128),
 497:       std::make_pair(ScalingType::BlockWise1x32, ScalingType::BlockWise1x32),
 498:       std::make_pair(ScalingType::BlockWise1x16, ScalingType::BlockWise1x16)
 499:     },
 500:     mat1, mat2, scale_a, scale_b);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 502-515
```cpp
 502:   TORCH_CHECK(!scale_result || (scale_result->numel() == 1 && scale_result->scalar_type() == kFloat),
 503:        "scale_result must be a float scalar");
 504:   TORCH_CHECK(!bias || bias->numel() == mat2.sizes()[1], "Bias must be size ", mat2.sizes()[1],
 505:        " but got ", bias->numel());
 506:   TORCH_CHECK(
 507:       mat1.sizes()[1] % 16 == 0,
 508:       "Expected trailing dimension of mat1 to be divisible by 16 ",
 509:       "but got mat1 shape: (",
 510:       mat1.sizes()[0],
 511:       "x",
 512:       mat1.sizes()[1],
 513:       ").");
 514:   TORCH_CHECK(mat2.sizes()[0] % 16 == 0 && mat2.sizes()[1] % 16 == 0, "mat2 shape (", mat2.sizes()[0], "x",
 515:        mat2.sizes()[1], ") must be divisible by 16");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 516-516
```cpp
 516:   // Check types
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 517-531
```cpp
 517:   TORCH_CHECK(!out_dtype || *out_dtype == out.scalar_type(), "out_dtype must match output matrix type");
 518:   TORCH_CHECK(isFloat8Type(mat1.scalar_type()) || mat1.scalar_type() == ScalarType::Float4_e2m1fn_x2, "Expected mat1 to be Float8 or Float4_x2 matrix got ", mat1.scalar_type());
 519:   TORCH_CHECK(isFloat8Type(mat2.scalar_type()) || mat2.scalar_type() == ScalarType::Float4_e2m1fn_x2, "Expected mat2 to be Float8 or Float4_x2 matrix got ", mat2.scalar_type());
 520: #ifndef USE_ROCM
 521:   // Type restrictions imposed by CuBLASLt as of CUDA-12.1
 522:   TORCH_CHECK_VALUE(mat1.scalar_type() != ScalarType::Float8_e5m2 || mat2.scalar_type() != ScalarType::Float8_e5m2,
 523:         "Multiplication of two Float8_e5m2 matrices is not supported");
 524: #endif
 525:   if (use_fast_accum) {
 526:     TORCH_CHECK(mat1.scalar_type() != ScalarType::Float4_e2m1fn_x2 && mat2.scalar_type() != ScalarType::Float4_e2m1fn_x2, "`use_fast_accum` is not supported when `mat1` or `mat2` tensors have the `Float4_e2m1fn_x2` dtype.");
 527:   }
 528: #ifdef USE_ROCM
 529:   if (mat1.scalar_type() == ScalarType::Float4_e2m1fn_x2 || mat2.scalar_type() == ScalarType::Float4_e2m1fn_x2) {
 530:     TORCH_CHECK(ROCM_VERSION >= 70000, "Float4_e2m1fn_x2 is only supported for ROCm 7.0 and above");
 531:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 532-553
```cpp
 532:   if (mat1.scalar_type() == ScalarType::Float8_e5m2 || mat2.scalar_type() == ScalarType::Float8_e5m2) {
 533:     TORCH_CHECK(ROCM_VERSION >= 60500, "Float8_e5m2 is only supported for ROCm 6.5 and above");
 534:   }
 535:   if (mat1.scalar_type() == ScalarType::Float8_e4m3fn || mat2.scalar_type() == ScalarType::Float8_e4m3fn) {
 536:     TORCH_CHECK(ROCM_VERSION >= 60500, "Float8_e4m3fn is only supported for ROCm 6.5 and above");
 537:   }
 538: #endif
 539:   if (bias) {
 540:     TORCH_CHECK(out.scalar_type() != kFloat,
 541:         "Bias is not supported when out_dtype is set to Float32");
 542: 
 543:     TORCH_CHECK(bias->scalar_type() == ScalarType::BFloat16 ||
 544:                 bias->scalar_type() == ScalarType::Half,
 545:         "Bias must be BFloat16 or Half, but got ", bias->scalar_type());
 546: 
 547:     TORCH_CHECK((out.scalar_type() != kFloat &&
 548:                  out.scalar_type() != ScalarType::BFloat16) ||
 549:                 bias->scalar_type() == ScalarType::BFloat16,
 550:         "Bias must be BFloat16 to compute ", out.scalar_type(),
 551:         " output, but got ", bias->scalar_type());
 552: 
 553:     TORCH_CHECK(out.scalar_type() != ScalarType::Half ||
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 554-560
```cpp
 554:                 bias->scalar_type() == ScalarType::Half,
 555:         "Bias must be Float16 to compute ", out.scalar_type(),
 556:         " output, but got ", bias->scalar_type());
 557:   }
 558:   {
 559:     auto bias_ = bias.value_or(Tensor());
 560:     auto scale_result_ = scale_result.value_or(Tensor());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 562-562
```cpp
 562:     // NOLINTNEXTLINE(*c-array*)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 563-571
```cpp
 563:     TensorArg targs[]{{out, "out", 0}, {mat1, "mat1", 1}, {mat2, "mat2", 2},
 564:                       {bias_, "bias", 3}, {scale_a, "scale_a", 4}, {scale_b, "scale_b", 5},
 565:                       {scale_result_, "scale_result", 6}};
 566:     checkAllSameGPU(__func__, targs);
 567:   }
 568:   // Validation checks have passed lets resize the output to actual size
 569:   IntArrayRef mat1_sizes = mat1.sizes();
 570:   IntArrayRef mat2_sizes = mat2.sizes();
 571:   at::native::resize_output(out, {mat1_sizes[0], mat2_sizes[1]});
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 573-574
```cpp
 573:   // If any of M, K, N is 0 - return early (the tensorwise/rowwise float8 gemm kernels
 574:   // do not support this case).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 575-584
```cpp
 575:   if (mat1_sizes[0] == 0 || mat1_sizes[1] == 0 || mat2_sizes[1] == 0) {
 576:     // `out` was created with `at::empty`. In the case where we are multiplying
 577:     // MxK by KxN and K is the zero dim, we need to initialize here to properly
 578:     // return a tensor of zeros.
 579:     if (mat1_sizes[1] == 0) {
 580:       out.zero_();
 581:     }
 582: 
 583:     return out;
 584:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 586-588
```cpp
 586:   // NVIDIA's cuBLAS only started supporting row-wise scaling in version 12.9,
 587:   // and only for compute capability 9.0+. In other cases we use CUTLASS.
 588:   // We are doing row-wise scaling
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 589-610
```cpp
 589:   if (scaling_choice_a == ScalingType::RowWise && scaling_choice_b == ScalingType::RowWise) {
 590: #ifndef USE_ROCM
 591:     auto dprops = at::cuda::getCurrentDeviceProperties();
 592:     if ((dprops->major < 9 || CUBLAS_VERSION < 120900 || cublasLtGetVersion() < 120900)
 593:         // cuBLAS only supports tiled 1D factor layout for 1D block scaling, no 2D block scales
 594:         ||  (dprops->major >= 10 && (!scale_a.sizes().empty() || !scale_b.sizes().empty()))) {
 595:       TORCH_CHECK_VALUE(out.dtype() == kBFloat16 || out.dtype() == kHalf, "Only bf16 and fp16 high precision output types are supported for row-wise scaling.");
 596:       return _scaled_rowwise_rowwise(
 597:           mat1,
 598:           mat2,
 599:           scale_a,
 600:           scale_b,
 601:           bias,
 602:           out.scalar_type(),
 603:           use_fast_accum,
 604:           out);
 605:     }
 606: #else
 607:     // For ROCm, match behavior of f8f8bf16_rowwise type checking, for unit test purposes.
 608:     Tensor b = mat2;
 609:     if (_scaled_mm_is_fnuz()) {
 610:       TORCH_CHECK_VALUE(b.dtype() == at::kFloat8_e4m3fnuz,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 611-624
```cpp
 611:           "Expected b.dtype() == at::kFloat8_e4m3fnuz, got: ", b.dtype());
 612:     }
 613:     else {
 614:       TORCH_CHECK_VALUE(b.dtype() == at::kFloat8_e4m3fn,
 615:           "Expected b.dtype() == at::kFloat8_e4m3fn, got: ", b.dtype());
 616:     }
 617:     // Until more than bf16 is supported.
 618:     TORCH_CHECK_VALUE(out.scalar_type() == ScalarType::BFloat16,
 619:          "hipblaslt rowwise _scaled_mm only supports BFloat16 output but got ", out.scalar_type());
 620: #endif
 621:   }
 622:   else if (scaling_choice_a == ScalingType::BlockWise1x32 && scaling_choice_b == ScalingType::BlockWise1x32) {
 623: #ifdef USE_ROCM
 624:     #if ROCM_VERSION >= 70000
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 625-626
```cpp
 625:     TORCH_CHECK_NOT_IMPLEMENTED(at::detail::getCUDAHooks().isGPUArch({"gfx950"}),
 626:                 "Block-wise scaling for Float8_e8m0fnu is only supported on gfx950");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 628-636
```cpp
 628:     int packed_factor = 1;
 629:     if (mat1.scalar_type() == ScalarType::Float4_e2m1fn_x2) {
 630:       // For float4 data type, each byte stores two 4-bit floating-point values,
 631:       // effectively packing two elements into one byte.
 632:       packed_factor = 2;
 633:     }
 634:     TORCH_CHECK_VALUE(mat1.size(0) % 16 == 0 && (mat1.size(1) * packed_factor) % 128 == 0 &&
 635:                 mat2.size(1) % 16 == 0,
 636:                 "M, N must be multiples of 16 and K must be multiple of 128 for block-wise scaling");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 638-645
```cpp
 638:     TORCH_CHECK_VALUE(out.scalar_type() == ScalarType::BFloat16 ||
 639:                 out.scalar_type() == ScalarType::Half,
 640:                 "Block-wise scaling only supports BFloat16 or Half output types");
 641: #else
 642:     TORCH_CHECK_NOT_IMPLEMENTED(false, "Block-wise scaling for Float8_e8m0fnu requires ROCm 7.0 or later");
 643: #endif
 644: #endif
 645:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 647-648
```cpp
 647:   return _scaled_gemm(mat1, mat2, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
 648: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 650-662
```cpp
 650: Tensor
 651: _scaled_mm_cuda(const Tensor& mat_a, const Tensor& mat_b,
 652:           const Tensor& scale_a,
 653:           const Tensor& scale_b,
 654:           const std::optional<at::Tensor>& bias,
 655:           const std::optional<at::Tensor>& scale_result,
 656:           std::optional<c10::ScalarType> out_dtype,
 657:           bool use_fast_accum) {
 658:   const auto out_dtype_ = out_dtype.value_or(mat_a.scalar_type());
 659:   Tensor out = at::empty({0}, mat_a.options().dtype(out_dtype_));
 660: 
 661:   return _scaled_mm_out_cuda(mat_a, mat_b, scale_a, scale_b, bias, scale_result, out_dtype, use_fast_accum, out);
 662: }
```
- EN: This block defines or continues the implementation of `_scaled_mm_cuda`.
- CN: 该代码块定义或继续实现 `_scaled_mm_cuda`。

### Lines 664-685
```cpp
 664: namespace {
 665: 
 666: using acceptance_fn = std::function<bool(c10::ScalarType, std::vector<ScalingType>&, ArrayRef<Tensor>&, c10::ScalarType, std::vector<ScalingType>&, ArrayRef<Tensor>&)>;
 667: using namespace std::placeholders;
 668: 
 669: std::array<std::tuple<std::string, acceptance_fn, ScaledGemmImplementation>, 9> scale_kernel_dispatch = {{
 670:   { "tensorwise_tensorwise", scaled_blas::check_tensorwise_recipe, ScaledGemmImplementation::TENSORWISE_TENSORWISE },
 671:   { "rowwise_rowwise", scaled_blas::check_rowwise_recipe, ScaledGemmImplementation::ROWWISE_ROWWISE},
 672:   { "block_1x128_128x128", std::bind(scaled_blas::check_deepseek_recipe, ScalingType::BlockWise1x128, ScalingType::BlockWise128x128, _1, _2, _3, _4, _5, _6),
 673:     ScaledGemmImplementation::BLOCK_1x128_128x128},
 674:   { "block_128x128_1x128", std::bind(scaled_blas::check_deepseek_recipe, ScalingType::BlockWise128x128, ScalingType::BlockWise1x128, _1, _2, _3, _4, _5, _6),
 675:     ScaledGemmImplementation::BLOCK_128x128_1x128},
 676:   { "block_1x128_1x128", std::bind(scaled_blas::check_deepseek_recipe, ScalingType::BlockWise1x128, ScalingType::BlockWise1x128, _1, _2, _3, _4, _5, _6),
 677:     ScaledGemmImplementation::BLOCK_1x128_1x128},
 678:   { "nvfp4_nvfp4", scaled_blas::check_nvfp4_recipe, ScaledGemmImplementation::NVFP4_NVFP4},
 679:   { "nvfp4_nvfp4_single_scale", scaled_blas::check_nvfp4_recipe_single_scale, ScaledGemmImplementation::NVFP4_NVFP4_SINGLE_SCALE },
 680:   { "mxfp8_mxfp8", scaled_blas::check_mxfp8_recipe, ScaledGemmImplementation::MXFP8_MXFP8},
 681:   { "mxfp4_mxfp4", scaled_blas::check_mxfp4_recipe, ScaledGemmImplementation::MXFP4_MXFP4}}};
 682: 
 683: Tensor&
 684: _scaled_tensorwise_tensorwise(
 685:           const Tensor& mat_a, const Tensor& mat_b,
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 686-705
```cpp
 686:           const Tensor& scale_a, const Tensor& scale_b,
 687:           const std::optional<Tensor>& bias,
 688:           const c10::ScalarType out_dtype,
 689:           bool use_fast_accum,
 690:           Tensor& out) {
 691:   // Restrictions:
 692:   // A, B are FP8, scales are fp32
 693:   //
 694:   TORCH_CHECK_VALUE(isFloat8Type(mat_a.scalar_type()) && isFloat8Type(mat_b.scalar_type()), "mat_a and mat_b must be fp8 types, got: ",
 695:       mat_a.scalar_type(), mat_b.scalar_type());
 696:   TORCH_CHECK_VALUE(scale_a.numel() == 1 && scale_a.scalar_type() == kFloat, "scale_a must have 1 Float element")
 697:   TORCH_CHECK_VALUE(scale_b.numel() == 1 && scale_b.scalar_type() == kFloat, "scale_b must have 1 Float element")
 698: 
 699:   auto scaling_choice_a = ScalingType::TensorWise;
 700:   auto scaling_choice_b = ScalingType::TensorWise;
 701: 
 702:   _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
 703: 
 704:   return out;
 705: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 708-729
```cpp
 708: Tensor&
 709: _scaled_rowwise_rowwise(
 710:           const Tensor& mat_a, const Tensor& mat_b,
 711:           const Tensor& scale_a, const Tensor& scale_b,
 712:           const std::optional<Tensor>& bias,
 713:           const c10::ScalarType out_dtype,
 714:           bool use_fast_accum,
 715:           Tensor& out) {
 716:   // Restrictions:
 717:   // A, B are FP8, scales are fp32, shape M/N for A/B
 718:   TORCH_CHECK_VALUE(isFloat8Type(mat_a.scalar_type()) && isFloat8Type(mat_b.scalar_type()), "mat_a and mat_b must be fp8 types, got: ",
 719:       mat_a.scalar_type(), mat_b.scalar_type());
 720:   TORCH_CHECK_VALUE(scale_a.size(0) == mat_a.size(0) && scale_a.size(1) == 1, "scale_a must have shape [", mat_a.size(0), ", 1], got [", scale_a.sizes(), "]");
 721:   TORCH_CHECK_VALUE(scale_a.numel() == mat_a.size(0) && scale_a.scalar_type() == kFloat, "scale_a must have ", mat_a.size(0), " Float elements, got ", scale_a.numel())
 722:   TORCH_CHECK_VALUE(scale_b.numel() == mat_b.size(1) && scale_b.scalar_type() == kFloat, "scale_b must have ", mat_b.size(1), " Float elements, got ", scale_b.numel())
 723: 
 724:   // if we have a scale of shape [256, 1] (say), then stride can be [1, 0] - handle this case
 725:   TORCH_CHECK_VALUE(
 726:       scale_a.stride(1) == 1 ||
 727:       scale_a.size(1) == 1,
 728:       "expected scale_a.stride(1) to be 1, but got ", scale_a.stride(1)
 729:   );
```
- EN: This block defines or continues the implementation of `_scaled_rowwise_rowwise`.
- CN: 该代码块定义或继续实现 `_scaled_rowwise_rowwise`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 730-730
```cpp
 730:   TORCH_CHECK_VALUE(scale_b.stride(1) == 1, "expected scale_b.stride(1) to be 1, but got ", scale_b.stride(1));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 732-753
```cpp
 732:   auto scaling_choice_a = ScalingType::RowWise;
 733:   auto scaling_choice_b = ScalingType::RowWise;
 734:   //
 735:   // NVIDIA's cuBLAS only started supporting row-wise scaling in version 12.9,
 736:   // and only for compute capability 9.0+. In other cases we use CUTLASS.
 737: #ifndef USE_ROCM
 738:   // We are doing row-wise scaling
 739:   auto dprops = at::cuda::getCurrentDeviceProperties();
 740:   if (((dprops->major < 9 || CUBLAS_VERSION < 120900 || cublasLtGetVersion() < 120900)
 741:       // cuBLAS only supports tiled 1D factor layout for 1D block scaling, no 2D block scales
 742:       ||  (dprops->major >= 10 && (!scale_a.sizes().empty() || !scale_b.sizes().empty())))) {
 743:     TORCH_CHECK_VALUE(out.dtype() == kBFloat16 || out.dtype() == kHalf, "Only bf16 and fp16 high precision output types are supported for row-wise scaling.");
 744:     at::cuda::detail::f8f8bf16_rowwise(
 745:         mat_a,
 746:         mat_b,
 747:         scale_a,
 748:         scale_b,
 749:         bias,
 750:         use_fast_accum,
 751:         out);
 752:     return out;
 753:   }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 754-754
```cpp
 754: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 756-757
```cpp
 756:   // For ROCm, match behavior of f8f8bf16_rowwise type checking, for unit test purposes.
 757:   //Tensor b = mat_b;
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 758-767
```cpp
 758:   if (_scaled_mm_is_fnuz()) {
 759:     TORCH_CHECK_VALUE(mat_b.dtype() == at::kFloat8_e4m3fnuz, "expected mat_b.dtype() to be at::kFloat8_e4m3fnuz, but got ", mat_b.dtype());
 760:   }
 761:   else {
 762:     TORCH_CHECK_VALUE(mat_b.dtype() == at::kFloat8_e4m3fn, "expected mat_b.dtype() to be at::kFloat8_e4m3fn, but got ", mat_b.dtype());
 763:   }
 764:   // Until more than bf16 is supported.
 765:   TORCH_CHECK_VALUE(out.scalar_type() == ScalarType::BFloat16,
 766:        "hipblaslt rowwise _scaled_mm only supports BFloat16 output but got ", out.scalar_type());
 767: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 769-769
```cpp
 769:   _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 771-772
```cpp
 771:   return out;
 772: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 774-790
```cpp
 774: void
 775: _check_deepseek_support() {
 776: #ifndef USE_ROCM
 777:   auto dprops = at::cuda::getCurrentDeviceProperties();
 778:   if (dprops->major != 9) {
 779:     // Only on Hopper GPUs
 780:     TORCH_CHECK_NOT_IMPLEMENTED(
 781:       dprops->major == 9,
 782:       "DeepSeek style (1x128, 128x128) scaling only supported in CUDA for SM90")
 783:   }
 784:   // Only in cublasLt >= 12.9
 785:   TORCH_CHECK_NOT_IMPLEMENTED(
 786:     CUBLAS_VERSION >= 120900 && cublasLtGetVersion() >= 120900,
 787:     "DeepSeek style (1x128, 128x128) scaling requires cublasLt >= 12.9"
 788:   );
 789: #endif
 790: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_check_deepseek_support`.
- CN: 该代码块定义或继续实现 `_check_deepseek_support`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 792-812
```cpp
 792: Tensor&
 793: _scaled_block1x128_block1x128(
 794:           const Tensor& mat_a, const Tensor& mat_b,
 795:           const Tensor& scale_a, const Tensor& scale_b,
 796:           const std::optional<Tensor>& bias,
 797:           const c10::ScalarType out_dtype,
 798:           const bool use_fast_accum,
 799:           Tensor& out) {
 800: #ifndef USE_ROCM
 801:   // Restrictions:
 802:   // A, B are FP8, scales are fp32, shape K//128
 803:   // As: [M x K // 128], stride: [1, M]
 804:   // Bs: [N x K // 128], stride: [1, N]
 805:   _check_deepseek_support();
 806: 
 807:   // check types
 808:   TORCH_CHECK_VALUE(
 809:     isFloat8Type(mat_a.scalar_type()) &&
 810:     isFloat8Type(mat_b.scalar_type()),
 811:     "mat_a and mat_b must be fp8 types, got: ", mat_a.scalar_type(), mat_b.scalar_type()
 812:   );
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_block1x128_block1x128`.
- CN: 该代码块定义或继续实现 `_scaled_block1x128_block1x128`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 814-816
```cpp
 814:   const int64_t M = mat_a.sizes()[0];
 815:   const int64_t K = mat_a.sizes()[1];
 816:   const int64_t N = mat_b.sizes()[1];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 818-818
```cpp
 818:   // scale_a shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 819-832
```cpp
 819:   TORCH_CHECK_VALUE(
 820:     scale_a.size(0) == M &&
 821:     scale_a.size(1) == ceil_div<int64_t>(K, 128) &&
 822:     scale_a.scalar_type() == kFloat,
 823:     "scale_a must have shape ", M, " x ", ceil_div<int64_t>(K, 128), " Float elements, got ", scale_a.sizes()
 824:   );
 825:   // scale_a stride
 826:   TORCH_CHECK_VALUE(
 827:     scale_a.stride(0) == 1 &&
 828:     (
 829:       scale_a.stride(1) == M ||
 830:       (scale_a.size(1) == 1 && scale_a.stride(1) == 1)
 831:     ),
 832:     "scale_a strides must be (", 1, ", ", M, "); got: ", scale_a.strides()
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 833-833
```cpp
 833:   );
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 835-835
```cpp
 835:   // scale_b shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 836-849
```cpp
 836:   TORCH_CHECK_VALUE(
 837:     scale_b.size(0) == N &&
 838:     scale_b.size(1) == ceil_div<int64_t>(K, 128) &&
 839:     scale_b.scalar_type() == kFloat,
 840:     "scale_b must have shape ", N, " x ", ceil_div<int64_t>(K, 128), " Float elements, got ", scale_b.sizes()
 841:   );
 842:   // scale_b stride
 843:   TORCH_CHECK_VALUE(
 844:     scale_b.stride(0) == 1 &&
 845:     (
 846:       scale_b.stride(1) == N ||
 847:       (
 848:         scale_b.size(1) == 1 &&
 849:         scale_b.stride(1) == 1
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 850-853
```cpp
 850:       )
 851:     ),
 852:     "scale_b strides must be (", 1, ", ", N, "); got: ", scale_b.strides()
 853:   );
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 855-856
```cpp
 855:   auto scaling_choice_a = ScalingType::BlockWise1x128;
 856:   auto scaling_choice_b = ScalingType::BlockWise1x128;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 858-858
```cpp
 858:   _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 860-867
```cpp
 860:   return out;
 861: #else
 862:   TORCH_CHECK_NOT_IMPLEMENTED(
 863:     false,
 864:     "1x128 and 128x128 scaling not available with ROCm"
 865:   );
 866: #endif
 867: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 869-890
```cpp
 869: Tensor&
 870: _scaled_block128x128_block1x128(
 871:           const Tensor& mat_a, const Tensor& mat_b,
 872:           const Tensor& scale_a, const Tensor& scale_b,
 873:           const std::optional<Tensor>& bias,
 874:           const c10::ScalarType out_dtype,
 875:           const bool use_fast_accum,
 876:           Tensor& out) {
 877: #ifndef USE_ROCM
 878:   // Restrictions:
 879:   _check_deepseek_support();
 880: 
 881:   // A: [M, K], B: [K, N] are FP8, scales are fp32
 882:   // As: [round_up(K // 128, 4), M // 128], stride: [M // 128, 1]
 883:   // Bs: [N x K // 128], stride: [1, N]
 884:   TORCH_CHECK_VALUE(
 885:     isFloat8Type(mat_a.scalar_type()) &&
 886:     isFloat8Type(mat_b.scalar_type()),
 887:     "mat_a and mat_b must be fp8 types, got: ",  mat_a.scalar_type(), mat_b.scalar_type()
 888:   );
 889: 
 890:   const int64_t M = mat_a.sizes()[0];
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_block128x128_block1x128`.
- CN: 该代码块定义或继续实现 `_scaled_block128x128_block1x128`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 891-892
```cpp
 891:   const int64_t K = mat_a.sizes()[1];
 892:   const int64_t N = mat_b.sizes()[1];
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 894-894
```cpp
 894:   // scale_a shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 895-908
```cpp
 895:   TORCH_CHECK_VALUE(
 896:     scale_a.size(0) == round_up<int64_t>(ceil_div<int64_t>(K, 128), 4) &&
 897:     scale_a.size(1) == ceil_div<int64_t>(M, 128) &&
 898:     scale_a.scalar_type() == kFloat,
 899:     "scale_a must have shape ", round_up<int64_t>(ceil_div<int64_t>(K, 128), 4), " x ",
 900:       ceil_div<int64_t>(M, 128), " Float elements, got ", scale_a.sizes()
 901:   );
 902:   // scale_a stride
 903:   TORCH_CHECK_VALUE(
 904:     scale_a.stride(0) == 1 &&
 905:     (
 906:       scale_a.stride(1) == round_up<int64_t>(ceil_div<int64_t>(K, 128), 4) ||
 907:       (
 908:         scale_a.size(1) == 1 &&
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 909-913
```cpp
 909:         scale_a.stride(1) == 1
 910:       )
 911:     ),
 912:     "scale_a must have strides (1, ", round_up<int64_t>(ceil_div<int64_t>(K, 128), 4), "); got ", scale_b.strides()
 913:   );
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 915-915
```cpp
 915:   // scale_b shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 916-929
```cpp
 916:   TORCH_CHECK_VALUE(
 917:     scale_b.size(0) == N &&
 918:     scale_b.size(1) == ceil_div<int64_t>(K, 128) &&
 919:     scale_b.scalar_type() == kFloat,
 920:     "scale_b must have shape ", N, " x ", ceil_div<int64_t>(K, 128), " Float elements, got ", scale_b.sizes()
 921:   );
 922:   // scale_b stride
 923:   TORCH_CHECK_VALUE(
 924:     scale_b.stride(0) == 1 &&
 925:     (
 926:       scale_b.stride(1) == N ||
 927:       (
 928:         scale_b.size(1) == 1 &&
 929:         scale_b.stride(1) == 1
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 930-933
```cpp
 930:       )
 931:     ),
 932:     "scale_b must have strides (1, ", N, "); got ", scale_b.strides()
 933:   );
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 935-936
```cpp
 935:   auto scaling_choice_a = ScalingType::BlockWise128x128;
 936:   auto scaling_choice_b = ScalingType::BlockWise1x128;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 938-938
```cpp
 938:   _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 940-947
```cpp
 940:   return out;
 941: #else
 942:   TORCH_CHECK_NOT_IMPLEMENTED(
 943:     false,
 944:     "1x128 and 128x128 scaling not available with ROCm"
 945:   );
 946: #endif
 947: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 949-970
```cpp
 949: Tensor&
 950: _scaled_block1x128_block128x128(
 951:           const Tensor& mat_a, const Tensor& mat_b,
 952:           const Tensor& scale_a, const Tensor& scale_b,
 953:           const std::optional<Tensor>& bias,
 954:           const c10::ScalarType out_dtype,
 955:           const bool use_fast_accum,
 956:           Tensor& out) {
 957: #ifndef USE_ROCM
 958:   // Restrictions:
 959:   _check_deepseek_support();
 960:   // A: [M, K], B: [K, N] are FP8, scales are fp32
 961:   // As: [M x K // 128], stride: [1, M]
 962:   // Bs: [round_up(K // 128, 4) x N // 128], stride: [1, N // 128]
 963:   TORCH_CHECK_VALUE(
 964:     isFloat8Type(mat_a.scalar_type()) &&
 965:     isFloat8Type(mat_b.scalar_type()),
 966:     "mat_a and mat_b must be fp8 types, got: ", mat_a.scalar_type(), mat_b.scalar_type()
 967:   );
 968: 
 969:   int64_t M = mat_a.size(0);
 970:   int64_t K = mat_a.size(1);
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_block1x128_block128x128`.
- CN: 该代码块定义或继续实现 `_scaled_block1x128_block128x128`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 971-971
```cpp
 971:   int64_t N = mat_b.size(1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 973-973
```cpp
 973:   // scale_a shape
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 974-987
```cpp
 974:   TORCH_CHECK_VALUE(
 975:     scale_a.size(0) == M &&
 976:     scale_a.size(1) == ceil_div<int64_t>(K, 128) &&
 977:     scale_a.scalar_type() == kFloat,
 978:     "scale_a must have shape ", M, " x ", ceil_div<int64_t>(K, 128), " Float elements, got ", scale_a.sizes()
 979:   );
 980:   // scale_a stride
 981:   TORCH_CHECK_VALUE(
 982:     scale_a.stride(0) == 1 &&
 983:     (
 984:       scale_a.stride(1) == M ||
 985:       (
 986:         scale_a.size(1) == 1 &&
 987:         scale_a.stride(1) == 1
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 988-1001
```cpp
 988:       )
 989:     ),
 990:     "scale_a must have strides (1, ", M, "); got ", scale_a.strides()
 991:   );
 992:   // scale_b shape
 993:   TORCH_CHECK_VALUE(
 994:     scale_b.size(0) == round_up<int64_t>(ceil_div<int64_t>(K, 128), 4) &&
 995:     scale_b.size(1) == ceil_div<int64_t>(N, 128) &&
 996:     scale_b.scalar_type() == kFloat,
 997:     "scale_b must have shape ", round_up<int64_t>(ceil_div<int64_t>(K, 128), 4), " x ", ceil_div<int64_t>(N, 128), " Float elements, got ", scale_b.sizes()
 998:   );
 999:   // scale_b stride
1000:   TORCH_CHECK_VALUE(
1001:     scale_b.stride(0) == 1 &&
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1002-1010
```cpp
1002:     (
1003:       scale_b.stride(1) == round_up<int64_t>(ceil_div<int64_t>(K, 128), 4) ||
1004:       (
1005:         scale_b.size(1) == 1 &&
1006:         scale_b.stride(1) == 1
1007:       )
1008:     ),
1009:     "scale_b must have strides (1, ", round_up<int64_t>(ceil_div<int64_t>(K, 128), 4), "); got ", scale_b.strides()
1010:   );
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1012-1013
```cpp
1012:   auto scaling_choice_a = ScalingType::BlockWise1x128;
1013:   auto scaling_choice_b = ScalingType::BlockWise128x128;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1015-1015
```cpp
1015:   _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, use_fast_accum, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1017-1024
```cpp
1017:   return out;
1018: #else
1019:   TORCH_CHECK_NOT_IMPLEMENTED(
1020:     false,
1021:     "1x128 and 128x128 scaling not available with ROCm"
1022:   );
1023: #endif
1024: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1026-1047
```cpp
1026: Tensor&
1027: _scaled_mxfp8_mxfp8(
1028:           const Tensor& mat_a, const Tensor& mat_b,
1029:           const Tensor& scale_a, const SwizzleType swizzle_a,
1030:           const Tensor& scale_b, const SwizzleType swizzle_b,
1031:           const std::optional<Tensor>& bias,
1032:           const c10::ScalarType out_dtype,
1033:           Tensor& out) {
1034:   // Restrictions:
1035:   // A, B are FP8, scales are e8m0, A: shape K//32, B: K, N//32
1036:   // Scales must be swizzled
1037:   TORCH_CHECK_VALUE(isFloat8Type(mat_a.scalar_type()) && isFloat8Type(mat_b.scalar_type()), "mat_a and mat_b must be fp8 types, got: ",
1038:       mat_a.scalar_type(), mat_b.scalar_type());
1039: 
1040: #ifdef USE_ROCM
1041:   auto scale_a_elems = ceil_div<int64_t>(mat_a.size(0), 32) * mat_a.size(1);
1042:   auto scale_b_elems = ceil_div<int64_t>(mat_b.size(1), 32) * mat_b.size(0);
1043: #else
1044:   auto scale_a_elems = round_up<int64_t>(mat_a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_a.size(1), 32), 4);
1045:   auto scale_b_elems = round_up<int64_t>(mat_b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_b.size(0), 32), 4);
1046: #endif
1047:   TORCH_CHECK_VALUE(scale_a_elems == scale_a.numel(),
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_mxfp8_mxfp8`.
- CN: 该代码块定义或继续实现 `_scaled_mxfp8_mxfp8`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1048-1050
```cpp
1048:          "For Blockwise scaling scale_a should have ", scale_a_elems, " elements, got: ", scale_a.numel());
1049:   TORCH_CHECK_VALUE(scale_b_elems == scale_b.numel(),
1050:          "For Blockwise scaling scale_b should have ", scale_b_elems, " elements, got: ", scale_b.numel());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1052-1052
```cpp
1052: #ifndef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1053-1055
```cpp
1053:   TORCH_CHECK_VALUE(swizzle_a == SwizzleType::SWIZZLE_32_4_4, "scale_a must be swizzled to SWIZZLE_32_4_4 format");
1054:   TORCH_CHECK_VALUE(swizzle_b == SwizzleType::SWIZZLE_32_4_4, "scale_b must be swizzled to SWIZZLE_32_4_4 format");
1055: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1057-1058
```cpp
1057:   TORCH_CHECK_VALUE(scale_a.is_contiguous() && scale_b.is_contiguous(),
1058:         "For Blockwise scaling both scales should be contiguous");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1060-1060
```cpp
1060:   TORCH_CHECK_VALUE(out.scalar_type() == out_dtype, "expected out.scalar_type() to be ", out_dtype, ", but got ", out_dtype);
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1062-1063
```cpp
1062:   auto scaling_choice_a = ScalingType::BlockWise1x32;
1063:   auto scaling_choice_b = ScalingType::BlockWise1x32;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1065-1066
```cpp
1065: #ifdef USE_ROCM
1066: #if ROCM_VERSION >= 70000
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1067-1068
```cpp
1067:   TORCH_CHECK_NOT_IMPLEMENTED(at::detail::getCUDAHooks().isGPUArch({"gfx950"}),
1068:               "Block-wise scaling for Float8_e8m0fnu is only supported on gfx950");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1070-1072
```cpp
1070:   TORCH_CHECK_VALUE(mat_a.size(0) % 32 == 0 && mat_a.size(1) % 32 == 0 &&
1071:               mat_b.size(0) % 32 == 0 && mat_b.size(1) % 32 == 0,
1072:               "Matrix dimensions must be multiples of 32 for block-wise scaling");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1074-1080
```cpp
1074:   TORCH_CHECK_VALUE(out.scalar_type() == ScalarType::BFloat16 ||
1075:               out.scalar_type() == ScalarType::Half,
1076:               "Block-wise scaling only supports BFloat16 or Half output types");
1077: #else
1078:     TORCH_CHECK_NOT_IMPLEMENTED(false, "Block-wise scaling for Float8_e8m0fnu requires ROCm 7.0 or later");
1079: #endif
1080: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1082-1083
```cpp
1082:   return _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, false /* use_fast_accum */, out);
1083: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1085-1096
```cpp
1085: void
1086: _check_mxfp4_support() {
1087: #ifndef USE_ROCM
1088:   auto dprops = at::cuda::getCurrentDeviceProperties();
1089:   // Only on B200 GPUs
1090:   TORCH_CHECK_NOT_IMPLEMENTED(
1091:     // B200 = 10.0, B300 = 10.3
1092:     dprops->major == 10,
1093:     "MXFP4 scaling only supported in CUDA for B200/B300"
1094:   );
1095: #endif
1096: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_check_mxfp4_support`.
- CN: 该代码块定义或继续实现 `_check_mxfp4_support`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1099-1120
```cpp
1099: Tensor&
1100: _scaled_mxfp4_mxfp4(
1101:           const Tensor& mat_a, const Tensor& mat_b,
1102:           const Tensor& scale_a, const SwizzleType swizzle_a,
1103:           const Tensor& scale_b, const SwizzleType swizzle_b,
1104:           const std::optional<Tensor>& bias,
1105:           const c10::ScalarType out_dtype,
1106:           Tensor& out) {
1107: #if defined(_WIN32) || (!defined(USE_ROCM) && !defined(USE_MSLK))
1108:   TORCH_CHECK_NOT_IMPLEMENTED(false, "MXFP4 scaling supported on ROCM and CUDA+FBGEMM_GENAI only");
1109: #else
1110:   _check_mxfp4_support();
1111:   // Restrictions:
1112:   // A, B are FP4, scales are e8m0, A: shape K//32, B: K, N//32
1113:   TORCH_CHECK_VALUE(mat_a.scalar_type() == at::kFloat4_e2m1fn_x2 && mat_b.scalar_type() == at::kFloat4_e2m1fn_x2, "mat_a and mat_b must be fp4 types, got: ",
1114:       mat_a.scalar_type(), mat_b.scalar_type());
1115: 
1116:   // Packed FP4 format means actual-K = 2 * reported-K -- adjust
1117:   auto K_multiplier = 2;
1118: #ifdef USE_ROCM
1119:   // AMD
1120:   auto scale_a_elems = ceil_div<int64_t>(K_multiplier * mat_a.size(0), 32) * mat_a.size(1);
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_mxfp4_mxfp4`.
- CN: 该代码块定义或继续实现 `_scaled_mxfp4_mxfp4`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1121-1130
```cpp
1121:   auto scale_b_elems = ceil_div<int64_t>(K_multiplier * mat_b.size(1), 32) * mat_b.size(0);
1122: #else
1123:   // NVIDIA
1124:   auto scale_a_elems = round_up<int64_t>(mat_a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(K_multiplier * mat_a.size(1), 32), 4);
1125:   auto scale_b_elems = round_up<int64_t>(mat_b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(K_multiplier * mat_b.size(0), 32), 4);
1126: #endif
1127:   TORCH_CHECK_VALUE(scale_a_elems == scale_a.numel(),
1128:          "For Blockwise scaling scale_a should have ", scale_a_elems, " elements, got: ", scale_a.numel());
1129:   TORCH_CHECK_VALUE(scale_b_elems == scale_b.numel(),
1130:          "For Blockwise scaling scale_b should have ", scale_b_elems, " elements, got: ", scale_b.numel());
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1132-1132
```cpp
1132: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1133-1133
```cpp
1133:   // AMD
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1134-1140
```cpp
1134:   TORCH_CHECK_VALUE(swizzle_a == SwizzleType::NO_SWIZZLE, "scale_a must not be swizzled (NO_SWIZZLE format)");
1135:   TORCH_CHECK_VALUE(swizzle_b == SwizzleType::NO_SWIZZLE, "scale_b must not be swizzled (NO_SWIZZLE format)");
1136: #else
1137:   // NVIDIA
1138:   TORCH_CHECK_VALUE(swizzle_a == SwizzleType::SWIZZLE_32_4_4, "scale_a must be swizzled to SWIZZLE_32_4_4 format");
1139:   TORCH_CHECK_VALUE(swizzle_b == SwizzleType::SWIZZLE_32_4_4, "scale_b must be swizzled to SWIZZLE_32_4_4 format");
1140: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1142-1143
```cpp
1142:   TORCH_CHECK_VALUE(scale_a.is_contiguous() && scale_b.is_contiguous(),
1143:         "For Blockwise scaling both scales should be contiguous");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1145-1145
```cpp
1145:   TORCH_CHECK_VALUE(out.scalar_type() == out_dtype, "expected out.scalar_type() to be ", out_dtype, ", but got ", out_dtype);
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1147-1147
```cpp
1147: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1148-1148
```cpp
1148:   // AMD
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1149-1150
```cpp
1149:   auto scaling_choice_a = ScalingType::BlockWise1x32;
1150:   auto scaling_choice_b = ScalingType::BlockWise1x32;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1152-1152
```cpp
1152: #if ROCM_VERSION >= 70000
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1153-1154
```cpp
1153:   TORCH_CHECK_NOT_IMPLEMENTED(at::detail::getCUDAHooks().isGPUArch({"gfx950"}),
1154:               "Block-wise scaling for Float8_e8m0fnu is only supported on gfx950");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1156-1158
```cpp
1156:   TORCH_CHECK_VALUE(mat_a.size(0) % 32 == 0 && mat_a.size(1) % 32 == 0 &&
1157:               mat_b.size(0) % 32 == 0 && mat_b.size(1) % 32 == 0,
1158:               "Matrix dimensions must be multiples of 32 for block-wise scaling");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1160-1163
```cpp
1160:   TORCH_CHECK_VALUE(out.scalar_type() == ScalarType::BFloat16 ||
1161:               out.scalar_type() == ScalarType::Half,
1162:               "Block-wise scaling only supports BFloat16 or Half output types");
1163: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1165-1178
```cpp
1165:   return _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, false /* use_fast_accum */, out);
1166: #else
1167:   // NVIDIA
1168:   mslk::gemm::f4f4bf16(
1169:       mat_a,
1170:       mat_b.transpose(-2, -1),
1171:       scale_a,
1172:       scale_b,
1173:       out,
1174:       std::nullopt /* global_scale */
1175:   );
1176:   return out;
1177: #endif
1178: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1179-1179
```cpp
1179: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1181-1202
```cpp
1181: Tensor&
1182: _scaled_nvfp4_nvfp4(
1183:           const Tensor& mat_a, const Tensor& mat_b,
1184:           const Tensor& scale_a, const SwizzleType swizzle_a,
1185:           const Tensor& scale_b, const SwizzleType swizzle_b,
1186:           const std::optional<Tensor>& bias,
1187:           const c10::ScalarType out_dtype,
1188:           Tensor& out,
1189:           const std::optional<Tensor>& global_scale_a = std::nullopt,
1190:           const std::optional<Tensor>& global_scale_b = std::nullopt) {
1191: #ifdef USE_ROCM
1192:   TORCH_CHECK_NOT_IMPLEMENTED(false, "NVFP4 scaling not supported on ROCM");
1193: #endif
1194:   std::optional<Tensor> alpha = std::nullopt;
1195:   // Note: "Or" here means that if only one scale is passed, we check for the other. Otherwise,
1196:   //       if this is "And" we would silently do nothing in the case where one global scale is
1197:   //       passed and not the other.
1198:   if (global_scale_a.has_value() || global_scale_b.has_value()) {
1199:     TORCH_CHECK_VALUE(global_scale_a.has_value(),
1200:         "For two-level-scaled NVFP4, global_scale_a must have a value");
1201:     TORCH_CHECK_VALUE(global_scale_b.has_value(),
1202:         "For two-level-scaled NVFP4, global_scale_b must have a value");
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `_scaled_nvfp4_nvfp4`.
- CN: 该代码块定义或继续实现 `_scaled_nvfp4_nvfp4`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1203-1216
```cpp
1203:     alpha = global_scale_a.value().mul(global_scale_b.value());
1204:   }
1205:   // Restrictions:
1206:   // A, B are FP4, scales are e8m0, A: shape K//32, B: K, N//32
1207:   // Scales must be swizzled
1208:   TORCH_CHECK_VALUE(mat_a.scalar_type() == at::kFloat4_e2m1fn_x2 && mat_b.scalar_type() == at::kFloat4_e2m1fn_x2, "mat_a and mat_b must be fp4 types, got: ",
1209:       mat_a.scalar_type(), mat_b.scalar_type());
1210:   // Note: fp4x2 format, need to double the K dimension for checking purposes.
1211:   auto scale_a_elems = round_up<int64_t>(mat_a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_a.size(1) * 2, 16), 4);
1212:   auto scale_b_elems = round_up<int64_t>(mat_b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_b.size(0) * 2, 16), 4);
1213:   TORCH_CHECK_VALUE(scale_a_elems == scale_a.numel(),
1214:          "For Blockwise scaling scale_a should have ", scale_a_elems, " elements, got: ", scale_a.numel());
1215:   TORCH_CHECK_VALUE(scale_b_elems == scale_b.numel(),
1216:          "For Blockwise scaling scale_b should have ", scale_b_elems, " elements, got: ", scale_b.numel());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1218-1219
```cpp
1218:   TORCH_CHECK_VALUE(swizzle_a == SwizzleType::SWIZZLE_32_4_4, "scale_a must be swizzled to SWIZZLE_32_4_4 format");
1219:   TORCH_CHECK_VALUE(swizzle_b == SwizzleType::SWIZZLE_32_4_4, "scale_b must be swizzled to SWIZZLE_32_4_4 format");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1221-1222
```cpp
1221:   TORCH_CHECK_VALUE(scale_a.is_contiguous() && scale_b.is_contiguous(),
1222:         "For Blockwise scaling both scales should be contiguous");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1224-1227
```cpp
1224:   auto scaling_choice_a = ScalingType::BlockWise1x16;
1225:   auto scaling_choice_b = ScalingType::BlockWise1x16;
1226:   return _scaled_gemm(mat_a, mat_b, scale_a, scale_b, scaling_choice_a, scaling_choice_b, bias, false /* use_fast_accum */, out, alpha);
1227: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1229-1250
```cpp
1229: void check_swizzle_lengths(ScaledGemmImplementation impl,
1230:                            std::vector<SwizzleType>& swizzle_a,
1231:                            std::vector<SwizzleType>& swizzle_b) {
1232:   // Store implementations that care about swizzling, and how many swizzle arguments
1233:   // they have to have
1234:   // NOTE(slayton): auto here is unable to deduce the correct type..
1235:   std::array<std::tuple<ScaledGemmImplementation, unsigned int>, 4> swizzled_impl = {{
1236:     // {implementation, # required arguments}
1237:     {ScaledGemmImplementation::MXFP8_MXFP8, 1},
1238:     {ScaledGemmImplementation::NVFP4_NVFP4, 2},
1239:     {ScaledGemmImplementation::NVFP4_NVFP4_SINGLE_SCALE, 1},
1240:     {ScaledGemmImplementation::MXFP4_MXFP4, 1}
1241:   }};
1242: 
1243:   // Only check MX/NVFP formats on NVIDIA
1244:   for (auto [check_impl, num_args] : swizzled_impl) {
1245:     if (impl != check_impl) {
1246:       continue;
1247:     }
1248: #ifdef USE_ROCM
1249:     if (
1250:         check_impl != ScaledGemmImplementation::MXFP8_MXFP8 &&
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `check_swizzle_lengths`.
- CN: 该代码块定义或继续实现 `check_swizzle_lengths`。

### Lines 1251-1264
```cpp
1251:         check_impl != ScaledGemmImplementation::MXFP4_MXFP4) {
1252:       // ROCm currently does not support NVFP4 paths.
1253:       break;
1254:     }
1255:     TORCH_CHECK_VALUE(
1256:         swizzle_a.size() == 1 && swizzle_b.size() == 1,
1257:         "For ROCM MX gemm, swizzle_a and swizzle_b must each have 1 value, got ",
1258:         swizzle_a.size(),
1259:         " and ",
1260:         swizzle_b.size());
1261:     TORCH_CHECK_VALUE(
1262:         swizzle_a[0] == SwizzleType::NO_SWIZZLE &&
1263:             swizzle_b[0] == SwizzleType::NO_SWIZZLE,
1264:         "For ROCM MX gemm, swizzle_a and swizzle_b must both be NO_SWIZZLE");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1265-1265
```cpp
1265: #else
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1266-1279
```cpp
1266:     TORCH_CHECK_VALUE(
1267:         swizzle_a.size() == num_args,
1268:         "swizzle_a must have ",
1269:         num_args,
1270:         " value",
1271:         num_args == 1 ? "" : "s",
1272:         ", got ",
1273:         swizzle_a.size());
1274:     TORCH_CHECK_VALUE(
1275:         swizzle_b.size() == num_args,
1276:         "swizzle_b must have ",
1277:         num_args,
1278:         " value",
1279:         num_args == 1 ? "" : "s",
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1280-1282
```cpp
1280:         ", got ",
1281:         swizzle_b.size());
1282: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1284-1284
```cpp
1284:     // No need to check anything else
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1285-1287
```cpp
1285:     break;
1286:   }
1287: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1289-1289
```cpp
1289: };  // anonymous namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1291-1311
```cpp
1291: // V2: Computes matrix multiply + bias while applying scaling to input and output matrices
1292: // Scales are only applicable when matrices are of Float8 type and assumed to be equal to 1.0 by default.
1293: // If output matrix type is 16 or 32-bit type, scale_result is not applied.
1294: // Known limitations:
1295: //  - Only works if mat1 is row-major and mat2 is column-major
1296: //  - Only works if matrices sizes are divisible by 32
1297: //  - If 1-dimensional tensors are used then scale_a should be size = mat1.size(0)
1298: //    and scale_b should have size = to mat2.size(1)
1299: //  Arguments:
1300: //    - `mat1`: the first operand of the matrix multiply, can be type `torch.float8_e4m3fn` or `torch.float8_e5m2`
1301: //    - `mat2`: the second operand of the matrix multiply, can be type `torch.float8_e4m3fn` or `torch.float8_e5m2`
1302: //    - `scale_a`: a tensor with the inverse scale of `mat1`, whose shape/strides/dtype depend on the scaling scheme
1303: //    - `scale_recipe_a`: An integer corresponding to an enum describing the scaling scheme used for `scale_a`
1304: //    - `swizzle_a`: An integer corresponding to a `SwizzleType` enum describing the swizzling scheme for `scale_a`
1305: //    - `scale_b`: a tensor with the inverse scale of `mat2`, whose shape/strides/dtype depend on the scaling scheme
1306: //    - `scale_recipe_b`: An integer corresponding to an enum describing the scaling scheme used for `scale_b`
1307: //    - `swizzle_b`: An integer corresponding to a `SwizzleType` enum describing the swizzling scheme for `scale_b`
1308: //    - `bias`: the bias, can be type `torch.float16` or `torch.bfloat16`
1309: //    - `out_dtype`: the output dtype, can either be a float8 or a higher precision floating point type
1310: //    - `use_fast_accum`: if true, enables fast float8 accumulation. Backends may ignore this option if not applicable.
1311: //    - `out`: a reference to the output tensor
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1312-1333
```cpp
1312: Tensor&
1313: _scaled_mm_cuda_v2_out(
1314:           const Tensor& mat_a, const Tensor& mat_b,
1315:           ArrayRef<Tensor> scale_a,
1316:           IntArrayRef scale_recipe_a,
1317:           IntArrayRef swizzle_a,
1318:           ArrayRef<Tensor> scale_b,
1319:           IntArrayRef scale_recipe_b,
1320:           IntArrayRef swizzle_b,
1321:           const std::optional<Tensor>& bias,
1322:           const std::optional<c10::ScalarType> out_dtype,
1323:           IntArrayRef contraction_dim,
1324:           bool use_fast_accum,
1325:           Tensor& out) {
1326:   // Check sizes
1327:   bool allowed_device = _scaled_mm_allowed_device();
1328:   TORCH_CHECK_NOT_IMPLEMENTED(allowed_device,
1329:       "torch._scaled_mm is only supported on CUDA devices with compute capability >= 9.0 or 8.9, or ROCm MI300+");
1330:   TORCH_CHECK_VALUE(mat_a.dim() == 2, "mat_a must be a matrix");
1331:   TORCH_CHECK_VALUE(mat_b.dim() == 2, "mat_b must be a matrix");
1332: 
1333:   // If any of M, K, N is 0 - return early (the tensorwise/rowwise float8 gemm kernels
```
- EN: This block defines or continues the implementation of `_scaled_mm_cuda_v2_out`.
- CN: 该代码块定义或继续实现 `_scaled_mm_cuda_v2_out`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1334-1334
```cpp
1334:   // do not support this case).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1335-1345
```cpp
1335:   if (mat_a.size(0) == 0 || mat_a.size(1) == 0 || mat_b.size(1) == 0) {
1336:     // `out` was created with `at::empty`. In the case where we are multiplying
1337:     // MxK by KxN and K is the zero dim, we need to initialize here to properly
1338:     // return a tensor of zeros.
1339:     at::native::resize_output(out, {mat_a.size(0), mat_b.size(1)});
1340:     if (mat_a.size(1) == 0) {
1341:       out.zero_();
1342:     }
1343: 
1344:     return out;
1345:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1347-1349
```cpp
1347:   // Check if the input matrix sizes can be multiplied
1348:   // - if optional contraction dims are provided, use those
1349:   //   -- mostly for < 1B formats (i.e. nvfp4x2) where cheap .t() is not available.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1350-1362
```cpp
1350:   if (!contraction_dim.empty()) {
1351:     TORCH_CHECK_VALUE(contraction_dim.size() == 2, "contraction_dim must have exactly 2 elements");
1352:     auto mat_a_dim = contraction_dim[0];
1353:     auto mat_b_dim = contraction_dim[1];
1354:     TORCH_CHECK_VALUE(
1355:         mat_a.size(mat_a_dim) == mat_b.size(mat_b_dim), "mat_a and mat_b shapes cannot be multiplied (",
1356:         mat_a.size(0), "x", mat_a.size(1), " and ", mat_b.size(0), "x", mat_b.size(1), ") ",
1357:         "with contraction dims mat_a: ", mat_a_dim, ", mat_b: ", mat_b_dim);
1358:   } else {
1359:     TORCH_CHECK_VALUE(
1360:         mat_a.size(1) == mat_b.size(0), "mat_a and mat_b shapes cannot be multiplied (",
1361:         mat_a.size(0), "x", mat_a.size(1), " and ", mat_b.size(0), "x", mat_b.size(1), ")");
1362:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1364-1364
```cpp
1364:   // Handle fp4 packed-K dimension
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1365-1365
```cpp
1365:   int K_multiplier = (mat_a.scalar_type() == ScalarType::Float4_e2m1fn_x2) ? 2 : 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1367-1378
```cpp
1367:   TORCH_CHECK_VALUE(!bias || bias->numel() == mat_b.sizes()[1], "Bias must be size ", mat_b.sizes()[1],
1368:        " but got ", bias->numel());
1369:   TORCH_CHECK_VALUE(
1370:       K_multiplier * mat_a.sizes()[1] % 16 == 0,
1371:       "Expected trailing dimension of mat1 to be divisible by 16 ",
1372:       "but got mat1 shape: (",
1373:       mat_a.sizes()[0],
1374:       "x",
1375:       K_multiplier * mat_a.sizes()[1],
1376:       ").");
1377:   TORCH_CHECK_VALUE(K_multiplier * mat_b.sizes()[0] % 16 == 0 && mat_b.sizes()[1] % 16 == 0, "mat2 shape (", mat_b.sizes()[0], "x",
1378:        mat_b.sizes()[1], ") must be divisible by 16");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1380-1380
```cpp
1380:   // TODO(slayton): Existing checks, not sure if they should really be here.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1381-1394
```cpp
1381:   TORCH_CHECK_VALUE(!out_dtype || *out_dtype == out.scalar_type(), "out_dtype must match output matrix type");
1382:   TORCH_CHECK_VALUE(isFloat8Type(mat_a.scalar_type()) || mat_a.scalar_type() == ScalarType::Float4_e2m1fn_x2,
1383:       "Expected mat_a to be Float8 or Float4_x2 matrix got ", mat_a.scalar_type());
1384:   TORCH_CHECK_VALUE(isFloat8Type(mat_b.scalar_type()) || mat_b.scalar_type() == ScalarType::Float4_e2m1fn_x2,
1385:       "Expected mat_b to be Float8 or Float4_x2 matrix got ", mat_b.scalar_type());
1386: #ifndef USE_ROCM
1387:   // Type restrictions imposed by CuBLASLt as of CUDA-12.1
1388:   TORCH_CHECK_VALUE(mat_a.scalar_type() != ScalarType::Float8_e5m2 || mat_b.scalar_type() != ScalarType::Float8_e5m2,
1389:         "Multiplication of two Float8_e5m2 matrices is not supported");
1390: #endif
1391:   if (use_fast_accum) {
1392:     TORCH_CHECK_VALUE(mat_a.scalar_type() != ScalarType::Float4_e2m1fn_x2 && mat_b.scalar_type() != ScalarType::Float4_e2m1fn_x2, "`use_fast_accum` is not supported when `mat_a` or `mat_b` tensors have the `Float4_e2m1fn_x2` dtype.");
1393:   }
1394: #ifdef USE_ROCM
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1395-1416
```cpp
1395:   if (mat_a.scalar_type() == ScalarType::Float4_e2m1fn_x2 || mat_b.scalar_type() == ScalarType::Float4_e2m1fn_x2) {
1396:     TORCH_CHECK_NOT_IMPLEMENTED(ROCM_VERSION >= 70000,
1397:         "Float4_e2m1fn_x2 is only supported for ROCm 7.0 and above");
1398:   }
1399:   if (mat_a.scalar_type() == ScalarType::Float8_e5m2 || mat_b.scalar_type() == ScalarType::Float8_e5m2) {
1400:     TORCH_CHECK_NOT_IMPLEMENTED(ROCM_VERSION >= 60500,
1401:         "Float8_e5m2 is only supported for ROCm 6.5 and above");
1402:   }
1403:   if (mat_a.scalar_type() == ScalarType::Float8_e4m3fn || mat_b.scalar_type() == ScalarType::Float8_e4m3fn) {
1404:     TORCH_CHECK_NOT_IMPLEMENTED(ROCM_VERSION >= 60500,
1405:         "Float8_e4m3fn is only supported for ROCm 6.5 and above");
1406:   }
1407: #endif
1408:   if (bias) {
1409:     TORCH_CHECK_VALUE(out.scalar_type() != kFloat,
1410:         "Bias is not supported when out_dtype is set to Float32");
1411: 
1412:     TORCH_CHECK_VALUE(bias->scalar_type() == ScalarType::BFloat16 ||
1413:                 bias->scalar_type() == ScalarType::Half,
1414:         "Bias must be BFloat16 or Half, but got ", bias->scalar_type());
1415: 
1416:     TORCH_CHECK_VALUE((out.scalar_type() != kFloat &&
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1417-1420
```cpp
1417:                  out.scalar_type() != ScalarType::BFloat16) ||
1418:                 bias->scalar_type() == ScalarType::BFloat16,
1419:         "Bias must be BFloat16 to compute ", out.scalar_type(),
1420:         " output, but got ", bias->scalar_type());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1422-1428
```cpp
1422:     TORCH_CHECK_VALUE(out.scalar_type() != ScalarType::Half ||
1423:                 bias->scalar_type() == ScalarType::Half,
1424:         "Bias must be Float16 to compute ", out.scalar_type(),
1425:         " output, but got ", bias->scalar_type());
1426:   }
1427:   {
1428:     auto bias_ = bias.value_or(Tensor());
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1430-1430
```cpp
1430:     // NOLINTNEXTLINE(*c-array*)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1431-1434
```cpp
1431:     TensorArg targs[]{{out, "out", 0}, {mat_a, "mat_a", 1}, {mat_b, "mat_b", 2},
1432:                       {bias_, "bias", 3}, {scale_a[0], "scale_a", 4}, {scale_b[0], "scale_b", 5}};
1433:     checkAllSameGPU(__func__, targs);
1434:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1436-1436
```cpp
1436:   auto out_dtype_ = out_dtype.value_or(at::ScalarType::BFloat16);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1438-1438
```cpp
1438:   // Conversion of implicitly-defined enums to explicit
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1439-1442
```cpp
1439:   auto scale_recipe_a_enum = convert_int_to_enum<ScalingType>(scale_recipe_a);
1440:   auto swizzle_a_enum = convert_int_to_enum<SwizzleType>(swizzle_a);
1441:   auto scale_recipe_b_enum = convert_int_to_enum<ScalingType>(scale_recipe_b);
1442:   auto swizzle_b_enum = convert_int_to_enum<SwizzleType>(swizzle_b);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1444-1447
```cpp
1444:   // at this point we can start working out what we want to be doing
1445:   // Try to do as few steps as possible.
1446:   // NOTE: support is deliberately sparse, can explicitly enumerate all combinations allowed.
1447:   // Do this via a list of defined (name, acceptance, concrete_impl) tuples.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1448-1449
```cpp
1448:   bool found_impl = false;
1449:   ScaledGemmImplementation gemm_impl = ScaledGemmImplementation::NONE;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1451-1464
```cpp
1451:   for (const auto& fn_entry : scale_kernel_dispatch) {
1452:     const auto [name, accept_fn, scaled_gemm_impl] = fn_entry;
1453:     bool ok = accept_fn(mat_a.scalar_type(),
1454:                         scale_recipe_a_enum,
1455:                         scale_a,
1456:                         mat_b.scalar_type(),
1457:                         scale_recipe_b_enum,
1458:                         scale_b);
1459:     if (ok) {
1460:       gemm_impl = scaled_gemm_impl;
1461:       found_impl = true;
1462:       break;
1463:     }
1464:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1465-1476
```cpp
1465:   TORCH_CHECK_VALUE(
1466:     found_impl,
1467:     "Invalid scaling configuration.\n"
1468:     "- For TensorWise scaling, a and b should be float8, scales should be float and singletons.\n"
1469:     "- For RowWise scaling, a and b should be float8, scales should be float, scale_a should be (", mat_a.size(0), ", 1) and scale_b should be (1, ", mat_b.size(1), "), and both should be contiguous.\n"
1470:     "- For BlockWise 1x128 scaling, a and b should be float8, scales should be float, scale_a should be (", mat_a.size(0), ", ", ceil_div<int64_t>(mat_a.size(1), 128), ") and scale_b should be (", ceil_div<int64_t>(mat_b.size(0), 128), ", ", mat_b.size(1), "), and both should be outer-dim-major.\n"
1471:     "- For BlockWise 128x128 scaling, a and b should be float8, scales should be float, scale_a should be (", ceil_div<int64_t>(mat_a.size(0), 128), ", ", ceil_div<int64_t>(mat_a.size(1), 128), ") and scale_b should be (", ceil_div<int64_t>(mat_b.size(0), 128), ", ", ceil_div<int64_t>(mat_b.size(1), 128), "), and both should be near-inner-dim-major (with 16-byte aligned strides).\n"
1472:     "- For Blockwise 1x32 scaling, a and b should be float8, scales should be float8_e8m0fnu, scale_a should have ", round_up<int64_t>(mat_a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_a.size(1), 32), 4), " elements and scale_b should have ", round_up<int64_t>(mat_b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_b.size(0), 32), 4), " elements, and both should be contiguous.\n"
1473:     "- For Blockwise 1x16 scaling, a and b should be float4 (packed 2x), scales should be float8_e4m3fn, scale_a should have ", round_up<int64_t>(mat_a.size(0), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_a.size(1) * 2, 16), 4), " elements and scale_b should have ", round_up<int64_t>(mat_b.size(1), 128) * round_up<int64_t>(ceil_div<int64_t>(mat_b.size(0) * 2, 16), 4), " elements, and both should be contiguous.\n"
1474:     "Got mat_a.dtype()=", mat_a.scalar_type(), ", scale_a[0].dtype()=", scale_a[0].scalar_type(), ", scale_a[0].size()=", scale_a[0].sizes(), ", scale_a[0].stride()=", scale_a[0].strides(), ", ",
1475:     "mat_b.dtype()=", mat_b.scalar_type(), ", scale_b[0].dtype()=", scale_b[0].scalar_type(), ", scale_b[0].size()=", scale_b[0].sizes(), " and scale_b[0].stride()=", scale_b[0].strides()
1476:   );
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1478-1478
```cpp
1478:   at::native::resize_output(out, {mat_a.size(0), mat_b.size(1)});
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1480-1480
```cpp
1480:   auto bias_ = bias.value_or(Tensor());
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1482-1482
```cpp
1482:   check_swizzle_lengths(gemm_impl, swizzle_a_enum, swizzle_b_enum);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1484-1484
```cpp
1484:   // dispatch to appropriate lower-level calls for error checking & execution
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 1485-1506
```cpp
1485:   if (gemm_impl == ScaledGemmImplementation::TENSORWISE_TENSORWISE) {
1486:     return _scaled_tensorwise_tensorwise(mat_a, mat_b, scale_a[0], scale_b[0], bias, out_dtype_, use_fast_accum, out);
1487:   } else if (gemm_impl == ScaledGemmImplementation::ROWWISE_ROWWISE) {
1488:     return _scaled_rowwise_rowwise(mat_a, mat_b, scale_a[0], scale_b[0], bias, out_dtype_, use_fast_accum, out);
1489:   } else if (gemm_impl == ScaledGemmImplementation::BLOCK_128x128_1x128) {
1490:     return _scaled_block128x128_block1x128(mat_a, mat_b, scale_a[0], scale_b[0], bias, out_dtype_, use_fast_accum, out);
1491:   } else if (gemm_impl == ScaledGemmImplementation::BLOCK_1x128_128x128) {
1492:     return _scaled_block1x128_block128x128(mat_a, mat_b, scale_a[0], scale_b[0], bias, out_dtype_, use_fast_accum, out);
1493:   } else if (gemm_impl == ScaledGemmImplementation::BLOCK_1x128_1x128) {
1494:     return _scaled_block1x128_block1x128(mat_a, mat_b, scale_a[0], scale_b[0], bias, out_dtype_, use_fast_accum, out);
1495:   } else if (gemm_impl == ScaledGemmImplementation::MXFP8_MXFP8) {
1496:     return _scaled_mxfp8_mxfp8(mat_a, mat_b, scale_a[0], swizzle_a_enum[0], scale_b[0], swizzle_b_enum[0], bias, out_dtype_, out);
1497:   } else if (gemm_impl == ScaledGemmImplementation::NVFP4_NVFP4) {
1498:     return _scaled_nvfp4_nvfp4(mat_a, mat_b, scale_a[0], swizzle_a_enum[0], scale_b[0], swizzle_b_enum[0], bias, out_dtype_, out,
1499:                                scale_a[1], scale_b[1]);
1500:   } else if (gemm_impl == ScaledGemmImplementation::NVFP4_NVFP4_SINGLE_SCALE) {
1501:     return _scaled_nvfp4_nvfp4(mat_a, mat_b, scale_a[0], swizzle_a_enum[0], scale_b[0], swizzle_b_enum[0], bias, out_dtype_, out);
1502:   } else if (gemm_impl == ScaledGemmImplementation::MXFP4_MXFP4) {
1503:     return _scaled_mxfp4_mxfp4(mat_a, mat_b, scale_a[0], swizzle_a_enum[0], scale_b[0], swizzle_b_enum[0], bias, out_dtype_, out);
1504:   } else {
1505:     TORCH_CHECK_VALUE(false, "Invalid state - found an implementation, but not really");
1506:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1507-1507
```cpp
1507: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1509-1530
```cpp
1509: Tensor
1510: _scaled_mm_cuda_v2(
1511:           const Tensor& mat_a, const Tensor& mat_b,
1512:           ArrayRef<Tensor> scale_a,
1513:           IntArrayRef scale_recipe_a,
1514:           IntArrayRef swizzle_a,
1515:           ArrayRef<Tensor> scale_b,
1516:           IntArrayRef scale_recipe_b,
1517:           IntArrayRef swizzle_b,
1518:           const std::optional<Tensor>& bias,
1519:           const std::optional<c10::ScalarType> out_dtype,
1520:           IntArrayRef contraction_dim,
1521:           bool use_fast_accum) {
1522:   const auto out_dtype_ = out_dtype.value_or(mat_a.scalar_type());
1523:   Tensor out = at::empty({0}, mat_a.options().dtype(out_dtype_));
1524: 
1525:   return _scaled_mm_cuda_v2_out(
1526:                       mat_a, mat_b,
1527:                       scale_a, scale_recipe_a, swizzle_a,
1528:                       scale_b, scale_recipe_b, swizzle_b,
1529:                       bias,
1530:                       out_dtype,
```
- EN: This block defines or continues the implementation of `_scaled_mm_cuda_v2`.
- CN: 该代码块定义或继续实现 `_scaled_mm_cuda_v2`。

### Lines 1531-1534
```cpp
1531:                       contraction_dim,
1532:                       use_fast_accum,
1533:                       out);
1534: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1536-1536
```cpp
1536: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。
- cuBLAS is used for CUDA linear algebra primitives. / 这里使用 cuBLAS 提供 CUDA 线性代数原语。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<cstdint>`
  - `<c10/util/typeid.h>`
  - `<c10/util/Exception.h>`
  - `<c10/util/SmallVector.h>`
  - `<c10/core/Scalar.h>`
  - `<c10/core/ScalarType.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/core/NamedTensor.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/ExpandUtils.h>`
  - `<ATen/OpMathType.h>`
  - `<ATen/TensorUtils.h>`
- Runtime symbols / 运行时符号:
  - `AT_DISPATCH_V2`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::tunable::ScaledGemmTunableOp`
  - `at::cuda::tunable::ScaledGemmParams`
  - `at::cuda::tunable::BlasOp::T`
  - `at::cuda::tunable::BlasOp::N`
  - `at::cuda::blas::scaled_gemm`
  - `at::cuda::tunable::getTuningContext`
  - `at::cuda::detail::f8f8bf16_rowwise`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
