# JitLoops.cuh — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/JitLoops.cuh`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `jitted_gpu_kernel`, `opmath_jitted_gpu_kernel_with_scalars`.
- 用途（中文）: 声明或定义与 `jitted_gpu_kernel`, `opmath_jitted_gpu_kernel_with_scalars` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
   1: #pragma once
   2: 
   3: #include <ATen/jit_macros.h>
   4: 
   5: #if AT_USE_JITERATOR()
   6: 
   7: #include <ATen/cuda/CUDAConfig.h>
   8: 
   9: #include <ATen/OpMathType.h>
  10: #include <ATen/TensorIterator.h>
  11: #include <ATen/native/TensorIteratorDynamicCasting.h>
  12: 
  13: #include <ATen/native/cuda/MemoryAccess.cuh>
  14: 
  15: #include <ATen/native/cuda/CUDAJitLoops.cuh>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/jit_macros.h>`, `<ATen/cuda/CUDAConfig.h>`, `<ATen/OpMathType.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/jit_macros.h>`, `<ATen/cuda/CUDAConfig.h>`, `<ATen/OpMathType.h>`。
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 17-37
```cpp
  17: namespace at::native {
  18: 
  19: /* Note [Jiterator]
  20: The "jiterator" simply just-in-time compiles the same kernels that
  21: Loops.cuh (and CUDALoops.cuh) usually build. This reduces build time,
  22: build size, and initial CUDA context size.
  23: 
  24: By default on non-Windows systems, it also caches compiled kernels in ~/.cache/torch/kernels.
  25: This behavior is controlled with two environment variables:
  26:   - USE_PYTORCH_KERNEL_CACHE, if set to zero then this will disable all cache use
  27:   - PYTORCH_KERNEL_CACHE_PATH, if set specifies the folder to use for cached kernels
  28: 
  29: The jiterator currently has some limitations, however. It cannot:
  30:   - handle math on complex datatypes
  31:   - handle kernels with scalar parameters
  32: 
  33: These improvements will likely come soon.
  34: 
  35: For examples of how to use the jiterator see the i1 and gcd kernel
  36: implementations, which pass jittable strings implementing their
  37: operations instead of the typical CUDA functors.
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 39-43
```cpp
  39: To pass a runtime argument (similar to lambda captures in non-JIT kernels),
  40: we need to pass to additional arguments to `jitted_gpu_kernel` by value.
  41: Currently only primitive C++ types used for computation are valid.
  42: The order of these extra arguments should be same as the order they appear
  43: in kernel's function signature. (look at polygamma for example)
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 45-50
```cpp
  45: NOTE: One big restriction being that these arguments should be after the
  46: arguments provided by TensorIterator. Eg. While capturing `n`, where
  47: `scalar_t x` and `scalar_t y` are provided by TensorIterator,
  48: * foo(scalar_t x, scalar_t y, int n) works!
  49: * foo(int n, scalar_t x, scalar_y) doesn't work
  50: * foo(scalar_t x, int n, scalar_y) doesn't work
```
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 52-61
```cpp
  52: */
  53: 
  54: // Entrypoint for jitted GPU kernels.
  55: // Only handles elementwise unary and binary kernels with a
  56: //   common dtype and a single output.
  57: // NOTE: this assumes the op's iterator has a common_dtype.
  58: // NOTE: We use std::tuple instead of parameter pack
  59: //  for `extra_args` due to following
  60: // bug on older versions of clang
  61: // https://bugs.llvm.org/show_bug.cgi?id=23029
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 62-83
```cpp
  62: template <
  63:     char const* name,
  64:     typename return_type,
  65:     typename f_inputs_type,
  66:     int arity,
  67:     typename... Args>
  68: void jitted_gpu_kernel(
  69:     TensorIteratorBase& iter,
  70:     const std::string& f,
  71:     at::cuda::jit::BinaryFuncVariant scalar_pos =
  72:         at::cuda::jit::BinaryFuncVariant::NoScalar,
  73:     at::opmath_type<f_inputs_type> scalar_val = 0,
  74:     std::tuple<Args...> extra_args = std::make_tuple()) {
  75:   // TODO: much of preamble is common to both jitted_gpu_kernel and gpu_kernel
  76:   //   Maybe it could be refactored?
  77:   for (int arg = 0; arg < iter.ntensors(); arg++) {
  78:     TORCH_INTERNAL_ASSERT(
  79:       iter.device(arg).is_cuda(),
  80:       "argument ", arg, ": expected a CUDA device but found ", iter.device(arg));
  81:   }
  82: 
  83:   if (iter.numel() == 0) {
```
- EN: This block defines or continues the implementation of `jitted_gpu_kernel`.
- CN: 该代码块定义或继续实现 `jitted_gpu_kernel`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 84-85
```cpp
  84:     return;
  85:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 87-94
```cpp
  87:   if (!iter.can_use_32bit_indexing()) {
  88:     for (auto& sub_iter : iter.with_32bit_indexing()) {
  89:       jitted_gpu_kernel<name, return_type, f_inputs_type, arity>(
  90:           sub_iter, f, scalar_pos, scalar_val, extra_args);
  91:     }
  92: 
  93:     return;
  94:   }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 96-101
```cpp
  96:   // Computes if dynamic casting is needed
  97:   // Dynamic casting is needed if an input's dtype differs from the common dtype
  98:   //   or if the result dtype differs from the output's dtype
  99:   // Note: this is intentionally divergent from calling needs_dynamic_casting,
 100:   //   which is more general and inspects a lambda to determine if dynamic
 101:   //   casting is needed.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 102-102
```cpp
 102:   bool needs_dynamic_casting = false;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 104-104
```cpp
 104:   // Checks output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 105-109
```cpp
 105:   const ScalarType return_scalar_type = c10::CppTypeToScalarType<return_type>::value;
 106:   const auto dtype0 = iter.dtype(0);
 107:   if (dtype0 != return_scalar_type) {
 108:     needs_dynamic_casting = true;
 109:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 111-111
```cpp
 111:   // Checks input(s)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 112-133
```cpp
 112:   const ScalarType inputs_scalar_type = c10::CppTypeToScalarType<f_inputs_type>::value;
 113:   for (auto i = decltype(arity){1}; i < (arity + 1); ++i) {
 114:     const auto dtypei = iter.dtype(i);
 115:     if (dtypei != inputs_scalar_type) {
 116:       needs_dynamic_casting = true;
 117:       break;
 118:     }
 119:   }
 120:   if (scalar_pos == at::cuda::jit::BinaryFuncVariant::NoScalar) {
 121:     // NOTE: With `scalar_pos=NoScalar`,`scalar_val` is not used
 122:     // for computation in the generated code and hence we pass a dummy
 123:     // value of `0`.
 124:     jitted_gpu_kernel_impl<
 125:         /*name*/ name,
 126:         /*return_type=*/return_type,
 127:         /*f_inputs_type=*/f_inputs_type,
 128:         arity,
 129:         at::cuda::jit::BinaryFuncVariant::NoScalar>(
 130:         iter, f, needs_dynamic_casting, /*scalar_val=*/scalar_val, extra_args);
 131:   } else if (scalar_pos == at::cuda::jit::BinaryFuncVariant::RhsScalar) {
 132:     jitted_gpu_kernel_impl<
 133:         /*name*/ name,
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 134-135
```cpp
 134:         /*return_type=*/return_type,
 135:         /*f_inputs_type=*/f_inputs_type,
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 136-142
```cpp
 136:         arity,
 137:         at::cuda::jit::BinaryFuncVariant::RhsScalar>(
 138:         iter,
 139:         f,
 140:         needs_dynamic_casting,
 141:         scalar_val,
 142:         extra_args);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 144-157
```cpp
 144:   } else {
 145:     jitted_gpu_kernel_impl<
 146:         /*name*/ name,
 147:         /*return_type=*/return_type,
 148:         /*f_inputs_type=*/f_inputs_type,
 149:         arity,
 150:         at::cuda::jit::BinaryFuncVariant::LhsScalar>(
 151:         iter,
 152:         f,
 153:         needs_dynamic_casting,
 154:         scalar_val,
 155:         extra_args);
 156:   }
 157: }
