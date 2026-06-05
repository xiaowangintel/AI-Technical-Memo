# tokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/hf_transformers/tokenizer.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for token processing helpers. / 为 SGLang 运行时提供面向Token 处理辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 14-49: Module setup and shared state / 模块设置与共享状态
```python
"""Tokenizer loading utilities."""

import json
import logging
import warnings
from pathlib import Path
from typing import Optional, Union

from transformers import (
    AutoTokenizer,
    PreTrainedTokenizer,
    PreTrainedTokenizerFast,
)

from sglang.srt.connector import create_remote_connector
from sglang.srt.utils import is_remote_url, logger
from sglang.srt.utils.patch_tokenizer import patch_tokenizer

from ..hf_transformers_patches import _ensure_gguf_version
from .common import (
    _resolve_local_or_cached_file,
    attach_additional_stop_token_ids,
    check_gguf_file,
    resolve_runai_obj_uri,
)
from .mistral_utils import (
    _MISTRAL_TOKENIZER_REDIRECTS,
    patch_mistral_common_tokenizer,
    retry_without_mistral_common_kwargs,
)

# A fast LLaMA tokenizer with the pre-processed `tokenizer.json` file.
_FAST_LLAMA_TOKENIZER = "hf-internal-testing/llama-tokenizer"

# Class name used by transformers v5 when no tokenizer mapping exists for a model_type.
_TOKENIZERS_BACKEND = "TokenizersBackend"
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `json`, `logging`, `warnings`, `pathlib`, `typing`, `transformers`. It also defines symbols such as `_FAST_LLAMA_TOKENIZER`, `_TOKENIZERS_BACKEND` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `json`, `logging`, `warnings`, `pathlib`, `typing`, `transformers`。 同时定义了 `_FAST_LLAMA_TOKENIZER`, `_TOKENIZERS_BACKEND` 等符号，供后续逻辑使用。

### Lines 52-123: Function `_load_tokenizer_by_declared_class` / 函数 `_load_tokenizer_by_declared_class`
```python
def _load_tokenizer_by_declared_class(tokenizer_name, *args, **kwargs):
    """Load tokenizer by the class declared in tokenizer_config.json.

    AutoTokenizer resolves to TokenizersBackend when the model's config
    model_type has no tokenizer class mapping (e.g. deepseek_vl_v2), even
    though tokenizer_config.json declares a standard class like
    LlamaTokenizerFast.  Returns None if it cannot improve on AutoTokenizer.
    """
    import transformers

    try:
        revision = kwargs.get("revision") or kwargs.get("tokenizer_revision")
        config_file = _resolve_local_or_cached_file(
            tokenizer_name, "tokenizer_config.json", revision
        )
        with open(config_file) as f:
            tok_config = json.load(f)
        tok_class_name = tok_config.get("tokenizer_class")
    except FileNotFoundError:
        return None
    except (OSError, json.JSONDecodeError) as e:
        logger.debug(
            "Failed to read tokenizer_config.json for %s: %s", tokenizer_name, e
        )
        return None

    if not tok_class_name:
        return None
# ... omitted for brevity ...
            "Falling back to AutoTokenizer result.",
            tok_class_name,
            tokenizer_name,
            e,
        )
        return None
```
**EN:** This function implements `_load_tokenizer_by_declared_class`. It primarily calls `getattr`, `logger.info`, `_resolve_local_or_cached_file`, `tok_config.get`, `kwargs.get`, `tok_cls.from_pretrained` to complete its work. State updates are written into `tok_cls`, `revision`, `config_file`, `tok_class_name`, `tok_config`, `auto_map`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该函数实现了 `_load_tokenizer_by_declared_class`。 它主要通过调用 `getattr`, `logger.info`, `_resolve_local_or_cached_file`, `tok_config.get`, `kwargs.get`, `tok_cls.from_pretrained` 来完成任务。 状态更新主要写入 `tok_cls`, `revision`, `config_file`, `tok_class_name`, `tok_config`, `auto_map`。 实现中使用了条件分支、上下文管理资源、错误处理。

### Lines 127-127: Class `TokenizerWarningsFilter` declaration / 类 `TokenizerWarningsFilter` 声明
```python
class TokenizerWarningsFilter(logging.Filter):
```
**EN:** This class establishes `TokenizerWarningsFilter` as the main container/coordinator for the surrounding logic. It inherits from `logging.Filter`. Its core interface includes methods such as `filter`.
**CN:** 该类将 `TokenizerWarningsFilter` 定义为周边逻辑的主要封装体或协调者。 它继承自 `logging.Filter`。 其核心接口包括 `filter` 等方法。

### Lines 128-129: Method `TokenizerWarningsFilter.filter` / 方法 `TokenizerWarningsFilter.filter`
```python
    def filter(self, record: logging.LogRecord) -> bool:
        return "Calling super().encode with" not in record.getMessage()
