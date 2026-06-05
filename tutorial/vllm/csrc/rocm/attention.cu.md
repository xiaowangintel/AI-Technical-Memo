# attention.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/rocm/attention.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements ROCm attention kernels and dispatch wrappers. / 实现 ROCm 注意力内核及其分派封装。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-29)
```cpp
/*
 * Copyright (c) 2024, The vLLM team.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
// ...

// ROCm 6.2 compatibility: map OCP fp8 types to FNUZ variants if OCP is absent
#if !defined(HIP_FP8_TYPE_OCP)
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: paged_attention_ll4mi_QKV_mfma16_kernel (lines 317-911)
```cpp
template <typename scalar_t, typename cache_t,
          vllm::Fp8KVCacheDataType KV_DTYPE, typename OUTT, int BLOCK_SIZE,
          int HEAD_SIZE, int NUM_THREADS, bool ALIBI_ENABLED, int GQA_RATIO, MFMAType MFMA_TYPE>
__global__
__launch_bounds__(NUM_THREADS, 5) void paged_attention_ll4mi_QKV_mfma16_kernel(
    const scalar_t* __restrict__ q,         // [num_seqs, num_heads, head_size]
    const cache_t* __restrict__ k_cache,    // [num_blocks, num_kv_heads, head_size/x, block_size, x]
    const cache_t* __restrict__ v_cache,    // [num_blocks, num_kv_heads, head_size, block_size]
    const int num_kv_heads,   
    const float scale,    
    const int* __restrict__ block_tables,   // [num_seqs, max_num_blocks_per_seq]
    const int* __restrict__ seq_lens,   // [num_seqs]
    const int* __restrict__ query_start_loc_ptr,   // [num_seqs]
    const int max_num_blocks_per_seq,
// ...
    }
  }
}
```
**EN:** This kernel is the GPU hot path for attention: it maps sequence/head work onto blocks and performs the core score/value computation.
**CN:** 该内核是注意力计算的 GPU 热路径：它将序列/头维度的工作映射到线程块，并完成核心打分与取值计算。

### Function / Kernel: paged_attention_ll4mi_QKV_mfma4_kernel (lines 917-1414)
```cpp
template <typename scalar_t, typename cache_t,
          vllm::Fp8KVCacheDataType KV_DTYPE, typename OUTT, int BLOCK_SIZE,
          int HEAD_SIZE, int NUM_THREADS, bool ALIBI_ENABLED,
          int GQA_RATIO>
__global__
__launch_bounds__(NUM_THREADS) void paged_attention_ll4mi_QKV_mfma4_kernel(
    const scalar_t* __restrict__ q,         // [num_seqs, num_heads, head_size]
    const cache_t* __restrict__ k_cache,    // [num_blocks, num_kv_heads, head_size/x, block_size, x]
    const cache_t* __restrict__ v_cache,    // [num_blocks, num_kv_heads, head_size, block_size]
    const int num_kv_heads,
    const float scale,
    const int* __restrict__ block_tables,   // [num_seqs, max_num_blocks_per_seq]
    const int* __restrict__ seq_lens,   // [num_seqs]
    const int* __restrict__ query_start_loc_ptr,   // [num_seqs]
// ...
    }
  }  // warpid == 0
}
```
**EN:** This kernel is the GPU hot path for attention: it maps sequence/head work onto blocks and performs the core score/value computation.
**CN:** 该内核是注意力计算的 GPU 热路径：它将序列/头维度的工作映射到线程块，并完成核心打分与取值计算。

### Function / Kernel: paged_attention_ll4mi_QKV_mfma16_kernel (lines 3118-3143)
```cpp
template <typename scalar_t, typename cache_t,
          vllm::Fp8KVCacheDataType KV_DTYPE, typename OUTT, int BLOCK_SIZE,
          int HEAD_SIZE, int NUM_THREADS, bool ALIBI_ENABLED,
          int GQA_RATIO, MFMAType MFMA_TYPE>
