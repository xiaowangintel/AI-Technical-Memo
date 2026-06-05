# turbo_layer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/turbo_layer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `_SeqAllToAll`, `_SeqAllToAllQKV`, and `DistributedAttention`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `_SeqAllToAll`、`_SeqAllToAllQKV` 和 `DistributedAttention` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-26: module setup and imports / 模块初始化与导入
```python
from typing import Any, Callable, List, Tuple, Type, Union

import torch
import torch.distributed as dist
from einops import rearrange
from torch import Tensor
from torch.distributed import ProcessGroup
from torch.nn import Module

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionImpl,
)
from sglang.multimodal_gen.runtime.layers.attention.backends.sparse_linear_attn import (
    SageSparseLinearAttentionBackend,
    SparseLinearAttentionBackend,
)
from sglang.multimodal_gen.runtime.layers.attention.selector import get_attn_backend
from sglang.multimodal_gen.runtime.managers.forward_context import (
    ForwardContext,
    get_forward_context,
)
from sglang.multimodal_gen.runtime.platforms.interface import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import get_compute_dtype
```
**EN:** This block establishes the module context and imports `typing`, `torch`, `torch.distributed`, `einops`, `torch.nn`, and `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch`、`torch.distributed`、`einops`、`torch.nn` 和 `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`。这些依赖为后续实现提供所需符号。

### Lines 28-28: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 31-41: `post_all2all` implementation / `post_all2all` 实现
```python
def post_all2all(local_seq_2_local_head, seq_world_size):
    def post_func(input):
        # b, s, n, h
        if local_seq_2_local_head:
            output = rearrange(input, "w bs seq h d -> bs (w seq) h d")
        else:
            output = rearrange(input, "w bs s h d -> bs s (w h) d", w=seq_world_size)

        return output

    return post_func
```
**EN:** This block defines function `post_all2all`. It post-processes all2all. Key calls include `rearrange`. The implementation branches on conditions. Parameters such as `local_seq_2_local_head`, and `seq_world_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `post_all2all`。 它用于后处理all2all。 关键调用包括 `rearrange`。 实现中包含条件分支。 本段逻辑主要由 `local_seq_2_local_head` 和 `seq_world_size` 等参数驱动。

### Lines 44-74: `single_all_to_all` implementation / `single_all_to_all` 实现
```python
def single_all_to_all(input, local_seq_2_local_head, group, async_op=False):
    seq_world_size = dist.get_world_size(group)

    # b, s, n, h
    if local_seq_2_local_head:
        bs, local_seq_len, num_total_head, head_dim = input.shape
        assert (
            num_total_head % seq_world_size == 0
        ), f"Number of heads ({num_total_head}) must be divisible by the sequence parallel size ({seq_world_size})!"
        input_t = rearrange(
            input,
            "bs seq_len (w h) d -> w bs seq_len h d",
            w=seq_world_size,
            h=num_total_head // seq_world_size,
        ).contiguous()
        post_all2all_fun = post_all2all(local_seq_2_local_head, seq_world_size)
    else:
        bs, global_seq_len, num_local_head, head_dim = input.shape
        input_t = rearrange(
            input,
            "bs (w s) h d -> w bs s h d",
            w=seq_world_size,
            s=global_seq_len // seq_world_size,
        ).contiguous()
        post_all2all_fun = post_all2all(local_seq_2_local_head, seq_world_size)

    output = torch.empty_like(input_t)
    dist.all_to_all_single(output, input_t, group=group, async_op=async_op)

    res = post_all2all_fun(output)
    return res
```
**EN:** This block defines function `single_all_to_all`. It handles single all to all logic. Key calls include `dist.get_world_size`, `torch.empty_like`, `dist.all_to_all_single`, `post_all2all_fun`, and `rearrange.contiguous`. The implementation branches on conditions. Parameters such as `input`, `local_seq_2_local_head`, `group`, and `async_op` drive the behavior in this section.
**CN:** 该代码块定义了函数 `single_all_to_all`。 它用于处理 single all to all 相关逻辑。 关键调用包括 `dist.get_world_size`、`torch.empty_like`、`dist.all_to_all_single`、`post_all2all_fun` 和 `rearrange.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `input`、`local_seq_2_local_head`、`group` 和 `async_op` 等参数驱动。

