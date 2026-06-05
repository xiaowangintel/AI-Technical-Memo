# speculative_sampling.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/speculative/speculative_sampling.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Headers and compile-time setup
```cpp
/*
 * Copyright (c) 2025 by SGLang team.
 * Copyright (c) 2025 by FlashInfer team.
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
#include "pytorch_extension_utils.h"
#include "speculative_sampling.cuh"

using namespace flashinfer;
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 22-43: Runtime integration and dispatch
```cpp
// predicts: [tot_num_draft_tokens]
// accept_index: [bs, num_spec_step]
// accept_token_num: [bs]
// candidates: [bs, num_draft_tokens]
// retrive_index: [bs, num_draft_tokens]
// retrive_next_token: [bs, num_draft_tokens]
// retrive_next_sibling: [bs, num_draft_tokens]
// uniform_samples: [bs, num_draft_tokens]
// target_probs: [bs, num_draft_tokens, vocab_size]
void tree_speculative_sampling_target_only(
    at::Tensor predicts,
    at::Tensor accept_index,
    at::Tensor accept_token_num,  // mutable
    at::Tensor candidates,
    at::Tensor retrive_index,
    at::Tensor retrive_next_token,
    at::Tensor retrive_next_sibling,
    at::Tensor uniform_samples,
    at::Tensor uniform_samples_for_final_sampling,
    at::Tensor target_probs,
    at::Tensor draft_probs,
    double threshold_single,
```
**EN:** This section uses the surrounding logic to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过相关逻辑等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 44-65: Local implementation details
```cpp
    double threshold_acc,
    bool deterministic = true) {
  CHECK_INPUT(candidates);
  CHECK_INPUT(retrive_index);
  CHECK_INPUT(retrive_next_token);
  CHECK_INPUT(retrive_next_sibling);
  CHECK_INPUT(uniform_samples);
  CHECK_INPUT(uniform_samples_for_final_sampling);
  CHECK_INPUT(target_probs);
  auto device = target_probs.device();
  CHECK_EQ(candidates.device(), device);
  CHECK_EQ(retrive_index.device(), device);
  CHECK_EQ(retrive_next_token.device(), device);
  CHECK_EQ(retrive_next_sibling.device(), device);
  CHECK_EQ(uniform_samples.device(), device);
  CHECK_EQ(uniform_samples_for_final_sampling.device(), device);
  CHECK_EQ(target_probs.device(), device);
  CHECK_DIM(1, predicts);
  CHECK_DIM(2, accept_index);
  CHECK_DIM(1, accept_token_num);
  CHECK_DIM(2, candidates);
  CHECK_DIM(2, retrive_index);
```
**EN:** This section fills in the local implementation details around `CHECK_INPUT`, `device`, `CHECK_EQ`, completing the behavior required by the file.
**CN:** 本段补充了`CHECK_INPUT`、`device`、`CHECK_EQ`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 66-90: Runtime integration and dispatch
```cpp
  CHECK_DIM(2, retrive_next_token);
  CHECK_DIM(2, retrive_next_sibling);
  CHECK_DIM(2, uniform_samples);
  CHECK_DIM(3, target_probs);
  CHECK_DIM(3, draft_probs);
  unsigned int batch_size = uniform_samples.size(0);
  unsigned int num_spec_step = accept_index.size(1);
  unsigned int num_draft_tokens = candidates.size(1);
  unsigned int vocab_size = target_probs.size(2);
  CHECK_EQ(batch_size, candidates.size(0));
  CHECK_EQ(batch_size, retrive_index.size(0));
  CHECK_EQ(batch_size, retrive_next_token.size(0));
  CHECK_EQ(batch_size, retrive_next_sibling.size(0));
  CHECK_EQ(batch_size, target_probs.size(0));
  CHECK_EQ(num_draft_tokens, retrive_index.size(1));
  CHECK_EQ(num_draft_tokens, retrive_next_token.size(1));
  CHECK_EQ(num_draft_tokens, retrive_next_sibling.size(1));
  CHECK_EQ(num_draft_tokens, uniform_samples.size(1));
  CHECK_EQ(num_draft_tokens, target_probs.size(1));
  CHECK_EQ(vocab_size, target_probs.size(2));
  CHECK_EQ(batch_size, accept_index.size(0));
  CHECK_EQ(batch_size, accept_token_num.size(0));
  if (predicts.scalar_type() != at::kInt) {
    throw std::runtime_error("Expected 'predicts' to be of type int (torch.int32).");
  }
```
**EN:** This section uses `CHECK_DIM`, `CHECK_EQ`, `runtime_error` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_DIM`、`CHECK_EQ`、`runtime_error`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 91-111: Runtime integration and dispatch
```cpp
  if (accept_index.scalar_type() != at::kInt) {
    throw std::runtime_error("Expected 'accept_index' to be of type int (torch.int32).");
  }
  if (accept_token_num.scalar_type() != at::kInt) {
    throw std::runtime_error("Expected 'accept_token_num' to be of type int (torch.int32).");
  }
  if (candidates.scalar_type() != at::kLong) {
    throw std::runtime_error("Expected 'candidates' to be of type long (torch.int64).");
  }
  if (retrive_index.scalar_type() != at::kLong) {
    throw std::runtime_error("Expected 'retrive_index' to be of type long (torch.int64).");
  }
  if (retrive_next_token.scalar_type() != at::kLong) {
    throw std::runtime_error("Expected 'retrive_next_token' to be of type long (torch.int64).");
  }
  if (retrive_next_sibling.scalar_type() != at::kLong) {
    throw std::runtime_error("Expected 'retrive_next_sibling' to be of type long (torch.int64).");
  }
  if (uniform_samples.scalar_type() != at::kFloat) {
    throw std::runtime_error("Expected 'uniform_samples' to be of type float (torch.float32).");
  }
```
**EN:** This section uses `runtime_error` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`runtime_error`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 112-133: Runtime integration and dispatch
```cpp
  if (uniform_samples_for_final_sampling.scalar_type() != at::kFloat) {
    throw std::runtime_error("Expected 'uniform_samples_for_final_sampling' to be of type float (torch.float32).");
  }
  if (target_probs.scalar_type() != at::kFloat) {
    throw std::runtime_error("Expected 'target_probs' to be of type float (torch.float32).");
  }
  if (draft_probs.scalar_type() != at::kFloat) {
    throw std::runtime_error("Expected 'target_probs' to be of type float (torch.float32).");
  }
  CHECK_GE(threshold_single, 0);
  CHECK_GE(1, threshold_single);
  CHECK_GE(threshold_acc, 0);
  CHECK_GE(1, threshold_acc);

  cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  cudaError_t status = sampling::TreeSpeculativeSamplingTargetOnly<float, int32_t, int64_t>(
      static_cast<int32_t*>(predicts.data_ptr()),
      static_cast<int32_t*>(accept_index.data_ptr()),
      static_cast<int32_t*>(accept_token_num.data_ptr()),
      static_cast<int64_t*>(candidates.data_ptr()),
      static_cast<int64_t*>(retrive_index.data_ptr()),
      static_cast<int64_t*>(retrive_next_token.data_ptr()),
```
**EN:** This section uses `runtime_error`, `CHECK_GE`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`runtime_error`、`CHECK_GE`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 134-151: Runtime integration and dispatch
```cpp
      static_cast<int64_t*>(retrive_next_sibling.data_ptr()),
      static_cast<float*>(uniform_samples.data_ptr()),
      static_cast<float*>(uniform_samples_for_final_sampling.data_ptr()),
      static_cast<float*>(target_probs.data_ptr()),
      static_cast<float*>(draft_probs.data_ptr()),
      batch_size,
      num_spec_step,
      num_draft_tokens,
      vocab_size,
      static_cast<float>(threshold_single),
      static_cast<float>(threshold_acc),
      deterministic,
      stream);

  TORCH_CHECK(
      status == cudaSuccess,
      "TreeSpeculativeSamplingTargetOnly failed with error code " + std::string(cudaGetErrorString(status)));
}
```
**EN:** This section uses `data_ptr`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `pytorch_extension_utils.h`, `speculative_sampling.cuh`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: speculative / speculative_sampling.cu
