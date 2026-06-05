# pos_encoding_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/pos_encoding_kernels.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Applies rotary positional embeddings (RoPE) to query and optional key tensors on CUDA, supporting both GPT-NeoX and GPT-J style layouts. / [CN] 在 CUDA 上对 query 和可选 key 张量应用旋转位置编码（RoPE），同时支持 GPT-NeoX 与 GPT-J 两种布局。

## Line-by-Line Analysis / 逐行分析

### [Token-level rotation helper / 单 token 旋转辅助函数]
```cpp
template <typename scalar_t, typename cache_t, bool IS_NEOX>
inline __device__ void apply_token_rotary_embedding(
    scalar_t* __restrict__ arr, const cache_t* __restrict__ cos_ptr,
    const cache_t* __restrict__ sin_ptr, int rot_offset, int embed_dim,
    const bool inverse) {
  int x_index, y_index;
  float cos_f, sin_f;
  if (IS_NEOX) {
    x_index = rot_offset;
    y_index = embed_dim + rot_offset;
  } else {
    x_index = 2 * rot_offset;
    y_index = 2 * rot_offset + 1;
  }
  ...
  arr[x_index] = static_cast<scalar_t>(x_f * cos_f - y_f * sin_f);
  arr[y_index] = static_cast<scalar_t>(y_f * cos_f + x_f * sin_f);
}
```
**EN:** This helper performs the actual 2D rotation for one rotary pair. The important detail is layout handling: NeoX stores the two halves separately (`x`, `y`), while GPT-J interleaves them. The `inverse` flag flips the sign of sine, so the same code can be reused for forward or reverse rotation.  
**CN:** 这个辅助函数完成单个旋转维对的 2D 旋转。关键在于布局处理：NeoX 把两个半维分开存放，而 GPT-J 采用交错布局。`inverse` 标志会翻转正弦项符号，因此同一套代码既可用于正向旋转，也可用于逆向旋转。

### [Per-token loop over heads / 针对每个 token 的多头循环]
```cpp
template <typename scalar_t, typename cache_t, bool IS_NEOX>
inline __device__ void apply_rotary_embedding(
    scalar_t* __restrict__ query, scalar_t* __restrict__ key,
    const cache_t* cache_ptr, const int head_size, const int num_heads,
    const int num_kv_heads, const int rot_dim, const int token_idx,
    const int64_t query_stride, const int64_t key_stride,
    const int64_t head_stride, const int64_t rope_dim_offset,
    const bool inverse) {
  const int embed_dim = rot_dim / 2;
  ...
  for (int i = threadIdx.x; i < nq; i += blockDim.x) {
    const int head_idx = i / embed_dim;
    const int64_t token_head =
        token_idx * query_stride + head_idx * head_stride + rope_dim_offset;
    ...
  }
}
```
**EN:** `apply_rotary_embedding` maps CUDA threads across all rotary dimensions of all heads for one token. It computes the proper memory offset using the token stride, head stride, and optional `rope_dim_offset`, so the same kernel works for flattened `[tokens, heads * head_size]` tensors and explicit `[tokens, heads, head_size]` tensors. If a key tensor is present, the function repeats the same rotation logic for KV heads.  
**CN:** `apply_rotary_embedding` 把 CUDA 线程分布到单个 token 的所有头、所有旋转维度上。它结合 token stride、head stride 和可选的 `rope_dim_offset` 计算正确内存偏移，因此同一个内核既支持扁平的 `[tokens, heads * head_size]` 布局，也支持显式的 `[tokens, heads, head_size]` 布局。如果存在 key 张量，则会对 KV 头重复相同的旋转逻辑。

### [Kernel launch and PyTorch wrapper / 内核启动与 PyTorch 封装]
```cpp
template <typename scalar_t, typename cache_t, bool IS_NEOX>
__global__ void rotary_embedding_kernel(...)
{
  const int token_idx = blockIdx.x;
  int64_t pos = positions[token_idx];
  const cache_t* cache_ptr = cos_sin_cache + pos * rot_dim;
  apply_rotary_embedding<scalar_t, cache_t, IS_NEOX>(...);
}

void rotary_embedding(..., torch::Tensor& cos_sin_cache,
                      bool is_neox, int64_t rope_dim_offset, bool inverse) {
  ...
  VLLM_DISPATCH_FLOATING_TYPES(query.scalar_type(), "rotary_embedding", [&] {
    VLLM_DISPATCH_FLOATING_TYPES(
        cos_sin_cache.scalar_type(), "rotary_embedding_cache", [&] {
          ...
        });
  });
}
```
**EN:** The global kernel uses one block per token, loads that token’s position, and points into the cached cosine/sine table at `pos * rot_dim`. The host wrapper validates tensor shapes, derives head counts and strides from the runtime layout, guards the active CUDA device, and dispatches both query dtype and cache dtype through `VLLM_DISPATCH_FLOATING_TYPES`.  
**CN:** 全局内核采用“每个 token 一个 block”的映射方式：先读取该 token 的位置，再把 `cos_sin_cache` 指针移动到 `pos * rot_dim` 的位置。主机侧封装函数会校验张量形状，从运行时布局推导头数与步幅，设置 CUDA 设备 guard，并通过 `VLLM_DISPATCH_FLOATING_TYPES` 同时分发 query 和 cache 的数据类型。

## Key Concepts / 关键概念
- **EN:** The code separates layout-specific indexing from the actual rotation formula, which keeps NeoX and GPT-J support in one templated implementation.  
  **CN:** 代码把“布局相关索引”与“旋转公式”分离，从而用同一套模板实现同时支持 NeoX 和 GPT-J。
- **EN:** `rope_dim_offset` allows rotary embedding to affect only a suffix or subrange of each head.  
  **CN:** `rope_dim_offset` 允许旋转位置编码只作用于每个 head 的某个后缀或子区间。
- **EN:** The kernel mutates `query` in-place and optionally `key` in-place; there is no separate output tensor.  
  **CN:** 该内核会原地修改 `query`，并在存在 `key` 时原地修改 `key`；没有独立输出张量。

## Dependencies / 依赖关系
- **EN:** Depends on `cuda_compat.h` for `VLLM_LDG` and `dispatch_utils.h` for dtype dispatch macros.  
  **CN:** 依赖 `cuda_compat.h` 中的 `VLLM_LDG` 以及 `dispatch_utils.h` 中的数据类型分发宏。
- **EN:** Uses ATen CUDA stream/device helpers so the op integrates with PyTorch execution context.  
  **CN:** 使用 ATen 的 CUDA stream / device 辅助接口，以便该算子融入 PyTorch 的执行上下文。
