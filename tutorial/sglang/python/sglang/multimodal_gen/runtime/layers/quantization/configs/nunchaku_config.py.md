# nunchaku_config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/quantization/configs/nunchaku_config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `NunchakuConfig`, `is_nunchaku_available`, and `_patch_native_svdq_linear`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `NunchakuConfig`、`is_nunchaku_available` 和 `_patch_native_svdq_linear` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-15: module setup and imports / 模块初始化与导入
```python
import json
import os
from dataclasses import dataclass
from functools import lru_cache
from typing import Any, Optional

import torch
from safetensors.torch import load_file as safetensors_load_file
from torch import nn

from sglang.multimodal_gen.runtime.layers.linear import LinearBase
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

from .base_config import QuantizationConfig, QuantizeMethodBase
```
**EN:** This block establishes the module context and imports `json`, `os`, `dataclasses`, `functools`, `typing`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`os`、`dataclasses`、`functools`、`typing` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 17-17: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 20-28: `is_nunchaku_available` implementation / `is_nunchaku_available` 实现
```python
@lru_cache(maxsize=1)
def is_nunchaku_available() -> bool:
    try:
        import nunchaku  # noqa

        logger.debug("Nunchaku package detected")
        return True
    except Exception:
        return False
```
**EN:** This block defines function `is_nunchaku_available`. It handles is nunchaku available logic. Key calls include `lru_cache`, and `logger.debug`. The implementation handles exceptional paths.
**CN:** 该代码块定义了函数 `is_nunchaku_available`。 它用于处理 is nunchaku available 相关逻辑。 关键调用包括 `lru_cache` 和 `logger.debug`。 实现中处理异常路径。

### Lines 32-46: `NunchakuConfig` class overview / `NunchakuConfig` 类概览
```python
class NunchakuConfig(QuantizationConfig):
    """
    Configuration for Nunchaku (SVDQuant) W4A4-style quantization.

    Attributes:
        precision: Quantization precision type. Options:
            - "int4": Standard INT4 quantization
            - "nvfp4": FP4 quantization
        rank: SVD low-rank dimension for absorbing outliers
        group_size: Quantization group size (automatically set based on precision)
        act_unsigned: Use unsigned activation quantization
        transformer_weights_path: Path to pre-quantized transformer weights (.safetensors)
        model_cls: DiT model class that provides quantization rules via get_nunchaku_quant_rules()
    """
```
**EN:** This block defines class `NunchakuConfig`. Configuration for Nunchaku (SVDQuant) W4A4-style quantization. Attributes: precision: Quantization precision type. It inherits from `QuantizationConfig`.
**CN:** 该代码块定义了类 `NunchakuConfig`。 它用于封装 nunchaku config 相关行为。 它继承自 `QuantizationConfig`。

### Lines 47-52: supporting statements / 辅助语句
```python
    precision: str = "int4"
    rank: int = 32
    group_size: Optional[int] = None
    act_unsigned: bool = False
    transformer_weights_path: Optional[str] = None
    model_cls: Optional[type] = None
```
**EN:** This block gathers supporting statements inside `NunchakuConfig`. It updates names such as `precision`, `rank`, `group_size`, `act_unsigned`, `transformer_weights_path`, and `model_cls`.
**CN:** 该代码块汇集了位于 `NunchakuConfig` 内部的辅助语句。 它会更新 `precision`、`rank`、`group_size`、`act_unsigned`、`transformer_weights_path` 和 `model_cls` 等名称。

### Lines 54-56: `get_name` implementation / `get_name` 实现
```python
    @classmethod
    def get_name(cls) -> str:
        return "svdquant"
```
**EN:** This block defines method `get_name` on `NunchakuConfig`. It retrieves name.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `get_name`。 它用于获取name。

### Lines 58-60: `get_supported_act_dtypes` implementation / `get_supported_act_dtypes` 实现
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> list[torch.dtype]:
        return [torch.bfloat16, torch.float16]
```
**EN:** This block defines method `get_supported_act_dtypes` on `NunchakuConfig`. It retrieves supported act dtypes.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `get_supported_act_dtypes`。 它用于获取supported act dtypes。

### Lines 62-64: `get_min_capability` implementation / `get_min_capability` 实现
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 70
```
**EN:** This block defines method `get_min_capability` on `NunchakuConfig`. It retrieves min capability.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `get_min_capability`。 它用于获取min capability。

### Lines 66-68: `get_config_filenames` implementation / `get_config_filenames` 实现
```python
    @staticmethod
    def get_config_filenames() -> list[str]:
        return ["quantization_config.json", "quant_config.json"]
```
**EN:** This block defines method `get_config_filenames` on `NunchakuConfig`. It retrieves config filenames.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `get_config_filenames`。 它用于获取config filenames。

