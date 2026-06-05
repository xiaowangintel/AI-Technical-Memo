# cutlass_mla_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/attention/cutlass_mla_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements attention-related kernels, layouts, or dispatch helpers for high-throughput inference. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现面向高吞吐推理的 attention 内核、数据布局或分发辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Headers and compile-time setup
```cpp
/*
Copyright (c) 2025, NVIDIA CORPORATION.  All rights reserved.
Copyright 2025 SGLang Team. All Rights Reserved.

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

#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <cutlass/cutlass.h>
#include <cutlass/kernel_hardware_info.h>
#include <torch/all.h>
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 24-46: Runtime integration and dispatch
```cpp
#include <cute/tensor.hpp>
#include <iostream>

#include "cutlass_sm100_mla/device/sm100_mla.hpp"
#include "cutlass_sm100_mla/kernel/sm100_mla_tile_scheduler.hpp"
#include "utils.h"

// clang-format off
#if !defined(CUDA_VERSION) || CUDA_VERSION < 12040
void cutlass_mla_decode(
    torch::Tensor const& out,
    torch::Tensor const& q_nope,
    torch::Tensor const& q_pe,
    torch::Tensor const& kv_c_and_k_pe_cache,
    torch::Tensor const& seq_lens,
    torch::Tensor const& page_table,
    torch::Tensor const& workspace,
    int64_t num_kv_splits) {
  TORCH_CHECK(false, "CUDA version must be >= 12.4 for cutlass_mla_decode");
}
int64_t cutlass_mla_get_workspace_size(int64_t max_seq_len, int64_t num_batches, int64_t sm_count, int64_t num_kv_splits) {
  TORCH_CHECK(false, "CUDA version must be >= 12.4 for cutlass_mla_get_workspace_size");
}
```
**EN:** This section uses `defined`, `cutlass_mla_get_workspace_size`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`cutlass_mla_get_workspace_size`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 47-68: Types and data layout
```cpp
#else

#define CUTLASS_CHECK(status)                                                       \
  {                                                                                 \
    cutlass::Status error = status;                                                 \
    TORCH_CHECK(error == cutlass::Status::kSuccess, cutlassGetStatusString(error)); \
  }

using namespace cute;
using namespace cutlass::fmha::kernel;

template <bool v>
struct IsPersistent {
  static const bool value = v;
};

template <typename T, bool IsPaged128, typename PersistenceOption = IsPersistent<true>>
struct MlaSm100 {
  using Element = T;
  using ElementAcc = float;
  using ElementOut = T;
```
**EN:** This section defines `IsPersistent`, `MlaSm100`, `TORCH_CHECK`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`IsPersistent`、`MlaSm100`、`TORCH_CHECK`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 69-93: Device helpers and synchronization
```cpp
  using TileShape = Shape<_128, _128, Shape<_512, _64>>;
  using TileShapeH = cute::tuple_element_t<0, TileShape>;
  using TileShapeD = cute::tuple_element_t<2, TileShape>;

  // H K (D_latent D_rope) B
  using ProblemShape = cute::tuple<TileShapeH, int, TileShapeD, int>;

  using StrideQ = cute::tuple<int64_t, _1, int64_t>;  // H D B
  using StrideK = cute::tuple<int64_t, _1, int64_t>;  // K D B
  using StrideO = StrideK;                            // H D B
  using StrideLSE = cute::tuple<_1, int>;             // H B

  using TileScheduler =
      std::conditional_t<PersistenceOption::value, Sm100MlaPersistentTileScheduler, Sm100MlaIndividualTileScheduler>;

  using FmhaKernel = cutlass::fmha::kernel::Sm100FmhaMlaKernelTmaWarpspecialized<
      TileShape,
      Element,
      ElementAcc,
      ElementOut,
      ElementAcc,
      TileScheduler,
      /*kIsCpAsync=*/!IsPaged128>;
  using Fmha = cutlass::fmha::device::MLA<FmhaKernel>;
};
```
**EN:** This section implements `TileShape`, `TileShapeH`, `TileShapeD`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`TileShape`、`TileShapeH`、`TileShapeD`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 94-117: Runtime integration and dispatch
```cpp

