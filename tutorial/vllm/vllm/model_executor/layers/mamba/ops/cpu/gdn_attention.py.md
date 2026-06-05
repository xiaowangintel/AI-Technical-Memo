# gdn_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/ops/cpu/gdn_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides helpers such as `cpu_gdn_attention_core`, `cpu_gdn_attention_core_fake`, `cpu_gdn_attention_core_amx` for Mamba/state-space layers and kernels. / 提供诸如 `cpu_gdn_attention_core`, `cpu_gdn_attention_core_fake`, `cpu_gdn_attention_core_amx` 之类的辅助函数，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-25)
```python
from __future__ import annotations

import torch

import vllm._custom_ops as ops
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.model_executor.layers.mamba.mamba_utils import is_conv_state_dim_first
from vllm.model_executor.layers.mamba.ops.cpu.causal_conv1d import (
    causal_conv1d_torch,
    causal_conv1d_update_torch,
)
from vllm.model_executor.layers.mamba.ops.cpu.recurrent_gated_delta_rule import (
    chunk_gated_delta_rule,
    gdn_gating,
    recurrent_gated_delta_rule,
)
from vllm.utils.torch_utils import (
    LayerNameType,
    _resolve_layer_name,
    direct_register_custom_op,
)
from vllm.v1.attention.backends.gdn_attn import GDNAttentionMetadata
```
**EN:** This opening block pulls in external dependencies such as `__future__`, `torch` and internal modules such as `vllm._custom_ops`, `vllm.forward_context`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.mamba.ops.cpu.causal_conv1d`, `vllm.model_executor.layers.mamba.ops.cpu.recurrent_gated_delta_rule`, `vllm.utils.torch_utils`. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `__future__`, `torch`）以及内部模块（如 `vllm._custom_ops`, `vllm.forward_context`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.mamba.ops.cpu.causal_conv1d`, `vllm.model_executor.layers.mamba.ops.cpu.recurrent_gated_delta_rule`, `vllm.utils.torch_utils`）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 27-27)
```python
_CPU_GDN_ATTENTION_OPS_REGISTERED = False
```
**EN:** This block defines module-level metadata or constants such as `_CPU_GDN_ATTENTION_OPS_REGISTERED`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the Mamba/state-space layers and kernels pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `_CPU_GDN_ATTENTION_OPS_REGISTERED`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在Mamba/状态空间模型层与内核流程中复用。

### Function `cpu_gdn_attention_core` (lines 30-194)
```python
def cpu_gdn_attention_core(
    mixed_qkv: torch.Tensor,
    b: torch.Tensor,
    a: torch.Tensor,
    core_attn_out: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    """CPU custom op for the core GDN attention computation."""
    layer_name = _resolve_layer_name(layer_name)
    forward_context: ForwardContext = get_forward_context()
    layer = forward_context.no_compile_layers[layer_name]

    attn_metadata = forward_context.attn_metadata

    if attn_metadata is None:
        return

    assert isinstance(attn_metadata, dict)
    attn_metadata_i = attn_metadata[layer.prefix]
    assert isinstance(attn_metadata_i, GDNAttentionMetadata)

    if attn_metadata_i.num_actual_tokens == 0:
        return

    assert (
        attn_metadata_i.spec_sequence_masks is None
        and attn_metadata_i.num_accepted_tokens is None
    ), "speculative decode not supported in CPU GDN attention."

    if torch.cpu._is_amx_tile_supported():
        return cpu_gdn_attention_core_amx(
            mixed_qkv,
            b,
            a,
            core_attn_out,
            attn_metadata_i,
            layer,
        )
# ... truncated for analysis ...
        initial_state[~prefill_has_initial_state, ...] = 0
        attn_out, last_recurrent_state = chunk_gated_delta_rule(
            q=query,
            k=key,
            v=value,
            g=g,
            beta=beta,
            scale=None,
            initial_state=initial_state,
            cu_seqlens=prefill_query_start_loc,
            use_qk_l2norm_in_kernel=True,
        )
        ssm_state[prefill_state_indices] = last_recurrent_state.to(ssm_state.dtype)
        core_attn_out[prefill_token_start:prefill_token_end] = attn_out.squeeze(0)
```
**EN:** Defines function `cpu_gdn_attention_core` with signature `cpu_gdn_attention_core(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, layer_name: LayerNameType) -> None`. It mainly works with `mixed_qkv`, `b`, `a`, `core_attn_out`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `_resolve_layer_name`, `get_forward_context`, `isinstance`, `torch.cpu._is_amx_tile_supported`, `layer.conv1d.weight.view`, `cpu_gdn_attention_core_amx`.
**CN:** 定义函数 `cpu_gdn_attention_core`，其签名为 `cpu_gdn_attention_core(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, layer_name: LayerNameType) -> None`。它主要围绕 `mixed_qkv`, `b`, `a`, `core_attn_out`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `_resolve_layer_name`, `get_forward_context`, `isinstance`, `torch.cpu._is_amx_tile_supported`, `layer.conv1d.weight.view`, `cpu_gdn_attention_core_amx`。

