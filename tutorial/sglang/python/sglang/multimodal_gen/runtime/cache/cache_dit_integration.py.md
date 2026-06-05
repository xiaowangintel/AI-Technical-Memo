# cache_dit_integration.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/cache/cache_dit_integration.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `CacheDitConfig`, `_patch_cache_dit_similarity`, and `_build_parallelism_config`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: cache-dit integration module for SGLang DiT pipelines. This module provides helper functions to enable cache-dit acceleration on transformer modules in SGLang's modular pipeline architecture. / 该文件属于多模态生成模块。它围绕 `CacheDitConfig`、`_patch_cache_dit_similarity` 和 `_build_parallelism_config` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-20: module setup and imports / 模块初始化与导入
```python
"""
cache-dit integration module for SGLang DiT pipelines.

This module provides helper functions to enable cache-dit acceleration
on transformer modules in SGLang's modular pipeline architecture.
"""

from dataclasses import dataclass
from typing import List, Optional

import torch
import torch.distributed as dist

from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_ring_parallel_world_size,
    get_tp_world_size,
    get_ulysses_parallel_world_size,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `torch.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`torch.distributed`、`sglang.multimodal_gen.runtime.distributed.parallel_state` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 22-38: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

import cache_dit
from cache_dit import (
    BlockAdapter,
    DBCacheConfig,
    ForwardPattern,
    ParamsModifier,
    TaylorSeerCalibratorConfig,
    steps_mask,
)
from cache_dit.caching.block_adapters import BlockAdapterRegister
from cache_dit.parallelism import ParallelismBackend, ParallelismConfig

from sglang.multimodal_gen.runtime.distributed.parallel_state import get_dit_group

_original_similarity = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_original_similarity`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_original_similarity` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 41-102: `_patch_cache_dit_similarity` implementation / `_patch_cache_dit_similarity` 实现
```python
def _patch_cache_dit_similarity():
    from cache_dit.caching.cache_contexts import cache_manager

    global _original_similarity
    if _original_similarity is not None:
        return

    _original_similarity = cache_manager.CachedContextManager.similarity

    def patched_similarity(self, t1, t2, *, threshold, parallelized=False, prefix="Fn"):
        if not parallelized:
            return _original_similarity(
                self,
                t1,
                t2,
                threshold=threshold,
                parallelized=parallelized,
                prefix=prefix,
            )

        sp_group = getattr(self, "_sglang_sp_group", None)
        tp_group = getattr(self, "_sglang_tp_group", None)
        tp_sp_group = getattr(self, "_sglang_tp_sp_group", None)
        target_group = tp_sp_group or sp_group or tp_group

        if target_group is None:
            return _original_similarity(
                self,
                t1,
                t2,
                threshold=threshold,
                parallelized=parallelized,
                prefix=prefix,
            )

        # Adapted from https://github.com/vipshop/cache-dit/blob/main/src/cache_dit/caching/cache_contexts/cache_manager.py#L495-L523
        condition_thresh = self.get_important_condition_threshold()
        if condition_thresh > 0.0:
            raw_diff = (t1 - t2).abs()
            token_m_df = raw_diff.mean(dim=-1)
            token_m_t1 = t1.abs().mean(dim=-1)
            token_diff = token_m_df / token_m_t1
            condition = token_diff > condition_thresh
            if condition.sum() > 0:
                condition = condition.unsqueeze(-1).expand_as(raw_diff)
                mean_diff = raw_diff[condition].mean()
                mean_t1 = t1[condition].abs().mean()
            else:
                mean_diff = (t1 - t2).abs().mean()
                mean_t1 = t1.abs().mean()
        else:
            mean_diff = (t1 - t2).abs().mean()
            mean_t1 = t1.abs().mean()

        dist.all_reduce(mean_diff, op=dist.ReduceOp.AVG, group=target_group)
        dist.all_reduce(mean_t1, op=dist.ReduceOp.AVG, group=target_group)

        diff = (mean_diff / mean_t1).item()
        self.add_residual_diff(diff)
        return diff < threshold

    cache_manager.CachedContextManager.similarity = patched_similarity
```
**EN:** This block defines function `_patch_cache_dit_similarity`. It handles patch cache dit similarity logic. Key calls include `getattr`, `self.get_important_condition_threshold`, `dist.all_reduce`, `item`, and `self.add_residual_diff`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `_patch_cache_dit_similarity`。 它用于处理 patch cache dit similarity 相关逻辑。 关键调用包括 `getattr`、`self.get_important_condition_threshold`、`dist.all_reduce`、`item` 和 `self.add_residual_diff`。 实现中包含条件分支。

