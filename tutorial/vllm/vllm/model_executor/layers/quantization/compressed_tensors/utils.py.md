# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/compressed_tensors/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helper utilities used across model-executor layers. / 提供模型执行层共享的辅助工具。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-21)
```python
from collections.abc import Iterable, Mapping
from types import MappingProxyType

import regex as re
from compressed_tensors import CompressionFormat
from compressed_tensors.quantization import QuantizationStrategy
from torch.nn import Module

from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
)
from vllm.model_executor.parameter import (
    BlockQuantScaleParameter,
    ChannelQuantScaleParameter,
    PerTensorScaleParameter,
)
```
**EN:** This opening block pulls in external dependencies such as `collections`, `types`, `regex`, `compressed_tensors`, `torch` and internal modules such as `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `types`, `regex`, `compressed_tensors`, `torch`）以及内部模块（如 `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 25-37)
```python
STRATEGY_TO_PARAMETER_TYPE = {
    QuantizationStrategy.BLOCK: BlockQuantScaleParameter,
    QuantizationStrategy.CHANNEL: ChannelQuantScaleParameter,
    QuantizationStrategy.TENSOR: PerTensorScaleParameter,
}

# Maps quantization strategy to the vLLM weight-quant key used for
# kernel selection.  Shared across compressed-tensor scheme classes.
STRATEGY_TO_WEIGHT_QUANT_KEY = {
    QuantizationStrategy.BLOCK: kFp8Static128BlockSym,
    QuantizationStrategy.CHANNEL: kFp8StaticChannelSym,
    QuantizationStrategy.TENSOR: kFp8StaticTensorSym,
}
```
**EN:** This block defines module-level metadata or constants such as `STRATEGY_TO_PARAMETER_TYPE`, `STRATEGY_TO_WEIGHT_QUANT_KEY`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `STRATEGY_TO_PARAMETER_TYPE`, `STRATEGY_TO_WEIGHT_QUANT_KEY`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `is_activation_quantization_format` (lines 40-47)
```python
def is_activation_quantization_format(format: str) -> bool:
    _ACTIVATION_QUANTIZATION_FORMATS = [
        CompressionFormat.naive_quantized.value,
        CompressionFormat.int_quantized.value,
        CompressionFormat.float_quantized.value,
        CompressionFormat.nvfp4_pack_quantized.value,
    ]
    return format in _ACTIVATION_QUANTIZATION_FORMATS
```
**EN:** Defines function `is_activation_quantization_format` with signature `is_activation_quantization_format(format: str) -> bool`. It mainly works with `format`; handles quantization-related transformation logic. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `is_activation_quantization_format`，其签名为 `is_activation_quantization_format(format: str) -> bool`。它主要围绕 `format` 展开；处理量化相关的变换逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `should_ignore_layer` (lines 50-102)
```python
def should_ignore_layer(
    layer_name: str | None,
    ignore: Iterable[str] = tuple(),
    fused_mapping: Mapping[str, list[str]] = MappingProxyType({}),
) -> bool:
    if layer_name is None:
        return False

    # layer_name = model.layers.0.self_attn.qkv_proj
    # proj_name = qkv_proj
    proj_name = layer_name.split(".")[-1]

    # Fused layers like gate_up_proj or qkv_proj will not be fused
    # in the safetensors checkpoint. So, we convert the name
    # from the fused version to unfused + check to make sure that
    # each shard of the fused layer has the same scheme.
    if proj_name in fused_mapping and layer_name not in ignore:
        shard_proj_names = fused_mapping[proj_name]

        # Convert fused_name --> [shard_names]
        shard_names = [
            layer_name.replace(proj_name, shard_proj_name)
            for shard_proj_name in shard_proj_names
        ]

        # Layer should be ignored if shards are ignored.
        should_ignore_layer = None
        for shard_name in shard_names:
            should_ignore_shard = check_equal_or_regex_match(
                layer_name=shard_name, targets=ignore
            )

            # If shard_idx=0, set layer ignore to match shard.
            if should_ignore_layer is None:
                should_ignore_layer = should_ignore_shard

            # If shard_idx=1+ confirm scheme matches prior shards.
            elif should_ignore_shard != should_ignore_layer:
                raise ValueError(
                    f"Found a different quantization schemes for "
                    f"{shard_proj_names} in {layer_name}. vLLM "
                    "requires all to use the same scheme."
                )

    # Unfused layers like down_proj and o_proj will match
    # the safetensors checkpoint already.
    else:
        should_ignore_layer = check_equal_or_regex_match(
            layer_name=layer_name, targets=ignore
        )

    assert should_ignore_layer is not None
    return should_ignore_layer
```
**EN:** Defines function `should_ignore_layer` with signature `should_ignore_layer(layer_name: str | None, ignore: Iterable[str]=tuple(), fused_mapping: Mapping[str, list[str]]=MappingProxyType({})) -> bool`. It mainly works with `layer_name`, `ignore`, `fused_mapping`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, comprehensions, validation/error handling. Key calls include `tuple`, `MappingProxyType`, `layer_name.split`, `check_equal_or_regex_match`, `layer_name.replace`, `ValueError`.
**CN:** 定义函数 `should_ignore_layer`，其签名为 `should_ignore_layer(layer_name: str | None, ignore: Iterable[str]=tuple(), fused_mapping: Mapping[str, list[str]]=MappingProxyType({})) -> bool`。它主要围绕 `layer_name`, `ignore`, `fused_mapping` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、推导式、校验或报错逻辑。关键调用包括 `tuple`, `MappingProxyType`, `layer_name.split`, `check_equal_or_regex_match`, `layer_name.replace`, `ValueError`。

