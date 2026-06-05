# AsyncMM.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/cuda/AsyncMM.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for async mm in the c10d CUDA helpers. Representative routines include `async_input_mm_impl`, `constexpr`, `TORCH_CHECK`, `C10_CUDA_KERNEL_LAUNCH_CHECK`, `__VA_ARGS__`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d CUDA 辅助工具中提供async mm 的实现逻辑。 代表性例程包括 `async_input_mm_impl`、`constexpr`、`TORCH_CHECK`、`C10_CUDA_KERNEL_LAUNCH_CHECK`、`__VA_ARGS__`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
2: #include <ATen/Dispatch.h>
3: #include <ATen/core/Tensor.h>
4: #include <ATen/cuda/CUDAContext.h>
5: #include <ATen/cuda/nvrtc_stub/ATenNVRTC.h>
6: #include <c10/cuda/CUDAGuard.h>
7: 
8: // Two warnings in Cutlass included header files
9: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wset-but-not-used")
10: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-parameter")
11: C10_DIAGNOSTIC_PUSH_AND_IGNORED_IF_DEFINED("-Wunused-but-set-variable")
12: 
13: #if !defined(USE_ROCM) && !defined(_WIN32) && defined(CUDA_VERSION)
14: #define BUILD_ASYNC_MM_KERNEL
15: #endif
16: 
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；包含面向 CUDA 的声明、内核或启动流程。

### Lines 17-32 / 第 17-32 行

```cpp
17: #if defined(BUILD_ASYNC_MM_KERNEL)
18: 
19: #include <cutlass/core_io.h>
20: #include <cutlass/cutlass.h>
21: #include <cutlass/gemm/device/gemm.h>
22: #include <cutlass/half.h>
23: #include <cutlass/numeric_types.h>
24: #include <cutlass/trace.h>
25: #include <cutlass/util/host_tensor.h>
26: #include <cute/tensor.hpp>
27: 
28: #include <cutlass/version.h>
29: #include <cutlass/gemm/collective/collective_builder.hpp>
30: #include <cutlass/gemm/device/gemm_universal_adapter.h>
31: #include <cutlass/epilogue/collective/collective_builder.hpp>
32: 
```

- EN: Lines 17-32 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 17-32 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端；包含面向 CUDA 的声明、内核或启动流程。

### Lines 33-48 / 第 33-48 行

```cpp
33: #include <cute/atom/mma_atom.hpp>
34: #include <cutlass/gemm/dispatch_policy.hpp>
35: #include <cutlass/gemm/kernel/gemm_universal.hpp>
36: #include <cutlass/util/packed_stride.hpp>
37: 
38: #include <torch/csrc/distributed/c10d/cuda/cutlass/gemm/kernel/persistent_async_input_scheduler.cuh>
39: 
40: C10_DIAGNOSTIC_POP()
41: C10_DIAGNOSTIC_POP()
42: C10_DIAGNOSTIC_POP()
43: 
44: namespace {
45: 
46: using namespace cute;
47: 
48: template <typename LayoutB, typename TileShape_MNK, typename ClusterShape_MNK>
```

- EN: Lines 33-48 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 33-48 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

### Lines 49-64 / 第 49-64 行

```cpp
49: at::Tensor async_input_mm_impl(
50:     at::Tensor a,
51:     at::Tensor b,
52:     at::Tensor a_chunk_signals,
53:     int64_t a_chunk_pivot,
54:     at::Tensor out) {
55:   c10::cuda::CUDAGuard guard(a.device());
56: 
57:   using ElementA = cutlass::bfloat16_t;
58:   using LayoutA = cutlass::layout::RowMajor;
59:   constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
60: 
61:   using ElementB = cutlass::bfloat16_t;
62:   constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;
63: 
64:   using ElementC = cutlass::bfloat16_t;
```

- EN: Lines 49-64 introduces executable logic in routines such as `async_input_mm_impl`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-64 行在 `async_input_mm_impl` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 65-80 / 第 65-80 行