### Lines 105-127: `_build_parallelism_config` implementation / `_build_parallelism_config` 实现
```python
def _build_parallelism_config(
    sp_group: Optional[torch.distributed.ProcessGroup],
    tp_group: Optional[torch.distributed.ProcessGroup],
):
    if sp_group is None and tp_group is None:
        return None

    ulysses_size = None
    ring_size = None
    if sp_group is not None:
        ulysses_size = get_ulysses_parallel_world_size()
        ring_size = get_ring_parallel_world_size()

    tp_size = None
    if tp_group is not None:
        tp_size = get_tp_world_size()

    return ParallelismConfig(
        backend=ParallelismBackend.AUTO,
        ulysses_size=ulysses_size,
        ring_size=ring_size,
        tp_size=tp_size,
    )
```
**EN:** This block defines function `_build_parallelism_config`. It builds parallelism config. Key calls include `ParallelismConfig`, `get_ulysses_parallel_world_size`, `get_ring_parallel_world_size`, and `get_tp_world_size`. The implementation branches on conditions. Parameters such as `sp_group`, and `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_build_parallelism_config`。 它用于构建parallelism config。 关键调用包括 `ParallelismConfig`、`get_ulysses_parallel_world_size`、`get_ring_parallel_world_size` 和 `get_tp_world_size`。 实现中包含条件分支。 本段逻辑主要由 `sp_group` 和 `tp_group` 等参数驱动。

### Lines 130-135: `_mark_transformer_parallelized` implementation / `_mark_transformer_parallelized` 实现
```python
def _mark_transformer_parallelized(transformer, config, sp_group, tp_group):
    if config is None:
        return

    transformer._is_parallelized = True
    transformer._parallelism_config = config
```
**EN:** This block defines function `_mark_transformer_parallelized`. It handles mark transformer parallelized logic. The implementation branches on conditions. Parameters such as `transformer`, `config`, `sp_group`, and `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_mark_transformer_parallelized`。 它用于处理 mark transformer parallelized 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `transformer`、`config`、`sp_group` 和 `tp_group` 等参数驱动。

