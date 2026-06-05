# humming_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/humming_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-17)
```python
from typing import Any

import regex as re
import torch
from humming.layer import HummingInputSchema, HummingMethod
from humming.schema import BaseWeightSchema

from vllm import envs
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEQuantConfig,
    FusedMoEQuantDesc,
)
from vllm.model_executor.layers.fused_moe.routed_experts import RoutedExperts
from vllm.model_executor.layers.linear import LinearBase
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
```
**EN:** This opening block pulls in external dependencies such as `typing`, `regex`, `torch`, `humming` and internal modules such as `vllm`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.routed_experts`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `typing`, `regex`, `torch`, `humming`）以及内部模块（如 `vllm`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.routed_experts`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `humming_is_layer_skipped` (lines 20-42)
```python
def humming_is_layer_skipped(config: dict[str, Any], prefix: str):
    if not config:
        return True

    keys = ["ignored_layers", "ignore", "modules_to_not_convert"]
    ignored_layers: list[str] = []
    for key in keys:
        ignored_layers = config.get(key, []) or []
        if not ignored_layers:
            break

    if any(module_name in prefix for module_name in ignored_layers):
        return True
    if "lm_head" in prefix:
        return True

    for regex in config.get("dynamic", {}):
        if regex[:1] != "-":
            continue
        if re.match(regex[2:], prefix):
            return True

    return False
```
**EN:** Defines function `humming_is_layer_skipped` with signature `humming_is_layer_skipped(config: dict[str, Any], prefix: str)`. It mainly works with `config`, `prefix`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, comprehensions. Key calls include `any`, `config.get`, `re.match`.
**CN:** 定义函数 `humming_is_layer_skipped`，其签名为 `humming_is_layer_skipped(config: dict[str, Any], prefix: str)`。它主要围绕 `config`, `prefix` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、推导式。关键调用包括 `any`, `config.get`, `re.match`。

### Function `prepare_humming_layer` (lines 45-82)
```python
def prepare_humming_layer(layer: LinearBase, quant_config: dict):
    weight_schema = BaseWeightSchema.from_config(quant_config)
    input_schema = HummingInputSchema()

    shape_k_stacks = [layer.input_size_per_partition]
    shape_n_stacks = layer.output_partition_sizes

    # Step 1: convert weight to humming standard format
    weight_schema, tensors = weight_schema.convert_humming(
        tensors=layer.named_parameters(),
        shape_n_stacks=shape_n_stacks,
        shape_k_stacks=shape_k_stacks,
        param_dtype=layer.params_dtype,
    )

    layer.weight_schema = weight_schema

    for name, _ in list(layer.named_parameters()):
        delattr(layer, name)

    for name, tensor in tensors.items():
        param = torch.nn.Parameter(tensor, requires_grad=False)
        setattr(layer, name, param)

    # Step 2: transform weight (humming standard format) for forwarding
    HummingMethod.prepare_layer_meta(
        layer=layer,
        shape_n=layer.output_partition_sizes_sum,
        shape_k=layer.input_size_per_partition,
        weight_schema=weight_schema,
        input_schema=input_schema,
        pad_n_to_multiple=256,
        pad_k_to_multiple=128,
        has_bias=layer.has_bias,
        torch_dtype=layer.param_dtype,
    )

    HummingMethod.transform_humming_layer(layer)
```
**EN:** Defines function `prepare_humming_layer` with signature `prepare_humming_layer(layer: LinearBase, quant_config: dict)`. It mainly works with `layer`, `quant_config`; constructs runtime helpers from the current configuration. The body uses iteration, tensor/kernel operations. Key calls include `BaseWeightSchema.from_config`, `HummingInputSchema`, `weight_schema.convert_humming`, `list`, `tensors.items`, `HummingMethod.prepare_layer_meta`.
**CN:** 定义函数 `prepare_humming_layer`，其签名为 `prepare_humming_layer(layer: LinearBase, quant_config: dict)`。它主要围绕 `layer`, `quant_config` 展开；根据当前配置构建运行时辅助对象。函数体包含循环处理、张量或内核操作。关键调用包括 `BaseWeightSchema.from_config`, `HummingInputSchema`, `weight_schema.convert_humming`, `list`, `tensors.items`, `HummingMethod.prepare_layer_meta`。

