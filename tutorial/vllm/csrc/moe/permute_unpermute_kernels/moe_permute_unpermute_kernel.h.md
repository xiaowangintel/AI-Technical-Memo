# moe_permute_unpermute_kernel.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/permute_unpermute_kernels/moe_permute_unpermute_kernel.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines low-level permutation/unpermutation CUDA kernels and dispatch helpers for MoE routing. / 定义 MoE 路由所需的底层置换/逆置换 CUDA 内核及分派辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-12)
```cpp
#pragma once
// reference from tensorrt_llm moe kernel implementation archive in
// https://github.com/BBuf/tensorrt-llm-moe/tree/master

#include <c10/core/ScalarType.h>
#include <torch/all.h>
#include "dispatch.h"
#include <cub/cub.cuh>
#include <cub/device/device_radix_sort.cuh>
#include <cub/util_type.cuh>
#include "cutlass/numeric_size.h"
#include "cutlass/array.h"
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: get_ptr (lines 14-17)
```cpp
template <typename T>
inline T* get_ptr(torch::Tensor& t) {
  return reinterpret_cast<T*>(t.data_ptr());
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: get_ptr (lines 19-22)
```cpp
template <typename T>
inline const T* get_ptr(const torch::Tensor& t) {
  return reinterpret_cast<const T*>(t.data_ptr());
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Class: CubKeyValueSorter (lines 24-43)
```cpp
class CubKeyValueSorter {
 public:
  CubKeyValueSorter();

  CubKeyValueSorter(int const num_experts);

  void updateNumExperts(int const num_experts);

  static size_t getWorkspaceSize(size_t const num_key_value_pairs,
                                 int const num_experts);

  void run(void* workspace, size_t const workspace_size, int const* keys_in,
           int* keys_out, int const* values_in, int* values_out,
           size_t const num_key_value_pairs, cudaStream_t stream);
// ...
  int num_experts_;
  int num_bits_;
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Top-k selection / Top-k 选择
- Token permutation / Token 置换
- Token restoration / Token 还原
- CUDA programming model / CUDA 编程模型
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: PyTorch / c10, PyTorch / ATen, C++ standard library or platform support, CUB block primitives, CUTLASS / CUTE templates
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
