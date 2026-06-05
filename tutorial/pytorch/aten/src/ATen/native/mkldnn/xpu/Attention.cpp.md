# Attention.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/Attention.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on attention; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 attention；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/Context.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/transformers/attention.h>
#include <ATen/native/transformers/sdp_utils.h>
#include <ATen/native/transformers/sdp_utils_cpp.h>
#include <ATen/native/transformers/xpu/sdp_utils.h>
#include <c10/util/Array.h>
#include <torch/library.h>

namespace {
bool check_head_dim_size_xpu(sdp::sdp_params const& params, bool debug) {
  const auto query_size_last = params.query.sym_size(-1);
  const auto key_size_last = params.key.sym_size(-1);
  const auto value_size_last = params.value.sym_size(-1);
  if (query_size_last != key_size_last) {
    if (debug) {
      TORCH_WARN(
          "OneDNN attention requires q,k to have the same last dimension.",
          " Got Query.size(-1): ",
          query_size_last,
          ", Key.size(-1): ",
          key_size_last,
          " instead.");
    }
    return false;
  }

  constexpr int MAX_HEAD_DIM = 576;
  const auto max_size_last = query_size_last.max(value_size_last);
  if (max_size_last > MAX_HEAD_DIM) {
    if (debug) {
      TORCH_WARN(
          "OneDNN attention requires q,k,v to have head dimension less than ",
          MAX_HEAD_DIM,
          ". Got ",
          max_size_last,
          " instead.");
    }
    return false;
  }
  return true;
}

bool check_no_grad(sdp::sdp_params const& params, bool debug) {
  const bool any_inputs_require_grad = params.query.requires_grad() ||
      params.key.requires_grad() || params.value.requires_grad();
  const bool gradmode_enabled = at::GradMode::is_enabled();
  if (debug && any_inputs_require_grad && gradmode_enabled) {
    TORCH_WARN("Backward or grad to be supported.");
  }
  return !any_inputs_require_grad || !gradmode_enabled;
}

bool can_use_overrideable_attention(sdp::sdp_params const& params, bool debug) {
  constexpr auto supported_dtypes = c10::array_of<at::ScalarType>(
      at::kFloat, at::kBFloat16, at::kHalf); // double is not supported

  // Define gate functions that determine if a flash kernel can be run
  constexpr auto constraints = c10::array_of<bool (*)(
      sdp::sdp_params const&, bool)>(
```
- EN: Lines 1-60 pull in 8 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 8 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
      sdp::check_nested_tensor,
      sdp::check_for_dropout,
      sdp::check_tensor_shapes,
      sdp::check_batch_size_and_num_heads_dense<true /*supports GQA*/>,
      sdp::check_attn_mask_shape,
      sdp::check_nonzero_sequence_lengths_dense,
      sdp::check_last_dim_stride_equals_1_dense<false /*ignore_singleton_dim*/>,
      check_head_dim_size_xpu,
      check_no_grad);
  for (auto& constraint : constraints) {
    if (!constraint(params, debug)) {
      return false;
    }
  }
  return sdp::check_tensor_dtype(params, supported_dtypes, debug);
}

bool can_use_cudnn_attention(sdp::sdp_params const& params, bool debug) {
  if (debug) {
    TORCH_WARN("XPU don't support SDPA cudnn attention backend.");
  }
  return false;
}

int64_t minimum_gemm_alignment(sdp::sdp_params const& params) {
  bool is_half = (params.query.dtype() == at::kHalf) ||
      (params.query.dtype() == at::kBFloat16);
  int64_t matmul_alignment_mn = 4;
  int64_t bits_per_scalar = is_half ? 16 : 32;
  matmul_alignment_mn = std::max(matmul_alignment_mn, 128 / bits_per_scalar);

  return matmul_alignment_mn;
}

bool check_head_dim_size_mem_efficient(
    sdp::sdp_params const& params,
    bool debug) {
  const auto query_size_last = params.query.sym_size(-1);
  const auto value_size_last = params.value.sym_size(-1);
  const int64_t alignment = minimum_gemm_alignment(params);
  if (!(query_size_last == params.key.sym_size(-1) &&
        query_size_last % alignment == 0 && query_size_last > 0 &&
        value_size_last % alignment == 0 && value_size_last > 0)) {
    if (debug) {
      TORCH_WARN(
          "Mem efficient attention requires last dimension of inputs to be divisible by ",
          alignment,
          ". ",
          "Got Query.size(-1): ",
          query_size_last,
          ", Key.size(-1): ",
          params.key.sym_size(-1),
          ", Value.size(-1): ",
          params.value.sym_size(-1),
          " instead.");
    }
    return false;
  }
  return true;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are can_use_cudnn_attention, minimum_gemm_alignment, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 can_use_cudnn_attention, minimum_gemm_alignment，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp

bool can_use_mem_efficient_attention(
    sdp::sdp_params const& params,
    bool debug) {
  // Define gate functions that determine if a mem efficient can be run
  constexpr auto general_constraints =
      c10::array_of<bool (*)(sdp::sdp_params const&, bool)>(
          sdp::check_runtime_disabled_mem_efficient,
          sdp::check_tensor_shapes,
          check_head_dim_size_mem_efficient);
  for (auto& constraint : general_constraints) {
    if (!constraint(params, debug)) {
      return false;
    }
  }
  if (has_for_nested_inputs(params)) {
    constexpr auto nested_constraints =
        c10::array_of<bool (*)(sdp::sdp_params const&, bool)>(
            sdp::check_requires_grad_and_nested,
            sdp::check_batch_size_nested,
            sdp::check_for_seq_len_0_nested_tensor);
    for (auto& constraint : nested_constraints) {
      if (!constraint(params, debug)) {
        return false;
      }
    }
  }
  if (has_only_dense_inputs(params)) {
    constexpr auto dense_constraints =
        c10::array_of<bool (*)(sdp::sdp_params const&, bool)>(
            sdp::check_nonzero_sequence_lengths_dense,
            sdp::check_last_dim_stride_equals_1_dense<false>,
            sdp::check_batch_size_and_num_heads_dense<false>);
    for (auto& constraint : dense_constraints) {
      if (!constraint(params, debug)) {
        return false;
      }
    }
  }
  return true;
}

bool priority_order_init = false;

std::array<sdp::SDPBackend, sdp::num_backends> priority_order(
    sdp::sdp_params const& params) {
  if (!priority_order_init) {
    priority_order_init = true;
    const std::vector<int64_t> priority_order = {
        static_cast<int64_t>(at::SDPBackend::overrideable),
        static_cast<int64_t>(at::SDPBackend::flash_attention),
        static_cast<int64_t>(at::SDPBackend::math),
        static_cast<int64_t>(at::SDPBackend::efficient_attention),
        static_cast<int64_t>(at::SDPBackend::cudnn_attention)};
    at::globalContext().setSDPPriorityOrder(priority_order);
  }
  return at::globalContext().sDPPriorityOrder();
}

sdp::SDPBackend select_sdp_backend_xpu(sdp::sdp_params const& kernel_params) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are select_sdp_backend_xpu, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 select_sdp_backend_xpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-240
```cpp
  // This function defines the priority order of the different sdp backends
  // 1. Flash Attention
  // 2. Math fallback
  auto& ctx = at::globalContext();
  // use overridable linked to onednn as overridable implementation
  if (!ctx.userEnabledMathSDP() && !ctx.userEnabledOverrideableSDP() &&
      !ctx.userEnabledFlashSDP() && !ctx.userEnabledMemEfficientSDP()) {
    return sdp::SDPBackend::error;
  }

  // Get ideal kernel ordering
  const auto ordering = priority_order(kernel_params);

  // Because TORCHCHECK checks if condition is true we negate debug so that
  // The statements will be printed when debug is true
  bool print_debug = false;
  for (auto& backend : ordering) {
    switch (backend) {
      case sdp::SDPBackend::overrideable:
        if (ctx.userEnabledOverrideableSDP() &&
            can_use_overrideable_attention(kernel_params, print_debug)) {
          return sdp::SDPBackend::overrideable;
        }
        break;
      case sdp::SDPBackend::math:
        if (ctx.userEnabledMathSDP()) {
          return sdp::SDPBackend::math;
        }
        break;
      case sdp::SDPBackend::flash_attention:
        if (ctx.userEnabledFlashSDP() &&
            sdp::can_use_flash_attention(kernel_params, print_debug)) {
          return sdp::SDPBackend::flash_attention;
        }
        break;
      case sdp::SDPBackend::cudnn_attention:
        if (ctx.userEnabledCuDNNSDP() &&
            can_use_cudnn_attention(kernel_params, print_debug)) {
          TORCH_CHECK(false, "Invalid backend");
        }
        break;
      case sdp::SDPBackend::efficient_attention:
        if (ctx.userEnabledMemEfficientSDP() &&
            can_use_mem_efficient_attention(kernel_params, print_debug)) {
          TORCH_WARN_ONCE(
              "SDPA Memory Efficient Attention backend is not supported on XPU, falling back to math backend.");
          return sdp::SDPBackend::math;
        }
        break;
      default:
        TORCH_CHECK(false, "Invalid backend");
    }
  }
  // If we have gotten to this point then two things have happened:
  // 1. can_use_overridable_attention did not satisfy the constraints to be ran
  // 2. The user has explicitly disabled the math kernel
  // We then re-run the kernel checks with debug enabled to print out the
  // reason why the kernel was not selected

  print_debug = true;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are userEnabledFlashSDP, can_use_overrideable_attention, can_use_flash_attention, can_use_cudnn_attention, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 userEnabledFlashSDP, can_use_overrideable_attention, can_use_flash_attention, can_use_cudnn_attention，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
  TORCH_WARN("Flash attention kernel not used because:");
  sdp::can_use_flash_attention(kernel_params, print_debug);
  TORCH_WARN("Overrideable attention kernel not used because:");
  can_use_overrideable_attention(kernel_params, print_debug);
  TORCH_WARN("CuDNN attention kernel not used because:");
  can_use_cudnn_attention(kernel_params, print_debug);
  TORCH_WARN("Memory Efficient attention kernel not used because:");
  can_use_mem_efficient_attention(kernel_params, print_debug);
  TORCH_CHECK(!print_debug, "No available kernel. Aborting execution.")
  return sdp::SDPBackend::error;
}
} // namespace

namespace at::native {
int64_t _fused_sdp_choice_xpu(
    const at::Tensor& query_,
    const at::Tensor& key,
    const at::Tensor& value,
    const std::optional<at::Tensor>& attn_mask_,
    double dropout_p,
    bool is_causal,
    std::optional<double> scale,
    bool enable_gqa) {
  sdp::sdp_params kernel_params{
      query_, key, value, attn_mask_, dropout_p, is_causal, enable_gqa};
  auto backend = select_sdp_backend_xpu(kernel_params);

  if (backend == sdp::SDPBackend::error) {
    TORCH_CHECK(
        false,
        "No viable backend for scaled_dot_product_attention was found. ",
        "This is likely due to turning off both the math kernel and the overrideable kernels.");
  }
  return static_cast<int64_t>(backend);
}

std::tuple<
    at::Tensor,
    at::Tensor,
    at::Tensor,
    at::Tensor,
    c10::SymInt,
    c10::SymInt,
    at::Tensor,
    at::Tensor,
    at::Tensor>
_scaled_dot_product_fused_attention_overrideable_xpu(
    const at::Tensor& query,
    const at::Tensor& key,
    const at::Tensor& value,
    const std::optional<at::Tensor>& attn_bias,
    double dropout_p,
    bool is_causal,
    bool return_debug_mask,
    std::optional<double> scale) {
  TORCH_INTERNAL_ASSERT(
      query.dim() == 4 && key.dim() == 4 && value.dim() == 4,
      "scaled_dot_product_fused_attention_overrideable_xpu: Accept only 4 dims inputs shape of {(B), H, T, K}");
  TORCH_INTERNAL_ASSERT(
      (key.size(0) == value.size(0)) && (key.size(1) == value.size(1)) &&
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_CHECK, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_CHECK，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-360
```cpp
          (key.size(2) == value.size(2)),
      "scaled_dot_product_fused_attention_overrideable_xpu: K/V should have the same batch / seq / num_head");
  TORCH_INTERNAL_ASSERT(
      query.size(3) == key.size(3),
      "scaled_dot_product_fused_attention_overrideable_xpu: Q/K should have the same head_dim");
  TORCH_INTERNAL_ASSERT(
      query.size(1) % key.size(1) == 0,
      "scaled_dot_product_fused_attention_overrideable_xpu: number of heads in K/V must divide number of heads in Q");
  TORCH_INTERNAL_ASSERT(
      dropout_p == 0.0,
      "scaled_dot_product_fused_attention_overrideable_xpu: Currently do not support dropout > 0");
  TORCH_INTERNAL_ASSERT(
      !(attn_bias.has_value() && is_causal),
      "scaled_dot_product_fused_attention_overrideable_xpu: attn_bias cannot present with is_causal");

  const int64_t batch_size = query.size(0);
  const int64_t num_head_q = query.size(1);
  const int64_t num_head_kv = key.size(1);
  const int64_t head_dim_qk = query.size(3);
  const int64_t head_dim_v = value.size(3);
  const int64_t seq_len_q = query.size(2);
  const int64_t seq_len_kv = key.size(2);

  at::Tensor output;
  std::vector<int64_t> output_shape = {
      batch_size, num_head_q, seq_len_q, head_dim_v};
  alloc_with_matching_layout(query, output, output_shape);
  at::Tensor logsumexp, debug_attn_mask; // not supported

  at::native::onednn::sdpa(
      batch_size,
      seq_len_q,
      seq_len_kv,
      num_head_q,
      num_head_kv,
      head_dim_qk,
      head_dim_v,
      query,
      key,
      value,
      attn_bias,
      is_causal,
      scale.has_value() ? scale.value() : (1.0 / std::sqrt(head_dim_qk)),
      output,
      false,
      logsumexp);

  // rng not used
  auto philox_seed = at::empty({}, at::dtype(at::kLong));
  auto philox_offset = at::empty({}, at::dtype(at::kLong));
  return std::make_tuple(
      output,
      logsumexp,
      /* cum_seq_q */ at::Tensor(),
      /* cum_seq_k */ at::Tensor(),
      seq_len_q,
      seq_len_kv,
      philox_seed,
      philox_offset,
      debug_attn_mask);
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 361-364
```cpp
}

REGISTER_XPU_DISPATCH(_fused_sdp_choice_stub, &_fused_sdp_choice_xpu);
} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention, can_use_cudnn_attention, minimum_gemm_alignment, check_head_dim_size_mem_efficient, can_use_mem_efficient_attention, priority_order.
- CN: 重要符号：check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention, can_use_cudnn_attention, minimum_gemm_alignment, check_head_dim_size_mem_efficient, can_use_mem_efficient_attention, priority_order。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Context.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/transformers/attention.h, ATen/native/transformers/sdp_utils.h, ATen/native/transformers/sdp_utils_cpp.h, ATen/native/transformers/xpu/sdp_utils.h, c10/util/Array.h, torch/library.h`.
- CN: 主要内部头文件：`ATen/Context.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/transformers/attention.h, ATen/native/transformers/sdp_utils.h, ATen/native/transformers/sdp_utils_cpp.h, ATen/native/transformers/xpu/sdp_utils.h, c10/util/Array.h, torch/library.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention, can_use_cudnn_attention, minimum_gemm_alignment, check_head_dim_size_mem_efficient, can_use_mem_efficient_attention, priority_order, select_sdp_backend_xpu, TORCH_CHECK`.
- CN: 实现围绕 `check_head_dim_size_xpu, check_no_grad, can_use_overrideable_attention, can_use_cudnn_attention, minimum_gemm_alignment, check_head_dim_size_mem_efficient, can_use_mem_efficient_attention, priority_order, select_sdp_backend_xpu, TORCH_CHECK` 等符号展开。
