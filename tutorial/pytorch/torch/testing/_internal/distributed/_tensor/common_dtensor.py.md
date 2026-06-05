# common_dtensor.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/distributed/_tensor/common_dtensor.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common dtensor, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common dtensor 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12
```python
# mypy: allow-untyped-defs

# Copyright (c) Meta Platforms, Inc. and affiliates

import contextlib
import copy
import functools
import itertools
import sys
import threading
import types
import unittest
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `contextlib`, `copy`, `functools`, `itertools`, `sys`, `threading`, `...`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`contextlib`, `copy`, `functools`, `itertools`, `sys`, `threading`, `...`。

### Lines 13-26
```python
from collections.abc import Callable, Iterator, Sequence
from dataclasses import dataclass
from functools import partial, wraps
from typing import Any, cast, TypeVar

import torch
import torch.distributed as dist
import torch.nn as nn
import torch.nn.functional as F
from torch.distributed._functional_collectives import (
    all_gather_tensor_autograd,
    reduce_scatter_tensor_autograd,
)
from torch.distributed._local_tensor import (
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 27-44
```python
    LocalIntNode,
    LocalTensor,
    LocalTensorMode,
    maybe_disable_local_tensor_mode,
    maybe_run_for_local_tensor,
)
from torch.distributed.tensor import (
    DeviceMesh,
    distribute_module,
    distribute_tensor,
    DTensor,
    init_device_mesh,
    Partial,
    Placement,
    Replicate,
    Shard,
)
from torch.distributed.tensor._dtensor_spec import ShardOrderEntry
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：无明显局部符号。

### Lines 45-63
```python
from torch.distributed.tensor._redistribute import redistribute_local_tensor
from torch.distributed.tensor.parallel import (
    ColwiseParallel,
    parallelize_module,
    ParallelStyle,
    PrepareModuleInput,
    RowwiseParallel,
    SequenceParallel,
)
from torch.testing._internal.common_distributed import (
    ACCELERATOR_DIST_BACKENDS,
    MultiProcContinuousTest,
    MultiProcessTestCase,
    MultiThreadedTestCase,
    run_subtests,
    skip_if_lt_x_gpu,
    TEST_SKIPS,
)
from torch.testing._internal.common_utils import (
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 64-75
```python
    TEST_CUDA,
    TEST_HPU,
    TEST_PRIVATEUSE1,
    TEST_WITH_ROCM,
    TEST_XPU,
)
from torch.testing._internal.distributed.fake_pg import FakeStore
from torch.utils._pytree import tree_flatten, tree_unflatten, TreeSpec


DEVICE_COUNT: int

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 76-88
```python
if TEST_CUDA or TEST_XPU or TEST_HPU or TEST_PRIVATEUSE1:
    DEVICE_TYPE = torch.accelerator.current_accelerator().type
    DEVICE_COUNT = torch.accelerator.device_count()
    PG_BACKEND = dist.Backend.default_device_backend_map[DEVICE_TYPE]
else:
    DEVICE_TYPE = "cpu"
    PG_BACKEND = "gloo"

if TEST_WITH_ROCM:
    NUM_DEVICES = min(4, max(2, torch.cuda.device_count()))
else:
    NUM_DEVICES = 4

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；处理条件控制流。关键符号：无明显局部符号。

### Lines 89-103
```python
# We use this as a proxy for "multiple GPUs exist"
if (TEST_CUDA or TEST_XPU or TEST_HPU or TEST_PRIVATEUSE1) and DEVICE_COUNT > 1:
    # when we actually have multiple GPUs, relax the requirement to smaller counts.
    NUM_DEVICES = min(NUM_DEVICES, DEVICE_COUNT)

T = TypeVar("T")


# simple RMSNorm layer for testing
class RMSNormPython(torch.nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = torch.nn.Parameter(torch.ones(dim))

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `T`, `RMSNormPython`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`T`, `RMSNormPython`。

### Lines 104-119
```python
    def _norm(self, x):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)

    def forward(self, x):
        output = self._norm(x)
        return output * self.weight


class MLPModule(nn.Module):
    def __init__(self, device, bias: bool = True):
        super().__init__()
        torch.manual_seed(5)
        self.net1 = nn.Linear(10, 16, bias=bias, device=device)
        self.relu = nn.ReLU()
        self.net2 = nn.Linear(16, 10, bias=bias, device=device)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `_norm`, `forward`, `MLPModule`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`_norm`, `forward`, `MLPModule`, `__init__`。

### Lines 120-132
```python
    def forward(self, x):
        return self.net2(self.relu(self.net1(x)))

    def reset_parameters(self):
        self.net1.reset_parameters()
        self.net2.reset_parameters()


class MLPStacked(nn.Module):
    def __init__(self, device, n_layers: int = 2):
        super().__init__()
        self.layers = nn.ModuleList([MLPModule(device) for i in range(n_layers)])

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `forward`, `reset_parameters`, `MLPStacked`, `__init__`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`forward`, `reset_parameters`, `MLPStacked`, `__init__`。

### Lines 133-151
```python
    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x


@dataclass
class ModelArgs:
    n_layers: int = 2
    vocab_size: int = 8
    max_seq_len: int = 16
    dim: int = 16
    n_heads: int = 4
    dropout_p: float = 0.1
    use_attn_mask: bool = True
    weight_tying: bool = True
    checkpoint_activations: bool = False
    num_experts: int = 0

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `forward`, `ModelArgs`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`forward`, `ModelArgs`。

### Lines 152-165
```python

class Attention(nn.Module):
    def __init__(self, args: ModelArgs):
        super().__init__()
        if args.dim % args.n_heads != 0:
            raise AssertionError(
                f"Expected args.dim % args.n_heads == 0, got {args.dim} % {args.n_heads}"
            )
        self.head_dim = args.dim // args.n_heads
        self.n_heads = args.n_heads
        self.dropout_p = args.dropout_p
        self.resid_dropout = nn.Dropout(args.dropout_p)
        self.use_attn_mask = args.use_attn_mask

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Attention`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Attention`。

### Lines 166-177
```python
        self.wq = nn.Linear(args.dim, args.dim, bias=False)
        self.wk = nn.Linear(args.dim, args.dim, bias=False)
        self.wv = nn.Linear(args.dim, args.dim, bias=False)
        self.wo = nn.Linear(args.dim, args.dim, bias=False)

    def forward(self, x):
        bsz, seq_len, _ = x.size()
        queries, keys, values = self.wq(x), self.wk(x), self.wv(x)
        queries = queries.view(bsz, seq_len, self.n_heads, self.head_dim)
        keys = keys.view(bsz, seq_len, self.n_heads, self.head_dim)
        values = values.view(bsz, seq_len, self.n_heads, self.head_dim)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 178-191
```python
        queries = queries.transpose(1, 2)  # (bsz, n_heads, seq_len, head_dim)
        keys = keys.transpose(1, 2)  # (bsz, n_heads, seq_len, head_dim)
        values = values.transpose(1, 2)  # (bsz, n_heads, seq_len, head_dim)

        output = F.scaled_dot_product_attention(
            queries,
            keys,
            values,
            None,
            self.dropout_p if self.training else 0,
            self.use_attn_mask,
        )
        output = output.transpose(1, 2).contiguous().view(bsz, seq_len, -1)
        return self.resid_dropout(self.wo(output))
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 192-203
```python


class FeedForward(nn.Module):
    def __init__(self, dim, hidden_dim, dropout_p):
        super().__init__()
        self.w1 = nn.Linear(dim, hidden_dim)
        self.gelu = nn.GELU()
        self.w2 = nn.Linear(hidden_dim, dim)
        self.resid_dropout = nn.Dropout(dropout_p)

    def forward(self, x):
        return self.resid_dropout(self.w2(self.gelu(self.w1(x))))
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FeedForward`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FeedForward`。

### Lines 204-215
```python


class Experts(nn.Module):
    def __init__(self, dim: int, hidden_dim: int, num_experts: int):
        super().__init__()
        self.num_experts = num_experts
        self.w1 = nn.Parameter(torch.empty(num_experts, hidden_dim, dim))
        self.w2 = nn.Parameter(torch.empty(num_experts, dim, hidden_dim))
        self.reset_parameters()
        self.gelu = nn.GELU()

    def reset_parameters(self):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `Experts`, `__init__`, `reset_parameters`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`Experts`, `__init__`, `reset_parameters`。

### Lines 216-230
```python
        nn.init.normal_(self.w1, std=0.02)
        nn.init.normal_(self.w2, std=0.02)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # Weights are DTensors (sharded by EP/TP) but x is a plain tensor
        # (dispatched by EP hooks), so extract local shards for bmm.
        if isinstance(self.w1, DTensor):
            w1, w2 = self.w1.to_local(), self.w2.to_local()
        else:
            w1, w2 = self.w1, self.w2
        E = w1.shape[0]
        x_exp = x.unsqueeze(0).expand(E, -1, -1)
        h = self.gelu(torch.bmm(x_exp, w1.transpose(-2, -1)))
        out = torch.bmm(h, w2.transpose(-2, -1))
        return out.sum(dim=0)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 231-244
```python


class ExpertLayer(nn.Module):
    def __init__(self, num_experts: int, dim: int, hidden_dim: int):
        super().__init__()
        self.num_experts = num_experts
        self.experts = Experts(dim=dim, hidden_dim=hidden_dim, num_experts=num_experts)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        bs, slen, dim = x.shape
        x_flat = x.view(-1, dim)
        expert_out = self.experts(x_flat)
        expert_out = expert_out / self.num_experts
        return expert_out.view(bs, slen, dim)
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExpertLayer`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExpertLayer`。

### Lines 245-258
```python


class TensorParallelForExpert(ParallelStyle):
    """TP for Experts: shard w1 colwise (Shard(1)), w2 rowwise (Shard(2)).

    For seq parallel, set input_layouts=Shard(0) and output_layouts=Shard(0)
    to all-gather tokens before computation and reduce-scatter after.
    """

    def __init__(self, *, input_layouts=None, output_layouts=None):
        super().__init__()
        self.input_layouts = input_layouts
        self.output_layouts = output_layouts or Replicate()

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TensorParallelForExpert`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TensorParallelForExpert`。

### Lines 259-270
```python
    def _partition_fn(self, name, mod, device_mesh):
        for pn, p in mod.named_parameters(recurse=False):
            if pn == "w1":
                placement = [Shard(1)]
            elif pn == "w2":
                placement = [Shard(2)]
            else:
                continue
            mod.register_parameter(
                pn, nn.Parameter(distribute_tensor(p, device_mesh, placement))
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_partition_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_partition_fn`。

### Lines 271-282
```python
    def _input_fn(self, mod, inputs, device_mesh):
        if self.input_layouts is not None:
            x = inputs[0]
            x = (
                DTensor.from_local(x, device_mesh, [self.input_layouts])
                .redistribute(device_mesh, [Replicate()])
                .to_local()
            )
            return (x,)
        return inputs

    def _output_fn(self, mod, outputs, device_mesh):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_input_fn`, `_output_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_input_fn`, `_output_fn`。

### Lines 283-297
```python
        return (
            DTensor.from_local(outputs, device_mesh, [Partial()])
            .redistribute(device_mesh, [self.output_layouts])
            .to_local()
        )

    def _apply(self, module, device_mesh):
        return distribute_module(
            module,
            device_mesh,
            partition_fn=self._partition_fn,
            input_fn=self._input_fn,
            output_fn=self._output_fn,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_apply`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_apply`。

### Lines 298-310
```python

class ExpertParallel(ParallelStyle):
    """Distributes experts across ranks with Shard(0) on the expert dimension.

    Dispatch: all-gather tokens so every rank sees all tokens.
    Combine: reduce-scatter (sum) expert outputs back to token owners.
    """

    def _partition_fn(self, name: str, mod: nn.Module, device_mesh: DeviceMesh) -> None:
        for param_name, param in mod.named_parameters(recurse=False):
            dist_param = nn.Parameter(distribute_tensor(param, device_mesh, [Shard(0)]))
            mod.register_parameter(param_name, dist_param)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExpertParallel`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExpertParallel`。

### Lines 311-322
```python
    def _token_dispatch(
        self, mod: nn.Module, inputs: tuple, device_mesh: DeviceMesh
    ) -> tuple[torch.Tensor]:
        (x,) = inputs
        x_gathered = all_gather_tensor_autograd(
            x,
            gather_dim=0,
            group=device_mesh.get_group(),
        )
        x_gathered = torch.ops._c10d_functional.wait_tensor(x_gathered)
        return (x_gathered,)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_token_dispatch`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_token_dispatch`。

### Lines 323-334
```python
    def _token_combine(
        self, mod: nn.Module, output: torch.Tensor, device_mesh: DeviceMesh
    ) -> torch.Tensor:
        result = reduce_scatter_tensor_autograd(
            output,
            "sum",
            scatter_dim=0,
            group=device_mesh.get_group(),
        )
        result = torch.ops._c10d_functional.wait_tensor(result)
        return result

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_token_combine`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_token_combine`。

### Lines 335-347
```python
    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        return distribute_module(
            module,
            device_mesh,
            partition_fn=self._partition_fn,
            input_fn=self._token_dispatch,
            output_fn=self._token_combine,
        )


class ExpertParallelWithTP(ParallelStyle):
    """Combined EP + TP for experts.

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `_apply`, `ExpertParallelWithTP`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`_apply`, `ExpertParallelWithTP`。

### Lines 348-363
```python
    Applied to ExpertLayer. Distributes expert params on a 2D (ep, tp) mesh
    with [Shard(0), Shard(1/2)]. Token dispatch/combine hooks are registered
    on ExpertLayer (outer), TP reduction hook on Experts (inner), so forward
    execution is: EP dispatch -> TP input -> forward -> TP reduce -> EP combine.
    """

    def __init__(
        self,
        ep_mesh: DeviceMesh,
        tp_mesh: DeviceMesh,
    ):
        super().__init__()
        self.ep_mesh = ep_mesh
        self.tp_mesh = tp_mesh
        self.ep_tp_mesh = DeviceMesh._concatenate([ep_mesh, tp_mesh])

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 364-378
```python
    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        experts = module.experts  # type: ignore[attr-defined]

        # Partition expert weights on 2D (ep, tp) mesh
        for pn, p in experts.named_parameters(recurse=False):
            if pn == "w1":
                placements = [Shard(0), Shard(1)]
            elif pn == "w2":
                placements = [Shard(0), Shard(2)]
            else:
                continue
            experts.register_parameter(
                pn, nn.Parameter(distribute_tensor(p, self.ep_tp_mesh, placements))
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_apply`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_apply`。

### Lines 379-391
```python
        # EP dispatch/combine hooks on ExpertLayer (outer module)
        ep_mesh = self.ep_mesh

        def ep_dispatch(mod, inputs):
            (x,) = inputs
            x = all_gather_tensor_autograd(x, gather_dim=0, group=ep_mesh.get_group())
            return (torch.ops._c10d_functional.wait_tensor(x),)

        def ep_combine(mod, inputs, output):
            out = reduce_scatter_tensor_autograd(
                output, "sum", scatter_dim=0, group=ep_mesh.get_group()
            )
            return torch.ops._c10d_functional.wait_tensor(out)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `ep_dispatch`, `ep_combine`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`ep_dispatch`, `ep_combine`。

### Lines 392-406
```python

        module.register_forward_pre_hook(ep_dispatch)
        module.register_forward_hook(ep_combine)

        # TP reduction hook on Experts (inner module)
        tp_mesh = self.tp_mesh

        def tp_allreduce_input_grad(mod, inputs):
            (x,) = inputs
            return (
                DTensor.from_local(x, tp_mesh, [Replicate()]).to_local(
                    grad_placements=[Partial()]
                ),
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tp_allreduce_input_grad`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tp_allreduce_input_grad`。

### Lines 407-418
```python
        experts.register_forward_pre_hook(tp_allreduce_input_grad)

        def tp_reduce(mod, inputs, output):
            return (
                DTensor.from_local(output, tp_mesh, [Partial()])
                .redistribute(tp_mesh, [Replicate()])
                .to_local()
            )

        experts.register_forward_hook(tp_reduce)
        return module

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tp_reduce`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tp_reduce`。

### Lines 419-440
```python

class TransformerBlock(nn.Module):
    def __init__(self, args: ModelArgs):
        super().__init__()
        self.attention_norm = nn.LayerNorm(args.dim)
        self.attention = Attention(args)
        self.ffn_norm = nn.LayerNorm(args.dim)

        self.has_experts = args.num_experts > 0
        if self.has_experts:
            self.expert_layer = ExpertLayer(
                args.num_experts,
                dim=args.dim,
                hidden_dim=4 * args.dim,
            )
        else:
            self.feed_forward = FeedForward(
                args.dim,
                hidden_dim=4 * args.dim,
                dropout_p=args.dropout_p,
            )

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TransformerBlock`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TransformerBlock`。

### Lines 441-452
```python
    def forward(self, x):
        h = x + self.attention(self.attention_norm(x))
        if self.has_experts:
            out = h + self.expert_layer(self.ffn_norm(h))
        else:
            out = h + self.feed_forward(self.ffn_norm(h))
        return out


# A toy transformer model, partly inspired by the nanoGPT model:
# https://github.com/karpathy/nanoGPT.
class Transformer(nn.Module):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `forward`, `Transformer`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`forward`, `Transformer`。

### Lines 453-465
```python
    def __init__(self, args: ModelArgs):
        super().__init__()
        if args.vocab_size is None:
            raise AssertionError("Expected args.vocab_size to not be None")
        if args.max_seq_len is None:
            raise AssertionError("Expected args.max_seq_len to not be None")
        self.model_args = args
        self.max_seq_len = args.max_seq_len
        self.tok_embeddings = nn.Embedding(args.vocab_size, args.dim)
        self.pos_embeddings = nn.Embedding(args.max_seq_len, args.dim)
        self.dropout = nn.Dropout(args.dropout_p)
        self.layers = nn.ModuleList()
        for _ in range(args.n_layers):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 466-484
```python
            self.layers.append(TransformerBlock(args))
        self.norm = nn.LayerNorm(args.dim)
        self.output = nn.Linear(args.dim, args.vocab_size, bias=False)
        if args.weight_tying:
            self.output.weight = self.tok_embeddings.weight
        self.checkpoint_activations = args.checkpoint_activations

    def forward(self, tokens):
        _bsz, seq_len = tokens.size()
        if seq_len > self.max_seq_len:
            raise AssertionError(
                f"Expected seq_len <= max_seq_len, got {seq_len} > {self.max_seq_len}"
            )
        h = self.tok_embeddings(tokens)
        pos = torch.arange(0, seq_len, device=tokens.device)
        p = self.pos_embeddings(pos)  # positional embeddings of shape (seq_len, dim)
        h = h + p
        h = self.dropout(h)
        for layer in self.layers:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `forward`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`forward`。

### Lines 485-501
```python
            if self.checkpoint_activations:
                h = torch.utils.checkpoint.checkpoint(layer, h, use_reentrant=False)
            else:
                h = layer(h)
        h = self.norm(h)
        output = self.output(h).float()
        return output

    @staticmethod
    def parallelize(
        module: "Transformer",
        tp_mesh: DeviceMesh | None,
        use_seq_parallel: bool,
        local_output_for_attn: bool = False,
        ep_mesh: DeviceMesh | None = None,
    ) -> nn.Module:
        if not isinstance(module, Transformer):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `parallelize`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`parallelize`。

### Lines 502-525
```python
            raise AssertionError(f"Requires Transformer but got {module}")
        if tp_mesh is None and ep_mesh is None:
            raise ValueError("At least one of tp_mesh or ep_mesh must be provided")

        # Parallelize the root submodules with TP.
        if tp_mesh is not None:
            if use_seq_parallel:
                root_plan = {
                    "tok_embeddings": RowwiseParallel(
                        input_layouts=Replicate(), output_layouts=Shard(1)
                    ),
                    "pos_embeddings": RowwiseParallel(
                        input_layouts=Replicate(), output_layouts=Shard(0)
                    ),
                    "norm": SequenceParallel(),
                }
            else:
                root_plan = {
                    "tok_embeddings": RowwiseParallel(
                        input_layouts=Replicate(), output_layouts=Replicate()
                    ),
                    "pos_embeddings": RowwiseParallel(
                        input_layouts=Replicate(), output_layouts=Replicate()
                    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 526-549
```python
                }
            parallelize_module(module, tp_mesh, root_plan)

        # Parallelize the attention and feed forward submodules.
        for layer in module.layers:
            if tp_mesh is not None:
                layer_parallelize_plan = {}
                if use_seq_parallel:
                    layer_parallelize_plan["attention"] = PrepareModuleInput(
                        input_layouts=Shard(1),
                        desired_input_layouts=Replicate(),
                    )
                    # shard the RMSNorms
                    layer_parallelize_plan["attention_norm"] = SequenceParallel()
                    layer_parallelize_plan["ffn_norm"] = SequenceParallel()
                layer_parallelize_plan["attention.wq"] = ColwiseParallel(
                    use_local_output=local_output_for_attn
                )
                layer_parallelize_plan["attention.wk"] = ColwiseParallel(
                    use_local_output=local_output_for_attn
                )
                layer_parallelize_plan["attention.wv"] = ColwiseParallel(
                    use_local_output=local_output_for_attn
                )
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 550-564
```python
                layer_parallelize_plan["attention.wo"] = (
                    RowwiseParallel(output_layouts=Shard(1))
                    if use_seq_parallel
                    else RowwiseParallel()
                )

                if not layer.has_experts:
                    layer_parallelize_plan["feed_forward.w1"] = (
                        ColwiseParallel(input_layouts=Shard(1))
                        if use_seq_parallel
                        else ColwiseParallel()
                    )
                    layer_parallelize_plan["feed_forward.w2"] = (
                        RowwiseParallel(output_layouts=Shard(1))
                        if use_seq_parallel
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 565-577
```python
                        else RowwiseParallel()
                    )
                elif ep_mesh is None:
                    # No EP mesh provided, use TP for experts
                    layer_parallelize_plan["expert_layer.experts"] = (
                        TensorParallelForExpert(
                            input_layouts=Shard(0),
                            output_layouts=Shard(0),
                        )
                        if use_seq_parallel
                        else TensorParallelForExpert()
                    )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 578-592
```python
                parallelize_module(layer, tp_mesh, layer_parallelize_plan)

            # EP (+ optional TP) for experts
            if ep_mesh is not None and layer.has_experts:
                if tp_mesh is not None:
                    parallelize_module(
                        layer.expert_layer,
                        ep_mesh,
                        ExpertParallelWithTP(ep_mesh, tp_mesh),
                    )
                else:
                    parallelize_module(
                        layer.expert_layer.experts, ep_mesh, ExpertParallel()
                    )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 593-607
```python
        if tp_mesh is not None:
            # Parallelize the output submodule. If weight tying is enabled,
            # we need to make sure output.weight is sharded consistently as
            # tok_embeddings.weight, at the cost of the all_reduce operation
            # using RowwiseParallel.
            output_parallelize_plan = (
                ColwiseParallel(
                    input_layouts=Shard(1),
                    output_layouts=Replicate(),
                )
                if use_seq_parallel
                else ColwiseParallel(output_layouts=Replicate())
            )
            parallelize_module(module.output, tp_mesh, output_parallelize_plan)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 608-619
```python
            if local_output_for_attn:
                for layer in module.layers:
                    layer.attention.n_heads = (
                        module.model_args.n_heads // tp_mesh.size()
                    )

            # Manually set output.weight so that parameters and gradients
            # are shared.
            if module.model_args.weight_tying:
                module.output.weight = module.tok_embeddings.weight

        return module
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 620-632
```python


def skip_unless_torch_gpu(method: T) -> T:
    """
    Test decorator which skips the test unless there's a GPU available to torch.

    >>> # xdoctest: +SKIP
    >>> @skip_unless_torch_gpu
    >>> def test_some_method(self) -> None:
    >>>   ...
    """
    # The builtin @skip_if_no_gpu relies on os.environ['WORLD_SIZE'] being set.
    return cast(T, skip_if_lt_x_gpu(NUM_DEVICES)(method))
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `skip_unless_torch_gpu`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`skip_unless_torch_gpu`。

### Lines 633-644
```python


class DTensorTestMixin:
    """Shared test helpers for DTensorTestBase and DTensorContinuousTestBase."""

    @property
    def is_local_tensor_enabled(self) -> bool:
        return False

    @property
    def device_type(self) -> str:
        if (
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DTensorTestMixin`, `is_local_tensor_enabled`, `device_type`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DTensorTestMixin`, `is_local_tensor_enabled`, `device_type`。

### Lines 645-657
```python
            not (TEST_CUDA or TEST_XPU or TEST_HPU or TEST_PRIVATEUSE1)
            or DEVICE_COUNT < self.world_size
        ):
            return "cpu"
        else:
            return DEVICE_TYPE

    def build_device_mesh(self) -> DeviceMesh:
        return init_device_mesh(self.device_type, (self.world_size,))

    def init_manual_seed_for_rank(self) -> None:
        torch.manual_seed(self.rank)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `build_device_mesh`, `init_manual_seed_for_rank`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`build_device_mesh`, `init_manual_seed_for_rank`。

### Lines 658-673
```python
    def _test_op_on_dtensor(self, op_call, *args, **kwargs) -> None:
        """
        Checks ``op_call(dtensor).full_tensor() == op_call(dtensor.full_tensor())``.
        Unlike _test_op where the DTensor sharding is generated by DTensorConverter,
        this function takes in DTensor object directly as argument and test the equality
        of calling op on full_tensor() and DTensor.
        """
        args_flattened, args_spec = tree_flatten(args)
        full_tensor_args_flattened = tuple(
            arg.full_tensor().detach().clone() if isinstance(arg, DTensor) else arg
            for arg in args_flattened
        )
        full_tensor_args = tree_unflatten(full_tensor_args_flattened, args_spec)
        full_tensor_kwargs = {
            k: v.full_tensor() if isinstance(v, DTensor) else v
            for k, v in kwargs.items()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_test_op_on_dtensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_test_op_on_dtensor`。

### Lines 674-687
```python
        }

        out_flattened, _ = tree_flatten(
            op_call(*full_tensor_args, **full_tensor_kwargs)
        )
        d_out_flattened, _ = tree_flatten(op_call(*args, **kwargs))
        d_out_full_tensor_flattened = [dt.full_tensor() for dt in d_out_flattened]
        self.assertEqual(out_flattened, d_out_full_tensor_flattened)

    # pyre-ignore[2]:
    def _test_op(self, mesh: DeviceMesh, op_call, *args, **kwargs) -> None:
        out = op_call(*args, **kwargs)
        dtc = DTensorConverter(mesh, args, kwargs)
        for d_args, d_kwargs in dtc:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_test_op`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_test_op`。

### Lines 688-699
```python
            # pyre can't find assertTrue anymore?
            self.assertEqual(dtc.successful(), True)
            d_out = op_call(*d_args, **d_kwargs)
            self.assertEqual(d_out.full_tensor(), out)

    def run_subtests(self, *args, **kwargs):
        return run_subtests(self, *args, **kwargs)


class DTensorContinuousTestBase(DTensorTestMixin, MultiProcContinuousTest):
    @classmethod
    def backend_str(cls) -> str:
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `run_subtests`, `DTensorContinuousTestBase`, `backend_str`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`run_subtests`, `DTensorContinuousTestBase`, `backend_str`。

### Lines 700-713
```python
        backend = dist.get_default_backend_for_device(DEVICE_TYPE)
        return backend

    @classmethod
    def _init_pg(cls, rank, world_size, rdvz_file):
        # Set device before initializing process group to ensure
        # each rank is bound to the correct GPU. However, if world_size > device_count,
        # we skip the test.
        if torch.accelerator.is_available():
            if world_size > torch.accelerator.device_count():
                sys.exit(TEST_SKIPS[f"multi-gpu-{world_size}"].exit_code)
            else:
                torch.accelerator.set_device_index(rank)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_init_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_init_pg`。

### Lines 714-725
```python
        # Call parent's _init_pg to do the actual process group initialization
        super()._init_pg(rank, world_size, rdvz_file)


class LocalDTensorContinuousTestBase(DTensorContinuousTestBase):
    @property
    def is_local_tensor_enabled(self) -> bool:
        return True

    def _handle_test_skip(self, msg: str) -> None:
        self.skipTest(msg)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LocalDTensorContinuousTestBase`, `is_local_tensor_enabled`, `_handle_test_skip`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LocalDTensorContinuousTestBase`, `is_local_tensor_enabled`, `_handle_test_skip`。

### Lines 726-746
```python
    def _get_local_tensor_mode(self):
        return LocalTensorMode(frozenset(range(self.world_size)))

    @classmethod
    def _ensure_processes_spawned(cls):
        if cls._processes_spawned:
            return
        if cls.world_size == -2:
            cls.world_size = NUM_DEVICES
        store = FakeStore()
        dist.init_process_group(
            backend="fake",
            world_size=cls.world_size,
            rank=0,
            store=store,
        )
        cls.processes = []
        cls.task_queues = []
        cls.completion_queues = []
        cls._processes_spawned = True

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_local_tensor_mode`, `_ensure_processes_spawned`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_local_tensor_mode`, `_ensure_processes_spawned`。

### Lines 747-758
```python
    @classmethod
    def tearDownClass(cls):
        if cls._processes_spawned:
            dist.destroy_process_group()
            cls._processes_spawned = False
        unittest.TestCase.tearDownClass()

    def setUp(self):
        unittest.TestCase.setUp(self)
        self.__class__._ensure_processes_spawned()
        torch.autograd._enable_record_function(False)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tearDownClass`, `setUp`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tearDownClass`, `setUp`。

### Lines 759-770
```python
    def tearDown(self):
        from torch.distributed.tensor import _random as random

        random._rng_tracker = None
        unittest.TestCase.tearDown(self)
        torch.autograd._enable_record_function(True)

    def __init__(self, method_name="runTest", methodName="runTest"):
        if methodName != "runTest":
            method_name = methodName
        unittest.TestCase.__init__(self, method_name)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tearDown`, `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tearDown`, `__init__`。

### Lines 771-782
```python
    @property
    def rank(self):
        return torch.SymInt(LocalIntNode({r: r for r in range(self.world_size)}))

    @rank.setter
    def rank(self, rank):
        pass

    def build_device_mesh(self) -> DeviceMesh:
        with maybe_disable_local_tensor_mode():
            return super().build_device_mesh()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rank`, `build_device_mesh`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rank`, `build_device_mesh`。

### Lines 783-795
```python
    def init_manual_seed_for_rank(self) -> None:
        torch.manual_seed(0)


class DTensorTestBase(DTensorTestMixin, MultiProcessTestCase):
    @property
    def world_size(self) -> int:
        return NUM_DEVICES

    @property
    def backend(self) -> str:
        backend = dist.get_default_backend_for_device(self.device_type)
        return backend
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `init_manual_seed_for_rank`, `DTensorTestBase`, `world_size`, `backend`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`init_manual_seed_for_rank`, `DTensorTestBase`, `world_size`, `backend`。

### Lines 796-808
```python

    def init_pg(self, eager_init, backend: str | None = None) -> None:
        if backend is None:
            backend = self.backend

        requires_gpu = any(
            gpu_backend in backend for gpu_backend in ACCELERATOR_DIST_BACKENDS
        )
        if requires_gpu and torch.accelerator.device_count() < self.world_size:
            sys.exit(TEST_SKIPS[f"multi-gpu-{self.world_size}"].exit_code)

        curr_backend = dist.get_default_backend_for_device(self.device_type)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `init_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`init_pg`。

### Lines 809-822
```python
        if backend not in [
            "nccl",
            "gloo",
            "mpi",
            f"cpu:gloo,{self.device_type}:{curr_backend}",
            "cpu:gloo,cuda:ncclx",
            "cuda:ncclx",
            "hccl",
            "xccl",
            "fake",
            "cpu:gloo,xpu:xccl",
        ]:
            raise RuntimeError(f"Backend {backend} not supported!")

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 823-844
```python
        device_id = None
        if "nccl" in backend or "xccl" in backend:
            # set device for nccl pg for collectives
            # TODO: if users want to enable testing across hosts, we may need
            # to change this part.
            torch.accelerator.set_device_index(self.rank)
            # we only need to set device_id for nccl backend with eager init
            device_id = (
                torch.device(f"{self.device_type}:{self.rank}") if eager_init else None
            )

        # For nccl backend, bind the device to the process if device_id is not None
        # so the nccl communicator is immediately formed and we can use `ncclCommSplit`
        # for form subgroup to avoid unnecessary overhead.
        dist.init_process_group(
            backend=backend,
            world_size=self.world_size,
            rank=self.rank,  # pyre-ignore[16]
            init_method=f"file://{self.file_name}",  # pyre-ignore[16]
            device_id=device_id,
        )

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 845-856
```python
    def destroy_pg(self, device_id: int | None = None) -> None:
        # Wait for all ranks to reach here before starting shutdown.
        # FIXME dist.barrier deadlocks with multiple threads and NCCL: https://github.com/pytorch/pytorch/issues/95895
        # dist.all_reduce(torch.zeros((1,), device="cuda" if TEST_CUDA else "cpu"))
        # FIXME can't use the above all_reduce as it causes hangs on bionic and focal. It hangs:
        #  test_dtensor.py  -- DTensorMeshTest.test_dtensor_device_mesh_device_conversion
        if device_id is None:
            device_id = (
                torch.cuda.current_device() if self.device_type == "cuda" else self.rank
            )

        if self.device_type == "cpu":
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `destroy_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`destroy_pg`。

### Lines 857-868
```python
            # NOTE: when `device_id` is not None, barrier() will choose the accelerator
            # of the most pripority, which means if the test specifies to use CPU for
            # testing while CUDA is available on the host, the barrier() will use CUDA.
            # To avoid this and better respect `self.device_type`, we add this branch to
            # enforce barrier() to use CPU when `self.device_type` is CPU and other
            # accelerator is also available.
            dist.barrier()
        else:
            dist.barrier(device_ids=[device_id])

        dist.destroy_process_group()

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 869-882
```python
    def setUp(self) -> None:
        super().setUp()
        self._spawn_processes()


TestFunc = Callable[[...], object]


# wrapper to initialize comms (processgroup)
def with_comms(
    eager_init: TestFunc | bool = False,
    backend: str | None = None,
) -> TestFunc:
    def decorator(func, eager_init: bool = False, backend: str | None = None):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `setUp`, `with_comms`, `decorator`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`setUp`, `with_comms`, `decorator`。

### Lines 883-894
```python
        @wraps(func)  # pyre-ignore[6]
        def wrapper(
            self,
            *args: tuple[object],
            **kwargs: dict[str, Any],  # type: ignore[misc]
        ) -> None:
            # just passthrough if harness doesn't
            # support init_pg e.g., DTensorOpTestBase
            if not hasattr(self, "init_pg"):
                func(self, *args, **kwargs)
                return

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapper`。

### Lines 895-906
```python
            self.init_pg(eager_init, backend)

            try:
                func(self, *args, **kwargs)  # type: ignore[misc]
            except Exception as e:
                dist.destroy_process_group()
                raise e

            self.destroy_pg()

        return wrapper

```
- EN: This block bridges to backend-specific execution artifacts; reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 907-918
```python
    return (
        decorator(func=eager_init)
        if callable(eager_init)
        else partial(decorator, eager_init=eager_init, backend=backend)
    )


class DTensorOpTestBase(MultiThreadedTestCase):
    @property
    def world_size(self) -> int:
        return NUM_DEVICES

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DTensorOpTestBase`, `world_size`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DTensorOpTestBase`, `world_size`。

### Lines 919-930
```python
    @property
    def device_type(self) -> str:
        return DEVICE_TYPE

    def build_device_mesh(self):
        return init_device_mesh(self.device_type, (self.world_size,))

    def setUp(self) -> None:
        super().setUp()
        # Enable thread-safe lock for ShardingPropagator since we run
        # multi-threaded tests.
        from torch.distributed.tensor._sharding_prop import ShardingPropagator
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `device_type`, `build_device_mesh`, `setUp`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`device_type`, `build_device_mesh`, `setUp`。

### Lines 931-942
```python

        self._orig_fake_mode_lock = ShardingPropagator._fake_mode_lock
        ShardingPropagator._fake_mode_lock = threading.Lock()
        self._spawn_threads()

    def tearDown(self) -> None:
        # Restore the original (no-op) lock
        from torch.distributed.tensor._sharding_prop import ShardingPropagator

        ShardingPropagator._fake_mode_lock = self._orig_fake_mode_lock
        super().tearDown()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tearDown`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tearDown`。

### Lines 943-961
```python

# This is a class for converting args/kwargs of an op into distributed args/kwargs
class DTensorConverter:
    def __init__(
        self,
        mesh: DeviceMesh,
        args: tuple[object, ...],
        kwargs: dict[str, object],
        replicate_only: bool = False,
    ) -> None:
        self.hit = 0
        self.miss = 0
        self.mesh = mesh
        self.args = args
        self.kwargs = kwargs
        self.replicate_only = replicate_only
        flatten_args, flatten_args_spec = tree_flatten(args)
        flatten_kwargs, flatten_kwargs_spec = tree_flatten(kwargs)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DTensorConverter`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DTensorConverter`。

### Lines 962-975
```python
        self.flatten_args: list[object] = flatten_args
        self.flatten_args_spec: TreeSpec = flatten_args_spec
        self.flatten_kwargs: list[object] = flatten_kwargs
        self.flatten_kwargs_spec: TreeSpec = flatten_kwargs_spec

        choices_for_args = [
            self.gen_sharding_choices_for_arg(arg)
            for arg in self.flatten_args
            if isinstance(arg, torch.Tensor)
        ]

        choices_for_args.extend(
            self.gen_sharding_choices_for_arg(arg)
            for arg in self.flatten_kwargs
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 976-992
```python
            if isinstance(arg, torch.Tensor)
        )

        self.sharding_combs: Iterator[Sequence[Placement]] = iter(
            itertools.product(*choices_for_args)
        )

    def successful(self) -> bool:
        return self.hit > 0 and self.miss == 0

    def is_supported_tensor(self, t: torch.Tensor) -> bool:
        # TODO: dist tensor need to support quantized and sparse
        # tensors, quantized tensor might be relatively easy, but
        # sparse tensor have special layouts that we need to possibly
        # deal with, until we are clear about them, we don't officially
        # support them.
        return not any(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `successful`, `is_supported_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`successful`, `is_supported_tensor`。

### Lines 993-1008
```python
            [
                t.is_sparse_csr,
                t.is_sparse,
                t.is_mkldnn,
                t.is_quantized,
                t.is_nested,
                torch._is_functional_tensor(t),
                t.is_neg(),
                t.is_conj(),
                t.device.type in ("lazy", "meta"),
                # We need a way to test if a tensor is batched but there
                # is no official APi to do it
                # torch._C._is_batched(t),
            ]
        )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 1009-1023
```python
    def gen_sharding_choices_for_arg(self, arg: torch.Tensor) -> Sequence[Placement]:
        # If replicate_only is set, only use Replicate placement
        if self.replicate_only:
            return [Replicate()]

        mesh_size = self.mesh.size()
        sharding_choices: list[Placement] = [Replicate()]
        # c10d collective does not support bool tensor
        # for bool tensor we treat it as replicated
        if arg.dtype != torch.bool:
            # only generating choices with: replicate, or sharding
            # evenly on a dimension that could be sharded
            sharding_choices = sharding_choices + [
                Shard(i)
                for i, s in enumerate(arg.shape)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `gen_sharding_choices_for_arg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`gen_sharding_choices_for_arg`。

### Lines 1024-1035
```python
                if s > 1 and s % mesh_size == 0
            ]
        # TODO: add multi mesh choices
        # all_choices = itertools.product(
        #     *(self.mesh.ndim * [sharding_choices])
        # )
        return sharding_choices

    def __iter__(self) -> "DTensorConverter":
        return self

    def __next__(self) -> tuple[tuple[object, ...], dict[str, object]]:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__iter__`, `__next__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__iter__`, `__next__`。

### Lines 1036-1051
```python
        try:
            next_sharding_choices = next(self.sharding_combs)
            idx = 0

            new_args: list[object] = []
            for arg in self.flatten_args:
                if isinstance(arg, torch.Tensor):
                    new_args.append(
                        self.to_dist_tensor(
                            arg, self.mesh, [next_sharding_choices[idx]]
                        )
                    )
                    idx += 1
                else:
                    new_args.append(arg)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1052-1063
```python
            new_kwargs: list[object] = []
            for arg in self.flatten_kwargs:
                if isinstance(arg, torch.Tensor):
                    new_kwargs.append(
                        self.to_dist_tensor(
                            arg, self.mesh, [next_sharding_choices[idx]]
                        )
                    )
                    idx += 1
                else:
                    new_kwargs.append(arg)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1064-1075
```python
            return (
                tree_unflatten(new_args, self.flatten_args_spec),
                tree_unflatten(new_kwargs, self.flatten_kwargs_spec),
            )
        except StopIteration as e:
            raise StopIteration from e

    def to_dist_tensor(
        self, t: torch.Tensor, mesh: DeviceMesh, placements: list[Placement]
    ) -> torch.Tensor:
        if type(t) is torch.Tensor or type(t) is nn.Parameter or type(t) is LocalTensor:
            if self.is_supported_tensor(t):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `to_dist_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`to_dist_tensor`。

### Lines 1076-1087
```python
                self.hit += 1
                if t.ndim == 0:
                    # scalar tensor by default will be replicated
                    r = distribute_tensor(t, mesh, [Replicate()] * mesh.ndim)
                else:
                    # distribute non-scalar tensors
                    r = distribute_tensor(t, mesh, placements)
                if isinstance(t, nn.Parameter):
                    r = nn.Parameter(  # type: ignore[assignment]
                        r, requires_grad=r.requires_grad
                    )
                return r
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1088-1100
```python
            else:
                self.miss += 1
                return t
        elif torch.overrides.is_tensor_like(t):
            # Blindly converting tensor subclasses to dist tensor can cause
            # unpredictable problems, we explicitly disable this conversion
            # for now (i.e. we don't support DTensor holding tensor subclass
            # until there's a strong reason later).
            self.miss += 1
            return t
        else:
            raise RuntimeError(f"Trying to convert to DTensor, but got {type(t)}")

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1101-1112
```python

class LocalDTensorOpTestBase(DTensorOpTestBase):
    @property
    def is_local_tensor_enabled(self) -> bool:
        return True

    def _handle_test_skip(self, msg: str) -> None:
        self.skipTest(msg)

    def _get_local_tensor_mode(self):
        return LocalTensorMode(frozenset(range(self.world_size)))

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LocalDTensorOpTestBase`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LocalDTensorOpTestBase`。

### Lines 1113-1124
```python
    def setUp(self) -> None:
        super().setUp()
        torch.autograd._enable_record_function(False)

    def tearDown(self) -> None:
        from torch.distributed.tensor import _random as random

        random._rng_tracker = None
        super().tearDown()
        torch.autograd._enable_record_function(True)

    @property
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `setUp`, `tearDown`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`setUp`, `tearDown`。

### Lines 1125-1136
```python
    def rank(self):
        return torch.SymInt(LocalIntNode({r: r for r in range(self.world_size)}))

    @rank.setter
    def rank(self, rank):
        pass

    def join_or_run(self, fn):
        @wraps(fn)
        def wrapper(self):
            fn()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rank`, `join_or_run`, `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rank`, `join_or_run`, `wrapper`。

### Lines 1137-1150
```python
        return types.MethodType(wrapper, self)

    def build_device_mesh(self) -> DeviceMesh:
        with maybe_disable_local_tensor_mode():
            return super().build_device_mesh()

    def init_pg(self, eager_init, backend: str | None = None) -> None:
        dist.init_process_group("fake", rank=0, world_size=self.world_size)
        self._pg = dist.distributed_c10d._get_default_group()

    def destroy_pg(self, device_id: int | None = None) -> None:
        dist.destroy_process_group(self._pg)
        self._pg = None

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `build_device_mesh`, `init_pg`, `destroy_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`build_device_mesh`, `init_pg`, `destroy_pg`。

### Lines 1151-1162
```python
    def _spawn_processes(self) -> None:
        pass

    def _spawn_threads(self) -> None:
        pass

    def run_test(self, test_name: str, parent_pipe) -> None:
        getattr(self, test_name)()

    def init_manual_seed_for_rank(self) -> None:
        torch.manual_seed(0)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_spawn_processes`, `_spawn_threads`, `run_test`, `init_manual_seed_for_rank`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_spawn_processes`, `_spawn_threads`, `run_test`, `init_manual_seed_for_rank`。

### Lines 1163-1174
```python

class LocalDTensorTestBase(DTensorTestBase):
    @property
    def is_local_tensor_enabled(self) -> bool:
        return True

    def _handle_test_skip(self, msg: str) -> None:
        self.skipTest(msg)

    def _get_local_tensor_mode(self):
        return LocalTensorMode(frozenset(range(self.world_size)))

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `LocalDTensorTestBase`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`LocalDTensorTestBase`。

### Lines 1175-1186
```python
    def setUp(self) -> None:
        super().setUp()
        torch.autograd._enable_record_function(False)

    def tearDown(self) -> None:
        from torch.distributed.tensor import _random as random

        random._rng_tracker = None
        super().tearDown()
        torch.autograd._enable_record_function(True)

    @property
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `setUp`, `tearDown`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`setUp`, `tearDown`。

### Lines 1187-1198
```python
    def rank(self):
        return torch.SymInt(LocalIntNode({r: r for r in range(self.world_size)}))

    @rank.setter
    def rank(self, rank):
        pass

    def join_or_run(self, fn):
        @wraps(fn)
        def wrapper(self):
            fn()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `rank`, `join_or_run`, `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`rank`, `join_or_run`, `wrapper`。

### Lines 1199-1212
```python
        return types.MethodType(wrapper, self)

    def build_device_mesh(self) -> DeviceMesh:
        with maybe_disable_local_tensor_mode():
            return super().build_device_mesh()

    def init_pg(self, eager_init, backend: str | None = None) -> None:
        dist.init_process_group("fake", rank=0, world_size=self.world_size)
        self._pg = dist.distributed_c10d._get_default_group()

    def destroy_pg(self, device_id: int | None = None) -> None:
        dist.destroy_process_group(self._pg)
        self._pg = None

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `build_device_mesh`, `init_pg`, `destroy_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`build_device_mesh`, `init_pg`, `destroy_pg`。

### Lines 1213-1224
```python
    def _spawn_processes(self) -> None:
        pass

    def run_test(self, test_name: str, parent_pipe) -> None:
        getattr(self, test_name)()

    def init_manual_seed_for_rank(self) -> None:
        torch.manual_seed(0)


def make_wrapped(fn, ctxs):
    @functools.wraps(fn)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_spawn_processes`, `run_test`, `init_manual_seed_for_rank`, `make_wrapped`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_spawn_processes`, `run_test`, `init_manual_seed_for_rank`, `make_wrapped`。

### Lines 1225-1237
```python
    def wrapped(self):
        torch._dynamo.reset()
        stack = contextlib.ExitStack()
        for ctx in ctxs:
            if callable(ctx):
                stack.enter_context(ctx(self))
            else:
                stack.enter_context(ctx)
        try:
            out = fn(self)
        finally:
            stack.close()
        return out
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapped`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapped`。

### Lines 1238-1249
```python

    return wrapped


def create_local_tensor_test_class(
    orig_cls, skipped_tests=None, base_class=LocalDTensorTestBase
):
    if skipped_tests is None:
        skipped_tests = []

    dct = orig_cls.__dict__.copy()
    for name in list(dct.keys()):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `create_local_tensor_test_class`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`create_local_tensor_test_class`。

### Lines 1250-1267
```python
        fn = dct[name]
        if not callable(fn):
            continue
        elif name in skipped_tests:
            dct[name] = lambda self: self.skipTest("Skipped test")
        elif name.startswith("test_"):
            ctxs = [
                lambda test: test._get_local_tensor_mode(),
            ]
            dct[name] = make_wrapped(fn, ctxs)

    cls = type(
        orig_cls.__name__ + "WithLocalTensor",
        (base_class,) + orig_cls.__bases__,
        dct,
    )
    cls.__file__ = __file__
    return cls
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1268-1279
```python


@maybe_run_for_local_tensor
def map_local_tensor_for_rank(tensor, rank, func):
    return func(tensor, rank)


@maybe_run_for_local_tensor
def map_local_for_rank(rank, func):
    return func(rank)


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `map_local_tensor_for_rank`, `map_local_for_rank`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`map_local_tensor_for_rank`, `map_local_for_rank`。

### Lines 1280-1291
```python
def reduce_local_int(val, func):
    return func(val.node._local_ints)


def _convert_shard_order_dict_to_ShardOrder(shard_order):
    """Convert shard_order dict to ShardOrder"""
    return tuple(
        ShardOrderEntry(tensor_dim=tensor_dim, mesh_dims=tuple(mesh_dims))
        for tensor_dim, mesh_dims in shard_order.items()
    )


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reduce_local_int`, `_convert_shard_order_dict_to_ShardOrder`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reduce_local_int`, `_convert_shard_order_dict_to_ShardOrder`。

### Lines 1292-1304
```python
# TODO(zpcore): remove once the native redistribute supports shard_order arg
def redistribute(
    dtensor_input,
    device_mesh,
    placements,
    shard_order,
    use_graph_based_transform=True,
):
    """
    wrapper function to support shard_order for redistribution
    This is a simpler version of Redistribute, only considers the forward.
    """
    if placements is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `redistribute`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`redistribute`。

### Lines 1305-1326
```python
        placements = shard_order_to_placement(shard_order, device_mesh)
    placements = tuple(placements)
    old_spec = dtensor_input._spec
    new_spec = copy.deepcopy(old_spec)
    new_spec.placements = placements
    if shard_order is not None:
        new_spec.shard_order = shard_order
    else:
        new_spec.shard_order = ()
    if old_spec == new_spec:
        return dtensor_input
    dtensor_input = DTensor.from_local(
        redistribute_local_tensor(
            dtensor_input.to_local(),
            old_spec,
            new_spec,
            use_graph_based_transform=use_graph_based_transform,
        ),
        device_mesh,
    )
    dtensor_input._spec = copy.deepcopy(new_spec)
    return dtensor_input  # returns DTensor
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1327-1340
```python


# TODO(zpcore): remove once the native distribute_tensor supports
# shard_order arg
def patched_distribute_tensor(
    input_tensor,
    device_mesh,
    placements,
    shard_order,
    use_graph_based_transform=True,
    src_data_rank: int | None = 0,
):
    """wrapper function to support shard_order for tensor distribution"""
    if placements is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `patched_distribute_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`patched_distribute_tensor`。

### Lines 1341-1353
```python
        placements = shard_order_to_placement(shard_order, device_mesh)
    placements = tuple(placements)
    tensor_dt = distribute_tensor(
        input_tensor, device_mesh, placements, src_data_rank=src_data_rank
    )
    # Do not consider _StridedShard to express shard order
    tensor_dt._spec.use_strided_shard_as_shard_order = False
    tensor_dt._spec.__post_init__()
    # fix the shard order
    return redistribute(
        tensor_dt, device_mesh, placements, shard_order, use_graph_based_transform
    )

```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1354-1366
```python

# TODO(zpcore): remove once the native redistribute supports shard_order arg
def make_full_tensor(dtensor_input):
    """wrapper function to support DTensor.full_tensor"""
    return redistribute(
        dtensor_input, dtensor_input.device_mesh, placements=None, shard_order=()
    ).to_local()


def shard_order_to_placement(shard_order, mesh):
    """convert shard_order to placement with only Replicate() and Shard()"""
    placements: list[Any] = [Replicate() for _ in range(mesh.ndim)]
    if shard_order is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_full_tensor`, `shard_order_to_placement`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_full_tensor`, `shard_order_to_placement`。

### Lines 1367-1378
```python
        for entry in shard_order:
            tensor_dim = entry.tensor_dim
            mesh_dims = entry.mesh_dims
            for mesh_dim in mesh_dims:
                placements[mesh_dim] = Shard(tensor_dim)
    return tuple(placements)


def generate_shard_orders(mesh, tensor_rank):
    # Generate all possible sharding placement of tensor with rank
    # `tensor_rank` over mesh.
    def _split_list(lst: list, N: int):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `generate_shard_orders`, `_split_list`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`generate_shard_orders`, `_split_list`。

### Lines 1379-1393
```python
        def compositions(n: int, k: int):
            # yields lists of length k, positive ints summing to n
            for cuts in itertools.combinations(range(1, n), k - 1):
                # add 0 and n as sentinels, then take consecutive differences
                yield [b - a for a, b in itertools.pairwise((0, *cuts, n))]

        length = len(lst)
        for comp in compositions(length, N):
            result = []
            start = 0
            for size in comp:
                result.append(lst[start : start + size])
                start += size
            yield result

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `compositions`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`compositions`。

### Lines 1394-1409
```python
    all_mesh = list(range(mesh.ndim))
    all_device_order = list(itertools.permutations(all_mesh))
    for device_order in all_device_order:
        # split on device orders, and assign each device order segment to a tensor dim
        for num_split in range(1, mesh.ndim + 1):
            for splitted_list in _split_list(list(range(mesh.ndim)), num_split):
                for tensor_dims in itertools.combinations(
                    range(tensor_rank), len(splitted_list)
                ):
                    shard_order = {}
                    if len(tensor_dims) != len(splitted_list):
                        raise AssertionError(
                            f"Expected len(tensor_dims) == len(splitted_list), "
                            f"got {len(tensor_dims)} != {len(splitted_list)}"
                        )
                    for tensor_dim, mesh_dims in zip(tensor_dims, splitted_list):
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1410-1428
```python
                        shard_order[tensor_dim] = device_order[
                            mesh_dims[0] : mesh_dims[-1] + 1
                        ]
                    yield _convert_shard_order_dict_to_ShardOrder(shard_order)


def validate_sharding_rule_sample(
    op, full_args, full_kwargs, input_placements, output_placements, device_mesh
):
    from torch.utils import _pytree as pytree

    # Extract tensors from args in order, pair with placements
    full_tensors = [
        a for a in pytree.tree_leaves(full_args) if isinstance(a, torch.Tensor)
    ]
    full_tensors += [
        a for a in pytree.tree_leaves(full_kwargs) if isinstance(a, torch.Tensor)
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `validate_sharding_rule_sample`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`validate_sharding_rule_sample`。

### Lines 1429-1442
```python
    dtensors = [
        distribute_tensor(t, device_mesh, (p,))
        for t, p in zip(full_tensors, input_placements)
    ]

    # Build sharded args by replacing tensors with their sharded local versions
    dtensor_idx = 0

    def _to_local_shard(a):
        nonlocal dtensor_idx
        if isinstance(a, torch.Tensor):
            local = dtensors[dtensor_idx].to_local()
            dtensor_idx += 1
            return local
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_to_local_shard`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_to_local_shard`。

### Lines 1443-1459
```python
        return a

    local_args, local_kwargs = pytree.tree_map(
        _to_local_shard, (full_args, full_kwargs)
    )

    # run and compare
    ref_output = op(*full_args, **full_kwargs)
    local_output = op(*local_args, **local_kwargs)

    ref_tensors = [
        t for t in pytree.tree_leaves(ref_output) if isinstance(t, torch.Tensor)
    ]
    local_tensors = [
        t for t in pytree.tree_leaves(local_output) if isinstance(t, torch.Tensor)
    ]

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 1460-1471
```python
    for ref, local, plc in zip(ref_tensors, local_tensors, output_placements):
        dt = DTensor.from_local(local, device_mesh, (plc,))
        full = dt.redistribute(device_mesh, (Replicate(),)).to_local()
        if ref.shape != full.shape or not torch.allclose(
            ref, full, atol=1e-5, rtol=1e-5, equal_nan=True
        ):
            return False
    return True


@contextlib.contextmanager
def op_strategy_context(op_overload, strategy_func, schema_info=None):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `op_strategy_context`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`op_strategy_context`。

### Lines 1472-1483
```python
    """
    Context manager for setting and clearing op strategies.
    Args:
        op_overload: The operator overload to set or clear the strategy for.
        strategy_func: The strategy function to set for the operator overload.
        schema_info: Optional schema information for the operator overload.
    Yields:
        None
    """
    from torch.distributed.tensor._ops.utils import register_op_strategy
    from torch.distributed.tensor.debug import _clear_sharding_prop_cache

```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1484-1499
```python
    propagator = DTensor._op_dispatcher.sharding_propagator
    _origin_op_strategy_funcs = None
    _origin_op_strategy_schema = None
    try:
        # register the op strategy
        if op_overload in propagator.op_strategy_funcs:
            _origin_op_strategy_funcs = propagator.op_strategy_funcs[op_overload]
            del propagator.op_strategy_funcs[op_overload]
        if op_overload in propagator.op_to_schema_info:
            _origin_op_strategy_schema = propagator.op_to_schema_info[op_overload]
            del propagator.op_to_schema_info[op_overload]
        register_op_strategy(op_overload, schema_info=schema_info)(strategy_func)
        yield
    finally:
        # clear this op strategy cache
        if _origin_op_strategy_funcs is None:
```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 1500-1509
```python
            if op_overload in propagator.op_strategy_funcs:
                del propagator.op_strategy_funcs[op_overload]
        else:
            propagator.op_strategy_funcs[op_overload] = _origin_op_strategy_funcs
        if _origin_op_strategy_schema is None:
            if op_overload in propagator.op_to_schema_info:
                del propagator.op_to_schema_info[op_overload]
        else:
            propagator.op_to_schema_info[op_overload] = _origin_op_strategy_schema
        _clear_sharding_prop_cache()
```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.distributed`, `torch.nn`, `torch.nn.functional`, `torch.distributed._functional_collectives`, `torch.distributed._local_tensor`, `torch.distributed.tensor`, `torch.distributed.tensor._dtensor_spec`, `torch.distributed.tensor._redistribute`, `torch.distributed.tensor.parallel`, `...`
- External imports / 外部导入: `contextlib`, `copy`, `functools`, `itertools`, `sys`, `threading`, `types`, `unittest`, `collections.abc`, `dataclasses`, `...`
- Representative symbols / 代表性符号: `T`, `RMSNormPython`, `MLPModule`, `MLPStacked`, `ModelArgs`, `Attention`, `FeedForward`, `Experts`, `ExpertLayer`, `TensorParallelForExpert`, `...`
