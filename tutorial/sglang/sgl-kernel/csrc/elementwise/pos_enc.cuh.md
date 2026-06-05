# pos_enc.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/elementwise/pos_enc.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Headers and compile-time setup
```cpp
/*
 * Copyright (c) 2023 by FlashInfer team.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
#ifndef SGL_POS_ENC_CUH_
#define SGL_POS_ENC_CUH_

#include <flashinfer/pos_enc.cuh>  // upstream

namespace flashinfer {

namespace kv_buffer_saver {
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 25-48: Device helpers and synchronization
```cpp
template <typename DType, typename IdType, uint32_t vec_size>
__device__ __forceinline__ void prepare(
    vec_t<float, vec_size>& v_vec,
    IdType& kv_cache_offset,
    DType* v,
    IdType* kv_cache_loc,
    uint32_t idx,
    uint32_t tx,
    uint32_t kv_head_idx,
    size_t v_stride_n,
    size_t v_stride_h) {
  kv_cache_offset = kv_cache_loc[idx];

  DType* v_ptr = v + get_elem_offset_impl(idx, kv_head_idx, 0, v_stride_n, v_stride_h);
  v_vec.cast_load(v_ptr + tx * vec_size);
}

template <typename DType, typename IdType, uint32_t vec_size>
__device__ __forceinline__ void save(
    IdType& kv_cache_offset,
    vec_t<float, vec_size>& k_vec,
    vec_t<float, vec_size>& v_vec,
    DType* k_buffer,
    DType* v_buffer,
```
**EN:** This section implements `prepare`, `get_elem_offset_impl`, `cast_load`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`prepare`、`get_elem_offset_impl`、`cast_load`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 49-72: Templates, aliases, and constants
```cpp
    uint32_t idx,
    uint32_t tx,
    uint32_t kv_head_idx,
    size_t k_buffer_stride_n,
    size_t k_buffer_stride_h,
    size_t v_buffer_stride_n,
    size_t v_buffer_stride_h) {
  DType* k_buffer_ptr =
      k_buffer + get_elem_offset_impl(kv_cache_offset, kv_head_idx, 0, k_buffer_stride_n, k_buffer_stride_h);
  DType* v_buffer_ptr =
      v_buffer + get_elem_offset_impl(kv_cache_offset, kv_head_idx, 0, v_buffer_stride_n, v_buffer_stride_h);
  k_vec.cast_store(k_buffer_ptr + tx * vec_size);
  v_vec.cast_store(v_buffer_ptr + tx * vec_size);
}

}  // namespace kv_buffer_saver

template <
    bool save_kv_cache,
    bool interleave,
    uint32_t head_dim,
    uint32_t vec_size,
    uint32_t bdx,
    typename DType,
```
**EN:** This section defines `get_elem_offset_impl`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`get_elem_offset_impl`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 73-96: Kernel implementation
```cpp
    typename IdType>
