# custom_all_reduce.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/custom_all_reduce.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Runtime integration and dispatch
```cpp
// Adapted from: https://github.com/vllm-project/vllm/blob/v0.8.2/csrc/custom_all_reduce.cu
#include <ATen/cuda/Exceptions.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/cuda/CUDAStream.h>
#include <torch/all.h>

#include "custom_all_reduce.cuh"

// Fake pointer type, must match fptr_t type in ops.h.
// We use this type alias to indicate when pointers are passed in as int64_t.
using fptr_t = int64_t;
static_assert(sizeof(void*) == sizeof(fptr_t));

fptr_t
init_custom_ar(const std::vector<fptr_t>& fake_ipc_ptrs, torch::Tensor& rank_data, int64_t rank, bool full_nvlink) {
  int world_size = fake_ipc_ptrs.size();
  if (world_size > 8) throw std::invalid_argument("world size > 8 is not supported");
  if (world_size % 2 != 0) throw std::invalid_argument("Odd num gpus is not supported for now");
  if (rank < 0 || rank >= world_size) throw std::invalid_argument("invalid rank passed in");

  sglang::Signal* ipc_ptrs[8];
  for (int i = 0; i < world_size; i++) {
    ipc_ptrs[i] = reinterpret_cast<sglang::Signal*>(fake_ipc_ptrs[i]);
  }
```
**EN:** This section uses `init_custom_ar`, `static_assert`, `fptr_t` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`init_custom_ar`、`static_assert`、`fptr_t`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 25-48: Runtime integration and dispatch
```cpp
  return (fptr_t) new sglang::CustomAllreduce(
      ipc_ptrs, rank_data.data_ptr(), rank_data.numel(), rank, world_size, full_nvlink);
}

/**
 * Make sure tensor t's data lies completely within ((char)t.data_ptr()) +
 * t.numel() * t.element_size(). This is slightly weaker than t.is_contiguous()
 * because it allows transpose of contiguous slice (i.e. slicing the first
 * dimension). Currently, we require this because stride information is not
 * passed into the kernels and we treat input tensors as flat.
 *
 * Examples
 * A = torch.zeros(3, 3, 3)
 * 1. A: OK
 * 2. A[1:]: OK
 * 3. A.permute(2, 0, 1): OK
 * 4. A[1:].permute(2, 0, 1): OK
 * 5. A[None].expand(2, -1, -1, -1): Not OK
 * 6. A[:, 1:, 1:]: Not OK
 */
bool _is_weak_contiguous(torch::Tensor& t) {
  return t.is_contiguous() ||
         (t.storage().nbytes() - t.storage_offset() * t.element_size() == t.numel() * t.element_size());
}
```
**EN:** This section uses `_is_weak_contiguous`, `is_contiguous` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_is_weak_contiguous`、`is_contiguous`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 49-73: Runtime integration and dispatch
```cpp

/**
 * Performs an out-of-place allreduce and stores result in out.
 *
 * If _reg_buffer is null, assumes inp.data_ptr() is already IPC-registered.
 * Otherwise, _reg_buffer is assumed to be IPC-registered and inp is first
 * copied into _reg_buffer.
 */
