# GroupMM.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/GroupMM.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ceildiv`, `round_up_to_nearest_multiple`, `bf16bf16_grouped_gemm_impl_sm90_sm100`, `dispatch_bf16_grouped_kernel_on_tile_size`.
- 用途（中文）: 实现与 `ceildiv`, `round_up_to_nearest_multiple`, `bf16bf16_grouped_gemm_impl_sm90_sm100`, `dispatch_bf16_grouped_kernel_on_tile_size` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/core/Tensor.h>
   4: #include <ATen/cuda/CUDAContext.h>
   5: #include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
   6: #include <c10/cuda/CUDACachingAllocator.h>
   7: #include <c10/macros/Macros.h>
   8: #include <c10/util/irange.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/core/Tensor.h>`, `<ATen/cuda/CUDAContext.h>`。

### Lines 11-11
```cpp
  11: // Three warnings in Cutlass included header files
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 12-14
```cpp
  12: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wset-but-not-used")
  13: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-parameter")
  14: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-variable")
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 16-18
```cpp
  16: // Determine if the architecture supports rowwise scaled mm
  17: // Currently failing on windows with:
  18: // https://github.com/NVIDIA/cutlass/issues/1571
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 19-32
```cpp
  19: #if !defined(USE_ROCM) && !defined(_WIN32) && defined(CUDA_VERSION)
  20: 
  21: #define BUILD_GG_KERNEL
  22: #endif
  23: 
  24: #if defined(BUILD_GG_KERNEL)
  25: 
  26: #include <cute/tensor.hpp>
  27: #include <cutlass/core_io.h>
  28: #include <cutlass/cutlass.h>
  29: #include <cutlass/gemm/device/gemm.h>
  30: #include <cutlass/numeric_types.h>
  31: #include <cutlass/trace.h>
  32: #include <cutlass/version.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cute/tensor.hpp>`, `<cutlass/core_io.h>`, `<cutlass/cutlass.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cute/tensor.hpp>`, `<cutlass/core_io.h>`, `<cutlass/cutlass.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 33-46
```cpp
  33: #include <ATen/native/cuda/GroupMMCommon.cuh>
  34: 
  35: #include <cutlass/epilogue/collective/collective_builder.hpp>
  36: #include <cutlass/epilogue/threadblock/fusion/visitors.hpp>
  37: #include <cutlass/gemm/collective/collective_builder.hpp>
  38: #include <cutlass/gemm/device/gemm_universal.h>
  39: #include <cutlass/gemm/device/gemm_universal_adapter.h>
  40: #include <cutlass/gemm/kernel/default_gemm_universal_with_visitor.h>
  41: 
  42: #include <cute/atom/mma_atom.hpp>
  43: #include <cutlass/gemm/dispatch_policy.hpp>
  44: #include <cutlass/gemm/kernel/gemm_universal.hpp>
  45: 
  46: #include <ATen/native/cuda/cutlass_common.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/GroupMMCommon.cuh>`, `<cutlass/epilogue/collective/collective_builder.hpp>`, `<cutlass/epilogue/threadblock/fusion/visitors.hpp>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/GroupMMCommon.cuh>`, `<cutlass/epilogue/collective/collective_builder.hpp>`, `<cutlass/epilogue/threadblock/fusion/visitors.hpp>`。

### Lines 48-69
```cpp
  48: namespace {
  49: using Strides = at::cuda::detail::Strides; // std::array<int64_t, 3>;
  50: 
  51: template <typename ArchTag, bool PONGOr2SM, typename TB_M, typename TB_N, typename TB_K>
  52: struct Schedule {
  53:   // SM90
  54:   using CooperativeSchedule =
  55:       cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperative;
  56:   using PongSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpong;
  57:   using CooperativeEpilogueSchedule =
  58:       cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  59:   using PongEpilogueSchedule =
  60:       cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  61:   // SM100
  62:   using MMA1SMKernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecialized1SmSm100;
  63:   using MMA1SMEpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized1Sm;
  64:   using MMA2SMKernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecialized2SmSm100;
  65:   using MMA2SMEpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecialized2Sm;
  66: 
  67:   using KernelSchedule = cute::conditional_t<std::is_same_v<ArchTag, cutlass::arch::Sm100>,
  68:     cute::conditional_t<PONGOr2SM, MMA2SMKernelSchedule, MMA1SMKernelSchedule>,
  69:     cute::conditional_t<PONGOr2SM, PongSchedule, CooperativeSchedule>>;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 70-72
```cpp
  70:   using EpilogueSchedule = cute::conditional_t<std::is_same_v<ArchTag, cutlass::arch::Sm100>,
  71:     cute::conditional_t<PONGOr2SM, MMA2SMEpilogueSchedule, MMA1SMEpilogueSchedule>,
  72:     cute::conditional_t<PONGOr2SM, PongEpilogueSchedule, CooperativeEpilogueSchedule>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 74-74
