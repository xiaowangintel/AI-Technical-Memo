# attention_kernels.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/attention/attention_kernels.cuh`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Implements the core paged-attention CUDA/HIP kernels, including QK score computation, softmax, V-weighted accumulation, optional FP8 KV-cache dequantization, block-sparse masking, and partitioned reduction. **CN:** 该文件实现 paged attention 的核心 CUDA/HIP 内核，包括 QK 分数计算、softmax、按 V 加权求和、可选的 FP8 KV cache 反量化、块稀疏掩码，以及分区归约逻辑。

## Line-by-Line Analysis / 逐行分析
### [kernel dependencies and block-wide sum / 依赖与块级求和]
```cpp
#include "attention_dtypes.h"
#include "attention_utils.cuh"
#include "../cuda_compat.h"

template <int NUM_WARPS>
inline __device__ float block_sum(float* red_smem, float sum) {
  int warp = threadIdx.x / WARP_SIZE;
  int lane = threadIdx.x % WARP_SIZE;
#pragma unroll
  for (int mask = WARP_SIZE / 2; mask >= 1; mask /= 2) {
    sum += VLLM_SHFL_XOR_SYNC(sum, mask);
  }
  if (lane == 0) {
    red_smem[warp] = sum;
  }
  __syncthreads();
  if (lane < NUM_WARPS) {
    sum = red_smem[lane];
  }
#pragma unroll
  for (int mask = NUM_WARPS / 2; mask >= 1; mask /= 2) {
    sum += VLLM_SHFL_XOR_SYNC(sum, mask);
  }
  return VLLM_SHFL_SYNC(sum, 0);
}
```
**EN:** `block_sum` is the softmax reduction helper. It first reduces within each warp using shuffle instructions, writes one partial sum per warp to shared memory, then lets the leading warp reduce those partials and broadcast the final result back to all threads.

**CN:** `block_sum` 是 softmax 使用的块级归约辅助函数。它先在每个 warp 内用 shuffle 做归约，再把每个 warp 的部分和写入共享内存，然后由前导 warp 完成跨 warp 归约，并把最终结果广播给整个线程块。

### [kernel interface and partition geometry / 内核接口与分区几何]
```cpp
template <typename scalar_t, typename cache_t, int HEAD_SIZE, int BLOCK_SIZE,
          int NUM_THREADS, vllm::Fp8KVCacheDataType KV_DTYPE,
          bool IS_BLOCK_SPARSE,
          int PARTITION_SIZE = 0>  // Zero means no partitioning.
__device__ void paged_attention_kernel(
    float* __restrict__ exp_sums,  // [num_seqs, num_heads, max_num_partitions]
    float* __restrict__ max_logits,  // [num_seqs, num_heads,
                                     // max_num_partitions]
    scalar_t* __restrict__ out,  // [num_seqs, num_heads, max_num_partitions,
                                 // head_size]
    const scalar_t* __restrict__ q,       // [num_seqs, num_heads, head_size]
    const cache_t* __restrict__ k_cache,  // [num_blocks, num_kv_heads,
                                          // head_size/x, block_size, x]
    const cache_t* __restrict__ v_cache,  // [num_blocks, num_kv_heads,
                                          // head_size, block_size]
    const int num_kv_heads,               // [num_heads]
    const float scale,
    const int* __restrict__ block_tables,  // [num_seqs, max_num_blocks_per_seq]
    const int* __restrict__ seq_lens,      // [num_seqs]
    const int max_num_blocks_per_seq,
    const float* __restrict__ alibi_slopes,  // [num_heads]
    const int q_stride, const int kv_block_stride, const int kv_head_stride,
    const float* k_scale, const float* v_scale, const int tp_rank,
    const int blocksparse_local_blocks, const int blocksparse_vert_stride,
    const int blocksparse_block_size, const int blocksparse_head_sliding_step) {
  const int seq_idx = blockIdx.y;
  const int partition_idx = blockIdx.z;
  const int max_num_partitions = gridDim.z;
  constexpr bool USE_PARTITIONING = PARTITION_SIZE > 0;
  const int seq_len = seq_lens[seq_idx];
  if (USE_PARTITIONING && partition_idx * PARTITION_SIZE >= seq_len) {
    return;
  }

  const int num_seq_blocks = DIVIDE_ROUND_UP(seq_len, BLOCK_SIZE);
  const int num_blocks_per_partition =
      USE_PARTITIONING ? PARTITION_SIZE / BLOCK_SIZE : num_seq_blocks;
  const int start_block_idx =
      USE_PARTITIONING ? partition_idx * num_blocks_per_partition : 0;
  const int end_block_idx =
      MIN(start_block_idx + num_blocks_per_partition, num_seq_blocks);

  const int start_token_idx = start_block_idx * BLOCK_SIZE;
  const int end_token_idx =
      MIN(start_token_idx + (end_block_idx - start_block_idx) * BLOCK_SIZE,
          seq_len);
  const int num_tokens = end_token_idx - start_token_idx;
}
```
**EN:** The device function is heavily templated so the compiler can specialize for head size, block size, thread count, cache dtype, and whether block-sparse logic is enabled. Grid X indexes heads, Y indexes sequences, and Z indexes partitions when long contexts are split for version-2 attention.

