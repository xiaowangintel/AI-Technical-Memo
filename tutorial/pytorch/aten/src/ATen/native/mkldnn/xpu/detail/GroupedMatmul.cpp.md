# GroupedMatmul.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/GroupedMatmul.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on grouped matmul; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 grouped matmul；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/Tensor.h>
#include <ATen/core/Tensor.h>
#include <c10/core/ScalarType.h>

#include <ATen/native/mkldnn/xpu/detail/Attr.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNNContext.h>
#include <ATen/native/mkldnn/xpu/detail/Utils.h>

#include <oneapi/dnnl/dnnl.hpp>

namespace at::native::onednn {

// Describes how to configure oneDNN scales for a given role/ScalingType.
// When use_mask_only is true, apply via set_scales_mask(arg, mask).
// Otherwise, apply via set_scales(arg, mask, groups, dtype).
struct ScaleSpec {
  int mask;
  dnnl::memory::dims groups;
  dnnl::memory::data_type dtype;
  bool use_mask_only;

  void apply(dnnl::primitive_attr& attr, int arg) const {
    if (use_mask_only) {
      attr.set_scales_mask(arg, mask);
    } else {
      attr.set_scales(arg, mask, groups, dtype);
    }
  }
};
```
- EN: Lines 1-30 pull in 8 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are apply, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 8 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 apply，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp

inline ScaleSpec make_scale_spec(
    at::blas::ScalingType scaling_type,
    int64_t K,
    const std::string& arg_type,
    bool a_is_2d,
    bool b_is_2d) {
  TORCH_CHECK(
      arg_type == "src" || arg_type == "wei",
      "Expected arg_type to be 'src' or 'wei', but got '",
      arg_type,
      "'");
  TORCH_CHECK(a_is_2d && !b_is_2d, "Currently only 2d x 3d grouped matmul with offsets is supported");
  bool is_src = (arg_type == "src");

  switch (scaling_type) {
    case at::blas::ScalingType::RowWise:
      // For FP8 rowwise with grouped matmul, use set_scales_mask (no groups)
      // following the official grouped matmul example:
      //   src: mask=(1<<0) -> per-token scale, shape [M]
      //   wei: mask=(1<<0)|(1<<2) -> per-group-per-N scale, shape [G, N]
      return {
        is_src ? (1 << 0) : (1 << 0) | (1 << 2),
        {},
        dnnl::memory::data_type::f32,
        /*use_mask_only=*/true};
    case at::blas::ScalingType::BlockWise1x32:
      return {
        is_src ? (1 << 0) | (1 << 1) : (1 << 0) | (1 << 1) | (1 << 2),
        is_src ? dnnl::memory::dims{1, 32} : dnnl::memory::dims{32, 1},
```
- EN: The main callable definitions or declarations in this block are set_scales_mask, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段的主要可调用定义或声明包括 set_scales_mask，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
        dnnl::memory::data_type::e8m0,
        /*use_mask_only=*/false};
    case at::blas::ScalingType::BlockWise1x16:
      return {
        is_src ? (1 << 0) | (1 << 1) : (1 << 0) | (1 << 1) | (1 << 2),
        is_src ? dnnl::memory::dims{1, 16} : dnnl::memory::dims{16, 1},
        dnnl::memory::data_type::f8_e4m3,
        /*use_mask_only=*/false};
    default:
      TORCH_CHECK(false, "Unknown scaling_type: ", static_cast<int>(scaling_type));
  }
}