template <typename T>
typename T::Fmha::Arguments args_from_options(
    at::Tensor const& out,
    at::Tensor const& q_nope,
    at::Tensor const& q_pe,
    at::Tensor const& kv_c_and_k_pe_cache,
    at::Tensor const& seq_lens,
    at::Tensor const& page_table,
    double sm_scale,
    int64_t num_kv_splits) {
  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = q_nope.device().index();
  hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);

  int batches = q_nope.size(0);
  int page_count_per_seq = page_table.size(1);
  int page_count_total = kv_c_and_k_pe_cache.size(0);
  int page_size = kv_c_and_k_pe_cache.size(1);
  int max_seq_len = page_size * page_count_per_seq;
  using TileShapeH = typename T::TileShapeH;
  using TileShapeD = typename T::TileShapeD;
  auto problem_shape = cute::make_tuple(TileShapeH{}, max_seq_len, TileShapeD{}, batches);
```
**EN:** This section uses `args_from_options`, `device`, `query_device_multiprocessor_count` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`args_from_options`、`device`、`query_device_multiprocessor_count`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 118-138: Templates, aliases, and constants
```cpp
  auto [H, K, D, B] = problem_shape;
  auto [D_latent, D_rope] = D;

  float scale = float(sm_scale);

  using StrideQ = typename T::StrideQ;
  using StrideK = typename T::StrideK;
  using StrideO = typename T::StrideO;
  using StrideLSE = typename T::StrideLSE;

  StrideQ stride_Q_nope = cute::make_tuple(
      static_cast<int64_t>(q_nope.stride(1)), _1{}, static_cast<int64_t>(q_nope.stride(0)));
  StrideQ stride_Q_pe = cute::make_tuple(
      static_cast<int64_t>(q_pe.stride(1)), _1{}, static_cast<int64_t>(q_pe.stride(0)));

  StrideK stride_C = cute::make_tuple(
      static_cast<int64_t>(0 + D_latent + D_rope), _1{}, static_cast<int64_t>(page_size * (D_latent + D_rope)));
  StrideLSE stride_PT = cute::make_stride(_1{}, page_count_per_seq);
  StrideLSE stride_LSE = cute::make_tuple(_1{}, 0 + H);
  StrideO stride_O = cute::make_tuple(static_cast<int64_t>(0 + D_latent), _1{}, static_cast<int64_t>(0 + H * D_latent));