```
**EN:** This method implements `filter` on `TokenizerWarningsFilter`. It primarily calls `record.getMessage` to complete its work.
**CN:** 该方法（属于 `TokenizerWarningsFilter`）实现了 `filter`。 它主要通过调用 `record.getMessage` 来完成任务。

### Lines 137-159: Function `_resolve_tokenizer_name` / 函数 `_resolve_tokenizer_name`
```python
def _resolve_tokenizer_name(tokenizer_name, kwargs):
    """Resolve special name formats (GGUF, remote URLs, etc.) to a local path.

    May mutate *kwargs* (e.g. to add ``gguf_file``).
    """
    tokenizer_name = _MISTRAL_TOKENIZER_REDIRECTS.get(tokenizer_name, tokenizer_name)

    if check_gguf_file(tokenizer_name):
        _ensure_gguf_version()
        kwargs["gguf_file"] = tokenizer_name
        tokenizer_name = Path(tokenizer_name).parent

    tokenizer_name = resolve_runai_obj_uri(tokenizer_name)

    if is_remote_url(tokenizer_name):
        # BaseConnector implements __del__() to clean up the local dir.
        # Since config files need to exist all the time, so we DO NOT use
        # with statement to avoid closing the client.
        client = create_remote_connector(tokenizer_name)
        client.pull_files(ignore_pattern=["*.pt", "*.safetensors", "*.bin"])
        tokenizer_name = client.get_local_dir()

    return tokenizer_name
```
**EN:** This function implements `_resolve_tokenizer_name`. It primarily calls `_MISTRAL_TOKENIZER_REDIRECTS.get`, `check_gguf_file`, `resolve_runai_obj_uri`, `is_remote_url`, `_ensure_gguf_version`, `create_remote_connector` to complete its work. State updates are written into `tokenizer_name`, `kwargs`, `client`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_resolve_tokenizer_name`。 它主要通过调用 `_MISTRAL_TOKENIZER_REDIRECTS.get`, `check_gguf_file`, `resolve_runai_obj_uri`, `is_remote_url`, `_ensure_gguf_version`, `create_remote_connector` 来完成任务。 状态更新主要写入 `tokenizer_name`, `kwargs`, `client`。 实现中使用了条件分支。

