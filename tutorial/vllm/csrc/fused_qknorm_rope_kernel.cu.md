# fused_qknorm_rope_kernel.cu — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/fused_qknorm_rope_kernel.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Provides fused in-place Q/K RMSNorm+RoPE kernels for combined QKV tensors, including a baseline warp-per-head implementation and a cp.async-optimized multi-head variant. **CN:** 为组合式 QKV 张量提供原地融合的 Q/K RMSNorm+RoPE 内核，包括基础版“一 warp 一头”实现，以及基于 cp.async 优化的多头变体。

## Line-by-Line Analysis / 逐行分析

### Validation macros and warp utilities / 校验宏与 warp 工具
```cpp
#define CHECK_TYPE(x, st)                                              \
  TORCH_CHECK(x.scalar_type() == st, #x " dtype is ", x.scalar_type(), \
              ", while ", st, " is expected")
#define CHECK_TH_CUDA(x) TORCH_CHECK(x.is_cuda(), #x " must be a CUDA tensor")
#define CHECK_CONTIGUOUS(x) \
  TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
#define CHECK_INPUT(x) \
  CHECK_TH_CUDA(x);    \
  CHECK_CONTIGUOUS(x)

#ifdef USE_ROCM
  #define FINAL_MASK 0xffffffffffffffffULL

  #if defined(HIP_VERSION) && HIP_VERSION < 70000000
// On ROCm versions before 7.0, __syncwarp isn't defined. The below
// implementation is copy/pasted from the implementation in ROCm 7.0
__device__ inline void __syncwarp() {
  __builtin_amdgcn_fence(__ATOMIC_RELEASE, "wavefront");
  __builtin_amdgcn_wave_barrier();
  __builtin_amdgcn_fence(__ATOMIC_ACQUIRE, "wavefront");
}
  #endif
#else
  #define FINAL_MASK 0xffffffff
#endif

namespace tensorrt_llm::common {
template <typename T, int num>
struct packed_as;
// Specialization for packed_as used in this kernel.
template <>
struct packed_as<uint, 1> {
  using type = uint;
};

template <>
struct packed_as<uint, 2> {
  using type = uint2;
};

template <>
struct packed_as<uint, 4> {
  using type = uint4;
};

template <typename T>
__inline__ __device__ T warpReduceSum(T val) {
#pragma unroll
  for (int mask = 16; mask > 0; mask >>= 1)
    val += __shfl_xor_sync(FINAL_MASK, val, mask, 32);
  return val;
}

template <typename T>
inline __device__ __host__ T divUp(T m, T n) {
  return (m + n - 1) / n;
}

}  // namespace tensorrt_llm::common
```
**EN:** The file starts by defining tensor-validation helpers, ROCm-compatible FINAL_MASK/__syncwarp behavior, and small utilities such as packed_as, warpReduceSum, and divUp. These utilities are reused by both kernel variants and their launchers.
**CN:** 文件开头定义了张量校验辅助宏、ROCm 兼容的 FINAL_MASK/__syncwarp 行为，以及 packed_as、warpReduceSum、divUp 等小工具。这些工具会被两个内核变体及其启动器复用。

