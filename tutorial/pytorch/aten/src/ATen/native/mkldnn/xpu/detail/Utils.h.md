# Utils.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/Utils.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on utils; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 utils；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once
#include <ATen/ATen.h>
#include <ATen/Tensor.h>
#include <ATen/core/Tensor.h>
#include <iostream>
#include <optional>

#include <ATen/core/grad_mode.h>
#include <c10/core/MemoryFormat.h>
#include <oneapi/dnnl/dnnl.hpp>
#include <oneapi/dnnl/dnnl_graph.hpp>
#include <oneapi/dnnl/dnnl_graph_sycl.hpp>
#include <oneapi/dnnl/dnnl_sycl.hpp>
#include <oneapi/dnnl/dnnl_version.h>

#include <ATen/native/mkldnn/xpu/detail/oneDNNContext.h>

#define ONEDNN_SUPPORT_DETERMINISTIC \
  (DNNL_VERSION_MAJOR >= 3 && DNNL_VERSION_MINOR >= 4)

namespace at::native::onednn {

dnnl::memory::format_tag get_dnnl_default_format(
    int ndims,
    bool is_channels_last = false,
    bool allow_undef = false);

dnnl::memory::data_type get_onednn_dtype(
    const at::Tensor& tensor,
    bool allow_undef = false);
```
- EN: Lines 1-30 pull in 13 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 13 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp

dnnl::memory::data_type get_onednn_dtype_include_double(
    const at::Tensor& tensor,
    bool allow_undef = false);

bool is_supported_onednn_dtype(const at::Tensor& tensor);

dnnl::memory::dims get_onednn_dims(const at::Tensor& tensor);

dnnl::memory::dims get_onednn_strides(const at::Tensor& tensor);
dnnl::memory::desc get_onednn_md(const at::Tensor& tensor);

bool onednn_strides_check(const at::Tensor& src);
bool is_broadcast(const at::Tensor& t);
void undo_broadcast_on_batch(at::Tensor& m1, at::Tensor& m2);
void undo_broadcast(at::Tensor& tensor);

bool is_onednn_matmul_strides(const at::Tensor& tensor);

bool is_64_bytes_aligned(const at::Tensor& tensor);

at::Tensor make_contiguous_and_aligned(
    const at::Tensor& tensor,
    std::optional<at::MemoryFormat> memory_format = std::nullopt);

bool is_broadcast_from_other_to_self(
    const at::Tensor& self,
    const at::Tensor& other);

at::MemoryFormat get_cl_tag_by_ndim(const int64_t ndim);
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp

void apply_tf32_if_allowed(dnnl::primitive_attr& primitive_attr);

bool binary_valid(
    const at::Tensor& self,
    const at::Tensor& other,
    bool is_fusion = false);

bool use_channels_last_for_conv(
    const at::Tensor& src,
    const at::Tensor& weight);

dnnl::memory::format_tag conv_src_fmt(
    const int64_t ndim,
    const bool is_channels_last = false);

dnnl::memory::dims compatible_weight_dims(
    const int64_t ndim,
    const int64_t groups,
    const int64_t oc,
    const int64_t ic,
    const IntArrayRef wsizes);

dnnl::memory::format_tag conv_weight_fmt(
    const int64_t ndim,
    const bool grouped = false,
    const bool is_channels_last = false);

template <typename Vec>
dnnl::memory::dims compatible_dilation(Vec&& dilation) {
```
- EN: The main callable definitions or declarations in this block are compatible_dilation, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段的主要可调用定义或声明包括 compatible_dilation，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
  dnnl::memory::dims ret = dilation.vec();
  for (auto it = ret.begin(); it != ret.end(); it++) {
    *it -= 1;
  }
  return ret;
}

