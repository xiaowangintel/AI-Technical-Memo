# tensor.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/tensor.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates expression nodes in the Tensor Expression subsystem.
- **Purpose (CN)**: 定义或操作 Tensor Expression 子系统中的表达式节点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```cpp
#include <torch/csrc/jit/tensorexpr/tensor.h>

#include <c10/util/irange.h>
#include <torch/csrc/jit/tensorexpr/reduction.h>

namespace torch::jit::tensorexpr {

StmtPtr Tensor::constructStmt(
    const std::vector<VarPtr>& args,
    const ExprPtr& body,
    const std::vector<ExprPtr>& reduce_dims,
    const std::vector<VarPtr>& reduce_args) const {
  std::vector<ExprPtr> indices(args.begin(), args.end());
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/tensor.h, torch/csrc/jit/tensorexpr/reduction.h; ATen/c10 facilities such as c10/util/irange.h. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/tensor.h、torch/csrc/jit/tensorexpr/reduction.h；ATen/c10 基础设施，如 c10/util/irange.h。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 15-30
```cpp
  size_t ndim = buf()->ndim();
  size_t reduce_ndim = reduce_dims.size();
  auto reduce_op = to<ReduceOp>(body);
  auto acc_buf = reduce_ndim > 0 ? reduce_op->getAccBuf() : nullptr;

  StmtPtr s = alloc<Store>(buf_, indices, body);
  if (reduce_ndim > 0) {
    TORCH_INTERNAL_ASSERT(reduce_op != nullptr);
    if (acc_buf != nullptr) {
      auto reducer = reduce_op->reducer();
      std::vector<ExprPtr> output_args(args.begin(), args.end());
      ExprPtr new_reduce_op = reducer(
          to<Buf>(acc_buf),
          alloc<Cast>(acc_buf->dtype(), reduce_op->getRiOperand()),
          output_args,
          reduce_args);
```
- **EN**: This chunk defines `output_args`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `output_args`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 31-41
```cpp
      new_reduce_op->set_dtype(acc_buf->dtype());
      s = alloc<Store>(to<Buf>(acc_buf), indices, new_reduce_op);
    }
  }

  if (ndim == 0 && reduce_ndim == 0) {
    return s;
  }

  if (reduce_ndim > 0) {
    TORCH_INTERNAL_ASSERT(reduce_op != nullptr);
```
- **EN**: This chunk continues `output_args` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `output_args`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 43-57
```cpp
    for (const auto i : c10::irange(reduce_ndim)) {
      // Going in reverse order: from innermost loop to the outermost
      size_t dim_index = reduce_ndim - i - 1;
      auto const& dim = reduce_dims[dim_index];
      s = alloc<For>(reduce_args[dim_index], immLike(dim, 0), dim, s);
    }
    s = alloc<Block>(std::vector<StmtPtr>({s}));

    BufPtr init_buf = acc_buf ? to<Buf>(acc_buf) : buf();
    ExprPtr init_expr =
        acc_buf ? to<Buf>(acc_buf)->initializer() : buf()->initializer();
    if (init_expr) {
      StorePtr init_stmt = alloc<Store>(init_buf, indices, init_expr);
      to<Block>(s)->prepend_stmt(init_stmt);
    }
```
- **EN**: This chunk continues `output_args` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `output_args`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 59-71
```cpp
    if (acc_buf != nullptr) {
      LoadPtr load_acc = alloc<Load>(acc_buf, indices);
      auto cast = alloc<Cast>(buf()->dtype(), load_acc);
      StorePtr post_stmt = alloc<Store>(buf(), indices, cast);
      to<Block>(s)->append_stmt(post_stmt);
    }
  }

  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(
      buf_->is_contiguous() ||
      buf_->is_contiguous(at::MemoryFormat::ChannelsLast) ||
      buf_->is_contiguous(at::MemoryFormat::ChannelsLast3d) ||
      buf_->is_channels_last_1d_contiguous());
