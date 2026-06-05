# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/compressed_tensors/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides helper functions and kernel-facing utilities for utils quantization flows. / 该模块提供了面向 工具 量化流程的辅助函数与内核工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and setup
```python
# Adapted from https://github.com/vllm-project/vllm/tree/main/vllm/model_executor/layers/quantization/compressed_tensors
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import re
from types import MappingProxyType
from typing import Iterable, List, Mapping, Optional

from compressed_tensors import CompressionFormat
from torch.nn import Module
```
**EN:** This block imports compressed_tensors, re, torch.nn, types and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 compressed_tensors, re, torch.nn, types 等依赖，并为当前量化实现准备模块命名空间。

### Lines 13-20: is_activation_quantization_format()
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
**EN:** This block defines `is_activation_quantization_format()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `is_activation_quantization_format()`，用于检查供上层控制流使用的条件。

### Lines 23-75: should_ignore_layer()
```python
def should_ignore_layer(
    layer_name: Optional[str],
    ignore: Iterable[str] = tuple(),
    fused_mapping: Mapping[str, List[str]] = MappingProxyType({}),
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
**EN:** This block defines `should_ignore_layer()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `should_ignore_layer()`，用于实现量化栈中的可复用模块逻辑。

### Lines 78-86: check_equal_or_regex_match()
```python
def check_equal_or_regex_match(layer_name: str, targets: Iterable[str]) -> bool:
    """
    Checks whether a layer_name is exactly equal or a regex match for
    if target starts with 're:' to any target in list.
    """
    for target in targets:
        if _is_equal_or_regex_match(layer_name, target, check_contains=True):
            return True
    return False
```
**EN:** This block defines `check_equal_or_regex_match()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_equal_or_regex_match()`，用于实现量化栈中的可复用模块逻辑。

### Lines 89-135: find_matched_target()
```python
def find_matched_target(
    layer_name: Optional[str],
    module: Module,
    targets: Iterable[str],
    fused_mapping: Mapping[str, List[str]] = MappingProxyType({}),
) -> str:
    """
    Helper function to look up which "target" in the compressed-tensors
    config that a layer corresponds to.

    Recall that a compressed-tensors configs has a concept of
    config_groups, where each layer can be quantized with with a different
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

    if matched_target is None:
        raise ValueError(
            f"Unable to find matching target for {layer_name} in the "
            "compressed-tensors config."
        )

    return matched_target
```
**EN:** This block defines `find_matched_target()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `find_matched_target()`，用于实现量化栈中的可复用模块逻辑。

### Lines 138-154: _find_first_match()
```python
def _find_first_match(
    value: str, targets: Iterable[str], check_contains: bool = False
) -> Optional[str]:
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
**EN:** This block defines `_find_first_match()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_find_first_match()`，用于实现量化栈中的可复用模块逻辑。

### Lines 157-175: _is_equal_or_regex_match()
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
**EN:** This block defines `_is_equal_or_regex_match()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_is_equal_or_regex_match()`，用于实现量化栈中的可复用模块逻辑。

### Lines 178-220: _match_fused_layer()
```python
def _match_fused_layer(
    layer_name: str,
    target_layers: Iterable[str],
    fused_mapping: Mapping[str, List[str]],
) -> Optional[str]:
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
    unfused_matches: List[Optional[str]] = []
    for unfused in unfused_paths:
        for target in target_layers:
            if _is_equal_or_regex_match(unfused, target):
                unfused_matches.append(target)
                break
        else:
            unfused_matches.append(None)

    return unfused_matches[0] if all(unfused_matches) else None
```
**EN:** This block defines `_match_fused_layer()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `_match_fused_layer()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `is_activation_quantization_format()` : A public function that checks a condition used by higher-level control flow. / `is_activation_quantization_format()`：一个公开函数，用于检查供上层控制流使用的条件。
- `should_ignore_layer()` : A public function that implements reusable module logic for the quantization stack. / `should_ignore_layer()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `check_equal_or_regex_match()` : A public function that implements reusable module logic for the quantization stack. / `check_equal_or_regex_match()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `find_matched_target()` : A public function that implements reusable module logic for the quantization stack. / `find_matched_target()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。

## Dependencies / 依赖关系
- **External / 外部**: `compressed_tensors`, `re`, `torch.nn`, `types`, `typing`
- **Internal / 内部**: Minimal package-local imports. / 包内直接导入较少。