std::tuple<dnnl::memory::desc, dnnl::memory::desc, dnnl::memory::desc> get_grouped_gemm_md(
    int64_t M,
    int64_t N,
    int64_t K,
    int64_t group_count,
    dnnl::memory::data_type dtype,
    dnnl::memory::data_type dst_dtype,
    bool a_is_2d,
    bool b_is_2d) {
  dnnl::memory::desc src_md, weights_md, dst_md;
  if (a_is_2d && !b_is_2d){
    src_md = dnnl::memory::desc::grouped(
        {M, K},
        dtype,
        0,
        group_count);
    weights_md = dnnl::memory::desc(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-120
```cpp
        {group_count, K, N},
        dtype,
        {K * N, 1, K});
    dst_md = dnnl::memory::desc::grouped(
        {M, N},
        dst_dtype,
        0,
        group_count);
  } else{
    TORCH_CHECK(false, "Currently only 2d x 3d grouped matmul with offsets is supported, but got mat_a.dim() = ", M, " and mat_b.dim() = ", N);
  }
  return {src_md, weights_md, dst_md};
}

std::tuple<dnnl::memory, dnnl::memory, dnnl::memory> make_grouped_gemm_mem(
    const dnnl::memory::desc& src_md,
    const dnnl::memory::desc& weights_md,
    const dnnl::memory::desc& dst_md,
    const Tensor& src,
    const Tensor& weights,
    const Tensor& dst,
    const Tensor& offs,
    dnnl::engine& engine,
    bool a_is_2d,
    bool b_is_2d) {
  dnnl::memory src_mem, weights_mem, dst_mem;
  if (a_is_2d && !b_is_2d){
    src_mem = make_onednn_grouped_memory(src_md, engine, src.data_ptr(), offs.data_ptr());
    weights_mem = make_onednn_memory(weights_md, engine, weights.data_ptr());
    dst_mem = make_onednn_grouped_memory(dst_md, engine, dst.data_ptr(), offs.data_ptr());
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-150
```cpp
  } else{
    TORCH_CHECK(false, "Currently only 2d x 3d grouped matmul with offsets is supported, but got mat_a.dim() = ", src.dim(), " and mat_b.dim() = ", weights.dim());
  }
  return {src_mem, weights_mem, dst_mem};
}

sycl::event scaled_grouped_matmul(
    const Tensor& mat_a,
    const Tensor& mat_b,
    const std::optional<Tensor> scale_a,
    const std::optional<Tensor> scale_b,
    const std::optional<at::blas::ScalingType> scaling_choice_a,
    const std::optional<at::blas::ScalingType> scaling_choice_b,
    const Tensor& offs,
    Tensor& out,
    const std::optional<Tensor> alpha) {
  bool a_is_2d = mat_a.dim() == 2;
  bool b_is_2d = mat_b.dim() == 2;

  TORCH_CHECK((a_is_2d && !b_is_2d), "Currently only 2d x 3d grouped matmul with offsets is supported");

  bool is_fp4 = mat_a.scalar_type() == at::kFloat4_e2m1fn_x2;
  int32_t M, N, K, group_count;
  M = mat_a.size(-2);
  K = is_fp4 ? mat_a.size(-1) * 2 : mat_a.size(-1);
  N = mat_b.size(-1);
  group_count = offs.size(0);
  
  // Input tensors should be K-contiguous and address 64bytes aligned for oneDNN.
  // Output tensor is new allocated by caller so that it's already contiguous and aligned.
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
  Tensor contiguous_mat_a = make_contiguous_and_aligned(mat_a);
  Tensor contiguous_mat_b = make_contiguous_and_aligned(mat_b.transpose(-2, -1)).transpose(-2, -1);
  TORCH_CHECK(out.is_contiguous() && is_64_bytes_aligned(out), "Output tensor must be contiguous and 64-byte aligned for oneDNN");
  Tensor contiguous_out = out;

  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();
  
  // 1.1 Create memory descriptor
  dnnl::memory::data_type dtype = get_onednn_dtype(mat_a);
  dnnl::memory::data_type dst_dtype = get_onednn_dtype(out);
  auto [src_md, weights_md, dst_md] = get_grouped_gemm_md(M, N, K, group_count, dtype, dst_dtype, a_is_2d, b_is_2d);

  // 1.2 Create the matmul primitive descriptor
  dnnl::primitive_attr op_attr = dnnl::primitive_attr();

  if (scaling_choice_a.has_value() && scaling_choice_b.has_value()) {
    const ScaleSpec src_spec = make_scale_spec(scaling_choice_a.value(), K, "src", a_is_2d, b_is_2d);
    const ScaleSpec wei_spec = make_scale_spec(scaling_choice_b.value(), K, "wei", a_is_2d, b_is_2d);
    src_spec.apply(op_attr, DNNL_ARG_SRC);
    wei_spec.apply(op_attr, DNNL_ARG_WEIGHTS);
  } 

  dnnl::memory::desc alpha_md;
  if (alpha.has_value()) {
    // set alpha in post-op attr, this is used for NVFP4 case where we need to combine two global fp32 scales into a single alpha value
    alpha_md = get_onednn_md(alpha.value());
    dnnl::post_ops post_ops;
    post_ops.append_binary(
        dnnl::algorithm::binary_mul, alpha_md);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-210
```cpp
    op_attr.set_post_ops(post_ops);
  }

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn())
    op_attr.set_deterministic(true);
#endif

  op_attr.set_scratchpad_mode(dnnl::scratchpad_mode::user);

  // 1.3 Create the matmul primitive descriptor
  dnnl::matmul::primitive_desc matmul_pd = 
      dnnl::matmul::primitive_desc(
            engine, src_md, weights_md, dst_md, op_attr);

  // 2. Prepare memory
  auto [src_mem, weights_mem, dst_mem] = make_grouped_gemm_mem(
      src_md, weights_md, dst_md, contiguous_mat_a, contiguous_mat_b, contiguous_out, offs, engine, a_is_2d, b_is_2d);
  dnnl::memory scratchpad =
      make_onednn_memory(matmul_pd.scratchpad_desc(), engine, nullptr);

  // 3. Setup Args for exec
  std::unordered_map<int, dnnl::memory> args;
  args.insert({DNNL_ARG_SRC, src_mem});
  args.insert({DNNL_ARG_WEIGHTS, weights_mem});
  args.insert({DNNL_ARG_DST, dst_mem});
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad});

  if (scaling_choice_a.has_value() && scaling_choice_b.has_value()) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 211-233
```cpp
    at::Tensor src_scale = scale_a.value();
    at::Tensor wei_scale = scale_b.value();
    dnnl::memory::desc src_scale_md = get_onednn_md(src_scale);
    dnnl::memory::desc wei_scale_md = get_onednn_md(wei_scale);
    auto src_scale_mem = make_onednn_memory(src_scale_md, engine, src_scale.data_ptr());
    auto wei_scale_mem = make_onednn_memory(wei_scale_md, engine, wei_scale.data_ptr());

    args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_SRC, src_scale_mem});
    args.insert({DNNL_ARG_ATTR_SCALES | DNNL_ARG_WEIGHTS, wei_scale_mem});
  }
  if (alpha.has_value()) {
    // set alpha in args for post-op binary
    dnnl::memory alpha_mem = make_onednn_memory(alpha_md, engine, alpha.value().data_ptr());
    args.insert({DNNL_ARG_ATTR_MULTIPLE_POST_OP(0) | DNNL_ARG_SRC_1, alpha_mem});
  }

  dnnl::matmul matmul_p = dnnl::matmul(matmul_pd);
  sycl::event matmul_fwd_event =
      dnnl::sycl_interop::execute(matmul_p, stream, args);
  return matmul_fwd_event;
}

} // namespace at::native::onednn
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: apply, make_scale_spec, get_grouped_gemm_md, make_grouped_gemm_mem, scaled_grouped_matmul.
- CN: 重要符号：apply, make_scale_spec, get_grouped_gemm_md, make_grouped_gemm_mem, scaled_grouped_matmul。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Tensor.h, ATen/core/Tensor.h, c10/core/ScalarType.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h, ATen/native/mkldnn/xpu/detail/Utils.h`.
- CN: 主要内部头文件：`ATen/Tensor.h, ATen/core/Tensor.h, c10/core/ScalarType.h, ATen/native/mkldnn/xpu/detail/Attr.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/detail/oneDNNContext.h, ATen/native/mkldnn/xpu/detail/Utils.h`。
- EN: External/system headers: `oneapi/dnnl/dnnl.hpp`.
- CN: 外部/系统头文件：`oneapi/dnnl/dnnl.hpp`。
- EN: The implementation revolves around symbols such as `apply, make_scale_spec, get_grouped_gemm_md, make_grouped_gemm_mem, scaled_grouped_matmul`.
- CN: 实现围绕 `apply, make_scale_spec, get_grouped_gemm_md, make_grouped_gemm_mem, scaled_grouped_matmul` 等符号展开。