### Function `cpu_gdn_attention_core_fake` (lines 197-205)
```python
def cpu_gdn_attention_core_fake(
    mixed_qkv: torch.Tensor,
    b: torch.Tensor,
    a: torch.Tensor,
    core_attn_out: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    """Fake implementation for torch.compile."""
    return
```
**EN:** Defines function `cpu_gdn_attention_core_fake` with signature `cpu_gdn_attention_core_fake(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, layer_name: LayerNameType) -> None`. It mainly works with `mixed_qkv`, `b`, `a`, `core_attn_out`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `cpu_gdn_attention_core_fake`，其签名为 `cpu_gdn_attention_core_fake(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, layer_name: LayerNameType) -> None`。它主要围绕 `mixed_qkv`, `b`, `a`, `core_attn_out`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `cpu_gdn_attention_core_amx` (lines 208-334)
```python
def cpu_gdn_attention_core_amx(
    mixed_qkv: torch.Tensor,
    b: torch.Tensor,
    a: torch.Tensor,
    core_attn_out: torch.Tensor,
    attn_metadata_i: GDNAttentionMetadata,
    layer: torch.nn.Module,
):
    state_indices_tensor = attn_metadata_i.non_spec_state_indices_tensor
    query_start_loc = attn_metadata_i.non_spec_query_start_loc
    assert state_indices_tensor is not None
    assert query_start_loc is not None

    # [num_allocated_slots, kernel - 1, conv_dim]
    conv_state = layer.kv_cache[0]
    if is_conv_state_dim_first():
        raise RuntimeError("AMX GDN attention requires `SD` conv_state layout.")
    # reshape to [num_allocated_slots, conv_dim, kernel - 1]
    conv_state_t = conv_state.transpose(1, 2)

    # [num_allocated_slots, num_v_heads / tp_size, v_dim, k_dim]
    ssm_state = layer.kv_cache[1]
    # rehape to [num_allocated_slots, num_v_heads / tp_size, k_dim, v_dim]
    num_allocated_slots, head_num, v_dim, k_dim = ssm_state.size()
    ssm_state = ssm_state.view(
        num_allocated_slots,
        head_num,
        k_dim,
        v_dim,
    )

    mixed_qkv = mixed_qkv.contiguous()
    a = a.contiguous()
    b = b.contiguous()

    num_decodes = attn_metadata_i.num_decodes
    num_decode_tokens = attn_metadata_i.num_decode_tokens
    num_prefills = attn_metadata_i.num_prefills
# ... truncated for analysis ...
            key=key,
            value=value,
            g=g,
            beta=beta,
            initial_state=initial_state,
            output_final_state=True,
            cu_seqlens=prefill_query_start_loc,
            head_first=False,
            use_qk_l2norm_in_kernel=True,
        )
        ssm_state[prefill_state_indices] = last_recurrent_state.to(
            ssm_state.dtype, copy=False
        )
        core_attn_out[prefill_token_start:prefill_token_end] = attn_out.squeeze(0)
```
**EN:** Defines function `cpu_gdn_attention_core_amx` with signature `cpu_gdn_attention_core_amx(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, attn_metadata_i: GDNAttentionMetadata, layer: torch.nn.Module)`. It mainly works with `mixed_qkv`, `b`, `a`, `core_attn_out`, `attn_metadata_i`, `layer`; implements one step of the Mamba/SSM execution path. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `is_conv_state_dim_first`, `conv_state.transpose`, `ssm_state.size`, `ssm_state.view`, `mixed_qkv.contiguous`, `a.contiguous`.
**CN:** 定义函数 `cpu_gdn_attention_core_amx`，其签名为 `cpu_gdn_attention_core_amx(mixed_qkv: torch.Tensor, b: torch.Tensor, a: torch.Tensor, core_attn_out: torch.Tensor, attn_metadata_i: GDNAttentionMetadata, layer: torch.nn.Module)`。它主要围绕 `mixed_qkv`, `b`, `a`, `core_attn_out`, `attn_metadata_i`, `layer` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `is_conv_state_dim_first`, `conv_state.transpose`, `ssm_state.size`, `ssm_state.view`, `mixed_qkv.contiguous`, `a.contiguous`。

### Function `register_cpu_gdn_attention_ops` (lines 337-348)
```python
def register_cpu_gdn_attention_ops() -> None:
    global _CPU_GDN_ATTENTION_OPS_REGISTERED
    if _CPU_GDN_ATTENTION_OPS_REGISTERED:
        return

    direct_register_custom_op(
        op_name="cpu_gdn_attention_core",
        op_func=cpu_gdn_attention_core,
        mutates_args=["core_attn_out"],
        fake_impl=cpu_gdn_attention_core_fake,
    )
    _CPU_GDN_ATTENTION_OPS_REGISTERED = True
```
**EN:** Defines function `register_cpu_gdn_attention_ops` with signature `register_cpu_gdn_attention_ops() -> None`. It mainly works with object context only; connects the implementation to a registry or backend lookup path. The body uses branching. Key calls include `direct_register_custom_op`.
**CN:** 定义函数 `register_cpu_gdn_attention_ops`，其签名为 `register_cpu_gdn_attention_ops() -> None`。它主要围绕 仅依赖对象上下文 展开；把实现接入注册表或后端查找路径。函数体包含分支判断。关键调用包括 `direct_register_custom_op`。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level helpers such as `cpu_gdn_attention_core`, `cpu_gdn_attention_core_fake`, `cpu_gdn_attention_core_amx`, `register_cpu_gdn_attention_ops` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `cpu_gdn_attention_core`, `cpu_gdn_attention_core_fake`, `cpu_gdn_attention_core_amx`, `register_cpu_gdn_attention_ops` 为主要类提供了过程式入口。
- **EN:** Backend-specific kernels are important here, as the module imports Triton/custom-op infrastructure or sits under an ops directory.
  **CN:** 这里明显依赖后端特定内核，因为模块导入了 Triton/自定义算子基础设施，或本身位于 ops 目录下。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`
- **Internal / 内部**: `vllm._custom_ops`, `vllm.forward_context`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.mamba.ops.cpu.causal_conv1d`, `vllm.model_executor.layers.mamba.ops.cpu.recurrent_gated_delta_rule`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.gdn_attn`