### Lines 162-199: Function `_auto_tokenizer_from_pretrained` / 函数 `_auto_tokenizer_from_pretrained`
```python
def _auto_tokenizer_from_pretrained(tokenizer_name, *args, **common_kwargs):
    """Call ``AutoTokenizer.from_pretrained`` with error handling."""
    try:
        tokenizer = AutoTokenizer.from_pretrained(
            tokenizer_name, *args, **common_kwargs
        )
        logging.getLogger(tokenizer.__class__.__module__).addFilter(
            TokenizerWarningsFilter()
        )
        return tokenizer
    except TypeError as e:
        err_msg = (
            "Failed to load the tokenizer. If you are using a LLaMA V1 model "
            f"consider using '{_FAST_LLAMA_TOKENIZER}' instead of the "
            "original tokenizer."
        )
        raise RuntimeError(err_msg) from e
    except ValueError as e:
        # MistralCommon tokenizers reject standard HF kwargs like
        # trust_remote_code, use_fast etc. Retry without them.
        if "are not supported by" in str(e) and "MistralCommon" in str(e):
            return retry_without_mistral_common_kwargs(
                tokenizer_name, *args, **common_kwargs
            )
        # If the error pertains to the tokenizer class not existing or not
        # currently being imported, suggest using the --trust-remote-code flag.
        if not common_kwargs.get("trust_remote_code") and (
            "does not exist or is not currently imported." in str(e)
# ... omitted for brevity ...
                "tokenizer not yet available in the HuggingFace transformers "
                "library, consider setting `trust_remote_code=True` in LLM "
                "or using the `--trust-remote-code` flag in the CLI."
            )
            raise RuntimeError(err_msg) from e
        raise
```
**EN:** This function implements `_auto_tokenizer_from_pretrained`. It primarily calls `AutoTokenizer.from_pretrained`, `logging.getLogger.addFilter`, `TokenizerWarningsFilter`, `RuntimeError`, `logging.getLogger`, `retry_without_mistral_common_kwargs` to complete its work. State updates are written into `tokenizer`, `err_msg`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_auto_tokenizer_from_pretrained`。 它主要通过调用 `AutoTokenizer.from_pretrained`, `logging.getLogger.addFilter`, `TokenizerWarningsFilter`, `RuntimeError`, `logging.getLogger`, `retry_without_mistral_common_kwargs` 来完成任务。 状态更新主要写入 `tokenizer`, `err_msg`。 实现中使用了条件分支、错误处理。

### Lines 202-248: Function `_resolve_tokenizers_backend` / 函数 `_resolve_tokenizers_backend`
```python
def _resolve_tokenizers_backend(tokenizer_name, *args, **common_kwargs):
    """Resolve generic ``TokenizersBackend`` to a proper tokenizer class.

    In transformers v5, ``AutoTokenizer`` falls back to ``TokenizersBackend``
    when the model_type has no tokenizer mapping.  This retries with
    ``use_fast=False``, then attempts loading by the class declared in
    ``tokenizer_config.json``.  May still return a ``TokenizersBackend``
    if all retries fail (with a warning).
    """
    logger.warning(
        "Tokenizer loaded as generic TokenizersBackend for %s, "
        "retrying with use_fast=False",
        tokenizer_name,
    )
    common_kwargs = {**common_kwargs, "use_fast": False}
    try:
        tokenizer = AutoTokenizer.from_pretrained(
            tokenizer_name, *args, **common_kwargs
        )
    except (ValueError, TypeError, OSError, ImportError, RuntimeError) as e:
        raise RuntimeError(
            f"Retry with use_fast=False for {tokenizer_name} also failed "
            f"(initial load returned TokenizersBackend): {e}"
        ) from e

    if type(tokenizer).__name__ == _TOKENIZERS_BACKEND:
        tokenizer = (
            _load_tokenizer_by_declared_class(tokenizer_name, *args, **common_kwargs)
# ... omitted for brevity ...
                "Tokenizer for %s loaded as generic TokenizersBackend. "
                "Set --trust-remote-code to load the model-specific tokenizer.",
                tokenizer_name,
            )

    return tokenizer
```
**EN:** This function implements `_resolve_tokenizers_backend`. It primarily calls `logger.warning`, `AutoTokenizer.from_pretrained`, `common_kwargs.get`, `RuntimeError`, `type`, `_load_tokenizer_by_declared_class` to complete its work. State updates are written into `common_kwargs`, `tokenizer`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_resolve_tokenizers_backend`。 它主要通过调用 `logger.warning`, `AutoTokenizer.from_pretrained`, `common_kwargs.get`, `RuntimeError`, `type`, `_load_tokenizer_by_declared_class` 来完成任务。 状态更新主要写入 `common_kwargs`, `tokenizer`。 实现中使用了条件分支、错误处理。

