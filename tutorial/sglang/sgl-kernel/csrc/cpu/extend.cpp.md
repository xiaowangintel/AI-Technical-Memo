# extend.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/extend.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Namespace and shared declarations
```cpp
#include "common.h"
#include "flash_attn.h"
#include "gemm.h"

namespace {

// [NOTE]: extend attention for CPU
//   1. BLOCK_M and BLOCK_N tuned for various seq lengths
//   2. can handle non-contiguous k_extend and v_extend
//   3. computes attention for prefix and extend separately
//   4. TODO: apply head dimension blocking to optimize GQA
//

template <typename scalar_t, typename index_t, int BLOCK_M, int BLOCK_N>
void extend_attention_kernel_impl(
    scalar_t* __restrict__ o_extend,
    const scalar_t* __restrict__ q_extend,
    const scalar_t* __restrict__ k_extend,
    const scalar_t* __restrict__ v_extend,
    const scalar_t* __restrict__ k_buffer,
    const scalar_t* __restrict__ v_buffer,
    const index_t* __restrict__ req_to_token,
    const int64_t* __restrict__ req_pool_indices,
    const int64_t* __restrict__ seq_lens,
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 25-53: Local implementation details
```cpp
    const index_t* __restrict__ extend_seq_lens,
    const index_t* __restrict__ extend_start_loc,
    const void* __restrict__ buffer,
    int batches,
    int num_heads,
    int num_heads_kv,
    int head_size,
    int head_size_v,
    int q_strideM,
    int q_strideH,
    int ke_strideN,
    int ke_strideH,
    int ve_strideN,
    int ve_strideH,
    int k_strideN,
    int k_strideH,
    int v_strideN,
    int v_strideH,
    float sm_scale,
    int max_num_reqs,
    int max_context_len,
    int max_total_num_tokens,
    int max_len_extend,
    int buffer_size_per_thread,
    bool is_prefix_skipped) {
  // strides
  const int o_strideM = num_heads * head_size_v;
  const int o_strideH = head_size_v;
```
**EN:** This section fills in the local implementation details around `o_strideM`, `o_strideH`, completing the behavior required by the file.
**CN:** 本段补充了`o_strideM`、`o_strideH`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 54-78: Runtime integration and dispatch
```cpp
  // we use same buffer for packed key and value
  const int ldb_tmp = std::max(head_size, head_size_v);

  const int num_groups = num_heads / num_heads_kv;
  TORCH_CHECK(num_groups * num_heads_kv == num_heads);

  // number of blocks along M
  int MB = div_up(max_len_extend, BLOCK_M);

  // parallel on [batches, num_heads, BM]
  at::parallel_for(0, batches * num_heads * MB, 0, [&](int begin, int end) {
    int bs{0}, head_id{0}, mb{0};
    data_index_init(begin, bs, batches, head_id, num_heads, mb, MB);

    int tid = at::get_thread_num();
    // s_i and s_delta: [BLOCK_M, BLOCK_N]
    float* __restrict__ s_i = reinterpret_cast<float*>((char*)(buffer) + tid * buffer_size_per_thread);
    scalar_t* __restrict__ s_delta = reinterpret_cast<scalar_t*>(s_i);

    // v_prime: [BLOCK_M, head_size_v]
    float* __restrict__ v_prime = s_i + BLOCK_M * BLOCK_N;

    // Btmp: [BLOCK_N, max(head_size, head_size_v)]
    scalar_t* __restrict__ Btmp = reinterpret_cast<scalar_t*>(v_prime + BLOCK_M * head_size_v);
```
**EN:** This section uses `at::parallel_for`, `max`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`at::parallel_for`、`max`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 79-101: Runtime integration and dispatch
```cpp
    // init Btmp just once for each thread to prevent NaN
    fill_stub(Btmp, 0.f, BLOCK_N * ldb_tmp);

    alignas(64) float s_prime[BLOCK_M];
    alignas(64) float m_prime[BLOCK_M];

    for (int i = begin; i < end; ++i) {
      // seq_len = prefix + extend
      int head_kv_id = head_id / num_groups;
      int seq_len = seq_lens[bs];
      int seq_len_extend = extend_seq_lens[bs];
      int seq_len_prefix = seq_len - seq_len_extend;
      int seq_extend_start_loc = extend_start_loc[bs];

      int req_pool_id = req_pool_indices[bs];
      TORCH_CHECK(seq_len_prefix >= 0, "prefix len < 0!");
      TORCH_CHECK(seq_len <= max_context_len, "seq_len out of scope!");
      TORCH_CHECK(req_pool_id < max_num_reqs, "req_pool_id out of scope!");

      if (is_prefix_skipped) {
        TORCH_CHECK(seq_len_prefix == 0, "extend attention: expect seq_len_prefix to be 0, got ", seq_len_prefix);
      }
```
**EN:** This section uses `fill_stub`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fill_stub`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 102-125: Control flow and branching
```cpp
      // offset and size in MB
      int m = mb * BLOCK_M;
      int m_size = std::min(BLOCK_M, seq_len_extend - m);

      if (m_size <= 0) {
        data_index_step(bs, batches, head_id, num_heads, mb, MB);
        continue;
      }

      // get query
      const scalar_t* __restrict__ q_ptr = q_extend + (seq_extend_start_loc + m) * q_strideM + head_id * q_strideH;

      // init v', s' and m'
      fill_stub(v_prime, 0.f, m_size * head_size_v);
      fill_stub(s_prime, 0.f, m_size);
      fill_stub(m_prime, -std::numeric_limits<scalar_t>::infinity(), m_size);

      // stage 1: compute scores with prefix
      for (int n = 0; n < seq_len_prefix; n += BLOCK_N) {
        int n_size = std::min(BLOCK_N, seq_len_prefix - n);

        // `n_size` is K in 2nd gemm, pad to TILE_K;
        const int padded_n_size = div_up(n_size, TILE_K) * TILE_K;
```
**EN:** This section drives `min`, `data_index_step`, `fill_stub` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`min`、`data_index_step`、`fill_stub`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 126-148: Runtime integration and dispatch
```cpp
        // get key and pack
        pack_vnni<scalar_t, index_t>(
            /*    dst */ Btmp,
            /*    src */ k_buffer + head_kv_id * k_strideH,
            /*    ind */ req_to_token + req_pool_id * max_context_len + n,
            /*     N  */ n_size,
            /*     K  */ head_size,
            /* ld_src */ k_strideN,
            /* ld_dst */ BLOCK_N);

        // calculate s_i <- Q @ K
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ head_size,
            /* lda   */ q_strideM,
            /* ldb   */ BLOCK_N,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ q_ptr,
            /* B     */ Btmp,
            /* C     */ s_i);
```
**EN:** This section uses `brgemm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`brgemm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 149-175: Runtime integration and dispatch
```cpp
        flash_attn_softmax<scalar_t, BLOCK_M, BLOCK_N>::apply(
            s_i, s_delta, v_prime, s_prime, m_prime, m_size, n_size, padded_n_size, head_size_v, sm_scale);

        // get value and pack
        pack_vnni2<scalar_t, index_t>(
            /*    dst */ Btmp,
            /*    src */ v_buffer + head_kv_id * v_strideH,
            /*    ind */ req_to_token + req_pool_id * max_context_len + n,
            /*     K  */ n_size,
            /*     N  */ head_size_v,
            /* ld_src */ v_strideN,
            /* ld_dst */ head_size_v);

        // calculate V' <- s_delta @ V + V'
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ head_size_v,
            /* K     */ padded_n_size,  // n_size
            /* lda   */ BLOCK_N,
            /* ldb   */ head_size_v,
            /* ldc   */ head_size_v,
            /* add_C */ true,
            /* A     */ s_delta,
            /* B     */ Btmp,
            /* C     */ v_prime);
      }  // loop with seq_len_prefix
```
**EN:** This section uses `apply`, `brgemm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`apply`、`brgemm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 176-205: Runtime integration and dispatch
```cpp
      // stage 2: compute the triangle part
      int num_keys = std::min(seq_len_extend, m + BLOCK_M);
      for (int n = 0; n < num_keys; n += BLOCK_N) {
        int n_size = std::min(BLOCK_N, num_keys - n);

        // `n_size` is K in 2nd gemm, pad to TILE_K;
        const int padded_n_size = div_up(n_size, TILE_K) * TILE_K;

        // get key and pack
        pack_vnni<scalar_t>(
            /*    dst */ Btmp,
            /*    src */ k_extend + (seq_extend_start_loc + n) * ke_strideN + head_kv_id * ke_strideH,
            /*     N  */ n_size,
            /*     K  */ head_size,
            /* ld_src */ ke_strideN,
            /* ld_dst */ BLOCK_N);

        // calculate s_i <- Q @ K
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ n_size,
            /* K     */ head_size,
            /* lda   */ q_strideM,
            /* ldb   */ BLOCK_N,
            /* ldc   */ BLOCK_N,
            /* add_C */ false,
            /* A     */ q_ptr,
            /* B     */ Btmp,
            /* C     */ s_i);
```
**EN:** This section uses `min`, `brgemm`, `padded_n_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`min`、`brgemm`、`padded_n_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 206-232: Control flow and branching
```cpp
        // apply causal mask
        // [Note] condition to apply causal mask.
        // Mask any block whose last key (n + n_size - 1) is strictly after the first query position (m), i.e. n +
        // n_size - 1 > m. The original condition was `num_keys - n <= BLOCK_N` (last n-block only). That was correct
        // when BLOCK_M <= BLOCK_N/2 because earlier n-blocks were guaranteed to contain only past keys.  With
        // BLOCK_M=512, BLOCK_N=768:
        //   BLOCK_M > BLOCK_N/2, so the first n-block can contain future keys.
        //   Example: m=512 (mb=1), num_keys=1024, first n-block covers keys [0, 768).
        //   Query row=0 is at position 512, so keys 513..767 are future and must be
        //   masked — but `num_keys - 0 = 1024 > BLOCK_N` skips masking entirely,
        //   producing wrong (non-causal) attention for rows 0..254 of this m-block.
        if (n + n_size - 1 > m) {
          for (int row = 0; row < m_size; ++row) {
            int last_col = m + row - n;
            // [Note] mask the entire row if last_col < 0.
            // Clamp to -1: when n > m + row every key in this block is a future
            // key, so the entire row should be masked.  Without this clamp,
            // last_col+1 <= 0 and fill_stub would write before row_ptr.
            // Example:
            //  For max_len_extend > 4096 → selects BLOCK_M=512, BLOCK_N=768
            //  m + BLOCK_M = 512 + 512 = 1024 > BLOCK_N = 768, this means we can have a a second n-block at n=768.
            //  For m = 512, row = 0, n = 768, last_col = 512 + 0 - 768 = -256 → out of bounds write in fill_stub
            last_col = std::max(last_col, -1);
            // fill [last_col + 1, n_size) to -inf
            float* row_ptr = s_i + row * BLOCK_N;
            fill_stub(row_ptr + last_col + 1, -std::numeric_limits<float>::infinity(), n_size - last_col - 1);
          }
```
**EN:** This section drives `max`, `fill_stub` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`max`、`fill_stub`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 233-260: Runtime integration and dispatch
```cpp
        }

        flash_attn_softmax<scalar_t, BLOCK_M, BLOCK_N>::apply(
            s_i, s_delta, v_prime, s_prime, m_prime, m_size, n_size, padded_n_size, head_size_v, sm_scale);

        // get value and pack
        pack_vnni2<scalar_t>(
            /*    dst */ Btmp,
            /*    src */ v_extend + (seq_extend_start_loc + n) * ve_strideN + head_kv_id * ve_strideH,
            /*     K  */ n_size,
            /*     N  */ head_size_v,
            /* ld_src */ ve_strideN,
            /* ld_dst */ head_size_v);

        // calculate V' <- s_delta @ V + V'
        at::native::cpublas::brgemm(
            /* M     */ m_size,
            /* N     */ head_size_v,
            /* K     */ padded_n_size,  // n_size
            /* lda   */ BLOCK_N,
            /* ldb   */ head_size_v,
            /* ldc   */ head_size_v,
            /* add_C */ true,
            /* A     */ s_delta,
            /* B     */ Btmp,
            /* C     */ v_prime);
      }  // loop with seq_len_extend
```
**EN:** This section uses `apply`, `brgemm` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`apply`、`brgemm`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 261-283: Runtime integration and dispatch
```cpp
      scalar_t* __restrict__ out_ptr = o_extend + (seq_extend_start_loc + m) * o_strideM + head_id * o_strideH;
      for (int row = 0; row < m_size; ++row) {
        float s = 1 / s_prime[row];
        copy_stub<scalar_t>(out_ptr + row * o_strideM, v_prime + row * head_size_v, s, head_size_v);
      }

      // move to the next index
      data_index_step(bs, batches, head_id, num_heads, mb, MB);
    }
    at::native::cpublas::brgemm_release();
  });
}

}  // anonymous namespace

template <int BLOCK_M, int BLOCK_N>
inline int resize_buffer(at::Tensor& buffer, int num_threads, int head_size, int head_size_v) {
  static_assert(BLOCK_M <= BLOCK_N, "Make sure BLOCK_M <= BLOCK_N to prevent buffer overflows during causal masking");
  const int size_per_thread =
      /* s_i     */ BLOCK_M * BLOCK_N * sizeof(float) +
      /* v_prime */ BLOCK_M * head_size_v * sizeof(float) +
      /* Btmp    */ BLOCK_N * std::max(head_size, head_size_v) * sizeof(uint16_t);
```
**EN:** This section uses `resize_buffer`, `data_index_step`, `brgemm_release` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`resize_buffer`、`data_index_step`、`brgemm_release`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 284-307: Local implementation details
```cpp
  buffer.resize_({num_threads, size_per_thread});
  return size_per_thread;
}

#define LAUNCH_EXTEND_ATTENTION_KERNEL(BLOCK_M, BLOCK_N)                                   \
  do {                                                                                     \
    int sz = resize_buffer<BLOCK_M, BLOCK_N>(buffer, num_threads, head_size, head_size_v); \
                                                                                           \
    extend_attention_kernel_impl<scalar_t, index_t, BLOCK_M, BLOCK_N>(                     \
        o_extend.data_ptr<scalar_t>(),                                                     \
        q_extend.data_ptr<scalar_t>(),                                                     \
        k_extend.data_ptr<scalar_t>(),                                                     \
        v_extend.data_ptr<scalar_t>(),                                                     \
        k_buffer.data_ptr<scalar_t>(),                                                     \
        v_buffer.data_ptr<scalar_t>(),                                                     \
        req_to_token.data_ptr<index_t>(),                                                  \
        req_pool_indices.data_ptr<int64_t>(),                                              \
        seq_lens.data_ptr<int64_t>(),                                                      \
        extend_seq_lens.data_ptr<index_t>(),                                               \
        extend_start_loc.data_ptr<index_t>(),                                              \
        buffer.data_ptr(),                                                                 \
        num_seqs,                                                                          \
        num_heads,                                                                         \
        num_heads_kv,                                                                      \
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 308-328: Control flow and branching
```cpp
        head_size,                                                                         \
        head_size_v,                                                                       \
        q_strideM,                                                                         \
        q_strideH,                                                                         \
        ke_strideN,                                                                        \
        ke_strideH,                                                                        \
        ve_strideN,                                                                        \
        ve_strideH,                                                                        \
        k_strideN,                                                                         \
        k_strideH,                                                                         \
        v_strideN,                                                                         \
        v_strideH,                                                                         \
        sm_scale,                                                                          \
        max_num_reqs,                                                                      \
        max_context_len,                                                                   \
        max_total_num_tokens,                                                              \
        max_len_extend,                                                                    \
        sz,                                                                                \
        is_prefix_skipped);                                                                \
  } while (0)
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 329-352: Runtime integration and dispatch
```cpp
// q_extend, k_extend, v_extend, o_extend: contiguous tensors
// k_buffer, v_buffer: (prefix + extend) tensors in mem_manager
//
// q_extend: [num_tokens, num_heads, head_size]
// k_extend: [num_extend_tokens, num_heads, head_size]
// v_extend: [num_extend_tokens, num_heads, head_size]
// o_extend: [num_tokens, num_heads, head_size]
// k_buffer: [max_total_num_tokens, num_heads, head_size]
// v_buffer: [max_total_num_tokens, num_heads, head_size]
// req_to_token: [max_num_reqs, max_context_len] int32 or int64
// req_pool_indices: [num_seqs] int64
// seq_lens: [num_seqs] int64
// extend_seq_lens: [num_seqs]
// extend_start_loc: [num_seqs]
//
void extend_attention_cpu(
    at::Tensor& q_extend,
    at::Tensor& k_extend,
    at::Tensor& v_extend,
    at::Tensor& o_extend,
    at::Tensor& k_buffer,
    at::Tensor& v_buffer,
    at::Tensor& req_to_token,
    at::Tensor& req_pool_indices,
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 353-375: Runtime integration and dispatch
```cpp
    at::Tensor& seq_lens,
    at::Tensor& extend_seq_lens,
    at::Tensor& extend_start_loc,
    int64_t max_len_extend,
    double sm_scale,
    double logit_cap) {
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(q_extend);
  CHECK_INPUT(o_extend);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(k_extend);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(v_extend);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(k_buffer);
  CHECK_LAST_DIM_CONTIGUOUS_INPUT(v_buffer);

  int num_seqs = seq_lens.size(0);
  int max_num_reqs = req_to_token.size(0);
  int max_context_len = req_to_token.size(1);
  int max_total_num_tokens = k_buffer.size(0);

  int num_heads = q_extend.size(1);
  int num_heads_kv = k_extend.size(1);
  int head_size = q_extend.size(2);
  int head_size_v = v_extend.size(2);
```
**EN:** This section uses `CHECK_LAST_DIM_CONTIGUOUS_INPUT`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_LAST_DIM_CONTIGUOUS_INPUT`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 376-399: Local implementation details
```cpp
  // strides for q_extend, k_extend and v_extend
  int q_strideM = q_extend.stride(0);
  int q_strideH = q_extend.stride(1);
  int ke_strideN = k_extend.stride(0);
  int ke_strideH = k_extend.stride(1);
  int ve_strideN = v_extend.stride(0);
  int ve_strideH = v_extend.stride(1);

  // strides for k_buffer and v_buffer
  int k_strideN = k_buffer.stride(0);
  int k_strideH = k_buffer.stride(1);
  int v_strideN = v_buffer.stride(0);
  int v_strideH = v_buffer.stride(1);

  // check sizes
  CHECK_EQ(req_pool_indices.size(0), num_seqs);
  CHECK_EQ(extend_seq_lens.size(0), num_seqs);
  CHECK_EQ(extend_start_loc.size(0), num_seqs);
  CHECK_EQ(v_extend.size(1), num_heads_kv);
  CHECK_EQ(k_buffer.size(1), v_buffer.size(1));

  // MLA will skip prefix part
  const bool is_prefix_skipped = k_buffer.size(1) != num_heads_kv;
```
**EN:** This section fills in the local implementation details around `stride`, `CHECK_EQ`, `is_prefix_skipped`, completing the behavior required by the file.
**CN:** 本段补充了`stride`、`CHECK_EQ`、`is_prefix_skipped`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 400-421: Runtime integration and dispatch
```cpp
  // check index data types
  const auto index_dtype = req_to_token.scalar_type();
  TORCH_CHECK(
      index_dtype == at::kInt || index_dtype == at::kLong,
      "extend: expect req_to_token to be int32 or int64, got ",
      index_dtype);
  TORCH_CHECK(seq_lens.scalar_type() == at::kLong, "extend: expect req_lens to be int64, got ", seq_lens.scalar_type());
  TORCH_CHECK(
      req_pool_indices.scalar_type() == at::kLong,
      "extend: expect req_pool_indices to be int64, got ",
      req_pool_indices.scalar_type());
  TORCH_CHECK(
      extend_seq_lens.scalar_type() == index_dtype && extend_start_loc.scalar_type() == index_dtype,
      "extend: expect extend_seq_lens and extend_start_loc to have same dtype as req_to_token.");

  // D and DV need to be 32x as we transpose by 512-bit
  TORCH_CHECK(head_size % 32 == 0, "invalid head_size ", head_size);
  TORCH_CHECK(head_size_v % 32 == 0, "invalid head_size_v ", head_size_v);

  int num_threads = at::get_num_threads();
  auto buffer = at::empty({}, q_extend.options().dtype(at::kChar));
```
**EN:** This section uses `scalar_type`, `TORCH_CHECK`, `get_num_threads` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`scalar_type`、`TORCH_CHECK`、`get_num_threads`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 422-435: Control flow and branching
```cpp
  AT_DISPATCH_REDUCED_FLOATING_TYPES(q_extend.scalar_type(), "extend_attention_kernel", [&] {
    AT_DISPATCH_INDEX_TYPES(index_dtype, "extend_attention_indices", [&] {
      if (max_len_extend <= 256) {
        LAUNCH_EXTEND_ATTENTION_KERNEL(32, 64);
      } else if (max_len_extend <= 1024) {
        LAUNCH_EXTEND_ATTENTION_KERNEL(128, 256);
      } else if (max_len_extend <= 4096) {
        LAUNCH_EXTEND_ATTENTION_KERNEL(256, 768);
      } else {  // max_len_extend > 4096
        LAUNCH_EXTEND_ATTENTION_KERNEL(512, 768);
      }
    });
  });
}
```
**EN:** This section drives `LAUNCH_EXTEND_ATTENTION_KERNEL` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`LAUNCH_EXTEND_ATTENTION_KERNEL`相关逻辑，决定采用哪条执行路径或数据处理策略。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `flash_attn.h`, `gemm.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / extend.cpp