### Lines 70-79: `from_config` implementation / `from_config` 实现
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "NunchakuConfig":

        return cls(
            precision=config.get("precision", "int4"),
            rank=int(config.get("rank", 32)),
            group_size=config.get("group_size"),
            act_unsigned=bool(config.get("act_unsigned", False)),
            transformer_weights_path=config.get("transformer_weights_path"),
        )
```
**EN:** This block defines method `from_config` on `NunchakuConfig`. It constructs from config. Key calls include `cls`, `config.get`, `int`, and `bool`. Parameters such as `config` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `from_config`。 它用于从…构造config。 关键调用包括 `cls`、`config.get`、`int` 和 `bool`。 本段逻辑主要由 `config` 等参数驱动。

### Lines 81-121: `get_quant_method` implementation / `get_quant_method` 实现
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        if not isinstance(layer, LinearBase):
            return None

        # get quantization rules from model class
        quant_rules = self._get_quant_rules()

        # priority: skip > awq_w4a16 > svdq_w4a4 > default
        skip_patterns = quant_rules.get("skip", [])
        for pattern in skip_patterns:
            if pattern in prefix.lower():
                return None

        awq_patterns = quant_rules.get("awq_w4a16", [])
        for pattern in awq_patterns:
            if pattern in prefix:
                from ..nunchaku_linear import NunchakuAWQLinearMethod

                return NunchakuAWQLinearMethod(group_size=64)

        svdq_patterns = quant_rules.get("svdq_w4a4", [])
        for pattern in svdq_patterns:
            if pattern in prefix:
                from ..nunchaku_linear import NunchakuSVDQLinearMethod

                return NunchakuSVDQLinearMethod(
                    precision=self.precision,
                    rank=self.rank,
                    act_unsigned=self.act_unsigned,
                )

        # default: apply svdq_w4a4 to all remaining linear layers
        from ..nunchaku_linear import NunchakuSVDQLinearMethod

        return NunchakuSVDQLinearMethod(
            precision=self.precision,
            rank=self.rank,
            act_unsigned=self.act_unsigned,
        )
```
**EN:** This block defines method `get_quant_method` on `NunchakuConfig`. It retrieves quant method. Key calls include `self._get_quant_rules`, `quant_rules.get`, `NunchakuSVDQLinearMethod`, `isinstance`, and `prefix.lower`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `layer`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `get_quant_method`。 它用于获取quant method。 关键调用包括 `self._get_quant_rules`、`quant_rules.get`、`NunchakuSVDQLinearMethod`、`isinstance` 和 `prefix.lower`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `layer` 和 `prefix` 等参数驱动。

### Lines 123-128: `_get_quant_rules` implementation / `_get_quant_rules` 实现
```python
    def _get_quant_rules(self) -> dict[str, list[str]]:
        if self.model_cls is not None and hasattr(
            self.model_cls, "get_nunchaku_quant_rules"
        ):
            return self.model_cls.get_nunchaku_quant_rules()
        return {}
```
**EN:** This block defines method `_get_quant_rules` on `NunchakuConfig`. It retrieves quant rules. Key calls include `hasattr`, and `self.model_cls.get_nunchaku_quant_rules`. The implementation branches on conditions.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `_get_quant_rules`。 它用于获取quant rules。 关键调用包括 `hasattr` 和 `self.model_cls.get_nunchaku_quant_rules`。 实现中包含条件分支。

### Lines 130-147: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        if self.group_size is None:
            if self.precision == "nvfp4":
                self.group_size = 16
            elif self.precision == "int4":
                self.group_size = 64
            else:
                raise ValueError(
                    f"Invalid precision: {self.precision}. Must be 'int4' or 'nvfp4'"
                )

        if self.precision not in ["int4", "nvfp4"]:
            raise ValueError(
                f"Invalid precision: {self.precision}. Must be 'int4' or 'nvfp4'"
            )

        if self.rank <= 0:
            raise ValueError(f"Rank must be positive, got {self.rank}")
```
**EN:** This block defines method `__post_init__` on `NunchakuConfig`. It post-processes init. Key calls include `ValueError`. The implementation branches on conditions.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `ValueError`。 实现中包含条件分支。

### Lines 149-152: `from_dict` implementation / `from_dict` 实现
```python
    @classmethod
    def from_dict(cls, config_dict: dict) -> "NunchakuConfig":
        """Create configuration from dictionary."""
        return cls(**config_dict)