### Function `check_equal_or_regex_match` (lines 105-110)
```python
def check_equal_or_regex_match(layer_name: str, targets: Iterable[str]) -> bool:
    """
    Checks whether a layer_name is exactly equal or a regex match for
    if target starts with 're:' to any target in list.
    """
    return any(_is_equal_or_regex_match(layer_name, target) for target in targets)
```
**EN:** Defines function `check_equal_or_regex_match` with signature `check_equal_or_regex_match(layer_name: str, targets: Iterable[str]) -> bool`. It mainly works with `layer_name`, `targets`; implements one step in the quantized-weight execution flow. The body uses comprehensions. Key calls include `any`, `_is_equal_or_regex_match`.
**CN:** 定义函数 `check_equal_or_regex_match`，其签名为 `check_equal_or_regex_match(layer_name: str, targets: Iterable[str]) -> bool`。它主要围绕 `layer_name`, `targets` 展开；实现量化权重执行流程中的一个步骤。函数体包含推导式。关键调用包括 `any`, `_is_equal_or_regex_match`。

### Function `find_matched_target` (lines 113-153)
```python
def find_matched_target(
    layer_name: str | None,
    module: Module,
    targets: Iterable[str],
    fused_mapping: Mapping[str, list[str]] = MappingProxyType({}),
) -> str | None:
    """
    Helper function to look up which "target" in the compressed-tensors
    config that a layer corresponds to.

    Recall that a compressed-tensors configs has a concept of
    config_groups, where each layer can be quantized with a different
    scheme.

    targets in each config_group will be a list of either layer names
    (or regexes corresponding to layer names) or names of torch Modules.

    First, we try to match the layer_name with a target
    Second, we try to match the module's name with a target
    Third, we try to map the layer_name to a list of fused module names.
        *All* component module names must match in order for a match to be
        successful. A successful match returns the first component target

    :param layer_name: layer name
    :param module: torch.nn.Module
    :param targets: list of targets to match the layer against
    :param fused_mapping: map from fused layer names to its components
    :param fused_strategy: either "all" or "any". If using "all", fused
        layers match if "all" of its components match
    """

    if layer_name is None:
        layer_name = ""

    matched_target = (
        _find_first_match(layer_name, targets)
        or _find_first_match(module.__class__.__name__, targets, True)
        or _match_fused_layer(layer_name, targets, fused_mapping)
    )

    return matched_target
```
**EN:** Defines function `find_matched_target` with signature `find_matched_target(layer_name: str | None, module: Module, targets: Iterable[str], fused_mapping: Mapping[str, list[str]]=MappingProxyType({})) -> str | None`. It mainly works with `layer_name`, `module`, `targets`, `fused_mapping`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `MappingProxyType`, `_find_first_match`, `_match_fused_layer`.
**CN:** 定义函数 `find_matched_target`，其签名为 `find_matched_target(layer_name: str | None, module: Module, targets: Iterable[str], fused_mapping: Mapping[str, list[str]]=MappingProxyType({})) -> str | None`。它主要围绕 `layer_name`, `module`, `targets`, `fused_mapping` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `MappingProxyType`, `_find_first_match`, `_match_fused_layer`。

