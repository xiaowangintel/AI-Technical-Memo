# softmax.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/operators/softmax.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#include <torch/csrc/jit/tensorexpr/operators/softmax.h>

namespace torch::jit::tensorexpr {

using namespace torch::jit::tensorexpr;

Tensor computeSoftmax(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    bool log_softmax) {
  // Softmax is computed as follows:
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/operators/softmax.h. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `computeSoftmax`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/operators/softmax.h。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `computeSoftmax`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 13-24
```cpp
  //    softmax(vi) = exp(vi) / sum(exp(vi))
  //
  // In order to avoid overflow issues due to exp of a large number, we
  // subtract the max of that dim before computing exp.
  //    softmax(vi) = exp(vi - max(vi)) / sum(exp(vi - max(vi)))
  //
  // This is implemented as 4 loopnests:
  //   - First loop computes the max over the softmax dim.
  //   - Second loop computes exp for every element in v after subtracting
  //     the max of the softmax dim it belongs to.
  //   - Third loop computes the sum over the softmax dim.
  //   - Final loop computes softmax for every element in v.
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 26-37
```cpp
  // LogSoftmax is computed as follows:
  //    log_softmax(vi) = log(softmax(vi))
  //                    = vi - log(sum(exp(vi)))
  //
  // Using the same max trick as above:
  //    log_softmax(vi) = vi - max(vi) - log(sum(exp(vi - max(vi))))
  //
  // This is implemented as 5 loopnests:
  //   - First loop computes the max over the softmax dim.
  //   - Second loop computes exp for every element in v after subtracting
  //     the max of the softmax dim it belongs to.
  //   - Third loop computes the sum over the softmax dim.
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 38-49
```cpp
  //   - Fourth loop computes log for every element in the sum.
  //   - Final loop computes the log_softmax for every element in v.

  TORCH_INTERNAL_ASSERT(inputs.size() == 3);

  // We do not handle None for dims (input 1) because that is supposed to
  // be deprecated.
  TORCH_INTERNAL_ASSERT(std::get_if<int64_t>(&inputs[1]));
  int64_t rank = valueShape(inputs[0]).size();
  size_t softmax_dim =
      normalizeAndCheckIndex(std::get<int64_t>(inputs[1]), rank);
  std::vector<ExprHandle> non_softmax_dims;
```
- **EN**: This chunk continues `computeSoftmax` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `computeSoftmax`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 50-59
```cpp
  for (size_t i = 0; i < outputShape.size(); ++i) {
    if (i != softmax_dim) {
      non_softmax_dims.push_back(outputShape[i]);
    }
  }

  // Softmax implementation includes two reductions, one to find the max and
  // the other to calculate the sum along the softmax dim. These reductions
  // will have the softmax dimension as the inner most loop. So, the innermost
  // index in the indices will refer to the softmax dimension.
```
- **EN**: This chunk continues `computeSoftmax` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `computeSoftmax`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 61-72
```cpp
  // Update the indices by moving the softmax dimension index to the
  // appropriate position.
  auto move_softmax_dim_index_to_pos = [&](const ParameterList& indices) {
    std::vector<ExprHandle> new_indices;
    for (const auto& ind : indices) {
      new_indices.push_back(ind);
    }
    for (size_t i = softmax_dim; i < indices.size() - 1; ++i) {
      new_indices[i + 1] = indices[i];
    }
    new_indices[softmax_dim] = indices[indices.size() - 1];
    return new_indices;
```
- **EN**: This chunk continues `computeSoftmax` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeSoftmax`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 73-84
```cpp
  };

  // Remove the index corresponding to the softmax dimension.
  auto remove_softmax_dim_index = [&](const ParameterList& indices) {
    std::vector<ExprHandle> new_indices;
    for (size_t i = 0; i < indices.size(); ++i) {
      if (i != softmax_dim) {
        new_indices.push_back(indices[i]);
      }
    }
    return new_indices;
  };
```
- **EN**: This chunk continues `computeSoftmax` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `computeSoftmax`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 86-94
```cpp
  auto convert_indices_to_expr_handle = [&](const ParameterList& indices) {
    std::vector<ExprHandle> new_indices(indices.size());
    for (size_t i = 0; i < indices.size(); ++i) {
      new_indices[i] = indices[i];
    }
    return new_indices;
  };

  auto inp_buf = std::get<BufHandle>(inputs[0]);
```
- **EN**: This chunk defines `new_indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `new_indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 96-107
```cpp
  auto dtype = inp_buf.dtype();
  if (auto d = std::get_if<int64_t>(&inputs[2])) {
    dtype = ToDtype(static_cast<ScalarType>(*d));
  }

  auto max = Reduce(
      "aten_softmax_max",
      non_softmax_dims,
      std::nullopt,
      Maximum(dtype),
      [&](ParameterList& indices) {
        return tensorOrConstant(
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 108-119
```cpp
            inputs[0], move_softmax_dim_index_to_pos(indices));
      },
      {outputShape[softmax_dim]});
  auto e = Compute(
      "aten_softmax_exp",
      outputShape,
      std::nullopt,
      [&](ParameterList& indices) {
        auto inp = tensorOrConstant(
            inputs[0], convert_indices_to_expr_handle(indices));
        return exp(inp - max.load(remove_softmax_dim_index(indices)));
      });
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 120-131
```cpp
  auto sum = Reduce(
      "aten_softmax_sum",
      non_softmax_dims,
      std::nullopt,
      Sum(),
      [&](ParameterList& indices) {
        return e.load(move_softmax_dim_index_to_pos(indices));
      },
      {outputShape[softmax_dim]});
  if (!log_softmax) {
    auto result = Compute(
        "aten_softmax", outputShape, std::nullopt, [&](ParameterList& indices) {
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 132-138
```cpp
          return e.load(indices) / sum.load(remove_softmax_dim_index(indices));
        });
    return Tensor(
        result.buf(),
        alloc<tensorexpr::Block>(std::vector<StmtPtr>(
            {max.stmt(), e.stmt(), sum.stmt(), result.stmt()})));
  }
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 140-151
```cpp
  auto log_sum = Compute(
      "aten_softmax_log_sum",
      non_softmax_dims,
      std::nullopt,
      [&](ParameterList& indices) { return log(sum.load(indices)); });
  auto result = Compute(
      "aten_log_softmax",
      outputShape,
      std::nullopt,
      [&](ParameterList& indices) {
        auto inp = tensorOrConstant(
            inputs[0], convert_indices_to_expr_handle(indices));
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 152-162
```cpp
        auto non_softmax_indices = remove_softmax_dim_index(indices);
        return inp - max.load(non_softmax_indices) -
            log_sum.load(non_softmax_indices);
      });
  return Tensor(
      result.buf(),
      alloc<tensorexpr::Block>(std::vector<StmtPtr>(
          {max.stmt(), e.stmt(), sum.stmt(), log_sum.stmt(), result.stmt()})));
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `new_indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `new_indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **computeSoftmax**
  - EN: `computeSoftmax` is a central symbol declared or implemented in this file.
  - CN: `computeSoftmax` 是本文件声明或实现的核心符号。
- **new_indices**
  - EN: `new_indices` is a central symbol declared or implemented in this file.
  - CN: `new_indices` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Loop nest scheduling**
  - EN: Reorders, splits, and fuses loops to improve locality and code generation quality.
  - CN: 通过循环重排、切分与融合来提升局部性和代码生成质量。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/operators/softmax.h`
- **Primary symbols in this file / 本文件核心符号**: `computeSoftmax`, `new_indices`
