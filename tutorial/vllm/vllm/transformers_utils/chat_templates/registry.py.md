# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/chat_templates/registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Maintains the chat-template registry that maps model identifiers to bundled template assets. / [CN] 维护聊天模板注册表，将模型标识映射到仓库内置的模板资源。

## Line-by-Line Analysis / 逐行分析

### Lines 3-7: Imports
```python
from collections.abc import Callable
from pathlib import Path
from typing import TypeAlias

from vllm.logger import init_logger
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `collections.abc`, `pathlib`, `typing`, external APIs such as none, and internal vLLM modules such as `vllm.logger`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `collections.abc`, `pathlib`, `typing`，外部 API 如 无，以及 vLLM 内部模块如 `vllm.logger`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 9-13: Module state and constants
```python
logger = init_logger(__file__)

CHAT_TEMPLATES_DIR = Path(__file__).parent

ChatTemplatePath: TypeAlias = Path | Callable[[str], Path | None]
```
**EN:** This block defines module-level constants/defaults such as `logger`, `CHAT_TEMPLATES_DIR`, `ChatTemplatePath`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `logger`, `CHAT_TEMPLATES_DIR`, `ChatTemplatePath`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 16-20: Function `_get_qwen_chat_template_fallback`
```python
def _get_qwen_chat_template_fallback(tokenizer_name_or_path: str) -> Path | None:
    if tokenizer_name_or_path.endswith("-Chat"):
        return CHAT_TEMPLATES_DIR / "template_chatml.jinja"

    return CHAT_TEMPLATES_DIR / "template_basic.jinja"
```
**EN:** This private function retrieves qwen chat template fallback. Main inputs include `tokenizer_name_or_path`.
**CN:** 该私有函数负责完成 `_get_qwen_chat_template_fallback` 对应的核心步骤。 主要输入参数包括 `tokenizer_name_or_path`。

### Lines 23-29: Function `_get_minicpmv_chat_template_fallback`
```python
def _get_minicpmv_chat_template_fallback(tokenizer_name_or_path: str) -> Path | None:
    # MiniCPM-V-4.5 version uses a dedicated template
    if "4.5" in tokenizer_name_or_path or "4_5" in tokenizer_name_or_path:
        return CHAT_TEMPLATES_DIR / "template_minicpmv45.jinja"

    # Other versions use chatml template
    return CHAT_TEMPLATES_DIR / "template_chatml.jinja"
```
**EN:** This private function retrieves minicpmv chat template fallback. Main inputs include `tokenizer_name_or_path`.
**CN:** 该私有函数负责完成 `_get_minicpmv_chat_template_fallback` 对应的核心步骤。 主要输入参数包括 `tokenizer_name_or_path`。

### Lines 32-47: Module state and constants
```python
_MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK: dict[str, ChatTemplatePath] = {
    "blip-2": CHAT_TEMPLATES_DIR / "template_blip2.jinja",
    "chameleon": CHAT_TEMPLATES_DIR / "template_basic.jinja",
    "clip": CHAT_TEMPLATES_DIR / "template_basic.jinja",
    "colpali": CHAT_TEMPLATES_DIR / "template_basic.jinja",
    "deepseek_ocr": CHAT_TEMPLATES_DIR / "template_deepseek_ocr.jinja",
    "deepseek_ocr2": CHAT_TEMPLATES_DIR / "template_deepseek_ocr.jinja",
    "deepseek_vl_v2": CHAT_TEMPLATES_DIR / "template_deepseek_vl2.jinja",
    "fuyu": CHAT_TEMPLATES_DIR / "template_fuyu.jinja",
    "minicpmv": _get_minicpmv_chat_template_fallback,
    "minicpmv4_6": _get_minicpmv_chat_template_fallback,
    "paligemma": CHAT_TEMPLATES_DIR / "template_basic.jinja",
    "qwen": _get_qwen_chat_template_fallback,
    "siglip": CHAT_TEMPLATES_DIR / "template_basic.jinja",
    "siglip2": CHAT_TEMPLATES_DIR / "template_basic.jinja",
}
```
**EN:** This block defines module-level constants/defaults such as `_MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `_MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 50-62: Function `register_chat_template_fallback_path`
```python
def register_chat_template_fallback_path(
    model_type: str,
    chat_template: ChatTemplatePath,
) -> None:
    if model_type in _MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK:
        logger.warning(
            "Model type %s already has a chat template registered. "
            "It will be overwritten by the new chat template %s.",
            model_type,
            chat_template,
        )

    _MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK[model_type] = chat_template
```
**EN:** This function registers chat template fallback path. Main inputs include `model_type`, `chat_template`.
**CN:** 该函数负责完成 `register_chat_template_fallback_path` 对应的核心步骤。 主要输入参数包括 `model_type`, `chat_template`。

### Lines 65-76: Function `get_chat_template_fallback_path`
```python
def get_chat_template_fallback_path(
    model_type: str,
    tokenizer_name_or_path: str,
) -> Path | None:
    chat_template = _MODEL_TYPE_TO_CHAT_TEMPLATE_FALLBACK.get(model_type)
    if callable(chat_template):
        chat_template = chat_template(tokenizer_name_or_path)

    if chat_template is None:
        return None

    return chat_template
```
**EN:** This function retrieves chat template fallback path. Main inputs include `model_type`, `tokenizer_name_or_path`.
**CN:** 该函数负责完成 `get_chat_template_fallback_path` 对应的核心步骤。 主要输入参数包括 `model_type`, `tokenizer_name_or_path`。

## Key Concepts / 关键概念
- **EN:** Chat-template resolution is registry-driven: model identifiers are mapped to concrete prompt template assets.
- **CN:** 聊天模板解析采用注册表驱动方式：模型标识会被映射到具体的提示词模板资源。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `collections.abc`, `pathlib`, `typing`.
- **CN:** 标准库模块：`collections.abc`, `pathlib`, `typing`。
- **EN:** External packages: none.
- **CN:** 外部依赖包：无。
- **EN:** Internal modules: `vllm.logger`.
- **CN:** 内部模块：`vllm.logger`。
