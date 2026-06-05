# kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/marlin/kernel.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Headers and compile-time setup
```cpp

#ifndef MARLIN_NAMESPACE_NAME
#define MARLIN_NAMESPACE_NAME marlin
#endif

#include "marlin.cuh"
#include "marlin_dtypes.cuh"
#include "scalar_type.hpp"

#define MARLIN_KERNEL_PARAMS                                                                                         \
  const int4 *__restrict__ A, const int4 *__restrict__ B, int4 *__restrict__ C, int4 *__restrict__ C_tmp,            \
      const int4 *__restrict__ scales_ptr, const uint16_t *__restrict__ scale2_ptr, const int4 *__restrict__ zp_ptr, \
      const int *__restrict__ g_idx, int num_groups, int prob_m, int prob_n, int prob_k, int lda, int *locks,        \
      bool use_atomic_add, bool use_fp32_reduce, int max_shared_mem
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 16-35: Namespace and shared declarations
```cpp
namespace MARLIN_NAMESPACE_NAME {
template <
    typename scalar_t,                     // compute dtype, half or nv_float16
    const sglang::ScalarTypeId w_type_id,  // weight ScalarType id
    const int threads,                     // number of threads in a threadblock
    const int thread_m_blocks,             // number of 16x16 blocks in the m
                                           // dimension (batchsize) of the
                                           // threadblock
    const int thread_n_blocks,             // same for n dimension (output)
    const int thread_k_blocks,             // same for k dimension (reduction)
    const bool m_block_size_8,             // whether m_block_size == 8
                                           // only works when thread_m_blocks == 1
    const int stages,                      // number of stages for the async global->shared
                                           // fetch pipeline
    const int group_blocks,                // number of consecutive 16x16 blocks
                                           // with a separate quantization scale
    const bool is_zp_float                 // is zero point of float16 type?
    >
__global__ void Marlin(MARLIN_KERNEL_PARAMS);
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 36-36: Local implementation details
```cpp
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `marlin.cuh`, `marlin_dtypes.cuh`, `scalar_type.hpp`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: gemm / marlin / kernel.h
