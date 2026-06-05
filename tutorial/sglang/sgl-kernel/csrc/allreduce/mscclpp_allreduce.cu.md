# mscclpp_allreduce.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/allreduce/mscclpp_allreduce.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Handles multi-GPU all-reduce communication kernels and their supporting runtime logic. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 处理多 GPU all-reduce 通信内核及其配套运行时逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Types and data layout
```cpp
#include <c10/cuda/CUDAGuard.h>
#include <c10/cuda/CUDAStream.h>
#include <torch/all.h>
#include <torch/library.h>

#include "mscclpp_allreduce.cuh"

enum MscclContextSelection {
  MSCCL1NODELL = 1,
  MSCCL2NODELL = 2,
};

class MscclContext {
 public:
  MscclContextSelection selection_;
  std::shared_ptr<sglang::Msccl1NodeLLcontext> msccl_1nodeLL_context;
  std::shared_ptr<sglang::Msccl2NodeLLcontext> msccl_2nodeLL_context;
  MscclContext(MscclContextSelection selection) : selection_(selection) {}
  template <typename T>
  void allreduce(
      cudaStream_t stream, T* input, T* output, const size_t input_numel, int threads = 512, int block_limit = 21) {
    if (selection_ == MSCCL1NODELL) {
      msccl_1nodeLL_context->allreduce<T>(stream, input, output, input_numel, threads, block_limit);
    } else if (selection_ == MSCCL2NODELL) {
      msccl_2nodeLL_context->allreduce<T>(stream, input, output, input_numel, threads, block_limit);
    }
```
**EN:** This section defines `MscclContextSelection`, `MscclContext`, `allreduce`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`MscclContextSelection`、`MscclContext`、`allreduce`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 27-50: Runtime integration and dispatch
```cpp
  }
};

using fptr_t = int64_t;
static_assert(sizeof(void*) == sizeof(fptr_t));

torch::Tensor _unique_id2tensor(const mscclpp::UniqueId& unique_id) {
  auto options = torch::TensorOptions().dtype(torch::kByte).device(torch::kCPU);
  auto tensor = torch::empty({static_cast<int64_t>(unique_id.size())}, options);
  std::memcpy(tensor.data_ptr<uint8_t>(), unique_id.data(), unique_id.size());
  return tensor;
}

// Function to convert vector of int32_t back to array of uint8_t
mscclpp::UniqueId _tensor2unique_id(const torch::Tensor& tensor) {
  mscclpp::UniqueId unique_id;
  std::memcpy(unique_id.data(), tensor.data_ptr<uint8_t>(), unique_id.size());
  return unique_id;
}

torch::Tensor mscclpp_generate_unique_id() {
  mscclpp::UniqueId unique_id = mscclpp::TcpBootstrap::createUniqueId();
  return _unique_id2tensor(unique_id);
}
```
**EN:** This section uses `_unique_id2tensor`, `_tensor2unique_id`, `mscclpp_generate_unique_id` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_unique_id2tensor`、`_tensor2unique_id`、`mscclpp_generate_unique_id`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 51-72: Runtime integration and dispatch
```cpp

fptr_t mscclpp_init_context(
    const torch::Tensor& unique_id,
    const int64_t rank,
    const int64_t world_size,
    torch::Tensor& scratch,
    torch::Tensor& put_buffer,
    const int64_t nranks_per_node,
    const std::vector<int64_t>& rank_to_node,
    const std::vector<int64_t>& rank_to_ib,
    const int64_t context_selection) {
  MscclContext* context_ptr = new MscclContext(static_cast<MscclContextSelection>(context_selection));
  mscclpp::UniqueId uid = _tensor2unique_id(unique_id);
  if (context_selection == MSCCL1NODELL) {
    void* scratch_ptr = reinterpret_cast<void*>(scratch.data_ptr());
    const size_t scratch_bytes = scratch.numel() * scratch.element_size();
    context_ptr->msccl_1nodeLL_context = std::make_shared<sglang::Msccl1NodeLLcontext>(
        uid, rank, world_size, scratch_ptr, scratch_bytes, nranks_per_node, rank_to_node, rank_to_ib);
  } else if (context_selection == MSCCL2NODELL) {
    void* scratch_ptr = reinterpret_cast<void*>(scratch.data_ptr());
    const size_t scratch_bytes = scratch.numel() * scratch.element_size();
    void* put_buffer_ptr = reinterpret_cast<void*>(put_buffer.data_ptr());
```
**EN:** This section uses `mscclpp_init_context`, `MscclContext`, `_tensor2unique_id` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`mscclpp_init_context`、`MscclContext`、`_tensor2unique_id`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 73-94: Runtime integration and dispatch
```cpp
    const size_t put_buffer_bytes = put_buffer.numel() * put_buffer.element_size();
    context_ptr->msccl_2nodeLL_context = std::make_shared<sglang::Msccl2NodeLLcontext>(
        uid,
        rank,
        world_size,
        scratch_ptr,
        scratch_bytes,
        put_buffer_ptr,
        put_buffer_bytes,
        nranks_per_node,
        rank_to_node,
        rank_to_ib);
  } else {
    throw std::runtime_error("invalid context selection");
  }
  return (fptr_t)context_ptr;
}

