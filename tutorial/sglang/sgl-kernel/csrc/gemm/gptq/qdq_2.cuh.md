# qdq_2.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/gptq/qdq_2.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Namespace and shared declarations
```cpp
/*
Copied from https://github.com/turboderp/exllamav2
*/

#ifndef _qdq_2_cuh
#define _qdq_2_cuh

#include "qdq_util.cuh"

namespace sglang {
namespace gptq {

// Permutation:
//
// ffddbb99 77553311  eeccaa88 66442200

__forceinline__ __device__ void shuffle_2bit_16(uint32_t* q, int stride) {
  uint32_t qa = q[0];
  uint32_t qb = 0;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 21-40: Device helpers and synchronization
```cpp
#pragma unroll
  for (int i = 0; i < 8; i++) {
    uint32_t qa0 = qa & 0x03;
    uint32_t qa1 = (qa & 0x0c) >> 2;
    qa >>= 4;
    qb |= (qa1 << (i * 2 + 16));
    qb |= (qa0 << (i * 2));
  }
  q[0] = qb;
}

__forceinline__ __device__ void dequant_2bit_16(const uint32_t q_0, half2 (&dq)[8], int stride, const uint32_t zero) {
  const uint32_t c0 = 0x64006400;
  const half y4_ = __float2half_rn(1.0f / 4.0f);
  const half y16_ = __float2half_rn(1.0f / 16.0f);
  const half y64_ = __float2half_rn(1.0f / 64.0f);
  const half2 y4 = __halves2half2(y4_, y4_);
  const half2 y16 = __halves2half2(y16_, y16_);
  const half2 y64 = __halves2half2(y64_, y64_);
```
**EN:** This section implements `dequant_2bit_16`, `__float2half_rn`, `__halves2half2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dequant_2bit_16`、`__float2half_rn`、`__halves2half2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 41-60: Local implementation details
```cpp
  const half_uint16 z1_(0xe400 | zero);  // half(-1024.0f - zero);
  const half z4_ = __hsub(__int2half_rn(-256), __int2half_rn(zero));
  const half z16_ = __hsub(__int2half_rn(-64), __int2half_rn(zero));
  const half z64_ = __hsub(__int2half_rn(-16), __int2half_rn(zero));
  const half2 z1 = __half2half2(z1_.as_half);
  const half2 z4 = __half2half2(z4_);
  const half2 z16 = __half2half2(z16_);
  const half2 z64 = __half2half2(z64_);

  uint32_t qa = q_0;
  half2_uint32 q0((qa & 0x00030003) | c0);  // half2(q[ 0], q[ 1])      + 1024
  half2_uint32 q1((qa & 0x000c000c) | c0);  // half2(q[ 2], q[ 3]) *  4 + 1024
  half2_uint32 q2((qa & 0x00300030) | c0);  // half2(q[ 4], q[ 5]) * 16 + 1024
  half2_uint32 q3((qa & 0x00c000c0) | c0);  // half2(q[ 6], q[ 7]) * 64 + 1024
  qa >>= 8;
  half2_uint32 q4((qa & 0x00030003) | c0);  // half2(q[ 8], q[ 8])      + 1024
  half2_uint32 q5((qa & 0x000c000c) | c0);  // half2(q[10], q[11]) *  4 + 1024
  half2_uint32 q6((qa & 0x00300030) | c0);  // half2(q[12], q[13]) * 16 + 1024
  half2_uint32 q7((qa & 0x00c000c0) | c0);  // half2(q[14], q[15]) * 64 + 1024
```
**EN:** This section fills in the local implementation details around `z1_`, `__hsub`, `__half2half2`, completing the behavior required by the file.
**CN:** 本段补充了`z1_`、`__hsub`、`__half2half2`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 61-73: Local implementation details
```cpp
  dq[0] = __hadd2(q0.as_half2, z1);
  dq[1] = __hfma2(q1.as_half2, y4, z4);
  dq[2] = __hfma2(q2.as_half2, y16, z16);
  dq[3] = __hfma2(q3.as_half2, y64, z64);
  dq[4] = __hadd2(q4.as_half2, z1);
  dq[5] = __hfma2(q5.as_half2, y4, z4);
  dq[6] = __hfma2(q6.as_half2, y16, z16);
  dq[7] = __hfma2(q7.as_half2, y64, z64);
}

}  // namespace gptq
}  // namespace sglang
```
**EN:** This section fills in the local implementation details around `__hadd2`, `__hfma2`, completing the behavior required by the file.
**CN:** 本段补充了`__hadd2`、`__hfma2`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 74-74: Local implementation details
```cpp
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `qdq_util.cuh`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: gemm / gptq / qdq_2.cuh
