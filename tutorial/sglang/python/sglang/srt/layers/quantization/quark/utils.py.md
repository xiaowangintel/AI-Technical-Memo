# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/quark/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides helper functions and kernel-facing utilities for utils quantization flows. / 该模块提供了面向 工具 量化流程的辅助函数与内核工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0

import re
from collections.abc import Iterable, Mapping
from types import MappingProxyType
from typing import Any, Optional

import torch
```
**EN:** This block imports aiter.ops.triton.quant, collections.abc, re, torch and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.triton.quant, collections.abc, re, torch 等依赖，并为当前量化实现准备模块命名空间。

### Lines 10-19: guarded import or fallback path
```python
try:
    from aiter.ops.triton.quant import dynamic_mxfp4_quant
except ImportError as err:

    def raise_aiter_import_error(*args, **kwargs):
        raise ImportError(
            "Failed to import aiter. " "Make sure AITER is installed and accessible."
        )

    dynamic_mxfp4_quant = raise_aiter_import_error
```
**EN:** This block uses exception handling to provide a fallback implementation when optional functionality is unavailable.
**CN:** 该代码块通过异常处理在可选功能不可用时提供后备实现。

### Lines 20-20: module imports and setup
```python
from torch import nn
```
**EN:** This block imports aiter.ops.triton.quant, collections.abc, re, torch and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 aiter.ops.triton.quant, collections.abc, re, torch 等依赖，并为当前量化实现准备模块命名空间。

### Lines 23-33: deep_compare()
```python
def deep_compare(dict1: Any, dict2: Any) -> bool:
    if type(dict1) is not type(dict2):
        return False
    if isinstance(dict1, dict):
        if dict1.keys() != dict2.keys():
            return False
        return all(deep_compare(dict1[k], dict2[k]) for k in dict1)
    elif isinstance(dict1, list):
        return set(dict1) == set(dict2)
    else:
        return dict1 == dict2
```
**EN:** This block defines `deep_compare()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `deep_compare()`，用于实现量化栈中的可复用模块逻辑。

### Lines 36-89: should_ignore_layer()
```python
def should_ignore_layer(
    layer_name: Optional[str],
    ignore: Iterable[str],
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
    if proj_name in fused_mapping:
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

### Lines 92-100: check_equal_or_regex_match()
```python
def check_equal_or_regex_match(layer_name: str, targets: Iterable[str]) -> bool:
    """
    Checks whether a layer_name is exactly equal or a regex match for
    if target starts with 're:' to any target in list.
    """
    for target in targets:
        if _is_equal_or_regex_match(layer_name, target):
            return True
    return False
```
**EN:** This block defines `check_equal_or_regex_match()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `check_equal_or_regex_match()`，用于实现量化栈中的可复用模块逻辑。

### Lines 103-121: _is_equal_or_regex_match()
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

### Lines 125-128: b_dynamic_mxfp4_quant()
```python
def b_dynamic_mxfp4_quant(x):
    h, b, d = x.shape
    x, x_scales = dynamic_mxfp4_quant(x.reshape(-1, d))
    return x.view(h, b, d // 2), x_scales.view(h, b, d // 32)
```
**EN:** This block defines `b_dynamic_mxfp4_quant()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `b_dynamic_mxfp4_quant()`，用于处理张量或权重量化逻辑。

### Lines 131-160: mxfp4_to_f32()
```python
def mxfp4_to_f32(x, is_3d):
    # 2 because we pack fp4 in uint8.
    x = x.repeat_interleave(2, dim=-1)
    if is_3d:
        x[..., ::2] = x[..., ::2] & 0xF
        x[..., 1::2] = x[..., 1::2] >> 4
    else:
        x[:, ::2] = x[:, ::2] & 0xF
        x[:, 1::2] = x[:, 1::2] >> 4

    mxfp4_list = [
        0.0,
        0.5,
        1.0,
        1.5,
        2.0,
        3.0,
        4.0,
        6.0,
        -0.0,
        -0.5,
        -1.0,
        -1.5,
        -2.0,
        -3.0,
        -4.0,
        -6.0,
    ]
    mxfp4_in_f32 = torch.tensor(mxfp4_list, dtype=torch.float32, device="cuda")
    return mxfp4_in_f32[x.long()]
```
**EN:** This block defines `mxfp4_to_f32()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `mxfp4_to_f32()`，用于实现量化栈中的可复用模块逻辑。

