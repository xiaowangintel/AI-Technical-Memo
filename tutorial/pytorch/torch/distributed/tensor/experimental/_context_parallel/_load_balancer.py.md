# _load_balancer.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/experimental/_context_parallel/_load_balancer.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include _LoadBalancer, _HeadTailLoadBalancer, _create_default_load_balancer.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 _LoadBalancer, _HeadTailLoadBalancer, _create_default_load_balancer。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# this file contains the `_LoadBalancer` class and its family of implementation
# for different load-balancing strategies in tensor sharding.
import functools
from abc import ABC, abstractmethod

import torch
from torch import Tensor
from torch.nn.attention.flex_attention import BlockMask


# make it private since it's still a prototype
class _LoadBalancer(ABC):
    @abstractmethod
    def _generate_indices(self, restore: bool = False) -> Tensor | None:
        """
        Generate indices for load balancing.
        Args:
            restore (bool):

        Returns:
````

- **L1** EN: Keeps the inline comment or directive: this file contains the `_LoadBalancer` class and its family of implementation | CN: 保留这一行注释或指令：this file contains the `_LoadBalancer` class and its family of implementation
- **L2** EN: Keeps the inline comment or directive: for different load-balancing strategies in tensor sharding. | CN: 保留这一行注释或指令：for different load-balancing strategies in tensor sharding.
- **L3** EN: Imports module dependencies: `functools`. | CN: 导入模块依赖：`functools`。
- **L4** EN: Imports selected names from `abc`. | CN: 从 `abc` 导入指定名称。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L7** EN: Imports selected names from `torch`. | CN: 从 `torch` 导入指定名称。
- **L8** EN: Imports selected names from `torch.nn.attention.flex_attention`. | CN: 从 `torch.nn.attention.flex_attention` 导入指定名称。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L11** EN: Keeps the inline comment or directive: make it private since it's still a prototype | CN: 保留这一行注释或指令：make it private since it's still a prototype
- **L12** EN: Defines class `_LoadBalancer`. | CN: 定义类 `_LoadBalancer`。
- **L13** EN: Applies decorator `abstractmethod` to the following definition. | CN: 将装饰器 `abstractmethod` 应用于后续定义。
- **L14** EN: Defines function `_generate_indices`. | CN: 定义函数 `_generate_indices`。
- **L15** EN: Starts the docstring for the function _generate_indices. | CN: 开始定义 function _generate_indices 的文档字符串。
- **L16** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L17** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L18** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L19** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L20** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 21-40 / 第 21-40 行

````python
            The generated indices of shape `(1, seq_len)` if the load-balancing is
            identical within the batch, or `(batch_size, seq_len)` if the load-balancing
            should vary within the batch.

        Warning:
            For Multi-Head Attention, we require the masks over the head dimension are identical
            (i.e. the return value of `_generate_indices()` does not have `heads` dimension).

        Example:
            Here is the causal mask for attention where q_len == kv_len == 8:
                            KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 0, 0, 0, 0, 0]
            Q_index [1, 1, 1, 1, 0, 0, 0, 0]
                    [1, 1, 1, 1, 1, 0, 0, 0]
                    [1, 1, 1, 1, 1, 1, 0, 0]
                    [1, 1, 1, 1, 1, 1, 1, 0]
                    [1, 1, 1, 1, 1, 1, 1, 1]

````

- **L21** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L22** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L23** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L24** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L25** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L29** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L30** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L31** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L32** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L33** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L34** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L35** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L36** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L37** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L38** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L40** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
            This mask matrix also represents the computation required to compute
            the masked Q @ K^T by:
            - mask[i, j] == 1: the computation of Q[i, :] dot K[j, :] is required
            - mask[i, j] == 0: the computation should be skipped

            Therefore the number of 1s in matrix represents the amount of computation
            required.

            Assume we want to distribute this Q @ K^T computation to 2 devices, then
            the matrix is also distributed as:
                            KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 0, 0, 0, 0, 0]    rank 0
                    [1, 1, 1, 1, 0, 0, 0, 0]
            Q_index ------------------------
                    [1, 1, 1, 1, 1, 0, 0, 0]
                    [1, 1, 1, 1, 1, 1, 0, 0]    rank 1
                    [1, 1, 1, 1, 1, 1, 1, 0]
                    [1, 1, 1, 1, 1, 1, 1, 1]
````

- **L41** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L42** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L43** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L44** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L45** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L46** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L47** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L48** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L49** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L50** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L51** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L52** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L55** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L56** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L57** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L58** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L59** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L60** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python

            An imbalance of computation is observed on these 2 ranks and this could make
            rank 1 the straggler when performing Context Parallel. In order to balance
            the computation, we need to rearrange the QKV tensors before sharding in such a
            way that the result mask matrix is evenly distributed over devices and each
            rank has the number of 1s as close as possible.

            This method defines the strategy of how to rearrange the QKV tensor for better
            load-balance:
            - when `restore == False`, this method returns an indices tensor `rearrange_idx`
            such that Q[rearrange_idx] is the desired Q tensor after rearranging.
            - when `restore == True`, this method returns an indices tensor `restore_idx`
            such that Q[rearrange_idx][restore_idx] == Q, i.e. restoring the rearranged tensor
            back to the original status before rearranging.
        """


