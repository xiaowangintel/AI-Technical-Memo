# persistent_async_input_scheduler.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/cuda/cutlass/gemm/kernel/persistent_async_input_scheduler.cuh`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for persistent async input scheduler in the CUDA CUTLASS scheduling support for c10d. Top-of-file note: This file contains PersistentTileSchedulerSm90, a forked version of PersistentTileSchedulerSm90 that supports consuming asynchronous input. This tile scheduler introduces the fo... Key types include `KernelSchedule`, `PersistentAsyncInputScheduler`, `PersistentTileSchedulerSm90AsyncInputParams`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 的 CUDA CUTLASS 调度支持中提供persistent async input scheduler 的接口与类型声明。文件开头备注：This file contains PersistentTileSchedulerSm90, a forked version of PersistentTileSchedulerSm90 that supports consuming asynchronous input. This tile scheduler introduces the fo... 关键类型包括 `KernelSchedule`、`PersistentAsyncInputScheduler`、`PersistentTileSchedulerSm90AsyncInputParams`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
1: /**
2:  * This file contains PersistentTileSchedulerSm90, a forked version of PersistentTileSchedulerSm90
3:  * that supports consuming asynchronous input. This tile scheduler introduces the following arguments:
4:  *
5:  * - tiles_per_chunk_m – Specifies the size of an M chunk. Chunks are the granularity at which the
6:  *   asynchronous input becomes ready. It must be an integer multiple of the size of an M tile.
7:  *
8:  * - chunk_signals – chunk_signals[i] == 1 indicates that chunk i is ready. Before returning a work
9:  *   tile, get_current_work() waits for the signal to ensure that the corresponding chunk is ready.
10:  *
11:  * - tile_idx_pivot_m – After applying swizzling, apply `pivot(m) => (m + tile_idx_pivot_m) %
12:  *   tiles_m` to `m`. In a distributed setting, this allows different ranks to process different m
13:  *   indices at the same time, thus avoiding communication hotspots.
14:  *
15:  * Note that this scheduler currently only supports the KernelTmaWarpSpecializedCooperative kernel
16:  * schedule. This is enforced via the template argument KernelSchedule.
17:  *
18:  * Usage:
19:  *
20:  * using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
```

- EN: Lines 1-20 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-20 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 21-40 / 第 21-40 行

```cpp
21:  *    Shape<int, int, int, int>,
22:  *    CollectiveMainloop,
23:  *    CollectiveEpilogue,
24:  *    cutlass::gemm::PersistentAsyncInputScheduler<KernelSchedule>>;
25:  *
26:  * Unfortunately, the CRTP base class for tile schedulers (StaticPersistentTileScheduler) doesn't
27:  * provide enough flexibility for the required customization. We had to create a new tile scheduler
28:  * by copying PersistentTileSchedulerSm90 and StaticPersistentTileScheduler then customize on top of
29:  * it. In PersistentTileSchedulerSm90AsyncInput, we marked the customizations with "CUSTOM LOGIC BEGIN"
30:  * and "CUSTOM LOGIC END" comment blocks.
31:  */
32: 
33: #pragma once
34: #include <cutlass/gemm/kernel/static_tile_scheduler.hpp>
35: 
36: namespace {
37: 
38: __device__ __forceinline__ void wait_signal(uint32_t* addr) {
39:   int ready = *addr;
40:   while (!ready) {
```

- EN: Lines 21-40 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `wait_signal`.
- CN: 第 21-40 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `wait_signal` 等例程中引入具体执行逻辑。

### Lines 41-60 / 第 41-60 行

```cpp
41:     asm volatile("ld.volatile.global.b32 %0, [%1];"
42:                  : "=r"(ready)
43:                  : "l"(addr)
44:                  : "memory");
45: #if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ > 700)
46:     asm volatile("nanosleep.u32 20;");
47: #endif
48:   };
49: }
50: 
51: }
52: 
53: ////////////////////////////////////////////////////////////////////////////////
54: 
55: namespace cutlass::gemm {
56: 
57: ////////////////////////////////////////////////////////////////////////////////
58: 
59: template<
60:   class KernelSchedule,
```

- EN: Lines 41-60 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `KernelSchedule`.
- CN: 第 41-60 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `KernelSchedule` 等类型。

### Lines 61-80 / 第 61-80 行

```cpp
61:   typename = cute::enable_if_t<
62:     cute::is_same_v<KernelSchedule, cutlass::gemm::KernelTmaWarpSpecializedCooperative>>>
63: struct PersistentAsyncInputScheduler {};
64: 
65: ////////////////////////////////////////////////////////////////////////////////
66: 
67: } // namespace cutlass::gemm
68: 
69: ////////////////////////////////////////////////////////////////////////////////
70: 
71: ////////////////////////////////////////////////////////////////////////////////
72: 
73: namespace cutlass::gemm::kernel::detail {
74: 
75: ////////////////////////////////////////////////////////////////////////////////
76: 
77: 
78: class PersistentTileSchedulerSm90AsyncInputParams :
79:   public PersistentTileSchedulerSm90Params {
80: public:
```

- EN: Lines 61-80 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `PersistentAsyncInputScheduler`, `PersistentTileSchedulerSm90AsyncInputParams`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 61-80 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `PersistentAsyncInputScheduler`、`PersistentTileSchedulerSm90AsyncInputParams` 等类型；包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-100 / 第 81-100 行

```cpp
81:   int tile_idx_pivot_m;
82:   int tiles_per_chunk_m = 0;
83:   uint32_t* chunk_signals = nullptr;
84: };
85: 
86: class PersistentTileSchedulerSm90AsyncInput {
87: private:
88:   uint64_t current_work_linear_idx_;
89:   uint64_t total_grid_size_;
90:   // ==============================
91:   // CUSTOM LOGIC BEGIN
92:   // ==============================
93:   bool is_mainloop_producer_;
94:   // ==============================
95:   // CUSTOM LOGIC END
96:   // ==============================
97: 
98: public:
99:   struct WorkTileInfo {
100:     int32_t M_idx = 0;
```

- EN: Lines 81-100 declares or defines types such as `PersistentTileSchedulerSm90AsyncInput`, `WorkTileInfo`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-100 行声明或定义了 `PersistentTileSchedulerSm90AsyncInput`、`WorkTileInfo` 等类型；包含面向 CUDA 的声明、内核或启动流程。

### Lines 101-120 / 第 101-120 行

```cpp
101:     int32_t N_idx = 0;
102:     int32_t L_idx = 0;
103:     bool is_valid_tile = false;
104: 
105:     CUTLASS_HOST_DEVICE
106:     bool
107:     is_valid() const {
108:       return is_valid_tile;
109:     }
110: 
111:     CUTLASS_HOST_DEVICE
112:     static WorkTileInfo
113:     invalid_work_tile() {
114:       return {-1, -1, -1, false};
115:     }
116: 
117:     CUTLASS_HOST_DEVICE
118:     bool
119:     is_final_split(uint32_t k_tiles_per_output_tile) const {
120:       return true;
```

- EN: Lines 101-120 introduces executable logic in routines such as `is_valid`, `invalid_work_tile`, `is_final_split`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 101-120 行在 `is_valid`、`invalid_work_tile`、`is_final_split` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 121-140 / 第 121-140 行

```cpp
121:     }
122: 
123:     CUTLASS_HOST_DEVICE
124:     int32_t
125:     reduction_subtile_idx() const {
126:       return -1;
127:     }
128:   };
129: 
130:   // ==============================
131:   // CUSTOM LOGIC BEGIN
132:   // ==============================
133:   using Params = PersistentTileSchedulerSm90AsyncInputParams;
134:   // ==============================
135:   // CUSTOM LOGIC END
136:   // ==============================
137:   using RasterOrder = typename Params::RasterOrder;
138:   using RasterOrderOptions = typename Params::RasterOrderOptions;
139:   static constexpr bool IsDynamicPersistent = false;
140: 
```

- EN: Lines 121-140 introduces executable logic in routines such as `reduction_subtile_idx`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 121-140 行在 `reduction_subtile_idx` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 141-160 / 第 141-160 行

```cpp
141:   using Pipeline = PipelineEmpty;
142:   using PipelineStorage = typename Pipeline::SharedStorage;
143:   using ThrottlePipeline = PipelineEmpty;
144:   using ThrottlePipelineStorage = typename ThrottlePipeline::SharedStorage;
145: 
146:   struct CLCResponse {};
147: 
148:   class SharedStorage {
149:   public:
150:     CUTLASS_DEVICE PipelineStorage pipeline() { return PipelineStorage{}; }
151:     CUTLASS_DEVICE ThrottlePipelineStorage throttle_pipeline() { return ThrottlePipelineStorage{}; }
152:     CUTLASS_DEVICE CLCResponse* data() { return nullptr; }
153:   };
154: 
155: public:
156:   // ==============================
157:   // CUSTOM LOGIC BEGIN
158:   // ==============================
159:   struct Arguments {
160:     int max_swizzle_size;
```

- EN: Lines 141-160 declares or defines types such as `CLCResponse`, `SharedStorage`, `Arguments`; introduces executable logic in routines such as `pipeline`, `throttle_pipeline`, `data`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 141-160 行声明或定义了 `CLCResponse`、`SharedStorage`、`Arguments` 等类型；在 `pipeline`、`throttle_pipeline`、`data` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 161-180 / 第 161-180 行

```cpp
161:     RasterOrderOptions raster_order;
162: 
163:     // Async input specific
164:     int tile_idx_pivot_m;
165:     int tiles_per_chunk_m;
166:     uint32_t* chunk_signals;
167: 
168:     Arguments():
169:       max_swizzle_size(1),
170:       raster_order(RasterOrderOptions::Heuristic),
171:       tile_idx_pivot_m(0),
172:       tiles_per_chunk_m(0),
173:       chunk_signals(nullptr) {}
174:   // ==============================
175:   // CUSTOM LOGIC END
176:   // ==============================
177:   };
178: 
179:   template <class ProblemShapeMNKL, class TileShape, class ClusterShape>
180:   static Params
```

- EN: Lines 161-180 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 161-180 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 181-200 / 第 181-200 行

```cpp
181:   to_underlying_arguments(
182:       ProblemShapeMNKL problem_shape_mnkl,
183:       TileShape tile_shape,
184:       ClusterShape cluster_shape,
185:       [[maybe_unused]] KernelHardwareInfo const& hw_info,
186:       Arguments const& arguments,
187:       [[maybe_unused]] void* workspace=nullptr,
188:       [[maybe_unused]] const uint32_t epilogue_subtile = 1,
189:       [[maybe_unused]] uint32_t ktile_start_alignment_count = 1u) {
190: 
191:     // We only need the tile and cluster shape during scheduler setup, so let FTAD do the magic
192:     static_assert(cute::is_static<TileShape>::value);
193:     static_assert(cute::is_static<ClusterShape>::value);
194: 
195:     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape, cluster_shape);
196: 
197:     Params params;
198:     params.initialize(
199:       problem_blocks,
200:       to_gemm_coord(cluster_shape),
```

- EN: Lines 181-200 introduces executable logic in routines such as `to_underlying_arguments`, `static_assert`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 181-200 行在 `to_underlying_arguments`、`static_assert` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 201-220 / 第 201-220 行

```cpp
201:       hw_info,
202:       arguments.max_swizzle_size,
203:       arguments.raster_order
204:     );
205: 
206:     // ==============================
207:     // CUSTOM LOGIC BEGIN
208:     // ==============================
209:     params.tile_idx_pivot_m = arguments.tile_idx_pivot_m;
210:     params.tiles_per_chunk_m = arguments.tiles_per_chunk_m;
211:     params.chunk_signals = arguments.chunk_signals;
212:     // ==============================
213:     // CUSTOM LOGIC END
214:     // ==============================
215: 
216:     return params;
217:   }
218: 
219:   CUTLASS_HOST_DEVICE
220:   static bool
```

- EN: Lines 201-220 contains CUDA-oriented declarations, kernels, or launch plumbing; returns computed state or forwards results to the surrounding caller.
- CN: 第 201-220 行包含面向 CUDA 的声明、内核或启动流程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 221-240 / 第 221-240 行

```cpp
221:   can_implement(Arguments const& args) {
222:     return args.max_swizzle_size >= 1;
223:   }
224: 
225:   CUTLASS_HOST_DEVICE
226:   PersistentTileSchedulerSm90AsyncInput() { }
227: 
228:   CUTLASS_DEVICE explicit PersistentTileSchedulerSm90AsyncInput(Params const& params_) : scheduler_params(params_) {
229:     // MSVC requires protecting use of CUDA-specific nonstandard syntax,
230:     // like blockIdx and gridDim, with __CUDA_ARCH__.
231: #if defined(__CUDA_ARCH__)
232:     if (params_.raster_order_ == RasterOrder::AlongN) {
233:       current_work_linear_idx_ = uint64_t(blockIdx.x) + uint64_t(blockIdx.y) * uint64_t(gridDim.x);
234:     }
235:     else {
236:       current_work_linear_idx_ = uint64_t(blockIdx.x) * uint64_t(gridDim.y) + uint64_t(blockIdx.y);
237:     }
238: 
239:     // ==============================
240:     // CUSTOM LOGIC BEGIN
```

- EN: Lines 221-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `can_implement`, `PersistentTileSchedulerSm90AsyncInput`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 221-240 行使用条件编译来适配特性开关、平台或可选后端；在 `can_implement`、`PersistentTileSchedulerSm90AsyncInput` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 241-260 / 第 241-260 行

```cpp
241:     // ==============================
242:     int warp_group_role = canonical_warp_group_idx();
243:     int producer_warp_group_role = canonical_warp_idx_sync() % NumWarpsPerWarpGroup;
244:     is_mainloop_producer_ = warp_group_role == 0 && producer_warp_group_role == 0;
245:     total_grid_size_ = uint64_t(gridDim.x) * uint64_t(gridDim.y) * uint64_t(gridDim.z);
246:     // ==============================
247:     // CUSTOM LOGIC END
248:     // ==============================
249: #else
250:     CUTLASS_ASSERT(false && "This line should never be reached");
251: #endif
252:   }
253: 
254:   // Returns the initial work tile info that will be computed over
255:   template <class ClusterShape>
256:   CUTLASS_DEVICE
257:   WorkTileInfo
258:   initial_work_tile_info(ClusterShape cluster_shape) {
259:     return get_current_work();
260:   }
```

- EN: Lines 241-260 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `CUTLASS_ASSERT`, `initial_work_tile_info`, `get_current_work`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 241-260 行使用条件编译来适配特性开关、平台或可选后端；在 `CUTLASS_ASSERT`、`initial_work_tile_info`、`get_current_work` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 261-280 / 第 261-280 行

```cpp
261: 
262:   CUTLASS_DEVICE
263:   WorkTileInfo
264:   get_current_work() const {
265:     return get_current_work_for_linear_idx(current_work_linear_idx_);
266:   }
267: 
268:   CUTLASS_DEVICE
269:   WorkTileInfo
270:   get_current_work_for_linear_idx(uint64_t linear_idx) const {
271:     if (linear_idx >= scheduler_params.blocks_per_problem_) {
272:       return WorkTileInfo::invalid_work_tile();
273:     }
274: 
275:     // Map worker's linear index into the CTA tiled problem shape to the corresponding MNL indices
276:     uint64_t work_idx_l, remainder;
277:     scheduler_params.divmod_batch_(work_idx_l, remainder, linear_idx);
278: 
279:     uint64_t blk_per_grid_dim = scheduler_params.divmod_cluster_shape_minor_.divide(remainder);
280: 
```

- EN: Lines 261-280 introduces executable logic in routines such as `get_current_work`, `get_current_work_for_linear_idx`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 261-280 行在 `get_current_work`、`get_current_work_for_linear_idx` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 281-300 / 第 281-300 行

```cpp
281:     // ==============================
282:     // CUSTOM LOGIC BEGIN
283:     // ==============================
284:     uint64_t cluster_id, cluster_major_offset = 0, cluster_minor_offset = 0;
285:     scheduler_params.divmod_cluster_shape_major_(cluster_id, cluster_major_offset, blk_per_grid_dim);
286: 
287:     auto [cta_m_in_cluster, cta_n_in_cluster, _] = cute::block_id_in_cluster();
288:     if (scheduler_params.raster_order_ == RasterOrder::AlongN) {
289:       cluster_minor_offset = cta_m_in_cluster;
290:     }
291:     else {
292:       cluster_minor_offset = cta_n_in_cluster;
293:     }
294: 
295:     uint64_t cluster_idx_minor, cluster_idx_major;
296: 
297:     uint64_t cluster_idx_minor_div_swizzle, extra, offset;
298: 
299:     offset = cluster_id & ((1 << scheduler_params.log_swizzle_size_) - 1);
300:     extra = cluster_id >> scheduler_params.log_swizzle_size_;
```

- EN: Lines 281-300 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 281-300 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 301-320 / 第 301-320 行

```cpp
301: 
302:     scheduler_params.divmod_cluster_blk_major_(cluster_idx_minor_div_swizzle, cluster_idx_major, extra);
303: 
304:     cluster_idx_minor = cluster_idx_minor_div_swizzle * (1 << scheduler_params.log_swizzle_size_) + offset;
305: 
306:     auto minor_work_idx = static_cast<int32_t>(cluster_idx_minor * scheduler_params.divmod_cluster_shape_minor_.divisor +
307:                                                cluster_minor_offset);
308:     auto major_work_idx = static_cast<int32_t>(cluster_idx_major * scheduler_params.divmod_cluster_shape_major_.divisor +
309:                                                cluster_major_offset);
310: 
311:     int m, n;
312:     if (scheduler_params.raster_order_ == RasterOrder::AlongN) {
313:       m = minor_work_idx;
314:       n = major_work_idx;
315:     } else {
316:       m = major_work_idx;
317:       n = minor_work_idx;
318:     }
319: 
320:     // Pivot after swizzling
```

- EN: Lines 301-320 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 301-320 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 321-340 / 第 321-340 行

```cpp
321:     auto tiles_m = scheduler_params.problem_tiles_m_ * scheduler_params.cluster_shape_m_;
322:     m = (m + scheduler_params.tile_idx_pivot_m) % tiles_m;
323: 
324:     if (is_mainloop_producer_) {
325:       if (threadIdx.x == 0) {
326:         size_t chunk_idx = m / scheduler_params.tiles_per_chunk_m;
327:         wait_signal(scheduler_params.chunk_signals + chunk_idx);
328:       }
329: 
330:       // An arbitrary, non-default id
331:       constexpr int barrier_id = 8;
332:       arch::NamedBarrier barrier(NumThreadsPerWarp, barrier_id);
333:       barrier.arrive_and_wait();
334:     }
335: 
336:     return {m, n, static_cast<int32_t>(work_idx_l), true};
337:     // ==============================
338:     // CUSTOM LOGIC END
339:     // ==============================
340:   }
```

- EN: Lines 321-340 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 321-340 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 341-360 / 第 341-360 行

```cpp
341: 
342:   CUTLASS_DEVICE
343:   void
344:   advance_to_next_work(uint32_t advance_count = 1) {
345:     current_work_linear_idx_ += total_grid_size_ * uint64_t(advance_count);
346:   }
347: 
348:   CUTLASS_DEVICE
349:   bool is_last_tile(WorkTileInfo& work_tile_info, uint32_t advance_count = 1) const {
350:     if (continue_current_work(work_tile_info)) {
351:       return false;
352:     }
353:     return not get_current_work_for_linear_idx(
354:         current_work_linear_idx_ + (total_grid_size_ * uint64_t(advance_count))
355:     ).is_valid();
356:   }
357: 
358:   // Computes the linear index within a batch given M and N tile offsets within the batch.
359:   // This essentially inverts the mapping performed in get_work_idx_m_and_n
360:   static CUTLASS_DEVICE
```

- EN: Lines 341-360 introduces executable logic in routines such as `advance_to_next_work`, `is_last_tile`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 341-360 行在 `advance_to_next_work`、`is_last_tile` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 361-380 / 第 361-380 行

```cpp
361:   uint64_t
362:   get_linear_idx_from_m_and_n(
363:     int32_t tile_m,
364:     int32_t tile_n,
365:     FastDivmodU64Pow2 const& divmod_cluster_shape_major,
366:     FastDivmodU64Pow2 const& divmod_cluster_shape_minor,
367:     FastDivmodU64 const& divmod_cluster_blk_major,
368:     int32_t log_swizzle_size,
369:     RasterOrder raster_order) {
370: 
371:     uint64_t minor_work_idx, major_work_idx, cluster_minor_offset;
372:     if (raster_order == RasterOrder::AlongN) {
373:       minor_work_idx = static_cast<uint64_t>(tile_m);
374:       major_work_idx = static_cast<uint64_t>(tile_n);
375:       uint64_t cluster_m = divmod_cluster_shape_minor.divide(tile_m) * divmod_cluster_shape_minor.divisor;
376:       cluster_minor_offset = tile_m - cluster_m;
377:     }
378:     else {
379:       major_work_idx = static_cast<uint64_t>(tile_m);
380:       minor_work_idx = static_cast<uint64_t>(tile_n);
```

- EN: Lines 361-380 introduces executable logic in routines such as `get_linear_idx_from_m_and_n`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 361-380 行在 `get_linear_idx_from_m_and_n` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 381-400 / 第 381-400 行

```cpp
381:       uint64_t cluster_n = divmod_cluster_shape_minor.divide(tile_n) * divmod_cluster_shape_minor.divisor;
382:       cluster_minor_offset = tile_n - cluster_n;
383:     }
384: 
385:     uint64_t cluster_idx_minor, cluster_idx_major, cluster_major_offset;
386:     cluster_idx_minor = divmod_cluster_shape_minor.divide(minor_work_idx - cluster_minor_offset);
387:     divmod_cluster_shape_major(cluster_idx_major, cluster_major_offset, major_work_idx);
388: 
389:     uint64_t cluster_idx_minor_div_swizzle = cluster_idx_minor >> log_swizzle_size;
390:     uint64_t offset = cluster_idx_minor & ((1 << log_swizzle_size) - 1);
391: 
392:     uint64_t extra = cluster_idx_minor_div_swizzle * divmod_cluster_blk_major.divisor + cluster_idx_major;
393: 
394:     uint64_t cluster_id = (extra << log_swizzle_size) | offset;
395:     return (cluster_id * divmod_cluster_shape_major.divisor + cluster_major_offset) * divmod_cluster_shape_minor.divisor + cluster_minor_offset;
396:   }
397: 
398:   // Given the inputs, computes the total number of output blocks over which this problem will compute.
399:   // Note that this is only the logical size of our grid, not the physical grid we will actually launch.
400:   template<class ProblemShapeMNKL, class BlockShape, class ClusterShape>
```

- EN: Lines 381-400 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 381-400 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 401-420 / 第 401-420 行

```cpp
401:   CUTLASS_HOST_DEVICE static
402:   dim3
403:   get_tiled_cta_shape_mnl(ProblemShapeMNKL problem_shape_mnkl, BlockShape cta_shape, ClusterShape cluster_shape) {
404:     auto cta_m = cute::size(cute::ceil_div(cute::shape<0>(problem_shape_mnkl), cute::shape<0>(cta_shape)));
405:     auto cta_n = cute::size(cute::ceil_div(cute::shape<1>(problem_shape_mnkl), cute::shape<1>(cta_shape)));
406: 
407:     return Params::get_tiled_cta_shape_mnl(
408:       to_gemm_coord(problem_shape_mnkl),
409:       to_gemm_coord(cluster_shape),
410:       cta_m, cta_n
411:     );
412:   }
413: 
414:   // Reloaded interface that receives WorkTileInfo to deduce next work.
415:   // Kernel helper function to get next work tile
416:   CUTLASS_DEVICE
417:   auto
418:   fetch_next_work(WorkTileInfo work_tile_info) {
419:     if (continue_current_work(work_tile_info)) {
420:       return cute::make_tuple(work_tile_info, true);
```

- EN: Lines 401-420 introduces executable logic in routines such as `get_tiled_cta_shape_mnl`, `fetch_next_work`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 401-420 行在 `get_tiled_cta_shape_mnl`、`fetch_next_work` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 421-440 / 第 421-440 行

```cpp
421:     }
422: 
423:     advance_to_next_work();
424:     return cute::make_tuple(get_current_work(), true);
425:   }
426: 
427:   // Kernel helper function to get next work tile
428:   template <class TileSchedulerPipeline, class TileSchedulerPipelineState>
429:   CUTLASS_DEVICE
430:   auto
431:   fetch_next_work(
432:       WorkTileInfo work_tile_info,
433:       TileSchedulerPipeline& scheduler_pipeline,
434:       TileSchedulerPipelineState scheduler_pipe_consumer_state) {
435:     return fetch_next_work(work_tile_info);
436:   }
437: 
438:   // Given the inputs, computes the total number of output blocks over which this problem will compute.
439:   // Note that this is only the logical size of our grid, not the physical grid we will actually launch.
440:   template<class ProblemShapeMNKL, class TileShape, class AtomThrShape, class ClusterShape>
```

- EN: Lines 421-440 introduces executable logic in routines such as `advance_to_next_work`, `fetch_next_work`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 421-440 行在 `advance_to_next_work`、`fetch_next_work` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 441-460 / 第 441-460 行

```cpp
441:   CUTLASS_HOST_DEVICE static
442:   dim3
443:   get_tiled_cta_shape_mnl(ProblemShapeMNKL problem_shape_mnkl,
444:                           TileShape tile_shape_mnk,
445:                           AtomThrShape atom_thr_shape_mnk,
446:                           ClusterShape cluster_shape_mnk) {
447:     auto [tiles_m, tiles_n, tiles_l] = product_each(ceil_div(select<0,1,3>(problem_shape_mnkl), take<0,2>(tile_shape_mnk)));
448:     auto cta_m = round_nearest(tiles_m * size<0>(atom_thr_shape_mnk), size<0>(cluster_shape_mnk));
449:     auto cta_n = round_nearest(tiles_n * size<1>(atom_thr_shape_mnk), size<1>(cluster_shape_mnk));
450: 
451:     return Params::get_tiled_cta_shape_mnl(
452:       to_gemm_coord(problem_shape_mnkl),
453:       to_gemm_coord(cluster_shape_mnk),
454:       cta_m, cta_n
455:     );
456:   }
457: 
458:   CUTLASS_DEVICE
459:   static auto
460:   work_tile_to_cta_coord(WorkTileInfo work_tile_info) {
```

- EN: Lines 441-460 introduces executable logic in routines such as `get_tiled_cta_shape_mnl`, `work_tile_to_cta_coord`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 441-460 行在 `get_tiled_cta_shape_mnl`、`work_tile_to_cta_coord` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 461-480 / 第 461-480 行

```cpp
461:     // Get every cta coord in three dimensions of the cluster
462:     auto [cta_m_in_cluster, cta_n_in_cluster, cta_l_in_cluster] = cute::block_id_in_cluster();
463:     return make_coord(
464:       work_tile_info.M_idx + static_cast<int32_t>(cta_m_in_cluster),
465:       work_tile_info.N_idx + static_cast<int32_t>(cta_n_in_cluster),
466:       _,
467:       work_tile_info.L_idx + static_cast<int32_t>(cta_l_in_cluster)
468:     );
469:   }
470: 
471:   CUTLASS_DEVICE
472:   static auto
473:   work_tile_to_cta_coord(WorkTileInfo work_tile_info, dim3 block_id_in_cluster) {
474:     // Get every cta coord in three dimensions of the cluster
475:     auto [cta_m_in_cluster, cta_n_in_cluster, cta_l_in_cluster] = block_id_in_cluster;
476:     return make_coord(
477:       work_tile_info.M_idx + static_cast<int32_t>(cta_m_in_cluster),
478:       work_tile_info.N_idx + static_cast<int32_t>(cta_n_in_cluster),
479:       _,
480:       work_tile_info.L_idx + static_cast<int32_t>(cta_l_in_cluster)
```

- EN: Lines 461-480 introduces executable logic in routines such as `work_tile_to_cta_coord`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 461-480 行在 `work_tile_to_cta_coord` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 481-500 / 第 481-500 行

```cpp
481:     );
482:   }
483: 
484:   // Given the inputs, computes the physical grid we should launch.
485:   template<class ProblemShapeMNKL, class BlockShape, class ClusterShape>
486:   CUTLASS_HOST_DEVICE static
487:   dim3
488:   get_grid_shape(
489:       [[maybe_unused]] Params const& params,
490:       ProblemShapeMNKL problem_shape_mnk,
491:       BlockShape cta_shape,
492:       ClusterShape cluster_shape,
493:       KernelHardwareInfo hw_info,
494:       Arguments arguments = Arguments{},
495:       bool truncate_by_problem_size=true) {
496: 
497:     auto problem_shape_mnkl = cute::append<4>(problem_shape_mnk, cute::Int<1>{});
498:     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, cta_shape, cluster_shape);
499: 
500:     return Params::get_grid_shape(
```

- EN: Lines 481-500 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 481-500 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 501-520 / 第 501-520 行

```cpp
501:       problem_blocks,
502:       to_gemm_coord(cluster_shape),
503:       hw_info,
504:       arguments.max_swizzle_size,
505:       arguments.raster_order,
506:       /* truncate_by_problem_size = */true
507:     );
508:   }
509: 
510:   // Given the inputs, computes the physical grid we should launch.
511:   template<class ProblemShapeMNKL, class TileShape, class AtomThrShape, class ClusterShape>
512:   static dim3
513:   get_grid_shape(
514:       Params const& params,
515:       ProblemShapeMNKL problem_shape_mnkl,
516:       TileShape tile_shape_mnk,
517:       AtomThrShape atom_thr_shape_mnk,
518:       ClusterShape cluster_shape_mnk,
519:       KernelHardwareInfo hw_info) {
520: 
```

- EN: Lines 501-520 introduces executable logic in routines such as `get_grid_shape`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 501-520 行在 `get_grid_shape` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 521-540 / 第 521-540 行

```cpp
521:     dim3 problem_blocks = get_tiled_cta_shape_mnl(problem_shape_mnkl, tile_shape_mnk, atom_thr_shape_mnk, cluster_shape_mnk);
522:     Arguments args{};
523:     if constexpr (!std::is_const_v<decltype(args.max_swizzle_size)>) {
524:       args.max_swizzle_size = 1 << params.log_swizzle_size_;
525:     }
526:     args.raster_order = params.raster_order_ == RasterOrder::AlongN ? RasterOrderOptions::AlongN : RasterOrderOptions::AlongM;
527: 
528:     return Params::get_grid_shape(
529:       problem_blocks,
530:       to_gemm_coord(cluster_shape_mnk),
531:       hw_info,
532:       args.max_swizzle_size,
533:       args.raster_order,
534:       /* truncate_by_problem_size = */true
535:     );
536:   }
537: 
538:   // Convert CTA-level work tile info to cluster-level tile coord
539:   CUTLASS_DEVICE
540:   auto
```

- EN: Lines 521-540 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 521-540 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 541-560 / 第 541-560 行

```cpp
541:   work_tile_to_cluster_coord_mnkl(WorkTileInfo work_tile_info) const {
542:     // TileScheduler works at CTA-level, kernel works at cluster-level
543:     int m_coord = idx2crd(work_tile_info.M_idx / scheduler_params.cluster_shape_m_,
544:                           scheduler_params.problem_tiles_m_);
545:     int n_coord = idx2crd(work_tile_info.N_idx / scheduler_params.cluster_shape_n_,
546:                           scheduler_params.problem_tiles_n_);
547:     int l_coord = idx2crd(work_tile_info.L_idx,
548:                           scheduler_params.problem_tiles_l_);
549:     return make_coord(m_coord, n_coord, _, l_coord);
550:   }
551: 
552:   // Returns whether the block assigned this work should compute the epilogue for the corresponding
553:   // output tile. For the basic tile scheduler, this is always true.
554:   CUTLASS_HOST_DEVICE
555:   static bool
556:   compute_epilogue(WorkTileInfo const&, Params const&) {
557:     return true;
558:   }
559: 
560:   CUTLASS_HOST_DEVICE
```

- EN: Lines 541-560 introduces executable logic in routines such as `work_tile_to_cluster_coord_mnkl`, `compute_epilogue`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 541-560 行在 `work_tile_to_cluster_coord_mnkl`、`compute_epilogue` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 561-580 / 第 561-580 行

```cpp
561:   static bool
562:   compute_epilogue(WorkTileInfo const&) {
563:     return true;
564:   }
565: 
566:   // Performs the reduction across splits for a given output tile. Since this scheduler does
567:   // not split output tiles, no reduction is needed.
568:   template <class FrgTensorC>
569:   CUTLASS_DEVICE
570:   static void
571:   fixup(Params const&, WorkTileInfo const&, FrgTensorC&, uint32_t, uint32_t) {}
572: 
573:   // Performs the reduction across splits for a given output tile. No fixup is required for
574:   // work units returned by this scheduler.
575:   template <class FrgTensorC>
576:   CUTLASS_DEVICE
577:   void
578:   fixup(WorkTileInfo const&, FrgTensorC&, uint32_t, uint32_t) const { }
579: 
580:   // Returns whether the current WorkTileInfo passed in should continue to be used. Since
```

- EN: Lines 561-580 introduces executable logic in routines such as `compute_epilogue`, `fixup`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 561-580 行在 `compute_epilogue`、`fixup` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 581-600 / 第 581-600 行

```cpp
581:   // this scheduler only schedules work in units of single, full output tiles, the WorkTileInfo
582:   // passed in should not be used after having been processed.
583:   CUTLASS_DEVICE
584:   static bool
585:   continue_current_work(WorkTileInfo&) {
586:     return false;
587:   }
588: 
589:   template <class ProblemShapeMNKL, class TileShape, class Shape>
590:   CUTLASS_DEVICE
591:   auto
592:   get_k_tile_iterator(WorkTileInfo const& work_tile_info, ProblemShapeMNKL problem_shape_MNKL, TileShape tile_shape, Shape) {
593:     auto k_tiles = cute::ceil_div(cute::get<2>(problem_shape_MNKL), cute::get<2>(tile_shape));
594:     return cute::make_coord_iterator(k_tiles);
595:   }
596: 
597:   template <class ProblemShape, class TileShape>
598:   CUTLASS_HOST_DEVICE
599:   static int
600:   get_work_k_tile_count(WorkTileInfo const& work_tile_info, ProblemShape problem_shape, TileShape tile_shape) {
```

- EN: Lines 581-600 introduces executable logic in routines such as `continue_current_work`, `get_k_tile_iterator`, `get_work_k_tile_count`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 581-600 行在 `continue_current_work`、`get_k_tile_iterator`、`get_work_k_tile_count` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 601-620 / 第 601-620 行

```cpp
601:     // All work units returned by this scheduler cover the entire K iteration
602:     // space of the output tile assigned to the work unit.
603:     return cute::size(cute::ceil_div(cute::get<2>(problem_shape), cute::get<2>(tile_shape)));
604:   }
605: 
606:   CUTLASS_HOST_DEVICE
607:   static uint32_t
608:   get_work_k_tile_start(WorkTileInfo const&) {
609:     // All work units returned by this scheduler start from K tile 0
610:     return 0u;
611:   }
612: 
613:   CUTLASS_DEVICE
614:   static bool
615:   need_separate_reduction(Params const& params) {
616:     return false;
617:   }
618: 
619:   CUTLASS_DEVICE
620:   bool
```

- EN: Lines 601-620 introduces executable logic in routines such as `get_work_k_tile_start`, `need_separate_reduction`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 601-620 行在 `get_work_k_tile_start`、`need_separate_reduction` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 621-640 / 第 621-640 行

```cpp
621:   is_work_tile_for_reduction(WorkTileInfo const& work_tile_info, Params const& params) {
622:     return false;
623:   }
624: 
625:   template <class FrgTensorC>
626:   CUTLASS_DEVICE
627:   void
628:   separate_reduction(
629:     Params const& params,
630:     WorkTileInfo const& work_tile_info,
631:     FrgTensorC& accumulators,
632:     uint32_t num_barriers,
633:     uint32_t barrier_idx) {
634:   }
635: 
636:   // Shares the accumulator set with peers in the global workspace
637:   template <class FrgTensorC>
638:   CUTLASS_DEVICE
639:   static void
640:   share(
```

- EN: Lines 621-640 introduces executable logic in routines such as `is_work_tile_for_reduction`, `separate_reduction`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 621-640 行在 `is_work_tile_for_reduction`、`separate_reduction` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 641-660 / 第 641-660 行

```cpp
641:     Params const& params,
642:     WorkTileInfo const& work_tile_info,
643:     FrgTensorC& accumulators,
644:     uint32_t num_barriers,
645:     uint32_t barrier_idx) {
646:   }
647: 
648:   CUTLASS_DEVICE
649:   static bool
650:   valid_warpgroup_in_work_tile(WorkTileInfo const& work_tile_info) {
651:     return true;
652:   }
653: 
654:   CUTLASS_DEVICE
655:   static bool
656:   requires_separate_reduction(Params const& params) {
657:     return false;
658:   }
659: 
660:   // The basic tile scheduler does not require any additional workspace
```

- EN: Lines 641-660 introduces executable logic in routines such as `valid_warpgroup_in_work_tile`, `requires_separate_reduction`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 641-660 行在 `valid_warpgroup_in_work_tile`、`requires_separate_reduction` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 661-680 / 第 661-680 行

```cpp
661:   template <class ProblemShape, class ElementAccumulator>
662:   static size_t
663:   get_workspace_size(Arguments const&, ProblemShape, KernelHardwareInfo const&, uint32_t, const uint32_t = 1, uint32_t = 1) {
664:     return 0;
665:   }
666: 
667:   template <class ProblemShape, class ElementAccumulator>
668:   static cutlass::Status
669:   initialize_workspace(Arguments const&, void*, cudaStream_t, ProblemShape, KernelHardwareInfo const&,
670:     uint32_t, const uint32_t = 1, uint32_t = 1, CudaHostAdapter* cuda_adapter = nullptr) {
671:     return Status::kSuccess;
672:   }
673: 
674: public:
675:   // Sink scheduler params as a member
676:   Params scheduler_params;
677: };
678: 
679: // Selector
680: template <
```

- EN: Lines 661-680 introduces executable logic in routines such as `get_workspace_size`, `initialize_workspace`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 661-680 行在 `get_workspace_size`、`initialize_workspace` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 681-700 / 第 681-700 行

```cpp
681:   class KernelSchedule,
682:   class TileShape,
683:   class ClusterShape,
684:   uint32_t SchedulerPipelineStageCount,
685:   class ProblemShapeType
686: >
687: struct TileSchedulerSelector<
688:   PersistentAsyncInputScheduler<KernelSchedule>,
689:   arch::Sm90,
690:   TileShape,
691:   ClusterShape,
692:   SchedulerPipelineStageCount,
693:   ProblemShapeType
694:   > {
695:   using Scheduler = PersistentTileSchedulerSm90AsyncInput;
696: };
697: 
698: ///////////////////////////////////////////////////////////////////////////////
699: 
700: } // namespace cutlass::gemm::kernel::detail
```

- EN: Lines 681-700 opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `KernelSchedule`, `TileShape`, `ClusterShape`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 681-700 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `KernelSchedule`、`TileShape`、`ClusterShape` 等类型；包含面向 CUDA 的声明、内核或启动流程。

### Lines 701-702 / 第 701-702 行

```cpp
701: 
702: ///////////////////////////////////////////////////////////////////////////////
```

- EN: Lines 701-702 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 701-702 行包含面向 CUDA 的声明、内核或启动流程。

## Key Concepts / 关键概念

- EN: Subsystem: CUDA CUTLASS scheduling support for c10d.
- CN: 子系统：c10d 的 CUDA CUTLASS 调度支持。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `KernelSchedule`, `PersistentAsyncInputScheduler`, `PersistentTileSchedulerSm90AsyncInputParams`, `PersistentTileSchedulerSm90AsyncInput`
- CN: 核心符号：`KernelSchedule`、`PersistentAsyncInputScheduler`、`PersistentTileSchedulerSm90AsyncInputParams`、`PersistentTileSchedulerSm90AsyncInput`
- EN: Notable themes: CUDA paths.
- CN: 值得关注的主题：CUDA 路径。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: None / 无
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: None / 无
- External or system headers / 外部或系统头文件: `cutlass/gemm/kernel/static_tile_scheduler.hpp`
- Local symbols / 本地符号: `KernelSchedule`, `PersistentAsyncInputScheduler`, `PersistentTileSchedulerSm90AsyncInputParams`, `PersistentTileSchedulerSm90AsyncInput`