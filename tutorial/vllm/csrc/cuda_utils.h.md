# cuda_utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cuda_utils.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Collects lightweight CUDA utility macros and declarations shared by multiple extension translation units. / [CN] 汇总多个扩展编译单元共享的轻量级 CUDA 工具宏与函数声明。

## Line-by-Line Analysis / 逐行分析
### Inline attribute macros / 内联属性宏
```cpp
#if defined(__HIPCC__)
  #define HOST_DEVICE_INLINE __host__ __device__
  #define DEVICE_INLINE __device__
  #define HOST_INLINE __host__
#elif defined(__CUDACC__) || defined(_NVHPC_CUDA)
  #define HOST_DEVICE_INLINE __host__ __device__ __forceinline__
  #define DEVICE_INLINE __device__ __forceinline__
  #define HOST_INLINE __host__ __forceinline__
#else
  #define HOST_DEVICE_INLINE inline
  #define DEVICE_INLINE inline
  #define HOST_INLINE inline
#endif
```
**EN:** These macros normalize `__host__`, `__device__`, and force-inline annotations across HIP, CUDA, and plain C++ compilation so helper templates can be included in both host and device code.
**CN:** 这些宏在 HIP、CUDA 和普通 C++ 编译环境之间统一 `__host__`、`__device__` 与强制内联标注，使辅助模板既能用于主机端，也能用于设备端代码。

### Runtime error checking / 运行时错误检查
```cpp
#define CUDA_CHECK(cmd)                                             \
  do {                                                              \
    cudaError_t e = cmd;                                            \
    if (e != cudaSuccess) {                                         \
      printf("Failed: Cuda error %s:%d '%s'\n", __FILE__, __LINE__, \
             cudaGetErrorString(e));                                \
      exit(EXIT_FAILURE);                                           \
    }                                                               \
  } while (0)
```
**EN:** `CUDA_CHECK` is the extension’s basic runtime guard: execute a CUDA API call, print file/line context on failure, and terminate immediately.
**CN:** `CUDA_CHECK` 是扩展里最基础的运行时保护：执行 CUDA API，若失败则打印文件/行号上下文并立即终止。

### Small math helper / 小型数学辅助函数
```cpp
int64_t get_device_attribute(int64_t attribute, int64_t device_id);

int64_t get_max_shared_memory_per_block_device_attribute(int64_t device_id);

namespace cuda_utils {

template <typename T>
HOST_DEVICE_INLINE constexpr std::enable_if_t<std::is_integral_v<T>, T>
ceil_div(T a, T b) {
  return (a + b - 1) / b;
}

};  // namespace cuda_utils
```
**EN:** Besides exposing device-attribute queries, the header defines an integer-only `ceil_div`, which is used repeatedly when kernels convert problem sizes into grid or tile counts.
**CN:** 除了导出设备属性查询函数之外，这个头文件还定义了只适用于整数的 `ceil_div`，用于把问题规模转换为 grid 或 tile 数量。

## Key Concepts / 关键概念
- **EN:** The utilities here are intentionally minimal and ubiquitous.
  **CN:** 这里的工具刻意保持轻量且通用。
- **EN:** `ceil_div` is small but central because CUDA launch geometry often depends on rounded-up integer division.
  **CN:** `ceil_div` 虽小却很核心，因为 CUDA 启动几何经常依赖向上取整的整数除法。

## Dependencies / 依赖关系
- **EN:** The header is used by many kernels and paired with `cuda_utils_kernels.cu` for the actual runtime query implementations.
  **CN:** 该头文件被很多内核使用，并与 `cuda_utils_kernels.cu` 搭配提供真正的运行时查询实现。
- **EN:** It assumes CUDA/HIP runtime symbols such as `cudaError_t` and `cudaDeviceGetAttribute` are available to consumers.
  **CN:** 它默认消费者环境中可用 `cudaError_t`、`cudaDeviceGetAttribute` 等 CUDA/HIP runtime 符号。
