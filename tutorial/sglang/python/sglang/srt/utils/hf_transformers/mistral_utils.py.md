# mistral_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/hf_transformers/mistral_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `mistral_utils` and the surrounding SGLang serving stack. / 提供围绕 `mistral_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 4-14: Module setup and shared state / 模块设置与共享状态
```python
import json
import tempfile
from functools import lru_cache
from pathlib import Path
from typing import Any, Optional

from transformers import AutoConfig, PretrainedConfig, WhisperConfig

from sglang.srt.utils import logger

from .common import _ensure_sub_configs, download_from_hf
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `json`, `tempfile`, `functools`, `pathlib`, `typing`, `transformers`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `json`, `tempfile`, `functools`, `pathlib`, `typing`, `transformers`。

### Lines 17-137: Function `adapt_config_dict` / 函数 `adapt_config_dict`
```python
def adapt_config_dict(
    config_dict: dict[str, Any], model: str, **kwargs
) -> tuple[dict, PretrainedConfig]:
    config_dict.update(kwargs)
    config_dict = _remap_general_mistral_args(config_dict)

    if bool(config_dict.get("quantization")):
        config_dict = _remap_mistral_quantization_args(config_dict)

    is_moe = bool(config_dict.get("moe"))
    is_mistral_large_3 = (
        is_moe and (config_dict["moe"].get("num_shared_experts") or 0) > 0
    )
    is_eagle = "eagle" in model.lower()
    is_mla_eagle = is_eagle and any(
        config_dict.get(k) is not None
        for k in ("kv_lora_rank", "q_lora_rank", "v_head_dim")
    )
    if is_eagle and not is_moe and is_mla_eagle:
        # Dense MLA EAGLE draft model (e.g. Mistral Small 4 EAGLE).
        # Uses MLA attention like MistralLarge3 but has no MoE layers.
        # Set model_type to deepseek_v3 for MLA support, and override
        # MoE fields so all layers are dense.
        config_dict["model_type"] = "deepseek_v3"
        config_dict["architectures"] = ["MistralLarge3ForCausalLMEagle"]
        num_layers = config_dict.get("num_hidden_layers", 0)
        config_dict["n_routed_experts"] = 1
        config_dict["first_k_dense_replace"] = num_layers
# ... omitted for brevity ...

    config = PretrainedConfig.from_dict(config_dict)

    logger.debug("Initialized config %s", config)

    return config_dict, config
```
**EN:** This function implements `adapt_config_dict`. It primarily calls `config_dict.update`, `_remap_general_mistral_args`, `bool`, `PretrainedConfig.from_dict`, `logger.debug`, `config_dict.get` to complete its work. State updates are written into `config_dict`, `is_moe`, `is_mistral_large_3`, `is_eagle`, `is_mla_eagle`, `is_vision`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `adapt_config_dict`。 它主要通过调用 `config_dict.update`, `_remap_general_mistral_args`, `bool`, `PretrainedConfig.from_dict`, `logger.debug`, `config_dict.get` 来完成任务。 状态更新主要写入 `config_dict`, `is_moe`, `is_mistral_large_3`, `is_eagle`, `is_mla_eagle`, `is_vision`。 实现中使用了条件分支、迭代逻辑。

### Lines 140-156: Function `_remap_mistral_vision_args` / 函数 `_remap_mistral_vision_args`
```python
def _remap_mistral_vision_args(config: dict) -> dict:
    if config.get("multimodal"):
        vision_config = config.pop("multimodal")
    else:
        vision_config = config.pop("vision_encoder")

    quant_config = config.get("quantization_config")

    config = {
        "model_type": "pixtral",
        "architectures": ["PixtralForConditionalGeneration"],
        "text_config": config,
        "vision_config": {"model_type": "pixtral", **vision_config},
    }
    if quant_config:
        config["quantization_config"] = quant_config
    return config
```
**EN:** This function implements `_remap_mistral_vision_args`. It primarily calls `config.get`, `config.pop` to complete its work. State updates are written into `quant_config`, `config`, `vision_config`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_remap_mistral_vision_args`。 它主要通过调用 `config.get`, `config.pop` 来完成任务。 状态更新主要写入 `quant_config`, `config`, `vision_config`。 实现中使用了条件分支。

