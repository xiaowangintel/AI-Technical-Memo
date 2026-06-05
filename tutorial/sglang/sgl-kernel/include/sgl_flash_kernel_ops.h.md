# sgl_flash_kernel_ops.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/sgl_flash_kernel_ops.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Local implementation details
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#pragma once
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 18-35: Headers and compile-time setup
```cpp
#include <ATen/ATen.h>
#include <ATen/Tensor.h>
#include <Python.h>
#include <torch/library.h>
#include <torch/torch.h>

#include <vector>

#include "sgl_kernel_torch_shim.h"

#define TORCH_LIBRARY_EXPAND(NAME, MODULE) TORCH_LIBRARY(NAME, MODULE)

#define _CONCAT(A, B) A##B
#define CONCAT(A, B) _CONCAT(A, B)

#define _STRINGIFY(A) #A
#define STRINGIFY(A) _STRINGIFY(A)
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 36-53: Types and data layout
```cpp
#define REGISTER_EXTENSION(NAME)                                                                      \
  PyMODINIT_FUNC CONCAT(PyInit_, NAME)() {                                                            \
    static struct PyModuleDef module = {PyModuleDef_HEAD_INIT, STRINGIFY(NAME), nullptr, 0, nullptr}; \
    return PyModule_Create(&module);                                                                  \
  }

/*
 * From flash-attention
 */
std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor> mha_fwd(
    at::Tensor q,  // (b, s_q, h, d) or (total_q, h, d) if there is cu_seqlens_q
    at::Tensor k,  // (b_k, s_k, h_k, d) or (total_k, h_k, d) if there is cu_seqlens_k or (num_pages, page_size,
                   // h_k, d) if there is page_table.
    at::Tensor v,  // (b_k, s_k, h_k, dv) or (total_k, h_k, dv) if there is cu_seqlens_k or (num_pages,
                   // page_size, h_k, dv) if there is page_table.
    std::optional<at::Tensor> k_new_,  // (b, s_k_new, h_k, d) or (total_k_new, h_k, d) if there is cu_seqlens_k_new
    std::optional<at::Tensor> v_new_,  // (b, s_k_new, h_k, dv) or (total_k_new, h_k, dv) if there is cu_seqlens_k_new
    std::optional<at::Tensor> q_v_,    // (b, s_q, h, dv) or (total_q_new, h, dv) if there is cu_seqlens_q
```
**EN:** This section defines `PyModuleDef`, `REGISTER_EXTENSION`, `PyModule_Create`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`PyModuleDef`、`REGISTER_EXTENSION`、`PyModule_Create`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 54-71: Runtime integration and dispatch
```cpp
    std::optional<at::Tensor> out_,    // (b, s_q, h, dv) or (total_q, h, dv) if there is cu_seqlens_q
    std::optional<at::Tensor> cu_seqlens_q_,      // b+1
    std::optional<at::Tensor> cu_seqlens_k_,      // b+1
    std::optional<at::Tensor> cu_seqlens_k_new_,  // b+1
    std::optional<at::Tensor>
        seqused_q_,  // b. If given, only this many elements of each batch element's queries and outputs are used.
    std::optional<at::Tensor>
        seqused_k_,  // b. If given, only this many elements of each batch element's keys are used.
    std::optional<int64_t> max_seqlen_q_,
    // TODO: check if we need max_seqlen_k
    std::optional<int64_t> max_seqlen_k_,
    std::optional<at::Tensor> page_table_,      // (b_k, max_num_pages_per_seq)
    std::optional<at::Tensor> kv_batch_idx_,    // b. indices to index into the KV cache
    std::optional<at::Tensor> leftpad_k_,       // b
    std::optional<at::Tensor> rotary_cos_,      // seqlen_ro x (rotary_dim / 2)
    std::optional<at::Tensor> rotary_sin_,      // seqlen_ro x (rotary_dim / 2)
    std::optional<at::Tensor> seqlens_rotary_,  // b
    std::optional<at::Tensor> q_descale_,       // (b, h_k), not (b, h)
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 72-86: Runtime integration and dispatch
```cpp
    std::optional<at::Tensor> k_descale_,       // (b, h_k)
    std::optional<at::Tensor> v_descale_,       // (b, h_k)
    std::optional<double> softmax_scale_,
    bool is_causal,
    int64_t window_size_left,
    int64_t window_size_right,
    int64_t attention_chunk,
    double softcap,
    bool is_rotary_interleaved,  // if true, rotary combines indices 0 & 1, else indices 0 & rotary_dim / 2
    std::optional<at::Tensor> scheduler_metadata_,  // (b + 1)
    int64_t num_splits,
    std::optional<bool> pack_gqa_,
    int64_t sm_margin,
    std::optional<const at::Tensor>& sinks_);  // (h)
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 87-104: Runtime integration and dispatch
```cpp
/*
 * From flash-attention: get_scheduler_metadata
 * Precomputes tile scheduling metadata for FA3 so that the prepare_varlen_num_blocks
 * kernel does not need to run per-layer.
 */
at::Tensor mha_fwd_get_scheduler_metadata(
    int64_t batch_size,
    int64_t max_seqlen_q,
    int64_t max_seqlen_k,
    int64_t num_heads,
    int64_t num_heads_k,
    int64_t headdim,
    int64_t headdim_v,
    at::ScalarType qkv_dtype,
    at::Tensor seqused_k,
    std::optional<at::Tensor> cu_seqlens_q_,
    std::optional<at::Tensor> cu_seqlens_k_,
    std::optional<at::Tensor> cu_seqlens_k_new_,
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 105-116: Runtime integration and dispatch
```cpp
    std::optional<at::Tensor> seqused_q_,
    std::optional<at::Tensor> leftpad_k_,
    std::optional<int64_t> page_size,
    int64_t max_seqlen_k_new,
    bool is_causal,
    int64_t window_size_left,
    int64_t window_size_right,
    int64_t attention_chunk,
    bool has_softcap,
    int64_t num_splits,
    std::optional<bool> pack_gqa_,
    int64_t sm_margin);
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `sgl_kernel_torch_shim.h`
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/Tensor.h`, `Python.h`, `torch/library.h`, `torch/torch.h`, `vector`
- **Path context / 路径上下文**: include / sgl_flash_kernel_ops.h