inline std::vector<int64_t> padding_r(
    IntArrayRef padding,
    IntArrayRef output_padding) {
  // ConvTranspose padding adjustment
  //
  // PyTorch uses padding/output_padding:
  //   osize = (isize - 1) * stride - 2 * padding + dilation * (kernel_size - 1)
  //   + output_padding + 1
  //
  // MKLDNN uses padding_l/padding_r:
  //   osize = (isize - 1) * stride - padding_l - padding_r + dilation *
  //   (kernel_size - 1) + 1
  //
  // So: padding_l = padding, padding_r = padding - output_padding
  //
  auto dim = padding.size();
  std::vector<int64_t> pad_r(dim);
  for (const auto d : c10::irange(dim)) {
    pad_r[d] = padding[d] - output_padding[d];
  }
  return pad_r;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 121-150
```cpp
template <typename T>
dnnl::memory dnnl_memory_from_host_scalar(
    T host_value,
    Tensor& holder,
    dnnl::engine& engine) {
  auto options = at::TensorOptions()
                     .dtype(c10::CppTypeToScalarType<T>::value)
                     .device(kXPU);
  holder = at::empty({1}, options).fill_(host_value);
  dnnl::memory::desc md = get_onednn_md(holder);
  dnnl::memory mem = make_onednn_memory(md, engine, holder.data_ptr());
  return mem;
}

struct PartitionCache {
  std::unordered_map<std::bitset<32>, dnnl::graph::partition> partition_map_{};

  // The first 8 bits are reserved
  // bit 0: is int8
  // bit 1: is uint8
  // bit 2: fp16(0) / bf16(1)
  // bit 3: is fp32
  // bit 4: is sdpa pattern
  // bit 5: is sdpa backward pattern
  // bit 6-7: reserved for future use
  // The rest of the bits depend upon the arguments provided
  // However, down the line, we might have different bitsets for different
  // patterns
  enum class BitType : uint8_t {
    Int8 = 0,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TensorOptions, dtype, fp16, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TensorOptions, dtype, fp16，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-174
```cpp
    Uint8 = 1,
    Bfloat16 = 2,
    Float32 = 3,
    SdpaPattern = 4,
    SdpaBwdPattern = 5
  };

  dnnl::graph::partition& insert_partition_cache(
      std::bitset<32>& patternID,
      dnnl::graph::partition& p) {
    partition_map_[patternID] = std::move(p);
    return partition_map_[patternID];
  }
  std::optional<std::reference_wrapper<dnnl::graph::partition>> find_partition(
      std::bitset<32>& patternID) {
    auto iter = partition_map_.find(patternID);
    if (iter != partition_map_.end()) {
      return iter->second;
    }
    return std::nullopt;
  }
};

} // namespace at::native::onednn
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: compatible_dilation, padding_r, dnnl_memory_from_host_scalar, insert_partition_cache, find_partition.
- CN: 重要符号：compatible_dilation, padding_r, dnnl_memory_from_host_scalar, insert_partition_cache, find_partition。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/ATen.h, ATen/Tensor.h, ATen/core/Tensor.h, ATen/core/grad_mode.h, c10/core/MemoryFormat.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`.
- CN: 主要内部头文件：`ATen/ATen.h, ATen/Tensor.h, ATen/core/Tensor.h, ATen/core/grad_mode.h, c10/core/MemoryFormat.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`。
- EN: External/system headers: `iostream, optional, oneapi/dnnl/dnnl.hpp, oneapi/dnnl/dnnl_graph.hpp, oneapi/dnnl/dnnl_graph_sycl.hpp, oneapi/dnnl/dnnl_sycl.hpp, oneapi/dnnl/dnnl_version.h`.
- CN: 外部/系统头文件：`iostream, optional, oneapi/dnnl/dnnl.hpp, oneapi/dnnl/dnnl_graph.hpp, oneapi/dnnl/dnnl_graph_sycl.hpp, oneapi/dnnl/dnnl_sycl.hpp, oneapi/dnnl/dnnl_version.h`。
- EN: The implementation revolves around symbols such as `compatible_dilation, padding_r, dnnl_memory_from_host_scalar, insert_partition_cache, find_partition`.
- CN: 实现围绕 `compatible_dilation, padding_r, dnnl_memory_from_host_scalar, insert_partition_cache, find_partition` 等符号展开。