```cpp
  74: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 76-78
```cpp
  76: int ceildiv(int a, int b) {
  77:   return (a + b - 1) / b;
  78: }
```
- EN: This block defines or continues the implementation of `ceildiv`.
- CN: 该代码块定义或继续实现 `ceildiv`。

### Lines 80-82
```cpp
  80: int round_up_to_nearest_multiple(int a, int b) {
  81:   return ceildiv(a, b) * b;
  82: }
```
- EN: This block defines or continues the implementation of `round_up_to_nearest_multiple`.
- CN: 该代码块定义或继续实现 `round_up_to_nearest_multiple`。

### Lines 84-105
```cpp
  84: template <
  85:     typename ArchTag,
  86:     bool a_row_major,
  87:     bool b_row_major,
  88:     bool PONGOr2SM,
  89:     typename TB_M,
  90:     typename TB_N,
  91:     typename TB_K>
  92: void bf16bf16_grouped_gemm_impl_sm90_sm100(
  93:     at::Tensor mat_a, // bf16
  94:     at::Tensor mat_b, // bf16
  95:     std::optional<at::Tensor> offs,
  96:     std::optional<at::Tensor> bias, // BF16
  97:     at::Tensor& out) {
  98:   using DtypeA = cutlass::bfloat16_t;
  99:   using DtypeB = cutlass::bfloat16_t;
 100:   using DtypeOutput = cutlass::bfloat16_t;
 101:   using DtypeAccum = float;
 102:   using LayoutA = cute::conditional_t<
 103:       a_row_major,
 104:       cutlass::layout::RowMajor,
 105:       cutlass::layout::ColumnMajor>;
```
- EN: This block defines or continues the implementation of `bf16bf16_grouped_gemm_impl_sm90_sm100`.
- CN: 该代码块定义或继续实现 `bf16bf16_grouped_gemm_impl_sm90_sm100`。

### Lines 106-106
```cpp
 106:   constexpr int AlignmentA = 16 / sizeof(DtypeA);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 108-121
```cpp
 108:   using LayoutB = cute::conditional_t<
 109:       b_row_major,
 110:       cutlass::layout::RowMajor,
 111:       cutlass::layout::ColumnMajor>;
 112:   constexpr int AlignmentB = 16 / sizeof(DtypeB);
 113:   using LayoutOutput = cutlass::layout::RowMajor;
 114:   constexpr int AlignmentOutput = 16 / sizeof(DtypeOutput);
 115:   using OperatorClass = cutlass::arch::OpClassTensorOp;
 116:   using TileShape = cute::Shape<TB_M, TB_N, TB_K>;
 117:   using ClusterShape = cute::Shape<cute::_2, cute::_1, cute::_1>;
 118:   using KernelSchedule =
 119:       typename Schedule<ArchTag, PONGOr2SM, TB_M, TB_N, TB_K>::KernelSchedule;
 120:   using EpilogueSchedule =
 121:       typename Schedule<ArchTag, PONGOr2SM, TB_M, TB_N, TB_K>::EpilogueSchedule;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 122-124
```cpp
 122:   using ProblemShape = cutlass::gemm::GroupProblemShape<
 123:       cute::Shape<int32_t, int32_t, int32_t>>; // <M,N,K> per
 124:                                                // group
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 126-139
```cpp
 126:   using CollectiveEpilogue =
 127:       typename cutlass::epilogue::collective::CollectiveBuilder<
 128:           ArchTag,
 129:           OperatorClass,
 130:           TileShape,
 131:           ClusterShape,
 132:           cutlass::epilogue::collective::EpilogueTileAuto,
 133:           DtypeAccum,
 134:           DtypeAccum,
 135:           void, // Indicate there is no beta scaling to save register
 136:           LayoutOutput*,
 137:           AlignmentOutput,
 138:           DtypeOutput,
 139:           LayoutOutput*,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 140-143
```cpp
 140:           AlignmentOutput,
 141:           EpilogueSchedule,
 142:           cutlass::epilogue::fusion::
 143:               LinearCombination<DtypeOutput, DtypeAccum>>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 145-158
```cpp
 145:   using CollectiveMainloop =
 146:       typename cutlass::gemm::collective::CollectiveBuilder<
 147:           ArchTag,
 148:           OperatorClass,
 149:           DtypeA,
 150:           LayoutA*,
 151:           AlignmentA,
 152:           DtypeB,
 153:           LayoutB*,
 154:           AlignmentB,
 155:           DtypeAccum,
 156:           TileShape,
 157:           ClusterShape,
 158:           cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 159-160
```cpp
 159:               sizeof(typename CollectiveEpilogue::SharedStorage))>,
 160:           KernelSchedule>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 162-165
