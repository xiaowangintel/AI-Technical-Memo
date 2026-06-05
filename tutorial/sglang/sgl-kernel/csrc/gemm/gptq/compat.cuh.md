# compat.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/gptq/compat.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Namespace and shared declarations
```cpp
/*
Copied from https://github.com/turboderp/exllamav2
*/

#ifndef _compat_cuh
#define _compat_cuh

namespace sglang {
namespace gptq {
// atomicAdd for half types, to support CC < 7.x

__device__ __forceinline__ void atomicAdd_half(half* address, half val) {
  unsigned int* address_as_ui = (unsigned int*)((char*)address - ((size_t)address & 2));
  unsigned int old = *address_as_ui;
  unsigned int assumed;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 17-29: Device helpers and synchronization
```cpp
  do {
    assumed = old;
    __half_raw hsum;
    hsum.x = (size_t)address & 2 ? (old >> 16) : (old & 0xffff);
    half tmpres = __hadd(hsum, val);
    hsum = __half_raw(tmpres);
    old = (size_t)address & 2 ? (old & 0xffff) | (hsum.x << 16) : (old & 0xffff0000) | hsum.x;
    old = atomicCAS(address_as_ui, assumed, old);
  } while (assumed != old);
}

// atomicAdd for half2 types
```
**EN:** This section implements `__hadd`, `__half_raw`, `atomicCAS`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__hadd`、`__half_raw`、`atomicCAS`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 30-46: Device helpers and synchronization
```cpp
__device__ __forceinline__ void atomicAdd_half2(half2* address, half2 val) {
  unsigned int* address_as_ui = (unsigned int*)address;
  unsigned int old = *address_as_ui;
  unsigned int assumed;
  do {
    assumed = old;
    half2 old_val = *((half2*)&old);
    half2 new_val = __hadd2(old_val, val);
    old = atomicCAS(address_as_ui, assumed, *((unsigned int*)&new_val));
  } while (assumed != old);
}

//

#if defined(__CUDA_ARCH__) || defined(USE_ROCM)
#if __CUDA_ARCH__ < 700 || defined(USE_ROCM)
```
**EN:** This section implements `atomicAdd_half2`, `__hadd2`, `atomicCAS`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`atomicAdd_half2`、`__hadd2`、`atomicCAS`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 47-59: Device helpers and synchronization
```cpp
__device__ __forceinline__ void atomicAdd(half* address, half val) {
  atomicAdd_half(address, val);
}

#if __CUDA_ARCH__ < 600 || defined(USE_ROCM)
__device__ __forceinline__ void atomicAdd(half2* address, half2 val) {
  atomicAdd_half2(address, val);
}
#endif

#endif
#endif
```
**EN:** This section implements `atomicAdd`, `defined`, `atomicAdd_half`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`atomicAdd`、`defined`、`atomicAdd_half`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 60-62: Local implementation details
```cpp
}  // namespace gptq
}  // namespace sglang
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: gemm / gptq / compat.cuh
