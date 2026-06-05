# SpectralOpsUtils.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/SpectralOpsUtils.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on spectral ops utils; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 spectral ops utils；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <string>
#include <stdexcept>
#include <sstream>
#include <c10/core/ScalarType.h>
#include <c10/util/ArrayRef.h>
#include <c10/util/Exception.h>
#include <ATen/native/DispatchStub.h>
#include <ATen/core/TensorBase.h>

namespace at::native {

// Normalization types used in _fft_with_size
enum class fft_norm_mode {
  none,       // No normalization
  by_root_n,  // Divide by sqrt(signal_size)
  by_n,       // Divide by signal_size
};

// NOTE [ Fourier Transform Conjugate Symmetry ]
//
// Real-to-complex Fourier transform satisfies the conjugate symmetry. That is,
// assuming X is the transformed K-dimensional signal, we have
//
//     X[i_1, ..., i_K] = X[j_i, ..., j_K]*,
//
//       where j_k  = (N_k - i_k)  mod N_k, N_k being the signal size at dim k,
//             * is the conjugate operator.
//
```
- EN: Lines 1-30 pull in 8 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are sqrt, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 8 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 sqrt，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
// Therefore, in such cases, FFT libraries return only roughly half of the
// values to avoid redundancy:
//
//     X[:, :, ..., :floor(N / 2) + 1]
//
// This is also the assumption in cuFFT and MKL. In ATen SpectralOps, such
// halved signal will also be returned by default (flag onesided=True).
// The following infer_ft_real_to_complex_onesided_size function calculates the
// onesided size from the twosided size.
//
// Note that this loses some information about the size of signal at last
// dimension. E.g., both 11 and 10 maps to 6. Hence, the following
// infer_ft_complex_to_real_onesided_size function takes in optional parameter
// to infer the twosided size from given onesided size.
//
// cuFFT doc: http://docs.nvidia.com/cuda/cufft/index.html#multi-dimensional
// MKL doc: https://software.intel.com/en-us/mkl-developer-reference-c-dfti-complex-storage-dfti-real-storage-dfti-conjugate-even-storage#CONJUGATE_EVEN_STORAGE

inline int64_t infer_ft_real_to_complex_onesided_size(int64_t real_size) {
  return (real_size / 2) + 1;
}

inline int64_t infer_ft_complex_to_real_onesided_size(int64_t complex_size,
                                                      int64_t expected_size=-1) {
  int64_t base = (complex_size - 1) * 2;
  if (expected_size < 0) {
    return base + 1;
  } else if (base == expected_size) {
    return base;
  } else if (base + 1 == expected_size) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are infer_ft_real_to_complex_onesided_size, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 infer_ft_real_to_complex_onesided_size，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-84
```cpp
    return base + 1;
  } else {
    std::ostringstream ss;
    ss << "expected real signal size " << expected_size << " is incompatible "
       << "with onesided complex frequency size " << complex_size;
    TORCH_CHECK(false, ss.str());
  }
}

using fft_fill_with_conjugate_symmetry_fn =
    void (*)(ScalarType dtype, IntArrayRef mirror_dims, IntArrayRef half_sizes,
             IntArrayRef in_strides, const void* in_data,
             IntArrayRef out_strides, void* out_data);
DECLARE_DISPATCH(fft_fill_with_conjugate_symmetry_fn, fft_fill_with_conjugate_symmetry_stub)

// In real-to-complex transform, cuFFT and MKL only fill half of the values
// due to conjugate symmetry. This function fills in the other half of the full
// fft by using the Hermitian symmetry in the signal.
// self should be the shape of the full signal and dims.back() should be the
// one-sided dimension.
// See NOTE [ Fourier Transform Conjugate Symmetry ]
TORCH_API void _fft_fill_with_conjugate_symmetry_(const Tensor& self, IntArrayRef dims);

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are DECLARE_DISPATCH, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 DECLARE_DISPATCH，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: infer_ft_real_to_complex_onesided_size, infer_ft_complex_to_real_onesided_size.
- CN: 重要符号：infer_ft_real_to_complex_onesided_size, infer_ft_complex_to_real_onesided_size。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/core/ScalarType.h, c10/util/ArrayRef.h, c10/util/Exception.h, ATen/native/DispatchStub.h, ATen/core/TensorBase.h`.
- CN: 主要内部头文件：`c10/core/ScalarType.h, c10/util/ArrayRef.h, c10/util/Exception.h, ATen/native/DispatchStub.h, ATen/core/TensorBase.h`。
- EN: External/system headers: `string, stdexcept, sstream`.
- CN: 外部/系统头文件：`string, stdexcept, sstream`。
- EN: The implementation revolves around symbols such as `infer_ft_real_to_complex_onesided_size, infer_ft_complex_to_real_onesided_size`.
- CN: 实现围绕 `infer_ft_real_to_complex_onesided_size, infer_ft_complex_to_real_onesided_size` 等符号展开。