### Lines 159-184: Function `_remap_mistral_yarn_args` / 函数 `_remap_mistral_yarn_args`
```python
def _remap_mistral_yarn_args(config: dict) -> dict:
    yarn_config_map = {
        "factor": "factor",
        "original_max_position_embeddings": "original_max_position_embeddings",
        "beta": "beta_fast",
        "alpha": "beta_slow",
        "apply_scale": "apply_yarn_scaling",
    }
    yarn_config = config.get("yarn") or {}
    config["rope_scaling"] = {
        "rope_type": "deepseek_yarn",
        "mscale_all_dim": 1,
    }
    # Include rope_theta in rope_scaling if present at the top level,
    # as transformers yarn validation requires it.
    if "rope_theta" in config:
        config["rope_scaling"]["rope_theta"] = config["rope_theta"]
    for old_name, new_name in yarn_config_map.items():
        if old_name in yarn_config:
            value = yarn_config.pop(old_name)
            if new_name is not None:
                config["rope_scaling"][new_name] = value

    assert len(yarn_config) == 0, f"Unparsed yarn config: {yarn_config}"

    return config
```
**EN:** This function implements `_remap_mistral_yarn_args`. It primarily calls `yarn_config_map.items`, `config.get`, `len`, `yarn_config.pop` to complete its work. State updates are written into `yarn_config_map`, `yarn_config`, `config`, `value`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_remap_mistral_yarn_args`。 它主要通过调用 `yarn_config_map.items`, `config.get`, `len`, `yarn_config.pop` 来完成任务。 状态更新主要写入 `yarn_config_map`, `yarn_config`, `config`, `value`。 实现中使用了条件分支、迭代逻辑。

### Lines 187-213: Function `_remap_general_mistral_args` / 函数 `_remap_general_mistral_args`
```python
def _remap_general_mistral_args(config: dict) -> dict:
    # Mistral key -> HF key
    config_mapping = {
        "dim": "hidden_size",
        "norm_eps": "rms_norm_eps",
        "n_kv_heads": "num_key_value_heads",
        "n_layers": "num_hidden_layers",
        "n_heads": "num_attention_heads",
        "hidden_dim": "intermediate_size",
    }
    # HF key -> (Mistral key, default value)
    top_level_mapping_with_default = {
        "model_type": ("model_type", "transformer"),
        "hidden_act": ("activation", "silu"),
        "tie_word_embeddings": ("tied_embeddings", False),
        "max_seq_len": ("max_seq_len", 128_000),
        "max_position_embeddings": ("max_position_embeddings", 128_000),
    }

    for key, new_key in config_mapping.items():
        if key in config:
            config[new_key] = config.pop(key)

    for new_key, (key, default_value) in top_level_mapping_with_default.items():
        config[new_key] = config.pop(key, default_value)

    return config
```
**EN:** This function implements `_remap_general_mistral_args`. It primarily calls `config_mapping.items`, `top_level_mapping_with_default.items`, `config.pop` to complete its work. State updates are written into `config_mapping`, `top_level_mapping_with_default`, `config`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_remap_general_mistral_args`。 它主要通过调用 `config_mapping.items`, `top_level_mapping_with_default.items`, `config.pop` 来完成任务。 状态更新主要写入 `config_mapping`, `top_level_mapping_with_default`, `config`。 实现中使用了条件分支、迭代逻辑。

