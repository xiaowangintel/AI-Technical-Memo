# req.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/dllm/mixin/req.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed LLM coordination part of the SRT runtime and implements logic centered on `req`. It exposes primary entry points such as `DllmReqPhase`, `ReqDllmMixin`. / 该模块属于 SRT 运行时的分布式 LLM 协调部分，主要实现围绕 `req` 的逻辑。 它对外提供的主要入口包括 `DllmReqPhase`, `ReqDllmMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module imports, constants, and setup
```python
from __future__ import annotations

import enum
from typing import TYPE_CHECKING, Optional

from sglang.srt.dllm.config import DllmConfig

if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 12-16: Class DllmReqPhase
```python
class DllmReqPhase(str, enum.Enum):
    STAGING_PREFILL = "staging_prefill"
    STAGING_DECODE = "staging_decode"
    INCOMING_PREFILL = "incoming_prefill"
    INCOMING_DECODE = "incoming_decode"
```
**EN:** This range introduces `DllmReqPhase` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DllmReqPhase`，并定义其后续方法依赖的结构或元数据。

### Lines 19-19: Class ReqDllmMixin
```python
class ReqDllmMixin:
```
**EN:** This range introduces `ReqDllmMixin` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ReqDllmMixin`，并定义其后续方法依赖的结构或元数据。

### Lines 20-29: Method ReqDllmMixin.init_diffusion_llm
```python
    def init_diffusion_llm(self: Req, dllm_config: DllmConfig):
        self.dllm_phase: Optional[DllmReqPhase] = None
        self.dllm_block_offset = 0
        self.dllm_config = dllm_config

        if self.dllm_config is not None:
            if len(self.origin_input_ids) < self.dllm_config.block_size:
                self.dllm_phase = DllmReqPhase.INCOMING_DECODE
            else:
                self.dllm_phase = DllmReqPhase.INCOMING_PREFILL
```
**EN:** This callable implements `ReqDllmMixin.init_diffusion_llm`. It takes `dllm_config` and mainly implements init diffusion llm.
**CN:** 这一可调用对象实现了 `ReqDllmMixin.init_diffusion_llm`。它接收 `dllm_config`，主要用于实现 init diffusion llm 相关逻辑。

### Lines 31-32: Method ReqDllmMixin.is_dllm
```python
    def is_dllm(self: Req) -> bool:
        return self.dllm_config is not None
```
**EN:** This callable implements `ReqDllmMixin.is_dllm` and mainly implements is dllm.
**CN:** 这一可调用对象实现了 `ReqDllmMixin.is_dllm`，主要用于实现 is dllm 相关逻辑。

### Lines 34-38: Method ReqDllmMixin.is_dllm_prefill
```python
    def is_dllm_prefill(self: Req) -> bool:
        return self.dllm_phase in [
            DllmReqPhase.STAGING_PREFILL,
            DllmReqPhase.INCOMING_PREFILL,
        ]
```
**EN:** This callable implements `ReqDllmMixin.is_dllm_prefill` and mainly implements is dllm prefill.
**CN:** 这一可调用对象实现了 `ReqDllmMixin.is_dllm_prefill`，主要用于实现 is dllm prefill 相关逻辑。

### Lines 40-54: Method ReqDllmMixin.determine_dllm_phase
```python
    def determine_dllm_phase(self: Req):
        prefix_length = len(self.prefix_indices)
        min_required_length = prefix_length + self.dllm_config.block_size

        if len(self.fill_ids) < min_required_length:
            # still incoming stage
            return

        input_block = self.fill_ids[prefix_length:min_required_length]
        is_prefill_phase = self.dllm_config.mask_id not in input_block

        if is_prefill_phase:
            self.dllm_phase = DllmReqPhase.STAGING_PREFILL
        else:
            self.dllm_phase = DllmReqPhase.STAGING_DECODE
```
**EN:** This callable implements `ReqDllmMixin.determine_dllm_phase` and mainly implements determine dllm phase.
**CN:** 这一可调用对象实现了 `ReqDllmMixin.determine_dllm_phase`，主要用于实现 determine dllm phase 相关逻辑。

### Lines 56-66: Method ReqDllmMixin._init_fill_ids_for_dllm
```python
    def _init_fill_ids_for_dllm(self: Req):
        self.dllm_block_offset = (
            0
            if not self.fill_ids
            else self.dllm_block_offset + self.dllm_config.block_size
        )
        self.fill_ids = (
            self.origin_input_ids
            + self.output_ids
            + [self.dllm_config.mask_id] * self.dllm_config.block_size
        )
```
**EN:** This callable implements `ReqDllmMixin._init_fill_ids_for_dllm` and mainly implements init fill ids for dllm.
**CN:** 这一可调用对象实现了 `ReqDllmMixin._init_fill_ids_for_dllm`，主要用于实现 init fill ids for dllm 相关逻辑。

### Lines 68-74: Method ReqDllmMixin._update_block_offset_for_dllm
```python
    def _update_block_offset_for_dllm(self):
        prefix_len = len(self.prefix_indices)
        assert (
            prefix_len % self.dllm_config.block_size == 0
        ), f"Unexpected prefix len: {prefix_len}"
        if prefix_len > self.dllm_block_offset:
            self.dllm_block_offset = prefix_len
```
**EN:** This callable implements `ReqDllmMixin._update_block_offset_for_dllm` and mainly updates existing runtime state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ReqDllmMixin._update_block_offset_for_dllm`，主要用于更新现有运行时状态。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `DllmReqPhase`: core class or state container / 核心类或状态容器
- `ReqDllmMixin`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `enum`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.dllm.config`, `sglang.srt.managers.schedule_batch`