### Lines 138-178: `get_scm_mask` implementation / `get_scm_mask` 实现
```python
def get_scm_mask(
    preset: str,
    num_inference_steps: int,
    compute_bins: Optional[List[int]] = None,
    cache_bins: Optional[List[int]] = None,
) -> Optional[List[int]]:
    """
    Get SCM mask using cache-dit's steps_mask().

    This is a thin wrapper that delegates to cache-dit's built-in
    steps_mask() function which handles all presets and scaling logic.

    Args:
        preset: Preset name ("none", "slow", "medium", "fast", "ultra").
        compute_bins: Custom compute bins (overrides preset).
        cache_bins: Custom cache bins (overrides preset).

    Returns:
        SCM mask list (1=compute, 0=cache), or None if disabled.
    """
    if preset == "none" and not (compute_bins and cache_bins):
        return None

    # Use cache-dit's steps_mask() directly
    mask = steps_mask(
        compute_bins=compute_bins,
        cache_bins=cache_bins,
        total_steps=num_inference_steps,
        mask_policy=preset if preset != "none" else "medium",
    )

    compute_count = sum(mask)
    cache_count = len(mask) - compute_count
    logger.info(
        "SCM: generated mask with %d compute steps, %d cache steps (preset=%s)",
        compute_count,
        cache_count,
        preset,
    )

    return mask
```
**EN:** This block defines function `get_scm_mask`. Get SCM mask using cache-dit's steps_mask(). This is a thin wrapper that delegates to cache-dit's built-in steps_mask() function which handles all presets and scaling logic. Key calls include `steps_mask`, `sum`, `logger.info`, and `len`. The implementation branches on conditions. Parameters such as `preset`, `num_inference_steps`, `compute_bins`, and `cache_bins` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_scm_mask`。 它用于获取scm mask。 关键调用包括 `steps_mask`、`sum`、`logger.info` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `preset`、`num_inference_steps`、`compute_bins` 和 `cache_bins` 等参数驱动。

### Lines 182-199: `CacheDitConfig` class overview / `CacheDitConfig` 类概览
```python
class CacheDitConfig:
    """Configuration for cache-dit integration.

    Attributes:
        enabled: Whether to enable cache-dit acceleration.
        Fn_compute_blocks: Number of first blocks to always compute (DBCache F).
        Bn_compute_blocks: Number of last blocks to always compute (DBCache B).
        max_warmup_steps: Number of warmup steps before caching starts (DBCache W).
        residual_diff_threshold: Threshold for residual difference (DBCache R).
        max_continuous_cached_steps: Maximum consecutive cached steps (DBCache MC).
        enable_taylorseer: Whether to enable TaylorSeer calibrator.
        taylorseer_order: Order of Taylor expansion (1 or 2).
        num_inference_steps: Total number of inference steps (required for transformer-only mode).
        steps_computation_mask: Binary mask for step-level caching (1=compute, 0=cache).
            Generated by get_scm_mask() (wrapper around cache_dit.steps_mask()).
        steps_computation_policy: Caching policy for SCM ("dynamic" or "static").
    """
```
**EN:** This block defines class `CacheDitConfig`. Configuration for cache-dit integration. Attributes: enabled: Whether to enable cache-dit acceleration.
**CN:** 该代码块定义了类 `CacheDitConfig`。 它用于封装 cache dit config 相关行为。

### Lines 200-222: supporting statements / 辅助语句
```python
    enabled: bool = False
    Fn_compute_blocks: int = 1
    Bn_compute_blocks: int = 0
    # Use 4 as default warmup steps instead of 8 in cache-dit, thus making
    # DBCache work for few steps distilled models, e.g., Z-Image w/ 8-steps.
    max_warmup_steps: int = 4
    # Use a relatively higher residual diff threshold (namely, 0.24) as default
    # to allow more aggressive caching due to we have already applied max continuous
    # cached steps limit, otherwise, we should use a lower threshold here like 0.12.
    residual_diff_threshold: float = 0.24
    max_continuous_cached_steps: int = 3
    # TaylorSeer is not suitable for few steps distilled models, so, we choose
    # to disable it by default. Reference:
    # - From Reusing to Forecasting: Accelerating Diffusion Models with TaylorSeers,
    #   https://arxiv.org/pdf/2503.06923
    # - FoCa: Forecast then Calibrate: Feature Caching as ODE for Efficient
    #   Diffusion Transformers, https://arxiv.org/pdf/2508.16211
    enable_taylorseer: bool = False
    taylorseer_order: int = 1
    num_inference_steps: Optional[int] = None
    # SCM fields (generated by _maybe_enable_cache_dit from env configuration)
    steps_computation_mask: Optional[List[int]] = None
    steps_computation_policy: str = "dynamic"
