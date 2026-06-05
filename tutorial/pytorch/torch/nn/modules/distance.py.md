# distance.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/distance.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
import torch.nn.functional as F
from torch import Tensor

from .module import Module


__all__ = ["PairwiseDistance", "CosineSimilarity"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 10-21
```python
class PairwiseDistance(Module):
    r"""
    Computes the pairwise distance between input vectors, or between columns of input matrices.

    Distances are computed using ``p``-norm, with constant ``eps`` added to avoid division by zero
    if ``p`` is negative, i.e.:

    .. math ::
        \mathrm{dist}\left(x, y\right) = \left\Vert x-y + \epsilon e \right\Vert_p,

    where :math:`e` is the vector of ones and the ``p``-norm is given by.
```
- **EN**: Declares `PairwiseDistance(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `PairwiseDistance(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 22-29
```python
    .. math ::
        \Vert x \Vert _p = \left( \sum_{i=1}^n  \vert x_i \vert ^ p \right) ^ {1/p}.

    Args:
        p (real, optional): the norm degree. Can be negative. Default: 2
        eps (float, optional): Small value to avoid division by zero.
            Default: 1e-6
        keepdim (bool, optional): Determines whether or not to keep the vector dimension.
```
- **EN**: This block continues `PairwiseDistance` and works to prepare neural-network operators or module behavior.
- **CN**: 该代码块继续实现 `PairwiseDistance`，用于准备神经网络算子或模块行为。

### Lines 30-37
```python
            Default: False
    Shape:
        - Input1: :math:`(N, D)` or :math:`(D)` where `N = batch dimension` and `D = vector dimension`
        - Input2: :math:`(N, D)` or :math:`(D)`, same shape as the Input1
        - Output: :math:`(N)` or :math:`()` based on input dimension.
          If :attr:`keepdim` is ``True``, then :math:`(N, 1)` or :math:`(1)` based on input dimension.

    Examples:
```
- **EN**: This block continues `PairwiseDistance` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `PairwiseDistance`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 38-48
```python
        >>> pdist = nn.PairwiseDistance(p=2)
        >>> input1 = torch.randn(100, 128)
        >>> input2 = torch.randn(100, 128)
        >>> output = pdist(input1, input2)
    """

    __constants__ = ["norm", "eps", "keepdim"]
    norm: float
    eps: float
    keepdim: bool
```
- **EN**: This block continues `PairwiseDistance` and works to prepare neural-network operators or module behavior.
- **CN**: 该代码块继续实现 `PairwiseDistance`，用于准备神经网络算子或模块行为。

### Lines 49-56
```python
    def __init__(
        self, p: float = 2.0, eps: float = 1e-6, keepdim: bool = False
    ) -> None:
        super().__init__()
        self.norm = p
        self.eps = eps
        self.keepdim = keepdim
```
- **EN**: Declares `PairwiseDistance(Module)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `PairwiseDistance(Module)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 57-66
```python
    def forward(self, x1: Tensor, x2: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return F.pairwise_distance(x1, x2, self.norm, self.eps, self.keepdim)


class CosineSimilarity(Module):
    r"""Returns cosine similarity between :math:`x_1` and :math:`x_2`, computed along `dim`.
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 67-74
```python
    .. math ::
        \text{similarity} = \dfrac{x_1 \cdot x_2}{\max(\Vert x_1 \Vert _2 \cdot \Vert x_2 \Vert _2, \epsilon)}.

    Args:
        dim (int, optional): Dimension where cosine similarity is computed. Default: 1
        eps (float, optional): Small value to avoid division by zero.
            Default: 1e-8
    Shape:
```
- **EN**: This block continues `CosineSimilarity` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `CosineSimilarity`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 75-86
```python
        - Input1: :math:`(\ast_1, D, \ast_2)` where D is at position `dim`
        - Input2: :math:`(\ast_1, D, \ast_2)`, same number of dimensions as x1, matching x1 size at dimension `dim`,
          and broadcastable with x1 at other dimensions.
        - Output: :math:`(\ast_1, \ast_2)`

    Examples:
        >>> input1 = torch.randn(100, 128)
        >>> input2 = torch.randn(100, 128)
        >>> cos = nn.CosineSimilarity(dim=1, eps=1e-6)
        >>> output = cos(input1, input2)
    """
```
- **EN**: This block continues `CosineSimilarity` and works to prepare neural-network operators or module behavior.
- **CN**: 该代码块继续实现 `CosineSimilarity`，用于准备神经网络算子或模块行为。

### Lines 87-95
```python
    __constants__ = ["dim", "eps"]
    dim: int
    eps: float

    def __init__(self, dim: int = 1, eps: float = 1e-8) -> None:
        super().__init__()
        self.dim = dim
        self.eps = eps
```
- **EN**: Declares `CosineSimilarity(Module)`; this class packages state and methods that initialize learnable tensors and related state.
- **CN**: 声明 `CosineSimilarity(Module)`；该类封装了状态与方法，用于初始化可学习张量及相关状态。

### Lines 96-100
```python
    def forward(self, x1: Tensor, x2: Tensor) -> Tensor:
        """
        Runs the forward pass.
        """
        return F.cosine_similarity(x1, x2, self.dim, self.eps)
```
- **EN**: Defines the `CosineSimilarity.forward` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`CosineSimilarity.forward` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.nn.functional`, `torch`, `.module`
- **Primary symbols / 核心符号**: `__all__`, `PairwiseDistance`, `CosineSimilarity`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