### Baseline kernel: load and RMSNorm / 基础内核：加载并执行 RMSNorm
```cpp
template <typename scalar_t_in, typename scalar_t_cache, int head_dim,
          bool interleave>
__global__ void fusedQKNormRopeKernel(
    void* qkv_void,                  // Combined QKV tensor
    int const num_heads_q,           // Number of query heads
    int const num_heads_k,           // Number of key heads
    int const num_heads_v,           // Number of value heads
    float const eps,                 // Epsilon for RMS normalization
    void const* q_weight_void,       // RMSNorm weights for query
    void const* k_weight_void,       // RMSNorm weights for key
    void const* cos_sin_cache_void,  // Pre-computed cos/sin cache
    int64_t const* position_ids,     // Position IDs for RoPE
    int const num_tokens,            // Number of tokens
    int const rotary_dim             // Dimension for RoPE
) {
#if (!defined(__CUDA_ARCH__) || __CUDA_ARCH__ < 800) && !defined(USE_ROCM)
  if constexpr ((std::is_same_v<scalar_t_in, c10::BFloat16>) ||
                std::is_same_v<scalar_t_cache, c10::BFloat16>) {
    return;
  } else {
#endif

    using Converter = vllm::_typeConvert<scalar_t_in>;
    static_assert(Converter::exists,
                  "Input QKV data type is not supported for this CUDA "
                  "architecture or toolkit version.");
    using T_in = typename Converter::hip_type;
    using T2_in = typename Converter::packed_hip_type;

    using CacheConverter = vllm::_typeConvert<scalar_t_cache>;
    static_assert(CacheConverter::exists,
                  "Cache data type is not supported for this CUDA architecture "
                  "or toolkit version.");
    using T_cache = typename CacheConverter::hip_type;

    T_in* qkv = reinterpret_cast<T_in*>(qkv_void);
    T_in const* q_weight = reinterpret_cast<T_in const*>(q_weight_void);
    T_in const* k_weight = reinterpret_cast<T_in const*>(k_weight_void);
    T_cache const* cos_sin_cache =
        reinterpret_cast<T_cache const*>(cos_sin_cache_void);

    int const warpsPerBlock = blockDim.x / 32;
    int const warpId = threadIdx.x / 32;
    int const laneId = threadIdx.x % 32;

    // Calculate global warp index to determine which head/token this warp
    // processes
    int const globalWarpIdx = blockIdx.x * warpsPerBlock + warpId;

    // Total number of attention heads (Q and K)
    int const total_qk_heads = num_heads_q + num_heads_k;

    // Determine which token and head type (Q or K) this warp processes
    int const tokenIdx = globalWarpIdx / total_qk_heads;
    int const localHeadIdx = globalWarpIdx % total_qk_heads;

    // Skip if this warp is assigned beyond the number of tokens
    if (tokenIdx >= num_tokens) return;

    bool const isQ = localHeadIdx < num_heads_q;
    int const headIdx = isQ ? localHeadIdx : localHeadIdx - num_heads_q;

    int const num_heads = num_heads_q + num_heads_k + num_heads_v;

    static_assert(head_dim % (32 * 2) == 0,
                  "head_dim must be divisible by 64 (each warp processes one "
                  "head, and each thread gets even number of "
                  "elements)");
    constexpr int numElemsPerThread = head_dim / 32;
    float elements[numElemsPerThread];
    constexpr int elemSizeBytes = numElemsPerThread * sizeof(__nv_bfloat16);
    static_assert(elemSizeBytes % 4 == 0,
                  "numSizeBytes must be a multiple of 4");
    constexpr int vecSize =
        elemSizeBytes /
        4;  // Use packed_as<uint, vecSize> to perform loading/saving.
    using vec_T = typename tensorrt_llm::common::packed_as<uint, vecSize>::type;

    int offsetWarp;  // Offset for the warp
    if (isQ) {
      // Q segment: token offset + head offset within Q segment
      offsetWarp = tokenIdx * num_heads * head_dim + headIdx * head_dim;
    } else {
      // K segment: token offset + entire Q segment + head offset within K
      // segment
      offsetWarp = tokenIdx * num_heads * head_dim + num_heads_q * head_dim +
                   headIdx * head_dim;
    }
    int offsetThread = offsetWarp + laneId * numElemsPerThread;

    // Sum of squares for RMSNorm
    float sumOfSquares = 0.0f;

    // Load.
    {
      vec_T vec = *reinterpret_cast<vec_T const*>(&qkv[offsetThread]);
      constexpr int num_packed_elems = elemSizeBytes / sizeof(T2_in);
#pragma unroll
      for (int i = 0; i < num_packed_elems; i++) {
        // Interpret the generic vector chunk as the specific packed type
        T2_in packed_val = *(reinterpret_cast<T2_in*>(&vec) + i);
        // Convert to float2 for computation
        float2 vals = Converter::convert(packed_val);
        sumOfSquares += vals.x * vals.x;
        sumOfSquares += vals.y * vals.y;

        elements[2 * i] = vals.x;
        elements[2 * i + 1] = vals.y;
      }
    }

    // Reduce sum across warp using the utility function
    sumOfSquares = tensorrt_llm::common::warpReduceSum(sumOfSquares);

    // Compute RMS normalization factor
    float rms_rcp = rsqrtf(sumOfSquares / static_cast<float>(head_dim) + eps);

    // Normalize elements
#pragma unroll
    for (int i = 0; i < numElemsPerThread; i++) {
      int dim = laneId * numElemsPerThread + i;
      float weight = isQ ? Converter::convert(q_weight[dim])
                         : Converter::convert(k_weight[dim]);
      elements[i] *= rms_rcp * weight;
    }
```
**EN:** fusedQKNormRopeKernel assigns one warp to one Q or K head, loads the head tile from the flattened QKV buffer, accumulates the sum of squares across the warp, and applies RMS normalization with either q_weight or k_weight depending on whether the warp is processing Q or K.
**CN:** fusedQKNormRopeKernel 为每个 Q 或 K 头分配一个 warp，从扁平化的 QKV 缓冲区加载该头的数据，在 warp 内归约平方和，并根据当前处理的是 Q 还是 K，分别使用 q_weight 或 k_weight 执行 RMSNorm。