### Lines 163-174: e8m0_to_f32()
```python
def e8m0_to_f32(x):
    # Convert the input tensor `x` (assumed to be in e8m0 format) to float32.
    # e8m0 is a custom 8-bit floating point format with 8 bits for exponent, 0 for mantissa.
    # This means the value is essentially 2^(exponent - 127), similar to how IEEE-754 stores floats.

    # Convert x to float32 for computation, and compute the power of 2 by subtracting the bias (127).
    x_f32 = 2 ** ((x.to(torch.float32)) - 127)

    # If the exponent value was 255 (i.e., 2^(128)), this is a special case usually used to represent NaN or Inf.
    # Since this custom format has no mantissa, treat 2^128 as NaN.
    x_f32[x_f32 == 128] = float("nan")
    return x_f32
```
**EN:** This block defines `e8m0_to_f32()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `e8m0_to_f32()`，用于实现量化栈中的可复用模块逻辑。

### Lines 177-214: quark_post_load_weights()
```python
def quark_post_load_weights(self_attn: nn.Module, w: torch.Tensor, quant_format: str):
    if "mxfp4" in quant_format:
        # when dtype is bf16, the processing flow is to dynamic quantize bf16 tensor to uint8 tensor
        # do w_kc (bf16) first to get the w_kc(uint8) w_s_kc(uint8)
        # and w_vc repeating the same procedure of w_kc to get  w_vc(uint8) w_s_vc(uint8)
        if w.dtype == torch.bfloat16:
            w_kc, w_vc = w.unflatten(
                0, (-1, self_attn.qk_nope_head_dim + self_attn.v_head_dim)
            ).split([self_attn.qk_nope_head_dim, self_attn.v_head_dim], dim=1)
            w_kc, w_s_kc = b_dynamic_mxfp4_quant(w_kc.transpose(-2, -1))
            w_kc = w_kc.transpose(-2, -1)
            w_s_kc = w_s_kc.transpose(-2, -1)
            w_vc, w_s_vc = b_dynamic_mxfp4_quant(w_vc)
            w_s_kc = w_s_kc.transpose(1, 2).contiguous().transpose(1, 2)
            w_s_vc = w_s_vc.contiguous().transpose(1, 2)
        elif w.dtype == torch.uint8:  # static quant for mxfp4
            # when dtype is uint8, it means the w has been quantized to mxfp4 format
            # but we must separate it to w_kc and w_vc.
            # The quantized tensor size is only half of original tensor size
            # and the scaling factor is 1/32, the transpose behavior will be not correct
            # need to upcast it to fp32 to separate w to w_kc and w_vc
            # to ensure the following transpose behavior is correct
            # and then do mxfp4 quant again
            w = mxfp4_to_f32(w, True).to(torch.bfloat16)
            w_scales = self_attn.kv_b_proj.weight_scale.repeat_interleave(32, dim=-1)
            w_scales = e8m0_to_f32(w_scales).to(torch.bfloat16)
            w = w * w_scales
            w_kc, w_vc = w.unflatten(
                0, (-1, (self_attn.qk_nope_head_dim + self_attn.v_head_dim))
            ).split([self_attn.qk_nope_head_dim, self_attn.v_head_dim], dim=1)
            w_kc, w_s_kc = b_dynamic_mxfp4_quant(w_kc.transpose(-2, -1))
            w_kc = w_kc.transpose(-2, -1)
            w_s_kc = w_s_kc.transpose(-2, -1)
            w_vc, w_s_vc = b_dynamic_mxfp4_quant(w_vc)
            w_s_kc = w_s_kc.transpose(1, 2).contiguous().transpose(1, 2)
            w_s_vc = w_s_vc.contiguous().transpose(1, 2)

        return w_kc, w_s_kc, w_vc, w_s_vc
```
**EN:** This block defines `quark_post_load_weights()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `quark_post_load_weights()`，用于处理检查点加载或布局转换。

## Key Concepts / 关键概念
- `deep_compare()` : A public function that implements reusable module logic for the quantization stack. / `deep_compare()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `should_ignore_layer()` : A public function that implements reusable module logic for the quantization stack. / `should_ignore_layer()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `check_equal_or_regex_match()` : A public function that implements reusable module logic for the quantization stack. / `check_equal_or_regex_match()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `b_dynamic_mxfp4_quant()` : A public function that handles tensor or weight quantization work. / `b_dynamic_mxfp4_quant()`：一个公开函数，用于处理张量或权重量化逻辑。
- `mxfp4_to_f32()` : A public function that implements reusable module logic for the quantization stack. / `mxfp4_to_f32()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `e8m0_to_f32()` : A public function that implements reusable module logic for the quantization stack. / `e8m0_to_f32()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `quark_post_load_weights()` : A public function that handles checkpoint loading or layout conversion. / `quark_post_load_weights()`：一个公开函数，用于处理检查点加载或布局转换。

## Dependencies / 依赖关系
- **External / 外部**: `aiter.ops.triton.quant`, `collections.abc`, `re`, `torch`, `types`, `typing`
- **Internal / 内部**: Minimal package-local imports. / 包内直接导入较少。
