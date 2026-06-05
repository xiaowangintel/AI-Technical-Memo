# adaptive.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/modules/adaptive.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Defines reusable nn.Module classes, their state, and their forward-path helpers.
- **Purpose (CN)**: 定义可复用的 nn.Module 类、其状态以及前向路径辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```python
# mypy: allow-untyped-defs

import itertools
from collections import namedtuple
from collections.abc import Sequence

import torch
import torch.nn.functional as F
from torch import Tensor

from .container import ModuleList, Sequential
from .linear import Linear
from .module import Module


__all__ = ["AdaptiveLogSoftmaxWithLoss"]

_ASMoutput = namedtuple("_ASMoutput", ["output", "loss"])


class AdaptiveLogSoftmaxWithLoss(Module):
    (
        """Efficient softmax approximation.
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 25-49
```python
    As described in
    `Efficient softmax approximation for GPUs by Edouard Grave, Armand Joulin,
    Moustapha Ciss\u00e9, David Grangier, and Herv\u00e9 J\u00e9gou
    <https://arxiv.org/abs/1609.04309>`__.
"""
        r"""
    Adaptive softmax is an approximate strategy for training models with large
    output spaces. It is most effective when the label distribution is highly
    imbalanced, for example in natural language modelling, where the word
    frequency distribution approximately follows the `Zipf's law`_.

    Adaptive softmax partitions the labels into several clusters, according to
    their frequency. These clusters may contain different number of targets
    each.
    Additionally, clusters containing less frequent labels assign lower
    dimensional embeddings to those labels, which speeds up the computation.
    For each minibatch, only clusters for which at least one target is
    present are evaluated.

    The idea is that the clusters which are accessed frequently
    (like the first one, containing most frequent labels), should also be cheap
    to compute -- that is, contain a small number of assigned labels.

    We highly recommend taking a look at the original paper for more details.
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss` and works to hand work to a compiler/backend pipeline.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss`，用于将工作移交给编译器或后端流水线。

### Lines 50-71
```python
    * :attr:`cutoffs` should be an ordered Sequence of integers sorted
      in the increasing order.
      It controls number of clusters and the partitioning of targets into
      clusters. For example setting ``cutoffs = [10, 100, 1000]``
      means that first `10` targets will be assigned
      to the 'head' of the adaptive softmax, targets `11, 12, ..., 100` will be
      assigned to the first cluster, and targets `101, 102, ..., 1000` will be
      assigned to the second cluster, while targets
      `1001, 1002, ..., n_classes - 1` will be assigned
      to the last, third cluster.

    * :attr:`div_value` is used to compute the size of each additional cluster,
      which is given as
      :math:`\left\lfloor\frac{\texttt{in\_features}}{\texttt{div\_value}^{idx}}\right\rfloor`,
      where :math:`idx` is the cluster index (with clusters
      for less frequent words having larger indices,
      and indices starting from :math:`1`).

    * :attr:`head_bias` if set to True, adds a bias term to the 'head' of the
      adaptive softmax. See paper for details. Set to False in the official
      implementation.
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss` and works to prepare neural-network operators or module behavior. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss`，用于准备神经网络算子或模块行为。 同时它还会遍历输入、节点或簿记结构。

### Lines 72-94
```python
    .. warning::
        Labels passed as inputs to this module should be sorted according to
        their frequency. This means that the most frequent label should be
        represented by the index `0`, and the least frequent
        label should be represented by the index `n_classes - 1`.

    .. note::
        This module returns a ``NamedTuple`` with ``output``
        and ``loss`` fields. See further documentation for details.

    .. note::
        To compute log-probabilities for all classes, the ``log_prob``
        method can be used.

    Args:
        in_features (int): Number of features in the input tensor
        n_classes (int): Number of classes in the dataset
        cutoffs (Sequence): Cutoffs used to assign targets to their buckets
        div_value (float, optional): value used as an exponent to compute sizes
            of the clusters. Default: 4.0
        head_bias (bool, optional): If ``True``, adds a bias term to the 'head' of the
            adaptive softmax. Default: ``False``
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss` and works to organize reusable module behavior and state.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss`，用于组织可复用的模块行为与状态。

### Lines 95-119
```python
    Returns:
        ``NamedTuple`` with ``output`` and ``loss`` fields:
            * **output** is a Tensor of size ``N`` containing computed target
              log probabilities for each example
            * **loss** is a Scalar representing the computed negative
              log likelihood loss

    Shape:
        - input: :math:`(N, \texttt{in\_features})` or :math:`(\texttt{in\_features})`
        - target: :math:`(N)` or :math:`()` where each value satisfies :math:`0 <= \texttt{target[i]} <= \texttt{n\_classes}`
        - output1: :math:`(N)` or :math:`()`
        - output2: ``Scalar``

    .. _Zipf's law: https://en.wikipedia.org/wiki/Zipf%27s_law
    """
    )

    in_features: int
    n_classes: int
    cutoffs: list[int]
    div_value: float
    head_bias: bool
    head: Linear
    tail: ModuleList
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 120-137
```python
    def __init__(
        self,
        in_features: int,
        n_classes: int,
        cutoffs: Sequence[int],
        div_value: float = 4.0,
        head_bias: bool = False,
        device=None,
        dtype=None,
    ) -> None:
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()

        cutoffs = list(cutoffs)

        if len(cutoffs) == 0:
            raise ValueError("cutoffs should be a sequence of length larger than 0")
