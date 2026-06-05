# RowwiseScaledMM.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/RowwiseScaledMM.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ceildiv`, `round_up_to_nearest_multiple`, `f8f8bf16_rowwise_impl`, `f8f8bf16_rowwise_impl_sm100_sm120`.
- 用途（中文）: 实现与 `ceildiv`, `round_up_to_nearest_multiple`, `f8f8bf16_rowwise_impl`, `f8f8bf16_rowwise_impl_sm100_sm120` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/core/Tensor.h>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
   6: #include <c10/macros/Macros.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`。

### Lines 8-8
```cpp
   8: // Two warnings in Cutlass included header files
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 9-12
```cpp
   9: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wset-but-not-used")
  10: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-parameter")
  11: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wmissing-field-initializers")
  12: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-variable")
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 14-16
```cpp
  14: // Determine if the architecture supports rowwise scaled mm
  15: // Currently failing on windows with:
  16: // https://github.com/NVIDIA/cutlass/issues/1571
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 17-30
```cpp
  17: #if !defined(USE_ROCM) && !defined(_WIN32) && defined(CUDA_VERSION)
  18: 
  19: #define BUILD_ROWWISE_FP8_KERNEL
  20: #endif
  21: 
  22: #if defined(BUILD_ROWWISE_FP8_KERNEL)
  23: 
  24: #include <cute/tensor.hpp>
  25: #include <cutlass/core_io.h>
  26: #include <cutlass/cutlass.h>
  27: #include <cutlass/gemm/device/gemm.h>
  28: #include <cutlass/half.h>
  29: #include <cutlass/numeric_types.h>
  30: #include <cutlass/trace.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cute/tensor.hpp>`, `<cutlass/core_io.h>`, `<cutlass/cutlass.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cute/tensor.hpp>`, `<cutlass/core_io.h>`, `<cutlass/cutlass.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 31-44
```cpp
  31: #include <cutlass/util/host_tensor.h>
  32: #include <cutlass/version.h>
  33: 
  34: #include <cutlass/gemm/collective/collective_builder.hpp>
  35: #include <cutlass/gemm/device/gemm_universal.h>
  36: #include <cutlass/gemm/device/gemm_universal_adapter.h>
  37: #include <cutlass/gemm/kernel/default_gemm_universal_with_visitor.h>
  38: #include <cutlass/epilogue/collective/collective_builder.hpp>
  39: #include <cutlass/epilogue/threadblock/fusion/visitors.hpp>
  40: 
  41: #include <cute/atom/mma_atom.hpp>
  42: #include <cutlass/gemm/dispatch_policy.hpp>
  43: #include <cutlass/gemm/kernel/gemm_universal.hpp>
  44: #include <cutlass/util/packed_stride.hpp>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cutlass/util/host_tensor.h>`, `<cutlass/version.h>`, `<cutlass/gemm/collective/collective_builder.hpp>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cutlass/util/host_tensor.h>`, `<cutlass/version.h>`, `<cutlass/gemm/collective/collective_builder.hpp>`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 45-46
```cpp
  45: 
  46: #include <ATen/native/cuda/cutlass_common.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/cutlass_common.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/cutlass_common.cuh>`。

