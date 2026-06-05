# UnaryGammaKernels.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/UnaryGammaKernels.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `gpu_kernel`, `trigamma_kernel_cuda`, `polygamma_kernel_cuda`, `lgamma_kernel_cuda`.
- 用途（中文）: 实现与 `gpu_kernel`, `trigamma_kernel_cuda`, `polygamma_kernel_cuda`, `lgamma_kernel_cuda` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <limits>
   3: #include <ATen/native/UnaryOps.h>
   4: #include <ATen/native/cuda/JitLoops.cuh>
   5: #include <ATen/native/cuda/Loops.cuh>
   6: #include <ATen/AccumulateType.h>
   7: #include <ATen/Dispatch.h>
   8: #include <ATen/native/DispatchStub.h>
   9: #include <ATen/native/TensorIterator.h>
  10: #include <ATen/native/cuda/Math.cuh>
  11: #include <ATen/native/Math.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<limits>`, `<ATen/native/UnaryOps.h>`, `<ATen/native/cuda/JitLoops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<limits>`, `<ATen/native/UnaryOps.h>`, `<ATen/native/cuda/JitLoops.cuh>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 13-34
```cpp
  13: namespace at::native {
  14: 
  15: #if AT_USE_JITERATOR()
  16: constexpr char digamma_name[] = "digamma";
  17: #endif // AT_USE_JITERATOR()
  18: // See note [Jiterator]
  19: void digamma_kernel_cuda(TensorIteratorBase& iter) {
  20:   #if AT_USE_JITERATOR()
  21:     AT_DISPATCH_FLOATING_TYPES_AND2(
  22:       at::ScalarType::Half,
  23:       at::ScalarType::BFloat16,
  24:       iter.common_dtype(), "digamma_cuda", [&]() {
  25:         jitted_gpu_kernel</*name=*/digamma_name,
  26:                           /*return_dtype=*/ scalar_t,
  27:                           /*common_dtype=*/ scalar_t,
  28:                           /*arity=*/ 1>(iter, digamma_string);
  29:     });
  30:   #else
  31:     AT_DISPATCH_FLOATING_TYPES_AND2(
  32:       at::ScalarType::Half,
  33:       at::ScalarType::BFloat16,
  34:       iter.common_dtype(), "digamma_cuda", [&]() {
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 35-40
```cpp
  35:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
  36:           return calc_digamma(a);
  37:         });
  38:     });
  39:   #endif // AT_USE_JITERATOR()
  40: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 42-42
```cpp
  42: // See note [Jiterator]
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 43-64
```cpp
  43: constexpr char trigamma_name[] = "trigamma";
  44: void trigamma_kernel_cuda(TensorIteratorBase& iter) {
  45:   #if AT_USE_JITERATOR()
  46:     AT_DISPATCH_FLOATING_TYPES_AND2(
  47:       at::ScalarType::Half,
  48:       at::ScalarType::BFloat16,
  49:       iter.common_dtype(), "trigamma_cuda", [&]() {
  50:         jitted_gpu_kernel</*name=*/trigamma_name,
  51:                           /*return_dtype=*/ scalar_t,
  52:                           /*common_dtype=*/ scalar_t,
  53:                           /*arity=*/ 1>(iter, trigamma_string);
  54:     });
  55:   #else
  56:     AT_DISPATCH_FLOATING_TYPES_AND2(
  57:       at::ScalarType::Half,
  58:       at::ScalarType::BFloat16,
  59:       iter.common_dtype(), "trigamma_cuda", [&]() {
  60:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
  61:           return calc_trigamma(a);
  62:         });
  63:     });
  64:   #endif // AT_USE_JITERATOR()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `trigamma_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `trigamma_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 65-65
