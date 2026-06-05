# launch_bounds_utils.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/launch_bounds_utils.h`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Provides compile-time and runtime helpers for choosing a reasonable blocks-per-SM value for __launch_bounds__ and related occupancy tuning. **CN:** 提供编译期与运行期辅助工具，用于为 __launch_bounds__ 以及相关占用率调优选择合适的每个 SM block 数。

## Line-by-Line Analysis / 逐行分析

### Architecture-to-max-threads mapping / 架构到每个 SM 最大线程数的映射
```cpp
// maximum blocks per SM cap
#ifndef VLLM_LAUNCH_BLOCKS_CAP
  #define VLLM_LAUNCH_BLOCKS_CAP 4
#endif

// Compile-time estimate of max threads per SM for launch bounds.
// Families: 1024, 1536, 2048 threads/SM.
#ifndef VLLM_MAX_THREADS_PER_SM
  #ifdef __CUDA_ARCH__

    /* 1024 thr/SM: Turing (sm_75) */
    #if (__CUDA_ARCH__ == 750)
      #define VLLM_MAX_THREADS_PER_SM 1024

    /* 1536 thr/SM: Ampere GA10x (sm_86/87), Ada (sm_89),
        GB20x consumer (sm_120/121), Thor (sm_101 or sm_110) */
    #elif (__CUDA_ARCH__ == 860) || (__CUDA_ARCH__ == 870) || \
        (__CUDA_ARCH__ == 890) || (__CUDA_ARCH__ == 1010) ||  \
        (__CUDA_ARCH__ == 1100) || (__CUDA_ARCH__ == 1200) || \
        (__CUDA_ARCH__ == 1210)
      #define VLLM_MAX_THREADS_PER_SM 1536

    /* 2048 thr/SM: Volta (sm_70/72), Ampere GA100 (sm_80),
        Hopper (sm_90), Blackwell (sm_100/103) */
    #elif (__CUDA_ARCH__ == 700) || (__CUDA_ARCH__ == 720) || \
        (__CUDA_ARCH__ == 800) || (__CUDA_ARCH__ == 900) ||   \
        (__CUDA_ARCH__ == 1000) || (__CUDA_ARCH__ == 1030)
      #define VLLM_MAX_THREADS_PER_SM 2048

    /* Fallback: use 2048 for unknown future CCs */
    #else
      #define VLLM_MAX_THREADS_PER_SM 2048
    #endif

  #else
  /* Host pass (no __CUDA_ARCH__): neutral default */
    #define VLLM_MAX_THREADS_PER_SM 2048
  #endif
```
**EN:** The header groups GPU architectures by their typical max-threads-per-SM capacity and assigns a compile-time VLLM_MAX_THREADS_PER_SM constant. Unknown future architectures fall back to 2048 threads/SM.
**CN:** 该头文件按 GPU 架构的典型每个 SM 最大线程数进行分组，并给出编译期常量 VLLM_MAX_THREADS_PER_SM。未知的未来架构默认回退到 2048 threads/SM。

### Compile-time blocks-per-SM macros / 编译期 blocks-per-SM 宏
```cpp
// compute the number of blocks per SM to request in __launch_bounds__
#define VLLM_BLOCKS_DIV(VAL) (VLLM_MAX_THREADS_PER_SM / (VAL))
#define VLLM_CLAMP_BLOCKS_PER_SM(VAL) \
  (((VAL) <= 0)                       \
       ? 1                            \
       : (((VAL) < VLLM_LAUNCH_BLOCKS_CAP) ? (VAL) : VLLM_LAUNCH_BLOCKS_CAP))
#define VLLM_BLOCKS_PER_SM(BLOCK_THREADS) \
  VLLM_CLAMP_BLOCKS_PER_SM(VLLM_BLOCKS_DIV(BLOCK_THREADS))
```
**EN:** VLLM_BLOCKS_DIV, VLLM_CLAMP_BLOCKS_PER_SM, and VLLM_BLOCKS_PER_SM translate a block size into a clamped launch-bounds request, with a global cap defined by VLLM_LAUNCH_BLOCKS_CAP.
**CN:** VLLM_BLOCKS_DIV、VLLM_CLAMP_BLOCKS_PER_SM 和 VLLM_BLOCKS_PER_SM 会把 block size 转换成经过钳制的 launch-bounds 请求，其全局上限由 VLLM_LAUNCH_BLOCKS_CAP 决定。

### Runtime occupancy helper / 运行期占用率辅助函数
```cpp
// runtime-time helper to compute blocks/SM
static inline int vllm_runtime_blocks_per_sm(int block_threads) {
  int device = -1;
  cudaGetDevice(&device);
  int max_threads_per_sm = VLLM_MAX_THREADS_PER_SM;
  cudaDeviceGetAttribute(&max_threads_per_sm,
                         cudaDevAttrMaxThreadsPerMultiProcessor, device);
  int blocks = (block_threads > 0) ? (max_threads_per_sm / block_threads) : 1;
  return VLLM_CLAMP_BLOCKS_PER_SM(blocks);
}
```
**EN:** vllm_runtime_blocks_per_sm queries cudaDevAttrMaxThreadsPerMultiProcessor at runtime and applies the same clamping rule, which is useful when kernel launch policy needs the actual device limit rather than the compile-time estimate.
**CN:** vllm_runtime_blocks_per_sm 会在运行时查询 cudaDevAttrMaxThreadsPerMultiProcessor，并应用相同的钳制规则；当内核启动策略需要真实设备限制而不是编译期估计时，这个函数就很有用。

## Key Concepts / 关键概念

- **EN:** launch_bounds tuning is a balance between occupancy and per-thread resource usage.
  **CN:** launch_bounds 调优本质上是在占用率与单线程资源使用之间做平衡。
- **EN:** The file exposes both compile-time macros and a runtime query helper because different call sites need different trade-offs.
  **CN:** 该文件同时提供编译期宏和运行期查询函数，因为不同调用点需要不同的权衡方式。

## Dependencies / 依赖关系

- **EN:** Depends only on cuda_runtime_api.h and the standard algorithm header.
  **CN:** 仅依赖 cuda_runtime_api.h 与标准库 algorithm 头文件。
- **EN:** Included by kernels that want a consistent blocks-per-SM policy across architectures.
  **CN:** 被希望在不同架构上统一 blocks-per-SM 策略的内核文件包含。
