# rl_rollout.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/post_training/rl_rollout.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `RLRolloutArgs`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: CLI- and API-facing configuration for diffusion post-training / rollout paths. / 该文件属于配置层。它围绕 `RLRolloutArgs` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 3-12: module setup and imports / 模块初始化与导入
```python
"""CLI- and API-facing configuration for diffusion post-training / rollout paths."""

from __future__ import annotations

import argparse
import math
from dataclasses import dataclass
from typing import Any, Callable

from sglang.multimodal_gen.utils import StoreBoolean
```
**EN:** This block establishes the module context and imports `__future__`, `argparse`, `math`, `dataclasses`, `typing`, and `sglang.multimodal_gen.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`argparse`、`math`、`dataclasses`、`typing` 和 `sglang.multimodal_gen.utils`。这些依赖为后续实现提供所需符号。

### Lines 14-14: supporting statements / 辅助语句
```python
_VALID_ROLLOUT_SDE_TYPES = ("sde", "cps", "ode")
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_VALID_ROLLOUT_SDE_TYPES`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_VALID_ROLLOUT_SDE_TYPES` 等名称。

### Lines 18-20: `RLRolloutArgs` class overview / `RLRolloutArgs` 类概览
```python
class RLRolloutArgs:
    """Rollout (log-prob trajectory) options used by SamplingParams and APIs."""
```
**EN:** This block defines class `RLRolloutArgs`. Rollout (log-prob trajectory) options used by SamplingParams and APIs.
**CN:** 该代码块定义了类 `RLRolloutArgs`。 它用于封装 rlrollout args 相关行为。

### Lines 21-25: supporting statements / 辅助语句
```python
    rollout: bool = False
    rollout_sde_type: str = "sde"
    rollout_noise_level: float = 0.7
    rollout_log_prob_no_const: bool = False
    rollout_debug_mode: bool = False
```
**EN:** This block gathers supporting statements inside `RLRolloutArgs`. It updates names such as `rollout`, `rollout_sde_type`, `rollout_noise_level`, `rollout_log_prob_no_const`, and `rollout_debug_mode`.
**CN:** 该代码块汇集了位于 `RLRolloutArgs` 内部的辅助语句。 它会更新 `rollout`、`rollout_sde_type`、`rollout_noise_level`、`rollout_log_prob_no_const` 和 `rollout_debug_mode` 等名称。

### Lines 27-40: `validate` implementation / `validate` 实现
```python
    def validate(self) -> None:
        noise = self.rollout_noise_level
        if isinstance(noise, bool) or not isinstance(noise, (int, float)):
            raise ValueError(f"rollout_noise_level must be a number, got {noise!r}")
        if not math.isfinite(float(noise)):
            raise ValueError(f"rollout_noise_level must be finite, got {noise!r}")
        if float(noise) < 0.0:
            raise ValueError(f"rollout_noise_level must be non-negative, got {noise!r}")

        if self.rollout_sde_type not in _VALID_ROLLOUT_SDE_TYPES:
            raise ValueError(
                f"rollout_sde_type must be one of {_VALID_ROLLOUT_SDE_TYPES}, "
                f"got {self.rollout_sde_type!r}"
            )
```
**EN:** This block defines method `validate` on `RLRolloutArgs`. It validates function. Key calls include `isinstance`, `ValueError`, `math.isfinite`, and `float`. The implementation branches on conditions.
**CN:** 该代码块定义了 `RLRolloutArgs` 的方法 `validate`。 它用于校验函数。 关键调用包括 `isinstance`、`ValueError`、`math.isfinite` 和 `float`。 实现中包含条件分支。