```cpp
 162:   using GemmKernelBase = cutlass::gemm::kernel::GemmUniversal<
 163:       ProblemShape,
 164:       CollectiveMainloop,
 165:       CollectiveEpilogue>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 167-170
```cpp
 167:   using GemmKernel = std::conditional_t<
 168:       std::is_same_v<ArchTag, cutlass::arch::Sm100>,
 169:       at::cuda::detail::enable_3x_kernel_for_sm10<GemmKernelBase>,
 170:       at::cuda::detail::enable_3x_kernel_for_sm9x<GemmKernelBase>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 172-176
```cpp
 172:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
 173:   using StrideA = typename Gemm::GemmKernel::InternalStrideA;
 174:   using StrideB = typename Gemm::GemmKernel::InternalStrideB;
 175:   using StrideOutput = typename Gemm::GemmKernel::InternalStrideD;
 176:   int32_t M, N, K, group_count;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 178-180
```cpp
 178:   M = mat_a.size(-2);
 179:   K = mat_a.size(-1);
 180:   N = mat_b.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 182-195
```cpp
 182:   if (mat_a.dim() == 2 && mat_b.dim() == 2) {
 183:     // if both inputs are ragged, K is dynamic, M and N come from inputs
 184:     group_count = offs->size(0);
 185:     K = -1;
 186:   } else if (mat_a.dim() == 2) {
 187:     group_count = mat_b.size(0);
 188:     M = -1;
 189:   } else if (mat_b.dim() == 2) {
 190:     group_count = mat_a.size(0);
 191:     N = -1;
 192:   } else {
 193:     // regular bmm
 194:     group_count = mat_a.size(0);
 195:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 197-199
```cpp
 197:   TORCH_CHECK(group_count < 1024, "Can't process more than 1024 groups");
 198:   const int64_t problem_shape_size =
 199:       group_count * ((int64_t)sizeof(ProblemShape::UnderlyingProblemShape));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 201-201