### Function `prepare_humming_moe_layer` (lines 85-164)
```python
def prepare_humming_moe_layer(layer: RoutedExperts, quant_config: dict):
    weight_schema = BaseWeightSchema.from_config(quant_config)
    input_quant_config = envs.VLLM_HUMMING_INPUT_QUANT_CONFIG or {}
    if humming_is_layer_skipped(input_quant_config, layer.layer_name):
        input_schema = HummingInputSchema()
    else:
        # TODO: read input_quant_config from quant_config
        input_schema = HummingInputSchema.from_config(input_quant_config)

    is_gated = layer.activation.is_gated
    shape_config = {
        "w13": (
            layer.moe_config.intermediate_size_per_partition * 2,
            layer.moe_config.hidden_dim,
        ),
        "w2": (
            layer.moe_config.hidden_dim,
            layer.moe_config.intermediate_size_per_partition * (1 if is_gated else 2),
        ),
    }

    layer.weight_schemas = {}
    layer.input_schemas = {}

    for sublayer_name in shape_config:
        # Step 1: convert weight to humming standard format
        tensors: dict[str, torch.Tensor] = dict(
            (key.removeprefix(sublayer_name + "_"), value)
            for key, value in layer.state_dict().items()
            if key.startswith(sublayer_name + "_")
        )

        shape_n, shape_k = shape_config[sublayer_name]
        shape_n_stacks = [shape_n]
        shape_k_stacks = [shape_k]
        if sublayer_name == "w13":
            shape_n_stacks = [shape_n // 2] * 2

# ... truncated for analysis ...
            input_schema=input_schema,
            weight_schema=weight_schema_new,
            has_bias=layer.moe_config.has_bias,
            num_experts=layer.num_experts,
            torch_dtype=layer.params_dtype,
            sublayer_name=sublayer_name,
        )

        HummingMethod.transform_humming_layer(layer, sublayer_name=sublayer_name)

    if not hasattr(layer, "locks"):
        device = layer.w13_weight.device
        locks = torch.zeros(1024, dtype=torch.int32, device=device)
        layer.register_buffer("locks", locks)
```
**EN:** Defines function `prepare_humming_moe_layer` with signature `prepare_humming_moe_layer(layer: RoutedExperts, quant_config: dict)`. It mainly works with `layer`, `quant_config`; constructs runtime helpers from the current configuration. The body uses branching, iteration, comprehensions, tensor/kernel operations. Key calls include `BaseWeightSchema.from_config`, `humming_is_layer_skipped`, `HummingInputSchema`, `HummingInputSchema.from_config`, `dict`, `weight_schema.convert_humming`.
**CN:** 定义函数 `prepare_humming_moe_layer`，其签名为 `prepare_humming_moe_layer(layer: RoutedExperts, quant_config: dict)`。它主要围绕 `layer`, `quant_config` 展开；根据当前配置构建运行时辅助对象。函数体包含分支判断、循环处理、推导式、张量或内核操作。关键调用包括 `BaseWeightSchema.from_config`, `humming_is_layer_skipped`, `HummingInputSchema`, `HummingInputSchema.from_config`, `dict`, `weight_schema.convert_humming`。

### Function `get_humming_moe_quant_config` (lines 167-214)
```python
def get_humming_moe_quant_config(layer: RoutedExperts):
    input_schema = layer.input_schemas["w13"]
    weight_schema = layer.weight_schemas["w13"]

    a_dtype = input_schema.a_dtype
    if a_dtype is None or a_dtype.num_bits == 16:
        a_quant_desc = FusedMoEQuantDesc(dtype=None)
    else:
        shape = GroupShape(row=1, col=-1)
        a_quant_desc = FusedMoEQuantDesc(dtype=str(a_dtype), shape=shape)

    weight_scale_group_size = weight_schema.weight_scale_group_size
    weight_scale_group_size_n = weight_schema.weight_scale_group_size_n
    weight_group_shape: tuple[int, ...] = ()
    if weight_scale_group_size_n > 1:
        weight_group_shape = GroupShape(
            row=weight_scale_group_size,
            col=weight_scale_group_size_n,
        )
    elif weight_scale_group_size == 0:
        weight_group_shape = GroupShape(row=-1, col=1)
    else:
        weight_group_shape = GroupShape(row=weight_scale_group_size, col=1)

    w1_quant_desc = FusedMoEQuantDesc(
        dtype=str(weight_schema.b_dtype),
        shape=weight_group_shape,
        scale=getattr(layer, "w13_weight_scale", None),
        alpha_or_gscale=getattr(layer, "w13_global_scale", None),
        zp=getattr(layer, "w13_zero_point", None),
        bias=getattr(layer, "w13_bias", None),
    )

    w2_quant_desc = FusedMoEQuantDesc(
        dtype=str(weight_schema.b_dtype),
        shape=weight_group_shape,
        scale=getattr(layer, "w2_weight_scale", None),
        alpha_or_gscale=getattr(layer, "w2_global_scale", None),
        zp=getattr(layer, "w2_zero_point", None),
        bias=getattr(layer, "w2_bias", None),
    )

    return FusedMoEQuantConfig(
        _a1=a_quant_desc,
        _a2=a_quant_desc,
        _w1=w1_quant_desc,
        _w2=w2_quant_desc,
    )
```
**EN:** Defines function `get_humming_moe_quant_config` with signature `get_humming_moe_quant_config(layer: RoutedExperts)`. It mainly works with `layer`; handles quantization-related transformation logic. The body uses branching. Key calls include `FusedMoEQuantDesc`, `FusedMoEQuantConfig`, `GroupShape`, `str`, `getattr`.
**CN:** 定义函数 `get_humming_moe_quant_config`，其签名为 `get_humming_moe_quant_config(layer: RoutedExperts)`。它主要围绕 `layer` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 `FusedMoEQuantDesc`, `FusedMoEQuantConfig`, `GroupShape`, `str`, `getattr`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `humming_is_layer_skipped`, `prepare_humming_layer`, `prepare_humming_moe_layer`, `get_humming_moe_quant_config` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `humming_is_layer_skipped`, `prepare_humming_layer`, `prepare_humming_moe_layer`, `get_humming_moe_quant_config` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `typing`, `regex`, `torch`, `humming`
- **Internal / 内部**: `vllm`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.routed_experts`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`
