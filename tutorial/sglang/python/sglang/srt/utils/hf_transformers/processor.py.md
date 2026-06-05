# processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/hf_transformers/processor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `processor` and the surrounding SGLang serving stack. / 提供围绕 `processor` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 14-50: Module setup and shared state / 模块设置与共享状态
```python
"""Processor loading utilities."""

import json
from pathlib import Path
from typing import Optional

from transformers import (
    AutoProcessor,
    AutoTokenizer,
    PreTrainedTokenizerBase,
)

from sglang.srt.multimodal.customized_mm_processor_utils import _CUSTOMIZED_MM_PROCESSOR
from sglang.srt.utils import logger

from .common import (
    AutoConfig,
    _is_deepseek_ocr2_model,
    _is_deepseek_ocr_model,
    _override_v_head_dim_if_zero,
    _resolve_local_or_cached_file,
    attach_additional_stop_token_ids,
    download_from_hf,
    get_tokenizer_from_processor,
    resolve_runai_obj_uri,
)
from .mistral_utils import (
    is_mistral_model,
# ... omitted for brevity ...
)
from .tokenizer import (
    _TOKENIZERS_BACKEND,
    _fix_added_tokens_encoding,
    _fix_special_tokens_pattern,
)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `json`, `pathlib`, `typing`, `transformers`, `sglang.srt.multimodal.customized_mm_processor_utils`, `sglang.srt.utils`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `json`, `pathlib`, `typing`, `transformers`, `sglang.srt.multimodal.customized_mm_processor_utils`, `sglang.srt.utils`。

### Lines 53-135: Function `_build_processor_manually` / 函数 `_build_processor_manually`
```python
def _build_processor_manually(
    model_path, config, trust_remote_code, revision, **kwargs
):
    """Build processor when AutoProcessor fails to resolve feature_extractor_type.

    In transformers v5, AutoProcessor.from_pretrained calls
    AutoFeatureExtractor.from_pretrained which fails if
    preprocessor_config.json lacks 'feature_extractor_type'. This resolves
    the processor class via dynamic module resolution and constructs it with
    individually-loaded components.
    """
    import transformers
    from transformers import AutoImageProcessor, AutoTokenizer
    from transformers.dynamic_module_utils import get_class_from_dynamic_module

    # Resolve processor class from auto_map -- check both the model config
    # and the preprocessor_config.json (some models like MiniCPM-o only
    # declare AutoProcessor in the latter).
    auto_map = getattr(config, "auto_map", None) or {}
    proc_ref = auto_map.get("AutoProcessor")
    if not proc_ref:
        try:
            pp_file = _resolve_local_or_cached_file(
                model_path, "preprocessor_config.json", revision
            )
            with open(pp_file) as f:
                pp_auto_map = json.load(f).get("auto_map", {})
            proc_ref = pp_auto_map.get("AutoProcessor")
# ... omitted for brevity ...
                "Feature extractor class %s not found in transformers for %s",
                fe_class_name,
                model_path,
            )

    return proc_cls(**init_kwargs)
```
**EN:** This function implements `_build_processor_manually`. It primarily calls `auto_map.get`, `get_class_from_dynamic_module`, `AutoTokenizer.from_pretrained`, `getattr`, `proc_cls`, `ValueError` to complete its work. State updates are written into `auto_map`, `proc_ref`, `proc_cls`, `tokenizer`, `init_kwargs`, `fe_class_name`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该函数实现了 `_build_processor_manually`。 它主要通过调用 `auto_map.get`, `get_class_from_dynamic_module`, `AutoTokenizer.from_pretrained`, `getattr`, `proc_cls`, `ValueError` 来完成任务。 状态更新主要写入 `auto_map`, `proc_ref`, `proc_cls`, `tokenizer`, `init_kwargs`, `fe_class_name`。 实现中使用了条件分支、上下文管理资源、错误处理。

### Lines 138-298: Function `get_processor` / 函数 `get_processor`
```python
def get_processor(
    tokenizer_name: str,
    *args,
    tokenizer_mode: str = "auto",
    trust_remote_code: bool = False,
    tokenizer_revision: Optional[str] = None,
    use_fast: Optional[bool] = True,
    tokenizer_backend: str = "huggingface",
    **kwargs,
):
    if tokenizer_backend == "fastokens":
        from .tokenizer import _ensure_fastokens_patched

        _ensure_fastokens_patched()

    revision = kwargs.pop("revision", tokenizer_revision)
    tokenizer_name = resolve_runai_obj_uri(tokenizer_name)

    if is_mistral_model(tokenizer_name):
        config = load_mistral_config(
            tokenizer_name,
            trust_remote_code=trust_remote_code,
            revision=revision,
        )
    else:
        config = AutoConfig.from_pretrained(
            tokenizer_name,
            trust_remote_code=trust_remote_code,
# ... omitted for brevity ...

    patch_mistral_common_tokenizer(tokenizer)
    _fix_special_tokens_pattern(tokenizer)
    _fix_added_tokens_encoding(tokenizer)
    attach_additional_stop_token_ids(tokenizer)
    return processor
```
**EN:** This function implements `get_processor`. It primarily calls `kwargs.pop`, `resolve_runai_obj_uri`, `is_mistral_model`, `_is_deepseek_ocr2_model`, `get_tokenizer_from_processor`, `patch_mistral_common_tokenizer` to complete its work. State updates are written into `revision`, `tokenizer_name`, `is_ocr2`, `tokenizer`, `config`, `config.model_type`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_processor`。 它主要通过调用 `kwargs.pop`, `resolve_runai_obj_uri`, `is_mistral_model`, `_is_deepseek_ocr2_model`, `get_tokenizer_from_processor`, `patch_mistral_common_tokenizer` 来完成任务。 状态更新主要写入 `revision`, `tokenizer_name`, `is_ocr2`, `tokenizer`, `config`, `config.model_type`。 实现中使用了条件分支、错误处理。

## Key Concepts / 关键概念
- **Functions / 函数**: `_build_processor_manually`, `get_processor`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.multimodal.customized_mm_processor_utils`, `sglang.srt.utils`, `.common`, `.mistral_utils`, `.tokenizer`
- **External / 外部依赖**: `transformers`, `transformers.dynamic_module_utils`
- **Standard library / 标准库**: `json`, `pathlib`, `typing`