```
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 159-159
```cpp
 159: // TODO: support runtime state capture similar to `jitted_gpu_kernel`.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 160-181
```cpp
 160: template <char const *name, typename return_type, typename f_inputs_type>
 161: void opmath_jitted_gpu_kernel_with_scalars(TensorIteratorBase& iter, const std::string& f) {
 162:   TORCH_INTERNAL_ASSERT(iter.ntensors() == 3);
 163:   //currently jiterator only handles binary functions where both inputs are of the same type (f_inputs_type)
 164:   using opmath_t = at::opmath_type<f_inputs_type>;
 165:   if (iter.is_cpu_scalar(1)) {
 166:     auto scalar_val = iter.scalar_value<opmath_t>(1);
 167:     iter.remove_operand(1);
 168:     // TODO: When all kernels that use gpu_kernel_with_scalars are
 169:     // ported to structured, this device guard can be deleted.  This
 170:     // works around incorrect device guard generation for pre-structured
 171:     // kernels device guards, but structured kernels do it right and
 172:     // we can assume the device is already set correctly
 173:     const OptionalDeviceGuard device_guard(iter.device(1));
 174:     jitted_gpu_kernel<name, return_type, f_inputs_type, 1>(iter, f, at::cuda::jit::BinaryFuncVariant::LhsScalar, scalar_val);
 175:   } else if (iter.is_cpu_scalar(2)) {
 176:     auto scalar_val = iter.scalar_value<opmath_t>(2);
 177:     iter.remove_operand(2);
 178:     jitted_gpu_kernel<name, return_type, f_inputs_type, 1>(iter, f, at::cuda::jit::BinaryFuncVariant::RhsScalar, scalar_val);
 179:   } else {
 180:     jitted_gpu_kernel<name, return_type, f_inputs_type, 2>(iter, f);
 181:   }
```
- EN: This block defines or continues the implementation of `opmath_jitted_gpu_kernel_with_scalars`.
- CN: 该代码块定义或继续实现 `opmath_jitted_gpu_kernel_with_scalars`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 182-182
```cpp
 182: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 184-184
```cpp
 184: }  // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

