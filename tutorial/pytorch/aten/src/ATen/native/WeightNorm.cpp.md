# WeightNorm.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/WeightNorm.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on weight norm; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 weight norm；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/TensorUtils.h>
#include <ATen/TensorOperators.h>
#include <ATen/native/cpu/WeightNormKernel.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_weight_norm_differentiable_backward_native.h>
#include <ATen/ops/_weight_norm_interface.h>
#include <ATen/ops/_weight_norm_interface_backward_native.h>
#include <ATen/ops/_weight_norm_interface_native.h>
#include <ATen/ops/_weight_norm_native.h>
#include <ATen/ops/empty_strided.h>
#include <ATen/ops/norm_except_dim.h>
#include <ATen/ops/norm_except_dim_native.h>
#endif

#include <vector>

namespace at::native {

DEFINE_DISPATCH(weight_norm_stub);
DEFINE_DISPATCH(weight_norm_backward_stub);

// Staying faithful to the Python for now for clarity, look for optimizations later
// (e.g., single return statement for RVO)
Tensor norm_except_dim(const Tensor & v, int64_t pow, int64_t dim)
```
- EN: Lines 1-30 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are norm_except_dim, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 norm_except_dim，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
{
  // I assume tensor.contiguous(), view(), norm(), etc. here will dispatch through VariableType.
  if (dim == -1) {
    return v.norm(pow);
  } else if (dim == 0) {
    std::vector<int64_t> output_size(v.dim(), 1);
    output_size[0] = v.size(0);
    return v.contiguous().view({v.size(0), -1}).norm(pow, 1).view(output_size);
  } else if (dim == v.dim() - 1) {
    std::vector<int64_t> output_size(v.dim(), 1);
    output_size[v.dim() - 1] = v.size(v.dim() - 1);
    return v.contiguous().view({-1, v.size(v.dim() - 1)}).norm(pow, 0).view(output_size);
  } else {
    // To consider: at::native::norm_except_dim is probably fine as well,
    // and would avoid an additional dynamic dispatch.
    return at::norm_except_dim(v.transpose(0, dim), pow, 0).transpose(0, dim); // optimize?
  }
}

std::tuple<Tensor,Tensor> weight_norm_cpu(
    const Tensor& v,
    const Tensor& g,
    int64_t dim) {
  auto w = at::empty_like(v, at::MemoryFormat::Contiguous);

  // align with cuda behavior, keep norm in 'Float' when g is 'BFloat16'/'Half'
  const auto dtype = (g.scalar_type() == at::ScalarType::BFloat16 || g.scalar_type() == at::ScalarType::Half) ?
      at::ScalarType::Float : g.scalar_type();
  auto norm = at::empty_strided(g.sizes(), g.strides(), g.options().dtype(dtype));
  weight_norm_stub(kCPU, w, norm, v, g, dim);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp

  return std::tuple<Tensor, Tensor>{std::move(w), std::move(norm)};
}

std::tuple<Tensor, Tensor> weight_norm_backward_cpu(
    const Tensor& grad_w,
    const Tensor& saved_v,
    const Tensor& saved_g,
    const Tensor& saved_norm,
    int64_t dim) {
  TORCH_CHECK(saved_v.is_contiguous(), "saved_v must be contiguous");
  TORCH_CHECK(saved_g.is_contiguous(), "saved_g must be contiguous");
  TORCH_CHECK(saved_norm.is_contiguous(), "saved_norm must be contiguous");

  auto grad_v = at::empty_like(saved_v, at::MemoryFormat::Contiguous);
  auto grad_g = at::empty_like(saved_g, at::MemoryFormat::Contiguous);
  weight_norm_backward_stub(kCPU, grad_v, grad_g, grad_w, saved_v, saved_g, saved_norm, dim);

  return std::tuple<Tensor, Tensor>{std::move(grad_v), std::move(grad_g)};
}

Tensor _weight_norm
  (const Tensor & v_in,
   const Tensor & g_in,
   int64_t dim)
{

  TORCH_CHECK(
    v_in.device() == g_in.device(),
    "weight_norm: expected v_in and g_in to be on the same device, but v_in is "
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    "on ", v_in.device(), " and g_in is on ", g_in.device());

  auto v = v_in.contiguous();
  auto g = g_in.contiguous();

  bool can_use_fused = (dim == 0) || (dim == v.dim() - 1);

  if (can_use_fused) {
    // weight_norm does not have a derivative defined for it, so this will route back through
    // VariableType.cpp, and construct a WeightNormFusedBackward object in the autograd graph.
    return std::get<0>(at::_weight_norm_interface(v, g, dim));
  } else {
    // Double-differentiable primitive ops
    // at::native::norm_except_dim would probably be fine as well.
    return v*(g/at::norm_except_dim(v, 2, dim));
  }
}

// Differentiable backward path, an alternative to weight_norm_backward, to be used
// when backward is itself creating a graph.
// The GradMode::is_enabled() check must be performed within Functions.cpp; that's why we
// define a separate function here, instead of inlining it in weight_norm_cuda_backward.
std::tuple<Tensor, Tensor> _weight_norm_differentiable_backward
  (const Tensor & grad_w,
   const Tensor & saved_v,
   const Tensor & saved_g,
   const Tensor & saved_norms,
   int64_t dim)
{
  // In Functions.cpp, the HardshrinkBackward object supplies "grad.contiguous()"
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
  // as the first argument, so grad_w should be contiguous here.
  // All these checks should succeed:
  TORCH_CHECK(grad_w.is_contiguous(), "grad_w must be contiguous");
  TORCH_CHECK(saved_v.is_contiguous(), "saved_v must be contiguous");
  TORCH_CHECK(saved_g.is_contiguous(), "saved_g must be contiguous");
  TORCH_CHECK(saved_norms.is_contiguous(), "saved_norms must be contiguous");

  int64_t last_dim = saved_v.dim() - 1;
  int64_t last_size = saved_v.size(last_dim);

  // Like weight_norm_fused_backward, weight_norm_differentiable_backward should only ever be called
  // through a WeightNormFusedBackward object, so we expect that dim == 0 || dim == saved_v.size(-1)
  TORCH_CHECK(dim == 0 || dim == last_dim, "Expected dim to be the first or last dimension");

  // saved_g and saved_norms are already shaped to broadcast over the correct dimensions

  // ...but saved_norms might be Float when saved_g and saved_v are half.
  // To consider:  saved_norms.to(..., True /*non_blocking*/);
  auto norms = saved_norms.to(saved_g.scalar_type());

  std::vector<int64_t> bcast_size(saved_v.dim(), 1);

  // Analytic backward path using differentiable primitive ops
  if (dim == 0) {
    bcast_size[0] = saved_v.size(0);
    auto per_dim_sums = (grad_w*saved_v).view({saved_v.size(0), -1}).sum(1).view(bcast_size);
    auto grad_v = (saved_g/norms)*(grad_w - saved_v*(per_dim_sums/(norms*norms)));
    auto grad_g = per_dim_sums/norms;
    return std::tuple<Tensor, Tensor>{std::move(grad_v), std::move(grad_g)};
  } else { // dim == last_dim
```
- EN: The main callable definitions or declarations in this block are size, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段的主要可调用定义或声明包括 size，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 151-159
```cpp
    bcast_size[last_dim] = last_size;
    auto per_dim_sums = (grad_w*saved_v).view({-1, last_size}).sum(0).view(bcast_size);
    auto grad_v = (saved_g/norms)*(grad_w - saved_v*(per_dim_sums/(norms*norms)));
    auto grad_g = per_dim_sums/norms;
    return std::tuple<Tensor, Tensor>{std::move(grad_v), std::move(grad_g)};
  }
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: norm_except_dim, weight_norm_cpu, weight_norm_backward_cpu, _weight_norm, _weight_norm_differentiable_backward.
- CN: 重要符号：norm_except_dim, weight_norm_cpu, weight_norm_backward_cpu, _weight_norm, _weight_norm_differentiable_backward。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/TensorUtils.h, ATen/TensorOperators.h, ATen/native/cpu/WeightNormKernel.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_weight_norm_differentiable_backward_native.h, ATen/ops/_weight_norm_interface.h, ATen/ops/_weight_norm_interface_backward_native.h, ATen/ops/_weight_norm_interface_native.h, ATen/ops/_weight_norm_native.h, ATen/ops/empty_strided.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/TensorUtils.h, ATen/TensorOperators.h, ATen/native/cpu/WeightNormKernel.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_weight_norm_differentiable_backward_native.h, ATen/ops/_weight_norm_interface.h, ATen/ops/_weight_norm_interface_backward_native.h, ATen/ops/_weight_norm_interface_native.h, ATen/ops/_weight_norm_native.h, ATen/ops/empty_strided.h`。
- EN: External/system headers: `vector`.
- CN: 外部/系统头文件：`vector`。
- EN: The implementation revolves around symbols such as `norm_except_dim, weight_norm_cpu, weight_norm_backward_cpu, _weight_norm, _weight_norm_differentiable_backward`.
- CN: 实现围绕 `norm_except_dim, weight_norm_cpu, weight_norm_backward_cpu, _weight_norm, _weight_norm_differentiable_backward` 等符号展开。
