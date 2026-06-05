# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/operators/registry.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""Operator registry for mapping operation names to operator instances."""

from torchfuzz.operators.arg import ArgOperator
from torchfuzz.operators.argsort import ArgsortOperator
from torchfuzz.operators.base import Operator
from torchfuzz.operators.constant import ConstantOperator
from torchfuzz.operators.gather import GatherOperator
from torchfuzz.operators.index_select import IndexSelectOperator
from torchfuzz.operators.item import ItemOperator
from torchfuzz.operators.layout import (
    CatOperator,
    ChunkOperator,
    FlattenOperator,
    ReshapeOperator,
    SqueezeOperator,
    StackOperator,
    UnsqueezeOperator,
    ViewOperator,
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators.arg, torchfuzz.operators.argsort, torchfuzz.operators.base, and 5 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators.arg、torchfuzz.operators.argsort、torchfuzz.operators.base 等共 8 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 19-36
```python
)
from torchfuzz.operators.masked_select import MaskedSelectOperator
from torchfuzz.operators.matrix_multiply import (
    AddmmOperator,
    BmmOperator,
    MatmulOperator,
    MMOperator,
)
from torchfuzz.operators.nn_functional import (
    BatchNormOperator,
    DropoutOperator,
    ELUOperator,
    EmbeddingOperator,
    GELUOperator,
    GroupNormOperator,
    LayerNormOperator,
    LeakyReLUOperator,
    LinearOperator,
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators.masked_select, torchfuzz.operators.matrix_multiply, torchfuzz.operators.nn_functional.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators.masked_select、torchfuzz.operators.matrix_multiply、torchfuzz.operators.nn_functional。

### Lines 37-54
```python
    MultiHeadAttentionForwardOperator,
    ReLUOperator,
    RMSNormOperator,
    ScaledDotProductAttentionOperator,
    SigmoidOperator,
    SiLUOperator,
    SoftmaxOperator,
    TanhOperator,
)
from torchfuzz.operators.nonzero import NonzeroOperator
from torchfuzz.operators.scalar_pointwise import (
    ScalarAddOperator,
    ScalarDivOperator,
    ScalarMulOperator,
    ScalarSubOperator,
)
from torchfuzz.operators.tensor_pointwise import (
    AddOperator,
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators.nonzero, torchfuzz.operators.scalar_pointwise, torchfuzz.operators.tensor_pointwise.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators.nonzero、torchfuzz.operators.scalar_pointwise、torchfuzz.operators.tensor_pointwise。

### Lines 55-63
```python
    ClampOperator,
    CumsumOperator,
    DivOperator,
    MulOperator,
    SubOperator,
)
from torchfuzz.operators.unique import UniqueOperator
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators.unique.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators.unique。

### Lines 64-72
```python
class OperatorRegistry:
    """Registry for managing operator instances."""

    def __init__(self):
        """Initialize the registry with default operators."""
        self._operators: dict[str, Operator] = {}
        self._register_default_operators()

    def _register_default_operators(self):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as OperatorRegistry, which package state and behavior for this tooling task. This chunk defines `_register_default_operators`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 OperatorRegistry 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `_register_default_operators`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 73-81
```python
        """Register the default set of operators."""
        # Individual tensor pointwise operators (preferred)
        self.register(AddOperator())
        self.register(MulOperator())
        self.register(SubOperator())
        self.register(DivOperator())
        self.register(ClampOperator())
        self.register(CumsumOperator())
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 82-91
```python
        # Individual scalar pointwise operators (preferred)
        self.register(ScalarAddOperator())
        self.register(ScalarMulOperator())
        self.register(ScalarSubOperator())
        self.register(ScalarDivOperator())

        # Leaf Input operators
        self.register(ConstantOperator())
        self.register(ArgOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 92-100
```python
        # # Data-dependent operators
        self.register(NonzeroOperator())
        self.register(MaskedSelectOperator())
        self.register(GatherOperator())
        self.register(IndexSelectOperator())
        self.register(ArgsortOperator())
        self.register(ItemOperator())
        self.register(UniqueOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 101-110
```python
        # Tensor layout operators
        self.register(ViewOperator())
        self.register(ReshapeOperator())
        self.register(FlattenOperator())
        self.register(SqueezeOperator())
        self.register(UnsqueezeOperator())
        self.register(CatOperator())
        self.register(StackOperator())
        self.register(ChunkOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 111-122
```python
        # Matrix multiplication operators
        self.register(MMOperator())
        self.register(AddmmOperator())
        self.register(BmmOperator())
        self.register(MatmulOperator())

        # Neural network functional operators
        self.register(EmbeddingOperator())
        self.register(LinearOperator())
        self.register(ScaledDotProductAttentionOperator())
        self.register(MultiHeadAttentionForwardOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 123-132
```python
        # Activation functions
        self.register(ReLUOperator())
        self.register(LeakyReLUOperator())
        self.register(ELUOperator())
        self.register(GELUOperator())
        self.register(SiLUOperator())
        self.register(SigmoidOperator())
        self.register(TanhOperator())
        self.register(SoftmaxOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 133-141
```python
        # Normalization layers
        self.register(LayerNormOperator())
        self.register(RMSNormOperator())
        self.register(BatchNormOperator())
        self.register(GroupNormOperator())

        # Regularization
        self.register(DropoutOperator())
```
- **EN**: This chunk continues `_register_default_operators` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `_register_default_operators`，进一步展开其内部控制流或数据流转。

### Lines 142-152
```python
    def register(self, operator: Operator):
        """Register an operator in the registry."""
        self._operators[operator.name] = operator

    def get(self, op_name: str) -> Operator | None:
        """Get an operator by name."""
        # Handle special arg_ operations by mapping them to the ArgOperator
        if op_name.startswith("arg_"):
            return self._operators.get("arg")
        return self._operators.get(op_name)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `get`, which implements a focused step inside the torch fuzzing pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `get`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 153-161
```python
    def list_operators(self) -> dict[str, Operator]:
        """List all registered operators."""
        return self._operators.copy()


# Global registry instance
_global_registry = OperatorRegistry()
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `list_operators`, which implements a focused step inside the torch fuzzing pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `list_operators`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 162-170
```python
def get_operator(op_name: str) -> Operator | None:
    """Get an operator from the global registry."""
    return _global_registry.get(op_name)


def register_operator(operator: Operator):
    """Register an operator in the global registry."""
    _global_registry.register(operator)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `register_operator`, which implements a focused step inside the torch fuzzing pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `register_operator`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 171-179
```python

def list_operators() -> dict[str, Operator]:
    """List all operators in the global registry."""
    return _global_registry.list_operators()


def set_operator_weight(op_name: str, weight: float) -> None:
    """Set the selection weight for a specific operator.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `set_operator_weight`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `set_operator_weight`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 180-190
```python
    Args:
        op_name: The registered operator name (e.g., "add", "arg") OR fully-qualified torch op
                 (e.g., "torch.nn.functional.relu", "torch.matmul")
        weight: New relative selection weight (must be > 0)
    """
    if weight <= 0:
        raise ValueError("Operator weight must be > 0")

    # Try by registry key
    op = _global_registry.get(op_name)
    if op is not None:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `set_operator_weight` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `set_operator_weight`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 191-199
```python
        op.weight = float(weight)
        return

    # Fallback: try to locate by fully-qualified torch op name
    for candidate in _global_registry.list_operators().values():
        if getattr(candidate, "torch_op_name", None) == op_name:
            candidate.weight = float(weight)
            return
```
- **EN**: This chunk continues `set_operator_weight` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `set_operator_weight`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 200-208
```python
    raise KeyError(f"Operator '{op_name}' not found by registry name or torch op name")


def set_operator_weights(weights: dict[str, float]) -> None:
    """Bulk-update operator weights from a mapping of name -> weight."""
    for name, w in weights.items():
        set_operator_weight(name, w)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `set_operator_weights`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `set_operator_weights`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 209-218
```python
def set_operator_weight_by_torch_op(torch_op_name: str, weight: float) -> None:
    """Set operator weight by fully-qualified torch op name."""
    if weight <= 0:
        raise ValueError("Operator weight must be > 0")
    for candidate in _global_registry.list_operators().values():
        if getattr(candidate, "torch_op_name", None) == torch_op_name:
            candidate.weight = float(weight)
            return
    raise KeyError(f"Torch op '{torch_op_name}' not found in registry")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `set_operator_weight_by_torch_op`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `set_operator_weight_by_torch_op`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 219-223
```python

def set_operator_weights_by_torch_op(weights: dict[str, float]) -> None:
    """Bulk-update weights by fully-qualified torch op names."""
    for name, w in weights.items():
        set_operator_weight_by_torch_op(name, w)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `set_operator_weights_by_torch_op`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `set_operator_weights_by_torch_op`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
- **OperatorRegistry**
  - EN: `OperatorRegistry` is one of the main local symbols exposed or implemented here.
  - CN: `OperatorRegistry` 是此处暴露或实现的主要局部符号之一。
- **get_operator**
  - EN: `get_operator` is one of the main local symbols exposed or implemented here.
  - CN: `get_operator` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **External packages / 外部依赖包**: `torchfuzz.operators.arg`, `torchfuzz.operators.argsort`, `torchfuzz.operators.base`, `torchfuzz.operators.constant`, `torchfuzz.operators.gather`, `torchfuzz.operators.index_select`, `torchfuzz.operators.item`, `torchfuzz.operators.layout`, `torchfuzz.operators.masked_select`, `torchfuzz.operators.matrix_multiply`, `torchfuzz.operators.nn_functional`, `torchfuzz.operators.nonzero`, `torchfuzz.operators.scalar_pointwise`, `torchfuzz.operators.tensor_pointwise`, `torchfuzz.operators.unique`
- **Primary symbols in this file / 本文件核心符号**: `OperatorRegistry`, `get_operator`, `register_operator`, `list_operators`, `set_operator_weight`, `set_operator_weights`, `set_operator_weight_by_torch_op`, `set_operator_weights_by_torch_op`
