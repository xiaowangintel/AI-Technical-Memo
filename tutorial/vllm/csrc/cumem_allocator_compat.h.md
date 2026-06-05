# cumem_allocator_compat.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `csrc/cumem_allocator_compat.h`
- **Repository**: `vllm-project/vllm`
- **Purpose**: **EN:** Provides a thin compatibility layer so the allocator can call CUDA driver-style virtual-memory APIs on both NVIDIA CUDA and AMD ROCm. **CN:** 提供一层轻量兼容抽象，让分配器在 NVIDIA CUDA 与 AMD ROCm 上都能调用类似 CUDA Driver 的虚拟内存管理接口。

## Line-by-Line Analysis / 逐行分析

### ROCm type aliases and macro bridge / ROCm 类型别名与宏桥接
```cpp
#ifdef USE_ROCM
////////////////////////////////////////
// For compatibility with CUDA and ROCm
////////////////////////////////////////
  #include <hip/hip_runtime_api.h>

extern "C" {
  #ifndef CUDA_SUCCESS
    #define CUDA_SUCCESS hipSuccess
  #endif  // CUDA_SUCCESS

// https://rocm.docs.amd.com/projects/HIPIFY/en/latest/tables/CUDA_Driver_API_functions_supported_by_HIP.html
typedef unsigned long long CUdevice;
typedef hipDeviceptr_t CUdeviceptr;
typedef hipError_t CUresult;
typedef hipCtx_t CUcontext;
typedef hipStream_t CUstream;
typedef hipMemGenericAllocationHandle_t CUmemGenericAllocationHandle;
typedef hipMemAllocationGranularity_flags CUmemAllocationGranularity_flags;
typedef hipMemAllocationProp CUmemAllocationProp;
typedef hipMemAccessDesc CUmemAccessDesc;

  #define CU_MEM_ALLOCATION_TYPE_PINNED hipMemAllocationTypePinned
  #define CU_MEM_LOCATION_TYPE_DEVICE hipMemLocationTypeDevice
  #define CU_MEM_ACCESS_FLAGS_PROT_READWRITE hipMemAccessFlagsProtReadWrite
  #define CU_MEM_ALLOC_GRANULARITY_MINIMUM hipMemAllocationGranularityMinimum

  // https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__TYPES.html
  #define CU_MEM_ALLOCATION_COMP_NONE 0x0
```
**EN:** Under USE_ROCM the header remaps CUDA driver concepts to HIP equivalents: result codes, contexts, streams, allocation handles, memory-location enums, and access flags. This keeps the allocator implementation mostly platform-agnostic.
**CN:** 在 USE_ROCM 条件下，头文件把 CUDA Driver 里的结果码、上下文、流、分配句柄、内存位置枚举与访问标志映射为 HIP 对应项，使分配器主体代码基本保持平台无关。

### Error and context shims / 错误处理与上下文封装
```cpp
// Error Handling
// https://docs.nvidia.com/cuda/archive/11.4.4/cuda-driver-api/group__CUDA__ERROR.html
CUresult cuGetErrorString(CUresult hipError, const char** pStr) {
  *pStr = hipGetErrorString(hipError);
  return CUDA_SUCCESS;
}

// Context Management
// https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__CTX.html
CUresult cuCtxGetCurrent(CUcontext* ctx) {
  // This API is deprecated on the AMD platform, only for equivalent cuCtx
  // driver API on the NVIDIA platform.
  return hipCtxGetCurrent(ctx);
}

CUresult cuCtxSetCurrent(CUcontext ctx) {
  // This API is deprecated on the AMD platform, only for equivalent cuCtx
  // driver API on the NVIDIA platform.
  return hipCtxSetCurrent(ctx);
}

// Primary Context Management
// https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__PRIMARY__CTX.html
CUresult cuDevicePrimaryCtxRetain(CUcontext* ctx, CUdevice dev) {
  return hipDevicePrimaryCtxRetain(ctx, dev);
}
```
**EN:** These wrappers emulate CUDA driver entry points such as cuGetErrorString, cuCtxGetCurrent, cuCtxSetCurrent, and cuDevicePrimaryCtxRetain by directly forwarding to HIP runtime/context APIs.
**CN:** 这些包装函数通过直接转发到 HIP 运行时/上下文接口，模拟 cuGetErrorString、cuCtxGetCurrent、cuCtxSetCurrent、cuDevicePrimaryCtxRetain 等 CUDA Driver 入口。