__global__
__launch_bounds__(NUM_THREADS) void paged_attention_ll4mi_QKV_mfma16_kernel(
    const scalar_t* __restrict__ q,         // [num_seqs, num_heads, head_size]
    const cache_t* __restrict__ k_cache,    // [num_blocks, num_kv_heads, head_size/x, block_size, x]
    const cache_t* __restrict__ v_cache,    // [num_blocks, num_kv_heads, head_size, block_size]
    const int num_kv_heads,
    const float scale,
    const int* __restrict__ block_tables,    // [num_seqs, max_num_blocks_per_seq]
    const int* __restrict__ seq_lens,    // [num_seqs]
    const int* __restrict__ query_start_loc_ptr,  // [num_seqs]
// ...
    int max_ctx_blocks, const float* k_scale, const float* v_scale) {
  UNREACHABLE_CODE
}
```
**EN:** This kernel is the GPU hot path for attention: it maps sequence/head work onto blocks and performs the core score/value computation.
**CN:** 该内核是注意力计算的 GPU 热路径：它将序列/头维度的工作映射到线程块，并完成核心打分与取值计算。

### Function / Kernel: paged_attention_custom_launcher (lines 3219-3373)
```cpp
template <typename T, typename KVT, vllm::Fp8KVCacheDataType KV_DTYPE,
          int BLOCK_SIZE, int HEAD_SIZE, typename OUTT, int PARTITION_SIZE_OLD,
          bool ALIBI_ENABLED, MFMAType MFMA_TYPE>
void paged_attention_custom_launcher(
    torch::Tensor& out, torch::Tensor& exp_sums, torch::Tensor& max_logits,
    torch::Tensor& tmp_out, torch::Tensor& query, torch::Tensor& key_cache,
    torch::Tensor& value_cache, const int num_kv_heads, float scale,
    torch::Tensor& block_tables, torch::Tensor& seq_lens,
    const std::optional<torch::Tensor>& query_start_loc, int max_seq_len,
    const std::optional<torch::Tensor>& alibi_slopes, torch::Tensor& k_scale,
    torch::Tensor& v_scale, const std::optional<torch::Tensor>& fp8_out_scale) {
  int num_seqs = block_tables.size(0);
  int num_heads = query.size(1);
  int head_size = query.size(2);
// ...
      break;
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Function / Kernel: paged_attention_custom_launcher_navi (lines 3375-3546)
```cpp
template <typename T, typename KVT, vllm::Fp8KVCacheDataType KV_DTYPE,
          int BLOCK_SIZE, int HEAD_SIZE, typename OUTT, int PARTITION_SIZE_OLD,
          bool ALIBI_ENABLED, MFMAType MFMA_TYPE>
void paged_attention_custom_launcher_navi(
    torch::Tensor& out, torch::Tensor& exp_sums, torch::Tensor& max_logits,
    torch::Tensor& tmp_out, torch::Tensor& query, torch::Tensor& key_cache,
    torch::Tensor& value_cache, const int num_kv_heads, float scale,
    torch::Tensor& block_tables, torch::Tensor& seq_lens,
    const std::optional<torch::Tensor>& query_start_loc, int max_seq_len,
    const std::optional<torch::Tensor>& alibi_slopes, torch::Tensor& k_scale,
    torch::Tensor& v_scale) {
  int num_seqs = block_tables.size(0);
  int num_heads = query.size(1);
  int head_size = query.size(2);
// ...
      break;
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

## Key Concepts / 关键概念
- Softmax normalization / Softmax 归一化
- Token permutation / Token 置换
- Attention computation / 注意力计算
- Tiled matrix multiplication / 分块矩阵乘法
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- SIMD vectorization / SIMD 向量化
- CUDA programming model / CUDA 编程模型

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `../cuda_compat.h`, `../attention/dtype_fp8.cuh`, `../quantization/w8a8/fp8/amd/quant_utils.cuh`
- **External libraries / 外部库**: PyTorch / ATen, PyTorch / c10, ROCm / HIP runtime, C++ standard library, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