**CN:** 这个 device 函数采用大量模板参数，使编译器能针对 head size、block size、线程数、cache dtype，以及是否启用块稀疏逻辑进行完全特化。网格的 X 维对应 head，Y 维对应序列，Z 维在 v2 attention 中用于表示上下文分区。

### [thread-group sizing and query preload / 线程组尺寸与 query 预加载]
```cpp
constexpr int THREAD_GROUP_SIZE = MAX(WARP_SIZE / BLOCK_SIZE, 1);
constexpr int NUM_THREAD_GROUPS = NUM_THREADS / THREAD_GROUP_SIZE;
constexpr int NUM_TOKENS_PER_THREAD_GROUP =
    DIVIDE_ROUND_UP(BLOCK_SIZE, WARP_SIZE);
constexpr int NUM_WARPS = NUM_THREADS / WARP_SIZE;

constexpr int VEC_SIZE = MAX(16 / (THREAD_GROUP_SIZE * sizeof(scalar_t)), 1);
using K_vec = typename Vec<scalar_t, VEC_SIZE>::Type;
using Q_vec = typename Vec<scalar_t, VEC_SIZE>::Type;
using Quant_vec = typename Vec<cache_t, VEC_SIZE>::Type;

const scalar_t* q_ptr = q + seq_idx * q_stride + head_idx * HEAD_SIZE;
__shared__ Q_vec q_vecs[THREAD_GROUP_SIZE][NUM_VECS_PER_THREAD];
#pragma unroll
for (int i = thread_group_idx; i < NUM_VECS_PER_THREAD;
     i += NUM_THREAD_GROUPS) {
  const int vec_idx = thread_group_offset + i * THREAD_GROUP_SIZE;
  q_vecs[thread_group_offset][i] =
      *reinterpret_cast<const Q_vec*>(q_ptr + vec_idx * VEC_SIZE);
}
__syncthreads();
```
**EN:** The kernel chooses `VEC_SIZE` so each thread group cooperatively moves 16 bytes at a time, which aligns well with GPU memory transactions. Query vectors are loaded once into shared memory and then reused for every KV block, avoiding repeated global-memory fetches during the sequence scan.

**CN:** 内核通过 `VEC_SIZE` 的计算确保每个线程组一次协同搬运 16 字节数据，这与 GPU 的内存访问粒度更匹配。Query 向量只从全局内存加载一次到共享内存，随后在遍历所有 KV block 时重复复用，避免了反复读取 Q 的开销。

