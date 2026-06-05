# flatten.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/flatten.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```python
# mypy: allow-untyped-defs

from torch import Tensor
from torch.types import _size

from .module import Module


__all__ = ["Flatten", "Unflatten"]


class Flatten(Module):
    r"""
    Flattens a contiguous range of dims into a tensor.

    For use with :class:`~nn.Sequential`, see :meth:`torch.flatten` for details.
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 18-29
```python
    Shape:
        - Input: :math:`(*, S_{\text{start}},..., S_{i}, ..., S_{\text{end}}, *)`,'
          where :math:`S_{i}` is the size at dimension :math:`i` and :math:`*` means any
          number of dimensions including none.
        - Output: :math:`(*, \prod_{i=\text{start}}^{\text{end}} S_{i}, *)`.

    Args:
        start_dim: first dim to flatten (default = 1).
        end_dim: last dim to flatten (default = -1).

    Examples::
        >>> input = torch.randn(32, 1, 5, 5)
```
- **EN**: This block continues `Flatten` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `Flatten`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 30-45
```python
        >>> # With default parameters
        >>> m = nn.Flatten()
        >>> output = m(input)
        >>> output.size()
        torch.Size([32, 25])
        >>> # With non-default parameters
        >>> m = nn.Flatten(0, 2)
        >>> output = m(input)
        >>> output.size()
        torch.Size([160, 5])
    """

    __constants__ = ["start_dim", "end_dim"]
    start_dim: int
    end_dim: int
```
- **EN**: This block continues `Flatten` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `Flatten`，用于组织可复用的模块行为与状态。

### Lines 46-63
```python
    def __init__(self, start_dim: int = 1, end_dim: int = -1) -> None:
        super().__init__()
        self.start_dim = start_dim
        self.end_dim = end_dim

    def forward(self, input: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return input.flatten(self.start_dim, self.end_dim)

    def extra_repr(self) -> str:
        """
        Returns the extra representation of the module.
        """
        return f"start_dim={self.start_dim}, end_dim={self.end_dim}"
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 64-80
```python
class Unflatten(Module):
    r"""
    Unflattens a tensor dim expanding it to a desired shape. For use with :class:`~nn.Sequential`.

    * :attr:`dim` specifies the dimension of the input tensor to be unflattened, and it can
      be either `int` or `str` when `Tensor` or `NamedTensor` is used, respectively.

    * :attr:`unflattened_size` is the new shape of the unflattened dimension of the tensor and it can be
      a `tuple` of ints or a `list` of ints or `torch.Size` for `Tensor` input;  a `NamedShape`
      (tuple of `(name, size)` tuples) for `NamedTensor` input.

    Shape:
        - Input: :math:`(*, S_{\text{dim}}, *)`, where :math:`S_{\text{dim}}` is the size at
          dimension :attr:`dim` and :math:`*` means any number of dimensions including none.
        - Output: :math:`(*, U_1, ..., U_n, *)`, where :math:`U` = :attr:`unflattened_size` and
          :math:`\prod_{i=1}^n U_i = S_{\text{dim}}`.
```
- **EN**: Declares `Unflatten(Module)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `Unflatten(Module)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 81-92
```python
    Args:
        dim (Union[int, str]): Dimension to be unflattened
        unflattened_size (Union[torch.Size, Tuple, List, NamedShape]): New shape of the unflattened dimension

    Examples:
        >>> input = torch.randn(2, 50)
        >>> # With tuple of ints
        >>> m = nn.Sequential(
        >>>     nn.Linear(50, 50),
        >>>     nn.Unflatten(1, (2, 5, 5))
        >>> )
        >>> output = m(input)
```
- **EN**: This block continues `Unflatten` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `Unflatten`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 93-110
```python
        >>> output.size()
        torch.Size([2, 2, 5, 5])
        >>> # With torch.Size
        >>> m = nn.Sequential(
        >>>     nn.Linear(50, 50),
        >>>     nn.Unflatten(1, torch.Size([2, 5, 5]))
        >>> )
        >>> output = m(input)
        >>> output.size()
        torch.Size([2, 2, 5, 5])
        >>> # With namedshape (tuple of tuples)
        >>> input = torch.randn(2, 50, names=("N", "features"))
        >>> unflatten = nn.Unflatten("features", (("C", 2), ("H", 5), ("W", 5)))
        >>> output = unflatten(input)
        >>> output.size()
        torch.Size([2, 2, 5, 5])
    """
```
- **EN**: This block continues `Unflatten` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `Unflatten`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 111-126
```python
    NamedShape = tuple[tuple[str, int]]

    __constants__ = ["dim", "unflattened_size"]
    dim: int | str
    unflattened_size: _size | NamedShape

    def __init__(self, dim: int | str, unflattened_size: _size | NamedShape) -> None:
        super().__init__()

        if isinstance(dim, int):
            self._require_tuple_int(unflattened_size)
        elif isinstance(dim, str):
            self._require_tuple_tuple(unflattened_size)
        else:
            raise TypeError("invalid argument type for dim parameter")
```
- **EN**: Declares `Unflatten(Module)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `Unflatten(Module)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 127-143
```python
        self.dim = dim
        self.unflattened_size = unflattened_size

    def _require_tuple_tuple(self, input) -> None:
        if isinstance(input, tuple):
            for idx, elem in enumerate(input):
                if not isinstance(elem, tuple):
                    raise TypeError(
                        "unflattened_size must be tuple of tuples, "
                        + f"but found element of type {type(elem).__name__} at pos {idx}"
                    )
            return
        raise TypeError(
            "unflattened_size must be a tuple of tuples, "
            + f"but found type {type(input).__name__}"
        )
```
- **EN**: Declares `Unflatten(Module)`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `Unflatten(Module)`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 144-156
```python
    def _require_tuple_int(self, input) -> None:
        if isinstance(input, (tuple, list)):
            for idx, elem in enumerate(input):
                if not isinstance(elem, int):
                    raise TypeError(
                        "unflattened_size must be tuple of ints, "
                        + f"but found element of type {type(elem).__name__} at pos {idx}"
                    )
            return
        raise TypeError(
            f"unflattened_size must be a tuple of ints, but found type {type(input).__name__}"
        )
```
- **EN**: Declares `Unflatten(Module)`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `Unflatten(Module)`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 157-167
```python
    def forward(self, input: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return input.unflatten(self.dim, self.unflattened_size)

    def extra_repr(self) -> str:
        """
        Returns the extra representation of the module.
        """
        return f"dim={self.dim}, unflattened_size={self.unflattened_size}"
```
- **EN**: Declares `Unflatten(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `Unflatten(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.types`, `.module`
- **Primary symbols / 核心符号**: `__all__`, `Flatten`, `Unflatten`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
