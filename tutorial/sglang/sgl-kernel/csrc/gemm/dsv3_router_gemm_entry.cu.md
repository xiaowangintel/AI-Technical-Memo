# dsv3_router_gemm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/dsv3_router_gemm_entry.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Headers and compile-time setup
```cpp
/*
 * Adapted from
 * https://github.com/NVIDIA/TensorRT-LLM/blob/main/cpp/tensorrt_llm/kernels/dsv3MinLatencyKernels/dsv3RouterGemm.cu
 * https://github.com/NVIDIA/TensorRT-LLM/blob/main/cpp/tensorrt_llm/thop/dsv3RouterGemmOp.cpp
 *
 * Copyright (c) 2019-2023, NVIDIA CORPORATION.  All rights reserved.
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
 */

#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 24-47: Types and data layout
```cpp
#include "cuda_bf16.h"
#include "cuda_runtime.h"
#include "utils.h"

static constexpr int DEFAULT_NUM_EXPERTS = 256;
static constexpr int KIMI_K2_NUM_EXPERTS = 384;
static constexpr int DEFAULT_HIDDEN_DIM = 7168;

template <typename T, int kNumTokens, int kNumExperts, int kHiddenDim>
void invokeRouterGemmFloatOutput(float* output, T const* mat_a, T const* mat_b, cudaStream_t stream);

template <typename T, int kNumTokens, int kNumExperts, int kHiddenDim>
void invokeRouterGemmBf16Output(__nv_bfloat16* output, T const* mat_a, T const* mat_b, cudaStream_t stream);

template <int kBegin, int kEnd, int kNumExperts, int kHiddenDim>
struct LoopUnroller {
  static void unroll_float_output(
      int num_tokens, float* output, __nv_bfloat16 const* input, __nv_bfloat16 const* weights, cudaStream_t stream) {
    if (num_tokens == kBegin) {
      invokeRouterGemmFloatOutput<__nv_bfloat16, kBegin, kNumExperts, kHiddenDim>(output, input, weights, stream);
    } else {
      LoopUnroller<kBegin + 1, kEnd, kNumExperts, kHiddenDim>::unroll_float_output(
          num_tokens, output, input, weights, stream);
    }
```
**EN:** This section defines `LoopUnroller`, `unroll_float_output`, `invokeRouterGemmFloatOutput`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`LoopUnroller`、`unroll_float_output`、`invokeRouterGemmFloatOutput`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 48-73: Types and data layout
```cpp
  }

  static void unroll_bf16_output(
      int num_tokens,
      __nv_bfloat16* output,
      __nv_bfloat16 const* input,
      __nv_bfloat16 const* weights,
      cudaStream_t stream) {
    if (num_tokens == kBegin) {
      invokeRouterGemmBf16Output<__nv_bfloat16, kBegin, kNumExperts, kHiddenDim>(output, input, weights, stream);
    } else {
      LoopUnroller<kBegin + 1, kEnd, kNumExperts, kHiddenDim>::unroll_bf16_output(
          num_tokens, output, input, weights, stream);
    }
  }
};

template <int kEnd, int kNumExperts, int kHiddenDim>
struct LoopUnroller<kEnd, kEnd, kNumExperts, kHiddenDim> {
  static void unroll_float_output(
      int num_tokens, float* output, __nv_bfloat16 const* input, __nv_bfloat16 const* weights, cudaStream_t stream) {
    if (num_tokens == kEnd) {
      invokeRouterGemmFloatOutput<__nv_bfloat16, kEnd, kNumExperts, kHiddenDim>(output, input, weights, stream);
    } else {
      throw std::invalid_argument("Invalid num_tokens, only supports 1 to 16");
    }
```
**EN:** This section defines `LoopUnroller`, `unroll_bf16_output`, `unroll_float_output`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`LoopUnroller`、`unroll_bf16_output`、`unroll_float_output`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 74-96: Runtime integration and dispatch
```cpp
  }

  static void unroll_bf16_output(
      int num_tokens,
      __nv_bfloat16* output,
      __nv_bfloat16 const* input,
      __nv_bfloat16 const* weights,
      cudaStream_t stream) {
    if (num_tokens == kEnd) {
      invokeRouterGemmBf16Output<__nv_bfloat16, kEnd, kNumExperts, kHiddenDim>(output, input, weights, stream);
    } else {
      throw std::invalid_argument("Invalid num_tokens, only supports 1 to 16");
    }
  }
};

