# nccl_reduce_scatter_offset.cu — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/ops/nccl_reduce_scatter_offset.cu`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for nccl reduce scatter offset in the c10d symmetric-memory operations. Key types include `ReduceScatterOffsetsInfo`. GPU-oriented code paths are present in this file.
- 用途 (CN): 该文件在c10d 对称内存操作中提供nccl reduce scatter offset 的实现逻辑。 关键类型包括 `ReduceScatterOffsetsInfo`。 该文件还包含面向 GPU 的代码路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <c10/cuda/CUDAGuard.h>
2: #include <ATen/Dispatch.h>
3: #include <ATen/cuda/CUDAContext.h>
4: #include <torch/csrc/distributed/c10d/NCCLUtils.hpp>
5: #include <torch/csrc/distributed/c10d/symm_mem/macros.hpp>
6: #include <torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp>
7: #include <torch/csrc/distributed/c10d/symm_mem/nccl_extension.hpp>
8: #include <torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp>
9: #include <torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp>
10: 
11: // Simultaneously reduce N blocks of a 2-D input tensor from a symmetric memory
12: // buffer, routing each block to a specific destination rank (dst_ranks[i]).
13: // Only the destination rank writes the reduced value to a contiguous output
14: // tensor, with the same shape as the owned block.
15: //
16: // The `dim` argument controls which dimension is sharded (0 or 1):
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；包含面向 CUDA 的声明、内核或启动流程。

### Lines 17-32 / 第 17-32 行

```cpp
17: //   dim=1 (column sharding): each block spans input[:, offsets[i-1]:offsets[i]]
18: //   dim=0 (row sharding):    each block spans input[offsets[i-1]:offsets[i], :]
19: //
20: // Blocks are described by inclusive-prefix-sum offsets along `dim`.
21: // For each j, out[j] must have the same shape across all ranks (i.e. the j-th
22: // owned block on every rank must have equal size); different j's may differ.
23: //
24: // If offsets is nullopt, input.size(dim) is divided equally into group_size blocks.
25: // If dst_ranks is nullopt, blocks are distributed round-robin across ranks.
26: //
27: // Ownership must be balanced: every rank must own the same number of blocks
28: // (N % group_size == 0 and dst_ranks distributes evenly).
29: 
30: namespace c10d::nccl_extension {
31: 
32: using namespace c10d::symmetric_memory;
```

- EN: Lines 17-32 opens or closes namespaces to place the code in the correct distributed component; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 17-32 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；包含面向 CUDA 的声明、内核或启动流程。

### Lines 33-48 / 第 33-48 行

```cpp
33: 
34: // Kernel requires device-side API: ncclLsaReduceSum.
35: #ifdef NCCL_DEVICE_HAS_REDUCE_COPY
36: 
37: // Naming conventions in this file:
38: // "BLOCK" means tensor block (as opposed to CUDA block);
39: // "CTA" means CUDA block;
40: // "RS" means Reduce Scatter;
41: // "slot" means which tensor block a CTA is assigned to.
42: 
43: constexpr int RS_MAX_BLOCKS = 64;           // max total blocks being scattered (N)
44: constexpr int RS_MAX_BLOCKS_PER_RANK = 16;  // max blocks owned by a single rank
45: constexpr int RS_MAX_CTAS_PER_BLOCK = 16;   // max CTAs assigned to one block
46: // Threads per CTA; defaults to a medium value to fit medium-width blocks.
47: constexpr int RS_THREADS_PER_CTA = 128;
48: // Total LSA barrier slots needed: one per CTA across all owned blocks.
```

- EN: Lines 33-48 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 33-48 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 49-64 / 第 49-64 行

```cpp
49: constexpr int RS_MAX_CTA_COUNT = (RS_MAX_BLOCKS_PER_RANK * RS_MAX_CTAS_PER_BLOCK);
50: 
51: // Per-slot data passed to the kernel in a single struct to avoid multiple
52: // kernel arguments.  Indexed by owned slot (0..n_owned-1).
53: struct ReduceScatterOffsetsInfo {
54:   size_t byte_offsets[RS_MAX_BLOCKS_PER_RANK]; // byte offset into the NCCL window
55:   void* dst_ptrs[RS_MAX_BLOCKS_PER_RANK];      // output pointer (contiguous)
56:   uint16_t dst_block_size[RS_MAX_BLOCKS_PER_RANK]; // per-slot size along the sharding dim
57:   uint16_t ctas_offset[RS_MAX_BLOCKS_PER_RANK]; // inclusive prefix sum of per-slot CTA counts
58:   uint8_t cta_slot[RS_MAX_CTA_COUNT];          // slot index for each flat CTA
59:   int n_owned;
60: };
61: 
62: // Grid: 1D, total_ctas = sum of per-slot CTA counts (info.ctas_offset[n_owned]).
63: // Each CTA belongs to one slot; blockIdx.x is the flat CTA index used as the
64: // LSA barrier index, ensuring all ranks assign the same index to each logical
```

