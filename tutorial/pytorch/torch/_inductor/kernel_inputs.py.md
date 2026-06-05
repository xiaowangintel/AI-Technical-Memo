# kernel_inputs.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel_inputs.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `KernelInputs`, and `MMKernelInputs`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `KernelInputs`、`MMKernelInputs` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import Any, TYPE_CHECKING

import torch
import torch._inductor.config
from torch._inductor import ir
from torch._inductor.virtualized import V

from .ir import FixedLayout, FlexibleLayout, Layout


if TYPE_CHECKING:
    from collections.abc import Sequence

    import sympy


class KernelInputs(ABC):
````
- **EN**: Imports dependencies such as `__future__`, `abc`, `typing`, `torch`, `torch._inductor.config`, `torch._inductor`, and `...+4` for the logic in this range. Introduces class `KernelInputs`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`abc`、`typing`、`torch`、`torch._inductor.config`、`torch._inductor`、`另有4项` 等依赖，为后续逻辑提供基础能力。这里定义了类`KernelInputs`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
    """
    Class to store and provide access to input nodes for kernels.
    This class takes in a tuple of input nodes and provides methods to access
    information about these nodes, such as their device type and device.
    """

    def __init__(
        self,
        input_nodes: list[Any],
        scalars: dict[str, float | int] | None = None,
        out_dtype: torch.dtype | None = None,
    ):
        """
        Initialize with a tuple of input nodes.

        Args:
            input_nodes: A tuple of input nodes to store
            out_dtype: Optional output dtype to store
        """
        self._input_nodes = input_nodes
````
- **EN**: Introduces function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `input_nodes`, `scalars`, `out_dtype`, and `Args`.
- **CN**: 这里定义了函数`__init__`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `input_nodes`、`scalars`、`out_dtype`、`Args` 等值。

### Lines 41-60 / 第 41-60 行
````python
        self._device_name: str | None = None
        self._scalars = scalars if scalars is not None else {}
        self._out_dtype = out_dtype
        assert len(input_nodes) > 0, "Expected at least one input node"

    def nodes(self, reorder: Sequence[int] | None = None) -> list[Any]:
        """
        Return the stored input nodes, optionally reordered.

        Args:
            reorder: Optional sequence of indices to reorder the nodes.
                    For example, (2, 0, 1) would return nodes in that order.

        Returns:
            The tuple of input nodes, optionally reordered
        """
        if reorder is None:
            return self._input_nodes
        assert len(self._input_nodes) == len(reorder), (
            f"Reorder length mismatch: {len(self._input_nodes)} vs {len(reorder)}"
````
- **EN**: Introduces function `nodes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `reorder`, and `Returns`.
- **CN**: 这里定义了函数`nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`reorder`、`Returns` 等值。

### Lines 61-80 / 第 61-80 行
````python
        )
        return [self._input_nodes[i] for i in reorder]

    @property
    def count(self) -> int:
        """
        Get the number of input nodes.

        Returns:
            The number of input nodes
        """
        return len(self._input_nodes)

    @property
    def device_type(self) -> str | None:
        """
        Get the device type of the first node.

        Returns:
            The device type (e.g., 'cuda', 'cpu')
````
- **EN**: Introduces function `count`, function `device_type`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`count`、函数`device_type`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python
        """

        return ir.get_device_type(self._input_nodes[0])

    def device(self) -> torch.device:
        """
        Get the device of the first node.

        Returns:
            The device of the first node
        """
        return self._input_nodes[0].get_device()

    def device_name(self) -> str | None:
        """
        Get the device name information.

        Returns:
            A tuple of (gpu_name, vendor, model)
        """
````
- **EN**: Introduces function `device`, function `device_name`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Returns`.
- **CN**: 这里定义了函数`device`、函数`device_name`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Returns` 等值。

