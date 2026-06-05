# NestedTensorBinaryOps.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/cuda/NestedTensorBinaryOps.cu`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor binary ops; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor binary ops；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/native/nested/NestedTensorBinaryOps.h>

#include <type_traits>

#include <ATen/ATen.h>
#include <ATen/Dispatch.h>

#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/detail/KernelUtils.h>
#include <ATen/cuda/detail/IndexUtils.cuh>
#include <ATen/native/cuda/Loops.cuh>
#include <ATen/native/cuda/MemoryAccess.cuh>

#include <c10/cuda/CUDAMathCompat.h>
#include <c10/cuda/CUDAStream.h>


#include <ATen/native/nested/NestedTensorUtils.h>

#define BLOCK_DIM 256

namespace at {
namespace native {


// only for nested [B, *, D], dense [B, 1, D]
template <typename T, typename func_t>
__global__ void op_dense_esuhm(
    const T* input,
    const T* dense,
```
- EN: Lines 1-30 pull in 12 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 12 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
    T* output,
    int64_t embedding_dim,
    const int64_t* offsets,
    const func_t& f)
{
  // each batch is handled by a block
  const int64_t batch_idx  = blockIdx.x;
  const int64_t grain_size = blockDim.x;
  const int64_t tid = threadIdx.x;
  const int64_t range = offsets[batch_idx + 1] - offsets[batch_idx];
  // each thread handles (embedding_dim // grain_size + (embedding_dim % grain_size <= tid)) elems
  // of the dense embedding
  for (int64_t idx = tid; idx < embedding_dim; idx += grain_size) {
    const T dense_elem = dense[batch_idx * embedding_dim + idx];
    for (int64_t nested_idx = idx; nested_idx < range; nested_idx += embedding_dim) {
      output[offsets[batch_idx] + nested_idx] = f(input[offsets[batch_idx] + nested_idx], dense_elem);
    }
  }
}

template <typename T, typename func_t>
void nested_op_dense_kernelLauncher(
    const T* input, // [sum(*) x embedding_dim]
    const T* dense, // [batch_size x embedding_dim]
    T* output, // [sum(*) x embedding_dim]
    int64_t batch_size,
    int64_t embedding_dim,
    const int64_t* input_offsets,  // [batch_size]
    func_t f)
{
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 61-90
```cpp
  dim3 grid;
  grid.x = batch_size;
  const auto stream = at::cuda::getCurrentCUDAStream();

  op_dense_esuhm<<<grid, BLOCK_DIM, 0, stream>>>(
      input,
      dense,
      output,
      embedding_dim,
      input_offsets,
      f);
}

template <typename scalar_t, typename func_t>
void _nested_op_dense_esuhm_kernel(Tensor& result, const Tensor& self, const Tensor& other, func_t f) {
  auto self_ptr = get_nested_tensor_impl(self);
  auto result_ptr = get_nested_tensor_impl(result);

  const auto self_buffer = self_ptr->get_buffer();
  const auto offsets = self_ptr->get_storage_offsets();
  const auto batch_size = other.size(0);
  const auto embedding_size = other.size(2);

  auto result_buffer = result_ptr->get_buffer();
  auto result_offsets = at::cat({offsets, at::tensor(self_ptr->numel())});
  result_offsets = result_offsets.to(kCUDA);

  const scalar_t* self_data_ptr = self_buffer.const_data_ptr<scalar_t>();
  const scalar_t* other_data_ptr = other.const_data_ptr<scalar_t>();
  scalar_t* result_data_ptr = result_buffer.data_ptr<scalar_t>();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _nested_op_dense_esuhm_kernel, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _nested_op_dense_esuhm_kernel，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
  int64_t* result_offsets_ptr = result_offsets.template data_ptr<int64_t>();

  nested_op_dense_kernelLauncher(
    self_data_ptr,
    other_data_ptr,
    result_data_ptr,
    batch_size,
    embedding_size,
    result_offsets_ptr,
    f);
}

void _nested_op_dense_esuhm_cuda(Tensor& result, const Tensor& self, const Tensor& other, const NESTED_DENSE_OP& op) {
  AT_DISPATCH_ALL_TYPES_AND2(
    ScalarType::Half, ScalarType::BFloat16, self.scalar_type(), "_nested_op_dense_esuhm", [&]() {
    switch (op) {
      case NESTED_DENSE_OP::ADD :
        _nested_op_dense_esuhm_kernel<scalar_t>(result, self, other, [] __host__ __device__ (scalar_t a, scalar_t b) -> scalar_t { return a + b; });
        break;
      case NESTED_DENSE_OP::MUL :
        _nested_op_dense_esuhm_kernel<scalar_t>(result, self, other, [] __host__ __device__ (scalar_t a, scalar_t b) -> scalar_t { return a * b; });
        break;
    }
  });
}

REGISTER_CUDA_DISPATCH(nested_dense_elementwise_stub, &_nested_op_dense_esuhm_cuda)

} // namespace native
} // namespace at
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _nested_op_dense_esuhm_cuda, scalar_type, REGISTER_CUDA_DISPATCH, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _nested_op_dense_esuhm_cuda, scalar_type, REGISTER_CUDA_DISPATCH，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Backend focus: CUDA backend.
- CN: 后端重点：CUDA 后端。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: op_dense_esuhm, nested_op_dense_kernelLauncher, _nested_op_dense_esuhm_kernel, _nested_op_dense_esuhm_cuda, AT_DISPATCH_ALL_TYPES_AND2, REGISTER_CUDA_DISPATCH.
- CN: 重要符号：op_dense_esuhm, nested_op_dense_kernelLauncher, _nested_op_dense_esuhm_kernel, _nested_op_dense_esuhm_cuda, AT_DISPATCH_ALL_TYPES_AND2, REGISTER_CUDA_DISPATCH。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/nested/NestedTensorBinaryOps.h, ATen/ATen.h, ATen/Dispatch.h, ATen/cuda/CUDAContext.h, ATen/cuda/detail/KernelUtils.h, ATen/cuda/detail/IndexUtils.cuh, ATen/native/cuda/Loops.cuh, ATen/native/cuda/MemoryAccess.cuh, c10/cuda/CUDAMathCompat.h, c10/cuda/CUDAStream.h, ATen/native/nested/NestedTensorUtils.h`.
- CN: 主要内部头文件：`ATen/native/nested/NestedTensorBinaryOps.h, ATen/ATen.h, ATen/Dispatch.h, ATen/cuda/CUDAContext.h, ATen/cuda/detail/KernelUtils.h, ATen/cuda/detail/IndexUtils.cuh, ATen/native/cuda/Loops.cuh, ATen/native/cuda/MemoryAccess.cuh, c10/cuda/CUDAMathCompat.h, c10/cuda/CUDAStream.h, ATen/native/nested/NestedTensorUtils.h`。
- EN: External/system headers: `type_traits`.
- CN: 外部/系统头文件：`type_traits`。
- EN: The implementation revolves around symbols such as `op_dense_esuhm, nested_op_dense_kernelLauncher, _nested_op_dense_esuhm_kernel, _nested_op_dense_esuhm_cuda, AT_DISPATCH_ALL_TYPES_AND2, REGISTER_CUDA_DISPATCH`.
- CN: 实现围绕 `op_dense_esuhm, nested_op_dense_kernelLauncher, _nested_op_dense_esuhm_kernel, _nested_op_dense_esuhm_cuda, AT_DISPATCH_ALL_TYPES_AND2, REGISTER_CUDA_DISPATCH` 等符号展开。
