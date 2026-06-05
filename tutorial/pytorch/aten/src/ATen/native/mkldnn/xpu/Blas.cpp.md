# Blas.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/Blas.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on blas; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 blas；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/WrapDimUtilsMulti.h>
#include <ATen/native/Resize.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/xpu/Blas.h>
#include <torch/library.h>
#ifndef AT_PER_OPERATOR_HEADERS

#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_addmm_activation_native.h>
#include <ATen/ops/addmm_native.h>
#include <ATen/ops/addmv_native.h>
#include <ATen/ops/baddbmm_native.h>
#include <ATen/ops/bmm_native.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/mm_native.h>
#endif

namespace at::native {
namespace xpu {

// result = beta * self + alpha * (mat1 * mat2)
Tensor& addmm_out(
    const Tensor& self,
    const Tensor& mat1,
    const Tensor& mat2,
    const Scalar& beta,
    const Scalar& alpha,
    Tensor& result) {
  checkBackend("addmm_out", {result, self, mat1, mat2}, Backend::XPU);
  TORCH_CHECK(
      mat1.dim() == 2, "mat1 must be a matrix, got ", mat1.dim(), "-D tensor");
  TORCH_CHECK(
      mat2.dim() == 2, "mat2 must be a matrix, got ", mat2.dim(), "-D tensor");
  TORCH_CHECK(
      mat1.sizes()[1] == mat2.sizes()[0],
      "mat1 and mat2 shapes cannot be multiplied (",
      mat1.sizes()[0],
      "x",
      mat1.sizes()[1],
      " and ",
      mat2.sizes()[0],
      "x",
      mat2.sizes()[1],
      ")");
  TORCH_CHECK(
      mat1.dtype() == mat2.dtype(),
      "expected mat1 and mat2 to have the same dtype, but got: ",
      mat1.dtype(),
      " != ",
      mat2.dtype())

  // complex case
  if (self.is_complex()) {
    at::native::addmm_complex_out_xpu(self, mat1, mat2, beta, alpha, result);

    return result;
  }

  std::vector<int64_t> result_shape = {mat1.size(0), mat2.size(1)};
  result.resize_(result_shape);

  IntArrayRef result_sizes = result.sizes();
  if ((result_sizes[0] == 0) || (result_sizes[1] == 0)) {
    return result;
  }

  if (mat1.numel() == 0) {
    if (beta.to<float>() == 0.f) {
      return result.zero_();
    }
    return at::mul_out(
        result,
        self.expand(result.sizes()),
        at::native::scalar_tensor(
            beta, self.scalar_type(), std::nullopt, at::kCPU, std::nullopt));
  }

  TORCH_CHECK(
      are_expandable(self.sizes(), result_shape),
      "addmm_out input must be expanable to:",
      result_shape,
      " but got:",
      self.sizes());

  // Bypass OneDNN optimization path for float64 due to lack of full double
  // precision support.
  if (mat1.scalar_type() == at::kDouble) {
    bool is_inplace = self.is_same(result);
    bool is_beta_ne_zero = beta.to<double>() != 0.0;

    Tensor self_copy;
    if (is_inplace && is_beta_ne_zero) {
      self_copy = self.clone();
    }

    onednn::matmul(result, mat1, mat2, Tensor(), true, onednn::Attr());
```
- EN: Lines 1-100 pull in 14 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are dtype, concentrating a specific part of the operator behavior.
- CN: 第 1-100 行引入了 14 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 dtype，它们承载了某一部分算子行为的核心逻辑。

### Lines 101-200
```cpp
    if (alpha.to<double>() != 1.0) {
      result.mul_(alpha);
    }

    if (is_beta_ne_zero) {
      result.add_(is_inplace ? self_copy : self, beta);
    }

    return result;
  }

  // general case
  Tensor bias = Tensor();
  onednn::Attr attr;
  float beta_ = beta.to<float>();
  float alpha_ = beta_ == 0.f ? alpha.to<float>() : alpha.to<float>() / beta_;
  if (beta_ == 0.f) {
    attr.append_post_eltwise(1.f, alpha_, 0.f, attr.kind_with_linear);
  } else if (alpha_ == 1.f && beta_ == 1.f && !result.is_same(self)) {
    // if result and self are the same tensor, we use post op sum.
    bias = self;
  } else {
    Tensor binary = self.dim() < 1 ? self.unsqueeze(0) : self;
    binary = binary.dim() == 1 ? binary.unsqueeze(0) : binary;
    bool inplace = binary.is_same(result);
    if (inplace) {
      attr.append_post_eltwise(
          1.f, alpha.to<float>(), 0.f, attr.kind_with_linear);
      attr.append_post_sum(beta_);
    } else {
      if (at::native::onednn::is_broadcast(binary)) {
        at::native::onednn::undo_broadcast(binary);
      }
      // in test_addmv_rowmajor_colmajor_incx_incy_lda, binary is a tensor with
      // shape (5, 1) but stride(2, 2)
      binary = at::native::onednn::is_onednn_matmul_strides(binary)
          ? binary
          : binary.contiguous();
      // Tensor binary = self.expand_as(result);
      // For post-binary-add, onednn needs binary scale=1.f
      // Thus we need the following transformation
      // alpha * matmul(mat1, mat2) + beta * binary
      // beta * (alpha/beta * matmul(src, wei) + binary)
      attr.append_post_eltwise(1.f, alpha_, 0.f, attr.kind_with_linear);
      attr.append_post_binary<true>(attr.kind_with_binary_add, binary);
      attr.append_post_eltwise(1.f, beta_, 0.f, attr.kind_with_linear);
    }
  }
  onednn::matmul(result, mat1, mat2, bias, true, attr);
  return result;
}

Tensor& _addmm_activation_out(
    const Tensor& self,
    const Tensor& mat1,
    const Tensor& mat2,
    const Scalar& beta,
    const Scalar& alpha,
    bool use_gelu,
    at::Tensor& result) {
  addmm_out(self, mat1, mat2, beta, alpha, result);
  if (use_gelu) {
    at::gelu_(result);
  } else {
    at::relu_(result);
  }
  return result;
}

Tensor& mm_out(const Tensor& self, const Tensor& mat2, Tensor& result) {
  checkBackend("mm_out", {result, self, mat2}, Backend::XPU);
  TORCH_CHECK(self.dim() == 2, "self must be a matrix");
  TORCH_CHECK(mat2.dim() == 2, "mat2 must be a matrix");
  TORCH_CHECK(
      self.sizes()[1] == mat2.sizes()[0],
      "mat1 and mat2 shapes cannot be multiplied (",
      self.sizes()[0],
      "x",
      self.sizes()[1],
      " and ",
      mat2.sizes()[0],
      "x",
      mat2.sizes()[1],
      ")");
  TORCH_CHECK(
      self.dtype() == mat2.dtype(),
      "expected mat1 and mat2 to have the same dtype, but got: ",
      self.dtype(),
      " != ",
      mat2.dtype())

  result.resize_({self.size(0), mat2.size(1)});
  if (self.numel() == 0 || mat2.numel() == 0) {
    if (result.numel() > 0)
      result.zero_();
    return result;
  }

  if (self.is_complex()) {
    at::native::mm_complex_out_xpu(self, mat2, result);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are shape, is_onednn_matmul_strides, matmul, mm_out, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 shape, is_onednn_matmul_strides, matmul, mm_out，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 201-300
```cpp

    return result;
  }

  onednn::matmul(result, self, mat2, Tensor(), true, onednn::Attr());
  return result;
}

// result = beta * input + alpha * (batch1 @ batch2)
Tensor& baddbmm_out(
    const Tensor& input,
    const Tensor& batch1,
    const Tensor& batch2,
    const Scalar& beta,
    const Scalar& alpha,
    Tensor& result) {
  checkBackend("baddbmm_out", {input, batch1, batch2}, Backend::XPU);
  TORCH_CHECK(batch1.dim() == 3, "expected 3D tensor");
  TORCH_CHECK(batch2.dim() == 3, "expected 3D tensor");

  std::vector<int64_t> result_shape = {
      batch1.size(0), batch1.size(1), batch2.size(2)};
  result.resize_(result_shape);
  if (result.numel() == 0) {
    return result;
  } else if (batch1.size(2) == 0) {
    if (beta.to<c10::complex<double>>() == 0.0) {
      return result.zero_();
    } else {
      at::mul_out(result, input, beta);
      return result;
    }
  }

  TORCH_CHECK(
      are_expandable(input.sizes(), result_shape),
      "baddbmm_out input must be expanable to:",
      result_shape,
      " but got:",
      input.sizes());

  // complex case
  if (input.is_complex()) {
    at::native::baddbmm_complex_out_xpu(
        input, batch1, batch2, beta, alpha, result);

    return result;
  }

  // Bypass OneDNN optimization path for float64 due to lack of full double
  // precision support.
  if (batch1.scalar_type() == at::kDouble ||
      batch2.scalar_type() == at::kDouble) {
    bool is_inplace = input.is_same(result);
    bool is_beta_ne_zero = beta.to<double>() != 0.0;

    Tensor input_copy;
    if (is_inplace && is_beta_ne_zero) {
      input_copy = input.clone();
    }

    onednn::matmul(result, batch1, batch2, Tensor(), true, onednn::Attr());

    if (alpha.to<double>() != 1.0) {
      result.mul_(alpha);
    }

    if (is_beta_ne_zero) {
      result.add_(is_inplace ? input_copy : input, beta);
    }

    return result;
  }

  // general case
  onednn::Attr attr;
  float beta_ = beta.to<float>();
  float alpha_ = beta_ == 0.f ? alpha.to<float>() : alpha.to<float>() / beta_;
  Tensor binary;
  if (beta_ == 0.f) {
    attr.append_post_eltwise(1.f, alpha_, 0.f, attr.kind_with_linear);
  } else {
    Tensor binary = input.dim() < 1 ? input.unsqueeze(0) : input;
    binary = binary.dim() < 3 ? binary.unsqueeze(0) : binary;
    // If input is a 1d tensor need be broadcasted, we need unsqueeze twice.
    binary = binary.dim() < 3 ? binary.unsqueeze_(0) : binary;
    bool inplace = binary.is_same(result);
    if (inplace) {
      attr.append_post_eltwise(
          1.f, alpha.to<float>(), 0.f, attr.kind_with_linear);
      attr.append_post_sum(beta_);
    } else {
      if (at::native::onednn::is_broadcast(binary)) {
        at::native::onednn::undo_broadcast(binary);
      }
      binary = at::native::onednn::is_onednn_matmul_strides(binary)
          ? binary
          : binary.contiguous();
      attr.append_post_eltwise(1.f, alpha_, 0.f, attr.kind_with_linear);
      attr.append_post_binary<true>(attr.kind_with_binary_add, binary);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are scalar_type, is_onednn_matmul_strides, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 scalar_type, is_onednn_matmul_strides，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-400
```cpp
      attr.append_post_eltwise(1.f, beta_, 0.f, attr.kind_with_linear);
    }
  }
  onednn::matmul(result, batch1, batch2, at::Tensor(), true, attr);
  return result;
}

Tensor& bmm_out(const Tensor& self, const Tensor& batch2, Tensor& result) {
  checkBackend("bmm_out", {result, self, batch2}, Backend::XPU);
  TORCH_CHECK(self.dim() == 3, "expected 3D tensor");
  TORCH_CHECK(batch2.dim() == 3, "expected 3D tensor");

  result.resize_({self.size(0), self.size(1), batch2.size(2)});
  if (self.numel() == 0 || batch2.numel() == 0) {
    if (result.numel() > 0)
      result.zero_();
    return result;
  }

  // complex case
  if (self.is_complex()) {
    at::native::bmm_complex_out_xpu(self, batch2, result);

    return result;
  }

  onednn::matmul(result, self, batch2, at::Tensor(), true, onednn::Attr());
  return result;
}

Tensor& addmv_out(
    const Tensor& self,
    const Tensor& mat,
    const Tensor& vec,
    const Scalar& beta,
    const Scalar& alpha,
    Tensor& out) {
  Tensor self_v;
  TORCH_CHECK(
      (mat.dim() == 2 && vec.dim() == 1 && self.dim() <= 1),
      "vector + matrix @ vector expected, got ",
      self.dim(),
      ", ",
      mat.dim(),
      ", ",
      vec.dim());
  if (self.dim() == 1 && self.size(0) != 1) {
    TORCH_CHECK(
        (mat.size(1) == vec.size(0) && mat.size(0) == self.size(0)),
        "size mismatch, get ",
        self.size(0),
        ", ",
        mat.size(0),
        "x",
        mat.size(1),
        ",",
        vec.size(0));
    self_v = self.view({self.size(0), 1});
  } else {
    TORCH_CHECK(
        (mat.size(1) == vec.size(0)),
        "size mismatch, get ",
        mat.size(0),
        "x",
        mat.size(1),
        ",",
        vec.size(0));
    self_v = self;
  }

  Tensor vec_v = vec.view({vec.size(0), 1});

  bool is_float64 =
      mat.scalar_type() == at::kDouble || vec.scalar_type() == at::kDouble;
  bool is_inplace = self.is_same(out);
  if (is_float64 && is_inplace) {
    Tensor self_v_copy = self_v.clone();
    at::native::xpu::addmm_out(self_v_copy, mat, vec_v, beta, alpha, out);
    out.resize_({mat.size(0)});
    return out;
  }

  at::native::xpu::addmm_out(self_v, mat, vec_v, beta, alpha, out);
  out.resize_({mat.size(0)});
  return out;
}

Tensor& tensordot_out(
    const Tensor& input1,
    const Tensor& input2,
    IntArrayRef dims1,
    IntArrayRef dims2,
    Tensor& result) {
  Tensor result_tmp = at::tensordot(input1, input2, dims1, dims2);
  auto result_dtype = result_tmp.scalar_type();
  auto output_tensor_dtype = result.scalar_type();
  auto output_device = result.device();
  auto input1_device = input1.device();
  auto input2_device = input2.device();
  if (result.defined()) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are bmm_out, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 bmm_out，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 401-500
```cpp
    TORCH_CHECK(
        !(result.requires_grad() && at::GradMode::is_enabled() &&
          result.sizes() != result_tmp.sizes()),
        "tensordot(): the 'out' tensor was specified and requires gradients, and its shape does not match the expected result. "
        "Either remove the 'out' argument, ensure it does not require gradients, or make sure its shape matches the expected output.");
  }
  // check if the input & output tensors are on the same device.
  TORCH_CHECK(
      (output_device == input1_device) && (input1_device == input2_device),
      "tensordot: Expected the output and input tensors to be on the "
      "same device, but got the output tensor on ",
      output_device,
      ", input tensor a on ",
      input1_device,
      ", and input tensor b on ",
      input2_device);
  // check if the computed result has the same dtype as the out tensor
  // (because tensordot does not support type promotion)
  TORCH_CHECK(
      result_dtype == output_tensor_dtype,
      "tensordot",
      ": Expected the output tensor to have dtype ",
      result_dtype,
      ", but got an output tensor with dtype ",
      output_tensor_dtype);
  at::native::resize_output(result, result_tmp.sizes());
  result.copy_(result_tmp);
  return result;
}

TORCH_LIBRARY_IMPL(aten, XPU, m) {
  m.impl("tensordot.out", TORCH_FN(tensordot_out));
}
} // namespace xpu

TORCH_IMPL_FUNC(addmm_out_xpu)
(const Tensor& self,
 const Tensor& mat1,
 const Tensor& mat2,
 const Scalar& beta,
 const Scalar& alpha,
 const Tensor& result) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  xpu::addmm_out(self, mat1, mat2, beta, alpha, const_cast<Tensor&>(result));
}

TORCH_IMPL_FUNC(mm_out_xpu)
(const Tensor& self, const Tensor& mat2, const Tensor& result) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  xpu::mm_out(self, mat2, const_cast<Tensor&>(result));
}

TORCH_IMPL_FUNC(bmm_out_xpu)
(const Tensor& self, const Tensor& batch2, const Tensor& result) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  xpu::bmm_out(self, batch2, const_cast<Tensor&>(result));
}

TORCH_IMPL_FUNC(addmm_activation_out_xpu)
(const Tensor& self,
 const Tensor& mat1,
 const Tensor& mat2,
 const Scalar& beta,
 const Scalar& alpha,
 bool use_gelu,
 const Tensor& result) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  xpu::_addmm_activation_out(
      self, mat1, mat2, beta, alpha, use_gelu, const_cast<Tensor&>(result));
}

TORCH_IMPL_FUNC(baddbmm_out_xpu)
(const Tensor& self,
 const Tensor& batch1,
 const Tensor& batch2,
 const Scalar& beta,
 const Scalar& alpha,
 const Tensor& result) {
  xpu::baddbmm_out(
      self,
      batch1,
      batch2,
      beta,
      alpha,
      // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
      const_cast<Tensor&>(result));
}

TORCH_IMPL_FUNC(addmv_out_xpu)
(const Tensor& self,
 const Tensor& mat,
 const Tensor& vec,
 const Scalar& beta,
 const Scalar& alpha,
 const Tensor& result) {
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
  xpu::addmv_out(self, mat, vec, beta, alpha, const_cast<Tensor&>(result));
}

Tensor _weight_int4pack_mm_xpu(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_LIBRARY_IMPL, TORCH_IMPL_FUNC, NOLINTNEXTLINE, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_LIBRARY_IMPL, TORCH_IMPL_FUNC, NOLINTNEXTLINE，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

### Lines 501-600
```cpp
    const Tensor& A,
    const Tensor& B,
    int64_t qGroupSize,
    const Tensor& qScale,
    const Tensor& qZeros) {
  auto M = A.size(0); // M
  auto N = B.size(0); // N1=LCM(N, K)
  TORCH_CHECK(
      A.dtype() == kBFloat16 || A.dtype() == kHalf || A.dtype() == kFloat,
      __func__,
      " : expect A to be either 32-bit or 16-bit float tensor.");
  TORCH_CHECK(A.is_contiguous(), __func__, " : expect A to be contiguous.");
  TORCH_CHECK(A.dim() == 2, __func__, " : expect A to be 2D tensor.");

  TORCH_CHECK(B.dtype() == kInt, __func__, " : expect B to be int32 tensor.");
  TORCH_CHECK(
      qZeros.dtype() == kChar,
      __func__,
      " : expect qZeros to be int8 tensor currently.");
  TORCH_CHECK(B.dim() == 2, __func__, " : expect B to 2d tensor.");

  TORCH_CHECK(
      qGroupSize > 1 && qGroupSize % 32 == 0,
      __func__,
      " : expect qGroupSize to be multiple of 32 and greater than 1, got ",
      qGroupSize);

  TORCH_CHECK(
      qScale.dim() == 2 && qScale.size(1) == N,
      __func__,
      ": expect qScale to be 2d tensor with sizes [:, ",
      N,
      "]");
  TORCH_CHECK(
      qZeros.dim() == 2 && qZeros.size(1) == N,
      __func__,
      ": expect qZeros to be 2d tensor with sizes [:, ",
      N,
      "]");

  auto C = at::empty({M, N}, A.options());

  // qscale:[K/qGroupSize, N]
  // qzp:[K/qGroupSize, N]
  at::native::onednn::woq_matmul_int4(C, A, B, qScale, qZeros, qGroupSize);

  return C;
}

Tensor& _int_mm_out_xpu(
    const Tensor& self,
    const Tensor& mat2,
    Tensor& result) {
  TORCH_CHECK(
      self.dim() == 2,
      "Expected self to be of dimension 2 but got ",
      self.dim());
  TORCH_CHECK(
      mat2.dim() == 2,
      "Expected mat2 to be of dimension 2 but got ",
      mat2.dim());
  TORCH_CHECK(
      self.size(1) == mat2.size(0),
      "self.size(1) needs to match mat2.size(0) but got ",
      self.size(1),
      " and ",
      mat2.size(0));

  TORCH_CHECK(
      self.dtype() == at::kChar,
      "Expected self dtype to be of type int8 but got ",
      self.dtype());
  TORCH_CHECK(
      mat2.dtype() == at::kChar,
      "Expected mat2 dtype to be of type int8 but got ",
      mat2.dtype());
  TORCH_CHECK(
      result.dtype() == at::kInt,
      "Expected result dtype to be of type kInt but got ",
      result.dtype());
  TORCH_CHECK(
      result.size(0) == self.size(0),
      "Expected result.size(0) to be ",
      self.size(0),
      " but got ",
      result.size(0));
  TORCH_CHECK(
      result.size(1) == mat2.size(1),
      "Expected result.size(1) to be ",
      mat2.size(1),
      " but got ",
      result.size(1));

  TORCH_CHECK(
      result.dim() == 2,
      "Expected result to be of dimension 2 but got ",
      result.dim());

  TORCH_CHECK(result.is_contiguous(), "Expected result to be contiguous.");
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are LCM, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 LCM，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 601-700
```cpp
  if (result.numel() == 0 || self.size(1) == 0) {
    return result.zero_();
  }

  Tensor bias = at::Tensor();
  Tensor mat2_scales = at::ones({1}, mat2.options().dtype(at::kFloat));
  Tensor mat2_zero_points = at::Tensor();
  auto post_op_args = torch::List<std::optional<at::Scalar>>();

  at::native::onednn::quantized_matmul(
      self.contiguous(),
      1.0,
      0,
      mat2,
      mat2_scales,
      mat2_zero_points,
      bias,
      result,
      1.0,
      0,
      result.scalar_type(),
      /*other*/ std::nullopt,
      /*other scale*/ 1.0,
      /*other zp*/ 0,
      /*binary post op*/ "none",
      /*binary alpha*/ 1.0,
      /*post_op_name*/ "none",
      post_op_args,
      /*post_op_algorithm*/ "none",
      /*m2_trans*/ true);
  return result;
}

Tensor _int_mm_xpu(const Tensor& self, const Tensor& mat2) {
  Tensor result =
      at::empty({self.size(0), mat2.size(1)}, self.options().dtype(at::kInt));
  return _int_mm_out_xpu(self, mat2, result);
}

Tensor _weight_int8pack_mm_xpu(
    const Tensor& A,
    const Tensor& B,
    const Tensor& scales) {
  auto M = A.size(0);
  auto N = B.size(0);
  auto K = A.size(1);

  TORCH_CHECK(
      A.dtype() == kBFloat16 || A.dtype() == kHalf || A.dtype() == kFloat,
      " : expect A to be either 32-bit or 16-bit float tensor.");
  TORCH_CHECK(A.dim() == 2, __func__, " : expect A to be 2D tensor.");
  TORCH_CHECK(
      A.stride(1) == 1, " : A must be contiguous on the last dimension.");
  TORCH_CHECK(B.dtype() == kChar, " : expect B to be int8 tensor.");
  TORCH_CHECK(B.is_contiguous(), " : expect B to be contiguous.");
  TORCH_CHECK(B.size(1) == K, " : expect B.size(1) == ", K);

  TORCH_CHECK(
      scales.dim() == 1 && scales.size(0) == N,
      " : expect scales to be 1d tensor with size ",
      N);

  auto C = at::empty({M, N}, A.options());

  // --- Launch kernel ---
  Tensor bias = at::Tensor();
  Tensor mat2_zero_points = at::Tensor();
  Tensor non_const_scales = scales;
  auto post_op_args = torch::List<std::optional<at::Scalar>>();

  at::native::onednn::quantized_matmul(
      A.contiguous(),
      1.0,
      0,
      B,
      non_const_scales,
      mat2_zero_points,
      bias,
      C,
      1.0,
      0,
      C.scalar_type(),
      /*other*/ std::nullopt,
      /*other scale*/ 1.0,
      /*other zp*/ 0,
      /*binary post op*/ "none",
      /*binary alpha*/ 1.0,
      /*post_op_name*/ "none",
      post_op_args,
      /*post_op_algorithm*/ "none",
      /*m2_trans*/ false);

  return C;
}

Tensor _bmm_dtype_xpu(
    const Tensor& batch1,
    const Tensor& batch2,
    const at::ScalarType out_dtype) {
  Tensor out = at::empty(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _int_mm_xpu, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _int_mm_xpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 701-800
```cpp
      {batch1.size(0), batch1.size(1), batch2.size(2)},
      batch1.options().dtype(out_dtype));
  return _bmm_out_dtype_xpu(batch1, batch2, out_dtype, out);
}

