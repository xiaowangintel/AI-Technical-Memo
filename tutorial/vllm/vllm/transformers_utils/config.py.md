# config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/config.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides the central configuration loading, patching, and compatibility helpers that adapt Hugging Face configs for vLLM. / [CN] 提供核心配置加载、补丁处理与兼容性辅助逻辑，用于把 Hugging Face 配置适配到 vLLM。

## Line-by-Line Analysis / 逐行分析

### Lines 4-49: Imports
```python
import os
from collections.abc import Callable, Iterator
from contextlib import contextmanager
from dataclasses import asdict
from functools import cache, partial
from importlib.metadata import version
from pathlib import Path
from typing import Any, Literal, TypeAlias

import huggingface_hub
import torch
from huggingface_hub import constants, get_safetensors_metadata
from packaging.version import Version
from safetensors.torch import _TYPES as _SAFETENSORS_TO_TORCH_DTYPE
from transformers import GenerationConfig, PretrainedConfig
from transformers.models.auto.image_processing_auto import get_image_processor_config
from transformers.models.auto.modeling_auto import (
    MODEL_FOR_CAUSAL_LM_MAPPING_NAMES,
    MODEL_MAPPING_NAMES,
)
from transformers.models.auto.tokenization_auto import get_tokenizer_config
from transformers.utils import CONFIG_NAME as HF_CONFIG_NAME

from vllm import envs
from vllm.logger import init_logger
from vllm.transformers_utils.repo_utils import is_mistral_model_repo
from vllm.transformers_utils.utils import (
    parse_safetensors_file_metadata,
    without_trust_remote_code,
)
from vllm.utils.torch_utils import common_broadcastable_dtype

from .config_parser_base import ConfigParserBase
from .gguf_utils import (
    check_gguf_file,
    is_gguf,
    is_remote_gguf,
    split_remote_gguf,
)
from .repo_utils import (
    file_or_path_exists,
    get_hf_file_to_dict,
    list_repo_files,
    try_get_local_file,
    with_retry,
)
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `os`, `collections.abc`, `contextlib`, `dataclasses`, `functools`, `importlib.metadata`, `pathlib`, `typing`, external APIs such as `huggingface_hub`, `torch`, `packaging.version`, `safetensors.torch`, `transformers`, `transformers.models.auto.image_processing_auto`, `transformers.models.auto.modeling_auto`, `transformers.models.auto.tokenization_auto`, ... (+1 more), and internal vLLM modules such as `vllm`, `vllm.logger`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.torch_utils`, `.config_parser_base`, `.gguf_utils`, `.repo_utils`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `os`, `collections.abc`, `contextlib`, `dataclasses`, `functools`, `importlib.metadata`, `pathlib`, `typing`，外部 API 如 `huggingface_hub`, `torch`, `packaging.version`, `safetensors.torch`, `transformers`, `transformers.models.auto.image_processing_auto`, `transformers.models.auto.modeling_auto`, `transformers.models.auto.tokenization_auto`, ... (+1 more)，以及 vLLM 内部模块如 `vllm`, `vllm.logger`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.torch_utils`, `.config_parser_base`, `.gguf_utils`, `.repo_utils`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 65-67: Module state and constants
```python
MISTRAL_CONFIG_NAME = "params.json"

