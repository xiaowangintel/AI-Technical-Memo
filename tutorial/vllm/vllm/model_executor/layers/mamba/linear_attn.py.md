# linear_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/linear_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MiniMaxText01RMSNormTP`, `MiniMaxText01LinearKernel`, `MiniMaxText01LinearAttention` for Mamba/state-space layers and kernels. / 实现 `MiniMaxText01RMSNormTP`, `MiniMaxText01LinearKernel`, `MiniMaxText01LinearAttention`，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-35)
```python
import math
from collections.abc import Callable
from functools import partial

import torch
import torch.nn.functional as F
from einops import rearrange
from torch import nn

from vllm.config import CacheConfig, ModelConfig, get_current_vllm_config
from vllm.distributed.communication_op import tensor_model_parallel_all_reduce
from vllm.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.lightning_attn import (
    lightning_attention,
    linear_decode_forward_triton,
)
from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear
from vllm.model_executor.layers.mamba.abstract import MambaBase
from vllm.model_executor.layers.mamba.mamba_utils import (
    MambaStateDtypeCalculator,
    MambaStateShapeCalculator,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.utils.torch_utils import direct_register_custom_op
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.linear_attn import LinearAttentionMetadata
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum
```
**EN:** This opening block pulls in external dependencies such as `math`, `collections`, `functools`, `torch`, `einops` and internal modules such as `vllm.config`, `vllm.distributed.communication_op`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.lightning_attn`. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `math`, `collections`, `functools`, `torch`, `einops`）以及内部模块（如 `vllm.config`, `vllm.distributed.communication_op`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.lightning_attn`）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `clear_linear_attention_cache_for_new_sequences` (lines 124-143)
```python
def clear_linear_attention_cache_for_new_sequences(
    kv_cache: torch.Tensor,
    state_indices_tensor: torch.Tensor,
    attn_metadata: LinearAttentionMetadata,
) -> None:
    num_prefills = getattr(attn_metadata, "num_prefills", 0)
    if num_prefills <= 0:
        return

    num_decode_tokens = getattr(attn_metadata, "num_decode_tokens", 0)
    for prefill_idx in range(num_prefills):
        q_start = attn_metadata.query_start_loc[num_decode_tokens + prefill_idx]
        q_end = attn_metadata.query_start_loc[num_decode_tokens + prefill_idx + 1]
        query_len = q_end - q_start
        context_len = (
            attn_metadata.seq_lens[num_decode_tokens + prefill_idx] - query_len
        )
        if context_len == 0:
            block_to_clear = state_indices_tensor[num_decode_tokens + prefill_idx]
            kv_cache[block_to_clear, ...] = 0
```
**EN:** Defines function `clear_linear_attention_cache_for_new_sequences` with signature `clear_linear_attention_cache_for_new_sequences(kv_cache: torch.Tensor, state_indices_tensor: torch.Tensor, attn_metadata: LinearAttentionMetadata) -> None`. It mainly works with `kv_cache`, `state_indices_tensor`, `attn_metadata`; implements one step of the Mamba/SSM execution path. The body uses branching, iteration. Key calls include `getattr`, `range`.
**CN:** 定义函数 `clear_linear_attention_cache_for_new_sequences`，其签名为 `clear_linear_attention_cache_for_new_sequences(kv_cache: torch.Tensor, state_indices_tensor: torch.Tensor, attn_metadata: LinearAttentionMetadata) -> None`。它主要围绕 `kv_cache`, `state_indices_tensor`, `attn_metadata` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `getattr`, `range`。