static void out_dtype_checks(
    const Tensor& mat1,
    const at::ScalarType out_dtype,
    const Tensor& out) {
  TORCH_CHECK(
      out_dtype == out.scalar_type(),
      "out_dtype must be the same as the dtype of the provided out tensor");
  TORCH_CHECK(
      out_dtype == mat1.scalar_type() ||
          (out_dtype == at::ScalarType::Float &&
           (mat1.scalar_type() == at::ScalarType::Half ||
            mat1.scalar_type() == at::ScalarType::BFloat16)),
      "out_dtype must be the same as input dtype or fp32 for fp16/bf16 inputs");
}

Tensor& _bmm_out_dtype_xpu(
    const Tensor& batch1,
    const Tensor& batch2,
    const at::ScalarType out_dtype,
    Tensor& out) {
  out_dtype_checks(batch1, out_dtype, out);
  xpu::bmm_out(batch1, batch2, const_cast<Tensor&>(out));
  return out;
}

Tensor _baddbmm_dtype_xpu(
    const Tensor& self,
    const Tensor& batch1,
    const Tensor& batch2,
    const at::ScalarType out_dtype,
    const Scalar& beta,
    const Scalar& alpha) {
  TORCH_CHECK(
      self.scalar_type() == out_dtype || self.scalar_type() == batch1.dtype(),
      "self dtype must match either out_dtype or batch1 dtype");
  Tensor out = at::empty(
      {batch1.size(0), batch1.size(1), batch2.size(2)},
      batch1.options().dtype(out_dtype));
  return _baddbmm_out_dtype_xpu(
      self, batch1, batch2, out_dtype, beta, alpha, out);
}

