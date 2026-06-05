# matrix_multiply.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/operators/matrix_multiply.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
"""Matrix multiplication operator implementations."""

import random

import torch

from torchfuzz.operators.base import Operator
from torchfuzz.tensor_fuzzer import Spec, TensorSpec
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as torch; Python standard-library modules such as random; external packages such as torchfuzz.operators.base, torchfuzz.tensor_fuzzer. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 torch；Python 标准库模块，如 random；外部依赖包，如 torchfuzz.operators.base、torchfuzz.tensor_fuzzer。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。

### Lines 10-19
```python

# Type promotion imports removed since we now use explicit casting in codegen


class MatrixMultiplyOperator(Operator):
    """Base class for matrix multiplication operations."""

    def __init__(self, name: str):
        super().__init__(name)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as MatrixMultiplyOperator, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 MatrixMultiplyOperator 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 20-28
```python
    def can_produce(self, output_spec: Spec) -> bool:
        """Matrix multiply operations can produce float/complex tensors of dimension >= 2."""
        if not isinstance(output_spec, TensorSpec):
            return False

        # Must have at least 2 dimensions for matrix multiplication
        if len(output_spec.size) < 2:
            return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `can_produce`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `can_produce`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 29-38
```python
        # Matrix multiply doesn't work with bool or integer types for gradients
        if output_spec.dtype in [
            torch.bool,
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
        ]:
            return False
```
- **EN**: This chunk continues `can_produce` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `can_produce`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 39-48
```python
        return True

    def _get_compatible_dtype(self, output_dtype):
        """Get a compatible dtype for matrix multiplication."""
        return [output_dtype, output_dtype]


class MMOperator(MatrixMultiplyOperator):
    """Operator for matrix multiplication (torch.mm)."""
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as MMOperator, which package state and behavior for this tooling task. This chunk defines `_get_compatible_dtype`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 MMOperator 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `_get_compatible_dtype`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 49-57
```python
    def __init__(self):
        super().__init__("mm")
        self.weight = 5.0

    @property
    def torch_op_name(self) -> str | None:
        """Return the torch operation name."""
        return "torch.mm"
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `torch_op_name`, which implements a focused step inside the torch fuzzing pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `torch_op_name`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 58-66
```python
    def can_produce(self, output_spec: Spec) -> bool:
        """MM requires exactly 2D tensors."""
        if not isinstance(output_spec, TensorSpec):
            return False

        # Must have exactly 2 dimensions for torch.mm
        if len(output_spec.size) != 2:
            return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `can_produce`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `can_produce`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 67-76
```python
        # Matrix multiply doesn't work with bool or integer types for gradients
        if output_spec.dtype in [
            torch.bool,
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
        ]:
            return False
```
- **EN**: This chunk continues `can_produce` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `can_produce`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 77-86
```python
        return True

    def fuzz_inputs_specs(self, output_spec: Spec) -> list[Spec]:
        """Generate input specs for matrix multiplication."""
        if not isinstance(output_spec, TensorSpec):
            raise ValueError("MMOperator can only produce TensorSpec outputs")

        if len(output_spec.size) != 2:
            raise ValueError("torch.mm requires 2D tensors")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fuzz_inputs_specs`, which constructs randomized inputs to probe edge cases and robustness boundaries. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fuzz_inputs_specs`，其作用是构造随机输入以探测边界情况与健壮性极限。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 87-99
```python
        m, n = output_spec.size
        # Choose a random inner dimension k
        k = random.randint(1, 16)

        dtypes = self._get_compatible_dtype(output_spec.dtype)

        # First tensor: [m, k]
        input1_spec = TensorSpec(
            size=(m, k),
            stride=(k, 1),  # Contiguous stride
            dtype=dtypes[0],
        )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。

### Lines 100-108
```python
        # Second tensor: [k, n]
        input2_spec = TensorSpec(
            size=(k, n),
            stride=(n, 1),  # Contiguous stride
            dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
        )

        return [input1_spec, input2_spec]
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 109-117
```python
    def codegen(
        self, output_name: str, input_names: list[str], output_spec: Spec
    ) -> str:
        """Generate code for matrix multiplication."""
        if len(input_names) != 2:
            raise ValueError("torch.mm requires exactly 2 inputs")

        # Get target dtype
        if isinstance(output_spec, TensorSpec):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `codegen`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `codegen`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 118-129
```python
            target_dtype_str = f"torch.{output_spec.dtype}".replace(
                "torch.torch.", "torch."
            )
            # Cast inputs to ensure compatible types
            return (
                f"{output_name} = torch.mm("
                f"{input_names[0]}.to({target_dtype_str}), "
                f"{input_names[1]}.to({target_dtype_str}))"
            )
        else:
            return f"{output_name} = torch.mm({input_names[0]}, {input_names[1]})"
