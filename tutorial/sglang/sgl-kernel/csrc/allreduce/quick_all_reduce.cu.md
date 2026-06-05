# quick_all_reduce.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/quick_all_reduce.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Control flow and branching
```cpp
#include <ATen/cuda/Exceptions.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/cuda/CUDAStream.h>
#include <torch/all.h>

#ifdef USE_ROCM

#include "quick_all_reduce.h"

quickreduce::fptr_t init_custom_qr(int64_t rank, int64_t world_size, std::optional<int64_t> qr_max_size) {
  if (world_size > 8) throw std::invalid_argument("world size > 8 is not supported");
  if (world_size == 6) throw std::invalid_argument("world size == 6 is not supported");
  if (world_size % 2 != 0) throw std::invalid_argument("Odd num gpus is not supported for now");
  if (rank < 0 || rank >= world_size) throw std::invalid_argument("invalid rank passed in");
  quickreduce::DeviceComms* fptr = new quickreduce::DeviceComms();
  fptr->init(world_size, rank, qr_max_size);
  return (quickreduce::fptr_t)fptr;
}
```
**EN:** This section drives `init_custom_qr`, `DeviceComms`, `init` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`init_custom_qr`、`DeviceComms`、`init`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 19-36: Runtime integration and dispatch
```cpp

void qr_destroy(quickreduce::fptr_t _fa) {
  if (_fa) {
    auto fa = reinterpret_cast<quickreduce::DeviceComms*>(_fa);
    fa->destroy();
    delete fa;
  }
}

torch::Tensor qr_get_handle(quickreduce::fptr_t _fa) {
  auto fa = reinterpret_cast<quickreduce::DeviceComms*>(_fa);
  hipIpcMemHandle_t handle = fa->get_handle();
  auto options = torch::TensorOptions().dtype(torch::kUInt8).device(torch::kCPU);
  auto data_handle = torch::empty({static_cast<int64_t>(sizeof(hipIpcMemHandle_t))}, options);
  std::memcpy(data_handle.data_ptr(), &handle, sizeof(hipIpcMemHandle_t));
  return data_handle;
}
```
**EN:** This section uses `qr_destroy`, `qr_get_handle`, `destroy` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`qr_destroy`、`qr_get_handle`、`destroy`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 37-55: Runtime integration and dispatch
```cpp
void qr_open_handles(quickreduce::fptr_t _fa, const std::vector<torch::Tensor>& handles) {
  auto fa = reinterpret_cast<quickreduce::DeviceComms*>(_fa);
  std::vector<hipIpcMemHandle_t> ipc_handles;
  ipc_handles.reserve(handles.size());
  for (auto& handle : handles) {
    // Ensure the tensor is on the same device as the current device.
    hipIpcMemHandle_t ipc_handle;
    std::memcpy(&ipc_handle, handle.data_ptr(), sizeof(hipIpcMemHandle_t));
    ipc_handles.push_back(ipc_handle);
  }
  fa->open_ipc_handles(ipc_handles);
}

void qr_all_reduce(
    quickreduce::fptr_t _fa, torch::Tensor& inp, torch::Tensor& out, int64_t quant_level, bool cast_bf2half) {
  auto fa = reinterpret_cast<quickreduce::DeviceComms*>(_fa);
  const at::cuda::OptionalCUDAGuard device_guard(device_of(inp));
  auto stream = at::cuda::getCurrentHIPStreamMasqueradingAsCUDA();
```
**EN:** This section uses `qr_open_handles`, `qr_all_reduce`, `reserve` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`qr_open_handles`、`qr_all_reduce`、`reserve`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 56-73: Runtime integration and dispatch
```cpp
  TORCH_CHECK_EQ(inp.scalar_type(), out.scalar_type());
  TORCH_CHECK_EQ(inp.numel(), out.numel());
  TORCH_CHECK_LE(out.numel(), fa->kMaxProblemSize);
  if (out.scalar_type() == at::ScalarType::Half) {
    fa->allreduce<half, false>(
        reinterpret_cast<half*>(inp.data_ptr()),
        reinterpret_cast<half*>(out.data_ptr()),
        out.numel(),
        quant_level,
        stream);
  } else if (out.scalar_type() == at::ScalarType::BFloat16) {
    if (cast_bf2half) {
      fa->allreduce<half, true>(
          reinterpret_cast<half*>(inp.data_ptr()),
          reinterpret_cast<half*>(out.data_ptr()),
          out.numel(),
          quant_level,
          stream);
```
**EN:** This section uses `TORCH_CHECK_EQ`, `TORCH_CHECK_LE`, `data_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK_EQ`、`TORCH_CHECK_LE`、`data_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 74-91: Local implementation details
```cpp
    } else {
      fa->allreduce<quickreduce::nv_bfloat16, false>(
          reinterpret_cast<quickreduce::nv_bfloat16*>(inp.data_ptr()),
          reinterpret_cast<quickreduce::nv_bfloat16*>(out.data_ptr()),
          out.numel(),
          quant_level,
          stream);
    }
  } else {
    throw std::runtime_error("quick allreduce only supports float16 and bfloat16");
  }
}

int64_t qr_max_size() {
  // The default is 2GB (2,147,483,648 bytes)
  return static_cast<int64_t>(std::numeric_limits<int32_t>::max()) + 1;
}
```
**EN:** This section fills in the local implementation details around `qr_max_size`, `data_ptr`, `runtime_error`, completing the behavior required by the file.
**CN:** 本段补充了`qr_max_size`、`data_ptr`、`runtime_error`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 92-110: Types and data layout
```cpp
#define INSTANTIATE_FOR_WORLDSIZE(T, Codec, cast_bf2half)                      \
  template struct quickreduce::AllReduceTwoshot<T, Codec<T, 2>, cast_bf2half>; \
  template struct quickreduce::AllReduceTwoshot<T, Codec<T, 4>, cast_bf2half>; \
  template struct quickreduce::AllReduceTwoshot<T, Codec<T, 8>, cast_bf2half>;

INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecFP, false)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ4, false)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ6, false)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ8, false)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecFP, true)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ4, true)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ6, true)
INSTANTIATE_FOR_WORLDSIZE(quickreduce::nv_bfloat16, quickreduce::CodecQ8, true)

INSTANTIATE_FOR_WORLDSIZE(half, quickreduce::CodecFP, false)
INSTANTIATE_FOR_WORLDSIZE(half, quickreduce::CodecQ4, false)
INSTANTIATE_FOR_WORLDSIZE(half, quickreduce::CodecQ6, false)
INSTANTIATE_FOR_WORLDSIZE(half, quickreduce::CodecQ8, false)
```
**EN:** This section defines `quickreduce::AllReduceTwoshot`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`quickreduce::AllReduceTwoshot`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 111-111: Local implementation details
```cpp
#endif  // USE_ROCM
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `quick_all_reduce.h`
- **External headers / 外部头文件**: `ATen/cuda/Exceptions.h`, `c10/cuda/CUDAGuard.h`, `c10/cuda/CUDAStream.h`, `torch/all.h`
- **Path context / 路径上下文**: allreduce / quick_all_reduce.cu
