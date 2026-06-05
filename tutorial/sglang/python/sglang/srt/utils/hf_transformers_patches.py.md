# hf_transformers_patches.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/hf_transformers_patches.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for Hugging Face integration helpers. / 为 SGLang 运行时提供面向Hugging Face 集成辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 14-29: Module setup and shared state / 模块设置与共享状态
```python
"""Monkey-patches on transformers internals.

Mix of backward-compat shims (re-add symbols removed in v5), workarounds
for transformers v5 bugs, fixes for remote-model-code (trust_remote_code)
that hasn't been updated for v5 yet, and CI-only patches (e.g. neutralize
HF API calls to avoid rate limits).

Import this module early (before any ``from_pretrained`` call) to activate
all patches.  It is safe to import multiple times -- patches are idempotent.
"""

import inspect

from sglang.srt.utils import logger

_applied = False
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `inspect`, `sglang.srt.utils`. It also defines symbols such as `_applied` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `inspect`, `sglang.srt.utils`。 同时定义了 `_applied` 等符号，供后续逻辑使用。

### Lines 37-71: Function `apply_all` / 函数 `apply_all`
```python
def apply_all():
    """Apply all transformers compatibility patches (idempotent).

    Call this once at import time.  It is safe to call multiple times.

    No-op when the ``transformers`` package is not installed -- frontend-only
    sglang users should not be forced to install transformers just to import
    the top-level ``sglang`` package.
    """
    global _applied
    if _applied:
        return
    try:
        import transformers  # noqa: F401
    except ImportError:
        _applied = True
        return
    _applied = True

    # v5.4 patches
    _patch_flash_attn_availability()
    _patch_rope_parameters_validation()
    _patch_removed_symbols()
    _patch_image_processor_kwargs()
    _patch_image_process_cuda_tensor()
    _patch_nemotron_h_pattern()

    # v5 general patches
    _ensure_clean_up_tokenization_compat()
    _ensure_is_torch_fx_available_compat()

    # CI-only: neutralize HF API calls inside tokenizer from_pretrained
    patch_is_base_mistral_in_ci()

    logger.debug("transformers compatibility patches applied")
```
**EN:** This function implements `apply_all`. It primarily calls `_patch_flash_attn_availability`, `_patch_rope_parameters_validation`, `_patch_removed_symbols`, `_patch_image_processor_kwargs`, `_patch_image_process_cuda_tensor`, `_patch_nemotron_h_pattern` to complete its work. State updates are written into `_applied`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `apply_all`。 它主要通过调用 `_patch_flash_attn_availability`, `_patch_rope_parameters_validation`, `_patch_removed_symbols`, `_patch_image_processor_kwargs`, `_patch_image_process_cuda_tensor`, `_patch_nemotron_h_pattern` 来完成任务。 状态更新主要写入 `_applied`。 实现中使用了条件分支、错误处理。

### Lines 79-105: Function `normalize_rope_scaling_compat` / 函数 `normalize_rope_scaling_compat`
```python
def normalize_rope_scaling_compat(config) -> None:
    """Ensure rope_scaling dicts have ``"type"`` alongside ``"rope_type"``.

    Transformers v5 standardises rope_scaling to use ``"rope_type"`` and may
    omit the legacy ``"type"`` key.  Remote-code models (e.g. Kimi-VL) still
    read ``rope_scaling["type"]``, causing a ``KeyError``.  This helper adds
    ``"type"`` from ``"rope_type"`` whenever it is missing, recursively across
    the config and all its sub-configs.
    """

    def _patch(cfg):
        rs = getattr(cfg, "rope_scaling", None)
        if isinstance(rs, dict) and "rope_type" in rs and "type" not in rs:
            rs["type"] = rs["rope_type"]
        # Recurse into sub-configs
        for attr in (
            "text_config",
            "llm_config",
            "language_config",
            "vision_config",
            "thinker_config",
        ):
            sub = getattr(cfg, attr, None)
            if sub is not None:
                _patch(sub)

    _patch(config)