### Lines 48-50
```cpp
  48: C10_DIAGNOSTIC_POP()
  49: C10_DIAGNOSTIC_POP()
  50: C10_DIAGNOSTIC_POP()
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 52-73
```cpp
  52: namespace {
  53: 
  54: using DtypeScale = float;
  55: using DtypeAccum = float;
  56: using DtypeEpilogue = float;
  57: 
  58: using Multiply = cutlass::epilogue::fusion::Sm90Compute<
  59:     cutlass::multiplies,
  60:     DtypeEpilogue,
  61:     DtypeEpilogue,
  62:     cutlass::FloatRoundStyle::round_to_nearest>;
  63: 
  64: using Add = cutlass::epilogue::fusion::Sm90Compute<
  65:     cutlass::plus,
  66:     DtypeEpilogue,
  67:     DtypeEpilogue,
  68:     cutlass::FloatRoundStyle::round_to_nearest>;
  69: 
  70: template <bool LargeTile, bool FastAccum>
  71: struct Schedule;
  72: 
  73: template <>
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 74-77
```cpp
  74: struct Schedule</*LargeTile=*/false, /*FastAccum=*/false> {
  75:   using type = cutlass::gemm::KernelTmaWarpSpecialized;
  76:   using epilogue_type = cutlass::epilogue::TmaWarpSpecialized;
  77: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 79-86
```cpp
  79: template <>
  80: struct Schedule</*LargeTile=*/true, /*FastAccum=*/false> {
  81:   // For a 128x128x128 tile with fastAccum = false, using
  82:   // pingpong schedule will lead to spilling, and WarpSpecialized w/o pingpong
  83:   // is slow
  84:   using type = cutlass::gemm::KernelTmaWarpSpecializedCooperative;
  85:   using epilogue_type = cutlass::epilogue::TmaWarpSpecializedCooperative;
  86: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 88-92
```cpp
  88: template <>
  89: struct Schedule</*LargeTile=*/false, /*FastAccum=*/true> {
  90:   using type = cutlass::gemm::KernelTmaWarpSpecializedFP8FastAccum;
  91:   using epilogue_type = cutlass::epilogue::TmaWarpSpecialized;
  92: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 94-98
```cpp
  94: template <>
  95: struct Schedule</*LargeTile=*/true, /*FastAccum=*/true> {
  96:   using type = cutlass::gemm::KernelTmaWarpSpecializedPingpongFP8FastAccum;
  97:   using epilogue_type = cutlass::epilogue::TmaWarpSpecialized;
  98: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 100-102
```cpp
 100: int ceildiv(int a, int b) {
 101:   return (a + b - 1) / b;
 102: }
```
- EN: This block defines or continues the implementation of `ceildiv`.
- CN: 该代码块定义或继续实现 `ceildiv`。

### Lines 104-106
```cpp
 104: int round_up_to_nearest_multiple(int a, int b) {
 105:   return ceildiv(a, b) * b;
 106: }
```
- EN: This block defines or continues the implementation of `round_up_to_nearest_multiple`.
- CN: 该代码块定义或继续实现 `round_up_to_nearest_multiple`。

### Lines 108-108
```cpp
 108: // Cutlass rowwise kernel for sm90
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 109-122
```cpp
 109: template <
 110:     typename TileShape,
 111:     typename ClusterShape,
 112:     typename Transposed,
 113:     typename FastAccum,
 114:     typename DtypeA,
 115:     typename DtypeB,
 116:     typename DtypeBias,
 117:     typename DtypeOutput>
 118: void f8f8bf16_rowwise_impl(
 119:     at::Tensor XQ, // FP8
 120:     at::Tensor WQ, // FP8
 121:     at::Tensor x_scale,
 122:     at::Tensor w_scale,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 123-143
```cpp
 123:     std::optional<at::Tensor> bias,
 124:     at::Tensor out,
 125:     const int swizzle) {
 126:   int M = XQ.size(0);
 127:   int N = WQ.size(1);
 128:   int K = XQ.size(1);
 129: 
 130:   // Workaround for https://github.com/pytorch/pytorch/issues/133334.
 131:   if (M % 256 > 0) {
 132:     int padded_M = ((M - 1) / 256 + 1) * 256;
 133:     at::Tensor padded_x_scale = x_scale.new_empty({padded_M, 1});
 134:     padded_x_scale.slice(/*dim=*/0, /*start=*/0, /*end=*/M)
 135:         .copy_(std::move(x_scale));
 136:     x_scale = std::move(padded_x_scale);
 137:   }
 138: 
 139:   using LayoutInputA = cutlass::layout::RowMajor;
 140:   constexpr int AlignmentInputA = 16 / sizeof(DtypeA);
 141: 
 142:   using LayoutInputB = cutlass::layout::ColumnMajor;
 143:   constexpr int AlignmentInputB = 16 / sizeof(DtypeB);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 145-149
```cpp
 145:   using LayoutOutput = std::conditional_t<
 146:       Transposed::value,
 147:       cutlass::layout::ColumnMajor,
 148:       cutlass::layout::RowMajor>;
 149:   constexpr int AlignmentOutput = 16 / sizeof(DtypeOutput);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 151-151
```cpp
 151:   // Tag indicating the minimum SM that supports the intended feature
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 152-153
```cpp
 152:   using ArchTag = cutlass::arch::Sm90;
 153:   using OperatorClass = cutlass::arch::OpClassTensorOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 155-155
```cpp
 155:   // Implement rowwise scaling epilogue.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 156-157
```cpp
 156:   constexpr int ColBroadcastStages = 0;
 157:   constexpr int RowBroadcastStages = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 159-160
```cpp
 159:   using XScale = cutlass::epilogue::fusion::
 160:       Sm90ColBroadcast<ColBroadcastStages, TileShape, DtypeScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 162-163
```cpp
 162:   using WScale = cutlass::epilogue::fusion::
 163:       Sm90RowBroadcast<RowBroadcastStages, TileShape, DtypeScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 165-170
```cpp
 165:   using Bias = std::conditional_t<
 166:       Transposed::value,
 167:       cutlass::epilogue::fusion::
 168:           Sm90ColBroadcast<ColBroadcastStages, TileShape, DtypeBias>,
 169:       cutlass::epilogue::fusion::
 170:           Sm90RowBroadcast<RowBroadcastStages, TileShape, DtypeBias>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 172-176
```cpp
 172:   using Accum = cutlass::epilogue::fusion::Sm90AccFetch;
 173:   using AccumScale = cutlass::epilogue::fusion::Sm90EVT<
 174:       Multiply,
 175:       WScale,
 176:       cutlass::epilogue::fusion::Sm90EVT<Multiply, XScale, Accum>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-188
```cpp
 178:   using Cast = cutlass::epilogue::fusion::Sm90Compute<
 179:       cutlass::epilogue::thread::Identity,
 180:       DtypeOutput,
 181:       DtypeEpilogue,
 182:       cutlass::FloatRoundStyle::round_to_nearest>;
 183:   using EpilogueEVT = cutlass::epilogue::fusion::Sm90EVT<
 184:       Cast,
 185:       cutlass::epilogue::fusion::Sm90EVT<
 186:           Add,
 187:           Bias,
 188:           AccumScale>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 190-190
```cpp
 190:   constexpr bool large_tile = std::is_same_v<TileShape, cute::Shape<cute::_128, cute::_128, cute::_128>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 192-205
```cpp
 192:   using CollectiveEpilogue =
 193:       typename cutlass::epilogue::collective::CollectiveBuilder<
 194:           ArchTag,
 195:           OperatorClass,
 196:           TileShape,
 197:           ClusterShape,
 198:           cutlass::epilogue::collective::EpilogueTileAuto,
 199:           DtypeAccum,
 200:           DtypeEpilogue,
 201:           void, // Indicate there is no beta scaling to save register
 202:           LayoutOutput,
 203:           AlignmentOutput,
 204:           DtypeOutput,
 205:           LayoutOutput,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 206-208
```cpp
 206:           AlignmentOutput,
 207:           typename Schedule<large_tile, FastAccum::value>::epilogue_type,
 208:           EpilogueEVT>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 210-223
```cpp
 210:   using CollectiveMainloop =
 211:       typename cutlass::gemm::collective::CollectiveBuilder<
 212:           ArchTag,
 213:           OperatorClass,
 214:           DtypeA,
 215:           LayoutInputA,
 216:           AlignmentInputA,
 217:           DtypeB,
 218:           LayoutInputB,
 219:           AlignmentInputB,
 220:           DtypeAccum,
 221:           TileShape,
 222:           ClusterShape,
 223:           cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 224-226
```cpp
 224:               sizeof(typename CollectiveEpilogue::SharedStorage))>,
 225:           typename Schedule<large_tile, FastAccum::value>::type>::
 226:           CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 228-232
```cpp
 228:   using GemmKernel = at::cuda::detail::enable_3x_kernel_for_sm9x<
 229:       cutlass::gemm::kernel::GemmUniversal<
 230:           cute::Shape<int, int, int>,
 231:           CollectiveMainloop,
 232:           CollectiveEpilogue>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 234-234
```cpp
 234:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 236-238
```cpp
 236:   using StrideInputA = typename Gemm::GemmKernel::StrideA;
 237:   using StrideInputB = typename Gemm::GemmKernel::StrideB;
 238:   using StrideOutput = typename Gemm::GemmKernel::StrideC;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 240-245
```cpp
 240:   StrideInputA stride_a = cutlass::make_cute_packed_stride(
 241:       StrideInputA{}, cute::make_shape(M, static_cast<int>(XQ.stride(0)), 1));
 242:   StrideInputB stride_b = cutlass::make_cute_packed_stride(
 243:       StrideInputB{}, cute::make_shape(N, static_cast<int>(WQ.stride(1)), 1));
 244:   StrideOutput stride_output = cutlass::make_cute_packed_stride(
 245:       StrideOutput{}, cute::make_shape(M, static_cast<int>(out.stride(0)), 1));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 247-261
```cpp
 247:   typename Gemm::Arguments arguments{
 248:       cutlass::gemm::GemmUniversalMode::kGemm,
 249:       {M, N, K},
 250:       {reinterpret_cast<DtypeA*>(XQ.data_ptr()),
 251:        stride_a,
 252:        reinterpret_cast<DtypeB*>(WQ.data_ptr()),
 253:        stride_b},
 254:       {{{{bias.has_value() ? reinterpret_cast<DtypeBias*>(bias->data_ptr())
 255:                            : nullptr},
 256:          {{reinterpret_cast<DtypeScale*>(w_scale.data_ptr())},
 257:           {{reinterpret_cast<DtypeScale*>(x_scale.data_ptr())}}}}},
 258:        nullptr,
 259:        stride_output,
 260:        reinterpret_cast<DtypeOutput*>(out.data_ptr()),
 261:        stride_output}};
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 263-263
```cpp
 263:   Gemm gemm;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 265-266
```cpp
 265:   // Using the arguments, query for extra workspace required for matrix
 266:   // multiplication computation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 267-267
```cpp
 267:   size_t workspace_size = Gemm::get_workspace_size(arguments);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 269-269
```cpp
 269:   // Ensure persistent kernels leave enough free SMs for NCCL background ops.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 270-274
```cpp
 270:   if (at::globalContext()._SMCarveout_EXPERIMENTAL().has_value()) {
 271:     arguments.hw_info.sm_count =
 272:         at::cuda::getDeviceProperties(out.device().index())->multiProcessorCount -
 273:         at::globalContext()._SMCarveout_EXPERIMENTAL().value();
 274:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 276-276
```cpp
 276:   // Set the swizzle size
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 277-277
```cpp
 277:   arguments.scheduler.max_swizzle_size = swizzle;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 279-279
```cpp
 279:   // Allocate workspace memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 280-282
```cpp
 280:   auto workspace = XQ.new_empty(
 281:       {static_cast<int64_t>(workspace_size)},
 282:       at::TensorOptions().dtype(at::kByte));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 284-284
```cpp
 284:   // Check the problem size is supported or not
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 285-288
```cpp
 285:   cutlass::Status status = gemm.can_implement(arguments);
 286:   if (status != cutlass::Status::kSuccess) {
 287:     throw std::runtime_error("cutlass cannot implement");
 288:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 290-290
```cpp
 290:   // Initialize CUTLASS kernel with arguments and workspace pointer
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 291-294
```cpp
 291:   status = gemm.initialize(arguments, workspace.data_ptr(), at::cuda::getCurrentCUDAStream());
 292:   if (status != cutlass::Status::kSuccess) {
 293:     throw std::runtime_error("cutlass cannot initialize");
 294:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 296-303
```cpp
 296:   status = gemm(at::cuda::getCurrentCUDAStream());
 297:   if (status != cutlass::Status::kSuccess) {
 298:     throw std::runtime_error(
 299:         std::string("cutlass cannot run") +
 300:         cutlass::cutlassGetStatusString(status));
 301:   }
 302:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 303: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 306-306
```cpp
 306: // Cutlass rowwise kernel for SM100/SM120
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 307-320
```cpp
 307: template <
 308:     typename ArchTag,
 309:     typename TileShape,
 310:     typename ClusterShape,
 311:     typename Transposed,
 312:     typename FastAccum,
 313:     typename DtypeA,
 314:     typename DtypeB,
 315:     typename DtypeBias,
 316:     typename DtypeOutput>
 317: void f8f8bf16_rowwise_impl_sm100_sm120(
 318:     at::Tensor XQ, // FP8
 319:     at::Tensor WQ, // FP8
 320:     at::Tensor x_scale,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 321-342
```cpp
 321:     at::Tensor w_scale,
 322:     std::optional<at::Tensor> bias,
 323:     at::Tensor out,
 324:     const int swizzle) {
 325:   int M = XQ.size(0);
 326:   int N = WQ.size(1);
 327:   int K = XQ.size(1);
 328: 
 329:   // Workaround for https://github.com/pytorch/pytorch/issues/133334.
 330:   if (M % 256 > 0) {
 331:     int padded_M = ((M - 1) / 256 + 1) * 256;
 332:     at::Tensor padded_x_scale = x_scale.new_empty({padded_M, 1});
 333:     padded_x_scale.slice(/*dim=*/0, /*start=*/0, /*end=*/M)
 334:         .copy_(std::move(x_scale));
 335:     x_scale = std::move(padded_x_scale);
 336:   }
 337: 
 338:   using LayoutInputA = cutlass::layout::RowMajor;
 339:   constexpr int AlignmentInputA = 16 / sizeof(DtypeA);
 340: 
 341:   using LayoutInputB = cutlass::layout::ColumnMajor;
 342:   constexpr int AlignmentInputB = 16 / sizeof(DtypeB);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 344-348
```cpp
 344:   using LayoutOutput = std::conditional_t<
 345:       Transposed::value,
 346:       cutlass::layout::ColumnMajor,
 347:       cutlass::layout::RowMajor>;
 348:   constexpr int AlignmentOutput = 16 / sizeof(DtypeOutput);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 350-350
```cpp
 350:   using OperatorClass = cutlass::arch::OpClassTensorOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 352-352
```cpp
 352:   // Implement rowwise scaling epilogue.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 353-354
```cpp
 353:   constexpr int ColBroadcastStages = 0;
 354:   constexpr int RowBroadcastStages = 0;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 356-357
```cpp
 356:   using XScale = cutlass::epilogue::fusion::
 357:       Sm90ColBroadcast<ColBroadcastStages, TileShape, DtypeScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 359-360
```cpp
 359:   using WScale = cutlass::epilogue::fusion::
 360:       Sm90RowBroadcast<RowBroadcastStages, TileShape, DtypeScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 362-367
```cpp
 362:   using Bias = std::conditional_t<
 363:       Transposed::value,
 364:       cutlass::epilogue::fusion::
 365:           Sm90ColBroadcast<ColBroadcastStages, TileShape, DtypeBias>,
 366:       cutlass::epilogue::fusion::
 367:           Sm90RowBroadcast<RowBroadcastStages, TileShape, DtypeBias>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 369-373
```cpp
 369:   using Accum = cutlass::epilogue::fusion::Sm90AccFetch;
 370:   using AccumScale = cutlass::epilogue::fusion::Sm90EVT<
 371:       Multiply,
 372:       WScale,
 373:       cutlass::epilogue::fusion::Sm90EVT<Multiply, XScale, Accum>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 375-385
```cpp
 375:   using Cast = cutlass::epilogue::fusion::Sm90Compute<
 376:       cutlass::epilogue::thread::Identity,
 377:       DtypeOutput,
 378:       DtypeEpilogue,
 379:       cutlass::FloatRoundStyle::round_to_nearest>;
 380:   using EpilogueEVT = cutlass::epilogue::fusion::Sm90EVT<
 381:       Cast,
 382:       cutlass::epilogue::fusion::Sm90EVT<
 383:           Add,
 384:           Bias,
 385:           AccumScale>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 387-396
```cpp
 387:   using EpilogueScheduleType = cutlass::epilogue::collective::EpilogueScheduleAuto;
 388:   using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
 389:       ArchTag, OperatorClass,
 390:       TileShape, ClusterShape,
 391:       cutlass::epilogue::collective::EpilogueTileAuto,
 392:       DtypeAccum, DtypeEpilogue,
 393:       void, LayoutOutput, AlignmentOutput,
 394:       DtypeOutput, LayoutOutput, AlignmentOutput,
 395:       EpilogueScheduleType,
 396:       EpilogueEVT>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 398-400
```cpp
 398:   // as of CUTLASS 3.9.2, on sm120, KernelScheduleAuto resolves to
 399:   // KernelTmaWarpSpecializedCooperativeSm120<2>>,
 400:   // which does not support TileShape.M < 128
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 401-414
```cpp
 401:   using MainloopScheduleType = std::conditional_t<
 402:       std::is_same_v<ArchTag, cutlass::arch::Sm120> && cute::size<0>(TileShape{}) < 128,
 403:       cutlass::gemm::KernelTmaWarpSpecializedPingpong,
 404:       cutlass::gemm::collective::KernelScheduleAuto>;
 405:   using CollectiveMainloop =
 406:       typename cutlass::gemm::collective::CollectiveBuilder<
 407:           ArchTag,
 408:           OperatorClass,
 409:           DtypeA,
 410:           LayoutInputA,
 411:           AlignmentInputA,
 412:           DtypeB,
 413:           LayoutInputB,
 414:           AlignmentInputB,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 415-419
```cpp
 415:           DtypeAccum,
 416:           TileShape,
 417:           ClusterShape,
 418:           cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>,
 419:           MainloopScheduleType>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 421-425
```cpp
 421:   using GemmKernel = at::cuda::detail::enable_3x_kernel_for_sm10_or_later<
 422:       cutlass::gemm::kernel::GemmUniversal<
 423:           cute::Shape<int, int, int>,
 424:           CollectiveMainloop,
 425:           CollectiveEpilogue>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 427-427
```cpp
 427:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 429-431
```cpp
 429:   using StrideInputA = typename Gemm::GemmKernel::StrideA;
 430:   using StrideInputB = typename Gemm::GemmKernel::StrideB;
 431:   using StrideOutput = typename Gemm::GemmKernel::StrideC;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 433-438
```cpp
 433:   StrideInputA stride_a = cutlass::make_cute_packed_stride(
 434:       StrideInputA{}, cute::make_shape(M, static_cast<int>(XQ.stride(0)), 1));
 435:   StrideInputB stride_b = cutlass::make_cute_packed_stride(
 436:       StrideInputB{}, cute::make_shape(N, static_cast<int>(WQ.stride(1)), 1));
 437:   StrideOutput stride_output = cutlass::make_cute_packed_stride(
 438:       StrideOutput{}, cute::make_shape(M, static_cast<int>(out.stride(0)), 1));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 440-454
```cpp
 440:   typename Gemm::Arguments arguments{
 441:       cutlass::gemm::GemmUniversalMode::kGemm,
 442:       {M, N, K},
 443:       {reinterpret_cast<DtypeA*>(XQ.data_ptr()),
 444:        stride_a,
 445:        reinterpret_cast<DtypeB*>(WQ.data_ptr()),
 446:        stride_b},
 447:       {{{{bias.has_value() ? reinterpret_cast<DtypeBias*>(bias->data_ptr())
 448:                            : nullptr},
 449:          {{reinterpret_cast<DtypeScale*>(w_scale.data_ptr())},
 450:           {{reinterpret_cast<DtypeScale*>(x_scale.data_ptr())}}}}},
 451:        nullptr,
 452:        stride_output,
 453:        reinterpret_cast<DtypeOutput*>(out.data_ptr()),
 454:        stride_output}};
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 456-456
```cpp
 456:   Gemm gemm;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 458-459
```cpp
 458:   // Using the arguments, query for extra workspace required for matrix
 459:   // multiplication computation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 460-460
```cpp
 460:   size_t workspace_size = Gemm::get_workspace_size(arguments);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 462-462
```cpp
 462:   // Ensure persistent kernels leave enough free SMs for NCCL background ops.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 463-467
```cpp
 463:   if (at::globalContext()._SMCarveout_EXPERIMENTAL().has_value()) {
 464:     arguments.hw_info.sm_count =
 465:         at::cuda::getDeviceProperties(out.device().index())->multiProcessorCount -
 466:         at::globalContext()._SMCarveout_EXPERIMENTAL().value();
 467:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 469-469
```cpp
 469:   // Set the swizzle size
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 470-470
```cpp
 470:   arguments.scheduler.max_swizzle_size = swizzle;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 472-472
```cpp
 472:   // Allocate workspace memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 473-475
```cpp
 473:   auto workspace = XQ.new_empty(
 474:       {static_cast<int64_t>(workspace_size)},
 475:       at::TensorOptions().dtype(at::kByte));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 477-477
```cpp
 477:   // Check the problem size is supported or not
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 478-481
```cpp
 478:   cutlass::Status status = gemm.can_implement(arguments);
 479:   if (status != cutlass::Status::kSuccess) {
 480:     throw std::runtime_error("cutlass cannot implement");
 481:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 483-483
```cpp
 483:   // Initialize CUTLASS kernel with arguments and workspace pointer
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 484-487
```cpp
 484:   status = gemm.initialize(arguments, workspace.data_ptr(), at::cuda::getCurrentCUDAStream());
 485:   if (status != cutlass::Status::kSuccess) {
 486:     throw std::runtime_error("cutlass cannot initialize");
 487:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 489-496
```cpp
 489:   status = gemm(at::cuda::getCurrentCUDAStream());
 490:   if (status != cutlass::Status::kSuccess) {
 491:     throw std::runtime_error(
 492:         std::string("cutlass cannot run") +
 493:         cutlass::cutlassGetStatusString(status));
 494:   }
 495:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 496: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 498-498
```cpp
 498: // Cutlass rowwise kernel for SM89
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 499-512
```cpp
 499: template <
 500:     typename ThreadblockShape,
 501:     typename WarpShape,
 502:     int NumStages,
 503:     typename FastAccum,
 504:     typename DtypeA,
 505:     typename DtypeB,
 506:     typename DtypeBias,
 507:     typename DtypeOutput>
 508: void f8f8bf16_rowwise_impl_sm89(
 509:     at::Tensor XQ, // FP8
 510:     at::Tensor WQ, // FP8
 511:     at::Tensor x_scale,
 512:     at::Tensor w_scale,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 513-533
```cpp
 513:     std::optional<at::Tensor> bias,
 514:     at::Tensor out) {
 515:   int M = XQ.size(0);
 516:   int N = WQ.size(1);
 517:   int K = XQ.size(1);
 518: 
 519:   using LayoutInputA = cutlass::layout::RowMajor;
 520:   constexpr int AlignmentInputA = 16 / sizeof(DtypeA);
 521: 
 522:   using LayoutInputB = cutlass::layout::ColumnMajor;
 523:   constexpr int AlignmentInputB = 16 / sizeof(DtypeB);
 524: 
 525:   using LayoutOutput = cutlass::layout::RowMajor;
 526:   constexpr int AlignmentOutput = 16 / sizeof(DtypeOutput);
 527: 
 528:   // Tag indicating the minimum SM that supports the intended feature
 529:   using ArchTag = cutlass::arch::Sm89;
 530:   using OperatorClass = cutlass::arch::OpClassTensorOp;
 531: 
 532:   using ThreadblockSwizzle =
 533:       cutlass::gemm::threadblock::ThreadblockSwizzleStreamK;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 535-535
```cpp
 535:   using InstructionShape = cutlass::gemm::GemmShape<16, 8, 32>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 537-541
```cpp
 537:   using Operator = std::conditional_t<
 538:       FastAccum::value,
 539:       cutlass::arch::OpMultiplyAddFastAccum,
 540:       cutlass::arch::OpMultiplyAdd>;
 541:   constexpr auto NumEVTEpilogueStages = 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 543-549
```cpp
 543:   using OutputTileThreadMap =
 544:       cutlass::epilogue::threadblock::OutputTileThreadLayout<
 545:           ThreadblockShape,
 546:           WarpShape,
 547:           DtypeOutput,
 548:           AlignmentOutput,
 549:           NumEVTEpilogueStages>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 551-551
```cpp
 551:   using Accum = cutlass::epilogue::threadblock::VisitorAccFetch;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 553-556
```cpp
 553:   using XScale = cutlass::epilogue::threadblock::VisitorColBroadcast<
 554:       OutputTileThreadMap, DtypeScale,
 555:       cute::Stride<cute::_1, cute::_0, int64_t>>;
 556:   using XScaleArguments = typename XScale::Arguments;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 558-561
```cpp
 558:   using WScale = cutlass::epilogue::threadblock::VisitorRowBroadcast<
 559:       OutputTileThreadMap, DtypeScale,
 560:       cute::Stride<cute::_0, cute::_1, int64_t>>;
 561:   using WScaleArguments = typename WScale::Arguments;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 563-566
```cpp
 563:   using Bias = cutlass::epilogue::threadblock::VisitorRowBroadcast<
 564:       OutputTileThreadMap, DtypeBias,
 565:       cute::Stride<cute::_0, cute::_1, int64_t>>;
 566:   using BiasArguments = typename Bias::Arguments;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 568-575
```cpp
 568:   using ApplyXScale = cutlass::epilogue::threadblock::VisitorCompute<
 569:       cutlass::multiplies, DtypeEpilogue, DtypeEpilogue,
 570:       cutlass::FloatRoundStyle::round_to_nearest
 571:   >;
 572:   using EVTApplyXScale = cutlass::epilogue::threadblock::Sm80EVT<
 573:       ApplyXScale,
 574:       Accum,
 575:       XScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 577-584
```cpp
 577:   using ApplyWScale = cutlass::epilogue::threadblock::VisitorCompute<
 578:       cutlass::multiplies, DtypeEpilogue, DtypeEpilogue,
 579:       cutlass::FloatRoundStyle::round_to_nearest
 580:   >;
 581:   using EVTApplyWScale = cutlass::epilogue::threadblock::Sm80EVT<
 582:       ApplyWScale,
 583:       EVTApplyXScale,
 584:       WScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 586-593
```cpp
 586:   using ApplyBias = cutlass::epilogue::threadblock::VisitorCompute<
 587:       cutlass::plus, DtypeEpilogue, DtypeEpilogue,
 588:       cutlass::FloatRoundStyle::round_to_nearest
 589:   >;
 590:   using EVTApplyBias = cutlass::epilogue::threadblock::Sm80EVT<
 591:       ApplyBias,
 592:       EVTApplyWScale,
 593:       Bias>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 595-599
```cpp
 595:   using Output = cutlass::epilogue::threadblock::VisitorAuxStore<
 596:       OutputTileThreadMap, DtypeOutput,
 597:       cutlass::FloatRoundStyle::round_to_nearest,
 598:       cute::Stride<int64_t, cute::_1, int64_t> // StrideMNL
 599:   >;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 601-603
```cpp
 601:   using EVTOutput = cutlass::epilogue::threadblock::Sm80EVT<
 602:       Output,
 603:       EVTApplyBias>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 605-618
```cpp
 605:   using EVTKernel = at::cuda::detail::enable_2x_kernel_for_sm89<
 606:       typename cutlass::gemm::kernel::DefaultGemmWithVisitor<
 607:           DtypeA, LayoutInputA, cutlass::ComplexTransform::kNone, AlignmentInputA,
 608:           DtypeB, LayoutInputB, cutlass::ComplexTransform::kNone, AlignmentInputB,
 609:           DtypeOutput, LayoutOutput, AlignmentOutput,
 610:           DtypeAccum,
 611:           DtypeEpilogue,
 612:           OperatorClass,
 613:           ArchTag,
 614:           ThreadblockShape,
 615:           WarpShape,
 616:           InstructionShape,
 617:           EVTOutput,
 618:           ThreadblockSwizzle,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 619-621
```cpp
 619:           NumStages,
 620:           Operator,
 621:           NumEVTEpilogueStages>::GemmKernel>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 623-623
```cpp
 623:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<EVTKernel>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 625-626
```cpp
 625:   cutlass::gemm::GemmCoord problem_size(M, N, K);
 626:   constexpr auto SplitKFactor = 1;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 628-642
```cpp
 628:   XScaleArguments x_scale_arguments{
 629:       (DtypeScale*)x_scale.data_ptr(),
 630:       DtypeScale(1),
 631:       {cute::_1{}, cute::_0{}, problem_size.m()}
 632:   };
 633:   WScaleArguments w_scale_arguments{
 634:       (DtypeScale*)w_scale.data_ptr(),
 635:       DtypeScale(1),
 636:       {cute::_0{}, cute::_1{}, problem_size.n()}
 637:   };
 638:   BiasArguments bias_arguments{
 639:       bias.has_value() ? reinterpret_cast<DtypeBias*>(bias->data_ptr()) : nullptr,
 640:       DtypeBias(0),
 641:       {cute::_0{}, cute::_1{}, problem_size.n()}
 642:   };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 643-662
```cpp
 643:   typename Output::Arguments output_arguments{
 644:     (DtypeOutput*)out.data_ptr(),
 645:     {problem_size.n(), cute::_1{}, problem_size.mn().product()}
 646:   };
 647:   typename EVTOutput::Arguments callback_arguments{
 648:     {
 649:       {
 650:         {
 651:           {},                 // Accum
 652:           x_scale_arguments,  // XScale
 653:           {}                  // ApplyXScale
 654:         },                    // EVTApplyXScale
 655:         w_scale_arguments,    // WScale
 656:         {}                    // ApplyWScale
 657:       },                      // EVTApplyWScale
 658:       bias_arguments,         // Bias
 659:       {}                      // ApplyBias
 660:     },                        // EVTApplyBias
 661:     output_arguments          // Output
 662:   };                          // EVTOutput
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 664-677
```cpp
 664:   typename Gemm::Arguments arguments(
 665:     cutlass::gemm::GemmUniversalMode::kGemm,
 666:     problem_size,
 667:     SplitKFactor,
 668:     callback_arguments,           // arguments of EVT callbacks
 669:     (DtypeA*)XQ.data_ptr(),
 670:     (DtypeB*)WQ.data_ptr(),
 671:     nullptr,                      // ptr C (unused)
 672:     nullptr,                      // ptr D (unused)
 673:     problem_size.mk().product(),  // batch stride A
 674:     problem_size.nk().product(),  // batch stride B
 675:     0,                            // batch stride C (unused)
 676:     0,                            // batch stride D (unused)
 677:     problem_size.k(),             // stride A
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 678-680
```cpp
 678:     problem_size.k(),             // stride B
 679:     0,                            // stride C (unused)
 680:     0);                           // stride D (unused)
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 682-682
```cpp
 682:   Gemm gemm;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 684-685
```cpp
 684:   // Using the arguments, query for extra workspace required for matrix
 685:   // multiplication computation
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 686-686
```cpp
 686:   size_t workspace_size = Gemm::get_workspace_size(arguments);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 688-688
```cpp
 688:   // Allocate workspace memory
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 689-691
```cpp
 689:   auto workspace = XQ.new_empty(
 690:       {static_cast<int64_t>(workspace_size)},
 691:       at::TensorOptions().dtype(at::kByte));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 693-693
```cpp
 693:   // Check the problem size is supported or not
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 694-697
```cpp
 694:   cutlass::Status status = gemm.can_implement(arguments);
 695:   if (status != cutlass::Status::kSuccess) {
 696:     throw std::runtime_error("cutlass cannot implement");
 697:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 699-699
```cpp
 699:   // Initialize CUTLASS kernel with arguments and workspace pointer
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 700-703
```cpp
 700:   status = gemm.initialize(arguments, workspace.data_ptr(), at::cuda::getCurrentCUDAStream());
 701:   if (status != cutlass::Status::kSuccess) {
 702:     throw std::runtime_error("cutlass cannot initialize");
 703:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 705-712
```cpp
 705:   status = gemm(at::cuda::getCurrentCUDAStream());
 706:   if (status != cutlass::Status::kSuccess) {
 707:     throw std::runtime_error(
 708:         std::string("cutlass cannot run") +
 709:         cutlass::cutlassGetStatusString(status));
 710:   }
 711:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 712: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 714-735
```cpp
 714: template <typename ClusterShape, typename ArchTag, typename... Types>
 715: void dispatch_fp8_rowwise_kernel_on_tile_size(
 716:     at::Tensor XQ,
 717:     at::Tensor WQ,
 718:     at::Tensor x_scale,
 719:     at::Tensor w_scale,
 720:     std::optional<at::Tensor> bias,
 721:     at::Tensor out,
 722:     const int swizzle) {
 723:   int M = XQ.size(0);
 724:   int N = WQ.size(1);
 725: 
 726:   int smTarget = at::cuda::getDeviceProperties(out.device().index())->multiProcessorCount;
 727:   if (at::globalContext()._SMCarveout_EXPERIMENTAL().has_value()) {
 728:     smTarget -= at::globalContext()._SMCarveout_EXPERIMENTAL().value();
 729:   }
 730: 
 731:   // We prefer to use smaller tiles (less wasted compute in case of padding),
 732:   // but if this causes us to have more CUDA blocks than there are SMs on the
 733:   // GPU then we'll hit wave quantization, hence we'll switch to larger tiles.
 734:   const bool use_smaller_tiles = ceildiv(M, 64 * cute::get<0>(ClusterShape{})) *
 735:           ceildiv(N, 128 * cute::get<1>(ClusterShape{})) <=
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_tile_size`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_tile_size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 736-736
```cpp
 736:       smTarget / cute::size(ClusterShape{});
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 738-759
```cpp
 738:   if (use_smaller_tiles) {
 739:     if constexpr (std::is_same_v<ArchTag, cutlass::arch::Sm90>) {
 740:       return f8f8bf16_rowwise_impl<
 741:           /*TileShape=*/cute::Shape<cute::_64, cute::_128, cute::_128>,
 742:           ClusterShape,
 743:           Types...>(XQ, WQ, x_scale, w_scale, bias, out, swizzle);
 744:     } else {
 745:       return f8f8bf16_rowwise_impl_sm100_sm120<
 746:         ArchTag,
 747:         /*TileShape=*/cute::Shape<cute::_64, cute::_128, cute::_128>,
 748:         ClusterShape,
 749:         Types...>(XQ, WQ, x_scale, w_scale, bias, out, swizzle);
 750:     }
 751:   } else {
 752:     if constexpr (std::is_same_v<ArchTag, cutlass::arch::Sm90>) {
 753:       return f8f8bf16_rowwise_impl<
 754:         /*TileShape=*/cute::Shape<cute::_128, cute::_128, cute::_128>,
 755:         ClusterShape,
 756:         Types...>(XQ, WQ, x_scale, w_scale, bias, out, swizzle);
 757:     } else {
 758:       return f8f8bf16_rowwise_impl_sm100_sm120<
 759:         ArchTag,
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 760-760
```cpp
 760:         /*TileShape=*/cute::Shape<cute::_128, cute::_128, cute::_128>,
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 761-765
```cpp
 761:         ClusterShape,
 762:         Types...>(XQ, WQ, x_scale, w_scale, bias, out, swizzle);
 763:     }
 764:   }
 765: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 767-780
```cpp
 767: template <
 768:     typename ClusterShape,
 769:     typename Transposed,
 770:     typename ArchTag,
 771:     typename FastAccum,
 772:     typename DtypeA,
 773:     typename DtypeB,
 774:     typename DtypeBias,
 775:     typename DtypeOutput>
 776: void handle_transposition(
 777:     at::Tensor XQ,
 778:     at::Tensor WQ,
 779:     at::Tensor x_scale,
 780:     at::Tensor w_scale,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 781-802
```cpp
 781:     std::optional<at::Tensor> bias,
 782:     at::Tensor out,
 783:     const int swizzle=1) {
 784:   if constexpr (!Transposed::value) {
 785:     dispatch_fp8_rowwise_kernel_on_tile_size<
 786:         ClusterShape,
 787:         ArchTag,
 788:         Transposed,
 789:         FastAccum,
 790:         DtypeA,
 791:         DtypeB,
 792:         DtypeBias,
 793:         DtypeOutput>(XQ, WQ, x_scale, w_scale, bias, out, swizzle);
 794:   } else {
 795:     dispatch_fp8_rowwise_kernel_on_tile_size<
 796:         ClusterShape,
 797:         ArchTag,
 798:         Transposed,
 799:         FastAccum,
 800:         DtypeB,
 801:         DtypeA,
 802:         DtypeBias,
```
- EN: This block defines or continues the implementation of `constexpr`.
- CN: 该代码块定义或继续实现 `constexpr`。

### Lines 803-805
```cpp
 803:         DtypeOutput>(WQ.t(), XQ.t(), w_scale.t(), x_scale.t(), bias, out.t(), swizzle);
 804:   }
 805: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 807-828
```cpp
 807: template <typename... Types>
 808: void dispatch_fp8_rowwise_kernel_on_cluster_size_and_transpose(
 809:     at::Tensor XQ,
 810:     at::Tensor WQ,
 811:     at::Tensor x_scale,
 812:     at::Tensor w_scale,
 813:     std::optional<at::Tensor> bias,
 814:     at::Tensor out) {
 815:   int M = XQ.size(0);
 816:   int N = WQ.size(1);
 817: 
 818:   // All the tiles we use have sizes which are multiples of 64, hence any
 819:   // non-multiple of 64 will get padded anyways. Let's round up to simplify.
 820:   M = round_up_to_nearest_multiple(M, 64);
 821:   N = round_up_to_nearest_multiple(N, 64);
 822: 
 823:   // Small/skinny shapes with odd multiples of 64.
 824:   if (M == 64 && N >= 3072) {
 825:     return handle_transposition<
 826:         /*ClusterShape=*/cute::Shape<cute::_1, cute::_2, cute::_1>,
 827:         /*Transposed=*/std::false_type,
 828:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_cluster_size_and_transpose`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_cluster_size_and_transpose`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 829-842
```cpp
 829:   }
 830:   if (N == 64 && M >= 3072) {
 831:     return handle_transposition<
 832:         /*ClusterShape=*/cute::Shape<cute::_1, cute::_2, cute::_1>,
 833:         /*Transposed=*/std::true_type,
 834:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 835:   }
 836:   if (M == 192 && N >= 4096) {
 837:     return handle_transposition<
 838:         /*ClusterShape=*/cute::Shape<cute::_1, cute::_2, cute::_1>,
 839:         /*Transposed=*/std::true_type,
 840:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 841:   }
 842:   if (N == 192 && M >= 4096) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 843-847
```cpp
 843:     return handle_transposition<
 844:         /*ClusterShape=*/cute::Shape<cute::_1, cute::_2, cute::_1>,
 845:         /*Transposed=*/std::false_type,
 846:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 847:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 849-849
```cpp
 849:   // Now to odd multiples of 128 (but only if not too large).
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 850-863
```cpp
 850:   if (M * N <= 4096 * 4096) {
 851:     if (M % 256 > 0 && N % 256 == 0) {
 852:       return handle_transposition<
 853:           /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 854:           /*Transposed=*/std::true_type,
 855:           Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 856:     }
 857:     if (N % 256 > 0 && M % 256 == 0) {
 858:       return handle_transposition<
 859:           /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 860:           /*Transposed=*/std::false_type,
 861:           Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 862:     }
 863:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 864-876
```cpp
 864:   if (M % 256 > 0 && N % 256 > 0) {
 865:     if ((M <= N) ^ (M * N <= 1024 * 1024)) {
 866:       return handle_transposition<
 867:           /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 868:           /*Transposed=*/std::true_type,
 869:           Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 870:     } else {
 871:       return handle_transposition<
 872:           /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 873:           /*Transposed=*/std::false_type,
 874:           Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 875:     }
 876:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 878-880
```cpp
 878:   // General case for large tensors.
 879: 
 880:   // Large M, N, k
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 881-902
```cpp
 881:   if (M >= 4096 && N >= 4096) {
 882:     if (M >= N){
 883:           return handle_transposition<
 884:           /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 885:           /*Transposed=*/std::false_type,
 886:           Types...>(XQ, WQ, x_scale, w_scale, bias, out, 8);
 887:     }
 888:     return handle_transposition<
 889:         /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 890:         /*Transposed=*/std::true_type,
 891:         Types...>(XQ, WQ, x_scale, w_scale, bias, out, 8);
 892:   }
 893:   if ((M <= N) ^ (M >= 2048 && N >= 2048)) {
 894:     return handle_transposition<
 895:         /*ClusterShape=*/cute::Shape<cute::_1, cute::_2, cute::_1>,
 896:         /*Transposed=*/std::true_type,
 897:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 898:   } else {
 899:     return handle_transposition<
 900:         /*ClusterShape=*/cute::Shape<cute::_2, cute::_1, cute::_1>,
 901:         /*Transposed=*/std::true_type,
 902:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 903-904
```cpp
 903:   }
 904: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 906-927
```cpp
 906: template <typename... Types>
 907: void dispatch_fp8_rowwise_kernel_sm89(
 908:     at::Tensor XQ,
 909:     at::Tensor WQ,
 910:     at::Tensor x_scale,
 911:     at::Tensor w_scale,
 912:     std::optional<at::Tensor> bias,
 913:     at::Tensor out) {
 914:   int M = XQ.size(0);
 915: 
 916:   if (M <= 16) {
 917:     return f8f8bf16_rowwise_impl_sm89<
 918:         /*ThreadblockShape=*/cutlass::gemm::GemmShape<16, 64, 128>,
 919:         /*WarpShape=*/cutlass::gemm::GemmShape<16, 64, 64>,
 920:         /*NumStages=*/5,
 921:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 922:   } else if (M <= 32) {
 923:     return f8f8bf16_rowwise_impl_sm89<
 924:         /*ThreadblockShape=*/cutlass::gemm::GemmShape<32, 64, 128>,
 925:         /*WarpShape=*/cutlass::gemm::GemmShape<16, 64, 64>,
 926:         /*NumStages=*/5,
 927:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_sm89`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_sm89`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 928-941
```cpp
 928:   } else if (M <= 64) {
 929:     return f8f8bf16_rowwise_impl_sm89<
 930:         /*ThreadblockShape=*/cutlass::gemm::GemmShape<64, 64, 128>,
 931:         /*WarpShape=*/cutlass::gemm::GemmShape<32, 64, 64>,
 932:         /*NumStages=*/5,
 933:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 934:   } else if (M <= 256) {
 935:     return f8f8bf16_rowwise_impl_sm89<
 936:         /*ThreadblockShape=*/cutlass::gemm::GemmShape<64, 128, 128>,
 937:         /*WarpShape=*/cutlass::gemm::GemmShape<64, 64, 64>,
 938:         /*NumStages=*/3,
 939:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 940:   } else {
 941:     return f8f8bf16_rowwise_impl_sm89<
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 942-944
```cpp
 942:         /*ThreadblockShape=*/cutlass::gemm::GemmShape<128, 128, 64>,
 943:         /*WarpShape=*/cutlass::gemm::GemmShape<64, 64, 64>,
 944:         /*NumStages=*/5,
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 945-947
```cpp
 945:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 946:   }
 947: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 949-970
```cpp
 949: template <typename... Types>
 950: void dispatch_fp8_rowwise_kernel_on_sm(
 951:     at::Tensor XQ,
 952:     at::Tensor WQ,
 953:     at::Tensor x_scale,
 954:     at::Tensor w_scale,
 955:     std::optional<at::Tensor> bias,
 956:     at::Tensor out) {
 957:   cudaDeviceProp* properties = at::cuda::getCurrentDeviceProperties();
 958:   const bool sm89 = properties != nullptr && properties->major == 8 && properties->minor == 9;
 959:   const bool sm9x = properties != nullptr && properties->major == 9;
 960:   const bool sm10x = properties != nullptr && properties->major == 10;
 961:   const bool sm11x = properties != nullptr && properties->major == 11;
 962:   const bool sm12x = properties != nullptr && properties->major == 12;
 963:   if (!(sm89 || sm9x || sm10x || sm11x || sm12x)) {
 964:     TORCH_CHECK(
 965:         false, "Rowwise scaling is not currently supported on your device");
 966:   }
 967: 
 968:   if (sm9x) {
 969:     dispatch_fp8_rowwise_kernel_on_cluster_size_and_transpose<
 970:       /*ArchTag=*/cutlass::arch::Sm90,
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_sm`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_sm`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 971-984
```cpp
 971:       Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 972:   } else if (sm10x || sm11x) {
 973:     dispatch_fp8_rowwise_kernel_on_cluster_size_and_transpose<
 974:       /*ArchTag=*/cutlass::arch::Sm100,
 975:       Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 976:   } else if (sm12x) {
 977:     // sm12x doesn't have multicast feature
 978:     handle_transposition<
 979:       /*ClusterShape=*/cute::Shape<cute::_1, cute::_1, cute::_1>,
 980:       /*Transposed=*/std::false_type,
 981:       /*ArchTag=*/cutlass::arch::Sm120,
 982:       Types...>(XQ, WQ, x_scale, w_scale, bias, out);
 983:   } else {
 984:     dispatch_fp8_rowwise_kernel_sm89<Types...>(XQ, WQ, x_scale, w_scale, bias, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 985-986
```cpp
 985:   }
 986: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 988-1006
```cpp
 988: template <typename... Types>
 989: void dispatch_fp8_rowwise_kernel_on_fast_accum(
 990:     at::Tensor XQ,
 991:     at::Tensor WQ,
 992:     at::Tensor x_scale,
 993:     at::Tensor w_scale,
 994:     std::optional<at::Tensor> bias,
 995:     bool use_fast_accum,
 996:     at::Tensor out) {
 997:   if (use_fast_accum) {
 998:     dispatch_fp8_rowwise_kernel_on_sm<
 999:         std::true_type,
1000:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
1001:   } else {
1002:     dispatch_fp8_rowwise_kernel_on_sm<
1003:         std::false_type,
1004:         Types...>(XQ, WQ, x_scale, w_scale, bias, out);
1005:   }
1006: }
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_fast_accum`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_fast_accum`。

### Lines 1008-1028
```cpp
1008: template <typename... Types>
1009: void dispatch_fp8_rowwise_kernel_on_input_dtypes(
1010:     at::Tensor XQ,
1011:     at::Tensor WQ,
1012:     at::Tensor x_scale,
1013:     at::Tensor w_scale,
1014:     std::optional<at::Tensor> bias,
1015:     bool use_fast_accum,
1016:     at::Tensor out) {
1017:   if (XQ.dtype() == at::kFloat8_e5m2) {
1018:     dispatch_fp8_rowwise_kernel_on_fast_accum<
1019:         cutlass::float_e5m2_t,
1020:         cutlass::float_e4m3_t,
1021:         Types...>(XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1022:   } else {
1023:     dispatch_fp8_rowwise_kernel_on_fast_accum<
1024:         cutlass::float_e4m3_t,
1025:         cutlass::float_e4m3_t,
1026:         Types...>(XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1027:   }
1028: }
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_input_dtypes`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_input_dtypes`。

### Lines 1030-1051
```cpp
1030: void dispatch_fp8_rowwise_kernel_on_bias_dtype(
1031:     at::Tensor XQ,
1032:     at::Tensor WQ,
1033:     at::Tensor x_scale,
1034:     at::Tensor w_scale,
1035:     std::optional<at::Tensor> bias,
1036:     bool use_fast_accum,
1037:     at::Tensor out) {
1038:   if (bias.has_value() && bias->dtype() == at::kBFloat16) {
1039:     dispatch_fp8_rowwise_kernel_on_input_dtypes<
1040:         cutlass::bfloat16_t,
1041:         cutlass::bfloat16_t>
1042:         (XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1043:   } else if (bias.has_value() && bias->dtype() == at::kHalf){
1044:     TORCH_CHECK(out.dtype() == at::kHalf, "Output should be Float16 when bias is Float16");
1045:     dispatch_fp8_rowwise_kernel_on_input_dtypes<
1046:         cutlass::half_t,
1047:         cutlass::half_t>
1048:         (XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1049:   } else {
1050:     dispatch_fp8_rowwise_kernel_on_input_dtypes<
1051:         float,
```
- EN: This block defines or continues the implementation of `dispatch_fp8_rowwise_kernel_on_bias_dtype`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_rowwise_kernel_on_bias_dtype`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 1052-1056
```cpp
1052:         cutlass::bfloat16_t>
1053:         //Types...>
1054:         (XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1055:   }
1056: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1058-1079
```cpp
1058: void check_inputs(
1059:     const at::Tensor& a,
1060:     const at::Tensor& b,
1061:     const at::Tensor& scale_a,
1062:     const at::Tensor& scale_b,
1063:     const std::optional<at::Tensor>& bias,
1064:     const at::Tensor& out) {
1065:   TORCH_CHECK(a.is_cuda());
1066:   TORCH_CHECK(a.device() == b.device());
1067:   TORCH_CHECK(scale_a.device() == a.device());
1068:   TORCH_CHECK(scale_b.device() == b.device());
1069: 
1070:   TORCH_CHECK(a.dtype() == at::kFloat8_e4m3fn || a.dtype() == at::kFloat8_e5m2);
1071:   TORCH_CHECK(b.dtype() == at::kFloat8_e4m3fn);
1072:   TORCH_CHECK(scale_a.dtype() == at::kFloat);
1073:   TORCH_CHECK(scale_b.dtype() == at::kFloat);
1074: 
1075:   TORCH_CHECK(a.dim() == 2);
1076:   TORCH_CHECK(b.dim() == 2);
1077:   TORCH_CHECK(a.size(1) == b.size(0));
1078:   TORCH_CHECK(scale_a.dim() == 2);
1079:   TORCH_CHECK(scale_b.dim() == 2);
```
- EN: This block defines or continues the implementation of `check_inputs`.
- CN: 该代码块定义或继续实现 `check_inputs`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1080-1083
```cpp
1080:   TORCH_CHECK(scale_a.size(0) == a.size(0));
1081:   TORCH_CHECK(scale_a.size(1) == 1);
1082:   TORCH_CHECK(scale_b.size(0) == 1);
1083:   TORCH_CHECK(scale_b.size(1) == b.size(1));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1085-1090
```cpp
1085:   TORCH_CHECK(a.stride(1) == 1);
1086:   TORCH_CHECK(a.stride(0) >= a.size(1));
1087:   TORCH_CHECK(b.stride(0) == 1);
1088:   TORCH_CHECK(b.stride(1) >= b.size(0));
1089:   TORCH_CHECK(scale_a.stride(0) == 1);
1090:   TORCH_CHECK(scale_b.stride(1) == 1);
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1092-1098
```cpp
1092:   if (bias.has_value()) {
1093:     TORCH_CHECK(bias->device() == b.device());
1094:     TORCH_CHECK(bias->dtype() == at::kFloat || bias->dtype() == at::kBFloat16 || bias->dtype() == at::kHalf);
1095:     TORCH_CHECK(bias->dim() == 1);
1096:     TORCH_CHECK(bias->size(0) == b.size(1));
1097:     TORCH_CHECK(bias->stride(0) == 1);
1098:   }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1100-1107
```cpp
1100:   TORCH_CHECK(out.device() == a.device());
1101:   TORCH_CHECK(out.dtype() == at::kBFloat16 || out.dtype() == at::kHalf);
1102:   TORCH_CHECK(out.dim() == 2);
1103:   TORCH_CHECK(out.size(0) == a.size(0));
1104:   TORCH_CHECK(out.size(1) == b.size(1));
1105:   TORCH_CHECK(out.stride(1) == 1);
1106:   TORCH_CHECK(out.stride(0) >= out.size(1));
1107: }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 1109-1109
```cpp
1109: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 1111-1111
```cpp
1111: #endif // !defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 1113-1133
```cpp
1113: namespace at::cuda::detail {
1114: void f8f8bf16_rowwise(
1115:     at::Tensor XQ, // FP8
1116:     at::Tensor WQ, // FP8
1117:     at::Tensor x_scale, // FP32
1118:     at::Tensor w_scale, // FP32
1119:     std::optional<at::Tensor> bias, // BF16
1120:     bool use_fast_accum,
1121:     at::Tensor& out) {
1122: #if defined(BUILD_ROWWISE_FP8_KERNEL)
1123:   check_inputs(XQ, WQ, x_scale, w_scale, bias, out);
1124: 
1125:   dispatch_fp8_rowwise_kernel_on_bias_dtype(
1126:       XQ, WQ, x_scale, w_scale, bias, use_fast_accum, out);
1127: #else // BUILD_ROWWISE_FP8_KERNEL
1128:   TORCH_CHECK(
1129:       false, "Rowwise scaling is not currently supported on your device");
1130: #endif
1131: }
1132: 
1133: } // namespace at::cuda::detail
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `f8f8bf16_rowwise`.
- CN: 该代码块定义或继续实现 `f8f8bf16_rowwise`。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/nvrtc_stub/ATenNVRTC.h>`
  - `<c10/macros/Macros.h>`
  - `<cute/tensor.hpp>`
  - `<cutlass/core_io.h>`
  - `<cutlass/cutlass.h>`
  - `<cutlass/gemm/device/gemm.h>`
  - `<cutlass/half.h>`
  - `<cutlass/numeric_types.h>`
  - `<cutlass/trace.h>`
- Runtime symbols / 运行时符号:
  - `nvrtc_stub`
  - `at::cuda::detail::enable_3x_kernel_for_sm9x`
  - `at::cuda::getDeviceProperties`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::detail::enable_3x_kernel_for_sm10_or_later`
  - `at::cuda::detail::enable_2x_kernel_for_sm89`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::detail`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