bool _mscclpp_is_weak_contiguous(torch::Tensor& t) {
  return t.is_contiguous() ||
         (t.storage().nbytes() - t.storage_offset() * t.element_size() == t.numel() * t.element_size());
}
```
**EN:** This section uses `_mscclpp_is_weak_contiguous`, `numel`, `runtime_error` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`_mscclpp_is_weak_contiguous`、`numel`、`runtime_error`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 95-114: Runtime integration and dispatch
```cpp
void mscclpp_allreduce(fptr_t _context, torch::Tensor& inp, torch::Tensor& out, int64_t nthreads, int64_t nblocks) {
  MscclContext* context = reinterpret_cast<MscclContext*>(_context);
  const at::cuda::OptionalCUDAGuard device_guard(device_of(inp));
  auto stream = c10::cuda::getCurrentCUDAStream().stream();

  TORCH_CHECK_EQ(inp.scalar_type(), out.scalar_type());
  TORCH_CHECK_EQ(inp.numel(), out.numel());
  TORCH_CHECK(_mscclpp_is_weak_contiguous(out));
  TORCH_CHECK(_mscclpp_is_weak_contiguous(inp));
  switch (out.scalar_type()) {
    case at::ScalarType::Float: {
      context->allreduce<float>(
          stream,
          reinterpret_cast<float*>(inp.data_ptr()),
          reinterpret_cast<float*>(out.data_ptr()),
          inp.numel(),
          nthreads,
          nblocks);
      break;
    }
```
**EN:** This section uses `mscclpp_allreduce`, `device_guard`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`mscclpp_allreduce`、`device_guard`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 115-135: Runtime integration and dispatch
```cpp
    case at::ScalarType::Half: {
      context->allreduce<half>(
          stream,
          reinterpret_cast<half*>(inp.data_ptr()),
          reinterpret_cast<half*>(out.data_ptr()),
          inp.numel(),
          nthreads,
          nblocks);
      break;
    }
#if (__CUDA_ARCH__ >= 800 || !defined(__CUDA_ARCH__))
    case at::ScalarType::BFloat16: {
      context->allreduce<__nv_bfloat16>(
          stream,
          reinterpret_cast<__nv_bfloat16*>(inp.data_ptr()),
          reinterpret_cast<__nv_bfloat16*>(out.data_ptr()),
          inp.numel(),
          nthreads,
          nblocks);
      break;
    }
```
**EN:** This section uses `data_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 136-140: Local implementation details
```cpp
#endif
    default:
      throw std::runtime_error("custom allreduce only supports float32, float16 and bfloat16");
  }
}
```
**EN:** This section fills in the local implementation details around `runtime_error`, completing the behavior required by the file.
**CN:** 本段补充了`runtime_error`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Collective communication / 集合通信**: Coordinates reductions across devices or ranks. / 在设备或 rank 之间协调归约。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `mscclpp_allreduce.cuh`
- **External headers / 外部头文件**: `c10/cuda/CUDAGuard.h`, `c10/cuda/CUDAStream.h`, `torch/all.h`, `torch/library.h`
- **Path context / 路径上下文**: allreduce / mscclpp_allreduce.cu