### Baseline kernel: RoPE and store / 基础内核：RoPE 与回写
```cpp
    // Apply RoPE to normalized elements
    float elements2[numElemsPerThread];  // Additional buffer required for RoPE.

    int64_t pos_id = position_ids[tokenIdx];

    // Calculate cache pointer for this position - similar to
    // pos_encoding_kernels.cu
    T_cache const* cache_ptr = cos_sin_cache + pos_id * rotary_dim;
    int const embed_dim = rotary_dim / 2;
    T_cache const* cos_ptr = cache_ptr;
    T_cache const* sin_ptr = cache_ptr + embed_dim;
    int const rotary_lanes = rotary_dim / numElemsPerThread;  // rotary range
    if (laneId < rotary_lanes) {
      if constexpr (interleave) {
        // Perform interleaving. Use pre-computed cos/sin values.
#pragma unroll
        for (int i = 0; i < numElemsPerThread / 2; ++i) {
          int const idx0 = 2 * i;
          int const idx1 = 2 * i + 1;
          // Global dimension index in the head
          int const dim_idx = laneId * numElemsPerThread + idx0;

          float const val0 = elements[idx0];
          float const val1 = elements[idx1];

          int const half_dim = dim_idx / 2;
          float const cos_val =
              CacheConverter::convert(VLLM_LDG(cos_ptr + half_dim));
          float const sin_val =
              CacheConverter::convert(VLLM_LDG(sin_ptr + half_dim));

          elements[idx0] = val0 * cos_val - val1 * sin_val;
          elements[idx1] = val0 * sin_val + val1 * cos_val;
        }
      } else {
        // Before data exchange with in warp, we need to sync.
        __syncwarp();
        int pairOffset = (rotary_dim / 2) / numElemsPerThread;
        // Get the data from the other half of the warp. Use pre-computed
        // cos/sin values.
#pragma unroll
        for (int i = 0; i < numElemsPerThread; i++) {
          elements2[i] = __shfl_xor_sync(FINAL_MASK, elements[i], pairOffset);

          if (laneId < pairOffset) {
            elements2[i] = -elements2[i];
          }
          int dim_idx = laneId * numElemsPerThread + i;

          dim_idx = (dim_idx * 2) % rotary_dim;
          int half_dim = dim_idx / 2;
          float cos_val = CacheConverter::convert(VLLM_LDG(cos_ptr + half_dim));
          float sin_val = CacheConverter::convert(VLLM_LDG(sin_ptr + half_dim));

          elements[i] = elements[i] * cos_val + elements2[i] * sin_val;
        }
        // __shfl_xor_sync does not provide memfence. Need to sync again.
        __syncwarp();
      }
    }
    // Store.
    {
      vec_T vec;
      constexpr int num_packed_elems = elemSizeBytes / sizeof(T2_in);
#pragma unroll
      for (int i = 0; i < num_packed_elems; i++) {
        // Convert from float2 back to the specific packed type
        T2_in packed_val = Converter::convert(
            make_float2(elements[2 * i], elements[2 * i + 1]));
        // Place it into the generic vector
        *(reinterpret_cast<T2_in*>(&vec) + i) = packed_val;
      }
      *reinterpret_cast<vec_T*>(&qkv[offsetThread]) = vec;
    }
```
**EN:** After normalization, the baseline kernel reads the position-specific cos/sin cache, applies either GPT-J interleaved or Neox-style paired rotation, and writes the transformed values back into the original QKV buffer in place.
**CN:** 归一化完成后，基础内核读取与位置对应的 cos/sin cache，按 GPT-J 交错式或 Neox 风格配对旋转应用 RoPE，然后把变换后的值原地写回原始 QKV 缓冲区。

