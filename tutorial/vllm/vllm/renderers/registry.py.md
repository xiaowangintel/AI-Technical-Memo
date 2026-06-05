# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/renderers/registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements rendering adapters used to turn structured inputs into prompts or outputs. / 实现把结构化输入转换为提示词或输出内容的渲染适配层。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-32)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from dataclasses import dataclass, field
from typing import TYPE_CHECKING

from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike
from vllm.tokenizers.registry import (
    cached_tokenizer_from_config,
    tokenizer_args_from_config,
)
from vllm.utils.import_utils import resolve_obj_by_qualname

from .base import BaseRenderer

if TYPE_CHECKING:
    from vllm.config import VllmConfig

logger = init_logger(__name__)


_VLLM_RENDERERS = {
    "deepseek_v32": ("deepseek_v32", "DeepseekV32Renderer"),
    "deepseek_v4": ("deepseek_v4", "DeepseekV4Renderer"),
    "fastokens": ("hf", "HfRenderer"),
    "grok2": ("grok2", "Grok2Renderer"),
    "hf": ("hf", "HfRenderer"),
    "kimi_audio": ("hf", "HfRenderer"),
    "mistral": ("mistral", "MistralRenderer"),
    "qwen_vl": ("hf", "HfRenderer"),
    "terratorch": ("terratorch", "TerratorchRenderer"),
}
```
**EN:** Sets up the module with standard-library support such as `dataclasses`, `typing`, vLLM modules such as `vllm.logger`, `vllm.tokenizers`, `vllm.tokenizers.registry`. It prepares the symbols later used by `RendererRegistry`, `renderer_from_config`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.tokenizers`, `vllm.tokenizers.registry` 等 vLLM 内部依赖。 这些准备工作为后续的 `RendererRegistry`, `renderer_from_config` 提供上下文。

### RendererRegistry (lines 36-70)
```python
class RendererRegistry:
    # Renderer mode ->  (renderer module, renderer class)
    renderers: dict[str, tuple[str, str]] = field(default_factory=dict)

    def register(self, renderer_mode: str, module: str, class_name: str) -> None:
        if renderer_mode in self.renderers:
            logger.warning(
                "%s.%s is already registered for renderer_mode=%r. "
                "It is overwritten by the new one.",
                module,
                class_name,
                renderer_mode,
            )

        self.renderers[renderer_mode] = (module, class_name)

        return None

    def load_renderer_cls(self, renderer_mode: str) -> type[BaseRenderer]:
        if renderer_mode not in self.renderers:
            raise ValueError(f"No renderer registered for {renderer_mode=!r}.")

        module, class_name = self.renderers[renderer_mode]
        logger.debug_once(f"Loading {class_name} for {renderer_mode=!r}")

        return resolve_obj_by_qualname(f"{module}.{class_name}")

    def load_renderer(
        self,
        renderer_mode: str,
        config: "VllmConfig",
        tokenizer: TokenizerLike | None,
    ) -> BaseRenderer:
        renderer_cls = self.load_renderer_cls(renderer_mode)
        return renderer_cls(config, tokenizer)
```
**EN:** Defines the `RendererRegistry` class used by this module. Key methods include `register`, `load_renderer_cls`, `load_renderer`.
**CN:** `RendererRegistry` 是该文件中的核心类，用于封装与 `RendererRegistry` 相关的状态和行为。 关键方法包括 `register`, `load_renderer_cls`, `load_renderer`。

### renderer_from_config (lines 82-88)
```python
def renderer_from_config(config: "VllmConfig", **kwargs):
    model_config = config.model_config

    tokenizer = cached_tokenizer_from_config(model_config, **kwargs)
    renderer_mode, *_ = tokenizer_args_from_config(model_config, **kwargs)

    return RENDERER_REGISTRY.load_renderer(renderer_mode, config, tokenizer)
```
**EN:** `renderer_from_config` implements helper logic used by this module. It mainly works with `config`, `**kwargs`. Inside the body, it relies on `cached_tokenizer_from_config`, `tokenizer_args_from_config`, `RENDERER_REGISTRY.load_renderer` to complete the main steps.
**CN:** `renderer_from_config` 负责实现本模块使用的辅助逻辑。 它主要处理 `config`, `**kwargs` 等参数。 实现过程中会调用 `cached_tokenizer_from_config`, `tokenizer_args_from_config`, `RENDERER_REGISTRY.load_renderer` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`RendererRegistry`**: Core class that organizes module behavior. / **`RendererRegistry`**：组织模块行为的核心类。
- **`renderer_from_config`**: Key helper or entry point in this file. / **`renderer_from_config`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: dataclasses, typing
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.tokenizers, vllm.tokenizers.registry, vllm.utils.import_utils, .base, vllm.config
