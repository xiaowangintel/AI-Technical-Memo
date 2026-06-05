# dsv3_router_gemm_entry.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/dsv3_router_gemm_entry.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements DeepSeek-V3 router GEMM kernels and host-side entry logic. / 实现 DeepSeek-V3 路由 GEMM 内核及宿主端入口逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-29)
```cpp
/*
 * Adapted from SGLang's sgl-kernel implementation, which was adapted from
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
// ...

#include "core/registration.h"
#include "dsv3_router_gemm_utils.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Compile-time setup: static constexpr int DEFAULT_NUM_EXPERTS = 256; (lines 31-31)
```cpp
static constexpr int DEFAULT_NUM_EXPERTS = 256;
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

### Compile-time setup: static constexpr int KIMI_K2_NUM_EXPERTS = 384; (lines 32-32)
```cpp
static constexpr int KIMI_K2_NUM_EXPERTS = 384;
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

### Compile-time setup: static constexpr int DEFAULT_HIDDEN_DIM = 7168; (lines 33-33)
```cpp
static constexpr int DEFAULT_HIDDEN_DIM = 7168;
```
**EN:** This block defines compile-time constants or helper macros that shape tiling, vector width, or backend-specific behavior.
**CN:** 该代码块定义了编译期常量或辅助宏，用来决定分块方式、向量宽度或后端特定行为。

### Struct: LoopUnroller (lines 44-72)
```cpp
struct LoopUnroller {
  static void unroll_float_output(int num_tokens, float* output,
                                  __nv_bfloat16 const* input,
                                  __nv_bfloat16 const* weights,
                                  cudaStream_t stream) {
    if (num_tokens == kBegin) {
      invokeRouterGemmFloatOutput<__nv_bfloat16, kBegin, kNumExperts,
                                  kHiddenDim>(output, input, weights, stream);
    } else {
      LoopUnroller<kBegin + 1, kEnd, kNumExperts,
                   kHiddenDim>::unroll_float_output(num_tokens, output, input,
                                                    weights, stream);
    }
  }
// ...
    }
  }
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Struct: LoopUnroller (lines 75-99)
```cpp
struct LoopUnroller<kEnd, kEnd, kNumExperts, kHiddenDim> {
  static void unroll_float_output(int num_tokens, float* output,
                                  __nv_bfloat16 const* input,
                                  __nv_bfloat16 const* weights,
                                  cudaStream_t stream) {
    if (num_tokens == kEnd) {
      invokeRouterGemmFloatOutput<__nv_bfloat16, kEnd, kNumExperts, kHiddenDim>(
          output, input, weights, stream);
    } else {
      throw std::invalid_argument("Invalid num_tokens, only supports 1 to 16");
    }
  }

  static void unroll_bf16_output(int num_tokens, __nv_bfloat16* output,
// ...
    }
  }
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Function / Kernel: dsv3_router_gemm (lines 101-165)
```cpp
void dsv3_router_gemm(at::Tensor& output,       // [num_tokens, num_experts]
                      const at::Tensor& mat_a,  // [num_tokens, hidden_dim]
                      const at::Tensor& mat_b   // [num_experts, hidden_dim]
) {
  TORCH_CHECK(output.dim() == 2 && mat_a.dim() == 2 && mat_b.dim() == 2);

  const int num_tokens = mat_a.size(0);
  const int num_experts = mat_b.size(0);
  const int hidden_dim = mat_a.size(1);

  TORCH_CHECK(mat_a.size(1) == mat_b.size(1),
              "mat_a and mat_b must have the same hidden_dim");
  TORCH_CHECK(hidden_dim == DEFAULT_HIDDEN_DIM,
              "Expected hidden_dim=", DEFAULT_HIDDEN_DIM,
// ...
    }
  }
}
```
**EN:** This host-side function validates inputs and dispatches to the best specialization or launch configuration for the current runtime shape.
**CN:** 这个宿主端函数负责校验输入，并根据当前运行时形状选择最佳专用实现或启动配置。

### Registration: TORCH_LIBRARY (lines 167-169)
```cpp
TORCH_LIBRARY_IMPL_EXPAND(TORCH_EXTENSION_NAME, CUDA, m) {
  m.impl("dsv3_router_gemm", &dsv3_router_gemm);
}
```
**EN:** This block exposes the low-level implementation as a PyTorch extension entrypoint, so Python code can invoke the kernel through a stable schema.
**CN:** 该代码块把底层实现注册为 PyTorch 扩展入口，使 Python 侧能够通过稳定的 schema 调用这些内核。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Tiled matrix multiplication / 分块矩阵乘法
- CUDA programming model / CUDA 编程模型
- PyTorch custom operator registration / PyTorch 自定义算子注册
- CUDA kernel launch and thread mapping / CUDA 内核启动与线程映射
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `core/registration.h`
- **External libraries / 外部库**: PyTorch / ATen, CUDA runtime / CUDA headers, C++ standard library or platform support
- **Runtime coupling / 运行时耦合**: Registered through the PyTorch extension mechanism / 通过 PyTorch 扩展机制注册; Launches GPU kernels and relies on CUDA/HIP execution semantics / 启动 GPU 内核并依赖 CUDA/HIP 执行语义; Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
