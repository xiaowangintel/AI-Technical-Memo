# marlin.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/marlin.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Core Marlin CUDA utilities: constants, tiny vector wrappers, and `cp.async` helpers shared by kernels. / [CN] Marlin CUDA 核心公共工具：常量、小型向量封装，以及被各个内核复用的 `cp.async` 辅助函数。

## Line-by-Line Analysis / 逐行分析
### ABI-safe includes and namespace setup
```cpp
#ifndef TORCH_TARGET_VERSION
  #include <torch/all.h>
  #include <ATen/cuda/CUDAContext.h>
  #include <c10/cuda/CUDAGuard.h>
#endif
#ifndef MARLIN_NAMESPACE_NAME
  #define MARLIN_NAMESPACE_NAME marlin
#endif
namespace MARLIN_NAMESPACE_NAME {
```
**EN:** The file avoids hard-requiring Torch headers for stable-ABI builds. Only non-stable callers pull in Torch/CUDA runtime integration, while all builds still get the low-level CUDA helpers under a configurable namespace.
**CN:** 这个文件避免在稳定 ABI 构建中强制依赖 Torch 头文件。只有非稳定调用方才包含 Torch/CUDA 运行时集成，而所有构建都能在可配置命名空间下使用底层 CUDA 辅助函数。

### Global tuning constants
```cpp
static constexpr int default_threads = 256;
static constexpr int pipe_stages = 4;
static constexpr int min_thread_n = 64;
static constexpr int min_thread_k = 64;
static constexpr int max_thread_n = 256;
static constexpr int tile_size = 16;
static constexpr int max_par = 16;
static constexpr int repack_stages = 8;
static constexpr int repack_threads = 256;
static constexpr int tile_k_size = tile_size;
static constexpr int tile_n_size = tile_k_size * 4;
```
**EN:** These constants define the basic geometry used across Marlin: 16x16 tensor-core tiles, preferred 256-thread blocks, async pipeline depth, and the 16x64 repack tile used by AWQ/GPTQ preprocessing.
**CN:** 这些常量定义了 Marlin 的基础几何参数：16x16 Tensor Core tile、优选的 256 线程线程块、异步流水深度，以及 AWQ/GPTQ 预处理使用的 16x64 重排 tile。

### Tiny vector wrapper and helpers
```cpp
template <typename T, int n>
struct Vec {
  T elems[n];
  __device__ T& operator[](int i) { return elems[i]; }
};
using I4 = Vec<int, 4>;
constexpr int div_ceil(int a, int b) { return (a + b - 1) / b; }
```
**EN:** `Vec` is the lightweight fragment container reused throughout the codebase for MMA fragments, scales, zero points, and packed integers. `I4` mirrors one `int4` worth of integer payload, and `div_ceil` is used heavily for tile scheduling.
**CN:** `Vec` 是一个轻量片段容器，整个代码库都用它表示 MMA 片段、scale、zero point 和打包整数。`I4` 对应一个 `int4` 大小的整数载荷，`div_ceil` 则被广泛用于 tile 调度。

### Fallback async-copy emulation for pre-SM80
```cpp
#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ < 800
__device__ inline void cp_async4(void* smem_ptr, const void* glob_ptr) {
  reinterpret_cast<int4*>(smem_ptr)[0] =
      reinterpret_cast<const int4*>(glob_ptr)[0];
}
__device__ inline void cp_async_fence() {}
template <int n>
__device__ inline void cp_async_wait() {}
#endif
```
**EN:** Older GPUs do not support hardware `cp.async`, so Marlin degrades to normal global-to-shared copies and makes the fence/wait operations no-ops. This keeps the higher-level pipeline code architecture-independent.
**CN:** 较老的 GPU 不支持硬件 `cp.async`，因此 Marlin 退化为普通的 global-to-shared 拷贝，并把 fence/wait 变成空操作。这样上层流水线代码就能保持架构无关。

### Native PTX `cp.async` wrappers for Ampere+
```cpp
__device__ inline void cp_async4_pred(void* smem_ptr, const void* glob_ptr,
                                      bool pred = true) {
  const int BYTES = 16;
  uint32_t smem = static_cast<uint32_t>(__cvta_generic_to_shared(smem_ptr));
  asm volatile(
      "{
"
      "   .reg .pred p;
"
      "   setp.ne.b32 p, %0, 0;
"
      "   @p cp.async.cg.shared.global [%1], [%2], %3;
"
      "}
" ::"r"((int)pred), "r"(smem), "l"(glob_ptr), "n"(BYTES));
}
__device__ inline void cp_async_fence() {
  asm volatile("cp.async.commit_group;
" ::);
}
template <int n>
__device__ inline void cp_async_wait() {
  asm volatile("cp.async.wait_group %0;
" ::"n"(n));
}
```
**EN:** On Ampere and newer GPUs, Marlin exposes thin PTX wrappers for cache-all (`ca`) and cache-global (`cg`) asynchronous copies, plus explicit commit/wait group control. The rest of the kernel code builds a software pipeline on top of these primitives.
**CN:** 在 Ampere 及更新架构上，Marlin 提供了对 PTX `cp.async` 的薄封装，包括 cache-all (`ca`) 与 cache-global (`cg`) 异步拷贝，以及显式的 commit/wait group 控制。其余内核代码都基于这些原语构建软件流水线。

## Key Concepts / 关键概念
- `tile_size=16` is the fundamental Tensor Core block size. / `tile_size=16` 是 Tensor Core 的基础块大小。
- `cp_async*` hides architecture differences behind a uniform API. / `cp_async*` 用统一接口屏蔽了不同架构差异。
- `repack_*` constants are shared by AWQ/GPTQ weight layout transforms. / `repack_*` 常量被 AWQ/GPTQ 权重布局转换共用。

## Dependencies / 依赖关系
- Includes CUDA runtime headers and, conditionally, Torch CUDA headers. / 依赖 CUDA 运行时头文件，并按条件依赖 Torch CUDA 头文件。
- Used by `marlin_template.h`, `marlin_mma.h`, repack kernels, and dtype helpers. / 被 `marlin_template.h`、`marlin_mma.h`、重排内核和 dtype 辅助代码使用。