### Lines 216-234: Function `_remap_mistral_quantization_args` / 函数 `_remap_mistral_quantization_args`
```python
def _remap_mistral_quantization_args(config: dict) -> dict:
    if config.get("quantization"):
        quantization = config.pop("quantization", {})
        if quantization.get("qformat_weight") == "fp8_e4m3":
            qscheme_act = quantization.get("qscheme_act")
            assert qscheme_act in (
                "NO_SCALES",
                "TENSOR",
                None,
            ), "Only NO_SCALES and TENSOR (default) are supported for qscheme_act"
            is_dynamic = qscheme_act == "NO_SCALES"
            config["quantization_config"] = {
                "quant_method": "fp8",
                "activation_scheme": "dynamic" if is_dynamic else "static",
            }
        else:
            raise ValueError(f"Found unknown quantization='{quantization}' in config")

    return config
```
**EN:** This function implements `_remap_mistral_quantization_args`. It primarily calls `config.get`, `config.pop`, `quantization.get`, `ValueError` to complete its work. State updates are written into `quantization`, `qscheme_act`, `is_dynamic`, `config`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_remap_mistral_quantization_args`。 它主要通过调用 `config.get`, `config.pop`, `quantization.get`, `ValueError` 来完成任务。 状态更新主要写入 `quantization`, `qscheme_act`, `is_dynamic`, `config`。 实现中使用了条件分支。

### Lines 237-264: Function `_remap_mistral_audio_args` / 函数 `_remap_mistral_audio_args`
```python
def _remap_mistral_audio_args(config: dict) -> dict:
    whisper_args = config["multimodal"].pop("whisper_model_args")
    encoder_args = whisper_args["encoder_args"]
    downsample_args = whisper_args["downsample_args"]

    quant_config = config.get("quantization_config")
    config = {
        "model_type": "whixtral",
        "architectures": ["VoxtralForConditionalGeneration"],
        "text_config": PretrainedConfig.from_dict(config),
        "audio_config": WhisperConfig(
            num_mel_bins=encoder_args["audio_encoding_args"]["num_mel_bins"],
            window_size=encoder_args["audio_encoding_args"]["window_size"],
            sampling_rate=encoder_args["audio_encoding_args"]["sampling_rate"],
            hop_length=encoder_args["audio_encoding_args"]["hop_length"],
            downsample_factor=downsample_args["downsample_factor"],
            d_model=encoder_args["dim"],
            encoder_layers=encoder_args["n_layers"],
            encoder_ffn_dim=encoder_args["hidden_dim"],
            encoder_attention_heads=encoder_args["n_heads"],
            vocab_size=encoder_args["vocab_size"],
            max_source_positions=encoder_args["max_source_positions"],
            is_encoder_decoder=False,  # Override WhisperConfig default
        ),
    }
    if quant_config:
        config["quantization_config"] = quant_config
    return config
```
**EN:** This function implements `_remap_mistral_audio_args`. It primarily calls `config.pop`, `config.get`, `PretrainedConfig.from_dict`, `WhisperConfig` to complete its work. State updates are written into `whisper_args`, `encoder_args`, `downsample_args`, `quant_config`, `config`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_remap_mistral_audio_args`。 它主要通过调用 `config.pop`, `config.get`, `PretrainedConfig.from_dict`, `WhisperConfig` 来完成任务。 状态更新主要写入 `whisper_args`, `encoder_args`, `downsample_args`, `quant_config`, `config`。 实现中使用了条件分支。

### Lines 267-289: Function `_remap_moe_args` / 函数 `_remap_moe_args`
```python
def _remap_moe_args(config: dict) -> dict:
    moe_config_map = {
        "route_every_n": "moe_layer_freq",
        "first_k_dense_replace": "first_k_dense_replace",
        "num_experts_per_tok": "num_experts_per_tok",
        "num_experts": "n_routed_experts",
        "expert_hidden_dim": "moe_intermediate_size",
        "routed_scale": "routed_scaling_factor",
        "num_shared_experts": "n_shared_experts",
        "num_expert_groups": "n_group",
        "num_expert_groups_per_tok": "topk_group",
    }
    moe_config = config.get("moe", {})
    for old_name, new_name in moe_config_map.items():
        if old_name in moe_config:
            value = moe_config.pop(old_name)
            config[new_name] = value

    config["topk_method"] = None
    config["scoring_func"] = "softmax"
    config["routing_method_type"] = 1  # RoutingMethodType.Renormalize

    return config
```
**EN:** This function implements `_remap_moe_args`. It primarily calls `config.get`, `moe_config_map.items`, `moe_config.pop` to complete its work. State updates are written into `moe_config_map`, `moe_config`, `config`, `value`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_remap_moe_args`。 它主要通过调用 `config.get`, `moe_config_map.items`, `moe_config.pop` 来完成任务。 状态更新主要写入 `moe_config_map`, `moe_config`, `config`, `value`。 实现中使用了条件分支、迭代逻辑。

### Lines 292-292: Class `MistralConfigParser` declaration / 类 `MistralConfigParser` 声明
```python
class MistralConfigParser:
```
**EN:** This class establishes `MistralConfigParser` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `get_hf_file_to_dict`, `_download_mistral_config_file`, `parse`.
**CN:** 该类将 `MistralConfigParser` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `get_hf_file_to_dict`, `_download_mistral_config_file`, `parse` 等方法。

### Lines 293-301: Method `MistralConfigParser.get_hf_file_to_dict` / 方法 `MistralConfigParser.get_hf_file_to_dict`
```python
    def get_hf_file_to_dict(
        self, file_name: str, model: str | Path, revision: str | None = "main"
    ):
        file_path = Path(model) / file_name
        if not file_path.is_file():
            raise FileNotFoundError(f"File not found {model}, {file_name}")

        with open(file_path) as file:
            return json.load(file)
