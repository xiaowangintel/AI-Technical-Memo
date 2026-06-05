# deepseek_v4_hook.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/arg_groups/deepseek_v4_hook.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the argument grouping and hook injection part of the SRT runtime and implements logic centered on `deepseek_v4_hook`. It exposes primary entry points such as `apply_deepseek_v4_defaults`, `validate_deepseek_v4_cp`. / 该模块属于 SRT 运行时的参数分组与钩子注入部分，主要实现围绕 `deepseek_v4_hook` 的逻辑。 它对外提供的主要入口包括 `apply_deepseek_v4_defaults`, `validate_deepseek_v4_cp`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module imports, constants, and setup
```python
import logging
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 10-59: Function apply_deepseek_v4_defaults
```python
def apply_deepseek_v4_defaults(server_args: "ServerArgs", model_arch: str) -> None:
    """Apply DeepSeek V4 model-specific server arg defaults and constraints."""
    from sglang.srt.environ import envs
    from sglang.srt.server_args import ServerArgs

    server_args.attention_backend = "dsv4"
    server_args.page_size = 256
    logger.info(
        f"Use dsv4 attention backend for {model_arch}, setting page_size to 256."
    )

    if server_args.max_running_requests is None:
        server_args.max_running_requests = 256
        logger.warning(
            f"Setting max_running_requests to {server_args.max_running_requests} for {model_arch}."
        )

    if server_args.kv_cache_dtype == "auto":
        server_args.kv_cache_dtype = "fp8_e4m3"
        logger.warning(
            f"Setting KV cache dtype to {server_args.kv_cache_dtype} for {model_arch}."
        )
    assert server_args.kv_cache_dtype in [
        "fp8_e4m3"
    ], f"{server_args.kv_cache_dtype} is not supported for {model_arch}"

    if server_args.speculative_algorithm is not None:
        assert (
            server_args.speculative_algorithm == "EAGLE"
        ), f"Only EAGLE speculative algorithm is supported for {model_arch}"
        assert (
            server_args.speculative_eagle_topk == 1
        ), f"Only EAGLE speculative algorithm with topk == 1 is supported for {model_arch}"

        if not envs.SGLANG_ENABLE_SPEC_V2.get():
            envs.SGLANG_ENABLE_SPEC_V2.set(True)
            logger.warning("Spec v2 is enabled for EAGLE speculative decoding.")

    if server_args.swa_full_tokens_ratio == ServerArgs.swa_full_tokens_ratio:
        server_args.swa_full_tokens_ratio = 0.1
        logger.info(
            f"Setting swa_full_tokens_ratio to {server_args.swa_full_tokens_ratio} for {model_arch}."
        )

    if server_args.disaggregation_mode != "null" and server_args.pp_size > 1:
        # get_mla_kv_ptrs_with_pp cannot slice V4's buffer-type-organized
        # flat KV ptrs by PP layer range.
        raise ValueError(
            f"V4 PD disaggregation requires pp_size=1, got pp_size={server_args.pp_size}."
        )
```
**EN:** This callable implements `apply_deepseek_v4_defaults`. It takes `server_args`, `model_arch` and mainly implements apply deepseek v4 defaults. The docstring states: "Apply DeepSeek V4 model-specific server arg defaults and constraints." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `apply_deepseek_v4_defaults`。它接收 `server_args`, `model_arch`，主要用于实现 apply deepseek v4 defaults 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 62-86: Function validate_deepseek_v4_cp
```python
def validate_deepseek_v4_cp(server_args: "ServerArgs") -> None:
    """Validate DeepSeek V4 context-parallel configuration."""
    if not server_args.enable_nsa_prefill_context_parallel:
        return

    if server_args.nsa_prefill_cp_mode != "round-robin-split":
        raise ValueError(
            f"DeepSeekV4 only supports round-robin-split CP mode, "
            f"got {server_args.nsa_prefill_cp_mode}"
        )

    server_args.enable_dp_attention = True
    server_args.moe_dense_tp_size = 1
    server_args.attn_cp_size = server_args.tp_size // server_args.dp_size
    assert (
        server_args.dp_size == 1
    ), "For round-robin split mode, dp attention is not supported."
    assert (
        server_args.tp_size <= 8
    ), "Context parallel only supports single machine (tp_size <= 8). Cross-machine CP has precision issues."
    logger.warning(
        f"Enable Context Parallel for DeepSeekV4, "
        f"dp_size={server_args.dp_size}, moe_dense_tp_size={server_args.moe_dense_tp_size}, "
        f"attn_cp_size={server_args.attn_cp_size}, ep_size={server_args.ep_size}, tp_size={server_args.tp_size}"
    )
```
**EN:** This callable implements `validate_deepseek_v4_cp`. It takes `server_args` and mainly validates inputs and invariants. The docstring states: "Validate DeepSeek V4 context-parallel configuration." In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `validate_deepseek_v4_cp`。它接收 `server_args`，主要用于校验输入与不变量。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

## Key Concepts / 关键概念
- `apply_deepseek_v4_defaults`: implements apply deepseek v4 defaults / 实现 apply deepseek v4 defaults 相关逻辑
- `validate_deepseek_v4_cp`: validates inputs and invariants / 校验输入与不变量

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.server_args`, `sglang.srt.environ`