### Lines 186-186
```cpp
 186: #endif // AT_USE_JITERATOR()
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `gpu_kernel_with_scalars` handles elementwise CUDA work while folding scalar operands efficiently. / `gpu_kernel_with_scalars` 在高效折叠标量操作数的同时执行逐元素 CUDA 计算。
- `jitted_gpu_kernel` relies on Jiterator-style runtime code generation. / `jitted_gpu_kernel` 依赖 Jiterator 风格的运行时代码生成。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/jit_macros.h>`
  - `<ATen/cuda/CUDAConfig.h>`
  - `<ATen/OpMathType.h>`
  - `<ATen/TensorIterator.h>`
  - `<ATen/native/TensorIteratorDynamicCasting.h>`
  - `<ATen/native/cuda/MemoryAccess.cuh>`
  - `<ATen/native/cuda/CUDAJitLoops.cuh>`
- Runtime symbols / 运行时符号:
  - `gpu_kernel`
  - `gpu_kernel_with_scalars`
  - `jitted_gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `at::cuda::jit::BinaryFuncVariant`
  - `at::cuda::jit::BinaryFuncVariant::NoScalar`
  - `at::cuda::jit::BinaryFuncVariant::RhsScalar`
  - `at::cuda::jit::BinaryFuncVariant::LhsScalar`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