```
**EN:** This block gathers supporting statements inside `CacheDitConfig`. It updates names such as `enabled`, `Fn_compute_blocks`, `Bn_compute_blocks`, `max_warmup_steps`, `residual_diff_threshold`, and `max_continuous_cached_steps`.
**CN:** 该代码块汇集了位于 `CacheDitConfig` 内部的辅助语句。 它会更新 `enabled`、`Fn_compute_blocks`、`Bn_compute_blocks`、`max_warmup_steps`、`residual_diff_threshold` 和 `max_continuous_cached_steps` 等名称。

### Lines 225-336: `enable_cache_on_transformer` implementation / `enable_cache_on_transformer` 实现
```python
def enable_cache_on_transformer(
    transformer: torch.nn.Module,
    config: CacheDitConfig,
    model_name: str = "transformer",
    sp_group: Optional[torch.distributed.ProcessGroup] = None,
    tp_group: Optional[torch.distributed.ProcessGroup] = None,
) -> torch.nn.Module:
    """Enable cache-dit on a transformer module, by wrapping the module with cache-dit

    This function enables cache-dit acceleration using the BlockAdapterRegister
    for pre-registered models

    Args:
        model_name: Name of the model for logging purposes.
        sp_group: Sequence parallel process group (for Ulysses/Ring).
        tp_group: Tensor parallel process group.

    """
    if not config.enabled:
        return transformer

    if config.num_inference_steps is None:
        raise ValueError(
            "num_inference_steps is required for transformer-only mode. "
            "Please provide it in CacheDitConfig."
        )

    # Check if the transformer is pre-registered in cache-dit
    if not BlockAdapterRegister.is_supported(transformer):
        transformer_cls_name = transformer.__class__.__name__
        raise ValueError(
            f"{transformer_cls_name} is not officially supported by cache-dit. "
            "Supported cache-dit DiT families include Flux, QwenImage, HunyuanDiT, "
            "HunyuanVideo, Wan, CogVideoX, Mochi, and others. "
            "Please ensure your transformer belongs to one of these families or "
            "define a custom BlockAdapter."
        )

    # Build cache config (including SCM fields if provided)
    cache_config = DBCacheConfig(
        num_inference_steps=config.num_inference_steps,
        Fn_compute_blocks=config.Fn_compute_blocks,
        Bn_compute_blocks=config.Bn_compute_blocks,
        max_warmup_steps=config.max_warmup_steps,
        residual_diff_threshold=config.residual_diff_threshold,
        max_continuous_cached_steps=config.max_continuous_cached_steps,
        # SCM fields
        steps_computation_mask=config.steps_computation_mask,
        steps_computation_policy=config.steps_computation_policy,
    )

    # Build calibrator config if TaylorSeer is enabled
    calibrator_config = None
    if config.enable_taylorseer:
        calibrator_config = TaylorSeerCalibratorConfig(
            taylorseer_order=config.taylorseer_order,
        )

    # Enable cache-dit on the transformer
    logger.info(
        "Enabling cache-dit on %s with config: Fn=%d, Bn=%d, W=%d, R=%.2f, MC=%d, "
        "TaylorSeer=%s (order=%d), steps=%d",
        model_name,
        config.Fn_compute_blocks,
        config.Bn_compute_blocks,
        config.max_warmup_steps,
        config.residual_diff_threshold,
        config.max_continuous_cached_steps,
        config.enable_taylorseer,
        config.taylorseer_order,
        config.num_inference_steps,
    )

    # Log SCM configuration if enabled
    if config.steps_computation_mask:
        compute_steps = sum(config.steps_computation_mask)
        cache_steps = len(config.steps_computation_mask) - compute_steps
        logger.info(
            "SCM enabled: %d compute steps, %d cache steps, policy=%s",
            compute_steps,
            cache_steps,
            config.steps_computation_policy,
        )

    parallelism_config = _build_parallelism_config(sp_group, tp_group)
    if parallelism_config is not None:
        _patch_cache_dit_similarity()

    _mark_transformer_parallelized(transformer, parallelism_config, sp_group, tp_group)

    cache_dit.enable_cache(
        transformer,
        cache_config=cache_config,
        calibrator_config=calibrator_config,
        parallelism_config=None,
    )

    if parallelism_config is not None:
        context_manager = getattr(transformer, "_context_manager", None)
        if context_manager is not None:
            context_manager._sglang_sp_group = sp_group
            context_manager._sglang_tp_group = tp_group
            # In mixed TP + SP (Ulysses/Ring) mode, cache-dit decisions must be consistent
            # across the full TP×SP model-parallel slice. Prefer using SGLang's DIT group
            # as a conservative superset group; fallback to None.
            tp_sp_group = None
            if sp_group is not None and tp_group is not None:
                tp_sp_group = get_dit_group()

            context_manager._sglang_tp_sp_group = tp_sp_group

    return transformer