### Function `linear_attention_decode` (lines 146-165)
```python
def linear_attention_decode(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    kv_cache: torch.Tensor,
    slope_rate: torch.Tensor,
    state_indices_tensor: torch.Tensor,
    q_start: int = 0,
    q_end: int | None = None,
    slot_start: int = 0,
    slot_end: int | None = None,
    block_size: int = 32,
) -> torch.Tensor:
    q = q[q_start:q_end].unsqueeze(2).contiguous()
    k = k[q_start:q_end].unsqueeze(2).contiguous()
    v = v[q_start:q_end].unsqueeze(2).contiguous()
    slot_id = state_indices_tensor[slot_start:slot_end]
    return linear_decode_forward_triton(
        q, k, v, kv_cache, slope_rate, slot_id, block_size
    )
```
**EN:** Defines function `linear_attention_decode` with signature `linear_attention_decode(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_cache: torch.Tensor, slope_rate: torch.Tensor, state_indices_tensor: torch.Tensor, q_start: int=0, q_end: int | None=None, slot_start: int=0, slot_end: int | None=None, block_size: int=32) -> torch.Tensor`. It mainly works with `q`, `k`, `v`, `kv_cache`, `slope_rate`, `state_indices_tensor`, `q_start`, `q_end`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `q.unsqueeze.contiguous`, `k.unsqueeze.contiguous`, `v.unsqueeze.contiguous`, `linear_decode_forward_triton`, `q.unsqueeze`, `k.unsqueeze`.
**CN:** 定义函数 `linear_attention_decode`，其签名为 `linear_attention_decode(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_cache: torch.Tensor, slope_rate: torch.Tensor, state_indices_tensor: torch.Tensor, q_start: int=0, q_end: int | None=None, slot_start: int=0, slot_end: int | None=None, block_size: int=32) -> torch.Tensor`。它主要围绕 `q`, `k`, `v`, `kv_cache`, `slope_rate`, `state_indices_tensor`, `q_start`, `q_end` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `q.unsqueeze.contiguous`, `k.unsqueeze.contiguous`, `v.unsqueeze.contiguous`, `linear_decode_forward_triton`, `q.unsqueeze`, `k.unsqueeze`。

### Function `linear_attention_prefill_and_mix` (lines 168-216)
```python
def linear_attention_prefill_and_mix(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    kv_cache: torch.Tensor,
    state_indices_tensor: torch.Tensor,
    attn_metadata: LinearAttentionMetadata,
    slope_rate: torch.Tensor,
    block_size: int,
    decode_fn: Callable[..., torch.Tensor],
    prefix_fn: Callable[..., torch.Tensor],
    layer_idx: int | None = None,
) -> torch.Tensor:
    hidden = []
    for _prefill_idx in range(getattr(attn_metadata, "num_prefills", 0)):
        if _prefill_idx >= len(attn_metadata.query_start_loc):
            break
        if _prefill_idx >= len(state_indices_tensor):
            break
        offset = attn_metadata.num_decode_tokens
        _start = attn_metadata.query_start_loc[offset + _prefill_idx]
        _end = attn_metadata.query_start_loc[offset + _prefill_idx + 1]
        slot_id = state_indices_tensor[offset + _prefill_idx]
        qs = q[_start:_end].transpose(0, 1).contiguous()
        ks = k[_start:_end].transpose(0, 1).contiguous()
        vs = v[_start:_end].transpose(0, 1).contiguous()
        slice_layer_cache = kv_cache[slot_id, ...]
        out_slice = prefix_fn(
            qs,
            ks,
            vs,
            slice_layer_cache,
            slope_rate,
            block_size,
            layer_idx=layer_idx,
        )
        hidden.append(out_slice.contiguous())

    if attn_metadata.num_decode_tokens > 0:
        hidden_decode = decode_fn(
            q, k, v, kv_cache, state_indices_tensor, attn_metadata
        )
        hidden.insert(0, hidden_decode)

    if not hidden:
        return torch.empty((0, q.size(-1)), device=q.device, dtype=q.dtype)

    hidden = torch.concat(hidden, dim=0).contiguous()
    return hidden
```
**EN:** Defines function `linear_attention_prefill_and_mix` with signature `linear_attention_prefill_and_mix(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_cache: torch.Tensor, state_indices_tensor: torch.Tensor, attn_metadata: LinearAttentionMetadata, slope_rate: torch.Tensor, block_size: int, decode_fn: Callable[..., torch.Tensor], prefix_fn: Callable[..., torch.Tensor], layer_idx: int | None=None) -> torch.Tensor`. It mainly works with `q`, `k`, `v`, `kv_cache`, `state_indices_tensor`, `attn_metadata`, `slope_rate`, `block_size`; implements one step of the Mamba/SSM execution path. The body uses branching, iteration, tensor/kernel operations. Key calls include `range`, `torch.concat.contiguous`, `getattr`, `q.transpose.contiguous`, `k.transpose.contiguous`, `v.transpose.contiguous`.
**CN:** 定义函数 `linear_attention_prefill_and_mix`，其签名为 `linear_attention_prefill_and_mix(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_cache: torch.Tensor, state_indices_tensor: torch.Tensor, attn_metadata: LinearAttentionMetadata, slope_rate: torch.Tensor, block_size: int, decode_fn: Callable[..., torch.Tensor], prefix_fn: Callable[..., torch.Tensor], layer_idx: int | None=None) -> torch.Tensor`。它主要围绕 `q`, `k`, `v`, `kv_cache`, `state_indices_tensor`, `attn_metadata`, `slope_rate`, `block_size` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `range`, `torch.concat.contiguous`, `getattr`, `q.transpose.contiguous`, `k.transpose.contiguous`, `v.transpose.contiguous`。

