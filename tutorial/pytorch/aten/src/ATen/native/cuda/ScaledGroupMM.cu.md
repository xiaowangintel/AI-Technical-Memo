# ScaledGroupMM.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/ScaledGroupMM.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ceildiv`, `round_up_to_nearest_multiple`, `f8f8bf16_grouped_gemm_impl_sm90`, `dispatch_fp8_grouped_gemm_on_tile_size`.
- 用途（中文）: 实现与 `ceildiv`, `round_up_to_nearest_multiple`, `f8f8bf16_grouped_gemm_impl_sm90`, `dispatch_fp8_grouped_gemm_on_tile_size` 相关的 CUDA / 原生内核逻辑。

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

### Lines 10-10
```cpp
  10: // Two warnings in Cutlass included header files
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 11-13
```cpp
  11: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wset-but-not-used")
  12: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-parameter")
  13: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-variable")
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 15-17
```cpp
  15: // Determine if the architecture supports rowwise scaled mm
  16: // Currently failing on windows with:
  17: // https://github.com/NVIDIA/cutlass/issues/1571
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 18-31
```cpp
  18: #if !defined(USE_ROCM) && !defined(_WIN32) && defined(CUDA_VERSION)
  19: 
  20: #define BUILD_ROWWISE_FP8_KERNEL
  21: #endif
  22: 
  23: #if defined(BUILD_ROWWISE_FP8_KERNEL)
  24: 
  25: #include <ATen/native/cuda/GroupMMCommon.cuh>
  26: 
  27: #include <cute/tensor.hpp>
  28: #include <cutlass/core_io.h>
  29: #include <cutlass/cutlass.h>
  30: #include <cutlass/gemm/device/gemm.h>
  31: #include <cutlass/numeric_types.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/cuda/GroupMMCommon.cuh>`, `<cute/tensor.hpp>`, `<cutlass/core_io.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/cuda/GroupMMCommon.cuh>`, `<cute/tensor.hpp>`, `<cutlass/core_io.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 32-45
```cpp
  32: #include <cutlass/trace.h>
  33: #include <cutlass/version.h>
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
  45: #include <cutlass/util/packed_stride.hpp>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<cutlass/trace.h>`, `<cutlass/version.h>`, `<cutlass/epilogue/collective/collective_builder.hpp>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<cutlass/trace.h>`, `<cutlass/version.h>`, `<cutlass/epilogue/collective/collective_builder.hpp>`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 47-49