```
**EN:** This block defines function `enable_cache_on_transformer`. Enable cache-dit on a transformer module, by wrapping the module with cache-dit This function enables cache-dit acceleration using the BlockAdapterRegister for pre-registered models Args: model_name: Name of the model for logging purposes. sp_group: Sequence parallel process group (for Ulysses/Ring). Key calls include `DBCacheConfig`, `logger.info`, `_build_parallelism_config`, `_mark_transformer_parallelized`, and `cache_dit.enable_cache`. The implementation branches on conditions. Parameters such as `transformer`, `config`, `model_name`, `sp_group`, and `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `enable_cache_on_transformer`。 它用于处理 enable cache on transformer 相关逻辑。 关键调用包括 `DBCacheConfig`、`logger.info`、`_build_parallelism_config`、`_mark_transformer_parallelized` 和 `cache_dit.enable_cache`。 实现中包含条件分支。 本段逻辑主要由 `transformer`、`config`、`model_name`、`sp_group` 和 `tp_group` 等参数驱动。

### Lines 339-524: `enable_cache_on_dual_transformer` implementation / `enable_cache_on_dual_transformer` 实现
```python
def enable_cache_on_dual_transformer(
    transformer: torch.nn.Module,
    transformer_2: torch.nn.Module,
    primary_config: CacheDitConfig,
    secondary_config: CacheDitConfig,
    model_name: str = "wan2.2",
    sp_group: Optional[torch.distributed.ProcessGroup] = None,
    tp_group: Optional[torch.distributed.ProcessGroup] = None,
) -> tuple[torch.nn.Module, torch.nn.Module]:
    """Enable cache-dit on dual transformers using BlockAdapter.

    For models with two transformers (high-noise expert and low-noise expert),
    cache-dit requires enabling cache on both simultaneously via BlockAdapter.
    This cannot be done by calling enable_cache separately on each transformer.

    Args:
        primary_config: CacheDitConfig for primary transformer.
        secondary_config: CacheDitConfig for secondary transformer.
        sp_group: Sequence parallel process group (for Ulysses/Ring).
        tp_group: Tensor parallel process group.
    """
    _supported_dual_transformer_models = [
        "wan2.2",  # Currently, only Wan2.2 will run into dual-transformer case
    ]
    if model_name not in _supported_dual_transformer_models:
        raise ValueError(
            f"Dual-transformer cache-dit is only supported for "
            f"{_supported_dual_transformer_models}, got {model_name}."
        )

    if not primary_config.enabled:
        return transformer, transformer_2

    if primary_config.num_inference_steps is None:
        raise ValueError(
            "num_inference_steps is required for dual-transformer mode. "
            "Please provide it in CacheDitConfig."
        )

    # Build DBCacheConfig for primary transformer
    primary_cache_config = DBCacheConfig(
        num_inference_steps=primary_config.num_inference_steps,
        Fn_compute_blocks=primary_config.Fn_compute_blocks,
        Bn_compute_blocks=primary_config.Bn_compute_blocks,
        max_warmup_steps=primary_config.max_warmup_steps,
        residual_diff_threshold=primary_config.residual_diff_threshold,
        max_continuous_cached_steps=primary_config.max_continuous_cached_steps,
        steps_computation_mask=primary_config.steps_computation_mask,
        steps_computation_policy=primary_config.steps_computation_policy,
    )

    # Build DBCacheConfig for secondary transformer
    secondary_cache_config = DBCacheConfig(
        num_inference_steps=secondary_config.num_inference_steps,
        Fn_compute_blocks=secondary_config.Fn_compute_blocks,
        Bn_compute_blocks=secondary_config.Bn_compute_blocks,
        max_warmup_steps=secondary_config.max_warmup_steps,
        residual_diff_threshold=secondary_config.residual_diff_threshold,
        max_continuous_cached_steps=secondary_config.max_continuous_cached_steps,
        steps_computation_mask=secondary_config.steps_computation_mask,
        steps_computation_policy=secondary_config.steps_computation_policy,
    )

    # Build calibrator configs if TaylorSeer is enabled
    primary_calibrator = None
    if primary_config.enable_taylorseer:
        primary_calibrator = TaylorSeerCalibratorConfig(
            taylorseer_order=primary_config.taylorseer_order,
        )

    secondary_calibrator = None
    if secondary_config.enable_taylorseer:
        secondary_calibrator = TaylorSeerCalibratorConfig(
            taylorseer_order=secondary_config.taylorseer_order,
        )

    # Build ParamsModifier for each transformer
    primary_modifier = ParamsModifier(
        cache_config=primary_cache_config,
        calibrator_config=primary_calibrator,
    )
    secondary_modifier = ParamsModifier(
        cache_config=secondary_cache_config,
        calibrator_config=secondary_calibrator,
    )

    # Log configuration
    logger.info(
        "Enabling cache-dit on %s dual transformers with BlockAdapter",
        model_name,
    )
    logger.info(
        "  Primary (transformer): Fn=%d, Bn=%d, W=%d, R=%.2f, MC=%d, TaylorSeer=%s",
        primary_config.Fn_compute_blocks,
        primary_config.Bn_compute_blocks,
        primary_config.max_warmup_steps,
        primary_config.residual_diff_threshold,
        primary_config.max_continuous_cached_steps,
        primary_config.enable_taylorseer,
    )
    logger.info(
        "  Secondary (transformer_2): Fn=%d, Bn=%d, W=%d, R=%.2f, MC=%d, TaylorSeer=%s",
        secondary_config.Fn_compute_blocks,
        secondary_config.Bn_compute_blocks,
        secondary_config.max_warmup_steps,
        secondary_config.residual_diff_threshold,
        secondary_config.max_continuous_cached_steps,
        secondary_config.enable_taylorseer,
    )

    # Log SCM configuration if enabled
    if primary_config.steps_computation_mask:
        compute_steps = sum(primary_config.steps_computation_mask)
        cache_steps = len(primary_config.steps_computation_mask) - compute_steps
        logger.info(
            "  SCM enabled for primary transformer: %d compute steps, %d cache steps, policy=%s",
            compute_steps,
            cache_steps,
            primary_config.steps_computation_policy,
        )
    if secondary_config.steps_computation_mask:
        compute_steps = sum(secondary_config.steps_computation_mask)
        cache_steps = len(secondary_config.steps_computation_mask) - compute_steps
        logger.info(
            "  SCM enabled for secondary transformer: %d compute steps, %d cache steps, policy=%s",
            compute_steps,
            cache_steps,
            secondary_config.steps_computation_policy,
        )

    parallelism_config = _build_parallelism_config(sp_group, tp_group)
    if parallelism_config is not None:
        _patch_cache_dit_similarity()

    _mark_transformer_parallelized(transformer, parallelism_config, sp_group, tp_group)
    _mark_transformer_parallelized(
        transformer_2, parallelism_config, sp_group, tp_group
    )

    # Get blocks attribute - Wan transformers use 'blocks' attribute
    transformer_blocks = getattr(transformer, "blocks", None)
    transformer_2_blocks = getattr(transformer_2, "blocks", None)

    if transformer_blocks is None or transformer_2_blocks is None:
        raise ValueError(
            "Dual transformers must have 'blocks' attribute for cache-dit. "
            f"transformer has blocks: {transformer_blocks is not None}, "
            f"transformer_2 has blocks: {transformer_2_blocks is not None}"
        )

    # Enable cache-dit using BlockAdapter for both transformers simultaneously
    # This is required for Wan2.2 and similar dual-transformer architectures
    if model_name == "wan2.2":
        # Use Pattern_2 for Wan2.2 dual-transformer. We should check `model_name`
        # to ensure we only apply this for supported models. Different models
        # may require different ForwardPattern.
        cache_dit.enable_cache(
            BlockAdapter(
                transformer=[transformer, transformer_2],
                blocks=[transformer_blocks, transformer_2_blocks],
                forward_pattern=[ForwardPattern.Pattern_2, ForwardPattern.Pattern_2],
                params_modifiers=[primary_modifier, secondary_modifier],
                has_separate_cfg=True,
            ),
            parallelism_config=None,
        )
    else:
        raise ValueError(
            f"Dual-transformer is not implemented for model {model_name} yet."
        )

    if parallelism_config is not None:
        for t in [transformer, transformer_2]:
            context_manager = getattr(t, "_context_manager", None)
            if context_manager is not None:
                context_manager._sglang_sp_group = sp_group
                context_manager._sglang_tp_group = tp_group
                tp_sp_group = None
                if sp_group is not None and tp_group is not None:
                    try:
                        tp_sp_group = get_dit_group()
                    except Exception:
                        tp_sp_group = None
                context_manager._sglang_tp_sp_group = tp_sp_group

    return transformer, transformer_2
```
**EN:** This block defines function `enable_cache_on_dual_transformer`. Enable cache-dit on dual transformers using BlockAdapter. For models with two transformers (high-noise expert and low-noise expert), cache-dit requires enabling cache on both simultaneously via BlockAdapter. Key calls include `DBCacheConfig`, `ParamsModifier`, `logger.info`, `_build_parallelism_config`, and `_mark_transformer_parallelized`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `transformer`, `transformer_2`, `primary_config`, `secondary_config`, and `model_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `enable_cache_on_dual_transformer`。 它用于处理 enable cache on dual transformer 相关逻辑。 关键调用包括 `DBCacheConfig`、`ParamsModifier`、`logger.info`、`_build_parallelism_config` 和 `_mark_transformer_parallelized`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `transformer`、`transformer_2`、`primary_config`、`secondary_config` 和 `model_name` 等参数驱动。

### Lines 527-548: `refresh_context_on_transformer` implementation / `refresh_context_on_transformer` 实现
```python
def refresh_context_on_transformer(
    transformer: torch.nn.Module,
    num_inference_steps: int,
    scm_preset: str | None = None,
    verbose: bool = False,
) -> None:
    """Refresh cache-dit context for transformer."""
    steps_computation_mask = None
    if scm_preset is not None:
        steps_computation_mask = cache_dit.steps_mask(
            mask_policy=scm_preset, total_steps=num_inference_steps
        )
    cache_dit.refresh_context(
        transformer,
        cache_config=DBCacheConfig().reset(
            num_inference_steps=num_inference_steps,
            steps_computation_mask=steps_computation_mask,
            steps_computation_policy=scm_preset,
        ),
        verbose=verbose,
    )
    logger.debug(f"cache-dit refreshed on transformer (steps={num_inference_steps})")