logger = init_logger(__name__)
```
**EN:** This block defines module-level constants/defaults such as `MISTRAL_CONFIG_NAME`, `logger`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `MISTRAL_CONFIG_NAME`, `logger`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 87-148: Module state and constants
```python
_CONFIG_REGISTRY: dict[str, type[PretrainedConfig]] = LazyConfigDict(
    afmoe="AfmoeConfig",
    bagel="BagelConfig",
    umm="CheersConfig",
    chatglm="ChatGLMConfig",
    modernvbert="ColModernVBertConfig",
    colpali="ColPaliConfig",
    colqwen3="ColQwen3Config",
    ops_colqwen3="OpsColQwen3Config",
    qwen3_vl_nemotron_embed="Qwen3VLNemotronEmbedConfig",
    deepseek_vl_v2="DeepseekVLV2Config",
    deepseek_v32="DeepseekV3Config",
    deepseek_v4="DeepseekV4Config",
    flex_olmo="FlexOlmoConfig",
    fireredlid="FireRedLIDConfig",
    funaudiochat="FunAudioChatConfig",
    granite4_vision="Granite4VisionConfig",
    hyperclovax_vlm="HCXVisionConfig",
    hunyuan_vl="HunYuanVLConfig",
    hy_v3="HYV3Config",
    isaac="IsaacConfig",
    kimi_k2="DeepseekV3Config",  # Kimi K2 uses same architecture as DeepSeek V3
    kimi_linear="KimiLinearConfig",
    kimi_vl="KimiVLConfig",
    kimi_k25="KimiK25Config",
    RefinedWeb="RWConfig",  # For tiiuae/falcon-40b(-instruct)
    RefinedWebModel="RWConfig",  # For tiiuae/falcon-7b(-instruct)
    jais="JAISConfig",
    mlp_speculator="MLPSpeculatorConfig",
    medusa="MedusaConfig",
    midashenglm="MiDashengLMConfig",
    moondream3="Moondream3Config",
    eagle="EAGLEConfig",
    speculators="SpeculatorsConfig",
    nemotron="NemotronConfig",
    olmo_hybrid="OlmoHybridConfig",
    ovis="OvisConfig",
    ultravox="UltravoxConfig",
    step3_vl="Step3VLConfig",
    step3_text="Step3TextConfig",
# ... omitted for brevity ...

_SPECULATIVE_DECODING_CONFIGS: set[str] = {"eagle", "speculators"}

_CONFIG_ATTRS_MAPPING: dict[str, str] = {
    "llm_config": "text_config",
}

_AUTO_CONFIG_KWARGS_OVERRIDES: dict[str, dict[str, Any]] = {
    "internvl_chat": {"has_no_defaults_at_init": True},
    "Llama_Nemotron_Nano_VL": {"attn_implementation": "eager"},
    "NVLM_D": {"has_no_defaults_at_init": True},
}
```
**EN:** This block defines module-level registries and lookup tables such as `_CONFIG_REGISTRY`, `_SPECULATIVE_DECODING_CONFIGS`, `_CONFIG_ATTRS_MAPPING`, `_AUTO_CONFIG_KWARGS_OVERRIDES`. These values drive later dispatch and compatibility decisions.
**CN:** 该代码块定义了模块级注册表与查找表，例如 `_CONFIG_REGISTRY`, `_SPECULATIVE_DECODING_CONFIGS`, `_CONFIG_ATTRS_MAPPING`, `_AUTO_CONFIG_KWARGS_OVERRIDES`。这些值会驱动后续的分发与兼容性判断。

### Lines 172-265: Class `HFConfigParser`
```python
class HFConfigParser(ConfigParserBase):
    def parse(
        self,
        model: str | Path,
        trust_remote_code: bool,
        revision: str | None = None,
        code_revision: str | None = None,
        **kwargs,
    ) -> tuple[dict, PretrainedConfig]:
        kwargs["local_files_only"] = huggingface_hub.constants.HF_HUB_OFFLINE
        trust_remote_code |= kwargs.get("trust_remote_code", False)
        kwargs = without_trust_remote_code(kwargs)
        config_dict, _ = PretrainedConfig.get_config_dict(
            model,
            revision=revision,
            code_revision=code_revision,
            **kwargs,
        )
        # Use custom model class if it's in our registry
        model_type = config_dict.get("model_type")
        if model_type is None:
            model_type = (
                "speculators"
                if config_dict.get("speculators_config") is not None
                else model_type
            )
        # Allow hf_overrides to override model_type before checking _CONFIG_REGISTRY
        if (hf_overrides := kwargs.pop("hf_overrides", None)) is not None:
            if isinstance(hf_overrides, dict) and "model_type" in hf_overrides:
                model_type = hf_overrides["model_type"]
            elif callable(hf_overrides):
                # If hf_overrides doesn't modify model_type, it will be passed straight
                # through and remain unchanged by this elif block
                dummy_model_type = f"dummy_{model_type}"
                dummy_kwargs = dict(architectures=[""], model_type=dummy_model_type)
                dummy_config = PretrainedConfig(**dummy_kwargs)
                dummy_model_type = hf_overrides(dummy_config).model_type
                model_type = dummy_model_type.removeprefix("dummy_")

        if model_type in _SPECULATIVE_DECODING_CONFIGS:
# ... omitted for brevity ...
                    err_msg = (
                        "Failed to load the model config. If the model "
                        "is a custom model not yet available in the "
                        "HuggingFace transformers library, consider setting "
                        "`trust_remote_code=True` in LLM or using the "
                        "`--trust-remote-code` flag in the CLI."
                    )
                    raise RuntimeError(err_msg) from e
                else:
                    raise e
        config = _maybe_remap_hf_config_attrs(config)
        return config_dict, config
```
**EN:** Defines `HFConfigParser`, a model-configuration class derived from `ConfigParserBase`. Key methods include `parse`.
**CN:** 定义 `HFConfigParser`，这是一个模型配置类，继承自 `ConfigParserBase`。 关键方法包括 `parse`。

### Lines 268-322: Class `MistralConfigParser`
```python
class MistralConfigParser(ConfigParserBase):
    def parse(
        self,
        model: str | Path,
        trust_remote_code: bool,
        revision: str | None = None,
        code_revision: str | None = None,
        **kwargs,
    ) -> tuple[dict, PretrainedConfig]:
        # This function loads a params.json config which
        # should be used when loading models in mistral format
        config_dict = _download_mistral_config_file(model, revision)
        if (
            max_position_embeddings := config_dict.get("max_position_embeddings")
        ) is None:
            max_position_embeddings = _maybe_retrieve_max_pos_from_hf(
                model, revision, **kwargs
            )
            config_dict["max_position_embeddings"] = max_position_embeddings

        from vllm.transformers_utils.configs.mistral import adapt_config_dict

        # Get missing fields from HF config if available
        try:
            hf_config_dict, _ = PretrainedConfig.get_config_dict(
                model,
                revision=revision,
                code_revision=code_revision,
                **without_trust_remote_code(kwargs),
            )
        except OSError:  # Not found
            hf_config_dict = {}

        if config_dict.get("dtype") is None:
            with _mistral_patch_hf_hub_constants():
                model_str = model if isinstance(model, str) else model.as_posix()
                param_mt = get_safetensors_params_metadata(model_str, revision=revision)
            if param_mt:
                param_dtypes: set[torch.dtype] = {
                    _SAFETENSORS_TO_TORCH_DTYPE[dtype]
                    for info in param_mt.values()
                    if (dtype := info.get("dtype", None))
                    and dtype in _SAFETENSORS_TO_TORCH_DTYPE
                }

                if param_dtypes:
                    config_dict["dtype"] = common_broadcastable_dtype(param_dtypes)
                    logger.info_once(
                        "Inferred from consolidated*.safetensors files "
                        f"{config_dict['dtype']} dtype."
                    )

        config = adapt_config_dict(config_dict, defaults=hf_config_dict)

        return config_dict, config
```
**EN:** Defines `MistralConfigParser`, a model-configuration class derived from `ConfigParserBase`. Key methods include `parse`.
**CN:** 定义 `MistralConfigParser`，这是一个模型配置类，继承自 `ConfigParserBase`。 关键方法包括 `parse`。

### Lines 325-334: Module state and constants
```python
_CONFIG_FORMAT_TO_CONFIG_PARSER: dict[str, type[ConfigParserBase]] = {
    "hf": HFConfigParser,
    "mistral": MistralConfigParser,
}

ConfigFormat = Literal[
    "auto",
    "hf",
    "mistral",
]
```
**EN:** This block defines module-level constants/defaults such as `_CONFIG_FORMAT_TO_CONFIG_PARSER`, `ConfigFormat`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `_CONFIG_FORMAT_TO_CONFIG_PARSER`, `ConfigFormat`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 344-392: Function `register_config_parser`
```python
def register_config_parser(config_format: str):
    """Register a customized vllm config parser.
     When a config format is not supported by vllm, you can register a customized
    config parser to support it.
     Args:
         config_format (str): The config parser format name.
     Examples:

         >>> from vllm.transformers_utils.config import (get_config_parser,
                                                         register_config_parser)
         >>> from vllm.transformers_utils.config_parser_base import ConfigParserBase
         >>>
         >>> @register_config_parser("custom_config_parser")
         ... class CustomConfigParser(ConfigParserBase):
         ...     def parse(
         ...         self,
         ...         model: Union[str, Path],
         ...         trust_remote_code: bool,
         ...         revision: str | None = None,
         ...         code_revision: str | None = None,
         ...         **kwargs,
         ...     ) -> tuple[dict, PretrainedConfig]:
         ...         raise NotImplementedError
         >>>
         >>> type(get_config_parser("custom_config_parser"))
         <class 'CustomConfigParser'>
    """  # noqa: E501

    def _wrapper(config_parser_cls):
        if config_format in _CONFIG_FORMAT_TO_CONFIG_PARSER:
            logger.warning(
                "Config format `%s` is already registered, and will be "
                "overwritten by the new parser class `%s`.",
                config_format,
                config_parser_cls,
            )
        if not issubclass(config_parser_cls, ConfigParserBase):
            raise ValueError(
                "The config parser must be a subclass of `ConfigParserBase`."
            )
        _CONFIG_FORMAT_TO_CONFIG_PARSER[config_format] = config_parser_cls
        logger.info(
            "Registered config parser `%s` with config format `%s`",
            config_parser_cls,
            config_format,
        )
        return config_parser_cls

    return _wrapper
```
**EN:** This function registers config parser. The docstring states that Register a customized vllm config parser. Main inputs include `config_format`.
**CN:** 该函数负责完成 `register_config_parser` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config_format`。

### Lines 404-452: Function `patch_legacy_rope_type`
```python
def patch_legacy_rope_type(rope_parameters: dict[str, Any] | None) -> None:
    """Patch legacy RoPE type fields for backwards compatibility with
    older custom models which would otherwise fail to load."""

    # No RoPE parameters to patch
    if rope_parameters is None:
        return

    def _patch_legacy_rope_type(rope_parameters: dict[str, Any]) -> None:
        # Case 1: Both legacy and modern fields present - check for conflicts
        if "rope_type" in rope_parameters and "type" in rope_parameters:
            rope_type = rope_parameters["rope_type"]
            rope_type_legacy = rope_parameters["type"]
            if (rope_type_legacy == "su" and rope_type == "longrope") or (
                rope_type_legacy == "mrope" and rope_type == "default"
            ):
                pass  # No action needed
            elif rope_type != rope_type_legacy:
                raise ValueError(
                    f"Found conflicts between 'rope_type={rope_type}' (modern "
                    f"field) and 'type={rope_type_legacy}' (legacy field). "
                    "You should only specify one of them."
                )
        # Case 2: Only legacy field present - patch to modern format with warning
        if "rope_type" not in rope_parameters and "type" in rope_parameters:
            rope_parameters["rope_type"] = rope_parameters["type"]
            logger.info("Replacing legacy 'type' key with 'rope_type'")
        # Case 3: No rope_type field at all - cannot determine RoPE type, raise error
        if "rope_type" not in rope_parameters:
            raise ValueError("rope_parameters should have a 'rope_type' key")
        # Patch legacy rope_type values with warning
        if rope_parameters["rope_type"] == "su":
            rope_parameters["rope_type"] = "longrope"
            logger.warning("Replacing legacy rope_type 'su' with 'longrope'")
        elif rope_parameters["rope_type"] == "mrope":
            if "mrope_section" not in rope_parameters:
                raise ValueError(
                    "Legacy rope_type 'mrope' requires "
                    "'mrope_section' in rope_parameters"
                )
            rope_parameters["rope_type"] = "default"
            logger.warning("Replacing legacy rope_type 'mrope' with 'default'")

    # Handle nested rope_parameters in interleaved sliding attention models
    if is_rope_parameters_nested(rope_parameters):
        for rope_parameters_layer_type in rope_parameters.values():
            _patch_legacy_rope_type(rope_parameters_layer_type)
    else:
        _patch_legacy_rope_type(rope_parameters)
```
**EN:** This function patches legacy rope type. The docstring states that Patch legacy RoPE type fields for backwards compatibility with Main inputs include `rope_parameters`.
**CN:** 该函数负责完成 `patch_legacy_rope_type` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `rope_parameters`。

### Lines 455-500: Function `patch_rope_parameters`
```python
def patch_rope_parameters(config: PretrainedConfig) -> None:
    """Provide backwards compatibility for RoPE."""
    from vllm.config.utils import getattr_iter

    # Older custom models may use non-standard field names
    # which need patching for both Transformers v4 and v5.
    names = ["rope_theta", "rotary_emb_base"]
    rope_theta = getattr_iter(config, names, None, warn=True)
    names = ["partial_rotary_factor", "rotary_pct", "rotary_emb_fraction"]
    partial_rotary_factor = getattr_iter(config, names, None, warn=True)
    ompe = getattr(config, "original_max_position_embeddings", None)

    if Version(version("transformers")) < Version("5.0.0"):
        # Transformers v4 installed, legacy config fields may be present.
        if is_rope_parameters_nested(getattr(config, "rope_parameters", {})):
            # Loading nested rope_parameters (from Transformers v5) in Transformers v4.
            # Skip legacy patching since it should already be in the correct format.
            pass
        else:
            if (rope_scaling := getattr(config, "rope_scaling", None)) is not None:
                config.rope_parameters = rope_scaling
            if (
                rope_theta is not None
                or partial_rotary_factor is not None
                or ompe is not None
            ) and not getattr(config, "rope_parameters", None):
                config.rope_parameters = {"rope_type": "default"}
            # Patch legacy fields into rope_parameters
            if rope_theta is not None:
                config.rope_parameters["rope_theta"] = rope_theta
            if partial_rotary_factor is not None:
                config.rope_parameters["partial_rotary_factor"] = partial_rotary_factor
            if ompe is not None:
                config.rope_parameters["original_max_position_embeddings"] = ompe
            patch_legacy_rope_type(getattr(config, "rope_parameters", None))
    elif rope_theta is not None or getattr(config, "rope_parameters", None):
        # Transformers v5 installed
        # Patch these fields in case they used non-standard names
        if rope_theta is not None:
            config.rope_theta = rope_theta
        if partial_rotary_factor is not None:
            config.partial_rotary_factor = partial_rotary_factor
        # Standardize and validate RoPE parameters
        patch_legacy_rope_type(getattr(config, "rope_parameters", None))
        config.standardize_rope_params()
        config.validate_rope()
```
**EN:** This function patches rope parameters. The docstring states that Provide backwards compatibility for RoPE. Main inputs include `config`.
**CN:** 该函数负责完成 `patch_rope_parameters` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config`。

### Lines 520-530: Function `thinker_uses_mrope`
```python
def thinker_uses_mrope(config: PretrainedConfig) -> bool:
    """Detect if the model contains a thinker config and it uses M-ROPE."""
    thinker_config = getattr(config, "thinker_config", None)
    if thinker_config is None:
        return False

    thinker_text_config = getattr(thinker_config, "text_config", None)
    if thinker_text_config is None:
        return False

    return uses_mrope(thinker_text_config)
```
**EN:** This function implements `thinker_uses_mrope`. The docstring states that Detect if the model contains a thinker config and it uses M-ROPE. Main inputs include `config`.
**CN:** 该函数实现 `thinker_uses_mrope` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config`。

### Lines 533-547: Function `uses_xdrope_dim`
```python
def uses_xdrope_dim(config: PretrainedConfig) -> int:
    """Detect if the model with this config uses XD-ROPE."""
    xdrope_section = getattr(config, "xdrope_section", None)
    if xdrope_section is not None and isinstance(xdrope_section, list):
        return len(xdrope_section)
    rope_scaling = getattr(config, "rope_scaling", None)
    if rope_scaling is None:
        return 0

    if isinstance(rope_scaling, dict) and "xdrope_section" in rope_scaling:
        xdrope_section = rope_scaling["xdrope_section"]
        if xdrope_section is not None and isinstance(xdrope_section, list):
            return len(xdrope_section)

    return 0
```
**EN:** This function implements `uses_xdrope_dim`. The docstring states that Detect if the model with this config uses XD-ROPE. Main inputs include `config`.
**CN:** 该函数实现 `uses_xdrope_dim` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config`。

### Lines 588-649: Function `maybe_override_with_speculators`
```python
def maybe_override_with_speculators(
    model: str,
    tokenizer: str | None,
    trust_remote_code: bool,
    revision: str | None = None,
    vllm_speculative_config: dict[str, Any] | None = None,
    hf_token: bool | str | None = None,
    **kwargs,
) -> tuple[str, str | None, dict[str, Any] | None]:
    """
    Resolve model configuration when speculators are detected.

    Checks if the provided model is a speculators model and if so, extracts
    the target model configuration and builds the speculative config.

    Args:
        model: Model name or path
        tokenizer: Tokenizer name or path
        trust_remote_code: Whether to trust remote code
        revision: Model revision
        vllm_speculative_config: Existing vLLM speculative config
        hf_token: HuggingFace token for authenticated model access

    Returns:
        Tuple of (resolved_model, resolved_tokenizer, speculative_config)
    """
    if check_gguf_file(model):
        kwargs["gguf_file"] = Path(model).name
        gguf_model_repo = Path(model).parent
    elif is_remote_gguf(model):
        repo_id, _ = split_remote_gguf(model)
        gguf_model_repo = Path(repo_id)
    else:
        gguf_model_repo = None
    kwargs["local_files_only"] = huggingface_hub.constants.HF_HUB_OFFLINE
    config_dict, _ = PretrainedConfig.get_config_dict(
        model if gguf_model_repo is None else gguf_model_repo,
        revision=revision,
        token=hf_token,
        **without_trust_remote_code(kwargs),
# ... omitted for brevity ...
    speculative_config = SpeculatorsConfig.extract_vllm_speculative_config(
        config_dict=config_dict
    )

    # Set the draft model to the speculators model
    speculative_config["model"] = model

    # Override model and tokenizer with the verifier model from config
    verifier_model = speculators_config["verifier"]["name_or_path"]
    model = tokenizer = verifier_model

    return model, tokenizer, speculative_config
```
**EN:** This function implements `maybe_override_with_speculators`. The docstring states that Resolve model configuration when speculators are detected. Main inputs include `model`, `tokenizer`, `trust_remote_code`, `revision`, `vllm_speculative_config`, ... (+1 more).
**CN:** 该函数实现 `maybe_override_with_speculators` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model`, `tokenizer`, `trust_remote_code`, `revision`, `vllm_speculative_config`, ... (+1 more)。

### Lines 652-837: Function `get_config`
```python
def get_config(
    model: str | Path,
    trust_remote_code: bool,
    revision: str | None = None,
    code_revision: str | None = None,
    config_format: str | ConfigFormat = "auto",
    hf_overrides_kw: dict[str, Any] | None = None,
    hf_overrides_fn: Callable[[PretrainedConfig], PretrainedConfig] | None = None,
    **kwargs,
) -> PretrainedConfig:
    # Separate model folder from file path for GGUF models

    _is_gguf = is_gguf(model)
    _is_remote_gguf = is_remote_gguf(model)
    if _is_gguf:
        if check_gguf_file(model):
            # Local GGUF file
            kwargs["gguf_file"] = Path(model).name
            model = Path(model).parent
        elif _is_remote_gguf:
            # Remote GGUF - extract repo_id from repo_id:quant_type format
            # The actual GGUF file will be downloaded later by GGUFModelLoader
            # Keep model as repo_id:quant_type for download, but use repo_id for config
            model, _ = split_remote_gguf(model)

    if config_format == "auto":
        try:
            # First check for Mistral to avoid defaulting to
            # Transformers implementation.
            if is_mistral_model_repo(
                model_name_or_path=str(model), revision=revision
            ) and file_or_path_exists(
                model=model, config_name=MISTRAL_CONFIG_NAME, revision=revision
            ):
                config_format = "mistral"
            elif (_is_gguf and not _is_remote_gguf) or file_or_path_exists(
                model, HF_CONFIG_NAME, revision=revision
            ):
                config_format = "hf"
            # Remote GGUF models must have config.json in repo,
# ... omitted for brevity ...
    patch_rope_parameters(config)
    patch_rope_parameters(config.get_text_config())
    SubConfigs: TypeAlias = dict[str, PretrainedConfig]
    sub_configs: SubConfigs | None = getattr(config, "sub_configs", None)
    if sub_configs:
        for sub_config in sub_configs:
            patch_rope_parameters(getattr(config, sub_config))

    if trust_remote_code:
        maybe_register_config_serialize_by_value()

    return config
```
**EN:** This function retrieves config. Main inputs include `model`, `trust_remote_code`, `revision`, `code_revision`, `config_format`, ... (+2 more).
**CN:** 该函数负责完成 `get_config` 对应的核心步骤。 主要输入参数包括 `model`, `trust_remote_code`, `revision`, `code_revision`, `config_format`, ... (+2 more)。

### Lines 841-915: Function `get_pooling_config`
```python
def get_pooling_config(
    model: str,
    revision: str | None = "main",
) -> dict[str, Any] | None:
    """
    This function gets the pooling and normalize
    config from the model - only applies to
    sentence-transformers models.

    Args:
        model: The name of the Hugging Face model.
        revision: The specific version of the model to use.
            Defaults to 'main'.

    Returns:
        A dictionary containing the pooling type and whether
            normalization is used, or None if no pooling configuration is found.
    """
    if is_remote_gguf(model):
        model, _ = split_remote_gguf(model)

    modules_file_name = "modules.json"

    modules_dict = None
    if file_or_path_exists(
        model=model, config_name=modules_file_name, revision=revision
    ):
        modules_dict = get_hf_file_to_dict(modules_file_name, model, revision)

    if modules_dict is None:
        return None

    logger.info("Found sentence-transformers modules configuration.")

    pooling = next(
        (
            item
            for item in modules_dict
            if item["type"] == "sentence_transformers.models.Pooling"
        ),
# ... omitted for brevity ...
            if val is True:
                pooling_type = parse_pooling_type(key)
                if pooling_type in SEQ_POOLING_TYPES:
                    config["seq_pooling_type"] = pooling_type
                elif pooling_type in TOK_POOLING_TYPES:
                    config["tok_pooling_type"] = pooling_type
                else:
                    logger.debug("Skipping unrelated field: %r=%r", key, val)

        return config

    return None
```
**EN:** This function retrieves pooling config. The docstring states that This function gets the pooling and normalize Main inputs include `model`, `revision`. Decorators such as `cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `get_pooling_config` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model`, `revision`。 装饰器如 `cache` 会影响缓存、校验或分发行为。

### Lines 918-928: Function `parse_pooling_type`
```python
def parse_pooling_type(pooling_name: str):
    if "pooling_mode_" in pooling_name:
        pooling_name = pooling_name.replace("pooling_mode_", "")

    if "_" in pooling_name:
        pooling_name = pooling_name.split("_", 1)[0]

    if "lasttoken" in pooling_name:
        pooling_name = "last"

    return pooling_name.upper()
```
**EN:** This function parses pooling type. Main inputs include `pooling_name`.
**CN:** 该函数负责完成 `parse_pooling_type` 对应的核心步骤。 主要输入参数包括 `pooling_name`。

### Lines 932-989: Function `get_sentence_transformer_tokenizer_config`
```python
def get_sentence_transformer_tokenizer_config(
    model: str | Path, revision: str | None = "main"
) -> dict[str, Any] | None:
    """
    Returns the tokenization configuration dictionary for a
    given Sentence Transformer BERT model.

    Parameters:
    - model (str|Path): The name of the Sentence Transformer
    BERT model.
    - revision (str, optional): The revision of the m
    odel to use. Defaults to 'main'.

    Returns:
    - dict: A dictionary containing the configuration parameters
    for the Sentence Transformer BERT model.
    """
    sentence_transformer_config_files = [
        "sentence_bert_config.json",
        "sentence_roberta_config.json",
        "sentence_distilbert_config.json",
        "sentence_camembert_config.json",
        "sentence_albert_config.json",
        "sentence_xlm-roberta_config.json",
        "sentence_xlnet_config.json",
    ]
    encoder_dict = None

    for config_file in sentence_transformer_config_files:
        if (
            try_get_local_file(model=model, file_name=config_file, revision=revision)
            is not None
        ):
            encoder_dict = get_hf_file_to_dict(config_file, model, revision)
            if encoder_dict:
                break

    if not encoder_dict and not Path(model).is_absolute():
        try:
            # If model is on HuggingfaceHub, get the repo files
            repo_files = list_repo_files(model, revision=revision)
        except Exception:
            repo_files = []

        for config_name in sentence_transformer_config_files:
            if config_name in repo_files:
                encoder_dict = get_hf_file_to_dict(config_name, model, revision)
                if encoder_dict:
                    break

    if not encoder_dict:
        return None

    logger.info("Found sentence-transformers tokenize configuration.")

    if all(k in encoder_dict for k in ("max_seq_length", "do_lower_case")):
        return encoder_dict
    return None
```
**EN:** This function retrieves sentence transformer tokenizer config. The docstring states that Returns the tokenization configuration dictionary for a Main inputs include `model`, `revision`. Decorators such as `cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `get_sentence_transformer_tokenizer_config` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model`, `revision`。 装饰器如 `cache` 会影响缓存、校验或分发行为。

### Lines 992-1065: Function `maybe_register_config_serialize_by_value`
```python
def maybe_register_config_serialize_by_value() -> None:
    """Try to register HF model configuration class to serialize by value

    If trust_remote_code is set, and the model's config file specifies an
    `AutoConfig` class, then the config class is typically an instance of
    a custom class imported from the HF modules cache.

    Examples:

    >>> from transformers import AutoConfig
    >>> klass = AutoConfig.from_pretrained(
    ...     "meta-llama/Meta-Llama-3-8B", trust_remote_code=True
    ... )
    >>> klass.__class__  # transformers.models.llama.configuration_llama.LlamaConfig
    >>> import transformers_modules  # error, not initialized
    >>> klass = AutoConfig.from_pretrained(
    ...     "deepseek-ai/DeepSeek-V2.5", trust_remote_code=True
    ... )
    >>> import transformers_modules  # success, initialized
    >>> klass.__class__  # transformers_modules.deepseek-ai.DeepSeek-V2.5.98b11844770b2c3ffc18b175c758a803640f4e77.configuration_deepseek.DeepseekV2Config

    In the DeepSeek example, the config class is an instance of a custom
    class that is not serializable by default. This class will not be
    importable in spawned workers, and won't exist at all on
    other nodes, which breaks serialization of the config.

    In this function we tell the cloudpickle serialization library to pass
    instances of these generated classes by value instead of by reference,
    i.e. the class definition is serialized along with its data so that the
    class module does not need to be importable on the receiving end.

    See: https://github.com/cloudpipe/cloudpickle?tab=readme-ov-file#overriding-pickles-serialization-mechanism-for-importable-constructs
    """  # noqa
    try:
        import transformers_modules

        transformers_modules_available = True
    except ImportError:
        transformers_modules_available = False

# ... omitted for brevity ...

            if ray:
                ray.cloudpickle.register_pickle_by_value(transformers_modules)

    except Exception as e:
        logger.warning(
            "Unable to register remote classes used by"
            " trust_remote_code with by-value serialization. This may"
            " lead to a later error. If remote code is not needed"
            " remove `--trust-remote-code`",
            exc_info=e,
        )
```
**EN:** This function implements `maybe_register_config_serialize_by_value`. The docstring states that Try to register HF model configuration class to serialize by value
**CN:** 该函数实现 `maybe_register_config_serialize_by_value` 相关逻辑。 文档字符串进一步说明了它的输入与行为。

### Lines 1068-1084: Function `get_hf_image_processor_config`
```python
def get_hf_image_processor_config(
    model: str | Path,
    hf_token: bool | str | None = None,
    revision: str | None = None,
    **kwargs,
) -> dict[str, Any]:
    # ModelScope does not provide an interface for image_processor
    if envs.VLLM_USE_MODELSCOPE:
        return dict()
    # Separate model folder from file path for GGUF models
    if check_gguf_file(model):
        model = Path(model).parent
    elif is_remote_gguf(model):
        model, _ = split_remote_gguf(model)
    return get_image_processor_config(
        model, token=hf_token, revision=revision, **kwargs
    )
```
**EN:** This function retrieves hf image processor config. Main inputs include `model`, `hf_token`, `revision`.
**CN:** 该函数负责完成 `get_hf_image_processor_config` 对应的核心步骤。 主要输入参数包括 `model`, `hf_token`, `revision`。

### Lines 1087-1101: Function `get_hf_text_config`
```python
def get_hf_text_config(config: PretrainedConfig):
    """Get the "sub" config relevant to llm for multi modal models.
    No op for pure text models.
    """
    text_config = config.get_text_config()

    if text_config is not config and not hasattr(text_config, "num_attention_heads"):
        raise ValueError(
            "The text_config extracted from the model config does not have "
            "`num_attention_heads` attribute. This indicates a mismatch "
            "between the model config and vLLM's expectations. Please "
            "ensure that the model config is compatible with vLLM."
        )

    return text_config
```
**EN:** This function retrieves hf text config. The docstring states that Get the "sub" config relevant to llm for multi modal models. Main inputs include `config`.
**CN:** 该函数负责完成 `get_hf_text_config` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config`。

### Lines 1104-1135: Function `try_get_generation_config`
```python
def try_get_generation_config(
    model: str,
    trust_remote_code: bool,
    revision: str | None = None,
    config_format: str | ConfigFormat = "auto",
    hf_token: bool | str | None = None,
) -> GenerationConfig | None:
    # GGUF files don't have generation_config.json - their config is embedded
    # in the file header. Skip all filesystem lookups to avoid re-reading the
    # memory-mapped file, which can hang in multi-process scenarios when the
    # EngineCore process already has the file mapped.
    if is_gguf(model):
        return None

    try:
        return GenerationConfig.from_pretrained(
            model,
            revision=revision,
            token=hf_token,
        )
    except OSError:  # Not found
        try:
            config = get_config(
                model,
                trust_remote_code=trust_remote_code,
                revision=revision,
                config_format=config_format,
                token=hf_token,
            )
            return GenerationConfig.from_model_config(config)
        except OSError:  # Not found
            return None
```
**EN:** This function implements `try_get_generation_config`. Main inputs include `model`, `trust_remote_code`, `revision`, `config_format`, `hf_token`.
**CN:** 该函数实现 `try_get_generation_config` 相关逻辑。 主要输入参数包括 `model`, `trust_remote_code`, `revision`, `config_format`, `hf_token`。

### Lines 1138-1152: Function `try_get_safetensors_metadata`
```python
def try_get_safetensors_metadata(
    model: str,
    *,
    revision: str | None = None,
):
    get_safetensors_metadata_partial = partial(
        get_safetensors_metadata, model, revision=revision
    )

    try:
        return with_retry(
            get_safetensors_metadata_partial, "Error retrieving safetensors"
        )
    except Exception:
        return None
```
**EN:** This function implements `try_get_safetensors_metadata`. Main inputs include `model`, `revision`.
**CN:** 该函数实现 `try_get_safetensors_metadata` 相关逻辑。 主要输入参数包括 `model`, `revision`。

### Lines 1155-1167: Function `try_get_tokenizer_config`
```python
def try_get_tokenizer_config(
    pretrained_model_name_or_path: str | os.PathLike,
    trust_remote_code: bool,
    revision: str | None = None,
) -> dict[str, Any] | None:
    try:
        return get_tokenizer_config(
            pretrained_model_name_or_path,
            trust_remote_code=trust_remote_code,
            revision=revision,
        )
    except Exception:
        return None
```
**EN:** This function implements `try_get_tokenizer_config`. Main inputs include `pretrained_model_name_or_path`, `trust_remote_code`, `revision`.
**CN:** 该函数实现 `try_get_tokenizer_config` 相关逻辑。 主要输入参数包括 `pretrained_model_name_or_path`, `trust_remote_code`, `revision`。

### Lines 1171-1203: Function `try_get_dense_modules`
```python
def try_get_dense_modules(
    model: str | Path,
    revision: str | None = None,
) -> list[dict[str, Any]] | None:
    try:
        modules = get_hf_file_to_dict("modules.json", model, revision)
        if not modules:
            return None

        if isinstance(modules, dict):
            modules = modules.get("modules", [])

        _DENSE_MODULE_TYPES = {
            "sentence_transformers.models.Dense",
            "pylate.models.Dense.Dense",
        }
        dense_modules = [m for m in modules if m.get("type") in _DENSE_MODULE_TYPES]
        if not dense_modules:
            return None

        layer_configs = []
        for module in dense_modules:
            folder = module.get("path", "")

            config_path = f"{folder}/config.json" if folder else "config.json"
            layer_config = get_hf_file_to_dict(config_path, model, revision)
            if not layer_config:
                continue
            layer_config["folder"] = folder
            layer_configs.append(layer_config)
        return layer_configs
    except Exception:
        return None
```
**EN:** This function implements `try_get_dense_modules`. Main inputs include `model`, `revision`. Decorators such as `cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数实现 `try_get_dense_modules` 相关逻辑。 主要输入参数包括 `model`, `revision`。 装饰器如 `cache` 会影响缓存、校验或分发行为。

### Lines 1206-1231: Function `get_safetensors_params_metadata`
```python
def get_safetensors_params_metadata(
    model: str,
    *,
    revision: str | None = None,
) -> dict[str, Any]:
    """
    Get the safetensors parameters metadata for remote/local model repository.
    """
    full_metadata = {}
    if (model_path := Path(model)).exists():
        safetensors_to_check = model_path.glob("*.safetensors")
        full_metadata = {
            param_name: info
            for file_path in safetensors_to_check
            if file_path.is_file()
            for param_name, info in parse_safetensors_file_metadata(file_path).items()
        }
    else:
        repo_mt = try_get_safetensors_metadata(model, revision=revision)
        if repo_mt and (files_mt := repo_mt.files_metadata):
            full_metadata = {
                param_name: asdict(info)
                for file_mt in files_mt.values()
                for param_name, info in file_mt.tensors.items()
            }
    return full_metadata
```
**EN:** This function retrieves safetensors params metadata. The docstring states that Get the safetensors parameters metadata for remote/local model repository. Main inputs include `model`, `revision`.
**CN:** 该函数负责完成 `get_safetensors_params_metadata` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model`, `revision`。

## Key Concepts / 关键概念
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `os`, `collections.abc`, `contextlib`, `dataclasses`, `functools`, `importlib.metadata`, `pathlib`, `typing`, `multiprocessing`, `pickle`.
- **CN:** 标准库模块：`os`, `collections.abc`, `contextlib`, `dataclasses`, `functools`, `importlib.metadata`, `pathlib`, `typing`, `multiprocessing`, `pickle`。
- **EN:** External packages: `huggingface_hub`, `torch`, `packaging.version`, `safetensors.torch`, `transformers`, `transformers.models.auto.image_processing_auto`, `transformers.models.auto.modeling_auto`, `transformers.models.auto.tokenization_auto`, `transformers.utils`, `transformers.configuration_utils`, ... (+3 more).
- **CN:** 外部依赖包：`huggingface_hub`, `torch`, `packaging.version`, `safetensors.torch`, `transformers`, `transformers.models.auto.image_processing_auto`, `transformers.models.auto.modeling_auto`, `transformers.models.auto.tokenization_auto`, `transformers.utils`, `transformers.configuration_utils`, ... (+3 more)。
- **EN:** Internal modules: `vllm`, `vllm.logger`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.torch_utils`, `.config_parser_base`, `.gguf_utils`, `.repo_utils`, `vllm.config.utils`, `vllm.transformers_utils.configs.speculators.base`, ... (+5 more).
- **CN:** 内部模块：`vllm`, `vllm.logger`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.torch_utils`, `.config_parser_base`, `.gguf_utils`, `.repo_utils`, `vllm.config.utils`, `vllm.transformers_utils.configs.speculators.base`, ... (+5 more)。