```
- **EN**: This chunk continues `output_args` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `output_args`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 73-85
```cpp
  auto loop_order_fn = [&]() {
    std::vector<int32_t> loop_order;
    if (buf_->is_contiguous()) {
      for (int32_t i = args.size() - 1; i >= 0; i--) {
        loop_order.push_back(i);
      }
    } else if (buf_->is_contiguous(c10::MemoryFormat::ChannelsLast)) {
      loop_order = {1, 3, 2, 0};
    } else if (buf_->is_contiguous(c10::MemoryFormat::ChannelsLast3d)) {
      loop_order = {1, 4, 3, 2, 0};
    } else {
      loop_order = {1, 2, 0};
    }
```
- **EN**: This chunk continues `output_args` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `output_args`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 87-96
```cpp
    return loop_order;
  };

  auto loop_order = loop_order_fn();
  for (auto dim_index : loop_order) {
    auto const& dim = buf()->dim(dim_index);
    s = alloc<For>(args[dim_index], immLike(dim, 0), dim, s);
  }
  return s;
}
```
- **EN**: This chunk continues `output_args` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `output_args`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 98-113
```cpp
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const std::function<ExprHandle(const std::vector<VarHandle>&)>& body_func) {
  std::vector<VarHandle> args = create_index_vars(dims);
  ExprHandle body = body_func(args);
  BufHandle buf = Buf::make(name, dims, body.dtype(), std::nullopt, strides);
  return Tensor(buf, args, body);
}
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::function<ExprHandle(const std::vector<VarHandle>&)>& body_func) {
  return Compute(name, dims, std::nullopt, body_func);
}
```
- **EN**: This chunk defines `Compute`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `Compute`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 115-130
```cpp
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const std::function<ExprHandle(const VarHandle&)>& body_func) {
  if (dims.size() != 1) {
    throw malformed_input("mismatch between body and arg size (1)");
  }

  std::vector<VarHandle> args = create_index_vars(dims);
  ExprHandle body = body_func(args[0]);
  BufHandle buf = Buf::make(name, dims, body.dtype(), std::nullopt, strides);
  return Tensor(buf, args, body);
}
Tensor Compute(
    const std::string& name,
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 131-146
```cpp
    const std::vector<ExprHandle>& dims,
    const std::function<ExprHandle(const VarHandle&)>& body_func) {
  return Compute(name, dims, std::nullopt, body_func);
}

Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const std::function<ExprHandle(const VarHandle&, const VarHandle&)>&
        body_func) {
  if (dims.size() != 2) {
    throw malformed_input("mismatch between body and arg size (2)");
  }
  std::vector<VarHandle> args = create_index_vars(dims);
  ExprHandle body = body_func(args[0], args[1]);
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 147-156
```cpp
  BufHandle buf = Buf::make(name, dims, body.dtype(), std::nullopt, strides);
  return Tensor(buf, args, body);
}
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::function<ExprHandle(const VarHandle&, const VarHandle&)>&
        body_func) {
  return Compute(name, dims, std::nullopt, body_func);
}
```
- **EN**: This chunk defines `Compute`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `Compute`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 158-173
```cpp
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const std::function<
        ExprHandle(const VarHandle&, const VarHandle&, const VarHandle&)>&
        body_func) {
  if (dims.size() != 3) {
    throw malformed_input("mismatch between body and arg size (3)");
  }
  std::vector<VarHandle> args = create_index_vars(dims);
  ExprHandle body = body_func(args[0], args[1], args[2]);
  BufHandle buf = Buf::make(name, dims, body.dtype(), std::nullopt, strides);
  return Tensor(buf, args, body);
}
Tensor Compute(
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 174-189
```cpp
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::function<
        ExprHandle(const VarHandle&, const VarHandle&, const VarHandle&)>&
        body_func) {
  return Compute(name, dims, std::nullopt, body_func);
}

Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const std::function<ExprHandle(
        const VarHandle&,
        const VarHandle&,
        const VarHandle&,
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 190-205
```cpp
        const VarHandle&)>& body_func) {
  if (dims.size() != 4) {
    throw malformed_input("mismatch between body and arg size (4)");
  }
  std::vector<VarHandle> args = create_index_vars(dims);
  ExprHandle body = body_func(args[0], args[1], args[2], args[3]);
  BufHandle buf = Buf::make(name, dims, body.dtype(), std::nullopt, strides);
  return Tensor(buf, args, body);
}
Tensor Compute(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::function<ExprHandle(
        const VarHandle&,
        const VarHandle&,
        const VarHandle&,
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 206-221
```cpp
        const VarHandle&)>& body_func) {
  return Compute(name, dims, std::nullopt, body_func);
}

Tensor Reduce(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const Reducer& reducer,
    const BufHandle& buffer,
    const std::vector<ExprHandle>& reduce_dims) {
  return Reduce(
      name,
      dims,
      strides,
      reducer,
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 222-232
```cpp
      [&](ParameterList& p) { return buffer.load(p); },
      reduce_dims);
}
Tensor Reduce(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const Reducer& reducer,
    const BufHandle& buffer,
    const std::vector<ExprHandle>& reduce_dims) {
  return Reduce(name, dims, std::nullopt, reducer, buffer, reduce_dims);
}
```
- **EN**: This chunk defines `Reduce`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `Reduce`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 234-249
```cpp
Tensor Reduce(
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const std::optional<std::vector<ExprHandle>>& strides,
    const Reducer& reducer,
    const Tensor& tensor,
    const std::vector<ExprHandle>& reduce_dims) {
  return Reduce(
      name,
      dims,
      strides,
      reducer,
      [&](ParameterList& p) { return tensor.load(p); },
      reduce_dims);
}
Tensor Reduce(
```
- **EN**: This chunk continues `Reduce` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `Reduce`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 250-258
```cpp
    const std::string& name,
    const std::vector<ExprHandle>& dims,
    const Reducer& reducer,
    const Tensor& tensor,
    const std::vector<ExprHandle>& reduce_dims) {
  return Reduce(name, dims, std::nullopt, reducer, tensor, reduce_dims);
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `Reduce` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `Reduce`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **constructStmt**
  - EN: `constructStmt` is a central symbol declared or implemented in this file.
  - CN: `constructStmt` 是本文件声明或实现的核心符号。
- **indices**
  - EN: `indices` is a central symbol declared or implemented in this file.
  - CN: `indices` 是本文件声明或实现的核心符号。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/tensor.h`, `torch/csrc/jit/tensorexpr/reduction.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `c10/util/irange.h`
- **Primary symbols in this file / 本文件核心符号**: `constructStmt`, `indices`, `output_args`, `Compute`, `malformed_input`, `Reduce`