```
**EN:** This block defines function `refresh_context_on_transformer`. Refresh cache-dit context for transformer. Key calls include `cache_dit.refresh_context`, `logger.debug`, `cache_dit.steps_mask`, `DBCacheConfig.reset`, and `DBCacheConfig`. The implementation branches on conditions. Parameters such as `transformer`, `num_inference_steps`, `scm_preset`, and `verbose` drive the behavior in this section.
**CN:** 该代码块定义了函数 `refresh_context_on_transformer`。 它用于处理 refresh context on transformer 相关逻辑。 关键调用包括 `cache_dit.refresh_context`、`logger.debug`、`cache_dit.steps_mask`、`DBCacheConfig.reset` 和 `DBCacheConfig`。 实现中包含条件分支。 本段逻辑主要由 `transformer`、`num_inference_steps`、`scm_preset` 和 `verbose` 等参数驱动。

### Lines 551-589: `refresh_context_on_dual_transformer` implementation / `refresh_context_on_dual_transformer` 实现
```python
def refresh_context_on_dual_transformer(
    transformer: torch.nn.Module,
    transformer_2: torch.nn.Module,
    num_high_noise_steps: int,
    num_low_noise_steps: int,
    scm_preset: str | None = None,
    verbose: bool = False,
) -> None:
    """Refresh cache-dit context for dual transformers."""
    high_noise_steps_computation_mask = None
    low_noise_steps_computation_mask = None
    if scm_preset is not None:
        high_noise_steps_computation_mask = cache_dit.steps_mask(
            mask_policy=scm_preset, total_steps=num_high_noise_steps
        )
        low_noise_steps_computation_mask = cache_dit.steps_mask(
            mask_policy=scm_preset, total_steps=num_low_noise_steps
        )
    cache_dit.refresh_context(
        transformer,
        cache_config=DBCacheConfig().reset(
            num_inference_steps=num_high_noise_steps,
            steps_computation_mask=high_noise_steps_computation_mask,
            steps_computation_policy=scm_preset,
        ),
        verbose=verbose,
    )
    cache_dit.refresh_context(
        transformer_2,
        cache_config=DBCacheConfig().reset(
            num_inference_steps=num_low_noise_steps,
            steps_computation_mask=low_noise_steps_computation_mask,
            steps_computation_policy=scm_preset,
        ),
        verbose=verbose,
    )
    logger.debug(
        f"cache-dit refreshed on dual transformers (steps={num_high_noise_steps}, {num_low_noise_steps})"
    )
