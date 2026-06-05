# topk.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/topk.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Provides the PyTorch-facing launcher for vLLM’s persistent top-k kernel, including runtime policy decisions, shared-memory sizing, occupancy checks, and safe fallbacks. / [CN] 提供面向 PyTorch 的 persistent top-k 启动器，负责运行时策略选择、共享内存大小计算、occupancy 检查以及安全回退。

## Line-by-Line Analysis / 逐行分析

### [High-level launcher decision / 顶层启动策略]
```cpp
template <int TopK>
void launch_persistent_topk(const torch::Tensor& logits,
                            const torch::Tensor& lengths,
                            torch::Tensor& output,
                            torch::Tensor& workspace,
                            int64_t max_seq_len) {
  ...
  if (num_rows > 32 && max_smem_per_block >= 128 * 1024) {
    cudaError_t status =
        vllm::FilteredTopKRaggedTransform<float, int32_t, TopK>(...);
    TORCH_CHECK(status == cudaSuccess, ...);
  } else {
    ...
  }
}
```
**EN:** The launcher first checks whether the batch is large enough and the device supports enough shared memory to use the simpler `FilteredTopKRaggedTransform` path from `persistent_topk.cuh`. Otherwise it stays on the persistent kernel path, which is better suited to small batches and mixed row lengths.  
**CN:** 启动器首先判断 batch 是否足够大、设备是否支持足够的共享内存，以便直接使用 `persistent_topk.cuh` 中更简单的 `FilteredTopKRaggedTransform` 路径。否则就走 persistent kernel 路径，该路径更适合小 batch 和混合长度的行。

### [Chunk sizing and occupancy guard / 分块大小与 occupancy 保护]
```cpp
uint32_t vec_size = 1;
if (stride % 4 == 0) vec_size = 4;
else if (stride % 2 == 0) vec_size = 2;
...
cudaOccupancyMaxActiveBlocksPerMultiprocessor(
    &occupancy, P::persistent_topk_kernel<TopK, 4>,
    P::kThreadsPerBlock, smem_size);
...
const bool needs_cooperative =
    static_cast<uint32_t>(max_seq_len) > P::RADIX_THRESHOLD;
```
**EN:** When using the persistent kernel, the host code derives a vector width from the row stride, computes a chunk size that fits in dynamic shared memory, and queries occupancy for the exact kernel instantiation that will be launched. This matters because the large-row path uses an inter-CTA spin barrier; overestimating residency could deadlock the kernel.  
**CN:** 在使用 persistent kernel 时，主机侧代码会根据行步幅推导向量宽度，计算能够塞进动态共享内存的 chunk 大小，并针对即将发射的具体模板实例查询 occupancy。之所以这样做，是因为大序列路径依赖 CTA 间自旋屏障；如果对可驻留块数估计过高，就可能导致内核死锁。

### [Cooperative launch capacity and fallback / 协作启动容量与回退]
```cpp
const uint32_t hw_resident_cap =
    static_cast<uint32_t>(num_sms) * static_cast<uint32_t>(occupancy);
...
uint32_t total_ctas = num_groups * ctas_per_group;
if (needs_cooperative && total_ctas > hw_resident_cap) {
  TORCH_CHECK(max_smem_per_block >= 128 * 1024, ...);
  cudaError_t status =
      vllm::FilteredTopKRaggedTransform<float, int32_t, TopK>(...);
  ...
  return;
}
```
**EN:** The launcher explicitly models whether the cooperative radix path can fit on hardware without oversubscription. If the planned CTA groups would exceed the resident capacity, it refuses to risk a deadlock and falls back to the one-block-per-row filtered kernel instead. This policy layer is a major part of the file’s value: it protects a fast but synchronization-sensitive kernel with runtime safety checks.  
**CN:** 启动器会显式评估协作 radix 路径是否能够在硬件上安全驻留而不发生超量订阅。如果计划中的 CTA 组数量会超过可驻留容量，它不会冒险让内核死锁，而是回退到“每行一个 block”的 filtered 内核。这一策略层是本文件的重要价值所在：它用运行时安全检查保护了一个高性能但同步敏感的内核。

