# sgl_kernel_musa_ops.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/sgl_kernel_musa_ops.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Local implementation details
```cpp
/*
 * Copyright (c) 2020-2026, Moore Threads Technology Co., Ltd("Moore Threads").
 * All rights reserved.
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

#pragma once
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 20-35: Runtime integration and dispatch
```cpp
#include <ATen/ATen.h>
#include <ATen/Tensor.h>
#include <torch/torch.h>

#include <optional>

void batched_rotary_embedding_contiguous(
    torch::Tensor& positions,
    torch::Tensor& query,
    torch::Tensor& key,
    int64_t head_size,
    torch::Tensor& cos_sin_cache,
    bool is_neox,
    int64_t rot_dim,
    torch::Tensor& cos_sin_cache_offsets);
```
**EN:** This section uses `batched_rotary_embedding_contiguous` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`batched_rotary_embedding_contiguous`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 36-56: Runtime integration and dispatch
```cpp
void rotary_embedding_contiguous(
    torch::Tensor& positions,
    torch::Tensor& query,
    torch::Tensor& key,
    int64_t head_size,
    torch::Tensor& cos_sin_cache,
    bool is_neox);

void fused_moe_gemv(
    torch::Tensor& A,
    torch::Tensor& B,
    torch::Tensor& C,
    const c10::optional<torch::Tensor>& A_scale,
    const c10::optional<torch::Tensor>& B_scale,
    torch::Tensor& topk_weights,
    torch::Tensor& topk_ids,
    bool mul_routed_weight,
    int64_t topk,
    bool use_int4_w4a16,
    bool use_swigelu);
```
**EN:** This section uses `rotary_embedding_contiguous`, `fused_moe_gemv` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`rotary_embedding_contiguous`、`fused_moe_gemv`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 57-70: Runtime integration and dispatch
```cpp
void musa_fused_gemv(
    torch::Tensor& A,
    torch::Tensor& B,
    torch::Tensor& C,
    const c10::optional<torch::Tensor>& A_scale,
    const c10::optional<torch::Tensor>& B_scale,
    bool use_int4_w4a16,
    bool use_swigelu,
    bool use_rms_norm,
    const c10::optional<torch::Tensor>& gamma,
    double eps);

void fused_mul_add(torch::Tensor& output, torch::Tensor& self, torch::Tensor& bias, double scale);
```
**EN:** This section uses `musa_fused_gemv`, `fused_mul_add` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`musa_fused_gemv`、`fused_mul_add`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 71-90: Runtime integration and dispatch
```cpp
void musa_top_k_top_p_sampling_from_probs(
    at::Tensor probs,
    at::Tensor output,
    std::optional<at::Tensor> maybe_indices,
    std::optional<at::Tensor> maybe_top_k_arr,
    double top_k_val,
    std::optional<at::Tensor> maybe_top_p_arr,
    double top_p_val,
    bool deterministic,
    std::optional<at::Generator> gen);

void min_p_sampling_from_probs(
    at::Tensor probs,
    at::Tensor output,
    std::optional<at::Tensor> maybe_indices,
    std::optional<at::Tensor> maybe_min_p_arr,
    double min_p_val,
    bool deterministic,
    std::optional<at::Generator> gen);
```
**EN:** This section uses `musa_top_k_top_p_sampling_from_probs`, `min_p_sampling_from_probs` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`musa_top_k_top_p_sampling_from_probs`、`min_p_sampling_from_probs`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 91-98: Runtime integration and dispatch
```cpp
void top_p_sampling_from_probs(
    at::Tensor probs,
    at::Tensor output,
    std::optional<at::Tensor> maybe_indices,
    std::optional<at::Tensor> maybe_top_p_arr,
    double top_p_val,
    bool deterministic,
    std::optional<at::Generator> gen);
```
**EN:** This section uses `top_p_sampling_from_probs` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`top_p_sampling_from_probs`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **MUSA portability / MUSA 可移植性**: Contains conditional logic for the MUSA toolchain or runtime. / 包含面向 MUSA 工具链或运行时的条件分支。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/Tensor.h`, `torch/torch.h`, `optional`
- **Path context / 路径上下文**: include / sgl_kernel_musa_ops.h