void dsv3_router_gemm(
    torch::Tensor& output,       // [num_tokens, num_experts]
    const torch::Tensor& mat_a,  // [num_tokens, hidden_dim]
    const torch::Tensor& mat_b   // [num_experts, hidden_dim]
) {
  TORCH_CHECK(output.dim() == 2 && mat_a.dim() == 2 && mat_b.dim() == 2);
```
**EN:** This section uses `unroll_bf16_output`, `dsv3_router_gemm`, `invalid_argument` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`unroll_bf16_output`、`dsv3_router_gemm`、`invalid_argument`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 97-122: Runtime integration and dispatch
```cpp
  const int num_tokens = mat_a.size(0);
  const int num_experts = mat_b.size(0);
  const int hidden_dim = mat_a.size(1);

  TORCH_CHECK(mat_a.size(1) == mat_b.size(1), "mat_a and mat_b must have the same hidden_dim");
  TORCH_CHECK(
      hidden_dim == DEFAULT_HIDDEN_DIM,
      "Expected hidden_dim=",
      DEFAULT_HIDDEN_DIM,
      ", but got hidden_dim=",
      hidden_dim);
  TORCH_CHECK(
      num_experts == DEFAULT_NUM_EXPERTS || num_experts == KIMI_K2_NUM_EXPERTS,
      "Expected num_experts=",
      DEFAULT_NUM_EXPERTS,
      " or num_experts=",
      KIMI_K2_NUM_EXPERTS,
      ", but got num_experts=",
      num_experts);
  TORCH_CHECK(
      num_tokens >= 1 && num_tokens <= 16, "currently num_tokens must be less than or equal to 16 for router_gemm");
  TORCH_CHECK(mat_a.dtype() == torch::kBFloat16, "mat_a must be bf16");
  TORCH_CHECK(mat_b.dtype() == torch::kBFloat16, "mat_b must be bf16");
  TORCH_CHECK(
      output.dtype() == torch::kFloat32 || output.dtype() == torch::kBFloat16, "output must be float32 or bf16");
```
**EN:** This section uses `TORCH_CHECK`, `num_tokens`, `num_experts` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`num_tokens`、`num_experts`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 123-147: Runtime integration and dispatch
```cpp
  auto const sm = getSMVersion();
#ifndef USE_MUSA
  TORCH_CHECK(sm >= 90, "required CUDA ARCH >= SM_90");
#else
  TORCH_CHECK(sm >= 22, "required MUSA ARCH >= MP_22");
#endif

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  if (output.dtype() == torch::kFloat32) {
    if (num_experts == DEFAULT_NUM_EXPERTS) {
      LoopUnroller<1, 16, DEFAULT_NUM_EXPERTS, DEFAULT_HIDDEN_DIM>::unroll_float_output(
          num_tokens,
          reinterpret_cast<float*>(output.mutable_data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_a.data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_b.data_ptr()),
          stream);
    } else if (num_experts == KIMI_K2_NUM_EXPERTS) {
      LoopUnroller<1, 16, KIMI_K2_NUM_EXPERTS, DEFAULT_HIDDEN_DIM>::unroll_float_output(
          num_tokens,
          reinterpret_cast<float*>(output.mutable_data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_a.data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_b.data_ptr()),
          stream);
    }
```
**EN:** This section uses `getSMVersion`, `TORCH_CHECK`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`getSMVersion`、`TORCH_CHECK`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 148-165: Runtime integration and dispatch
```cpp
  } else if (output.dtype() == torch::kBFloat16) {
    if (num_experts == DEFAULT_NUM_EXPERTS) {
      LoopUnroller<1, 16, DEFAULT_NUM_EXPERTS, DEFAULT_HIDDEN_DIM>::unroll_bf16_output(
          num_tokens,
          reinterpret_cast<__nv_bfloat16*>(output.mutable_data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_a.data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_b.data_ptr()),
          stream);
    } else if (num_experts == KIMI_K2_NUM_EXPERTS) {
      LoopUnroller<1, 16, KIMI_K2_NUM_EXPERTS, DEFAULT_HIDDEN_DIM>::unroll_bf16_output(
          num_tokens,
          reinterpret_cast<__nv_bfloat16*>(output.mutable_data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_a.data_ptr()),
          reinterpret_cast<__nv_bfloat16 const*>(mat_b.data_ptr()),
          stream);
    }
  }
}
```
**EN:** This section uses `unroll_bf16_output` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`unroll_bf16_output`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **MUSA portability / MUSA 可移植性**: Contains conditional logic for the MUSA toolchain or runtime. / 包含面向 MUSA 工具链或运行时的条件分支。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cuda_bf16.h`, `cuda_runtime.h`, `utils.h`
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/cuda/CUDAContext.h`
- **Path context / 路径上下文**: gemm / dsv3_router_gemm_entry.cu
