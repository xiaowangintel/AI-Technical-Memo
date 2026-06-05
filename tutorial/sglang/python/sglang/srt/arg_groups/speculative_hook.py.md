# speculative_hook.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/arg_groups/speculative_hook.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the argument grouping and hook injection part of the SRT runtime and implements logic centered on `speculative_hook`. It exposes primary entry points such as `_resolve_speculative_algorithm_alias`, `handle_speculative_decoding`, `_handle_dflash`. / 该模块属于 SRT 运行时的参数分组与钩子注入部分，主要实现围绕 `speculative_hook` 的逻辑。 它对外提供的主要入口包括 `_resolve_speculative_algorithm_alias`, `handle_speculative_decoding`, `_handle_dflash`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module imports, constants, and setup
```python
import json
import logging
from typing import TYPE_CHECKING, Optional

from sglang.srt.environ import envs

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 13-47: Function _resolve_speculative_algorithm_alias
```python
def _resolve_speculative_algorithm_alias(
    speculative_algorithm: Optional[str],
    speculative_draft_model_path: Optional[str],
    trust_remote_code: bool = False,
) -> Optional[str]:
    """Resolve CLI speculative algorithm; NEXTN/EAGLE may become FROZEN_KV_MTP for Gemma4 assistant drafts."""

    is_gemma4_draft = False
    if speculative_draft_model_path:
        from sglang.srt.utils.hf_transformers_utils import get_config

        cfg = get_config(
            speculative_draft_model_path, trust_remote_code=trust_remote_code
        )
        is_gemma4_draft = "Gemma4AssistantForCausalLM" in (
            getattr(cfg, "architectures", None) or []
        )

    if speculative_algorithm == "EAGLE3" and is_gemma4_draft:
        raise ValueError(
            "Gemma4AssistantForCausalLM draft requires "
            "--speculative-algorithm NEXTN or EAGLE; EAGLE3 is "
            "not supported for this draft architecture."
        )

    if speculative_algorithm == "NEXTN" or speculative_algorithm == "EAGLE":
        if is_gemma4_draft:
            logger.info(
                "Detected Gemma4AssistantForCausalLM draft; "
                f"promoting --speculative-algorithm {speculative_algorithm} to FROZEN_KV_MTP."
            )
            return "FROZEN_KV_MTP"
        return "EAGLE"

    return speculative_algorithm
```
**EN:** This callable implements `_resolve_speculative_algorithm_alias`. It takes `speculative_algorithm`, `speculative_draft_model_path`, `trust_remote_code` and mainly implements resolve speculative algorithm alias. The docstring states: "Resolve CLI speculative algorithm; NEXTN/EAGLE may become FROZEN_KV_MTP for Gemma4 assistant drafts." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_resolve_speculative_algorithm_alias`。它接收 `speculative_algorithm`, `speculative_draft_model_path`, `trust_remote_code`，主要用于实现 resolve speculative algorithm alias 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 50-109: Function handle_speculative_decoding (part 1/2)
```python
def handle_speculative_decoding(server_args: "ServerArgs") -> None:
    if (
        server_args.speculative_draft_model_path is not None
        and server_args.speculative_draft_model_revision is None
    ):
        server_args.speculative_draft_model_revision = "main"

    if server_args.speculative_moe_runner_backend is None:
        server_args.speculative_moe_runner_backend = server_args.moe_runner_backend

    if server_args.speculative_algorithm is not None:
        server_args.speculative_algorithm = server_args.speculative_algorithm.upper()

    server_args.speculative_algorithm = _resolve_speculative_algorithm_alias(
        server_args.speculative_algorithm,
        server_args.speculative_draft_model_path,
        trust_remote_code=server_args.trust_remote_code,
    )

    # Validate --speculative-draft-window-size once, regardless of algorithm.
    # Consumed by DFLASH (compact draft KV cache) and Llama EAGLE-3 (drafter attention SWA).
    if server_args.speculative_draft_window_size is not None:
        window_size = int(server_args.speculative_draft_window_size)
        if window_size <= 0:
            raise ValueError(
                f"--speculative-draft-window-size must be positive, got {window_size}."
            )
        server_args.speculative_draft_window_size = window_size
        if server_args.speculative_algorithm not in ("EAGLE3", "DFLASH"):
            logger.warning(
                "--speculative-draft-window-size has no effect with "
                "speculative_algorithm=%s (honored by Llama EAGLE-3 and DFLASH only).",
                server_args.speculative_algorithm,
            )

    if server_args.speculative_algorithm is not None:
        from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
        from sglang.srt.speculative.spec_registry import CustomSpecAlgo

        algo = SpeculativeAlgorithm.from_string(server_args.speculative_algorithm)

        # TODO: move the per-algorithm validation below into spec module hooks.
        if isinstance(algo, CustomSpecAlgo) and algo.validate_server_args is not None:
            algo.validate_server_args(server_args)

    if server_args.speculative_skip_dp_mlp_sync:
        assert server_args.speculative_algorithm == "EAGLE", (
            "--speculative-skip-dp-mlp-sync is only supported with "
            f"speculative_algorithm == EAGLE, got {server_args.speculative_algorithm}."
        )

    if server_args.speculative_algorithm == "DFLASH":
        _handle_dflash(server_args)
    elif server_args.speculative_algorithm == "FROZEN_KV_MTP":
        _handle_frozen_kv_mtp(server_args)
    elif server_args.speculative_algorithm in ("EAGLE", "EAGLE3", "STANDALONE"):
        _handle_eagle_family(server_args)
    elif server_args.speculative_algorithm == "NGRAM":
        _handle_ngram(server_args)

```
**EN:** This callable implements `handle_speculative_decoding`. It takes `server_args` and mainly implements handle speculative decoding. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `handle_speculative_decoding`。它接收 `server_args`，主要用于实现 handle speculative decoding 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 110-111: Function handle_speculative_decoding (part 2/2)
```python
    if server_args.speculative_adaptive:
        _maybe_disable_adaptive(server_args)
```
**EN:** This callable implements `handle_speculative_decoding`. It takes `server_args` and mainly implements handle speculative decoding. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `handle_speculative_decoding`。它接收 `server_args`，主要用于实现 handle speculative decoding 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。