```cpp
 201:   const int64_t stride_size = 3 * group_count * ((int64_t)sizeof(StrideA));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 203-206
```cpp
 203:   // dummy tmas are created based on these pointer-to-pointers
 204:   // the actual values are never used, they are replaced
 205:   // by real addresses, but for dummy tma creation to succeed
 206:   // due to bug in cuda < 12.4 the pointers have to be aligned to 128 bits
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 207-211
```cpp
 207:   const int group_alignment = 16 / sizeof(void*);
 208:   const int aligned_group_count =
 209:       round_up_to_nearest_multiple(group_count, group_alignment);
 210:   int64_t input_args_size = aligned_group_count * 3 * sizeof(void*) +
 211:       problem_shape_size + stride_size;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 213-226
```cpp
 213:   auto& allocator = *c10::cuda::CUDACachingAllocator::get();
 214:   auto input_buf = allocator.allocate(input_args_size);
 215:   void* buf_ptr = input_buf.get();
 216:   DtypeA** inputA_ptrs = reinterpret_cast<DtypeA**>(buf_ptr);
 217:   DtypeB** inputB_ptrs =
 218:       reinterpret_cast<DtypeB**>(inputA_ptrs + aligned_group_count);
 219:   DtypeOutput** output_ptrs =
 220:       reinterpret_cast<DtypeOutput**>(inputB_ptrs + aligned_group_count);
 221:   static_assert(
 222:       sizeof(StrideA) == 8, "expected StrideA to be 8 bytes for alignment");
 223:   StrideA* stride_A =
 224:       reinterpret_cast<StrideA*>(output_ptrs + aligned_group_count);
 225:   StrideB* stride_B = reinterpret_cast<StrideB*>(stride_A + group_count);
 226:   StrideOutput* stride_output =
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 227-230
```cpp
 227:       reinterpret_cast<StrideOutput*>(stride_B + group_count);
 228:   ProblemShape::UnderlyingProblemShape* problem_sizes =
 229:       reinterpret_cast<ProblemShape::UnderlyingProblemShape*>(
 230:           stride_output + group_count);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 232-232
```cpp
 232:   auto stream = at::cuda::getCurrentCUDAStream().stream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 234-238
```cpp
 234:   auto make_strides = [](at::IntArrayRef strides) -> Strides {
 235:     Strides out;
 236:     std::copy(strides.begin(), strides.end(), out.begin());
 237:     return out;
 238:   };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 240-244
```cpp
 240:   Strides tensor_StrideA = make_strides(mat_a.strides());
 241:   Strides tensor_StrideB = make_strides(mat_b.strides());
 242:   Strides tensor_StrideOutput = make_strides(out.strides());
 243:   Strides tensor_ShapeA = make_strides(mat_a.sizes());
 244:   Strides tensor_ShapeB = make_strides(mat_b.sizes());
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 246-259
```cpp
 246:   at::cuda::detail::prepare_grouped_gemm_data<<<1, group_count, 0, stream>>>(
 247:       reinterpret_cast<DtypeA*>(mat_a.data_ptr()),
 248:       reinterpret_cast<DtypeB*>(mat_b.data_ptr()),
 249:       reinterpret_cast<DtypeOutput*>(out.data_ptr()),
 250:       static_cast<float*>(nullptr), // type for template inference
 251:       static_cast<float*>(nullptr), // type for template inference
 252:       inputA_ptrs,
 253:       inputB_ptrs,
 254:       output_ptrs,
 255:       static_cast<float**>(nullptr), // type for template inference
 256:       static_cast<float**>(nullptr), // type for template inference
 257:       problem_sizes,
 258:       stride_A,
 259:       stride_B,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 260-273
```cpp
 260:       stride_output,
 261:       offs.has_value() ? offs->const_data_ptr<int32_t>() : nullptr,
 262:       M,
 263:       N,
 264:       K,
 265:       tensor_StrideA,
 266:       tensor_StrideB,
 267:       tensor_StrideOutput,
 268:       tensor_ShapeA,
 269:       tensor_ShapeB,
 270:       0,
 271:       0,
 272:       a_row_major,
 273:       b_row_major);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 275-275
```cpp
 275:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 277-288
```cpp
 277:   typename Gemm::Arguments arguments{
 278:       cutlass::gemm::GemmUniversalMode::kGrouped,
 279:       {group_count, problem_sizes, nullptr},
 280:       {(const DtypeA**)inputA_ptrs,
 281:        stride_A,
 282:        (const DtypeB**)inputB_ptrs,
 283:        stride_B},
 284:       {{},
 285:        nullptr,
 286:        stride_output,
 287:        output_ptrs,
 288:        stride_output}};
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 290-291
```cpp
 290:   arguments.epilogue.thread.alpha = 1.0;
 291:   arguments.epilogue.thread.dAlpha = {cute::_0{}, cute::_0{}, 0};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 293-298
```cpp
 293:   int sm_count =
 294:       at::cuda::getDeviceProperties(out.device().index())->multiProcessorCount;
 295:   if (at::globalContext()._SMCarveout_EXPERIMENTAL().has_value()) {
 296:     sm_count -= at::globalContext()._SMCarveout_EXPERIMENTAL().value();
 297:   }
 298:   arguments.hw_info.sm_count = sm_count;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 300-313