### [block-sparse gating and QK score generation / 块稀疏筛选与 QK 分数生成]
```cpp
if constexpr (IS_BLOCK_SPARSE) {
  const int k_bs_block_id = block_idx * BLOCK_SIZE / blocksparse_block_size;
  const bool is_remote =
      ((k_bs_block_id + bs_block_offset) % blocksparse_vert_stride == 0);
  const bool is_local =
      (k_bs_block_id > q_bs_block_id - blocksparse_local_blocks);
  if (!is_remote && !is_local) {
    if (thread_group_offset == 0) {
      logits[token_idx - start_token_idx] = -FLT_MAX;
    }
    continue;
  }
}

if constexpr (KV_DTYPE == Fp8KVCacheDataType::kAuto) {
  k_vecs[j] = *reinterpret_cast<const K_vec*>(
      k_ptr + offset1 * BLOCK_SIZE * x + offset2);
} else {
  Quant_vec k_vec_quant = *reinterpret_cast<const Quant_vec*>(
      k_ptr + offset1 * BLOCK_SIZE * x + offset2);
  k_vecs[j] = fp8::scaled_convert<K_vec, Quant_vec, KV_DTYPE>(
      k_vec_quant, *k_scale);
}

float qk = scale * Qk_dot<scalar_t, THREAD_GROUP_SIZE>::dot(
                       q_vecs[thread_group_offset], k_vecs);
qk += (alibi_slope != 0) ? alibi_slope * (token_idx - seq_len + 1) : 0;
```
**EN:** The kernel can skip entire KV blocks under block-sparse attention by writing `-FLT_MAX` logits for skipped tokens so softmax ignores them. For dense or accepted blocks, it loads packed keys directly or dequantizes FP8 cache values on the fly, computes the scaled QK dot product, and then adds the optional ALiBi positional bias.

**CN:** 在块稀疏注意力模式下，内核可以直接跳过整块 KV：对被跳过 token 写入 `-FLT_MAX`，这样 softmax 会自然忽略它们。对于需要参与计算的块，内核会直接加载打包的 K，或者在读取时对 FP8 cache 进行反量化，然后计算带缩放的 QK 点积，并额外叠加可选的 ALiBi 位置偏置。

### [softmax normalization and partition metadata / softmax 归一化与分区元数据]
```cpp
for (int mask = WARP_SIZE / 2; mask >= THREAD_GROUP_SIZE; mask /= 2) {
  qk_max = fmaxf(qk_max, VLLM_SHFL_XOR_SYNC(qk_max, mask));
}
if (lane == 0) {
  red_smem[warp_idx] = qk_max;
}
__syncthreads();

qk_max = lane < NUM_WARPS ? red_smem[lane] : -FLT_MAX;
#pragma unroll
for (int mask = NUM_WARPS / 2; mask >= 1; mask /= 2) {
  qk_max = fmaxf(qk_max, VLLM_SHFL_XOR_SYNC(qk_max, mask));
}
qk_max = VLLM_SHFL_SYNC(qk_max, 0);

float exp_sum = 0.f;
for (int i = thread_idx; i < num_tokens; i += NUM_THREADS) {
  float val = __expf(logits[i] - qk_max);
  logits[i] = val;
  exp_sum += val;
}
exp_sum = block_sum<NUM_WARPS>(&red_smem[NUM_WARPS], exp_sum);
```
**EN:** Softmax is stabilized in the standard way: compute a block-wide maximum logit, subtract it before exponentiation, then reduce the exponentials to get the denominator. When partitioning is enabled, the kernel also writes `qk_max` and `exp_sum` so a later reduction kernel can merge multiple partitions without losing numerical stability.

**CN:** softmax 使用标准的数值稳定策略：先求整个线程块范围内的最大 logit，指数化之前先减去它，再对指数结果做归约得到分母。若启用了分区模式，内核还会把 `qk_max` 与 `exp_sum` 写出，供后续 reduce kernel 以稳定方式合并多个分区结果。