### Lines 101-120 / 第 101-120 行
````python
        if self._device_name is None:
            device = self.device()
            if self.device_type == "cuda":
                device_properties = torch.cuda.get_device_properties(device)
                self._device_name = device_properties.gcnArchName
        return self._device_name

    def shapes_symbolic(self) -> tuple[tuple[Any, ...], ...]:
        """
        Get the symbolic shapes of all input nodes.

        Returns:
            A tuple of shape tuples for each input node
        """
        return tuple(node.get_size() for node in self._input_nodes)

    def shapes_hinted(self) -> tuple[tuple[int, ...], ...]:
        """
        Get the size hints for shapes of all input nodes.

````
- **EN**: Introduces function `shapes_symbolic`, function `shapes_hinted`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`shapes_symbolic`、函数`shapes_hinted`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
        Returns:
            A tuple of shape tuples with integer hints for each input node
        """
        return tuple(
            V.graph.sizevars.optimization_hints(node.get_size())
            for node in self._input_nodes
        )

    def strides_symbolic(self) -> tuple[tuple[sympy.Integer, ...], ...]:
        """
        Get the symbolic strides of all input nodes.

        Returns:
            A tuple of stride tuples for each input node
        """
        return tuple(node.get_stride() for node in self._input_nodes)

    def strides_hinted(self) -> tuple[tuple[int, ...], ...]:
        """
        Get the size hints for strides of all input nodes.
````
- **EN**: Introduces function `strides_symbolic`, function `strides_hinted`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`strides_symbolic`、函数`strides_hinted`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python

        Returns:
            A tuple of stride tuples with integer hints for each input node
        """
        return tuple(
            V.graph.sizevars.optimization_hints(node.get_stride())
            for node in self._input_nodes
        )

    def dtypes(self) -> tuple[torch.dtype, ...]:
        """
        Get the dtypes of all input nodes.

        Returns:
            A tuple of dtypes for each input node
        """
        return tuple(node.get_dtype() for node in self._input_nodes)

    def dtype(self, idx: int = 0) -> torch.dtype:
        """
````
- **EN**: Introduces function `dtypes`, function `dtype`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`dtypes`、函数`dtype`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 161-180 / 第 161-180 行
````python
        Get the dtype of a specific input node.

        Args:
            idx: Index of the node to get the dtype from (default: 0)

        Returns:
            The dtype of the specified input node
        """
        return self._input_nodes[idx].get_dtype()

    @abstractmethod
    def out_dtype(self) -> torch.dtype:
        """
        Get the output dtype, whether passed in or inferred from the nodes

        Returns:
            The output dtype
        """

    def get_scalar(self, name: str) -> float | int:
````
- **EN**: Introduces function `out_dtype`, function `get_scalar`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`out_dtype`、函数`get_scalar`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 181-200 / 第 181-200 行
````python
        """
        Get the scalar value for a given name.

        Args:
            name: Name of the scalar to get

        Returns:
            The scalar value
        """
        assert name in self._scalars, f"Scalar {name} not found, but required"
        return self._scalars[name]

    @abstractmethod
    def output_layout(self, flexible: bool = True) -> Layout:
        """
        Abstract method to handle output layout generation.

        Args:
            out_dtype: Optional output dtype. If not provided, infer from inputs
            flexible: If True, return FlexibleLayout, otherwise FixedLayout
````
- **EN**: Introduces function `output_layout`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`output_layout`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
        """


class MMKernelInputs(KernelInputs):
    """
    Specialized KernelInputs for matrix multiplication operations.
    Provides additional methods to access M, N, K dimensions.
    """

    def __init__(
        self,
        input_nodes: list[Any],
        scalars: dict[str, float | int] | None = None,
        out_dtype: torch.dtype | None = None,
        mat1_idx: int = -2,
        mat2_idx: int = -1,
    ):
        """
        Initialize with a tuple of input nodes.

````
- **EN**: Introduces class `MMKernelInputs`, function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `input_nodes`, `scalars`, `out_dtype`, `mat1_idx`, and `mat2_idx`.
- **CN**: 这里定义了类`MMKernelInputs`、函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `input_nodes`、`scalars`、`out_dtype`、`mat1_idx`、`mat2_idx` 等值。

### Lines 221-240 / 第 221-240 行
````python
        By default, we assume the last 2 input nodes are mat1 and mat2, but
        the caller can adjust when necessary
        """
        super().__init__(input_nodes, scalars, out_dtype)
        # for mm, we need at least 2 nodes, and we need to know which nodes
        # are the main matrixes e.g. addmm is (bias, mat1, mat2) whereas others
        # might be (mat1, mat2, scale), etc.
        assert len(self._input_nodes) >= 2, "Expected at least 2 input nodes"

        # Adjust assertions to handle negative indices
        m1_idx, m2_idx = mat1_idx, mat2_idx
        if mat1_idx < 0:
            m1_idx += len(input_nodes)
        if mat2_idx < 0:
            m2_idx += len(input_nodes)

        assert 0 <= m1_idx < len(input_nodes), f"Invalid mat1_idx: {mat1_idx}"
        assert 0 <= m2_idx < len(input_nodes), f"Invalid mat2_idx: {mat2_idx}"

        self._mat1_idx = mat1_idx
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `MMKernelInputs.__init__`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`MMKernelInputs.__init__` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
        self._mat2_idx = mat2_idx

    def mnk_symbolic(
        self,
    ) -> tuple[sympy.Integer, sympy.Integer, sympy.Integer]:
        """
        Get the symbolic M, N, K dimensions for matrix multiplication.
        Handles both 2D (MM) and 3D (BMM) tensors.

        M is extracted from the second-to-last dimension of the first operand (mat1).
        N is extracted from the last dimension of the second operand (mat2).
        K is extracted from the last dimension of the first operand (mat1).

        Returns:
            A tuple of (M, N, K) dimensions
        """
        mat1 = self.nodes()[self._mat1_idx]
        mat2 = self.nodes()[self._mat2_idx]

        m = mat1.get_size()[-2]  # M from second-to-last dimension of mat1