```cpp
 300:   size_t workspace_size = Gemm::get_workspace_size(arguments);
 301:   auto workspace = allocator.allocate(workspace_size);
 302:   Gemm gemm;
 303:   TORCH_CHECK(
 304:       gemm.can_implement(arguments) == cutlass::Status::kSuccess,
 305:       "cutlass cannot implement");
 306:   TORCH_CHECK(
 307:       gemm.initialize(arguments, workspace.get()) == cutlass::Status::kSuccess,
 308:       "cutlass cannot initialize");
 309:   auto status = gemm(at::cuda::getCurrentCUDAStream());
 310:   TORCH_CHECK(
 311:       status == cutlass::Status::kSuccess,
 312:       "cutlass cannot run, error ",
 313:       int(status));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 314-315
```cpp
 314:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 315: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 317-338
```cpp
 317: template <bool a_row_major, bool b_row_major>
 318: void dispatch_bf16_grouped_kernel_on_tile_size(
 319:     at::Tensor mat_a, // bf16
 320:     at::Tensor mat_b, // bf16
 321:     std::optional<at::Tensor> offs,
 322:     std::optional<at::Tensor> bias, // BF16
 323:     at::Tensor& out) {
 324:   int32_t M, N, K, group_count;
 325: 
 326:   M = mat_a.size(-2);
 327:   K = mat_a.size(-1);
 328:   N = mat_b.size(-1);
 329: 
 330:   // below we assume that gemms are approx same size
 331:   if (mat_a.dim() == 2 && mat_b.dim() == 2) {
 332:     // if both inputs are ragged, K is dynamic, M and N come from inputs
 333:     group_count = offs->size(0);
 334:     K = K / group_count;
 335:   } else if (mat_a.dim() == 2) {
 336:     group_count = mat_b.size(0);
 337:     M = M / group_count;
 338:   } else if (mat_b.dim() == 2) {
```
- EN: This block defines or continues the implementation of `dispatch_bf16_grouped_kernel_on_tile_size`.
- CN: 该代码块定义或继续实现 `dispatch_bf16_grouped_kernel_on_tile_size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 339-348
```cpp
 339:     group_count = mat_a.size(0);
 340:     N = N / group_count;
 341:   }
 342:   //   bool large =
 343:   //       ((M >= 2048 && K >= 2048) || (M >= 2048 && N >= 2048) ||
 344:   //        (K >= 2048 && N >= 2048));
 345:   bool small = (M <= 128 || N <= 128);
 346:   cudaDeviceProp* properties = at::cuda::getCurrentDeviceProperties();
 347:   const bool sm10x = properties != nullptr && properties->major == 10;
 348:   const bool sm11x = properties != nullptr && properties->major == 11;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 350-371