```cpp
65:   using LayoutC = cutlass::layout::RowMajor;
66:   constexpr int AlignmentC = 128 / cutlass::sizeof_bits<ElementC>::value;
67: 
68:   using ElementAccumulator = float;
69: 
70:   using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedCooperative;
71:   using EpilogueSchedule = cutlass::epilogue::TmaWarpSpecializedCooperative;
72: 
73:   using CollectiveEpilogue =
74:       typename cutlass::epilogue::collective::CollectiveBuilder<
75:           cutlass::arch::Sm90,
76:           cutlass::arch::OpClassTensorOp,
77:           TileShape_MNK,
78:           ClusterShape_MNK,
79:           cutlass::epilogue::collective::EpilogueTileAuto,
80:           ElementAccumulator,
```

- EN: Lines 65-80 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 65-80 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-96 / 第 81-96 行

```cpp
81:           ElementAccumulator,
82:           void, // Indicate there is no beta scaling to save register
83:           LayoutC,
84:           AlignmentC,
85:           ElementC,
86:           LayoutC,
87:           AlignmentC,
88:           EpilogueSchedule>::CollectiveOp;
89: 
90:   using CollectiveMainloop =
91:       typename cutlass::gemm::collective::CollectiveBuilder<
92:           cutlass::arch::Sm90,
93:           cutlass::arch::OpClassTensorOp,
94:           ElementA,
95:           LayoutA,
96:           AlignmentA,
```

- EN: Lines 81-96 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-96 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-112 / 第 97-112 行

```cpp
97:           ElementB,
98:           LayoutB,
99:           AlignmentB,
100:           ElementAccumulator,
101:           TileShape_MNK,
102:           ClusterShape_MNK,
103:           cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
104:               sizeof(typename CollectiveEpilogue::SharedStorage))>,
105:           KernelSchedule>::CollectiveOp;
106: 
107:   using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
108:       Shape<int, int, int>,
109:       CollectiveMainloop,
110:       CollectiveEpilogue,
111:       cutlass::gemm::PersistentAsyncInputScheduler<KernelSchedule>>;
112:   using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
```

- EN: Lines 97-112 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-112 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 113-128 / 第 113-128 行

```cpp
113: 
114:   using StrideA = typename Gemm::GemmKernel::StrideA;
115:   using StrideB = typename Gemm::GemmKernel::StrideB;
116:   using StrideC = typename Gemm::GemmKernel::StrideC;
117: 
118:   TORCH_CHECK(a.dim() == 2 && b.dim() == 2 && out.dim() == 2);
119:   TORCH_CHECK(a.is_contiguous() && out.is_contiguous());
120: 
121:   if constexpr (std::is_same_v<LayoutB, cutlass::layout::RowMajor>) {
122:     TORCH_CHECK(b.is_contiguous());
123:   } else {
124:     TORCH_CHECK(b.stride(1) == b.size(0));
125:     TORCH_CHECK(b.stride(0) == 1);
126:   }
127:   TORCH_CHECK_EQ(a.scalar_type(), at::kBFloat16);
128:   TORCH_CHECK_EQ(b.scalar_type(), at::kBFloat16);
```

- EN: Lines 113-128 introduces executable logic in routines such as `constexpr`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 113-128 行在 `constexpr` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 129-144 / 第 129-144 行

