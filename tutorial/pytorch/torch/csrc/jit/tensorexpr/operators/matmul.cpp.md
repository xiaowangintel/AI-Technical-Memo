# matmul.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/operators/matmul.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```cpp
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/operators/matmul.h>

namespace torch::jit::tensorexpr {
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/tensorexpr/ir_simplifier.h, torch/csrc/jit/tensorexpr/operators/matmul.h. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/tensorexpr/ir_simplifier.h、torch/csrc/jit/tensorexpr/operators/matmul.h。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。

### Lines 6-13
```cpp
Tensor computeMatmul(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  Dtype dtype = kFloat;
  if (outputType) {
```
- **EN**: This chunk defines `computeMatmul`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `computeMatmul`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 14-18
```cpp
    dtype = Dtype(*outputType);
  }
  BufHandle ResultBuf("matmul", outputShape, dtype);
  const BufHandle a = std::get<BufHandle>(inputs[0]);
  const BufHandle b = std::get<BufHandle>(inputs[1]);
```
- **EN**: This chunk declares `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation.
- **CN**: 这一段声明了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。

### Lines 20-27
```cpp
  auto size_a = a.dims();
  auto size_b = b.dims();
  // We currently only support rank 2 matmuls
  TORCH_INTERNAL_ASSERT(size_a.size() == 2 && size_b.size() == 2);
  auto total_size =
      to<LongImm>(IRSimplifier::simplify(
                      cast<int64_t>(size_a[0]) * cast<int64_t>(size_a[1]) *
                      cast<int64_t>(size_b[1]))
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 28-35
```cpp
                      .node());

  // For small sizes, where N*M*K < 1000, lower matmul to a naive 3-level
  // loopnest. The number is not tuned very carefully, and in future we should
  // fine-tune it as well as we should add more advanced native TE lowerings for
  // matmuls. For bigger sizes we generate a TE ExternalCall, which would call
  // an aten::matmul.
  // Native, even naive, lowering is beneficial when the sizes are small because
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。

### Lines 36-43
```cpp
  // it allows to eliminate dispatch overhead.
  if (total_size && total_size->value() < 1000) {
    return Reduce(
        "nnc_matmul",
        {size_a[0], size_b[1]},
        Sum(),
        [&](const ExprHandle& m, const ExprHandle& n, const ExprHandle& k) {
          return Load::make(a, {m, k}) * Load::make(b, {k, n});
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 44-51
```cpp
        },
        {size_a[1]});
  } else {
    return Tensor(
        ResultBuf.node(),
        ExternalCall::make(ResultBuf, "nnc_aten_matmul", {a, b}, {}));
  }
}
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 53-60
```cpp
Tensor computeAddMM(
    const std::vector<ArgValue>& inputs,
    const std::vector<ExprHandle>& outputShape,
    const std::vector<ExprHandle>& outputStrides,
    const std::optional<ScalarType>& outputType,
    at::Device device) {
  Dtype dtype = kFloat;
  if (outputType) {
```
- **EN**: This chunk defines `computeAddMM`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `computeAddMM`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 61-68
```cpp
    dtype = Dtype(*outputType);
  }
  BufHandle ResultBuf("addmm", outputShape, dtype);
  return Tensor(
      ResultBuf.node(),
      ExternalCall::make(
          ResultBuf,
          "nnc_aten_addmm",
```
- **EN**: This chunk declares `ResultBuf`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `ResultBuf`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 69-75
```cpp
          {std::get<BufHandle>(inputs[0]),
           std::get<BufHandle>(inputs[1]),
           std::get<BufHandle>(inputs[2])},
          {std::get<int64_t>(inputs[3]),
           std::get<int64_t>(
               inputs[4])})); // TODO: handle other dtypes of alpha and beta
}
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。

### Lines 77-77
```cpp
} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk continues `ResultBuf` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ResultBuf`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **computeMatmul**
  - EN: `computeMatmul` is a central symbol declared or implemented in this file.
  - CN: `computeMatmul` 是本文件声明或实现的核心符号。
- **ResultBuf**
  - EN: `ResultBuf` is a central symbol declared or implemented in this file.
  - CN: `ResultBuf` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Loop nest scheduling**
  - EN: Reorders, splits, and fuses loops to improve locality and code generation quality.
  - CN: 通过循环重排、切分与融合来提升局部性和代码生成质量。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/operators/matmul.h`
- **Primary symbols in this file / 本文件核心符号**: `computeMatmul`, `ResultBuf`, `computeAddMM`