class _HeadTailLoadBalancer(_LoadBalancer):
    def __init__(self, seq_length: int, world_size: int, device: str | torch.device):
        self.seq_length = seq_length
````

- **L61** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L62** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L63** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L64** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L65** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L66** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L67** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L68** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L69** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L70** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L71** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L72** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L73** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L74** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L75** EN: Closes the docstring for the function _generate_indices. | CN: 结束 function _generate_indices 的文档字符串。
- **L76** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Defines class `_HeadTailLoadBalancer`. | CN: 定义类 `_HeadTailLoadBalancer`。
- **L79** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L80** EN: Assigns or updates `self.seq_length`. | CN: 对 `self.seq_length` 进行赋值或更新。

### Lines 81-100 / 第 81-100 行

````python
        self.world_size = world_size
        self.device = device

    def _generate_indices(self, restore: bool = False) -> Tensor:
        """
        Generate head-and-tail load balancing indices or restore indices.
        Args:
            restore:
                If True, generate restore indices that map head-and-tail rearranged
                positions back to original positions. If False, generate load
                balance indices that rearrange original positions to head-and-tail pattern.

        Returns:
            The generated indices of shape `(1, seq_len)` because the load-balancing is
            identical within the batch.

        Warning:
            For Multi-Head Attention, we require the masks over the head dimension are identical
            (i.e. the return value of `_generate_indices()` does not have `heads` dimension).

````

- **L81** EN: Assigns or updates `self.world_size`. | CN: 对 `self.world_size` 进行赋值或更新。
- **L82** EN: Assigns or updates `self.device`. | CN: 对 `self.device` 进行赋值或更新。
- **L83** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L84** EN: Defines function `_generate_indices`. | CN: 定义函数 `_generate_indices`。
- **L85** EN: Starts the docstring for the function _generate_indices. | CN: 开始定义 function _generate_indices 的文档字符串。
- **L86** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L87** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L88** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L89** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L90** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L91** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L92** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L93** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L94** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L95** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L96** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L97** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L98** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L99** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L100** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 101-120 / 第 101-120 行

````python
        Example:
            Here is the causal mask for attention where q_len == kv_len == 8:
                            KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 0, 0, 0, 0, 0]
            Q_index [1, 1, 1, 1, 0, 0, 0, 0]
                    [1, 1, 1, 1, 1, 0, 0, 0]
                    [1, 1, 1, 1, 1, 1, 0, 0]
                    [1, 1, 1, 1, 1, 1, 1, 0]
                    [1, 1, 1, 1, 1, 1, 1, 1]

            Head-tail load-balance strategy rearranges the Q tensor by combining
            Q[0:k] (on seq dim) and Q[-k:] for rank 0, Q[k:2k] and Q[-2k:-k] for
            rank 1, and so on. In python code it looks like:

                k = Q.size(0) // (2 * cp_world_size)
                for rank in range(cp_world_size):
                    reordered_Q[rank * 2 * k : (rank + 1) * 2 * k] = torch.cat(
                        (Q[rank * k : (rank + 1) * k], Q[-(rank + 1) * k : -rank * k])
````

- **L101** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L102** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L103** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L104** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L105** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L106** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L107** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L108** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L109** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L110** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L111** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L112** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L113** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L114** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L115** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L116** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L117** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L118** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L119** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L120** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 121-140 / 第 121-140 行

````python
                    )

            This can also be done by tensor slicing. For the above example, the indices
            tensor for slicing is:
                slice_indices = Tensor([0, 7, 1, 6, 2, 5, 3, 4])

            After reordering QKV using the `slice_indices`, the corresponding mask matrix
            distributing over 2 devices becomes well-balanced:
                            KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 1, 1, 1, 1, 1]
                    [1, 1, 0, 0, 0, 0, 0, 0]    rank 0
                    [1, 1, 1, 1, 1, 1, 1, 0]
            Q_index ------------------------
                    [1, 1, 1, 0, 0, 0, 0, 0]
                    [1, 1, 1, 1, 1, 1, 0, 0]    rank 1
                    [1, 1, 1, 1, 0, 0, 0, 0]
                    [1, 1, 1, 1, 1, 0, 0, 0]

            To restore the reordering and putting the tensor back, slicing op can do the
````

- **L121** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L122** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L123** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L124** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L125** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L126** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L127** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L128** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L129** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L130** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L131** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L132** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L133** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L134** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L135** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L136** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L137** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L138** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L139** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L140** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 141-160 / 第 141-160 行

````python
            trick with a `restore_indices` such that:
                slice_indices[restore_indices] == Tensor([0, 1, 2, ...])

            In this way, `reordered_Q[restore_indices]` will just be the original Q.
        """
        seq_length = self.seq_length
        world_size = self.world_size
        if seq_length % (world_size * 2) != 0:
            raise AssertionError
        chunk_size = seq_length // (world_size * 2)

        # Split sequence into 2*world_size chunks, then pair chunk r with
        # chunk (2*world_size - 1 - r) for each rank.
        indices = torch.arange(seq_length, dtype=torch.int, device=self.device)
        chunks = indices.view(world_size * 2, chunk_size)
        head_idx = torch.arange(world_size, device=self.device)
        tail_idx = 2 * world_size - 1 - head_idx
        paired = torch.stack([chunks[head_idx], chunks[tail_idx]], dim=1)
        all_indices_tensor = paired.reshape(-1)