### Lines 114-173: Function _handle_dflash (part 1/2)
```python
def _handle_dflash(server_args: "ServerArgs") -> None:
    if server_args.enable_dp_attention:
        raise ValueError(
            "Currently DFLASH speculative decoding does not support dp attention."
        )

    if server_args.pp_size != 1:
        raise ValueError(
            "Currently DFLASH speculative decoding only supports pp_size == 1."
        )

    if server_args.speculative_draft_model_path is None:
        raise ValueError(
            "DFLASH speculative decoding requires setting --speculative-draft-model-path."
        )

    # DFLASH does not use EAGLE-style `num_steps`/`topk`, but those fields still
    # affect generic scheduler/KV-cache accounting (buffer sizing, KV freeing,
    # RoPE reservation). Force them to 1 to avoid surprising memory behavior.
    #
    # For DFlash, the natural unit is `block_size` (verify window length).
    if server_args.speculative_num_steps is None:
        server_args.speculative_num_steps = 1
    elif int(server_args.speculative_num_steps) != 1:
        logger.warning(
            "DFLASH only supports speculative_num_steps == 1; overriding speculative_num_steps=%s to 1.",
            server_args.speculative_num_steps,
        )
        server_args.speculative_num_steps = 1

    if server_args.speculative_eagle_topk is None:
        server_args.speculative_eagle_topk = 1
    elif int(server_args.speculative_eagle_topk) != 1:
        logger.warning(
            "DFLASH only supports speculative_eagle_topk == 1; overriding speculative_eagle_topk=%s to 1.",
            server_args.speculative_eagle_topk,
        )
        server_args.speculative_eagle_topk = 1

    if server_args.speculative_dflash_block_size is not None:
        if int(server_args.speculative_dflash_block_size) <= 0:
            raise ValueError(
                "DFLASH requires --speculative-dflash-block-size to be positive, "
                f"got {server_args.speculative_dflash_block_size}."
            )
        if server_args.speculative_num_draft_tokens is not None and int(
            server_args.speculative_num_draft_tokens
        ) != int(server_args.speculative_dflash_block_size):
            raise ValueError(
                "Both --speculative-num-draft-tokens and --speculative-dflash-block-size are set "
                "but they differ. For DFLASH they must match. "
                f"speculative_num_draft_tokens={server_args.speculative_num_draft_tokens}, "
                f"speculative_dflash_block_size={server_args.speculative_dflash_block_size}."
            )
        server_args.speculative_num_draft_tokens = int(
            server_args.speculative_dflash_block_size
        )

    if server_args.speculative_num_draft_tokens is None:
        from sglang.srt.speculative.dflash_utils import (
```
**EN:** This callable implements `_handle_dflash`. It takes `server_args` and mainly implements handle dflash. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_handle_dflash`。它接收 `server_args`，主要用于实现 handle dflash 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 174-230: Function _handle_dflash (part 2/2)
```python
            parse_dflash_draft_config,
        )

        model_override_args = json.loads(server_args.json_model_override_args)
        inferred_block_size = None
        try:
            from sglang.srt.utils.hf_transformers_utils import get_config

            draft_hf_config = get_config(
                server_args.speculative_draft_model_path,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.speculative_draft_model_revision,
                model_override_args=model_override_args,
            )
            inferred_block_size = parse_dflash_draft_config(
                draft_hf_config=draft_hf_config
            ).resolve_block_size(default=None)
        except Exception as e:
            logger.warning(
                "Failed to infer DFLASH block_size from draft model config; "
                "defaulting speculative_num_draft_tokens to 16. Error: %s",
                e,
            )

        if inferred_block_size is None:
            inferred_block_size = 16
            logger.warning(
                "speculative_num_draft_tokens is not set; defaulting to %d for DFLASH.",
                inferred_block_size,
            )
        server_args.speculative_num_draft_tokens = inferred_block_size

    if server_args.speculative_draft_window_size is not None:
        draft_tokens = int(server_args.speculative_num_draft_tokens)
        if server_args.speculative_draft_window_size < draft_tokens:
            raise ValueError(
                "--speculative-draft-window-size must be >= "
                "--speculative-num-draft-tokens (block_size). "
                f"window_size={server_args.speculative_draft_window_size}, block_size={draft_tokens}."
            )

    if server_args.max_running_requests is None:
        server_args.max_running_requests = 48
        logger.warning(
            "Max running requests is reset to 48 for speculative decoding. You can override this by explicitly setting --max-running-requests."
        )

    server_args.disable_overlap_schedule = True
    logger.warning(
        "Overlap scheduler is disabled when using DFLASH speculative decoding (spec v2 is not supported yet)."
    )

    if server_args.enable_mixed_chunk:
        server_args.enable_mixed_chunk = False
        logger.warning(
            "Mixed chunked prefill is disabled because of using dflash speculative decoding."
        )
