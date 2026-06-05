# nunchaku.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/quantization/nunchaku.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `NunchakuArgsResolution`, and `NunchakuSVDQuantArgs`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `NunchakuArgsResolution` 和 `NunchakuSVDQuantArgs` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-18: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

import os
import re
from dataclasses import dataclass, replace
from typing import Any

import torch

from sglang.multimodal_gen.runtime.layers.quantization.configs.nunchaku_config import (
    NunchakuConfig,
    is_nunchaku_available,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import StoreBoolean
```
**EN:** This block establishes the module context and imports `__future__`, `os`, `re`, `dataclasses`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`os`、`re`、`dataclasses`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 20-20: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 24-26: `NunchakuArgsResolution` class overview / `NunchakuArgsResolution` 类概览
```python
class NunchakuArgsResolution:
    """Normalized runtime settings derived from Nunchaku CLI-facing args."""
```
**EN:** This block defines class `NunchakuArgsResolution`. Normalized runtime settings derived from Nunchaku CLI-facing args.
**CN:** 该代码块定义了类 `NunchakuArgsResolution`。 它用于封装 nunchaku args resolution 相关行为。

### Lines 27-28: supporting statements / 辅助语句
```python
    transformer_weights_path: str | None = None
    nunchaku_config: NunchakuConfig | None = None
```
**EN:** This block gathers supporting statements inside `NunchakuArgsResolution`. It updates names such as `transformer_weights_path`, and `nunchaku_config`.
**CN:** 该代码块汇集了位于 `NunchakuArgsResolution` 内部的辅助语句。 它会更新 `transformer_weights_path` 和 `nunchaku_config` 等名称。

### Lines 32-38: `NunchakuSVDQuantArgs` class overview / `NunchakuSVDQuantArgs` 类概览
```python
class NunchakuSVDQuantArgs:
    """CLI-facing configuration for Nunchaku (SVDQuant) inference.

    This is intentionally lightweight and only contains arguments needed to
    construct `runtime.layers.quantization.nunchaku_config.NunchakuConfig`.
    """
```
**EN:** This block defines class `NunchakuSVDQuantArgs`. CLI-facing configuration for Nunchaku (SVDQuant) inference. This is intentionally lightweight and only contains arguments needed to construct `runtime.layers.quantization.nunchaku_config.NunchakuConfig`.
**CN:** 该代码块定义了类 `NunchakuSVDQuantArgs`。 它用于封装 nunchaku svdquant args 相关行为。

### Lines 39-43: supporting statements / 辅助语句
```python
    enable_svdquant: bool = False
    transformer_weights_path: str | None = None
    quantization_precision: str | None = None  # "int4" or "nvfp4"
    quantization_rank: int | None = None
    quantization_act_unsigned: bool = False
```
**EN:** This block gathers supporting statements inside `NunchakuSVDQuantArgs`. It updates names such as `enable_svdquant`, `transformer_weights_path`, `quantization_precision`, `quantization_rank`, and `quantization_act_unsigned`.
**CN:** 该代码块汇集了位于 `NunchakuSVDQuantArgs` 内部的辅助语句。 它会更新 `enable_svdquant`、`transformer_weights_path`、`quantization_precision`、`quantization_rank` 和 `quantization_act_unsigned` 等名称。

### Lines 45-70: `_infer_from_weights_path` implementation / `_infer_from_weights_path` 实现
```python
    def _infer_from_weights_path(self) -> tuple[bool, str | None, int | None]:
        """Infer whether SVDQuant is enabled and parse precision/rank from filename."""
        inferred_precision = None
        inferred_rank = None
        enable_svdquant = self.enable_svdquant

        if not self.transformer_weights_path:
            return enable_svdquant, inferred_precision, inferred_rank

        filename = os.path.basename(self.transformer_weights_path)
        if not enable_svdquant and re.search(r"svdq-(int4|fp4)_r(\d+)", filename):
            enable_svdquant = True

        if not enable_svdquant:
            return enable_svdquant, inferred_precision, inferred_rank

        # Expected pattern: svdq-{precision}_r{rank}-...
        # e.g., svdq-int4_r32-qwen-image.safetensors
        match = re.search(r"svdq-(int4|fp4)_r(\d+)", filename)

        if match:
            p_str, r_str = match.groups()
            inferred_precision = "nvfp4" if p_str == "fp4" else "int4"
            inferred_rank = int(r_str)

        return enable_svdquant, inferred_precision, inferred_rank
```
**EN:** This block defines method `_infer_from_weights_path` on `NunchakuSVDQuantArgs`. Infer whether SVDQuant is enabled and parse precision/rank from filename. Key calls include `os.path.basename`, `re.search`, `match.groups`, and `int`. The implementation branches on conditions.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `_infer_from_weights_path`。 它用于推断from weights path。 关键调用包括 `os.path.basename`、`re.search`、`match.groups` 和 `int`。 实现中包含条件分支。

### Lines 72-99: `_normalized` implementation / `_normalized` 实现
```python
    def _normalized(self) -> "NunchakuSVDQuantArgs":
        enable_svdquant, inferred_precision, inferred_rank = (
            self._infer_from_weights_path()
        )
        normalized = replace(
            self,
            enable_svdquant=enable_svdquant,
            quantization_precision=(
                self.quantization_precision or inferred_precision or "int4"
            ),
            quantization_rank=self.quantization_rank or inferred_rank or 32,
        )

        if self.quantization_precision is None and inferred_precision:
            if inferred_precision:
                logger.info(
                    f"inferred --quantization-precision: {normalized.quantization_precision} "
                    f"from --transformer-weights-path: {self.transformer_weights_path}"
                )

        if self.quantization_rank is None and inferred_rank:
            if inferred_rank:
                logger.info(
                    f"inferred --quantization-rank: {normalized.quantization_rank} "
                    f"from --transformer-weights-path: {self.transformer_weights_path}"
                )

        return normalized
```
**EN:** This block defines method `_normalized` on `NunchakuSVDQuantArgs`. It handles normalized logic. Key calls include `self._infer_from_weights_path`, `replace`, and `logger.info`. The implementation branches on conditions.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `_normalized`。 它用于处理 normalized 相关逻辑。 关键调用包括 `self._infer_from_weights_path`、`replace` 和 `logger.info`。 实现中包含条件分支。

### Lines 101-148: `_validate` implementation / `_validate` 实现
```python
    def _validate(self) -> None:
        # TODO: warn if the served model doesn't support nunchaku
        if not self.enable_svdquant:
            return

        if not current_platform.is_cuda():
            raise ValueError(
                "Nunchaku SVDQuant is only supported on NVIDIA CUDA GPUs "
                "(Ampere SM8x or SM12x)."
            )

        device_count = torch.cuda.device_count()

        unsupported: list[str] = []
        for i in range(device_count):
            major, minor = torch.cuda.get_device_capability(i)
            if major == 9:
                unsupported.append(f"cuda:{i} (SM{major}{minor}, Hopper)")
            elif major not in (8, 12):
                unsupported.append(f"cuda:{i} (SM{major}{minor})")

        if unsupported:
            raise ValueError(
                "Nunchaku SVDQuant is currently only supported on Ampere (SM8x) or SM12x GPUs; "
                f"Unsupported devices: {', '.join(unsupported)}. "
                "Disable it with --enable-svdquant false."
            )

        if not self.transformer_weights_path:
            raise ValueError(
                "--enable-svdquant requires --transformer-weights-path to be set"
            )

        if not is_nunchaku_available():
            raise ValueError(
                "Nunchaku is enabled, but not installed. Please refer to https://nunchaku.tech/docs/nunchaku/installation/installation.html for detailed installation methods."
            )

        if self.quantization_precision not in ("int4", "nvfp4"):
            raise ValueError(
                f"Invalid --quantization-precision: {self.quantization_precision}. "
                "Must be one of: int4, nvfp4"
            )

        if self.quantization_rank <= 0:
            raise ValueError(
                f"Invalid --quantization-rank: {self.quantization_rank}. Must be > 0"
            )
```
**EN:** This block defines method `_validate` on `NunchakuSVDQuantArgs`. It validates function. Key calls include `torch.cuda.device_count`, `range`, `current_platform.is_cuda`, `ValueError`, and `torch.cuda.get_device_capability`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `_validate`。 它用于校验函数。 关键调用包括 `torch.cuda.device_count`、`range`、`current_platform.is_cuda`、`ValueError` 和 `torch.cuda.get_device_capability`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 150-168: `resolve_runtime_config` implementation / `resolve_runtime_config` 实现
```python
    def resolve_runtime_config(self) -> NunchakuArgsResolution:
        normalized = self._normalized()
        normalized._validate()

        if not normalized.enable_svdquant or not normalized.transformer_weights_path:
            return NunchakuArgsResolution(
                transformer_weights_path=normalized.transformer_weights_path,
                nunchaku_config=None,
            )

        return NunchakuArgsResolution(
            transformer_weights_path=normalized.transformer_weights_path,
            nunchaku_config=NunchakuConfig(
                precision=normalized.quantization_precision,
                rank=normalized.quantization_rank,
                act_unsigned=normalized.quantization_act_unsigned,
                transformer_weights_path=normalized.transformer_weights_path,
            ),
        )
```
**EN:** This block defines method `resolve_runtime_config` on `NunchakuSVDQuantArgs`. It resolves runtime config. Key calls include `self._normalized`, `normalized._validate`, `NunchakuArgsResolution`, and `NunchakuConfig`. The implementation branches on conditions.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `resolve_runtime_config`。 它用于解析runtime config。 关键调用包括 `self._normalized`、`normalized._validate`、`NunchakuArgsResolution` 和 `NunchakuConfig`。 实现中包含条件分支。

### Lines 170-204: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser) -> None:
        parser.add_argument(
            "--enable-svdquant",
            action=StoreBoolean,
            default=NunchakuSVDQuantArgs.enable_svdquant,
            help="Enable Nunchaku SVDQuant (W4A4-style) inference.",
        )
        parser.add_argument(
            "--transformer-weights-path",
            type=str,
            default=NunchakuSVDQuantArgs.transformer_weights_path,
            help=(
                "Path to pre-quantized transformer weights. Can be a single .safetensors "
                "file, a directory, or a HuggingFace repo ID. Used by Nunchaku (SVDQuant) and quantized single-file checkpoints."
            ),
        )
        parser.add_argument(
            "--quantization-precision",
            type=str,
            default=None,
            help="Quantization precision: int4 or nvfp4. If not specified, inferred from model path or defaults to int4.",
        )
        parser.add_argument(
            "--quantization-rank",
            type=int,
            default=None,
            help="SVD low-rank dimension (e.g., 32). If not specified, inferred from model path or defaults to 32.",
        )
        parser.add_argument(
            "--quantization-act-unsigned",
            action=StoreBoolean,
            default=NunchakuSVDQuantArgs.quantization_act_unsigned,
            help="Use unsigned activation quantization (if supported).",
        )
```
**EN:** This block defines method `add_cli_args` on `NunchakuSVDQuantArgs`. It handles add cli args logic. Key calls include `parser.add_argument`. Parameters such as `parser` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument`。 本段逻辑主要由 `parser` 等参数驱动。

### Lines 206-222: `from_dict` implementation / `from_dict` 实现
```python
    @classmethod
    def from_dict(cls, kwargs: dict[str, Any]) -> "NunchakuSVDQuantArgs":
        # Map CLI/config keys to dataclass fields (keep backwards compatibility).
        path = (
            kwargs.get("transformer_weights_path")
            or kwargs.get("transformer_quantized_path")
            or kwargs.get("quantized_model_path")
        )
        return cls(
            enable_svdquant=bool(kwargs.get("enable_svdquant", cls.enable_svdquant)),
            transformer_weights_path=path,
            quantization_precision=kwargs.get("quantization_precision"),
            quantization_rank=kwargs.get("quantization_rank"),
            quantization_act_unsigned=bool(
                kwargs.get("quantization_act_unsigned", cls.quantization_act_unsigned)
            ),
        )
```
**EN:** This block defines method `from_dict` on `NunchakuSVDQuantArgs`. It constructs from dict. Key calls include `cls`, `kwargs.get`, and `bool`. Parameters such as `kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuSVDQuantArgs` 的方法 `from_dict`。 它用于从…构造dict。 关键调用包括 `cls`、`kwargs.get` 和 `bool`。 本段逻辑主要由 `kwargs` 等参数驱动。

## Key Concepts / 关键概念
- `NunchakuArgsResolution`: Normalized runtime settings derived from Nunchaku CLI-facing args. / 核心类，用于封装 nunchaku args resolution 相关行为。
- `NunchakuSVDQuantArgs`: CLI-facing configuration for Nunchaku (SVDQuant) inference. / 核心类，用于封装 nunchaku svdquant args 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`, `re`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.quantization.configs.nunchaku_config`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 222
