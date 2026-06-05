# cuda_compat.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cuda_compat.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides CUDA/ROCm portability wrappers for warp size, cached loads, shuffle intrinsics, and function attributes. / [CN] 提供 warp 大小、缓存读取、shuffle intrinsic 与函数属性等方面的 CUDA/ROCm 可移植性封装。

## Line-by-Line Analysis / 逐行分析
### Warp-size abstraction / Warp 大小抽象
```cpp
#ifdef USE_ROCM
struct Utils {
  static __host__ int get_warp_size() {
    static bool is_cached = false;
    static int result;

    if (!is_cached) {
      int device_id;
      cudaDeviceProp deviceProp;
      cudaGetDevice(&device_id);
      cudaGetDeviceProperties(&deviceProp, device_id);

      result = deviceProp.warpSize;
      is_cached = true;
    }

    return result;
  }

  static __device__ constexpr int get_warp_size() {
  #ifdef __GFX9__
    return 64;
  #else
    return 32;
  #endif
  }
};

  #define WARP_SIZE Utils::get_warp_size()
#else
  #define WARP_SIZE 32
#endif
```
**EN:** ROCm hardware may expose warp sizes of 32 or 64, so the file defines a `Utils` helper that can query the host-side value once and provide a compile-time device-side constant for gfx9. CUDA simply hardcodes `WARP_SIZE` to 32.
**CN:** ROCm 硬件可能使用 32 或 64 的 warp 大小，因此这里定义了 `Utils`：主机侧只查询一次实际值，设备侧则为 gfx9 提供编译期常量。CUDA 路径则直接把 `WARP_SIZE` 固定为 32。

### Load and shuffle wrappers / 读取与 shuffle 封装
```cpp
#ifndef USE_ROCM
  #define VLLM_LDG(arg) __ldg(arg)
#else
  #define VLLM_LDG(arg) *(arg)
#endif

#ifndef USE_ROCM
  #define VLLM_SHFL_XOR_SYNC(var, lane_mask) \
    __shfl_xor_sync(uint32_t(-1), var, lane_mask)
  #define VLLM_SHFL_XOR_SYNC_WIDTH(var, lane_mask, width) \
    __shfl_xor_sync(uint32_t(-1), var, lane_mask, width)
#else
  #define VLLM_SHFL_XOR_SYNC(var, lane_mask) __shfl_xor(var, lane_mask)
  #define VLLM_SHFL_XOR_SYNC_WIDTH(var, lane_mask, width) \
    __shfl_xor(var, lane_mask, width)
#endif

#ifndef USE_ROCM
  #define VLLM_SHFL_SYNC(var, src_lane) __shfl_sync(uint32_t(-1), var, src_lane)
#else
  #define VLLM_SHFL_SYNC(var, src_lane) __shfl(var, src_lane)
#endif

#ifndef USE_ROCM
  #define VLLM_SHFL_DOWN_SYNC(var, lane_delta) \
    __shfl_down_sync(uint32_t(-1), var, lane_delta)
#else
  #define VLLM_SHFL_DOWN_SYNC(var, lane_delta) __shfl_down(var, lane_delta)
#endif
```
**EN:** `VLLM_LDG` maps to `__ldg` on CUDA but becomes a plain dereference on ROCm. The shuffle macros similarly choose between CUDA’s sync variants and ROCm’s legacy forms while preserving one common call site.
**CN:** `VLLM_LDG` 在 CUDA 上映射到 `__ldg`，而在 ROCm 上退化为普通解引用。shuffle 宏也会在 CUDA 的 sync 版本与 ROCm 的传统形式之间切换，但保持统一调用接口。

### Function attribute wrapper / 函数属性封装
```cpp
#ifndef USE_ROCM
  #define VLLM_DevFuncAttribute_SET_MaxDynamicSharedMemorySize(FUNC, VAL) \
    cudaFuncSetAttribute(FUNC, cudaFuncAttributeMaxDynamicSharedMemorySize, VAL)
#else
  #define VLLM_DevFuncAttribute_SET_MaxDynamicSharedMemorySize(FUNC, VAL) \
    hipFuncSetAttribute(FUNC, hipFuncAttributeMaxDynamicSharedMemorySize, VAL)
#endif
```
**EN:** This final macro hides the CUDA-vs-HIP API name difference for setting maximum dynamic shared memory size on kernels.
**CN:** 最后的宏把设置内核最大动态共享内存大小时 CUDA 与 HIP 的 API 名称差异隐藏起来。

## Key Concepts / 关键概念
- **EN:** The goal is not feature parity but source-level portability for the rest of the extension.
  **CN:** 其目标不是完全特性对齐，而是为扩展其余部分提供源码级可移植性。
- **EN:** Backend-specific decisions are centralized here so kernels can stay readable.
  **CN:** 后端相关决策集中在这里，从而让内核代码保持可读。

## Dependencies / 依赖关系
- **EN:** Uses CUDA or HIP runtime headers depending on `USE_ROCM`.
  **CN:** 根据 `USE_ROCM` 选择依赖 CUDA 或 HIP runtime 头文件。
- **EN:** Many kernels in `activation_kernels.cu`, `cache_kernels.cu`, and fused attention code rely on these macros.
  **CN:** `activation_kernels.cu`、`cache_kernels.cu` 以及融合注意力代码中的许多内核都依赖这些宏。
