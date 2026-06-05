# kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/kernel.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Declare the fully-templated Marlin GEMM kernel behind a shared parameter macro. / [CN] 通过共享参数宏声明完全模板化的 Marlin GEMM 内核。

## Line-by-Line Analysis / 逐行分析
### Shared dtype include
```cpp
#include "marlin_dtypes.cuh"
namespace MARLIN_NAMESPACE_NAME {
```
**EN:** `kernel.h` only needs the fragment/dtype definitions, because this file is about the kernel interface rather than the implementation.
**CN:** `kernel.h` 只需要片段和 dtype 定义，因为这个文件关注的是内核接口，而不是实现细节。

### Long kernel parameter list packed into a macro
```cpp
#define MARLIN_KERNEL_PARAMS                                                     const int4 *__restrict__ A, const int4 *__restrict__ B,                            int4 *__restrict__ C, int4 *__restrict__ C_tmp,                                const int4 *__restrict__ b_bias_ptr,                                           const float *__restrict__ a_scales_ptr,                                        const int4 *__restrict__ scales_ptr,                                           const float *__restrict__ global_scale_ptr,                                    const int4 *__restrict__ zp_ptr, const int *__restrict__ g_idx,                int num_groups, int prob_m, int prob_n, int prob_k, int lda, int *locks,       bool has_bias, bool use_atomic_add, bool use_fp32_reduce,                      int max_shared_mem
```
**EN:** The macro centralizes the very long argument list used by all Marlin kernel instantiations. It includes activation/weight pointers, scales, zero-points, act-order metadata, workspace locks, and runtime behavior flags.
**CN:** 这个宏把所有 Marlin 内核实例共享的长参数列表集中管理起来，其中包含激活/权重指针、scale、zero point、act-order 元数据、workspace 锁以及运行时行为开关。

### Templated kernel declaration
```cpp
template <const vllm::ScalarTypeId a_type_id,
          const vllm::ScalarTypeId b_type_id,
          const vllm::ScalarTypeId c_type_id,
          const vllm::ScalarTypeId s_type_id,
          const int threads,
          const int thread_m_blocks,
          const int thread_n_blocks,
          const int thread_k_blocks,
          const bool m_block_size_8,
          const int stages,
          const int group_blocks,
          const bool is_zp_float>
__global__ void Marlin(MARLIN_KERNEL_PARAMS);
```
**EN:** The kernel is specialized at compile time for activation type, weight type, output type, scale type, tile shape, pipeline depth, grouping behavior, and zero-point representation. This avoids runtime branching inside the hot GEMM loop.
**CN:** 该内核在编译期就对激活类型、权重类型、输出类型、scale 类型、tile 形状、流水深度、分组方式以及 zero-point 表示做了特化，从而避免在热点 GEMM 循环中出现运行时分支。

## Key Concepts / 关键概念
- Macro-based signature sharing keeps declaration, definition, and selector consistent. / 基于宏的签名复用让声明、定义和选择器保持一致。
- Nearly all performance-critical behavior is encoded in template parameters. / 几乎所有性能关键行为都编码在模板参数里。

## Dependencies / 依赖关系
- Depends on `marlin_dtypes.cuh` for fragment types. / 依赖 `marlin_dtypes.cuh` 提供片段类型。
- Implemented in `marlin_template.h` and instantiated/selected from `marlin.cu`. / 实现在 `marlin_template.h` 中，并由 `marlin.cu` 负责实例化与选择。