```
- **EN**: This chunk continues `codegen` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `codegen`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 130-138
```python

class AddmmOperator(MatrixMultiplyOperator):
    """Operator for additive matrix multiplication (torch.addmm)."""

    def __init__(self):
        super().__init__("addmm")
        self.weight = 5.0

    @property
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as AddmmOperator, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 AddmmOperator 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 139-147
```python
    def torch_op_name(self) -> str | None:
        """Return the torch operation name."""
        return "torch.addmm"

    def can_produce(self, output_spec: Spec) -> bool:
        """Addmm requires exactly 2D tensors."""
        if not isinstance(output_spec, TensorSpec):
            return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `can_produce`, which implements a focused step inside the torch fuzzing pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `can_produce`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 148-161
```python
        # Must have exactly 2 dimensions for torch.addmm
        if len(output_spec.size) != 2:
            return False

        # Matrix multiply doesn't work with bool or integer types for gradients
        if output_spec.dtype in [
            torch.bool,
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
        ]:
            return False
```
- **EN**: This chunk continues `can_produce` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `can_produce`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 162-171
```python
        return True

    def fuzz_inputs_specs(self, output_spec: Spec) -> list[Spec]:
        """Generate input specs for additive matrix multiplication."""
        if not isinstance(output_spec, TensorSpec):
            raise ValueError("AddmmOperator can only produce TensorSpec outputs")

        if len(output_spec.size) != 2:
            raise ValueError("torch.addmm requires 2D output tensor")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fuzz_inputs_specs`, which constructs randomized inputs to probe edge cases and robustness boundaries. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fuzz_inputs_specs`，其作用是构造随机输入以探测边界情况与健壮性极限。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 172-184
```python
        m, n = output_spec.size
        # Choose a random inner dimension k
        k = random.randint(1, 16)

        dtypes = self._get_compatible_dtype(output_spec.dtype)

        # Bias tensor: [m, n] (same shape as output)
        bias_spec = TensorSpec(
            size=(m, n),
            stride=(n, 1),  # Contiguous stride
            dtype=dtypes[0],
        )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。

### Lines 185-198
```python
        # First matrix: [m, k]
        input1_spec = TensorSpec(
            size=(m, k),
            stride=(k, 1),  # Contiguous stride
            dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
        )

        # Second matrix: [k, n]
        input2_spec = TensorSpec(
            size=(k, n),
            stride=(n, 1),  # Contiguous stride
            dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
        )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 199-207
```python
        return [bias_spec, input1_spec, input2_spec]

    def codegen(
        self, output_name: str, input_names: list[str], output_spec: Spec
    ) -> str:
        """Generate code for additive matrix multiplication."""
        if len(input_names) != 3:
            raise ValueError("torch.addmm requires exactly 3 inputs")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `codegen`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `codegen`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 208-222
```python
        # Get target dtype
        if isinstance(output_spec, TensorSpec):
            target_dtype_str = f"torch.{output_spec.dtype}".replace(
                "torch.torch.", "torch."
            )
            # Cast inputs to ensure compatible types
            return (
                f"{output_name} = torch.addmm("
                f"{input_names[0]}.to({target_dtype_str}), "
                f"{input_names[1]}.to({target_dtype_str}), "
                f"{input_names[2]}.to({target_dtype_str}))"
            )
        else:
            return f"{output_name} = torch.addmm({input_names[0]}, {input_names[1]}, {input_names[2]})"
```
- **EN**: This chunk continues `codegen` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `codegen`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 223-231
```python