### NTokenHeads kernel: async preload into shared memory / NTokenHeads 内核：异步预取到共享内存
```cpp
template <typename scalar_t_in, typename scalar_t_cache, int head_dim,
          bool interleave, int HEADS_PER_WARP>
__global__ void fusedQKNormRopeKernelNTokenHeads(
    void* qkv_void, int const num_heads_q, int const num_heads_k,
    int const num_heads_v, float const eps, void const* q_weight_void,
    void const* k_weight_void, void const* cos_sin_cache_void,
    int64_t const* position_ids, int const num_tokens, int const rotary_dim) {
#if (!defined(__CUDA_ARCH__) || __CUDA_ARCH__ < 800) && !defined(USE_ROCM)
  if constexpr ((std::is_same_v<scalar_t_in, c10::BFloat16>) ||
                std::is_same_v<scalar_t_cache, c10::BFloat16>) {
    return;
  } else {
#endif

    using Converter = vllm::_typeConvert<scalar_t_in>;
    static_assert(Converter::exists,
                  "Input QKV data type is not supported for this CUDA "
                  "architecture or toolkit version.");
    using T_in = typename Converter::hip_type;
    using T2_in = typename Converter::packed_hip_type;

    using CacheConverter = vllm::_typeConvert<scalar_t_cache>;
    static_assert(CacheConverter::exists,
                  "Cache data type is not supported for this CUDA architecture "
                  "or toolkit version.");
    using T_cache = typename CacheConverter::hip_type;

    extern __shared__ char smem_storage[];
    // Shared memory layout:
    //   [0, cos_sin_bytes)           : cos/sin for each warp  (warpsPerBlock *
    //   rotary_dim * sizeof(T_cache))
    // [cos_sin_bytes, ...)         : QKV tiles
    //   per warp     (warpsPerBlock * HEADS_PER_WARP * 32 * elemSizeBytes)
    T_cache* const smem = reinterpret_cast<T_cache*>(smem_storage);

    T_in* qkv = reinterpret_cast<T_in*>(qkv_void);
    T_in const* q_weight = reinterpret_cast<T_in const*>(q_weight_void);
    T_in const* k_weight = reinterpret_cast<T_in const*>(k_weight_void);
    T_cache const* cos_sin_cache =
        reinterpret_cast<T_cache const*>(cos_sin_cache_void);

    int const warpsPerBlock = blockDim.x / 32;
    int const warpId = threadIdx.x / 32;
    int const laneId = threadIdx.x % 32;

    int const total_qk_heads = num_heads_q + num_heads_k;
    int const num_heads = num_heads_q + num_heads_k + num_heads_v;
    int const head_chunks_per_token =
        (total_qk_heads + HEADS_PER_WARP - 1) / HEADS_PER_WARP;

    int const warp_global = blockIdx.x * warpsPerBlock + warpId;
    int const tokenIdx = warp_global / head_chunks_per_token;
    int const headChunk = warp_global % head_chunks_per_token;
    int const first_head = headChunk * HEADS_PER_WARP;
    int const num_heads_this_warp =
        (first_head + HEADS_PER_WARP <= total_qk_heads)
            ? HEADS_PER_WARP
            : (total_qk_heads - first_head);

    if (tokenIdx >= num_tokens) return;

    static_assert(head_dim % (32 * 2) == 0, "head_dim must be divisible by 64");
    constexpr int numElemsPerThread = head_dim / 32;
    constexpr int elemSizeBytes = numElemsPerThread * sizeof(__nv_bfloat16);
    static_assert(elemSizeBytes % 4 == 0,
                  "elemSizeBytes must be a multiple of 4");
    constexpr int vecSize = elemSizeBytes / 4;
    using vec_T = typename tensorrt_llm::common::packed_as<uint, vecSize>::type;

    int const cos_sin_bytes =
        warpsPerBlock * rotary_dim * static_cast<int>(sizeof(T_cache));
    int const qkv_tile_bytes = 32 * elemSizeBytes;
    char* const this_warp_head_smem =
        smem_storage + cos_sin_bytes +
        warpId * (HEADS_PER_WARP * qkv_tile_bytes);

    // === Group 0: async load all heads' QKV into smem (issued first). ===
    for (int k = 0; k < num_heads_this_warp; ++k) {
      int const localHeadIdx = first_head + k;
      bool const isQ = localHeadIdx < num_heads_q;
      int const headIdx = isQ ? localHeadIdx : localHeadIdx - num_heads_q;
      int offWarp;
      if (isQ) {
        offWarp = tokenIdx * num_heads * head_dim + headIdx * head_dim;
      } else {
        offWarp = tokenIdx * num_heads * head_dim + num_heads_q * head_dim +
                  headIdx * head_dim;
      }
      int const offThread = offWarp + laneId * numElemsPerThread;
      char* smem_dst =
          this_warp_head_smem + k * qkv_tile_bytes + laneId * elemSizeBytes;
      cp_async_shared_global_ca(smem_dst,
                                reinterpret_cast<const char*>(&qkv[offThread]),
                                elemSizeBytes);
    }
    cp_async_commit_group();  // commit group 0 (QKV)

    // === Group 1: async load cos/sin into smem (issued second). ===
    int64_t const pos_id = position_ids[tokenIdx];
    T_cache const* const cache_ptr = cos_sin_cache + pos_id * rotary_dim;
    int const copy_bytes = rotary_dim * static_cast<int>(sizeof(T_cache));
    int const num_copies = (copy_bytes + 15) / 16;
    for (int copyId = laneId; copyId < num_copies; copyId += 32) {
      char* smem_ptr =
          reinterpret_cast<char*>(&smem[warpId * rotary_dim]) + copyId * 16;
      const char* glob_ptr =
          reinterpret_cast<const char*>(cache_ptr) + copyId * 16;
      cp_async_shared_global_16_cg(smem_ptr, glob_ptr);
    }
    cp_async_commit_group();  // commit group 1 (cos/sin)

    // wait<1>: allow at most 1 pending group (group 1) → group 0 (QKV) is done.
    cp_async_wait_group<1>();
```
**EN:** The optimized variant lets one warp process multiple heads for the same token. It first cp.async-loads several Q/K head tiles and the shared cos/sin cache into shared memory so later computation can reuse the cache data across multiple heads.
**CN:** 优化版本让一个 warp 处理同一 token 的多个头。它先通过 cp.async 把多个 Q/K 头 tile 以及共享的 cos/sin cache 预取到共享内存，后续计算便可在多个头之间复用这些 cache 数据。

