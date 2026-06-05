# custom_quickreduce.cu — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/custom_quickreduce.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Exposes ROCm-only quickreduce primitives to Torch/Python, wrapping communicator setup, IPC handle exchange, and dtype-specific all-reduce dispatch. **CN:** 把仅在 ROCm 上可用的 quickreduce 原语暴露给 Torch/Python，封装通信器初始化、IPC 句柄交换与按 dtype 分发的 all-reduce。

## Line-by-Line Analysis / 逐行分析

### DeviceComms lifecycle and IPC handle exchange / DeviceComms 生命周期与 IPC 句柄交换
```cpp
quickreduce::fptr_t init_custom_qr(int64_t rank, int64_t world_size,
                                   std::optional<int64_t> qr_max_size) {
  if (world_size > 8)
    throw std::invalid_argument("world size > 8 is not supported");
  if (world_size == 6)
    throw std::invalid_argument("world size == 6 is not supported");
  if (world_size % 2 != 0)
    throw std::invalid_argument("Odd num gpus is not supported for now");
  if (rank < 0 || rank >= world_size)
    throw std::invalid_argument("invalid rank passed in");
  quickreduce::DeviceComms* fptr = new quickreduce::DeviceComms();
  fptr->init(world_size, rank, qr_max_size);
  return (quickreduce::fptr_t)fptr;
}

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
  auto options =
      torch::TensorOptions().dtype(torch::kUInt8).device(torch::kCPU);
  auto data_handle =
      torch::empty({static_cast<int64_t>(sizeof(hipIpcMemHandle_t))}, options);
  std::memcpy(data_handle.data_ptr(), &handle, sizeof(hipIpcMemHandle_t));
  return data_handle;
}

void qr_open_handles(quickreduce::fptr_t _fa,
                     const std::vector<torch::Tensor>& handles) {
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
```
**EN:** The file is compiled only under USE_ROCM. It validates supported world sizes, constructs quickreduce::DeviceComms, exports its IPC handle as a CPU uint8 tensor, and reopens peer handles supplied by Python.
**CN:** 该文件只在 USE_ROCM 下编译。它会校验支持的 world size，构造 quickreduce::DeviceComms，把 IPC handle 导出为 CPU uint8 张量，并重新打开由 Python 传入的对端句柄。

### ROCm quick all-reduce dispatch / ROCm quick all-reduce 分发
```cpp
void qr_all_reduce(quickreduce::fptr_t _fa, torch::Tensor& inp,
                   torch::Tensor& out, int64_t quant_level, bool cast_bf2half) {
  auto fa = reinterpret_cast<quickreduce::DeviceComms*>(_fa);
  const at::cuda::OptionalCUDAGuard device_guard(device_of(inp));
  auto stream = at::cuda::getCurrentHIPStreamMasqueradingAsCUDA();

  TORCH_CHECK_EQ(inp.scalar_type(), out.scalar_type());
  TORCH_CHECK_EQ(inp.numel(), out.numel());
  TORCH_CHECK_LE(out.numel(), fa->kMaxProblemSize);
  if (out.scalar_type() == at::ScalarType::Half) {
    fa->allreduce<half, false>(reinterpret_cast<half*>(inp.data_ptr()),
                               reinterpret_cast<half*>(out.data_ptr()),
                               out.numel(), quant_level, stream);
  } else if (out.scalar_type() == at::ScalarType::BFloat16) {
    if (cast_bf2half) {
      fa->allreduce<half, true>(reinterpret_cast<half*>(inp.data_ptr()),
                                reinterpret_cast<half*>(out.data_ptr()),
                                out.numel(), quant_level, stream);
    } else {
      fa->allreduce<quickreduce::nv_bfloat16, false>(
          reinterpret_cast<quickreduce::nv_bfloat16*>(inp.data_ptr()),
          reinterpret_cast<quickreduce::nv_bfloat16*>(out.data_ptr()),
          out.numel(), quant_level, stream);
    }
  } else {
    throw std::runtime_error(
        "quick allreduce only supports float16 and bfloat16");
  }
}
```
**EN:** qr_all_reduce chooses the current HIP stream, validates tensor compatibility, checks the problem-size cap, and dispatches to quickreduce templates for half or bf16. The cast_bf2half flag selects whether bf16 inputs are reduced through a half-based codec path.
**CN:** qr_all_reduce 会选择当前 HIP stream，校验张量兼容性，检查问题规模上限，并把计算分发到 half 或 bf16 的 quickreduce 模板。cast_bf2half 标志控制 bf16 输入是否走基于 half 的 codec 路径。

### Maximum size and explicit template instantiation / 最大规模与显式模板实例化
```cpp
int64_t qr_max_size() {
  // The default is 2GB (2,147,483,648 bytes)
  return static_cast<int64_t>(std::numeric_limits<int32_t>::max()) + 1;
}

  #define INSTANTIATE_FOR_WORLDSIZE(T, Codec, cast_bf2half)       \
    template struct quickreduce::AllReduceTwoshot<T, Codec<T, 2>, \
                                                  cast_bf2half>;  \
    template struct quickreduce::AllReduceTwoshot<T, Codec<T, 4>, \
                                                  cast_bf2half>;  \
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
**EN:** qr_max_size publishes the default 2GB limit, and the INSTANTIATE_FOR_WORLDSIZE macro eagerly instantiates supported codec/world-size combinations so the extension exposes all required kernel variants at link time.
**CN:** qr_max_size 暴露默认的 2GB 上限，而 INSTANTIATE_FOR_WORLDSIZE 宏会显式实例化支持的 codec/world-size 组合，确保扩展在链接时就具备所需的全部内核变体。

## Key Concepts / 关键概念

- **EN:** This file is ROCm-specific; there is no CUDA-side implementation here.
  **CN:** 该文件是 ROCm 专用路径，这里没有对应的 CUDA 实现。
- **EN:** DeviceComms owns the communication substrate, while the wrapper handles Torch tensor marshalling.
  **CN:** DeviceComms 负责通信底座，而包装层负责 Torch 张量编组。
- **EN:** Explicit instantiation prevents missing kernel variants for different codecs and world sizes.
  **CN:** 显式模板实例化可避免不同 codec 与 world size 组合缺少内核符号。

## Dependencies / 依赖关系

- **EN:** Depends on quickreduce/quick_reduce.h under USE_ROCM.
  **CN:** 在 USE_ROCM 下依赖 quickreduce/quick_reduce.h。
- **EN:** Uses HIP IPC handle types and ROCm stream helpers.
  **CN:** 依赖 HIP IPC 句柄类型与 ROCm stream 辅助函数。
- **EN:** Torch tensors provide the Python-facing handle and data transport format.
  **CN:** Torch 张量承担面向 Python 的句柄与数据传输格式。