void all_reduce(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out, fptr_t _reg_buffer, int64_t reg_buffer_sz_bytes) {
  auto fa = reinterpret_cast<sglang::CustomAllreduce*>(_fa);
  const at::cuda::OptionalCUDAGuard device_guard(device_of(inp));
  auto stream = c10::cuda::getCurrentCUDAStream().stream();

  TORCH_CHECK_EQ(inp.scalar_type(), out.scalar_type());
  TORCH_CHECK_EQ(inp.numel(), out.numel());
  TORCH_CHECK(_is_weak_contiguous(out));
  TORCH_CHECK(_is_weak_contiguous(inp));
  auto input_size = inp.numel() * inp.element_size();
  auto reg_buffer = reinterpret_cast<void*>(_reg_buffer);
  if (reg_buffer) {
    TORCH_CHECK_LE(input_size, reg_buffer_sz_bytes);
    AT_CUDA_CHECK(cudaMemcpyAsync(reg_buffer, inp.data_ptr(), input_size, cudaMemcpyDeviceToDevice, stream));
  } else {
    reg_buffer = inp.data_ptr();
  }
```
**EN:** This section uses `all_reduce`, `device_guard`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`all_reduce`、`device_guard`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 74-97: Runtime integration and dispatch
```cpp
  switch (out.scalar_type()) {
    case at::ScalarType::Float: {
      fa->allreduce<float>(
          stream, reinterpret_cast<float*>(reg_buffer), reinterpret_cast<float*>(out.data_ptr()), out.numel());
      break;
    }
    case at::ScalarType::Half: {
      fa->allreduce<half>(
          stream, reinterpret_cast<half*>(reg_buffer), reinterpret_cast<half*>(out.data_ptr()), out.numel());
      break;
    }
#if (__CUDA_ARCH__ >= 800 || !defined(__CUDA_ARCH__))
    case at::ScalarType::BFloat16: {
      fa->allreduce<nv_bfloat16>(
          stream,
          reinterpret_cast<nv_bfloat16*>(reg_buffer),
          reinterpret_cast<nv_bfloat16*>(out.data_ptr()),
          out.numel());
      break;
    }
#endif
    default:
      throw std::runtime_error("custom allreduce only supports float32, float16 and bfloat16");
  }
```
**EN:** This section uses `data_ptr`, `runtime_error` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`、`runtime_error`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 98-117: Runtime integration and dispatch
```cpp
}

void dispose(fptr_t _fa) {
  delete reinterpret_cast<sglang::CustomAllreduce*>(_fa);
}

int64_t meta_size() {
  return sizeof(sglang::Signal);
}

void register_buffer(fptr_t _fa, const std::vector<fptr_t>& fake_ipc_ptrs) {
  auto fa = reinterpret_cast<sglang::CustomAllreduce*>(_fa);
  TORCH_CHECK(fake_ipc_ptrs.size() == fa->world_size_);
  void* ipc_ptrs[8];
  for (int i = 0; i < fake_ipc_ptrs.size(); i++) {
    ipc_ptrs[i] = reinterpret_cast<void*>(fake_ipc_ptrs[i]);
  }
  fa->register_buffer(ipc_ptrs);
}
```
**EN:** This section uses `dispose`, `meta_size`, `register_buffer` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`dispose`、`meta_size`、`register_buffer`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 118-137: Control flow and branching
```cpp
// Use vector<int64_t> to represent byte data for python binding compatibility.
std::tuple<std::vector<int64_t>, std::vector<int64_t>> get_graph_buffer_ipc_meta(fptr_t _fa) {
  auto fa = reinterpret_cast<sglang::CustomAllreduce*>(_fa);
  auto [handle, offsets] = fa->get_graph_buffer_ipc_meta();
  std::vector<int64_t> bytes(handle.begin(), handle.end());
  return std::make_tuple(bytes, offsets);
}

// Use vector<int64_t> to represent byte data for python binding compatibility.
void register_graph_buffers(
    fptr_t _fa, const std::vector<std::vector<int64_t>>& handles, const std::vector<std::vector<int64_t>>& offsets) {
  auto fa = reinterpret_cast<sglang::CustomAllreduce*>(_fa);
  std::vector<std::string> bytes;
  bytes.reserve(handles.size());
  for (int i = 0; i < handles.size(); i++) {
    bytes.emplace_back(handles[i].begin(), handles[i].end());
  }
  bytes.reserve(handles.size());
  fa->register_graph_buffers(bytes, offsets);
}
```
**EN:** This section drives `get_graph_buffer_ipc_meta`, `register_graph_buffers`, `bytes` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`get_graph_buffer_ipc_meta`、`register_graph_buffers`、`bytes`相关逻辑，决定采用哪条执行路径或数据处理策略。

## Key Concepts / 关键概念
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `custom_all_reduce.cuh`
- **External headers / 外部头文件**: `ATen/cuda/Exceptions.h`, `c10/cuda/CUDAGuard.h`, `c10/cuda/CUDAStream.h`, `torch/all.h`
- **Path context / 路径上下文**: allreduce / custom_all_reduce.cu