### NTokenHeads kernel: per-head compute loop / NTokenHeads 内核：逐头计算循环
```cpp
    for (int k = 0; k < num_heads_this_warp; ++k) {
      int const localHeadIdx = first_head + k;
      bool const isQ = localHeadIdx < num_heads_q;
      int const headIdx = isQ ? localHeadIdx : localHeadIdx - num_heads_q;

      int offsetWarp;
      if (isQ) {
        offsetWarp = tokenIdx * num_heads * head_dim + headIdx * head_dim;
      } else {
        offsetWarp = tokenIdx * num_heads * head_dim + num_heads_q * head_dim +
                     headIdx * head_dim;
      }
      int const offsetThread = offsetWarp + laneId * numElemsPerThread;

      // === Part 1: QK Norm (read from smem; group 0 already done). ===
      float sumOfSquares = 0.0f;
      {
        char const* smem_src =
            this_warp_head_smem + k * qkv_tile_bytes + laneId * elemSizeBytes;
        vec_T vec = *reinterpret_cast<vec_T const*>(smem_src);
        constexpr int num_packed_elems = elemSizeBytes / sizeof(T2_in);
#pragma unroll
        for (int i = 0; i < num_packed_elems; i++) {
          T2_in packed_val = *(reinterpret_cast<T2_in*>(&vec) + i);
          float2 vals = Converter::convert(packed_val);
          sumOfSquares += vals.x * vals.x;
          sumOfSquares += vals.y * vals.y;
          elements[2 * i] = vals.x;
          elements[2 * i + 1] = vals.y;
        }
      }

      sumOfSquares = tensorrt_llm::common::warpReduceSum(sumOfSquares);
      float rms_rcp = rsqrtf(sumOfSquares / static_cast<float>(head_dim) + eps);

#pragma unroll
      for (int i = 0; i < numElemsPerThread; i++) {
        elements[i] *= rms_rcp * (isQ ? q_w[i] : k_w[i]);
      }

      // On first head: wait for group 1 (cos/sin) before RoPE.
      if (k == 0) cp_async_wait_group<0>();

      // === Part 2: RoPE using cos/sin from shared memory. ===
      if (laneId < rotary_lanes) {
        if constexpr (interleave) {
#pragma unroll
          for (int i = 0; i < numElemsPerThread / 2; ++i) {
            int const idx0 = 2 * i;
            int const idx1 = 2 * i + 1;
            int const dim_idx = laneId * numElemsPerThread + idx0;
            float const val0 = elements[idx0];
            float const val1 = elements[idx1];
            int const half_dim = dim_idx / 2;
            float const cos_val = CacheConverter::convert(cos_smem[half_dim]);
            float const sin_val = CacheConverter::convert(sin_smem[half_dim]);
            elements[idx0] = val0 * cos_val - val1 * sin_val;
            elements[idx1] = val0 * sin_val + val1 * cos_val;
          }
        } else {
          __syncwarp();
          int const pairOffset = (rotary_dim / 2) / numElemsPerThread;
#pragma unroll
          for (int i = 0; i < numElemsPerThread; i++) {
            elements2[i] = __shfl_xor_sync(FINAL_MASK, elements[i], pairOffset);
            if (laneId < pairOffset) elements2[i] = -elements2[i];
            int dim_idx = laneId * numElemsPerThread + i;
            dim_idx = (dim_idx * 2) % rotary_dim;
            int const half_dim = dim_idx / 2;
            float const cos_val = CacheConverter::convert(cos_smem[half_dim]);
            float const sin_val = CacheConverter::convert(sin_smem[half_dim]);
            elements[i] = elements[i] * cos_val + elements2[i] * sin_val;
          }
          __syncwarp();
        }
      }

      // Store.
      {
        vec_T vec;
        constexpr int num_packed_elems = elemSizeBytes / sizeof(T2_in);
#pragma unroll
        for (int i = 0; i < num_packed_elems; i++) {
          T2_in packed_val = Converter::convert(
              make_float2(elements[2 * i], elements[2 * i + 1]));
          *(reinterpret_cast<T2_in*>(&vec) + i) = packed_val;
        }
        *reinterpret_cast<vec_T*>(&qkv[offsetThread]) = vec;
      }
```
**EN:** Inside the head loop, the warp reads each head tile from shared memory, performs RMSNorm, waits for cos/sin readiness on the first head, applies RoPE, and writes the transformed tile back to global memory. Reusing q_weight/k_weight registers and shared cos/sin lowers memory pressure.
**CN:** 在逐头循环中，warp 从共享内存读取每个头的 tile，执行 RMSNorm，在第一个头上等待 cos/sin 数据就绪，再应用 RoPE，并把结果写回全局内存。对 q_weight/k_weight 寄存器和共享 cos/sin 的复用降低了内存压力。

