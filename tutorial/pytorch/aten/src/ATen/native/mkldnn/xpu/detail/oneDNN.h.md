# oneDNN.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/oneDNN.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on one dnn; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 one dnn；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <ATen/ATen.h>
#include <ATen/BlasBackend.h>
#include <ATen/native/ScaledBlasUtils.h>
#include <ATen/native/mkldnn/xpu/detail/Attr.h>
#include <ATen/native/mkldnn/xpu/detail/Utils.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNNContext.h>

namespace at::native::onednn {

TORCH_API sycl::event matmul(
    at::Tensor& result,
    const at::Tensor& mat1,
    const at::Tensor& mat2,
    const at::Tensor& b_raw,
    bool m2_trans,
    Attr attr,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event convolution(
    at::Tensor& dst,
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& bia,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
```
- EN: Lines 1-30 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
    Attr& attr,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event convolution_backward_weights(
    at::Tensor& diff_weight,
    at::Tensor& diff_bia,
    const at::Tensor& diff_dst,
    const at::Tensor& src,
    IntArrayRef diff_weight_aten_size,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event convolution_backward_data(
    at::Tensor& diff_src,
    const at::Tensor& diff_dst,
    const at::Tensor& weight,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation,
    int64_t groups,
    bool bias_defined,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event deconvolution(
    at::Tensor& dst,
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
    const at::Tensor& src,
    const at::Tensor& weight,
    const at::Tensor& bia,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    Attr& attr,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event deconvolution_backward_data(
    at::Tensor& diff_src,
    const at::Tensor& diff_dst,
    const at::Tensor& weight,
    IntArrayRef stride,
    IntArrayRef padding,
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    bool bias_defined,
    const std::vector<sycl::event>& deps = {});

TORCH_API sycl::event deconvolution_backward_weights(
    at::Tensor& diff_weight,
    at::Tensor& diff_bia,
    const at::Tensor& diff_dst,
    const at::Tensor& src,
    IntArrayRef stride,
    IntArrayRef padding,
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    IntArrayRef dst_padding,
    IntArrayRef dilation,
    int64_t groups,
    const std::vector<sycl::event>& deps = {});

TORCH_API void woq_matmul_int4(
    at::Tensor& result, // dst, [M, N]
    const at::Tensor& mat1_, // src, [M, K]
    const at::Tensor& mat2_, // quantized weight, [K/8, N]
    const at::Tensor& scale, // [K/group_size, N]
    const at::Tensor& zp, // [k/group_size, N]
    int64_t group_size,
    bool pri_cache = true);

dnnl::memory::dims conv_dst_size(
    int64_t ndim,
    IntArrayRef src_tz,
    IntArrayRef wgh_tz,
    IntArrayRef padding_front_top_left,
    IntArrayRef padding_back_bottom_right,
    IntArrayRef stride,
    IntArrayRef dilation);

dnnl::memory::dims deconv_dst_size(
    IntArrayRef src_size,
    IntArrayRef wgh_size,
    IntArrayRef padding,
    IntArrayRef stride,
    IntArrayRef dilation,
    IntArrayRef dst_padding,
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
    int64_t groups);

at::Tensor quantized_convolution(
    at::Tensor act,
    double act_scale,
    int64_t act_zero_point,
    at::Tensor weight,
    at::Tensor weight_scales,
    at::Tensor weight_zero_points,
    std::optional<at::Tensor> bias,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    bool transposed,
    int64_t groups,
    at::Tensor output,
    double inv_output_scale,
    int64_t output_zero_point,
    std::optional<at::Tensor> accum,
    double accum_scale,
    int64_t accum_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::optional<std::string_view> binary_attr,
    std::optional<at::Scalar> binary_alpha,
    std::optional<std::string_view> unary_attr,
    torch::List<std::optional<at::Scalar>> unary_scalars,
    std::optional<std::string_view> unary_algorithm);

void quantized_matmul(
    at::Tensor mat1, // act
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
    double input_scale,
    int64_t input_zero_point,
    at::Tensor mat2, // weight
    at::Tensor& weight_scales,
    at::Tensor& weight_zero_points,
    at::Tensor& b_raw,
    at::Tensor result, // output
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::optional<at::Tensor> other, // extra input for binary-post-op
    double other_scale,
    int64_t other_zero_point,
    const std::string_view& binary_post_op,
    double binary_alpha,
    const std::string_view& unary_post_op,
    torch::List<std::optional<at::Scalar>>& unary_post_op_args,
    std::string_view unary_post_op_algorithm,
    bool m2_trnas);

void sdpa(
    int batch_size,
    int seq_len_q,
    int seq_len_kv,
    int num_head_q,
    int num_head_kv,
    int head_dim_qk,
    int head_dim_v,
    const Tensor& query,
    const Tensor& key,
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-210
```cpp
    const Tensor& value,
    std::optional<at::Tensor> attn_mask,
    bool is_causal,
    float softmax_scale,
    const Tensor& attention,
    bool compute_logsumexp,
    const Tensor& logsumexp);

void sdpa_backward(
    int batch_size,
    int num_head_q,
    int num_head_kv,
    int seq_len_q,
    int seq_len_kv,
    int head_dim_qk,
    int head_dim_v,
    const Tensor& grad_out,
    const Tensor& query,
    const Tensor& key,
    const Tensor& value,
    const Tensor& out,
    const Tensor& logsumexp,
    std::optional<at::Tensor> attn_mask,
    bool is_causal,
    float softmax_scale,
    Tensor& grad_query,
    Tensor& grad_key,
    Tensor& grad_value);

sycl::event scaled_matmul(
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 211-232
```cpp
    const Tensor& mat1,
    const Tensor& mat2,
    Tensor& result,
    const Tensor& scale_a,
    const Tensor& scale_b,
    at::blas::ScalingType scaling_choice_a,
    at::blas::ScalingType scaling_choice_b,
    const std::optional<at::Tensor>& bias,
    const std::optional<at::Tensor>& scale_result,
    bool use_fast_accum);

sycl::event scaled_grouped_matmul(
    const Tensor& mat_a,
    const Tensor& mat_b,
    const std::optional<Tensor> scale_a,
    const std::optional<Tensor> scale_b,
    const std::optional<at::blas::ScalingType> scaling_choice_a,
    const std::optional<at::blas::ScalingType> scaling_choice_b,
    const Tensor& offs,
    Tensor& out,
    const std::optional<Tensor> alpha = std::nullopt);
} // namespace at::native::onednn
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/ATen.h, ATen/BlasBackend.h, ATen/native/ScaledBlasUtils.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`.
- CN: 主要内部头文件：`ATen/ATen.h, ATen/BlasBackend.h, ATen/native/ScaledBlasUtils.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/Utils.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