```cpp
  47: C10_DIAGNOSTIC_POP()
  48: C10_DIAGNOSTIC_POP()
  49: C10_DIAGNOSTIC_POP()
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 51-72
```cpp
  51: namespace {
  52: 
  53: using Strides = at::cuda::detail::Strides;
  54: 
  55: using DtypeScale = float;
  56: using DtypeAccum = float;
  57: using DtypeEpilogue = float;
  58: using DtypeOutput = cutlass::bfloat16_t;
  59: 
  60: using Multiply = cutlass::epilogue::fusion::Sm90Compute<
  61:     cutlass::multiplies,
  62:     DtypeEpilogue,
  63:     DtypeEpilogue,
  64:     cutlass::FloatRoundStyle::round_to_nearest>;
  65: 
  66: using Add = cutlass::epilogue::fusion::Sm90Compute<
  67:     cutlass::plus,
  68:     DtypeEpilogue,
  69:     DtypeEpilogue,
  70:     cutlass::FloatRoundStyle::round_to_nearest>;
  71: 
  72: using Cast = cutlass::epilogue::fusion::Sm90Compute<
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 73-76
```cpp
  73:     cutlass::epilogue::thread::Identity,
  74:     DtypeOutput,
  75:     DtypeEpilogue,
  76:     cutlass::FloatRoundStyle::round_to_nearest>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 78-80
```cpp
  78: using ProblemShape = cutlass::gemm::GroupProblemShape<
  79:     cute::Shape<int32_t, int32_t, int32_t>>; // <M,N,K> per
  80:                                              // group
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 82-103
```cpp
  82: template <
  83:     bool FastAccum,
  84:     bool PONG,
  85:     typename TB_M,
  86:     typename TB_N,
  87:     typename TB_K>
  88: struct Schedule {
  89:   using FastCooperativeSchedule =
  90:       cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8FastAccum;
  91:   using CooperativeSchedule =
  92:       cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperative;
  93:   using FastPongSchedule =
  94:       cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8FastAccum;
  95:   using PongSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpong;
  96:   using CooperativeEpilogueSchedule =
  97:       cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  98:   using PongEpilogueSchedule =
  99:       cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
 100:   using KernelSchedule = cute::conditional_t<
 101:       PONG,
 102:       cute::conditional_t<FastAccum, FastPongSchedule, PongSchedule>,
 103:       cute::conditional_t<
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 104-111
```cpp
 104:           FastAccum,
 105:           FastCooperativeSchedule,
 106:           CooperativeSchedule>>;
 107:   using EpilogueSchedule = cute::
 108:       conditional_t<PONG, PongEpilogueSchedule, CooperativeEpilogueSchedule>;
 109:   using TileShape = cute::Shape<TB_M, TB_N, TB_K>;
 110:   using ClusterShape = cute::Shape<cute::_2, cute::_2, cute::_1>;
 111: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 113-115
```cpp
 113: int ceildiv(int a, int b) {
 114:   return (a + b - 1) / b;
 115: }
```
- EN: This block defines or continues the implementation of `ceildiv`.
- CN: 该代码块定义或继续实现 `ceildiv`。

### Lines 117-119
```cpp
 117: int round_up_to_nearest_multiple(int a, int b) {
 118:   return ceildiv(a, b) * b;
 119: }
```
- EN: This block defines or continues the implementation of `round_up_to_nearest_multiple`.
- CN: 该代码块定义或继续实现 `round_up_to_nearest_multiple`。

### Lines 121-134
```cpp
 121: template <
 122:     typename FastAccum,
 123:     typename BiasType,
 124:     typename Pong,
 125:     typename TB_M,
 126:     typename TB_N,
 127:     typename TB_K>
 128: void f8f8bf16_grouped_gemm_impl_sm90(
 129:     at::Tensor mat_a, // FP8
 130:     at::Tensor mat_b, // FP8
 131:     at::Tensor scale_a, // FP32
 132:     at::Tensor scale_b, // FP32
 133:     std::optional<at::Tensor> offs,
 134:     std::optional<at::Tensor> bias, // BF16
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 135-156
```cpp
 135:     bool use_fast_accum,
 136:     at::Tensor& out) {
 137:   using DtypeA = cutlass::float_e4m3_t;
 138:   using DtypeB = cutlass::float_e4m3_t;
 139:   using DtypeOutput = cutlass::bfloat16_t;
 140:   using LayoutA = cutlass::layout::RowMajor;
 141:   constexpr int AlignmentA = 16 / sizeof(DtypeA);
 142:   using LayoutB = cutlass::layout::ColumnMajor;
 143:   constexpr int AlignmentB = 16 / sizeof(DtypeB);
 144:   using LayoutOutput = cutlass::layout::RowMajor;
 145:   constexpr int AlignmentOutput = 16 / sizeof(DtypeOutput);
 146: 
 147:   // Tag indicating the minimum SM that supports the intended feature
 148:   using ArchTag = cutlass::arch::Sm90;
 149:   using OperatorClass = cutlass::arch::OpClassTensorOp;
 150: 
 151:   using TileShape =
 152:       typename Schedule<FastAccum::value, Pong::value, TB_M, TB_N, TB_K>::
 153:           TileShape;
 154:   using ClusterShape =
 155:       typename Schedule<FastAccum::value, Pong::value, TB_M, TB_N, TB_K>::
 156:           ClusterShape;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 157-168
```cpp
 157:   using KernelSchedule =
 158:       typename Schedule<FastAccum::value, Pong::value, TB_M, TB_N, TB_K>::
 159:           KernelSchedule;
 160:   using EpilogueSchedule =
 161:       typename Schedule<FastAccum::value, Pong::value, TB_M, TB_N, TB_K>::
 162:           EpilogueSchedule;
 163:   using ScaleA = cutlass::epilogue::fusion::Sm90ColBroadcast<
 164:       0,
 165:       TileShape,
 166:       DtypeScale*,
 167:       DtypeScale,
 168:       cute::Stride<cute::Int<1>, cute::Int<0>, cute::Int<0>>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 170-175
```cpp
 170:   using ScaleB = cutlass::epilogue::fusion::Sm90RowBroadcast<
 171:       0,
 172:       TileShape,
 173:       DtypeScale*,
 174:       DtypeScale,
 175:       cute::Stride<cute::Int<0>, cute::Int<1>, cute::Int<0>>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 177-177
```cpp
 177:   using Accum = cutlass::epilogue::fusion::Sm90AccFetch;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 179-182
```cpp
 179:   using AccumScale = cutlass::epilogue::fusion::Sm90EVT<
 180:       Multiply,
 181:       ScaleB,
 182:       cutlass::epilogue::fusion::Sm90EVT<Multiply, ScaleA, Accum>>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 184-184
```cpp
 184:   using EpilogueEVT = cutlass::epilogue::fusion::Sm90EVT<Cast, AccumScale>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 186-199
```cpp
 186:   using CollectiveEpilogue =
 187:       typename cutlass::epilogue::collective::CollectiveBuilder<
 188:           ArchTag,
 189:           OperatorClass,
 190:           TileShape,
 191:           ClusterShape,
 192:           cutlass::epilogue::collective::EpilogueTileAuto,
 193:           DtypeAccum,
 194:           DtypeAccum,
 195:           void, // Indicate there is no beta scaling to save register
 196:           LayoutOutput*,
 197:           AlignmentOutput,
 198:           DtypeOutput,
 199:           LayoutOutput*,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 200-202
```cpp
 200:           AlignmentOutput,
 201:           EpilogueSchedule,
 202:           EpilogueEVT>::CollectiveOp;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 204-217
```cpp
 204:   using CollectiveMainloop =
 205:       typename cutlass::gemm::collective::CollectiveBuilder<
 206:           ArchTag,
 207:           OperatorClass,
 208:           DtypeA,
 209:           LayoutA*,
 210:           AlignmentA,
 211:           DtypeB,
 212:           LayoutB*,
 213:           AlignmentB,
 214:           DtypeAccum,
 215:           TileShape,
 216:           ClusterShape,
 217:           cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 218-221
```cpp
 218:               sizeof(typename CollectiveEpilogue::SharedStorage))>,
 219:           KernelSchedule>::CollectiveOp;
 220:   using GemmKernel = cutlass::gemm::kernel::
 221:       GemmUniversal<ProblemShape, CollectiveMainloop, CollectiveEpilogue>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 223-226
```cpp
 223:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
 224:   using StrideA = typename Gemm::GemmKernel::InternalStrideA;
 225:   using StrideB = typename Gemm::GemmKernel::InternalStrideB;
 226:   using StrideOutput = typename Gemm::GemmKernel::InternalStrideD;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 228-228
```cpp
 228:   int32_t M, N, K, group_count;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 230-232
```cpp
 230:   M = mat_a.size(-2);
 231:   K = mat_a.size(-1);
 232:   N = mat_b.size(-1);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 234-247
```cpp
 234:   if (mat_a.dim() == 2 && mat_b.dim() == 2) {
 235:     // if both inputs are ragged, K is dynamic, M and N come from inputs
 236:     group_count = offs->size(0);
 237:     K = -1;
 238:   } else if (mat_a.dim() == 2) {
 239:     group_count = mat_b.size(0);
 240:     M = -1;
 241:   } else if (mat_b.dim() == 2) {
 242:     group_count = mat_a.size(0);
 243:     N = -1;
 244:   } else {
 245:     // regular bmm
 246:     group_count = mat_a.size(0);
 247:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 249-249
```cpp
 249:   TORCH_CHECK(group_count < 1024, "Can't process more than 1024 groups");
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 251-252
```cpp
 251:   const int64_t problem_shape_size =
 252:       group_count * ((int64_t)sizeof(ProblemShape::UnderlyingProblemShape));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 254-254
```cpp
 254:   const int64_t stride_size = 3 * group_count * ((int64_t)sizeof(StrideA));
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 256-259
```cpp
 256:   // dummy tmas are created based on these pointer-to-pointers
 257:   // the actual values are never used, they are replaced
 258:   // by real addresses, but for dummy tma creation to succeed
 259:   // due to bug in cuda < 12.4 the pointers have to be aligned to 128 bits
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 260-264
```cpp
 260:   const int group_alignment = 16 / sizeof(void*);
 261:   const int aligned_group_count =
 262:       round_up_to_nearest_multiple(group_count, group_alignment);
 263:   int64_t input_args_size = aligned_group_count * 5 * sizeof(void*) +
 264:       problem_shape_size + stride_size;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 266-279
```cpp
 266:   auto& allocator = *c10::cuda::CUDACachingAllocator::get();
 267:   auto input_buf = allocator.allocate(input_args_size);
 268:   void* buf_ptr = input_buf.get();
 269:   DtypeA** inputA_ptrs = reinterpret_cast<DtypeA**>(buf_ptr);
 270:   DtypeB** inputB_ptrs =
 271:       reinterpret_cast<DtypeB**>(inputA_ptrs + aligned_group_count);
 272:   DtypeOutput** output_ptrs =
 273:       reinterpret_cast<DtypeOutput**>(inputB_ptrs + aligned_group_count);
 274:   DtypeScale** inputA_scale_ptrs =
 275:       reinterpret_cast<DtypeScale**>(output_ptrs + aligned_group_count);
 276:   DtypeScale** inputB_scale_ptrs =
 277:       reinterpret_cast<DtypeScale**>(inputA_scale_ptrs + aligned_group_count);
 278:   static_assert(
 279:       sizeof(StrideA) == 8, "expected StrideA to be 8 bytes for alignment");
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 280-287
```cpp
 280:   StrideA* stride_A =
 281:       reinterpret_cast<StrideA*>(inputB_scale_ptrs + aligned_group_count);
 282:   StrideB* stride_B = reinterpret_cast<StrideB*>(stride_A + group_count);
 283:   StrideOutput* stride_output =
 284:       reinterpret_cast<StrideOutput*>(stride_B + group_count);
 285:   ProblemShape::UnderlyingProblemShape* problem_sizes =
 286:       reinterpret_cast<ProblemShape::UnderlyingProblemShape*>(
 287:           stride_output + group_count);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 289-289
```cpp
 289:   auto stream = at::cuda::getCurrentCUDAStream().stream();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 291-295
```cpp
 291:   auto make_strides = [](at::IntArrayRef strides) -> Strides {
 292:     Strides out;
 293:     std::copy(strides.begin(), strides.end(), out.begin());
 294:     return out;
 295:   };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 297-301
```cpp
 297:   Strides tensor_StrideA = make_strides(mat_a.strides());
 298:   Strides tensor_StrideB = make_strides(mat_b.strides());
 299:   Strides tensor_StrideOutput = make_strides(out.strides());
 300:   Strides tensor_ShapeA = make_strides(mat_a.sizes());
 301:   Strides tensor_ShapeB = make_strides(mat_b.sizes());
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 303-304
```cpp
 303:   // scale stride will be used inside the kernel only if needed,
 304:   // so for 1d scales the "1" assigned here won't be used
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 305-306
```cpp
 305:   int64_t a_scale_stride = scale_a.stride(0);
 306:   int64_t b_scale_stride = scale_b.stride(0);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 308-321
```cpp
 308:   at::cuda::detail::prepare_grouped_gemm_data<<<1, group_count, 0, stream>>>(
 309:       reinterpret_cast<DtypeA*>(mat_a.data_ptr()),
 310:       reinterpret_cast<DtypeB*>(mat_b.data_ptr()),
 311:       reinterpret_cast<DtypeOutput*>(out.data_ptr()),
 312:       scale_a.data_ptr<DtypeScale>(),
 313:       scale_b.data_ptr<DtypeScale>(),
 314:       inputA_ptrs,
 315:       inputB_ptrs,
 316:       output_ptrs,
 317:       inputA_scale_ptrs,
 318:       inputB_scale_ptrs,
 319:       problem_sizes,
 320:       stride_A,
 321:       stride_B,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 322-333
```cpp
 322:       stride_output,
 323:       offs.has_value() ? offs->const_data_ptr<int32_t>() : nullptr,
 324:       M,
 325:       N,
 326:       K,
 327:       tensor_StrideA,
 328:       tensor_StrideB,
 329:       tensor_StrideOutput,
 330:       tensor_ShapeA,
 331:       tensor_ShapeB,
 332:       a_scale_stride,
 333:       b_scale_stride);
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 335-335
```cpp
 335:   C10_CUDA_KERNEL_LAUNCH_CHECK();
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 337-350
```cpp
 337:   //   auto buf_cpu = mat_a.new_empty(
 338:   //       input_args_size,
 339:   //       at::TensorOptions().dtype(at::kByte).device(at::kCPU));
 340:   //   AT_CUDA_CHECK(cudaMemcpy(
 341:   //       (char*)buf_cpu.data_ptr(),
 342:   //       buf_ptr,
 343:   //       input_args_size,
 344:   //       cudaMemcpyDeviceToHost));
 345:   //   char* buf_ptr_cpu = (char*)buf_cpu.data_ptr();
 346:   //   DtypeA** inputA_ptrs_h = reinterpret_cast<DtypeA**>(buf_ptr_cpu);
 347:   //   DtypeB** inputB_ptrs_h =
 348:   //       reinterpret_cast<DtypeB**>(inputA_ptrs_h + aligned_group_count);
 349:   //   DtypeOutput** output_ptrs_h =
 350:   //       reinterpret_cast<DtypeOutput**>(inputB_ptrs_h + aligned_group_count);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 351-364
```cpp
 351:   //   DtypeScale** inputA_scale_ptrs_h =
 352:   //       reinterpret_cast<DtypeScale**>(output_ptrs_h + aligned_group_count);
 353:   //   DtypeScale** inputB_scale_ptrs_h =
 354:   //       reinterpret_cast<DtypeScale**>(inputA_scale_ptrs_h +
 355:   //       aligned_group_count);
 356:   //   StrideA* stride_A_h =
 357:   //       reinterpret_cast<StrideA*>(inputB_scale_ptrs_h +
 358:   //       aligned_group_count);
 359:   //   StrideB* stride_B_h = reinterpret_cast<StrideB*>(stride_A_h +
 360:   //   group_count); StrideOutput* stride_output_h =
 361:   //       reinterpret_cast<StrideOutput*>(stride_B_h + group_count);
 362:   //   ProblemShape::UnderlyingProblemShape* problem_sizes_h =
 363:   //       reinterpret_cast<ProblemShape::UnderlyingProblemShape*>(
 364:   //           stride_output_h + group_count);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 365-378
```cpp
 365: 
 366:   //   std::cout << "PTRS " << mat_a.data_ptr() << ' ' << mat_b.data_ptr() << "
 367:   //   "
 368:   //             << out.data_ptr() << ' ' << scale_a.data_ptr() << ' '
 369:   //             << scale_b.data_ptr() << "\n";
 370:   //   for (int i = 0; i < group_count; i++) {
 371:   //     std::cout << "A " << (void*)inputA_ptrs_h[i] << "\n";
 372:   //     std::cout << "B " << (void*)inputB_ptrs_h[i] << "\n";
 373:   //     std::cout << "O " << (void*)output_ptrs_h[i] << "\n";
 374:   //     std::cout << "A_scale " << (void*)inputA_scale_ptrs_h[i] << "\n";
 375:   //     std::cout << "B_scale " << (void*)inputB_scale_ptrs_h[i] << "\n";
 376:   //     std::cout << "sizes " << problem_sizes_h[i] << "\n";
 377:   //     std::cout << "strideA" << stride_A_h[i] << "\n";
 378:   //     std::cout << "strideB" << stride_B_h[i] << "\n";
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 379-383
```cpp
 379:   //     std::cout << "stride_output" << stride_output_h[i] << "\n";
 380:   //   }
 381:   //   int device_id = 0;
 382:   //   cutlass::KernelHardwareInfo kernel_hw_info =
 383:   //   cutlass::KernelHardwareInfo::make_kernel_hardware_info<Gemm::GemmKernel>(device_id);
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 385-396
```cpp
 385:   typename Gemm::Arguments arguments{
 386:       cutlass::gemm::GemmUniversalMode::kGrouped,
 387:       {group_count, problem_sizes, nullptr},
 388:       {(const DtypeA**)inputA_ptrs,
 389:        stride_A,
 390:        (const DtypeB**)inputB_ptrs,
 391:        stride_B},
 392:       {{{{inputB_scale_ptrs}, {{inputA_scale_ptrs}, {}, {}}, {}}, {}},
 393:        nullptr,
 394:        stride_output,
 395:        output_ptrs,
 396:        stride_output}};
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 398-403
```cpp
 398:   int sm_count =
 399:       at::cuda::getDeviceProperties(out.device().index())->multiProcessorCount;
 400:   if (at::globalContext()._SMCarveout_EXPERIMENTAL().has_value()) {
 401:     sm_count -= at::globalContext()._SMCarveout_EXPERIMENTAL().value();
 402:   }
 403:   arguments.hw_info.sm_count = sm_count;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 405-418
```cpp
 405:   size_t workspace_size = Gemm::get_workspace_size(arguments);
 406:   auto workspace = allocator.allocate(workspace_size);
 407:   Gemm gemm;
 408:   TORCH_CHECK(
 409:       gemm.can_implement(arguments) == cutlass::Status::kSuccess,
 410:       "cutlass cannot implement");
 411:   TORCH_CHECK(
 412:       gemm.initialize(arguments, workspace.get()) == cutlass::Status::kSuccess,
 413:       "cutlass cannot initialize");
 414:   auto status = gemm(at::cuda::getCurrentCUDAStream());
 415:   TORCH_CHECK(
 416:       status == cutlass::Status::kSuccess,
 417:       "cutlass cannot run, error ",
 418:       int(status));
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 419-420
```cpp
 419:   C10_CUDA_KERNEL_LAUNCH_CHECK();
 420: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 422-443
```cpp
 422: template <typename FastAccum, typename BiasType>
 423: void dispatch_fp8_grouped_gemm_on_tile_size(
 424:     at::Tensor mat_a, // FP8
 425:     at::Tensor mat_b, // FP8
 426:     at::Tensor scale_a, // FP32
 427:     at::Tensor scale_b, // FP32
 428:     std::optional<at::Tensor> offs,
 429:     std::optional<at::Tensor> bias, // BF16
 430:     bool use_fast_accum,
 431:     at::Tensor& out) {
 432:   int32_t M, N, K, group_count;
 433: 
 434:   M = mat_a.size(-2);
 435:   K = mat_a.size(-1);
 436:   N = mat_b.size(-1);
 437: 
 438:   // below we assume that gemms are approx same size
 439:   if (mat_a.dim() == 2 && mat_b.dim() == 2) {
 440:     // if both inputs are ragged, K is dynamic, M and N come from inputs
 441:     group_count = offs->size(0);
 442:     K = K / group_count;
 443:   } else if (mat_a.dim() == 2) {
```
- EN: This block defines or continues the implementation of `dispatch_fp8_grouped_gemm_on_tile_size`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_grouped_gemm_on_tile_size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 444-457
```cpp
 444:     group_count = mat_b.size(0);
 445:     M = M / group_count;
 446:   } else if (mat_b.dim() == 2) {
 447:     group_count = mat_a.size(0);
 448:     N = N / group_count;
 449:   }
 450:   bool large =
 451:       ((M >= 2048 && K >= 2048) || (M >= 2048 && N >= 2048) ||
 452:        (K >= 2048 && N >= 2048));
 453:   bool small = (M <= 128 || N <= 128);
 454:   if (small) {
 455:     f8f8bf16_grouped_gemm_impl_sm90<
 456:         FastAccum,
 457:         BiasType,
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 458-458
```cpp
 458:         /*Pong*/ std::true_type,
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 459-472
```cpp
 459:         cute::_64,
 460:         cute::_128,
 461:         cute::_128>(
 462:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 463:   } else if (large && FastAccum::value) {
 464:     f8f8bf16_grouped_gemm_impl_sm90<
 465:         FastAccum,
 466:         BiasType,
 467:         /*Pong*/ std::false_type,
 468:         cute::_256,
 469:         cute::_128,
 470:         cute::_128>(
 471:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 472:   } else if (large) { // use smaller tile for slow accum to avoid spilling
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 473-480
```cpp
 473:     f8f8bf16_grouped_gemm_impl_sm90<
 474:         FastAccum,
 475:         BiasType,
 476:         /*Pong*/ std::false_type,
 477:         cute::_128,
 478:         cute::_128,
 479:         cute::_128>(
 480:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 482-491
```cpp
 482:   } else
 483:     f8f8bf16_grouped_gemm_impl_sm90<
 484:         FastAccum,
 485:         BiasType,
 486:         /*Pong*/ std::false_type,
 487:         cute::_128,
 488:         cute::_256,
 489:         cute::_64>(
 490:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 491: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 493-510
```cpp
 493: template <typename BiasType>
 494: void dispatch_fp8_grouped_gemm_on_fast_accum(
 495:     at::Tensor mat_a, // FP8
 496:     at::Tensor mat_b, // FP8
 497:     at::Tensor scale_a, // FP32
 498:     at::Tensor scale_b, // FP32
 499:     std::optional<at::Tensor> offs,
 500:     std::optional<at::Tensor> bias, // BF16
 501:     bool use_fast_accum,
 502:     at::Tensor& out) {
 503:   if (use_fast_accum) {
 504:     dispatch_fp8_grouped_gemm_on_tile_size<std::true_type, BiasType>(
 505:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 506:   } else {
 507:     dispatch_fp8_grouped_gemm_on_tile_size<std::false_type, BiasType>(
 508:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 509:   }
 510: }
```
- EN: This block defines or continues the implementation of `dispatch_fp8_grouped_gemm_on_fast_accum`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_grouped_gemm_on_fast_accum`。

### Lines 512-528
```cpp
 512: void dispatch_fp8_grouped_gemm_on_bias_dtype(
 513:     at::Tensor mat_a, // FP8
 514:     at::Tensor mat_b, // FP8
 515:     at::Tensor scale_a, // FP32
 516:     at::Tensor scale_b, // FP32
 517:     std::optional<at::Tensor> offs,
 518:     std::optional<at::Tensor> bias, // BF16
 519:     bool use_fast_accum,
 520:     at::Tensor& out) {
 521:   if (bias.has_value() && bias->dtype() == at::kBFloat16) {
 522:     dispatch_fp8_grouped_gemm_on_fast_accum<cutlass::bfloat16_t>(
 523:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 524:   } else {
 525:     dispatch_fp8_grouped_gemm_on_fast_accum<float>(
 526:         mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 527:   }
 528: }
```
- EN: This block defines or continues the implementation of `dispatch_fp8_grouped_gemm_on_bias_dtype`.
- CN: 该代码块定义或继续实现 `dispatch_fp8_grouped_gemm_on_bias_dtype`。

### Lines 530-530
```cpp
 530: } // namespace
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 532-532
```cpp
 532: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 534-552
```cpp
 534: namespace at::cuda::detail {
 535: void f8f8bf16_grouped_mm(
 536:     at::Tensor mat_a, // FP8
 537:     at::Tensor mat_b, // FP8
 538:     at::Tensor scale_a, // FP32
 539:     at::Tensor scale_b, // FP32
 540:     std::optional<at::Tensor> offs,
 541:     std::optional<at::Tensor> bias, // BF16
 542:     bool use_fast_accum,
 543:     at::Tensor& out) {
 544: #if defined(BUILD_ROWWISE_FP8_KERNEL)
 545:   dispatch_fp8_grouped_gemm_on_bias_dtype(
 546:       mat_a, mat_b, scale_a, scale_b, offs, bias, use_fast_accum, out);
 547: #else
 548:   TORCH_CHECK(false, "grouped mm is not supported on your system");
 549: #endif
 550: }
 551: 
 552: } // namespace at::cuda::detail
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `f8f8bf16_grouped_mm`.
- CN: 该代码块定义或继续实现 `f8f8bf16_grouped_mm`。

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
  - `<ATen/native/cuda/GroupMMCommon.cuh>`
  - `<cute/tensor.hpp>`
  - `<cutlass/core_io.h>`
  - `<cutlass/cutlass.h>`
  - `<cutlass/gemm/device/gemm.h>`
- Runtime symbols / 运行时符号:
  - `nvrtc_stub`
  - `at::cuda::detail::Strides`
  - `at::cuda::getCurrentCUDAStream`
  - `at::cuda::detail::prepare_grouped_gemm_data`
  - `at::cuda::getDeviceProperties`
  - `at::cuda::detail`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