- EN: Lines 49-64 declares or defines types such as `ReduceScatterOffsetsInfo`; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 49-64 行声明或定义了 `ReduceScatterOffsetsInfo` 等类型；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 65-80 / 第 65-80 行

```cpp
65: // (slot, local_block) pair (because owned_sizes[j] is consistent across ranks).
66: //
67: // UseMultimem=true: uses ncclMultimemReduceSum for hardware reduction via
68: // NVLink multicast; requires devcomm created with lsaMultimem=true.
69: // UseMultimem=false: uses ncclLsaReduceSum (software reduce via LSA reads).
70: template <typename T, bool UseMultimem>
71: __global__ void reduce_scatter_offset_kernel(
72:     ncclWindow_t window,
73:     ReduceScatterOffsetsInfo info,
74:     int fixed_dim_size,   // input.size(1-dim): constant across all slots
75:     bool col_sharded,     // true when dim==1
76:     int64_t outer_stride, // row stride of the input buffer (in elements)
77:     ncclDevComm devComm) {
78:   // cta_slot maps the flat CTA index to its owned slot.
79:   const int slot = info.cta_slot[blockIdx.x];
80:   // ctas_offset is an inclusive prefix sum, so slot_start is the flat index
```

- EN: Lines 65-80 contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 65-80 行包含面向 CUDA 的声明、内核或启动流程。

### Lines 81-96 / 第 81-96 行

```cpp
81:   // of the first CTA assigned to this slot.
82:   const int slot_start = slot > 0 ? info.ctas_offset[slot - 1] : 0;
83:   // local_block is this CTA's position within its slot (0-based row tile index).
84:   const int local_block = static_cast<int>(blockIdx.x) - slot_start;
85:   // Number of CTAs sharing this slot; used as the row-loop stride.
86:   const int ctas_for_slot = info.ctas_offset[slot] - slot_start;
87:   const ncclCoopCta coop{};
88: 
89:   // One LSA barrier per CTA; all ranks must call both syncs unconditionally.
90:   ncclLsaBarrierSession<ncclCoopCta> bar{
91:       coop,
92:       devComm,
93:       ncclTeamLsa(devComm),
94:       devComm.lsaBarrier,
95:       blockIdx.x};
96:   // Acquire: wait until all peers have written their data into the window.
```

- EN: Lines 81-96 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 81-96 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 97-112 / 第 97-112 行

