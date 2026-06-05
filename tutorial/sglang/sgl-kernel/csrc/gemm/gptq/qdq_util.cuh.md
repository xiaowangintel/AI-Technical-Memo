# qdq_util.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/gptq/qdq_util.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Namespace and shared declarations
```cpp
/*
Copied from https://github.com/turboderp/exllamav2
*/

#ifndef _qdq_util_cuh
#define _qdq_util_cuh

namespace sglang {
namespace gptq {

union half2_uint32 {
  uint32_t as_uint32;
  half2 as_half2;
  __device__ half2_uint32(uint32_t val) : as_uint32(val) {}
  __device__ half2_uint32(half2 val) : as_half2(val) {}
};
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 18-36: Device helpers and synchronization
```cpp
union half_uint16 {
  uint16_t as_uint16;
  half as_half;
  __device__ half_uint16(uint16_t val) : as_uint16(val) {}
  __device__ half_uint16(half val) : as_half(val) {}
};

// Max_scale premultiplied by 1/256

__forceinline__ __device__ half dq_scale(const int qs, const half max_scale) {
  int qs_i = qs + 1;
  half qs_h = __int2half_rn(qs_i * qs_i);
  qs_h = __hmul(qs_h, max_scale);
  return qs_h;
}

__forceinline__ __device__ half dq(const int q, const int qzero, const half scale) {
  return __hmul(__int2half_rn(q - qzero), scale);
}
```
**EN:** This section implements `half_uint16`, `dq_scale`, `dq`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`half_uint16`、`dq_scale`、`dq`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 37-50: Device helpers and synchronization
```cpp

__forceinline__ __device__ half dq_ns(const int q, const int qzero) {
  // return __hsub(__int2half_rn(q), __int2half_rn(qzero));
  return __int2half_rn(q - qzero);
}

__forceinline__ __device__ int exb(const uint32_t q, const int shift, const int mask) {
  return (int)((q >> shift) & mask);
}

__forceinline__ __device__ int exb(const uint32_t q1, const uint32_t q0, const int shift, const int mask) {
  return (int)(__funnelshift_rc(q0, q1, shift) & mask);
}
```
**EN:** This section implements `dq_ns`, `exb`, `__int2half_rn`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dq_ns`、`exb`、`__int2half_rn`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 51-53: Local implementation details
```cpp
}  // namespace gptq
}  // namespace sglang
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: gemm / gptq / qdq_util.cuh
