# custom_all_reduce.cu — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/custom_all_reduce.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Provides the Torch/Python-facing wrapper around vLLM's CustomAllreduce object, including tensor validation, buffer registration, and IPC-handle helpers. **CN:** 为 vLLM 的 CustomAllreduce 对象提供面向 Torch/Python 的包装层，包括张量校验、缓冲区注册和 IPC 句柄辅助函数。

## Line-by-Line Analysis / 逐行分析

### Constructor bridge from Python handles / 从 Python 句柄构造对象的桥接层
```cpp
fptr_t init_custom_ar(const std::vector<fptr_t>& fake_ipc_ptrs,
                      torch::Tensor& rank_data, int64_t rank,
                      bool fully_connected) {
  int world_size = fake_ipc_ptrs.size();
  if (world_size > 8)
    throw std::invalid_argument("world size > 8 is not supported");
  if (world_size % 2 != 0)
    throw std::invalid_argument("Odd num gpus is not supported for now");
  if (rank < 0 || rank >= world_size)
    throw std::invalid_argument("invalid rank passed in");

  vllm::Signal* ipc_ptrs[8];
  for (int i = 0; i < world_size; i++) {
    ipc_ptrs[i] = reinterpret_cast<vllm::Signal*>(fake_ipc_ptrs[i]);
  }
  return (fptr_t) new vllm::CustomAllreduce(ipc_ptrs, rank_data.data_ptr(),
                                            rank_data.numel(), rank, world_size,
                                            fully_connected);
}
```
**EN:** init_custom_ar converts int64 fake pointers received from Python back into Signal pointers, validates world size and rank constraints, and constructs a CustomAllreduce instance on the heap.
**CN:** init_custom_ar 把从 Python 传来的 int64 伪指针还原为 Signal 指针，校验 world size 与 rank 约束，然后在堆上创建一个 CustomAllreduce 实例。

### Weak contiguity check / 弱连续性检查
```cpp
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
         (t.storage().nbytes() - t.storage_offset() * t.element_size() ==
          t.numel() * t.element_size());
}
```
**EN:** The kernels flatten tensors without passing stride metadata, so the wrapper accepts either true contiguous tensors or views whose storage range still forms one dense byte span. This is slightly weaker than is_contiguous() but still safe for flat indexing.
**CN:** 内核把输入视为扁平数组处理，却不会传递 stride 元数据，因此包装层接受两类张量：真正连续的张量，以及虽然不是严格 contiguous 但底层存储区间仍构成单一密集字节段的视图。这个条件比 is_contiguous() 略宽，但对扁平索引仍然安全。

### Type-dispatched all_reduce wrapper / 按类型分发的 all_reduce 包装
```cpp
void all_reduce(fptr_t _fa, torch::Tensor& inp, torch::Tensor& out,
                fptr_t _reg_buffer, int64_t reg_buffer_sz_bytes) {
  auto fa = reinterpret_cast<vllm::CustomAllreduce*>(_fa);
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
    AT_CUDA_CHECK(cudaMemcpyAsync(reg_buffer, inp.data_ptr(), input_size,
                                  cudaMemcpyDeviceToDevice, stream));
  } else {
    reg_buffer = inp.data_ptr();
  }
  switch (out.scalar_type()) {
    case at::ScalarType::Float: {
      fa->allreduce<float>(stream, reinterpret_cast<float*>(reg_buffer),
                           reinterpret_cast<float*>(out.data_ptr()),
                           out.numel());
      break;
    }
    case at::ScalarType::Half: {
      fa->allreduce<half>(stream, reinterpret_cast<half*>(reg_buffer),
                          reinterpret_cast<half*>(out.data_ptr()), out.numel());
      break;
    }
#if (__CUDA_ARCH__ >= 800 || !defined(__CUDA_ARCH__))
    case at::ScalarType::BFloat16: {
      fa->allreduce<nv_bfloat16>(
          stream, reinterpret_cast<nv_bfloat16*>(reg_buffer),
          reinterpret_cast<nv_bfloat16*>(out.data_ptr()), out.numel());
      break;
    }
#endif
    default:
      throw std::runtime_error(
          "custom allreduce only supports float32, float16 and bfloat16");
  }
}
```
**EN:** The wrapper chooses the current CUDA stream, optionally copies input into a pre-registered buffer, checks shape/type compatibility, and dispatches to CustomAllreduce::allreduce for float, half, or bf16. Unsupported dtypes fail early with a runtime error.
**CN:** 该包装函数选取当前 CUDA 流，可选地先把输入复制到预注册缓冲区，检查形状与类型一致性，然后按 float、half 或 bf16 调用 CustomAllreduce::allreduce。对于不支持的数据类型会直接报错。