### Virtual-memory API forwarding / 虚拟内存 API 转发
```cpp
// Virtual Memory Management
// https://docs.nvidia.com/cuda/cuda-driver-api/group__CUDA__VA.html
CUresult cuMemAddressFree(CUdeviceptr ptr, size_t size) {
  return hipMemAddressFree(ptr, size);
}

CUresult cuMemAddressReserve(CUdeviceptr* ptr, size_t size, size_t alignment,
                             CUdeviceptr addr, unsigned long long flags) {
  return hipMemAddressReserve(ptr, size, alignment, addr, flags);
}

CUresult cuMemCreate(CUmemGenericAllocationHandle* handle, size_t size,
                     const CUmemAllocationProp* prop,
                     unsigned long long flags) {
  return hipMemCreate(handle, size, prop, flags);
}

CUresult cuMemGetAllocationGranularity(
    size_t* granularity, const CUmemAllocationProp* prop,
    CUmemAllocationGranularity_flags option) {
  return hipMemGetAllocationGranularity(granularity, prop, option);
}

CUresult cuMemMap(CUdeviceptr dptr, size_t size, size_t offset,
                  CUmemGenericAllocationHandle handle,
                  unsigned long long flags) {
  return hipMemMap(dptr, size, offset, handle, flags);
}

CUresult cuMemRelease(CUmemGenericAllocationHandle handle) {
  return hipMemRelease(handle);
}

CUresult cuMemSetAccess(CUdeviceptr ptr, size_t size,
                        const CUmemAccessDesc* desc, size_t count) {
  return hipMemSetAccess(ptr, size, desc, count);
}

CUresult cuMemUnmap(CUdeviceptr ptr, size_t size) {
  return hipMemUnmap(ptr, size);
}
```
**EN:** The allocator depends on reserve/create/map/set-access/release style APIs. This block defines ROCm-side wrappers with CUDA-like names so the allocator can use a single VMM code path.
**CN:** 分配器依赖 reserve/create/map/set-access/release 这一组虚拟内存 API。本段用 CUDA 风格名称定义 ROCm 包装层，使分配器能够复用统一的 VMM 代码路径。

### Native CUDA fallback / 原生 CUDA 回退路径
```cpp
#else
////////////////////////////////////////
// Import CUDA headers for NVIDIA GPUs
////////////////////////////////////////
  #include <cuda_runtime_api.h>
  #include <cuda.h>
#endif
```
**EN:** On NVIDIA builds the compatibility layer is trivial: include cuda_runtime_api.h and cuda.h and use the real driver definitions directly.
**CN:** 在 NVIDIA 构建下，兼容层几乎为空：直接包含 cuda_runtime_api.h 与 cuda.h，使用真实的 Driver API 定义。

## Key Concepts / 关键概念

- **EN:** The file standardizes the CUDA virtual-memory API surface across CUDA and HIP.
  **CN:** 该文件把 CUDA 虚拟内存 API 表面统一到 CUDA 与 HIP 两个平台上。
- **EN:** extern "C" wrappers preserve C linkage for the shim symbols.
  **CN:** extern "C" 包装用于保持这些 shim 符号的 C 链接约定。
- **EN:** Allocator code can stay focused on VMM logic instead of scattering #ifdef branches everywhere.
  **CN:** 分配器代码因此可以专注于 VMM 逻辑，而不必在各处散落大量 #ifdef 分支。

## Dependencies / 依赖关系

- **EN:** Depends on HIP runtime headers when USE_ROCM is enabled.
  **CN:** 启用 USE_ROCM 时依赖 HIP runtime 头文件。
- **EN:** Depends on CUDA driver/runtime headers on NVIDIA builds.
  **CN:** 在 NVIDIA 构建下依赖 CUDA driver/runtime 头文件。
- **EN:** Included directly by csrc/cumem_allocator.cpp.
  **CN:** 被 csrc/cumem_allocator.cpp 直接包含使用。
