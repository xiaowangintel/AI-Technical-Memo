# selective_scan_fwd.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/mamba/mamba_ssm/selective_scan_fwd.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the CUDA forward selective-scan kernel used by Mamba state-space models. / 实现 Mamba 状态空间模型使用的 CUDA 前向 selective-scan 内核。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-21)
```cpp
// clang-format off
// adapted from https://github.com/state-spaces/mamba/blob/main/csrc/selective_scan/selective_scan_fwd_kernel.cuh
#include <torch/all.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include "selective_scan.h"

#include <c10/util/BFloat16.h>
#include <c10/util/Half.h>
#ifdef USE_ROCM
    #include <c10/hip/HIPException.h>  // For C10_HIP_CHECK and C10_HIP_KERNEL_LAUNCH_CHECK
#else
    #include <c10/cuda/CUDAException.h>  // For C10_CUDA_CHECK and C10_CUDA_KERNEL_LAUNCH_CHECK
#endif
// ...
    #include <cub/block/block_scan.cuh>
#else
    #include <hipcub/hipcub.hpp>
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Struct: Selective_Scan_fwd_kernel_traits (lines 31-75)
```cpp
struct Selective_Scan_fwd_kernel_traits {
    static_assert(kNItems_ % 4 == 0);
    using input_t = input_t_;
    using weight_t = weight_t_;
    using state_t = state_t_;
    static constexpr int kNThreads = kNThreads_;
    // Setting MinBlocksPerMP to be 3 (instead of 2) for 128 threads improves occupancy.
    static constexpr int kMinBlocks = kNThreads < 128 ? 5 : 3;
    static constexpr int kNItems = kNItems_;
    static constexpr int kNRows = kNRows_;
    static constexpr int kNBytes = sizeof(input_t);
    static_assert(kNBytes == 2 || kNBytes == 4);
    static constexpr int kNElts = kNBytes == 4 ? 4 : constexpr_min(8, kNItems);
    static_assert(kNItems % kNElts == 0);
// ...
                                                 sizeof(typename BlockStoreVecT::TempStorage)});
    static constexpr int kSmemSize = kSmemIOSize + sizeof(typename BlockScanT::TempStorage);
};
```
**EN:** This type centralizes parameters and compile-time traits, making the later kernel code simpler and safer to specialize.
**CN:** 该类型集中保存参数和编译期 trait，使后续内核更容易进行专用化并保持实现安全。

### Function / Kernel: selective_scan_fwd_kernel (lines 77-400)
```cpp
template<typename Ktraits>
__global__ __launch_bounds__(Ktraits::kNThreads, Ktraits::kMinBlocks)
void selective_scan_fwd_kernel(SSMParamsBase params) {
    constexpr bool kIsVariableB = Ktraits::kIsVariableB;
    constexpr bool kIsVariableC = Ktraits::kIsVariableC;
    constexpr bool kHasZ = Ktraits::kHasZ;
    constexpr bool kVarlen = Ktraits::kVarlen;
    constexpr int kNItems = Ktraits::kNItems;
    constexpr int kNRows = Ktraits::kNRows;
    constexpr bool kDirectIO = Ktraits::kDirectIO;
    using input_t = typename Ktraits::input_t;
    using weight_t = typename Ktraits::weight_t;
    using scan_t = typename Ktraits::scan_t;

// ...
        current_position += chunk_tokens;
    }
}
```
**EN:** This kernel performs the forward selective-scan recurrence, updating state-space variables along the sequence in parallel-friendly chunks.
**CN:** 该内核执行前向 selective-scan 递推，沿序列以适合并行的分块方式更新状态空间变量。

### Function / Kernel: selective_scan_fwd_launch (lines 402-431)
```cpp
template<int kNThreads, int kNItems, typename input_t, typename weight_t, typename state_t>
void selective_scan_fwd_launch(SSMParamsBase &params, cudaStream_t stream) {
    // Only kNRows == 1 is tested for now, which ofc doesn't differ from previously when we had each block
    // processing 1 row.
    constexpr int kNRows = 1;
    // kIsVariableB, kIsVariableC and kHasZ are all set to True to reduce binary size
    constexpr bool kIsVariableB = true;
    constexpr bool kIsVariableC = true;
    BOOL_SWITCH(params.seqlen % (kNThreads * kNItems) == 0, kIsEvenLen, [&] {
        BOOL_SWITCH(params.z_ptr != nullptr , kHasZ, [&] {
            BOOL_SWITCH(params.query_start_loc_ptr != nullptr , kVarlen, [&] {
                using Ktraits = Selective_Scan_fwd_kernel_traits<kNThreads, kNItems, kNRows, kIsEvenLen, kIsVariableB, kIsVariableC, kHasZ,  kVarlen, input_t, weight_t, state_t>;
                constexpr int kSmemSize = Ktraits::kSmemSize + kNRows * MAX_DSTATE * sizeof(typename Ktraits::scan_t);
                dim3 grid(params.batch, params.dim / kNRows);
// ...
        });
    });
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: selective_scan_fwd_cuda (lines 433-463)
```cpp
template<typename input_t, typename weight_t, typename state_t>
void selective_scan_fwd_cuda(SSMParamsBase &params, cudaStream_t stream) {

    #ifndef USE_ROCM
        if (params.cache_enabled && params.block_size == 1024) {
            selective_scan_fwd_launch<64, 16, input_t, weight_t, state_t>(params, stream);
        } else if (params.seqlen <= 128) {
            selective_scan_fwd_launch<32, 4, input_t, weight_t, state_t>(params, stream);
        } else if (params.seqlen <= 256) {
            selective_scan_fwd_launch<32, 8, input_t, weight_t, state_t>(params, stream);
        } else if (params.seqlen <= 512) {
            selective_scan_fwd_launch<32, 16, input_t, weight_t, state_t>(params, stream);
        } else if (params.seqlen <= 1024) {
            selective_scan_fwd_launch<64, 16, input_t, weight_t, state_t>(params, stream);
// ...
        }
    #endif
}
```
**EN:** This function provides a reusable piece of the file’s main compute pipeline, usually handling validation, indexing, or a fused math step.
**CN:** 该函数是本文件主计算流水线中的一个可复用环节，通常负责校验、索引计算或融合数学步骤。

### Function / Kernel: scalar_type (lines 828-830)
```cpp
    DISPATCH_WTYPE_ITYPE_FLOAT_AND_HALF_AND_BF16(u.scalar_type(), ssm_states.scalar_type(), "selective_scan_fwd", [&] {
        selective_scan_fwd_cuda<input_t, weight_t, state_t>(params, stream);
    });
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Softplus-based gating / 基于 Softplus 的门控
- State-space selective scan / 状态空间 selective scan
- SIMD vectorization / SIMD 向量化
- CUDA programming model / CUDA 编程模型
- ROCm/HIP backend / ROCm/HIP 后端
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / ATen, PyTorch / c10, C++ standard library or platform support, CUB block primitives
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
