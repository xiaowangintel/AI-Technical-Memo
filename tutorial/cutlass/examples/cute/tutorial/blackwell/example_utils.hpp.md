# example_utils.hpp — Code Analysis / 代码分析

**Source / 源文件**: `examples/cute/tutorial/blackwell/example_utils.hpp`

**Purpose / 用途**: Shared helper utilities (reference GEMM, numerical comparison, tensor initialization) used by all three SM100 tutorial examples / 为所有三个 SM100 示例提供共享辅助工具（参考 GEMM 实现、数值比较、张量初始化）

---

## Line-by-Line Analysis / 逐行分析

```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 * ...
 **************************************************************************************************/
```

**EN**: Standard NVIDIA copyright block. BSD-3-Clause license applies to all tutorial code in this file.  
**CN**: 标准 NVIDIA 版权声明块，本文件中所有示例代码均适用 BSD-3-Clause 许可证。

---

```cpp
#pragma once
```

**EN**: Include guard using the `#pragma once` directive — ensures the header is processed only once per translation unit, even if included from multiple source files.  
**CN**: 使用 `#pragma once` 作为头文件保护，确保每个翻译单元只处理一次该头文件，即使被多个源文件包含也不会重复。

---

```cpp
#include <cute/tensor.hpp>                    // CuTe tensor implementation
#include <cute/arch/copy_sm90_desc.hpp>
```

**EN**: Brings in the CuTe tensor abstraction layer (shapes, strides, layouts, slicing) and SM90 copy descriptors. The SM90 descriptor utilities are reused on SM100 for tensor-print helpers.  
**CN**: 引入 CuTe 张量抽象层（形状、步幅、布局、切片）以及 SM90 复制描述符。SM90 描述符工具在 SM100 上被复用于张量打印等辅助功能。

---

```cpp
template <class AccType,
          class TensorA, class TensorB,
          class TensorC, class TensorD,
          class Alpha, class Beta>
void
reference_gemm(TensorA const& tensor_A, TensorB const& tensor_B,
               TensorC const& tensor_C, TensorD      & tensor_D,
               Alpha alpha, Beta beta)
```

**EN**: Declares the host-side **reference GEMM** function template. It is fully generic over the accumulator type (`AccType`) and the four tensor types, plus the scaling scalars `alpha` and `beta`. This runs on the CPU and produces a ground-truth result for correctness checking.  
**CN**: 声明主机端**参考 GEMM** 函数模板。对累加器类型（`AccType`）、四个张量类型以及缩放标量 `alpha`/`beta` 完全泛化。该函数在 CPU 上运行，产生用于正确性验证的参考结果。

---

```cpp
{
  using namespace cute;
  for (int m = 0; m < size<0>(tensor_D); ++m) {
    for (int n = 0; n < size<1>(tensor_D); ++n) {
      AccType c = AccType(0.f);
      for (int k = 0; k < size<1>(tensor_A); ++k) {
        c += tensor_A(m,k) * tensor_B(n,k);
      }
      tensor_D(m,n) = alpha * c + beta * tensor_C(m,n);
    }
  }
}
```

**EN**: Implements a naïve triple-loop GEMM: D(m,n) = alpha * sum_k(A(m,k) * B(n,k)) + beta * C(m,n). Note that B is indexed as B(n,k), matching the **K-major (transposed)** layout used in the tutorials (both A and B are K-major, i.e., BLAS "T" and "N" respectively when viewed column-major). `size<0>(tensor_D)` and `size<1>(tensor_D)` use CuTe's layout-aware size queries.  
**CN**: 实现了一个简单的三重循环 GEMM：D(m,n) = alpha * sum_k(A(m,k) * B(n,k)) + beta * C(m,n)。注意 B 以 B(n,k) 索引，与教程中使用的 **K-major（转置）** 布局匹配。`size<0>(tensor_D)` 和 `size<1>(tensor_D)` 使用 CuTe 的布局感知尺寸查询。

---

```cpp
template <class TensorA, class TensorB,
          class TensorC, class TensorD,
          class RefTensorD>
bool
compare_results(TensorA const& tensor_A, TensorB const& tensor_B,
                TensorC const& tensor_C, TensorD const& tensor_D,
                RefTensorD const& ref_tensor_D,
                bool print_diff = false)
```

**EN**: Declares the **numerical comparison** function template. Takes five CuTe tensors (A, B, C, computed D, reference D) and a flag to optionally print element-wise differences.  
**CN**: 声明**数值比较**函数模板。接受五个 CuTe 张量（A、B、C、计算所得 D、参考 D），以及一个可选的逐元素差值打印标志。

---

```cpp
{
  using namespace cute;
  auto norm_A     = matrix_inf_norm(tensor_A);
  auto norm_B     = matrix_inf_norm(tensor_B);
  auto norm_C     = matrix_inf_norm(tensor_C);
  auto norm_D     = matrix_inf_norm(tensor_D);
  auto norm_ref_D = matrix_inf_norm(ref_tensor_D);
  auto norm_diff  = matrix_diff_inf_norm(tensor_D, ref_tensor_D);
```