```
**EN:** This function implements `normalize_rope_scaling_compat`. It primarily calls `_patch`, `getattr`, `isinstance` to complete its work. State updates are written into `rs`, `sub`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `normalize_rope_scaling_compat`。 它主要通过调用 `_patch`, `getattr`, `isinstance` 来完成任务。 状态更新主要写入 `rs`, `sub`。 实现中使用了条件分支、迭代逻辑。

### Lines 108-130: Function `_ensure_gguf_version` / 函数 `_ensure_gguf_version`
```python
def _ensure_gguf_version():
    """Workaround for transformers v5 bug where is_gguf_available() fails
    when the gguf package lacks __version__ and metadata lookup also fails,
    resulting in packaging.version.InvalidVersion: Invalid version: 'N/A'."""
    try:
        import gguf

        if not hasattr(gguf, "__version__"):
            import importlib.metadata

            try:
                gguf.__version__ = importlib.metadata.version("gguf")
            except importlib.metadata.PackageNotFoundError:
                gguf.__version__ = "0.0.0"
            except (ValueError, OSError, TypeError) as e:
                logger.warning(
                    "Failed to determine gguf package version: %s. "
                    "Falling back to '0.0.0'.",
                    e,
                )
                gguf.__version__ = "0.0.0"
    except ImportError:
        pass
```
**EN:** This function implements `_ensure_gguf_version`. It primarily calls `hasattr`, `importlib.metadata.version`, `logger.warning` to complete its work. State updates are written into `gguf.__version__`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_ensure_gguf_version`。 它主要通过调用 `hasattr`, `importlib.metadata.version`, `logger.warning` 来完成任务。 状态更新主要写入 `gguf.__version__`。 实现中使用了条件分支、错误处理。

### Lines 138-184: Function `_patch_rope_parameters_validation` / 函数 `_patch_rope_parameters_validation`
```python
def _patch_rope_parameters_validation():
    """Fix rope_parameters validation for unregistered model types.

    For unregistered model types (e.g. ``deepseek_v32``), the generic
    ``PretrainedConfig`` lacks a ``rope_parameters`` field so the conversion
    that injects ``rope_theta`` from the top-level config is skipped.
    Additionally, ``standardize_rope_params()`` accesses
    ``self.max_position_embeddings`` during ``__post_init__`` before extra
    kwargs are set as attributes, causing ``AttributeError``.

    Fix: (1) patch ``from_dict`` to inject ``rope_theta`` into
    ``rope_scaling``, (2) guard ``standardize_rope_params`` against missing
    ``max_position_embeddings``.

    TODO(upstream): remove once unregistered model types handle rope
    standardization correctly in transformers.
    """
    from transformers import PretrainedConfig

    original = PretrainedConfig.from_dict.__func__

    @classmethod  # type: ignore[misc]
    def patched(cls, config_dict, **kwargs):
        rope_scaling = config_dict.get("rope_scaling")
        rope_theta = config_dict.get("rope_theta")
        if (
            isinstance(rope_scaling, dict)
            and rope_theta is not None
# ... omitted for brevity ...
        def _safe_standardize(self):
            if not hasattr(self, "max_position_embeddings"):
                return
            return _orig_standardize(self)

        PretrainedConfig.standardize_rope_params = _safe_standardize
```
**EN:** This function implements `_patch_rope_parameters_validation`. It primarily calls `hasattr`, `config_dict.get`, `original`, `isinstance`, `config_dict.copy`, `_orig_standardize` to complete its work. State updates are written into `original`, `PretrainedConfig.from_dict`, `rope_scaling`, `rope_theta`, `_orig_standardize`, `PretrainedConfig.standardize_rope_params`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_patch_rope_parameters_validation`。 它主要通过调用 `hasattr`, `config_dict.get`, `original`, `isinstance`, `config_dict.copy`, `_orig_standardize` 来完成任务。 状态更新主要写入 `original`, `PretrainedConfig.from_dict`, `rope_scaling`, `rope_theta`, `_orig_standardize`, `PretrainedConfig.standardize_rope_params`。 实现中使用了条件分支。

### Lines 187-206: Function `_patch_flash_attn_availability` / 函数 `_patch_flash_attn_availability`
```python
def _patch_flash_attn_availability():
    """Prevent flash-attn-4 from masquerading as flash-attn-2.

    flash-attn-4 registers a bare ``flash_attn`` namespace that makes
    ``is_flash_attn_2_available()`` return True, but lacks the v2 API.
    Remote model code (e.g. Kimi-VL) guarded by that check will crash.

    TODO(upstream): model authors should check for specific API symbols.
    """
    try:
        import flash_attn as _fa

        if not hasattr(_fa, "flash_attn_func"):
            import transformers.utils as _u
            import transformers.utils.import_utils as _ui

            _ui.is_flash_attn_2_available = lambda: False
            _u.is_flash_attn_2_available = lambda: False
    except ImportError:
        pass