### Lines 77-124: `async_a2a_communicate` implementation / `async_a2a_communicate` 实现
```python
def async_a2a_communicate(
    a2a_inputs: Union[torch.Tensor, List[torch.Tensor]],
    cp_size: int,
    cp_group: ProcessGroup,
    cp_stream: torch.get_device_module().Stream,
    local_seq_2_local_head: bool,
) -> Union[torch.Tensor, List[torch.Tensor]]:
    """
    A2A communication for context parallelism. best used in communicate qkv
    Modified from Nvidia Transformer Engine.
    """
    a2a_inputs = [a2a_inputs] if not isinstance(a2a_inputs, list) else a2a_inputs
    a2a_outputs, a2a_reqs = [None] * len(a2a_inputs), [None] * len(a2a_inputs)
    a2a_post_fns = [None] * len(a2a_inputs)
    if local_seq_2_local_head:
        for i in range(len(a2a_inputs) + 2):
            if 0 < i < len(a2a_inputs) + 1:
                a2a_outputs[i - 1] = torch.empty_like(a2a_inputs[i - 1])
                a2a_reqs[i - 1] = torch.distributed.all_to_all_single(
                    a2a_outputs[i - 1], a2a_inputs[i - 1], group=cp_group, async_op=True
                )
                a2a_post_fns[i - 1] = post_all2all(local_seq_2_local_head, cp_size)
            if i > 1:
                with torch.get_device_module().stream(cp_stream):
                    a2a_reqs[i - 2].wait()
                    a2a_outputs[i - 2] = a2a_post_fns[i - 2](a2a_outputs[i - 2])
            if i < len(a2a_inputs):
                a2a_inputs[i] = rearrange(
                    a2a_inputs[i], "bs seq_len (w h) d -> w bs seq_len h d", w=cp_size
                ).contiguous()
    else:
        for i in range(len(a2a_inputs) + 2):
            if 0 < i < len(a2a_inputs) + 1:
                a2a_outputs[i - 1] = torch.empty_like(a2a_inputs[i - 1])
                a2a_reqs[i - 1] = torch.distributed.all_to_all_single(
                    a2a_outputs[i - 1], a2a_inputs[i - 1], group=cp_group, async_op=True
                )
                a2a_post_fns[i - 1] = post_all2all(local_seq_2_local_head, cp_size)
            if i < len(a2a_inputs):
                a2a_inputs[i] = rearrange(
                    a2a_inputs[i], "bs (w s) h d -> w bs s h d", w=cp_size
                ).contiguous()
            if i > 1:
                with torch.get_device_module().stream(cp_stream):
                    a2a_reqs[i - 2].wait()
                    a2a_outputs[i - 2] = a2a_post_fns[i - 2](a2a_outputs[i - 2])
    torch.get_device_module().current_stream().wait_stream(cp_stream)
    return a2a_outputs[0] if len(a2a_inputs) == 1 else a2a_outputs
```
**EN:** This block defines function `async_a2a_communicate`. A2A communication for context parallelism. best used in communicate qkv Modified from Nvidia Transformer Engine. Key calls include `torch.get_device_module.current_stream.wait_stream`, `len`, `range`, `torch.get_device_module`, and `isinstance`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `a2a_inputs`, `cp_size`, `cp_group`, `cp_stream`, and `local_seq_2_local_head` drive the behavior in this section.
**CN:** 该代码块定义了函数 `async_a2a_communicate`。 它用于处理 async a2a communicate 相关逻辑。 关键调用包括 `torch.get_device_module.current_stream.wait_stream`、`len`、`range`、`torch.get_device_module` 和 `isinstance`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `a2a_inputs`、`cp_size`、`cp_group`、`cp_stream` 和 `local_seq_2_local_head` 等参数驱动。

### Lines 127-127: `_SeqAllToAll` class overview / `_SeqAllToAll` 类概览
```python
class _SeqAllToAll(torch.autograd.Function):
```
**EN:** This block defines class `_SeqAllToAll`. It encapsulates seq all to all behavior. It inherits from `torch.autograd.Function`.
**CN:** 该代码块定义了类 `_SeqAllToAll`。 它用于封装 seq all to all 相关行为。 它继承自 `torch.autograd.Function`。

