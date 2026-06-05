# Matmul.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/detail/Matmul.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on matmul; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 matmul；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp

#include <c10/xpu/XPUFunctions.h>

#include <ATen/ATen.h>
#include <ATen/record_function.h>

#include <Attr.h>
#include <Utils.h>

#include <c10/core/ScalarType.h>
#include <oneapi/dnnl/dnnl.hpp>

namespace at::native::onednn {

sycl::event matmul(
    at::Tensor& result,
    const at::Tensor& mat1,
    const at::Tensor& mat2,
    const at::Tensor& b_raw,
    bool m2_trans,
    Attr attr,
    const std::vector<sycl::event>& deps) {
  // m2_trans means mat2 is transposed from the nn.Linear perspective.
  // m2_trans==true means mat2 is [k, n] layout.
  // m2_trans==false means mat2 is [n, k] layout, aka, the default layout in
  // nn.Linear.
  int64_t dims = result.dim();
  TORCH_CHECK(
      dims == 2 || dims == 3,
      "oneDNN matmul only works with 2D or 3D, got ",
```
- EN: Lines 1-30 pull in 7 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-30 行引入了 7 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 31-60
```cpp
      dims);
  TORCH_CHECK(
      dims == mat1.dim() && dims == mat2.dim(),
      "oneDNN input matrixes must have the same ranks");
  TORCH_CHECK(result.defined(), "oneDNN matmul result should be defined");

  auto& engine = GpuEngineManager::Instance().get_engine();
  auto& stream = GpuStreamManager::Instance().get_stream();

  at::Tensor m1 = mat1;
  at::Tensor m2 = mat2;

  undo_broadcast_on_batch(m1, m2);

  m1 = is_onednn_matmul_strides(m1) ? m1 : m1.contiguous();
  m2 = is_onednn_matmul_strides(m2) ? m2 : m2.contiguous();
  at::Tensor dst =
      is_onednn_matmul_strides(result) ? result : result.contiguous();

  int64_t m = dst.size(-2);
  int64_t n = dst.size(-1);
  int64_t k = m1.size(-1);
  int64_t mb = 1;

  if (dims == 3) {
    mb = dst.size(0);
    TORCH_CHECK(
        mb == mat1.size(0) && mb == mat2.size(0),
        "batch size mismatch, dst mb: ",
        mb,
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
        "m1 mb",
        mat1.size(0),
        " m2 mb: ",
        mat2.size(0));
  }

  // validate bias and make it compatible with oneDNN implementation
  bool with_bias = false;
  at::Tensor b = b_raw;
  if (b.defined()) {
    with_bias = true;
    if (b.dim() == 1) {
      TORCH_CHECK(
          b.size(0) == n || b.size(0) == 1,
          "matmul supports [n] or [1] when bias dim is 1 ...");
      if (b.size(0) == 0) {
        with_bias = false;
      } else if (m1.dim() == 3) {
        b = b.expand({mb, m, n}).contiguous();
      } else if (m1.dim() == 2) {
        b = b.expand({1, n}).contiguous();
      }
    } else if (b.dim() == 2) {
      TORCH_CHECK(
          (b.size(0) == m && b.size(1) == n) ||
              (b.size(0) == 1 && b.size(1) == n) ||
              (b.size(0) == m && b.size(1) == 1) ||
              (b.size(0) == 1 && b.size(1) == 1),
          "matmul supports [m, n] or [1, n] or [m, 1] or [1, 1] when bias dim is 2 ...");
      if (b.size(0) == 1 && b.size(1) == 1)
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-120
```cpp
        b = b.expand({1, n}).contiguous();
    } else if (b.dim() == 3) {
      TORCH_CHECK(
          at::are_expandable({mb, m, n}, b.sizes()),
          "matmul bias must be expandable to:",
          dst.sizes(),
          " but got:",
          b.sizes());
      b = b.expand({mb, m, n}).contiguous();
    } else if (b.dim() == 0) {
      TORCH_CHECK(
          b.numel() == 1, "matmul supports 1 numel when bias dim is [] ...");
      if (m1.dim() == 3) {
        b = b.expand({mb, m, n}).contiguous();
      } else {
        b = b.expand({1, n}).contiguous();
      }
    } else {
      TORCH_CHECK(0, "unsupported bias dim in matmul ...");
    }
  }

  b = b.contiguous(); // avoid reorder 2 times

  // xpu matmul support both ab/ba shape for m2 tensor, we don't check any more
  auto m1_usr_dt = get_onednn_dtype_include_double(m1);
  auto m2_usr_dt = get_onednn_dtype_include_double(m2);
  auto dst_usr_dt = get_onednn_dtype_include_double(dst);

  auto m1_dt = m1_usr_dt;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-150
```cpp
  auto m2_dt = m2_usr_dt;
  auto dst_dt = dst_usr_dt;
  dnnl::memory::data_type bias_dt;

  dnnl::memory::desc m1_md, m1_usr_md, m1_any_md;
  dnnl::memory::desc m2_md, m2_usr_md, m2_any_md;
  dnnl::memory::desc dst_md, dst_usr_md, dst_any_md;
  dnnl::memory::desc bias_md;

  // Naive Master weight
  if (m1_dt == dnnl::memory::data_type::bf16 &&
      m2_dt == dnnl::memory::data_type::f32) {
    m2_dt = dnnl::memory::data_type::bf16;
    dst_dt = dnnl::memory::data_type::bf16;
  } else if (
      m1_dt == dnnl::memory::data_type::f32 &&
      m2_dt == dnnl::memory::data_type::bf16) {
    m1_dt = dnnl::memory::data_type::bf16;
    dst_dt = dnnl::memory::data_type::bf16;
  }

  dnnl::memory::dims m1_dims, m2_dims, dst_dims, bias_dims;
  dnnl::memory::dims m1_strides, m2_strides, dst_strides, bias_strides;
  if (dims == 2) {
    m1_dims = {m, k};
    m2_dims = {k, n};
    dst_dims = {m, n};

    m1_strides = {m1.stride(0), m1.stride(1)};
    if (m2_trans) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 151-180
```cpp
      m2_strides = {m2.stride(0), m2.stride(1)};
    } else {
      m2_strides = {m2.stride(1), m2.stride(0)};
    }
    dst_strides = {dst.stride(0), dst.stride(1)};
  } else {
    m1_dims = {m1.size(0), m, k};
    m2_dims = {m2.size(0), k, n};
    dst_dims = {mb, m, n};

    m1_strides = {m1.stride(0), m1.stride(1), m1.stride(2)};
    if (m2_trans) {
      m2_strides = {m2.stride(0), m2.stride(1), m2.stride(2)};
    } else {
      m2_strides = {m2.stride(0), m2.stride(2), m2.stride(1)};
    }
    dst_strides = {dst.stride(0), dst.stride(1), dst.stride(2)};
  }

  if (with_bias) {
    bias_dims = get_onednn_dims(b);
    bias_dt = get_onednn_dtype_include_double(b);
    bias_strides = get_onednn_strides(b);
  }

  dnnl::post_ops po = attr.extract_post_ops(dst);

  std::unordered_map<int, dnnl::memory> args;
  dnnl::matmul matmul_p;
  dnnl::matmul::primitive_desc matmul_pd;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-210
```cpp

  // STEP1: create memory desc
  m1_md = dnnl::memory::desc(m1_dims, m1_dt, m1_strides);
  m2_md = dnnl::memory::desc(m2_dims, m2_dt, m2_strides);
  dst_md = dnnl::memory::desc(dst_dims, dst_dt, dst_strides);

  // STEP2: creat attribute
  dnnl::primitive_attr pattr;
  pattr.set_post_ops(po);

#if ONEDNN_SUPPORT_DETERMINISTIC
  if (at::globalContext().deterministicAlgorithms() ||
      at::globalContext().deterministicMkldnn())
    pattr.set_deterministic(true);
#endif

  // scratchpad
  pattr.set_scratchpad_mode(dnnl::scratchpad_mode::user);

  if (m1_dt == dnnl::memory::data_type::f32) {
    bool allow_tf32 = at::globalContext().allowTF32OneDNN();
    if (allow_tf32) {
      pattr.set_fpmath_mode(dnnl::fpmath_mode::tf32);
    } else {
      pattr.set_fpmath_mode(dnnl::fpmath_mode::strict);
    }
  }

  // STEP3: create primitive
  if (with_bias) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are globalContext, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 globalContext，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 211-240
```cpp
    bias_md = dnnl::memory::desc(bias_dims, bias_dt, bias_strides);
    matmul_pd = dnnl::matmul::primitive_desc(
        engine, m1_md, m2_md, bias_md, dst_md, pattr);
  } else {
    matmul_pd =
        dnnl::matmul::primitive_desc(engine, m1_md, m2_md, dst_md, pattr);
  }

  matmul_p = dnnl::matmul(matmul_pd);

  m1_usr_md = dnnl::memory::desc(m1_dims, m1_usr_dt, m1_strides);
  m2_usr_md = dnnl::memory::desc(m2_dims, m2_usr_dt, m2_strides);
  dst_usr_md = dnnl::memory::desc(dst_dims, dst_usr_dt, dst_strides);

  // STEP4: create memory
  auto m1_usr_m = make_onednn_memory(m1_usr_md, engine, m1.data_ptr());
  auto m2_usr_m = make_onednn_memory(m2_usr_md, engine, m2.data_ptr());
  auto dst_usr_m = make_onednn_memory(dst_usr_md, engine, dst.data_ptr());

  auto expected_m1_md = matmul_pd.src_desc();
  auto expected_m2_md = matmul_pd.weights_desc();
  auto expected_dst_md = matmul_pd.dst_desc();

  dnnl::memory m1_m = m1_usr_m, m2_m = m2_usr_m, dst_m = dst_usr_m;
  at::Tensor m1_, m2_, dst_;

  if (attr.with_binary())
    attr.construct_post_binary(matmul_pd, args);

  size_t scratchpad_size = matmul_pd.scratchpad_desc().get_size();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-266
```cpp
  at::Tensor scratchpad_tensor = at::empty(
      {static_cast<int64_t>(scratchpad_size)},
      m1.options().dtype(at::kByte),
      std::nullopt);
  auto scratchpad_memory = make_onednn_memory(
      matmul_pd.scratchpad_desc(), engine, scratchpad_tensor.data_ptr());
  args.insert({DNNL_ARG_SCRATCHPAD, scratchpad_memory});

  args.insert({DNNL_ARG_SRC, m1_m});
  args.insert({DNNL_ARG_WEIGHTS, m2_m});
  args.insert({DNNL_ARG_DST, dst_m});
  if (with_bias) {
    auto bias_m = make_onednn_memory(bias_md, engine, b.data_ptr());
    args.insert({DNNL_ARG_BIAS, bias_m});
  }

  sycl::event matmul_event =
      dnnl::sycl_interop::execute(matmul_p, stream, args, deps);

  if (!dst.is_same(result))
    result.copy_(dst);

  return matmul_event;
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
- EN: Notable symbols: matmul.
- CN: 重要符号：matmul。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/xpu/XPUFunctions.h, ATen/ATen.h, ATen/record_function.h, c10/core/ScalarType.h`.
- CN: 主要内部头文件：`c10/xpu/XPUFunctions.h, ATen/ATen.h, ATen/record_function.h, c10/core/ScalarType.h`。
- EN: External/system headers: `Attr.h, Utils.h, oneapi/dnnl/dnnl.hpp`.
- CN: 外部/系统头文件：`Attr.h, Utils.h, oneapi/dnnl/dnnl.hpp`。
- EN: The implementation revolves around symbols such as `matmul`.
- CN: 实现围绕 `matmul` 等符号展开。