```
**EN:** This callable implements `_handle_dflash`. It takes `server_args` and mainly implements handle dflash. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_handle_dflash`。它接收 `server_args`，主要用于实现 handle dflash 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 233-250: Function _handle_frozen_kv_mtp
```python
def _handle_frozen_kv_mtp(server_args: "ServerArgs") -> None:
    if server_args.max_running_requests is None:
        server_args.max_running_requests = 48
        logger.warning(
            "Max running requests is reset to 48 for speculative decoding. You can override this by explicitly setting --max-running-requests."
        )

    server_args.disable_overlap_schedule = True
    logger.warning(
        "Overlap scheduler is disabled when using Frozen-KV MTP speculative decoding (spec v2 is not supported yet)."
    )

    if server_args.enable_mixed_chunk:
        server_args.enable_mixed_chunk = False
        logger.warning(
            "Mixed chunked prefill is disabled because of using "
            "Frozen-KV MTP speculative decoding."
        )
```
**EN:** This callable implements `_handle_frozen_kv_mtp`. It takes `server_args` and mainly implements handle frozen kv mtp. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_handle_frozen_kv_mtp`。它接收 `server_args`，主要用于实现 handle frozen kv mtp 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 253-312: Function _handle_eagle_family (part 1/2)
```python
def _handle_eagle_family(server_args: "ServerArgs") -> None:
    if (
        server_args.speculative_algorithm == "STANDALONE"
        and server_args.enable_dp_attention
    ):
        # TODO: support dp attention for standalone speculative decoding
        raise ValueError(
            "Currently standalone speculative decoding does not support dp attention."
        )

    if server_args.max_running_requests is None:
        server_args.max_running_requests = 48
        logger.warning(
            "Max running requests is reset to 48 for speculative decoding. You can override this by explicitly setting --max-running-requests."
        )

    spec_v1_reason = None
    if (
        server_args.speculative_eagle_topk is not None
        and server_args.speculative_eagle_topk > 1
        and not server_args.disable_overlap_schedule
    ):
        server_args.disable_overlap_schedule = True
        spec_v1_reason = "spec v2 currently only supports topk = 1"
    elif (
        not envs.SGLANG_ENABLE_SPEC_V2.get()
        and not server_args.disable_overlap_schedule
    ):
        server_args.disable_overlap_schedule = True
        spec_v1_reason = "SGLANG_ENABLE_SPEC_V2=False"

    if server_args.disable_overlap_schedule:
        logger.warning(
            "Spec v1 is used for eagle/eagle3/standalone speculative decoding because %s.",
            spec_v1_reason or "overlap schedule is disabled",
        )
    else:
        logger.warning(
            "Spec v2 is enabled by default for eagle/eagle3/standalone speculative decoding."
        )

    if server_args.enable_mixed_chunk:
        server_args.enable_mixed_chunk = False
        logger.warning(
            "Mixed chunked prefill is disabled because of using "
            "eagle speculative decoding."
        )

    model_arch = server_args.get_model_config().hf_config.architectures[0]
    if model_arch in [
        "DeepseekV32ForCausalLM",
        "DeepseekV3ForCausalLM",
        "DeepseekV4ForCausalLM",
        "Glm4MoeForCausalLM",
        "Glm4MoeLiteForCausalLM",
        "GlmMoeDsaForCausalLM",
        "BailingMoeForCausalLM",
        "BailingMoeV2ForCausalLM",
        "BailingMoeV2_5ForCausalLM",
        "MistralLarge3ForCausalLM",
```
**EN:** This callable implements `_handle_eagle_family`. It takes `server_args` and mainly implements handle eagle family. This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_handle_eagle_family`。它接收 `server_args`，主要用于实现 handle eagle family 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；读取环境变量驱动的配置。

### Lines 313-368: Function _handle_eagle_family (part 2/2)
```python
        "PixtralForConditionalGeneration",
        "HYV3ForCausalLM",
    ]:
        if server_args.speculative_draft_model_path is None:
            server_args.speculative_draft_model_path = server_args.model_path
            server_args.speculative_draft_model_revision = server_args.revision
        else:
            if model_arch not in [
                "MistralLarge3ForCausalLM",
                "PixtralForConditionalGeneration",
            ]:
                logger.warning(
                    "DeepSeek MTP does not require setting speculative_draft_model_path."
                )

    if server_args.speculative_num_steps is None:
        assert (
            server_args.speculative_eagle_topk is None
            and server_args.speculative_num_draft_tokens is None
        )
        from sglang.srt.server_args import auto_choose_speculative_params

        (
            server_args.speculative_num_steps,
            server_args.speculative_eagle_topk,
            server_args.speculative_num_draft_tokens,
        ) = auto_choose_speculative_params(server_args)

    if (
        server_args.attention_backend == "trtllm_mha"
        or server_args.decode_attention_backend == "trtllm_mha"
        or server_args.prefill_attention_backend == "trtllm_mha"
    ):
        if server_args.speculative_eagle_topk > 1:
            raise ValueError(
                "trtllm_mha backend only supports topk = 1 for speculative decoding."
            )

    if (
        server_args.speculative_eagle_topk == 1
        and server_args.speculative_num_draft_tokens
        != server_args.speculative_num_steps + 1
    ):
        logger.warning(
            "speculative_num_draft_tokens is adjusted to speculative_num_steps + 1 when speculative_eagle_topk == 1"
        )
        server_args.speculative_num_draft_tokens = server_args.speculative_num_steps + 1

    if (
        server_args.speculative_eagle_topk > 1
        and server_args.page_size > 1
        and server_args.attention_backend not in ["flashinfer", "fa3"]
    ):
        raise ValueError(
            "speculative_eagle_topk > 1 with page_size > 1 is unstable and produces incorrect results for paged attention backends. This combination is only supported for the 'flashinfer' backend."
        )
```
**EN:** This callable implements `_handle_eagle_family`. It takes `server_args` and mainly implements handle eagle family. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_handle_eagle_family`。它接收 `server_args`，主要用于实现 handle eagle family 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 371-429: Function _handle_ngram
```python
def _handle_ngram(server_args: "ServerArgs") -> None:
    if not server_args.device.startswith("cuda"):
        raise ValueError("Ngram speculative decoding only supports CUDA device.")

    if server_args.max_running_requests is None:
        server_args.max_running_requests = 48
        logger.warning(
            "Max running requests is reset to 48 for speculative decoding. You can override this by explicitly setting --max-running-requests."
        )

    server_args.disable_overlap_schedule = True
    server_args.enable_mixed_chunk = False
    server_args.speculative_eagle_topk = server_args.speculative_ngram_max_bfs_breadth
    if server_args.speculative_num_draft_tokens is None:
        server_args.speculative_num_draft_tokens = 12
        logger.warning(
            "speculative_num_draft_tokens is set to 12 by default for ngram speculative decoding. "
            "You can override this by explicitly setting --speculative-num-draft-tokens."
        )
    if server_args.speculative_ngram_external_corpus_path is not None:
        if server_args.speculative_ngram_external_sam_budget <= 0:
            raise ValueError(
                "--speculative-ngram-external-sam-budget must be positive when "
                "--speculative-ngram-external-corpus-path is set."
            )
        if server_args.speculative_ngram_external_corpus_max_tokens <= 0:
            raise ValueError(
                "--speculative-ngram-external-corpus-max-tokens must be positive when "
                "--speculative-ngram-external-corpus-path is set."
            )
        if (
            server_args.speculative_ngram_external_sam_budget
            > server_args.speculative_num_draft_tokens - 1
        ):
            raise ValueError(
                "speculative_ngram_external_sam_budget must be less than or equal to "
                f"speculative_num_draft_tokens - 1 ({server_args.speculative_num_draft_tokens - 1})."
            )
    logger.warning(
        "The overlap scheduler and mixed chunked prefill are disabled because of "
        "using ngram speculative decoding."
    )

    if (
        server_args.speculative_eagle_topk > 1
        and server_args.page_size > 1
        and server_args.attention_backend != "flashinfer"
    ):
        raise ValueError(
            f"speculative_eagle_topk({server_args.speculative_eagle_topk}) > 1 "
            f"with page_size({server_args.page_size}) > 1 is unstable "
            "and produces incorrect results for paged attention backends. "
            "This combination is only supported for the 'flashinfer' backend."
        )
    if server_args.enable_dp_attention:
        # TODO: support dp attention for ngram speculative decoding
        raise ValueError(
            "Currently ngram speculative decoding does not support dp attention."
        )
```
**EN:** This callable implements `_handle_ngram`. It takes `server_args` and mainly implements handle ngram. In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_handle_ngram`。它接收 `server_args`，主要用于实现 handle ngram 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

### Lines 432-443: Function _maybe_disable_adaptive
```python
def _maybe_disable_adaptive(server_args: "ServerArgs") -> None:
    from sglang.srt.speculative.adaptive_spec_params import (
        adaptive_unsupported_reason,
    )

    reason = adaptive_unsupported_reason(server_args)
    if reason is not None:
        logger.warning(
            f"speculative_adaptive disabled: {reason}. "
            "Falling back to static speculative params."
        )
        server_args.speculative_adaptive = False
```
**EN:** This callable implements `_maybe_disable_adaptive`. It takes `server_args` and mainly implements maybe disable adaptive. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_maybe_disable_adaptive`。它接收 `server_args`，主要用于实现 maybe disable adaptive 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

## Key Concepts / 关键概念
- `_resolve_speculative_algorithm_alias`: implements resolve speculative algorithm alias / 实现 resolve speculative algorithm alias 相关逻辑
- `handle_speculative_decoding`: implements handle speculative decoding / 实现 handle speculative decoding 相关逻辑
- `_handle_dflash`: implements handle dflash / 实现 handle dflash 相关逻辑
- `_handle_frozen_kv_mtp`: implements handle frozen kv mtp / 实现 handle frozen kv mtp 相关逻辑
- `_handle_eagle_family`: implements handle eagle family / 实现 handle eagle family 相关逻辑
- `_handle_ngram`: implements handle ngram / 实现 handle ngram 相关逻辑
- `_maybe_disable_adaptive`: implements maybe disable adaptive / 实现 maybe disable adaptive 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.server_args`, `sglang.srt.speculative.adaptive_spec_params`, `sglang.srt.utils.hf_transformers_utils`, `sglang.srt.speculative.spec_info`, `sglang.srt.speculative.spec_registry`, `sglang.srt.speculative.dflash_utils`