```
**EN:** This block defines method `from_dict` on `NunchakuConfig`. Create configuration from dictionary. Key calls include `cls`. Parameters such as `config_dict` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `from_dict`。 它用于从…构造dict。 关键调用包括 `cls`。 本段逻辑主要由 `config_dict` 等参数驱动。

### Lines 154-162: `to_dict` implementation / `to_dict` 实现
```python
    def to_dict(self) -> dict:
        """Convert configuration to dictionary."""
        return {
            "precision": self.precision,
            "rank": self.rank,
            "group_size": self.group_size,
            "act_unsigned": self.act_unsigned,
            "transformer_weights_path": self.transformer_weights_path,
        }
```
**EN:** This block defines method `to_dict` on `NunchakuConfig`. Convert configuration to dictionary.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `to_dict`。 它用于转换为dict。

### Lines 164-173: `from_pretrained` implementation / `from_pretrained` 实现
```python
    @classmethod
    def from_pretrained(cls, model_path: str) -> Optional["NunchakuConfig"]:
        for filename in cls.get_config_filenames():
            config_path = os.path.join(model_path, filename)
            if os.path.exists(config_path):
                with open(config_path, "r") as f:
                    config_dict = json.load(f)
                if config_dict.get("quant_method") == cls.get_name():
                    return cls.from_config(config_dict)
        return None
```
**EN:** This block defines method `from_pretrained` on `NunchakuConfig`. It constructs from pretrained. Key calls include `cls.get_config_filenames`, `os.path.join`, `os.path.exists`, `open`, and `json.load`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `model_path` drive the behavior in this section.
**CN:** 该代码块定义了 `NunchakuConfig` 的方法 `from_pretrained`。 它用于从…构造pretrained。 关键调用包括 `cls.get_config_filenames`、`os.path.join`、`os.path.exists`、`open` 和 `json.load`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `model_path` 等参数驱动。

### Lines 176-185: `_patch_native_svdq_linear` implementation / `_patch_native_svdq_linear` 实现
```python
def _patch_native_svdq_linear(
    module: nn.Module, tensor: Any, svdq_linear_cls: type
) -> bool:
    if (
        isinstance(module, svdq_linear_cls)
        and getattr(module, "wtscale", None) is not None
    ):
        module.wtscale = tensor
        return True
    return False
```
**EN:** This block defines function `_patch_native_svdq_linear`. It handles patch native svdq linear logic. Key calls include `isinstance`, and `getattr`. The implementation branches on conditions. Parameters such as `module`, `tensor`, and `svdq_linear_cls` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patch_native_svdq_linear`。 它用于处理 patch native svdq linear 相关逻辑。 关键调用包括 `isinstance` 和 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `module`、`tensor` 和 `svdq_linear_cls` 等参数驱动。

### Lines 188-207: `_patch_sglang_svdq_linear` implementation / `_patch_sglang_svdq_linear` 实现
```python
def _patch_sglang_svdq_linear(
    module: nn.Module, tensor: Any, svdq_method_cls: type
) -> bool:
    quant_method = getattr(module, "quant_method", None)
    if not isinstance(quant_method, svdq_method_cls):
        return False

    existing = getattr(module, "wtscale", None)
    if isinstance(existing, nn.Parameter):
        with torch.no_grad():
            existing.data.copy_(tensor.to(existing.data.dtype))
    else:
        module.wtscale = tensor

    # Keep alpha in sync (kernel reads `layer._nunchaku_alpha`)
    try:
        module._nunchaku_alpha = float(tensor.detach().cpu().item())
    except Exception:
        module._nunchaku_alpha = None
    return True
```
**EN:** This block defines function `_patch_sglang_svdq_linear`. It handles patch sglang svdq linear logic. Key calls include `getattr`, `isinstance`, `float`, `torch.no_grad`, and `existing.data.copy_`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `module`, `tensor`, and `svdq_method_cls` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patch_sglang_svdq_linear`。 它用于处理 patch sglang svdq linear 相关逻辑。 关键调用包括 `getattr`、`isinstance`、`float`、`torch.no_grad` 和 `existing.data.copy_`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `module`、`tensor` 和 `svdq_method_cls` 等参数驱动。

### Lines 210-223: `_patch_sglang_svdq_wcscales` implementation / `_patch_sglang_svdq_wcscales` 实现
```python
def _patch_sglang_svdq_wcscales(
    module: nn.Module, tensor: Any, svdq_method_cls: type
) -> bool:
    quant_method = getattr(module, "quant_method", None)
    if not isinstance(quant_method, svdq_method_cls):
        return False

    existing = getattr(module, "wcscales", None)
    if isinstance(existing, nn.Parameter):
        with torch.no_grad():
            existing.data.copy_(tensor.to(existing.data.dtype))
    else:
        module.wcscales = tensor
    return True