### [value loading and weighted accumulation / Value 读取与加权累加]
```cpp
scalar_t zero_value;
zero(zero_value);
for (int block_idx = start_block_idx + warp_idx; block_idx < end_block_idx;
     block_idx += NUM_WARPS) {
  const int64_t physical_block_number =
      static_cast<int64_t>(block_table[block_idx]);
  const int physical_block_offset = (lane % NUM_V_VECS_PER_ROW) * V_VEC_SIZE;
  const int token_idx = block_idx * BLOCK_SIZE + physical_block_offset;
  L_vec logits_vec;
  from_float(logits_vec, *reinterpret_cast<Float_L_vec*>(logits + token_idx -
                                                         start_token_idx));

  const cache_t* v_ptr = v_cache + physical_block_number * kv_block_stride +
                         kv_head_idx * kv_head_stride;
#pragma unroll
  for (int i = 0; i < NUM_ROWS_PER_THREAD; i++) {
    const int row_idx = lane / NUM_V_VECS_PER_ROW + i * NUM_ROWS_PER_ITER;
    if (row_idx < HEAD_SIZE) {
      const int offset = row_idx * BLOCK_SIZE + physical_block_offset;
      V_vec v_vec;

      if constexpr (KV_DTYPE == Fp8KVCacheDataType::kAuto) {
        v_vec = *reinterpret_cast<const V_vec*>(v_ptr + offset);
      } else {
        V_quant_vec v_quant_vec =
            *reinterpret_cast<const V_quant_vec*>(v_ptr + offset);
        v_vec = fp8::scaled_convert<V_vec, V_quant_vec, KV_DTYPE>(v_quant_vec,
                                                                  *v_scale);
      }
      if (block_idx == num_seq_blocks - 1) {
        scalar_t* v_vec_ptr = reinterpret_cast<scalar_t*>(&v_vec);
#pragma unroll
        for (int j = 0; j < V_VEC_SIZE; j++) {
          v_vec_ptr[j] = token_idx + j < seq_len ? v_vec_ptr[j] : zero_value;
        }
      }
      accs[i] += dot(logits_vec, v_vec);
    }
  }
}
```
**EN:** After softmax, each warp reads value vectors and multiplies them by the corresponding probabilities. The final sequence block needs special handling: tokens beyond `seq_len` may hold garbage or NaNs in cache memory, so the kernel explicitly zeros those elements before computing the weighted sum.

**CN:** 在 softmax 完成后，每个 warp 会读取 value 向量并按对应概率加权。序列的最后一个 block 需要额外处理：超出 `seq_len` 的位置在 cache 中可能带有垃圾值甚至 NaN，因此内核会在参与加权之前显式把这些元素清零。

### [cross-warp reduction and output writeback / 跨 warp 归约与输出写回]
```cpp
float* out_smem = reinterpret_cast<float*>(shared_mem);
#pragma unroll
for (int i = NUM_WARPS; i > 1; i /= 2) {
  int mid = i / 2;
  if (warp_idx >= mid && warp_idx < i) {
    float* dst = &out_smem[(warp_idx - mid) * HEAD_SIZE];
#pragma unroll
    for (int i = 0; i < NUM_ROWS_PER_THREAD; i++) {
      const int row_idx = lane / NUM_V_VECS_PER_ROW + i * NUM_ROWS_PER_ITER;
      if (row_idx < HEAD_SIZE && lane % NUM_V_VECS_PER_ROW == 0) {
        dst[row_idx] = accs[i];
      }
    }
  }
  __syncthreads();

  if (warp_idx < mid) {
    const float* src = &out_smem[warp_idx * HEAD_SIZE];
#pragma unroll
    for (int i = 0; i < NUM_ROWS_PER_THREAD; i++) {
      const int row_idx = lane / NUM_V_VECS_PER_ROW + i * NUM_ROWS_PER_ITER;
      if (row_idx < HEAD_SIZE && lane % NUM_V_VECS_PER_ROW == 0) {
        accs[i] += src[row_idx];
      }
    }
  }
  __syncthreads();
}

if (warp_idx == 0) {
  scalar_t* out_ptr =
      out + seq_idx * num_heads * max_num_partitions * HEAD_SIZE +
      head_idx * max_num_partitions * HEAD_SIZE + partition_idx * HEAD_SIZE;
#pragma unroll
  for (int i = 0; i < NUM_ROWS_PER_THREAD; i++) {
    const int row_idx = lane / NUM_V_VECS_PER_ROW + i * NUM_ROWS_PER_ITER;
    if (row_idx < HEAD_SIZE && lane % NUM_V_VECS_PER_ROW == 0) {
      from_float(*(out_ptr + row_idx), accs[i]);
    }
  }
}
```
**EN:** Each warp first accumulates a partial output vector. The kernel then performs a tree-style reduction across warps using shared memory, reusing the same dynamic shared-memory buffer that previously stored logits. Finally, warp 0 converts the FP32 accumulators back to `scalar_t` and writes the output slice.