```cpp
 350:   if (sm10x || sm11x) {
 351:     if (small){
 352:       bf16bf16_grouped_gemm_impl_sm90_sm100<
 353:         cutlass::arch::Sm100,
 354:         a_row_major,
 355:         b_row_major,
 356:         /*PONGOr2SM*/ false,
 357:         cute::_128,
 358:         cute::_256,
 359:         cute::_64>(mat_a, mat_b, offs, bias, out); // Tile shape taken from CUTLASS examples, 64 = 128/sizeof(bfloat16)
 360:     } else {
 361:       bf16bf16_grouped_gemm_impl_sm90_sm100<
 362:         cutlass::arch::Sm100,
 363:         a_row_major,
 364:         b_row_major,
 365:         /*PONGOr2SM*/ true,
 366:         cute::_256,
 367:         cute::_256,
 368:         cute::_64>(mat_a, mat_b, offs, bias, out); // Same as above ^
 369:     }
 370:   } else {
 371:     if(small) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 372-385
```cpp
 372:       bf16bf16_grouped_gemm_impl_sm90_sm100<
 373:         cutlass::arch::Sm90,
 374:         a_row_major,
 375:         b_row_major,
 376:         /*PONGOr2SM*/ true,
 377:         cute::_64,
 378:         cute::_128,
 379:         cute::_128>(mat_a, mat_b, offs, bias, out);
 380:     } else {
 381:       bf16bf16_grouped_gemm_impl_sm90_sm100<
 382:         cutlass::arch::Sm90,
 383:         a_row_major,
 384:         b_row_major,
 385:         /*PONGOr2SM*/ false,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 386-391
```cpp
 386:         cute::_128,
 387:         cute::_256,
 388:         cute::_64>(mat_a, mat_b, offs, bias, out);
 389:     }
 390:   }
 391: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 393-414
```cpp
 393: void dispatch_bf16_grouped_kernel_on_ab_transpose(
 394:     at::Tensor mat_a, // bf16
 395:     at::Tensor mat_b, // bf16
 396:     std::optional<at::Tensor> offs,
 397:     std::optional<at::Tensor> bias, // BF16
 398:     at::Tensor& out) {
 399:   // we already checked that one of the strides is 1
 400:   bool a_row_major = mat_a.stride(-1) == 1;
 401:   bool b_row_major = mat_b.stride(-1) == 1;
 402:   if (a_row_major && b_row_major) {
 403:     dispatch_bf16_grouped_kernel_on_tile_size<true, true>(
 404:         mat_a, mat_b, offs, bias, out);
 405:   } else if (a_row_major && !b_row_major) {
 406:     dispatch_bf16_grouped_kernel_on_tile_size<true, false>(
 407:         mat_a, mat_b, offs, bias, out);
 408:   } else if (!a_row_major && b_row_major) {
 409:     dispatch_bf16_grouped_kernel_on_tile_size<false, true>(
 410:         mat_a, mat_b, offs, bias, out);
 411:   } else {
 412:     dispatch_bf16_grouped_kernel_on_tile_size<false, false>(
 413:         mat_a, mat_b, offs, bias, out);
 414:   }
```
- EN: This block defines or continues the implementation of `dispatch_bf16_grouped_kernel_on_ab_transpose`.
- CN: 该代码块定义或继续实现 `dispatch_bf16_grouped_kernel_on_ab_transpose`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 415-415
```cpp
 415: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 417-418
```cpp
 417: } // namespace
 418: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 420-435
```cpp
 420: namespace at::cuda::detail {
 421: 
 422: void bf16bf16_grouped_mm(
 423:     at::Tensor mat_a, // bf16
 424:     at::Tensor mat_b, // bf16
 425:     std::optional<at::Tensor> offs,
 426:     std::optional<at::Tensor> bias, // BF16
 427:     at::Tensor& out) {
 428: #if defined(BUILD_GG_KERNEL)
 429:   dispatch_bf16_grouped_kernel_on_ab_transpose(mat_a, mat_b, offs, bias, out);
 430: #else
 431:   TORCH_CHECK(false, "grouped mm is not supported on your system");
 432: #endif
 433: }
 434: 
 435: } // namespace at::cuda::detail
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `bf16bf16_grouped_mm`.
- CN: 该代码块定义或继续实现 `bf16bf16_grouped_mm`。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/core/Tensor.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/cuda/nvrtc_stub/ATenNVRTC.h>`
  - `<c10/cuda/CUDACachingAllocator.h>`
  - `<c10/macros/Macros.h>`
  - `<c10/util/irange.h>`
  - `<cute/tensor.hpp>`
  - `<cutlass/core_io.h>`
  - `<cutlass/cutlass.h>`
  - `<cutlass/gemm/device/gemm.h>`
  - `<cutlass/numeric_types.h>`
- Runtime symbols / 运行时符号:
  - `nvrtc_stub`
  - `at::cuda::detail::Strides`
  - `at::cuda::detail::enable_3x_kernel_for_sm10`
  - `at::cuda::detail::enable_3x_kernel_for_sm9x`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::detail::prepare_grouped_gemm_data`
  - `at::cuda::getDeviceProperties`
  - `at::cuda::getCurrentDeviceProperties`
  - `at::cuda::detail`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