```
**EN:** This function implements `_patch_flash_attn_availability`. It primarily calls `hasattr` to complete its work. State updates are written into `_ui.is_flash_attn_2_available`, `_u.is_flash_attn_2_available`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_patch_flash_attn_availability`。 它主要通过调用 `hasattr` 来完成任务。 状态更新主要写入 `_ui.is_flash_attn_2_available`, `_u.is_flash_attn_2_available`。 实现中使用了条件分支、错误处理。

### Lines 209-263: Function `_patch_removed_symbols` / 函数 `_patch_removed_symbols`
```python
def _patch_removed_symbols():
    """Re-export symbols removed in transformers v5.4.0.

    Remote model code (e.g. DeepSeek-OCR) still imports these.
    ``check_imports`` in ``dynamic_module_utils.py`` validates imports at
    config-load time, so these must exist before any ``from_pretrained``.

    Removed symbols:
    - ``LlamaFlashAttention2`` -- replaced by unified ``LlamaAttention``
    - ``is_flash_attn_greater_or_equal_2_10`` -- replaced by
      ``is_flash_attn_greater_or_equal("2.10.0")``

    TODO(upstream): DeepSeek-OCR / deepseek_vl_v2 remote code needs update.
    """
    # LlamaFlashAttention2
    try:
        import logging

        # Importing modeling_llama triggers a deep import chain:
        #   modeling_llama -> modeling_utils -> quantizers -> torchao
        # torchao emits a noisy warning about incompatible torch versions
        # that is irrelevant here — suppress it during this import.
        _torchao_logger = logging.getLogger("torchao")
        _prev_level = _torchao_logger.level
        _torchao_logger.setLevel(logging.ERROR)
        try:
            from transformers.models.llama import modeling_llama
        finally:
# ... omitted for brevity ...
                _u.is_flash_attn_greater_or_equal_2_10 = lambda: False
    except ImportError:
        logger.warning(
            "Could not import transformers.utils; "
            "is_flash_attn_greater_or_equal_2_10 compat patch not applied."
        )
```
**EN:** This function implements `_patch_removed_symbols`. It primarily calls `logging.getLogger`, `_torchao_logger.setLevel`, `hasattr`, `logger.warning`, `_u.is_flash_attn_greater_or_equal` to complete its work. State updates are written into `_torchao_logger`, `_prev_level`, `modeling_llama.LlamaFlashAttention2`, `_u.is_flash_attn_greater_or_equal_2_10`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_patch_removed_symbols`。 它主要通过调用 `logging.getLogger`, `_torchao_logger.setLevel`, `hasattr`, `logger.warning`, `_u.is_flash_attn_greater_or_equal` 来完成任务。 状态更新主要写入 `_torchao_logger`, `_prev_level`, `modeling_llama.LlamaFlashAttention2`, `_u.is_flash_attn_greater_or_equal_2_10`。 实现中使用了条件分支、错误处理。

### Lines 266-312: Function `_patch_image_processor_kwargs` / 函数 `_patch_image_processor_kwargs`
```python
def _patch_image_processor_kwargs():
    """Allow remote image processors that lack ``**kwargs`` in preprocess().

    Transformers v5.4 passes new kwargs (e.g. ``device``) through
    ``BaseImageProcessor.__call__`` -> ``preprocess()``.  Remote model code
    (e.g. KimiVL) that defines ``preprocess()`` without ``**kwargs`` will
    crash with ``TypeError``.

    Fix: wrap ``__call__`` to catch ``TypeError`` and retry with only the
    kwargs that ``preprocess()`` actually accepts.

    TODO(upstream): KimiVL image_processing_kimi_vl.py needs ``**kwargs``.
    """
    try:
        from transformers.image_processing_utils import BaseImageProcessor

        original = BaseImageProcessor.__call__

        def safe_call(self, images, *args, **kwargs):
            try:
                return original(self, images, *args, **kwargs)
            except TypeError as e:
                if "unexpected keyword argument" not in str(e):
                    raise
                sig = inspect.signature(self.preprocess)
                params = sig.parameters
                if any(
                    p.kind == inspect.Parameter.VAR_KEYWORD for p in params.values()
# ... omitted for brevity ...

        BaseImageProcessor.__call__ = safe_call
    except ImportError:
        logger.debug(
            "_patch_image_processor_kwargs: BaseImageProcessor not importable, patch skipped"
        )
```
**EN:** This function implements `_patch_image_processor_kwargs`. It primarily calls `logger.debug`, `original`, `inspect.signature`, `any`, `str`, `logger.warning` to complete its work. State updates are written into `original`, `BaseImageProcessor.__call__`, `sig`, `params`, `dropped`, `valid`. The implementation relies on conditional branches, error handling.
**CN:** 该函数实现了 `_patch_image_processor_kwargs`。 它主要通过调用 `logger.debug`, `original`, `inspect.signature`, `any`, `str`, `logger.warning` 来完成任务。 状态更新主要写入 `original`, `BaseImageProcessor.__call__`, `sig`, `params`, `dropped`, `valid`。 实现中使用了条件分支、错误处理。

### Lines 315-345: Function `_patch_image_process_cuda_tensor` / 函数 `_patch_image_process_cuda_tensor`
```python
def _patch_image_process_cuda_tensor():
    """Fix ``process_image()`` crashing on CUDA tensors.

    Transformers v5.4's PIL image processing backend calls
    ``image.numpy()`` on torch tensors, which fails for CUDA tensors.
    Patch to call ``.cpu().numpy()`` instead.

    TODO(upstream): report to HF transformers.
    """
    try:
        import torch
        import transformers.image_processing_backends as ipb

        for cls_name in ("PilBackend", "PilImageProcessingMixin"):
            cls = getattr(ipb, cls_name, None)
            if cls is None or not hasattr(cls, "process_image"):
                continue
            original = cls.process_image

            def patched_process_image(
                self, image, *args, _orig=original, _Tensor=torch.Tensor, **kwargs
            ):
                if isinstance(image, _Tensor) and image.is_cuda:
                    image = image.cpu()
                return _orig(self, image, *args, **kwargs)

            cls.process_image = patched_process_image
    except ImportError:
        logger.debug(
            "_patch_image_process_cuda_tensor: required modules not importable, patch skipped"
        )
```
**EN:** This function implements `_patch_image_process_cuda_tensor`. It primarily calls `getattr`, `logger.debug`, `_orig`, `hasattr`, `isinstance`, `image.cpu` to complete its work. State updates are written into `cls`, `original`, `cls.process_image`, `image`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_patch_image_process_cuda_tensor`。 它主要通过调用 `getattr`, `logger.debug`, `_orig`, `hasattr`, `isinstance`, `image.cpu` 来完成任务。 状态更新主要写入 `cls`, `original`, `cls.process_image`, `image`。 实现中使用了条件分支、迭代逻辑、错误处理。

### Lines 348-380: Function `_patch_nemotron_h_pattern` / 函数 `_patch_nemotron_h_pattern`
```python
def _patch_nemotron_h_pattern():
    """Fix ``_pattern_to_list()`` crashing on ``-`` in hybrid_override_pattern.

    Nemotron-H models (e.g. NVIDIA-Nemotron-Nano-9B-v2) use patterns like
    ``M-M-M-MM-M-*-...`` where ``-`` denotes an MLP layer.  The upstream
    ``_pattern_to_list`` tries to map every character and crashes with
    ``KeyError: '-'``.  We skip ``-`` (and any other unmapped chars)
    since ``layers_block_type`` only tracks mamba/moe/attention layers.
    SGLang reads MLP positions from ``hybrid_override_pattern`` directly.

    TODO(upstream): report to HF transformers.
    """
    try:
        from transformers.models.nemotron_h.configuration_nemotron_h import (
            NemotronHConfig,
        )

        @staticmethod
        def _pattern_to_list(pattern: str) -> list:
            pattern_mapping = {
                "M": "mamba",
                "E": "moe",
                "*": "attention",
            }
            return [
                pattern_mapping[char] for char in pattern if char in pattern_mapping
            ]

        NemotronHConfig._pattern_to_list = _pattern_to_list
    except ImportError:
        logger.debug(
            "_patch_nemotron_h_pattern: NemotronHConfig not importable, patch skipped"
        )
```
**EN:** This function implements `_patch_nemotron_h_pattern`. It primarily calls `logger.debug` to complete its work. State updates are written into `NemotronHConfig._pattern_to_list`, `pattern_mapping`. The implementation relies on error handling.
**CN:** 该函数实现了 `_patch_nemotron_h_pattern`。 它主要通过调用 `logger.debug` 来完成任务。 状态更新主要写入 `NemotronHConfig._pattern_to_list`, `pattern_mapping`。 实现中使用了错误处理。

### Lines 388-417: Function `_ensure_clean_up_tokenization_compat` / 函数 `_ensure_clean_up_tokenization_compat`
```python
def _ensure_clean_up_tokenization_compat() -> None:
    """Re-add ``clean_up_tokenization`` removed in transformers v5.

    Remote-code tokenizers (e.g. InternLM2Tokenizer) call
    ``self.clean_up_tokenization()`` which was a static method on
    ``PreTrainedTokenizerBase`` in v4 but removed in v5. Patch it back
    so existing HuggingFace Hub tokenizer code keeps working.
    """
    from transformers import PreTrainedTokenizerBase

    if hasattr(PreTrainedTokenizerBase, "clean_up_tokenization"):
        return

    @staticmethod
    def clean_up_tokenization(out_string: str) -> str:
        out_string = (
            out_string.replace(" .", ".")
            .replace(" ?", "?")
            .replace(" !", "!")
            .replace(" ,", ",")
            .replace(" ' ", "'")
            .replace(" n't", "n't")
            .replace(" 'm", "'m")
            .replace(" 's", "'s")
            .replace(" 've", "'ve")
            .replace(" 're", "'re")
        )
        return out_string

    PreTrainedTokenizerBase.clean_up_tokenization = clean_up_tokenization
```
**EN:** This function implements `_ensure_clean_up_tokenization_compat`. It primarily calls `hasattr`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace` to complete its work. State updates are written into `PreTrainedTokenizerBase.clean_up_tokenization`, `out_string`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_ensure_clean_up_tokenization_compat`。 它主要通过调用 `hasattr`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace.replace`, `out_string.replace.replace.replace.replace.replace.replace` 来完成任务。 状态更新主要写入 `PreTrainedTokenizerBase.clean_up_tokenization`, `out_string`。 实现中使用了条件分支。

### Lines 420-433: Function `_ensure_is_torch_fx_available_compat` / 函数 `_ensure_is_torch_fx_available_compat`
```python
def _ensure_is_torch_fx_available_compat() -> None:
    """Re-add ``is_torch_fx_available`` removed in transformers v5.

    Remote-code models (e.g. MiniCPM-V) import ``is_torch_fx_available``
    from ``transformers.utils.import_utils``.  The function was removed
    in v5.  Patch it back so existing HuggingFace Hub model code keeps
    working.  torch.fx is always available in PyTorch >= 2.0.
    """
    import transformers.utils.import_utils as _import_utils

    if hasattr(_import_utils, "is_torch_fx_available"):
        return

    _import_utils.is_torch_fx_available = lambda: True
```
**EN:** This function implements `_ensure_is_torch_fx_available_compat`. It primarily calls `hasattr` to complete its work. State updates are written into `_import_utils.is_torch_fx_available`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_ensure_is_torch_fx_available_compat`。 它主要通过调用 `hasattr` 来完成任务。 状态更新主要写入 `_import_utils.is_torch_fx_available`。 实现中使用了条件分支。

### Lines 440-440: Constants and shared state / 常量与共享状态
```python
_is_base_mistral_patched = False
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_is_base_mistral_patched`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_is_base_mistral_patched`。

### Lines 443-476: Function `patch_is_base_mistral_in_ci` / 函数 `patch_is_base_mistral_in_ci`
```python
def patch_is_base_mistral_in_ci():
    """Patch transformers' _patch_mistral_regex to avoid HF API calls in CI.

    transformers defines is_base_mistral as a local function inside
    _patch_mistral_regex, so it cannot be patched via module attribute.
    Instead we replace the entire _patch_mistral_regex classmethod with a
    version that simply returns the tokenizer unchanged.

    In CI this prevents exhausting the 3000 req/5min HF API rate limit.

    TODO(upstream): remove once transformers stops calling model_info()
    inside _patch_mistral_regex (or removes the method entirely).
    """
    global _is_base_mistral_patched
    if _is_base_mistral_patched:
        return

    from sglang.srt.environ import envs

    if not envs.SGLANG_IS_IN_CI.get():
        return

    from transformers import PreTrainedTokenizerFast

    if hasattr(PreTrainedTokenizerFast, "_patch_mistral_regex"):

        @classmethod
        def _noop_patch_mistral_regex(cls, tokenizer, *args, **kwargs):
            return tokenizer

        PreTrainedTokenizerFast._patch_mistral_regex = _noop_patch_mistral_regex
        logger.info("CI: patched _patch_mistral_regex to skip HF API calls")

    _is_base_mistral_patched = True
```
**EN:** This function implements `patch_is_base_mistral_in_ci`. It primarily calls `hasattr`, `envs.SGLANG_IS_IN_CI.get`, `logger.info` to complete its work. State updates are written into `_is_base_mistral_patched`, `PreTrainedTokenizerFast._patch_mistral_regex`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `patch_is_base_mistral_in_ci`。 它主要通过调用 `hasattr`, `envs.SGLANG_IS_IN_CI.get`, `logger.info` 来完成任务。 状态更新主要写入 `_is_base_mistral_patched`, `PreTrainedTokenizerFast._patch_mistral_regex`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `apply_all`, `normalize_rope_scaling_compat`, `_ensure_gguf_version`, `_patch_rope_parameters_validation`, `_patch_flash_attn_availability`, `_patch_removed_symbols`, `_patch_image_processor_kwargs`, `_patch_image_process_cuda_tensor`
- **Themes / 主题**: `patch`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.utils`, `sglang.srt.environ`
- **External / 外部依赖**: `transformers`, `transformers.utils.import_utils`, `gguf`, `flash_attn`, `transformers.utils`, `transformers.image_processing_utils`, `torch`, `transformers.image_processing_backends`, `transformers.models.nemotron_h.configuration_nemotron_h`, `transformers.models.llama`
- **Standard library / 标准库**: `inspect`, `logging`, `importlib.metadata`