**CN:** 每个 warp 先得到一份局部输出向量，然后内核使用共享内存做树形的跨 warp 归约；这里还复用了前面存放 logits 的动态共享内存。最后由 warp 0 把 FP32 累计结果转换回 `scalar_t`，并写回对应的输出切片。

### [v1/v2 launch wrappers and partition merge / v1/v2 启动包装与分区合并]
```cpp
template <typename scalar_t, typename cache_t, int HEAD_SIZE, int BLOCK_SIZE,
          int NUM_THREADS, vllm::Fp8KVCacheDataType KV_DTYPE,
          bool IS_BLOCK_SPARSE>
__global__ void paged_attention_v1_kernel(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ q,
    const cache_t* __restrict__ k_cache,
    const cache_t* __restrict__ v_cache,
    const int num_kv_heads,
    const float scale,
    const int* __restrict__ block_tables,
    const int* __restrict__ seq_lens,
    const int max_num_blocks_per_seq,
    const float* __restrict__ alibi_slopes,
    const int q_stride, const int kv_block_stride, const int kv_head_stride,
    const float* k_scale, const float* v_scale, const int tp_rank,
    const int blocksparse_local_blocks, const int blocksparse_vert_stride,
    const int blocksparse_block_size, const int blocksparse_head_sliding_step) {
  paged_attention_kernel<scalar_t, cache_t, HEAD_SIZE, BLOCK_SIZE, NUM_THREADS,
                         KV_DTYPE, IS_BLOCK_SPARSE>(
      /* exp_sums */ nullptr, /* max_logits */ nullptr, out, q, k_cache,
      v_cache, num_kv_heads, scale, block_tables, seq_lens,
      max_num_blocks_per_seq, alibi_slopes, q_stride, kv_block_stride,
      kv_head_stride, k_scale, v_scale, tp_rank, blocksparse_local_blocks,
      blocksparse_vert_stride, blocksparse_block_size,
      blocksparse_head_sliding_step);
}
```
**EN:** Version 1 is the simple path: no partitioning, so it directly materializes the final output from one kernel invocation.

**CN:** v1 是简单路径：不做分区，因此一次 kernel 调用就直接产生最终输出。

