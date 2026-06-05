# transfer.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/kvcacheio/transfer.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Headers and compile-time setup
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAException.h>
#include <c10/util/irange.h>
#include <cuda_runtime.h>

#include <cstdint>
#include <limits>
#include <vector>

#if !defined(USE_ROCM) && !defined(USE_MUSA)
#include <dlfcn.h>
#define WARP_SIZE 32
#include "pytorch_extension_utils.h"
#else
#include "pytorch_extension_utils_rocm.h"
#include "utils.h"  // WARP_SIZE
#endif

__device__ __forceinline__ void
transfer_item_warp(int32_t lane_id, const void* src_addr, void* dst_addr, int64_t item_size_bytes) {
  const uint64_t* __restrict__ src = static_cast<const uint64_t*>(src_addr);
  uint64_t* __restrict__ dst = static_cast<uint64_t*>(dst_addr);
  const int total_chunks = item_size_bytes / sizeof(uint64_t);

#pragma unroll
  for (int j = lane_id; j < total_chunks; j += WARP_SIZE) {
#if !defined(USE_ROCM) && !defined(USE_MUSA)
    uint64_t tmp;
    asm volatile("ld.global.nc.b64 %0,[%1];" : "=l"(tmp) : "l"(src + j) : "memory");
    asm volatile("st.global.cg.b64 [%0],%1;" ::"l"(dst + j), "l"(tmp) : "memory");
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 32-61: Device helpers and synchronization
```cpp
#else
    uint64_t tmp = __builtin_nontemporal_load(src + j);
    __builtin_nontemporal_store(tmp, dst + j);
#endif
  }
}

template <typename T>
__device__ __forceinline__ T* get_global_offset_lf(
    T* base,
    const uintptr_t* __restrict__ /*unused*/,
    int64_t layer_id,
    int64_t layer_dim,
    int64_t page_id,
    int64_t item_size_bytes) {
  // layer first
  return base + layer_id * layer_dim + page_id * item_size_bytes;
}

template <typename T>
__device__ __forceinline__ T* get_global_offset_pf(
    T* base,
    const uintptr_t* __restrict__ /*unused*/,
    int64_t layer_id,
    int64_t page_dim,
    int64_t page_id,
    int64_t item_size_bytes) {
  // page first
  return base + page_id * page_dim + layer_id * item_size_bytes;
}
```
**EN:** This section implements `get_global_offset_lf`, `get_global_offset_pf`, `__builtin_nontemporal_load`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_global_offset_lf`、`get_global_offset_pf`、`__builtin_nontemporal_load`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 62-89: Device helpers and synchronization
```cpp

// get offset from layer base table when layers are not contiguous
template <typename T>
__device__ __forceinline__ T* get_global_offset_lf_tbl(
    T* /*unused*/,
    const uintptr_t* __restrict__ layer_base_tbl,
    int64_t layer_id,
    int64_t /*unused*/,
    int64_t page_id,
    int64_t item_size_bytes) {
  return reinterpret_cast<T*>(layer_base_tbl[layer_id]) + page_id * item_size_bytes;
}

template <typename T>
__device__ __forceinline__ T* get_global_offset_per_head_lf(
    T* base,
    const uintptr_t* __restrict__ /*unused*/,
    int64_t layer_id,
    int64_t layer_dim,
    int64_t page_id,
    int64_t item_size_bytes,
    int64_t head_id,
    int64_t head_num,
    int64_t /*unused*/) {
  // layer first offset func per head
  return base + layer_id * layer_dim + page_id * item_size_bytes + item_size_bytes / head_num * head_id;
}
```
**EN:** This section implements `get_global_offset_lf_tbl`, `get_global_offset_per_head_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_global_offset_lf_tbl`、`get_global_offset_per_head_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 90-121: Device helpers and synchronization
```cpp
template <typename T>
__device__ __forceinline__ T* get_global_offset_per_head_lf_tbl(
    T* /*unused*/,
    const uintptr_t* __restrict__ layer_base_tbl,
    int64_t layer_id,
    int64_t /*unused*/,
    int64_t page_id,
    int64_t item_size_bytes,
    int64_t head_id,
    int64_t head_num,
    int64_t /*unused*/) {
  return reinterpret_cast<T*>(layer_base_tbl[layer_id]) + page_id * item_size_bytes +
         item_size_bytes / head_num * head_id;
}

template <typename T>
__device__ __forceinline__ T* get_global_offset_ph(
    T* base,
    const uintptr_t* __restrict__ /*unused*/,
    int64_t layer_id,
    int64_t page_dim,
    int64_t page_id,
    int64_t item_size_bytes,
    int64_t head_id,
    int64_t head_num,
    int64_t page_size) {
  // page head layout: [page_num, head_num, page_size, layer_num, head_dim]
  return base + page_id / page_size * page_size * page_dim +  // page_num dimension offset
         page_dim / head_num * head_id * page_size +          // head_num dimension offset
         page_id % page_size * page_dim / head_num +          // page_size dimension offset
         layer_id * item_size_bytes / head_num;               // layer_num dimension offset
}
```
**EN:** This section implements `get_global_offset_per_head_lf_tbl`, `get_global_offset_ph`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_global_offset_per_head_lf_tbl`、`get_global_offset_ph`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 122-148: Kernel implementation
```cpp

template <auto SrcOffsetFn, auto DstOffsetFn>
__global__ void transfer_page_head_kernel_impl(
    const void* __restrict__ src_k,
    void* __restrict__ dst_k,
    const void* __restrict__ src_v,
    void* __restrict__ dst_v,
    const int64_t* __restrict__ src_indices,
    const int64_t* __restrict__ dst_indices,
    int64_t start_layer_id,
    int64_t num_layers_to_process,
    int64_t num_items,
    int64_t items_per_warp,
    int64_t item_size_bytes,
    int64_t src_layout_dim,
    int64_t dst_layout_dim,
    const uintptr_t* __restrict__ src_k_layer_tbl,
    const uintptr_t* __restrict__ dst_k_layer_tbl,
    const uintptr_t* __restrict__ src_v_layer_tbl,
    const uintptr_t* __restrict__ dst_v_layer_tbl,
    const int64_t page_size,
    const int64_t head_num) {
  int32_t tid = blockIdx.x * blockDim.x + threadIdx.x;
  int32_t lane_id = tid % WARP_SIZE;
  int32_t warp_id = tid / WARP_SIZE;
  const int64_t head_size_bytes = item_size_bytes / head_num;
```
**EN:** This section implements `transfer_page_head_kernel_impl`, `head_size_bytes`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`transfer_page_head_kernel_impl`、`head_size_bytes`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 149-182: Device helpers and synchronization
```cpp
  for (int i = 0; i < items_per_warp; ++i) {
    int64_t item_id = warp_id * items_per_warp + i;
    if (item_id >= num_items) {
      break;
    }
    const int64_t src_page_id = src_indices[item_id];
    const int64_t dst_page_id = dst_indices[item_id];

    // Loop over layers if necessary
    for (int64_t layer_id = start_layer_id; layer_id < start_layer_id + num_layers_to_process; ++layer_id) {
      // For page head layout, the cache of each head in the token is discontinuous, need to loop
      for (int64_t head_id = 0; head_id < head_num; ++head_id) {
        const char* src_k_ptr = SrcOffsetFn(
            static_cast<const char*>(src_k),
            src_k_layer_tbl,
            layer_id,
            src_layout_dim,
            src_page_id,
            item_size_bytes,
            head_id,
            head_num,
            page_size);
        char* dst_k_ptr = DstOffsetFn(
            static_cast<char*>(dst_k),
            dst_k_layer_tbl,
            layer_id,
            dst_layout_dim,
            dst_page_id,
            item_size_bytes,
            head_id,
            head_num,
            page_size);
        transfer_item_warp(lane_id, src_k_ptr, dst_k_ptr, head_size_bytes);
```
**EN:** This section implements `SrcOffsetFn`, `DstOffsetFn`, `transfer_item_warp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`SrcOffsetFn`、`DstOffsetFn`、`transfer_item_warp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 183-208: Device helpers and synchronization
```cpp
        const char* src_v_ptr = SrcOffsetFn(
            static_cast<const char*>(src_v),
            src_v_layer_tbl,
            layer_id,
            src_layout_dim,
            src_page_id,
            item_size_bytes,
            head_id,
            head_num,
            page_size);
        char* dst_v_ptr = DstOffsetFn(
            static_cast<char*>(dst_v),
            dst_v_layer_tbl,
            layer_id,
            dst_layout_dim,
            dst_page_id,
            item_size_bytes,
            head_id,
            head_num,
            page_size);
        transfer_item_warp(lane_id, src_v_ptr, dst_v_ptr, head_size_bytes);
      }
    }
  }
}
```
**EN:** This section implements `SrcOffsetFn`, `DstOffsetFn`, `transfer_item_warp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`SrcOffsetFn`、`DstOffsetFn`、`transfer_item_warp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 209-236: Kernel implementation
```cpp
template <auto SrcOffsetFn, auto DstOffsetFn, bool IsMLA>
__global__ void transfer_kernel_impl(
    const void* __restrict__ src_k,
    void* __restrict__ dst_k,
    const void* __restrict__ src_v,
    void* __restrict__ dst_v,
    const int64_t* __restrict__ src_indices,
    const int64_t* __restrict__ dst_indices,
    int64_t start_layer_id,
    int64_t num_layers_to_process,
    int64_t num_items,
    int64_t items_per_warp,
    int64_t item_size_bytes,
    int64_t src_layout_dim,
    int64_t dst_layout_dim,
    const uintptr_t* __restrict__ src_k_layer_tbl,
    const uintptr_t* __restrict__ dst_k_layer_tbl,
    const uintptr_t* __restrict__ src_v_layer_tbl,
    const uintptr_t* __restrict__ dst_v_layer_tbl) {
  int32_t tid = blockIdx.x * blockDim.x + threadIdx.x;
  int32_t lane_id = tid % WARP_SIZE;
  int32_t warp_id = tid / WARP_SIZE;

  for (int i = 0; i < items_per_warp; ++i) {
    int64_t item_id = warp_id * items_per_warp + i;
    if (item_id >= num_items) {
      break;
    }
```
**EN:** This section implements `transfer_kernel_impl`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`transfer_kernel_impl`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 237-258: Device helpers and synchronization
```cpp
    const int64_t src_page_id = src_indices[item_id];
    const int64_t dst_page_id = dst_indices[item_id];

    // Loop over layers if necessary
    for (int64_t layer_id = start_layer_id; layer_id < start_layer_id + num_layers_to_process; ++layer_id) {
      const char* src_ptr = SrcOffsetFn(
          static_cast<const char*>(src_k), src_k_layer_tbl, layer_id, src_layout_dim, src_page_id, item_size_bytes);
      char* dst_ptr = DstOffsetFn(
          static_cast<char*>(dst_k), dst_k_layer_tbl, layer_id, dst_layout_dim, dst_page_id, item_size_bytes);
      transfer_item_warp(lane_id, src_ptr, dst_ptr, item_size_bytes);

      if constexpr (!IsMLA) {
        const char* src_v_ptr = SrcOffsetFn(
            static_cast<const char*>(src_v), src_v_layer_tbl, layer_id, src_layout_dim, src_page_id, item_size_bytes);
        char* dst_v_ptr = DstOffsetFn(
            static_cast<char*>(dst_v), dst_v_layer_tbl, layer_id, dst_layout_dim, dst_page_id, item_size_bytes);
        transfer_item_warp(lane_id, src_v_ptr, dst_v_ptr, item_size_bytes);
      }
    }
  }
}
```
**EN:** This section implements `SrcOffsetFn`, `DstOffsetFn`, `transfer_item_warp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`SrcOffsetFn`、`DstOffsetFn`、`transfer_item_warp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 259-286: Device helpers and synchronization
```cpp
template <auto SrcOffsetFn, auto DstOffsetFn, bool IsMLA, bool PageHeadLayout = false>
void transfer_kv_launcher(
    const at::Tensor& src_k,
    at::Tensor& dst_k,
    const at::Tensor& src_v,
    at::Tensor& dst_v,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t start_layer_id,
    int64_t num_layers_to_process,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t dst_layout_dim,
    const at::Tensor& src_k_layers,
    const at::Tensor& dst_k_layers,
    const at::Tensor& src_v_layers,
    const at::Tensor& dst_v_layers,
    int64_t block_quota,
    int64_t num_warps_per_block,
    const int64_t page_size = 16,
    const int64_t head_num = 1) {
  TORCH_CHECK(src_indices.is_cuda(), "Source indices must be a CUDA tensor");
  TORCH_CHECK(dst_indices.is_cuda(), "Destination indices must be a CUDA tensor");
  TORCH_CHECK(src_indices.scalar_type() == at::kLong, "Source indices must be of type long");
  TORCH_CHECK(dst_indices.scalar_type() == at::kLong, "Destination indices must be of type long");
  TORCH_CHECK(src_indices.numel() == dst_indices.numel(), "Source and destination indices must have the same length");
  TORCH_CHECK(item_size % 8 == 0, "Item byte size must be divisible by 8");
```
**EN:** This section implements `transfer_kv_launcher`, `TORCH_CHECK`, `page_size`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_launcher`、`TORCH_CHECK`、`page_size`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 287-314: Kernel implementation
```cpp
  auto div_up = [](int64_t x, int64_t y) { return (x + y - 1) / y; };
  const int64_t num_items = src_indices.numel();
  const int64_t items_per_warp = div_up(num_items, block_quota * num_warps_per_block);
  const int32_t num_blocks = div_up(num_items, items_per_warp * num_warps_per_block);
  dim3 grid_dim(num_blocks, 1, 1);
  const int32_t threads_per_block = num_warps_per_block * WARP_SIZE;

  const void* src_k_ptr = src_k.defined() ? src_k.data_ptr() : nullptr;
  void* dst_k_ptr = dst_k.defined() ? dst_k.data_ptr() : nullptr;
  const void* src_v_ptr = IsMLA || !src_v.defined() ? nullptr : src_v.data_ptr();
  void* dst_v_ptr = IsMLA || !dst_v.defined() ? nullptr : dst_v.data_ptr();
  const uintptr_t* src_k_tbl_ptr = src_k_layers.defined() ? src_k_layers.data_ptr<uintptr_t>() : nullptr;
  const uintptr_t* dst_k_tbl_ptr = dst_k_layers.defined() ? dst_k_layers.data_ptr<uintptr_t>() : nullptr;
  const uintptr_t* src_v_tbl_ptr = IsMLA || !src_v_layers.defined() ? nullptr : src_v_layers.data_ptr<uintptr_t>();
  const uintptr_t* dst_v_tbl_ptr = IsMLA || !dst_v_layers.defined() ? nullptr : dst_v_layers.data_ptr<uintptr_t>();

  cudaStream_t torch_current_stream = at::cuda::getCurrentCUDAStream();
  if constexpr (PageHeadLayout) {
    transfer_page_head_kernel_impl<SrcOffsetFn, DstOffsetFn><<<grid_dim, threads_per_block, 0, torch_current_stream>>>(
        src_k_ptr,
        dst_k_ptr,
        src_v_ptr,
        dst_v_ptr,
        src_indices.data_ptr<int64_t>(),
        dst_indices.data_ptr<int64_t>(),
        start_layer_id,
        num_layers_to_process,
        num_items,
```
**EN:** This section implements `numel`, `div_up`, `grid_dim`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`numel`、`div_up`、`grid_dim`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 315-344: Kernel implementation
```cpp
        items_per_warp,
        item_size,
        src_layout_dim,
        dst_layout_dim,
        src_k_tbl_ptr,
        dst_k_tbl_ptr,
        src_v_tbl_ptr,
        dst_v_tbl_ptr,
        page_size,
        head_num);
  } else {
    transfer_kernel_impl<SrcOffsetFn, DstOffsetFn, IsMLA><<<grid_dim, threads_per_block, 0, torch_current_stream>>>(
        src_k_ptr,
        dst_k_ptr,
        src_v_ptr,
        dst_v_ptr,
        src_indices.data_ptr<int64_t>(),
        dst_indices.data_ptr<int64_t>(),
        start_layer_id,
        num_layers_to_process,
        num_items,
        items_per_warp,
        item_size,
        src_layout_dim,
        dst_layout_dim,
        src_k_tbl_ptr,
        dst_k_tbl_ptr,
        src_v_tbl_ptr,
        dst_v_tbl_ptr);
  }
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 345-377: Device helpers and synchronization
```cpp
  C10_CUDA_KERNEL_LAUNCH_CHECK();
}

void transfer_kv_per_layer(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf<const char>, get_global_offset_lf<char>, false>(
      src_k,
      dst_k,
      src_v,
      dst_v,
      src_indices,
      dst_indices,
      0,
      1,
      item_size,
      0,
      0,
      empty,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_per_layer`, `C10_CUDA_KERNEL_LAUNCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer`、`C10_CUDA_KERNEL_LAUNCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 378-410: Device helpers and synchronization
```cpp

void transfer_kv_per_layer_pf_lf(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_pf<const char>, get_global_offset_lf<char>, false>(
      src_k,
      dst_k,
      src_v,
      dst_v,
      src_indices,
      dst_indices,
      layer_id,
      1,
      item_size,
      src_layout_dim,
      0,
      empty,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_per_layer_pf_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_pf_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 411-438: Device helpers and synchronization
```cpp

void transfer_kv_per_layer_ph_lf(
    const at::Tensor src_k,
    at::Tensor dst_k,
    const at::Tensor src_v,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t page_size,
    int64_t head_num,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_ph<const char>, get_global_offset_per_head_lf<char>, false, true>(
      src_k,
      dst_k,
      src_v,
      dst_v,
      src_indices,
      dst_indices,
      layer_id,
      1,
      item_size,
      src_layout_dim,
      0,
```
**EN:** This section implements `transfer_kv_per_layer_ph_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_ph_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 439-466: Device helpers and synchronization
```cpp
      empty,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block,
      page_size,
      head_num);
}

void transfer_kv_all_layer(
    const at::Tensor src_k_layers,
    const at::Tensor dst_k_layers,
    const at::Tensor src_v_layers,
    const at::Tensor dst_v_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  TORCH_CHECK(num_layers == src_k_layers.size(0), "Number of layers in source k tensor does not match num_layers");
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf_tbl<const char>, get_global_offset_lf_tbl<char>, false>(
      empty,
      empty,
      empty,
      empty,
```
**EN:** This section implements `transfer_kv_all_layer`, `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer`、`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 467-494: Device helpers and synchronization
```cpp
      src_indices,
      dst_indices,
      0,
      num_layers,
      item_size,
      0,
      0,
      src_k_layers,
      dst_k_layers,
      src_v_layers,
      dst_v_layers,
      block_quota,
      num_warps_per_block);
}

void transfer_kv_all_layer_lf_pf(
    const at::Tensor src_k_layers,
    at::Tensor dst_k,
    const at::Tensor src_v_layers,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  TORCH_CHECK(num_layers == src_k_layers.size(0), "Number of layers in source k tensor does not match num_layers");
```
**EN:** This section implements `transfer_kv_all_layer_lf_pf`, `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer_lf_pf`、`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 495-522: Device helpers and synchronization
```cpp
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf_tbl<const char>, get_global_offset_pf<char>, false>(
      empty,
      dst_k,
      empty,
      dst_v,
      src_indices,
      dst_indices,
      0,
      num_layers,
      item_size,
      0,
      dst_layout_dim,
      src_k_layers,
      empty,
      src_v_layers,
      empty,
      block_quota,
      num_warps_per_block);
}

void transfer_kv_all_layer_lf_ph(
    const at::Tensor src_k_layers,
    at::Tensor dst_k,
    const at::Tensor src_v_layers,
    at::Tensor dst_v,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 523-552: Device helpers and synchronization
```cpp
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t page_size,
    int64_t head_num,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  TORCH_CHECK(num_layers == src_k_layers.size(0), "Number of layers in source k tensor does not match num_layers");
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_per_head_lf_tbl<const char>, get_global_offset_ph<char>, false, true>(
      empty,
      dst_k,
      empty,
      dst_v,
      src_indices,
      dst_indices,
      0,
      num_layers,
      item_size,
      0,
      dst_layout_dim,
      src_k_layers,
      empty,
      src_v_layers,
      empty,
      block_quota,
      num_warps_per_block,
      page_size,
      head_num);
}
```
**EN:** This section implements `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 553-581: Device helpers and synchronization
```cpp

void transfer_kv_per_layer_mla(
    const at::Tensor src,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf<const char>, get_global_offset_lf<char>, true>(
      src,
      dst,
      empty,
      empty,
      src_indices,
      dst_indices,
      0,
      1,
      item_size,
      0,
      0,
      empty,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_per_layer_mla`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_mla`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 582-612: Device helpers and synchronization
```cpp

void transfer_kv_per_layer_mla_pf_lf(
    const at::Tensor src,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t layer_id,
    int64_t item_size,
    int64_t src_layout_dim,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_pf<const char>, get_global_offset_lf<char>, true>(
      src,
      dst,
      empty,
      empty,
      src_indices,
      dst_indices,
      layer_id,
      1,
      item_size,
      src_layout_dim,
      0,
      empty,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_per_layer_mla_pf_lf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_per_layer_mla_pf_lf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 613-643: Device helpers and synchronization
```cpp

void transfer_kv_all_layer_mla(
    const at::Tensor src_layers,
    const at::Tensor dst_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  TORCH_CHECK(num_layers == src_layers.size(0), "Number of layers in source tensor does not match num_layers");
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf_tbl<const char>, get_global_offset_lf_tbl<char>, true>(
      empty,
      empty,
      empty,
      empty,
      src_indices,
      dst_indices,
      0,
      num_layers,
      item_size,
      0,
      0,
      src_layers,
      dst_layers,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_all_layer_mla`, `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer_mla`、`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 644-675: Device helpers and synchronization
```cpp

void transfer_kv_all_layer_mla_lf_pf(
    const at::Tensor src_layers,
    at::Tensor dst,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t item_size,
    int64_t dst_layout_dim,
    int64_t num_layers,
    int64_t block_quota,
    int64_t num_warps_per_block) {
  TORCH_CHECK(num_layers == src_layers.size(0), "Number of layers in source tensor does not match num_layers");
  at::Tensor empty;
  transfer_kv_launcher<get_global_offset_lf_tbl<const char>, get_global_offset_pf<char>, true>(
      empty,
      dst,
      empty,
      empty,
      src_indices,
      dst_indices,
      0,
      num_layers,
      item_size,
      0,
      dst_layout_dim,
      src_layers,
      empty,
      empty,
      empty,
      block_quota,
      num_warps_per_block);
}
```
**EN:** This section implements `transfer_kv_all_layer_mla_lf_pf`, `TORCH_CHECK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`transfer_kv_all_layer_mla_lf_pf`、`TORCH_CHECK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 676-703: Runtime integration and dispatch
```cpp

inline void transfer_page_direct(
    const at::Tensor src_buffer,
    at::Tensor dst_buffer,
    int64_t src_page_index,
    int64_t dst_page_index,
    int64_t page_size) {
  dst_buffer.slice(0, dst_page_index, dst_page_index + page_size)
      .copy_(
          src_buffer.slice(0, src_page_index, src_page_index + page_size),
          /* non_blocking= */ true);
}

void transfer_kv_direct(
    const std::vector<at::Tensor>& src_layers,
    std::vector<at::Tensor> dst_layers,
    const at::Tensor src_indices,
    const at::Tensor dst_indices,
    int64_t page_size) {
  TORCH_CHECK(
      src_layers.size() == dst_layers.size(), "Source and destination layers must have the same number of layers");
  TORCH_CHECK(src_indices.numel() == dst_indices.numel(), "Source and destination indices must have the same length");
  TORCH_CHECK(page_size > 0, "Page size must be positive");
  TORCH_CHECK(src_indices.numel() % page_size == 0, "Source indices size must be divisible by page size");

  auto src_indices_cpu = src_indices.cpu();
  auto dst_indices_cpu = dst_indices.cpu();
```
**EN:** This section uses `transfer_page_direct`, `transfer_kv_direct`, `slice` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`transfer_page_direct`、`transfer_kv_direct`、`slice`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 704-732: Control flow and branching
```cpp
  const auto num_indices = src_indices_cpu.numel();
  const int64_t num_layers = src_layers.size();
  int64_t* src_indices_ptr = src_indices_cpu.data_ptr<int64_t>();
  int64_t* dst_indices_ptr = dst_indices_cpu.data_ptr<int64_t>();

  int64_t start_index = 0;
  int64_t end_index = 0;

  for (int64_t i = 0; i < num_indices; ++i) {
    if (i < num_indices - 1) {
      auto src_diff = src_indices_ptr[i + 1] - src_indices_ptr[i];
      auto dst_diff = dst_indices_ptr[i + 1] - dst_indices_ptr[i];

      if (src_diff == 1 && dst_diff == 1) {
        continue;
      }
      end_index = i + 1;
    } else {  // last batch
      end_index = num_indices;
    }
    auto src_index = src_indices_ptr[start_index];
    auto dst_index = dst_indices_ptr[start_index];
    auto num_tokens = end_index - start_index;

    for (int64_t j = 0; j < num_layers; ++j) {
      transfer_page_direct(src_layers[j], dst_layers[j], src_index, dst_index, num_tokens);
    }
    start_index = end_index;
  }
```
**EN:** This section drives `numel`, `transfer_page_direct`, `num_indices` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`numel`、`transfer_page_direct`、`num_indices`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 733-760: Runtime integration and dispatch
```cpp
}

template <bool IsLf2Pf>
inline void transfer_kv_page_first_direct_impl(
    const std::vector<at::Tensor>& src_ptrs,
    std::vector<at::Tensor> dst_ptrs,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t start_layer_id,
    int64_t page_size) {
  TORCH_CHECK(src_indices.numel() == dst_indices.numel(), "Source and destination indices must have the same length");
  TORCH_CHECK(page_size > 0, "Page size must be positive");
  TORCH_CHECK(src_indices.numel() % page_size == 0, "Source indices size must be divisible by page size");

  auto src_indices_cpu = src_indices.cpu();
  auto dst_indices_cpu = dst_indices.cpu();
  const int64_t num_pages = src_indices_cpu.size(0) / page_size;
  int64_t* src_indices_ptr = src_indices_cpu.data_ptr<int64_t>();
  int64_t* dst_indices_ptr = dst_indices_cpu.data_ptr<int64_t>();

  auto fallback_to_page_copy = [&]() {
    if constexpr (IsLf2Pf) {
      const bool is_mla = dst_ptrs.size() == 1;
      const int64_t num_layers = is_mla ? src_ptrs.size() : src_ptrs.size() / 2;
      for (const auto i : c10::irange(num_pages)) {
        const int64_t s_index = src_indices_ptr[i * page_size];
        const int64_t d_index = dst_indices_ptr[i * page_size] / page_size;
        for (int64_t j = 0; j < num_layers; ++j) {
```
**EN:** This section uses `transfer_kv_page_first_direct_impl`, `TORCH_CHECK`, `cpu` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`transfer_kv_page_first_direct_impl`、`TORCH_CHECK`、`cpu`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 761-789: Runtime integration and dispatch
```cpp
          transfer_page_direct(
              src_ptrs[j], dst_ptrs[0].select(0, d_index).select(0, start_layer_id + j), s_index, 0, page_size);
          if (!is_mla) {
            transfer_page_direct(
                src_ptrs[j + num_layers],
                dst_ptrs[1].select(0, d_index).select(0, start_layer_id + j),
                s_index,
                0,
                page_size);
          }
        }
      }
    } else {
      const bool is_mla = src_ptrs.size() == 1;
      const int64_t num_layers = is_mla ? dst_ptrs.size() : dst_ptrs.size() / 2;
      for (const auto i : c10::irange(num_pages)) {
        const int64_t s_index = src_indices_ptr[i * page_size] / page_size;
        const int64_t d_index = dst_indices_ptr[i * page_size];
        for (int64_t j = 0; j < num_layers; ++j) {
          transfer_page_direct(
              src_ptrs[0].select(0, s_index).select(0, start_layer_id + j), dst_ptrs[j], 0, d_index, page_size);
          if (!is_mla) {
            transfer_page_direct(
                src_ptrs[1].select(0, s_index).select(0, start_layer_id + j),
                dst_ptrs[j + num_layers],
                0,
                d_index,
                page_size);
          }
```
**EN:** This section uses `transfer_page_direct`, `is_mla`, `num_layers` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`transfer_page_direct`、`is_mla`、`num_layers`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 790-814: Runtime integration and dispatch
```cpp
        }
      }
    }
  };

#if defined(USE_ROCM) || !defined(CUDA_VERSION) || CUDA_VERSION < 12080
  fallback_to_page_copy();
  return;

#else
  // Driver capability gate: only use cudaMemcpyBatchAsync on CUDA 12.8+ drivers.
  int driver_version = 0;
  cudaError_t driver_version_err = cudaDriverGetVersion(&driver_version);
  if (driver_version_err != cudaSuccess || driver_version < 12080) {
    fallback_to_page_copy();
    return;
  }

  // Symbol gate: runtime may not expose cudaMemcpyBatchAsync in some environments.
  static void* cuda_memcpy_batch_async_sym = dlsym(RTLD_DEFAULT, "cudaMemcpyBatchAsync");
  if (cuda_memcpy_batch_async_sym == nullptr) {
    fallback_to_page_copy();
    return;
  }
```
**EN:** This section uses `defined`, `cudaDriverGetVersion`, `fallback_to_page_copy` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`cudaDriverGetVersion`、`fallback_to_page_copy`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 815-840: Runtime integration and dispatch
```cpp
  // CUDA 13.0 removed the failIdx parameter from cudaMemcpyBatchAsync. The ABI
  // of the dlsym'd symbol is determined by the libcudart loaded in this process,
  // not the host driver — a cu12 runtime on a cu13 driver host (common in
  // containers) still exposes the 9-param v12 signature. Dispatching on the
  // driver version here would segfault in that case (verified empirically).
  // Use cudaRuntimeGetVersion so the signature follows the runtime. The
  // runtime version is process-constant, so cache the query (static init is
  // thread-safe in C++11+) to keep the KV-transfer hot path free of a redundant
  // runtime API call per invocation.
  static int runtime_version = 0;
  static cudaError_t runtime_version_err = cudaRuntimeGetVersion(&runtime_version);
  if (runtime_version_err != cudaSuccess) {
    fallback_to_page_copy();
    return;
  }
  static const bool use_v13_signature = runtime_version >= 13000;

  size_t num_copies = 0;
  std::vector<void*> batch_srcs;
  std::vector<void*> batch_dsts;
  std::vector<size_t> batch_sizes;
  std::vector<size_t> attrs_idxs(1, 0);
  cudaMemcpyAttributes attrs{};
  const int device_id = at::cuda::current_device();
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
**EN:** This section uses `cudaRuntimeGetVersion`, `fallback_to_page_copy`, `attrs_idxs` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cudaRuntimeGetVersion`、`fallback_to_page_copy`、`attrs_idxs`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 841-867: Runtime integration and dispatch
```cpp
  auto append_copy = [&](void* src, void* dst, size_t size_bytes) {
    batch_srcs.push_back(src);
    batch_dsts.push_back(dst);
    batch_sizes.push_back(size_bytes);
  };

  if constexpr (IsLf2Pf) {
    const bool is_mla = dst_ptrs.size() == 1;
    const int64_t num_layers = is_mla ? src_ptrs.size() : src_ptrs.size() / 2;

    const int64_t dst_stride0 = dst_ptrs[0].stride(0);
    const int64_t dst_stride1 = dst_ptrs[0].stride(1);
    const int64_t src_stride0 = src_ptrs[0].stride(0);
    const int64_t elem_size = dst_ptrs[0].element_size();
    const int64_t copy_size_bytes = page_size * src_stride0 * elem_size;
    attrs.srcAccessOrder = cudaMemcpySrcAccessOrderStream;
    attrs.srcLocHint.type = cudaMemLocationTypeDevice;
    attrs.srcLocHint.id = device_id;
    attrs.dstLocHint.type = cudaMemLocationTypeHost;
    attrs.dstLocHint.id = 0;
    attrs.flags = 0;

    num_copies = static_cast<size_t>(num_pages) * static_cast<size_t>(num_layers) * static_cast<size_t>(is_mla ? 1 : 2);
    batch_srcs.reserve(num_copies);
    batch_dsts.reserve(num_copies);
    batch_sizes.reserve(num_copies);
```
**EN:** This section uses `push_back`, `stride`, `element_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`push_back`、`stride`、`element_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 868-891: Runtime integration and dispatch
```cpp
    for (const auto i : c10::irange(num_pages)) {
      auto s_index = src_indices_ptr[i * page_size];
      auto d_index = dst_indices_ptr[i * page_size] / page_size;

      for (int64_t j = 0; j < num_layers; ++j) {
        const char* src_k_ptr = static_cast<const char*>(src_ptrs[j].data_ptr()) + s_index * src_stride0 * elem_size;
        char* dst_k_ptr = static_cast<char*>(dst_ptrs[0].data_ptr()) + d_index * dst_stride0 * elem_size +
                          (start_layer_id + j) * dst_stride1 * elem_size;
        append_copy(const_cast<char*>(src_k_ptr), dst_k_ptr, copy_size_bytes);

        if (!is_mla) {
          const char* src_v_ptr =
              static_cast<const char*>(src_ptrs[j + num_layers].data_ptr()) + s_index * src_stride0 * elem_size;
          char* dst_v_ptr = static_cast<char*>(dst_ptrs[1].data_ptr()) + d_index * dst_stride0 * elem_size +
                            (start_layer_id + j) * dst_stride1 * elem_size;
          append_copy(const_cast<char*>(src_v_ptr), dst_v_ptr, copy_size_bytes);
        }
      }
    }

  } else {
    const bool is_mla = src_ptrs.size() == 1;
    const int64_t num_layers = is_mla ? dst_ptrs.size() : dst_ptrs.size() / 2;
```
**EN:** This section uses `append_copy`, `src_k_ptr`, `elem_size` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`append_copy`、`src_k_ptr`、`elem_size`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 892-918: Runtime integration and dispatch
```cpp
    const int64_t src_stride0 = src_ptrs[0].stride(0);
    const int64_t src_stride1 = src_ptrs[0].stride(1);
    const int64_t dst_stride0 = dst_ptrs[0].stride(0);
    const int64_t elem_size = src_ptrs[0].element_size();
    const int64_t copy_size_bytes = page_size * dst_stride0 * elem_size;
    attrs.srcAccessOrder = cudaMemcpySrcAccessOrderStream;
    attrs.srcLocHint.type = cudaMemLocationTypeHost;
    attrs.srcLocHint.id = 0;
    attrs.dstLocHint.type = cudaMemLocationTypeDevice;
    attrs.dstLocHint.id = device_id;
    attrs.flags = 0;

    num_copies = static_cast<size_t>(num_pages) * static_cast<size_t>(num_layers) * static_cast<size_t>(is_mla ? 1 : 2);
    batch_srcs.reserve(num_copies);
    batch_dsts.reserve(num_copies);
    batch_sizes.reserve(num_copies);

    for (const auto i : c10::irange(num_pages)) {
      auto s_index = src_indices_ptr[i * page_size] / page_size;
      auto d_index = dst_indices_ptr[i * page_size];

      for (int64_t j = 0; j < num_layers; ++j) {
        const char* src_k_ptr = static_cast<const char*>(src_ptrs[0].data_ptr()) + s_index * src_stride0 * elem_size +
                                (start_layer_id + j) * src_stride1 * elem_size;
        char* dst_k_ptr = static_cast<char*>(dst_ptrs[j].data_ptr()) + d_index * dst_stride0 * elem_size;
        append_copy(const_cast<char*>(src_k_ptr), dst_k_ptr, copy_size_bytes);
```
**EN:** This section uses `stride`, `element_size`, `reserve` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`stride`、`element_size`、`reserve`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 919-946: Runtime integration and dispatch
```cpp
        if (!is_mla) {
          const char* src_v_ptr = static_cast<const char*>(src_ptrs[1].data_ptr()) + s_index * src_stride0 * elem_size +
                                  (start_layer_id + j) * src_stride1 * elem_size;
          char* dst_v_ptr = static_cast<char*>(dst_ptrs[j + num_layers].data_ptr()) + d_index * dst_stride0 * elem_size;
          append_copy(const_cast<char*>(src_v_ptr), dst_v_ptr, copy_size_bytes);
        }
      }
    }
  }

  TORCH_CHECK(batch_srcs.size() == num_copies, "Batch memcpy count mismatch");
  if (num_copies > 0) {
    cudaError_t err;
    size_t fail_idx = std::numeric_limits<size_t>::max();
    if (use_v13_signature) {
      using FnV13 = cudaError_t (*)(
          void* const*,
          const void* const*,
          const size_t*,
          size_t,
          cudaMemcpyAttributes*,
          size_t*,
          size_t,
          cudaStream_t);
      auto fn = reinterpret_cast<FnV13>(cuda_memcpy_batch_async_sym);
      err = fn(
          batch_dsts.data(), batch_srcs.data(), batch_sizes.data(), num_copies, &attrs, attrs_idxs.data(), 1, stream);
    } else {
```
**EN:** This section uses `append_copy`, `TORCH_CHECK`, `max` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`append_copy`、`TORCH_CHECK`、`max`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 947-971: Runtime integration and dispatch
```cpp
      using FnV12 = cudaError_t (*)(
          void**, void**, size_t*, size_t, cudaMemcpyAttributes*, size_t*, size_t, size_t*, cudaStream_t);
      auto fn = reinterpret_cast<FnV12>(cuda_memcpy_batch_async_sym);
      err =
          fn(batch_dsts.data(),
             batch_srcs.data(),
             batch_sizes.data(),
             num_copies,
             &attrs,
             attrs_idxs.data(),
             1,
             &fail_idx,
             stream);
    }
    if (err == cudaErrorNotSupported || err == cudaErrorCallRequiresNewerDriver) {
      fallback_to_page_copy();
      return;
    }
    if (err != cudaSuccess) {
      TORCH_CHECK(false, "cudaMemcpyBatchAsync failed. failIdx=", fail_idx, " error=", cudaGetErrorString(err));
    }
  }
#endif
}
```
**EN:** This section uses `cudaError_t`, `fn`, `fallback_to_page_copy` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cudaError_t`、`fn`、`fallback_to_page_copy`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 972-989: Runtime integration and dispatch
```cpp
void transfer_kv_per_layer_direct_pf_lf(
    const std::vector<at::Tensor>& src_ptrs,
    std::vector<at::Tensor> dst_ptrs,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t layer_id,
    int64_t page_size) {
  transfer_kv_page_first_direct_impl<false>(src_ptrs, dst_ptrs, src_indices, dst_indices, layer_id, page_size);
}

void transfer_kv_all_layer_direct_lf_pf(
    const std::vector<at::Tensor>& src_ptrs,
    std::vector<at::Tensor> dst_ptrs,
    const at::Tensor& src_indices,
    const at::Tensor& dst_indices,
    int64_t page_size) {
  transfer_kv_page_first_direct_impl<true>(src_ptrs, dst_ptrs, src_indices, dst_indices, 0, page_size);
}
```
**EN:** This section uses `transfer_kv_per_layer_direct_pf_lf`, `transfer_kv_all_layer_direct_lf_pf` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`transfer_kv_per_layer_direct_pf_lf`、`transfer_kv_all_layer_direct_lf_pf`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **MUSA portability / MUSA 可移植性**: Contains conditional logic for the MUSA toolchain or runtime. / 包含面向 MUSA 工具链或运行时的条件分支。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `pytorch_extension_utils.h`, `pytorch_extension_utils_rocm.h`, `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAException.h`, `c10/util/irange.h`, `cuda_runtime.h`, `cstdint`, `limits`, `vector`, `dlfcn.h`
- **Path context / 路径上下文**: kvcacheio / transfer.cu