```
**EN:** This block defines function `_patch_sglang_svdq_wcscales`. It handles patch sglang svdq wcscales logic. Key calls include `getattr`, `isinstance`, `torch.no_grad`, `existing.data.copy_`, and `tensor.to`. The implementation branches on conditions, uses context-managed resources. Parameters such as `module`, `tensor`, and `svdq_method_cls` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patch_sglang_svdq_wcscales`。 它用于处理 patch sglang svdq wcscales 相关逻辑。 关键调用包括 `getattr`、`isinstance`、`torch.no_grad`、`existing.data.copy_` 和 `tensor.to`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `module`、`tensor` 和 `svdq_method_cls` 等参数驱动。

### Lines 226-283: `_patch_nunchaku_scales` implementation / `_patch_nunchaku_scales` 实现
```python
def _patch_nunchaku_scales(
    model: nn.Module,
    safetensors_list: list[str],
) -> None:
    """Patch transformer module with Nunchaku scale tensors from safetensors weights.

    For NVFP4 checkpoints, correctness depends on `wtscale` and attention
    `wcscales`. The FSDP loader may skip some of these metadata tensors.
    """

    if not safetensors_list:
        return

    if len(safetensors_list) != 1:
        logger.warning(
            "Nunchaku scale patch expects a single safetensors file, "
            "but got %d files. Skipping.",
            len(safetensors_list),
        )
        return

    from nunchaku.models.linear import SVDQW4A4Linear  # type: ignore[import]

    state_dict = safetensors_load_file(safetensors_list[0])
    if state_dict is None:
        return

    num_wtscale = 0
    num_wcscales = 0

    from ..nunchaku_linear import NunchakuSVDQLinearMethod

    for name, module in model.named_modules():
        wt = state_dict.get(f"{name}.wtscale")
        if wt is not None:
            if _patch_native_svdq_linear(module, wt, SVDQW4A4Linear):
                num_wtscale += 1
            elif _patch_sglang_svdq_linear(module, wt, NunchakuSVDQLinearMethod):
                num_wtscale += 1

        wc = state_dict.get(f"{name}.wcscales")
        if wc is not None:
            # Some modules may have wcscales as a direct attribute/Parameter.
            existing = getattr(module, "wcscales", None)
            if isinstance(existing, nn.Parameter):
                with torch.no_grad():
                    existing.data.copy_(wc.to(existing.data.dtype))
                num_wcscales += 1
            elif existing is not None:
                setattr(module, "wcscales", wc)
                num_wcscales += 1
            elif _patch_sglang_svdq_wcscales(module, wc, NunchakuSVDQLinearMethod):
                num_wcscales += 1

    if num_wtscale > 0:
        logger.info("Patched wtscale for %d layers", num_wtscale)
    if num_wcscales > 0:
        logger.info("Patched wcscales for %d layers", num_wcscales)
```
**EN:** This block defines function `_patch_nunchaku_scales`. Patch transformer module with Nunchaku scale tensors from safetensors weights. For NVFP4 checkpoints, correctness depends on `wtscale` and attention `wcscales`. Key calls include `safetensors_load_file`, `model.named_modules`, `len`, `logger.warning`, and `state_dict.get`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `model`, and `safetensors_list` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_patch_nunchaku_scales`。 它用于处理 patch nunchaku scales 相关逻辑。 关键调用包括 `safetensors_load_file`、`model.named_modules`、`len`、`logger.warning` 和 `state_dict.get`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `model` 和 `safetensors_list` 等参数驱动。

## Key Concepts / 关键概念
- `is_nunchaku_available`: Top-level function that handles is nunchaku available logic. / 顶层函数，用于处理 is nunchaku available 相关逻辑。
- `NunchakuConfig`: Configuration for Nunchaku (SVDQuant) W4A4-style quantization. / 核心类，用于封装 nunchaku config 相关行为。
- `_patch_native_svdq_linear`: Top-level function that handles patch native svdq linear logic. / 顶层函数，用于处理 patch native svdq linear 相关逻辑。
- `_patch_sglang_svdq_linear`: Top-level function that handles patch sglang svdq linear logic. / 顶层函数，用于处理 patch sglang svdq linear 相关逻辑。
- `_patch_sglang_svdq_wcscales`: Top-level function that handles patch sglang svdq wcscales logic. / 顶层函数，用于处理 patch sglang svdq wcscales 相关逻辑。
- `_patch_nunchaku_scales`: Patch transformer module with Nunchaku scale tensors from safetensors weights. / 顶层函数，用于处理 patch nunchaku scales 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `dataclasses`, `functools`, `typing`
- **Third-party / 第三方依赖**: `torch`, `safetensors.torch`, `nunchaku.models.linear`, `nunchaku`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `.base_config`, `..nunchaku_linear`

- **Total lines / 总行数**: 283