### [Workspace reset and actual launch / Workspace 清零与实际发射]
```cpp
size_t state_bytes = num_groups * sizeof(P::RadixRowState);
TORCH_CHECK(workspace.size(0) >= static_cast<int64_t>(state_bytes), ...);
{
  cudaError_t mz_err =
      cudaMemsetAsync(workspace.data_ptr<uint8_t>(), 0, state_bytes, stream);
  TORCH_CHECK(mz_err == cudaSuccess, ...);
}
...
auto kernel = &P::persistent_topk_kernel<TOPK_VAL, VS>;
cudaFuncSetAttribute(kernel,
    cudaFuncAttributeMaxDynamicSharedMemorySize, smem_size);
kernel<<<total_ctas, P::kThreadsPerBlock, smem_size, stream>>>(params);
```
**EN:** Before launching, the code zeroes the `RadixRowState` workspace on the stream. The long comment above this block explains why the memset is unconditional: CUDA graph capture must record the reset even if the capture-time batch does not take the cooperative path, otherwise stale counters could corrupt later replays. After that, it sets the dynamic shared-memory attribute and launches the specialized kernel.  
**CN:** 在真正发射之前，代码会在当前 stream 上把 `RadixRowState` workspace 清零。该块上方的长注释解释了为什么这个 memset 必须无条件执行：即便在图捕获时当前 batch 没有走协作路径，CUDA graph 也必须记录这次清零，否则旧计数器会污染后续 replay。完成清零后，代码设置动态共享内存属性并启动相应模板实例的内核。

### [Public wrapper and K dispatch / 对外封装与 K 分发]
```cpp
void persistent_topk(const torch::Tensor& logits,
                     const torch::Tensor& lengths,
                     torch::Tensor& output,
                     torch::Tensor& workspace,
                     int64_t k, int64_t max_seq_len) {
  TORCH_CHECK(logits.dtype() == torch::kFloat32, ...);
  TORCH_CHECK(k == 512 || k == 1024 || k == 2048, ...);
  if (k == 512) {
    launch_persistent_topk<512>(...);
  } else if (k == 1024) {
    launch_persistent_topk<1024>(...);
  } else {
    launch_persistent_topk<2048>(...);
  }
}
```
**EN:** The exported function mostly enforces the contract: CUDA tensors only, `float32` logits, `int32` lengths and output, compatible dimensions, and a restricted set of supported `k` values. It then dispatches to a compile-time specialization so the underlying kernels can hard-code `TopK` and optimize shared-memory layout around that constant.  
**CN:** 对外导出的函数主要负责约束接口：只接受 CUDA 张量、`float32` logits、`int32` 长度和输出、匹配的维度，以及受限的 `k` 值集合。之后它分发到编译期特化版本，使底层内核能够把 `TopK` 视为常量，并据此优化共享内存布局。

## Key Concepts / 关键概念
- **EN:** This file is a runtime policy layer, not the top-k algorithm itself. The actual kernels live in `persistent_topk.cuh`.  
  **CN:** 这个文件是运行时策略层，而不是 top-k 算法本体；真正的内核实现在 `persistent_topk.cuh` 中。
- **EN:** Correct occupancy estimation is essential because the cooperative path contains a global spin barrier.  
  **CN:** 正确的 occupancy 估计至关重要，因为协作路径内部包含全局自旋屏障。
- **EN:** The unconditional workspace memset is a correctness fix for CUDA graph replay as well as normal launches.  
  **CN:** 无条件的 workspace memset 不仅服务普通启动，也是针对 CUDA graph replay 的正确性修复。

## Dependencies / 依赖关系
- **EN:** Depends directly on `persistent_topk.cuh` for constants, workspace layout, and all kernel implementations.  
  **CN:** 直接依赖 `persistent_topk.cuh` 中的常量、workspace 布局以及全部内核实现。
- **EN:** Uses ATen CUDA stream helpers and CUDA runtime occupancy / attribute APIs.  
  **CN:** 使用 ATen 的 CUDA stream 辅助接口，以及 CUDA runtime 的 occupancy / attribute API。
- **EN:** `torch_bindings.cpp` registers `persistent_topk` as a PyTorch custom op.  
  **CN:** `torch_bindings.cpp` 会把 `persistent_topk` 注册为 PyTorch 自定义算子。
