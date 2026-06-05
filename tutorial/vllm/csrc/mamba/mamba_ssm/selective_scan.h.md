# selective_scan.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/mamba/mamba_ssm/selective_scan.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Declares parameter structs and device helpers for selective-scan kernels. / 声明 selective-scan 内核所需的参数结构与设备端辅助函数。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-15)
```cpp
/******************************************************************************
 * Copyright (c) 2023, Tri Dao.
 ******************************************************************************/
// clang-format off
// adapted from https://github.com/state-spaces/mamba/blob/main/csrc/selective_scan/selective_scan.h

#pragma once

#ifndef USE_ROCM
    #include <cuda_bf16.h>
#else
    #include <hip/hip_bf16.h>
#endif
#include <cuda_fp16.h>
////////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Struct: SSMParamsBase (lines 17-77)
```cpp
struct SSMParamsBase {
    using index_t = size_t;

    int batch, dim, seqlen, dstate, n_groups;
    int dim_ngroups_ratio;
    bool is_variable_B;
    bool is_variable_C;
    int64_t null_block_id;

    bool delta_softplus;
    bool cache_enabled;
    int block_size;

    index_t A_d_stride;
// ...
    void *__restrict__ cu_chunk_seqlen_ptr;      // (nchunks+1,) - cumulative chunk token offsets
    void *__restrict__ last_chunk_indices_ptr;   // (batch,) - index of last chunk per sequence
};
```
**EN:** This type centralizes parameters and compile-time traits, making the later kernel code simpler and safer to specialize.
**CN:** 该类型集中保存参数和编译期 trait，使后续内核更容易进行专用化并保持实现安全。

### Function / Kernel: custom_max (lines 84-87)
```cpp
    constexpr size_t custom_max(std::initializer_list<size_t> ilist) 
    {
        return std::max(ilist);
    }
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: operator (lines 206-210)
```cpp
    __device__ scan_t operator()(scan_t block_aggregate) {
        scan_t old_prefix = running_prefix;
        running_prefix = SSMScanOp<scalar_t>()(running_prefix, block_aggregate);
        return old_prefix;
    }
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: Ktraits::input_t (lines 215-234)
```cpp
template<typename Ktraits>
inline __device__ void load_input(typename Ktraits::input_t *u,
                                  typename Ktraits::input_t (&u_vals)[Ktraits::kNItems],
                                  typename Ktraits::BlockLoadT::TempStorage &smem_load,
                                  int seqlen) {
    if constexpr (Ktraits::kIsEvenLen && !Ktraits::kVarlen) {
        auto& smem_load_vec = reinterpret_cast<typename Ktraits::BlockLoadVecT::TempStorage&>(smem_load);
        using vec_t = typename Ktraits::vec_t;
        typename Ktraits::BlockLoadVecT(smem_load_vec).Load(
            reinterpret_cast<vec_t*>(u),
            reinterpret_cast<vec_t(&)[Ktraits::kNLoads]>(u_vals)
            #ifdef USE_ROCM
                , Ktraits::kNThreads * Ktraits::kNLoads
            #endif
// ...
        typename Ktraits::BlockLoadT(smem_load).Load(u, u_vals, seqlen, 0.f);
    }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: Ktraits::weight_t (lines 237-257)
```cpp
template<typename Ktraits>
inline __device__ void load_weight(typename Ktraits::input_t *Bvar,
                                   typename Ktraits::weight_t (&B_vals)[Ktraits::kNItems],
                                   typename Ktraits::BlockLoadWeightT::TempStorage &smem_load_weight,
                                   int seqlen) {
    constexpr int kNItems = Ktraits::kNItems;
    typename Ktraits::input_t B_vals_load[kNItems];
    if constexpr (Ktraits::kIsEvenLen && !Ktraits::kVarlen) {
        auto& smem_load_weight_vec = reinterpret_cast<typename Ktraits::BlockLoadWeightVecT::TempStorage&>(smem_load_weight);
        using vec_t = typename Ktraits::vec_t;
        typename Ktraits::BlockLoadWeightVecT(smem_load_weight_vec).Load(
            reinterpret_cast<vec_t*>(Bvar),
            reinterpret_cast<vec_t(&)[Ktraits::kNLoads]>(B_vals_load)
      );
// ...
    // for (int i = 0; i < kNItems; ++i) { B_vals[i] = B_vals_load[i]; }
    Converter<typename Ktraits::input_t, kNItems>::to_float(B_vals_load, B_vals);
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: float (lines 259-277)
```cpp
template<typename Ktraits>
inline __device__ void store_output(typename Ktraits::input_t *out,
                                    const float (&out_vals)[Ktraits::kNItems],
                                    typename Ktraits::BlockStoreT::TempStorage &smem_store,
                                    int seqlen) {
    typename Ktraits::input_t write_vals[Ktraits::kNItems];
    #pragma unroll
    for (int i = 0; i < Ktraits::kNItems; ++i) { write_vals[i] = out_vals[i]; }
    if constexpr (Ktraits::kIsEvenLen && !Ktraits::kVarlen) {
        auto& smem_store_vec = reinterpret_cast<typename Ktraits::BlockStoreVecT::TempStorage&>(smem_store);
        using vec_t = typename Ktraits::vec_t;
        typename Ktraits::BlockStoreVecT(smem_store_vec).Store(
            reinterpret_cast<vec_t*>(out),
            reinterpret_cast<vec_t(&)[Ktraits::kNLoads]>(write_vals)
// ...
        typename Ktraits::BlockStoreT(smem_store).Store(out, write_vals, seqlen);
    }
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

## Key Concepts / 关键概念
- Softplus-based gating / 基于 Softplus 的门控
- State-space selective scan / 状态空间 selective scan
- Low-bit quantization / 低比特量化
- Integer kernel specialization / 整数内核专用化
- SIMD vectorization / SIMD 向量化
- CUDA programming model / CUDA 编程模型
- ROCm/HIP backend / ROCm/HIP 后端
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: CUDA runtime / CUDA headers, ROCm / HIP runtime
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