### Lines 128-135: `forward` implementation / `forward` 实现
```python
    @staticmethod
    def forward(
        ctx: Any, group: dist.ProcessGroup, input: Tensor, local_seq_2_local_head: bool
    ) -> Tensor:
        ctx.group = group
        res = single_all_to_all(input, local_seq_2_local_head, group, False)
        ctx.local_seq_2_local_head = local_seq_2_local_head
        return res
```
**EN:** This block defines method `forward` on `_SeqAllToAll`. It executes function. Key calls include `single_all_to_all`. Parameters such as `ctx`, `group`, `input`, and `local_seq_2_local_head` drive the behavior in this section.
**CN:** 该代码块定义了 `_SeqAllToAll` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `single_all_to_all`。 本段逻辑主要由 `ctx`、`group`、`input` 和 `local_seq_2_local_head` 等参数驱动。

### Lines 137-143: `backward` implementation / `backward` 实现
```python
    @staticmethod
    def backward(ctx: Any, *grad_output: Tensor) -> Tuple[None, Tensor, None]:
        return (
            None,
            _SeqAllToAll.apply(ctx.group, *grad_output, not ctx.local_seq_2_local_head),
            None,
        )
```
**EN:** This block defines method `backward` on `_SeqAllToAll`. It computes function. Key calls include `_SeqAllToAll.apply`. Parameters such as `ctx` drive the behavior in this section.
**CN:** 该代码块定义了 `_SeqAllToAll` 的方法 `backward`。 它用于执行反向计算函数。 关键调用包括 `_SeqAllToAll.apply`。 本段逻辑主要由 `ctx` 等参数驱动。

### Lines 146-146: `_SeqAllToAllQKV` class overview / `_SeqAllToAllQKV` 类概览
```python
class _SeqAllToAllQKV(torch.autograd.Function):
```
**EN:** This block defines class `_SeqAllToAllQKV`. It encapsulates seq all to all qkv behavior. It inherits from `torch.autograd.Function`.
**CN:** 该代码块定义了类 `_SeqAllToAllQKV`。 它用于封装 seq all to all qkv 相关行为。 它继承自 `torch.autograd.Function`。

### Lines 147-165: `forward` implementation / `forward` 实现
```python
    @staticmethod
    def forward(
        ctx: Any,
        group: dist.ProcessGroup,
        q: Tensor,
        k: Tensor,
        v: Tensor,
        cp_size: int,
        cp_stream: torch.get_device_module().Stream,
        local_seq_2_local_head: bool,
    ) -> Tuple[Tensor, Tensor, Tensor]:
        ctx.group = group
        ctx.cp_size = cp_size
        ctx.cp_stream = cp_stream
        ctx.local_seq_2_local_head = local_seq_2_local_head
        q, k, v = async_a2a_communicate(
            [q, k, v], cp_size, group, cp_stream, local_seq_2_local_head
        )
        return q, k, v
```
**EN:** This block defines method `forward` on `_SeqAllToAllQKV`. It executes function. Key calls include `async_a2a_communicate`, and `torch.get_device_module`. Parameters such as `ctx`, `group`, `q`, `k`, and `v` drive the behavior in this section.
**CN:** 该代码块定义了 `_SeqAllToAllQKV` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `async_a2a_communicate` 和 `torch.get_device_module`。 本段逻辑主要由 `ctx`、`group`、`q`、`k` 和 `v` 等参数驱动。

### Lines 167-178: `backward` implementation / `backward` 实现
```python
    @staticmethod
    def backward(
        ctx: Any, *grad_output: Tensor
    ) -> Tuple[None, Tensor, Tensor, Tensor, None, None, None]:
        q_grad, k_grad, v_grad = _SeqAllToAllQKV.apply(
            ctx.group,
            *grad_output,
            ctx.cp_size,
            ctx.cp_stream,
            not ctx.local_seq_2_local_head,
        )
        return (None, q_grad, k_grad, v_grad, None, None, None)
```
**EN:** This block defines method `backward` on `_SeqAllToAllQKV`. It computes function. Key calls include `_SeqAllToAllQKV.apply`. Parameters such as `ctx` drive the behavior in this section.
**CN:** 该代码块定义了 `_SeqAllToAllQKV` 的方法 `backward`。 它用于执行反向计算函数。 关键调用包括 `_SeqAllToAllQKV.apply`。 本段逻辑主要由 `ctx` 等参数驱动。