### Registration and graph IPC metadata / 注册流程与图捕获 IPC 元数据
```cpp
void dispose(fptr_t _fa) {
  delete reinterpret_cast<vllm::CustomAllreduce*>(_fa);
}

int64_t meta_size() { return sizeof(vllm::Signal); }

void register_buffer(fptr_t _fa, const std::vector<fptr_t>& fake_ipc_ptrs) {
  auto fa = reinterpret_cast<vllm::CustomAllreduce*>(_fa);
  TORCH_CHECK(fake_ipc_ptrs.size() == fa->world_size_);
  void* ipc_ptrs[8];
  for (int i = 0; i < fake_ipc_ptrs.size(); i++) {
    ipc_ptrs[i] = reinterpret_cast<void*>(fake_ipc_ptrs[i]);
  }
  fa->register_buffer(ipc_ptrs);
}

// Use vector<int64_t> to represent byte data for python binding compatibility.
std::tuple<std::vector<int64_t>, std::vector<int64_t>>
get_graph_buffer_ipc_meta(fptr_t _fa) {
  auto fa = reinterpret_cast<vllm::CustomAllreduce*>(_fa);
  auto [handle, offsets] = fa->get_graph_buffer_ipc_meta();
  std::vector<int64_t> bytes(handle.begin(), handle.end());
  return std::make_tuple(bytes, offsets);
}

// Use vector<int64_t> to represent byte data for python binding compatibility.
void register_graph_buffers(fptr_t _fa,
                            const std::vector<std::vector<int64_t>>& handles,
                            const std::vector<std::vector<int64_t>>& offsets) {
  auto fa = reinterpret_cast<vllm::CustomAllreduce*>(_fa);
  std::vector<std::string> bytes;
  bytes.reserve(handles.size());
  for (int i = 0; i < handles.size(); i++) {
    bytes.emplace_back(handles[i].begin(), handles[i].end());
  }
  bytes.reserve(handles.size());
  fa->register_graph_buffers(bytes, offsets);
}
```
**EN:** These helpers expose register_buffer, get_graph_buffer_ipc_meta, and register_graph_buffers to Python. They convert raw addresses and byte arrays into the C++ formats expected by the CustomAllreduce object during normal execution and CUDA graph replay.
**CN:** 这些辅助函数把 register_buffer、get_graph_buffer_ipc_meta 和 register_graph_buffers 暴露给 Python。它们负责把原始地址和字节数组转换成 CustomAllreduce 在常规执行与 CUDA Graph 回放阶段所需的 C++ 格式。

### Shared buffer IPC helpers / 共享缓冲区 IPC 辅助函数
```cpp
std::tuple<fptr_t, torch::Tensor> allocate_shared_buffer_and_handle(
    int64_t size) {
  auto device_index = c10::cuda::current_device();
  at::DeviceGuard device_guard(at::Device(at::DeviceType::CUDA, device_index));
  void* buffer;
  cudaStreamCaptureMode mode = cudaStreamCaptureModeRelaxed;
  auto stream = c10::cuda::getCurrentCUDAStream().stream();
  AT_CUDA_CHECK(cudaThreadExchangeStreamCaptureMode(&mode));

  // Allocate buffer
#if defined(USE_ROCM)
  // data buffers need to be "uncached" for signal on MI200
  AT_CUDA_CHECK(
      hipExtMallocWithFlags((void**)&buffer, size, hipDeviceMallocUncached));
#else
  AT_CUDA_CHECK(cudaMalloc((void**)&buffer, size));
#endif
  AT_CUDA_CHECK(cudaMemsetAsync(buffer, 0, size, stream));
  AT_CUDA_CHECK(cudaStreamSynchronize(stream));
  AT_CUDA_CHECK(cudaThreadExchangeStreamCaptureMode(&mode));

  // Create IPC memhandle for the allocated buffer.
  // Will use it in open_mem_handle.
  auto options =
      torch::TensorOptions().dtype(torch::kUInt8).device(torch::kCPU);
  auto handle =
      torch::empty({static_cast<int64_t>(sizeof(cudaIpcMemHandle_t))}, options);
  AT_CUDA_CHECK(
      cudaIpcGetMemHandle((cudaIpcMemHandle_t*)handle.data_ptr(), buffer));

  return std::make_tuple(reinterpret_cast<fptr_t>(buffer), handle);
}

fptr_t open_mem_handle(torch::Tensor& mem_handle) {
  void* ipc_ptr;
  AT_CUDA_CHECK(cudaIpcOpenMemHandle(
      (void**)&ipc_ptr, *((const cudaIpcMemHandle_t*)mem_handle.data_ptr()),
      cudaIpcMemLazyEnablePeerAccess));
  return reinterpret_cast<fptr_t>(ipc_ptr);
}

void free_shared_buffer(fptr_t buffer) {
  AT_CUDA_CHECK(cudaFree(reinterpret_cast<void*>(buffer)));
}
```
**EN:** The final group allocates a device buffer, zeros it, exports a cudaIpcMemHandle_t as a CPU tensor, opens peer handles lazily, and frees shared buffers when they are no longer needed. This is the transport layer used by Python-side setup code.
**CN:** 最后一组函数负责分配设备缓冲区、清零、把 cudaIpcMemHandle_t 导出为 CPU 张量、按需打开对端句柄，并在不再需要时释放共享缓冲区。这是 Python 侧初始化流程使用的传输层。

## Key Concepts / 关键概念

- **EN:** Python bindings pass device pointers as int64 placeholders because that is easier to marshal through Torch bindings.
  **CN:** Python 绑定把设备指针包装成 int64 占位值传递，因为这样更容易通过 Torch 绑定层进行编组。
- **EN:** The wrapper decouples tensor validation and IPC setup from the lower-level reduction kernels.
  **CN:** 该包装层把张量校验与 IPC 准备逻辑，从更底层的归约内核中分离出来。
- **EN:** CUDA graph support requires stable kernel arguments, so peer pointer metadata is managed explicitly.
  **CN:** 为了支持 CUDA Graph，内核参数必须稳定，因此这里显式管理了对端指针元数据。

## Dependencies / 依赖关系

- **EN:** Directly depends on the CustomAllreduce implementation in csrc/custom_all_reduce.cuh.
  **CN:** 直接依赖 csrc/custom_all_reduce.cuh 中的 CustomAllreduce 实现。
- **EN:** Uses ATen/C10 CUDA guards and stream helpers to run on the correct device/stream.
  **CN:** 依赖 ATen/C10 的 CUDA guard 与 stream helper，以确保在正确设备和流上执行。
- **EN:** Exposes buffer/handle helpers used by Python distributed setup code.
  **CN:** 暴露给 Python 分布式初始化代码使用的缓冲区/句柄辅助函数。
