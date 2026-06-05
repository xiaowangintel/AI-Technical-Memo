# bmm_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/bmm_fp8.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Local implementation details
```cpp
/*
 * Copyright (c) 2024 by FlashInfer team.
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

#include <driver_types.h>
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 19-40: Runtime integration and dispatch
```cpp
#include <flashinfer/gemm/bmm_fp8.cuh>

#include "pytorch_extension_utils.h"

void bmm_fp8(
    at::Tensor A,
    at::Tensor B,
    at::Tensor D,
    at::Tensor A_scale,
    at::Tensor B_scale,
    at::Tensor workspace_buffer,
    int64_t cublas_handle) {
  TORCH_CHECK(A.is_cuda(), "A must be a CUDA tensor");
  TORCH_CHECK(B.is_cuda(), "B must be a CUDA tensor");
  TORCH_CHECK(D.is_cuda(), "D must be a CUDA tensor");
  TORCH_CHECK(A.dim() == 3, "Expected 3D tensor for A");
  TORCH_CHECK(B.dim() == 3, "Expected 3D tensor for B");
  TORCH_CHECK(D.dim() == 3, "Expected 3D tensor for D");
  TORCH_CHECK(A.size(0) == B.size(0) && A.size(0) == D.size(0), "Batch sizes must match");
  TORCH_CHECK(A.size(2) == B.size(1), "Incompatible matrix sizes");
  TORCH_CHECK(A.size(1) == D.size(1) && B.size(2) == D.size(2), "Result tensor has incorrect shape");
```
**EN:** This section uses `bmm_fp8`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`bmm_fp8`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 41-54: Runtime integration and dispatch
```cpp
  // PyTorch is row major by default. cuBLASLt is column major by default.
  // We need row major D as expected.
  // A ^ T * B = D, so D ^ T = B ^ T * A
  DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP8(B.scalar_type(), b_type, [&] {
    return DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP8(A.scalar_type(), a_type, [&] {
      return DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FP16(D.scalar_type(), d_type, [&] {
        auto batch_size = A.size(0);
        auto m = A.size(1);
        auto k = A.size(2);
        auto n = B.size(2);

        auto lt_handle = reinterpret_cast<cublasLtHandle_t>(cublas_handle);
        auto stream = at::cuda::getCurrentCUDAStream();
```
**EN:** This section uses `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 55-75: Runtime integration and dispatch
```cpp
        auto status = flashinfer::bmm_fp8::bmm_fp8_internal_cublaslt(
            workspace_buffer.data_ptr(),
            workspace_buffer.numel(),
            static_cast<b_type*>(B.data_ptr()),
            static_cast<a_type*>(A.data_ptr()),
            static_cast<d_type*>(D.data_ptr()),
            batch_size,
            n,
            m,
            k,
            static_cast<float*>(B_scale.data_ptr()),
            static_cast<float*>(A_scale.data_ptr()),
            lt_handle,
            stream);
        TORCH_CHECK(
            status == CUBLAS_STATUS_SUCCESS, "bmm_fp8_internal_cublaslt failed: ", cublasGetStatusString(status));
        return true;
      });
    });
  });
}
```
**EN:** This section uses `bmm_fp8_internal_cublaslt`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`bmm_fp8_internal_cublaslt`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `pytorch_extension_utils.h`
- **External headers / 外部头文件**: `driver_types.h`, `flashinfer/gemm/bmm_fp8.cuh`
- **Path context / 路径上下文**: gemm / bmm_fp8.cu