```
**EN:** This method implements `get_hf_file_to_dict` on `MistralConfigParser`. It primarily calls `Path`, `file_path.is_file`, `FileNotFoundError`, `open`, `json.load` to complete its work. State updates are written into `file_path`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `MistralConfigParser`）实现了 `get_hf_file_to_dict`。 它主要通过调用 `Path`, `file_path.is_file`, `FileNotFoundError`, `open`, `json.load` 来完成任务。 状态更新主要写入 `file_path`。 实现中使用了条件分支、上下文管理资源。

### Lines 303-313: Method `MistralConfigParser._download_mistral_config_file` / 方法 `MistralConfigParser._download_mistral_config_file`
```python
    def _download_mistral_config_file(self, model, revision) -> dict:
        config_file_name = "params.json"
        config_dict = self.get_hf_file_to_dict(config_file_name, model, revision)
        if config_dict is None:
            raise ValueError(
                f"Failed to load mistral '{config_file_name}' config for model "
                f"{model}. Please check if the model is a mistral-format model "
                f"and if the config file exists."
            )
        assert isinstance(config_dict, dict)
        return config_dict
```
**EN:** This method implements `_download_mistral_config_file` on `MistralConfigParser`. It primarily calls `self.get_hf_file_to_dict`, `isinstance`, `ValueError` to complete its work. State updates are written into `config_file_name`, `config_dict`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MistralConfigParser`）实现了 `_download_mistral_config_file`。 它主要通过调用 `self.get_hf_file_to_dict`, `isinstance`, `ValueError` 来完成任务。 状态更新主要写入 `config_file_name`, `config_dict`。 实现中使用了条件分支。

### Lines 315-345: Method `MistralConfigParser.parse` / 方法 `MistralConfigParser.parse`
```python
    def parse(
        self,
        model: str | Path,
        revision: str | None = None,
        **kwargs,
    ) -> tuple[dict, PretrainedConfig]:
        config_dict = self._download_mistral_config_file(model, revision)
        if config_dict.get("max_position_embeddings") is None:
            logger.warning(
                "The params.json file is missing 'max_position_embeddings'"
                " and could not get a value from the HF config."
                " Defaulting to 128000"
            )
            config_dict["max_position_embeddings"] = 128_000

        config_dict, config = adapt_config_dict(config_dict, model)

        # Mistral configs may define sliding_window as list[int]. Convert it
        # to int and add the layer_types list[str] to make it HF compatible
        if (sliding_window := getattr(config, "sliding_window", None)) and isinstance(
            sliding_window, list
        ):
            pattern_repeats = config.num_hidden_layers // len(sliding_window)
            layer_types = sliding_window * pattern_repeats
            config.layer_types = [
                "full_attention" if layer_type is None else "sliding_attention"
                for layer_type in layer_types
            ]
            config.sliding_window = next(filter(None, sliding_window), None)

        return config_dict, config
```
**EN:** This method implements `parse` on `MistralConfigParser`. It primarily calls `self._download_mistral_config_file`, `adapt_config_dict`, `config_dict.get`, `logger.warning`, `isinstance`, `next` to complete its work. State updates are written into `config_dict`, `pattern_repeats`, `layer_types`, `config.layer_types`, `config.sliding_window`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MistralConfigParser`）实现了 `parse`。 它主要通过调用 `self._download_mistral_config_file`, `adapt_config_dict`, `config_dict.get`, `logger.warning`, `isinstance`, `next` 来完成任务。 状态更新主要写入 `config_dict`, `pattern_repeats`, `layer_types`, `config.layer_types`, `config.sliding_window`。 实现中使用了条件分支。

### Lines 348-358: Function `is_mistral_model` / 函数 `is_mistral_model`
```python
def is_mistral_model(name) -> bool:
    """Return True if *name* refers to a Mistral model needing the custom parser."""
    lower = str(name).lower()
    if "mistral-large-3" in lower or "mistral-small-4" in lower or "leanstral" in lower:
        return True
    # EAGLE drafts for Mistral targets ship native-format only (params.json +
    # consolidated.safetensors, no config.json), so route them through the
    # custom parser regardless of the base model name.
    if "eagle" in lower and "mistral" in lower:
        return True
    return False