### Lines 256-304: Function `_fix_v5_tokenizer_components` / 函数 `_fix_v5_tokenizer_components`
```python
def _fix_v5_tokenizer_components(tokenizer, model_name_or_path, revision=None):
    """Fix pre_tokenizer/decoder when a v5 tokenizer class overwrites them.

    In transformers v5, some tokenizer classes (e.g. LlamaTokenizer) have a
    custom __init__ that rebuilds the pre_tokenizer and decoder from scratch
    with class-specific components, discarding the originals from tokenizer.json.
    This breaks models that specify LlamaTokenizerFast but actually use a
    different tokenizer architecture (e.g. DeepSeek-V3.2 uses ByteLevel).

    Detects the mismatch by comparing against the raw tokenizer.json and
    restores the original components when they differ.
    """
    backend = getattr(tokenizer, "_tokenizer", None)
    if backend is None:
        return

    try:
        from tokenizers import Tokenizer as RawTokenizer

        tok_file = _resolve_local_or_cached_file(
            model_name_or_path, "tokenizer.json", revision
        )
        raw = RawTokenizer.from_file(tok_file)
    except FileNotFoundError:
        return
    except (OSError, ValueError, RuntimeError) as e:
        logger.warning(
            "_fix_v5_tokenizer_components: unexpected error loading tokenizer.json "
# ... omitted for brevity ...
            raw_pre,
            type(backend.decoder).__name__ if backend.decoder else None,
            type(raw.decoder).__name__ if raw.decoder else None,
        )
        backend.pre_tokenizer = raw.pre_tokenizer
        backend.decoder = raw.decoder
```
**EN:** This function implements `_fix_v5_tokenizer_components`. It primarily calls `getattr`, `_resolve_local_or_cached_file`, `RawTokenizer.from_file`, `logger.info`, `logger.warning`, `type` to complete its work. State updates are written into `backend`, `raw_pre`, `loaded_pre`, `tok_file`, `raw`, `backend.pre_tokenizer`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_fix_v5_tokenizer_components`。 它主要通过调用 `getattr`, `_resolve_local_or_cached_file`, `RawTokenizer.from_file`, `logger.info`, `logger.warning`, `type` 来完成任务。 状态更新主要写入 `backend`, `raw_pre`, `loaded_pre`, `tok_file`, `raw`, `backend.pre_tokenizer`。 实现中使用了条件分支、错误处理。

### Lines 307-401: Function `_fix_v5_add_bos_eos_token` / 函数 `_fix_v5_add_bos_eos_token`
```python
def _fix_v5_add_bos_eos_token(tokenizer, model_name_or_path, revision=None):
    """Restore add_bos_token/add_eos_token stripped by transformers v5.

    In transformers v5, _from_pretrained() strips add_bos_token and
    add_eos_token from init kwargs when a tokenizer.json file is present,
    assuming the tokenizer.json post-processor handles BOS/EOS addition.
    However, many models (e.g. DeepSeek-V3) have a tokenizer.json whose
    post-processor does NOT add BOS/EOS, and rely on the add_bos_token flag
    from tokenizer_config.json instead. This causes silent accuracy regressions.

    This function reads the tokenizer_config.json and restores the values,
    but only for tokenizer classes that actually supported these flags in v4.
    Classes like Qwen2Tokenizer did not support add_bos_token/add_eos_token
    in v4, so restoring them would change behavior.
    """
    # In transformers v4, only certain tokenizer classes supported
    # add_bos_token / add_eos_token as init parameters.  Restoring these
    # flags for classes that never supported them (e.g. Qwen2Tokenizer)
    # would incorrectly change tokenization behavior.
    _V4_CLASSES_WITH_BOS_EOS_FLAGS = frozenset(
        {
            "LlamaTokenizer",
            "LlamaTokenizerFast",
            "CodeLlamaTokenizer",
            "CodeLlamaTokenizerFast",
            "GemmaTokenizer",
            "GemmaTokenizerFast",
            "CohereTokenizerFast",
# ... omitted for brevity ...
            setattr(tokenizer, f"_{attr}", config_val)
            changed = True

    # Rebuild the post-processor so it respects the restored flags
    if changed and hasattr(tokenizer, "update_post_processor"):
        tokenizer.update_post_processor()
```
**EN:** This function implements `_fix_v5_add_bos_eos_token`. It primarily calls `frozenset`, `config.get`, `_resolve_local_or_cached_file`, `logger.debug`, `getattr`, `hasattr` to complete its work. State updates are written into `_V4_CLASSES_WITH_BOS_EOS_FLAGS`, `tokenizer_class`, `_V4_DEFAULTS`, `changed`, `config_file`, `config_val`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `_fix_v5_add_bos_eos_token`。 它主要通过调用 `frozenset`, `config.get`, `_resolve_local_or_cached_file`, `logger.debug`, `getattr`, `hasattr` 来完成任务。 状态更新主要写入 `_V4_CLASSES_WITH_BOS_EOS_FLAGS`, `tokenizer_class`, `_V4_DEFAULTS`, `changed`, `config_file`, `config_val`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 404-413: Function `_fix_special_tokens_pattern` / 函数 `_fix_special_tokens_pattern`
```python
def _fix_special_tokens_pattern(tokenizer):
    """Fix https://github.com/huggingface/transformers/pull/42563 which defaults
    special_tokens_pattern to "cls_sep", inserting None into token IDs when
    cls_token/sep_token are undefined (e.g. Kimi-VL's TikTokenTokenizer).
    """
    pattern = getattr(tokenizer, "special_tokens_pattern", None)
    if pattern == "cls_sep" and (
        tokenizer.cls_token_id is None or tokenizer.sep_token_id is None
    ):
        tokenizer.special_tokens_pattern = "none"
```
**EN:** This function implements `_fix_special_tokens_pattern`. It primarily calls `getattr` to complete its work. State updates are written into `pattern`, `tokenizer.special_tokens_pattern`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_fix_special_tokens_pattern`。 它主要通过调用 `getattr` 来完成任务。 状态更新主要写入 `pattern`, `tokenizer.special_tokens_pattern`。 实现中使用了条件分支。

### Lines 416-430: Function `_apply_post_load_fixes` / 函数 `_apply_post_load_fixes`
```python
def _apply_post_load_fixes(tokenizer, tokenizer_name, revision):
    """Apply all post-load patches and return the final tokenizer."""
    _fix_v5_tokenizer_components(tokenizer, tokenizer_name, revision)
    _fix_v5_add_bos_eos_token(tokenizer, tokenizer_name, revision)

    if not isinstance(tokenizer, PreTrainedTokenizerFast):
        warnings.warn(
            "Using a slow tokenizer. This might cause a significant "
            "slowdown. Consider using a fast tokenizer instead."
        )

    patch_mistral_common_tokenizer(tokenizer)
    _fix_special_tokens_pattern(tokenizer)
    attach_additional_stop_token_ids(tokenizer)
    return patch_tokenizer(tokenizer)
```
**EN:** This function implements `_apply_post_load_fixes`. It primarily calls `_fix_v5_tokenizer_components`, `_fix_v5_add_bos_eos_token`, `patch_mistral_common_tokenizer`, `_fix_special_tokens_pattern`, `attach_additional_stop_token_ids`, `patch_tokenizer` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_apply_post_load_fixes`。 它主要通过调用 `_fix_v5_tokenizer_components`, `_fix_v5_add_bos_eos_token`, `patch_mistral_common_tokenizer`, `_fix_special_tokens_pattern`, `attach_additional_stop_token_ids`, `patch_tokenizer` 来完成任务。 实现中使用了条件分支。

### Lines 438-438: Constants and shared state / 常量与共享状态
```python
_fastokens_patched = False
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_fastokens_patched`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_fastokens_patched`。

### Lines 441-456: Function `_ensure_fastokens_patched` / 函数 `_ensure_fastokens_patched`
```python
def _ensure_fastokens_patched():
    """Monkey-patch transformers to use the fastokens backend (once)."""
    global _fastokens_patched
    if _fastokens_patched:
        return
    try:
        import fastokens
    except ImportError:
        raise ImportError(
            "The fastokens package is required when --tokenizer-backend=fastokens. "
            "Install it with: pip install 'sglang[fastokens]'"
        ) from None

    fastokens.patch_transformers()
    _fastokens_patched = True
    logger.info("fastokens backend enabled - transformers patched successfully")
```
**EN:** This function implements `_ensure_fastokens_patched`. It primarily calls `fastokens.patch_transformers`, `logger.info`, `ImportError` to complete its work. State updates are written into `_fastokens_patched`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_ensure_fastokens_patched`。 它主要通过调用 `fastokens.patch_transformers`, `logger.info`, `ImportError` 来完成任务。 状态更新主要写入 `_fastokens_patched`。 实现中使用了条件分支、错误处理。

### Lines 459-523: Function `get_tokenizer` / 函数 `get_tokenizer`
```python
def get_tokenizer(
    tokenizer_name: str,
    *args,
    tokenizer_mode: str = "auto",
    trust_remote_code: bool = False,
    tokenizer_revision: Optional[str] = None,
    tokenizer_backend: str = "huggingface",
    **kwargs,
) -> Union[PreTrainedTokenizer, PreTrainedTokenizerFast]:
    """Gets a tokenizer for the given model name via Huggingface."""
    # Tiktoken format has its own backend — no fastokens patching needed.
    if tokenizer_name.endswith(".json"):
        from sglang.srt.tokenizer.tiktoken_tokenizer import TiktokenTokenizer

        return TiktokenTokenizer(tokenizer_name)

    if tokenizer_backend == "fastokens":
        _ensure_fastokens_patched()

    if tokenizer_mode == "slow":
        if kwargs.get("use_fast", False):
            raise ValueError("Cannot use the fast tokenizer in slow tokenizer mode.")
        kwargs["use_fast"] = False
    elif tokenizer_mode == "auto":
        # Transformers v5 AutoTokenizer ignores use_fast (always fast), but
        # some code paths pass kwargs to non-AutoTokenizer loaders where
        # use_fast still matters. Set explicitly for those fallback paths.
        if "use_fast" not in kwargs:
# ... omitted for brevity ...
                f"fastokens failed to load tokenizer for {tokenizer_name!r}. "
                f"This model's tokenizer may not be supported by fastokens — "
                f"see https://github.com/crusoecloud/fastokens. "
                f"Re-run without --tokenizer-backend=fastokens to use the default backend."
            ) from e
        raise
```
**EN:** This function implements `get_tokenizer`. It primarily calls `tokenizer_name.endswith`, `_resolve_tokenizer_name`, `dict`, `TiktokenTokenizer`, `_ensure_fastokens_patched`, `kwargs.get` to complete its work. State updates are written into `tokenizer_name`, `common_kwargs`, `kwargs`, `tokenizer`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `get_tokenizer`。 它主要通过调用 `tokenizer_name.endswith`, `_resolve_tokenizer_name`, `dict`, `TiktokenTokenizer`, `_ensure_fastokens_patched`, `kwargs.get` 来完成任务。 状态更新主要写入 `tokenizer_name`, `common_kwargs`, `kwargs`, `tokenizer`。 实现中使用了条件分支、错误处理。

### Lines 531-595: Function `_fix_added_tokens_encoding` / 函数 `_fix_added_tokens_encoding`
```python
def _fix_added_tokens_encoding(tokenizer):
    """Ensure special tokens encode as single tokens in transformers v5.

    Some model tokenizers (e.g. MiniCPM-V-4) define special tokens like <image>,
    <slice> as attributes on the tokenizer class with corresponding IDs in the
    vocabulary (via tokenizer.json's added_tokens). In transformers v5, these
    tokens may not appear in get_added_vocab() and encode() splits them into
    subwords, breaking multimodal pipelines that rely on finding them in input_ids.

    This function discovers such tokens by scanning tokenizer attributes, checks
    if they encode correctly, and re-registers any that don't.
    """

    # Discover special token strings from tokenizer attributes.
    # Model tokenizers (e.g. MiniCPMVTokenizerFast) store them as attributes
    # like im_start="<image>", slice_start="<slice>", etc.
    def _is_special_token_attr(val):
        return (
            isinstance(val, str)
            and val.startswith("<")
            and val.endswith(">")
            and len(val) <= 20
        )

    candidates = {}
    for attr in dir(tokenizer):
        if attr.startswith("_"):
            continue
# ... omitted for brevity ...
    tokenizer.add_tokens(tokens_to_add, special_tokens=True)
    logger.info(
        "Re-registered %d special tokens for correct v5 encoding: %s",
        len(broken),
        broken[:10],
    )
```
**EN:** This function implements `_fix_added_tokens_encoding`. It primarily calls `dir`, `tokenizer.add_tokens`, `logger.info`, `attr.startswith`, `tokenizer.convert_tokens_to_ids`, `AddedToken` to complete its work. State updates are written into `candidates`, `broken`, `tokens_to_add`, `token_id`, `val`, `ids`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_fix_added_tokens_encoding`。 它主要通过调用 `dir`, `tokenizer.add_tokens`, `logger.info`, `attr.startswith`, `tokenizer.convert_tokens_to_ids`, `AddedToken` 来完成任务。 状态更新主要写入 `candidates`, `broken`, `tokens_to_add`, `token_id`, `val`, `ids`。 实现中使用了条件分支、迭代逻辑、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `TokenizerWarningsFilter`
- **Functions / 函数**: `_load_tokenizer_by_declared_class`, `_resolve_tokenizer_name`, `_auto_tokenizer_from_pretrained`, `_resolve_tokenizers_backend`, `_fix_v5_tokenizer_components`, `_fix_v5_add_bos_eos_token`, `_fix_special_tokens_pattern`, `_apply_post_load_fixes`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.connector`, `sglang.srt.utils`, `sglang.srt.utils.patch_tokenizer`, `..hf_transformers_patches`, `.common`, `.mistral_utils`, `sglang.srt.tokenizer.tiktoken_tokenizer`
- **External / 外部依赖**: `transformers`, `tokenizers`, `fastokens`, `transformers.dynamic_module_utils`
- **Standard library / 标准库**: `json`, `logging`, `warnings`, `pathlib`, `typing`
