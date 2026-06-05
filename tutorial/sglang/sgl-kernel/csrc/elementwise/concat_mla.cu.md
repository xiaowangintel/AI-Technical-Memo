# concat_mla.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/elementwise/concat_mla.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Kernel implementation
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/CUDADataType.h>
#include <cuda_runtime.h>

#include "pytorch_extension_utils.h"
#include "utils.cuh"

constexpr int NUM_LOCAL_HEADS = 128;
constexpr int QK_NOPE_HEAD_DIM = 128;
constexpr int QK_ROPE_HEAD_DIM = 64;
constexpr int K_HEAD_DIM = QK_NOPE_HEAD_DIM + QK_ROPE_HEAD_DIM;

constexpr int HEAD_CHUNK_SIZE = 16;
constexpr int NUM_HEAD_CHUNKS = NUM_LOCAL_HEADS / HEAD_CHUNK_SIZE;

__global__ void concat_mla_k_kernel(
    nv_bfloat16* __restrict__ k,
    const nv_bfloat16* __restrict__ k_nope,
    const nv_bfloat16* __restrict__ k_rope,
    const int num_tokens,
    const int64_t k_stride_0,
    const int k_stride_1,
```
**EN:** This section implements `NUM_LOCAL_HEADS`, `QK_NOPE_HEAD_DIM`, `QK_ROPE_HEAD_DIM`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`NUM_LOCAL_HEADS`、`QK_NOPE_HEAD_DIM`、`QK_ROPE_HEAD_DIM`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 23-43: Device helpers and synchronization
```cpp
    const int64_t k_nope_stride_0,
    const int k_nope_stride_1,
    const int64_t k_rope_stride_0) {
  const int flat_warp_id = (blockIdx.x * blockDim.x + threadIdx.x) / 32;
  const int token_id = flat_warp_id / NUM_HEAD_CHUNKS;
  const int head_chunk_id = flat_warp_id % NUM_HEAD_CHUNKS;
  const int lane_id = get_lane_id();
  if (token_id >= num_tokens) return;

  using NopeVec = int2;  // 8B/thread，32 thread = 256B/row
  using RopeVec = int;   // 4B/thread，32 thread = 128B/row
  static_assert(sizeof(NopeVec) * 32 == QK_NOPE_HEAD_DIM * sizeof(nv_bfloat16), "nope vec mismatch");
  static_assert(sizeof(RopeVec) * 32 == QK_ROPE_HEAD_DIM * sizeof(nv_bfloat16), "rope vec mismatch");

  const int head_row0 = head_chunk_id * HEAD_CHUNK_SIZE;

  const int2* __restrict__ nope_src =
      reinterpret_cast<const int2*>(k_nope + token_id * k_nope_stride_0 + head_row0 * k_nope_stride_1) + lane_id;

  int2* __restrict__ nope_dst = reinterpret_cast<int2*>(k + token_id * k_stride_0 + head_row0 * k_stride_1) + lane_id;
```
**EN:** This section implements `get_lane_id`, `static_assert`, `flat_warp_id`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_lane_id`、`static_assert`、`flat_warp_id`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 44-65: Device helpers and synchronization
```cpp
  int* __restrict__ rope_dst =
      reinterpret_cast<int*>(k + token_id * k_stride_0 + head_row0 * k_stride_1 + QK_NOPE_HEAD_DIM) + lane_id;

  const int nope_src_stride_v = (k_nope_stride_1 >> 2);  // int2 covers 4 bf16
  const int nope_dst_stride_v = (k_stride_1 >> 2);
  const int rope_dst_stride_v = (k_stride_1 >> 1);  // int covers 2 bf16

  const int* rope_base = reinterpret_cast<const int*>(k_rope + token_id * k_rope_stride_0);
  const RopeVec rope_val = ld_na_global_v1(rope_base + lane_id);

  prefetch_L2(nope_src);
  NopeVec cur = ld_na_global_v2(nope_src);

#pragma unroll
  for (int i = 0; i < HEAD_CHUNK_SIZE; ++i) {
    NopeVec next;
    if (i + 1 < HEAD_CHUNK_SIZE) {
      const int2* next_src = nope_src + nope_src_stride_v;
      prefetch_L2(next_src);
      next = ld_na_global_v2(next_src);
    }
```
**EN:** This section implements `ld_na_global_v1`, `prefetch_L2`, `ld_na_global_v2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ld_na_global_v1`、`prefetch_L2`、`ld_na_global_v2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 66-86: Device helpers and synchronization
```cpp
    st_na_global_v2(nope_dst, cur);
    st_na_global_v1(rope_dst, rope_val);

    nope_src += nope_src_stride_v;
    nope_dst += nope_dst_stride_v;
    rope_dst += rope_dst_stride_v;

    cur = next;
  }
}