### Launch helpers for base and multi-head variants / 基础版与多头版的启动辅助逻辑
```cpp
template <typename scalar_t_in, typename scalar_t_cache>
void launchFusedQKNormRope(void* qkv, int const num_tokens,
                           int const num_heads_q, int const num_heads_k,
                           int const num_heads_v, int const head_dim,
                           int const rotary_dim, float const eps,
                           void const* q_weight, void const* k_weight,
                           void const* cos_sin_cache, bool const interleave,
                           int64_t const* position_ids, cudaStream_t stream) {
  constexpr int blockSize = 256;
  int const warpsPerBlock = blockSize / 32;
  int const totalQKHeads = num_heads_q + num_heads_k;
  int const totalWarps = num_tokens * totalQKHeads;
  int const gridSize = common::divUp(totalWarps, warpsPerBlock);
  dim3 gridDim(gridSize);
  dim3 blockDim(blockSize);
  switch (head_dim) {
    case 64:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<scalar_t_in, scalar_t_cache, 64, INTERLEAVE>
            <<<gridDim, blockDim, 0, stream>>>(
                qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight,
                k_weight, cos_sin_cache, position_ids, num_tokens, rotary_dim);
      });
      break;
    case 128:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<scalar_t_in, scalar_t_cache, 128, INTERLEAVE>
            <<<gridDim, blockDim, 0, stream>>>(
                qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight,
                k_weight, cos_sin_cache, position_ids, num_tokens, rotary_dim);
      });
      break;
    case 256:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<scalar_t_in, scalar_t_cache, 256, INTERLEAVE>
            <<<gridDim, blockDim, 0, stream>>>(
                qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight,
                k_weight, cos_sin_cache, position_ids, num_tokens, rotary_dim);
      });
      break;
    default:
      TORCH_CHECK(false,
                  "Unsupported head dimension for fusedQKNormRope: ", head_dim);
  }
}

// Launch: one warp processes token_heads_per_warp token-heads (1, 2, 4, or 8).
// When token_heads_per_warp == 1, delegates to the 1-head baseline above.
template <typename scalar_t_in, typename scalar_t_cache>
void launchFusedQKNormRopeNTokenHeads(
    void* qkv, int const num_tokens, int const num_heads_q,
    int const num_heads_k, int const num_heads_v, int const head_dim,
    int const rotary_dim, float const eps, void const* q_weight,
    void const* k_weight, void const* cos_sin_cache, bool const interleave,
    int64_t const* position_ids, int const token_heads_per_warp,
    cudaStream_t stream) {
  TORCH_CHECK(token_heads_per_warp == 1 || token_heads_per_warp == 2 ||
                  token_heads_per_warp == 4 || token_heads_per_warp == 8,
              "token_heads_per_warp must be 1, 2, 4, or 8, got ",
              token_heads_per_warp);

  // token_heads_per_warp == 1: delegate to the 1-head baseline kernel.
  if (token_heads_per_warp == 1) {
    launchFusedQKNormRope<scalar_t_in, scalar_t_cache>(
        qkv, num_tokens, num_heads_q, num_heads_k, num_heads_v, head_dim,
        rotary_dim, eps, q_weight, k_weight, cos_sin_cache, interleave,
        position_ids, stream);
    return;
  }

  // NTokenHeads kernel uses cp.async to load cos/sin in 16-byte chunks.
  // If rotary_dim * sizeof(cache_dtype) is not a multiple of 16, the last
  // cp.async would write past the shared memory allocation.
  // Fall back to the base kernel instead of failing.
  {
    size_t const rotary_bytes =
        static_cast<size_t>(rotary_dim) *
        (std::is_same_v<scalar_t_cache, float> ? sizeof(float) : 2u);
    if (rotary_bytes % 16 != 0) {
      launchFusedQKNormRope<scalar_t_in, scalar_t_cache>(
          qkv, num_tokens, num_heads_q, num_heads_k, num_heads_v, head_dim,
          rotary_dim, eps, q_weight, k_weight, cos_sin_cache, interleave,
          position_ids, stream);
      return;
    }
  }

  constexpr int blockSize = 256;
  int const warpsPerBlock = blockSize / 32;
  int const totalQKHeads = num_heads_q + num_heads_k;
  // Grid: one warp per (token, head_chunk); same token → reuse cos/sin in smem.
  int const head_chunks_per_token =
      (totalQKHeads + token_heads_per_warp - 1) / token_heads_per_warp;
  int const total_warps = num_tokens * head_chunks_per_token;
  int const gridSize = common::divUp(total_warps, warpsPerBlock);
  dim3 gridDim(gridSize);
  dim3 blockDim(blockSize);
  // Cache element size: float=4, bfloat16=2 (host-safe; kernel uses same
  // layout).
  size_t const cache_elem_size =
      std::is_same_v<scalar_t_cache, float> ? sizeof(float) : 2u;
  // QKV smem: token_heads_per_warp tiles per warp, each tile 32*(head_dim/32*2)
  // = 2*head_dim bytes.
  size_t const qkv_smem_per_warp = static_cast<size_t>(token_heads_per_warp) *
                                   2u * static_cast<size_t>(head_dim);
  size_t const smem_bytes =
      warpsPerBlock * static_cast<size_t>(rotary_dim) * cache_elem_size +
      warpsPerBlock * qkv_smem_per_warp;

#define LAUNCH_N_TOKEN_HEADS(N)                                              \
  do {                                                                       \
    switch (head_dim) {                                                      \
      case 64:                                                               \
        DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {                        \
          fusedQKNormRopeKernelNTokenHeads<scalar_t_in, scalar_t_cache, 64,  \
                                           INTERLEAVE, (N)>                  \
              <<<gridDim, blockDim, smem_bytes, stream>>>(                   \
                  qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight, \
                  k_weight, cos_sin_cache, position_ids, num_tokens,         \
                  rotary_dim);                                               \
        });                                                                  \
        break;                                                               \
      case 128:                                                              \
        DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {                        \
          fusedQKNormRopeKernelNTokenHeads<scalar_t_in, scalar_t_cache, 128, \
                                           INTERLEAVE, (N)>                  \
              <<<gridDim, blockDim, smem_bytes, stream>>>(                   \
                  qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight, \
                  k_weight, cos_sin_cache, position_ids, num_tokens,         \
                  rotary_dim);                                               \
        });                                                                  \
        break;                                                               \
      case 256:                                                              \
        DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {                        \
          fusedQKNormRopeKernelNTokenHeads<scalar_t_in, scalar_t_cache, 256, \
                                           INTERLEAVE, (N)>                  \
              <<<gridDim, blockDim, smem_bytes, stream>>>(                   \
                  qkv, num_heads_q, num_heads_k, num_heads_v, eps, q_weight, \
                  k_weight, cos_sin_cache, position_ids, num_tokens,         \
                  rotary_dim);                                               \
        });                                                                  \
        break;                                                               \
      default:                                                               \
        TORCH_CHECK(false, "Unsupported head dimension: ", head_dim);        \
    }                                                                        \
  } while (0)

  if (token_heads_per_warp == 2) {
    LAUNCH_N_TOKEN_HEADS(2);
  } else if (token_heads_per_warp == 4) {
    LAUNCH_N_TOKEN_HEADS(4);
  } else if (token_heads_per_warp == 8) {
    LAUNCH_N_TOKEN_HEADS(8);
  }
#undef LAUNCH_N_TOKEN_HEADS
}
```
**EN:** launchFusedQKNormRope chooses a head-dimension specialization for the baseline kernel, while launchFusedQKNormRopeNTokenHeads validates token_heads_per_warp, falls back when rotary cache bytes are not 16-byte aligned, computes shared-memory requirements, and launches a HEADS_PER_WARP specialization of the optimized kernel.
**CN:** launchFusedQKNormRope 为基础内核选择 head_dim 特化；launchFusedQKNormRopeNTokenHeads 则负责校验 token_heads_per_warp，在 rotary cache 字节数无法按 16 字节对齐时回退，计算共享内存需求，并启动相应 HEADS_PER_WARP 特化的优化内核。