Tensor& _baddbmm_out_dtype_xpu(
    const Tensor& self,
    const Tensor& batch1,
    const Tensor& batch2,
    const at::ScalarType out_dtype,
    const Scalar& beta,
    const Scalar& alpha,
    Tensor& out) {
  out_dtype_checks(batch1, out_dtype, out);
  xpu::baddbmm_out(
      self,
      batch1,
      batch2,
      beta,
      alpha,
      // NOLINTNEXTLINE(cppcoreguidelines-pro-type-const-cast)
      const_cast<Tensor&>(out));
  return out;
}

Tensor _mm_dtype_xpu(
    const Tensor& self,
    const Tensor& mat2,
    const at::ScalarType out_dtype) {
  Tensor result =
      at::empty({self.size(0), mat2.size(1)}, self.options().dtype(out_dtype));
  return _mm_dtype_out_xpu(self, mat2, out_dtype, result);
}

Tensor& _mm_dtype_out_xpu(
    const Tensor& self,
    const Tensor& mat2,
    const at::ScalarType out_dtype,
    Tensor& out) {
  out_dtype_checks(self, out_dtype, out);
  xpu::mm_out(self, mat2, const_cast<Tensor&>(out));
  return out;
}

Tensor _addmm_dtype_xpu(
    const Tensor& self,
    const Tensor& mat1,
    const Tensor& mat2,
    const at::ScalarType out_dtype,
    const Scalar& beta,
    const Scalar& alpha) {
  TORCH_CHECK(
      mat1.dim() == 2, "mat1 must be a matrix, got ", mat1.dim(), "-D tensor");
  TORCH_CHECK(
      mat2.dim() == 2, "mat2 must be a matrix, got ", mat2.dim(), "-D tensor");
  Tensor result =
      at::empty({mat1.size(0), mat2.size(1)}, self.options().dtype(out_dtype));
  return _addmm_dtype_out_xpu(self, mat1, mat2, out_dtype, beta, alpha, result);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NOLINTNEXTLINE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NOLINTNEXTLINE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 801-816
```cpp
}

Tensor& _addmm_dtype_out_xpu(
    const Tensor& self,
    const Tensor& mat1,
    const Tensor& mat2,
    const at::ScalarType out_dtype,
    const Scalar& beta,
    const Scalar& alpha,
    Tensor& out) {
  out_dtype_checks(mat1, out_dtype, out);
  xpu::addmm_out(self, mat1, mat2, beta, alpha, out);
  return out;
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: addmm_out, TORCH_CHECK, _addmm_activation_out, mm_out, baddbmm_out, bmm_out, addmv_out, tensordot_out.
- CN: 重要符号：addmm_out, TORCH_CHECK, _addmm_activation_out, mm_out, baddbmm_out, bmm_out, addmv_out, tensordot_out。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/WrapDimUtilsMulti.h, ATen/native/Resize.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/xpu/Blas.h, torch/library.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_addmm_activation_native.h, ATen/ops/addmm_native.h, ATen/ops/addmv_native.h, ATen/ops/baddbmm_native.h, ATen/ops/bmm_native.h`.
- CN: 主要内部头文件：`ATen/WrapDimUtilsMulti.h, ATen/native/Resize.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/xpu/Blas.h, torch/library.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_addmm_activation_native.h, ATen/ops/addmm_native.h, ATen/ops/addmv_native.h, ATen/ops/baddbmm_native.h, ATen/ops/bmm_native.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `addmm_out, TORCH_CHECK, _addmm_activation_out, mm_out, baddbmm_out, bmm_out, addmv_out, tensordot_out, TORCH_LIBRARY_IMPL, TORCH_IMPL_FUNC`.
- CN: 实现围绕 `addmm_out, TORCH_CHECK, _addmm_activation_out, mm_out, baddbmm_out, bmm_out, addmv_out, tensordot_out, TORCH_LIBRARY_IMPL, TORCH_IMPL_FUNC` 等符号展开。