````

- **L141** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L142** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L143** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L144** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L145** EN: Closes the docstring for the function _generate_indices. | CN: 结束 function _generate_indices 的文档字符串。
- **L146** EN: Assigns or updates `seq_length`. | CN: 对 `seq_length` 进行赋值或更新。
- **L147** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L148** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L149** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L150** EN: Assigns or updates `chunk_size`. | CN: 对 `chunk_size` 进行赋值或更新。
- **L151** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L152** EN: Keeps the inline comment or directive: Split sequence into 2*world_size chunks, then pair chunk r with | CN: 保留这一行注释或指令：Split sequence into 2*world_size chunks, then pair chunk r with
- **L153** EN: Keeps the inline comment or directive: chunk (2*world_size - 1 - r) for each rank. | CN: 保留这一行注释或指令：chunk (2*world_size - 1 - r) for each rank.
- **L154** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L155** EN: Assigns or updates `chunks`. | CN: 对 `chunks` 进行赋值或更新。
- **L156** EN: Assigns or updates `head_idx`. | CN: 对 `head_idx` 进行赋值或更新。
- **L157** EN: Assigns or updates `tail_idx`. | CN: 对 `tail_idx` 进行赋值或更新。
- **L158** EN: Assigns or updates `paired`. | CN: 对 `paired` 进行赋值或更新。
- **L159** EN: Assigns or updates `all_indices_tensor`. | CN: 对 `all_indices_tensor` 进行赋值或更新。
- **L160** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 161-180 / 第 161-180 行

````python
        if restore:
            all_indices_tensor = torch.argsort(all_indices_tensor)

        return all_indices_tensor.unsqueeze(0)  # add batch dim


class _PerDocumentHeadTailLoadBalancer(_LoadBalancer):
    def __init__(
        self,
        seq_length_per_doc: list[list[int]],
        world_size: int,
        device: str | torch.device,
    ):
        """
        `seq_length_per_doc` has size (B, seq_len) if the load-balancing should vary
        within the batch. Otherwise `seq_length_per_doc` should have size (1, seq_len).
        """
        self.seq_length_per_doc = seq_length_per_doc
        self.world_size = world_size
        self.device = device
````

- **L161** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L162** EN: Assigns or updates `all_indices_tensor`. | CN: 对 `all_indices_tensor` 进行赋值或更新。
- **L163** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L164** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L165** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L166** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L167** EN: Defines class `_PerDocumentHeadTailLoadBalancer`. | CN: 定义类 `_PerDocumentHeadTailLoadBalancer`。
- **L168** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L169** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L170** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L171** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L172** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L173** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L174** EN: Starts the docstring for the function __init__. | CN: 开始定义 function __init__ 的文档字符串。
- **L175** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L176** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L177** EN: Closes the docstring for the function __init__. | CN: 结束 function __init__ 的文档字符串。
- **L178** EN: Assigns or updates `self.seq_length_per_doc`. | CN: 对 `self.seq_length_per_doc` 进行赋值或更新。
- **L179** EN: Assigns or updates `self.world_size`. | CN: 对 `self.world_size` 进行赋值或更新。
- **L180** EN: Assigns or updates `self.device`. | CN: 对 `self.device` 进行赋值或更新。