### Lines 42-55: `validate_sampling_params` implementation / `validate_sampling_params` 实现
```python
    @classmethod
    def validate_sampling_params(cls, params: Any) -> None:
        """Validate rollout fields on a duck-typed object (e.g. ``SamplingParams``).

        Mirrors how ``ServerArgs`` runs ``NunchakuSVDQuantArgs.validate()`` from
        ``_adjust_quant_config`` instead of inlining checks in a large validator.
        """
        cls(
            rollout=params.rollout,
            rollout_sde_type=params.rollout_sde_type,
            rollout_noise_level=params.rollout_noise_level,
            rollout_log_prob_no_const=params.rollout_log_prob_no_const,
            rollout_debug_mode=params.rollout_debug_mode,
        ).validate()
```
**EN:** This block defines method `validate_sampling_params` on `RLRolloutArgs`. Validate rollout fields on a duck-typed object (e.g. ``SamplingParams``). Key calls include `cls.validate`, and `cls`. Parameters such as `params` drive the behavior in this section.
**CN:** 该代码块定义了 `RLRolloutArgs` 的方法 `validate_sampling_params`。 它用于校验sampling params。 关键调用包括 `cls.validate` 和 `cls`。 本段逻辑主要由 `params` 等参数驱动。

### Lines 57-105: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(
        parser: Any,
        add_argument: Callable[..., Any] | None = None,
    ) -> None:
        """Register rollout-related CLI flags on ``parser``.

        If ``add_argument`` is provided (e.g. SamplingParams' wrapper with
        ``default=argparse.SUPPRESS``), it is used; otherwise a local wrapper
        is applied.
        """

        if add_argument is None:

            def _add(*name_or_flags: Any, **kwargs: Any):
                kwargs.setdefault("default", argparse.SUPPRESS)
                return parser.add_argument(*name_or_flags, **kwargs)

            add_argument = _add

        add_argument(
            "--rollout",
            action="store_true",
            help="Enable rollout mode and return per-step log_prob trajectory",
        )
        add_argument(
            "--rollout-sde-type",
            type=str,
            choices=list(_VALID_ROLLOUT_SDE_TYPES),
            help="Rollout step objective type used in log-prob computation.",
        )
        add_argument(
            "--rollout-noise-level",
            type=float,
            help="Noise level used by rollout SDE/CPS step objective.",
        )
        add_argument(
            "--rollout-log-prob-no-const",
            action=StoreBoolean,
            help="If true, return rollout log-prob without constant terms.",
        )
        add_argument(
            "--rollout-debug-mode",
            action=StoreBoolean,
            help=(
                "If true, return rollout debug tensors "
                "(variance noise, mean, std, model output)."
            ),
        )
```
**EN:** This block defines method `add_cli_args` on `RLRolloutArgs`. Register rollout-related CLI flags on ``parser``. If ``add_argument`` is provided (e.g. Key calls include `add_argument`, `kwargs.setdefault`, `parser.add_argument`, and `list`. The implementation branches on conditions. Parameters such as `parser`, and `add_argument` drive the behavior in this section.
**CN:** 该代码块定义了 `RLRolloutArgs` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `add_argument`、`kwargs.setdefault`、`parser.add_argument` 和 `list`。 实现中包含条件分支。 本段逻辑主要由 `parser` 和 `add_argument` 等参数驱动。

### Lines 107-121: `from_dict` implementation / `from_dict` 实现
```python
    @classmethod
    def from_dict(cls, kwargs: dict[str, Any]) -> RLRolloutArgs:
        return cls(
            rollout=bool(kwargs.get("rollout", cls.rollout)),
            rollout_sde_type=str(kwargs.get("rollout_sde_type", cls.rollout_sde_type)),
            rollout_noise_level=float(
                kwargs.get("rollout_noise_level", cls.rollout_noise_level)
            ),
            rollout_log_prob_no_const=bool(
                kwargs.get("rollout_log_prob_no_const", cls.rollout_log_prob_no_const)
            ),
            rollout_debug_mode=bool(
                kwargs.get("rollout_debug_mode", cls.rollout_debug_mode)
            ),
        )
```
**EN:** This block defines method `from_dict` on `RLRolloutArgs`. It constructs from dict. Key calls include `cls`, `bool`, `str`, `float`, and `kwargs.get`. Parameters such as `kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `RLRolloutArgs` 的方法 `from_dict`。 它用于从…构造dict。 关键调用包括 `cls`、`bool`、`str`、`float` 和 `kwargs.get`。 本段逻辑主要由 `kwargs` 等参数驱动。

## Key Concepts / 关键概念
- `RLRolloutArgs`: Rollout (log-prob trajectory) options used by SamplingParams and APIs. / 核心类，用于封装 rlrollout args 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `argparse`, `math`, `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 121
