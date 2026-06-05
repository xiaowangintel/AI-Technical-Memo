# SortImpl.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/SortImpl.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `infer_dense_strides_dim_last`.
- 用途（中文）: 实现与 `infer_dense_strides_dim_last` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <thrust/execution_policy.h>
   4: #include <thrust/sort.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/core/Tensor.h>`, `<thrust/execution_policy.h>`, `<thrust/sort.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/core/Tensor.h>`, `<thrust/execution_policy.h>`, `<thrust/sort.h>`。

### Lines 6-27
```cpp
   6: namespace at::native {
   7: 
   8: std::vector<int64_t> infer_dense_strides_dim_last(const Tensor & self, int64_t dim) {
   9:   int64_t ndim = self.dim();
  10:   // sort the strides in descending order according to its value,
  11:   // keeping dim the last.
  12:   std::vector<int64_t> strides = self.strides().vec();
  13:   strides[dim] = -1;
  14:   std::vector<int64_t> original_dim(ndim);
  15:   for (int64_t i = 0; i < ndim; i++) {
  16:     original_dim[i] = i;
  17:   }
  18:   thrust::stable_sort_by_key(
  19:     thrust::host, strides.data(), strides.data() + ndim, original_dim.data(),
  20:     thrust::greater<int64_t>()
  21:   );
  22:   // generate contiguous strides on permuted dims
  23:   std::vector<int64_t> new_strides(ndim);
  24:   std::vector<int64_t> new_strides_unsort(ndim);
  25:   int64_t cumprod = 1;
  26:   for (int64_t i = 0; i < ndim; i++) {
  27:     new_strides[ndim - 1 - i] = cumprod;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `infer_dense_strides_dim_last`.
- CN: 该代码块定义或继续实现 `infer_dense_strides_dim_last`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 28-35
```cpp
  28:     cumprod *= self.sizes()[original_dim[ndim - 1 - i]];
  29:   }
  30:   // unsort new strides
  31:   for (int64_t i = 0; i < ndim; i++) {
  32:     new_strides_unsort[original_dim[i]] = new_strides[i];
  33:   }
  34:   return new_strides_unsort;
  35: }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 37-37
```cpp
  37: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- Thrust utilities supply STL-like CUDA algorithms and containers. / Thrust 工具提供类似 STL 的 CUDA 算法与容器。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/core/Tensor.h>`
  - `<thrust/execution_policy.h>`
  - `<thrust/sort.h>`
- Runtime symbols / 运行时符号:
  - `thrust::stable_sort_by_key`
  - `thrust::host`
  - `thrust::greater`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