```cpp
97:   bar.sync(coop, cuda::memory_order_acquire);
98: 
99:   const size_t base_byte_offset = info.byte_offsets[slot]; // start of this block in the window
100:   T* dst_base = reinterpret_cast<T*>(info.dst_ptrs[slot]); // start of out[slot]
101:   const int block_size = info.dst_block_size[slot]; // size along the sharding dim
102:   const int rows = col_sharded ? fixed_dim_size : block_size;
103:   const int cols = col_sharded ? block_size : fixed_dim_size;
104: 
105:   // Each CTA handles a strided subset of rows; the reduce reads from all peers
106:   // and writes cols elements starting at dst_row.
107:   for (int row = local_block; row < rows; row += ctas_for_slot) {
108:     const size_t row_offset =
109:         base_byte_offset +
110:         static_cast<size_t>(row * outer_stride) * sizeof(T);
111:     T* dst_row = dst_base + row * cols;
112:     if constexpr (UseMultimem) {
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 113-128 / 第 113-128 行

```cpp
113:       ncclMultimemReduceSum(
114:           coop, window, row_offset, dst_row, cols, devComm.lsaMultimem);
115:     } else {
116:       ncclLsaReduceSum(coop, window, row_offset, dst_row, cols, devComm);
117:     }
118:   }
119: 
120:   // Release: signal peers that we are done reading window memory.
121:   bar.sync(coop, cuda::memory_order_release);
122: }
123: 
124: #endif // NCCL_DEVICE_HAS_REDUCE_COPY
125: 
126: // Host entry point.  Validates arguments, resolves defaults, builds the
127: // per-slot ReduceScatterOffsetsInfo, and launches the kernel.
128: // See file-level comment for semantics.
```

- EN: Lines 113-128 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 113-128 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 129-144 / 第 129-144 行

```cpp
129: void nccl_reduce_scatter_offset(
130:     const at::Tensor& input,
131:     at::TensorList out,
132:     const std::string& group_name,
133:     int64_t dim,
134:     std::optional<at::IntArrayRef> offsets,
135:     std::optional<at::IntArrayRef> dst_ranks,
136:     const std::string& red_op) {
137: #ifdef NCCL_DEVICE_HAS_REDUCE_COPY
138:   TORCH_CHECK(
139:       red_op == "sum",
140:       "nccl_reduce_scatter_offset: only red_op='sum' is supported, got '", red_op, "'");
141: 
142:   TORCH_CHECK(
143:       input.dim() == 2,
144:       "nccl_reduce_scatter_offset: input must be 2-D");
```

- EN: Lines 129-144 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `nccl_reduce_scatter_offset`, `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 129-144 行使用条件编译来适配特性开关、平台或可选后端；在 `nccl_reduce_scatter_offset`、`TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 145-160 / 第 145-160 行

```cpp
145:   TORCH_CHECK(
146:       dim == 0 || dim == 1,
147:       "nccl_reduce_scatter_offset: dim must be 0 or 1, got ", dim);
148:   TORCH_CHECK(
149:       input.stride(-1) == 1,
150:       "nccl_reduce_scatter_offset: innermost dimension must be contiguous "
151:       "(stride[-1] == 1)");
152: 
153:   // rendezvous retrieves the symmetric memory handle; the tensor must have
154:   // been allocated via empty_strided_p2p with the NCCL backend.
155:   auto symm_mem = c10d::symmetric_memory::rendezvous(input, group_name);
156:   TORCH_CHECK(
157:       symm_mem != nullptr,
158:       "nccl_reduce_scatter_offset: input must be allocated via NCCL symmetric "
159:       "memory (use empty_strided_p2p with NCCL backend)");
160: 
```

- EN: Lines 145-160 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161:   auto* nccl_hdl = dynamic_cast<NCCLSymmetricMemory*>(symm_mem.get());
162:   TORCH_CHECK(
163:       nccl_hdl != nullptr,
164:       "nccl_reduce_scatter_offset: requires NCCL symmetric memory backend");
165: 
166:   c10::cuda::CUDAGuard guard(input.device());
167:   auto stream = at::cuda::getCurrentCUDAStream();
168:   auto device = input.device();
169: 
170:   auto& manager = c10d::symmetric_memory::NCCLDevCommManager::get(device);
171:   // Get the host-side communicator.
172:   ncclComm_t comm = manager.get_comm(group_name);
173: 
174:   const bool use_multimem = nccl_hdl->has_multicast_support();
175: 
176:   // The devcomm is cached per (group, key); create it on first use.
```

- EN: Lines 161-176 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-176 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 177-192 / 第 177-192 行

```cpp
177:   // lsaBarrierCount must cover the maximum number of concurrent CTAs.
178:   // lsaMultimem is set when the allocation has multicast support, so that
179:   // devComm.lsaMultimem is valid for ncclMultimemReduceSum in the kernel.
180:   static constexpr char const kDevcommKey[] = "nccl_reduce_scatter_offset";
181:   auto devcomm_opt = manager.get_devcomm(group_name, kDevcommKey);
182:   if (!devcomm_opt) {
183:     ncclDevCommRequirements reqs = NCCL_DEV_COMM_REQUIREMENTS_INITIALIZER;
184:     reqs.lsaBarrierCount = RS_MAX_CTA_COUNT;
185:     reqs.lsaMultimem = use_multimem;
186:     ncclDevComm devcomm;
187:     C10D_NCCL_CHECK(
188:         ncclDevCommCreate(comm, &reqs, &devcomm),
189:         "ncclDevCommCreate failed in nccl_reduce_scatter_offset");
190:     // Cache the device communicator.
191:     devcomm_opt = manager.register_devcomm(group_name, devcomm, kDevcommKey);
192:   }
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 193-208 / 第 193-208 行

```cpp
193:   ncclDevComm& devcomm = devcomm_opt->get();
194: 
195:   const int my_rank = devcomm.rank;
196:   const int group_size = devcomm.nRanks;
197: 
198:   // Determine n_blocks: from offsets if given, else group_size (equal-size default).
199:   const int n_blocks = offsets.has_value()
200:       ? static_cast<int>(offsets->size())
201:       : group_size;
202:   TORCH_CHECK(
203:       n_blocks > 0,
204:       "nccl_reduce_scatter_offset: must have at least one block");
205: 
206:   // Fill dst_ranks default: round-robin across ranks.
207:   std::vector<int64_t> dst_ranks_vec;
208:   at::IntArrayRef effective_dst_ranks;
```

- EN: Lines 193-208 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 193-208 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 209-224 / 第 209-224 行

```cpp
209:   if (dst_ranks.has_value()) {
210:     effective_dst_ranks = *dst_ranks;
211:   } else {
212:     dst_ranks_vec.resize(n_blocks);
213:     for (int i = 0; i < n_blocks; i++) {
214:       dst_ranks_vec[i] = i % group_size;
215:     }
216:     effective_dst_ranks = at::IntArrayRef(dst_ranks_vec);
217:   }
218: 
219:   // Fill offsets default: divide input.size(dim) equally among group_size blocks.
220:   std::vector<int64_t> offsets_vec;
221:   at::IntArrayRef effective_offsets;
222:   if (offsets.has_value()) {
223:     effective_offsets = *offsets;
224:     TORCH_CHECK(
```

- EN: Lines 209-224 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 209-224 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 225-240 / 第 225-240 行

```cpp
225:         effective_offsets[n_blocks - 1] <= input.size(dim),
226:         "nccl_reduce_scatter_offset: offsets exceed input size along dim ", dim);
227:   } else {
228:     const int64_t total = input.size(dim);
229:     TORCH_CHECK(
230:         total % group_size == 0,
231:         "nccl_reduce_scatter_offset: input.size(", dim, ")=", total,
232:         " must be divisible by group size (", group_size, ")");
233:     const int64_t block_size = total / group_size;
234:     offsets_vec.resize(n_blocks);
235:     for (int i = 0; i < n_blocks; i++) {
236:       offsets_vec[i] = (i + 1) * block_size;
237:     }
238:     effective_offsets = at::IntArrayRef(offsets_vec);
239:   }
240: 
```

- EN: Lines 225-240 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 225-240 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-256 / 第 241-256 行

```cpp
241:   TORCH_CHECK(
242:       n_blocks <= RS_MAX_BLOCKS,
243:       "nccl_reduce_scatter_offset: too many blocks: ", n_blocks,
244:       " (max ", RS_MAX_BLOCKS, ")");
245:   TORCH_CHECK(
246:       static_cast<int>(effective_dst_ranks.size()) == n_blocks,
247:       "nccl_reduce_scatter_offset: dst_ranks.size() must match offsets.size()");
248: 
249:   const int64_t outer_stride = input.stride(0);
250: 
251:   // Collect owned blocks (in order).
252:   std::vector<int> owned_indices;
253:   for (int i = 0; i < n_blocks; i++) {
254:     if (static_cast<int>(effective_dst_ranks[i]) == my_rank) {
255:       owned_indices.push_back(i);
256:     }
```

- EN: Lines 241-256 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 241-256 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 257-272 / 第 257-272 行

```cpp
257:   }
258:   const int n_owned = static_cast<int>(owned_indices.size());
259:   TORCH_CHECK(
260:       n_owned * group_size == n_blocks,
261:       "nccl_reduce_scatter_offset: dst_ranks must distribute blocks evenly "
262:       "(rank owns ", n_owned, "/", n_blocks, ", group_size=", group_size, ")");
263:   TORCH_CHECK(
264:       n_owned <= RS_MAX_BLOCKS_PER_RANK,
265:       "nccl_reduce_scatter_offset: too many owned blocks: ", n_owned,
266:       " (max ", RS_MAX_BLOCKS_PER_RANK, ")");
267:   // Balance is guaranteed above (n_owned * group_size == n_blocks), so
268:   // rank_counter[r] never exceeds n_owned during the owned_sizes loop.
269: 
270:   // For each j, out[j] must have the same shape across all ranks.  That means
271:   // all blocks that are the j-th owned block on their respective rank must have
272:   // equal size.  Different j's may differ in size.
```

- EN: Lines 257-272 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 257-272 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 273-288 / 第 273-288 行

```cpp
273:   //
274:   // Compute the size for each j by iterating all blocks in order, tracking
275:   // how many blocks each rank has seen so far (= the j-index for that block).
276:   std::vector<int64_t> owned_sizes(n_owned, -1);
277:   {
278:     std::vector<int> rank_counter(group_size, 0);
279:     for (int i = 0; i < n_blocks; i++) {
280:       const int r = static_cast<int>(effective_dst_ranks[i]);
281:       const int j = rank_counter[r]++;
282:       const int64_t sz =
283:           effective_offsets[i] - (i > 0 ? effective_offsets[i - 1] : 0);
284:       if (owned_sizes[j] < 0) {
285:         owned_sizes[j] = sz;
286:       } else {
287:         TORCH_CHECK(
288:             sz == owned_sizes[j],
```

- EN: Lines 273-288 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 273-288 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 289-304 / 第 289-304 行

```cpp
289:             "nccl_reduce_scatter_offset: all output at position j=", j,
290:             " must have equal size across all ranks");
291:       }
292:     }
293:   }
294: 
295:   TORCH_CHECK(
296:       static_cast<int>(out.size()) == n_owned,
297:       "nccl_reduce_scatter_offset: out.size() must be ", n_owned);
298:   for (int j = 0; j < n_owned; j++) {
299:     // dim=1: out[j] shape is (input.size(0), owned_sizes[j])
300:     // dim=0: out[j] shape is (owned_sizes[j], input.size(1))
301:     const int64_t exp0 = dim == 1 ? input.size(0) : owned_sizes[j];
302:     const int64_t exp1 = dim == 1 ? owned_sizes[j] : input.size(1);
303:     TORCH_CHECK(
304:         out[j].size(0) == exp0 && out[j].size(1) == exp1,
```

- EN: Lines 289-304 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 289-304 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 305-320 / 第 305-320 行

```cpp
305:         "nccl_reduce_scatter_offset: out[", j, "] must have shape (",
306:         exp0, ", ", exp1, ")");
307:     TORCH_CHECK(
308:         out[j].is_contiguous(),
309:         "nccl_reduce_scatter_offset: out[", j, "] must be contiguous");
310:     TORCH_CHECK(
311:         out[j].scalar_type() == input.scalar_type(),
312:         "nccl_reduce_scatter_offset: out[", j, "] must have the same dtype as input");
313:   }
314: 
315:   // Per-slot CTA count: sized for each slot independently.  owned_sizes[j] is
316:   // consistent across ranks, so ctas_offset is identical on every rank, which
317:   // guarantees all ranks launch the same total CTA count and agree on the
318:   // flat barrier index for each (slot, local_block) pair.
319:   const bool col_sharded = (dim == 1);
320:   const int fixed_dim_size = static_cast<int>(col_sharded ? input.size(0) : input.size(1));
```

- EN: Lines 305-320 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 305-320 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 321-336 / 第 321-336 行

```cpp
321:   const int unroll = 4 * 16 / static_cast<int>(input.element_size());
322:   const int elems_per_cta = RS_THREADS_PER_CTA * unroll;
323:   const size_t window_base_offset = nccl_hdl->get_offset();
324: 
325:   // Build the per-slot info struct.
326:   // For dim=1: byte_offsets encodes the column-block start within the window.
327:   // For dim=0: byte_offsets encodes the row-block start within the window.
328:   ReduceScatterOffsetsInfo info;
329:   info.n_owned = n_owned;
330:   for (int j = 0; j < n_owned; j++) {
331:     const int i = owned_indices[j];
332:     const int64_t block_start = (i > 0 ? effective_offsets[i - 1] : 0);
333:     const size_t elem_offset = col_sharded
334:         ? static_cast<size_t>(input.storage_offset() + block_start)
335:         : static_cast<size_t>(input.storage_offset()) +
336:               static_cast<size_t>(block_start) * outer_stride;
```

- EN: Lines 321-336 adds callable APIs, operators, or helper routines for this part of the file; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 321-336 行为本段代码补充可调用 API、操作符或辅助例程；包含面向 CUDA 的声明、内核或启动流程。

### Lines 337-352 / 第 337-352 行

```cpp
337:     info.byte_offsets[j] = window_base_offset + elem_offset * input.element_size();
338:     info.dst_ptrs[j] = out[j].data_ptr();
339:     info.dst_block_size[j] = static_cast<uint16_t>(owned_sizes[j]);
340:     const int numel_j = static_cast<int>(owned_sizes[j]) * fixed_dim_size;
341:     const int ctas_j = std::max(1, std::min(
342:         (numel_j + elems_per_cta - 1) / elems_per_cta, RS_MAX_CTAS_PER_BLOCK));
343:     info.ctas_offset[j] = static_cast<uint16_t>((j > 0 ? info.ctas_offset[j - 1] : 0) + ctas_j);
344:     const int slot_start = j > 0 ? info.ctas_offset[j - 1] : 0;
345:     for (int k = slot_start; k < info.ctas_offset[j]; ++k) {
346:       info.cta_slot[k] = static_cast<uint8_t>(j);
347:     }
348:   }
349:   const int total_ctas = info.ctas_offset[n_owned - 1];
350: 
351:   auto window = nccl_hdl->get_window();
352:   TORCH_CHECK(window != nullptr, "nccl_reduce_scatter_offset: NCCL window is null");
```

- EN: Lines 337-352 introduces executable logic in routines such as `TORCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing; performs validation and error handling to keep distributed state consistent.
- CN: 第 337-352 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程；执行校验与错误处理，以保持分布式状态一致。

### Lines 353-368 / 第 353-368 行

```cpp
353: 
354:   // Each owned (slot, local_block) pair gets one CTA; the flat CTA index is
355:   // the LSA barrier index.  All ranks launch the same total_ctas because
356:   // owned_sizes[j] is consistent, so every rank's ctas_offset is identical.
357:   AT_DISPATCH_NV_FLOATS(
358:       input.scalar_type(),
359:       "nccl_reduce_scatter_offset",
360:       [&]() {
361:         if (use_multimem) {
362:           reduce_scatter_offset_kernel<scalar_t, true>
363:               <<<total_ctas, RS_THREADS_PER_CTA, 0, stream>>>(
364:                   window, info, fixed_dim_size, col_sharded, outer_stride, devcomm);
365:           C10_CUDA_KERNEL_LAUNCH_CHECK();
366:         } else {
367:           reduce_scatter_offset_kernel<scalar_t, false>
368:               <<<total_ctas, RS_THREADS_PER_CTA, 0, stream>>>(
```

- EN: Lines 353-368 introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`; contains CUDA-oriented declarations, kernels, or launch plumbing.
- CN: 第 353-368 行在 `C10_CUDA_KERNEL_LAUNCH_CHECK` 等例程中引入具体执行逻辑；包含面向 CUDA 的声明、内核或启动流程。

### Lines 369-380 / 第 369-380 行

```cpp
369:                   window, info, fixed_dim_size, col_sharded, outer_stride, devcomm);
370:           C10_CUDA_KERNEL_LAUNCH_CHECK();
371:         }
372:       });
373: #else
374:   TORCH_CHECK(
375:       false,
376:       "nccl_reduce_scatter_offset requires NCCL >= 2.29.7 with reduce copy support");
377: #endif // NCCL_DEVICE_HAS_REDUCE_COPY
378: }
379: 
380: } // namespace c10d::nccl_extension
```

- EN: Lines 369-380 uses conditional compilation to adapt to feature flags, platforms, or optional backends; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `C10_CUDA_KERNEL_LAUNCH_CHECK`, `TORCH_CHECK`.
- CN: 第 369-380 行使用条件编译来适配特性开关、平台或可选后端；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `C10_CUDA_KERNEL_LAUNCH_CHECK`、`TORCH_CHECK` 等例程中引入具体执行逻辑。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory operations.
- CN: 子系统：c10d 对称内存操作。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `ReduceScatterOffsetsInfo`
- CN: 核心符号：`ReduceScatterOffsetsInfo`
- EN: Notable themes: CUDA paths.
- CN: 值得关注的主题：CUDA 路径。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/NCCLUtils.hpp`, `torch/csrc/distributed/c10d/symm_mem/macros.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_dev_cap.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_extension.hpp`, `torch/csrc/distributed/c10d/symm_mem/nccl_devcomm_manager.hpp`, `torch/csrc/distributed/c10d/symm_mem/NCCLSymmetricMemory.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/cuda/CUDAGuard.h`, `ATen/Dispatch.h`, `ATen/cuda/CUDAContext.h`
- External or system headers / 外部或系统头文件: None / 无
- Local symbols / 本地符号: `ReduceScatterOffsetsInfo`