### Lines 181-188: `DistributedAttention` class overview / `DistributedAttention` 类概览
```python
class DistributedAttention(torch.nn.Module):
    """Initialization.

    Arguments:
        local_attention (Module): local attention with q,k,v
        sequence_process_group (ProcessGroup): sequence parallel process group
    """
```
**EN:** This block defines class `DistributedAttention`. Initialization. Arguments: local_attention (Module): local attention with q,k,v sequence_process_group (ProcessGroup): sequence parallel process group It inherits from `torch.nn.Module`.
**CN:** 该代码块定义了类 `DistributedAttention`。 它用于封装 distributed attention 相关行为。 它继承自 `torch.nn.Module`。

### Lines 189-193: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, local_attention: Union[Module, Callable]) -> None:
        super(DistributedAttention, self).__init__()
        self.local_attn = local_attention
        self.pg = None
        self.stream = None
```
**EN:** This block defines method `__init__` on `DistributedAttention`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `local_attention` drive the behavior in this section.
**CN:** 该代码块定义了 `DistributedAttention` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `local_attention` 等参数驱动。

### Lines 195-222: `forward` implementation / `forward` 实现
```python
    def forward(
        self, query: Tensor, key: Tensor, value: Tensor, ctx_attn_metadata
    ) -> Tensor:
        """forward

        Arguments:
            query (Tensor): query input to the layer
            key (Tensor): key input to the layer
            value (Tensor): value input to the layer

        Returns:
            * output (Tensor): context output
        """
        if self.pg is None:
            return self.local_attn(query, key, value, ctx_attn_metadata)
        pg_size = dist.get_world_size(self.pg)
        if pg_size < 2:
            return self.local_attn(query, key, value, ctx_attn_metadata)

        query_layer, key_layer, value_layer = _SeqAllToAllQKV.apply(
            self.pg, query, key, value, pg_size, self.stream, True
        )
        context_layer = self.local_attn(
            query_layer, key_layer, value_layer, ctx_attn_metadata
        )

        output = _SeqAllToAll.apply(self.pg, context_layer, False)
        return output
```
**EN:** This block defines method `forward` on `DistributedAttention`. forward Arguments: query (Tensor): query input to the layer key (Tensor): key input to the layer value (Tensor): value input to the layer Returns: * output (Tensor): context output Key calls include `dist.get_world_size`, `_SeqAllToAllQKV.apply`, `self.local_attn`, and `_SeqAllToAll.apply`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `ctx_attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `DistributedAttention` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `dist.get_world_size`、`_SeqAllToAllQKV.apply`、`self.local_attn` 和 `_SeqAllToAll.apply`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `ctx_attn_metadata` 等参数驱动。

### Lines 224-226: `set_context_parallel_group` implementation / `set_context_parallel_group` 实现
```python
    def set_context_parallel_group(self, group, stream):
        self.pg = group
        self.stream = stream
```
**EN:** This block defines method `set_context_parallel_group` on `DistributedAttention`. It configures context parallel group. Parameters such as `group`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `DistributedAttention` 的方法 `set_context_parallel_group`。 它用于设置context parallel group。 本段逻辑主要由 `group` 和 `stream` 等参数驱动。

### Lines 229-229: `MinimalA2AAttnOp` class overview / `MinimalA2AAttnOp` 类概览
```python
class MinimalA2AAttnOp(DistributedAttention):
```
**EN:** This block defines class `MinimalA2AAttnOp`. It encapsulates minimal a2 aattn op behavior. It inherits from `DistributedAttention`.
**CN:** 该代码块定义了类 `MinimalA2AAttnOp`。 它用于封装 minimal a2 aattn op 相关行为。 它继承自 `DistributedAttention`。