class BmmOperator(MatrixMultiplyOperator):
    """Operator for batch matrix multiplication (torch.bmm)."""

    def __init__(self):
        super().__init__("bmm")
        self.weight = 5.0

    @property
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as BmmOperator, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 BmmOperator 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 232-240
```python
    def torch_op_name(self) -> str | None:
        """Return the torch operation name."""
        return "torch.bmm"

    def can_produce(self, output_spec: Spec) -> bool:
        """Batch matrix multiply requires 3D tensors."""
        if not isinstance(output_spec, TensorSpec):
            return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `can_produce`, which implements a focused step inside the torch fuzzing pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `can_produce`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 241-254
```python
        # Must have exactly 3 dimensions for batch matrix multiplication
        if len(output_spec.size) != 3:
            return False

        # Matrix multiply doesn't work with bool or integer types for gradients
        if output_spec.dtype in [
            torch.bool,
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
        ]:
            return False
```
- **EN**: This chunk continues `can_produce` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `can_produce`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 255-264
```python
        return True

    def fuzz_inputs_specs(self, output_spec: Spec) -> list[Spec]:
        """Generate input specs for batch matrix multiplication."""
        if not isinstance(output_spec, TensorSpec):
            raise ValueError("BmmOperator can only produce TensorSpec outputs")

        if len(output_spec.size) != 3:
            raise ValueError("torch.bmm requires 3D tensors")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fuzz_inputs_specs`, which constructs randomized inputs to probe edge cases and robustness boundaries. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fuzz_inputs_specs`，其作用是构造随机输入以探测边界情况与健壮性极限。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 265-277
```python
        b, m, n = output_spec.size
        # Choose a random inner dimension k
        k = random.randint(1, 16)

        dtypes = self._get_compatible_dtype(output_spec.dtype)

        # First tensor: [b, m, k]
        input1_spec = TensorSpec(
            size=(b, m, k),
            stride=(m * k, k, 1),  # Contiguous stride
            dtype=dtypes[0],
        )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。

### Lines 278-286
```python
        # Second tensor: [b, k, n]
        input2_spec = TensorSpec(
            size=(b, k, n),
            stride=(k * n, n, 1),  # Contiguous stride
            dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
        )

        return [input1_spec, input2_spec]
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 287-295
```python
    def codegen(
        self, output_name: str, input_names: list[str], output_spec: Spec
    ) -> str:
        """Generate code for batch matrix multiplication."""
        if len(input_names) != 2:
            raise ValueError("torch.bmm requires exactly 2 inputs")

        # Get target dtype
        if isinstance(output_spec, TensorSpec):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `codegen`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `codegen`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 296-307
```python
            target_dtype_str = f"torch.{output_spec.dtype}".replace(
                "torch.torch.", "torch."
            )
            # Cast inputs to ensure compatible types
            return (
                f"{output_name} = torch.bmm("
                f"{input_names[0]}.to({target_dtype_str}), "
                f"{input_names[1]}.to({target_dtype_str}))"
            )
        else:
            return f"{output_name} = torch.bmm({input_names[0]}, {input_names[1]})"
```
- **EN**: This chunk continues `codegen` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `codegen`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 308-316
```python

class MatmulOperator(MatrixMultiplyOperator):
    """Operator for general matrix multiplication (torch.matmul)."""

    def __init__(self):
        super().__init__("matmul")
        self.weight = 500.0

    @property
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as MatmulOperator, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 MatmulOperator 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 317-325
```python
    def torch_op_name(self) -> str | None:
        """Return the torch operation name."""
        return "torch.matmul"

    def can_produce(self, output_spec: Spec) -> bool:
        """Matmul can handle various tensor dimensions >= 1."""
        if not isinstance(output_spec, TensorSpec):
            return False
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `can_produce`, which implements a focused step inside the torch fuzzing pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `can_produce`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 326-339
```python
        # Must have at least 1 dimension
        if len(output_spec.size) < 1:
            return False

        # Matrix multiply doesn't work with bool or integer types for gradients
        if output_spec.dtype in [
            torch.bool,
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
        ]:
            return False
```
- **EN**: This chunk continues `can_produce` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `can_produce`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 340-349
```python
        return True

    def fuzz_inputs_specs(self, output_spec: Spec) -> list[Spec]:
        """Generate input specs for general matrix multiplication."""
        if not isinstance(output_spec, TensorSpec):
            raise ValueError("MatmulOperator can only produce TensorSpec outputs")

        output_size = output_spec.size
        output_dims = len(output_size)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fuzz_inputs_specs`, which constructs randomized inputs to probe edge cases and robustness boundaries. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fuzz_inputs_specs`，其作用是构造随机输入以探测边界情况与健壮性极限。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 350-358