### [v2 reduction kernel / v2 归约内核]
```cpp
const int seq_len = seq_lens[seq_idx];
const int num_partitions = DIVIDE_ROUND_UP(seq_len, PARTITION_SIZE);
if (num_partitions == 1) {
  scalar_t* out_ptr =
      out + seq_idx * num_heads * HEAD_SIZE + head_idx * HEAD_SIZE;
  const scalar_t* tmp_out_ptr =
      tmp_out + seq_idx * num_heads * max_num_partitions * HEAD_SIZE +
      head_idx * max_num_partitions * HEAD_SIZE;
  for (int i = threadIdx.x; i < HEAD_SIZE; i += blockDim.x) {
    out_ptr[i] = tmp_out_ptr[i];
  }
  return;
}

for (int i = threadIdx.x; i < num_partitions; i += blockDim.x) {
  float l = shared_max_logits[i];
  float rescaled_exp_sum = exp_sums_ptr[i] * expf(l - max_logit);
  global_exp_sum += rescaled_exp_sum;
  shared_exp_sums[i] = rescaled_exp_sum;
}
__syncthreads();
global_exp_sum = block_sum<NUM_WARPS>(&red_smem[NUM_WARPS], global_exp_sum);
const float inv_global_exp_sum = __fdividef(1.0f, global_exp_sum + 1e-6f);

const scalar_t* tmp_out_ptr =
    tmp_out + seq_idx * num_heads * max_num_partitions * HEAD_SIZE +
    head_idx * max_num_partitions * HEAD_SIZE;
scalar_t* out_ptr =
    out + seq_idx * num_heads * HEAD_SIZE + head_idx * HEAD_SIZE;
#pragma unroll
for (int i = threadIdx.x; i < HEAD_SIZE; i += NUM_THREADS) {
  float acc = 0.0f;
  for (int j = 0; j < num_partitions; ++j) {
    acc += to_float(tmp_out_ptr[j * HEAD_SIZE + i]) * shared_exp_sums[j] *
           inv_global_exp_sum;
  }
  from_float(out_ptr[i], acc);
}
```
**EN:** Version 2 splits long contexts into partitions. The reduce kernel first finds the global max logit across partitions, rescales each partition’s `exp_sum` into the shared global exponent frame, and then combines `tmp_out` using those corrected weights. This preserves the exact softmax semantics while avoiding a single giant attention kernel.

**CN:** v2 会把长上下文拆成多个 partition。归约内核先找出所有 partition 的全局最大 logit，再把每个 partition 的 `exp_sum` 重缩放到统一的指数坐标系中，最后按修正后的权重合并 `tmp_out`。这样既保持了 softmax 语义，又避免了单个超大 attention kernel 的代价。

## Key Concepts / 关键概念
- **EN:** The kernel implements paged KV-cache traversal through `block_tables`, not contiguous sequence memory.
- **CN:** 该内核通过 `block_tables` 遍历分页式 KV cache，而不是依赖连续的序列内存。
- **EN:** Query data is reused from shared memory, while K/V data is streamed block by block.
- **CN:** Query 数据先缓存到共享内存中复用，而 K/V 数据则按 block 流式读取。
- **EN:** FP8 cache support is integrated by on-the-fly dequantization using `k_scale` and `v_scale`.
- **CN:** FP8 cache 支持通过 `k_scale` 和 `v_scale` 在读取时即时反量化完成。
- **EN:** Version-2 partitioning stores softmax statistics separately so multiple partial attentions can be merged numerically correctly.
- **CN:** v2 分区模式会额外保存 softmax 统计量，以便多个局部注意力结果可以按数值正确的方式合并。
- **EN:** Block-sparse mode skips disallowed blocks before both the QK and V phases.
- **CN:** 块稀疏模式会在 QK 阶段和 V 阶段都提前跳过不允许访问的块。

## Dependencies / 依赖关系
- **EN:** Depends on `attention_utils.cuh` for `Qk_dot` and on `attention_dtypes.h` for dtype-specific vector operations.
- **CN:** 依赖 `attention_utils.cuh` 提供 `Qk_dot`，并依赖 `attention_dtypes.h` 提供各 dtype 的向量运算。
- **EN:** Depends on `cuda_compat.h` for warp shuffle macros and GPU portability helpers.
- **CN:** 依赖 `cuda_compat.h` 提供 warp shuffle 宏与 GPU 兼容辅助。
- **EN:** Depends on FP8 quantization helpers under `../quantization/w8a8/fp8/.../quant_utils.cuh` when `KV_DTYPE` is FP8.
- **CN:** 当 `KV_DTYPE` 为 FP8 时，依赖 `../quantization/w8a8/fp8/.../quant_utils.cuh` 中的反量化辅助。
- **EN:** Consumes runtime metadata such as `seq_lens`, `block_tables`, ALiBi slopes, tensor-parallel rank, and block-sparse parameters.
- **CN:** 运行时依赖 `seq_lens`、`block_tables`、ALiBi slope、张量并行 rank 以及块稀疏参数等元数据。