```
**EN:** This section defines `float`, `stride`, `StrideQ`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`float`、`stride`、`StrideQ`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 139-160: Templates, aliases, and constants
```cpp
  using Element = typename T::Element;
  using ElementOut = typename T::ElementOut;
  using ElementAcc = typename T::ElementAcc;
  auto Q_nope_ptr = static_cast<Element*>(q_nope.data_ptr());
  auto Q_pe_ptr = static_cast<Element*>(q_pe.data_ptr());
  auto C_ptr = static_cast<Element*>(kv_c_and_k_pe_cache.data_ptr());
  typename T::Fmha::Arguments arguments{
      problem_shape,
      {scale,
       Q_nope_ptr,
       stride_Q_nope,
       Q_pe_ptr,
       stride_Q_pe,
       C_ptr,
       stride_C,
       C_ptr + D_latent,
       stride_C,
       static_cast<int*>(seq_lens.data_ptr()),
       static_cast<int*>(page_table.data_ptr()),
       stride_PT,
       page_count_total,
       page_size},
```
**EN:** This section defines `data_ptr`, `Element`, `ElementOut`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_ptr`、`Element`、`ElementOut`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 161-176: Local implementation details
```cpp
      {static_cast<ElementOut*>(out.data_ptr()), stride_O, static_cast<ElementAcc*>(nullptr), stride_LSE},
      hw_info,
      // TODO(trevor-m): Change split_kv back to -1 when
      // https://github.com/NVIDIA/cutlass/issues/2274 is fixed. Split_kv=1 will
      // perform worse with larger context length and smaller batch sizes.
      static_cast<int>(num_kv_splits), // split_kv
      nullptr,       // is_var_split_kv
  };
  // TODO(kaixih@nvidia): When split_kv=-1 and is_var_split_kv=false, we compute
  // split_kv automatically based on batch size and sequence length to balance
  // workload across available SMs. Consider using var_split_kv for manual
  // control if needed.
  T::Fmha::set_split_kv(arguments);
  return arguments;
}
```
**EN:** This section fills in the local implementation details around `set_split_kv`, completing the behavior required by the file.
**CN:** 本段补充了`set_split_kv`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 177-198: Runtime integration and dispatch
```cpp
template <typename Element, bool IsPaged128, typename PersistenceOption>
void runMla(
    at::Tensor const& out,
    at::Tensor const& q_nope,
    at::Tensor const& q_pe,
    at::Tensor const& kv_c_and_k_pe_cache,
    at::Tensor const& seq_lens,
    at::Tensor const& page_table,
    at::Tensor const& workspace,
    double sm_scale,
    int64_t num_kv_splits,
    cudaStream_t stream) {
  using MlaSm100Type = MlaSm100<Element, IsPaged128, PersistenceOption>;
  typename MlaSm100Type::Fmha fmha;
  auto arguments = args_from_options<MlaSm100Type>(out, q_nope, q_pe, kv_c_and_k_pe_cache, seq_lens, page_table, sm_scale, num_kv_splits);

  CUTLASS_CHECK(fmha.can_implement(arguments));

  CUTLASS_CHECK(fmha.initialize(arguments, workspace.data_ptr(), stream));

  CUTLASS_CHECK(fmha.run(arguments, workspace.data_ptr(), stream));
}
```
**EN:** This section uses `runMla`, `CUTLASS_CHECK`, `MlaSm100Type` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`runMla`、`CUTLASS_CHECK`、`MlaSm100Type`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 199-224: Runtime integration and dispatch
```cpp

#define DISPATCH_BOOL(expr, const_expr, ...) \
  [&]() -> bool {                            \
    if (expr) {                              \
      constexpr bool const_expr = true;      \
      return __VA_ARGS__();                  \
    } else {                                 \
      constexpr bool const_expr = false;     \
      return __VA_ARGS__();                  \
    }                                        \
  }()

void cutlass_mla_decode(
    torch::Tensor const& out,
    torch::Tensor const& q_nope,
    torch::Tensor const& q_pe,
    torch::Tensor const& kv_c_and_k_pe_cache,
    torch::Tensor const& seq_lens,
    torch::Tensor const& page_table,
    torch::Tensor const& workspace,
    double sm_scale,
    int64_t num_kv_splits) {
  auto sm_version = getSMVersion();
  // On SM103a, half of the accuracy tests are failing.
  TORCH_CHECK(sm_version == 100, "cutlass_mla_decode is only supported on compute capability 10.0, but found sm version ", sm_version);
```
**EN:** This section uses `DISPATCH_BOOL`, `cutlass_mla_decode`, `__VA_ARGS__` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`DISPATCH_BOOL`、`cutlass_mla_decode`、`__VA_ARGS__`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 225-246: Runtime integration and dispatch
```cpp
  auto in_dtype = q_nope.dtype();
  at::cuda::CUDAGuard device_guard{(char)q_nope.get_device()};
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream(q_nope.get_device());
  const int page_size = kv_c_and_k_pe_cache.size(1);

  // NOTE(alcanderian): IsPersistent has bug with manual split_kv.
  // Kernel will hang if batch is too large with large num_kv_splits. (for example bs=8, num_kv_splits=8)
  // Maybe per batch split kv will fix this.
  DISPATCH_BOOL(page_size == 128, IsPaged128, [&] {
    DISPATCH_BOOL(num_kv_splits <= 1, NotManualSplitKV, [&] {
      if (in_dtype == at::ScalarType::Half) {
        runMla<cutlass::half_t, IsPaged128, IsPersistent<NotManualSplitKV>>(
          out, q_nope, q_pe, kv_c_and_k_pe_cache, seq_lens, page_table, workspace, sm_scale, num_kv_splits, stream);
      } else if (in_dtype == at::ScalarType::BFloat16) {
        runMla<cutlass::bfloat16_t, IsPaged128, IsPersistent<NotManualSplitKV>>(
          out, q_nope, q_pe, kv_c_and_k_pe_cache, seq_lens, page_table, workspace, sm_scale, num_kv_splits, stream);
      } else if (in_dtype == at::ScalarType::Float8_e4m3fn) {
        runMla<cutlass::float_e4m3_t, IsPaged128, IsPersistent<NotManualSplitKV>>(
          out, q_nope, q_pe, kv_c_and_k_pe_cache, seq_lens, page_table, workspace, sm_scale, num_kv_splits, stream);
      } else {
        TORCH_CHECK(false, "Unsupported input data type of MLA");
      }
```
**EN:** This section uses `dtype`, `getCurrentCUDAStream`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`dtype`、`getCurrentCUDAStream`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 247-269: Templates, aliases, and constants
```cpp
      return true;
    });
    return true;
  });
}

int64_t cutlass_mla_get_workspace_size(int64_t max_seq_len, int64_t num_batches, int64_t sm_count, int64_t num_kv_splits) {
  // Workspace size depends on ElementAcc and ElementLSE (same as ElementAcc)
  // which are float, so Element type here doesn't matter.
  using MlaSm100Type = MlaSm100<cutlass::half_t, true>;

  // Get split kv. Requires problem shape and sm_count only.
  typename MlaSm100Type::Fmha::Arguments arguments;
  using TileShapeH = typename MlaSm100Type::TileShapeH;
  using TileShapeD = typename MlaSm100Type::TileShapeD;
  arguments.problem_shape =
      cute::make_tuple(TileShapeH{}, static_cast<int>(max_seq_len), TileShapeD{}, static_cast<int>(num_batches));
  // Assumes device 0 when getting sm_count.
  arguments.hw_info.sm_count =
      sm_count <= 0 ? cutlass::KernelHardwareInfo::query_device_multiprocessor_count(/*device_id=*/0) : sm_count;
  arguments.split_kv = static_cast<int>(num_kv_splits);
  MlaSm100Type::Fmha::set_split_kv(arguments);
```
**EN:** This section defines `cutlass_mla_get_workspace_size`, `set_split_kv`, `MlaSm100Type`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cutlass_mla_get_workspace_size`、`set_split_kv`、`MlaSm100Type`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 270-272: Local implementation details
```cpp
  return MlaSm100Type::Fmha::get_workspace_size(arguments);
}
```
**EN:** This section fills in the local implementation details around `get_workspace_size`, completing the behavior required by the file.
**CN:** 本段补充了`get_workspace_size`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 273-274: Local implementation details
```cpp
#endif
// clang-format on
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Validation / 校验测试**: Contains checks or scaffolding for correctness verification. / 包含正确性验证所需的检查或脚手架。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass_sm100_mla/device/sm100_mla.hpp`, `cutlass_sm100_mla/kernel/sm100_mla_tile_scheduler.hpp`, `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `cutlass/cutlass.h`, `cutlass/kernel_hardware_info.h`, `torch/all.h`, `cute/tensor.hpp`, `iostream`
- **Path context / 路径上下文**: attention / cutlass_mla_kernel.cu