```
**EN:** This function implements `is_mistral_model`. It primarily calls `str.lower`, `str` to complete its work. State updates are written into `lower`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `is_mistral_model`。 它主要通过调用 `str.lower`, `str` 来完成任务。 状态更新主要写入 `lower`。 实现中使用了条件分支。

### Lines 361-384: Function `load_mistral_config` / 函数 `load_mistral_config`
```python
@lru_cache(maxsize=2)
def load_mistral_config(
    model_path: str,
    trust_remote_code: bool = False,
    revision: Optional[str] = None,
):
    """Load and parse a Mistral model config via the custom params.json format.

    Returns a ``PretrainedConfig`` with dict sub-configs (text_config,
    vision_config) converted to proper AutoConfig objects.
    """
    local_path = download_from_hf(model_path)
    parser = MistralConfigParser()
    config_dict, _ = parser.parse(local_path)

    with tempfile.NamedTemporaryFile(mode="w+", suffix=".json") as f:
        json.dump(config_dict, f)
        f.flush()
        loaded_config = AutoConfig.from_pretrained(
            f.name, trust_remote_code=trust_remote_code, revision=revision
        )
    _ensure_sub_configs(loaded_config, "text_config", "vision_config")

    return loaded_config
```
**EN:** This function implements `load_mistral_config`. It primarily calls `lru_cache`, `download_from_hf`, `MistralConfigParser`, `parser.parse`, `_ensure_sub_configs`, `tempfile.NamedTemporaryFile` to complete its work. State updates are written into `local_path`, `parser`, `loaded_config`. The implementation relies on context-managed resources.
**CN:** 该函数实现了 `load_mistral_config`。 它主要通过调用 `lru_cache`, `download_from_hf`, `MistralConfigParser`, `parser.parse`, `_ensure_sub_configs`, `tempfile.NamedTemporaryFile` 来完成任务。 状态更新主要写入 `local_path`, `parser`, `loaded_config`。 实现中使用了上下文管理资源。

### Lines 387-412: Function `wrap_as_pixtral` / 函数 `wrap_as_pixtral`
```python
def wrap_as_pixtral(processor, config):
    """Wrap a tokenizer as a PixtralProcessor for Mistral vision models."""
    from transformers.models.pixtral.image_processing_pixtral import (
        PixtralImageProcessor,
    )
    from transformers.models.pixtral.processing_pixtral import (
        PixtralProcessor as HFPixtralProcessor,
    )

    vision_config = config.vision_config
    patch_size = vision_config.patch_size
    image_size = vision_config.image_size
    spatial_merge_size = getattr(vision_config, "spatial_merge_size", 1)

    effective_patch = patch_size * spatial_merge_size
    image_processor = PixtralImageProcessor(
        do_resize=True,
        size={"longest_edge": image_size},
        patch_size={"height": effective_patch, "width": effective_patch},
    )
    return HFPixtralProcessor(
        image_processor=image_processor,
        tokenizer=processor,
        patch_size=patch_size,
        spatial_merge_size=spatial_merge_size,
    )
```
**EN:** This function implements `wrap_as_pixtral`. It primarily calls `getattr`, `PixtralImageProcessor`, `HFPixtralProcessor` to complete its work. State updates are written into `vision_config`, `patch_size`, `image_size`, `spatial_merge_size`, `effective_patch`, `image_processor`.
**CN:** 该函数实现了 `wrap_as_pixtral`。 它主要通过调用 `getattr`, `PixtralImageProcessor`, `HFPixtralProcessor` 来完成任务。 状态更新主要写入 `vision_config`, `patch_size`, `image_size`, `spatial_merge_size`, `effective_patch`, `image_processor`。

### Lines 416-430: Constants and shared state / 常量与共享状态
```python
_MISTRAL_COMMON_REJECTED_KWARGS = frozenset(
    {
        "trust_remote_code",
        "tokenizer_revision",
        "use_fast",
        "_from_auto",
        "clean_up_tokenization_spaces",
    }
)

# Models whose tokenizer should be loaded from a different checkpoint.
_MISTRAL_TOKENIZER_REDIRECTS = {
    # TODO(Xinyuan): Remove this once we have a proper tokenizer for Devstral
    "mistralai/Devstral-Small-2505": "mistralai/Mistral-Small-3.1-24B-Instruct-2503",
}
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_MISTRAL_COMMON_REJECTED_KWARGS`, `_MISTRAL_TOKENIZER_REDIRECTS`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_MISTRAL_COMMON_REJECTED_KWARGS`, `_MISTRAL_TOKENIZER_REDIRECTS`。