__global__ void BatchQKApplyRotaryPosIdsCosSinCacheEnhancedHeadParallelismKernel(
    DType* q,
    DType* k,
    DType* v,
    DType* q_rope,
    DType* k_rope,
    DType* k_buffer,
    DType* v_buffer,
    float* __restrict__ cos_sin_cache,
    IdType* __restrict__ pos_ids,
    uint32_t nnz,
    uint32_t num_qo_heads,
    uint32_t num_kv_heads,
    uint32_t rotary_dim,
    size_t q_stride_n,
    size_t q_stride_h,
    size_t k_stride_n,
    size_t k_stride_h,
    size_t v_stride_n,
    size_t v_stride_h,
    size_t q_rope_stride_n,
    size_t q_rope_stride_h,
    size_t k_rope_stride_n,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 97-117: Device helpers and synchronization
```cpp
    size_t k_rope_stride_h,
    size_t k_buffer_stride_n,
    size_t k_buffer_stride_h,
    size_t v_buffer_stride_n,
    size_t v_buffer_stride_h,
    IdType* __restrict__ kv_cache_loc) {
  uint32_t bx = blockIdx.x, tx = threadIdx.x, ty = threadIdx.y;
  uint32_t by = blockIdx.y;
  const uint32_t bdy = blockDim.y;

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900))
  cudaGridDependencySynchronize();
#endif

  vec_t<float, vec_size> cos, sin;
  if (bx * bdy + ty < nnz) {
    const uint32_t idx = bx * bdy + ty;
    const IdType pos = pos_ids[idx];

    const int half_rotary_dim = rotary_dim / 2;
```
**EN:** This section implements `bdy`, `idx`, `pos`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`bdy`、`idx`、`pos`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 118-145: Templates, aliases, and constants
```cpp
    // 1. if interleave:
    //  - cos = cos_sin_cache[pos_id][tx * vec_size // 2]
    //  - sin = cos_sin_cache[pos_id][(rot_dim // 2) + tx * vec_size // 2]
    // 2. if not interleave
    //  - cos = cos_cache[pos_id][(tx * vec_size) % (rot_dim // 2)]
    //  - sin = sin_cache[pos_id][(rot_dim // 2) + (tx * vec_size) % (rot_dim // 2)]
    if (tx * vec_size < rotary_dim) {
      int sin_offset = rotary_dim / 2;
      int vec_idx;
      if constexpr (interleave) {
        vec_idx = (tx * vec_size) / 2;  // Force integer division
      } else {
        vec_idx = (tx * vec_size) % half_rotary_dim;  // Use half_rotary_dim
      }
      cos.load(cos_sin_cache + (pos * rotary_dim) + vec_idx);
      sin.load(cos_sin_cache + (pos * rotary_dim) + (sin_offset + vec_idx));
    }

    if (by < num_qo_heads) {
      uint32_t qo_head_idx = by;
      DType* q_ptr = q + get_elem_offset_impl(idx, qo_head_idx, 0, q_stride_n, q_stride_h);
      DType* q_rope_ptr = q_rope + get_elem_offset_impl(idx, qo_head_idx, 0, q_rope_stride_n, q_rope_stride_h);
      vec_t<float, vec_size> q_vec;
      if constexpr (interleave) {
        q_vec = vec_apply_llama_rope_cos_sin_interleave_reuse_half<vec_size, bdx>(q_ptr, cos, sin, rotary_dim);
      } else {
        q_vec = vec_apply_llama_rope_cos_sin<vec_size, bdx>(q_ptr, cos, sin, rotary_dim);
      }
```
**EN:** This section defines `load`, `get_elem_offset_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`load`、`get_elem_offset_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 146-167: Templates, aliases, and constants
```cpp
      q_vec.cast_store(q_rope_ptr + tx * vec_size);
    } else {
      uint32_t kv_head_idx = by - num_qo_heads;
      DType* k_ptr = k + get_elem_offset_impl(idx, kv_head_idx, 0, k_stride_n, k_stride_h);

      DType* k_rope_ptr = k_rope + get_elem_offset_impl(idx, kv_head_idx, 0, k_rope_stride_n, k_rope_stride_h);

      vec_t<float, vec_size> v_vec;
      IdType kv_cache_offset;
      if constexpr (save_kv_cache) {
        kv_buffer_saver::prepare<DType, IdType, vec_size>(
            v_vec, kv_cache_offset, v, kv_cache_loc, idx, tx, kv_head_idx, v_stride_n, v_stride_h);
      }

      vec_t<float, vec_size> k_vec;
      if constexpr (interleave) {
        k_vec = vec_apply_llama_rope_cos_sin_interleave_reuse_half<vec_size, bdx>(k_ptr, cos, sin, rotary_dim);
      } else {
        k_vec = vec_apply_llama_rope_cos_sin<vec_size, bdx>(k_ptr, cos, sin, rotary_dim);
      }
      k_vec.cast_store(k_rope_ptr + tx * vec_size);
```
**EN:** This section defines `cast_store`, `get_elem_offset_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_store`、`get_elem_offset_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 168-190: Templates, aliases, and constants
```cpp
      if constexpr (save_kv_cache) {
        kv_buffer_saver::save<DType, IdType, vec_size>(
            kv_cache_offset,
            k_vec,
            v_vec,
            k_buffer,
            v_buffer,
            idx,
            tx,
            kv_head_idx,
            k_buffer_stride_n,
            k_buffer_stride_h,
            v_buffer_stride_n,
            v_buffer_stride_h);
      }
    }
  }

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900))
  cudaTriggerProgrammaticLaunchCompletion();
#endif
}
```
**EN:** This section defines the surrounding logic, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了相关逻辑等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 191-214: Kernel implementation
```cpp
template <
    bool save_kv_cache,
    bool interleave,
    uint32_t head_dim,
    uint32_t vec_size,
    uint32_t bdx,
    typename DType,
    typename IdType>