### Lines 181-200 / 第 181-200 行

````python

    def _generate_indices(self, restore: bool = False) -> Tensor:
        """
        Generate the per-document head-and-tail rearrange indices so that after rearranging
        the input is load-balanced in per-document head-and-tail style.

        Args:
            restore:
                If True, generate restore indices that map per-document head-and-tail
                rearranged positions back to original positions. If False, generate load
                balance indices that rearrange original positions to per-document
                head-and-tail pattern.

        Returns:
            The generated indices of shape `(batch_size, seq_len)` if the load-balancing
            should vary within the batch. Otherwise, it should have shape `(1, seq_len)`.

        Warning:
            For Multi-Head Attention, we require the masks over the head dimension are identical
            (i.e. `seq_length_per_doc` must have size (B, seq_len) or (1, seq_len)).
````

- **L181** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L182** EN: Defines function `_generate_indices`. | CN: 定义函数 `_generate_indices`。
- **L183** EN: Starts the docstring for the function _generate_indices. | CN: 开始定义 function _generate_indices 的文档字符串。
- **L184** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L185** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L186** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L187** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L188** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L189** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L190** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L191** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L192** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L193** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L194** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L195** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L196** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L197** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L198** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L199** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L200** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 201-220 / 第 201-220 行

````python

        Example:
            Here is the document causal mask for attention where q_len == kv_len == 16:
                                        KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
            Q_index [0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1]
````

- **L201** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L202** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L203** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L204** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L205** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L206** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L207** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L208** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L209** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L210** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L211** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L212** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L213** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L214** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L215** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L216** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L217** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L218** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L219** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L220** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 221-240 / 第 221-240 行

````python

            The per-document head-and-tail load-balancer will apply head-and-tail
            reordering within each document. After load-balancing for context-parallel
            on 2 devices, the above mask matrix will look like this:
                                        KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]
            Q_index [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1]
                    ------------------------------------------------
                    [1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0]
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0]
````

- **L221** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L222** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L223** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L224** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L225** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L226** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L227** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L228** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L229** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L230** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L231** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L232** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L233** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L234** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L235** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L236** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L237** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L238** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L239** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L240** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 241-260 / 第 241-260 行

````python
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0]
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0]
        """
        return torch.stack(
            [
                self._generate_indices_for_batch(seq_lengths, restore)
                for seq_lengths in self.seq_length_per_doc
            ]
        )

    def _generate_indices_for_batch(self, seq_length_per_doc, restore) -> Tensor:  # type: ignore[no-untyped-def]
        world_size = self.world_size
        device = self.device
        if not all(
            seq_length % (2 * world_size) == 0 for seq_length in seq_length_per_doc
        ):
            raise AssertionError
        chunk_length_per_doc = [
            seq_length // (2 * world_size) for seq_length in seq_length_per_doc
        ]
````

- **L241** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L242** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L243** EN: Closes the docstring for the function _generate_indices. | CN: 结束 function _generate_indices 的文档字符串。
- **L244** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L245** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L246** EN: Calls `self._generate_indices_for_batch` as part of the current workflow. | CN: 在当前流程中调用 `self._generate_indices_for_batch`。
- **L247** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L248** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L249** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L250** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L251** EN: Defines function `_generate_indices_for_batch`. | CN: 定义函数 `_generate_indices_for_batch`。
- **L252** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L253** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L254** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L255** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L256** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L257** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L258** EN: Assigns or updates `chunk_length_per_doc`. | CN: 对 `chunk_length_per_doc` 进行赋值或更新。
- **L259** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L260** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 261-280 / 第 261-280 行