### Lines 433-446: Function `retry_without_mistral_common_kwargs` / 函数 `retry_without_mistral_common_kwargs`
```python
def retry_without_mistral_common_kwargs(tokenizer_name, *args, **common_kwargs):
    """Retry ``AutoTokenizer.from_pretrained`` without kwargs that MistralCommon rejects.

    Returns the loaded tokenizer, or *None* if the error is not a
    MistralCommon kwargs rejection.
    """
    from transformers import AutoTokenizer

    stripped = {
        k: v
        for k, v in common_kwargs.items()
        if k not in _MISTRAL_COMMON_REJECTED_KWARGS
    }
    return AutoTokenizer.from_pretrained(tokenizer_name, *args, **stripped)
```
**EN:** This function implements `retry_without_mistral_common_kwargs`. It primarily calls `AutoTokenizer.from_pretrained`, `common_kwargs.items` to complete its work. State updates are written into `stripped`.
**CN:** 该函数实现了 `retry_without_mistral_common_kwargs`。 它主要通过调用 `AutoTokenizer.from_pretrained`, `common_kwargs.items` 来完成任务。 状态更新主要写入 `stripped`。

### Lines 449-518: Function `patch_mistral_common_tokenizer` / 函数 `patch_mistral_common_tokenizer`
```python
def patch_mistral_common_tokenizer(tokenizer):
    """Patch MistralCommonTokenizer/Backend to be compatible with HF tokenizer API.

    MistralCommon tokenizers (used by Voxtral, Pixtral, etc.) reject several
    standard kwargs and lack some attributes that sglang expects.  We wrap the
    offending methods once at load time so that the rest of the codebase does
    not need any special-casing.
    """
    cls_name = type(tokenizer).__name__
    if "MistralCommon" not in cls_name:
        return tokenizer
    if getattr(tokenizer, "_mistral_common_patched", False):
        return tokenizer
    tokenizer._mistral_common_patched = True

    if not hasattr(tokenizer, "get_added_vocab"):
        tokenizer.get_added_vocab = lambda: {}

    # Set a chat_template containing "audio" so that sglang's content format
    # detector returns "openai" (which preserves audio_url extraction).
    if not hasattr(tokenizer, "chat_template") or tokenizer.chat_template is None:
        tokenizer.chat_template = "<!-- audio/image multimodal -->"

    _orig_convert = tokenizer.convert_tokens_to_ids

    def _safe_convert(val):
        try:
            return _orig_convert(val)
# ... omitted for brevity ...
                cleaned.append(msg)
            else:
                cleaned.append(msg)
        return tokenizer._orig_apply_chat_template(cleaned, **kwargs)

    tokenizer.apply_chat_template = _safe_apply_chat_template
```
**EN:** This function implements `patch_mistral_common_tokenizer`. It primarily calls `getattr`, `_drop_kwargs`, `type`, `hasattr`, `kwargs.pop`, `tokenizer._orig_apply_chat_template` to complete its work. State updates are written into `cls_name`, `tokenizer._mistral_common_patched`, `_orig_convert`, `tokenizer.convert_tokens_to_ids`, `tokenizer.decode`, `tokenizer.batch_decode`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `patch_mistral_common_tokenizer`。 它主要通过调用 `getattr`, `_drop_kwargs`, `type`, `hasattr`, `kwargs.pop`, `tokenizer._orig_apply_chat_template` 来完成任务。 状态更新主要写入 `cls_name`, `tokenizer._mistral_common_patched`, `_orig_convert`, `tokenizer.convert_tokens_to_ids`, `tokenizer.decode`, `tokenizer.batch_decode`。 实现中使用了条件分支、迭代逻辑、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `MistralConfigParser`
- **Functions / 函数**: `adapt_config_dict`, `_remap_mistral_vision_args`, `_remap_mistral_yarn_args`, `_remap_general_mistral_args`, `_remap_mistral_quantization_args`, `_remap_mistral_audio_args`, `_remap_moe_args`, `is_mistral_model`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.utils`, `.common`
- **External / 外部依赖**: `transformers`, `transformers.models.pixtral.image_processing_pixtral`, `transformers.models.pixtral.processing_pixtral`
- **Standard library / 标准库**: `json`, `tempfile`, `functools`, `pathlib`, `typing`
