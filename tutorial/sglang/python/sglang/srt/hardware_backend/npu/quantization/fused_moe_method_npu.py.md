# fused_moe_method_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/quantization/fused_moe_method_npu.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for quantization helpers inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的量化辅助逻辑支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING, Optional

import numpy as np
import torch

from sglang.srt.hardware_backend.npu.utils import npu_format_cast
from sglang.srt.layers.quantization.base_config import FusedMoEMethodBase

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.layers.quantization.base_config import QuantizationConfig
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `numpy`, `torch`, `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `numpy`, `torch`, `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`。

### Lines 17-100: Function `npu_fused_experts_w4a4` / 函数 `npu_fused_experts_w4a4`
```python
def npu_fused_experts_w4a4(
    hidden_states: torch.Tensor,
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    top_k: int,
):
    original_shape = hidden_states.shape
    original_dtype = hidden_states.dtype
    scale_dtype = original_dtype if original_dtype == torch.bfloat16 else torch.float32
    if len(original_shape) == 3:
        hidden_states = hidden_states.view(-1, hidden_states.shape[-1])
    num_tokens = hidden_states.shape[0]
    num_experts = w13.shape[0]

    hidden_states, expanded_row_idx, expert_tokens, _ = (
        torch.ops.npu.npu_moe_init_routing_v2(
            hidden_states,
            topk_ids,
            active_num=num_tokens * top_k,
            expert_num=num_experts,
            expert_tokens_num_type=1,
            expert_tokens_num_flag=True,
            active_expert_range=[0, num_experts],
            quant_mode=-1,
# ... omitted for brevity ...
        export_for_source_row=topk_ids,
        drop_pad_mode=2,
    )
    if len(original_shape) == 3:
        final_hidden_states = final_hidden_states.view(original_shape)
    return final_hidden_states
```
**EN:** This function implements `npu_fused_experts_w4a4`. It primarily calls `torch.ops.npu.npu_moe_init_routing_v2`, `expert_tokens.to`, `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_moe_finalize_routing`, `len` to complete its work. State updates are written into `original_shape`, `original_dtype`, `scale_dtype`, `num_tokens`, `num_experts`, `expert_tokens`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `npu_fused_experts_w4a4`。 它主要通过调用 `torch.ops.npu.npu_moe_init_routing_v2`, `expert_tokens.to`, `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_moe_finalize_routing`, `len` 来完成任务。 状态更新主要写入 `original_shape`, `original_dtype`, `scale_dtype`, `num_tokens`, `num_experts`, `expert_tokens`。 实现中使用了条件分支。

### Lines 103-202: Function `npu_fused_experts` / 函数 `npu_fused_experts`
```python
def npu_fused_experts(
    hidden_states: torch.Tensor,
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    top_k: int,
    **kwargs,
):
    w13_offset = kwargs.get("w13_offset", None)
    w2_offset = kwargs.get("w2_offset", None)
    use_wna16 = kwargs.get("use_wna16", False)

    original_shape = hidden_states.shape
    original_dtype = hidden_states.dtype
    scale_dtype = original_dtype if original_dtype == torch.bfloat16 else torch.float32
    if len(original_shape) == 3:
        hidden_states = hidden_states.view(-1, hidden_states.shape[-1])
    num_tokens = hidden_states.shape[0]
    num_experts = w13.shape[0]
    row_idx_len = num_tokens * top_k
    row_idx = (
        torch.arange(0, row_idx_len, dtype=torch.int32, device=topk_weights.device)
        .view(top_k, -1)
        .permute(1, 0)
        .contiguous()
# ... omitted for brevity ...
        expanded_src_to_dst_row=expanded_row_idx,
        export_for_source_row=topk_ids,
    )
    if len(original_shape) == 3:
        final_hidden_states = final_hidden_states.view(original_shape)
    return final_hidden_states
```
**EN:** This function implements `npu_fused_experts`. It primarily calls `kwargs.get`, `torch.arange.view.permute.contiguous`, `torch.ops.npu.npu_moe_init_routing`, `torch.ops.npu.npu_moe_compute_expert_tokens`, `expert_tokens.to`, `torch.ops.npu.npu_moe_finalize_routing` to complete its work. State updates are written into `w13_offset`, `w2_offset`, `use_wna16`, `original_shape`, `original_dtype`, `scale_dtype`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `npu_fused_experts`。 它主要通过调用 `kwargs.get`, `torch.arange.view.permute.contiguous`, `torch.ops.npu.npu_moe_init_routing`, `torch.ops.npu.npu_moe_compute_expert_tokens`, `expert_tokens.to`, `torch.ops.npu.npu_moe_finalize_routing` 来完成任务。 状态更新主要写入 `w13_offset`, `w2_offset`, `use_wna16`, `original_shape`, `original_dtype`, `scale_dtype`。 实现中使用了条件分支。

### Lines 205-274: Function `npu_fused_experts_w8a8_decode` / 函数 `npu_fused_experts_w8a8_decode`
```python
def npu_fused_experts_w8a8_decode(
    hidden_states: torch.Tensor,
    w13: torch.Tensor,
    w13_scale: torch.Tensor,
    w2: torch.Tensor,
    w2_scale: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    top_k: int,
    **kwargs,
):
    num_tokens = hidden_states.shape[:-1].numel()
    first_expert_idx = 0
    last_expert_idx = w13.shape[0]
    global_num_experts = w13.shape[0]
    original_shape = hidden_states.shape
    group_list_type = 1

    sorted_hidden_states, expanded_row_idx, expert_tokens, pertoken_scale = (
        torch.ops.npu.npu_moe_init_routing_v2(
            hidden_states,
            topk_ids,
            active_num=num_tokens * top_k,
            expert_num=global_num_experts,
            expert_tokens_num_type=group_list_type,
            expert_tokens_num_flag=True,
            active_expert_range=[first_expert_idx, last_expert_idx],
            quant_mode=1,
# ... omitted for brevity ...
        probs=topk_weights,
    )
    if len(original_shape) == 3:
        final_hidden_states = final_hidden_states.view(original_shape)

    return final_hidden_states
```
**EN:** This function implements `npu_fused_experts_w8a8_decode`. It primarily calls `hidden_states.shape.numel`, `torch.ops.npu.npu_moe_init_routing_v2`, `torch.ops.npu.npu_dequant_swiglu_quant`, `torch.ops.npu.npu_moe_token_unpermute`, `torch.ops.npu.npu_grouped_matmul`, `len` to complete its work. State updates are written into `num_tokens`, `first_expert_idx`, `last_expert_idx`, `global_num_experts`, `original_shape`, `group_list_type`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `npu_fused_experts_w8a8_decode`。 它主要通过调用 `hidden_states.shape.numel`, `torch.ops.npu.npu_moe_init_routing_v2`, `torch.ops.npu.npu_dequant_swiglu_quant`, `torch.ops.npu.npu_moe_token_unpermute`, `torch.ops.npu.npu_grouped_matmul`, `len` 来完成任务。 状态更新主要写入 `num_tokens`, `first_expert_idx`, `last_expert_idx`, `global_num_experts`, `original_shape`, `group_list_type`。 实现中使用了条件分支。

### Lines 277-305: Function `npu_fused_moe_without_routing_weights_bf16` / 函数 `npu_fused_moe_without_routing_weights_bf16`
```python
def npu_fused_moe_without_routing_weights_bf16(
    layer, hidden_states, group_list_type, group_list, output_dtype
):
    from sgl_kernel_npu.activation.swiglu_quant import swiglu_quant

    # gmm1: gate_up_proj
    hidden_states = torch.ops.npu.npu_grouped_matmul(
        x=[hidden_states],
        weight=[layer.w13_weight],
        split_item=2,
        group_list_type=group_list_type,
        group_type=0,
        group_list=group_list,
        output_dtype=output_dtype,
    )[0]
    hidden_states, _ = swiglu_quant(
        hidden_states, group_list, group_list_type, need_quant=False
    )
    # gmm2: down_proj
    hidden_states = torch.ops.npu.npu_grouped_matmul(
        x=[hidden_states],
        weight=[layer.w2_weight],
        split_item=2,
        group_list_type=group_list_type,
        group_type=0,
        group_list=group_list,
        output_dtype=output_dtype,
    )[0]
    return hidden_states
```
**EN:** This function implements `npu_fused_moe_without_routing_weights_bf16`. It primarily calls `swiglu_quant`, `torch.ops.npu.npu_grouped_matmul` to complete its work. State updates are written into `hidden_states`.
**CN:** 该函数实现了 `npu_fused_moe_without_routing_weights_bf16`。 它主要通过调用 `swiglu_quant`, `torch.ops.npu.npu_grouped_matmul` 来完成任务。 状态更新主要写入 `hidden_states`。

### Lines 308-384: Function `fused_moe_npu` / 函数 `fused_moe_npu`
```python
def fused_moe_npu(
    x,
    w1,
    w2,
    topk_output,
    moe_runner_config,
):
    # TODO: reuse the codes of UnquantizedFusedMoEMethod-forward_npu
    topk_weights, topk_ids, _ = topk_output
    original_dtype = x.dtype
    num_tokens = x.shape[0]
    topk_weights = topk_weights.to(x.dtype)
    topk_ids = topk_ids.to(torch.int32)
    num_experts = w1.shape[0]
    top_k = topk_weights.shape[-1]
    row_idx_len = num_tokens * top_k
    row_idx = (
        torch.arange(0, row_idx_len, dtype=torch.int32, device=topk_weights.device)
        .view(top_k, -1)
        .permute(1, 0)
        .contiguous()
    )

    hidden_states, expanded_row_idx, expanded_expert_idx = (
        torch.ops.npu.npu_moe_init_routing(
            x, row_idx=row_idx, expert_idx=topk_ids, active_num=num_tokens
        )
    )
# ... omitted for brevity ...
        bias=None,
        scales=topk_weights,
        expanded_src_to_dst_row=expanded_row_idx,
        export_for_source_row=topk_ids,
    )
    return final_hidden_states
```
**EN:** This function implements `fused_moe_npu`. It primarily calls `topk_weights.to`, `topk_ids.to`, `torch.arange.view.permute.contiguous`, `torch.ops.npu.npu_moe_init_routing`, `torch.ops.npu.npu_moe_compute_expert_tokens`, `expert_tokens.to` to complete its work. State updates are written into `original_dtype`, `num_tokens`, `topk_weights`, `topk_ids`, `num_experts`, `top_k`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `fused_moe_npu`。 它主要通过调用 `topk_weights.to`, `topk_ids.to`, `torch.arange.view.permute.contiguous`, `torch.ops.npu.npu_moe_init_routing`, `torch.ops.npu.npu_moe_compute_expert_tokens`, `expert_tokens.to` 来完成任务。 状态更新主要写入 `original_dtype`, `num_tokens`, `topk_weights`, `topk_ids`, `num_experts`, `top_k`。 实现中使用了条件分支。

### Lines 387-387: Class `_NPUFusedMoEMethodBase` declaration / 类 `_NPUFusedMoEMethodBase` 声明
```python
class _NPUFusedMoEMethodBase(FusedMoEMethodBase):
```
**EN:** This class establishes `_NPUFusedMoEMethodBase` as the main container/coordinator for the surrounding logic. It inherits from `FusedMoEMethodBase`. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `_NPUFusedMoEMethodBase` 定义为周边逻辑的主要封装体或协调者。 它继承自 `FusedMoEMethodBase`。 其核心接口包括 `__init__` 等方法。

### Lines 389-393: Method `_NPUFusedMoEMethodBase.__init__` / 方法 `_NPUFusedMoEMethodBase.__init__`
```python
    def __init__(
        self,
        quant_config: Optional["QuantizationConfig"] = None,
    ):
        self.quant_config = quant_config
```
**EN:** This method implements `__init__` on `_NPUFusedMoEMethodBase`. State updates are written into `self.quant_config`.
**CN:** 该方法（属于 `_NPUFusedMoEMethodBase`）实现了 `__init__`。 状态更新主要写入 `self.quant_config`。

### Lines 396-396: Class `NPUW4A4Int4DynamicMoEMethod` declaration / 类 `NPUW4A4Int4DynamicMoEMethod` 声明
```python
class NPUW4A4Int4DynamicMoEMethod(_NPUFusedMoEMethodBase):
```
**EN:** This class establishes `NPUW4A4Int4DynamicMoEMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPUFusedMoEMethodBase`. Its core interface includes methods such as `process_weights_after_loading`, `_pack_to_int32`, `apply`, `apply_without_routing_weights`.
**CN:** 该类将 `NPUW4A4Int4DynamicMoEMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPUFusedMoEMethodBase`。 其核心接口包括 `process_weights_after_loading`, `_pack_to_int32`, `apply`, `apply_without_routing_weights` 等方法。

### Lines 398-433: Method `NPUW4A4Int4DynamicMoEMethod.process_weights_after_loading` / 方法 `NPUW4A4Int4DynamicMoEMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.w13_weight.data = npu_format_cast(layer.w13_weight.data.transpose(1, 2))
        layer.w13_weight.data = self._pack_to_int32(
            layer.w13_weight.data.to(torch.int32)
        )

        layer.w2_weight.data = npu_format_cast(layer.w2_weight.data.transpose(1, 2))

        scale_np = layer.w13_weight_scale.data.cpu().numpy()
        scale_np.dtype = np.uint32
        scale_uint64_tensor = torch.from_numpy(scale_np.astype(np.int64)).npu()

        layer.w13_weight_scale = torch.nn.Parameter(
            scale_uint64_tensor.squeeze(-1), requires_grad=False
        )
        layer.w2_weight_scale = torch.nn.Parameter(
            layer.w2_weight_scale.data.squeeze(-1), requires_grad=False
        )

        # Compressed-tensors format doesn't have this field
        if hasattr(layer, "w13_weight_offset"):
            layer.w13_weight_offset = torch.nn.Parameter(
                layer.w13_weight_offset.data.squeeze(-1),
                requires_grad=False,
            )
        if hasattr(layer, "w2_weight_offset"):
            layer.w2_weight_offset = torch.nn.Parameter(
                layer.w2_weight_offset.data.squeeze(-1),
                requires_grad=False,
            )

        # Quantizes in int4 separately from the dispatcher
        # since deep_ep does not support quantization in int4
        # dispatching works in bf16
        if hasattr(layer, "dispatcher"):
            layer.dispatcher.set_quant_config({"dispatcher_output_dtype": "bf16"})
```
**EN:** This method implements `process_weights_after_loading` on `NPUW4A4Int4DynamicMoEMethod`. It primarily calls `npu_format_cast`, `self._pack_to_int32`, `layer.w13_weight_scale.data.cpu.numpy`, `torch.from_numpy.npu`, `torch.nn.Parameter`, `hasattr` to complete its work. State updates are written into `layer.w13_weight.data`, `layer.w2_weight.data`, `scale_np`, `scale_np.dtype`, `scale_uint64_tensor`, `layer.w13_weight_scale`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A4Int4DynamicMoEMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `npu_format_cast`, `self._pack_to_int32`, `layer.w13_weight_scale.data.cpu.numpy`, `torch.from_numpy.npu`, `torch.nn.Parameter`, `hasattr` 来完成任务。 状态更新主要写入 `layer.w13_weight.data`, `layer.w2_weight.data`, `scale_np`, `scale_np.dtype`, `scale_uint64_tensor`, `layer.w13_weight_scale`。 实现中使用了条件分支。

### Lines 435-442: Method `NPUW4A4Int4DynamicMoEMethod._pack_to_int32` / 方法 `NPUW4A4Int4DynamicMoEMethod._pack_to_int32`
```python
    def _pack_to_int32(self, weight: torch.Tensor):
        # pack 8 int4 to int32, we use a int32 to represent a int4
        assert (
            weight.shape[-1] % 8 == 0
        ), "the last dim of weight needs to be divided by 8"
        new_weight = torch.ops.npu.npu_convert_weight_to_int4pack(weight.flatten(0, 1))
        new_weight = new_weight.view(weight.shape[0], weight.shape[1], -1)
        return new_weight
```
**EN:** This method implements `_pack_to_int32` on `NPUW4A4Int4DynamicMoEMethod`. It primarily calls `torch.ops.npu.npu_convert_weight_to_int4pack`, `new_weight.view`, `weight.flatten` to complete its work. State updates are written into `new_weight`.
**CN:** 该方法（属于 `NPUW4A4Int4DynamicMoEMethod`）实现了 `_pack_to_int32`。 它主要通过调用 `torch.ops.npu.npu_convert_weight_to_int4pack`, `new_weight.view`, `weight.flatten` 来完成任务。 状态更新主要写入 `new_weight`。

### Lines 444-467: Method `NPUW4A4Int4DynamicMoEMethod.apply` / 方法 `NPUW4A4Int4DynamicMoEMethod.apply`
```python
    def apply(
        self,
        layer,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        topk_weights, topk_ids, _ = topk_output
        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(x.dtype)
        output = npu_fused_experts_w4a4(
            hidden_states=x,
            w13=layer.w13_weight,
            w13_scale=layer.w13_weight_scale,
            w2=layer.w2_weight,
            w2_scale=layer.w2_weight_scale,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            top_k=topk_ids.shape[1],
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This method implements `apply` on `NPUW4A4Int4DynamicMoEMethod`. It primarily calls `topk_ids.to`, `topk_weights.to`, `npu_fused_experts_w4a4`, `StandardCombineInput` to complete its work. State updates are written into `x`, `topk_output`, `topk_ids`, `topk_weights`, `output`.
**CN:** 该方法（属于 `NPUW4A4Int4DynamicMoEMethod`）实现了 `apply`。 它主要通过调用 `topk_ids.to`, `topk_weights.to`, `npu_fused_experts_w4a4`, `StandardCombineInput` 来完成任务。 状态更新主要写入 `x`, `topk_output`, `topk_ids`, `topk_weights`, `output`。

### Lines 469-509: Method `NPUW4A4Int4DynamicMoEMethod.apply_without_routing_weights` / 方法 `NPUW4A4Int4DynamicMoEMethod.apply_without_routing_weights`
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        hidden_states, hidden_states_scale = torch.ops.npu.npu_dynamic_quant(
            hidden_states, dst_type=torch.quint4x2
        )
        # gmm1: up_gate_proj
        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[layer.w13_weight],
            scale=[layer.w13_weight_scale],
            per_token_scale=[hidden_states_scale],
            split_item=2,
            group_list_type=group_list_type,
            group_type=0,
            group_list=group_list,
            output_dtype=output_dtype,
        )[0]
        # act_fn: swiglu
        hidden_states = torch.ops.npu.npu_swiglu(hidden_states)
        hidden_states, pertoken_scale = torch.ops.npu.npu_dynamic_quant(hidden_states)

# ... omitted for brevity ...
            group_list_type=group_list_type,
            group_type=0,
            group_list=group_list,
            output_dtype=output_dtype,
        )[0]
        return hidden_states
```
**EN:** This method implements `apply_without_routing_weights` on `NPUW4A4Int4DynamicMoEMethod`. It primarily calls `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_grouped_matmul`, `layer.w2_weight_scale.to` to complete its work. State updates are written into `hidden_states`.
**CN:** 该方法（属于 `NPUW4A4Int4DynamicMoEMethod`）实现了 `apply_without_routing_weights`。 它主要通过调用 `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_grouped_matmul`, `layer.w2_weight_scale.to` 来完成任务。 状态更新主要写入 `hidden_states`。

### Lines 512-512: Class `NPUW8A8Int8DynamicMoEMethod` declaration / 类 `NPUW8A8Int8DynamicMoEMethod` 声明
```python
class NPUW8A8Int8DynamicMoEMethod(_NPUFusedMoEMethodBase):
```
**EN:** This class establishes `NPUW8A8Int8DynamicMoEMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPUFusedMoEMethodBase`. Its core interface includes methods such as `process_weights_after_loading`, `apply`, `apply_without_routing_weights`.
**CN:** 该类将 `NPUW8A8Int8DynamicMoEMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPUFusedMoEMethodBase`。 其核心接口包括 `process_weights_after_loading`, `apply`, `apply_without_routing_weights` 等方法。

### Lines 514-542: Method `NPUW8A8Int8DynamicMoEMethod.process_weights_after_loading` / 方法 `NPUW8A8Int8DynamicMoEMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        layer.w13_weight.data = npu_format_cast(layer.w13_weight.data.transpose(1, 2))
        layer.w2_weight.data = npu_format_cast(layer.w2_weight.data.transpose(1, 2))
        layer.w13_weight_scale = torch.nn.Parameter(
            layer.w13_weight_scale.data.squeeze(-1), requires_grad=False
        )
        layer.w2_weight_scale = torch.nn.Parameter(
            layer.w2_weight_scale.data.squeeze(-1), requires_grad=False
        )
        layer.w13_weight_scale_bf16 = torch.nn.Parameter(
            layer.w13_weight_scale.data.to(dtype=torch.bfloat16), requires_grad=False
        )
        layer.w2_weight_scale_bf16 = torch.nn.Parameter(
            layer.w2_weight_scale.data.to(dtype=torch.bfloat16), requires_grad=False
        )
        # Compressed-tensors format doesn't have this field
        if hasattr(layer, "w13_weight_offset"):
            layer.w13_weight_offset = torch.nn.Parameter(
                layer.w13_weight_offset.data.squeeze(-1),
                requires_grad=False,
            )
        if hasattr(layer, "w2_weight_offset"):
            layer.w2_weight_offset = torch.nn.Parameter(
                layer.w2_weight_offset.data.squeeze(-1),
                requires_grad=False,
            )

        if hasattr(layer, "dispatcher"):
            layer.dispatcher.set_quant_config({"dispatcher_output_dtype": "int8"})
```
**EN:** This method implements `process_weights_after_loading` on `NPUW8A8Int8DynamicMoEMethod`. It primarily calls `npu_format_cast`, `torch.nn.Parameter`, `hasattr`, `layer.w13_weight.data.transpose`, `layer.w2_weight.data.transpose`, `layer.w13_weight_scale.data.squeeze` to complete its work. State updates are written into `layer.w13_weight.data`, `layer.w2_weight.data`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_weight_scale_bf16`, `layer.w2_weight_scale_bf16`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8DynamicMoEMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `npu_format_cast`, `torch.nn.Parameter`, `hasattr`, `layer.w13_weight.data.transpose`, `layer.w2_weight.data.transpose`, `layer.w13_weight_scale.data.squeeze` 来完成任务。 状态更新主要写入 `layer.w13_weight.data`, `layer.w2_weight.data`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_weight_scale_bf16`, `layer.w2_weight_scale_bf16`。 实现中使用了条件分支。

### Lines 544-587: Method `NPUW8A8Int8DynamicMoEMethod.apply` / 方法 `NPUW8A8Int8DynamicMoEMethod.apply`
```python
    def apply(
        self,
        layer,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        # release fp32 scale to save memory
        layer.w13_weight_scale = None
        layer.w2_weight_scale = None

        hidden_states = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        topk_weights, topk_ids, _ = topk_output
        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(hidden_states.dtype)

        # prefill
        if not torch.npu.is_current_stream_capturing():
            output = npu_fused_experts(
                hidden_states=hidden_states,
                w13=layer.w13_weight,
                w13_scale=layer.w13_weight_scale_bf16,
                w2=layer.w2_weight,
                w2_scale=layer.w2_weight_scale_bf16,
                topk_weights=topk_weights,
                topk_ids=topk_ids,
# ... omitted for brevity ...
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                top_k=topk_ids.shape[1],
            )

        return StandardCombineInput(hidden_states=output)
```
**EN:** This method implements `apply` on `NPUW8A8Int8DynamicMoEMethod`. It primarily calls `topk_ids.to`, `topk_weights.to`, `StandardCombineInput`, `torch.npu.is_current_stream_capturing`, `npu_fused_experts`, `npu_fused_experts_w8a8_decode` to complete its work. State updates are written into `layer.w13_weight_scale`, `layer.w2_weight_scale`, `hidden_states`, `topk_output`, `topk_ids`, `topk_weights`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW8A8Int8DynamicMoEMethod`）实现了 `apply`。 它主要通过调用 `topk_ids.to`, `topk_weights.to`, `StandardCombineInput`, `torch.npu.is_current_stream_capturing`, `npu_fused_experts`, `npu_fused_experts_w8a8_decode` 来完成任务。 状态更新主要写入 `layer.w13_weight_scale`, `layer.w2_weight_scale`, `hidden_states`, `topk_output`, `topk_ids`, `topk_weights`。 实现中使用了条件分支。

### Lines 589-634: Method `NPUW8A8Int8DynamicMoEMethod.apply_without_routing_weights` / 方法 `NPUW8A8Int8DynamicMoEMethod.apply_without_routing_weights`
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        # gmm1: gate_up_proj
        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[layer.w13_weight],
            split_item=2,
            group_list_type=group_list_type,
            group_type=0,
            group_list=group_list,
            output_dtype=torch.int32,
        )[0]

        # act_fn: swiglu
        hidden_states, swiglu_out_scale = torch.ops.npu.npu_dequant_swiglu_quant(
            x=hidden_states,
            weight_scale=layer.w13_weight_scale,
            activation_scale=hidden_states_scale,
            bias=None,
            quant_scale=None,
            quant_offset=None,
# ... omitted for brevity ...
            group_list_type=group_list_type,
            group_type=0,
            group_list=group_list,
            output_dtype=output_dtype,
        )[0]
        return hidden_states
```
**EN:** This method implements `apply_without_routing_weights` on `NPUW8A8Int8DynamicMoEMethod`. It primarily calls `torch.ops.npu.npu_dequant_swiglu_quant`, `torch.ops.npu.npu_grouped_matmul`, `layer.w2_weight_scale.to` to complete its work. State updates are written into `hidden_states`.
**CN:** 该方法（属于 `NPUW8A8Int8DynamicMoEMethod`）实现了 `apply_without_routing_weights`。 它主要通过调用 `torch.ops.npu.npu_dequant_swiglu_quant`, `torch.ops.npu.npu_grouped_matmul`, `layer.w2_weight_scale.to` 来完成任务。 状态更新主要写入 `hidden_states`。

### Lines 637-637: Class `NPUW4A8Int8DynamicMoEMethod` declaration / 类 `NPUW4A8Int8DynamicMoEMethod` 声明
```python
class NPUW4A8Int8DynamicMoEMethod(_NPUFusedMoEMethodBase):
```
**EN:** This class establishes `NPUW4A8Int8DynamicMoEMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPUFusedMoEMethodBase`. Its core interface includes methods such as `_process_scale`, `_update_bias`, `_pack_to_int32`, `process_weights_after_loading`, `_process_weights_without_clip`, `_process_weights_with_clip`.
**CN:** 该类将 `NPUW4A8Int8DynamicMoEMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPUFusedMoEMethodBase`。 其核心接口包括 `_process_scale`, `_update_bias`, `_pack_to_int32`, `process_weights_after_loading`, `_process_weights_without_clip`, `_process_weights_with_clip` 等方法。

### Lines 639-672: Method `NPUW4A8Int8DynamicMoEMethod._process_scale` / 方法 `NPUW4A8Int8DynamicMoEMethod._process_scale`
```python
    def _process_scale(
        self, weight: torch.Tensor, scale, per_group_scale, is_per_channel_weight
    ):
        scale = scale.transpose(1, 2).contiguous()

        if is_per_channel_weight:
            scale_np = scale.cpu().numpy()
            scale_np.dtype = np.uint32
            scale_uint64_tensor = torch.from_numpy(scale_np.astype(np.int64)).npu()
            return scale_uint64_tensor, None

        per_group_scale = per_group_scale.transpose(1, 2).contiguous()
        group_num, k, n = weight.shape
        # the weight of the new version is reduced by half by pack n, so it needs to be restored
        n = n * 2
        per_group_scale = per_group_scale.reshape(group_num, -1, n)
        group_num, quantgroup_num, n = per_group_scale.shape
        bias = None

        scale_fp32 = (scale * per_group_scale).to(torch.float16).to(torch.float32)
        scale_fp32_np = scale_fp32.cpu().numpy()
        scale_fp32_np.dtype = np.uint32
        sscale_uint64 = np.zeros((group_num, quantgroup_num, n * 2), dtype=np.uint32)

        sscale_uint64[..., ::2] = scale_fp32_np

        sscale_uint64_buffer = np.frombuffer(
            sscale_uint64.tobytes(), dtype=np.int64
        ).copy()
        sscale_uint64_tensor = torch.from_numpy(sscale_uint64_buffer).reshape(
            group_num, quantgroup_num, n
        )
        sscale_uint64_tensor = sscale_uint64_tensor.npu()
        return sscale_uint64_tensor, bias
```
**EN:** This method implements `_process_scale` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `scale.transpose.contiguous`, `per_group_scale.transpose.contiguous`, `per_group_scale.reshape`, `to.to`, `scale_fp32.cpu.numpy`, `np.zeros` to complete its work. State updates are written into `scale`, `per_group_scale`, `n`, `bias`, `scale_fp32`, `scale_fp32_np`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `_process_scale`。 它主要通过调用 `scale.transpose.contiguous`, `per_group_scale.transpose.contiguous`, `per_group_scale.reshape`, `to.to`, `scale_fp32.cpu.numpy`, `np.zeros` 来完成任务。 状态更新主要写入 `scale`, `per_group_scale`, `n`, `bias`, `scale_fp32`, `scale_fp32_np`。 实现中使用了条件分支。

### Lines 674-680: Method `NPUW4A8Int8DynamicMoEMethod._update_bias` / 方法 `NPUW4A8Int8DynamicMoEMethod._update_bias`
```python
    def _update_bias(self, layer, w13_bias, w2_bias):
        layer.w13_scale_bias.data = (
            layer.w13_scale_bias.data.transpose(1, 2).contiguous().sum(axis=1)
        )
        layer.w2_scale_bias.data = (
            layer.w2_scale_bias.data.transpose(1, 2).contiguous().sum(axis=1)
        )
```
**EN:** This method implements `_update_bias` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `layer.w13_scale_bias.data.transpose.contiguous.sum`, `layer.w2_scale_bias.data.transpose.contiguous.sum`, `layer.w13_scale_bias.data.transpose.contiguous`, `layer.w2_scale_bias.data.transpose.contiguous`, `layer.w13_scale_bias.data.transpose`, `layer.w2_scale_bias.data.transpose` to complete its work. State updates are written into `layer.w13_scale_bias.data`, `layer.w2_scale_bias.data`.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `_update_bias`。 它主要通过调用 `layer.w13_scale_bias.data.transpose.contiguous.sum`, `layer.w2_scale_bias.data.transpose.contiguous.sum`, `layer.w13_scale_bias.data.transpose.contiguous`, `layer.w2_scale_bias.data.transpose.contiguous`, `layer.w13_scale_bias.data.transpose`, `layer.w2_scale_bias.data.transpose` 来完成任务。 状态更新主要写入 `layer.w13_scale_bias.data`, `layer.w2_scale_bias.data`。

### Lines 682-687: Method `NPUW4A8Int8DynamicMoEMethod._pack_to_int32` / 方法 `NPUW4A8Int8DynamicMoEMethod._pack_to_int32`
```python
    def _pack_to_int32(self, weight: torch.Tensor):
        # pack 4 int8(int4*2) to int32, because in pytorch, we need to use int32 to represent int4
        assert (
            weight.shape[-1] % 4 == 0
        ), "the last dim of weight needs to be divided by 4"
        return weight.view(torch.int32).contiguous()
```
**EN:** This method implements `_pack_to_int32` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `weight.view.contiguous`, `weight.view` to complete its work.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `_pack_to_int32`。 它主要通过调用 `weight.view.contiguous`, `weight.view` 来完成任务。

### Lines 689-711: Method `NPUW4A8Int8DynamicMoEMethod.process_weights_after_loading` / 方法 `NPUW4A8Int8DynamicMoEMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(
        self, layer: torch.nn.Module, is_per_channel_weight, activation_use_clip
    ) -> None:
        if not activation_use_clip:
            self._process_weights_without_clip(layer, is_per_channel_weight)
        else:
            self._process_weights_with_clip(layer)

        layer.w13_weight = torch.nn.Parameter(
            layer.w13_weight.data.transpose(1, 2).contiguous(), requires_grad=False
        )
        layer.w2_weight = torch.nn.Parameter(
            layer.w2_weight.data.transpose(1, 2).contiguous(), requires_grad=False
        )

        layer.w13_weight.data = npu_format_cast(layer.w13_weight.data)
        layer.w2_weight.data = npu_format_cast(layer.w2_weight.data)

        layer.w13_weight.data = self._pack_to_int32(layer.w13_weight.data)
        layer.w2_weight.data = self._pack_to_int32(layer.w2_weight.data)

        if hasattr(layer, "dispatcher"):
            layer.dispatcher.set_quant_config({"dispatcher_output_dtype": "int8"})
```
**EN:** This method implements `process_weights_after_loading` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `torch.nn.Parameter`, `npu_format_cast`, `self._pack_to_int32`, `hasattr`, `self._process_weights_without_clip`, `self._process_weights_with_clip` to complete its work. State updates are written into `layer.w13_weight`, `layer.w2_weight`, `layer.w13_weight.data`, `layer.w2_weight.data`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `torch.nn.Parameter`, `npu_format_cast`, `self._pack_to_int32`, `hasattr`, `self._process_weights_without_clip`, `self._process_weights_with_clip` 来完成任务。 状态更新主要写入 `layer.w13_weight`, `layer.w2_weight`, `layer.w13_weight.data`, `layer.w2_weight.data`。 实现中使用了条件分支。

### Lines 713-745: Method `NPUW4A8Int8DynamicMoEMethod._process_weights_without_clip` / 方法 `NPUW4A8Int8DynamicMoEMethod._process_weights_without_clip`
```python
    def _process_weights_without_clip(
        self, layer: torch.nn.Module, is_per_channel_weight
    ) -> None:
        w13_weight_scale_second = (
            layer.w13_weight_scale_second.data
            if hasattr(layer, "w13_weight_scale_second")
            else None
        )
        w2_weight_scale_second = (
            layer.w2_weight_scale_second.data
            if hasattr(layer, "w2_weight_scale_second")
            else None
        )
        layer.w13_weight_scale.data, w13_bias = self._process_scale(
            layer.w13_weight,
            layer.w13_weight_scale.data,
            w13_weight_scale_second,
            is_per_channel_weight,
        )
        layer.w2_weight_scale.data, w2_bias = self._process_scale(
            layer.w2_weight,
            layer.w2_weight_scale.data,
            w2_weight_scale_second,
            is_per_channel_weight,
        )
        if hasattr(layer, "w13_weight_scale_second"):
            # scale_second is no longer used, release this part of the memory
            del layer.w13_weight_scale_second
            del layer.w2_weight_scale_second
            del layer.w13_weight_offset_second
            del layer.w2_weight_offset_second

        self._update_bias(layer, w13_bias, w2_bias)
```
**EN:** This method implements `_process_weights_without_clip` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `self._process_scale`, `hasattr`, `self._update_bias` to complete its work. State updates are written into `w13_weight_scale_second`, `w2_weight_scale_second`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `_process_weights_without_clip`。 它主要通过调用 `self._process_scale`, `hasattr`, `self._update_bias` 来完成任务。 状态更新主要写入 `w13_weight_scale_second`, `w2_weight_scale_second`。 实现中使用了条件分支。

### Lines 747-759: Method `NPUW4A8Int8DynamicMoEMethod._process_weights_with_clip` / 方法 `NPUW4A8Int8DynamicMoEMethod._process_weights_with_clip`
```python
    def _process_weights_with_clip(self, layer: torch.nn.Module) -> None:
        w13_weight_scale = (
            layer.w13_weight_scale.data.squeeze(-1).contiguous().unsqueeze(1)
        )
        w2_weight_scale = (
            layer.w2_weight_scale.data.squeeze(-1).contiguous().unsqueeze(1)
        )
        layer.w13_weight_scale = torch.nn.Parameter(
            w13_weight_scale, requires_grad=False
        )
        layer.w2_weight_scale = torch.nn.Parameter(w2_weight_scale, requires_grad=False)
        layer.w13_scale_bias = layer.w13_bias
        layer.w2_scale_bias = layer.w2_bias
```
**EN:** This method implements `_process_weights_with_clip` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `layer.w13_weight_scale.data.squeeze.contiguous.unsqueeze`, `layer.w2_weight_scale.data.squeeze.contiguous.unsqueeze`, `torch.nn.Parameter`, `layer.w13_weight_scale.data.squeeze.contiguous`, `layer.w2_weight_scale.data.squeeze.contiguous`, `layer.w13_weight_scale.data.squeeze` to complete its work. State updates are written into `w13_weight_scale`, `w2_weight_scale`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_scale_bias`, `layer.w2_scale_bias`.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `_process_weights_with_clip`。 它主要通过调用 `layer.w13_weight_scale.data.squeeze.contiguous.unsqueeze`, `layer.w2_weight_scale.data.squeeze.contiguous.unsqueeze`, `torch.nn.Parameter`, `layer.w13_weight_scale.data.squeeze.contiguous`, `layer.w2_weight_scale.data.squeeze.contiguous`, `layer.w13_weight_scale.data.squeeze` 来完成任务。 状态更新主要写入 `w13_weight_scale`, `w2_weight_scale`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_scale_bias`, `layer.w2_scale_bias`。

### Lines 761-843: Method `NPUW4A8Int8DynamicMoEMethod.apply` / 方法 `NPUW4A8Int8DynamicMoEMethod.apply`
```python
    def apply(
        self,
        layer,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        hidden_states = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        topk_weights, topk_ids, _ = topk_output
        top_k = topk_ids.shape[1]
        group_list_type = 1
        original_shape = hidden_states.shape
        topk_weights = topk_weights

        num_tokens = hidden_states.shape[:-1].numel()

        first_expert_idx = 0
        last_expert_idx = layer.num_experts
        global_num_experts = layer.num_experts

        sorted_hidden_states, expanded_row_idx, expert_tokens, pertoken_scale = (
            torch.ops.npu.npu_moe_init_routing_v2(
                hidden_states,
                topk_ids,
                active_num=num_tokens * top_k,
                expert_num=global_num_experts,
# ... omitted for brevity ...
            probs=topk_weights,
        )
        if len(original_shape) == 3:
            final_hidden_states = final_hidden_states.view(original_shape)

        return StandardCombineInput(hidden_states=final_hidden_states)
```
**EN:** This method implements `apply` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `hidden_states.shape.numel`, `torch.ops.npu.npu_moe_init_routing_v2`, `expert_tokens.to`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_moe_token_unpermute` to complete its work. State updates are written into `hidden_states`, `topk_output`, `top_k`, `group_list_type`, `original_shape`, `topk_weights`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `apply`。 它主要通过调用 `hidden_states.shape.numel`, `torch.ops.npu.npu_moe_init_routing_v2`, `expert_tokens.to`, `torch.ops.npu.npu_swiglu`, `torch.ops.npu.npu_dynamic_quant`, `torch.ops.npu.npu_moe_token_unpermute` 来完成任务。 状态更新主要写入 `hidden_states`, `topk_output`, `top_k`, `group_list_type`, `original_shape`, `topk_weights`。 实现中使用了条件分支。

### Lines 845-886: Method `NPUW4A8Int8DynamicMoEMethod.apply_without_routing_weights` / 方法 `NPUW4A8Int8DynamicMoEMethod.apply_without_routing_weights`
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        from sgl_kernel_npu.activation.swiglu_quant import swiglu_quant

        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[layer.w13_weight],
            scale=[layer.w13_weight_scale],
            bias=[layer.w13_scale_bias],
            per_token_scale=[hidden_states_scale],
            group_list=group_list,
            split_item=2,
            group_type=0,
            group_list_type=group_list_type,
            output_dtype=output_dtype,
        )[0]

        hidden_states, swiglu_out_scale = swiglu_quant(
            hidden_states, group_list, group_list_type
        )

# ... omitted for brevity ...
            group_type=0,
            group_list_type=group_list_type,
            output_dtype=output_dtype,
        )[0]

        return hidden_states
```
**EN:** This method implements `apply_without_routing_weights` on `NPUW4A8Int8DynamicMoEMethod`. It primarily calls `swiglu_quant`, `torch.ops.npu.npu_grouped_matmul` to complete its work. State updates are written into `hidden_states`.
**CN:** 该方法（属于 `NPUW4A8Int8DynamicMoEMethod`）实现了 `apply_without_routing_weights`。 它主要通过调用 `swiglu_quant`, `torch.ops.npu.npu_grouped_matmul` 来完成任务。 状态更新主要写入 `hidden_states`。

### Lines 889-889: Class `NPUW4A16Int4DynamicMoEMethod` declaration / 类 `NPUW4A16Int4DynamicMoEMethod` 声明
```python
class NPUW4A16Int4DynamicMoEMethod(_NPUFusedMoEMethodBase):
```
**EN:** This class establishes `NPUW4A16Int4DynamicMoEMethod` as the main container/coordinator for the surrounding logic. It inherits from `_NPUFusedMoEMethodBase`. Its core interface includes methods such as `_pack_to_int32`, `_unpack_from_int32`, `process_weights_after_loading`, `apply`, `apply_without_routing_weights`.
**CN:** 该类将 `NPUW4A16Int4DynamicMoEMethod` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_NPUFusedMoEMethodBase`。 其核心接口包括 `_pack_to_int32`, `_unpack_from_int32`, `process_weights_after_loading`, `apply`, `apply_without_routing_weights` 等方法。

### Lines 891-910: Method `NPUW4A16Int4DynamicMoEMethod._pack_to_int32` / 方法 `NPUW4A16Int4DynamicMoEMethod._pack_to_int32`
```python
    def _pack_to_int32(self, weight: torch.Tensor):
        assert weight.dim() == 3
        if weight.dtype == torch.int32:
            # pack 8 int4 to int32, we use a int32 to represent a int4
            assert (
                weight.shape[-1] % 8 == 0
            ), "the last dim of weight needs to be divided by 8"
            new_weight = torch.ops.npu.npu_convert_weight_to_int4pack(
                weight.flatten(0, 1)
            )
            new_weight = new_weight.view(weight.shape[0], weight.shape[1], -1)
        elif weight.dtype == torch.int8:
            # pack 4 int8(int4*2) to int32, because in pytorch, we need to use int32 to represent int4
            assert (
                weight.shape[-1] % 4 == 0
            ), "the last dim of weight needs to be divided by 4"
            new_weight = weight.view(torch.int32).contiguous()
        else:
            raise ValueError(f"{weight.dtype=} is not supported !")
        return new_weight
```
**EN:** This method implements `_pack_to_int32` on `NPUW4A16Int4DynamicMoEMethod`. It primarily calls `weight.dim`, `torch.ops.npu.npu_convert_weight_to_int4pack`, `new_weight.view`, `weight.flatten`, `weight.view.contiguous`, `ValueError` to complete its work. State updates are written into `new_weight`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A16Int4DynamicMoEMethod`）实现了 `_pack_to_int32`。 它主要通过调用 `weight.dim`, `torch.ops.npu.npu_convert_weight_to_int4pack`, `new_weight.view`, `weight.flatten`, `weight.view.contiguous`, `ValueError` 来完成任务。 状态更新主要写入 `new_weight`。 实现中使用了条件分支。

### Lines 912-974: Method `NPUW4A16Int4DynamicMoEMethod._unpack_from_int32` / 方法 `NPUW4A16Int4DynamicMoEMethod._unpack_from_int32`
```python
    def _unpack_from_int32(
        self,
        value: torch.Tensor,
        num_bits: int,
        shape: torch.Size = None,
        packed_dim=1,
    ) -> torch.Tensor:
        """
        Unpacks a tensor of packed int32 weights into individual int8s, maintaining the
        original bit range.

        Return tensors in int8

        :param value: tensor to unpack
        :param num_bits: number of bits to unpack each data point into
        :param shape: shape to unpack into, used to remove padding
        :returns: unpacked int8 tensor
        """
        if value.dtype is not torch.int32:
            raise ValueError(
                f"Expected {torch.int32} but got {value.dtype}, Aborting unpack."
            )

        if num_bits > 8:
            raise ValueError("Unpacking is only supported for less than 8 bits")

        pack_factor = 32 // num_bits

# ... omitted for brevity ...
        # bits are packed in unsigned format, reformat to signed
        # update the value range from unsigned to signed
        offset = pow(2, num_bits) // 2
        unpacked = (unpacked - offset).to(torch.int8)

        return unpacked
```
**EN:** This method implements `_unpack_from_int32` on `NPUW4A16Int4DynamicMoEMethod`. It primarily calls `to`, `ValueError`, `torch.zeros`, `range`, `int`, `pow` to complete its work. State updates are written into `pack_factor`, `mask`, `offset`, `unpacked`, `original_row_size`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `NPUW4A16Int4DynamicMoEMethod`）实现了 `_unpack_from_int32`。 它主要通过调用 `to`, `ValueError`, `torch.zeros`, `range`, `int`, `pow` 来完成任务。 状态更新主要写入 `pack_factor`, `mask`, `offset`, `unpacked`, `original_row_size`。 实现中使用了条件分支、迭代逻辑。

### Lines 976-1018: Method `NPUW4A16Int4DynamicMoEMethod.process_weights_after_loading` / 方法 `NPUW4A16Int4DynamicMoEMethod.process_weights_after_loading`
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        w13_weight_scale = layer.w13_weight_scale.data.transpose(-1, -2).contiguous()
        w2_weight_scale = layer.w2_weight_scale.data.transpose(-1, -2).contiguous()
        layer.w13_weight_scale = torch.nn.Parameter(
            w13_weight_scale, requires_grad=False
        )
        layer.w2_weight_scale = torch.nn.Parameter(w2_weight_scale, requires_grad=False)

        layer.w13_weight_offset = torch.nn.Parameter(
            layer.w13_weight_offset.data.transpose(-1, -2).contiguous(),
            requires_grad=False,
        )
        layer.w2_weight_offset = torch.nn.Parameter(
            layer.w2_weight_offset.data.transpose(-1, -2).contiguous(),
            requires_grad=False,
        )

        # w = [n, k // 8]  --> [k, n // 8]
        # w13_weight = layer.w13_weight.data.transpose(1, 2).contiguous()
        # w2_weight = layer.w2_weight.data.transpose(1, 2).contiguous()
        unpacked_w13_weight = (
            self._unpack_from_int32(layer.w13_weight.data.flatten(0, 1), 4)
            .view(layer.w13_weight.data.shape[0], layer.w13_weight.data.shape[1], -1)
            .transpose(1, 2)
            .contiguous()
            .int()
        )
        unpacked_w2_weight = (
# ... omitted for brevity ...

        layer.w13_weight = torch.nn.Parameter(w13_weight, requires_grad=False)
        layer.w2_weight = torch.nn.Parameter(w2_weight, requires_grad=False)

        if hasattr(layer, "dispatcher"):
            layer.dispatcher.set_quant_config({"dispatcher_output_dtype": "bf16"})
```
**EN:** This method implements `process_weights_after_loading` on `NPUW4A16Int4DynamicMoEMethod`. It primarily calls `layer.w13_weight_scale.data.transpose.contiguous`, `layer.w2_weight_scale.data.transpose.contiguous`, `torch.nn.Parameter`, `self._unpack_from_int32.view.transpose.contiguous.int`, `self._pack_to_int32`, `hasattr` to complete its work. State updates are written into `w13_weight_scale`, `w2_weight_scale`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_weight_offset`, `layer.w2_weight_offset`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A16Int4DynamicMoEMethod`）实现了 `process_weights_after_loading`。 它主要通过调用 `layer.w13_weight_scale.data.transpose.contiguous`, `layer.w2_weight_scale.data.transpose.contiguous`, `torch.nn.Parameter`, `self._unpack_from_int32.view.transpose.contiguous.int`, `self._pack_to_int32`, `hasattr` 来完成任务。 状态更新主要写入 `w13_weight_scale`, `w2_weight_scale`, `layer.w13_weight_scale`, `layer.w2_weight_scale`, `layer.w13_weight_offset`, `layer.w2_weight_offset`。 实现中使用了条件分支。

### Lines 1020-1046: Method `NPUW4A16Int4DynamicMoEMethod.apply` / 方法 `NPUW4A16Int4DynamicMoEMethod.apply`
```python
    def apply(
        self,
        layer,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        topk_weights, topk_ids, _ = topk_output
        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(x.dtype)
        output = npu_fused_experts(
            hidden_states=x,
            w13=layer.w13_weight,
            w13_scale=layer.w13_weight_scale,
            w13_offset=layer.w13_weight_offset,
            w2=layer.w2_weight,
            w2_scale=layer.w2_weight_scale,
            w2_offset=layer.w2_weight_offset,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            top_k=topk_ids.shape[1],
            use_wna16=True,
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This method implements `apply` on `NPUW4A16Int4DynamicMoEMethod`. It primarily calls `topk_ids.to`, `topk_weights.to`, `npu_fused_experts`, `StandardCombineInput` to complete its work. State updates are written into `x`, `topk_output`, `topk_ids`, `topk_weights`, `output`.
**CN:** 该方法（属于 `NPUW4A16Int4DynamicMoEMethod`）实现了 `apply`。 它主要通过调用 `topk_ids.to`, `topk_weights.to`, `npu_fused_experts`, `StandardCombineInput` 来完成任务。 状态更新主要写入 `x`, `topk_output`, `topk_ids`, `topk_weights`, `output`。

### Lines 1048-1091: Method `NPUW4A16Int4DynamicMoEMethod.apply_without_routing_weights` / 方法 `NPUW4A16Int4DynamicMoEMethod.apply_without_routing_weights`
```python
    def apply_without_routing_weights(
        self,
        layer,
        hidden_states,
        hidden_states_scale,
        group_list_type,
        group_list,
        output_dtype,
    ):
        if hidden_states_scale is None:
            # gmm1: gate_up_proj
            hidden_states = torch.ops.npu.npu_grouped_matmul(
                x=[hidden_states],
                weight=[layer.w13_weight],
                antiquant_scale=[layer.w13_weight_scale],
                antiquant_offset=[layer.w13_weight_offset],
                split_item=2,
                group_list_type=group_list_type,
                group_type=0,
                group_list=group_list,
                output_dtype=output_dtype,
            )[0]

            # act_fn: swiglu
            hidden_states = torch.ops.npu.npu_swiglu(hidden_states)

            # gmm2: down_proj
            out_hidden = torch.ops.npu.npu_grouped_matmul(
# ... omitted for brevity ...
        else:
            raise ValueError(
                "when weight is int4, hidden_states only supports non-quant dtype!"
            )

        return out_hidden
```
**EN:** This method implements `apply_without_routing_weights` on `NPUW4A16Int4DynamicMoEMethod`. It primarily calls `torch.ops.npu.npu_swiglu`, `ValueError`, `torch.ops.npu.npu_grouped_matmul` to complete its work. State updates are written into `hidden_states`, `out_hidden`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUW4A16Int4DynamicMoEMethod`）实现了 `apply_without_routing_weights`。 它主要通过调用 `torch.ops.npu.npu_swiglu`, `ValueError`, `torch.ops.npu.npu_grouped_matmul` 来完成任务。 状态更新主要写入 `hidden_states`, `out_hidden`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `_NPUFusedMoEMethodBase`, `NPUW4A4Int4DynamicMoEMethod`, `NPUW8A8Int8DynamicMoEMethod`, `NPUW4A8Int8DynamicMoEMethod`, `NPUW4A16Int4DynamicMoEMethod`
- **Functions / 函数**: `npu_fused_experts_w4a4`, `npu_fused_experts`, `npu_fused_experts_w8a8_decode`, `npu_fused_moe_without_routing_weights_bf16`, `fused_moe_npu`, `__init__`, `process_weights_after_loading`, `_pack_to_int32`
- **Themes / 主题**: `quant`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.npu.utils`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.activation`
- **External / 外部依赖**: `numpy`, `torch`, `sgl_kernel_npu.activation.swiglu_quant`
- **Standard library / 标准库**: `typing`
