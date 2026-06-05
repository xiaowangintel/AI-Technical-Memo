# filesystem_resolver.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/plugins/lora_resolvers/filesystem_resolver.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `filesystem_resolver`-related logic centered around `FilesystemResolver`, `register_filesystem_resolver`. / 实现与 `filesystem_resolver` 相关的逻辑，核心符号包括 `FilesystemResolver`, `register_filesystem_resolver`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-8)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import json
import os

import vllm.envs as envs
from vllm.lora.request import LoRARequest
from vllm.lora.resolver import LoRAResolver, LoRAResolverRegistry
```
**EN:** Sets up the module with standard-library support such as `json`, `os`, vLLM modules such as `vllm.envs`, `vllm.lora.request`, `vllm.lora.resolver`. It prepares the symbols later used by `FilesystemResolver`, `register_filesystem_resolver`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.lora.request`, `vllm.lora.resolver` 等 vLLM 内部依赖。 这些准备工作为后续的 `FilesystemResolver`, `register_filesystem_resolver` 提供上下文。

### FilesystemResolver (lines 11-46)
```python
class FilesystemResolver(LoRAResolver):
    def __init__(self, lora_cache_dir: str):
        self.lora_cache_dir = lora_cache_dir

    async def resolve_lora(
        self, base_model_name: str, lora_name: str
    ) -> LoRARequest | None:
        lora_path = os.path.join(self.lora_cache_dir, lora_name)
        maybe_lora_request = await self._get_lora_req_from_path(
            lora_name, lora_path, base_model_name
        )
        return maybe_lora_request

    async def _get_lora_req_from_path(
        self, lora_name: str, lora_path: str, base_model_name: str
    ) -> LoRARequest | None:
        """Builds a LoraRequest pointing to the lora path if it's a valid
        LoRA adapter and has a matching base_model_name.
        """
        if os.path.exists(lora_path):
            adapter_config_path = os.path.join(lora_path, "adapter_config.json")

            if os.path.exists(adapter_config_path):
                with open(adapter_config_path) as file:
                    adapter_config = json.load(file)
                if (
                    adapter_config["peft_type"] == "LORA"
                    and adapter_config["base_model_name_or_path"] == base_model_name
                ):
                    lora_request = LoRARequest(
                        lora_name=lora_name,
                        lora_int_id=abs(hash(lora_name)),
                        lora_path=lora_path,
                    )
                    return lora_request
        return None
```
**EN:** Defines the `FilesystemResolver` class used by this module. It extends `LoRAResolver`. Key methods include `__init__`, `resolve_lora`.
**CN:** `FilesystemResolver` 是该文件中的核心类，用于封装与 `FilesystemResolver` 相关的状态和行为。 它继承自 `LoRAResolver`。 关键方法包括 `__init__`, `resolve_lora`。

### register_filesystem_resolver (lines 49-62)
```python
def register_filesystem_resolver():
    """Register the filesystem LoRA Resolver with vLLM"""

    lora_cache_dir = envs.VLLM_LORA_RESOLVER_CACHE_DIR
    if lora_cache_dir:
        if not os.path.exists(lora_cache_dir) or not os.path.isdir(lora_cache_dir):
            raise ValueError(
                "VLLM_LORA_RESOLVER_CACHE_DIR must be set to a valid directory \
                for Filesystem Resolver plugin to function"
            )
        fs_resolver = FilesystemResolver(lora_cache_dir)
        LoRAResolverRegistry.register_resolver("Filesystem Resolver", fs_resolver)

    return
```
**EN:** `register_filesystem_resolver`: Register the filesystem LoRA Resolver with vLLM. Inside the body, it relies on `FilesystemResolver`, `LoRAResolverRegistry.register_resolver`, `ValueError` to complete the main steps.
**CN:** `register_filesystem_resolver` 负责在分发表中注册实现。 实现过程中会调用 `FilesystemResolver`, `LoRAResolverRegistry.register_resolver`, `ValueError` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`FilesystemResolver`**: Core class that organizes module behavior. / **`FilesystemResolver`**：组织模块行为的核心类。
- **`register_filesystem_resolver`**: Key helper or entry point in this file. / **`register_filesystem_resolver`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: json, os
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.lora.request, vllm.lora.resolver