inline void check_tensor(const at::Tensor& t, int64_t shape0, int64_t shape1, int64_t shape2, c10::ScalarType dtype) {
  TORCH_CHECK_EQ(t.dim(), 3);
  TORCH_CHECK_EQ(t.size(0), shape0);
  TORCH_CHECK_EQ(t.size(1), shape1);
  TORCH_CHECK_EQ(t.size(2), shape2);
  TORCH_CHECK_EQ(t.dtype(), dtype);
  TORCH_CHECK(t.device().is_cuda());
  TORCH_CHECK_EQ(((int64_t)t.data_ptr()) % 16, 0);  // alignment
}
```
**EN:** This section implements `check_tensor`, `st_na_global_v2`, `st_na_global_v1`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`check_tensor`、`st_na_global_v2`、`st_na_global_v1`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 87-102: Device helpers and synchronization
```cpp
void concat_mla_k(at::Tensor k, at::Tensor k_nope, at::Tensor k_rope) {
  const int num_tokens = k.size(0);

  check_tensor(k, num_tokens, NUM_LOCAL_HEADS, K_HEAD_DIM, at::kBFloat16);
  check_tensor(k_nope, num_tokens, NUM_LOCAL_HEADS, QK_NOPE_HEAD_DIM, at::kBFloat16);
  check_tensor(k_rope, num_tokens, 1, QK_ROPE_HEAD_DIM, at::kBFloat16);
  TORCH_CHECK_EQ(k.stride(2), 1);
  TORCH_CHECK_EQ(k_nope.stride(2), 1);
  TORCH_CHECK_EQ(k_rope.stride(2), 1);

  const auto stream = at::cuda::getCurrentCUDAStream().stream();

  constexpr int num_warps_per_block = 32;
  const int grid_size = ceil_div(num_tokens * NUM_HEAD_CHUNKS, num_warps_per_block);
  const int block_size = num_warps_per_block * 32;
```
**EN:** This section implements `concat_mla_k`, `check_tensor`, `TORCH_CHECK_EQ`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`concat_mla_k`、`check_tensor`、`TORCH_CHECK_EQ`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 103-122: Kernel implementation
```cpp
  concat_mla_k_kernel<<<grid_size, block_size, 0, stream>>>(
      reinterpret_cast<nv_bfloat16*>(k.data_ptr()),
      reinterpret_cast<nv_bfloat16*>(k_nope.data_ptr()),
      reinterpret_cast<nv_bfloat16*>(k_rope.data_ptr()),
      num_tokens,
      k.stride(0),
      k.stride(1),
      k_nope.stride(0),
      k_nope.stride(1),
      k_rope.stride(0));
  cudaError_t err = cudaGetLastError();
  TORCH_CHECK(err == cudaSuccess, "CUDA kernel launch failed: ", cudaGetErrorString(err));
}

// ============================== concat_mla_absorb_q ==============================