````python

        indices = []
        document_start_idx = 0
        for seq_length, chunk_length in zip(seq_length_per_doc, chunk_length_per_doc):
            # Generate the indices for the current document
            for rank in range(world_size):
                head_chunk_start_idx = document_start_idx + chunk_length * rank
                tail_chunk_end_idx = document_start_idx + chunk_length * (
                    2 * world_size - rank
                )
                indices.append(
                    torch.arange(
                        head_chunk_start_idx,
                        head_chunk_start_idx + chunk_length,
                        device=device,
                    )
                )
                indices.append(
                    torch.arange(
                        tail_chunk_end_idx - chunk_length,
````

- **L261** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L262** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L263** EN: Assigns or updates `document_start_idx`. | CN: 对 `document_start_idx` 进行赋值或更新。
- **L264** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L265** EN: Keeps the inline comment or directive: Generate the indices for the current document | CN: 保留这一行注释或指令：Generate the indices for the current document
- **L266** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L267** EN: Assigns or updates `head_chunk_start_idx`. | CN: 对 `head_chunk_start_idx` 进行赋值或更新。
- **L268** EN: Assigns or updates `tail_chunk_end_idx`. | CN: 对 `tail_chunk_end_idx` 进行赋值或更新。
- **L269** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L270** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L271** EN: Calls `indices.append` as part of the current workflow. | CN: 在当前流程中调用 `indices.append`。
- **L272** EN: Calls `torch.arange` as part of the current workflow. | CN: 在当前流程中调用 `torch.arange`。
- **L273** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L274** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L275** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L276** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L277** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L278** EN: Calls `indices.append` as part of the current workflow. | CN: 在当前流程中调用 `indices.append`。
- **L279** EN: Calls `torch.arange` as part of the current workflow. | CN: 在当前流程中调用 `torch.arange`。
- **L280** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。

### Lines 281-300 / 第 281-300 行

````python
                        tail_chunk_end_idx,
                        device=device,
                    )
                )

            document_start_idx += seq_length

        indices_tensor = torch.cat(indices)
        if restore:
            indices_tensor = torch.argsort(indices_tensor)

        return indices_tensor


class _PTRRLoadBalancer(_LoadBalancer):
    """
    Processing-Time based Round-Robin (PTRR) load balancer. This load balancer should
    only be used for flex_attention() since it leverages `BlockMask`.
    """

````

- **L281** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L282** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L283** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L284** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L285** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L286** EN: Continues the implementation inside function `_generate_indices_for_batch`. | CN: 继续说明函数 `_generate_indices_for_batch` 内部的实现。
- **L287** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L288** EN: Assigns or updates `indices_tensor`. | CN: 对 `indices_tensor` 进行赋值或更新。
- **L289** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L290** EN: Assigns or updates `indices_tensor`. | CN: 对 `indices_tensor` 进行赋值或更新。
- **L291** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L292** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L293** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L294** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L295** EN: Defines class `_PTRRLoadBalancer`. | CN: 定义类 `_PTRRLoadBalancer`。
- **L296** EN: Starts the docstring for the class _PTRRLoadBalancer. | CN: 开始定义 class _PTRRLoadBalancer 的文档字符串。
- **L297** EN: Continues the docstring text for the class _PTRRLoadBalancer. | CN: 继续补充 class _PTRRLoadBalancer 的文档字符串内容。
- **L298** EN: Continues the docstring text for the class _PTRRLoadBalancer. | CN: 继续补充 class _PTRRLoadBalancer 的文档字符串内容。
- **L299** EN: Closes the docstring for the class _PTRRLoadBalancer. | CN: 结束 class _PTRRLoadBalancer 的文档字符串。
- **L300** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 301-320 / 第 301-320 行

````python
    def __init__(
        self,
        block_mask: BlockMask,
        world_size: int,
    ):
        """
        `block_mask` must have shape (B, 1, seq_len, seq_len) or (1, 1, seq_len, seq_len).
        """
        self.block_mask = block_mask
        self.world_size = world_size

    @staticmethod
    def ptrr_scheduling(process_time: Tensor, group_size: int) -> Tensor:
        """
        Separate the tasks into `group_size` groups using PTRR scheduling.
        process_time:
            1D tensor of size n, where n is the number of tasks. The value
            is the process time of the task. Size `n` must be divisible by
            `group_size`.
        group_size:
````

- **L301** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L302** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L303** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L304** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L305** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L306** EN: Starts the docstring for the function __init__. | CN: 开始定义 function __init__ 的文档字符串。
- **L307** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L308** EN: Closes the docstring for the function __init__. | CN: 结束 function __init__ 的文档字符串。
- **L309** EN: Assigns or updates `self.block_mask`. | CN: 对 `self.block_mask` 进行赋值或更新。
- **L310** EN: Assigns or updates `self.world_size`. | CN: 对 `self.world_size` 进行赋值或更新。
- **L311** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L312** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L313** EN: Defines function `ptrr_scheduling`. | CN: 定义函数 `ptrr_scheduling`。
- **L314** EN: Starts the docstring for the function ptrr_scheduling. | CN: 开始定义 function ptrr_scheduling 的文档字符串。
- **L315** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L316** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L317** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L318** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L319** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L320** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。

### Lines 321-340 / 第 321-340 行

````python
            the number of groups

        Returns:
        tasks_in_group (list[list[int]]):
            A collection of list[int] and each list should have size `n // group_size`
            (`group_size` lists in total). Each element is an index in the input
            `process_time` (i.e. [0, len(process_time) - 1]).

        Example:
            process_time = [9, 14, 2, 20, 10, 15, 8, 14, 16, 19, 15, 3, 12, 1, 12, 10]
            tasks_in_group = [
                [3, 12, 13, 14],    # values = [1, 12, 12, 20], sum = 45
                [2, 4, 7, 9],       # values = [2, 10, 14, 19], sum = 45
                [1, 8, 11, 15],     # values = [14, 16, 3, 10], sum = 43
                [0, 5, 6, 10]       # values = [9, 15, 8, 15], sum = 47
            ]
        """
        if process_time.ndim != 1:
            raise AssertionError