```
- **EN**: Defines the `AdaptiveLogSoftmaxWithLoss.__init__` method; this block introduces logic that initialize learnable tensors and related state.
- **CN**: 定义`AdaptiveLogSoftmaxWithLoss.__init__` 方法；该代码块引入了用于初始化可学习张量及相关状态的逻辑。

### Lines 138-160
```python
        if (
            (cutoffs != sorted(cutoffs))
            or (min(cutoffs) <= 0)
            or (max(cutoffs) > (n_classes - 1))
            or (len(set(cutoffs)) != len(cutoffs))
            or any(int(c) != c for c in cutoffs)
        ):
            raise ValueError(
                "cutoffs should be a sequence of unique, positive "
                "integers sorted in an increasing order, where "
                "each value is between 1 and n_classes-1"
            )

        self.in_features = in_features
        self.n_classes = n_classes
        self.cutoffs = cutoffs + [n_classes]
        self.div_value = div_value
        self.head_bias = head_bias

        self.shortlist_size = self.cutoffs[0]
        self.n_clusters = len(self.cutoffs) - 1
        self.head_size = self.shortlist_size + self.n_clusters
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss.__init__` and works to validate invariants and surface meaningful failures. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss.__init__`，用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 161-185
```python
        self.head = Linear(
            self.in_features, self.head_size, bias=self.head_bias, **factory_kwargs
        )
        self.tail = ModuleList()

        for i in range(self.n_clusters):
            hsz = int(self.in_features // (self.div_value ** (i + 1)))
            osz = self.cutoffs[i + 1] - self.cutoffs[i]

            projection = Sequential(
                Linear(self.in_features, hsz, bias=False, **factory_kwargs),
                Linear(hsz, osz, bias=False, **factory_kwargs),
            )

            self.tail.append(projection)

    def reset_parameters(self) -> None:
        """
        Resets parameters based on their initialization used in ``__init__``.
        """
        self.head.reset_parameters()
        for i2h, h2o in self.tail:  # type: ignore[misc]
            i2h.reset_parameters()  # type: ignore[has-type]
            h2o.reset_parameters()  # type: ignore[has-type]
```
- **EN**: Declares `AdaptiveLogSoftmaxWithLoss(Module)`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `AdaptiveLogSoftmaxWithLoss(Module)`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 186-203
```python
    def forward(self, input_: Tensor, target_: Tensor) -> _ASMoutput:
        """
        Runs the forward pass.
        """
        targ_dim = target_.dim()

        if targ_dim == 1:
            if input_.size(0) != target_.size(0):
                raise RuntimeError(
                    "Input and target should have the same size in the batch dimension."
                )
            if input_.dim() != 2:
                raise RuntimeError(
                    "1D target tensor expects 2D input tensors, "
                    "but found inputs with size",
                    input_.size(),
                )
        elif targ_dim == 0:
```
- **EN**: Defines the `AdaptiveLogSoftmaxWithLoss.forward` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`AdaptiveLogSoftmaxWithLoss.forward` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

### Lines 204-229
```python
            if input_.dim() != 1:
                raise RuntimeError(
                    "0D target tensor expects 1D input tensors, "
                    "but found inputs with size",
                    input_.size(),
                )
        else:
            raise RuntimeError(
                "0D or 1D target tensor expected, multi-target not supported"
            )

        is_batched = targ_dim > 0
        input = input_ if is_batched else input_.unsqueeze(0)
        target = target_ if is_batched else target_.unsqueeze(0)

        used_rows = 0
        batch_size = target.size(0)

        output = input.new_zeros(batch_size)
        gather_inds = target.new_empty(batch_size)

        cutoff_values = [0] + self.cutoffs
        for i in range(len(cutoff_values) - 1):
            low_idx = cutoff_values[i]
            high_idx = cutoff_values[i + 1]
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss.forward` and works to validate invariants and surface meaningful failures. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss.forward`，用于校验不变量并给出有意义的失败信息。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 230-252
```python
            target_mask = (target >= low_idx) & (target < high_idx)
            row_indices = target_mask.nonzero().squeeze()

            if row_indices.numel() == 0:
                continue

            if i == 0:
                gather_inds.index_copy_(0, row_indices, target[target_mask])

            else:
                relative_target = target[target_mask] - low_idx
                input_subset = input.index_select(0, row_indices)

                cluster_output = self.tail[i - 1](input_subset)
                cluster_index = self.shortlist_size + i - 1

                gather_inds.index_fill_(0, row_indices, cluster_index)
                cluster_logprob = F.log_softmax(cluster_output, dim=1)
                local_logprob = cluster_logprob.gather(1, relative_target.unsqueeze(1))
                output.index_copy_(0, row_indices, local_logprob.squeeze(1))

            used_rows += row_indices.numel()
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss.forward` and works to prepare neural-network operators or module behavior. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss.forward`，用于准备神经网络算子或模块行为。 同时它还会根据运行时条件分支处理。