```cpp
129:   TORCH_CHECK_EQ(out.scalar_type(), at::kBFloat16);
130: 
131:   int M = static_cast<int>(a.sizes()[0]);
132:   int N = static_cast<int>(b.sizes()[1]);
133:   int K = static_cast<int>(a.sizes()[1]);
134:   TORCH_CHECK_EQ(b.sizes()[0], K);
135:   TORCH_CHECK_EQ(out.sizes()[0], M);
136:   TORCH_CHECK_EQ(out.sizes()[1], N);
137: 
138:   auto stride_A = cutlass::make_cute_packed_stride(StrideA{}, {M, K, 1});
139:   auto stride_B = cutlass::make_cute_packed_stride(StrideB{}, {N, K, 1});
140:   auto stride_C = cutlass::make_cute_packed_stride(StrideC{}, {M, N, 1});
141: 
142:   Gemm gemm;
143: 
144:   typename Gemm::Arguments arguments{
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 145-160 / 第 145-160 行

```cpp
145:       cutlass::gemm::GemmUniversalMode::kGemm,
146:       {M, N, K},
147:       {
148:           reinterpret_cast<ElementA*>(a.data_ptr<at::BFloat16>()),
149:           stride_A,
150:           reinterpret_cast<ElementB*>(b.data_ptr<at::BFloat16>()),
151:           stride_B,
152:       },
153:       {{},
154:        nullptr,
155:        stride_C,
156:        reinterpret_cast<ElementC*>(out.data_ptr<at::BFloat16>()),
157:        stride_C},
158:   };
159: 
160:   TORCH_CHECK(
```

- EN: Lines 145-160 contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161:       a_chunk_signals.dim() == 1,
162:       "async_input_mm: `a_chunk_signals` must be a 1D tensor.");
163:   size_t num_chunks_M = a_chunk_signals.numel();
164: 
165:   TORCH_CHECK(
166:       M % num_chunks_M == 0,
167:       "async_input_mm: `a.shape(0)` must be an integer multiple of `a_chunk_signals.numel()`");
168:   size_t chunk_size_M = M / num_chunks_M;
169:   size_t tile_size_M = cute::get<0>(TileShape_MNK{});
170: 
171:   TORCH_CHECK(chunk_size_M % tile_size_M == 0);
172: 
173:   // We want to swizzle within a chunk
174:   arguments.scheduler.max_swizzle_size = chunk_size_M / tile_size_M;
175: 
176:   // PersistentAsyncInputScheduler currently only supports rastering along N
```

- EN: Lines 161-176 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-176 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 177-192 / 第 177-192 行

```cpp
177:   using RasterOrderOptions = typename cutlass::gemm::kernel::detail::
178:       PersistentTileSchedulerSm90::RasterOrderOptions;
179:   arguments.scheduler.raster_order = RasterOrderOptions::AlongN;
180: 
181:   // Convert the number of chunks to pivot to the number of m idx to pivot
182:   arguments.scheduler.tile_idx_pivot_m =
183:       a_chunk_pivot * (chunk_size_M / tile_size_M);
184:   arguments.scheduler.tiles_per_chunk_m = chunk_size_M / tile_size_M;
185:   arguments.scheduler.chunk_signals = a_chunk_signals.data_ptr<uint32_t>();
186: 
187:   size_t workspace_size = Gemm::get_workspace_size(arguments);
188:   cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
189: 
190:   TORCH_CHECK(gemm.can_implement(arguments) == cutlass::Status::kSuccess);
191:   TORCH_CHECK(
192:       gemm.initialize(arguments, workspace.get()) == cutlass::Status::kSuccess);
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 193-208 / 第 193-208 行

```cpp
193:   TORCH_CHECK(
194:       gemm(at::cuda::getCurrentCUDAStream()) == cutlass::Status::kSuccess);
195: 
196:   C10_CUDA_KERNEL_LAUNCH_CHECK();
197:   return out;
198: }
199: 
200: } // namespace
201: 
202: #endif
203: 
204: namespace c10d::cuda::detail {
205: 
206: #define DISPATCH_LAYOUT_B(is_b_row_major, ...)    \
207:   if (is_b_row_major) {                           \
208:     using LayoutB = cutlass::layout::RowMajor;    \
```

- EN: Lines 193-208 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`.
- CN: 第 193-208 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑。

### Lines 209-224 / 第 209-224 行

```cpp
209:     __VA_ARGS__();                                \
210:   } else {                                        \
211:     using LayoutB = cutlass::layout::ColumnMajor; \
212:     __VA_ARGS__();                                \
213:   }
214: 
215: at::Tensor async_input_mm_out(
216:     at::Tensor a,
217:     at::Tensor b,
218:     at::Tensor a_chunk_signals,
219:     int64_t a_chunk_pivot,
220:     at::Tensor out) {
221:   TORCH_CHECK(
222:       a.dim() == 2 && b.dim() == 2 && out.dim() == 2,
223:       "async_input_mm: `a`, `b` and `out` must be matrices")
224:   TORCH_CHECK(
```

- EN: Lines 209-224 introduces executable logic in routines such as `__VA_ARGS__`, `async_input_mm_out`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 209-224 行在 `__VA_ARGS__`、`async_input_mm_out` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 225-240 / 第 225-240 行

```cpp
225:       a.is_contiguous() && out.is_contiguous(),
226:       "async_input_mm: `a` and `out` must be in row-major layout");
227: 
228:   if (!b.is_contiguous()) {
229:     TORCH_CHECK(b.stride(1) == b.size(0));
230:     TORCH_CHECK(b.stride(0) == 1);
231:   }
232:   TORCH_CHECK_EQ(a.scalar_type(), at::kBFloat16);
233:   TORCH_CHECK_EQ(b.scalar_type(), at::kBFloat16);
234:   TORCH_CHECK_EQ(out.scalar_type(), at::kBFloat16);
235: 
236:   int64_t M = a.sizes()[0];
237:   int64_t N = b.sizes()[1];
238:   int64_t K = a.sizes()[1];
239:   TORCH_CHECK_EQ(b.sizes()[0], K);
240:   TORCH_CHECK_EQ(out.sizes()[0], M);
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-256 / 第 241-256 行

```cpp
241:   TORCH_CHECK_EQ(out.sizes()[1], N);
242: 
243: #if defined(BUILD_ASYNC_MM_KERNEL)
244:   const bool is_b_row_major = b.is_contiguous();
245:   DISPATCH_LAYOUT_B(is_b_row_major, [&]() {
246:     // TODO(yifu): tuning
247:     async_input_mm_impl<LayoutB, Shape<_128, _256, _64>, Shape<_2, _1, _1>>(
248:         a, b, a_chunk_signals, a_chunk_pivot, out);
249:   });
250: #else
251:   TORCH_CHECK(
252:       false, "async_input_mm is not currently supported on your device");
253: #endif
254:   return out;
255: }
256: 
```

- EN: Lines 241-256 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-256 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 257-272 / 第 257-272 行

```cpp
257: at::Tensor async_input_mm(
258:     at::Tensor a,
259:     at::Tensor b,
260:     at::Tensor a_chunk_signals,
261:     int64_t a_chunk_pivot) {
262:   TORCH_CHECK(
263:       a.dim() == 2 && b.dim() == 2,
264:       "async_input_mm: `a`, `b` and `out` must all be a matrix")
265: 
266:   int64_t M = a.sizes()[0];
267:   int64_t N = b.sizes()[1];
268:   auto out = a.new_empty({M, N});
269:   return async_input_mm_out(a, b, a_chunk_signals, a_chunk_pivot, out);
270: }
271: 
272: } // namespace c10d::cuda::detail
```

- EN: Lines 257-272 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `async_input_mm`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 257-272 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `async_input_mm` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: c10d CUDA helpers.
- CN: 子系统：c10d CUDA 辅助工具。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `async_input_mm_impl`, `constexpr`, `TORCH_CHECK`, `C10_CUDA_KERNEL_LAUNCH_CHECK`, `__VA_ARGS__`, `async_input_mm_out`
- CN: 核心符号：`async_input_mm_impl`、`constexpr`、`TORCH_CHECK`、`C10_CUDA_KERNEL_LAUNCH_CHECK`、`__VA_ARGS__`、`async_input_mm_out`
- EN: Notable themes: CUDA paths, collective communication logic.
- CN: 值得关注的主题：CUDA 路径、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/cuda/cutlass/gemm/kernel/persistent_async_input_scheduler.cuh`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/Dispatch.h`, `ATen/core/Tensor.h`, `ATen/cuda/CUDAContext.h`, `ATen/cuda/nvrtc_stub/ATenNVRTC.h`, `c10/cuda/CUDAGuard.h`
- External or system headers / 外部或系统头文件: `cutlass/core_io.h`, `cutlass/cutlass.h`, `cutlass/gemm/device/gemm.h`, `cutlass/half.h`, `cutlass/numeric_types.h`, `cutlass/trace.h`, `cutlass/util/host_tensor.h`, `cute/tensor.hpp`, `cutlass/version.h`, `cutlass/gemm/collective/collective_builder.hpp`, `cutlass/gemm/device/gemm_universal_adapter.h`, `cutlass/epilogue/collective/collective_builder.hpp`, ... (+4 more)
- Local symbols / 本地符号: `async_input_mm_impl`, `constexpr`, `TORCH_CHECK`, `C10_CUDA_KERNEL_LAUNCH_CHECK`, `__VA_ARGS__`, `async_input_mm_out`, `async_input_mm`