````

- **L321** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L322** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L323** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L324** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L325** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L326** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L327** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L328** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L329** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L330** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L331** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L332** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L333** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L334** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L335** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L336** EN: Continues the docstring text for the function ptrr_scheduling. | CN: 继续补充 function ptrr_scheduling 的文档字符串内容。
- **L337** EN: Closes the docstring for the function ptrr_scheduling. | CN: 结束 function ptrr_scheduling 的文档字符串。
- **L338** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L339** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L340** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 341-360 / 第 341-360 行

````python
        num_tasks = process_time.size(0)

        if num_tasks % group_size != 0:
            raise NotImplementedError(
                f"num_tasks {num_tasks} must be divisible by group_size {group_size}"
            )

        device = process_time.device
        _, sorted_indices_descending = torch.sort(
            process_time, descending=True, stable=True
        )  # if process time is tied, the order is preserved
        sorted_indices_descending_reversed = torch.flip(
            sorted_indices_descending.view(-1, group_size), dims=[1]
        ).view(-1)
        tasks_in_group = torch.where(
            torch.arange(num_tasks, device=device) // group_size % 2 == 0,
            sorted_indices_descending,
            sorted_indices_descending_reversed,
        )
        tasks_in_group = tasks_in_group.view(-1, group_size).transpose(
````

- **L341** EN: Assigns or updates `num_tasks`. | CN: 对 `num_tasks` 进行赋值或更新。
- **L342** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L343** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L344** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L345** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L346** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L347** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L348** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L349** EN: Assigns or updates `_, sorted_indices_descending`. | CN: 对 `_, sorted_indices_descending` 进行赋值或更新。
- **L350** EN: Assigns or updates `process_time, descending`. | CN: 对 `process_time, descending` 进行赋值或更新。
- **L351** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L352** EN: Assigns or updates `sorted_indices_descending_reversed`. | CN: 对 `sorted_indices_descending_reversed` 进行赋值或更新。
- **L353** EN: Calls `sorted_indices_descending.view` as part of the current workflow. | CN: 在当前流程中调用 `sorted_indices_descending.view`。
- **L354** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L355** EN: Assigns or updates `tasks_in_group`. | CN: 对 `tasks_in_group` 进行赋值或更新。
- **L356** EN: Calls `torch.arange` as part of the current workflow. | CN: 在当前流程中调用 `torch.arange`。
- **L357** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L358** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L359** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L360** EN: Assigns or updates `tasks_in_group`. | CN: 对 `tasks_in_group` 进行赋值或更新。

### Lines 361-380 / 第 361-380 行

````python
            0, 1
        )  # (group_size, n // group_size)

        # sort each group. This step should not have impact on correctness
        # nor execution run time, but it helps users visualize the mask
        tasks_in_group, _ = torch.sort(tasks_in_group, dim=1)
        return tasks_in_group

    def _generate_indices(self, restore: bool = False) -> Tensor:
        """
        Generate the PTRR reorder indices of shape `(1, seq_len)` or `(batch_size, seq_len)`.

        Args:
            restore:
                If True, generate restore indices that map Processing-Time based Round-Robin
                (PTRR) rearranged positions back to original positions. If False, generate
                load balance indices that rearrange original positions to PTRR pattern.

            Returns:
                The generated indices of shape `(1, seq_len)` if the load-balancing is
````

- **L361** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L362** EN: Continues the implementation inside function `ptrr_scheduling`. | CN: 继续说明函数 `ptrr_scheduling` 内部的实现。
- **L363** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L364** EN: Keeps the inline comment or directive: sort each group. This step should not have impact on correctness | CN: 保留这一行注释或指令：sort each group. This step should not have impact on correctness
- **L365** EN: Keeps the inline comment or directive: nor execution run time, but it helps users visualize the mask | CN: 保留这一行注释或指令：nor execution run time, but it helps users visualize the mask
- **L366** EN: Assigns or updates `tasks_in_group, _`. | CN: 对 `tasks_in_group, _` 进行赋值或更新。
- **L367** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L368** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L369** EN: Defines function `_generate_indices`. | CN: 定义函数 `_generate_indices`。
- **L370** EN: Starts the docstring for the function _generate_indices. | CN: 开始定义 function _generate_indices 的文档字符串。
- **L371** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L372** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L373** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L374** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L375** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L376** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L377** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L378** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L379** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L380** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 381-400 / 第 381-400 行

````python
                identical within the batch (i.e. `BlockMask.shape[0] == 1`), or
                `(batch_size, seq_len)` if the load-balancing should vary within the batch.

        Warning:
            For Multi-Head Attention, we require the masks over the head dimension are identical
            (i.e. `self.block_mask` must have shape (B, 1, seq_len, seq_len) or (1, 1, seq_len, seq_len)).

        Example:
            Here is the document causal mask for attention whereq_len == kv_len == 16 * BLOCK_SIZE
            (each entry is a block):
                                        KV_index
                    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 1
                    [1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 2
                    [1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 3
                    [1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 4
                    [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 1
                    [0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 2
                    [0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 3
            Q_index [0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 4
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0]  -> row value = 5
````

- **L381** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L382** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L383** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L384** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L385** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L386** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L387** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L388** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L389** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L390** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L391** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L392** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L393** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L394** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L395** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L396** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L397** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L398** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L399** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L400** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 401-420 / 第 401-420 行

````python
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0]  -> row value = 6
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0]  -> row value = 7
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0]  -> row value = 8
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]  -> row value = 1
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0]  -> row value = 2
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0]  -> row value = 3
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1]  -> row value = 4

            The reorder indices will be: [2, 3, 5, 6, 8, 11, 12, 13, 0, 1, 4, 7, 9, 10, 14, 15] and
            the mask matrix will look like:
                                        KV_index
                    [1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 3
                    [1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 4
                    [0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 2
                    [0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 3
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0]  -> row value = 5  rank 0 (sum=28)
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0]  -> row value = 8
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0]  -> row value = 1
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0]  -> row value = 2
                    ------------------------------------------------
````

- **L401** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L402** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L403** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L404** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L405** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L406** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L407** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L408** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L409** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L410** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L411** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L412** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L413** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L414** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L415** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L416** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L417** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L418** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L419** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L420** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。

### Lines 421-440 / 第 421-440 行

````python
                    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 1
                    [1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 2
                    [0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 1
                    [0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0]  -> row value = 4
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0]  -> row value = 6  rank 1 (sum=28)
                    [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0]  -> row value = 7
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0]  -> row value = 3
                    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1]  -> row value = 4
        """
        block_mask = self.block_mask
        kv_num_blocks = block_mask.kv_num_blocks
        full_kv_num_blocks = block_mask.full_kv_num_blocks
        non_sparse_kv_num_blocks = (
            kv_num_blocks + full_kv_num_blocks
            if full_kv_num_blocks is not None
            else kv_num_blocks
        )
        B, H, Q = non_sparse_kv_num_blocks.shape
        # requirement: the masking is identical across heads (i.e. H == 1 in BlockMask)
        non_sparse_kv_num_blocks = non_sparse_kv_num_blocks.view(-1, Q)  # (B, Q_BLK)
