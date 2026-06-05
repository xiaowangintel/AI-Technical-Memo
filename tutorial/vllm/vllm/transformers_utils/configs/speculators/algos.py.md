# algos.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/configs/speculators/algos.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines speculative-decoding configuration helpers for Algos. / [CN] 为 Algos 定义推测解码配置辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-4: Module state and constants
```python
SUPPORTED_SPECULATORS_TYPES = {}
```
**EN:** This block defines module-level constants/defaults such as `SUPPORTED_SPECULATORS_TYPES`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `SUPPORTED_SPECULATORS_TYPES`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 7-12: Function `register_speculator`
```python
def register_speculator(name):
    def decorator(fn):
        SUPPORTED_SPECULATORS_TYPES[name] = fn
        return fn

    return decorator
```
**EN:** This function registers speculator. Main inputs include `name`.
**CN:** 该函数负责完成 `register_speculator` 对应的核心步骤。 主要输入参数包括 `name`。

### Lines 16-43: Function `update_eagle3`
```python
def update_eagle3(config_dict: dict, pre_trained_config: dict) -> None:
    """
    Apply Eagle-3 specific configuration transformations to the `dict` used to
    construct the Transformers PreTrainedConfig.

    Eagle-3 specific fields:
    - draft_vocab_size: Size of the draft model's vocabulary
    - target_hidden_size: Hidden size of the target model
    - norm_before_residual: Whether to apply norm before residual connection
    - norm_before_fc: Whether to apply RMSNorm before the fc projection
    - eagle_aux_hidden_state_layer_ids: List of layer indices from the base
        model to use as auxiliary inputs for the Eagle3 drafter. These layers
        provide intermediate hidden states that help the drafter make better
        predictions. This is the standard field used in Eagle3 checkpoints.
    """

    pre_trained_config["draft_vocab_size"] = config_dict.get("draft_vocab_size")
    if config_dict.get("target_hidden_size") is not None:
        pre_trained_config["target_hidden_size"] = config_dict["target_hidden_size"]
    pre_trained_config["norm_before_residual"] = config_dict.get(
        "norm_before_residual", True
    )
    pre_trained_config["norm_before_fc"] = config_dict.get("norm_before_fc", False)
    pre_trained_config["architectures"] = ["Eagle3LlamaForCausalLM"]
    if config_dict.get("eagle_aux_hidden_state_layer_ids"):
        pre_trained_config["eagle_aux_hidden_state_layer_ids"] = config_dict[
            "eagle_aux_hidden_state_layer_ids"
        ]
```
**EN:** This function updates eagle3. The docstring states that Apply Eagle-3 specific configuration transformations to the `dict` used to Main inputs include `config_dict`, `pre_trained_config`. Decorators such as `register_speculator('eagle3')` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `update_eagle3` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config_dict`, `pre_trained_config`。 装饰器如 `register_speculator('eagle3')` 会影响缓存、校验或分发行为。

### Lines 47-74: Function `update_peagle`
```python
def update_peagle(config_dict: dict, pre_trained_config: dict) -> None:
    """
    Apply PEagle (Parallel Eagle) specific configuration transformations to
    the `dict` used to construct the Transformers PreTrainedConfig.

    PEagle specific fields:
    - draft_vocab_size: Size of the draft model's vocabulary
    - target_hidden_size: Hidden size of the target model
    - norm_before_residual: Whether to apply norm before residual connection
    - norm_before_fc: Whether to apply RMSNorm before the fc projection
    - mask_token_id (required): Token ID used for parallel drafting mask
        placeholders, mapped to pard_token for the proposer
    - eagle_aux_hidden_state_layer_ids: Layer indices from the target model
        whose intermediate hidden states are used as auxiliary inputs
    """
    pre_trained_config["architectures"] = ["PeagleLlamaForCausalLM"]
    pre_trained_config["draft_vocab_size"] = config_dict.get("draft_vocab_size")
    if config_dict.get("target_hidden_size") is not None:
        pre_trained_config["target_hidden_size"] = config_dict["target_hidden_size"]
    pre_trained_config["norm_before_residual"] = config_dict.get(
        "norm_before_residual", False
    )
    pre_trained_config["norm_before_fc"] = config_dict.get("norm_before_fc", False)
    pre_trained_config["pard_token"] = config_dict["mask_token_id"]
    if config_dict.get("eagle_aux_hidden_state_layer_ids"):
        pre_trained_config["eagle_aux_hidden_state_layer_ids"] = config_dict[
            "eagle_aux_hidden_state_layer_ids"
        ]
```
**EN:** This function updates peagle. The docstring states that Apply PEagle (Parallel Eagle) specific configuration transformations to Main inputs include `config_dict`, `pre_trained_config`. Decorators such as `register_speculator('peagle')` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `update_peagle` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config_dict`, `pre_trained_config`。 装饰器如 `register_speculator('peagle')` 会影响缓存、校验或分发行为。

### Lines 78-105: Function `update_dflash`
```python
def update_dflash(config_dict: dict, pre_trained_config: dict) -> None:
    """
    Apply DFlash specific configuration transformations to the `dict` used to
    construct the Transformers PreTrainedConfig.

    DFlash specific fields:
    - draft_vocab_size: Size of the draft model's vocabulary
    - target_hidden_size: Hidden size of the target model
    - mask_token_id (required): Token ID used for parallel drafting mask
        placeholders
    - aux_hidden_state_layer_ids (required): Layer indices from the target
        model whose intermediate hidden states are used as context for the
        DFlash drafter. Mapped to both eagle_aux_hidden_state_layer_ids
        (for gpu_model_runner) and dflash_config.target_layer_ids (for the
        DFlash model).
    """
    pre_trained_config["architectures"] = ["DFlashDraftModel"]
    pre_trained_config["draft_vocab_size"] = config_dict.get("draft_vocab_size")
    if config_dict.get("target_hidden_size") is not None:
        pre_trained_config["target_hidden_size"] = config_dict["target_hidden_size"]

    aux_layer_ids = config_dict["aux_hidden_state_layer_ids"]
    pre_trained_config["eagle_aux_hidden_state_layer_ids"] = aux_layer_ids

    pre_trained_config["dflash_config"] = {
        "mask_token_id": config_dict["mask_token_id"],
        "target_layer_ids": aux_layer_ids,
    }
```
**EN:** This function updates dflash. The docstring states that Apply DFlash specific configuration transformations to the `dict` used to Main inputs include `config_dict`, `pre_trained_config`. Decorators such as `register_speculator('dflash')` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `update_dflash` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `config_dict`, `pre_trained_config`。 装饰器如 `register_speculator('dflash')` 会影响缓存、校验或分发行为。

## Key Concepts / 关键概念
- **EN:** Speculative-decoding configs expose extra knobs for draft models, acceptance checks, and auxiliary generation behavior.
- **CN:** 推测解码配置会暴露草稿模型、接受判定和辅助生成行为所需的额外参数。
- **EN:** Configuration classes extend upstream Hugging Face metadata with vLLM-specific defaults, validation, or compatibility patches.
- **CN:** 配置类会在上游 Hugging Face 元数据之上加入 vLLM 专用默认值、校验逻辑或兼容补丁。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: none.
- **CN:** 外部依赖包：无。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