```cpp
  65: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 67-88
```cpp
  67: constexpr char polygamma_name[] = "polygamma";
  68: void polygamma_kernel_cuda(TensorIteratorBase& iter, int64_t n) {
  69:   if (n == 0) {
  70:     digamma_kernel_cuda(iter);
  71:   } else if (n == 1) {
  72:     trigamma_kernel_cuda(iter);
  73:   } else {
  74: #if AT_USE_JITERATOR()
  75:     // TODO : `unary_jitted_gpu_kernel` for cleaner UX.
  76:     AT_DISPATCH_FLOATING_TYPES_AND2(
  77:       at::ScalarType::Half,
  78:       at::ScalarType::BFloat16,
  79:         iter.common_dtype(), "polygamma_cuda", [&]() {
  80:           jitted_gpu_kernel<
  81:               /*name=*/polygamma_name,
  82:               /*return_dtype=*/scalar_t,
  83:               /*common_dtype=*/scalar_t,
  84:               /*arity=*/1>(
  85:               iter,
  86:               polygamma_string,
  87:               /*scalar_pos=*/at::cuda::jit::BinaryFuncVariant::NoScalar,
  88:               /*scalar_val=*/0,
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `polygamma_kernel_cuda`.
- CN: 该代码块定义或继续实现 `polygamma_kernel_cuda`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 89-89
```cpp
  89:               /*extra_args=*/std::make_tuple(n));
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 90-102
```cpp
  90:         });
  91: #else
  92:     AT_DISPATCH_FLOATING_TYPES_AND2(
  93:       at::ScalarType::Half,
  94:       at::ScalarType::BFloat16,
  95:         iter.common_dtype(), "polygamma_cuda", [&]() {
  96:           gpu_kernel(iter, [=] GPU_LAMBDA(scalar_t a) -> scalar_t {
  97:             return calc_polygamma<scalar_t, /*is_cuda=*/true>(a, static_cast<int>(n));
  98:           });
  99:         });
 100: #endif // AT_USE_JITERATOR()
 101:   }
 102: }
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `gpu_kernel`.
- CN: 该代码块定义或继续实现 `gpu_kernel`。
- EN: The `AT_DISPATCH` family expands one code path across the scalar types supported by this operator.
- CN: `AT_DISPATCH` 系列宏会把一套逻辑扩展到该算子支持的多种标量类型。

### Lines 104-125
```cpp
 104: constexpr char lgamma_name[] = "lgamma_kernel";
 105: void lgamma_kernel_cuda(TensorIteratorBase& iter) {
 106:   #if AT_USE_JITERATOR()
 107:     AT_DISPATCH_FLOATING_TYPES_AND2(
 108:       at::ScalarType::Half,
 109:       at::ScalarType::BFloat16,
 110:       iter.common_dtype(), "lgamma_cuda", [&]() {
 111:         jitted_gpu_kernel</*name=*/lgamma_name,
 112:                           /*return_dtype=*/ scalar_t,
 113:                           /*common_dtype=*/ scalar_t,
 114:                           /*arity=*/ 1>(iter, lgamma_string);
 115:     });
 116:   #else
 117:     AT_DISPATCH_FLOATING_TYPES_AND2(
 118:       at::ScalarType::Half,
 119:       at::ScalarType::BFloat16,
 120:       iter.common_dtype(), "lgamma_cuda", [&]() {
 121:         gpu_kernel(iter, []GPU_LAMBDA(scalar_t a) -> scalar_t {
 122:           return ::lgamma(a);
 123:         });
 124:     });
 125:   #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `lgamma_kernel_cuda`, `gpu_kernel`.
- CN: 该代码块定义或继续实现 `lgamma_kernel_cuda`, `gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 126-126
```cpp
 126: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 128-130
```cpp
 128: REGISTER_DISPATCH(digamma_stub, &digamma_kernel_cuda)
 129: REGISTER_DISPATCH(polygamma_stub, &polygamma_kernel_cuda)
 130: REGISTER_DISPATCH(lgamma_stub, &lgamma_kernel_cuda)
```
- EN: The registration macro binds this implementation into PyTorch dispatch so higher-level operators can call the CUDA specialization.
- CN: 注册宏把该实现接入 PyTorch 分发系统，使上层算子能够调用这个 CUDA 特化版本。

### Lines 132-132
```cpp
 132: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `jitted_gpu_kernel` relies on Jiterator-style runtime code generation. / `jitted_gpu_kernel` 依赖 Jiterator 风格的运行时代码生成。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<limits>`
  - `<ATen/native/UnaryOps.h>`
  - `<ATen/native/cuda/JitLoops.cuh>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/AccumulateType.h>`
  - `<ATen/Dispatch.h>`
  - `<ATen/native/DispatchStub.h>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/cuda/Math.cuh>`
  - `<ATen/native/Math.h>`
- Runtime symbols / 运行时符号:
  - `digamma_stub`
  - `polygamma_stub`
  - `lgamma_stub`
  - `gpu_kernel`
  - `jitted_gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_FLOATING_TYPES_AND2`
  - `REGISTER_DISPATCH`
  - `at::cuda::jit::BinaryFuncVariant::NoScalar`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