```
**EN:** This block defines function `refresh_context_on_dual_transformer`. Refresh cache-dit context for dual transformers. Key calls include `cache_dit.refresh_context`, `logger.debug`, `cache_dit.steps_mask`, `DBCacheConfig.reset`, and `DBCacheConfig`. The implementation branches on conditions. Parameters such as `transformer`, `transformer_2`, `num_high_noise_steps`, `num_low_noise_steps`, and `scm_preset` drive the behavior in this section.
**CN:** 该代码块定义了函数 `refresh_context_on_dual_transformer`。 它用于处理 refresh context on dual transformer 相关逻辑。 关键调用包括 `cache_dit.refresh_context`、`logger.debug`、`cache_dit.steps_mask`、`DBCacheConfig.reset` 和 `DBCacheConfig`。 实现中包含条件分支。 本段逻辑主要由 `transformer`、`transformer_2`、`num_high_noise_steps`、`num_low_noise_steps` 和 `scm_preset` 等参数驱动。

## Key Concepts / 关键概念
- `_patch_cache_dit_similarity`: Top-level function that handles patch cache dit similarity logic. / 顶层函数，用于处理 patch cache dit similarity 相关逻辑。
- `_build_parallelism_config`: Top-level function that builds parallelism config. / 顶层函数，用于构建parallelism config。
- `_mark_transformer_parallelized`: Top-level function that handles mark transformer parallelized logic. / 顶层函数，用于处理 mark transformer parallelized 相关逻辑。
- `get_scm_mask`: Get SCM mask using cache-dit's steps_mask(). / 顶层函数，用于获取scm mask。
- `CacheDitConfig`: Configuration for cache-dit integration. / 核心类，用于封装 cache dit config 相关行为。
- `enable_cache_on_transformer`: Enable cache-dit on a transformer module, by wrapping the module with cache-dit This function enables cache-dit acceleration using the BlockAdapterRegister for pre-registered models Args: model_name: Name of the model for logging purposes. / 顶层函数，用于处理 enable cache on transformer 相关逻辑。
- `enable_cache_on_dual_transformer`: Enable cache-dit on dual transformers using BlockAdapter. / 顶层函数，用于处理 enable cache on dual transformer 相关逻辑。
- `refresh_context_on_transformer`: Refresh cache-dit context for transformer. / 顶层函数，用于处理 refresh context on transformer 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `cache_dit`, `cache_dit.caching.block_adapters`, `cache_dit.parallelism`, `cache_dit.caching.cache_contexts`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 589
