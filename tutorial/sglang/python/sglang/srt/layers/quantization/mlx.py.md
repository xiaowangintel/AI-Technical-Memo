# mlx.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/mlx.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for mlx quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 MLX 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module documentation
```python
"""Marker config and auto-detect hook for MLX backend quantization presets.

The MLX backend (``python/sglang/srt/hardware_backend/mlx/``) performs its own
quantization at model load time via :func:`mlx_lm.utils.quantize_model`. The
standard PyTorch ``QuantizationConfig`` machinery is never invoked on that
path.

This module serves two purposes:

1. Registry registration. Listing ``mlx_q4`` and ``mlx_q8`` in
   ``QUANTIZATION_METHODS`` lets :meth:`ModelConfig._verify_quantization`
   recognize them as known methods without backend-specific exceptions in
   the generic config code.

2. Auto-detection for mlx-community HF repos.
   :meth:`override_quantization_method` claims ``config.json`` blocks of
   the form ``{"group_size": <int>, "bits": <int>}`` with no
   ``quant_method`` key and resolves them to the matching preset.
   Already-quantized mlx-community repos load on Apple Silicon without the
   user passing ``--quantization`` on the CLI. Resolves #25119.

The PyTorch path constructors (``from_config``, ``get_quant_method``) raise
``NotImplementedError`` with a clear pointer to ``SGLANG_USE_MLX=1``, since
this class is not a real PyTorch quantization implementation.
"""
```
**EN:** This block stores descriptive documentation for the surrounding module or class.
**CN:** 该代码块为周围模块或类保存说明性文档。

### Lines 27-36: module imports and setup
```python
from __future__ import annotations

from typing import Any, Dict, List, Optional

import torch

from sglang.srt.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
)
```
**EN:** This block imports __future__, torch, typing, sglang.srt.layers.quantization.base_config and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, torch, typing, sglang.srt.layers.quantization.base_config 等依赖，并为当前量化实现准备模块命名空间。

### Lines 39-46: class MlxQuantizationConfig: definition
```python
class MlxQuantizationConfig(QuantizationConfig):
    """Marker config for MLX backend on-the-fly quantization presets.

    Not a real quantization config — the MLX backend handles quantization
    itself. Any standard-PyTorch-path method that touches this class raises
    a helpful error pointing the user at ``SGLANG_USE_MLX=1``.
    """
```
**EN:** This block declares `MlxQuantizationConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, get_name, get_supported_act_dtypes, get_min_capability.
**CN:** 该代码块声明 `MlxQuantizationConfig`，它是量化栈中的配置类，组织了 __init__, get_name, get_supported_act_dtypes, get_min_capability 等行为。

### Lines 47-52: MlxQuantizationConfig member: initialize _ERR
```python
    _ERR = (
        "MLX on-the-fly quantization (--quantization mlx_q4 / mlx_q8) is "
        "handled by the MLX backend at model-load time via mlx_lm.utils."
        "quantize_model, not by this QuantizationConfig class. If you "
        "reached this error, SGLANG_USE_MLX=1 is likely not set."
    )
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _ERR.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _ERR。

### Lines 54-56: MlxQuantizationConfig.__init__()
```python
    def __init__(self, preset: str):
        super().__init__()
        self.preset = preset
```
**EN:** This block defines `MlxQuantizationConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `MlxQuantizationConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 58-60: MlxQuantizationConfig.get_name()
```python
    @classmethod
    def get_name(cls) -> str:
        return "mlx"
```
**EN:** This block defines `MlxQuantizationConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MlxQuantizationConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 62-64: MlxQuantizationConfig.get_supported_act_dtypes()
```python
    @classmethod
    def get_supported_act_dtypes(cls) -> List[torch.dtype]:
        return []
```
**EN:** This block defines `MlxQuantizationConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MlxQuantizationConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 66-69: MlxQuantizationConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        # Capability check is for NVIDIA SM versions; not meaningful for MLX.
        return 0
```
**EN:** This block defines `MlxQuantizationConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MlxQuantizationConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 71-73: MlxQuantizationConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> List[str]:
        return []
```
**EN:** This block defines `MlxQuantizationConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MlxQuantizationConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 75-77: MlxQuantizationConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: Dict[str, Any]) -> "MlxQuantizationConfig":
        raise NotImplementedError(cls._ERR)
```
**EN:** This block defines `MlxQuantizationConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `MlxQuantizationConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 79-119: MlxQuantizationConfig.override_quantization_method()
```python
    @classmethod
    def override_quantization_method(cls, hf_quant_cfg, user_quant) -> Optional[str]:
        """Auto-detect mlx-community-shape quantization configs.

        mlx-community models ship ``config.json`` with::

            "quantization_config": {"group_size": <int>, "bits": <int>}

        No ``quant_method`` key, no other identifying field. Without this
        override, :meth:`ModelConfig._verify_quantization` cannot match the
        shape to any registered method and raises ``Unknown quantization
        method`` (see #25119). Match it here and return the preset whose
        bit-width agrees, so pre-quantized HF repos load on Apple Silicon
        without the user having to pass ``--quantization`` on the CLI.

        Returns ``None`` for any input that does not look like a bare MLX
        preset: non-dict, dict with an explicit ``quant_method``, missing
        keys, non-integer values, or unsupported bit-width. Also defers to
        any explicit ``--quantization`` CLI choice (``user_quant``) per the
        registry contract: CLI selection takes priority over auto-detect.
        """
        if user_quant is not None:
            # User passed --quantization explicitly; respect that choice
            # regardless of the HF config shape. Matches the moe_wna16 /
            # modelopt convention.
            return None
        if not isinstance(hf_quant_cfg, dict):
            return None
        if "quant_method" in hf_quant_cfg:
            # Configs that declare a quant_method belong to whichever method
            # registers under that name; do not hijack them.
            return None
        bits = hf_quant_cfg.get("bits")
        group_size = hf_quant_cfg.get("group_size")
        if not isinstance(bits, int) or not isinstance(group_size, int):
            return None
        if bits == 4:
            return "mlx_q4"
        if bits == 8:
            return "mlx_q8"
        return None
```
**EN:** This block defines `MlxQuantizationConfig.override_quantization_method()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `MlxQuantizationConfig.override_quantization_method()`，用于处理张量或权重量化逻辑。

### Lines 121-124: MlxQuantizationConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional[QuantizeMethodBase]:
        raise NotImplementedError(self._ERR)
```
**EN:** This block defines `MlxQuantizationConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `MlxQuantizationConfig.get_quant_method()`，用于为调用方获取或计算派生值。

## Key Concepts / 关键概念
- `MlxQuantizationConfig`: A configuration class that structures file-level quantization behavior. / `MlxQuantizationConfig` 是一个配置类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.quantization.base_config`