````

- **L421** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L422** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L423** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L424** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L425** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L426** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L427** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L428** EN: Continues the docstring text for the function _generate_indices. | CN: 继续补充 function _generate_indices 的文档字符串内容。
- **L429** EN: Closes the docstring for the function _generate_indices. | CN: 结束 function _generate_indices 的文档字符串。
- **L430** EN: Assigns or updates `block_mask`. | CN: 对 `block_mask` 进行赋值或更新。
- **L431** EN: Assigns or updates `kv_num_blocks`. | CN: 对 `kv_num_blocks` 进行赋值或更新。
- **L432** EN: Assigns or updates `full_kv_num_blocks`. | CN: 对 `full_kv_num_blocks` 进行赋值或更新。
- **L433** EN: Assigns or updates `non_sparse_kv_num_blocks`. | CN: 对 `non_sparse_kv_num_blocks` 进行赋值或更新。
- **L434** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L435** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L436** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L437** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L438** EN: Assigns or updates `B, H, Q`. | CN: 对 `B, H, Q` 进行赋值或更新。
- **L439** EN: Keeps the inline comment or directive: requirement: the masking is identical across heads (i.e. H == 1 in BlockMask) | CN: 保留这一行注释或指令：requirement: the masking is identical across heads (i.e. H == 1 in BlockMask)
- **L440** EN: Assigns or updates `non_sparse_kv_num_blocks`. | CN: 对 `non_sparse_kv_num_blocks` 进行赋值或更新。

