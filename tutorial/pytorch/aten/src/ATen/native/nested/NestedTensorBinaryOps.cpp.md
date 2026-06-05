# NestedTensorBinaryOps.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/NestedTensorBinaryOps.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor binary ops; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor binary ops；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/native/nested/NestedTensorMath.h>
#include  <ATen/native/nested/NestedTensorBinaryOps.h>

#include <ATen/AccumulateType.h>
#include <ATen/Dispatch.h>
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#include <ATen/NestedTensorImpl.h>
#include <ATen/ScalarOps.h>
#include <ATen/TensorIndexing.h>
#include <ATen/TensorOperators.h>
#include <ATen/TensorUtils.h>
#include <ATen/core/Tensor.h>
#include <ATen/native/layer_norm.h>
#include <ATen/native/nested/NestedTensorUtils.h>

namespace at::native {

DEFINE_DISPATCH(nested_dense_elementwise_stub);
REGISTER_NO_CPU_DISPATCH(nested_dense_elementwise_stub)

std::pair<NestedTensorImpl*, NestedTensorImpl*>
static get_elementwise_nested_tensor_impl(
    const Tensor& self,
    const Tensor& other,
    const std::string& op_name) {
  if (self.is_nested() && !(other.is_nested())) {
    TORCH_CHECK(
        false,
        "Expected both self and other to be nested, but got a nested self and non-nested other");
  } else if (!(self.is_nested()) && other.is_nested()) {
    TORCH_CHECK(
        false,
        "Expected both self and other to be nested, but got a non-nested self and nested other");
  } else if (!(self.is_nested()) || !(other.is_nested())) {
    TORCH_CHECK(
        false,
        "Expected both self and other to be nested, but got a non-nested self and non-nested other");
  }

  auto self_ptr = get_nested_tensor_impl(self);
  auto other_ptr = get_nested_tensor_impl(other);

  TORCH_CHECK(
      self.dim() == other.dim(),
      op_name,
      " does not support broadcasting when given a NestedTensor");
  TORCH_CHECK(
      at::equal(
          self_ptr->get_nested_sizes(),
          other_ptr->get_nested_sizes()),
      op_name,
      " does not support broadcasting when given a NestedTensor");
  TORCH_CHECK(
      at::equal(
          self_ptr->get_nested_strides(),
          other_ptr->get_nested_strides()),
      op_name,
      " requires strides to match when given NestedTensors");
  const auto self_offsets = self_ptr->get_storage_offsets();
```
- EN: Lines 1-60 pull in 14 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are REGISTER_NO_CPU_DISPATCH, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 14 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 REGISTER_NO_CPU_DISPATCH，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
  const int64_t *self_offsets_ptr = self_offsets.const_data_ptr<int64_t>();
  const int64_t *other_offsets_ptr = other_ptr->get_storage_offsets().const_data_ptr<int64_t>();
  bool offsets_match = true;
  for (auto i = 0; i < self_offsets.size(0); i++) {
    offsets_match = offsets_match && (self_offsets_ptr[i] == other_offsets_ptr[i]);
  }
  TORCH_CHECK(
      offsets_match,
      op_name,
      " requires offsets to match when given NestedTensors");
  return std::make_pair(self_ptr, other_ptr);
}

template <typename Func>
static Tensor NestedTensor_elementwise_Tensor(
    const Tensor& self,
    const Tensor& other,
    const std::string& op_name,
    bool supports_striding,
    Func f) {
  Tensor self_contiguous = self;
  Tensor other_contiguous = other;
  // self is a scalar
  if (!self.is_nested() && self.dim() == 0 && self.numel() == 1) {
    auto other_impl = get_nested_tensor_impl(other);
    return wrap_buffer(
      f(self, other_impl->get_unsafe_storage_as_tensor()),
      other_impl->get_nested_sizes().clone(),
      other_impl->get_nested_strides().clone(),
      other_impl->get_storage_offsets()
    );
  }
  // other is a scalar
  if (!other.is_nested() && other.dim() == 0 && other.numel() == 1) {
    auto self_impl = get_nested_tensor_impl(self);
    return wrap_buffer(
      f(self_impl->get_unsafe_storage_as_tensor(), other),
      self_impl->get_nested_sizes().clone(),
      self_impl->get_nested_strides().clone(),
      self_impl->get_storage_offsets()
    );
  }
  // special case when other is dense (CUDA only for now)
  if (self.is_nested() && !other.is_nested() && self.is_cuda() && other.is_cuda()) {
    auto self_ptr = get_nested_tensor_impl(self);
    auto other_ = other;
    // check for the [B, *, D], [B, 1, D] case -> use custom kernel
    // TODO: this if statement is ugly and hopefully we will remove this in the near future
    bool is_broadcastable_3d = (
        self_ptr->dim() == 3 &&
        other.dim() == 3 &&
        self_ptr->size(0) == other.size(0) &&
        other.size(1) == 1 &&
        self_ptr->opt_size(2).has_value() &&
        self_ptr->opt_size(2) == other.size(2));
    // check for the [B, *], [B, 1] case -> treat as 3D with [B, *, 1], [B, 1, 1]
    bool is_broadcastable_2d = (
        self_ptr->dim() == 2 &&
        other.dim() == 2 &&
        self_ptr->size(0) == other.size(0) &&
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_storage_offsets, dense, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_storage_offsets, dense，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
        other.size(1) == 1);
    if(is_broadcastable_2d) {
        other_ = other.unsqueeze(-1);
        is_broadcastable_3d = true;
    }

    if (is_broadcastable_3d) {
      self_contiguous = self.contiguous();
      self_ptr = get_nested_tensor_impl(self_contiguous);
      const auto self_buffer = self_ptr->get_buffer();
      const auto self_sizes = self_ptr->get_nested_sizes();
      auto result_buffer = at::empty_like(self_buffer);
      auto result = wrap_buffer(result_buffer, self_sizes);
      if (op_name == "add") {
        nested_dense_elementwise_stub(self.device().type(), result, self, other_, NESTED_DENSE_OP::ADD);
      } else if (op_name == "mul") {
        nested_dense_elementwise_stub(self.device().type(), result, self, other_, NESTED_DENSE_OP::MUL);
      } else {
        TORCH_CHECK(false, "Unsupported nested dense elementwise op: ", op_name, ".");
      }
      return result;
    }

    // check for the [B, C, *, *], [C, 1, 1] case
    bool is_broadcastable_4d_3d = (
        self_ptr->dim() == 4 &&
        other.dim() == 3 &&
        self_ptr->opt_size(1).has_value() &&
        self_ptr->size(1) == other.size(0) &&
        other.size(1) == 1 &&
        other.size(2) == 1);
    if (is_broadcastable_4d_3d) {
      std::vector<Tensor> results;
      for (const auto& t : self.unbind()) {
        results.push_back(f(t, other));
      }
      return at::_nested_tensor_from_tensor_list(results);
    }

    TORCH_CHECK(
        false,
        "Expected both self and other to be nested, but got a nested self and non-nested other for op: ",
        op_name,
        ".");
  }

  self_contiguous = supports_striding ? self.contiguous() : self;
  other_contiguous = supports_striding ? other.contiguous() : other;

  auto [self_impl, other_impl] =
      get_elementwise_nested_tensor_impl(self_contiguous, other_contiguous, op_name);
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(self_impl);
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(other_impl);
  return wrap_buffer(
      f(self_impl->get_unsafe_storage_as_tensor(),
        other_impl->get_unsafe_storage_as_tensor()),
      self_impl->get_nested_sizes(),
      self_impl->get_nested_strides(),
      self_impl->get_storage_offsets());
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-240
```cpp

Tensor NestedTensor_add_Tensor(
    const Tensor& self,
    const Tensor& other,
    const Scalar& alpha) {
  return NestedTensor_elementwise_Tensor(
      self, other, "add", true /* supports_striding*/, [alpha](const Tensor& b1, const Tensor& b2) {
        return at::add(b1, b2, alpha);
      });
}

Tensor NestedTensor_sub_Tensor(
    const Tensor& self,
    const Tensor& other,
    const Scalar& alpha) {
  return NestedTensor_elementwise_Tensor(
      self, other, "sub", true /* supports_striding*/, [alpha](const Tensor& b1, const Tensor& b2) {
        return at::sub(b1, b2, alpha);
      });
}

Tensor NestedTensor_mul_Tensor(const Tensor& self, const Tensor& other) {
  return NestedTensor_elementwise_Tensor(
      self, other, "mul", false /* supports_striding*/, [](const Tensor& b1, const Tensor& b2) {
        return at::mul(b1, b2);
      });
}

// Only usable on the C++ side; scalars are converted to tensors coming from Python.
Tensor NestedTensor_mul_Scalar(const Tensor& self, const Scalar& other) {
  return NestedTensor_mul_Tensor(self, wrapped_scalar_tensor(other));
}

Tensor NestedTensor_div_Tensor(const Tensor& self, const Tensor& other) {
  return NestedTensor_elementwise_Tensor(
      self, other, "div", false /* supports_striding*/, [](const Tensor& b1, const Tensor& b2) {
        return at::div(b1, b2);
      });
}

// Only usable on the C++ side; scalars are converted to tensors coming from Python.
Tensor NestedTensor_div_Scalar(const Tensor& self, const Scalar& other) {
  return NestedTensor_div_Tensor(self, wrapped_scalar_tensor(other));
}
Tensor NestedTensor_masked_fill(
    const Tensor& self,
    const Tensor& mask,
    const Scalar& value) {
  return NestedTensor_elementwise_Tensor(
      self, mask, "masked_fill", false /* supports_striding*/, [value](const Tensor& b1, const Tensor& b2) {
        return at::masked_fill(b1, b2, value);
      });
}


template <typename Func>
static Tensor& NestedTensor_elementwise__Tensor(
    Tensor& self,
    const Tensor& other,
    const std::string& op_name,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor, NestedTensor_div_Scalar, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor, NestedTensor_div_Scalar，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-300
```cpp
    Func f) {
  // self is a scalar
  if (!self.is_nested() && self.dim() == 0 && self.numel() == 1) {
    auto other_impl = get_nested_tensor_impl(other);
    f(self, other_impl->get_buffer());
    return self;
  }
  // other is a scalar
  if (!other.is_nested() && other.dim() == 0 && other.numel() == 1) {
    auto self_impl = get_nested_tensor_impl(self);
    f(self_impl->get_buffer(), other);
    return self;
  }
  auto [self_impl, other_impl] =
      get_elementwise_nested_tensor_impl(self, other, op_name);
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(self_impl);
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(other_impl);
  const auto& nt_self = *self_impl;
  const auto& nt_other = *other_impl;
  f(nt_self.get_buffer().view({-1}), nt_other.get_buffer().view({-1}));
  return self;
}

Tensor& NestedTensor_add__Tensor(
    Tensor& self,
    const Tensor& other,
    const Scalar& alpha) {
  return NestedTensor_elementwise__Tensor(
      self, other, "add_", [alpha](const Tensor& b1, const Tensor& b2) {
        return b1.add_(b2, alpha);
      });
}

Tensor& NestedTensor_mul__Tensor(Tensor& self, const Tensor& other) {
  return NestedTensor_elementwise__Tensor(
      self, other, "mul_", [](const Tensor& b1, const Tensor& b2) {
        return b1.mul_(b2);
      });
}

// Only usable on the C++ side; scalars are converted to tensors coming from Python.
Tensor& NestedTensor_mul__Scalar(Tensor& self, const Scalar& other) {
  return NestedTensor_mul__Tensor(self, wrapped_scalar_tensor(other));
}

Tensor& fill_nested_(Tensor& self, const Scalar& value) {
  const auto& self_buf = get_nested_tensor_impl(self)->get_buffer();
  self_buf.fill_(value);
  return self;
}

Tensor& fill_nested_(Tensor& self, const Tensor& value) {
  const auto& self_buf = get_nested_tensor_impl(self)->get_buffer();
  self_buf.fill_(value);
  return self;
}

Tensor ge_scalar_nested(const Tensor& self, const Scalar& other) {
  return NestedTensor_elementwise_Tensor(
      self, wrapped_scalar_tensor(other), "ge", false /*supports_striding*/,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_mul__Tensor, NestedTensor_mul__Scalar, fill_nested_, ge_scalar_nested, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_mul__Tensor, NestedTensor_mul__Scalar, fill_nested_, ge_scalar_nested，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-331
```cpp
      [](const Tensor& b1, const Tensor& b2) {
        return b1.ge(b2);
      });
}

Tensor gt_scalar_nested(const Tensor& self, const Scalar& other) {
  return NestedTensor_elementwise_Tensor(
      self, wrapped_scalar_tensor(other), "gt", false /*supports_striding*/,
      [](const Tensor& b1, const Tensor& b2) {
        return b1.gt(b2);
      });
}

Tensor eq_scalar_nested(const Tensor& self, const Scalar& other) {
  return NestedTensor_elementwise_Tensor(
      self, wrapped_scalar_tensor(other), "eq", false /*supports_striding*/,
      [](const Tensor& b1, const Tensor& b2) {
        return b1.eq(b2);
      });
}

Tensor eq_tensor_nested(const Tensor& self, const Tensor& other) {
  TORCH_CHECK(!other.is_nested(), "eq does not support nested tensor as other value.");
  return NestedTensor_elementwise_Tensor(
      self, other, "eq", false /*supports_striding*/,
      [](const Tensor& b1, const Tensor& b2) {
        return b1.eq(b2);
      });
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are gt_scalar_nested, eq_scalar_nested, eq_tensor_nested, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 gt_scalar_nested, eq_scalar_nested, eq_tensor_nested，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: get_elementwise_nested_tensor_impl, NestedTensor_elementwise_Tensor, wrap_buffer, NestedTensor_add_Tensor, NestedTensor_sub_Tensor, NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor.
- CN: 重要符号：get_elementwise_nested_tensor_impl, NestedTensor_elementwise_Tensor, wrap_buffer, NestedTensor_add_Tensor, NestedTensor_sub_Tensor, NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorBinaryOps.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/ScalarOps.h, ATen/TensorIndexing.h, ATen/TensorOperators.h, ATen/TensorUtils.h, ATen/core/Tensor.h`.
- CN: 主要内部头文件：`ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorBinaryOps.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/ScalarOps.h, ATen/TensorIndexing.h, ATen/TensorOperators.h, ATen/TensorUtils.h, ATen/core/Tensor.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `get_elementwise_nested_tensor_impl, NestedTensor_elementwise_Tensor, wrap_buffer, NestedTensor_add_Tensor, NestedTensor_sub_Tensor, NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor, NestedTensor_div_Scalar, NestedTensor_masked_fill`.
- CN: 实现围绕 `get_elementwise_nested_tensor_impl, NestedTensor_elementwise_Tensor, wrap_buffer, NestedTensor_add_Tensor, NestedTensor_sub_Tensor, NestedTensor_mul_Tensor, NestedTensor_mul_Scalar, NestedTensor_div_Tensor, NestedTensor_div_Scalar, NestedTensor_masked_fill` 等符号展开。
