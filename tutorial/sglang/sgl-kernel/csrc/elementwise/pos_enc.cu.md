# pos_enc.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/elementwise/pos_enc.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Device helpers and synchronization
```cpp
// Adapted from
// https://github.com/vllm-project/vllm/blob/014ece97c7aa49084a1119dca792af081a18dbc1/csrc/pos_encoding_kernels.cu

#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <torch/all.h>

#include "utils.h"

template <typename scalar_t, bool IS_NEOX>
inline __device__ void apply_token_rotary_embedding(
    scalar_t* __restrict__ arr,
    const scalar_t* __restrict__ cos_ptr,
    const scalar_t* __restrict__ sin_ptr,
    int rot_offset,
    int embed_dim) {
  int x_index, y_index;
  scalar_t cos, sin;
  if (IS_NEOX) {
    // GPT-NeoX style rotary embedding.
    x_index = rot_offset;
    y_index = embed_dim + rot_offset;
```
**EN:** This section implements `apply_token_rotary_embedding`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`apply_token_rotary_embedding`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 23-38: Local implementation details
```cpp
    cos = SGLANG_LDG(cos_ptr + x_index);
    sin = SGLANG_LDG(sin_ptr + x_index);
  } else {
    // GPT-J style rotary embedding.
    x_index = 2 * rot_offset;
    y_index = 2 * rot_offset + 1;
    cos = SGLANG_LDG(cos_ptr + x_index / 2);
    sin = SGLANG_LDG(sin_ptr + x_index / 2);
  }

  const scalar_t x = arr[x_index];
  const scalar_t y = arr[y_index];
  arr[x_index] = x * cos - y * sin;
  arr[y_index] = y * cos + x * sin;
}
```
**EN:** This section fills in the local implementation details around `SGLANG_LDG`, `x`, `y`, completing the behavior required by the file.
**CN:** 本段补充了`SGLANG_LDG`、`x`、`y`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 39-60: Device helpers and synchronization
```cpp
template <typename scalar_t, bool IS_NEOX>
inline __device__ void apply_rotary_embedding(
    scalar_t* __restrict__ query,  // [batch_size, seq_len, num_heads,
                                   // head_size] or [num_tokens, num_heads,
                                   // head_size]
    scalar_t* __restrict__ key,    // nullptr or
                                   // [batch_size, seq_len, num_kv_heads,
                                   // head_size] or [num_tokens, num_kv_heads,
                                   // head_size]
    const scalar_t* cache_ptr,
    const int head_size,
    const int num_heads,
    const int num_kv_heads,
    const int rot_dim,
    const int token_idx,
    const int64_t query_stride,
    const int64_t key_stride,
    const int64_t head_stride) {
  const int embed_dim = rot_dim / 2;
  const scalar_t* cos_ptr = cache_ptr;
  const scalar_t* sin_ptr = cache_ptr + embed_dim;
```
**EN:** This section implements `apply_rotary_embedding`, `embed_dim`, `cos_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`apply_rotary_embedding`、`embed_dim`、`cos_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 61-79: Device helpers and synchronization
```cpp
  const int nq = num_heads * embed_dim;
  for (int i = threadIdx.x; i < nq; i += blockDim.x) {
    const int head_idx = i / embed_dim;
    const int64_t token_head = token_idx * query_stride + head_idx * head_stride;
    const int rot_offset = i % embed_dim;
    apply_token_rotary_embedding<scalar_t, IS_NEOX>(query + token_head, cos_ptr, sin_ptr, rot_offset, embed_dim);
  }

  if (key != nullptr) {
    const int nk = num_kv_heads * embed_dim;
    for (int i = threadIdx.x; i < nk; i += blockDim.x) {
      const int head_idx = i / embed_dim;
      const int64_t token_head = token_idx * key_stride + head_idx * head_stride;
      const int rot_offset = i % embed_dim;
      apply_token_rotary_embedding<scalar_t, IS_NEOX>(key + token_head, cos_ptr, sin_ptr, rot_offset, embed_dim);
    }
  }
}
```
**EN:** This section implements `nq`, `head_idx`, `token_head`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`nq`、`head_idx`、`token_head`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 80-104: Kernel implementation
```cpp
template <typename scalar_t, bool IS_NEOX>
__global__ void rotary_embedding_kernel(
    const int64_t* __restrict__ positions,       // [batch_size, seq_len] or
                                                 // [num_tokens]
    scalar_t* __restrict__ query,                // [batch_size, seq_len, num_heads,
                                                 // head_size] or [num_tokens, num_heads,
                                                 // head_size]
    scalar_t* __restrict__ key,                  // nullptr or
                                                 // [batch_size, seq_len, num_kv_heads,
                                                 // head_size] or [num_tokens, num_kv_heads,
                                                 // head_size]
    const scalar_t* __restrict__ cos_sin_cache,  // [max_position, 2, rot_dim //
                                                 // 2]
    const int rot_dim,
    const int64_t query_stride,
    const int64_t key_stride,
    const int64_t head_stride,
    const int num_heads,
    const int num_kv_heads,
    const int head_size) {
  // Each thread block is responsible for one token.
  const int token_idx = blockIdx.x;
  int64_t pos = positions[token_idx];
  const scalar_t* cache_ptr = cos_sin_cache + pos * rot_dim;
```
**EN:** This section implements `rotary_embedding_kernel`, `token_idx`, `cache_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`rotary_embedding_kernel`、`token_idx`、`cache_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 105-126: Runtime integration and dispatch
```cpp
  apply_rotary_embedding<scalar_t, IS_NEOX>(
      query,
      key,
      cache_ptr,
      head_size,
      num_heads,
      num_kv_heads,
      rot_dim,
      token_idx,
      query_stride,
      key_stride,
      head_stride);
}

void rotary_embedding(
    torch::Tensor& positions,  // [batch_size, seq_len] or [num_tokens]
    torch::Tensor& query,      // [batch_size, seq_len, num_heads * head_size] or
                               // [num_tokens, num_heads * head_size] or
                               // [batch_size, seq_len, num_heads, head_size] or
                               // [num_tokens, num_heads, head_size]
    std::optional<torch::Tensor> key,
    // null or
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 127-151: Runtime integration and dispatch
```cpp
    // [batch_size, seq_len, num_kv_heads * head_size] or
    // [num_tokens, num_kv_heads * head_size] or
    // [batch_size, seq_len, num_heads, head_size] or
    // [num_tokens, num_heads, head_size]
    int64_t head_size,
    torch::Tensor& cos_sin_cache,  // [max_position, rot_dim]
    bool is_neox) {
  // num_tokens = batch_size * seq_len
  int64_t num_tokens = positions.numel();
  int positions_ndim = positions.dim();

  // Make sure num_tokens dim is consistent across positions, query, and key
  TORCH_CHECK(
      positions_ndim == 1 || positions_ndim == 2, "positions must have shape [num_tokens] or [batch_size, seq_len]");
  if (positions_ndim == 1) {
    TORCH_CHECK(
        query.size(0) == positions.size(0) && (!key.has_value() || key->size(0) == positions.size(0)),
        "query, key and positions must have the same number of tokens");
  }
  if (positions_ndim == 2) {
    TORCH_CHECK(
        query.size(0) == positions.size(0) && (!key.has_value() || key->size(0) == positions.size(0)) &&
            query.size(1) == positions.size(1) && (!key.has_value() || key->size(1) == positions.size(1)),
        "query, key and positions must have the same batch_size and seq_len");
  }
```
**EN:** This section uses `numel`, `dim`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`numel`、`dim`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 152-174: Runtime integration and dispatch
```cpp

  // Make sure head_size is valid for query and key
  // hidden_size = num_heads * head_size
  int query_hidden_size = query.numel() / num_tokens;
  int key_hidden_size = key.has_value() ? key->numel() / num_tokens : 0;
  TORCH_CHECK(query_hidden_size % head_size == 0);
  TORCH_CHECK(key_hidden_size % head_size == 0);

  // Make sure query and key have consistent number of heads
  int num_heads = query_hidden_size / head_size;
  int num_kv_heads = key.has_value() ? key_hidden_size / head_size : num_heads;
  TORCH_CHECK(num_heads % num_kv_heads == 0);

  int rot_dim = cos_sin_cache.size(1);
  int seq_dim_idx = positions_ndim - 1;
  int64_t query_stride = query.stride(seq_dim_idx);
  int64_t key_stride = key.has_value() ? key->stride(seq_dim_idx) : 0;
  // Determine head stride: for [*, heads, head_size] use stride of last dim;
  // for flat [*, heads*head_size], heads blocks are contiguous of size
  // head_size
  int query_ndim = query.dim();
  int64_t head_stride = (query_ndim == positions_ndim + 2) ? query.stride(-2) : head_size;
```
**EN:** This section uses `TORCH_CHECK`, `stride`, `dim` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`stride`、`dim`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 175-196: Kernel implementation
```cpp
  dim3 grid(num_tokens);
  dim3 block(std::min<int64_t>(num_heads * rot_dim / 2, 512));
  const at::cuda::OptionalCUDAGuard device_guard(device_of(query));
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  DISPATCH_FLOAT_TYPES(query.scalar_type(), "rotary_embedding", [&] {
    if (is_neox) {
      rotary_embedding_kernel<scalar_t, true><<<grid, block, 0, stream>>>(
          positions.data_ptr<int64_t>(),
          query.data_ptr<scalar_t>(),
          key.has_value() ? key->data_ptr<scalar_t>() : nullptr,
          cos_sin_cache.data_ptr<scalar_t>(),
          rot_dim,
          query_stride,
          key_stride,
          head_stride,
          num_heads,
          num_kv_heads,
          head_size);
    } else {
      rotary_embedding_kernel<scalar_t, false><<<grid, block, 0, stream>>>(
          positions.data_ptr<int64_t>(),
          query.data_ptr<scalar_t>(),
```
**EN:** This section implements `grid`, `block`, `device_guard`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`grid`、`block`、`device_guard`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 197-208: Local implementation details
```cpp
          key.has_value() ? key->data_ptr<scalar_t>() : nullptr,
          cos_sin_cache.data_ptr<scalar_t>(),
          rot_dim,
          query_stride,
          key_stride,
          head_stride,
          num_heads,
          num_kv_heads,
          head_size);
    }
  });
}
```
**EN:** This section fills in the local implementation details around `has_value`, completing the behavior required by the file.
**CN:** 本段补充了`has_value`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `torch/all.h`
- **Path context / 路径上下文**: elementwise / pos_enc.cu