**EN**: Computes the **infinity norm** (max absolute element value) for each matrix and for the element-wise difference D − ref_D. These norms are used both for printing diagnostics and for pass/fail judgement.  
**CN**: 计算每个矩阵及 D − ref_D 逐元素差值的**无穷范数**（最大绝对元素值）。这些范数既用于打印诊断信息，也用于通过/失败判断。

---

```cpp
  if (print_diff) {
    for (int m = 0; m < size<0>(tensor_D); ++m) {
      for (int n = 0; n < size<1>(tensor_D); ++n) {
        std::cout << m << "," << n << " : "
                  << tensor_D(m,n) << " vs. " << ref_tensor_D(m,n) << std::endl;
      }
    }
  }
```

**EN**: Optional verbose mode: iterates every element position (m,n) and prints both the computed and reference values side-by-side. Useful during debugging to pinpoint which tiles are wrong.  
**CN**: 可选的详细模式：遍历每个元素位置 (m,n) 并并排打印计算值和参考值，在调试时有助于定位错误的 tile。

---

```cpp
  std::cout << "norm (A)       : " << norm_A.inf_norm       << std::endl;
  std::cout << "norm (B)       : " << norm_B.inf_norm       << std::endl;
  std::cout << "norm (C)       : " << norm_C.inf_norm       << std::endl;
  std::cout << "norm (D)       : " << norm_D.inf_norm       << std::endl;
  std::cout << "norm (ref_D)   : " << norm_ref_D.inf_norm   << std::endl;
  std::cout << "norm (D-ref_D) : " << norm_diff.inf_norm    << std::endl;
```

**EN**: Prints all six norms to stdout so that the user can immediately see whether the computed result is numerically close to the reference.  
**CN**: 将六个范数全部输出到标准输出，以便用户立即判断计算结果是否在数值上接近参考值。

---

```cpp
  return (!norm_A.found_nan) && (!norm_B.found_nan) &&
         (!norm_C.found_nan) && (!norm_D.found_nan) && (!norm_ref_D.found_nan) &&
         (norm_A.inf_norm > 0.0) && (norm_B.inf_norm > 0.0) &&
         (norm_C.inf_norm > 0.0) && (norm_D.inf_norm > 0.0) && (norm_ref_D.inf_norm > 0.0) &&
         (norm_diff.inf_norm <= 0.0);
}
```

**EN**: Three-condition pass criterion:  
1. **No NaNs** in any of the five tensors.  
2. **All tensors non-zero** (guards against trivially passing with all-zero inputs).  
3. **Zero difference norm** (computed D equals reference D exactly, as expected for integer-valued test inputs).  

**CN**: 三重通过判断标准：  
1. 五个张量中**均无 NaN**。  
2. **所有张量均非零**（防止全零输入导致假阳性通过）。  
3. **差值范数为零**（对于整数值测试输入，计算所得 D 应精确等于参考 D）。

---

```cpp
template <class Tensor>
void
initialize_tensor(Tensor& tensor, cute::tuple<int, int> value_range = {-2, 2})
{
  using DataType = typename Tensor::element_type;
  auto [min, max] = value_range;
  for (int i = 0; i < cute::size(tensor); i++) {
    tensor(i) = DataType(int((max-min)*(rand() / double(RAND_MAX)) + min));
  }
}
```

**EN**: Fills any CuTe tensor with **random integer values** in `[min, max)` (default `-2` to `2`). The cast chain `DataType(int(...))` converts the double random value to an integer then to the target type (e.g. `cutlass::half_t`). Small integer values are chosen deliberately so that the reference GEMM and the GPU GEMM produce exactly the same result (no floating-point rounding ambiguity).  
**CN**: 用 `[min, max)` 范围内（默认 `-2` 到 `2`）的**随机整数值**填充任意 CuTe 张量。类型转换链 `DataType(int(...))` 先将 double 随机值转换为整数，再转为目标类型（如 `cutlass::half_t`）。特意选择小整数值，确保参考 GEMM 与 GPU GEMM 产生完全相同的结果（无浮点舍入歧义）。

---

## Key Concepts / 关键概念

- **CuTe `matrix_inf_norm` / `matrix_diff_inf_norm`** — Layout-aware infinite-norm helpers for numerical validation / 布局感知的无穷范数辅助函数，用于数值验证
- **Reference GEMM** — CPU triple-loop GEMM serves as correctness oracle / CPU 三重循环 GEMM 作为正确性参考
- **Integer-valued random initialization** — Eliminates rounding differences between reference and GPU results / 整数值随机初始化消除参考与 GPU 结果间的舍入差异
- **CuTe generic tensor indexing** — `tensor(m,k)` and `tensor(i)` work for any layout / CuTe 泛型张量索引，对任意布局均有效
- **`cute::size(tensor)`** — Returns total number of elements regardless of layout / 返回张量元素总数，与布局无关

## Dependencies / 依赖项

- `<cute/tensor.hpp>` — Core CuTe tensor abstractions (Layout, Tensor, size, cosize) / CuTe 核心张量抽象
- `<cute/arch/copy_sm90_desc.hpp>` — SM90-era descriptor helpers (reused for norm/print utilities) / SM90 描述符辅助工具（复用于范数/打印）
