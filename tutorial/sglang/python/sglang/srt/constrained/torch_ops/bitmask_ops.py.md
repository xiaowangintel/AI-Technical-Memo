# bitmask_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/torch_ops/bitmask_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `bitmask_ops`. It exposes primary entry points such as `apply_token_bitmask_inplace_torch`. / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `bitmask_ops` 的逻辑。 它对外提供的主要入口包括 `apply_token_bitmask_inplace_torch`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module imports, constants, and setup
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import torch


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 17-33: Function apply_token_bitmask_inplace_torch
```python
def apply_token_bitmask_inplace_torch(
    logits: torch.Tensor,
    bitmask: torch.Tensor,
) -> None:
    """Backend-agnostic torch fallback for packed-bitmask application.

    This path is currently used as a fallback on NPU in xgrammar backend.
    """
    vocab_size = logits.shape[-1]
    bitmask_cpu = bitmask.detach().cpu()
    token_ids = torch.arange(vocab_size, device="cpu", dtype=torch.int32)
    word_idx = token_ids // 32
    bit_idx = token_ids % 32
    words = bitmask_cpu[:, word_idx].to(torch.int32)
    allowed = ((words >> bit_idx) & 1).to(torch.bool)
    allowed = allowed.to(logits.device, non_blocking=True)
    logits.masked_fill_(~allowed, float("-inf"))
```
**EN:** This callable implements `apply_token_bitmask_inplace_torch`. It takes `logits`, `bitmask` and mainly converts data into another representation. The docstring states: "Backend-agnostic torch fallback for packed-bitmask application." In this range it handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `apply_token_bitmask_inplace_torch`。它接收 `logits`, `bitmask`，主要用于将数据转换为另一种表示。 在这一范围内，它会处理语法或正则约束。

## Key Concepts / 关键概念
- `apply_token_bitmask_inplace_torch`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`