### Lines 253-276
```python
        if used_rows != batch_size:
            raise RuntimeError(
                f"Target values should be in [0, {self.n_classes - 1}], "
                f"but values in range [{target.min().item()}, {target.max().item()}] "
                "were found. "
            )

        head_output = self.head(input)
        head_logprob = F.log_softmax(head_output, dim=1)
        output += head_logprob.gather(1, gather_inds.unsqueeze(1)).squeeze()
        loss = (-output).mean()

        if not is_batched:
            output = output.squeeze(0)

        return _ASMoutput(output, loss)

    def _get_full_log_prob(self, input, head_output):
        """Given input tensor, and output of ``self.head``, compute the log of the full distribution."""
        out = input.new_empty((head_output.size(0), self.n_classes))
        head_logprob = F.log_softmax(head_output, dim=1)

        out[:, : self.shortlist_size] = head_logprob[:, : self.shortlist_size]
```
- **EN**: Declares `AdaptiveLogSoftmaxWithLoss(Module)`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `AdaptiveLogSoftmaxWithLoss(Module)`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 277-302
```python
        for i, (start_idx, stop_idx) in enumerate(itertools.pairwise(self.cutoffs)):
            cluster_output = self.tail[i](input)
            cluster_logprob = F.log_softmax(cluster_output, dim=1)
            output_logprob = cluster_logprob + head_logprob[
                :, self.shortlist_size + i
            ].unsqueeze(1)

            out[:, start_idx:stop_idx] = output_logprob

        return out

    def log_prob(self, input: Tensor) -> Tensor:
        r"""Compute log probabilities for all :math:`\texttt{n\_classes}`.

        Args:
            input (Tensor): a minibatch of examples

        Returns:
            log-probabilities of for each class :math:`c`
            in range :math:`0 <= c <= \texttt{n\_classes}`, where :math:`\texttt{n\_classes}` is a
            parameter passed to ``AdaptiveLogSoftmaxWithLoss`` constructor.

        Shape:
            - Input: :math:`(N, \texttt{in\_features})`
            - Output: :math:`(N, \texttt{n\_classes})`
```
- **EN**: Declares `AdaptiveLogSoftmaxWithLoss(Module)`; this class packages state and methods that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 声明 `AdaptiveLogSoftmaxWithLoss(Module)`；该类封装了状态与方法，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 303-329
```python
        """
        head_output = self.head(input)
        return self._get_full_log_prob(input, head_output)

    def predict(self, input: Tensor) -> Tensor:
        r"""Return the class with the highest probability for each example in the input minibatch.

        This is equivalent to ``self.log_prob(input).argmax(dim=1)``, but is more efficient in some cases.

        Args:
            input (Tensor): a minibatch of examples

        Returns:
            output (Tensor): a class with the highest probability for each example

        Shape:
            - Input: :math:`(N, \texttt{in\_features})`
            - Output: :math:`(N)`
        """
        head_output = self.head(input)
        output = torch.argmax(head_output, dim=1)
        not_in_shortlist = output >= self.shortlist_size
        all_in_shortlist = not (not_in_shortlist.any())

        if all_in_shortlist:
            return output
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 330-339
```python
        elif not_in_shortlist.all():
            log_prob = self._get_full_log_prob(input, head_output)
            return torch.argmax(log_prob, dim=1)

        else:
            log_prob = self._get_full_log_prob(
                input[not_in_shortlist], head_output[not_in_shortlist]
            )
            output[not_in_shortlist] = torch.argmax(log_prob, dim=1)
            return output
```
- **EN**: This block continues `AdaptiveLogSoftmaxWithLoss.predict` and works to prepare neural-network operators or module behavior. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `AdaptiveLogSoftmaxWithLoss.predict`，用于准备神经网络算子或模块行为。 同时它还会计算并返回中间值或结果。

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
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.functional`, `.container`, `.linear`, `.module`
- **Standard library / 标准库**: `itertools`, `collections`, `collections.abc`
- **Primary symbols / 核心符号**: `__all__`, `AdaptiveLogSoftmaxWithLoss`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