### Torch wrapper and heuristic auto-tuning / Torch 包装器与启发式自动调优
```cpp
void fused_qk_norm_rope(
    torch::Tensor& qkv,       // Combined QKV tensor [num_tokens,
                              // (num_heads_q+num_heads_k+num_heads_v)*head_dim]
    int64_t num_heads_q,      // Number of query heads
    int64_t num_heads_k,      // Number of key heads
    int64_t num_heads_v,      // Number of value heads
    int64_t head_dim,         // Dimension per head
    double eps,               // Epsilon for RMS normalization
    torch::Tensor& q_weight,  // RMSNorm weights for query [head_dim]
    torch::Tensor& k_weight,  // RMSNorm weights for key [head_dim]
    torch::Tensor& cos_sin_cache,  // Cos/sin cache [max_position, head_dim]
    bool is_neox,                  // Whether RoPE is applied in Neox style
    torch::Tensor& position_ids,   // Position IDs for RoPE [num_tokens]
    int64_t forced_token_heads_per_warp  // -1 = auto-select, >0 = forced value
) {
  // Input validation
  CHECK_INPUT(qkv);
  CHECK_INPUT(position_ids);
  CHECK_INPUT(q_weight);
  CHECK_INPUT(k_weight);
  CHECK_INPUT(cos_sin_cache);
  CHECK_TYPE(position_ids, torch::kInt64);

  TORCH_CHECK(qkv.dim() == 2,
              "QKV tensor must be 2D: [num_tokens, "
              "(num_heads_q+num_heads_k+num_heads_v)*head_dim]");
  TORCH_CHECK(position_ids.dim() == 1, "Position IDs must be 1D: [num_tokens]");
  TORCH_CHECK(q_weight.dim() == 1, "Query weights must be 1D: [head_dim]");
  TORCH_CHECK(k_weight.dim() == 1, "Key weights must be 1D: [head_dim]");
  TORCH_CHECK(cos_sin_cache.dim() == 2,
              "Cos/sin cache must be 2D: [max_position, head_dim]");
  TORCH_CHECK(q_weight.size(0) == head_dim,
              "Query weights size must match head dimension");
  TORCH_CHECK(k_weight.size(0) == head_dim,
              "Key weights size must match head dimension");

  TORCH_CHECK(cos_sin_cache.size(1) % 2 == 0, "rotary_dim must be even");
  TORCH_CHECK(cos_sin_cache.size(1) <= head_dim,
              "rotary_dim must be less than or equal to head_dim");

  TORCH_CHECK(qkv.scalar_type() == q_weight.scalar_type() &&
                  qkv.scalar_type() == k_weight.scalar_type(),
              "qkv, q_weight and k_weight must have the same dtype");

  int64_t num_tokens = qkv.size(0);
  TORCH_CHECK(position_ids.size(0) == num_tokens,
              "Number of tokens in position_ids must match QKV");

  int64_t total_heads = num_heads_q + num_heads_k + num_heads_v;
  TORCH_CHECK(
      qkv.size(1) == total_heads * head_dim,
      "QKV tensor size must match total number of heads and head dimension");

  auto device_id = qkv.get_device();
  auto stream = at::cuda::getCurrentCUDAStream(device_id);

  // Select token_heads_per_warp: forced value if >0, else auto-select.
  // Auto thresholds are calibrated on SM 9.0 (H100). On other architectures,
  // fall back to token_heads_per_warp=1 (base kernel) until profiled.
  int token_heads_per_warp;
  if (forced_token_heads_per_warp > 0) {  // only support SM80+
    token_heads_per_warp = static_cast<int>(forced_token_heads_per_warp);
  } else {
    token_heads_per_warp = 1;
    auto* dev_prop = at::cuda::getDeviceProperties(device_id);
    int sm_version = dev_prop->major * 10 + dev_prop->minor;
    int64_t total_qk_units = num_tokens * (num_heads_q + num_heads_k);
    if (sm_version == 90) {
      if (head_dim >= 256) {
        if (total_qk_units < 4096LL) {
          token_heads_per_warp = 1;
        } else if (total_qk_units < 8192LL) {
          token_heads_per_warp = 2;
        } else {
          token_heads_per_warp = 4;
        }
      } else {
        if (total_qk_units < 10240LL) {
          token_heads_per_warp = 1;
        } else if (total_qk_units < 40960LL) {
          token_heads_per_warp = 4;
        } else {
          token_heads_per_warp = 8;
        }
      }
    }
  }

  VLLM_DISPATCH_HALF_TYPES(qkv.scalar_type(), "fused_qk_norm_rope_kernel", [&] {
    using qkv_scalar_t = scalar_t;
    VLLM_DISPATCH_FLOATING_TYPES(
        cos_sin_cache.scalar_type(), "fused_qk_norm_rope_kernel", [&] {
          using cache_scalar_t = scalar_t;
          tensorrt_llm::kernels::launchFusedQKNormRopeNTokenHeads<
              qkv_scalar_t, cache_scalar_t>(
              qkv.data_ptr(), static_cast<int>(num_tokens),
              static_cast<int>(num_heads_q), static_cast<int>(num_heads_k),
              static_cast<int>(num_heads_v), static_cast<int>(head_dim),
              static_cast<int>(cos_sin_cache.size(1)), static_cast<float>(eps),
              q_weight.data_ptr(), k_weight.data_ptr(),
              cos_sin_cache.data_ptr(), !is_neox,
              reinterpret_cast<int64_t const*>(position_ids.data_ptr()),
              token_heads_per_warp, stream);
        });
  });
}
```
**EN:** The exported fused_qk_norm_rope validates QKV layout, weight/cache shapes, and dtypes, then auto-selects token_heads_per_warp based on SM version, head_dim, and num_tokens*(num_heads_q+num_heads_k). The tuned thresholds are currently only enabled for SM90; other architectures default to the conservative base kernel unless forced.
**CN:** 导出的 fused_qk_norm_rope 会校验 QKV 布局、权重/缓存形状以及 dtype，然后根据 SM 版本、head_dim 和 num_tokens*(num_heads_q+num_heads_k) 自动选择 token_heads_per_warp。当前这些阈值只针对 SM90 启用；其他架构除非强制指定，否则都会回退到更保守的基础内核。