### Lines 441-460 / 第 441-460 行

````python

        batch_ptrr = torch.vmap(
            functools.partial(
                _PTRRLoadBalancer.ptrr_scheduling,
                group_size=self.world_size,
            )
        )
        ptrr_indices = batch_ptrr(
            non_sparse_kv_num_blocks
        )  # (B, group_size, num_blks_in_group)
        ptrr_indices = ptrr_indices.reshape(B, -1)  # (B, num_blocks)

        # NOTE: only support the case where the qkv block size are equal
        q_blk_size, kv_blk_size = block_mask.BLOCK_SIZE
        if q_blk_size != kv_blk_size:
            raise AssertionError("for now only support q_blk_size == kv_blk_size")

        indices = torch.arange(
            q_blk_size * ptrr_indices.size(1), device=ptrr_indices.device
        ).view(-1, q_blk_size)  # (NUM_BLOCKS, BLOCK_SIZE)
````

- **L441** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L442** EN: Assigns or updates `batch_ptrr`. | CN: 对 `batch_ptrr` 进行赋值或更新。
- **L443** EN: Calls `functools.partial` as part of the current workflow. | CN: 在当前流程中调用 `functools.partial`。
- **L444** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L445** EN: Assigns or updates `group_size`. | CN: 对 `group_size` 进行赋值或更新。
- **L446** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L447** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L448** EN: Assigns or updates `ptrr_indices`. | CN: 对 `ptrr_indices` 进行赋值或更新。
- **L449** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L450** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L451** EN: Assigns or updates `ptrr_indices`. | CN: 对 `ptrr_indices` 进行赋值或更新。
- **L452** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L453** EN: Keeps the inline comment or directive: NOTE: only support the case where the qkv block size are equal | CN: 保留这一行注释或指令：NOTE: only support the case where the qkv block size are equal
- **L454** EN: Assigns or updates `q_blk_size, kv_blk_size`. | CN: 对 `q_blk_size, kv_blk_size` 进行赋值或更新。
- **L455** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L456** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L457** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L458** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L459** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。
- **L460** EN: Continues the implementation inside function `_generate_indices`. | CN: 继续说明函数 `_generate_indices` 内部的实现。

### Lines 461-478 / 第 461-478 行

````python
        indices = indices[ptrr_indices].view(B, -1)  # (B, qkv_size)

        if restore:
            # pyrefly: ignore[missing-argument]
            indices = torch.vmap(torch.argsort)(indices)

        return indices


def _create_default_load_balancer(
    seq_length: int, world_size: int, device: str | torch.device
) -> _LoadBalancer | None:
    from ._attention import _cp_options

    if _cp_options.enable_load_balance:
        return _HeadTailLoadBalancer(seq_length, world_size, device)
    else:
        return None
````

- **L461** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L462** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L463** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L464** EN: Keeps the inline comment or directive: pyrefly: ignore[missing-argument] | CN: 保留这一行注释或指令：pyrefly: ignore[missing-argument]
- **L465** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L466** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L467** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L468** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L469** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L470** EN: Defines function `_create_default_load_balancer`. | CN: 定义函数 `_create_default_load_balancer`。
- **L471** EN: Continues the implementation inside function `_create_default_load_balancer`. | CN: 继续说明函数 `_create_default_load_balancer` 内部的实现。
- **L472** EN: Continues the implementation inside function `_create_default_load_balancer`. | CN: 继续说明函数 `_create_default_load_balancer` 内部的实现。
- **L473** EN: Imports selected names from `._attention`. | CN: 从 `._attention` 导入指定名称。
- **L474** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L475** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L476** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L477** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L478** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: Primary classes: _LoadBalancer, _HeadTailLoadBalancer, _PerDocumentHeadTailLoadBalancer, _PTRRLoadBalancer  
  **CN**: 主要类：_LoadBalancer, _HeadTailLoadBalancer, _PerDocumentHeadTailLoadBalancer, _PTRRLoadBalancer
- **EN**: Core callables: _create_default_load_balancer  
  **CN**: 核心可调用对象：_create_default_load_balancer

## Dependencies / 依赖关系

- **Internal / 内部**: `._attention`
- **PyTorch / PyTorch**: `torch`, `torch.nn.attention.flex_attention`
- **Python Stdlib / Python 标准库**: `abc`, `functools`
- **Third-party / 第三方**: None detected / 未检测到

