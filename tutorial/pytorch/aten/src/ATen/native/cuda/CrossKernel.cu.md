# CrossKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/CrossKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `cross_kernel`, `launch_cross_kernel`, `cross_impl`, `cross_stub`.
- 用途（中文）: 实现与 `cross_kernel`, `launch_cross_kernel`, `cross_impl`, `cross_stub` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/native/Cross.h>
   3: #include <ATen/cuda/detail/KernelUtils.h>
   4: #include <ATen/native/cuda/Loops.cuh>
   5: #include <ATen/Dispatch.h>
   6: #include <ATen/core/Tensor.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/native/Cross.h>`, `<ATen/cuda/detail/KernelUtils.h>`, `<ATen/native/cuda/Loops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/native/Cross.h>`, `<ATen/cuda/detail/KernelUtils.h>`, `<ATen/native/cuda/Loops.cuh>`。

### Lines 8-27
```cpp
   8: namespace at::native {
   9: 
  10: template <typename T, typename OffsetCalc, typename StrideType>
  11: __global__ void cross_kernel(
  12:     int numel, T* out, const T* x1, const T* x2, OffsetCalc offset_calculator,
  13:     StrideType ostride, StrideType x1stride, StrideType x2stride) {
  14:   CUDA_KERNEL_LOOP(i, numel) {
  15:     const auto offsets = offset_calculator.get(i);
  16:     auto* out_row = out + offsets[0];
  17:     const auto* x1_row = x1 + offsets[1];
  18:     const auto* x2_row = x2 + offsets[2];
  19: 
  20:     const T val0 = (x1_row[1 * x1stride] * x2_row[2 * x2stride] -
  21:                     x1_row[2 * x1stride] * x2_row[1 * x2stride]);
  22: 
  23:     const T val1 = (x1_row[2 * x1stride] * x2_row[0 * x2stride] -
  24:                     x1_row[0 * x1stride] * x2_row[2 * x2stride]);
  25: 
  26:     const T val2 = (x1_row[0 * x1stride] * x2_row[1 * x2stride] -
  27:                     x1_row[1 * x1stride] * x2_row[0 * x2stride]);
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines GPU kernel entry point(s) `cross_kernel`, which run directly on CUDA threads.
- CN: 该代码块定义了 GPU 内核入口 `cross_kernel`，它们会直接在 CUDA 线程上执行。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 30-34
```cpp
  30:     out_row[0 * ostride] = val0;
  31:     out_row[1 * ostride] = val1;
  32:     out_row[2 * ostride] = val2;
  33:   }
  34: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 36-57
```cpp
  36: void launch_cross_kernel(const TensorIteratorBase& iter, int64_t ostride,
  37:                          int64_t x1stride, int64_t x2stride) {
  38:   const auto N = iter.numel();
  39:   auto offset_calculator = make_element_offset_calculator<3>(iter);
  40:   TORCH_INTERNAL_ASSERT_DEBUG_ONLY(N > 0 && N <= std::numeric_limits<int32_t>::max());
  41:   int64_t grid = (N + num_threads() - 1) / num_threads();
  42:   auto stream = at::cuda::getCurrentCUDAStream();
  43: 
  44:   AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2(kHalf, kBFloat16, iter.common_dtype(), "cross_cuda", [&] {
  45:     auto out = static_cast<scalar_t*>(iter.data_ptr(0));
  46:     auto x1 = static_cast<const scalar_t*>(iter.data_ptr(1));
  47:     auto x2 = static_cast<const scalar_t*>(iter.data_ptr(2));
  48:     constexpr int64_t int_max = std::numeric_limits<int>::max();
  49:     if (ostride * 2 > int_max || x1stride * 2 > int_max || x2stride * 2 > int_max) {
  50:       cross_kernel<<<grid, num_threads(), 0, stream>>>(
  51:           N, out, x1, x2, offset_calculator, ostride, x1stride, x2stride);
  52:       C10_CUDA_KERNEL_LAUNCH_CHECK();
  53:     } else {
  54:       cross_kernel<<<grid, num_threads(), 0, stream>>>(
  55:           N, out, x1, x2, offset_calculator,
  56:           static_cast<int>(ostride),
  57:           static_cast<int>(x1stride),
```
- EN: This block defines or continues the implementation of `launch_cross_kernel`.
- CN: 该代码块定义或继续实现 `launch_cross_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 58-62
```cpp
  58:           static_cast<int>(x2stride));
  59:       C10_CUDA_KERNEL_LAUNCH_CHECK();
  60:     }
  61:   });
  62: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 64-85
```cpp
  64: void cross_impl(const Tensor& result, const Tensor& x1, const Tensor& x2, int64_t dim) {
  65:   const int64_t ostride = result.stride(dim);
  66:   const int64_t x1stride = x1.stride(dim);
  67:   const int64_t x2stride = x2.stride(dim);
  68: 
  69:   auto iter = TensorIteratorConfig()
  70:       .add_output(result)
  71:       .add_const_input(x1)
  72:       .add_const_input(x2)
  73:       .resize_outputs(false)
  74:       .declare_static_shape(result.sizes(), /*squash_dims=*/dim)
  75:       .build();
  76: 
  77:   if (iter.numel() == 0) {
  78:     return;
  79:   }
  80: 
  81:   if (iter.can_use_32bit_indexing()) {
  82:     launch_cross_kernel(iter, ostride, x1stride, x2stride);
  83:   } else {
  84:     for (auto&& sub_iter: iter.with_32bit_indexing()) {
  85:       launch_cross_kernel(sub_iter, ostride, x1stride, x2stride);
```
- EN: This block defines or continues the implementation of `cross_impl`.
- CN: 该代码块定义或继续实现 `cross_impl`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 86-88
```cpp
  86:     }
  87:   }
  88: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 90-90
```cpp
  90: REGISTER_DISPATCH(cross_stub, &cross_impl)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 92-92
```cpp
  92: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `TensorIteratorConfig` builds iterator state before launch. / `TensorIteratorConfig` 在启动前构建迭代器状态。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `__global__` marks a CUDA kernel entry point executed by many GPU threads. / `__global__` 表示由大量 GPU 线程执行的 CUDA 内核入口。
- `CUDA_KERNEL_LOOP` expands index-space iteration on the GPU. / `CUDA_KERNEL_LOOP` 展开 GPU 上的索引空间循环。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/native/Cross.h>`
  - `<ATen/cuda/detail/KernelUtils.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/Dispatch.h>`
  - `<ATen/core/Tensor.h>`
- Runtime symbols / 运行时符号:
  - `cross_stub`
  - `launch_cross_kernel`
  - `TensorIteratorBase`
  - `TensorIteratorConfig`
  - `AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND2`
  - `REGISTER_DISPATCH`
  - `at::cuda::getCurrentCUDAStream`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
