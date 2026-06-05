# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/triton_ops/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Triton kernels for speculative decoding. / 该文件提供了推测解码使用的 Triton 内核。实现重点在于 GPU 侧的物化、校验或缓存移动，以加速草稿模型与目标模型之间的协同。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20: Module header, imports, and shared constants
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
# ==============================================================================
"""Triton kernels for speculative decoding."""

from sglang.srt.speculative.triton_ops.fused_kv_materialize import (
    FusedKVMaterializeHelper,
)

__all__ = ["FusedKVMaterializeHelper"]
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

## Key Concepts / 关键概念
- Triton GPU kernels / Triton GPU 内核
- Speculative decoding / 推测解码

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.speculative.triton_ops.fused_kv_materialize`
### External / 外部
- None / 无