### Function `_find_first_match` (lines 156-172)
```python
def _find_first_match(
    value: str, targets: Iterable[str], check_contains: bool = False
) -> str | None:
    """
    Returns first element of target that matches value either
    exactly or as a regex after 're:'. If check_contains is set to True,
    additionally checks if the target string is contained within the value.

    :param value: string to compare the list of targets against
    :param targets: list of targets to match the layer against
    :param check_contains: whether or not to do a substring match
    """

    for target in targets:
        if _is_equal_or_regex_match(value, target, check_contains=check_contains):
            return target
    return None
```
**EN:** Defines function `_find_first_match` with signature `_find_first_match(value: str, targets: Iterable[str], check_contains: bool=False) -> str | None`. It mainly works with `value`, `targets`, `check_contains`; implements one step in the quantized-weight execution flow. The body uses branching, iteration. Key calls include `_is_equal_or_regex_match`.
**CN:** 定义函数 `_find_first_match`，其签名为 `_find_first_match(value: str, targets: Iterable[str], check_contains: bool=False) -> str | None`。它主要围绕 `value`, `targets`, `check_contains` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `_is_equal_or_regex_match`。

### Function `_is_equal_or_regex_match` (lines 175-193)
```python
def _is_equal_or_regex_match(
    value: str, target: str, check_contains: bool = False
) -> bool:
    """
    Checks whether a value is exactly equal or a regex match for target
    if target starts with 're:'. If check_contains is set to True,
    additionally checks if the target string is contained within the value.
    """

    if target.startswith("re:"):
        pattern = target[3:]
        if re.match(pattern, value):
            return True
    elif check_contains:
        if target.lower() in value.lower():
            return True
    elif target == value:
        return True
    return False
```
**EN:** Defines function `_is_equal_or_regex_match` with signature `_is_equal_or_regex_match(value: str, target: str, check_contains: bool=False) -> bool`. It mainly works with `value`, `target`, `check_contains`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `target.startswith`, `re.match`, `target.lower`, `value.lower`.
**CN:** 定义函数 `_is_equal_or_regex_match`，其签名为 `_is_equal_or_regex_match(value: str, target: str, check_contains: bool=False) -> bool`。它主要围绕 `value`, `target`, `check_contains` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `target.startswith`, `re.match`, `target.lower`, `value.lower`。

### Function `_match_fused_layer` (lines 196-238)
```python
def _match_fused_layer(
    layer_name: str,
    target_layers: Iterable[str],
    fused_mapping: Mapping[str, list[str]],
) -> str | None:
    """
    Match a fused layer name to its corresponding individual layer in
    target_layers. Returns first value in fused_mapping which matches targets

    Implements an "all" matching strategy where a fused layer matches iff
    "all" of its components match

    :param layer_name: layer name
    :param target_layers: list of targets to match the layer against
    :param fused_mapping: map from fused layer names to its components

    Examples:
        layer_name = "model.layers.0.self_attn.qkv_proj"
        target_layers = ["model.layers.0.self_attn.q_proj",
                        "model.layers.0.self_attn.k_proj",
                        "model.layers.0.self_attn.v_proj"]
    """
    # find layer_name in mapping
    fused = next((key for key in fused_mapping if layer_name.endswith(key)), None)
    if fused is None:
        return None

    # expand path of unfused components
    unfused_paths = [
        layer_name.replace(fused, unfused) for unfused in fused_mapping[fused]
    ]

    # for each unfused component, find a match in targets
    unfused_matches: list[str | None] = []
    for unfused in unfused_paths:
        for target in target_layers:
            if _is_equal_or_regex_match(unfused, target):
                unfused_matches.append(target)
                break
        else:
            unfused_matches.append(None)

    return unfused_matches[0] if all(unfused_matches) else None
```
**EN:** Defines function `_match_fused_layer` with signature `_match_fused_layer(layer_name: str, target_layers: Iterable[str], fused_mapping: Mapping[str, list[str]]) -> str | None`. It mainly works with `layer_name`, `target_layers`, `fused_mapping`; implements one step in the quantized-weight execution flow. The body uses branching, iteration, comprehensions. Key calls include `next`, `layer_name.replace`, `all`, `_is_equal_or_regex_match`, `unfused_matches.append`, `layer_name.endswith`.
**CN:** 定义函数 `_match_fused_layer`，其签名为 `_match_fused_layer(layer_name: str, target_layers: Iterable[str], fused_mapping: Mapping[str, list[str]]) -> str | None`。它主要围绕 `layer_name`, `target_layers`, `fused_mapping` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、循环处理、推导式。关键调用包括 `next`, `layer_name.replace`, `all`, `_is_equal_or_regex_match`, `unfused_matches.append`, `layer_name.endswith`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `is_activation_quantization_format`, `should_ignore_layer`, `check_equal_or_regex_match`, `find_matched_target`, `_find_first_match` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `is_activation_quantization_format`, `should_ignore_layer`, `check_equal_or_regex_match`, `find_matched_target`, `_find_first_match` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `types`, `regex`, `compressed_tensors`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.parameter`