## Key Concepts / 关键概念

- **EN:** The kernel only transforms Q and K heads; V heads are present in the flattened tensor layout but are not modified.
  **CN:** 该内核只变换 Q 和 K 头；V 头虽然存在于扁平化布局中，但不会被修改。
- **EN:** cp.async is used as a latency-hiding mechanism when multiple heads of the same token share the same RoPE cache row.
  **CN:** 当同一 token 的多个头共享同一行 RoPE cache 时，cp.async 被用作隐藏延迟的机制。
- **EN:** The wrapper exposes a manual override for token_heads_per_warp so heuristics can be bypassed for tuning or debugging.
  **CN:** 包装器暴露了 token_heads_per_warp 的手动覆盖参数，便于调优或调试时绕过自动启发式。

## Dependencies / 依赖关系

- **EN:** Uses async_util.cuh for cp.async helpers and cuda_compat.h/type_convert.cuh for portable conversion primitives.
  **CN:** 依赖 async_util.cuh 提供 cp.async 辅助函数，并使用 cuda_compat.h/type_convert.cuh 提供可移植的转换原语。
- **EN:** Uses dispatch_utils.h to dispatch both QKV dtype and cache dtype.
  **CN:** 依赖 dispatch_utils.h 同时分发 QKV dtype 与 cache dtype。
- **EN:** Adapted from TensorRT-LLM and therefore mirrors some of its helper namespaces and kernel structure.
  **CN:** 该实现改编自 TensorRT-LLM，因此保留了其中部分辅助命名空间与内核结构。