__global__ void BatchQKApplyRotaryPosIdsCosSinCacheEnhancedKernel(
    DType* q,
    DType* k,
    DType* v,
    DType* q_rope,
    DType* k_rope,
    DType* k_buffer,
    DType* v_buffer,
    float* __restrict__ cos_sin_cache,
    IdType* __restrict__ pos_ids,
    uint32_t nnz,
    uint32_t num_qo_heads,
    uint32_t num_kv_heads,
    uint32_t rotary_dim,
    size_t q_stride_n,
    size_t q_stride_h,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 215-240: Device helpers and synchronization
```cpp
    size_t k_stride_n,
    size_t k_stride_h,
    size_t v_stride_n,
    size_t v_stride_h,
    size_t q_rope_stride_n,
    size_t q_rope_stride_h,
    size_t k_rope_stride_n,
    size_t k_rope_stride_h,
    size_t k_buffer_stride_n,
    size_t k_buffer_stride_h,
    size_t v_buffer_stride_n,
    size_t v_buffer_stride_h,
    IdType* __restrict__ kv_cache_loc) {
  uint32_t bx = blockIdx.x, tx = threadIdx.x, ty = threadIdx.y;
  const uint32_t bdy = blockDim.y;

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900))
  cudaGridDependencySynchronize();
#endif

  vec_t<float, vec_size> cos, sin;
  if (bx * bdy + ty < nnz) {
    const uint32_t idx = bx * bdy + ty;
    const IdType pos = pos_ids[idx];
    const int half_rotary_dim = rotary_dim / 2;
```
**EN:** This section implements `bdy`, `idx`, `pos`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`bdy`、`idx`、`pos`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 241-269: Templates, aliases, and constants
```cpp
    // 1. if interleave:
    //  - cos = cos_sin_cache[pos_id][tx * vec_size // 2]
    //  - sin = cos_sin_cache[pos_id][(rot_dim // 2) + tx * vec_size // 2]
    // 2. if not interleave
    //  - cos = cos_cache[pos_id][(tx * vec_size) % (rot_dim // 2)]
    //  - sin = sin_cache[pos_id][(rot_dim // 2) + (tx * vec_size) % (rot_dim // 2)]
    if (tx * vec_size < rotary_dim) {
      int sin_offset = rotary_dim / 2;
      int vec_idx;
      if constexpr (interleave) {
        vec_idx = (tx * vec_size) / 2;  // Force integer division
      } else {
        vec_idx = (tx * vec_size) % half_rotary_dim;  // Use half_rotary_dim
      }
      cos.load(cos_sin_cache + (pos * rotary_dim) + vec_idx);
      sin.load(cos_sin_cache + (pos * rotary_dim) + (sin_offset + vec_idx));
    }

    // not to unroll the loop, because num head might be large and might lead to worse performance
#pragma unroll 1
    for (uint32_t qo_head_idx = 0; qo_head_idx < num_qo_heads; ++qo_head_idx) {
      DType* q_ptr = q + get_elem_offset_impl(idx, qo_head_idx, 0, q_stride_n, q_stride_h);
      DType* q_rope_ptr = q_rope + get_elem_offset_impl(idx, qo_head_idx, 0, q_rope_stride_n, q_rope_stride_h);
      vec_t<float, vec_size> q_vec;
      if constexpr (interleave) {
        q_vec = vec_apply_llama_rope_cos_sin_interleave_reuse_half<vec_size, bdx>(q_ptr, cos, sin, rotary_dim);
      } else {
        q_vec = vec_apply_llama_rope_cos_sin<vec_size, bdx>(q_ptr, cos, sin, rotary_dim);
      }
```
**EN:** This section defines `load`, `get_elem_offset_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`load`、`get_elem_offset_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 270-293: Templates, aliases, and constants
```cpp
      q_vec.cast_store(q_rope_ptr + tx * vec_size);
    }

#pragma unroll 1
    for (uint32_t kv_head_idx = 0; kv_head_idx < num_kv_heads; ++kv_head_idx) {
      DType* k_ptr = k + get_elem_offset_impl(idx, kv_head_idx, 0, k_stride_n, k_stride_h);

      DType* k_rope_ptr = k_rope + get_elem_offset_impl(idx, kv_head_idx, 0, k_rope_stride_n, k_rope_stride_h);

      vec_t<float, vec_size> v_vec;
      IdType kv_cache_offset;
      if constexpr (save_kv_cache) {
        kv_buffer_saver::prepare<DType, IdType, vec_size>(
            v_vec, kv_cache_offset, v, kv_cache_loc, idx, tx, kv_head_idx, v_stride_n, v_stride_h);
      }

      vec_t<float, vec_size> k_vec;
      if constexpr (interleave) {
        k_vec = vec_apply_llama_rope_cos_sin_interleave_reuse_half<vec_size, bdx>(k_ptr, cos, sin, rotary_dim);
      } else {
        k_vec = vec_apply_llama_rope_cos_sin<vec_size, bdx>(k_ptr, cos, sin, rotary_dim);
      }
      k_vec.cast_store(k_rope_ptr + tx * vec_size);
```
**EN:** This section defines `cast_store`, `get_elem_offset_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_store`、`get_elem_offset_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 294-316: Templates, aliases, and constants
```cpp
      if constexpr (save_kv_cache) {
        kv_buffer_saver::save<DType, IdType, vec_size>(
            kv_cache_offset,
            k_vec,
            v_vec,
            k_buffer,
            v_buffer,
            idx,
            tx,
            kv_head_idx,
            k_buffer_stride_n,
            k_buffer_stride_h,
            v_buffer_stride_n,
            v_buffer_stride_h);
      }
    }
  }

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 900))
  cudaTriggerProgrammaticLaunchCompletion();