// TODO give a name prefix, also maybe refactor code above
constexpr int A_LAST_DIM = 512;
constexpr int B_LAST_DIM = 64;
```
**EN:** This section implements `data_ptr`, `cudaGetLastError`, `TORCH_CHECK`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`、`cudaGetLastError`、`TORCH_CHECK`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 123-144: Kernel implementation
```cpp
__global__ void concat_mla_absorb_q_kernel(
    nv_bfloat16* a,
    nv_bfloat16* b,
    nv_bfloat16* out,
    const int num_items,
    const int dim_1,
    const int64_t a_stride_0,
    const int a_stride_1,
    const int64_t b_stride_0,
    const int b_stride_1,
    const int64_t out_stride_0,
    const int out_stride_1) {
  const int flat_warp_id = (blockIdx.x * blockDim.x + threadIdx.x) / 32;
  const int lane_id = get_lane_id();

  const int idx_0 = flat_warp_id / dim_1;
  const int idx_1 = flat_warp_id % dim_1;

  if (flat_warp_id >= num_items) {
    return;
  }
```
**EN:** This section implements `concat_mla_absorb_q_kernel`, `get_lane_id`, `flat_warp_id`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`concat_mla_absorb_q_kernel`、`get_lane_id`、`flat_warp_id`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 145-165: Templates, aliases, and constants
```cpp
  using ABufType = int4;
  constexpr int A_NUM_UNROLL = 2;
  static_assert(sizeof(ABufType) * A_NUM_UNROLL == A_LAST_DIM * sizeof(a[0]) / 32);
  ABufType a_buf[A_NUM_UNROLL];

  using BBufType = int;
  constexpr int B_NUM_UNROLL = 1;
  static_assert(sizeof(BBufType) * B_NUM_UNROLL == B_LAST_DIM * sizeof(b[0]) / 32);
  BBufType b_buf;

  {
    const BBufType* base_addr = reinterpret_cast<BBufType*>(b + idx_0 * b_stride_0 + idx_1 * b_stride_1);
    b_buf = *(base_addr + lane_id);
  }

#pragma unroll
  for (int i = 0; i < A_NUM_UNROLL; ++i) {
    const ABufType* base_addr = reinterpret_cast<ABufType*>(a + idx_0 * a_stride_0 + idx_1 * a_stride_1);
    a_buf[i] = *(base_addr + i * 32 + lane_id);
  }
```
**EN:** This section defines `static_assert`, `ABufType`, `A_NUM_UNROLL`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`static_assert`、`ABufType`、`A_NUM_UNROLL`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 166-186: Runtime integration and dispatch
```cpp
  {
    BBufType* base_addr = reinterpret_cast<BBufType*>(out + idx_0 * out_stride_0 + idx_1 * out_stride_1 + A_LAST_DIM);
    *(base_addr + lane_id) = b_buf;
  }

#pragma unroll
  for (int i = 0; i < A_NUM_UNROLL; ++i) {
    ABufType* base_addr = reinterpret_cast<ABufType*>(out + idx_0 * out_stride_0 + idx_1 * out_stride_1);
    *(base_addr + i * 32 + lane_id) = a_buf[i];
  }
}

inline void check_tensor_concat_mla_absorb_q(const at::Tensor& t, int64_t shape2) {
  TORCH_CHECK_EQ(t.dim(), 3);
  TORCH_CHECK_EQ(t.size(2), shape2);
  TORCH_CHECK_EQ(t.stride(2), 1);
  TORCH_CHECK_EQ(t.dtype(), at::kBFloat16);
  TORCH_CHECK(t.device().is_cuda());
  TORCH_CHECK_EQ(((int64_t)t.data_ptr()) % 16, 0);  // alignment
}
```
**EN:** This section uses `check_tensor_concat_mla_absorb_q`, `TORCH_CHECK_EQ`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`check_tensor_concat_mla_absorb_q`、`TORCH_CHECK_EQ`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 187-202: Device helpers and synchronization
```cpp
// TODO further optimize it later
void concat_mla_absorb_q(at::Tensor a, at::Tensor b, at::Tensor out) {
  check_tensor_concat_mla_absorb_q(a, A_LAST_DIM);
  check_tensor_concat_mla_absorb_q(b, B_LAST_DIM);
  check_tensor_concat_mla_absorb_q(out, A_LAST_DIM + B_LAST_DIM);

  const auto stream = at::cuda::getCurrentCUDAStream().stream();

  TORCH_CHECK_EQ(a.size(0) * a.size(1), b.size(0) * b.size(1));
  TORCH_CHECK_EQ(a.size(1), b.size(1));
  const int num_items = a.size(0) * a.size(1);

  constexpr int num_warps_per_block = 32;
  const int grid_size = ceil_div(num_items, num_warps_per_block);
  const int block_size = num_warps_per_block * 32;
```
**EN:** This section implements `concat_mla_absorb_q`, `check_tensor_concat_mla_absorb_q`, `getCurrentCUDAStream`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`concat_mla_absorb_q`、`check_tensor_concat_mla_absorb_q`、`getCurrentCUDAStream`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 203-217: Kernel implementation
```cpp
  concat_mla_absorb_q_kernel<<<grid_size, block_size, 0, stream>>>(
      reinterpret_cast<nv_bfloat16*>(a.data_ptr()),
      reinterpret_cast<nv_bfloat16*>(b.data_ptr()),
      reinterpret_cast<nv_bfloat16*>(out.data_ptr()),
      num_items,
      a.size(1),
      a.stride(0),
      a.stride(1),
      b.stride(0),
      b.stride(1),
      out.stride(0),
      out.stride(1));
  cudaError_t err = cudaGetLastError();
  TORCH_CHECK(err == cudaSuccess, "CUDA kernel launch failed: ", cudaGetErrorString(err));
}
```
**EN:** This section implements `data_ptr`, `cudaGetLastError`, `TORCH_CHECK`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`、`cudaGetLastError`、`TORCH_CHECK`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 218-218: Local implementation details
```cpp
// test-1
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Validation / 校验测试**: Contains checks or scaffolding for correctness verification. / 包含正确性验证所需的检查或脚手架。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `pytorch_extension_utils.h`, `utils.cuh`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `ATen/cuda/CUDADataType.h`, `cuda_runtime.h`
- **Path context / 路径上下文**: elementwise / concat_mla.cu