````
- **EN**: Introduces function `mnk_symbolic`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `Returns`, `mat1`, `mat2`, and `m`.
- **CN**: 这里定义了函数`mnk_symbolic`。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `Returns`、`mat1`、`mat2`、`m` 等值。

### Lines 261-280 / 第 261-280 行
````python
        k = mat1.get_size()[-1]  # K from last dimension of mat1
        n = mat2.get_size()[-1]  # N from last dimension of mat2

        # Ensure K dimensions match between operands
        k0 = mat2.get_size()[-2]  # K from second-to-last dimension of mat2
        V.graph.sizevars.check_equals(k, k0)
        return (m, n, k)

    def out_dtype(self) -> torch.dtype:
        """
        Get the output dtype, whether passed in or inferred from the nodes

        Returns:
            The output dtype
        """
        if self._out_dtype is not None:
            return self._out_dtype
        return self.mat1mat2()[0].get_dtype()

    def output_layout(self, flexible: bool = True) -> Layout:
````
- **EN**: Introduces function `out_dtype`, function `output_layout`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`out_dtype`、函数`output_layout`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 281-300 / 第 281-300 行
````python
        """
        Handle output layout generation for matrix multiplication.

        Args:
            out_dtype: Optional output dtype. If not provided, infer from inputs
            flexible: If True, return FlexibleLayout, otherwise FixedLayout
        """
        mat1, mat2 = self.mat1mat2()
        out_dtype = self.out_dtype()
        # NOTE: taken from mm_common.mm_args
        *b1, m, k1 = mat1.get_size()
        *b2, k2, n = mat2.get_size()
        b = [V.graph.sizevars.check_equals_and_simplify(a, b) for a, b in zip(b1, b2)]
        size = [*b, m, n]
        if flexible:
            return FlexibleLayout(self.device(), out_dtype, size)
        else:
            return FixedLayout(self.device(), out_dtype, size)

    def mat1mat2(self) -> tuple[Any, Any]:
````
- **EN**: Introduces function `mat1mat2`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`mat1mat2`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 301-320 / 第 301-320 行
````python
        """
        Get the mat1 and mat2 nodes.

        Returns:
            A tuple of (mat1, mat2) nodes
        """
        nodes = self.nodes()
        return nodes[self._mat1_idx], nodes[self._mat2_idx]

    def mnk_hinted(self) -> tuple[int, int, int]:
        """
        Get the hinted M, N, K dimensions for matrix multiplication.
        Handles both 2D (MM) and 3D (BMM) tensors.

        Uses shapes_hinted from the base class to get integer hints for dimensions.

        Returns:
            A tuple of (M, N, K) dimensions as integers
        """
        hinted_shapes = self.shapes_hinted()
````
- **EN**: Introduces function `mnk_hinted`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Returns`, `nodes`, and `hinted_shapes`.
- **CN**: 这里定义了函数`mnk_hinted`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Returns`、`nodes`、`hinted_shapes` 等值。

### Lines 321-340 / 第 321-340 行
````python
        mat1_shape = hinted_shapes[self._mat1_idx]
        mat2_shape = hinted_shapes[self._mat2_idx]

        m = mat1_shape[-2]  # M from second-to-last dimension of mat1
        k = mat1_shape[-1]  # K from last dimension of mat1
        n = mat2_shape[-1]  # N from last dimension of mat2

        # Ensure K dimensions match between operands
        k_check = mat2_shape[-2]  # K from second-to-last dimension of mat2
        assert k == k_check, f"K dimensions don't match: {k} vs {k_check}"

        return (m, n, k)

    def batch_hinted(self) -> int:
        """
        Get the hinted batch size for batched matrix multiplication.
        Returns 1 for non-batched (2D) operations.

        Returns:
            The batch size as an integer
````
- **EN**: Introduces function `batch_hinted`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `mat1_shape`, `mat2_shape`, `m`, `k`, `n`, `k_check`, and `...+1`.
- **CN**: 这里定义了函数`batch_hinted`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `mat1_shape`、`mat2_shape`、`m`、`k`、`n`、`k_check`、`另有1项` 等值。

### Lines 341-347 / 第 341-347 行
````python
        """
        hinted_shapes = self.shapes_hinted()
        mat1_shape = hinted_shapes[self._mat1_idx]

        if len(mat1_shape) >= 3:
            return mat1_shape[-3]  # Batch from third-to-last dimension
        return 1  # Non-batched operation
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `hinted_shapes`, and `mat1_shape`. This range continues the implementation of function `MMKernelInputs.batch_hinted`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `hinted_shapes`、`mat1_shape` 等值。这一段延续了函数`MMKernelInputs.batch_hinted` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `KernelInputs`, and `MMKernelInputs`  
  **CN**: 主要类：`KernelInputs`、`MMKernelInputs`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `abc`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.config`, `torch._inductor`, `torch._inductor.virtualized`, `.ir`