#endif
}
```
**EN:** This section defines the surrounding logic, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了相关逻辑等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 317-340: Templates, aliases, and constants
```cpp
#define DISPATCH_SAVE_KV_CACHE(save_kv_cache, SAVE_KV_CACHE, ...) \
  if (save_kv_cache) {                                            \
    const bool SAVE_KV_CACHE = true;                              \
    __VA_ARGS__                                                   \
  } else {                                                        \
    const bool SAVE_KV_CACHE = false;                             \
    __VA_ARGS__                                                   \
  }

template <typename DType, typename IdType>
cudaError_t BatchQKApplyRotaryPosIdsCosSinCacheEnhanced(
    DType* q,
    DType* k,
    DType* v,
    DType* q_rope,
    DType* k_rope,
    DType* k_buffer,
    DType* v_buffer,
    float* cos_sin_cache,
    IdType* pos_ids,
    uint32_t nnz,
    uint32_t num_qo_heads,
    uint32_t num_kv_heads,
    uint32_t rotary_dim,
```
**EN:** This section defines `DISPATCH_SAVE_KV_CACHE`, `SAVE_KV_CACHE`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`DISPATCH_SAVE_KV_CACHE`、`SAVE_KV_CACHE`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 341-365: Runtime integration and dispatch
```cpp
    uint32_t head_dim,
    size_t q_stride_n,
    size_t q_stride_h,
    size_t k_stride_n,
    size_t k_stride_h,
    size_t v_stride_n,
    size_t v_stride_h,
    size_t q_rope_stride_n,
    size_t q_rope_stride_h,
    size_t k_rope_stride_n,
    size_t k_rope_stride_h,
    size_t k_buffer_stride_n,
    size_t k_buffer_stride_h,
    size_t v_buffer_stride_n,
    size_t v_buffer_stride_h,
    IdType* kv_cache_loc,
    bool interleave,
    bool save_kv_cache,
    bool enable_pdl,
    cudaStream_t stream = nullptr) {
  int dev_id = 0;
  int num_sms = 0;
  FLASHINFER_CUDA_CALL(cudaGetDevice(&dev_id));
  FLASHINFER_CUDA_CALL(cudaDeviceGetAttribute(&num_sms, cudaDevAttrMultiProcessorCount, dev_id));
```
**EN:** This section uses `FLASHINFER_CUDA_CALL` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`FLASHINFER_CUDA_CALL`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 366-389: Kernel implementation
```cpp
#define LAUNCH_KERNEL_RAW(kernel_name)                                \
  do {                                                                \
    cudaLaunchConfig_t config = {};                                   \
    config.gridDim = nblks;                                           \
    config.blockDim = nthrs;                                          \
    config.dynamicSmemBytes = 0;                                      \
    config.stream = stream;                                           \
    cudaLaunchAttribute attrs[1] = {};                                \
    attrs[0].id = cudaLaunchAttributeProgrammaticStreamSerialization; \
    attrs[0].val.programmaticStreamSerializationAllowed = enable_pdl; \
    config.numAttrs = 1;                                              \
    config.attrs = attrs;                                             \
                                                                      \
    FLASHINFER_CUDA_CALL(cudaLaunchKernelEx(                          \
        &config,                                                      \
        kernel_name,                                                  \
        q,                                                            \
        k,                                                            \
        v,                                                            \
        q_rope,                                                       \
        k_rope,                                                       \
        k_buffer,                                                     \
        v_buffer,                                                     \
        cos_sin_cache,                                                \
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 390-411: Control flow and branching
```cpp
        pos_ids,                                                      \
        nnz,                                                          \
        num_qo_heads,                                                 \
        num_kv_heads,                                                 \
        rotary_dim,                                                   \
        q_stride_n,                                                   \
        q_stride_h,                                                   \
        k_stride_n,                                                   \
        k_stride_h,                                                   \
        v_stride_n,                                                   \
        v_stride_h,                                                   \
        q_rope_stride_n,                                              \
        q_rope_stride_h,                                              \
        k_rope_stride_n,                                              \
        k_rope_stride_h,                                              \
        k_buffer_stride_n,                                            \
        k_buffer_stride_h,                                            \
        v_buffer_stride_n,                                            \
        v_buffer_stride_h,                                            \
        kv_cache_loc));                                               \
  } while (0)
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 412-434: Templates, aliases, and constants
```cpp
  DISPATCH_SAVE_KV_CACHE(save_kv_cache, SAVE_KV_CACHE, {
    DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
      DISPATCH_HEAD_DIM(head_dim, HEAD_DIM, {
        // operate on 16 Bytes at a time
        constexpr uint32_t vec_size = std::max(16 / sizeof(DType), HEAD_DIM / 32);
        // how many threads needed per head_dim
        constexpr uint32_t bdx = HEAD_DIM / vec_size;
        // how many threads needed per block
        uint32_t num_threads = std::max(128U, bdx);
        // how many tokens can we process in a block
        uint32_t bdy = num_threads / bdx;
        // how many blocks needed to process all tokens
        uint32_t nblks_x = (nnz + bdy - 1) / bdy;

        auto kernel_0 = BatchQKApplyRotaryPosIdsCosSinCacheEnhancedKernel<
            SAVE_KV_CACHE,
            INTERLEAVE,
            HEAD_DIM,
            vec_size,
            bdx,
            DType,
            IdType>;
```
**EN:** This section defines `max`, `vec_size`, `bdx`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`max`、`vec_size`、`bdx`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 435-456: Control flow and branching
```cpp
        int num_blocks_per_sm_0 = 0;
        FLASHINFER_CUDA_CALL(cudaOccupancyMaxActiveBlocksPerMultiprocessor(
            &num_blocks_per_sm_0, kernel_0, num_threads, /*smem_size=*/0));
        uint32_t num_ctas_0 = num_blocks_per_sm_0 * num_sms;

        if ((nnz + bdy - 1) / bdy >= num_ctas_0) {
          dim3 nblks(nblks_x);
          dim3 nthrs(bdx, bdy);
          LAUNCH_KERNEL_RAW(kernel_0);
        } else {
          dim3 nblks(nblks_x, num_qo_heads + num_kv_heads);
          dim3 nthrs(bdx, bdy);
          auto kernel_1 = BatchQKApplyRotaryPosIdsCosSinCacheEnhancedHeadParallelismKernel<
              SAVE_KV_CACHE,
              INTERLEAVE,
              HEAD_DIM,
              vec_size,
              bdx,
              DType,
              IdType>;
          LAUNCH_KERNEL_RAW(kernel_1);
        }
```
**EN:** This section drives `FLASHINFER_CUDA_CALL`, `nblks`, `nthrs` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`FLASHINFER_CUDA_CALL`、`nblks`、`nthrs`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 457-466: Local implementation details
```cpp
      });
    });
  });
#undef LAUNCH_KERNEL_RAW

  return cudaSuccess;
}

}  // namespace flashinfer
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 467-467: Local implementation details
```cpp
#endif  // SGL_POS_ENC_CUH_
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `flashinfer/pos_enc.cuh`
- **Path context / 路径上下文**: elementwise / pos_enc.cuh