### Function `linear_attention` (lines 465-473)
```python
def linear_attention(
    hidden_states: torch.Tensor,
    output: torch.Tensor,
    positions: torch.Tensor,
    layer_name: str,
) -> None:
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    self._forward(hidden_states=hidden_states, output=output, positions=positions)
```
**EN:** Defines function `linear_attention` with signature `linear_attention(hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor, layer_name: str) -> None`. It mainly works with `hidden_states`, `output`, `positions`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `get_forward_context`, `self._forward`.
**CN:** 定义函数 `linear_attention`，其签名为 `linear_attention(hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor, layer_name: str) -> None`。它主要围绕 `hidden_states`, `output`, `positions`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_forward_context`, `self._forward`。

### Function `linear_attention_fake` (lines 476-482)
```python
def linear_attention_fake(
    hidden_states: torch.Tensor,
    output: torch.Tensor,
    positions: torch.Tensor,
    layer_name: str,
) -> None:
    return
```
**EN:** Defines function `linear_attention_fake` with signature `linear_attention_fake(hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor, layer_name: str) -> None`. It mainly works with `hidden_states`, `output`, `positions`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `linear_attention_fake`，其签名为 `linear_attention_fake(hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor, layer_name: str) -> None`。它主要围绕 `hidden_states`, `output`, `positions`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `MiniMaxText01RMSNormTP` overview (lines 39-121)
```python
class MiniMaxText01RMSNormTP(CustomOp):
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        *,
        weight_shard_world_size: int | None = None,
        weight_shard_rank: int | None = None,
    ) -> None:
        super().__init__()
        self.tp_world = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.weight_shard_world = weight_shard_world_size or self.tp_world
        self.weight_shard_rank = (
            self.tp_rank if weight_shard_rank is None else weight_shard_rank
        )

        self.weight = nn.Parameter(torch.ones(hidden_size // self.weight_shard_world))
        self.weight.weight_loader = partial(
            self.weight_loader,
            shard_world_size=self.weight_shard_world,
            shard_rank=self.weight_shard_rank,
        )
        self.variance_epsilon = eps
```
**EN:** Defines class `MiniMaxText01RMSNormTP` with base classes `CustomOp` and decorators `CustomOp.register('minimax_text01_rmsnorm_tp')`. It acts as a Mamba/SSM module building block and exposes 5 direct methods, with notable entries `__init__`, `weight_loader`, `_forward`, `forward`, `forward_qk`.
**CN:** 定义类 `MiniMaxText01RMSNormTP`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('minimax_text01_rmsnorm_tp')`。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 5 个方法，较重要的包括 `__init__`, `weight_loader`, `_forward`, `forward`, `forward_qk`。

### Method `MiniMaxText01RMSNormTP.__init__` (lines 40-62)
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        *,
        weight_shard_world_size: int | None = None,
        weight_shard_rank: int | None = None,
    ) -> None:
        super().__init__()
        self.tp_world = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.weight_shard_world = weight_shard_world_size or self.tp_world
        self.weight_shard_rank = (
            self.tp_rank if weight_shard_rank is None else weight_shard_rank
        )

        self.weight = nn.Parameter(torch.ones(hidden_size // self.weight_shard_world))
        self.weight.weight_loader = partial(
            self.weight_loader,
            shard_world_size=self.weight_shard_world,
            shard_rank=self.weight_shard_rank,
        )
        self.variance_epsilon = eps
```
**EN:** Defines function `MiniMaxText01RMSNormTP.__init__` with signature `__init__(self, hidden_size: int, eps: float=1e-06, *, weight_shard_world_size: int | None=None, weight_shard_rank: int | None=None) -> None`. It mainly works with `hidden_size`, `eps`, `weight_shard_world_size`, `weight_shard_rank`; initializes the object state and cached resources. The body uses branching, tensor/kernel operations. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `nn.Parameter`, `partial`, `torch.ones`.
**CN:** 定义函数 `MiniMaxText01RMSNormTP.__init__`，其签名为 `__init__(self, hidden_size: int, eps: float=1e-06, *, weight_shard_world_size: int | None=None, weight_shard_rank: int | None=None) -> None`。它主要围绕 `hidden_size`, `eps`, `weight_shard_world_size`, `weight_shard_rank` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、张量或内核操作。关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `nn.Parameter`, `partial`, `torch.ones`。

### Method `MiniMaxText01RMSNormTP.weight_loader` (lines 65-78)
```python
    def weight_loader(
        param: nn.Parameter,
        loaded_weight: torch.Tensor,
        shard_world_size: int | None = None,
        shard_rank: int | None = None,
    ) -> None:
        if shard_world_size is None:
            shard_world_size = get_tensor_model_parallel_world_size()
        if shard_rank is None:
            shard_rank = get_tensor_model_parallel_rank()

        shard_size = loaded_weight.shape[0] // shard_world_size
        shard = slice(shard_rank * shard_size, (shard_rank + 1) * shard_size)
        param.data.copy_(loaded_weight[shard])
```
**EN:** Defines function `MiniMaxText01RMSNormTP.weight_loader` with signature `weight_loader(param: nn.Parameter, loaded_weight: torch.Tensor, shard_world_size: int | None=None, shard_rank: int | None=None) -> None`. It mainly works with `param`, `loaded_weight`, `shard_world_size`, `shard_rank`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `slice`, `param.data.copy_`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`.
**CN:** 定义函数 `MiniMaxText01RMSNormTP.weight_loader`，其签名为 `weight_loader(param: nn.Parameter, loaded_weight: torch.Tensor, shard_world_size: int | None=None, shard_rank: int | None=None) -> None`。它主要围绕 `param`, `loaded_weight`, `shard_world_size`, `shard_rank` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `slice`, `param.data.copy_`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`。

### Method `MiniMaxText01RMSNormTP.forward` (lines 93-99)
```python
    def forward(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        assert residual is None, "RMSNorm does not support residual connection."
        return self._forward(x)
```
**EN:** Defines function `MiniMaxText01RMSNormTP.forward` with signature `forward(self, x: torch.Tensor, residual: torch.Tensor | None=None) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `residual`; runs the main forward-path computation. The body uses validation/error handling. Key calls include `self._forward`.
**CN:** 定义函数 `MiniMaxText01RMSNormTP.forward`，其签名为 `forward(self, x: torch.Tensor, residual: torch.Tensor | None=None) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `residual` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 `self._forward`。

### Method `MiniMaxText01RMSNormTP.forward_qk` (lines 102-121)
```python
    def forward_qk(
        q_norm: "MiniMaxText01RMSNormTP",
        k_norm: "MiniMaxText01RMSNormTP",
        q: torch.Tensor,
        k: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        orig_dtype = q.dtype
        q = q.to(torch.float32)
        k = k.to(torch.float32)
        q_var = q.pow(2).mean(dim=-1, keepdim=True)
        k_var = k.pow(2).mean(dim=-1, keepdim=True)
        if q_norm.tp_world > 1:
            qk_var = torch.cat([q_var, k_var], dim=-1)
            qk_var = tensor_model_parallel_all_reduce(qk_var) / q_norm.tp_world
            q_var, k_var = qk_var.chunk(2, dim=-1)
        q = q * torch.rsqrt(q_var + q_norm.variance_epsilon) * q_norm.weight
        k = k * torch.rsqrt(k_var + k_norm.variance_epsilon) * k_norm.weight
        q = q.to(orig_dtype)
        k = k.to(orig_dtype)
        return q, k
```
**EN:** Defines function `MiniMaxText01RMSNormTP.forward_qk` with signature `forward_qk(q_norm: 'MiniMaxText01RMSNormTP', k_norm: 'MiniMaxText01RMSNormTP', q: torch.Tensor, k: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `q_norm`, `k_norm`, `q`, `k`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `q.to`, `k.to`, `q.pow.mean`, `k.pow.mean`, `torch.cat`, `qk_var.chunk`.
**CN:** 定义函数 `MiniMaxText01RMSNormTP.forward_qk`，其签名为 `forward_qk(q_norm: 'MiniMaxText01RMSNormTP', k_norm: 'MiniMaxText01RMSNormTP', q: torch.Tensor, k: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `q_norm`, `k_norm`, `q`, `k` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `q.to`, `k.to`, `q.pow.mean`, `k.pow.mean`, `torch.cat`, `qk_var.chunk`。

### Class `MiniMaxText01LinearKernel` overview (lines 219-245)
```python
class MiniMaxText01LinearKernel:
    @staticmethod
    def jit_linear_forward_prefix(
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        kv_caches: torch.Tensor,
        slope_rate: torch.Tensor,
        block_size: int,
        layer_idx: int | None = None,
        **kwargs,
    ) -> torch.Tensor:
        slope_rate = slope_rate.to(torch.float32)
        should_pad_dim = q.dim() == 3
        if should_pad_dim:
            q = q.unsqueeze(0)
            k = k.unsqueeze(0)
            v = v.unsqueeze(0)
        b, h, n, d = q.shape
        e = d
        kv_history = kv_caches.reshape(1, h, d, e).contiguous()
        output, kv_history = lightning_attention(
            q, k, v, slope_rate, block_size=block_size, kv_history=kv_history
        )
        kv_caches.copy_(kv_history[:, :, -1, :, :].reshape(h, d, e))
```
**EN:** Defines class `MiniMaxText01LinearKernel` with base classes no explicit base class and decorators none. It acts as a Mamba/SSM module building block and exposes 1 direct methods, with notable entries `jit_linear_forward_prefix`.
**CN:** 定义类 `MiniMaxText01LinearKernel`，其基类为 无显式基类，装饰器为 无。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 1 个方法，较重要的包括 `jit_linear_forward_prefix`。

### Method `MiniMaxText01LinearKernel.jit_linear_forward_prefix` (lines 221-245)
```python
    def jit_linear_forward_prefix(
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        kv_caches: torch.Tensor,
        slope_rate: torch.Tensor,
        block_size: int,
        layer_idx: int | None = None,
        **kwargs,
    ) -> torch.Tensor:
        slope_rate = slope_rate.to(torch.float32)
        should_pad_dim = q.dim() == 3
        if should_pad_dim:
            q = q.unsqueeze(0)
            k = k.unsqueeze(0)
            v = v.unsqueeze(0)
        b, h, n, d = q.shape
        e = d
        kv_history = kv_caches.reshape(1, h, d, e).contiguous()
        output, kv_history = lightning_attention(
            q, k, v, slope_rate, block_size=block_size, kv_history=kv_history
        )
        kv_caches.copy_(kv_history[:, :, -1, :, :].reshape(h, d, e))
        assert output.shape[0] == 1, "batch size must be 1"
        return rearrange(output.squeeze(0), "h n d -> n (h d)")
```
**EN:** Defines function `MiniMaxText01LinearKernel.jit_linear_forward_prefix` with signature `jit_linear_forward_prefix(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_caches: torch.Tensor, slope_rate: torch.Tensor, block_size: int, layer_idx: int | None=None, **kwargs) -> torch.Tensor`. It mainly works with `q`, `k`, `v`, `kv_caches`, `slope_rate`, `block_size`, `layer_idx`, `**kwargs`; implements one step of the Mamba/SSM execution path. The body uses branching, validation/error handling. Key calls include `slope_rate.to`, `kv_caches.reshape.contiguous`, `lightning_attention`, `kv_caches.copy_`, `rearrange`, `q.dim`.
**CN:** 定义函数 `MiniMaxText01LinearKernel.jit_linear_forward_prefix`，其签名为 `jit_linear_forward_prefix(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, kv_caches: torch.Tensor, slope_rate: torch.Tensor, block_size: int, layer_idx: int | None=None, **kwargs) -> torch.Tensor`。它主要围绕 `q`, `k`, `v`, `kv_caches`, `slope_rate`, `block_size`, `layer_idx`, `**kwargs` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `slope_rate.to`, `kv_caches.reshape.contiguous`, `lightning_attention`, `kv_caches.copy_`, `rearrange`, `q.dim`。

### Class `MiniMaxText01LinearAttention` overview (lines 248-462)
```python
class MiniMaxText01LinearAttention(nn.Module, MambaBase):
    @property
    def mamba_type(self) -> MambaAttentionBackendEnum:
        return MambaAttentionBackendEnum.LINEAR

    def get_state_dtype(self) -> tuple[torch.dtype]:
        assert self.model_config is not None
        assert self.cache_config is not None
        return MambaStateDtypeCalculator.linear_attention_state_dtype(
            self.model_config.dtype,
            self.cache_config.mamba_cache_dtype,
        )

    def get_state_shape(self) -> tuple[tuple[int, int, int], ...]:
        return MambaStateShapeCalculator.linear_attention_state_shape(
            num_heads=self.num_heads, tp_size=self.tp_size, head_dim=self.head_dim
        )

    def __init__(
        self,
        hidden_size: int,
        hidden_inner_size: int,
        num_heads: int,
        head_dim: int,
        max_position: int,
```
**EN:** Defines class `MiniMaxText01LinearAttention` with base classes `nn.Module`, `MambaBase` and decorators none. It acts as an attention-related layer or kernel wrapper and exposes 10 direct methods, with notable entries `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `weight_direct_load`, `_build_slope_tensor`.
**CN:** 定义类 `MiniMaxText01LinearAttention`，其基类为 `nn.Module`, `MambaBase`，装饰器为 无。它在整体实现中充当与注意力相关的层或内核封装，并直接暴露 10 个方法，较重要的包括 `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `weight_direct_load`, `_build_slope_tensor`。

### Method `MiniMaxText01LinearAttention.get_state_dtype` (lines 253-259)
```python
    def get_state_dtype(self) -> tuple[torch.dtype]:
        assert self.model_config is not None
        assert self.cache_config is not None
        return MambaStateDtypeCalculator.linear_attention_state_dtype(
            self.model_config.dtype,
            self.cache_config.mamba_cache_dtype,
        )
```
**EN:** Defines function `MiniMaxText01LinearAttention.get_state_dtype` with signature `get_state_dtype(self) -> tuple[torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses validation/error handling. Key calls include `MambaStateDtypeCalculator.linear_attention_state_dtype`.
**CN:** 定义函数 `MiniMaxText01LinearAttention.get_state_dtype`，其签名为 `get_state_dtype(self) -> tuple[torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 `MambaStateDtypeCalculator.linear_attention_state_dtype`。

### Method `MiniMaxText01LinearAttention.get_state_shape` (lines 261-264)
```python
    def get_state_shape(self) -> tuple[tuple[int, int, int], ...]:
        return MambaStateShapeCalculator.linear_attention_state_shape(
            num_heads=self.num_heads, tp_size=self.tp_size, head_dim=self.head_dim
        )
```
**EN:** Defines function `MiniMaxText01LinearAttention.get_state_shape` with signature `get_state_shape(self) -> tuple[tuple[int, int, int], ...]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `MambaStateShapeCalculator.linear_attention_state_shape`.
**CN:** 定义函数 `MiniMaxText01LinearAttention.get_state_shape`，其签名为 `get_state_shape(self) -> tuple[tuple[int, int, int], ...]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `MambaStateShapeCalculator.linear_attention_state_shape`。

### Method `MiniMaxText01LinearAttention.__init__` (lines 266-342)
```python
    def __init__(
        self,
        hidden_size: int,
        hidden_inner_size: int,
        num_heads: int,
        head_dim: int,
        max_position: int,
        block_size: int,
        num_hidden_layer: int,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        layer_idx: int = 0,
        linear_layer_idx: int = 0,
        prefix: str = "linear_attn",
    ) -> None:
        super().__init__()

        self.layer_idx = layer_idx
        self.BLOCK = block_size
        self.hidden_size = hidden_size
        self.num_heads = num_heads
        self.head_dim = head_dim
        self.total_num_heads = num_heads
        self.hidden_inner_size = hidden_inner_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()

        assert self.total_num_heads % self.tp_size == 0
        self.tp_heads = self.total_num_heads // self.tp_size
        self.qkv_size = self.num_heads * self.head_dim
        self.tp_hidden = self.head_dim * self.tp_heads
        self.model_config = model_config
        self.cache_config = cache_config
        self.prefix = prefix

        self.qkv_proj = ColumnParallelLinear(
            hidden_size,
# ... truncated for analysis ...
        if num_hidden_layer <= 1:
            self.slope_rate = slope_rate * (1 + 1e-5)
        else:
            self.slope_rate = slope_rate * (
                1 - layer_idx / (num_hidden_layer - 1) + 1e-5
            )
        self.tp_slope = self.slope_rate[
            self.tp_rank * self.tp_heads : (self.tp_rank + 1) * self.tp_heads
        ].contiguous()

        compilation_config = get_current_vllm_config().compilation_config
        if prefix in compilation_config.static_forward_context:
            raise ValueError(f"Duplicate layer name: {prefix}")
        compilation_config.static_forward_context[prefix] = self
```
**EN:** Defines function `MiniMaxText01LinearAttention.__init__` with signature `__init__(self, hidden_size: int, hidden_inner_size: int, num_heads: int, head_dim: int, max_position: int, block_size: int, num_hidden_layer: int, model_config: ModelConfig | None=None, cache_config: CacheConfig | None=None, quant_config: QuantizationConfig | None=None, layer_idx: int=0, linear_layer_idx: int=0, prefix: str='linear_attn') -> None`. It mainly works with `hidden_size`, `hidden_inner_size`, `num_heads`, `head_dim`, `max_position`, `block_size`, `num_hidden_layer`, `model_config`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `ColumnParallelLinear`, `RowParallelLinear`, `MiniMaxText01RMSNormTP`.
**CN:** 定义函数 `MiniMaxText01LinearAttention.__init__`，其签名为 `__init__(self, hidden_size: int, hidden_inner_size: int, num_heads: int, head_dim: int, max_position: int, block_size: int, num_hidden_layer: int, model_config: ModelConfig | None=None, cache_config: CacheConfig | None=None, quant_config: QuantizationConfig | None=None, layer_idx: int=0, linear_layer_idx: int=0, prefix: str='linear_attn') -> None`。它主要围绕 `hidden_size`, `hidden_inner_size`, `num_heads`, `head_dim`, `max_position`, `block_size`, `num_hidden_layer`, `model_config` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `ColumnParallelLinear`, `RowParallelLinear`, `MiniMaxText01RMSNormTP`。

### Method `MiniMaxText01LinearAttention.forward` (lines 405-413)
```python
    def forward(
        self, hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor
    ) -> None:
        torch.ops.vllm.linear_attention(
            hidden_states,
            output,
            positions,
            self.prefix,
        )
```
**EN:** Defines function `MiniMaxText01LinearAttention.forward` with signature `forward(self, hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor) -> None`. It mainly works with `hidden_states`, `output`, `positions`; runs the main forward-path computation. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.linear_attention`.
**CN:** 定义函数 `MiniMaxText01LinearAttention.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, output: torch.Tensor, positions: torch.Tensor) -> None`。它主要围绕 `hidden_states`, `output`, `positions` 展开；执行主要的前向计算路径。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.linear_attention`。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level classes include `MiniMaxText01RMSNormTP`, `MiniMaxText01LinearKernel`, `MiniMaxText01LinearAttention`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MiniMaxText01RMSNormTP`, `MiniMaxText01LinearKernel`, `MiniMaxText01LinearAttention`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `clear_linear_attention_cache_for_new_sequences`, `linear_attention_decode`, `linear_attention_prefill_and_mix`, `linear_attention`, `linear_attention_fake` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `clear_linear_attention_cache_for_new_sequences`, `linear_attention_decode`, `linear_attention_prefill_and_mix`, `linear_attention`, `linear_attention_fake` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `math`, `collections`, `functools`, `torch`, `einops`
- **Internal / 内部**: `vllm.config`, `vllm.distributed.communication_op`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.lightning_attn`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.mamba.abstract`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.quantization`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.linear_attn`, `vllm.v1.attention.backends.registry`