### Lines 230-262: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        attention_type: str,
        topk: float,
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        prefix: str = "",
    ):
        dtype = get_compute_dtype()
        attn_backend = get_attn_backend(
            head_size, dtype, supported_attention_backends=supported_attention_backends
        )
        # Maintained for compatibility purposes; can be removed when CI allows setting Attention_backend or when TurboWan supports FA.
        if attn_backend not in (
            SparseLinearAttentionBackend,
            SageSparseLinearAttentionBackend,
        ):
            logger.warning_once(
                "TurboWan now only supports `sla_attn` or `sage_sla_attn` and has been automatically set to attention_type. Please set --attention-backend to `sla_attn` or `sage_sla_attn`."
            )
            if attention_type == "sagesla":
                attn_backend = SageSparseLinearAttentionBackend
            else:
                attn_backend = SparseLinearAttentionBackend
        impl_cls: Type["AttentionImpl"] = attn_backend.get_impl_cls()
        local_attn = impl_cls(
            num_heads=num_heads,
            head_size=head_size,
            topk_ratio=topk,
            prefix=f"{prefix}.impl",
        )
        super(MinimalA2AAttnOp, self).__init__(local_attn)
```
**EN:** This block defines method `__init__` on `MinimalA2AAttnOp`. It initializes the instance state. Key calls include `get_compute_dtype`, `get_attn_backend`, `attn_backend.get_impl_cls`, `impl_cls`, and `super.__init__`. The implementation branches on conditions. Parameters such as `num_heads`, `head_size`, `attention_type`, `topk`, and `supported_attention_backends` drive the behavior in this section.
**CN:** 该代码块定义了 `MinimalA2AAttnOp` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `get_compute_dtype`、`get_attn_backend`、`attn_backend.get_impl_cls`、`impl_cls` 和 `super.__init__`。 实现中包含条件分支。 本段逻辑主要由 `num_heads`、`head_size`、`attention_type`、`topk` 和 `supported_attention_backends` 等参数驱动。

### Lines 264-266: `set_context_parallel_group` implementation / `set_context_parallel_group` 实现
```python
    def set_context_parallel_group(self, process_group, ranks, stream):
        del ranks
        super().set_context_parallel_group(process_group, stream)
```
**EN:** This block defines method `set_context_parallel_group` on `MinimalA2AAttnOp`. It configures context parallel group. Key calls include `super.set_context_parallel_group`, and `super`. Parameters such as `process_group`, `ranks`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `MinimalA2AAttnOp` 的方法 `set_context_parallel_group`。 它用于设置context parallel group。 关键调用包括 `super.set_context_parallel_group` 和 `super`。 本段逻辑主要由 `process_group`、`ranks` 和 `stream` 等参数驱动。

### Lines 268-274: `forward` implementation / `forward` 实现
```python
    def forward(
        self, query: Tensor, key: Tensor, value: Tensor, *args: Any, **kwargs
    ) -> Tensor:
        forward_context: ForwardContext = get_forward_context()
        ctx_attn_metadata = forward_context.attn_metadata
        results = super().forward(query, key, value, ctx_attn_metadata)
        return rearrange(results, "b ... h l -> b ... (h l)")
```
**EN:** This block defines method `forward` on `MinimalA2AAttnOp`. It executes function. Key calls include `get_forward_context`, `super.forward`, `rearrange`, and `super`. Parameters such as `query`, `key`, and `value` drive the behavior in this section.
**CN:** 该代码块定义了 `MinimalA2AAttnOp` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `get_forward_context`、`super.forward`、`rearrange` 和 `super`。 本段逻辑主要由 `query`、`key` 和 `value` 等参数驱动。

## Key Concepts / 关键概念
- `post_all2all`: Top-level function that post-processes all2all. / 顶层函数，用于后处理all2all。
- `single_all_to_all`: Top-level function that handles single all to all logic. / 顶层函数，用于处理 single all to all 相关逻辑。
- `async_a2a_communicate`: A2A communication for context parallelism. / 顶层函数，用于处理 async a2a communicate 相关逻辑。
- `_SeqAllToAll`: Primary class that encapsulates seq all to all behavior. / 核心类，用于封装 seq all to all 相关行为。
- `_SeqAllToAllQKV`: Primary class that encapsulates seq all to all qkv behavior. / 核心类，用于封装 seq all to all qkv 相关行为。
- `DistributedAttention`: Initialization. / 核心类，用于封装 distributed attention 相关行为。
- `MinimalA2AAttnOp`: Primary class that encapsulates minimal a2 aattn op behavior. / 核心类，用于封装 minimal a2 aattn op 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `einops`, `torch.nn`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.layers.attention.backends.sparse_linear_attn`, `sglang.multimodal_gen.runtime.layers.attention.selector`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.platforms.interface`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 274