```python
        dtypes = self._get_compatible_dtype(output_spec.dtype)

        if output_dims == 1:
            # Matrix-vector multiplication: (n,) = (k,) @ (k, n) or (n,) = (n, k) @ (k,)
            n = output_size[0]
            k = random.randint(1, 16)

            # Randomly choose between two valid patterns
            if random.choice([True, False]):
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 359-374
```python
                # Pattern 1: (n,) = (k,) @ (k, n)
                input1_spec = TensorSpec(size=(k,), stride=(1,), dtype=dtypes[0])
                input2_spec = TensorSpec(
                    size=(k, n),
                    stride=(n, 1),
                    dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
                )
            else:
                # Pattern 2: (n,) = (n, k) @ (k,)
                input1_spec = TensorSpec(size=(n, k), stride=(k, 1), dtype=dtypes[0])
                input2_spec = TensorSpec(
                    size=(k,),
                    stride=(1,),
                    dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
                )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 375-386
```python
        elif output_dims == 2:
            # Matrix multiplication: (m, n) = (m, k) @ (k, n)
            m, n = output_size
            k = random.randint(1, 16)

            input1_spec = TensorSpec(size=(m, k), stride=(k, 1), dtype=dtypes[0])
            input2_spec = TensorSpec(
                size=(k, n),
                stride=(n, 1),
                dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
            )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 387-395
```python
        else:
            # Batched matrix multiplication: (..., m, n) = (..., m, k) @ (..., k, n)
            *batch_dims, m, n = output_size
            k = random.randint(1, 16)

            # Calculate strides for contiguous tensors
            input1_size = tuple(batch_dims + [m, k])
            input2_size = tuple(batch_dims + [k, n])
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 396-406
```python
            # Contiguous strides
            input1_stride = [1]
            for i in reversed(range(len(input1_size) - 1)):
                input1_stride.append(input1_stride[-1] * input1_size[i + 1])
            input1_stride = tuple(reversed(input1_stride))

            input2_stride = [1]
            for i in reversed(range(len(input2_size) - 1)):
                input2_stride.append(input2_stride[-1] * input2_size[i + 1])
            input2_stride = tuple(reversed(input2_stride))
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 407-415
```python
            input1_spec = TensorSpec(
                size=input1_size, stride=input1_stride, dtype=dtypes[0]
            )
            input2_spec = TensorSpec(
                size=input2_size,
                stride=input2_stride,
                dtype=dtypes[1] if len(dtypes) > 1 else dtypes[0],
            )
```
- **EN**: This chunk continues `fuzz_inputs_specs` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_inputs_specs`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 416-424
```python
        return [input1_spec, input2_spec]

    def codegen(
        self, output_name: str, input_names: list[str], output_spec: Spec
    ) -> str:
        """Generate code for general matrix multiplication."""
        if len(input_names) != 2:
            raise ValueError("torch.matmul requires exactly 2 inputs")
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `codegen`, which generates derived source text, templates, or metadata outputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `codegen`，其作用是生成派生源码文本、模板或元数据输出。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 425-437
```python
        # Get target dtype
        if isinstance(output_spec, TensorSpec):
            target_dtype_str = f"torch.{output_spec.dtype}".replace(
                "torch.torch.", "torch."
            )
            # Cast inputs to ensure compatible types
            return (
                f"{output_name} = torch.matmul("
                f"{input_names[0]}.to({target_dtype_str}), "
                f"{input_names[1]}.to({target_dtype_str}))"
            )
        else:
            return f"{output_name} = torch.matmul({input_names[0]}, {input_names[1]})"
```
- **EN**: This chunk continues `codegen` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `codegen`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
- **MatrixMultiplyOperator**
  - EN: `MatrixMultiplyOperator` is one of the main local symbols exposed or implemented here.
  - CN: `MatrixMultiplyOperator` 是此处暴露或实现的主要局部符号之一。
- **MMOperator**
  - EN: `MMOperator` is one of the main local symbols exposed or implemented here.
  - CN: `MMOperator` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `torch`
- **Python standard library / Python 标准库**: `random`
- **External packages / 外部依赖包**: `torchfuzz.operators.base`, `torchfuzz.tensor_fuzzer`
- **Primary symbols in this file / 本文件核心符号**: `MatrixMultiplyOperator`, `MMOperator`, `AddmmOperator`, `BmmOperator`, `MatmulOperator`
